# programmatic_dependent_launch.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/programmatic_dependent_launch.py`  
**Purpose / 用途**: Tutorial example showing programmatic dependent launch in CuTeDSL. / 这是一个关于 programmatic dependent launch 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 30-36 / 第 30-36 行

~~~~python
import argparse
import cuda.bindings.driver as cuda

import cutlass
import cutlass.cute as cute
import cutlass.cute.testing as testing
from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 39-40 / 第 39-40 行

~~~~python
def supports_pdl():
    import torch
~~~~

**EN**: Defines `supports_pdl`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `supports_pdl`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 42-42 / 第 42-42 行

~~~~python
    return torch.cuda.get_device_capability()[0] >= 9
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 45-120 / 第 45-120 行

~~~~python
"""
This example demonstrates the use of Programmatic Dependent Launch (PDL) using
CuTe DSL.

PDL is a mechanism which allows for overlapping execution of back-to-back kernels
within the same stream.
For example, consider the following two elementwise add operations, where the second
operation's first operand is the result of the first operation. While performing
``w = u + v`` we will load u and v, add them, and then store the result. Once we
have finished loading data, we are no longer utilizing the read bandwidth.
To effectively utilize the read bandwidth, we can start loading ``x``
immediately upon finishing reading. This is what PDL enables us to do.

.. code-block:: bash

w = u + v
y = w + x

To enable PDL, we need to do two things:

1. Insert the ``griddepcontrol.launch_dependents`` and ``griddepcontrol.wait`` instructions in the kernel.
2. Set the PDL launch attribute when launching the kernel.

The ``griddepcontrol.launch_dependents`` and ``griddepcontrol.wait``
instructions enable fine-grained control over kernel execution in PDL.
Once all thread blocks execute the ``griddepcontrol.launch_dependents``
instruction, the dependent kernels can opportunistically be early-launched.
``griddepcontrol.wait`` functions as a synchronization barrier - any warp
executing this instruction will block until the previous kernel finishes
execution. This allows precise control over data dependencies between kernels.

The following diagram shows the overlapping execution of two dependent kernels.
We call the instructions before ``griddepcontrol.wait`` as prologue (``P0``),
which may include barrier initialization and loading of independent data, etc.
We call the instructions after ``griddepcontrol.launch_dependents`` as epilogue
(``P2``), which may include math operations, data stores, etc. PDL enables
these prologue and epilogue phases to execute concurrently across dependent
kernels, improving GPU resource utilization. This is particularly beneficial
when prologue and epilogue are bound by different resources (e.g., memory
bandwidth vs compute throughput).

      # P0: Prologue, P1: Main compute, P2: Epilogue

         P0    P1    P2
  K1: |=====|+++++|-----|

                  <-----> K2 can start early
                         (K1's P2 overlaps with K2's P0)

                     P0        P1    P2
  K2:             |=====|   |+++++|-----|
                          ^
                          |
                          wait for K1 to complete
Time ------------------------------------------------------>

We could run this example with and without PDL:

.. code-block:: bash

    python examples/blackwell/programmatic_dependent_launch.py --benchmark
    python examples/blackwell/programmatic_dependent_launch.py --benchmark --use_pdl

From the benchmark results, you can see some speedups for the PDL version in most cases, benefiting from
the overlapping execution of consecutive kernels. Moreover, you can use nsys to observe the overlapping execution.

.. code-block:: bash

    nsys profile python examples/blackwell/programmatic_dependent_launch.py --benchmark --use_pdl

Note, PDL feature is supported on Hopper and later GPUs.

See [the programming guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization)
and the [PTX documentation](https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#parallel-synchronization-and-communication-instructions-griddepcontrol)
for more details.
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 123-135 / 第 123-135 行

~~~~python
@cute.kernel
def elementwise_add_kernel(
    gA: cute.Tensor,
    gB: cute.Tensor,
    gC: cute.Tensor,
    cC: cute.Tensor,  # coordinate tensor
    shape: cute.Shape,
    thr_layout: cute.Layout,
    val_layout: cute.Layout,
    is_first_kernel: cutlass.Constexpr = True,
):
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `elementwise_add_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `elementwise_add_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 137-141 / 第 137-141 行

~~~~python
    blk_coord = ((None, None), bidx)
    blkA = gA[blk_coord]  # (TileM,TileN)
    blkB = gB[blk_coord]  # (TileM,TileN)
    blkC = gC[blk_coord]  # (TileM,TileN)
    blkCrd = cC[blk_coord]  # (TileM, TileN)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 143-144 / 第 143-144 行

~~~~python
    copy_atom_load = cute.make_copy_atom(cute.nvgpu.CopyUniversalOp(), gA.element_type)
    copy_atom_store = cute.make_copy_atom(cute.nvgpu.CopyUniversalOp(), gC.element_type)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 146-148 / 第 146-148 行

~~~~python
    tiled_copy_A = cute.make_tiled_copy_tv(copy_atom_load, thr_layout, val_layout)
    tiled_copy_B = cute.make_tiled_copy_tv(copy_atom_load, thr_layout, val_layout)
    tiled_copy_C = cute.make_tiled_copy_tv(copy_atom_store, thr_layout, val_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 150-152 / 第 150-152 行

~~~~python
    thr_copy_A = tiled_copy_A.get_slice(tidx)
    thr_copy_B = tiled_copy_B.get_slice(tidx)
    thr_copy_C = tiled_copy_C.get_slice(tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 154-156 / 第 154-156 行

~~~~python
    thrA = thr_copy_A.partition_S(blkA)
    thrB = thr_copy_B.partition_S(blkB)
    thrC = thr_copy_C.partition_S(blkC)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 158-160 / 第 158-160 行

~~~~python
    frgA = cute.make_fragment_like(thrA)
    frgB = cute.make_fragment_like(thrB)
    frgC = cute.make_fragment_like(thrC)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 162-163 / 第 162-163 行

~~~~python
    thrCrd = thr_copy_C.partition_S(blkCrd)
    frgPred = cute.make_rmem_tensor(thrCrd.shape, cutlass.Boolean)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 165-167 / 第 165-167 行

~~~~python
    for i in range(cute.size(frgPred)):
        val = cute.elem_less(thrCrd[i], shape)
        frgPred[i] = val
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 169-172 / 第 169-172 行

~~~~python
    # Note: when not using cuda-graph, the kernel execution may be blocked by the host overhead.
    # In this case we won't see overlapping even when pdl is enabled.
    # In this example, we add a loop (10 times) for all the copy and compute operations in the following code
    # to make kernel running longer and make pdl benefits observable for both cuda-graph enabled and disabled cases.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 173-186 / 第 173-186 行

~~~~python
    if is_first_kernel:
        for _ in range(10):
            cute.copy(copy_atom_load, thrA, frgA, pred=frgPred)
            cute.copy(copy_atom_load, thrB, frgB, pred=frgPred)
        # Here we add the launch dependents instruction for the first kernel as a hint to the runtime to early-launch
        # the next kernel. If the next kernel becomes concurrent, we will have overlap where the second kernel
        # can start reading x to ensure an E2E speedup. Note the placement of launch dependents has no implication
        # on correctness, only performance.
        cute.arch.griddepcontrol_launch_dependents()
    else:
        # In this example, the second kernel's second operand ``gB`` has no dependencies, its loading can overlap
        # with the computation of ``gC`` from the first kernel.
        for _ in range(10):
            cute.copy(copy_atom_load, thrB, frgB, pred=frgPred)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 188-191 / 第 188-191 行

~~~~python
        # For the second kernel, its first operand ``gA`` is dependent on the previous kernel, we must call
        # griddepcontrol.wait to assure correctness. This instruction will block until the prior kernels finishes
        # and its memory operations are visible. Since gA is written by the prior kernel, this will block until gA
        # is visible to our kernel. Without it, we would have undefined behavior due to a race condition.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 192-192 / 第 192-192 行

~~~~python
        cute.arch.griddepcontrol_wait()
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 194-195 / 第 194-195 行

~~~~python
        for _ in range(10):
            cute.copy(copy_atom_load, thrA, frgA, pred=frgPred)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 197-200 / 第 197-200 行

~~~~python
    for _ in range(10):
        result = frgA.load() + frgB.load()
        frgC.store(result)
    cute.copy(copy_atom_store, frgC, thrC, pred=frgPred)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 203-214 / 第 203-214 行

~~~~python
@cute.jit
def elementwise_add(
    mA,
    mB,
    mC,
    stream: cuda.CUstream,
    use_pdl: cutlass.Constexpr = True,
    is_first_kernel: cutlass.Constexpr = True,
):
    dtype = mA.element_type
    # copy_bits for a thread is 128 bits, and we use 128 // dtype.width to get the vector size
    vector_size = 128 // dtype.width
~~~~

**EN**: Marks `elementwise_add` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `elementwise_add` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 216-218 / 第 216-218 行

~~~~python
    thr_layout = cute.make_ordered_layout((4, 32), order=(1, 0))
    val_layout = cute.make_ordered_layout((4, vector_size), order=(1, 0))
    tiler_mn, tv_layout = cute.make_layout_tv(thr_layout, val_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 220-222 / 第 220-222 行

~~~~python
    gA = cute.zipped_divide(mA, tiler_mn)  # ((TileM,TileN),(RestM,RestN))
    gB = cute.zipped_divide(mB, tiler_mn)  # ((TileM,TileN),(RestM,RestN))
    gC = cute.zipped_divide(mC, tiler_mn)  # ((TileM,TileN),(RestM,RestN))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 224-225 / 第 224-225 行

~~~~python
    idC = cute.make_identity_tensor(mC.shape)
    cC = cute.zipped_divide(idC, tiler=tiler_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 227-234 / 第 227-234 行

~~~~python
    elementwise_add_kernel(
        gA, gB, gC, cC, mC.shape, thr_layout, val_layout, is_first_kernel
    ).launch(
        grid=[cute.size(gC, mode=[1]), 1, 1],
        block=[cute.size(tv_layout, mode=[0]), 1, 1],
        stream=stream,
        use_pdl=use_pdl,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 237-246 / 第 237-246 行

~~~~python
def run_pdl_example(
    M,
    N,
    skip_ref_check=False,
    benchmark=False,
    warmup_iterations=5,
    iterations=100,
    use_pdl=True,
):
    import torch
~~~~

**EN**: Defines `run_pdl_example`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `run_pdl_example`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 248-249 / 第 248-249 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("Blackwell/Hopper GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 251-253 / 第 251-253 行

~~~~python
    print("\nRunning Elementwise Add test with:")
    print(f"Tensor dimensions: [{M}, {N}]")
    print(f"Use PDL: {use_pdl}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 255-259 / 第 255-259 行

~~~~python
    u = torch.randn(M, N, dtype=torch.float32, device="cuda")
    v = torch.randn(M, N, dtype=torch.float32, device="cuda")
    w = torch.randn(M, N, dtype=torch.float32, device="cuda")
    x = torch.randn(M, N, dtype=torch.float32, device="cuda")
    y = torch.empty(M, N, dtype=torch.float32, device="cuda")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 261-265 / 第 261-265 行

~~~~python
    u_tensor = from_dlpack(u).mark_layout_dynamic()
    v_tensor = from_dlpack(v).mark_layout_dynamic()
    w_tensor = from_dlpack(w).mark_layout_dynamic()
    x_tensor = from_dlpack(x).mark_layout_dynamic()
    y_tensor = from_dlpack(y).mark_layout_dynamic()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 267-290 / 第 267-290 行

~~~~python
    stream = torch.cuda.Stream()
    current_stream = cuda.CUstream(stream.cuda_stream)
    # Since is_first_kernel is cutlass.Constexpr, we need to compile for
    # the first and second kernel separately.
    compiled_func_first_kernel = cute.compile(
        elementwise_add,
        u_tensor,
        v_tensor,
        w_tensor,
        current_stream,
        use_pdl,
        is_first_kernel=True,
        options="--enable-tvm-ffi",
    )
    compiled_func_second_kernel = cute.compile(
        elementwise_add,
        w_tensor,
        x_tensor,
        y_tensor,
        current_stream,
        use_pdl,
        is_first_kernel=False,
        options="--enable-tvm-ffi",
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 292-292 / 第 292-292 行

~~~~python
    # launch and run the two consecutive kernels in a same stream.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 293-309 / 第 293-309 行

~~~~python
    def run_func(current_stream, u, v, w, x, y):
        # Run first operation: w_tensor = u_tensor + v_tensor
        compiled_func_first_kernel(
            u,
            v,
            w,
            current_stream,
        )
        # Run second operation: y_tensor = w_tensor + x_tensor
        # its first operand ``w_tensor`` is the result of the first operation,
        # they use the same memory space.
        compiled_func_second_kernel(
            w,
            x,
            y,
            current_stream,
        )
~~~~

**EN**: Defines `run_func`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `run_func`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 311-315 / 第 311-315 行

~~~~python
    if not skip_ref_check:
        run_func(current_stream, u, v, w, x, y)
        print("Verifying results...")
        torch.testing.assert_close(u.cpu() + v.cpu() + x.cpu(), y.cpu())
        print("Results verified successfully!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 317-318 / 第 317-318 行

~~~~python
    if not benchmark:
        return
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 320-325 / 第 320-325 行

~~~~python
    def generate_kernel_arguments():
        u = torch.randn(M, N, dtype=torch.float32, device="cuda")
        v = torch.randn(M, N, dtype=torch.float32, device="cuda")
        w = torch.randn(M, N, dtype=torch.float32, device="cuda")
        x = torch.randn(M, N, dtype=torch.float32, device="cuda")
        y = torch.empty(M, N, dtype=torch.float32, device="cuda")
~~~~

**EN**: Defines `generate_kernel_arguments`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `generate_kernel_arguments`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 327-327 / 第 327-327 行

~~~~python
        return testing.JitArguments(current_stream, u, v, w, x, y)
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 329-338 / 第 329-338 行

~~~~python
    avg_time_us = testing.benchmark(
        run_func,
        workspace_generator=generate_kernel_arguments,
        workspace_count=10,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
        stream=current_stream,
        use_cuda_graphs=True,
    )
    print(f"Execution time: {avg_time_us:.4f} us")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 341-351 / 第 341-351 行

~~~~python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="example of Programmatic Dependent Launch (PDL) using CuTe DSL"
    )
    parser.add_argument("--M", default=256, type=int)
    parser.add_argument("--N", default=256, type=int)
    parser.add_argument("--warmup_iterations", default=5, type=int)
    parser.add_argument("--iterations", default=10, type=int)
    parser.add_argument("--skip_ref_check", action="store_true")
    parser.add_argument("--benchmark", action="store_true")
    parser.add_argument("--use_pdl", action="store_true")
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 353-368 / 第 353-368 行

~~~~python
    args = parser.parse_args()
    if supports_pdl():
        run_pdl_example(
            args.M,
            args.N,
            skip_ref_check=args.skip_ref_check,
            benchmark=args.benchmark,
            warmup_iterations=args.warmup_iterations,
            iterations=args.iterations,
            use_pdl=args.use_pdl,
        )
        print("\nPASS")
    else:
        print(
            "PDL is not supported on this device, it requires Hopper or newer generations"
        )
~~~~

**EN**: Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Hopper GPU execution model / Hopper GPU 执行模型
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
