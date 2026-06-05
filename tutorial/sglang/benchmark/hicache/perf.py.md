# perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache perf. It primarily prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 hicache perf 相关流程。它主要用于准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Imports and setup / 导入与初始化
```python
from __future__ import annotations

from typing import Any, Callable, NamedTuple

import torch
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会准备张量并调用 GPU 内核。

### Lines 8-30: Function `jit_hicache_impl` / 函数 `jit_hicache_impl`
```python
def jit_hicache_impl(
    k_cache_dst: torch.Tensor,
    v_cache_dst: torch.Tensor,
    indices_dst: torch.Tensor,
    k_cache_src: torch.Tensor,
    v_cache_src: torch.Tensor,
    indices_src: torch.Tensor,
    item_bytes: int,
    block_quota: int,
) -> None:
    from sglang.jit_kernel.hicache import transfer_hicache_one_layer

    _ = item_bytes

    transfer_hicache_one_layer(
        k_cache_dst=k_cache_dst,
        v_cache_dst=v_cache_dst,
        indices_dst=indices_dst,
        k_cache_src=k_cache_src,
        v_cache_src=v_cache_src,
        indices_src=indices_src,
        block_quota=block_quota,
    )
```
**EN:** `jit_hicache_impl` is a function that prepares tensors and invokes GPU kernels. Notable calls include `transfer_hicache_one_layer`.
**CN:** `jit_hicache_impl` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `transfer_hicache_one_layer`。

### Lines 33-54: Function `ref_hicache_impl` / 函数 `ref_hicache_impl`
```python
def ref_hicache_impl(
    k_cache_dst: torch.Tensor,
    v_cache_dst: torch.Tensor,
    indices_dst: torch.Tensor,
    k_cache_src: torch.Tensor,
    v_cache_src: torch.Tensor,
    indices_src: torch.Tensor,
    item_bytes: int,
    block_quota: int,
) -> None:
    from sgl_kernel import transfer_kv_per_layer

    transfer_kv_per_layer(
        src_k=k_cache_src,
        src_v=v_cache_src,
        dst_k=k_cache_dst,
        dst_v=v_cache_dst,
        src_indices=indices_src,
        dst_indices=indices_dst,
        item_size=item_bytes,
        block_quota=block_quota,
    )
```
**EN:** `ref_hicache_impl` is a function that prepares tensors and invokes GPU kernels. Notable calls include `transfer_kv_per_layer`.
**CN:** `ref_hicache_impl` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `transfer_kv_per_layer`。

### Lines 57-57: Class `HicacheBenchArgs` declaration / 类 `HicacheBenchArgs` 声明
```python
class HicacheBenchArgs(NamedTuple):
```
**EN:** This block introduces class `HicacheBenchArgs`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `HicacheBenchArgs`，用于通过统一接口组织相关行为。

### Lines 58-60: Class-level state / 类级状态
```python
    cache_item_size: int
    dtype: torch.dtype
    block_quota: int
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it prepares tensors and invokes GPU kernels.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会准备张量并调用 GPU 内核。

### Lines 63-75: Function `perf` / 函数 `perf`
```python
def perf(f: Callable[[], Any], loop: int = 100) -> float:
    tic = torch.cuda.Event(enable_timing=True)
    toc = torch.cuda.Event(enable_timing=True)
    torch.cuda.synchronize()
    # warm up
    f()
    torch.cuda._sleep(10**8)
    tic.record()
    for _ in range(loop):
        f()
    toc.record()
    toc.synchronize()
    return tic.elapsed_time(toc) / loop
```
**EN:** `perf` is a function that prepares tensors and invokes GPU kernels. It returns `tic.elapsed_time(toc) / loop` to the caller. Notable calls include `torch.cuda.Event`, `torch.cuda.synchronize`, `f`.
**CN:** `perf` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `tic.elapsed_time(toc) / loop`。其中较关键的调用包括 `torch.cuda.Event`, `torch.cuda.synchronize`, `f`。

### Lines 79-207: Function `test_hicache_kernel` / 函数 `test_hicache_kernel`
```python
def test_hicache_kernel(args: HicacheBenchArgs) -> None:
    CACHE_ITEM_SIZE, DTYPE, BLOCK_QUOTA = args

    CUDA_CACHE_SIZE = 1024 * 1024
    HOST_CACHE_SIZE = CUDA_CACHE_SIZE * 2

    cuda_cache = torch.randn(
        (2, CUDA_CACHE_SIZE, CACHE_ITEM_SIZE),
        dtype=DTYPE,
        device="cuda",
    )
    host_cache = torch.empty(
        (2, HOST_CACHE_SIZE, CACHE_ITEM_SIZE),
        dtype=DTYPE,
        device="cpu",
        pin_memory=True,
    )

    ITEM_BYTES = cuda_cache.element_size() * CACHE_ITEM_SIZE

    def _gen_indices(size: int, bs: int) -> torch.Tensor:
        assert bs <= size
        result = (
            (torch.randperm(size, dtype=torch.int64, device="cuda")[:bs]).sort().values
        )
        if not (torch.all(result >= 0) and torch.all(result < size)):
            where = (result < 0) | (result >= size)
            place = where.nonzero(as_tuple=False)
            print("Invalid indices at positions:", place)
            print("Invalid indices values:", result[place])
            raise ValueError("Generated invalid indices")
        return result

    def _calc_tput(dur: float) -> float:
        return (MEM / (1024**3)) / (dur / 1000)  # GB/s

    def _gain_str(aot_dur: float, jit_dur: float) -> str:
        gain = 100 * (aot_dur / jit_dur - 1)
        if gain >= 0:
            return f"+{gain:>6.2f}%"
        else:
            return f"-{-gain:>6.2f}%"

    print(f"{CACHE_ITEM_SIZE = }, {DTYPE = }, {BLOCK_QUOTA = }")

    def _fast_test_correctness(bs: int):
        src_indices = _gen_indices(CUDA_CACHE_SIZE, bs)
        dst_indices = _gen_indices(HOST_CACHE_SIZE, bs)
        host_cache_cuda = torch.randn_like(host_cache, device="cuda")
        host_cache.copy_(host_cache_cuda, non_blocking=True)

        # copy from cuda to host
        jit_hicache_impl(
            k_cache_dst=host_cache[0],
            v_cache_dst=host_cache[1],
            indices_dst=dst_indices,
            k_cache_src=cuda_cache[0],
            v_cache_src=cuda_cache[1],
            indices_src=src_indices,
            item_bytes=ITEM_BYTES,
            block_quota=BLOCK_QUOTA,
        )
        dst_indices = dst_indices.cpu()
        assert torch.all(
            host_cache[0][dst_indices].cuda() == cuda_cache[0][src_indices]
        )

    BS_RANGE = [2**n for n in range(8, 18)]
    for bs in BS_RANGE:
        _fast_test_correctness(bs)

    print("Correctness passed! Start HiCache kernel performance test...")
    print("=" * 70)

    for bs in BS_RANGE:
        indices_dst = _gen_indices(CUDA_CACHE_SIZE, bs)
        indices_src = _gen_indices(HOST_CACHE_SIZE, bs)
        MEM = 2 * bs * ITEM_BYTES

        def _run_kernel_h2d(impl):
            return impl(
                k_cache_dst=cuda_cache[0],
                v_cache_dst=cuda_cache[1],
                indices_dst=indices_dst,
                k_cache_src=host_cache[0],
                v_cache_src=host_cache[1],
                indices_src=indices_src,
                item_bytes=ITEM_BYTES,
                block_quota=BLOCK_QUOTA,
            )

        our_h2d_dur = perf(lambda: _run_kernel_h2d(jit_hicache_impl))
        ref_h2d_dur = perf(lambda: _run_kernel_h2d(ref_hicache_impl))
        print(
            f"{bs = :6d}, H->D",
            f"| aot {_calc_tput(ref_h2d_dur):<6.2f} GB/s",
            f"| jit {_calc_tput(our_h2d_dur):<6.2f} GB/s",
            f"| {_gain_str(ref_h2d_dur, our_h2d_dur)}",
        )

    print("=" * 70)

    for bs in BS_RANGE:
        indices_dst = _gen_indices(HOST_CACHE_SIZE, bs)
        indices_src = _gen_indices(CUDA_CACHE_SIZE, bs)
        MEM = 2 * bs * ITEM_BYTES

        def _run_kernel_d2h(impl):
            return impl(
                k_cache_dst=host_cache[0],
                v_cache_dst=host_cache[1],
                indices_dst=indices_dst,
                k_cache_src=cuda_cache[0],
                v_cache_src=cuda_cache[1],
                indices_src=indices_src,
                item_bytes=ITEM_BYTES,
                block_quota=BLOCK_QUOTA,
            )

        our_d2h_dur = perf(lambda: _run_kernel_d2h(jit_hicache_impl))
        ref_d2h_dur = perf(lambda: _run_kernel_d2h(ref_hicache_impl))
        print(
            f"{bs = :6d}, D->H",
            f"| aot {_calc_tput(ref_d2h_dur):<6.2f} GB/s",
            f"| jit {_calc_tput(our_d2h_dur):<6.2f} GB/s",
            f"| {_gain_str(ref_d2h_dur, our_d2h_dur)}",
        )

    print("=" * 70)
```
**EN:** `test_hicache_kernel` is a function that prepares tensors and invokes GPU kernels. It returns `result` to the caller. Notable calls include `torch.inference_mode`, `torch.randn`, `torch.empty`.
**CN:** `test_hicache_kernel` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `result`。其中较关键的调用包括 `torch.inference_mode`, `torch.randn`, `torch.empty`。

### Lines 210-244: Function `main` / 函数 `main`
```python
def main() -> None:
    torch.cuda.set_device(0)
    stream = torch.cuda.Stream()
    torch.cuda.set_stream(stream)

    tic = torch.cuda.Event(enable_timing=True)
    toc = torch.cuda.Event(enable_timing=True)

    BUF_SIZE = 1024 * 1024 * 1024
    cuda_mem = torch.empty(BUF_SIZE, dtype=torch.uint8, device="cuda")
    host_mem = torch.empty(BUF_SIZE, dtype=torch.uint8, device="cpu", pin_memory=True)

    # test peak bandwidth
    tic.record()
    cuda_mem.copy_(host_mem, non_blocking=True)
    toc.record()
    toc.synchronize()
    dur = tic.elapsed_time(toc)
    print(f"Peak H->D Bandwidth: {(BUF_SIZE / (1024**3)) / (dur / 1000):.2f} GB/s")

    tic.record()
    host_mem.copy_(cuda_mem, non_blocking=True)
    toc.record()
    toc.synchronize()
    dur = tic.elapsed_time(toc)
    print(f"Peak D->H Bandwidth: {(BUF_SIZE / (1024**3)) / (dur / 1000):.2f} GB/s")

    for block_quota in [1, 2, 3, 4]:
        for cache_item_size in [128, 256, 512, 1024]:
            args = HicacheBenchArgs(
                cache_item_size=cache_item_size,
                dtype=torch.float16,
                block_quota=block_quota,
            )
            test_hicache_kernel(args)
```
**EN:** `main` is a function that prepares tensors and invokes GPU kernels. Notable calls include `torch.cuda.set_device`, `torch.cuda.Stream`, `torch.cuda.set_stream`.
**CN:** `main` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `torch.cuda.set_device`, `torch.cuda.Stream`, `torch.cuda.set_stream`。

### Lines 247-248: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方依赖**: `torch`, `sgl_kernel`
- **Internal / 项目内部依赖**: `sglang.jit_kernel.hicache`
