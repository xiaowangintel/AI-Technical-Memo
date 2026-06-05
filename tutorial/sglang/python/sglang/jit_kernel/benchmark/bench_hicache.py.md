# bench_hicache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_hicache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Benchmark for HiCache JIT kernel performance.". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Benchmark for HiCache JIT kernel performance.”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-46: Imports and module setup
```python
"""Benchmark for HiCache JIT kernel performance.

This benchmark tests the performance of KV cache transfer operations
between GPU and CPU (host pinned memory), comparing:
- SGL AOT Kernel: Pre-compiled transfer_kv kernels from sgl_kernel
- SGL JIT Kernel: JIT-compiled hicache kernels
- PyTorch Indexing: Plain PyTorch index copy
- PyTorch 2 Stream: PyTorch implementation using 2 CUDA streams

Tests cover:
- One Layer: CPU->GPU
- All Layer: GPU->CPU

Note: Uses do_bench instead of do_bench_cudagraph since CUDA graph
capture doesn't support CPU-GPU memory transfers.
"""

import itertools
import os
from dataclasses import dataclass
from typing import Tuple

import torch
import triton
import triton.testing
from sgl_kernel import transfer_kv_all_layer, transfer_kv_per_layer

from sglang.jit_kernel.benchmark.utils import DEFAULT_QUANTILES, get_benchmark_range
from sglang.jit_kernel.hicache import (
    can_use_hicache_jit_kernel,
    transfer_hicache_all_layer,
    transfer_hicache_one_layer,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=29, suite="base-b-kernel-benchmark-1-gpu-large")

DISABLE_TORCH = os.environ.get("DISABLE_TORCH", "0") == "1"
PAGE_SIZE = 1
ENABLE_SORT = True
GPU_CACHE_SIZE = 256 * 1024  # 256K tokens on GPU
HOST_CACHE_SIZE = 512 * 1024  # 512K tokens on CPU
NUM_LAYERS = 8


@dataclass(frozen=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 47-69: Class `HiCacheCache`
```python
class HiCacheCache:
    k_cache_cuda: torch.Tensor
    v_cache_cuda: torch.Tensor
    k_cache_host: torch.Tensor
    v_cache_host: torch.Tensor

    def get_slice(self, num_layers: int, element_size: int) -> "HiCacheCache":
        def slice_cuda(t: torch.Tensor) -> torch.Tensor:
            needed_cuda = num_layers * GPU_CACHE_SIZE
            return t.view(-1, element_size)[:needed_cuda].unflatten(0, (num_layers, -1))

        def slice_host(t: torch.Tensor) -> torch.Tensor:
            needed_host = num_layers * HOST_CACHE_SIZE
            return t.view(-1, element_size)[:needed_host].unflatten(0, (num_layers, -1))

        return HiCacheCache(
            k_cache_cuda=slice_cuda(self.k_cache_cuda),
            v_cache_cuda=slice_cuda(self.v_cache_cuda),
            k_cache_host=slice_host(self.k_cache_host),
            v_cache_host=slice_host(self.v_cache_host),
        )
```
**EN:** This block declares the `HiCacheCache` class and establishes the behavior or state it encapsulates. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `HiCacheCache` 类，并建立其封装的行为或状态。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 70-81: Function `gen_indices`
```python
def gen_indices(
    size: int, max_size: int, *, page_size: int = PAGE_SIZE
) -> torch.Tensor:
    def align(x: int) -> int:
        return (x + page_size - 1) // page_size

    assert size <= max_size and max_size % page_size == 0
    indices = torch.randperm(align(max_size))[: align(size)]
    offsets = torch.arange(page_size)
    return (indices[:, None] * page_size + offsets).flatten().cuda()[:size]
```
**EN:** This block defines `gen_indices`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gen_indices`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 82-102: Function `sglang_aot_transfer_one`
```python
def sglang_aot_transfer_one(
    k_cache_dst: torch.Tensor,
    v_cache_dst: torch.Tensor,
    indices_dst: torch.Tensor,
    k_cache_src: torch.Tensor,
    v_cache_src: torch.Tensor,
    indices_src: torch.Tensor,
    item_size: int,
) -> None:
    """SGL AOT Kernel for single layer transfer."""
    transfer_kv_per_layer(
        k_cache_src,
        k_cache_dst,
        v_cache_src,
        v_cache_dst,
        indices_src,
        indices_dst,
        item_size,
    )
```
**EN:** This block defines `sglang_aot_transfer_one`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_aot_transfer_one`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 103-123: Function `sglang_jit_transfer_one`
```python
def sglang_jit_transfer_one(
    k_cache_dst: torch.Tensor,
    v_cache_dst: torch.Tensor,
    indices_dst: torch.Tensor,
    k_cache_src: torch.Tensor,
    v_cache_src: torch.Tensor,
    indices_src: torch.Tensor,
    element_dim: int,
) -> None:
    """SGL JIT Kernel for single layer transfer."""
    transfer_hicache_one_layer(
        k_cache_dst,
        v_cache_dst,
        indices_dst,
        k_cache_src,
        v_cache_src,
        indices_src,
        element_dim=element_dim,
    )
```
**EN:** This block defines `sglang_jit_transfer_one`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_jit_transfer_one`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 124-146: Function `sglang_aot_transfer_all`
```python
def sglang_aot_transfer_all(
    k_ptrs_dst: torch.Tensor,
    v_ptrs_dst: torch.Tensor,
    indices_dst: torch.Tensor,
    k_ptrs_src: torch.Tensor,
    v_ptrs_src: torch.Tensor,
    indices_src: torch.Tensor,
    item_size: int,
    num_layers: int,
) -> None:
    """SGL AOT Kernel for all layer transfer."""
    transfer_kv_all_layer(
        k_ptrs_src,
        k_ptrs_dst,
        v_ptrs_src,
        v_ptrs_dst,
        indices_src,
        indices_dst,
        item_size,
        num_layers,
    )
```
**EN:** This block defines `sglang_aot_transfer_all`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_aot_transfer_all`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 147-170: Function `sglang_jit_transfer_all`
```python
def sglang_jit_transfer_all(
    k_ptrs_dst: torch.Tensor,
    v_ptrs_dst: torch.Tensor,
    indices_dst: torch.Tensor,
    k_ptrs_src: torch.Tensor,
    v_ptrs_src: torch.Tensor,
    indices_src: torch.Tensor,
    stride_bytes: int,
    element_size: int,
) -> None:
    """SGL JIT Kernel for all layer transfer."""
    transfer_hicache_all_layer(
        k_ptrs_dst,
        v_ptrs_dst,
        indices_dst,
        k_ptrs_src,
        v_ptrs_src,
        indices_src,
        kv_cache_src_stride_bytes=stride_bytes,
        kv_cache_dst_stride_bytes=stride_bytes,
        element_size=element_size,
    )
```
**EN:** This block defines `sglang_jit_transfer_all`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_jit_transfer_all`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 171-220: Function `pytorch_transfer`
```python
def pytorch_transfer(
    k_cache_dst: torch.Tensor,
    v_cache_dst: torch.Tensor,
    indices_dst_on_dst: torch.Tensor,
    k_cache_src: torch.Tensor,
    v_cache_src: torch.Tensor,
    indices_src_on_src: torch.Tensor,
) -> None:
    """PyTorch indexing baseline."""
    dst_device = k_cache_dst.device
    k_cache_dst[indices_dst_on_dst] = k_cache_src[indices_src_on_src].to(dst_device)
    v_cache_dst[indices_dst_on_dst] = v_cache_src[indices_src_on_src].to(dst_device)


# Benchmark configuration

BS_RANGE = get_benchmark_range(
    full_range=[2**n for n in range(0, 16)],
    ci_range=[16],
)
ELEMENT_SIZE_RANGE = get_benchmark_range(
    full_range=[64, 128, 256, 512, 1024],
    ci_range=[1024],
)

LINE_VALS = ["aot", "jit", "torch"]
LINE_NAMES = ["SGL AOT Kernel", "SGL JIT Kernel", "PyTorch"]
STYLES = [("orange", "-"), ("blue", "--"), ("red", ":")]

CONFIGS = list(itertools.product(ELEMENT_SIZE_RANGE, BS_RANGE))


# =============================================================================
# One Layer Benchmarks
# =============================================================================


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["element_size", "batch_size"],
        x_vals=CONFIGS,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="hicache-one-layer-h2d",
        args={},
    )
)
```
**EN:** This block defines `pytorch_transfer`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `pytorch_transfer`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 221-301: Function `benchmark_one_layer_h2d`
```python
def benchmark_one_layer_h2d(
    element_size: int, batch_size: int, provider: str
) -> Tuple[float, float, float]:
    """One Layer: Host (CPU) -> Device (GPU)."""
    global cache
    cache_local = cache.get_slice(num_layers=NUM_LAYERS, element_size=element_size)
    k_cache_src = cache_local.k_cache_host
    v_cache_src = cache_local.v_cache_host
    k_cache_dst = cache_local.k_cache_cuda
    v_cache_dst = cache_local.v_cache_cuda
    torch.manual_seed(batch_size * 65536 + element_size)
    indices_src_gpu = gen_indices(batch_size, HOST_CACHE_SIZE)
    indices_dst_gpu = gen_indices(batch_size, GPU_CACHE_SIZE)

    if ENABLE_SORT:
        indices_src_gpu, mapping = indices_src_gpu.sort()
        indices_dst_gpu = indices_dst_gpu[mapping]
    indices_src_cpu = indices_src_gpu.cpu()
    torch.cuda.synchronize()

    element_bytes = element_size * k_cache_src.element_size()

    FN_MAP = {
        "aot": lambda: [
            sglang_aot_transfer_one(
                k_cache_dst[i],
                v_cache_dst[i],
                indices_dst_gpu,
                k_cache_src[i],
                v_cache_src[i],
                indices_src_gpu,
                element_bytes,
            )
            for i in range(NUM_LAYERS)
        ],
        "jit": lambda: [
            sglang_jit_transfer_one(
                k_cache_dst[i],
                v_cache_dst[i],
                indices_dst_gpu,
                k_cache_src[i],
                v_cache_src[i],
                indices_src_gpu,
                element_size,
            )
            for i in range(NUM_LAYERS)
        ],
        "torch": lambda: [
            pytorch_transfer(
                k_cache_dst[i],
                v_cache_dst[i],
                indices_dst_gpu,
                k_cache_src[i],
                v_cache_src[i],
                indices_src_cpu,
            )
            for i in range(NUM_LAYERS)
        ],
    }

    if provider == "jit" and not can_use_hicache_jit_kernel(element_size=element_bytes):
        return (float("nan"), float("nan"), float("nan"))

    if DISABLE_TORCH and provider in ["torch"]:
        return (float("nan"), float("nan"), float("nan"))

    ms, min_ms, max_ms = triton.testing.do_bench(  # type: ignore
        FN_MAP[provider], quantiles=DEFAULT_QUANTILES, warmup=5, rep=25
    )
    return (
        1000 * ms / NUM_LAYERS,
        1000 * max_ms / NUM_LAYERS,
        1000 * min_ms / NUM_LAYERS,
    )


# =============================================================================
# All Layer Benchmarks
# =============================================================================

# ...
```
**EN:** This block defines `benchmark_one_layer_h2d`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark_one_layer_h2d`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 302-323: Function `_create_ptr_tensor`
```python
def _create_ptr_tensor(tensors, device="cuda"):
    """Create a tensor of data pointers."""
    return torch.tensor(
        [t.data_ptr() for t in tensors],
        dtype=torch.uint64,
        device=device,
    )


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["element_size", "batch_size"],
        x_vals=CONFIGS,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="hicache-all-layer-d2h",
        args={},
    )
)
```
**EN:** This block defines `_create_ptr_tensor`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_create_ptr_tensor`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 324-421: Function `benchmark_all_layer_d2h`
```python
def benchmark_all_layer_d2h(
    element_size: int, batch_size: int, provider: str
) -> Tuple[float, float, float]:
    """All Layer: Device (GPU) -> Host (CPU)."""
    global cache
    cache_local = cache.get_slice(num_layers=NUM_LAYERS, element_size=element_size)
    k_caches_src = cache_local.k_cache_cuda
    v_caches_src = cache_local.v_cache_cuda
    k_caches_dst = cache_local.k_cache_host
    v_caches_dst = cache_local.v_cache_host
    torch.manual_seed(batch_size * 65536 + element_size)

    indices_src_gpu = gen_indices(batch_size, GPU_CACHE_SIZE)
    indices_dst_gpu = gen_indices(batch_size, HOST_CACHE_SIZE)
    if ENABLE_SORT:
        indices_dst_gpu, mapping = indices_dst_gpu.sort()
        indices_src_gpu = indices_src_gpu[mapping]
    indices_dst_cpu = indices_dst_gpu.cpu()
    torch.cuda.synchronize()

    element_bytes = element_size * k_caches_src.element_size()

    k_ptrs_src = _create_ptr_tensor([k_caches_src[i] for i in range(NUM_LAYERS)])
    v_ptrs_src = _create_ptr_tensor([v_caches_src[i] for i in range(NUM_LAYERS)])
    k_ptrs_dst = _create_ptr_tensor([k_caches_dst[i] for i in range(NUM_LAYERS)])
    v_ptrs_dst = _create_ptr_tensor([v_caches_dst[i] for i in range(NUM_LAYERS)])

    FN_MAP = {
        "aot": lambda: sglang_aot_transfer_all(
            k_ptrs_dst,
            v_ptrs_dst,
            indices_dst_gpu,
            k_ptrs_src,
            v_ptrs_src,
            indices_src_gpu,
            element_bytes,
            NUM_LAYERS,
        ),
        "jit": lambda: sglang_jit_transfer_all(
            k_ptrs_dst,
            v_ptrs_dst,
            indices_dst_gpu,
            k_ptrs_src,
            v_ptrs_src,
            indices_src_gpu,
            element_bytes,
            element_bytes,
        ),
        "torch": lambda: [
            pytorch_transfer(
                k_caches_dst[i],
                v_caches_dst[i],
                indices_dst_cpu,
                k_caches_src[i],
                v_caches_src[i],
                indices_src_gpu,
            )
            for i in range(NUM_LAYERS)
        ],
    }

    if provider == "jit" and not can_use_hicache_jit_kernel(element_size=element_bytes):
        return (float("nan"), float("nan"), float("nan"))

    if DISABLE_TORCH and provider in ["torch"]:
        return (float("nan"), float("nan"), float("nan"))

    ms, min_ms, max_ms = triton.testing.do_bench(  # type: ignore
        FN_MAP[provider], quantiles=DEFAULT_QUANTILES, warmup=5, rep=25
    )
    return (
        1000 * ms / NUM_LAYERS,
        1000 * max_ms / NUM_LAYERS,
        1000 * min_ms / NUM_LAYERS,
    )


if __name__ == "__main__":
    MAX_SIZE = max(ELEMENT_SIZE_RANGE)
    DEVICE_SHAPE = (NUM_LAYERS * GPU_CACHE_SIZE, MAX_SIZE)
# ...
```
**EN:** This block defines `benchmark_all_layer_d2h`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark_all_layer_d2h`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `os`
- `dataclasses -> dataclass`
- `typing -> Tuple`
- `torch`
- `triton`
- `triton.testing`
- `sgl_kernel -> transfer_kv_all_layer`
- `sglang.jit_kernel.benchmark.utils -> DEFAULT_QUANTILES`
- `sglang.jit_kernel.hicache -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
