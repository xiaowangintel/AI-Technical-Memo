# cute_dsl_jax_kernels.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/jax/cute_dsl_jax_kernels.py`  
**Purpose / 用途**: Tutorial example showing cute dsl jax kernels in CuTeDSL. / 这是一个关于 cute dsl jax kernels 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-32 / 第 29-32 行

~~~~python
import cutlass
import cutlass.cute as cute
import cutlass.jax as cjax
import cuda.bindings.driver as cuda
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 34-66 / 第 34-66 行

~~~~python
"""
CuTe DSL kernels used by the ``cute_dsl_jax.ipynb`` notebook.

This module defines GPU kernels written in CuTe DSL (CUTLASS 4.x Python DSL)
that are called from JAX via ``cutlass.jax.cutlass_call``. ``cutlass_call`` is a
JAX primitive that triggers compilation of the kernel during lowering and embeds
it into the HLO computation, so XLA can launch it efficiently without callback
to Python.

Kernels provided:

- ``vector_add``        — element-wise c = a + b (3-D CuTe layout)
- ``saxpy``             — y = alpha * x + y
- ``relu``              — element-wise ReLU with flat indexing
- ``fused_bias_relu``   — fused bias addition + ReLU
- ``gemm``              — tiled matrix multiplication
- ``elementwise_add``   — 2-D element-wise add (flat indexing, ``jax.export``-compatible)

The notebook imports these kernels and wraps each one with ``cutlass_call``
inside ``@jax.jit`` functions. See ``cute_dsl_jax.ipynb`` for usage, validation,
and step-by-step explanations.

This module is imported by the notebook and by ``cute_dsl_jax.py``. It can also
be run directly to validate every kernel:

.. code-block:: bash

    # Interactive notebook (recommended for learning)
    jupyter lab cute_dsl_jax.ipynb

    # Full demo as a standalone script
    python cute_dsl_jax_kernels.py
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 69-71 / 第 69-71 行

~~~~python
# ------------------------------------------------------------------ #
#  Vector Add: c = a + b                                             #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 72-76 / 第 72-76 行

~~~~python
@cute.kernel
def vector_add_kernel(a: cute.Tensor, b: cute.Tensor, c: cute.Tensor):
    """Per-thread kernel: each thread adds one element."""
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `vector_add_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `vector_add_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 78-80 / 第 78-80 行

~~~~python
    frgA = cute.make_rmem_tensor(cute.size(a, mode=[0]), a.element_type)
    frgB = cute.make_rmem_tensor(cute.size(b, mode=[0]), b.element_type)
    frgC = cute.make_rmem_tensor(cute.size(c, mode=[0]), c.element_type)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 82-85 / 第 82-85 行

~~~~python
    cute.autovec_copy(a[None, tidx, bidx], frgA)
    cute.autovec_copy(b[None, tidx, bidx], frgB)
    frgC.store(frgA.load() + frgB.load())
    cute.autovec_copy(frgC, c[None, tidx, bidx])
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 88-97 / 第 88-97 行

~~~~python
@cute.jit
def launch_vector_add(
    stream: cuda.CUstream,
    a: cute.Tensor, b: cute.Tensor, c: cute.Tensor,
):
    vector_add_kernel(a, b, c).launch(
        grid=[a.shape[-1], 1, 1],
        block=[a.shape[-2], 1, 1],
        stream=stream,
    )
~~~~

**EN**: Marks `launch_vector_add` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `launch_vector_add` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 100-102 / 第 100-102 行

~~~~python
# ------------------------------------------------------------------ #
#  SAXPY: y = alpha * x + y                                          #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 103-107 / 第 103-107 行

~~~~python
@cute.kernel
def saxpy_kernel(x: cute.Tensor, y: cute.Tensor, out: cute.Tensor, alpha: float):
    """SAXPY: out[i] = alpha * x[i] + y[i]."""
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `saxpy_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `saxpy_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 109-111 / 第 109-111 行

~~~~python
    frgX = cute.make_rmem_tensor(cute.size(x, mode=[0]), x.element_type)
    frgY = cute.make_rmem_tensor(cute.size(y, mode=[0]), y.element_type)
    frgO = cute.make_rmem_tensor(cute.size(out, mode=[0]), out.element_type)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 113-116 / 第 113-116 行

~~~~python
    cute.autovec_copy(x[None, tidx, bidx], frgX)
    cute.autovec_copy(y[None, tidx, bidx], frgY)
    frgO.store(alpha * frgX.load() + frgY.load())
    cute.autovec_copy(frgO, out[None, tidx, bidx])
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 119-129 / 第 119-129 行

~~~~python
@cute.jit
def launch_saxpy(
    stream: cuda.CUstream,
    x: cute.Tensor, y: cute.Tensor, out: cute.Tensor,
    *, alpha: float,
):
    saxpy_kernel(x, y, out, alpha).launch(
        grid=[x.shape[-1], 1, 1],
        block=[x.shape[-2], 1, 1],
        stream=stream,
    )
~~~~

**EN**: Marks `launch_saxpy` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `launch_saxpy` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 132-134 / 第 132-134 行

~~~~python
# ------------------------------------------------------------------ #
#  ReLU: out = max(0, x)                                             #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 135-140 / 第 135-140 行

~~~~python
@cute.kernel
def relu_kernel(x: cute.Tensor, out: cute.Tensor, N: int):
    """Per-thread kernel: each thread computes ReLU of one element."""
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
    bdx, _, _ = cute.arch.block_dim()
~~~~

**EN**: Declares `relu_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `relu_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 142-145 / 第 142-145 行

~~~~python
    idx = bidx * bdx + tidx
    if idx < N:
        val = x[idx]
        out[idx] = cutlass.max(val, cutlass.Float32(0.0))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 148-160 / 第 148-160 行

~~~~python
@cute.jit
def launch_relu(
    stream: cuda.CUstream,
    x: cute.Tensor, out: cute.Tensor,
    *, N: int,
):
    BLOCK_SIZE = 256
    grid_size = (N + BLOCK_SIZE - 1) // BLOCK_SIZE
    relu_kernel(x, out, N).launch(
        grid=[grid_size, 1, 1],
        block=[BLOCK_SIZE, 1, 1],
        stream=stream,
    )
~~~~

**EN**: Marks `launch_relu` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `launch_relu` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 163-165 / 第 163-165 行

~~~~python
# ------------------------------------------------------------------ #
#  Fused Bias + ReLU: out = max(0, x + bias[col])                    #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 166-173 / 第 166-173 行

~~~~python
@cute.kernel
def fused_bias_relu_kernel(
    x: cute.Tensor, bias: cute.Tensor, out: cute.Tensor, N: int, width: int,
):
    """Per-thread: out[i] = max(0, x[i] + bias[i % width])."""
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
    bdx, _, _ = cute.arch.block_dim()
~~~~

**EN**: Declares `fused_bias_relu_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `fused_bias_relu_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 175-179 / 第 175-179 行

~~~~python
    idx = bidx * bdx + tidx
    if idx < N:
        col = idx % width
        val = x[idx] + bias[col]
        out[idx] = cutlass.max(val, cutlass.Float32(0.0))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 182-194 / 第 182-194 行

~~~~python
@cute.jit
def launch_fused_bias_relu(
    stream: cuda.CUstream,
    x: cute.Tensor, bias: cute.Tensor, out: cute.Tensor,
    *, N: int, width: int,
):
    BLOCK_SIZE = 256
    grid_size = (N + BLOCK_SIZE - 1) // BLOCK_SIZE
    fused_bias_relu_kernel(x, bias, out, N, width).launch(
        grid=[grid_size, 1, 1],
        block=[BLOCK_SIZE, 1, 1],
        stream=stream,
    )
~~~~

**EN**: Marks `launch_fused_bias_relu` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `launch_fused_bias_relu` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 197-199 / 第 197-199 行

~~~~python
# ------------------------------------------------------------------ #
#  GEMM: D = A @ B                                                   #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 200-208 / 第 200-208 行

~~~~python
@cute.kernel
def gemm_kernel(
    A: cute.Tensor, B: cute.Tensor, D: cute.Tensor,
    M: int, N: int, K: int, BLOCK_M: int, BLOCK_N: int,
):
    """Tiled GEMM: each thread accumulates output elements."""
    tidx, _, _ = cute.arch.thread_idx()
    bm, bn, _ = cute.arch.block_idx()
    bdx, _, _ = cute.arch.block_dim()
~~~~

**EN**: Declares `gemm_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `gemm_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 210-219 / 第 210-219 行

~~~~python
    for i in cutlass.range(tidx, BLOCK_M * BLOCK_N, bdx):
        row = i // BLOCK_N
        col = i % BLOCK_N
        m_idx = bm * BLOCK_M + row
        n_idx = bn * BLOCK_N + col
        if m_idx < M and n_idx < N:
            acc = cutlass.Float32(0.0)
            for k in cutlass.range(K):
                acc += A[m_idx * K + k] * B[k * N + n_idx]
            D[m_idx * N + n_idx] = acc
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 222-235 / 第 222-235 行

~~~~python
@cute.jit
def launch_gemm(
    stream: cuda.CUstream,
    A: cute.Tensor, B: cute.Tensor, D: cute.Tensor,
    *, M: int, N: int, K: int,
):
    BLOCK_M, BLOCK_N = 64, 64
    grid_m = (M + BLOCK_M - 1) // BLOCK_M
    grid_n = (N + BLOCK_N - 1) // BLOCK_N
    gemm_kernel(A, B, D, M, N, K, BLOCK_M, BLOCK_N).launch(
        grid=[grid_m, grid_n, 1],
        block=[256, 1, 1],
        stream=stream,
    )
~~~~

**EN**: Marks `launch_gemm` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `launch_gemm` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 238-240 / 第 238-240 行

~~~~python
# ------------------------------------------------------------------ #
#  Element-wise Add (2-D, flat indexing)                             #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 241-246 / 第 241-246 行

~~~~python
@cute.kernel
def elementwise_add_kernel(gA: cute.Tensor, gB: cute.Tensor, gC: cute.Tensor):
    """Per-thread kernel: 2-D element-wise add using flat indexing."""
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
    bdim, _, _ = cute.arch.block_dim()
~~~~

**EN**: Declares `elementwise_add_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `elementwise_add_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 248-248 / 第 248-248 行

~~~~python
    thread_idx = bidx * bdim + tidx
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 250-252 / 第 250-252 行

~~~~python
    m, n = gA.shape
    ni = thread_idx % n
    mi = thread_idx // n
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 254-256 / 第 254-256 行

~~~~python
    a_val = gA[mi, ni]
    b_val = gB[mi, ni]
    gC[mi, ni] = a_val + b_val
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 259-270 / 第 259-270 行

~~~~python
@cute.jit
def launch_elementwise_add(
    stream: cuda.CUstream,
    mA: cute.Tensor, mB: cute.Tensor, mC: cute.Tensor,
):
    num_threads_per_block = 256
    m, n = mA.shape
    elementwise_add_kernel(mA, mB, mC).launch(
        grid=((m * n) // num_threads_per_block, 1, 1),
        block=(num_threads_per_block, 1, 1),
        stream=stream,
    )
~~~~

**EN**: Marks `launch_elementwise_add` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `launch_elementwise_add` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 273-275 / 第 273-275 行

~~~~python
# ------------------------------------------------------------------ #
#  Self-tests                                                         #
# ------------------------------------------------------------------ #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 276-278 / 第 276-278 行

~~~~python
if __name__ == '__main__':
    import os
    os.environ.setdefault("TF_CPP_MIN_LOG_LEVEL", "2")
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 280-282 / 第 280-282 行

~~~~python
    import jax
    import jax.numpy as jnp
    import numpy as np
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 284-285 / 第 284-285 行

~~~~python
    BLOCK = 256
    N_BLOCKS = 4
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 287-288 / 第 287-288 行

~~~~python
    # ── Vector Add ────────────────────────────────────────────────────
    # 3-D CuTe layout: (elems_per_thread, threads_per_block, num_blocks)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 289-298 / 第 289-298 行

~~~~python
    a = jax.random.normal(jax.random.PRNGKey(0), (1, BLOCK, N_BLOCKS), dtype=jnp.float32)
    b = jax.random.normal(jax.random.PRNGKey(1), (1, BLOCK, N_BLOCKS), dtype=jnp.float32)
    call = cjax.cutlass_call(
        launch_vector_add,
        output_shape_dtype=jax.ShapeDtypeStruct(a.shape, a.dtype),
        use_static_tensors=True,
    )
    c = jax.jit(call)(a, b)
    np.testing.assert_allclose(np.array(c), np.array(a + b), rtol=1e-5, atol=1e-5)
    print('vector_add:       PASSED')
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 300-300 / 第 300-300 行

~~~~python
    # ── SAXPY ─────────────────────────────────────────────────────────
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 301-312 / 第 301-312 行

~~~~python
    x = jax.random.normal(jax.random.PRNGKey(2), (1, BLOCK, N_BLOCKS), dtype=jnp.float32)
    y = jax.random.normal(jax.random.PRNGKey(3), (1, BLOCK, N_BLOCKS), dtype=jnp.float32)
    alpha = 2.5
    call = cjax.cutlass_call(
        launch_saxpy,
        output_shape_dtype=jax.ShapeDtypeStruct(x.shape, x.dtype),
        use_static_tensors=True,
        alpha=alpha,
    )
    out = jax.jit(call)(x, y)
    np.testing.assert_allclose(np.array(out), np.array(alpha * x + y), rtol=1e-5, atol=1e-5)
    print('saxpy:            PASSED')
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 314-314 / 第 314-314 行

~~~~python
    # ── ReLU ──────────────────────────────────────────────────────────
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 315-324 / 第 315-324 行

~~~~python
    N_ELEM = BLOCK * N_BLOCKS
    x = jax.random.normal(jax.random.PRNGKey(4), (N_ELEM,), dtype=jnp.float32)
    call = cjax.cutlass_call(
        launch_relu,
        output_shape_dtype=jax.ShapeDtypeStruct(x.shape, x.dtype),
        N=N_ELEM,
    )
    out = jax.jit(call)(x)
    np.testing.assert_allclose(np.array(out), np.array(jnp.maximum(x, 0)), rtol=1e-5, atol=1e-5)
    print('relu:             PASSED')
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 326-326 / 第 326-326 行

~~~~python
    # ── Fused Bias + ReLU ─────────────────────────────────────────────
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 327-338 / 第 327-338 行

~~~~python
    ROWS, COLS = 16, 64
    x = jax.random.normal(jax.random.PRNGKey(5), (ROWS * COLS,), dtype=jnp.float32)
    bias = jax.random.normal(jax.random.PRNGKey(6), (COLS,), dtype=jnp.float32)
    call = cjax.cutlass_call(
        launch_fused_bias_relu,
        output_shape_dtype=jax.ShapeDtypeStruct(x.shape, x.dtype),
        N=ROWS * COLS, width=COLS,
    )
    out = jax.jit(call)(x, bias)
    ref = jnp.maximum(x.reshape(ROWS, COLS) + bias, 0).reshape(-1)
    np.testing.assert_allclose(np.array(out), np.array(ref), rtol=1e-5, atol=1e-5)
    print('fused_bias_relu:  PASSED')
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 340-340 / 第 340-340 行

~~~~python
    # ── GEMM ──────────────────────────────────────────────────────────
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 341-352 / 第 341-352 行

~~~~python
    M, N, K = 128, 128, 64
    A = jax.random.normal(jax.random.PRNGKey(7), (M * K,), dtype=jnp.float32)
    B = jax.random.normal(jax.random.PRNGKey(8), (K * N,), dtype=jnp.float32)
    call = cjax.cutlass_call(
        launch_gemm,
        output_shape_dtype=jax.ShapeDtypeStruct((M * N,), A.dtype),
        M=M, N=N, K=K,
    )
    D = jax.jit(call)(A, B)
    ref = A.reshape(M, K) @ B.reshape(K, N)
    np.testing.assert_allclose(np.array(D.reshape(M, N)), np.array(ref), rtol=1e-2, atol=1e-2)
    print('gemm:             PASSED')
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 354-354 / 第 354-354 行

~~~~python
    # ── Elementwise Add (2-D) ─────────────────────────────────────────
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 355-364 / 第 355-364 行

~~~~python
    M, N = 16, 256
    a = jax.random.normal(jax.random.PRNGKey(9), (M, N), dtype=jnp.float32)
    b = jax.random.normal(jax.random.PRNGKey(10), (M, N), dtype=jnp.float32)
    call = cjax.cutlass_call(
        launch_elementwise_add,
        output_shape_dtype=jax.ShapeDtypeStruct(a.shape, a.dtype),
    )
    c = jax.jit(call)(a, b)
    np.testing.assert_allclose(np.array(c), np.array(a + b), rtol=1e-5, atol=1e-5)
    print('elementwise_add:  PASSED')
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 366-366 / 第 366-366 行

~~~~python
    print('\nAll kernels passed.')
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.jax` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `jax` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.numpy` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `numpy` — supplies host-side array utilities / 提供宿主端数组工具
