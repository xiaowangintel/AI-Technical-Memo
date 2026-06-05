# cutedsl_grouped_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/vendored_templates/cutedsl/kernels/cutedsl_grouped_gemm.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `GroupedGemmKernel`. It exposes functions such as `create_tensor_and_stride`, `create_tensors_for_all_groups`, and `run`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `GroupedGemmKernel` 等类。同时提供 `create_tensor_and_stride`、`create_tensors_for_all_groups`、`run` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
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

````
- **EN**: Implements the local expressions and calls needed for this part of the module.
- **CN**: 实现了该模块这一部分所需的局部表达式与调用逻辑。

### Lines 29-56 / 第 29-56 行
````python
import argparse
import functools
from typing import List, Type, Union
from inspect import isclass

import torch
import cuda.bindings.driver as cuda

import cutlass
import cutlass.cute as cute
import cutlass.cute.testing as testing
import cutlass.utils as utils
import cutlass.pipeline as pipeline
from cutlass.pipeline import pipeline_init_arrive, pipeline_init_wait
from cutlass.cute.nvgpu import cpasync, tcgen05
import cutlass.utils.blackwell_helpers as sm100_utils
import cutlass.torch as cutlass_torch

"""
A grouped GEMM example for the NVIDIA Blackwell SM100 architecture using CUTE DSL

This example demonstrates an implementation of grouped GEMM using a TMA plus Blackwell SM100 TensorCore
warp-specialized persistent kernel.
The grouped GEMM workload computes a batch of GEMM operations with distinct problem sizes. Pointers to matrices
in global memory are passed to the kernel in an array (also held in global memory). Similarly, problem shapes and
strides are also stored in arrays in GMEM.

This differs from "Batched Array" GEMM since the size of each GEMM problem in the grouped GEMM concept may be distinct.
````
- **EN**: Imports dependencies such as `argparse`, `functools`, `typing`, `inspect`, `torch`, `cuda.bindings.driver`, and `...+8` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里导入了 `argparse`、`functools`、`typing`、`inspect`、`torch`、`cuda.bindings.driver`、`另有8项` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。

### Lines 57-84 / 第 57-84 行
````python

To run this example:

.. code-block:: bash

    python examples/blackwell/grouped_gemm.py                                                 \
      --ab_dtype Float16 --c_dtype Float16 --acc_dtype Float32                                \
      --mma_tiler_mn 128,64 --cluster_shape_mn 1,1                                            \
      --problem_sizes_mnkl "(8192,1280,32,1),(16,384,1536,1),(640,1280,16,1),(640,160,16,1)"  \
      --num_groups 4  --tensormap_update_mode SMEM

The above example command makes 4 groups of different m, n, k sizes. The Blackwell tcgen05 MMA tile shape
is specified as (128, 64) and the cluster shape is (1,1). The input, mma accumulator and output data type
are set as fp16, fp32 and fp16, respectively.

To collect performance with NCU profiler:

.. code-block:: bash

    ncu python examples/blackwell/grouped_gemm.py                                             \
      --ab_dtype Float16 --c_dtype Float16 --acc_dtype Float32                                \
      --mma_tiler_mn 128,64 --cluster_shape_mn 1,1                                            \
      --problem_sizes_mnkl "(8192,1280,32,1),(16,384,1536,1),(640,1280,16,1),(640,160,16,1)"  \
      --num_groups 4  --tensormap_update_mode SMEM                                            \
      --warmup_iterations 1 --iterations 10 --skip_ref_check

There are some constrains for this example. Besides the constrains from the Balckwell dense GEMM persistent example,
there are also the following constrains:
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 85-112 / 第 85-112 行
````python
* Only fp16 and bf16 data types are supported as inputs.
* Output data types could be fp16, bf16 or fp32.
* The contiguous dimension of each tensor must be at least 16 bytes aligned.
* The l mode(aka, batch size) for each group must be 1.
* The majorness for A, B and C must be the same across all groups.
"""


class GroupedGemmKernel:
    def __init__(
        self,
        acc_dtype: type[cutlass.Numeric],
        use_2cta_instrs: bool,
        mma_tiler_mn: tuple[int, int],
        cluster_shape_mn: tuple[int, int],
        tensormap_update_mode: utils.TensorMapUpdateMode = utils.TensorMapUpdateMode.SMEM,
    ):
        """Initializes the configuration for a Blackwell grouped GEMM kernel.

        Besides configurations for dense persistent GEMM, there is an extra config specific to grouped GEMM:

        Tensormap Update Mode:
        - tensormap_update_mode: Specifies whether the tensormap is
            updated in global memory(GMEM) or shared memory(SMEM).
           The 2 modes are functionally equivalent and the difference are:
            - We buffer 3 tensormaps in SMEM for A, B, and C tensors (each TMA descriptor takes 128B) when TMA updates performed on SMEM.
            - Performance varies between modes depending on problem size; optimal choice differs across workloads.

````
- **EN**: Introduces class `GroupedGemmKernel`, function `__init__`. Initializes or updates values such as `acc_dtype`, `use_2cta_instrs`, `mma_tiler_mn`, `cluster_shape_mn`, and `tensormap_update_mode`.
- **CN**: 这里定义了类`GroupedGemmKernel`、函数`__init__`。初始化或更新了 `acc_dtype`、`use_2cta_instrs`、`mma_tiler_mn`、`cluster_shape_mn`、`tensormap_update_mode` 等值。

### Lines 113-140 / 第 113-140 行
````python
        :param acc_dtype: Data type of the accumulator.
        :type acc_dtype: type[cutlass.Numeric]
        :param use_2cta_instrs: Boolean, True to use cta_group=2 MMA variant.
        :type use_2cta_instrs: bool
        :param mma_tiler_mn: tuple (M, N) shape of the MMA instruction.
        :type mma_tiler_mn: tuple[int, int]
        :param cluster_shape_mn: tuple (ClusterM, ClusterN) shape of the cluster.
        :type cluster_shape_mn: tuple[int, int]
        :param tensormap_update_mode: Mode for updating the tensormap (GMEM or SMEM), defaults to SMEM.
        :type tensormap_update_mode: utils.TensorMapUpdateMode, optional
        """
        self.acc_dtype: Type[cutlass.Numeric] = acc_dtype
        self.use_2cta_instrs = use_2cta_instrs
        self.cluster_shape_mn = cluster_shape_mn
        # K dimension is deferred in _setup_attributes
        self.mma_tiler = (*mma_tiler_mn, 1)
        self.cta_group = (
            tcgen05.CtaGroup.TWO if use_2cta_instrs else tcgen05.CtaGroup.ONE
        )

        self.tensormap_update_mode = tensormap_update_mode
        # Delegate tensormap ab initialization to MMA warp when SMEM mode is used for better latency hiding
        self.delegate_tensormap_ab_init = (
            tensormap_update_mode == utils.TensorMapUpdateMode.SMEM
        )

        self.num_mcast_ctas_a = 1
        self.num_mcast_ctas_b = 1
````
- **EN**: Initializes or updates values such as `tensormap_update_mode`. This range continues the implementation of function `GroupedGemmKernel.__init__`.
- **CN**: 初始化或更新了 `tensormap_update_mode` 等值。这一段延续了函数`GroupedGemmKernel.__init__` 的具体实现。

### Lines 141-168 / 第 141-168 行
````python
        self.is_a_mcast = False
        self.is_b_mcast = False

        self.occupancy = 1
        # Set specialized warp ids
        self.epilog_warp_id = (
            0,
            1,
            2,
            3,
        )
        self.mma_warp_id = 4
        self.tma_warp_id = 5
        self.threads_per_cta = 32 * len(
            (self.mma_warp_id, self.tma_warp_id, *self.epilog_warp_id)
        )
        # Set barrier for epilog sync, tmem ptr sync and tensormap update sync
        self.epilog_sync_barrier = pipeline.NamedBarrier(
            barrier_id=1,
            num_threads=32 * len(self.epilog_warp_id),
        )
        self.tmem_alloc_barrier = pipeline.NamedBarrier(
            barrier_id=2,
            num_threads=32 * len((self.mma_warp_id, *self.epilog_warp_id)),
        )
        # Barrier used by MMA/TMA warps to signal A/B tensormap initialization completion
        self.tensormap_ab_init_barrier = pipeline.NamedBarrier(
            barrier_id=3,
````
- **EN**: Initializes or updates values such as `barrier_id`, and `num_threads`. This range continues the implementation of function `GroupedGemmKernel.__init__`.
- **CN**: 初始化或更新了 `barrier_id`、`num_threads` 等值。这一段延续了函数`GroupedGemmKernel.__init__` 的具体实现。

### Lines 169-196 / 第 169-196 行
````python
            num_threads=32 * (len(self.epilog_warp_id) + 1),
        )
        self.smem_capacity = utils.get_smem_capacity_in_bytes("sm_100")
        self.num_tma_load_bytes = 0

    def _setup_attributes(self):
        """Set up configurations that are dependent on GEMM inputs

        Most of the implementation follows standard dense GEMM patterns,
        with the key difference being additional consideration for SMEM
        buffer needed for tensormap updates.
        """
        # Configure tiled mma
        tiled_mma = sm100_utils.make_trivial_tiled_mma(
            self.a_dtype,
            self.a_major_mode,
            self.b_major_mode,
            self.acc_dtype,
            self.cta_group,
            self.mma_tiler[:2],
        )

        # Compute mma/cluster/tile shapes
        mma_inst_shape_k = cute.size(tiled_mma.shape_mnk, mode=[2])
        mma_inst_tile_k = 4
        self.mma_tiler = (
            self.mma_tiler[0],
            self.mma_tiler[1],
````
- **EN**: Introduces function `_setup_attributes`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_threads`, `tiled_mma`, `mma_inst_shape_k`, and `mma_inst_tile_k`.
- **CN**: 这里定义了函数`_setup_attributes`。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_threads`、`tiled_mma`、`mma_inst_shape_k`、`mma_inst_tile_k` 等值。

### Lines 197-224 / 第 197-224 行
````python
            mma_inst_shape_k * mma_inst_tile_k,
        )
        self.cta_tile_shape_mnk = (
            self.mma_tiler[0] // cute.size(tiled_mma.thr_id.shape),
            self.mma_tiler[1],
            self.mma_tiler[2],
        )
        self.cluster_tile_shape_mnk = tuple(
            x * y for x, y in zip(self.cta_tile_shape_mnk, (*self.cluster_shape_mn, 1))
        )

        # Compute cluster layout
        self.cluster_layout_vmnk = cute.tiled_divide(
            cute.make_layout((*self.cluster_shape_mn, 1)),
            (tiled_mma.thr_id.shape,),
        )

        # Compute number of multicast CTAs for A/B
        self.num_mcast_ctas_a = cute.size(self.cluster_layout_vmnk.shape[2])
        self.num_mcast_ctas_b = cute.size(self.cluster_layout_vmnk.shape[1])
        self.is_a_mcast = self.num_mcast_ctas_a > 1
        self.is_b_mcast = self.num_mcast_ctas_b > 1

        # Compute epilogue subtile
        self.epi_tile = utils.compute_epilogue_tile_shape(
            self.cta_tile_shape_mnk,
            self.use_2cta_instrs,
            self.c_layout,
````
- **EN**: This range continues the implementation of function `GroupedGemmKernel._setup_attributes`.
- **CN**: 这一段延续了函数`GroupedGemmKernel._setup_attributes` 的具体实现。

### Lines 225-252 / 第 225-252 行
````python
            self.c_dtype,
        )

        # Setup A/B/C stage count in shared memory and ACC stage count in tensor memory
        (
            self.num_acc_stage,
            self.num_ab_stage,
            self.num_epi_stage,
        ) = self._compute_stages(
            tiled_mma,
            self.mma_tiler,
            self.a_dtype,
            self.b_dtype,
            self.epi_tile,
            self.c_dtype,
            self.c_layout,
            self.smem_capacity,
            self.occupancy,
        )

        self.a_smem_layout_staged = sm100_utils.make_smem_layout_a(
            tiled_mma,
            self.mma_tiler,
            self.a_dtype,
            self.num_ab_stage,
        )
        self.b_smem_layout_staged = sm100_utils.make_smem_layout_b(
            tiled_mma,
````
- **EN**: This range continues the implementation of function `GroupedGemmKernel._setup_attributes`.
- **CN**: 这一段延续了函数`GroupedGemmKernel._setup_attributes` 的具体实现。

### Lines 253-280 / 第 253-280 行
````python
            self.mma_tiler,
            self.b_dtype,
            self.num_ab_stage,
        )
        self.epi_smem_layout_staged = sm100_utils.make_smem_layout_epi(
            self.c_dtype,
            self.c_layout,
            self.epi_tile,
            self.num_epi_stage,
        )

        mbar_smem_bytes = self._get_mbar_smem_bytes(
            num_acc_stage=self.num_acc_stage,
            num_ab_stage=self.num_ab_stage,
            num_epi_stage=self.num_epi_stage,
        )
        tensormap_smem_bytes = self._get_tensormap_smem_bytes(
            self.tensormap_update_mode
        )
        if (
            mbar_smem_bytes
            + tensormap_smem_bytes
            + GroupedGemmKernel.tensor_memory_management_bytes
            > self.reserved_smem_bytes
        ):
            raise ValueError(
                f"smem consumption for mbar and tensormap {mbar_smem_bytes + tensormap_smem_bytes} exceeds the "
                f"reserved smem bytes {self.reserved_smem_bytes}"
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mbar_smem_bytes`, `num_acc_stage`, `num_ab_stage`, `num_epi_stage`, and `tensormap_smem_bytes`. This range continues the implementation of function `GroupedGemmKernel._setup_attributes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `mbar_smem_bytes`、`num_acc_stage`、`num_ab_stage`、`num_epi_stage`、`tensormap_smem_bytes` 等值。这一段延续了函数`GroupedGemmKernel._setup_attributes` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
            )

        # Compute the number of tensor memory allocation columns
        self.num_tmem_alloc_cols = self._compute_num_tmem_alloc_cols(
            tiled_mma, self.mma_tiler, self.num_acc_stage
        )

    @cute.jit
    def __call__(
        self,
        initial_a: cute.Tensor,
        initial_b: cute.Tensor,
        initial_c: cute.Tensor,
        group_count: cutlass.Constexpr[int],
        problem_shape_mnkl: cute.Tensor,
        strides_abc: cute.Tensor,
        tensor_address_abc: cute.Tensor,
        total_num_clusters: cutlass.Constexpr[int],
        tensormap_cute_tensor: cute.Tensor,
        max_active_clusters: cutlass.Constexpr[int],
        stream: cuda.CUstream,
    ):
        """Execute the GEMM operation in steps:
        - Setup static attributes before smem/grid/tma computation
        - Setup TMA load/store atoms and tensors
        - Compute grid size with regard to hardware constraints
        - Define shared storage for kernel
        - Launch the kernel synchronously
````
- **EN**: Introduces function `__call__`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `initial_a`, `initial_b`, `initial_c`, `group_count`, `problem_shape_mnkl`, `strides_abc`, and `...+5`.
- **CN**: 这里定义了函数`__call__`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `initial_a`、`initial_b`、`initial_c`、`group_count`、`problem_shape_mnkl`、`strides_abc`、`另有5项` 等值。

### Lines 309-336 / 第 309-336 行
````python

        For grouped GEMM, tensor shapes, tensor strides, and tensor address are all provided
        by different tensors in global memory. The "initial" tensors only carry data type and
        majorness information.

        :param initial_a: Initial tensor A, used for data type and majorness information.
        :type initial_a: cute.Tensor
        :param initial_b: Initial tensor B, used for data type and majorness information.
        :type initial_b: cute.Tensor
        :param initial_c: Initial tensor C, used for data type and majorness information.
        :type initial_c: cute.Tensor
        :param group_count: The number of GEMM groups.
        :type group_count: cutlass.Constexpr[int]
        :param problem_shape_mnkl: Tensor containing the (M, N, K, L) shape for each group.
        :type problem_shape_mnkl: cute.Tensor
        :param strides_abc: Tensor containing the strides for A, B, and C for each group.
        :type strides_abc: cute.Tensor
        :param tensor_address_abc: Tensor containing the base addresses for A, B, and C for each group.
        :type tensor_address_abc: cute.Tensor
        :param total_num_clusters: Total number of clusters needed for all groups.
        :type total_num_clusters: cutlass.Constexpr[int]
        :param tensormap_cute_tensor: Tensor for storing tensormaps.
        :type tensormap_cute_tensor: cute.Tensor
        :param max_active_clusters: Maximum number of active clusters.
        :type max_active_clusters: cutlass.Constexpr[int]
        :param stream: CUDA stream for asynchronous execution.
        :type stream: cuda.CUstream
        :raises TypeError: If A and B data types do not match.
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. This range continues the implementation of function `GroupedGemmKernel.__call__`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。这一段延续了函数`GroupedGemmKernel.__call__` 的具体实现。

### Lines 337-364 / 第 337-364 行
````python
        """

        self.a_dtype = initial_a.element_type
        self.b_dtype = initial_b.element_type
        self.c_dtype = initial_c.element_type

        self.a_major_mode = utils.LayoutEnum.from_tensor(initial_a).mma_major_mode()
        self.b_major_mode = utils.LayoutEnum.from_tensor(initial_b).mma_major_mode()
        self.c_layout = utils.LayoutEnum.from_tensor(initial_c)
        if cutlass.const_expr(self.a_dtype != self.b_dtype):
            raise TypeError(f"Type mismatch: {self.a_dtype} != {self.b_dtype}")

        # Setup attributes that dependent on gemm inputs
        self._setup_attributes()

        tiled_mma = sm100_utils.make_trivial_tiled_mma(
            self.a_dtype,
            self.a_major_mode,
            self.b_major_mode,
            self.acc_dtype,
            self.cta_group,
            self.mma_tiler[:2],
        )
        atom_thr_size = cute.size(tiled_mma.thr_id.shape)

        # Setup TMA load for A
        a_op = sm100_utils.cluster_shape_to_tma_atom_A(
            self.cluster_shape_mn, tiled_mma.thr_id
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tiled_mma`, `atom_thr_size`, and `a_op`. This range continues the implementation of function `GroupedGemmKernel.__call__`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tiled_mma`、`atom_thr_size`、`a_op` 等值。这一段延续了函数`GroupedGemmKernel.__call__` 的具体实现。

### Lines 365-392 / 第 365-392 行
````python
        )
        a_smem_layout = cute.slice_(self.a_smem_layout_staged, (None, None, None, 0))
        tma_atom_a, tma_tensor_a = cute.nvgpu.make_tiled_tma_atom_A(
            a_op,
            initial_a,
            a_smem_layout,
            self.mma_tiler,
            tiled_mma,
            self.cluster_layout_vmnk.shape,
        )

        # Setup TMA load for B
        b_op = sm100_utils.cluster_shape_to_tma_atom_B(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        b_smem_layout = cute.slice_(self.b_smem_layout_staged, (None, None, None, 0))
        tma_atom_b, tma_tensor_b = cute.nvgpu.make_tiled_tma_atom_B(
            b_op,
            initial_b,
            b_smem_layout,
            self.mma_tiler,
            tiled_mma,
            self.cluster_layout_vmnk.shape,
        )

        a_copy_size = cute.size_in_bytes(self.a_dtype, a_smem_layout)
        b_copy_size = cute.size_in_bytes(self.b_dtype, b_smem_layout)
        self.num_tma_load_bytes = (a_copy_size + b_copy_size) * atom_thr_size
````
- **EN**: Initializes or updates values such as `a_smem_layout`, `b_op`, `b_smem_layout`, `a_copy_size`, and `b_copy_size`. This range continues the implementation of function `GroupedGemmKernel.__call__`.
- **CN**: 初始化或更新了 `a_smem_layout`、`b_op`、`b_smem_layout`、`a_copy_size`、`b_copy_size` 等值。这一段延续了函数`GroupedGemmKernel.__call__` 的具体实现。

### Lines 393-420 / 第 393-420 行
````python

        # Setup TMA store for C
        tma_atom_c = None
        tma_tensor_c = None
        epi_smem_layout = cute.slice_(self.epi_smem_layout_staged, (None, None, 0))
        tma_atom_c, tma_tensor_c = cpasync.make_tiled_tma_atom(
            cpasync.CopyBulkTensorTileS2GOp(),
            initial_c,
            epi_smem_layout,
            self.epi_tile,
        )

        self.tile_sched_params, grid = self._compute_grid(
            total_num_clusters, self.cluster_shape_mn, max_active_clusters
        )

        self.buffer_align_bytes = 1024
        self.size_tensormap_in_i64 = (
            0
            if self.tensormap_update_mode == utils.TensorMapUpdateMode.GMEM
            else GroupedGemmKernel.num_tensormaps
            * GroupedGemmKernel.bytes_per_tensormap
            // 8
        )

        # Define shared storage for kernel
        @cute.struct
        class SharedStorage:
````
- **EN**: Introduces class `SharedStorage`. Applies decorators to register behavior or alter how the following definition is constructed. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tma_atom_c`, `tma_tensor_c`, and `epi_smem_layout`.
- **CN**: 这里定义了类`SharedStorage`。使用装饰器来注册行为，或改变后续定义的构造方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `tma_atom_c`、`tma_tensor_c`、`epi_smem_layout` 等值。

### Lines 421-448 / 第 421-448 行
````python
            tensormap_buffer: cute.struct.MemRange[
                cutlass.Int64, self.size_tensormap_in_i64
            ]
            ab_full_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_ab_stage]
            ab_empty_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_ab_stage]
            acc_full_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_acc_stage]
            acc_empty_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_acc_stage]
            tmem_dealloc_mbar_ptr: cutlass.Int64
            tmem_holding_buf: cutlass.Int32
            # (EPI_TILE_M, EPI_TILE_N, STAGE)
            sC: cute.struct.Align[
                cute.struct.MemRange[
                    self.c_dtype,
                    cute.cosize(self.epi_smem_layout_staged.outer),
                ],
                self.buffer_align_bytes,
            ]
            # (MMA, MMA_M, MMA_K, STAGE)
            sA: cute.struct.Align[
                cute.struct.MemRange[
                    self.a_dtype, cute.cosize(self.a_smem_layout_staged.outer)
                ],
                self.buffer_align_bytes,
            ]
            # (MMA, MMA_N, MMA_K, STAGE)
            sB: cute.struct.Align[
                cute.struct.MemRange[
                    self.b_dtype, cute.cosize(self.b_smem_layout_staged.outer)
````
- **EN**: Initializes or updates values such as `tensormap_buffer`, `ab_full_mbar_ptr`, `ab_empty_mbar_ptr`, `acc_full_mbar_ptr`, `acc_empty_mbar_ptr`, `tmem_dealloc_mbar_ptr`, and `...+4`. This range continues the implementation of class `GroupedGemmKernel.__call__.SharedStorage`.
- **CN**: 初始化或更新了 `tensormap_buffer`、`ab_full_mbar_ptr`、`ab_empty_mbar_ptr`、`acc_full_mbar_ptr`、`acc_empty_mbar_ptr`、`tmem_dealloc_mbar_ptr`、`另有4项` 等值。这一段延续了类`GroupedGemmKernel.__call__.SharedStorage` 的具体实现。

### Lines 449-476 / 第 449-476 行
````python
                ],
                self.buffer_align_bytes,
            ]

        self.shared_storage = SharedStorage

        # Launch the kernel synchronously
        self.kernel(
            tiled_mma,
            tma_atom_a,
            tma_tensor_a,
            tma_atom_b,
            tma_tensor_b,
            tma_atom_c,
            tma_tensor_c,
            self.cluster_layout_vmnk,
            self.a_smem_layout_staged,
            self.b_smem_layout_staged,
            self.epi_smem_layout_staged,
            self.epi_tile,
            self.tile_sched_params,
            group_count,
            problem_shape_mnkl,
            strides_abc,
            tensor_address_abc,
            tensormap_cute_tensor,
        ).launch(
            grid=grid,
````
- **EN**: Initializes or updates values such as `grid`. This range continues the implementation of function `GroupedGemmKernel.__call__`.
- **CN**: 初始化或更新了 `grid` 等值。这一段延续了函数`GroupedGemmKernel.__call__` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
            block=[self.threads_per_cta, 1, 1],
            cluster=(*self.cluster_shape_mn, 1),
            stream=stream,
            min_blocks_per_mp=1,
        )
        return

    # GPU device kernel
    @cute.kernel
    def kernel(
        self,
        tiled_mma: cute.TiledMma,
        tma_atom_a: cute.CopyAtom,
        mA_mkl: cute.Tensor,
        tma_atom_b: cute.CopyAtom,
        mB_nkl: cute.Tensor,
        tma_atom_c: cute.CopyAtom,
        mC_mnl: cute.Tensor,
        cluster_layout_vmnk: cute.Layout,
        a_smem_layout_staged: cute.ComposedLayout,
        b_smem_layout_staged: cute.ComposedLayout,
        epi_smem_layout_staged: Union[cute.Layout, cute.ComposedLayout],
        epi_tile: cute.Tile,
        tile_sched_params: utils.PersistentTileSchedulerParams,
        group_count: cutlass.Constexpr[int],
        problem_sizes_mnkl: cute.Tensor,
        strides_abc: cute.Tensor,
        ptrs_abc: cute.Tensor,
````
- **EN**: Introduces function `kernel`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`kernel`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 505-532 / 第 505-532 行
````python
        tensormaps: cute.Tensor,
    ):
        """
        GPU device kernel performing the grouped GEMM computation.
        """
        warp_idx = cute.arch.warp_idx()
        warp_idx = cute.arch.make_warp_uniform(warp_idx)

        #
        # Prefetch tma desc
        #
        if warp_idx == self.tma_warp_id:
            cpasync.prefetch_descriptor(tma_atom_a)
            cpasync.prefetch_descriptor(tma_atom_b)
            cpasync.prefetch_descriptor(tma_atom_c)

        use_2cta_instrs = cute.size(tiled_mma.thr_id.shape) == 2

        #
        # Setup cta/thread coordinates
        #
        # Coord inside cluster
        bid = cute.arch.block_idx()
        mma_tile_coord_v = bid[0] % cute.size(tiled_mma.thr_id.shape)
        is_leader_cta = mma_tile_coord_v == 0
        cta_rank_in_cluster = cute.arch.make_warp_uniform(
            cute.arch.block_idx_in_cluster()
        )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensormaps`, `warp_idx`, `use_2cta_instrs`, `bid`, `mma_tile_coord_v`, `is_leader_cta`, and `...+1`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `tensormaps`、`warp_idx`、`use_2cta_instrs`、`bid`、`mma_tile_coord_v`、`is_leader_cta`、`另有1项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python
        block_in_cluster_coord_vmnk = cluster_layout_vmnk.get_flat_coord(
            cta_rank_in_cluster
        )
        # Coord inside cta
        tidx, _, _ = cute.arch.thread_idx()

        #
        # Alloc and init: tensormap buffer, a+b full/empty, accumulator full/empty, tensor memory dealloc barrier
        #
        smem = utils.SmemAllocator()
        storage = smem.allocate(self.shared_storage)

        tensormap_a_smem_ptr = None
        tensormap_b_smem_ptr = None
        tensormap_c_smem_ptr = None
        if cutlass.const_expr(
            self.tensormap_update_mode == utils.TensorMapUpdateMode.SMEM
        ):
            tensormap_smem_ptr = storage.tensormap_buffer.data_ptr()
            tensormap_a_smem_ptr = tensormap_smem_ptr
            tensormap_b_smem_ptr = (
                tensormap_a_smem_ptr + GroupedGemmKernel.bytes_per_tensormap // 8
            )
            tensormap_c_smem_ptr = (
                tensormap_b_smem_ptr + GroupedGemmKernel.bytes_per_tensormap // 8
            )

        #  init barrier for loading A, B with TMA
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `block_in_cluster_coord_vmnk`, `smem`, `storage`, `tensormap_a_smem_ptr`, `tensormap_b_smem_ptr`, `tensormap_c_smem_ptr`, and `...+1`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `block_in_cluster_coord_vmnk`、`smem`、`storage`、`tensormap_a_smem_ptr`、`tensormap_b_smem_ptr`、`tensormap_c_smem_ptr`、`另有1项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 561-588 / 第 561-588 行
````python
        ab_pipeline_producer_group = pipeline.CooperativeGroup(pipeline.Agent.Thread)
        num_tma_producer = self.num_mcast_ctas_a + self.num_mcast_ctas_b - 1
        ab_pipeline_consumer_group = pipeline.CooperativeGroup(
            pipeline.Agent.Thread, num_tma_producer
        )
        ab_pipeline = pipeline.PipelineTmaUmma.create(
            barrier_storage=storage.ab_full_mbar_ptr.data_ptr(),
            num_stages=self.num_ab_stage,
            producer_group=ab_pipeline_producer_group,
            consumer_group=ab_pipeline_consumer_group,
            tx_count=self.num_tma_load_bytes,
            cta_layout_vmnk=cluster_layout_vmnk,
            defer_sync=True,
        )
        # Accumulator barrier init
        acc_pipeline_producer_group = pipeline.CooperativeGroup(pipeline.Agent.Thread)
        num_acc_consumer_threads = len(self.epilog_warp_id) * (
            2 if use_2cta_instrs else 1
        )
        acc_pipeline_consumer_group = pipeline.CooperativeGroup(
            pipeline.Agent.Thread, num_acc_consumer_threads
        )
        acc_pipeline = pipeline.PipelineUmmaAsync.create(
            barrier_storage=storage.acc_full_mbar_ptr.data_ptr(),
            num_stages=self.num_acc_stage,
            producer_group=acc_pipeline_producer_group,
            consumer_group=acc_pipeline_consumer_group,
            cta_layout_vmnk=cluster_layout_vmnk,
````
- **EN**: Initializes or updates values such as `ab_pipeline_producer_group`, `num_tma_producer`, `ab_pipeline_consumer_group`, `ab_pipeline`, `barrier_storage`, `num_stages`, and `...+9`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `ab_pipeline_producer_group`、`num_tma_producer`、`ab_pipeline_consumer_group`、`ab_pipeline`、`barrier_storage`、`num_stages`、`另有9项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 589-616 / 第 589-616 行
````python
            defer_sync=True,
        )
        # Tensor memory dealloc barrier init
        tmem = utils.TmemAllocator(
            storage.tmem_holding_buf,
            barrier_for_retrieve=self.tmem_alloc_barrier,
            allocator_warp_id=self.epilog_warp_id[0],
            is_two_cta=use_2cta_instrs,
            two_cta_tmem_dealloc_mbar_ptr=storage.tmem_dealloc_mbar_ptr,
        )

        # Cluster arrive after barrier init
        pipeline_init_arrive(cluster_shape_mn=self.cluster_shape_mn, is_relaxed=True)

        #
        # Setup smem tensor A/B/C
        #
        # (EPI_TILE_M, EPI_TILE_N, STAGE)
        sC = storage.sC.get_tensor(
            epi_smem_layout_staged.outer, swizzle=epi_smem_layout_staged.inner
        )
        # (MMA, MMA_M, MMA_K, STAGE)
        sA = storage.sA.get_tensor(
            a_smem_layout_staged.outer, swizzle=a_smem_layout_staged.inner
        )
        # (MMA, MMA_N, MMA_K, STAGE)
        sB = storage.sB.get_tensor(
            b_smem_layout_staged.outer, swizzle=b_smem_layout_staged.inner
````
- **EN**: Initializes or updates values such as `defer_sync`, `tmem`, `barrier_for_retrieve`, `allocator_warp_id`, `is_two_cta`, `two_cta_tmem_dealloc_mbar_ptr`, and `...+3`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `defer_sync`、`tmem`、`barrier_for_retrieve`、`allocator_warp_id`、`is_two_cta`、`two_cta_tmem_dealloc_mbar_ptr`、`另有3项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 617-644 / 第 617-644 行
````python
        )

        #
        # Compute multicast mask for A/B buffer full and empty
        #
        a_full_mcast_mask = None
        b_full_mcast_mask = None
        ab_empty_mcast_mask = None
        if cutlass.const_expr(self.is_a_mcast or self.is_b_mcast or use_2cta_instrs):
            a_full_mcast_mask = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mcast_mode=2
            )
            b_full_mcast_mask = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mcast_mode=1
            )
            ab_empty_mcast_mask = a_full_mcast_mask | b_full_mcast_mask
        acc_full_mcast_mask = None
        if cutlass.const_expr(use_2cta_instrs):
            acc_full_mcast_mask = cute.make_layout_image_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mode=0
            )
            block_in_cluster_coord_vmnk_peer = (
                block_in_cluster_coord_vmnk[0] ^ 1,
                *block_in_cluster_coord_vmnk[1:],
            )
            a_full_mcast_mask_peer = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk_peer, mcast_mode=2
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `a_full_mcast_mask`, `b_full_mcast_mask`, `ab_empty_mcast_mask`, `acc_full_mcast_mask`, `block_in_cluster_coord_vmnk_peer`, and `a_full_mcast_mask_peer`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `a_full_mcast_mask`、`b_full_mcast_mask`、`ab_empty_mcast_mask`、`acc_full_mcast_mask`、`block_in_cluster_coord_vmnk_peer`、`a_full_mcast_mask_peer` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 645-672 / 第 645-672 行
````python
            b_full_mcast_mask_peer = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk_peer, mcast_mode=1
            )
            ab_empty_mcast_mask = (
                a_full_mcast_mask_peer
                | b_full_mcast_mask_peer
                | cutlass.Int16(
                    0 if ab_empty_mcast_mask is None else ab_empty_mcast_mask
                )
            )

        #
        # Local_tile partition global tensors
        #
        # (bM, bK, RestM, RestK, RestL)
        gA_mkl = cute.local_tile(
            mA_mkl, cute.slice_(self.mma_tiler, (None, 0, None)), (None, None, None)
        )
        # (bN, bK, RestN, RestK, RestL)
        gB_nkl = cute.local_tile(
            mB_nkl, cute.slice_(self.mma_tiler, (0, None, None)), (None, None, None)
        )
        # (bM, bN, RestM, RestN, RestL)
        gC_mnl = cute.local_tile(
            mC_mnl, cute.slice_(self.mma_tiler, (None, None, 0)), (None, None, None)
        )

        #
````
- **EN**: Initializes or updates values such as `b_full_mcast_mask_peer`, `ab_empty_mcast_mask`, `gA_mkl`, `gB_nkl`, and `gC_mnl`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `b_full_mcast_mask_peer`、`ab_empty_mcast_mask`、`gA_mkl`、`gB_nkl`、`gC_mnl` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
        # Partition global tensor for TiledMMA_A/B/C
        #
        thr_mma = tiled_mma.get_slice(mma_tile_coord_v)
        # (MMA, MMA_M, MMA_K, RestM, RestK, RestL)
        tCgA = thr_mma.partition_A(gA_mkl)
        # (MMA, MMA_N, MMA_K, RestN, RestK, RestL)
        tCgB = thr_mma.partition_B(gB_nkl)
        # (MMA, MMA_M, MMA_N, RestM, RestN, RestL)
        tCgC = thr_mma.partition_C(gC_mnl)

        #
        # Partition global/shared tensor for load A, B with TMA
        #
        a_cta_layout = cute.make_layout(
            cute.slice_(cluster_layout_vmnk, (0, 0, None, 0)).shape
        )
        # ((atom_v, rest_v), STAGE)
        # ((atom_v, rest_v), RestM, RestK, RestL)
        tAsA, tAgA = cpasync.tma_partition(
            tma_atom_a,
            block_in_cluster_coord_vmnk[2],
            a_cta_layout,
            cute.group_modes(sA, 0, 3),
            cute.group_modes(tCgA, 0, 3),
        )
        # TMA load B partition_S/D
        b_cta_layout = cute.make_layout(
            cute.slice_(cluster_layout_vmnk, (0, None, 0, 0)).shape
````
- **EN**: Initializes or updates values such as `thr_mma`, `tCgA`, `tCgB`, `tCgC`, `a_cta_layout`, and `b_cta_layout`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `thr_mma`、`tCgA`、`tCgB`、`tCgC`、`a_cta_layout`、`b_cta_layout` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python
        )
        # ((atom_v, rest_v), STAGE)
        # ((atom_v, rest_v), RestM, RestK, RestL)
        tBsB, tBgB = cpasync.tma_partition(
            tma_atom_b,
            block_in_cluster_coord_vmnk[1],
            b_cta_layout,
            cute.group_modes(sB, 0, 3),
            cute.group_modes(tCgB, 0, 3),
        )

        #
        # Partition shared/tensor memory tensor for TiledMMA_A/B/C
        #
        # (MMA, MMA_M, MMA_K, STAGE)
        tCrA = tiled_mma.make_fragment_A(sA)
        # (MMA, MMA_N, MMA_K, STAGE)
        tCrB = tiled_mma.make_fragment_B(sB)
        # (MMA, MMA_M, MMA_N)
        acc_shape = tiled_mma.partition_shape_C(self.mma_tiler[:2])
        # (MMA, MMA_M, MMA_N, STAGE)
        tCtAcc_fake = tiled_mma.make_fragment_C(
            cute.append(acc_shape, self.num_acc_stage)
        )

        #
        # Cluster wait before tensor memory alloc
        #
````
- **EN**: Initializes or updates values such as `tCrA`, `tCrB`, `acc_shape`, and `tCtAcc_fake`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `tCrA`、`tCrB`、`acc_shape`、`tCtAcc_fake` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python
        pipeline_init_wait(cluster_shape_mn=self.cluster_shape_mn)

        #
        # Get tensormap buffer address
        #
        grid_dim = cute.arch.grid_dim()
        tensormap_workspace_idx = (
            bid[2] * grid_dim[1] * grid_dim[0] + bid[1] * grid_dim[0] + bid[0]
        )

        tensormap_manager = utils.TensorMapManager(
            self.tensormap_update_mode, GroupedGemmKernel.bytes_per_tensormap
        )
        tensormap_a_ptr = tensormap_manager.get_tensormap_ptr(
            tensormaps[(tensormap_workspace_idx, 0, None)].iterator
        )
        tensormap_b_ptr = tensormap_manager.get_tensormap_ptr(
            tensormaps[(tensormap_workspace_idx, 1, None)].iterator
        )
        tensormap_c_ptr = tensormap_manager.get_tensormap_ptr(
            tensormaps[(tensormap_workspace_idx, 2, None)].iterator
        )
        # Setup tensormap initialization pointer based on the mode
        if cutlass.const_expr(
            self.tensormap_update_mode == utils.TensorMapUpdateMode.SMEM
        ):
            tensormap_a_init_ptr = tensormap_a_smem_ptr
            tensormap_b_init_ptr = tensormap_b_smem_ptr
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `grid_dim`, `tensormap_workspace_idx`, `tensormap_manager`, `tensormap_a_ptr`, `tensormap_b_ptr`, `tensormap_c_ptr`, and `...+2`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `grid_dim`、`tensormap_workspace_idx`、`tensormap_manager`、`tensormap_a_ptr`、`tensormap_b_ptr`、`tensormap_c_ptr`、`另有2项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 757-784 / 第 757-784 行
````python
            tensormap_c_init_ptr = tensormap_c_smem_ptr
        else:
            tensormap_a_init_ptr = tensormap_a_ptr
            tensormap_b_init_ptr = tensormap_b_ptr
            tensormap_c_init_ptr = tensormap_c_ptr

        #
        # Persistent tile scheduling loop
        #
        # When the problem shapes are on device, we launch one CTA per SM.
        # The if condition later prevents the warps from extra CTAs from doing any work.
        tile_sched = utils.StaticPersistentGroupTileScheduler.create(
            tile_sched_params,
            bid,
            grid_dim,
            self.cluster_tile_shape_mnk,
            utils.create_initial_search_state(),
            group_count,
            problem_sizes_mnkl,
        )
        initial_work_tile_info = tile_sched.initial_work_tile_info()

        #
        # Specialized TMA load warp
        #
        if warp_idx == self.tma_warp_id and initial_work_tile_info.is_valid_tile:
            # Initialize tensormaps for A, B
            if cutlass.const_expr(self.delegate_tensormap_ab_init == False):
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensormap_c_init_ptr`, `else`, `tensormap_a_init_ptr`, `tensormap_b_init_ptr`, `tile_sched`, and `initial_work_tile_info`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `tensormap_c_init_ptr`、`else`、`tensormap_a_init_ptr`、`tensormap_b_init_ptr`、`tile_sched`、`initial_work_tile_info` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 785-812 / 第 785-812 行
````python
                tensormap_manager.init_tensormap_from_atom(
                    tma_atom_a, tensormap_a_init_ptr, self.tma_warp_id
                )
                tensormap_manager.init_tensormap_from_atom(
                    tma_atom_b, tensormap_b_init_ptr, self.tma_warp_id
                )

            tensormap_init_done = cutlass.Boolean(False)
            # group index of last tile
            last_group_idx = cutlass.Int32(-1)

            work_tile = initial_work_tile_info
            ab_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_ab_stage
            )

            while work_tile.is_valid_tile:
                grouped_gemm_cta_tile_info = work_tile.group_search_result

                cur_k_tile_cnt = grouped_gemm_cta_tile_info.cta_tile_count_k
                is_k_tile_cnt_zero = cur_k_tile_cnt == 0
                cur_group_idx = grouped_gemm_cta_tile_info.group_idx
                # Do not load any data if cur_k_tile_cnt is 0
                if not is_k_tile_cnt_zero:
                    is_group_changed = cur_group_idx != last_group_idx
                    # skip tensormap update if we're working on the same group
                    if is_group_changed:
                        real_tensor_a = self.make_tensor_for_tensormap_update(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensormap_init_done`, `last_group_idx`, `work_tile`, `ab_producer_state`, `grouped_gemm_cta_tile_info`, `cur_k_tile_cnt`, and `...+4`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tensormap_init_done`、`last_group_idx`、`work_tile`、`ab_producer_state`、`grouped_gemm_cta_tile_info`、`cur_k_tile_cnt`、`另有4项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
                            cur_group_idx,
                            self.a_dtype,
                            (
                                grouped_gemm_cta_tile_info.problem_shape_m,
                                grouped_gemm_cta_tile_info.problem_shape_n,
                                grouped_gemm_cta_tile_info.problem_shape_k,
                            ),
                            strides_abc,
                            ptrs_abc,
                            0,  # 0 for tensor A
                        )
                        real_tensor_b = self.make_tensor_for_tensormap_update(
                            cur_group_idx,
                            self.b_dtype,
                            (
                                grouped_gemm_cta_tile_info.problem_shape_m,
                                grouped_gemm_cta_tile_info.problem_shape_n,
                                grouped_gemm_cta_tile_info.problem_shape_k,
                            ),
                            strides_abc,
                            ptrs_abc,
                            1,  # 1 for tensor B
                        )
                        # wait tensormap initialization complete before update
                        if not tensormap_init_done:
                            if cutlass.const_expr(self.delegate_tensormap_ab_init):
                                self.tensormap_ab_init_barrier.arrive_and_wait()
                            tensormap_manager.fence_tensormap_initialization()
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `real_tensor_b`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `real_tensor_b` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
                            tensormap_init_done = True

                        tensormap_manager.update_tensormap(
                            (real_tensor_a, real_tensor_b),
                            (tma_atom_a, tma_atom_b),
                            (tensormap_a_ptr, tensormap_b_ptr),
                            self.tma_warp_id,
                            (tensormap_a_smem_ptr, tensormap_b_smem_ptr),
                        )

                    mma_tile_coord_mnl = (
                        grouped_gemm_cta_tile_info.cta_tile_idx_m
                        // cute.size(tiled_mma.thr_id.shape),
                        grouped_gemm_cta_tile_info.cta_tile_idx_n,
                        0,
                    )

                    #
                    # Slice to per mma tile index
                    #
                    # ((atom_v, rest_v), RestK)
                    tAgA_slice = tAgA[
                        (None, mma_tile_coord_mnl[0], None, mma_tile_coord_mnl[2])
                    ]
                    # ((atom_v, rest_v), RestK)
                    tBgB_slice = tBgB[
                        (None, mma_tile_coord_mnl[1], None, mma_tile_coord_mnl[2])
                    ]
````
- **EN**: Initializes or updates values such as `tensormap_init_done`, `mma_tile_coord_mnl`, `tAgA_slice`, and `tBgB_slice`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `tensormap_init_done`、`mma_tile_coord_mnl`、`tAgA_slice`、`tBgB_slice` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 869-896 / 第 869-896 行
````python

                    # Peek (try_wait) AB buffer empty for k_tile = prefetch_k_tile_cnt
                    ab_producer_state.reset_count()
                    peek_ab_empty_status = cutlass.Boolean(1)
                    if ab_producer_state.count < cur_k_tile_cnt:
                        peek_ab_empty_status = ab_pipeline.producer_try_acquire(
                            ab_producer_state
                        )
                    # ensure the update to tensormap has completed before using it
                    if is_group_changed:
                        tensormap_manager.fence_tensormap_update(tensormap_a_ptr)
                        tensormap_manager.fence_tensormap_update(tensormap_b_ptr)
                        #
                        # Tma load loop
                        #
                    for k_tile in cutlass.range(0, cur_k_tile_cnt, 1, unroll=1):
                        # Wait for AB buffer empty
                        ab_pipeline.producer_acquire(
                            ab_producer_state, peek_ab_empty_status
                        )

                        # Load A/B with TMA
                        cute.copy(
                            tma_atom_a,
                            tAgA_slice[(None, ab_producer_state.count)],
                            tAsA[(None, ab_producer_state.index)],
                            tma_bar_ptr=ab_pipeline.producer_get_barrier(
                                ab_producer_state
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `peek_ab_empty_status`, and `tma_bar_ptr`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `peek_ab_empty_status`、`tma_bar_ptr` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 897-924 / 第 897-924 行
````python
                            ),
                            mcast_mask=a_full_mcast_mask,
                            tma_desc_ptr=tensormap_manager.get_tensormap_ptr(
                                tensormap_a_ptr,
                                cute.AddressSpace.generic,
                            ),
                        )
                        cute.copy(
                            tma_atom_b,
                            tBgB_slice[(None, ab_producer_state.count)],
                            tBsB[(None, ab_producer_state.index)],
                            tma_bar_ptr=ab_pipeline.producer_get_barrier(
                                ab_producer_state
                            ),
                            mcast_mask=b_full_mcast_mask,
                            tma_desc_ptr=tensormap_manager.get_tensormap_ptr(
                                tensormap_b_ptr,
                                cute.AddressSpace.generic,
                            ),
                        )

                        # Peek (try_wait) AB buffer empty for k_tile = prefetch_k_tile_cnt + k_tile + 1
                        ab_producer_state.advance()
                        peek_ab_empty_status = cutlass.Boolean(1)
                        if ab_producer_state.count < cur_k_tile_cnt:
                            peek_ab_empty_status = ab_pipeline.producer_try_acquire(
                                ab_producer_state
                            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mcast_mask`, `tma_desc_ptr`, `tma_bar_ptr`, and `peek_ab_empty_status`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `mcast_mask`、`tma_desc_ptr`、`tma_bar_ptr`、`peek_ab_empty_status` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 925-952 / 第 925-952 行
````python
                else:
                    # If tensormap initialization is not done, wait for it to complete
                    if not tensormap_init_done:
                        if cutlass.const_expr(self.delegate_tensormap_ab_init):
                            self.tensormap_ab_init_barrier.arrive_and_wait()
                        tensormap_manager.fence_tensormap_initialization()
                        tensormap_init_done = True
                # Advance to next tile
                tile_sched.advance_to_next_work()
                work_tile = tile_sched.get_current_work()
                last_group_idx = cur_group_idx

            #
            # Wait A/B buffer empty
            #
            ab_pipeline.producer_tail(ab_producer_state)

        #
        # Specialized MMA warp
        #
        if warp_idx == self.mma_warp_id and initial_work_tile_info.is_valid_tile:
            #  Bar sync for retrieve tmem ptr from shared mem
            tmem.wait_for_alloc()

            #
            # Retrieving tensor memory ptr and make accumulator tensor
            #
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `tensormap_init_done`, `work_tile`, `last_group_idx`, and `tmem_ptr`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`tensormap_init_done`、`work_tile`、`last_group_idx`、`tmem_ptr` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
            # (MMA, MMA_M, MMA_N, STAGE)
            tCtAcc_base = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)

            #
            # Persistent tile scheduling loop
            #
            work_tile = initial_work_tile_info
            ab_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_ab_stage
            )
            acc_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_acc_stage
            )

            # tile count we have searched
            while work_tile.is_valid_tile:
                cur_group_idx = work_tile.group_search_result.group_idx
                problem_shape_k = work_tile.group_search_result.problem_shape_k

                # MMA warp is only interested in number of tiles along K dimension
                cur_k_tile_cnt = (
                    problem_shape_k + self.cluster_tile_shape_mnk[2] - 1
                ) // self.cluster_tile_shape_mnk[2]
                is_k_tile_cnt_zero = cur_k_tile_cnt == 0

                # (MMA, MMA_M, MMA_N)
                tCtAcc = tCtAcc_base[(None, None, None, acc_producer_state.index)]

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tCtAcc_base`, `work_tile`, `ab_consumer_state`, `acc_producer_state`, `cur_group_idx`, `problem_shape_k`, and `...+3`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tCtAcc_base`、`work_tile`、`ab_consumer_state`、`acc_producer_state`、`cur_group_idx`、`problem_shape_k`、`另有3项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 981-1008 / 第 981-1008 行
````python
                # Peek (try_wait) AB buffer full for k_tile = 0
                ab_consumer_state.reset_count()
                peek_ab_full_status = cutlass.Boolean(1)
                if is_leader_cta:
                    if ab_consumer_state.count < cur_k_tile_cnt:
                        peek_ab_full_status = ab_pipeline.consumer_try_wait(
                            ab_consumer_state
                        )

                    #
                    # Wait for accumulator buffer empty
                    #
                    if not is_k_tile_cnt_zero:
                        acc_pipeline.producer_acquire(acc_producer_state)

                    #
                    # Reset the ACCUMULATE field for each tile
                    #
                    tiled_mma.set(tcgen05.Field.ACCUMULATE, False)

                    #
                    # Mma mainloop
                    #
                    for k_tile in cutlass.range(0, cur_k_tile_cnt, 1, unroll=1):
                        # Wait for AB buffer full
                        ab_pipeline.consumer_wait(
                            ab_consumer_state, peek_ab_full_status
                        )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `peek_ab_full_status`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `peek_ab_full_status` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1009-1036 / 第 1009-1036 行
````python
                        # tCtAcc += tCrA * tCrB
                        num_kblocks = cute.size(tCrA, mode=[2])
                        for kblock_idx in cutlass.range(num_kblocks, unroll_full=True):
                            kblock_coord = (
                                None,
                                None,
                                kblock_idx,
                                ab_consumer_state.index,
                            )

                            cute.gemm(
                                tiled_mma,
                                tCtAcc,
                                tCrA[kblock_coord],
                                tCrB[kblock_coord],
                                tCtAcc,
                            )
                            # Enable accumulate on tCtAcc after first kblock
                            tiled_mma.set(tcgen05.Field.ACCUMULATE, True)

                        # Async arrive AB buffer empty
                        ab_pipeline.consumer_release(ab_consumer_state)

                        # Peek (try_wait) AB buffer full for k_tile = k_tile + 1
                        ab_consumer_state.advance()
                        peek_ab_full_status = cutlass.Boolean(1)
                        if ab_consumer_state.count < cur_k_tile_cnt:
                            peek_ab_full_status = ab_pipeline.consumer_try_wait(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_kblocks`, `kblock_coord`, and `peek_ab_full_status`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `num_kblocks`、`kblock_coord`、`peek_ab_full_status` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python
                                ab_consumer_state
                            )

                    #
                    # Async arrive accumulator buffer full
                    #
                    if not is_k_tile_cnt_zero:
                        acc_pipeline.producer_commit(acc_producer_state)
                        acc_producer_state.advance()

                #
                # Advance to next tile
                #
                tile_sched.advance_to_next_work()
                work_tile = tile_sched.get_current_work()
            #
            # Wait for accumulator buffer empty
            #
            acc_pipeline.producer_tail(acc_producer_state)

        #
        # Specialized epilogue warps
        #
        if warp_idx < self.mma_warp_id and initial_work_tile_info.is_valid_tile:
            # initialize tensormap A, B for TMA warp
            if cutlass.const_expr(self.delegate_tensormap_ab_init):
                tensormap_manager.init_tensormap_from_atom(
                    tma_atom_a, tensormap_a_init_ptr, self.epilog_warp_id[0]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `work_tile`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `work_tile` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1065-1092 / 第 1065-1092 行
````python
                )
                tensormap_manager.init_tensormap_from_atom(
                    tma_atom_b, tensormap_b_init_ptr, self.epilog_warp_id[0]
                )
                # signal tensormap initialization has finished
                self.tensormap_ab_init_barrier.arrive_and_wait()
            # initialize tensorap for C
            tensormap_manager.init_tensormap_from_atom(
                tma_atom_c,
                tensormap_c_init_ptr,
                self.epilog_warp_id[0],
            )
            # Alloc tensor memory buffer
            tmem.allocate(self.num_tmem_alloc_cols)

            #
            # Bar sync for retrieve tensor memory ptr from shared memory
            #
            tmem.wait_for_alloc()

            #
            # Retrieving tensor memory ptr and make accumulator tensor
            #
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
            # (MMA, MMA_M, MMA_N, STAGE)
            tCtAcc_base = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)

            epi_tidx = tidx
````
- **EN**: Initializes or updates values such as `tmem_ptr`, `tCtAcc_base`, and `epi_tidx`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `tmem_ptr`、`tCtAcc_base`、`epi_tidx` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1093-1120 / 第 1093-1120 行
````python
            #
            # Partition for epilogue
            #
            (
                tiled_copy_t2r,
                tTR_tAcc_base,
                tTR_rAcc,
            ) = self.epilog_tmem_copy_and_partition(
                epi_tidx, tCtAcc_base, tCgC, epi_tile, use_2cta_instrs
            )

            tTR_rC = cute.make_rmem_tensor(tTR_rAcc.shape, self.c_dtype)
            tiled_copy_r2s, tRS_rC, tRS_sC = self.epilog_smem_copy_and_partition(
                tiled_copy_t2r, tTR_rC, epi_tidx, sC
            )
            (
                tma_atom_c,
                bSG_sC,
                bSG_gC_partitioned,
            ) = self.epilog_gmem_copy_and_partition(tma_atom_c, tCgC, epi_tile, sC)

            #
            # Persistent tile scheduling loop
            #

            work_tile = initial_work_tile_info

            # wait tensormap initialization complete before update
````
- **EN**: Initializes or updates values such as `tTR_rC`, and `work_tile`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `tTR_rC`、`work_tile` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1121-1148 / 第 1121-1148 行
````python
            tensormap_manager.fence_tensormap_initialization()
            acc_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_acc_stage
            )
            # Threads/warps participating in tma store pipeline
            c_producer_group = pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                32 * len(self.epilog_warp_id),
            )
            c_pipeline = pipeline.PipelineTmaStore.create(
                num_stages=self.num_epi_stage,
                producer_group=c_producer_group,
            )
            # group index of last tile
            last_group_idx = cutlass.Int32(-1)
            while work_tile.is_valid_tile:
                grouped_gemm_cta_tile_info = work_tile.group_search_result
                cur_group_idx = grouped_gemm_cta_tile_info.group_idx
                cur_k_tile_cnt = grouped_gemm_cta_tile_info.cta_tile_count_k
                is_k_tile_cnt_zero = cur_k_tile_cnt == 0
                is_group_changed = cur_group_idx != last_group_idx
                # We still need to store 0s when k_tile_cnt is 0
                if is_group_changed:
                    # construct tensor C based on real address, shape and stride information
                    real_tensor_c = self.make_tensor_for_tensormap_update(
                        cur_group_idx,
                        self.c_dtype,
                        (
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `acc_consumer_state`, `c_producer_group`, `c_pipeline`, `num_stages`, `producer_group`, `last_group_idx`, and `...+6`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `acc_consumer_state`、`c_producer_group`、`c_pipeline`、`num_stages`、`producer_group`、`last_group_idx`、`另有6项` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1149-1176 / 第 1149-1176 行
````python
                            grouped_gemm_cta_tile_info.problem_shape_m,
                            grouped_gemm_cta_tile_info.problem_shape_n,
                            grouped_gemm_cta_tile_info.problem_shape_k,
                        ),
                        strides_abc,
                        ptrs_abc,
                        2,  # 2 for tensor C
                    )
                    tensormap_manager.update_tensormap(
                        ((real_tensor_c),),
                        ((tma_atom_c),),
                        ((tensormap_c_ptr),),
                        self.epilog_warp_id[0],
                        (tensormap_c_smem_ptr,),
                    )

                mma_tile_coord_mnl = (
                    grouped_gemm_cta_tile_info.cta_tile_idx_m
                    // cute.size(tiled_mma.thr_id.shape),
                    grouped_gemm_cta_tile_info.cta_tile_idx_n,
                    0,
                )

                #
                # Slice to per mma tile index
                #
                # ((ATOM_V, REST_V), EPI_M, EPI_N)
                bSG_gC = bSG_gC_partitioned[
````
- **EN**: Initializes or updates values such as `mma_tile_coord_mnl`, and `bSG_gC`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 初始化或更新了 `mma_tile_coord_mnl`、`bSG_gC` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1177-1204 / 第 1177-1204 行
````python
                    (
                        None,
                        None,
                        None,
                        *mma_tile_coord_mnl,
                    )
                ]
                # (T2R, T2R_M, T2R_N, EPI_M, EPI_M)
                tTR_tAcc = tTR_tAcc_base[
                    (None, None, None, None, None, acc_consumer_state.index)
                ]
                #
                # Wait for accumulator buffer full
                #
                # Not waiting for accumulator buffer full when k_tile_cnt is 0
                if not is_k_tile_cnt_zero:
                    acc_pipeline.consumer_wait(acc_consumer_state)

                tTR_tAcc = cute.group_modes(tTR_tAcc, 3, cute.rank(tTR_tAcc))
                bSG_gC = cute.group_modes(bSG_gC, 1, cute.rank(bSG_gC))
                # ensure the update to tensormap has completed before using it
                if is_group_changed:
                    if warp_idx == self.epilog_warp_id[0]:
                        tensormap_manager.fence_tensormap_update(tensormap_c_ptr)
                #
                # Store accumulator to global memory in subtiles
                #
                subtile_cnt = cute.size(tTR_tAcc.shape, mode=[3])
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tTR_tAcc`, `bSG_gC`, and `subtile_cnt`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tTR_tAcc`、`bSG_gC`、`subtile_cnt` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1205-1232 / 第 1205-1232 行
````python
                num_prev_subtiles = tile_sched.num_tiles_executed * subtile_cnt
                for subtile_idx in range(subtile_cnt):
                    #
                    # Store C to shared memory
                    #
                    epi_buffer = (num_prev_subtiles + subtile_idx) % self.num_epi_stage
                    #
                    # Load accumulator from tensor memory buffer to register
                    #
                    tTR_tAcc_mn = tTR_tAcc[(None, None, None, subtile_idx)]
                    if not is_k_tile_cnt_zero:
                        cute.copy(tiled_copy_t2r, tTR_tAcc_mn, tTR_rAcc)

                        #
                        # Convert to output type
                        #
                        acc_vec = tiled_copy_r2s.retile(tTR_rAcc).load()
                        tRS_rC.store(acc_vec.to(self.c_dtype))
                    else:
                        tRS_rC.fill(0)
                    cute.copy(
                        tiled_copy_r2s,
                        tRS_rC,
                        tRS_sC[(None, None, None, epi_buffer)],
                    )
                    # Fence and barrier to make sure shared memory store is visible to TMA store
                    cute.arch.fence_proxy(
                        "async.shared",
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_prev_subtiles`, `epi_buffer`, `tTR_tAcc_mn`, `acc_vec`, and `else`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_prev_subtiles`、`epi_buffer`、`tTR_tAcc_mn`、`acc_vec`、`else` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1233-1260 / 第 1233-1260 行
````python
                        space="cta",
                    )
                    self.epilog_sync_barrier.arrive_and_wait()
                    #
                    # store C to global memory with TMA
                    #
                    if warp_idx == self.epilog_warp_id[0]:
                        cute.copy(
                            tma_atom_c,
                            bSG_sC[(None, epi_buffer)],
                            bSG_gC[(None, subtile_idx)],
                            tma_desc_ptr=tensormap_manager.get_tensormap_ptr(
                                tensormap_c_ptr,
                                cute.AddressSpace.generic,
                            ),
                        )
                        # Fence and barrier to make sure shared memory store is visible to TMA store
                        c_pipeline.producer_commit()
                        c_pipeline.producer_acquire()
                    self.epilog_sync_barrier.arrive_and_wait()
                #
                # Async arrive accumulator buffer empty
                #
                if not is_k_tile_cnt_zero:
                    with cute.arch.elect_one():
                        acc_pipeline.consumer_release(acc_consumer_state)
                    acc_consumer_state.advance()

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `space`, and `tma_desc_ptr`. This range continues the implementation of function `GroupedGemmKernel.kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `space`、`tma_desc_ptr` 等值。这一段延续了函数`GroupedGemmKernel.kernel` 的具体实现。

### Lines 1261-1288 / 第 1261-1288 行
````python
                #
                # Advance to next tile
                #
                tile_sched.advance_to_next_work()
                work_tile = tile_sched.get_current_work()
                last_group_idx = cur_group_idx

            #
            # Dealloc the tensor memory buffer
            #
            tmem.relinquish_alloc_permit()
            self.epilog_sync_barrier.arrive_and_wait()
            tmem.free(tmem_ptr)

            #
            # Wait for C store complete
            #
            c_pipeline.producer_tail()

    @cute.jit
    def make_tensor_for_tensormap_update(
        self,
        group_idx: cutlass.Int32,
        dtype: Type[cutlass.Numeric],
        problem_shape_mnk: tuple[cutlass.Int32, cutlass.Int32, cutlass.Int32],
        strides_abc: cute.Tensor,
        tensor_address_abc: cute.Tensor,
        tensor_index: int,
````
- **EN**: Introduces function `make_tensor_for_tensormap_update`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `work_tile`, `last_group_idx`, `group_idx`, `dtype`, `problem_shape_mnk`, `strides_abc`, and `...+2`.
- **CN**: 这里定义了函数`make_tensor_for_tensormap_update`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `work_tile`、`last_group_idx`、`group_idx`、`dtype`、`problem_shape_mnk`、`strides_abc`、`另有2项` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
    ):
        """Extract stride and tensor address for a given group and construct a global tensor.

        This function is used within the kernel to dynamically create a CUTE tensor
        representing A, B, or C for the current group being processed, using the
        group-specific address, shape, and stride information.

        :param group_idx: The index of the current group within the grouped GEMM.
        :type group_idx: cutlass.Int32
        :param dtype: The data type of the tensor elements (e.g., cutlass.Float16).
        :type dtype: Type[cutlass.Numeric]
        :param problem_shape_mnk: The (M, N, K) problem shape for the current group.
        :type problem_shape_mnk: tuple[cutlass.Int32, cutlass.Int32, cutlass.Int32]
        :param strides_abc: Tensor containing strides for A, B, C for all groups. Layout: (group_count, 3, 2).
        :type strides_abc: cute.Tensor
        :param tensor_address_abc: Tensor containing global memory addresses for A, B, C for all groups. Layout: (group_count, 3).
        :type tensor_address_abc: cute.Tensor
        :param tensor_index: Specifies which tensor to create: 0 for A, 1 for B, 2 for C.
        :type tensor_index: int
        :return: A CUTE tensor representing the requested global memory tensor (A, B, or C) for the specified group.
        :rtype: cute.Tensor
        :raises TypeError: If the provided dtype is not a subclass of cutlass.Numeric.
        """
        ptr_i64 = tensor_address_abc[(group_idx, tensor_index)]
        if cutlass.const_expr(
            not isclass(dtype) or not issubclass(dtype, cutlass.Numeric)
        ):
            raise TypeError(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ptr_i64`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ptr_i64` 等值。

### Lines 1317-1344 / 第 1317-1344 行
````python
                f"dtype must be a type of cutlass.Numeric, got {type(dtype)}"
            )
        tensor_gmem_ptr = cute.make_ptr(
            dtype, ptr_i64, cute.AddressSpace.gmem, assumed_align=16
        )

        strides_tensor_gmem = strides_abc[(group_idx, tensor_index, None)]
        strides_tensor_reg = cute.make_rmem_tensor(
            cute.make_layout(2),
            strides_abc.element_type,
        )
        cute.autovec_copy(strides_tensor_gmem, strides_tensor_reg)
        stride_mn = strides_tensor_reg[0]
        stride_k = strides_tensor_reg[1]
        c1 = cutlass.Int32(1)
        c0 = cutlass.Int32(0)

        if cutlass.const_expr(tensor_index == 0):  # tensor A
            m = problem_shape_mnk[0]
            k = problem_shape_mnk[2]
            return cute.make_tensor(
                tensor_gmem_ptr,
                cute.make_layout((m, k, c1), stride=(stride_mn, stride_k, c0)),
            )
        elif cutlass.const_expr(tensor_index == 1):  # tensor B
            n = problem_shape_mnk[1]
            k = problem_shape_mnk[2]
            return cute.make_tensor(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensor_gmem_ptr`, `strides_tensor_gmem`, `strides_tensor_reg`, `stride_mn`, `stride_k`, `c1`, and `...+4`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tensor_gmem_ptr`、`strides_tensor_gmem`、`strides_tensor_reg`、`stride_mn`、`stride_k`、`c1`、`另有4项` 等值。

### Lines 1345-1372 / 第 1345-1372 行
````python
                tensor_gmem_ptr,
                cute.make_layout((n, k, c1), stride=(stride_mn, stride_k, c0)),
            )
        else:  # tensor C
            m = problem_shape_mnk[0]
            n = problem_shape_mnk[1]
            return cute.make_tensor(
                tensor_gmem_ptr,
                cute.make_layout((m, n, c1), stride=(stride_mn, stride_k, c0)),
            )

    def epilog_tmem_copy_and_partition(
        self,
        tidx: cutlass.Int32,
        tAcc: cute.Tensor,
        gC_mnl: cute.Tensor,
        epi_tile: cute.Tile,
        use_2cta_instrs: Union[cutlass.Boolean, bool],
    ) -> tuple[cute.TiledCopy, cute.Tensor, cute.Tensor]:
        """
        Make tiledCopy for tensor memory load, then use it to partition tensor memory (source) and register array (destination).

        :param tidx: The thread index in epilogue warp groups
        :type tidx: cutlass.Int32
        :param tAcc: The accumulator tensor to be copied and partitioned
        :type tAcc: cute.Tensor
        :param gC_mnl: The global tensor C
        :type gC_mnl: cute.Tensor
````
- **EN**: Introduces function `epilog_tmem_copy_and_partition`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`epilog_tmem_copy_and_partition`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1373-1400 / 第 1373-1400 行
````python
        :param epi_tile: The epilogue tiler
        :type epi_tile: cute.Tile
        :param use_2cta_instrs: Whether use_2cta_instrs is enabled
        :type use_2cta_instrs: bool

        :return: A tuple containing (tiled_copy_t2r, tTR_tAcc, tTR_rAcc) where:
            - tiled_copy_t2r: The tiled copy operation for tmem to register copy(t2r)
            - tTR_tAcc: The partitioned accumulator tensor
            - tTR_rAcc: The accumulated tensor in register used to hold t2r results
        :rtype: Tuple[cute.TiledCopy, cute.Tensor, cute.Tensor]
        """
        # Make tiledCopy for tensor memory load(t2r)
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
        tiled_copy_t2r = tcgen05.make_tmem_copy(
            copy_atom_t2r, tAcc_epi[(None, None, 0, 0, 0)]
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `copy_atom_t2r`, `tAcc_epi`, and `tiled_copy_t2r`. This range continues the implementation of function `GroupedGemmKernel.epilog_tmem_copy_and_partition`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `copy_atom_t2r`、`tAcc_epi`、`tiled_copy_t2r` 等值。这一段延续了函数`GroupedGemmKernel.epilog_tmem_copy_and_partition` 的具体实现。

### Lines 1401-1428 / 第 1401-1428 行
````python
        )

        thr_copy_t2r = tiled_copy_t2r.get_slice(tidx)
        # (T2R, T2R_M, T2R_N, EPI_M, EPI_M, STAGE)
        tTR_tAcc = thr_copy_t2r.partition_S(tAcc_epi)

        # (EPI_TILE_M, EPI_TILE_N, EPI_M, EPI_N, RestM, RestN, RestL)
        gC_mnl_epi = cute.flat_divide(
            gC_mnl[((None, None), 0, 0, None, None, None)], epi_tile
        )
        # (T2R, T2R_M, T2R_N, EPI_M, EPI_N, RestM, RestN, RestL)
        tTR_gC = thr_copy_t2r.partition_D(gC_mnl_epi)
        # (T2R, T2R_M, T2R_N)
        tTR_rAcc = cute.make_rmem_tensor(
            tTR_gC[(None, None, None, 0, 0, 0, 0, 0)].shape, self.acc_dtype
        )
        return tiled_copy_t2r, tTR_tAcc, tTR_rAcc

    def epilog_smem_copy_and_partition(
        self,
        tiled_copy_t2r: cute.TiledCopy,
        tTR_rC: cute.Tensor,
        tidx: cutlass.Int32,
        sC: cute.Tensor,
    ) -> tuple[cute.TiledCopy, cute.Tensor, cute.Tensor]:
        """
        Make tiledCopy for shared memory store, then use it to partition register array (source) and shared memory (destination).

````
- **EN**: Introduces function `epilog_smem_copy_and_partition`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `thr_copy_t2r`, `tTR_tAcc`, `gC_mnl_epi`, `tTR_gC`, `tTR_rAcc`, `tiled_copy_t2r`, and `...+3`.
- **CN**: 这里定义了函数`epilog_smem_copy_and_partition`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `thr_copy_t2r`、`tTR_tAcc`、`gC_mnl_epi`、`tTR_gC`、`tTR_rAcc`、`tiled_copy_t2r`、`另有3项` 等值。

### Lines 1429-1456 / 第 1429-1456 行
````python
        :param tiled_copy_t2r: The tiled copy operation for tmem to register copy(t2r)
        :type tiled_copy_t2r: cute.TiledCopy
        :param tTR_rC: The partitioned accumulator tensor
        :type tTR_rC: cute.Tensor
        :param tidx: The thread index in epilogue warp groups
        :type tidx: cutlass.Int32
        :param sC: The shared memory tensor to be copied and partitioned
        :type sC: cute.Tensor

        :return: A tuple containing (tiled_copy_r2s, tRS_rC, tRS_sC) where:
            - tiled_copy_r2s: The tiled copy operation for register to smem copy(r2s)
            - tRS_rC: The partitioned tensor C (register source)
            - tRS_sC: The partitioned tensor C (smem destination)
        :rtype: Tuple[cute.TiledCopy, cute.Tensor, cute.Tensor]
        """
        copy_atom_r2s = sm100_utils.get_smem_store_op(
            self.c_layout, self.c_dtype, self.acc_dtype, tiled_copy_t2r
        )
        tiled_copy_r2s = cute.make_tiled_copy_D(copy_atom_r2s, tiled_copy_t2r)
        # (R2S, R2S_M, R2S_N, PIPE_D)
        thr_copy_r2s = tiled_copy_r2s.get_slice(tidx)
        tRS_sC = thr_copy_r2s.partition_D(sC)
        # (R2S, R2S_M, R2S_N)
        tRS_rC = tiled_copy_r2s.retile(tTR_rC)
        return tiled_copy_r2s, tRS_rC, tRS_sC

    def epilog_gmem_copy_and_partition(
        self,
````
- **EN**: Introduces function `epilog_gmem_copy_and_partition`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `copy_atom_r2s`, `tiled_copy_r2s`, `thr_copy_r2s`, `tRS_sC`, and `tRS_rC`.
- **CN**: 这里定义了函数`epilog_gmem_copy_and_partition`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `copy_atom_r2s`、`tiled_copy_r2s`、`thr_copy_r2s`、`tRS_sC`、`tRS_rC` 等值。

### Lines 1457-1484 / 第 1457-1484 行
````python
        tma_atom_c: cute.CopyAtom,
        gC_mnl: cute.Tensor,
        epi_tile: cute.Tile,
        sC: cute.Tensor,
    ) -> tuple[cute.CopyAtom, cute.Tensor, cute.Tensor]:
        """Make tiledCopy for global memory store, then use it to partition
        shared memory (source) and global memory (destination) for TMA store version.

        :param tma_atom_c: The TMA copy atom configured for storing tensor C.
        :type tma_atom_c: cute.CopyAtom
        :param gC_mnl: The global memory tensor C.
        :type gC_mnl: cute.Tensor
        :param epi_tile: The epilogue tiler defining the granularity of the operation.
        :type epi_tile: cute.Tile
        :param sC: The shared memory epilogue buffer tensor.
        :type sC: cute.Tensor
        :return: A tuple containing:
                 - tma_atom_c: The input TMA copy atom (passed through).
                 - bSG_sC: The source shared memory tensor partitioned for the TMA operation.
                 - tCgC: The destination global memory tensor partitioned for the TMA operation.
        :rtype: tuple[cute.CopyAtom, cute.Tensor, cute.Tensor]
        """
        # (EPI_TILE_M, EPI_TILE_N, EPI_M, EPI_N, RestM, RestN, RestL)
        gC_epi = cute.flat_divide(
            gC_mnl[((None, None), 0, 0, None, None, None)], epi_tile
        )
        sC_for_tma_partition = cute.group_modes(sC, 0, 2)
        gC_for_tma_partition = cute.group_modes(gC_epi, 0, 2)
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `tma_atom_c`, `gC_mnl`, `epi_tile`, `sC`, `gC_epi`, `sC_for_tma_partition`, and `...+1`. This range continues the implementation of function `GroupedGemmKernel.epilog_gmem_copy_and_partition`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `tma_atom_c`、`gC_mnl`、`epi_tile`、`sC`、`gC_epi`、`sC_for_tma_partition`、`另有1项` 等值。这一段延续了函数`GroupedGemmKernel.epilog_gmem_copy_and_partition` 的具体实现。

### Lines 1485-1512 / 第 1485-1512 行
````python
        # ((ATOM_V, REST_V), EPI_M, EPI_N)
        # ((ATOM_V, REST_V), EPI_M, EPI_N, RestM, RestN, RestL)
        bSG_sC, bSG_gC = cpasync.tma_partition(
            tma_atom_c,
            0,
            cute.make_layout(1),
            sC_for_tma_partition,
            gC_for_tma_partition,
        )
        return tma_atom_c, bSG_sC, bSG_gC

    @staticmethod
    def _compute_stages(
        tiled_mma: cute.TiledMma,
        mma_tiler_mnk: tuple[int, int, int],
        a_dtype: type[cutlass.Numeric],
        b_dtype: type[cutlass.Numeric],
        epi_tile: cute.Tile,
        c_dtype: type[cutlass.Numeric],
        c_layout: utils.LayoutEnum,
        smem_capacity: int,
        occupancy: int,
    ) -> tuple[int, int, int]:
        """Computes the number of stages for accumulator, A/B operands, and epilogue based on heuristics.

        :param tiled_mma: The tiled MMA object defining the core computation.
        :type tiled_mma: cute.TiledMma
        :param mma_tiler_mnk: The shape (M, N, K) of the MMA tiler.
````
- **EN**: Introduces function `_compute_stages`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `tiled_mma`, `mma_tiler_mnk`, `a_dtype`, `b_dtype`, `epi_tile`, `c_dtype`, and `...+3`.
- **CN**: 这里定义了函数`_compute_stages`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `tiled_mma`、`mma_tiler_mnk`、`a_dtype`、`b_dtype`、`epi_tile`、`c_dtype`、`另有3项` 等值。

### Lines 1513-1540 / 第 1513-1540 行
````python
        :type mma_tiler_mnk: tuple[int, int, int]
        :param a_dtype: Data type of operand A.
        :type a_dtype: type[cutlass.Numeric]
        :param b_dtype: Data type of operand B.
        :type b_dtype: type[cutlass.Numeric]
        :param epi_tile: The epilogue tile shape.
        :type epi_tile: cute.Tile
        :param c_dtype: Data type of operand C (output).
        :type c_dtype: type[cutlass.Numeric]
        :param c_layout: Layout enum of operand C in global memory.
        :type c_layout: utils.LayoutEnum
        :param smem_capacity: Total available shared memory capacity in bytes.
        :type smem_capacity: int
        :param occupancy: Target number of CTAs per SM (occupancy).
        :type occupancy: int

        :return: A tuple containing the computed number of stages for:
                 (accumulator stages, A/B operand stages, epilogue stages)
        :rtype: tuple[int, int, int]
        """
        # Default accumulator and epilogue stages
        num_acc_stage = 2
        num_epi_stage = 2

        # Calculate smem layout and size for one stage of A, B, and Epilogue
        a_smem_layout_stage_one = sm100_utils.make_smem_layout_a(
            tiled_mma,
            mma_tiler_mnk,
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `num_acc_stage`, `num_epi_stage`, and `a_smem_layout_stage_one`. This range continues the implementation of function `GroupedGemmKernel._compute_stages`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `num_acc_stage`、`num_epi_stage`、`a_smem_layout_stage_one` 等值。这一段延续了函数`GroupedGemmKernel._compute_stages` 的具体实现。

### Lines 1541-1568 / 第 1541-1568 行
````python
            a_dtype,
            1,  # stage=1
        )
        b_smem_layout_staged_one = sm100_utils.make_smem_layout_b(
            tiled_mma,
            mma_tiler_mnk,
            b_dtype,
            1,  # stage=1
        )
        epi_smem_layout_staged_one = sm100_utils.make_smem_layout_epi(
            c_dtype,
            c_layout,
            epi_tile,
            1,  # stage=1
        )
        ab_bytes_per_stage = cute.size_in_bytes(
            a_dtype, a_smem_layout_stage_one
        ) + cute.size_in_bytes(b_dtype, b_smem_layout_staged_one)

        epi_bytes_per_stage = cute.size_in_bytes(c_dtype, epi_smem_layout_staged_one)
        epi_bytes = epi_bytes_per_stage * num_epi_stage

        # Calculate A/B stages:
        # Start with total smem per CTA (capacity / occupancy)
        # Subtract reserved bytes and initial epilogue bytes
        # Divide remaining by bytes needed per A/B stage
        num_ab_stage = (
            smem_capacity // occupancy
````
- **EN**: Initializes or updates values such as `b_smem_layout_staged_one`, `epi_smem_layout_staged_one`, `ab_bytes_per_stage`, `epi_bytes_per_stage`, `epi_bytes`, and `num_ab_stage`. This range continues the implementation of function `GroupedGemmKernel._compute_stages`.
- **CN**: 初始化或更新了 `b_smem_layout_staged_one`、`epi_smem_layout_staged_one`、`ab_bytes_per_stage`、`epi_bytes_per_stage`、`epi_bytes`、`num_ab_stage` 等值。这一段延续了函数`GroupedGemmKernel._compute_stages` 的具体实现。

### Lines 1569-1596 / 第 1569-1596 行
````python
            - GroupedGemmKernel.reserved_smem_bytes
            - epi_bytes
        ) // ab_bytes_per_stage

        # Refine epilogue stages:
        # Calculate remaining smem after allocating for A/B stages and reserved bytes
        # Add remaining unused smem to epilogue
        remaining_smem = (
            smem_capacity
            - occupancy * ab_bytes_per_stage * num_ab_stage
            - occupancy * (GroupedGemmKernel.reserved_smem_bytes + epi_bytes)
        )
        num_epi_stage += remaining_smem // (occupancy * epi_bytes_per_stage)
        return num_acc_stage, num_ab_stage, num_epi_stage

    @staticmethod
    def _compute_grid(
        total_num_clusters: int,
        cluster_shape_mn: tuple[int, int],
        max_active_clusters: cutlass.Constexpr[int],
    ) -> tuple[utils.PersistentTileSchedulerParams, tuple[int, int, int]]:
        """Compute tile scheduler parameters and grid shape for grouped GEMM operations.

        :param total_num_clusters: Total number of clusters to process across all groups.
        :type total_num_clusters: int
        :param cluster_shape_mn: Shape of each cluster in M, N dimensions.
        :type cluster_shape_mn: tuple[int, int]
        :param max_active_clusters: Maximum number of active clusters.
````
- **EN**: Introduces function `_compute_grid`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `remaining_smem`, `total_num_clusters`, `cluster_shape_mn`, and `max_active_clusters`.
- **CN**: 这里定义了函数`_compute_grid`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `remaining_smem`、`total_num_clusters`、`cluster_shape_mn`、`max_active_clusters` 等值。

### Lines 1597-1624 / 第 1597-1624 行
````python
        :type max_active_clusters: cutlass.Constexpr[int]

        :return: A tuple containing:
            - tile_sched_params: Parameters for the persistent tile scheduler.
            - grid: Grid shape for kernel launch.
        :rtype: tuple[utils.PersistentTileSchedulerParams, tuple[int, ...]]
        """
        # Create problem shape with M, N dimensions from cluster shape
        # and L dimension representing the total number of clusters.
        problem_shape_ntile_mnl = (
            cluster_shape_mn[0],
            cluster_shape_mn[1],
            cutlass.Int32(total_num_clusters),
        )

        tile_sched_params = utils.PersistentTileSchedulerParams(
            problem_shape_ntile_mnl, (*cluster_shape_mn, 1)
        )

        grid = utils.StaticPersistentGroupTileScheduler.get_grid_shape(
            tile_sched_params, max_active_clusters
        )

        return tile_sched_params, grid

    @staticmethod
    def _get_mbar_smem_bytes(**kwargs_stages: int) -> int:
        """Calculate shared memory consumption for memory barriers based on provided stages.
````
- **EN**: Introduces function `_get_mbar_smem_bytes`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `problem_shape_ntile_mnl`, `tile_sched_params`, and `grid`.
- **CN**: 这里定义了函数`_get_mbar_smem_bytes`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `problem_shape_ntile_mnl`、`tile_sched_params`、`grid` 等值。

### Lines 1625-1652 / 第 1625-1652 行
````python

        Each stage requires 2 barriers, and each barrier consumes 8 bytes of shared memory.
        The total consumption is the sum across all provided stages. This function calculates the total
        shared memory needed for these barriers.

        :param kwargs_stages: Variable keyword arguments where each key is a stage name
                              (e.g., num_acc_stage, num_ab_stage) and each value is the
                              number of stages of that type.
        :type kwargs_stages: int
        :return: Total shared memory bytes required for all memory barriers.
        :rtype: int
        """
        num_barriers_per_stage = 2
        num_bytes_per_barrier = 8
        mbar_smem_consumption = sum(
            [
                num_barriers_per_stage * num_bytes_per_barrier * stage
                for stage in kwargs_stages.values()
            ]
        )
        return mbar_smem_consumption

    @staticmethod
    def _get_tensormap_smem_bytes(
        tensormap_update_mode: utils.TensorMapUpdateMode,
    ) -> int:
        """Get the SMEM consumption for the tensormap buffer based on the update mode.

````
- **EN**: Introduces function `_get_tensormap_smem_bytes`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_tensormap_smem_bytes`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1653-1680 / 第 1653-1680 行
````python
        :param tensormap_update_mode: Specifies whether tensormaps are updated in GMEM or SMEM.
        :type tensormap_update_mode: utils.TensorMapUpdateMode
        :return: The shared memory bytes required for the tensormap buffer. Returns 0 if mode is GMEM.
        :rtype: int
        :raises ValueError: If an invalid tensormap update mode is provided.
        """
        if tensormap_update_mode == utils.TensorMapUpdateMode.GMEM:
            return 0
        elif tensormap_update_mode == utils.TensorMapUpdateMode.SMEM:
            return (
                GroupedGemmKernel.bytes_per_tensormap * GroupedGemmKernel.num_tensormaps
            )
        else:
            raise ValueError(f"Invalid tensormap update mode: {tensormap_update_mode}")

    @staticmethod
    def _compute_num_tmem_alloc_cols(
        tiled_mma: cute.TiledMma,
        mma_tiler: tuple[int, int, int],
        num_acc_stage: int,
    ) -> int:
        """
        Compute the number of tensor memory allocation columns.

        :param tiled_mma: The tiled MMA object defining the core computation.
        :type tiled_mma: cute.TiledMma
        :param mma_tiler: The shape (M, N, K) of the MMA tile.
        :type mma_tiler: tuple[int, int, int]
````
- **EN**: Introduces function `_compute_num_tmem_alloc_cols`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_compute_num_tmem_alloc_cols`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1681-1708 / 第 1681-1708 行
````python
        :param acc_stage: The stage of the accumulator tensor.
        :type acc_stage: int

        :return: The number of tensor memory allocation columns.
        :rtype: int
        """
        acc_shape = tiled_mma.partition_shape_C(mma_tiler[:2])
        tCtAcc_fake = tiled_mma.make_fragment_C(cute.append(acc_shape, num_acc_stage))
        num_tmem_alloc_cols = utils.get_num_tmem_alloc_cols(tCtAcc_fake)

        return num_tmem_alloc_cols

    # Size of smem we reserved for mbarrier, tensor memory management and tensormap update
    reserved_smem_bytes = 1024
    bytes_per_tensormap = 128
    num_tensormaps = 3
    # size of smem used for tensor memory management
    tensor_memory_management_bytes = 12


# Create tensor and return the pointer, tensor, and stride
def create_tensor_and_stride(
    l: int,
    mode0: int,
    mode1: int,
    is_mode0_major: bool,
    dtype: type[cutlass.Numeric],
    is_dynamic_layout: bool = True,
````
- **EN**: Introduces function `create_tensor_and_stride`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `acc_shape`, `tCtAcc_fake`, `num_tmem_alloc_cols`, `reserved_smem_bytes`, `bytes_per_tensormap`, `num_tensormaps`, and `...+7`.
- **CN**: 这里定义了函数`create_tensor_and_stride`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `acc_shape`、`tCtAcc_fake`、`num_tmem_alloc_cols`、`reserved_smem_bytes`、`bytes_per_tensormap`、`num_tensormaps`、`另有7项` 等值。

### Lines 1709-1736 / 第 1709-1736 行
````python
    torch_tensor_cpu: torch.Tensor = None,
) -> tuple[int, torch.Tensor, cute.Tensor, torch.Tensor, tuple[int, int]]:
    """Create GPU tensor from either a new or existing CPU tensor.

    :param torch_tensor_cpu: Optional existing CPU tensor to reuse. If None, creates a new one.
    :type torch_tensor_cpu: torch.Tensor, optional
    """
    if torch_tensor_cpu is None:
        # Create new CPU tensor
        torch_tensor_cpu = cutlass_torch.matrix(l, mode0, mode1, is_mode0_major, dtype)

    # Create GPU tensor from CPU tensor (new or existing)
    cute_tensor, torch_tensor = cutlass_torch.cute_tensor_like(
        torch_tensor_cpu, dtype, is_dynamic_layout, assumed_align=16
    )
    return (
        torch_tensor.data_ptr(),
        torch_tensor,
        cute_tensor,
        torch_tensor_cpu,
        torch_tensor.stride()[:-1],
    )


def create_tensors_for_all_groups(
    problem_sizes_mnkl: List[tuple[int, int, int, int]],
    ab_dtype: Type[cutlass.Numeric],
    c_dtype: Type[cutlass.Numeric],
````
- **EN**: Introduces function `create_tensors_for_all_groups`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`create_tensors_for_all_groups`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1737-1764 / 第 1737-1764 行
````python
    a_major: str,
    b_major: str,
    c_major: str,
    torch_fp32_tensors_abc: List[List[torch.Tensor]] = None,
) -> tuple[
    List[List[int]],
    List[List[torch.Tensor]],
    List[tuple],
    List[List[tuple]],
    List[List[torch.Tensor]],
]:
    if torch_fp32_tensors_abc is not None and len(torch_fp32_tensors_abc) != len(
        problem_sizes_mnkl
    ):
        raise ValueError("torch_fp32_tensors_abc must have one entry per group")

    # Initialize lists to store tensors for all groups
    new_torch_fp32_tensors_abc = (
        [] if torch_fp32_tensors_abc is None else torch_fp32_tensors_abc
    )
    torch_tensors_abc = []
    cute_tensors_abc = []
    strides_abc = []
    ptrs_abc = []

    # Iterate through all groups and create tensors for each group
    for group_idx, (m, n, k, l) in enumerate(problem_sizes_mnkl):
        # Get existing CPU tensors if available, otherwise None
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `a_major`, `b_major`, `c_major`, `torch_fp32_tensors_abc`, `new_torch_fp32_tensors_abc`, `torch_tensors_abc`, and `...+3`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `a_major`、`b_major`、`c_major`、`torch_fp32_tensors_abc`、`new_torch_fp32_tensors_abc`、`torch_tensors_abc`、`另有3项` 等值。

### Lines 1765-1792 / 第 1765-1792 行
````python
        existing_cpu_a = (
            torch_fp32_tensors_abc[group_idx][0] if torch_fp32_tensors_abc else None
        )
        existing_cpu_b = (
            torch_fp32_tensors_abc[group_idx][1] if torch_fp32_tensors_abc else None
        )
        existing_cpu_c = (
            torch_fp32_tensors_abc[group_idx][2] if torch_fp32_tensors_abc else None
        )

        # Create tensors (reusing CPU tensors if provided)
        (
            ptr_a,
            torch_tensor_a,
            cute_tensor_a,
            tensor_fp32_a,
            stride_mk_a,
        ) = create_tensor_and_stride(
            l, m, k, a_major == "m", ab_dtype, torch_tensor_cpu=existing_cpu_a
        )
        (
            ptr_b,
            torch_tensor_b,
            cute_tensor_b,
            tensor_fp32_b,
            stride_nk_b,
        ) = create_tensor_and_stride(
            l, n, k, b_major == "n", ab_dtype, torch_tensor_cpu=existing_cpu_b
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `existing_cpu_a`, `existing_cpu_b`, and `existing_cpu_c`. This range continues the implementation of function `create_tensors_for_all_groups`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `existing_cpu_a`、`existing_cpu_b`、`existing_cpu_c` 等值。这一段延续了函数`create_tensors_for_all_groups` 的具体实现。

### Lines 1793-1820 / 第 1793-1820 行
````python
        )
        (
            ptr_c,
            torch_tensor_c,
            cute_tensor_c,
            tensor_fp32_c,
            stride_mn_c,
        ) = create_tensor_and_stride(
            l, m, n, c_major == "m", c_dtype, torch_tensor_cpu=existing_cpu_c
        )

        # Only append to new_torch_fp32_tensors_abc if we created new CPU tensors
        if torch_fp32_tensors_abc is None:
            new_torch_fp32_tensors_abc.append(
                [tensor_fp32_a, tensor_fp32_b, tensor_fp32_c]
            )

        ptrs_abc.append([ptr_a, ptr_b, ptr_c])
        torch_tensors_abc.append([torch_tensor_a, torch_tensor_b, torch_tensor_c])
        strides_abc.append([stride_mk_a, stride_nk_b, stride_mn_c])
        cute_tensors_abc.append(
            (
                cute_tensor_a,
                cute_tensor_b,
                cute_tensor_c,
            )
        )

````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `create_tensors_for_all_groups`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。这一段延续了函数`create_tensors_for_all_groups` 的具体实现。

### Lines 1821-1848 / 第 1821-1848 行
````python
    return (
        ptrs_abc,
        torch_tensors_abc,
        cute_tensors_abc,
        strides_abc,
        new_torch_fp32_tensors_abc,
    )


def run(
    num_groups: int,
    problem_sizes_mnkl: tuple[int, int, int, int],
    host_problem_shape_available: bool,
    ab_dtype: Type[cutlass.Numeric],
    c_dtype: Type[cutlass.Numeric],
    acc_dtype: Type[cutlass.Numeric],
    a_major: str,
    b_major: str,
    c_major: str,
    mma_tiler_mn: tuple[int, int],
    cluster_shape_mn: tuple[int, int],
    use_2cta_instrs: bool,
    tensormap_update_mode: utils.TensorMapUpdateMode,
    tolerance: float,
    warmup_iterations: int,
    iterations: int,
    skip_ref_check: bool,
    use_cold_l2: bool = False,
````
- **EN**: Introduces function `run`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `num_groups`, `problem_sizes_mnkl`, `host_problem_shape_available`, `ab_dtype`, `c_dtype`, `acc_dtype`, and `...+12`.
- **CN**: 这里定义了函数`run`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `num_groups`、`problem_sizes_mnkl`、`host_problem_shape_available`、`ab_dtype`、`c_dtype`、`acc_dtype`、`另有12项` 等值。

### Lines 1849-1876 / 第 1849-1876 行
````python
    **kwargs,
):
    """Run grouped GEMM example with specified configurations.

    :param use_cold_l2: Whether to use circular buffer strategy to ensure cold L2 cache, defaults to False
    :type use_cold_l2: bool, optional
    :return: Execution time of the GEMM kernel in microseconds
    :rtype: float
    """
    print("Running Blackwell Grouped GEMM test with:")
    print(f"{num_groups} groups")
    for i, (m, n, k, l) in enumerate(problem_sizes_mnkl):
        print(f"Group {i}: {m}x{n}x{k}x{l}")
    print(f"AB dtype: {ab_dtype}, C dtype: {c_dtype}, Acc dtype: {acc_dtype}")
    print(f"Matrix majors - A: {a_major}, B: {b_major}, C: {c_major}")
    print(f"Mma Tiler (M, N): {mma_tiler_mn}, Cluster Shape (M, N): {cluster_shape_mn}")
    print(f"2CTA MMA instructions: {'True' if use_2cta_instrs else 'False'}")
    print(f"Tensor map update mode: {tensormap_update_mode}")
    print(f"Tolerance: {tolerance}")
    print(f"Warmup iterations: {warmup_iterations}")
    print(f"Iterations: {iterations}")
    print(f"Skip reference checking: {skip_ref_check}")
    print(f"Use cold L2: {'True' if use_cold_l2 else 'False'}")

    # Skip unsupported types
    if ab_dtype not in {
        cutlass.Float16,
        cutlass.BFloat16,
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1877-1904 / 第 1877-1904 行
````python
    }:
        raise ValueError(f"Skip unsupported ab_dtype {ab_dtype}")
    if c_dtype not in {cutlass.Float16, cutlass.BFloat16, cutlass.Float32}:
        raise ValueError(f"Skip unsupported c_dtype {c_dtype}")
    # Skip unsupported acc dtype
    if acc_dtype not in {cutlass.Float32, cutlass.Float16}:
        raise ValueError(f"Skip unsupported acc_dtype {acc_dtype}")
    # Skip invalid ab_dtype and acc_dtype combination
    if ab_dtype == cutlass.BFloat16 and acc_dtype == cutlass.Float16:
        raise ValueError("Skip invalid ab_dtype and acc_dtype combination")
    # Skip invalid mma tile shape
    if not (
        (not use_2cta_instrs and mma_tiler_mn[0] in [64, 128])
        or (use_2cta_instrs and mma_tiler_mn[0] in [128, 256])
    ):
        raise ValueError(f"Skip invalid mma tiler M {mma_tiler_mn[0]}")
    if mma_tiler_mn[1] not in range(32, 257, 32):
        raise ValueError(f"Skip invalid mma tiler N {mma_tiler_mn[1]}")
    # Skip illegal cluster shape
    if cluster_shape_mn[0] % (2 if use_2cta_instrs else 1) != 0:
        raise ValueError(
            f"cluster_shape_m need align with use_2cta_instrs config {cluster_shape_mn}"
        )
    # Skip invalid cluster shape
    is_power_of_2 = lambda x: x > 0 and (x & (x - 1)) == 0
    if (
        cluster_shape_mn[0] * cluster_shape_mn[1] > 16
        or cluster_shape_mn[0] <= 0
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_power_of_2`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_power_of_2` 等值。

### Lines 1905-1932 / 第 1905-1932 行
````python
        or cluster_shape_mn[1] <= 0
        or not is_power_of_2(cluster_shape_mn[0])
        or not is_power_of_2(cluster_shape_mn[1])
    ):
        raise ValueError(f"Skip invalid cluster shape {cluster_shape_mn}")

    # Skip illegal problem shape for load/store alignment
    def check_contigous_16B_alignment(dtype, is_mode0_major, tensor_shape):
        major_mode_idx = 0 if is_mode0_major else 1
        num_major_elements = tensor_shape[major_mode_idx]
        num_contiguous_elements = 16 * 8 // dtype.width
        return num_major_elements % num_contiguous_elements == 0

    if (
        not check_contigous_16B_alignment(ab_dtype, a_major == "m", (m, k, l))
        or not check_contigous_16B_alignment(ab_dtype, b_major == "n", (n, k, l))
        or not check_contigous_16B_alignment(c_dtype, c_major == "m", (m, n, l))
    ):
        raise ValueError("Skip invalid problem alignment")
    if not torch.cuda.is_available():
        raise RuntimeError("GPU is required to run this example!")

    # Create tensors for all groups using the new function
    (
        ptrs_abc,
        torch_tensors_abc,
        cute_tensors_abc,
        strides_abc,
````
- **EN**: Introduces function `check_contigous_16B_alignment`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`check_contigous_16B_alignment`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1933-1960 / 第 1933-1960 行
````python
        torch_fp32_tensors_abc,
    ) = create_tensors_for_all_groups(
        problem_sizes_mnkl,
        ab_dtype,
        c_dtype,
        a_major,
        b_major,
        c_major,
    )

    # Setup inital tensors for TMA of A,B and C
    alignment = 16  # 16 bytes aligned
    min_ab_size = alignment * 8 // ab_dtype.width
    min_c_size = alignment * 8 // c_dtype.width
    initial_cute_tensors_abc = [
        create_tensor_and_stride(1, min_ab_size, min_ab_size, a_major == "m", ab_dtype)[
            2
        ],
        create_tensor_and_stride(1, min_ab_size, min_ab_size, b_major == "n", ab_dtype)[
            2
        ],
        create_tensor_and_stride(1, min_c_size, min_c_size, c_major == "m", c_dtype)[2],
    ]

    hardware_info = utils.HardwareInfo()
    sm_count = hardware_info.get_max_active_clusters(1)
    max_active_clusters = hardware_info.get_max_active_clusters(
        cluster_shape_mn[0] * cluster_shape_mn[1]
````
- **EN**: Initializes or updates values such as `alignment`, `min_ab_size`, `min_c_size`, `initial_cute_tensors_abc`, `hardware_info`, `sm_count`, and `...+1`. This range continues the implementation of function `run`.
- **CN**: 初始化或更新了 `alignment`、`min_ab_size`、`min_c_size`、`initial_cute_tensors_abc`、`hardware_info`、`sm_count`、`另有1项` 等值。这一段延续了函数`run` 的具体实现。

### Lines 1961-1988 / 第 1961-1988 行
````python
    )

    # Prepare tensormap buffer for each SM
    num_tensormap_buffers = sm_count
    tensormap_shape = (
        num_tensormap_buffers,
        GroupedGemmKernel.num_tensormaps,
        GroupedGemmKernel.bytes_per_tensormap // 8,
    )
    tensor_of_tensormap, tensor_of_tensormap_torch = cutlass_torch.cute_tensor_like(
        torch.empty(tensormap_shape, dtype=torch.int64),
        cutlass.Int64,
        is_dynamic_layout=False,
    )

    grouped_gemm = GroupedGemmKernel(
        acc_dtype,
        use_2cta_instrs,
        mma_tiler_mn,
        cluster_shape_mn,
        tensormap_update_mode,
    )

    # layout (num_groups, 4):(4, 1)
    (
        tensor_of_dim_size_mnkl,
        tensor_of_dim_size_mnkl_torch,
    ) = cutlass_torch.cute_tensor_like(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `num_tensormap_buffers`, `tensormap_shape`, `is_dynamic_layout`, and `grouped_gemm`. This range continues the implementation of function `run`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `num_tensormap_buffers`、`tensormap_shape`、`is_dynamic_layout`、`grouped_gemm` 等值。这一段延续了函数`run` 的具体实现。

### Lines 1989-2016 / 第 1989-2016 行
````python
        torch.tensor(problem_sizes_mnkl, dtype=torch.int32),
        cutlass.Int32,
        is_dynamic_layout=False,
        assumed_align=16,
    )

    # layout (num_groups, 3, 2):(6, 2, 1)
    tensor_of_strides_abc, tensor_of_strides_abc_torch = cutlass_torch.cute_tensor_like(
        torch.tensor(strides_abc, dtype=torch.int32),
        cutlass.Int32,
        is_dynamic_layout=False,
        assumed_align=16,
    )

    # layout (num_groups,3):(3, 1)
    tensor_of_ptrs_abc, tensor_of_ptrs_abc_torch = cutlass_torch.cute_tensor_like(
        torch.tensor(ptrs_abc, dtype=torch.int64),
        cutlass.Int64,
        is_dynamic_layout=False,
        assumed_align=16,
    )

    # Compute total number of cluster tiles we need to compute for given grouped GEMM problem
    def compute_total_num_clusters(
        problem_sizes_mnkl: List[tuple[int, int, int, int]],
        cluster_tile_shape_mn: tuple[int, int],
    ) -> int:
        total_num_clusters = 0
````
- **EN**: Introduces function `compute_total_num_clusters`. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `is_dynamic_layout`, `assumed_align`, `problem_sizes_mnkl`, `cluster_tile_shape_mn`, and `total_num_clusters`.
- **CN**: 这里定义了函数`compute_total_num_clusters`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `is_dynamic_layout`、`assumed_align`、`problem_sizes_mnkl`、`cluster_tile_shape_mn`、`total_num_clusters` 等值。

### Lines 2017-2044 / 第 2017-2044 行
````python
        for m, n, _, _ in problem_sizes_mnkl:
            num_clusters_mn = tuple(
                (x + y - 1) // y for x, y in zip((m, n), cluster_tile_shape_mn)
            )
            total_num_clusters += functools.reduce(lambda x, y: x * y, num_clusters_mn)
        return total_num_clusters

    # Compute cluster tile shape
    def compute_cluster_tile_shape(
        mma_tiler_mn: tuple[int, int],
        cluster_shape_mn: tuple[int, int],
        use_2cta_instrs: bool,
    ) -> tuple[int, int]:
        cta_tile_shape_mn = list(mma_tiler_mn)
        if use_2cta_instrs:
            cta_tile_shape_mn[0] = cta_tile_shape_mn[0] // 2
        return tuple(x * y for x, y in zip(cta_tile_shape_mn, cluster_shape_mn))

    cluster_tile_shape_mn = compute_cluster_tile_shape(
        mma_tiler_mn, cluster_shape_mn, use_2cta_instrs
    )

    # If the host problem shape is available, we will launch the grid with only
    # the necessary clusters. The function compute_total_num_clusters() does that.
    # If the problem shape only exists on device, we will need to launch all active
    # clusters possible on a device.
    if host_problem_shape_available:
        print("Problem shapes available on host and device")
````
- **EN**: Introduces function `compute_cluster_tile_shape`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`compute_cluster_tile_shape`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2045-2072 / 第 2045-2072 行
````python
        total_num_clusters = compute_total_num_clusters(
            problem_sizes_mnkl, cluster_tile_shape_mn
        )
    else:
        print("Problem shapes available only on device")
        total_num_clusters = max_active_clusters

    # Initialize Stream
    current_stream = cutlass_torch.default_stream()

    # try to check CUDA version to decide the opt level
    try:
        from cutlass import CUDA_VERSION

        opt_level = (
            3
            if CUDA_VERSION.major < 13
            or (CUDA_VERSION.major == 13 and CUDA_VERSION.minor < 1)
            else 2
        )
    except ImportError:
        opt_level = 3
    # Compile grouped GEMM kernel
    compiled_grouped_gemm = cute.compile(
        grouped_gemm,
        initial_cute_tensors_abc[0],
        initial_cute_tensors_abc[1],
        initial_cute_tensors_abc[2],
````
- **EN**: Imports dependencies such as `cutlass` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `total_num_clusters`, `else`, `current_stream`, `try`, `opt_level`, and `compiled_grouped_gemm`.
- **CN**: 这里导入了 `cutlass` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `total_num_clusters`、`else`、`current_stream`、`try`、`opt_level`、`compiled_grouped_gemm` 等值。

### Lines 2073-2100 / 第 2073-2100 行
````python
        num_groups,
        tensor_of_dim_size_mnkl,
        tensor_of_strides_abc,
        tensor_of_ptrs_abc,
        total_num_clusters,
        tensor_of_tensormap,
        max_active_clusters,
        current_stream,
        options=f"--opt-level {opt_level}",
    )

    if not skip_ref_check:
        compiled_grouped_gemm(
            initial_cute_tensors_abc[0],
            initial_cute_tensors_abc[1],
            initial_cute_tensors_abc[2],
            tensor_of_dim_size_mnkl,
            tensor_of_strides_abc,
            tensor_of_ptrs_abc,
            tensor_of_tensormap,
            current_stream,
        )

        # Compute reference result
        for i, (a, b, c) in enumerate(torch_tensors_abc):
            ref = torch.einsum(
                "mkl,nkl->mnl",
                a.cpu().to(dtype=torch.float32),
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `options`, and `ref`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `options`、`ref` 等值。

### Lines 2101-2128 / 第 2101-2128 行
````python
                b.cpu().to(dtype=torch.float32),
            )
            print(f"checking group {i}")
            torch.testing.assert_close(
                c.cpu(),
                ref.to(cutlass_torch.dtype(c_dtype)),
                atol=tolerance,
                rtol=1e-05,
            )

    if iterations <= 0:
        return 0

    def generate_tensors():
        # Reuse existing CPU tensors and create new GPU tensors from them
        (
            ptrs_abc_workspace,
            torch_tensors_abc_workspace,
            cute_tensors_abc_workspace,
            strides_abc_workspace,
            _,
        ) = create_tensors_for_all_groups(
            problem_sizes_mnkl,
            ab_dtype,
            c_dtype,
            a_major,
            b_major,
            c_major,
````
- **EN**: Introduces function `generate_tensors`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`generate_tensors`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2129-2156 / 第 2129-2156 行
````python
            torch_fp32_tensors_abc,
        )

        initial_cute_tensors_abc_workspace = [
            create_tensor_and_stride(
                1, min_ab_size, min_ab_size, a_major == "m", ab_dtype
            )[2],
            create_tensor_and_stride(
                1, min_ab_size, min_ab_size, b_major == "n", ab_dtype
            )[2],
            create_tensor_and_stride(
                1, min_c_size, min_c_size, c_major == "m", c_dtype
            )[2],
        ]

        # Create new tensors for this workspace
        tensor_of_strides_abc_workspace, _ = cutlass_torch.cute_tensor_like(
            torch.tensor(strides_abc_workspace, dtype=torch.int32),
            cutlass.Int32,
            is_dynamic_layout=False,
            assumed_align=16,
        )

        tensor_of_ptrs_abc_workspace, _ = cutlass_torch.cute_tensor_like(
            torch.tensor(ptrs_abc_workspace, dtype=torch.int64),
            cutlass.Int64,
            is_dynamic_layout=False,
            assumed_align=16,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `initial_cute_tensors_abc_workspace`, `is_dynamic_layout`, and `assumed_align`. This range continues the implementation of function `run.generate_tensors`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `initial_cute_tensors_abc_workspace`、`is_dynamic_layout`、`assumed_align` 等值。这一段延续了函数`run.generate_tensors` 的具体实现。

### Lines 2157-2184 / 第 2157-2184 行
````python
        )

        tensormap_workspace, _ = cutlass_torch.cute_tensor_like(
            torch.empty(tensormap_shape, dtype=torch.int64),
            cutlass.Int64,
            is_dynamic_layout=False,
        )

        args = testing.JitArguments(
            initial_cute_tensors_abc_workspace[0],
            initial_cute_tensors_abc_workspace[1],
            initial_cute_tensors_abc_workspace[2],
            tensor_of_dim_size_mnkl,
            tensor_of_strides_abc_workspace,
            tensor_of_ptrs_abc_workspace,
            tensormap_workspace,
            current_stream,
        )
        args.add_to_scope([torch_tensors_abc_workspace])
        return args

    workspace_count = 1
    if use_cold_l2:
        one_workspace_bytes = (
            sum(
                [
                    sum(
                        [
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_dynamic_layout`, `args`, `workspace_count`, and `one_workspace_bytes`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_dynamic_layout`、`args`、`workspace_count`、`one_workspace_bytes` 等值。

### Lines 2185-2212 / 第 2185-2212 行
````python
                            torch_tensor.numel() * torch_tensor.element_size()
                            for torch_tensor in group_tensors
                        ]
                    )
                    for group_tensors in torch_tensors_abc
                ]
            )
            +
            # Add size of strides tensor
            tensor_of_strides_abc_torch.numel()
            * tensor_of_strides_abc_torch.element_size()
            +
            # Add size of ptrs tensor
            tensor_of_ptrs_abc_torch.numel() * tensor_of_ptrs_abc_torch.element_size()
            +
            # Add size of tensormap tensor
            tensor_of_tensormap_torch.numel() * tensor_of_tensormap_torch.element_size()
        )
        workspace_count = testing.get_workspace_count(
            one_workspace_bytes, warmup_iterations, iterations
        )

    exec_time = testing.benchmark(
        compiled_grouped_gemm,
        workspace_generator=generate_tensors,
        workspace_count=workspace_count,
        stream=current_stream,
        warmup_iterations=warmup_iterations,
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `workspace_count`, `exec_time`, `workspace_generator`, `stream`, and `warmup_iterations`. This range continues the implementation of function `run`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含分支、循环或上下文管理等控制流。初始化或更新了 `workspace_count`、`exec_time`、`workspace_generator`、`stream`、`warmup_iterations` 等值。这一段延续了函数`run` 的具体实现。

### Lines 2213-2240 / 第 2213-2240 行
````python
        iterations=iterations,
    )

    runtime_s = exec_time / 1.0e6
    fmas = 0
    for group in range(num_groups):
        [M, N, K, _] = problem_sizes_mnkl[group]
        fmas += M * N * K
    flop = 2 * fmas
    gflop = flop / 1.0e9
    gflops = gflop / runtime_s

    print("Average Runtime : ", exec_time / 1000, "ms")
    print("GFLOPS          : ", gflops)

    return exec_time  # Return execution time in microseconds


if __name__ == "__main__":

    def parse_comma_separated_ints(s: str) -> tuple[int, ...]:
        try:
            return tuple(int(x.strip()) for x in s.split(","))
        except ValueError:
            raise argparse.ArgumentTypeError(
                "Invalid format. Expected comma-separated integers."
            )

````
- **EN**: Introduces function `parse_comma_separated_ints`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`parse_comma_separated_ints`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2241-2268 / 第 2241-2268 行
````python
    def parse_comma_separated_tuples(s: str) -> List[tuple[int, ...]]:
        if s.strip().startswith("("):
            # Split on ),( to separate tuples
            tuples = s.strip("()").split("),(")
            result = []
            tuple_len = None

            for t in tuples:
                # Parse individual tuple
                nums = [int(x.strip()) for x in t.split(",")]

                # Validate tuple length consistency
                if tuple_len is None:
                    tuple_len = len(nums)
                elif len(nums) != tuple_len:
                    raise argparse.ArgumentTypeError(
                        "All tuples must have the same length"
                    )

                result.append(tuple(nums))
            return result

        raise argparse.ArgumentTypeError(
            "Invalid format. Expected comma-separated integers or list of tuples"
        )

    parser = argparse.ArgumentParser(
        description="Example of Grouped GEMM on Blackwell."
````
- **EN**: Introduces function `parse_comma_separated_tuples`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tuples`, `result`, `tuple_len`, `nums`, `parser`, and `description`.
- **CN**: 这里定义了函数`parse_comma_separated_tuples`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tuples`、`result`、`tuple_len`、`nums`、`parser`、`description` 等值。

### Lines 2269-2296 / 第 2269-2296 行
````python
    )
    parser.add_argument(
        "--num_groups",
        type=int,
        default=3,
        help="Number of groups",
    )
    parser.add_argument(
        "--problem_sizes_mnkl",
        type=parse_comma_separated_tuples,
        default=((128, 128, 128, 1), (512, 128, 128, 1), (128, 256, 128, 1)),
        help="a tuple of problem sizes for each group (comma-separated tuples)",
    )
    parser.add_argument(
        "--host_problem_shape_available",
        action="store_true",
        help="Enable the compute of grid based upon host problem shape",
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
````
- **EN**: Initializes or updates values such as `type`, `default`, `help`, and `action`.
- **CN**: 初始化或更新了 `type`、`default`、`help`、`action` 等值。

### Lines 2297-2324 / 第 2297-2324 行
````python
        help="Cluster shape (comma-separated)",
    )
    parser.add_argument(
        "--tensormap_update_mode",
        type=str,
        default="SMEM",
        help="Tensor map update mode",
    )
    parser.add_argument("--ab_dtype", type=cutlass.dtype, default=cutlass.Float16)
    parser.add_argument("--c_dtype", type=cutlass.dtype, default=cutlass.Float16)
    parser.add_argument("--acc_dtype", type=cutlass.dtype, default=cutlass.Float32)
    parser.add_argument(
        "--use_2cta_instrs",
        action="store_true",
        help="Enable 2CTA MMA instructions feature",
    )
    parser.add_argument("--a_major", choices=["k", "m"], type=str, default="k")
    parser.add_argument("--b_major", choices=["k", "n"], type=str, default="k")
    parser.add_argument("--c_major", choices=["n", "m"], type=str, default="n")
    parser.add_argument(
        "--tolerance", type=float, default=1e-01, help="Tolerance for validation"
    )
    parser.add_argument(
        "--warmup_iterations", type=int, default=0, help="Warmup iterations"
    )
    parser.add_argument(
        "--iterations",
        type=int,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `help`, `type`, `default`, and `action`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `help`、`type`、`default`、`action` 等值。

### Lines 2325-2352 / 第 2325-2352 行
````python
        default=1,
        help="Number of iterations to run the kernel",
    )
    parser.add_argument(
        "--skip_ref_check", action="store_true", help="Skip reference checking"
    )
    parser.add_argument(
        "--use_cold_l2",
        action="store_true",
        default=False,
        help="Use circular buffer tensor sets to ensure L2 cold cache",
    )

    args = parser.parse_args()

    if (
        len(args.problem_sizes_mnkl) != 0
        and len(args.problem_sizes_mnkl) != args.num_groups
    ):
        parser.error("--problem_sizes_mnkl must contain exactly num_groups tuples")

    # l mode must be 1 for all groups
    for _, _, _, l in args.problem_sizes_mnkl:
        if l != 1:
            parser.error("l must be 1 for all groups")

    if len(args.mma_tiler_mn) != 2:
        parser.error("--mma_tiler_mn must contain exactly 2 values")
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `default`, `help`, `action`, and `args`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `default`、`help`、`action`、`args` 等值。

### Lines 2353-2380 / 第 2353-2380 行
````python

    if len(args.cluster_shape_mn) != 2:
        parser.error("--cluster_shape_mn must contain exactly 2 values")

    if args.tensormap_update_mode not in ["GMEM", "SMEM"]:
        parser.error("--tensormap_update_mode must be GMEM or SMEM")

    if args.tensormap_update_mode == "GMEM":
        tensormap_update_mode = utils.TensorMapUpdateMode.GMEM
    else:
        tensormap_update_mode = utils.TensorMapUpdateMode.SMEM

    torch.manual_seed(2025)

    run(
        args.num_groups,
        args.problem_sizes_mnkl,
        args.host_problem_shape_available,
        args.ab_dtype,
        args.c_dtype,
        args.acc_dtype,
        args.a_major,
        args.b_major,
        args.c_major,
        args.mma_tiler_mn,
        args.cluster_shape_mn,
        args.use_2cta_instrs,
        tensormap_update_mode,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensormap_update_mode`, and `else`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tensormap_update_mode`、`else` 等值。

### Lines 2381-2387 / 第 2381-2387 行
````python
        args.tolerance,
        args.warmup_iterations,
        args.iterations,
        args.skip_ref_check,
        args.use_cold_l2,
    )
    print("PASS")
````
- **EN**: Implements the local expressions and calls needed for this part of the module.
- **CN**: 实现了该模块这一部分所需的局部表达式与调用逻辑。

## Key Concepts / 关键概念
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `GroupedGemmKernel`  
  **CN**: 主要类：`GroupedGemmKernel`
- **EN**: Primary functions: `create_tensor_and_stride`, `create_tensors_for_all_groups`, and `run`  
  **CN**: 主要函数：`create_tensor_and_stride`、`create_tensors_for_all_groups`、`run`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `functools`, `typing`, `inspect`
- **Third-party / 第三方**: `cuda.bindings.driver`, `cutlass`, `cutlass.cute`, `cutlass.cute.testing`, `cutlass.utils`, `cutlass.pipeline`, `cutlass.cute.nvgpu`, `cutlass.utils.blackwell_helpers`, `cutlass.torch`
- **PyTorch/Internal / PyTorch 内部**: `torch`
