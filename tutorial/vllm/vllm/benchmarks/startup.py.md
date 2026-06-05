# startup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/startup.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Benchmark the cold and warm startup time of vLLM models. / 该文件的核心目的为：Benchmark the cold and warm startup time of vLLM models.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-9)
```python
"""Benchmark the cold and warm startup time of vLLM models.

This script measures total startup time (including model loading, compilation,
and cache operations) for both cold and warm scenarios:
- Cold startup: Fresh start with no caches (temporary cache directories)
- Warm startup: Using cached compilation and model info
"""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 11-28)
```python
import argparse

import json

import multiprocessing

import os

import shutil

import tempfile

import time

from contextlib import contextmanager

from typing import Any, NamedTuple

import numpy as np

from tqdm import tqdm

from vllm.benchmarks.lib.utils import (
    convert_to_pytorch_benchmark_format,
    write_to_json,
)

from vllm.engine.arg_utils import EngineArgs
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 30-59)
```python
PERCENTAGES = [10, 25, 50, 75, 90, 99]

_BASE_METRICS = [
    MetricDesc("total_startup_time", "startup", "Startup time"),
    MetricDesc("compilation_time", "compilation", "Compilation time"),
]

_ENCODER_METRIC = MetricDesc(
    "encoder_compilation_time",
    "encoder_compilation",
    "Encoder compilation time",
)
```
**EN:** This constant/configuration block defines `PERCENTAGES`, `_BASE_METRICS`, `_ENCODER_METRIC`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `PERCENTAGES`, `_BASE_METRICS`, `_ENCODER_METRIC`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `MetricDesc` (lines 33-38)
```python
class MetricDesc(NamedTuple):
    """Descriptor for a metric to collect from each iteration."""

    iter_key: str  # key in the iteration result dict
    suffix: str  # result key suffix, e.g. "startup", "compilation"
    display_name: str
```
**EN:** Class `MetricDesc` is a structured building block in this module. It inherits from `NamedTuple`. The class docstring says: Descriptor for a metric to collect from each iteration.
**CN:** 类 `MetricDesc` 是该模块中的结构化构件，继承自 `NamedTuple`。 类文档说明：Descriptor for a metric to collect from each iteration.

### Class `MetricStats` (lines 41-48)
```python
class MetricStats(NamedTuple):
    """Aggregated statistics for a single benchmark metric."""

    key: str  # e.g. "cold_startup", "warm_encoder_compilation"
    display_name: str
    values: list[float]
    avg: float
    percentiles: dict[int, float]
```
**EN:** Class `MetricStats` is a structured building block in this module. It inherits from `NamedTuple`. The class docstring says: Aggregated statistics for a single benchmark metric.
**CN:** 类 `MetricStats` 是该模块中的结构化构件，继承自 `NamedTuple`。 类文档说明：Aggregated statistics for a single benchmark metric.

### Function `_compute_metric` (lines 62-75)
```python
def _compute_metric(
    phase: str,
    desc: MetricDesc,
    iterations: list[dict[str, float]],
) -> MetricStats:
    values = [m[desc.iter_key] for m in iterations]
    arr = np.array(values)
    return MetricStats(
        key=f"{phase}_{desc.suffix}",
        display_name=desc.display_name,
        values=values,
        avg=float(np.mean(arr)),
        percentiles=dict(zip(PERCENTAGES, np.percentile(arr, PERCENTAGES).tolist())),
    )
```
**EN:** Function `_compute_metric` provides a reusable helper around the module's main workflow. Key calls such as `np.array`, `MetricStats`, `float`, `np.mean`, `dict` show the concrete execution path.
**CN:** Function `_compute_metric` 为模块主流程提供可复用的辅助逻辑。 像 `np.array`, `MetricStats`, `float`, `np.mean`, `dict` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_collect_phase_metrics` (lines 78-86)
```python
def _collect_phase_metrics(
    phase: str,
    iterations: list[dict[str, float]],
    has_encoder: bool,
) -> list[MetricStats]:
    metrics = [_compute_metric(phase, desc, iterations) for desc in _BASE_METRICS]
    if has_encoder:
        metrics.append(_compute_metric(phase, _ENCODER_METRIC, iterations))
    return metrics
```
**EN:** Function `_collect_phase_metrics` provides a reusable helper around the module's main workflow. Key calls such as `_compute_metric`, `metrics.append` show the concrete execution path.
**CN:** Function `_collect_phase_metrics` 为模块主流程提供可复用的辅助逻辑。 像 `_compute_metric`, `metrics.append` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_print_phase` (lines 89-96)
```python
def _print_phase(phase_name: str, metrics: list[MetricStats]) -> None:
    print(f"\n{phase_name}:")
    for m in metrics:
        print(f"Avg {m.display_name.lower()}: {m.avg:.2f} seconds")
    for m in metrics:
        print(f"{m.display_name} percentiles:")
        for pct, val in m.percentiles.items():
            print(f"  {pct}%: {val:.2f} seconds")
```
**EN:** Function `_print_phase` provides a reusable helper around the module's main workflow. Key calls such as `print`, `m.display_name.lower`, `m.percentiles.items` show the concrete execution path.
**CN:** Function `_print_phase` 为模块主流程提供可复用的辅助逻辑。 像 `print`, `m.display_name.lower`, `m.percentiles.items` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_metric_to_json` (lines 99-104)
```python
def _metric_to_json(m: MetricStats) -> dict[str, Any]:
    return {
        f"avg_{m.key}_time": m.avg,
        f"{m.key}_times": m.values,
        f"{m.key}_percentiles": m.percentiles,
    }
```
**EN:** Function `_metric_to_json` provides a reusable helper around the module's main workflow.
**CN:** Function `_metric_to_json` 为模块主流程提供可复用的辅助逻辑。

### Function `cold_startup` (lines 108-130)
```python
def cold_startup():
    """
    Context manager to measure cold startup time:
    1. Uses a temporary directory for vLLM cache to avoid any pollution
       between cold startup iterations.
    2. Uses inductor's fresh_cache to clear torch.compile caches.
    """
    from torch._inductor.utils import fresh_cache

    # Use temporary directory for caching to avoid any pollution between cold startups
    original_cache_root = os.environ.get("VLLM_CACHE_ROOT")
    temp_cache_dir = tempfile.mkdtemp(prefix="vllm_startup_bench_cold_")
    try:
        os.environ["VLLM_CACHE_ROOT"] = temp_cache_dir
        with fresh_cache():
            yield
    finally:
        # Clean up temporary cache directory
        shutil.rmtree(temp_cache_dir, ignore_errors=True)
        if original_cache_root:
            os.environ["VLLM_CACHE_ROOT"] = original_cache_root
        else:
            os.environ.pop("VLLM_CACHE_ROOT", None)
```
**EN:** Function `cold_startup` provides a reusable helper around the module's main workflow. The docstring highlights: Context manager to measure cold startup time: 1. Key calls such as `os.environ.get`, `tempfile.mkdtemp`, `fresh_cache`, `shutil.rmtree`, `os.environ.pop` show the concrete execution path.
**CN:** Function `cold_startup` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Context manager to measure cold startup time: 1. 像 `os.environ.get`, `tempfile.mkdtemp`, `fresh_cache`, `shutil.rmtree`, `os.environ.pop` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_startup_in_subprocess` (lines 133-173)
```python
def run_startup_in_subprocess(engine_args, result_queue):
    """
    Run LLM startup in a subprocess and return timing metrics via a queue.
    This ensures complete isolation between iterations.
    """
    try:
        # Import inside the subprocess to avoid issues with forking
        from vllm import LLM

        # Measure total startup time
        start_time = time.perf_counter()

        llm = LLM.from_engine_args(engine_args)

        total_startup_time = time.perf_counter() - start_time

        # Extract compilation time if available
        compilation_time = 0.0
        encoder_compilation_time = 0.0
        if hasattr(llm.llm_engine, "vllm_config"):
            vllm_config = llm.llm_engine.vllm_config
            if (
                hasattr(vllm_config, "compilation_config")
                and vllm_config.compilation_config is not None
            ):
    # ... omitted for brevity ...
        result_queue.put(None)
        result_queue.put(str(e))
```
**EN:** Function `run_startup_in_subprocess` provides a reusable helper around the module's main workflow. The docstring highlights: Run LLM startup in a subprocess and return timing metrics via a queue. Key calls such as `time.perf_counter`, `LLM.from_engine_args`, `hasattr`, `result_queue.put`, `str` show the concrete execution path.
**CN:** Function `run_startup_in_subprocess` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Run LLM startup in a subprocess and return timing metrics via a queue. 像 `time.perf_counter`, `LLM.from_engine_args`, `hasattr`, `result_queue.put`, `str` 这样的关键调用展示了该代码块的具体执行路径。

### Function `save_to_pytorch_benchmark_format` (lines 176-190)
```python
def save_to_pytorch_benchmark_format(
    args: argparse.Namespace, metrics: list[MetricStats]
) -> None:
    base_name = os.path.splitext(args.output_json)[0]
    for m in metrics:
        records = convert_to_pytorch_benchmark_format(
            args=args,
            metrics={f"avg_{m.key}_time": [m.avg]},
            extra_info={
                f"{m.key}_times": m.values,
                f"{m.key}_percentiles": m.percentiles,
            },
        )
        if records:
            write_to_json(f"{base_name}.{m.key}.pytorch.json", records)
```
**EN:** Function `save_to_pytorch_benchmark_format` serializes data and writes it to a target representation. Key calls such as `os.path.splitext`, `convert_to_pytorch_benchmark_format`, `write_to_json` show the concrete execution path.
**CN:** Function `save_to_pytorch_benchmark_format` 负责序列化数据并写入目标表示。 像 `os.path.splitext`, `convert_to_pytorch_benchmark_format`, `write_to_json` 这样的关键调用展示了该代码块的具体执行路径。

### Function `add_cli_args` (lines 193-220)
```python
def add_cli_args(parser: argparse.ArgumentParser):
    parser.add_argument(
        "--num-iters-cold",
        type=int,
        default=3,
        help="Number of cold startup iterations.",
    )
    parser.add_argument(
        "--num-iters-warmup",
        type=int,
        default=1,
        help="Number of warmup iterations before benchmarking warm startups.",
    )
    parser.add_argument(
        "--num-iters-warm",
        type=int,
        default=3,
        help="Number of warm startup iterations.",
    )
    parser.add_argument(
        "--output-json",
        type=str,
        default=None,
        help="Path to save the startup time results in JSON format.",
    )

    parser = EngineArgs.add_cli_args(parser)
    return parser
```
**EN:** Function `add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument`, `EngineArgs.add_cli_args` show the concrete execution path.
**CN:** Function `add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument`, `EngineArgs.add_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 223-305)
```python
def main(args: argparse.Namespace):
    # Set multiprocessing start method to 'spawn' for clean process isolation
    # This ensures each subprocess starts fresh without inheriting state
    multiprocessing.set_start_method("spawn", force=True)

    engine_args = EngineArgs.from_cli_args(args)

    def create_llm_and_measure_startup():
        """
        Create LLM instance in a subprocess and measure startup time.
        Returns timing metrics, using subprocess for complete isolation.
        """

        # Create a queue for inter-process communication
        result_queue = multiprocessing.Queue()
        process = multiprocessing.Process(
            target=run_startup_in_subprocess,
            args=(
                engine_args,
                result_queue,
            ),
        )
        process.start()
        process.join()

    # ... omitted for brevity ...
            json.dump(results, f, indent=4)
        save_to_pytorch_benchmark_format(args, all_metrics)
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `multiprocessing.set_start_method`, `EngineArgs.from_cli_args`, `multiprocessing.Queue`, `multiprocessing.Process`, `process.start` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `multiprocessing.set_start_method`, `EngineArgs.from_cli_args`, `multiprocessing.Queue`, `multiprocessing.Process`, `process.start` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import json`, `import multiprocessing`, `import os`, `import shutil`, `import tempfile`, `import time`, `from contextlib import contextmanager`, `from typing import Any, NamedTuple`
- **Third-party / 第三方**: `import numpy as np`, `from tqdm import tqdm`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.lib.utils import convert_to_pytorch_benchmark_format, write_to_json`, `from vllm.engine.arg_utils import EngineArgs`
