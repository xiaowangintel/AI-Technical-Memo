# dense_block_scaled_gemm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute_ext/blackwell/dense_block_scaled_gemm.py`  
**Purpose / 用途**: Example module for dense block scaled gemm. / 这是一个关于 dense block scaled gemm 的 CuTeDSL 示例模块。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行

~~~~python
# Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 30-35 / 第 30-35 行

~~~~python
import argparse
from typing import Type, Tuple
from dataclasses import dataclass
import torch
import cutlass
from cutlass import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 36-43 / 第 36-43 行

~~~~python
    cute as cute,
    utils as utils,
)
from cutlass.cute import experimental as cute_ext
from cutlass.cute.runtime import from_dlpack
from cutlass.cute.nvgpu import cpasync, tcgen05
import cutlass.utils.blackwell_helpers as sm100_utils
import cutlass.utils.blockscaled_layout as blockscaled_utils
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 45-49 / 第 45-49 行

~~~~python
"""

This is an implementation of dense block scaled GEMM.

"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 50-66 / 第 50-66 行

~~~~python
class BlockScaledDenseGemmKernel:
    def __init__(
        self,
        mma_inst_mn: tuple[int, int],
        mma_dtype: tuple[Type[cutlass.Numeric], Type[cutlass.Numeric]],
        sf_dtype: Type[cutlass.Numeric],
        sf_vec_size: int,
        epilogue_op=lambda x: x,
    ):
        self.ab_dtype, self.acc_dtype = mma_dtype
        self.sf_dtype = sf_dtype
        self.sf_vec_size = sf_vec_size
        self.mma_inst_shape_mn = mma_inst_mn
        self.use_2cta_instrs = False
        self.cta_group = (
            tcgen05.CtaGroup.TWO if self.use_2cta_instrs else tcgen05.CtaGroup.ONE
        )
~~~~

**EN**: Defines `BlockScaledDenseGemmKernel`, a reusable Python class that packages configuration and behavior for this example. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `BlockScaledDenseGemmKernel`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 68-71 / 第 68-71 行

~~~~python
        self.epilogue_op = epilogue_op
        # TODO: instead of using max shared memory, we should define a SharedStorage and then
        # query its size.
        self.smem_capacity = utils.get_smem_capacity_in_bytes("sm_100")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 73-73 / 第 73-73 行

~~~~python
        # Stages
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 74-74 / 第 74-74 行

~~~~python
        self.num_acc_stages = 1 if self.mma_inst_shape_mn[1] == 256 else 2
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 76-77 / 第 76-77 行

~~~~python
        # TODO: provide a computation for this so that it is not fixed;
        # fitting as many stages as there is available shared memory
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 78-78 / 第 78-78 行

~~~~python
        self.num_main_stages = 4
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 80-80 / 第 80-80 行

~~~~python
        self.tma_store_stages = 4
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 82-100 / 第 82-100 行

~~~~python
    @cute.experimental.jit
    def __call__(
        self,
        mA: cute.Tensor,
        mSFA: cute.Tensor,
        mB: cute.Tensor,
        mSFB: cute.Tensor,
        mC: cute.Tensor,
    ):
        tile_mn = (*self.mma_inst_shape_mn, 1)
        div = cute.tiled_divide(mC, tile_mn)
        grid = (div.shape[1], div.shape[2], div.shape[3])
        self.kernel(mA, mSFA, mB, mSFB, mC).launch(
            grid=grid,
            # Using a total of 6 warps (1x load + 1x mma + 4x epilogue)
            block=(192, 1, 1),
            cluster=(1, 1, 1),
            smem=self.smem_capacity,
        )
~~~~

**EN**: Defines `__call__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `__call__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 102-115 / 第 102-115 行

~~~~python
    @cute.experimental.kernel
    def kernel(
        self,
        mA: cute.Tensor,
        mSFA: cute.Tensor,
        mB: cute.Tensor,
        mSFB: cute.Tensor,
        mC: cute.Tensor,
    ):
        # Prologue
        tidx, _, _ = cute.arch.thread_idx()
        warp_idx = cute.arch.warp_idx()
        warp_idx = cute.arch.make_warp_uniform(warp_idx)
        cta_m, cta_n, cta_l = cute.arch.block_idx()
~~~~

**EN**: Defines `kernel`, grouping related logic behind a named Python callable so the example stays modular and reusable. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `kernel`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 117-122 / 第 117-122 行

~~~~python
        a_dtype: Type[cutlass.Numeric] = mA.element_type
        sf_dtype: Type[cutlass.Numeric] = mSFA.element_type
        c_dtype: Type[cutlass.Numeric] = mC.element_type
        a_major_mode = utils.LayoutEnum.from_tensor(mA).mma_major_mode()
        b_major_mode = utils.LayoutEnum.from_tensor(mB).mma_major_mode()
        d_layout = utils.LayoutEnum.from_tensor(mC)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 124-132 / 第 124-132 行

~~~~python
        tiled_mma = sm100_utils.make_blockscaled_trivial_tiled_mma(
            a_dtype,
            a_major_mode,
            b_major_mode,
            sf_dtype,
            self.sf_vec_size,
            self.cta_group,
            self.mma_inst_shape_mn,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 134-140 / 第 134-140 行

~~~~python
        mma_inst_shape_k = cute.size(tiled_mma.shape_mnk, mode=[2])
        mma_inst_tile_k = 4
        mma_tiler_mnk = (
            self.mma_inst_shape_mn[0],
            self.mma_inst_shape_mn[1],
            mma_inst_shape_k * mma_inst_tile_k,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 142-144 / 第 142-144 行

~~~~python
        tiler_mk = (mma_tiler_mnk[0], mma_tiler_mnk[2])
        tiler_nk = (mma_tiler_mnk[1], mma_tiler_mnk[2])
        tiler_mn = (mma_tiler_mnk[0], mma_tiler_mnk[1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 146-146 / 第 146-146 行

~~~~python
        # ((Atom_M, Rest_M),(Atom_K, Rest_K), RestL)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 147-148 / 第 147-148 行

~~~~python
        sfa_layout = blockscaled_utils.tile_atom_to_shape_SF(mA.shape, self.sf_vec_size)
        sfa_tensor = cute.make_tensor(mSFA.iterator, sfa_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 150-150 / 第 150-150 行

~~~~python
        # ((Atom_N, Rest_N),(Atom_K, Rest_K), RestL)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 151-152 / 第 151-152 行

~~~~python
        sfb_layout = blockscaled_utils.tile_atom_to_shape_SF(mB.shape, self.sf_vec_size)
        sfb_tensor = cute.make_tensor(mSFB.iterator, sfb_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 154-158 / 第 154-158 行

~~~~python
        gA = cute.zipped_divide(mA, tiler_mk)
        gB = cute.zipped_divide(mB, tiler_nk)
        gSFA = cute.zipped_divide(sfa_tensor, tiler_mk)
        gSFB = cute.zipped_divide(sfb_tensor, tiler_nk)
        gC = cute.zipped_divide(mC, tiler_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 160-164 / 第 160-164 行

~~~~python
        gA_tile = gA[(None, None), (cta_m, None, cta_l)]
        gB_tile = gB[(None, None), (cta_n, None, cta_l)]
        gSFA_tile = gSFA[(None, None), (cta_m, None, cta_l)]
        gSFB_tile = gSFB[(None, None), (cta_n, None, cta_l)]
        gC_tile = gC[(None, None), (cta_m, cta_n, cta_l)]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 166-167 / 第 166-167 行

~~~~python
        # Shared memory layouts for A/B/SFA/SFB/D
        # (MMA, MMA_M, MMA_K, PIPE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 168-173 / 第 168-173 行

~~~~python
        a_smem_layout_staged = sm100_utils.make_smem_layout_a(
            tiled_mma,
            mma_tiler_mnk,
            self.ab_dtype,
            self.num_main_stages,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 175-175 / 第 175-175 行

~~~~python
        # (MMA, MMA_N, MMA_K, PIPE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 176-181 / 第 176-181 行

~~~~python
        b_smem_layout_staged = sm100_utils.make_smem_layout_b(
            tiled_mma,
            mma_tiler_mnk,
            self.ab_dtype,
            self.num_main_stages,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 183-183 / 第 183-183 行

~~~~python
        # (MMA, MMA_M, MMA_K, PIPE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 184-189 / 第 184-189 行

~~~~python
        sfa_smem_layout_staged = blockscaled_utils.make_smem_layout_sfa(
            tiled_mma,
            mma_tiler_mnk,
            self.sf_vec_size,
            self.num_main_stages,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 191-191 / 第 191-191 行

~~~~python
        # (MMA, MMA_N, MMA_K, PIPE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 192-197 / 第 192-197 行

~~~~python
        sfb_smem_layout_staged = blockscaled_utils.make_smem_layout_sfb(
            tiled_mma,
            mma_tiler_mnk,
            self.sf_vec_size,
            self.num_main_stages,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 199-213 / 第 199-213 行

~~~~python
        cta_tile_shape_mnk = cute.shape_div(
            mma_tiler_mnk, (cute.size(tiled_mma.thr_id.shape), 1, 1)
        )
        epi_tile = sm100_utils.compute_epilogue_tile_shape(
            cta_tile_shape_mnk,
            self.use_2cta_instrs,
            d_layout,
            c_dtype,
        )
        smem_epi_staged_layout = sm100_utils.make_smem_layout_epi(
            c_dtype,
            d_layout,
            epi_tile,
            self.tma_store_stages,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 215-216 / 第 215-216 行

~~~~python
        # UMMA ACC TMEM Layout
        # ((MMA_M, MMA_N), REST_MMA_M, REST_MMA_N, ACC_STAGES)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 217-219 / 第 217-219 行

~~~~python
        tmem_accs_layout = cute_ext.make_tmem_layout_acc(
            tiled_mma, mma_tiler_mnk, self.num_acc_stages
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 221-226 / 第 221-226 行

~~~~python
        sfa_tmem_layout = blockscaled_utils.make_tmem_layout_sfa(
            tiled_mma,
            mma_tiler_mnk,
            self.sf_vec_size,
            cute.slice_(sfa_smem_layout_staged, (None, None, None, 0)),
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 228-233 / 第 228-233 行

~~~~python
        sfb_tmem_layout = blockscaled_utils.make_tmem_layout_sfb(
            tiled_mma,
            mma_tiler_mnk,
            self.sf_vec_size,
            cute.slice_(sfb_smem_layout_staged, (None, None, None, 0)),
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 235-235 / 第 235-235 行

~~~~python
        # Allocate UMMA Buffers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 236-241 / 第 236-241 行

~~~~python
        buffer_smem_a = cute_ext.allocate(
            self.ab_dtype,
            cute.AddressSpace.smem,
            a_smem_layout_staged,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 243-248 / 第 243-248 行

~~~~python
        buffer_smem_b = cute_ext.allocate(
            self.ab_dtype,
            cute.AddressSpace.smem,
            b_smem_layout_staged,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 250-255 / 第 250-255 行

~~~~python
        buffer_smem_sfa = cute_ext.allocate(
            self.sf_dtype,
            cute.AddressSpace.smem,
            sfa_smem_layout_staged,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 257-262 / 第 257-262 行

~~~~python
        buffer_smem_sfb = cute_ext.allocate(
            self.sf_dtype,
            cute.AddressSpace.smem,
            sfb_smem_layout_staged,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 264-269 / 第 264-269 行

~~~~python
        buffer_tmem_accs = cute_ext.allocate(
            self.acc_dtype,
            cute.AddressSpace.tmem,
            tmem_accs_layout,
            alignment=16,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 271-276 / 第 271-276 行

~~~~python
        buffer_tmem_sfa = cute_ext.allocate(
            self.sf_dtype,
            cute.AddressSpace.tmem,
            sfa_tmem_layout,
            alignment=16,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 278-283 / 第 278-283 行

~~~~python
        buffer_tmem_sfb = cute_ext.allocate(
            self.sf_dtype,
            cute.AddressSpace.tmem,
            sfb_tmem_layout,
            alignment=16,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 285-286 / 第 285-286 行

~~~~python
        buffer_tmem_sfa_compact = cute.filter_zeros(buffer_tmem_sfa)
        buffer_tmem_sfb_compact = cute.filter_zeros(buffer_tmem_sfb)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 288-288 / 第 288-288 行

~~~~python
        # Make S2T CopyAtom and tiledCopy
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 289-292 / 第 289-292 行

~~~~python
        copy_atom_s2t = cute.make_copy_atom(
            tcgen05.Cp4x32x128bOp(self.cta_group),
            self.sf_dtype,
        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 294-299 / 第 294-299 行

~~~~python
        tiled_copy_s2t_sfa = cute.nvgpu.tcgen05.make_s2t_copy(
            copy_atom_s2t, buffer_tmem_sfa_compact
        )
        tiled_copy_s2t_sfb = cute.nvgpu.tcgen05.make_s2t_copy(
            copy_atom_s2t, buffer_tmem_sfb_compact
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 301-301 / 第 301-301 行

~~~~python
        # Allocate SMEM buffer for C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 302-307 / 第 302-307 行

~~~~python
        buffer_smem_d = cute_ext.allocate(
            c_dtype,
            cute.AddressSpace.smem,
            smem_epi_staged_layout,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 309-309 / 第 309-309 行

~~~~python
        # Create the TMEM load atom
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 310-317 / 第 310-317 行

~~~~python
        copy_atom_t2r = sm100_utils.get_tmem_load_op(
            cta_tile_shape_mnk,
            d_layout,
            c_dtype,
            self.acc_dtype,
            epi_tile,
            self.use_2cta_instrs,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 319-319 / 第 319-319 行

~~~~python
        # Derive tiled_copy_t2r from the allocated TMEM buffer
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 320-323 / 第 320-323 行

~~~~python
        accumulators = cute.zipped_divide(buffer_tmem_accs, ((epi_tile), 1))
        acc_epi_div = accumulators[((None, None), 0), 0]
        tiled_copy_t2r = tcgen05.make_tmem_copy(copy_atom_t2r, acc_epi_div)
        thr_copy_t2r = tiled_copy_t2r.get_slice(tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 325-325 / 第 325-325 行

~~~~python
        # Derive per-thread RMEM layout for the T2R epilogue copy
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 326-329 / 第 326-329 行

~~~~python
        gC_tile_epi = cute.flat_divide(gC_tile, epi_tile)
        acc_epi_rmem_layout = cute_ext.make_t2r_rmem_layout(
            tiled_copy_t2r, gC_tile_epi, tidx
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 331-331 / 第 331-331 行

~~~~python
        # Allocate RMEM buffers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 332-343 / 第 332-343 行

~~~~python
        buffer_rmem_t2r = cute_ext.allocate(
            self.acc_dtype,
            cute.AddressSpace.rmem,
            acc_epi_rmem_layout,
            alignment=32,
        )
        buffer_rmem_r2s = cute_ext.allocate(
            c_dtype,
            cute.AddressSpace.rmem,
            acc_epi_rmem_layout,
            alignment=32,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 345-345 / 第 345-345 行

~~~~python
        # TMA -> UMMA
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 346-349 / 第 346-349 行

~~~~python
        mainloop_pipe = cute_ext.TMAToUMMAPipeline.create(
            num_stages=self.num_main_stages,
            mma_operation_type=cute_ext.OperationTypeEnum.SM100_MMA_1SM_SS,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 351-351 / 第 351-351 行

~~~~python
        # UMMA -> TMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 352-357 / 第 352-357 行

~~~~python
        acc_pipe = cute_ext.UMMAtoAsyncPipeline.create(
            num_stages=self.num_acc_stages,
            mma_operation_type=cute_ext.OperationTypeEnum.SM100_MMA_1SM_SS,
            consumer=cute_ext.OperationTypeEnum.SM100_COPY_T2R,
            consumer_arv_count=128,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 359-359 / 第 359-359 行

~~~~python
        # warp assignment: [0]-tma_store, [0-3]-epi, [4]-mma, [5]-tma_load
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 360-365 / 第 360-365 行

~~~~python
        tma_store_warp_id = 0
        mma_warp_id = 4
        tma_load_warp_id = 5
        is_tma_load_warp = warp_idx == tma_load_warp_id
        is_mma_warp = warp_idx == mma_warp_id
        is_epi_warp = warp_idx < 4
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 367-367 / 第 367-367 行

~~~~python
        # SMEM -> GMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 368-373 / 第 368-373 行

~~~~python
        tma_store_pipe = cute_ext.TMAStorePipeline(
            stages=self.tma_store_stages,
            arv_count=128,
            barrier_id=1,
            tma_warp_id=tma_store_warp_id,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 375-375 / 第 375-375 行

~~~~python
        k_tile_size = cute.size(gA, mode=[1, 1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 377-382 / 第 377-382 行

~~~~python
        if is_tma_load_warp:
            for k_tile_idx in cutlass.range(0, k_tile_size, 1, unroll=1):
                gA_k = gA_tile[None, None, k_tile_idx]
                gB_k = gB_tile[None, None, k_tile_idx]
                gSFA_k = gSFA_tile[None, None, k_tile_idx]
                gSFB_k = gSFB_tile[None, None, k_tile_idx]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 384-384 / 第 384-384 行

~~~~python
                # Scoped state management - pipeline object manages state internally
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 385-394 / 第 385-394 行

~~~~python
                (
                    producer_stage_token,
                    stage_idx,
                ) = mainloop_pipe.producer_acquire_and_get_stage()
                mbar = cute_ext.get_mbarrier(producer_stage_token)
                ## producer_body begin ##
                buffer_smem_a_sliced = buffer_smem_a[None, None, None, stage_idx]
                buffer_smem_b_sliced = buffer_smem_b[None, None, None, stage_idx]
                buffer_smem_sfa_sliced = buffer_smem_sfa[None, None, None, stage_idx]
                buffer_smem_sfb_sliced = buffer_smem_sfb[None, None, None, stage_idx]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 396-407 / 第 396-407 行

~~~~python
                a_cta_v_map = cute_ext.get_cta_v_map_ab(
                    mA, mma_tiler_mnk, tiled_mma, "A"
                )
                b_cta_v_map = cute_ext.get_cta_v_map_ab(
                    mB, mma_tiler_mnk, tiled_mma, "B"
                )
                sfa_cta_v_map = cute_ext.get_cta_v_map_ab(
                    sfa_tensor, mma_tiler_mnk, tiled_mma, "SFA"
                )
                sfb_cta_v_map = cute_ext.get_cta_v_map_ab(
                    sfb_tensor, mma_tiler_mnk, tiled_mma, "SFB"
                )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 409-434 / 第 409-434 行

~~~~python
                cute_ext.tma_load(
                    gA_k,
                    buffer_smem_a_sliced,
                    mbar,
                    cta_v_map=a_cta_v_map,
                )
                cute_ext.tma_load(
                    gB_k,
                    buffer_smem_b_sliced,
                    mbar,
                    cta_v_map=b_cta_v_map,
                )
                cute_ext.tma_load(
                    gSFA_k,
                    buffer_smem_sfa_sliced,
                    mbar,
                    cta_v_map=sfa_cta_v_map,
                )
                cute_ext.tma_load(
                    gSFB_k,
                    buffer_smem_sfb_sliced,
                    mbar,
                    cta_v_map=sfb_cta_v_map,
                )
                ## producer_body end ##
                mainloop_pipe.producer_commit_and_advance()
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 436-441 / 第 436-441 行

~~~~python
        if is_mma_warp:
            producer_stage_token, acc_stage_idx = (
                acc_pipe.producer_acquire_and_get_stage()
            )
            ## acc_producer_body begin ##
            accumulators_sliced = buffer_tmem_accs[None, None, None, acc_stage_idx]
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 443-444 / 第 443-444 行

~~~~python
            mma_atom = cute.make_mma_atom(tiled_mma.op)
            mma_atom.set(cute.nvgpu.tcgen05.Field.ACCUMULATE, False)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 446-447 / 第 446-447 行

~~~~python
            filtered_buffer_smem_sfa = cute.filter_zeros(buffer_smem_sfa)
            filtered_buffer_smem_sfb = cute.filter_zeros(buffer_smem_sfb)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 449-467 / 第 449-467 行

~~~~python
            for k_tile_idx in cutlass.range(0, k_tile_size, 1, unroll=1):
                # Scoped state management - pipeline object manages consumer state internally
                (
                    _,
                    mainloop_idx,
                ) = mainloop_pipe.consumer_wait_and_get_stage()
                ## tma_consumer_body begin ##
                buffer_smem_a_sliced_stage = buffer_smem_a[
                    (None, None, None, mainloop_idx)
                ]
                buffer_smem_b_sliced_stage = buffer_smem_b[
                    (None, None, None, mainloop_idx)
                ]
                filtered_buffer_smem_sfa_sliced_stage = filtered_buffer_smem_sfa[
                    (None, None, None, mainloop_idx)
                ]
                filtered_buffer_smem_sfb_sliced_stage = filtered_buffer_smem_sfb[
                    (None, None, None, mainloop_idx)
                ]
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 469-469 / 第 469-469 行

~~~~python
                # Copy SFA/SFB from SMEM to TMEM (UTCCP)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 470-481 / 第 470-481 行

~~~~python
                src_partitioned_SFA = cute_ext.partition(
                    filtered_buffer_smem_sfa_sliced_stage,
                    cute.Int32(0),
                    layout_tv=tiled_copy_s2t_sfa.layout_src_tv_tiled,
                    tiler=cute.core._pack_tile(tiled_copy_s2t_sfa.tiler_mn),
                )
                dst_partitioned_SFA = cute_ext.partition(
                    buffer_tmem_sfa_compact,
                    cute.Int32(0),
                    layout_tv=tiled_copy_s2t_sfa.layout_dst_tv_tiled,
                    tiler=cute.core._pack_tile(tiled_copy_s2t_sfa.tiler_mn),
                )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 483-485 / 第 483-485 行

~~~~python
                cute_ext.copy(
                    src_partitioned_SFA, dst_partitioned_SFA, copy_atom=copy_atom_s2t
                )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 487-498 / 第 487-498 行

~~~~python
                src_partitioned_SFB = cute_ext.partition(
                    filtered_buffer_smem_sfb_sliced_stage,
                    cute.Int32(0),
                    layout_tv=tiled_copy_s2t_sfb.layout_src_tv_tiled,
                    tiler=cute.core._pack_tile(tiled_copy_s2t_sfb.tiler_mn),
                )
                dst_partitioned_SFB = cute_ext.partition(
                    buffer_tmem_sfb_compact,
                    cute.Int32(0),
                    layout_tv=tiled_copy_s2t_sfb.layout_dst_tv_tiled,
                    tiler=cute.core._pack_tile(tiled_copy_s2t_sfb.tiler_mn),
                )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 500-502 / 第 500-502 行

~~~~python
                cute_ext.copy(
                    src_partitioned_SFB, dst_partitioned_SFB, copy_atom=copy_atom_s2t
                )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 504-510 / 第 504-510 行

~~~~python
                for k_block_idx in cutlass.range(mma_inst_tile_k, unroll_full=True):
                    buffer_smem_a_sliced = buffer_smem_a_sliced_stage[
                        None, None, k_block_idx
                    ]
                    buffer_smem_b_sliced = buffer_smem_b_sliced_stage[
                        None, None, k_block_idx
                    ]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 512-520 / 第 512-520 行

~~~~python
                    cute_ext.dot_block_scaled(
                        mma_atom,
                        cute.append_ones(buffer_smem_a_sliced, up_to_rank=3),
                        buffer_tmem_sfa[None, None, k_block_idx],
                        cute.append_ones(buffer_smem_b_sliced, up_to_rank=3),
                        buffer_tmem_sfb[None, None, k_block_idx],
                        accumulators_sliced,
                    )
                    mma_atom.set(cute.nvgpu.tcgen05.Field.ACCUMULATE, True)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 522-522 / 第 522-522 行

~~~~python
                ## tma_consumer_body end ##
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 523-523 / 第 523-523 行

~~~~python
                mainloop_pipe.consumer_release_and_advance()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 525-525 / 第 525-525 行

~~~~python
            ## acc_producer_body end ##
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 526-526 / 第 526-526 行

~~~~python
            acc_pipe.producer_commit_and_advance()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 528-536 / 第 528-536 行

~~~~python
        if is_epi_warp:
            _, acc_stage_idx = acc_pipe.consumer_wait_and_get_stage()
            ## acc_consume_body begin ##
            tmem_acc_stage = buffer_tmem_accs[
                (None, None), 0, 0, acc_stage_idx
            ]  # (MMA_M, MMA_N)
            # (EPI_TILE_M, EPI_TILE_N, EPI_REST_M, EPI_REST_N)
            # we have an implicit assumption that EPI_REST_M == 1
            tmem_acc_epi_stage = cute.flat_divide(tmem_acc_stage, epi_tile)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 538-545 / 第 538-545 行

~~~~python
            subtile_cnt = cute.size(tmem_acc_epi_stage.shape, mode=[3])  # EPI_REST_N
            for subtile_idx in range(subtile_cnt):
                # TMEM -> RMEM
                cute_ext.partition_and_copy(
                    thr_copy_t2r,
                    tmem_acc_epi_stage[(None, None, 0, subtile_idx)],
                    buffer_rmem_t2r,
                )
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 547-547 / 第 547-547 行

~~~~python
                # RMEM -> RMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 548-550 / 第 548-550 行

~~~~python
                buffer_rmem_r2s.store(
                    self.epilogue_op(buffer_rmem_t2r.load().to(c_dtype))
                )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 552-552 / 第 552-552 行

~~~~python
                # Acquire pipeline stage and synchronize before RMEM->SMEM copy
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 553-554 / 第 553-554 行

~~~~python
                tma_store_pipe.acquire_sync()
                tma_store_idx = tma_store_pipe.get_index()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 556-556 / 第 556-556 行

~~~~python
                # RMEM -> SMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 557-565 / 第 557-565 行

~~~~python
                tiled_copy_r2s = cute.make_tiled_copy_D(
                    cute.make_copy_atom(cute.nvgpu.CopyUniversalOp(), c_dtype),
                    tiled_copy_t2r,
                )
                cute_ext.partition_and_copy(
                    tiled_copy_r2s.get_slice(tidx),
                    buffer_rmem_r2s,
                    buffer_smem_d[None, None, tma_store_idx],
                )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 567-567 / 第 567-567 行

~~~~python
                # Fence SMEM writes and synchronize before TMA store
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 568-568 / 第 568-568 行

~~~~python
                tma_store_pipe.commit_sync()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 570-570 / 第 570-570 行

~~~~python
                # SMEM -> GMEM (only designated TMA store warp performs TMA store)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 571-577 / 第 571-577 行

~~~~python
                if warp_idx == tma_store_warp_id:
                    c_cta_v_map = cute_ext.get_cta_v_map_c(mC, epi_tile)
                    cute_ext.tma_store(
                        buffer_smem_d[None, None, tma_store_idx],
                        gC_tile_epi[(None, None, 0, subtile_idx)],
                        cta_v_map=c_cta_v_map,
                    )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 579-579 / 第 579-579 行

~~~~python
                # Release pipeline stage and advance
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 580-580 / 第 580-580 行

~~~~python
                tma_store_pipe.release_advance()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 582-583 / 第 582-583 行

~~~~python
            tma_store_pipe.tail()
            acc_pipe.consumer_release_and_advance()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 586-601 / 第 586-601 行

~~~~python
@cute.experimental.jit
def cvt_sf_MKL_to_M32x4xrm_K4xrk_L(
    sf_ref_tensor: cute.Tensor,
    sf_mma_tensor: cute.Tensor,
):
    """
    Convert scale factor tensor from MKL layout to mma specification
    M(32x4xrest_m)xK(4xrest_k)xL layout
    """
    # sf_mma_tensor has flatten shape (32, 4, rest_m, 4, rest_k, l)
    # group to ((32, 4, rest_m), (4, rest_k), l)
    sf_mma_tensor = cute.group_modes(sf_mma_tensor, 0, 3)
    sf_mma_tensor = cute.group_modes(sf_mma_tensor, 1, 3)
    for i in cutlass.range(cute.size(sf_ref_tensor)):
        mkl_coord = sf_ref_tensor.layout.get_hier_coord(i)
        sf_mma_tensor[mkl_coord] = sf_ref_tensor[mkl_coord]
~~~~

**EN**: Defines `cvt_sf_MKL_to_M32x4xrm_K4xrk_L`, grouping related logic behind a named Python callable so the example stays modular and reusable. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `cvt_sf_MKL_to_M32x4xrm_K4xrk_L`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 604-604 / 第 604-604 行

~~~~python
# TODO: add residual support (C)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 605-632 / 第 605-632 行

~~~~python
@dataclass
class BlockScaledGemmTestbed:
    """
    Testbed for block-scaled GEMM operations on Blackwell (SM100) architecture.

    This class manages test data and tensors for block-scaled matrix multiplication:
        D = (A * scale_factor_A) @ (B * scale_factor_B)

    The testbed maintains three representations of each tensor:
    1. Reference tensors (f32 on CPU) - used for reference computation and validation
    2. CUTE tensors - device tensors passed directly to CUDA kernels
    3. PyTorch tensors - mirrors of CUTE tensors for host-side operations

    Attributes:
        a_ref, b_ref: Reference input matrices (f32 format)
        sfa_ref, sfb_ref: Reference scale factors for A and B matrices (f32 format)
        d_ref: Reference output matrice (f32 format)

        a_tensor, b_tensor: CUTE tensors for input matrices (device)
        sfa_tensor, sfb_tensor: CUTE tensors for scale factors (device)
        d_tensor: CUTE tensors for output (device)

        a_torch, b_torch: PyTorch mirrors of A and B CUTE tensors
        sfa_torch, sfb_torch: PyTorch mirrors of scale factor CUTE tensors
        d_torch: PyTorch mirrors of D CUTE tensors

    The class provides:
    - Automatic tensor creation with proper layouts and alignment
~~~~

**EN**: Defines `BlockScaledGemmTestbed`, a reusable Python class that packages configuration and behavior for this example. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `BlockScaledGemmTestbed`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 633-647 / 第 633-647 行

~~~~python
    - Scale factor tensor generation with block-scaled MMA layout
    - Reference checking via einsum-based computation

    Example:
        testbed = BlockScaledGemmTestbed(
            MNKL=(128, 128, 64, 1),
            mma_dtypes=(cutlass.Float16, cutlass.Float16, cutlass.Float32),
            c_dtypes=(cutlass.Float16),
            sf_dtype=cutlass.Float16,
            sf_vec_size=32,
            a_major='m', b_major='n', d_major='m'
        )
        # ... run kernel with testbed.a_tensor, testbed.b_tensor, etc.
        testbed.reference_check()  # Validate results
    """
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 649-649 / 第 649-649 行

~~~~python
    import torch
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 651-652 / 第 651-652 行

~~~~python
    # Reference tensors (all are in f32 format for simplicity of
    # reference checks)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 653-656 / 第 653-656 行

~~~~python
    a_ref: torch.Tensor
    b_ref: torch.Tensor
    sfa_ref: torch.Tensor
    sfb_ref: torch.Tensor
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 658-658 / 第 658-658 行

~~~~python
    # CUTE tensors (to be passed to the device kernel)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 659-663 / 第 659-663 行

~~~~python
    a_tensor: cute.Tensor
    b_tensor: cute.Tensor
    sfa_tensor: cute.Tensor
    sfb_tensor: cute.Tensor
    d_tensor: cute.Tensor
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 665-667 / 第 665-667 行

~~~~python
    # PyTorch tensors (mirrors the CUTE tensors above); these tensors
    # can be used on the host, for example if certain trivial epilogue
    # needs to be performed.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 668-672 / 第 668-672 行

~~~~python
    a_torch: torch.Tensor
    b_torch: torch.Tensor
    sfa_torch: torch.Tensor
    sfb_torch: torch.Tensor
    d_torch: torch.Tensor
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 674-687 / 第 674-687 行

~~~~python
    def __init__(
        self,
        MNKL: Tuple[int, int, int, int],
        mma_dtypes: tuple[
            Type[cutlass.Numeric], Type[cutlass.Numeric], Type[cutlass.Numeric]
        ],
        c_dtype: Type[cutlass.Numeric],
        sf_dtype: Type[cutlass.Numeric],
        sf_vec_size: int,
        a_major: str,
        b_major: str,
        d_major: str,
    ):
        import cutlass.torch as cutlass_torch
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 689-689 / 第 689-689 行

~~~~python
        self.d_major = d_major
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 691-691 / 第 691-691 行

~~~~python
        # Problem size
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 692-692 / 第 692-692 行

~~~~python
        (M, N, K, L) = MNKL
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 694-694 / 第 694-694 行

~~~~python
        a_dtype, b_dtype, _ = mma_dtypes
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 696-698 / 第 696-698 行

~~~~python
        assert a_major in ("m", "k"), f"a_major must be 'm' or 'k', got {a_major}"
        assert b_major in ("n", "k"), f"b_major must be 'n' or 'k', got {b_major}"
        assert d_major in ("m", "n"), f"d_major must be 'm' or 'n', got {d_major}"
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 700-702 / 第 700-702 行

~~~~python
        self.a_ref = cutlass_torch.matrix(L, M, K, a_major == "m", cutlass.Float32)
        self.b_ref = cutlass_torch.matrix(L, N, K, b_major == "n", cutlass.Float32)
        self.d_temp = cutlass_torch.matrix(L, M, N, d_major == "m", cutlass.Float32)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 704-712 / 第 704-712 行

~~~~python
        self.a_tensor, self.a_torch = cutlass_torch.cute_tensor_like(
            self.a_ref, a_dtype, is_dynamic_layout=True, assumed_align=16
        )
        self.b_tensor, self.b_torch = cutlass_torch.cute_tensor_like(
            self.b_ref, b_dtype, is_dynamic_layout=True, assumed_align=16
        )
        self.d_tensor, self.d_torch = cutlass_torch.cute_tensor_like(
            self.d_temp, c_dtype, is_dynamic_layout=True, assumed_align=16
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 714-714 / 第 714-714 行

~~~~python
        # Mark tensor with element divisibility for 16B alignment
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 715-729 / 第 715-729 行

~~~~python
        self.a_tensor.mark_compact_shape_dynamic(
            mode=1 if a_major == "k" else 0,
            stride_order=(2, 0, 1) if a_major == "k" else (2, 1, 0),
            divisibility=32 if a_dtype == cutlass.Float4E2M1FN else 16,
        )
        self.b_tensor.mark_compact_shape_dynamic(
            mode=1 if b_major == "k" else 0,
            stride_order=(2, 0, 1) if b_major == "k" else (2, 1, 0),
            divisibility=32 if b_dtype == cutlass.Float4E2M1FN else 16,
        )
        self.d_tensor.mark_compact_shape_dynamic(
            mode=1 if d_major == "k" else 0,
            stride_order=(2, 0, 1) if d_major == "n" else (2, 1, 0),
            divisibility=32 if c_dtype == cutlass.Float4E2M1FN else 16,
        )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 731-736 / 第 731-736 行

~~~~python
        self.sfa_ref, self.sfa_tensor, self.sfa_torch = self.create_scale_factor_tensor(
            L, M, K, sf_vec_size, sf_dtype
        )
        self.sfb_ref, self.sfb_tensor, self.sfb_torch = self.create_scale_factor_tensor(
            L, N, K, sf_vec_size, sf_dtype
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 738-738 / 第 738-738 行

~~~~python
    # Create scale factor tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 739-742 / 第 739-742 行

~~~~python
    @staticmethod
    def create_scale_factor_tensor(l, mn, k, sf_vec_size, dtype):
        import torch
        import cutlass.torch as cutlass_torch
~~~~

**EN**: Defines `create_scale_factor_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_scale_factor_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 744-745 / 第 744-745 行

~~~~python
        def ceil_div(a, b):
            return (a + b - 1) // b
~~~~

**EN**: Defines `ceil_div`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `ceil_div`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 747-749 / 第 747-749 行

~~~~python
        sf_k = ceil_div(k, sf_vec_size)
        ref_shape = (l, mn, sf_k)
        ref_permute_order = (1, 2, 0)  # MKL
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 751-761 / 第 751-761 行

~~~~python
        atom_m = (32, 4)
        atom_k = 4
        mma_shape = (
            l,
            ceil_div(mn, atom_m[0] * atom_m[1]),
            ceil_div(sf_k, atom_k),
            atom_m[0],
            atom_m[1],
            atom_k,
        )
        mma_permute_order = (3, 4, 1, 5, 2, 0)  # M(32x4xrest_m)xK(4xrest_k)xL
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 763-763 / 第 763-763 行

~~~~python
        # Create f32 ref torch tensor (cpu)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 764-773 / 第 764-773 行

~~~~python
        ref_f32_torch_tensor_cpu = cutlass_torch.create_and_permute_torch_tensor(
            ref_shape,
            torch.float32,
            permute_order=ref_permute_order,
            init_type=cutlass_torch.TensorInitType.RANDOM,
            init_config=cutlass_torch.RandomInitConfig(
                min_val=1,
                max_val=3,
            ),
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 775-775 / 第 775-775 行

~~~~python
        # Create f32 cute torch tensor (cpu)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 776-785 / 第 776-785 行

~~~~python
        cute_f32_torch_tensor_cpu = cutlass_torch.create_and_permute_torch_tensor(
            mma_shape,
            torch.float32,
            permute_order=mma_permute_order,
            init_type=cutlass_torch.TensorInitType.RANDOM,
            init_config=cutlass_torch.RandomInitConfig(
                min_val=0,
                max_val=1,
            ),
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 787-787 / 第 787-787 行

~~~~python
        # convert ref f32 tensor to cute f32 tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 788-792 / 第 788-792 行

~~~~python
        cvt_sf_MKL_to_M32x4xrm_K4xrk_L(
            from_dlpack(ref_f32_torch_tensor_cpu),
            from_dlpack(cute_f32_torch_tensor_cpu),
        )
        cute_f32_torch_tensor = cute_f32_torch_tensor_cpu.cuda()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 794-794 / 第 794-794 行

~~~~python
        # reshape makes memory contiguous
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 795-803 / 第 795-803 行

~~~~python
        ref_f32_torch_tensor_cpu = (
            ref_f32_torch_tensor_cpu.permute(2, 0, 1)
            .unsqueeze(-1)
            .expand(l, mn, sf_k, sf_vec_size)
            .reshape(l, mn, sf_k * sf_vec_size)
            .permute(*ref_permute_order)
        )
        # prune to mkl for reference check.
        ref_f32_torch_tensor_cpu = ref_f32_torch_tensor_cpu[:, :k, :]
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 805-805 / 第 805-805 行

~~~~python
        # Create dtype cute torch tensor (cpu)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 806-811 / 第 806-811 行

~~~~python
        cute_tensor, cute_torch_tensor = cutlass_torch.cute_tensor_like(
            cute_f32_torch_tensor_cpu,
            dtype,
            is_dynamic_layout=True,
            assumed_align=16,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 813-813 / 第 813-813 行

~~~~python
        # Convert f32 cute tensor to dtype cute tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 814-820 / 第 814-820 行

~~~~python
        cute_tensor = cutlass_torch.convert_cute_tensor(
            cute_f32_torch_tensor,
            cute_tensor,
            dtype,
            is_dynamic_layout=True,
        )
        return ref_f32_torch_tensor_cpu, cute_tensor, cute_torch_tensor
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 822-823 / 第 822-823 行

~~~~python
    # Transfers results back to CPU and uses PyTorch's methods to do
    # reference checks
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 824-825 / 第 824-825 行

~~~~python
    def reference_check(self):
        import torch
~~~~

**EN**: Defines `reference_check`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `reference_check`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 827-828 / 第 827-828 行

~~~~python
        # Compute reference result, simulate block-scaled GEMV via 2 FFMA
        # based elementwise multiplication and 1 FFMA based matmul computations
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 829-831 / 第 829-831 行

~~~~python
        res_a = torch.einsum("mkl,mkl->mkl", self.a_ref, self.sfa_ref)
        res_b = torch.einsum("nkl,nkl->nkl", self.b_ref, self.sfb_ref)
        ref_output = torch.einsum("mkl,nkl->mnl", res_a, res_b)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 833-833 / 第 833-833 行

~~~~python
        # Convert d back to f32 for comparison.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 834-840 / 第 834-840 行

~~~~python
        d_epi_device = self.d_temp.cuda()
        cute.testing.convert(
            self.d_tensor,
            from_dlpack(d_epi_device, assumed_align=16).mark_layout_dynamic(
                leading_dim=(1 if self.d_major == "n" else 0)
            ),
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 842-842 / 第 842-842 行

~~~~python
        # abs(actual - expected) <= atol + rtol * abs(expected)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 843-846 / 第 843-846 行

~~~~python
        torch.testing.assert_close(
            d_epi_device.cpu(), ref_output, atol=1e-01, rtol=1e-02
        )
        print("Reference check finished.")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 849-876 / 第 849-876 行

~~~~python
def run(
    mnkl: Tuple[int, int, int, int],
    mma_inst_mn: Tuple[int, int],
    cluster_shape_mn: Tuple[int, int],
    ab_dtype: Type[cutlass.Numeric],
    sf_dtype: Type[cutlass.Numeric],
    sf_vec_size: int,
    d_dtype: Type[cutlass.Numeric],
    acc_dtype: Type[cutlass.Numeric],
    a_major: str,
    b_major: str,
    d_major: str,
):
    """Execute a batched block scaled dense GEMM operation on Blackwell architecture.

    This function prepares input tensors, configures and launches the GEMM kernel,
    and performs reference validation.

    :param mnkl: Problem size (M, N, K, L)
    :type mnkl: Tuple[int, int, int, int]
    :param mma_inst_mn: MMA instruction shape.
    :type mma_inst_mn: Tuple[int, int]
    :param cluster_shape_mn: Cluster shape.
    :type cluster_shape_mn: Tuple[int, int]
    :param ab_dtype: Data type for input tensors A and B
    :type ab_dtype: Type[Numeric]
    :param sf_dtype: Data type for scale factors (SFA/SFB)
    :type sf_dtype: Type[Numeric]
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 877-898 / 第 877-898 行

~~~~python
    :param sf_vec_size: Vector size for the scale factor
    :type sf_vec_size: int
    :param c_dtype: Data type for output tensor D
    :type c_dtype: Type[Numeric]
    :param acc_dtype: Accumulator data type (precision)
    :type acc_dtype: Type[Numeric]
    :param a_major: Major-ness of A tensor (m or k)
    :type a_major: str
    :param b_major: Major-ness of B tensor (n or k)
    :type b_major: str
    :param d_major: Major-ness of D tensor (m or n)
    :type d_major: str
    """
    print("Running Blackwell Dense Block Scaled GEMM test with:")
    print(f"mnkl: {mnkl}")
    print(f"A: {ab_dtype}, B: {ab_dtype}, D: {d_dtype}, Acc dtype: {acc_dtype}")
    print(f"Block scaled MMA with SF: {sf_dtype}, vector size: {sf_vec_size}")
    print(f"Matrix majors - A: {a_major}-major, B: {b_major}-major, D: {d_major}-major")
    print(
        f"Mma Tiler (M, N): {mma_inst_mn}, Cluster Shape: {cluster_shape_mn[0]}x{cluster_shape_mn[1]}x1"
    )
    import torch
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 900-900 / 第 900-900 行

~~~~python
    # TODO: add can_implement to exclude unsupported/un-implemented test cases
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 901-908 / 第 901-908 行

~~~~python
    if cluster_shape_mn != (1, 1):
        raise RuntimeError("Only 1x1x1 cluster shapes are supported right now.")
    if mma_inst_mn != (128, 128):
        raise RuntimeError("MMA instruction shape not supported yet.")
    if ab_dtype not in (cutlass.Float8E4M3FN, cutlass.Float8E5M2):
        raise RuntimeError("Input data type not supported.")
    if sf_dtype not in (cutlass.Float8E8M0FNU, cutlass.Float8E4M3FN):
        raise RuntimeError("Scale factor data type not supported.")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 910-911 / 第 910-911 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("A GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 913-913 / 第 913-913 行

~~~~python
    # Manual seed
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 914-914 / 第 914-914 行

~~~~python
    torch.manual_seed(111)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 916-916 / 第 916-916 行

~~~~python
    # Create tensors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 917-926 / 第 917-926 行

~~~~python
    tb = BlockScaledGemmTestbed(
        mnkl,
        (ab_dtype, ab_dtype, acc_dtype),
        d_dtype,
        sf_dtype,
        sf_vec_size,
        a_major,
        b_major,
        d_major,
    )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 928-928 / 第 928-928 行

~~~~python
    # JIT-Compile the device kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 929-934 / 第 929-934 行

~~~~python
    block_scaled_gemm = BlockScaledDenseGemmKernel(
        mma_inst_mn=mma_inst_mn,
        mma_dtype=(ab_dtype, acc_dtype),
        sf_dtype=sf_dtype,
        sf_vec_size=sf_vec_size,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 936-943 / 第 936-943 行

~~~~python
    compiled_kernel = cute.experimental.compile(
        block_scaled_gemm,
        tb.a_tensor,
        tb.sfa_tensor,
        tb.b_tensor,
        tb.sfb_tensor,
        tb.d_tensor,
    )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 945-945 / 第 945-945 行

~~~~python
    # Launch the device kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 946-952 / 第 946-952 行

~~~~python
    compiled_kernel(
        tb.a_tensor,
        tb.sfa_tensor,
        tb.b_tensor,
        tb.sfb_tensor,
        tb.d_tensor,
    )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 954-954 / 第 954-954 行

~~~~python
    tb.reference_check()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 957-957 / 第 957-957 行

~~~~python
if __name__ == "__main__":
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。

### Lines 959-965 / 第 959-965 行

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

### Lines 967-969 / 第 967-969 行

~~~~python
    parser = argparse.ArgumentParser(
        description="Example of Sm100 Dense BlockScaled GEMM."
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 971-996 / 第 971-996 行

~~~~python
    parser.add_argument(
        "--mnkl",
        type=parse_comma_separated_ints,
        default=(512, 256, 256, 1),
        help="mnkl dimensions (comma-separated)",
    )
    parser.add_argument(
        "--mma_inst_mn",
        type=parse_comma_separated_ints,
        default=(128, 128),
        help="Mma instruction shape (comma-separated)",
    )
    parser.add_argument(
        "--cluster_shape_mn",
        type=parse_comma_separated_ints,
        default=(1, 1),
        help="Cluster shape (comma-separated)",
    )
    parser.add_argument("--ab_dtype", type=cutlass.dtype, default=cutlass.Float8E4M3FN)
    parser.add_argument("--sf_dtype", type=cutlass.dtype, default=cutlass.Float8E8M0FNU)
    parser.add_argument("--sf_vec_size", type=int, default=32)
    parser.add_argument("--acc_dtype", type=cutlass.dtype, default=cutlass.Float32)
    parser.add_argument("--d_dtype", type=cutlass.dtype, default=cutlass.Float16)
    parser.add_argument("--a_major", choices=["k", "m"], type=str, default="k")
    parser.add_argument("--b_major", choices=["k", "n"], type=str, default="k")
    parser.add_argument("--d_major", choices=["n", "m"], type=str, default="n")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 998-998 / 第 998-998 行

~~~~python
    args = parser.parse_args()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 1000-1001 / 第 1000-1001 行

~~~~python
    if len(args.mnkl) != 4:
        parser.error("--mnkl must contain exactly 4 values")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1003-1004 / 第 1003-1004 行

~~~~python
    if len(args.mma_inst_mn) != 2:
        parser.error("--mma_inst_mn must contain exactly 2 values")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1006-1007 / 第 1006-1007 行

~~~~python
    if len(args.cluster_shape_mn) != 2:
        parser.error("--cluster_shape_mn must contain exactly 2 values")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1009-1021 / 第 1009-1021 行

~~~~python
    run(
        args.mnkl,
        args.mma_inst_mn,
        args.cluster_shape_mn,
        args.ab_dtype,
        args.sf_dtype,
        args.sf_vec_size,
        args.d_dtype,
        args.acc_dtype,
        args.a_major,
        args.b_major,
        args.d_major,
    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `typing.Type` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `dataclasses.dataclass` — used by this example / 供该示例使用
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.experimental` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `cutlass.cute.nvgpu.cpasync` — provides asynchronous copy and TMA primitives / 提供异步拷贝与 TMA 原语
- `cutlass.cute.nvgpu.tcgen05` — provides Tensor Core/TMEM helpers for Blackwell-style kernels / 提供面向 Blackwell 风格内核的 Tensor Core/TMEM 辅助工具
- `cutlass.utils.blackwell_helpers` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.blockscaled_layout` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
