# mixed_input_host_utils.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/mixed_input_gemm/mixed_input_host_utils.py`  
**Purpose / 用途**: Kernel example implementing mixed input host utils with CuTeDSL. / 这是一个使用 CuTeDSL 实现 mixed input host utils 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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
from typing import Optional

import torch

import cutlass
import cutlass.cute as cute
import cutlass.torch as cutlass_torch
import cutlass.utils.mixed_input_helpers as mixed_input_utils
from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 39-41 / 第 39-41 行

~~~~python
"""
This file contains common host-side utilities for mixed-input GEMM.
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 44-62 / 第 44-62 行

~~~~python
def create_cumsum_tensor(
    num_groups: int,
    fused_n: int,
    alignment: int,
    uniform_distribution: bool = False,
) -> tuple[cute.Tensor, torch.Tensor]:
    """
    Create a tensor of shape (num_groups + 1) recording the cumulative sum of the elements in each group.
    """
    assert fused_n % alignment == 0, "fused_n must be divisible by alignment"
    if uniform_distribution:
        # keep a uniform distribution for debug and performance collection
        group_counts = torch.tensor([fused_n // num_groups] * num_groups)
    else:
        # sample group sizes with equal probability for each group
        probs = torch.ones(num_groups) / num_groups
        group_sizes = torch.multinomial(probs, fused_n // alignment, replacement=True)
        group_counts = torch.bincount(group_sizes, minlength=num_groups) * alignment
    print(group_counts.tolist())
~~~~

**EN**: Defines `create_cumsum_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 定义 `create_cumsum_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 64-64 / 第 64-64 行

~~~~python
    # Create cumulative sum
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 65-66 / 第 65-66 行

~~~~python
    cumsum_torch = torch.cat([torch.tensor([0]), group_counts.cumsum(0)])
    print(cumsum_torch.tolist())
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 68-70 / 第 68-70 行

~~~~python
    cumsum_tensor, _ = cutlass_torch.cute_tensor_like(
        cumsum_torch, cutlass.Int32, is_dynamic_layout=False
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 72-72 / 第 72-72 行

~~~~python
    return cumsum_tensor, cumsum_torch.to("cpu")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 75-102 / 第 75-102 行

~~~~python
def create_i4_tensor_and_scale(
    l: int,
    m: int,
    k: int,
    is_m_major: bool,
    dtype: type[cutlass.Numeric],
    shuffle_a: bool,
    scale_granularity_m: int,
    scale_granularity_k: int,
    is_dynamic_layout: bool = True,
    init_config: tuple = (
        cutlass_torch.TensorInitType.RANDOM,
        cutlass_torch.RandomInitConfig(min_val=-7, max_val=6),
    ),
    divisibility: int = 16,
    transformed_dtype: Optional[type[cutlass.Numeric]] = None,
) -> tuple[
    cute.Tensor,
    torch.Tensor,
    torch.Tensor,
    cute.Tensor,
    torch.Tensor,
    torch.Tensor,
]:
    """
    Create quantized 4-bit tensor and corresponding scale tensor.
    """
    lb_4b = -8 if dtype == cutlass.Int4 else 0
~~~~

**EN**: Defines `create_i4_tensor_and_scale`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_i4_tensor_and_scale`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 103-110 / 第 103-110 行

~~~~python
    up_4b = 7 if dtype == cutlass.Int4 else 15
    if not (
        init_config[0] == cutlass_torch.TensorInitType.RANDOM
        or init_config[0] == cutlass_torch.TensorInitType.SCALAR
    ):
        raise ValueError(
            "Only random and scalar initialization is supported for 4bit data type"
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 112-112 / 第 112-112 行

~~~~python
    # Construct reference tensor in f32
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 113-133 / 第 113-133 行

~~~~python
    ref_fp32 = cutlass_torch.matrix(l, m, k, is_m_major, cutlass.Float32, *init_config)
    # Generate scale data and perform quantization
    num_scales = k // scale_granularity_k
    ref = ref_fp32.to(dtype=cutlass_torch.dtype(transformed_dtype)).reshape(
        m, num_scales, scale_granularity_k, l
    )
    # Get elements with maximum absolute value to compute scaling factors
    a_max = (
        torch.maximum(ref / up_4b, ref / lb_4b)
        if dtype == cutlass.Int4
        else ref / up_4b
    )
    a_scales, _ = torch.max(a_max, dim=2, keepdim=True)
    a_scale_inv = torch.where(a_scales == 0, 0, 1 / a_scales)
    a_quant = ref * a_scale_inv
    # Convert values to integer to avoid computation errors
    a_quant = a_quant.to(dtype=torch.int32).reshape((m, k, l)).to(dtype=torch.float32)
    # Construct cute scale tensor
    a_scales = a_scales.random_(-3, 3).reshape((m, num_scales, l))
    # Scale tensor is always m-major
    a_scales = a_scales.permute(2, 1, 0).contiguous().permute(2, 1, 0).to(device="cuda")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 134-159 / 第 134-159 行

~~~~python
    if shuffle_a:
        # shuffle within each group of 8 elements
        perm = torch.tensor([0, 2, 1, 3, 4, 6, 5, 7], device=a_quant.device)
        a_shuffled = (
            a_quant.view(m, k // 8, 8, l)[:, :, perm, :]
            .reshape(a_quant.shape)
            .permute(2, 0, 1)
            .contiguous()
            .permute(1, 2, 0)
        )
        # Construct A quantized tensor
        cute_a_quant_tensor, torch_a_quant_tensor = cutlass_torch.cute_tensor_like(
            a_shuffled,
            dtype,
            is_dynamic_layout=is_dynamic_layout,
            assumed_align=divisibility,
        )
    else:
        # Construct A quantized tensor
        cute_a_quant_tensor, torch_a_quant_tensor = cutlass_torch.cute_tensor_like(
            a_quant,
            dtype,
            is_dynamic_layout=is_dynamic_layout,
            assumed_align=divisibility,
        )
    cute_scale_tensor = from_dlpack(a_scales, assumed_align=divisibility)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 160-167 / 第 160-167 行

~~~~python
    for i, stride in enumerate(a_scales.stride()):
        if stride == 1:
            leading_dim = i
            break
    if is_dynamic_layout:
        cute_scale_tensor = cute_scale_tensor.mark_layout_dynamic(
            leading_dim=leading_dim
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 169-176 / 第 169-176 行

~~~~python
    return (
        cute_a_quant_tensor,
        torch_a_quant_tensor,
        a_quant.to("cpu"),
        cute_scale_tensor,
        a_scales,
        a_scales.to("cpu"),
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 179-206 / 第 179-206 行

~~~~python
def create_tensor_a(
    l: int,
    m: int,
    k: int,
    a_major: str,
    a_dtype: type[cutlass.Numeric],
    shuffle_a: bool,
    scale_granularity_m: int = 0,
    scale_granularity_k: int = 0,
    transformed_dtype: Optional[type[cutlass.Numeric]] = None,
) -> tuple[cute.Tensor, Optional[cute.Tensor], torch.Tensor, Optional[torch.Tensor]]:
    """
    Create tensor A and scale tensor.
    """
    a_scale_tensor = None
    a_scale_torch_cpu = None
    if a_dtype in (cutlass.Int4,):
        (
            a_tensor,
            a_torch_gpu,
            a_torch_cpu,
            a_scale_tensor,
            a_scale_torch_gpu,
            a_scale_torch_cpu,
        ) = create_i4_tensor_and_scale(
            l,
            m,
            k,
~~~~

**EN**: Defines `create_tensor_a`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_tensor_a`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 207-231 / 第 207-231 行

~~~~python
            a_major == "m",
            a_dtype,
            shuffle_a,
            scale_granularity_m,
            scale_granularity_k,
            divisibility=mixed_input_utils.get_divisibility(m if a_major == "m" else k),
            transformed_dtype=transformed_dtype,
        )
    else:
        a_torch_cpu = cutlass_torch.matrix(
            l,
            m,
            k,
            a_major == "m",
            a_dtype,
        )
        a_tensor, _ = cutlass_torch.cute_tensor_like(
            a_torch_cpu,
            a_dtype,
            is_dynamic_layout=True,
            assumed_align=mixed_input_utils.get_divisibility(
                m if a_major == "m" else k
            ),
        )
    return a_tensor, a_scale_tensor, a_torch_cpu, a_scale_torch_cpu
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 234-261 / 第 234-261 行

~~~~python
def create_tensors_for_contiguous_grouped_mixed_input_gemm(
    l: int,
    m: int,
    n: int,
    k: int,
    a_major: str,
    b_major: str,
    c_major: str,
    a_dtype: type[cutlass.Numeric],
    b_dtype: type[cutlass.Numeric],
    c_dtype: type[cutlass.Numeric],
    shuffle_a: bool = False,
    scale_granularity_m: int = 0,
    scale_granularity_k: int = 0,
    uniform_group_sizes: bool = False,
) -> tuple:
    """
    Create all input and output tensors for the contiguous grouped mixed-input GEMM.
    """
    a_tensor, a_scale_tensor, a_torch_cpu, a_scale_torch_cpu = create_tensor_a(
        l,
        m,
        k,
        a_major,
        a_dtype,
        shuffle_a,
        scale_granularity_m,
        scale_granularity_k,
~~~~

**EN**: Defines `create_tensors_for_contiguous_grouped_mixed_input_gemm`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_tensors_for_contiguous_grouped_mixed_input_gemm`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 262-263 / 第 262-263 行

~~~~python
        b_dtype,
    )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 265-266 / 第 265-266 行

~~~~python
    # In GROUP mode, l specifies the number of groups. We'll fuse group into the n mode for tensor B and C.
    # Batch mode will be set to 1.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 267-283 / 第 267-283 行

~~~~python
    num_groups = l
    fused_n = n * num_groups
    b_torch_cpu = cutlass_torch.matrix(
        1,  # batch=1
        fused_n,
        k,
        b_major == "n",
        b_dtype,
        cutlass_torch.TensorInitType.RANDOM,
        cutlass_torch.RandomInitConfig(min_val=-10, max_val=10),
    )
    b_tensor, _ = cutlass_torch.cute_tensor_like(
        b_torch_cpu,
        b_dtype,
        is_dynamic_layout=True,
        assumed_align=mixed_input_utils.get_divisibility(n if b_major == "n" else k),
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 285-307 / 第 285-307 行

~~~~python
    c_torch_cpu = cutlass_torch.matrix(
        1,  # batch=1
        m,
        fused_n,
        c_major == "m",
        c_dtype,
    )
    c_tensor, c_torch_gpu = cutlass_torch.cute_tensor_like(
        c_torch_cpu,
        c_dtype,
        is_dynamic_layout=True,
        assumed_align=mixed_input_utils.get_divisibility(m if c_major == "m" else n),
    )
    c_tensor = c_tensor.mark_compact_shape_dynamic(
        mode=(0 if c_major == "m" else 1),
        stride_order=(2, 1, 0) if c_major == "m" else (2, 0, 1),
        divisibility=mixed_input_utils.get_divisibility(m if c_major == "m" else n),
    )
    # We need to ensure mode N satisfies 16B alignment for each group
    alignment_n = 16 * 8 // b_dtype.width
    cumsum_tensor, cumsum_torch = create_cumsum_tensor(
        num_groups, fused_n, alignment_n, uniform_distribution=uniform_group_sizes
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 309-320 / 第 309-320 行

~~~~python
    return (
        a_tensor,
        a_scale_tensor,
        b_tensor,
        cumsum_tensor,
        c_tensor,
        a_torch_cpu,
        a_scale_torch_cpu,
        b_torch_cpu,
        cumsum_torch,
        c_torch_gpu,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 323-341 / 第 323-341 行

~~~~python
def create_tensors_for_batched_mixed_input_gemm(
    l: int,
    m: int,
    n: int,
    k: int,
    a_major: str,
    b_major: str,
    c_major: str,
    a_dtype: type[cutlass.Numeric],
    b_dtype: type[cutlass.Numeric],
    c_dtype: type[cutlass.Numeric],
    shuffle_a: bool = False,
    scale_granularity_m: int = 0,
    scale_granularity_k: int = 0,
) -> tuple:
    """
    Create all input and output tensors for the batched mixed-input GEMM.
    """
    torch.manual_seed(2025)
~~~~

**EN**: Defines `create_tensors_for_batched_mixed_input_gemm`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_tensors_for_batched_mixed_input_gemm`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 343-353 / 第 343-353 行

~~~~python
    a_tensor, a_scale_tensor, a_torch_cpu, a_scale_torch_cpu = create_tensor_a(
        l,
        m,
        k,
        a_major,
        a_dtype,
        shuffle_a,
        scale_granularity_m,
        scale_granularity_k,
        b_dtype,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 355-370 / 第 355-370 行

~~~~python
    b_torch_cpu = cutlass_torch.matrix(
        l,
        n,
        k,
        b_major == "n",
        b_dtype,
        cutlass_torch.TensorInitType.RANDOM,
        cutlass_torch.RandomInitConfig(min_val=-10, max_val=10),
    )
    c_torch_cpu = cutlass_torch.matrix(
        l,
        m,
        n,
        c_major == "m",
        c_dtype,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 372-388 / 第 372-388 行

~~~~python
    b_tensor, _ = cutlass_torch.cute_tensor_like(
        b_torch_cpu,
        b_dtype,
        is_dynamic_layout=True,
        assumed_align=mixed_input_utils.get_divisibility(n if b_major == "n" else k),
    )
    c_tensor, c_torch_gpu = cutlass_torch.cute_tensor_like(
        c_torch_cpu,
        c_dtype,
        is_dynamic_layout=True,
        assumed_align=mixed_input_utils.get_divisibility(m if c_major == "m" else n),
    )
    c_tensor = c_tensor.mark_compact_shape_dynamic(
        mode=(0 if c_major == "m" else 1),
        stride_order=(2, 1, 0) if c_major == "m" else (2, 0, 1),
        divisibility=mixed_input_utils.get_divisibility(m if c_major == "m" else n),
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 390-399 / 第 390-399 行

~~~~python
    return (
        a_tensor,
        a_scale_tensor,
        b_tensor,
        c_tensor,
        a_torch_cpu,
        a_scale_torch_cpu,
        b_torch_cpu,
        c_torch_gpu,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 402-429 / 第 402-429 行

~~~~python
def run_contiguous_grouped_ref_and_compare(
    a_torch_cpu: torch.Tensor,
    b_torch_cpu: torch.Tensor,
    a_scale_torch_cpu: Optional[torch.Tensor],
    cumsum_torch_cpu: torch.Tensor,
    c_torch_gpu: torch.Tensor,
    c_dtype: type[cutlass.Numeric],
    tolerance: float,
) -> None:
    """
    Compare kernel result with reference computation.
    """
    kernel_result = c_torch_gpu.cpu()
    assert kernel_result.shape[2] == 1, "batch mode must be 1"
    kernel_result = kernel_result.reshape(
        kernel_result.shape[0], kernel_result.shape[1]
    )
    # Compute reference result
    a_for_gemm = a_torch_cpu
    if a_scale_torch_cpu is not None:
        scale_shape = a_scale_torch_cpu.shape
        a_shape = a_torch_cpu.shape
        a_scale_torch_cpu = a_scale_torch_cpu.to(dtype=torch.float32).reshape(
            scale_shape[0], scale_shape[1], 1, scale_shape[2]
        )
        a_torch_cpu = a_torch_cpu.to(dtype=torch.float32).reshape(
            a_torch_cpu.shape[0], scale_shape[1], -1, a_torch_cpu.shape[2]
        )
~~~~

**EN**: Defines `run_contiguous_grouped_ref_and_compare`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `run_contiguous_grouped_ref_and_compare`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 430-454 / 第 430-454 行

~~~~python
        a_for_gemm = (a_torch_cpu * a_scale_torch_cpu).reshape(a_shape)
    # A in (m, k, l), b in (n, k), c in (m, n)
    assert cumsum_torch_cpu.shape[0] == a_for_gemm.shape[-1] + 1, (
        "cumsum tensor must have one more element than a_for_gemm"
    )
    assert b_torch_cpu.shape[2] == 1, (
        "b_torch_cpu must have a singleton dimension in the last position"
    )
    prev_idx = 0
    ref = torch.zeros((a_for_gemm.shape[0], b_torch_cpu.shape[0]), dtype=torch.float32)
    for group_idx in range(1, cumsum_torch_cpu.shape[0]):
        # No computation for current group
        if cumsum_torch_cpu[group_idx] == prev_idx:
            continue
        # Get A slice for current group
        sliced_a = a_for_gemm[:, :, group_idx - 1]
        # Get B slice for current group
        sliced_b = b_torch_cpu[prev_idx : cumsum_torch_cpu[group_idx], :, 0]
        sliced_ref = torch.einsum(
            "mk,nk->mn",
            sliced_a.to(dtype=torch.float32),
            sliced_b.to(dtype=torch.float32),
        )
        ref[:, prev_idx : cumsum_torch_cpu[group_idx]] = sliced_ref
        prev_idx = cumsum_torch_cpu[group_idx]
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 455-459 / 第 455-459 行

~~~~python
    # Convert ref to c_dtype
    _, ref_torch_gpu = cutlass_torch.cute_tensor_like(
        ref, c_dtype, is_dynamic_layout=True, assumed_align=16
    )
    ref_result = ref_torch_gpu.cpu()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 461-461 / 第 461-461 行

~~~~python
    # Assert close results
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 462-462 / 第 462-462 行

~~~~python
    torch.testing.assert_close(kernel_result, ref_result, atol=tolerance, rtol=1e-05)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 465-492 / 第 465-492 行

~~~~python
def run_batched_mixed_input_ref_and_compare(
    a_torch_cpu: torch.Tensor,
    b_torch_cpu: torch.Tensor,
    a_scale_torch_cpu: Optional[torch.Tensor],
    c_torch_gpu: torch.Tensor,
    c_dtype: type[cutlass.Numeric],
    tolerance: float,
) -> None:
    """
    Compare kernel result with reference computation.
    """
    kernel_result = c_torch_gpu.cpu()
    # Compute reference result
    if a_scale_torch_cpu is not None:
        scale_shape = a_scale_torch_cpu.shape
        a_shape = a_torch_cpu.shape
        a_scale_torch_cpu = a_scale_torch_cpu.to(dtype=torch.float32).reshape(
            scale_shape[0], scale_shape[1], 1, scale_shape[2]
        )
        a_torch_cpu = a_torch_cpu.to(dtype=torch.float32).reshape(
            a_torch_cpu.shape[0], scale_shape[1], -1, a_torch_cpu.shape[2]
        )
        a_dequant = a_torch_cpu * a_scale_torch_cpu
        ref = torch.einsum(
            "mkl,nkl->mnl",
            a_dequant.reshape(a_shape),
            b_torch_cpu.to(dtype=torch.float32),
        )
~~~~

**EN**: Defines `run_batched_mixed_input_ref_and_compare`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `run_batched_mixed_input_ref_and_compare`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 493-503 / 第 493-503 行

~~~~python
    else:
        ref = torch.einsum(
            "mkl,nkl->mnl",
            a_torch_cpu.to(dtype=torch.float32),
            b_torch_cpu.to(dtype=torch.float32),
        )
    # Convert ref to c_dtype
    _, ref_torch_gpu = cutlass_torch.cute_tensor_like(
        ref, c_dtype, is_dynamic_layout=True, assumed_align=16
    )
    ref_result = ref_torch_gpu.cpu()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 505-505 / 第 505-505 行

~~~~python
    # Assert close results
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 506-506 / 第 506-506 行

~~~~python
    torch.testing.assert_close(kernel_result, ref_result, atol=tolerance, rtol=1e-05)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出
- Fused epilogues and low-precision numerics / 融合式尾处理与低精度数值

## Dependencies / 依赖项

- `typing.Optional` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.mixed_input_helpers` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
