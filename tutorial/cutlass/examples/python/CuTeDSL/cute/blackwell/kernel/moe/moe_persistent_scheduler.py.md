# moe_persistent_scheduler.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/moe/moe_persistent_scheduler.py`  
**Purpose / 用途**: MoE Persistent Tile Scheduler. / 这是一个使用 CuTeDSL 实现 moe persistent scheduler 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-45 / 第 29-45 行

~~~~python
"""
MoE Persistent Tile Scheduler

A specialized tile scheduler for MoE (Mixture of Experts) grouped GEMM operations.
This scheduler handles tile iteration across all experts, producing MoEWorkTileInfo
(expert_idx, tile_m_idx, tile_n_idx, k_tile_cnt) for each tile.

Scenarios:
- 2Dx3D (Forward): A(tokens_sum, hidden) x B(experts, intermediate, hidden) -> C(tokens_sum, intermediate)
- 2Dx2D (Backward): A(intermediate, tokens_sum) x B(hidden, tokens_sum) -> C(experts, intermediate, hidden)

Key design principle:
- Scheduler is ONLY responsible for tile iteration (tensor-agnostic, TMA-agnostic)
- Domain conversion (fake tensor -> real expert tensor) is handled by MoESchedExtension
- TMA descriptor management is handled by OnlineTensormapDescCreator
- The kernel orchestrates all three components
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 47-51 / 第 47-51 行

~~~~python
from typing import List, Tuple, Literal

import cutlass
import cutlass.cute as cute
from cutlass.cutlass_dsl import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 52-60 / 第 52-60 行

~~~~python
    Boolean,
    Int32,
    Integer,
    extract_mlir_values,
    new_from_mlir_values,
    const_expr,
    dsl_user_op,
)
from cutlass._mlir import ir
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 62-64 / 第 62-64 行

~~~~python
# =============================================================================
# Work Tile Info
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 66-84 / 第 66-84 行

~~~~python
class MoEWorkTileInfo:
    """
    Work tile information for MoE scheduler.
    
    Contains CTA-level tile information for executor warps:
    - expert_idx: Which expert (-1 means invalid/done)
    - tile_m_idx: CTA tile index along GEMM M dimension
    - tile_n_idx: CTA tile index along GEMM N dimension  
    - k_tile_cnt: Number of CTA tiles along K dimension
    
    Note: These are CTA-level indices, not cluster-level.
    tile_l_idx is always 0 for MoE, executor can hardcode it.
    
    For 2Dx3D (Forward):
        M = tokens_i (dynamic), N = intermediate (fixed), K = hidden (fixed)
    
    For 2Dx2D (Backward):
        M = intermediate (fixed), N = hidden (fixed), K = tokens_i (dynamic)
    """
~~~~

**EN**: Defines `MoEWorkTileInfo`, a reusable Python class that packages configuration and behavior for this example. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `MoEWorkTileInfo`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 86-96 / 第 86-96 行

~~~~python
    def __init__(
        self,
        expert_idx: Int32,      # -1 means invalid tile
        tile_m_idx: Int32,
        tile_n_idx: Int32,
        k_tile_cnt: Int32,
    ):
        self.expert_idx = expert_idx
        self.tile_m_idx = tile_m_idx
        self.tile_n_idx = tile_n_idx
        self.k_tile_cnt = k_tile_cnt
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 98-101 / 第 98-101 行

~~~~python
    @property
    def is_valid_tile(self) -> Boolean:
        """Check if this is a valid work tile (expert_idx >= 0)."""
        return self.expert_idx >= Int32(0)
~~~~

**EN**: Defines `is_valid_tile`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `is_valid_tile`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 103-108 / 第 103-108 行

~~~~python
    def __extract_mlir_values__(self) -> List[ir.Value]:
        values = extract_mlir_values(self.expert_idx)
        values.extend(extract_mlir_values(self.tile_m_idx))
        values.extend(extract_mlir_values(self.tile_n_idx))
        values.extend(extract_mlir_values(self.k_tile_cnt))
        return values
~~~~

**EN**: Defines `__extract_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__extract_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 110-117 / 第 110-117 行

~~~~python
    def __new_from_mlir_values__(self, values: List[ir.Value]) -> "MoEWorkTileInfo":
        assert len(values) == 4
        return MoEWorkTileInfo(
            expert_idx=new_from_mlir_values(self.expert_idx, [values[0]]),
            tile_m_idx=new_from_mlir_values(self.tile_m_idx, [values[1]]),
            tile_n_idx=new_from_mlir_values(self.tile_n_idx, [values[2]]),
            k_tile_cnt=new_from_mlir_values(self.k_tile_cnt, [values[3]]),
        )
~~~~

**EN**: Defines `__new_from_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__new_from_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 119-126 / 第 119-126 行

~~~~python
    def to_rmem_tensor(self):
        """Pack work tile info fields into an rmem tensor of shape (4,) for vectorized smem copy."""
        rmem = cute.make_rmem_tensor((4,), Int32)
        rmem[0] = self.expert_idx
        rmem[1] = self.tile_m_idx
        rmem[2] = self.tile_n_idx
        rmem[3] = self.k_tile_cnt
        return rmem
~~~~

**EN**: Defines `to_rmem_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Creates per-thread or per-warp tensor fragments in shared memory, registers, or tensor memory for the next compute stage. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `to_rmem_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 在共享内存、寄存器或张量内存中创建线程级/warp 级片段，为下一阶段计算做准备。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 128-136 / 第 128-136 行

~~~~python
    @staticmethod
    def from_rmem_tensor(rmem) -> "MoEWorkTileInfo":
        """Unpack work tile info from an rmem tensor of shape (4,)."""
        return MoEWorkTileInfo(
            expert_idx=rmem[0],  # type: ignore[arg-type]
            tile_m_idx=rmem[1],  # type: ignore[arg-type]
            tile_n_idx=rmem[2],  # type: ignore[arg-type]
            k_tile_cnt=rmem[3],  # type: ignore[arg-type]
        )
~~~~

**EN**: Defines `from_rmem_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `from_rmem_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 139-141 / 第 139-141 行

~~~~python
# =============================================================================
# Scheduler Parameters
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 143-161 / 第 143-161 行

~~~~python
class MoEStaticSchedulerParams:
    """
    Parameters for MoE tile scheduler.
    
    Uses unified semantics for both scenarios:
    - expert_shape: (expert_cnt, intermediate, hidden)
    
    For 2Dx3D: GEMM is (M=tokens_i, N=intermediate, K=hidden) per expert
    For 2Dx2D: GEMM is (M=hidden, N=intermediate, K=tokens_i) per expert
    
    Tile hierarchy:
    - cta_tile_shape_mnk: Single CTA tile shape (tile_m, tile_n, tile_k)
    - cluster_shape_mn: CTAs per cluster (cluster_m, cluster_n)
    - cluster_tile_shape_mn: Cluster tile shape = cta_tile_shape * cluster_shape
    
    This class is used both on host (for grid shape calculation) and on device
    (stored in scheduler). Codegen-time constants (scenario, cta_tile_shape_mnk,
    cluster_shape_mn) are NOT serialized to MLIR values.
    """
~~~~

**EN**: Defines `MoEStaticSchedulerParams`, a reusable Python class that packages configuration and behavior for this example. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `MoEStaticSchedulerParams`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 163-176 / 第 163-176 行

~~~~python
    def __init__(
        self,
        scenario: Literal["2Dx3D", "2Dx2D"],
        expert_shape: Tuple[int | Int32, int | Int32, int | Int32],  # (expert_cnt, intermediate, hidden)
        cta_tile_shape_mnk: Tuple[int, int, int],  # (tile_m, tile_n, tile_k)
        cluster_shape_mn: Tuple[int, int],         # (cluster_m, cluster_n)
    ):
        self.scenario = scenario
        e, i, h = expert_shape
        self.expert_cnt = e if isinstance(e, Int32) else Int32(e)
        self.intermediate = i if isinstance(i, Int32) else Int32(i)
        self.hidden = h if isinstance(h, Int32) else Int32(h)
        self.cta_tile_shape_mnk = cta_tile_shape_mnk
        self.cluster_shape_mn = cluster_shape_mn
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 178-181 / 第 178-181 行

~~~~python
    @property
    def cluster_tile_m(self) -> int:
        """Cluster tile size along M = cta_tile_m * cluster_m."""
        return self.cta_tile_shape_mnk[0] * self.cluster_shape_mn[0]
~~~~

**EN**: Defines `cluster_tile_m`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cluster_tile_m`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 183-186 / 第 183-186 行

~~~~python
    @property
    def cluster_tile_n(self) -> int:
        """Cluster tile size along N = cta_tile_n * cluster_n."""
        return self.cta_tile_shape_mnk[1] * self.cluster_shape_mn[1]
~~~~

**EN**: Defines `cluster_tile_n`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cluster_tile_n`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 188-191 / 第 188-191 行

~~~~python
    @property
    def cta_tile_k(self) -> int:
        """CTA tile size along K (same as cluster since cluster_k = 1)."""
        return self.cta_tile_shape_mnk[2]
~~~~

**EN**: Defines `cta_tile_k`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cta_tile_k`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 193-199 / 第 193-199 行

~~~~python
    def __extract_mlir_values__(self) -> List[ir.Value]:
        """Only serialize runtime values, not codegen-time constants."""
        values = []
        values.extend(extract_mlir_values(self.expert_cnt))
        values.extend(extract_mlir_values(self.intermediate))
        values.extend(extract_mlir_values(self.hidden))
        return values
~~~~

**EN**: Defines `__extract_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__extract_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 201-212 / 第 201-212 行

~~~~python
    def __new_from_mlir_values__(self, values: List[ir.Value]) -> "MoEStaticSchedulerParams":
        assert len(values) == 3
        return MoEStaticSchedulerParams(
            scenario=self.scenario,
            expert_shape=(
                new_from_mlir_values(self.expert_cnt, [values[0]]),
                new_from_mlir_values(self.intermediate, [values[1]]),
                new_from_mlir_values(self.hidden, [values[2]]),
            ),
            cta_tile_shape_mnk=self.cta_tile_shape_mnk,
            cluster_shape_mn=self.cluster_shape_mn,
        )
~~~~

**EN**: Defines `__new_from_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__new_from_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 214-230 / 第 214-230 行

~~~~python
    @staticmethod
    def get_grid_shape(
        params: "MoEStaticSchedulerParams",
        max_active_clusters: int,
    ) -> Tuple[int, int, int]:
        """
        Compute grid shape for kernel launch.
        
        Since host doesn't know token distribution across experts,
        we launch max_active_clusters and let device-side scheduler
        determine which tiles are valid.
        """
        return (
            params.cluster_shape_mn[0],
            params.cluster_shape_mn[1],
            max_active_clusters,
        )
~~~~

**EN**: Defines `get_grid_shape`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `get_grid_shape`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 233-235 / 第 233-235 行

~~~~python
# =============================================================================
# Scheduler (Device-side)
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 237-264 / 第 237-264 行

~~~~python
class MoEStaticPersistentTileScheduler:
    """
    Persistent tile scheduler specialized for MoE grouped GEMM.

    This scheduler is ONLY responsible for tile iteration. It does NOT know
    about tensor types, TMA descriptors, or domain conversion. Those concerns
    are handled by MoESchedExtension and OnlineTensormapDescCreator respectively.

    Architecture:
    - Scheduler warp: Holds scheduler instance, iterates tiles, broadcasts work_tile_info
    - Executor warps: Read work_tile_info from smem, use MoESchedExtension for
      domain conversion and TMA desc selection

    The scheduler handles:
    - 2Dx3D: Dynamic M per expert (from offs), fixed N (intermediate) and K (hidden)
    - 2Dx2D: Fixed M (intermediate) and N (hidden), dynamic K per expert (reduction axis)

    Usage (Scheduler warp):
        scheduler = MoEStaticPersistentTileScheduler.create(params, offs, block_idx, grid_dim)
        work_tile_info = scheduler.initial_work_tile_info()
        # Broadcast work_tile_info to smem...

        while work_tile_info.is_valid_tile:
            # ... do work ...
            work_tile_info = scheduler.advance_to_next_work()
            # Broadcast work_tile_info to smem...

    Usage (Executor warps - via MoESchedExtension):
~~~~

**EN**: Defines `MoEStaticPersistentTileScheduler`, a reusable Python class that packages configuration and behavior for this example. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `MoEStaticPersistentTileScheduler`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 265-269 / 第 265-269 行

~~~~python
        # Read work_tile_info from smem...
        real_a, desc_a = ext.get_gmem_tensor("a", tma_tensor_a, offs, work_tile_info)
        real_b, desc_b = ext.get_gmem_tensor("b", tma_tensor_b, offs, work_tile_info)
        real_c, desc_c = ext.get_gmem_tensor("c", tma_tensor_c, offs, work_tile_info)
    """
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 271-294 / 第 271-294 行

~~~~python
    def __init__(
        self,
        # Params (contains scenario, expert_cnt, intermediate, hidden, tile/cluster shapes)
        params: MoEStaticSchedulerParams,
        # Runtime tensor for scheduling
        offs: cute.Tensor,             # (experts,) cumsum of token counts
        # Scheduling state
        num_persistent_clusters: Int32,
        current_work_linear_idx: Int32,
        cta_id_in_cluster: cute.Coord,
        # Expert tracking state (for O(1) advance within same expert)
        current_expert_idx: Int32,
        expert_tile_start: Int32,      # cumsum of tiles before current expert
        expert_tile_end: Int32,        # cumsum of tiles including current expert
    ):
        self.params = params
        self.offs = offs
        self.num_persistent_clusters = num_persistent_clusters
        self._current_work_linear_idx = current_work_linear_idx
        self.cta_id_in_cluster = cta_id_in_cluster
        # Expert tracking
        self.current_expert_idx = current_expert_idx
        self.expert_tile_start = expert_tile_start
        self.expert_tile_end = expert_tile_end
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 296-298 / 第 296-298 行

~~~~python
    # =========================================================================
    # Convenience accessors for params
    # =========================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 300-302 / 第 300-302 行

~~~~python
    @property
    def scenario(self) -> Literal["2Dx3D", "2Dx2D"]:
        return self.params.scenario
~~~~

**EN**: Defines `scenario`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `scenario`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 304-306 / 第 304-306 行

~~~~python
    @property
    def expert_cnt(self) -> Int32:
        return self.params.expert_cnt
~~~~

**EN**: Defines `expert_cnt`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `expert_cnt`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 308-310 / 第 308-310 行

~~~~python
    @property
    def intermediate(self) -> Int32:
        return self.params.intermediate
~~~~

**EN**: Defines `intermediate`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `intermediate`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 312-314 / 第 312-314 行

~~~~python
    @property
    def hidden(self) -> Int32:
        return self.params.hidden
~~~~

**EN**: Defines `hidden`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `hidden`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 316-318 / 第 316-318 行

~~~~python
    @property
    def cta_tile_shape_mnk(self) -> Tuple[int, int, int]:
        return self.params.cta_tile_shape_mnk
~~~~

**EN**: Defines `cta_tile_shape_mnk`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cta_tile_shape_mnk`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 320-322 / 第 320-322 行

~~~~python
    @property
    def cluster_shape_mn(self) -> Tuple[int, int]:
        return self.params.cluster_shape_mn
~~~~

**EN**: Defines `cluster_shape_mn`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cluster_shape_mn`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 324-326 / 第 324-326 行

~~~~python
    @property
    def cluster_tile_m(self) -> int:
        return self.params.cluster_tile_m
~~~~

**EN**: Defines `cluster_tile_m`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cluster_tile_m`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 328-330 / 第 328-330 行

~~~~python
    @property
    def cluster_tile_n(self) -> int:
        return self.params.cluster_tile_n
~~~~

**EN**: Defines `cluster_tile_n`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cluster_tile_n`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 332-334 / 第 332-334 行

~~~~python
    @property
    def cta_tile_k(self) -> int:
        return self.params.cta_tile_k
~~~~

**EN**: Defines `cta_tile_k`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `cta_tile_k`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 336-338 / 第 336-338 行

~~~~python
    # =========================================================================
    # MLIR value serialization (for SSA value passing in device code)
    # =========================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 340-354 / 第 340-354 行

~~~~python
    def __extract_mlir_values__(self) -> List[ir.Value]:
        values = []
        # Params (only runtime values are extracted)
        values.extend(extract_mlir_values(self.params))
        # Runtime tensor for scheduling
        values.extend(extract_mlir_values(self.offs))
        # Scheduling state
        values.extend(extract_mlir_values(self.num_persistent_clusters))
        values.extend(extract_mlir_values(self._current_work_linear_idx))
        values.extend(extract_mlir_values(self.cta_id_in_cluster))
        # Expert tracking state
        values.extend(extract_mlir_values(self.current_expert_idx))
        values.extend(extract_mlir_values(self.expert_tile_start))
        values.extend(extract_mlir_values(self.expert_tile_end))
        return values
~~~~

**EN**: Defines `__extract_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__extract_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 356-359 / 第 356-359 行

~~~~python
    def __new_from_mlir_values__(
        self, values: List[ir.Value]
    ) -> "MoEStaticPersistentTileScheduler":
        idx = 0
~~~~

**EN**: Defines `__new_from_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `__new_from_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 361-361 / 第 361-361 行

~~~~python
        # Params (3 values: expert_cnt, intermediate, hidden)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 362-363 / 第 362-363 行

~~~~python
        new_params = new_from_mlir_values(self.params, values[idx:idx + 3])
        idx += 3
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 365-365 / 第 365-365 行

~~~~python
        # Runtime tensor for scheduling (variable size)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 366-368 / 第 366-368 行

~~~~python
        offs_len = len(extract_mlir_values(self.offs))
        new_offs = new_from_mlir_values(self.offs, values[idx:idx + offs_len])
        idx += offs_len
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 370-370 / 第 370-370 行

~~~~python
        # Scheduling state
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 371-378 / 第 371-378 行

~~~~python
        new_num_persistent_clusters = new_from_mlir_values(
            self.num_persistent_clusters, [values[idx]]
        )
        idx += 1
        new_current_work_linear_idx = new_from_mlir_values(
            self._current_work_linear_idx, [values[idx]]
        )
        idx += 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 380-380 / 第 380-380 行

~~~~python
        # cta_id_in_cluster (3 values for Coord)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 381-384 / 第 381-384 行

~~~~python
        new_cta_id_in_cluster = new_from_mlir_values(
            self.cta_id_in_cluster, values[idx:idx + 3]
        )
        idx += 3
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 386-386 / 第 386-386 行

~~~~python
        # Expert tracking state
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 387-398 / 第 387-398 行

~~~~python
        new_current_expert_idx = new_from_mlir_values(
            self.current_expert_idx, [values[idx]]
        )
        idx += 1
        new_expert_tile_start = new_from_mlir_values(
            self.expert_tile_start, [values[idx]]
        )
        idx += 1
        new_expert_tile_end = new_from_mlir_values(
            self.expert_tile_end, [values[idx]]
        )
        idx += 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 400-409 / 第 400-409 行

~~~~python
        return MoEStaticPersistentTileScheduler(
            params=new_params,
            offs=new_offs,
            num_persistent_clusters=new_num_persistent_clusters,
            current_work_linear_idx=new_current_work_linear_idx,
            cta_id_in_cluster=new_cta_id_in_cluster,
            current_expert_idx=new_current_expert_idx,
            expert_tile_start=new_expert_tile_start,
            expert_tile_end=new_expert_tile_end,
        )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 411-413 / 第 411-413 行

~~~~python
    # =========================================================================
    # Factory method
    # =========================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 415-436 / 第 415-436 行

~~~~python
    @staticmethod
    @dsl_user_op
    def create(
        params: MoEStaticSchedulerParams,
        offs: cute.Tensor,
        block_idx: Tuple[Integer, Integer, Integer],
        grid_dim: Tuple[Integer, Integer, Integer],
        *,
        loc=None,
        ip=None,
    ) -> "MoEStaticPersistentTileScheduler":
        """
        Create a MoE persistent tile scheduler.
        
        :param params: Scheduler parameters (from host)
        :param offs: Cumsum tensor of token counts per expert, shape (experts,)
        :param block_idx: CUDA block index
        :param grid_dim: CUDA grid dimensions
        """
        num_persistent_clusters = cute.size(grid_dim, loc=loc, ip=ip) // cute.size(
            params.cluster_shape_mn, loc=loc, ip=ip
        )
~~~~

**EN**: Defines `create` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `create` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 438-439 / 第 438-439 行

~~~~python
        bidx, bidy, bidz = block_idx
        current_work_linear_idx = Int32(bidz)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 441-445 / 第 441-445 行

~~~~python
        cta_id_in_cluster = (
            Int32(bidx % params.cluster_shape_mn[0]),
            Int32(bidy % params.cluster_shape_mn[1]),
            Int32(0),
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 447-448 / 第 447-448 行

~~~~python
        # Initialize expert tracking to "before expert 0"
        # The first call to _get_work_tile_for_linear_idx will advance to the correct expert
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 449-451 / 第 449-451 行

~~~~python
        current_expert_idx = Int32(0)
        expert_tile_start = Int32(0)
        expert_tile_end = Int32(0)  # Will be computed on first access
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 453-462 / 第 453-462 行

~~~~python
        return MoEStaticPersistentTileScheduler(
            params=params,
            offs=offs,
            num_persistent_clusters=num_persistent_clusters,
            current_work_linear_idx=current_work_linear_idx,
            cta_id_in_cluster=cta_id_in_cluster,
            current_expert_idx=current_expert_idx,
            expert_tile_start=expert_tile_start,
            expert_tile_end=expert_tile_end,
        )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 464-466 / 第 464-466 行

~~~~python
    # =========================================================================
    # Tile iteration methods
    # =========================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 468-474 / 第 468-474 行

~~~~python
    @dsl_user_op
    @cute.jit
    def initial_work_tile_info(self, *, loc=None, ip=None) -> MoEWorkTileInfo:
        """Get the initial work tile info."""
        return self._get_work_tile_for_linear_idx(
            self._current_work_linear_idx, loc=loc, ip=ip
        )
~~~~

**EN**: Marks `initial_work_tile_info` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `initial_work_tile_info` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 476-483 / 第 476-483 行

~~~~python
    @dsl_user_op
    @cute.jit
    def advance_to_next_work(self, *, loc=None, ip=None) -> MoEWorkTileInfo:
        """Advance to the next work tile and return its info."""
        self._current_work_linear_idx += self.num_persistent_clusters
        return self._get_work_tile_for_linear_idx(
            self._current_work_linear_idx, loc=loc, ip=ip
        )
~~~~

**EN**: Marks `advance_to_next_work` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `advance_to_next_work` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 485-503 / 第 485-503 行

~~~~python
    @dsl_user_op
    @cute.jit
    def _get_work_tile_for_linear_idx(
        self, 
        cluster_linear_idx: Int32, 
        *, 
        loc=None, 
        ip=None
    ) -> MoEWorkTileInfo:
        """
        Convert a linear cluster index to MoEWorkTileInfo.
        
        Uses cached expert tracking state for O(1) fast path when staying
        within the same expert. Advances expert state when needed.
        
        Returns an invalid tile (expert_idx = -1) if cluster_linear_idx is out of range.
        """
        # Ensure expert tracking is initialized and up-to-date
        self._advance_expert_to_contain(cluster_linear_idx, loc=loc, ip=ip)
~~~~

**EN**: Marks `_get_work_tile_for_linear_idx` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_get_work_tile_for_linear_idx` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 505-505 / 第 505-505 行

~~~~python
        # Check if valid (still within expert range after advancing)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 506-506 / 第 506-506 行

~~~~python
        is_valid = self.current_expert_idx < self.expert_cnt
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 508-513 / 第 508-513 行

~~~~python
        work_tile_info = MoEWorkTileInfo(
            expert_idx=Int32(-1),
            tile_m_idx=Int32(0),
            tile_n_idx=Int32(0),
            k_tile_cnt=Int32(0),
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 515-520 / 第 515-520 行

~~~~python
        if is_valid:
            # Compute local cluster tile indices within current expert
            local_idx = cluster_linear_idx - self.expert_tile_start
            cluster_tile_m_idx, cluster_tile_n_idx = self._decompose_local_idx(
                local_idx, self.current_expert_idx, loc=loc, ip=ip
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 522-523 / 第 522-523 行

~~~~python
            # Convert cluster tile indices to CTA tile indices
            # cta_tile_idx = cluster_tile_idx * cluster_shape + cta_id_in_cluster
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 524-533 / 第 524-533 行

~~~~python
            cta_tile_m_idx = (
                cluster_tile_m_idx * self.cluster_shape_mn[0]
                + self.cta_id_in_cluster[0]  # type: ignore[index]
            )
            cta_tile_n_idx = (
                cluster_tile_n_idx * self.cluster_shape_mn[1]
                + self.cta_id_in_cluster[1]  # type: ignore[index]
            )
            # Compute k_tile_cnt
            k_tile_cnt = self._compute_k_tile_cnt(self.current_expert_idx, loc=loc, ip=ip)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 535-541 / 第 535-541 行

~~~~python
            work_tile_info = MoEWorkTileInfo(
                expert_idx=self.current_expert_idx,
                tile_m_idx=cta_tile_m_idx,
                tile_n_idx=cta_tile_n_idx,
                k_tile_cnt=k_tile_cnt,
            )
        return work_tile_info
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 543-561 / 第 543-561 行

~~~~python
    @dsl_user_op
    @cute.jit
    def _advance_expert_to_contain(
        self,
        cluster_linear_idx: Int32,
        *,
        loc=None,
        ip=None,
    ) -> None:
        """
        Advance expert tracking state until current expert contains cluster_linear_idx,
        or we run out of experts.
        
        Fast path: If already in correct expert, no work needed.
        """
        # Initialize expert_tile_end if this is the first call (expert_tile_end == 0)
        if self.expert_tile_end == Int32(0):
            tiles_for_expert_0 = self._compute_tiles_for_expert(Int32(0), loc=loc, ip=ip)
            self.expert_tile_end = tiles_for_expert_0
~~~~

**EN**: Marks `_advance_expert_to_contain` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_advance_expert_to_contain` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 563-563 / 第 563-563 行

~~~~python
        # Advance until cluster_linear_idx < expert_tile_end or no more experts
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 564-566 / 第 564-566 行

~~~~python
        while cluster_linear_idx >= self.expert_tile_end and self.current_expert_idx < self.expert_cnt:
            self.current_expert_idx = self.current_expert_idx + 1
            self.expert_tile_start = self.expert_tile_end
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 568-572 / 第 568-572 行

~~~~python
            if self.current_expert_idx < self.expert_cnt:
                tiles_for_expert = self._compute_tiles_for_expert(
                    self.current_expert_idx, loc=loc, ip=ip
                )
                self.expert_tile_end = self.expert_tile_end + tiles_for_expert
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 574-600 / 第 574-600 行

~~~~python
    @dsl_user_op
    @cute.jit
    def _compute_tiles_for_expert(
        self,
        expert_idx: Int32,
        *,
        loc=None,
        ip=None,
    ) -> Int32:
        """Compute total cluster tiles for a given expert."""
        if const_expr(self.scenario == "2Dx2D"):
            # Fixed M=hidden, N=intermediate
            cluster_tile_m_cnt = (self.hidden + self.cluster_tile_m - 1) // self.cluster_tile_m
            cluster_tile_n_cnt = (self.intermediate + self.cluster_tile_n - 1) // self.cluster_tile_n
            return cluster_tile_m_cnt * cluster_tile_n_cnt
        else:  # 2Dx3D
            # Variable M (tokens), fixed N
            tokens_i = self.offs[expert_idx]
            if expert_idx > 0:
                tokens_i = tokens_i - self.offs[expert_idx - 1]  # type: ignore[operator]
            cluster_tile_m_cnt = (
                tokens_i + self.cluster_tile_m - 1  # type: ignore[operator]
            ) // self.cluster_tile_m
            cluster_tile_n_cnt = (
                self.intermediate + self.cluster_tile_n - 1
            ) // self.cluster_tile_n
            return cluster_tile_m_cnt * cluster_tile_n_cnt
~~~~

**EN**: Marks `_compute_tiles_for_expert` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `_compute_tiles_for_expert` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 602-627 / 第 602-627 行

~~~~python
    @dsl_user_op
    @cute.jit
    def _decompose_local_idx(
        self,
        local_idx: Int32,
        expert_idx: Int32,
        *,
        loc=None,
        ip=None,
    ) -> Tuple[Int32, Int32]:
        """
        Decompose local cluster tile index within expert to (cluster_tile_m_idx, cluster_tile_n_idx).
        
        Uses "short side first" strategy: the shorter dimension changes faster.
        This maximizes overlap between adjacent clusters for better L2 cache utilization.
        
        For example, if m_cnt=2, n_cnt=8:
        - N is longer, so M changes faster: local_idx = n_idx * m_cnt + m_idx
        - Linearization order: (0,0), (1,0), (0,1), (1,1), (0,2), (1,2), ...
        """
        # Get tile counts for M and N
        cluster_tile_m_cnt, cluster_tile_n_cnt = self._get_cluster_tile_counts(
            expert_idx, loc=loc, ip=ip
        )
        cluster_tile_m_idx = -1
        cluster_tile_n_idx = -1
~~~~

**EN**: Marks `_decompose_local_idx` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_decompose_local_idx` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 629-633 / 第 629-633 行

~~~~python
        # Short side first: shorter dimension changes faster
        # If m_cnt <= n_cnt: m is shorter, m changes faster
        #   local_idx = n_idx * m_cnt + m_idx
        # If n_cnt < m_cnt: n is shorter, n changes faster
        #   local_idx = m_idx * n_cnt + n_idx
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 634-641 / 第 634-641 行

~~~~python
        if cluster_tile_m_cnt <= cluster_tile_n_cnt:
            # M is shorter or equal, M changes faster
            cluster_tile_m_idx = local_idx % cluster_tile_m_cnt
            cluster_tile_n_idx = local_idx // cluster_tile_m_cnt
        else:
            # N is shorter, N changes faster
            cluster_tile_n_idx = local_idx % cluster_tile_n_cnt
            cluster_tile_m_idx = local_idx // cluster_tile_n_cnt
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 643-643 / 第 643-643 行

~~~~python
        return (cluster_tile_m_idx, cluster_tile_n_idx)
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 645-670 / 第 645-670 行

~~~~python
    @dsl_user_op
    @cute.jit
    def _get_cluster_tile_counts(
        self,
        expert_idx: Int32,
        *,
        loc=None,
        ip=None,
    ) -> Tuple[Int32, Int32]:
        """Get (cluster_tile_m_cnt, cluster_tile_n_cnt) for a given expert."""
        if const_expr(self.scenario == "2Dx2D"):
            # Fixed M=hidden, N=intermediate
            cluster_tile_m_cnt = (self.hidden + self.cluster_tile_m - 1) // self.cluster_tile_m
            cluster_tile_n_cnt = (self.intermediate + self.cluster_tile_n - 1) // self.cluster_tile_n
        else:  # 2Dx3D
            # Variable M (tokens), fixed N
            tokens_i = self.offs[expert_idx]
            if expert_idx > 0:
                tokens_i = tokens_i - self.offs[expert_idx - 1]  # type: ignore[operator]
            cluster_tile_m_cnt = (
                tokens_i + self.cluster_tile_m - 1  # type: ignore[operator]
            ) // self.cluster_tile_m
            cluster_tile_n_cnt = (
                self.intermediate + self.cluster_tile_n - 1
            ) // self.cluster_tile_n
        return (cluster_tile_m_cnt, cluster_tile_n_cnt)
~~~~

**EN**: Marks `_get_cluster_tile_counts` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `_get_cluster_tile_counts` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 672-695 / 第 672-695 行

~~~~python
    @dsl_user_op
    @cute.jit
    def _compute_k_tile_cnt(
        self,
        expert_idx: Int32,
        *,
        loc=None,
        ip=None,
    ) -> Int32:
        """
        Compute the number of K tiles for this expert.
        
        2Dx3D: K = hidden (fixed) -> k_tile_cnt = ceil(hidden / cta_tile_k)
        2Dx2D: K = tokens_i (variable) -> k_tile_cnt = ceil(tokens_i / cta_tile_k)
        """
        if const_expr(self.scenario == "2Dx3D"):
            # K is hidden (fixed)
            return (self.hidden + self.cta_tile_k - 1) // self.cta_tile_k
        else:  # 2Dx2D
            # K is tokens_i (variable per expert)
            tokens_i = self.offs[expert_idx]
            if expert_idx > cutlass.Int32(0):
                tokens_i = tokens_i - self.offs[expert_idx - 1]  # type: ignore[operator]
            return (tokens_i + self.cta_tile_k - 1) // self.cta_tile_k  # type: ignore[return-value, operator]
~~~~

**EN**: Marks `_compute_k_tile_cnt` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `_compute_k_tile_cnt` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Framework interoperability and export / 框架互操作与导出
- Mixture-of-experts scheduling / Mixture-of-Experts 调度

## Dependencies / 依赖项

- `typing.List` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Literal` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cutlass_dsl.Boolean` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.Int32` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.Integer` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.extract_mlir_values` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.new_from_mlir_values` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.const_expr` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.dsl_user_op` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass._mlir.ir` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
