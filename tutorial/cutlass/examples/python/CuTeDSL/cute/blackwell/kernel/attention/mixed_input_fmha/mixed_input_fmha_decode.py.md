# mixed_input_fmha_decode.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/attention/mixed_input_fmha/mixed_input_fmha_decode.py`  
**Purpose / 用途**: Kernel example implementing mixed input fmha decode with CuTeDSL. / 这是一个使用 CuTeDSL 实现 mixed input fmha decode 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-49 / 第 29-49 行

~~~~python
import argparse
import math
from typing import Type, Tuple
from functools import partial

import torch
from torch.nn.functional import scaled_dot_product_attention
from torch.nn.attention import SDPBackend, sdpa_kernel

import cuda.bindings.driver as cuda

import cutlass
import cutlass.cute as cute
from cutlass.cute.nvgpu import tcgen05, OperandMajorMode
import cutlass.utils as utils
import cutlass.pipeline as pipeline
import cutlass.torch as cutlass_torch
import cutlass.utils.blackwell_helpers as sm100_utils
import cutlass.cute.testing as testing
from cutlass.cute.runtime import from_dlpack
from cutlass.cute.typing import *
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 51-51 / 第 51-51 行

~~~~python
# Kernel invariants
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 52-57 / 第 52-57 行

~~~~python
mma_modes = (0, 1, 2)
mma_dice = (None, None, None)  # (MMA, #MMA_M, #MMA_K)
cpy_dice = (None,) + mma_dice  # (CPY, #CPY_MMA, #CPY_M, #CPY_K)
warp_threads = 32
warpgroup_warps = 4
warpgroup_threads = 128
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 59-59 / 第 59-59 行

~~~~python
# Math helpers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 60-68 / 第 60-68 行

~~~~python
log2_e = math.log2(math.e)  # change exponential base
use_tensor_ssa_math = False  # experimental
fadd2 = cute.arch.add_packed_f32x2
fmul2 = cute.arch.mul_packed_f32x2
ffma2 = cute.arch.fma_packed_f32x2
exp2 = partial(cute.math.exp2, fastmath=True)
warp_fmax = partial(cute.arch.warp_redux_sync, kind="fmax", nan=True)
smem_fmax = partial(cute.arch.atomic_fmax, sem="relaxed", scope="cta")
gmem_fmax = partial(cute.arch.atomic_fmax, sem="relaxed", scope="gpu")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 71-83 / 第 71-83 行

~~~~python
class MixedInputFusedMultiHeadAttentionDecode:
    def __init__(
        self,
        headdim,
        block_scaledim,  # headdim per scale factor; scale factor shape is (batches, heads_k, seqlen, headdim / block_scaledim)
        grouped_head_tile,  # GQA packing tile size, can be less than group size
        convert_warpgroups=1,  # Multiple warpgroups striding on convert stages
    ):
        self.headdim = headdim
        self.grouped_head_tile = grouped_head_tile
        self.block_scaledim = block_scaledim
        self.scaledim = headdim // block_scaledim
        self.convert_warpgroups = convert_warpgroups
~~~~

**EN**: Defines `MixedInputFusedMultiHeadAttentionDecode`, a reusable Python class that packages configuration and behavior for this example. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `MixedInputFusedMultiHeadAttentionDecode`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 85-86 / 第 85-86 行

~~~~python
        assert headdim % block_scaledim == 0
        assert grouped_head_tile % 8 == 0 and 0 < grouped_head_tile <= 32
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 88-88 / 第 88-88 行

~~~~python
        warpgroup_id = 0
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 90-91 / 第 90-91 行

~~~~python
        self.softmax_warpgroup_id = warpgroup_id
        warpgroup_id += 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 93-96 / 第 93-96 行

~~~~python
        self.cvt_warpgroup_ids = tuple(
            range(warpgroup_id, warpgroup_id + convert_warpgroups)
        )
        warpgroup_id += convert_warpgroups
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 98-102 / 第 98-102 行

~~~~python
        # Why 2 MMA+TMA warps when not MMA bound?
        # Less register pressure per warp promotes concise SASS
        # hides MMA 'switching' latency that gets exposed with less concise SASS
        # We would have 2 leftover warps if we do warpgroup reg realloc
        # and less register pressure gives more realloc flexibility
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 103-108 / 第 103-108 行

~~~~python
        self.mma_kq_warp_id = warpgroup_id * warpgroup_warps + 0
        self.mma_vp_warp_id = warpgroup_id * warpgroup_warps + 1
        self.tma_kv_warp_id = warpgroup_id * warpgroup_warps + 2
        self.tma_qo_warp_id = warpgroup_id * warpgroup_warps + 3
        self.mma_tma_warpgroup_id = warpgroup_id
        warpgroup_id += 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 110-110 / 第 110-110 行

~~~~python
        self.threads_per_cta = warpgroup_id * warpgroup_threads
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 112-124 / 第 112-124 行

~~~~python
        self.use_reg_reconfig = grouped_head_tile > 16
        max_regs_per_wg_thread = 64 * 1024 // warpgroup_threads  # 64K regs per SM
        self.mma_tma_regs = 72
        self.cvt_regs = 112
        self.softmax_regs = (
            max_regs_per_wg_thread
            - self.mma_tma_regs
            - self.cvt_regs * convert_warpgroups
        )
        self.softmax_regs = max(128, min(256, self.softmax_regs))
        assert (
            self.mma_tma_regs + self.softmax_regs + self.cvt_regs * convert_warpgroups
        ) <= max_regs_per_wg_thread or not self.use_reg_reconfig
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 126-128 / 第 126-128 行

~~~~python
        self.bs_stages = 2
        self.sp_stages = 2
        self.o_stages = 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 130-139 / 第 130-139 行

~~~~python
    def can_implement(
        self,
        problem_shape,
        kv_splits,
        q_dtype,
        kv_dtype,
        o_dtype,
        acc_dtype,
    ):
        b, h_q, h_k, s_k, d = problem_shape
~~~~

**EN**: Defines `can_implement`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `can_implement`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 141-142 / 第 141-142 行

~~~~python
        if kv_dtype is cutlass.Float8E4M3:
            raise ValueError("use Float8E4M3FN instead of Float8E4M3")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 144-145 / 第 144-145 行

~~~~python
        if d % 64 != 0:
            raise testing.CantImplementError(f"headdim({d}) must be multiple of 64")
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 147-150 / 第 147-150 行

~~~~python
        if h_q % h_k != 0:
            raise testing.CantImplementError(
                f"heads_q({h_q}) must be a multiple of heads_k({h_k})"
            )
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 152-158 / 第 152-158 行

~~~~python
        align_scale_bits = 128  # TMA requirement
        if self.scaledim * q_dtype.width < align_scale_bits:
            align_seq = align_scale_bits // (self.scaledim * q_dtype.width)
            if s_k % align_seq != 0:
                raise testing.CantImplementError(
                    f"seqlen({s_k}) must be a multiple of {align_seq}"
                )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 160-163 / 第 160-163 行

~~~~python
        if kv_dtype.width < 8 and d % 128 != 0:  # TMA requirement
            raise testing.CantImplementError(
                f"headdim({d}) must be multiple of 128 for {kv_dtype} KV"
            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 165-192 / 第 165-192 行

~~~~python
    @cute.jit
    def __call__(
        self,
        problem_shape: Tuple[
            cutlass.Int32, cutlass.Int32, cutlass.Int32, cutlass.Int32, cutlass.Int32
        ],  # b, h_q, h_k, s_k, d
        kv_splits: cutlass.Int32,  # threadblocks per sequence
        q_iter: cute.Pointer,
        k_iter: cute.Pointer,
        v_iter: cute.Pointer,
        k_scale_iter: cute.Pointer,
        v_scale_iter: cute.Pointer,
        o_iter: cute.Pointer,
        m_iter: cute.Pointer,  # colmax_s, must be -inf initialized
        l_iter: cute.Pointer,  # logsumexp
        o_partial_iter: cute.Pointer,  # partial O per kv split
        m_partial_iter: cute.Pointer,  # partial colmax_s per kv split
        l_partial_iter: cute.Pointer,  # partial colsum_p per kv split
        scale_qs: cutlass.Float32,
        scale_o: cutlass.Float32,
        stream: cuda.CUstream,
    ):
        ##############################
        # TiledMma creation
        ##############################
        mma_dtype = q_iter.dtype
        acc_dtype = o_partial_iter.dtype
        assert acc_dtype is cutlass.Float32  # don't support other acc types for now
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 194-194 / 第 194-194 行

~~~~python
        # Block tile sets the granularity at which threadblocks consume work
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 195-198 / 第 195-198 行

~~~~python
        blk_tile_s = 128
        blk_tile_h = self.grouped_head_tile
        blk_tile_d = self.headdim
        blk_tile_shd = (blk_tile_s, blk_tile_h, blk_tile_d)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 200-200 / 第 200-200 行

~~~~python
        # MMA tile sets the granularity at which TMAs + MMAs are issued
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 201-205 / 第 201-205 行

~~~~python
        mma_tile_m = 128
        mma_tile_n = self.grouped_head_tile
        mma_tile_k = 128 if self.headdim % 128 == 0 else 64
        mma_tile_mnk = (mma_tile_m, mma_tile_n, mma_tile_k)
        assert self.headdim % mma_tile_k == 0
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 207-207 / 第 207-207 行

~~~~python
        # GEMM1: (S_K, H_R, D, (H_K, B))
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 208-217 / 第 208-217 行

~~~~python
        tiled_mma_kq = sm100_utils.make_trivial_tiled_mma(
            mma_dtype,
            mma_dtype,
            OperandMajorMode.K,  # K
            OperandMajorMode.K,  # Q
            acc_dtype,
            tcgen05.CtaGroup.ONE,
            mma_tile_mnk[:2],
            tcgen05.OperandSource.TMEM,  # converted K in tmem
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 219-219 / 第 219-219 行

~~~~python
        # GEMM2: (D, H_R, S_K, (H_K, B))
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 220-229 / 第 220-229 行

~~~~python
        tiled_mma_vp = sm100_utils.make_trivial_tiled_mma(  #
            mma_dtype,
            mma_dtype,
            OperandMajorMode.K,  # V
            OperandMajorMode.MN,  # P
            acc_dtype,
            tcgen05.CtaGroup.ONE,
            mma_tile_mnk[:2],
            tcgen05.OperandSource.TMEM,  # converted V in tmem
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 231-231 / 第 231-231 行

~~~~python
        # Calculate Q stages
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 232-232 / 第 232-232 行

~~~~python
        self.q_stages = blk_tile_d // mma_tile_k
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 234-234 / 第 234-234 行

~~~~python
        # Perf heuristics
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 235-237 / 第 235-237 行

~~~~python
        cap_kv_stages = k_iter.dtype.width >= 8
        max_cvt_stages = 4 if self.grouped_head_tile == 32 and mma_tile_k == 128 else 8
        max_kv_stages = 8 if mma_tile_k == 128 else 14
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 239-239 / 第 239-239 行

~~~~python
        # Calculate KV tmem stages
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 240-247 / 第 240-247 行

~~~~python
        tmem_alloc_cols = mma_tile_n * self.sp_stages
        tmem_alloc_cols += mma_tile_n * self.o_stages * (blk_tile_d // mma_tile_m)
        tmem_capacity = 512
        cvt_stage_cols = mma_tile_k * mma_dtype.width // 32
        self.cvt_stages = (tmem_capacity - tmem_alloc_cols) // cvt_stage_cols
        self.cvt_stages = (
            min(self.cvt_stages, max_cvt_stages) if cap_kv_stages else self.cvt_stages
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 249-252 / 第 249-252 行

~~~~python
        tmem_alloc_cols += self.cvt_stages * cvt_stage_cols
        self.tmem_alloc_cols = 2 ** math.ceil(
            math.log2(tmem_alloc_cols)
        )  # Tmem alloc must be PO2
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 254-254 / 第 254-254 行

~~~~python
        print(f"\tcvt stages: {self.cvt_stages}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 256-256 / 第 256-256 行

~~~~python
        # Calculate KV smem stages
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 257-269 / 第 257-269 行

~~~~python
        self.mbarrier_reserved_bytes = 768
        smem_alloc_bits = self.mbarrier_reserved_bytes * 8
        smem_alloc_bits += mma_tile_n * acc_dtype.width  # colmax
        smem_alloc_bits += (
            self.scaledim * blk_tile_s * self.bs_stages * mma_dtype.width
        )  # block scale
        smem_alloc_bits += mma_tile_n * warpgroup_warps * acc_dtype.width  # colsum
        smem_alloc_bits += (
            mma_tile_n * mma_tile_k * self.q_stages * mma_dtype.width
        )  # Q
        smem_alloc_bits += (
            mma_tile_m * mma_tile_n * self.sp_stages * mma_dtype.width
        )  # P
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 271-278 / 第 271-278 行

~~~~python
        smem_capacity = utils.get_smem_capacity_in_bytes("sm_100")
        kv_smem_dtype = cutlass.Int8 if k_iter.dtype.width < 8 else k_iter.dtype
        self.kv_stages = (smem_capacity * 8 - smem_alloc_bits) // (
            mma_tile_m * mma_tile_k * kv_smem_dtype.width
        )
        self.kv_stages = (
            min(self.kv_stages, max_kv_stages) if cap_kv_stages else self.kv_stages
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 280-280 / 第 280-280 行

~~~~python
        print(f"\tkv stages: {self.kv_stages}")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 282-284 / 第 282-284 行

~~~~python
        ##############################
        # TMA creation
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 285-286 / 第 285-286 行

~~~~python
        b, h_q, h_k, s_k, d = problem_shape
        h_r = h_q // h_k
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 288-291 / 第 288-291 行

~~~~python
        q = cute.make_tensor(
            q_iter,
            cute.make_ordered_layout(shape=(h_r, d, (h_k, b)), order=(1, 0, (2, 3))),
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 293-297 / 第 293-297 行

~~~~python
        k = cute.make_tensor(
            k_iter,
            cute.make_ordered_layout(shape=(s_k, d, (h_k, b)), order=(1, 0, (2, 3))),
        )
        assert k_iter.dtype is not q_iter.dtype
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 299-303 / 第 299-303 行

~~~~python
        v = cute.make_tensor(
            v_iter,
            cute.make_ordered_layout(shape=(d, s_k, (h_k, b)), order=(0, 1, (2, 3))),
        )
        assert v_iter.dtype is k_iter.dtype
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 305-310 / 第 305-310 行

~~~~python
        o_partial = cute.make_tensor(
            o_partial_iter,
            cute.make_ordered_layout(
                shape=(d, h_r, (h_k, b), kv_splits), order=(0, 1, (2, 3), 4)
            ),
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 312-319 / 第 312-319 行

~~~~python
        m = cute.make_tensor(
            m_iter,
            cute.make_ordered_layout(
                shape=(h_r, (h_k, b)),
                order=(0, (1, 2)),
            ),
        )
        assert m_iter.dtype is acc_dtype
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 321-328 / 第 321-328 行

~~~~python
        m_partial = cute.make_tensor(
            m_partial_iter,
            cute.make_ordered_layout(
                shape=(h_r, (h_k, b), kv_splits),
                order=(0, (1, 2), 3),
            ),
        )
        assert m_partial_iter.dtype is acc_dtype
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 330-337 / 第 330-337 行

~~~~python
        l_partial = cute.make_tensor(
            l_partial_iter,
            cute.make_ordered_layout(
                shape=(h_r, (h_k, b), kv_splits),
                order=(0, (1, 2), 3),
            ),
        )
        assert l_partial_iter.dtype is acc_dtype
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 339-349 / 第 339-349 行

~~~~python
        align_scale_bits = 128  # TMA requirement
        if cutlass.const_expr(self.scaledim * mma_dtype.width >= align_scale_bits):
            scale_layout = cute.make_ordered_layout(
                shape=(self.scaledim, s_k, (h_k, b)), order=(0, 1, (2, 3))
            )
        else:
            align_seq = align_scale_bits // (self.scaledim * mma_dtype.width)
            s_ks = (align_seq, s_k // align_seq)
            scale_layout = cute.make_ordered_layout(
                shape=(self.scaledim, s_ks, (h_k, b)), order=(0, (1, 2), (3, 4))
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 351-352 / 第 351-352 行

~~~~python
        k_scale = cute.make_tensor(k_scale_iter, scale_layout)
        assert k_scale_iter.dtype is mma_dtype
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 354-355 / 第 354-355 行

~~~~python
        v_scale = cute.make_tensor(v_scale_iter, scale_layout)
        assert v_scale_iter.dtype is mma_dtype
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 357-357 / 第 357-357 行

~~~~python
        # (MMA, MMA_M/N, MMA_K, Stages)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 358-366 / 第 358-366 行

~~~~python
        smem_layout_q = sm100_utils.make_smem_layout_b(
            tiled_mma_kq, mma_tile_mnk, q_iter.dtype, self.q_stages
        )
        smem_layout_k = sm100_utils.make_smem_layout_a(
            tiled_mma_kq, mma_tile_mnk, kv_smem_dtype, self.kv_stages
        )
        smem_layout_v = sm100_utils.make_smem_layout_a(
            tiled_mma_vp, mma_tile_mnk, kv_smem_dtype, self.kv_stages, is_k_major=False
        )  # V is always headdim-major (GEMM2 M-major) in gmem+smem
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 368-368 / 第 368-368 行

~~~~python
        smem_layout_bs = cute.make_layout((self.scaledim, blk_tile_s, self.bs_stages))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 370-376 / 第 370-376 行

~~~~python
        smem_layout_atom_o = tcgen05.make_smem_layout_atom(
            tcgen05.mma.SmemLayoutAtomKind.MN_SW128, o_partial_iter.dtype
        )
        smem_layout_o = cute.tile_to_shape(
            smem_layout_atom_o, (blk_tile_d, blk_tile_h), order=(1, 0)
        )
        smem_layout_o = cute.flat_divide(smem_layout_o, (mma_tile_m, mma_tile_n))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 378-379 / 第 378-379 行

~~~~python
        tma_load_op = cute.nvgpu.cpasync.CopyBulkTensorTileG2SOp()
        tma_store_op = cute.nvgpu.cpasync.CopyBulkTensorTileS2GOp()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 381-408 / 第 381-408 行

~~~~python
        tma_atom_q, tma_tensor_q = cute.nvgpu.make_tiled_tma_atom_B(
            tma_load_op,
            q,
            cute.select(smem_layout_q, mma_modes),
            mma_tile_mnk,
            tiled_mma_kq,
        )
        tma_atom_k, tma_tensor_k = cute.nvgpu.make_tiled_tma_atom_A(
            tma_load_op,
            k,
            cute.select(smem_layout_k, mma_modes),
            mma_tile_mnk,
            tiled_mma_kq,
            internal_type=kv_smem_dtype,
        )
        tma_atom_v, tma_tensor_v = cute.nvgpu.make_tiled_tma_atom_A(
            tma_load_op,
            v,
            cute.select(smem_layout_v, mma_modes),
            mma_tile_mnk,
            tiled_mma_vp,
            internal_type=kv_smem_dtype,
        )
        tma_atom_ks, tma_tensor_ks = cute.nvgpu.cpasync.make_tiled_tma_atom(
            tma_load_op,
            k_scale,
            cute.select(smem_layout_bs, mode=[0, 1]),
            smem_layout_bs.shape[:2],
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 409-421 / 第 409-421 行

~~~~python
        )
        tma_atom_vs, tma_tensor_vs = cute.nvgpu.cpasync.make_tiled_tma_atom(
            tma_load_op,
            v_scale,
            cute.select(smem_layout_bs, mode=[0, 1]),
            smem_layout_bs.shape[:2],
        )
        tma_atom_o, tma_tensor_o = cute.nvgpu.cpasync.make_tiled_tma_atom(
            tma_store_op,
            o_partial,
            cute.select(smem_layout_o, mode=[0, 1]),
            mma_tile_mnk[:2],
        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 423-424 / 第 423-424 行

~~~~python
        # K scale and V scale will have the same TMA tensor (coord tensor)
        # only difference is base ptr which is stored in copy atom
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 425-425 / 第 425-425 行

~~~~python
        tma_tensor_bs = tma_tensor_ks
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 427-429 / 第 427-429 行

~~~~python
        ##############################
        # Decode Kernel launch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 430-430 / 第 430-430 行

~~~~python
        scale_qs_log2_e = scale_qs * log2_e
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 432-434 / 第 432-434 行

~~~~python
        n_tiles = cute.ceil_div(h_r, blk_tile_h)
        l_tiles = b * h_k
        grid = (kv_splits, n_tiles, l_tiles)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 436-463 / 第 436-463 行

~~~~python
        self.decode(
            blk_tile_shd,
            mma_tile_mnk,
            tiled_mma_kq,
            tiled_mma_vp,
            q_iter.dtype,
            smem_layout_q,
            tma_atom_q,
            tma_tensor_q,
            k_iter.dtype,
            smem_layout_k,
            tma_atom_k,
            tma_tensor_k,
            v_iter.dtype,
            smem_layout_v,
            tma_atom_v,
            tma_tensor_v,
            smem_layout_bs,
            tma_atom_ks,
            tma_atom_vs,
            tma_tensor_bs,
            o_partial_iter.dtype,
            smem_layout_o,
            tma_atom_o,
            tma_tensor_o,
            m,
            m_partial,
            l_partial,
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 464-472 / 第 464-472 行

~~~~python
            scale_qs,
            scale_qs_log2_e,
        ).launch(
            grid=grid,
            block=[self.threads_per_cta, 1, 1],
            cluster=[1, 1, 1],
            stream=stream,
            min_blocks_per_mp=1,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 474-476 / 第 474-476 行

~~~~python
        ##############################
        # Reduction Kernel launch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 477-479 / 第 477-479 行

~~~~python
        o = cute.make_tensor(o_iter, cute.make_layout((d, h_q, b)))
        m = cute.make_tensor(m_iter, cute.make_layout((h_q, b)))
        l = cute.make_tensor(l_iter, cute.make_layout((h_q, b)))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 481-489 / 第 481-489 行

~~~~python
        o_partial = cute.make_tensor(
            o_partial_iter, cute.make_layout((d, h_q, b, kv_splits))
        )
        m_partial = cute.make_tensor(
            m_partial_iter, cute.make_layout((h_q, b, kv_splits))
        )
        l_partial = cute.make_tensor(
            l_partial_iter, cute.make_layout((h_q, b, kv_splits))
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 491-492 / 第 491-492 行

~~~~python
        d_per_blk = 128
        d_blks = cute.ceil_div(d, d_per_blk)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 494-499 / 第 494-499 行

~~~~python
        self.reduction(o, m, l, o_partial, m_partial, l_partial, scale_o).launch(
            grid=[d_blks, h_q, b],
            block=[d_per_blk, 1, 1],
            cluster=[1, 1, 1],
            stream=stream,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 501-528 / 第 501-528 行

~~~~python
    @cute.kernel
    def decode(
        self,
        # MMA
        blk_tile_shd: cute.Tile,
        mma_tile_mnk: cute.Tile,
        tiled_mma_kq: cute.TiledMma,
        tiled_mma_vp: cute.TiledMma,
        # Q
        q_dtype: Type[cutlass.Numeric],
        smem_layout_q: cute.ComposedLayout,
        tma_atom_q: cute.CopyAtom,
        mQ: cute.Tensor,
        # K
        k_dtype: Type[cutlass.Numeric],
        smem_layout_k: cute.ComposedLayout,
        tma_atom_k: cute.CopyAtom,
        mK: cute.Tensor,
        # V
        v_dtype: Type[cutlass.Numeric],
        smem_layout_v: cute.ComposedLayout,
        tma_atom_v: cute.CopyAtom,
        mV: cute.Tensor,
        # K/V block scale
        smem_layout_bs: cute.Layout,
        tma_atom_ks: cute.CopyAtom,
        tma_atom_vs: cute.CopyAtom,
        mBS: cute.Tensor,
~~~~

**EN**: Declares `decode` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 `decode` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 529-550 / 第 529-550 行

~~~~python
        # O
        o_dtype: Type[cutlass.Numeric],
        smem_layout_o: cute.ComposedLayout,
        tma_atom_o: cute.CopyAtom,
        mO: cute.Tensor,
        # Rest
        mM: cute.Tensor,
        mM_partial: cute.Tensor,
        mL_partial: cute.Tensor,
        scale_qs: cutlass.Float32,
        scale_qs_log2_e: cutlass.Float32,
    ):
        # Read special registers
        kv_splits, tiles_hr, tiles_hb = cute.arch.grid_dim()
        kv_split_idx, coord_hr, coord_hb = cute.arch.block_idx()
        tidx, _, _ = cute.arch.thread_idx()
        lane_idx = cute.arch.lane_idx()
        warp_idx = cute.arch.make_warp_uniform(tidx // warp_threads)
        warpgroup_idx = cute.arch.make_warp_uniform(tidx // warpgroup_threads)
        warpgroup_tidx = tidx % warpgroup_threads
        warpgroup_widx = warp_idx % warpgroup_warps
        init_warp = 0
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 552-552 / 第 552-552 行

~~~~python
        # No multicast
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 553-554 / 第 553-554 行

~~~~python
        mcast_coord = 0
        mcast_layout = cute.make_layout((1, 1, 1, 1))  # vmnk
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 556-556 / 第 556-556 行

~~~~python
        # Alias types
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 557-559 / 第 557-559 行

~~~~python
        mma_dtype = q_dtype
        acc_dtype = o_dtype
        kv_smem_dtype = cutlass.Int8 if k_dtype.width < 8 else k_dtype
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 561-561 / 第 561-561 行

~~~~python
        # Shapes for MMA tile indexing (Read TMA partition for example)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 562-575 / 第 562-575 行

~~~~python
        blk_tile_s, blk_tile_h, blk_tile_d = blk_tile_shd
        mma_tile_m, mma_tile_n, mma_tile_k = mma_tile_mnk
        tiles_dm, tiles_sk = cute.ceil_div(
            (blk_tile_d, blk_tile_s), (mma_tile_m, mma_tile_k)
        )
        tiles_dk, tiles_sm = cute.ceil_div(
            (blk_tile_d, blk_tile_s), (mma_tile_k, mma_tile_m)
        )
        tiles_s = cute.ceil_div(mK.shape[0], blk_tile_s)
        iters_s = cute.ceil_div(tiles_s - kv_split_idx, kv_splits)
        prefetch_iters = self.sp_stages - 1
        if iters_s < prefetch_iters:
            prefetch_iters = iters_s
        assert tiles_sm == 1
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 577-577 / 第 577-577 行

~~~~python
        # Runtime checks
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 578-579 / 第 578-579 行

~~~~python
        exit_early = kv_split_idx >= tiles_s
        lane_store_max = mma_tile_n == warp_threads or lane_idx < mma_tile_n
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 581-581 / 第 581-581 行

~~~~python
        # Smem alloc helper
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 582-584 / 第 582-584 行

~~~~python
        svector_align = 16
        stensor_align = 128
        smem = utils.SmemAllocator()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 586-588 / 第 586-588 行

~~~~python
        ##############################
        # Prefetch TMA descriptor
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 589-596 / 第 589-596 行

~~~~python
        if warp_idx == init_warp and not exit_early:
            cute.nvgpu.cpasync.prefetch_descriptor(tma_atom_q)
            cute.nvgpu.cpasync.prefetch_descriptor(tma_atom_k)
            cute.nvgpu.cpasync.prefetch_descriptor(tma_atom_v)
            cute.nvgpu.cpasync.prefetch_descriptor(tma_atom_ks)
            cute.nvgpu.cpasync.prefetch_descriptor(tma_atom_vs)
            cute.nvgpu.cpasync.prefetch_descriptor(tma_atom_o)
        init_warp += 1
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 598-600 / 第 598-600 行

~~~~python
        ##############################
        # Tmem Allocation
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 601-604 / 第 601-604 行

~~~~python
        tmem_ptr_smem_ptr = smem.allocate_array(cutlass.Int32)
        if warp_idx == init_warp and not exit_early:
            cute.arch.alloc_tmem(self.tmem_alloc_cols, tmem_ptr_smem_ptr)
        init_warp += 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 606-609 / 第 606-609 行

~~~~python
        ##############################
        # Pipeline Allocation + Init
        ##############################
        # Allocate Mbarriers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 610-616 / 第 610-616 行

~~~~python
        q_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.q_stages * 2)
        kv_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.kv_stages * 2)
        bs_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.bs_stages * 2)
        cvt_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.cvt_stages * 2)
        s_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.sp_stages * 2)
        p_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.sp_stages * 2)
        o_pipeline_ptr = smem.allocate_array(cutlass.Int64, self.o_stages * 2)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 618-618 / 第 618-618 行

~~~~python
        # Declare named barriers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 619-623 / 第 619-623 行

~~~~python
        softmax_nbar = pipeline.NamedBarrier(
            barrier_id=1, num_threads=warpgroup_threads
        )
        mma_kq_nbar = pipeline.NamedBarrier(barrier_id=2, num_threads=64)
        mma_vp_nbar = pipeline.NamedBarrier(barrier_id=3, num_threads=64)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 625-625 / 第 625-625 行

~~~~python
        # Alias thread cooperatives
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 626-636 / 第 626-636 行

~~~~python
        elect_one_cooperative = pipeline.CooperativeGroup(pipeline.Agent.Thread)
        warpgroup_cooperative = pipeline.CooperativeGroup(
            pipeline.Agent.Thread, warpgroup_threads
        )
        mma_group = elect_one_cooperative
        tma_group = elect_one_cooperative
        cvt_group = warpgroup_cooperative
        cvt_groups = pipeline.CooperativeGroup(
            pipeline.Agent.Thread, warpgroup_threads * self.convert_warpgroups
        )
        softmax_group = warpgroup_cooperative
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 638-638 / 第 638-638 行

~~~~python
        # Initialize pipelines
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 639-666 / 第 639-666 行

~~~~python
        q_producer, q_consumer = pipeline.PipelineTmaAsync.create(
            num_stages=self.q_stages,
            producer_group=tma_group,
            consumer_group=softmax_group,  # Reuse Q consumer mbarriers to sync O store
            tx_count=cute.size_in_bytes(q_dtype, cute.select(smem_layout_q, mma_modes)),
            barrier_storage=q_pipeline_ptr,
            tidx=mcast_coord,
            cta_layout_vmnk=mcast_layout,
            defer_sync=True,
        ).make_participants()
        kv_producer, kv_consumer = pipeline.PipelineTmaAsync.create(
            num_stages=self.kv_stages,
            producer_group=tma_group,
            consumer_group=cvt_group,
            tx_count=cute.size_in_bytes(k_dtype, cute.select(smem_layout_k, mma_modes)),
            barrier_storage=kv_pipeline_ptr,
            tidx=mcast_coord,
            cta_layout_vmnk=mcast_layout,
            defer_sync=True,
        ).make_participants()
        bs_producer, bs_consumer = pipeline.PipelineTmaAsync.create(
            num_stages=self.bs_stages,
            producer_group=tma_group,
            consumer_group=cvt_groups,
            tx_count=cute.size_in_bytes(
                mma_dtype, cute.select(smem_layout_bs, mode=[0, 1])
            ),
            barrier_storage=bs_pipeline_ptr,
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 667-694 / 第 667-694 行

~~~~python
            tidx=mcast_coord,
            cta_layout_vmnk=mcast_layout,
            defer_sync=True,
        ).make_participants()
        cvt_producer, cvt_consumer = pipeline.PipelineAsyncUmma.create(
            num_stages=self.cvt_stages,
            producer_group=cvt_group,
            consumer_group=mma_group,
            barrier_storage=cvt_pipeline_ptr,
            defer_sync=True,
        ).make_participants()
        s_producer, s_consumer = pipeline.PipelineUmmaAsync.create(
            num_stages=self.sp_stages,
            producer_group=mma_group,
            consumer_group=softmax_group,
            barrier_storage=s_pipeline_ptr,
            defer_sync=True,
        ).make_participants()
        p_producer, p_consumer = pipeline.PipelineAsyncUmma.create(
            num_stages=self.sp_stages,
            producer_group=softmax_group,
            consumer_group=mma_group,
            barrier_storage=p_pipeline_ptr,
            defer_sync=True,
        ).make_participants()
        o_producer, o_consumer = pipeline.PipelineUmmaAsync.create(
            num_stages=self.o_stages,
            producer_group=mma_group,
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 695-698 / 第 695-698 行

~~~~python
            consumer_group=softmax_group,
            barrier_storage=o_pipeline_ptr,
            defer_sync=True,
        ).make_participants()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 700-700 / 第 700-700 行

~~~~python
        # Ensure visibility of local mbarrier inits and tmem alloc
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 701-701 / 第 701-701 行

~~~~python
        cute.arch.sync_threads()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 703-706 / 第 703-706 行

~~~~python
        ##############################
        # MMA Partition + Allocate
        ##############################
        # Threadblock slice
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 707-708 / 第 707-708 行

~~~~python
        thrblk_mma_kq = tiled_mma_kq.get_slice(0)
        thrblk_mma_vp = tiled_mma_vp.get_slice(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 710-710 / 第 710-710 行

~~~~python
        # M - colmax
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 711-713 / 第 711-713 行

~~~~python
        sM_layout = cute.make_layout(shape=(mma_tile_m, mma_tile_n), stride=(0, 1))
        sM = smem.allocate_tensor(acc_dtype, sM_layout, svector_align)
        tCsM = thrblk_mma_kq.partition_C(sM)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 715-715 / 第 715-715 行

~~~~python
        # L - colsum
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 716-720 / 第 716-720 行

~~~~python
        sL_layout = cute.make_layout(
            shape=(mma_tile_m, mma_tile_n, warpgroup_warps), stride=(0, 1, mma_tile_n)
        )
        sL = smem.allocate_tensor(acc_dtype, sL_layout, svector_align)
        tCsL = thrblk_mma_kq.partition_C(sL)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 722-722 / 第 722-722 行

~~~~python
        # BS - block scale
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 723-725 / 第 723-725 行

~~~~python
        sBS = smem.allocate_tensor(
            mma_dtype, smem_layout_bs, stensor_align
        )  # (SCALE, TILE_S, bs_stages)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 727-727 / 第 727-727 行

~~~~python
        # Q
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 728-730 / 第 728-730 行

~~~~python
        tBsQ = smem.allocate_tensor(
            q_dtype, smem_layout_q.outer, stensor_align, smem_layout_q.inner
        )  # (MMA, #MMA_N, #MMA_K, q_stages)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 732-732 / 第 732-732 行

~~~~python
        # K
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 733-739 / 第 733-739 行

~~~~python
        tAsK = smem.allocate_tensor(
            kv_smem_dtype, smem_layout_k.outer, stensor_align, smem_layout_k.inner
        )  # (MMA, #MMA_M, #MMA_K, kv_stages)
        tAtK_cvt_shape = tiled_mma_kq.partition_shape_A(
            (mma_tile_m, mma_tile_k, self.cvt_stages)
        )  # (MMA, #MMA_M, #MMA_K, cvt_stages)
        tAtK_cvt = thrblk_mma_kq.make_fragment_A(tAtK_cvt_shape)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 741-741 / 第 741-741 行

~~~~python
        # V
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 742-751 / 第 742-751 行

~~~~python
        tAsV_iterator = cute.recast_ptr(
            tAsK.iterator, smem_layout_v.inner, dtype=kv_smem_dtype
        )  # KV share input buffers
        tAsV = cute.make_tensor(
            tAsV_iterator, smem_layout_v.outer
        )  # (MMA, #MMA_M, #MMA_K, kv_stages)
        tAtV_cvt_shape = tiled_mma_vp.partition_shape_A(
            (mma_tile_m, mma_tile_k, self.cvt_stages)
        )  # (MMA, #MMA_M, #MMA_K, cvt_stages)
        tAtV_cvt = thrblk_mma_vp.make_fragment_A(tAtV_cvt_shape)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 753-753 / 第 753-753 行

~~~~python
        # S
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 754-759 / 第 754-759 行

~~~~python
        tCtS_shape = tiled_mma_kq.partition_shape_C(
            (mma_tile_m, mma_tile_n, self.sp_stages)
        )
        tCtS = thrblk_mma_kq.make_fragment_C(
            tCtS_shape
        )  # (MMA_MN, #MMA_M=1, #MMA_N=1, sp_stages)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 761-762 / 第 761-762 行

~~~~python
        # P - Treat MN C tile of BMM0 as NM B tile of BMM1
        # (MMA_NK, #MMA_N, #MMA_K=MMA_TILE_M/MMA_K, sp_stages)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 763-769 / 第 763-769 行

~~~~python
        mma_tile_nm = (None, mma_tile_n, mma_tile_m)
        tBsP_nm_layout = sm100_utils.make_smem_layout_b(
            tiled_mma_vp, mma_tile_nm, mma_dtype, self.sp_stages
        )
        tBsP_nm = smem.allocate_tensor(
            mma_dtype, tBsP_nm_layout.outer, stensor_align, tBsP_nm_layout.inner
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 771-772 / 第 771-772 行

~~~~python
        # Tile for NK B tile iteration
        # (MMA_NK, #MMA_N, #MMA_K=MMA_TILE_K/MMA_K, #TILES_SK=MMA_TILE_M/MMA_TILE_K, sp_stages)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 773-774 / 第 773-774 行

~~~~python
        tBsP_nk_tile = thrblk_mma_vp.partition_shape_B((mma_tile_n, mma_tile_k))
        tBsP_nk = cute.local_tile(tBsP_nm, tBsP_nk_tile, (0, 0, None, None))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 776-778 / 第 776-778 行

~~~~python
        # Reshape NM B tile of BMM1 to become MN C tile of BMM0
        # (MMA_NK, #MMA_N, #MMA_K=MMA_TILE_M/MMA_K, sp_stages) ->
        # (MMA_MN, #MMA_M, #MMA_N, sp_stages)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 779-780 / 第 779-780 行

~~~~python
        tCsP_tile = cute.make_ordered_layout(tCtS_shape, order=((2, 0), 3, 1, 4))
        tCsP = cute.composition(tBsP_nm, tCsP_tile)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 782-782 / 第 782-782 行

~~~~python
        # O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 783-792 / 第 783-792 行

~~~~python
        sO_iterator = cute.recast_ptr(
            tBsQ.iterator, smem_layout_o.inner, dtype=o_dtype
        )  # Reuse QKV smem for O TMA store
        sO_mma = cute.make_tensor(
            sO_iterator, smem_layout_o.outer
        )  # (MMA_TILE_M, MMA_TILE_N, #TILE_DM, #TILE_HN)
        tCsO = thrblk_mma_vp.partition_C(
            sO_mma
        )  # (MMA, #MMA_M, #MMA_N, #TILE_DM, #TILE_HN)
        tCtO = thrblk_mma_vp.make_fragment_C(tCsO.shape)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 794-794 / 第 794-794 行

~~~~python
        # Tmem tensor allocation
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 795-796 / 第 795-796 行

~~~~python
        tmem_ptr = cute.arch.retrieve_tmem_ptr(cutlass.Int32, 16, tmem_ptr_smem_ptr)
        tmem_offset = 0
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 798-804 / 第 798-804 行

~~~~python
        tAtK_cvt = cute.make_tensor(
            cute.recast_ptr(tmem_ptr + tmem_offset, dtype=mma_dtype), tAtK_cvt.layout
        )
        tAtV_cvt = cute.make_tensor(
            cute.recast_ptr(tmem_ptr + tmem_offset, dtype=mma_dtype), tAtV_cvt.layout
        )
        tmem_offset += tcgen05.find_tmem_tensor_col_offset(tAtK_cvt)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 806-809 / 第 806-809 行

~~~~python
        tCtS = cute.make_tensor(
            cute.recast_ptr(tmem_ptr + tmem_offset, dtype=acc_dtype), tCtS.layout
        )
        tmem_offset += tcgen05.find_tmem_tensor_col_offset(tCtS)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 811-814 / 第 811-814 行

~~~~python
        tCtO = cute.make_tensor(
            cute.recast_ptr(tmem_ptr + tmem_offset, dtype=acc_dtype), tCtO.layout
        )
        tmem_offset += tcgen05.find_tmem_tensor_col_offset(tCtO)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 816-819 / 第 816-819 行

~~~~python
        print(
            f"\t{tmem_offset} tmem cols used, {self.tmem_alloc_cols} tmem cols allocated"
        )
        assert tmem_offset <= self.tmem_alloc_cols
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 821-823 / 第 821-823 行

~~~~python
        ##############################
        # Exit early
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 824-825 / 第 824-825 行

~~~~python
        if exit_early:
            noop = None  # early return not supported # noqa: F841
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 827-829 / 第 827-829 行

~~~~python
        ##############################
        # TMA KV Dispatch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 830-833 / 第 830-833 行

~~~~python
        elif warp_idx == self.tma_kv_warp_id:
            # Free registers
            if cutlass.const_expr(self.use_reg_reconfig):
                cute.arch.setmaxregister_decrease(self.mma_tma_regs)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 835-835 / 第 835-835 行

~~~~python
            # Apply block tiler and slice
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 836-841 / 第 836-841 行

~~~~python
            gK = cute.local_tile(
                mK, tiler=(blk_tile_s, blk_tile_d), coord=(None, 0, coord_hb)
            )  # (TILE_S, TILE_D, #TILE_S)
            gV = cute.local_tile(
                mV, tiler=(blk_tile_d, blk_tile_s), coord=(0, None, coord_hb)
            )  # (TILE_D, TILE_S, #TILE_S)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 843-843 / 第 843-843 行

~~~~python
            # Apply MMA tiler and MMA partition
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 844-855 / 第 844-855 行

~~~~python
            gK_mma = cute.flat_divide(
                gK, (mma_tile_m, mma_tile_k)
            )  # (MMA_TILE_M, MMA_TILE_K, #TILE_SM, #TILE_DK, #TILE_S)
            gV_mma = cute.flat_divide(
                gV, (mma_tile_m, mma_tile_k)
            )  # (MMA_TILE_M, MMA_TILE_K, #TILE_DM, #TILE_SK, #TILE_S)
            tAgK = thrblk_mma_kq.partition_A(
                gK_mma
            )  # (MMA, #MMA_M, #MMA_K, #TILE_SM, #TILE_DK, #TILE_S)
            tAgV = thrblk_mma_vp.partition_A(
                gV_mma
            )  # (MMA, #MMA_M, #MMA_K, #TILE_DM, #TILE_SK, #TILE_S)
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 857-865 / 第 857-865 行

~~~~python
            # #TILE_SM=TILE_S/MMA_TILE_M, #TILE_HN=TILE_H/MMA_TILE_N, #TILE_DK=TILE_D/MMA_TILE_K
            # #TILE_DM=TILE_D/MMA_TILE_M, #TILE_HN=TILE_H/MMA_TILE_N, #TILE_SK=TILE_S/MMA_TILE_K
            #
            # Example with TILE_S=MMA_TILE_M=128, TILE_H=MMA_TILE_N=8, MMA_TILE_K=64, TILE_D=512
            # BMM1: MMA=128x8x16, #MMA_M=1, #MMA_N=1, #MMA_K=4, #TILE_SM=1, #TILE_HN=1, #TILE_DK=8, #TILE_S=S/128
            # BMM2: MMA=128x8x16, #MMA_M=1, #MMA_N=1, #MMA_K=4, #TILE_DM=4, #TILE_HN=1, #TILE_SK=2, #TILE_S=S/128

            # TMA partition
            # (MMA, #MMA_M, #MMA_K, Rest...) -> (TMA, Rest...)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 866-872 / 第 866-872 行

~~~~python
            tGSsK, tGSgK = cute.nvgpu.cpasync.tma_partition(
                tma_atom_k,
                mcast_coord,
                mcast_layout,
                smem_tensor=cute.group_modes(tAsK, 0, 3),
                gmem_tensor=cute.group_modes(tAgK, 0, 3),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 874-880 / 第 874-880 行

~~~~python
            tGSsV, tGSgV = cute.nvgpu.cpasync.tma_partition(
                tma_atom_v,
                mcast_coord,
                mcast_layout,
                smem_tensor=cute.group_modes(tAsV, 0, 3),
                gmem_tensor=cute.group_modes(tAgV, 0, 3),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 882-884 / 第 882-884 行

~~~~python
            #
            # Sequence loop
            #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 885-897 / 第 885-897 行

~~~~python
            prefetch_tiles = prefetch_iters * kv_splits
            for s in cutlass.range(kv_split_idx, prefetch_tiles + tiles_s, kv_splits):
                # Load K
                if s < tiles_s:
                    tGSgK_s = tGSgK[None, None, None, s]
                    for dk in cutlass.range_constexpr(tiles_dk):
                        k_handle = kv_producer.acquire_and_advance()
                        cute.copy(
                            tma_atom_k,
                            tGSgK_s[None, 0, dk],
                            tGSsK[None, k_handle.index],
                            tma_bar_ptr=k_handle.barrier,
                        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 899-899 / 第 899-899 行

~~~~python
                # Load V
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 900-910 / 第 900-910 行

~~~~python
                if s >= prefetch_tiles:
                    tGSgV_s = tGSgV[None, None, None, s - prefetch_tiles]
                    for sk in cutlass.range_constexpr(tiles_sk):
                        for dm in cutlass.range_constexpr(tiles_dm):
                            v_handle = kv_producer.acquire_and_advance()
                            cute.copy(
                                tma_atom_v,
                                tGSgV_s[None, dm, sk],
                                tGSsV[None, v_handle.index],
                                tma_bar_ptr=v_handle.barrier,
                            )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 912-914 / 第 912-914 行

~~~~python
        ##############################
        # TMA QO Dispatch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 915-918 / 第 915-918 行

~~~~python
        elif warp_idx == self.tma_qo_warp_id:
            # Free registers
            if cutlass.const_expr(self.use_reg_reconfig):
                cute.arch.setmaxregister_decrease(self.mma_tma_regs)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 920-920 / 第 920-920 行

~~~~python
            # Apply block tiler and slice
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 921-931 / 第 921-931 行

~~~~python
            gQ = cute.local_tile(
                mQ, tiler=(blk_tile_h, blk_tile_d), coord=(coord_hr, 0, coord_hb)
            )  # (TILE_H, TILE_D)
            gO = cute.local_tile(
                mO,
                tiler=(blk_tile_d, blk_tile_h),
                coord=(0, coord_hr, coord_hb, kv_split_idx),
            )  # (TILE_D, TILE_H)
            gBS = cute.local_tile(
                mBS, tiler=(self.scaledim, blk_tile_s), coord=(0, None, coord_hb)
            )  # (SCALE, TILE_S, #TILE_S)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 933-933 / 第 933-933 行

~~~~python
            # Apply MMA tiler and MMA partition
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 934-942 / 第 934-942 行

~~~~python
            gQ_mma = cute.flat_divide(
                gQ, (mma_tile_n, mma_tile_k)
            )  # (MMA_TILE_N, MMA_TILE_K, #TILE_HN, #TILE_DK)
            gO_mma = cute.flat_divide(
                gO, (mma_tile_m, mma_tile_n)
            )  # (MMA_TILE_M, MMA_TILE_N, #TILE_DM, #TILE_HN)
            tBgQ = thrblk_mma_kq.partition_B(
                gQ_mma
            )  # (MMA, #MMA_N, #MMA_K, #TILE_HN, #TILE_DK)
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 944-944 / 第 944-944 行

~~~~python
            # TMA partition
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 945-951 / 第 945-951 行

~~~~python
            tGSsQ, tGSgQ = cute.nvgpu.cpasync.tma_partition(
                tma_atom_q,
                mcast_coord,
                mcast_layout,
                smem_tensor=cute.group_modes(tBsQ, 0, 3),
                gmem_tensor=cute.group_modes(tBgQ, 0, 3),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 953-959 / 第 953-959 行

~~~~python
            tSGsO, tSGgO = cute.nvgpu.cpasync.tma_partition(
                tma_atom_o,
                mcast_coord,
                mcast_layout,
                smem_tensor=cute.group_modes(sO_mma, 0, 2),
                gmem_tensor=cute.group_modes(gO_mma, 0, 2),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 961-967 / 第 961-967 行

~~~~python
            tGSsBS, tGSgBS = cute.nvgpu.cpasync.tma_partition(
                tma_atom_ks,
                mcast_coord,
                mcast_layout,
                smem_tensor=cute.group_modes(sBS, 0, 2),
                gmem_tensor=cute.group_modes(gBS, 0, 2),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 969-969 / 第 969-969 行

~~~~python
            # Load Q
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 970-977 / 第 970-977 行

~~~~python
            for dk in cutlass.range_constexpr(tiles_dk):
                q_handle = q_producer.acquire_and_advance()
                cute.copy(
                    tma_atom_q,
                    tGSgQ[None, 0, dk],  # stages_q == tiles_dk by construction
                    tGSsQ[None, dk],
                    tma_bar_ptr=q_handle.barrier,
                )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 979-979 / 第 979-979 行

~~~~python
            # Sequence Loop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 980-989 / 第 980-989 行

~~~~python
            prefetch_tiles = prefetch_iters * kv_splits
            for s in cutlass.range(kv_split_idx, prefetch_tiles + tiles_s, kv_splits):
                if s < tiles_s:
                    bs_handle = bs_producer.acquire_and_advance()
                    cute.copy(
                        tma_atom_ks,
                        tGSgBS[None, s],
                        tGSsBS[None, bs_handle.index],
                        tma_bar_ptr=bs_handle.barrier,
                    )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 991-998 / 第 991-998 行

~~~~python
                if s >= prefetch_tiles:
                    bs_handle = bs_producer.acquire_and_advance()
                    cute.copy(
                        tma_atom_vs,
                        tGSgBS[None, s - prefetch_tiles],
                        tGSsBS[None, bs_handle.index],
                        tma_bar_ptr=bs_handle.barrier,
                    )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 1000-1000 / 第 1000-1000 行

~~~~python
            # Store O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1001-1003 / 第 1001-1003 行

~~~~python
            for dm in cutlass.range_constexpr(tiles_dm):
                q_producer.acquire_and_advance()  # Reuse Q load barriers to sync O store
                cute.copy(tma_atom_o, tSGsO[None, dm, 0], tSGgO[None, dm, 0])
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1005-1007 / 第 1005-1007 行

~~~~python
        ##############################
        # Convert Dispatch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1008-1011 / 第 1008-1011 行

~~~~python
        elif warpgroup_idx in self.cvt_warpgroup_ids:
            # Free registers
            if cutlass.const_expr(self.use_reg_reconfig):
                cute.arch.setmaxregister_decrease(self.cvt_regs)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1013-1013 / 第 1013-1013 行

~~~~python
            # Intermediate convert type
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1014-1019 / 第 1014-1019 行

~~~~python
            cvt_type = cutlass.Float32
            if cutlass.const_expr(
                mma_dtype is cutlass.BFloat16
                and k_dtype in (cutlass.Int4, cutlass.Int8)
            ):
                cvt_type = mma_dtype
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1021-1021 / 第 1021-1021 行

~~~~python
            # Initialize for multiple warpgroups if necessary
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1022-1029 / 第 1022-1029 行

~~~~python
            convert_phase = 0
            if cutlass.const_expr(self.convert_warpgroups > 1):
                assert tiles_dk % self.convert_warpgroups == 0
                assert (tiles_dm * tiles_sk) % self.convert_warpgroups == 0
                convert_phase = warpgroup_idx % self.convert_warpgroups
                for _ in cutlass.range(convert_phase):
                    kv_consumer.advance()
                    cvt_producer.advance()
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1031-1031 / 第 1031-1031 行

~~~~python
            # Construct tiled copy and partition K
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1032-1043 / 第 1032-1043 行

~~~~python
            mma_k_bits = mma_tile_k * mma_dtype.width
            tmem_store_atom_k = cute.make_copy_atom(
                tcgen05.St16x256bOp(tcgen05.Repetition(mma_k_bits // 256)),
                mma_dtype,
            )
            smem_load_atom_k = cute.make_copy_atom(
                cute.nvgpu.warp.LdMatrix8x16x8bOp(
                    num_matrices=4,
                    unpack_bits=(k_dtype.width if k_dtype.width < 8 else None),
                ),
                kv_smem_dtype,
            )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1045-1050 / 第 1045-1050 行

~~~~python
            tmem_store_k = tcgen05.make_tmem_copy(
                tmem_store_atom_k, tAtK_cvt[mma_dice + (0,)]
            )
            thr_store_k = tmem_store_k.get_slice(warpgroup_tidx)
            tKrK_cvt_shape = thr_store_k.partition_S(tAtK_cvt).shape[:-1]
            tKtK_cvt = thr_store_k.partition_D(tAtK_cvt)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1052-1055 / 第 1052-1055 行

~~~~python
            smem_load_k = cute.make_tiled_copy_S(smem_load_atom_k, tmem_store_k)
            thr_load_k = smem_load_k.get_slice(warpgroup_tidx)
            tKsK = thr_load_k.partition_S(tAsK)
            tKrK_shape = thr_load_k.partition_D(tAsK).shape[:-1]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1057-1057 / 第 1057-1057 行

~~~~python
            # Construct tiled copy and partition V
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1058-1066 / 第 1058-1066 行

~~~~python
            tmem_store_atom_v = cute.make_copy_atom(
                tcgen05.St16x256bOp(tcgen05.Repetition(mma_k_bits // 256)), mma_dtype
            )
            smem_load_op_v = cute.nvgpu.warp.LdMatrix16x16x8bOp(
                transpose=True,
                num_matrices=2,
                unpack_bits=(v_dtype.width if v_dtype.width < 8 else None),
            )
            smem_load_atom_v = cute.make_copy_atom(smem_load_op_v, kv_smem_dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1068-1073 / 第 1068-1073 行

~~~~python
            tmem_store_v = tcgen05.make_tmem_copy(
                tmem_store_atom_v, tAtV_cvt[mma_dice + (0,)]
            )
            thr_store_v = tmem_store_v.get_slice(warpgroup_tidx)
            tVrV_cvt_shape = thr_store_v.partition_S(tAtV_cvt).shape[:-1]
            tVtV_cvt = thr_store_v.partition_D(tAtV_cvt)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1075-1078 / 第 1075-1078 行

~~~~python
            smem_load_v = cute.make_tiled_copy_S(smem_load_atom_v, tmem_store_v)
            thr_load_v = smem_load_v.get_slice(warpgroup_tidx)
            tVsV = thr_load_v.partition_S(tAsV)
            tVrV_shape = thr_load_v.partition_D(tAsV).shape[:-1]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1080-1080 / 第 1080-1080 行

~~~~python
            # Partition KS - K block scale
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1081-1100 / 第 1081-1100 行

~~~~python
            sKS_layout = cute.make_layout(
                shape=(
                    blk_tile_s,
                    (self.block_scaledim, self.scaledim),
                    self.bs_stages,
                ),
                stride=(self.scaledim, (0, 1), blk_tile_s * self.scaledim),
            )
            sKS = cute.make_tensor(
                sBS.iterator, sKS_layout
            )  # (TILE_S, TILE_D, bs_stages)
            sKS_mma = cute.group_modes(
                cute.flat_divide(sKS, (mma_tile_m, mma_tile_k)), 2, 4
            )  # (MMA_TILE_M, MMA_TILE_K, (#TILE_SM, #TILE_DK), bs_stages)
            tAsKS = thrblk_mma_kq.partition_A(
                sKS_mma
            )  # (MMA, #MMA_M, #MMA_K, (#TILE_SM, #TILE_DK), bs_stages)
            tKsKS = thr_load_k.partition_D(
                tAsKS
            )  # (CPY, CPY_MMA, CPY_M, CPY_K, #TILE, bs_stages)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1102-1102 / 第 1102-1102 行

~~~~python
            # Partition VS - V block scale
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1103-1122 / 第 1103-1122 行

~~~~python
            sVS_layout = cute.make_layout(
                shape=(
                    (self.block_scaledim, self.scaledim),
                    blk_tile_s,
                    self.bs_stages,
                ),
                stride=((0, 1), self.scaledim, blk_tile_s * self.scaledim),
            )
            sVS = cute.make_tensor(
                sBS.iterator, sVS_layout
            )  # (TILE_D, TILE_S, bs_stages)
            sVS_mma = cute.group_modes(
                cute.flat_divide(sVS, (mma_tile_m, mma_tile_k)), 2, 4
            )  # (MMA_TILE_M, MMA_TILE_K, (#TILE_DM, #TILE_SK), bs_stages)
            tAsVS = thrblk_mma_vp.partition_A(
                sVS_mma
            )  # (MMA, #MMA_M, #MMA_K, (#TILE_DM, #TILE_SK), bs_stages)
            tVsVS = thr_load_v.partition_D(
                tAsVS
            )  # (CPY, CPY_MMA, CPY_M, CPY_K, #TILE, bs_stages)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1124-1126 / 第 1124-1126 行

~~~~python
            #
            # Sequence loop
            #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1127-1136 / 第 1127-1136 行

~~~~python
            for s in cutlass.range(prefetch_iters + iters_s):
                if s < iters_s:
                    # Load K scale
                    bs_handle = bs_consumer.wait_and_advance()
                    tKrKS = cute.make_rmem_tensor_like(
                        tKsKS[cpy_dice + (None, 0)]
                    )  # 'like' preserves 0 strides
                    cute.autovec_copy(tKsKS[cpy_dice + (None, bs_handle.index)], tKrKS)
                    cute.arch.fence_view_async_shared()
                    bs_handle.release()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1138-1138 / 第 1138-1138 行

~~~~python
                    # Convert and scale K
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1139-1143 / 第 1139-1143 行

~~~~python
                    for dk in cutlass.range(
                        tiles_dk // self.convert_warpgroups, unroll=2
                    ):
                        tKrK = cute.make_rmem_tensor(tKrK_shape, kv_smem_dtype)
                        tKrK_cvt = cute.make_rmem_tensor(tKrK_cvt_shape, mma_dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1145-1148 / 第 1145-1148 行

~~~~python
                        kv_handle = kv_consumer.wait_and_advance()
                        cute.copy(thr_load_k, tKsK[cpy_dice + (kv_handle.index,)], tKrK)
                        cute.arch.fence_view_async_shared()
                        kv_handle.release()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1150-1150 / 第 1150-1150 行

~~~~python
                        # Sign extend unpacked int4 to int8
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1151-1158 / 第 1151-1158 行

~~~~python
                        if cutlass.const_expr(k_dtype is cutlass.Int4):
                            tKrK_unpacked_i4_vec = tKrK.load().maybe_downcast()
                            tKrK_i8_vec = cute.arch.sext_unpacked_i4_i8_intrinsic(
                                tKrK_unpacked_i4_vec, cute.size(tKrK_shape)
                            )
                            tKrK.store(
                                cute.TensorSSA(tKrK_i8_vec, tKrK_shape, cutlass.Int8)
                            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1160-1163 / 第 1160-1163 行

~~~~python
                        coord_dk = dk * self.convert_warpgroups + convert_phase
                        scale_k = tKrKS[cpy_dice + (coord_dk,)].load()
                        tKrK_ssa = tKrK.load().to(cvt_type).to(mma_dtype) * scale_k
                        tKrK_cvt.store(tKrK_ssa.reshape(tKrK_cvt_shape))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1165-1172 / 第 1165-1172 行

~~~~python
                        cvt_handle = cvt_producer.acquire_and_advance()
                        cute.copy(
                            thr_store_k,
                            tKrK_cvt,
                            tKtK_cvt[cpy_dice + (cvt_handle.index,)],
                        )
                        cute.arch.fence_view_async_tmem_store()
                        cvt_handle.commit()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1174-1174 / 第 1174-1174 行

~~~~python
                        # Advance again for multiple warpgroups
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1175-1177 / 第 1175-1177 行

~~~~python
                        for _ in cutlass.range_constexpr(self.convert_warpgroups - 1):
                            kv_consumer.advance()
                            cvt_producer.advance()
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1179-1187 / 第 1179-1187 行

~~~~python
                if s >= prefetch_iters:
                    # Load V scale
                    bs_handle = bs_consumer.wait_and_advance()
                    tVrVS = cute.make_rmem_tensor_like(
                        tVsVS[cpy_dice + (None, 0)]
                    )  # 'like' preserves 0 strides
                    cute.autovec_copy(tVsVS[cpy_dice + (None, bs_handle.index)], tVrVS)
                    cute.arch.fence_view_async_shared()
                    bs_handle.release()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1189-1189 / 第 1189-1189 行

~~~~python
                    # Convert and scale V
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1190-1194 / 第 1190-1194 行

~~~~python
                    for dmsk in cutlass.range(
                        tiles_dm * tiles_sk // self.convert_warpgroups, unroll=2
                    ):
                        tVrV = cute.make_rmem_tensor(tVrV_shape, kv_smem_dtype)
                        tVrV_cvt = cute.make_rmem_tensor(tVrV_cvt_shape, mma_dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1196-1199 / 第 1196-1199 行

~~~~python
                        kv_handle = kv_consumer.wait_and_advance()
                        cute.copy(thr_load_v, tVsV[cpy_dice + (kv_handle.index,)], tVrV)
                        cute.arch.fence_view_async_shared()
                        kv_handle.release()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1201-1201 / 第 1201-1201 行

~~~~python
                        # Sign extend unpacked int4 to int8
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1202-1209 / 第 1202-1209 行

~~~~python
                        if cutlass.const_expr(v_dtype is cutlass.Int4):
                            tVrV_unpacked_i4_vec = tVrV.load().maybe_downcast()
                            tVrV_i8_vec = cute.arch.sext_unpacked_i4_i8_intrinsic(
                                tVrV_unpacked_i4_vec, cute.size(tVrV_shape)
                            )
                            tVrV.store(
                                cute.TensorSSA(tVrV_i8_vec, tVrV_shape, cutlass.Int8)
                            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1211-1214 / 第 1211-1214 行

~~~~python
                        coord_dmsk = dmsk * self.convert_warpgroups + convert_phase
                        scale_v = tVrVS[cpy_dice + (coord_dmsk,)].load()
                        tVrV_ssa = tVrV.load().to(cvt_type).to(mma_dtype) * scale_v
                        tVrV_cvt.store(tVrV_ssa.reshape(tVrV_cvt_shape))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1216-1223 / 第 1216-1223 行

~~~~python
                        cvt_handle = cvt_producer.acquire_and_advance()
                        cute.copy(
                            thr_store_v,
                            tVrV_cvt,
                            tVtV_cvt[cpy_dice + (cvt_handle.index,)],
                        )
                        cute.arch.fence_view_async_tmem_store()
                        cvt_handle.commit()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1225-1225 / 第 1225-1225 行

~~~~python
                        # Advance again for multiple warpgroups
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1226-1228 / 第 1226-1228 行

~~~~python
                        for _ in cutlass.range_constexpr(self.convert_warpgroups - 1):
                            kv_consumer.advance()
                            cvt_producer.advance()
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1230-1232 / 第 1230-1232 行

~~~~python
        ##############################
        # MMA KQ Dispatch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1233-1236 / 第 1233-1236 行

~~~~python
        elif warp_idx == self.mma_kq_warp_id:
            # Free registers
            if cutlass.const_expr(self.use_reg_reconfig):
                cute.arch.setmaxregister_decrease(self.mma_tma_regs)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1238-1238 / 第 1238-1238 行

~~~~python
            # Setup mma descriptors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1239-1239 / 第 1239-1239 行

~~~~python
            tBsQ_desc = thrblk_mma_kq.make_fragment_B(tBsQ)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1241-1241 / 第 1241-1241 行

~~~~python
            # Wait for Q
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1242-1243 / 第 1242-1243 行

~~~~python
            for dk in cutlass.range_constexpr(tiles_dk):
                q_consumer.wait_and_advance()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1245-1245 / 第 1245-1245 行

~~~~python
            # Sequence loop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1246-1271 / 第 1246-1271 行

~~~~python
            s_token = True  # Producer always acquires first
            for s in cutlass.range(iters_s):
                # BMM1
                k_token = cvt_consumer.try_wait()
                tiled_mma_kq.set(tcgen05.Field.ACCUMULATE, False)
                s_handle = s_producer.acquire_and_advance(s_token)
                for dk in cutlass.range_constexpr(tiles_dk):
                    is_last_iter = dk == tiles_dk - 1
                    k_handle = cvt_consumer.wait_and_advance(k_token)
                    # Signal BMM2 to start
                    if is_last_iter:
                        mma_kq_nbar.arrive()
                    for mma_k in cutlass.range_constexpr(tAtK_cvt.shape[2]):
                        cute.gemm(
                            tiled_mma_kq,
                            tCtS[mma_dice + (s_handle.index,)],
                            tAtK_cvt[None, None, mma_k, k_handle.index],
                            tBsQ_desc[None, None, mma_k, dk],
                            tCtS[mma_dice + (s_handle.index,)],
                        )
                        if dk == 0 and mma_k == 0:
                            tiled_mma_kq.set(tcgen05.Field.ACCUMULATE, True)
                    k_handle.release()
                    if not is_last_iter:
                        k_token = cvt_consumer.try_wait()
                s_handle.commit()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1273-1273 / 第 1273-1273 行

~~~~python
                # Advance and wait for BMM 2
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1274-1278 / 第 1274-1278 行

~~~~python
                if s > 0:
                    for _ in cutlass.range_constexpr(tiles_dm * tiles_sk):
                        cvt_consumer.advance()
                    mma_vp_nbar.arrive_and_wait()
                    s_token = s_producer.try_acquire()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1280-1282 / 第 1280-1282 行

~~~~python
        ##############################
        # MMA VP Dispatch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1283-1286 / 第 1283-1286 行

~~~~python
        elif warp_idx == self.mma_vp_warp_id:
            # Free registers
            if cutlass.const_expr(self.use_reg_reconfig):
                cute.arch.setmaxregister_decrease(self.mma_tma_regs)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1288-1288 / 第 1288-1288 行

~~~~python
            # Setup mma descriptors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1289-1290 / 第 1289-1290 行

~~~~python
            tiled_mma_vp.set(tcgen05.Field.ACCUMULATE, True)
            tBsP_desc = thrblk_mma_vp.make_fragment_B(tBsP_nk)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1292-1292 / 第 1292-1292 行

~~~~python
            # Advance and wait for BMM1
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1293-1295 / 第 1293-1295 行

~~~~python
            for _ in cutlass.range_constexpr(tiles_dk):
                cvt_consumer.advance()
            mma_kq_nbar.arrive_and_wait()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1297-1297 / 第 1297-1297 行

~~~~python
            # Sequence loop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1298-1306 / 第 1298-1306 行

~~~~python
            p_token = False
            o_token = True  # Producer always acquires first
            for s in cutlass.range(iters_s):
                # Advance and wait for BMM1
                if s < iters_s - 1:
                    for _ in cutlass.range_constexpr(tiles_dk):
                        cvt_consumer.advance()
                    mma_kq_nbar.arrive_and_wait()
                    p_token = p_consumer.try_wait()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1308-1308 / 第 1308-1308 行

~~~~python
                # BMM2
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1309-1332 / 第 1309-1332 行

~~~~python
                v_token = cvt_consumer.try_wait()
                p_handle = p_consumer.wait_and_advance(p_token)
                o_handle = o_producer.acquire_and_advance(o_token)
                for sk in cutlass.range_constexpr(tiles_sk):
                    for dm in cutlass.range_constexpr(tiles_dm):
                        is_last_iter = sk == tiles_sk - 1 and dm == tiles_dm - 1
                        v_handle = cvt_consumer.wait_and_advance(v_token)
                        # Signal BMM1 to start
                        if is_last_iter:
                            mma_vp_nbar.arrive()
                        for mma_k in cutlass.range_constexpr(tAtV_cvt.shape[2]):
                            cute.gemm(
                                tiled_mma_vp,
                                tCtO[mma_dice + (dm, 0)],
                                tAtV_cvt[None, None, mma_k, v_handle.index],
                                tBsP_desc[None, None, mma_k, sk, p_handle.index],
                                tCtO[mma_dice + (dm, 0)],
                            )
                        v_handle.release()
                        if not is_last_iter:
                            v_token = cvt_consumer.try_wait()
                p_handle.release()
                o_handle.commit()
                o_token = o_producer.try_acquire()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1334-1334 / 第 1334-1334 行

~~~~python
            # Wait for signal to dealloc tmem, then dealloc
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1335-1337 / 第 1335-1337 行

~~~~python
            o_producer.tail()
            cute.arch.relinquish_tmem_alloc_permit()
            cute.arch.dealloc_tmem(tmem_ptr, self.tmem_alloc_cols)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1339-1341 / 第 1339-1341 行

~~~~python
        ##############################
        # Softmax + Correction Dispatch
        ##############################
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1342-1345 / 第 1342-1345 行

~~~~python
        elif warpgroup_idx == self.softmax_warpgroup_id:
            # Alloc registers
            if cutlass.const_expr(self.use_reg_reconfig):
                cute.arch.setmaxregister_increase(self.softmax_regs)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1347-1347 / 第 1347-1347 行

~~~~python
            # Construct tiled copies
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1348-1358 / 第 1348-1358 行

~~~~python
            tmem_op_width = 32
            tmem_op_repeat = tcgen05.Repetition(
                mma_tile_n * acc_dtype.width // tmem_op_width
            )
            tmem_load_atom_s = cute.make_copy_atom(
                tcgen05.Ld32x32bOp(tmem_op_repeat), acc_dtype
            )
            tmem_load_s = tcgen05.make_tmem_copy(
                tmem_load_atom_s, tCtS[mma_dice + (0,)]
            )
            thr_load_s = tmem_load_s.get_slice(warpgroup_tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1360-1366 / 第 1360-1366 行

~~~~python
            tmem_store_atom_o = cute.make_copy_atom(
                tcgen05.St32x32bOp(tmem_op_repeat), o_dtype
            )
            tmem_store_o = tcgen05.make_tmem_copy(
                tmem_store_atom_o, tCtO[mma_dice + (0, 0)]
            )
            thr_store_o = tmem_store_o.get_slice(warpgroup_tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1368-1368 / 第 1368-1368 行

~~~~python
            # Partition S and P
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1369-1374 / 第 1369-1374 行

~~~~python
            tStS = thr_load_s.partition_S(
                tCtS
            )  # (CPY, #CPY_MMA, #CPY_M, #CPY_N, stages_sp)
            tSsP = thr_load_s.partition_D(
                tCsP
            )  # (CPY, #CPY_MMA, #CPY_M, #CPY_N, stages_sp)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1376-1376 / 第 1376-1376 行

~~~~python
            # Partition O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1377-1383 / 第 1377-1383 行

~~~~python
            tStO = thr_load_s.partition_S(
                tCtO
            )  # (CPY, #CPY_MMA, #CPY_M, #CPY_N, #TILE_DM, #TILE_HN)
            tSsO = thr_load_s.partition_D(
                tCsO
            )  # (CPY, #CPY_MMA, #CPY_M, #CPY_N, #TILE_DM, #TILE_HN)
            tSrO = cute.make_rmem_tensor_like(tSsO)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1385-1385 / 第 1385-1385 行

~~~~python
            # Partition colmax and initialize in RF
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1386-1388 / 第 1386-1388 行

~~~~python
            tSsM = thr_load_s.partition_D(tCsM)  # (CPY, #CPY_MMA, #CPY_M, #CPY_N)
            tSrM_prev = cute.make_rmem_tensor_like(tSsM)
            tSrM_prev.fill(-cutlass.Float32.inf)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1390-1391 / 第 1390-1391 行

~~~~python
            # Partition colsum and initialize in RF
            # Each thread maintains a local colsum in RF, smem reduction happens after loop
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1392-1396 / 第 1392-1396 行

~~~~python
            tSsL = thr_load_s.partition_D(
                tCsL
            )  # (CPY, #CPY_MMA, #CPY_M, #CPY_N, WARPS)
            tSrL = cute.make_rmem_tensor_like(tSsL[cpy_dice + (0,)])
            tSrL.fill(cutlass.Float32(0))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 1398-1398 / 第 1398-1398 行

~~~~python
            assert warp_threads >= cute.size(tSsM)
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1400-1400 / 第 1400-1400 行

~~~~python
            # get gmem colmax + colsum to store to
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1401-1414 / 第 1401-1414 行

~~~~python
            inbound_hr = coord_hr * blk_tile_h + lane_idx < mM.shape[0]
            gM = cute.local_tile(
                mM, tiler=(mma_tile_n, 1), coord=(coord_hr, coord_hb)
            )  # (TILE_H) = (MMA_TILE_N)
            gM_partial = cute.local_tile(
                mM_partial,
                tiler=(mma_tile_n, 1),
                coord=(coord_hr, coord_hb, kv_split_idx),
            )
            gL_partial = cute.local_tile(
                mL_partial,
                tiler=(mma_tile_n, 1),
                coord=(coord_hr, coord_hb, kv_split_idx),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1416-1416 / 第 1416-1416 行

~~~~python
            # Initialize O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1417-1418 / 第 1417-1418 行

~~~~python
            tSrO.fill(cutlass.Float32(0))
            cute.copy(thr_store_o, tSrO, tStO)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1420-1420 / 第 1420-1420 行

~~~~python
            # Initialize colsum and colmax in smem and wait
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1421-1425 / 第 1421-1425 行

~~~~python
            if warpgroup_widx == 0 and lane_store_max:
                tSsM[lane_idx] = -cutlass.Float32.inf
            if warpgroup_widx == 1 and lane_store_max:
                tSsL[lane_idx] = cutlass.Float32(0)
            softmax_nbar.arrive_and_wait()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1427-1429 / 第 1427-1429 行

~~~~python
            #
            # Sequence loop
            #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1430-1436 / 第 1430-1436 行

~~~~python
            for s in cutlass.range(iters_s):
                # Load S from tmem
                s_handle = s_consumer.wait_and_advance()
                tSrS = cute.make_rmem_tensor(tSsP.shape[:-1], acc_dtype)
                cute.copy(tmem_load_s, tStS[cpy_dice + (s_handle.index,)], tSrS)
                cute.arch.fence_view_async_tmem_load()
                s_handle.release()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 1438-1438 / 第 1438-1438 行

~~~~python
                # Reduce colmax in warp RF
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1439-1444 / 第 1439-1444 行

~~~~python
                tSrM = cute.make_rmem_tensor_like(tSsM)
                tSrM_lane = cutlass.Float32(0)  # Avoid dynamic register indexing
                for i in cutlass.range_constexpr(cute.size(tSrS)):
                    tSrM[i] = warp_fmax(tSrS[i])
                    if i == lane_idx:
                        tSrM_lane = tSrM[i]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 1446-1446 / 第 1446-1446 行

~~~~python
                # Reduce colmax in smem
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1447-1448 / 第 1447-1448 行

~~~~python
                if lane_store_max:
                    smem_fmax(tSsM.iterator + tSsM.layout(lane_idx), tSrM_lane)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1450-1450 / 第 1450-1450 行

~~~~python
                # Wait for colmax then load
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1451-1452 / 第 1451-1452 行

~~~~python
                softmax_nbar.arrive_and_wait()
                cute.autovec_copy(tSsM, tSrM)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1454-1454 / 第 1454-1454 行

~~~~python
                # Compute online softmax
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1455-1468 / 第 1455-1468 行

~~~~python
                tSrP = cute.make_rmem_tensor(tSsP.shape[:-1], mma_dtype)
                if cutlass.const_expr(use_tensor_ssa_math):
                    tSrP_f32 = exp2(scale_qs_log2_e * (tSrS.load() - tSrM.load()))
                    tSrP.store(tSrP_f32.to(mma_dtype))  # convert
                else:
                    tSrP_f32 = cute.make_rmem_tensor(tSrS.shape, acc_dtype)
                    for i in cutlass.range_constexpr(0, cute.size(tSrS), 2):
                        p_f32x2 = fadd2(
                            (tSrS[i], tSrS[i + 1]), (-tSrM[i], -tSrM[i + 1])
                        )
                        p_f32x2 = fmul2(p_f32x2, (scale_qs_log2_e, scale_qs_log2_e))
                        tSrP_f32[i] = exp2(p_f32x2[0])
                        tSrP_f32[i + 1] = exp2(p_f32x2[1])
                    tSrP.store(tSrP_f32.load().to(mma_dtype))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1470-1470 / 第 1470-1470 行

~~~~python
                # Store P to smem
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1471-1474 / 第 1471-1474 行

~~~~python
                p_handle = p_producer.acquire_and_advance()
                cute.autovec_copy(tSrP, tSsP[cpy_dice + (p_handle.index,)])
                cute.arch.fence_view_async_shared()
                p_handle.commit()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1476-1476 / 第 1476-1476 行

~~~~python
                # Compute correction and correct colsum
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1477-1498 / 第 1477-1498 行

~~~~python
                if cutlass.const_expr(use_tensor_ssa_math):
                    correction = exp2(
                        scale_qs_log2_e * (tSrM_prev.load() - tSrM.load())
                    )
                    tSrL.store(tSrL.load() * correction + tSrP_f32)
                else:
                    correction = cute.make_rmem_tensor_like(tSrM)
                    for i in cutlass.range_constexpr(0, cute.size(tSrM), 2):
                        c_f32x2 = fadd2(
                            (tSrM_prev[i], tSrM_prev[i + 1]), (-tSrM[i], -tSrM[i + 1])
                        )
                        c_f32x2 = fmul2(c_f32x2, (scale_qs_log2_e, scale_qs_log2_e))
                        c_f32x2 = (exp2(c_f32x2[0]), exp2(c_f32x2[1]))
                        correction[i] = c_f32x2[0]
                        correction[i + 1] = c_f32x2[1]
                        l_f32x2 = ffma2(
                            c_f32x2,
                            (tSrL[i], tSrL[i + 1]),
                            (tSrP_f32[i], tSrP_f32[i + 1]),
                        )
                        tSrL[i] = l_f32x2[0]
                        tSrL[i + 1] = l_f32x2[1]
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1500-1500 / 第 1500-1500 行

~~~~python
                # Correct O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1501-1503 / 第 1501-1503 行

~~~~python
                if s > 0:
                    # Wait for O
                    o_handle = o_consumer.wait_and_advance()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 1505-1505 / 第 1505-1505 行

~~~~python
                    # Apply correction
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1506-1508 / 第 1506-1508 行

~~~~python
                    for dm in cutlass.range_constexpr(tiles_dm):
                        tSrO_dm = cute.make_rmem_tensor_like(tSsO[cpy_dice + (0, 0)])
                        cute.copy(thr_load_s, tStO[cpy_dice + (dm, 0)], tSrO_dm)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 1510-1516 / 第 1510-1516 行

~~~~python
                        for i in cutlass.range_constexpr(0, cute.size(tSrO_dm), 2):
                            o_f32x2 = fmul2(
                                (tSrO_dm[i], tSrO_dm[i + 1]),
                                (correction[i], correction[i + 1]),
                            )
                            tSrO_dm[i] = o_f32x2[0]
                            tSrO_dm[i + 1] = o_f32x2[1]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1518-1518 / 第 1518-1518 行

~~~~python
                        cute.copy(thr_store_o, tSrO_dm, tStO[cpy_dice + (dm, 0)])
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1520-1520 / 第 1520-1520 行

~~~~python
                    # Notify MMA
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1521-1522 / 第 1521-1522 行

~~~~python
                    cute.arch.fence_view_async_tmem_store()
                    o_handle.release()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1524-1524 / 第 1524-1524 行

~~~~python
                # Update colmax
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1525-1525 / 第 1525-1525 行

~~~~python
                tSrM_prev.store(tSrM.load())
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1527-1531 / 第 1527-1531 行

~~~~python
            #
            # Softmax Epilogue
            #

            # Reduce colsum in warp RF
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1532-1536 / 第 1532-1536 行

~~~~python
            tSrL_lane = cutlass.Float32(0.0)
            for i in cutlass.range_constexpr(cute.size(tSrL)):
                tSrL[i] = cute.arch.warp_reduction_sum(tSrL[i])
                if i == lane_idx:
                    tSrL_lane = tSrL[i]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1538-1538 / 第 1538-1538 行

~~~~python
            # Store partial colsum in smem
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1539-1540 / 第 1539-1540 行

~~~~python
            if lane_store_max:
                tSsL[cpy_dice + (warpgroup_widx,)][lane_idx] = tSrL_lane
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1542-1542 / 第 1542-1542 行

~~~~python
            # Wait for colsum
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1543-1543 / 第 1543-1543 行

~~~~python
            softmax_nbar.arrive_and_wait()
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1545-1549 / 第 1545-1549 行

~~~~python
            if warpgroup_widx == 0 and lane_store_max and inbound_hr:
                # Load colsum and colmax
                sL_lane_wg = sL[0, lane_idx, None]
                sL_lane = sL_lane_wg[0] + sL_lane_wg[1] + sL_lane_wg[2] + sL_lane_wg[3]
                sM_lane = sM[0, lane_idx]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1551-1551 / 第 1551-1551 行

~~~~python
                # Scale colmax
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1552-1552 / 第 1552-1552 行

~~~~python
                sM_lane = sM_lane * scale_qs
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1554-1554 / 第 1554-1554 行

~~~~python
                # Store colsum and colmax
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1555-1557 / 第 1555-1557 行

~~~~python
                gL_partial[lane_idx] = sL_lane
                gM_partial[lane_idx] = sM_lane
                gmem_fmax(gM.iterator + gM.layout(lane_idx), sM_lane)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1559-1562 / 第 1559-1562 行

~~~~python
            o_handle = o_consumer.wait_and_advance()
            cute.copy(thr_load_s, tStO, tSrO)
            cute.arch.fence_view_async_tmem_load()
            o_handle.release()  # Final release signals tmem dealloc
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 1564-1564 / 第 1564-1564 行

~~~~python
            # Store O to smem
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1565-1567 / 第 1565-1567 行

~~~~python
            for dm in cutlass.range_constexpr(tiles_dm):
                tOrO_dm = tSrO[cpy_dice + (dm, 0)]
                tOsO_dm = tSsO[cpy_dice + (dm, 0)]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1569-1570 / 第 1569-1570 行

~~~~python
                cute.autovec_copy(tOrO_dm, tOsO_dm)
                cute.arch.fence_view_async_shared()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1572-1572 / 第 1572-1572 行

~~~~python
                # Reuse Q consumer barriers to notify O TMA store
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1573-1574 / 第 1573-1574 行

~~~~python
                q_consumer.release()
                q_consumer.advance()
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1576-1576 / 第 1576-1576 行

~~~~python
        return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1578-1592 / 第 1578-1592 行

~~~~python
    @staticmethod
    @cute.kernel
    def reduction(
        o: cute.Tensor,
        m: cute.Tensor,
        l: cute.Tensor,
        o_partial: cute.Tensor,
        m_partial: cute.Tensor,
        l_partial: cute.Tensor,
        scale_o: cutlass.Float32,
    ):
        d_blk_idx, coord_h, coord_b = cute.arch.block_idx()
        d_per_blk, _, _ = cute.arch.block_dim()
        d_idx, _, _ = cute.arch.thread_idx()
        coord_d = d_blk_idx * d_per_blk + d_idx
~~~~

**EN**: Declares `reduction` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `reduction` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1594-1596 / 第 1594-1596 行

~~~~python
        o_dhb = cutlass.Float32(0)
        m_hb = m[coord_h, coord_b]
        l_hb = cutlass.Float32(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1598-1600 / 第 1598-1600 行

~~~~python
        o_partial_dhb = o_partial[coord_d, coord_h, coord_b, None]
        m_partial_hb = m_partial[coord_h, coord_b, None]
        l_partial_hb = l_partial[coord_h, coord_b, None]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1602-1605 / 第 1602-1605 行

~~~~python
        for partial_idx in cutlass.range(o_partial.shape[-1]):
            correction = exp2(log2_e * (m_partial_hb[partial_idx] - m_hb))
            o_dhb += correction * o_partial_dhb[partial_idx]
            l_hb += correction * l_partial_hb[partial_idx]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1607-1608 / 第 1607-1608 行

~~~~python
        if coord_d < o.shape[0]:
            o[coord_d, coord_h, coord_b] = o.element_type(scale_o * (o_dhb / l_hb))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1610-1611 / 第 1610-1611 行

~~~~python
        if d_blk_idx == 0 and d_idx == 0:
            l[coord_h, coord_b] = m_hb + cute.math.log(l_hb, fastmath=False)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1613-1613 / 第 1613-1613 行

~~~~python
        return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1616-1643 / 第 1616-1643 行

~~~~python
def run(
    batches: int = 1,
    seqlen: int = 1024,
    heads_q: int = 32,
    heads_k: int = 4,
    headdim: int = 512,
    block_scaledim: int = 512,
    kv_splits: int = 0,
    q_dtype: Type[cutlass.Numeric] = cutlass.BFloat16,
    kv_dtype: Type[cutlass.Numeric] = cutlass.Int8,
    o_dtype: Type[cutlass.Numeric] = cutlass.BFloat16,
    acc_dtype: Type[cutlass.Numeric] = cutlass.Float32,
    tolerance: float = 0.1,
    scale_q: float = 1.0,
    scale_o: float = 1.0,
    scale_s: float = 0.0,
    warmup_iterations: int = 0,
    iterations: int = 0,
    skip_ref_check: bool = False,
    use_cold_l2: bool = False,
    **kwargs,
):
    print("Running Blackwell SM100 Mixed Input FMHA Decode test with:")
    print(f"\tbatches: {batches}, seqlen: {seqlen}")
    print(f"\theads_q: {heads_q}, heads_k: {heads_k}")
    print(f"\theaddim: {headdim}, block_scaledim: {block_scaledim}")
    print(f"\tkv_splits: {kv_splits}")
    print(f"\tq_dtype: {q_dtype}")
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1644-1654 / 第 1644-1654 行

~~~~python
    print(f"\tkv_dtype: {kv_dtype}")
    print(f"\to_dtype: {o_dtype}")
    print(f"\tacc_dtype: {acc_dtype}")
    print(f"\ttolerance: {tolerance}")
    print(f"\tscale_q: {scale_q}")
    print(f"\tscale_o: {scale_o}")
    print(f"\tscale_s: {scale_s}")
    print(f"\twarmup_iterations: {warmup_iterations}")
    print(f"\titerations: {iterations}")
    print(f"\tskip_ref_check: {skip_ref_check}")
    print(f"\tuse_cold_l2: {use_cold_l2}")
~~~~

**EN**: Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1656-1657 / 第 1656-1657 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 1659-1661 / 第 1659-1661 行

~~~~python
    #
    # Config Kernel
    #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1662-1673 / 第 1662-1673 行

~~~~python
    grouped_heads = heads_q // heads_k
    convert_warpgroups = 1
    if headdim == 512 and grouped_heads == 8 and kv_dtype.width == 4:
        convert_warpgroups = 4
    elif headdim > 128:
        convert_warpgroups = 2
    fmha = MixedInputFusedMultiHeadAttentionDecode(
        headdim=headdim,
        block_scaledim=block_scaledim,
        grouped_head_tile=min(cute.round_up(grouped_heads, 8), 32),
        convert_warpgroups=convert_warpgroups,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1675-1677 / 第 1675-1677 行

~~~~python
    if scale_s == 0.0:  # default to 1/sqrt(d)
        scale_s = 1.0 / math.sqrt(headdim)
    scale_qs = scale_q * scale_s
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1679-1688 / 第 1679-1688 行

~~~~python
    if kv_splits == 0:
        hardware_info = cutlass.utils.HardwareInfo()
        sm_count = hardware_info.get_device_multiprocessor_count()
        sm_count = 148 if sm_count <= 0 else sm_count
        grid_yz = batches * heads_k * math.ceil(grouped_heads / fmha.grouped_head_tile)
        kv_splits = sm_count // grid_yz  # 1 wave
        kv_splits = max(1, kv_splits)
        if sm_count == 148 and grid_yz == 32:
            kv_splits = 9  # 2 waves
        print(f"\tauto kv_splits: {kv_splits}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1690-1691 / 第 1690-1691 行

~~~~python
    seqlen_q = 1
    seqlen_k = seqlen
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1693-1693 / 第 1693-1693 行

~~~~python
    problem_shape = (batches, heads_q, heads_k, seqlen_k, headdim)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1695-1695 / 第 1695-1695 行

~~~~python
    fmha.can_implement(problem_shape, kv_splits, q_dtype, kv_dtype, o_dtype, acc_dtype)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1697-1699 / 第 1697-1699 行

~~~~python
    #
    # Allocate Tensors
    #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1700-1700 / 第 1700-1700 行

~~~~python
    torch.manual_seed(1111)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1702-1704 / 第 1702-1704 行

~~~~python
    def create_tensor(shape, dtype, init=True):
        init_type = cutlass_torch.TensorInitType.RANDOM
        init_config = cutlass_torch.RandomInitConfig(min_val=-2, max_val=2)
~~~~

**EN**: Defines `create_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `create_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1706-1722 / 第 1706-1722 行

~~~~python
        if init is False or init is None:
            init_type = cutlass_torch.TensorInitType.SKIP
            init_config = None
        elif isinstance(init, int) or isinstance(init, float):
            init_type = cutlass_torch.TensorInitType.SCALAR
            init_config = cutlass_torch.ScalarInitConfig(value=init)
        elif isinstance(init, tuple) or isinstance(init, list):
            if len(init) == 2:
                init_type = cutlass_torch.TensorInitType.RANDOM
                init_config = cutlass_torch.RandomInitConfig(
                    min_val=init[0], max_val=init[1]
                )
            if len(init) == 3:
                init_type = cutlass_torch.TensorInitType.GAUSSIAN
                init_config = cutlass_torch.RandomInitConfig(
                    mean=init[0], std=init[1], scale=init[2]
                )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1724-1730 / 第 1724-1730 行

~~~~python
        f32_torch_tensor = cutlass_torch.create_and_permute_torch_tensor(
            shape,
            torch.float32,
            permute_order=None,
            init_type=init_type,
            init_config=init_config,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1732-1737 / 第 1732-1737 行

~~~~python
        _, torch_tensor = cutlass_torch.cute_tensor_like(
            f32_torch_tensor,
            dtype,
            is_dynamic_layout=True,
            assumed_align=16,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1739-1739 / 第 1739-1739 行

~~~~python
        # Create dtype cute tensor with offset (gpu)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1740-1741 / 第 1740-1741 行

~~~~python
        cute_tensor = from_dlpack(torch_tensor, assumed_align=16)
        cute_tensor.element_type = dtype
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1743-1747 / 第 1743-1747 行

~~~~python
        return (
            f32_torch_tensor,
            cute_tensor,
            torch_tensor,
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 1749-1751 / 第 1749-1751 行

~~~~python
    qo_shape = (kv_splits, batches, heads_q, seqlen_q, headdim)
    kv_shape = (batches, heads_k, seqlen_k, headdim)
    scale_shape = (batches, heads_k, seqlen_k, fmha.scaledim)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1753-1769 / 第 1753-1769 行

~~~~python
    q_ref, q_cute, q_torch = create_tensor(qo_shape[1:], q_dtype, init=[-8, 7])
    k_ref, k_cute, k_torch = create_tensor(kv_shape, kv_dtype, init=[-8, 7])
    v_ref, v_cute, v_torch = create_tensor(kv_shape, kv_dtype, init=[-8, 7])
    k_scale_ref, k_scale_cute, k_scale_torch = create_tensor(
        scale_shape, q_dtype, init=[-2, 2]
    )
    v_scale_ref, v_scale_cute, v_scale_torch = create_tensor(
        scale_shape, q_dtype, init=[-2, 2]
    )
    _, o_cute, o_torch = create_tensor(qo_shape[1:], o_dtype, init=False)
    _, m_cute, m_torch = create_tensor(qo_shape[1:-1], acc_dtype, init=-math.inf)
    _, l_cute, l_torch = create_tensor(qo_shape[1:-1], acc_dtype, init=False)
    _, o_partial_cute, o_partial_torch = create_tensor(qo_shape, acc_dtype, init=0)
    _, m_partial_cute, m_partial_torch = create_tensor(
        qo_shape[:-1], acc_dtype, init=-math.inf
    )
    _, l_partial_cute, l_partial_torch = create_tensor(qo_shape[:-1], acc_dtype, init=0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1771-1773 / 第 1771-1773 行

~~~~python
    #
    # Compile
    #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1774-1795 / 第 1774-1795 行

~~~~python
    current_stream = cutlass_torch.default_stream()
    compiled_fmha = cute.compile(
        fmha,
        problem_shape,
        kv_splits,
        q_cute.iterator,
        k_cute.iterator,
        v_cute.iterator,
        k_scale_cute.iterator,
        v_scale_cute.iterator,
        o_cute.iterator,
        m_cute.iterator,
        l_cute.iterator,
        o_partial_cute.iterator,
        m_partial_cute.iterator,
        l_partial_cute.iterator,
        scale_qs,
        scale_o,
        current_stream,
        options="--opt-level 2",
    )
    print("Finished Compiling")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1797-1799 / 第 1797-1799 行

~~~~python
    #
    # Refcheck
    #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1800-1806 / 第 1800-1806 行

~~~~python
    def run_torch_fmha(
        q_ref, k_ref, v_ref, k_scale_ref, v_scale_ref, scale_qs=1.0, scale_o=1.0
    ):
        for i in range(0, headdim // block_scaledim):
            j = i * block_scaledim
            k_ref[..., j : j + block_scaledim] *= k_scale_ref[..., i : i + 1]
            v_ref[..., j : j + block_scaledim] *= v_scale_ref[..., i : i + 1]
~~~~

**EN**: Defines `run_torch_fmha`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `run_torch_fmha`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 1808-1820 / 第 1808-1820 行

~~~~python
        with sdpa_kernel(
            [SDPBackend.FLASH_ATTENTION, SDPBackend.MATH], set_priority=True
        ):
            o_ref = scaled_dot_product_attention(
                q_ref,
                k_ref,
                v_ref,
                attn_mask=None,
                dropout_p=0.0,
                scale=scale_qs,
                is_causal=False,
                enable_gqa=(heads_q != heads_k),
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1822-1822 / 第 1822-1822 行

~~~~python
        return o_ref * scale_o
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 1824-1851 / 第 1824-1851 行

~~~~python
    if not skip_ref_check:
        # Execute kernel once for reference checking
        print("Running...")
        compiled_fmha(
            problem_shape,
            kv_splits,
            q_cute.iterator,
            k_cute.iterator,
            v_cute.iterator,
            k_scale_cute.iterator,
            v_scale_cute.iterator,
            o_cute.iterator,
            m_cute.iterator,
            l_cute.iterator,
            o_partial_cute.iterator,
            m_partial_cute.iterator,
            l_partial_cute.iterator,
            scale_qs,
            scale_o,
            current_stream,
        )
        print("Verifying results...")
        o_ref = run_torch_fmha(
            q_ref, k_ref, v_ref, k_scale_ref, v_scale_ref, scale_qs, scale_o
        )
        torch.testing.assert_close(
            o_ref, o_torch.float().cpu(), atol=tolerance, rtol=1e-05
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1853-1880 / 第 1853-1880 行

~~~~python
    def generate_tensors():
        _, q_cute, _ = create_tensor(qo_shape[1:], q_dtype, init=[-8, 7])
        _, k_cute, _ = create_tensor(kv_shape, kv_dtype, init=[-8, 7])
        _, v_cute, _ = create_tensor(kv_shape, kv_dtype, init=[-8, 7])
        _, k_scale_cute, _ = create_tensor(scale_shape, q_dtype, init=[-8, 7])
        _, v_scale_cute, _ = create_tensor(scale_shape, q_dtype, init=[-8, 7])
        _, o_cute, _ = create_tensor(qo_shape[1:], o_dtype, init=False)
        _, m_cute, _ = create_tensor(qo_shape[1:-1], acc_dtype, init=-math.inf)
        _, l_cute, _ = create_tensor(qo_shape[1:-1], acc_dtype, init=False)
        _, o_partial_cute, _ = create_tensor(qo_shape, acc_dtype, init=0)
        _, m_partial_cute, _ = create_tensor(qo_shape[:-1], acc_dtype, init=-math.inf)
        _, l_partial_cute, _ = create_tensor(qo_shape[:-1], acc_dtype, init=0)
        args = testing.JitArguments(
            problem_shape,
            kv_splits,
            q_cute.iterator,
            k_cute.iterator,
            v_cute.iterator,
            k_scale_cute.iterator,
            v_scale_cute.iterator,
            o_cute.iterator,
            m_cute.iterator,
            l_cute.iterator,
            o_partial_cute.iterator,
            m_partial_cute.iterator,
            l_partial_cute.iterator,
            scale_qs,
            scale_o,
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1881-1897 / 第 1881-1897 行

~~~~python
            current_stream,
        )
        args.add_to_scope(
            [
                q_cute,
                k_cute,
                v_cute,
                k_scale_cute,
                v_scale_cute,
                o_cute,
                m_cute,
                l_cute,
                o_partial_cute,
                m_partial_cute,
                l_partial_cute,
            ]
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1899-1901 / 第 1899-1901 行

~~~~python
    #
    # Profile
    #
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 1902-1929 / 第 1902-1929 行

~~~~python
    workspace_count = 1
    if use_cold_l2:
        q_torch_effective = q_torch.values() if q_torch.is_nested else q_torch
        k_torch_effective = k_torch.values() if k_torch.is_nested else k_torch
        v_torch_effective = v_torch.values() if v_torch.is_nested else v_torch
        k_scale_torch_effective = (
            k_scale_torch.values() if k_scale_torch.is_nested else k_scale_torch
        )
        v_scale_torch_effective = (
            v_scale_torch.values() if v_scale_torch.is_nested else v_scale_torch
        )
        o_torch_effective = o_torch.values() if o_torch.is_nested else o_torch
        m_torch_effective = m_torch.values() if m_torch.is_nested else m_torch
        l_torch_effective = l_torch.values() if l_torch.is_nested else l_torch
        o_partial_torch_effective = (
            o_partial_torch.values() if o_partial_torch.is_nested else o_partial_torch
        )
        m_partial_torch_effective = (
            m_partial_torch.values() if m_partial_torch.is_nested else m_partial_torch
        )
        l_partial_torch_effective = (
            l_partial_torch.values() if l_partial_torch.is_nested else l_partial_torch
        )
        one_workspace_bytes = (
            q_torch_effective.numel() * q_torch_effective.element_size()
            + k_torch_effective.numel() * k_torch_effective.element_size()
            + v_torch_effective.numel() * v_torch_effective.element_size()
            + k_scale_torch_effective.numel() * k_scale_torch_effective.element_size()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1930-1943 / 第 1930-1943 行

~~~~python
            + v_scale_torch_effective.numel() * v_scale_torch_effective.element_size()
            + o_torch_effective.numel() * o_torch_effective.element_size()
            + m_torch_effective.numel() * m_torch_effective.element_size()
            + l_torch_effective.numel() * l_torch_effective.element_size()
            + o_partial_torch_effective.numel()
            * o_partial_torch_effective.element_size()
            + m_partial_torch_effective.numel()
            * m_partial_torch_effective.element_size()
            + l_partial_torch_effective.numel()
            * l_partial_torch_effective.element_size()
        )
        workspace_count = testing.get_workspace_count(
            one_workspace_bytes, warmup_iterations, iterations
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1945-1955 / 第 1945-1955 行

~~~~python
    exec_time = 0.0
    if iterations > 0:
        exec_time = testing.benchmark(
            compiled_fmha,
            workspace_generator=generate_tensors,
            workspace_count=workspace_count,
            stream=current_stream,
            warmup_iterations=warmup_iterations,
            iterations=iterations,
        )
        print(exec_time)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1957-1957 / 第 1957-1957 行

~~~~python
    return exec_time  # Return execution time in microseconds
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1960-1960 / 第 1960-1960 行

~~~~python
if __name__ == "__main__":
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1962-1968 / 第 1962-1968 行

~~~~python
    def parse_comma_separated_ints(s: str):
        try:
            return tuple(cutlass.int(x.strip()) for x in s.split(","))
        except ValueError:
            raise argparse.ArgumentTypeError(
                "Invalid format. Expected comma-separated integers."
            )
~~~~

**EN**: Defines `parse_comma_separated_ints`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 定义 `parse_comma_separated_ints`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 1970-1970 / 第 1970-1970 行

~~~~python
    parser = argparse.ArgumentParser(description="Example of FMHA on Blackwell.")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1972-1974 / 第 1972-1974 行

~~~~python
    parser.add_argument(
        "--batches", "--batch", "--b", type=int, default=1, help="batch size"
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 1976-1984 / 第 1976-1984 行

~~~~python
    parser.add_argument(
        "--seqlen",
        "--seqlen_k",
        "--seq",
        "--s",
        type=int,
        default=1024,
        help="key/value sequence length",
    )
~~~~

**EN**: Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1986-1986 / 第 1986-1986 行

~~~~python
    parser.add_argument("--heads_q", "--h_q", type=int, default=32, help="query heads")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1988-1990 / 第 1988-1990 行

~~~~python
    parser.add_argument(
        "--heads_k", "--h_k", type=int, default=4, help="key/value heads"
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1992-1994 / 第 1992-1994 行

~~~~python
    parser.add_argument(
        "--headdim", "--d", type=int, default=512, help="head dimmension"
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 1996-2002 / 第 1996-2002 行

~~~~python
    parser.add_argument(
        "--block_scaledim",
        "--bs",
        type=int,
        default=512,
        help="headdim per scale factor",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2004-2010 / 第 2004-2010 行

~~~~python
    parser.add_argument(
        "--kv_splits",
        "--splits",
        type=int,
        default=0,
        help="threadblocks per sequence",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2012-2017 / 第 2012-2017 行

~~~~python
    parser.add_argument(
        "--q_dtype",
        type=cutlass.dtype,
        default=cutlass.BFloat16,
        help="query data type",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2019-2024 / 第 2019-2024 行

~~~~python
    parser.add_argument(
        "--kv_dtype",
        type=cutlass.dtype,
        default=cutlass.Int8,
        help="key/value data type",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2026-2031 / 第 2026-2031 行

~~~~python
    parser.add_argument(
        "--o_dtype",
        type=cutlass.dtype,
        default=cutlass.BFloat16,
        help="output data type",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2033-2038 / 第 2033-2038 行

~~~~python
    parser.add_argument(
        "--acc_dtype",
        type=cutlass.dtype,
        default=cutlass.Float32,
        help="accumulator/reduction data type",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2040-2042 / 第 2040-2042 行

~~~~python
    parser.add_argument(
        "--tolerance", type=float, default=1e-01, help="Tolerance for validation"
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2044-2049 / 第 2044-2049 行

~~~~python
    parser.add_argument(
        "--scale_q",
        type=float,
        default=1.0,
        help="query scale factor",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 2051-2056 / 第 2051-2056 行

~~~~python
    parser.add_argument(
        "--scale_o",
        type=float,
        default=1.0,
        help="output scale factor",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2058-2064 / 第 2058-2064 行

~~~~python
    parser.add_argument(
        "--scale_s",
        "--scale",
        type=float,
        default=0.0,
        help="score (Q*K) scale factor; if zero, defaults to 1/sqrt(D)",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 2066-2071 / 第 2066-2071 行

~~~~python
    parser.add_argument(
        "--warmup_iterations",
        type=int,
        default=0,
        help="Number of iterations for warmup",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2073-2078 / 第 2073-2078 行

~~~~python
    parser.add_argument(
        "--iterations",
        type=int,
        default=0,
        help="Number of iterations after warmup",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2080-2084 / 第 2080-2084 行

~~~~python
    parser.add_argument(
        "--skip_ref_check",
        action="store_true",
        help="Skip reference check",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 2086-2091 / 第 2086-2091 行

~~~~python
    parser.add_argument(
        "--use_cold_l2",
        action="store_true",
        default=False,
        help="Use circular buffer tensor sets to ensure L2 cold cache",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2093-2094 / 第 2093-2094 行

~~~~python
    kwargs = vars(parser.parse_args())
    run(**kwargs)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 2096-2096 / 第 2096-2096 行

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
- Specialized sequence or attention dataflow / 专门化的序列/注意力数据流
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Multistage pipeline and synchronization / 多级流水线与同步

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `math` — used by this example / 供该示例使用
- `typing.Type` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `functools.partial` — supplies small Python helpers such as partial application / 提供 partial 等 Python 辅助能力
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `torch.nn.functional.scaled_dot_product_attention` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `torch.nn.attention.SDPBackend` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `torch.nn.attention.sdpa_kernel` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.nvgpu.tcgen05` — provides Tensor Core/TMEM helpers for Blackwell-style kernels / 提供面向 Blackwell 风格内核的 Tensor Core/TMEM 辅助工具
- `cutlass.cute.nvgpu.OperandMajorMode` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.pipeline` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils.blackwell_helpers` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `cutlass.cute.typing` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
