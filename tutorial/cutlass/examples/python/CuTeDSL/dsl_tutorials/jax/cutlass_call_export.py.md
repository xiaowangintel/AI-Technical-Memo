# cutlass_call_export.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/jax/cutlass_call_export.py`  
**Purpose / 用途**: Examples of using jax.export APIs with functions using cutlass_call. / 这是一个关于 cutlass call export 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-51 / 第 29-51 行

~~~~python
"""
Examples of using jax.export APIs with functions using cutlass_call.

This example demonstrates three export modes:

1. Concrete shapes  -- shapes are fixed constants baked into the export.
2. Unconstrained symbolic shapes ("a, b")
3. Constrained symbolic shapes ("32*M, 16*N")

The JAX function being exported is the same in all three cases; only the
shape specification passed to jax.export differs.

It assumes familiarity with CuTe DSL concepts such as layouts and dynamic shapes
as well as JAX's exporting and serialization features:
https://docs.jax.dev/en/latest/export/index.html#export

To run this example:

.. code-block:: bash

    python examples/jax/cutlass_call_export.py --M 512 --N 256

"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 53-64 / 第 53-64 行

~~~~python
import argparse

import cuda.bindings.driver as cuda

import cutlass.cute as cute

import jax
import jax.numpy as jnp
from jax import export

from cutlass.jax import cutlass_call, get_export_disabled_safety_checks, TensorSpec
from cutlass.jax.testing import create_tensor
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 67-67 / 第 67-67 行

~~~~python
# Simple element-wise addition kernel: gC[i,j] = gA[i,j] + gB[i,j]
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 68-72 / 第 68-72 行

~~~~python
@cute.kernel
def kernel(gA: cute.Tensor, gB: cute.Tensor, gC: cute.Tensor):
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
    bdim, _, _ = cute.arch.block_dim()
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 74-74 / 第 74-74 行

~~~~python
    thread_idx = bidx * bdim + tidx
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 76-78 / 第 76-78 行

~~~~python
    m, n = gA.shape
    ni = thread_idx % n
    mi = thread_idx // n
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 80-82 / 第 80-82 行

~~~~python
    a_val = gA[mi, ni]
    b_val = gB[mi, ni]
    gC[mi, ni] = a_val + b_val
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 85-93 / 第 85-93 行

~~~~python
@cute.jit
def launch(stream: cuda.CUstream, mA: cute.Tensor, mB: cute.Tensor, mC: cute.Tensor):
    num_threads_per_block = 256
    m, n = mA.shape
    kernel(mA, mB, mC).launch(
        grid=((m * n) // num_threads_per_block, 1, 1),
        block=(num_threads_per_block, 1, 1),
        stream=stream,
    )
~~~~

**EN**: Marks `launch` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 将 `launch` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 96-101 / 第 96-101 行

~~~~python
def _export_and_run(f, ref_f, input_shape_dtype, run_shapes):
    """Export f, serialize/deserialize, then run on each shape in run_shapes.

    Both inputs (a, b) are assumed to share the same input_shape_dtype.
    """
    print(f"Exporting with input signature: ({input_shape_dtype}, {input_shape_dtype})")
~~~~

**EN**: Defines `_export_and_run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 定义 `_export_and_run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 103-105 / 第 103-105 行

~~~~python
    # jax.export can be used to export a jit function containing cutlass_call.
    # CUTLASS custom call targets are not on JAX's built-in stable custom-call
    # allowlist, so we pass them via disabled_checks to suppress that safety check.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 106-107 / 第 106-107 行

~~~~python
    exported = jax.export.export(f, disabled_checks=get_export_disabled_safety_checks())
    traced = exported(input_shape_dtype, input_shape_dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 109-110 / 第 109-110 行

~~~~python
    blob = traced.serialize()
    print(f"Serialized computation is {len(blob)} bytes.")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 112-112 / 第 112-112 行

~~~~python
    rehydrated = export.deserialize(blob)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 114-121 / 第 114-121 行

~~~~python
    key = jax.random.key(1123)
    a_key, b_key = jax.random.split(key, 2)
    for shape in run_shapes:
        a = create_tensor(shape, dtype=jnp.float32, key=a_key)
        b = create_tensor(shape, dtype=jnp.float32, key=b_key)
        c = rehydrated.call(a, b)
        assert jnp.allclose(c, ref_f(a, b)), f"Mismatch at shape {shape}"
        print(f"  shape {shape}: OK")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 124-127 / 第 124-127 行

~~~~python
def run_example(M, N):
    @jax.jit
    def ref_f(a, b):
        return jax.nn.sigmoid(a + b)
~~~~

**EN**: Defines `run_example`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `run_example`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 129-130 / 第 129-130 行

~~~~python
    # The same JAX function is used in all three examples below. The export
    # mode is determined entirely by the shape spec passed to jax.export.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 131-134 / 第 131-134 行

~~~~python
    @jax.jit
    def f(a, b):
        call = cutlass_call(launch, output_shape_dtype=a)
        return jax.nn.sigmoid(call(a, b))
~~~~

**EN**: Defines `f`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `f`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 136-138 / 第 136-138 行

~~~~python
    # ── 1. Concrete shapes ────────────────────────────────────────────────────
    # Shapes are fixed constants baked into the export. The deserialized
    # computation only accepts exactly these dimensions at runtime.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 139-139 / 第 139-139 行

~~~~python
    print("\nConcrete shapes:")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 141-147 / 第 141-147 行

~~~~python
    input_shape_dtype = jax.ShapeDtypeStruct((M, N), jnp.float32)
    _export_and_run(
        f,
        ref_f,
        input_shape_dtype,
        run_shapes=[(M, N)],  # concrete exports reject any other shape
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 149-151 / 第 149-151 行

~~~~python
    # ── 2. Unconstrained symbolic shapes ─────────────────────────────────────
    # Both dimensions are fully dynamic. The exported computation accepts any
    # (M, N) at runtime without recompilation.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 152-152 / 第 152-152 行

~~~~python
    print("\nUnconstrained symbolic shapes:")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 154-161 / 第 154-161 行

~~~~python
    a_sym, b_sym = export.symbolic_shape("a, b")
    input_shape_dtype = jax.ShapeDtypeStruct((a_sym, b_sym), jnp.float32)
    _export_and_run(
        f,
        ref_f,
        input_shape_dtype,
        run_shapes=[(M, N), (M * 2, N * 4), (M * 4, N * 4)],
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 163-168 / 第 163-168 行

~~~~python
    # ── 3. Constrained symbolic shapes (divisibility) ─────────────────────────
    # Shapes are declared as multiples of a tile size via TensorSpec.divisibility.
    # The symbolic expression "32*M, 16*N" tells jax.export that dim 0 is always
    # a multiple of 32 and dim 1 is always a multiple of 16. This lets the
    # compiler generate more efficient code (e.g. no remainder handling).
    # Runtime shapes must satisfy these divisibility constraints.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 169-169 / 第 169-169 行

~~~~python
    print("\nConstrained symbolic shapes:")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 171-180 / 第 171-180 行

~~~~python
    @jax.jit
    def f_divisible(a, b):
        spec = TensorSpec(divisibility=(32, 16))
        call = cutlass_call(
            launch,
            output_shape_dtype=a,
            input_spec=(spec, spec),
            output_spec=spec,
        )
        return jax.nn.sigmoid(call(a, b))
~~~~

**EN**: Defines `f_divisible`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `f_divisible`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 182-189 / 第 182-189 行

~~~~python
    m_sym, n_sym = export.symbolic_shape("32*M, 16*N")
    input_shape_dtype = jax.ShapeDtypeStruct((m_sym, n_sym), jnp.float32)
    _export_and_run(
        f_divisible,
        ref_f,
        input_shape_dtype,
        run_shapes=[(M, N), (M * 2, N * 2), (M * 4, N * 4)],
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 192-197 / 第 192-197 行

~~~~python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Demonstration of using jax.export with functions with cutlass_call"
    )
    parser.add_argument("--M", default=512, type=int)
    parser.add_argument("--N", default=256, type=int)
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 199-201 / 第 199-201 行

~~~~python
    args = parser.parse_args()
    run_example(args.M, args.N)
    print("PASS")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `jax` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.numpy` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.export` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `cutlass.jax.cutlass_call` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.jax.get_export_disabled_safety_checks` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.jax.TensorSpec` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.jax.testing.create_tensor` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
