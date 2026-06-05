# 09-persistent-matmul.py — Code Analysis / 代码分析

## Source / 来源

- **EN:** Source file: `/root/xw/triton/python/tutorials/09-persistent-matmul.py`. This analysis is written against the current file contents without modifying the source.
- **CN:** 源文件：`/root/xw/triton/python/tutorials/09-persistent-matmul.py`。本文基于当前源码内容编写，未修改源文件。
- **EN:** Output file: `/root/xw/triton/tutorial/python/tutorials/09-persistent-matmul.py.md`.
- **CN:** 输出文件：`/root/xw/triton/tutorial/python/tutorials/09-persistent-matmul.py.md`。

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-87): Module overview, imports, runtime detection, and launch metadata

```python
"""
Persistent Matmul
=====================
This script demonstrates persistent kernel implementations of matrix multiplication using Triton.
Various matmul methods are included, such as naive, persistent, and TMA (Tensor Memory Accelerator) based approaches.
The kernels support both FP16 and FP8 data types but the FP8 implementation is only available on CUDA devices with compute capability >= 9.0.

Triton and cuBLAS implementations are benchmarked under different configurations and evaluated using the proton profiler.
Users can pass command-line arguments to specify matrix dimensions and iteration steps flexibly.

.. code-block:: bash

    # FP8
    python 09-persistent-matmul.py --prec fp8 --K_range 128 1024 --K_step 128

    # FP16
    python 09-persistent-matmul.py --prec fp16 --K_range 128 1024 --K_step 128

Note that currently this tutorial will fail on devices with a small shared memory size, such as RTX-4090.
"""

import argparse
import itertools

import torch
import triton
import triton.language as tl
import triton.profiler as proton
from triton.tools.tensor_descriptor import TensorDescriptor
from contextlib import contextmanager

from typing import Optional


def is_cuda():
    return triton.runtime.driver.active.get_current_target().backend == "cuda"


def is_hip():
    return triton.runtime.driver.active.get_current_target().backend == "hip"


if is_cuda():
    from triton._C.libtriton import nvidia
    device_workspace = torch.empty(32 * 1024 * 1024, device="cuda", dtype=torch.uint8)
    device_blas = nvidia.cublas.CublasLt(device_workspace)
elif is_hip():
    from triton._C.libtriton import amd
    device_workspace = torch.empty(32 * 1024 * 1024, device="cuda", dtype=torch.uint8)
    device_blas = amd.hipblas.HipblasLt(device_workspace)
else:
    device_blas = None


def device_blas_name():
    return 'cuBLAS' if is_cuda() else 'hipBLAS'


def supports_tma():
    return is_cuda() and torch.cuda.get_device_capability()[0] >= 9


def is_hopper():
    return torch.cuda.get_device_capability()[0] == 9


def supports_ws():
    return is_cuda() and torch.cuda.get_device_capability()[0] >= 9


def _matmul_launch_metadata(grid, kernel, args):
    ret = {}
    M, N, K, WS = args["M"], args["N"], args["K"], args.get("WARP_SPECIALIZE", False)
    ws_str = "_ws" if WS else ""
    ret["name"] = f"{kernel.name}{ws_str} [M={M}, N={N}, K={K}]"
    if "c_ptr" in args:
        bytes_per_elem = args["c_ptr"].element_size()
    else:
        bytes_per_elem = 1 if args["FP8_OUTPUT"] else 2
    ret[f"flops{bytes_per_elem * 8}"] = 2. * M * N * K
    ret["bytes"] = bytes_per_elem * (M * K + N * K + M * N)
    return ret


HAS_TENSOR_DESC = supports_tma() and hasattr(tl, "make_tensor_descriptor")
HAS_HOST_TENSOR_DESC = supports_tma() and hasattr(triton.tools.tensor_descriptor, "TensorDescriptor")
HAS_WARP_SPECIALIZE = supports_ws() and HAS_TENSOR_DESC
```
**EN:** This opening section explains the tutorial purpose, imports PyTorch/Triton/profiler utilities, detects whether the active backend is CUDA or HIP, initializes the vendor BLAS wrapper, and defines small capability helpers. `_matmul_launch_metadata` standardizes profiling metadata so each kernel launch records human-readable names, FLOP counts, and byte traffic. The final flags (`HAS_TENSOR_DESC`, `HAS_HOST_TENSOR_DESC`, `HAS_WARP_SPECIALIZE`) gate later code paths based on hardware and API availability.
**CN:** 开头部分先说明教程目标，然后导入 PyTorch、Triton 与 profiler 工具，检测当前后端是 CUDA 还是 HIP，并初始化对应的厂商 BLAS 封装。几个辅助函数用于判断 TMA、Hopper、warp specialization 等能力。`_matmul_launch_metadata` 统一了 kernel 启动时的性能元数据，便于 profiler 记录可读名称、FLOP 数和访存字节数。最后的几个布尔标志决定后续哪些实现路径可以启用。

### Block 2 (Lines 90-160): Autotuned baseline matmul kernel

```python
def matmul_get_configs(pre_hook=None):
    return [
        triton.Config({'BLOCK_SIZE_M': BM, 'BLOCK_SIZE_N': BN, "BLOCK_SIZE_K": BK, "GROUP_SIZE_M": 8}, num_stages=s,
                      num_warps=w, pre_hook=pre_hook)
        for BM in [128]
        for BN in [128, 256]
        for BK in [64, 128]
        for s in ([2, 3, 4])
        for w in [4, 8]
    ]


@triton.autotune(
    configs=matmul_get_configs(),
    key=["M", "N", "K"],
)
@triton.jit(launch_metadata=_matmul_launch_metadata)
def matmul_kernel(a_ptr, b_ptr, c_ptr,  #
                  M, N, K,  #
                  stride_am, stride_ak,  #
                  stride_bk, stride_bn,  #
                  stride_cm, stride_cn,  #
                  BLOCK_SIZE_M: tl.constexpr,  #
                  BLOCK_SIZE_N: tl.constexpr,  #
                  BLOCK_SIZE_K: tl.constexpr,  #
                  GROUP_SIZE_M: tl.constexpr,  #
                  ):
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    num_pid_in_group = GROUP_SIZE_M * num_pid_n
    group_id = pid // num_pid_in_group
    first_pid_m = group_id * GROUP_SIZE_M
    group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
    pid_m = first_pid_m + (pid % group_size_m)
    pid_n = (pid % num_pid_in_group) // group_size_m

    start_m = pid_m * BLOCK_SIZE_M
    start_n = pid_n * BLOCK_SIZE_N

    offs_am = start_m + tl.arange(0, BLOCK_SIZE_M)
    offs_bn = start_n + tl.arange(0, BLOCK_SIZE_N)
    offs_am = tl.where(offs_am < M, offs_am, 0)
    offs_bn = tl.where(offs_bn < N, offs_bn, 0)

    offs_am = tl.max_contiguous(tl.multiple_of(offs_am, BLOCK_SIZE_M), BLOCK_SIZE_M)
    offs_bn = tl.max_contiguous(tl.multiple_of(offs_bn, BLOCK_SIZE_N), BLOCK_SIZE_N)
    offs_k = tl.arange(0, BLOCK_SIZE_K)
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

    accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)

    for k in range(0, tl.cdiv(K, BLOCK_SIZE_K)):
        a = tl.load(a_ptrs, mask=offs_k[None, :] < K - k * BLOCK_SIZE_K, other=0.0)
        b = tl.load(b_ptrs, mask=offs_k[:, None] < K - k * BLOCK_SIZE_K, other=0.0)
        accumulator = tl.dot(a, b, accumulator)
        a_ptrs += BLOCK_SIZE_K * stride_ak
        b_ptrs += BLOCK_SIZE_K * stride_bk

    if (c_ptr.dtype.element_ty == tl.float8e4nv):
        c = accumulator.to(tl.float8e4nv)
    else:
        c = accumulator.to(tl.float16)

    offs_cm = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
    offs_cn = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
    c_ptrs = c_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(c_ptrs, c, mask=c_mask)

```
**EN:** `matmul_get_configs` enumerates autotuning candidates over block sizes, pipeline stages, and warp counts. `matmul_kernel` is the baseline pointer-based GEMM kernel: it maps a 1D program id to grouped M/N tiles, computes pointer grids for A and B, iterates over K tiles, accumulates in FP32, converts to FP16 or FP8, and writes the masked result tile to C. This block is the reference implementation that later persistent and TMA variants optimize.
**CN:** `matmul_get_configs` 枚举了 autotune 的候选配置，包括分块大小、流水级数和 warp 数量。`matmul_kernel` 是基线版的指针式 GEMM kernel：它把一维 program id 映射到分组后的 M/N tile，计算 A、B 的指针网格，沿 K 维分块循环，在 FP32 中累加，最后转成 FP16 或 FP8 并带掩码写回 C。后续的 persistent 与 TMA 版本都是在这一基础逻辑上继续优化。

### Block 3 (Lines 162-180): Baseline Python wrapper

```python
def matmul(a, b):
    # Check constraints.
    assert a.shape[1] == b.shape[0], "Incompatible dimensions"
    assert a.dtype == b.dtype, "Incompatible dtypes"
    M, K = a.shape
    K, N = b.shape
    dtype = a.dtype

    c = torch.empty((M, N), device=a.device, dtype=dtype)
    # 1D launch kernel where each block gets its own program.
    grid = lambda META: (triton.cdiv(M, META["BLOCK_SIZE_M"]) * triton.cdiv(N, META["BLOCK_SIZE_N"]), )
    matmul_kernel[grid](
        a, b, c,  #
        M, N, K,  #
        a.stride(0), a.stride(1),  #
        b.stride(0), b.stride(1),  #
        c.stride(0), c.stride(1),  #
    )
    return c
```
**EN:** `matmul` is the user-facing wrapper for the baseline kernel. It validates shapes and dtypes, allocates the output tensor, defines a 1D launch grid from the tile geometry, and launches `matmul_kernel` with the correct strides. This isolates Triton launch details from the rest of the tutorial.
**CN:** `matmul` 是基线 kernel 的 Python 包装层。它先检查矩阵维度和数据类型，分配输出张量，再根据 tile 划分定义一维 grid，并把正确的 stride 传给 `matmul_kernel`。这样其余代码只需要调用高层接口，不必直接处理 Triton 启动细节。

### Block 4 (Lines 183-240): TMA descriptor hook and non-persistent TMA kernel

```python
def matmul_tma_set_block_size_hook(nargs):
    EPILOGUE_SUBTILE = nargs.get("EPILOGUE_SUBTILE", False)
    BLOCK_M = nargs["BLOCK_SIZE_M"]
    BLOCK_N = nargs["BLOCK_SIZE_N"]
    BLOCK_K = nargs["BLOCK_SIZE_K"]
    nargs["a_desc"].block_shape = [BLOCK_M, BLOCK_K]
    nargs["b_desc"].block_shape = [BLOCK_N, BLOCK_K]
    if EPILOGUE_SUBTILE:
        nargs["c_desc"].block_shape = [BLOCK_M, BLOCK_N // 2]
    else:
        nargs["c_desc"].block_shape = [BLOCK_M, BLOCK_N]


@triton.autotune(
    configs=matmul_get_configs(pre_hook=matmul_tma_set_block_size_hook),
    key=["M", "N", "K", "WARP_SPECIALIZE"],
)
@triton.jit(launch_metadata=_matmul_launch_metadata)
def matmul_kernel_tma(a_desc, b_desc, c_desc,  #
                      M, N, K,  #
                      BLOCK_SIZE_M: tl.constexpr,  #
                      BLOCK_SIZE_N: tl.constexpr,  #
                      BLOCK_SIZE_K: tl.constexpr,  #
                      GROUP_SIZE_M: tl.constexpr,  #
                      FP8_OUTPUT: tl.constexpr,  #
                      WARP_SPECIALIZE: tl.constexpr,  #
                      ):
    dtype = tl.float8e4nv if FP8_OUTPUT else tl.float16

    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    num_pid_in_group = GROUP_SIZE_M * num_pid_n
    group_id = pid // num_pid_in_group
    first_pid_m = group_id * GROUP_SIZE_M
    group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
    pid_m = first_pid_m + (pid % group_size_m)
    pid_n = (pid % num_pid_in_group) // group_size_m

    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)

    offs_am = pid_m * BLOCK_SIZE_M
    offs_bn = pid_n * BLOCK_SIZE_N

    accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)

    for k in tl.range(k_tiles, warp_specialize=WARP_SPECIALIZE):
        offs_k = k * BLOCK_SIZE_K
        a = a_desc.load([offs_am, offs_k])
        b = b_desc.load([offs_bn, offs_k])
        accumulator = tl.dot(a, b.T, accumulator)

    c = accumulator.to(dtype)

    offs_cm = pid_m * BLOCK_SIZE_M
    offs_cn = pid_n * BLOCK_SIZE_N
    c_desc.store([offs_cm, offs_cn], c)

```
**EN:** `matmul_tma_set_block_size_hook` is a pre-hook used by autotuning: once a configuration is chosen, it rewrites the tensor-descriptor block shapes to match that configuration, including the split epilogue case. `matmul_kernel_tma` replaces raw pointer arithmetic with descriptor loads/stores, using TMA-friendly access patterns and optional warp specialization while keeping the same tiled GEMM structure.
**CN:** `matmul_tma_set_block_size_hook` 是 autotune 的预处理钩子：一旦某个配置被选中，它会把张量描述符的 block shape 改成该配置对应的值，并处理 epilogue subtiling 的情况。`matmul_kernel_tma` 不再直接做裸指针寻址，而是通过 descriptor 进行加载与存储，使用更适合 TMA 的访问方式，并可选启用 warp specialization，同时保持相同的分块 GEMM 结构。

### Block 5 (Lines 242-270): Host-side TMA wrapper

```python
def matmul_tma(a, b, warp_specialize: bool):
    # Check constraints.
    assert a.shape[1] == b.shape[1], "Incompatible dimensions"  # b is transposed
    assert a.dtype == b.dtype, "Incompatible dtypes"

    M, K = a.shape
    N, K = b.shape
    dtype = a.dtype

    c = torch.empty((M, N), device=a.device, dtype=dtype)

    # A dummy block value that will be overwritten when we have the real block size
    dummy_block = [1, 1]
    a_desc = TensorDescriptor.from_tensor(a, dummy_block)
    b_desc = TensorDescriptor.from_tensor(b, dummy_block)
    c_desc = TensorDescriptor.from_tensor(c, dummy_block)

    def grid(META):
        BLOCK_M = META["BLOCK_SIZE_M"]
        BLOCK_N = META["BLOCK_SIZE_N"]
        return (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), )

    matmul_kernel_tma[grid](
        a_desc, b_desc, c_desc,  #
        M, N, K,  #
        FP8_OUTPUT=dtype == torch.float8_e4m3fn,  #
        WARP_SPECIALIZE=warp_specialize,  #
    )
    return c
```
**EN:** `matmul_tma` adapts the interface for descriptor-based TMA execution. Because the kernel expects B in transposed layout, the wrapper validates `a.shape[1] == b.shape[1]`, builds dummy `TensorDescriptor` objects that the pre-hook will later resize, defines the grid, and launches `matmul_kernel_tma` with output-type and warp-specialization flags.
**CN:** `matmul_tma` 是基于 descriptor 的 TMA 路径包装层。由于该 kernel 期望传入的是转置后的 B，因此它检查 `a.shape[1] == b.shape[1]`，先用占位 block 大小创建 `TensorDescriptor`，再交给 pre-hook 在真正运行时改写，随后定义 grid，并带着输出类型和 warp specialization 开关启动 `matmul_kernel_tma`。

### Block 6 (Lines 273-344): Tile-id helper and pointer-based persistent kernel

```python
@triton.jit
def _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS):
    group_id = tile_id // num_pid_in_group
    first_pid_m = group_id * GROUP_SIZE_M
    group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
    pid_m = first_pid_m + (tile_id % group_size_m)
    pid_n = (tile_id % num_pid_in_group) // group_size_m
    return pid_m, pid_n


@triton.autotune(
    configs=matmul_get_configs(),
    key=["M", "N", "K"],
)
@triton.jit(launch_metadata=_matmul_launch_metadata)
def matmul_kernel_persistent(a_ptr, b_ptr, c_ptr,  #
                             M, N, K,  #
                             stride_am, stride_ak,  #
                             stride_bk, stride_bn,  #
                             stride_cm, stride_cn,  #
                             BLOCK_SIZE_M: tl.constexpr,  #
                             BLOCK_SIZE_N: tl.constexpr,  #
                             BLOCK_SIZE_K: tl.constexpr,  #
                             GROUP_SIZE_M: tl.constexpr,  #
                             NUM_SMS: tl.constexpr,  #
                             ):
    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n

    # NOTE: There is currently a bug in blackwell pipelining that means it can't handle a value being
    # used in both the prologue and epilogue, so we duplicate the counters as a work-around.
    tile_id_c = start_pid - NUM_SMS

    offs_k_for_mask = tl.arange(0, BLOCK_SIZE_K)
    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    for tile_id in tl.range(start_pid, num_tiles, NUM_SMS, flatten=True):
        pid_m, pid_n = _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS)
        start_m = pid_m * BLOCK_SIZE_M
        start_n = pid_n * BLOCK_SIZE_N
        offs_am = start_m + tl.arange(0, BLOCK_SIZE_M)
        offs_bn = start_n + tl.arange(0, BLOCK_SIZE_N)
        offs_am = tl.where(offs_am < M, offs_am, 0)
        offs_bn = tl.where(offs_bn < N, offs_bn, 0)
        offs_am = tl.max_contiguous(tl.multiple_of(offs_am, BLOCK_SIZE_M), BLOCK_SIZE_M)
        offs_bn = tl.max_contiguous(tl.multiple_of(offs_bn, BLOCK_SIZE_N), BLOCK_SIZE_N)

        accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
        for ki in range(k_tiles):
            offs_k = ki * BLOCK_SIZE_K + tl.arange(0, BLOCK_SIZE_K)
            a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
            b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

            a = tl.load(a_ptrs, mask=offs_k_for_mask[None, :] < K - ki * BLOCK_SIZE_K, other=0.0)
            b = tl.load(b_ptrs, mask=offs_k_for_mask[:, None] < K - ki * BLOCK_SIZE_K, other=0.0)
            accumulator = tl.dot(a, b, accumulator)

        tile_id_c += NUM_SMS
        pid_m, pid_n = _compute_pid(tile_id_c, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS)
        offs_cm = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
        offs_cn = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
        c_ptrs = c_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
        c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
        if (c_ptr.dtype.element_ty == tl.float8e4nv):
            c = accumulator.to(tl.float8e4nv)
        else:
            c = accumulator.to(tl.float16)
        tl.store(c_ptrs, c, mask=c_mask)

```
**EN:** `_compute_pid` centralizes the grouped tile-id to `(pid_m, pid_n)` mapping. `matmul_kernel_persistent` then implements the persistent scheduling strategy: instead of launching one program per tile, it launches up to `NUM_SMS` programs and lets each program iterate across multiple output tiles in strides of the SM count. This improves residency and reduces launch overhead, while preserving the baseline pointer-based load/compute/store flow.
**CN:** `_compute_pid` 把分组后的 tile id 统一映射成 `(pid_m, pid_n)`。`matmul_kernel_persistent` 则实现 persistent 调度：它不再为每个 tile 单独启动一个 program，而是最多启动 `NUM_SMS` 个 program，让每个 program 按 SM 数量为步长依次处理多个输出 tile。这样可以提高驻留效率并减少启动开销，同时仍保留基线版本的指针式加载、计算和写回流程。

### Block 7 (Lines 346-366): Persistent wrapper

```python
def matmul_persistent(a, b):
    # Check constraints.
    assert a.shape[1] == b.shape[0], "Incompatible dimensions"
    assert a.dtype == b.dtype, "Incompatible dtypes"
    NUM_SMS = torch.cuda.get_device_properties("cuda").multi_processor_count
    M, K = a.shape
    K, N = b.shape
    dtype = a.dtype
    # Allocates output.
    c = torch.empty((M, N), device=a.device, dtype=dtype)
    # 1D launch kernel where each block gets its own program.
    grid = lambda META: (min(NUM_SMS, triton.cdiv(M, META["BLOCK_SIZE_M"]) * triton.cdiv(N, META["BLOCK_SIZE_N"])), )
    matmul_kernel_persistent[grid](
        a, b, c,  #
        M, N, K,  #
        a.stride(0), a.stride(1),  #
        b.stride(0), b.stride(1),  #
        c.stride(0), c.stride(1),  #
        NUM_SMS=NUM_SMS,  #
    )
    return c
```
**EN:** `matmul_persistent` computes the device SM count, allocates output storage, and limits the launch grid to the smaller of `NUM_SMS` and the total tile count. That launch policy is the host-side counterpart of persistence: the kernel itself loops over the remaining tiles.
**CN:** `matmul_persistent` 负责读取设备 SM 数量、分配输出张量，并把 launch grid 限制为 `NUM_SMS` 与总 tile 数中的较小值。这个启动策略正是 persistent 思想在 host 侧的体现：剩余 tile 由 kernel 内部循环完成。

### Block 8 (Lines 369-446): TMA-persistent configs and kernel

```python
def matmul_tma_persistent_get_configs(pre_hook=None):
    return [
        triton.Config(
            {
                'BLOCK_SIZE_M': BM, 'BLOCK_SIZE_N': BN, "BLOCK_SIZE_K": BK, "GROUP_SIZE_M": 8, "EPILOGUE_SUBTILE":
                SUBTILE
            }, num_stages=s, num_warps=w, pre_hook=pre_hook)  #
        for BM in [128]  #
        for BN in [128, 256]  #
        for BK in [64, 128]  #
        for s in ([2, 3, 4])  #
        for w in [4, 8]  #
        for SUBTILE in [True, False]  #
    ]


@triton.autotune(
    configs=matmul_tma_persistent_get_configs(pre_hook=matmul_tma_set_block_size_hook),
    key=["M", "N", "K", "WARP_SPECIALIZE"],
)
@triton.jit(launch_metadata=_matmul_launch_metadata)
def matmul_kernel_tma_persistent(a_desc, b_desc, c_desc,  #
                                 M, N, K,  #
                                 BLOCK_SIZE_M: tl.constexpr,  #
                                 BLOCK_SIZE_N: tl.constexpr,  #
                                 BLOCK_SIZE_K: tl.constexpr,  #
                                 GROUP_SIZE_M: tl.constexpr,  #
                                 FP8_OUTPUT: tl.constexpr,  #
                                 EPILOGUE_SUBTILE: tl.constexpr,  #
                                 NUM_SMS: tl.constexpr,  #
                                 WARP_SPECIALIZE: tl.constexpr,  #
                                 ):
    dtype = tl.float8e4nv if FP8_OUTPUT else tl.float16
    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n

    tile_id_c = start_pid - NUM_SMS
    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    # Enable warp specialization to leverage async warp scheduling in the GPU.
    # FIXME: This only works on Blackwell right now. On older GPUs, this will
    # use software pipelining.
    for tile_id in tl.range(start_pid, num_tiles, NUM_SMS, flatten=True, warp_specialize=WARP_SPECIALIZE):
        pid_m, pid_n = _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS)
        offs_am = pid_m * BLOCK_SIZE_M
        offs_bn = pid_n * BLOCK_SIZE_N

        accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
        for ki in range(k_tiles):
            offs_k = ki * BLOCK_SIZE_K
            a = a_desc.load([offs_am, offs_k])
            b = b_desc.load([offs_bn, offs_k])
            accumulator = tl.dot(a, b.T, accumulator)

        tile_id_c += NUM_SMS
        pid_m, pid_n = _compute_pid(tile_id_c, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS)
        offs_am_c = pid_m * BLOCK_SIZE_M
        offs_bn_c = pid_n * BLOCK_SIZE_N

        # Epilogue subtiling is a technique to break our computation and stores into multiple pieces
        # By subtiling we can reduce shared memory consumption by the epilogue and instead use that
        # memory to increase our stage count.
        # In this case we partition the accumulator into 2 BLOCK_SIZE_M x BLOCK_SIZE_N // 2 tensors
        if EPILOGUE_SUBTILE:
            acc = tl.reshape(accumulator, (BLOCK_SIZE_M, 2, BLOCK_SIZE_N // 2))
            acc = tl.permute(acc, (0, 2, 1))
            acc0, acc1 = tl.split(acc)
            c0 = acc0.to(dtype)
            c_desc.store([offs_am_c, offs_bn_c], c0)
            c1 = acc1.to(dtype)
            c_desc.store([offs_am_c, offs_bn_c + BLOCK_SIZE_N // 2], c1)
        else:
            accumulator = accumulator.to(dtype)
            c_desc.store([offs_am_c, offs_bn_c], accumulator)

```
**EN:** `matmul_tma_persistent_get_configs` extends the config space with `EPILOGUE_SUBTILE`, allowing autotuning to trade store behavior against pipeline depth. `matmul_kernel_tma_persistent` combines three ideas at once: persistent scheduling, descriptor/TMA loads and stores, and optional warp specialization. Its epilogue can either store the full accumulator directly or split it into two half-width tiles to reduce shared-memory pressure.
**CN:** `matmul_tma_persistent_get_configs` 在配置空间里加入了 `EPILOGUE_SUBTILE`，让 autotune 可以在写回方式与流水深度之间做权衡。`matmul_kernel_tma_persistent` 同时融合了三种思想：persistent 调度、descriptor/TMA 读写、以及可选的 warp specialization。它的 epilogue 既可以直接写回完整累加器，也可以把结果拆成两个半宽 tile，以降低共享内存压力。

### Block 9 (Lines 448-483): Host-side TMA-persistent wrapper

```python
def matmul_tma_persistent(a, b, warp_specialize: bool):
    # Check constraints.
    assert a.shape[1] == b.shape[1], "Incompatible dimensions"  # b is transposed
    assert a.dtype == b.dtype, "Incompatible dtypes"

    M, K = a.shape
    N, K = b.shape
    dtype = a.dtype

    c = torch.empty((M, N), device=a.device, dtype=dtype)

    NUM_SMS = torch.cuda.get_device_properties("cuda").multi_processor_count

    # A dummy block value that will be overwritten when we have the real block size
    dummy_block = [1, 1]
    a_desc = TensorDescriptor.from_tensor(a, dummy_block)
    b_desc = TensorDescriptor.from_tensor(b, dummy_block)
    c_desc = TensorDescriptor.from_tensor(c, dummy_block)

    def grid(META):
        nonlocal a_desc, b_desc, c_desc
        BLOCK_M = META["BLOCK_SIZE_M"]
        BLOCK_N = META["BLOCK_SIZE_N"]
        return (min(
            NUM_SMS,
            triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N),
        ), )

    matmul_kernel_tma_persistent[grid](
        a_desc, b_desc, c_desc,  #
        M, N, K,  #
        FP8_OUTPUT=dtype == torch.float8_e4m3fn,  #
        NUM_SMS=NUM_SMS,  #
        WARP_SPECIALIZE=warp_specialize,  #
    )
    return c
```
**EN:** `matmul_tma_persistent` mirrors the earlier TMA wrapper but now also threads through `NUM_SMS` so the kernel can persist across tiles. The descriptors are again created with dummy blocks and corrected by the pre-hook selected through autotuning.
**CN:** `matmul_tma_persistent` 与前面的 TMA 包装层相似，但这里还会把 `NUM_SMS` 传入 kernel，使其能够跨 tile 持续执行。描述符依旧先用占位 block 创建，再由 autotune 选中的 pre-hook 修正为真实 block 大小。

### Block 10 (Lines 486-570): Config pruning and device-side descriptor persistent kernel

```python
def prune_invalid_configs(configs, named_args, **kwargs):
    FLATTEN = kwargs["FLATTEN"]
    # Filter out configs where EPILOGUE_SUBTILE is true and HOPPER is true
    return [conf for conf in configs if not (conf.kwargs.get("EPILOGUE_SUBTILE", True) and FLATTEN is False)]


@triton.autotune(configs=matmul_tma_persistent_get_configs(), key=["M", "N", "K", "WARP_SPECIALIZE", "FLATTEN"],
                 prune_configs_by={'early_config_prune': prune_invalid_configs})
@triton.jit(launch_metadata=_matmul_launch_metadata)
def matmul_kernel_descriptor_persistent(
    a_ptr,
    b_ptr,
    c_ptr,  #
    M,
    N,
    K,  #
    BLOCK_SIZE_M: tl.constexpr,  #
    BLOCK_SIZE_N: tl.constexpr,  #
    BLOCK_SIZE_K: tl.constexpr,  #
    GROUP_SIZE_M: tl.constexpr,  #
    EPILOGUE_SUBTILE: tl.constexpr,  #
    NUM_SMS: tl.constexpr,  #
    WARP_SPECIALIZE: tl.constexpr,  #
    FLATTEN: tl.constexpr,
):
    # Matmul using TMA and device-side descriptor creation
    dtype = c_ptr.dtype.element_ty
    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n

    a_desc = tl.make_tensor_descriptor(
        a_ptr,
        shape=[M, K],
        strides=[K, 1],
        block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_K],
    )
    b_desc = tl.make_tensor_descriptor(
        b_ptr,
        shape=[N, K],
        strides=[K, 1],
        block_shape=[BLOCK_SIZE_N, BLOCK_SIZE_K],
    )
    c_desc = tl.make_tensor_descriptor(
        c_ptr,
        shape=[M, N],
        strides=[N, 1],
        block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_N if not EPILOGUE_SUBTILE else BLOCK_SIZE_N // 2],
    )

    # tile_id_c is used in the epilogue to break the dependency between
    # the prologue and the epilogue
    tile_id_c = start_pid - NUM_SMS
    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    for tile_id in tl.range(start_pid, num_tiles, NUM_SMS, flatten=FLATTEN, warp_specialize=WARP_SPECIALIZE):
        pid_m, pid_n = _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS)
        offs_am = pid_m * BLOCK_SIZE_M
        offs_bn = pid_n * BLOCK_SIZE_N

        accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
        for ki in range(k_tiles):
            offs_k = ki * BLOCK_SIZE_K
            a = a_desc.load([offs_am, offs_k])
            b = b_desc.load([offs_bn, offs_k])
            accumulator = tl.dot(a, b.T, accumulator)

        tile_id_c += NUM_SMS
        pid_m, pid_n = _compute_pid(tile_id_c, num_pid_in_group, num_pid_m, GROUP_SIZE_M, NUM_SMS)
        offs_cm = pid_m * BLOCK_SIZE_M
        offs_cn = pid_n * BLOCK_SIZE_N

        if EPILOGUE_SUBTILE:
            acc = tl.reshape(accumulator, (BLOCK_SIZE_M, 2, BLOCK_SIZE_N // 2))
            acc = tl.permute(acc, (0, 2, 1))
            acc0, acc1 = tl.split(acc)
            c0 = acc0.to(dtype)
            c_desc.store([offs_cm, offs_cn], c0)
            c1 = acc1.to(dtype)
            c_desc.store([offs_cm, offs_cn + BLOCK_SIZE_N // 2], c1)
        else:
            c = accumulator.to(dtype)
            c_desc.store([offs_cm, offs_cn], c)
```
**EN:** `prune_invalid_configs` removes combinations that are known to be unsupported when flattening is disabled. `matmul_kernel_descriptor_persistent` moves descriptor construction onto the device with `tl.make_tensor_descriptor`, so the kernel receives raw pointers and creates the TMA descriptors internally. This block shows the most self-contained Triton variant in the file: autotuned, persistent, descriptor-based, optionally warp-specialized, and capable of split-epilogue stores.
**CN:** `prune_invalid_configs` 会裁掉在禁用 flatten 时已知不合法的配置组合。`matmul_kernel_descriptor_persistent` 则把 descriptor 的创建下沉到设备端，通过 `tl.make_tensor_descriptor` 直接在 kernel 内从原始指针构造 TMA 描述符。它是全文件中最“自包含”的 Triton 版本：支持 autotune、persistent、descriptor、可选 warp specialization，以及可拆分 epilogue 写回。

### Block 11 (Lines 573-605): Descriptor-persistent wrapper and allocator hook

```python
def matmul_descriptor_persistent(a, b, warp_specialize: bool):
    # Check constraints.
    assert a.shape[1] == b.shape[1], "Incompatible dimensions"  # b is transposed
    assert a.dtype == b.dtype, "Incompatible dtypes"

    M, K = a.shape
    N, K = b.shape
    dtype = a.dtype

    c = torch.empty((M, N), device=a.device, dtype=dtype)
    NUM_SMS = torch.cuda.get_device_properties("cuda").multi_processor_count

    # TMA descriptors require a global memory allocation
    def alloc_fn(size: int, alignment: int, stream: Optional[int]):
        return torch.empty(size, device="cuda", dtype=torch.int8)

    triton.set_allocator(alloc_fn)

    # Hopper warpspec doesn't work with flatten
    flatten = False if (warp_specialize and is_hopper()) else True
    grid = lambda META: (min(NUM_SMS, triton.cdiv(M, META["BLOCK_SIZE_M"]) * triton.cdiv(N, META["BLOCK_SIZE_N"])), )
    matmul_kernel_descriptor_persistent[grid](
        a,
        b,
        c,  #
        M,
        N,
        K,  #
        NUM_SMS=NUM_SMS,  #
        WARP_SPECIALIZE=warp_specialize,  #
        FLATTEN=flatten,
    )
    return c
```
**EN:** `matmul_descriptor_persistent` is the Python wrapper for the device-side descriptor kernel. It installs a custom allocator via `triton.set_allocator`, because TMA descriptor support requires globally allocated memory, computes the Hopper-specific `flatten` policy, defines the persistent launch grid, and launches `matmul_kernel_descriptor_persistent`.
**CN:** `matmul_descriptor_persistent` 是设备端 descriptor kernel 的 Python 包装层。它通过 `triton.set_allocator` 安装自定义分配器，因为 TMA descriptor 支持需要全局内存分配；同时根据 Hopper 的限制决定 `flatten` 策略，定义 persistent grid，并最终启动 `matmul_kernel_descriptor_persistent`。

### Block 12 (Lines 608-680): Reference matmuls, profiling context, and benchmark driver

```python
def device_blas_matmul(a, b):
    # Check constraints.
    assert a.shape[1] == b.shape[1], "Incompatible dimensions"  # b is transposed
    M, K = a.shape
    N, K = b.shape
    dtype = a.dtype
    c = torch.empty((M, N), device=a.device, dtype=dtype)
    bytes_per_elem = a.element_size()
    flops_str = f"flops{bytes_per_elem * 8}"
    blas_name = device_blas_name()
    with proton.scope(f"{blas_name} [M={M}, N={N}, K={K}]",
                      {"bytes": bytes_per_elem * (M * K + N * K + M * N), flops_str: 2. * M * N * K}):
        device_blas.matmul(a, b, c)
    return c


def torch_matmul(a, b):
    M, K = a.shape
    N, K = b.shape
    bytes_per_elem = a.element_size()
    flops_str = f"flops{bytes_per_elem * 8}"
    with proton.scope(f"torch [M={M}, N={N}, K={K}]",
                      {"bytes": bytes_per_elem * (M * K + N * K + M * N), flops_str: 2. * M * N * K}):
        c = torch.matmul(a, b.T)
    return c


@contextmanager
def proton_context():
    proton.activate(0)
    try:
        yield
    finally:
        proton.deactivate(0)


def bench_fn(label, reps, warmup_reps, fn, *args):
    print(f"Benchmarking {label}: ...", end="")
    for _ in range(warmup_reps):
        fn(*args)
    with proton_context():
        for _ in range(reps):
            fn(*args)
    print(f"\rBenchmarking {label}: done")


def bench(K, dtype, reps=10000, warmup_reps=10000):
    M = 8192
    N = 8192
    a = torch.randn((M, K), device="cuda", dtype=torch.float16).to(dtype)
    b = torch.randn((K, N), device="cuda", dtype=torch.float16).to(dtype)

    b = b.T.contiguous()

    if device_blas is not None:
        blas_name = device_blas_name()
        bench_fn(blas_name, reps, warmup_reps, device_blas_matmul, a, b)
    if dtype == torch.float16:
        bench_fn("torch", reps, warmup_reps, torch_matmul, a, b)
    bench_fn("naive", reps, warmup_reps, matmul, a, b.T)
    bench_fn("persistent", reps, warmup_reps, matmul_persistent, a, b.T)
    warp_specialize = [False, True] if HAS_WARP_SPECIALIZE else [False]
    for ws in warp_specialize:
        ws_str = "_ws" if ws else ""
        # disable on-host warpspec on Hopper
        if HAS_HOST_TENSOR_DESC and not (is_hopper() and ws):
            bench_fn(f"tma_persistent{ws_str}", reps, warmup_reps, lambda a, b: matmul_tma_persistent(a, b, ws), a, b)
            bench_fn(f"tma{ws_str}", reps, warmup_reps, lambda a, b: matmul_tma(a, b, ws), a, b)
        if HAS_TENSOR_DESC:
            bench_fn(f"descriptor_persistent{ws_str}", reps, warmup_reps,
                     lambda a, b: matmul_descriptor_persistent(a, b, ws), a, b)


```
**EN:** These helpers provide comparison and measurement infrastructure. `device_blas_matmul` wraps cuBLAS/hipBLAS with explicit proton scopes, `torch_matmul` provides a PyTorch reference path, `proton_context` safely activates and deactivates profiling, `bench_fn` handles warmup plus measured repetitions, and `bench` builds large test matrices before benchmarking BLAS, Torch, baseline Triton, persistent Triton, TMA-persistent Triton, plain TMA Triton, and the descriptor-persistent kernel depending on feature availability.
**CN:** 这一组辅助函数负责对照实现与性能测量。`device_blas_matmul` 用 proton scope 包装 cuBLAS/hipBLAS，`torch_matmul` 提供 PyTorch 参考路径，`proton_context` 负责安全地开启和关闭 profiler，`bench_fn` 统一处理预热与正式计时，而 `bench` 会构造大矩阵，并按功能可用性依次测试 BLAS、Torch、基线 Triton、persistent Triton、TMA persistent、普通 TMA，以及 descriptor persistent kernel。

### Block 13 (Lines 681-716): Correctness validation flow

```python
def run_test(expect, fn, a, b, label, enabled=True):
    print(f"  {label}: ...", end="")
    if enabled:
        actual = fn(a, b)
        passed = torch.allclose(expect, actual.to(expect.dtype), atol=1.0)
        icon = "✅" if passed else "❌"
    else:
        icon = "⭕"
    print(f"\r  {label}: {icon}  ")


def validate(M, N, K, dtype):
    print(f"{M=}, {N=}, {K=}, verification naive vs: ")
    a = torch.randn((M, K), device="cuda", dtype=torch.float16).to(dtype)
    b = torch.randn((K, N), device="cuda", dtype=torch.float16).to(dtype)
    b = b.T.contiguous()

    naive_result = matmul(a, b.T).to(torch.float16)
    run_test(naive_result, torch_matmul, a, b, "Torch", enabled=dtype == torch.float16)
    run_test(naive_result, device_blas_matmul, a, b, device_blas_name(), enabled=device_blas is not None)
    run_test(naive_result, matmul_persistent, a, b.T, "Persistent")

    kernels = [
        (matmul_tma, "TMA", HAS_HOST_TENSOR_DESC),
        (matmul_tma_persistent, "TMA Persistent", HAS_HOST_TENSOR_DESC),
        (matmul_descriptor_persistent, "Tensor Descriptor Persistent", HAS_TENSOR_DESC),
    ]
    warp_specialize = [False, True] if HAS_WARP_SPECIALIZE else [False]

    for (kernel, label, enabled), warp_specialize in itertools.product(kernels, warp_specialize):
        label = f"{label} (warp_specialize={warp_specialize})"
        # skip if hopper and warp_specialize and not on-device
        skipped = is_hopper() and warp_specialize and kernel != matmul_descriptor_persistent
        enabled = enabled and (not warp_specialize or HAS_TENSOR_DESC) and (not skipped)
        run_test(naive_result, lambda a, b: kernel(a, b, warp_specialize), a, b, label, enabled)
    print()
```
**EN:** `run_test` standardizes result comparison and console reporting. `validate` creates random inputs, uses the baseline Triton kernel as the expected result, compares Torch and vendor BLAS when available, then iterates over the TMA-related kernels and both warp-specialization choices. The enable/skip logic documents important platform restrictions such as Hopper host-side warp specialization behavior.
**CN:** `run_test` 统一了结果比较与终端输出格式。`validate` 会生成随机输入，先以基线 Triton kernel 的结果作为期望值，再在可用时比较 Torch 和厂商 BLAS，随后遍历所有 TMA 相关 kernel 以及两种 warp specialization 选项。这里的 enable/skip 逻辑也明确体现了平台限制，例如 Hopper 上 host-side warp specialization 的特殊情况。

### Block 14 (Lines 719-758): Profile display and CLI entrypoint

```python
def show_profile(precision, profile_name):
    import triton.profiler.viewer as proton_viewer
    metric_names = ["time/ms"]
    if precision == 'fp8':
        metric_names = ["tflop8/s"] + metric_names
    elif precision == 'fp16':
        metric_names = ["tflop16/s"] + metric_names
    file_name = f"{profile_name}.hatchet"
    tree, metrics = proton_viewer.parse(metric_names, file_name)
    proton_viewer.print_tree(tree, metrics)


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-K", type=int, required=False, default=512)
    parser.add_argument("--K_range", type=int, nargs=2)
    parser.add_argument("--K_step", type=int, default=512)
    parser.add_argument("--prec", type=str, choices=["fp8", "fp16"], default="fp16")
    args = parser.parse_args()

    if args.prec == 'fp8' and (not hasattr(torch, "float8_e4m3fn") or not is_cuda()):
        print("This example requires CUDA/HIP with fp8 support.")
    else:
        dtype = torch.float8_e4m3fn if args.prec == 'fp8' else torch.float16

        if args.K and args.K_range is None:
            args.K_range = [args.K, args.K]
            args.K_step = 1  # doesn't matter as long as it's not 0

        torch.manual_seed(0)

        validate(32, 32, 32, dtype)
        validate(8192, 8192, args.K_range[0], dtype)

        proton.start("matmul", hook="triton")
        proton.deactivate()
        for K in range(args.K_range[0], args.K_range[1] + 1, args.K_step):
            bench(K, dtype)
        proton.finalize()
        show_profile(args.prec, "matmul")
```
**EN:** `show_profile` parses the proton output file and prints a metric tree tuned to FP8 or FP16 runs. The `__main__` block defines the command-line interface (`-K`, `--K_range`, `--K_step`, `--prec`), checks FP8 support, selects the dtype, normalizes single-K input into a range, seeds randomness, runs small and large validation passes, starts proton tracing, benchmarks all requested K values, finalizes the trace, and finally prints the summarized profile. This is the end-to-end tutorial flow from argument parsing to correctness and performance output.
**CN:** `show_profile` 负责解析 proton 输出文件，并根据 FP8 或 FP16 选择合适的性能指标树。`__main__` 部分定义了命令行接口（`-K`、`--K_range`、`--K_step`、`--prec`），检查 FP8 支持情况，选择 dtype，把单个 K 值规范化成区间，设置随机种子，先做小规模和大规模正确性验证，再启动 proton 跟踪，对请求的每个 K 值执行 benchmark，最后结束 profiling 并打印汇总结果。这也是整份教程从参数解析到正确性与性能输出的完整执行流程。

## Key Concepts / 关键概念

- **Baseline tiled matmul / 基线分块矩阵乘法:** Uses pointer arithmetic, grouped tile scheduling, FP32 accumulation, and autotuned tile shapes as the performance baseline.
- **Persistent kernels / 持久化内核:** Limit launched programs to roughly the SM count and let each program process multiple tiles, trading extra in-kernel looping for better residency and lower launch overhead.
- **TMA (Tensor Memory Accelerator) / 张量内存加速器:** Replaces manual pointer-based block movement with descriptor-driven loads/stores that align with newer NVIDIA hardware capabilities.
- **Tensor descriptors / 张量描述符:** The file demonstrates both host-created descriptors (`TensorDescriptor.from_tensor`) and device-created descriptors (`tl.make_tensor_descriptor`).
- **Warp specialization / Warp 专门化:** An optional scheduling mode that can overlap work differently depending on hardware generation; the tutorial guards it carefully with capability checks.
- **Epilogue subtiling / 尾声子分块:** Splits the output store into two narrower stores so shared memory can be repurposed for deeper pipelining.
- **Autotuning / 自动调优:** Triton explores multiple block sizes, stage counts, and warp counts, while hooks and pruning rules keep descriptor-based configurations valid.
- **Correctness + profiling / 正确性与性能分析:** The script validates every variant against the baseline result, then profiles selected runs with Proton and prints summarized metrics.

## Dependencies / 依赖关系

- **External libraries / 外部库:** `torch` supplies tensors and device queries; `triton` and `triton.language` provide kernel compilation/runtime APIs; `triton.profiler` (`proton`) handles profiling; `TensorDescriptor` and `tl.make_tensor_descriptor` provide host/device descriptor support; vendor BLAS backends come from Triton C extensions (`nvidia.cublas.CublasLt` or `amd.hipblas.HipblasLt`).
- **Capability gating / 能力门控:** `is_cuda()`, `is_hip()`, `supports_tma()`, `is_hopper()`, `supports_ws()`, `HAS_TENSOR_DESC`, `HAS_HOST_TENSOR_DESC`, and `HAS_WARP_SPECIALIZE` determine which kernels and benchmark branches are legal on the current GPU/runtime.
- **Kernel wrappers to kernels / 包装层到内核的依赖:** `matmul()` → `matmul_kernel`; `matmul_tma()` → `matmul_tma_set_block_size_hook` + `matmul_kernel_tma`; `matmul_persistent()` → `matmul_kernel_persistent`; `matmul_tma_persistent()` → `matmul_tma_set_block_size_hook` + `matmul_kernel_tma_persistent`; `matmul_descriptor_persistent()` → `prune_invalid_configs` + `matmul_kernel_descriptor_persistent`.
- **Shared helper flow / 共享辅助流程:** `_matmul_launch_metadata` annotates all autotuned kernels; `_compute_pid` is reused by both persistent kernels and the descriptor-persistent kernel to map tile ids to output coordinates.
- **Benchmark and validation flow / 基准与验证流程:** `validate()` depends on `matmul`, `torch_matmul`, `device_blas_matmul`, `matmul_persistent`, `matmul_tma`, `matmul_tma_persistent`, and `matmul_descriptor_persistent`; `bench()` benchmarks the same family under profiler scopes.
- **CLI execution chain / CLI 执行链:** `__main__` → argument parsing → dtype/capability checks → `validate()` on small and large cases → `proton.start()` / repeated `bench()` calls → `proton.finalize()` → `show_profile()`.
