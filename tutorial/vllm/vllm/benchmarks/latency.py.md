# latency.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/latency.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Benchmark the latency of processing a single batch of requests. / 该文件的核心目的为：Benchmark the latency of processing a single batch of requests.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Benchmark the latency of processing a single batch of requests."""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 5-17)
```python
import argparse

import json

import os

import time

from typing import Any

import numpy as np

from tqdm import tqdm

from vllm.benchmarks.lib.utils import convert_to_pytorch_benchmark_format, write_to_json

from vllm.engine.arg_utils import EngineArgs

from vllm.inputs import PromptType

from vllm.sampling_params import BeamSearchParams
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `save_to_pytorch_benchmark_format` (lines 20-30)
```python
def save_to_pytorch_benchmark_format(
    args: argparse.Namespace, results: dict[str, Any]
) -> None:
    pt_records = convert_to_pytorch_benchmark_format(
        args=args,
        metrics={"latency": results["latencies"]},
        extra_info={k: results[k] for k in ["avg_latency", "percentiles"]},
    )
    if pt_records:
        pt_file = f"{os.path.splitext(args.output_json)[0]}.pytorch.json"
        write_to_json(pt_file, pt_records)
```
**EN:** Function `save_to_pytorch_benchmark_format` serializes data and writes it to a target representation. Key calls such as `convert_to_pytorch_benchmark_format`, `os.path.splitext`, `write_to_json` show the concrete execution path.
**CN:** Function `save_to_pytorch_benchmark_format` 负责序列化数据并写入目标表示。 像 `convert_to_pytorch_benchmark_format`, `os.path.splitext`, `write_to_json` 这样的关键调用展示了该代码块的具体执行路径。

### Function `add_cli_args` (lines 33-76)
```python
def add_cli_args(parser: argparse.ArgumentParser):
    parser.add_argument("--input-len", type=int, default=32)
    parser.add_argument("--output-len", type=int, default=128)
    parser.add_argument("--batch-size", type=int, default=8)
    parser.add_argument(
        "--n",
        type=int,
        default=1,
        help="Number of generated sequences per prompt.",
    )
    parser.add_argument("--use-beam-search", action="store_true")
    parser.add_argument(
        "--num-iters-warmup",
        type=int,
        default=10,
        help="Number of iterations to run for warmup.",
    )
    parser.add_argument(
        "--num-iters", type=int, default=30, help="Number of iterations to run."
    )
    parser.add_argument(
        "--profile",
        action="store_true",
        help="profile the generation process of a single batch",
    )
    # ... omitted for brevity ...
    # numbers. We need to disable prefix caching by default.
    parser.set_defaults(enable_prefix_caching=False)
```
**EN:** Function `add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument`, `EngineArgs.add_cli_args`, `parser.set_defaults` show the concrete execution path.
**CN:** Function `add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument`, `EngineArgs.add_cli_args`, `parser.set_defaults` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 79-171)
```python
def main(args: argparse.Namespace):
    engine_args = EngineArgs.from_cli_args(args)

    # Lazy import to avoid importing LLM when the bench command is not selected.
    from vllm import LLM, SamplingParams

    # NOTE(woosuk): If the request cannot be processed in a single batch,
    # the engine will automatically process the request in multiple batches.
    llm = LLM.from_engine_args(engine_args)
    assert llm.llm_engine.model_config.max_model_len >= (
        args.input_len + args.output_len
    ), (
        "Please ensure that max_model_len is greater than"
        " the sum of input_len and output_len."
    )

    sampling_params = SamplingParams(
        n=args.n,
        temperature=1.0,
        top_p=1.0,
        ignore_eos=True,
        max_tokens=args.output_len,
        detokenize=not args.disable_detokenize,
    )
    dummy_prompt_token_ids = np.random.randint(
    # ... omitted for brevity ...
            json.dump(results, f, indent=4)
        save_to_pytorch_benchmark_format(args, results)
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `EngineArgs.from_cli_args`, `LLM.from_engine_args`, `SamplingParams`, `np.random.randint`, `dummy_prompt_token_ids.tolist` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `EngineArgs.from_cli_args`, `LLM.from_engine_args`, `SamplingParams`, `np.random.randint`, `dummy_prompt_token_ids.tolist` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import json`, `import os`, `import time`, `from typing import Any`
- **Third-party / 第三方**: `import numpy as np`, `from tqdm import tqdm`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.lib.utils import convert_to_pytorch_benchmark_format, write_to_json`, `from vllm.engine.arg_utils import EngineArgs`, `from vllm.inputs import PromptType`, `from vllm.sampling_params import BeamSearchParams`
