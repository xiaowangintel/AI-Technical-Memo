# benchmark_deepgemm_fp8_gemm_blackwell.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/deepseek/benchmark_deepgemm_fp8_gemm_blackwell.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels deepseek deepgemm fp8 gemm. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels deepseek deepgemm fp8 gemm 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and setup / 导入与初始化
```python
import argparse
from typing import Tuple

import torch
import triton
from deep_gemm import ceil_div
from flashinfer.gemm import gemm_fp8_nt_groupwise

from sglang.benchmark.bench_utils import run_bench
from sglang.srt.layers.quantization.fp8_kernel import (
    sglang_per_token_group_quant_fp8,
    w8a8_block_fp8_matmul_deepgemm,
)
from sglang.srt.layers.quantization.fp8_utils import requant_weight_ue8m0

BLOCK_SIZE = 128
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 19-32: Function `per_block_cast_to_fp8` / 函数 `per_block_cast_to_fp8`
```python
def per_block_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    assert BLOCK_SIZE == 128
    m, n = x.shape
    x_padded = torch.zeros(
        (ceil_div(m, 128) * 128, ceil_div(n, 128) * 128), dtype=x.dtype, device=x.device
    )
    x_padded[:m, :n] = x
    x_view = x_padded.view(-1, 128, x_padded.size(1) // 128, 128)
    x_amax = x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp(1e-4)
    x_scaled = (x_view * (448.0 / x_amax)).to(torch.float8_e4m3fn)
    return x_scaled.view_as(x_padded)[:m, :n].contiguous(), (x_amax / 448.0).view(
        x_view.size(0), x_view.size(2)
    )
```
**EN:** `per_block_cast_to_fp8` is a function that prepares tensors and invokes GPU kernels. It returns `(x_scaled.view_as(x_padded)[:m, :n].contiguous(), (x_amax / 448.0).view(x_view.size(0), x_view.si...` to the caller. Notable calls include `torch.zeros`, `x_padded.view`, `x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp`.
**CN:** `per_block_cast_to_fp8` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(x_scaled.view_as(x_padded)[:m, :n].contiguous(), (x_amax / 448.0).view(x_view.size(0), x_view.si...`。其中较关键的调用包括 `torch.zeros`, `x_padded.view`, `x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp`。

### Lines 35-65: Function `get_weight_shapes` / 函数 `get_weight_shapes`
```python
def get_weight_shapes(tp_size):
    # cannot TP
    total = [
        (512 + 64, 7168),
        ((128 + 64) * 128, 7168),
        (128 * (128 + 128), 512),
        (7168, 16384),
        (7168, 18432),
    ]
    # N can TP
    n_tp = [
        (18432 * 2, 7168),
        ((128 + 64) * 128, 7168),
        (128 * (128 + 128), 512),
        (24576, 1536),
        (4096, 7168),
    ]
    # K can TP
    k_tp = [(7168, 18432), (7168, 16384), (7168, 2048)]

    weight_shapes = []
    for t in total:
        weight_shapes.append(t)
    for n_t in n_tp:
        new_t = (n_t[0] // tp_size, n_t[1])
        weight_shapes.append(new_t)
    for k_t in k_tp:
        new_t = (k_t[0], k_t[1] // tp_size)
        weight_shapes.append(new_t)

    return weight_shapes
```
**EN:** `get_weight_shapes` is a function that implements the core logic for this scope. It returns `weight_shapes` to the caller. Notable calls include `weight_shapes.append`.
**CN:** `get_weight_shapes` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `weight_shapes`。其中较关键的调用包括 `weight_shapes.append`。

### Lines 68-77: Function `create_benchmark_configs` / 函数 `create_benchmark_configs`
```python
def create_benchmark_configs(tp_size):
    configs = []
    weight_shapes = get_weight_shapes(tp_size)
    batch_sizes = [8, 16, 32, 64, 128, 256, 1024, 2048, 4096]

    for n, k in weight_shapes:
        for m in batch_sizes:
            configs.append((m, n, k, tp_size))

    return configs
```
**EN:** `create_benchmark_configs` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `configs` to the caller. Notable calls include `get_weight_shapes`, `configs.append`.
**CN:** `create_benchmark_configs` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `configs`。其中较关键的调用包括 `get_weight_shapes`, `configs.append`。

### Lines 80-95: Function `fp8_gemm_flashinfer` / 函数 `fp8_gemm_flashinfer`
```python
def fp8_gemm_flashinfer(
    x_fp8: torch.Tensor,
    x_scale: torch.Tensor,
    y_fp8: torch.Tensor,
    y_scale: torch.Tensor,
):
    """Flashinfer implementation of FP8 GEMM"""
    output = gemm_fp8_nt_groupwise(
        x_fp8,
        y_fp8,
        x_scale,
        y_scale,
        out_dtype=torch.bfloat16,
        backend="trtllm",
    )
    return output
```
**EN:** `fp8_gemm_flashinfer` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Flashinfer implementation of FP8 GEMM It returns `output` to the caller. Notable calls include `gemm_fp8_nt_groupwise`.
**CN:** `fp8_gemm_flashinfer` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `output`。其中较关键的调用包括 `gemm_fp8_nt_groupwise`。

### Lines 98-109: Function `fp8_gemm_deepgemm_blackwell` / 函数 `fp8_gemm_deepgemm_blackwell`
```python
def fp8_gemm_deepgemm_blackwell(
    x_fp8: torch.Tensor,
    x_scale: torch.Tensor,
    y_fp8: torch.Tensor,
    y_scale: torch.Tensor,
):
    """DeepGEMM implementation of FP8 GEMM"""
    block_size = [BLOCK_SIZE, BLOCK_SIZE]
    output = w8a8_block_fp8_matmul_deepgemm(
        x_fp8, y_fp8, x_scale, y_scale, block_size, output_dtype=torch.bfloat16
    )
    return output
```
**EN:** `fp8_gemm_deepgemm_blackwell` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: DeepGEMM implementation of FP8 GEMM It returns `output` to the caller. Notable calls include `w8a8_block_fp8_matmul_deepgemm`.
**CN:** `fp8_gemm_deepgemm_blackwell` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `output`。其中较关键的调用包括 `w8a8_block_fp8_matmul_deepgemm`。

### Lines 112-133: Function `check_accuracy` / 函数 `check_accuracy`
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

### Lines 136-175: Function `calculate_diff` / 函数 `calculate_diff`
```python
def calculate_diff(m: int, n: int, k: int):
    x = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
    y = torch.randn((n, k), device="cuda", dtype=torch.bfloat16)

    y_fp8, y_scale = per_block_cast_to_fp8(y)
    x_fp8, x_scale = sglang_per_token_group_quant_fp8(
        x, BLOCK_SIZE, column_major_scales=True
    )
    out_flashinfer = fp8_gemm_flashinfer(
        x_fp8,
        x_scale,
        y_fp8,
        y_scale,
    )

    dg_x_fp8, dg_x_scale = sglang_per_token_group_quant_fp8(
        x,
        BLOCK_SIZE,
        column_major_scales=True,
        scale_tma_aligned=True,
        scale_ue8m0=True,
    )
    # We can directly quantize y here, but to mimic the behavior of the actual
    # implementations, we requant it here.
    dg_y_fp8, dg_y_scale = requant_weight_ue8m0(
        y_fp8, y_scale, [BLOCK_SIZE, BLOCK_SIZE]
    )
    out_deepgemm = fp8_gemm_deepgemm_blackwell(
        dg_x_fp8, dg_x_scale, dg_y_fp8, dg_y_scale
    )

    print(f"Shape m={m}, n={n}, k={k}:")
    print(f"Flashinfer output: {out_flashinfer[0, 0:5]}")
    print(f"DeepGEMM output: {out_deepgemm[0, 0:5]}")

    flashinfer_deepgemm_match = check_accuracy(
        out_flashinfer, out_deepgemm, 0.1, 0.6, 0.95
    )
    print("Correctness check:")
    print(f"  - Flashinfer vs DeepGEMM: {'✅' if flashinfer_deepgemm_match else '❌'}")
```
**EN:** `calculate_diff` is a function that prepares tensors and invokes GPU kernels, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. Notable calls include `torch.randn`, `per_block_cast_to_fp8`, `sglang_per_token_group_quant_fp8`.
**CN:** `calculate_diff` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `torch.randn`, `per_block_cast_to_fp8`, `sglang_per_token_group_quant_fp8`。

### Lines 178-228: Function `_benchmark` / 函数 `_benchmark`
```python
def _benchmark(m, n, k, tp_size, provider):
    print(f"Shape (m={m}, n={n}, k={k}, tp={tp_size}), Provider: {provider}")
    x = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
    y = torch.randn((n, k), device="cuda", dtype=torch.bfloat16)

    # Preprocess data before benchmarking
    y_fp8, y_scale = per_block_cast_to_fp8(y)
    x_fp8, x_scale = sglang_per_token_group_quant_fp8(
        x, BLOCK_SIZE, column_major_scales=True
    )
    dg_x_fp8, dg_x_scale = sglang_per_token_group_quant_fp8(
        x,
        BLOCK_SIZE,
        column_major_scales=True,
        scale_tma_aligned=True,
        scale_ue8m0=True,
    )
    dg_y_fp8, dg_y_scale = requant_weight_ue8m0(
        y_fp8, y_scale, [BLOCK_SIZE, BLOCK_SIZE]
    )

    quantiles = (0.5, 0.2, 0.8)

    if provider == "deepgemm":
        ms, min_ms, max_ms = run_bench(
            lambda: fp8_gemm_deepgemm_blackwell(
                dg_x_fp8,
                dg_x_scale,
                dg_y_fp8,
                dg_y_scale,
            ),
            quantiles=quantiles,
        )
    elif provider == "flashinfer":
        ms, min_ms, max_ms = run_bench(
            lambda: fp8_gemm_flashinfer(
                x_fp8,
                x_scale,
                y_fp8,
                y_scale,
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
**EN:** `_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `(ms, max_ms, min_ms)` to the caller. Notable calls include `print`, `torch.randn`, `per_block_cast_to_fp8`.
**CN:** `_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(ms, max_ms, min_ms)`。其中较关键的调用包括 `print`, `torch.randn`, `per_block_cast_to_fp8`。

### Lines 231-253: Function `get_benchmark_plot_friendly` / 函数 `get_benchmark_plot_friendly`
```python
def get_benchmark_plot_friendly(tp_size):
    all_configs = create_benchmark_configs(tp_size)
    x_vals = list(range(len(all_configs)))

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["cfg_id"],
            x_vals=x_vals,
            line_arg="provider",
            line_vals=["deepgemm", "flashinfer"],
            line_names=["DeepGEMM", "Flashinfer"],
            styles=[("blue", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"fp8-gemm-performance-comparison-tp{tp_size}",
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

### Lines 256-276: Function `get_benchmark` / 函数 `get_benchmark`
```python
def get_benchmark(tp_size):
    all_configs = create_benchmark_configs(tp_size)

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["m", "n", "k", "tp_size"],
            x_vals=[list(config) for config in all_configs],
            line_arg="provider",
            line_vals=["deepgemm", "flashinfer"],
            line_names=["DeepGEMM", "Flashinfer"],
            styles=[("blue", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"fp8-gemm-performance-comparison-tp{tp_size}",
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

### Lines 279-330: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    if not torch.cuda.is_available() or torch.cuda.get_device_capability()[0] != 10:
        print("Skipping benchmark because the device is not supported")
        exit(0)

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save-path",
        type=str,
        default="./configs/benchmark_ops/fp8_gemm/",
        help="Path to save fp8 gemm benchmark results",
    )
    parser.add_argument(
        "--run-correctness",
        action="store_true",
        default=True,
        help="Whether to run correctness test",
    )
    parser.add_argument(
        "--tp-size",
        type=int,
        default=1,
        help="Tensor parallelism size to benchmark (default: 1)",
    )
    parser.add_argument(
        "--plot-friendly",
        action="store_true",
        default=False,
        help="Plot x axis as the config index instead of the m",
    )
    args = parser.parse_args()

    # Set random seed for reproducibility
    torch.manual_seed(0)
    torch.cuda.manual_seed(0)

    # Run correctness tests on a few examples
    if args.run_correctness:
        print("Running correctness tests...")
        calculate_diff(64, 512, 7168)  # Small test
        calculate_diff(64, 7168, 16384)  # Medium test
        calculate_diff(64, 18432, 7168)  # Large test

    # Get the benchmark function with the specified tp_size
    benchmark = (
        get_benchmark_plot_friendly(args.tp_size)
        if args.plot_friendly
        else get_benchmark(args.tp_size)
    )

    print(f"Running performance benchmark for TP size = {args.tp_size}...")
    benchmark.run(print_data=True, save_path=args.save_path)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `typing`
- **Third-party / 第三方依赖**: `torch`, `triton`, `deep_gemm`, `flashinfer.gemm`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`
