# rmsnorm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/rmsnorm/rmsnorm.py`  
**Purpose / 用途**: Kernel example implementing rmsnorm with CuTeDSL. / 这是一个使用 CuTeDSL 实现 rmsnorm 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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
import ctypes
import functools
import math
from typing import Optional, Tuple, Union

import cuda.bindings.driver as cuda
import torch

import cutlass
import cutlass.cute as cute
import cutlass.cute.testing as testing
import cutlass.torch as cutlass_torch
import cutlass.utils as utils
from cutlass import Boolean, Float32, Int32, Int64
from cutlass.cute.runtime import make_ptr
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 46-46 / 第 46-46 行

~~~~python
# Support both direct execution and module import
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 47-50 / 第 47-50 行

~~~~python
try:
    from .reduce import row_reduce
except ImportError:
    from reduce import row_reduce
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 52-98 / 第 52-98 行

~~~~python
"""
RMSNorm: Root Mean Square Layer Normalization for Hopper & Blackwell (SM90+)
====================================================================

A high-performance RMSNorm implementation using CuTe DSL with cluster-based
reduction for large hidden dimensions.

RMSNorm computes: y = x / sqrt(mean(x²) + eps) * weight

Key Features:
-------------
1. CLUSTER SYNCHRONIZATION (SM90+)
   - Multiple CTAs cooperate to process large N dimensions
   - Each CTA handles N/cluster_n elements, then reduces across the cluster
   - Uses mbarrier for efficient cross-CTA synchronization

2. ARCHITECTURE-SPECIFIC TUNING
   - SM80 (Ampere): Single-CTA execution (cluster_n=1)
   - SM90 (Hopper): Cluster support enabled for large N
   - SM100 (Blackwell): Same as SM90

3. VECTORIZED MEMORY ACCESS
   - 128-bit vectorized loads/stores for optimal memory throughput
   - TiledCopy abstraction for organized gmem↔smem↔rmem transfers

Cluster Size Selection (FP16):
------------------------------
- N <= 16K: cluster_n = 1 (single CTA)
- N <= 32K: cluster_n = 2
- N <= 64K: cluster_n = 4
- N <= 128K: cluster_n = 8
- Larger: cluster_n = 16

To run this example:

.. code-block:: bash

    python examples/python/CuTeDSL/blackwell/rmsnorm.py --M 2048 --N 4096 --dtype BFloat16
    python examples/python/CuTeDSL/blackwell/rmsnorm.py --M 2048 --N 4096 --dtype BFloat16 --benchmark
    python examples/python/CuTeDSL/blackwell/rmsnorm.py --M 2048 --N 32768 --dtype BFloat16 --benchmark

To collect performance with NCU profiler:

.. code-block:: bash

    ncu python examples/python/CuTeDSL/blackwell/rmsnorm.py --M 2048 --N 4096 --dtype BFloat16 --skip_ref_check
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 100-102 / 第 100-102 行

~~~~python
# =============================================================================
# Architecture Detection
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 105-111 / 第 105-111 行

~~~~python
@functools.lru_cache(maxsize=16)
def get_sm_version(device: Optional[Union[int, torch.device, str]] = None) -> int:
    """Get the SM (compute capability) version of a CUDA device."""
    if not torch.cuda.is_available():
        return 80  # Default fallback
    props = torch.cuda.get_device_properties(device)
    return props.major * 10 + props.minor
~~~~

**EN**: Defines `get_sm_version`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `get_sm_version`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 114-116 / 第 114-116 行

~~~~python
def supports_cluster() -> bool:
    """Check if the current device supports cluster operations (SM90+)."""
    return get_sm_version() >= 90
~~~~

**EN**: Defines `supports_cluster`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `supports_cluster`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 119-121 / 第 119-121 行

~~~~python
# =============================================================================
# Predicate Utility
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 124-137 / 第 124-137 行

~~~~python
@cute.jit
def predicate_k(tXcX: cute.Tensor, limit: int) -> cute.Tensor:
    """Create predicate tensor for bounds checking."""
    tXpX = cute.make_rmem_tensor(
        cute.make_layout(
            (cute.size(tXcX, mode=[0, 1]), cute.size(tXcX, mode=[1]), cute.size(tXcX, mode=[2])),
            stride=(cute.size(tXcX, mode=[2]), 0, 1),
        ),
        Boolean,
    )
    for rest_v in cutlass.range_constexpr(tXpX.shape[0]):
        for rest_k in cutlass.range_constexpr(tXpX.shape[2]):
            tXpX[rest_v, 0, rest_k] = cute.elem_less(tXcX[(0, rest_v), 0, rest_k][1], limit)
    return tXpX
~~~~

**EN**: Marks `predicate_k` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Returns the constructed object or computed result to the caller.
**CN**: 将 `predicate_k` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 把构造好的对象或计算结果返回给调用方。

### Lines 140-142 / 第 140-142 行

~~~~python
# =============================================================================
# RMSNorm Configuration Class
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 145-151 / 第 145-151 行

~~~~python
class RMSNormConfig:
    """
    Configuration for the RMSNorm kernel.

    This class encapsulates all kernel configuration computed once at initialization,
    following CuTe-DSL conventions from official CUTLASS examples.
    """
~~~~

**EN**: Defines `RMSNormConfig`, a reusable Python class that packages configuration and behavior for this example. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `RMSNormConfig`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 153-153 / 第 153-153 行

~~~~python
    COPY_BITS = 128  # 128-bit vectorized loads
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 155-165 / 第 155-165 行

~~~~python
    def __init__(
        self,
        dtype: type[cutlass.Numeric],
        N: int,
        has_weight: bool = True,
        sm_version: int | None = None,
    ):
        self.dtype = dtype
        self.N = N
        self.has_weight = has_weight
        self.sm_version = sm_version if sm_version is not None else get_sm_version()
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 167-167 / 第 167-167 行

~~~~python
        # Vector size for 128-bit loads
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 168-168 / 第 168-168 行

~~~~python
        self.vec_size = self.COPY_BITS // dtype.width
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 170-170 / 第 170-170 行

~~~~python
        # Compute cluster size (SM90+ only)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 171-171 / 第 171-171 行

~~~~python
        self.cluster_n = self._compute_cluster_n(N, dtype, self.sm_version)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 173-173 / 第 173-173 行

~~~~python
        # N per CTA for cluster case
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 174-174 / 第 174-174 行

~~~~python
        self.N_per_cta = N // self.cluster_n
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 176-176 / 第 176-176 行

~~~~python
        # Thread configuration using static methods
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 177-178 / 第 177-178 行

~~~~python
        self.threads_per_row = self._compute_threads_per_row(self.N_per_cta)
        self.num_threads = self._compute_num_threads(self.N_per_cta)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 180-180 / 第 180-180 行

~~~~python
        # Derived values
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 181-186 / 第 181-186 行

~~~~python
        self.num_vec_blocks = max(
            1, (self.N_per_cta // self.vec_size + self.threads_per_row - 1) // self.threads_per_row
        )
        self.rows_per_block = self.num_threads // self.threads_per_row
        self.cols_per_tile = self.vec_size * self.num_vec_blocks * self.threads_per_row
        self.warps_per_row = max(self.threads_per_row // 32, 1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 188-192 / 第 188-192 行

~~~~python
    @staticmethod
    def _compute_cluster_n(N: int, dtype: type[cutlass.Numeric], sm_version: int) -> int:
        """Compute optimal cluster size based on N and architecture."""
        if sm_version < 90:
            return 1
~~~~

**EN**: Defines `_compute_cluster_n`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_compute_cluster_n`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 194-215 / 第 194-215 行

~~~~python
        if dtype.width == 16:  # FP16/BF16
            if N <= 16 * 1024:
                return 1
            elif N <= 32 * 1024:
                return 2
            elif N <= 64 * 1024:
                return 4
            elif N <= 128 * 1024:
                return 8
            else:
                return 16
        else:  # FP32
            if N <= 32 * 1024:
                return 1
            elif N <= 64 * 1024:
                return 2
            elif N <= 128 * 1024:
                return 4
            elif N <= 256 * 1024:
                return 8
            else:
                return 16
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 217-231 / 第 217-231 行

~~~~python
    @staticmethod
    def _compute_threads_per_row(N_per_cta: int) -> int:
        """Compute optimal threads per row based on N per CTA."""
        if N_per_cta <= 64:
            return 8
        elif N_per_cta <= 128:
            return 16
        elif N_per_cta <= 3072:
            return 32
        elif N_per_cta <= 6144:
            return 64
        elif N_per_cta <= 16384:
            return 128
        else:
            return 256
~~~~

**EN**: Defines `_compute_threads_per_row`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_compute_threads_per_row`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 233-236 / 第 233-236 行

~~~~python
    @staticmethod
    def _compute_num_threads(N_per_cta: int) -> int:
        """Compute total threads per block."""
        return 128 if N_per_cta <= 16384 else 256
~~~~

**EN**: Defines `_compute_num_threads`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_compute_num_threads`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 238-254 / 第 238-254 行

~~~~python
    @staticmethod
    def _make_tv_layout(
        threads_per_row: int,
        rows_per_block: int,
        vec_size: int,
        num_vec_blocks: int,
    ) -> tuple:
        """Create Thread-Value layout for coalesced vectorized memory access."""
        shape = (
            (threads_per_row, rows_per_block),
            (vec_size, num_vec_blocks),
        )
        stride = (
            (vec_size * rows_per_block, 1),
            (rows_per_block, rows_per_block * vec_size * threads_per_row),
        )
        return shape, stride
~~~~

**EN**: Defines `_make_tv_layout`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `_make_tv_layout`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 256-261 / 第 256-261 行

~~~~python
    def smem_size_in_bytes(self) -> int:
        """Calculate shared memory requirement in bytes."""
        tile_bytes = self.rows_per_block * self.cols_per_tile * (self.dtype.width // 8)
        reduction_bytes = self.rows_per_block * self.warps_per_row * self.cluster_n * 4
        mbar_bytes = 8 if self.cluster_n > 1 else 0
        return tile_bytes + reduction_bytes + mbar_bytes
~~~~

**EN**: Defines `smem_size_in_bytes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `smem_size_in_bytes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 264-266 / 第 264-266 行

~~~~python
# =============================================================================
# RMSNorm Kernel Class
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 269-281 / 第 269-281 行

~~~~python
class RMSNormKernel:
    """
    RMSNorm kernel with cluster synchronization for large N.

    Features:
    - Cluster-based reduction for large N (SM90+)
    - Multiple CTAs cooperate via mbarrier
    - Single reduction (sum of squares) with cluster-level aggregation

    Example:
        >>> kernel = RMSNormKernel(cutlass.Float16, N=4096)
        >>> kernel(x_ptr, w_ptr, o_ptr, M, eps, stream)
    """
~~~~

**EN**: Defines `RMSNormKernel`, a reusable Python class that packages configuration and behavior for this example. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 定义 `RMSNormKernel`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 283-294 / 第 283-294 行

~~~~python
    def __init__(
        self,
        dtype: cutlass.Numeric,
        N: int,
        has_weight: bool = True,
        config: RMSNormConfig | None = None,
    ):
        # Use provided config or create new one
        if config is not None:
            self.cfg = config
        else:
            self.cfg = RMSNormConfig(dtype, N, has_weight)
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 296-296 / 第 296-296 行

~~~~python
        # Expose key attributes for convenience
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 297-300 / 第 297-300 行

~~~~python
        self.dtype = self.cfg.dtype
        self.N = self.cfg.N
        self.has_weight = self.cfg.has_weight
        self.cluster_n = self.cfg.cluster_n
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 302-313 / 第 302-313 行

~~~~python
    @cute.jit
    def __call__(
        self,
        x_ptr: cute.Pointer,
        w_ptr: cute.Pointer | None,
        o_ptr: cute.Pointer,
        M: Int32,
        eps: Float32,
        stream: cuda.CUstream,
    ):
        """Host function to launch the RMSNorm kernel."""
        cfg = self.cfg
~~~~

**EN**: Marks `__call__` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 将 `__call__` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 315-315 / 第 315-315 行

~~~~python
        # Create CuTe tensors from raw pointers
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 316-323 / 第 316-323 行

~~~~python
        mX = cute.make_tensor(
            x_ptr,
            cute.make_layout((M, cfg.N), stride=(cfg.N, 1)),
        )
        mO = cute.make_tensor(
            o_ptr,
            cute.make_layout((M, cfg.N), stride=(cfg.N, 1)),
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 325-331 / 第 325-331 行

~~~~python
        if cutlass.const_expr(cfg.has_weight and w_ptr is not None):
            mW = cute.make_tensor(
                w_ptr,
                cute.make_layout((cfg.N,), stride=(1,)),
            )
        else:
            mW = None
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 333-333 / 第 333-333 行

~~~~python
        # Create TV layout using static helper
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 334-341 / 第 334-341 行

~~~~python
        tv_shape, tv_stride = RMSNormConfig._make_tv_layout(
            cfg.threads_per_row,
            cfg.rows_per_block,
            cfg.vec_size,
            cfg.num_vec_blocks,
        )
        tv_layout = cute.make_layout(tv_shape, stride=tv_stride)
        tiler_mn = (cfg.rows_per_block, cfg.cols_per_tile)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 343-349 / 第 343-349 行

~~~~python
        self.kernel(mX, mW, mO, eps, tv_layout, tiler_mn).launch(
            grid=[cute.ceil_div(M, cfg.rows_per_block), cfg.cluster_n, 1],
            block=[cfg.num_threads, 1, 1],
            cluster=[1, cfg.cluster_n, 1] if cutlass.const_expr(cfg.cluster_n > 1) else None,
            smem=cfg.smem_size_in_bytes(),
            stream=stream,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 351-364 / 第 351-364 行

~~~~python
    @cute.kernel
    def kernel(
        self,
        mX: cute.Tensor,
        mW: cute.Tensor | None,
        mO: cute.Tensor,
        eps: Float32,
        tv_layout: cute.Layout,
        tiler_mn: cute.Shape,
    ):
        """Device kernel implementing RMSNorm with cluster support."""
        cfg = self.cfg
        tidx, _, _ = cute.arch.thread_idx()
        bidx, _, _ = cute.arch.block_idx()
~~~~

**EN**: Declares `kernel` as a CuTeDSL kernel, so the Python body becomes device code with CUDA-style thread and block semantics after compilation. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 把 `kernel` 声明为 CuTeDSL 内核，因此其 Python 主体在编译后会变成具有 CUDA 风格线程块语义的设备端代码。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 366-369 / 第 366-369 行

~~~~python
        if cutlass.const_expr(cfg.cluster_n > 1):
            cluster_y = cute.arch.block_idx()[1]
        else:
            cluster_y = cutlass.const_expr(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Reads CUDA execution coordinates or elects a single participating thread, tying this Python DSL code directly to GPU thread/block behavior. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 读取 CUDA 执行坐标，或选出一个参与线程，把这段 Python DSL 代码直接绑定到 GPU 的线程/线程块行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 371-374 / 第 371-374 行

~~~~python
        M = mX.shape[0]
        threads_per_row = tv_layout.shape[0][0]
        warps_per_row = max(threads_per_row // 32, 1)
        rows_per_block = tiler_mn[0]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 376-378 / 第 376-378 行

~~~~python
        # =====================================================================
        # Allocate shared memory
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 379-379 / 第 379-379 行

~~~~python
        smem = utils.SmemAllocator()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 381-385 / 第 381-385 行

~~~~python
        sX = smem.allocate_tensor(
            mX.element_type,
            cute.make_ordered_layout(tiler_mn, order=(1, 0)),
            byte_alignment=16,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 387-400 / 第 387-400 行

~~~~python
        if cutlass.const_expr(cfg.cluster_n == 1):
            reduction_buffer = smem.allocate_tensor(
                Float32,
                cute.make_layout((rows_per_block, warps_per_row)),
                byte_alignment=4,
            )
            mbar_ptr = None
        else:
            reduction_buffer = smem.allocate_tensor(
                Float32,
                cute.make_layout((rows_per_block, (warps_per_row, cfg.cluster_n))),
                byte_alignment=4,
            )
            mbar_ptr = smem.allocate_array(Int64, num_elems=1)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 402-404 / 第 402-404 行

~~~~python
        # =====================================================================
        # Initialize cluster
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 405-410 / 第 405-410 行

~~~~python
        if cutlass.const_expr(cfg.cluster_n > 1):
            if tidx == 0:
                cute.arch.mbarrier_init(mbar_ptr, 1)
            cute.arch.mbarrier_init_fence()
            cute.arch.cluster_arrive_relaxed()
            cute.arch.cluster_wait()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 412-414 / 第 412-414 行

~~~~python
        # =====================================================================
        # Create identity tensor and partition
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 415-415 / 第 415-415 行

~~~~python
        idX = cute.make_identity_tensor(mX.shape)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 417-419 / 第 417-419 行

~~~~python
        gX = cute.local_tile(mX, tiler_mn, (bidx, cluster_y))
        gO = cute.local_tile(mO, tiler_mn, (bidx, cluster_y))
        cX = cute.local_tile(idX, tiler_mn, (bidx, cluster_y))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 421-426 / 第 421-426 行

~~~~python
        if cutlass.const_expr(cfg.has_weight and mW is not None):
            mW_expanded_layout = cute.prepend(
                mW.layout, cute.make_layout((tiler_mn[0],), stride=(0,))
            )
            mW_2d = cute.make_tensor(mW.iterator, mW_expanded_layout)
            gW = cute.local_tile(mW_2d, tiler_mn, (0, cluster_y))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Partitions large tensors into CTA- or thread-sized tiles, which is the core idea behind CUTLASS/CuTe tiling.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 把大张量切分为 CTA 或线程粒度的子块，这是 CUTLASS/CuTe 分块思想的核心。

### Lines 428-430 / 第 428-430 行

~~~~python
        # =====================================================================
        # Create TiledCopy operations
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 431-435 / 第 431-435 行

~~~~python
        copy_atom_load_async = cute.make_copy_atom(
            cute.nvgpu.cpasync.CopyG2SOp(),
            mX.element_type,
            num_bits_per_copy=RMSNormConfig.COPY_BITS,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 437-441 / 第 437-441 行

~~~~python
        copy_atom_load_W = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(),
            mX.element_type,
            num_bits_per_copy=RMSNormConfig.COPY_BITS,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 443-447 / 第 443-447 行

~~~~python
        copy_atom_store = cute.make_copy_atom(
            cute.nvgpu.CopyUniversalOp(),
            mO.element_type,
            num_bits_per_copy=RMSNormConfig.COPY_BITS,
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。

### Lines 449-451 / 第 449-451 行

~~~~python
        tiled_copy_load = cute.make_tiled_copy(copy_atom_load_async, tv_layout, tiler_mn)
        tiled_copy_W = cute.make_tiled_copy(copy_atom_load_W, tv_layout, tiler_mn)
        tiled_copy_store = cute.make_tiled_copy(copy_atom_store, tv_layout, tiler_mn)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 453-455 / 第 453-455 行

~~~~python
        thr_copy_X = tiled_copy_load.get_slice(tidx)
        thr_copy_W = tiled_copy_W.get_slice(tidx)
        thr_copy_O = tiled_copy_store.get_slice(tidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 457-457 / 第 457-457 行

~~~~python
        # Partition tensors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 458-461 / 第 458-461 行

~~~~python
        tXgX = thr_copy_X.partition_S(gX)
        tXsX = thr_copy_X.partition_D(sX)
        tXgO = thr_copy_O.partition_D(gO)
        tXcX = thr_copy_X.partition_S(cX)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 463-463 / 第 463-463 行

~~~~python
        # Register fragments
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 464-465 / 第 464-465 行

~~~~python
        tXrX = cute.make_fragment_like(tXgX)
        tXrO = cute.make_fragment_like(tXgO)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 467-470 / 第 467-470 行

~~~~python
        if cutlass.const_expr(cfg.has_weight and mW is not None):
            tWgW = thr_copy_W.partition_S(gW)
            tWrW = cute.make_fragment_like(tWgW)
            tXrW = thr_copy_X.retile(tWrW)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。

### Lines 472-474 / 第 472-474 行

~~~~python
        # =====================================================================
        # Bounds checking
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 475-475 / 第 475-475 行

~~~~python
        tXpX = predicate_k(tXcX, limit=cfg.N)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 477-478 / 第 477-478 行

~~~~python
        row_coord = tXcX[(0, 0), 0, 0]
        row_in_bounds = row_coord[0] < M
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 480-482 / 第 480-482 行

~~~~python
        # =====================================================================
        # Async copy global → shared
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 483-484 / 第 483-484 行

~~~~python
        if row_in_bounds:
            cute.copy(copy_atom_load_async, tXgX, tXsX, pred=tXpX)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 486-486 / 第 486-486 行

~~~~python
        cute.arch.cp_async_commit_group()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 488-488 / 第 488-488 行

~~~~python
        # Load weight while waiting
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 489-491 / 第 489-491 行

~~~~python
        if cutlass.const_expr(cfg.has_weight and mW is not None):
            tWpW = predicate_k(thr_copy_W.partition_S(cX), limit=cfg.N)
            cute.copy(copy_atom_load_W, tWgW, tWrW, pred=tWpW)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 493-493 / 第 493-493 行

~~~~python
        cute.arch.cp_async_wait_group(0)
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 495-497 / 第 495-497 行

~~~~python
        # =====================================================================
        # Pass 1: Compute sum of squares with cluster reduction
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 498-499 / 第 498-499 行

~~~~python
        cute.autovec_copy(tXsX, tXrX)
        x = tXrX.load().to(Float32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 501-510 / 第 501-510 行

~~~~python
        x_sq = x * x
        sum_sq = row_reduce(
            x_sq,
            cute.ReductionOp.ADD,
            threads_per_row,
            reduction_buffer,
            mbar_ptr,
            cfg.cluster_n,
            Float32(0.0),
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 512-512 / 第 512-512 行

~~~~python
        # rstd = 1 / sqrt(mean(x²) + eps)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 513-514 / 第 513-514 行

~~~~python
        mean_sq = sum_sq / cfg.N
        rstd = cute.math.rsqrt(mean_sq + eps, fastmath=True)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 516-516 / 第 516-516 行

~~~~python
        # Sync after reduction
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 517-521 / 第 517-521 行

~~~~python
        if cutlass.const_expr(cfg.cluster_n > 1):
            cute.arch.cluster_arrive_relaxed()
            cute.arch.cluster_wait()
        else:
            cute.arch.barrier()
~~~~

**EN**: Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 523-525 / 第 523-525 行

~~~~python
        # =====================================================================
        # Pass 2: Normalize and output
        # =====================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 526-527 / 第 526-527 行

~~~~python
        cute.autovec_copy(tXsX, tXrX)
        x = tXrX.load().to(Float32)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 529-529 / 第 529-529 行

~~~~python
        y = x * rstd
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 531-531 / 第 531-531 行

~~~~python
        # Apply weight if present
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 532-534 / 第 532-534 行

~~~~python
        if cutlass.const_expr(cfg.has_weight and mW is not None):
            w = tXrW.load().to(Float32)
            y = y * w
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 536-536 / 第 536-536 行

~~~~python
        # Store to global memory
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 537-537 / 第 537-537 行

~~~~python
        tXrO.store(y.to(cfg.dtype))
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 539-540 / 第 539-540 行

~~~~python
        if row_in_bounds:
            cute.copy(copy_atom_store, tXrO, tXgO, pred=tXpX)
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 543-547 / 第 543-547 行

~~~~python
# =============================================================================
# Kernel Compilation and Caching
# =============================================================================

# Mapping from torch dtype to cutlass dtype
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 548-552 / 第 548-552 行

~~~~python
_torch_to_cutlass_dtype = {
    torch.float16: cutlass.Float16,
    torch.bfloat16: cutlass.BFloat16,
    torch.float32: cutlass.Float32,
}
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 554-554 / 第 554-554 行

~~~~python
# Cache for compiled kernels
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 555-555 / 第 555-555 行

~~~~python
_compile_cache: dict = {}
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 558-581 / 第 558-581 行

~~~~python
def get_compiled_kernel(
    dtype: type[cutlass.Numeric],
    N: int,
    has_weight: bool,
    stream: cuda.CUstream,
):
    """
    Get or compile the RMSNorm kernel for the given configuration.

    Uses compilation cache to avoid recompiling for same (dtype, N, has_weight) tuples.

    :param dtype: Data type (Float16, BFloat16, Float32)
    :type dtype: type[cutlass.Numeric]
    :param N: Hidden dimension size
    :type N: int
    :param has_weight: Whether weight is applied
    :type has_weight: bool
    :param stream: CUDA stream
    :type stream: cuda.CUstream
    :return: Compiled kernel function
    """
    key = (dtype, N, has_weight)
    if key not in _compile_cache:
        kernel_obj = RMSNormKernel(dtype, N, has_weight)
~~~~

**EN**: Defines `get_compiled_kernel`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `get_compiled_kernel`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 583-583 / 第 583-583 行

~~~~python
        # Compile with representative arguments
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 584-594 / 第 584-594 行

~~~~python
        compiled_kernel = cute.compile(
            kernel_obj,
            make_ptr(dtype, 16, cute.AddressSpace.gmem, assumed_align=16),  # x_ptr
            make_ptr(dtype, 16, cute.AddressSpace.gmem, assumed_align=16)
            if has_weight
            else None,  # w_ptr
            make_ptr(dtype, 16, cute.AddressSpace.gmem, assumed_align=16),  # o_ptr
            Int32(1),  # M (dummy)
            Float32(1e-6),  # eps (dummy)
            stream,
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 596-596 / 第 596-596 行

~~~~python
        _compile_cache[key] = compiled_kernel
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 598-598 / 第 598-598 行

~~~~python
    return _compile_cache[key]
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 601-603 / 第 601-603 行

~~~~python
# =============================================================================
# Tensor Creation Utilities
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 606-614 / 第 606-614 行

~~~~python
def create_tensors(
    M: int,
    N: int,
    dtype: type[cutlass.Numeric],
    has_weight: bool,
) -> Tuple:
    """Create input, weight, and output tensors for RMSNorm."""
    torch.manual_seed(42)
    torch_dtype = cutlass_torch.dtype(dtype)
~~~~

**EN**: Defines `create_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 定义 `create_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 616-618 / 第 616-618 行

~~~~python
    x = torch.randn(M, N, device="cuda", dtype=torch_dtype)
    weight = torch.randn(N, device="cuda", dtype=torch_dtype) if has_weight else None
    out = torch.empty_like(x)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 620-620 / 第 620-620 行

~~~~python
    return x, weight, out
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 623-634 / 第 623-634 行

~~~~python
def rmsnorm_ref(
    x: torch.Tensor,
    weight: torch.Tensor | None = None,
    eps: float = 1e-6,
) -> torch.Tensor:
    """Reference RMSNorm implementation in PyTorch."""
    x_f32 = x.float()
    rms = torch.sqrt(torch.mean(x_f32**2, dim=-1, keepdim=True) + eps)
    x_norm = x_f32 / rms
    if weight is not None:
        x_norm = x_norm * weight.float()
    return x_norm.to(x.dtype)
~~~~

**EN**: Defines `rmsnorm_ref`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 定义 `rmsnorm_ref`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 637-639 / 第 637-639 行

~~~~python
# =============================================================================
# Run Function
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 642-669 / 第 642-669 行

~~~~python
def run(
    M: int,
    N: int,
    dtype: type[cutlass.Numeric],
    has_weight: bool = True,
    eps: float = 1e-6,
    tolerance: float = 1e-2,
    warmup_iterations: int = 2,
    iterations: int = 100,
    skip_ref_check: bool = False,
    benchmark: bool = False,
) -> float:
    """
    Execute RMSNorm and optionally benchmark performance.

    :param M: Number of rows (batch size * sequence length)
    :type M: int
    :param N: Hidden dimension size
    :type N: int
    :param dtype: Data type (Float16, BFloat16, Float32)
    :type dtype: type[cutlass.Numeric]
    :param has_weight: Whether to apply learnable weight
    :type has_weight: bool
    :param eps: Epsilon for numerical stability
    :type eps: float
    :param tolerance: Tolerance for correctness check
    :type tolerance: float
    :param warmup_iterations: Warmup iterations for benchmarking
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 670-685 / 第 670-685 行

~~~~python
    :type warmup_iterations: int
    :param iterations: Number of benchmark iterations
    :type iterations: int
    :param skip_ref_check: Skip reference correctness check
    :type skip_ref_check: bool
    :param benchmark: Enable benchmarking
    :type benchmark: bool
    :return: Execution time in microseconds (if benchmark=True, else 0)
    :rtype: float
    """
    print("Running RMSNorm test with:")
    print(f"  M: {M}, N: {N}")
    print(f"  dtype: {dtype}")
    print(f"  has_weight: {has_weight}")
    print(f"  eps: {eps}")
    print(f"  SM version: {get_sm_version()}")
~~~~

**EN**: Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 687-688 / 第 687-688 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("CUDA GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 690-690 / 第 690-690 行

~~~~python
    # Get CUDA stream
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 691-692 / 第 691-692 行

~~~~python
    torch_stream = torch.cuda.current_stream()
    stream = cuda.CUstream(torch_stream.cuda_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 694-694 / 第 694-694 行

~~~~python
    # Create tensors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 695-695 / 第 695-695 行

~~~~python
    x, weight, out = create_tensors(M, N, dtype, has_weight)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 697-697 / 第 697-697 行

~~~~python
    # Get configuration info
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 698-701 / 第 698-701 行

~~~~python
    config = RMSNormConfig(dtype, N, has_weight)
    print(f"  cluster_n: {config.cluster_n}")
    print(f"  threads_per_row: {config.threads_per_row}")
    print(f"  rows_per_block: {config.rows_per_block}")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 703-703 / 第 703-703 行

~~~~python
    # Get compiled kernel
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 704-704 / 第 704-704 行

~~~~python
    compiled_kernel = get_compiled_kernel(dtype, N, has_weight, stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 706-706 / 第 706-706 行

~~~~python
    # Create pointers for kernel call
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 707-709 / 第 707-709 行

~~~~python
    x_ptr = make_ptr(dtype, x.data_ptr())
    w_ptr = make_ptr(dtype, weight.data_ptr()) if weight is not None else None
    out_ptr = make_ptr(dtype, out.data_ptr())
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 711-711 / 第 711-711 行

~~~~python
    # Run kernel and verify
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 712-714 / 第 712-714 行

~~~~python
    if not skip_ref_check:
        compiled_kernel(x_ptr, w_ptr, out_ptr, Int32(M), Float32(eps), stream)
        torch.cuda.synchronize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 716-718 / 第 716-718 行

~~~~python
        ref = rmsnorm_ref(x, weight, eps)
        torch.testing.assert_close(out, ref, atol=tolerance, rtol=tolerance)
        print("Correctness check passed!")
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 720-721 / 第 720-721 行

~~~~python
    if not benchmark:
        return 0.0
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 723-723 / 第 723-723 行

~~~~python
    # Benchmark
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 724-724 / 第 724-724 行

~~~~python
    print(f"\nBenchmarking with {warmup_iterations} warmup, {iterations} iterations...")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 726-731 / 第 726-731 行

~~~~python
    def generate_tensors():
        x, weight, out = create_tensors(M, N, dtype, has_weight)
        x_ptr = make_ptr(dtype, x.data_ptr())
        w_ptr = make_ptr(dtype, weight.data_ptr()) if weight is not None else None
        out_ptr = make_ptr(dtype, out.data_ptr())
        return testing.JitArguments(x_ptr, w_ptr, out_ptr, Int32(M), Float32(eps), stream)
~~~~

**EN**: Defines `generate_tensors`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `generate_tensors`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 733-740 / 第 733-740 行

~~~~python
    exec_time_us = testing.benchmark(
        compiled_kernel,
        workspace_generator=generate_tensors,
        workspace_count=10,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
        stream=stream,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 742-742 / 第 742-742 行

~~~~python
    # Calculate throughput
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 743-747 / 第 743-747 行

~~~~python
    torch_dtype = cutlass_torch.dtype(dtype)
    bytes_per_elem = torch.tensor([], dtype=torch_dtype).element_size()
    total_bytes = M * N * bytes_per_elem * 2  # read x, write out
    if has_weight:
        total_bytes += N * bytes_per_elem  # read weight (amortized across M)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 749-749 / 第 749-749 行

~~~~python
    throughput_gbps = (total_bytes / (exec_time_us / 1e6)) / 1e9
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 751-752 / 第 751-752 行

~~~~python
    print(f"Kernel execution time: {exec_time_us:.2f} us")
    print(f"Memory throughput: {throughput_gbps:.2f} GB/s")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 754-754 / 第 754-754 行

~~~~python
    return exec_time_us
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 757-759 / 第 757-759 行

~~~~python
# =============================================================================
# Main Entry Point
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 762-765 / 第 762-765 行

~~~~python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="RMSNorm kernel example for Blackwell (SM100)"
    )
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 767-791 / 第 767-791 行

~~~~python
    parser.add_argument("--M", type=int, default=2048, help="Number of rows")
    parser.add_argument("--N", type=int, default=4096, help="Hidden dimension size")
    parser.add_argument(
        "--dtype",
        type=cutlass.dtype,
        default=cutlass.BFloat16,
        help="Data type (Float16, BFloat16, Float32)",
    )
    parser.add_argument(
        "--has_weight",
        action="store_true",
        default=True,
        help="Apply learnable weight",
    )
    parser.add_argument(
        "--no_weight",
        action="store_true",
        help="Disable learnable weight",
    )
    parser.add_argument(
        "--eps",
        type=float,
        default=1e-6,
        help="Epsilon for numerical stability",
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 792-819 / 第 792-819 行

~~~~python
    parser.add_argument(
        "--tolerance",
        type=float,
        default=1e-2,
        help="Tolerance for correctness check",
    )
    parser.add_argument(
        "--warmup_iterations",
        type=int,
        default=2,
        help="Warmup iterations for benchmarking",
    )
    parser.add_argument(
        "--iterations",
        type=int,
        default=100,
        help="Number of benchmark iterations",
    )
    parser.add_argument(
        "--skip_ref_check",
        action="store_true",
        help="Skip reference correctness check",
    )
    parser.add_argument(
        "--benchmark",
        action="store_true",
        help="Enable benchmarking",
    )
~~~~

**EN**: Builds the command-line interface used to configure and launch the example from Python. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 构建命令行接口，用于从 Python 配置并启动该示例。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 821-821 / 第 821-821 行

~~~~python
    args = parser.parse_args()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。

### Lines 823-823 / 第 823-823 行

~~~~python
    # Handle weight flag
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 824-824 / 第 824-824 行

~~~~python
    has_weight = args.has_weight and not args.no_weight
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 826-837 / 第 826-837 行

~~~~python
    run(
        M=args.M,
        N=args.N,
        dtype=args.dtype,
        has_weight=has_weight,
        eps=args.eps,
        tolerance=args.tolerance,
        warmup_iterations=args.warmup_iterations,
        iterations=args.iterations,
        skip_ref_check=args.skip_ref_check,
        benchmark=args.benchmark,
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 839-839 / 第 839-839 行

~~~~python
    print("PASS")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Ampere GPU execution model / Ampere GPU 执行模型
- Hopper GPU execution model / Hopper GPU 执行模型
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Multistage pipeline and synchronization / 多级流水线与同步

## Dependencies / 依赖项

- `argparse` — parses command-line arguments for the example / 解析示例的命令行参数
- `ctypes` — used by this example / 供该示例使用
- `functools` — supplies small Python helpers such as partial application / 提供 partial 等 Python 辅助能力
- `math` — used by this example / 供该示例使用
- `typing.Optional` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Union` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.utils` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.Boolean` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.Float32` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.Int32` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.Int64` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.runtime.make_ptr` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `reduce.row_reduce` — used by this example / 供该示例使用
