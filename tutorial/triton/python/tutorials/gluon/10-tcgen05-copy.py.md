# 10-tcgen05-copy.py — Code Analysis / 代码分析
## Source / 来源
- Source file: `/root/xw/triton/python/tutorials/gluon/10-tcgen05-copy.py`
- Purpose: Demonstrates Blackwell `tcgen05_copy` for asynchronous shared-memory to tensor-memory transfer, then reuses it in a matmul-accumulate pipeline.
- 源文件：`/root/xw/triton/python/tutorials/gluon/10-tcgen05-copy.py`
- 作用：Demonstrates Blackwell `tcgen05_copy` for asynchronous shared-memory to tensor-memory transfer, then reuses it in a matmul-accumulate pipeline.

## Line-by-Line Analysis / 逐行分析
### Lines 1-157 — Tutorial overview, imports, Blackwell guard, and the first `tcgen05_copy` example
```python
"""
TCGen05 Copy Instruction
========================

This tutorial will cover the `tcgen05_copy` instruction: how to use it and its
applications.

The `tcgen05_copy` instruction is an asynchronous tensorcore operation that copies
data from shared memory to tensor memory. The completion of `tcgen05_copy` is
tracked with `tcgen05_commit` on an mbarrier just like `tcgen05_mma`. The
completion of a single or multiple `tcgen05_copy` operations can be tracked by a
single `tcgen05_commit`:

```python
tcgen05_copy(lhs_smem, lhs_tmem)
tcgen05_copy(acc_smem, acc_tmem)
tcgen05_commit(bar)
mbarrier.wait(bar, phase=phase)
acc = acc_tmem.load()
lhs = lhs_tmem.load()
```

`tcgen05_copy` can be used to copy data into tensor memory that is fed into a
`tcgen05_mma` instruction. Because `tcgen05_copy` is implicitly pipelined with
`tcgen05_mma`, even though it is asynchronous, the MMA is guaranteed to start
after the copy is complete:

```python
tcgen05_copy(smem, lhs_tmem)
tcgen05_mma(lhs_tmem, rhs_smem, acc_tmem)
tcgen05_commit(bar)
mbarrier.wait(bar, phase=phase)
```

The implicit pipelining is because the PTX-level `tcgen05.copy` and `tcgen05.mma`
instructions are executed by the tensor core pipe on the SM, which you can think
of as a single thread running tensor core specific instructions on the SM,
asynchronously from the rest of the SM. In other words, all `tcgen05_*` instructions
enqueue a tensor core operation on the tensor pipe, which are executed in order.

The following is also valid.

```python
tcgen05_copy(lhs_smem0, lhs_tmem)
tcgen05_mma(lhs_tmem, rhs_smem, acc_tmem)
tcgen05_commit(bar)

tcgen05_copy(lhs_smem1, lhs_tmem)
tcgen05_mma(lhs_tmem, rhs_smem, acc_tmem)
```

Because the second `tcgen05_copy` will only execute after the preceeding
`tcgen05_mma` is complete. In other words, `tcgen05_copy`, `tcgen05_mma`, and
`tcgen05_commit` are all implicitly pipelined and executed in order.

`tcgen05_copy` accesses shared memory via the async proxy, just like `tcgen05_mma`.
Make sure to insert fences as appropriate:

```python
lhs_smem.store(value1)
fence_async_shared()
tcgen05_copy(lhs_smem, lhs_tmem)
tcgen05_commit(bar)

mbarrier.wait(bar, phase=phase)
lhs_smem.store(value0)
```

Note that a fence is not needed between `tcgen05_copy` and the second write to
`lhs_smem` because waiting on the completion of the `tcgen05_copy` operation
via the mbarrier implicitly fences the generic and async proxies.

What makes using `tcgen05_copy` particularly tricky is selecting the right
shared memory and tensor memory layouts, as `tcgen05_copy` only supports a
limited set of instruction shapes for copy data from shared to tensor memory.
"""

import itertools
import importlib
import pytest
import triton
import torch
import triton.experimental.gluon as gluon
import triton.experimental.gluon.language as gl
from triton.experimental.gluon.nvidia.hopper import TensorDescriptor
from triton.experimental.gluon.language.nvidia.blackwell import (
    TensorMemoryLayout,
    tensor_memory_descriptor,
    allocate_tensor_memory,
    fence_async_shared,
    tcgen05_copy,
    tcgen05_commit,
    tcgen05_mma,
    mbarrier,
    tma,
)


def is_blackwell():
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] == 10


if __name__ == "__main__" and not is_blackwell():
    raise RuntimeError("This tutorial requires a Blackwell NVIDIA GPU")

# Re-use utilities from the previous tutorials.
t7 = importlib.import_module("07-persistence")
t8 = importlib.import_module("08-warp-specialization")

# %%
# Let's write an example kernel that uses `tcgen05_copy` and and show what the
# requirements are for the shared and tensor memory layouts.


@gluon.jit
def tcgen05_copy_kernel(in_ptr, in_stride0, in_stride1, out_ptr, out_stride0, out_stride1, M: gl.constexpr,
                        N: gl.constexpr, smem_layout: gl.constexpr, tmem_layout: gl.constexpr):
    coalesced_2d_layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, gl.num_warps()], [1, 0])
    offs_m = gl.arange(0, M, gl.SliceLayout(1, coalesced_2d_layout))
    offs_n = gl.arange(0, N, gl.SliceLayout(0, coalesced_2d_layout))

    input = gl.load(in_ptr + offs_m[:, None] * in_stride0 + offs_n[None, :] * in_stride1)

    # Allocate shared memory and tensor memory with the tile shape [M, N].
    smem = gl.allocate_shared_memory(input.dtype, (M, N), smem_layout)
    tmem = allocate_tensor_memory(input.dtype, (M, N), tmem_layout)

    bar = gl.allocate_shared_memory(gl.int64, [1], gl.constexpr(mbarrier.MBarrierLayout()))
    mbarrier.init(bar, count=1)

    # Copy data from shared memory to tensor memory.
    smem.store(input)
    # Fence generic and async proxies
    fence_async_shared()
    # Issue the async copy
    tcgen05_copy(smem, tmem)
    # Track completion of the async copy
    tcgen05_commit(bar)
    # Wait for the async copy to complete
    mbarrier.wait(bar, phase=0)
    mbarrier.invalidate(bar)

    # Read the data from tensor memory.
    output = tmem.load()

    # Write using a coalesced layout.
    output = gl.convert_layout(output, coalesced_2d_layout)
    gl.store(out_ptr + offs_m[:, None] * out_stride0 + offs_n[None, :] * out_stride1, output)


def tcgen05_copy_example(M, N, smem_layout, tmem_layout, dtype):
    input = torch.randn(M, N, dtype=dtype, device="cuda")
    output = torch.empty_like(input)
    tcgen05_copy_kernel[(1, )](input, *input.stride(), output, *output.stride(), M, N, smem_layout, tmem_layout)
    # Just check that the input and output are equal.
    torch.testing.assert_close(input, output, atol=0, rtol=0)
```
**EN:** The opening docstring explains the semantics of `tcgen05_copy`: it is asynchronous, completion is tracked with `tcgen05_commit` and `mbarrier.wait`, and it is implicitly ordered with later `tcgen05_mma` instructions on the tensor-core pipe. The first kernel allocates shared memory, tensor memory, and an mbarrier, stages a tile into shared memory, fences the async proxy with `fence_async_shared()`, copies into TMEM, waits for completion, and finally reads the value back to global memory for correctness checking.

**CN:** 开头的文档字符串先说明 `tcgen05_copy` 的语义：它是异步的，完成状态通过 `tcgen05_commit` 和 `mbarrier.wait` 跟踪，并且会与后续 `tcgen05_mma` 在 tensor-core pipe 上保持隐式顺序。第一个 kernel 分配 shared memory、tensor memory 和 mbarrier，把输入 tile 先写入 shared memory，再用 `fence_async_shared()` 同步 generic/async proxy，然后复制到 TMEM，等待完成后再读回到全局内存做正确性验证。
### Lines 160-203 — Valid layout search space and parameterized layout test
```python
# %%
# Let's first explore the valid shared memory layouts for the source of
# `tcgen05_copy` when the destination tensor memory layout is a
# `TensorMemoryLayout`, which is common when using TMAs and tensor core
# instructions.
#
# Recall that `TensorMemoryLayout` only supports 2D memory descriptors. When the
# destination tensor memory layout is a `TensorMemoryLayout`, the source shared
# memory layout is typically an `NVMMASharedLayout`. Other exotic layouts are
# supported, such as some `SharedLinearLayout`, but we won't cover them in this
# tutorial.
#
# Additional, the current restrictions apply to the `NVMMASharedLayout`:
# - The layout must be swizzled (swizzle_byte_width > 0).
# - The dtype must be 32-bit (e.g. gl.float32).
# - `TensorMemoryLayout` blockM must be 128.
# - The layout cannot be transposed.

configs = []
TMEM_BLOCK_M = 128
for TMEM_BLOCK_N in [1, 2, 4, 8, 16, 32, 64, 128, 256]:
    for M, N in itertools.product([128, 256], [16, 32, 64, 128, 256]):
        if M < TMEM_BLOCK_M or N < TMEM_BLOCK_N or M * N * 4 > 228 * 1024:
            continue
        configs.append((M, N, TMEM_BLOCK_N))


@pytest.mark.parametrize("M, N, TMEM_BLOCK_N", configs)
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize("swizzle", [32, 64, 128])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tcgen05_copy_nvmma_shared(M, N, TMEM_BLOCK_N, dtype, swizzle):
    bitwidth = dtype.itemsize * 8
    # There are still some shared memory layouts for which an implementation does not exist.
    if M == 256 and swizzle // TMEM_BLOCK_N >= 8:
        pytest.skip("no tcgen05.copy atom exists for codegen")
    # NVMMASharedLayout swizzle block shape has a minimum size.
    if N < swizzle // dtype.itemsize:
        pytest.skip("block shape along contiguous dimension is too small for the swizzle byte width")

    bitwidth = dtype.itemsize * 8
    smem_layout = gl.NVMMASharedLayout(swizzle_byte_width=swizzle, element_bitwidth=bitwidth, rank=2)
    tmem_layout = TensorMemoryLayout(block=(TMEM_BLOCK_M, TMEM_BLOCK_N), col_stride=32 // bitwidth)
    tcgen05_copy_example(M, N, smem_layout, tmem_layout, dtype)
```
**EN:** This block enumerates legal `(M, N, TMEM_BLOCK_N)` configurations for `TensorMemoryLayout` destinations and tests them with swizzled `NVMMASharedLayout` sources. The test encodes Blackwell-specific constraints: `TensorMemoryLayout` requires `blockM=128`, source layouts must be swizzled and 32-bit, and certain large/swizzle combinations are skipped because no matching `tcgen05.copy` atom exists in code generation.

**CN:** 这一段枚举了写入 `TensorMemoryLayout` 时合法的 `(M, N, TMEM_BLOCK_N)` 组合，并用带 swizzle 的 `NVMMASharedLayout` 作为源布局进行测试。测试里体现了 Blackwell 的限制：`TensorMemoryLayout` 要求 `blockM=128`，源 shared layout 必须是 swizzled 且元素宽度为 32-bit，某些尺寸和 swizzle 组合因为当前代码生成里没有对应的 `tcgen05.copy` atom 而被跳过。
### Lines 206-266 — Aggregate arguments and load partition for the matmul-accumulate pipeline
```python
# %%
# Although tcgen05_copy into TensorMemoryLayout only supports 32-bit dtypes,
# this is useful for writing matmul accumulate kernels: `D = A @ B + C`.
# Specifically, we can use TMA to load `C`, asynchronously copy it into tensor
# memory with `tcgen05_copy`, and then issue `tcgen05_mma` to perform the matmul
# while accumulating into tensor memory.
#
# We will use `gl.store` to write the output tiles to save shared memory, since
# C will require a large float32 buffer. We will use warp specialization to
# efficiently overlap the epilogue store with the rest of the kernel. Avoiding
# TMA for the epilogue store also reduces contention for the TMA pipe.


@gluon.aggregate
class PartitionArgs:
    a_desc: tma.tensor_descriptor
    b_desc: tma.tensor_descriptor
    c_desc: tma.tensor_descriptor
    d_ptr: gl.tensor
    d_stride_m: gl.tensor
    d_stride_n: gl.tensor
    a_bufs: gl.shared_memory_descriptor
    b_bufs: gl.shared_memory_descriptor
    load_empty_bars: gl.shared_memory_descriptor
    load_ready_bars: gl.shared_memory_descriptor
    c_buf: gl.shared_memory_descriptor
    c_empty_bar: gl.shared_memory_descriptor
    c_ready_bar: gl.shared_memory_descriptor
    acc_bufs: tensor_memory_descriptor
    acc_empty_bars: gl.shared_memory_descriptor
    acc_ready_bars: gl.shared_memory_descriptor
    SchedulerImpl: gl.constexpr


@gluon.jit
def matmul_accumulate_load_partition(p):
    BLOCK_M: gl.constexpr = p.c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = p.c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = p.a_desc.block_type.shape[1]
    K = p.a_desc.shape[1]

    c_phase = 1
    state = t8.Counter.create(1, p.load_empty_bars.shape[0])
    scheduler = p.SchedulerImpl.initialize(p.c_desc.shape[0], p.c_desc.shape[1], BLOCK_M, BLOCK_N)
    for idx in range(scheduler.get_num_tiles()):
        pid_m, pid_n = scheduler.get_tile(idx)
        off_m = pid_m * BLOCK_M
        off_n = pid_n * BLOCK_N
        # Issue the async TMA load for the C tile.
        mbarrier.wait(p.c_empty_bar, c_phase)
        mbarrier.expect(p.c_ready_bar, p.c_desc.block_type.nbytes)
        tma.async_load(p.c_desc, [off_m, off_n], p.c_ready_bar, p.c_buf)
        c_phase ^= 1
        # Inner loop loads.
        for k in range(0, K, BLOCK_K):
            bar = p.load_ready_bars.index(state.index)
            mbarrier.wait(p.load_empty_bars.index(state.index), state.phase)
            mbarrier.expect(bar, p.a_desc.block_type.nbytes + p.b_desc.block_type.nbytes)
            tma.async_load(p.a_desc, [off_m, k], bar, p.a_bufs.index(state.index))
            tma.async_load(p.b_desc, [k, off_n], bar, p.b_bufs.index(state.index))
            state = state.next()
```
**EN:** After the introductory comments, `PartitionArgs` packages descriptors, shared-memory buffers, tensor-memory buffers, and scheduler metadata so the warp-specialized partitions can share one structured argument object. `matmul_accumulate_load_partition` acts as the producer: it iterates over scheduler tiles, TMA-loads the `C` tile into shared memory, then pipelines `A` and `B` TMA loads into rotating stage buffers guarded by mbarriers.

**CN:** 在过渡说明之后，`PartitionArgs` 把 descriptor、shared memory buffer、tensor memory buffer 以及调度器元数据打包成一个聚合参数对象，方便 warp-specialized 各个 partition 共享。`matmul_accumulate_load_partition` 是生产者分区：它按照调度器遍历 tile，先把 `C` tile 通过 TMA 异步载入 shared memory，再把 `A`、`B` 的 TMA 载入流水化到循环轮转的 stage buffer 中，并用 mbarrier 管理依赖。
### Lines 270-331 — MMA partition and epilogue partition
```python
def matmul_accmulate_mma_partition(p):
    BLOCK_M: gl.constexpr = p.c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = p.c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = p.a_desc.block_type.shape[1]
    K = p.a_desc.shape[1]

    c_phase = 0
    load_state = t8.Counter.create(0, p.load_empty_bars.shape[0])
    acc_state = t8.Counter.create(1, p.acc_empty_bars.shape[0])
    scheduler = p.SchedulerImpl.initialize(p.c_desc.shape[0], p.c_desc.shape[1], BLOCK_M, BLOCK_N)
    for _ in range(scheduler.get_num_tiles()):
        # We expect the load of C to take longer than the previous epilogue to
        # release the accumulator, so acquire c_buf first.
        mbarrier.wait(p.c_ready_bar, c_phase)
        mbarrier.wait(p.acc_empty_bars.index(acc_state.index), acc_state.phase)
        acc_buf = p.acc_bufs.index(acc_state.index)
        tcgen05_copy(p.c_buf, acc_buf)
        # Release c_buf when the copy is complete. We don't need to wait for the
        # copy to complete because it will be implicitly pipelined with the first MMA.
        tcgen05_commit(p.c_empty_bar)
        c_phase ^= 1
        for k in range(0, K, BLOCK_K):
            # Wait for the operands to be ready.
            mbarrier.wait(p.load_ready_bars.index(load_state.index), load_state.phase)
            # Issue the MMA and release the load buffers then it completes.
            tcgen05_mma(p.a_bufs.index(load_state.index), p.b_bufs.index(load_state.index), acc_buf, use_acc=True)
            tcgen05_commit(p.load_empty_bars.index(load_state.index))
            load_state = load_state.next()
        # Release the accumulator when the last MMA is complete.
        tcgen05_commit(p.acc_ready_bars.index(acc_state.index))
        acc_state = acc_state.next()


@gluon.jit
def matmul_accumulate_epilogue_partition(p):
    BLOCK_M: gl.constexpr = p.c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = p.c_desc.block_type.shape[1]

    coalesced_2d_layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, gl.num_warps()], [1, 0])
    range_m = gl.arange(0, BLOCK_M, gl.SliceLayout(1, coalesced_2d_layout))
    range_n = gl.arange(0, BLOCK_N, gl.SliceLayout(0, coalesced_2d_layout))

    acc_state = t8.Counter.create(0, p.acc_empty_bars.shape[0])
    scheduler = p.SchedulerImpl.initialize(p.c_desc.shape[0], p.c_desc.shape[1], BLOCK_M, BLOCK_N)
    for idx in range(scheduler.get_num_tiles()):
        pid_m, pid_n = scheduler.get_tile(idx)
        off_m = pid_m * BLOCK_M
        off_n = pid_n * BLOCK_N
        # Wait for the accumulator.
        mbarrier.wait(p.acc_ready_bars.index(acc_state.index), acc_state.phase)
        acc = p.acc_bufs.index(acc_state.index).load()
        mbarrier.arrive(p.acc_empty_bars.index(acc_state.index), count=1)
        acc_state = acc_state.next()
        offs_m = (off_m + range_m)
        offs_n = (off_n + range_n)
        # This `convert_layout` is fairly expensive and it uses a lot of shared
        # memory, because the default TMEM register layout assigns contiguous
        # columns to the same thread, but the coalesced layout assigns
        # contiguous columns to different threads for efficient global writes.
        # We could subtile the store to reduce the shared memory usage.
        acc = gl.convert_layout(acc, coalesced_2d_layout)
        gl.store(p.d_ptr + offs_m[:, None] * p.d_stride_m + offs_n[None, :] * p.d_stride_n, acc)
```
**EN:** The MMA partition waits for the `C` tile and an empty accumulator buffer, uses `tcgen05_copy` to seed TMEM with `C`, and then launches `tcgen05_mma(..., use_acc=True)` over all K tiles so the result becomes `A @ B + C`. The epilogue partition waits for accumulator readiness, loads the TMEM tile, converts it to a coalesced register layout suitable for global stores, and writes the final result back to `D`.

**CN:** MMA 分区先等待 `C` tile 和空闲 accumulator buffer，就绪后用 `tcgen05_copy` 把 `C` 复制到 TMEM 作为初值，再对所有 K tile 发射 `tcgen05_mma(..., use_acc=True)`，从而实现 `A @ B + C`。Epilogue 分区等待 accumulator 就绪后，从 TMEM 读出结果 tile，把寄存器布局转换成更适合全局写回的 coalesced 布局，然后写回到 `D`。
### Lines 334-392 — Top-level warp-specialized kernel and Python launcher
```python
@gluon.jit(do_not_specialize=["d_stride_m", "d_stride_n"])
def matmul_accumulate_kernel(a_desc, b_desc, c_desc, d_ptr, d_stride_m, d_stride_n, SchedulerImpl: gl.constexpr,
                             num_buffers: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype

    a_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + a_desc.block_type.shape, a_desc.layout)
    b_bufs = gl.allocate_shared_memory(dtype, [num_buffers] + b_desc.block_type.shape, b_desc.layout)
    load_empty_bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    load_ready_bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(num_buffers):
        mbarrier.init(load_empty_bars.index(i), count=1)
        mbarrier.init(load_ready_bars.index(i), count=1)

    c_buf = gl.allocate_shared_memory(c_desc.dtype, c_desc.block_type.shape, c_desc.layout)
    c_empty_bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
    c_ready_bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(c_empty_bar, count=1)
    mbarrier.init(c_ready_bar, count=1)

    tmem_layout: gl.constexpr = TensorMemoryLayout([BLOCK_M, BLOCK_N], col_stride=1)
    acc_bufs = allocate_tensor_memory(gl.float32, [2, BLOCK_M, BLOCK_N], tmem_layout)
    acc_empty_bars = gl.allocate_shared_memory(gl.int64, [2, 1], mbarrier.MBarrierLayout())
    acc_ready_bars = gl.allocate_shared_memory(gl.int64, [2, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(2):
        mbarrier.init(acc_empty_bars.index(i), count=1)
        mbarrier.init(acc_ready_bars.index(i), count=1)

    p = PartitionArgs(a_desc, b_desc, c_desc, d_ptr, d_stride_m, d_stride_n, a_bufs, b_bufs, load_empty_bars,
                      load_ready_bars, c_buf, c_empty_bar, c_ready_bar, acc_bufs, acc_empty_bars, acc_ready_bars,
                      SchedulerImpl)
    gl.warp_specialize([
        (matmul_accumulate_epilogue_partition, (p, )),
        (matmul_accmulate_mma_partition, (p, )),
        (matmul_accumulate_load_partition, (p, )),
    ], [1, 1], [24, 24])


def matmul_accumulate(A, B, C, BLOCK_M=128, BLOCK_N=128, BLOCK_K=64, GROUP_SIZE_M=8, num_buffers=3):
    SchedulerImpl = t7.GroupedPersistentTileScheduler(GROUP_SIZE_M)
    M, N = C.shape

    dtype = getattr(gl, str(A.dtype).split('.')[1])
    acc_dtype = getattr(gl, str(C.dtype).split('.')[1])
    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], dtype)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], dtype)
    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], acc_dtype)

    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(B, [BLOCK_K, BLOCK_N], b_layout)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)
    D = torch.empty((M, N), dtype=C.dtype, device="cuda")

    num_sms = torch.cuda.get_device_properties("cuda").multi_processor_count
    num_pid = triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N)
    grid = (min(num_sms, num_pid), )
    matmul_accumulate_kernel[grid](a_desc, b_desc, c_desc, D, *D.stride(), SchedulerImpl, num_buffers)
    return D
```
**EN:** The top-level kernel allocates all double-buffered shared-memory/TMEM resources, initializes the mbarriers, constructs `PartitionArgs`, and launches load/MMA/epilogue partitions with `gl.warp_specialize`. The Python wrapper selects grouped persistent scheduling from tutorial 7, builds TMA descriptors from PyTorch tensors, sizes the persistent grid to the available SM count, and launches the fused accumulate kernel.

**CN:** 顶层 kernel 负责分配所有双缓冲 shared-memory/TMEM 资源、初始化 mbarrier、构造 `PartitionArgs`，并通过 `gl.warp_specialize` 启动 load/MMA/epilogue 三个分区。Python 包装函数则复用了教程 7 的 grouped persistent scheduler，从 PyTorch tensor 构造 TMA descriptor，并根据 SM 数量设置 persistent grid 后启动这个融合 accumulate kernel。
### Lines 395-437 — Correctness test and transition to scaled MMA
```python
@pytest.mark.parametrize("M, N, K", [(1024, 1024, 2048), (4096, 4096, 4096)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N", [(128, 128), (128, 64)])
@pytest.mark.parametrize("BLOCK_K, num_buffers", [(64, 3)])
@pytest.mark.parametrize("dtype", [torch.float16])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_matmul_accumulate(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers, dtype):
    torch.manual_seed(0)
    A = torch.randn(M, K, dtype=dtype, device="cuda")
    B = torch.randn(K, N, dtype=dtype, device="cuda")
    C = torch.randn(M, N, dtype=torch.float32, device="cuda")
    D = matmul_accumulate(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers=num_buffers)
    torch.testing.assert_close(A @ B + C, D, atol=5e-3, rtol=1e-2)


# %%
# Another important use case for `tcgen05_copy` is to asynchronously copy tensor
# scales from shared memory to tensor memory for use by `tcgen05_mma_scaled`.
# In the next tutorial, we will cover `tcgen05_mma_scaled` in more detail, but
# for now just know that the tensor scales must be supplied to `tcgen05_mma_scaled`
# via tensor memory, and the layout of the scales tensor memory must be
# `TensorMemoryScalesLayout`. If we load the scales via TMAs into shared memory,
# we can efficiently copy the scales into tensor memory with `tcgen05_copy`
# which can be implicitly pipelined with the `tcgen05_mma_scaled` instruction:
#
# ```python
# tma.async_load(a_scale_desc, ..., bar, a_scale_buf)
# tma.async_load(b_scale_desc, ..., bar, b_scale_buf)
# mbarrier.wait(bar, phase)
#
# tcgen05_copy(a_scale_buf, a_scale_tmem)
# tcgen05_copy(b_scale_buf, b_scale_tmem)
# tcgen05_mma_scaled(a_buf, b_buf, acc_tmem, a_scale_tmem, b_scale_tmem, ...)
# tcgen05_commit(mma_bar)
# ```
#
# The main takeaway from this tutorial is understanding how to use `tcgen05_copy`
# to asynchronously copy data from shared memory to tensor memory. `tcgen05_copy`
# doesn't support all layouts, but should support typical NVMMASharedLayouts.
# The instruction is useful in specific cases to copy data from shared to tensor
# memory without round-tripping the data through registers, which increases
# register pressure and is slow. It is also asynchronous and can be implicitly
# pipelined with other `tcgen05` instructions.
```
**EN:** The final test compares the fused kernel against `A @ B + C` on large matrices, confirming both the accumulate path and the synchronization logic. The closing comments connect this tutorial to the next one: `tcgen05_copy` is especially valuable for moving scale tensors into `TensorMemoryScalesLayout` before `tcgen05_mma_scaled`.

**CN:** 最后的测试把融合 kernel 的输出与 `A @ B + C` 做比较，验证 accumulate 路径和同步逻辑都正确。收尾注释把本教程自然过渡到下一篇：`tcgen05_copy` 尤其适合把 scale tensor 复制到 `TensorMemoryScalesLayout`，供后续 `tcgen05_mma_scaled` 使用。
## Key Concepts / 关键概念
- `tcgen05_copy` is an async shared→TMEM transfer primitive whose completion is tracked with `tcgen05_commit` and `mbarrier.wait` / `tcgen05_copy` 是 Blackwell 上的异步 shared→TMEM 传输指令，完成状态由 `tcgen05_commit` 与 `mbarrier.wait` 跟踪。
- Blackwell tensor-core instructions are implicitly ordered on the tensor pipe, so `tcgen05_copy` can feed later MMA operations without extra explicit synchronization / Blackwell 的 tensor-core 指令在 tensor pipe 上天然按序执行，因此 `tcgen05_copy` 可以直接为后续 MMA 供数。
- Warp specialization separates load, MMA, and epilogue work so TMA, TMEM, and global-store activity can overlap / warp specialization 将 load、MMA 和 epilogue 拆分，让 TMA、TMEM 与全局写回更容易重叠。

## Dependencies / 依赖关系
- `triton.experimental.gluon.language.nvidia.blackwell`: provides `TensorMemoryLayout`, `allocate_tensor_memory`, `tcgen05_copy`, `tcgen05_mma`, `tcgen05_commit`, `mbarrier`, and `tma` / 提供 Blackwell 专用的 TMEM、`tcgen05_*`、mbarrier 与 TMA API。
- `TensorDescriptor` from the Hopper namespace is used to wrap PyTorch tensors as TMA descriptors / Hopper 命名空间中的 `TensorDescriptor` 用来把 PyTorch tensor 包装成 TMA descriptor。
- Tutorial helpers from `07-persistence` and `08-warp-specialization` provide the persistent scheduler and counter abstractions used by the advanced kernel / `07-persistence` 与 `08-warp-specialization` 提供了高级 kernel 所需的 persistent scheduler 与 counter 抽象。
- `pytest` parameterization drives exhaustive layout testing and numerical validation / `pytest` 参数化用于覆盖布局组合并做数值验证。
