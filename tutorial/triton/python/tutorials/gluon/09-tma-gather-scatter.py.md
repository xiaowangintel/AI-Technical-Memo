# 09-tma-gather-scatter.py — Code Analysis / 代码分析

## Source / 来源

- **Source file:** `/root/xw/triton/python/tutorials/gluon/09-tma-gather-scatter.py`
- **Purpose / 用途:** Demonstrates Triton Experimental Gluon native TMA gather/scatter on Blackwell GPUs, including layout constraints, standalone gather/scatter kernels, and a fused matmul pipeline with gathered input rows and scattered output rows. / 演示 Triton Experimental Gluon 在 Blackwell GPU 上的原生 TMA gather/scatter，用例包括布局约束、独立 gather/scatter 内核，以及带输入行 gather 与输出行 scatter 的融合矩阵乘流水线。

## Line-by-Line Analysis / 逐行分析

### Lines 1-63

````python
"""
Native TMA Gather and Scatter
=============================

This tutorial explains how to use the native async TMA gather and scatter
operations available on Blackwell GPUs. Native gather and scatter operations on
Blackwell GPUs are implemented in the `gl.nvidia.blackwell.tma.async_gather` and
`gl.nvidia.blackwell.tma.async_scatter` functions respectively.

TMA gather and scatter operations only support 2D tensor descriptors, where the
first dimension of the block shape must be 1. Gather accepts a 2D tensor
descriptor, a 1D tensor of row offsets, and a scalar column offset. If the block
shape of the 2D tensor descriptor is `[1, BLOCK_Y]`, gather performs the
following operation returning a 2D tensor:

```python
out = tensor_desc[x_offsets, y_offset:y_offset + BLOCK_Y]
```

Where `out.shape` is `(x_offsets.shape[0], BLOCK_Y)`. In other words, gather
loads `x_offsets.shape[0]` separately-indexed rows of size `BLOCK_Y` from the
tensor descriptor, starting at `y_offset`.

Scatter accepts a 2D tensor descriptor, a 1D tensor of row offsets, a scalar
column offset, and a 2D source tensor. If the block shape of the 2D tensor
descriptor is `[1, BLOCK_Y]`, scatter performs the following operation:

```python
tensor_desc[x_offsets, y_offset:y_offset + BLOCK_Y] = src
```

Where `src.shape` must be `(x_offsets.shape[0], BLOCK_Y)`. In other words,
scatter writes `src` to the tensor descriptor starting at `y_offset` but to
separately-indexed rows of size `BLOCK_Y`.

Like `async_copy_global_to_shared` and `async_copy_shared_to_global`,
`async_gather` and `async_scatter` access shared memory through the async
proxy, so fences need to be inserted as appropriate.
"""

import sys
import pytest
import torch
import triton
import importlib
import triton.experimental.gluon as gluon
import triton.experimental.gluon.language as gl
from triton._C.libtriton import ir, gluon_ir

from triton.experimental.gluon.nvidia.hopper import TensorDescriptor
from triton.experimental.gluon.language.nvidia.blackwell import (tma, mbarrier, fence_async_shared)


def is_blackwell():
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] == 10


if __name__ == "__main__" and not is_blackwell():
    raise RuntimeError("This tutorial requires a Blackwell NVIDIA GPU")

# Re-use utilities from the previous tutorials.
t7 = importlib.import_module("07-persistence")
````

**EN:** Introduces the tutorial, defines the gather/scatter semantics in prose, imports Triton/Gluon internals, and adds a runtime Blackwell guard. The module also imports tutorial `07-persistence` so later sections can reuse its persistent scheduler and MMA helpers.

**CN:** 本段给出教程总览与 gather/scatter 语义说明，导入 Triton/Gluon 的底层组件，并用 `is_blackwell()` 做 Blackwell GPU 运行时检查。同时导入教程 `07-persistence`，以复用后续矩阵乘内核需要的持久化调度器和 MMA 辅助逻辑。

### Lines 65-149

````python
# %%
# `async_gather` and `async_scatter` impose constraints on the layout of the 1D
# row offsets tensor.
#
# Specifically, suppose the row offset tensor is divided into chunks of 4
# consecutive elements, then the layout must map each chunk to consecutive
# registers in the same thread. In addition, the chunks must be broadcasted
# across all threads in the same warp, i.e. all threads in the same warp must
# contain the same data.
#
# These constraints arise from the underlying `gather4` and `scatter4` PTX
# instructions used by `async_gather` and `async_scatter`. Each is a warp-level
# instruction that loads to or stores from 4 consecutive rows in shared memory.
#
# For example, the following layout is always valid for any row offsets tensor:
#
# ```python
# gl.SliceLayout(
#     dim=0,
#     parent=gl.BlockedLayout(
#         size_per_thread=[1, 4],
#         threads_per_warp=[num_threads_per_warp, 1],
#         warps_per_cta=[1, num_warps],
#         order=[1, 0],
#     ),
# )
# ```
#
# Recall from `02-layouts` that the parent `BlockedLayout` specified above will
# tile the dim=1 into chunks of 4 consecutive elements mapped to 4 consecutive
# registers in the same thread, and then tile dim=1 along all the warps. dim=0
# is only tiled across the threads in a warp, but when we take the `SliceLayout`
# along dim=0, all threads in a warp will map to the same 4 consecutive
# elements.
#
# Note that transposing the blocked layout and slicing along dim=1 yields an
# identical layout:
#
# ```python
# gl.SliceLayout(
#     dim=1,
#     parent=gl.BlockedLayout(
#         size_per_thread=[4, 1],
#         threads_per_warp=[1, num_threads_per_warp],
#         warps_per_cta=[num_warps, 1],
#         order=[0, 1],
#     ),
# )
# ```
#
# These are not the only valid layouts for the row offsets tensor. For example,
# given a row offset tensor with the shape `(BLOCK_X)`, a valid layout could be:
#
# ```python
# gl.BlockedLayout(
#     size_per_thread=[BLOCK_X]
#     threads_per_warp=[num_threads_per_warp],
#     warps_per_cta=[num_warps],
#     order=[0],
# )
# ```
#
# This layout is valid because all elements are mapped consecutively to the
# registers in all of the threads, but it is less efficient; because all warps
# have the same data, the compiler will pick only warp 0 to emit all the
# instructions. For example, if `BLOCK_X=256`, warp 0 will execute
# `256 // 4 = 64` gather4 instructions while the rest of the warps do nothing,
# whereas the sliced layouts above will spread the work across all warps,
# resulting in `256 // 4 // 4 = 16` gather4 instructions per warp, assuming
# there are 4 warps.
#
# In general, a layout is valid if its linear layout representation satisfies:
# - The first 2 register bases must be [1] and [2]
# - The lane bases must all be [0]

# %%
# Let's write a tool to convert any layout to a linear layout to help illustrate
# this concept.


def to_linear_layout(layout, shape):
    context = ir.context()
    ir.load_dialects(context)
    builder = gluon_ir.GluonOpBuilder(context)
    return builder.to_linear_layout(layout._to_ir(builder), shape)
````

**EN:** Explains the row-offset layout contract required by Blackwell `gather4/scatter4`: groups of 4 consecutive row indices must live in consecutive registers of one thread and be broadcast across a warp. `to_linear_layout()` converts a Gluon layout to the compiler's linear-layout form so these invariants can be inspected directly.

**CN:** 解释 Blackwell `gather4/scatter4` 对行索引布局的要求：连续 4 个行号必须落在同一线程的连续寄存器中，并在整个 warp 内广播。`to_linear_layout()` 用于把 Gluon 布局转换成编译器内部的线性布局表示，便于直接检查这些约束是否满足。

### Lines 152-212

```python
if __name__ == "__main__":
    num_threads_per_warp = 32
    num_warps = 4
    BLOCK_X = 256

    layout = gl.SliceLayout(
        dim=0,
        parent=gl.BlockedLayout(
            size_per_thread=[1, 4],
            threads_per_warp=[num_threads_per_warp, 1],
            warps_per_cta=[1, num_warps],
            order=[1, 0],
        ),
    )
    # DistributedLinearLayout(
    #     reg_bases=[[1], [2], [16], [32], [64], [128]],
    #     lane_bases=[[0], [0], [0], [0], [0]],
    #     warp_bases=[[4], [8]],
    #     block_bases=[],
    #     shape=[256]
    # )
    print(to_linear_layout(layout, [256]))

    layout = gl.BlockedLayout(
        size_per_thread=[BLOCK_X],
        threads_per_warp=[num_threads_per_warp],
        warps_per_cta=[num_warps],
        order=[0],
    )
    # DistributedLinearLayout(
    #     reg_bases=[[1], [2], [4], [8], [16], [32], [64], [128]],
    #     lane_bases=[[0], [0], [0], [0], [0]],
    #     warp_bases=[[0], [0]],
    #     block_bases=[],
    #     shape=[256]
    # )
    print(to_linear_layout(layout, [256]))

    # Notice how in the two layouts above, the first two register bases are
    # indeed [1] and [2], and all lane bases are [0]. The different is the
    # second layout's warp bases are all [0], which leads to inefficient code
    # generation for `async_gather` and `async_scatter`.

    # Here is an example of an invalid layout:
    layout = gl.BlockedLayout(
        size_per_thread=[4],
        threads_per_warp=[num_threads_per_warp],
        warps_per_cta=[num_warps],
        order=[0],
    )
    # DistributedLinearLayout(
    #     reg_bases=[[1], [2]],
    #     lane_bases=[[4], [8], [16], [32], [64]],
    #     warp_bases=[[128], [0]],
    #     block_bases=[],
    #     shape=[256]
    # )
    print(to_linear_layout(layout, [256]))

    # This layout is invalid because the lane bases are not all [0].

```

**EN:** Runs concrete layout examples. The sliced blocked layout is valid and distributes work across warps; the 1D blocked layout is still legal but inefficient because warp bases are all zero, so one warp issues the gather/scatter instructions; the final layout is invalid because its lane bases are non-zero.

**CN:** 这里给出具体布局示例：切片后的 blocked 布局既合法又能把工作分摊到多个 warp；纯 1D blocked 布局虽然也合法，但由于 warp bases 全为 0，通常会退化成由单个 warp 发出 gather/scatter 指令；最后一个布局则非法，因为 lane bases 不是 0。

### Lines 213-240

```python
# %%
# Let's demonstrate how to use `async_gather` and `async_scatter` by writing
# simple kernels. Note that both `async_gather` and `async_scatter` have several
# additional constraints. As we already mentioned, the tensor descriptor must be
# 2D with a block shape in the form of `[1, BLOCK_Y]`. Additionally:
#
# - The row offset tensor must have at least 8 elements. I.e. at least 8 rows
#   must be loaded by async gather or stored by async scatter.
#
# - There is a minimum number of columns based on the dtype. Specifically,
#   `BLOCK_Y >= (32 // tensor_desc.dtype.primitive_bitwidth) * 8`. For example,
#   a `float16` tensor descriptor must have `BLOCK_Y >= 16`.
#
# - The `y_offset` must be aligned to 16 bytes. I.e.
#   `y_offset % (16 // (tensor_desc.dtype.primitive_bitwidth // 8)) == 0`.
#   For example, for `float16`, `y_offset` must be a multiple of 8. This is checked
#   at runtime by the hardware, and if `y_offset` is not aligned to 16 bytes, the
#   CUDA driver will emit an illegal instruction error.
#
# - Elements of `x_offsets` may be out-of-bounds, in which case the loaded rows of
#   `async_gather` will be all zeros, and stored rows in `async_scatter` will be ignored.
#
# - `y_offset` can be out-of-bounds. Row elements in `y_offset:y_offset + BLOCK_Y` that
#   are out-of-bounds will be loaded as zeros by `async_gather` and ignored when stored by `async_scatter`.
#
# - `x_offsets` elements and `y_offset` may only be negative for `async_gather`. If `async_scatter`
#   receives negative row of column offsets, the CUDA driver will emit an illegal instruction error.

```

**EN:** Summarizes the hardware/runtime constraints for native TMA gather/scatter: a 2D descriptor with block shape `[1, BLOCK_Y]`, at least 8 rows, a dtype-dependent minimum `BLOCK_Y`, 16-byte alignment for `y_offset`, zero-fill/ignore behavior for out-of-bounds accesses, and no negative offsets for scatter.

**CN:** 总结原生 TMA gather/scatter 的硬件与运行时约束：张量描述符必须是块形状为 `[1, BLOCK_Y]` 的二维描述符；行数至少为 8；`BLOCK_Y` 受数据类型位宽约束；`y_offset` 必须按 16 字节对齐；越界读取会补零、越界写入会被忽略；scatter 不能使用负偏移。

### Lines 242-289

```python
# The kernel computes `out = tensor_desc[x_offsets, y_offset:y_offset + BLOCK_Y]`.
@gluon.jit
def async_gather_kernel(out_ptr, out_stride_x, out_stride_y, tensor_desc, x_offsets_ptr, y_offset,
                        BLOCK_X: gl.constexpr):
    BLOCK_Y: gl.constexpr = tensor_desc.block_type.shape[1]

    # Load the offsets using a coalesced layout for efficient load vectorization.
    coalesced_1d_layout: gl.constexpr = gl.BlockedLayout([1], [32], [gl.num_warps()], [0])
    x_offsets = gl.load(x_offsets_ptr + gl.arange(0, BLOCK_X, coalesced_1d_layout))

    # Convert the offsets layout to a slice layout that satisfies the constraints for `async_gather`.
    offsets_layout: gl.constexpr = gl.SliceLayout(0, gl.BlockedLayout([1, 4], [32, 1], [1, gl.num_warps()], [1, 0]))
    x_offsets = gl.convert_layout(x_offsets, offsets_layout)

    # `async_gather` loads the rows from a tensor descriptor and writes them into shared memory.
    # The layout of the shared memory descriptor must match the shared memory layout of the tensor descriptor.
    smem_dest = gl.allocate_shared_memory(tensor_desc.dtype, [BLOCK_X, BLOCK_Y], tensor_desc.layout)

    # `async_gather` is an asynchronous operation that uses an mbarrier to track its completion.
    bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)
    # Invoke `mbarrier.expect` on the mbarrier with the number of bytes to be loaded.
    mbarrier.expect(bar, BLOCK_X * tensor_desc.block_type.nbytes)

    # Issue the async gather and wait.
    tma.async_gather(tensor_desc, x_offsets, y_offset, barrier=bar, result=smem_dest)
    mbarrier.wait(bar, phase=0)
    mbarrier.invalidate(bar)

    # Write the result using a coalesced layout.
    coalesced_2d_layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, gl.num_warps()], [1, 0])
    out = smem_dest.load(coalesced_2d_layout)

    indices_x = gl.arange(0, BLOCK_X, gl.SliceLayout(1, coalesced_2d_layout))[:, None] * out_stride_x
    indices_y = gl.arange(0, BLOCK_Y, gl.SliceLayout(0, coalesced_2d_layout))[None, :] * out_stride_y
    gl.store(out_ptr + indices_x + indices_y, out)


def async_gather(input, x_offsets, y_offset, BLOCK_X, BLOCK_Y):
    gl_dtype = getattr(gl, str(input.dtype).split('.')[1])
    # When picking the shared memory layout, we use the dimensions of the shared
    # memory descriptor, which will be [BLOCK_X, BLOCK_Y]. But the block shape of the
    # tensor descriptor must still be [1, BLOCK_Y] to be used with async gather.
    layout = gl.NVMMASharedLayout.get_default_for([BLOCK_X, BLOCK_Y], gl_dtype)
    tensor_desc = TensorDescriptor.from_tensor(input, [1, BLOCK_Y], layout)
    out = torch.empty((BLOCK_X, BLOCK_Y), dtype=input.dtype, device="cuda")
    async_gather_kernel[(1, )](out, *out.stride(), tensor_desc, x_offsets, y_offset, BLOCK_X)
    return out
```

**EN:** Implements a minimal async gather kernel. The kernel loads row offsets in a coalesced layout, converts them to a legal slice layout for TMA gather, allocates shared-memory output plus an mbarrier, issues `tma.async_gather`, waits for completion, then stores the gathered tile back to global memory in a coalesced 2D layout. The Python wrapper builds a `TensorDescriptor` with block shape `[1, BLOCK_Y]` and a matching shared-memory layout.

**CN:** 实现最小化的 async gather 内核：先以合并访问友好的布局读取行索引，再转换成满足 TMA gather 要求的 slice 布局；随后分配共享内存输出缓冲区和 mbarrier，发出 `tma.async_gather`，等待完成后再以二维合并布局写回全局内存。Python 包装函数负责创建块形状为 `[1, BLOCK_Y]` 的 `TensorDescriptor`，并选取匹配的共享内存布局。

### Lines 292-362

````python
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float32])
@pytest.mark.parametrize("BLOCK_X", [8, 128])
@pytest.mark.parametrize("BLOCK_Y", [16, 128])
@pytest.mark.parametrize("y_offset", [-16, 0, 48, 1000])
@pytest.mark.parametrize("X_MAX, Y_MAX", [(1024, 1024)])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_async_gather(BLOCK_X, BLOCK_Y, y_offset, dtype, X_MAX, Y_MAX, fresh_knobs):
    triton.knobs.compilation.instrumentation_mode = "iisan"
    torch.manual_seed(0)

    input = torch.randn((X_MAX, Y_MAX), dtype=dtype, device="cuda")
    # Span row offsets from negative to out-of-bounds to test the masked load behavior.
    x_offsets = torch.linspace(-X_MAX, 2 * X_MAX, BLOCK_X, dtype=torch.int32, device="cuda")
    # Randomly shuffle the row offsets.
    x_offsets = x_offsets[torch.randperm(BLOCK_X, device="cuda")]

    out = async_gather(input, x_offsets, y_offset, BLOCK_X, BLOCK_Y)

    # Mask out-of-bounds and negative row offsets.
    x_offsets = torch.where(x_offsets >= X_MAX, -1, x_offsets)
    mask = (x_offsets >= 0).unsqueeze(1)

    # Mask out-of-bounds and negative column offsets by padding with zeros.
    y_lo, y_hi = max(0, y_offset), min(y_offset + BLOCK_Y, Y_MAX)
    ref = input[x_offsets, y_lo:y_hi] * mask
    lo_zeros = torch.zeros(BLOCK_X, y_lo - y_offset, dtype=dtype, device="cuda")
    hi_zeros = torch.zeros(BLOCK_X, y_offset + BLOCK_Y - y_hi, dtype=dtype, device="cuda")
    ref = torch.cat((lo_zeros, ref, hi_zeros), dim=1)

    torch.testing.assert_close(out, ref, atol=0, rtol=0)


# %%
# The CUDA driver will emit an illegal instruction error if `y_offset` is not
# aligned to 16 bytes for both `async_gather` and `async_scatter`, or if negative
# row or column offsets are used for `async_scatter`.

if __name__ == "__main__":
    # Note that any illegal instruction errors will corrupt the CUDA context in current Python
    # process, which prevents executing any other code. Guard each of these examples with a
    # flag so that only 1 is executed at a time.
    if len(sys.argv) > 1 and sys.argv[1] == "test_illegal_gather":
        try:
            # y_offset=2 is not 16-byte aligned for bfloat16
            test_async_gather(BLOCK_X=128, BLOCK_Y=128, y_offset=2, dtype=torch.bfloat16)
        except RuntimeError as e:
            assert "an illegal instruction was encountered" in str(e)
            raise

# %%
# Illegal instruction errors can be frustrating to debug. They typically occur
# because an executed instruction does not match some runtime invariants. To
# figure out which instruction is causing the error, you can run the program
# inside the debugger `cuda-gdb`. For example, if we run
#
# ```bash
# cuda-gdb --args python python/tutorials/gluon/09-tma-gather-scatter.py test_illegal_gather
# ```
#
# Send `r` to run the program, and the debugger will break on the instruction
# that triggered the illegal instruction error:
#
# ```
# CUDA Exception: Warp Illegal Instruction
# The exception was triggered at PC 0x628fbe590  async_gather_kernel  (09-tma-gather-scatter.py:245)
#
# Thread 1 "python" received signal CUDA_EXCEPTION_4, Warp Illegal Instruction.
# [Switching focus to CUDA kernel 0, grid 9, block (0,0,0), thread (96,0,0), device 0, sm 148, warp 0, lane 0]
# 0x0000000628fbe700 in async_gather_kernel<<<(1,1,1),(128,1,1)>>> () at /root/code/triton/python/tutorials/gluon/09-tma-gather-scatter.py:245
# 245         tma.async_gather(tensor_desc, x_offsets, y_offset, barrier=bar, result=smem_dest)
# ```
````

**EN:** Tests gather across dtypes, tile sizes, aligned and out-of-range column offsets, and randomized row indices. The reference path mirrors hardware semantics by masking invalid rows and padding invalid columns with zeros. The `__main__` example deliberately triggers an illegal-instruction case (misaligned `y_offset`) and documents how `cuda-gdb` pinpoints the faulting `tma.async_gather` instruction.

**CN:** 这一段对 gather 做参数化测试，覆盖不同数据类型、块大小、对齐/越界列偏移以及随机行索引。参考实现严格模拟硬件行为：非法行索引置零，越界列区间通过零填充补齐。`__main__` 中还刻意构造了一个非法指令场景（`y_offset` 未按要求对齐），并说明如何借助 `cuda-gdb` 定位到触发异常的 `tma.async_gather` 指令。

### Lines 365-406

```python
# This kernel computes `tensor_desc[x_offsets, y_offset:y_offset + BLOCK_Y] = src`.


@gluon.jit
def async_scatter_kernel(tensor_desc, x_offsets_ptr, y_offset, src_ptr, src_stride_x, src_stride_y,
                         BLOCK_X: gl.constexpr):
    BLOCK_Y: gl.constexpr = tensor_desc.block_type.shape[1]

    # Load the source using a coalesced layout for efficient load vectorization.
    coalesced_2d_layout: gl.constexpr = gl.BlockedLayout([1, 1], [1, 32], [1, gl.num_warps()], [1, 0])
    indices_x = gl.arange(0, BLOCK_X, gl.SliceLayout(1, coalesced_2d_layout))[:, None] * src_stride_x
    indices_y = gl.arange(0, BLOCK_Y, gl.SliceLayout(0, coalesced_2d_layout))[None, :] * src_stride_y
    src = gl.load(src_ptr + indices_x + indices_y)

    # Load the offsets using a coalesced layout for efficient load vectorization.
    coalesced_1d_layout: gl.constexpr = gl.BlockedLayout([1], [32], [gl.num_warps()], [0])
    x_offsets = gl.load(x_offsets_ptr + gl.arange(0, BLOCK_X, coalesced_1d_layout))

    # Convert the offsets layout to a slice layout that satisfies the constraints for `async_scatter`.
    offsets_layout: gl.constexpr = gl.SliceLayout(0, gl.BlockedLayout([1, 4], [32, 1], [1, gl.num_warps()], [1, 0]))
    x_offsets = gl.convert_layout(x_offsets, offsets_layout)

    # `async_scatter` stores the rows to a tensor descriptor from shared memory.
    smem_src = gl.allocate_shared_memory(tensor_desc.dtype, [BLOCK_X, BLOCK_Y], tensor_desc.layout)
    smem_src.store(src)
    # An async fence is required between the store to shared memory and the async scatter.
    # Recall from `04-tma` that a fence is needed when using different proxies to access shared
    # memory (generic proxy for the store, and async proxy for the `async_scatter`).
    fence_async_shared()
    tma.async_scatter(tensor_desc, x_offsets, y_offset, smem_src)
    # Wait for the completion of the async scatter using `store_wait`.
    tma.store_wait(0)


def async_scatter(input, x_offsets, y_offset, src, BLOCK_X, BLOCK_Y):
    gl_dtype = getattr(gl, str(input.dtype).split('.')[1])
    # When picking the shared memory layout, we use the dimensions of the shared
    # memory descriptor, which will be [BLOCK_X, BLOCK_Y]. But the block shape of the
    # tensor descriptor must still be [1, BLOCK_Y] to be used with async scatter.
    layout = gl.NVMMASharedLayout.get_default_for([BLOCK_X, BLOCK_Y], gl_dtype)
    tensor_desc = TensorDescriptor.from_tensor(input, [1, BLOCK_Y], layout)
    async_scatter_kernel[(1, )](tensor_desc, x_offsets, y_offset, src, *src.stride(), BLOCK_X)
```

**EN:** Implements async scatter as the mirror image of gather. It loads the source tile and row offsets, converts the offsets to the required layout, stages the source tile in shared memory, inserts `fence_async_shared()` because the producer store uses the generic proxy while TMA uses the async proxy, launches `tma.async_scatter`, and finishes with `tma.store_wait(0)`.

**CN:** 这里实现与 gather 对称的 async scatter：先读取源 tile 和行索引，再把索引转换成所需布局；随后把源 tile 放入共享内存，并调用 `fence_async_shared()`，因为前面的共享内存写入走的是 generic proxy，而 TMA scatter 走的是 async proxy；最后发出 `tma.async_scatter` 并用 `tma.store_wait(0)` 等待完成。

### Lines 409-439

```python
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float32])
@pytest.mark.parametrize("BLOCK_X", [8, 128])
@pytest.mark.parametrize("BLOCK_Y", [16, 128])
@pytest.mark.parametrize("y_offset", [0, 48, 1000])
@pytest.mark.parametrize("X_MAX, Y_MAX", [(1024, 1024)])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_async_scatter(BLOCK_X, BLOCK_Y, y_offset, dtype, X_MAX, Y_MAX, fresh_knobs):
    triton.knobs.compilation.instrumentation_mode = "iisan"
    torch.manual_seed(0)

    input = torch.randn((X_MAX, Y_MAX), dtype=dtype, device="cuda")
    input_ref = input.clone()

    # Span row offsets from 0 to out-of-bounds to test the masked store behavior.
    x_offsets = torch.linspace(0, 2 * X_MAX, BLOCK_X, dtype=torch.int32, device="cuda")
    # Randomly shuffle the row offsets.
    x_offsets = x_offsets[torch.randperm(BLOCK_X, device="cuda")]

    src = torch.randn((BLOCK_X, BLOCK_Y), dtype=dtype, device="cuda")
    async_scatter(input, x_offsets, y_offset, src, BLOCK_X, BLOCK_Y)

    # Mask out-of-bounds row offsets.
    mask = x_offsets < X_MAX
    x_offsets = x_offsets[mask]
    src = src[mask]

    # Mask out-of-bounds column offsets.
    y_hi = min(y_offset + BLOCK_Y, Y_MAX)

    input_ref[x_offsets, y_offset:y_hi] = src[:, :y_hi - y_offset]
    torch.testing.assert_close(input, input_ref, atol=0, rtol=0)
```

**EN:** Validates scatter semantics. The test randomizes valid and out-of-bounds row indices, performs the scatter, then updates a PyTorch reference only for rows still inside bounds and only for the valid column prefix. This matches the documented ignore-on-OOB behavior of native scatter.

**CN:** 这一段验证 scatter 的语义：测试会混合合法与越界的行索引，执行 scatter 后，再只对仍在范围内的行以及合法列前缀更新 PyTorch 参考结果，从而与原生 scatter“越界写入忽略”的行为保持一致。

### Lines 442-487

```python
# %%
# `async_gather` and `async_scatter` can be pipelined just like `async_copy_global_to_shared`
# and `async_copy_shared_to_global`. To demonstrate this, we will write a matmul kernel
# that has a fused gather and fused scatter along the M dimension:
# `out[out_scatter_indx, :] = X[X_gather_indx, :] @ W`.
#
# Recall in `06-tcgen05-mma` that we demonstrated how to write matmul kernels
# with `tcgen05_mma`. This example performs pipelining of the TMA loads, including `async_gather`,
# with `tcgen05_mma` and pipelining of the `async_scatter` with the persistent outer loop.
#
# In our blocked matmul kernrel with fused gather and scatter, for each tile of the output,
# we will load the M dimension offsets for the X tensor tile and the N dimension offsets for the W
# tensor tile via `gl.load` and schedule them sufficiently ahead of their use to account for the
# latency of the global loads.


@gluon.jit
def issue_loads(producer, X_desc, W_desc, X_gather_indx_ptr, off_m, off_n, k, bars, x_bufs, w_bufs,
                BLOCK_M: gl.constexpr, num_buffers: gl.constexpr, pred=True):
    # Load the M dimension offsets for the X tensor tile. We expect the load to be small
    # enough (no more than 128 elements) that we don't need to use a coalesced layout. Load directly into the layout
    # required by `async_gather` to avoid the layout conversion.
    gather_indx_layout: gl.constexpr = gl.SliceLayout(0, gl.BlockedLayout([1, 4], [32, 1], [1, gl.num_warps()], [1, 0]))
    offs_x_m = gl.load(X_gather_indx_ptr + off_m + gl.arange(0, BLOCK_M, gather_indx_layout))

    index = producer % num_buffers
    producer += 1
    bar = bars.index(index)

    # The W tensor tile is loaded using a regular `async_copy_global_to_shared`.
    mbarrier.expect(bar, W_desc.block_type.nbytes + BLOCK_M * X_desc.block_type.nbytes)
    tma.async_gather(X_desc, offs_x_m, k, bar, x_bufs.index(index), pred)
    tma.async_load(W_desc, [k, off_n], bar, w_bufs.index(index), pred)
    return producer


@gluon.jit
def issue_mma(consumer, mma, bars, x_bufs, w_bufs, num_buffers: gl.constexpr):
    index = consumer % num_buffers
    b_index = consumer % num_buffers
    phase = consumer // num_buffers & 1
    consumer += 1
    mbarrier.wait(bars.index(index), phase)
    mma = mma.wait_num_outstanding(0)
    mma = mma.issue_async_mma(x_bufs.index(index), w_bufs.index(b_index))
    return consumer, mma
```

**EN:** Introduces a fused matmul pipeline `out[out_scatter_indx, :] = X[X_gather_indx, :] @ W`. `issue_loads()` multi-buffers two asynchronous producers: gathered rows from `X` via `tma.async_gather` and dense tiles from `W` via `tma.async_load`, both tracked by mbarriers. `issue_mma()` waits on a ready buffer and issues asynchronous tensor-core MMA using helper code imported from tutorial 07.

**CN:** 本段引出融合 gather/scatter 的矩阵乘流水线：`out[out_scatter_indx, :] = X[X_gather_indx, :] @ W`。`issue_loads()` 负责双重异步生产者的多缓冲：一边用 `tma.async_gather` 取 `X` 的离散行，一边用 `tma.async_load` 读取 `W` 的稠密块，并统一由 mbarrier 跟踪。`issue_mma()` 则等待缓冲区就绪后，调用教程 07 中复用的辅助逻辑发出异步 Tensor Core MMA。

### Lines 491-572

```python
def matmul_fused_gather_scatter_kernel(X_desc, W_desc, out_desc, X_gather_indx_ptr, out_scatter_indx_ptr,
                                       BLOCK_M: gl.constexpr, SchedulerImpl: gl.constexpr, num_buffers: gl.constexpr):
    BLOCK_N: gl.constexpr = W_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = W_desc.block_type.shape[0]
    dtype: gl.constexpr = X_desc.dtype
    M = X_desc.shape[0]
    N = W_desc.shape[1]
    K = X_desc.shape[1]

    # Allocate shared memory for the input tiles.
    x_bufs = gl.allocate_shared_memory(dtype, [num_buffers, BLOCK_M, BLOCK_K], X_desc.layout)
    w_bufs = gl.allocate_shared_memory(dtype, [num_buffers, BLOCK_K, BLOCK_N], W_desc.layout)

    # Allocate shared memory for the output tile.
    out_smem = gl.allocate_shared_memory(dtype, [BLOCK_M, BLOCK_N], out_desc.layout)

    # Initialize barriers for multibuffering the loads.
    bars = gl.allocate_shared_memory(gl.int64, [num_buffers, 1], mbarrier.MBarrierLayout())
    for i in gl.static_range(num_buffers):
        mbarrier.init(bars.index(i), count=1)
    producer = 0
    consumer = 0

    mma = t7.MMAv5.initialize(dtype, BLOCK_M, BLOCK_N, gl.num_warps())
    scheduler = SchedulerImpl.initialize(M, N, BLOCK_M, BLOCK_N)
    num_tiles = scheduler.get_num_tiles()

    # Peeled inner loop prologue.
    idx = 0
    pid_m, pid_n = scheduler.get_tile(idx)
    off_m = pid_m * BLOCK_M
    off_n = pid_n * BLOCK_N
    for ki in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
        producer = issue_loads(producer, X_desc, W_desc, X_gather_indx_ptr, off_m, off_n, ki, bars, x_bufs, w_bufs,
                               BLOCK_M, num_buffers)
    k = BLOCK_K * (num_buffers - 2)
    producer = issue_loads(producer, X_desc, W_desc, X_gather_indx_ptr, off_m, off_n, k, bars, x_bufs, w_bufs, BLOCK_M,
                           num_buffers)

    for _ in range(num_tiles):
        consumer, mma = issue_mma(consumer, mma, bars, x_bufs, w_bufs, num_buffers)
        for k in range(BLOCK_K * (num_buffers - 1), K, BLOCK_K):
            producer = issue_loads(producer, X_desc, W_desc, X_gather_indx_ptr, off_m, off_n, k, bars, x_bufs, w_bufs,
                                   BLOCK_M, num_buffers)
            consumer, mma = issue_mma(consumer, mma, bars, x_bufs, w_bufs, num_buffers)

        epilogue_off_m = off_m
        epilogue_off_n = off_n

        # Load the M dimension offsets for the output tile. We expect the load to be small
        # enough (no more than 128 elements) that we don't need to use a coalesced layout.
        # Load directly into the layout required by `async_scatter` to avoid the layout conversion.
        scatter_indx_layout: gl.constexpr = gl.SliceLayout(
            0, gl.BlockedLayout([1, 4], [32, 1], [1, gl.num_warps()], [1, 0]))
        out_offs_m = gl.load(out_scatter_indx_ptr + epilogue_off_m + gl.arange(0, BLOCK_M, scatter_indx_layout))

        # Peel the next prologue and fuse it with the pipeline drain loop.
        idx += 1
        pid_m, pid_n = scheduler.get_tile(idx)
        off_m = pid_m * BLOCK_M
        off_n = pid_n * BLOCK_N

        # Predicate the peeled prologue instead of using a conditional.
        pred = idx < num_tiles
        for ki in gl.static_range(0, BLOCK_K * (num_buffers - 2), BLOCK_K):
            producer = issue_loads(producer, X_desc, W_desc, X_gather_indx_ptr, off_m, off_n, ki, bars, x_bufs, w_bufs,
                                   BLOCK_M, num_buffers, pred)
            consumer, mma = issue_mma(consumer, mma, bars, x_bufs, w_bufs, num_buffers)
        k = BLOCK_K * (num_buffers - 2)
        producer = issue_loads(producer, X_desc, W_desc, X_gather_indx_ptr, off_m, off_n, k, bars, x_bufs, w_bufs,
                               BLOCK_M, num_buffers)

        mma = mma.wait_num_outstanding(0)
        out, mma = mma.take_result()
        out = out.to(dtype)
        # Pipeline the async scatter by waiting for the previous store to complete.
        tma.store_wait(pendings=0)
        out_smem.store(out)
        fence_async_shared()
        tma.async_scatter(out_desc, out_offs_m, epilogue_off_n, out_smem)
    # Wait for the last async scatter to complete.
    tma.store_wait(pendings=0)
```

**EN:** Defines the main fused kernel. It allocates multi-buffer shared-memory staging areas for `X` and `W`, a shared output tile, and per-buffer mbarriers; uses the persistent grouped scheduler from tutorial 07; primes the K-loop pipeline; overlaps further gathers/loads with MMA consumption; loads output scatter indices for the epilogue; then writes the accumulated tile to shared memory and emits `tma.async_scatter` so the store stage overlaps with the next tile's work.

**CN:** 定义主融合内核：先为 `X`、`W` 分配多缓冲共享内存 staging 区，为输出分配共享内存 tile，并为每个缓冲区准备 mbarrier；随后使用教程 07 的持久化 grouped scheduler，先填满 K 维流水线，再在消费 MMA 的同时继续发起后续 gather/load；尾声阶段读取输出的 scatter 行索引，把累计结果写入共享内存后通过 `tma.async_scatter` 发回全局内存，使写回过程与下一 tile 的计算重叠。

### Lines 575-643

```python
# %%
# We will pick reasonable defaults for the block sizes and number of load buffers.
# Tuning and optimizing the performance of this kernel is left as an exercise for the reader,
# as the primary objective of this tutorial is to demonstrate the use of async gather and scatter.
#
# The only alternative way to implement a matmul kernel with fused gather and
# scatter is to use async_copy (recall `03-async-copy`) or `gl.load` to load
# from global memory and `gl.store` to write to the output tensor in the
# epilogue. While these instructions provide more flexible indexing, they are
# much slower than TMA and async gather and scatter.
#
# One extra note: it is of course possible to use async gather and async scatter with
# warp-specialized kernels. Just keep in mind that because the row offsets is a tensor, you may want
# to give the load and epilogue partitions more than 1 warp to increase instruction issue throughput,
# particularly for the loads as they are on the critical path.


def matmul_fused_gather_scatter(X, X_gather_indx, W, out_scatter_indx, BLOCK_M=128, BLOCK_N=128, BLOCK_K=64,
                                GROUP_SIZE_M=8, num_buffers=3):
    M = X.shape[0]
    N = W.shape[1]
    out = torch.empty((M, N), dtype=X.dtype, device="cuda")

    # Convert torch dtype to gluon dtype.
    dtype = getattr(gl, str(X.dtype).split('.')[1])
    # Setup descriptors for inputs and outputs.
    X_desc_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], dtype)
    W_desc_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], dtype)
    out_desc_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], dtype)

    X_desc = TensorDescriptor.from_tensor(X, [1, BLOCK_K], X_desc_layout)
    W_desc = TensorDescriptor.from_tensor(W, [BLOCK_K, BLOCK_N], W_desc_layout)
    out_desc = TensorDescriptor.from_tensor(out, [1, BLOCK_N], out_desc_layout)

    # Persistent kernel grid.
    num_sms = torch.cuda.get_device_properties("cuda").multi_processor_count
    num_pid = triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N)
    grid = (min(num_sms, num_pid), )
    SchedulerImpl = t7.GroupedPersistentTileScheduler(GROUP_SIZE_M)
    matmul_fused_gather_scatter_kernel[grid](X_desc, W_desc, out_desc, X_gather_indx, out_scatter_indx, BLOCK_M,
                                             SchedulerImpl, num_buffers)
    return out


@pytest.mark.parametrize("M, N, K", [(1024, 1024, 2048), (4096, 4096, 4096)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N", [(128, 128), (128, 64)])
@pytest.mark.parametrize("BLOCK_K, num_buffers", [(128, 2), (64, 3)])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_matmul_fused_gather_scatter(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, num_buffers):
    torch.manual_seed(0)

    # Randomize the gather indices.
    X_gather_indx = torch.arange(0, M, dtype=torch.int32, device="cuda")
    shfl = torch.randperm(M, device="cuda")
    X_gather_indx = X_gather_indx[shfl]

    # Randomize the scatter indices.
    out_scatter_indx = torch.arange(0, M, dtype=torch.int32, device="cuda")
    shfl = torch.randperm(M, device="cuda")
    out_scatter_indx = out_scatter_indx[shfl]

    X = torch.randn(M, K, dtype=torch.bfloat16, device="cuda")
    W = torch.randn(K, N, dtype=torch.bfloat16, device="cuda")
    out = matmul_fused_gather_scatter(X, X_gather_indx, W, out_scatter_indx, BLOCK_M, BLOCK_N, BLOCK_K,
                                      num_buffers=num_buffers)

    out_ref = torch.empty_like(out)
    out_ref[out_scatter_indx, :] = X[X_gather_indx, :] @ W
    torch.testing.assert_close(out, out_ref, atol=1e-3, rtol=1e-3)
```

**EN:** Provides the Python launcher and correctness test for the fused kernel. The wrapper creates descriptors for `X`, `W`, and the output tensor, chooses a persistent grid of `min(num_sms, num_tiles)`, and launches the kernel with configurable block sizes and buffer count. The test independently randomizes gather rows and scatter rows, then checks against the reference expression `out_ref[out_scatter_indx, :] = X[X_gather_indx, :] @ W`.

**CN:** 给出融合内核的 Python 启动器和正确性测试。包装函数为 `X`、`W` 和输出张量创建描述符，采用 `min(num_sms, num_tiles)` 的持久化网格，并允许配置块大小与缓冲区数量。测试部分分别随机化 gather 行索引和 scatter 行索引，再与参考表达式 `out_ref[out_scatter_indx, :] = X[X_gather_indx, :] @ W` 对比。

### Lines 646-662

```python
# %%
# The main takeaway from this tutorial is understanding how to use `async_gather`
# and `async_scatter`. These instructions provide a middle-ground between
# block DMAs like `async_copy_global_to_shared` and `async_copy_shared_to_global`
# and regular global loads and stores (`gl.load` and `gl.store`) by allowing
# separately-indexed columns while maintaining the performance of TMAs.
#
# Keep in mind the following:
# - `async_gather` and `async_scatter` are typically faster than `gl.load` and
#   `gl.store` when they can be used, but this is not always the case. Plus, TMA
#   instructions use shared memory.
# - Sometimes using `async_gather` or `async_scatter` instead of block DMA
#   instructions like `async_copy_global_to_shared` and `async_copy_shared_to_global`
#   is actually faster, but these situations are rare.
#
# In general, you should consider these instructions when writing kernels and
# experiment to see what is the best way to write a kernel.
```

**EN:** Closes with the tutorial's main guidance: native async gather/scatter sit between coarse block DMA and ordinary `gl.load`/`gl.store`. They preserve TMA-style throughput while allowing row-wise indirection, but they still consume shared memory and only win when the shape/alignment constraints match the workload.

**CN:** 结尾强调本教程的核心观点：原生 async gather/scatter 位于“大块 DMA”与普通 `gl.load`/`gl.store` 之间，既保留了接近 TMA 的吞吐优势，又提供了按行间接寻址能力；但它们依然依赖共享内存，且只有在形状、对齐和访问模式都合适时才真正占优。

## Key Concepts / 关键概念

- **Native TMA gather/scatter** — **EN:** Operates on 2D tensor descriptors but lets each row come from a different row index, giving more flexibility than plain block TMA. **CN:** 作用于二维张量描述符，但允许每一行来自不同的行索引，因此比普通块式 TMA 更灵活。
- **Layout legality** — **EN:** The row-index tensor must follow the gather4/scatter4 register/lane pattern; otherwise codegen may be illegal or severely underutilized. **CN:** 行索引张量必须满足 gather4/scatter4 对寄存器与 lane 映射的要求，否则代码生成可能非法，或严重浪费并行度。
- **Async proxy + fences** — **EN:** Shared-memory data exchanged with TMA must respect proxy rules; scatter paths need `fence_async_shared()` when generic shared-memory stores precede async-proxy TMA access. **CN:** 与 TMA 共享的共享内存数据必须遵守 proxy 规则；当 generic proxy 的共享内存写入先于 async proxy 的 TMA 访问时，需要插入 `fence_async_shared()`。
- **mbarrier synchronization** — **EN:** Gather and async loads signal completion through mbarriers, allowing producer/consumer pipelining without a full CTA-wide barrier. **CN:** gather 与异步加载通过 mbarrier 报告完成，从而无需整个 CTA 的大同步点，也能实现生产者/消费者流水线。
- **Persistent pipelining** — **EN:** The fused matmul kernel keeps SMs resident on a persistent grid and overlaps gather, dense TMA loads, MMA, and scatter across tiles and K-slices. **CN:** 融合矩阵乘内核采用持久化网格，让 SM 持续驻留，并在不同 tile 与 K 分块之间重叠 gather、稠密 TMA load、MMA 和 scatter。

## Dependencies / 依赖关系

- **`torch`** — **EN:** Allocates CUDA tensors, prepares randomized test data, and checks numerical correctness. **CN:** 负责分配 CUDA 张量、生成随机测试数据，并校验数值正确性。
- **`triton`** — **EN:** Provides runtime target detection, kernel launch syntax, utility functions like `cdiv`, and compilation knobs. **CN:** 提供运行时目标检测、kernel 启动语法、`cdiv` 等工具函数以及编译选项。
- **`triton.experimental.gluon` / `language as gl`** — **EN:** Defines the Gluon JIT, layouts, shared-memory objects, tensor operations, and compile-time constants used throughout the kernels. **CN:** 提供 Gluon JIT、布局系统、共享内存对象、张量操作以及内核中使用的编译期常量。
- **`TensorDescriptor`** — **EN:** Wraps PyTorch tensors as hardware TMA descriptors; gather/scatter specifically require 2D descriptors with block shape `[1, BLOCK_Y]`. **CN:** 把 PyTorch 张量包装为硬件 TMA 描述符；其中 gather/scatter 特别要求二维描述符且块形状为 `[1, BLOCK_Y]`。
- **Blackwell-specific `tma`, `mbarrier`, `fence_async_shared`** — **EN:** Expose the native Blackwell async gather/scatter/load/store primitives and their synchronization model. **CN:** 暴露 Blackwell 原生 async gather/scatter/load/store 原语及其同步模型。
- **`triton._C.libtriton.ir` and `gluon_ir`** — **EN:** Used only for layout introspection in `to_linear_layout()`, not for the fast path kernels. **CN:** 仅用于 `to_linear_layout()` 中的布局分析，不参与性能路径内核的执行。
- **Tutorial `07-persistence`** — **EN:** Supplies `GroupedPersistentTileScheduler` and `MMAv5`, reused to build the persistent fused matmul pipeline. **CN:** 提供 `GroupedPersistentTileScheduler` 和 `MMAv5`，用于复用到融合矩阵乘的持久化流水线中。
- **`pytest`** — **EN:** Drives parameterized correctness tests and Blackwell-only skips. **CN:** 用于参数化正确性测试，并在非 Blackwell 环境下跳过执行。

