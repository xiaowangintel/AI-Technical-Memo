# elementwise_apply_example.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/jax/elementwise_apply_example.py`  
**Purpose / 用途**: Tutorial example showing elementwise apply example in CuTeDSL. / 这是一个关于 elementwise apply example 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 30-37 / 第 30-37 行

~~~~python
import argparse
import operator
from functools import partial
from typing import List

import cuda.bindings.driver as cuda
import cutlass
import cutlass.cute as cute
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 39-58 / 第 39-58 行

~~~~python
"""
An Elementwise Apply Example using CuTe DSL with cutlass.jax.cutlass_call

This example is similar to examples/ampere/elementwise_apply.py but demonstrates
how to run the code in a jax specific way using the cutlass_call primitive. It assumes
familiarity with basic CuTe DSL concepts as well as the cutlass_call primitive.

To run this example:

.. code-block:: bash

    # Run with addition operation
    python examples/jax/elementwise_apply_example.py --M 1024 --N 512 --op add

    # Run with multiplication operation
    python examples/ampere/elementwise_apply_example.py --M 1024 --N 512 --op mul

    # Run with subtraction operation
    python examples/ampere/elementwise_apply_example.py --M 1024 --N 512 --op sub
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 61-71 / 第 61-71 行

~~~~python
@cute.kernel
def elementwise_apply_kernel(
    op: cutlass.Constexpr,
    mInputs: List[cute.Tensor],
    mC: cute.Tensor,
    cC: cute.Tensor,  # coordinate tensor
    shape: cute.Shape,
    tv_layout: cute.Layout,  # (tid, vid) -> logic coord
):
    tidx, _, _ = cute.arch.thread_idx()
    bidx, bidy, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `elementwise_apply_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `elementwise_apply_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 73-75 / 第 73-75 行

~~~~python
    ###############################################################################
    # Slice to local tile of thread block
    ###############################################################################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 76-76 / 第 76-76 行

~~~~python
    blk_crd = ((None, None), (bidx, bidy))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 78-80 / 第 78-80 行

~~~~python
    # Leverage the meta-programming capability of the DSL to slice the tensors for each input
    # All for loops below on input tensors would be fully unrolled automatically at compile time
    # logical coord -> memory address
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 81-83 / 第 81-83 行

~~~~python
    gInputs = [t[blk_crd] for t in mInputs]  # (TileM, TileN)
    gC = mC[blk_crd]  # (TileM, TileN)
    gCrd = cC[blk_crd]  # (TileM, TileN)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 85-89 / 第 85-89 行

~~~~python
    print("[DSL INFO] Sliced Tensors per thread block:")
    for i in cutlass.range_constexpr(len(gInputs)):
        print(f"[DSL INFO]   ctaInputs{i} = {gInputs[i].type}")
    print(f"[DSL INFO]   gC = {gC.type}")
    print(f"[DSL INFO]   gCrd = {gCrd.type}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 91-94 / 第 91-94 行

~~~~python
    ###############################################################################
    # Compose with thread block TV layout to map thread & value indices to memory address
    ###############################################################################
    # (tid, vid) -> memory address
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 95-97 / 第 95-97 行

~~~~python
    tidfrgInputs = [cute.composition(t, tv_layout) for t in gInputs]
    tidfrgC = cute.composition(gC, tv_layout)
    tidfrgCrd = cute.composition(gCrd, tv_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 99-99 / 第 99-99 行

~~~~python
    # repeat None like vid to remove hierarchy of layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 100-100 / 第 100-100 行

~~~~python
    thr_crd = (tidx, cute.repeat_like(None, tidfrgInputs[0][1]))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 102-105 / 第 102-105 行

~~~~python
    ###############################################################################
    # Slice to local tile of thread
    ###############################################################################
    # vid -> address
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 106-108 / 第 106-108 行

~~~~python
    thrInputs = [t[thr_crd] for t in tidfrgInputs]  # (V)
    thrC = tidfrgC[thr_crd]  # (V)
    thrCrd = tidfrgCrd[thr_crd]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 110-114 / 第 110-114 行

~~~~python
    print("[DSL INFO] Sliced Tensors per thread:")
    for i in cutlass.range_constexpr(len(thrInputs)):
        print(f"[DSL INFO]   thrInputs{i} = {thrInputs[i].type}")
    print(f"[DSL INFO]   thrC = {thrC.type}")
    print(f"[DSL INFO]   thrCrd = {thrCrd.type}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 116-118 / 第 116-118 行

~~~~python
    ###############################################################################
    # Compute predicate for out of boundary checks
    ###############################################################################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 119-120 / 第 119-120 行

~~~~python
    frgPred = cute.make_fragment(thrCrd.shape, cutlass.Boolean)
    print(f"[DSL INFO]   frgPred = {frgPred.type}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 122-123 / 第 122-123 行

~~~~python
    for i in cutlass.range_constexpr(cute.size(frgPred)):
        frgPred[i] = cute.elem_less(thrCrd[i], shape)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 125-133 / 第 125-133 行

~~~~python
    # if tidx == 0 and bidx == 0:
    #     cute.print_tensor(frgPred)

    ##########################################################
    # Load data and compute result
    ##########################################################

    # Load data before use. The compiler will optimize the copy and load
    # operations to convert some memory ld/st into register uses.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 134-135 / 第 134-135 行

~~~~python
    result = op(*[thrInput.load() for thrInput in thrInputs])
    thrC.store(result)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 138-155 / 第 138-155 行

~~~~python
@cute.jit
def elementwise_apply(
    op: cutlass.Constexpr, inputs, result: cute.Tensor, stream: cuda.CUstream
):
    """CUDA kernel applying binary operator on each element of two n-D input tensors in
    CuTe Python and store to result tensor.

    :param op: Binary operator or lambda function to apply element-wise
    :type op: cutlass.Constexpr
    :param a: First input tensor
    :type a: cute.Tensor
    :param b: Second input tensor
    :type b: cute.Tensor
    :param result: Output tensor to store the results of op(a, b)
    :type result: cute.Tensor
    :return: None
    :rtype: None
    """
~~~~

**EN**: Marks `elementwise_apply` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `elementwise_apply` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 157-184 / 第 157-184 行

~~~~python
    # Baseline: naive TV layout
    #   * mA layout: (4096, 4096):(4096, 1)
    #   * TV layout map to (512, 4) tile
    #   * tidx maps to mode-0 but input layout is contiguous on mode-1, performance will be bad
    # tv_layout = cute.make_layout((128, (4, 4)), stride=(4, (512, 1)))
    # cta_tiler = (512, 4)

    # Opt-1: better TV layout with better 1D thread layout (SOL with 1D thread layout)
    #   * mA layout: (4096, 4096):(4096, 1)
    #   * TV layout map to (4, 512) tile
    #   * tidx maps to mode-1 which is leading mode of input tensor for coalesced load
    # tv_layout = cute.make_layout((128, (4, 4)), stride=(16, (4, 1)))
    # cta_tiler = (4, 512)

    # Opt-2: 2D tile but worse
    #   * mA layout: (4096, 4096):(4096, 1)
    #   * TV layout map to (128, 16) logical tile
    #   * V layout is bad as contiguous mode is not on right-most
    #     * `cute.copy` only supports vectorize when stride-1 of v-layout on right-most )
    # tv_layout = cute.make_layout(((32, 4), (4, 4)), stride=((4, 512), (1, 128)))
    # cta_tiler = (128, 16)

    # Opt-3: SOL with 2D thread tile
    #   * mA layout: (4096, 4096):(4096, 1)
    #   * TV layout map to (64, 256) logical tile
    #   * tidx maps to mode-1 and input layout is contiguous on mode-1 for coalesced load-store

    # Use 128bit(16B) load as canonicalized form of val_layout then recast to target element-type
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 185-185 / 第 185-185 行

~~~~python
    coalesced_ldst_bytes = 16
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 187-187 / 第 187-187 行

~~~~python
    # Compile time validation: expect same element type for all input tensors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 188-189 / 第 188-189 行

~~~~python
    assert all(t.element_type == inputs[0].element_type for t in inputs)
    dtype = inputs[0].element_type
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 191-194 / 第 191-194 行

~~~~python
    thr_layout = cute.make_ordered_layout((4, 64), order=(1, 0))
    val_layout = cute.make_ordered_layout((16, coalesced_ldst_bytes), order=(1, 0))
    val_layout = cute.recast_layout(dtype.width, 8, val_layout)
    tiler_mn, tv_layout = cute.make_layout_tv(thr_layout, val_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 196-199 / 第 196-199 行

~~~~python
    print("[DSL INFO] Input Tensors:")
    for i, t in enumerate(inputs):
        print(f"[DSL INFO]   inputs{i} = {t}")
    print(f"[DSL INFO]   result = {result}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 201-203 / 第 201-203 行

~~~~python
    print("[DSL INFO] Tiling Parameters:")
    print(f"[DSL INFO]   tiler_mn = {tiler_mn} per thread block")
    print(f"[DSL INFO]   tv_layout = {tv_layout}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 205-208 / 第 205-208 行

~~~~python
    print("[DSL INFO] Tiled Tensors:")
    mInputs = [cute.zipped_divide(input, tiler_mn) for input in inputs]
    # ((TileM, TileN), (RestM, RestN))
    mC = cute.zipped_divide(result, tiler_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 210-210 / 第 210-210 行

~~~~python
    # (RestM, RestN) -> (RestN, RestM)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 211-217 / 第 211-217 行

~~~~python
    remap_block = cute.make_ordered_layout(
        cute.select(mInputs[0].shape[1], mode=[1, 0]), order=(1, 0)
    )
    for i, t in enumerate(mInputs):
        print(f"[DSL INFO]   gInputs{i}            = {mInputs[i]}")
        mInputs[i] = cute.composition(t, (None, remap_block))
        print(f"[DSL INFO]   gInputs{i} (remapped) = {mInputs[i]}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 219-220 / 第 219-220 行

~~~~python
    mC = cute.composition(mC, (None, remap_block))
    print(f"[DSL INFO]   gC = {mC}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 222-224 / 第 222-224 行

~~~~python
    idC = cute.make_identity_tensor(result.shape)
    cC = cute.zipped_divide(idC, tiler=tiler_mn)
    print(f"[DSL INFO]   coord tensor = {cC}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 226-227 / 第 226-227 行

~~~~python
    # Launch the kernel asynchronously
    # Group input tensors into a list as a single argument
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 228-233 / 第 228-233 行

~~~~python
    elementwise_apply_kernel(op, mInputs, mC, cC, result.shape, tv_layout).launch(
        # Compute production at each mode of mC.shape[1] to get multi-dimensional grid size
        grid=cute.product_each(mC.shape[1]),
        block=[cute.size(tv_layout, mode=[0]), 1, 1],
        stream=stream,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 236-238 / 第 236-238 行

~~~~python
@cutlass.dsl_user_op
def leaky_relu(x, alpha, *, loc=None, ip=None):
    return cute.where(x > 0, x, alpha * x, loc=loc, ip=ip)
~~~~

**EN**: Defines `leaky_relu`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `leaky_relu`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 241-242 / 第 241-242 行

~~~~python
def leaky_relu_ref(x, alpha):
    import jax.numpy as jnp
~~~~

**EN**: Defines `leaky_relu_ref`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `leaky_relu_ref`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 244-244 / 第 244-244 行

~~~~python
    return jnp.where(x > 0, x, alpha * x)
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 247-251 / 第 247-251 行

~~~~python
def run_and_verify(op, M, N, dtype, skip_ref_check=False):
    import jax
    import jax.numpy as jnp
    import cutlass.jax as cjax
    import cutlass.jax.testing as testing
~~~~

**EN**: Defines `run_and_verify`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `run_and_verify`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 253-260 / 第 253-260 行

~~~~python
    if op == "leaky_relu":
        op = partial(leaky_relu, alpha=0.01)
        ref_op = partial(leaky_relu_ref, alpha=0.01)
        num_inputs = 1
    else:
        op = getattr(operator, op)
        ref_op = op
        num_inputs = 2
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 262-263 / 第 262-263 行

~~~~python
    # This jax function is transformed using jax.jit to compile its contents
    # into an efficient HLO executable.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 264-275 / 第 264-275 行

~~~~python
    @partial(jax.jit, static_argnums=[1])
    def jax_function(inputs, op):
        call = cjax.cutlass_call(
            # Bind jax arguments to kernel signature
            lambda stream, inputs, output, *, op: elementwise_apply(
                op, inputs, output, stream
            ),
            # Specify output shape/dtype of result
            output_shape_dtype=jax.ShapeDtypeStruct(inputs[0].shape, inputs[0].dtype),
            # Pass static/constexpr values as kwargs
            op=op,
        )
~~~~

**EN**: Defines `jax_function`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `jax_function`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 277-277 / 第 277-277 行

~~~~python
        # Call the kernel!
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 278-278 / 第 278-278 行

~~~~python
        return call(inputs)
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 280-282 / 第 280-282 行

~~~~python
    @partial(jax.jit, static_argnums=[1])
    def jax_ref_function(inputs, op):
        return op(*inputs)
~~~~

**EN**: Defines `jax_ref_function`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `jax_ref_function`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 284-286 / 第 284-286 行

~~~~python
    print("\nRunning Elementwise Apply test with:")
    print(f"Tensor dimensions: [{M}, {N}]")
    print(f"Input and Output Data type: {dtype}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 288-290 / 第 288-290 行

~~~~python
    jax_dtype = cjax.cutlass_to_jax_dtype(dtype)
    keys = jax.random.split(jax.random.key(1435), num_inputs)
    inputs = [testing.create_tensor((M, N), jax_dtype, key) for key in keys]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 292-294 / 第 292-294 行

~~~~python
    print("Input tensor shapes:")
    for i in range(num_inputs):
        print(f"inputs[{i}]: {inputs[i].shape}, dtype: {inputs[i].dtype}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 296-298 / 第 296-298 行

~~~~python
    epsilon = 1.2
    if op in (operator.truediv, operator.floordiv):
        inputs[1] = jnp.where(inputs[1] == 0, epsilon, inputs[1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 300-300 / 第 300-300 行

~~~~python
    # Call the jax.jit function which will compile the kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 301-301 / 第 301-301 行

~~~~python
    c = jax_function(inputs, op)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 303-309 / 第 303-309 行

~~~~python
    if not skip_ref_check:
        print("Executing elementwise apply kernel...")
        c = jax_function(inputs, op)
        print("Verifying results...")
        assert jnp.allclose(ref_op(*inputs), c)
        print("Results verified successfully!")
        print(f"First few elements of result: \n{c[:3, :3]}")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 312-319 / 第 312-319 行

~~~~python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Demonstration of calling a kernel with cutlass_call"
    )
    parser.add_argument("--M", default=4096, type=int)
    parser.add_argument("--N", default=4096, type=int)
    parser.add_argument("--op", default="add", type=str)
    parser.add_argument("--skip_ref_check", action="store_true")
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 321-329 / 第 321-329 行

~~~~python
    args = parser.parse_args()
    run_and_verify(
        args.op,
        args.M,
        args.N,
        dtype=cutlass.Float32,
        skip_ref_check=args.skip_ref_check,
    )
    print("\nPASS")
~~~~

**EN**: Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Ampere GPU execution model / Ampere GPU 执行模型
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `operator` — used by this example / 供该示例使用
- `functools.partial` — supplies small Python helpers such as partial application / 提供 partial 等 Python 辅助能力
- `typing.List` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `jax.numpy` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `cutlass.jax` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.jax.testing` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
