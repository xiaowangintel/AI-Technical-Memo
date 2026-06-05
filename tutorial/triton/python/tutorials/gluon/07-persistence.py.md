# 07-persistence.py — Code Analysis / 代码分析

## Source / 来源

- Source file: `/root/xw/triton/python/tutorials/gluon/07-persistence.py`

- Purpose / 目的: This Triton Gluon tutorial builds and studies persistent matrix multiplication kernels for Hopper and Blackwell GPUs, starting from a reusable MMA abstraction and ending with a persistent, outer-loop-pipelined design. / 这个 Triton Gluon 教程围绕 Hopper 与 Blackwell GPU 构建并分析 persistent 矩阵乘法内核：先抽象统一的 MMA 接口，再发展到支持外层循环流水的 persistent 设计。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 — Tutorial overview / 教程概述

```python
"""
Persistent Kernels
==================

So far, we have defined kernels such that one programs handles one block of work
and we span all the work using the grid dimensions. This creates a large number
of programs, and we rely on the GPU to schedule the work. The primary benefit is
the GPU will dynamically load-balance the work across its SMs.

However, this approach has downsides. The scheduler incurs an overhead, and the
GPU is not aware of the memory access patterns of the kernels. This also
prevents overlapping across blocks of work, as the GPU waits until kernels have
fully exited before issuing more work.

Persistent kernels is a technique where we assign multiple blocks of work to
each program, and the programs "persist" on the GPU until all the work is
complete. The work assignment is typically static, although dynamic scheduling
is still possible with more advanced techniques or hardware features like
cluster launch control.

In this tutorial, we will explore persistent kernels by implementing a
persistent matmul. We will then show how we can pipeline across the persistent
outer loop to achieve greater overlap and more throughput.
"""
```

**EN:** The opening docstring introduces persistent kernels as an alternative to launching one Triton program per output tile. It explains the core motivation: reduce scheduling overhead and enable better overlap across load, compute, and epilogue work on the GPU.

**CN:** 开头的文档字符串把 persistent kernel 介绍为“每个输出 tile 启动一个 Triton program”之外的另一种方案。它点明了核心动机：降低调度开销，并让 GPU 上的加载、计算和收尾阶段获得更好的重叠。

### Lines 26-61 — Imports and optional cuBLAS setup / 导入与可选 cuBLAS 初始化

```python
import itertools
import pytest
import torch
import triton
import importlib
import sys
from functools import partial
from typing import Union
from triton.experimental import gluon
from triton.experimental.gluon import language as gl

from triton.experimental.gluon.nvidia.hopper import TensorDescriptor
from triton.experimental.gluon.language.nvidia.hopper import (
    tma,
    mbarrier,
    fence_async_shared,
    warpgroup_mma,
    warpgroup_mma_wait,
    warpgroup_mma_accumulator,
)
from triton.experimental.gluon.language.nvidia.blackwell import (
    TensorMemoryLayout,
    tensor_memory_descriptor,
    allocate_tensor_memory,
    tcgen05_mma,
    tcgen05_commit,
)

if torch.cuda.is_available():
    from triton._C.libtriton import nvidia
    cublas_workspace = torch.empty(32 * 1024 * 1024, device="cuda", dtype=torch.uint8)
    cublas = nvidia.cublas.CublasLt(cublas_workspace)
else:
    cublas = None

t5 = importlib.import_module("05-wgmma")
```

**EN:** This block imports PyTorch, Triton, Gluon, and architecture-specific Hopper/Blackwell primitives. When CUDA is available, it also constructs a CublasLt handle and workspace for later performance comparisons, and imports tutorial 05-wgmma for a Hopper layout helper.

**CN:** 这一段导入了 PyTorch、Triton、Gluon，以及 Hopper/Blackwell 专用原语。若 CUDA 可用，它还创建 CublasLt 句柄和工作区以便后续做性能对比，并导入教程 05-wgmma 中的 Hopper 布局辅助函数。

### Lines 64-77 — Hardware guards and utility helpers / 硬件检查与辅助函数

```python
def is_hopper_or_newer():
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] >= 9


if __name__ == "__main__" and not is_hopper_or_newer():
    raise RuntimeError("This tutorial requires Hopper or newer NVIDIA GPU")

profiling_with_ncu = len(sys.argv) > 1 and sys.argv[1] == "profile"


def get_flops(ms, M, N, K):
    flops = 2 * M * N * K
    return flops * 1e-12 / (ms * 1e-3)
```

**EN:** The script requires CUDA devices with compute capability 9.0 or newer, meaning Hopper or Blackwell. It also parses a simple `profile` CLI flag and defines `get_flops` to convert benchmark latency into TFLOPS.

**CN:** 脚本要求 CUDA 设备的计算能力至少为 9.0，也就是 Hopper 或 Blackwell。它还解析简单的 `profile` 命令行参数，并用 `get_flops` 把基准测试延迟换算成 TFLOPS。

### Lines 80-119 — Hopper MMA abstraction / Hopper MMA 抽象

```python
# %%
# In the previous two tutorials, we introduced tensor core operations for Hopper
# and Blackwell NVIDIA GPUs. To make this tutorial more accessible, and to
# demonstrate some Gluon features, we will build an abstraction around both sets
# of tensor core operations so that our persistent matmul can be used on both
# Hopper and Blackwell.
#
# We can use @gluon.aggregate to define a class that contains the state of the
# matmul. We will define the API of our MMA wrapper to be like WGMMA's, because
# is the more restrictive of the two.


# MMA wrapper for WGMMA, which maps directly to the WGMMA functions.
@gluon.aggregate
class WGMMA:
    acc: Union[warpgroup_mma_accumulator, gl.tensor]
    use_acc: gl.tensor

    @gluon.jit
    def initialize(dtype: gl.constexpr, BLOCK_M: gl.constexpr, BLOCK_N: gl.constexpr, num_warps: gl.constexpr):
        mma_layout: gl.constexpr = t5.pick_wgmma_layout(dtype, BLOCK_M, BLOCK_N, num_warps)
        acc = gl.zeros((BLOCK_M, BLOCK_N), dtype=gl.float32, layout=mma_layout)
        return WGMMA(acc, gl.to_tensor(False))

    @gluon.jit
    def issue_async_mma(self, a, b):
        acc = warpgroup_mma(a, b, self.acc, is_async=True, use_acc=self.use_acc)
        # Note that aggregates don't support in-place mutation, so we need to
        # return a new instance and re-assign it at the callsite.
        return WGMMA(acc, gl.to_tensor(True))

    @gluon.jit
    def wait_num_outstanding(self, num_outstanding: gl.constexpr):
        acc = warpgroup_mma_wait(num_outstanding, (self.acc, ))
        return WGMMA(acc, self.use_acc)

    # Take the result and reset the accumulator.
    @gluon.jit
    def take_result(self, splitn: gl.constexpr = False):
        return self.acc, WGMMA(self.acc, gl.to_tensor(False))
```

**EN:** The comments explain why the tutorial builds a shared MMA interface, then `WGMMA` implements it for Hopper. The aggregate stores the accumulator plus a `use_acc` flag, and each method returns a new aggregate because Gluon aggregates are immutable value objects inside JIT code.

**CN:** 注释先说明为什么教程要构建统一的 MMA 接口，然后 `WGMMA` 为 Hopper 实现了这套接口。该 aggregate 保存累加器和 `use_acc` 标志位，而每个方法都会返回新的 aggregate，因为在 Gluon JIT 中 aggregate 是不可原地修改的值对象。

### Lines 122-167 — Blackwell MMA abstraction and selection / Blackwell MMA 抽象与选择逻辑

```python
# MMA wrapper for tcgen05. In order to implement `wait_num_outstanding`, we
# need to allocate barriers and keep track of how many MMAs have been issued.
# State will be tracked with an accumulator.
@gluon.aggregate
class MMAv5:
    use_acc: gl.tensor
    acc_tmem: tensor_memory_descriptor
    bar: gl.shared_memory_descriptor
    counter: gl.tensor

    @gluon.jit
    def initialize(dtype: gl.constexpr, BLOCK_M: gl.constexpr, BLOCK_N: gl.constexpr, num_warps: gl.constexpr):
        layout: gl.constexpr = TensorMemoryLayout([BLOCK_M, BLOCK_N], col_stride=1)
        acc_tmem = allocate_tensor_memory(gl.float32, [BLOCK_M, BLOCK_N], layout)
        bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
        mbarrier.init(bar, count=1)
        return MMAv5(gl.to_tensor(False), acc_tmem, bar, gl.to_tensor(0))

    @gluon.jit
    def issue_async_mma(self, a, b):
        tcgen05_mma(a, b, self.acc_tmem, use_acc=self.use_acc)
        tcgen05_commit(self.bar)
        return MMAv5(gl.to_tensor(True), self.acc_tmem, self.bar, self.counter + 1)

    @gluon.jit
    def wait_num_outstanding(self, num_outstanding: gl.constexpr):
        mbarrier.wait(self.bar, (self.counter - 1 - num_outstanding) & 1)
        return self

    @gluon.jit
    def take_result(self, splitn: gl.constexpr = False):
        next = MMAv5(gl.to_tensor(False), self.acc_tmem, self.bar, self.counter)
        if splitn:
            layout: gl.constexpr = self.acc_tmem.get_reg_layout(instr_variant="32x32b_splitn")
            return self.acc_tmem.load(layout), next
        else:
            return self.acc_tmem.load(), next


def select_mma_impl():
    if torch.cuda.get_device_capability()[0] == 9:
        return WGMMA
    elif torch.cuda.get_device_capability()[0] == 10:
        return MMAv5
    else:
        return None
```

**EN:** `MMAv5` mirrors the Hopper API on top of Blackwell tcgen05 instructions. Unlike WGMMA, it explicitly allocates tensor memory for the accumulator, uses an mbarrier to track committed MMAs, and counts issued operations so `wait_num_outstanding` can wait on the correct phase. `select_mma_impl` picks the implementation from compute capability.

**CN:** `MMAv5` 在 Blackwell 的 tcgen05 指令之上复用了与 Hopper 相同的接口。与 WGMMA 不同，它显式为累加器分配 tensor memory，用 mbarrier 跟踪已提交的 MMA，并统计已发射次数，使 `wait_num_outstanding` 能等待正确的相位；`select_mma_impl` 则根据计算能力选择实现。

### Lines 170-199 — Reusable load and compute pipeline stages / 可复用的加载与计算流水阶段

```python
# %%
# Let's validate our abstraction by implementing a matmul where we pipeline both
# the MMA and the loads. This achieves async overlap of both the TMA loads and
# the MMAs by requiring at least two operand buffers. This will make the
# persistent kernel more interesting by allowing us to overlap more things.
#
# We will factor our kernel into components we can re-use between
# implementations.


@gluon.jit
def issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers: gl.constexpr, pred=True):
    index = producer % num_buffers
    producer += 1
    bar = bars.index(index)
    mbarrier.expect(bar, a_desc.block_type.nbytes + b_desc.block_type.nbytes, pred=pred)
    tma.async_load(a_desc, [off_m, k], bar, a_bufs.index(index), pred)
    tma.async_load(b_desc, [k, off_n], bar, b_bufs.index(index), pred)
    return producer


@gluon.jit
def issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers: gl.constexpr):
    index = consumer % num_buffers
    phase = consumer // num_buffers & 1
    consumer += 1
    mbarrier.wait(bars.index(index), phase)
    mma = mma.wait_num_outstanding(0)
    mma = mma.issue_async_mma(a_bufs.index(index), b_bufs.index(index))
    return consumer, mma
```

**EN:** These helpers factor the kernel pipeline into two reusable steps. `issue_loads` posts asynchronous TMA loads for the next K-slice and programs the expected byte count on the barrier, while `issue_mma` waits for the corresponding buffer to become ready and then launches one asynchronous MMA from shared memory.

**CN:** 这两个辅助函数把内核流水拆成两个可复用步骤。`issue_loads` 为下一个 K 分块发起异步 TMA 加载，并在 barrier 上登记期望字节数；`issue_mma` 则等待对应缓冲就绪后，从共享内存发起一次异步 MMA。

### Lines 202-247 — Non-persistent pipelined matmul kernel / 非 persistent 的流水 matmul 内核

```python
@gluon.jit
def matmul_pipelined_kernel(a_desc, b_desc, c_desc, MMAImpl: gl.constexpr, num_buffers: gl.constexpr,
                            num_warps: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = a_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype
    K = a_desc.shape[1]

    gl.static_assert(num_buffers >= 2, "expected at least 2 buffers")
    a_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + a_desc.block_type.shape, a_desc.layout)
    b_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + b_desc.block_type.shape, b_desc.layout)
    bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(num_buffers):
        mbarrier.init(bars.index(i), count=1)
    # Separate producer and consumer indices, to support more than 2 buffers.
    producer = 0
    consumer = 0

    pid_m = gl.program_id(axis=0)
    pid_n = gl.program_id(axis=1)
    off_m = pid_m * BLOCK_M
    off_n = pid_n * BLOCK_N

    # Use our MMA abstraction!
    mma = MMAImpl.initialize(dtype, BLOCK_M, BLOCK_N, num_warps)

    # Prefetch at most num_buffers-2 loads to allow the MMA to overlap.
    for k in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
        producer = issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers)

    for k in range(BLOCK_K * (num_buffers - 2), K, BLOCK_K):
        producer = issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers)
        consumer, mma = issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers)

    for _ in gl.static_range(num_buffers - 2):
        consumer, mma = issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers)

    mma = mma.wait_num_outstanding(0)
    c_smem = gl.allocate_shared_memory(dtype, c_desc.block_type.shape, c_desc.layout)
    c, mma = mma.take_result()
    c_smem.store(c.to(dtype))
    fence_async_shared()
    tma.async_copy_shared_to_global(c_desc, [off_m, off_n], c_smem)
    tma.store_wait(pendings=0)

```

**EN:** This kernel allocates multiple shared-memory operand buffers plus one barrier per buffer slot, prefetches the first `num_buffers-2` K-tiles, and then overlaps later loads with MMAs. After draining the pipeline it stores the accumulator into shared memory, fences it for TMA visibility, and writes the result tile to global memory asynchronously.

**CN:** 这个内核为操作数分配多个共享内存缓冲区，并为每个槽位分配一个 barrier；它先预取前 `num_buffers-2` 个 K 分块，再把后续加载与 MMA 重叠执行。流水清空后，内核把累加结果写入共享内存，通过 `fence_async_shared` 让 TMA 可见，然后异步写回全局内存。

### Lines 249-275 — Host wrapper and correctness test for pipelined matmul / 流水 matmul 的主机封装与正确性测试

```python
def matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps):
    MMAImpl = select_mma_impl()
    M, N = C.shape

    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gl.float16)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], gl.float16)
    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], gl.float16)
    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(B, [BLOCK_K, BLOCK_N], b_layout)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)

    grid = (triton.cdiv(M, BLOCK_M), triton.cdiv(N, BLOCK_N))
    matmul_pipelined_kernel[grid](a_desc, b_desc, c_desc, MMAImpl, num_buffers, num_warps=num_warps)


@pytest.mark.parametrize("M, N, K", [(2000, 1000, 2000)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 256, 64)])
@pytest.mark.parametrize("num_buffers", [2, 3, 4])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper or newer")
def test_pipelined_matmul(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps):
    torch.manual_seed(0)
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)
    matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps)
    torch.testing.assert_close(A @ B, C, rtol=1e-3, atol=1e-1)
```

**EN:** The Python wrapper builds `TensorDescriptor` objects with NV MMA-friendly shared-memory layouts, launches a 2D grid over output tiles, and the pytest case verifies numerical correctness against `A @ B`. The test sweeps buffer counts and warp counts because those parameters directly affect overlap and register pressure.

**CN:** Python 封装层使用适合 NV MMA 的共享内存布局构造 `TensorDescriptor`，再以 2D 网格覆盖输出 tile；pytest 用 `A @ B` 校验数值正确性。测试会扫描缓冲区数量和 warp 数，因为这两个参数直接影响重叠程度与寄存器压力。

### Lines 278-320 — Benchmark harness and first performance discussion / 基准测试与第一轮性能讨论

```python
# %%
# The optimal block shapes for our kernel are BLOCK_M=128 and BLOCK_N=256, which
# gives the maximum instruction shape on both Blackwell and Hopper. However, on
# Hopper we need 8 warps to fit the accumulator in registers.

if __name__ == "__main__":
    M, N, K = 8192, 8192, 16 * 1024
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)

if __name__ == "__main__" and not profiling_with_ncu:
    BLOCK_M = 128
    BLOCK_N = 256
    is_hopper = torch.cuda.get_device_capability()[0] == 9
    warps = [8] if is_hopper else [4, 8]
    print("Benchmarking pipelined matmul")
    print("=============================")
    print(f"BLOCK_M={BLOCK_M}, BLOCK_N={BLOCK_N}")
    print("BLOCK_K num_buffers num_warps tflops/s")
    for (BLOCK_K, num_buffers), num_warps in itertools.product([(128, 2), (64, 3), (64, 4)], warps):
        print(f"{BLOCK_K:>7} {num_buffers:>11} {num_warps:>9}", end=" ")
        fn = lambda: matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps)
        ms = triton.testing.do_bench_cudagraph(fn)
        print(f"{get_flops(ms, M, N, K):8.2f}")
    print()

# %%
# ```text
# BLOCK_K num_buffers num_warps Blackwell  Hopper
#     128           2         4    735.96
#     128           2         8    697.97  489.26
#      64           3         4   1054.00
#      64           3         8    973.94  673.67
#      64           4         4   1175.70
#      64           4         8   1072.83  669.16
# ```
#
# Blackwell performance lines up with what we have seen in previous tutorials,
# but on Hopper we see some wins. On Hopper, performance plateaus at 3 buffers,
# but on Blackwell we see benefits of 4 buffers. This suggests the throughput
# ratio has increased in favour of MMAs from Hopper to Blackwell. Noteworthy is
# our kernels are occupancy 1.
```

**EN:** The script benchmarks several `(BLOCK_K, num_buffers, num_warps)` combinations and prints TFLOPS. The embedded result comments interpret the measurements: Blackwell benefits more from deeper buffering, while Hopper reaches a plateau earlier, indicating a different balance between TMA throughput and MMA throughput.

**CN:** 脚本基准测试了多组 `(BLOCK_K, num_buffers, num_warps)` 配置并输出 TFLOPS。后面的结果注释给出了解读：Blackwell 更受益于更深的缓冲，而 Hopper 更早进入平台期，这说明两代架构在 TMA 吞吐与 MMA 吞吐之间的平衡不同。

### Lines 323-358 — Static persistent tile scheduler / 静态 persistent tile 调度器

```python
# To make the kernel persistent, all we have to do is put an outer loop around
# the kernel and iterate over the output tiles assigned to that kernel.
#
# Let's define a tile scheduler abstraction that will allow us to change the
# scheduling strategy, starting with a basic row-major tile scheduler.


@gluon.aggregate
class PersistentTileScheduler:
    pid_start: gl.tensor
    pid_end: gl.tensor
    num_pid_m: gl.tensor

    @gluon.jit
    def initialize(M, N, BLOCK_M: gl.constexpr, BLOCK_N: gl.constexpr):
        kernel_id = gl.program_id(axis=0)
        num_kernels = gl.num_programs(axis=0)
        num_pid_m = gl.cdiv(M, BLOCK_M)
        num_pid_n = gl.cdiv(N, BLOCK_N)
        num_pid = num_pid_m * num_pid_n
        pid_per_kernel = gl.cdiv(num_pid, num_kernels)
        pid_start = kernel_id * pid_per_kernel
        pid_end = min(pid_start + pid_per_kernel, num_pid)
        return PersistentTileScheduler(pid_start, pid_end, num_pid_m)

    @gluon.jit
    def get_num_tiles(self):
        return self.pid_end - self.pid_start

    @gluon.jit
    def get_tile(self, idx):
        # Delinearize the tile ID along M.
        pid = self.pid_start + idx
        pid_m = pid % self.num_pid_m
        pid_n = pid // self.num_pid_m
        return pid_m, pid_n
```

**EN:** `PersistentTileScheduler` is the minimal scheduling abstraction needed for persistence. Each resident program computes a contiguous linear tile interval `[pid_start, pid_end)`, and `get_tile` converts the linear ID back to `(pid_m, pid_n)` in row-major order.

**CN:** `PersistentTileScheduler` 是实现 persistent 执行所需的最小调度抽象。每个常驻 program 计算自己负责的线性 tile 区间 `[pid_start, pid_end)`，而 `get_tile` 再把线性编号还原成按行优先排列的 `(pid_m, pid_n)`。

### Lines 361-410 — Persistent matmul kernel / persistent matmul 内核

```python
# %%
# We can make the kernel persistent by literally placing the outer loop around
# the whole kernel, but let's re-use the TMA barrier and MMA state.
# We must scope the operand buffers to the inner loop so the shared memory
# allocator knows their liveranges do not intersect with the TMA store buffer.


@gluon.jit
def persistent_matmul_kernel(a_desc, b_desc, c_desc, MMAImpl: gl.constexpr, SchedulerImpl: gl.constexpr,
                             num_buffers: gl.constexpr, num_warps: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = a_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype
    K = a_desc.shape[1]

    bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(num_buffers):
        mbarrier.init(bars.index(i), count=1)
    # Producer and consumer indices.
    producer = 0
    consumer = 0

    mma = MMAImpl.initialize(dtype, BLOCK_M, BLOCK_N, num_warps)
    scheduler = SchedulerImpl.initialize(c_desc.shape[0], c_desc.shape[1], BLOCK_M, BLOCK_N)
    for idx in range(scheduler.get_num_tiles()):
        pid_m, pid_n = scheduler.get_tile(idx)
        off_m = pid_m * BLOCK_M
        off_n = pid_n * BLOCK_N

        a_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + a_desc.block_type.shape, a_desc.layout)
        b_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + b_desc.block_type.shape, b_desc.layout)
        for k in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
            producer = issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers)

        for k in range(BLOCK_K * (num_buffers - 2), K, BLOCK_K):
            producer = issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers)
            consumer, mma = issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers)

        for _ in gl.static_range(num_buffers - 2):
            consumer, mma = issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers)

        mma = mma.wait_num_outstanding(0)
        c_smem = gl.allocate_shared_memory(dtype, c_desc.block_type.shape, c_desc.layout)
        c, mma = mma.take_result()
        c_smem.store(c.to(dtype))
        fence_async_shared()
        tma.async_copy_shared_to_global(c_desc, [off_m, off_n], c_smem)
        tma.store_wait(pendings=0)

```

**EN:** The persistent kernel keeps barrier state, producer/consumer counters, and the MMA state alive across multiple output tiles. The outer loop comes from the scheduler; inside each iteration it reuses the same load/MMA pipeline structure, then performs the same shared-memory epilogue and asynchronous TMA store as the non-persistent version.

**CN:** persistent 内核让 barrier 状态、producer/consumer 计数器和 MMA 状态在多个输出 tile 之间持续存在。外层循环由调度器提供；每次迭代内部仍复用同样的加载/MMA 流水结构，然后执行与非 persistent 版本相同的共享内存收尾和异步 TMA 写回。

### Lines 412-458 — Persistent launcher, test, and benchmark / persistent 封装、测试与基准

```python
def persistent_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps, SchedulerImpl):
    M, N = C.shape
    MMAImpl = select_mma_impl()

    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gl.float16)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], gl.float16)
    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], gl.float16)
    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(B, [BLOCK_K, BLOCK_N], b_layout)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)

    num_sms = torch.cuda.get_device_properties("cuda").multi_processor_count
    num_pid = triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N)
    grid = (min(num_sms, num_pid), )
    persistent_matmul_kernel[grid](a_desc, b_desc, c_desc, MMAImpl, SchedulerImpl, num_buffers, num_warps=num_warps)


schedulers = [PersistentTileScheduler]


@pytest.mark.parametrize("M, N, K", [(2000, 1000, 2000)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 256, 64)])
@pytest.mark.parametrize("num_buffers", [2, 3, 4])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("SchedulerImpl", schedulers)
@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper or newer")
def test_persistent_matmul(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps, SchedulerImpl):
    torch.manual_seed(0)
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)
    persistent_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps, SchedulerImpl)
    torch.testing.assert_close(A @ B, C, rtol=1e-3, atol=1e-1)


if __name__ == "__main__" and not profiling_with_ncu:
    print("Benchmarking persistent matmul")
    print("==============================")
    print(f"BLOCK_M={BLOCK_M} BLOCK_N={BLOCK_N}")
    print("BLOCK_K num_buffers num_warps tflops/s")
    for (BLOCK_K, num_buffers), num_warps in itertools.product([(128, 2), (64, 3), (64, 4)], warps):
        print(f"{BLOCK_K:>7} {num_buffers:>11} {num_warps:>9}", end=" ")
        fn = lambda: persistent_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps,
                                       PersistentTileScheduler)
        ms = triton.testing.do_bench_cudagraph(fn)
        print(f"{get_flops(ms, M, N, K):8.2f}")
    print()
```

**EN:** The persistent host wrapper intentionally launches only up to one program per SM, or fewer if the tile count is smaller. That launch shape is what makes the kernel persistent: each resident program must consume several tiles. The accompanying test and benchmark compare this strategy to the earlier non-persistent pipeline.

**CN:** persistent 的主机封装会刻意把启动规模限制为“每个 SM 最多一个 program”，如果 tile 数更少则更小。正是这种启动方式使内核成为 persistent：每个常驻 program 必须处理多个 tile。配套测试和基准则拿它与前面的非 persistent 流水版本做对比。

### Lines 460-499 — Profiling notes and L2-locality motivation / 性能剖析说明与 L2 局部性动机

```python
# %%
# ```text
# BLOCK_K num_buffers num_warps  Blackwell  Hopper
#     128           2         4     712.25
#     128           2         8     686.64  502.84
#      64           3         4    1032.16
#      64           3         8     938.81  661.11
#      64           4         4    1142.26
#      64           4         8    1071.46  658.84
# ```
#
# The Hopper kernel sees a modest improvement, but the Blackwell kernel
# performance is slightly lower. Let's capture a profile of the kernels on
# Blackwell using ncu. Pass `profile` to this script's arguments to run the two
# kernels once.

if __name__ == "__main__" and profiling_with_ncu:
    matmul_pipelined(A, B, C, 128, 256, 64, 4, 4)
    persistent_matmul(A, B, C, 128, 256, 64, 4, 4, PersistentTileScheduler)

# %%
# There are many reasons the persistent kernel can be slower. Load imbalance can
# arise due to inefficient scheduling (work is not evenly distributed). But it
# can also arise from drift at runtime, such as some TMA accesses taking longer
# than others, which a static tile scheduler cannot compensate for.
#
# Another reason we suspect is the global memory access pattern:
#
# ```
# ncu --set full -o pipelined  --kernel-name matmul_pipelined_kernel  python 07-persistence.py profile
# ncu --set full -o persistent --kernel-name persistent_matmul_kernel python 07-persistence.py profile
# ncu --import  pipelined.ncu-rep | grep "L2 Hit Rate"
#     L2 Hit Rate                            %        61.11
# ncu --import persistent.ncu-rep | grep "L2 Hit Rate"
#     L2 Hit Rate                            %        52.93
# ```
#
# The persistent kernel's L2 hit rate is 10% lower. We can improve L2 efficiency
# by "super-grouping" the tiles along columns. See 03-matrix-multiplication.py
# for more details. Let's encode this strategy in a new tile scheduler.
```

**EN:** This comment-only section explains why persistence can underperform despite lower scheduling overhead. Static work assignment may create imbalance, but the tutorial also identifies a memory-system issue: the persistent kernel shows a lower L2 hit rate in Nsight Compute, which motivates a new tile order that improves cache reuse.

**CN:** 这一段纯注释解释了为什么 persistent 即使减少了调度开销也可能变慢。静态分工会带来负载不均，而教程还指出了内存系统问题：在 Nsight Compute 中，persistent 内核的 L2 命中率更低，因此需要设计新的 tile 顺序来改善缓存复用。

### Lines 502-557 — Grouped persistent scheduler / 分组 persistent 调度器

```python
def GroupedPersistentTileScheduler(GROUP_SIZE_M):
    # Bind this as a constexpr so it can be captured.
    GROUP_SIZE_M = gl.constexpr(GROUP_SIZE_M)

    # Like C++ templates!
    @gluon.aggregate
    class GroupedPersistentTileSchedulerImpl:
        start_pid: gl.tensor
        num_pid_m: gl.tensor
        num_pid_in_group: gl.tensor
        num_pid: gl.tensor

        @gluon.jit
        def initialize(M, N, BLOCK_M: gl.constexpr, BLOCK_N: gl.constexpr):
            start_pid = gl.program_id(axis=0)
            num_pid_m = gl.cdiv(M, BLOCK_M)
            num_pid_n = gl.cdiv(N, BLOCK_N)
            num_pid_in_group = GROUP_SIZE_M * num_pid_n
            num_pid = num_pid_m * num_pid_n
            return GroupedPersistentTileSchedulerImpl(start_pid, num_pid_m, num_pid_in_group, num_pid)

        @gluon.jit
        def get_num_tiles(self):
            return gl.cdiv(self.num_pid - self.start_pid, gl.num_programs(axis=0))

        @gluon.jit
        def get_tile(self, idx):
            tile_id = self.start_pid + idx * gl.num_programs(axis=0)
            group_id = tile_id // self.num_pid_in_group
            first_pid_m = group_id * GROUP_SIZE_M
            group_size_m = min(self.num_pid_m - first_pid_m, GROUP_SIZE_M)
            pid_m = first_pid_m + (tile_id % group_size_m)
            pid_n = (tile_id % self.num_pid_in_group) // group_size_m
            return pid_m, pid_n

    GroupedPersistentTileSchedulerImpl.__name__ = f"GroupedPersistentTileScheduler({GROUP_SIZE_M.value})"
    return GroupedPersistentTileSchedulerImpl


# Add this to the testsuite.
schedulers += [GroupedPersistentTileScheduler(1), GroupedPersistentTileScheduler(8)]

if __name__ == "__main__" and not profiling_with_ncu:
    num_warps = 8 if is_hopper else 4
    num_buffers = 3 if is_hopper else 4
    print("Benchmarking grouped scheduler")
    print("=============================")
    print(f"BLOCK_M={BLOCK_M} BLOCK_N={BLOCK_N} BLOCK_K={BLOCK_K}")
    print(f"num_buffers={num_buffers} num_warps={num_warps}")
    print("GROUP_SIZE_M tflops/s")
    for GROUP_SIZE_M in [1, 2, 4, 6, 8]:
        print(f"{GROUP_SIZE_M:>12}", end=" ")
        fn = lambda: persistent_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps,
                                       GroupedPersistentTileScheduler(GROUP_SIZE_M))
        ms = triton.testing.do_bench_cudagraph(fn)
        print(f"{get_flops(ms, M, N, K):8.2f}")
```

**EN:** `GroupedPersistentTileScheduler` is a scheduler factory that captures `GROUP_SIZE_M` as a constexpr, then returns a specialized aggregate class. Its tile mapping walks tiles in grouped column-major chunks, similar in spirit to Triton grouped GEMM scheduling, and the benchmark loop measures which grouping gives the best locality/performance trade-off.

**CN:** `GroupedPersistentTileScheduler` 是一个调度器工厂，它把 `GROUP_SIZE_M` 绑定为 constexpr，然后返回专门化的 aggregate 类。它的 tile 映射按列方向的分组块遍历，思路与 Triton 常见的 grouped GEMM 调度相近；后面的基准循环则测量哪种分组在局部性与性能之间最优。

### Lines 560-598 — Why outer-loop pipelining is harder / 为什么外层循环流水更难

```python
# %%
# ```text
# GROUP_SIZE_M Blackwell  Hopper
#            1   1025.11  649.09
#            2   1050.43  651.32
#            4   1032.71  655.51
#            6   1057.27  652.39
#            8   1179.94  648.42
# ```
#
# At GROUP_SIZE_M=8, we recover performance on Blackwell. In fact, under ncu we
# see the L2 hit rate increases to 70%, which suggests there are other ways to
# improve the scheduling.
#
# Performance decreases on Hopper with this scheduler. The L2 hit rate of the
# persistent kernel is 86% and 89% for the non-persistent kernel. The grouped
# scheduler does not affect the L2 hit rate but it does increase load imbalance.

# %%
# Pipelining across the outer loop benefits smaller K shapes more because a
# larger proportion of time is spent in the epilogue. We can try overlapping the
# TMA store with the next tile by rotating the TMA store wait.
#
# However, this causes the liverange of the TMA store buffer to overlap with the
# operand buffers, decreasing our max num_buffers to 3. While Hopper is fine
# with 3 buffers, on Blackwell performance can suffer. There are 3 remedies:
#
# 1. Use gl.store which does not require shared memory but it cannot be
#    pipelined. However, the layout conversion requires shared memory.
# 2. Break up the TMA store to multiple steps, allowing us to use smaller
#    buffers, we will only be able to pipeline the last step.
#    reduces the amount of overlap.
# 3. Borrow one of the b_bufs.
#
# For BLOCK_{M,N,K} = (128, 256, 64), one B buffer is half the size of the
# accumulator, but we have enough memory to use 5 buffers for B just so that we
# can steal two buffers for the epilogue, even though the inner loop only uses
# 4 at a time.

```

**EN:** The tutorial now tries to overlap the epilogue store of one tile with the loads/compute of the next tile. The challenge is shared-memory lifetime overlap: the epilogue buffer wants to live at the same time as the operand buffers. The comments list three remedies and choose the buffer-stealing approach for the tutorial implementation.

**CN:** 教程接下来尝试把一个 tile 的收尾写回与下一个 tile 的加载/计算重叠起来。难点在于共享内存生命周期重叠：收尾缓冲与操作数缓冲想同时存在。注释列出了三种办法，最终教程选择“偷用缓冲区”的实现路线。

### Lines 600-623 — Pipeline helpers with B-buffer stealing / 支持偷用 B 缓冲的流水辅助函数

```python
# Forked versions of issue_loads and issue_mma that support `stealb`.
@gluon.jit
def issue_loads_stealb(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, stealb: gl.constexpr,
                       num_buffers: gl.constexpr, pred=True):
    index = producer % num_buffers
    b_index = producer % (num_buffers + stealb)
    producer += 1
    bar = bars.index(index)
    mbarrier.expect(bar, a_desc.block_type.nbytes + b_desc.block_type.nbytes, pred=pred)
    tma.async_load(a_desc, [off_m, k], bar, a_bufs.index(index), pred)
    tma.async_load(b_desc, [k, off_n], bar, b_bufs.index(b_index), pred)
    return producer


@gluon.jit
def issue_mma_stealb(consumer, mma, bars, a_bufs, b_bufs, stealb: gl.constexpr, num_buffers: gl.constexpr):
    index = consumer % num_buffers
    b_index = consumer % (num_buffers + stealb)
    phase = consumer // num_buffers & 1
    consumer += 1
    mbarrier.wait(bars.index(index), phase)
    mma = mma.wait_num_outstanding(0)
    mma = mma.issue_async_mma(a_bufs.index(index), b_bufs.index(b_index))
    return consumer, mma
```

**EN:** These variants keep A-buffer indexing unchanged but allow B to use `num_buffers + stealb` slots. That detail matters because the epilogue may temporarily borrow one or two B slots for storing the output tile before a TMA store, while the barrier protocol still tracks only the active compute-stage buffers.

**CN:** 这些变体保持 A 缓冲的索引方式不变，但允许 B 使用 `num_buffers + stealb` 个槽位。这一点很关键，因为收尾阶段可能暂时借用一个或两个 B 槽位来暂存输出 tile，再通过 TMA 写回；与此同时，barrier 协议仍只跟踪计算阶段真正使用的缓冲。

### Lines 626-650 — Persistent pipelined kernel setup / persistent 流水内核的初始化

```python
@gluon.jit
def persistent_matmul_pipelined_kernel(a_desc, b_desc, c_desc, c_half_desc, MMAImpl: gl.constexpr,
                                       SchedulerImpl: gl.constexpr, num_buffers: gl.constexpr, STEALB: gl.constexpr,
                                       num_warps: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = a_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype
    K = a_desc.shape[1]

    # All buffers share the same liverange.
    gl.static_assert(num_buffers >= 3, "expected at least 3 buffers")
    a_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + a_desc.block_type.shape, a_desc.layout)
    # Add an extra B buffer when stealing.
    b_bufs = gl.allocate_shared_memory(dtype, [num_buffers + STEALB] + b_desc.block_type.shape, b_desc.layout)
    if not STEALB:
        c_smem = gl.allocate_shared_memory(dtype, c_desc.block_type.shape, c_desc.layout)
    else:
        gl.static_assert(BLOCK_M == BLOCK_K or BLOCK_M == 2 * BLOCK_K,
                         "expected one or two B tiles to cover the epilogue tile")
    bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(num_buffers):
        mbarrier.init(bars.index(i), count=1)
    producer = 0
    consumer = 0
```

**EN:** The kernel setup now allocates reusable operand buffers up front, optionally skips a dedicated `c_smem` buffer when stealing B storage, and enforces `num_buffers >= 3` because outer-loop pipelining needs more slack than the earlier kernel. It also builds the scheduler and records the total tile count for the persistent loop.

**CN:** 这个内核的初始化会提前分配可复用的操作数缓冲；如果采用偷用 B 缓冲的方案，就不再单独分配 `c_smem`。由于外层循环流水需要比前一个内核更大的缓冲余量，这里还强制要求 `num_buffers >= 3`，并建立调度器、记录 persistent 循环要处理的 tile 总数。

### Lines 652-723 — Outer-loop pipelining and epilogue overlap / 外层循环流水与收尾重叠

```python
    mma = MMAImpl.initialize(dtype, BLOCK_M, BLOCK_N, num_warps)
    scheduler = SchedulerImpl.initialize(c_desc.shape[0], c_desc.shape[1], BLOCK_M, BLOCK_N)
    num_tiles = scheduler.get_num_tiles()

    # Peeled inner loop prologue.
    idx = 0
    pid_m, pid_n = scheduler.get_tile(idx)
    off_m = pid_m * BLOCK_M
    off_n = pid_n * BLOCK_N
    for ki in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
        producer = issue_loads_stealb(producer, a_desc, b_desc, off_m, off_n, ki, bars, a_bufs, b_bufs, STEALB,
                                      num_buffers)
    k = BLOCK_K * (num_buffers - 2)
    producer = issue_loads_stealb(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, STEALB, num_buffers)

    for _ in range(num_tiles):
        consumer, mma = issue_mma_stealb(consumer, mma, bars, a_bufs, b_bufs, STEALB, num_buffers)
        if STEALB:
            # Wait for the epilogue before the first TMA load.
            tma.store_wait(pendings=0)
        for k in range(BLOCK_K * (num_buffers - 1), K, BLOCK_K):
            producer = issue_loads_stealb(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, STEALB,
                                          num_buffers)
            consumer, mma = issue_mma_stealb(consumer, mma, bars, a_bufs, b_bufs, STEALB, num_buffers)

        epilogue_off_m = off_m
        epilogue_off_n = off_n

        # Peel the next prologue and fuse it with the pipeline drain loop.
        idx += 1
        pid_m, pid_n = scheduler.get_tile(idx)
        off_m = pid_m * BLOCK_M
        off_n = pid_n * BLOCK_N
        # Predicate the peeled prologue instead of using a conditional.
        pred = idx < num_tiles
        for ki in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
            producer = issue_loads_stealb(producer, a_desc, b_desc, off_m, off_n, ki, bars, a_bufs, b_bufs, STEALB,
                                          num_buffers, pred)
            consumer, mma = issue_mma_stealb(consumer, mma, bars, a_bufs, b_bufs, STEALB, num_buffers)
        k = BLOCK_K * (num_buffers - 2)
        producer = issue_loads_stealb(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, STEALB,
                                      num_buffers)

        mma = mma.wait_num_outstanding(0)
        use_split_n_load: gl.constexpr = STEALB and BLOCK_M != BLOCK_K
        c, mma = mma.take_result(splitn=use_split_n_load)
        c = c.to(dtype)
        if not STEALB:
            c_buf = c_smem
            tma.store_wait(pendings=0)
            c_buf.store(c)
            fence_async_shared()
            tma.async_copy_shared_to_global(c_desc, [epilogue_off_m, epilogue_off_n], c_buf)
        elif BLOCK_M == BLOCK_K:
            c_buf = b_bufs.index(producer % (num_buffers + STEALB))
            c_buf.store(c)
            fence_async_shared()
            tma.async_copy_shared_to_global(c_desc, [epilogue_off_m, epilogue_off_n], c_buf)
        else:
            # Steal the next 2 B buffers for the epilogue.
            c0, c1 = c.reshape((BLOCK_M, 2, BLOCK_N // 2)).permute(0, 2, 1).split()
            c0_buf = b_bufs.index(producer % (num_buffers + STEALB))._reinterpret(shape=c_half_desc.block_type.shape,
                                                                                  layout=c_half_desc.layout)
            c1_buf = b_bufs.index(
                (producer + 1) % (num_buffers + STEALB))._reinterpret(shape=c_half_desc.block_type.shape,
                                                                      layout=c_half_desc.layout)
            c0_buf.store(c0)
            c1_buf.store(c1)
            fence_async_shared()
            tma.async_copy_shared_to_global(c_half_desc, [epilogue_off_m, epilogue_off_n], c0_buf)
            tma.async_copy_shared_to_global(c_half_desc, [epilogue_off_m, epilogue_off_n + BLOCK_N // 2], c1_buf)
    tma.store_wait(pendings=0)
```

**EN:** This is the most sophisticated part of the tutorial. The kernel peels a prologue for the first tile, then in each iteration overlaps current-tile MMA work, next-tile TMA prefetch, and previous-tile epilogue storage. When `STEALB` is enabled, the output tile may be written through one stolen B buffer or split across two reinterpreted half-width buffers so shared memory can be reused without allocating a separate epilogue tile.

**CN:** 这是整份教程中最复杂的部分。内核先为第一个 tile 剥离出一个前导阶段，然后在每次迭代中把“当前 tile 的 MMA”“下一个 tile 的 TMA 预取”“上一个 tile 的收尾写回”三件事重叠起来。启用 `STEALB` 时，输出 tile 会通过一个被借用的 B 缓冲写回，或者拆成两个半宽缓冲来写回，从而在不单独分配收尾缓冲的情况下复用共享内存。

### Lines 726-760 — Launcher and tests for the pipelined persistent kernel / 流水 persistent 内核的封装与测试

```python
def persistent_matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps, SchedulerImpl):
    M, N = C.shape
    MMAImpl = select_mma_impl()

    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gl.float16)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], gl.float16)
    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], gl.float16)
    c_half_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N // 2], gl.float16)

    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(B, [BLOCK_K, BLOCK_N], b_layout)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)
    c_half_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N // 2], c_half_layout)

    num_sms = torch.cuda.get_device_properties("cuda").multi_processor_count
    num_pid = triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N)
    grid = (min(num_sms, num_pid), )
    stealb = num_buffers == 4
    persistent_matmul_pipelined_kernel[grid](a_desc, b_desc, c_desc, c_half_desc, MMAImpl, SchedulerImpl, num_buffers,
                                             STEALB=stealb, num_warps=num_warps)


@pytest.mark.parametrize("M, N, K", [(208, 416, 304), (2000, 1000, 2000)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(64, 64, 64), (128, 256, 64)])
@pytest.mark.parametrize("num_buffers", [3, 4])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("SchedulerImpl", schedulers)
@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper or newer")
def test_persistent_matmul_pipelined(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps, SchedulerImpl):
    torch.manual_seed(0)
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)
    persistent_matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, num_warps, SchedulerImpl)
    torch.testing.assert_close(A @ B, C, rtol=1e-3, atol=1e-1)
```

**EN:** The wrapper prepares both a full-width and half-width output descriptor because the epilogue-stealing path may need two half-tile TMA stores. It then infers `stealb` from the 4-buffer configuration and validates the kernel on both small and large shapes across multiple schedulers.

**CN:** 这个封装同时准备了全宽和半宽的输出描述符，因为偷用缓冲的收尾路径可能需要两次半 tile 的 TMA 写回。随后它根据 4-buffer 配置推断 `stealb`，并在多种调度器、大小矩阵上验证内核正确性。

### Lines 763-790 — End-to-end benchmark sweep / 端到端基准扫描

```python
if __name__ == "__main__":
    args = {
        "BLOCK_M": 128,
        "BLOCK_N": 256,
        "BLOCK_K": 64,
        "num_buffers": 3 if is_hopper else 4,
        "num_warps": 8 if is_hopper else 4,
    }
    scheduler = PersistentTileScheduler if is_hopper else GroupedPersistentTileScheduler(8)
    nonpersistent = partial(matmul_pipelined, **args)
    persistent = partial(persistent_matmul, **args, SchedulerImpl=scheduler)
    persistent_pipelined = partial(persistent_matmul_pipelined, **args, SchedulerImpl=scheduler)

    M, N = 8192, 8192
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)
    print("Benchmarking pipelined persistent")
    print("=================================")
    print("    K     nonpersistent    persistent   pipelined    cublas")
    for K in [2**i for i in range(9, 15)]:
        as_flops = partial(get_flops, M=M, N=N, K=K)
        A = torch.randn(M, K, device="cuda", dtype=torch.float16)
        B = torch.randn(K, N, device="cuda", dtype=torch.float16)
        BT = B.T.contiguous()
        r0 = as_flops(triton.testing.do_bench_cudagraph(lambda: nonpersistent(A, B, C)))
        r1 = as_flops(triton.testing.do_bench_cudagraph(lambda: persistent(A, B, C)))
        r2 = as_flops(triton.testing.do_bench_cudagraph(lambda: persistent_pipelined(A, B, C)))
        r3 = as_flops(triton.testing.do_bench(lambda: cublas.matmul(A, BT, C)))
        print(f"{K:>5} {r0:>17.2f} {r1:>13.2f} {r2:>11.2f} {r3:>9.2f}")
```

**EN:** The final driver chooses architecture-specific defaults, picks the grouped scheduler only on Blackwell, and compares three Triton variants against cuBLAS over a sweep of K values. This benchmark captures the practical question of the tutorial: when does persistence and outer-loop pipelining actually win?

**CN:** 最后的驱动代码会选择与架构匹配的默认参数：Blackwell 使用分组调度器，Hopper 使用基础调度器；然后在一组不同的 K 值上，把三种 Triton 方案与 cuBLAS 做对比。这个基准直接回答了教程最实际的问题：persistent 与外层循环流水究竟在什么场景下真正占优？

### Lines 792-850 — Result interpretation and takeaways / 结果解读与总结

```python
# %%
# ```text
# Blackwell results:
#
#     K     nonpersistent    persistent   pipelined    cublas
#   512            615.86        828.70      993.50   1108.11
#  1024            997.16       1077.28     1173.31   1347.44
#  2048           1152.74       1190.55     1133.37   1435.01
#  4096           1164.05       1120.92     1143.47   1563.98
#  8192           1160.93       1074.97     1185.40   1491.84
# 16384           1185.62       1096.34     1296.93   1548.42
# ```
#
# ```text
# Hopper results:
#
#     K     nonpersistent    persistent   pipelined    cublas
#   512            491.74        485.01      539.88    588.15
#  1024            554.24        575.02      602.52    588.32
#  2048            573.87        594.72      625.91    615.58
#  4096            609.36        630.10      640.48    646.30
#  8192            629.44        646.22      661.57    661.11
# 16384            653.79        660.29      670.00    665.49
# ```
#
# Persistent matmul, when pipelined, gains more performance relative to
# nonpersistent at lower K, as we would expect. Load balancing can be
# particularly difficult when the number of SMs do not evenly divide the number
# of blocks, and with 8192x8192, we are smack in the middle with ~13.5 and
# ~15.5 blocks per SM for Hopper and Blackwell, respectively.
#
# On Hopper, our pipelined kernel is competitive with cublas, even pulling ahead
# for medium-sized K. However, cublas has a definitive advantage at low K. On
# Blackwell, it's not even close: cublas is significantly faster.
#
# Some matmul performance takes:
#
# - On Hopper, software pipelining is sufficient to reach peak performance for
#   medium and large K.
# - cublas uses 2-CTA matmul, which uses distributed shared memory to allow
#   256x256 instruction shape. 2-CTA support in Gluon is very spotty,
#   but this enables cublas to more efficiently feed the MMA, which matters more
#   on Blackwell due to the relative increase in MMA throughput vs TMA.
# - cublas matmul is warp-specialized which is necessary on Hopper to fully
#   overlap the epilogue at small K.
# - Our Blackwell implementation is limited by the shared API we designed for
#   Hopper and Blackwell: we are not double-buffering the accumulator and
#   leaving 256 columns of TMEM unused.
# - On Blackwell, we can use `clusterlaunchcontrol` to dynamically schedule
#   work in conjunction with the GPU, getting the best of both worlds. This is
#   explored further in tutorial 12.
#
# Main takeaways:
#
# - Persistent kernels replace GPU block scheduling with a (typically) static
#   schedule. This allows more resource and compute coordination/overlap between
#   blocks at the cost of losing dynamic scheduling.
# - Persistent kernels tend to benefit smaller problem sizes, but still deliver
#   benefits for large problem sizes.
```

**EN:** The closing comments explain the measured behavior rather than adding new code. They conclude that persistent kernels are most valuable when they unlock extra overlap or reduce small-problem overhead, but they also expose the implementation to load-balance and locality issues. The final notes explain why Hopper looks stronger here and why Blackwell needs more advanced scheduling or kernel structure to match cuBLAS.

**CN:** 最后的注释主要解释实验现象，而不是引入新代码。结论是：persistent kernel 的价值主要体现在额外的执行重叠或减少小问题上的调度开销，但它也更容易暴露负载均衡与局部性问题。末尾还说明了为什么这里 Hopper 的表现更强，以及为什么 Blackwell 需要更高级的调度或内核结构才能接近 cuBLAS。

## Key Concepts / 关键概念

- **Persistent kernels / 常驻内核**: Launching at most one program per SM keeps programs resident and lets each one process multiple output tiles, trading dynamic block scheduling for explicit software scheduling. / 每个 SM 最多启动一个 program，让 program 常驻并处理多个输出 tile，用显式的软件调度换取对 GPU 动态块调度的控制。

- **Unified MMA abstraction / 统一的 MMA 抽象**: `WGMMA` and `MMAv5` present the same API so the matmul pipeline can target Hopper and Blackwell with one kernel structure. / `WGMMA` 与 `MMAv5` 暴露相同 API，使同一套 matmul 流水结构可以同时覆盖 Hopper 与 Blackwell。

- **TMA + mbarrier pipeline / TMA 与 mbarrier 流水**: `tma.async_load`, `mbarrier.expect`, and `mbarrier.wait` form the producer-consumer protocol that overlaps global-memory movement with tensor-core compute. / `tma.async_load`、`mbarrier.expect` 与 `mbarrier.wait` 组成了生产者-消费者协议，用于把全局内存搬运与张量核心计算重叠起来。

- **Tile scheduling / Tile 调度**: The tutorial compares row-major static scheduling with grouped scheduling to show how tile order changes L2 locality and load balance. / 教程对比了按行的静态调度和分组调度，展示 tile 顺序如何影响 L2 局部性与负载均衡。

- **Outer-loop pipelining / 外层循环流水**: The later kernel overlaps the current tile epilogue with the next tile prologue and compute, extending software pipelining beyond the inner K loop. / 后半部分内核把当前 tile 的收尾与下一个 tile 的前导加载和计算重叠起来，把软件流水从内层 K 循环扩展到了外层 tile 循环。

- **Buffer stealing / 缓冲偷用**: Reusing extra B-buffer slots for the epilogue is a memory-layout trick that avoids dedicating separate shared memory for the output tile. / 复用额外的 B 缓冲槽位来完成收尾，是一种共享内存布局技巧，可避免专门为输出 tile 额外分配缓冲。

## Dependencies / 依赖关系

- `torch`: allocates CUDA tensors, queries device capabilities/SM count, seeds random inputs, and validates results with `torch.testing.assert_close`. / `torch` 用于分配 CUDA 张量、查询设备能力和 SM 数、生成测试输入，并通过 `torch.testing.assert_close` 校验结果。

- `triton`: provides kernel launch syntax, `cdiv`, benchmarking helpers, and access to the active target backend. / `triton` 提供内核启动语法、`cdiv`、基准测试工具，以及当前目标后端信息。

- `triton.experimental.gluon` and `triton.experimental.gluon.language as gl`: provide `@gluon.jit`, `@gluon.aggregate`, descriptors, layouts, static ranges, and shared-memory allocation primitives. / `triton.experimental.gluon` 与 `triton.experimental.gluon.language as gl` 提供 `@gluon.jit`、`@gluon.aggregate`、描述符、布局、静态循环与共享内存分配原语。

- Hopper-specific Gluon ops: `TensorDescriptor`, `tma`, `mbarrier`, `fence_async_shared`, `warpgroup_mma`, `warpgroup_mma_wait`, and `warpgroup_mma_accumulator`. / Hopper 专用的 Gluon 原语包括 `TensorDescriptor`、`tma`、`mbarrier`、`fence_async_shared`、`warpgroup_mma`、`warpgroup_mma_wait` 与 `warpgroup_mma_accumulator`。

- Blackwell-specific Gluon ops: `TensorMemoryLayout`, `tensor_memory_descriptor`, `allocate_tensor_memory`, `tcgen05_mma`, and `tcgen05_commit`. / Blackwell 专用原语包括 `TensorMemoryLayout`、`tensor_memory_descriptor`、`allocate_tensor_memory`、`tcgen05_mma` 与 `tcgen05_commit`。

- `05-wgmma`: imported dynamically to reuse `pick_wgmma_layout`, which chooses a Hopper accumulator/register layout compatible with the selected tile shape and warp count. / 通过动态导入 `05-wgmma` 复用 `pick_wgmma_layout`，为 Hopper 选择与 tile 形状和 warp 数匹配的累加器/寄存器布局。

- `pytest`, `itertools`, `functools.partial`, `importlib`, and `sys`: support tests, benchmark parameter sweeps, function specialization, dynamic imports, and command-line argument handling. / `pytest`、`itertools`、`functools.partial`、`importlib` 与 `sys` 分别用于测试、参数扫描、函数偏应用、动态导入和命令行参数处理。

- Optional `triton._C.libtriton.nvidia.cublas.CublasLt`: used only when CUDA is available to compare Triton kernels against cuBLAS performance. / 可选的 `triton._C.libtriton.nvidia.cublas.CublasLt` 仅在 CUDA 可用时启用，用于把 Triton 内核与 cuBLAS 做性能对比。
