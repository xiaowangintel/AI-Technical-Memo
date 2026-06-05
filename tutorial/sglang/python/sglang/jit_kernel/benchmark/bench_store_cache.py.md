# bench_store_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_store_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup
```python
import itertools
from typing import Tuple

import torch
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import (
    DEFAULT_DEVICE,
    DEFAULT_DTYPE,
    DEFAULT_QUANTILES,
    get_benchmark_range,
)
from sglang.jit_kernel.kvcache import store_cache
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=9, suite="base-b-kernel-benchmark-1-gpu-large")
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 20-30: Function `sglang_jit_store_cache`
```python
def sglang_jit_store_cache(
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    indices: torch.Tensor,
) -> None:
    store_cache(k, v, k_cache, v_cache, indices)


@torch.compile()
```
**EN:** This block defines `sglang_jit_store_cache`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_jit_store_cache`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 31-44: Function `torch_compile_store_cache`
```python
def torch_compile_store_cache(
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    indices: torch.Tensor,
) -> None:
    k_cache[indices] = k
    v_cache[indices] = v


alt_stream = torch.cuda.Stream()
```
**EN:** This block defines `torch_compile_store_cache`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `torch_compile_store_cache`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 45-91: Function `torch_streams_store_cache`
```python
def torch_streams_store_cache(
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    indices: torch.Tensor,
) -> None:
    current_stream = torch.cuda.current_stream()
    alt_stream.wait_stream(current_stream)
    k_cache[indices] = k
    with torch.cuda.stream(alt_stream):
        v_cache[indices] = v
    current_stream.wait_stream(alt_stream)


NUM_LAYERS = 8
CACHE_SIZE = 2 * 1024 * 1024 // NUM_LAYERS

BS_RANGE = get_benchmark_range(
    full_range=[2**n for n in range(0, 15)],
    ci_range=[16],
)
ITEM_SIZE = get_benchmark_range(
    full_range=[64, 128, 256, 512, 1024],
    ci_range=[1024],
)

LINE_VALS = ["jit", "torch_compile", "torch_streams"]
LINE_NAMES = ["SGL JIT Kernel", "PyTorch Compile", "PyTorch 2 Stream"]
STYLES = [("blue", "--"), ("red", ":"), ("green", "-.")]
X_NAMES = ["item_size", "batch_size"]
CONFIGS = list(itertools.product(ITEM_SIZE, BS_RANGE))


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=X_NAMES,
        x_vals=CONFIGS,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="store-kvcache-performance",
        args={},
    )
)
```
**EN:** This block defines `torch_streams_store_cache`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `torch_streams_store_cache`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 92-133: Function `benchmark`
```python
def benchmark(
    batch_size: int, item_size: int, provider: str
) -> Tuple[float, float, float]:
    k = torch.randn(
        (NUM_LAYERS, batch_size, item_size), dtype=DEFAULT_DTYPE, device=DEFAULT_DEVICE
    )
    v = torch.randn(
        (NUM_LAYERS, batch_size, item_size), dtype=DEFAULT_DTYPE, device=DEFAULT_DEVICE
    )
    k_cache = torch.randn(
        (NUM_LAYERS, CACHE_SIZE, item_size), dtype=DEFAULT_DTYPE, device=DEFAULT_DEVICE
    )
    v_cache = torch.randn(
        (NUM_LAYERS, CACHE_SIZE, item_size), dtype=DEFAULT_DTYPE, device=DEFAULT_DEVICE
    )
    indices = torch.randperm(CACHE_SIZE, device=DEFAULT_DEVICE)[:batch_size]
    torch.cuda.synchronize()

    FN_MAP = {
        "jit": sglang_jit_store_cache,
        "torch_compile": torch_compile_store_cache,
        "torch_streams": torch_streams_store_cache,
    }

    def fn():
        impl = FN_MAP[provider]
        for i in range(NUM_LAYERS):
            impl(k[i], v[i], k_cache[i], v_cache[i], indices)

    # Custom time calculation: divide by NUM_LAYERS
    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
        fn, quantiles=DEFAULT_QUANTILES
    )
    return (
        1000 * ms / NUM_LAYERS,
        1000 * max_ms / NUM_LAYERS,
        1000 * min_ms / NUM_LAYERS,
    )


if __name__ == "__main__":
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `typing -> Tuple`
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> (`
- `sglang.jit_kernel.kvcache -> store_cache`
- `sglang.test.ci.ci_register -> register_cuda_ci`
