# bench_qserve_w4a8_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_qserve_w4a8_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `QServe w4a8 GEMM` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `QServe w4a8 GEMM` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup
````python
import argparse
import copy
import itertools
import os

import torch
import triton
from sgl_kernel import (
    int8_scaled_mm,
    qserve_w4a8_per_chn_gemm,
    qserve_w4a8_per_group_gemm,
)

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 16-16: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 19-20: `to_int8` definition
````python
def to_int8(tensor: torch.Tensor) -> torch.Tensor:
    return torch.round(tensor.clamp(min=-128, max=127)).to(dtype=torch.int8)
````
**EN:** This section defines `to_int8` and implements the core logic associated with to INT8.
**CN:** 该部分定义 `to_int8`，并实现与 to INT8 相关的核心逻辑。

### Lines 23-70: Constants and configuration
````python
WEIGHT_SHAPES = {
    "meta-llama/Llama-3.1-8B-Instruct": [
        ([4096, 6144], 1),
        ([4096, 4096], 0),
        ([4096, 28672], 1),
        ([14336, 4096], 0),
    ],
    "meta-llama/Llama-3.3-70B-Instruct": [
        ([8192, 10240], 1),
        ([8192, 8192], 0),
        ([8192, 57344], 1),
        ([28672, 8192], 0),
    ],
    "mistralai/Mistral-Large-Instruct-2407": [
        ([12288, 14336], 1),
        ([12288, 12288], 0),
        ([12288, 57344], 1),
        ([28672, 12288], 0),
    ],
    "Qwen/Qwen2.5-7B-Instruct": [
        ([3584, 4608], 1),
        ([3584, 3584], 0),
        ([3584, 37888], 1),
        ([18944, 3584], 0),
    ],
    "Qwen/Qwen2.5-32B-Instruct": [
        ([5120, 7168], 1),
        ([5120, 5120], 0),
        ([5120, 55296], 1),
        ([27648, 5120], 0),
    ],
    "Qwen/Qwen2.5-72B-Instruct": [
        ([8192, 10240], 1),
        ([8192, 8192], 0),
        ([8192, 59136], 1),
        ([29568, 8192], 0),
    ],
    "deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct": [
        ([2048, 3072], 1),
        ([2048, 4096], 1),
        ([2048, 2048], 0),
        ([2048, 576], 0),
        ([2048, 21888], 1),
        ([10944, 2048], 0),
        ([2048, 2816], 1),
        ([1408, 2048], 0),
    ],
}
````
**EN:** This block defines shared constants or configuration values such as `WEIGHT_SHAPES`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `WEIGHT_SHAPES`），供后续函数或控制流程复用。

### Lines 71-73: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 74-77: Conditional logic
````python
if IS_CI:
    batch_sizes = [1, 16]  # Simplified for CI
else:
    batch_sizes = [1, 16, 32, 64, 128, 256, 512, 1024, 2048]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 80-166: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size"],
        x_vals=batch_sizes,
        x_log=False,
        line_arg="provider",
        line_vals=["FP16", "W8A8", "Qserve_W4A8_Per_Channel", "Qserve_W4A8_Per_Group"],
        line_names=["FP16", "W8A8", "Qserve_W4A8_Per_Channel", "Qserve_W4A8_Per_Group"],
        styles=[("blue", "-"), ("orange", "-"), ("green", "-"), ("red", "-")],
        ylabel="ms",
        plot_name="FP16_vs_W8A8_vs_Qserve_W4A8_GEMM",
        args={},
    )
)
def benchmark(batch_size, provider, N, K):
    M = batch_size
    # For W8A8
    a = to_int8(torch.randn((M, K), device="cuda") * 5)
    b = to_int8(torch.randn((N, K), device="cuda").t() * 5)
    a_fp16 = a.to(torch.float16)
    b_fp16 = b.to(torch.float16)
    scale_a = torch.randn((M,), device="cuda", dtype=torch.float32)
    scale_b = torch.randn((N,), device="cuda", dtype=torch.float32)

    # For Qserve W4A8 per channel
    a_qserve_chn = a
    # two int4s pack into one int8
    b_qserve_chn = to_int8(torch.randn((N, K // 2), device="cuda") * 5)
    # b_qserve_chn = b.t().contiguous()
    scale_a_qserve_chn = scale_a.to(torch.float16)
    scale_b_qserve_chn = scale_b.to(torch.float16)
    szero_b_qserve_chn = torch.randn((N,), device="cuda", dtype=torch.float16)
    a_sum_qserve_chn = torch.randn((M,), device="cuda", dtype=torch.float16)

    # For Qserve W4A8 per group
    group_size = 128
    assert K % group_size == 0, "K must be divisible by group_size"
    a_qserve_group = a
    # two int4s pack into one int8
    b_qserve_group = to_int8(torch.randn((N, K // 2), device="cuda") * 5)
    # b_qserve_group = b.t().contiguous()
    scale_a_qserve_group = scale_a.to(torch.float16)
    scale_b_qserve_group = scale_b.to(torch.float16)
    scale_i8_b_qserve_group = to_int8(
        torch.randn((K // group_size, N), device="cuda", dtype=torch.float16)
    )
    zero_i8_b_qserve_group = to_int8(
        torch.randn((K // group_size, N), device="cuda", dtype=torch.float16)
    )

    quantiles = [0.5, 0.2, 0.8]
    if provider == "FP16":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: torch.matmul(a_fp16, b_fp16),
            quantiles=quantiles,
        )
    if provider == "W8A8":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: int8_scaled_mm(a, b, scale_a, scale_b, torch.float16),
            quantiles=quantiles,
        )
    if provider == "Qserve_W4A8_Per_Channel":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: qserve_w4a8_per_chn_gemm(
                a_qserve_chn,
                b_qserve_chn,
                scale_b_qserve_chn,
                scale_a_qserve_chn,
                szero_b_qserve_chn,
                a_sum_qserve_chn,
            ),
            quantiles=quantiles,
        )
    if provider == "Qserve_W4A8_Per_Group":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: qserve_w4a8_per_group_gemm(
                a_qserve_group,
                b_qserve_group,
                zero_i8_b_qserve_group,
                scale_i8_b_qserve_group,
                scale_b_qserve_group,
                scale_a_qserve_group,
            ),
            quantiles=quantiles,
        )

    return ms, max_ms, min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. It also performs explicit assertions or shape checks before continuing. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 169-178: `prepare_shapes` definition
````python
def prepare_shapes(args):
    KN_model_names = []
    models_tps = list(itertools.product(args.models, args.tp_sizes))
    for model, tp_size in models_tps:
        assert model in WEIGHT_SHAPES
        for KN, tp_split_dim in copy.deepcopy(WEIGHT_SHAPES[model]):
            KN[tp_split_dim] = KN[tp_split_dim] // tp_size
            KN.append(model)
            KN_model_names.append(KN)
    return KN_model_names
````
**EN:** This section defines `prepare_shapes`. It prepares the `shapes` path used by the module. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `prepare_shapes`。它负责准备模块中与 `shapes` 相关的处理路径。 它还会在继续执行前进行显式断言或形状检查。

### Lines 181-214: Command-line entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=["meta-llama/Llama-3.1-8B-Instruct"],
        help="List of models to benchmark",
    )
    parser.add_argument(
        "--tp-sizes",
        nargs="+",
        type=int,
        default=[1],
        help="List of tensor parallel sizes",
    )
    args = parser.parse_args()

    # Skip in CI environment
    if IS_CI:
        print("Skipping QServe W4A8 GEMM benchmark in CI environment")
        print("QServe operations may have compatibility issues in CI")
    else:
        KN_model_names = prepare_shapes(args)

        for K, N, model_name in KN_model_names:
            print(f"{model_name} N={N} K={K}: ")
            benchmark.run(
                print_data=True,
                N=N,
                K=K,
            )

        print("Benchmark finished!")
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `to_int8`, `benchmark`, `prepare_shapes`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.utils`
- **External / 外部**: `argparse`, `copy`, `itertools`, `os`, `torch`, `triton`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
