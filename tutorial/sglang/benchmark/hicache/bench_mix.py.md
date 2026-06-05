# bench_mix.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/bench_mix.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache mix. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 hicache mix 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and setup / 导入与初始化
```python
import argparse
import asyncio
import json
import logging
import os
import queue
import random
import threading
import time
from dataclasses import dataclass
from functools import wraps

import aiohttp

from sglang.bench_serving import RequestFuncOutput
from sglang.benchmark.datasets.random import sample_random_requests
from sglang.benchmark.utils import get_tokenizer, remove_prefix

# Set up logger
logger = logging.getLogger(__name__)

# Set up JSONL file for debug logging
debug_log_file = None
# Create a lock for thread-safe debug log writing
debug_log_lock = threading.Lock()
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

### Lines 28-39: Function `write_debug_log` / 函数 `write_debug_log`
```python
def write_debug_log(data):
    global debug_log_file

    """Write debug information to a JSONL file"""
    if debug_log_file is None:
        return

    # Acquire lock for thread-safe writing
    with debug_log_lock:
        # Write as JSONL (JSON Line format)
        debug_log_file.write(json.dumps(data) + "\n")
        debug_log_file.flush()
```
**EN:** `write_debug_log` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. Notable calls include `debug_log_file.write`, `debug_log_file.flush`, `json.dumps`.
**CN:** `write_debug_log` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。其中较关键的调用包括 `debug_log_file.write`, `debug_log_file.flush`, `json.dumps`。

### Lines 42-89: Function `parse_args` / 函数 `parse_args`
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Script to benchmark concurrent requests to a server."
    )
    parser.add_argument(
        "--model-path",
        type=str,
        default="/data/models/Qwen3-0.6B",
        help="model path compatible with Hugging Face Transformers",
    )
    parser.add_argument(
        "--dataset-path",
        type=str,
        default="/data/models/ShareGPT_V3_unfiltered_cleaned_split/ShareGPT_V3_unfiltered_cleaned_split.json",
        help="local dataset to sample tokens from",
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
        "--duration",
        type=int,
        default=600,
        help="Duration to run the benchmark in seconds (default: 300 seconds)",
    )
    parser.add_argument(
        "--log-level",
        type=str,
        default="info",
        choices=["debug", "info"],
        help="Set the logging level (default: info)",
    )
    parser.add_argument(
        "--debug-log-file",
        type=str,
        default="debug.log.jsonl",
        help="File to write debug logs in JSONL format",
    )
    return parser.parse_args()
```
**EN:** `parse_args` is a function that builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. It returns `parser.parse_args()` to the caller. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `parse_args` 是一个函数，用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `parser.parse_args()`。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 92-121: Function `load_config` / 函数 `load_config`
```python
def load_config():
    config_path = os.getenv("CONFIG_PATH")
    if not config_path:
        raise ValueError("Environment variable 'CONFIG_PATH' is not set.")

    with open(config_path, "r") as f:
        config = json.load(f)

    required_keys = [
        "num_rounds",
        "num_clients",
        "round_ratios",
        "mean_new_tokens_per_round",
        "mean_return_tokens_per_round",
        "mean_inter_round_interval",
    ]

    for key in required_keys:
        if key not in config:
            raise KeyError(f"Missing required configuration key: {key}")

    num_rounds = config["num_rounds"]
    assert len(config["round_ratios"]) == num_rounds
    assert len(config["mean_new_tokens_per_round"]) == num_rounds
    assert len(config["mean_return_tokens_per_round"]) == num_rounds
    assert len(config["mean_inter_round_interval"]) == num_rounds

    print(config)

    return config
```
**EN:** `load_config` is a function that loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and reads or writes local files and intermediate artifacts. It returns `config` to the caller. Notable calls include `os.getenv`, `print`, `ValueError`.
**CN:** `load_config` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、读写本地文件及中间产物。它会向调用方返回 `config`。其中较关键的调用包括 `os.getenv`, `print`, `ValueError`。

### Lines 125-125: Class `UserData` declaration / 类 `UserData` 声明
```python
class UserData:
```
**EN:** This block introduces class `UserData`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `UserData`，用于通过统一接口组织相关行为。

### Lines 126-131: Class-level state / 类级状态
```python
    user_id: int
    current_round: int
    total_rounds: int
    prompt: str
    return_tokens: int
    start: int
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it processes tokenized prompts or decoded outputs.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会处理分词后的提示词或解码后的输出。

### Lines 134-143: Function `synchronized` / 函数 `synchronized`
```python
def synchronized():
    def _decorator(func):
        @wraps(func)
        def wrapper(self, *args, **kwargs):
            with self.lock:
                return func(self, *args, **kwargs)

        return wrapper

    return _decorator
```
**EN:** `synchronized` is a function that implements the core logic for this scope. It returns `_decorator` to the caller. Notable calls include `wraps`, `func`.
**CN:** `synchronized` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `_decorator`。其中较关键的调用包括 `wraps`, `func`。

### Lines 146-146: Class `UserGenerator` declaration / 类 `UserGenerator` 声明
```python
class UserGenerator:
```
**EN:** This block introduces class `UserGenerator`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `UserGenerator`，用于通过统一接口组织相关行为。

### Lines 147-193: Method `UserGenerator.__init__` / 方法 `UserGenerator.__init__`
```python
    def __init__(self, config, model_path, dataset_path):
        self.tokenizer_path = model_path
        self.tokenizer = get_tokenizer(self.tokenizer_path)
        self.dataset_path = dataset_path

        self.user_id = 0
        self.lock = threading.Lock()

        self.num_rounds = config["num_rounds"]

        self.cumulative_ratios = [
            sum(config["round_ratios"][: i + 1])
            for i in range(len(config["round_ratios"]))
        ]
        self.mean_new_tokens_per_round = config["mean_new_tokens_per_round"]
        self.mean_return_tokens_per_round = config["mean_return_tokens_per_round"]
        self.mean_inter_round_interval = config["mean_inter_round_interval"]

        self.sigma = 100
        self.range_ratio = 0.8
        assert self.range_ratio <= 1

        self.candidate_inputs = [
            [
                r
                for r in sample_random_requests(
                    input_len=(
                        self.mean_new_tokens_per_round[i] * (2 - self.range_ratio)
                    ),
                    output_len=(
                        self.mean_return_tokens_per_round[i] * (2 - self.range_ratio)
                    ),
                    num_prompts=config["num_clients"],
                    range_ratio=self.range_ratio / (2 - self.range_ratio),
                    tokenizer=self.tokenizer,
                    dataset_path=self.dataset_path,
                    random_sample=False,
                )
            ]
            for i in range(self.num_rounds)
        ]

        self.multiturn_queue = []

        self.user_stats = [0 for _ in range(self.num_rounds)]
        self.input_stats = [[0, 0] for _ in range(self.num_rounds)]
        self.output_stats = [[0, 0] for _ in range(self.num_rounds)]
```
**EN:** `UserGenerator.__init__` is a method that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. Notable calls include `get_tokenizer`, `threading.Lock`, `sum`.
**CN:** `UserGenerator.__init__` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `get_tokenizer`, `threading.Lock`, `sum`。

### Lines 195-227: Method `UserGenerator.gen` / 方法 `UserGenerator.gen`
```python
    def gen(self):
        user_id = self.user_id
        self.user_id += 1

        rand_ratio = random.randint(0, self.cumulative_ratios[-1])
        i = len(self.cumulative_ratios)
        for idx, cumulative_ratio in enumerate(self.cumulative_ratios):
            if rand_ratio >= cumulative_ratio:
                continue
            else:
                i = idx + 1
                break
        total_rounds = i
        current_round = 0

        candidate_input = random.sample(self.candidate_inputs[current_round], 1)[0]
        self.input_stats[0][0] += candidate_input.prompt_len
        self.input_stats[0][1] += 1
        prompt = f"{user_id} " + candidate_input.prompt
        return_tokens = int(
            random.gauss(self.mean_return_tokens_per_round[current_round], self.sigma)
        )
        if return_tokens <= 0:
            return_tokens = self.mean_return_tokens_per_round[current_round]
        start = 0

        user_data = UserData(
            user_id, current_round, total_rounds, prompt, return_tokens, start
        )

        self.user_stats[total_rounds - 1] += 1

        return user_data
```
**EN:** `UserGenerator.gen` is a method that processes tokenized prompts or decoded outputs. It returns `user_data` to the caller. Notable calls include `random.randint`, `len`, `enumerate`.
**CN:** `UserGenerator.gen` 是一个方法，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `user_data`。其中较关键的调用包括 `random.randint`, `len`, `enumerate`。

### Lines 230-267: Method `UserGenerator.push` / 方法 `UserGenerator.push`
```python
    def push(self, user_data, generated_text, len_itl):
        self.output_stats[user_data.current_round][0] += len_itl + 1
        self.output_stats[user_data.current_round][1] += 1
        user_data.current_round += 1
        if user_data.current_round >= user_data.total_rounds:
            return

        candidate_input = random.sample(
            self.candidate_inputs[user_data.current_round], 1
        )[0]
        self.input_stats[user_data.current_round][0] += candidate_input.prompt_len
        self.input_stats[user_data.current_round][1] += 1
        user_data.prompt += generated_text + candidate_input.prompt
        user_data.return_tokens = int(
            random.gauss(
                self.mean_return_tokens_per_round[user_data.current_round], self.sigma
            )
        )
        if user_data.return_tokens <= 0:
            user_data.return_tokens = self.mean_return_tokens_per_round[
                user_data.current_round
            ]
        interval = random.gauss(
            self.mean_inter_round_interval[user_data.current_round], self.sigma
        )
        if interval <= 0:
            interval = self.mean_inter_round_interval[user_data.current_round]
        user_data.start = time.perf_counter() + interval

        if len(self.multiturn_queue) == 0:
            self.multiturn_queue.append(user_data)
        else:
            i = len(self.multiturn_queue)
            for idx, d in enumerate(self.multiturn_queue):
                if user_data.start < d.start:
                    i = idx
                    break
            self.multiturn_queue.insert(idx, user_data)
```
**EN:** `UserGenerator.push` is a method that measures runtime latency, throughput, or other benchmark metrics and processes tokenized prompts or decoded outputs. Notable calls include `synchronized`, `int`, `random.gauss`.
**CN:** `UserGenerator.push` 是一个方法，用于测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `synchronized`, `int`, `random.gauss`。

### Lines 270-276: Method `UserGenerator.pop` / 方法 `UserGenerator.pop`
```python
    def pop(self):
        if (
            len(self.multiturn_queue)
            and time.perf_counter() > self.multiturn_queue[0].start
        ):
            return self.multiturn_queue.pop(0)
        return self.gen()
```
**EN:** `UserGenerator.pop` is a method that measures runtime latency, throughput, or other benchmark metrics. It returns `self.gen()` to the caller. Notable calls include `synchronized`, `self.gen`, `len`.
**CN:** `UserGenerator.pop` 是一个方法，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `self.gen()`。其中较关键的调用包括 `synchronized`, `self.gen`, `len`。

### Lines 279-293: Function `gen_payload` / 函数 `gen_payload`
```python
def gen_payload(prompt, output_len):
    payload = {
        "text": prompt,
        "sampling_params": {
            "temperature": 0.0,
            "max_new_tokens": output_len,
            "ignore_eos": True,
        },
        "stream": True,
        "stream_options": {"include_usage": True},
        "lora_path": "",
        "return_logprob": False,
        "logprob_start_len": -1,
    }
    return payload
```
**EN:** `gen_payload` is a function that processes tokenized prompts or decoded outputs. It returns `payload` to the caller.
**CN:** `gen_payload` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `payload`。

### Lines 296-296: Top-level execution logic / 顶层执行逻辑
```python
AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=20 * 60 * 60)
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。

### Lines 299-368: Async function `async_request_sglang_generate` / 异步函数 `async_request_sglang_generate`
```python
async def async_request_sglang_generate(
    user_data,
    url,
    atomic_counter,
):
    """
    Sends a streaming request to the server. Gathers text token-by-token.
    """
    async with aiohttp.ClientSession(timeout=AIOHTTP_TIMEOUT) as session:
        headers = {}
        generated_text = ""
        ttft = 0.0
        st = time.perf_counter()
        most_recent_timestamp = st
        output = RequestFuncOutput()
        payload = gen_payload(user_data.prompt, user_data.return_tokens)
        write_debug_log({"timestamp": st, "user_data": user_data.__dict__})

        try:
            async with session.post(url=url, json=payload, headers=headers) as response:
                if response.status == 200:
                    prompt_tokens = 0
                    cached_tokens = 0
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

                            if data.get("text"):
                                timestamp = time.perf_counter()
                                # First token
                                if ttft == 0.0:
                                    ttft = time.perf_counter() - st
                                    output.ttft = ttft
                                    prompt_tokens = (data.get("meta_info") or {}).get(
                                        "prompt_tokens", 0
                                    )
                                    cached_tokens = (data.get("meta_info") or {}).get(
                                        "cached_tokens", 0
                                    )

                                # Decoding phase
                                else:
                                    output.itl.append(timestamp - most_recent_timestamp)

                                most_recent_timestamp = timestamp
                                generated_text = data["text"]

                    output.generated_text = generated_text
                    output.success = True
                    output.latency = latency
                    output.prompt_len = prompt_tokens
                    output.cached_tokens = cached_tokens
                else:
                    output.error = response.reason or ""
                    output.success = False
        except Exception as e:
            output.success = False
            output.error = str(e)
            print(f"Request failed: {e}")

    atomic_counter.increment(1)
    return output
```
**EN:** `async_request_sglang_generate` is a async function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. The docstring frames it as: Sends a streaming request to the server. It returns `output` to the caller. Notable calls include `atomic_counter.increment`, `aiohttp.ClientSession`, `time.perf_counter`.
**CN:** `async_request_sglang_generate` 是一个异步函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `output`。其中较关键的调用包括 `atomic_counter.increment`, `aiohttp.ClientSession`, `time.perf_counter`。

### Lines 371-371: Class `AtomicCounter` declaration / 类 `AtomicCounter` 声明
```python
class AtomicCounter:
```
**EN:** This block introduces class `AtomicCounter`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `AtomicCounter`，用于通过统一接口组织相关行为。

### Lines 372-374: Method `AtomicCounter.__init__` / 方法 `AtomicCounter.__init__`
```python
    def __init__(self, initial_value=0):
        self._value = initial_value
        self.lock = threading.Lock()
```
**EN:** `AtomicCounter.__init__` is a method that implements the core logic for this scope. Notable calls include `threading.Lock`.
**CN:** `AtomicCounter.__init__` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `threading.Lock`。

### Lines 377-378: Method `AtomicCounter.increment` / 方法 `AtomicCounter.increment`
```python
    def increment(self, amount=1):
        self._value += amount
```
**EN:** `AtomicCounter.increment` is a method that implements the core logic for this scope. Notable calls include `synchronized`.
**CN:** `AtomicCounter.increment` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `synchronized`。

### Lines 381-382: Method `AtomicCounter.get` / 方法 `AtomicCounter.get`
```python
    def get(self):
        return self._value
```
**EN:** `AtomicCounter.get` is a method that implements the core logic for this scope. It returns `self._value` to the caller. Notable calls include `synchronized`.
**CN:** `AtomicCounter.get` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `self._value`。其中较关键的调用包括 `synchronized`。

### Lines 385-385: Class `WorkloadGenerator` declaration / 类 `WorkloadGenerator` 声明
```python
class WorkloadGenerator:
```
**EN:** This block introduces class `WorkloadGenerator`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `WorkloadGenerator`，用于通过统一接口组织相关行为。

### Lines 386-415: Method `WorkloadGenerator.__init__` / 方法 `WorkloadGenerator.__init__`
```python
    def __init__(self, args):
        config = load_config()
        user_generator = UserGenerator(
            config,
            args.model_path,
            args.dataset_path,
        )

        self.url = f"http://{args.host}:{args.port}/generate"

        self.tokenizer = user_generator.tokenizer
        self.start_time = None
        self.finished_time = None
        self.duration = args.duration
        self.done = False

        self.sent_requests = 0
        self.completed_requests = 0

        self.user_generator = user_generator
        self.response_queue = queue.Queue()
        self.performance_metrics = {
            "ttft": [],
            "latency": [],
            "prompt_len": [],
            "cached_tokens": [],
        }
        self.max_parallel = config["num_clients"]

        self.atomic_counter = AtomicCounter()
```
**EN:** `WorkloadGenerator.__init__` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `load_config`, `UserGenerator`, `queue.Queue`.
**CN:** `WorkloadGenerator.__init__` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `load_config`, `UserGenerator`, `queue.Queue`。

### Lines 417-425: Async method `WorkloadGenerator.handle_request` / 异步方法 `WorkloadGenerator.handle_request`
```python
    async def handle_request(self, user_data):
        try:
            response = await async_request_sglang_generate(
                user_data, self.url, self.atomic_counter
            )
            self.response_queue.put((user_data, response))
        except Exception as e:
            print(f"Request failed: {e}")
            self.completed_requests += 1
```
**EN:** `WorkloadGenerator.handle_request` is a async method that sends requests to serving or OpenAI-compatible APIs and coordinates asynchronous or parallel execution. Notable calls include `self.response_queue.put`, `async_request_sglang_generate`, `print`.
**CN:** `WorkloadGenerator.handle_request` 是一个异步方法，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行。其中较关键的调用包括 `self.response_queue.put`, `async_request_sglang_generate`, `print`。

### Lines 427-453: Method `WorkloadGenerator.request_sender` / 方法 `WorkloadGenerator.request_sender`
```python
    def request_sender(self):
        async def request_loop():
            tasks = []
            while True:
                if self.sent_requests - self.completed_requests < self.max_parallel:
                    new_request = self.user_generator.pop()
                    if new_request:
                        task = asyncio.create_task(self.handle_request(new_request))
                        tasks.append(task)
                        self.sent_requests += 1
                else:
                    await asyncio.sleep(0.05)
                    continue

                if time.perf_counter() - self.start_time > self.duration:
                    self.done = True
                    break

            # Cancel all pending tasks and wait for them to finish
            for task in tasks:
                task.cancel()
            await asyncio.gather(*tasks, return_exceptions=True)

        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        loop.run_until_complete(request_loop())
        loop.close()
```
**EN:** `WorkloadGenerator.request_sender` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `asyncio.new_event_loop`, `asyncio.set_event_loop`, `loop.run_until_complete`.
**CN:** `WorkloadGenerator.request_sender` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `asyncio.new_event_loop`, `asyncio.set_event_loop`, `loop.run_until_complete`。

### Lines 455-480: Method `WorkloadGenerator.response_handler` / 方法 `WorkloadGenerator.response_handler`
```python
    def response_handler(self):
        while True:
            try:
                user_data, response = self.response_queue.get(timeout=10)
                logger.info(
                    f"{((time.perf_counter()-self.start_time)/self.duration*100):.2f}%"
                )
                if not response.success:
                    raise ValueError(f"Request failed with error: {response.error}")

                self.user_generator.push(
                    user_data, response.generated_text, len(response.itl)
                )
                self.performance_metrics["ttft"].append(response.ttft)
                self.performance_metrics["latency"].append(response.latency)
                self.performance_metrics["prompt_len"].append(response.prompt_len)
                self.performance_metrics["cached_tokens"].append(response.cached_tokens)
                self.completed_requests += 1
                self.finished_time = time.perf_counter()

            except queue.Empty:
                if self.done:
                    break
            except ValueError as e:
                print(f"Error processing response for client {user_data}: {e}")
                continue
```
**EN:** `WorkloadGenerator.response_handler` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. Notable calls include `self.response_queue.get`, `logger.info`, `self.user_generator.push`.
**CN:** `WorkloadGenerator.response_handler` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `self.response_queue.get`, `logger.info`, `self.user_generator.push`。

### Lines 482-548: Method `WorkloadGenerator.run` / 方法 `WorkloadGenerator.run`
```python
    def run(self):
        request_thread = threading.Thread(target=self.request_sender, daemon=True)
        response_thread = threading.Thread(target=self.response_handler, daemon=True)

        self.start_time = time.perf_counter()
        request_thread.start()
        response_thread.start()

        request_thread.join()
        response_thread.join()

        performance_data = {
            "summary": {
                "total_requests": len(self.performance_metrics["ttft"]),
                "average_ttft": sum(self.performance_metrics["ttft"])
                / len(self.performance_metrics["ttft"]),
                "p90_ttft": sorted(self.performance_metrics["ttft"])[
                    int(0.9 * len(self.performance_metrics["ttft"]))
                ],
                "median_ttft": sorted(self.performance_metrics["ttft"])[
                    len(self.performance_metrics["ttft"]) // 2
                ],
                "average_latency": sum(self.performance_metrics["latency"])
                / len(self.performance_metrics["latency"]),
                "p90_latency": sorted(self.performance_metrics["latency"])[
                    int(0.9 * len(self.performance_metrics["latency"]))
                ],
                "median_latency": sorted(self.performance_metrics["latency"])[
                    len(self.performance_metrics["latency"]) // 2
                ],
                "throughput": self.atomic_counter.get()
                / (self.finished_time - self.start_time),
                "cache_hit_rate": (
                    0
                    if sum(self.performance_metrics["prompt_len"]) == 0
                    else sum(self.performance_metrics["cached_tokens"])
                    / sum(self.performance_metrics["prompt_len"])
                ),
            },
        }
        print("All requests completed")
        print("Performance metrics summary:")
        print(f"  Total requests: {performance_data['summary']['total_requests']}")
        print(f"  Average TTFT: {performance_data['summary']['average_ttft']:.2f}")
        print(f"  P90 TTFT: {performance_data['summary']['p90_ttft']:.2f}")
        print(f"  Median TTFT: {performance_data['summary']['median_ttft']:.2f}")
        print(
            f"  Average latency: {performance_data['summary']['average_latency']:.2f}"
        )
        print(f"  P90 latency: {performance_data['summary']['p90_latency']:.2f}")
        print(f"  Median latency: {performance_data['summary']['median_latency']:.2f}")
        print(
            f"  Throughput: {performance_data['summary']['throughput']:.2f} requests per second"
        )
        print(f"  Cache Hit Rate: {performance_data['summary']['cache_hit_rate']:.6f}")

        user_stats = self.user_generator.user_stats
        input_stats = self.user_generator.input_stats
        output_stats = self.user_generator.output_stats
        print(f"round_ratios: {user_stats}")
        print(
            f"mean_new_tokens_per_round: {[int(a/b) if b > 0 else 0 for a, b in input_stats]}"
        )
        print(
            f"mean_return_tokens_per_round: {[int(a/b) if b > 0 else 0 for a, b in output_stats]}"
        )
        return performance_data
```
**EN:** `WorkloadGenerator.run` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. It returns `performance_data` to the caller. Notable calls include `threading.Thread`, `time.perf_counter`, `request_thread.start`.
**CN:** `WorkloadGenerator.run` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。它会向调用方返回 `performance_data`。其中较关键的调用包括 `threading.Thread`, `time.perf_counter`, `request_thread.start`。

### Lines 551-567: Function `main` / 函数 `main`
```python
def main():
    global debug_log_file

    args = parse_args()
    if args.log_level == "debug":
        logging.basicConfig(level=logging.DEBUG)
        logger.info("use log_level debug")
        # Initialize debug log file
        debug_log_file = open(args.debug_log_file, "w")
    else:
        logging.basicConfig(level=logging.INFO)
        logger.info("use log_level info")
    performance_data = WorkloadGenerator(args).run()

    # Close debug log file if it was opened
    if debug_log_file:
        debug_log_file.close()
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration and reads or writes local files and intermediate artifacts. Notable calls include `parse_args`, `WorkloadGenerator(args).run`, `logging.basicConfig`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、读写本地文件及中间产物。其中较关键的调用包括 `parse_args`, `WorkloadGenerator(args).run`, `logging.basicConfig`。

### Lines 570-571: CLI entry point / 命令行入口
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
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `logging`, `os`, `queue`, `random`, `threading`, `time`, `dataclasses`, `functools`
- **Third-party / 第三方依赖**: `aiohttp`
- **Internal / 项目内部依赖**: `sglang.bench_serving`, `sglang.benchmark.datasets.random`, `sglang.benchmark.utils`
