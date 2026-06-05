# bench_fp8_blockwise_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_fp8_blockwise_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `FP8 blockwise GEMM` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `FP8 blockwise GEMM` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup
````python
import argparse
import copy
import itertools
import os

import deep_gemm
import torch
import triton
from deep_gemm.utils.layout import get_mn_major_tma_aligned_tensor
from sgl_kernel import fp8_blockwise_scaled_mm

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 13-14: Comments and local context
````python

# Optional vLLM import
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 15-21: Optional dependency handling
````python
try:
    from vllm._custom_ops import cutlass_scaled_mm as vllm_scaled_mm

    VLLM_AVAILABLE = True
except ImportError:
    vllm_scaled_mm = None
    VLLM_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 23-25: Imports and module setup
````python
from sglang.srt.layers.quantization.fp8_kernel import (
    w8a8_block_fp8_matmul_triton as w8a8_block_fp8_matmul,
)
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 27-27: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 30-66: `get_weight_shapes` definition
````python
def get_weight_shapes(args):
    models_tps = list(itertools.product(args.models, args.tp_sizes))
    # NOTE(HandH1998): The weight shapes only works for DeepSeek-V3. Modify them, if you tune for another different model.
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
    # only support Deepseek-V3
    SUPPORT_MODEL = ["deepseek-ai/DeepSeek-V3"]

    weight_shapes = []
    for model, tp_size in models_tps:
        assert model in SUPPORT_MODEL
        for t in total:
            new_t = [t[0], t[1], model]
            weight_shapes.append(new_t)
        for n_t in n_tp:
            new_t = [n_t[0] // tp_size, n_t[1], model]
            weight_shapes.append(new_t)
        for k_t in k_tp:
            new_t = [k_t[0], k_t[1] // tp_size, model]
            weight_shapes.append(new_t)
    return weight_shapes
````
**EN:** This section defines `get_weight_shapes`. It retrieves or computes the `weight shapes` path used by the module. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `get_weight_shapes`。它负责获取或计算模块中与 `weight shapes` 相关的处理路径。 它还会在继续执行前进行显式断言或形状检查。

### Lines 69-71: `cdiv` definition
````python
def cdiv(a: int, b: int) -> int:
    """Ceiling division."""
    return -(a // -b)
````
**EN:** This section defines `cdiv` and implements the core logic associated with cdiv. Docstring summary: Ceiling division.
**CN:** 该部分定义 `cdiv`，并实现与 cdiv 相关的核心逻辑。 文档字符串摘要：Ceiling division.

### Lines 74-88: `fp8_gemm_deepgemm` definition
````python
def fp8_gemm_deepgemm(
    x_fp8: torch.Tensor,
    x_scale: torch.Tensor,
    y_fp8: torch.Tensor,
    y_scale: torch.Tensor,
    m: int,
    n: int,
    k: int,
):
    """DeepGEMM implementation of FP8 GEMM"""
    out = torch.empty((m, n), device="cuda", dtype=torch.bfloat16)

    # Run DeepGEMM kernel
    deep_gemm.fp8_gemm_nt((x_fp8, x_scale), (y_fp8, y_scale), out)
    return out
````
**EN:** This section defines `fp8_gemm_deepgemm` and implements the core logic associated with FP8 GEMM deepgemm. Docstring summary: DeepGEMM implementation of FP8 GEMM
**CN:** 该部分定义 `fp8_gemm_deepgemm`，并实现与 FP8 GEMM deepgemm 相关的核心逻辑。 文档字符串摘要：DeepGEMM implementation of FP8 GEMM

### Lines 91-93: `scale_shape` definition
````python
def scale_shape(shape, group_shape):
    assert len(shape) == len(group_shape)
    return tuple(cdiv(shape[i], group_shape[i]) for i in range(len(group_shape)))
````
**EN:** This section defines `scale_shape` and implements the core logic associated with scale shape. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `scale_shape`，并实现与 scale shape 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 94-96: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 97-100: Conditional logic
````python
if IS_CI:
    batch_sizes = [1, 8]  # Simplified for CI
else:
    batch_sizes = [1, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 101-102: Comments and local context
````python

# Filter providers based on availability
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 103-105: Constants and configuration
````python
available_providers = ["sgl-kernel"]
available_names = ["sgl-kernel"]
available_styles = [("orange", "-")]
````
**EN:** This block defines shared constants or configuration values such as `available_providers`, `available_names`, `available_styles`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `available_providers`, `available_names`, `available_styles`），供后续函数或控制流程复用。

### Lines 107-110: Conditional logic
````python
if VLLM_AVAILABLE:
    available_providers.insert(0, "vllm")
    available_names.insert(0, "vllm")
    available_styles.insert(0, ("blue", "-"))
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 112-112: Top-level logic
````python
available_providers.append("triton")
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 113-113: Top-level logic
````python
available_names.append("sglang triton")
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 114-114: Top-level logic
````python
available_styles.append(("red", "-"))
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 115-116: Comments and local context
````python

# Add deepgemm if available
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 117-124: Optional dependency handling
````python
try:
    import deep_gemm

    available_providers.append("deepgemm")
    available_names.append("deepgemm")
    available_styles.append(("yellow", "-"))
except ImportError:
    pass
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 127-194: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size"],
        x_vals=batch_sizes,
        x_log=False,
        line_arg="provider",
        line_vals=available_providers,
        line_names=available_names,
        styles=available_styles,
        ylabel="GB/s",
        plot_name="fp8 blockwise scaled matmul",
        args={},
    )
)
def benchmark(batch_size, provider, N, K):
    M = batch_size
    fp8_info = torch.finfo(torch.float8_e4m3fn)
    fp8_max, fp8_min = fp8_info.max, fp8_info.min

    a_fp32 = (torch.rand(M, K, dtype=torch.float32, device="cuda") - 0.5) * 2 * fp8_max
    a_fp8 = a_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

    b_fp32 = (torch.rand(N, K, dtype=torch.float32, device="cuda") - 0.5) * 2 * fp8_max
    b_fp8 = b_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

    scale_a_group_shape = (1, 128)
    scale_b_group_shape = (128, 128)
    scale_a_shape = scale_shape(a_fp8.shape, scale_a_group_shape)
    scale_b_shape = scale_shape(b_fp8.shape, scale_b_group_shape)

    scale_a = torch.randn(scale_a_shape, device="cuda", dtype=torch.float32)
    scale_b = torch.randn(scale_b_shape, device="cuda", dtype=torch.float32)

    quantiles = [0.5, 0.2, 0.8]
    if provider == "sgl-kernel":
        scale_a = scale_a.t().contiguous().t()
        b_fp8, scale_b = b_fp8.t(), scale_b.t()
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: fp8_blockwise_scaled_mm(
                a_fp8, b_fp8, scale_a, scale_b, torch.float16
            ),
            quantiles=quantiles,
        )
    elif provider == "vllm":
        if not VLLM_AVAILABLE:
            return (0, 0, 0)
        scale_a = scale_a.t().contiguous().t()
        b_fp8, scale_b = b_fp8.t(), scale_b.t()
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: vllm_scaled_mm(a_fp8, b_fp8, scale_a, scale_b, torch.float16),
            quantiles=quantiles,
        )
    elif provider == "triton":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: w8a8_block_fp8_matmul(
                a_fp8, b_fp8, scale_a, scale_b, [128, 128], torch.float16
            ),
            quantiles=quantiles,
        )
    if provider == "deepgemm":
        scale_a_col_major = get_mn_major_tma_aligned_tensor(scale_a.clone())
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: fp8_gemm_deepgemm(
                a_fp8, scale_a_col_major, b_fp8, scale_b, M, N, K
            ),
            quantiles=quantiles,
        )
    return ms * 1000, max_ms * 1000, min_ms * 1000  # convert to ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 197-237: Command-line entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=["deepseek-ai/DeepSeek-V3"],
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

    # Simplify for CI environment
    if IS_CI:
        args.models = [args.models[0]]  # Use only first model
        args.tp_sizes = [args.tp_sizes[0]]  # Use only first TP size

    NK_model_names = get_weight_shapes(args)

    # Limit iterations in CI
    if IS_CI:
        NK_model_names = NK_model_names[:2]  # Only test first 2 shapes in CI

    for N, K, model_name in NK_model_names:
        if N % 128 != 0 or K % 128 != 0:
            print(f"Skip {N=}, {K=} now")
            continue
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
- **Primary symbols / 主要符号**: `get_weight_shapes`, `cdiv`, `fp8_gemm_deepgemm`, `scale_shape`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.utils`
- **External / 外部**: `argparse`, `copy`, `deep_gemm`, `deep_gemm.utils.layout`, `itertools`, `os`, `torch`, `triton`, `vllm._custom_ops`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
