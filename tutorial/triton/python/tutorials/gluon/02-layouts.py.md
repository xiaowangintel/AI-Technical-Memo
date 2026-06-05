# 02-layouts.py — Code Analysis / 代码分析

## Source / 来源

- **Source file / 源文件:** `/root/xw/triton/python/tutorials/gluon/02-layouts.py`
- **Purpose (EN):** Explains Gluon tensor layouts, how layouts map elements to registers/lanes/warps, and how those choices affect 1D and 2D memcpy performance.
- **目的（中文）:** 讲解 Gluon 张量布局如何把元素映射到寄存器、lane 和 warp，以及这些布局选择如何影响一维和二维 memcpy 的性能。

## Line-by-Line Analysis / 逐行分析

### Lines 1-35

````python
"""
Tensor Layouts
==============

Tensors in Gluon require layouts. Layouts specify how the elements of the tensor
are distributed among the threads in a thread block. Tensors are distributed
with respect to the hierarchy of the GPU beginning with thread blocks, then
warps, then lanes, and finally individual registers in each lane.

Tensors are evenly distributed across theads, meaning that all threads own the
same number of elements. Because Triton requires that all tile dimensions are
powers of 2, this means that the number of elements per thread is a power of 2.

A layout, in general, defines a mapping stating the element owned by a given
register, lane, and warp. `BlockedLayout` is the most common kind of layout in
Gluon. A `BlockedLayout` defines how elements are organized in a "block" of the
same rank as the tensor.

Consider the following example:

```python
gl.BlockedLayout(
    size_per_thread=[2, 4],
    threads_per_warp=[16, 2],
    warps_per_cta=[2, 2],
    order=[1, 0],
)
```

We obtain the block shape by multiplying `size_per_thread`, `threads_per_warp`,
and `warps_per_cta` elementwise: [64, 16]. Within this block, the layout
describes a hierarchy of register, thread, and warp tiling over the logical
elements of the tensor. The `order` specifies the order in which the dimensions
of the tensor are tiled.
````
**EN:** The first section defines what a Gluon layout is. The tutorial uses `BlockedLayout` to formalize how tensor dimensions are tiled across registers, lanes, warps, and CTAs, and explains that block shape is derived by multiplying the tiling factors elementwise.
**CN:** 第一部分先定义什么是 Gluon layout。教程用 `BlockedLayout` 形式化描述张量维度如何在寄存器、lane、warp 和 CTA 之间切分，并说明 block shape 是由各层 tiling 因子逐维相乘得到的。

### Lines 36-83

````python
In this example, `size_per_thread=[2, 4]` indicates that within each block, each
thread owns a contiguous `2x4` subtile of the tensor, stored as registers in
that thread. `order=[1, 0]` indicates that the layout tiles the rows first
then the columns, i.e. row-major order. For a thread T, the tile looks like:

```
[[T:0, T:1, T:2, T:3],
 [T:4, T:5, T:6, T:7]]
```

When visualizing layouts, we sometimes represent which warp, lane, and register
are mapped to which tensor element. Notice that the registers increment over the
inner dimension.

If `order` was `[0, 1]` (col-major order), the tile would look like:

```
[[T:0, T:2, T:4, T:6],
 [T:1, T:3, T:5, T:7]]
```

Likewise, `threads_per_warp=[16, 2]` indicates how the tensor elements owned by
a single thread are tiled to obtain the elements owned by a single warp. For
`order=[1, 0]`, the warp tile of threads looks like:

```
[[ T0,  T1],
 [ T2,  T3],
 ...
 [T28, T29],
 [T30, T31]]
```

Note that the size of the warp tile must match the number of threads per warp,
which for NVIDIA hardware is 32. If we substitute each thread with its thread
tile, we obtain the warp tile over the elements of the tensor:

```
[[ T0:0,  T0:1,  T0:2,  T0:3,  T1:0,  T1:1,  T1:2,  T1:3],
 [ T0:4,  T0:5,  T0:6,  T0:7,  T1:4,  T1:5,  T1:6,  T1:7],
 [ T2:0,  T2:1,  T2:2,  T2:3,  T3:0,  T3:1,  T3:2,  T3:3],
 [ T2:4,  T2:5,  T2:6,  T2:7,  T3:4,  T3:5,  T3:6,  T3:7],
 ...
 [T28:0, T28:1, T28:2, T28:3, T29:0, T29:1, T29:2, T29:3],
 [T28:4, T28:5, T28:6, T28:7, T29:4, T29:5, T29:6, T29:7],
 [T30:0, T30:1, T30:2, T30:3, T31:0, T31:1, T31:2, T31:3],
 [T30:4, T30:5, T30:6, T30:7, T31:4, T31:5, T31:6, T31:7]]
```
````
**EN:** These examples unpack the meaning of `size_per_thread`, `threads_per_warp`, and `order`. By showing thread-local and warp-level ownership diagrams, the tutorial makes clear that layout is not metadata for the compiler only; it directly changes which thread owns which elements and in what register order.
**CN:** 这一段进一步拆解 `size_per_thread`、`threads_per_warp` 和 `order` 的含义。通过线程局部与 warp 级别的映射图，教程强调 layout 并不只是编译器的附属信息，而是直接决定“哪个线程持有哪些元素、以什么寄存器顺序持有”。

### Lines 85-138

````python
We can again repeat this process for `warps_per_cta=[2, 2]` to obtain a full
mapping of tensor elements within a block to all the threads in a program.

If the tensor is the same size as the block, then the elements are distributed
according to the block layout. If the tensor shape is different, we need to
either tile the block or broadcast the tensor elements. Consider a `128x128xf32`
tensor. Dividing the block shape into the tensor shape, we obtain a `[2, 8]`
tiling of the block. The block is tiled according to `order=[1, 0]` by adding
more registers to each thread:

```
[[ B0,  B1,  B2,  B3,  B4,  B5,  B6,  B7],
 [ B8,  B9, B10, B11, B12, B13, B14, B15]]
```

In each block, each thread owns 8 registers. Thus over the whole tensor, each
thread owns `8 * 16 = 128` registers. Knowing how many registers a tensor uses
is important for managing register pressure and budget in the kernel.

Consider a smaller tensor, say `32x8xf32`. The number of tiles at each level of
the block does not change, thus even though the tensor has only `32 * 8 = 256`
elements, it will be stored as `64 * 16 = 1024` physical registers in each
program. The tensor is broadcasted along each dimension to fit the block
starting with warps, then threads, then registers.

Dividing the tensor shape into the block shape, we obtain `[2, 2]`. Since this
exactly matches `warps_per_cta=[2, 2]`, this means each warp has a full copy of
the tensor, mapped to its lanes in the same way. From the perspective of the
tensor, this looks like:

```
[[  T0:0| T32:0| T64:0| T96:0, ...,   T1:3| T33:3| T65:3| T97:3],
 [  T0:4| T32:4| T64:4| T96:4, ...,   T1:7| T33:7| T65:7| T97:7],
 ...
 [ T30:0| T62:0| T94:0|T126:0, ...,  T31:3| T63:3| T95:3|T127:3]
 [ T30:4| T62:4| T94:4|T126:4, ...,  T31:7| T63:7| T95:7|T127:7]]
```

There are many different kinds of layouts in Gluon. Many of them are specialized
layouts required for specific operations, like MMA instructions utilizing tensor
cores. Some of them are used to represent the results of manipulating the shape
of tensors via `expand_dims`, `broadcast`, `reshape`, `join`, `split`, etc.
Please see TritonGPUAttrDefs.td for more information on layouts.

Blocked layouts are typically the most common form of layouts in Gluon. They are
primarily used to represent coalesced layouts for global memory accesses and to
represent certain register layouts for tensors stored in Tensor Memory on
NVIDIA Blackwell GPUs.

Now that we have a basic understanding of blocked layouts, let's look at an
example of how layouts can affect the performance of the kernel by expanding on
the `memcpy` example from the previous tutorial. Using a `BlockedLayout`, we
will have each program load and store a whole tile rather than one scalar.
"""
````
**EN:** The tutorial then discusses tensors whose logical shape differs from the layout block shape. Larger tensors require block tiling, while smaller tensors are effectively broadcast to fill the physical distribution. This is important because it directly determines register footprint and therefore register pressure.
**CN:** 接着教程讨论逻辑形状与 layout block shape 不一致的张量。更大的张量需要用 block 平铺，而更小的张量则会被广播以填满物理分布。这一点很重要，因为它直接决定寄存器占用，也就影响寄存器压力。

### Lines 153-188

````python
def _enabled(label):
    from sys import argv
    return len(argv) == 1 or label in argv[1].split(",")


# %%
# Parameterize the kernel over the layout so we can test different layouts. Each
# program copies a block of data, but we will use the layout to distribute
# the work over all the threads.


@gluon.jit
def memcpy_1d_kernel(in_ptr, out_ptr, xnumel, XBLOCK: gl.constexpr, layout: gl.constexpr):
    pid = gl.program_id(0)
    start = pid * XBLOCK

    # The main difference between writing this kernel in Triton and Gluon is
    # we need to specify the layout of the 1D tensor. Layouts are propagated
    # forwards through type inference, so we only need to specify the layout for
    # the indices tensor.
    indices = gl.arange(0, XBLOCK, layout=layout)

    offsets = start + indices
    in_ptrs = in_ptr + offsets
    mask = offsets < xnumel

    value = gl.load(in_ptrs, mask=mask)
    out_ptrs = out_ptr + offsets
    gl.store(out_ptrs, value, mask=mask)


def memcpy_1d_impl(input, output, XBLOCK, layout, num_warps):
    xnumel = input.numel()
    grid = (triton.cdiv(xnumel, XBLOCK), )
    compiled_kernel = memcpy_1d_kernel[grid](input, output, xnumel, XBLOCK, layout, num_warps=num_warps)
    return compiled_kernel
````
**EN:** `_enabled()` lets the script selectively run expensive tutorial sections, and the first real kernel, `memcpy_1d_kernel`, parameterizes the layout explicitly. The crucial Gluon difference from Triton is visible here: `gl.arange` must carry a layout, and that layout propagates through offsets, loads, and stores.
**CN:** `_enabled()` 允许脚本按需执行较重的教程片段，而第一个实际内核 `memcpy_1d_kernel` 则显式地把 layout 作为参数。这里体现出 Gluon 与 Triton 的关键差异：`gl.arange` 需要携带 layout，且该 layout 会沿着 offset、load、store 一路传播。

### Lines 209-239

````python
def get_throughput(input, ms):
    tbytes = (2 * input.numel() * input.element_size() >> 30) / 1024
    return tbytes / (ms * 1e-3)


def bench_memcpy_impl(input, output, impl):
    compiled_kernel = impl(input, output)
    fn = lambda: impl(input, output)
    ms = triton.testing.do_bench(fn)
    return compiled_kernel, get_throughput(input, ms)


def bench_memcpy(impl):
    torch.manual_seed(0)
    xnumel = 2 << 30
    input = torch.randn(xnumel, device="cuda")
    output = torch.empty_like(input)

    return bench_memcpy_impl(input, output, impl)


@pytest.mark.parametrize("XBLOCK", [128, 256])
@pytest.mark.parametrize("xnumel", [200, 1000])
@pytest.mark.parametrize("num_warps", [4])
def test_memcpy_1d(XBLOCK, xnumel, num_warps):
    torch.manual_seed(0)
    input = torch.randn(xnumel, device="cuda")
    output = torch.empty_like(input)
    layout = gl.BlockedLayout([1], [32], [num_warps], [0])
    memcpy_1d_impl(input, output, XBLOCK, layout, num_warps=num_warps)
    torch.testing.assert_close(input, output, atol=0, rtol=0)
````
**EN:** These helpers build a reproducible throughput benchmark and a basic correctness test. `bench_memcpy_impl` compiles the kernel once, times repeated launches, and converts runtime into TB/s, making layout experiments comparable.
**CN:** 这些辅助函数搭建了可复现的吞吐量基准和基本正确性测试。`bench_memcpy_impl` 先编译内核，再对重复启动计时，并把运行时间换算成 TB/s，从而让不同 layout 的实验可以直接比较。

### Lines 246-260

````python
if __name__ == "__main__" and _enabled("R_vs_throughput"):
    print("R vs. Throughput")
    print("================")
    XBLOCK = 2048
    num_warps = 4
    kernel = partial(memcpy_1d_impl, XBLOCK=XBLOCK, num_warps=num_warps)
    compiled_kernels = []
    for i in range(0, 5):
        R = 2**i
        layout = gl.BlockedLayout([R], [32], [num_warps], [0])
        impl = partial(kernel, layout=layout)
        compiled_kernel, throughput = bench_memcpy(impl)
        compiled_kernels.append((R, compiled_kernel))
        print(f"R={R:<3} {throughput:.3f} TB/s")
    print()
````
**EN:** The first experiment sweeps the per-thread register tile size `R` for a fixed `XBLOCK`. Because 1D blocked layouts are constrained, this is effectively a study of vectorization width: larger `R` means each thread owns more contiguous elements.
**CN:** 第一个实验在固定 `XBLOCK` 下扫描每线程寄存器 tile 大小 `R`。由于一维 blocked layout 的选择空间有限，这实际上是在研究向量化宽度：`R` 越大，单个线程连续持有的元素越多。

### Lines 276-335

````python
if __name__ == "__main__" and _enabled("LDG_STG_instructions"):
    print("LDG/STG instructions")
    print("====================")
    for R, compiled_kernel in compiled_kernels:
        print(f"\nR={R}")
        print("==========")
        sass = compiled_kernel.asm["sass"]
        for line in sass.split("\n"):
            if "LDG.E" in line or "STG.E" in line:
                print(line)
    print()

# %%
# We see that the layout affects read/write vectorization and striding:
#
# | R  | width | vec_len | n_loads | stride |
# |----|-------|---------|---------|--------|
# | 1  | 32    | 32      | 1       | 0x00   |
# | 2  | 64    | 64      | 1       | 0x00   |
# | 4  | 128   | 128     | 1       | 0x00   |
# | 8  | 256   | 128     | 2       | 0x10   |
# | 16 | 512   | 128     | 4       | 0x10   |
#
# Modern NVIDIA GPUs have 128-byte cache lines, divided into 32-byte sectors.
# These sectors are the granularity at which global memory is accessed. Thus,
# the GPU attempts to minimize the number of sector accesses by "coalescing"
# contiguous accesses to the same sectors.
#
# When R=1, each `LDG.E` at the warp level reads exactly 128 contiguous bytes of
# global memory, which fits into a cache line. Note that PyTorch allocates
# tensors aligned to 256 bytes.
#
# Increasing R to 2 or 4 widens each `LDG.E` instruction but slows down the
# kernel, despite the number of 32B sector reads remaining unchanged. This can
# be due to a variety of obscure hardware factors, but if you look at the
# annotations printed to the left of the instructions, you can see one potential
# factor:
#
# ```
# 16:1:2:-:1	@!P0 LDG.E R0, desc[UR4][R8.64];
# --:-:3:-:1	@!P0 LDG.E R15, desc[UR4][R4.64];
# --:-:4:-:1	@!P0 LDG.E R17, desc[UR4][R4.64+0x200];
# ...
# 08:0:-:-:1	@!P0 STG.E desc[UR4][R6.64], R15;
# 16:0:-:-:1	@!P0 STG.E desc[UR4][R6.64+0x200], R17;
# 04:0:-:-:1	@!P0 STG.E desc[UR4][R6.64+0x400], R19;
# ```
#
# These annotations are
#
# ```
# wait_mask : read_barrier : write_barrier : yield : stall
# ```
#
# The load instructions set a `write_barrier` because they are writing to
# registers. Subsequent `STG.E` instructions have a `wait_mask` that block until
# the barrier is cleared. By issuing smaller granularity loads, the store
# instructions can start executing earlier.
#
# It is difficult to tell why R=8 is faster than R=2 and R=4 without a profiler.
````
**EN:** This section correlates throughput with generated SASS. By examining `LDG.E` and `STG.E`, the tutorial explains why layout choices alter vector width, number of memory instructions, and synchronization/barrier behavior between loads and stores at the instruction scheduling level.
**CN:** 这一部分把吞吐量与生成的 SASS 联系起来。通过查看 `LDG.E` 与 `STG.E`，教程解释了为什么 layout 会影响向量宽度、内存指令数量，以及指令调度层面上 load/store 之间的 barrier 行为。

### Lines 337-374

````python
if __name__ == "__main__" and _enabled("XBLOCK_R_vs_throughput"):
    print("(XBLOCK, R) vs. Throughput")
    print("==========================")
    num_warps = 4

    print("XBLOCK   ", end=" ")
    for i in range(0, 5):
        print(f"R={2**i:<3}", end=" ")
    print()

    for j in range(10, 15):
        XBLOCK = 2**j
        print(f"{XBLOCK:<8}", end=" ")
        kernel = partial(memcpy_1d_impl, XBLOCK=XBLOCK, num_warps=num_warps)
        for i in range(0, 5):
            R = 2**i
            layout = gl.BlockedLayout([R], [32], [num_warps], [0])
            impl = partial(kernel, layout=layout)
            compiled_kernel, throughput = bench_memcpy(impl)
            print(f"{throughput:.3f}", end=" ")
        print()
    print()

# %%
# If we run this experiment with a variety of XBLOCK, we see that R=8 is
# not always faster than R=2 and R=4.
#
# ```
# XBLOCK    R=1   R=2   R=4   R=8   R=16
# 1024     6.566 6.548 6.542 6.550 5.226
# 2048     6.572 6.474 6.474 6.504 6.218
# 4096     6.554 6.492 6.454 6.396 6.182
# 8192     6.606 6.532 6.482 6.478 6.176
# 16384    6.522 6.556 6.486 6.510 6.146
# ```
#
# From these tests, R=1 and XBLOCK=8192 give the best throughput. These
# parameters can be autotuned over a larger range if needed.
````
**EN:** Sweeping both `XBLOCK` and `R` shows that no single local observation should be overgeneralized. The takeaway is empirical: `R=1` with `XBLOCK=8192` happens to win on the tested GPU, but the right answer depends on hardware and problem size.
**CN:** 同时扫描 `XBLOCK` 和 `R` 说明：不能把某个局部观察轻易泛化。这里的结论是经验性的：在测试 GPU 上 `R=1` 与 `XBLOCK=8192` 最优，但真正最佳组合仍取决于硬件和问题规模。

### Lines 376-452

````python
# %%
# Picking the right layout for higher-dimensional tensors is a lot less
# forgiving because the tensors can be accessed in non-contiguous ways. We will
# illustrate this with a 2D memcpy.
#
# We index into a strided 2D tensor by computing 1D offsets for the rows and
# columns, multiplying them by the strides, and broadcasting and adding them
# together. The offsets will have a 2D BlockedLayout, but we need to use a
# SliceLayout for the 1D offsets.
#
# ```python
# gl.SliceLayout(dim=1, parent=layout)
# ```
#
# A slice layout is obtained from a parent layout by dropping the `dim`
# dimension. For example, consider this blocked layout
#
# ```python
# layout = gl.BlockedLayout(
#     size_per_thread=[2, 4],
#     threads_per_warp=[16, 2],
#     warps_per_cta=[2, 2],
#     order=[1, 0],
# )
# ```
#
# The tensor element mapping is:
#
# ```
# [[ T0:0,  T0:1,  T0:2,  T0:3,  T1:0,  T1:1,  T1:2,  T1:3],
#  [ T0:4,  T0:5,  T0:6,  T0:7,  T1:4,  T1:5,  T1:6,  T1:7],
#  [ T2:0,  T2:1,  T2:2,  T2:3,  T3:0,  T3:1,  T3:2,  T3:3],
#  [ T2:4,  T2:5,  T2:6,  T2:7,  T3:4,  T3:5,  T3:6,  T3:7],
#  ...
#  [T28:0, T28:1, T28:2, T28:3, T29:0, T29:1, T29:2, T29:3],
#  [T28:4, T28:5, T28:6, T28:7, T29:4, T29:5, T29:6, T29:7],
#  [T30:0, T30:1, T30:2, T30:3, T31:0, T31:1, T31:2, T31:3],
#  [T30:4, T30:5, T30:6, T30:7, T31:4, T31:5, T31:6, T31:7]]
# ```
#
# To form the slice layout along dim=1, first collapse the mappings in each row
# together:
#
# ```
# [  T0:0| T0:1| T0:2| T0:3| T1:0| T1:1| T1:2| T1:3,
#    T0:4| T0:5| T0:6| T0:7| T1:4| T1:5| T1:6| T1:7,
#    T2:0| T2:1| T2:2| T2:3| T3:0| T3:1| T3:2| T3:3,
#    T2:4| T2:5| T2:6| T2:7| T3:4| T3:5| T3:6| T3:7,
#  ...
#   T28:0|T28:1|T28:2|T28:3|T29:0|T29:1|T29:2|T29:3,
#   T28:4|T28:5|T28:6|T28:7|T29:4|T29:5|T29:6|T29:7,
#   T30:0|T30:1|T30:2|T30:3|T31:0|T31:1|T31:2|T31:3,
#   T30:4|T30:5|T30:6|T30:7|T31:4|T31:5|T31:6|T31:7]
# ```
#
# Then remove redundant register mappings within each thread:
#
# ```
# [  T0:0| T1:0,
#    T0:1| T1:1,
#    T2:0| T3:0,
#    T2:1| T3:1,
#  ...
#   T28:0|T29:0,
#   T28:1|T29:1,
#   T30:0|T31:0,
#   T30:1|T31:1]
# ```
#
# This layout would result from reducing a 2D tensor along dim=1. You can see
# that each element in the reduction result would be broadcasted to two threads.
#
# Likewise, to expand a 1D tensor to 2D, we start with the tensor in slice
# layout and perform the reverse transformation by duplicating each element of
# the 1D tensor until it fills the rows to the desired size. Because this
# happens in virtual registers, broadcasting is a zero-cost operation.
````
**EN:** Before moving to 2D tensors, the tutorial introduces `SliceLayout`. A slice layout drops one dimension from a parent layout and is used to build 1D index vectors that later expand and broadcast into 2D offsets. This is the key abstraction that makes higher-rank address generation ergonomic in Gluon.
**CN:** 在进入二维张量之前，教程引入了 `SliceLayout`。Slice layout 从父 layout 中“去掉”一个维度，用于先构造一维索引，再扩展并广播成二维 offset。这是 Gluon 在高维地址生成中非常关键的抽象。

### Lines 454-507

````python
@gluon.jit
def memcpy_2d_kernel(in_ptr, out_ptr,  #
                     xnumel, ynumel, xstride_in, ystride_in, xstride_out, ystride_out,  #
                     layout: gl.constexpr, XBLOCK: gl.constexpr, YBLOCK: gl.constexpr):
    pid_x = gl.program_id(0)
    pid_y = gl.program_id(1)

    start_x = pid_x * XBLOCK
    start_y = pid_y * YBLOCK
    # For the 1D indices, use a SliceLayout along the dimensions we will expand.
    indices_x = start_x + gl.arange(0, XBLOCK, layout=gl.SliceLayout(dim=1, parent=layout))
    indices_y = start_y + gl.arange(0, YBLOCK, layout=gl.SliceLayout(dim=0, parent=layout))

    # expand_dims along the slice dimension returns a tensor with the parent
    # layout, so this yields [XBLOCK, 1] and [1, YBLOCK] tensors with the same
    # layout which can be broadcasted together to [XBLOCK, YBLOCK].
    in_offsets = xstride_in * indices_x[:, None] + ystride_in * indices_y[None, :]
    out_offsets = xstride_out * indices_x[:, None] + ystride_out * indices_y[None, :]

    # Compute the mask the same way: select for indices along each dimension
    # that are in bounds and broadcast them together.
    mask = (indices_x[:, None] < xnumel) & (indices_y[None, :] < ynumel)

    value = gl.load(in_ptr + in_offsets, mask=mask)
    gl.store(out_ptr + out_offsets, value, mask=mask)


def memcpy_2d_impl(input, output, XBLOCK, YBLOCK, layout, num_warps):
    xnumel, ynumel = input.shape
    grid = (triton.cdiv(xnumel, XBLOCK), triton.cdiv(ynumel, YBLOCK))
    # Pass the strides of the input and output tensors into the kernel. The
    # compiler will specialize the kernel if any of the strides are 1, which is
    # common for the inner dimension of tensors.
    compiled_kernel = memcpy_2d_kernel[grid](  #
        input, output, xnumel, ynumel,  #
        *input.stride(), *output.stride(),  #
        layout, XBLOCK, YBLOCK, num_warps=num_warps)
    return compiled_kernel


@pytest.mark.parametrize("XBLOCK, YBLOCK", [(128, 256), (256, 128)])
@pytest.mark.parametrize("xnumel, ynumel", [(100, 2000), (1000, 200)])
@pytest.mark.parametrize("transposed", [False, True])
@pytest.mark.parametrize("num_warps", [4])
def test_memcpy_2d(XBLOCK, YBLOCK, xnumel, ynumel, transposed, num_warps):
    torch.manual_seed(0)
    input = torch.randn((xnumel, ynumel), device="cuda")
    output = torch.empty_like(input)
    # Transposing the tensor makes it non-contiguous along the inner dimension.
    input = input.T if transposed else input
    output = output.T if transposed else output
    layout = gl.BlockedLayout([1, 1], [1, 32], [1, num_warps], [1, 0])
    memcpy_2d_impl(input, output, XBLOCK, YBLOCK, layout, num_warps=num_warps)
    torch.testing.assert_close(input, output, atol=0, rtol=0)
````
**EN:** `memcpy_2d_kernel` computes row and column indices with slice layouts, forms broadcasted offsets and masks, then performs a tiled 2D load/store. The accompanying test also checks both contiguous and transposed tensors, making it clear that correctness and performance must be treated separately.
**CN:** `memcpy_2d_kernel` 使用 slice layout 计算行列索引，构造广播后的 offset 与 mask，然后执行分块二维 load/store。配套测试同时覆盖 contiguous 和 transposed 张量，强调“正确性”和“性能”是两个不同层面的问题。

### Lines 516-579

````python
def bench_memcpy_2d(impl, transposed=False):
    # 8 GB tensor, but spread across 2 dimensions.
    xnumel = 32 * 1024
    ynumel = 64 * 1024
    input = torch.randn((xnumel, ynumel), device="cuda")
    output = torch.empty_like(input)
    input = input.T if transposed else input
    output = output.T if transposed else output
    return bench_memcpy_impl(input, output, impl)


# %%
# Choosing XBLOCK=1 means each program will process a row vector, and we can
# pick a blocked layout that behaves the same as the R=1 layout does in 1D.

if __name__ == "__main__" and _enabled("memcpy_2d_layout"):
    print("Benchmarking 2D memcpy")
    print("======================")
    XBLOCK = 1
    YBLOCK = 2048
    layout = gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
    impl = partial(memcpy_2d_impl, XBLOCK=XBLOCK, YBLOCK=YBLOCK, layout=layout, num_warps=4)
    _, throughput = bench_memcpy_2d(impl)
    print(f"Throughput: {throughput:.3f} TB/s")

# %%
# This yields 6.260 TB/s, which is 5% slower than the 1D memcpy. There are a
# variety of reasons why, such as more complex 2D arithmetic, but let's dig
# deeper first.
#
# Our 2D memcpy kernel has another problem: the optimal layout depends on the
# layout of the tensors in global memory. Let's check the throughput when the
# input tensor is transposed:

if __name__ == "__main__" and _enabled("memcpy_2d_layout"):
    _, throughput = bench_memcpy_2d(impl, transposed=True)
    print(f"Transposed throughput: {throughput:.3f} TB/s")

# %%
# Performance craters to 0.774 TB/s. Because the inner dimension is no longer
# contiguous, we get no coalescing. Simply swapping the block sizes and
# transposing the layout restores performance:

if __name__ == "__main__" and _enabled("memcpy_2d_layout"):
    layout = gl.BlockedLayout([1, 1], [32, 1], [4, 1], [0, 1])
    impl = partial(memcpy_2d_impl, XBLOCK=2048, YBLOCK=1, layout=layout, num_warps=4)
    _, throughput = bench_memcpy_2d(impl, transposed=True)
    print(f"Fixed throughput: {throughput:.3f} TB/s")
    print()

# %%
# This yields 6.590 TB/s, slightly faster than the 1D memcpy!
#
# Between the transposed and non-transposed inputs and layouts, each program
# accesses memory in the same way. The variation in performance is due to where
# the programs get scheduled on the GPU, which affects data locality. Even
# though each program accesses unique data, there are many mechanisms in the GPU
# cache structure that favour access locality. For example, the GPU caches
# virtual address translations in TLBs, and on H100 the L2 cache is divided into
# partitions that communicate with each other.
#
# In a subsequent tutorial, we will explore implementing persistent kernels and
# how they can be used to better control scheduling, among other benefits, to
# improve performance.
````
**EN:** These benchmark blocks show that a layout tuned for contiguous access in one dimension can collapse when the tensor is transposed. Switching block shape and layout order restores coalescing, and the comments point out that scheduler locality and cache/TLB effects also influence the final numbers.
**CN:** 这些基准代码说明：对某个连续方向优化过的 layout，在张量转置后可能会彻底失效。通过交换 block 形状与 layout 顺序，可以恢复 coalescing；而注释也提醒我们，调度位置、缓存和 TLB 局部性也会影响最终结果。

### Lines 590-630

````python
if __name__ == "__main__" and _enabled("memcpy_2d_contig"):
    print("Non-contiguous memcpy")
    print("=====================")
    # 8 GB tensor.
    xnumel = 32 * 1024
    ynumel = 64 * 1024
    input = torch.randn((xnumel, ynumel), device="cuda")
    # Take a view over every other row.
    input = input[::2]
    output = torch.empty_like(input)
    assert not input.is_contiguous() and output.is_contiguous()

    # Benchmark 2D memcpy.
    layout = gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
    impl = partial(memcpy_2d_impl, XBLOCK=1, YBLOCK=2048, layout=layout, num_warps=4)
    _, throughput = bench_memcpy_impl(input, output, impl)
    print(f"2D memcpy: {throughput:.3f} TB/s")

    # Benchmark PyTorch contiguous.
    fn = lambda: input.contiguous()
    ms = triton.testing.do_bench(fn)
    throughput = get_throughput(input, ms)
    print(f"torch.Tensor.contiguous: {throughput:.3f} TB/s")

    # We can eke out even more performance by using the transposed "trick".
    layout = gl.BlockedLayout([1, 1], [32, 1], [4, 1], [0, 1])
    impl = partial(memcpy_2d_impl, XBLOCK=2048, YBLOCK=1, layout=layout, num_warps=4)
    _, throughput = bench_memcpy_impl(input.T, output.T, impl)
    print(f"2D memcpy (transposed): {throughput:.3f} TB/s")
    print()

# %%
# ```
# 2D memcpy: 6.258 TB/s
# torch.Tensor.contiguous: 2.946 TB/s
# 2D memcpy (transposed): 6.398 TB/s
# ```
#
# Our 2D memcpy provides similar performance even when the input tensor has
# an exotic layout. It's already over 2x faster than the PyTorch implementation
````
**EN:** The non-contiguous benchmark is especially instructive: Gluon’s explicit 2D copy remains fast even when the input is a strided view, and it significantly outperforms `torch.Tensor.contiguous()` for that case. This demonstrates why flexible layout control matters in real tensor pipelines.
**CN:** 非连续张量的基准尤其有启发性：即使输入是带步长的视图，Gluon 的显式二维拷贝依然很快，而且在该场景下明显快于 `torch.Tensor.contiguous()`。这说明灵活的 layout 控制在真实张量流水线里非常有价值。

### Lines 637-680

````python
if __name__ == "__main__" and _enabled("memcpy_2d_inout"):
    print("2D memcpy in/out layouts")
    print("=========================")

    # Input is contiguous along dim 1.
    input = torch.randn((32 * 1024, 32 * 1024), device="cuda")

    # Output is contiguous along dim 0.
    output = torch.empty((input.shape[1], input.shape[0]), device="cuda").T

    # order=[1, 0]
    layout = gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
    impl = partial(memcpy_2d_impl, XBLOCK=1, YBLOCK=2048, layout=layout, num_warps=4)
    _, throughput = bench_memcpy_impl(input, output, impl)
    print(f"2D memcpy (order=[1, 0]): {throughput:.3f} TB/s")

    # order=[0, 1]
    layout = gl.BlockedLayout([1, 1], [32, 1], [4, 1], [0, 1])
    impl = partial(memcpy_2d_impl, XBLOCK=2048, YBLOCK=1, layout=layout, num_warps=4)
    _, throughput = bench_memcpy_impl(input, output, impl)
    print(f"2D memcpy (order=[0, 1]): {throughput:.3f} TB/s")

# %%
# Performance is terrible regardless of which layout we pick:
#
# ```
# 2D memcpy (order=[1, 0]): 0.978 TB/s
# 2D memcpy (order=[0, 1]): 1.674 TB/s
# ```
#
# The solution is to use two layouts for `gl.load` and `gl.store`, both derived
# from the layouts of the global tensors.


def get_layout_for_gmem_access(tensor, num_warps):
    if len(tensor.shape) == 1:
        return gl.BlockedLayout([1], [32], [num_warps], [0])

    assert len(tensor.shape) == 2, "only 1D and 2D tensors are supported"
    assert 1 in tensor.stride(), "expected at least 1 contiguous dimension"
    if tensor.stride(1) == 1:
        return gl.BlockedLayout([1, 1], [1, 32], [1, num_warps], [1, 0])
    else:
        return gl.BlockedLayout([1, 1], [32, 1], [num_warps, 1], [0, 1])
````
**EN:** When input and output have opposite memory layouts, neither single layout works well for both ends. `get_layout_for_gmem_access()` therefore derives a load-friendly or store-friendly layout from tensor strides, which sets up the need for an explicit layout conversion between them.
**CN:** 当输入和输出具有相反的内存布局时，单一 layout 无法同时兼顾两端的高效访问。于是 `get_layout_for_gmem_access()` 会根据张量 stride 推导更适合 load 或 store 的 layout，并由此引出显式 layout 转换的需求。

### Lines 702-779

````python
@gluon.jit
def get_mask_and_offsets(start_x, start_y, xnumel, ynumel, xstride, ystride,  #
                         XBLOCK: gl.constexpr, YBLOCK: gl.constexpr, layout: gl.constexpr):
    indices_x = start_x + gl.arange(0, XBLOCK, layout=gl.SliceLayout(dim=1, parent=layout))
    indices_y = start_y + gl.arange(0, YBLOCK, layout=gl.SliceLayout(dim=0, parent=layout))

    mask = (indices_x[:, None] < xnumel) & (indices_y[None, :] < ynumel)
    offsets = xstride * indices_x[:, None] + ystride * indices_y[None, :]
    return mask, offsets


@gluon.jit
def memcpy_2d_inout_kernel(in_ptr, out_ptr,  #
                           xnumel, ynumel, xstride_in, ystride_in, xstride_out, ystride_out,  #
                           layout_in: gl.constexpr, layout_out: gl.constexpr,  #
                           XBLOCK: gl.constexpr, YBLOCK: gl.constexpr):
    pid_x = gl.program_id(0)
    pid_y = gl.program_id(1)

    start_x = pid_x * XBLOCK
    start_y = pid_y * YBLOCK

    # We need two sets of indices and masks for each layout. If the layouts
    # happen to be the same, the compiler will optimize away the extra code and
    # layout conversion.
    mask_in, in_offsets = get_mask_and_offsets(start_x, start_y, xnumel, ynumel, xstride_in, ystride_in,  #
                                               XBLOCK, YBLOCK, layout_in)
    mask_out, out_offsets = get_mask_and_offsets(start_x, start_y, xnumel, ynumel, xstride_out, ystride_out,  #
                                                 XBLOCK, YBLOCK, layout_out)

    value = gl.load(in_ptr + in_offsets, mask=mask_in)

    # Use `gl.convert_layout` to perform layout conversions.
    value = gl.convert_layout(value, layout_out)

    gl.store(out_ptr + out_offsets, value, mask=mask_out)


def memcpy_2d_inout(input, output, num_warps=4):
    assert input.shape == output.shape, "input and output must have the same shape"
    XBLOCK = 128
    YBLOCK = 128
    layout_in = get_layout_for_gmem_access(input, num_warps)
    layout_out = get_layout_for_gmem_access(output, num_warps)
    grid = (triton.cdiv(input.shape[0], XBLOCK), triton.cdiv(input.shape[1], YBLOCK))
    return memcpy_2d_inout_kernel[grid](  #
        input, output,  #
        input.shape[0], input.shape[1],  #
        *input.stride(), *output.stride(),  #
        layout_in, layout_out,  #
        XBLOCK, YBLOCK, num_warps=num_warps)


@pytest.mark.parametrize("xnumel, ynumel", [(300, 400)])
@pytest.mark.parametrize("transpose_in, transpose_out", [(True, False), (False, True)])
def test_memcpy_2d_inout(xnumel, ynumel, transpose_in, transpose_out):
    torch.manual_seed(0)
    if transpose_in:
        input = torch.randn((ynumel, xnumel), device="cuda").T
    else:
        input = torch.randn((xnumel, ynumel), device="cuda")
    if transpose_out:
        output = torch.empty((ynumel, xnumel), device="cuda").T
    else:
        output = torch.empty((xnumel, ynumel), device="cuda")
    memcpy_2d_inout(input, output)
    torch.testing.assert_close(input, output, atol=0, rtol=0)


if __name__ == "__main__" and _enabled("memcpy_2d_inout"):
    _, throughput = bench_memcpy_impl(input, output, memcpy_2d_inout)
    print(f"2D memcpy (in/out layouts): {throughput:.3f} TB/s")

# %%
# This yields much more reasonable performance:
#
# ```
# 2D memcpy (in/out layouts): 4.814 TB/s
````
**EN:** `get_mask_and_offsets` factors out repeated index logic, and `memcpy_2d_inout_kernel` uses separate input/output layouts plus `gl.convert_layout`. This is the central Gluon lesson of the file: paying an explicit layout-conversion cost is often better than performing globally uncoalesced memory accesses.
**CN:** `get_mask_and_offsets` 把重复的索引逻辑抽成辅助函数，而 `memcpy_2d_inout_kernel` 则使用独立的输入/输出 layout，并通过 `gl.convert_layout` 做转换。这是本文件最核心的 Gluon 观点：显式承担一次 layout 转换的代价，往往比执行全局内存非合并访问更划算。

### Lines 785-880

````python
# %%
# So far in this tutorial, we have covered block layouts, slice layouts, and
# layout conversions. We have also explored the performance implications of
# layouts. Here are other of things where layouts can affect performance:
#
# Reductions, scans, gathers, or in general any operation that may require
# communication across threads and/or warps, can be more efficient if the layout
# of the inputs is selected to reduce the amount of communication. This includes
# layout conversions themselves.
#
# Suppose that we have a `128x128xf32` tensor that we want to reduce along the
# inner dimension. If the layout is:
#
# ```python
# gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
# ```
#
# Which is a layout we might use to load the tensor from global memory, then
# every elements in a row is owned by a different thread. The compiler will
# generate butterfly shuffles to reduce within each warp, then pick a leader
# warp to reduce the remaining 4 values per row through shared memory.
#
# If instead the layout is
#
# ```python
# gl.BlockedLayout([1, 128], [32, 1], [4, 1], [0, 1])
# ```
#
# Then each thread owns exactly one row of the tensor. Thus, the reduction
# requires no inter-thread communication.
#
# Unlike global memory accesses, the compiler does a good job of generating
# efficient reductions, scans, etc. regardless of the input layout, thus it is
# typically more expensive to convert_layout to an efficient layout and then
# perform the reeduction. However, in cases where you can choose between
# multiple layouts at the same cost, keep in mind efficient reduction layouts.
#
# Reads and writes to shared memory are affected by both the shared memory
# layout and the register layout of the tensor. This is because shared memory is
# organized into banks that can only serve one address per cycle per warp. The
# compiler generates code that minimizes bank conflicts, but the number of bank
# conflicts is still affected by the layouts.

# %%
# In Gluon, there is no canonical layout representation. Multiple layouts can
# represent the same tensor element mapping. For example, the following layouts
# are equivalent:
#
# ```python
# gl.BlockedLayout([1], [32], [4], [0])
# gl.SliceLayout(1, gl.BlockedLayout([1, 1], [32, 1], [4, 1], [1, 0]))
# ```
#
# When converting between layouts you know are equivalent, or at most only
# require reordering registers within a thread (which is free), you can use
# `gl.convert_layout(x, layout, assert_trivial=True)` to ensure this.
#
# While Gluon layouts have no canonical representation, all Gluon layouts can be
# represented as linear layouts. Linear layouts are the most expressive and
# powerful layout representation in Gluon: they allow expressing zero-cost
# splits, joins, reshapes, and permutes. However, they are relatively uncommon
# and can be difficult to understand.
#
# See `include/triton/Tools/LinearLayout.h` for more details on the data
# structure, and see the associated paper https://arxiv.org/abs/2505.23819 for
# a deeper dive into linear layouts.
#
# The linear layout equivalent to the 2 layouts above is:
#
# ```python
# gl.DistributedLinearLayout(
#   reg_bases=[],
#   lane_bases=[[1], [2], [4], [8], [16]],
#   warp_bases=[[32], [64]],
#   block_bases=[],
#   shape=[128],
# )
# ```
#
# You can see that this linear layout is a 7x7 identity matrix over the bits of
# the 1D tensor element index, where we interpret the lower 5 bits as the lane
# and the upper 2 bits as the warp.
#
# Linear layouts are extremely poweful, and can be used in conjunction with
# higher dimensional tensors (e.g. 5D or 7D) and reshapes to perform coalesced
# loads and efficient transformations of data within the kernel.
#
# Main takeaways:
#
# - Gluon requires explicit layout management, and there many kinds of layouts
#   in Gluon that serve different purposes.
# - Layouts affect performance, sometimes dramatically. Layouts affect
#   performance of global memory accesses, operations that may require
#   inter-thread communication, among other things.
# - Layouts are powerful tools for writing flexible yet performant kernels.
````
**EN:** The final discussion broadens the scope beyond memcpy. It explains how layouts affect reductions, scans, shared-memory bank conflicts, trivial conversions, and even linear layouts as a more general representation. In other words, layout in Gluon is a first-class optimization language, not a one-off API detail.
**CN:** 最后的讨论把范围从 memcpy 扩展出去，说明 layout 还会影响 reduction、scan、共享内存 bank conflict、平凡转换，乃至更一般的 linear layout 表示。换句话说，在 Gluon 中 layout 是一门“一等优化语言”，而不是某个零散 API 细节。

## Key Concepts / 关键概念

- **Blocked layouts / BlockedLayout 布局**
  - **EN:** A blocked layout defines ownership hierarchically across registers, lanes, warps, and CTAs.
  - **CN:** Blocked layout 以层次化方式定义寄存器、lane、warp 与 CTA 的元素归属。
- **Slice layouts / SliceLayout 切片布局**
  - **EN:** Slice layouts derive lower-rank index layouts from a parent layout and are essential for higher-dimensional indexing.
  - **CN:** Slice layout 从父布局派生出更低秩的索引布局，是高维索引生成的关键工具。
- **Coalescing and vectorization / 合并访问与向量化**
  - **EN:** Layout directly controls memory coalescing, instruction width, and effective bandwidth.
  - **CN:** Layout 会直接影响内存合并访问、指令向量宽度以及最终有效带宽。
- **Layout conversion / 布局转换**
  - **EN:** `gl.convert_layout` lets Gluon separate load-optimal and store-optimal distributions when tensor strides disagree.
  - **CN:** `gl.convert_layout` 使 Gluon 能在输入输出 stride 不一致时分别使用最优的 load/store 分布。
- **Linear layouts / 线性布局**
  - **EN:** Linear layouts provide a more general algebraic representation behind many equivalent Gluon layouts.
  - **CN:** Linear layout 为多个等价的 Gluon 布局提供了更一般的代数表示。

## Dependencies / 依赖关系

- **`pytest`**
  - **EN:** Covers correctness for 1D, 2D, and converted-layout memcpy kernels.
  - **CN:** 覆盖 1D、2D 以及带布局转换 memcpy 内核的正确性测试。
- **`torch`**
  - **EN:** Creates contiguous, transposed, and strided CUDA tensors for experiments.
  - **CN:** 用于创建 contiguous、transposed 和带步长的 CUDA 张量进行实验。
- **`triton`**
  - **EN:** Provides launch helpers and benchmarking utilities.
  - **CN:** 提供启动辅助函数和基准测试工具。
- **`functools.partial`**
  - **EN:** Used to freeze layout/meta-parameter choices for benchmark sweeps.
  - **CN:** 用于在基准扫描中固定 layout 和元参数。
- **`triton.experimental.gluon.language as gl`**
  - **EN:** Defines layout objects, layout conversion, and memory operations.
  - **CN:** 提供布局对象、布局转换以及内存操作原语。
