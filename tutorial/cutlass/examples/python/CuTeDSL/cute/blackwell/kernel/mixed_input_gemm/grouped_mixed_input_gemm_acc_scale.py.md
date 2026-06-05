# grouped_mixed_input_gemm_acc_scale.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/mixed_input_gemm/grouped_mixed_input_gemm_acc_scale.py`  
**Purpose / 用途**: Kernel example implementing grouped mixed input gemm acc scale with CuTeDSL. / 这是一个使用 CuTeDSL 实现 grouped mixed input gemm acc scale 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-47 / 第 29-47 行

~~~~python
import argparse
from math import log2, ceil
from typing import Union
import os
import sys

import torch
import cuda.bindings.driver as cuda

import cutlass
import cutlass.cute as cute
import cutlass.pipeline as pipeline
from cutlass.pipeline import pipeline_init_arrive, pipeline_init_wait
import cutlass.utils as utils
import cutlass.utils.blackwell_helpers as sm100_utils
import cutlass.utils.mixed_input_helpers as mixed_input_utils
from cutlass.utils.mixed_input_helpers import TransformMode
import cutlass.cute.testing as testing
from cutlass.cute.nvgpu import cpasync, tcgen05
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 49-51 / 第 49-51 行

~~~~python
if __name__ == "__main__":
    current_dir = os.path.dirname(os.path.abspath(__file__))
    sys.path.insert(0, os.path.join(current_dir, "../../../.."))
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 53-53 / 第 53-53 行

~~~~python
from cute.blackwell.kernel.mixed_input_gemm.mixed_input_host_utils import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 54-56 / 第 54-56 行

~~~~python
    create_tensors_for_contiguous_grouped_mixed_input_gemm as create_tensors,
    run_contiguous_grouped_ref_and_compare as run_ref_and_compare,
)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 58-99 / 第 58-99 行

~~~~python
"""
A mixed-input grouped GEMM example for the NVIDIA Blackwell SM100 architecture using CUTE DSL.

Compared to the grouped_mixed_input_gemm.py example, this acc_scale example demonstrates a different implementation by
performing the scaling step on the accumulator instead of the input A tensor. The original computation is
```
C = (type_convert(A) * scale) x B.
```
In this example, we swap the scaling step and MMA and reformulate the computation as follows:
```
C = scale * (type_convert(A) x B).
```
The reformulation is valid when `scale_granularity_k` is an exact multiple of `mma_tiler_mnk[2]`. With this change,
the number of operations in the scaling step changes from `CTA_TILE_M * CTA_TILE_K` to `CTA_TILE_M * CTA_TILE_N`.
When `CTA_TILE_N` is smaller than `CTA_TILE_K` (common in decoding), the acc_scale implementation can
be more efficient due to fewer scaling operations.
Only convert-scale mode is supported in this example; convert-only mode has no scaling.
Other than the above changes, the computation flow is the same as the grouped_mixed_input_gemm.py example.

To run this example:

.. code-block:: bash

    python examples/blackwell/mixed_input_gemm/grouped_mixed_input_gemm_acc_scale.py      \
      --a_dtype Int4 --b_dtype BFloat16                                  \
      --scale_granularity_m 1 --scale_granularity_k 256                  \
      --c_dtype BFloat16 --acc_dtype Float32                             \
      --mma_tiler_mnk 256,16,128 --cluster_shape_mn 2,1                  \
      --use_2cta_instrs --mnkl 1024,8192,6144,16                         \

To collect performance with NCU profiler:

.. code-block:: bash

    ncu python examples/blackwell/mixed_input_gemm/grouped_mixed_input_gemm_acc_scale.py  \
      --a_dtype Int4 --b_dtype BFloat16                                  \
      --scale_granularity_m 1 --scale_granularity_k 256                  \
      --c_dtype BFloat16 --acc_dtype Float32                             \
      --mma_tiler_mnk 128,8,256 --cluster_shape_mn 1,1                   \
      --mnkl 4096,8,8192,32                                              \
      --warmup_iterations 1 --iterations 10 --skip_ref_check
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 102-128 / 第 102-128 行

~~~~python
class GroupedMixedInputGemmAccScaleKernel:
    """
    Mixed-input grouped GEMM kernel with scaling applied on accumulator for NVIDIA Blackwell SM100 architecture.

    This kernel supports GEMM operations where input tensors A and B have different data types, with tensor A
    being converted to the precision of tensor B before matrix multiplication. The scaling is applied on
    the accumulator instead of the input A tensor to reduce the number of operations in the scaling step.
    Tensor A is in shape of [M, K, L] with L being the number of groups. Tensor B is in shape of [N, K] and a group search algorithm
    is applied along the N mode to find the group index for each CTA tile. A cumsum tensor provides the offsets of each group along the N mode.

    :param scale_granularity_m: Number of elements sharing the same scale factor along the M mode
    :type scale_granularity_m: int
    :param scale_granularity_k: Number of elements sharing the same scale factor along the K mode
    :type scale_granularity_k: int
    :param acc_dtype: Data type for accumulation during computation
    :type acc_dtype: type[cutlass.Numeric]
    :param use_2cta_instrs: Whether to use CTA group 2 for advanced thread cooperation
    :type use_2cta_instrs: bool
    :param mma_tiler_mnk: Shape of the Matrix Multiply-Accumulate (MMA) tile (M, N, K)
    :type mma_tiler_mnk: tuple[int, int, int]
    :param cluster_shape_mn: Cluster dimensions (M,N) for parallel processing
    :type cluster_shape_mn: tuple[int, int]
    :param group_count: The total number of groups
    :type group_count: int
    :param shuffle_a: Whether to use shuffle intrinsic for int4-to-bf16 conversion
    :type shuffle_a: bool
    """
~~~~

**EN**: Defines `GroupedMixedInputGemmAccScaleKernel`, a reusable Python class that packages configuration and behavior for this example. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `GroupedMixedInputGemmAccScaleKernel`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 130-157 / 第 130-157 行

~~~~python
    def __init__(
        self,
        scale_granularity_m: int,
        scale_granularity_k: int,
        acc_dtype: type[cutlass.Numeric],
        use_2cta_instrs: bool,
        mma_tiler_mnk: tuple[int, int, int],
        cluster_shape_mn: tuple[int, int],
        group_count: int,
        shuffle_a: bool,
    ):
        """
        Initializes the mixed-input GEMM kernel with a specified configuration.
        """
        # Scale granularity defines how many elements share the same scale factor
        # along the M and K modes.
        self.scale_granularity_m = scale_granularity_m
        self.scale_granularity_k = scale_granularity_k
        # Set transform mode
        if cutlass.const_expr(
            self.scale_granularity_m == 0 and self.scale_granularity_k == 0
        ):
            # Acc-update kernel variant is only for convert-scale mode.
            raise ValueError("convert-only mode is not supported for acc-scale kernel")
        self.scale_mode = TransformMode.ConvertScale
        # scale_granularity_k must be exactly multiple of CTA tile shape K to allow acc-update recipe.
        if cutlass.const_expr(self.scale_granularity_k % mma_tiler_mnk[2] != 0):
            raise ValueError(
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 158-159 / 第 158-159 行

~~~~python
                "scale_granularity_k must be exactly multiple of CTA tile shape K"
            )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 161-181 / 第 161-181 行

~~~~python
        self.group_count = group_count
        self.acc_dtype = acc_dtype
        self.use_2cta_instrs = use_2cta_instrs
        self.cluster_shape_mn = cluster_shape_mn
        self.mma_tiler = mma_tiler_mnk
        self.shuffle_a = shuffle_a
        self.cta_group = (
            tcgen05.CtaGroup.TWO if self.use_2cta_instrs else tcgen05.CtaGroup.ONE
        )
        # transformation ktile loop unrolling factor
        self.transform_k_tile_unroll_factor = 2
        # Set specialized warp ids
        self.epilog_warp_id = (
            0,
            1,
            2,
            3,
        )
        self.mma_warp_id = 4
        self.tma_warp_id = 5
        self.scale_tma_warp_id = 6
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 182-208 / 第 182-208 行

~~~~python
        # Schedule warp to do the group search
        self.schedule_warp_id = 7
        self.transform_warp_id = (
            8,
            9,
            10,
            11,
        )
        # Define expected register count for different warps
        # Reserve more registers for transformation and epilogue warps
        self.num_regs_epilogue_warps = 168
        self.num_regs_mma_warp = 80
        self.num_regs_tma_warps = 72
        self.num_regs_transform_warps = 240
        self.num_regs_schedule_warp = 64
        self.threads_per_cta = 32 * (
            max(
                (
                    self.mma_warp_id,
                    self.tma_warp_id,
                    self.scale_tma_warp_id,
                    *self.epilog_warp_id,
                    *self.transform_warp_id,
                )
            )
            + 1
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 210-210 / 第 210-210 行

~~~~python
        # Set barrier id for cta sync, epilogue sync, tmem ptr sync, and transform sync
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 211-216 / 第 211-216 行

~~~~python
        self.epilog_sync_barrier = pipeline.NamedBarrier(
            1, 32 * len(self.epilog_warp_id)
        )
        self.tmem_ptr_sync_barrier = pipeline.NamedBarrier(2, self.threads_per_cta)
        self.cta_sync_barrier = pipeline.NamedBarrier(3, self.threads_per_cta)
        self.sched_sync_barrier = pipeline.NamedBarrier(4, 32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 218-218 / 第 218-218 行

~~~~python
        self.smem_buffer_align_bytes = 1024
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 220-247 / 第 220-247 行

~~~~python
    def _setup_attributes(self):
        """
        Set up configurations that are dependent on GEMM inputs
        """
        # Deduce where the transformed A tensor is stored, shared memory(SMEM) or tensor memory(TMEM)
        self.transform_a_source = mixed_input_utils.get_transform_a_source(
            self.a_major_mode
        )
        tiled_mma = sm100_utils.make_trivial_tiled_mma(
            self.mma_dtype,
            self.a_major_mode,
            self.b_major_mode,
            self.acc_dtype,
            self.cta_group,
            self.mma_tiler[:2],
            self.transform_a_source,
        )
        self.cta_tile_shape_mnk = (
            self.mma_tiler[0] // cute.size(tiled_mma.thr_id.shape),
            self.mma_tiler[1],
            self.mma_tiler[2],
        )
        self.cluster_tile_shape_mnk = (
            self.cluster_shape_mn[0] * self.cta_tile_shape_mnk[0],
            self.cluster_shape_mn[1] * self.cta_tile_shape_mnk[1],
            self.cta_tile_shape_mnk[2],
        )
        self.cluster_layout_vmnk = cute.tiled_divide(
~~~~

**EN**: Defines `_setup_attributes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 定义 `_setup_attributes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 248-254 / 第 248-254 行

~~~~python
            cute.make_layout((*self.cluster_shape_mn, 1)),
            (tiled_mma.thr_id.shape,),
        )
        self.num_mcast_ctas_a = cute.size(self.cluster_layout_vmnk.shape[2])
        self.num_mcast_ctas_b = cute.size(self.cluster_layout_vmnk.shape[1])
        self.is_a_mcast = self.num_mcast_ctas_a > 1
        self.is_b_mcast = self.num_mcast_ctas_b > 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 256-261 / 第 256-261 行

~~~~python
        self.epi_tile = sm100_utils.compute_epilogue_tile_shape(
            self.cta_tile_shape_mnk,
            self.use_2cta_instrs,
            self.c_layout,
            self.c_dtype,
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 263-263 / 第 263-263 行

~~~~python
        # Compute tensor memory(TMEM) columns and stages for each pipeline
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 264-286 / 第 264-286 行

~~~~python
        (
            self.num_load2trans_stage,
            self.num_scale_load2accu_stage,
            self.num_trans2mma_stage,
            self.num_acc_stage,
            self.num_c_stage,
            self.num_tile_info_stage,
            self.num_acc_tmem_cols,
            self.num_a_tmem_cols,
        ) = self._compute_stages_and_tmem_cols(
            tiled_mma,
            self.mma_tiler,
            self.cta_tile_shape_mnk,
            self.epi_tile,
            self.a_dtype,
            self.b_dtype,
            self.c_dtype,
            self.c_layout,
            self.transform_a_source,
            self.scale_granularity_m,
            self.scale_granularity_k,
            self.smem_buffer_align_bytes,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 288-290 / 第 288-290 行

~~~~python
        # Align TMEM columns for allocation
        # TMEM allocation requires power-of-2 column alignment
        # and must meet minimum allocation requirements
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 291-316 / 第 291-316 行

~~~~python
        self.num_tmem_alloc_cols = cute.round_up(
            self.num_acc_tmem_cols + self.num_a_tmem_cols,
            cute.arch.get_min_tmem_alloc_cols("sm_100"),
        )
        self.num_tmem_alloc_cols = 2 ** (ceil(log2(self.num_tmem_alloc_cols)))
        # Get smem layout for C tensor
        self.c_smem_layout_staged = sm100_utils.make_smem_layout_epi(
            self.c_dtype,
            self.c_layout,
            self.epi_tile,
            self.num_c_stage,
        )
        # Get smem layout for A, transformed A, and B
        (
            self.smem_layout_a,
            self.smem_layout_a_transform,
            self.smem_layout_b,
        ) = mixed_input_utils.compute_smem_layout(
            tiled_mma,
            self.mma_tiler,
            self.a_dtype,
            self.b_dtype,
            self.num_load2trans_stage,
            self.num_trans2mma_stage,
        )
        # Check if stages match the requirements for unrolling
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 317-336 / 第 317-336 行

~~~~python
        if (
            self.num_scale_load2accu_stage < self.transform_k_tile_unroll_factor
            or self.num_trans2mma_stage < self.transform_k_tile_unroll_factor
        ):
            raise ValueError("Not enough SMEM capacity for selected tile size")
        # Get scale tile shape and smem layout for scale tensor
        # ((M_SHARING_SCALE, NUM_SCALES_M),(K_SHARING_SCALE, NUM_SCALES_K), STAGES)
        (
            self.scale_tile_shape,
            self.smem_layout_scale_per_stage,
            self.smem_layout_scale,
        ) = mixed_input_utils.get_smem_layout_scale(
            self.mma_tiler,
            self.use_2cta_instrs,
            self.scale_granularity_m,
            self.scale_granularity_k,
            self.scale_major_mode,
            self.a_scale_dtype,
            self.num_scale_load2accu_stage,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 338-353 / 第 338-353 行

~~~~python
    def _validate_inputs(
        self,
        a: cute.Tensor,
        a_scale: cute.Tensor,
        b: cute.Tensor,
        c: cute.Tensor,
    ) -> None:
        """
        Validates input tensors and their properties.
        """
        # Validate scale tensor major mode
        if cutlass.const_expr(
            utils.LayoutEnum.from_tensor(a_scale).mma_major_mode()
            != tcgen05.OperandMajorMode.MN
        ):
            raise ValueError("scale_major_mode must be M-major")
~~~~

**EN**: Defines `_validate_inputs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `_validate_inputs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 355-373 / 第 355-373 行

~~~~python
    @cute.jit
    def __call__(
        self,
        a: cute.Tensor,
        a_scale: cute.Tensor,
        b: cute.Tensor,
        cumsum: cute.Tensor,
        c: cute.Tensor,
        max_active_clusters: cutlass.Constexpr,
        stream: cuda.CUstream,
    ):
        """
        Executes the Mixed Input Grouped GEMM operation.
        """
        self.a_dtype: type[cutlass.Numeric] = a.element_type
        self.a_scale_dtype: type[cutlass.Numeric] = a_scale.element_type
        self.b_dtype: type[cutlass.Numeric] = b.element_type
        self.c_dtype: type[cutlass.Numeric] = c.element_type
        self.mma_dtype = self.b_dtype
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 375-385 / 第 375-385 行

~~~~python
        self.a_major_mode = utils.LayoutEnum.from_tensor(a).mma_major_mode()
        self.scale_major_mode = utils.LayoutEnum.from_tensor(a_scale).mma_major_mode()
        self.b_major_mode = utils.LayoutEnum.from_tensor(b).mma_major_mode()
        self.c_layout = utils.LayoutEnum.from_tensor(c)
        # Get gmem layout for scale tensor
        self.gmem_layout_scale = mixed_input_utils.get_gmem_layout_scale(
            a.shape,
            self.scale_granularity_m,
            self.scale_granularity_k,
            self.scale_major_mode,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 387-387 / 第 387-387 行

~~~~python
        # Validate inputs
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 388-388 / 第 388-388 行

~~~~python
        self._validate_inputs(a, a_scale, b, c)
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 390-390 / 第 390-390 行

~~~~python
        # Setup attributes that dependent on gemm inputs
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 391-391 / 第 391-391 行

~~~~python
        self._setup_attributes()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 393-409 / 第 393-409 行

~~~~python
        tiled_mma = sm100_utils.make_trivial_tiled_mma(
            self.mma_dtype,
            self.a_major_mode,
            self.b_major_mode,
            self.acc_dtype,
            self.cta_group,
            self.mma_tiler[:2],
            self.transform_a_source,
        )
        # Set up gmem copy atoms for A, scale, and B
        a_op = mixed_input_utils.get_tma_atom_kind(
            self.is_a_mcast, self.use_2cta_instrs, is_b=False
        )
        b_op = mixed_input_utils.get_tma_atom_kind(
            self.is_b_mcast, self.use_2cta_instrs, is_b=True
        )
        a_scale_op = a_op
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 410-422 / 第 410-422 行

~~~~python
        # Deduce TMA copy atom and TMA tensor for A, scale, and B
        smem_layout_a_per_stage = cute.slice_(self.smem_layout_a, (None, None, None, 0))
        tma_atom_a, tma_tensor_a = cute.nvgpu.make_tiled_tma_atom_A(
            a_op,
            a,
            smem_layout_a_per_stage,
            self.mma_tiler,
            tiled_mma,
            self.cluster_layout_vmnk.shape,
            internal_type=(
                cutlass.TFloat32 if a.element_type is cutlass.Float32 else None
            ),
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 424-424 / 第 424-424 行

~~~~python
        # Partition smem layout for scale tensor to make it compatible with TMA atom
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 425-444 / 第 425-444 行

~~~~python
        smem_layout_for_tma_atom = cute.get(
            tiled_mma._thrfrg_A(self.smem_layout_scale_per_stage.outer), mode=[1]
        )
        # ((MMA_M, MMA_K), REST_M, REST_K)
        smem_layout_for_tma_atom = cute.dice(
            smem_layout_for_tma_atom,
            (1, (1,) * cute.rank(self.smem_layout_scale_per_stage.outer)),
        )
        tma_atom_scale, tma_tensor_scale = cute.nvgpu.make_tiled_tma_atom_A(
            a_scale_op,
            cute.make_tensor(a_scale.iterator, self.gmem_layout_scale),
            smem_layout_for_tma_atom,
            # (SCALE_M, 1, SCALE_K)
            (self.scale_tile_shape[0], 1, self.scale_tile_shape[1]),
            tiled_mma,
            self.cluster_layout_vmnk.shape,
            internal_type=(
                cutlass.TFloat32 if a_scale.element_type is cutlass.Float32 else None
            ),
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 446-457 / 第 446-457 行

~~~~python
        smem_layout_b_per_stage = cute.slice_(self.smem_layout_b, (None, None, None, 0))
        tma_atom_b, tma_tensor_b = cute.nvgpu.make_tiled_tma_atom_B(
            b_op,
            b,
            smem_layout_b_per_stage,
            self.mma_tiler,
            tiled_mma,
            self.cluster_layout_vmnk.shape,
            internal_type=(
                cutlass.TFloat32 if b.element_type is cutlass.Float32 else None
            ),
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 459-459 / 第 459-459 行

~~~~python
        # Calculate copy size for tensor A, B, and scale
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 460-464 / 第 460-464 行

~~~~python
        a_copy_size = cute.size_in_bytes(self.a_dtype, smem_layout_a_per_stage)
        b_copy_size = cute.size_in_bytes(self.b_dtype, smem_layout_b_per_stage)
        a_scale_copy_size = cute.size_in_bytes(
            self.a_scale_dtype, self.smem_layout_scale_per_stage
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 466-474 / 第 466-474 行

~~~~python
        self.num_tma_load_bytes_a = a_copy_size
        self.num_tma_load_bytes_b = b_copy_size * cute.size(tiled_mma.thr_id.shape)
        self.num_tma_load_bytes_scale = a_scale_copy_size
        self.tile_sched_params, grid = self._compute_grid(
            c,
            self.cta_tile_shape_mnk,
            self.cluster_shape_mn,
            max_active_clusters,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 476-482 / 第 476-482 行

~~~~python
        epi_smem_layout = cute.slice_(self.c_smem_layout_staged, (None, None, 0))
        tma_atom_c, tma_tensor_c = cpasync.make_tiled_tma_atom(
            cpasync.CopyBulkTensorTileS2GOp(),
            c,
            epi_smem_layout,
            self.epi_tile,
        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 484-484 / 第 484-484 行

~~~~python
        # Shared memory structure
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 485-512 / 第 485-512 行

~~~~python
        @cute.struct
        class SharedStorage:
            # buffer holding group search results
            tile_info: cute.struct.MemRange[cutlass.Int32, 4 * self.num_tile_info_stage]
            a_load2trans_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_load2trans_stage
            ]
            a_load2trans_empty_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_load2trans_stage
            ]
            a_scale_load2accu_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_scale_load2accu_stage
            ]
            a_scale_load2accu_empty_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_scale_load2accu_stage
            ]
            a_trans2mma_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_trans2mma_stage
            ]
            a_trans2mma_empty_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_trans2mma_stage
            ]
            b_load2mma_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_load2trans_stage
            ]
            b_load2mma_empty_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_load2trans_stage
            ]
~~~~

**EN**: Defines `SharedStorage`, a reusable Python class that packages configuration and behavior for this example. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `SharedStorage`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 513-522 / 第 513-522 行

~~~~python
            acc_full_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_acc_stage]
            acc_empty_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_acc_stage]
            tile_info_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_tile_info_stage
            ]
            tile_info_empty_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_tile_info_stage
            ]
            tmem_dealloc_mbar: cutlass.Int64
            tmem_holding_buf: cutlass.Int32
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 524-524 / 第 524-524 行

~~~~python
        self.shared_storage = SharedStorage
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 526-526 / 第 526-526 行

~~~~python
        # Launch kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 527-554 / 第 527-554 行

~~~~python
        self.kernel(
            tiled_mma,
            tma_atom_a,
            tma_tensor_a,
            tma_atom_scale,
            tma_tensor_scale,
            tma_atom_b,
            tma_tensor_b,
            tma_atom_c,
            tma_tensor_c,
            c,
            cumsum,
            self.group_count,
            self.cluster_layout_vmnk,
            self.smem_layout_a,
            self.smem_layout_scale,
            self.smem_layout_a_transform,
            self.smem_layout_b,
            self.c_smem_layout_staged,
            self.epi_tile,
            self.tile_sched_params,
        ).launch(
            grid=grid,
            block=[self.threads_per_cta, 1, 1],
            cluster=(*self.cluster_shape_mn, 1),
            min_blocks_per_mp=1,
            stream=stream,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 555-555 / 第 555-555 行

~~~~python
        return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 557-557 / 第 557-557 行

~~~~python
    # GPU device kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 558-585 / 第 558-585 行

~~~~python
    @cute.kernel
    def kernel(
        self,
        tiled_mma: cute.TiledMma,
        tma_atom_a: cute.CopyAtom,
        mA_mkl: cute.Tensor,
        tma_atom_s: cute.CopyAtom,
        mS_mkl: cute.Tensor,
        tma_atom_b: cute.CopyAtom,
        mB_nkl: cute.Tensor,
        tma_atom_c: cute.CopyAtom,
        mC_mnl: cute.Tensor,
        tensor_c: cute.Tensor,
        cumsum: cute.Tensor,
        group_count: cutlass.Constexpr[int],
        cluster_layout_vmnk: cute.Layout,
        a_smem_layout: cute.ComposedLayout,
        scale_smem_layout: cute.ComposedLayout,
        a_smem_layout_transform: cute.ComposedLayout,
        b_smem_layout: cute.ComposedLayout,
        c_smem_layout_staged: cute.ComposedLayout,
        epi_tile: cute.Tile,
        tile_sched_params: utils.PersistentTileSchedulerParams,
    ):
        """
        GPU device kernel performing the Persistent Mixed-Input Grouped GEMM computation.
        """
        warp_idx = cute.arch.make_warp_uniform(cute.arch.warp_idx())
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 586-593 / 第 586-593 行

~~~~python
        tidx, _, _ = cute.arch.thread_idx()
        bidx, bidy, bidz = cute.arch.block_idx()
        # Prefetch TMA descriptors
        if warp_idx == self.epilog_warp_id[0]:
            cpasync.prefetch_descriptor(tma_atom_a)
            cpasync.prefetch_descriptor(tma_atom_b)
            cpasync.prefetch_descriptor(tma_atom_s)
            cpasync.prefetch_descriptor(tma_atom_c)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 595-598 / 第 595-598 行

~~~~python
        use_2cta_instrs = cute.size(tiled_mma.thr_id.shape) == 2
        bidx, bidy, bidz = cute.arch.block_idx()
        # Compute how many k_tiles share the same scale
        num_k_tiles_per_scale = self.scale_granularity_k // self.cta_tile_shape_mnk[2]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 600-608 / 第 600-608 行

~~~~python
        mma_tile_coord_v = bidx % cute.size(tiled_mma.thr_id.shape)
        is_leader_cta = mma_tile_coord_v == 0
        cta_rank_in_cluster = cute.arch.make_warp_uniform(
            cute.arch.block_idx_in_cluster()
        )
        block_in_cluster_coord_vmnk = cluster_layout_vmnk.get_flat_coord(
            cta_rank_in_cluster
        )
        tidx, _, _ = cute.arch.thread_idx()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 610-611 / 第 610-611 行

~~~~python
        smem = utils.SmemAllocator()
        storage = smem.allocate(self.shared_storage)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 613-614 / 第 613-614 行

~~~~python
        # Initialize load2transform pipeline, which tracks the dependencies between TMA's loading
        # of A and B, and the transformation of A and MMA's consumption
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 615-634 / 第 615-634 行

~~~~python
        transform_thread_idx = (
            tidx - 32 * self.transform_warp_id[0]
            if tidx >= 32 * self.transform_warp_id[0]
            else tidx
        )
        a_load2trans_pipeline = pipeline.PipelineTmaAsync.create(
            barrier_storage=storage.a_load2trans_full_mbar_ptr.data_ptr(),
            num_stages=self.num_load2trans_stage,
            producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread),
            consumer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                self.num_mcast_ctas_a * len(self.transform_warp_id),
            ),
            tx_count=self.num_tma_load_bytes_a,
            cta_layout_vmnk=cluster_layout_vmnk,
            tidx=transform_thread_idx,
            mcast_mode_mn=(1, 0),  # multicast for A will only happen on the M-mode
            defer_sync=True,
        )
        # Initialize scale_load2accu pipeline, which tracks the dependencies between TMA's loading
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 635-654 / 第 635-654 行

~~~~python
        # of scale, and the accumulator update
        num_producers_a_scale = self.num_mcast_ctas_a
        scale_load2accu_pipeline = pipeline.PipelineTmaAsync.create(
            barrier_storage=storage.a_scale_load2accu_full_mbar_ptr.data_ptr(),
            num_stages=self.num_scale_load2accu_stage,
            producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread),
            consumer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                num_producers_a_scale * len(self.epilog_warp_id),
            ),
            tx_count=self.num_tma_load_bytes_scale,
            cta_layout_vmnk=cluster_layout_vmnk,
            tidx=tidx,
            mcast_mode_mn=(
                1,
                0,
            ),  # multicast for scale_a will only happen on the M-mode
            defer_sync=True,
        )
        # Initialize transform2mma pipeline, which tracks the dependencies between the type conversion
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 655-682 / 第 655-682 行

~~~~python
        # of A and MMA's consumption on converted A
        cta_v_size = cute.size(cluster_layout_vmnk, mode=[0])
        trans2mma_pipeline = pipeline.PipelineAsyncUmma.create(
            barrier_storage=storage.a_trans2mma_full_mbar_ptr.data_ptr(),
            num_stages=self.num_trans2mma_stage,
            producer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                32 * len(self.transform_warp_id) * cta_v_size,
            ),
            consumer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread),
            cta_layout_vmnk=cluster_layout_vmnk,
            defer_sync=True,
        )
        # Initialize pipeline for tensor B load to MMA
        # MMA warp informs TMA warp to proceed to load next tile of B tensor
        b_load2mma_pipeline = pipeline.PipelineTmaUmma.create(
            barrier_storage=storage.b_load2mma_full_mbar_ptr.data_ptr(),
            num_stages=self.num_load2trans_stage,
            producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread),
            consumer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread, self.num_mcast_ctas_b
            ),
            tx_count=self.num_tma_load_bytes_b,
            cta_layout_vmnk=cluster_layout_vmnk,
            mcast_mode_mn=(0, 1),  # multicast for B will only happen on the N-mode
            defer_sync=True,
        )
        # Initialize accumulator pipeline, which tracks the dependencies between
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 683-707 / 第 683-707 行

~~~~python
        # MMA's computation of accumulators and epilogue warps' consumption of accumulators
        acc_pipeline = pipeline.PipelineUmmaAsync.create(
            barrier_storage=storage.acc_full_mbar_ptr.data_ptr(),
            num_stages=self.num_acc_stage,
            producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread),
            consumer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread, cta_v_size * len(self.epilog_warp_id)
            ),
            cta_layout_vmnk=cluster_layout_vmnk,
            defer_sync=True,
        )
        # Initialize tile info pipeline, which tracks the dependencies between
        # tile scheduling warp and other warps
        # Skip scheduler warp when computing consumer thread count
        num_tile_info_pipeline_consumer_threads = self.threads_per_cta - 32
        tile_info_pipeline = pipeline.PipelineAsync.create(
            barrier_storage=storage.tile_info_full_mbar_ptr.data_ptr(),
            num_stages=self.num_tile_info_stage,
            producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread, 32),
            consumer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                num_tile_info_pipeline_consumer_threads,
            ),
            defer_sync=True,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 709-709 / 第 709-709 行

~~~~python
        # Tensor memory dealloc barrier init
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 710-716 / 第 710-716 行

~~~~python
        tmem = utils.TmemAllocator(
            storage.tmem_holding_buf.ptr,
            barrier_for_retrieve=self.tmem_ptr_sync_barrier,
            allocator_warp_id=self.epilog_warp_id[0],
            is_two_cta=use_2cta_instrs,
            two_cta_tmem_dealloc_mbar_ptr=storage.tmem_dealloc_mbar.ptr,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 718-718 / 第 718-718 行

~~~~python
        # Cluster arrive after barrier init
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 719-719 / 第 719-719 行

~~~~python
        pipeline_init_arrive(cluster_shape_mn=self.cluster_shape_mn, is_relaxed=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 721-721 / 第 721-721 行

~~~~python
        # Setup smem tensor A/scale/B/C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 722-747 / 第 722-747 行

~~~~python
        sC = smem.allocate_tensor(
            element_type=self.c_dtype,
            layout=c_smem_layout_staged.outer,
            byte_alignment=self.smem_buffer_align_bytes,
            swizzle=c_smem_layout_staged.inner,
        )
        sA_input = smem.allocate_tensor(
            element_type=self.a_dtype,
            layout=a_smem_layout.outer,
            byte_alignment=self.smem_buffer_align_bytes,
            swizzle=a_smem_layout.inner,
        )
        sS_input = smem.allocate_tensor(
            element_type=self.mma_dtype,
            layout=scale_smem_layout.outer,
            byte_alignment=self.smem_buffer_align_bytes,
            swizzle=scale_smem_layout.inner,
        )
        sB_input = smem.allocate_tensor(
            element_type=self.b_dtype,
            layout=b_smem_layout.outer,
            byte_alignment=self.smem_buffer_align_bytes,
            swizzle=b_smem_layout.inner,
        )
        sA_transform = None
        # Get smem tensor for transformed A when transform_a_source is SMEM
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 748-757 / 第 748-757 行

~~~~python
        if cutlass.const_expr(self.transform_a_source == tcgen05.OperandSource.SMEM):
            sA_transform = smem.allocate_tensor(
                element_type=self.mma_dtype,
                layout=a_smem_layout_transform.outer,
                byte_alignment=self.smem_buffer_align_bytes,
                swizzle=a_smem_layout_transform.inner,
            )
        sTile_info = storage.tile_info.get_tensor(
            cute.make_layout((4, self.num_tile_info_stage), stride=(1, 4))
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 759-759 / 第 759-759 行

~~~~python
        # Compute multicast mask for A/B buffer full
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 760-771 / 第 760-771 行

~~~~python
        a_full_mcast_mask = None
        b_full_mcast_mask = None
        s_full_mcast_mask = None
        if cutlass.const_expr(self.is_a_mcast or self.is_b_mcast or use_2cta_instrs):
            a_full_mcast_mask = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mcast_mode=2
            )
            # Scale tensor shares the same multicast mask as the A tensor
            s_full_mcast_mask = a_full_mcast_mask
            b_full_mcast_mask = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mcast_mode=1
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 773-774 / 第 773-774 行

~~~~python
        # local_tile partition global tensors
        # (bM, bK, loopM, loopK, loopL)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 775-793 / 第 775-793 行

~~~~python
        gA_mkl = cute.local_tile(
            mA_mkl, cute.slice_(self.mma_tiler, (None, 0, None)), (None, None, None)
        )
        # (bM, bK, loopM, loopK, loopL)
        gS_mkl = cute.local_tile(
            mS_mkl, cute.slice_(self.mma_tiler, (None, 0, None)), (None, None, None)
        )
        # (bN, bK, loopN, loopK, loopL)
        gB_nkl = cute.local_tile(
            mB_nkl, cute.slice_(self.mma_tiler, (0, None, None)), (None, None, None)
        )
        # (bM, bN, loopM, loopN, loopL)
        gC_mnl = cute.local_tile(
            mC_mnl, cute.slice_(self.mma_tiler, (None, None, 0)), (None, None, None)
        )
        gC_mnl_simt = cute.local_tile(
            tensor_c, cute.slice_(self.mma_tiler, (None, None, 0)), (None, None, None)
        )
        k_tile_cnt = cute.size(gA_mkl, mode=[3])
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 795-795 / 第 795-795 行

~~~~python
        # Partition global tensor for TiledMMA_A/B/C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 796-805 / 第 796-805 行

~~~~python
        thr_mma = tiled_mma.get_slice(mma_tile_coord_v)
        # (MMA, MMA_M, MMA_K, loopM, loopK, loopL)
        tCgA = thr_mma.partition_A(gA_mkl)
        # (MMA, MMA_M, MMA_K, loopM, loopK, loopL)
        tCgS = thr_mma.partition_A(gS_mkl)
        # (MMA, MMA_N, MMA_K, loopN, loopK, loopL)
        tCgB = thr_mma.partition_B(gB_nkl)
        # (MMA, MMA_M, MMA_N, loopM, loopN, loopL)
        tCgC = thr_mma.partition_C(gC_mnl)
        tCgC_simt = thr_mma.partition_C(gC_mnl_simt)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 807-807 / 第 807-807 行

~~~~python
        # Setup copy atom to load A from shared memory for further transformation
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 808-821 / 第 808-821 行

~~~~python
        copy_atom_a_input = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(), self.a_dtype, num_bits_per_copy=32
        )
        a_smem_shape = tiled_mma.partition_shape_A(
            cute.dice(self.mma_tiler, (1, None, 1))
        )
        # Setup copy atom to store transformed A into tensor memory or shared memory
        copy_atom_a_transform = mixed_input_utils.get_copy_atom_a_transform(
            self.mma_dtype,
            self.use_2cta_instrs,
            self.transform_a_source,
            a_smem_shape,
            self.a_dtype,
        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 823-824 / 第 823-824 行

~~~~python
        # Partition global/shared tensor for TMA load A/B
        # TMA load A partition_S/D
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 825-836 / 第 825-836 行

~~~~python
        a_cta_layout = cute.make_layout(
            cute.slice_(cluster_layout_vmnk, (0, 0, None, 0)).shape
        )
        # ((atom_v, rest_v), STAGE)
        # ((atom_v, rest_v), loopM, loopK, loopL)
        tAsA, tAgA = cpasync.tma_partition(
            tma_atom_a,
            block_in_cluster_coord_vmnk[2],
            a_cta_layout,
            cute.group_modes(sA_input, 0, 3),
            cute.group_modes(tCgA, 0, 3),
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 838-849 / 第 838-849 行

~~~~python
        thr_mma_leader_cta = tiled_mma.get_slice(0)
        # (MMA, MMA_M, MMA_K, STAGE)
        tCsS = thr_mma_leader_cta.partition_A(sS_input)
        # ((atom_v, rest_v), STAGE)
        # ((atom_v, rest_v), loopM, loopK, loopL)
        tSsS, tSgS = mixed_input_utils.scale_tma_partition(
            tCsS,
            tCgS,
            tma_atom_s,
            block_in_cluster_coord_vmnk,
            a_cta_layout,
        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 851-851 / 第 851-851 行

~~~~python
        # TMA load B partition_S/D
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 852-863 / 第 852-863 行

~~~~python
        b_cta_layout = cute.make_layout(
            cute.slice_(cluster_layout_vmnk, (0, None, 0, 0)).shape
        )
        # ((atom_v, rest_v), STAGE)
        # ((atom_v, rest_v), loopM, loopK, loopL)
        tBsB, tBgB = cpasync.tma_partition(
            tma_atom_b,
            block_in_cluster_coord_vmnk[1],
            b_cta_layout,
            cute.group_modes(sB_input, 0, 3),
            cute.group_modes(tCgB, 0, 3),
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 865-865 / 第 865-865 行

~~~~python
        # (MMA, MMA_N, MMA_K, STAGE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 866-871 / 第 866-871 行

~~~~python
        tCrB = tiled_mma.make_fragment_B(sB_input)
        # (MMA, MMA_M, MMA_N)
        acc_shape = tiled_mma.partition_shape_C(self.mma_tiler[:2])
        tCtAcc_fake = tiled_mma.make_fragment_C(
            cute.append(acc_shape, self.num_acc_stage)
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 873-873 / 第 873-873 行

~~~~python
        # Cluster wait before TMEM alloc and ensure pipelines are ready
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 874-874 / 第 874-874 行

~~~~python
        pipeline_init_wait(cluster_shape_mn=self.cluster_shape_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 876-876 / 第 876-876 行

~~~~python
        # TMEM allocation
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 877-881 / 第 877-881 行

~~~~python
        tmem.allocate(self.num_tmem_alloc_cols)
        tmem.wait_for_alloc()
        # Get the pointer to the TMEM buffer
        tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
        accumulators = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 883-893 / 第 883-893 行

~~~~python
        tCrA = None
        if cutlass.const_expr(self.transform_a_source == tcgen05.OperandSource.TMEM):
            tmem_ptr_transform = cute.recast_ptr(
                accumulators.iterator + self.num_acc_tmem_cols, dtype=self.mma_dtype
            )
            tCrA = cute.make_tensor(
                tmem_ptr_transform,
                tiled_mma.make_fragment_A(a_smem_layout_transform.outer).layout,
            )
        else:
            tCrA = tiled_mma.make_fragment_A(sA_transform)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 895-895 / 第 895-895 行

~~~~python
        # Schedule warp
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 896-923 / 第 896-923 行

~~~~python
        if warp_idx == self.schedule_warp_id:
            cute.arch.setmaxregister_decrease(self.num_regs_schedule_warp)
            # Persistent tile scheduling loop
            tile_sched = utils.StaticPersistentRuntimeTileScheduler.create(
                tile_sched_params,
                (bidx, bidy, bidz),
                cute.arch.grid_dim(),
                inner_mode=0,
            )
            work_tile = tile_sched.initial_work_tile_info()
            tile_info_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_tile_info_stage
            )
            # Create initial group search state
            search_state = (
                mixed_input_utils.create_initial_contiguous_group_search_state()
            )
            not_last_tile = cutlass.Boolean(1)
            while not_last_tile:
                tile_info_pipeline.producer_acquire(tile_info_producer_state)
                cluster_tile_coord_mnl = work_tile.tile_idx
                cta_tile_coord_m = (
                    cluster_tile_coord_mnl[0] * self.cluster_shape_mn[0]
                    + block_in_cluster_coord_vmnk[1] * cute.size(tiled_mma.thr_id.shape)
                    + block_in_cluster_coord_vmnk[0]
                )
                cta_tile_offset_n = block_in_cluster_coord_vmnk[2]
                search_state = mixed_input_utils.contiguous_group_search(
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 924-951 / 第 924-951 行

~~~~python
                    self.cluster_tile_shape_mnk,
                    group_count,
                    cluster_tile_coord_mnl[1],
                    search_state,
                    cumsum,
                    1,  # mode index to perform the search. 0 for M and 1 for N
                )
                cur_sTile_info = sTile_info[(None, tile_info_producer_state.index)]
                not_last_tile = search_state.cur_group_idx <= group_count
                # Store tile info into shared memory buffer
                with cute.arch.elect_one():
                    cur_sTile_info[0] = cta_tile_coord_m
                    cur_sTile_info[1] = (
                        search_state.cur_start
                        + cta_tile_offset_n * self.cta_tile_shape_mnk[1]
                    )
                    cur_sTile_info[2] = search_state.cur_group_idx - 1
                    cur_sTile_info[3] = (
                        search_state.cur_boundary
                        - search_state.cur_start
                        - (cta_tile_offset_n * self.cta_tile_shape_mnk[1])
                    )
                # Fence and barrier to ensure tile info store has finished
                cute.arch.fence_proxy(
                    "async.shared",
                    space="cta",
                )
                self.sched_sync_barrier.arrive_and_wait()
~~~~

**EN**: Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 952-958 / 第 952-958 行

~~~~python
                # Commit tile info pipeline
                tile_info_pipeline.producer_commit(tile_info_producer_state)
                # Advance to next tile
                tile_info_producer_state.advance()
                tile_sched.advance_to_next_work()
                work_tile = tile_sched.get_current_work()
            tile_info_pipeline.producer_tail(tile_info_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 960-960 / 第 960-960 行

~~~~python
        # Specialized TMA load warp for A/B tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 961-982 / 第 961-982 行

~~~~python
        if warp_idx == self.tma_warp_id:
            cute.arch.setmaxregister_decrease(self.num_regs_tma_warps)
            # Persistent tile scheduling loop
            tile_info_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_tile_info_stage
            )
            tile_info_pipeline.consumer_wait(tile_info_consumer_state)
            work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                group_count, sTile_info[(None, tile_info_consumer_state.index)]
            )
            cute.arch.fence_proxy(
                "async.shared",
                space="cta",
            )
            tile_info_pipeline.consumer_release(tile_info_consumer_state)
            tile_info_consumer_state.advance()
            a_load2trans_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_load2trans_stage
            )
            b_load2mma_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_load2trans_stage
            )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 984-1008 / 第 984-1008 行

~~~~python
            while work_tile.is_valid_tile:
                tAgA_slice = tAgA[
                    (
                        None,
                        work_tile.cta_coord_m // cute.size(tiled_mma.thr_id.shape),
                        None,
                        work_tile.group_idx,
                    )
                ]
                # Apply offset to B tensor based on group search result
                coord_n_offset = (
                    (work_tile.coord_n, 0, 0)
                    if cutlass.const_expr(
                        self.b_major_mode == tcgen05.OperandMajorMode.MN
                    )
                    else (0, work_tile.coord_n, 0)
                )
                tBgB_slice = cute.make_tensor(
                    (
                        tBgB.iterator[0] + coord_n_offset[0],
                        coord_n_offset[1] + tBgB.iterator[1],
                        coord_n_offset[2] + tBgB.iterator[2],
                    ),
                    cute.slice_(tBgB.layout, (None, 0, None, 0)),
                )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1010-1037 / 第 1010-1037 行

~~~~python
                a_load2trans_producer_state.reset_count()
                peek_load2trans_empty_status = cutlass.Boolean(1)
                if a_load2trans_producer_state.count < k_tile_cnt:
                    peek_load2trans_empty_status = (
                        a_load2trans_pipeline.producer_try_acquire(
                            a_load2trans_producer_state
                        )
                    )
                b_load2mma_producer_state.reset_count()
                for k_tile in cutlass.range(0, k_tile_cnt, 1, unroll=1):
                    a_load2trans_pipeline.producer_acquire(
                        a_load2trans_producer_state, peek_load2trans_empty_status
                    )
                    b_load2mma_pipeline.producer_acquire(b_load2mma_producer_state)
                    # TMA load A/B
                    cute.copy(
                        tma_atom_a,
                        tAgA_slice[(None, a_load2trans_producer_state.count)],
                        tAsA[(None, a_load2trans_producer_state.index)],
                        tma_bar_ptr=a_load2trans_pipeline.producer_get_barrier(
                            a_load2trans_producer_state
                        ),
                        mcast_mask=a_full_mcast_mask,
                    )
                    cute.copy(
                        tma_atom_b,
                        tBgB_slice[(None, b_load2mma_producer_state.count)],
                        tBsB[(None, b_load2mma_producer_state.index)],
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1038-1063 / 第 1038-1063 行

~~~~python
                        tma_bar_ptr=b_load2mma_pipeline.producer_get_barrier(
                            b_load2mma_producer_state
                        ),
                        mcast_mask=b_full_mcast_mask,
                    )
                    a_load2trans_pipeline.producer_commit(a_load2trans_producer_state)
                    b_load2mma_pipeline.producer_commit(b_load2mma_producer_state)
                    a_load2trans_producer_state.advance()
                    b_load2mma_producer_state.advance()
                    if a_load2trans_producer_state.count < k_tile_cnt:
                        peek_load2trans_empty_status = (
                            a_load2trans_pipeline.producer_try_acquire(
                                a_load2trans_producer_state
                            )
                        )
                # Advance to next tile
                tile_info_pipeline.consumer_wait(tile_info_consumer_state)
                work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                    group_count, sTile_info[(None, tile_info_consumer_state.index)]
                )
                cute.arch.fence_proxy(
                    "async.shared",
                    space="cta",
                )
                tile_info_pipeline.consumer_release(tile_info_consumer_state)
                tile_info_consumer_state.advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1064-1066 / 第 1064-1066 行

~~~~python
            # Wait A/B buffer empty
            a_load2trans_pipeline.producer_tail(a_load2trans_producer_state)
            b_load2mma_pipeline.producer_tail(b_load2mma_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1068-1068 / 第 1068-1068 行

~~~~python
        # Specialized TMA load for scale tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1069-1089 / 第 1069-1089 行

~~~~python
        if warp_idx == self.scale_tma_warp_id:
            cute.arch.setmaxregister_decrease(self.num_regs_tma_warps)
            if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
                # Persistent tile scheduling loop
                tile_info_consumer_state = pipeline.make_pipeline_state(
                    pipeline.PipelineUserType.Consumer, self.num_tile_info_stage
                )
                tile_info_pipeline.consumer_wait(tile_info_consumer_state)
                work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                    group_count, sTile_info[(None, tile_info_consumer_state.index)]
                )
                cute.arch.fence_proxy(
                    "async.shared",
                    space="cta",
                )
                tile_info_pipeline.consumer_release(tile_info_consumer_state)
                tile_info_consumer_state.advance()
                scale_load2accu_producer_state = pipeline.make_pipeline_state(
                    pipeline.PipelineUserType.Producer, self.num_scale_load2accu_stage
                )
                scale_k_tile_cnt = cute.size(mS_mkl.layout.shape[1][1])
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1091-1109 / 第 1091-1109 行

~~~~python
                while work_tile.is_valid_tile:
                    # ((atom_v, rest_v), RestK)
                    tSgS_slice = tSgS[
                        (
                            None,
                            work_tile.cta_coord_m // cute.size(tiled_mma.thr_id.shape),
                            None,
                            work_tile.group_idx,
                        )
                    ]
                    # Filter zeros in rest mode
                    rest_filtered = cute.filter_zeros(tSgS_slice[(0, None)].layout)
                    tSgS_slice_filtered = cute.make_tensor(
                        tSgS_slice.iterator,
                        cute.make_layout(
                            (tSgS_slice.layout[0].shape, rest_filtered.shape),
                            stride=(tSgS_slice.layout[0].stride, rest_filtered.stride),
                        ),
                    )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1111-1135 / 第 1111-1135 行

~~~~python
                    scale_load2accu_producer_state.reset_count()
                    peek_scale_load2accu_empty_status = cutlass.Boolean(1)
                    if scale_load2accu_producer_state.count < scale_k_tile_cnt:
                        peek_scale_load2accu_empty_status = (
                            scale_load2accu_pipeline.producer_try_acquire(
                                scale_load2accu_producer_state
                            )
                        )
                    for k_tile in cutlass.range(0, scale_k_tile_cnt, 1, unroll=1):
                        scale_load2accu_pipeline.producer_acquire(
                            scale_load2accu_producer_state,
                            peek_scale_load2accu_empty_status,
                        )
                        # TMA load scale
                        cute.copy(
                            tma_atom_s,
                            tSgS_slice_filtered[
                                (None, scale_load2accu_producer_state.count)
                            ],
                            tSsS[(None, scale_load2accu_producer_state.index)],
                            tma_bar_ptr=scale_load2accu_pipeline.producer_get_barrier(
                                scale_load2accu_producer_state
                            ),
                            mcast_mask=s_full_mcast_mask,
                        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1137-1157 / 第 1137-1157 行

~~~~python
                        scale_load2accu_producer_state.advance()
                        peek_scale_load2accu_empty_status = cutlass.Boolean(1)
                        if scale_load2accu_producer_state.count < scale_k_tile_cnt:
                            peek_scale_load2accu_empty_status = (
                                scale_load2accu_pipeline.producer_try_acquire(
                                    scale_load2accu_producer_state
                                )
                            )
                    # Advance to next tile
                    tile_info_pipeline.consumer_wait(tile_info_consumer_state)
                    work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                        group_count, sTile_info[(None, tile_info_consumer_state.index)]
                    )
                    cute.arch.fence_proxy(
                        "async.shared",
                        space="cta",
                    )
                    tile_info_pipeline.consumer_release(tile_info_consumer_state)
                    tile_info_consumer_state.advance()
                # Wait scale buffer empty
                scale_load2accu_pipeline.producer_tail(scale_load2accu_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1159-1159 / 第 1159-1159 行

~~~~python
        # Specialized transform warps
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1160-1187 / 第 1160-1187 行

~~~~python
        if warp_idx >= self.transform_warp_id[0]:
            cute.arch.setmaxregister_increase(self.num_regs_transform_warps)
            transform_local_tidx = tidx - 32 * self.transform_warp_id[0]
            # Partition tensors for transform input and output and set up the copy atom
            # used for loading and storing transformed A tensor
            src_copy_a, dst_copy_a, tAsA_input, tAsA_transform = (
                mixed_input_utils.transform_partition(
                    self.transform_a_source,
                    self.scale_mode,
                    copy_atom_a_input,
                    copy_atom_a_transform,
                    sA_input,
                    (
                        tCrA
                        if self.transform_a_source == tcgen05.OperandSource.TMEM
                        else sA_transform
                    ),
                    transform_local_tidx,
                )
            )
            # make fragment for input A and transformed A
            tArA_load = cute.make_rmem_tensor(
                cute.append(
                    tAsA_input[(None, None, None, None, 0)].shape,
                    self.transform_k_tile_unroll_factor,
                ),
                tAsA_input.element_type,
            )
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1189-1200 / 第 1189-1200 行

~~~~python
            tArA_transform = cute.make_rmem_tensor(
                cute.append(
                    tAsA_input[(None, None, None, None, 0)].shape,
                    self.transform_k_tile_unroll_factor,
                ),
                self.mma_dtype,
            )
            # Deduce a sub-tile size and tile tensors
            transform_tiler_size = min(
                cute.size(cute.coalesce(tAsA_input.layout), mode=[0]), 32
            )
            transform_tiler = cute.make_layout(transform_tiler_size)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1202-1226 / 第 1202-1226 行

~~~~python
            tile_info_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_tile_info_stage
            )
            tile_info_pipeline.consumer_wait(tile_info_consumer_state)
            work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                group_count, sTile_info[(None, tile_info_consumer_state.index)]
            )
            cute.arch.fence_proxy(
                "async.shared",
                space="cta",
            )
            tile_info_pipeline.consumer_release(tile_info_consumer_state)
            tile_info_consumer_state.advance()
            a_load2trans_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer,
                self.num_load2trans_stage,
            )
            a_load2trans_consumer_state0 = a_load2trans_consumer_state.clone()
            trans2mma_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer,
                self.num_trans2mma_stage,
            )
            trans2mma_producer_state0 = trans2mma_producer_state.clone()
            k_tile_cnt_unrolled2 = k_tile_cnt // self.transform_k_tile_unroll_factor
            is_tile_cnt_odd = k_tile_cnt % self.transform_k_tile_unroll_factor == 1
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1227-1254 / 第 1227-1254 行

~~~~python
            while work_tile.is_valid_tile:
                a_load2trans_consumer_state.reset_count()
                a_load2trans_consumer_state0, a_load2trans_consumer_state = (
                    self.pipeline_state_clone_and_advance(a_load2trans_consumer_state)
                )
                trans2mma_producer_state.reset_count()
                trans2mma_producer_state0, trans2mma_producer_state = (
                    self.pipeline_state_clone_and_advance(trans2mma_producer_state)
                )
                for k_tile in cutlass.range(0, k_tile_cnt_unrolled2, 1, unroll=1):
                    tAsA_input_slice0, tAsA_input_slice1 = (
                        self.slice_and_divide_with_index_pair(
                            tAsA_input,
                            (
                                a_load2trans_consumer_state0.index,
                                a_load2trans_consumer_state.index,
                            ),
                            transform_tiler,
                        )
                    )
                    # reg buffer0 and buffer1 for A_load
                    tArA_load_slice0, tArA_load_slice1 = (
                        self.slice_and_divide_with_index_pair(
                            tArA_load,
                            (0, 1),
                            transform_tiler,
                        )
                    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1255-1270 / 第 1255-1270 行

~~~~python
                    # reg buffer0 and buffer1 for A_transform
                    tArA_transform_buffer0 = tArA_transform[(None, None, None, None, 0)]
                    tArA_transform_buffer1 = tArA_transform[(None, None, None, None, 1)]
                    tArA_transform_slice0 = self.divide_tensor_by_tiler(
                        tArA_transform_buffer0,
                        transform_tiler,
                    )
                    tArA_transform_slice1 = self.divide_tensor_by_tiler(
                        tArA_transform_buffer1,
                        transform_tiler,
                    )
                    # Check if input A data are ready
                    a_load2trans_pipeline.consumer_wait(a_load2trans_consumer_state0)
                    a_load2trans_pipeline.consumer_wait(a_load2trans_consumer_state)
                    trans2mma_pipeline.producer_acquire(trans2mma_producer_state)
                    # Transformation in buffer0
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1271-1298 / 第 1271-1298 行

~~~~python
                    for idx in cutlass.range_constexpr(
                        cute.size(tArA_load_slice0, mode=[1])
                    ):
                        # Load A from shared memory
                        cute.autovec_copy(
                            tAsA_input_slice0[(None, idx)],
                            tArA_load_slice0[(None, idx)],
                        )
                        # Convert it to mma dtype
                        tensor_transformed = mixed_input_utils.cvt_tensor_a(
                            tArA_load_slice0[(None, idx)],
                            self.mma_dtype,
                            self.shuffle_a,
                        )
                        # Load A from shared memory
                        cute.autovec_copy(
                            tAsA_input_slice1[(None, idx)],
                            tArA_load_slice1[(None, idx)],
                        )
                        tArA_transform_slice0[(None, idx)].store(tensor_transformed)
                    # Store transformed A to tensor memory or shared memory
                    mixed_input_utils.store_transformed_a(
                        tArA_transform_buffer0,
                        tAsA_transform[
                            (None, None, None, None, trans2mma_producer_state0.index)
                        ],
                        dst_copy_a,
                    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1299-1320 / 第 1299-1320 行

~~~~python
                    if cutlass.const_expr(
                        self.transform_a_source == tcgen05.OperandSource.TMEM
                    ):
                        cute.arch.fence_view_async_tmem_store()
                    else:
                        cute.arch.fence_proxy(
                            "async.shared",
                            space="cta",
                        )
                    # Signal the completion of transformation in buffer0
                    trans2mma_pipeline.producer_commit(trans2mma_producer_state0)
                    a_load2trans_pipeline.consumer_release(a_load2trans_consumer_state0)
                    for idx in cutlass.range_constexpr(
                        cute.size(tArA_load_slice1, mode=[1])
                    ):
                        # Convert it to mma dtype
                        tensor_transformed = mixed_input_utils.cvt_tensor_a(
                            tArA_load_slice1[(None, idx)],
                            self.mma_dtype,
                            self.shuffle_a,
                        )
                        tArA_transform_slice1[(None, idx)].store(tensor_transformed)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1321-1343 / 第 1321-1343 行

~~~~python
                    # Store transformed A to tensor memory or shared memory
                    mixed_input_utils.store_transformed_a(
                        tArA_transform_buffer1,
                        tAsA_transform[
                            (None, None, None, None, trans2mma_producer_state.index)
                        ],
                        dst_copy_a,
                    )
                    if cutlass.const_expr(
                        self.transform_a_source == tcgen05.OperandSource.TMEM
                    ):
                        cute.arch.fence_view_async_tmem_store()
                    else:
                        cute.arch.fence_proxy(
                            "async.shared",
                            space="cta",
                        )
                    # Signal the completion of transformation
                    trans2mma_pipeline.producer_commit(trans2mma_producer_state)
                    trans2mma_producer_state.advance()
                    trans2mma_producer_state0, trans2mma_producer_state = (
                        self.pipeline_state_clone_and_advance(trans2mma_producer_state)
                    )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1345-1372 / 第 1345-1372 行

~~~~python
                    a_load2trans_pipeline.consumer_release(a_load2trans_consumer_state)
                    a_load2trans_consumer_state.advance()
                    a_load2trans_consumer_state0, a_load2trans_consumer_state = (
                        self.pipeline_state_clone_and_advance(
                            a_load2trans_consumer_state
                        )
                    )
                # Handle the last tile if needed
                if is_tile_cnt_odd:
                    tAsA_input_slice = tAsA_input[
                        (None, None, None, None, a_load2trans_consumer_state0.index)
                    ]
                    tAsA_input_slice = self.divide_tensor_by_tiler(
                        tAsA_input_slice, transform_tiler
                    )
                    tArA_load_slice = tArA_load[(None, None, None, None, 0)]
                    tArA_load_slice = self.divide_tensor_by_tiler(
                        tArA_load_slice, transform_tiler
                    )
                    tArA_transform_buffer = tArA_transform[(None, None, None, None, 0)]
                    tArA_transform_slice = self.divide_tensor_by_tiler(
                        tArA_transform_buffer, transform_tiler
                    )
                    a_load2trans_pipeline.consumer_wait(a_load2trans_consumer_state0)
                    trans2mma_pipeline.producer_acquire(trans2mma_producer_state0)
                    for idx in cutlass.range_constexpr(
                        cute.size(tArA_load_slice, mode=[1])
                    ):
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1373-1397 / 第 1373-1397 行

~~~~python
                        # Load A from shared memory
                        cute.autovec_copy(
                            tAsA_input_slice[(None, idx)],
                            tArA_load_slice[(None, idx)],
                        )
                        # Convert it to mma dtype
                        tensor_transformed = mixed_input_utils.cvt_tensor_a(
                            tArA_load_slice[(None, idx)],
                            self.mma_dtype,
                            self.shuffle_a,
                        )
                        tArA_transform_slice[(None, idx)].store(tensor_transformed)
                    a_load2trans_pipeline.consumer_release(a_load2trans_consumer_state0)
                    # Store transformed A to tensor memory or shared memory
                    mixed_input_utils.store_transformed_a(
                        tArA_transform_buffer,
                        tAsA_transform[
                            (None, None, None, None, trans2mma_producer_state0.index)
                        ],
                        dst_copy_a,
                    )
                    if cutlass.const_expr(
                        self.transform_a_source == tcgen05.OperandSource.TMEM
                    ):
                        cute.arch.fence_view_async_tmem_store()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1398-1422 / 第 1398-1422 行

~~~~python
                    else:
                        cute.arch.fence_proxy(
                            "async.shared",
                            space="cta",
                        )
                    # Signal the completion of transformation
                    trans2mma_pipeline.producer_commit(trans2mma_producer_state0)
                    trans2mma_producer_state0.advance()
                    a_load2trans_consumer_state0.advance()
                # Keep pipeline state ready for next available buffer
                trans2mma_producer_state = trans2mma_producer_state0.clone()
                a_load2trans_consumer_state = a_load2trans_consumer_state0.clone()
                # Advance to next tile
                tile_info_pipeline.consumer_wait(tile_info_consumer_state)
                work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                    group_count, sTile_info[(None, tile_info_consumer_state.index)]
                )
                cute.arch.fence_proxy(
                    "async.shared",
                    space="cta",
                )
                tile_info_pipeline.consumer_release(tile_info_consumer_state)
                tile_info_consumer_state.advance()
            # Wait a_transform buffer empty
            trans2mma_pipeline.producer_tail(trans2mma_producer_state0)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1424-1424 / 第 1424-1424 行

~~~~python
        # Specialized MMA warp
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1425-1452 / 第 1425-1452 行

~~~~python
        if warp_idx == self.mma_warp_id:
            cute.arch.setmaxregister_decrease(self.num_regs_mma_warp)
            tCtAcc_base = accumulators
            # Persistent tile scheduling loop
            tile_info_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_tile_info_stage
            )
            tile_info_pipeline.consumer_wait(tile_info_consumer_state)
            work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                group_count, sTile_info[(None, tile_info_consumer_state.index)]
            )
            cute.arch.fence_proxy(
                "async.shared",
                space="cta",
            )
            tile_info_pipeline.consumer_release(tile_info_consumer_state)
            tile_info_consumer_state.advance()
            trans2mma_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_trans2mma_stage
            )
            b_load2mma_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_load2trans_stage
            )
            acc_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_acc_stage
            )
            while work_tile.is_valid_tile:
                b_load2mma_consumer_state.reset_count()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1453-1480 / 第 1453-1480 行

~~~~python
                trans2mma_consumer_state.reset_count()
                peek_trans2mma_full_status = cutlass.Boolean(1)
                if is_leader_cta:
                    if trans2mma_consumer_state.count < k_tile_cnt:
                        peek_trans2mma_full_status = (
                            trans2mma_pipeline.consumer_try_wait(
                                trans2mma_consumer_state
                            )
                        )
                    k_block_cnt = cute.ceil_div(k_tile_cnt, num_k_tiles_per_scale)
                    num_k_tiles_executed = 0
                    # Loop over K blocks with different scales and commit
                    # before starting the next tile requiring new scales
                    for k_block in cutlass.range(0, k_block_cnt, 1, unroll=1):
                        # (MMA, MMA_M, MMA_N)
                        tCtAcc = tCtAcc_base[
                            (None, None, None, acc_producer_state.index)
                        ]
                        acc_pipeline.producer_acquire(acc_producer_state)
                        cur_num_k_tiles = min(
                            num_k_tiles_per_scale, k_tile_cnt - num_k_tiles_executed
                        )
                        tiled_mma.set(tcgen05.Field.ACCUMULATE, False)
                        for k_tile in cutlass.range(0, cur_num_k_tiles, 1, unroll=1):
                            trans2mma_pipeline.consumer_wait(
                                trans2mma_consumer_state, peek_trans2mma_full_status
                            )
                            b_load2mma_pipeline.consumer_wait(b_load2mma_consumer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1481-1496 / 第 1481-1496 行

~~~~python
                            num_kslices = cute.size(tCrA, mode=[2])
                            for kslice_idx in cutlass.range(
                                num_kslices, unroll_full=True
                            ):
                                kblock_coord_a = (
                                    None,
                                    None,
                                    kslice_idx,
                                    trans2mma_consumer_state.index,
                                )
                                kblock_coord_b = (
                                    None,
                                    None,
                                    kslice_idx,
                                    b_load2mma_consumer_state.index,
                                )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1498-1525 / 第 1498-1525 行

~~~~python
                                cute.gemm(
                                    tiled_mma,
                                    tCtAcc,
                                    tCrA[kblock_coord_a],
                                    tCrB[kblock_coord_b],
                                    tCtAcc,
                                )
                                # Enable accumulate on tCtAcc after first kblock
                                tiled_mma.set(tcgen05.Field.ACCUMULATE, True)
                            trans2mma_pipeline.consumer_release(
                                trans2mma_consumer_state
                            )
                            b_load2mma_pipeline.consumer_release(
                                b_load2mma_consumer_state
                            )
                            trans2mma_consumer_state.advance()
                            b_load2mma_consumer_state.advance()
                            peek_trans2mma_full_status = cutlass.Boolean(1)
                            if trans2mma_consumer_state.count < k_tile_cnt:
                                peek_trans2mma_full_status = (
                                    trans2mma_pipeline.consumer_try_wait(
                                        trans2mma_consumer_state
                                    )
                                )
                        num_k_tiles_executed += cur_num_k_tiles
                        # Async arrive accumulator buffer full
                        acc_pipeline.producer_commit(acc_producer_state)
                        acc_producer_state.advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1527-1527 / 第 1527-1527 行

~~~~python
                # Advance to next tile
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1528-1539 / 第 1528-1539 行

~~~~python
                tile_info_pipeline.consumer_wait(tile_info_consumer_state)
                work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                    group_count, sTile_info[(None, tile_info_consumer_state.index)]
                )
                cute.arch.fence_proxy(
                    "async.shared",
                    space="cta",
                )
                tile_info_pipeline.consumer_release(tile_info_consumer_state)
                tile_info_consumer_state.advance()
            # Wait for accumulator buffer empty
            acc_pipeline.producer_tail(acc_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1541-1541 / 第 1541-1541 行

~~~~python
        # Specialized acc update and epilogue warps
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1542-1569 / 第 1542-1569 行

~~~~python
        if warp_idx < self.mma_warp_id:
            cute.arch.setmaxregister_increase(self.num_regs_epilogue_warps)
            epi_tidx = tidx
            tCtAcc_base = accumulators
            # Construct scale tensor view as C
            scale_view_as_C_layout = cute.make_layout(
                (
                    scale_smem_layout.outer[0].shape,
                    self.cta_tile_shape_mnk[1],
                    scale_smem_layout.outer[2].shape,
                ),
                stride=(
                    scale_smem_layout.outer[0].stride,
                    0,
                    scale_smem_layout.outer[2].stride,
                ),
            )
            scale_view_as_C = cute.make_tensor(
                sS_input.iterator,
                scale_view_as_C_layout,
            )
            # Partition for epilogue and accumulator update
            tiled_copy_t2r, tTR_tAcc_base, tTR_rAcc, tTR_rAcc_final, tTR_sScale = (
                self.epilog_and_acc_update_tmem_copy_and_partition(
                    epi_tidx,
                    tCtAcc_base,
                    tCgC,
                    scale_view_as_C,
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1570-1573 / 第 1570-1573 行

~~~~python
                    epi_tile,
                    use_2cta_instrs,
                )
            )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1575-1598 / 第 1575-1598 行

~~~~python
            tTR_rC = cute.make_rmem_tensor(tTR_rAcc.shape, self.c_dtype)
            tiled_copy_r2s, tRS_rC, tRS_sC = (
                mixed_input_utils.epilog_smem_copy_and_partition(
                    self.c_layout,
                    self.c_dtype,
                    self.acc_dtype,
                    tiled_copy_t2r,
                    tTR_rC,
                    epi_tidx,
                    sC,
                )
            )
            (tma_atom_c, bSG_sC, bSG_gC_partitioned, simt_atom, tTR_gC_partitioned) = (
                mixed_input_utils.epilog_gmem_copy_and_partition(
                    self.c_dtype,
                    epi_tidx,
                    tma_atom_c,
                    tiled_copy_t2r,
                    tCgC,
                    tCgC_simt,
                    epi_tile,
                    sC,
                )
            )
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1600-1600 / 第 1600-1600 行

~~~~python
            # Predicates
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1601-1607 / 第 1601-1607 行

~~~~python
            thr_mapping = cute.make_identity_tensor(
                (self.cta_tile_shape_mnk[0], self.cta_tile_shape_mnk[1])
            )
            thr_mapping_mn = cute.flat_divide(thr_mapping, epi_tile)
            thr_copy_t2r = tiled_copy_t2r.get_slice(epi_tidx)
            m_thr_offset = thr_copy_t2r.partition_D(thr_mapping_mn)
            m_thr_offset = cute.group_modes(m_thr_offset, 3, cute.rank(m_thr_offset))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 1609-1615 / 第 1609-1615 行

~~~~python
            acc_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_acc_stage
            )
            scale_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer,
                self.num_scale_load2accu_stage,
            )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1617-1640 / 第 1617-1640 行

~~~~python
            c_producer_group = pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                32 * len(self.epilog_warp_id),
            )
            c_pipeline = pipeline.PipelineTmaStore.create(
                num_stages=self.num_c_stage,
                producer_group=c_producer_group,
            )
            # Persistent tile scheduling loop
            tile_info_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_tile_info_stage
            )
            tile_info_pipeline.consumer_wait(tile_info_consumer_state)
            work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                group_count, sTile_info[(None, tile_info_consumer_state.index)]
            )
            cute.arch.fence_proxy(
                "async.shared",
                space="cta",
            )
            tile_info_pipeline.consumer_release(tile_info_consumer_state)
            tile_info_consumer_state.advance()
            num_prev_subtiles = cutlass.Int32(0)
            scale_k_tile_cnt = cute.size(mS_mkl.layout.shape[1][1])
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1641-1668 / 第 1641-1668 行

~~~~python
            while work_tile.is_valid_tile:
                # perform accumulator update with scales
                tTR_rAcc_final.fill(0.0)
                tTR_rScale = cute.make_rmem_tensor(
                    cute.slice_(tTR_sScale, (None, None, None, 0, None, 0)).shape,
                    self.a_scale_dtype,
                )
                scale_consumer_state.reset_count()
                peek_scale_full_status = cutlass.Boolean(1)
                if scale_consumer_state.count < scale_k_tile_cnt:
                    peek_scale_full_status = scale_load2accu_pipeline.consumer_try_wait(
                        scale_consumer_state
                    )
                acc_consumer_state.reset_count()
                peek_acc_full_status = cutlass.Boolean(1)
                if acc_consumer_state.count < scale_k_tile_cnt:
                    peek_acc_full_status = acc_pipeline.consumer_try_wait(
                        acc_consumer_state
                    )
                for k_tile in cutlass.range(0, scale_k_tile_cnt, 1, unroll=1):
                    tTR_tAcc = tTR_tAcc_base[
                        (None, None, None, None, None, acc_consumer_state.index)
                    ]
                    tTR_sScale_slice = cute.slice_(
                        tTR_sScale,
                        (None, None, None, 0, None, scale_consumer_state.index),
                    )
                    scale_load2accu_pipeline.consumer_wait(
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1669-1695 / 第 1669-1695 行

~~~~python
                        scale_consumer_state, peek_scale_full_status
                    )
                    cute.autovec_copy(tTR_sScale_slice, tTR_rScale)
                    acc_pipeline.consumer_wait(acc_consumer_state, peek_acc_full_status)
                    tTR_tAcc = cute.group_modes(tTR_tAcc, 3, cute.rank(tTR_tAcc))
                    subtile_cnt = cute.size(tTR_tAcc.shape, mode=[3])
                    for subtile_idx in cutlass.range_constexpr(subtile_cnt):
                        tTR_tAcc_mn = tTR_tAcc[(None, None, None, subtile_idx)]
                        cute.copy(tiled_copy_t2r, tTR_tAcc_mn, tTR_rAcc)
                        tTR_rAcc_subtile = tTR_rAcc_final[
                            (None, None, None, subtile_idx)
                        ]
                        tTR_rScale_subtile = tTR_rScale[(None, None, None, subtile_idx)]
                        acc_vec = tTR_rAcc.load()
                        final_vec = tTR_rAcc_subtile.load()
                        scale = tTR_rScale_subtile.load().to(self.acc_dtype)
                        final_vec = acc_vec * scale + final_vec
                        tTR_rAcc_subtile.store(final_vec)
                    scale_load2accu_pipeline.consumer_release(scale_consumer_state)
                    scale_consumer_state.advance()
                    peek_scale_full_status = cutlass.Boolean(1)
                    if scale_consumer_state.count < scale_k_tile_cnt:
                        peek_scale_full_status = (
                            scale_load2accu_pipeline.consumer_try_wait(
                                scale_consumer_state
                            )
                        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1696-1723 / 第 1696-1723 行

~~~~python
                    with cute.arch.elect_one():
                        acc_pipeline.consumer_release(acc_consumer_state)
                    acc_consumer_state.advance()
                    peek_acc_full_status = cutlass.Boolean(1)
                    if acc_consumer_state.count < scale_k_tile_cnt:
                        peek_acc_full_status = acc_pipeline.consumer_try_wait(
                            acc_consumer_state
                        )
                # epilogue partition
                bSG_gC = bSG_gC_partitioned[
                    (
                        None,
                        None,
                        None,
                        work_tile.cta_coord_m // cute.size(tiled_mma.thr_id.shape),
                        0,
                        0,
                    )
                ]
                tma_store_offset_coord = (
                    (work_tile.coord_n, 0, 0)
                    if cutlass.const_expr(self.c_layout.is_n_major_c())
                    else (0, work_tile.coord_n, 0)
                )
                bSG_gC = cute.make_tensor(
                    (
                        tma_store_offset_coord[0] + bSG_gC.iterator[0],
                        tma_store_offset_coord[1] + bSG_gC.iterator[1],
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1724-1745 / 第 1724-1745 行

~~~~python
                        tma_store_offset_coord[2] + bSG_gC.iterator[2],
                    ),
                    bSG_gC.layout,
                )
                tTR_gC = tTR_gC_partitioned[
                    (
                        None,
                        None,
                        None,
                        None,
                        None,
                        work_tile.cta_coord_m // cute.size(tiled_mma.thr_id.shape),
                        0,
                        0,
                    )
                ]
                tTR_gC = cute.make_tensor(
                    tTR_gC.iterator + (work_tile.coord_n * tensor_c.layout.stride[1]),
                    tTR_gC.layout,
                )
                bSG_gC = cute.group_modes(bSG_gC, 1, cute.rank(bSG_gC))
                tTR_gC = cute.group_modes(tTR_gC, 3, cute.rank(tTR_gC))
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1747-1747 / 第 1747-1747 行

~~~~python
                # Store accumulator to global memory in subtiles
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1748-1775 / 第 1748-1775 行

~~~~python
                subtile_cnt = cute.size(tTR_rAcc_final.shape, mode=[3])
                for subtile_idx in cutlass.range(subtile_cnt):
                    tTR_rAcc_subtile = tTR_rAcc_final[(None, None, None, subtile_idx)]
                    if work_tile.distance_to_boundary >= self.cta_tile_shape_mnk[1]:
                        # Convert to C type
                        acc_vec = tiled_copy_r2s.retile(tTR_rAcc_subtile).load()
                        acc_vec = acc_vec.to(self.c_dtype)
                        tRS_rC.store(acc_vec)
                        num_prev_subtiles += 1
                        c_buffer = num_prev_subtiles % self.num_c_stage
                        # Store C to shared memory
                        cute.copy(
                            tiled_copy_r2s,
                            tRS_rC,
                            tRS_sC[(None, None, None, c_buffer)],
                        )
                        # Fence and barrier to make sure shared memory store is visible to TMA store
                        cute.arch.fence_proxy(
                            "async.shared",
                            space="cta",
                        )
                        self.epilog_sync_barrier.arrive_and_wait()
                        # TMA store C to global memory
                        if warp_idx == self.epilog_warp_id[0]:
                            cute.copy(
                                tma_atom_c,
                                bSG_sC[(None, c_buffer)],
                                bSG_gC[(None, subtile_idx)],
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1776-1803 / 第 1776-1803 行

~~~~python
                            )
                            c_pipeline.producer_commit()
                            c_pipeline.producer_acquire()
                        self.epilog_sync_barrier.arrive_and_wait()
                    else:
                        # Convert to C type
                        acc_vec = tTR_rAcc_subtile.load()
                        acc_vec = acc_vec.to(self.c_dtype)
                        tTR_rC.store(acc_vec)
                        # Compute predicate for SIMT store
                        tCpC = cute.make_rmem_tensor(
                            cute.make_layout(tTR_rC.shape),
                            cutlass.Boolean,
                        )
                        m_thr_slice = m_thr_offset[(None, None, None, subtile_idx)]
                        for i in cutlass.range(cute.size(tCpC), unroll_full=True):
                            tCpC[i] = (
                                m_thr_slice[(i)][0]
                                + work_tile.cta_coord_m * self.cta_tile_shape_mnk[0]
                                < tensor_c.shape[0]
                            ) and (m_thr_slice[(i)][1] < work_tile.distance_to_boundary)
                        # Store C to global memory
                        cute.copy(
                            simt_atom,
                            cute.flatten(tTR_rC),
                            cute.flatten(tTR_gC[(None, None, None, subtile_idx)]),
                            pred=cute.flatten(tCpC),
                        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 1804-1814 / 第 1804-1814 行

~~~~python
                # Advance to next tile
                tile_info_pipeline.consumer_wait(tile_info_consumer_state)
                work_tile = mixed_input_utils.make_contiguous_group_work_tile_info(
                    group_count, sTile_info[(None, tile_info_consumer_state.index)]
                )
                cute.arch.fence_proxy(
                    "async.shared",
                    space="cta",
                )
                tile_info_pipeline.consumer_release(tile_info_consumer_state)
                tile_info_consumer_state.advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1816-1816 / 第 1816-1816 行

~~~~python
            # Dealloc the tensor memory buffer
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1817-1820 / 第 1817-1820 行

~~~~python
            tmem.relinquish_alloc_permit()
            self.epilog_sync_barrier.arrive_and_wait()
            tmem.free(tmem_ptr)
            c_pipeline.producer_tail()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1822-1831 / 第 1822-1831 行

~~~~python
    def divide_tensor_by_tiler(
        self, tensor: cute.Tensor, transform_tiler: cute.Layout
    ) -> cute.Tensor:
        """
        Divide the input tensor by given tiler and organize the resulting layout to 2 modes.
        The first mode is the tile mode and the second mode is the rest mode.
        """
        divided_tensor = cute.flat_divide(tensor, transform_tiler)
        divided_tensor = cute.group_modes(divided_tensor, 1, cute.rank(divided_tensor))
        return divided_tensor
~~~~

**EN**: Defines `divide_tensor_by_tiler`, grouping related logic behind a named Python callable so the example stays modular and reusable. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Returns the constructed object or computed result to the caller.
**CN**: 定义 `divide_tensor_by_tiler`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 把构造好的对象或计算结果返回给调用方。

### Lines 1833-1850 / 第 1833-1850 行

~~~~python
    def slice_and_divide_with_index_pair(
        self,
        tensor: cute.Tensor,
        index_pair: tuple[cutlass.Int32, cutlass.Int32],
        tiler: cute.Layout,
        slice_mode=4,
    ) -> tuple[cute.Tensor, cute.Tensor]:
        """
        Perform the slice and divide_tensor_by_tiler operation on the sliced tensor with index pair. Coords used for slice are
        """
        # pad None before the slice_mode
        tensor0_slice, tensor1_slice = (
            tensor[(None,) * slice_mode + (index_pair[0],)],
            tensor[(None,) * slice_mode + (index_pair[1],)],
        )
        return self.divide_tensor_by_tiler(
            tensor0_slice, tiler
        ), self.divide_tensor_by_tiler(tensor1_slice, tiler)
~~~~

**EN**: Defines `slice_and_divide_with_index_pair`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `slice_and_divide_with_index_pair`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 1852-1860 / 第 1852-1860 行

~~~~python
    def pipeline_state_clone_and_advance(
        self, pipeline_state: pipeline.PipelineState
    ) -> tuple[pipeline.PipelineState, pipeline.PipelineState]:
        """
        Clones the pipeline state and advances it.
        """
        pipeline_state_clone = pipeline_state.clone()
        pipeline_state.advance()
        return pipeline_state_clone, pipeline_state
~~~~

**EN**: Defines `pipeline_state_clone_and_advance`, grouping related logic behind a named Python callable so the example stays modular and reusable. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Returns the constructed object or computed result to the caller.
**CN**: 定义 `pipeline_state_clone_and_advance`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 把构造好的对象或计算结果返回给调用方。

### Lines 1862-1889 / 第 1862-1889 行

~~~~python
    def epilog_and_acc_update_tmem_copy_and_partition(
        self,
        tidx: cutlass.Int32,
        tAcc: cute.Tensor,
        gC_mnl: cute.Tensor,
        scale_tensor: cute.Tensor,
        epi_tile: cute.Tile,
        use_2cta_instrs: Union[cutlass.Boolean, bool],
    ) -> tuple[cute.TiledCopy, cute.Tensor, cute.Tensor, cute.Tensor, cute.Tensor]:
        """
        Partitions source and destination tensors for a tensor memory load together with
        the scale and accumulator tensors for the accumulator update.
        """
        # Make tiledCopy for tensor memory load
        copy_atom_t2r = sm100_utils.get_tmem_load_op(
            self.cta_tile_shape_mnk,
            self.c_layout,
            self.c_dtype,
            self.acc_dtype,
            epi_tile,
            use_2cta_instrs,
        )
        # (EPI_TILE_M, EPI_TILE_N, EPI_M, EPI_N, STAGE)
        tAcc_epi = cute.flat_divide(
            tAcc[((None, None), 0, 0, None)],
            epi_tile,
        )
        # (EPI_TILE_M, EPI_TILE_N)
~~~~

**EN**: Defines `epilog_and_acc_update_tmem_copy_and_partition`, grouping related logic behind a named Python callable so the example stays modular and reusable. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `epilog_and_acc_update_tmem_copy_and_partition`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1890-1892 / 第 1890-1892 行

~~~~python
        tiled_copy_t2r = tcgen05.make_tmem_copy(
            copy_atom_t2r, tAcc_epi[(None, None, 0, 0, 0)]
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1894-1896 / 第 1894-1896 行

~~~~python
        thr_copy_t2r = tiled_copy_t2r.get_slice(tidx)
        # (T2R, T2R_M, T2R_N, EPI_M, EPI_M, STAGE)
        tTR_tAcc = thr_copy_t2r.partition_S(tAcc_epi)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1898-1898 / 第 1898-1898 行

~~~~python
        # (EPI_TILE_M, EPI_TILE_N, EPI_M, EPI_N, loopM, loopN, loopL)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1899-1917 / 第 1899-1917 行

~~~~python
        gC_mnl_epi = cute.flat_divide(
            gC_mnl[((None, None), 0, 0, None, None, None)], epi_tile
        )
        sScale_epi = cute.flat_divide(scale_tensor, epi_tile)
        # (T2R, T2R_M, T2R_N, EPI_M, EPI_N, loopM, loopN, loopL)
        tTR_gC = thr_copy_t2r.partition_D(gC_mnl_epi)
        tTR_sScale = thr_copy_t2r.partition_D(sScale_epi)
        # (T2R, T2R_M, T2R_N)
        tTR_rAcc = cute.make_rmem_tensor(
            tTR_gC[(None, None, None, 0, 0, 0, 0, 0)].shape, self.acc_dtype
        )
        # (T2R, T2R_M, T2R_N, EPI_M, EPI_N)
        tTR_rAcc_final_ = cute.make_rmem_tensor(
            tTR_gC[(None, None, None, None, None, 0, 0, 0)].shape, self.acc_dtype
        )
        tTR_rAcc_final = cute.group_modes(
            tTR_rAcc_final_, 3, cute.rank(tTR_rAcc_final_)
        )
        return tiled_copy_t2r, tTR_tAcc, tTR_rAcc, tTR_rAcc_final, tTR_sScale
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 1919-1946 / 第 1919-1946 行

~~~~python
    @staticmethod
    def _compute_stages_and_tmem_cols(
        tiled_mma: cute.TiledMma,
        mma_tiler_mnk: tuple[int, int, int],
        cta_tile_shape_mnk: tuple[int, int, int],
        epi_tile: cute.Tile,
        a_dtype: type[cutlass.Numeric],
        b_dtype: type[cutlass.Numeric],
        c_dtype: type[cutlass.Numeric],
        c_layout: utils.LayoutEnum,
        transform_a_source: tcgen05.OperandSource,
        scale_granularity_m: int,
        scale_granularity_k: int,
        smem_buffer_align_bytes: int,
    ) -> tuple[int, int, int, int, int, int, int, int]:
        """
        Compute pipeline stages and TMEM column allocation configurations.
        """
        # Compute tmem columns required for accumulator
        acc_shape = tiled_mma.partition_shape_C(mma_tiler_mnk[:2])
        tCtAcc_stage1 = tiled_mma.make_fragment_C(cute.append(acc_shape, 1))
        num_tmem_acc_col_per_stage = cute.round_up(
            tcgen05.find_tmem_tensor_col_offset(tCtAcc_stage1), 2
        )
        # Heuristic to decide the number of stages for accumulator
        sm100_tmem_columns = cute.arch.get_max_tmem_alloc_cols("sm_100")
        accumulator_stage_count = sm100_tmem_columns // num_tmem_acc_col_per_stage
        if transform_a_source == tcgen05.OperandSource.TMEM:
~~~~

**EN**: Defines `_compute_stages_and_tmem_cols`, grouping related logic behind a named Python callable so the example stays modular and reusable. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `_compute_stages_and_tmem_cols`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1947-1962 / 第 1947-1962 行

~~~~python
            if num_tmem_acc_col_per_stage < 128:
                accumulator_stage_count = 3
            elif num_tmem_acc_col_per_stage < 256:
                accumulator_stage_count = 2
            else:
                accumulator_stage_count = 1
        # transformed A in 16bit, thus 1 tmem column could hold 2 elements
        num_elts_per_tmem_col = 32 // tiled_mma.op.a_dtype.width
        num_tmem_cols_a_per_stage = cute.round_up(
            (
                cta_tile_shape_mnk[2] // num_elts_per_tmem_col
                if transform_a_source == tcgen05.OperandSource.TMEM
                else 0
            ),
            4,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1964-1980 / 第 1964-1980 行

~~~~python
        bytes_per_pipeline_stage = 16
        # By default, we use 2 stages for tile info
        num_tile_info_stage = 2
        tile_info_bytes = (
            cute.size_in_bytes(cute.Int32, cute.make_layout((4, num_tile_info_stage)))
            + bytes_per_pipeline_stage * num_tile_info_stage
        )
        # Just keep 1 stage as tileN is small for decoding cases
        c_stage_count = 1
        c_smem_layout_staged_one = sm100_utils.make_smem_layout_epi(
            c_dtype,
            c_layout,
            epi_tile,
            1,
        )
        c_bytes_per_stage = cute.size_in_bytes(c_dtype, c_smem_layout_staged_one)
        c_bytes = c_bytes_per_stage * c_stage_count
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1982-2002 / 第 1982-2002 行

~~~~python
        smem_capacity = utils.get_smem_capacity_in_bytes("sm_100")
        # Ensure we have 4 buffers for scale tiles needed for 1 CTA tile
        a_scale_k_mode = max(cta_tile_shape_mnk[2] // scale_granularity_k, 1)
        a_scale_m_mode = max(cta_tile_shape_mnk[0] // scale_granularity_m, 1)
        scale_load2accu_stage_count = 4
        a_scale_bytes_per_stage = cute.round_up(
            cute.size_in_bytes(
                tiled_mma.op.a_dtype,
                cute.make_layout((a_scale_m_mode, a_scale_k_mode)),
            ),
            smem_buffer_align_bytes,
        )
        a_scale_bytes = (
            a_scale_bytes_per_stage + bytes_per_pipeline_stage
        ) * scale_load2accu_stage_count
        carveout_smem_bytes = (
            bytes_per_pipeline_stage * accumulator_stage_count
            + a_scale_bytes
            + c_bytes
            + tile_info_bytes
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2004-2004 / 第 2004-2004 行

~~~~python
        # Compute transform stages if A is in TMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2005-2007 / 第 2005-2007 行

~~~~python
        num_tmem_acc_cols = cute.round_up(
            accumulator_stage_count * num_tmem_acc_col_per_stage, 4
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2009-2036 / 第 2009-2036 行

~~~~python
        transform2mma_stage_count_a_source_tmem_potential = (
            (sm100_tmem_columns - num_tmem_acc_cols) // num_tmem_cols_a_per_stage
            if transform_a_source == tcgen05.OperandSource.TMEM
            else -1
        )
        if (
            transform_a_source == tcgen05.OperandSource.TMEM
            and transform2mma_stage_count_a_source_tmem_potential <= 0
        ):
            raise ValueError("Not enough TMEM capacity for selected tile size")
        a_load_bytes_per_stage = cute.round_up(
            cute.size_in_bytes(
                a_dtype,
                cute.make_layout((cta_tile_shape_mnk[0], cta_tile_shape_mnk[2])),
            ),
            smem_buffer_align_bytes,
        )
        b_load_bytes_per_stage = cute.round_up(
            cute.size_in_bytes(
                b_dtype,
                cute.make_layout(
                    (
                        cta_tile_shape_mnk[1] // cute.size(tiled_mma.thr_id),
                        cta_tile_shape_mnk[2],
                    )
                ),
            ),
            smem_buffer_align_bytes,
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 2037-2053 / 第 2037-2053 行

~~~~python
        )
        ab_load_bytes_per_stage = (
            a_load_bytes_per_stage
            + b_load_bytes_per_stage
            + 2 * bytes_per_pipeline_stage
        )
        a_transform_bytes_per_stage = (
            cute.round_up(
                cute.size_in_bytes(
                    tiled_mma.op.a_dtype,
                    cute.make_layout((cta_tile_shape_mnk[0], cta_tile_shape_mnk[2])),
                ),
                smem_buffer_align_bytes,
            )
            if transform_a_source == tcgen05.OperandSource.SMEM
            else 0
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 2055-2080 / 第 2055-2080 行

~~~~python
        a_transform_bytes_per_stage = (
            a_transform_bytes_per_stage + bytes_per_pipeline_stage
        )
        transform2mma_stage_count_a_source_smem_potential = (
            smem_capacity - carveout_smem_bytes
        ) // (ab_load_bytes_per_stage + a_transform_bytes_per_stage)
        transform2mma_stage_count = (
            min(
                transform2mma_stage_count_a_source_tmem_potential,
                transform2mma_stage_count_a_source_smem_potential,
            )
            if transform_a_source == tcgen05.OperandSource.TMEM
            else transform2mma_stage_count_a_source_smem_potential
        )
        load2transform_stage_count = (
            smem_capacity
            - carveout_smem_bytes
            - (transform2mma_stage_count * a_transform_bytes_per_stage)
        ) // ab_load_bytes_per_stage
        if (
            load2transform_stage_count < 2
            or transform2mma_stage_count < 2
            or accumulator_stage_count < 1
        ):
            raise ValueError("Not enough SMEM or TMEM capacity for selected tile size")
        num_tmem_a_cols = transform2mma_stage_count * num_tmem_cols_a_per_stage
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 2081-2088 / 第 2081-2088 行

~~~~python
        # Check if we can increase c_stage_count with leftover smem
        c_stage_count += (
            smem_capacity
            - load2transform_stage_count * ab_load_bytes_per_stage
            - transform2mma_stage_count * a_transform_bytes_per_stage
            - scale_load2accu_stage_count * a_scale_bytes_per_stage
            - c_bytes
        ) // c_bytes_per_stage
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2090-2099 / 第 2090-2099 行

~~~~python
        return (
            load2transform_stage_count,
            scale_load2accu_stage_count,
            transform2mma_stage_count,
            accumulator_stage_count,
            c_stage_count,
            num_tile_info_stage,
            num_tmem_acc_cols,
            num_tmem_a_cols,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 2101-2114 / 第 2101-2114 行

~~~~python
    @staticmethod
    def _compute_grid(
        c: cute.Tensor,
        cta_tile_shape_mnk: tuple[int, int, int],
        cluster_shape_mn: tuple[int, int],
        max_active_clusters: cutlass.Constexpr,
    ) -> tuple[utils.PersistentTileSchedulerParams, tuple[int, int, int]]:
        """
        Use persistent tile scheduler to compute the grid size for the output tensor C.
        """
        c_shape = cute.slice_(cta_tile_shape_mnk, (None, None, 0))
        gc = cute.zipped_divide(c, tiler=c_shape)
        num_ctas_mnl = gc[(0, (None, None, None))].shape
        cluster_shape_mnl = (*cluster_shape_mn, 1)
~~~~

**EN**: Defines `_compute_grid`, grouping related logic behind a named Python callable so the example stays modular and reusable. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 定义 `_compute_grid`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 2116-2119 / 第 2116-2119 行

~~~~python
        tile_sched_params = utils.PersistentTileSchedulerParams(
            num_ctas_mnl, cluster_shape_mnl
        )
        grid = (cluster_shape_mn[0], cluster_shape_mn[1], max_active_clusters)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2121-2121 / 第 2121-2121 行

~~~~python
        return tile_sched_params, grid
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2123-2140 / 第 2123-2140 行

~~~~python
    def can_implement(
        mnkl: tuple[int, int, int, int],
        a_dtype: type[cutlass.Numeric],
        b_dtype: type[cutlass.Numeric],
        c_dtype: type[cutlass.Numeric],
        a_major: str,
        b_major: str,
        c_major: str,
        scale_granularity_m: int,
        scale_granularity_k: int,
        mma_tiler: tuple[int, int, int],
        cluster_shape_mn: tuple[int, int],
        use_2cta_instrs: bool,
    ) -> bool:
        """
        Check if the kernel can be implemented for the given tensor shapes and data types.
        """
        m, n, k, l = mnkl
~~~~

**EN**: Defines `can_implement`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `can_implement`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2142-2149 / 第 2142-2149 行

~~~~python
        if not mixed_input_utils.is_valid_mma_tiler_and_cluster_shape(
            mma_tiler, cluster_shape_mn, use_2cta_instrs
        ):
            return False
        if not mixed_input_utils.is_valid_scale_granularity(
            scale_granularity_m, scale_granularity_k, a_dtype, k, mma_tiler[2]
        ):
            return False
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 2151-2151 / 第 2151-2151 行

~~~~python
        # Check tensor alignment
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2152-2156 / 第 2152-2156 行

~~~~python
        def check_contiguous_NB_alignment(
            dtype, contiguous_dim_size, expected_align_bytes
        ):
            expected_alignment = expected_align_bytes * 8 // dtype.width
            return contiguous_dim_size % expected_alignment == 0
~~~~

**EN**: Defines `check_contiguous_NB_alignment`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `check_contiguous_NB_alignment`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 2158-2163 / 第 2158-2163 行

~~~~python
        if not (
            check_contiguous_NB_alignment(a_dtype, m if a_major == "m" else k, 16)
            and check_contiguous_NB_alignment(b_dtype, n if b_major == "n" else k, 16)
        ):
            return False
        return True
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 2166-2170 / 第 2166-2170 行

~~~~python
def get_advanced_compiler_control_path():
    """
    Return the path to the advanced compiler control file of this example. If not found, return None.
    """
    import os
~~~~

**EN**: Defines `get_advanced_compiler_control_path`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `get_advanced_compiler_control_path`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2172-2174 / 第 2172-2174 行

~~~~python
    need_advanced_compiler_control = False
    try:
        from cutlass import CUDA_VERSION
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2176-2179 / 第 2176-2179 行

~~~~python
        if CUDA_VERSION.major == 13 and CUDA_VERSION.minor == 1:
            need_advanced_compiler_control = True
    except ImportError:
        pass
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2181-2190 / 第 2181-2190 行

~~~~python
    if not need_advanced_compiler_control:
        return None
    # Get the path to the advanced compiler control file
    current_dir = os.path.dirname(os.path.abspath(__file__))
    target_path = os.path.join(current_dir, "../../advanced_compiler_control/gemm0.bin")
    if os.path.exists(target_path):
        print(f"Found advanced compiler control file at {target_path}")
        return target_path
    else:
        return None
~~~~

**EN**: Returns the constructed object or computed result to the caller. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把构造好的对象或计算结果返回给调用方。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 2193-2220 / 第 2193-2220 行

~~~~python
def run(
    mnkl: tuple[int, int, int, int],
    scale_granularity_m: int,
    scale_granularity_k: int,
    a_dtype: type[cutlass.Numeric],
    b_dtype: type[cutlass.Numeric],
    c_dtype: type[cutlass.Numeric],
    acc_dtype: type[cutlass.Numeric],
    a_major: str,
    b_major: str,
    c_major: str,
    mma_tiler_mnk: tuple[int, int, int],
    cluster_shape_mn: tuple[int, int],
    use_2cta_instrs: bool,
    tolerance: float,
    warmup_iterations: int = 0,
    iterations: int = 1,
    skip_ref_check: bool = False,
    uniform_group_sizes: bool = False,
    use_cold_l2: bool = False,
    **kwargs,
) -> None:
    """
    Run the mixed-input GEMM kernel with specified parameters.

    This function creates tensors, validates parameters, executes the kernel,
    optionally compares results with a reference implementation and reports
    kernel execution time.
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2221-2222 / 第 2221-2222 行

~~~~python
    """
    m, n, k, l = mnkl
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 2224-2225 / 第 2224-2225 行

~~~~python
    if not torch.cuda.is_available():
        raise ValueError("CUDA is not available")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 2227-2227 / 第 2227-2227 行

~~~~python
    # Check if given configuration is supported
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2228-2242 / 第 2228-2242 行

~~~~python
    if not GroupedMixedInputGemmAccScaleKernel.can_implement(
        mnkl,
        a_dtype,
        b_dtype,
        c_dtype,
        a_major,
        b_major,
        c_major,
        scale_granularity_m,
        scale_granularity_k,
        mma_tiler_mnk,
        cluster_shape_mn,
        use_2cta_instrs,
    ):
        raise ValueError("GEMM configuration not supported")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 2244-2244 / 第 2244-2244 行

~~~~python
    # Get current CUDA stream from PyTorch
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2245-2247 / 第 2245-2247 行

~~~~python
    torch_stream = torch.cuda.current_stream()
    # Get the raw stream pointer as a CUstream
    current_stream = cuda.CUstream(torch_stream.cuda_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 2249-2256 / 第 2249-2256 行

~~~~python
    group_count = l
    shuffle_a = mixed_input_utils.is_shuffle_a(
        a_major, k, a_dtype, b_dtype, scale_granularity_k
    )
    # shuffle is supported since CUDA 13.1
    shuffle_supported = False
    try:
        from cutlass import CUDA_VERSION
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2258-2263 / 第 2258-2263 行

~~~~python
        if CUDA_VERSION.major > 13 or (
            CUDA_VERSION.major == 13 and CUDA_VERSION.minor >= 1
        ):
            shuffle_supported = True
    except ImportError:
        pass
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2265-2292 / 第 2265-2292 行

~~~~python
    shuffle_a = shuffle_a and shuffle_supported
    mixed_input_gemm = GroupedMixedInputGemmAccScaleKernel(
        scale_granularity_m,
        scale_granularity_k,
        acc_dtype,
        use_2cta_instrs,
        mma_tiler_mnk,
        cluster_shape_mn,
        group_count,
        shuffle_a,
    )
    torch.manual_seed(2025)
    (
        a_tensor,
        a_scale_tensor,
        b_tensor,
        cumsum_tensor,
        c_tensor,
        a_torch_cpu,
        a_scale_torch_cpu,
        b_torch_cpu,
        cumsum_torch_cpu,
        c_torch_gpu,
    ) = create_tensors(
        l,
        m,
        n,
        k,
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2293-2303 / 第 2293-2303 行

~~~~python
        a_major,
        b_major,
        c_major,
        a_dtype,
        b_dtype,
        c_dtype,
        shuffle_a,
        scale_granularity_m,
        scale_granularity_k,
        uniform_group_sizes,
    )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2305-2313 / 第 2305-2313 行

~~~~python
    max_active_clusters = utils.HardwareInfo().get_max_active_clusters(
        cluster_shape_mn[0] * cluster_shape_mn[1],
    )
    advanced_compiler_options = None
    advanced_compiler_control_path = get_advanced_compiler_control_path()
    if advanced_compiler_control_path:
        advanced_compiler_options = (
            f"--ptxas-options '--apply-controls={advanced_compiler_control_path}'"
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 2315-2325 / 第 2315-2325 行

~~~~python
    compiled_kernel = cute.compile(
        mixed_input_gemm,
        a_tensor,
        a_scale_tensor,
        b_tensor,
        cumsum_tensor,
        c_tensor,
        max_active_clusters,
        current_stream,
        options=advanced_compiler_options,
    )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2327-2344 / 第 2327-2344 行

~~~~python
    if not skip_ref_check:
        compiled_kernel(
            a_tensor,
            a_scale_tensor,
            b_tensor,
            cumsum_tensor,
            c_tensor,
            current_stream,
        )
        run_ref_and_compare(
            a_torch_cpu,
            b_torch_cpu,
            a_scale_torch_cpu,
            cumsum_torch_cpu,
            c_torch_gpu,
            c_dtype,
            tolerance,
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2346-2346 / 第 2346-2346 行

~~~~python
    # Early return if no performance measurement is needed
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2347-2348 / 第 2347-2348 行

~~~~python
    if iterations <= 0:
        return
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2350-2377 / 第 2350-2377 行

~~~~python
    def generate_tensors():
        (
            a_tensor,
            a_scale_tensor,
            b_tensor,
            cumsum_tensor,
            c_tensor,
            a_torch_cpu,
            a_scale_torch_cpu,
            b_torch_cpu,
            cumsum_torch_cpu,
            c_torch_gpu,
        ) = create_tensors(
            l,
            m,
            n,
            k,
            a_major,
            b_major,
            c_major,
            a_dtype,
            b_dtype,
            c_dtype,
            shuffle_a,
            scale_granularity_m,
            scale_granularity_k,
            uniform_group_sizes,
        )
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2378-2380 / 第 2378-2380 行

~~~~python
        return testing.JitArguments(
            a_tensor, a_scale_tensor, b_tensor, cumsum_tensor, c_tensor, current_stream
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 2382-2394 / 第 2382-2394 行

~~~~python
    workspace_count = 1
    if use_cold_l2:
        one_workspace_bytes = (
            a_torch_cpu.numel() * a_torch_cpu.element_size()
            + b_torch_cpu.numel() * b_torch_cpu.element_size()
            + c_torch_gpu.numel() * c_torch_gpu.element_size()
            + a_scale_torch_cpu.numel() * a_scale_torch_cpu.element_size()
            if a_scale_torch_cpu is not None
            else 0
        )
        workspace_count = testing.get_workspace_count(
            one_workspace_bytes, warmup_iterations, iterations
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2396-2403 / 第 2396-2403 行

~~~~python
    exec_time = testing.benchmark(
        compiled_kernel,
        workspace_generator=generate_tensors,
        workspace_count=workspace_count,
        stream=current_stream,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 2405-2405 / 第 2405-2405 行

~~~~python
    return exec_time  # Return execution time in microseconds
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2408-2408 / 第 2408-2408 行

~~~~python
if __name__ == "__main__":
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2410-2416 / 第 2410-2416 行

~~~~python
    def parse_comma_separated_ints(s: str) -> tuple[int, ...]:
        try:
            return tuple(int(x.strip()) for x in s.split(","))
        except ValueError:
            raise argparse.ArgumentTypeError(
                "Invalid format. Expected comma-separated integers."
            )
~~~~

**EN**: Defines `parse_comma_separated_ints`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 定义 `parse_comma_separated_ints`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 2418-2445 / 第 2418-2445 行

~~~~python
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--mnkl", type=parse_comma_separated_ints, default=(128, 128, 128, 1)
    )
    parser.add_argument(
        "--mma_tiler_mnk", type=parse_comma_separated_ints, default=(128, 128, 128)
    )
    parser.add_argument(
        "--cluster_shape_mn", type=parse_comma_separated_ints, default=(1, 1)
    )
    parser.add_argument(
        "--use_2cta_instrs",
        action="store_true",
        help="Enable 2CTA MMA instructions feature",
    )
    parser.add_argument(
        "--a_dtype",
        type=cutlass.dtype,
        default=cutlass.Int4,
        choices=[cutlass.Int8, cutlass.Uint8, cutlass.Int4],
    )
    parser.add_argument(
        "--b_dtype",
        type=cutlass.dtype,
        default=cutlass.BFloat16,
        choices=[cutlass.BFloat16, cutlass.Float16],
    )
    parser.add_argument("--c_dtype", type=cutlass.dtype, default=cutlass.BFloat16)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2446-2473 / 第 2446-2473 行

~~~~python
    parser.add_argument("--acc_dtype", type=cutlass.dtype, default=cutlass.Float32)
    parser.add_argument("--a_major", choices=["k", "m"], type=str, default="m")
    parser.add_argument("--b_major", choices=["k", "n"], type=str, default="k")
    parser.add_argument("--c_major", choices=["n", "m"], type=str, default="n")
    parser.add_argument(
        "--scale_granularity_m",
        type=int,
        default=1,
        help="Scale granularity along M dimension.",
    )
    parser.add_argument(
        "--scale_granularity_k",
        type=int,
        default=128,
        help="Scale granularity along K dimension.",
    )
    parser.add_argument(
        "--tolerance", type=float, default=1e-01, help="Tolerance for validation"
    )
    parser.add_argument(
        "--warmup_iterations", type=int, default=0, help="Warmup iterations"
    )
    parser.add_argument(
        "--iterations",
        type=int,
        default=1,
        help="Number of iterations to run the kernel",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2474-2501 / 第 2474-2501 行

~~~~python
    parser.add_argument(
        "--skip_ref_check", action="store_true", help="Skip reference checking"
    )
    parser.add_argument(
        "--uniform_group_sizes", action="store_true", help="Use uniform group sizes"
    )
    args = parser.parse_args()
    print(f"skip_ref_check={args.skip_ref_check}")
    run(
        args.mnkl,
        args.scale_granularity_m,
        args.scale_granularity_k,
        args.a_dtype,
        args.b_dtype,
        args.c_dtype,
        args.acc_dtype,
        args.a_major,
        args.b_major,
        args.c_major,
        args.mma_tiler_mnk,
        args.cluster_shape_mn,
        args.use_2cta_instrs,
        args.tolerance,
        args.warmup_iterations,
        args.iterations,
        args.skip_ref_check,
        args.uniform_group_sizes,
    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 2502-2502 / 第 2502-2502 行

~~~~python
    print("PASS")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

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
- `math.log2` — used by this example / 供该示例使用
- `math.ceil` — used by this example / 供该示例使用
- `typing.Union` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `sys` — used by this example / 供该示例使用
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.pipeline` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.pipeline.pipeline_init_arrive` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.pipeline.pipeline_init_wait` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.blackwell_helpers` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.mixed_input_helpers` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.mixed_input_helpers.TransformMode` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.cute.nvgpu.cpasync` — provides asynchronous copy and TMA primitives / 提供异步拷贝与 TMA 原语
- `cutlass.cute.nvgpu.tcgen05` — provides Tensor Core/TMEM helpers for Blackwell-style kernels / 提供面向 Blackwell 风格内核的 Tensor Core/TMEM 辅助工具
- `cute.blackwell.kernel.mixed_input_gemm.mixed_input_host_utils.create_tensors_for_contiguous_grouped_mixed_input_gemm` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `cute.blackwell.kernel.mixed_input_gemm.mixed_input_host_utils.run_contiguous_grouped_ref_and_compare` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `cutlass.CUDA_VERSION` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
