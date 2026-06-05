# bench_fp4_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/quantization/bench_fp4_quant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels quantization fp4 quant. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels quantization fp4 quant 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and setup / 导入与初始化
```python
"""Benchmark FP4 quantize: sglang jit_kernel vs flashinfer.

Compares ``sglang.jit_kernel.nvfp4.scaled_fp4_quant`` against
``flashinfer.fp4_quantize`` over a sweep of (M, K) shapes.

Timing uses ``flashinfer.testing.bench_gpu_time`` (CUDA-graph based with
rotating-buffer cold-L2).
"""

import argparse
import itertools

import numpy as np
import torch
from flashinfer import fp4_quantize as flashinfer_fp4_quantize
from flashinfer.testing import bench_gpu_time

from sglang.jit_kernel.nvfp4 import scaled_fp4_quant

Ms = [1, 8, 32, 128, 512, 1024, 2048, 4096, 8192, 16384, 32768]
Ks = [128, 256, 384, 512, 768, 1024, 1536, 2048, 3072, 4096, 5120, 6144, 8192, 16384]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 24-32: Function `_bench` / 函数 `_bench`
```python
def _bench(fn, input_args) -> float:
    times = bench_gpu_time(
        fn=fn,
        input_args=input_args,
        use_cuda_graph=True,
        dry_run_time_ms=25,
        repeat_time_ms=100,
    )
    return float(np.median(times))
```
**EN:** `_bench` is a function that prepares tensors and invokes GPU kernels. It returns `float(np.median(times))` to the caller. Notable calls include `bench_gpu_time`, `float`, `np.median`.
**CN:** `_bench` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `float(np.median(times))`。其中较关键的调用包括 `bench_gpu_time`, `float`, `np.median`。

### Lines 35-48: Function `benchmark` / 函数 `benchmark`
```python
def benchmark(M: int, K: int, dtype: torch.dtype, device: str):
    x = torch.randn(M, K, device=device, dtype=dtype)
    global_scale = torch.ones(1, device=device, dtype=torch.float32)

    sglang_ms = _bench(
        lambda x, gs: scaled_fp4_quant(x, gs),
        input_args=(x, global_scale),
    )
    flashinfer_ms = _bench(
        lambda x, gs: flashinfer_fp4_quantize(x, gs, backend="cute-dsl"),
        input_args=(x, global_scale),
    )

    return sglang_ms, flashinfer_ms
```
**EN:** `benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `(sglang_ms, flashinfer_ms)` to the caller. Notable calls include `torch.randn`, `torch.ones`, `_bench`.
**CN:** `benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `(sglang_ms, flashinfer_ms)`。其中较关键的调用包括 `torch.randn`, `torch.ones`, `_bench`。

### Lines 51-91: Function `plot_speedup` / 函数 `plot_speedup`
```python
def plot_speedup(rows, path):
    import matplotlib

    matplotlib.use("Agg")
    import matplotlib.pyplot as plt

    Ms_unique = sorted({int(r[0]) for r in rows})
    Ks_unique = sorted({int(r[1]) for r in rows})
    grid = np.full((len(Ms_unique), len(Ks_unique)), np.nan)
    m_idx = {m: i for i, m in enumerate(Ms_unique)}
    k_idx = {k: i for i, k in enumerate(Ks_unique)}
    for M, K, _, _, sp in rows:
        grid[m_idx[int(M)], k_idx[int(K)]] = float(sp)

    fig, ax = plt.subplots(figsize=(12, 8))
    vmax = max(2.0, np.nanmax(grid))
    vmin = min(0.5, np.nanmin(grid))
    im = ax.imshow(
        grid,
        aspect="auto",
        cmap="RdYlGn",
        vmin=vmin,
        vmax=vmax,
        origin="lower",
    )
    ax.set_xticks(range(len(Ks_unique)))
    ax.set_xticklabels(Ks_unique, rotation=45)
    ax.set_yticks(range(len(Ms_unique)))
    ax.set_yticklabels(Ms_unique)
    ax.set_xlabel("K")
    ax.set_ylabel("M")
    ax.set_title("Speedup: flashinfer / sglang  (>1 means sglang faster)")
    for i in range(len(Ms_unique)):
        for j in range(len(Ks_unique)):
            v = grid[i, j]
            if np.isfinite(v):
                ax.text(j, i, f"{v:.2f}", ha="center", va="center", fontsize=7)
    fig.colorbar(im, ax=ax, label="speedup")
    fig.tight_layout()
    fig.savefig(path, dpi=130)
    print(f"Saved plot to {path}")
```
**EN:** `plot_speedup` is a function that prepares tensors and invokes GPU kernels, computes evaluation scores and aggregate statistics, and plots or saves benchmark visualizations. Notable calls include `matplotlib.use`, `sorted`, `np.full`.
**CN:** `plot_speedup` 是一个函数，用于准备张量并调用 GPU 内核、计算评测分数与聚合统计结果、绘制或保存基准测试可视化结果。其中较关键的调用包括 `matplotlib.use`, `sorted`, `np.full`。

### Lines 94-133: Function `main` / 函数 `main`
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--dtype", choices=["bf16", "fp16"], default="bf16")
    parser.add_argument("--device", default="cuda")
    parser.add_argument("--csv", type=str, default=None)
    parser.add_argument("--plot", type=str, default=None)
    args = parser.parse_args()

    dtype = torch.bfloat16 if args.dtype == "bf16" else torch.float16

    rows = []
    header = (
        f"{'M':>8} {'K':>8} {'sglang(us)':>12} {'flashinfer(us)':>16} {'speedup':>10}"
    )
    print(header)
    print("-" * len(header))

    for M, K in itertools.product(Ms, Ks):
        try:
            sglang_ms, flashinfer_ms = benchmark(M, K, dtype, args.device)
        except Exception as e:
            print(f"{M:>8} {K:>8}  skipped: {e}")
            continue
        sglang_us = sglang_ms * 1e3
        flashinfer_us = flashinfer_ms * 1e3
        speedup = flashinfer_us / sglang_us
        print(
            f"{M:>8} {K:>8} {sglang_us:>12.3f} {flashinfer_us:>16.3f} {speedup:>10.3f}"
        )
        rows.append((M, K, sglang_us, flashinfer_us, speedup))

    if args.csv:
        with open(args.csv, "w") as f:
            f.write("M,K,sglang_us,flashinfer_us,speedup_flashinfer_over_sglang\n")
            for M, K, s, fi, sp in rows:
                f.write(f"{M},{K},{s:.6f},{fi:.6f},{sp:.6f}\n")
        print(f"Saved CSV to {args.csv}")

    if args.plot:
        plot_speedup(rows, args.plot)
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 136-137: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `itertools`
- **Third-party / 第三方依赖**: `numpy`, `torch`, `flashinfer`, `flashinfer.testing`, `matplotlib`, `matplotlib.pyplot`
- **Internal / 项目内部依赖**: `sglang.jit_kernel.nvfp4`
