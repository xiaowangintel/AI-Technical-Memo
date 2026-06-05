# bench_kimi_k2_moe_fused_gate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_kimi_k2_moe_fused_gate.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `Kimi k2 MoE fused gate` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `Kimi k2 MoE fused gate` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
````python
import itertools
import math
import os

import torch
import triton
import triton.language as tl
from sgl_kernel import kimi_k2_moe_fused_gate

from sglang.srt.layers.moe.topk import kimi_k2_biased_topk_impl
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 13-13: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 16-25: `kimi_k2_biased_topk_torch_compile` definition
````python
def kimi_k2_biased_topk_torch_compile(scores, bias, topk, routed_scaling_factor):
    """Original torch.compile-based implementation"""
    return kimi_k2_biased_topk_impl(
        scores,
        scores,
        bias,
        topk=topk,
        renormalize=True,
        routed_scaling_factor=routed_scaling_factor,
    )
````
**EN:** This section defines `kimi_k2_biased_topk_torch_compile` and implements the core logic associated with Kimi k2 biased topk torch compile. Docstring summary: Original torch.compile-based implementation
**CN:** 该部分定义 `kimi_k2_biased_topk_torch_compile`，并实现与 Kimi k2 biased topk torch compile 相关的核心逻辑。 文档字符串摘要：Original torch.compile-based implementation

### Lines 28-36: `kimi_k2_biased_topk_fused_kernel` definition
````python
def kimi_k2_biased_topk_fused_kernel(scores, bias, topk, routed_scaling_factor):
    """Our fused CUDA kernel implementation"""
    return kimi_k2_moe_fused_gate(
        scores,
        bias,
        topk=topk,
        renormalize=True,
        routed_scaling_factor=routed_scaling_factor,
    )
````
**EN:** This section defines `kimi_k2_biased_topk_fused_kernel` and implements the core logic associated with Kimi k2 biased topk fused kernel. Docstring summary: Our fused CUDA kernel implementation
**CN:** 该部分定义 `kimi_k2_biased_topk_fused_kernel`，并实现与 Kimi k2 biased topk fused kernel 相关的核心逻辑。 文档字符串摘要：Our fused CUDA kernel implementation

### Lines 37-39: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 40-62: Conditional logic
````python
if IS_CI:
    seq_length_range = [5000]  # Only test one sequence length in CI
else:
    seq_length_range = [
        1,
        8,
        16,
        32,
        64,
        128,
        256,
        512,
        1024,
        2048,
        4096,
        10000,
        15000,
        20000,
        25000,
        30000,
        35000,
        40000,
    ]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 64-64: Constants and configuration
````python
configs = [(sq,) for sq in seq_length_range]
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 67-106: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["seq_length"],
        x_vals=[list(_) for _ in configs],
        line_arg="provider",
        line_vals=["torch_compile", "fused_kernel"],
        line_names=["Torch Compile", "Fused Kernel"],
        styles=[("blue", "-"), ("red", "-")],
        ylabel="us",
        plot_name="kimi-k2-moe-fused-gate-performance",
        args={},
    )
)
def benchmark(seq_length, provider):
    dtype = torch.float32
    device = torch.device("cuda")
    num_experts, topk = 384, 6  # Kimi K2 configuration
    routed_scaling_factor = 2.872  # Kimi K2's routed scaling factor

    scores = torch.randn((seq_length, num_experts), device=device, dtype=dtype)
    bias = torch.rand(num_experts, device=device, dtype=dtype)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "torch_compile":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: kimi_k2_biased_topk_torch_compile(
                scores.clone(), bias.clone(), topk, routed_scaling_factor
            ),
            quantiles=quantiles,
        )
    elif provider == "fused_kernel":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: kimi_k2_biased_topk_fused_kernel(
                scores.clone(), bias.clone(), topk, routed_scaling_factor
            ),
            quantiles=quantiles,
        )

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 109-114: Command-line entry point
````python
if __name__ == "__main__":
    print("=" * 80)
    print("Benchmarking Kimi K2 MoE Fused Gate Performance")
    print("=" * 80)
    print("\nPerformance vs Sequence Length (384 experts, topk=6)")
    benchmark.run(print_data=True, save_path=".")
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `kimi_k2_biased_topk_torch_compile`, `kimi_k2_biased_topk_fused_kernel`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.layers.moe.topk`, `sglang.utils`
- **External / 外部**: `itertools`, `math`, `os`, `torch`, `triton`, `triton.language`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
