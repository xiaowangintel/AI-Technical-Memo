# benchmark_deepgemm_dsv3_router_gemm_blackwell.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/deepseek/benchmark_deepgemm_dsv3_router_gemm_blackwell.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels deepseek deepgemm dsv3 router. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels deepseek deepgemm dsv3 router 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and setup / 导入与初始化
```python
import argparse
import os
from typing import List

import torch
import triton
from flashinfer.gemm import mm_M1_16_K7168_N256
from sgl_kernel import dsv3_router_gemm

N = 256
K = 7168
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核。

### Lines 14-19: Function `create_benchmark_configs` / 函数 `create_benchmark_configs`
```python
def create_benchmark_configs(tp_sizes: List[int]):
    configs = []
    for tp_size in tp_sizes:
        for m in range(1, 17):
            configs.append((m, N, K, tp_size))
    return configs
```
**EN:** `create_benchmark_configs` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `configs` to the caller. Notable calls include `range`, `configs.append`.
**CN:** `create_benchmark_configs` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `configs`。其中较关键的调用包括 `range`, `configs.append`。

### Lines 22-36: Function `dsv3_router_gemm_flashinfer` / 函数 `dsv3_router_gemm_flashinfer`
```python
def dsv3_router_gemm_flashinfer(
    hidden_states: torch.Tensor,
    router_weights: torch.Tensor,
):
    """Flashinfer implementation of dsv3 router gemm"""
    output = torch.empty(
        hidden_states.shape[0],
        router_weights.shape[0],
        device="cuda",
        dtype=torch.float32,
    )
    mm_M1_16_K7168_N256(
        hidden_states, router_weights.t(), output, launch_with_pdl=args.use_pdl
    )
    return output
```
**EN:** `dsv3_router_gemm_flashinfer` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Flashinfer implementation of dsv3 router gemm It returns `output` to the caller. Notable calls include `torch.empty`, `mm_M1_16_K7168_N256`, `router_weights.t`.
**CN:** `dsv3_router_gemm_flashinfer` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `output`。其中较关键的调用包括 `torch.empty`, `mm_M1_16_K7168_N256`, `router_weights.t`。

### Lines 39-49: Function `dsv3_router_gemm_sgl` / 函数 `dsv3_router_gemm_sgl`
```python
def dsv3_router_gemm_sgl(
    hidden_states: torch.Tensor,
    router_weights: torch.Tensor,
):
    """SGLang implementation of dsv3 router gemm"""
    output = dsv3_router_gemm(
        hidden_states,
        router_weights,
        out_dtype=torch.float32,
    )
    return output
```
**EN:** `dsv3_router_gemm_sgl` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: SGLang implementation of dsv3 router gemm It returns `output` to the caller. Notable calls include `dsv3_router_gemm`.
**CN:** `dsv3_router_gemm_sgl` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `output`。其中较关键的调用包括 `dsv3_router_gemm`。

### Lines 52-73: Function `check_accuracy` / 函数 `check_accuracy`
```python
def check_accuracy(a, b, atol, rtol, percent):
    """Unified accuracy checking function with detailed error reporting."""
    if not torch.isfinite(a).all():
        print("Non-finite values in reference output")
        return False
    if not torch.isfinite(b).all():
        print("Non-finite values in actual output")
        return False
    assert a.shape == b.shape, f"Shape mismatch: {a.shape} vs {b.shape}"

    close = torch.isclose(a, b, atol=atol, rtol=rtol)
    match_ratio = close.float().mean()
    if match_ratio >= percent:
        return True

    mismatch_percent = 1.0 - match_ratio.item()
    if mismatch_percent > 1 - percent:
        print(
            f"Mismatch percentage is {mismatch_percent:.4f} for rtol {rtol} "
            f"(threshold: {1 - percent:.4f})"
        )
        return False
```
**EN:** `check_accuracy` is a function that prepares tensors and invokes GPU kernels and computes evaluation scores and aggregate statistics. The docstring frames it as: Unified accuracy checking function with detailed error reporting. It returns `False` to the caller. Notable calls include `torch.isclose`, `close.float().mean`, `torch.isfinite(a).all`.
**CN:** `check_accuracy` 是一个函数，用于准备张量并调用 GPU 内核、计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `False`。其中较关键的调用包括 `torch.isclose`, `close.float().mean`, `torch.isfinite(a).all`。

### Lines 76-97: Function `calculate_diff` / 函数 `calculate_diff`
```python
def calculate_diff(m: int, n: int, k: int):
    hidden_states = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
    router_weights = torch.randn((n, k), device="cuda", dtype=torch.bfloat16)

    out_flashinfer = dsv3_router_gemm_flashinfer(
        hidden_states.clone(memory_format=torch.contiguous_format),
        router_weights.clone(memory_format=torch.contiguous_format),
    )

    out_sgl = dsv3_router_gemm_sgl(
        hidden_states.clone(memory_format=torch.contiguous_format),
        router_weights.clone(memory_format=torch.contiguous_format),
    )

    print(f"Shape m={m}, n={n}, k={k}:")
    print(f"Using PDL={args.use_pdl}")
    print(f"Flashinfer output: {out_flashinfer[0, 0:5]}")
    print(f"SGLang output: {out_sgl[0, 0:5]}")

    flashinfer_sgl_match = check_accuracy(out_flashinfer, out_sgl, 0.1, 0.6, 0.95)
    print("Correctness check:")
    print(f"  - Flashinfer vs SGLang: {'✅' if flashinfer_sgl_match else '❌'}")
```
**EN:** `calculate_diff` is a function that prepares tensors and invokes GPU kernels and computes evaluation scores and aggregate statistics. Notable calls include `torch.randn`, `dsv3_router_gemm_flashinfer`, `dsv3_router_gemm_sgl`.
**CN:** `calculate_diff` 是一个函数，用于准备张量并调用 GPU 内核、计算评测分数与聚合统计结果。其中较关键的调用包括 `torch.randn`, `dsv3_router_gemm_flashinfer`, `dsv3_router_gemm_sgl`。

### Lines 100-134: Function `_benchmark` / 函数 `_benchmark`
```python
def _benchmark(m, n, k, tp_size, provider):
    print(f"Shape (m={m}, n={n}, k={k}, tp={tp_size}), Provider: {provider}")
    hidden_states = torch.randn(
        (m, k), device="cuda", dtype=torch.bfloat16
    ).contiguous()
    router_weights = torch.randn(
        (n, k), device="cuda", dtype=torch.bfloat16
    ).contiguous()

    quantiles = [0.5, 0.2, 0.8]

    if provider == "sglang":
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: dsv3_router_gemm_sgl(
                hidden_states.clone(memory_format=torch.contiguous_format),
                router_weights.clone(memory_format=torch.contiguous_format),
            ),
            quantiles=quantiles,
        )
    elif provider == "flashinfer":
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: dsv3_router_gemm_flashinfer(
                hidden_states.clone(memory_format=torch.contiguous_format),
                router_weights.clone(memory_format=torch.contiguous_format),
            ),
            quantiles=quantiles,
        )

    # Calculate TFLOPS
    flops = 2 * m * n * k  # multiply-adds
    tflops = flops / (ms * 1e-3) / 1e12

    # Print shape-specific results with TFLOPS
    print(f"Time: {ms*1000:.2f} us, TFLOPS: {tflops:.2f}")
    return ms, max_ms, min_ms
```
**EN:** `_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `(ms, max_ms, min_ms)` to the caller. Notable calls include `print`, `torch.randn((m, k), device='cuda', dtype=torch.bfloat16).contiguous`, `torch.randn((n, k), device='cuda', dtype=torch.bfloat16).contiguous`.
**CN:** `_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `(ms, max_ms, min_ms)`。其中较关键的调用包括 `print`, `torch.randn((m, k), device='cuda', dtype=torch.bfloat16).contiguous`, `torch.randn((n, k), device='cuda', dtype=torch.bfloat16).contiguous`。

### Lines 137-159: Function `get_benchmark_plot_friendly` / 函数 `get_benchmark_plot_friendly`
```python
def get_benchmark_plot_friendly(tp_sizes):
    all_configs = create_benchmark_configs(tp_sizes)
    x_vals = list(range(len(all_configs)))

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["cfg_id"],
            x_vals=x_vals,
            line_arg="provider",
            line_vals=["sglang", "flashinfer"],
            line_names=["SGLang", "Flashinfer"],
            styles=[("blue", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"fp8-gemm-performance-comparison-tp-{'-'.join(str(tp) for tp in tp_sizes)}",
            args={},
        )
    )
    def benchmark(cfg_id, provider):
        m, n, k, tp_size = all_configs[cfg_id]
        ms, min_ms, max_ms = _benchmark(m, n, k, tp_size, provider)
        return ms * 1000, max_ms * 1000, min_ms * 1000  # convert to ms

    return benchmark
```
**EN:** `get_benchmark_plot_friendly` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `benchmark` to the caller. Notable calls include `create_benchmark_configs`, `list`, `triton.testing.perf_report`.
**CN:** `get_benchmark_plot_friendly` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `benchmark`。其中较关键的调用包括 `create_benchmark_configs`, `list`, `triton.testing.perf_report`。

### Lines 162-187: Function `get_benchmark` / 函数 `get_benchmark`
```python
def get_benchmark(tp_sizes):
    all_configs = create_benchmark_configs(tp_sizes)

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=[
                "m",
                "n",
                "k",
                "tp_size",
            ],
            x_vals=[list(config) for config in all_configs],
            line_arg="provider",
            line_vals=["sglang", "flashinfer"],
            line_names=["SGLang", "Flashinfer"],
            styles=[("blue", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"fp8-gemm-performance-comparison-tp-{'-'.join(str(tp) for tp in tp_sizes)}",
            args={},
        )
    )
    def benchmark(m, n, k, tp_size, provider):
        ms, min_ms, max_ms = _benchmark(m, n, k, tp_size, provider)
        return ms * 1000, max_ms * 1000, min_ms * 1000  # convert to ms

    return benchmark
```
**EN:** `get_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `benchmark` to the caller. Notable calls include `create_benchmark_configs`, `triton.testing.perf_report`, `_benchmark`.
**CN:** `get_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `benchmark`。其中较关键的调用包括 `create_benchmark_configs`, `triton.testing.perf_report`, `_benchmark`。

### Lines 190-250: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    if not torch.cuda.is_available() or torch.cuda.get_device_capability()[0] != 10:
        print("Skipping benchmark because the device is not supported")
        exit(0)

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save-path",
        type=str,
        default="./configs/benchmark_ops/dsv3_router_gemm/",
        help="Path to save dsv3 router gemm benchmark results",
    )
    parser.add_argument(
        "--run-correctness",
        action="store_true",
        default=True,
        help="Whether to run correctness test",
    )
    parser.add_argument(
        "--tp-sizes",
        type=int,
        nargs="+",
        default=[1],
        help="List of tensor parallelism sizes to benchmark",
    )
    parser.add_argument(
        "--plot-friendly",
        action="store_true",
        default=False,
        help="Plot x axis as the config index instead of the m",
    )
    parser.add_argument(
        "--use-pdl",
        action="store_true",
        default=False,
        help="Use PDL if true.",
    )
    args = parser.parse_args()

    # Set random seed for reproducibility
    torch.manual_seed(0)
    torch.cuda.manual_seed(0)

    if args.use_pdl:
        os.environ["TRTLLM_ENABLE_PDL"] = "1"

    # Run correctness tests on a few examples
    if args.run_correctness:
        print("Running correctness tests...")
        for m, n, k, _ in create_benchmark_configs(args.tp_sizes):
            calculate_diff(m, n, k)

    # Get the benchmark function with the specified tp_size
    benchmark = (
        get_benchmark_plot_friendly(args.tp_sizes)
        if args.plot_friendly
        else get_benchmark(args.tp_sizes)
    )

    print(f"Running performance benchmark for TP sizes = {args.tp_sizes}...")
    benchmark.run(print_data=True, save_path=args.save_path)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `os`, `typing`
- **Third-party / 第三方依赖**: `torch`, `triton`, `flashinfer.gemm`, `sgl_kernel`
