# benchmark_serving_multi_turn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/multi_turn/benchmark_serving_multi_turn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, serving-side benchmarks, centered around `NUM_TOKENS_FROM_DATASET`, `TERM_SIGNAL`, `ConversationSampling`, `ClientArgs`. / 实现与基准测试编排、服务侧基准相关的逻辑，核心符号包括 `NUM_TOKENS_FROM_DATASET`, `TERM_SIGNAL`, `ConversationSampling`, `ClientArgs`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-33)
```python
import argparse
import asyncio
import json
import logging
import multiprocessing as mp
import os
import random
import time
from collections import Counter, deque
from datetime import datetime
from enum import Enum
from http import HTTPStatus
from statistics import mean
from typing import NamedTuple

import aiohttp  # type: ignore
import numpy as np  # type: ignore
import pandas as pd  # type: ignore
from bench_dataset import (
    ConversationsMap,
    ConvId,
    GenConvArgs,
    MessagesList,
    ShareGptConversations,
    conversations_dict_to_list,
    conversations_list_to_dict,
    generate_conversations,
    parse_input_json_file,
)
from bench_utils import TEXT_SEPARATOR, Color, logger
from transformers import AutoTokenizer  # type: ignore
```
**EN:** This block gathers standard-library helpers such as `argparse`, `asyncio`, `json`, `logging`, `multiprocessing`, `os`; third-party packages such as `aiohttp`, `numpy`, `pandas`, `transformers`; project-local modules such as `bench_dataset`, `bench_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `asyncio`, `json`, `logging`, `multiprocessing`, `os`；第三方依赖，如 `aiohttp`, `numpy`, `pandas`, `transformers`；项目内部模块，如 `bench_dataset`, `bench_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 35-36)
```python
NUM_TOKENS_FROM_DATASET = 0
TERM_SIGNAL = None
```
**EN:** This top-level block prepares shared state such as `NUM_TOKENS_FROM_DATASET`, `TERM_SIGNAL`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `NUM_TOKENS_FROM_DATASET`, `TERM_SIGNAL`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `ConversationSampling` (lines 39-44)
```python
class ConversationSampling(str, Enum):
    ROUND_ROBIN = "round_robin"
    RANDOM = "random"

    def __str__(self):
        return self.value
```
**EN:** Class `ConversationSampling` packages shared state and related operations for this benchmark module. It extends `str`, `Enum` and exposes methods such as `__str__`.
**CN:** 类 `ConversationSampling` 用于封装该基准模块的共享状态与相关操作。它继承自 `str`, `Enum`，并提供 `__str__` 等方法。

### Class `ClientArgs` (lines 47-58)
```python
class ClientArgs(NamedTuple):
    seed: int
    max_num_requests: int | None
    skip_first_turn: bool
    max_turns: int | None
    max_active_conversations: int
    verbose: bool
    print_content: bool
    verify_output: bool
    conversation_sampling: ConversationSampling
    request_rate: float
    max_retries: int
```
**EN:** Class `ClientArgs` packages shared state and related operations for this benchmark module. It extends `NamedTuple` and exposes methods such as its methods.
**CN:** 类 `ClientArgs` 用于封装该基准模块的共享状态与相关操作。它继承自 `NamedTuple`，并提供 its methods 等方法。

### Class `RequestArgs` (lines 61-67)
```python
class RequestArgs(NamedTuple):
    chat_url: str
    model: str
    stream: bool
    limit_min_tokens: int  # Use negative value for no limit
    limit_max_tokens: int  # Use negative value for no limit
    timeout_sec: int
```
**EN:** Class `RequestArgs` packages shared state and related operations for this benchmark module. It extends `NamedTuple` and exposes methods such as its methods.
**CN:** 类 `RequestArgs` 用于封装该基准模块的共享状态与相关操作。它继承自 `NamedTuple`，并提供 its methods 等方法。

### Class `BenchmarkArgs` (lines 70-73)
```python
class BenchmarkArgs(NamedTuple):
    url: str
    num_clients: int
    early_stop: bool
```
**EN:** Class `BenchmarkArgs` packages shared state and related operations for this benchmark module. It extends `NamedTuple` and exposes methods such as its methods.
**CN:** 类 `BenchmarkArgs` 用于封装该基准模块的共享状态与相关操作。它继承自 `NamedTuple`，并提供 its methods 等方法。

### Class `ServerResponse` (lines 76-87)
```python
class ServerResponse(NamedTuple):
    valid: bool
    ttft_ms: float  # time to first chunk
    tpot_ms: float  # time per output chunk (one or more tokens)
    latency_ms: float
    start_time_ms: float
    first_chunk: str  # first chunk of the content
    content: str  # includes the first_chunk
    num_chunks: int

    def __str__(self) -> str:
        return f"ttft_ms {self.ttft_ms:.2f}, tpot_ms {self.tpot_ms:.2f}, latency_ms {self.latency_ms:.2f}"  # noqa: E501
```
**EN:** Class `ServerResponse` packages shared state and related operations for this benchmark module. It extends `NamedTuple` and exposes methods such as `__str__`.
**CN:** 类 `ServerResponse` 用于封装该基准模块的共享状态与相关操作。它继承自 `NamedTuple`，并提供 `__str__` 等方法。

### Class `RequestStats` (lines 90-109)
```python
class RequestStats(NamedTuple):
    ttft_ms: float
    tpot_ms: float
    latency_ms: float
    start_time_ms: float
    input_num_turns: int
    input_num_tokens: int
    output_num_tokens: int
    output_num_chunks: int
    output_num_first_chunk_tokens: int
    approx_cached_percent: float
    conversation_id: str
    client_id: int

    def __str__(self) -> str:
        return (
            f"ttft_ms {self.ttft_ms:.2f}, tpot_ms {self.tpot_ms:.2f}, latency_ms {self.latency_ms:.2f}, input_num_tokens {self.input_num_tokens}, "  # noqa: E501
            f"output_num_tokens {self.output_num_tokens} ({self.output_num_chunks} chunks, {self.output_num_first_chunk_tokens} tokens in first chunk), "  # noqa: E501
            f"approx_cached_percent {self.approx_cached_percent:.2f}%"
        )
```
**EN:** Class `RequestStats` packages shared state and related operations for this benchmark module. It extends `NamedTuple` and exposes methods such as `__str__`.
**CN:** 类 `RequestStats` 用于封装该基准模块的共享状态与相关操作。它继承自 `NamedTuple`，并提供 `__str__` 等方法。

### Class `MetricStats` (lines 112-138)
```python
class MetricStats:
    def __init__(self) -> None:
        self.min: float | None = None
        self.max: float | None = None
        self.avg: float | None = None
        self.sum = 0.0
        self.count = 0

    def update(self, value: float) -> None:
        if self.min is None:
            self.min = value
        else:
            self.min = min(self.min, value)

        if self.max is None:
            self.max = value
        else:
            self.max = max(self.max, value)

        self.sum += value
        self.count += 1
        self.avg = self.sum / self.count

    def __repr__(self) -> str:
        if self.count == 0:
            return "no data"
        return f"avg: {self.avg:>10.3f}, min: {self.min:>10.3f}, max: {self.max:>10.3f}"
```
**EN:** Class `MetricStats` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `update`, `__repr__`.
**CN:** 类 `MetricStats` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `update`, `__repr__` 等方法。

### Method `MetricStats.__init__` (lines 113-118)
```python
    def __init__(self) -> None:
        self.min: float | None = None
        self.max: float | None = None
        self.avg: float | None = None
        self.sum = 0.0
        self.count = 0
```
**EN:** `__init__` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with no explicit parameters and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `MetricStats.update` (lines 120-133)
```python
    def update(self, value: float) -> None:
        if self.min is None:
            self.min = value
        else:
            self.min = min(self.min, value)

        if self.max is None:
            self.max = value
        else:
            self.max = max(self.max, value)

        self.sum += value
        self.count += 1
        self.avg = self.sum / self.count
```
**EN:** `update` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `value` and relies on `min`, `max` plus branching to move data through this part of the benchmark pipeline.
**CN:** `update` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `value`，并结合 `min`, `max` 以及 条件分支 来完成这一段基准测试流程。

### Class `MovingAverage` (lines 141-168)
```python
class MovingAverage:
    def __init__(self, window_size: int) -> None:
        self.window_size = window_size
        self.window = np.zeros(window_size)
        self.index = 0
        self.sum = 0.0
        self.count = 0
        self.avg: float | None = None

    def update(self, new_value: float) -> None:
        if self.count < self.window_size:
            # Filling up the window
            self.sum += new_value
            self.window[self.count] = new_value
            self.count += 1
        else:
            # Window is full, start replacing old values
            old_value = self.window[self.index]
            self.sum = self.sum - old_value + new_value
            self.window[self.index] = new_value
            self.index = (self.index + 1) % self.window_size

        self.avg = self.sum / self.count

    def __repr__(self) -> str:
        if self.count == 0:
            return "no data"
        return f"avg: {self.avg:>10.3f} ({self.count} samples)"
```
**EN:** Class `MovingAverage` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `update`, `__repr__`.
**CN:** 类 `MovingAverage` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `update`, `__repr__` 等方法。

### Method `MovingAverage.__init__` (lines 142-148)
```python
    def __init__(self, window_size: int) -> None:
        self.window_size = window_size
        self.window = np.zeros(window_size)
        self.index = 0
        self.sum = 0.0
        self.count = 0
        self.avg: float | None = None
```
**EN:** `__init__` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `window_size` and relies on `np.zeros` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `window_size`，并结合 `np.zeros` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `MovingAverage.update` (lines 150-163)
```python
    def update(self, new_value: float) -> None:
        if self.count < self.window_size:
            # Filling up the window
            self.sum += new_value
            self.window[self.count] = new_value
            self.count += 1
        else:
            # Window is full, start replacing old values
            old_value = self.window[self.index]
            self.sum = self.sum - old_value + new_value
            self.window[self.index] = new_value
            self.index = (self.index + 1) % self.window_size

        self.avg = self.sum / self.count
```
**EN:** `update` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `new_value` and relies on direct statements plus branching to move data through this part of the benchmark pipeline.
**CN:** `update` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `new_value`，并结合 direct statements 以及 条件分支 来完成这一段基准测试流程。

### Class `DebugStats` (lines 171-200)
```python
class DebugStats:
    def __init__(self, logger: logging.Logger, window_size: int) -> None:
        self.logger = logger
        self.metrics: dict[str, MovingAverage | MetricStats] = {
            "moving_avg_ttft_ms": MovingAverage(window_size),
            "moving_avg_tpot_ms": MovingAverage(window_size),
            "ttft_ms": MetricStats(),
            "tpot_ms": MetricStats(),
            "latency_ms": MetricStats(),
            "input_num_turns": MetricStats(),
            "input_num_tokens": MetricStats(),
            "output_num_tokens": MetricStats(),
        }

    def update(self, data: RequestStats) -> None:
        self.metrics["ttft_ms"].update(data.ttft_ms)
        self.metrics["moving_avg_ttft_ms"].update(data.ttft_ms)
        self.metrics["tpot_ms"].update(data.tpot_ms)
        self.metrics["moving_avg_tpot_ms"].update(data.tpot_ms)
        self.metrics["latency_ms"].update(data.latency_ms)
        self.metrics["input_num_turns"].update(data.input_num_turns)
        self.metrics["input_num_tokens"].update(data.input_num_tokens)
        self.metrics["output_num_tokens"].update(data.output_num_tokens)

    def print(self) -> None:
        self.logger.info("-" * 50)
        for k, v in self.metrics.items():
            kv_info = f"[{k:25}] {v}"
            self.logger.info(kv_info)
        self.logger.info("-" * 50)
```
**EN:** Class `DebugStats` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `update`, `print`.
**CN:** 类 `DebugStats` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `update`, `print` 等方法。

### Method `DebugStats.__init__` (lines 172-183)
```python
    def __init__(self, logger: logging.Logger, window_size: int) -> None:
        self.logger = logger
        self.metrics: dict[str, MovingAverage | MetricStats] = {
            "moving_avg_ttft_ms": MovingAverage(window_size),
            "moving_avg_tpot_ms": MovingAverage(window_size),
            "ttft_ms": MetricStats(),
            "tpot_ms": MetricStats(),
            "latency_ms": MetricStats(),
            "input_num_turns": MetricStats(),
            "input_num_tokens": MetricStats(),
            "output_num_tokens": MetricStats(),
        }
```
**EN:** `__init__` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `logger`, `window_size` and relies on `MovingAverage`, `MetricStats` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `logger`, `window_size`，并结合 `MovingAverage`, `MetricStats` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `DebugStats.update` (lines 185-193)
```python
    def update(self, data: RequestStats) -> None:
        self.metrics["ttft_ms"].update(data.ttft_ms)
        self.metrics["moving_avg_ttft_ms"].update(data.ttft_ms)
        self.metrics["tpot_ms"].update(data.tpot_ms)
        self.metrics["moving_avg_tpot_ms"].update(data.tpot_ms)
        self.metrics["latency_ms"].update(data.latency_ms)
        self.metrics["input_num_turns"].update(data.input_num_turns)
        self.metrics["input_num_tokens"].update(data.input_num_tokens)
        self.metrics["output_num_tokens"].update(data.output_num_tokens)
```
**EN:** `update` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `data` and relies on `self.metrics.update` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `update` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `data`，并结合 `self.metrics.update` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `DebugStats.print` (lines 195-200)
```python
    def print(self) -> None:
        self.logger.info("-" * 50)
        for k, v in self.metrics.items():
            kv_info = f"[{k:25}] {v}"
            self.logger.info(kv_info)
        self.logger.info("-" * 50)
```
**EN:** `print` formats results for display or export. It mainly works with no explicit parameters and relies on `self.logger.info`, `self.metrics.items` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `print` 负责格式化结果以便展示或导出。 它主要处理 无显式参数，并结合 `self.logger.info`, `self.metrics.items` 以及 循环迭代 来完成这一段基准测试流程。

### Function `nanosec_to_millisec` (lines 203-204)
```python
def nanosec_to_millisec(value: float) -> float:
    return value / 1000000.0
```
**EN:** `nanosec_to_millisec` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `value` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `nanosec_to_millisec` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `value`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `nanosec_to_sec` (lines 207-208)
```python
def nanosec_to_sec(value: float) -> float:
    return value / 1000000000.0
```
**EN:** `nanosec_to_sec` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `value` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `nanosec_to_sec` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `value`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Async function `send_request` (lines 211-337)
```python
async def send_request(
    session: aiohttp.ClientSession,
    messages: list[dict[str, str]],
    chat_url: str,
    model: str,
    stream: bool = True,
    min_tokens: int | None = None,
    max_tokens: int | None = None,
    timeout_sec: int = 120,
    conversation_id: str | None = None,
) -> ServerResponse:
    payload = {
        "model": model,
        "messages": messages,
        "seed": 0,
        "temperature": 0.0,
    }

    if conversation_id is not None:
        payload["conversation_id"] = conversation_id

    if stream:
        payload["stream"] = True
        payload["stream_options"] = {"include_usage": False}

    if min_tokens is not None:
        payload["min_tokens"] = min_tokens

    if max_tokens is not None:
        payload["max_tokens"] = max_tokens

    headers = {"Content-Type": "application/json"}

    # Calculate the timeout for the request
    # ... omitted for brevity ...
        tpot_ms=nanosec_to_millisec(tpot),
        latency_ms=nanosec_to_millisec(latency),
        start_time_ms=nanosec_to_millisec(start_time),
        first_chunk=first_chunk,
        content=generated_text,
        num_chunks=num_chunks,
    )
    return sr
```
**EN:** `send_request` implements backend request helpers. It mainly works with `session`, `messages`, `chat_url`, `model`, `stream`, ... and relies on `int`, `logger.info`, `aiohttp.ClientTimeout`, `time.perf_counter_ns`, `session.post`, `HTTPStatus` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `send_request` 负责实现后端请求辅助逻辑。 它主要处理 `session`, `messages`, `chat_url`, `model`, `stream`, ...，并结合 `int`, `logger.info`, `aiohttp.ClientTimeout`, `time.perf_counter_ns`, `session.post`, `HTTPStatus` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `get_short_string` (lines 340-345)
```python
def get_short_string(input: str) -> str:
    n = 20
    if len(input) < 400:
        return input

    return f"{input[:n]}...{input[-n:]}"
```
**EN:** `get_short_string` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `input` and relies on `len` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_short_string` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `input`，并结合 `len` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_token_count` (lines 348-349)
```python
def get_token_count(tokenizer: AutoTokenizer, text: str) -> int:
    return len(tokenizer(text, add_special_tokens=False).input_ids)
```
**EN:** `get_token_count` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `tokenizer`, `text` and relies on `len`, `tokenizer` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_token_count` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `tokenizer`, `text`，并结合 `len`, `tokenizer` 以及 结果返回 来完成这一段基准测试流程。

### Function `get_messages_token_count` (lines 352-359)
```python
def get_messages_token_count(
    tokenizer: AutoTokenizer, messages: list[dict[str, str]]
) -> int:
    token_count = 0
    for m in messages:
        token_count += get_token_count(tokenizer, m["content"])

    return token_count
```
**EN:** `get_messages_token_count` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `tokenizer`, `messages` and relies on `get_token_count` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `get_messages_token_count` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `tokenizer`, `messages`，并结合 `get_token_count` 以及 循环迭代 来完成这一段基准测试流程。

### Async function `send_turn` (lines 362-523)
```python
async def send_turn(
    session: aiohttp.ClientSession,
    client_id: int,
    conv_id: str,
    conversation_messages: MessagesList,
    messages_to_use: int,
    tokenizer: AutoTokenizer,
    req_args: RequestArgs,
    verbose: bool,
    verify_output: bool,
) -> RequestStats | None:
    assert messages_to_use > 0
    assert messages_to_use <= len(conversation_messages)

    messages = conversation_messages[:messages_to_use]

    # Index of the next message (the role should be "user")
    index = messages_to_use - 1

    # Verify that the message has only two keys, "role" and "content"
    assert len(messages[index].keys()) == 2
    assert "role" in messages[index] and "content" in messages[index]
    assert messages[index]["role"] == "user", (
        f"Failed on conversation ID {conv_id}, message role should be user"
    )

    if verbose:
        print(
            f"{Color.CYAN}Messages (conversation ID {conv_id},"
            f" {len(messages)} turns):{Color.RESET}",
            messages,
        )

    # None means that there is no upper/lower limit for the output token count
    # ... omitted for brevity ...
        # Update the answer
        conversation_messages[answer_index]["content"] = output_content
    else:
        # A user prompt that has no answer, add the answer as a new message
        new_answer = {"role": "assistant", "content": output_content}
        conversation_messages.append(new_answer)

    return rs
```
**EN:** `send_turn` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `session`, `client_id`, `conv_id`, `conversation_messages`, `messages_to_use`, ... and relies on `len`, `messages.keys`, `print`, `get_token_count`, `max`, `send_request` plus branching to move data through this part of the benchmark pipeline.
**CN:** `send_turn` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `session`, `client_id`, `conv_id`, `conversation_messages`, `messages_to_use`, ...，并结合 `len`, `messages.keys`, `print`, `get_token_count`, `max`, `send_request` 以及 条件分支 来完成这一段基准测试流程。

### Async function `poisson_sleep` (lines 526-533)
```python
async def poisson_sleep(request_rate: float, verbose: bool = False) -> None:
    # Generate a random time interval from the Poisson distribution
    assert request_rate > 0

    interval = np.random.exponential(1.0 / request_rate)
    if verbose:
        logger.info(f"Sleeping for {interval:.3f} seconds...")
    await asyncio.sleep(interval)
```
**EN:** `poisson_sleep` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `request_rate`, `verbose` and relies on `np.random.exponential`, `logger.info`, `asyncio.sleep` plus branching to move data through this part of the benchmark pipeline.
**CN:** `poisson_sleep` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `request_rate`, `verbose`，并结合 `np.random.exponential`, `logger.info`, `asyncio.sleep` 以及 条件分支 来完成这一段基准测试流程。

### Async function `exponential_backoff_sleep` (lines 536-552)
```python
async def exponential_backoff_sleep(
    attempt_cnt: int,
    base_rate: float = 1.0,
    backoff_factor: float = 2.0,
    jitter_fraction: float = 0.10,
    verbose: bool = False,
) -> None:
    # Sleep with exponential backoff and jitter after a failed request.
    backoff_delay = base_rate * (backoff_factor**attempt_cnt)
    jittered_delay = backoff_delay * (
        1 + np.random.uniform(-jitter_fraction, jitter_fraction)
    )

    if verbose:
        logger.info(f"Backoff for {jittered_delay:.3f} seconds...")

    await asyncio.sleep(jittered_delay)
```
**EN:** `exponential_backoff_sleep` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `attempt_cnt`, `base_rate`, `backoff_factor`, `jitter_fraction`, `verbose` and relies on `np.random.uniform`, `logger.info`, `asyncio.sleep` plus branching to move data through this part of the benchmark pipeline.
**CN:** `exponential_backoff_sleep` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `attempt_cnt`, `base_rate`, `backoff_factor`, `jitter_fraction`, `verbose`，并结合 `np.random.uniform`, `logger.info`, `asyncio.sleep` 以及 条件分支 来完成这一段基准测试流程。

### Async function `client_main` (lines 555-776)
```python
async def client_main(
    args: ClientArgs,
    req_args: RequestArgs,
    client_id: int,
    tokenizer: AutoTokenizer,
    stop_event: mp.Event,  # type: ignore
    task_queue: mp.Queue,
    result_queue: mp.Queue,
    conv_queue: mp.Queue,
) -> None:
    logger.info(
        f"{Color.CYAN}Started client {client_id}: max_num_requests={args.max_num_requests}, max_active_conversations={args.max_active_conversations}{Color.RESET}"  # noqa: E501
    )

    # Set unique seed per client (each client runs in its own process)
    # Add 1 to ensure no client uses the same seed as the main process
    client_seed = args.seed + client_id + 1
    random.seed(client_seed)
    np.random.seed(client_seed)

    # Active conversations
    active_convs: ConversationsMap = {}
    conv_id_queue: deque = deque(maxlen=args.max_active_conversations)

    # Keep track of how many messages have been used for each conversation
    turns_count: Counter = Counter()
    num_successes = 0
    num_failures = 0

    # Track the timestamp (time.perf_counter())
    # of the last turn per conversation (only for debug)
    time_of_last_turn: dict[ConvId, float] = {}

    # Flag that indicates that there are no new tasks (conversations) for the client
    # ... omitted for brevity ...

    # Send indication that the client is done
    conv_queue.put((TERM_SIGNAL, TERM_SIGNAL))

    logger.info(
        f"{Color.CYAN}Client {client_id} is done "
        f"({num_successes=}, {num_failures=}){Color.RESET}"
    )
```
**EN:** `client_main` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `args`, `req_args`, `client_id`, `tokenizer`, `stop_event`, ... and relies on `logger.info`, `random.seed`, `np.random.seed`, `deque`, `Counter`, `aiohttp.ClientSession` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `client_main` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `args`, `req_args`, `client_id`, `tokenizer`, `stop_event`, ...，并结合 `logger.info`, `random.seed`, `np.random.seed`, `deque`, `Counter`, `aiohttp.ClientSession` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `worker_function` (lines 779-800)
```python
def worker_function(
    client_id: int,
    tokenizer: AutoTokenizer,
    client_args: ClientArgs,
    req_args: RequestArgs,
    stop_event: mp.Event,  # type: ignore
    task_queue: mp.Queue,
    result_queue: mp.Queue,
    conv_queue: mp.Queue,
) -> None:
    asyncio.run(
        client_main(
            client_args,
            req_args,
            client_id,
            tokenizer,
            stop_event,
            task_queue,
            result_queue,
            conv_queue,
        )
    )
```
**EN:** `worker_function` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `client_id`, `tokenizer`, `client_args`, `req_args`, `stop_event`, ... and relies on `asyncio.run`, `client_main` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `worker_function` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `client_id`, `tokenizer`, `client_args`, `req_args`, `stop_event`, ...，并结合 `asyncio.run`, `client_main` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `get_client_config` (lines 803-885)
```python
def get_client_config(
    args: argparse.Namespace, input_conv: ConversationsMap
) -> tuple[ClientArgs, RequestArgs]:
    if args.num_clients < 1:
        raise ValueError("Number of clients must be a positive number")

    if len(input_conv) < args.num_clients:
        raise ValueError(
            "Number of conversations must be equal or larger than the number of clients"
        )

    max_req_per_client: int | None = None
    if args.max_num_requests is not None:
        # Max number of requests per client
        req_per_client = args.max_num_requests // args.num_clients
        if req_per_client < 1:
            raise ValueError("Number of requests should be at least one per client")
        max_req_per_client = req_per_client

    max_active_conversations = args.max_active_conversations
    if max_active_conversations is None:
        # Each client will have only one active conversation at a time
        max_active_conversations = args.num_clients

    if max_active_conversations > len(input_conv):
        raise ValueError(
            f"Max active conversations {max_active_conversations} "
            "must be equal or less than the total number of conversations"
        )

    # Max number of active conversations per client
    max_active_conv_per_client = max_active_conversations // args.num_clients
    if max_active_conv_per_client < 1:
        raise ValueError(
    # ... omitted for brevity ...
        model=model_name,
        stream=not args.no_stream,
        limit_min_tokens=args.limit_min_tokens,
        limit_max_tokens=args.limit_max_tokens,
        timeout_sec=args.request_timeout_sec,
    )

    return client_args, req_args
```
**EN:** `get_client_config` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `args`, `input_conv` and relies on `ValueError`, `len`, `ClientArgs`, `RequestArgs` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_client_config` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `args`, `input_conv`，并结合 `ValueError`, `len`, `ClientArgs`, `RequestArgs` 以及 条件分支 来完成这一段基准测试流程。

### Async function `main_mp` (lines 888-1067)
```python
async def main_mp(
    client_args: ClientArgs,
    req_args: RequestArgs,
    bench_args: BenchmarkArgs,
    tokenizer: AutoTokenizer,
    input_conv: ConversationsMap,
) -> tuple[ConversationsMap, list[RequestStats]]:
    # An event that will trigger graceful termination of all the clients
    stop_event = mp.Event()

    # Queue for input conversations (from the input file/dataset)
    task_queue: mp.Queue = mp.Queue()

    # Queue for client measurements (TTFT, TPOT, etc. for each request)
    result_queue: mp.Queue = mp.Queue()

    # Queue for output conversations (with the LLM answers, sent by the server)
    conv_queue: mp.Queue = mp.Queue()
    output_conv: ConversationsMap = {}
    client_metrics: list[RequestStats] = []

    # Start all clients
    start_time = time.perf_counter_ns()
    logger.info(f"{Color.GREEN}Starting {bench_args.num_clients} clients{Color.RESET}")

    clients = []
    for client_id in range(bench_args.num_clients):
        client = mp.Process(
            name=f"client_{client_id}",
            target=worker_function,
            args=(
                client_id,
                tokenizer,
                client_args,
    # ... omitted for brevity ...

    result_queue.close()
    result_queue.join_thread()

    conv_queue.close()
    conv_queue.join_thread()

    return output_conv, client_metrics
```
**EN:** `main_mp` serves as the main entry point for this script. It mainly works with `client_args`, `req_args`, `bench_args`, `tokenizer`, `input_conv` and relies on `mp.Event`, `mp.Queue`, `time.perf_counter_ns`, `logger.info`, `range`, `mp.Process` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main_mp` 充当该脚本的主入口。 它主要处理 `client_args`, `req_args`, `bench_args`, `tokenizer`, `input_conv`，并结合 `mp.Event`, `mp.Queue`, `time.perf_counter_ns`, `logger.info`, `range`, `mp.Process` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `get_filename_with_timestamp` (lines 1070-1074)
```python
def get_filename_with_timestamp(label: str, extension: str) -> str:
    time_now = datetime.now()
    timestamp = time_now.strftime("%d-%m-%Y_%H-%M-%S")
    filename = f"{label}__{timestamp}.{extension}"
    return filename
```
**EN:** `get_filename_with_timestamp` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `label`, `extension` and relies on `datetime.now`, `time_now.strftime` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_filename_with_timestamp` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `label`, `extension`，并结合 `datetime.now`, `time_now.strftime` 以及 结果返回 来完成这一段基准测试流程。

### Function `process_statistics` (lines 1077-1245)
```python
def process_statistics(
    client_metrics: list[RequestStats],
    warmup_percentages: list[float],
    test_params: dict,
    verbose: bool,
    gen_conv_args: GenConvArgs | None = None,
    excel_output: bool = False,
    warmup_runtime_sec: float | None = None,
) -> None:
    if len(client_metrics) == 0:
        logger.info("No samples to process")
        return

    logger.info(f"Processing {len(client_metrics)} samples...")

    raw_data = pd.DataFrame(client_metrics)

    if verbose:
        # Calculate the time between user turns in each conversation (in a new column)
        raw_data = raw_data.sort_values(by=["conversation_id", "start_time_ms"])
        raw_data["time_between_user_turns_sec"] = raw_data.groupby("conversation_id")[
            "start_time_ms"
        ].diff()

        # Convert milliseconds to seconds
        raw_data["time_between_user_turns_sec"] = (
            raw_data["time_between_user_turns_sec"] / 1000.0
        )

    # Final raw data should be sorted by time
    raw_data = raw_data.sort_values(by=["start_time_ms"])
    raw_data["end_time_ms"] = raw_data["start_time_ms"] + raw_data["latency_ms"]

    percentiles = [0.25, 0.5, 0.75, 0.9]
    # ... omitted for brevity ...
                )
                startrow += len(df_stats) + 3

            raw_data.to_excel(writer, sheet_name="Raw data", index=False, startrow=0)

        logger.info(
            f"{Color.GREEN}Client metrics exported to file: {filename}{Color.RESET}"
        )
```
**EN:** `process_statistics` implements a helper used by `benchmark_serving_multi_turn.py`. It mainly works with `client_metrics`, `warmup_percentages`, `test_params`, `verbose`, `gen_conv_args`, ... and relies on `len`, `logger.info`, `pd.DataFrame`, `raw_data.sort_values`, `raw_data.groupby.diff`, `raw_data.groupby` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `process_statistics` 负责实现 `benchmark_serving_multi_turn.py` 使用的辅助逻辑。 它主要处理 `client_metrics`, `warmup_percentages`, `test_params`, `verbose`, `gen_conv_args`, ...，并结合 `len`, `logger.info`, `pd.DataFrame`, `raw_data.sort_values`, `raw_data.groupby.diff`, `raw_data.groupby` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Async function `get_server_info` (lines 1248-1273)
```python
async def get_server_info(url: str) -> None:
    logger.info(f"{Color.BLUE}Collecting information from server: {url}{Color.RESET}")
    async with aiohttp.ClientSession() as session:
        # Get server version (not mandatory, "version" endpoint may not exist)
        url_version = f"{url}/version"
        async with session.get(url_version) as response:
            if HTTPStatus(response.status) == HTTPStatus.OK:
                text = await response.text()
                logger.info(f"{Color.BLUE}Server version: {text}{Color.RESET}")

        # Get available models
        url_models = f"{url}/v1/models"
        async with session.get(url_models) as response:
            if HTTPStatus(response.status) == HTTPStatus.OK:
                text = await response.text()
                logger.info(f"{Color.BLUE}Models:{Color.RESET}")
                models_data = json.loads(text)
                models_list = models_data["data"]
                for model in models_list:
                    model_id = model["id"]
                    max_model_len = model.get("max_model_len", "N/A")
                    logger.info(
                        f"{Color.BLUE}\t{model_id=}, {max_model_len=}{Color.RESET}"
                    )
            else:
                logger.info(f"{Color.RED}Failed to get models{Color.RESET}")
```
**EN:** `get_server_info` coordinates or measures serving-side benchmarks. It mainly works with `url` and relies on `logger.info`, `aiohttp.ClientSession`, `session.get`, `HTTPStatus`, `response.text`, `json.loads` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `get_server_info` 负责协调或测量服务侧基准。 它主要处理 `url`，并结合 `logger.info`, `aiohttp.ClientSession`, `session.get`, `HTTPStatus`, `response.text`, `json.loads` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Async function `main` (lines 1276-1694)
```python
async def main() -> None:
    parser = argparse.ArgumentParser(
        prog="Benchmark serving with multi-turn conversations",
        description="Benchmark online inference using REST API",
    )
    parser.add_argument("--version", action="version", version="%(prog)s 1.0")

    parser.add_argument(
        "-i",
        "--input-file",
        type=str,
        required=True,
        help="Input JSON file with ShareGPT conversations or "
        "configuration file for generation of synthetic conversations",
    )
    parser.add_argument(
        "-o",
        "--output-file",
        type=str,
        default=None,
        help="Output JSON file containing conversations with updated assistant answers",
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=0,
        help="Seed for random number generators (default: 0)",
    )

    parser.add_argument(
        "-m", "--model", type=str, required=True, help="Path of the LLM model"
    )
    parser.add_argument(
    # ... omitted for brevity ...
        # Write a JSON file with the updated conversations
        # The "assistant" content will contain the answers from the tested LLM
        output_data: ShareGptConversations = conversations_dict_to_list(client_convs)
        logger.info(
            f"{Color.GREEN}Writing conversations file: {args.output_file}{Color.RESET}"
        )
        with open(args.output_file, "w") as f:
            json.dump(output_data, f, indent=4)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `int`, `os.environ.get`, `list`, `parser.parse_args` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `int`, `os.environ.get`, `list`, `parser.parse_args` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 1697-1698)
```python
if __name__ == "__main__":
    asyncio.run(main())
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `asyncio.run`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `asyncio.run`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Distributed execution: the code coordinates work across processes, devices, or Ray workers.
- **CN:** 分布式执行：代码会在多个进程、设备或 Ray worker 之间协调工作。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `asyncio`, `json`, `logging`, `multiprocessing`, `os`.
- **CN:** 标准库依赖：`argparse`, `asyncio`, `json`, `logging`, `multiprocessing`, `os`。
- **EN:** Third-party packages: `aiohttp`, `numpy`, `pandas`, `transformers`.
- **CN:** 第三方依赖：`aiohttp`, `numpy`, `pandas`, `transformers`。
- **EN:** Internal modules: `bench_dataset`, `bench_utils`.
- **CN:** 内部模块：`bench_dataset`, `bench_utils`。
