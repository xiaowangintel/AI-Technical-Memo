# benchmark_topk_topp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_topk_topp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, sampling/top-k benchmarks, centered around `BenchmarkConfig`, `calculate_ops_pct`, `create_logits`, `measure_memory`. / 实现与基准测试编排、采样与 top-k 基准相关的逻辑，核心符号包括 `BenchmarkConfig`, `calculate_ops_pct`, `create_logits`, `measure_memory`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-15)
```python
"""
Benchmark comparing Triton vs PyTorch sort-based top-k/top-p implementations.

Compares:
- apply_top_k_top_p_triton (Triton binary search)
- apply_top_k_top_p (PyTorch sort-based)

Scenarios:
- top_k only (whole batch, partial batch)
- top_p only (whole batch, partial batch)
- mix of top_k and top_p
"""
```
**EN:** The module docstring introduces Benchmark comparing Triton vs PyTorch sort-based top-k/top-p implementations. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark comparing Triton vs PyTorch sort-based top-k/top-p implementations 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 17-27)
```python
import argparse
import gc
from dataclasses import dataclass

import torch

from vllm.v1.sample.ops.topk_topp_sampler import apply_top_k_top_p_pytorch
from vllm.v1.sample.ops.topk_topp_triton import (
    apply_top_k_top_p_triton,
    reset_buffer_cache,
)
```
**EN:** This block gathers standard-library helpers such as `argparse`, `gc`, `dataclasses`; third-party packages such as `torch`; project-local modules such as `vllm.v1.sample.ops.topk_topp_sampler`, `vllm.v1.sample.ops.topk_topp_triton`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `gc`, `dataclasses`；第三方依赖，如 `torch`；项目内部模块，如 `vllm.v1.sample.ops.topk_topp_sampler`, `vllm.v1.sample.ops.topk_topp_triton`。这些依赖构成了后续基准测试逻辑的基础。

### Class `BenchmarkConfig` (lines 31-41)
```python
class BenchmarkConfig:
    """Configuration for a benchmark run."""

    name: str
    batch_size: int
    vocab_size: int
    # k and p can be tensors or None
    k_values: torch.Tensor | None  # [batch_size] or None
    p_values: torch.Tensor | None  # [batch_size] or None
    description: str
    ops_pct: float = 0.0  # Percentage of ops relative to batch size
```
**EN:** Class `BenchmarkConfig` is the main object-oriented wrapper for this module. Its docstring says: Configuration for a benchmark run. It extends `object` and organizes behavior through its methods.
**CN:** 类 `BenchmarkConfig` 是该模块中的主要面向对象封装。文档字符串指出：Configuration for a benchmark run。它继承自 `object`，并通过 its methods 组织行为。

### Function `calculate_ops_pct` (lines 44-66)
```python
def calculate_ops_pct(
    k_values: torch.Tensor | None,
    p_values: torch.Tensor | None,
    vocab_size: int,
    batch_size: int,
) -> float:
    """
    Calculate the percentage of active top-k and top-p operations.

    Returns percentage where 100% = batch_size ops.
    E.g., if all rows have both top-k and top-p active, returns 200%.
    """
    active_ops = 0

    if k_values is not None:
        # Count rows where k < vocab_size (active top-k filtering)
        active_ops += (k_values < vocab_size).sum().item()

    if p_values is not None:
        # Count rows where p < 1.0 (active top-p filtering)
        active_ops += (p_values < 1.0).sum().item()

    return (active_ops / batch_size) * 100 if batch_size > 0 else 0.0
```
**EN:** `calculate_ops_pct` Calculate the percentage of active top-k and top-p operations. It mainly works with `k_values`, `p_values`, `vocab_size`, `batch_size` and relies on `sum.item`, `sum` plus branching to move data through this part of the benchmark pipeline.
**CN:** `calculate_ops_pct` 的职责是：Calculate the percentage of active top-k and top-p operations。 它主要处理 `k_values`, `p_values`, `vocab_size`, `batch_size`，并结合 `sum.item`, `sum` 以及 条件分支 来完成这一段基准测试流程。

### Function `create_logits` (lines 69-92)
```python
def create_logits(
    batch_size: int, vocab_size: int, device: str = "cuda"
) -> torch.Tensor:
    """Create random logits mimicking a realistic LLM distribution.

    Uses a Zipf-like probability distribution (rank^-1.1) converted to logits
    via log, then randomly permuted per row. This produces a peaked distribution
    where a small number of tokens capture most probability mass, similar to
    real model outputs.
    """
    # Create Zipf-like probabilities: p(rank) ~ rank^(-alpha)
    ranks = torch.arange(1, vocab_size + 1, dtype=torch.float32, device=device)
    probs = ranks.pow(-1.1)
    probs = probs / probs.sum()

    # Convert to logits (log-probabilities, unnormalized is fine)
    base_logits = probs.log()

    # Broadcast to batch and randomly permute each row
    logits = base_logits.unsqueeze(0).expand(batch_size, -1).clone()
    for i in range(batch_size):
        logits[i] = logits[i, torch.randperm(vocab_size, device=device)]

    return logits
```
**EN:** `create_logits` Create random logits mimicking a realistic LLM distribution. It mainly works with `batch_size`, `vocab_size`, `device` and relies on `torch.arange`, `ranks.pow`, `probs.sum`, `probs.log`, `base_logits.unsqueeze.expand.clone`, `base_logits.unsqueeze.expand` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `create_logits` 的职责是：Create random logits mimicking a realistic LLM distribution。 它主要处理 `batch_size`, `vocab_size`, `device`，并结合 `torch.arange`, `ranks.pow`, `probs.sum`, `probs.log`, `base_logits.unsqueeze.expand.clone`, `base_logits.unsqueeze.expand` 以及 循环迭代 来完成这一段基准测试流程。

### Function `measure_memory` (lines 95-101)
```python
def measure_memory() -> tuple[int, int]:
    """Return (allocated, reserved) memory in bytes."""
    torch.accelerator.synchronize()
    return (
        torch.accelerator.memory_allocated(),
        torch.accelerator.max_memory_allocated(),
    )
```
**EN:** `measure_memory` Return (allocated, reserved) memory in bytes. It mainly works with no explicit parameters and relies on `torch.accelerator.synchronize`, `torch.accelerator.memory_allocated`, `torch.accelerator.max_memory_allocated` plus value production to move data through this part of the benchmark pipeline.
**CN:** `measure_memory` 的职责是：Return (allocated, reserved) memory in bytes。 它主要处理 无显式参数，并结合 `torch.accelerator.synchronize`, `torch.accelerator.memory_allocated`, `torch.accelerator.max_memory_allocated` 以及 结果返回 来完成这一段基准测试流程。

### Function `reset_memory_stats` (lines 104-109)
```python
def reset_memory_stats():
    """Reset peak memory statistics."""
    reset_buffer_cache()
    torch.accelerator.reset_peak_memory_stats()
    torch.accelerator.empty_cache()
    gc.collect()
```
**EN:** `reset_memory_stats` Reset peak memory statistics. It mainly works with no explicit parameters and relies on `reset_buffer_cache`, `torch.accelerator.reset_peak_memory_stats`, `torch.accelerator.empty_cache`, `gc.collect` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `reset_memory_stats` 的职责是：Reset peak memory statistics。 它主要处理 无显式参数，并结合 `reset_buffer_cache`, `torch.accelerator.reset_peak_memory_stats`, `torch.accelerator.empty_cache`, `gc.collect` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `benchmark_function` (lines 112-157)
```python
def benchmark_function(
    func,
    logits: torch.Tensor,
    k: torch.Tensor | None,
    p: torch.Tensor | None,
    warmup_iters: int = 5,
    benchmark_iters: int = 20,
) -> tuple[float, int]:
    """
    Benchmark a function and return (avg_time_ms, peak_memory_bytes).

    Returns average time in milliseconds and peak memory usage.
    """
    # Warmup
    for _ in range(warmup_iters):
        logits_copy = logits.clone()
        func(logits_copy, k, p)
    torch.accelerator.synchronize()

    # Reset memory stats before benchmark
    reset_memory_stats()

    # Benchmark
    start_events = [
        torch.cuda.Event(enable_timing=True) for _ in range(benchmark_iters)
    ]
    end_events = [torch.cuda.Event(enable_timing=True) for _ in range(benchmark_iters)]

    for i in range(benchmark_iters):
        logits_copy = logits.clone()
        start_events[i].record()
        func(logits_copy, k, p)
        end_events[i].record()

    torch.accelerator.synchronize()

    # Calculate timing
    times = [
        start_events[i].elapsed_time(end_events[i]) for i in range(benchmark_iters)
    ]
    avg_time = sum(times) / len(times)

    # Get peak memory
    _, peak_memory = measure_memory()

    return avg_time, peak_memory
```
**EN:** `benchmark_function` Benchmark a function and return (avg_time_ms, peak_memory_bytes). It mainly works with `func`, `logits`, `k`, `p`, `warmup_iters`, ... and relies on `range`, `logits.clone`, `func`, `torch.accelerator.synchronize`, `reset_memory_stats`, `torch.cuda.Event` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `benchmark_function` 的职责是：Benchmark a function and return (avg_time_ms, peak_memory_bytes)。 它主要处理 `func`, `logits`, `k`, `p`, `warmup_iters`, ...，并结合 `range`, `logits.clone`, `func`, `torch.accelerator.synchronize`, `reset_memory_stats`, `torch.cuda.Event` 以及 循环迭代 来完成这一段基准测试流程。

### Function `create_benchmark_configs` (lines 160-276)
```python
def create_benchmark_configs(
    batch_sizes: list[int],
    vocab_sizes: list[int],
    device: str = "cuda",
) -> list[BenchmarkConfig]:
    """Create all benchmark configurations."""
    configs = []

    for vocab_size in vocab_sizes:
        for batch_size in batch_sizes:
            # 1. Top-k only - whole batch (all rows have k < vocab_size)
            k_all = torch.full((batch_size,), 50, dtype=torch.int32, device=device)
            configs.append(
                BenchmarkConfig(
                    name=f"topk_whole_b{batch_size}_v{vocab_size // 1000}k",
                    batch_size=batch_size,
                    vocab_size=vocab_size,
                    k_values=k_all,
                    p_values=None,
                    description=f"Top-k only (whole batch, k=50), "
                    f"batch={batch_size}, vocab={vocab_size}",
                    ops_pct=calculate_ops_pct(k_all, None, vocab_size, batch_size),
                )
            )

            # 2. Top-k only - partial batch (half have k=50, half have k=vocab_size)
            k_partial = torch.full((batch_size,), 50, dtype=torch.int32, device=device)
            k_partial[batch_size // 2 :] = vocab_size  # No filtering for second half
            configs.append(
                BenchmarkConfig(
                    name=f"topk_partial_b{batch_size}_v{vocab_size // 1000}k",
                    batch_size=batch_size,
                    vocab_size=vocab_size,
                    k_values=k_partial,
    # ... omitted for brevity ...
                    p_values=p_mixed,
                    description=f"Mixed partial (1/3 k=50, 1/3 p=0.9, 1/3 both), "
                    f"batch={batch_size}, vocab={vocab_size}",
                    ops_pct=calculate_ops_pct(k_mixed, p_mixed, vocab_size, batch_size),
                )
            )

    return configs
```
**EN:** `create_benchmark_configs` Create all benchmark configurations. It mainly works with `batch_sizes`, `vocab_sizes`, `device` and relies on `torch.full`, `configs.append`, `BenchmarkConfig`, `calculate_ops_pct` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `create_benchmark_configs` 的职责是：Create all benchmark configurations。 它主要处理 `batch_sizes`, `vocab_sizes`, `device`，并结合 `torch.full`, `configs.append`, `BenchmarkConfig`, `calculate_ops_pct` 以及 循环迭代 来完成这一段基准测试流程。

### Function `format_memory` (lines 279-287)
```python
def format_memory(bytes_val: int) -> str:
    """Format memory in human-readable form."""
    if bytes_val >= 1024**3:
        return f"{bytes_val / (1024**3):.2f} GB"
    elif bytes_val >= 1024**2:
        return f"{bytes_val / (1024**2):.2f} MB"
    elif bytes_val >= 1024:
        return f"{bytes_val / 1024:.2f} KB"
    return f"{bytes_val} B"
```
**EN:** `format_memory` Format memory in human-readable form. It mainly works with `bytes_val` and relies on direct statements plus branching to move data through this part of the benchmark pipeline.
**CN:** `format_memory` 的职责是：Format memory in human-readable form。 它主要处理 `bytes_val`，并结合 direct statements 以及 条件分支 来完成这一段基准测试流程。

### Function `run_benchmark` (lines 290-357)
```python
def run_benchmark(
    configs: list[BenchmarkConfig],
    warmup_iters: int = 5,
    benchmark_iters: int = 20,
    verbose: bool = True,
):
    """Run all benchmarks and print results."""
    results = []

    print("=" * 100)
    print("Top-k/Top-p Benchmark: Triton vs PyTorch Sort-based")
    print("=" * 100)
    print()

    for config in configs:
        if verbose:
            print(f"Running: {config.description}")

        # Create fresh logits for this config
        logits = create_logits(config.batch_size, config.vocab_size)

        # Benchmark Triton
        reset_memory_stats()
        triton_time, triton_mem = benchmark_function(
            apply_top_k_top_p_triton,
            logits,
            config.k_values,
            config.p_values,
            warmup_iters,
            benchmark_iters,
        )

        # Benchmark PyTorch
        reset_memory_stats()
    # ... omitted for brevity ...
            print(f"  Speedup: {speedup:.2f}x, Memory ratio: {mem_ratio:.2f}x")
            print()

        # Clean up
        del logits
        reset_memory_stats()

    return results
```
**EN:** `run_benchmark` Run all benchmarks and print results. It mainly works with `configs`, `warmup_iters`, `benchmark_iters`, `verbose` and relies on `print`, `create_logits`, `reset_memory_stats`, `benchmark_function`, `float`, `results.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `run_benchmark` 的职责是：Run all benchmarks and print results。 它主要处理 `configs`, `warmup_iters`, `benchmark_iters`, `verbose`，并结合 `print`, `create_logits`, `reset_memory_stats`, `benchmark_function`, `float`, `results.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `print_summary_table` (lines 360-398)
```python
def print_summary_table(results: list[dict]):
    """Print a summary table of results."""
    print()
    print("=" * 130)
    print("SUMMARY TABLE")
    print("=" * 130)
    print()

    # Header
    header = (
        f"{'Scenario':<40} {'Batch':>6} {'Vocab':>7} {'Ops%':>6} "
        f"{'Triton (ms)':>12} {'PyTorch (ms)':>13} {'Speedup':>8} "
        f"{'Tri Mem':>10} {'Pyt Mem':>10}"
    )
    print(header)
    print("-" * 130)

    # Group by scenario type
    current_vocab = None
    for result in results:
        config = result["config"]

        # Add separator between vocab sizes
        if current_vocab != config.vocab_size:
            if current_vocab is not None:
                print("-" * 130)
            current_vocab = config.vocab_size

        scenario = config.name.split("_b")[0]  # Extract scenario name
        print(
            f"{scenario:<40} {config.batch_size:>6} {config.vocab_size:>7} "
            f"{config.ops_pct:>5.0f}% "
            f"{result['triton_time_ms']:>12.3f} {result['pytorch_time_ms']:>13.3f} "
            f"{result['speedup']:>7.2f}x "
            f"{format_memory(result['triton_mem']):>10} "
            f"{format_memory(result['pytorch_mem']):>10}"
        )

    print("=" * 130)
```
**EN:** `print_summary_table` Print a summary table of results. It mainly works with `results` and relies on `print`, `config.name.split`, `format_memory` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_summary_table` 的职责是：Print a summary table of results。 它主要处理 `results`，并结合 `print`, `config.name.split`, `format_memory` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `main` (lines 401-470)
```python
def main():
    parser = argparse.ArgumentParser(
        description="Benchmark Triton vs PyTorch sort-based top-k/top-p implementations"
    )
    parser.add_argument(
        "--batch-sizes",
        type=int,
        nargs="+",
        default=[1, 4, 16, 64, 128, 512, 1024, 2048],
        help="Batch sizes to test (default: 1 4 16 64)",
    )
    parser.add_argument(
        "--vocab-sizes",
        type=int,
        nargs="+",
        default=[32768, 131072],  # 32k, 128k
        help="Vocabulary sizes to test (default: 32768 131072)",
    )
    parser.add_argument(
        "--warmup-iters",
        type=int,
        default=5,
        help="Number of warmup iterations (default: 5)",
    )
    parser.add_argument(
        "--benchmark-iters",
        type=int,
        default=20,
        help="Number of benchmark iterations (default: 20)",
    )
    parser.add_argument(
        "--quiet",
        action="store_true",
        help="Only print summary table",
    # ... omitted for brevity ...
        configs,
        warmup_iters=args.warmup_iters,
        benchmark_iters=args.benchmark_iters,
        verbose=not args.quiet,
    )

    # Print summary
    print_summary_table(results)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `torch.cuda.is_available`, `torch.cuda.get_device_name` plus branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `torch.cuda.is_available`, `torch.cuda.get_device_name` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 473-474)
```python
if __name__ == "__main__":
    main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `gc`, `dataclasses`.
- **CN:** 标准库依赖：`argparse`, `gc`, `dataclasses`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.v1.sample.ops.topk_topp_sampler`, `vllm.v1.sample.ops.topk_topp_triton`.
- **CN:** 内部模块：`vllm.v1.sample.ops.topk_topp_sampler`, `vllm.v1.sample.ops.topk_topp_triton`。
