# 14-multicta.py — Code Analysis / 代码分析
## Source / 来源
- Source file: `/root/xw/triton/python/tutorials/gluon/14-multicta.py`
- Purpose: Explores multi-CTA programming in Triton Gluon: CGA-aware softmax, 2CTA tcgen05 MMA, TMA multicast, and a warp-specialized multi-CTA matmul with CLC.
- 源文件：`/root/xw/triton/python/tutorials/gluon/14-multicta.py`
- 作用：Explores multi-CTA programming in Triton Gluon: CGA-aware softmax, 2CTA tcgen05 MMA, TMA multicast, and a warp-specialized multi-CTA matmul with CLC.

## Line-by-Line Analysis / 逐行分析
### Lines 1-124 — CGA motivation, imports, capability guards, and small tuning helpers
```python
"""
Multi-CTA
=========

In Hopper, NVIDIA added a new thread group level to the hierarchy, the CGA.
A CGA is a group of up to 16 CTAs that may collaborate with each other.
In particular they can:

- Load data from HBM collaboratively via TMA broadcasting
- Exchange data by accessing each other's shared memory. This is often called
  "using distributed shared memory"
- Starting in Blackwell, pairs CTA can collaboratively compute the result of a
  matrix multiplication
- Subsets of the CGA cluster can be selectively synchronized via `mbarrier`s

Of course, different CTAs may or may not be allocated to the same SM (in fact
the documentation does not provide any guarantees about this) so operations like
synchronisation or accessing each other's shared memory are **much** more costly
than accessing shared memory or synchronising the threads within a single CTA.
As such, when using CGAs, the name of the game is to maximise the collaboration
while not introducing unnecessary synchronisation points.

Multi-CTA Layouts
-----------------

Layouts can be sharded across CTAs in a natural way. For example, we can have a
blocked layout on a program with 4 warps and 2 CTAs of the form:

```
gl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0], cga_layout=[[1, 0]])
```

The `cga_layout` representation `[[1, 0]]` denotes a linear layout. In this case, it
denotes that the two CTAs are sharding the tensor along the 0th dimension into two
contiguous subtensors.

Similarly, if we had 8 CTAs and we wanted to shard a shared memory descriptor across
the 0th dimension using the first 4 CTAs and then across the 1st dimension using the
last 4 CTAs, the layout could look like

```python
gl.NVMMASharedLayout.get_default_for([M, N], gl.float16, cga_layout=[[1, 0], [2, 0], [0, 1]])
```

The `cga_layout` will always have `log2(numCTAs)` bases, and it will always denote sharding
the full tensor into contiguous chunks. For more sharding patterns where the CTAs may not
shard the tensor into contiguous subtensors, like

| CTA0 warp0 |
| CTA1 warp0 |
| CTA0 warp1 |
| CTA1 warp1 |

one may use the layouts `LinearEncoding` for data in registers and `SharedLinearEncoding`
for data in shared memory. In these cases, rather than having an attribute called `CGALayout`
the CGA layout is encoded as part of the `LinearLayout` under the input dimension named `block`.
The example above would then look like:

```python
gl.LinearEncoding(warps=[[2]], block=[[1]])
```

as we shard first along the CTAs and then along the warps.
"""

import importlib
import pytest
import torch
import triton

from triton.experimental import gluon
from triton.experimental.gluon import language as gl
from triton.experimental.gluon.language.nvidia.blackwell import (
    TensorMemoryLayout,
    allocate_tensor_memory,
    clc,
    tcgen05_commit,
    tcgen05_mma,
    tcgen05_mma_barrier_count,
    tensor_memory_descriptor,
)
from triton.experimental.gluon.language.nvidia.hopper import mbarrier, tma
from triton.experimental.gluon.nvidia.hopper import TensorDescriptor

# Re-use baseline tutorials for comparisons.
t8 = importlib.import_module("08-warp-specialization")


def is_hopper_or_newer():
    if not torch.cuda.is_available():
        return False
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] >= 9


def is_blackwell():
    if not torch.cuda.is_available():
        return False
    target = triton.runtime.driver.active.get_current_target()
    return target.backend == "cuda" and torch.cuda.get_device_capability()[0] == 10


if __name__ == "__main__" and not is_blackwell():
    raise RuntimeError("This tutorial requires a Blackwell NVIDIA GPU")


def tflops(ms, M, N, K):
    return 2 * M * N * K * 1e-12 / (ms * 1e-3)


def gbps(ms, num_bytes):
    return num_bytes * 1e-9 / (ms * 1e-3)


def pick_multicta_softmax_config(n_cols):
    warp_thresholds = [(3072, 1), (6144, 2)]
    cluster_thresholds = [(16 * 1024, 1), (32 * 1024, 2), (64 * 1024, 4), (128 * 1024, 8)]

    num_warps = next((v for limit, v in warp_thresholds if n_cols <= limit), 4)
    cluster_n = next((v for limit, v in cluster_thresholds if n_cols <= limit), 16)
    return {
        "num_warps": num_warps,
        "num_ctas": cluster_n,
    }
```
**EN:** The opening docstring frames CGA as Hopper/Blackwell’s extra hierarchy level above the CTA, emphasizing both its collaborative power and its synchronization cost. The helper functions that follow (`tflops`, `gbps`, `pick_multicta_softmax_config`) are simple, but they set up the first lesson: launch configuration matters once a kernel is intentionally spread across multiple CTAs.

**CN:** 开头的文档字符串把 CGA 描述成 Hopper/Blackwell 在 CTA 之上的额外层级，同时强调了它既能带来协作能力，也会引入更高的同步成本。后面的几个小辅助函数（`tflops`、`gbps`、`pick_multicta_softmax_config`）虽然简单，但为第一个例子做好了铺垫：当一个 kernel 被有意拆到多个 CTA 上执行时，启动配置会直接影响性能。
### Lines 127-244 — Multi-CTA softmax kernel, wrapper, test, and benchmark output
```python
# %%
# A multi-CTA kernel is launched with `num_ctas > 1` where num_ctas is a power of two
# and `num_ctas <= 16`.
#
# Layout-driven operations such as `gl.convert_layout`, `gl.reduce` and `gl.sum` use
# clusters automatically when the source and destination layouts shard the CTA dimension
# differently.
#
# The kernel below shards one row across multiple CTAs and uses the automatic
# cross-CTA reductions in `gl.max` and `gl.sum` to compute a numerically stable
# row-wise softmax.
#
# Without CGAs, we would need to switch to an iterative reduction or a
# multi-kernel approach once the row becomes too wide for a single CTA.


@gluon.jit
def multicta_softmax_kernel(
    x_ptr,
    out_ptr,
    x_row_stride,
    out_row_stride,
    BLOCK_N: gl.constexpr,
):
    pid = gl.program_id(0)
    cga_layout: gl.constexpr = ((1, ), (2, ), (4, ), (8, ), (16, ))[:gl.num_ctas().bit_length() - 1]
    layout: gl.constexpr = gl.BlockedLayout([4], [32], [gl.num_warps()], [0], cga_layout=cga_layout)
    offs_n = gl.arange(0, BLOCK_N, layout)
    mask = offs_n < BLOCK_N
    row_start = pid * x_row_stride
    out_row_start = pid * out_row_stride
    x = gl.load(x_ptr + row_start + offs_n, mask=mask, other=-float("inf"))
    row_max = gl.max(x, axis=0)
    y = gl.exp(x - row_max)
    row_sum = gl.sum(y, axis=0)
    z = y * (1.0 / row_sum)
    gl.store(out_ptr + out_row_start + offs_n, z, mask=mask)


def multicta_softmax_f32(x, out=None):
    M, N = x.shape
    cfg = pick_multicta_softmax_config(N)
    if out is None:
        out = torch.empty_like(x)

    multicta_softmax_kernel[(M, )](
        x,
        out,
        x.stride(0),
        out.stride(0),
        BLOCK_N=N,
        num_warps=cfg["num_warps"],
        num_ctas=cfg["num_ctas"],
    )
    return out


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper or newer")
@pytest.mark.parametrize("M, N", [(64, 64), (64, 256), (16, 2**16)])
def test_multicta_softmax_f32(M, N):
    x = torch.randn((M, N), device="cuda", dtype=torch.float32)
    out = multicta_softmax_f32(x)
    ref = torch.softmax(x, dim=1)
    torch.testing.assert_close(out, ref, atol=1e-5, rtol=1e-5)


def benchmark_multicta_softmax_f32():
    if not is_hopper_or_newer():
        raise RuntimeError("softmax benchmark requires Hopper or newer")

    SOFTMAX_BENCH_SHAPES = [
        (2**15, 2**8),
        (2**15, 2**9),
        (2**15, 2**10),
        (2**15, 2**11),
        (2**15, 2**12),
        (2**15, 2**13),
        (2**15, 2**14),
        (2**15, 2**15),
        (2**15, 2**16),
        (2**14, 2**17),
        (2**13, 2**18),
    ]
    print("Benchmarking multicta_softmax")
    print("============================")
    print("  shape         CTAs  warps  time (ms)  bandwidth (GB/s)")
    for M, N in SOFTMAX_BENCH_SHAPES:
        cfg = pick_multicta_softmax_config(N)
        x = torch.empty((M, N), device="cuda", dtype=torch.float32).uniform_(-1, 1)
        out = torch.empty_like(x)
        ms = triton.testing.do_bench_cudagraph(lambda: multicta_softmax_f32(x, out))
        num_bytes = 2 * x.numel() * x.element_size()
        print(f"{M:>6} x {N:<6}  {cfg['num_ctas']:>4}  {cfg['num_warps']:>5}  {ms:>9.3f}  {gbps(ms, num_bytes):>16.2f}")


benchmark_multicta_softmax_f32()

# %%
# Softmax benchmark results
# ```text
# Benchmarking multicta_softmax
# ============================
#   shape         CTAs  warps  time (ms)  bandwidth (GB/s)
#  32768 x 256        1      1      0.018           3661.46
#  32768 x 512        1      1      0.020           6746.45
#  32768 x 1024       1      1      0.040           6740.50
#  32768 x 2048       1      1      0.078           6920.01
#  32768 x 4096       1      2      0.152           7065.25
#  32768 x 8192       1      4      0.301           7136.76
#  32768 x 16384      1      4      0.600           7157.74
#  32768 x 32768      2      4      1.312           6545.11
#  32768 x 65536      4      4      2.836           6057.26
#  16384 x 131072     8      4      3.142           5468.66
#   8192 x 262144    16      4      3.627           4736.15
# ```
#
# We see that here using multiCTA we are able to get very good performance across the board.
#
```
**EN:** `multicta_softmax_kernel` builds a `BlockedLayout` whose `cga_layout` shards one logical row across multiple CTAs, then relies on layout-aware `gl.max` and `gl.sum` to do cross-CTA reductions for numerically stable softmax. The wrapper chooses CTA and warp counts from the row width, while the test and benchmark show when a single CTA stops scaling and clustered execution becomes worthwhile.

**CN:** `multicta_softmax_kernel` 构造了一个带 `cga_layout` 的 `BlockedLayout`，把一整行逻辑数据拆到多个 CTA 上，再利用具备布局感知能力的 `gl.max` 和 `gl.sum` 完成跨 CTA reduction，从而实现数值稳定的 softmax。包装函数根据行宽选择 CTA 数和 warp 数，后面的测试与 benchmark 则展示了单 CTA 何时开始失去扩展性、而 clustered 执行何时开始占优。
### Lines 246-319 — Discussion of cross-CTA synchronization rules
```python
# Multi-CTA synchronization
# -------------------------
#
# Since CTAs may be on different SMs, sychronization is much slower than
# within a CTA. As such, gluon provides a rather conservative automatic
# synchronization guarantee, and the user is responsible for the rest of
# the synchronization.
#
# Gluon will place synchronisaton primitives between operations like gl.convert_layout,
# gl.reduce and gl.sum when the source and destination layouts shard the CTA dimension
# differently. All the other operations like TMA, WGMMA, TCGen5MMA, etc. should be
# synchronized by the kernel writer via mbarriers, same as it's done for single-CTA kernels.
#
# The semantics of the cga_layout for a multi-CTA mbarrier are slightly different:
# As discussed in 02-layouts.py, a linear layout represents a map from F_2^n to F_2^m.
# In this case, the cga_layout is a map from the numCTAs (which is a power of two) to
# the number of barriers it represents. For example, we could have a mbarrier layout
# where each CTA has its own barrier.
#
# ```python
# num_ctas: gl.constexpr = 4
# bar = gl.allocate_shared_memory(gl.int64, [num_ctas], MBarrierLayout(cga_layout=[[1], [2], [4]]))
# ```
#
# So, we define the cga_layout matrix by its columns (in binary), which represents the 3x3 identity matrix.
# Since this pattern is so common, gluon provides a helper function to create it:
#
# ```python
# bar = mbarrier.allocate_mbarrier()
# ```
#
# Now, barrier layouts also allow for cross-CTA synchronization. For example, we an define a 2-CTA mbarrier
# for an 8CTA kernel as:
# ```python
# bar = gl.allocate_shared_memory(gl.int64, [4], MBarrierLayout(cga_layout=[[0], [1], [2]]))
# ```
#
# Note that now the non-zero bases are just [1] and [2], so there are just 2**2 = 4 barriers.
# Since it's an 8 CTA kernel, there are 2**3 = 8 bases though.
# The layout now has broadcasting on the 0th column. What that means is that any CTA that just differs on
# the 0th bit will share a barrier. For example, CTA0 and CTA1 will share a barrier, CTA2 and CTA3 will
# and so on. The lead CTA is the smallest CTA id in the group. For this layout, the even CTA IDs are the
# lead CTAs.
#
# In general, an mbarrier cta_layout is a sequence `[[2**i] for i in range(k)]` for `k <= log2(num_ctas)`
# with `log2(num_ctas) - k` zeros interleaved.
#
# All the operations that act on barriers generalize naturally to multi-CTA barriers. More explicitly:
# - `mbarrier.init` multiplies the count argument by the number of CTAs in the group
#                   and it's only initialized on the lead CTA
# - `mbarrier.expect` multiplies the size_per_cta argument by the number of CTAs in the group and it's
#                     only expected on the lead CTA
#                     since an expect counts as one arrival, all the non-lead CTAs will also emit one arrival
#                     to the lead CTA.
# - `mbarrier.arrive` every CTA in a group arrives on the lead CTA
# - `mbarrier.wait` just the lead CTA waits for the barrier
#
# Barriers used across CTAs need one extra ordering rule: every relevant
# `mbarrier.init` must complete before any CTA uses that barrier. This applies
# both to barriers that are themselves cross-CTA and to per-CTA barriers consumed
# by multicast or 2CTA ops. The compiler inserts the required
# `fence.mbarrier_init.release.cluster` plus a relaxed cluster barrier after the
# top-level init sequence and before the first use. Since cluster barriers must
# execute outside `warp_specialize`, initialize these barriers in the top-level
# part of the kernel before entering warp specialization.
#
# Final note on synchronization.
# cluster.arrive / cluster.wait (i.e., CGA barriers, the cluster equivalent of bar.sync for CTAs) must be
# executed by all threads in the kernel. As a result, they cannot be used inside a warp_specialize block.
#
# Moreover, operations such as convert_layout, reduce, sum, max, etc., emit CGA barriers when they cross CTAs.
# Therefore, these operations are also not allowed inside a warp_specialize block whenever they may span multiple
# CTAs.
#
```
**EN:** This prose block is essential context for the rest of the file: it explains when Gluon inserts CGA barriers automatically and when the kernel author must use explicit multi-CTA `mbarrier` logic. It also clarifies the subtle rule that barrier initialization for cluster-visible barriers must happen before entering `warp_specialize`, because the compiler needs to place a cluster-wide init fence.

**CN:** 这一段文字对后面的所有例子都非常关键：它解释了 Gluon 什么时候会自动插入 CGA barrier，什么时候则必须由 kernel 作者显式编写多 CTA `mbarrier` 同步逻辑。同时它也说明了一个细节：凡是 cluster 可见的 barrier，都必须在进入 `warp_specialize` 之前完成初始化，因为编译器需要在这里插入 cluster 级别的 init fence。
### Lines 320-434 — 2CTA tcgen05 MMA pattern and single-tile demo kernel
```python
# 2CTA TCGen5MMA
# --------------
#
# In 2CTA mode, the tcgen05_mma instruction uses data from every other CTA
# in a pair (i.e. CTA0 and CTA1, CTA2 and CTA3, etc.) to compute the result.
# In mathematical terms, it computes the outer product of the two operands,
# where the LHS holds the input sharded along the M dimension and the RHS holds
# the input sharded along the N dimension.
# In terms of cga_layouts, the LHS has its first basis equal to (1, 0) and the
# RHS has its first basis equal to (0, 1).
# The accumulator is also shared as (1, 0)
#
# Same as for single-CTA, the blockM shape of `TensorMemoryLayout` is the shape
# of the instruction, which can be either 64 or 128.
#
# The outer-product layouts above describe the required data placement: LHS
# `(1, 0)`, RHS `(0, 1)`, and accumulator/output `(1, 0)`. On top of that,
# enabling 2CTA has two required code changes, plus a third when waiting on MMA completion:
# - select 2CTA mode on the accumulator with
#   `TensorMemoryLayout(..., two_ctas=True)`;
# - create any barrier that must be waited on before the lead CTA issues
#   `tcgen05_mma` with
#   `mbarrier.allocate_mbarrier(..., two_ctas=True)`;
# - when initializing an mbarrier used to wait on MMA completion, pass
#   `two_ctas=...` to `tcgen05_mma_barrier_count`.
#
# Such a pre-MMA barrier needs `two_ctas=True` so that the lead CTA waits for
# both rows before issuing the collective op. In this single-tile example that is
# `tma_bar`; in the pipelined matmul below the corresponding examples are
# `load_ready_bars` and `acc_empty_bars`. These are the same 2CTA sites used by
# `03-matmul-multicta.py`.
#
# The `mma_bar` itself does not need `two_ctas=True`: `tcgen05_mma` multicasts
# its completion signal to the two CTAs in the pair. Once one `tcgen05_mma` in
# a kernel uses 2CTA mode, all `tcgen05_mma` instructions in that kernel must
# use 2CTA mode.
#
# The kernel `two_cta_tcgen05_kernel` shows the 2CTA TCGen5MMA pattern on a single tile.
#
# It's worth noting that the pattern changes a bit once the TMA has to wait on the tcgen05_mma
# We will cover this in the next section.


@gluon.jit
def two_cta_tcgen05_kernel(a_desc, b_desc, c_desc):
    gl.static_assert(gl.num_ctas() == 2)

    cluster_m: gl.constexpr = a_desc.block_shape[0]
    tile_n: gl.constexpr = b_desc.block_shape[1]
    cta_m: gl.constexpr = cluster_m // 2
    cga_layout: gl.constexpr = c_desc.layout.cga_layout

    smem_a = gl.allocate_shared_memory(a_desc.dtype, a_desc.block_shape, a_desc.layout)
    smem_b = gl.allocate_shared_memory(b_desc.dtype, b_desc.block_shape, b_desc.layout)

    tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
    mma_bar = mbarrier.allocate_mbarrier()
    mbarrier.init(tma_bar, count=1)
    mbarrier.init(mma_bar, count=1)

    mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
    tma.async_load(a_desc, [0, 0], tma_bar, smem_a)
    tma.async_load(b_desc, [0, 0], tma_bar, smem_b)
    mbarrier.wait(tma_bar, phase=0, deps=[smem_a, smem_b])
    mbarrier.invalidate(tma_bar)

    acc_layout: gl.constexpr = TensorMemoryLayout(
        block=(cta_m, tile_n),
        col_stride=1,
        cga_layout=cga_layout,
        two_ctas=True,
    )
    acc = allocate_tensor_memory(gl.float32, [cluster_m, tile_n], acc_layout)

    tcgen05_mma(smem_a, smem_b, acc, use_acc=False, mbarriers=[mma_bar])
    mbarrier.wait(mma_bar, phase=0, deps=[smem_a, smem_b])
    mbarrier.invalidate(mma_bar)

    c_smem = gl.allocate_shared_memory(c_desc.dtype, c_desc.block_shape, c_desc.layout)
    c_smem.store(acc.load().to(c_desc.dtype))
    tma.async_copy_shared_to_global(c_desc, [0, 0], c_smem)


def run_two_cta_tcgen05(a, b, c):
    M, N, K = a.shape[0], b.shape[1], a.shape[1]
    a_layout = gl.NVMMASharedLayout.get_default_for([M, K], gl.float16, cga_layout=[(1, 0)])
    b_layout = gl.NVMMASharedLayout.get_default_for([K, N], gl.float16, cga_layout=[(0, 1)])
    c_layout = gl.NVMMASharedLayout.get_default_for([M, N], gl.float16, cga_layout=[(1, 0)])

    a_desc = TensorDescriptor.from_tensor(a, [M, K], a_layout)
    b_desc = TensorDescriptor.from_tensor(b, [K, N], b_layout)
    c_desc = TensorDescriptor.from_tensor(c, [M, N], c_layout)

    two_cta_tcgen05_kernel[(1, )](a_desc, b_desc, c_desc, num_warps=4, num_ctas=2)


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_two_cta_tcgen05():
    M, N, K = 256, 128, 64
    a = torch.randn((M, K), device="cuda", dtype=torch.float16)
    b = torch.randn((K, N), device="cuda", dtype=torch.float16)
    c = torch.empty((M, N), device="cuda", dtype=torch.float16)

    run_two_cta_tcgen05(a, b, c)
    torch.testing.assert_close(c, torch.matmul(a, b), atol=1e-1, rtol=1e-2)


# %%
# Relative to the single-CTA case, the code below therefore changes the operand
# layouts, uses `two_ctas=True` on the accumulator layout, and uses
# `two_ctas=True` on the TMA hand-off barrier. The `mbarrier.expect` byte count
# remains per CTA, not for the whole pair.
#
# Note that there will be a few more changes once this is used in a for-loop and/or with TMA with multicast.
# More on this in the next section.
```
**EN:** The comments first explain the data-layout implications of 2CTA mode: A is sharded along M, B along N, and the accumulator uses `TensorMemoryLayout(..., two_ctas=True)` so paired CTAs jointly produce one result tile. `two_cta_tcgen05_kernel` then shows the minimum legal sequence—TMA-load both operands, wait on a two-CTA handoff barrier, issue `tcgen05_mma`, and materialize the TMEM accumulator back to global memory.

**CN:** 注释部分先解释了 2CTA 模式对数据布局的要求：A 沿 M 维分片，B 沿 N 维分片，而 accumulator 通过 `TensorMemoryLayout(..., two_ctas=True)` 让成对 CTA 共同生成一个结果 tile。`two_cta_tcgen05_kernel` 则给出了最小合法执行序列：先把两个操作数通过 TMA 载入，等待一个 two-CTA handoff barrier，再发射 `tcgen05_mma`，最后把 TMEM accumulator 物化回全局内存。
### Lines 437-537 — Broadcast `cga_layout` derivation and TMA multicast copy example
```python
# TMA with multicast
# ------------------
#
# Since Hopper onwards, TMA has the ability to multicast data to multiple CTAs.
# This is useful in multi-CTA kernels in Hopper as wgmma does not have a 2CTA mode,
# or on Blackwell+ kernels when using more than 2 CTAs.
#
# In this case, for a `cga_layout` for the accumulator, we may compute the layouts
# for A and B as follows:

# Example cga_layout
cga_layout = [(1, 0), (2, 0), (0, 1)]


def get_cga_layout(layout, op_idx, two_ctas):
    assert op_idx in (0, 1)
    if not layout:
        return layout

    # Broadcast along K (the reduction dimension)
    # We multiply by 2 for op_idx == 1, as we have added the (0, 1) basis.
    def broadcast(b):
        mul = 2 if two_ctas else 1
        return (b[0], 0) if op_idx == 0 else (0, mul * b[1])

    if not two_ctas:
        return tuple(map(broadcast, layout))

    # 2CTA performs an outer product so bases are [1, 0] and [0, 1]
    assert layout[0] == (1, 0)
    first = (1, 0) if op_idx == 0 else (0, 1)
    return (first, *map(broadcast, layout[1:]))


cga_layout_a = get_cga_layout(cga_layout, 0, two_ctas=False)
cga_layout_b = get_cga_layout(cga_layout, 1, two_ctas=False)

# %%
# In other words, the cga_layout of A and B is that of C zeroing out the inner dimension
# for each.
#
# This means that some bases are zero for A and/or B, so different CTAs will load the same data.
# Multicast will allow these CTAs to hit the L2 cache efficiently.
#
# The broadcast layouts above describe which CTAs should receive the same tile.
# After that, the pipelined matmul has three `multicast=True` sites:
# - `tma.async_load(..., multicast=True)` sends the tile to every CTA in the
#   broadcast group;
# - `tcgen05_mma(..., multicast=True)` multicasts its mbarrier arrival before
#   the next TMA overwrites the shared-memory tile;
# - `tcgen05_mma_barrier_count(..., multicast=True, ...)` initializes that
#   mbarrier with the matching arrival count.
#
# The TMA synchronization pattern itself is otherwise the same as for a regular
# load: initialize a barrier, `expect` the byte count, issue the TMA, then wait
# on the barrier.
#
# The reason this works is that the TMA instruction broadcasts its arrival to
# every CTA in the multicast group atomically, so the wait side does not need a
# different API.
#
# The TMA destination must use a broadcast `cga_layout`, so that both CTAs
# receive the same shared-memory tile. The barrier stays a regular 1D TMA
# barrier unless the kernel is in 2CTA mode.
#
# The example below keeps things intentionally simple: it multicasts one tile
# into shared memory and then materializes that same tile back to global memory.


@gluon.jit
def tma_multicast_copy_kernel(in_desc, out_desc):
    gl.static_assert(gl.num_ctas() == 2)

    smem = gl.allocate_shared_memory(in_desc.dtype, in_desc.block_shape, in_desc.layout)
    # This kernel is not in 2CTA mode, so the TMA barrier is per-CTA.
    bar = mbarrier.allocate_mbarrier()
    mbarrier.init(bar, count=1)

    mbarrier.expect(bar, in_desc.nbytes_per_cta)
    tma.async_load(in_desc, [0, 0], bar, smem, multicast=True)
    mbarrier.wait(bar, phase=0, deps=[smem])

    tma.async_copy_shared_to_global(out_desc, [0, 0], smem)


def run_tma_multicast_copy(inp, out):
    layout = gl.NVMMASharedLayout.get_default_for(inp.shape, gl.float16, cga_layout=[[0, 0]])
    in_desc = TensorDescriptor.from_tensor(inp, inp.shape, layout)
    out_desc = TensorDescriptor.from_tensor(out, inp.shape, layout)

    tma_multicast_copy_kernel[(1, )](in_desc, out_desc, num_warps=4, num_ctas=2)


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper or newer")
def test_tma_multicast_copy():
    M, N = 128, 128
    inp = torch.randn((M, N), device="cuda", dtype=torch.float16)
    out = torch.empty_like(inp)

    run_tma_multicast_copy(inp, out)
    torch.testing.assert_close(out, inp, atol=0, rtol=0)
```
**EN:** `get_cga_layout` derives operand broadcast layouts from the accumulator layout by zeroing the reduction dimension appropriately, and its `two_ctas` branch preserves the outer-product interpretation required by paired CTAs. `tma_multicast_copy_kernel` then demonstrates the simplest multicast use case: one TMA transaction populates the same shared-memory tile for multiple CTAs, and each CTA can materialize that tile back to memory independently.

**CN:** `get_cga_layout` 通过在适当维度上清零 reduction 方向，从 accumulator 的 `cga_layout` 推导出操作数的广播布局；而它的 `two_ctas` 分支则保留了成对 CTA 需要的 outer-product 语义。`tma_multicast_copy_kernel` 接着给出最简单的 multicast 用法：一次 TMA 事务把同一个 shared-memory tile 同时分发给多个 CTA，而每个 CTA 都可以独立地把这个 tile 重新写回内存。
### Lines 541-645 — Generic TMA→tcgen05 pipeline with multicast and 2CTA-aware barrier counts
```python
# TMA into MMA in a loop
# ----------------------
#
# Here we illustrate the fully generic approach to mixing TMA with (or without) multicast
# into a tcgen05_mma pipeline.
#
# In this case, the tcgen05_mma instruction needs to wait for all the CTAs in its multicast
# group to complete before it can continue the next iteration, as otherwise the next iteration's
# TMA loads will overwrite the data from the previous iteration before it has finished consuming it.
#
# As such, in this case, we need to use tcgen05_mma_barrier_count to compute the number of CTAs
# in a multicast group. Similarly we set the `multicast=True` flag on the tcgen05_mma instruction
# to note that it will have to wait for the multicast group to complete before it can continue.
#
# These functions are generic, so a pattern of this form would work also for non-multicast kernels
# or non-2CTA kernels.


@gluon.jit
def tma_tcgen05_kernel(a_desc, b_desc, out_desc, NUM_K_TILES: gl.constexpr, acc_tmem_layout: gl.constexpr):
    block_m: gl.constexpr = a_desc.block_shape[0]
    block_k: gl.constexpr = a_desc.block_shape[1]
    block_n: gl.constexpr = b_desc.block_shape[1]

    smem_a = gl.allocate_shared_memory(a_desc.dtype, a_desc.block_shape, a_desc.layout)
    smem_b = gl.allocate_shared_memory(b_desc.dtype, b_desc.block_shape, b_desc.layout)

    acc_tmem = allocate_tensor_memory(gl.float32, [block_m, block_n], acc_tmem_layout)
    tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
    mma_bar = mbarrier.allocate_mbarrier()
    mbarrier.init(tma_bar, count=1)
    mbarrier.init(
        mma_bar, count=tcgen05_mma_barrier_count([smem_a, smem_b], multicast=True,
                                                 two_ctas=acc_tmem.type.layout.two_ctas))

    phase_tma = 0
    phase_mma = 0

    for k in range(NUM_K_TILES):
        mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, k * block_k], tma_bar, smem_a, multicast=True)
        tma.async_load(b_desc, [k * block_k, 0], tma_bar, smem_b, multicast=True)
        mbarrier.wait(tma_bar, phase=phase_tma, deps=[smem_a, smem_b])
        phase_tma ^= 1

        tcgen05_mma(smem_a, smem_b, acc_tmem, use_acc=(k != 0), multicast=True, mbarriers=[mma_bar])
        mbarrier.wait(mma_bar, phase=phase_mma, deps=[smem_a, smem_b])
        phase_mma ^= 1

    mbarrier.invalidate(tma_bar)
    mbarrier.invalidate(mma_bar)

    out_smem = gl.allocate_shared_memory(out_desc.dtype, out_desc.block_shape, out_desc.layout)
    out_smem.store(acc_tmem.load().to(out_desc.dtype))
    tma.async_copy_shared_to_global(out_desc, [0, 0], out_smem)


def tma_tcgen05_example(a, b):
    BLOCK_M = 512
    BLOCK_N = 128
    BLOCK_K = 64
    NUM_K_TILES = 2
    cga_layout_a = ((1, 0), (2, 0))
    cga_layout_b = ((0, 1), (0, 0))
    cga_layout_c = ((1, 0), (2, 0))

    M, K = a.shape
    Kb, N = b.shape
    if K != Kb:
        raise ValueError(f"inner dimensions must match, got {K} and {Kb}")
    if M != BLOCK_M or N != BLOCK_N or K != BLOCK_K * NUM_K_TILES:
        raise ValueError(f"expected shapes {(BLOCK_M, BLOCK_K * NUM_K_TILES)} x "
                         f"{(BLOCK_K * NUM_K_TILES, BLOCK_N)}, got {tuple(a.shape)} x {tuple(b.shape)}")

    out = torch.empty((M, N), device="cuda", dtype=torch.float16)
    a_layout = gl.NVMMASharedLayout.get_default_for([M, BLOCK_K], gl.float16, cga_layout=cga_layout_a)
    b_layout = gl.NVMMASharedLayout.get_default_for([BLOCK_K, N], gl.float16, cga_layout=cga_layout_b)
    c_layout = gl.NVMMASharedLayout.get_default_for([M, N], gl.float16, cga_layout=cga_layout_c)
    acc_tmem_layout = TensorMemoryLayout(block=(128, N), col_stride=1, cga_layout=cga_layout_c, two_ctas=True)
    a_desc = TensorDescriptor.from_tensor(a, [M, BLOCK_K], a_layout)
    b_desc = TensorDescriptor.from_tensor(b, [BLOCK_K, N], b_layout)
    c_desc = TensorDescriptor.from_tensor(out, [M, N], c_layout)

    tma_tcgen05_kernel[(1, )](
        a_desc,
        b_desc,
        c_desc,
        NUM_K_TILES,
        acc_tmem_layout,
        num_warps=4,
        num_ctas=4,
    )
    return out


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tma_tcgen05():
    M = 512
    N = 128
    K = 128
    a = torch.randn((M, K), device="cuda", dtype=torch.float16)
    b = torch.randn((K, N), device="cuda", dtype=torch.float16)

    out = tma_tcgen05_example(a, b)
    torch.testing.assert_close(out, torch.matmul(a, b), atol=1e-1, rtol=1e-2)
```
**EN:** `tma_tcgen05_kernel` abstracts the safe loop structure for feeding `tcgen05_mma` from TMA, even when operands are multicast and the accumulator is in two-CTA mode. The crucial detail is `tcgen05_mma_barrier_count(..., multicast=True, two_ctas=...)`, which computes the correct arrival count so that the next TMA stage cannot overwrite shared memory before every participating CTA has finished consuming the current tile.

**CN:** `tma_tcgen05_kernel` 把“用 TMA 给 `tcgen05_mma` 供数”的安全循环结构抽象了出来，即使操作数使用 multicast，或者 accumulator 处于 two-CTA 模式也一样适用。这里最关键的细节是 `tcgen05_mma_barrier_count(..., multicast=True, two_ctas=...)`：它会计算正确的 barrier 到达计数，防止下一轮 TMA 在所有相关 CTA 都消费完当前 tile 之前就覆盖 shared memory。
### Lines 648-833 — CLC-aware scheduling helpers for the final multi-CTA matmul
```python
# %%
# A Speed Of Light matmul kernel
# ------------------------------
#
# Here we illustrate a fully generic approach to writing a matmul kernel that uses TMA (perhaps with
# multicast) into warp-specialized tcgen05_mma pipeline.
#
# For this example, we generalise the CLC ideas presented in 12-cluster-launch-control.py to a warp-specialized
# kernel by adding a new partition that handles the CLC generation and broadcasts this to all CTAs
#
# We also use an extra helper called _planar_snake to swizzle the program id's to improve L2 locality.

Counter = t8.Counter
cublas = t8.cublas


@gluon.constexpr_function
def get_split_dim(cga_layout, dim):
    return 1 << sum(b[dim] != 0 for b in cga_layout)


@gluon.jit
def _planar_snake(lin_idx, m_tiles, n_tiles, minor_dim: gl.constexpr, tile_width: gl.constexpr):
    major_size = n_tiles if minor_dim == 0 else m_tiles
    minor_size = m_tiles if minor_dim == 0 else n_tiles

    full_minor_tiles = minor_size // tile_width
    full_minor_size = full_minor_tiles * tile_width
    full_elements = full_minor_tiles * tile_width * major_size

    minor_tile_idx = lin_idx // (tile_width * major_size)

    full_minor_within = lin_idx % tile_width
    full_major_within = (lin_idx // tile_width) % major_size
    full_minor = minor_tile_idx * tile_width + full_minor_within
    full_major = gl.where((minor_tile_idx % 2) == 0, full_major_within, major_size - 1 - full_major_within)

    partial_width = minor_size - full_minor_size
    partial_width = gl.where(partial_width > 0, partial_width, 1)
    partial_lin = lin_idx - full_elements
    partial_minor_within = partial_lin % partial_width
    partial_major_within = (partial_lin // partial_width) % major_size
    partial_minor = minor_tile_idx * tile_width + partial_minor_within
    partial_major = gl.where((minor_tile_idx % 2) == 0, partial_major_within, major_size - 1 - partial_major_within)

    in_full_tile = lin_idx < full_elements
    minor = gl.where(in_full_tile, full_minor, partial_minor)
    major = gl.where(in_full_tile, full_major, partial_major)

    if minor_dim == 0:
        return minor, major
    return major, minor


@gluon.aggregate
class ClcTileSchedulerConsumer:
    has_work: gl.tensor
    tile_id: gl.tensor
    pid_m: gl.tensor
    pid_n: gl.tensor
    num_pid_m: gl.tensor
    num_pid_n: gl.tensor
    TILE_M: gl.constexpr
    TILE_N: gl.constexpr
    MINOR_DIM: gl.constexpr
    GRID_TILE_WIDTH: gl.constexpr
    clc_result_buffers: gl.shared_memory_descriptor
    clc_barriers: gl.shared_memory_descriptor
    clc_planar_pid_buffers: gl.shared_memory_descriptor
    clc_planar_ready_bars: gl.shared_memory_descriptor
    clc_consumed_bars: gl.shared_memory_descriptor
    counter: Counter
    consumed_counter: Counter

    @gluon.jit
    def initialize(M, N, TILE_M: gl.constexpr, TILE_N: gl.constexpr, MINOR_DIM: gl.constexpr,
                   GRID_TILE_WIDTH: gl.constexpr, clc_result_buffers, clc_barriers, clc_planar_pid_buffers,
                   clc_planar_ready_bars, clc_consumed_bars):
        tile_id = gl.program_id(axis=0)
        num_pid_m = gl.cdiv(M, TILE_M)
        num_pid_n = gl.cdiv(N, TILE_N)
        pid_m, pid_n = _planar_snake(tile_id, num_pid_m, num_pid_n, MINOR_DIM, GRID_TILE_WIDTH)
        return ClcTileSchedulerConsumer(
            gl.to_tensor(True),
            tile_id,
            pid_m,
            pid_n,
            num_pid_m,
            num_pid_n,
            TILE_M,
            TILE_N,
            MINOR_DIM,
            GRID_TILE_WIDTH,
            clc_result_buffers,
            clc_barriers,
            clc_planar_pid_buffers,
            clc_planar_ready_bars,
            clc_consumed_bars,
            Counter.create(0, clc_barriers.shape[0]),
            Counter.create(0, clc_barriers.shape[0]),
        )

    @gluon.jit
    def get_offsets(self):
        return self.pid_m * self.TILE_M, self.pid_n * self.TILE_N

    @gluon.jit
    def step(self, iteration):
        consumed_counter = self.consumed_counter
        if iteration > 0:
            mbarrier.arrive(self.clc_consumed_bars.index(consumed_counter.index))
            consumed_counter = consumed_counter.next()

        counter = self.counter
        barrier = self.clc_barriers.index(counter.index)
        result = self.clc_result_buffers.index(counter.index)
        mbarrier.wait(barrier, counter.phase)
        clc_res = clc.load_result(result)
        mbarrier.wait(self.clc_planar_ready_bars.index(counter.index), counter.phase)
        planar_slot = self.clc_planar_pid_buffers.index(counter.index)
        planar_layout: gl.constexpr = gl.BlockedLayout([1], [32], [gl.num_warps()], [0],
                                                       [[0]] * (gl.num_ctas().bit_length() - 1))
        packed_pid = planar_slot.load(planar_layout).reshape([])
        pid_m = ((packed_pid >> 32) & 0xFFFFFFFF).to(gl.int32)
        pid_n = (packed_pid & 0xFFFFFFFF).to(gl.int32)
        has_work = clc_res.is_canceled()
        tile_id = self.tile_id
        if has_work:
            tile_id = clc_res.program_id(0)
        return ClcTileSchedulerConsumer(
            has_work,
            tile_id,
            pid_m,
            pid_n,
            self.num_pid_m,
            self.num_pid_n,
            self.TILE_M,
            self.TILE_N,
            self.MINOR_DIM,
            self.GRID_TILE_WIDTH,
            self.clc_result_buffers,
            self.clc_barriers,
            self.clc_planar_pid_buffers,
            self.clc_planar_ready_bars,
            self.clc_consumed_bars,
            counter.next(),
            consumed_counter,
        )


@gluon.aggregate
class MatmulPartitionArgs:
    a_desc: tma.tensor_descriptor
    b_desc: tma.tensor_descriptor
    c_desc: tma.tensor_descriptor
    a_bufs: gl.shared_memory_descriptor
    b_bufs: gl.shared_memory_descriptor
    load_empty_bars: gl.shared_memory_descriptor
    load_ready_bars: gl.shared_memory_descriptor
    acc_bufs: tensor_memory_descriptor
    acc_empty_bars: gl.shared_memory_descriptor
    acc_ready_bars: gl.shared_memory_descriptor
    clc_result_buffers: gl.shared_memory_descriptor
    clc_barriers: gl.shared_memory_descriptor
    clc_planar_pid_buffers: gl.shared_memory_descriptor
    clc_planar_ready_bars: gl.shared_memory_descriptor
    clc_consumed_bars: gl.shared_memory_descriptor
    MINOR_DIM: gl.constexpr
    GRID_TILE_WIDTH: gl.constexpr
    SUBTILE_STAGES: gl.constexpr

    @gluon.jit
    def get_clc_consumer(self):
        return ClcTileSchedulerConsumer.initialize(
            self.c_desc.shape[0],
            self.c_desc.shape[1],
            self.a_desc.block_shape[0],
            self.b_desc.block_shape[1],
            self.MINOR_DIM,
            self.GRID_TILE_WIDTH,
            self.clc_result_buffers,
            self.clc_barriers,
            self.clc_planar_pid_buffers,
            self.clc_planar_ready_bars,
            self.clc_consumed_bars,
        )
```
**EN:** This section starts from small helpers like `get_split_dim` and `_planar_snake`, then builds a proper consumer-side scheduler around CLC results. `ClcTileSchedulerConsumer` decodes packed `(pid_m, pid_n)` values published by the CLC partition, and `MatmulPartitionArgs` aggregates all descriptors, stage buffers, barriers, and scheduling constants needed by the final warp-specialized kernel.

**CN:** 这一节先从 `get_split_dim` 与 `_planar_snake` 这样的辅助函数出发，再逐步构建出一个真正的 CLC 消费端调度器。`ClcTileSchedulerConsumer` 会解码由 CLC partition 发布的打包 `(pid_m, pid_n)` 值，而 `MatmulPartitionArgs` 则把最终 warp-specialized kernel 所需的所有 descriptor、stage buffer、barrier 和调度常量统一封装起来。
### Lines 836-963 — CLC, load, MMA, and epilogue partitions of the final kernel
```python
@gluon.jit
def matmul_clc_partition(p):
    tile_m: gl.constexpr = p.a_desc.block_shape[0]
    tile_n: gl.constexpr = p.b_desc.block_shape[1]
    has_work = gl.to_tensor(True)
    num_pid_m = gl.cdiv(p.c_desc.shape[0], tile_m)
    num_pid_n = gl.cdiv(p.c_desc.shape[1], tile_n)
    state = Counter.create(0, p.clc_barriers.shape[0])
    consumed_state = Counter.create(1, p.clc_barriers.shape[0])
    acc_stages: gl.constexpr = p.clc_barriers.shape[0]
    i = 0
    while has_work:
        mbarrier.wait(p.clc_consumed_bars.index(consumed_state.index), consumed_state.phase, pred=(i >= acc_stages))
        barrier = p.clc_barriers.index(state.index)
        result = p.clc_result_buffers.index(state.index)
        mbarrier.expect(barrier, 16)
        clc.try_cancel(result, barrier)
        mbarrier.wait(barrier, state.phase)
        clc_res = clc.load_result(result)
        has_work = clc_res.is_canceled()
        pid_m = gl.to_tensor(0)
        pid_n = gl.to_tensor(0)
        if has_work:
            tile_id = clc_res.program_id(0)
            pid_m, pid_n = _planar_snake(tile_id, num_pid_m, num_pid_n, p.MINOR_DIM, p.GRID_TILE_WIDTH)
        packed_pid = (pid_m.to(gl.int64) << 32) | (pid_n.to(gl.int64) & 0xFFFFFFFF)
        planar_slot = p.clc_planar_pid_buffers.index(state.index)
        planar_layout: gl.constexpr = gl.BlockedLayout([1], [32], [gl.num_warps()], [0],
                                                       [[0]] * (gl.num_ctas().bit_length() - 1))
        planar_slot.store(gl.full([1], packed_pid, gl.int64, layout=planar_layout))
        mbarrier.arrive(p.clc_planar_ready_bars.index(state.index))
        state = state.next()
        consumed_state = consumed_state.next()
        i += 1


@gluon.jit
def matmul_load_partition(p):
    block_k: gl.constexpr = p.a_desc.block_shape[1]
    K = p.a_desc.shape[1]

    concurrent_loads: gl.constexpr = p.load_ready_bars.shape[0]
    state = Counter.create(1, concurrent_loads)
    scheduler = p.get_clc_consumer()

    i = 0
    while scheduler.has_work:
        off_m, off_n = scheduler.get_offsets()
        for k in range(0, K, block_k):
            pred = (i > 0) or (k >= block_k * concurrent_loads)
            mbarrier.wait(p.load_empty_bars.index(state.index), state.phase, pred=pred)
            bar = p.load_ready_bars.index(state.index)
            mbarrier.expect(bar, p.a_desc.nbytes_per_cta + p.b_desc.nbytes_per_cta)
            tma.async_load(p.a_desc, [off_m, k], bar, p.a_bufs.index(state.index), multicast=True)
            tma.async_load(p.b_desc, [k, off_n], bar, p.b_bufs.index(state.index), multicast=True)
            state = state.next()
        scheduler = scheduler.step(i)
        i += 1


@gluon.jit
def matmul_mma_partition(p):
    block_k: gl.constexpr = p.a_desc.block_shape[1]
    K = p.a_desc.shape[1]
    acc_stages: gl.constexpr = p.acc_empty_bars.shape[0]

    load_state = Counter.create(0, p.load_empty_bars.shape[0])
    acc_state = Counter.create(1, acc_stages)
    scheduler = p.get_clc_consumer()

    i = 0
    while scheduler.has_work:
        acc_buf = p.acc_bufs.index(acc_state.index)
        mbarrier.wait(p.acc_empty_bars.index(acc_state.index), acc_state.phase, pred=(i >= acc_stages))
        use_acc = False
        for k in range(0, K, block_k):
            mbarrier.wait(p.load_ready_bars.index(load_state.index), load_state.phase)
            tcgen05_mma(
                p.a_bufs.index(load_state.index),
                p.b_bufs.index(load_state.index),
                acc_buf,
                use_acc=use_acc,
                multicast=True,
                mbarriers=[p.load_empty_bars.index(load_state.index)],
            )
            load_state = load_state.next()
            use_acc = True
        tcgen05_commit(p.acc_ready_bars.index(acc_state.index), descs=[p.a_bufs.index(0), p.b_bufs.index(0)])
        acc_state = acc_state.next()
        scheduler = scheduler.step(i)
        i += 1


@gluon.jit
def matmul_epilogue_partition(p):
    tile_m: gl.constexpr = p.a_desc.block_shape[0]
    tile_n: gl.constexpr = p.b_desc.block_shape[1]
    split_tile_n: gl.constexpr = p.c_desc.block_shape[1]
    # Separate knobs: SUBTILE_STAGES controls shared-memory usage,
    # and SUBTILE_FACTOR is the maximum number of subtiles into which we can split the tile.
    subtile_factor: gl.constexpr = tile_n // split_tile_n
    subtile_stages: gl.constexpr = p.SUBTILE_STAGES
    acc_stages: gl.constexpr = p.acc_empty_bars.shape[0]
    dtype: gl.constexpr = p.c_desc.dtype

    acc_state = Counter.create(0, acc_stages)
    acc_smems = gl.allocate_shared_memory(dtype, [subtile_stages, tile_m, split_tile_n], p.c_desc.layout)
    sub_acc_state = Counter.create(0, subtile_stages)
    scheduler = p.get_clc_consumer()

    i = 0
    while scheduler.has_work:
        off_m, off_n = scheduler.get_offsets()
        mbarrier.wait(p.acc_ready_bars.index(acc_state.index), acc_state.phase)
        acc_buf = p.acc_bufs.index(acc_state.index)

        for s in gl.static_range(subtile_factor):
            acc_sub = acc_buf.slice(split_tile_n * s, split_tile_n)
            acc_smem = acc_smems.index(sub_acc_state.index)
            acc = acc_sub.load().to(dtype)
            tma.store_wait(pendings=subtile_stages - 1)
            acc_smem.store(acc)
            tma.async_copy_shared_to_global(p.c_desc, [off_m, off_n + split_tile_n * s], acc_smem)
            sub_acc_state = sub_acc_state.next()
        mbarrier.arrive(p.acc_empty_bars.index(acc_state.index))
        acc_state = acc_state.next()
        scheduler = scheduler.step(i)
        i += 1
```
**EN:** The four partitions split responsibilities cleanly: `matmul_clc_partition` produces new tile assignments, `matmul_load_partition` issues multicast TMA loads, `matmul_mma_partition` drives `tcgen05_mma` on staged operands, and `matmul_epilogue_partition` slices the TMEM accumulator into storeable subtiles. The design shows how CGA programming scales when scheduling, loading, computing, and storing are all cluster-aware but still synchronized through explicit barrier protocols.

**CN:** 四个 partition 的职责划分很清晰：`matmul_clc_partition` 负责生成新的 tile 分配，`matmul_load_partition` 发起 multicast TMA 载入，`matmul_mma_partition` 用 staged 操作数驱动 `tcgen05_mma`，`matmul_epilogue_partition` 则把 TMEM accumulator 切成可以写回的 subtile。这个设计展示了当调度、载入、计算和写回都具有 cluster 语义时，CGA 编程如何仍然通过显式 barrier 协议保持可控。
### Lines 966-1193 — Entry kernel, Python wrapper, test, and benchmark against cuBLAS
```python
# The entry kernel allocates and initializes every barrier before
# `gl.warp_specialize`. Some of these barriers are cross-CTA barriers, and some
# otherwise per-CTA barriers are consumed by multicast or 2CTA operations. The
# compiler inserts the required init fence and relaxed cluster barrier after
# this top-level init sequence, before any partition can use the barriers.
# Keeping the init sequence here also keeps that mandatory cluster sync outside
# `warp_specialize`.
@gluon.jit
def matmul_multicta_kernel(
    a_desc,
    b_desc,
    c_desc,
    M,
    N,
    K,
    BLOCK_SIZE_M: gl.constexpr,
    BLOCK_SIZE_N: gl.constexpr,
    BLOCK_SIZE_K: gl.constexpr,
    GRID_MINOR_DIM: gl.constexpr,
    GRID_TILE_WIDTH: gl.constexpr,
    STAGES: gl.constexpr,
    ACC_STAGES: gl.constexpr,
    CGA_LAYOUT: gl.constexpr,
    EPILOGUE_SIZE_N: gl.constexpr,
    SUBTILE_STAGES: gl.constexpr,
):
    block_m: gl.constexpr = a_desc.block_shape[0]
    block_n: gl.constexpr = b_desc.block_shape[1]
    two_ctas: gl.constexpr = gl.num_ctas() > 1
    n_partitions: gl.constexpr = 4

    dtype: gl.constexpr = a_desc.dtype
    a_bufs = gl.allocate_shared_memory(dtype, [STAGES] + a_desc.block_shape, a_desc.layout)
    b_bufs = gl.allocate_shared_memory(dtype, [STAGES] + b_desc.block_shape, b_desc.layout)
    tmem_layout: gl.constexpr = TensorMemoryLayout(
        [BLOCK_SIZE_M, block_n // get_split_dim(CGA_LAYOUT, 1)],
        col_stride=1,
        cga_layout=CGA_LAYOUT,
        two_ctas=two_ctas,
    )
    acc_bufs = allocate_tensor_memory(gl.float32, [ACC_STAGES, block_m, block_n], tmem_layout)
    mma_barrier_count: gl.constexpr = tcgen05_mma_barrier_count([a_bufs.index(0), b_bufs.index(0)], multicast=True,
                                                                two_ctas=acc_bufs.index(0).type.layout.two_ctas)

    load_empty_bars = mbarrier.allocate_mbarrier(batch=STAGES)
    load_ready_bars = mbarrier.allocate_mbarrier(batch=STAGES, two_ctas=two_ctas)
    for i in gl.static_range(STAGES):
        mbarrier.init(load_empty_bars.index(i), count=mma_barrier_count)
        mbarrier.init(load_ready_bars.index(i), count=1)

    acc_empty_bars = mbarrier.allocate_mbarrier(batch=ACC_STAGES, two_ctas=two_ctas)
    acc_ready_bars = mbarrier.allocate_mbarrier(batch=ACC_STAGES)
    for i in gl.static_range(ACC_STAGES):
        mbarrier.init(acc_empty_bars.index(i), count=1)
        mbarrier.init(acc_ready_bars.index(i), count=mma_barrier_count)

    clc_barriers = mbarrier.allocate_mbarrier(batch=ACC_STAGES)
    clc_planar_ready_bars = mbarrier.allocate_mbarrier(batch=ACC_STAGES)
    clc_consumed_bars = mbarrier.allocate_mbarrier(batch=ACC_STAGES, two_ctas=two_ctas)
    for i in gl.static_range(ACC_STAGES):
        mbarrier.init(clc_barriers.index(i), count=1)
        mbarrier.init(clc_planar_ready_bars.index(i), count=1)
        mbarrier.init(clc_consumed_bars.index(i), count=n_partitions - 1)

    cga_layout: gl.constexpr = [[0]] * (gl.num_ctas().bit_length() - 1)
    clc_layout: gl.constexpr = gl.SwizzledSharedLayout(1, 1, 1, [0], cga_layout=cga_layout)
    clc_result_buffers = gl.allocate_shared_memory(
        gl.int64,
        [clc_barriers.shape[0], 2],
        clc_layout,
    )
    clc_planar_pid_buffers = gl.allocate_shared_memory(gl.int64, [clc_barriers.shape[0], 1], clc_layout)

    p = MatmulPartitionArgs(
        a_desc,
        b_desc,
        c_desc,
        a_bufs,
        b_bufs,
        load_empty_bars,
        load_ready_bars,
        acc_bufs,
        acc_empty_bars,
        acc_ready_bars,
        clc_result_buffers,
        clc_barriers,
        clc_planar_pid_buffers,
        clc_planar_ready_bars,
        clc_consumed_bars,
        GRID_MINOR_DIM,
        GRID_TILE_WIDTH,
        SUBTILE_STAGES,
    )

    gl.warp_specialize([
        (matmul_epilogue_partition, (p, )),
        (matmul_load_partition, (p, )),
        (matmul_mma_partition, (p, )),
        (matmul_clc_partition, (p, )),
    ], [1, 1, 1], [24, 24, 24])


def matmul_multicta(
        a,
        b,
        out=None,
        *,
        block_size_m=128,
        block_size_n=256,
        block_size_k=64,
        grid_minor_dim=0,
        grid_tile_width=16,
        stages=6,
        acc_stages=2,
        cga_layout=((1, 0), ),
        epilogue_size_n=32,
        subtile_stages=4,
):
    if block_size_n // get_split_dim(cga_layout, 1) > 256:
        raise ValueError(
            f"cga_layout={list(cga_layout)} only supports BLOCK_SIZE_N <= {256 * get_split_dim(cga_layout, 1)}")

    M, K = a.shape
    K1, N = b.shape
    if K != K1:
        raise ValueError(f"incompatible shapes: {a.shape} and {b.shape}")
    if a.dtype != torch.float16 or b.dtype != torch.float16:
        raise ValueError("matmul only supports fp16 inputs")

    if out is None:
        c = torch.empty((M, N), device=a.device, dtype=a.dtype)
    else:
        if out.shape != (M, N):
            raise ValueError(f"Output has invalid shape {out.shape}, expected {(M, N)}")
        c = out

    tile_m = block_size_m * get_split_dim(cga_layout, 0)
    two_ctas = bool(cga_layout)
    a_layout = gl.NVMMASharedLayout.get_default_for([tile_m, block_size_k], gl.float16,
                                                    cga_layout=get_cga_layout(cga_layout, 0, two_ctas))
    b_layout = gl.NVMMASharedLayout.get_default_for([block_size_k, block_size_n], gl.float16,
                                                    cga_layout=get_cga_layout(cga_layout, 1, two_ctas))
    c_layout = gl.NVMMASharedLayout.get_default_for([tile_m, epilogue_size_n], gl.float16, cga_layout=cga_layout)

    a_desc = TensorDescriptor.from_tensor(a, [tile_m, block_size_k], a_layout)
    b_desc = TensorDescriptor.from_tensor(b, [block_size_k, block_size_n], b_layout)
    c_desc = TensorDescriptor.from_tensor(c, [tile_m, epilogue_size_n], c_layout)

    def grid(meta):
        tile_m = meta["BLOCK_SIZE_M"] * get_split_dim(meta["CGA_LAYOUT"], 0)
        tile_n = meta["BLOCK_SIZE_N"]
        num_tiles = triton.cdiv(M, tile_m) * triton.cdiv(N, tile_n)
        return (num_tiles, )

    matmul_multicta_kernel[grid](
        a_desc,
        b_desc,
        c_desc,
        M,
        N,
        K,
        block_size_m,
        block_size_n,
        block_size_k,
        grid_minor_dim,
        grid_tile_width,
        stages,
        acc_stages,
        cga_layout,
        epilogue_size_n,
        subtile_stages,
        num_warps=4,
        num_ctas=2**len(cga_layout),
    )
    return c


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_matmul_multicta():
    M, N, K = 1024, 1024, 512
    a = torch.randn((M, K), device="cuda", dtype=torch.float16)
    b = torch.randn((K, N), device="cuda", dtype=torch.float16)
    c = matmul_multicta(a, b)
    torch.testing.assert_close(c, torch.matmul(a, b), atol=1e-1, rtol=1e-2)


if __name__ == "__main__" and is_blackwell():
    print("Benchmarking matmul_multicta")
    print("============================")
    cfg = {
        "block_size_m": 128,
        "block_size_n": 256,
        "block_size_k": 64,
        "grid_minor_dim": 0,
        "grid_tile_width": 16,
        "stages": 6,
        "acc_stages": 2,
        "cga_layout": ((1, 0), ),
        "epilogue_size_n": 32,
        "subtile_stages": 4,
    }

    M, N = 8192, 8192
    C = torch.empty((M, N), device="cuda", dtype=torch.float16)
    print("    K         multi-CTA    cublas")
    for K in [2**i for i in range(9, 15)]:
        A = torch.randn((M, K), device="cuda", dtype=torch.float16)
        B = torch.randn((K, N), device="cuda", dtype=torch.float16)
        BT = B.T.contiguous()
        r0 = tflops(triton.testing.do_bench(lambda: matmul_multicta(A, B, out=C, **cfg), warmup=200, rep=1000), M, N, K)
        r1 = tflops(triton.testing.do_bench(lambda: cublas.matmul(A, BT, C), warmup=200, rep=1000), M, N, K)
        print(f"{K:>5} {r0:>17.2f} {r1:>9.2f}")

# %%
# Benchmarking matmul_multicta
# ============================
#     K         multi-CTA    cublas
#   512           1096.31   1190.98
#  1024           1306.07   1344.48
#  2048           1379.80   1374.48
#  4096           1444.26   1431.93
#  8192           1302.33   1347.82
# 16384           1292.40   1371.82
#
# We are able to be competitive with cublas and even beating them in quite a range
# of relevant Ks for this particular configuration. If we chose different configurations
# for different shapes we would be able to beat cublas in a wider range of shapes.
```
**EN:** The entry kernel allocates every cross-CTA barrier before `gl.warp_specialize`, initializes them with counts derived from multicast and 2CTA semantics, and then launches all four partitions together. The Python wrapper derives operand layouts from the requested `cga_layout`, computes the grid in logical tiles, and the final benchmark compares the resulting multi-CTA matmul against cuBLAS to show that the approach is not just demonstrative but competitive.

**CN:** 入口 kernel 在 `gl.warp_specialize` 之前先分配并初始化所有跨 CTA barrier，计数设置同时考虑了 multicast 与 2CTA 语义，然后再一起启动四个 partition。Python 包装函数根据请求的 `cga_layout` 推导操作数布局、按逻辑 tile 计算 grid，而最终 benchmark 还把这个 multi-CTA matmul 与 cuBLAS 做对比，说明这里展示的不只是概念，而是具有竞争力的实现路线。
## Key Concepts / 关键概念
- CGA (`num_ctas > 1`) adds a cluster-level collaboration model that can be driven by layouts as well as explicit barriers / CGA（`num_ctas > 1`）提供了 cluster 级协作模型，它既可以由 layout 驱动，也可以由显式 barrier 驱动。
- 2CTA tcgen05 MMA requires coordinated operand layouts, accumulator TMEM layout, and barrier initialization counts / 2CTA tcgen05 MMA 同时要求操作数布局、accumulator 的 TMEM 布局以及 barrier 初始化计数彼此匹配。
- TMA multicast is the natural way to feed identical tiles to multiple CTAs without duplicating global-memory traffic / TMA multicast 是向多个 CTA 分发同一 tile、避免重复全局内存流量的自然方式。
- CLC can be integrated into warp-specialized multi-CTA kernels by treating scheduling as its own partition / 通过把调度本身作为一个独立 partition，CLC 可以自然融入 warp-specialized 的多 CTA kernel。

## Dependencies / 依赖关系
- `clc`, `tcgen05_mma`, `tcgen05_commit`, `tcgen05_mma_barrier_count`, and `TensorMemoryLayout` provide the Blackwell-specific multi-CTA math and scheduling primitives / `clc`、`tcgen05_mma`、`tcgen05_commit`、`tcgen05_mma_barrier_count` 和 `TensorMemoryLayout` 提供了 Blackwell 专用的多 CTA 计算与调度原语。
- `mbarrier` and `tma` from the Hopper namespace are reused for cluster-visible synchronization and multicast loads/stores / Hopper 命名空间中的 `mbarrier` 与 `tma` 被复用于 cluster 可见同步和 multicast 载入/写回。
- `TensorDescriptor` wraps input/output tensors with the chosen CGA-aware layouts / `TensorDescriptor` 按照选定的 CGA 感知布局封装输入输出张量。
- `08-warp-specialization` supplies counters and cuBLAS benchmarking helpers reused by the final kernel / `08-warp-specialization` 提供了最终 kernel 复用的 counter 与 cuBLAS benchmark 辅助函数。
