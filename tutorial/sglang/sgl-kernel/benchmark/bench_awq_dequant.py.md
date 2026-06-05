# bench_awq_dequant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_awq_dequant.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `AWQ dequantization` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `AWQ dequantization` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup
````python
import itertools
import os
from typing import List, Tuple

import torch
import triton
import triton.testing
from sgl_kernel import awq_dequantize

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-12: Comments and local context
````python

# Optional vLLM import
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 13-19: Optional dependency handling
````python
try:
    from vllm import _custom_ops as ops

    VLLM_AVAILABLE = True
except ImportError:
    ops = None
    VLLM_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 21-21: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 24-30: `vllm_awq_dequantize` definition
````python
def vllm_awq_dequantize(
    qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor
) -> Tuple[torch.Tensor, torch.Tensor]:
    if not VLLM_AVAILABLE:
        # Fallback to SGLang implementation
        return sglang_awq_dequantize(qweight, scales, qzeros)
    return ops.awq_dequantize(qweight, scales, qzeros, 0, 0, 0)
````
**EN:** This section defines `vllm_awq_dequantize` and implements the core logic associated with vllm AWQ dequantize.
**CN:** 该部分定义 `vllm_awq_dequantize`，并实现与 vllm AWQ dequantize 相关的核心逻辑。

### Lines 33-37: `sglang_awq_dequantize` definition
````python
def sglang_awq_dequantize(
    qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor
) -> Tuple[torch.Tensor, torch.Tensor]:

    return awq_dequantize(qweight, scales, qzeros)
````
**EN:** This section defines `sglang_awq_dequantize` and implements the core logic associated with sglang AWQ dequantize.
**CN:** 该部分定义 `sglang_awq_dequantize`，并实现与 sglang AWQ dequantize 相关的核心逻辑。

### Lines 40-76: `calculate_diff` definition
````python
def calculate_diff(qweight_row: int, qweight_col: int):
    """Calculate difference between VLLM and SGLang implementations."""
    device = torch.device("cuda")
    qweight = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (qweight_row, qweight_col),
        dtype=torch.int32,
        device=device,
    )
    group_size = qweight_row
    scales_row = qweight_row // group_size
    scales_col = qweight_col * 8
    scales = torch.rand(scales_row, scales_col, dtype=torch.float16, device=device)
    qzeros = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (scales_row, qweight_col),
        dtype=torch.int32,
        device=device,
    )

    if not VLLM_AVAILABLE:
        print("⚠️ vLLM not available, skipping comparison")
        return

    vllm_out = vllm_awq_dequantize(qweight, scales, qzeros)
    sglang_out = sglang_awq_dequantize(qweight, scales, qzeros)

    output_diff = torch.abs(vllm_out.float() - sglang_out.float()).mean().item()

    if torch.allclose(
        vllm_out.to(torch.float32), sglang_out.to(torch.float32), rtol=1e-3, atol=1e-5
    ):
        print("✅ All implementations match")
    else:
        print("❌ Implementations differ")
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module. Docstring summary: Calculate difference between VLLM and SGLang implementations.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。 文档字符串摘要：Calculate difference between VLLM and SGLang implementations.

### Lines 77-79: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 80-85: Conditional logic
````python
if IS_CI:
    qweight_row_range = [128]  # Single row size for CI
    qweight_cols_range = [16]  # Single column size for CI
else:
    qweight_row_range = [3584, 18944, 128, 256, 512, 1024]
    qweight_cols_range = [448, 576, 4736, 16, 32, 64, 128]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 87-87: Constants and configuration
````python
configs = list(itertools.product(qweight_row_range, qweight_cols_range))
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 90-140: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["qweight_row", "qweight_col"],
        x_vals=configs,
        line_arg="provider",
        line_vals=["vllm", "sglang"] if VLLM_AVAILABLE else ["sglang"],
        line_names=["VLLM", "SGL Kernel"] if VLLM_AVAILABLE else ["SGL Kernel"],
        styles=[("blue", "-"), ("green", "-")] if VLLM_AVAILABLE else [("green", "-")],
        ylabel="us",
        plot_name="awq-dequantize-performance",
        args={},
    )
)
def benchmark(qweight_row, qweight_col, provider):
    dtype = torch.float16
    device = torch.device("cuda")
    qweight = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (qweight_row, qweight_col),
        dtype=torch.int32,
        device=device,
    )
    group_size = qweight_row
    scales_row = qweight_row // group_size
    scales_col = qweight_col * 8
    scales = torch.rand(scales_row, scales_col, dtype=torch.float16, device=device)
    qzeros = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (scales_row, qweight_col),
        dtype=torch.int32,
        device=device,
    )

    quantiles = [0.5, 0.2, 0.8]

    if provider == "vllm":
        if not VLLM_AVAILABLE:
            return (0, 0, 0)
        fn = lambda: vllm_awq_dequantize(
            qweight.clone(), scales.clone(), qzeros.clone()
        )
    elif provider == "sglang":
        fn = lambda: sglang_awq_dequantize(
            qweight.clone(), scales.clone(), qzeros.clone()
        )

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles)

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 143-151: Command-line entry point
````python
if __name__ == "__main__":
    # Simplify for CI environment
    if IS_CI:
        qweight_row, qweight_col = 128, 16  # Smaller values for CI
    else:
        qweight_row, qweight_col = 3584, 448

    calculate_diff(qweight_row=qweight_row, qweight_col=qweight_col)
    benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `vllm_awq_dequantize`, `sglang_awq_dequantize`, `calculate_diff`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.utils`
- **External / 外部**: `itertools`, `os`, `torch`, `triton`, `triton.testing`, `typing`, `vllm`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
