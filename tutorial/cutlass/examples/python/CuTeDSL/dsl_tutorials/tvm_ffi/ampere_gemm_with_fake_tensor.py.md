# ampere_gemm_with_fake_tensor.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/tvm_ffi/ampere_gemm_with_fake_tensor.py`  
**Purpose / 用途**: Tutorial example showing ampere gemm with fake tensor in CuTeDSL. / 这是一个关于 ampere gemm with fake tensor 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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
import torch

import cutlass
import cutlass.cute as cute
from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 37-40 / 第 37-40 行

~~~~python
"""Demonstrates calling off-the-shelf kernels with TVM FFI without DLPack.

This example shows how to compile CuTe JIT function with fake tensors then run it with TVM FFI.
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 42-46 / 第 42-46 行

~~~~python
if __name__ == "__main__":
    # Add the current directory to sys.path
    current_dir = os.path.dirname(os.path.abspath(__file__))
    sys.path.insert(0, os.path.join(current_dir, "..", ".."))
from cute.ampere.kernel.dense_gemm.tensorop_gemm import TensorOpGemm
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 49-55 / 第 49-55 行

~~~~python
@cute.jit
def bmm(
    a: cute.Tensor,  # (l, m, k)
    b: cute.Tensor,  # (l, k, n)
    c: cute.Tensor,  # (l, m, n)
):
    gemm_op = TensorOpGemm(cutlass.Float16, cutlass.Float16, cutlass.Float32, (2, 2, 1))
~~~~

**EN**: Marks `bmm` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 将 `bmm` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 57-59 / 第 57-59 行

~~~~python
    # Permute to follow convention of CuTe

    # (l, m, k) -> (m, k, l)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 60-64 / 第 60-64 行

~~~~python
    a = cute.make_tensor(a.iterator, cute.select(a.layout, mode=[1, 2, 0]))
    # (l, k, n) -> (n, k, l)
    b = cute.make_tensor(b.iterator, cute.select(b.layout, mode=[2, 1, 0]))
    # (l, m, n) -> (m, n, l)
    c = cute.make_tensor(c.iterator, cute.select(c.layout, mode=[1, 2, 0]))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 66-66 / 第 66-66 行

~~~~python
    gemm_op(a, b, c)
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 69-70 / 第 69-70 行

~~~~python
def compile_bmm_dynamic_layout():
    from cutlass.cute.runtime import make_fake_compact_tensor
~~~~

**EN**: Defines `compile_bmm_dynamic_layout`, grouping related logic behind a named Python callable so the example stays modular and reusable. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 定义 `compile_bmm_dynamic_layout`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 72-75 / 第 72-75 行

~~~~python
    m = cute.sym_int()
    n = cute.sym_int(divisibility=16)
    k = cute.sym_int(divisibility=16)
    l = cute.sym_int()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 77-77 / 第 77-77 行

~~~~python
    # Contiguous on K
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 78-88 / 第 78-88 行

~~~~python
    fake_a = make_fake_compact_tensor(
        cutlass.Float16, (l, m, k), stride_order=(2, 1, 0), assumed_align=16
    )
    # Contiguous on N
    fake_b = make_fake_compact_tensor(
        cutlass.Float16, (l, k, n), stride_order=(2, 1, 0), assumed_align=16
    )
    # Contiguous on N
    fake_c = make_fake_compact_tensor(
        cutlass.Float16, (l, m, n), stride_order=(2, 1, 0), assumed_align=16
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 90-91 / 第 90-91 行

~~~~python
    compiled_fn = cute.compile(bmm, fake_a, fake_b, fake_c, options="--enable-tvm-ffi")
    return compiled_fn
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 94-95 / 第 94-95 行

~~~~python
def compile_bmm_static_layout(m, n, k, l):
    from cutlass.cute.runtime import make_fake_compact_tensor
~~~~

**EN**: Defines `compile_bmm_static_layout`, grouping related logic behind a named Python callable so the example stays modular and reusable. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 定义 `compile_bmm_static_layout`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 97-105 / 第 97-105 行

~~~~python
    fake_a = make_fake_compact_tensor(
        cutlass.Float16, (l, m, k), stride_order=(2, 1, 0), assumed_align=16
    )
    fake_b = make_fake_compact_tensor(
        cutlass.Float16, (l, k, n), stride_order=(2, 1, 0), assumed_align=16
    )
    fake_c = make_fake_compact_tensor(
        cutlass.Float16, (l, m, n), stride_order=(2, 1, 0), assumed_align=16
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 107-108 / 第 107-108 行

~~~~python
    compiled_fn = cute.compile(bmm, fake_a, fake_b, fake_c, options="--enable-tvm-ffi")
    return compiled_fn
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 111-112 / 第 111-112 行

~~~~python
def run_bmm_and_verify(compiled_fn, m, n, k, l):
    torch.manual_seed(1112)
~~~~

**EN**: Defines `run_bmm_and_verify`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `run_bmm_and_verify`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 114-116 / 第 114-116 行

~~~~python
    a = torch.randn(l, m, k, dtype=torch.float16, device="cuda")
    b = torch.randn(l, k, n, dtype=torch.float16, device="cuda")
    c = torch.randn(l, m, n, dtype=torch.float16, device="cuda")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 118-121 / 第 118-121 行

~~~~python
    print("[Runtime INFO] Input tensor shapes:")
    print(f"a: {a.shape=}, {a.stride()=}, {a.dtype=}")
    print(f"b: {b.shape=}, {b.stride()=}, {b.dtype=}")
    print(f"c: {c.shape=}, {c.stride()=}, {c.dtype=}\n")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 123-123 / 第 123-123 行

~~~~python
    # pass in torch tensor as input
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 124-125 / 第 124-125 行

~~~~python
    compiled_fn(a, b, c)
    torch.cuda.synchronize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 127-130 / 第 127-130 行

~~~~python
    ref = torch.bmm(a, b)
    torch.testing.assert_close(c, ref, atol=1e-05, rtol=1e-05)
    print("[Runtime INFO] Verification successful!")
    print(f"  First few elements of result: \n{c[:3, :3, :3]}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 133-134 / 第 133-134 行

~~~~python
if __name__ == "__main__":
    m, n, k, l = (512, 512, 256, 2)
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 136-137 / 第 136-137 行

~~~~python
    compiled_fn_dynamic = compile_bmm_dynamic_layout()
    run_bmm_and_verify(compiled_fn_dynamic, m, n, k, l)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 139-140 / 第 139-140 行

~~~~python
    compiled_fn_static = compile_bmm_static_layout(m, n, k, l)
    run_bmm_and_verify(compiled_fn_static, m, n, k, l)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 142-143 / 第 142-143 行

~~~~python
    # Error Check:
    #   1. mis-matched tensor dim raise error
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 144-150 / 第 144-150 行

~~~~python
    a = torch.randn(l, m, k, dtype=torch.float16, device="cuda")
    b = torch.randn(l, 2 * k, n, dtype=torch.float16, device="cuda")
    c = torch.randn(l, m, n, dtype=torch.float16, device="cuda")
    try:
        compiled_fn_dynamic(a, b, c)
    except Exception as e:
        print(f"\n[Runtime Error]: {e}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 152-152 / 第 152-152 行

~~~~python
    #   2. mis-matched divisibility
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 153-155 / 第 153-155 行

~~~~python
    a = torch.randn(l, m, k + 1, dtype=torch.float16, device="cuda")
    b = torch.randn(l, k + 1, n, dtype=torch.float16, device="cuda")
    c = torch.randn(l, m, n, dtype=torch.float16, device="cuda")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 157-160 / 第 157-160 行

~~~~python
    try:
        compiled_fn_dynamic(a, b, c)
    except Exception as e:
        print(f"\n[Runtime Error]: {e}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 162-162 / 第 162-162 行

~~~~python
    # 3. mis-matched static shape constraint
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 163-165 / 第 163-165 行

~~~~python
    a = torch.randn(l * 2, m, k, dtype=torch.float16, device="cuda")
    b = torch.randn(l * 2, k, n, dtype=torch.float16, device="cuda")
    c = torch.randn(l * 2, m, n, dtype=torch.float16, device="cuda")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 167-170 / 第 167-170 行

~~~~python
    try:
        compiled_fn_static(a, b, c)
    except Exception as e:
        print(f"\n[Runtime Error]: {e}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

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
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `cute.ampere.kernel.dense_gemm.tensorop_gemm.TensorOpGemm` — used by this example / 供该示例使用
- `cutlass.cute.runtime.make_fake_compact_tensor` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
