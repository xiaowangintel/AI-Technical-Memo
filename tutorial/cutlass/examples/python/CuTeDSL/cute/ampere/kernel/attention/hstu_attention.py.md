# hstu_attention.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/ampere/kernel/attention/hstu_attention.py`  
**Purpose / 用途**: Kernel example implementing hstu attention with CuTeDSL. / 这是一个使用 CuTeDSL 实现 hstu attention 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-38 / 第 29-38 行

~~~~python
from typing import Type
import argparse

import cuda.bindings.driver as cuda
import cutlass
import cutlass.cute as cute
from cutlass.cute.runtime import from_dlpack
from cutlass._mlir.dialects import llvm
import cutlass.pipeline as pipeline
import cutlass.utils as utils
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 40-62 / 第 40-62 行

~~~~python
"""
A HSTU attention forward pass example for NVIDIA Ampere SM80 architecture using Cute DSL, based on the example of flash_attention_v2 for Ampere.

The example showcases an implementation of HSTU attention(https://arxiv.org/abs/2402.17152) within generative recommender system. It utilize the formula: `mask(silu(q@k+rab))@v`. The implementation includes the following features:
- efficient fast sigmoid implementation
- block rasterization to improve L2 cache hit rate.
- The correct approach to verify the results of the HSTU attention with a Pytorch implementation.

To run this example:

.. code-block:: bash

    python examples/ampere/hstu_attention.py --batch_size 4 --seqlen_q 8192 --seqlen_kv 8192 --num_head 4 --head_dim 128 --m_block_size 128 --n_block_size 64 --is_causal --perf_test

The above example tests the performance of HSTU attention with batch size 4, sequence length 8192, 4 attention heads, and head dimension 128. The m_block_size is 128, and n_block_size is 64. The causal masking is enabled.

There are some constraints for this example:
* Only Float16 and BFloat16 are supported.
* The contiguous dimension of each tensor must be at least 16 bytes aligned.
* The values of `m_block_size`, `n_block_size`, and `head_dim` must be selected to stay within shared memory capacity limits.
* `m_block_size * 2` must be divisible by `num_threads`, otherwise the kernel will not be able to get the correct result.
* "seqlen_kv should be greater or equal to seqlen_q.
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 65-92 / 第 65-92 行

~~~~python
class HSTUAttentionForwardAmpere(object):
    def __init__(
        self,
        dtype,
        batch_size,
        seqlen_q,
        seqlen_kv,
        num_head,
        head_dim,
        m_block_size=128,
        n_block_size=128,
        num_threads=128,
        enable_fast_sigmoid=False,
        enable_block_rasterization=False,
        is_causal=False,
    ):
        self._dtype = dtype
        self._batch_size = batch_size
        self._seqlen_q = seqlen_q
        self._seqlen_kv = seqlen_kv
        self._num_head = num_head
        self._head_dim = head_dim
        self._m_block_size = m_block_size
        self._n_block_size = n_block_size
        # padded head_dim to 32 for cta tile.
        self._head_dim_padded = (head_dim + 31) // 32 * 32
        self._num_threads = num_threads
        self._enable_fast_sigmoid = enable_fast_sigmoid
~~~~

**EN**: Defines `HSTUAttentionForwardAmpere`, a reusable Python class that packages configuration and behavior for this example. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `HSTUAttentionForwardAmpere`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 93-109 / 第 93-109 行

~~~~python
        self._enable_block_rasterization = enable_block_rasterization
        self._is_causal = is_causal
        assert self._dtype == cutlass.Float16 or self._dtype == cutlass.BFloat16, (
            "Only Float16 or BFloat16 is supported"
        )
        assert self._head_dim % 8 == 0, "head dim should be multiply of 8"
        assert self._num_threads % 32 == 0, "num_threads should be multiply of 32"
        assert self._m_block_size * self._head_dim_padded // self._num_threads >= 8, (
            "Small m_block_size and too many threads"
        )
        assert self._n_block_size * self._head_dim_padded // self._num_threads >= 8, (
            "Small n_block_size and too many threads"
        )
        assert seqlen_kv >= seqlen_q, "seqlen_kv should be greater or equal to seqlen_q"
        self.cta_sync_barrier = pipeline.NamedBarrier(
            barrier_id=1, num_threads=num_threads
        )
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 111-138 / 第 111-138 行

~~~~python
    @cute.jit
    def __call__(
        self,
        mQ: cute.Tensor,
        mK: cute.Tensor,
        mV: cute.Tensor,
        mO: cute.Tensor,
        mRAB: cute.Tensor,
        stream: cuda.CUstream,
    ):
        """Configures and launches the HSTU attention kernel.
        mQ/mK/mV/mO/mRAB has same data types(supports fp16 and bf16).
        mQ has layout: (batch_size, seqlen_q, num_head, head_dim):(seqlen_q * num_head * head_dim, num_head * head_dim, head_dim, 1)
        mK/mV/mO has same layout: (batch_size, seqlen_kv, num_head, head_dim):(seqlen_kv * num_head * head_dim, num_head * head_dim, head_dim, 1)
        mRAB has layout: (batch_size, num_head, seqlen_q, seqlen_kv):(seqlen_q*seqlen_kv*num_head, seqlen_q*seqlen_kv, seqlen_kv, 1)

        Prepares the shared memory layout, tiled copy atoms, tiled mma and shared memory storage.
        Then launches the kernel function with the prepared parameters.

        :param mQ: query tensor
        :type mQ: cute.Tensor
        :param mK: key tensor
        :type mK: cute.Tensor
        :param mV: value tensor
        :type mV: cute.Tensor
        :param mO: output tensor
        :type mO: cute.Tensor
        :param mRAB: RAB tensor
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 139-166 / 第 139-166 行

~~~~python
        :type mRAB: cute.Tensor
        """
        # ///////////////////////////////////////////////////////////////////////////////
        # Shared memory layout: Q/K/V/RAB
        # ///////////////////////////////////////////////////////////////////////////////
        smem_k_block_size = 64 if self._head_dim_padded % 64 == 0 else 32
        swizzle_bits = 3 if smem_k_block_size == 64 else 2
        sQ_layout_atom = cute.make_composed_layout(
            cute.make_swizzle(swizzle_bits, 4, 3),
            0,
            cute.make_layout((8, smem_k_block_size), stride=(smem_k_block_size, 1)),
        )
        sQ_layout = cute.tile_to_shape(
            sQ_layout_atom,
            (self._m_block_size, self._head_dim_padded),
            (0, 1),
        )
        sKV_layout_atom = sQ_layout_atom
        sKV_layout = cute.tile_to_shape(
            sKV_layout_atom,
            (self._n_block_size, self._head_dim_padded),
            (0, 1),
        )
        sRAB_layout_atom = sQ_layout_atom
        sRAB_layout = cute.tile_to_shape(
            sRAB_layout_atom, (self._m_block_size, self._n_block_size), (0, 1)
        )
        sO_layout = sQ_layout
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 168-181 / 第 168-181 行

~~~~python
        @cute.struct
        class SharedStorage:
            sQ: cute.struct.Align[
                cute.struct.MemRange[self._dtype, cute.cosize(sQ_layout)], 1024
            ]
            sK: cute.struct.Align[
                cute.struct.MemRange[self._dtype, cute.cosize(sKV_layout)], 1024
            ]
            sV: cute.struct.Align[
                cute.struct.MemRange[self._dtype, cute.cosize(sKV_layout)], 1024
            ]
            sRAB: cute.struct.Align[
                cute.struct.MemRange[self._dtype, cute.cosize(sRAB_layout)], 1024
            ]
~~~~

**EN**: Defines `SharedStorage`, a reusable Python class that packages configuration and behavior for this example. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `SharedStorage`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 183-185 / 第 183-185 行

~~~~python
        assert SharedStorage.size_in_bytes() < utils.get_smem_capacity_in_bytes(
            "sm_80"
        ), "insufficient shared memory"
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 187-190 / 第 187-190 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # GMEM Tiled copy:
        # ///////////////////////////////////////////////////////////////////////////////
        # Thread layouts for copies
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 191-212 / 第 191-212 行

~~~~python
        universal_copy_bits = 128
        async_copy_elems = universal_copy_bits // self._dtype.width
        # atom_async_copy: async copy atom for QKV load
        atom_async_copy = cute.make_copy_atom(
            cute.nvgpu.cpasync.CopyG2SOp(),
            self._dtype,
            num_bits_per_copy=universal_copy_bits,
        )
        # atom_universal_copy: universal copy atom for O store
        atom_universal_copy = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(),
            self._dtype,
            num_bits_per_copy=universal_copy_bits,
        )
        # tQKV_layout: thread layout for QKV load
        tQKV_shape_dim_1 = sQ_layout_atom.outer.shape[1] // async_copy_elems
        tQKV_layout = cute.make_layout(
            (self._num_threads // tQKV_shape_dim_1, tQKV_shape_dim_1),
            stride=(tQKV_shape_dim_1, 1),
        )
        # tO_layout: thread layout for O store
        tO_layout = tQKV_layout
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 214-214 / 第 214-214 行

~~~~python
        # Value layouts for copies
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 215-216 / 第 215-216 行

~~~~python
        vQKV_layout = cute.make_layout((1, async_copy_elems))
        vO_layout = vQKV_layout
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 218-218 / 第 218-218 行

~~~~python
        # gmem_tiled_copy_QKV: tiled copy for QKV load
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 219-221 / 第 219-221 行

~~~~python
        gmem_tiled_copy_QKV = cute.make_tiled_copy_tv(
            atom_async_copy, tQKV_layout, vQKV_layout
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 223-223 / 第 223-223 行

~~~~python
        # gmem_tiled_copy_O: tiled copy for O store
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 224-226 / 第 224-226 行

~~~~python
        gmem_tiled_copy_O = cute.make_tiled_copy_tv(
            atom_universal_copy, tO_layout, vO_layout
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 228-230 / 第 228-230 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Tiled mma
        # ///////////////////////////////////////////////////////////////////////////////
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 231-235 / 第 231-235 行

~~~~python
        tiled_mma = cute.make_tiled_mma(
            cute.nvgpu.warp.MmaF16BF16Op(self._dtype, cutlass.Float32, (16, 8, 16)),
            (self._num_threads // 32, 1, 1),
            permutation_mnk=(self._num_threads // 32 * 16, 16, 16),
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 237-237 / 第 237-237 行

~~~~python
        # block rasterization
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 238-249 / 第 238-249 行

~~~~python
        if cutlass.const_expr(self._enable_block_rasterization):
            grid_dim = (
                self._batch_size,
                self._num_head,
                cute.ceil_div(mQ.shape[1], self._m_block_size),
            )
        else:
            grid_dim = (
                cute.ceil_div(mQ.shape[1], self._m_block_size),
                self._batch_size,
                self._num_head,
            )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 251-269 / 第 251-269 行

~~~~python
        self.kernel(
            mQ,
            mK,
            mV,
            mO,
            mRAB,
            sQ_layout,
            sKV_layout,
            sRAB_layout,
            sO_layout,
            gmem_tiled_copy_QKV,
            gmem_tiled_copy_O,
            tiled_mma,
            SharedStorage,
        ).launch(
            grid=grid_dim,
            block=[self._num_threads, 1, 1],
            stream=stream,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 271-298 / 第 271-298 行

~~~~python
    @cute.kernel
    def kernel(
        self,
        mQ: cute.Tensor,
        mK: cute.Tensor,
        mV: cute.Tensor,
        mO: cute.Tensor,
        mRAB: cute.Tensor,
        sQ_layout: cute.ComposedLayout,
        sKV_layout: cute.ComposedLayout,
        sRAB_layout: cute.ComposedLayout,
        sO_layout: cute.ComposedLayout,
        gmem_tiled_copy_QKV: cute.TiledCopy,
        gmem_tiled_copy_O: cute.TiledCopy,
        tiled_mma: cute.TiledMma,
        SharedStorage: cutlass.Constexpr,
    ):
        """Kernel function for HSTU attention.

        :param mQ: query tensor
        :type mQ: cute.Tensor
        :param mK: key tensor
        :type mK: cute.Tensor
        :param mV: value tensor
        :type mV: cute.Tensor
        :param mO: output tensor
        :type mO: cute.Tensor
        :param mRAB: RAB(Relative Attention Bias) tensor
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 299-318 / 第 299-318 行

~~~~python
        :type mRAB: cute.Tensor
        :param sQ_layout: shared memory layout for Q
        :type sQ_layout: cute.ComposedLayout
        :param sKV_layout: shared memory layout for K/V
        :type sKV_layout: cute.ComposedLayout
        :param sRAB_layout: shared memory layout for RAB
        :type sRAB_layout: cute.ComposedLayout
        :param sO_layout: shared memory layout for O
        :type sO_layout: cute.ComposedLayout
        :param gmem_tiled_copy_QKV: tiled copy for QKV load
        :type gmem_tiled_copy_QKV: cute.TiledCopy
        :param gmem_tiled_copy_O: tiled copy for O store
        :type gmem_tiled_copy_O: cute.TiledCopy
        :param tiled_mma: tiled mma
        :type tiled_mma: cute.TiledMma
        :param SharedStorage: shared storage
        :type SharedStorage: cutlass.Constexpr
        """
        # Thread index, block index
        tidx, _, _ = cute.arch.thread_idx()
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 320-323 / 第 320-323 行

~~~~python
        if cutlass.const_expr(self._enable_block_rasterization):
            batch_size, num_head, m_block = cute.arch.block_idx()
        else:
            m_block, batch_size, num_head = cute.arch.block_idx()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 325-325 / 第 325-325 行

~~~~python
        # reverse the m_block index
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 326-326 / 第 326-326 行

~~~~python
        m_block = cute.ceil_div(mQ.shape[1], self._m_block_size) - m_block - 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 328-337 / 第 328-337 行

~~~~python
        if cutlass.const_expr(self._is_causal):
            n_block = (
                cute.ceil_div(
                    min((m_block + 1) * self._m_block_size, mK.shape[1]),
                    self._n_block_size,
                )
                - 1
            )  # for causal case, only process the first n_block tiles
        else:
            n_block = cute.ceil_div(mK.shape[1], self._n_block_size) - 1
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 339-342 / 第 339-342 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Get the appropriate tiles for this thread block.
        # ///////////////////////////////////////////////////////////////////////////////
        # (m_block_size, head_dim)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 343-365 / 第 343-365 行

~~~~python
        gQ = cute.local_tile(
            mQ[batch_size, None, num_head, None],
            (self._m_block_size, self._head_dim_padded),
            (m_block, 0),
        )
        # (n_block_size, head_dim, n_block)
        gK = cute.local_tile(
            mK[batch_size, None, num_head, None],
            (self._n_block_size, self._head_dim_padded),
            (None, 0),
        )
        # (n_block_size, head_dim, n_block)
        gV = cute.local_tile(
            mV[batch_size, None, num_head, None],
            (self._n_block_size, self._head_dim_padded),
            (None, 0),
        )
        # (m_block_size, n_block_size)
        gRAB = cute.local_tile(
            mRAB[batch_size, num_head, None, None],
            (self._m_block_size, self._n_block_size),
            (m_block, None),
        )
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 367-369 / 第 367-369 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Get shared memory buffer
        # ///////////////////////////////////////////////////////////////////////////////
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 370-375 / 第 370-375 行

~~~~python
        smem = cutlass.utils.SmemAllocator()
        storage = smem.allocate(SharedStorage)
        sQ = storage.sQ.get_tensor(sQ_layout)
        sK = storage.sK.get_tensor(sKV_layout)
        sV = storage.sV.get_tensor(sKV_layout)
        sRAB = storage.sRAB.get_tensor(sRAB_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 377-377 / 第 377-377 行

~~~~python
        # Transpose view of V to tensor with layout (head_dim, n_block_size) for tiled mma
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 378-384 / 第 378-384 行

~~~~python
        sVt = cute.composition(
            sV,
            cute.make_layout(
                (self._head_dim_padded, self._n_block_size),
                stride=(self._n_block_size, 1),
            ),
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 386-398 / 第 386-398 行

~~~~python
        gmem_thr_copy_QKV = gmem_tiled_copy_QKV.get_slice(tidx)
        # (CPY_Atom, CPY_M, CPY_K)
        tQgQ = gmem_thr_copy_QKV.partition_S(gQ)
        tQsQ = gmem_thr_copy_QKV.partition_D(sQ)
        # (CPY_Atom, CPY_N, CPY_K, n_block)
        tKgK = gmem_thr_copy_QKV.partition_S(gK)
        tKsK = gmem_thr_copy_QKV.partition_D(sK)
        # (CPY_Atom, CPY_N, CPY_K, n_block)
        tVgV = gmem_thr_copy_QKV.partition_S(gV)
        tVsV = gmem_thr_copy_QKV.partition_D(sV)
        # (CPY_Atom, CPY_M, CPY_N, n_block)
        tRABgRAB = gmem_tiled_copy_QKV.get_slice(tidx).partition_S(gRAB)
        tRabsRAB = gmem_tiled_copy_QKV.get_slice(tidx).partition_D(sRAB)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 400-402 / 第 400-402 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Tile MMA compute thread partitions and allocate accumulators
        # ///////////////////////////////////////////////////////////////////////////////
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 403-411 / 第 403-411 行

~~~~python
        thr_mma = tiled_mma.get_slice(tidx)
        tSrQ = thr_mma.make_fragment_A(thr_mma.partition_A(sQ))
        tSrK = thr_mma.make_fragment_B(thr_mma.partition_B(sK))
        tOrVt = thr_mma.make_fragment_B(thr_mma.partition_B(sVt))
        acc_shape_O = thr_mma.partition_shape_C(
            (self._m_block_size, self._head_dim_padded)
        )
        acc_O = cute.make_rmem_tensor(acc_shape_O, cutlass.Float32)
        acc_O.fill(0.0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 413-415 / 第 413-415 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Smem copy atom tiling
        # ///////////////////////////////////////////////////////////////////////////////
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 416-435 / 第 416-435 行

~~~~python
        smem_copy_atom_Q = cute.make_copy_atom(
            cute.nvgpu.warp.LdMatrix8x8x16bOp(transpose=False, num_matrices=4),
            self._dtype,
        )
        smem_copy_atom_K = cute.make_copy_atom(
            cute.nvgpu.warp.LdMatrix8x8x16bOp(transpose=False, num_matrices=4),
            self._dtype,
        )
        smem_copy_atom_V = cute.make_copy_atom(
            cute.nvgpu.warp.LdMatrix8x8x16bOp(transpose=True, num_matrices=4),
            self._dtype,
        )
        smem_copy_atom_RAB = cute.make_copy_atom(
            cute.nvgpu.warp.LdMatrix8x8x16bOp(transpose=False, num_matrices=4),
            self._dtype,
        )
        smem_tiled_copy_Q = cute.make_tiled_copy_A(smem_copy_atom_Q, tiled_mma)
        smem_tiled_copy_K = cute.make_tiled_copy_B(smem_copy_atom_K, tiled_mma)
        smem_tiled_copy_V = cute.make_tiled_copy_B(smem_copy_atom_V, tiled_mma)
        smem_tiled_copy_RAB = cute.make_tiled_copy_C(smem_copy_atom_RAB, tiled_mma)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 437-440 / 第 437-440 行

~~~~python
        smem_thr_copy_Q = smem_tiled_copy_Q.get_slice(tidx)
        smem_thr_copy_K = smem_tiled_copy_K.get_slice(tidx)
        smem_thr_copy_V = smem_tiled_copy_V.get_slice(tidx)
        smem_thr_copy_RAB = smem_tiled_copy_RAB.get_slice(tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 442-448 / 第 442-448 行

~~~~python
        tSsQ = smem_thr_copy_Q.partition_S(sQ)
        tSrQ_copy_view = smem_thr_copy_Q.retile(tSrQ)
        tSsK = smem_thr_copy_K.partition_S(sK)
        tSrK_copy_view = smem_thr_copy_K.retile(tSrK)
        tOsVt = smem_thr_copy_V.partition_S(sVt)
        tOrVt_copy_view = smem_thr_copy_V.retile(tOrVt)
        tSsRAB = smem_thr_copy_RAB.partition_S(sRAB)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 450-454 / 第 450-454 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Predicate: Mark indices that need to copy when problem_shape isn't a multiple
        # of tile_shape
        # ///////////////////////////////////////////////////////////////////////////////
        # Construct identity layout for Q, KV and RAB
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 455-457 / 第 455-457 行

~~~~python
        mcQ = cute.make_identity_tensor(mQ.layout.shape)
        mcKV = cute.make_identity_tensor(mK.layout.shape)
        mcRAB = cute.make_identity_tensor(mRAB.layout.shape)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 459-473 / 第 459-473 行

~~~~python
        cQ = cute.local_tile(
            mcQ[batch_size, None, num_head, None],
            (self._m_block_size, self._head_dim_padded),
            (m_block, 0),
        )
        cKV = cute.local_tile(
            mcKV[batch_size, None, num_head, None],
            (self._n_block_size, self._head_dim_padded),
            (n_block, 0),
        )
        cRAB = cute.local_tile(
            mcRAB[batch_size, num_head, None, None],
            (self._m_block_size, self._n_block_size),
            (m_block, None),
        )
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 475-475 / 第 475-475 行

~~~~python
        # Repeat the partitioning with identity layouts
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 476-478 / 第 476-478 行

~~~~python
        tQcQ = gmem_thr_copy_QKV.partition_S(cQ)
        tKVcKV = gmem_thr_copy_QKV.partition_S(cKV)
        tRABcRAB = gmem_thr_copy_QKV.partition_S(cRAB)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 480-501 / 第 480-501 行

~~~~python
        tQpQ = cute.make_rmem_tensor(
            cute.make_layout(
                (
                    tQsQ.shape[0][1],
                    cute.size(tQsQ, mode=[1]),
                    cute.size(tQsQ, mode=[2]),
                ),
                stride=(cute.size(tQsQ, mode=[2]), 0, 1),
            ),
            cutlass.Boolean,
        )
        tKVpKV = cute.make_rmem_tensor(
            cute.make_layout(
                (
                    tKsK.shape[0][1],
                    cute.size(tKsK, mode=[1]),
                    cute.size(tKsK, mode=[2]),
                ),
                stride=(cute.size(tKsK, mode=[2]), 0, 1),
            ),
            cutlass.Boolean,
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 503-503 / 第 503-503 行

~~~~python
        # Set predicates for head_dim bounds, seqlen_q/k/v bounds is processed at the first tile.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 504-513 / 第 504-513 行

~~~~python
        for rest_v in cutlass.range_constexpr(tQpQ.shape[0]):
            for rest_k in cutlass.range_constexpr(tQpQ.shape[2]):
                tQpQ[rest_v, 0, rest_k] = cute.elem_less(
                    tQcQ[(0, rest_v), 0, rest_k][3], mQ.layout.shape[3]
                )
        for rest_v in cutlass.range_constexpr(tKVpKV.shape[0]):
            for rest_k in cutlass.range_constexpr(tKVpKV.shape[2]):
                tKVpKV[rest_v, 0, rest_k] = cute.elem_less(
                    tKVcKV[(0, rest_v), 0, rest_k][3], mK.layout.shape[3]
                )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 515-518 / 第 515-518 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Prefetch Prologue
        # ///////////////////////////////////////////////////////////////////////////////
        # Start async loads of the last mn-tile, where we take care of the mn residue
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 519-529 / 第 519-529 行

~~~~python
        for m in cutlass.range_constexpr(cute.size(tQsQ.shape[1])):
            if cute.elem_less(tQcQ[0, m, 0][1], mQ.layout.shape[1]):
                cute.copy(
                    gmem_tiled_copy_QKV,
                    tQgQ[None, m, None],
                    tQsQ[None, m, None],
                    pred=tQpQ[None, m, None],
                )
            else:
                # Clear the smem tiles to account for predicated off loads
                tQsQ[None, m, None].fill(0)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 531-541 / 第 531-541 行

~~~~python
        for n in cutlass.range_constexpr(cute.size(tKsK.shape[1])):
            if cute.elem_less(tKVcKV[0, n, 0][1], mK.layout.shape[1]):
                cute.copy(
                    gmem_tiled_copy_QKV,
                    tKgK[None, n, None, n_block],
                    tKsK[None, n, None],
                    pred=tKVpKV[None, n, None],
                )
            else:
                # Clear the smem tiles to account for predicated off loads
                tKsK[None, n, None].fill(0)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 543-558 / 第 543-558 行

~~~~python
        for m in cutlass.range_constexpr(cute.size(tRABcRAB.shape[1])):
            for n in cutlass.range_constexpr(cute.size(tRABcRAB.shape[2])):
                if cute.elem_less(
                    tRABcRAB[0, m, n, n_block][1], mRAB.layout.shape[2]
                ) and cute.elem_less(
                    tRABcRAB[0, m, n, n_block][2], mRAB.layout.shape[3]
                ):
                    cute.copy(
                        gmem_tiled_copy_QKV,
                        tRABgRAB[None, m, n, n_block],
                        tRabsRAB[None, m, n],
                    )
                else:
                    # Clear the smem tiles to account for predicated off loads
                    tRabsRAB[None, m, n].fill(0)
        cute.arch.cp_async_commit_group()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 560-562 / 第 560-562 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Mainloop
        # ///////////////////////////////////////////////////////////////////////////////
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 563-566 / 第 563-566 行

~~~~python
        for n_block_idx in range(n_block, -1, -1):
            # wait for smem tile QK before mma caculation for S
            cute.arch.cp_async_wait_group(0)
            self.cta_sync_barrier.arrive_and_wait()
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 568-586 / 第 568-586 行

~~~~python
            if n_block_idx == n_block:
                for n in cutlass.range_constexpr(cute.size(tVsV.shape[1])):
                    if cute.elem_less(tKVcKV[0, n, 0][1], mV.layout.shape[1]):
                        cute.copy(
                            gmem_tiled_copy_QKV,
                            tVgV[None, n, None, n_block_idx],
                            tVsV[None, n, None],
                            pred=tKVpKV[None, n, None],
                        )
                    else:
                        tVsV[None, n, None].fill(0)
            else:
                cute.copy(
                    gmem_tiled_copy_QKV,
                    tVgV[None, None, None, n_block_idx],
                    tVsV[None, None, None],
                    pred=tKVpKV[None, None, None],
                )
            cute.arch.cp_async_commit_group()
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 588-591 / 第 588-591 行

~~~~python
            acc_shape_S = thr_mma.partition_shape_C(
                (self._m_block_size, self._n_block_size)
            )
            acc_S = cute.make_rmem_tensor(acc_shape_S, cutlass.Float32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 593-603 / 第 593-603 行

~~~~python
            rRAB_shape_S = thr_mma.partition_shape_C(
                (self._m_block_size, self._n_block_size)
            )
            rRAB = cute.make_rmem_tensor(rRAB_shape_S, self._dtype)
            tSrRAB_copy_view = smem_thr_copy_RAB.retile(rRAB)
            cute.copy(
                smem_tiled_copy_RAB,
                tSsRAB[None, None, None],
                tSrRAB_copy_view[None, None, None],
            )
            acc_S.store(rRAB.load().to(cutlass.Float32))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 605-608 / 第 605-608 行

~~~~python
            # ///////////////////////////////////////////////////////////////////////////////
            # S gemm calculation
            # ///////////////////////////////////////////////////////////////////////////////
            # ldmatrix first QK k-block for mma
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 609-618 / 第 609-618 行

~~~~python
            cute.copy(
                smem_tiled_copy_Q,
                tSsQ[None, None, 0],
                tSrQ_copy_view[None, None, 0],
            )
            cute.copy(
                smem_tiled_copy_K,
                tSsK[None, None, 0],
                tSrK_copy_view[None, None, 0],
            )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 619-639 / 第 619-639 行

~~~~python
            for k in cutlass.range_constexpr(0, cute.size(tSsQ.shape[2])):
                # ldmatrix next QK k-block for mma
                if k < cute.size(tSsQ.shape[2]) - 1:
                    cute.copy(
                        smem_tiled_copy_Q,
                        tSsQ[None, None, k + 1],
                        tSrQ_copy_view[None, None, k + 1],
                    )
                    cute.copy(
                        smem_tiled_copy_K,
                        tSsK[None, None, k + 1],
                        tSrK_copy_view[None, None, k + 1],
                    )
                # mma for S=Q@K
                cute.gemm(
                    tiled_mma,
                    acc_S,
                    tSrQ[None, None, k],
                    tSrK[None, None, k],
                    acc_S,
                )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 641-641 / 第 641-641 行

~~~~python
            # wait for smem tile V for O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 642-643 / 第 642-643 行

~~~~python
            cute.arch.cp_async_wait_group(0)
            self.cta_sync_barrier.arrive_and_wait()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 645-663 / 第 645-663 行

~~~~python
            if n_block_idx > 0:
                cute.copy(
                    gmem_tiled_copy_QKV,
                    tKgK[None, None, None, n_block_idx - 1],
                    tKsK[None, None, None],
                    pred=tKVpKV[None, None, None],
                )
                # m residue handling for RAB
                for m in cutlass.range_constexpr(cute.size(tRABcRAB.shape[1])):
                    if cute.elem_less(
                        tRABcRAB[0, m, 0, n_block_idx - 1][1], mRAB.layout.shape[2]
                    ):
                        cute.copy(
                            gmem_tiled_copy_QKV,
                            tRABgRAB[None, m, None, n_block_idx - 1],
                            tRabsRAB[None, m, None],
                        )
                    else:
                        tRabsRAB[None, m, None].fill(0)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 665-665 / 第 665-665 行

~~~~python
                cute.arch.cp_async_commit_group()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 667-669 / 第 667-669 行

~~~~python
            # ///////////////////////////////////////////////////////////////////////////////
            # silu activation
            # ///////////////////////////////////////////////////////////////////////////////
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 670-696 / 第 670-696 行

~~~~python
            if self._enable_fast_sigmoid:
                t1 = acc_S.load()
                t2 = t1 * 0.5
                acc_S.store(t2)
                for i in cutlass.range_constexpr(cute.size(acc_S.shape[0])):
                    for j in cutlass.range_constexpr(cute.size(acc_S.shape[1])):
                        for k in cutlass.range_constexpr(cute.size(acc_S.shape[2])):
                            ret = llvm.inline_asm(
                                cutlass.Float32.mlir_type,
                                [acc_S[i, j, k].ir_value()],
                                "tanh.approx.f32 $0, $1;",
                                "=f,f",
                                has_side_effects=False,
                                is_align_stack=False,
                                asm_dialect=llvm.AsmDialect.AD_ATT,
                            )
                            acc_S[i, j, k] = ret
                t3 = acc_S.load()
                t4 = t2 * t3 + t2
                acc_S.store(t4)
            else:
                LOG2_E = 1.4426950408889634074
                t1 = acc_S.load()
                t2 = t1 * -LOG2_E
                t3 = cute.math.exp2(t2, fastmath=True) + 1.0
                t4 = t1 / t3
                acc_S.store(t4)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 698-705 / 第 698-705 行

~~~~python
            mACC = cute.make_identity_tensor(
                (mRAB.layout.shape[2], mRAB.layout.shape[3])
            )  # (seqlen_q, seqlen_kv)
            cACC = cute.local_tile(
                mACC[None, None],
                (self._m_block_size, self._n_block_size),
                (m_block, n_block_idx),
            )
~~~~

**EN**: Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 707-717 / 第 707-717 行

~~~~python
            if self._is_causal and (n_block - n_block_idx) < cute.ceil_div(
                self._m_block_size, self._n_block_size
            ):
                tACCcACC = thr_mma.partition_C(cACC)
                for i in cutlass.range_constexpr(cute.size(tACCcACC.shape[0])):
                    for j in cutlass.range_constexpr(cute.size(tACCcACC.shape[1])):
                        for k in cutlass.range_constexpr(cute.size(tACCcACC.shape[2])):
                            if cute.elem_less(
                                tACCcACC[i, j, k][0], tACCcACC[i, j, k][1]
                            ):
                                acc_S[i, j, k] = 0.0
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 719-720 / 第 719-720 行

~~~~python
            rP = cute.make_rmem_tensor_like(acc_S, self._dtype)
            rP.store(acc_S.load().to(self._dtype))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 722-727 / 第 722-727 行

~~~~python
            # ///////////////////////////////////////////////////////////////////////////////
            # O gemm calculation
            # ///////////////////////////////////////////////////////////////////////////////
            # Convert layout of acc_S to gemm O accept layout.
            # Due to the mma instruction shape is 16x8x16, we need to convert from (4, MMA_M, MMA_N) to ((4, 2), MMA_M, MMA_N / 2)
            # (4, MMA_M, MMA_N) -> (4, MMA_M, (2, MMA_N / 2))
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 728-741 / 第 728-741 行

~~~~python
            rP_layout_divided = cute.logical_divide(rP.layout, (None, None, 2))
            rP_mma_view = cute.make_layout(
                (
                    (rP_layout_divided.shape[0], rP_layout_divided.shape[2][0]),
                    rP_layout_divided.shape[1],
                    rP_layout_divided.shape[2][1],
                ),
                stride=(
                    (rP_layout_divided.stride[0], rP_layout_divided.stride[2][0]),
                    rP_layout_divided.stride[1],
                    rP_layout_divided.stride[2][1],
                ),
            )
            tOrP = cute.make_tensor(rP.iterator, rP_mma_view)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 743-743 / 第 743-743 行

~~~~python
            # ldmatrix first V k-block for mma
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 744-764 / 第 744-764 行

~~~~python
            cute.copy(
                smem_tiled_copy_V,
                tOsVt[None, None, 0],
                tOrVt_copy_view[None, None, 0],
            )
            for k in cutlass.range_constexpr(0, cute.size(tOrP.shape[2])):
                # ldmatrix next V k-block for mma
                if k < cute.size(tOrP.shape[2]) - 1:
                    cute.copy(
                        smem_tiled_copy_V,
                        tOsVt[None, None, k + 1],
                        tOrVt_copy_view[None, None, k + 1],
                    )
                # mma for O=P@V
                cute.gemm(
                    tiled_mma,
                    acc_O,
                    tOrP[None, None, k],
                    tOrVt[None, None, k],
                    acc_O,
                )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 766-769 / 第 766-769 行

~~~~python
        # ///////////////////////////////////////////////////////////////////////////////
        # Epilogue
        # ///////////////////////////////////////////////////////////////////////////////
        # store acc_O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 770-792 / 第 770-792 行

~~~~python
        rO = cute.make_rmem_tensor(acc_O.layout, self._dtype)
        rO.store(acc_O.load().to(self._dtype))
        # reuse sQ's data iterator
        sO_iter = cute.recast_ptr(sQ.iterator, sO_layout.inner)
        sO = cute.make_tensor(sO_iter, sO_layout.outer)
        smem_copy_atom_O = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(), self._dtype
        )
        smem_tiled_copy_O = cute.make_tiled_copy_C(smem_copy_atom_O, tiled_mma)
        smem_thr_copy_O = smem_tiled_copy_O.get_slice(tidx)
        taccOrO = smem_thr_copy_O.retile(rO)
        taccOsO = smem_thr_copy_O.partition_D(sO)
        # copy acc O from rmem to smem with sts.32(auto vectorization)
        cute.copy(
            smem_copy_atom_O,
            taccOrO,
            taccOsO,
        )
        gO = cute.local_tile(
            mO[batch_size, None, num_head, None],
            (self._m_block_size, self._head_dim_padded),
            (m_block, 0),
        )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。

### Lines 794-820 / 第 794-820 行

~~~~python
        gmem_thr_copy_O = gmem_tiled_copy_O.get_slice(tidx)
        tOsO = gmem_thr_copy_O.partition_S(sO)
        tOgO = gmem_thr_copy_O.partition_D(gO)
        tOrO = cute.make_fragment_like(tOgO, self._dtype)
        # sync before all sts are done.
        self.cta_sync_barrier.arrive_and_wait()
        # load acc O from smem to rmem for wider vectorization
        cute.copy(
            gmem_tiled_copy_O,
            tOsO,
            tOrO,
        )
        # predicate for O
        mcO = cute.make_identity_tensor(mO.layout.shape)
        cO = cute.local_tile(
            mcO[batch_size, None, num_head, None],
            (self._m_block_size, self._head_dim_padded),
            (m_block, 0),
        )
        tOcO = gmem_thr_copy_O.partition_D(cO)
        tOpO = cute.make_rmem_tensor(
            cute.make_layout(
                (tOgO.shape[0][1], tOgO.shape[1], tOgO.shape[2]),
                stride=(tOgO.shape[2], 0, 1),
            ),
            cutlass.Boolean,
        )
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 821-834 / 第 821-834 行

~~~~python
        for rest_v in cutlass.range_constexpr(tOpO.shape[0]):
            for rest_n in cutlass.range_constexpr(cute.size(tOpO.shape[2])):
                tOpO[rest_v, 0, rest_n] = cute.elem_less(
                    tOcO[(0, rest_v), 0, rest_n][3], mO.layout.shape[3]
                )
        # copy acc O from rmem to gmem
        for rest_m in cutlass.range_constexpr(cute.size(tOpO.shape[1])):
            if cute.elem_less(tOcO[0, rest_m, 0][1], mO.layout.shape[1]):
                cute.copy(
                    gmem_tiled_copy_O,
                    tOrO[None, rest_m, None],
                    tOgO[None, rest_m, None],
                    pred=tOpO[None, rest_m, None],
                )
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 837-855 / 第 837-855 行

~~~~python
def run_pytorch_hstu_test(
    dtype,
    q,
    k,
    v,
    rab,
    is_causal: bool,
):
    """Generate the reference output of the HSTU attention with Pytorch.

    :param dtype: data type of the input tensors
    :param q: query tensor
    :param k: key tensor
    :param v: value tensor
    :param rab: RAB tensor
    :param is_causal: whether to use causal masking
    :type is_causal: bool
    """
    import torch
~~~~

**EN**: Defines `run_pytorch_hstu_test`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `run_pytorch_hstu_test`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 857-860 / 第 857-860 行

~~~~python
    q = q.to(dtype)
    k = k.to(dtype)
    v = v.to(dtype)
    rab = rab.to(dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 862-867 / 第 862-867 行

~~~~python
    s_ = torch.matmul(q, k.transpose(-2, -1)) + rab
    s_ = torch.nn.functional.silu(s_)
    if is_causal:
        mask = torch.ones(1, 1, q.shape[2], k.shape[2], dtype=dtype)
        mask = torch.tril(mask)
        s_ = s_ * mask.cuda()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 869-870 / 第 869-870 行

~~~~python
    o = torch.matmul(s_, v).permute(0, 2, 1, 3).contiguous()
    return o
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 873-900 / 第 873-900 行

~~~~python
def run(
    dtype: Type[cutlass.Numeric],
    batch_size: int,
    seqlen_q: int,
    seqlen_kv: int,
    num_head: int,
    head_dim: int,
    m_block_size: int = 128,
    n_block_size: int = 128,
    num_threads: int = 128,
    enable_fast_sigmoid: bool = False,
    enable_block_rasterization: bool = False,
    is_causal: bool = False,
    perf_test: bool = False,
    **kwargs,
):
    """
    Run the HSTU attention kernel.

    :param dtype: data type of the input tensors
    :type dtype: Type[cutlass.Numeric]
    :param batch_size: batch size
    :type batch_size: int
    :param seqlen_q: sequence length of the query
    :type seqlen_q: int
    :param seqlen_kv: sequence length of the key
    :type seqlen_kv: int
    :param num_head: number of attention heads
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 901-917 / 第 901-917 行

~~~~python
    :type num_head: int
    :param head_dim: dimension of the head
    :type head_dim: int
    :param m_block_size: block size for the m dimension of computation
    :type m_block_size: int
    :param n_block_size: block size for the n dimension of computation
    :type n_block_size: int
    :param num_threads: number of threads
    :type num_threads: int
    :param enable_fast_sigmoid: whether to use fast sigmoid
    :type enable_fast_sigmoid: bool
    :param enable_block_rasterization: whether to use block rasterization
    :type enable_block_rasterization: bool
    :param is_causal: whether to use causal masking
    :type is_causal: bool
    """
    assert dtype == cutlass.Float16 or dtype == cutlass.BFloat16
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 919-920 / 第 919-920 行

~~~~python
    import torch
    import cutlass.torch as cutlass_torch
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 922-923 / 第 922-923 行

~~~~python
    torch_stream = torch.cuda.current_stream()
    stream = cuda.CUstream(torch_stream.cuda_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 925-937 / 第 925-937 行

~~~~python
    print("Running Ampere SM80 HSTUAttentionForward test with:")
    print("batch_size: ", batch_size)
    print("seqlen_q: ", seqlen_q)
    print("seqlen_kv: ", seqlen_kv)
    print("num_head: ", num_head)
    print("head_dim: ", head_dim)
    print("m_block_size: ", m_block_size)
    print("n_block_size: ", n_block_size)
    print("num_threads: ", num_threads)
    print("is_causal: ", is_causal)
    print("enable_fast_sigmoid: ", enable_fast_sigmoid)
    print("enable_block_rasterization: ", enable_block_rasterization)
    print("dtype: ", dtype)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 939-939 / 第 939-939 行

~~~~python
    # reduced tensor num and iter num for functionality test
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 940-946 / 第 940-946 行

~~~~python
    TENSOR_NUM = 1
    ITER_NUM = 1
    WARMUP_NUM = 0
    if perf_test:
        TENSOR_NUM = 3
        ITER_NUM = 100
        WARMUP_NUM = 10
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 948-948 / 第 948-948 行

~~~~python
    # Create tensor Q/K/V/O
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 949-972 / 第 949-972 行

~~~~python
    qs = [
        torch.randn(
            batch_size, seqlen_q, num_head, head_dim, dtype=cutlass_torch.dtype(dtype)
        ).cuda()
        for _ in range(TENSOR_NUM)
    ]
    ks = [
        torch.randn(
            batch_size, seqlen_kv, num_head, head_dim, dtype=cutlass_torch.dtype(dtype)
        ).cuda()
        for _ in range(TENSOR_NUM)
    ]
    vs = [
        torch.randn(
            batch_size, seqlen_kv, num_head, head_dim, dtype=cutlass_torch.dtype(dtype)
        ).cuda()
        for _ in range(TENSOR_NUM)
    ]
    os = [
        torch.randn(
            batch_size, seqlen_q, num_head, head_dim, dtype=cutlass_torch.dtype(dtype)
        ).cuda()
        for _ in range(TENSOR_NUM)
    ]
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 974-979 / 第 974-979 行

~~~~python
    rabs = [
        torch.randn(
            batch_size, num_head, seqlen_q, seqlen_kv, dtype=cutlass_torch.dtype(dtype)
        ).cuda()
        for _ in range(TENSOR_NUM)
    ]
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 981-994 / 第 981-994 行

~~~~python
    fa2_fwd = HSTUAttentionForwardAmpere(
        dtype,
        batch_size,
        seqlen_q,
        seqlen_kv,
        num_head,
        head_dim,
        m_block_size,
        n_block_size,
        num_threads,
        enable_fast_sigmoid=enable_fast_sigmoid,
        enable_block_rasterization=enable_block_rasterization,
        is_causal=is_causal,
    )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 995-1022 / 第 995-1022 行

~~~~python
    # assume input is 16B align.
    mqs = [
        (
            from_dlpack(qs[i], assumed_align=16)
            .mark_layout_dynamic(leading_dim=3)
            .mark_compact_shape_dynamic(
                mode=3,
                stride_order=qs[i].dim_order(),
                divisibility=(128 // dtype.width),
            )
        )
        for i in range(TENSOR_NUM)
    ]
    mks = [
        (
            from_dlpack(ks[i], assumed_align=16)
            .mark_layout_dynamic(leading_dim=3)
            .mark_compact_shape_dynamic(
                mode=3,
                stride_order=ks[i].dim_order(),
                divisibility=(128 // dtype.width),
            )
        )
        for i in range(TENSOR_NUM)
    ]
    mvs = [
        (
            from_dlpack(vs[i], assumed_align=16)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1023-1050 / 第 1023-1050 行

~~~~python
            .mark_layout_dynamic(leading_dim=3)
            .mark_compact_shape_dynamic(
                mode=3,
                stride_order=vs[i].dim_order(),
                divisibility=(128 // dtype.width),
            )
        )
        for i in range(TENSOR_NUM)
    ]
    mos = [
        (
            from_dlpack(os[i], assumed_align=16)
            .mark_layout_dynamic(leading_dim=3)
            .mark_compact_shape_dynamic(
                mode=3,
                stride_order=os[i].dim_order(),
                divisibility=(128 // dtype.width),
            )
        )
        for i in range(TENSOR_NUM)
    ]
    mrabs = [
        (
            from_dlpack(rabs[i], assumed_align=16)
            .mark_layout_dynamic(leading_dim=3)
            .mark_compact_shape_dynamic(
                mode=3,
                stride_order=rabs[i].dim_order(),
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1051-1057 / 第 1051-1057 行

~~~~python
                divisibility=(128 // dtype.width),
            )
        )
        for i in range(TENSOR_NUM)
    ]
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1059-1067 / 第 1059-1067 行

~~~~python
    kernel = cute.compile(
        fa2_fwd,
        mqs[0],
        mks[0],
        mvs[0],
        mos[0],
        mrabs[0],
        stream,
    )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1069-1080 / 第 1069-1080 行

~~~~python
    for i in range(0, ITER_NUM):
        if i == WARMUP_NUM:
            start_event.record(torch_stream)
        # Run the kernel
        kernel(
            mqs[i % TENSOR_NUM],
            mks[i % TENSOR_NUM],
            mvs[i % TENSOR_NUM],
            mos[i % TENSOR_NUM],
            mrabs[i % TENSOR_NUM],
            stream,
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1082-1083 / 第 1082-1083 行

~~~~python
    end_event.record(torch_stream)
    torch.cuda.synchronize(torch_stream)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1085-1086 / 第 1085-1086 行

~~~~python
    elapsed_time = start_event.elapsed_time(end_event)
    elapsed_time_avg = elapsed_time / (ITER_NUM - WARMUP_NUM)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1088-1092 / 第 1088-1092 行

~~~~python
    LAST_USED_TENSOR = (ITER_NUM - 1) % TENSOR_NUM
    q = qs[LAST_USED_TENSOR].permute(0, 2, 1, 3).contiguous()
    k = ks[LAST_USED_TENSOR].permute(0, 2, 1, 3).contiguous()
    v = vs[LAST_USED_TENSOR].permute(0, 2, 1, 3).contiguous()
    rab = rabs[LAST_USED_TENSOR]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1094-1094 / 第 1094-1094 行

~~~~python
    kernel_out = os[LAST_USED_TENSOR].cpu()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 1096-1099 / 第 1096-1099 行

~~~~python
    with torch.cuda.stream(torch_stream):
        ref_bf16 = run_pytorch_hstu_test(torch.bfloat16, q, k, v, rab, is_causal).cpu()
        ref_fp32 = run_pytorch_hstu_test(torch.float32, q, k, v, rab, is_causal).cpu()
    torch.cuda.synchronize(torch_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 1101-1104 / 第 1101-1104 行

~~~~python
    assert (kernel_out - ref_fp32).abs().max().item() <= 4 * (
        ref_bf16 - ref_fp32
    ).abs().max().item()
    print("Results verified successfully!")
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1106-1108 / 第 1106-1108 行

~~~~python
    if perf_test:
        print(f"Elapsed time: {elapsed_time_avg:.3f} ms")
    return elapsed_time_avg * 1000  # return in microseconds
~~~~

**EN**: Returns the constructed object or computed result to the caller. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把构造好的对象或计算结果返回给调用方。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 1111-1132 / 第 1111-1132 行

~~~~python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="example of HSTU attention with CuTe")
    parser.add_argument("--dtype", type=cutlass.dtype, default=cutlass.BFloat16)
    parser.add_argument("--batch_size", type=int, default=4)
    parser.add_argument("--seqlen_q", type=int, default=2048)
    parser.add_argument("--seqlen_kv", type=int, default=2048)
    parser.add_argument("--num_head", type=int, default=4)
    parser.add_argument("--head_dim", type=int, default=128)
    parser.add_argument("--m_block_size", type=int, default=64)
    parser.add_argument("--n_block_size", type=int, default=64)
    parser.add_argument("--num_threads", type=int, default=128)
    parser.add_argument(
        "--no_fast_sigmoid", action="store_false", dest="enable_fast_sigmoid"
    )
    parser.add_argument(
        "--no_block_rasterization",
        action="store_false",
        dest="enable_block_rasterization",
    )
    parser.add_argument("--is_causal", action="store_true", dest="is_causal")
    parser.add_argument("--perf_test", action="store_true", dest="perf_test")
    args = parser.parse_args()
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 1134-1149 / 第 1134-1149 行

~~~~python
    run(
        args.dtype,
        args.batch_size,
        args.seqlen_q,
        args.seqlen_kv,
        args.num_head,
        args.head_dim,
        args.m_block_size,
        args.n_block_size,
        args.num_threads,
        args.enable_fast_sigmoid,
        args.enable_block_rasterization,
        args.is_causal,
        args.perf_test,
    )
    print("PASS")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Ampere GPU execution model / Ampere GPU 执行模型
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Specialized sequence or attention dataflow / 专门化的序列/注意力数据流
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Multistage pipeline and synchronization / 多级流水线与同步

## Dependencies / 依赖项

- `typing.Type` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `cutlass._mlir.dialects.llvm` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.pipeline` — builds multistage pipelines and synchronization helpers / 构建多级流水线与同步辅助工具
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
