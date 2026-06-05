# bench_activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_activation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and module setup
```python
import itertools

import torch
import torch.nn.functional as F
import triton
import triton.testing
from sgl_kernel import gelu_and_mul as gelu_and_mul_aot
from sgl_kernel import gelu_tanh_and_mul as gelu_tanh_and_mul_aot
from sgl_kernel import silu_and_mul as silu_and_mul_aot

from sglang.jit_kernel.activation import gelu_and_mul as gelu_and_mul_jit
from sglang.jit_kernel.activation import gelu_tanh_and_mul as gelu_tanh_and_mul_jit
from sglang.jit_kernel.activation import silu_and_mul as silu_and_mul_jit
from sglang.jit_kernel.benchmark.utils import (
    DEFAULT_DEVICE,
    DEFAULT_DTYPE,
    get_benchmark_range,
    run_benchmark,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=30, suite="base-b-kernel-benchmark-1-gpu-large")


@torch.compile
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 26-31: Function `silu_and_mul`
```python
def silu_and_mul(input: torch.Tensor) -> torch.Tensor:
    lhs, rhs = input.split(input.shape[-1] // 2, dim=-1)
    return F.silu(lhs) * rhs


@torch.compile
```
**EN:** This block defines `silu_and_mul`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `silu_and_mul`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-37: Function `gelu_and_mul`
```python
def gelu_and_mul(input: torch.Tensor) -> torch.Tensor:
    lhs, rhs = input.split(input.shape[-1] // 2, dim=-1)
    return F.gelu(lhs, approximate="none") * rhs


@torch.compile
```
**EN:** This block defines `gelu_and_mul`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gelu_and_mul`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 38-66: Function `gelu_tanh_and_mul`
```python
def gelu_tanh_and_mul(input: torch.Tensor) -> torch.Tensor:
    lhs, rhs = input.split(input.shape[-1] // 2, dim=-1)
    return F.gelu(lhs, approximate="tanh") * rhs


OPS = {
    "silu": (silu_and_mul_aot, silu_and_mul_jit, silu_and_mul),
    "gelu": (gelu_and_mul_aot, gelu_and_mul_jit, gelu_and_mul),
    "gelu_tanh": (gelu_tanh_and_mul_aot, gelu_tanh_and_mul_jit, gelu_tanh_and_mul),
}
BS_LIST = get_benchmark_range(full_range=[2**x for x in range(0, 15)], ci_range=[8])
DIM_LIST = get_benchmark_range(full_range=[1024, 4096, 6144, 8192], ci_range=[4096])
CONFIGS = list(itertools.product(OPS, DIM_LIST, BS_LIST))
NUM_LAYERS = 4  # to eliminate L2 effect


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["op_name", "dim", "batch_size"],
        x_vals=CONFIGS,
        line_arg="provider",
        line_vals=["aot", "jit", "torch"],
        line_names=["AOT (sgl-kernel)", "JIT (jit_kernel)", "torch.compile"],
        styles=[("blue", "--"), ("orange", "-"), ("green", "-")],
        ylabel="us",
        plot_name="activation-aot-vs-jit",
        args={},
    )
)
```
**EN:** This block defines `gelu_tanh_and_mul`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gelu_tanh_and_mul`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 67-95: Function `benchmark`
```python
def benchmark(op_name: str, dim: int, batch_size: int, provider: str):
    x = torch.randn(
        NUM_LAYERS,
        batch_size,
        2 * dim,
        dtype=DEFAULT_DTYPE,
        device=DEFAULT_DEVICE,
    )
    aot_op, jit_op, torch_op = OPS[op_name]
    fn = {"aot": aot_op, "jit": jit_op, "torch": torch_op}[provider]

    def f():
        for i in range(NUM_LAYERS):
            fn(x[i])

    return run_benchmark(f, scale=NUM_LAYERS)


FILTER_OPS = ["silu", "gelu"]
FILTER_BS = get_benchmark_range(
    full_range=[64, 256, 1024, 4096, 16384], ci_range=[1024]
)
FILTER_DIMS = get_benchmark_range(full_range=[1024, 4096, 8192], ci_range=[4096])
FILTER_RATIOS = get_benchmark_range(full_range=[0.0, 0.25, 0.5], ci_range=[0.25])
FILTER_CONFIGS = list(
    itertools.product(FILTER_OPS, FILTER_DIMS, FILTER_BS, FILTER_RATIOS)
)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 96-118: Function `_make_expert_ids`
```python
def _make_expert_ids(num_tokens: int, skip_ratio: float) -> torch.Tensor:
    expert_ids = torch.randint(
        low=0, high=8, size=(num_tokens,), dtype=torch.int32, device=DEFAULT_DEVICE
    )
    if skip_ratio > 0:
        skip = torch.rand(num_tokens, device=DEFAULT_DEVICE) < skip_ratio
        expert_ids[skip] = -1
    return expert_ids


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["op_name", "dim", "batch_size", "skip_ratio"],
        x_vals=FILTER_CONFIGS,
        line_arg="provider",
        line_vals=["unfiltered", "filtered"],
        line_names=["JIT (no filter_expert)", "JIT (with expert_ids)"],
        styles=[("blue", "--"), ("orange", "-")],
        ylabel="us",
        plot_name="activation-filter-expert",
        args={},
    )
)
```
**EN:** This block defines `_make_expert_ids`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_make_expert_ids`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 119-157: Function `benchmark_filter`
```python
def benchmark_filter(
    op_name: str, dim: int, batch_size: int, skip_ratio: float, provider: str
):
    x = torch.randn(
        NUM_LAYERS,
        batch_size,
        2 * dim,
        dtype=DEFAULT_DTYPE,
        device=DEFAULT_DEVICE,
    )
    out = torch.empty(
        NUM_LAYERS,
        batch_size,
        dim,
        dtype=DEFAULT_DTYPE,
        device=DEFAULT_DEVICE,
    )
    expert_ids = _make_expert_ids(batch_size, skip_ratio)

    jit_fn = silu_and_mul_jit if op_name == "silu" else gelu_and_mul_jit

    if provider == "unfiltered":

        def f():
            for i in range(NUM_LAYERS):
                jit_fn(x[i], out[i])

    else:  # filtered

        def f():
            for i in range(NUM_LAYERS):
                jit_fn(x[i], out[i], expert_ids=expert_ids, expert_step=1)

    return run_benchmark(f, scale=NUM_LAYERS)


if __name__ == "__main__":
    benchmark.run(print_data=True)
    benchmark_filter.run(print_data=True)
```
**EN:** This block defines `benchmark_filter`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark_filter`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `torch`
- `torch.nn.functional as F`
- `triton`
- `triton.testing`
- `sgl_kernel -> gelu_and_mul`
- `sgl_kernel -> gelu_tanh_and_mul`
- `sgl_kernel -> silu_and_mul`
- `sglang.jit_kernel.activation -> gelu_and_mul`
- `sglang.jit_kernel.activation -> gelu_tanh_and_mul`
- `sglang.jit_kernel.activation -> silu_and_mul`
- `sglang.jit_kernel.benchmark.utils -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
