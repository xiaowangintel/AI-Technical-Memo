# bench_norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_norm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: Imports and module setup
```python
import itertools

import torch
import triton
import triton.testing
from flashinfer.norm import fused_add_rmsnorm as fi_fused_add_rmsnorm
from flashinfer.norm import rmsnorm as fi_rmsnorm

from sglang.jit_kernel.benchmark.utils import get_benchmark_range, run_benchmark
from sglang.jit_kernel.norm import fused_add_rmsnorm as jit_fused_add_rmsnorm
from sglang.jit_kernel.norm import rmsnorm as jit_rmsnorm
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=30, suite="base-b-kernel-benchmark-1-gpu-large")


DTYPE = torch.bfloat16
DEVICE = "cuda"

BS_LIST = get_benchmark_range(
    full_range=[2**n for n in range(0, 14)],
    ci_range=[16, 32],
)
HIDDEN_SIZE_LIST = get_benchmark_range(
    full_range=sorted([1536, *range(1024, 8192 + 1, 1024)]),
    ci_range=[512, 2048],
)

LINE_VALS = ["flashinfer", "jit"]
LINE_NAMES = ["FlashInfer", "SGL JIT Kernel"]
STYLES = [("blue", "--"), ("green", "-.")]
NUM_LAYERS = 4  # avoid L2 effect

configs_0 = list(itertools.product(HIDDEN_SIZE_LIST + [16384], BS_LIST))
configs_1 = list(itertools.product(HIDDEN_SIZE_LIST, BS_LIST))


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["hidden_size", "batch_size"],
        x_vals=configs_0,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="rmsnorm-performance",
        args={},
    )
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 51-78: Function `benchmark_rmsnorm`
```python
def benchmark_rmsnorm(hidden_size: int, batch_size: int, provider: str):
    input = torch.randn(
        (NUM_LAYERS, batch_size, hidden_size), dtype=DTYPE, device=DEVICE
    )
    weight = torch.randn((NUM_LAYERS, hidden_size), dtype=DTYPE, device=DEVICE)
    FN_MAP = {"jit": jit_rmsnorm, "flashinfer": fi_rmsnorm}

    def f():
        fn = FN_MAP[provider]
        for i in range(NUM_LAYERS):
            fn(input[i], weight[i], out=input[i])

    return run_benchmark(f, scale=NUM_LAYERS)


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["hidden_size", "batch_size"],
        x_vals=configs_1,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="fused-add-rmsnorm-performance",
        args={},
    )
)
```
**EN:** This block defines `benchmark_rmsnorm`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark_rmsnorm`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 79-100: Function `benchmark_fused_add_rmsnorm`
```python
def benchmark_fused_add_rmsnorm(hidden_size: int, batch_size: int, provider: str):
    input = torch.randn(
        (NUM_LAYERS, batch_size, hidden_size), dtype=DTYPE, device=DEVICE
    )
    residual = torch.randn_like(input)
    weight = torch.randn((NUM_LAYERS, hidden_size), dtype=DTYPE, device=DEVICE)
    FN_MAP = {"jit": jit_fused_add_rmsnorm, "flashinfer": fi_fused_add_rmsnorm}

    def f():
        fn = FN_MAP[provider]
        for i in range(NUM_LAYERS):
            fn(input[i], residual[i], weight[i])

    return run_benchmark(f, scale=NUM_LAYERS)


if __name__ == "__main__":
    print("Benchmarking rmsnorm...")
    benchmark_rmsnorm.run(print_data=True)

    print("Benchmarking fused_add_rmsnorm...")
    benchmark_fused_add_rmsnorm.run(print_data=True)
```
**EN:** This block defines `benchmark_fused_add_rmsnorm`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark_fused_add_rmsnorm`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `torch`
- `triton`
- `triton.testing`
- `flashinfer.norm -> fused_add_rmsnorm`
- `flashinfer.norm -> rmsnorm`
- `sglang.jit_kernel.benchmark.utils -> get_benchmark_range`
- `sglang.jit_kernel.norm -> fused_add_rmsnorm`
- `sglang.jit_kernel.norm -> rmsnorm`
- `sglang.test.ci.ci_register -> register_cuda_ci`
