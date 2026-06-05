# cta_norm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/hopper/kernel/cta_norm/cta_norm.py`  
**Purpose / 用途**: Kernel example implementing cta norm with CuTeDSL. / 这是一个使用 CuTeDSL 实现 cta norm 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-39 / 第 29-39 行

~~~~python
import argparse
import torch
import time
import math
from typing import Type, Optional

import cutlass
import cutlass.cute as cute
import cutlass.cute.testing as testing
import cutlass.torch as cutlass_torch
from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 42-68 / 第 42-68 行

~~~~python
"""
CTA-level LayerNorm / RMSNorm Example using CuTe DSL.

This example implements a CTA-level normalization kernel (LayerNorm or RMSNorm)
using the CuTe DSL. Each CTA processes one row of the input tensor and performs
the full normalization pipeline, including global memory loads, reduction,
normalization, and global memory stores.

In this kernel:

- Threads are arranged linearly within a CTA.
- Vectorized 128-bit loads/stores are used to maximize memory bandwidth.

To run this example:

.. code-block:: bash

    python examples/python/CuTeDSL/hopper/cta_norm.py
    python examples/python/CuTeDSL/hopper/cta_norm.py \
        --M 4096 --N 8192 --dtype fp16 --threads 256 \
        --norm_type rms --benchmark

To collect performance with NCU profiler:

.. code-block:: bash
    ncu -k regex:".*cutlass.*" python examples/python/CuTeDSL/hopper/cta_norm.py
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 70-74 / 第 70-74 行

~~~~python
DTYPE_MAP = {
    "fp16": cutlass.Float16,
    "bf16": cutlass.BFloat16,
    "fp32": cutlass.Float32,
}
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 76-87 / 第 76-87 行

~~~~python
class CtaNorm:
    def __init__(
        self, N: int,
        norm_type: str,
        threads_per_cta: Optional[int] = None,
    ):
        self.N = N # hidden_size
        self.norm_type = norm_type # "layer" or "rms"
        self.elems_per_thread = 8
        self.warp_size = 32
        self.threads_per_cta = threads_per_cta or self.heuristic_threads()
        self.warps_per_cta = (self.threads_per_cta + 31) // self.warp_size
~~~~

**EN**: Defines `CtaNorm`, a reusable Python class that packages configuration and behavior for this example. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `CtaNorm`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 89-95 / 第 89-95 行

~~~~python
    def heuristic_threads(self):
        elems_per_warp = self.elems_per_thread * self.warp_size
        heu_warps = (self.N + elems_per_warp - 1) // elems_per_warp // 4
        heu_warps = max(heu_warps, 1) # at least one warp
        heu_warps = (heu_warps + 1) // 2 * 2 # be multiple of 2
        heu_threads = heu_warps * 32
        return heu_threads
~~~~

**EN**: Defines `heuristic_threads`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `heuristic_threads`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 97-124 / 第 97-124 行

~~~~python
    @cute.jit
    def __call__(
        self,
        mY,
        mX,
        mWeight,
        mBias,
        eps: cutlass.Float32 = 1e-6,
    ):
        print("[DSL INFO] Input Tensors:")
        print(f"[DSL INFO]   mY = {mY.type}")
        print(f"[DSL INFO]   mX = {mX.type}")
        print(f"[DSL INFO]   mWeight = {mWeight.type}")
        if cutlass.const_expr(self.norm_type == "layer"):
            print(f"[DSL INFO]   mBias = {mBias.type}")
        M, _ = mX.shape
        atom_copy = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(),
            mX.element_type,
            num_bits_per_copy=128,
        )
        t_layout = cute.make_layout(self.threads_per_cta)  # thread layout within a CTA
        v_layout = cute.make_layout(self.elems_per_thread)  # per-thread vector layout
        tiled_copy = cute.make_tiled_copy_tv(atom_copy, t_layout, v_layout)
        print("[DSL INFO] Tiling Parameters:")
        print(f"[DSL INFO]   tiled_copy = {tiled_copy}")
        self.kernel(mY, mX, mWeight, mBias, tiled_copy, eps).launch(
            grid=[M, 1, 1],
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 125-126 / 第 125-126 行

~~~~python
            block=[self.warps_per_cta * self.warp_size, 1, 1],
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 128-140 / 第 128-140 行

~~~~python
    @cute.kernel
    def kernel(
        self,
        mY: cute.Tensor,
        mX: cute.Tensor,
        mWeight: Optional[cute.Tensor],
        mBias: Optional[cute.Tensor],
        tiled_copy: cute.TiledCopy,
        eps: cute.Float,
    ):
        tidx, _, _ = cute.arch.thread_idx()  # thread index
        bidx, _, _ = cute.arch.block_idx()  # cta index
        thr_copy = tiled_copy.get_slice(tidx)
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 142-167 / 第 142-167 行

~~~~python
        gY = cute.local_tile(mY, tiler=(1, self.N), coord=(bidx, 0))
        gX = cute.local_tile(mX, tiler=(1, self.N), coord=(bidx, 0))
        gY, gX = gY[0, None], gX[0, None]
        print("[DSL INFO] Tiled Tensors:")
        print(f"[DSL INFO]   gY = {gY.type}")
        print(f"[DSL INFO]   gX = {gX.type}")
        tYgY = thr_copy.partition_S(gY)
        pred = cute.make_rmem_tensor(
            cute.size(tYgY, mode=[1]), cutlass.Boolean,
        )
        for i in range(cute.size(pred)):
            offset = (i * self.threads_per_cta + tidx) * self.elems_per_thread
            pred[i] = offset < self.N
        tXgX = thr_copy.partition_S(gX)
        tWgW = thr_copy.partition_S(mWeight)
        if cutlass.const_expr(self.norm_type == "layer"):
            tBgB = thr_copy.partition_S(mBias)
        tXrX = cute.make_fragment_like(tXgX)
        tXrX.fill(0)  # initialize rmem fragment to zero to simplify reduction code
        tWrW = cute.make_fragment_like(tWgW)
        if cutlass.const_expr(self.norm_type == "layer"):
            tBrB = cute.make_fragment_like(tBgB)
        print("[DSL INFO] Sliced Tensors per thread:")
        print(f"[DSL INFO]   tYgY = {tYgY.type}")
        print(f"[DSL INFO]   tXgX = {tXgX.type}")
        print(f"[DSL INFO]   tWgW = {tWgW.type}")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 168-185 / 第 168-185 行

~~~~python
        if cutlass.const_expr(self.norm_type == "layer"):
            print(f"[DSL INFO]   tBgB = {tBgB.type}")
        print(f"[DSL INFO]   pred = {pred.type}")
        for i in range(cute.size(tXrX, mode=[1])):
            if pred[i]:
                cute.autovec_copy(tXgX[None, i], tXrX[None, i])  # Global load
                cute.autovec_copy(tWgW[None, i], tWrW[None, i])  # Global load
                if cutlass.const_expr(self.norm_type == "layer"):
                    cute.autovec_copy(tBgB[None, i], tBrB[None, i])  # Global load
        if cutlass.const_expr(self.norm_type == "layer"):
            tYrY = self.apply_layernorm(tXrX, tWrW, tBrB, eps, tidx, pred)
        elif cutlass.const_expr(self.norm_type == "rms"):
            tYrY = self.apply_rmsnorm(tXrX, tWrW, eps, tidx, pred)
        else:
            raise ValueError("norm_type must be 'layer' or 'rms'.")
        for i in range(cute.size(tXrX, mode=[1])):
            if pred[i]:
                cute.autovec_copy(tYrY[None, i], tYgY[None, i])  # STG.128
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 188-193 / 第 188-193 行

~~~~python
    @cute.jit
    def warp_reduce(self, val, reduce_size = 32):
        iters = int(math.log2(reduce_size))
        for i in range(iters):
            val = val + cute.arch.shuffle_sync_bfly(val, offset=1<<i)
        return val
~~~~

**EN**: Marks `warp_reduce` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Returns the constructed object or computed result to the caller.
**CN**: 将 `warp_reduce` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 把构造好的对象或计算结果返回给调用方。

### Lines 195-208 / 第 195-208 行

~~~~python
    @cute.jit
    def cta_reduce(self, val, acc, tidx):
        warp_id = tidx >> 5
        lane_id = tidx & 31
        if lane_id == 0:
            acc[warp_id] = val
        cute.arch.sync_threads()
        if warp_id == 0:
            val = acc[lane_id] if lane_id < self.warps_per_cta else cutlass.Float32(0)
            val = self.warp_reduce(val)
            acc[self.warps_per_cta] = val
        cute.arch.sync_threads()
        val = acc[self.warps_per_cta]
        return val
~~~~

**EN**: Marks `cta_reduce` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Returns the constructed object or computed result to the caller.
**CN**: 将 `cta_reduce` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 把构造好的对象或计算结果返回给调用方。

### Lines 210-237 / 第 210-237 行

~~~~python
    @cute.jit
    def apply_layernorm(
        self,
        x: cute.Tensor,
        weight: cute.Tensor,
        bias: cute.Tensor,
        eps: cute.Float,
        tidx: cutlass.Int32,
        pred: cute.Tensor,
    ):
        """
        mean = sum(x) / D
        var  = sum((x - mean) ^ 2) / D
        y[i] = (x[i] - mean) / sqrt(var + eps) * weight[i] + bias[i]
        """
        smem = cutlass.utils.SmemAllocator()
        acc = smem.allocate_tensor(cutlass.Float32, self.warps_per_cta + 1)
        # Reduce x
        val = cute.Float32(0.0)
        for idx in range(cute.size(x)):
            # Accumulate in FP32 to improve numerical precision.
            val += x[idx].to(cutlass.Float32)
        val = self.warp_reduce(val)
        val = self.cta_reduce(val, acc, tidx)
        mean = val / self.N
        # Reduce (x - mean) ^ 2
        val = cute.Float32(0.0)
        for i in range(cute.size(x, mode=[1])):
~~~~

**EN**: Marks `apply_layernorm` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 将 `apply_layernorm` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 238-250 / 第 238-250 行

~~~~python
            if pred[i]:
                for idx in range(cute.size(x[None, i])):
                    # Accumulate in FP32 to improve numerical precision.
                    x_fp32 = x[None, i][idx].to(cutlass.Float32)
                    val += (x_fp32 - mean) * (x_fp32 - mean)
        val = self.warp_reduce(val)
        val = self.cta_reduce(val, acc, tidx)
        factor = cute.rsqrt(val / self.N + eps)
        # Normalize
        normed = cute.make_fragment_like(x)
        value = (x.load() - mean) * factor * weight.load() + bias.load()
        normed.store(value.to(normed.element_type))
        return normed
~~~~

**EN**: Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 253-279 / 第 253-279 行

~~~~python
    @cute.jit
    def apply_rmsnorm(
        self,
        x: cute.Tensor,
        weight: cute.Tensor,
        eps: cute.Float,
        tidx: cutlass.Int32,
        pred: cute.Tensor,
    ):
        """
        y[i] = x[i] / sqrt(sum(x ^ 2) / D + eps) * w[i]
        """
        smem = cutlass.utils.SmemAllocator()
        acc = smem.allocate_tensor(cutlass.Float32, self.warps_per_cta + 1)
        val = cute.Float32(0.0)
        for i in range(cute.size(x, mode=[1])):
            if pred[i]:
                for idx in range(cute.size(x[None, i])):
                    # Accumulate in FP32 to improve numerical precision.
                    x_fp32 = x[None, i][idx].to(cutlass.Float32)
                    val += x_fp32 * x_fp32
        val = self.warp_reduce(val)
        acc_sq = self.cta_reduce(val, acc, tidx)
        factor = cute.rsqrt(acc_sq / self.N + eps)
        tNrN = cute.make_fragment_like(x)
        tNrN.store((x.load() * factor * weight.load()).to(tNrN.element_type))
        return tNrN
~~~~

**EN**: Marks `apply_rmsnorm` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Returns the constructed object or computed result to the caller.
**CN**: 将 `apply_rmsnorm` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 把构造好的对象或计算结果返回给调用方。

### Lines 282-298 / 第 282-298 行

~~~~python
def run_layernorm(
    M: int,
    N: int,
    threads_per_cta: int,
    norm_type: str,
    dtype: Type[cutlass.Numeric],
    skip_ref_check=False,
    benchmark=True,
    warmup_iterations=2,
    iterations=200,
    eps=1e-6,
):
    if N % 8 > 0:
        raise ValueError(f"N = {N} must be a multiple of 8 for this example.")
    if threads_per_cta is not None:
        if threads_per_cta % 32 != 0 or not (0 < threads_per_cta <= 1024):
            raise ValueError(f"Invalid threads_per_cta = {threads_per_cta}")
~~~~

**EN**: Defines `run_layernorm`, grouping related logic behind a named Python callable so the example stays modular and reusable. Measures performance or checks correctness so the example doubles as a validation harness. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `run_layernorm`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 测量性能或检查正确性，使该示例同时充当验证工具。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 300-302 / 第 300-302 行

~~~~python
    print("Running CtaNorm test with:")
    print(f"Tensor dimensions: [{M}, {N}]")
    print(f"Input and Output Data type: {dtype}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 304-310 / 第 304-310 行

~~~~python
    torch_dtype = cutlass_torch.dtype(dtype)
    x = torch.randn(M, N, device=torch.device("cuda"), dtype=torch_dtype)
    weight = torch.randn(N, device=torch.device("cuda"), dtype=torch_dtype)
    bias = None
    if norm_type == "layer":
        bias = torch.randn(N, device=torch.device("cuda"), dtype=torch_dtype)
    y = torch.empty_like(x)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 312-317 / 第 312-317 行

~~~~python
    print("Input tensor shapes:")
    print(f"x: {x.shape}, dtype: {x.dtype}")
    print(f"weight: {weight.shape}, dtype: {weight.dtype}")
    if norm_type == "layer":
        print(f"bias: {bias.shape}, dtype: {bias.dtype}")
    print(f"y: {y.shape}, dtype: {y.dtype}\n")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 319-324 / 第 319-324 行

~~~~python
    _x = from_dlpack(x, assumed_align=16, enable_tvm_ffi=True)
    _weight = from_dlpack(weight, assumed_align=16, enable_tvm_ffi=True)
    _bias = None
    if norm_type == "layer":
        _bias = from_dlpack(bias, assumed_align=16, enable_tvm_ffi=True)
    _y = from_dlpack(y, assumed_align=16, enable_tvm_ffi=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 326-338 / 第 326-338 行

~~~~python
    print("Compiling kernel with cute.compile ...")
    start_time = time.time()
    layernorm = CtaNorm(N, norm_type, threads_per_cta)
    if norm_type == "layer":
        compiled_func = cute.compile(
            layernorm, _y, _x, _weight, _bias, options="--generate-line-info --enable-tvm-ffi",
        )
    else:
        compiled_func = cute.compile(
            layernorm, _y, _x, _weight, _bias, options="--generate-line-info --enable-tvm-ffi",
        )
    compilation_time = time.time() - start_time
    print(f"Compilation time: {compilation_time:.4f} seconds")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 340-349 / 第 340-349 行

~~~~python
    print("Executing vector add kernel...")
    if not skip_ref_check:
        compiled_func(y, x, weight, bias, eps)
        print("Verifying results...")
        if norm_type == "layer":
            ref = torch.layer_norm(x, (N,), weight, bias, eps)
        else:
            ref = torch.rms_norm(x, (N,), weight, eps)
        torch.testing.assert_close(y, ref, atol=1e-3, rtol=1e-3)
        print("Results verified successfully!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 351-352 / 第 351-352 行

~~~~python
    if not benchmark:
        return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 354-361 / 第 354-361 行

~~~~python
    def generate_tensors():
        x = torch.randn(M, N, device=torch.device("cuda"), dtype=torch_dtype)
        weight = torch.randn(N, device=torch.device("cuda"), dtype=torch_dtype)
        bias = None
        if norm_type == "layer":
            bias = torch.randn(N, device=torch.device("cuda"), dtype=torch_dtype)
        y = torch.empty_like(x)
        return testing.JitArguments(y, x, weight, bias, eps)
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 363-367 / 第 363-367 行

~~~~python
    def torch_ref(y, x, weight, bias, eps):
        if norm_type == "layer":
            y = torch.layer_norm(x, (N,), weight, bias, eps)
        else:
            y = torch.rms_norm(x, (N,), weight, eps)
~~~~

**EN**: Defines `torch_ref`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `torch_ref`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 369-376 / 第 369-376 行

~~~~python
    def eval(func, name):
        avg_time_us = testing.benchmark(
            func,
            workspace_generator=generate_tensors,
            workspace_count=10,
            warmup_iterations=warmup_iterations,
            iterations=iterations,
        )
~~~~

**EN**: Defines `eval`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `eval`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 378-378 / 第 378-378 行

~~~~python
        # Print execution results
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 379-383 / 第 379-383 行

~~~~python
        print(f"\n{name}")
        print(f"Kernel execution time: {avg_time_us / 1e3:.4f} ms")
        print(
            f"Achieved memory throughput: {(2 * (x.numel() + weight.numel()) * dtype.width // 8) / (avg_time_us / 1e6) / 1e9:.2f} GB/s"
        )
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 385-386 / 第 385-386 行

~~~~python
    eval(compiled_func, f"CuTe {norm_type}norm kernel")
    eval(torch_ref, f"PyTorch {norm_type}norm reference")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 389-406 / 第 389-406 行

~~~~python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="example of elementwise add to demonstrate the numpy/pytorch as input for kernels"
    )
    parser.add_argument("--M", default=4096, type=int)
    parser.add_argument("--N", default=4096, type=int)
    parser.add_argument(
        "--dtype",
        default="fp16",
        choices=DTYPE_MAP.keys(),
        help="Data type for input/output tensors (e.g. float16, bf16, float32)",
    )
    parser.add_argument("--norm_type", choices=["layer", "rms"], default="layer", type=str)
    parser.add_argument("--threads", default=None, type=int)
    parser.add_argument("--warmup_iterations", default=2, type=int)
    parser.add_argument("--iterations", default=100, type=int)
    parser.add_argument("--skip_ref_check", action="store_true")
    parser.add_argument("--benchmark", action="store_true")
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 408-408 / 第 408-408 行

~~~~python
    args = parser.parse_args()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 410-411 / 第 410-411 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 413-424 / 第 413-424 行

~~~~python
    run_layernorm(
        args.M,
        args.N,
        args.threads,
        args.norm_type,
        dtype=cutlass.Float16,
        skip_ref_check=args.skip_ref_check,
        benchmark=args.benchmark,
        warmup_iterations=args.warmup_iterations,
        iterations=args.iterations,
    )
    print("\nPASS")
~~~~

**EN**: Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Hopper GPU execution model / Hopper GPU 执行模型
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `time` — measures host-side timing information / 测量宿主端时间信息
- `math` — used by this example / 供该示例使用
- `typing.Type` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Optional` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
