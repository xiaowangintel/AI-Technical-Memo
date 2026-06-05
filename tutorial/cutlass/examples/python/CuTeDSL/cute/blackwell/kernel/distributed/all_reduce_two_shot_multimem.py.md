# all_reduce_two_shot_multimem.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/distributed/all_reduce_two_shot_multimem.py`  
**Purpose / 用途**: Kernel example implementing all reduce two shot multimem with CuTeDSL. / 这是一个使用 CuTeDSL 实现 all reduce two shot multimem 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 30-45 / 第 30-45 行

~~~~python
import os
import time
import argparse

import numpy as np
import torch
import torch.distributed as dist
from cuda.core.experimental import Device
from cuda.pathfinder import load_nvidia_dynamic_lib

import cutlass
import cutlass.utils as utils
import cutlass.cute as cute
import cutlass.cute.testing as testing
import cutlass.torch as cutlass_torch
from cutlass.cute.runtime import from_dlpack
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 47-55 / 第 47-55 行

~~~~python
try:
    import nvshmem.core
except ImportError as exc:
    raise ImportError(
        "nvshmem4py is required but not installed. Please install it using:\n"
        "  For CUDA 12: pip install nvshmem4py-cu12\n"
        "  For CUDA 13: pip install nvshmem4py-cu13\n"
        "Note: nvshmem4py version >= 0.1.3 is recommended."
    ) from None
~~~~

**EN**: Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现或编排多 GPU 通信，把示例扩展到单设备之外。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 57-64 / 第 57-64 行

~~~~python
try:
    load_nvidia_dynamic_lib("nvshmem_host")
except RuntimeError as exc:
    raise ImportError(
        "nvshmem lib is required but not installed. Please install it using:\n"
        "  For CUDA 12: pip install nvidia-nvshmem-cu12\n"
        "  For CUDA 13: pip install nvidia-nvshmem-cu13\n"
    ) from None
~~~~

**EN**: Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现或编排多 GPU 通信，把示例扩展到单设备之外。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 67-92 / 第 67-92 行

~~~~python
"""
A Distributed Two-Shot All-Reduce Example using CuTe DSL and PyTorch Symmetric Memory.

This example kernel demonstrates how to leverage the multimem feature to do a two-shot all-reduce.
The multimem instruction is operated on symmetric memory, it can offload the broadcast and reduce 
to the Nvlink Switch so that the nvlink traffic will be reduced.

When calling a 'multimem.ld_reduce addrA', the corresponding data from each remote device will be sent to the NVLS 
and return the reduced data as result. And for 'multimem.st dataA addrA', the data will be sent to the NVLS once and 
the data will be broadcast to each remote device. So the memory traffic and instruction count is reduced by 8 times 
with multimem.

In this example, we are using two-shot styled all-reduce which means each device computes a portion
of data and stores them to each device. Compared to the one-shot styled all-reduce, the two-shot one can 
maximize the performance of throughput. The input and output are symmetric memory so we don't need extra 
communication buffers here. We use the `sm_wise_inter_gpu_multimem_barrier` to synchronize the data 
between each device. It is to make sure that each device has done the data transfer.

To run this example:

.. code-block:: bash

    torchrun --nproc-per-node 8  examples/distributed/all_reduce_two_shot_multimem.py --M 1024 --N 512
    torchrun --nproc-per-node 8  examples/distributed/all_reduce_two_shot_multimem.py \
        --M 1024 --N 1024 --benchmark --warmup_iterations 2 --iterations 100
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 95-107 / 第 95-107 行

~~~~python
@cute.kernel
def all_reduce_multimem_kernel(
    gIn: cute.Tensor,
    gOut: cute.Tensor,
    flag: cute.Tensor,
    flag_mc: cute.Tensor,
    thr_layout: cute.Layout,
    val_layout: cute.Layout,
    local_rank: cutlass.Constexpr,
    world_size: cutlass.Constexpr,
):
    tidx, _, _ = cute.arch.thread_idx()
    bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `all_reduce_multimem_kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `all_reduce_multimem_kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 109-110 / 第 109-110 行

~~~~python
    # slice for CTAs
    # logical id -> address
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 112-114 / 第 112-114 行

~~~~python
    num_ctas = cute.size(gIn, mode=[1])
    chunk_size = num_ctas // world_size
    blk_idx = local_rank * chunk_size + bidx
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 116-118 / 第 116-118 行

~~~~python
    blk_coord = ((None, None), blk_idx)
    local_tile_out = gOut[blk_coord]
    local_tile_in = gIn[blk_coord]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 120-120 / 第 120-120 行

~~~~python
    assert gIn.element_type == gOut.element_type
~~~~

**EN**: Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 122-128 / 第 122-128 行

~~~~python
    copy_atom_load = cute.make_copy_atom(
        cute.nvgpu.CopyUniversalOp(),
        gIn.element_type,
        num_bits_per_copy=128,
    )
    tiled_copy = cute.make_tiled_copy_tv(copy_atom_load, thr_layout, val_layout)
    thr_copy = tiled_copy.get_slice(tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 130-131 / 第 130-131 行

~~~~python
    thr_in = thr_copy.partition_S(local_tile_in)
    thr_out = thr_copy.partition_D(local_tile_out)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 133-134 / 第 133-134 行

~~~~python
    (_, rest_m), _, _ = thr_in.shape
    (_, rest_m_stride), _, _ = thr_in.stride
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 136-142 / 第 136-142 行

~~~~python
    for i in cutlass.range_constexpr(rest_m):
        x, y, z, w = utils.distributed.multimem_ld_reduce_4xf32(
            thr_in[(None, i), 0, 0].iterator
        )
        utils.distributed.multimem_st_4xb32(
            thr_out[(None, i), 0, 0].iterator, x, y, z, w
        )
~~~~

**EN**: Implements or orchestrates multi-GPU communication, extending the example beyond a single device. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现或编排多 GPU 通信，把示例扩展到单设备之外。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 144-144 / 第 144-144 行

~~~~python
    # Ensure all threads in cta have finish issue multimem.ld_reduce and multimem.st instructions
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 145-145 / 第 145-145 行

~~~~python
    cute.arch.sync_threads()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 147-168 / 第 147-168 行

~~~~python
    if tidx == 0:
        # Linear id of current SM.
        sm_id_linear = (
            cute.arch.block_idx()[0]
            + cute.arch.block_idx()[1] * cute.arch.grid_dim()[0]
            + cute.arch.block_idx()[2]
            * cute.arch.grid_dim()[0]
            * cute.arch.grid_dim()[1]
        )
        # Release flag with sys scope
        utils.distributed.multimem_red_add1(
            flag_mc.iterator + sm_id_linear,
            scope="sys",
            order="release",
        )
        # Relaxed spin-lock wait flag with sys scope
        utils.distributed.spin_lock_atom_cas_relaxed_wait(
            flag.iterator + sm_id_linear,
            expected_val=world_size,
            reset_val=0,
            scope="sys",
        )
~~~~

**EN**: Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 170-181 / 第 170-181 行

~~~~python
@cute.jit
def all_reduce_multimem(
    mIn: cute.Tensor,
    mOut: cute.Tensor,
    flag: cute.Tensor,
    flag_mc: cute.Tensor,
    local_rank: cutlass.Constexpr,
    world_size: cutlass.Constexpr,
    copy_bits: cutlass.Constexpr = 128,
):
    dtype = mIn.element_type
    vector_size = copy_bits // dtype.width
~~~~

**EN**: Marks `all_reduce_multimem` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 将 `all_reduce_multimem` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 183-183 / 第 183-183 行

~~~~python
    # we choose a 128x128 tile for a CTA
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 184-186 / 第 184-186 行

~~~~python
    thr_layout = cute.make_ordered_layout((4, 32), order=(1, 0))
    val_layout = cute.make_ordered_layout((32, vector_size), order=(1, 0))
    tiler_mn, tv_layout = cute.make_layout_tv(thr_layout, val_layout)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 188-189 / 第 188-189 行

~~~~python
    gIn = cute.zipped_divide(mIn, tiler_mn)
    gOut = cute.zipped_divide(mOut, tiler_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 191-203 / 第 191-203 行

~~~~python
    all_reduce_multimem_kernel(
        gIn,
        gOut,
        flag,
        flag_mc,
        thr_layout,
        val_layout,
        local_rank,
        world_size,
    ).launch(
        grid=[cute.size(gOut, mode=[1]) // world_size, 1, 1],
        block=[cute.size(tv_layout, mode=[0]), 1, 1],
    )
~~~~

**EN**: Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 实现或编排多 GPU 通信，把示例扩展到单设备之外。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 206-215 / 第 206-215 行

~~~~python
def run_all_reduce_multimem(
    M,
    N,
    warmup_iterations=2,
    iterations=10,
    skip_ref_check=False,
    benchmark=True,
):
    local_rank = torch.distributed.get_rank()
    world_size = torch.distributed.get_world_size()
~~~~

**EN**: Defines `run_all_reduce_multimem`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `run_all_reduce_multimem`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 217-218 / 第 217-218 行

~~~~python
    tile_m = 128
    tile_n = 128
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 220-223 / 第 220-223 行

~~~~python
    if local_rank == 0:
        print("\nRunning Elementwise Add test with:")
        print(f"Tensor dimensions: [{M}, {N}]")
        print(f"GPU count: {world_size}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 225-226 / 第 225-226 行

~~~~python
    local_input_tensor = nvshmem.core.tensor((M, N), dtype=torch.float32)
    input_tensor = nvshmem.core.get_multicast_tensor(nvshmem.core.Teams.TEAM_NODE, local_input_tensor)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 228-229 / 第 228-229 行

~~~~python
    local_output_tensor = nvshmem.core.tensor((M, N), dtype=torch.float32)
    output_tensor = nvshmem.core.get_multicast_tensor(nvshmem.core.Teams.TEAM_NODE, local_output_tensor)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 231-232 / 第 231-232 行

~~~~python
    local_flag = nvshmem.core.tensor((M*N//(tile_m*tile_n)), dtype=torch.int32)
    flag_mc = nvshmem.core.get_multicast_tensor(nvshmem.core.Teams.TEAM_NODE, local_flag)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 234-249 / 第 234-249 行

~~~~python
    if local_rank == 0:
        print("Compiling kernel with cute.compile ...")
    start_time = time.time()
    compiled_func = cute.compile(
        all_reduce_multimem,
        from_dlpack(input_tensor),
        from_dlpack(output_tensor),
        from_dlpack(local_flag),
        from_dlpack(flag_mc),
        local_rank,
        world_size,
    )
    compilation_time = time.time() - start_time
    if local_rank == 0:
        print(f"Compilation time: {compilation_time:.4f} seconds")
        print("Executing all-reduce two shot multimem kernel...")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 251-261 / 第 251-261 行

~~~~python
    if not skip_ref_check:
        dist.barrier(device_ids=[local_rank])
        compiled_func(
            from_dlpack(input_tensor),
            from_dlpack(output_tensor),
            from_dlpack(local_flag),
            from_dlpack(flag_mc),
        )
        dist.barrier(device_ids=[local_rank])
        if local_rank == 0:
            print("Verifying results...")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 263-269 / 第 263-269 行

~~~~python
        local_buffers = [nvshmem.core.get_peer_tensor(local_input_tensor, local_rank) for local_rank in range(world_size)]
        torch.testing.assert_close(sum([buffer.cpu() for buffer in local_buffers]), local_output_tensor.cpu())
        if local_rank == 0:
            print("Results verified successfully!")
        for i in range(world_size):
            if i != local_rank:
                nvshmem.core.free_tensor(local_buffers[i])
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 271-271 / 第 271-271 行

~~~~python
    # always free the multicast tensors first
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 272-277 / 第 272-277 行

~~~~python
    nvshmem.core.free_tensor(input_tensor)
    nvshmem.core.free_tensor(output_tensor)
    nvshmem.core.free_tensor(flag_mc)
    nvshmem.core.free_tensor(local_input_tensor)
    nvshmem.core.free_tensor(local_output_tensor)
    nvshmem.core.free_tensor(local_flag)
~~~~

**EN**: Implements or orchestrates multi-GPU communication, extending the example beyond a single device. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现或编排多 GPU 通信，把示例扩展到单设备之外。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 279-280 / 第 279-280 行

~~~~python
    if not benchmark:
        return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 282-284 / 第 282-284 行

~~~~python
    free_func_and_tensor_pairs = []
    def add_free_func_and_tensor(free_func, tensor):
        free_func_and_tensor_pairs.append((free_func, tensor))
~~~~

**EN**: Defines `add_free_func_and_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `add_free_func_and_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 286-288 / 第 286-288 行

~~~~python
    def generate_tensors():
        local_input_tensor = nvshmem.core.tensor((M, N), dtype=torch.float32)
        input_tensor_mc = nvshmem.core.get_multicast_tensor(nvshmem.core.Teams.TEAM_NODE, local_input_tensor)
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 290-291 / 第 290-291 行

~~~~python
        local_output_tensor = nvshmem.core.tensor((M, N), dtype=torch.float32)
        output_tensor_mc = nvshmem.core.get_multicast_tensor(nvshmem.core.Teams.TEAM_NODE, local_output_tensor)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 293-294 / 第 293-294 行

~~~~python
        local_flag = nvshmem.core.tensor((M*N//(tile_m*tile_n)), dtype=torch.int32)
        flag_mc = nvshmem.core.get_multicast_tensor(nvshmem.core.Teams.TEAM_NODE, local_flag)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 296-305 / 第 296-305 行

~~~~python
        ja = testing.JitArguments(
            from_dlpack(input_tensor_mc),
            from_dlpack(output_tensor_mc),
            from_dlpack(local_flag),
            from_dlpack(flag_mc),
        )
        tensors_to_free = [input_tensor_mc, output_tensor_mc, flag_mc, local_input_tensor, local_output_tensor, local_flag]
        for tensor in tensors_to_free:
            add_free_func_and_tensor(nvshmem.core.free_tensor, tensor)
        return ja
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。 把构造好的对象或计算结果返回给调用方。

### Lines 307-316 / 第 307-316 行

~~~~python
    dist.barrier(device_ids=[local_rank])
    avg_time_us = testing.benchmark(
        compiled_func,
        workspace_generator=generate_tensors,
        workspace_count=10,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
    )
    dist.barrier(device_ids=[local_rank])
    torch.cuda.synchronize()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 318-318 / 第 318-318 行

~~~~python
    # Print execution results
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 319-323 / 第 319-323 行

~~~~python
    if local_rank == 0:
        print(f"Kernel execution time: {avg_time_us / 1e3:.4f} ms")
        print(
            f"Achieved memory throughput: {((world_size + 1) * output_tensor.numel() * 32 // 8) / (avg_time_us / 1e6) / 1e9:.2f} GB/s"
        )
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 325-327 / 第 325-327 行

~~~~python
    for free_func, tensor in free_func_and_tensor_pairs:
        free_func(tensor)
    return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 330-338 / 第 330-338 行

~~~~python
def torchrun_uid_init_bcast():
    """
    Initialize NVSHMEM using UniqueID with `torchrun` as the launcher

    It uses torch.distributed.broadcast on a NumPy array to handle the broadcasting
    """
    # Set Torch device
    local_rank = int(os.environ['LOCAL_RANK'])
    torch.cuda.set_device(local_rank)
~~~~

**EN**: Defines `torchrun_uid_init_bcast`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 定义 `torchrun_uid_init_bcast`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 340-340 / 第 340-340 行

~~~~python
    # nvshmem4py requires a cuda.core Device at init time
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 341-344 / 第 341-344 行

~~~~python
    dev = Device(local_rank)
    dev.set_current()
    global stream
    stream = dev.create_stream()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 346-346 / 第 346-346 行

~~~~python
    # Initialize torch.distributed process group
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 347-349 / 第 347-349 行

~~~~python
    dist.init_process_group(
        backend="cpu:gloo,cuda:nccl",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 351-351 / 第 351-351 行

~~~~python
    # Extract rank, nranks from process group
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 352-352 / 第 352-352 行

~~~~python
    num_ranks = dist.get_world_size()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 354-354 / 第 354-354 行

~~~~python
    # Create an empty uniqueid for all ranks
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 355-360 / 第 355-360 行

~~~~python
    uid = nvshmem.core.get_unique_id(empty=(local_rank != 0))
    uid_bytes = uid._data.view(np.uint8).copy()
    uid_tensor = torch.from_numpy(uid_bytes).cuda()
    dist.broadcast(uid_tensor, src=0)
    dist.barrier()
    uid._data[:] = uid_tensor.cpu().numpy().view(uid._data.dtype)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 362-362 / 第 362-362 行

~~~~python
    nvshmem.core.init(device=dev, uid=uid, rank=local_rank, nranks=num_ranks, initializer_method="uid")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 365-367 / 第 365-367 行

~~~~python
def torchrun_finalize():
    nvshmem.core.finalize()
    dist.destroy_process_group()
~~~~

**EN**: Defines `torchrun_finalize`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 定义 `torchrun_finalize`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 370-379 / 第 370-379 行

~~~~python
def main():
    parser = argparse.ArgumentParser(
        description="example of elementwise add to demonstrate the numpy/pytorch as input for kernels"
    )
    parser.add_argument("--M", default=1024, type=int)
    parser.add_argument("--N", default=1024, type=int)
    parser.add_argument("--warmup_iterations", default=2, type=int)
    parser.add_argument("--iterations", default=10, type=int)
    parser.add_argument("--skip_ref_check", action="store_true")
    parser.add_argument("--benchmark", action="store_true")
~~~~

**EN**: Defines `main`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 定义 `main`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 381-381 / 第 381-381 行

~~~~python
    args = parser.parse_args()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 383-383 / 第 383-383 行

~~~~python
    torchrun_uid_init_bcast()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 385-385 / 第 385-385 行

~~~~python
    run_all_reduce_multimem(args.M, args.N, args.warmup_iterations, args.iterations, args.skip_ref_check, args.benchmark)
~~~~

**EN**: Measures performance or checks correctness so the example doubles as a validation harness. Implements or orchestrates multi-GPU communication, extending the example beyond a single device.
**CN**: 测量性能或检查正确性，使该示例同时充当验证工具。 实现或编排多 GPU 通信，把示例扩展到单设备之外。

### Lines 387-387 / 第 387-387 行

~~~~python
    torchrun_finalize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 389-389 / 第 389-389 行

~~~~python
    return
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 392-393 / 第 392-393 行

~~~~python
if __name__ == "__main__":
    main()
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出
- Multi-GPU communication / 多 GPU 通信

## Dependencies / 依赖项

- `os` — handles environment or file-system level host logic / 处理环境变量或文件系统级宿主逻辑
- `time` — measures host-side timing information / 测量宿主端时间信息
- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `numpy` — supplies host-side array utilities / 提供宿主端数组工具
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `torch.distributed` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cuda.core.experimental.Device` — exposes experimental CUDA Python runtime utilities / 提供实验性的 CUDA Python 运行时工具
- `cuda.pathfinder.load_nvidia_dynamic_lib` — used by this example / 供该示例使用
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.runtime.from_dlpack` — wraps framework tensors as CuTeDSL runtime tensors / 把框架张量包装为 CuTeDSL 运行时张量
- `nvshmem.core` — provides symmetric-memory communication across GPUs / 提供跨 GPU 的对称内存通信
