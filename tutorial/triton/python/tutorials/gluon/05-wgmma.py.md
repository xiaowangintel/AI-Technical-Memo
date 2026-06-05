# 05-wgmma.py — Code Analysis / 代码分析

## Source / 来源
- Source file: `/root/xw/triton/python/tutorials/gluon/05-wgmma.py`
- Purpose: Explain how this Triton Gluon tutorial uses Hopper WGMMA/TMA primitives to build, validate, and benchmark progressively more optimized matrix-multiplication kernels. / 作用：说明该 Triton Gluon 教程如何使用 Hopper 的 WGMMA/TMA 原语来构建、验证并基准测试逐步优化的矩阵乘法 kernel。

## Line-by-Line Analysis / 逐行分析

### Lines 1-38 / 第 1-38 行

```python
"""
Warp-Group MMA
==============

Warp-Group MMA (also known as WGMMA or MMAv3) is a Hopper-specific instruction
for performing matrix multiply-accumulate operations using the Tensor Cores.
WGMMA instructions are asynchronous, meaning they can be pipelined.

In this tutorial, we will cover how to use WGMMAs in Gluon. We will build a
simple matmul kernel to demonstrate practical uses of WGMMA, and show an example
where WGMMAs can be pipelined for better performance.
"""

import pytest
import torch
import triton
import itertools
from triton.experimental import gluon
from triton.experimental.gluon import language as gl

from triton.experimental.gluon.nvidia.hopper import TensorDescriptor
from triton.experimental.gluon.language.nvidia.hopper import (
    tma,
    mbarrier,
    fence_async_shared,
    warpgroup_mma_init,
    warpgroup_mma,
    warpgroup_mma_wait,
)


def is_hopper():
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] == 9


if __name__ == "__main__" and not is_hopper():
    raise RuntimeError("This tutorial requires a Hopper NVIDIA GPU")
```

**EN:** The file opens with a tutorial docstring, imports Triton/Gluon Hopper primitives, and defines `is_hopper()` to gate execution to NVIDIA Hopper (compute capability 9.x). The `__main__` guard fails fast on unsupported GPUs, which is important because WGMMA and TMA are Hopper-only features.

**CN:** 文件首先用文档字符串说明教程主题，然后导入 Triton/Gluon 的 Hopper 专用原语，并通过 `is_hopper()` 将执行限制在 NVIDIA Hopper（计算能力 9.x）设备上。`__main__` 的保护逻辑会在不支持的 GPU 上立即报错，因为 WGMMA 和 TMA 都是 Hopper 专属特性。

### Lines 40-120 / 第 40-120 行

```python
# %%
# Let's illustrate WGMMA with a trivial kernel launched with grid size (1, ).
# This kernel performs MMA on a small tensor.
#
# warpgroup_mma performs d = a * b + c. The `a` operand can be passed as
# registers or through shared memory. The `b` operand must be passed through
# shared memory, and the `c` operand must be passed through registers.
#
# warpgroup_mma itself is composed of many smaller `wgmma.mma_async` PTX
# instructions, which supports a limited set of instruction shapes.
#
# The instruction shape is specified as [m, n, k], where
#
# - `k` is always 256 / A.dtype.primitive_bitwidth
# - `m` is always 16
# - `n` can be can chosen as follows:
#
# For floating point dtypes, `n` must be a positive multiple of 8, up to and
# including 256. WGMMA supports 8-bit integers, but `n` must be chosen from:
#
#   224, 208, 192, 176, 160, 144, 128, 112, 96, 80, 64, 48, 32, 24, 16, 8
#
# `n` must be chosen such that it evenly divides into `BLOCK_N`, the inner
# dimension of the MMA tile, and it must be less than or equal to `maxN`, where
# `maxN` is computed as:
#
#     mReps = ceildiv(M, m)
#     nReps = ceildiv(num_warps, mReps)
#     maxN = max(N // nReps, 8)
#
# warpgroup_mma divides the MMA across warps using `warps_per_cta`, in the
# same way `BlockedLayout.warps_per_cta` tiles a tensor across warps. The
# smallest indivisible unit of `warps_per_cta` is `[4, 1]`. Note that this
# means WGMMA requires at least 4 warps, which together make up one warp group.
# To choose the right `warps_per_cta`, start from the atom `[4, 1]` and simply
# double it along any dimension until it matches the number of warps. Note that
# since `m=16` and must be at least 4 wraps along M, the M dimension must be at
# least 64.
#
# Note when `num_warps=8`, we can choose `[4, 2]` or `[8, 1]`, but recall from
# 02-layouts that this can affect the performance of, e.g., reductions.
#
# warpgroup_mma is an asynchronous operation whose completion is tracked by
# commit groups, like async copies and TMA stores. Issuing a WGMMA operation
# implicitly commits it to a WGMMA group, and we can wait until there are N
# outstanding operations.
#
# Because warpgroup_mma is an asynchronous, until the operation is complete,
# we cannot access the result even though it is in registers, and we cannot
# write to any of the shared memory inputs. WGMMA accesses shared memory through
# the async proxy. Since TMAs also access shared memory through the async proxy,
# we don't need fences between TMA and WGMMA instructions.
#
# ```python
# b_smem.store(b)
# fence_async_shared()
# warpgroup_mma(a, b_smem, c, is_async=True)
# ```
#
# A fence is needed between the shared store and warpgroup_mma to order their
# shared memory accesses.
#
# Completion of the WGMMA implies its reads from shared memory are complete.
# Thus, it is safe to write to the shared memory inputs after waiting:
#
# ```python
# d = warpgroup_mma(a, b_smem, c, is_async=True)
# d = warpgroup_mma_wait(num_outstanding=0, deps=(d, ))
# b_smem.store(b)
# ```
#
# If the LHS operand is supplied in registers via a shared load, completion of
# the WGMMA implies the shared load is complete, and subsequent accesses to the
# buffer via the async proxy do not require a fence:
#
# ```python
# a = a_smem.load(dot_operand_layout)
# d = warpgroup_mma(a, b_smem, c, is_async=True)
# d = warpgroup_mma_wait(num_outstanding=0, deps=(d, ))
# tma.async_load(a_desc, [0, 0], bar, a_smem)
# ```
```

**EN:** This large comment block is the contract for using `warpgroup_mma`. It explains operand placement rules (A in registers or shared memory, B in shared memory, C in registers), the legal WGMMA instruction shape `[m, n, k]`, how `warps_per_cta` maps a warp group onto the tile, and the async ordering model. The fence examples are especially important: shared-memory writes must be ordered before WGMMA reads, while `warpgroup_mma_wait` makes later reuse of the shared operands safe.

**CN:** 这一大段注释实际上定义了 `warpgroup_mma` 的使用契约：说明了操作数放置规则（A 可在寄存器或共享内存中，B 必须在共享内存中，C 必须在寄存器中）、合法的 WGMMA 指令形状 `[m, n, k]`、`warps_per_cta` 如何把一个 warp group 映射到 tile 上，以及异步执行下的顺序语义。这里的 fence 示例尤其关键：共享内存写入必须先于 WGMMA 的读取，而 `warpgroup_mma_wait` 则保证后续安全复用这些共享内存操作数。

### Lines 126-202 / 第 126-202 行

```python
@gluon.jit
def small_mma_kernel(a_desc, b_desc, c_desc, d_desc,  #
                     LHS_IN_REG: gl.constexpr, INSTR_SHAPE_N: gl.constexpr, num_warps: gl.constexpr):
    # Load A, B, and C tiles.
    bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)

    # A has shape [M, K].
    a_smem = gl.allocate_shared_memory(a_desc.dtype, a_desc.block_type.shape, a_desc.layout)
    # B has shape [K, N].
    b_smem = gl.allocate_shared_memory(b_desc.dtype, b_desc.block_type.shape, b_desc.layout)
    # C has shape [M, N].
    c_smem = gl.allocate_shared_memory(c_desc.dtype, c_desc.block_type.shape, c_desc.layout)

    mbarrier.expect(bar, a_desc.block_type.nbytes + b_desc.block_type.nbytes + c_desc.block_type.nbytes)
    tma.async_load(a_desc, [0, 0], bar, a_smem)
    tma.async_load(b_desc, [0, 0], bar, b_smem)
    tma.async_load(c_desc, [0, 0], bar, c_smem)
    mbarrier.wait(bar, phase=0)
    mbarrier.invalidate(bar)

    # Let's parameterize the kernel over LHS_IN_REG and INSTR_SHAPE_N to see how
    # it can affect performance.
    m: gl.constexpr = 16
    k: gl.constexpr = 256 // a_desc.dtype.primitive_bitwidth
    n: gl.constexpr = INSTR_SHAPE_N
    warps_per_cta: gl.constexpr = [num_warps, 1]

    # The MMA shape is passed through the layout of `c`, which must always have
    # an NVMMADistributedLayout.
    c_layout: gl.constexpr = gl.NVMMADistributedLayout(
        version=[3, 0],
        warps_per_cta=warps_per_cta,
        instr_shape=[m, n, k],
    )

    # When A is passed through registers, it must have the following layout:
    a_reg_layout: gl.constexpr = gl.DotOperandLayout(
        operand_index=0,
        parent=c_layout,
        k_width=32 // a_desc.dtype.primitive_bitwidth,
    )

    # When an operand is passed through shared memory, it must have an
    # NVMMASharedLayout. TMA requires using an NVMMASharedLayout.
    gl.static_assert(isinstance(a_smem.type.layout, gl.NVMMASharedLayout))
    gl.static_assert(isinstance(b_smem.type.layout, gl.NVMMASharedLayout))

    if LHS_IN_REG:
        a = a_smem.load(a_reg_layout)
    else:
        a = a_smem

    c = c_smem.load(c_layout)
    # Issue the async WGMMA. Note that `is_async=False` is the default value,
    # and all this does is immediately wait for 0 outstanding operations. In
    # this tutorial, we will always use `is_async=True`.
    #
    # Another important flag to consider is `use_acc`. When `use_acc=False`, the
    # `c` input is ignored and the accumulator is zero-initialized. This can be
    # an efficient way to zero the accumulator.
    d = warpgroup_mma(a, b_smem, c, is_async=True, use_acc=True)

    # To ensure correct ordering between `warpgroup_mma`, the wait, and uses of
    # the result, you must thread the `warpgroup_mma` result through the wait
    # via the `deps` argument and use the return value of the
    # `warpgroup_mma_wait`.
    #
    # Wait for 0 outstanding operations, so we know the WGMMA is complete.
    d = warpgroup_mma_wait(num_outstanding=0, deps=(d, ))

    d_smem = gl.allocate_shared_memory(d_desc.dtype, d_desc.block_type.shape, d_desc.layout)
    d_smem.store(d)
    fence_async_shared()
    tma.async_copy_shared_to_global(d_desc, [0, 0], d_smem)
    tma.store_wait(pendings=0)

```

**EN:** `small_mma_kernel` is the first real Gluon kernel. It allocates shared-memory buffers for A/B/C, uses a single mbarrier to synchronize three TMA async loads, and constructs an `NVMMADistributedLayout` that encodes the chosen WGMMA instruction shape. If `LHS_IN_REG` is enabled, A is loaded into registers with `DotOperandLayout`; otherwise A stays in shared memory. The kernel then issues `warpgroup_mma(..., is_async=True)`, waits for completion with dependency threading, writes the result tile to shared memory, and finally uses TMA to store back to global memory.

**CN:** `small_mma_kernel` 是第一个真正执行的 Gluon kernel。它为 A/B/C 分配共享内存缓冲区，使用一个 mbarrier 协调三次 TMA 异步加载，并构造 `NVMMADistributedLayout` 来编码所选的 WGMMA 指令形状。如果启用 `LHS_IN_REG`，A 会通过 `DotOperandLayout` 载入寄存器；否则 A 保持在共享内存中。随后 kernel 调用 `warpgroup_mma(..., is_async=True)` 发射异步 WGMMA，通过依赖链调用 `warpgroup_mma_wait` 等待完成，再把结果写回共享内存，并借助 TMA 存回全局内存。

### Lines 204-234 / 第 204-234 行

```python
def small_mma(A, B, C, D, INSTR_SHAPE_N, LHS_IN_REG=False, num_warps=4):
    a_layout = gl.NVMMASharedLayout.get_default_for(A.shape, gl.float16)
    b_layout = gl.NVMMASharedLayout.get_default_for(B.shape, gl.float16)
    cd_layout = gl.NVMMASharedLayout.get_default_for(C.shape, gl.float32)

    a_desc = TensorDescriptor.from_tensor(A, A.shape, a_layout)
    b_desc = TensorDescriptor.from_tensor(B, B.shape, b_layout)
    c_desc = TensorDescriptor.from_tensor(C, C.shape, cd_layout)
    d_desc = TensorDescriptor.from_tensor(D, D.shape, cd_layout)

    small_mma_kernel[(1, )](
        a_desc, b_desc, c_desc, d_desc,  #
        LHS_IN_REG, INSTR_SHAPE_N, num_warps=num_warps)


@pytest.mark.parametrize("M, N, K", [(64, 32, 32), (64, 256, 128)])
@pytest.mark.parametrize("LHS_IN_REG", [False, True])
@pytest.mark.parametrize("INSTR_SHAPE_N", [16, 64])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.skipif(not is_hopper(), reason="Requires Hopper")
def test_small_mma(M, N, K, LHS_IN_REG, INSTR_SHAPE_N, num_warps):
    maxN = max(N // triton.cdiv(num_warps, triton.cdiv(M, 16)), 8)
    if INSTR_SHAPE_N > maxN:
        pytest.skip(f"INSTR_SHAPE_N={INSTR_SHAPE_N} is too large for M={M}, N={N}, num_warps={num_warps}")

    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.randn(M, N, device="cuda", dtype=torch.float32)
    D = torch.empty_like(C)
    small_mma(A, B, C, D, INSTR_SHAPE_N, LHS_IN_REG, num_warps)
    torch.testing.assert_close(A @ B + C, D, atol=1e-3, rtol=1e-1)
```

**EN:** `small_mma` is the Python wrapper that creates `TensorDescriptor` objects with default `NVMMASharedLayout`s and launches exactly one program instance. The test matrix checks multiple shapes, operand-placement choices, instruction widths, and warp counts. Before launching, it skips impossible `INSTR_SHAPE_N` values using the same `maxN` rule described in the comments, then validates the result against `A @ B + C` in PyTorch.

**CN:** `small_mma` 是 Python 包装层：它为输入张量创建带默认 `NVMMASharedLayout` 的 `TensorDescriptor`，并只启动一个 program instance。测试覆盖了多种矩阵尺寸、左操作数放置方式、指令宽度和 warp 数量。启动前，它依据注释中相同的 `maxN` 规则跳过不合法的 `INSTR_SHAPE_N`，最后用 PyTorch 的 `A @ B + C` 校验结果。

### Lines 240-274 / 第 240-274 行

```python
if __name__ == "__main__":
    print("Benchmarking WGMMA")
    print("==================")
    M, N, K = 64, 128, 128
    num_warps = 4
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.randn(M, N, device="cuda", dtype=torch.float32)
    D = torch.empty_like(C)

    print("LHS_IN_REG INSTR_SHAPE_N time (us)")
    for LHS_IN_REG, INSTR_SHAPE_N in itertools.product([False, True], [16, 32, 64, 128]):
        fn = lambda: small_mma(A, B, C, D, INSTR_SHAPE_N, LHS_IN_REG, num_warps)
        ms = triton.testing.do_bench(fn)
        print(f"{LHS_IN_REG!s:>10} {INSTR_SHAPE_N:>13} {ms*1000:>9.2f}")
    print()

# %%
# ```
# LHS_IN_REG INSTR_SHAPE_N time (us)
#      False            16      9.47
#      False            32      8.48
#      False            64      8.32
#      False           128      8.32
#       True            16      9.32
#       True            32      8.60
#       True            64      8.37
#       True           128      8.36
# ```
#
# Picking the largest N results in the best performance, because each
# `wgmma.mma_async` instruction will process more data. In our case, placing LHS
# in registers is slower because we had to load the data out of shared memory.
# However, if the data was already in registers, it would be faster to use it in
# registers instead of placing it in shared memory.
```

**EN:** This benchmark sweeps the two tutorial knobs: whether the left operand is kept in registers and the WGMMA `n` dimension. The embedded results show the expected trend that larger instruction shapes deliver more work per `wgmma.mma_async`. In this specific microkernel, putting A in registers loses slightly because it first has to be loaded out of shared memory.

**CN:** 这一段基准测试扫描教程中的两个主要调参项：左操作数是否放在寄存器中，以及 WGMMA 的 `n` 维度。附带结果显示出典型趋势：更大的指令形状会让每条 `wgmma.mma_async` 处理更多数据，因此通常更快。在这个微型 kernel 中，把 A 放入寄存器反而略慢，因为它仍然需要先从共享内存中加载出来。

### Lines 276-323 / 第 276-323 行

```python
# %%
# Just like `warpgroup_mma` is composed of multiple `wgmma.mma_async`
# instructions tiled to cover our block size, we can also tile `warpgroup_mma`
# to cover a much larger matmul. We can tile along K within each kernel and span
# (M, N) with multiple programs. This leads to the classic blocked matmul
# implementation. Let's implement a basic version to demonstrate WGMMA.


# This decorator allows us to invoke the function from a Gluon constexpr.
@gluon.constexpr_function
def get_warps_per_cta(BLOCK_M, BLOCK_N, num_warps):
    warps_per_cta = [4, 1]
    m = 16
    # Tile the atom until we have enough warps.
    while warps_per_cta[0] * warps_per_cta[1] != num_warps:
        # Tile along M only if it would not cause broadcasting.
        if BLOCK_M > m * warps_per_cta[0]:
            warps_per_cta[0] *= 2
        else:
            warps_per_cta[1] *= 2
    return warps_per_cta


@gluon.constexpr_function
def get_instr_shape_n(BLOCK_M, BLOCK_N, num_warps):
    m = 16
    mReps = triton.cdiv(BLOCK_M, m)
    nReps = triton.cdiv(num_warps, mReps)
    maxN = max(BLOCK_N // nReps, 8)
    n = 256
    while n > maxN or BLOCK_N % n != 0:
        n -= 8
    assert n >= 8, "expected to find a valid n"
    return n


@gluon.constexpr_function
def pick_wgmma_layout(dtype, BLOCK_M, BLOCK_N, num_warps):
    m = 16
    k = 256 // dtype.primitive_bitwidth
    n = get_instr_shape_n(BLOCK_M, BLOCK_N, num_warps)
    warps_per_cta = get_warps_per_cta(BLOCK_M, BLOCK_N, num_warps)
    return gl.NVMMADistributedLayout(
        version=[3, 0],
        warps_per_cta=warps_per_cta,
        instr_shape=[m, n, k],
    )

```

**EN:** The tutorial now scales from one WGMMA tile to a blocked GEMM. The constexpr helpers compute layout metadata at compile time: `get_warps_per_cta` grows the `[4, 1]` warp-group atom to match `num_warps`, `get_instr_shape_n` chooses the largest legal WGMMA `n`, and `pick_wgmma_layout` packages those decisions into an `NVMMADistributedLayout` used by the accumulator.

**CN:** 教程在这里从单个 WGMMA tile 扩展到分块 GEMM。几个 constexpr 辅助函数在编译期计算布局元数据：`get_warps_per_cta` 从 `[4, 1]` 这个 warp-group 原子形状扩展到目标 `num_warps`，`get_instr_shape_n` 选择最大的合法 WGMMA `n`，`pick_wgmma_layout` 则把这些决定封装为累加器使用的 `NVMMADistributedLayout`。

### Lines 325-380 / 第 325-380 行

```python
@gluon.jit
def blocked_matmul_kernel(a_desc, b_desc, c_desc,  #
                          TRANSPOSE_B: gl.constexpr, num_warps: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = a_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype
    K = a_desc.shape[1]

    a_smem = gl.allocate_shared_memory(dtype, a_desc.block_type.shape, a_desc.layout)
    b_smem = gl.allocate_shared_memory(dtype, b_desc.block_type.shape, b_desc.layout)

    # The block of C this program is processing is (pid_m, pid_n).
    pid_m = gl.program_id(axis=0)
    pid_n = gl.program_id(axis=1)
    off_m = pid_m * BLOCK_M
    off_n = pid_n * BLOCK_N

    # Determine the WGMMA layout.
    mma_layout: gl.constexpr = pick_wgmma_layout(dtype, BLOCK_M, BLOCK_N, num_warps)
    acc = gl.zeros((BLOCK_M, BLOCK_N), dtype=gl.float32, layout=mma_layout)

    bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)
    phase = 0

    for k in range(0, K, BLOCK_K):
        # Load tiles of A and B.
        mbarrier.expect(bar, a_desc.block_type.nbytes + b_desc.block_type.nbytes)
        tma.async_load(a_desc, [off_m, k], bar, a_smem)
        if TRANSPOSE_B:
            tma.async_load(b_desc, [off_n, k], bar, b_smem)
        else:
            tma.async_load(b_desc, [k, off_n], bar, b_smem)
        mbarrier.wait(bar, phase=phase)
        phase ^= 1  # toggle the parity phase between 0 and 1

        # We can transpose B by creating a transposed view over tile of B in
        # shared memory. This forwards the transposition to WGMMA, which handles
        # it for us.
        if TRANSPOSE_B:
            b = b_smem.permute((1, 0))
        else:
            b = b_smem

        acc = warpgroup_mma(a_smem, b, acc, is_async=True)
        acc = warpgroup_mma_wait(num_outstanding=0, deps=(acc, ))

    mbarrier.invalidate(bar)

    # Downcast accumulator and store tile of C.
    c_smem = gl.allocate_shared_memory(dtype, c_desc.block_type.shape, c_desc.layout)
    c_smem.store(acc.to(dtype))
    fence_async_shared()
    tma.async_copy_shared_to_global(c_desc, [off_m, off_n], c_smem)
    tma.store_wait(pendings=0)
```

**EN:** `blocked_matmul_kernel` is a classic tiled GEMM built from WGMMA. Each program instance owns one `(BLOCK_M, BLOCK_N)` tile of the output, computes global offsets from `program_id`, loops over K tiles, loads A and B tiles with TMA, optionally represents B as a transposed shared-memory view, and accumulates with asynchronous `warpgroup_mma`. After the K loop it downcasts the fp32 accumulator to the output dtype, stages it in shared memory, fences, and performs a TMA store to global memory.

**CN:** `blocked_matmul_kernel` 是一个由 WGMMA 构建的经典分块 GEMM。每个 program instance 负责输出中的一个 `(BLOCK_M, BLOCK_N)` tile，通过 `program_id` 计算全局偏移，在 K 维上循环加载 tile，使用 TMA 读入 A 和 B，并在需要时把 B 表示为共享内存上的转置视图，再通过异步 `warpgroup_mma` 进行累加。K 循环结束后，它将 fp32 累加器降精度到输出 dtype，经共享内存暂存、fence 排序后，再用 TMA 写回全局内存。

### Lines 383-414 / 第 383-414 行

```python
def blocked_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, TRANSPOSE_B, num_warps):
    M, N = C.shape

    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gl.float16)
    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)

    B_BLOCK_SHAPE = [BLOCK_N, BLOCK_K] if TRANSPOSE_B else [BLOCK_K, BLOCK_N]
    b_layout = gl.NVMMASharedLayout.get_default_for(B_BLOCK_SHAPE, gl.float16)
    b_desc = TensorDescriptor.from_tensor(B, B_BLOCK_SHAPE, b_layout)

    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], gl.float16)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)

    grid = (triton.cdiv(M, BLOCK_M), triton.cdiv(N, BLOCK_N))
    blocked_matmul_kernel[grid](a_desc, b_desc, c_desc, TRANSPOSE_B, num_warps=num_warps)


@pytest.mark.parametrize("M, N, K", [(208, 416, 304), (2000, 1000, 2000)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(64, 64, 64), (128, 128, 128)])
@pytest.mark.parametrize("TRANSPOSE_B", [False, True])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.skipif(not is_hopper(), reason="Requires Hopper")
def test_blocked_matmul(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, TRANSPOSE_B, num_warps):
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn((N, K) if TRANSPOSE_B else (K, N), device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)

    blocked_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, TRANSPOSE_B, num_warps)

    C_ref = A @ (B.T if TRANSPOSE_B else B)
    torch.testing.assert_close(C_ref, C, rtol=1e-3, atol=1e-1)

```

**EN:** The wrapper prepares descriptors that match the chosen block shapes, including a different logical block shape for B when `TRANSPOSE_B` is true. The grid is two-dimensional over M and N tiles. The test verifies both normal and transposed-B paths against PyTorch matmul, confirming that the shared-memory `permute((1, 0))` view correctly forwards the transpose into WGMMA.

**CN:** 包装函数根据给定 block 大小构造描述符；当 `TRANSPOSE_B=True` 时，B 的逻辑 block 形状会相应改变。grid 在 M 和 N 两个维度上展开。测试同时验证普通路径和 B 转置路径，并与 PyTorch matmul 对比，从而确认共享内存上的 `permute((1, 0))` 视图确实把转置正确传递给 WGMMA。

### Lines 416-475 / 第 416-475 行

```python
# %%
# We can benchmark this kernel as a baseline, but we need to pick the best block
# sizes. Rather than autotuning over all possibilities, we can apply some
# principles to narrow down the search space.
#
# We should try to pick the largest `n` for the WGMMA layout. Based on the
# formula for `maxN` this requires `BLOCK_N>=256`. Because our kernel does not
# overlap the TMA loads with WGMMA, we will want more than program resident on
# each SM so that when one kernel stalls, the SM can switch to the other. This
# is known as "occupancy". In detail, each SM has limited resources, and the
# resource usage of a kernel determines its max occupancy. The SM schedules work
# by warp using its warp scheduler, which can efficiently swap executing warps,
# almost like hyperthreading.
#
# Based on register and smem constraints, we can filter configs for the desired
# occupancy. Keep in mind that these are rules of thumb. It's hard to know for
# sure if these lead to the best block sizes.


def find_configs(occupancy, dtype, num_buffers=1):
    dtype_bytes = torch.tensor([], dtype=dtype).element_size()

    # Assume ~1 KB of smem used by mbarriers, compiler-generated code, etc.
    smem = 228 * 1024 // occupancy - 1024

    configs = []
    BLOCK_MNK = [32, 64, 128, 256]
    for BLOCK_M, BLOCK_N, BLOCK_K, num_warps in itertools.product(BLOCK_MNK, BLOCK_MNK, BLOCK_MNK, [4, 8]):
        # Assume ~16 regs per thread of baseline usage.
        regs = 64 * 1024 // occupancy - 16 * num_warps * 32

        a_smem = BLOCK_M * BLOCK_K * dtype_bytes
        b_smem = BLOCK_N * BLOCK_K * dtype_bytes
        acc_smem = BLOCK_M * BLOCK_N * dtype_bytes
        # SMEM for A and B does not coexist with C.
        if max((a_smem + b_smem) * num_buffers, acc_smem) > smem:
            continue

        # The accumulator is the only in-memory tensor in f32.
        acc_regs = BLOCK_M * BLOCK_N
        # Max regs per thread is 256. Being near this can also cause spills.
        if acc_regs // num_warps // 32 >= 256:
            continue
        if acc_regs > regs:
            continue

        instr_shape_n = get_instr_shape_n(BLOCK_M, BLOCK_N, num_warps)
        configs.append((BLOCK_M, BLOCK_N, BLOCK_K, num_warps, instr_shape_n, occupancy))

    def filter_configs(configs, instr_shape_n):
        max_n_configs = [cfg for cfg in configs if cfg[4] == instr_shape_n]
        # Filter for configs with the largest BLOCK_M * BLOCK_K.
        max_block_mk = max(cfg[0] * cfg[2] for cfg in max_n_configs)
        return [cfg for cfg in max_n_configs if cfg[0] * cfg[2] == max_block_mk]

    top_instr_shape_n = sorted({cfg[4] for cfg in configs}, reverse=True)
    result_configs = filter_configs(configs, top_instr_shape_n[0])
    if len(top_instr_shape_n) > 1:
        result_configs += filter_configs(configs, top_instr_shape_n[1])
    return result_configs
```

**EN:** This section explains how to narrow the autotuning space with occupancy-aware heuristics. `find_configs` estimates shared-memory and register pressure for candidate `(BLOCK_M, BLOCK_N, BLOCK_K, num_warps)` tuples, rejects shapes that would exceed hardware limits or likely spill registers, computes the resulting instruction shape, and keeps the strongest candidates with the largest `instr_shape_n` and large `BLOCK_M * BLOCK_K` coverage.

**CN:** 这一部分说明如何用“占用率感知”的启发式方法缩小自动调优空间。`find_configs` 会估算候选 `(BLOCK_M, BLOCK_N, BLOCK_K, num_warps)` 组合的共享内存和寄存器压力，淘汰超出硬件限制或可能产生寄存器溢出的配置，再计算相应的指令形状，并保留 `instr_shape_n` 最大且 `BLOCK_M * BLOCK_K` 覆盖较大的强候选。

### Lines 478-514 / 第 478-514 行

```python
if __name__ == "__main__":
    print("Benchmarking selected configs")
    print("=============================")
    # Just in case, check occupancy 1 configs.
    configs = find_configs(occupancy=1, dtype=torch.float16)
    configs += find_configs(occupancy=2, dtype=torch.float16)
    # Benchmark the configs over a large matmul. Keep in mind that the best
    # hyperparameters can depend on the matmul shapes.
    M, N, K = 8192, 8192, 16 * 1024
    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)
    print("BLOCK_M BLOCK_N BLOCK_K num_warps instr_shape_n occupancy time (ms) tflops/s")
    for BLOCK_M, BLOCK_N, BLOCK_K, num_warps, instr_shape_n, occupancy in configs:
        fn = lambda: blocked_matmul(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, False, num_warps)
        ms = triton.testing.do_bench(fn)
        flops = 2 * M * N * K
        tflops_per_sec = flops * 1e-12 / (ms * 1e-3)
        print(f"{BLOCK_M:>7} {BLOCK_N:>7} {BLOCK_K:>7} {num_warps:>9} {instr_shape_n:>13} "
              f"{occupancy:>9} {ms:>9.2f} {tflops_per_sec:>8.2f}")
    print()

# %%
# ```
# BLOCK_M BLOCK_N BLOCK_K num_warps instr_shape_n occupancy time (ms) tflops/s
#     128     256     256         8           256         1      5.34   412.14
#     256     128     256         8           128         1      5.67   387.74
#      64     256     128         4           256         2      4.64   474.03
#      64     128     256         4           128         2      6.18   355.60
#     128     128     128         4           128         2      4.98   441.88
#     128     128     128         8           128         2      5.79   380.08
# ```
#
# The hypothesis that having occupancy 2 with `BLOCK_N=256` would be the best
# has held over our limited sample of hyperparameters. Autotuning over all
# hyperparameters is an exercise for the reader.

```

**EN:** The baseline benchmark runs the shortlisted non-pipelined kernels on a large GEMM and reports TFLOPS. The tutorial commentary notes that occupancy 2 with `BLOCK_N=256` wins in this sample, giving intuition that a balance of larger WGMMA tiles and enough resident programs is important when TMA and WGMMA are not overlapped.

**CN:** 这里对筛选出的非流水版本 kernel 在大规模 GEMM 上进行基准测试，并输出 TFLOPS。教程中的结论指出，在这组样本里，`BLOCK_N=256` 且 occupancy 为 2 的配置表现最好，说明当 TMA 与 WGMMA 尚未重叠时，需要在更大的 WGMMA tile 与足够的常驻 program 数量之间取得平衡。

### Lines 515-579 / 第 515-579 行

```python
# %%
# 466 TFLOPS is not a bad start. However, we aren't using the fact that WGMMA is
# asynchronous, and we aren't pipelining the TMA loads as shown in previous
# tutorials.
#
# For now, let's keep the loads synchronous and focus on pipelining the WGMMA.
# This requires us to double-buffer the operands, since we will be loading into
# the next set of buffers while WGMMA reads from the previous.


@gluon.jit
def blocked_matmul_pipelined_kernel(a_desc, b_desc, c_desc, num_warps: gl.constexpr):
    BLOCK_M: gl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: gl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: gl.constexpr = a_desc.block_type.shape[1]
    dtype: gl.constexpr = a_desc.dtype
    K = a_desc.shape[1]

    # Allocate 2 buffers for each A and B.
    a_smem = gl.allocate_shared_memory(dtype, [2] + a_desc.block_type.shape, a_desc.layout)
    b_smem = gl.allocate_shared_memory(dtype, [2] + b_desc.block_type.shape, b_desc.layout)
    index = 0

    pid_m = gl.program_id(axis=0)
    pid_n = gl.program_id(axis=1)
    off_m = pid_m * BLOCK_M
    off_n = pid_n * BLOCK_N

    mma_layout: gl.constexpr = pick_wgmma_layout(dtype, BLOCK_M, BLOCK_N, num_warps)
    acc = warpgroup_mma_init(gl.zeros((BLOCK_M, BLOCK_N), dtype=gl.float32, layout=mma_layout))

    bar = gl.allocate_shared_memory(gl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)
    phase = 0

    for k in range(0, K, BLOCK_K):
        a = a_smem.index(index)
        b = b_smem.index(index)

        mbarrier.expect(bar, a_desc.block_type.nbytes + b_desc.block_type.nbytes)
        tma.async_load(a_desc, [off_m, k], bar, a)
        tma.async_load(b_desc, [k, off_n], bar, b)
        mbarrier.wait(bar, phase=phase)
        phase ^= 1

        # Since `warpgroup_mma_wait` is a no-op when there are no WGMMAs in
        # flight, we can overlap the WGMMA by waiting first, then issuing the
        # async WGMMA.
        acc = warpgroup_mma_wait(num_outstanding=0, deps=(acc, ))
        acc = warpgroup_mma(a, b, acc, is_async=True)

        # Move to the next buffer. The TMA load will start while the WGMMA is
        # still running.
        index ^= 1

    # Wait for the last WGMMA to complete.
    acc = warpgroup_mma_wait(num_outstanding=0, deps=(acc, ))

    mbarrier.invalidate(bar)

    c_smem = gl.allocate_shared_memory(dtype, c_desc.block_type.shape, c_desc.layout)
    c_smem.store(acc.to(dtype))
    fence_async_shared()
    tma.async_copy_shared_to_global(c_desc, [off_m, off_n], c_smem)
    tma.store_wait(pendings=0)
```

**EN:** `blocked_matmul_pipelined_kernel` exploits WGMMA asynchrony more directly. It double-buffers A and B in shared memory, initializes the accumulator with `warpgroup_mma_init`, and in each K iteration first waits for previous WGMMAs to retire and then launches the next async WGMMA. Toggling `index` lets the next TMA loads target the other buffer while the current WGMMA is still consuming the previous one, which is the core overlap pattern demonstrated here.

**CN:** `blocked_matmul_pipelined_kernel` 更直接地利用了 WGMMA 的异步特性。它为 A 和 B 在共享内存中建立双缓冲，用 `warpgroup_mma_init` 初始化累加器，并在每个 K 迭代中先等待先前的 WGMMA 退休，再发射新的异步 WGMMA。通过翻转 `index`，下一轮 TMA 加载可以写入另一组缓冲区，而当前 WGMMA 仍在消费上一组缓冲区，这正是这里演示的核心重叠模式。

### Lines 582-607 / 第 582-607 行

```python
def blocked_matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_warps):
    M, N = C.shape

    a_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gl.float16)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], gl.float16)
    c_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], gl.float16)
    a_desc = TensorDescriptor.from_tensor(A, [BLOCK_M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(B, [BLOCK_K, BLOCK_N], b_layout)
    c_desc = TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], c_layout)

    grid = (triton.cdiv(M, BLOCK_M), triton.cdiv(N, BLOCK_N))
    blocked_matmul_pipelined_kernel[grid](a_desc, b_desc, c_desc, num_warps=num_warps)


@pytest.mark.parametrize("M, N, K", [(208, 416, 304), (2000, 1000, 2000)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(64, 64, 64), (128, 128, 128)])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.skipif(not is_hopper(), reason="Requires Hopper")
def test_blocked_matmul_pipelined(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, num_warps):

    A = torch.randn(M, K, device="cuda", dtype=torch.float16)
    B = torch.randn(K, N, device="cuda", dtype=torch.float16)
    C = torch.empty(M, N, device="cuda", dtype=torch.float16)

    blocked_matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_warps)
    torch.testing.assert_close(A @ B, C, rtol=1e-3, atol=1e-1)
```

**EN:** The pipelined wrapper mirrors the baseline version but always assumes non-transposed B and launches the new kernel. The test validates that pipelining preserves correctness for multiple shapes and warp counts by comparing against plain `A @ B`.

**CN:** 流水线版本的包装函数整体上与基线版本相似，但固定采用非转置的 B，并启动新的 kernel。测试通过与普通 `A @ B` 对比，验证在多种尺寸和 warp 数下，引入流水线后依然保持正确性。

### Lines 610-663 / 第 610-663 行

```python
# %%
# Search for another set of configs. Apply simiar principles to prune down the
# potential configs. Our previous best block config will use 160 KB of smem, too
# much for an occupancy of 2, but leaves performance on the table by not using
# the remaining 68 KB. It's likely the best kernel reduces BLOCK_N in favour of
# keeping 2 occupancy.

if __name__ == "__main__":
    print("Benchmarking pipelined matmul")
    print("=============================")
    configs = find_configs(occupancy=1, dtype=torch.float16, num_buffers=2)
    configs += find_configs(occupancy=2, dtype=torch.float16, num_buffers=2)
    # Add our previous best config since it doesn't get selected.
    configs.append([64, 256, 128, 4, 256, 2])

    print("BLOCK_M BLOCK_N BLOCK_K num_warps instr_shape_n occupancy time (ms) tflops/s")
    for BLOCK_M, BLOCK_N, BLOCK_K, num_warps, instr_shape_n, occupancy in configs:
        fn = lambda: blocked_matmul_pipelined(A, B, C, BLOCK_M, BLOCK_N, BLOCK_K, num_warps)
        ms = triton.testing.do_bench(fn)
        flops = 2 * M * N * K
        tflops_per_sec = flops * 1e-12 / (ms * 1e-3)
        print(f"{BLOCK_M:>7} {BLOCK_N:>7} {BLOCK_K:>7} {num_warps:>9} {instr_shape_n:>13} "
              f"{occupancy:>9} {ms:>9.2f} {tflops_per_sec:>8.2f}")
    print()

# %%
# ```
# BLOCK_M BLOCK_N BLOCK_K num_warps instr_shape_n occupancy time (ms) tflops/s
#     128     256     128         8           256         1      5.16   426.06
#     256     128     128         8           128         1      5.70   385.85
#      64     256      64         4           256         2      5.27   417.50
#      64     128     128         4           128         2      5.71   384.98
#     128     128      64         4           128         2      4.44   495.31
#     128     128      64         8           128         2      4.92   446.81
#      64     256     128         4           256         2      6.05   363.36
# ```
#
# We see indeed that the best config ends up with instr_shape_n=128. Note that
# our previous best config is over 100 TFLOPS slower now! Pipelining the WGMMA
# delivers a modest 5% speedup overall, but we had to re-tune the
# hyperparameters.
#
# Pipelining both the async TMA loads and the WGMMA is left as an exercise to
# the reader.
#
# Main takeaways:
#
# - WGMMA is a Hopper-specific instruction that performs block-level MMA.
# - WGMMA is asynchronous and can be overlapped with other operations.
# - WGMMA has a bunch of restrictions on its layout.
# - LHS operand can be in shared memory or registers.
# - WGMMA can handle transposed inputs, and we can create transposed views.
# - Pipelining the WGMMA leads to better performance by enabling overlap.
# - Hyperparameter tuning is critical for performance.
```

**EN:** The final benchmark searches a new config set under double-buffered shared-memory usage and compares pipelined performance. The reported best result shifts to `instr_shape_n=128`, and the comments emphasize an important lesson: once overlap changes the resource/performance trade-off, hyperparameters must be retuned. The closing bullets summarize the tutorial’s main WGMMA rules and performance takeaways.

**CN:** 最后一段基准测试在双缓冲共享内存前提下重新搜索配置，并比较流水线版本的性能。最佳结果转移到了 `instr_shape_n=128`，这也说明了一个关键经验：一旦重叠执行改变了资源与性能之间的权衡关系，就必须重新调参。结尾的要点列表总结了本教程关于 WGMMA 规则和性能优化的主要结论。

## Key Concepts / 关键概念

- **WGMMA instruction model / WGMMA 指令模型**: WGMMA is Hopper’s warp-group matrix multiply-accumulate path; Gluon exposes it through `warpgroup_mma`, with strict operand-layout requirements. / WGMMA 是 Hopper 的 warp-group 级矩阵乘加路径；Gluon 通过 `warpgroup_mma` 暴露它，并要求严格的操作数布局。
- **TensorDescriptor + TMA / TensorDescriptor 与 TMA**: `TensorDescriptor` captures tensor shape, block shape, and layout so `tma.async_load` and `tma.async_copy_shared_to_global` can move block tiles efficiently between global and shared memory. / `TensorDescriptor` 封装张量形状、block 形状和布局，使 `tma.async_load` 与 `tma.async_copy_shared_to_global` 能高效搬运全局内存与共享内存之间的 tile。
- **Layout-driven code generation / 布局驱动的代码生成**: `NVMMASharedLayout`, `NVMMADistributedLayout`, and `DotOperandLayout` are not cosmetic—they determine how operands are placed and how Triton lowers the operation to valid Hopper WGMMA instructions. / `NVMMASharedLayout`、`NVMMADistributedLayout` 和 `DotOperandLayout` 不是装饰性的，它们直接决定操作数摆放方式以及 Triton 如何将操作降级为合法的 Hopper WGMMA 指令。
- **Async ordering / 异步顺序控制**: `mbarrier`, `warpgroup_mma_wait`, and `fence_async_shared` together define when TMA loads are visible, when WGMMA results may be consumed, and when shared-memory writes are safe to hand off to later async operations. / `mbarrier`、`warpgroup_mma_wait` 与 `fence_async_shared` 共同定义了 TMA 何时可见、WGMMA 结果何时可消费，以及共享内存写入何时可以安全交给后续异步操作。
- **Performance tuning / 性能调优**: Instruction shape `n`, block sizes, warp count, occupancy, and pipelining all interact. The tutorial intentionally shows that the best parameters for a simple kernel are not necessarily best once overlap is introduced. / 指令形状 `n`、block 大小、warp 数、occupancy 和流水线化相互影响。教程特意展示了：在简单 kernel 中最优的参数，引入重叠执行后不一定仍然最优。

## Dependencies / 依赖关系

- **`torch`**: creates CUDA tensors, provides dtype/element-size info, computes PyTorch reference GEMMs, and performs correctness checks with `torch.testing.assert_close`. / 创建 CUDA 张量，提供 dtype 与元素大小信息，计算 PyTorch 参考 GEMM，并用 `torch.testing.assert_close` 做正确性检查。
- **`triton`**: supplies launch syntax, `cdiv`, benchmarking via `triton.testing.do_bench`, and runtime target detection. / 提供 kernel 启动语法、`cdiv`、`triton.testing.do_bench` 基准测试以及运行时目标检测。
- **`triton.experimental.gluon` + `language as gl`**: provide the JIT/constexpr system, tensor layouts, shared-memory allocation, program IDs, and type-level metadata used throughout the kernels. / 提供 JIT/constexpr 体系、张量布局、共享内存分配、program ID 以及 kernel 中广泛使用的类型级元数据。
- **Hopper-specific Gluon primitives** (`TensorDescriptor`, `tma`, `mbarrier`, `warpgroup_mma_init`, `warpgroup_mma`, `warpgroup_mma_wait`, `fence_async_shared`): these are the core building blocks for Hopper TMA transfers and asynchronous WGMMA execution. / Hopper 专用的 Gluon 原语（`TensorDescriptor`、`tma`、`mbarrier`、`warpgroup_mma_init`、`warpgroup_mma`、`warpgroup_mma_wait`、`fence_async_shared`）是 Hopper TMA 传输和异步 WGMMA 执行的核心构件。
- **`pytest` and `itertools`**: support parameterized tests and Cartesian-product benchmark sweeps. / 用于参数化测试以及基准测试中的笛卡尔积枚举。
