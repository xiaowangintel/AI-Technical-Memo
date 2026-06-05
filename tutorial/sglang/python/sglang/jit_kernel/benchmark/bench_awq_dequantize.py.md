# bench_awq_dequantize.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_awq_dequantize.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup
```python
import itertools

import torch
import triton
import triton.testing

from sglang.jit_kernel.awq_dequantize import awq_dequantize as jit_awq_dequantize
from sglang.jit_kernel.benchmark.utils import run_benchmark
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.utils import is_in_ci

register_cuda_ci(est_time=5, suite="base-b-kernel-benchmark-1-gpu-large")

try:
    from sgl_kernel import awq_dequantize as aot_awq_dequantize

    AOT_AVAILABLE = True
except ImportError:
    AOT_AVAILABLE = False

IS_CI = is_in_ci()

if IS_CI:
    qweight_row_range = [128]
    qweight_cols_range = [16]
else:
    qweight_row_range = [128, 256, 512, 1024, 3584]
    qweight_cols_range = [16, 32, 64, 128, 448]

configs = list(itertools.product(qweight_row_range, qweight_cols_range))
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 33-88: Function `check_correctness`
```python
def check_correctness():
    if not AOT_AVAILABLE:
        print("sgl_kernel AOT not available, skipping correctness check")
        return

    qweight_row, qweight_col = 128, 16
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

    jit_out = jit_awq_dequantize(qweight, scales, qzeros)
    aot_out = aot_awq_dequantize(qweight, scales, qzeros)
    torch.cuda.synchronize()
    torch.testing.assert_close(jit_out, aot_out, rtol=0, atol=0)
    print("Correctness check passed (JIT vs AOT)")


if AOT_AVAILABLE:
    line_vals = ["jit", "aot"]
    line_names = ["JIT Kernel", "AOT Kernel"]
    styles = [("blue", "-"), ("green", "-")]
else:
    line_vals = ["jit"]
    line_names = ["JIT Kernel"]
    styles = [("blue", "-")]


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["qweight_row", "qweight_col"],
        x_vals=configs,
        line_arg="provider",
        line_vals=line_vals,
        line_names=line_names,
        styles=styles,
        ylabel="us",
        plot_name="awq-dequantize-jit-vs-aot",
        args={},
    )
)
```
**EN:** This block defines `check_correctness`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `check_correctness`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 89-122: Function `benchmark`
```python
def benchmark(qweight_row, qweight_col, provider):
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

    if provider == "jit":
        fn = lambda: jit_awq_dequantize(qweight, scales, qzeros)
    elif provider == "aot":
        fn = lambda: aot_awq_dequantize(qweight, scales, qzeros)
    else:
        raise ValueError(f"Unknown provider: {provider}")

    return run_benchmark(fn)


if __name__ == "__main__":
    check_correctness()
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.awq_dequantize -> awq_dequantize`
- `sglang.jit_kernel.benchmark.utils -> run_benchmark`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.utils -> is_in_ci`
- `sgl_kernel -> awq_dequantize`
