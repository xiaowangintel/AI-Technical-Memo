# dense_gemm_ptr_array.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute_ext/blackwell/dense_gemm_ptr_array.py`  
**Purpose / 用途**: Example module for dense gemm ptr array. / 这是一个关于 dense gemm ptr array 的 CuTeDSL 示例模块。

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

### Lines 29-44 / 第 29-44 行

~~~~python
import argparse

import torch
from typing import Type, Tuple, List

import cutlass
from cutlass.cute import experimental as cute_ext
from cutlass.base_dsl.typing import Numeric
from cutlass import cute as cute
from cutlass import utils
from cutlass import torch as cutlass_torch
from cutlass.cute.experimental.host_runtime import QueryDeviceWorkspaceFunc
from cutlass.cute.runtime import from_dlpack
import cutlass.utils.blackwell_helpers as sm100_utils

import cutlass.cute.testing as testing
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 47-74 / 第 47-74 行

~~~~python
class DenseGemmPtrArrayKernel:
    def __init__(
        self,
        mn_tiler: tuple[int, int],
        mma_dtype: tuple[Type[Numeric], Type[Numeric], Type[Numeric]],
        tmem_output_dtype: Type[Numeric],
        batch_count: int,  # Number of batches, each batch will have its own pointer for the matrix
        A_shape: tuple,  # Shape of the matrix A
        A_stride: tuple,  # Stride of the matrix A
        B_shape: tuple,  # Shape of the matrix B
        B_stride: tuple,  # Stride of the matrix B
        D_shape: tuple,  # Shape of the matrix D
        D_stride: tuple,  # Stride of the matrix D
        epilogue_op=lambda x: x,
    ):
        self.mn_tiler = mn_tiler
        self.ab_dtype, self.acc_dtype, self.d_dtype = mma_dtype
        self.tmem_output_dtype = tmem_output_dtype
        self.use_2cta_instrs = False
        self.TMA_STORE_STAGE = 4
        self.epilogue_op = epilogue_op
        self.batch_count = batch_count
        self.A_shape = A_shape
        self.A_stride = A_stride
        self.B_shape = B_shape
        self.B_stride = B_stride
        self.D_shape = D_shape
        self.D_stride = D_stride
~~~~

**EN**: Defines `DenseGemmPtrArrayKernel`, a reusable Python class that packages configuration and behavior for this example. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `DenseGemmPtrArrayKernel`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 76-80 / 第 76-80 行

~~~~python
    """
    Helper function to convert an int64 to a cute.ptr of a certain type.
    The cute.ptr is always located in Gmem.
    This is used to load the pointers for A/B/D from the Ptr array.
    """
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 82-90 / 第 82-90 行

~~~~python
    @cute.experimental.jit
    def _get_pointer(self, address_as_int, cute_type):
        cute_ptr = cute.make_ptr(
            cute_type,
            address_as_int,
            mem_space=cute.AddressSpace.gmem,
            assumed_align=16,
        )
        return cute_ptr
~~~~

**EN**: Defines `_get_pointer`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_get_pointer`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 92-109 / 第 92-109 行

~~~~python
    @cute.experimental.jit
    def __call__(
        self, mA_tensor: cute.Tensor, mB_tensor: cute.Tensor, mD_tensor: cute.Tensor
    ):
        # Get the pointer to the first batch of D
        d_ptr = self._get_pointer(mD_tensor[0], self.d_dtype)
        d_ptr_base_tensor = cute.make_tensor(
            d_ptr, layout=cute.make_layout(self.D_shape, stride=self.D_stride)
        )
        tile_mn = cute.core._pack_shape((*self.mn_tiler, 1))
        div = cute.tiled_divide(d_ptr_base_tensor, tile_mn)
        grid = (div.shape[1], div.shape[2], div.shape[3])
        self.kernel(mA_tensor, mB_tensor, mD_tensor).launch(
            grid=grid,
            block=(192, 1, 1),
            cluster=(1, 1, 1),
            smem=cute.Int64(utils.get_smem_capacity_in_bytes("sm_100")),
        )
~~~~

**EN**: Defines `__call__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `__call__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 111-120 / 第 111-120 行

~~~~python
    @cute.experimental.kernel
    def kernel(
        self,
        mA_tensor: cute.Tensor,
        mB_tensor: cute.Tensor,
        mD_tensor: cute.Tensor,
    ):
        # Get pointers for the first batch to perform shape and stage calculations
        A_0_ptr = self._get_pointer(mA_tensor[0], self.ab_dtype)
        B_0_ptr = self._get_pointer(mB_tensor[0], self.ab_dtype)
~~~~

**EN**: Defines `kernel`, grouping related logic behind a named Python callable so the example stays modular and reusable.
**CN**: 定义 `kernel`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。

### Lines 122-124 / 第 122-124 行

~~~~python
        mA = cute.make_tensor(
            A_0_ptr, layout=cute.make_layout(self.A_shape, stride=self.A_stride)
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 126-128 / 第 126-128 行

~~~~python
        mB = cute.make_tensor(
            B_0_ptr, layout=cute.make_layout(self.B_shape, stride=self.B_stride)
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 130-138 / 第 130-138 行

~~~~python
        tiled_mma = sm100_utils.make_trivial_tiled_mma(
            self.ab_dtype,
            self.ab_dtype,
            utils.LayoutEnum.from_tensor(mA).mma_major_mode(),
            utils.LayoutEnum.from_tensor(mB).mma_major_mode(),
            self.acc_dtype,
            cute.nvgpu.tcgen05.CtaGroup.ONE,
            self.mn_tiler,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 140-146 / 第 140-146 行

~~~~python
        mma_inst_shape_k = cute.size(tiled_mma.shape_mnk, mode=[2])
        mma_inst_tile_k = 4
        mnk_tiler = (
            self.mn_tiler[0],
            self.mn_tiler[1],
            mma_inst_shape_k * mma_inst_tile_k,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 148-149 / 第 148-149 行

~~~~python
        tiler_mk = (mnk_tiler[0], mnk_tiler[2])
        tiler_nk = (mnk_tiler[1], mnk_tiler[2])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 151-152 / 第 151-152 行

~~~~python
        gA = cute.zipped_divide(mA, tiler_mk)
        gB = cute.zipped_divide(mB, tiler_nk)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 154-155 / 第 154-155 行

~~~~python
        mainloop_stage = 2
        acc_stage = 2
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 157-157 / 第 157-157 行

~~~~python
        cta_m, cta_n, cta_l = cute.arch.block_idx()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 159-160 / 第 159-160 行

~~~~python
        gA_tile = gA[(None, None), (cta_m, None, cta_l)]
        gB_tile = gB[(None, None), (cta_n, None, cta_l)]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 162-162 / 第 162-162 行

~~~~python
        # Compute A/B/C shared memory layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 163-174 / 第 163-174 行

~~~~python
        a_smem_layout_staged = sm100_utils.make_smem_layout_a(
            tiled_mma,
            mnk_tiler,
            self.ab_dtype,
            mainloop_stage,
        )
        b_smem_layout_staged = sm100_utils.make_smem_layout_b(
            tiled_mma,
            mnk_tiler,
            self.ab_dtype,
            mainloop_stage,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 176-178 / 第 176-178 行

~~~~python
        cta_tile_shape_mnk = cute.shape_div(
            mnk_tiler, (cute.size(tiled_mma.thr_id.shape), 1, 1)
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 180-180 / 第 180-180 行

~~~~python
        # UMMA ACC TMEM Layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 181-181 / 第 181-181 行

~~~~python
        tmem_layout = cute_ext.make_tmem_layout_acc(tiled_mma, mnk_tiler, acc_stage)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 183-183 / 第 183-183 行

~~~~python
        # Allocate UMMA Buffers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 184-189 / 第 184-189 行

~~~~python
        bufferA = cute_ext.allocate(
            self.ab_dtype,
            cute.AddressSpace.smem,
            a_smem_layout_staged,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 191-196 / 第 191-196 行

~~~~python
        bufferB = cute_ext.allocate(
            self.ab_dtype,
            cute.AddressSpace.smem,
            b_smem_layout_staged,
            alignment=1024,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 198-203 / 第 198-203 行

~~~~python
        bufferAcc = cute_ext.allocate(
            self.acc_dtype,
            cute.AddressSpace.tmem,
            tmem_layout,
            alignment=16,
        )
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 205-205 / 第 205-205 行

~~~~python
        # TMA -> UMMA
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 206-209 / 第 206-209 行

~~~~python
        mainloop_pipe = cute_ext.TMAToUMMAPipeline.create(
            num_stages=mainloop_stage,
            mma_operation_type=cute_ext.OperationTypeEnum.SM100_MMA_1SM_SS,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 211-211 / 第 211-211 行

~~~~python
        # UMMA -> TMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 212-217 / 第 212-217 行

~~~~python
        acc_pipe = cute_ext.UMMAtoAsyncPipeline.create(
            num_stages=acc_stage,
            mma_operation_type=cute_ext.OperationTypeEnum.SM100_MMA_1SM_SS,
            consumer=cute_ext.OperationTypeEnum.SM100_COPY_T2R,
            consumer_arv_count=128,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 219-227 / 第 219-227 行

~~~~python
        warp_idx = cute.arch.warp_idx()
        warp_idx = cute.arch.make_warp_uniform(warp_idx)
        # warp assignment: [0]-tma_store, [0-3]-epi, [4]-mma, [5]-tma_load
        tma_store_warp_id = 0
        mma_warp_id = 4
        tma_load_warp_id = 5
        is_tma_thr = warp_idx == tma_load_warp_id
        is_mma_thr = warp_idx == mma_warp_id
        is_epi_thr = warp_idx < 4
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 229-229 / 第 229-229 行

~~~~python
        # SMEM -> GMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 230-235 / 第 230-235 行

~~~~python
        tma_store_pipe = cute_ext.TMAStorePipeline(
            stages=self.TMA_STORE_STAGE,
            arv_count=128,
            barrier_id=1,
            tma_warp_id=tma_store_warp_id,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 237-237 / 第 237-237 行

~~~~python
        k_tile_size = cute.size(gA, mode=[1, 1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 239-243 / 第 239-243 行

~~~~python
        # Outer loop over batches and perform GEMM for each batch as usual
        # This is a dynamic for loop that lowers to an scf.for
        # Note that the tensor loading is done in the if `thread` warp specialized
        # sections. This is essential to ensure proper synchronization of tma loads
        # and tma updates across batches.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 244-248 / 第 244-248 行

~~~~python
        for batch_idx in range(0, self.batch_count):
            # Load pointers for the current batch
            ptr_A = self._get_pointer(mA_tensor[batch_idx], self.ab_dtype)
            ptr_B = self._get_pointer(mB_tensor[batch_idx], self.ab_dtype)
            ptr_D = self._get_pointer(mD_tensor[batch_idx], self.d_dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 250-251 / 第 250-251 行

~~~~python
            gALayout = cute.zipped_divide(mA, tiler_mk)
            k_tile_size = cute.size(gALayout, mode=[1, 1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 253-266 / 第 253-266 行

~~~~python
            if is_tma_thr:
                mA = cute.make_tensor(
                    ptr_A, layout=cute.make_layout(self.A_shape, stride=self.A_stride)
                )
                mB = cute.make_tensor(
                    ptr_B, layout=cute.make_layout(self.B_shape, stride=self.B_stride)
                )
                gA = cute.zipped_divide(mA, tiler_mk)
                gB = cute.zipped_divide(mB, tiler_nk)
                gA_tile = gA[(None, None), (cta_m, None, cta_l)]
                gB_tile = gB[(None, None), (cta_n, None, cta_l)]
                for k in cutlass.range(0, k_tile_size, 1, unroll=1):
                    gA_k = gA_tile[None, None, k]
                    gB_k = gB_tile[None, None, k]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 268-268 / 第 268-268 行

~~~~python
                    # Scoped state management - pipeline object manages state internally
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 269-296 / 第 269-296 行

~~~~python
                    (
                        producer_stage_token,
                        idx,
                    ) = mainloop_pipe.producer_acquire_and_get_stage()
                    mbar = cute_ext.get_mbarrier(producer_stage_token)
                    ## producer_body begin ##
                    bufferA_sliced = bufferA[None, None, None, idx]
                    bufferB_sliced = bufferB[None, None, None, idx]
                    a_cta_v_map = cute_ext.get_cta_v_map_ab(
                        mA, mnk_tiler, tiled_mma, "A"
                    )
                    b_cta_v_map = cute_ext.get_cta_v_map_ab(
                        mB, mnk_tiler, tiled_mma, "B"
                    )
                    cute_ext.tma_load(
                        gA_k,
                        bufferA_sliced,
                        mbar,
                        cta_v_map=a_cta_v_map,
                    )
                    cute_ext.tma_load(
                        gB_k,
                        bufferB_sliced,
                        mbar,
                        cta_v_map=b_cta_v_map,
                    )
                    ## producer_body end ##
                    mainloop_pipe.producer_commit_and_advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 298-298 / 第 298-298 行

~~~~python
            # MMA section remains same as a regular GEMM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 299-301 / 第 299-301 行

~~~~python
            if is_mma_thr:
                producer_stage_token, idx = acc_pipe.producer_acquire_and_get_stage()
                accumulators_sliced = bufferAcc[None, None, None, idx]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 303-314 / 第 303-314 行

~~~~python
                (updated_a_pipe, _updated_b_pipe) = cute_ext.mainloop_mma(
                    tiled_mma,
                    bufferA,
                    bufferB,
                    accumulators_sliced,
                    0,
                    k_tile_size,
                    mma_inst_tile_k,
                    mainloop_pipe,
                    mainloop_pipe,
                )
                mainloop_pipe = updated_a_pipe
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 316-316 / 第 316-316 行

~~~~python
                acc_pipe.producer_commit_and_advance()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 318-322 / 第 318-322 行

~~~~python
            if is_epi_thr:
                # Load the D tensor in the warp specialized section
                mD = cute.make_tensor(
                    ptr_D, layout=cute.make_layout(self.D_shape, stride=self.D_stride)
                )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 324-326 / 第 324-326 行

~~~~python
                _, idx = acc_pipe.consumer_wait_and_get_stage()
                accumulators_sliced = bufferAcc[(None, None), 0, 0, idx]
                cta_d_tile_coord = (cta_m, cta_n, cta_l)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 328-337 / 第 328-337 行

~~~~python
                tma_store_pipe = cute_ext.epilogue_tma_store(
                    cta_tile_shape_mnk,
                    self.use_2cta_instrs,
                    accumulators_sliced,
                    mD,
                    cta_d_tile_coord,
                    tma_store_pipe,
                    tma_store_warp_id,
                    self.epilogue_op,
                )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 339-339 / 第 339-339 行

~~~~python
                acc_pipe.consumer_release_and_advance()
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 342-343 / 第 342-343 行

~~~~python
def create_tensors(l, m, n, k, a_major, b_major, d_major, ab_dtype, d_dtype):
    torch.manual_seed(1111)
~~~~

**EN**: Defines `create_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `create_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 345-347 / 第 345-347 行

~~~~python
    a_torch_cpu = cutlass_torch.matrix(l, m, k, a_major == "m", ab_dtype)
    b_torch_cpu = cutlass_torch.matrix(l, n, k, b_major == "n", ab_dtype)
    d_torch_cpu = cutlass_torch.matrix(l, m, n, d_major == "m", d_dtype)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 349-357 / 第 349-357 行

~~~~python
    a_tensor, a_torch_gpu = cutlass_torch.cute_tensor_like(
        a_torch_cpu, ab_dtype, is_dynamic_layout=True, assumed_align=16
    )
    b_tensor, b_torch_gpu = cutlass_torch.cute_tensor_like(
        b_torch_cpu, ab_dtype, is_dynamic_layout=True, assumed_align=16
    )
    d_tensor, d_torch_gpu = cutlass_torch.cute_tensor_like(
        d_torch_cpu, d_dtype, is_dynamic_layout=True, assumed_align=16
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 359-369 / 第 359-369 行

~~~~python
    return (
        a_tensor,
        b_tensor,
        d_tensor,
        a_torch_cpu,
        b_torch_cpu,
        d_torch_cpu,
        a_torch_gpu,
        b_torch_gpu,
        d_torch_gpu,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 372-372 / 第 372-372 行

~~~~python
# Helper creates a cute.Tensor from a List of device pointers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 373-386 / 第 373-386 行

~~~~python
def make_tensor_of_ptrs(torch_tensor_array: List):
    tensor_of_ptrs_torch = torch.tensor(
        [t.data_ptr() for t in torch_tensor_array],
        dtype=torch.int64,
        device="cuda",
        requires_grad=False,
    )
    tensor_of_ptrs_cute, backing_torch_tensor = cutlass_torch.cute_tensor_like(
        tensor_of_ptrs_torch,
        cutlass.Int64,
        is_dynamic_layout=False,
        assumed_align=16,
    )
    return tensor_of_ptrs_cute, backing_torch_tensor
~~~~

**EN**: Defines `make_tensor_of_ptrs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `make_tensor_of_ptrs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 389-399 / 第 389-399 行

~~~~python
def create_tensors_for_ptr_array(
    l, m, n, k, a_major, b_major, d_major, ab_dtype, d_dtype
):
    # Store torch gpu pointers
    As_torch_gpu = []
    Bs_torch_gpu = []
    Ds_torch_gpu = []
    # Store cute tensors
    A_cutes = []
    B_cutes = []
    D_cutes = []
~~~~

**EN**: Defines `create_tensors_for_ptr_array`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `create_tensors_for_ptr_array`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 401-402 / 第 401-402 行

~~~~python
    for batch_idx in range(l):
        torch.manual_seed(111 + batch_idx)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 404-424 / 第 404-424 行

~~~~python
        (
            A_tensor,
            B_tensor,
            D_tensor,
            A_torch_cpu,
            B_torch_cpu,
            D_torch_cpu,
            A_torch_gpu,
            B_torch_gpu,
            D_torch_gpu,
        ) = create_tensors(
            1,  # outer loop creates a new tensor for each batch
            m,
            n,
            k,
            a_major,
            b_major,
            d_major,
            ab_dtype,
            d_dtype,
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 426-431 / 第 426-431 行

~~~~python
        A_cutes.append(A_tensor)
        B_cutes.append(B_tensor)
        D_cutes.append(D_tensor)
        As_torch_gpu.append(A_torch_gpu)
        Bs_torch_gpu.append(B_torch_gpu)
        Ds_torch_gpu.append(D_torch_gpu)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 433-433 / 第 433-433 行

~~~~python
    # Create cute tensors of pointers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 434-436 / 第 434-436 行

~~~~python
    a_tensor, a_backing_torch_tensor = make_tensor_of_ptrs(As_torch_gpu)
    b_tensor, b_backing_torch_tensor = make_tensor_of_ptrs(Bs_torch_gpu)
    d_tensor, d_backing_torch_tensor = make_tensor_of_ptrs(Ds_torch_gpu)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 438-451 / 第 438-451 行

~~~~python
    return (
        a_tensor,
        b_tensor,
        d_tensor,
        a_backing_torch_tensor,
        b_backing_torch_tensor,
        d_backing_torch_tensor,
        A_cutes,
        B_cutes,
        D_cutes,
        As_torch_gpu,
        Bs_torch_gpu,
        Ds_torch_gpu,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 454-455 / 第 454-455 行

~~~~python
def compare(a_torch_cpu, b_torch_cpu, d_torch_gpu, d_dtype, tolerance):
    ref = torch.einsum("mkl,nkl->mnl", a_torch_cpu, b_torch_cpu)
~~~~

**EN**: Defines `compare`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `compare`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 457-463 / 第 457-463 行

~~~~python
    _, ref_torch_gpu = cutlass_torch.cute_tensor_like(
        ref, d_dtype, is_dynamic_layout=True, assumed_align=16
    )
    ref_result = ref_torch_gpu.cpu()
    torch.testing.assert_close(
        d_torch_gpu.cpu(), ref_result, atol=tolerance, rtol=1e-05
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 466-493 / 第 466-493 行

~~~~python
def run(
    mnkl: Tuple[int, int, int, int],
    mma_tiler_mn: Tuple[int, int],
    cluster_shape_mn: Tuple[int, int],
    ab_dtype: Type[Numeric],
    c_dtype: Type[Numeric],
    acc_dtype: Type[Numeric],
    a_major: str,
    b_major: str,
    c_major: str,
    warmup_iterations: int = 0,
    iterations: int = 1,
    use_cold_l2: bool = False,
    tolerance: float = 1e-02,
    skip_ref_check: bool = False,
    **kwargs,
):
    """Execute a Pointer array batched dense GEMM operation on Blackwell architecture with performance benchmarking.
    The main difference between this and a regular bathced GEMM is that the inputs to the kernel are arrays of pointers.
    Every batch of each operand (A/B/D) has its own pointer. Thus, the size of the array of pointers is the batch size.
    These pointers NEED NOT be stored contiguously in memory.
    This example also demonstrates how cute_ext.tma_load/cute_ext.tma_store performs automatic device side TMA updates.
    Note that the dimensions of the operand for each batch are the same across all batches. That is, all batches of A have the same shape and stride, same for B and D.

    This function prepares input tensors, configures and launches the GEMM kernel,
    optionally performs reference validation, and benchmarks the execution performance.

    :param mnkl: Problem size (M, N, K, L)
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 494-513 / 第 494-513 行

~~~~python
    :type mnkl: Tuple[int, int, int, int]
    :param mma_tiler_mn: MMA tiling size.
    :type mma_tiler_mn: Tuple[int, int]
    :param cluster_shape_mn: Cluster shape.
    :type cluster_shape_mn: Tuple[int, int]
    :param ab_dtype: Data type for input tensors A and B
    :type ab_dtype: Type[Numeric]
    :param d_dtype: Data type for output tensor D
    :type d_dtype: Type[Numeric]
    """
    print("Running Blackwell Dense GEMM test with:")
    print(f"mnkl: {mnkl}")
    print(f"AB dtype: {ab_dtype}, D dtype: {c_dtype}, Acc dtype: {acc_dtype}")
    print(f"Matrix majors - A: {a_major}, B: {b_major}, D: {c_major}")
    print(f"Mma Tiler (M, N): {mma_tiler_mn}, Cluster Shape (M, N): {cluster_shape_mn}")
    print(f"Tolerance: {tolerance}")
    print(f"Warmup iterations: {warmup_iterations}")
    print(f"Iterations: {iterations}")
    print(f"Skip reference checking: {skip_ref_check}")
    print(f"Use cold L2: {'True' if use_cold_l2 else 'False'}")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 515-515 / 第 515-515 行

~~~~python
    m, n, k, l = mnkl
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 517-519 / 第 517-519 行

~~~~python
    ab_dtype = ab_dtype
    d_major = c_major
    d_dtype = c_dtype
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 521-532 / 第 521-532 行

~~~~python
    sm100_utils.check_gemm_tma_alignment(
        m,
        n,
        k,
        ab_dtype,
        ab_dtype,
        d_dtype,
        a_major,
        b_major,
        d_major,
        output_tensor_name="D",
    )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 534-535 / 第 534-535 行

~~~~python
    # a_tensor, b_tensor, d_tensor are cute Tensors where each element is an Int64 pointer to global memory
    # A_cutes, B_cutes, D_cutes are lists of cute Tensors for each batch of A/B/D
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 536-551 / 第 536-551 行

~~~~python
    (
        a_tensor,
        b_tensor,
        d_tensor,
        a_backing_torch_tensor,
        b_backing_torch_tensor,
        d_backing_torch_tensor,
        A_cutes,
        B_cutes,
        D_cutes,
        As_torch_gpu,
        Bs_torch_gpu,
        Ds_torch_gpu,
    ) = create_tensors_for_ptr_array(
        l, m, n, k, a_major, b_major, d_major, ab_dtype, d_dtype
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 553-564 / 第 553-564 行

~~~~python
    ptr_array_dense_gemm = DenseGemmPtrArrayKernel(
        mn_tiler=mma_tiler_mn,
        mma_dtype=(ab_dtype, acc_dtype, d_dtype),
        tmem_output_dtype=d_dtype,
        batch_count=l,
        A_shape=A_cutes[0].shape,
        A_stride=A_cutes[0].stride,
        B_shape=B_cutes[0].shape,
        B_stride=B_cutes[0].stride,
        D_shape=D_cutes[0].shape,
        D_stride=D_cutes[0].stride,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 566-568 / 第 566-568 行

~~~~python
    compiled_dense_gemm = cute_ext.compile(
        ptr_array_dense_gemm, a_tensor, b_tensor, d_tensor
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 570-575 / 第 570-575 行

~~~~python
    query = compiled_dense_gemm.get_aux_func(
        QueryDeviceWorkspaceFunc, kernel=ptr_array_dense_gemm.kernel
    )
    req = query(a_tensor, b_tensor, d_tensor)
    workspace = torch.empty(req.size_in_bytes, dtype=torch.uint8, device="cuda")
    workspace_cute = from_dlpack(workspace)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 577-577 / 第 577-577 行

~~~~python
    compiled_dense_gemm(a_tensor, b_tensor, d_tensor, workspace_cute)
~~~~

**EN**: Implements supporting Python/CuTeDSL logic needed by the example.
**CN**: 实现该示例所需的辅助 Python/CuTeDSL 逻辑。

### Lines 579-588 / 第 579-588 行

~~~~python
    if not skip_ref_check:
        for batch_idx in range(l):
            compare(
                As_torch_gpu[batch_idx].cpu(),
                Bs_torch_gpu[batch_idx].cpu(),
                Ds_torch_gpu[batch_idx],
                d_dtype,
                tolerance,
            )
        print("check reference: PASS")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 590-606 / 第 590-606 行

~~~~python
    def generate_tensors():
        (
            a_tensor,
            b_tensor,
            d_tensor,
            a_backing_torch_tensor,
            b_backing_torch_tensor,
            d_backing_torch_tensor,
            A_cutes,
            B_cutes,
            D_cutes,
            As_torch_gpu,
            Bs_torch_gpu,
            Ds_torch_gpu,
        ) = create_tensors_for_ptr_array(
            l, m, n, k, a_major, b_major, d_major, ab_dtype, d_dtype
        )
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 608-609 / 第 608-609 行

~~~~python
        ws = torch.empty(req.size_in_bytes, dtype=torch.uint8, device="cuda")
        ws_cute = from_dlpack(ws)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 611-613 / 第 611-613 行

~~~~python
        args = testing.JitArguments(a_tensor, b_tensor, d_tensor, ws_cute)
        args.add_to_scope([A_cutes, B_cutes, D_cutes])
        return args
~~~~

**EN**: Returns the constructed object or computed result to the caller.
**CN**: 把构造好的对象或计算结果返回给调用方。

### Lines 615-633 / 第 615-633 行

~~~~python
    workspace_count = 1
    if use_cold_l2:
        one_workspace_bytes = (
            sum(
                As_torch_gpu[batch_idx].numel() * As_torch_gpu[batch_idx].element_size()
                for batch_idx in range(l)
            )
            + sum(
                Bs_torch_gpu[batch_idx].numel() * Bs_torch_gpu[batch_idx].element_size()
                for batch_idx in range(l)
            )
            + sum(
                Ds_torch_gpu[batch_idx].numel() * Ds_torch_gpu[batch_idx].element_size()
                for batch_idx in range(l)
            )
        )
        workspace_count = testing.get_workspace_count(
            one_workspace_bytes, warmup_iterations, iterations
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 635-641 / 第 635-641 行

~~~~python
    exec_time = testing.benchmark(
        compiled_dense_gemm,
        workspace_generator=generate_tensors,
        workspace_count=workspace_count,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 643-643 / 第 643-643 行

~~~~python
    return exec_time
~~~~

**EN**: Returns the constructed object or computed result to the caller.
**CN**: 把构造好的对象或计算结果返回给调用方。

### Lines 646-646 / 第 646-646 行

~~~~python
if __name__ == "__main__":
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。

### Lines 648-654 / 第 648-654 行

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

### Lines 656-656 / 第 656-656 行

~~~~python
    parser = argparse.ArgumentParser(description="Example of Dense GEMM on Blackwell.")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 658-675 / 第 658-675 行

~~~~python
    parser.add_argument(
        "--mnkl",
        type=parse_comma_separated_ints,
        default=(256, 256, 512, 1),
        help="mnkl dimensions (comma-separated)",
    )
    parser.add_argument(
        "--mma_tiler_mn",
        type=parse_comma_separated_ints,
        default=(128, 128),
        help="Mma tile shape (comma-separated)",
    )
    parser.add_argument(
        "--cluster_shape_mn",
        type=parse_comma_separated_ints,
        default=(1, 1),
        help="Cluster shape (comma-separated)",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 677-679 / 第 677-679 行

~~~~python
    parser.add_argument("--ab_dtype", type=cutlass.dtype, default=cutlass.Float32)
    parser.add_argument("--d_dtype", type=cutlass.dtype, default=cutlass.Float32)
    parser.add_argument("--acc_dtype", type=cutlass.dtype, default=cutlass.Float32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 681-683 / 第 681-683 行

~~~~python
    parser.add_argument("--a_major", choices=["k", "m"], type=str, default="k")
    parser.add_argument("--b_major", choices=["k", "n"], type=str, default="k")
    parser.add_argument("--d_major", choices=["n", "m"], type=str, default="n")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 685-697 / 第 685-697 行

~~~~python
    parser.add_argument(
        "--warmup_iterations", type=int, default=0, help="Warmup iterations"
    )
    parser.add_argument(
        "--iterations", type=int, default=1, help="Number of iterations"
    )
    parser.add_argument("--use_cold_l2", action="store_true", help="Use cold L2")
    parser.add_argument(
        "--tolerance", type=float, default=1e-02, help="Tolerance for validation"
    )
    parser.add_argument(
        "--skip_ref_check", action="store_true", help="Skip reference checking"
    )
~~~~

**EN**: Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 699-699 / 第 699-699 行

~~~~python
    args = parser.parse_args()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 701-702 / 第 701-702 行

~~~~python
    if len(args.mnkl) != 4:
        parser.error("--mnkl must contain exactly 4 values")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 704-705 / 第 704-705 行

~~~~python
    if len(args.mma_tiler_mn) != 2:
        parser.error("--mma_tiler_mn must contain exactly 2 values")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 707-722 / 第 707-722 行

~~~~python
    exec_time = run(
        args.mnkl,
        args.mma_tiler_mn,
        args.cluster_shape_mn,
        args.ab_dtype,
        args.d_dtype,
        args.acc_dtype,
        args.a_major,
        args.b_major,
        args.d_major,
        args.warmup_iterations,
        args.iterations,
        args.use_cold_l2,
        args.tolerance,
        args.skip_ref_check,
    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 724-724 / 第 724-724 行

~~~~python
    print(f"Execution time: {exec_time} microseconds per iteration")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

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
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `typing.Type` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.List` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.experimental` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.base_dsl.typing.Numeric` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.experimental.host_runtime.QueryDeviceWorkspaceFunc` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `cutlass.utils.blackwell_helpers` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
