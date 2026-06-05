# export_to_c.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/export/export_to_c.py`  
**Purpose / 用途**: Tutorial example showing export to c in CuTeDSL. / 这是一个关于 export to c 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-34 / 第 29-34 行

~~~~python
import cutlass
import cutlass.cute as cute

import os
import subprocess
import cuda.bindings.driver as cuda
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 36-49 / 第 36-49 行

~~~~python
"""Example demonstrating how to export a CuTe function.

This example shows how to:
1. Compile a CuTe function
2. Export the compiled function to a object file and a C header file
3. Compile the object file to a shared library

To run this example:

.. code-block:: bash

    # export the compiled function to a object file and a C header file
    python cutlass_ir/compiler/python/examples/cute/export/export_to_c.py
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 52-54 / 第 52-54 行

~~~~python
@cute.kernel
def print_tensor_kernel(a: cute.Tensor):
    cute.printf("a: {}", a)
~~~~

**EN**: Declares `print_tensor_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 `print_tensor_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 57-59 / 第 57-59 行

~~~~python
@cute.jit
def print_tensor(a: cute.Tensor, stream: cuda.CUstream):
    print_tensor_kernel(a).launch(grid=(1, 1, 1), block=(1, 1, 1), stream=stream)
~~~~

**EN**: Marks `print_tensor` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 将 `print_tensor` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 62-64 / 第 62-64 行

~~~~python
@cute.kernel
def add_one_kernel(a: cute.Tensor, b: cute.Tensor):
    a[0] = b[0] + 1
~~~~

**EN**: Declares `add_one_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 把 `add_one_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 67-69 / 第 67-69 行

~~~~python
@cute.jit
def add_one(a: cute.Tensor, b: cute.Tensor, stream: cuda.CUstream):
    add_one_kernel(a, b).launch(grid=(1, 1, 1), block=(1, 1, 1), stream=stream)
~~~~

**EN**: Marks `add_one` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 将 `add_one` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 72-73 / 第 72-73 行

~~~~python
def run():
    from cutlass.cute.runtime import make_fake_compact_tensor
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 75-80 / 第 75-80 行

~~~~python
    shape = (cute.SymInt(divisibility=16), cute.SymInt())
    a = make_fake_compact_tensor(cutlass.Float32, shape, stride_order=(1, 0))
    b = make_fake_compact_tensor(cutlass.Float32, shape, stride_order=(1, 0))
    stream = cuda.CUstream(0)
    compiled_print_tensor = cute.compile(print_tensor, a, stream=stream)
    compiled_add_one = cute.compile(add_one, a, b, stream=stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 82-90 / 第 82-90 行

~~~~python
    os.makedirs("./build", exist_ok=True)
    compiled_print_tensor.export_to_c(
        file_path="./build",
        file_name="print_tensor_example",
        function_prefix="print_tensor",
    )
    compiled_add_one.export_to_c(
        file_path="./build", file_name="add_one_example", function_prefix="add_one"
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 92-92 / 第 92-92 行

~~~~python
    cc = os.environ.get("CC", "gcc")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 94-94 / 第 94-94 行

~~~~python
    # compile the object file to a shared library
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 95-103 / 第 95-103 行

~~~~python
    cmd = [
        cc,
        "-shared",
        "-o",
        "./build/libexport_example.so",
        "./build/print_tensor_example.o",
        "./build/add_one_example.o",
    ]
    subprocess.run(cmd, check=True)
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 106-107 / 第 106-107 行

~~~~python
if __name__ == "__main__":
    run()
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `subprocess` — used by this example / 供该示例使用
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass.cute.runtime.make_fake_compact_tensor` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
