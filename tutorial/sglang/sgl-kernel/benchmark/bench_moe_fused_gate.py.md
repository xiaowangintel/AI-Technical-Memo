# bench_moe_fused_gate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_moe_fused_gate.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `MoE fused gate` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `MoE fused gate` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
````python
import itertools
import math
import os

import torch
import triton
import triton.language as tl
from sgl_kernel import moe_fused_gate

from sglang.srt.layers.moe.topk import biased_grouped_topk
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

### Lines 16-26: `biased_grouped_topk_org` definition
````python
def biased_grouped_topk_org(scores, bias, num_expert_group, topk_group, topk):
    return biased_grouped_topk(
        scores,
        scores,
        bias,
        topk=topk,
        renormalize=True,
        num_expert_group=num_expert_group,
        topk_group=topk_group,
        routed_scaling_factor=2.5,  # DeepSeek-R1 : 2.5, Kimi K2: 2.872
    )
````
**EN:** This section defines `biased_grouped_topk_org` and implements the core logic associated with biased grouped topk org.
**CN:** 该部分定义 `biased_grouped_topk_org`，并实现与 biased grouped topk org 相关的核心逻辑。

### Lines 29-32: `biased_grouped_topk_org_fuse_kernel` definition
````python
def biased_grouped_topk_org_fuse_kernel(
    scores, bias, num_expert_group, topk_group, topk
):
    return moe_fused_gate(scores, bias, num_expert_group, topk_group, topk)
````
**EN:** This section defines `biased_grouped_topk_org_fuse_kernel` and implements the core logic associated with biased grouped topk org fuse kernel.
**CN:** 该部分定义 `biased_grouped_topk_org_fuse_kernel`，并实现与 biased grouped topk org fuse kernel 相关的核心逻辑。

### Lines 33-35: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 36-39: Conditional logic
````python
if IS_CI:
    seq_length_range = [5000]  # Only test one sequence length in CI
else:
    seq_length_range = [5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 41-41: Constants and configuration
````python
configs = [(sq,) for sq in seq_length_range]
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 44-82: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["seq_length"],
        x_vals=[list(_) for _ in configs],
        line_arg="provider",
        line_vals=["original", "kernel"],
        line_names=["Original", "SGL Kernel"],
        styles=[("blue", "-"), ("red", "-")],
        ylabel="us",
        plot_name="moe-fused-gate-performance",
        args={},
    )
)
def benchmark(seq_length, provider):
    dtype = torch.float32
    device = torch.device("cuda")
    num_experts, num_expert_group, topk_group, topk = 256, 8, 4, 8

    scores = torch.randn((seq_length, num_experts), device=device, dtype=dtype)
    bias = torch.rand(num_experts, device=device, dtype=dtype)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "original":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: biased_grouped_topk_org(
                scores.clone(), bias.clone(), num_expert_group, topk_group, topk
            ),
            quantiles=quantiles,
        )
    elif provider == "kernel":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: biased_grouped_topk_org_fuse_kernel(
                scores.clone(), bias.clone(), num_expert_group, topk_group, topk
            ),
            quantiles=quantiles,
        )

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 85-86: Command-line entry point
````python
if __name__ == "__main__":
    benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `biased_grouped_topk_org`, `biased_grouped_topk_org_fuse_kernel`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.layers.moe.topk`, `sglang.utils`
- **External / 外部**: `itertools`, `math`, `os`, `torch`, `triton`, `triton.language`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
