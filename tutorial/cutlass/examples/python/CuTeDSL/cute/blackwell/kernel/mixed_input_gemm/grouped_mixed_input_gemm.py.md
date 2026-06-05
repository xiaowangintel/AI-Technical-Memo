# grouped_mixed_input_gemm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/mixed_input_gemm/grouped_mixed_input_gemm.py`  
**Purpose / 用途**: Kernel example implementing grouped mixed input gemm with CuTeDSL. / 这是一个使用 CuTeDSL 实现 grouped mixed input gemm 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-46 / 第 29-46 行

~~~~python
import argparse
from math import log2, ceil
from typing import Optional
import os
import sys

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

### Lines 48-50 / 第 48-50 行

~~~~python
if __name__ == "__main__":
    current_dir = os.path.dirname(os.path.abspath(__file__))
    sys.path.insert(0, os.path.join(current_dir, "../../../.."))
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 52-52 / 第 52-52 行

~~~~python
from cute.blackwell.kernel.mixed_input_gemm.mixed_input_host_utils import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 53-56 / 第 53-56 行

~~~~python
    create_tensors_for_contiguous_grouped_mixed_input_gemm as create_tensors,
    run_contiguous_grouped_ref_and_compare as run_ref_and_compare,
)
from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 58-151 / 第 58-151 行

~~~~python
"""
A mixed-input grouped GEMM example for the NVIDIA Blackwell SM100 architecture using CUTE DSL.

This example demonstrates an implementation of mixed-input grouped GEMM using a TMA plus Blackwell
SM100 TensorCore warp-specialized persistent kernel. It can be viewed as an extension of the batched
mixed-input GEMM example to support a specific grouped GEMM pattern: grouped GEMM with contiguous offsets.

Specifically, the input A tensor is still in the shape of (M, K, L), and L is the number of groups. The
input B tensor is in the shape of (N, K) and the result C tensor is in the shape of (M, N). Tensor B
and tensor C are not divided into groups explicitly and there is an extra input tensor cumsum defining
the mapping between the N mode to groups. The cumsum tensor is in the shape of (N+1) and cumsum[i]
defines the accumulated size along N mode for groups up to i (not including i):

   ```
           Group 0  Group 1  Group 2 .....    Group L-1
        -+--------+--------+--------+.....+----------------+
         |        |        |        |                         |
         |<- N0 ->|<- N1 ->|<- N2 ->|.....|<--    NL-1     -->|
         |        |        |        |                         |
        -+--------+--------+--------+.....+-------------------+
cumsum:  |   0    |   N0   | N0+N1  |.....| sum(N0,N1,...NL-2) | sum(N0,N1,...NL-1)
   ```

The computation flow is the same as the batched mixed-input GEMM example. A is the narrow-precision tensor
and B holds data with a wider precision. MMA will work in the wide precision of tensor B and tensor A
will be transformed to the wide precision of tensor B following 1 of the 2 possible modes as follows:

1. convert-only mode:
    C = type_convert(A) x B

In convert-only mode, tensor A is directly converted to the wide precision of tensor B.

2. convert-scale mode:
    C = (type_convert(A) * scale) x B

In convert-scale mode, tensor A is first converted to the wide precision of tensor B and then scaled by the scale tensor.
The scale tensor is in the same precision as tensor B.
The mode is determined by tensor A's data type as follows:
- if tensor A is in int8 or uint8, convert-only mode is used.
- if tensor A is in int4, convert-scale mode is used.

The output tensor C could have the same precision as tensor B or fp32.

To run this example:

.. code-block:: bash

    python examples/blackwell/grouped_mixed_input_gemm.py      \
      --a_dtype Int8 --b_dtype BFloat16                        \
      --scale_granularity_m 0 --scale_granularity_k 0          \
      --c_dtype BFloat16 --acc_dtype Float32                   \
      --mma_tiler_mnk 128,128,64 --cluster_shape_mn 1,1        \
      --mnkl 256,512,8192,1

Input A and B have int8 and bf16 data types, respectively. The Blackwell tcgen05 MMA tile shape
is specified as (128,128,64) and the cluster shape is (1,1). The MMA accumulator and output data type
are set as fp32 and bf16, respectively. As tensor A is int8, convert-only mode is used.
scale_granularity_m and scale_granularity_k are set as 0 for convert-only mode.

Here is an example of running convert-scale mode:

.. code-block:: bash

    python examples/blackwell/mixed_input_gemm/grouped_mixed_input_gemm.py    \
      --a_dtype Int4 --b_dtype BFloat16                                       \
      --scale_granularity_m 1 --scale_granularity_k 256                       \
      --c_dtype BFloat16 --acc_dtype Float32                                  \
      --mma_tiler_mnk 256,128,128 --cluster_shape_mn 2,1                      \
      --use_2cta_instrs --mnkl 1024,8192,6144,16                              \

Input A and B have int4 and bf16 data types, respectively. The scale granularity is set as (1,256),
which means each element along the m mode of tensor A has its own scale element and 256 contiguous elements
along the k mode share the same scale element. There is no scale reuse along the L mode. If the GEMM shape is
(M, N, K, L), then the scale tensor shape is (M // scale_granularity_m, K // scale_granularity_k, L),
which is (1024, 6144/256, 16) in this example.
The Blackwell tcgen05 MMA tile shape is specified as (256,128,128) and tcgen05 2CTA feature is enabled.
The cluster shape is (2,1). The MMA accumulator and output data type are set as fp32 and bf16, respectively.
As tensor A is int4, the convert-scale mode is used.

To collect performance with NCU profiler:

.. code-block:: bash

    ncu python examples/blackwell/mixed_input_gemm/grouped_mixed_input_gemm.py    \
      --a_dtype Int8 --b_dtype BFloat16                                           \
      --scale_granularity_m 0 --scale_granularity_k 0                             \
      --c_dtype BFloat16 --acc_dtype Float32                                      \
      --mma_tiler_mnk 128,128,64 --cluster_shape_mn 1,1                           \
      --mnkl 256,512,8192,1                                                       \
      --warmup_iterations 1 --iterations 10 --skip_ref_check

Besides the requirements from the batched mixed-input GEMM example, there are some constraints for this example:
* --use_tma_store option is removed as no alignment assumption is made for each group.
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 154-180 / 第 154-180 行

~~~~python
class GroupedMixedInputGemmKernel:
    """
    Mixed-input grouped GEMM kernel for NVIDIA Blackwell SM100 architecture.

    This kernel supports GEMM operations where input tensors A and B have different
    data types, with tensor A being transformed to the precision of tensor B before
    matrix multiplication.
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

**EN**: Defines `GroupedMixedInputGemmKernel`, a reusable Python class that packages configuration and behavior for this example. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `GroupedMixedInputGemmKernel`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 182-209 / 第 182-209 行

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
            self.scale_mode = TransformMode.ConvertOnly
        else:
            self.scale_mode = TransformMode.ConvertScale
        self.group_count = group_count
        self.acc_dtype = acc_dtype
        self.use_2cta_instrs = use_2cta_instrs
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 210-233 / 第 210-233 行

~~~~python
        self.cluster_shape_mn = cluster_shape_mn
        self.mma_tiler = mma_tiler_mnk
        self.shuffle_a = shuffle_a
        self.cta_group = (
            tcgen05.CtaGroup.TWO if self.use_2cta_instrs else tcgen05.CtaGroup.ONE
        )
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
        # Schedule warp to do the group search
        self.schedule_warp_id = 7
        self.transform_warp_id = (
            8,
            9,
            10,
            11,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 234-251 / 第 234-251 行

~~~~python
        # Define expected register count for different warps
        self.num_regs_epilogue_warps = 192
        self.num_regs_mma_warp = 96
        self.num_regs_tma_warps = 80
        self.num_regs_transform_warps = 208
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

### Lines 253-253 / 第 253-253 行

~~~~python
        # Set barrier id for cta sync, epilogue sync, tmem ptr sync, and transform sync
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 254-262 / 第 254-262 行

~~~~python
        self.epilog_sync_barrier = pipeline.NamedBarrier(
            1, 32 * len(self.epilog_warp_id)
        )
        self.tmem_ptr_sync_barrier = pipeline.NamedBarrier(2, self.threads_per_cta)
        self.transform_sync_barrier = pipeline.NamedBarrier(
            3, 32 * len(self.transform_warp_id)
        )
        self.cta_sync_barrier = pipeline.NamedBarrier(4, self.threads_per_cta)
        self.sched_sync_barrier = pipeline.NamedBarrier(5, 32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 264-264 / 第 264-264 行

~~~~python
        self.smem_buffer_align_bytes = 1024
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 266-293 / 第 266-293 行

~~~~python
    def _setup_attributes(self):
        """Set up configurations that are dependent on GEMM inputs

        This method configures various attributes based on the input tensor properties
        (data types, leading dimensions) and kernel settings:
        - Deduce where the transformed A tensor is stored
        - Configuring tiled MMA
        - Computing MMA/cluster/tile shapes
        - Computing cluster layout
        - Computing multicast CTAs for A/B
        - Computing epilogue sub-tile
        - Setting up A/scale/B/C stage counts in shared memory
        - Setting up transformed A stage count in shared memory or tensor memory
        - Computing A/transformed A/scale/B/C memory layout
        - Computing tensor memory allocation columns
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
~~~~

**EN**: Defines `_setup_attributes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `_setup_attributes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 294-312 / 第 294-312 行

~~~~python
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
            cute.make_layout((*self.cluster_shape_mn, 1)),
            (tiled_mma.thr_id.shape,),
        )
        self.num_mcast_ctas_a = cute.size(self.cluster_layout_vmnk.shape[2])
        self.num_mcast_ctas_b = cute.size(self.cluster_layout_vmnk.shape[1])
        self.is_a_mcast = self.num_mcast_ctas_a > 1
        self.is_b_mcast = self.num_mcast_ctas_b > 1
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 314-319 / 第 314-319 行

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

### Lines 321-321 / 第 321-321 行

~~~~python
        # Compute tensor memory(TMEM) columns and stages for each pipeline
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 322-345 / 第 322-345 行

~~~~python
        (
            self.num_load2trans_stage,
            self.num_scale_load2trans_stage,
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
            self.scale_mode,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 347-349 / 第 347-349 行

~~~~python
        # Align TMEM columns for allocation
        # TMEM allocation requires power-of-2 column alignment
        # and must meet minimum allocation requirements
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 350-377 / 第 350-377 行

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
        # Get smem layout for scale tensor
        self.smem_layout_scale_per_stage = None
        self.smem_layout_scale = None
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 378-392 / 第 378-392 行

~~~~python
        if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
            # Get scale tile shape and smem layout for scale tensor
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
                self.num_scale_load2trans_stage,
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 394-420 / 第 394-420 行

~~~~python
    def _validate_inputs(
        self,
        a: cute.Tensor,
        a_scale: Optional[cute.Tensor],
        b: cute.Tensor,
        c: cute.Tensor,
    ) -> None:
        """
        Validates input tensors and their properties.

        :param a: Input tensor A.
        :type a: cute.Tensor
        :param a_scale: Scale tensor for tensor A (None for ConvertOnly mode).
        :type a_scale: Optional[cute.Tensor]
        :param b: Input tensor B.
        :type b: cute.Tensor
        :param c: Output tensor C.
        :type c: cute.Tensor
        :raises ValueError: If inputs don't meet kernel requirements.
        """
        # Validate scale tensor major mode
        if cutlass.const_expr(
            self.scale_mode == TransformMode.ConvertScale
            and utils.LayoutEnum.from_tensor(a_scale).mma_major_mode()
            != tcgen05.OperandMajorMode.MN
        ):
            raise ValueError("scale_major_mode must be M-major")
~~~~

**EN**: Defines `_validate_inputs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `_validate_inputs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 422-449 / 第 422-449 行

~~~~python
    @cute.jit
    def __call__(
        self,
        a: cute.Tensor,
        a_scale: Optional[cute.Tensor],  # None for ConvertOnly mode
        b: cute.Tensor,
        cumsum: cute.Tensor,
        c: cute.Tensor,
        max_active_clusters: cutlass.Constexpr,
        stream: cuda.CUstream,
    ):
        """
        Executes the Mixed Input Grouped GEMM operation.

        This method sets up the kernel parameters, computes the grid size,
        defines the shared storage, and launches the kernel.

        The execution steps are as follows:
        - Setup static attributes before smem/grid/tma computation.
        - Setup TMA load/store atoms and tensors.
        - Compute grid size with regard to hardware constraints.
        - Define shared storage for kernel.
        - Launch the kernel synchronously.

        :param a: Input tensor A.
        :type a: cute.Tensor
        :param a_scale: Scale tensor for tensor A (None for ConvertOnly mode).
        :type a_scale: Optional[cute.Tensor]
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 450-469 / 第 450-469 行

~~~~python
        :param b: Input tensor B.
        :type b: cute.Tensor
        :param cumsum: tensor containing the cumulative size of each group along the search mode(aka, N mode in this example).
        :type cumsum: cute.Tensor
        :param c: Output tensor C.
        :type c: cute.Tensor
        :param max_active_clusters: Maximum number of active clusters to launch.
        :type max_active_clusters: cutlass.Constexpr
        :param stream: CUDA stream to launch the kernel on.
        :type stream: cuda.CUstream
        """
        self.a_dtype: type[cutlass.Numeric] = a.element_type
        self.a_scale_dtype: type[cutlass.Numeric] = (
            a_scale.element_type
            if self.scale_mode is TransformMode.ConvertScale
            else None
        )
        self.b_dtype: type[cutlass.Numeric] = b.element_type
        self.c_dtype: type[cutlass.Numeric] = c.element_type
        self.mma_dtype = self.b_dtype
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 471-486 / 第 471-486 行

~~~~python
        self.a_major_mode = utils.LayoutEnum.from_tensor(a).mma_major_mode()
        self.scale_major_mode = (
            utils.LayoutEnum.from_tensor(a_scale).mma_major_mode()
            if self.scale_mode is TransformMode.ConvertScale
            else None
        )
        self.b_major_mode = utils.LayoutEnum.from_tensor(b).mma_major_mode()
        self.c_layout = utils.LayoutEnum.from_tensor(c)
        if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
            # Get gmem layout for scale tensor
            self.gmem_layout_scale = mixed_input_utils.get_gmem_layout_scale(
                a.shape,
                self.scale_granularity_m,
                self.scale_granularity_k,
                self.scale_major_mode,
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 488-488 / 第 488-488 行

~~~~python
        # Validate inputs
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 489-489 / 第 489-489 行

~~~~python
        self._validate_inputs(a, a_scale, b, c)
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 491-491 / 第 491-491 行

~~~~python
        # Setup attributes that dependent on gemm inputs
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 492-492 / 第 492-492 行

~~~~python
        self._setup_attributes()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 494-510 / 第 494-510 行

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
            self.is_a_mcast, self.use_2cta_instrs, False
        )
        b_op = mixed_input_utils.get_tma_atom_kind(
            self.is_b_mcast, self.use_2cta_instrs, True
        )
        a_scale_op = a_op
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 511-523 / 第 511-523 行

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

### Lines 525-549 / 第 525-549 行

~~~~python
        tma_atom_scale, tma_tensor_scale = None, None
        if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
            # Partition smem layout for scale tensor to make it compatible with TMA atom
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
                    cutlass.TFloat32
                    if a_scale.element_type is cutlass.Float32
                    else None
                ),
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 551-562 / 第 551-562 行

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

### Lines 564-564 / 第 564-564 行

~~~~python
        # Calculate copy size for tensor A, B, and scale
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 565-571 / 第 565-571 行

~~~~python
        a_copy_size = cute.size_in_bytes(self.a_dtype, smem_layout_a_per_stage)
        b_copy_size = cute.size_in_bytes(self.b_dtype, smem_layout_b_per_stage)
        a_scale_copy_size = (
            cute.size_in_bytes(self.a_scale_dtype, self.smem_layout_scale_per_stage)
            if self.scale_mode is TransformMode.ConvertScale
            else 0
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 573-581 / 第 573-581 行

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

### Lines 583-589 / 第 583-589 行

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

### Lines 591-618 / 第 591-618 行

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
            a_scale_load2trans_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_scale_load2trans_stage
            ]
            a_scale_load2trans_empty_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_scale_load2trans_stage
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

### Lines 619-628 / 第 619-628 行

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

### Lines 630-630 / 第 630-630 行

~~~~python
        self.shared_storage = SharedStorage
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 632-632 / 第 632-632 行

~~~~python
        # Launch kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 633-660 / 第 633-660 行

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

### Lines 661-661 / 第 661-661 行

~~~~python
        return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 663-663 / 第 663-663 行

~~~~python
    # GPU device kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 664-691 / 第 664-691 行

~~~~python
    @cute.kernel
    def kernel(
        self,
        tiled_mma: cute.TiledMma,
        tma_atom_a: cute.CopyAtom,
        mA_mkl: cute.Tensor,
        tma_atom_s: Optional[cute.CopyAtom],
        mS_mkl: Optional[cute.Tensor],
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

### Lines 692-700 / 第 692-700 行

~~~~python
        tidx, _, _ = cute.arch.thread_idx()
        bidx, bidy, bidz = cute.arch.block_idx()
        # Prefetch TMA descriptors
        if warp_idx == self.epilog_warp_id[0]:
            cpasync.prefetch_descriptor(tma_atom_a)
            cpasync.prefetch_descriptor(tma_atom_b)
            if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
                cpasync.prefetch_descriptor(tma_atom_s)
            cpasync.prefetch_descriptor(tma_atom_c)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 702-705 / 第 702-705 行

~~~~python
        use_2cta_instrs = cute.size(tiled_mma.thr_id.shape) == 2
        bidx, bidy, bidz = cute.arch.block_idx()
        # Compute how many k_tiles share the same scale
        num_k_tiles_per_scale = self.scale_granularity_k // self.cta_tile_shape_mnk[2]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 707-715 / 第 707-715 行

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

### Lines 717-718 / 第 717-718 行

~~~~python
        smem = utils.SmemAllocator()
        storage = smem.allocate(self.shared_storage)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 720-721 / 第 720-721 行

~~~~python
        # Initialize load2transform pipeline, which tracks the dependencies between TMA's loading
        # of A and B, and the transformation of A and MMA's consumption
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 722-743 / 第 722-743 行

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
        # Initialize scale_load2trans pipeline, which tracks the dependencies between TMA's loading
        # of scale, and the transformation of A
        scale_load2trans_pipeline = None
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 744-765 / 第 744-765 行

~~~~python
        if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
            num_producers_a_scale = self.num_mcast_ctas_a
            scale_load2trans_pipeline = pipeline.PipelineTmaAsync.create(
                barrier_storage=storage.a_scale_load2trans_full_mbar_ptr.data_ptr(),
                num_stages=self.num_scale_load2trans_stage,
                producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread),
                consumer_group=pipeline.CooperativeGroup(
                    pipeline.Agent.Thread,
                    num_producers_a_scale
                    * len(self.transform_warp_id)
                    * num_k_tiles_per_scale,
                ),
                tx_count=self.num_tma_load_bytes_scale,
                cta_layout_vmnk=cluster_layout_vmnk,
                tidx=transform_thread_idx,
                mcast_mode_mn=(
                    1,
                    0,
                ),  # multicast for scale_a will only happen on the M-mode
                defer_sync=True,
            )
        # Initialize transform2mma pipeline, which tracks the dependencies between the transformation
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 766-793 / 第 766-793 行

~~~~python
        # of A and MMA's consumption of transformed A
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

### Lines 794-807 / 第 794-807 行

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
        # Skip scheduler warp and TMA scale load warp when scale_mode is ConvertOnly
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 808-823 / 第 808-823 行

~~~~python
        # when computing consumer thread count
        num_tile_info_pipeline_consumer_threads = (
            self.threads_per_cta
            - 32
            - (32 if self.scale_mode is TransformMode.ConvertOnly else 0)
        )
        tile_info_pipeline = pipeline.PipelineAsync.create(
            barrier_storage=storage.tile_info_full_mbar_ptr.data_ptr(),
            num_stages=self.num_tile_info_stage,
            producer_group=pipeline.CooperativeGroup(pipeline.Agent.Thread, 32 * 1),
            consumer_group=pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                num_tile_info_pipeline_consumer_threads,
            ),
            defer_sync=True,
        )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 825-825 / 第 825-825 行

~~~~python
        # Tensor memory dealloc barrier init
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 826-832 / 第 826-832 行

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

### Lines 834-834 / 第 834-834 行

~~~~python
        # Cluster arrive after barrier init
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 835-835 / 第 835-835 行

~~~~python
        pipeline_init_arrive(cluster_shape_mn=self.cluster_shape_mn, is_relaxed=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 837-837 / 第 837-837 行

~~~~python
        # Setup smem tensor A/scale/B/C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 838-865 / 第 838-865 行

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
        sS_input = (
            smem.allocate_tensor(
                element_type=self.mma_dtype,
                layout=scale_smem_layout.outer,
                byte_alignment=self.smem_buffer_align_bytes,
                swizzle=scale_smem_layout.inner,
            )
            if self.scale_mode is TransformMode.ConvertScale
            else None
        )
        sB_input = smem.allocate_tensor(
            element_type=self.b_dtype,
            layout=b_smem_layout.outer,
            byte_alignment=self.smem_buffer_align_bytes,
            swizzle=b_smem_layout.inner,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 866-877 / 第 866-877 行

~~~~python
        sA_transform = None
        # Get smem tensor for transformed A when transform_a_source is SMEM
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

### Lines 879-879 / 第 879-879 行

~~~~python
        # Compute multicast mask for A/B buffer full
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 880-891 / 第 880-891 行

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

### Lines 893-894 / 第 893-894 行

~~~~python
        # local_tile partition global tensors
        # (bM, bK, loopM, loopK, loopL)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 895-917 / 第 895-917 行

~~~~python
        gA_mkl = cute.local_tile(
            mA_mkl, cute.slice_(self.mma_tiler, (None, 0, None)), (None, None, None)
        )
        # (bM, bK, loopM, loopK, loopL)
        gS_mkl = (
            cute.local_tile(
                mS_mkl, cute.slice_(self.mma_tiler, (None, 0, None)), (None, None, None)
            )
            if self.scale_mode is TransformMode.ConvertScale
            else None
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

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 919-919 / 第 919-919 行

~~~~python
        # Partition global tensor for TiledMMA_A/B/C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 920-933 / 第 920-933 行

~~~~python
        thr_mma = tiled_mma.get_slice(mma_tile_coord_v)
        # (MMA, MMA_M, MMA_K, loopM, loopK, loopL)
        tCgA = thr_mma.partition_A(gA_mkl)
        # (MMA, MMA_M, MMA_K, loopM, loopK, loopL)
        tCgS = (
            thr_mma.partition_A(gS_mkl)
            if self.scale_mode is TransformMode.ConvertScale
            else None
        )
        # (MMA, MMA_N, MMA_K, loopN, loopK, loopL)
        tCgB = thr_mma.partition_B(gB_nkl)
        # (MMA, MMA_M, MMA_N, loopM, loopN, loopL)
        tCgC = thr_mma.partition_C(gC_mnl)
        tCgC_simt = thr_mma.partition_C(gC_mnl_simt)
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 935-935 / 第 935-935 行

~~~~python
        # Setup copy atom to load A from shared memory for further transformation
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 936-953 / 第 936-953 行

~~~~python
        copy_atom_a_input = (
            cute.make_copy_atom(
                cute.nvgpu.CopyUniversalOp(), self.a_dtype, num_bits_per_copy=32
            )
            if self.scale_mode is TransformMode.ConvertScale
            else None
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

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 955-956 / 第 955-956 行

~~~~python
        # Partition global/shared tensor for TMA load A/B
        # TMA load A partition_S/D
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 957-968 / 第 957-968 行

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

### Lines 970-985 / 第 970-985 行

~~~~python
        tCsS = None
        tSsS = None
        tSgS = None
        if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
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

### Lines 987-987 / 第 987-987 行

~~~~python
        # TMA load B partition_S/D
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 988-999 / 第 988-999 行

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

### Lines 1001-1001 / 第 1001-1001 行

~~~~python
        # (MMA, MMA_N, MMA_K, STAGE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1002-1007 / 第 1002-1007 行

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

### Lines 1009-1009 / 第 1009-1009 行

~~~~python
        # Cluster wait before TMEM alloc and ensure pipelines are ready
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1010-1010 / 第 1010-1010 行

~~~~python
        pipeline_init_wait(cluster_shape_mn=self.cluster_shape_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1012-1012 / 第 1012-1012 行

~~~~python
        # TMEM allocation
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1013-1014 / 第 1013-1014 行

~~~~python
        tmem.allocate(self.num_tmem_alloc_cols)
        tmem.wait_for_alloc()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1016-1016 / 第 1016-1016 行

~~~~python
        # Schedule warp
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1017-1044 / 第 1017-1044 行

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

### Lines 1045-1072 / 第 1045-1072 行

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

### Lines 1073-1079 / 第 1073-1079 行

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

### Lines 1081-1081 / 第 1081-1081 行

~~~~python
        # Specialized TMA load warp for A/B tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1082-1103 / 第 1082-1103 行

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

### Lines 1105-1129 / 第 1105-1129 行

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

### Lines 1131-1158 / 第 1131-1158 行

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

### Lines 1159-1184 / 第 1159-1184 行

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

### Lines 1185-1187 / 第 1185-1187 行

~~~~python
            # Wait A/B buffer empty
            a_load2trans_pipeline.producer_tail(a_load2trans_producer_state)
            b_load2mma_pipeline.producer_tail(b_load2mma_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1189-1189 / 第 1189-1189 行

~~~~python
        # Specialized TMA load for scale tensor
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1190-1210 / 第 1190-1210 行

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
                scale_load2trans_producer_state = pipeline.make_pipeline_state(
                    pipeline.PipelineUserType.Producer, self.num_scale_load2trans_stage
                )
                scale_k_tile_cnt = cute.size(mS_mkl.layout.shape[1][1])
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1212-1230 / 第 1212-1230 行

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

### Lines 1232-1256 / 第 1232-1256 行

~~~~python
                    scale_load2trans_producer_state.reset_count()
                    peek_scale_load2trans_empty_status = cutlass.Boolean(1)
                    if scale_load2trans_producer_state.count < scale_k_tile_cnt:
                        peek_scale_load2trans_empty_status = (
                            scale_load2trans_pipeline.producer_try_acquire(
                                scale_load2trans_producer_state
                            )
                        )
                    for k_tile in cutlass.range(0, scale_k_tile_cnt, 1, unroll=1):
                        scale_load2trans_pipeline.producer_acquire(
                            scale_load2trans_producer_state,
                            peek_scale_load2trans_empty_status,
                        )
                        # TMA load scale
                        cute.copy(
                            tma_atom_s,
                            tSgS_slice_filtered[
                                (None, scale_load2trans_producer_state.count)
                            ],
                            tSsS[(None, scale_load2trans_producer_state.index)],
                            tma_bar_ptr=scale_load2trans_pipeline.producer_get_barrier(
                                scale_load2trans_producer_state
                            ),
                            mcast_mask=s_full_mcast_mask,
                        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1258-1278 / 第 1258-1278 行

~~~~python
                        scale_load2trans_producer_state.advance()
                        peek_scale_load2trans_empty_status = cutlass.Boolean(1)
                        if scale_load2trans_producer_state.count < scale_k_tile_cnt:
                            peek_scale_load2trans_empty_status = (
                                scale_load2trans_pipeline.producer_try_acquire(
                                    scale_load2trans_producer_state
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
                scale_load2trans_pipeline.producer_tail(scale_load2trans_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1280-1280 / 第 1280-1280 行

~~~~python
        # Specialized transform warps
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1281-1286 / 第 1281-1286 行

~~~~python
        if warp_idx >= self.transform_warp_id[0]:
            cute.arch.setmaxregister_increase(self.num_regs_transform_warps)
            transform_local_tidx = tidx - 32 * self.transform_warp_id[0]
            # Get the pointer to the TMEM buffer
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
            accumulators = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1288-1315 / 第 1288-1315 行

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
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1316-1339 / 第 1316-1339 行

~~~~python
            # make fragment for input A and transformed A
            tArA = cute.make_rmem_tensor(
                tAsA_input[(None, None, None, None, 0)].shape, tAsA_input.element_type
            )
            tArA_transform = cute.make_rmem_tensor(
                tAsA_input[(None, None, None, None, 0)].shape, self.mma_dtype
            )
            # Partition scale tensor
            smem_thr_copy_S = None
            tSsS_trans = None
            tSrS_copy = None
            tSrS = None
            if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
                smem_thr_copy_S, tSsS_trans, tSrS_copy, tSrS = (
                    mixed_input_utils.scale_partition(
                        src_copy_a, tCsS, transform_local_tidx, self.mma_dtype
                    )
                )
                assert cute.size(tSrS, mode=[0]) == cute.size(tArA, mode=[0]), (
                    "tSrS and tArA have different leading dimension"
                )
                assert cute.size(tSrS) == cute.size(tArA), (
                    "tSrS and tArA have different shape"
                )
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1340-1360 / 第 1340-1360 行

~~~~python
            # Deduce a sub-tile size and tile tensors
            transform_tiler_size = min(
                cute.size(cute.coalesce(tAsA_input.layout), mode=[0]), 64
            )
            transform_tiler = cute.make_layout(transform_tiler_size)
            tArA_load = cute.flat_divide(tArA, transform_tiler)
            tArA_load = cute.group_modes(tArA_load, 1, cute.rank(tArA_load))
            tSrS_load = (
                cute.flat_divide(tSrS, transform_tiler)
                if self.scale_mode is TransformMode.ConvertScale
                else None
            )
            tSrS_load = (
                cute.group_modes(tSrS_load, 1, cute.rank(tSrS_load))
                if self.scale_mode is TransformMode.ConvertScale
                else None
            )
            tArA_transform_store = cute.flat_divide(tArA_transform, transform_tiler)
            tArA_transform_store = cute.group_modes(
                tArA_transform_store, 1, cute.rank(tArA_transform_store)
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1362-1389 / 第 1362-1389 行

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
            scale_load2trans_consumer_state = (
                pipeline.make_pipeline_state(
                    pipeline.PipelineUserType.Consumer,
                    self.num_scale_load2trans_stage,
                )
                if self.scale_mode is TransformMode.ConvertScale
                else None
            )
            trans2mma_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer,
                self.num_trans2mma_stage,
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1390-1415 / 第 1390-1415 行

~~~~python
            )
            while work_tile.is_valid_tile:
                a_load2trans_consumer_state.reset_count()
                peek_load2trans_full_status = cutlass.Boolean(1)
                if a_load2trans_consumer_state.count < k_tile_cnt:
                    peek_load2trans_full_status = (
                        a_load2trans_pipeline.consumer_try_wait(
                            a_load2trans_consumer_state
                        )
                    )
                peek_scale_load2trans_full_status = cutlass.Boolean(1)
                if cutlass.const_expr(self.scale_mode == TransformMode.ConvertScale):
                    scale_load2trans_consumer_state.reset_count()
                    peek_scale_load2trans_full_status = (
                        scale_load2trans_pipeline.consumer_try_wait(
                            scale_load2trans_consumer_state
                        )
                    )
                trans2mma_producer_state.reset_count()
                peek_trans2mma_empty_status = cutlass.Boolean(1)
                if trans2mma_producer_state.count < k_tile_cnt:
                    peek_trans2mma_empty_status = (
                        trans2mma_pipeline.producer_try_acquire(
                            trans2mma_producer_state
                        )
                    )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1417-1444 / 第 1417-1444 行

~~~~python
                for k_tile in cutlass.range(0, k_tile_cnt, 1, unroll=1):
                    a_load2trans_pipeline.consumer_wait(
                        a_load2trans_consumer_state, peek_load2trans_full_status
                    )
                    tAsA_input_slice = tAsA_input[
                        (None, None, None, None, a_load2trans_consumer_state.index)
                    ]
                    tAsA_input_slice = cute.flat_divide(
                        tAsA_input_slice, transform_tiler
                    )
                    tAsA_input_slice = cute.group_modes(
                        tAsA_input_slice, 1, cute.rank(tAsA_input_slice)
                    )
                    if cutlass.const_expr(
                        self.scale_mode == TransformMode.ConvertScale
                    ):
                        scale_load2trans_pipeline.consumer_wait(
                            scale_load2trans_consumer_state,
                            peek_scale_load2trans_full_status,
                        )
                    trans2mma_pipeline.producer_acquire(
                        trans2mma_producer_state, peek_trans2mma_empty_status
                    )
                    # load scale tensor when needed
                    if cutlass.const_expr(
                        self.scale_mode == TransformMode.ConvertScale
                    ):
                        if k_tile % num_k_tiles_per_scale == 0:
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1445-1463 / 第 1445-1463 行

~~~~python
                            tSsS_slice = tSsS_trans[
                                (
                                    None,
                                    None,
                                    None,
                                    None,
                                    scale_load2trans_consumer_state.index,
                                )
                            ]
                            tSsS_slice_filtered = cute.make_tensor(
                                tSsS_slice.iterator,
                                cute.filter_zeros(tSsS_slice.layout),
                            )
                            cute.autovec_copy(tSsS_slice_filtered, tSrS_copy)
                        cur_scale_load2trans_consumer_state = (
                            scale_load2trans_consumer_state.clone()
                        )
                        if (k_tile + 1) % num_k_tiles_per_scale == 0:
                            scale_load2trans_consumer_state.advance()
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1465-1492 / 第 1465-1492 行

~~~~python
                    cur_a_load2trans_consumer_state = (
                        a_load2trans_consumer_state.clone()
                    )
                    for idx in cutlass.range_constexpr(cute.size(tArA_load, mode=[1])):
                        # Load A from shared memory
                        cute.autovec_copy(
                            tAsA_input_slice[(None, idx)],
                            tArA_load[(None, idx)],
                        )
                        if cutlass.const_expr(
                            idx == cute.size(tArA_load, mode=[1]) - 1
                        ):
                            a_load2trans_consumer_state.advance()
                            if a_load2trans_consumer_state.count < k_tile_cnt:
                                peek_load2trans_full_status = (
                                    a_load2trans_pipeline.consumer_try_wait(
                                        a_load2trans_consumer_state
                                    )
                                )
                                if cutlass.const_expr(
                                    self.scale_mode == TransformMode.ConvertScale
                                ):
                                    peek_scale_load2trans_full_status = (
                                        scale_load2trans_pipeline.consumer_try_wait(
                                            scale_load2trans_consumer_state
                                        )
                                    )
                        # Convert it to mma dtype
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1493-1520 / 第 1493-1520 行

~~~~python
                        tensor_transformed = mixed_input_utils.cvt_tensor_a(
                            tArA_load[(None, idx)], self.mma_dtype, self.shuffle_a
                        )
                        if cutlass.const_expr(
                            self.scale_mode == TransformMode.ConvertScale
                        ):
                            scale = cute.TensorSSA(
                                tSrS_load[(None, idx)].load(),
                                tensor_transformed.shape,
                                self.mma_dtype,
                            )
                            # Apply scale
                            tensor_transformed = tensor_transformed * scale
                        tArA_transform_store[(None, idx)].store(tensor_transformed)
                    # Store transformed A to tensor memory or shared memory
                    mixed_input_utils.store_transformed_a(
                        tArA_transform,
                        tAsA_transform[
                            (None, None, None, None, trans2mma_producer_state.index)
                        ],
                        dst_copy_a,
                    )
                    # Ensure all transform threads have finished the copy and reached the fence
                    self.transform_sync_barrier.arrive_and_wait()
                    if cutlass.const_expr(
                        self.transform_a_source == tcgen05.OperandSource.TMEM
                    ):
                        cute.arch.fence_view_async_tmem_store()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1521-1531 / 第 1521-1531 行

~~~~python
                    else:
                        cute.arch.fence_proxy(
                            "async.shared",
                            space="cta",
                        )
                    if cutlass.const_expr(
                        self.scale_mode == TransformMode.ConvertScale
                    ):
                        scale_load2trans_pipeline.consumer_release(
                            cur_scale_load2trans_consumer_state
                        )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1533-1557 / 第 1533-1557 行

~~~~python
                    a_load2trans_pipeline.consumer_release(
                        cur_a_load2trans_consumer_state
                    )
                    # Signal the completion of transformation
                    trans2mma_pipeline.producer_commit(trans2mma_producer_state)
                    trans2mma_producer_state.advance()
                    if trans2mma_producer_state.count < k_tile_cnt:
                        peek_trans2mma_empty_status = (
                            trans2mma_pipeline.producer_try_acquire(
                                trans2mma_producer_state
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
            # Wait a_transform buffer empty
            trans2mma_pipeline.producer_tail(trans2mma_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1559-1559 / 第 1559-1559 行

~~~~python
        # Specialized MMA warp
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1560-1587 / 第 1560-1587 行

~~~~python
        if warp_idx == self.mma_warp_id:
            cute.arch.setmaxregister_decrease(self.num_regs_mma_warp)
             # Get the pointer to the TMEM buffer
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
            accumulators = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)
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
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1588-1613 / 第 1588-1613 行

~~~~python
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
                # (MMA, MMA_M, MMA_N)
                tCtAcc = tCtAcc_base[(None, None, None, acc_producer_state.index)]
                b_load2mma_consumer_state.reset_count()
                trans2mma_consumer_state.reset_count()
                peek_trans2mma_full_status = cutlass.Boolean(1)
                if is_leader_cta:
                    if trans2mma_consumer_state.count < k_tile_cnt:
                        peek_trans2mma_full_status = (
                            trans2mma_pipeline.consumer_try_wait(
                                trans2mma_consumer_state
                            )
                        )
                    acc_pipeline.producer_acquire(acc_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1615-1635 / 第 1615-1635 行

~~~~python
                    tiled_mma.set(tcgen05.Field.ACCUMULATE, False)
                    # Mma mainloop
                    for k_tile in cutlass.range(0, k_tile_cnt, 1, unroll=1):
                        trans2mma_pipeline.consumer_wait(
                            trans2mma_consumer_state, peek_trans2mma_full_status
                        )
                        b_load2mma_pipeline.consumer_wait(b_load2mma_consumer_state)
                        num_kblocks = cute.size(tCrA, mode=[2])
                        for kblock_idx in cutlass.range(num_kblocks, unroll_full=True):
                            kblock_coord_a = (
                                None,
                                None,
                                kblock_idx,
                                trans2mma_consumer_state.index,
                            )
                            kblock_coord_b = (
                                None,
                                None,
                                kblock_idx,
                                b_load2mma_consumer_state.index,
                            )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1637-1659 / 第 1637-1659 行

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
                        trans2mma_pipeline.consumer_release(trans2mma_consumer_state)
                        b_load2mma_pipeline.consumer_release(b_load2mma_consumer_state)
                        trans2mma_consumer_state.advance()
                        b_load2mma_consumer_state.advance()
                        peek_trans2mma_full_status = cutlass.Boolean(1)
                        if trans2mma_consumer_state.count < k_tile_cnt:
                            peek_trans2mma_full_status = (
                                trans2mma_pipeline.consumer_try_wait(
                                    trans2mma_consumer_state
                                )
                            )
                    # Async arrive accumulator buffer full
                    acc_pipeline.producer_commit(acc_producer_state)
                acc_producer_state.advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1661-1661 / 第 1661-1661 行

~~~~python
                # Advance to next tile
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1662-1673 / 第 1662-1673 行

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

### Lines 1675-1675 / 第 1675-1675 行

~~~~python
        # Specialized epilogue warps
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1676-1696 / 第 1676-1696 行

~~~~python
        if warp_idx < self.mma_warp_id:
            cute.arch.setmaxregister_increase(self.num_regs_epilogue_warps)
            epi_tidx = tidx
            # Get the pointer to the TMEM buffer
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
            accumulators = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)
            tCtAcc_base = accumulators
            # Partition for epilogue
            tiled_copy_t2r, tTR_tAcc_base, tTR_rAcc = (
                mixed_input_utils.epilog_tmem_copy_and_partition(
                    self.cta_tile_shape_mnk,
                    self.c_layout,
                    self.c_dtype,
                    self.acc_dtype,
                    epi_tidx,
                    tCtAcc_base,
                    tCgC,
                    epi_tile,
                    self.use_2cta_instrs,
                )
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1698-1721 / 第 1698-1721 行

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

### Lines 1723-1723 / 第 1723-1723 行

~~~~python
            # Predicates
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1724-1730 / 第 1724-1730 行

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

### Lines 1732-1734 / 第 1732-1734 行

~~~~python
            acc_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_acc_stage
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 1736-1743 / 第 1736-1743 行

~~~~python
            c_producer_group = pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                32 * len(self.epilog_warp_id),
            )
            c_pipeline = pipeline.PipelineTmaStore.create(
                num_stages=self.num_c_stage,
                producer_group=c_producer_group,
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 1745-1745 / 第 1745-1745 行

~~~~python
            # Persistent tile scheduling loop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1746-1759 / 第 1746-1759 行

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
            num_prev_subtiles = cutlass.Int32(0)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1760-1787 / 第 1760-1787 行

~~~~python
            while work_tile.is_valid_tile:
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
                        tma_store_offset_coord[2] + bSG_gC.iterator[2],
                    ),
                    bSG_gC.layout,
                )
                tTR_gC = tTR_gC_partitioned[
                    (
                        None,
                        None,
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1788-1799 / 第 1788-1799 行

~~~~python
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
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1801-1805 / 第 1801-1805 行

~~~~python
                tTR_tAcc = tTR_tAcc_base[
                    (None, None, None, None, None, acc_consumer_state.index)
                ]
                # Wait for accumulator buffer full
                acc_pipeline.consumer_wait(acc_consumer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1807-1809 / 第 1807-1809 行

~~~~python
                tTR_tAcc = cute.group_modes(tTR_tAcc, 3, cute.rank(tTR_tAcc))
                bSG_gC = cute.group_modes(bSG_gC, 1, cute.rank(bSG_gC))
                tTR_gC = cute.group_modes(tTR_gC, 3, cute.rank(tTR_gC))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 1811-1811 / 第 1811-1811 行

~~~~python
                # Store accumulator to global memory in subtiles
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1812-1839 / 第 1812-1839 行

~~~~python
                subtile_cnt = cute.size(tTR_tAcc.shape, mode=[3])
                for subtile_idx in cutlass.range(subtile_cnt):
                    # Load accumulator from tensor memory buffer to register
                    tTR_tAcc_mn = tTR_tAcc[(None, None, None, subtile_idx)]
                    cute.copy(tiled_copy_t2r, tTR_tAcc_mn, tTR_rAcc)
                    if work_tile.distance_to_boundary >= self.cta_tile_shape_mnk[1]:
                        # Convert to C type
                        acc_vec = tiled_copy_r2s.retile(tTR_rAcc).load()
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
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1840-1867 / 第 1840-1867 行

~~~~python
                                bSG_sC[(None, c_buffer)],
                                bSG_gC[(None, subtile_idx)],
                            )
                            c_pipeline.producer_commit()
                            c_pipeline.producer_acquire()
                        self.epilog_sync_barrier.arrive_and_wait()
                    else:
                        # Convert to C type
                        acc_vec = tTR_rAcc.load()
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
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 1868-1884 / 第 1868-1884 行

~~~~python
                            pred=cute.flatten(tCpC),
                        )
                # Async arrive accumulator buffer empty
                with cute.arch.elect_one():
                    acc_pipeline.consumer_release(acc_consumer_state)
                acc_consumer_state.advance()
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

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1886-1886 / 第 1886-1886 行

~~~~python
            # Dealloc the tensor memory buffer
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1887-1890 / 第 1887-1890 行

~~~~python
            tmem.relinquish_alloc_permit()
            self.epilog_sync_barrier.arrive_and_wait()
            tmem.free(tmem_ptr)
            c_pipeline.producer_tail()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1892-1919 / 第 1892-1919 行

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
        scale_mode: TransformMode,
    ) -> tuple[int, int, int, int, int, int, int, int]:
        """
        Compute pipeline stages and TMEM column allocation configurations.

        This method calculates the number of pipeline stages for different operations
        (tile_info, load2trans, trans2mma, accumulator, etc.) and determines TMEM column allocation
        based on available memory resources and tile configuration.

        :param tiled_mma: The tiled MMA object defining the core computation.
        :type tiled_mma: cute.TiledMma
        :param mma_tiler_mnk: The shape (M, N, K) of the MMA tiler.
        :type mma_tiler_mnk: tuple[int, int, int]
        :param cta_tile_shape_mnk: The shape (M, N, K) of the CTA tile.
~~~~

**EN**: Defines `_compute_stages_and_tmem_cols`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `_compute_stages_and_tmem_cols`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1920-1947 / 第 1920-1947 行

~~~~python
        :type cta_tile_shape_mnk: tuple[int, int, int]
        :param epi_tile: The epilogue tile shape.
        :type epi_tile: cute.Tile
        :param a_dtype: Data type of operand A.
        :type a_dtype: type[cutlass.Numeric]
        :param b_dtype: Data type of operand B.
        :type b_dtype: type[cutlass.Numeric]
        :param c_dtype: Data type of operand C.
        :type c_dtype: type[cutlass.Numeric]
        :param c_layout: Layout enum of operand C.
        :type c_layout: utils.LayoutEnum
        :param transform_a_source: The source of the transformed A tensor.
        :type transform_a_source: tcgen05.OperandSource
        :param scale_granularity_m: The granularity of the scale tensor along the M mode.
        :type scale_granularity_m: int
        :param scale_granularity_k: The granularity of the scale tensor along the K mode.
        :type scale_granularity_k: int
        :param smem_buffer_align_bytes: The alignment of the shared memory buffer.
        :type smem_buffer_align_bytes: int
        :param scale_mode: The transform mode.
        :type scale_mode: TransformMode

        :return: A tuple containing the number of stages for:
                 (load2trans, scale_load2trans, transform2mma, accumulator, c, tile_info, tmem_acc_cols, tmem_a_cols)
        :rtype: tuple[int, int, int, int, int, int, int]
        - num_load2trans_stage: Stages for load-to-transform A and B tensors pipeline
        - num_scale_load2trans_stage: Stages for scale load-to-transform A tensor pipeline
        - num_trans2mma_stage: Stages for transform-to-MMA pipeline
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1948-1967 / 第 1948-1967 行

~~~~python
        - num_acc_stage: Stages for accumulator-to-epilogue pipeline
        - num_c_stage: Stages for epilogue-to-output C pipeline
        - num_tile_info_stage: Stages for buffers storing tile info
        - num_acc_tmem_cols: TMEM columns for accumulator
        - num_a_tmem_cols: TMEM columns for transformed A tensor
        """
        # Compute tmem columns required for accumulator
        acc_shape = tiled_mma.partition_shape_C(mma_tiler_mnk[:2])
        tCtAcc_stage1 = tiled_mma.make_fragment_C(cute.append(acc_shape, 1))
        num_tmem_acc_col_per_stage = utils.get_num_tmem_alloc_cols(tCtAcc_stage1, True)
        # Heuristic to decide the number of stages for accumulator
        sm100_tmem_columns = cute.arch.get_max_tmem_alloc_cols("sm_100")
        accumulator_stage_count = sm100_tmem_columns // num_tmem_acc_col_per_stage
        if transform_a_source == tcgen05.OperandSource.TMEM:
            if num_tmem_acc_col_per_stage < 128:
                accumulator_stage_count = 3
            elif num_tmem_acc_col_per_stage < 256:
                accumulator_stage_count = 2
            else:
                accumulator_stage_count = 1
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1968-1977 / 第 1968-1977 行

~~~~python
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

### Lines 1979-1985 / 第 1979-1985 行

~~~~python
        bytes_per_pipeline_stage = 16
        # By default, we use 2 stages for tile info
        num_tile_info_stage = 2
        tile_info_bytes = (
            cute.size_in_bytes(cute.Int32, cute.make_layout((4, num_tile_info_stage)))
            + bytes_per_pipeline_stage * num_tile_info_stage
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 1987-1995 / 第 1987-1995 行

~~~~python
        c_stage_count = 2
        c_smem_layout_staged_one = sm100_utils.make_smem_layout_epi(
            c_dtype,
            c_layout,
            epi_tile,
            1,
        )
        c_bytes_per_stage = cute.size_in_bytes(c_dtype, c_smem_layout_staged_one)
        c_bytes = c_bytes_per_stage * c_stage_count
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1997-2021 / 第 1997-2021 行

~~~~python
        smem_capacity = utils.get_smem_capacity_in_bytes("sm_100")
        if scale_mode == TransformMode.ConvertOnly:
            scale_load2trans_stage_count = 0
            a_scale_bytes_per_stage = 0
        else:
            # Ensure we have 4 buffers for scale tiles needed for 1 CTA tile
            a_scale_k_mode = max(cta_tile_shape_mnk[2] // scale_granularity_k, 1)
            a_scale_m_mode = max(cta_tile_shape_mnk[0] // scale_granularity_m, 1)
            scale_load2trans_stage_count = 4
            a_scale_bytes_per_stage = cute.round_up(
                cute.size_in_bytes(
                    tiled_mma.op.a_dtype,
                    cute.make_layout((a_scale_m_mode, a_scale_k_mode)),
                ),
                smem_buffer_align_bytes,
            )
        a_scale_bytes = (
            a_scale_bytes_per_stage + bytes_per_pipeline_stage
        ) * scale_load2trans_stage_count
        carveout_smem_bytes = (
            bytes_per_pipeline_stage * accumulator_stage_count
            + a_scale_bytes
            + c_bytes
            + tile_info_bytes
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2023-2023 / 第 2023-2023 行

~~~~python
        # Compute transform stages if A is in TMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2024-2026 / 第 2024-2026 行

~~~~python
        num_tmem_acc_cols = cute.round_up(
            accumulator_stage_count * num_tmem_acc_col_per_stage, 4
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2028-2055 / 第 2028-2055 行

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

### Lines 2056-2072 / 第 2056-2072 行

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

### Lines 2074-2099 / 第 2074-2099 行

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

### Lines 2100-2107 / 第 2100-2107 行

~~~~python
        # Check if we can increase c_stage_count with leftover smem
        c_stage_count += (
            smem_capacity
            - load2transform_stage_count * ab_load_bytes_per_stage
            - transform2mma_stage_count * a_transform_bytes_per_stage
            - scale_load2trans_stage_count * a_scale_bytes_per_stage
            - c_bytes
        ) // c_bytes_per_stage
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2109-2118 / 第 2109-2118 行

~~~~python
        return (
            load2transform_stage_count,
            scale_load2trans_stage_count,
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

### Lines 2120-2133 / 第 2120-2133 行

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

### Lines 2135-2138 / 第 2135-2138 行

~~~~python
        tile_sched_params = utils.PersistentTileSchedulerParams(
            num_ctas_mnl, cluster_shape_mnl
        )
        grid = (cluster_shape_mn[0], cluster_shape_mn[1], max_active_clusters)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2140-2140 / 第 2140-2140 行

~~~~python
        return tile_sched_params, grid
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2142-2159 / 第 2142-2159 行

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

### Lines 2161-2187 / 第 2161-2187 行

~~~~python
        if not mixed_input_utils.is_valid_mma_tiler_and_cluster_shape(
            mma_tiler, cluster_shape_mn, use_2cta_instrs
        ):
            return False
        if not mixed_input_utils.is_valid_scale_granularity(
            scale_granularity_m, scale_granularity_k, a_dtype, k, mma_tiler[2]
        ):
            return False
        if not mixed_input_utils.is_valid_tensor_alignment(
            m,
            n,
            k,
            a_dtype,
            b_dtype,
            c_dtype,
            b_dtype,
            a_major,
            b_major,
            c_major,
            mma_tiler,
            use_2cta_instrs,
            cluster_shape_mn,
            scale_granularity_m,
            scale_granularity_k,
        ):
            return False
        return True
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 2190-2194 / 第 2190-2194 行

~~~~python
def get_advanced_compiler_control_path():
    """
    Return the path to the advanced compiler control file of this example. If not found, return None.
    """
    import os
~~~~

**EN**: Defines `get_advanced_compiler_control_path`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `get_advanced_compiler_control_path`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2196-2198 / 第 2196-2198 行

~~~~python
    need_advanced_compiler_control = False
    try:
        from cutlass import CUDA_VERSION
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2200-2203 / 第 2200-2203 行

~~~~python
        if CUDA_VERSION.major == 13 and CUDA_VERSION.minor == 1:
            need_advanced_compiler_control = True
    except ImportError:
        pass
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2205-2214 / 第 2205-2214 行

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

### Lines 2217-2244 / 第 2217-2244 行

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

### Lines 2245-2247 / 第 2245-2247 行

~~~~python
    """
    m, n, k, l = mnkl
    import torch
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 2249-2250 / 第 2249-2250 行

~~~~python
    if not torch.cuda.is_available():
        raise ValueError("CUDA is not available")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 2252-2252 / 第 2252-2252 行

~~~~python
    # Check if given configuration is supported
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2253-2267 / 第 2253-2267 行

~~~~python
    if not GroupedMixedInputGemmKernel.can_implement(
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

### Lines 2269-2269 / 第 2269-2269 行

~~~~python
    # Get current CUDA stream from PyTorch
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2270-2272 / 第 2270-2272 行

~~~~python
    torch_stream = torch.cuda.current_stream()
    # Get the raw stream pointer as a CUstream
    current_stream = cuda.CUstream(torch_stream.cuda_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 2274-2281 / 第 2274-2281 行

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

### Lines 2283-2288 / 第 2283-2288 行

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

### Lines 2290-2317 / 第 2290-2317 行

~~~~python
    shuffle_a = shuffle_a and shuffle_supported
    mixed_input_gemm = GroupedMixedInputGemmKernel(
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

### Lines 2318-2328 / 第 2318-2328 行

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

### Lines 2330-2338 / 第 2330-2338 行

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

### Lines 2340-2350 / 第 2340-2350 行

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

### Lines 2352-2369 / 第 2352-2369 行

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

### Lines 2371-2371 / 第 2371-2371 行

~~~~python
    # Early return if no performance measurement is needed
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 2372-2373 / 第 2372-2373 行

~~~~python
    if iterations <= 0:
        return
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2375-2402 / 第 2375-2402 行

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

### Lines 2403-2405 / 第 2403-2405 行

~~~~python
        return testing.JitArguments(
            a_tensor, a_scale_tensor, b_tensor, cumsum_tensor, c_tensor, current_stream
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 2407-2419 / 第 2407-2419 行

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

### Lines 2421-2428 / 第 2421-2428 行

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

### Lines 2430-2430 / 第 2430-2430 行

~~~~python
    return exec_time  # Return execution time in microseconds
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2433-2433 / 第 2433-2433 行

~~~~python
if __name__ == "__main__":
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 2435-2441 / 第 2435-2441 行

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

### Lines 2443-2470 / 第 2443-2470 行

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

### Lines 2471-2498 / 第 2471-2498 行

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

### Lines 2499-2526 / 第 2499-2526 行

~~~~python
    parser.add_argument(
        "--skip_ref_check", action="store_true", help="Skip reference checking"
    )
    parser.add_argument(
        "--uniform_group_sizes", action="store_true", help="Use uniform group sizes"
    )
    args = parser.parse_args()
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
    print("PASS")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

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
- `typing.Optional` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `sys` — used by this example / 供该示例使用
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
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass.CUDA_VERSION` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
