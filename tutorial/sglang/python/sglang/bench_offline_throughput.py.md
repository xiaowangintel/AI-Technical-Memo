# bench_offline_throughput.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/bench_offline_throughput.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Benchmark the throughput in the offline mode. / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Module-level supporting statements
```python
"""
Benchmark the throughput in the offline mode.
It accepts server arguments (the same as launch_server.py) and benchmark arguments (the same as bench_serving.py).

# Usage
## Sharegpt dataset with default args
python -m sglang.bench_offline_throughput --model-path meta-llama/Meta-Llama-3.1-8B-Instruct --num-prompts 10

## Random dataset with default args
python -m sglang.bench_offline_throughput --model-path meta-llama/Meta-Llama-3.1-8B-Instruct --dataset-name random --random-input 1024 --random-output 1024
"""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
import argparse
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python
import asyncio
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python
import dataclasses
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Module-level supporting statements
```python
import inspect
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Module-level supporting statements
```python
import json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-18: Module-level supporting statements
```python
import logging
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 19-19: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 20-20: Module-level supporting statements
```python
import random
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-21: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 22-22: Module-level supporting statements
```python
from typing import Dict, List, Optional
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-23: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 24-24: Module-level supporting statements
```python
import numpy as np
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 25-25: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 26-26: Module-level supporting statements
```python
from sglang.benchmark.datasets import DatasetRow, get_dataset
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 27-27: Module-level supporting statements
```python
from sglang.benchmark.datasets.random import sample_random_requests
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 28-28: Module-level supporting statements
```python
from sglang.benchmark.utils import get_tokenizer, set_ulimit
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 29-29: Module-level supporting statements
```python
from sglang.lang.backend.runtime_endpoint import Runtime
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 30-30: Module-level supporting statements
```python
from sglang.srt.entrypoints.engine import Engine
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 31-31: Module-level supporting statements
```python
from sglang.srt.server_args import ServerArgs
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 32-34: Module-level supporting statements
```python


@dataclasses.dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 35-35: BenchArgs class declaration
```python
class BenchArgs:
```
**EN:** This block declares the `BenchArgs` class, which exists to benchmark a specific workload. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `BenchArgs` 类，其职责是对特定负载进行基准测试。它定义了本文件其余部分使用的结构与成员布局。

### Lines 36-36: Class-level supporting statements
```python
    backend: str = "engine"
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 37-37: Class-level supporting statements
```python
    result_filename: str = ""
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 38-38: Class-level supporting statements
```python
    dataset_name: str = "sharegpt"
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 39-39: Class-level supporting statements
```python
    dataset_path: str = ""
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 40-40: Class-level supporting statements
```python
    num_prompts: int = 1000
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 41-41: Class-level supporting statements
```python
    sharegpt_output_len: Optional[int] = None
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 42-42: Class-level supporting statements
```python
    sharegpt_context_len: Optional[int] = None
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 43-43: Class-level supporting statements
```python
    random_input_len: int = 1024
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 44-44: Class-level supporting statements
```python
    random_output_len: int = 1024
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 45-45: Class-level supporting statements
```python
    random_range_ratio: float = 0.0
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 46-46: Class-level supporting statements
```python
    gsp_num_groups: int = 64
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 47-47: Class-level supporting statements
```python
    gsp_prompts_per_group: int = 16
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 48-48: Class-level supporting statements
```python
    gsp_system_prompt_len: int = 2048
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 49-49: Class-level supporting statements
```python
    gsp_question_len: int = 128
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 50-50: Class-level supporting statements
```python
    gsp_output_len: int = 256
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 51-51: Class-level supporting statements
```python
    seed: int = 1
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 52-52: Class-level supporting statements
```python
    disable_ignore_eos: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 53-53: Class-level supporting statements
```python
    extra_request_body: Optional[str] = None
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 54-54: Class-level supporting statements
```python
    apply_chat_template: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 55-55: Class-level supporting statements
```python
    profile: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-56: Class-level supporting statements
```python
    skip_warmup: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 57-57: Class-level supporting statements
```python
    do_not_exit: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 58-58: Class-level supporting statements
```python
    prompt_suffix: str = ""
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 59-59: Class-level supporting statements
```python
    return_logprob: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 60-60: Class-level supporting statements
```python
    logprob_start_len: int = -1
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 61-62: Class-level supporting statements
```python

    @staticmethod
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 63-122: BenchArgs.add_cli_args method (part 1/3)
```python
    def add_cli_args(parser: argparse.ArgumentParser):
        parser.add_argument("--backend", type=str, default=BenchArgs.backend)
        parser.add_argument(
            "--result-filename", type=str, default=BenchArgs.result_filename
        )
        parser.add_argument(
            "--dataset-name",
            type=str,
            default="sharegpt",
            choices=["sharegpt", "random", "generated-shared-prefix"],
            help="Name of the dataset to benchmark on.",
        )
        parser.add_argument(
            "--dataset-path", type=str, default="", help="Path to the dataset."
        )
        parser.add_argument(
            "--num-prompts",
            type=int,
            default=BenchArgs.num_prompts,
            help="Number of prompts to process. Default is 1000.",
        )
        parser.add_argument(
            "--sharegpt-output-len",
            type=int,
            default=BenchArgs.sharegpt_output_len,
            help="Output length for each request. Overrides the output length from the ShareGPT dataset.",
        )
        parser.add_argument(
            "--sharegpt-context-len",
            type=int,
            default=BenchArgs.sharegpt_context_len,
            help="The context length of the model for the ShareGPT dataset. Requests longer than the context length will be dropped.",
        )
        parser.add_argument(
            "--random-input-len",
            type=int,
            default=BenchArgs.random_input_len,
            help="Number of input tokens per request, used only for random dataset.",
        )
        parser.add_argument(
            "--random-output-len",
            type=int,
            default=BenchArgs.random_output_len,
            help="Number of output tokens per request, used only for random dataset.",
        )
        parser.add_argument(
            "--random-range-ratio",
            type=float,
            default=BenchArgs.random_range_ratio,
            help="Range of sampled ratio of input/output length, "
            "used only for random dataset.",
        )
        parser.add_argument(
            "--gsp-num-groups",
            type=int,
            default=BenchArgs.gsp_num_groups,
            help="Number of groups with shared prefix, used"
            "only for generate-shared-prefix",
        )
        parser.add_argument(
```
**EN:** This block uses `BenchArgs.add_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `BenchArgs.add_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 123-182: BenchArgs.add_cli_args method (part 2/3)
```python
            "--gsp-prompts-per-group",
            type=int,
            default=BenchArgs.gsp_prompts_per_group,
            help="Number of prompts per group of shared prefix, used"
            "only for generate-shared-prefix",
        )
        parser.add_argument(
            "--gsp-system-prompt-len",
            type=int,
            default=BenchArgs.gsp_system_prompt_len,
            help="System prompt length, used" "only for generate-shared-prefix",
        )
        parser.add_argument(
            "--gsp-question-len",
            type=int,
            default=BenchArgs.gsp_question_len,
            help="Question length, used" "only for generate-shared-prefix",
        )
        parser.add_argument(
            "--gsp-output-len",
            type=int,
            default=BenchArgs.gsp_output_len,
            help="Target length in tokens for outputs in generated-shared-prefix dataset",
        )
        parser.add_argument("--seed", type=int, default=1, help="The random seed.")
        parser.add_argument(
            "--disable-ignore-eos",
            action="store_true",
            help="Disable ignore EOS token",
        )
        parser.add_argument(
            "--extra-request-body",
            metavar='{"key1": "value1", "key2": "value2"}',
            type=str,
            default=BenchArgs.extra_request_body,
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
            "--skip-warmup",
            action="store_true",
            help="Skip the warmup batches.",
        )
        parser.add_argument(
            "--do-not-exit",
            action="store_true",
            help="Do not exit the program. This is useful for nsys profile with --duration and --delay.",
        )
        parser.add_argument(
```
**EN:** This block uses `BenchArgs.add_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the middle-stage logic and data flow of the routine.
**CN:** 该代码块通过 `BenchArgs.add_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流。

### Lines 183-198: BenchArgs.add_cli_args method (part 3/3)
```python
            "--prompt-suffix",
            type=str,
            default="",
            help="Suffix applied to the end of all user prompts, followed by assistant prompt suffix.",
        )
        parser.add_argument(
            "--return-logprob",
            action="store_true",
            help="Enable returning log probabilities.",
        )
        parser.add_argument(
            "--logprob-start-len",
            type=int,
            default=-1,
            help="Start length for logprob. -1 means only return logprobs for output tokens (default). 0 means return logprobs for all tokens including input.",
        )
```
**EN:** This block uses `BenchArgs.add_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `BenchArgs.add_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 199-200: Class-level supporting statements
```python

    @classmethod
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 201-203: BenchArgs.from_cli_args method
```python
    def from_cli_args(cls, args: argparse.Namespace):
        attrs = [attr.name for attr in dataclasses.fields(cls)]
        return cls(**{attr: getattr(args, attr) for attr in attrs})
```
**EN:** This block uses `BenchArgs.from_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BenchArgs.from_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 204-205: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 206-265: throughput test once function (part 1/2)
```python
def throughput_test_once(
    backend_name: str,
    backend,
    reqs: List[DatasetRow],
    ignore_eos: bool,
    extra_request_body: Dict,
    profile: bool,
    return_logprob: bool = False,
    logprob_start_len: int = -1,
):
    measurement_results = {
        "backend": backend_name,
        "successful_requests": len(reqs),
        "total_latency": -1,
        "total_input_tokens": sum(r.prompt_len for r in reqs),
        "total_output_tokens": -1,
        "request_throughput": -1,
        "input_throughput": -1,
        "output_throughput": -1,
        "total_throughput": -1,
    }

    prompt = [r.prompt for r in reqs]
    sampling_params = [
        {
            "temperature": 0,
            "max_new_tokens": r.output_len,
            "ignore_eos": ignore_eos,
            **extra_request_body,
        }
        for r in reqs
    ]

    if profile:
        assert (
            "SGLANG_TORCH_PROFILER_DIR" in os.environ
        ), "Please set SGLANG_TORCH_PROFILER_DIR."
        os.makedirs(os.environ["SGLANG_TORCH_PROFILER_DIR"], exist_ok=True)
        backend.start_profile()

    st = time.perf_counter()
    gen_out = backend.generate(
        prompt=prompt,
        sampling_params=sampling_params,
        return_logprob=return_logprob,
        logprob_start_len=logprob_start_len,
    )
    latency = time.perf_counter() - st

    if profile:
        dir = os.getenv("SGLANG_TORCH_PROFILER_DIR")
        known_files = set(os.listdir(dir))
        backend.stop_profile()
        monitor_trace_file(known_files, dir)

    if backend_name == "runtime":
        gen_out = json.loads(gen_out)

    server_info = backend.get_server_info()
```
**EN:** This block uses `throughput_test_once` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `throughput_test_once` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 266-291: throughput test once function (part 2/2)
```python
    measurement_results["total_latency"] = latency
    measurement_results["total_output_tokens"] = sum(
        o["meta_info"]["completion_tokens"] for o in gen_out
    )
    measurement_results["request_throughput"] = (
        measurement_results["successful_requests"] / latency
    )
    measurement_results["input_throughput"] = (
        measurement_results["total_input_tokens"] / latency
    )
    measurement_results["output_throughput"] = (
        measurement_results["total_output_tokens"] / latency
    )
    measurement_results["total_throughput"] = (
        measurement_results["total_input_tokens"]
        + measurement_results["total_output_tokens"]
    ) / latency

    if inspect.isawaitable(server_info):
        server_info = asyncio.run(server_info)

    measurement_results["last_gen_throughput"] = server_info["internal_states"][0][
        "last_gen_throughput"
    ]

    return measurement_results
```
**EN:** This block uses `throughput_test_once` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `throughput_test_once` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 292-293: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 294-323: monitor trace file function
```python
def monitor_trace_file(known_files, directory, interval=1):
    print(f"Monitoring {directory} for new trace files...")

    while True:
        flag = False
        time.sleep(interval)
        current_files = set(os.listdir(directory))

        new_files = current_files - known_files
        for new_file in new_files:
            new_file_path = os.path.join(directory, new_file)
            print(f"New file detected: {new_file}")

            previous_size = 0
            while True:
                try:
                    current_size = os.path.getsize(new_file_path)
                except FileNotFoundError:
                    print(f"File {new_file} is no longer accessible.")
                    break

                if current_size > previous_size:
                    previous_size = current_size
                else:
                    flag = True
                    break

                time.sleep(interval)
        if flag:
            break
```
**EN:** This block uses `monitor_trace_file` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `monitor_trace_file` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 324-325: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 326-385: create ray engine backend function (part 1/2)
```python
def _create_ray_engine_backend(server_args: ServerArgs):
    """Create a RayEngine inside a Ray actor on a placement group.

    RayEngine requires a placement group, so we launch it inside a Ray actor
    and return a lightweight proxy that forwards calls via ray.get().
    """
    import ray
    from ray.runtime_env import RuntimeEnv
    from ray.util.placement_group import placement_group
    from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy

    env_vars = {"RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES": "1"}
    if os.environ.get("HF_TOKEN"):
        env_vars["HF_TOKEN"] = os.environ["HF_TOKEN"]
    if not ray.is_initialized():
        ray.init(runtime_env=RuntimeEnv(env_vars=env_vars))

    total_gpus = server_args.tp_size * server_args.pp_size
    pg = placement_group([{"CPU": 1, "GPU": total_gpus}], strategy="STRICT_PACK")
    ray.get(pg.ready())

    @ray.remote
    class _EngineActor:
        def __init__(self, **kwargs):
            from sglang.srt.ray.engine import RayEngine

            self.engine = RayEngine(**kwargs)

        def call(self, method, **kwargs):
            return getattr(self.engine, method)(**kwargs)

    actor = _EngineActor.options(
        num_cpus=1,
        num_gpus=0,
        scheduling_strategy=PlacementGroupSchedulingStrategy(
            placement_group=pg,
            placement_group_bundle_index=0,
        ),
    ).remote(**dataclasses.asdict(server_args))

    class _Proxy:
        """Forwards method calls to the remote RayEngine actor."""

        def generate(self, **kwargs):
            return ray.get(actor.call.remote("generate", **kwargs))

        def get_server_info(self, **kwargs):
            return ray.get(actor.call.remote("get_server_info", **kwargs))

        def start_profile(self, **kwargs):
            return ray.get(actor.call.remote("start_profile", **kwargs))

        def stop_profile(self, **kwargs):
            return ray.get(actor.call.remote("stop_profile", **kwargs))

        def shutdown(self):
            try:
                ray.get(actor.call.remote("shutdown"), timeout=60)
            except Exception:
                pass
```
**EN:** This block uses `_create_ray_engine_backend` to create runtime objects or artifacts. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `_create_ray_engine_backend` 来创建运行时对象或产物。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 386-391: create ray engine backend function (part 2/2)
```python
            try:
                ray.util.remove_placement_group(pg)
            except Exception:
                pass

    return _Proxy()
```
**EN:** This block uses `_create_ray_engine_backend` to create runtime objects or artifacts. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `_create_ray_engine_backend` 来创建运行时对象或产物。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 392-393: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 394-453: throughput test function (part 1/2)
```python
def throughput_test(
    server_args: ServerArgs,
    bench_args: BenchArgs,
):
    if bench_args.backend == "engine":
        if server_args.use_ray:
            backend = _create_ray_engine_backend(server_args)
        else:
            backend = Engine(**dataclasses.asdict(server_args))
        if not backend:
            raise ValueError("Please provide valid engine arguments")
    elif bench_args.backend == "runtime":
        backend = Runtime(**dataclasses.asdict(server_args))
    else:
        raise ValueError('Please set backend to either "engine" or "runtime"')

    tokenizer_id = server_args.tokenizer_path or server_args.model_path
    tokenizer = get_tokenizer(tokenizer_id)

    # Set global environments
    set_ulimit()
    random.seed(bench_args.seed)
    np.random.seed(bench_args.seed)

    # Parse args
    extra_request_body = {}
    if bench_args.extra_request_body:
        extra_request_body = json.loads(args.extra_request_body)

    # Read dataset
    input_requests = get_dataset(bench_args, tokenizer)

    warmup_requests = sample_random_requests(
        input_len=256,
        output_len=16,
        num_prompts=min(bench_args.num_prompts, 16),
        range_ratio=1.0,
        tokenizer=tokenizer,
        dataset_path=bench_args.dataset_path,
    )

    # Warm up
    if not bench_args.skip_warmup:
        logging.info("\nWarmup...")
        throughput_test_once(
            backend_name=bench_args.backend,
            backend=backend,
            reqs=warmup_requests,
            ignore_eos=not bench_args.disable_ignore_eos,
            extra_request_body=extra_request_body,
            profile=False,
            return_logprob=bench_args.return_logprob,
            logprob_start_len=bench_args.logprob_start_len,
        )
        time.sleep(0.5)

    logging.info("\nBenchmark...")
    result = throughput_test_once(
        backend_name=bench_args.backend,
        backend=backend,
```
**EN:** This block uses `throughput_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `throughput_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 454-504: throughput test function (part 2/2)
```python
        reqs=input_requests,
        ignore_eos=not bench_args.disable_ignore_eos,
        extra_request_body=extra_request_body,
        profile=bench_args.profile,
        return_logprob=bench_args.return_logprob,
        logprob_start_len=bench_args.logprob_start_len,
    )
    backend.shutdown()

    if bench_args.result_filename:
        with open(bench_args.result_filename, "a") as fout:
            fout.write(json.dumps(result) + "\n")

    print(
        "\n{s:{c}^{n}}".format(s=" Offline Throughput Benchmark Result ", n=50, c="=")
    )
    print("{:<40} {:<10}".format("Backend:", result["backend"]))
    print("{:<40} {:<10}".format("Successful requests:", result["successful_requests"]))
    print("{:<40} {:<10.2f}".format("Benchmark duration (s):", result["total_latency"]))
    print("{:<40} {:<10}".format("Total input tokens:", result["total_input_tokens"]))
    print(
        "{:<40} {:<10}".format("Total generated tokens:", result["total_output_tokens"])
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Last generation throughput (tok/s):", result["last_gen_throughput"]
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Request throughput (req/s):", result["request_throughput"]
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Input token throughput (tok/s):", result["input_throughput"]
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Output token throughput (tok/s):", result["output_throughput"]
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Total token throughput (tok/s):", result["total_throughput"]
        )
    )
    print("=" * 50)

    return result
```
**EN:** This block uses `throughput_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `throughput_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 505-506: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 507-543: Module-level supporting statements
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    BenchArgs.add_cli_args(parser)
    args = parser.parse_args()

    # handling ModelScope model downloads
    if os.getenv("SGLANG_USE_MODELSCOPE", "false").lower() in ("true", "1"):
        if os.path.exists(args.model_path):
            print(f"Using local model path: {args.model_path}")
        else:
            try:
                from modelscope import snapshot_download

                print(f"Using ModelScope to download model: {args.model_path}")

                # download the model and replace args.model_path
                args.model_path = snapshot_download(
                    args.model_path,
                )
                print(f"Model downloaded to: {args.model_path}")
            except Exception as e:
                print(f"ModelScope download failed: {str(e)}")
                raise e

    server_args = ServerArgs.from_cli_args(args)
    bench_args = BenchArgs.from_cli_args(args)

    logging.basicConfig(
        level=getattr(logging, server_args.log_level.upper()),
        format="%(message)s",
    )

    throughput_test(server_args, bench_args)

    while bench_args.do_not_exit:
        pass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Performance measurement / 性能测量
- Numerical data processing / 数值数据处理
- Runtime coordination / 运行时协同

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.benchmark.datasets`
- `sglang.benchmark.datasets.random`
- `sglang.benchmark.utils`
- `sglang.lang.backend.runtime_endpoint`
- `sglang.srt.entrypoints.engine`
- `sglang.srt.ray.engine`
- `sglang.srt.server_args`
### External / 外部
- `modelscope`
- `numpy`
- `ray`
- `argparse` (stdlib)
- `asyncio` (stdlib)
- `dataclasses` (stdlib)
- `inspect` (stdlib)
- `json` (stdlib)
- `logging` (stdlib)
- `os` (stdlib)
- `random` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
