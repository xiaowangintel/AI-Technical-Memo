# dataclass_immutable.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/dataclass_immutable.py`  
**Purpose / 用途**: Dataclass Example: Passing Tensors via Frozen Dataclass. / 这是一个关于 dataclass immutable 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-2 / 第 1-2 行

~~~~python
# Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause
~~~~

**EN**: Records the copyright, SPDX tags, and license conditions that govern how this example may be used and redistributed.
**CN**: 记录版权、SPDX 标记和许可证条件，说明该示例如何被使用和再分发。

### Lines 4-19 / 第 4-19 行

~~~~python
"""
Dataclass Example: Passing Tensors via Frozen Dataclass
=======================================================

Demonstrates passing tensors from `@cute.jit` to `@cute.kernel`
using `@dataclass(frozen=True)`.

Uses fake tensors for compilation and TVM-FFI for efficient runtime dispatch.

To run:

.. code-block:: bash

    python examples/python/CuTeDSL/cute/dataclass_immutable.py

"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 21-25 / 第 21-25 行

~~~~python
from dataclasses import dataclass

import cutlass
import cutlass.cute as cute
from cutlass import Int32
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 28-37 / 第 28-37 行

~~~~python
@dataclass(frozen=True)
class CopyParams:
    """
    Dataclass with one static attribute and two tensors.

    Attributes:
        num_threads: Threads per block (static - used for loop unrolling)
        src: Source tensor (dynamic)
        dst: Destination tensor (dynamic)
    """
~~~~

**EN**: Defines `CopyParams`, a reusable Python class that packages configuration and behavior for this example. In tutorial files, this structure keeps each learning step self-contained.
**CN**: 定义 `CopyParams`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 在教程文件中，这种结构能让每个学习步骤保持自包含。

### Lines 39-41 / 第 39-41 行

~~~~python
    num_threads: int  # static
    src: cute.Tensor  # dynamic
    dst: cute.Tensor  # dynamic
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 44-51 / 第 44-51 行

~~~~python
@cute.jit
def memcpy_jit(mSrc: cute.Tensor, mDst: cute.Tensor, num_threads: cutlass.Constexpr[int]):
    """Host function - passes tensors via dataclass."""
    params = CopyParams(
        num_threads=num_threads,
        src=mSrc,
        dst=mDst,
    )
~~~~

**EN**: Marks `memcpy_jit` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 将 `memcpy_jit` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 53-54 / 第 53-54 行

~~~~python
    N = mSrc.shape[0]
    num_blocks = cute.ceil_div(N, num_threads)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 56-59 / 第 56-59 行

~~~~python
    memcpy_kernel(params, N).launch(
        grid=[num_blocks, 1, 1],
        block=[num_threads, 1, 1],
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 62-66 / 第 62-66 行

~~~~python
@cute.kernel
def memcpy_kernel(params: CopyParams, N: Int32):
    """Device kernel - receives tensors via CopyParams dataclass."""
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `memcpy_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `memcpy_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 68-68 / 第 68-68 行

~~~~python
    idx = bidx * params.num_threads + tidx
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 70-71 / 第 70-71 行

~~~~python
    if idx < N:
        params.dst[idx] = params.src[idx]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 74-76 / 第 74-76 行

~~~~python
if __name__ == "__main__":
    import torch
    from cutlass.cute.runtime import make_fake_compact_tensor
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 78-81 / 第 78-81 行

~~~~python
    n = cute.sym_int64()
    # 1. Create fake tensors for compilation (no GPU memory allocated)
    fake_src = make_fake_compact_tensor(cutlass.Float32, (n,), stride_order=(0,))
    fake_dst = make_fake_compact_tensor(cutlass.Float32, (n,), stride_order=(0,))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 83-83 / 第 83-83 行

~~~~python
    # 2. Compile with --enable-tvm-ffi
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 84-86 / 第 84-86 行

~~~~python
    compiled_fn = cute.compile(
        memcpy_jit, fake_src, fake_dst, num_threads=128, options="--enable-tvm-ffi"
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 88-88 / 第 88-88 行

~~~~python
    # 3. Create real torch tensors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 89-90 / 第 89-90 行

~~~~python
    src = torch.randn(1024, device="cuda", dtype=torch.float32)
    dst = torch.zeros_like(src)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 92-92 / 第 92-92 行

~~~~python
    # 4. Run compiled kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 93-93 / 第 93-93 行

~~~~python
    compiled_fn(src, dst)
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 95-95 / 第 95-95 行

~~~~python
    # 5. Verify dst == src
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 96-98 / 第 96-98 行

~~~~python
    torch.cuda.synchronize()
    torch.testing.assert_close(dst, src)
    print("PASS")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `dataclasses.dataclass` — used by this example / 供该示例使用
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.Int32` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass.cute.runtime.make_fake_compact_tensor` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
