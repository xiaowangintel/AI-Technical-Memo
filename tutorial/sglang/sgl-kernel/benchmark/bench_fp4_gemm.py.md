# bench_fp4_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_fp4_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `FP4 GEMM` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `FP4 GEMM` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup
````python
import argparse
import csv
import logging
from functools import partial
from typing import List, Tuple

import torch
import triton
from flashinfer import mm_fp4
from flashinfer.autotuner import autotune
from flashinfer.jit.core import logger as flashinfer_logger
from flashinfer.testing import bench_gpu_time
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 14-14: Top-level logic
````python
flashinfer_logger.setLevel(logging.ERROR)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 16-22: Imports and module setup
````python
from sglang.jit_kernel.nvfp4 import cutlass_scaled_fp4_mm, scaled_fp4_quant
from sglang.srt.utils import (
    get_device_capability,
    is_sm100_supported,
    is_sm120_supported,
)
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 24-29: Constants and configuration
````python
IS_CI = is_in_ci()

FLOAT4_E2M1_MAX = 6.0
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

DEEPSEEK_R1_MODEL = "deepseek-ai/DeepSeek-R1-0528-FP4"
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `DEEPSEEK_R1_MODEL`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`, `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `DEEPSEEK_R1_MODEL`），供后续函数或控制流程复用。

### Lines 30-32: Comments and local context
````python

# Weight shapes are in the format: ([K, N], TP_SPLIT_DIM)
# TP split dim 0 means split K by tp size; dim 1 means split N by tp size.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 33-91: Constants and configuration
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
    "Qwen/Qwen3.5-27B": [
        ([5120, 8192], 1),
        ([6144, 5120], 0),
        ([5120, 34816], 1),
        ([17408, 5120], 0),
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

DEEPSEEK_R1_WEIGHT_SHAPES = {
    4: [[1024, 3584], [7168, 256], [7168, 2304], [9216, 3584]],
    8: [[512, 3584], [7168, 128], [7168, 1152], [4608, 3584]],
}
````
**EN:** This block defines shared constants or configuration values such as `WEIGHT_SHAPES`, `DEEPSEEK_R1_WEIGHT_SHAPES`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `WEIGHT_SHAPES`, `DEEPSEEK_R1_WEIGHT_SHAPES`），供后续函数或控制流程复用。

### Lines 94-120: `get_weight_shapes` definition
````python
def get_weight_shapes(args) -> List[Tuple[int, int, str]]:
    shapes: List[Tuple[int, int, str]] = []
    for model in args.models:
        if model == DEEPSEEK_R1_MODEL:
            for tp_size in args.tp_sizes:
                if tp_size in DEEPSEEK_R1_WEIGHT_SHAPES:
                    selected = DEEPSEEK_R1_WEIGHT_SHAPES[tp_size]
                else:
                    selected = (
                        DEEPSEEK_R1_WEIGHT_SHAPES[4] + DEEPSEEK_R1_WEIGHT_SHAPES[8]
                    )
                for n, packed_k in selected:
                    shapes.append((n, packed_k, model))
            continue

        if model not in WEIGHT_SHAPES:
            raise ValueError(f"Unsupported model: {model}")
        for tp_size in args.tp_sizes:
            for k_n, tp_split_dim in WEIGHT_SHAPES[model]:
                k, n = k_n
                if tp_split_dim == 0:
                    k = k // tp_size
                else:
                    n = n // tp_size
                packed_k = k // 2
                shapes.append((n, packed_k, model))
    return shapes
````
**EN:** This section defines `get_weight_shapes`. It retrieves or computes the `weight shapes` path used by the module.
**CN:** 该部分定义 `get_weight_shapes`。它负责获取或计算模块中与 `weight shapes` 相关的处理路径。

### Lines 123-143: Conditional logic
````python
if IS_CI:
    batch_sizes = [1, 8]
else:
    batch_sizes = [
        1,
        2,
        4,
        8,
        16,
        32,
        64,
        128,
        256,
        512,
        1024,
        2048,
        3072,
        4096,
        8192,
        16384,
    ]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 146-147: `_run_mm_fp4` definition
````python
def _run_mm_fp4(a_fp4, b_fp4_T, a_sf, b_sf_T, alpha, dtype, res_fi, backend):
    return mm_fp4(a_fp4, b_fp4_T, a_sf, b_sf_T, alpha, dtype, res_fi, backend=backend)
````
**EN:** This section defines `_run_mm_fp4` and implements the core logic associated with run mm FP4.
**CN:** 该部分定义 `_run_mm_fp4`，并实现与 run mm FP4 相关的核心逻辑。

### Lines 150-397: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size"],
        x_vals=batch_sizes,
        x_log=False,
        line_arg="provider",
        line_vals=(
            ["sglang_cutlass", "cutlass", "cudnn", "trtllm", "cute-dsl", "auto"]
            if is_sm100_supported()
            else ["sglang_cutlass", "cutlass", "cudnn", "cute-dsl", "auto"]
        ),
        line_names=(
            [
                "sglang cutlass fp4",
                "flashinfer cutlass fp4",
                "cudnn fp4",
                "trtllm fp4",
                "cute-dsl fp4",
                "auto fp4 (cudnn/cutlass)",
            ]
            if is_sm100_supported()
            else [
                "sglang cutlass fp4",
                "flashinfer cutlass fp4",
                "cudnn fp4",
                "cute-dsl fp4",
                "auto fp4",
            ]
        ),
        styles=(
            [
                ("red", "solid"),
                ("orange", "solid"),
                ("blue", "solid"),
                ("green", "solid"),
                ("brown", "solid"),
                ("purple", "solid"),
            ]
            if is_sm100_supported()
            else [
                ("red", "solid"),
                ("orange", "solid"),
                ("blue", "solid"),
                ("brown", "solid"),
                ("purple", "solid"),
            ]
        ),
        ylabel="bandwidth (GB/s)",
        plot_name="fp4_gemm_benchmark",
        args={},
    )
)
def benchmark(batch_size, provider, N, K, dtype, correctness, csv_file):
    M = batch_size
    packed_k = K
    K = 2 * packed_k
    a_dtype = torch.randn((M, K), dtype=dtype, device="cuda")
    b_dtype = torch.randn((N, K), dtype=dtype, device="cuda")
    a_global_scale = (
        (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.amax(a_dtype.flatten(), dim=-1)
    ).to(torch.float32)
    b_global_scale = (
        (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.amax(b_dtype.flatten(), dim=-1)
    ).to(torch.float32)
    alpha = 1.0 / (a_global_scale * b_global_scale)
    a_fp4, a_scale_interleaved = scaled_fp4_quant(a_dtype, a_global_scale)
    b_fp4, b_scale_interleaved = scaled_fp4_quant(b_dtype, b_global_scale)
    b_fp4_T = b_fp4.T
    b_sf_T = b_scale_interleaved.T
    res_fi = torch.empty((M, N), dtype=dtype, device="cuda")

    if provider == "sglang_cutlass":
        times_ms = bench_gpu_time(
            fn=cutlass_scaled_fp4_mm,
            input_args=(
                a_fp4,
                b_fp4,
                a_scale_interleaved,
                b_scale_interleaved,
                alpha,
                dtype,
            ),
            use_cuda_graph=True,
        )
    elif provider == "cutlass":
        with autotune():
            _run_mm_fp4(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
                backend="cutlass",
            )
        times_ms = bench_gpu_time(
            fn=partial(_run_mm_fp4, backend="cutlass"),
            input_args=(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
            ),
            use_cuda_graph=True,
        )
    elif provider == "cudnn":
        with autotune():
            _run_mm_fp4(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
                backend="cudnn",
            )
        times_ms = bench_gpu_time(
            fn=partial(_run_mm_fp4, backend="cudnn"),
            input_args=(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
            ),
            use_cuda_graph=True,
        )
    elif provider == "trtllm":
        a_sf_u8 = a_scale_interleaved.to(torch.uint8)
        b_sf_u8_T = b_sf_T.to(torch.uint8)
        with autotune():
            _run_mm_fp4(
                a_fp4,
                b_fp4_T,
                a_sf_u8,
                b_sf_u8_T,
                alpha,
                dtype,
                res_fi,
                backend="trtllm",
            )
        times_ms = bench_gpu_time(
            fn=partial(_run_mm_fp4, backend="trtllm"),
            input_args=(a_fp4, b_fp4_T, a_sf_u8, b_sf_u8_T, alpha, dtype, res_fi),
            use_cuda_graph=True,
        )
    elif provider == "cute-dsl":
        with autotune():
            _run_mm_fp4(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
                backend="cute-dsl",
            )
        times_ms = bench_gpu_time(
            fn=partial(_run_mm_fp4, backend="cute-dsl"),
            input_args=(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
            ),
            use_cuda_graph=True,
        )
    elif provider == "auto":
        with autotune():
            _run_mm_fp4(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
                backend="auto",
            )
        times_ms = bench_gpu_time(
            fn=partial(_run_mm_fp4, backend="auto"),
            input_args=(
                a_fp4,
                b_fp4_T,
                a_scale_interleaved,
                b_sf_T,
                alpha,
                dtype,
                res_fi,
            ),
            use_cuda_graph=True,
        )

    ms = torch.tensor(times_ms).median().item()

    # A: M×packed_k bytes (fp4 packed), B: N×packed_k bytes, C: M×N×element_size bytes
    element_size = torch.finfo(dtype).bits // 8
    total_bytes = M * packed_k + N * packed_k + M * N * element_size
    bandwidth_gbs = total_bytes / (ms * 1e-3) / 1e9

    if correctness:
        res_cutlass = cutlass_scaled_fp4_mm(
            a_fp4, b_fp4, a_scale_interleaved, b_scale_interleaved, alpha, dtype
        )
        mm_fp4(
            a_fp4,
            b_fp4_T,
            a_scale_interleaved,
            b_sf_T,
            alpha,
            dtype,
            res_fi,
            backend="cudnn",
        )
        assert torch.allclose(
            res_fi, res_cutlass, atol=1e-3, rtol=1e-3
        ), "cudnn fp4 doesn't match cutlass fp4"
        mm_fp4(
            a_fp4,
            b_fp4_T,
            a_scale_interleaved,
            b_sf_T,
            alpha,
            dtype,
            res_fi,
            backend="trtllm",
        )
        assert torch.allclose(
            res_fi, res_cutlass, atol=1e-3, rtol=1e-3
        ), "trtllm fp4 doesn't match cutlass fp4"

    if csv_file:
        with open(csv_file, "a", newline="") as f:
            writer = csv.writer(f)
            writer.writerow([provider, M, N, K, ms, bandwidth_gbs])

    return bandwidth_gbs
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. It also performs explicit assertions or shape checks before continuing. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 400-466: Command-line entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=[DEEPSEEK_R1_MODEL],
        help="List of models to benchmark. Supported: Llama 8B/70B, Qwen, Mistral, DeepSeek.",
    )
    parser.add_argument(
        "--tp-sizes",
        nargs="+",
        type=int,
        default=[1],
        help="List of tensor parallel sizes",
    )
    parser.add_argument(
        "--dtype",
        type=torch.dtype,
        default=torch.bfloat16,
        help="Output data type",
    )
    parser.add_argument(
        "--correctness",
        action="store_true",
        help="Check correctness",
    )
    parser.add_argument(
        "--csv",
        type=str,
        default="results_cutlass_cudnn.csv",
        help="CSV file to save results",
    )
    args = parser.parse_args()

    if IS_CI:
        args.tp_sizes = [args.tp_sizes[0]]

    if args.csv:
        with open(args.csv, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow(["provider", "m", "n", "k", "time_ms", "bandwidth_gbs"])

    major, minor = get_device_capability()
    if not (is_sm100_supported() or is_sm120_supported()):
        print("Skipping FP4 GEMM benchmark")
        if major is not None:
            print(f"FP4 operations require sm100+, but found sm{major}{minor}")
        else:
            print("Could not determine device capability")
    else:
        NKs = get_weight_shapes(args)

        if IS_CI:
            NKs = NKs[:2]

        for N, K, model_name in NKs:
            print(f"{model_name} N={N} packed_k={K}: ")
            benchmark.run(
                print_data=True,
                N=N,
                K=K,
                dtype=args.dtype,
                correctness=args.correctness,
                csv_file=args.csv,
            )
        print("Benchmark finished!")
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `get_weight_shapes`, `_run_mm_fp4`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.jit_kernel.nvfp4`, `sglang.srt.utils`, `sglang.utils`
- **External / 外部**: `argparse`, `csv`, `flashinfer`, `flashinfer.autotuner`, `flashinfer.jit.core`, `flashinfer.testing`, `functools`, `logging`, `torch`, `triton`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
