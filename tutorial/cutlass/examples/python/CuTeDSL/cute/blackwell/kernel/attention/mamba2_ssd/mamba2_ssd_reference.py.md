# mamba2_ssd_reference.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/attention/mamba2_ssd/mamba2_ssd_reference.py`  
**Purpose / 用途**: Kernel example implementing mamba2 ssd reference with CuTeDSL. / 这是一个使用 CuTeDSL 实现 mamba2 ssd reference 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-30 / 第 29-30 行

~~~~python
import torch
import torch.nn.functional as F
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 33-48 / 第 33-48 行

~~~~python
def ssd_reference_fp32_all(x, a, delta, B, C, Y_out, Fstate_out, D, has_d, d_has_hdim):
    """
    Rearrange tensor dimensions from cuda layout to reference layout, then directly call TriDao's ssd implementation
    Arguments:
        X/x: (D, L, C, H, B):(C*L, 1, L, D*C*L, H*D*C*L)
        A/delta: (L, C, H, B):(1, L, C*L, H*C*L)
        a: (H):(1)
        B/C: (L, N, C, G, B):(1, C*L, L, N*C*L, G*N*C*L)
        D: (1, H):(0, 1) or (D, H):(1, D)
        has_d: bool
        d_has_hdim: bool
    Return:
        Y_out: (L, D, C, H, B):(1, C*L, L, D*C*L, H*D*C*L)
        Fstate_out: (D, N, H, B):(N, 1, D*N, H*D*N)
    """
    assert x.dtype == a.dtype == delta.dtype == B.dtype == C.dtype
~~~~

**EN**: Defines `ssd_reference_fp32_all`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `ssd_reference_fp32_all`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 50-51 / 第 50-51 行

~~~~python
    A = delta * a.view(1, 1, -1, 1)
    X = x * delta.unsqueeze(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 53-53 / 第 53-53 行

~~~~python
    # Rearrange to match cutlass layout to tridao's layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 54-65 / 第 54-65 行

~~~~python
    block_len = A.shape[0]
    initial_states = None
    # A: l c h b-> b c l h
    A = A.permute(3, 1, 0, 2)
    # X: p l c h b -> b c l h p
    X = X.permute(4, 2, 1, 3, 0)
    # B: l n c g b -> b c l g n
    B = B.permute(4, 2, 0, 3, 1)
    # C: l n c g b -> b c l g n
    C = C.permute(4, 2, 0, 3, 1)
    # X/A/B/C: b c l ... -> b (c l) ...
    X, A, B, C = [x.reshape(x.shape[0], -1, *x.shape[3:]) for x in (X, A, B, C)]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 67-67 / 第 67-67 行

~~~~python
    # Ngroup (g to h) mapping
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 68-74 / 第 68-74 行

~~~~python
    B_val, CL_val, G_val, N_val = B.shape
    H_val = X.shape[2]
    ngroup_ratio = H_val // G_val
    # B/C: (B, CL, H, N)
    h_to_g_mapping = torch.arange(H_val, device=B.device) // ngroup_ratio
    B = B.gather(2, h_to_g_mapping.view(1, 1, -1, 1).expand(B_val, CL_val, -1, N_val))
    C = C.gather(2, h_to_g_mapping.view(1, 1, -1, 1).expand(B_val, CL_val, -1, N_val))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 76-77 / 第 76-77 行

~~~~python
    ###################################################################
    # Call reference implementation from Tri Dao ssd_minimal_discrete
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 78-81 / 第 78-81 行

~~~~python
    Y, final_state = ssd_minimal_discrete_fp32_all(
        X, A, B, C, block_len, initial_states
    )
    ###################################################################
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。

### Lines 83-87 / 第 83-87 行

~~~~python
    if has_d:
        D_val = Y.shape[3]
        if not d_has_hdim:
            D = D.expand(D_val, -1)
        Y = Y + torch.einsum("bchp,ph->bchp", X, D)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 89-90 / 第 89-90 行

~~~~python
    # Rearrange to match tridao's layout to cutlass layout
    # Y: b (c l) h p -> b c l h p
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 91-97 / 第 91-97 行

~~~~python
    Y = Y.reshape(Y.shape[0], -1, block_len, Y.shape[2], Y.shape[3])
    # Y: b c l h p -> l p c h b
    Y = Y.permute(2, 4, 1, 3, 0)
    # Fstate_out: b h p n -> p n h b
    Fstate_out.copy_(final_state.permute(2, 3, 1, 0))
    Y_out.copy_(Y)
    return
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 100-118 / 第 100-118 行

~~~~python
def ssd_reference_lowprecision_intermediates(
    x, a, delta, B, C, Y_out, Fstate_out, intermediate_dtype, D, has_d, d_has_hdim
):
    """
    Rearrange tensor dimensions from cuda layout to reference layout, then call a reduced intermediate dtype version of ssd implementation
    Arguments:
        X/x: (D, L, C, H, B):(C*L, 1, L, D*C*L, H*D*C*L)
        A/delta: (L, C, H, B):(1, L, C*L, H*C*L)
        a: (H):(1)
        B/C: (L, N, C, G, B):(1, C*L, L, N*C*L, G*N*C*L)
        intermediate_dtype: input and intermediate data type
        D: (1, H):(0, 1) or (D, H):(1, D)
        has_d: bool
        d_has_hdim: bool
    Return:
        Y_out: (L, D, C, H, B):(1, C*L, L, D*C*L, H*D*C*L)
        Fstate_out: (D, N, H, B):(N, 1, D*N, H*D*N)
    """
    assert x.dtype == a.dtype == delta.dtype == B.dtype == C.dtype
~~~~

**EN**: Defines `ssd_reference_lowprecision_intermediates`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `ssd_reference_lowprecision_intermediates`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 120-120 / 第 120-120 行

~~~~python
    A = delta * a.view(1, 1, -1, 1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 122-122 / 第 122-122 行

~~~~python
    # Rearrange to match cutlass layout to tridao's layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 123-139 / 第 123-139 行

~~~~python
    block_len = A.shape[0]
    initial_states = None
    # A: l c h b-> b c l h
    A = A.permute(3, 1, 0, 2)
    # delta: l c h b-> b c l h
    delta = delta.permute(3, 1, 0, 2)
    # x: p l c h b -> b c l h p
    x = x.permute(4, 2, 1, 3, 0)
    # B: l n c g b -> b c l g n
    B = B.permute(4, 2, 0, 3, 1)
    # C: l n c g b -> b c l g n
    C = C.permute(4, 2, 0, 3, 1)
    # x/A/delta/B/C: b c l ... -> b (c l) ...
    x, A, delta, B, C = [
        tensor.reshape(tensor.shape[0], -1, *tensor.shape[3:])
        for tensor in (x, A, delta, B, C)
    ]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 141-141 / 第 141-141 行

~~~~python
    # Ngroup (g to h) mapping
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 142-148 / 第 142-148 行

~~~~python
    B_val, CL_val, G_val, N_val = B.shape
    H_val = x.shape[2]
    ngroup_ratio = H_val // G_val
    # B/C: (B, CL, H, N)
    h_to_g_mapping = torch.arange(H_val, device=B.device) // ngroup_ratio
    B = B.gather(2, h_to_g_mapping.view(1, 1, -1, 1).expand(B_val, CL_val, -1, N_val))
    C = C.gather(2, h_to_g_mapping.view(1, 1, -1, 1).expand(B_val, CL_val, -1, N_val))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 150-150 / 第 150-150 行

~~~~python
    # Type convert input tensors to input dtype (same as intermediate dtype)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 151-155 / 第 151-155 行

~~~~python
    x = x.to(intermediate_dtype).to(torch.float32)
    A = A.to(intermediate_dtype).to(torch.float32)
    delta = delta.to(intermediate_dtype).to(torch.float32)
    B = B.to(intermediate_dtype).to(torch.float32)
    C = C.to(intermediate_dtype).to(torch.float32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 157-158 / 第 157-158 行

~~~~python
    #########################################################################
    # Call reference implementation ssd_minimal_discrete_bf16_intermediates
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 159-162 / 第 159-162 行

~~~~python
    Y, final_state = ssd_minimal_discrete_lowprecision_intermediates(
        x, A, delta, B, C, block_len, intermediate_dtype, initial_states
    )
    #########################################################################
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。

### Lines 164-169 / 第 164-169 行

~~~~python
    if has_d:
        D = D.to(intermediate_dtype).to(torch.float32)
        D_val = Y.shape[3]
        if not d_has_hdim:
            D = D.expand(D_val, -1)
        Y = Y + torch.einsum("bchp,ph->bchp", x, D)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 171-171 / 第 171-171 行

~~~~python
    # Type convert output tensors to output dtype (same as intermediate dtype)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 172-173 / 第 172-173 行

~~~~python
    Y = Y.to(intermediate_dtype).to(torch.float32)
    final_state = final_state.to(intermediate_dtype).to(torch.float32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 175-176 / 第 175-176 行

~~~~python
    # Rearrange to match tridao's layout to cutlass layout
    # Y: b (c l) h p -> b c l h p
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 177-183 / 第 177-183 行

~~~~python
    Y = Y.reshape(Y.shape[0], -1, block_len, Y.shape[2], Y.shape[3])
    # Y: b c l h p -> l p c h b
    Y = Y.permute(2, 4, 1, 3, 0)
    # Fstate_out: b h p n -> p n h b
    Fstate_out.copy_(final_state.permute(2, 3, 1, 0))
    Y_out.copy_(Y)
    return
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 186-192 / 第 186-192 行

~~~~python
def analyze_relative_diffs(actual, expected):
    """
    Print statistics of relative differences between actual and expected tensors
    """
    # Calculate relative differences
    abs_diff = (actual - expected).abs()
    rel_diff = abs_diff / (torch.maximum(expected.abs(), actual.abs()) + 0.00001)
~~~~

**EN**: Defines `analyze_relative_diffs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `analyze_relative_diffs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 194-194 / 第 194-194 行

~~~~python
    total_elements = rel_diff.numel()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 196-196 / 第 196-196 行

~~~~python
    # Handle special cases first
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 197-200 / 第 197-200 行

~~~~python
    nan_mask = torch.isnan(rel_diff)
    inf_mask = torch.isinf(rel_diff)
    nan_count = nan_mask.sum().item()
    inf_count = inf_mask.sum().item()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 202-202 / 第 202-202 行

~~~~python
    # Find position and value of maximum relative difference
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 203-212 / 第 203-212 行

~~~~python
    max_rel_diff = (
        rel_diff[~nan_mask & ~inf_mask].max()
        if (~nan_mask & ~inf_mask).any()
        else float("nan")
    )
    max_rel_diff_pos = (
        rel_diff[~nan_mask & ~inf_mask].argmax()
        if (~nan_mask & ~inf_mask).any()
        else -1
    )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 214-214 / 第 214-214 行

~~~~python
    # Print max relative difference info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 215-221 / 第 215-221 行

~~~~python
    print("Maximum relative difference:")
    print(f"Position: {max_rel_diff_pos}")
    print(f"Value: {max_rel_diff:.6e}")
    print(f"Actual value: {actual.flatten()[max_rel_diff_pos]}")
    print(f"Expected value: {expected.flatten()[max_rel_diff_pos]}")
    print(f"NaN values: {nan_count} ({100.0 * nan_count / total_elements:.2f}%)")
    print(f"Inf values: {inf_count} ({100.0 * inf_count / total_elements:.2f}%)\n")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 223-223 / 第 223-223 行

~~~~python
    # Check different rtol thresholds
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 224-224 / 第 224-224 行

~~~~python
    rtol_levels = [1e-5, 1e-4, 1e-3, 1e-2, 5e-02, 1e-01]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 226-230 / 第 226-230 行

~~~~python
    for i, rtol in enumerate(rtol_levels):
        if i == 0:
            mask = rel_diff <= rtol
        else:
            mask = (rel_diff <= rtol) & (rel_diff > rtol_levels[i - 1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 232-233 / 第 232-233 行

~~~~python
        count = mask.sum().item()
        percentage = (count / total_elements) * 100
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 235-240 / 第 235-240 行

~~~~python
        if i == 0:
            print(f"Elements with rtol <= {rtol:.0e}: {count} ({percentage:.2f}%)")
        else:
            print(
                f"Elements with {rtol_levels[i - 1]:.0e} < rtol <= {rtol:.0e}: {count} ({percentage:.2f}%)"
            )
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 242-242 / 第 242-242 行

~~~~python
    # Print elements exceeding the largest rtol
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 243-246 / 第 243-246 行

~~~~python
    mask = rel_diff > rtol_levels[-1]
    count = mask.sum().item()
    percentage = (count / total_elements) * 100
    print(f"Elements with rtol > {rtol_levels[-1]:.0e}: {count} ({percentage:.2f}%)\n")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 249-262 / 第 249-262 行

~~~~python
def segsum(x):
    """
    More stable segment sum calculation.
    x: b h c l
    """
    T = x.size(-1)
    # x: b h c l -> b h c l l
    x = x.unsqueeze(-1).expand(*x.shape, T)
    mask = torch.tril(torch.ones(T, T, device=x.device, dtype=bool), diagonal=-1)
    x = x.masked_fill(~mask, 0)
    x_segsum = torch.cumsum(x, dim=-2)
    mask = torch.tril(torch.ones(T, T, device=x.device, dtype=bool), diagonal=0)
    x_segsum = x_segsum.masked_fill(~mask, -torch.inf)
    return x_segsum
~~~~

**EN**: Defines `segsum`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `segsum`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 265-280 / 第 265-280 行

~~~~python
def ssd_minimal_discrete_fp32_all(X, A, B, C, block_len, initial_states=None):
    """
    This is same with https://github.com/state-spaces/mamba/blob/main/mamba_ssm/modules/ssd_minimal.py
    (all accumulation and intermediate results in fp32)

    Arguments:
        X: (batch(B), length(C*L), n_heads(H), d_head(D))
        A: (batch(B), length(C*L), n_heads(H))
        B: (batch(B), length(C*L), n_heads(H), d_state(N))
        C: (batch(B), length(C*L), n_heads(H), d_state(N))
    Return:
        Y: (batch(B), length(C*L), n_heads(H), d_head(D))
        final_state: (B, H, D, N)
    """
    assert X.dtype == A.dtype == B.dtype == C.dtype
    assert X.shape[1] % block_len == 0
~~~~

**EN**: Defines `ssd_minimal_discrete_fp32_all`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `ssd_minimal_discrete_fp32_all`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 282-283 / 第 282-283 行

~~~~python
    # Rearrange into blocks/chunks
    # X/A/B/C:b (c l) ... -> b c l ...
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 284-286 / 第 284-286 行

~~~~python
    X, A, B, C = [
        x.reshape(x.shape[0], -1, block_len, *x.shape[2:]) for x in (X, A, B, C)
    ]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 288-288 / 第 288-288 行

~~~~python
    # A: b c l h -> b h c l
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 289-291 / 第 289-291 行

~~~~python
    A = A.permute(0, 3, 1, 2)
    # A_cumsum: (B, H, C, L)
    A_cumsum = torch.cumsum(A, dim=-1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 293-293 / 第 293-293 行

~~~~python
    # 1. Compute the output for each intra-chunk (diagonal blocks)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 294-296 / 第 294-296 行

~~~~python
    segsum_A = segsum(A)
    L = torch.exp(segsum_A)
    Y_diag = torch.einsum("bclhn,bcshn,bhcls,bcshp->bclhp", C, B, L, X)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 298-299 / 第 298-299 行

~~~~python
    # 2. Compute the state for each intra-chunk
    # (right term of low-rank factorization of off-diagonal blocks; B terms)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 300-301 / 第 300-301 行

~~~~python
    decay_states = torch.exp((A_cumsum[:, :, :, -1:] - A_cumsum))
    states = torch.einsum("bclhn,bhcl,bclhp->bchpn", B, decay_states, X)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 303-304 / 第 303-304 行

~~~~python
    # 3. Compute the inter-chunk SSM recurrence; produces correct SSM states at chunk boundaries
    # (middle term of factorization of off-diag blocks; A terms)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 305-310 / 第 305-310 行

~~~~python
    if initial_states is None:
        initial_states = torch.zeros_like(states[:, :1])
    states = torch.cat([initial_states, states], dim=1)
    decay_chunk = torch.exp(segsum(F.pad(A_cumsum[:, :, :, -1], (1, 0))))
    new_states = torch.einsum("bhzc,bchpn->bzhpn", decay_chunk, states)
    states, final_state = new_states[:, :-1], new_states[:, -1]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 312-313 / 第 312-313 行

~~~~python
    # 4. Compute state -> output conversion per chunk
    # (left term of low-rank factorization of off-diagonal blocks; C terms)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 314-315 / 第 314-315 行

~~~~python
    state_decay_out = torch.exp(A_cumsum)
    Y_off = torch.einsum("bclhn,bchpn,bhcl->bclhp", C, states, state_decay_out)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 317-318 / 第 317-318 行

~~~~python
    # Add output of intra-chunk and inter-chunk terms (diagonal and off-diagonal blocks)
    # Y: b c l h p -> b (c l) h p
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 319-320 / 第 319-320 行

~~~~python
    Y = (Y_diag + Y_off).reshape(Y_diag.shape[0], -1, Y_diag.shape[3], Y_diag.shape[4])
    return Y, final_state
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 323-342 / 第 323-342 行

~~~~python
def ssd_minimal_discrete_lowprecision_intermediates(
    X, A, delta, B, C, block_len, intermediate_dtype, initial_states=None
):
    """
    This is adjusted from ssd_minimal_discrete_fp32_all, with exceptions:
    1. accumulation in fp32 but intermediates Q/b_tmem/P are in intermediate_dtype
    2. delta is not pre-multiplied with X, delta was applied to generate Q/b_tmem to match GPU implementation

    Arguments:
        X: (batch(B), length(C*L), n_heads(H), d_head(D))
        A: (batch(B), length(C*L), n_heads(H))
        delta: (batch(B), length(C*L), n_heads(H))
        B: (batch(B), length(C*L), n_heads(H), d_state(N))
        C: (batch(B), length(C*L), n_heads(H), d_state(N))
    Return:
        Y: (batch(B), length(C*L), n_heads(H), d_head(D))
        final_state: (B, H, D, N)
    """
    assert X.dtype == A.dtype == B.dtype == C.dtype
    assert X.shape[1] % block_len == 0
~~~~

**EN**: Defines `ssd_minimal_discrete_lowprecision_intermediates`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `ssd_minimal_discrete_lowprecision_intermediates`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 344-345 / 第 344-345 行

~~~~python
    # Rearrange into blocks/chunks
    # X/A/delta/B/C: b (c l) ... -> b c l ...
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 346-348 / 第 346-348 行

~~~~python
    X, A, delta, B, C = [
        x.reshape(x.shape[0], -1, block_len, *x.shape[2:]) for x in (X, A, delta, B, C)
    ]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 350-350 / 第 350-350 行

~~~~python
    # A: b c l h -> b h c l
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 351-355 / 第 351-355 行

~~~~python
    A = A.permute(0, 3, 1, 2)
    # delta: b c l h -> b h c l
    delta = delta.permute(0, 3, 1, 2)
    # A_cumsum: (B, H, C, L)
    A_cumsum = torch.cumsum(A, dim=-1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 357-357 / 第 357-357 行

~~~~python
    # 1. Compute the output for each intra-chunk (diagonal blocks)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 358-364 / 第 358-364 行

~~~~python
    segsum_A = segsum(A)
    L = torch.exp(segsum_A)
    intra_acc_0 = torch.einsum("bclhn,bcshn->bclhs", C, B)
    Q = torch.einsum("bclhs,bhcls,bhcs->bclhs", intra_acc_0, L, delta)
    Y_diag = torch.einsum(
        "bclhs,bcshp->bclhp", Q.to(intermediate_dtype).to(torch.float32), X
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 366-367 / 第 366-367 行

~~~~python
    # 2. Compute the state for each intra-chunk
    # (right term of low-rank factorization of off-diagonal blocks; B terms)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 368-372 / 第 368-372 行

~~~~python
    decay_states = torch.exp((A_cumsum[:, :, :, -1:] - A_cumsum))
    b_tmem = torch.einsum("bclhn,bhcl,bhcl->bclhn", B, decay_states, delta)
    states = torch.einsum(
        "bclhn,bclhp->bchpn", b_tmem.to(intermediate_dtype).to(torch.float32), X
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 374-375 / 第 374-375 行

~~~~python
    # 3. Compute the inter-chunk SSM recurrence; produces correct SSM states at chunk boundaries
    # (middle term of factorization of off-diag blocks; A terms)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 376-382 / 第 376-382 行

~~~~python
    if initial_states is None:
        initial_states = torch.zeros_like(states[:, :1])
    states = torch.cat([initial_states, states], dim=1)
    decay_chunk = torch.exp(segsum(F.pad(A_cumsum[:, :, :, -1], (1, 0))))
    new_states = torch.einsum("bhzc,bchpn->bzhpn", decay_chunk, states)
    states, final_state = new_states[:, :-1], new_states[:, -1]
    final_state = final_state
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 384-385 / 第 384-385 行

~~~~python
    # 4. Compute state -> output conversion per chunk
    # (left term of low-rank factorization of off-diagonal blocks; C terms)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 386-390 / 第 386-390 行

~~~~python
    state_decay_out = torch.exp(A_cumsum)
    Y_off_tmp = torch.einsum(
        "bclhn,bchpn->bclhp", C, states.to(intermediate_dtype).to(torch.float32)
    )
    Y_off = torch.einsum("bclhp,bhcl->bclhp", Y_off_tmp, state_decay_out)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 392-393 / 第 392-393 行

~~~~python
    # Add output of intra-chunk and inter-chunk terms (diagonal and off-diagonal blocks)
    # Y: b c l h p -> b (c l) h p
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 394-397 / 第 394-397 行

~~~~python
    Y = (Y_diag + Y_off).reshape(
        Y_diag.shape[0], -1, Y_diag.shape[3], Y_diag.shape[4]
    )  # b (c l) h p
    return Y, final_state
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Specialized sequence or attention dataflow / 专门化的序列/注意力数据流
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `torch.nn.functional` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
