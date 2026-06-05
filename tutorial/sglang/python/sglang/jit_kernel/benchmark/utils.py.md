# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Common utilities for jit_kernel benchmark files."""". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Common utilities for jit_kernel benchmark files."""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup
```python
"""Common utilities for jit_kernel benchmark files."""

from typing import Callable, List, Sequence, Tuple

import torch
import triton.testing

from sglang.utils import is_in_ci

# Common constants
DEFAULT_DTYPE = torch.bfloat16
DEFAULT_DEVICE = "cuda"
DEFAULT_QUANTILES = [0.5, 0.2, 0.8]
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 16-20: Function `get_benchmark_range`
```python
def get_benchmark_range(full_range: List, ci_range: List) -> List:
    """Return appropriate benchmark range based on CI environment."""
    return ci_range if is_in_ci() else full_range
```
**EN:** This block defines `get_benchmark_range`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_benchmark_range`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 21-40: Function `run_benchmark`
```python
def run_benchmark(
    fn: Callable,
    quantiles: Sequence[float] = (),
    scale: float = 1.0,
) -> Tuple[float, float, float]:
    """Execute benchmark using CUDA graph and return times in microseconds.

    Args:
        fn: Function to benchmark
        quantiles: Quantiles for timing measurements [median, min, max]
        scale: Scale the result down (usually num_layers).

    Returns:
        Tuple of (median_us, max_us, min_us)
    """
    quantiles = list(quantiles or DEFAULT_QUANTILES)
    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles)
    return 1000 * ms / scale, 1000 * max_ms / scale, 1000 * min_ms / scale
```
**EN:** This block defines `run_benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 41-48: Function `run_benchmark_no_cudagraph`
```python
def run_benchmark_no_cudagraph(
    fn: Callable,
    quantiles: Sequence[float] = (),
    scale: float = 1.0,
) -> Tuple[float, float, float]:
    quantiles = list(quantiles or DEFAULT_QUANTILES)
    ms, min_ms, max_ms = triton.testing.do_bench(fn, quantiles=quantiles)
    return 1000 * ms / scale, 1000 * max_ms / scale, 1000 * min_ms / scale
```
**EN:** This block defines `run_benchmark_no_cudagraph`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_benchmark_no_cudagraph`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `typing -> Callable`
- `torch`
- `triton.testing`
- `sglang.utils -> is_in_ci`
