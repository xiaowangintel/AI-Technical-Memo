# torch_grouped_mm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/moe/torch_grouped_mm.py`  
**Purpose / 用途**: Kernel example implementing torch grouped mm with CuTeDSL. / 这是一个使用 CuTeDSL 实现 torch grouped mm 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-41 / 第 29-41 行

~~~~python
import os
import sys
from typing import Optional, Tuple, Literal, Type, Union

import cuda.bindings.driver as cuda

import cutlass
import cutlass.cute as cute
from cutlass.cute.typing import Pointer
from cutlass.cute.nvgpu import cpasync, tcgen05
import cutlass.utils as utils
import cutlass.pipeline as pipeline
from cutlass.pipeline import pipeline_init_arrive, pipeline_init_wait
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 43-45 / 第 43-45 行

~~~~python
if __name__ == "__main__":
    current_dir = os.path.dirname(os.path.abspath(__file__))
    sys.path.insert(0, os.path.join(current_dir, "../../.."))
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 47-47 / 第 47-47 行

~~~~python
from blackwell.kernel.moe.moe_utils import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 48-60 / 第 48-60 行

~~~~python
    MoEGroupedGemmTensormapConstructor,
)
from blackwell.kernel.moe.moe_persistent_scheduler import (
    MoEStaticSchedulerParams,
    MoEStaticPersistentTileScheduler,
    MoEWorkTileInfo,
)
from blackwell.kernel.moe.moe_sched_extension import GroupedMmSchedExtension
from cutlass.utils.gemm.sm100 import (
    transform_partitioned_tensor_layout,
    epilogue_tmem_copy_and_partition,
    epilogue_smem_copy_and_partition,
)
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 63-86 / 第 63-86 行

~~~~python
class GroupedGemmKernel:
    """
    Grouped GEMM kernel for MoE operations.

    PyTorch interface (from torch.nn.functional.grouped_mm):
    - 2Dx3D (Forward): mat_a(tokens_sum, K) x mat_b(experts, K, N) -> out(tokens_sum, N)
    - 2Dx2D (Weight grad): mat_a(hidden, tokens_sum) x mat_b(tokens_sum, intermediate) -> out(experts, hidden, intermediate)

    Kernel interface uses "fake" GEMM MNKL domain:

    2Dx3D:
        A_cute: (gemm_fake_m, gemm_k, 1)     # fake_m = tokens_sum, scheduler will offset
        B_cute: (gemm_n, gemm_k, gemm_fake_l) # fake_l = expert_idx, scheduler will select
        C_cute: (gemm_fake_m, gemm_n, 1)     # fake_m = tokens_sum, scheduler will offset

    2Dx2D:
        A_cute: (gemm_m, gemm_fake_k, 1)     # fake_k = tokens_sum, scheduler will offset
        B_cute: (gemm_n, gemm_fake_k, 1)     # fake_k = tokens_sum, scheduler will offset
        C_cute: (gemm_m, gemm_n, gemm_fake_l) # fake_l = expert_idx, scheduler will select

    The scheduler handles the fake dimensions by:
    - For fake_m/fake_k: Computing token_offset from offs and adjusting tensor coord
    - For fake_l: Selecting expert slice via L coordinate
    """
~~~~

**EN**: Defines `GroupedGemmKernel`, a reusable Python class that packages configuration and behavior for this example. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `GroupedGemmKernel`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 88-110 / 第 88-110 行

~~~~python
    def __init__(
        self,
        scenario: Literal["2Dx3D", "2Dx2D"],
        out_dtype: Type[cutlass.Numeric],
        accumulate_on_output: bool,
        separate_tensormap_init: bool = True,
        fixed_expert_cnt: Optional[int] = None,
        acc_dtype: Type[cutlass.Numeric] = cutlass.Float32,
        mma_tiler_mnk: Tuple[int, int, int] = (128, 128, 64),
        cluster_shape_mnk: Tuple[int, int, int] = (1, 1, 1),
        use_2cta_instrs: bool = False,
    ):
        # User-provided configs
        self.scenario = scenario
        self.out_dtype = out_dtype
        self.accumulate_on_output = accumulate_on_output
        self.separate_tensormap_init = separate_tensormap_init
        self.fixed_expert_cnt = fixed_expert_cnt  # Not used yet...
        self.acc_dtype = acc_dtype
        self.mma_tiler_mnk = mma_tiler_mnk
        self.cluster_shape_mn = (cluster_shape_mnk[0], cluster_shape_mnk[1])
        self.use_2cta_instrs = use_2cta_instrs
        self.arch = "sm_100"
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 112-115 / 第 112-115 行

~~~~python
        if accumulate_on_output and scenario == "2Dx3D":
            raise ValueError(
                "Non-sense config: grad accumulate should only happens in 2Dx2D."
            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 117-117 / 第 117-117 行

~~~~python
        self._validate_mma_tiler_and_cluster_shape()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 119-119 / 第 119-119 行

~~~~python
        # K dimension is deferred in _setup_attributes
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 120-120 / 第 120-120 行

~~~~python
        self.mma_tiler = (mma_tiler_mnk[0], mma_tiler_mnk[1], 1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 122-122 / 第 122-122 行

~~~~python
        # CTA group for tcgen05 MMA
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 123-125 / 第 123-125 行

~~~~python
        self.cta_group = (
            tcgen05.CtaGroup.TWO if use_2cta_instrs else tcgen05.CtaGroup.ONE
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 127-127 / 第 127-127 行

~~~~python
        # Occupancy and warp specialization
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 128-140 / 第 128-140 行

~~~~python
        self.occupancy = 1
        self.epilogue_warp_id = (0, 1, 2, 3)
        self.mma_warp_id = 4
        self.tma_warp_id = 5
        self.sched_warp_id = 6
        self.threads_per_cta = 32 * len(
            (
                self.mma_warp_id,
                self.tma_warp_id,
                self.sched_warp_id,
                *self.epilogue_warp_id,
            )
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 142-142 / 第 142-142 行

~~~~python
        # Barrier IDs for synchronization
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 143-145 / 第 143-145 行

~~~~python
        self.epilog_sync_bar_id = 1
        self.tmem_alloc_sync_bar_id = 2
        self.tmem_dealloc_sync_bar_id = 3
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 147-150 / 第 147-150 行

~~~~python
    def _validate_mma_tiler_and_cluster_shape(self):
        """Validate codegen-time MMA tiler and cluster shape constraints."""
        m, n = self.mma_tiler_mnk[0], self.mma_tiler_mnk[1]
        cm, cn = self.cluster_shape_mn
~~~~

**EN**: Defines `_validate_mma_tiler_and_cluster_shape`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 定义 `_validate_mma_tiler_and_cluster_shape`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 152-160 / 第 152-160 行

~~~~python
        if self.use_2cta_instrs:
            valid_m = [128, 256]
        else:
            valid_m = [64, 128]
        if m not in valid_m:
            raise ValueError(
                f"mma_tiler M ({m}) must be one of {valid_m} "
                f"(use_2cta_instrs={self.use_2cta_instrs})"
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 162-163 / 第 162-163 行

~~~~python
        if n not in range(32, 257, 32):
            raise ValueError(f"mma_tiler N ({n}) must be a multiple of 32 in [32, 256]")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 165-168 / 第 165-168 行

~~~~python
        if cm % (2 if self.use_2cta_instrs else 1) != 0:
            raise ValueError(
                f"cluster_shape M ({cm}) must be even when use_2cta_instrs=True"
            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 170-175 / 第 170-175 行

~~~~python
        is_pow2 = lambda x: x > 0 and (x & (x - 1)) == 0
        if cm * cn > 16 or not is_pow2(cm) or not is_pow2(cn):
            raise ValueError(
                f"Invalid cluster_shape ({cm}, {cn}): each dim must be "
                f"a power of 2, and product must be <= 16"
            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 177-187 / 第 177-187 行

~~~~python
    def _create_tiled_mma(self) -> cute.TiledMma:
        """Create tiled MMA atom based on input dtypes and major modes."""
        return utils.sm100.make_trivial_tiled_mma(
            self.a_dtype,
            self.b_dtype,
            self.a_major_mode,
            self.b_major_mode,
            self.acc_dtype,
            self.cta_group,
            self.mma_tiler[:2],
        )
~~~~

**EN**: Defines `_create_tiled_mma`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_create_tiled_mma`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把构造好的对象或计算结果返回给调用方。

### Lines 189-204 / 第 189-204 行

~~~~python
    def _setup_attributes(self) -> None:
        """
        Set up configurations that depend on GEMM inputs.

        This method configures:
        - tiled_mma with correct dtypes and major modes
        - MMA/cluster/tile shapes
        - Cluster layout
        - Multicast CTA counts
        - Epilogue tile shape
        - Stage counts (ACC, A/B, C)
        - SMEM layouts for A/B/C
        - Tensor memory allocation columns
        - TMA load bytes
        """
        tiled_mma = self._create_tiled_mma()
~~~~

**EN**: Defines `_setup_attributes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `_setup_attributes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 206-207 / 第 206-207 行

~~~~python
        # Use user-specified K dimension directly from mma_tiler_mnk
        # Verify K is a multiple of the MMA instruction's native K size
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 208-217 / 第 208-217 行

~~~~python
        mma_inst_shape_k = cute.size(tiled_mma.shape_mnk, mode=[2])
        assert self.mma_tiler_mnk[2] % mma_inst_shape_k == 0, (
            f"mma_tiler K ({self.mma_tiler_mnk[2]}) must be a multiple of "
            f"MMA instruction K ({mma_inst_shape_k})"
        )
        self.mma_tiler = (
            self.mma_tiler[0],
            self.mma_tiler[1],
            self.mma_tiler_mnk[2],
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 219-219 / 第 219-219 行

~~~~python
        # CTA tile shape
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 220-224 / 第 220-224 行

~~~~python
        self.cta_tile_shape_mnk = (
            self.mma_tiler[0] // cute.size(tiled_mma.thr_id.shape),
            self.mma_tiler[1],
            self.mma_tiler[2],
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 226-226 / 第 226-226 行

~~~~python
        # Cluster layout
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 227-230 / 第 227-230 行

~~~~python
        self.cluster_layout_vmnk = cute.tiled_divide(
            cute.make_layout((*self.cluster_shape_mn, 1)),
            (tiled_mma.thr_id.shape,),
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 232-232 / 第 232-232 行

~~~~python
        # Multicast CTA counts
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 233-236 / 第 233-236 行

~~~~python
        self.num_mcast_ctas_a = cute.size(self.cluster_layout_vmnk.shape[2])
        self.num_mcast_ctas_b = cute.size(self.cluster_layout_vmnk.shape[1])
        self.is_a_mcast = self.num_mcast_ctas_a > 1
        self.is_b_mcast = self.num_mcast_ctas_b > 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 238-238 / 第 238-238 行

~~~~python
        # Epilogue tile shape (always use TMA store for MoE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 239-244 / 第 239-244 行

~~~~python
        self.epi_tile = utils.sm100.compute_epilogue_tile_shape(
            self.cta_tile_shape_mnk,
            self.use_2cta_instrs,
            self.c_layout,
            self.c_dtype,
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 246-246 / 第 246-246 行

~~~~python
        # C SMEM layout for epilogue
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 247-249 / 第 247-249 行

~~~~python
        c_smem_layout = utils.sm100.make_smem_layout_epi(
            self.c_dtype, self.c_layout, self.epi_tile, 1
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 251-251 / 第 251-251 行

~~~~python
        self.smem_capacity = utils.get_smem_capacity_in_bytes()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 253-253 / 第 253-253 行

~~~~python
        # Compute stage counts
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 254-255 / 第 254-255 行

~~~~python
        self.num_acc_stage = 2
        self.num_c_stage = 2  # Always use TMA store for MoE
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 257-262 / 第 257-262 行

~~~~python
        a_smem_layout_stage_one = utils.sm100.make_smem_layout_a(
            tiled_mma, self.mma_tiler, self.a_dtype, 1
        )
        b_smem_layout_stage_one = utils.sm100.make_smem_layout_b(
            tiled_mma, self.mma_tiler, self.b_dtype, 1
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 264-269 / 第 264-269 行

~~~~python
        ab_bytes_per_stage = cute.size_in_bytes(
            self.a_dtype, a_smem_layout_stage_one
        ) + cute.size_in_bytes(self.b_dtype, b_smem_layout_stage_one)
        mbar_helpers_bytes = 1024
        c_bytes_per_stage = cute.size_in_bytes(self.c_dtype, c_smem_layout)
        c_bytes = c_bytes_per_stage * self.num_c_stage
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 271-273 / 第 271-273 行

~~~~python
        self.num_sched_stages = 2
        sched_work_tile_bytes_per_stage = 16  # 4 fields * sizeof(Int32)
        sched_bytes = sched_work_tile_bytes_per_stage * self.num_sched_stages
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 275-275 / 第 275-275 行

~~~~python
        fixed_overhead = mbar_helpers_bytes + c_bytes + sched_bytes
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 277-279 / 第 277-279 行

~~~~python
        self.num_ab_stage = (
            self.smem_capacity // self.occupancy - fixed_overhead
        ) // ab_bytes_per_stage
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 281-281 / 第 281-281 行

~~~~python
        # Refine epilogue stages with remaining SMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 282-286 / 第 282-286 行

~~~~python
        self.num_c_stage += (
            self.smem_capacity
            - self.occupancy * ab_bytes_per_stage * self.num_ab_stage
            - self.occupancy * fixed_overhead
        ) // (self.occupancy * c_bytes_per_stage)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 288-288 / 第 288-288 行

~~~~python
        # SMEM layouts
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 289-297 / 第 289-297 行

~~~~python
        self.a_smem_layout_staged = utils.sm100.make_smem_layout_a(
            tiled_mma, self.mma_tiler, self.a_dtype, self.num_ab_stage
        )
        self.b_smem_layout_staged = utils.sm100.make_smem_layout_b(
            tiled_mma, self.mma_tiler, self.b_dtype, self.num_ab_stage
        )
        self.c_smem_layout_staged = utils.sm100.make_smem_layout_epi(
            self.c_dtype, self.c_layout, self.epi_tile, self.num_c_stage
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 299-299 / 第 299-299 行

~~~~python
        # Tensor memory allocation columns
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 300-306 / 第 300-306 行

~~~~python
        acc_shape = tiled_mma.partition_shape_C(self.mma_tiler[:2])
        tCtAcc_fake = tiled_mma.make_fragment_C(
            cute.append(acc_shape, self.num_acc_stage)
        )
        self.num_tmem_alloc_cols = utils.get_num_tmem_alloc_cols(
            tCtAcc_fake, arch=self.arch
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 308-308 / 第 308-308 行

~~~~python
        # TMA load bytes
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 309-314 / 第 309-314 行

~~~~python
        atom_thr_size = cute.size(tiled_mma.thr_id.shape)
        a_smem_layout = cute.slice_(self.a_smem_layout_staged, (None, None, None, 0))
        b_smem_layout = cute.slice_(self.b_smem_layout_staged, (None, None, None, 0))
        a_copy_size = cute.size_in_bytes(self.a_dtype, a_smem_layout)
        b_copy_size = cute.size_in_bytes(self.b_dtype, b_smem_layout)
        self.num_tma_load_bytes = (a_copy_size + b_copy_size) * atom_thr_size
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 316-325 / 第 316-325 行

~~~~python
    def get_workspace_size(self, expert_cnt: int) -> int:
        """
        Workspace size for expert-wise TMA descriptors.

        2Dx3D: Need C desc per expert -> expert_cnt * TensormapDescBytes
        2Dx2D: Need A and B desc per expert -> 2 * expert_cnt * TensormapDescBytes
        """
        return MoEGroupedGemmTensormapConstructor.get_workspace_size(
            self.scenario, expert_cnt
        )
~~~~

**EN**: Defines `get_workspace_size`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `get_workspace_size`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 327-351 / 第 327-351 行

~~~~python
    @cute.jit
    def __call__(
        self,
        mat_a: cute.Tensor,  # PyTorch mat_a
        mat_b: cute.Tensor,  # PyTorch mat_b
        out: cute.Tensor,  # PyTorch output
        offs: cute.Tensor,  # (experts,) cumsum
        bias: Optional[cute.Tensor],
        workspace: cute.Tensor,
        max_active_clusters: cutlass.Constexpr,
        stream: cuda.CUstream,
    ) -> None:
        """
        Launch the grouped GEMM kernel.

        This method:
        1. Transforms PyTorch tensors to GEMM domain tensors
        2. Infers dtypes and major modes from GEMM domain tensors
        3. Sets up kernel attributes
        4. Creates TMA atoms for A, B, C
        5. Creates scheduler parameters
        6. Launches the kernel
        """
        if cutlass.const_expr(bias is not None):
            raise NotImplementedError("bias is not supported yet (align with torch).")
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 353-355 / 第 353-355 行

~~~~python
        # =====================================================================
        # Step 1: Transform PyTorch tensors to GEMM domain (fake MNKL)
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 357-358 / 第 357-358 行

~~~~python
        c1 = cutlass.Int32(1)
        c0 = cutlass.Int32(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 360-369 / 第 360-369 行

~~~~python
        if cutlass.const_expr(self.scenario == "2Dx3D"):
            # mat_a: (tokens_sum, hidden) -> A_cute: (fake_m, k, 1)
            tokens_sum, hidden = mat_a.shape
            a_gemm = cute.make_tensor(
                mat_a.iterator,
                cute.make_layout(
                    (tokens_sum, hidden, c1),
                    stride=(mat_a.stride[0], mat_a.stride[1], c0),
                ),
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 371-371 / 第 371-371 行

~~~~python
            # mat_b: (experts, hidden, intermediate) -> B_cute: (n, k, fake_l)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 372-379 / 第 372-379 行

~~~~python
            experts, hidden_b, intermediate = mat_b.shape
            b_gemm = cute.make_tensor(
                mat_b.iterator,
                cute.make_layout(
                    (intermediate, hidden_b, experts),
                    stride=(mat_b.stride[2], mat_b.stride[1], mat_b.stride[0]),
                ),
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 381-381 / 第 381-381 行

~~~~python
            # out: (tokens_sum, intermediate) -> C_cute: (fake_m, n, 1)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 382-388 / 第 382-388 行

~~~~python
            c_gemm = cute.make_tensor(
                out.iterator,
                cute.make_layout(
                    (tokens_sum, intermediate, c1),
                    stride=(out.stride[0], out.stride[1], c0),
                ),
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 390-392 / 第 390-392 行

~~~~python
            expert_cnt = experts
            intermediate_dim = intermediate
            hidden_dim = hidden
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 394-403 / 第 394-403 行

~~~~python
        else:  # 2Dx2D
            # mat_a: (hidden, tokens_sum) -> A_cute: (m, fake_k, 1)
            hidden, tokens_sum = mat_a.shape
            a_gemm = cute.make_tensor(
                mat_a.iterator,
                cute.make_layout(
                    (hidden, tokens_sum, c1),
                    stride=(mat_a.stride[0], mat_a.stride[1], c0),
                ),
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 405-405 / 第 405-405 行

~~~~python
            # mat_b: (tokens_sum, intermediate) -> B_cute: (n, fake_k, 1)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 406-413 / 第 406-413 行

~~~~python
            tokens_sum_b, intermediate = mat_b.shape
            b_gemm = cute.make_tensor(
                mat_b.iterator,
                cute.make_layout(
                    (intermediate, tokens_sum_b, c1),
                    stride=(mat_b.stride[1], mat_b.stride[0], c0),
                ),
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 415-415 / 第 415-415 行

~~~~python
            # out: (experts, hidden, intermediate) -> C_cute: (m, n, fake_l)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 416-423 / 第 416-423 行

~~~~python
            experts, hidden_c, intermediate_c = out.shape
            c_gemm = cute.make_tensor(
                out.iterator,
                cute.make_layout(
                    (hidden_c, intermediate_c, experts),
                    stride=(out.stride[1], out.stride[2], out.stride[0]),
                ),
            )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 425-427 / 第 425-427 行

~~~~python
            expert_cnt = experts
            intermediate_dim = intermediate
            hidden_dim = hidden
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 429-431 / 第 429-431 行

~~~~python
        # =====================================================================
        # Step 2: Infer dtypes and major modes from GEMM domain tensors
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 433-438 / 第 433-438 行

~~~~python
        self.a_dtype: Type[cutlass.Numeric] = a_gemm.element_type
        self.b_dtype: Type[cutlass.Numeric] = b_gemm.element_type
        self.c_dtype: Type[cutlass.Numeric] = c_gemm.element_type
        self.a_major_mode = utils.LayoutEnum.from_tensor(a_gemm).mma_major_mode()
        self.b_major_mode = utils.LayoutEnum.from_tensor(b_gemm).mma_major_mode()
        self.c_layout = utils.LayoutEnum.from_tensor(c_gemm)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 440-442 / 第 440-442 行

~~~~python
        # =====================================================================
        # Step 3: Setup kernel attributes
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 444-456 / 第 444-456 行

~~~~python
        k = self.mma_tiler_mnk[2]
        a_tile_bits = self.a_dtype.width * k
        b_tile_bits = self.b_dtype.width * k
        if cutlass.const_expr(a_tile_bits % 256 != 0):
            raise ValueError(
                f"a_dtype ({self.a_dtype.width}b) * mma_tiler K ({k}) = "
                f"{a_tile_bits}b, must be a multiple of 256b (MMA instruction K width)"
            )
        if cutlass.const_expr(b_tile_bits % 256 != 0):
            raise ValueError(
                f"b_dtype ({self.b_dtype.width}b) * mma_tiler K ({k}) = "
                f"{b_tile_bits}b, must be a multiple of 256b (MMA instruction K width)"
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 458-459 / 第 458-459 行

~~~~python
        self._setup_attributes()
        tiled_mma = self._create_tiled_mma()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 461-465 / 第 461-465 行

~~~~python
        # =====================================================================
        # Step 4: Create TMA atoms for A, B, C
        # =====================================================================

        # TMA load for A
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 466-477 / 第 466-477 行

~~~~python
        a_op = utils.sm100.cluster_shape_to_tma_atom_A(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        a_smem_layout = cute.slice_(self.a_smem_layout_staged, (None, None, None, 0))
        tma_atom_a, tma_tensor_a = cute.nvgpu.make_tiled_tma_atom_A(
            a_op,
            a_gemm,
            a_smem_layout,
            self.mma_tiler,
            tiled_mma,
            self.cluster_layout_vmnk.shape,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 479-479 / 第 479-479 行

~~~~python
        # TMA load for B
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 480-491 / 第 480-491 行

~~~~python
        b_op = utils.sm100.cluster_shape_to_tma_atom_B(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        b_smem_layout = cute.slice_(self.b_smem_layout_staged, (None, None, None, 0))
        tma_atom_b, tma_tensor_b = cute.nvgpu.make_tiled_tma_atom_B(
            b_op,
            b_gemm,
            b_smem_layout,
            self.mma_tiler,
            tiled_mma,
            self.cluster_layout_vmnk.shape,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 493-493 / 第 493-493 行

~~~~python
        # TMA store for C (or TMA reduce for accumulate_on_output)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 494-497 / 第 494-497 行

~~~~python
        if cutlass.const_expr(self.accumulate_on_output):
            c_tma_op = cpasync.CopyReduceBulkTensorTileS2GOp()
        else:
            c_tma_op = cpasync.CopyBulkTensorTileS2GOp()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 499-502 / 第 499-502 行

~~~~python
        epi_smem_layout = cute.select(self.c_smem_layout_staged, mode=[0, 1])
        tma_atom_c, tma_tensor_c = cpasync.make_tiled_tma_atom(
            c_tma_op, c_gemm, epi_smem_layout, self.epi_tile
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 504-506 / 第 504-506 行

~~~~python
        # =====================================================================
        # Step 5: Create MoEStaticSchedulerParams and compute grid
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 508-513 / 第 508-513 行

~~~~python
        sched_params = MoEStaticSchedulerParams(
            scenario=self.scenario,
            expert_shape=(expert_cnt, intermediate_dim, hidden_dim),
            cta_tile_shape_mnk=self.cta_tile_shape_mnk,
            cluster_shape_mn=self.cluster_shape_mn,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 515-517 / 第 515-517 行

~~~~python
        grid = MoEStaticSchedulerParams.get_grid_shape(
            sched_params, max_active_clusters
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 519-529 / 第 519-529 行

~~~~python
        # =====================================================================
        # Step 5.5: Launch desc init kernel (if separate_tensormap_init)
        # =====================================================================
        #
        # Pre-initialize expert-wise TMA descriptors in workspace before
        # the main kernel. Stream ordering guarantees completion before
        # the main kernel starts.
        #
        # 2Dx3D: C desc per expert  (C has dynamic fake_m per expert)
        # 2Dx2D: A,B desc per expert (A,B have dynamic fake_k per expert)
        #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 531-550 / 第 531-550 行

~~~~python
        if cutlass.const_expr(self.separate_tensormap_init):
            self.desc_init_kernel(
                tiled_mma,
                a_gemm,
                b_gemm,
                c_gemm,
                offs,
                expert_cnt,
                workspace.iterator,
                self.cluster_layout_vmnk,
                self.a_smem_layout_staged,
                self.b_smem_layout_staged,
                self.c_smem_layout_staged,
                self.epi_tile,
            ).launch(
                grid=(expert_cnt, 1, 1),
                block=[32, 1, 1],
                stream=stream,
                min_blocks_per_mp=1,
            )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 552-554 / 第 552-554 行

~~~~python
        # =====================================================================
        # Step 6: Launch kernel
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 556-581 / 第 556-581 行

~~~~python
        self.kernel(
            tiled_mma,
            tma_atom_a,
            tma_tensor_a,
            tma_atom_b,
            tma_tensor_b,
            tma_atom_c,
            tma_tensor_c,
            a_gemm,
            b_gemm,
            c_gemm,
            offs,
            sched_params,
            workspace.iterator,
            self.cluster_layout_vmnk,
            self.a_smem_layout_staged,
            self.b_smem_layout_staged,
            self.c_smem_layout_staged,
            self.epi_tile,
        ).launch(
            grid=grid,
            block=[self.threads_per_cta, 1, 1],
            cluster=(*self.cluster_shape_mn, 1),
            stream=stream,
            min_blocks_per_mp=self.occupancy,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 583-583 / 第 583-583 行

~~~~python
    # GPU device kernel: TMA descriptor initialization
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 584-611 / 第 584-611 行

~~~~python
    @cute.kernel
    def desc_init_kernel(
        self,
        tiled_mma: cute.TiledMma,
        a_gemm: cute.Tensor,  # GEMM domain A (fake MNKL)
        b_gemm: cute.Tensor,  # GEMM domain B (fake MNKL)
        c_gemm: cute.Tensor,  # GEMM domain C (fake MNKL)
        offs: cute.Tensor,  # (experts,) cumsum
        expert_cnt: Union[cutlass.Int32, int],
        workspace_ptr: Pointer,
        cluster_layout_vmnk: cute.Layout,
        a_smem_layout_staged: cute.ComposedLayout,
        b_smem_layout_staged: cute.ComposedLayout,
        c_smem_layout_staged: Union[cute.Layout, cute.ComposedLayout],
        epi_tile: cute.Tile,
    ):
        """
        Separate kernel to pre-initialize expert-wise TMA descriptors.

        Grid: (expert_cnt, 1, 1) - one block per expert
        Block: (32, 1, 1) - one warp per block

        Each block constructs and writes TMA descriptors for one expert
        to the pre-allocated workspace buffer.

        2Dx3D: Creates C descriptor per expert (C has dynamic fake_m per expert)
        2Dx2D: Creates A and B descriptors per expert (A/B have dynamic fake_k per expert)
        """
~~~~

**EN**: Declares `desc_init_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 把 `desc_init_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 612-614 / 第 612-614 行

~~~~python
        # =================================================================
        # Reconstruct TMA constructor with explicit attributes
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 616-618 / 第 616-618 行

~~~~python
        a_smem_layout = cute.slice_(a_smem_layout_staged, (None, None, None, 0))
        b_smem_layout = cute.slice_(b_smem_layout_staged, (None, None, None, 0))
        epi_smem_layout = cute.select(c_smem_layout_staged, mode=[0, 1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 620-629 / 第 620-629 行

~~~~python
        a_tma_op = utils.sm100.cluster_shape_to_tma_atom_A(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        b_tma_op = utils.sm100.cluster_shape_to_tma_atom_B(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        if cutlass.const_expr(self.accumulate_on_output):
            c_tma_op = cpasync.CopyReduceBulkTensorTileS2GOp()
        else:
            c_tma_op = cpasync.CopyBulkTensorTileS2GOp()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 631-651 / 第 631-651 行

~~~~python
        tensormap_ctor = MoEGroupedGemmTensormapConstructor(
            scenario=self.scenario,
            a_dtype=self.a_dtype,
            b_dtype=self.b_dtype,
            c_dtype=self.c_dtype,
            a_smem_layout=a_smem_layout,
            b_smem_layout=b_smem_layout,
            epi_smem_layout=epi_smem_layout,
            a_tma_op=a_tma_op,
            b_tma_op=b_tma_op,
            c_tma_op=c_tma_op,
            tiled_mma=tiled_mma,
            mma_tiler=self.mma_tiler,
            cluster_layout_vmnk_shape=cluster_layout_vmnk.shape,
            epi_tile=epi_tile,
            a_tensor=a_gemm,
            b_tensor=b_gemm,
            c_tensor=c_gemm,
            offs=offs,
            workspace_ptr=workspace_ptr,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 653-655 / 第 653-655 行

~~~~python
        # =================================================================
        # Each block constructs descriptors for one expert
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 657-658 / 第 657-658 行

~~~~python
        expert_idx, _, _ = cute.arch.block_idx()
        tensormap_ctor.construct_and_write(expert_idx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 660-660 / 第 660-660 行

~~~~python
    # GPU device kernel: main GEMM kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 661-688 / 第 661-688 行

~~~~python
    @cute.kernel
    def kernel(
        self,
        tiled_mma: cute.TiledMma,
        tma_atom_a: cute.CopyAtom,
        tma_tensor_a: cute.Tensor,
        tma_atom_b: cute.CopyAtom,
        tma_tensor_b: cute.Tensor,
        tma_atom_c: cute.CopyAtom,
        tma_tensor_c: cute.Tensor,
        a_gemm: cute.Tensor,  # GEMM domain A (fake MNKL)
        b_gemm: cute.Tensor,  # GEMM domain B (fake MNKL)
        c_gemm: cute.Tensor,  # GEMM domain C (fake MNKL)
        offs: cute.Tensor,  # (experts,) cumsum
        sched_params: MoEStaticSchedulerParams,
        workspace_ptr: Pointer,
        cluster_layout_vmnk: cute.Layout,
        a_smem_layout_staged: cute.ComposedLayout,
        b_smem_layout_staged: cute.ComposedLayout,
        c_smem_layout_staged: Union[cute.Layout, cute.ComposedLayout],
        epi_tile: cute.Tile,
    ):
        """
        GPU device kernel for MoE Grouped GEMM.

        Warp specialization:
        - Warps 0-3: Epilogue warps (TMEM -> RMEM -> SMEM -> GMEM)
        - Warp 4: MMA warp (tcgen05.mma)
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 689-704 / 第 689-704 行

~~~~python
        - Warp 5: TMA load warp (also prefetches expert-wise TMA descriptors)

        The kernel uses MoEStaticPersistentTileScheduler to iterate over tiles
        across all experts. For each tile:
        1. TMA load warp fetches A/B tiles using get_gmem_tensor
        2. MMA warp performs matrix multiply-accumulate
        3. Epilogue warps store results using TMA store/reduce

        Note: Python objects holding MLIR values cannot be kernel params.
        The following are constructed inside the kernel from individually-passed params:
        - tensormap_ctor: MoEGroupedGemmTensormapConstructor (online tensormap builder)
        - ext: GroupedMmSchedExtension (domain conversion + TMA desc selection)
        """
        # =================================================================
        # Reconstruct dicts that can't be passed as kernel params
        # =================================================================
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 706-706 / 第 706-706 行

~~~~python
        # Construct TMA descriptor creator and scheduler extension
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 707-709 / 第 707-709 行

~~~~python
        a_smem_layout = cute.slice_(a_smem_layout_staged, (None, None, None, 0))
        b_smem_layout = cute.slice_(b_smem_layout_staged, (None, None, None, 0))
        epi_smem_layout = cute.select(c_smem_layout_staged, mode=[0, 1])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 711-720 / 第 711-720 行

~~~~python
        a_tma_op = utils.sm100.cluster_shape_to_tma_atom_A(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        b_tma_op = utils.sm100.cluster_shape_to_tma_atom_B(
            self.cluster_shape_mn, tiled_mma.thr_id
        )
        if cutlass.const_expr(self.accumulate_on_output):
            c_tma_op = cpasync.CopyReduceBulkTensorTileS2GOp()
        else:
            c_tma_op = cpasync.CopyBulkTensorTileS2GOp()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 722-745 / 第 722-745 行

~~~~python
        tensormap_ctor = MoEGroupedGemmTensormapConstructor(
            scenario=self.scenario,
            a_dtype=self.a_dtype,
            b_dtype=self.b_dtype,
            c_dtype=self.c_dtype,
            a_smem_layout=a_smem_layout,
            b_smem_layout=b_smem_layout,
            epi_smem_layout=epi_smem_layout,
            a_tma_op=a_tma_op,
            b_tma_op=b_tma_op,
            c_tma_op=c_tma_op,
            tiled_mma=tiled_mma,
            mma_tiler=self.mma_tiler,
            cluster_layout_vmnk_shape=cluster_layout_vmnk.shape,
            epi_tile=epi_tile,
            a_tensor=a_gemm,
            b_tensor=b_gemm,
            c_tensor=c_gemm,
            offs=offs,
            workspace_ptr=workspace_ptr,
        )
        ext = GroupedMmSchedExtension(
            scenario=self.scenario, tensormap_ctor=tensormap_ctor
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 747-749 / 第 747-749 行

~~~~python
        # =================================================================
        # Kernel setup
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 751-753 / 第 751-753 行

~~~~python
        warp_idx = cute.arch.warp_idx()
        warp_idx = cute.arch.make_warp_uniform(warp_idx)
        use_2cta_instrs = cute.size(tiled_mma.thr_id.shape) == 2
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 755-755 / 第 755-755 行

~~~~python
        # CTA/thread coordinates
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 756-765 / 第 756-765 行

~~~~python
        bidx, bidy, bidz = cute.arch.block_idx()
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

### Lines 767-769 / 第 767-769 行

~~~~python
        # =================================================================
        # SharedStorage
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 771-782 / 第 771-782 行

~~~~python
        @cute.struct
        class SharedStorage:
            ab_full_mbar_ptr: cute.struct.MemRange[cutlass.Int64, self.num_ab_stage * 2]
            acc_full_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_acc_stage * 2
            ]
            sched_buf: cute.struct.MemRange[cutlass.Int32, self.num_sched_stages * 4]
            sched_mbar_ptr: cute.struct.MemRange[
                cutlass.Int64, self.num_sched_stages * 2
            ]
            tmem_dealloc_mbar_ptr: cutlass.Int64
            tmem_holding_buf: cutlass.Int32
~~~~

**EN**: Defines `SharedStorage`, a reusable Python class that packages configuration and behavior for this example. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `SharedStorage`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 784-785 / 第 784-785 行

~~~~python
        smem = utils.SmemAllocator()
        storage = smem.allocate(SharedStorage)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 787-791 / 第 787-791 行

~~~~python
        # =================================================================
        # Pipelines
        # =================================================================

        # AB pipeline (TMA load → MMA)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 792-805 / 第 792-805 行

~~~~python
        ab_pipeline_producer_group = pipeline.CooperativeGroup(pipeline.Agent.Thread)
        num_tma_producer = self.num_mcast_ctas_a + self.num_mcast_ctas_b - 1
        ab_pipeline_consumer_group = pipeline.CooperativeGroup(
            pipeline.Agent.Thread, num_tma_producer
        )
        ab_producer, ab_consumer = pipeline.PipelineTmaUmma.create(
            barrier_storage=storage.ab_full_mbar_ptr.data_ptr(),
            num_stages=self.num_ab_stage,
            producer_group=ab_pipeline_producer_group,
            consumer_group=ab_pipeline_consumer_group,
            tx_count=self.num_tma_load_bytes,
            cta_layout_vmnk=cluster_layout_vmnk,
            defer_sync=True,
        ).make_participants()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 807-807 / 第 807-807 行

~~~~python
        # ACC pipeline (MMA → epilogue)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 808-822 / 第 808-822 行

~~~~python
        acc_pipeline_producer_group = pipeline.CooperativeGroup(pipeline.Agent.Thread)
        num_acc_consumer_threads = (
            len(self.epilogue_warp_id) * 32 * (2 if use_2cta_instrs else 1)
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
            defer_sync=True,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 824-824 / 第 824-824 行

~~~~python
        # Scheduler pipeline (sched warp → tma/mma/epi warps)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 825-838 / 第 825-838 行

~~~~python
        sched_producer_group = pipeline.CooperativeGroup(pipeline.Agent.Thread, 32)
        num_sched_consumer_threads = 32 * len(
            (self.tma_warp_id, self.mma_warp_id, *self.epilogue_warp_id)
        )
        sched_consumer_group = pipeline.CooperativeGroup(
            pipeline.Agent.Thread, num_sched_consumer_threads
        )
        sched_pipeline = pipeline.PipelineAsync.create(
            num_stages=self.num_sched_stages,
            producer_group=sched_producer_group,
            consumer_group=sched_consumer_group,
            barrier_storage=storage.sched_mbar_ptr.data_ptr(),
            defer_sync=True,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 840-840 / 第 840-840 行

~~~~python
        # TMEM allocator
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 841-851 / 第 841-851 行

~~~~python
        tmem_alloc_barrier = pipeline.NamedBarrier(
            barrier_id=self.tmem_alloc_sync_bar_id,
            num_threads=32 * len((self.mma_warp_id, *self.epilogue_warp_id)),
        )
        tmem = utils.TmemAllocator(
            storage.tmem_holding_buf.ptr,
            barrier_for_retrieve=tmem_alloc_barrier,
            allocator_warp_id=self.epilogue_warp_id[0],
            is_two_cta=use_2cta_instrs,
            two_cta_tmem_dealloc_mbar_ptr=storage.tmem_dealloc_mbar_ptr.ptr,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 853-853 / 第 853-853 行

~~~~python
        # Cluster barrier sync after init
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 854-854 / 第 854-854 行

~~~~python
        pipeline_init_arrive(cluster_shape_mn=self.cluster_shape_mn, is_relaxed=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 856-860 / 第 856-860 行

~~~~python
        # =================================================================
        # SMEM tensors A/B
        # =================================================================

        # (MMA, MMA_M, MMA_K, STAGE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 861-873 / 第 861-873 行

~~~~python
        sA = smem.allocate_tensor(
            element_type=self.a_dtype,
            layout=a_smem_layout_staged.outer,
            byte_alignment=128,
            swizzle=a_smem_layout_staged.inner,
        )
        # (MMA, MMA_N, MMA_K, STAGE)
        sB = smem.allocate_tensor(
            element_type=self.b_dtype,
            layout=b_smem_layout_staged.outer,
            byte_alignment=128,
            swizzle=b_smem_layout_staged.inner,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 875-875 / 第 875-875 行

~~~~python
        # Multicast masks
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 876-884 / 第 876-884 行

~~~~python
        a_full_mcast_mask = None
        b_full_mcast_mask = None
        if cutlass.const_expr(self.is_a_mcast or self.is_b_mcast or use_2cta_instrs):
            a_full_mcast_mask = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mcast_mode=2
            )
            b_full_mcast_mask = cpasync.create_tma_multicast_mask(
                cluster_layout_vmnk, block_in_cluster_coord_vmnk, mcast_mode=1
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 886-887 / 第 886-887 行

~~~~python
        # MMA fragments (SMEM → TMEM partitions)
        # (MMA, MMA_M, MMA_K, STAGE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 888-896 / 第 888-896 行

~~~~python
        tCrA = tiled_mma.make_fragment_A(sA)
        # (MMA, MMA_N, MMA_K, STAGE)
        tCrB = tiled_mma.make_fragment_B(sB)
        # (MMA, MMA_M, MMA_N)
        acc_shape = tiled_mma.partition_shape_C(self.mma_tiler[:2])
        # (MMA, MMA_M, MMA_N, STAGE)
        tCtAcc_fake = tiled_mma.make_fragment_C(
            cute.append(acc_shape, self.num_acc_stage)
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 898-898 / 第 898-898 行

~~~~python
        # Cluster wait before TMEM alloc
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 899-899 / 第 899-899 行

~~~~python
        pipeline_init_wait(cluster_shape_mn=self.cluster_shape_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 901-903 / 第 901-903 行

~~~~python
        # =================================================================
        # Scheduler warp (warp 6)
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 905-911 / 第 905-911 行

~~~~python
        sched_buf_ptr = storage.sched_buf.data_ptr()
        sched_copy_atom = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(), cutlass.Int32, num_bits_per_copy=128
        )
        sched_buf_tensor = cute.make_tensor(
            sched_buf_ptr, cute.make_layout((4, self.num_sched_stages), stride=(1, 4))
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 913-916 / 第 913-916 行

~~~~python
        if warp_idx == self.sched_warp_id:
            scheduler = MoEStaticPersistentTileScheduler.create(
                sched_params, offs, cute.arch.block_idx(), cute.arch.grid_dim()
            )
~~~~

**EN**: Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 918-920 / 第 918-920 行

~~~~python
            sched_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_sched_stages
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 922-922 / 第 922-922 行

~~~~python
            # Always produce the initial work_tile_info first
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 923-933 / 第 923-933 行

~~~~python
            work_tile_info = scheduler.initial_work_tile_info()
            sched_pipeline.producer_acquire(sched_producer_state)
            rmem = work_tile_info.to_rmem_tensor()
            cute.copy(
                sched_copy_atom,
                rmem,
                sched_buf_tensor[(None, sched_producer_state.index)],
            )
            cute.arch.fence_proxy("async.shared", space="cta")
            sched_pipeline.producer_commit(sched_producer_state)
            sched_producer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 935-935 / 第 935-935 行

~~~~python
            # Iterate remaining tiles starting from the first advance
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 936-948 / 第 936-948 行

~~~~python
            work_tile_info = scheduler.advance_to_next_work()
            while work_tile_info.is_valid_tile:
                ext.prefetch_for_expert(work_tile_info.expert_idx)
                sched_pipeline.producer_acquire(sched_producer_state)
                rmem = work_tile_info.to_rmem_tensor()
                cute.copy(
                    sched_copy_atom,
                    rmem,
                    sched_buf_tensor[(None, sched_producer_state.index)],
                )
                cute.arch.fence_proxy("async.shared", space="cta")
                sched_pipeline.producer_commit(sched_producer_state)
                sched_producer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 950-950 / 第 950-950 行

~~~~python
                work_tile_info = scheduler.advance_to_next_work()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 952-952 / 第 952-952 行

~~~~python
            # Write invalid sentinel (expert_idx = -1) so consumers exit
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 953-964 / 第 953-964 行

~~~~python
            sched_pipeline.producer_acquire(sched_producer_state)
            sentinel = MoEWorkTileInfo(
                cutlass.Int32(-1), cutlass.Int32(0), cutlass.Int32(0), cutlass.Int32(0)
            )
            rmem = sentinel.to_rmem_tensor()
            cute.copy(
                sched_copy_atom,
                rmem,
                sched_buf_tensor[(None, sched_producer_state.index)],
            )
            cute.arch.fence_proxy("async.shared", space="cta")
            sched_pipeline.producer_commit(sched_producer_state)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 966-966 / 第 966-966 行

~~~~python
            sched_pipeline.producer_tail(sched_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 968-970 / 第 968-970 行

~~~~python
        # =================================================================
        # TMA load warp (warp 5)
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 972-975 / 第 972-975 行

~~~~python
        if warp_idx == self.tma_warp_id:
            sched_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_sched_stages
            )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 977-977 / 第 977-977 行

~~~~python
            # Read initial work_tile_info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 978-988 / 第 978-988 行

~~~~python
            sched_pipeline.consumer_wait(sched_consumer_state)
            rmem = cute.make_rmem_tensor((4,), cutlass.Int32)
            cute.copy(
                sched_copy_atom,
                sched_buf_tensor[(None, sched_consumer_state.index)],
                rmem,
            )
            work_tile_info = MoEWorkTileInfo.from_rmem_tensor(rmem)
            cute.arch.fence_acq_rel_cta()
            sched_pipeline.consumer_release(sched_consumer_state)
            sched_consumer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 990-991 / 第 990-991 行

~~~~python
            while work_tile_info.is_valid_tile:
                k_tile_cnt = work_tile_info.k_tile_cnt
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 993-993 / 第 993-993 行

~~~~python
                # Get real GEMM domain tensors + TMA desc ptrs via extension
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 994-1005 / 第 994-1005 行

~~~~python
                real_a, desc_ptr_a = ext.get_gmem_tensor(
                    "a",
                    tma_tensor_a,
                    offs,
                    work_tile_info,
                )
                real_b, desc_ptr_b = ext.get_gmem_tensor(
                    "b",
                    tma_tensor_b,
                    offs,
                    work_tile_info,
                )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1007-1007 / 第 1007-1007 行

~~~~python
                # local_tile for this tile's A and B
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1008-1017 / 第 1008-1017 行

~~~~python
                gA_mkl = cute.local_tile(
                    real_a,
                    cute.slice_(self.mma_tiler, (None, 0, None)),
                    (None, None, None),
                )
                gB_nkl = cute.local_tile(
                    real_b,
                    cute.slice_(self.mma_tiler, (0, None, None)),
                    (None, None, None),
                )
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1019-1019 / 第 1019-1019 行

~~~~python
                # MMA partition for TMA
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1020-1022 / 第 1020-1022 行

~~~~python
                thr_mma = tiled_mma.get_slice(mma_tile_coord_v)
                tCgA = thr_mma.partition_A(gA_mkl)
                tCgB = thr_mma.partition_B(gB_nkl)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1024-1024 / 第 1024-1024 行

~~~~python
                # TMA partition
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1025-1044 / 第 1025-1044 行

~~~~python
                a_cta_layout = cute.make_layout(
                    cute.slice_(cluster_layout_vmnk, (0, 0, None, 0)).shape
                )
                b_cta_layout = cute.make_layout(
                    cute.slice_(cluster_layout_vmnk, (0, None, 0, 0)).shape
                )
                tAsA, tAgA = cpasync.tma_partition(
                    tma_atom_a,
                    block_in_cluster_coord_vmnk[2],
                    a_cta_layout,
                    cute.group_modes(sA, 0, 3),
                    cute.group_modes(tCgA, 0, 3),
                )
                tBsB, tBgB = cpasync.tma_partition(
                    tma_atom_b,
                    block_in_cluster_coord_vmnk[1],
                    b_cta_layout,
                    cute.group_modes(sB, 0, 3),
                    cute.group_modes(tCgB, 0, 3),
                )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1046-1046 / 第 1046-1046 行

~~~~python
                # Slice to current tile coords (L=0 for MoE, expert already selected)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1047-1051 / 第 1047-1051 行

~~~~python
                mma_tile_m = work_tile_info.tile_m_idx // cute.size(
                    tiled_mma.thr_id.shape
                )
                tAgA_slice = tAgA[(None, mma_tile_m, None, 0)]
                tBgB_slice = tBgB[(None, work_tile_info.tile_n_idx, None, 0)]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1053-1053 / 第 1053-1053 行

~~~~python
                # TMA load loop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1054-1055 / 第 1054-1055 行

~~~~python
                ab_producer.reset()
                peek_ab_empty_status = ab_producer.try_acquire()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1057-1077 / 第 1057-1077 行

~~~~python
                for k_tile in cutlass.range(0, k_tile_cnt, 1, unroll=1):
                    handle = ab_producer.acquire_and_advance(peek_ab_empty_status)
                    peek_ab_empty_status = cutlass.Boolean(1)
                    if handle.count + 1 < k_tile_cnt:
                        peek_ab_empty_status = ab_producer.try_acquire()
                    cute.copy(
                        tma_atom_a,
                        tAgA_slice[(None, handle.count)],
                        tAsA[(None, handle.index)],
                        tma_bar_ptr=handle.barrier,
                        tma_desc_ptr=desc_ptr_a,
                        mcast_mask=a_full_mcast_mask,
                    )
                    cute.copy(
                        tma_atom_b,
                        tBgB_slice[(None, handle.count)],
                        tBsB[(None, handle.index)],
                        tma_bar_ptr=handle.barrier,
                        tma_desc_ptr=desc_ptr_b,
                        mcast_mask=b_full_mcast_mask,
                    )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1079-1079 / 第 1079-1079 行

~~~~python
                # Read next work_tile_info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1080-1090 / 第 1080-1090 行

~~~~python
                sched_pipeline.consumer_wait(sched_consumer_state)
                rmem = cute.make_rmem_tensor((4,), cutlass.Int32)
                cute.copy(
                    sched_copy_atom,
                    sched_buf_tensor[(None, sched_consumer_state.index)],
                    rmem,
                )
                work_tile_info = MoEWorkTileInfo.from_rmem_tensor(rmem)
                cute.arch.fence_acq_rel_cta()
                sched_pipeline.consumer_release(sched_consumer_state)
                sched_consumer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1092-1092 / 第 1092-1092 行

~~~~python
            ab_producer.tail()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1094-1096 / 第 1094-1096 行

~~~~python
        # =================================================================
        # MMA warp (warp 4)
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1098-1102 / 第 1098-1102 行

~~~~python
        if warp_idx == self.mma_warp_id:
            # Retrieve TMEM
            tmem.wait_for_alloc()
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
            tCtAcc_base = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1104-1109 / 第 1104-1109 行

~~~~python
            acc_producer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Producer, self.num_acc_stage
            )
            sched_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_sched_stages
            )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1111-1111 / 第 1111-1111 行

~~~~python
            # Read initial work_tile_info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1112-1122 / 第 1112-1122 行

~~~~python
            sched_pipeline.consumer_wait(sched_consumer_state)
            rmem = cute.make_rmem_tensor((4,), cutlass.Int32)
            cute.copy(
                sched_copy_atom,
                sched_buf_tensor[(None, sched_consumer_state.index)],
                rmem,
            )
            work_tile_info = MoEWorkTileInfo.from_rmem_tensor(rmem)
            cute.arch.fence_acq_rel_cta()
            sched_pipeline.consumer_release(sched_consumer_state)
            sched_consumer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1124-1127 / 第 1124-1127 行

~~~~python
            while work_tile_info.is_valid_tile:
                k_tile_cnt = work_tile_info.k_tile_cnt
                if is_leader_cta:
                    tCtAcc = tCtAcc_base[(None, None, None, acc_producer_state.index)]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1129-1129 / 第 1129-1129 行

~~~~python
                    # AB consumer mainloop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1130-1134 / 第 1130-1134 行

~~~~python
                    ab_consumer.reset()
                    peek_ab_full_status = cutlass.Boolean(1)
                    if k_tile_cnt > 0:
                        peek_ab_full_status = ab_consumer.try_wait()
                        acc_pipeline.producer_acquire(acc_producer_state)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1136-1146 / 第 1136-1146 行

~~~~python
                    for k_tile in cutlass.range(0, k_tile_cnt, 1, unroll=1):
                        handle = ab_consumer.wait_and_advance(peek_ab_full_status)
                        peek_ab_full_status = cutlass.Boolean(1)
                        if handle.count + 1 < k_tile_cnt:
                            peek_ab_full_status = ab_consumer.try_wait()
                        tiled_mma.set(tcgen05.Field.ACCUMULATE, k_tile != 0)
                        tile_crd = (None, None, None, handle.index)
                        cute.gemm(
                            tiled_mma, tCtAcc, tCrA[tile_crd], tCrB[tile_crd], tCtAcc
                        )
                        handle.release()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1148-1151 / 第 1148-1151 行

~~~~python
                    if k_tile_cnt > 0:
                        acc_pipeline.producer_commit(acc_producer_state)
                if k_tile_cnt > 0:
                    acc_producer_state.advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1153-1153 / 第 1153-1153 行

~~~~python
                # Read next work_tile_info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1154-1164 / 第 1154-1164 行

~~~~python
                sched_pipeline.consumer_wait(sched_consumer_state)
                rmem = cute.make_rmem_tensor((4,), cutlass.Int32)
                cute.copy(
                    sched_copy_atom,
                    sched_buf_tensor[(None, sched_consumer_state.index)],
                    rmem,
                )
                work_tile_info = MoEWorkTileInfo.from_rmem_tensor(rmem)
                cute.arch.fence_acq_rel_cta()
                sched_pipeline.consumer_release(sched_consumer_state)
                sched_consumer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1166-1166 / 第 1166-1166 行

~~~~python
            acc_pipeline.producer_tail(acc_producer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1168-1170 / 第 1168-1170 行

~~~~python
        # =================================================================
        # SMEM tensor C (allocated after MMA section, same as dense)
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1172-1177 / 第 1172-1177 行

~~~~python
        sC = smem.allocate_tensor(
            element_type=self.c_dtype,
            layout=c_smem_layout_staged.outer,
            byte_alignment=128,
            swizzle=c_smem_layout_staged.inner,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1179-1181 / 第 1179-1181 行

~~~~python
        # =================================================================
        # Epilogue warps (warps 0-3)
        # =================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1183-1188 / 第 1183-1188 行

~~~~python
        if warp_idx < self.mma_warp_id:
            # Allocate TMEM
            tmem.allocate(self.num_tmem_alloc_cols)
            tmem.wait_for_alloc()
            tmem_ptr = tmem.retrieve_ptr(self.acc_dtype)
            tCtAcc_base = cute.make_tensor(tmem_ptr, tCtAcc_fake.layout)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1190-1202 / 第 1190-1202 行

~~~~python
            acc_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_acc_stage
            )
            sched_consumer_state = pipeline.make_pipeline_state(
                pipeline.PipelineUserType.Consumer, self.num_sched_stages
            )
            c_producer_group = pipeline.CooperativeGroup(
                pipeline.Agent.Thread,
                32 * len(self.epilogue_warp_id),
            )
            c_pipeline = pipeline.PipelineTmaStore.create(
                num_stages=self.num_c_stage, producer_group=c_producer_group
            )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1204-1207 / 第 1204-1207 行

~~~~python
            epilog_sync_barrier = pipeline.NamedBarrier(
                barrier_id=self.epilog_sync_bar_id,
                num_threads=32 * len(self.epilogue_warp_id),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1209-1211 / 第 1209-1211 行

~~~~python
            # Epilogue copy setup (same for all tiles, depends only on shapes)
            # Transform ACC layout: ((ATOM_M, ATOM_N), MMA_M, MMA_N, STAGE)
            #                    -> ((ATOM_M, MMA_M), (ATOM_N, MMA_N), STAGE)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1212-1212 / 第 1212-1212 行

~~~~python
            tCtAcc_transformed = transform_partitioned_tensor_layout(tCtAcc_base)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1214-1214 / 第 1214-1214 行

~~~~python
            num_tiles_executed = cutlass.Int32(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1216-1216 / 第 1216-1216 行

~~~~python
            # Read initial work_tile_info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1217-1227 / 第 1217-1227 行

~~~~python
            sched_pipeline.consumer_wait(sched_consumer_state)
            rmem = cute.make_rmem_tensor((4,), cutlass.Int32)
            cute.copy(
                sched_copy_atom,
                sched_buf_tensor[(None, sched_consumer_state.index)],
                rmem,
            )
            work_tile_info = MoEWorkTileInfo.from_rmem_tensor(rmem)
            cute.arch.fence_acq_rel_cta()
            sched_pipeline.consumer_release(sched_consumer_state)
            sched_consumer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1229-1237 / 第 1229-1237 行

~~~~python
            while work_tile_info.is_valid_tile:
                k_tile_cnt = work_tile_info.k_tile_cnt
                # Get real C tensor + TMA desc ptr via extension
                real_c, desc_ptr_c = ext.get_gmem_tensor(
                    "c",
                    tma_tensor_c,
                    offs,
                    work_tile_info,
                )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1239-1239 / 第 1239-1239 行

~~~~python
                # local_tile + partition for C
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1240-1247 / 第 1240-1247 行

~~~~python
                gC_mnl = cute.local_tile(
                    real_c,
                    cute.slice_(self.mma_tiler, (None, None, 0)),
                    (None, None, None),
                )
                thr_mma = tiled_mma.get_slice(mma_tile_coord_v)
                tCgC = thr_mma.partition_C(gC_mnl)
                tCgC_transformed = transform_partitioned_tensor_layout(tCgC)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1249-1253 / 第 1249-1253 行

~~~~python
                mma_tile_coord_mnl = (
                    work_tile_info.tile_m_idx // cute.size(tiled_mma.thr_id.shape),
                    work_tile_info.tile_n_idx,
                    cutlass.Int32(0),
                )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1255-1255 / 第 1255-1255 行

~~~~python
                # Partition for TMEM → RMEM copy
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1256-1269 / 第 1256-1269 行

~~~~python
                tiled_copy_t2r, tTR_tAcc_base, tTR_rAcc = (
                    epilogue_tmem_copy_and_partition(
                        self,
                        tidx,
                        tCtAcc_transformed,
                        tCgC_transformed,
                        epi_tile,
                        use_2cta_instrs,
                    )
                )
                tTR_rC = cute.make_rmem_tensor(tTR_rAcc.shape, self.c_dtype)
                tiled_copy_r2s, tRS_rC, tRS_sC = epilogue_smem_copy_and_partition(
                    self, tiled_copy_t2r, tTR_rC, tidx, sC
                )
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1271-1271 / 第 1271-1271 行

~~~~python
                # TMA partition for C store (with expert-wise desc_ptr)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1272-1280 / 第 1272-1280 行

~~~~python
                tCgC_epi = cute.flat_divide(tCgC_transformed, epi_tile)
                bSG_sC, bSG_gC_partitioned = cpasync.tma_partition(
                    tma_atom_c,
                    0,
                    cute.make_layout(1),
                    cute.group_modes(sC, 0, 2),
                    cute.group_modes(tCgC_epi, 0, 2),
                )
                bSG_gC = bSG_gC_partitioned[(None, None, None, *mma_tile_coord_mnl)]
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1282-1282 / 第 1282-1282 行

~~~~python
                # Set TMEM buffer for current tile
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1283-1285 / 第 1283-1285 行

~~~~python
                tTR_tAcc = tTR_tAcc_base[
                    (None, None, None, None, None, acc_consumer_state.index)
                ]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1287-1287 / 第 1287-1287 行

~~~~python
                # Wait for accumulator buffer full
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1288-1289 / 第 1288-1289 行

~~~~python
                if k_tile_cnt > 0:
                    acc_pipeline.consumer_wait(acc_consumer_state)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1291-1292 / 第 1291-1292 行

~~~~python
                tTR_tAcc = cute.group_modes(tTR_tAcc, 3, cute.rank(tTR_tAcc))
                bSG_gC = cute.group_modes(bSG_gC, 1, cute.rank(bSG_gC))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 1294-1294 / 第 1294-1294 行

~~~~python
                # Store accumulator to global memory in subtiles
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1295-1296 / 第 1295-1296 行

~~~~python
                subtile_cnt = cute.size(tTR_tAcc.shape, mode=[3])
                num_prev_subtiles = num_tiles_executed * subtile_cnt
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1298-1305 / 第 1298-1305 行

~~~~python
                for subtile_idx in range(subtile_cnt):
                    # TMEM → RMEM
                    tTR_tAcc_mn = tTR_tAcc[(None, None, None, subtile_idx)]
                    if cutlass.const_expr(self.scenario == "2Dx2D"):
                        if k_tile_cnt > 0:
                            cute.copy(tiled_copy_t2r, tTR_tAcc_mn, tTR_rAcc)
                    else:
                        cute.copy(tiled_copy_t2r, tTR_tAcc_mn, tTR_rAcc)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1307-1307 / 第 1307-1307 行

~~~~python
                    # Convert to output dtype
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1308-1315 / 第 1308-1315 行

~~~~python
                    acc_vec = cute.zeros_like(tiled_copy_r2s.retile(tTR_rAcc))
                    if cutlass.const_expr(self.scenario == "2Dx2D"):
                        if k_tile_cnt > 0:
                            acc_vec = tiled_copy_r2s.retile(tTR_rAcc).load()
                    else:
                        acc_vec = tiled_copy_r2s.retile(tTR_rAcc).load()
                    acc_vec = acc_vec.to(self.c_dtype)
                    tRS_rC.store(acc_vec)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1317-1317 / 第 1317-1317 行

~~~~python
                    # RMEM → SMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1318-1323 / 第 1318-1323 行

~~~~python
                    c_buffer = (num_prev_subtiles + subtile_idx) % self.num_c_stage
                    cute.copy(
                        tiled_copy_r2s, tRS_rC, tRS_sC[(None, None, None, c_buffer)]
                    )
                    cute.arch.fence_proxy("async.shared", space="cta")
                    epilog_sync_barrier.arrive_and_wait()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1325-1325 / 第 1325-1325 行

~~~~python
                    # SMEM → GMEM (TMA store or TMA reduce)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1326-1335 / 第 1326-1335 行

~~~~python
                    if warp_idx == self.epilogue_warp_id[0]:
                        cute.copy(
                            tma_atom_c,
                            bSG_sC[(None, c_buffer)],
                            bSG_gC[(None, subtile_idx)],
                            tma_desc_ptr=desc_ptr_c,
                        )
                        c_pipeline.producer_commit()
                        c_pipeline.producer_acquire()
                    epilog_sync_barrier.arrive_and_wait()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1337-1337 / 第 1337-1337 行

~~~~python
                # Release accumulator buffer
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1338-1341 / 第 1338-1341 行

~~~~python
                if k_tile_cnt > 0:
                    acc_pipeline.consumer_release(acc_consumer_state)
                    acc_consumer_state.advance()
                num_tiles_executed += cutlass.Int32(1)
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1343-1343 / 第 1343-1343 行

~~~~python
                # Read next work_tile_info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1344-1354 / 第 1344-1354 行

~~~~python
                sched_pipeline.consumer_wait(sched_consumer_state)
                rmem = cute.make_rmem_tensor((4,), cutlass.Int32)
                cute.copy(
                    sched_copy_atom,
                    sched_buf_tensor[(None, sched_consumer_state.index)],
                    rmem,
                )
                work_tile_info = MoEWorkTileInfo.from_rmem_tensor(rmem)
                cute.arch.fence_acq_rel_cta()
                sched_pipeline.consumer_release(sched_consumer_state)
                sched_consumer_state.advance()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1356-1356 / 第 1356-1356 行

~~~~python
            # Wait for C store complete
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1357-1357 / 第 1357-1357 行

~~~~python
            c_pipeline.producer_tail()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1359-1359 / 第 1359-1359 行

~~~~python
            # Free TMEM
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1360-1361 / 第 1360-1361 行

~~~~python
            tmem.relinquish_alloc_permit()
            tmem.free(tmem_ptr)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1364-1366 / 第 1364-1366 行

~~~~python
# =============================================================================
# Host Validation
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1368-1373 / 第 1368-1373 行

~~~~python
from dataclasses import dataclass, field
import re

import numpy as np
import torch
import cutlass.torch as cutlass_torch
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 1376-1386 / 第 1376-1386 行

~~~~python
def torch_version_lt(major: int, minor: int) -> bool:
    """Best-effort torch version check that tolerates local build suffixes."""
    match = re.match(r"^\s*(\d+)\.(\d+)", torch.__version__)
    if match is None:
        print(
            "WARNING: failed to parse torch.__version__, "
            "falling back to torch._grouped_mm host reference."
        )
        return True
    version = (int(match.group(1)), int(match.group(2)))
    return version < (major, minor)
~~~~

**EN**: Defines `torch_version_lt`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 定义 `torch_version_lt`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1389-1408 / 第 1389-1408 行

~~~~python
@dataclass
class ProblemDesc:
    tokens: int
    experts: int
    top_k_select: int
    balance_route: bool
    hidden: int
    intermediate: int
    scenario: Literal["2Dx3D", "2Dx2D"]
    ab_dtype: torch.dtype
    out_dtype: torch.dtype
    acc_dtype: torch.dtype
    grad_accumulate: bool = False
    # GEMM-domain layout control (which axis is stride-1)
    # A (M, K): "k_major" (default) or "m_major"
    # B (N, K): "n_major" (default) or "k_major"
    # C (M, N): "n_major" (default) or "m_major"
    a_layout: Literal["k_major", "m_major"] = "k_major"
    b_layout: Literal["k_major", "n_major"] = "n_major"
    c_layout: Literal["m_major", "n_major"] = "n_major"
~~~~

**EN**: Defines `ProblemDesc`, a reusable Python class that packages configuration and behavior for this example. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `ProblemDesc`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1410-1418 / 第 1410-1418 行

~~~~python
    def __str__(self) -> str:
        d = lambda t: str(t).split(".")[-1]
        route = "balanced" if self.balance_route else "random"
        return (
            f"ProblemDesc: {self.scenario} | tokens={self.tokens} experts={self.experts} "
            f"top_k={self.top_k_select} route={route} | hidden={self.hidden} intermediate={self.intermediate} | "
            f"{d(self.ab_dtype)}->{d(self.out_dtype)}(acc={d(self.acc_dtype)}) grad_acc={self.grad_accumulate} | "
            f"layout: A={self.a_layout} B={self.b_layout} C={self.c_layout}"
        )
~~~~

**EN**: Defines `__str__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__str__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 1421-1427 / 第 1421-1427 行

~~~~python
@dataclass
class ImplDesc:
    mma_tiler_mnk: Tuple[int, int, int]
    cluster_shape_mnk: Tuple[int, int, int]
    use_2cta_instrs: bool
    static_expert_cnt: Optional[int] = None
    separate_tensormap_init: bool = True
~~~~

**EN**: Defines `ImplDesc`, a reusable Python class that packages configuration and behavior for this example. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `ImplDesc`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1429-1438 / 第 1429-1438 行

~~~~python
    def __str__(self) -> str:
        tile = ",".join(map(str, self.mma_tiler_mnk))
        cluster = ",".join(map(str, self.cluster_shape_mnk))
        static_e = (
            self.static_expert_cnt if self.static_expert_cnt is not None else "dynamic"
        )
        return (
            f"ImplDesc: tile={tile} cluster={cluster} 2cta={self.use_2cta_instrs} | "
            f"static_E={static_e} sep_tmap={self.separate_tensormap_init}"
        )
~~~~

**EN**: Defines `__str__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__str__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 1441-1447 / 第 1441-1447 行

~~~~python
@dataclass
class MiscDesc:
    perf_run: bool = False
    perf_e2e: bool = False
    compare_with_bmm: bool = False
    compare_with_sol: bool = False
    no_torch_210: bool = field(init=False)
~~~~

**EN**: Defines `MiscDesc`, a reusable Python class that packages configuration and behavior for this example. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `MiscDesc`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1449-1456 / 第 1449-1456 行

~~~~python
    def __post_init__(self):
        self.no_torch_210 = torch_version_lt(2, 10)
        if self.perf_e2e and not self.perf_run:
            raise ValueError("--perf_e2e requires --perf_run to be enabled.")
        if self.perf_e2e and self.compare_with_sol:
            raise ValueError(
                "--perf_e2e and --compare_with_sol are mutually exclusive."
            )
~~~~

**EN**: Defines `__post_init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `__post_init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1458-1463 / 第 1458-1463 行

~~~~python
    def __str__(self) -> str:
        ref = "bmm" if self.compare_with_bmm else "grouped_mm"
        return (
            f"MiscDesc: perf={self.perf_run} perf_e2e={self.perf_e2e} "
            f"ref={ref} sol={self.compare_with_sol} no_torch_210={self.no_torch_210}"
        )
~~~~

**EN**: Defines `__str__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__str__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 1466-1470 / 第 1466-1470 行

~~~~python
def l2_flush(size_mb: int = 400) -> None:
    """Best-effort L2 flush by touching a large temporary tensor."""
    num_bytes = size_mb * 1024 * 1024
    flush_buf = torch.randint(0, 256, (num_bytes,), dtype=torch.uint8, device="cuda")
    del flush_buf
~~~~

**EN**: Defines `l2_flush`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `l2_flush`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1473-1477 / 第 1473-1477 行

~~~~python
class GroupedGemmTester:
    def __init__(self, problem: ProblemDesc, impl: ImplDesc, misc: MiscDesc):
        self.problem = problem
        self.impl = impl
        self.misc = misc
~~~~

**EN**: Defines `GroupedGemmTester`, a reusable Python class that packages configuration and behavior for this example. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `GroupedGemmTester`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 1479-1482 / 第 1479-1482 行

~~~~python
        self.tokens_after_repeat = problem.tokens * problem.top_k_select
        self.expert_cnt = problem.experts
        self.hidden = problem.hidden
        self.intermediate = problem.intermediate
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1484-1489 / 第 1484-1489 行

~~~~python
        self.A_tensor: torch.Tensor = None
        self.B_tensor: torch.Tensor = None
        self.C_tensor: torch.Tensor = None
        self.C_ref_tensor: torch.Tensor = None
        self.offs_tensor: torch.Tensor = None
        self.workspace_tensor: torch.Tensor = None
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1491-1491 / 第 1491-1491 行

~~~~python
        # This should be a common func
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1492-1496 / 第 1492-1496 行

~~~~python
        self.temp_type_mapping = {
            torch.float32: cutlass.Float32,
            torch.bfloat16: cutlass.BFloat16,
            torch.float16: cutlass.Float16,
        }
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1498-1504 / 第 1498-1504 行

~~~~python
    def _generate_offs(self) -> torch.Tensor:
        """Generate group-end offsets.

        Some experts may receive 0 tokens (valid in real MoE routing).
        """
        total = self.tokens_after_repeat
        expert_cnt = self.expert_cnt
~~~~

**EN**: Defines `_generate_offs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `_generate_offs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1506-1527 / 第 1506-1527 行

~~~~python
        if self.problem.balance_route:
            base = total // expert_cnt
            remainder = total % expert_cnt
            sizes = [base + (1 if i < remainder else 0) for i in range(expert_cnt)]
        else:
            proportions = np.random.dirichlet([0.5] * expert_cnt)
            raw = np.floor(proportions * total).astype(int)
            deficit = total - raw.sum()
            while deficit > 0:
                idx = int(np.argmin(raw / (proportions * total + 1e-12)))
                raw[idx] += 1
                deficit -= 1
            while deficit < 0:
                ratios = np.where(
                    raw > 0,
                    raw / (proportions * total + 1e-12),
                    -np.inf,
                )
                idx = int(np.argmax(ratios))
                raw[idx] -= 1
                deficit += 1
            sizes = raw.tolist()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1529-1529 / 第 1529-1529 行

~~~~python
        assert sum(sizes) == total
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1531-1536 / 第 1531-1536 行

~~~~python
        cum = 0
        offsets = []
        for s in sizes:
            cum += s
            offsets.append(cum)
        return torch.tensor(offsets, dtype=torch.int32, device="cuda")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 1538-1544 / 第 1538-1544 行

~~~~python
    def _generate_tensor(self, shape: Tuple) -> torch.Tensor:
        if self.misc.perf_run:
            return torch.randn(shape, dtype=self.problem.ab_dtype, device="cuda")
        else:
            return torch.randint(-1, 2, shape, device="cuda", dtype=torch.int8).to(
                self.problem.ab_dtype
            )
~~~~

**EN**: Defines `_generate_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_generate_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 1546-1547 / 第 1546-1547 行

~~~~python
    def _get_stream(self) -> cuda.CUstream:
        return cuda.CUstream(torch.cuda.current_stream().cuda_stream)
~~~~

**EN**: Defines `_get_stream`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_get_stream`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 1549-1550 / 第 1549-1550 行

~~~~python
    def generate_inputs(self) -> None:
        self.offs_tensor = self._generate_offs()
~~~~

**EN**: Defines `generate_inputs`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `generate_inputs`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 1552-1555 / 第 1552-1555 行

~~~~python
        tokens = self.tokens_after_repeat
        hidden = self.hidden
        intermediate = self.intermediate
        expert_cnt = self.expert_cnt
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1557-1559 / 第 1557-1559 行

~~~~python
        if self.problem.scenario == "2Dx3D":
            # PyTorch shape: A (tokens, hidden), B (expert_cnt, hidden, intermediate), C (tokens, intermediate)
            # GEMM domain:   A (M=tokens, K=hidden), B (N=intermediate, K=hidden), C (M=tokens, N=intermediate)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1561-1561 / 第 1561-1561 行

~~~~python
            # GEMM A: k_major → K(hidden) stride-1;  m_major → M(tokens) stride-1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1562-1565 / 第 1562-1565 行

~~~~python
            if self.problem.a_layout == "k_major":
                self.A_tensor = self._generate_tensor((tokens, hidden))
            else:
                self.A_tensor = self._generate_tensor((hidden, tokens)).T
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1567-1567 / 第 1567-1567 行

~~~~python
            # GEMM B: n_major → N(intermediate) stride-1;  k_major → K(hidden) stride-1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1568-1575 / 第 1568-1575 行

~~~~python
            if self.problem.b_layout == "n_major":
                self.B_tensor = self._generate_tensor(
                    (expert_cnt, hidden, intermediate)
                )
            else:
                self.B_tensor = self._generate_tensor(
                    (expert_cnt, intermediate, hidden)
                ).transpose(1, 2)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1577-1577 / 第 1577-1577 行

~~~~python
            # GEMM C: n_major → N(intermediate) stride-1;  m_major → M(tokens) stride-1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1578-1591 / 第 1578-1591 行

~~~~python
            if self.problem.c_layout == "n_major":
                self.C_tensor = torch.full(
                    (tokens, intermediate),
                    -1,
                    dtype=self.problem.out_dtype,
                    device="cuda",
                )
            else:
                self.C_tensor = torch.full(
                    (intermediate, tokens),
                    -1,
                    dtype=self.problem.out_dtype,
                    device="cuda",
                ).T
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1593-1596 / 第 1593-1596 行

~~~~python
        elif self.problem.scenario == "2Dx2D":
            # PyTorch shape: mat_a (hidden, tokens), mat_b (tokens, intermediate), out (expert_cnt, hidden, intermediate)
            # out matches weight shape (expert_cnt, hidden, intermediate) for weight gradient
            # GEMM domain:   A (M=hidden, K=tokens), B (N=intermediate, K=tokens), C (M=hidden, N=intermediate)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1598-1598 / 第 1598-1598 行

~~~~python
            # GEMM A: k_major → K(tokens) stride-1;  m_major → M(hidden) stride-1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1599-1602 / 第 1599-1602 行

~~~~python
            if self.problem.a_layout == "k_major":
                self.A_tensor = self._generate_tensor((hidden, tokens))
            else:
                self.A_tensor = self._generate_tensor((tokens, hidden)).T
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1604-1604 / 第 1604-1604 行

~~~~python
            # GEMM B: n_major → N(intermediate) stride-1;  k_major → K(tokens) stride-1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1605-1608 / 第 1605-1608 行

~~~~python
            if self.problem.b_layout == "n_major":
                self.B_tensor = self._generate_tensor((tokens, intermediate))
            else:
                self.B_tensor = self._generate_tensor((intermediate, tokens)).T
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1610-1610 / 第 1610-1610 行

~~~~python
            # GEMM C: n_major → N(intermediate) stride-1;  m_major → M(hidden) stride-1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1611-1628 / 第 1611-1628 行

~~~~python
            if self.problem.c_layout == "n_major":
                self.C_tensor = torch.full(
                    (expert_cnt, hidden, intermediate),
                    -1,
                    dtype=self.problem.out_dtype,
                    device="cuda",
                )
            else:
                self.C_tensor = torch.full(
                    (expert_cnt, intermediate, hidden),
                    -1,
                    dtype=self.problem.out_dtype,
                    device="cuda",
                ).transpose(1, 2)
            if self.problem.grad_accumulate:
                self.C_tensor *= 0
        else:
            raise ValueError(f"Unknown scenario: {self.problem.scenario}")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1630-1636 / 第 1630-1636 行

~~~~python
    def compute_reference(self) -> None:
        if self.misc.perf_run:
            return
        if self.misc.compare_with_bmm:
            self._compute_reference_bmm()
        else:
            self._compute_reference_grouped_mm()
~~~~

**EN**: Defines `compute_reference`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `compute_reference`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1638-1649 / 第 1638-1649 行

~~~~python
    def _compute_reference_grouped_mm(self) -> None:
        grouped_mm_op = (
            torch._grouped_mm
            if self.misc.no_torch_210
            else torch.nn.functional.grouped_mm
        )
        self.C_ref_tensor = grouped_mm_op(
            self.A_tensor,
            self.B_tensor,
            offs=self.offs_tensor,
            out_dtype=self.problem.out_dtype,
        )
~~~~

**EN**: Defines `_compute_reference_grouped_mm`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `_compute_reference_grouped_mm`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1651-1655 / 第 1651-1655 行

~~~~python
    def _compute_reference_bmm(self) -> None:
        """Manual per-expert torch.mm loop as reference (avoids grouped_mm bugs on small cases)."""
        # Preallocate the full reference output to avoid keeping both the per-expert
        # results list and the final cat/stack result alive at the same time.
        self.C_ref_tensor = torch.empty_like(self.C_tensor)
~~~~

**EN**: Defines `_compute_reference_bmm`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `_compute_reference_bmm`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1657-1670 / 第 1657-1670 行

~~~~python
        prev = 0
        for i in range(self.expert_cnt):
            cur = self.offs_tensor[i].item()
            if self.problem.scenario == "2Dx3D":
                # A (tokens, hidden), B (E, hidden, intermediate) → C_i (tokens_i, intermediate)
                a_slice = self.A_tensor[prev:cur, :]
                b_slice = self.B_tensor[i]
                self.C_ref_tensor[prev:cur, :].copy_(torch.mm(a_slice, b_slice))
            else:  # 2Dx2D
                # A (hidden, tokens), B (tokens, intermediate) → C_i (hidden, intermediate)
                a_slice = self.A_tensor[:, prev:cur]
                b_slice = self.B_tensor[prev:cur, :]
                self.C_ref_tensor[i, :, :].copy_(torch.mm(a_slice, b_slice))
            prev = cur
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1672-1684 / 第 1672-1684 行

~~~~python
    def create_kernel(self) -> GroupedGemmKernel:
        return GroupedGemmKernel(
            scenario=self.problem.scenario,
            out_dtype=self.temp_type_mapping[self.problem.out_dtype],
            accumulate_on_output=self.problem.grad_accumulate
            and self.problem.scenario == "2Dx2D",
            separate_tensormap_init=self.impl.separate_tensormap_init,
            fixed_expert_cnt=self.impl.static_expert_cnt,
            acc_dtype=self.temp_type_mapping[self.problem.acc_dtype],
            mma_tiler_mnk=self.impl.mma_tiler_mnk,
            cluster_shape_mnk=self.impl.cluster_shape_mnk,
            use_2cta_instrs=self.impl.use_2cta_instrs,
        )
~~~~

**EN**: Defines `create_kernel`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `create_kernel`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 1686-1695 / 第 1686-1695 行

~~~~python
    def run_kernel(self, kernel: GroupedGemmKernel) -> Optional[float]:
        """Run our CuTe kernel.

        Returns:
            Average kernel time in ms when perf_e2e is enabled, None otherwise.
        """
        workspace_size = kernel.get_workspace_size(self.expert_cnt)
        self.workspace_tensor = torch.full(
            (workspace_size,), 255, dtype=torch.uint8, device="cuda"
        )
~~~~

**EN**: Defines `run_kernel`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `run_kernel`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1697-1697 / 第 1697-1697 行

~~~~python
        torch.cuda.synchronize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1699-1700 / 第 1699-1700 行

~~~~python
        ab_cutlass_dtype = self.temp_type_mapping[self.problem.ab_dtype]
        out_cutlass_dtype = self.temp_type_mapping[self.problem.out_dtype]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1702-1723 / 第 1702-1723 行

~~~~python
        a_cute, self.A_tensor = cutlass_torch.cute_tensor_like(
            self.A_tensor, ab_cutlass_dtype, is_dynamic_layout=True, assumed_align=16
        )
        b_cute, self.B_tensor = cutlass_torch.cute_tensor_like(
            self.B_tensor, ab_cutlass_dtype, is_dynamic_layout=True, assumed_align=16
        )
        c_cute, self.C_tensor = cutlass_torch.cute_tensor_like(
            self.C_tensor, out_cutlass_dtype, is_dynamic_layout=True, assumed_align=16
        )
        is_dynamic_expert_cnt = self.impl.static_expert_cnt is None
        offs_cute, self.offs_tensor = cutlass_torch.cute_tensor_like(
            self.offs_tensor,
            cutlass.Int32,
            is_dynamic_layout=is_dynamic_expert_cnt,
            assumed_align=16,
        )
        workspace_cute, self.workspace_tensor = cutlass_torch.cute_tensor_like(
            self.workspace_tensor,
            cutlass.Uint8,
            is_dynamic_layout=is_dynamic_expert_cnt,
            assumed_align=128,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1725-1725 / 第 1725-1725 行

~~~~python
        # Query max active clusters from hardware
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1726-1733 / 第 1726-1733 行

~~~~python
        cluster_size = self.impl.cluster_shape_mnk[0] * self.impl.cluster_shape_mnk[1]
        max_active_clusters = utils.HardwareInfo().get_max_active_clusters(cluster_size)
        print(f"A_tensor: {tuple(self.A_tensor.shape)}:{self.A_tensor.stride()}")
        print(f"B_tensor: {tuple(self.B_tensor.shape)}:{self.B_tensor.stride()}")
        print(
            f"offset_tensor: {tuple(self.offs_tensor.shape)}:{self.offs_tensor.stride()}"
        )
        print(f"C_tensor: {tuple(self.C_tensor.shape)}:{self.C_tensor.stride()}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1735-1735 / 第 1735-1735 行

~~~~python
        stream = self._get_stream()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1737-1748 / 第 1737-1748 行

~~~~python
        if self.misc.perf_e2e:
            compiled = cute.compile(
                kernel,
                a_cute,
                b_cute,
                c_cute,
                offs_cute,
                None,  # bias
                workspace_cute,
                max_active_clusters,
                stream,
            )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1750-1751 / 第 1750-1751 行

~~~~python
            warmup_iters = 4
            timed_iters = 4
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1753-1764 / 第 1753-1764 行

~~~~python
            for _ in range(warmup_iters):
                l2_flush()
                compiled(
                    a_cute,
                    b_cute,
                    c_cute,
                    offs_cute,
                    None,  # bias
                    workspace_cute,
                    stream,
                )
            torch.cuda.synchronize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1766-1784 / 第 1766-1784 行

~~~~python
            times = []
            for _ in range(timed_iters):
                l2_flush()
                torch.cuda.synchronize()
                start_evt = torch.cuda.Event(enable_timing=True)
                end_evt = torch.cuda.Event(enable_timing=True)
                start_evt.record()
                compiled(
                    a_cute,
                    b_cute,
                    c_cute,
                    offs_cute,
                    None,  # bias
                    workspace_cute,
                    stream,
                )
                end_evt.record()
                torch.cuda.synchronize()
                times.append(start_evt.elapsed_time(end_evt))
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1786-1803 / 第 1786-1803 行

~~~~python
            avg_ms = sum(times) / len(times)
            print(f"[perf_e2e] Individual times (ms): {[f'{t:.4f}' for t in times]}")
            print(f"[perf_e2e] Average kernel time: {avg_ms:.4f} ms")
            return avg_ms
        else:
            l2_flush()
            kernel(
                a_cute,
                b_cute,
                c_cute,
                offs_cute,
                None,  # bias
                workspace_cute,
                max_active_clusters,
                stream,
            )
            torch.cuda.synchronize()
            return None
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1805-1809 / 第 1805-1809 行

~~~~python
    def validate(self) -> None:
        if not self.misc.perf_run:
            assert torch.equal(self.C_tensor, self.C_ref_tensor), (
                "Validation failed: C_tensor != C_ref_tensor"
            )
~~~~

**EN**: Defines `validate`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `validate`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1811-1817 / 第 1811-1817 行

~~~~python
    def run_sol_comparison(self) -> None:
        """Run a dense batched GEMM as Speed-of-Light reference.

        Reuses the same tensor memory from the grouped run by
        view/reshape/permute -- zero GPU allocation.
        """
        import sys, os
~~~~

**EN**: Defines `run_sol_comparison`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `run_sol_comparison`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1819-1823 / 第 1819-1823 行

~~~~python
        _examples_root = os.path.abspath(
            os.path.join(os.path.dirname(__file__), "..", "..", "..")
        )
        if _examples_root not in sys.path:
            sys.path.insert(0, _examples_root)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1825-1825 / 第 1825-1825 行

~~~~python
        from blackwell.kernel.dense_gemm.dense_gemm_persistent import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 1826-1827 / 第 1826-1827 行

~~~~python
            PersistentDenseGemmKernel,
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1829-1836 / 第 1829-1836 行

~~~~python
        tokens = self.tokens_after_repeat
        experts = self.expert_cnt
        assert tokens % experts == 0, (
            f"compare_with_sol requires tokens*top_k ({tokens}) "
            f"evenly divisible by experts ({experts}) so every group "
            f"has exactly the same size"
        )
        tpe = tokens // experts
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1838-1841 / 第 1838-1841 行

~~~~python
        if self.problem.scenario == "2Dx3D":
            M, N, K, L = tpe, self.intermediate, self.hidden, experts
        else:  # 2Dx2D
            M, N, K, L = self.hidden, self.intermediate, tpe, experts
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 1843-1845 / 第 1843-1845 行

~~~~python
        # Reshape into GEMM-domain batch-last: A(M,K,L), B(N,K,L), C(M,N,L).
        # Data values are irrelevant (perf only) — just need correct shape
        # and stride pattern so the dense kernel sees the right major mode.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1846-1851 / 第 1846-1851 行

~~~~python
        if self.problem.a_layout == "k_major":
            a_sol = self.A_tensor.contiguous().view(L, M, K).permute(1, 2, 0)
            leading_dim_a = 1
        else:
            a_sol = self.A_tensor.contiguous().view(L, K, M).permute(2, 1, 0)
            leading_dim_a = 0
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1853-1858 / 第 1853-1858 行

~~~~python
        if self.problem.b_layout == "n_major":
            b_sol = self.B_tensor.contiguous().view(L, K, N).permute(2, 1, 0)
            leading_dim_b = 0
        else:
            b_sol = self.B_tensor.contiguous().view(L, N, K).permute(1, 2, 0)
            leading_dim_b = 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1860-1865 / 第 1860-1865 行

~~~~python
        if self.problem.c_layout == "n_major":
            c_sol = self.C_tensor.contiguous().view(L, M, N).permute(1, 2, 0)
            leading_dim_c = 1
        else:
            c_sol = self.C_tensor.contiguous().view(L, N, M).permute(2, 1, 0)
            leading_dim_c = 0
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1867-1867 / 第 1867-1867 行

~~~~python
        from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 1869-1877 / 第 1869-1877 行

~~~~python
        a_cute_sol = from_dlpack(a_sol, assumed_align=16).mark_layout_dynamic(
            leading_dim=leading_dim_a
        )
        b_cute_sol = from_dlpack(b_sol, assumed_align=16).mark_layout_dynamic(
            leading_dim=leading_dim_b
        )
        c_cute_sol = from_dlpack(c_sol, assumed_align=16).mark_layout_dynamic(
            leading_dim=leading_dim_c
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1879-1882 / 第 1879-1882 行

~~~~python
        mma_tiler_mn = self.impl.mma_tiler_mnk[:2]
        cluster_shape_mn = self.impl.cluster_shape_mnk[:2]
        cluster_size = cluster_shape_mn[0] * cluster_shape_mn[1]
        max_active_clusters = utils.HardwareInfo().get_max_active_clusters(cluster_size)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1884-1890 / 第 1884-1890 行

~~~~python
        sol_kernel = PersistentDenseGemmKernel(
            acc_dtype=self.temp_type_mapping[self.problem.acc_dtype],
            use_2cta_instrs=self.impl.use_2cta_instrs,
            mma_tiler_mn=mma_tiler_mn,
            cluster_shape_mn=cluster_shape_mn,
            use_tma_store=True,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1892-1895 / 第 1892-1895 行

~~~~python
        print(f"\n[SOL] Dense BMM: M={M} N={N} K={K} L={L}")
        print(f"[SOL] a_sol: {tuple(a_sol.shape)}:{a_sol.stride()}")
        print(f"[SOL] b_sol: {tuple(b_sol.shape)}:{b_sol.stride()}")
        print(f"[SOL] c_sol: {tuple(c_sol.shape)}:{c_sol.stride()}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1897-1905 / 第 1897-1905 行

~~~~python
        l2_flush()
        sol_kernel(
            a_cute_sol,
            b_cute_sol,
            c_cute_sol,
            max_active_clusters,
            self._get_stream(),
        )
        torch.cuda.synchronize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1907-1908 / 第 1907-1908 行

~~~~python
    def run(self) -> None:
        from torch.profiler import profile, ProfilerActivity
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1910-1914 / 第 1910-1914 行

~~~~python
        print(self.problem)
        print(self.impl)
        print(self.misc)
        self.generate_inputs()
        kernel = self.create_kernel()
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1916-1930 / 第 1916-1930 行

~~~~python
        if self.misc.perf_e2e:
            self.run_kernel(kernel)
        else:
            with profile(
                activities=[ProfilerActivity.CUDA], record_shapes=True
            ) as prof:
                self.compute_reference()
                self.run_kernel(kernel)
                if (
                    self.misc.compare_with_sol
                    and self.misc.perf_run
                    and self.problem.balance_route
                ):
                    self.run_sol_comparison()
            print(prof.key_averages().table(sort_by="cuda_time_total", row_limit=20))
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1932-1932 / 第 1932-1932 行

~~~~python
        self.validate()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1935-1936 / 第 1935-1936 行

~~~~python
if __name__ == "__main__":
    import argparse
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1938-1939 / 第 1938-1939 行

~~~~python
    def parse_dtype(s: str) -> torch.dtype:
        return getattr(torch, s)
~~~~

**EN**: Defines `parse_dtype`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `parse_dtype`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 1941-1942 / 第 1941-1942 行

~~~~python
    def parse_tuple(s: str) -> Tuple[int, ...]:
        return tuple(int(x) for x in s.split(","))
~~~~

**EN**: Defines `parse_tuple`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `parse_tuple`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 1944-1971 / 第 1944-1971 行

~~~~python
    parser = argparse.ArgumentParser()
    parser.add_argument("--tokens", type=int, default=128)
    parser.add_argument("--experts", type=int, default=128)
    parser.add_argument("--top_k_select", type=int, default=8)
    parser.add_argument("--balance_route", action="store_true", default=False)
    parser.add_argument("--hidden", type=int, default=2048)
    parser.add_argument("--intermediate", type=int, default=7168)
    parser.add_argument(
        "--scenario", type=str, default="2Dx3D", choices=["2Dx3D", "2Dx2D"]
    )
    parser.add_argument("--ab_dtype", type=str, default="bfloat16")
    parser.add_argument("--out_dtype", type=str, default="bfloat16")
    parser.add_argument("--acc_dtype", type=str, default="float32")
    parser.add_argument("--grad_accumulate", action="store_true", default=False)
    parser.add_argument(
        "--a_layout", type=str, default="k_major", choices=["k_major", "m_major"]
    )
    parser.add_argument(
        "--b_layout", type=str, default="n_major", choices=["k_major", "n_major"]
    )
    parser.add_argument(
        "--c_layout", type=str, default="n_major", choices=["m_major", "n_major"]
    )
    parser.add_argument("--mma_tiler_mnk", type=str, default="128,128,64")
    parser.add_argument("--cluster_shape_mnk", type=str, default="1,1,1")
    parser.add_argument("--use_2cta_instrs", action="store_true", default=False)
    parser.add_argument("--static_expert_cnt", type=int, default=None)
    parser.add_argument("--separate_tensormap_init", action="store_true", default=False)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1972-1976 / 第 1972-1976 行

~~~~python
    parser.add_argument("--perf_run", action="store_true", default=False)
    parser.add_argument("--perf_e2e", action="store_true", default=False)
    parser.add_argument("--compare_with_bmm", action="store_true", default=False)
    parser.add_argument("--compare_with_sol", action="store_true", default=False)
    args = parser.parse_args()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 1978-2005 / 第 1978-2005 行

~~~~python
    problem = ProblemDesc(
        tokens=args.tokens,
        experts=args.experts,
        top_k_select=args.top_k_select,
        balance_route=args.balance_route,
        hidden=args.hidden,
        intermediate=args.intermediate,
        scenario=args.scenario,
        ab_dtype=parse_dtype(args.ab_dtype),
        out_dtype=parse_dtype(args.out_dtype),
        acc_dtype=parse_dtype(args.acc_dtype),
        grad_accumulate=args.grad_accumulate,
        a_layout=args.a_layout,
        b_layout=args.b_layout,
        c_layout=args.c_layout,
    )
    if not args.separate_tensormap_init:
        print(
            "Change separate_tensormap_init to True as current the fused version not implmented yet."
        )
        args.separate_tensormap_init = True
    impl = ImplDesc(
        mma_tiler_mnk=parse_tuple(args.mma_tiler_mnk),
        cluster_shape_mnk=parse_tuple(args.cluster_shape_mnk),
        use_2cta_instrs=args.use_2cta_instrs,
        static_expert_cnt=args.static_expert_cnt,
        separate_tensormap_init=args.separate_tensormap_init,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 2007-2015 / 第 2007-2015 行

~~~~python
    misc = MiscDesc(
        perf_run=args.perf_run,
        perf_e2e=args.perf_e2e,
        compare_with_bmm=args.compare_with_bmm,
        compare_with_sol=args.compare_with_sol,
    )
    if misc.no_torch_210:
        misc.compare_with_bmm = True
        print("Override to set --compare_with_bmm to avoid possible torch crash.")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 2017-2019 / 第 2017-2019 行

~~~~python
    tester = GroupedGemmTester(problem, impl, misc)
    tester.run()
    print("PASS")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块

## Dependencies / 依赖项

- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `sys` — used by this example / 供该示例使用
- `typing.Optional` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Literal` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Type` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Union` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.typing.Pointer` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.nvgpu.cpasync` — provides asynchronous copy and TMA primitives / 提供异步拷贝与 TMA 原语
- `cutlass.cute.nvgpu.tcgen05` — provides Tensor Core/TMEM helpers for Blackwell-style kernels / 提供面向 Blackwell 风格内核的 Tensor Core/TMEM 辅助工具
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.pipeline` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.pipeline.pipeline_init_arrive` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.pipeline.pipeline_init_wait` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `blackwell.kernel.moe.moe_utils.MoEGroupedGemmTensormapConstructor` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_persistent_scheduler.MoEStaticSchedulerParams` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_persistent_scheduler.MoEStaticPersistentTileScheduler` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_persistent_scheduler.MoEWorkTileInfo` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_sched_extension.GroupedMmSchedExtension` — used by this example / 供该示例使用
- `cutlass.utils.gemm.sm100.transform_partitioned_tensor_layout` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.gemm.sm100.epilogue_tmem_copy_and_partition` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.gemm.sm100.epilogue_smem_copy_and_partition` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `dataclasses.dataclass` — used by this example / 供该示例使用
- `dataclasses.field` — used by this example / 供该示例使用
- `re` — used by this example / 供该示例使用
- `numpy` — supplies host-side array utilities / 提供宿主端数组工具
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `blackwell.kernel.dense_gemm.dense_gemm_persistent.PersistentDenseGemmKernel` — used by this example / 供该示例使用
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `torch.profiler.profile` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `torch.profiler.ProfilerActivity` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
