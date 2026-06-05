# bench_hadamard.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_hadamard.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports and module setup
```python
import itertools
import math
from typing import Tuple

import torch
import torch.nn.functional as F
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import (
    DEFAULT_DEVICE,
    DEFAULT_DTYPE,
    get_benchmark_range,
    run_benchmark,
)
from sglang.jit_kernel.hadamard import hadamard_transform
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=5, suite="base-b-kernel-benchmark-1-gpu-large")

# AOT kernel: might not be available in all environments.
# This is used for performance baseline comparison.
try:
    from sgl_kernel import hadamard_transform as hadamard_transform_aot

    AOT_AVAILABLE = True
except Exception:
    AOT_AVAILABLE = False

# Naive reference implementation using scipy hadamard matrix.
try:
    from scipy.linalg import hadamard

    SCIPY_AVAILABLE = True
except ImportError:
    SCIPY_AVAILABLE = False

# CI environment uses simplified parameters
batch_sizes = get_benchmark_range(
    full_range=[1, 16, 64, 256],
    ci_range=[16],
)
dim_range = get_benchmark_range(
    full_range=[64, 256, 1024, 4096, 8192, 16384, 32768],
    ci_range=[1024],
)


# Naive reference implementation using precomputed scipy hadamard matrix.
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 50-87: Function `torch_hadamard_transform`
```python
def torch_hadamard_transform(x, scale, H, dim, dim_padded):
    flat = x.reshape(-1, dim)
    if dim != dim_padded:
        flat = F.pad(flat, (0, dim_padded - dim))
    out = F.linear(flat, H) * scale
    return out[..., :dim].reshape(x.shape)


available_providers = ["jit_kernel"]
available_names = ["JIT Kernel"]
available_styles = [("red", "-")]

if AOT_AVAILABLE:
    available_providers.insert(0, "aot_kernel")
    available_names.insert(0, "AOT Kernel")
    available_styles.insert(0, ("green", "-"))

if SCIPY_AVAILABLE:
    available_providers.append("naive")
    available_names.append("Naive (scipy)")
    available_styles.append(("blue", "-"))

configs = list(itertools.product(batch_sizes, dim_range))


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "dim"],
        x_vals=[list(c) for c in configs],
        line_arg="provider",
        line_vals=available_providers,
        line_names=available_names,
        styles=available_styles,
        ylabel="us",
        plot_name="hadamard-transform-performance",
        args={},
    )
)
```
**EN:** This block defines `torch_hadamard_transform`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `torch_hadamard_transform`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 88-119: Function `benchmark`
```python
def benchmark(batch_size: int, dim: int, provider: str) -> Tuple[float, float, float]:
    scale = 1.0 / math.sqrt(dim)
    x = torch.randn(batch_size, dim, device=DEFAULT_DEVICE, dtype=DEFAULT_DTYPE)

    FN_MAP = {
        "jit_kernel": lambda: hadamard_transform(x.clone(), scale=scale),
    }
    if AOT_AVAILABLE:
        FN_MAP["aot_kernel"] = lambda: hadamard_transform_aot(x.clone(), scale=scale)
    if SCIPY_AVAILABLE:
        # Precompute Hadamard matrix on GPU to avoid CPU-GPU transfer
        # during CUDA graph capture.
        log_dim = math.ceil(math.log2(dim)) if dim > 0 else 0
        dim_padded = 2**log_dim if dim > 0 else 1
        H = torch.tensor(
            hadamard(dim_padded, dtype=float),
            dtype=DEFAULT_DTYPE,
            device=DEFAULT_DEVICE,
        )
        FN_MAP["naive"] = lambda: torch_hadamard_transform(
            x.clone(), scale, H, dim, dim_padded
        )

    fn = FN_MAP[provider]
    return run_benchmark(fn)


if __name__ == "__main__":
    print("=" * 80)
    print("Benchmarking Fast Hadamard Transform")
    print("=" * 80)
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `math`
- `typing -> Tuple`
- `torch`
- `torch.nn.functional as F`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> (`
- `sglang.jit_kernel.hadamard -> hadamard_transform`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sgl_kernel -> hadamard_transform`
- `scipy.linalg -> hadamard`
