# reduce.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/reduce/reduce.py`  
**Purpose / 用途**: Hierarchical Reduction Utilities for CuTe-DSL Kernels. / 这是一个使用 CuTeDSL 实现 reduce 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行

~~~~python
# Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause

# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:

# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.

# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.

# 3. Neither the name of the copyright holder nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.

# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
~~~~

**EN**: Records the copyright, SPDX tags, and license conditions that govern how this example may be used and redistributed.
**CN**: 记录版权、SPDX 标记和许可证条件，说明该示例如何被使用和再分发。

### Lines 29-129 / 第 29-129 行

~~~~python
"""
Hierarchical Reduction Utilities for CuTe-DSL Kernels
=====================================================

This module provides reusable reduction primitives for GPU kernels that need to
reduce values across warps, thread blocks, and clusters (SM90+).

Overview
--------
GPU reductions typically follow a hierarchical pattern:

1. **Warp Reduction**: Threads within a warp reduce using shuffle instructions.
   Use `cute.arch.warp_reduction()` from the CuTe-DSL library.

2. **Block Reduction**: Multiple warps within a block reduce using shared memory.
   Use `block_reduce()` from this module.

3. **Cluster Reduction** (SM90+): Multiple CTAs in a cluster reduce using
   distributed shared memory and mbarrier synchronization.
   Use `cluster_reduce()` from this module.

4. **Row Reduction**: Orchestrates all levels based on problem configuration.
   Use `row_reduce()` from this module.

Shared Memory Buffer Layout Assumptions
---------------------------------------

For `block_reduce`:
    - Buffer shape: (rows_per_block, warps_per_row)
    - Each warp's lane 0 writes its reduced value to buffer[row_idx, col_idx]
    - Thread mapping: row_idx = warp_idx // warps_per_row
                      col_idx = warp_idx % warps_per_row

    Example for 8 warps, 2 rows, 4 warps per row:
        Warp 0 -> buffer[0, 0]    Warp 4 -> buffer[1, 0]
        Warp 1 -> buffer[0, 1]    Warp 5 -> buffer[1, 1]
        Warp 2 -> buffer[0, 2]    Warp 6 -> buffer[1, 2]
        Warp 3 -> buffer[0, 3]    Warp 7 -> buffer[1, 3]

For `cluster_reduce`:
    - Buffer shape: (rows_per_block, (warps_per_row, cluster_n))
    - The second dimension is hierarchical: (local_warp_slot, cta_rank)
    - Each CTA contributes to its own slot in the cluster dimension

    Example for cluster_n=4, 2 warps per row:
        CTA 0, Warp 0 -> buffer[row, (0, 0)]
        CTA 0, Warp 1 -> buffer[row, (1, 0)]
        CTA 1, Warp 0 -> buffer[row, (0, 1)]
        CTA 1, Warp 1 -> buffer[row, (1, 1)]
        ... etc for CTAs 2, 3

Mbarrier Requirements (Cluster Reduction)
-----------------------------------------
For cluster reduction, the caller must:
1. Allocate an mbarrier in shared memory
2. Initialize it with `cute.arch.mbarrier_init(mbar_ptr, thread_count)`
3. Pass the mbarrier pointer to `cluster_reduce()`

The cluster_reduce function handles:
- Setting up the expected transaction count
- Performing async cross-CTA stores
- Waiting for all stores to complete

Usage Example
-------------

.. code-block:: python

    from reduce import row_reduce, block_reduce, cluster_reduce

    @cute.jit
    def my_kernel(...):
        # Allocate shared memory for reduction
        # Shape depends on warps_per_row and cluster_n
        if cluster_n > 1:
            reduction_buffer = cute.make_smem_tensor(
                cute.make_layout((rows_per_block, (warps_per_row, cluster_n))),
                Float32
            )
        else:
            reduction_buffer = cute.make_smem_tensor(
                cute.make_layout((rows_per_block, warps_per_row)),
                Float32
            )

        # Perform row reduction
        result = row_reduce(
            tensor_ssa,
            cute.ReductionOp.ADD,
            threads_per_row,
            reduction_buffer,
            mbar_ptr,
            cluster_n,
            init_val=Float32(0.0)
        )

References
----------
The cluster synchronization primitives (set_block_rank, store_shared_remote)
are inspired by Quack: https://github.com/Dao-AILab/quack
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 131-138 / 第 131-138 行

~~~~python
import operator
from collections.abc import Callable

import cutlass
import cutlass.cute as cute
from cutlass import Float32, Int32
from cutlass._mlir.dialects import llvm
from cutlass.cutlass_dsl import T, dsl_user_op
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 141-147 / 第 141-147 行

~~~~python
# =============================================================================
# Inline PTX Operations for Cluster Communication
# =============================================================================
#
# These operations enable cross-CTA communication within a cluster (SM90+).
# They use inline PTX assembly for functionality not yet exposed in MLIR.
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 150-177 / 第 150-177 行

~~~~python
@dsl_user_op
def set_block_rank(
    smem_ptr: cute.Pointer, peer_cta_rank_in_cluster: Int32, *, loc=None, ip=None
) -> Int32:
    """
    Map a shared memory pointer to the equivalent address in another CTA's
    shared memory within the same cluster.

    This uses the PTX `mapa.shared::cluster` instruction to translate a local
    shared memory address to the corresponding address in a peer CTA's shared
    memory space.

    Args:
        smem_ptr: Pointer to local shared memory
        peer_cta_rank_in_cluster: Target CTA's rank within the cluster (0 to cluster_size-1)

    Returns:
        Int32 representing the mapped address in the peer CTA's shared memory

    Note:
        This operation requires SM90+ with cluster support enabled.
        The cluster must be launched with the appropriate cluster dimensions.
    """
    smem_ptr_i32 = smem_ptr.toint(loc=loc, ip=ip).ir_value()
    return Int32(
        llvm.inline_asm(
            T.i32(),
            [smem_ptr_i32, peer_cta_rank_in_cluster.ir_value()],
~~~~

**EN**: Defines `set_block_rank` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 将 `set_block_rank` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 178-184 / 第 178-184 行

~~~~python
            "mapa.shared::cluster.u32 $0, $1, $2;",
            "=r,r,r",
            has_side_effects=False,
            is_align_stack=False,
            asm_dialect=llvm.AsmDialect.AD_ATT,
        )
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 187-214 / 第 187-214 行

~~~~python
@dsl_user_op
def store_shared_remote(
    val: Float32,
    smem_ptr: cute.Pointer,
    mbar_ptr: cute.Pointer,
    peer_cta_rank_in_cluster: Int32,
    *,
    loc=None,
    ip=None,
) -> None:
    """
    Asynchronously store a Float32 value to shared memory on a remote CTA
    within the cluster, with mbarrier completion tracking.

    This uses the PTX `st.async.shared::cluster` instruction which:
    1. Translates the local smem address to the peer CTA's address space
    2. Performs an asynchronous store to the remote shared memory
    3. Signals the mbarrier when the store completes

    Args:
        val: The Float32 value to store
        smem_ptr: Pointer to the destination in local shared memory coordinates
        mbar_ptr: Pointer to the mbarrier that tracks completion
        peer_cta_rank_in_cluster: Target CTA's rank within the cluster

    Note:
        - The mbarrier must be initialized with the expected transaction byte count
        - Use `cute.arch.mbarrier_arrive_and_expect_tx()` to set up the transaction
~~~~

**EN**: Defines `store_shared_remote` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `store_shared_remote` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 215-232 / 第 215-232 行

~~~~python
        - Use `cute.arch.mbarrier_wait()` to wait for all stores to complete
        - This operation requires SM90+ with cluster support enabled
    """
    remote_smem_ptr_i32 = set_block_rank(
        smem_ptr, peer_cta_rank_in_cluster, loc=loc, ip=ip
    ).ir_value()
    remote_mbar_ptr_i32 = set_block_rank(
        mbar_ptr, peer_cta_rank_in_cluster, loc=loc, ip=ip
    ).ir_value()
    llvm.inline_asm(
        None,
        [remote_smem_ptr_i32, val.ir_value(loc=loc, ip=ip), remote_mbar_ptr_i32],
        "st.async.shared::cluster.mbarrier::complete_tx::bytes.f32 [$0], $1, [$2];",
        "r,f,r",
        has_side_effects=True,
        is_align_stack=False,
        asm_dialect=llvm.AsmDialect.AD_ATT,
    )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 235-250 / 第 235-250 行

~~~~python
@dsl_user_op
def elem_pointer(x: cute.Tensor, coord, *, loc=None, ip=None) -> cute.Pointer:
    """
    Get a pointer to an element at the specified coordinate in a tensor.

    This is useful for getting the shared memory address of a specific element
    when performing cross-CTA stores in cluster reduction.

    Args:
        x: The tensor (typically a shared memory tensor)
        coord: The coordinate tuple, can be hierarchical like (row, (col, cluster_idx))

    Returns:
        Pointer to the element at the specified coordinate
    """
    return x.iterator + cute.crd2idx(coord, x.layout, loc=loc, ip=ip)
~~~~

**EN**: Defines `elem_pointer` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Returns the constructed object or computed result to the caller.
**CN**: 将 `elem_pointer` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 把构造好的对象或计算结果返回给调用方。

### Lines 253-255 / 第 253-255 行

~~~~python
# =============================================================================
# Block-Level Reduction
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 258-285 / 第 258-285 行

~~~~python
@cute.jit
def block_reduce(
    val: Float32,
    op: Callable,
    reduction_buffer: cute.Tensor,
    init_val: Float32,
) -> Float32:
    """
    Reduce values across all warps within a thread block using shared memory.

    This function assumes each warp has already performed a warp-level reduction
    and is contributing a single value (from lane 0). The function then:
    1. Writes each warp's value to shared memory
    2. Synchronizes the block
    3. Performs a final warp reduction across the collected values

    Args:
        val: The warp-reduced value (only lane 0's value is used)
        op: Binary reduction operator, e.g., `operator.add` or `cute.arch.fmax`
        reduction_buffer: Shared memory tensor with shape (rows_per_block, warps_per_row)
        init_val: Identity element for the reduction (0 for sum, -inf for max)

    Returns:
        The block-reduced result (same value across all threads)

    Buffer Layout:
        - Shape: (rows_per_block, warps_per_row)
        - warps_per_row is inferred from reduction_buffer.shape[1]
~~~~

**EN**: Marks `block_reduce` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `block_reduce` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 286-305 / 第 286-305 行

~~~~python
        - Thread mapping:
            row_idx = warp_idx // warps_per_row
            col_idx = warp_idx % warps_per_row

    Example:
        For a block with 8 warps processing 2 rows (4 warps per row):

        .. code-block:: python

            reduction_buffer = cute.make_smem_tensor(
                cute.make_layout((2, 4)),  # 2 rows, 4 warps per row
                Float32
            )
            result = block_reduce(warp_val, operator.add, reduction_buffer, Float32(0.0))
    """
    lane_idx = cute.arch.lane_idx()
    warp_idx = cute.arch.warp_idx()
    warps_per_row = cute.size(reduction_buffer.shape[1])
    row_idx = warp_idx // warps_per_row
    col_idx = warp_idx % warps_per_row
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 307-307 / 第 307-307 行

~~~~python
    # Lane 0 of each warp writes its value to shared memory
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 308-310 / 第 308-310 行

~~~~python
    if lane_idx == 0:
        reduction_buffer[row_idx, col_idx] = val
    cute.arch.barrier()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 312-313 / 第 312-313 行

~~~~python
    # All lanes participate in reading and reducing
    # Only lanes < warps_per_row have valid data
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 314-317 / 第 314-317 行

~~~~python
    block_reduce_val = init_val
    if lane_idx < warps_per_row:
        block_reduce_val = reduction_buffer[row_idx, lane_idx]
    return cute.arch.warp_reduction(block_reduce_val, op)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 320-322 / 第 320-322 行

~~~~python
# =============================================================================
# Cluster-Level Reduction (SM90+)
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 325-352 / 第 325-352 行

~~~~python
@cute.jit
def cluster_reduce(
    val: Float32,
    op: Callable,
    reduction_buffer: cute.Tensor,
    mbar_ptr: cute.Pointer,
    cluster_n: cutlass.Constexpr[int],
    init_val: Float32,
) -> Float32:
    """
    Reduce values across all CTAs within a cluster using distributed shared memory.

    This function extends block reduction to work across multiple CTAs in a cluster
    using asynchronous cross-CTA stores and mbarrier synchronization. It:
    1. Sets up the mbarrier with expected transaction count
    2. Asynchronously stores each warp's value to all peer CTAs
    3. Waits for all stores to complete
    4. Reduces across all collected values

    Args:
        val: The warp-reduced value (only lane 0's value is used for stores)
        op: Binary reduction operator, e.g., `operator.add` or `cute.arch.fmax`
        reduction_buffer: Shared memory tensor with hierarchical shape
                          (rows_per_block, (warps_per_row, cluster_n))
        mbar_ptr: Pointer to an initialized mbarrier in shared memory
        cluster_n: Number of CTAs in the cluster (compile-time constant)
        init_val: Identity element for the reduction (0 for sum, -inf for max)
~~~~

**EN**: Marks `cluster_reduce` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `cluster_reduce` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 353-380 / 第 353-380 行

~~~~python
    Returns:
        The cluster-reduced result (same value across all threads in all CTAs)

    Buffer Layout:
        - Shape: (rows_per_block, (warps_per_row, cluster_n))
        - The second dimension is hierarchical:
          - First level: warps_per_row (local warp slots)
          - Second level: cluster_n (one slot per CTA in cluster)
        - Access pattern: buffer[row_idx, (col_idx, cta_rank)]

    Requirements:
        - SM90+ with cluster support
        - Mbarrier must be initialized before calling
        - Kernel must be launched with appropriate cluster dimensions

    Example:
        For a cluster of 4 CTAs, each with 2 warps per row:

        .. code-block:: python

            # Allocate buffer with cluster dimension
            reduction_buffer = cute.make_smem_tensor(
                cute.make_layout((rows_per_block, (2, 4))),  # 2 warps, 4 CTAs
                Float32
            )

            # Initialize mbarrier (once per kernel)
            mbar = cute.make_smem_tensor(cute.make_layout((1,)), cute.arch.Mbarrier)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 381-391 / 第 381-391 行

~~~~python
            cute.arch.mbarrier_init(mbar.iterator, thread_count)

            # Perform cluster reduction
            result = cluster_reduce(
                warp_val, operator.add, reduction_buffer,
                mbar.iterator, cluster_n=4, init_val=Float32(0.0)
            )
    """
    cta_rank_in_cluster = cute.arch.block_idx_in_cluster()
    lane_idx = cute.arch.lane_idx()
    warp_idx = cute.arch.warp_idx()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 393-394 / 第 393-394 行

~~~~python
    rows_per_block = reduction_buffer.shape[0]
    warps_per_row = reduction_buffer.shape[1][0]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 396-397 / 第 396-397 行

~~~~python
    row_idx = warp_idx // warps_per_row
    col_idx = warp_idx % warps_per_row
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 399-400 / 第 399-400 行

~~~~python
    # Warp 0, lane 0 sets up mbarrier with expected transaction count
    # Each warp sends cluster_n stores (one to each CTA), each store is 4 bytes
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 401-405 / 第 401-405 行

~~~~python
    if warp_idx == 0:
        with cute.arch.elect_one():
            num_warps = rows_per_block * warps_per_row
            expected_bytes = num_warps * cluster_n * 4  # 4 bytes per Float32
            cute.arch.mbarrier_arrive_and_expect_tx(mbar_ptr, expected_bytes)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。

### Lines 407-408 / 第 407-408 行

~~~~python
    # Each lane < cluster_n writes to a different CTA's shared memory
    # This distributes the warp's value to all CTAs in the cluster
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 409-415 / 第 409-415 行

~~~~python
    if lane_idx < cluster_n:
        store_shared_remote(
            val,
            elem_pointer(reduction_buffer, (row_idx, (col_idx, cta_rank_in_cluster))),
            mbar_ptr,
            peer_cta_rank_in_cluster=lane_idx,
        )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 417-417 / 第 417-417 行

~~~~python
    # Wait for all cross-CTA stores to complete
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 418-418 / 第 418-418 行

~~~~python
    cute.arch.mbarrier_wait(mbar_ptr, phase=0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 420-421 / 第 420-421 行

~~~~python
    # Now each CTA has all values from all CTAs in the cluster
    # Reduce across all collected values
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 422-423 / 第 422-423 行

~~~~python
    num_total = warps_per_row * cluster_n
    num_iter = cute.ceil_div(num_total, 32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 425-429 / 第 425-429 行

~~~~python
    block_reduce_val = init_val
    for i in cutlass.range_constexpr(num_iter):
        idx = lane_idx + i * 32
        if idx < num_total:
            block_reduce_val = op(block_reduce_val, reduction_buffer[row_idx, idx])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 431-431 / 第 431-431 行

~~~~python
    return cute.arch.warp_reduction(block_reduce_val, op)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 434-436 / 第 434-436 行

~~~~python
# =============================================================================
# Row Reduction (Orchestration Function)
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 439-466 / 第 439-466 行

~~~~python
@cute.jit
def row_reduce(
    x: cute.TensorSSA,
    op: cute.ReductionOp,
    threads_per_row: cutlass.Constexpr[int],
    reduction_buffer: cute.Tensor,
    mbar_ptr,
    cluster_n: cutlass.Constexpr[int],
    init_val: Float32,
):
    """
    Perform hierarchical row reduction with automatic selection of reduction strategy.

    This function orchestrates the full reduction pipeline:
    1. Local reduction: Each thread reduces its portion of the row
    2. Warp reduction: Threads within a warp reduce using shuffles
    3. Block reduction: If needed, warps reduce using shared memory
    4. Cluster reduction: If needed, CTAs reduce using distributed shared memory

    The function automatically selects the appropriate reduction level based on
    `threads_per_row` and `cluster_n`.

    Args:
        x: TensorSSA containing the values to reduce (in registers)
        op: Reduction operation (cute.ReductionOp.ADD or cute.ReductionOp.MAX)
        threads_per_row: Number of threads cooperating on each row (compile-time)
        reduction_buffer: Shared memory tensor for block/cluster reduction
        mbar_ptr: Mbarrier pointer (only used if cluster_n > 1)
~~~~

**EN**: Marks `row_reduce` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `row_reduce` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 467-494 / 第 467-494 行

~~~~python
        cluster_n: Number of CTAs in cluster (1 for single-CTA reduction)
        init_val: Identity element for the reduction

    Returns:
        The fully reduced result for each row

    Reduction Strategy Selection:
        - threads_per_row <= 32, cluster_n == 1: Warp reduction only
        - threads_per_row > 32, cluster_n == 1: Warp + block reduction
        - cluster_n > 1: Warp + cluster reduction (handles all cases)

    Example:
        .. code-block:: python

            # Sum reduction across 128 threads per row, single CTA
            result = row_reduce(
                tensor_ssa,
                cute.ReductionOp.ADD,
                threads_per_row=128,
                reduction_buffer=smem_buffer,
                mbar_ptr=None,
                cluster_n=1,
                init_val=Float32(0.0)
            )

            # Max reduction across 256 threads per row, 4 CTAs in cluster
            result = row_reduce(
                tensor_ssa,
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 495-504 / 第 495-504 行

~~~~python
                cute.ReductionOp.MAX,
                threads_per_row=256,
                reduction_buffer=smem_buffer,
                mbar_ptr=mbar.iterator,
                cluster_n=4,
                init_val=Float32.neg_inf
            )
    """
    # Step 1: Local reduction - each thread reduces its register values
    local_val = x.reduce(op, init_val=init_val, reduction_profile=0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 506-506 / 第 506-506 行

~~~~python
    # Map ReductionOp enum to binary operator for warp/block reductions
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 507-510 / 第 507-510 行

~~~~python
    warp_op = {
        cute.ReductionOp.ADD: operator.add,
        cute.ReductionOp.MAX: cute.arch.fmax,
    }[op]
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 512-513 / 第 512-513 行

~~~~python
    # Step 2: Warp reduction
    # If threads_per_row < 32, only use that many threads in the reduction
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 514-515 / 第 514-515 行

~~~~python
    warp_width = min(threads_per_row, 32)
    warp_val = cute.arch.warp_reduction(local_val, warp_op, threads_in_group=warp_width)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 517-517 / 第 517-517 行

~~~~python
    # Determine if we need additional reduction levels
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 518-518 / 第 518-518 行

~~~~python
    warps_per_row = max(threads_per_row // 32, 1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 520-520 / 第 520-520 行

~~~~python
    # Step 3 & 4: Block or cluster reduction (if needed)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 521-532 / 第 521-532 行

~~~~python
    if cutlass.const_expr(warps_per_row > 1 or cluster_n > 1):
        if cutlass.const_expr(cluster_n == 1):
            # Single CTA: use block reduction
            return block_reduce(warp_val, warp_op, reduction_buffer, init_val)
        else:
            # Multiple CTAs: use cluster reduction
            return cluster_reduce(
                warp_val, warp_op, reduction_buffer, mbar_ptr, cluster_n, init_val
            )
    else:
        # Single warp handles entire row: warp reduction is sufficient
        return warp_val
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出
- Multi-GPU communication / 多 GPU 通信

## Dependencies / 依赖项

- `operator` — used by this example / 供该示例使用
- `collections.abc.Callable` — used by this example / 供该示例使用
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.Float32` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.Int32` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass._mlir.dialects.llvm` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cutlass_dsl.T` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.dsl_user_op` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
