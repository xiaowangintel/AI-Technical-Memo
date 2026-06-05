# bench_dsv3_fused_a_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_dsv3_fused_a_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `DeepSeek-V3 fused a GEMM` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `DeepSeek-V3 fused a GEMM` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
````python
import argparse

import torch
import torch.nn.functional as F
import triton
import triton.testing
from sgl_kernel import dsv3_fused_a_gemm

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-11: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 12-13: Comments and local context
````python

# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 14-19: Conditional logic
````python
if IS_CI:
    num_tokens_vals = [1]  # Only test 1 value in CI
    line_vals = ["sgl-kernel"]  # Only test sgl-kernel implementation in CI
else:
    num_tokens_vals = [i + 1 for i in range(16)]  # Test 1-16 in full mode
    line_vals = ["torch", "sgl-kernel"]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 22-66: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["num_tokens"],
        x_vals=num_tokens_vals,
        x_log=False,
        line_arg="impl",
        line_vals=line_vals,
        line_names=(
            ["torch (bf16)", "dsv3_fused_a_gemm"]
            if not IS_CI
            else ["dsv3_fused_a_gemm"]
        ),
        styles=[("blue", "-"), ("orange", "-")] if not IS_CI else [("orange", "-")],
        ylabel="TFLOPs",
        plot_name="bf16 dsv3 fused a GEMM throughput",
        args={},
    )
)
def benchmark(num_tokens, impl):
    kHdIn = 7168
    kHdOut = 2112
    M, K, N = num_tokens, kHdIn, kHdOut

    mat_a = torch.randn((M, K), dtype=torch.bfloat16, device="cuda").contiguous()
    mat_b = torch.randn((N, K), dtype=torch.bfloat16, device="cuda").transpose(0, 1)

    quantiles = [0.5, 0.2, 0.8]

    if impl == "torch":

        def runner():
            F.linear(mat_a, mat_b.T)

    elif impl == "sgl-kernel":

        def runner():
            dsv3_fused_a_gemm(mat_a, mat_b)

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(runner, quantiles=quantiles)

    def tflops(t_ms):
        flops = 2 * M * K * N
        return flops / (t_ms * 1e-3) / 1e12

    return tflops(ms), tflops(max_ms), tflops(min_ms)
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 69-73: Command-line entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    args = parser.parse_args()

    benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.utils`
- **External / 外部**: `argparse`, `torch`, `torch.nn.functional`, `triton`, `triton.testing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
