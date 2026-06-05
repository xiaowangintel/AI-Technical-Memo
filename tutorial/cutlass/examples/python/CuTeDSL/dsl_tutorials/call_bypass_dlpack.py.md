# call_bypass_dlpack.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/call_bypass_dlpack.py`  
**Purpose / 用途**: Tutorial example showing call bypass dlpack in CuTeDSL. / 这是一个关于 call bypass dlpack 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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
import sys
import os
from typing import Tuple

import cutlass
import cutlass.cute as cute
from cutlass.cute.runtime import make_ptr
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 38-74 / 第 38-74 行

~~~~python
"""
An Example demonstrating how to call off-the-shelf kernel by-passing dlpack protocol

The example shows how to directly pass pointers from PyTorch tensors to off-the-shelf kernels
written by CuTe DSL with a thin customized wrapper jit function. The jit function will be
compiled with inline without introducing overhead.

To run this example:

.. code-block:: bash

    python examples/ampere/call_bypass_dlpack.py


It's worth to mention that by-passing dlpack protocol can resolve the issue that dlpack doesn't handle shape-1
mode correctly. For example, the following code will fail, because dlpack will convert the shape-1 mode
with stride-1 which propagate alignment incorrectly.

.. code-block:: python

    @cute.kernel
    def fails_kernel(gX: cute.Tensor):
        bidx, _, _ = cute.arch.block_idx()
        mX = gX[None, bidx, None]  # We wish to retain alignment
        # assert mX.iterator.alignment == 16


    @cute.jit
    def fails(gX_: cute.Tensor):
        gX = gX_
        fails_kernel(gX).launch(grid=(1, 1, 1), block=(128, 1, 1))


    gX_torch = torch.rand((128, 1, 128), device="cuda", dtype=torch.bfloat16)
    fails(from_dlpack(gX_torch, assumed_align=16))

"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 76-78 / 第 76-78 行

~~~~python
if __name__ == "__main__":
    current_dir = os.path.dirname(os.path.abspath(__file__))
    sys.path.insert(0, os.path.join(current_dir, ".."))
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 80-80 / 第 80-80 行

~~~~python
from cute.ampere.kernel.dense_gemm.tensorop_gemm import TensorOpGemm
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 83-94 / 第 83-94 行

~~~~python
@cute.jit
def tensor_op_gemm_wrapper(
    a_ptr: cute.Pointer,
    b_ptr: cute.Pointer,
    c_ptr: cute.Pointer,
    m: cutlass.Int32,
    n: cutlass.Int32,
    k: cutlass.Int32,
    l: cutlass.Int32,
):
    print("\n[DSL INFO] Input Parameters:")
    print(f"[DSL INFO]   mnkl: {(m, n, k, l)}")
~~~~

**EN**: Marks `tensor_op_gemm_wrapper` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 将 `tensor_op_gemm_wrapper` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 96-96 / 第 96-96 行

~~~~python
    # Assume alignment of shape to call tensorop_gemm example
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 97-98 / 第 97-98 行

~~~~python
    m = cute.assume(m, divby=8)
    n = cute.assume(n, divby=8)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 100-100 / 第 100-100 行

~~~~python
    # Torch is row major
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 101-106 / 第 101-106 行

~~~~python
    a_layout = cute.make_ordered_layout((m, k, l), order=(0, 1, 2))
    b_layout = cute.make_ordered_layout((n, k, l), order=(0, 1, 2))
    c_layout = cute.make_ordered_layout((m, n, l), order=(1, 0, 2))
    mA = cute.make_tensor(a_ptr, layout=a_layout)
    mB = cute.make_tensor(b_ptr, layout=b_layout)
    mC = cute.make_tensor(c_ptr, layout=c_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 108-110 / 第 108-110 行

~~~~python
    print(f"[DSL INFO]   mA: {mA}")
    print(f"[DSL INFO]   mB: {mB}")
    print(f"[DSL INFO]   mC: {mC}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 112-119 / 第 112-119 行

~~~~python
    tensor_op_gemm = TensorOpGemm(
        a_ptr.value_type, c_ptr.value_type, cutlass.Float32, (2, 2, 1)
    )
    print("\n[DSL INFO] Created TensorOpGemm instance")
    print(f"[DSL INFO]   Input dtype: {a_ptr.value_type}")
    print(f"[DSL INFO]   Output dtype: {c_ptr.value_type}")
    print(f"[DSL INFO]   Accumulation dtype: {cutlass.Float32}")
    print(f"[DSL INFO]   Atom layout: {(2, 2, 1)}")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 121-121 / 第 121-121 行

~~~~python
    # No need to compile inside jit function
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 122-123 / 第 122-123 行

~~~~python
    tensor_op_gemm(mA, mB, mC)
    print("\n[DSL INFO] Executed TensorOpGemm")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 126-127 / 第 126-127 行

~~~~python
def run_tensor_op_gemm_wrapper(mnkl: Tuple[int, int, int, int]):
    import torch
~~~~

**EN**: Defines `run_tensor_op_gemm_wrapper`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `run_tensor_op_gemm_wrapper`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 129-130 / 第 129-130 行

~~~~python
    print("\nRunning TensorOpGemm test with:")
    print(f"Tensor dimensions: {mnkl}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 132-132 / 第 132-132 行

~~~~python
    # (M,K,L)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 133-143 / 第 133-143 行

~~~~python
    a = torch.randn(
        mnkl[3], mnkl[2], mnkl[0], dtype=torch.float16, device="cuda"
    ).permute(2, 1, 0)
    # (N,K,L)
    b = torch.randn(
        mnkl[3], mnkl[2], mnkl[1], dtype=torch.float16, device="cuda"
    ).permute(2, 1, 0)
    # (N,M,L)
    c = torch.randn(
        mnkl[3], mnkl[0], mnkl[1], dtype=torch.float16, device="cuda"
    ).permute(1, 2, 0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 145-148 / 第 145-148 行

~~~~python
    print("Input tensor shapes:")
    print(f"a: {a.shape}, dtype: {a.dtype}")
    print(f"b: {b.shape}, dtype: {b.dtype}")
    print(f"c: {c.shape}, dtype: {c.dtype}\n")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 150-160 / 第 150-160 行

~~~~python
    a_ptr = make_ptr(
        cutlass.Float16, a.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
    )
    b_ptr = make_ptr(
        cutlass.Float16, b.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
    )
    c_ptr = make_ptr(
        cutlass.Float16, c.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
    )
    tensor_op_gemm_wrapper(a_ptr, b_ptr, c_ptr, *mnkl)
    torch.cuda.synchronize()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 162-165 / 第 162-165 行

~~~~python
    ref = torch.einsum("mkl,nkl->mnl", a, b)
    torch.testing.assert_close(c, ref, atol=1e-05, rtol=1e-05)
    print("\n[DSL INFO] Results verified successfully!")
    print(f"First few elements of result: \n{c[:3, :3, :3]}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 168-169 / 第 168-169 行

~~~~python
if __name__ == "__main__":
    run_tensor_op_gemm_wrapper((512, 256, 128, 16))
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Ampere GPU execution model / Ampere GPU 执行模型
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `sys` — used by this example / 供该示例使用
- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.runtime.make_ptr` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cute.ampere.kernel.dense_gemm.tensorop_gemm.TensorOpGemm` — used by this example / 供该示例使用
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
