# bench_cast.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_cast.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import torch". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import torch”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-53: Imports and module setup
```python
import torch
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import (
    DEFAULT_DEVICE,
    get_benchmark_range,
    run_benchmark,
)
from sglang.jit_kernel.cast import downcast_fp8 as downcast_fp8_jit
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=10, suite="base-b-kernel-benchmark-1-gpu-large")

DEVICE = DEFAULT_DEVICE
DTYPE = torch.bfloat16


# ── Config ranges ──────────────────────────────────────────────────────────────

SL_LIST = get_benchmark_range(
    full_range=[4, 16, 64, 256, 512, 1024, 2048],
    ci_range=[4, 64],
)

HEAD_DIM_LIST = get_benchmark_range(
    full_range=[(8, 128), (32, 128), (8, 256), (32, 256)],
    ci_range=[(8, 128)],
)

CONFIGS = [(sl, h, d, sl * 2) for sl in SL_LIST for h, d in HEAD_DIM_LIST]

LINE_VALS = ["jit"]
LINE_NAMES = ["JIT (cast.cuh, 256 threads, 2D grid)"]
STYLES = [("orange", "-")]


# ── Perf report ────────────────────────────────────────────────────────────────


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["input_sl", "head", "dim", "out_sl"],
        x_vals=CONFIGS,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="downcast-fp8-jit",
        args={},
    )
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 54-70: Function `benchmark`
```python
def benchmark(input_sl, head, dim, out_sl, provider):
    k = torch.randn(input_sl, head, dim, dtype=DTYPE, device=DEVICE)
    v = torch.randn(input_sl, head, dim, dtype=DTYPE, device=DEVICE)
    k_out = torch.zeros(out_sl, head, dim, dtype=torch.uint8, device=DEVICE)
    v_out = torch.zeros(out_sl, head, dim, dtype=torch.uint8, device=DEVICE)
    k_scale = torch.tensor([1.0], dtype=torch.float32, device=DEVICE)
    v_scale = torch.tensor([1.0], dtype=torch.float32, device=DEVICE)
    loc = torch.arange(input_sl, dtype=torch.int64, device=DEVICE)

    fn = lambda: downcast_fp8_jit(k, v, k_out, v_out, k_scale, v_scale, loc)

    return run_benchmark(fn)


# ── Bandwidth analysis ─────────────────────────────────────────────────────────
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 71-92: Function `_report_bandwidth`
```python
def _report_bandwidth(input_sl, head, dim, dtype):
    elem_bytes = torch.finfo(dtype).bits // 8
    total_bytes = input_sl * head * dim * (2 * elem_bytes + 2)

    k = torch.randn(input_sl, head, dim, dtype=dtype, device=DEVICE)
    v = torch.randn(input_sl, head, dim, dtype=dtype, device=DEVICE)
    k_out = torch.zeros(input_sl * 2, head, dim, dtype=torch.uint8, device=DEVICE)
    v_out = torch.zeros(input_sl * 2, head, dim, dtype=torch.uint8, device=DEVICE)
    k_scale = torch.tensor([1.0], dtype=torch.float32, device=DEVICE)
    v_scale = torch.tensor([1.0], dtype=torch.float32, device=DEVICE)
    loc = torch.arange(input_sl, dtype=torch.int64, device=DEVICE)

    jit_fn = lambda: downcast_fp8_jit(k, v, k_out, v_out, k_scale, v_scale, loc)

    jit_ms, _, _ = triton.testing.do_bench(jit_fn, quantiles=[0.5, 0.2, 0.8])

    def fmt(ms):
        return f"{ms*1000:6.2f}us {total_bytes/(ms*1e-3)/1e9:6.0f}GB/s"

    print(f"  sl={input_sl:5d}  h={head:2d}  d={dim:4d}" f"  |  jit {fmt(jit_ms)}")
```
**EN:** This block defines `_report_bandwidth`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_report_bandwidth`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 93-106: Function `report_bandwidth`
```python
def report_bandwidth():
    print(f"\n{'='*95}")
    print("  JIT (cast.cuh, 256 threads, 2D grid)")
    print(f"  dtype={DTYPE}, device={DEVICE}")
    print(f"{'='*95}")
    for sl in [64, 256, 1024, 2048]:
        for h, d in [(8, 128), (32, 128), (8, 256), (32, 256)]:
            _report_bandwidth(sl, h, d, DTYPE)
    print()


if __name__ == "__main__":
    benchmark.run(print_data=True)
    report_bandwidth()
```
**EN:** This block defines `report_bandwidth`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `report_bandwidth`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> (`
- `sglang.jit_kernel.cast -> downcast_fp8`
- `sglang.test.ci.ci_register -> register_cuda_ci`
