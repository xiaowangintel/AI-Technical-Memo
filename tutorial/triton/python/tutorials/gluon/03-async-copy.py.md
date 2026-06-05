# 03-async-copy.py — Code Analysis / 代码分析

## Source / 来源

- **Source file / 源文件:** `/root/xw/triton/python/tutorials/gluon/03-async-copy.py`
- **Purpose (EN):** Shows how to use NVIDIA `cp.async` from Gluon, then builds up to a software-pipelined elementwise-add kernel with multi-buffered shared memory.
- **目的（中文）:** 展示如何在 Gluon 中使用 NVIDIA `cp.async`，并逐步构建带多缓冲共享内存的软件流水化逐元素加法内核。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

````python
"""
Async Copy in Gluon
===================

Modern GPUs provide asynchronous instructions for long-running operations like
global memory reads and writes. Asynchronous operations allow overlapping memory
transactions with compute, also known as "pipelining".

Asynchronous instructions vary by GPU vendor and architecture, so this tutorial
focuses on NVIDIA GPUs. On NVIDIA GPUs, async copies transfer data between
global memory and shared memory, unlike `gl.load` and `gl.store` which
directly write to and read from the register file.
"""

import pytest
import torch
import triton
from triton.experimental import gluon
from triton.experimental.gluon import language as gl

from triton.experimental.gluon.language.nvidia.ampere import async_copy as cp


def is_ampere_or_newer():
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] >= 8


if __name__ == "__main__" and not is_ampere_or_newer():
    raise RuntimeError("This tutorial requires Ampere or newer NVIDIA GPU")
````
**EN:** The file starts by positioning async copy as a way to overlap long-latency memory movement with compute. The imports add the Ampere-specific `async_copy` namespace as `cp`, and `is_ampere_or_newer()` gates execution to hardware that actually supports these instructions.
**CN:** 文件开头先把异步拷贝定位为“用计算隐藏内存延迟”的手段。导入部分额外引入了 Ampere 专用的 `async_copy` 命名空间 `cp`，而 `is_ampere_or_newer()` 则确保只有真正支持这些指令的硬件才会执行教程。

### Lines 40-67

````python
@gluon.jit
def memcpy_1d_cpasync_kernel(in_ptr, out_ptr, xnumel, XBLOCK: gl.constexpr):
    pid = gl.program_id(0)

    layout: gl.constexpr = gl.BlockedLayout([1], [32], [4], [0])
    offsets = pid * XBLOCK + gl.arange(0, XBLOCK, layout=layout)
    mask = offsets < xnumel

    # For 1D tensor, pick a simple layout.
    smem_layout: gl.constexpr = gl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0])
    smem = gl.allocate_shared_memory(gl.float32, [XBLOCK], layout=smem_layout)

    # Issue the async copy.
    cp.async_load(smem, in_ptr + offsets, mask=mask)
    # `commit_group` puts all previously issued async copies into a group.
    cp.commit_group()

    # Wait until the number of pending groups reaches 0. Then we can retrieve
    # the data from shared memory.
    cp.wait_group(0)

    value = smem.load(layout)
    gl.store(out_ptr + offsets, value, mask=mask)


def memcpy_1d_cpasync(input, output, XBLOCK=8192, num_warps=4):
    grid = (triton.cdiv(input.numel(), XBLOCK), )
    memcpy_1d_cpasync_kernel[grid](input, output, input.numel(), XBLOCK, num_warps=num_warps)
````
**EN:** The first kernel reimplements 1D memcpy with `cp.async`. It chooses a register layout and a shared-memory layout, allocates shared memory explicitly, issues an async global-to-shared transfer, commits that transfer into a group, waits for completion, then reads from shared memory into registers before storing to global memory.
**CN:** 第一个内核用 `cp.async` 重写了一维 memcpy。它显式选择寄存器布局和共享内存布局，分配共享内存，发起异步 global-to-shared 传输，把传输提交到一个 group，等待完成，再从共享内存读入寄存器并写回全局内存。

### Lines 70-76

````python
@pytest.mark.parametrize("xnumel, XBLOCK", [(200, 128), (1000, 256)])
@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere or newer")
def test_memcpy_1d_cpasync(xnumel, XBLOCK):
    input = torch.randn(xnumel, device="cuda")
    output = torch.empty_like(input)
    memcpy_1d_cpasync(input, output, XBLOCK)
    torch.testing.assert_close(input, output, atol=0, rtol=0)
````
**EN:** This test ensures that the async path still behaves like a normal memcpy for both a partial tile and a larger tile. The `skipif` marker prevents false failures on unsupported GPUs.
**CN:** 这个测试验证了异步路径在部分 tile 和较大 tile 下都应当与普通 memcpy 行为一致。`skipif` 标记可以避免在不支持的 GPU 上产生无意义失败。

### Lines 89-125

````python
@gluon.jit
def elementwise_add_kernel(  #
        a_ptr, b_ptr, c_ptr, xnumel, ynumel,  #
        xstride_a, ystride_a, xstride_b, ystride_b, xstride_c, ystride_c,  #
        XBLOCK: gl.constexpr, YBLOCK: gl.constexpr,  #
):
    pid = gl.program_id(0)

    # Compute the offset to the row this program will process.
    layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
    xoffs = pid * XBLOCK + gl.arange(0, XBLOCK, gl.SliceLayout(1, layout))

    a_ptrs = a_ptr + xstride_a * xoffs[:, None]
    b_ptrs = b_ptr + xstride_b * xoffs[:, None]
    c_ptrs = c_ptr + xstride_c * xoffs[:, None]

    for yoff in range(0, ynumel, YBLOCK):
        # Offset to the column block.
        yoffs = yoff + gl.arange(0, YBLOCK, gl.SliceLayout(0, layout))
        mask = (xoffs < xnumel)[:, None] & (yoffs < ynumel)[None, :]

        a_val = gl.load(a_ptrs + ystride_a * yoffs[None, :], mask=mask)
        b_val = gl.load(b_ptrs + ystride_b * yoffs[None, :], mask=mask)

        c_val = a_val + b_val

        gl.store(c_ptrs + ystride_c * yoffs[None, :], c_val, mask=mask)


def elementwise_add(A, B, C, XBLOCK=32, YBLOCK=64):
    assert A.shape == B.shape == C.shape
    xnumel, ynumel = A.shape
    grid = (triton.cdiv(xnumel, XBLOCK), )
    return elementwise_add_kernel[grid](
        A, B, C, xnumel, ynumel,  #
        *A.stride(), *B.stride(), *C.stride(),  #
        XBLOCK, YBLOCK)
````
**EN:** `elementwise_add_kernel` is the synchronous baseline. Each program handles a row block, iterates across the Y dimension in tiles, loads A and B directly from global memory, computes `a+b`, and stores the result. This baseline is intentionally simple so later async optimizations are easy to compare against it.
**CN:** `elementwise_add_kernel` 是同步基线版本。每个 program 处理一个行块，沿 Y 维分 tile 迭代，直接从全局内存加载 A、B，计算 `a+b` 后写回结果。这个基线版本故意保持简洁，便于后续异步优化进行对照。

### Lines 128-135

````python
@pytest.mark.parametrize("xnumel, ynumel", [(1000, 2000)])
@pytest.mark.parametrize("XBLOCK, YBLOCK", [(32, 32), (128, 128)])
def test_elementwise_add(xnumel, ynumel, XBLOCK, YBLOCK):
    a = torch.randn(xnumel, ynumel, device="cuda")
    b = torch.randn(xnumel, ynumel, device="cuda")
    c = torch.empty_like(a, device="cuda")
    elementwise_add(a, b, c, XBLOCK, YBLOCK)
    torch.testing.assert_close(a + b, c, atol=0, rtol=0)
````
**EN:** The test confirms that the blocked 2D indexing logic and boundary masks are correct for multiple tile shapes. That matters because the later async kernels preserve the same indexing skeleton.
**CN:** 该测试确认二维分块索引和边界 mask 在多种 tile 形状下都正确。这很重要，因为后续异步内核沿用了同一套索引骨架。

### Lines 144-190

````python
@gluon.jit
def elementwise_add_cpasync_kernel(  #
        a_ptr, b_ptr, c_ptr, xnumel, ynumel,  #
        xstride_a, ystride_a, xstride_b, ystride_b, xstride_c, ystride_c,  #
        XBLOCK: gl.constexpr, YBLOCK: gl.constexpr,  #
        smem_layout: gl.constexpr,  #
):
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
    xoffs = pid * XBLOCK + gl.arange(0, XBLOCK, gl.SliceLayout(1, layout))
    a_ptrs = a_ptr + xstride_a * xoffs[:, None]
    b_ptrs = b_ptr + xstride_b * xoffs[:, None]
    c_ptrs = c_ptr + xstride_c * xoffs[:, None]

    # New: declare shared memory for the A tile and B tile.
    dtype: gl.constexpr = a_ptr.dtype.element_ty
    a_smem = gl.allocate_shared_memory(dtype, [XBLOCK, YBLOCK], layout=smem_layout)
    b_smem = gl.allocate_shared_memory(dtype, [XBLOCK, YBLOCK], layout=smem_layout)

    for yoff in range(0, ynumel, YBLOCK):
        yoffs = yoff + gl.arange(0, YBLOCK, gl.SliceLayout(0, layout))
        mask = (xoffs < xnumel)[:, None] & (yoffs < ynumel)[None, :]

        # Issue loads for both A and B tiles.
        cp.async_load(a_smem, a_ptrs + ystride_a * yoffs[None, :], mask=mask)
        cp.async_load(b_smem, b_ptrs + ystride_b * yoffs[None, :], mask=mask)
        # Commit both loads to the same group.
        cp.commit_group()
        # Wait until both loads are complete!
        cp.wait_group(0)

        a_val = a_smem.load(layout)
        b_val = b_smem.load(layout)

        c_val = a_val + b_val

        gl.store(c_ptrs + ystride_c * yoffs[None, :], c_val, mask=mask)


def elementwise_add_cpasync(A, B, C, smem_layout, XBLOCK=32, YBLOCK=64):
    assert A.shape == B.shape == C.shape
    xnumel, ynumel = A.shape
    grid = (triton.cdiv(xnumel, XBLOCK), )
    return elementwise_add_cpasync_kernel[grid](
        A, B, C, xnumel, ynumel,  #
        *A.stride(), *B.stride(), *C.stride(),  #
        XBLOCK, YBLOCK, smem_layout)
````
**EN:** The next kernel ports the same elementwise add to async copies without yet overlapping stages. A and B tiles are copied into shared memory, both copies are committed as one group, the kernel waits, then loads from shared memory and computes. Even without pipelining, this changes the memory path and can improve throughput.
**CN:** 接下来的内核把同样的逐元素加法迁移到异步拷贝路径，但暂时还没有让阶段重叠。A 和 B tile 先被拷到共享内存，两次拷贝作为同一个 group 提交，等待完成后再从共享内存加载并计算。即使没有流水化，这也改变了内存通路，因此仍可能提升吞吐。

### Lines 193-224

````python
@pytest.mark.parametrize("xnumel, ynumel", [(1000, 2000)])
@pytest.mark.parametrize("XBLOCK, YBLOCK", [(32, 32), (128, 128)])
@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere or newer")
def test_elementwise_add_cpasync(xnumel, ynumel, XBLOCK, YBLOCK):
    a = torch.randn(xnumel, ynumel, device="cuda")
    b = torch.randn(xnumel, ynumel, device="cuda")
    c = torch.empty_like(a, device="cuda")
    smem_layout = gl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])
    elementwise_add_cpasync(a, b, c, smem_layout, XBLOCK, YBLOCK)
    torch.testing.assert_close(a + b, c, atol=0, rtol=0)


def get_throughput(ms, C):
    # Because this kernel is memory-bound, we will measure bandwidth.
    tbytes = (3 * C.numel() * C.element_size() >> 30) / 1024
    return tbytes / (ms * 1e-3)


if __name__ == "__main__":
    print("Benchmarking elementwise_add")
    print("============================")
    xnumel, ynumel = 32 * 1024, 32 * 1024
    A = torch.randn(xnumel, ynumel, device="cuda")
    B = torch.randn(xnumel, ynumel, device="cuda")
    C = torch.empty_like(A, device="cuda")

    ms = triton.testing.do_bench(lambda: elementwise_add(A, B, C))
    print(f"elementwise_add: {get_throughput(ms, C):.2f} TB/s")

    smem_layout = gl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])
    ms = triton.testing.do_bench(lambda: elementwise_add_cpasync(A, B, C, smem_layout))
    print(f"elementwise_add_cpasync: {get_throughput(ms, C):.2f} TB/s")
````
**EN:** After testing the cp.async variant, the script benchmarks the baseline and async-copy versions and converts runtime into TB/s. The throughput helper reflects the kernel’s memory-bound nature: total bytes are two inputs plus one output.
**CN:** 在测试 cp.async 版本之后，脚本对基线与异步拷贝版本进行基准测试，并把运行时间换算成 TB/s。吞吐量辅助函数体现了该内核的带宽受限本质：总字节量来自两个输入和一个输出。

### Lines 226-255

````python
# %%
# ```
# elementwise_add: 1.48 TB/s
# elementwise_add_cpasync: 3.97 TB/s
# ```
#
# Surprisingly, the cpasync version is already significantly faster. We picked
# a non-swizzled shared memory layout. Shared memory is organized such that
# consecutive 32-bit elements are stored in separate banks, up to 32 banks. On
# newer GPUs, banks are dual-ported, allowing them to service two 32-bit
# requests per cycle per warp. Any more than that causes the bank to serialize
# the shared memory accesses.
#
# Our register layout maps 32 threads per warp to consecutive 32-bit elements,
# meaning even without swizzling, the shared memory load will not have bank
# conflicts. In other cases, like with 16-bit or 8-bit elements, swizzling and
# vector length is more important to reduce bank conflicts.

# %%
# Software pipelining is an optimization technique for hiding the latencies of
# operations that execute asynchronously with respect to each other. If we
# prefetch the loads of the next operands before the current add, we can overlap
# it with the add and store. This requires multi-buffering shared memory, so it
# can be used by both the load and the add at the same time.
#
# Based on the relative latencies of the operations, we can determine the
# "pipeline depth". This is the number of prefetched loads in-flight. For
# example, if a load takes 3 times as long as the add, we should pipeline with
# depth 3 so each load has time to complete before the operands are needed.
````
**EN:** The commentary explains why the non-pipelined cp.async kernel is already faster: shared-memory traffic can be better structured, and bank conflicts depend on both element type and layout. The section then motivates software pipelining as a way to prefetch future tiles while current arithmetic is executing.
**CN:** 注释解释了为什么即使未流水化，cp.async 版本也可能更快：共享内存访问路径更可控，而且 bank conflict 取决于元素类型与布局。随后该部分自然引出软件流水化——在当前算术执行时预取未来 tile。

### Lines 257-281

````python
@gluon.jit
def issue_loads(copy_idx, a_smem, b_smem, a_ptrs, ystride_a, b_ptrs, xmask, ynumel, y_idx, ystride_b,
                YBLOCK: gl.constexpr, num_buffers: gl.constexpr):
    # Masking the loads by yoffs < ynumel will handle the case where there
    # are fewer blocks to copy than `num_buffers-1`.
    yoffs = copy_idx * YBLOCK + y_idx
    mask = xmask & (yoffs < ynumel)[None, :]
    cp.async_load(a_smem.index(copy_idx % num_buffers),  #
                  a_ptrs + ystride_a * yoffs[None, :], mask)
    cp.async_load(b_smem.index(copy_idx % num_buffers),  #
                  b_ptrs + ystride_b * yoffs[None, :], mask)
    cp.commit_group()
    return copy_idx + 1


@gluon.jit
def perform_add(read_idx, a_smem, b_smem, c_ptrs, ynumel, ystride_c, y_idx, xmask, YBLOCK: gl.constexpr,
                num_buffers: gl.constexpr, layout: gl.constexpr):
    a_val = a_smem.index(read_idx % num_buffers).load(layout)
    b_val = b_smem.index(read_idx % num_buffers).load(layout)
    c_val = a_val + b_val
    yoffs = read_idx * YBLOCK + y_idx
    mask = xmask & (yoffs < ynumel)[None, :]
    gl.store(c_ptrs + ystride_c * yoffs[None, :], c_val, mask=mask)
    return read_idx + 1
````
**EN:** `issue_loads` and `perform_add` are small Gluon helper kernels that factor the pipeline into “copy stage” and “compute/store stage”. They rotate through shared-memory buffers with modulo indexing, which makes the later pipelined loop easier to read and reason about.
**CN:** `issue_loads` 和 `perform_add` 是两个小型的 Gluon 辅助内核，把流水线拆成“拷贝阶段”和“计算/存储阶段”。它们通过模运算在共享内存缓冲区间轮转，使后面的流水化循环更易读、更容易推理。

### Lines 284-343

````python
@gluon.jit
def elementwise_add_pipelined_kernel(  #
        a_ptr, b_ptr, c_ptr, xnumel, ynumel,  #
        xstride_a, ystride_a, xstride_b, ystride_b, xstride_c, ystride_c,  #
        XBLOCK: gl.constexpr, YBLOCK: gl.constexpr,  #
        smem_layout: gl.constexpr, num_buffers: gl.constexpr,  #
):
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
    xoffs = pid * XBLOCK + gl.arange(0, XBLOCK, gl.SliceLayout(1, layout))
    a_ptrs = a_ptr + xstride_a * xoffs[:, None]
    b_ptrs = b_ptr + xstride_b * xoffs[:, None]
    c_ptrs = c_ptr + xstride_c * xoffs[:, None]

    y_idx = gl.arange(0, YBLOCK, gl.SliceLayout(0, layout))
    xmask = (xoffs < xnumel)[:, None]

    # New: declare multi-buffered shared memory by adding a pipelining dimension
    # to the descriptors.
    dtype: gl.constexpr = a_ptr.dtype.element_ty
    a_smem = gl.allocate_shared_memory(dtype, [num_buffers, XBLOCK, YBLOCK], layout=smem_layout)
    b_smem = gl.allocate_shared_memory(dtype, [num_buffers, XBLOCK, YBLOCK], layout=smem_layout)
    copy_idx = 0
    read_idx = 0

    # Peel the `num_buffers-1` iterations from the inner loop to prefetch the
    # first set of copies, filling our pipeline.
    for _ in gl.static_range(num_buffers - 1):
        copy_idx = issue_loads(copy_idx, a_smem, b_smem, a_ptrs, ystride_a, b_ptrs, xmask, ynumel, y_idx, ystride_b,
                               YBLOCK, num_buffers)

    # Inner loop iterations with overlapped copies and compute. This is the
    # steady state of the pipeline.
    for _ in range(gl.cdiv(ynumel, YBLOCK) - (num_buffers - 1)):
        # Issue the overlapped copy.
        copy_idx = issue_loads(copy_idx, a_smem, b_smem, a_ptrs, ystride_a, b_ptrs, xmask, ynumel, y_idx, ystride_b,
                               YBLOCK, num_buffers)

        # Wait for `num_buffers-1` copies to complete, which is the last issued
        # copy. We can process that buffer.
        cp.wait_group(num_buffers - 1)
        read_idx = perform_add(read_idx, a_smem, b_smem, c_ptrs, ynumel, ystride_c, y_idx, xmask, YBLOCK, num_buffers,
                               layout)

    # Peeled iterations to drain the pipeline.
    for i in gl.static_range(num_buffers - 1):
        cp.wait_group(num_buffers - 2 - i)
        read_idx = perform_add(read_idx, a_smem, b_smem, c_ptrs, ynumel, ystride_c, y_idx, xmask, YBLOCK, num_buffers,
                               layout)


def elementwise_add_pipelined(A, B, C, XBLOCK=32, YBLOCK=64, num_buffers=2):
    assert A.shape == B.shape == C.shape
    xnumel, ynumel = A.shape
    grid = (triton.cdiv(xnumel, XBLOCK), )
    smem_layout = gl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])
    return elementwise_add_pipelined_kernel[grid](
        A, B, C, xnumel, ynumel,  #
        *A.stride(), *B.stride(), *C.stride(),  #
        XBLOCK, YBLOCK, smem_layout, num_buffers)
````
**EN:** `elementwise_add_pipelined_kernel` is the core optimization. It allocates multi-buffered shared memory, peels `num_buffers-1` iterations to fill the pipeline, runs a steady-state loop that overlaps async copies with computation, and finally drains the pipeline. `cp.wait_group(num_buffers - 1)` enforces exactly the dependency needed for the oldest in-flight tile.
**CN:** `elementwise_add_pipelined_kernel` 是整篇教程的核心优化。它分配多缓冲共享内存，先 peel 出 `num_buffers-1` 次迭代来填满流水线，再在稳态循环中重叠异步拷贝与计算，最后排空流水线。`cp.wait_group(num_buffers - 1)` 精确表达了“等待最老那一块可用”的依赖关系。

### Lines 346-355

````python
@pytest.mark.parametrize("xnumel, ynumel", [(1000, 2000), (4000, 120)])
@pytest.mark.parametrize("XBLOCK, YBLOCK", [(32, 64)])
@pytest.mark.parametrize("num_buffers", [1, 2, 3])
@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere or newer")
def test_elementwise_add_pipelined(xnumel, ynumel, XBLOCK, YBLOCK, num_buffers):
    a = torch.randn(xnumel, ynumel, device="cuda")
    b = torch.randn(xnumel, ynumel, device="cuda")
    c = torch.empty_like(a, device="cuda")
    elementwise_add_pipelined(a, b, c, XBLOCK, YBLOCK, num_buffers)
    torch.testing.assert_close(a + b, c, atol=0, rtol=0)
````
**EN:** The parametrized test runs the pipelined kernel with one, two, and three buffers, proving that the control structure remains correct across different pipeline depths and different tensor aspect ratios.
**CN:** 参数化测试以 1、2、3 个 buffer 运行流水化内核，证明这一控制结构在不同流水线深度以及不同张量长宽比下都保持正确。

### Lines 358-392

````python
if __name__ == "__main__":
    ms = triton.testing.do_bench(lambda: elementwise_add_pipelined(A, B, C, num_buffers=2))
    print(f"elementwise_add_pipelined (double buffer): {get_throughput(ms, C):.2f} TB/s")
    ms = triton.testing.do_bench(lambda: elementwise_add_pipelined(A, B, C, num_buffers=3))
    print(f"elementwise_add_pipelined (triple buffer): {get_throughput(ms, C):.2f} TB/s")

# %%
# ```
# elementwise_add_pipelined (double buffer): 4.20 TB/s
# elementwise_add_pipelined (triple buffer): 4.20 TB/s
# ```
#
# Pipelining with async copy yields a modest speedup. But notice that increasing
# the number of buffers further does not yield more performance, confirming that
# this kernel is memory-bound.
#
# One of the major issues getting in the way of more performance is register
# pressure. For each element, we need to store the 32-bit result, compute a
# 64-bit address, and the mask. With two inputs, this results in a lot of
# registers, where the maximum registers per thread is 256. This is why we used
# a small [32, 64] block size for the kernel. In the next tutorial, we will
# convert tensor descriptors and TMAs, and see how they can help reduce register
# pressure at the cost of addressing flexibility.
#
# Main takeaways:
#
# - Asynchronous instructions allow overlapping memory operations with compute.
# - Async copies enable asynchronous global memory reads, and are tracked with
#   commit groups.
# - Software pipelining is a loop optimization technique that is used to overlap
#   async operations.
# - Shared memory layouts affect performance just like tensor layouts. It is
#   important to choose a layout that minimizes bank conflicts, which is also a
#   function of the register layout.
````
**EN:** The final benchmark shows only a modest gain from adding more buffers, and the comments explain why: the kernel is still memory-bound, and register pressure limits tile size and occupancy. The closing bullets summarize the practical lessons on async copy, commit groups, pipelining, and layout-aware shared-memory performance.
**CN:** 最后的基准表明：增加更多 buffer 只带来有限收益。注释解释了原因——该内核依然是带宽受限的，而且寄存器压力限制了 tile 大小和 occupancy。收尾总结则概括了异步拷贝、commit group、流水化以及 layout 相关共享内存性能的核心经验。

## Key Concepts / 关键概念

- **`cp.async` pipeline / `cp.async` 流水线**
  - **EN:** Async copies move data from global memory to shared memory without stalling the arithmetic stage immediately.
  - **CN:** `cp.async` 可以把数据从全局内存搬到共享内存，而不会立刻阻塞算术阶段。
- **Commit groups / 提交组**
  - **EN:** Async copies are tracked in commit groups, and `wait_group` waits until only a chosen number of groups remain pending.
  - **CN:** 异步拷贝以提交组为单位跟踪，`wait_group` 用于等待未完成组的数量降到指定水平。
- **Shared-memory layout / 共享内存布局**
  - **EN:** Shared-memory bank behavior depends on both the SMEM layout and the register layout used to consume tiles.
  - **CN:** 共享内存 bank 行为同时取决于 SMEM 布局和消费这些 tile 的寄存器布局。
- **Software pipelining / 软件流水化**
  - **EN:** Multi-buffering lets one tile be copied while another tile is being computed and stored.
  - **CN:** 多缓冲允许在计算和存储当前 tile 时并行预取下一个 tile。

## Dependencies / 依赖关系

- **`pytest`**
  - **EN:** Used for correctness checks across memcpy and elementwise-add variants.
  - **CN:** 用于各类 memcpy 和逐元素加法变体的正确性验证。
- **`torch`**
  - **EN:** Provides CUDA tensors and reference results.
  - **CN:** 提供 CUDA 张量和参考结果。
- **`triton`**
  - **EN:** Supplies launch, benchmarking, and utility helpers.
  - **CN:** 提供启动、基准和通用辅助功能。
- **`triton.experimental.gluon.language.nvidia.ampere.async_copy as cp`**
  - **EN:** Exposes Ampere/Hopper async-copy primitives.
  - **CN:** 暴露 Ampere/Hopper 的异步拷贝原语。
- **Ampere or newer GPU**
  - **EN:** Required because `cp.async` is an NVIDIA async-copy feature starting on Ampere.
  - **CN:** 由于 `cp.async` 是从 Ampere 开始提供的 NVIDIA 异步拷贝特性，因此要求 Ampere 或更新架构。
