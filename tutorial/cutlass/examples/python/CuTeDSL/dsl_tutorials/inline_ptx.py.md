# inline_ptx.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/dsl_tutorials/inline_ptx.py`  
**Purpose / 用途**: Tutorial example showing inline ptx in CuTeDSL. / 这是一个关于 inline ptx 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-38 / 第 29-38 行

~~~~python
from functools import partial
from typing import Union

import cutlass.cute as cute
from cutlass import Constexpr
from cutlass.cute.runtime import from_dlpack
from cutlass._mlir.dialects import llvm
from cutlass.cute.typing import Boolean, Int32, Int
from cutlass.cutlass_dsl import T, dsl_user_op
from cutlass.cute.arch.nvvm_wrappers import FULL_MASK, WARP_SIZE
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 40-66 / 第 40-66 行

~~~~python
"""
A simple example to show how to wrap PTX instructions by using inline_asm op in llvm dialect.

Situations like:

1. Instructions that are not already exposed by CuTe DSL via `nvvm` module
2. Sequences of instructions that the compiler otherwise does not generate optimally

motivate developers to inline PTX themselves.

In this example, we inline the vote.sync.ballot.b32, vote.sync.any.pred, vote.sync.all.pred,
vote.sync.uni.pred, and use the corresponding ops in nvvm dialect for the test.

You can refer to the documentation of `inline_asm op in llvm dialect <https://mlir.llvm.org/docs/Dialects/LLVM/#llvminline_asm-llvminlineasmop>`_
and `vote.sync <https://docs.nvidia.com/cuda/parallel-thread-execution/#parallel-synchronization-and-communication-instructions-vote-sync>`_
for more details.

To run this example:

.. code-block:: bash

    python examples/dsl/inline_ptx.py

The example will run the vote kernel with inline ptx and nvvm dialect separately.
The results from inline ptx and nvvm dialect will be verified correspondingly.

"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on. As a tutorial block, it also frames the learning progression for later code sections.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。 作为教程内容，它还为后续代码建立学习路径。

### Lines 69-94 / 第 69-94 行

~~~~python
@dsl_user_op
def ptx_vote_sync_op(
    pred: Boolean, kind: str, mask: Int = FULL_MASK, *, loc=None, ip=None
) -> Union[Int32, Boolean]:
    return_type = Boolean
    return_type_str = "pred"
    return return_type(
        llvm.inline_asm(
            T.bool(),
            [
                Boolean(pred).ir_value(loc=loc, ip=ip),
                Int32(mask).ir_value(loc=loc, ip=ip),
            ],
            f"""{{\n\t
            .reg .pred ps;\n\t
            .reg .pred pd;\n\t
            setp.ne.b32 ps, $1, 0;\n\t
            vote.sync.{kind}.{return_type_str} pd, ps, $2;\n\t
            selp.b32 $0, 1, 0, pd;\n\t
            }}""",
            "=r,r,i",
            has_side_effects=True,
            is_align_stack=False,
            asm_dialect=llvm.AsmDialect.AD_ATT,
        )
    )
~~~~

**EN**: Defines `ptx_vote_sync_op` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 将 `ptx_vote_sync_op` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 97-99 / 第 97-99 行

~~~~python
ptx_vote_any_sync = partial(ptx_vote_sync_op, kind="any")
ptx_vote_all_sync = partial(ptx_vote_sync_op, kind="all")
ptx_vote_uni_sync = partial(ptx_vote_sync_op, kind="uni")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 102-125 / 第 102-125 行

~~~~python
@dsl_user_op
def ptx_vote_ballot_sync(
    pred: Boolean, mask: Int = FULL_MASK, *, loc=None, ip=None
) -> Union[Int32, Boolean]:
    return_type = Int32
    return_type_str = "b32"
    return return_type(
        llvm.inline_asm(
            T.i32(),
            [
                Boolean(pred).ir_value(loc=loc, ip=ip),
                Int32(mask).ir_value(loc=loc, ip=ip),
            ],
            f"""{{\n\t
            .reg .pred p;\n\t
            setp.ne.b32 p, $1, 0;\n\t
            vote.sync.ballot.{return_type_str} $0, p, $2;\n\t
            }}""",
            "=r,r,i",
            has_side_effects=True,
            is_align_stack=False,
            asm_dialect=llvm.AsmDialect.AD_ATT,
        )
    )
~~~~

**EN**: Defines `ptx_vote_ballot_sync` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 将 `ptx_vote_ballot_sync` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 128-136 / 第 128-136 行

~~~~python
@cute.kernel
def vote_kernel(
    mBallot: cute.Tensor,
    mAny: cute.Tensor,
    mAll: cute.Tensor,
    mUni: cute.Tensor,
    use_inline_ptx: Constexpr[bool],
):
    tidx, _, _ = cute.arch.thread_idx()
~~~~

**EN**: Declares `vote_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 把 `vote_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 138-157 / 第 138-157 行

~~~~python
    vote_ballot = (
        ptx_vote_ballot_sync(tidx < 10)
        if use_inline_ptx
        else cute.arch.vote_ballot_sync(tidx < 10)
    )
    vote_any = (
        ptx_vote_any_sync(tidx < 10)
        if use_inline_ptx
        else cute.arch.vote_any_sync(tidx < 10)
    )
    vote_all = (
        ptx_vote_all_sync(tidx < 10)
        if use_inline_ptx
        else cute.arch.vote_all_sync(tidx < 10)
    )
    vote_uni = (
        ptx_vote_uni_sync(tidx < 10)
        if use_inline_ptx
        else cute.arch.vote_uni_sync(tidx < 10)
    )
~~~~

**EN**: Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 159-162 / 第 159-162 行

~~~~python
    mBallot[tidx] = vote_ballot
    mAny[tidx] = vote_any
    mAll[tidx] = vote_all
    mUni[tidx] = vote_uni
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 165-182 / 第 165-182 行

~~~~python
@cute.jit
def vote(
    mBallot: cute.Tensor,
    mAny: cute.Tensor,
    mAll: cute.Tensor,
    mUni: cute.Tensor,
    use_inline_ptx: Constexpr[bool],
):
    vote_kernel(
        mBallot,
        mAny,
        mAll,
        mUni,
        use_inline_ptx,
    ).launch(
        grid=[1, 1, 1],
        block=[cute.size(WARP_SIZE, mode=[0]), 1, 1],
    )
~~~~

**EN**: Marks `vote` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 将 `vote` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 185-186 / 第 185-186 行

~~~~python
def run():
    import torch
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 188-199 / 第 188-199 行

~~~~python
    ballot_ptx = torch.randint(
        0, 100, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.int32
    )
    any_ptx = torch.randint(
        0, 2, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.bool
    )
    all_ptx = torch.randint(
        0, 2, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.bool
    )
    uni_ptx = torch.randint(
        0, 2, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.bool
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 201-204 / 第 201-204 行

~~~~python
    mBallotPTX = from_dlpack(ballot_ptx).mark_layout_dynamic()
    mAnyPTX = from_dlpack(any_ptx).mark_layout_dynamic()
    mAllPTX = from_dlpack(all_ptx).mark_layout_dynamic()
    mUniPTX = from_dlpack(uni_ptx).mark_layout_dynamic()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 206-206 / 第 206-206 行

~~~~python
    # get the results from ptx
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 207-207 / 第 207-207 行

~~~~python
    vote(mBallotPTX, mAnyPTX, mAllPTX, mUniPTX, use_inline_ptx=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 209-220 / 第 209-220 行

~~~~python
    ballot_nvvm = torch.randint(
        0, 100, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.int32
    )
    any_nvvm = torch.randint(
        0, 2, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.bool
    )
    all_nvvm = torch.randint(
        0, 2, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.bool
    )
    uni_nvvm = torch.randint(
        0, 2, (WARP_SIZE,), device=torch.device("cuda"), dtype=torch.bool
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 222-225 / 第 222-225 行

~~~~python
    mBallotNVVM = from_dlpack(ballot_nvvm).mark_layout_dynamic()
    mAnyNVVM = from_dlpack(any_nvvm).mark_layout_dynamic()
    mAllNVVM = from_dlpack(all_nvvm).mark_layout_dynamic()
    mUniNVVM = from_dlpack(uni_nvvm).mark_layout_dynamic()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 227-227 / 第 227-227 行

~~~~python
    # get the results from nvvm
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 228-228 / 第 228-228 行

~~~~python
    vote(mBallotNVVM, mAnyNVVM, mAllNVVM, mUniNVVM, use_inline_ptx=False)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 230-237 / 第 230-237 行

~~~~python
    print("Verifying ballot results...")
    torch.testing.assert_close(ballot_ptx, ballot_nvvm)
    print("Verifying any results...")
    torch.testing.assert_close(any_ptx, any_nvvm)
    print("Verifying all results...")
    torch.testing.assert_close(all_ptx, all_nvvm)
    print("Verifying uni results...")
    torch.testing.assert_close(uni_ptx, uni_nvvm)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 239-239 / 第 239-239 行

~~~~python
    print("Results verified successfully!")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 242-243 / 第 242-243 行

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
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出
- Inline PTX / low-level instruction control / 内联 PTX / 底层指令控制

## Dependencies / 依赖项

- `functools.partial` — supplies small Python helpers such as partial application / 提供 partial 等 Python 辅助能力
- `typing.Union` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.Constexpr` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `cutlass._mlir.dialects.llvm` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.typing.Boolean` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.typing.Int32` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.typing.Int` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cutlass_dsl.T` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.dsl_user_op` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cute.arch.nvvm_wrappers.FULL_MASK` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.arch.nvvm_wrappers.WARP_SIZE` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
