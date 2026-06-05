# 12-cluster-launch-control.py — Code Analysis / 代码分析
## Source / 来源
- Source file: `/root/xw/triton/python/tutorials/gluon/12-cluster-launch-control.py`
- Purpose: Shows how Blackwell Cluster Launch Control (CLC) can dynamically redistribute persistent matmul work and compares it with a static scheduler.
- 源文件：`/root/xw/triton/python/tutorials/gluon/12-cluster-launch-control.py`
- 作用：Shows how Blackwell Cluster Launch Control (CLC) can dynamically redistribute persistent matmul work and compares it with a static scheduler.

## Line-by-Line Analysis / 逐行分析
### Lines 1-117 — CLC motivation, imports, Blackwell guard, and scheduler definitions
```python
"""
Cluster Launch Control (CLC)
============================

Cluster Launch Control (CLC) is a Blackwell (SM100+) hardware feature that enables
dynamic work distribution between thread blocks. When a block finishes early, it can
cancel a not-yet-launched cluster and take over its work, improving load balancing.

This tutorial demonstrates:
1. The CLC API: try_cancel, is_canceled, get_first_ctaid
2. How to overlap CLC with computation to hide latency
3. A comparison with a statically scheduled persistent matmul

Key Insight
-----------
The critical optimization is issuing CLC during the TMA prologue and checking
the result after tile completion. This hides CLC latency behind computation.

CLC API
-------
- ``clc.try_cancel(result, mbar)``: Issue async CLC request to cancel a pending cluster
- ``clc_result = clc.load_result(result)``: Load CLC response into registers
- ``clc_result.is_canceled()``: Returns True if a cluster was successfully canceled
- ``clc_result.program_id(dim)``: Get the canceled cluster's program ID
"""

import torch
import triton
import pytest
import importlib

from triton.experimental import gluon
from triton.experimental.gluon import language as gl
from triton.experimental.gluon.nvidia.blackwell import TensorDescriptor
from triton.experimental.gluon.language.nvidia.blackwell import tma, mbarrier, fence_async_shared, clc


def is_blackwell():
    return torch.cuda.is_available() and torch.cuda.get_device_capability()[0] >= 10


if __name__ == "__main__" and not is_blackwell():
    raise RuntimeError("This tutorial requires Blackwell (SM100+) GPU")

# Re-use helpers from tutorial 7.
t7 = importlib.import_module("07-persistence")

# %%
# CLC Matmul Kernel
# -----------------
# This kernel processes its assigned tile, then attempts to steal additional work.
# CLC is issued during the prologue so the result is ready after tile completion.
#
# This is identical to the persistent_matmul_kernel from tutorial 7, except for the
# changed ClcTileScheduler interface to support dynamic scheduling.


@gluon.aggregate
class ClcTileScheduler:
    has_work: gl.tensor
    tile_id: gl.tensor
    clc_result_buf: gl.shared_memory_descriptor
    barrier: gl.shared_memory_descriptor
    phase: gl.tensor

    @gluon.jit
    def initialize(M, N, BLOCK_M, BLOCK_N):
        has_work = gl.to_tensor(True)
        starting_tile_id = gl.program_id(0)

        barrier = mbarrier.allocate_mbarrier()
        clc_result_buffer = gl.allocate_shared_memory(gl.int64, [2], gl.SwizzledSharedLayout(1, 1, 1, [0]))
        mbarrier.init(barrier, count=1)
        phase = gl.to_tensor(0)
        return ClcTileScheduler(has_work, starting_tile_id, clc_result_buffer, barrier, phase)

    @gluon.jit
    def try_cancel(self) -> None:
        clc.try_cancel(self.clc_result_buf, self.barrier)
        mbarrier.expect(self.barrier, 16)

    @gluon.jit
    def advance(self):
        mbarrier.wait(self.barrier, self.phase)
        clc_res = clc.load_result(self.clc_result_buf)
        has_work = clc_res.is_canceled()
        next_tile_id = clc_res.program_id(0)
        return ClcTileScheduler(has_work, next_tile_id, self.clc_result_buf, self.barrier, self.phase ^ 1)


# %%
# We also implement a static scheduler that conforms to the same interface,
# so we can directly compare the benifits of dynamic scheduling.


@gluon.aggregate
class StaticTileScheduler:
    has_work: gl.tensor
    tile_id: gl.tensor
    num_tiles: gl.tensor

    @gluon.jit
    def initialize(M, N, BLOCK_M, BLOCK_N):
        starting_tile_id = gl.program_id(0)
        num_tiles = gl.cdiv(M, BLOCK_M) * gl.cdiv(N, BLOCK_N)
        has_work = starting_tile_id < num_tiles
        return StaticTileScheduler(has_work, starting_tile_id, num_tiles)

    @gluon.jit
    def try_cancel(self) -> None:
        pass

    @gluon.jit
    def advance(self):
        next_tile_id = self.tile_id + gl.num_programs(0)
        has_work = next_tile_id < self.num_tiles
        return StaticTileScheduler(has_work, next_tile_id, self.num_tiles)
```
**EN:** The module docstring explains the Blackwell CLC model: a block can attempt to cancel a not-yet-launched cluster and steal its work, which helps smooth out runtime imbalance. `ClcTileScheduler` wraps the CLC request/result protocol with a shared-memory result buffer and mbarrier, while `StaticTileScheduler` exposes the same interface without dynamic work stealing so the matmul kernel can compare both strategies.

**CN:** 模块文档字符串先说明 Blackwell 的 CLC 模型：一个 block 可以尝试取消尚未启动的 cluster 并接管它的工作，从而缓解运行时负载不均。`ClcTileScheduler` 用 shared-memory 结果缓冲区和 mbarrier 封装了 CLC 请求/返回流程，而 `StaticTileScheduler` 则提供相同接口但不做动态偷取任务，方便同一个 matmul kernel 对比两种调度策略。
### Lines 120-181 — Tile-id to `(pid_m, pid_n)` mapping and the persistent matmul kernel
```python
@triton.jit
def _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M):
    group_id = tile_id // num_pid_in_group
    first_pid_m = group_id * GROUP_SIZE_M
    group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
    pid_m = first_pid_m + (tile_id % group_size_m)
    pid_n = (tile_id % num_pid_in_group) // group_size_m
    return pid_m, pid_n


@gluon.jit
def persistent_matmul_kernel(a_desc, b_desc, c_desc, MMAImpl: gl.constexpr, SchedulerImpl: gl.constexpr,
                             num_buffers: gl.constexpr, num_warps: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = a_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype
    K = a_desc.shape[1]
    N = c_desc.shape[0]
    M = c_desc.shape[1]

    num_pid_n = gl.cdiv(N, BLOCK_N)
    num_pid_m = gl.cdiv(M, BLOCK_M)
    GROUP_SIZE_M: gl.constexpr = 8
    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(num_buffers):
        mbarrier.init(bars.index(i), count=1)
    # Producer and consumer indices.
    producer = 0
    consumer = 0

    mma = MMAImpl.initialize(dtype, BLOCK_M, BLOCK_N, num_warps)
    scheduler = SchedulerImpl.initialize(M, N, BLOCK_M, BLOCK_N)
    while scheduler.has_work:
        pid_m, pid_n = _compute_pid(scheduler.tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M)
        off_m = pid_m * BLOCK_M
        off_n = pid_n * BLOCK_N

        scheduler.try_cancel()

        a_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + a_desc.block_type.shape, a_desc.layout)
        b_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + b_desc.block_type.shape, b_desc.layout)
        for k in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
            producer = t7.issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers)

        for k in range(BLOCK_K * (num_buffers - 2), K, BLOCK_K):
            producer = t7.issue_loads(producer, a_desc, b_desc, off_m, off_n, k, bars, a_bufs, b_bufs, num_buffers)
            consumer, mma = t7.issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers)

        for _ in gl.static_range(num_buffers - 2):
            consumer, mma = t7.issue_mma(consumer, mma, bars, a_bufs, b_bufs, num_buffers)

        mma = mma.wait_num_outstanding(0)
        c_smem = gl.allocate_shared_memory(dtype, c_desc.block_type.shape, c_desc.layout)
        c, mma = mma.take_result()
        c_smem.store(c.to(dtype))
        fence_async_shared()
        tma.async_copy_shared_to_global(c_desc, [off_m, off_n], c_smem)
        tma.store_wait(pendings=0)
        scheduler = scheduler.advance()
```
**EN:** `_compute_pid` preserves the grouped traversal pattern from the persistence tutorial, converting a linear tile id into grouped M/N coordinates. `persistent_matmul_kernel` keeps the familiar TMA + MMA pipeline, but the key CLC optimization is that `scheduler.try_cancel()` is issued during the prologue so cancellation latency is hidden behind load/compute work, and `scheduler.advance()` decides whether the program keeps running on a stolen tile or exits.

**CN:** `_compute_pid` 延续了 persistence 教程里的 grouped 遍历模式，把线性 tile id 映射回分组后的 M/N 坐标。`persistent_matmul_kernel` 仍然是熟悉的 TMA + MMA 流水线，但关键优化在于：`scheduler.try_cancel()` 被放在 prologue 阶段发起，让取消请求的延迟被后续载入和计算隐藏；`scheduler.advance()` 则决定当前 program 是继续处理偷来的 tile，还是直接退出。
### Lines 184-218 — Python launcher and correctness test
```python
def run_matmul_kernel(A, B, C, BLOCK_M=128, BLOCK_N=256, BLOCK_K=64, num_buffers=3, num_warps=4, use_clc=True):
    M, N = C.shape
    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gl.float16)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], gl.float16)
    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], gl.float16)
    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(B, [BLOCK_K, BLOCK_N], b_layout)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)

    if use_clc:
        num_pid_m = triton.cdiv(M, BLOCK_M)
        num_pid_n = triton.cdiv(N, BLOCK_N)
        grid = num_pid_m * num_pid_n
        SchedulerImpl = ClcTileScheduler
    else:
        dev_props = torch.cuda.get_device_properties(A.device)
        grid = dev_props.multi_processor_count
        SchedulerImpl = StaticTileScheduler

    MMAImpl = t7.MMAv5
    persistent_matmul_kernel[(grid, )](a_desc, b_desc, c_desc, MMAImpl, SchedulerImpl, num_buffers, num_warps=num_warps)


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
@pytest.mark.parametrize("M, N, K", [(8192, 8192, 8192), (1000, 1000, 1000)])
@pytest.mark.parametrize("use_clc", [True, False])
def test_op(M, N, K, use_clc):
    A = torch.randn(M, K, device='cuda', dtype=torch.float16)
    B = torch.randn(K, N, device='cuda', dtype=torch.float16)
    C = torch.empty(M, N, device='cuda', dtype=torch.float16)

    C_ref = torch.mm(A, B)
    run_matmul_kernel(A, B, C, use_clc=use_clc)

    torch.testing.assert_close(C, C_ref)
```
**EN:** The launcher prepares A/B/C descriptors and then chooses the launch grid differently for the two schedulers: CLC mode launches one program per logical tile, whereas the static baseline launches a persistent grid sized by the number of SMs. The test runs both modes against `torch.mm`, verifying that dynamic tile stealing changes scheduling rather than numerical behavior.

**CN:** Python 启动器负责构造 A/B/C descriptor，并根据调度策略采用不同的 grid：CLC 模式按逻辑 tile 数启动程序，而静态基线则按 SM 数量启动 persistent grid。测试会分别运行这两种模式并与 `torch.mm` 对比，证明动态偷取 tile 只改变调度方式，不改变数值结果。
### Lines 221-303 — Benchmark harness and performance interpretation
```python
# %%
# Benchmark
# ---------


def benchmark():
    print("=" * 60)
    print("Cluster Launch Control (CLC) Matmul - Blackwell")
    print("=" * 60)

    props = torch.cuda.get_device_properties(0)
    print(f"Device: {props.name}, SMs: {props.multi_processor_count}")

    M, N, K = 8192, 8192, 8192
    print(f"Matrix: {M}x{N}x{K}")

    A = torch.randn(M, K, device='cuda', dtype=torch.float16)
    B = torch.randn(K, N, device='cuda', dtype=torch.float16)
    C = torch.empty(M, N, device='cuda', dtype=torch.float16)

    # Static baseline
    def static_fn():
        run_matmul_kernel(A, B, C, use_clc=False)

    ms = triton.testing.do_bench_cudagraph(static_fn)
    static_tflops = t7.get_flops(ms, M, N, K)
    print(f"\nStatic:     {static_tflops:7.2f} TFLOPS")

    # CLC matmul
    def clc_fn():
        run_matmul_kernel(A, B, C)

    ms = triton.testing.do_bench_cudagraph(clc_fn)
    clc_tflops = t7.get_flops(ms, M, N, K)
    print(f"CLC:        {clc_tflops:7.2f} TFLOPS ({100*clc_tflops/static_tflops:.1f}% of static)")

    # Correctness check
    print("\nVerifying correctness...")
    A_test = torch.randn(1024, 1024, device='cuda', dtype=torch.float16)
    B_test = torch.randn(1024, 1024, device='cuda', dtype=torch.float16)
    C_ref = torch.mm(A_test, B_test)
    C_clc = torch.empty_like(C_ref)
    run_matmul_kernel(A_test, B_test, C_clc)
    torch.cuda.synchronize()
    max_diff = (C_ref - C_clc).abs().max().item()
    print(f"Max diff: {max_diff:.6f}")
    assert max_diff < 1.0, "Correctness check failed"
    print("✓ Correctness verified")


# %%
# A sample run of this benchmark may look like,
#
# ============================================================
# Cluster Launch Control (CLC) Matmul - Blackwell
# ============================================================
# Device: NVIDIA GB200, SMs: 152
# Matrix: 8192x8192x8192
#
# Static:     1040.13 TFLOPS
# CLC:        1080.74 TFLOPS (103.9% of static)
#
# Notice that we've achieved a 3.9% speedup (which will vary run to run),
# without improving the actual matmul computation at all. This is because there
# is always a slight variance between the time taken to compute each tile. For
# example, one may have inputs already cached in L2 and another might suffer a
# cache miss. With a static scheduler, the kernel takes as long as it takes the
# slowest SM to complete it's assigned `num_tiles / num_sms` tiles. However, CLC
# allows us to better balance the load by give more work to the SMs that finish
# early and less to those that are taking longer.
#
# This effect will be even more pronounced in kernels that have more run-time
# variation, e.g. in a ragged matmul where the k dim is different for different
# output tiles.
#
# Note that a similar effect can be achieved on pre-blackwell by using a global
# atomic counter to track the next available tile id. However, this requires
# additional run time overhead to zero out the counter before launching the
# kernel which may nullify the benefit for reasonably-balanced workloads.

if __name__ == "__main__":
    benchmark()
```
**EN:** The benchmark prints TFLOPS for the static and CLC variants, then performs a separate correctness sanity check on a smaller problem. The concluding commentary highlights the main lesson of CLC: even when the math kernel itself is unchanged, dynamic work redistribution can recover performance lost to per-tile variance such as cache misses or uneven memory latency.

**CN:** benchmark 会打印静态版本和 CLC 版本的 TFLOPS，并在一个较小问题上再做一次正确性检查。结尾注释强调了 CLC 的核心价值：即使数学计算本身完全不变，动态工作重分配也能弥补 tile 执行时间波动（例如 cache miss 或内存访问延迟差异）带来的性能损失。
## Key Concepts / 关键概念
- CLC lets Blackwell kernels reassign not-yet-launched work at runtime, reducing tail effects in persistent kernels / CLC 允许 Blackwell kernel 在运行时重分配尚未启动的工作，从而降低 persistent kernel 的尾部拖慢效应。
- The CLC request should be issued early enough that its latency overlaps with useful load/compute work / CLC 请求应尽早发起，使其延迟与实际载入或计算阶段重叠。
- A shared scheduler interface makes it easy to compare static and dynamic scheduling policies with the same kernel body / 通过统一的 scheduler 接口，可以用同一个 kernel 主体比较静态和动态调度策略。

## Dependencies / 依赖关系
- `clc`, `mbarrier`, `tma`, and `TensorDescriptor` are the hardware-facing building blocks for dynamic scheduling plus async GEMM staging / `clc`、`mbarrier`、`tma` 和 `TensorDescriptor` 是动态调度与异步 GEMM 预取的硬件接口基础。
- `07-persistence` is reused for the MMAv5 implementation, load helpers, and FLOP accounting / `07-persistence` 被复用于 MMAv5 实现、载入辅助函数和 FLOP 统计。
- `pytest` and `triton.testing.do_bench_cudagraph` provide correctness and benchmark coverage / `pytest` 与 `triton.testing.do_bench_cudagraph` 用于正确性与性能测试。
