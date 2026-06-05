# benchmark_deepgemm_fp8_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/deepseek/benchmark_deepgemm_fp8_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels deepseek deepgemm fp8 gemm. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 kernels deepseek deepgemm fp8 gemm 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and setup / 导入与初始化
```python
from typing import Tuple

import deep_gemm
import tilelang
import tilelang.language as T
import torch
import triton
from deep_gemm import ceil_div
from deep_gemm.utils.layout import get_mn_major_tma_aligned_tensor
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    w8a8_block_fp8_matmul as vllm_w8a8_block_fp8_matmul,
)

from sglang.benchmark.bench_utils import run_bench
from sglang.srt.layers.quantization.fp8_kernel import (
    w8a8_block_fp8_matmul_deepgemm as w8a8_block_fp8_matmul,
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、准备张量并调用 GPU 内核。

### Lines 21-95: Function `tl_gemm` / 函数 `tl_gemm`
```python
def tl_gemm(
    M,
    N,
    K,
    in_dtype,
    out_dtype,
    accum_dtype,
):
    assert in_dtype in [
        "e4m3_float8",
    ], "Currently only e4m3_float8 is supported"
    assert out_dtype in [
        "bfloat16",
        "float16",
    ], "Currently only bfloat16 and float16 are supported"

    TILE_SIZE = (128, 128, 128)
    block_M = TILE_SIZE[0]
    block_N = TILE_SIZE[1]
    block_K = TILE_SIZE[2]

    A_shape = (M, K)
    Scales_A_shape = (M, T.ceildiv(K, block_K))
    B_shape = (N, K)
    Scales_B_shape = (T.ceildiv(N, block_N), T.ceildiv(K, block_K))
    A_shared_shape = (block_M, block_K)
    B_shared_shape = (block_N, block_K)
    C_shared_shape = (block_M, block_N)

    @T.prim_func
    def main(
        A: T.Buffer(A_shape, in_dtype),
        scales_a: T.Buffer(Scales_A_shape, "float32"),
        B: T.Buffer(B_shape, in_dtype),
        scales_b: T.Buffer(Scales_B_shape, "float32"),
        C: T.Buffer((M, N), out_dtype),
    ):
        with T.Kernel(T.ceildiv(N, block_N), T.ceildiv(M, block_M), threads=128) as (
            bx,
            by,
        ):

            A_shared = T.alloc_shared(A_shared_shape, in_dtype)
            B_shared = T.alloc_shared(B_shared_shape, in_dtype)
            C_shared = T.alloc_shared(C_shared_shape, out_dtype)
            Scale_C_shared = T.alloc_shared((block_M), "float32")
            C_local = T.alloc_fragment(C_shared_shape, accum_dtype)
            C_local_accum = T.alloc_fragment(C_shared_shape, accum_dtype)

            # Improve L2 Cache
            T.use_swizzle(panel_size=10)

            T.clear(C_local)
            T.clear(C_local_accum)
            K_iters = T.ceildiv(K, block_K)
            for k in T.Pipelined(K_iters, num_stages=4):
                # Load A into shared memory
                T.copy(A[by * block_M, k * block_K], A_shared)
                # Load B into shared memory
                T.copy(B[bx * block_N, k * block_K], B_shared)
                # Load scale into shared memory
                Scale_B = scales_b[bx, k]
                for i in T.Parallel(block_M):
                    Scale_C_shared[i] = scales_a[by * block_M + i, k] * Scale_B

                T.gemm(A_shared, B_shared, C_local, transpose_B=True)
                # Promote to enable 2xAcc
                for i, j in T.Parallel(block_M, block_N):
                    C_local_accum[i, j] += C_local[i, j] * Scale_C_shared[i]
                T.clear(C_local)
            # TMA store
            T.copy(C_local_accum, C_shared)
            T.copy(C_shared, C[by * block_M, bx * block_N])

    return main
```
**EN:** `tl_gemm` is a function that prepares tensors and invokes GPU kernels. It returns `main` to the caller. Notable calls include `T.ceildiv`, `T.Buffer`, `T.Kernel`.
**CN:** `tl_gemm` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `main`。其中较关键的调用包括 `T.ceildiv`, `T.Buffer`, `T.Kernel`。

### Lines 98-105: Function `per_token_cast_to_fp8` / 函数 `per_token_cast_to_fp8`
```python
def per_token_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2 and x.size(1) % 128 == 0
    m, n = x.shape
    x_view = x.view(m, -1, 128)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    return (x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view(
        m, n
    ), (x_amax / 448.0).view(m, -1)
```
**EN:** `per_token_cast_to_fp8` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `((x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view(m, n), (x_amax / 448.0).vi...` to the caller. Notable calls include `x.view`, `x_view.abs().float().amax(dim=2).view(m, -1).clamp`, `(x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view`.
**CN:** `per_token_cast_to_fp8` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `((x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view(m, n), (x_amax / 448.0).vi...`。其中较关键的调用包括 `x.view`, `x_view.abs().float().amax(dim=2).view(m, -1).clamp`, `(x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view`。

### Lines 108-120: Function `per_block_cast_to_fp8` / 函数 `per_block_cast_to_fp8`
```python
def per_block_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
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

### Lines 123-137: Function `fp8_gemm_deepgemm` / 函数 `fp8_gemm_deepgemm`
```python
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
```
**EN:** `fp8_gemm_deepgemm` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: DeepGEMM implementation of FP8 GEMM It returns `out` to the caller. Notable calls include `torch.empty`, `deep_gemm.fp8_gemm_nt`.
**CN:** `fp8_gemm_deepgemm` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `out`。其中较关键的调用包括 `torch.empty`, `deep_gemm.fp8_gemm_nt`。

### Lines 140-156: Function `fp8_gemm_sglang` / 函数 `fp8_gemm_sglang`
```python
def fp8_gemm_sglang(
    x_fp8: torch.Tensor,
    x_scale: torch.Tensor,
    y_fp8: torch.Tensor,
    y_scale: torch.Tensor,
    m: int,
    n: int,
    k: int,
):
    """SGLang implementation of FP8 GEMM"""
    block_size = [128, 128]  # Matches the block size in per_block_cast_to_fp8

    # Run SGLang kernel
    out = w8a8_block_fp8_matmul(
        x_fp8, y_fp8, x_scale, y_scale, block_size, torch.bfloat16
    )
    return out
```
**EN:** `fp8_gemm_sglang` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: SGLang implementation of FP8 GEMM It returns `out` to the caller. Notable calls include `w8a8_block_fp8_matmul`.
**CN:** `fp8_gemm_sglang` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `out`。其中较关键的调用包括 `w8a8_block_fp8_matmul`。

### Lines 159-175: Function `fp8_gemm_vllm` / 函数 `fp8_gemm_vllm`
```python
def fp8_gemm_vllm(
    x_fp8: torch.Tensor,
    x_scale: torch.Tensor,
    y_fp8: torch.Tensor,
    y_scale: torch.Tensor,
    m: int,
    n: int,
    k: int,
):
    """vLLM implementation of FP8 GEMM"""
    block_size = [128, 128]  # Matches the block size in per_block_cast_to_fp8

    # Run vLLM kernel
    out = vllm_w8a8_block_fp8_matmul(
        x_fp8, y_fp8, x_scale, y_scale, block_size, torch.bfloat16
    )
    return out
```
**EN:** `fp8_gemm_vllm` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: vLLM implementation of FP8 GEMM It returns `out` to the caller. Notable calls include `vllm_w8a8_block_fp8_matmul`.
**CN:** `fp8_gemm_vllm` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `out`。其中较关键的调用包括 `vllm_w8a8_block_fp8_matmul`。

### Lines 178-233: Function `calculate_diff` / 函数 `calculate_diff`
```python
def calculate_diff(m: int, n: int, k: int):
    x = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
    y = torch.randn((n, k), device="cuda", dtype=torch.bfloat16)

    x_fp8, x_scale = per_token_cast_to_fp8(x.clone())
    y_fp8, y_scale = per_block_cast_to_fp8(y.clone())
    x_scale_col_major = get_mn_major_tma_aligned_tensor(x_scale.clone())

    out_deepgemm = fp8_gemm_deepgemm(
        x_fp8.clone(),
        x_scale_col_major.clone(),
        y_fp8.clone(),
        y_scale.clone(),
        m,
        n,
        k,
    )
    out_sglang = fp8_gemm_sglang(
        x_fp8.clone(), x_scale.clone(), y_fp8.clone(), y_scale.clone(), m, n, k
    )

    tilelang_func = tl_gemm(m, n, k, "e4m3_float8", "bfloat16", "float32")
    tilelang_kernel = tilelang.compile(tilelang_func, out_idx=[-1])
    out_tilelang = tilelang_kernel(
        x_fp8.clone(), x_scale.clone(), y_fp8.clone(), y_scale.clone()
    )

    diff_sglang_deepgemm = torch.abs(out_deepgemm - out_sglang).mean().item()
    diff_tilelang_deepgemm = torch.abs(out_deepgemm - out_tilelang).mean().item()
    diff_tilelang_sglang = torch.abs(out_tilelang - out_sglang).mean().item()

    print(f"Shape m={m}, n={n}, k={k}:")
    print(f"DeepGEMM output: {out_deepgemm[0, 0:5]}")
    print(f"SGLang output: {out_sglang[0, 0:5]}")
    print(f"TileLang output: {out_tilelang[0, 0:5]}")
    print(f"Mean absolute difference (SGLang-DeepGEMM): {diff_sglang_deepgemm}")
    print(f"Mean absolute difference (TileLang-DeepGEMM): {diff_tilelang_deepgemm}")
    print(f"Mean absolute difference (TileLang-SGLang): {diff_tilelang_sglang}")

    sglang_deepgemm_match = torch.allclose(
        out_deepgemm, out_sglang, atol=1e-2, rtol=1e-2
    )
    tilelang_deepgemm_match = torch.allclose(
        out_deepgemm, out_tilelang, atol=1e-2, rtol=1e-2
    )
    tilelang_sglang_match = torch.allclose(
        out_tilelang, out_sglang, atol=1e-2, rtol=1e-2
    )

    if sglang_deepgemm_match and tilelang_deepgemm_match and tilelang_sglang_match:
        print("✅ All implementations match\n")
    else:
        print("❌ Some implementations differ:")
        print(f"  - SGLang vs DeepGEMM: {'✅' if sglang_deepgemm_match else '❌'}")
        print(f"  - TileLang vs DeepGEMM: {'✅' if tilelang_deepgemm_match else '❌'}")
        print(f"  - TileLang vs SGLang: {'✅' if tilelang_sglang_match else '❌'}\n")
```
**EN:** `calculate_diff` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `torch.randn`, `per_token_cast_to_fp8`, `per_block_cast_to_fp8`.
**CN:** `calculate_diff` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `torch.randn`, `per_token_cast_to_fp8`, `per_block_cast_to_fp8`。

### Lines 236-266: Function `get_weight_shapes` / 函数 `get_weight_shapes`
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

### Lines 269-278: Function `create_benchmark_configs` / 函数 `create_benchmark_configs`
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

### Lines 281-356: Function `get_benchmark` / 函数 `get_benchmark`
```python
def get_benchmark(tp_size):
    all_configs = create_benchmark_configs(tp_size)

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["m", "n", "k", "tp_size"],
            x_vals=[list(config) for config in all_configs],
            line_arg="provider",
            line_vals=["deepgemm", "sglang", "tilelang"],
            line_names=["DeepGEMM", "SGLang", "TileLang"],
            styles=[("blue", "-"), ("red", "-"), ("green", "-")],
            ylabel="ms",
            plot_name=f"fp8-gemm-performance-comparison-tp{tp_size}",
            args={},
        )
    )
    def benchmark(m, n, k, tp_size, provider):
        print(f"Shape (m={m}, n={n}, k={k}, tp={tp_size}), Provider: {provider}")
        x = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
        y = torch.randn((n, k), device="cuda", dtype=torch.bfloat16)

        # Preprocess data before benchmarking
        x_fp8, x_scale = per_token_cast_to_fp8(x)
        y_fp8, y_scale = per_block_cast_to_fp8(y)
        x_scale_col_major = get_mn_major_tma_aligned_tensor(x_scale.clone())

        quantiles = (0.5, 0.2, 0.8)

        if provider == "deepgemm":
            ms, min_ms, max_ms = run_bench(
                lambda: fp8_gemm_deepgemm(
                    x_fp8.clone(),
                    x_scale_col_major.clone(),
                    y_fp8.clone(),
                    y_scale.clone(),
                    m,
                    n,
                    k,
                ),
                quantiles=quantiles,
            )
        elif provider == "sglang":
            ms, min_ms, max_ms = run_bench(
                lambda: fp8_gemm_sglang(
                    x_fp8.clone(),
                    x_scale.clone(),
                    y_fp8.clone(),
                    y_scale.clone(),
                    m,
                    n,
                    k,
                ),
                quantiles=quantiles,
            )
        else:  # tilelang
            tilelang_func = tl_gemm(m, n, k, "e4m3_float8", "bfloat16", "float32")
            tilelang_kernel = tilelang.compile(tilelang_func, out_idx=[-1])
            ms, min_ms, max_ms = run_bench(
                lambda: tilelang_kernel(
                    x_fp8.clone(),
                    x_scale.clone(),
                    y_fp8.clone(),
                    y_scale.clone(),
                ),
                quantiles=quantiles,
            )

        # Calculate TFLOPS
        flops = 2 * m * n * k  # multiply-adds
        tflops = flops / (ms * 1e-3) / 1e12

        # Print shape-specific results with TFLOPS
        print(f"Time: {ms*1000:.2f} ms, TFLOPS: {tflops:.2f}")
        return ms * 1000, max_ms * 1000, min_ms * 1000  # convert to ms

    return benchmark
```
**EN:** `get_benchmark` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `benchmark` to the caller. Notable calls include `create_benchmark_configs`, `triton.testing.perf_report`, `print`.
**CN:** `get_benchmark` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `benchmark`。其中较关键的调用包括 `create_benchmark_configs`, `triton.testing.perf_report`, `print`。

### Lines 359-402: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save_path",
        type=str,
        default="./configs/benchmark_ops/fp8_gemm/",
        help="Path to save fp8 gemm benchmark results",
    )
    parser.add_argument(
        "--run_correctness",
        action="store_true",
        default=True,
        help="Whether to run correctness test",
    )
    parser.add_argument(
        "--tp_size",
        type=int,
        default=1,
        help="Tensor parallelism size to benchmark (default: 1)",
    )
    args = parser.parse_args()

    # Set random seed for reproducibility
    torch.manual_seed(0)
    torch.cuda.manual_seed(0)

    # Enable TF32, adapted from https://github.com/deepseek-ai/DeepGEMM/blob/main/tests/test_core.py#L148
    torch.backends.cuda.matmul.allow_tf32 = True
    torch.backends.cudnn.allow_tf32 = True

    # Run correctness tests on a few examples
    if args.run_correctness:
        print("Running correctness tests...")
        calculate_diff(64, 512, 7168)  # Small test
        calculate_diff(64, 7168, 16384)  # Medium test
        calculate_diff(64, 18432, 7168)  # Large test

    # Get the benchmark function with the specified tp_size
    benchmark = get_benchmark(args.tp_size)

    print(f"Running performance benchmark for TP size = {args.tp_size}...")
    benchmark.run(print_data=True, save_path=args.save_path)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Visualization / 可视化**: Exports figures to compare benchmark runs. / 导出图表以比较不同基准测试结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`, `argparse`
- **Third-party / 第三方依赖**: `deep_gemm`, `tilelang`, `tilelang.language`, `torch`, `triton`, `deep_gemm.utils.layout`, `vllm.model_executor.layers.quantization.utils.fp8_utils`
- **Internal / 项目内部依赖**: `sglang.benchmark.bench_utils`, `sglang.srt.layers.quantization.fp8_kernel`
