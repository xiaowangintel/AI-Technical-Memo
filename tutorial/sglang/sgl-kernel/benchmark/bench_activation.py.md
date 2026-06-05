# bench_activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_activation.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `activation` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `activation` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Comments and local context
````python
# Benchmarks SGLang kernels versus vLLM across
# (kernel, dtype, batch_size, seq_len, dim) and prints speed-up.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 3-16: Imports and module setup
````python
import argparse
import itertools
import os
import re
from typing import List, Tuple

import sgl_kernel
import torch
import torch.nn.functional as F
import triton
import triton.testing
from sgl_kernel import gelu_and_mul, gelu_tanh_and_mul, silu_and_mul

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 17-18: Comments and local context
````python

# Optional vLLM import
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 19-25: Optional dependency handling
````python
try:
    from vllm import _custom_ops as vllm_ops

    VLLM_AVAILABLE = True
except ImportError:
    vllm_ops = None
    VLLM_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 27-27: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 28-29: Comments and local context
````python

# gelu_quick is only available on HIP/ROCm platforms
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 30-36: Optional dependency handling
````python
try:
    from sgl_kernel import gelu_quick

    GELU_QUICK_AVAILABLE = True
except ImportError:
    GELU_QUICK_AVAILABLE = False
    gelu_quick = None
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 38-39: Conditional logic
````python
if VLLM_AVAILABLE and not hasattr(vllm_ops, "silu_and_mul"):
    vllm_ops = torch.ops._C
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 42-47: `str2int_list` definition
````python
def str2int_list(arg: str) -> List[int]:
    if arg in ("", None):
        return []
    if re.fullmatch(r"\d+(,\d+)*", arg.strip()) is None:
        raise argparse.ArgumentTypeError(f"Bad int list: {arg}")
    return [int(x) for x in arg.split(",")]
````
**EN:** This section defines `str2int_list` and implements the core logic associated with str2int list.
**CN:** 该部分定义 `str2int_list`，并实现与 str2int list 相关的核心逻辑。

### Lines 50-88: `calculate_diff` definition
````python
def calculate_diff(
    kernel: str, dtype: torch.dtype, batch_size: int, seq_len: int, dim: int
) -> bool:
    """Compare vLLM with SGLang for one shape."""
    device = torch.device("cuda")

    if not VLLM_AVAILABLE:
        print(
            f"[{kernel:14s} | {str(dtype):9s} | B={batch_size:3d} | "
            f"L={seq_len:3d} | D={dim:5d}] ⚠️  vLLM not available, skipping comparison"
        )
        return True

    # activation-only quick GELU
    if kernel == "gelu_quick":
        if not GELU_QUICK_AVAILABLE:
            print(
                f"[{kernel:14s} | {str(dtype):9s} | B={batch_size:3d} | "
                f"L={seq_len:3d} | D={dim:5d}] ⚠️  not available on this platform"
            )
            return True
        x = torch.randn(batch_size, seq_len, dim, dtype=dtype, device=device)
        ref_out = torch.zeros_like(x)
        getattr(vllm_ops, kernel)(ref_out, x)
        test_out = getattr(sgl_kernel, kernel)(x)
    # fused activation x mul kernels
    else:
        x = torch.randn(batch_size, seq_len, 2 * dim, dtype=dtype, device=device)
        ref_out = torch.zeros(batch_size, seq_len, dim, dtype=dtype, device=device)
        getattr(vllm_ops, kernel)(ref_out, x)
        test_out = getattr(sgl_kernel, kernel)(x)

    ok = torch.allclose(ref_out, test_out, rtol=1e-3, atol=1e-5)
    tag = "✅ match" if ok else "❌ mismatch"
    print(
        f"[{kernel:14s} | {str(dtype):9s} | B={batch_size:3d} | "
        f"L={seq_len:3d} | D={dim:5d}] {tag}"
    )
    return ok
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module. Docstring summary: Compare vLLM with SGLang for one shape.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。 文档字符串摘要：Compare vLLM with SGLang for one shape.

### Lines 89-91: Comments and local context
````python


# CI environment uses simplified parameters for kernels and dtypes too
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 92-99: Conditional logic
````python
if IS_CI:
    kernels = ["silu_and_mul"]  # Only test one kernel in CI
    dtypes = [torch.float16]  # Only test one dtype in CI
else:
    kernels = ["silu_and_mul", "gelu_and_mul", "gelu_tanh_and_mul"]
    if GELU_QUICK_AVAILABLE:
        kernels.append("gelu_quick")
    dtypes = [torch.float16, torch.bfloat16]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 102-103: `make_configs` definition
````python
def make_configs(bsizes: List[int], slens: List[int], dims_: List[int]) -> List[Tuple]:
    return list(itertools.product(kernels, dtypes, bsizes, slens, dims_))
````
**EN:** This section defines `make_configs`. It constructs the `configs` path used by the module.
**CN:** 该部分定义 `make_configs`。它负责构造模块中与 `configs` 相关的处理路径。

### Lines 104-106: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 107-114: Conditional logic
````python
if IS_CI:
    default_batch_sizes = [1]  # Single batch size for CI
    default_seq_lens = [1]  # Single sequence length for CI
    default_dims = [1024]  # Single dimension for CI
else:
    default_batch_sizes = [2**i for i in range(0, 5, 2)]  # 1,4,16
    default_seq_lens = [2**i for i in range(0, 8, 2)]  # 1,4,16,64
    default_dims = [2**i for i in range(10, 15)]  # 1024...16384
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 117-177: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["kernel", "dtype", "batch_size", "seq_len", "dim"],
        x_vals=[],
        line_arg="provider",
        line_vals=["vllm", "sglang", "speedup"],
        line_names=["vLLM", "SGL Kernel", "Speed-up (x)"],
        styles=[("blue", "-"), ("green", "-"), ("red", "--")],
        ylabel="µs (median)  or  × (speed-up)",
        plot_name="activation-performance",
        args={},
    )
)
def benchmark(kernel, dtype, batch_size, seq_len, dim, provider):
    device = torch.device("cuda")
    in_mult = 1 if kernel == "gelu_quick" else 2
    x = torch.randn(batch_size, seq_len, in_mult * dim, dtype=dtype, device=device)
    y0 = torch.zeros(batch_size, seq_len, dim, dtype=dtype, device=device)

    if not VLLM_AVAILABLE and provider in ["vllm", "speedup"]:
        # Skip vLLM-related benchmarks if vLLM is not available
        return (0, 0, 0)

    if VLLM_AVAILABLE:
        vllm_kernel = getattr(vllm_ops, kernel)
    if kernel == "gelu_quick" and not GELU_QUICK_AVAILABLE:
        # Skip benchmark for gelu_quick if not available
        return (0, 0, 0)
    sglang_kernel = getattr(sgl_kernel, kernel)

    def baseline():
        if VLLM_AVAILABLE:
            tmp = y0.clone()
            vllm_kernel(tmp, x)
            return tmp
        else:
            return torch.zeros_like(y0)

    def sglang():
        return sglang_kernel(x)

    # timing helper
    def timed(fn):
        for _ in range(5):
            fn()
        torch.cuda.synchronize()
        ms, qmin, qmax = triton.testing.do_bench_cudagraph(
            fn, quantiles=[0.5, 0.2, 0.8]
        )
        return 1000 * ms, 1000 * qmax, 1000 * qmin

    if provider == "vllm":
        return timed(baseline)
    if provider == "sglang":
        return timed(sglang)

    # provider == "speedup"
    t_ref, _, _ = timed(baseline)
    t_sgl, _, _ = timed(sglang)
    spd = t_ref / t_sgl if t_ref > 0 else 1.0
    return (spd, spd, spd)
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 180-209: Command-line entry point
````python
if __name__ == "__main__":
    p = argparse.ArgumentParser("Activation kernel benchmark")
    p.add_argument("--batch_sizes", type=str2int_list, default=default_batch_sizes)
    p.add_argument("--seq_lens", type=str2int_list, default=default_seq_lens)
    p.add_argument("--dims", type=str2int_list, default=default_dims)
    p.add_argument("--verify_only", action="store_true")
    args = p.parse_args()

    # coerce lists
    if isinstance(args.batch_sizes, str):
        args.batch_sizes = str2int_list(args.batch_sizes)
    if isinstance(args.seq_lens, str):
        args.seq_lens = str2int_list(args.seq_lens)
    if isinstance(args.dims, str):
        args.dims = str2int_list(args.dims)

    # patch perf_report grid
    benchmark_grid = make_configs(args.batch_sizes, args.seq_lens, args.dims)
    if hasattr(benchmark, "benchmarks"):
        benchmark.benchmarks.x_vals = benchmark_grid
    else:
        benchmark.benchmark.x_vals = benchmark_grid

    if args.verify_only:
        # Test with the first available kernel
        test_kernel = kernels[0]
        ok = calculate_diff(test_kernel, torch.float16, 1, 1, args.dims[0])
        print("✅ sanity pass" if ok else "❌ mismatch")
    else:
        benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `str2int_list`, `calculate_diff`, `make_configs`, `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.utils`
- **External / 外部**: `argparse`, `itertools`, `os`, `re`, `torch`, `torch.nn.functional`, `triton`, `triton.testing`, `typing`, `vllm`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
