# benchmark_ngram_proposer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_ngram_proposer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, ngram proposer benchmarks, centered around `benchmark_propose`, `benchmark_batched_propose`, `invoke_main`. / 实现与基准测试编排、ngram 提议器基准相关的逻辑，核心符号包括 `benchmark_propose`, `benchmark_batched_propose`, `invoke_main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-25)
```python
import gc
import time
from unittest import mock

import numpy as np
from benchmark_utils import TimeCollector
from tabulate import tabulate

from vllm.config import (
    CacheConfig,
    DeviceConfig,
    LoadConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    SpeculativeConfig,
    VllmConfig,
)
from vllm.platforms import current_platform
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.v1.spec_decode.ngram_proposer import NgramProposer
from vllm.v1.worker.gpu_input_batch import InputBatch
from vllm.v1.worker.gpu_model_runner import GPUModelRunner
```
**EN:** This block gathers standard-library helpers such as `gc`, `time`, `unittest`; third-party packages such as `numpy`, `tabulate`; project-local modules such as `benchmark_utils`, `vllm.config`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.v1.spec_decode.ngram_proposer`, `vllm.v1.worker.gpu_input_batch`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `gc`, `time`, `unittest`；第三方依赖，如 `numpy`, `tabulate`；项目内部模块，如 `benchmark_utils`, `vllm.config`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.v1.spec_decode.ngram_proposer`, `vllm.v1.worker.gpu_input_batch`。这些依赖构成了后续基准测试逻辑的基础。

### Function `benchmark_propose` (lines 28-82)
```python
def benchmark_propose(args):
    rows = []
    for max_ngram in args.max_ngram:
        collector = TimeCollector(TimeCollector.US)

        model_config = ModelConfig(
            model="facebook/opt-125m",
            max_model_len=args.num_token + args.num_spec_token,
            tokenizer="facebook/opt-125m",
            tokenizer_mode="auto",
            dtype="auto",
            seed=0,
            trust_remote_code=False,
        )
        proposer = NgramProposer(
            vllm_config=VllmConfig(
                model_config=model_config,
                speculative_config=SpeculativeConfig(
                    prompt_lookup_min=args.min_ngram,
                    prompt_lookup_max=max_ngram,
                    num_speculative_tokens=args.num_spec_token,
                    method="ngram",
                ),
            )
        )

        # Warm up
        proposer.propose(np.random.randint(0, 20, (args.num_token,)))

        gc.collect()
        for _ in range(args.num_iteration):
            tokens = np.random.randint(0, 20, (args.num_req, args.num_token))
            with collector:
                for i in range(args.num_req):
    # ... omitted for brevity ...
                "Max Ngram",
                "Avg (us)",
                "Max (us)",
            ],
            tablefmt="grid",
            floatfmt=".3f",
        )
    )
```
**EN:** `benchmark_propose` coordinates or measures benchmark orchestration. It mainly works with `args` and relies on `TimeCollector`, `ModelConfig`, `NgramProposer`, `VllmConfig`, `SpeculativeConfig`, `proposer.propose` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_propose` 负责协调或测量基准测试编排。 它主要处理 `args`，并结合 `TimeCollector`, `ModelConfig`, `NgramProposer`, `VllmConfig`, `SpeculativeConfig`, `proposer.propose` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Function `benchmark_batched_propose` (lines 85-157)
```python
def benchmark_batched_propose(args):
    NUM_SPECULATIVE_TOKENS_NGRAM = 10
    PROMPT_LOOKUP_MIN = 5
    PROMPT_LOOKUP_MAX = 15
    MAX_MODEL_LEN = int(1e7)
    DEVICE = current_platform.device_type

    model_config = ModelConfig(model="facebook/opt-125m", runner="generate")

    speculative_config = SpeculativeConfig(
        target_model_config=model_config,
        target_parallel_config=ParallelConfig(),
        method="ngram",
        num_speculative_tokens=NUM_SPECULATIVE_TOKENS_NGRAM,
        prompt_lookup_max=PROMPT_LOOKUP_MAX,
        prompt_lookup_min=PROMPT_LOOKUP_MIN,
    )

    vllm_config = VllmConfig(
        model_config=model_config,
        cache_config=CacheConfig(),
        speculative_config=speculative_config,
        device_config=DeviceConfig(device=current_platform.device_type),
        parallel_config=ParallelConfig(),
        load_config=LoadConfig(),
        scheduler_config=SchedulerConfig(
            max_model_len=model_config.max_model_len,
            is_encoder_decoder=model_config.is_encoder_decoder,
        ),
    )

    # monkey patch vllm.v1.worker.gpu_model_runner.get_pp_group
    mock_pp_group = mock.MagicMock()
    mock_pp_group.world_size = 1
    # ... omitted for brevity ...
            start = time.time()
            runner.drafter.propose(
                sampled_token_ids,
                dummy_input_batch.num_tokens_no_spec,
                dummy_input_batch.token_ids_cpu,
            )
            end = time.time()
            print(f"Iteration time (s): {end - start}")
```
**EN:** `benchmark_batched_propose` implements batch behavior benchmarks. It mainly works with `args` and relies on `int`, `ModelConfig`, `SpeculativeConfig`, `ParallelConfig`, `VllmConfig`, `CacheConfig` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_batched_propose` 负责实现批处理行为基准。 它主要处理 `args`，并结合 `int`, `ModelConfig`, `SpeculativeConfig`, `ParallelConfig`, `VllmConfig`, `CacheConfig` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Function `invoke_main` (lines 160-203)
```python
def invoke_main() -> None:
    parser = FlexibleArgumentParser(
        description="Benchmark the performance of N-gram speculative decode drafting"
    )
    parser.add_argument(
        "--batched", action="store_true", help="consider time to prepare batch"
    )
    parser.add_argument(
        "--num-iteration",
        type=int,
        default=100,
        help="Number of iterations to run to stabilize final data readings",
    )
    parser.add_argument(
        "--num-req", type=int, default=128, help="Number of requests in the batch"
    )
    parser.add_argument(
        "--num-token", type=int, default=1500, help="Number of tokens for each request"
    )
    parser.add_argument(
        "--min-ngram",
        type=int,
        default=3,
        help="Minimum n-gram to match",
    )
    parser.add_argument(
        "--max-ngram",
        type=int,
        nargs="*",
        default=[5, 7, 10, 15, 20],
        help="Maximum n-gram to match",
    )
    parser.add_argument(
        "--num-spec-token",
        type=int,
        default=3,
        help="Number of speculative tokens to generate",
    )
    args = parser.parse_args()

    if not args.batched:
        benchmark_propose(args)
    else:
        benchmark_batched_propose(args)
```
**EN:** `invoke_main` implements a helper used by `benchmark_ngram_proposer.py`. It mainly works with no explicit parameters and relies on `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `benchmark_propose`, `benchmark_batched_propose` plus branching to move data through this part of the benchmark pipeline.
**CN:** `invoke_main` 负责实现 `benchmark_ngram_proposer.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `benchmark_propose`, `benchmark_batched_propose` 以及 条件分支 来完成这一段基准测试流程。

### Top-level setup (lines 206-210)
```python
"""
# Example command lines:
# time python3 benchmarks/benchmark_ngram_proposer.py
# time python3 benchmarks/benchmark_ngram_proposer.py --batched --num-iteration 4 --num-token 1000000 --num-req 128
"""  # noqa: E501
```
**EN:** This top-level block prepares shared state such as module-level state. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 module-level state。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Entry point (lines 211-212)
```python
if __name__ == "__main__":
    invoke_main()  # pragma: no cover
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `invoke_main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `invoke_main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。

## Dependencies / 依赖关系
- **EN:** Standard library: `gc`, `time`, `unittest`.
- **CN:** 标准库依赖：`gc`, `time`, `unittest`。
- **EN:** Third-party packages: `numpy`, `tabulate`.
- **CN:** 第三方依赖：`numpy`, `tabulate`。
- **EN:** Internal modules: `benchmark_utils`, `vllm.config`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.v1.spec_decode.ngram_proposer`, `vllm.v1.worker.gpu_input_batch`.
- **CN:** 内部模块：`benchmark_utils`, `vllm.config`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.v1.spec_decode.ngram_proposer`, `vllm.v1.worker.gpu_input_batch`。
