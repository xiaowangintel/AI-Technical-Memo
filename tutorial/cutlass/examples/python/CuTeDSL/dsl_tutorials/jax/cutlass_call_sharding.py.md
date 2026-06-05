# cutlass_call_sharding.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/jax/cutlass_call_sharding.py`  
**Purpose / 用途**: Tutorial example showing cutlass call sharding in CuTeDSL. / 这是一个关于 cutlass call sharding 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-39 / 第 29-39 行

~~~~python
from functools import partial

import jax
import jax.numpy as jnp
from jax.sharding import NamedSharding, PartitionSpec as P, AxisType
from jax.experimental.custom_partitioning import custom_partitioning

import cutlass.cute as cute
import cutlass.jax as cjax
from cutlass.jax.testing import create_tensor
import cuda.bindings.driver as cuda
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 42-52 / 第 42-52 行

~~~~python
"""
Examples of combining jax.jit, jax.shard_map and custom_partitioning for sharding
and executing kernels across multiple GPU devices.

To run this example:

.. code-block:: bash

    # Run with addition operation
    python examples/jax/cutlass_call_sharding.py
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 55-58 / 第 55-58 行

~~~~python
@cute.kernel
def kernel(a: cute.Tensor, b: cute.Tensor, c: cute.Tensor):
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 60-62 / 第 60-62 行

~~~~python
    frgA = cute.make_rmem_tensor(cute.size(a, mode=[0]), a.element_type)
    frgB = cute.make_rmem_tensor(cute.size(b, mode=[0]), b.element_type)
    frgC = cute.make_rmem_tensor(cute.size(c, mode=[0]), c.element_type)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 64-67 / 第 64-67 行

~~~~python
    cute.autovec_copy(a[None, tidx, bidx], frgA)
    cute.autovec_copy(b[None, tidx, bidx], frgB)
    frgC.store(frgA.load() + frgB.load())
    cute.autovec_copy(frgC, c[None, tidx, bidx])
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 70-82 / 第 70-82 行

~~~~python
@cute.jit
def launch(
    stream: cuda.CUstream,
    a: cute.Tensor,
    b: cute.Tensor,
    c: cute.Tensor,
):
    cute.printf("a: {}", a.layout)
    cute.printf("b: {}", b.layout)
    cute.printf("c: {}", c.layout)
    kernel(a, b, c).launch(
        grid=[a.shape[-1], 1, 1], block=[a.shape[-2], 1, 1], stream=stream
    )
~~~~

**EN**: Marks `launch` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `launch` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 85-93 / 第 85-93 行

~~~~python
def sharded_cutlass_call_impl(a_block, b_block):
    """The sharded implementation that operates on a single device."""
    call = cjax.cutlass_call(
        launch,
        use_static_tensors=True,
        output_shape_dtype=jax.ShapeDtypeStruct(a_block.shape, a_block.dtype),
    )
    ref_result = a_block + b_block
    return call(a_block, b_block), ref_result
~~~~

**EN**: Defines `sharded_cutlass_call_impl`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `sharded_cutlass_call_impl`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 96-98 / 第 96-98 行

~~~~python
@custom_partitioning
def custom_shared_call(a, b):
    return sharded_cutlass_call_impl(a, b)
~~~~

**EN**: Defines `custom_shared_call`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `custom_shared_call`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 101-103 / 第 101-103 行

~~~~python
def custom_shared_call_partitioner(mesh, arg_shapes, result_shape):
    arg_shardings = jax.tree.map(lambda x: x.sharding, arg_shapes)
    result_shardings = tuple([arg_shardings[0]] * len(result_shape))
~~~~

**EN**: Defines `custom_shared_call_partitioner`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `custom_shared_call_partitioner`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 105-106 / 第 105-106 行

~~~~python
    def lower_fn(*args):
        return sharded_cutlass_call_impl(*args)
~~~~

**EN**: Defines `lower_fn`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `lower_fn`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 108-108 / 第 108-108 行

~~~~python
    return mesh, lower_fn, result_shardings, arg_shardings
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 111-111 / 第 111-111 行

~~~~python
custom_shared_call.def_partition(custom_shared_call_partitioner)
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 114-117 / 第 114-117 行

~~~~python
def run_example():
    # Create a device mesh with one axis b
    ngpu = jax.device_count()
    mesh = jax.make_mesh((ngpu,), "b", axis_types=(AxisType.Explicit,))
~~~~

**EN**: Defines `run_example`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `run_example`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 119-120 / 第 119-120 行

~~~~python
    if ngpu == 1:
        print("Note: only 1 GPU was detected.")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 122-122 / 第 122-122 行

~~~~python
    # We will shard our 3D tensors over b
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 123-124 / 第 123-124 行

~~~~python
    sharding = P("b", None, None)
    named_sharding = NamedSharding(mesh, sharding)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 126-126 / 第 126-126 行

~~~~python
    print("Testing shard_map...")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 128-136 / 第 128-136 行

~~~~python
    @partial(
        jax.jit, static_argnums=[0, 1], out_shardings=(named_sharding, named_sharding)
    )
    def allocate_sharded_tensors(shape, dtype):
        key = jax.random.key(1123)
        a_key, b_key = jax.random.split(key, 2)
        a = create_tensor(shape, dtype, a_key)
        b = create_tensor(shape, dtype, b_key)
        return a, b
~~~~

**EN**: Defines `allocate_sharded_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `allocate_sharded_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 138-148 / 第 138-148 行

~~~~python
    @jax.jit
    def compute(a, b):
        # This jax.shard_map partitions the cutlass_call over the mesh.
        @partial(
            jax.shard_map,
            mesh=mesh,
            in_specs=(sharding, sharding),
            out_specs=(sharding, sharding),
        )
        def sharded_call(a_block, b_block):
            return sharded_cutlass_call_impl(a_block, b_block)
~~~~

**EN**: Defines `compute`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `compute`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 150-150 / 第 150-150 行

~~~~python
        return sharded_call(a, b)
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 152-152 / 第 152-152 行

~~~~python
    # Allocate (32, 16, 64) on each GPU
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 153-154 / 第 153-154 行

~~~~python
    shape = (32 * ngpu, 16, 64)
    dtype = jnp.float32
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 156-157 / 第 156-157 行

~~~~python
    a, b = allocate_sharded_tensors(shape, dtype)
    c, c_ref = compute(a, b)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 159-159 / 第 159-159 行

~~~~python
    assert jnp.allclose(c, c_ref)
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 161-161 / 第 161-161 行

~~~~python
    print("Testing custom_partitioning...")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 163-163 / 第 163-163 行

~~~~python
    # Test custom_partitioning implementation which should produce identical results
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 164-166 / 第 164-166 行

~~~~python
    @jax.jit
    def compute_cp(a, b):
        return custom_shared_call(a, b)
~~~~

**EN**: Defines `compute_cp`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `compute_cp`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 168-168 / 第 168-168 行

~~~~python
    c, c_ref = compute_cp(a, b)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 170-170 / 第 170-170 行

~~~~python
    assert jnp.allclose(c, c_ref)
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 173-175 / 第 173-175 行

~~~~python
if __name__ == "__main__":
    run_example()
    print("PASS")
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

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
- `jax.sharding.NamedSharding` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.sharding.PartitionSpec` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.sharding.AxisType` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `jax.experimental.custom_partitioning.custom_partitioning` — integrates the compiled example with JAX / 将编译后的示例接入 JAX
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.jax` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.jax.testing.create_tensor` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
