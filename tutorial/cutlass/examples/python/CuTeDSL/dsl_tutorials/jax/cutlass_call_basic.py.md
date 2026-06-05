# cutlass_call_basic.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/jax/cutlass_call_basic.py`  
**Purpose / 用途**: Tutorial example showing cutlass call basic in CuTeDSL. / 这是一个关于 cutlass call basic 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-35 / 第 29-35 行

~~~~python
from functools import partial
import jax
import jax.numpy as jnp

import cutlass.cute as cute
import cutlass.jax as cjax
import cuda.bindings.driver as cuda
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 37-54 / 第 37-54 行

~~~~python
"""
Examples of calling CuTe DSL from jax.jit function using cutlass_call.

cutlass_call is a Jax primitive the enables calling of CuTe DSL kernels within a
a jit-compiled Jax function. During the lowering process cutlass_call will
trigger compilation of the kernel and embed it into the HLO computation. It can
then be efficiently launched by XLA without callback to Python.

This example assumes familiarity with CuTe DSL concepts such as layouts and
dynamic shapes.

To run this example:

.. code-block:: bash

    # Run with addition operation
    python examples/jax/cutlass_call_basic.py
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 57-57 / 第 57-57 行

~~~~python
# This is a typical CuTe DSL kernel function that accepts both tensor and scalar values.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 58-77 / 第 58-77 行

~~~~python
@cute.jit
def launch(
    A: cute.Tensor,
    B: cute.Tensor,
    x: cute.Int32,
    y: cute.Int32,
    C: cute.Tensor,
    D: cute.Tensor,
    stream: cuda.CUstream,
):
    # Print layouts
    print("A layout: ", A.layout)
    print("B layout: ", B.layout)
    print("C layout: ", C.layout)
    print("D layout: ", D.layout)
    cute.printf("A layout: {}", A.layout)
    cute.printf("B layout: {}", B.layout)
    cute.printf("C layout: {}", C.layout)
    cute.printf("D layout: {}", D.layout)
    cute.printf("")
~~~~

**EN**: Marks `launch` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 将 `launch` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 79-79 / 第 79-79 行

~~~~python
    # Print non-tensor values
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 80-84 / 第 80-84 行

~~~~python
    print("X is: ", x)
    print("Y is: ", y)
    cute.printf("X is: {}", x)
    cute.printf("Y is: {}", y)
    print()
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 87-102 / 第 87-102 行

~~~~python
# cutlass_call uses a fixed function signature to pass arguments between Jax and CuTeDSL kernel.
#
# Function Signature Requirement:
#   stream, inputs, outputs, *, kwargs...
#
# The first argument must be the CUstream that the kernel is run. This stream is managed by the XLA runtime
# and is necessary to schedule and synchronize launches with the rest of your computation.
#
# The second set of arguments are the Jax arrays for inputs and outputs. Inputs must be passed before
# outputs.
#
# Lastly static arguments (i.e. static_argnums or static_argnames) values are passed as keyword only arguments
# by name.
#
# The the kernel does not match this signature a wrapper functions like the one shown below can be written
# or an inline lambda function can be used to rebind the arguments into the appropriate order.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 103-114 / 第 103-114 行

~~~~python
@cute.jit
def launch_jax_wrapper(
    stream: cuda.CUstream,
    A: cute.Tensor,
    B: cute.Tensor,
    C: cute.Tensor,
    D: cute.Tensor,
    *,
    x: cute.Int32,
    y: cute.Int32,
):
    launch(A, B, x, y, C, D, stream)
~~~~

**EN**: Marks `launch_jax_wrapper` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 将 `launch_jax_wrapper` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 117-126 / 第 117-126 行

~~~~python
@cute.jit
def launch_aliased(
    A: cute.Tensor, B: cute.Tensor, x: cute.Int32, y: cute.Int32, stream: cuda.CUstream
):
    # Print layouts
    print("A layout: ", A.layout)
    print("B layout: ", B.layout)
    cute.printf("A layout: {}", A.layout)
    cute.printf("B layout: {}", B.layout)
    cute.printf("")
~~~~

**EN**: Marks `launch_aliased` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 将 `launch_aliased` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 128-128 / 第 128-128 行

~~~~python
    # Print non-tensor values
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 129-133 / 第 129-133 行

~~~~python
    print("X is: ", x)
    print("Y is: ", y)
    cute.printf("X is: {}", x)
    cute.printf("Y is: {}", y)
    print()
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 136-136 / 第 136-136 行

~~~~python
if __name__ == "__main__":
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 138-150 / 第 138-150 行

~~~~python
    @partial(jax.jit, static_argnums=[2, 3])
    def run_cutlass_kernel(a, b, x, y):
        call = cjax.cutlass_call(
            launch_jax_wrapper,
            # Describe the shape and dtype of each output buffer.
            output_shape_dtype=(
                jax.ShapeDtypeStruct(a.shape, a.dtype),
                jax.ShapeDtypeStruct(b.shape, a.dtype),
            ),
            # Static jit arguments are passed via additional keyword arguments.
            x=x,
            y=y,
        )
~~~~

**EN**: Defines `run_cutlass_kernel`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `run_cutlass_kernel`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 152-152 / 第 152-152 行

~~~~python
        # Returned value is a callable to invoke the kernel passing only jax arrays.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 153-153 / 第 153-153 行

~~~~python
        return call(a, b)
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 155-158 / 第 155-158 行

~~~~python
    print("\nExample: example_basic_call_from_jit")
    A = jnp.zeros((512, 32, 64))
    B = jnp.zeros((1, 256, 64, 128))
    C, D = run_cutlass_kernel(A, B, 0, 1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 160-174 / 第 160-174 行

~~~~python
    @partial(jax.jit, static_argnums=[2, 3])
    def run_cutlass_kernel_lambda(a, b, x, y):
        call = cjax.cutlass_call(
            # A lambda function may be used to wrap and bind arguments passed by jax
            # to the kernel. Alternatively you can wrap using another separate cute.jit
            # function.
            lambda stream, a, b, c, d, *, x, y: launch(a, b, x, y, c, d, stream),
            output_shape_dtype=(
                jax.ShapeDtypeStruct(a.shape, a.dtype),
                jax.ShapeDtypeStruct(b.shape, a.dtype),
            ),
            # Static jit arguments are passed via additional keyword arguments.
            x=x,
            y=y,
        )
~~~~

**EN**: Defines `run_cutlass_kernel_lambda`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `run_cutlass_kernel_lambda`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 176-176 / 第 176-176 行

~~~~python
        # Returned value is a callable to invoke the kernel passing only jax arrays.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 177-177 / 第 177-177 行

~~~~python
        return call(a, b)
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 179-182 / 第 179-182 行

~~~~python
    print("\nExample: run_cutlass_kernel_lambda")
    A = jnp.zeros((512, 32, 64))
    B = jnp.zeros((1, 256, 64, 128))
    C, D = run_cutlass_kernel_lambda(A, B, 1, 2)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 184-202 / 第 184-202 行

~~~~python
    @partial(jax.jit, static_argnums=[2, 3])
    def run_cutlass_kernel_static_shapes(a, b, x, y):
        call = cjax.cutlass_call(
            lambda stream, a, b, c, d, *, x, y: launch(a, b, x, y, c, d, stream),
            output_shape_dtype=(
                jax.ShapeDtypeStruct(a.shape, a.dtype),
                jax.ShapeDtypeStruct(b.shape, a.dtype),
            ),
            # By default cutlass_call treats all tensors as dynamic shape.
            # Dynamic shapes are often expected for kernels so this default ensures
            # the broadest support. If you know that a kernel can accept fully static
            # tensors then you can enable this flag to compile all tensor shapes and
            # layouts as constexpr values known at compile time.
            # Individual tensors may opt out via .mark_layout_dynamic().
            use_static_tensors=True,
            x=x,
            y=y,
        )
        return call(a, b)
~~~~

**EN**: Defines `run_cutlass_kernel_static_shapes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `run_cutlass_kernel_static_shapes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 204-207 / 第 204-207 行

~~~~python
    print("\nExample: run_cutlass_kernel_static_shapes")
    A = jnp.zeros((512, 32, 64))
    B = jnp.zeros((1, 256, 64, 128))
    C, D = run_cutlass_kernel_static_shapes(A, B, 3, 4)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 209-228 / 第 209-228 行

~~~~python
    @partial(jax.jit, static_argnums=[2, 3])
    def run_cutlass_kernel_with_modes(a, b, x, y):
        # input_spec and output_spec accept TensorSpec values to attach layout
        # metadata to tensors.  mode remaps the logical dimension order seen by
        # the kernel.  static=True compiles that tensor's layout as constexpr.
        call = cjax.cutlass_call(
            lambda stream, a, b, c, d, *, x, y: launch(a, b, x, y, c, d, stream),
            output_shape_dtype=(
                jax.ShapeDtypeStruct(a.shape, a.dtype),
                jax.ShapeDtypeStruct(b.shape, a.dtype),
            ),
            input_spec=(
                cjax.TensorSpec(mode=(1, 0, 2), static=True),
                cjax.TensorSpec(mode=(3, 1, 2, 0)),
            ),
            output_spec=(None, cjax.TensorSpec(mode=(0, 1, 3, 2))),
            x=x,
            y=y,
        )
        return call(a, b)
~~~~

**EN**: Defines `run_cutlass_kernel_with_modes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `run_cutlass_kernel_with_modes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 230-233 / 第 230-233 行

~~~~python
    print("\nExample: run_cutlass_kernel_with_modes")
    A = jnp.zeros((512, 32, 64))
    B = jnp.zeros((1, 256, 64, 128))
    C, D = run_cutlass_kernel_with_modes(A, B, 5, 6)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 235-249 / 第 235-249 行

~~~~python
    @partial(jax.jit, static_argnums=[2, 3], donate_argnums=[0, 1])
    def run_cutlass_kernel_aliased_outputs(a, b, x, y):
        call = cjax.cutlass_call(
            lambda stream, a, b, *, x, y: launch_aliased(a, b, x, y, stream),
            output_shape_dtype=(
                jax.ShapeDtypeStruct(a.shape, a.dtype),
                jax.ShapeDtypeStruct(b.shape, b.dtype),
            ),
            # Map input indices to output indices so XLA can reuse the input
            # buffers for the outputs, avoiding extra allocations.
            input_output_aliases={0: 0, 1: 1},
            x=x,
            y=y,
        )
        return call(a, b)
~~~~

**EN**: Defines `run_cutlass_kernel_aliased_outputs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `run_cutlass_kernel_aliased_outputs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 251-254 / 第 251-254 行

~~~~python
    print("\nExample: run_cutlass_kernel_aliased_outputs")
    A = jnp.zeros((512, 32, 64))
    B = jnp.zeros((1, 256, 64, 128))
    A, B = run_cutlass_kernel_aliased_outputs(A, B, 7, 8)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `functools.partial` — supplies small Python helpers such as partial application / 提供 partial 等 Python 辅助能力
- `jax` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.numpy` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.jax` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
