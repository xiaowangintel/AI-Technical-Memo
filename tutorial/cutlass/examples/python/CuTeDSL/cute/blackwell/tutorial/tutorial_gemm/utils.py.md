# utils.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/tutorial/tutorial_gemm/utils.py`  
**Purpose / 用途**: Tutorial example showing utils in CuTeDSL. / 这是一个关于 utils 的 CuTeDSL 教程示例，用来逐步讲解相关概念与实现方法。

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

### Lines 29-37 / 第 29-37 行

~~~~python
import argparse
from typing import Tuple

import torch

import cutlass
import cutlass.cute as cute
import cutlass.torch as cutlass_torch
from cutlass.cute.runtime import make_ptr
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 40-46 / 第 40-46 行

~~~~python
def parse_comma_separated_ints(s: str) -> Tuple[int, ...]:
    try:
        return tuple(int(x.strip()) for x in s.split(","))
    except ValueError:
        raise argparse.ArgumentTypeError(
            "Invalid format. Expected comma-separated integers."
        )
~~~~

**EN**: Defines `parse_comma_separated_ints`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 定义 `parse_comma_separated_ints`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 49-65 / 第 49-65 行

~~~~python
def create_parser():
    parser = argparse.ArgumentParser(
        description="Example of Sm100 Dense BlockScaled GEMM."
    )
    parser.add_argument(
        "--mnkl",
        type=parse_comma_separated_ints,
        default=(8192, 8192, 8192, 8),
        help="mnkl dimensions (comma-separated)",
    )
    parser.add_argument(
        "--tolerance", type=float, default=1e-01, help="Tolerance for validation"
    )
    parser.add_argument(
        "--do_benchmark", action="store_true", default=False, help="Do benchmark test"
    )
    return parser
~~~~

**EN**: Defines `create_parser`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 定义 `create_parser`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 68-69 / 第 68-69 行

~~~~python
def ceil_div(a, b):
    return (a + b - 1) // b
~~~~

**EN**: Defines `ceil_div`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `ceil_div`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 72-74 / 第 72-74 行

~~~~python
# Helper function to create scale factor tensor SFA/SFB
# for 1x16 block scaled wise use case and follow the layout requirement
# defined in https://docs.nvidia.com/cuda/cublas/index.html?highlight=fp4#d-block-scaling-factors-layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 75-86 / 第 75-86 行

~~~~python
@cute.jit
def cvt_sf_MKL_to_M32x4xrm_K4xrk_L(
    sf_ref_ptr: cute.Pointer,
    sf_mma_ptr: cute.Pointer,
    mn: int,
    sf_k: int,
    l: int,
    mma_shape: tuple,
):
    mma_permute_order = (3, 4, 1, 5, 2, 0)
    permuted_shape = tuple(mma_shape[i] for i in mma_permute_order)
    cute_layout = cute.make_ordered_layout(permuted_shape, order=(2, 1, 4, 0, 3, 5))
~~~~

**EN**: Marks `cvt_sf_MKL_to_M32x4xrm_K4xrk_L` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `cvt_sf_MKL_to_M32x4xrm_K4xrk_L` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 88-91 / 第 88-91 行

~~~~python
    sf_ref_tensor = cute.make_tensor(
        sf_ref_ptr, cute.make_layout((mn, sf_k, l), stride=(sf_k, 1, mn * sf_k))
    )
    sf_mma_tensor = cute.make_tensor(sf_mma_ptr, cute_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 93-98 / 第 93-98 行

~~~~python
    sf_mma_tensor = cute.group_modes(sf_mma_tensor, 0, 3)
    sf_mma_tensor = cute.group_modes(sf_mma_tensor, 1, 3)
    for i in cutlass.range(cute.size(sf_ref_tensor)):
        mkl_coord = sf_ref_tensor.layout.get_hier_coord(i)
        sf_mma_tensor[mkl_coord] = sf_ref_tensor[mkl_coord]
    pass
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 101-102 / 第 101-102 行

~~~~python
def to_blocked(input_matrix):
    rows, cols = input_matrix.shape
~~~~

**EN**: Defines `to_blocked`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `to_blocked`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 104-104 / 第 104-104 行

~~~~python
    # Please ensure rows and cols are multiples of 128 and 4 respectively
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 105-106 / 第 105-106 行

~~~~python
    n_row_blocks = ceil_div(rows, 128)
    n_col_blocks = ceil_div(cols, 4)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 108-110 / 第 108-110 行

~~~~python
    padded = input_matrix
    blocks = padded.view(n_row_blocks, 128, n_col_blocks, 4).permute(0, 2, 1, 3)
    rearranged = blocks.reshape(-1, 4, 32, 4).transpose(1, 2).reshape(-1, 32, 16)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 112-112 / 第 112-112 行

~~~~python
    return rearranged.flatten()
~~~~

**EN**: Returns the constructed object or computed result to the caller. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把构造好的对象或计算结果返回给调用方。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 115-142 / 第 115-142 行

~~~~python
def run(
    gemm_class,
    ab_dtype,
    sf_dtype,
    c_dtype,
    sf_vec_size,
    mma_tiler_mn,
    cluster_shape_mnk,
    mnkl: Tuple[int, int, int, int],
    tolerance: float,
    warmup_iterations: int = 10,
    iterations: int = 100,
    use_cold_l2: bool = True,
    do_benchmark: bool = False,
):
    """
    Prepare A/B/SFA/SFB/C tensors, launch GPU kernel, and reference checking.
    """
    print("=" * 60)
    print("Launching Blackwell Dense BlockScaled GEMM Test")
    print("-" * 60)
    print(f"Input dimensions (m, n, k, l): {mnkl}")
    print(f"    m (rows): {mnkl[0]}")
    print(f"    n (cols): {mnkl[1]}")
    print(f"    k (inner): {mnkl[2]}")
    print(f"    l (batch): {mnkl[3]}")
    print(f"Data Types & Precision:")
    print(f"    Input matrices (A, B): {ab_dtype}")
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 143-151 / 第 143-151 行

~~~~python
    print(f"    Scale factors (SFA, SFB): {sf_dtype}")
    print(f"    Output matrix (C): {c_dtype}")
    print(f"    Scale factor vector size: {sf_vec_size}")
    print("Tile and cluster configuration:")
    print(f"    MMA tiler (M, N, K): {mma_tiler_mn}")
    print(f"     Cluster shape (M, N, K): {cluster_shape_mnk}")
    print(f"Validation tolerance: {tolerance}")
    print(f"Do benchmark: {do_benchmark}")
    print("=" * 60)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 153-153 / 第 153-153 行

~~~~python
    # Unpack parameters
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 154-154 / 第 154-154 行

~~~~python
    m, n, k, l = mnkl
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 156-157 / 第 156-157 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 159-159 / 第 159-159 行

~~~~python
    torch.manual_seed(1111)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 161-161 / 第 161-161 行

~~~~python
    # Create tensor A/B/C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 162-171 / 第 162-171 行

~~~~python
    a_ref = torch.randint(
        0, 2, (l, m, k // 2), dtype=torch.uint8, device="cuda"
    ).permute(1, 2, 0)
    b_ref = torch.randint(
        0, 2, (l, n, k // 2), dtype=torch.uint8, device="cuda"
    ).permute(1, 2, 0)
    # a_ref = torch.ones((l, m, k // 2), dtype=torch.uint8, device="cuda").permute(1, 2, 0)
    # b_ref = torch.ones((l, n, k // 2), dtype=torch.uint8, device="cuda").permute(1, 2, 0)
    a_ref_f4 = a_ref.view(torch.float4_e2m1fn_x2)
    b_ref_f4 = b_ref.view(torch.float4_e2m1fn_x2)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 173-175 / 第 173-175 行

~~~~python
    c_tensor = torch.randn((l, m, n), dtype=torch.float16, device="cuda").permute(
        1, 2, 0
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 177-177 / 第 177-177 行

~~~~python
    # Create a torch tensor for scale factor tensor of A and B
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 178-191 / 第 178-191 行

~~~~python
    def create_ref_scale_factor_tensor(l, mn, sf_k):
        """
        Create the reference scale factor tensor on CPU.
        Returns the reshaped/pruned tensor ready for ref computation and its original permuted form.
        """
        ref_shape = (l, mn, sf_k)
        ref_permute_order = (1, 2, 0)
        ref_f8_random_int = torch.randint(1, 3, ref_shape, dtype=torch.int8)
        ref_f8_torch_tensor_cpu = ref_f8_random_int.to(dtype=torch.float8_e4m3fn)
        # permute to match ref_permute_order
        ref_f8_torch_tensor_cpu_permuted = ref_f8_torch_tensor_cpu.permute(
            *ref_permute_order
        )
        return ref_f8_torch_tensor_cpu_permuted
~~~~

**EN**: Defines `create_ref_scale_factor_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 定义 `create_ref_scale_factor_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 193-193 / 第 193-193 行

~~~~python
    # Copy the reference scale factor tensor to the CUTE-format scale factor tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 194-208 / 第 194-208 行

~~~~python
    def create_cute_scale_factor_tensor(l, mn, sf_k, ref_f8_torch_tensor_cpu_permuted):
        """
        Create the CUTE-format scale factor tensor on CUDA based on the reference tensor.
        """
        atom_m = (32, 4)
        atom_k = 4
        mma_shape = (
            l,  # batch size
            ceil_div(mn, atom_m[0] * atom_m[1]),
            ceil_div(sf_k, atom_k),
            atom_m[0],
            atom_m[1],
            atom_k,
        )
        mma_permute_order = (3, 4, 1, 5, 2, 0)
~~~~

**EN**: Defines `create_cute_scale_factor_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_cute_scale_factor_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 210-210 / 第 210-210 行

~~~~python
        # Generate a random int8 tensor, then convert to float8_e4m3fn
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 211-214 / 第 211-214 行

~~~~python
        rand_int_tensor = torch.randint(0, 2, mma_shape, dtype=torch.int8)
        cute_f8_torch_tensor_cpu = rand_int_tensor.to(dtype=torch.float8_e4m3fn)
        # Permute according to mma_permute_order
        cute_f8_torch_tensor_cpu = cute_f8_torch_tensor_cpu.permute(*mma_permute_order)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 216-216 / 第 216-216 行

~~~~python
        # Call the helper function to do layout conversion
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 217-235 / 第 217-235 行

~~~~python
        cvt_sf_MKL_to_M32x4xrm_K4xrk_L(
            make_ptr(
                cutlass.Float8E4M3FN,
                ref_f8_torch_tensor_cpu_permuted.data_ptr(),
                cute.AddressSpace.gmem,
                assumed_align=32,
            ),
            make_ptr(
                cutlass.Float8E4M3FN,
                cute_f8_torch_tensor_cpu.data_ptr(),
                cute.AddressSpace.gmem,
                assumed_align=32,
            ),
            mn,
            sf_k,
            l,
            mma_shape,
        )
        return cute_f8_torch_tensor_cpu.cuda()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 237-243 / 第 237-243 行

~~~~python
    sf_k = ceil_div(k, sf_vec_size)
    sfa_ref = create_ref_scale_factor_tensor(l, m, sf_k)
    sfb_ref = create_ref_scale_factor_tensor(l, n, sf_k)
    # sfa_ref.fill_(1)
    # sfb_ref.fill_(1)
    sfa_tensor = create_cute_scale_factor_tensor(l, m, sf_k, sfa_ref)
    sfb_tensor = create_cute_scale_factor_tensor(l, n, sf_k, sfb_ref)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 245-245 / 第 245-245 行

~~~~python
    # Configure gemm kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 246-263 / 第 246-263 行

~~~~python
    gemm = gemm_class()
    # Initialize Stream
    current_stream = cutlass_torch.default_stream()
    a_ptr = make_ptr(
        ab_dtype, a_ref_f4.data_ptr(), cute.AddressSpace.gmem, assumed_align=16
    )
    b_ptr = make_ptr(
        ab_dtype, b_ref_f4.data_ptr(), cute.AddressSpace.gmem, assumed_align=16
    )
    c_ptr = make_ptr(
        c_dtype, c_tensor.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
    )
    sfa_ptr = make_ptr(
        sf_dtype, sfa_tensor.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
    )
    sfb_ptr = make_ptr(
        sf_dtype, sfb_tensor.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 264-284 / 第 264-284 行

~~~~python
    # Compile gemm kernel
    compiled_gemm = cute.compile(
        gemm,
        a_ptr,
        b_ptr,
        sfa_ptr,
        sfb_ptr,
        c_ptr,
        (m, n, k, l),
        current_stream,
    )
    # Launch GPU kernel
    compiled_gemm(a_ptr, b_ptr, sfa_ptr, sfb_ptr, c_ptr, (m, n, k, l), current_stream)
    # For batch l, do (m, k, l) @ (n, k, l).T along k for each batch.
    # Result: (m, n, l)
    # Allocate ref as (l, m, n) with n-contiguous layout, then permute to (m, n, l)
    ref = torch.empty(
        (l, m, n),
        dtype=torch.float16,
        device="cuda",
    ).permute(1, 2, 0)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 285-299 / 第 285-299 行

~~~~python
    for l_idx in range(l):
        # Convert the scale factor tensor to blocked format
        scale_a = to_blocked(sfa_ref[:, :, l_idx])
        scale_b = to_blocked(sfb_ref[:, :, l_idx])
        # (m, k) @ (n, k).T -> (m, n)
        res = torch._scaled_mm(
            a_ref_f4[:, :, l_idx],
            b_ref_f4[:, :, l_idx].transpose(0, 1),
            scale_a.cuda(),
            scale_b.cuda(),
            bias=None,
            out_dtype=torch.float16,
        )
        ref[:, :, l_idx] = res
    torch.testing.assert_close(c_tensor, ref, atol=tolerance, rtol=1e-02)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 301-301 / 第 301-301 行

~~~~python
    if do_benchmark:
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 303-329 / 第 303-329 行

~~~~python
        def generate_tensors():
            a_ptr = make_ptr(
                ab_dtype, a_ref_f4.data_ptr(), cute.AddressSpace.gmem, assumed_align=16
            )
            b_ptr = make_ptr(
                ab_dtype, b_ref_f4.data_ptr(), cute.AddressSpace.gmem, assumed_align=16
            )
            c_ptr = make_ptr(
                c_dtype, c_tensor.data_ptr(), cute.AddressSpace.gmem, assumed_align=32
            )
            sfa_ptr = make_ptr(
                sf_dtype,
                sfa_tensor.data_ptr(),
                cute.AddressSpace.gmem,
                assumed_align=32,
            )
            sfb_ptr = make_ptr(
                sf_dtype,
                sfb_tensor.data_ptr(),
                cute.AddressSpace.gmem,
                assumed_align=32,
            )
            args = cute.testing.JitArguments(
                a_ptr, b_ptr, sfa_ptr, sfb_ptr, c_ptr, (m, n, k, l), current_stream
            )
            args.add_to_scope([a_ref_f4, b_ref_f4, sfa_tensor, sfb_tensor, c_tensor])
            return args
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 331-342 / 第 331-342 行

~~~~python
        workspace_count = 1
        if use_cold_l2:
            one_workspace_bytes = (
                a_ref_f4.numel() * a_ref_f4.element_size()
                + b_ref_f4.numel() * b_ref_f4.element_size()
                + sfa_tensor.numel() * sfa_tensor.element_size()
                + sfb_tensor.numel() * sfb_tensor.element_size()
                + c_tensor.numel() * c_tensor.element_size()
            )
            workspace_count = cute.testing.get_workspace_count(
                one_workspace_bytes, warmup_iterations, iterations
            )
~~~~

**EN**: Advances the tutorial by implementing one more self-contained step in the overall CuTeDSL workflow. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 通过实现 CuTeDSL 工作流中的一个自包含步骤，继续推进本教程。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

### Lines 344-344 / 第 344-344 行

~~~~python
        # Return execution time in microseconds
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 345-352 / 第 345-352 行

~~~~python
        time = cute.testing.benchmark(
            compiled_gemm,
            workspace_generator=generate_tensors,
            workspace_count=workspace_count,
            stream=current_stream,
            warmup_iterations=warmup_iterations,
            iterations=iterations,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 354-366 / 第 354-366 行

~~~~python
        print(f"Execution time: {time} us")
        peta_flops = (4 * m * n * k * l) / (time * 1e-6) / 1e9 / 1000000
        print(f"FLOPS: {peta_flops} PFLOPS")
        bytes_transfer = (
            2 * m * k / 2 * l * a_ref_f4.element_size()
            + 2 * n * k / 2 * l * b_ref_f4.element_size()
            + 2 * m * n * l * c_tensor.element_size()
            + 2 * m * sf_k * l * sfa_tensor.element_size()
            + 2 * n * sf_k * l * sfb_tensor.element_size()
        )
        print(f"Bytes: {bytes_transfer} Bytes")
        bandwidth = bytes_transfer / time * 1e-3
        print(f"BW: {bandwidth} GB/s")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. In tutorial context, this block isolates one concept so the reader can connect the Python DSL syntax to the underlying CUDA mechanism step by step.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 在教程语境下，这个代码块把一个概念单独拿出来讲解，便于读者逐步把 Python DSL 语法和底层 CUDA 机制对应起来。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Step-by-step tutorial progression / 循序渐进的教程推进
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.runtime.make_ptr` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
