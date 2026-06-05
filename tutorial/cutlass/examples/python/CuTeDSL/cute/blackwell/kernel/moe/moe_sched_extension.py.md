# moe_sched_extension.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/moe/moe_sched_extension.py`  
**Purpose / 用途**: MoE Scheduler Extension. / 这是一个使用 CuTeDSL 实现 moe sched extension 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-63 / 第 29-63 行

~~~~python
"""
MoE Scheduler Extension.

Bridges the MoE tile scheduler (MoEStaticPersistentTileScheduler) with tensor-level
domain conversion and TMA descriptor selection. This is the "glue" layer between:

- Scheduler: produces MoEWorkTileInfo (expert_idx, tile_m, tile_n, k_tile_cnt)
- OnlineTensormapDescCreator: builds/retrieves TMA descriptors from workspace
- Kernel: orchestrates everything

Different kernel types (grouped_mm, scaled_grouped_mm, etc.) provide their own
MoESchedExtension subclass with kernel-specific domain conversion logic.

Key design principles:
- Unified interface: get_gmem_tensor() for all tensor types
- Free implementation: no role-based templates, each subclass writes its own logic
- Composable utilities: compute_expert_token_range, rewrite_tensor_shape, etc.
  are available as tools but not mandatory

Architecture:

    Scheduler ──(produces)──> MoEWorkTileInfo
                                    │
                           expert_idx, tile_m, tile_n, k_cnt
                                    │
                                    v
    Extension ──(uses)──> OnlineTensormapDescCreator
        │                         │
        │  get_gmem_tensor()      │  get_desc_ptr()
        │  prefetch_for_expert()  │  construct_and_write()
        │                         │
        └── internal calls ───────┘

    Kernel (caller): the only place that knows all three exist
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 65-76 / 第 65-76 行

~~~~python
from abc import ABC, abstractmethod
from typing import Literal, Tuple, Union

import cutlass
import cutlass.cute as cute
from cutlass.cute.typing import Pointer
from cutlass.cutlass_dsl import Int32

from dataclasses import dataclass

from cutlass.utils.blockscaled_layout import tile_atom_to_shape_SF
from blackwell.kernel.moe.moe_utils import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 77-83 / 第 77-83 行

~~~~python
    OnlineTensormapDescCreator,
    tensormap_ptr_for_copy,
    compute_expert_token_range,
    rewrite_tensor_shape,
    prefetch_tma_descriptor,
)
from blackwell.kernel.moe.moe_persistent_scheduler import MoEWorkTileInfo
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 86-111 / 第 86-111 行

~~~~python
@dataclass(frozen=True)
class MoESchedExtension(ABC):
    """
    Abstract base class for MoE scheduler extensions.

    Bridges MoEWorkTileInfo with tensor-level domain conversion and TMA
    descriptor selection. Each kernel type (grouped_mm, scaled_grouped_mm, etc.)
    provides its own subclass with kernel-specific logic.

    The extension:
    - Holds a reference to an OnlineTensormapDescCreator for expert-wise desc retrieval
    - Implements get_gmem_tensor() to convert MoE-view tensors to per-expert tensors
    - Implements prefetch_for_expert() to prefetch expert-wise TMA descriptors

    Subclasses are free to add any additional attributes in __init__ (scenario,
    codegen configs, etc.) and implement get_gmem_tensor with arbitrary logic
    per tensor_name. No role-based templates or rigid patterns are imposed.

    Usage in kernel (caller):
        ext = ConcreteSchedExtension(tensormap_ctor, scenario=...)

        while work_tile_info.is_valid_tile:
            real_a, desc_a = ext.get_gmem_tensor("a", tma_tensor_a, offs, work_tile_info)
            real_b, desc_b = ext.get_gmem_tensor("b", tma_tensor_b, offs, work_tile_info)
            # Use real_a, desc_a in cute.copy ...
    """
~~~~

**EN**: Defines `MoESchedExtension`, a reusable Python class that packages configuration and behavior for this example. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `MoESchedExtension`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 113-115 / 第 113-115 行

~~~~python
    def __init__(self, tensormap_ctor: OnlineTensormapDescCreator):
        super().__init__()
        self.tensormap_ctor = tensormap_ctor
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。

### Lines 117-144 / 第 117-144 行

~~~~python
    @abstractmethod
    def get_gmem_tensor(
        self,
        tensor_name: str,
        gmem_tensor_in_moe_view: cute.Tensor,
        offs: Union[cute.Tensor, Tuple[cute.Tensor, cute.Tensor]],
        work_tile_info: MoEWorkTileInfo,
    ) -> Tuple[cute.Tensor, "Pointer | None"]:
        """
        Convert an MoE-view tensor to the real per-expert tensor for the
        current work tile, and return the appropriate TMA descriptor pointer.

        The MoE-view tensor uses "fake" GEMM domain dimensions that span all
        experts (e.g., fake_m = tokens_sum). This method slices/offsets it
        to the current expert's actual region.

        :param tensor_name: Identifies which tensor (e.g., "a", "b", "c", "sfa")
        :param gmem_tensor_in_moe_view: Tensor in fake GEMM MNKL domain
        :param offs: Either a single cumsum tensor (experts,), or a tuple of
                     (offs_token, offs_padded) where offs_padded provides
                     padded offsets for scale-factor domain conversion.
        :param work_tile_info: Current work tile from the scheduler
        :return: (real_tensor, tma_desc_ptr_or_none)
                 - real_tensor: domain-offset and shape-rewritten tensor for this expert
                 - tma_desc_ptr: expert-wise desc ptr (already converted for cute.copy),
                   or None if the caller should use the global TMA descriptor
        """
        ...
~~~~

**EN**: Defines `get_gmem_tensor`, grouping related logic behind a named Python callable so the example stays modular and reusable. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `get_gmem_tensor`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 146-156 / 第 146-156 行

~~~~python
    @abstractmethod
    def prefetch_for_expert(self, expert_idx: Int32) -> None:
        """
        Prefetch expert-wise TMA descriptors for the given expert.

        Called when the scheduler advances to a new expert, allowing the TMA
        descriptor cache to be warmed up before the descriptors are needed.

        :param expert_idx: Index of the expert whose descriptors to prefetch
        """
        ...
~~~~

**EN**: Defines `prefetch_for_expert`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `prefetch_for_expert`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 159-161 / 第 159-161 行

~~~~python
# =============================================================================
# Grouped MM Extension
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 164-179 / 第 164-179 行

~~~~python
class GroupedMmSchedExtension(MoESchedExtension):
    """
    MoE scheduler extension for grouped_mm: handles tensors a, b, c.

    Domain conversion logic per scenario:

    2Dx3D:
        A: (fake_m, k, 1)  -> offset fake_m by token_offset, global desc
        B: (n, k, fake_l)  -> offset fake_l by expert_idx, global desc
        C: (fake_m, n, 1)  -> rewrite shape only, expert-wise desc

    2Dx2D:
        A: (m, fake_k, 1)  -> rewrite shape only, expert-wise desc
        B: (n, fake_k, 1)  -> rewrite shape only, expert-wise desc
        C: (m, n, fake_l)  -> offset fake_l by expert_idx, global desc
    """
~~~~

**EN**: Defines `GroupedMmSchedExtension`, a reusable Python class that packages configuration and behavior for this example. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `GroupedMmSchedExtension`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 181-187 / 第 181-187 行

~~~~python
    def __init__(
        self,
        scenario: Literal["2Dx3D", "2Dx2D"],
        tensormap_ctor: OnlineTensormapDescCreator,
    ):
        super().__init__(tensormap_ctor)
        self.scenario = scenario
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 189-198 / 第 189-198 行

~~~~python
    @cute.jit
    def get_gmem_tensor(
        self,
        tensor_name: str,
        gmem_tensor_in_moe_view: cute.Tensor,
        offs: cute.Tensor,
        work_tile_info: MoEWorkTileInfo,
    ):
        expert_idx = work_tile_info.expert_idx
        token_offset, tokens_i = compute_expert_token_range(offs, expert_idx)
~~~~

**EN**: Marks `get_gmem_tensor` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `get_gmem_tensor` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 200-201 / 第 200-201 行

~~~~python
        shape = gmem_tensor_in_moe_view.shape
        c1 = cutlass.Int32(1)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 203-223 / 第 203-223 行

~~~~python
        if cutlass.const_expr(self.scenario == "2Dx3D"):
            if cutlass.const_expr(tensor_name == "a"):
                # A: (fake_m, k, 1) -> offset fake_m, global desc
                real = cute.domain_offset((token_offset, 0, 0), gmem_tensor_in_moe_view)
                real = rewrite_tensor_shape(real, (tokens_i, shape[1], c1))  # type: ignore[index]
                return (real, None)
            elif cutlass.const_expr(tensor_name == "b"):
                # B: (n, k, fake_l) -> offset fake_l, global desc
                real = cute.domain_offset((0, 0, expert_idx), gmem_tensor_in_moe_view)
                real = rewrite_tensor_shape(real, (shape[0], shape[1], c1))  # type: ignore[index]
                return (real, None)
            elif cutlass.const_expr(tensor_name == "c"):
                # C: (fake_m, n, 1) -> expert-wise desc, no offset
                real = rewrite_tensor_shape(
                    gmem_tensor_in_moe_view,
                    (tokens_i, shape[1], c1),  # type: ignore[index]
                )
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("c", expert_idx)
                )
                return (real, desc)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 225-250 / 第 225-250 行

~~~~python
        elif cutlass.const_expr(self.scenario == "2Dx2D"):
            if cutlass.const_expr(tensor_name == "a"):
                # A: (m, fake_k, 1) -> expert-wise desc, no offset
                real = rewrite_tensor_shape(
                    gmem_tensor_in_moe_view,
                    (shape[0], tokens_i, c1),  # type: ignore[index]
                )
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("a", expert_idx)
                )
                return (real, desc)
            elif cutlass.const_expr(tensor_name == "b"):
                # B: (n, fake_k, 1) -> expert-wise desc, no offset
                real = rewrite_tensor_shape(
                    gmem_tensor_in_moe_view,
                    (shape[0], tokens_i, c1),  # type: ignore[index]
                )
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("b", expert_idx)
                )
                return (real, desc)
            elif cutlass.const_expr(tensor_name == "c"):
                # C: (m, n, fake_l) -> offset fake_l, global desc
                real = cute.domain_offset((0, 0, expert_idx), gmem_tensor_in_moe_view)
                real = rewrite_tensor_shape(real, (shape[0], shape[1], c1))  # type: ignore[index]
                return (real, None)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 252-252 / 第 252-252 行

~~~~python
        raise ValueError("Invalid scenario or GEMM tensor name.")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 254-262 / 第 254-262 行

~~~~python
    @cute.jit
    def prefetch_for_expert(self, expert_idx: Int32) -> None:
        if cutlass.const_expr(self.scenario == "2Dx3D"):
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("c", expert_idx))
        elif cutlass.const_expr(self.scenario == "2Dx2D"):
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("a", expert_idx))
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("b", expert_idx))
        else:
            raise ValueError("Invalid scenario.")
~~~~

**EN**: Marks `prefetch_for_expert` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 将 `prefetch_for_expert` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 266-268 / 第 266-268 行

~~~~python
# =============================================================================
# Scaled Grouped MM Extension (block-scaled MoE)
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 271-298 / 第 271-298 行

~~~~python
class ScaledGroupedMmSchedExtension(MoESchedExtension):
    """
    MoE scheduler extension for scaled_grouped_mm: handles a, b, c, sfa, sfb.

    Extends GroupedMmSchedExtension with scale-factor tensor support.
    SFA/SFB are passed as flat GEMM-domain tensors and atom-tiled per expert
    via tile_atom_to_shape_SF.

    The offs parameter is always a tuple (offs_token, offs_padded):
    - offs_token: cumsum offsets in data (activation) domain
    - offs_padded: cumsum offsets in scale-factor domain (padded to atom granularity)

    sf_vec_size is obtained from self.tensormap_ctor.sf_vec_size.

    Domain conversion logic per scenario:

    2Dx3D:
        A:   (fake_m, k, 1)               -> offset fake_m by token_offset, global desc
        B:   (n, k, fake_l)               -> offset fake_l by expert_idx, global desc
        C:   (fake_m, n, 1)               -> rewrite shape, expert-wise desc
        SFA: (fake_m_pad, k_pad, 1)       -> offset fake_m_pad by padded_offset,
                                              atom-tile, global desc
        SFB: (n_pad, k_pad, fake_l)       -> offset fake_l by expert_idx,
                                              atom-tile, global desc

    2Dx2D:
        A:   (m, fake_k, 1)               -> rewrite shape, expert-wise desc
        B:   (n, fake_k, 1)               -> rewrite shape, expert-wise desc
~~~~

**EN**: Defines `ScaledGroupedMmSchedExtension`, a reusable Python class that packages configuration and behavior for this example. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `ScaledGroupedMmSchedExtension`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 299-304 / 第 299-304 行

~~~~python
        C:   (m, n, fake_l)               -> offset fake_l by expert_idx, global desc
        SFA: (m_pad, fake_k_pad, 1)       -> offset fake_k_pad by padded_offset,
                                              atom-tile, expert-wise desc
        SFB: (n_pad, fake_k_pad, 1)       -> offset fake_k_pad by padded_offset,
                                              atom-tile, expert-wise desc
    """
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 306-312 / 第 306-312 行

~~~~python
    def __init__(
        self,
        scenario: Literal["2Dx3D", "2Dx2D"],
        tensormap_ctor: OnlineTensormapDescCreator,
    ):
        super().__init__(tensormap_ctor)
        self.scenario = scenario
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 314-323 / 第 314-323 行

~~~~python
    @cute.jit
    def get_gmem_tensor(
        self,
        tensor_name: str,
        gmem_tensor_in_moe_view: cute.Tensor,
        offs: Tuple[cute.Tensor, cute.Tensor],
        work_tile_info: MoEWorkTileInfo,
    ):
        # Unpack the offs tuple
        offs_token, offs_padded = offs
~~~~

**EN**: Marks `get_gmem_tensor` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `get_gmem_tensor` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 325-329 / 第 325-329 行

~~~~python
        expert_idx = work_tile_info.expert_idx
        token_offset, tokens_i = compute_expert_token_range(offs_token, expert_idx)
        padded_offset, padded_size_i = compute_expert_token_range(
            offs_padded, expert_idx
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 331-334 / 第 331-334 行

~~~~python
        shape = gmem_tensor_in_moe_view.shape
        stride = gmem_tensor_in_moe_view.stride
        c1 = cutlass.Int32(1)
        sf_vec_size = self.tensormap_ctor.sf_vec_size
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 336-341 / 第 336-341 行

~~~~python
        if cutlass.const_expr(self.scenario == "2Dx3D"):
            if cutlass.const_expr(tensor_name == "a"):
                # A: (fake_m, k, 1) -> offset fake_m, global desc
                real = cute.domain_offset((token_offset, 0, 0), gmem_tensor_in_moe_view)
                real = rewrite_tensor_shape(real, (tokens_i, shape[1], c1))  # type: ignore[index]
                return (real, None)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 343-347 / 第 343-347 行

~~~~python
            elif cutlass.const_expr(tensor_name == "b"):
                # B: (n, k, fake_l) -> offset fake_l, global desc
                real = cute.domain_offset((0, 0, expert_idx), gmem_tensor_in_moe_view)
                real = rewrite_tensor_shape(real, (shape[0], shape[1], c1))  # type: ignore[index]
                return (real, None)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 349-358 / 第 349-358 行

~~~~python
            elif cutlass.const_expr(tensor_name == "c"):
                # C: (fake_m, n, 1) -> expert-wise desc
                real = rewrite_tensor_shape(
                    gmem_tensor_in_moe_view,
                    (tokens_i, shape[1], c1),  # type: ignore[index]
                )
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("c", expert_idx)
                )
                return (real, desc)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 360-370 / 第 360-370 行

~~~~python
            elif cutlass.const_expr(tensor_name == "sfa"):
                # SFA: (fake_m_pad, k_pad, 1) -> offset fake_m_pad, atom-tile, global desc
                real = cute.domain_offset(
                    (padded_offset, 0, 0), gmem_tensor_in_moe_view
                )
                per_expert_shape = (padded_size_i, shape[1], c1)  # type: ignore[index]
                sf_layout = tile_atom_to_shape_SF(per_expert_shape, sf_vec_size)
                real = cute.make_tensor(
                    real.iterator, cute.make_layout(sf_layout.shape, stride=stride)
                )
                return (real, None)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 372-380 / 第 372-380 行

~~~~python
            elif cutlass.const_expr(tensor_name == "sfb"):
                # SFB: (n_pad, k_pad, fake_l) -> offset fake_l, atom-tile, global desc
                real = cute.domain_offset((0, 0, expert_idx), gmem_tensor_in_moe_view)
                per_expert_shape = (shape[0], shape[1], c1)  # type: ignore[index]
                sf_layout = tile_atom_to_shape_SF(per_expert_shape, sf_vec_size)
                real = cute.make_tensor(
                    real.iterator, cute.make_layout(sf_layout.shape, stride=stride)
                )
                return (real, None)
~~~~

**EN**: Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 382-392 / 第 382-392 行

~~~~python
        elif cutlass.const_expr(self.scenario == "2Dx2D"):
            if cutlass.const_expr(tensor_name == "a"):
                # A: (m, fake_k, 1) -> expert-wise desc
                real = rewrite_tensor_shape(
                    gmem_tensor_in_moe_view,
                    (shape[0], tokens_i, c1),  # type: ignore[index]
                )
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("a", expert_idx)
                )
                return (real, desc)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 394-403 / 第 394-403 行

~~~~python
            elif cutlass.const_expr(tensor_name == "b"):
                # B: (n, fake_k, 1) -> expert-wise desc
                real = rewrite_tensor_shape(
                    gmem_tensor_in_moe_view,
                    (shape[0], tokens_i, c1),  # type: ignore[index]
                )
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("b", expert_idx)
                )
                return (real, desc)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 405-409 / 第 405-409 行

~~~~python
            elif cutlass.const_expr(tensor_name == "c"):
                # C: (m, n, fake_l) -> offset fake_l, global desc
                real = cute.domain_offset((0, 0, expert_idx), gmem_tensor_in_moe_view)
                real = rewrite_tensor_shape(real, (shape[0], shape[1], c1))  # type: ignore[index]
                return (real, None)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 411-419 / 第 411-419 行

~~~~python
            elif cutlass.const_expr(tensor_name == "sfa"):
                # SFA: (m_pad, fake_k_pad, 1) -> offset fake_k_pad, atom-tile, expert-wise desc
                per_expert_shape = (shape[0], padded_size_i, c1)  # type: ignore[index]
                sf_layout = tile_atom_to_shape_SF(per_expert_shape, sf_vec_size)
                real = rewrite_tensor_shape(gmem_tensor_in_moe_view, sf_layout.shape)
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("sfa", expert_idx)
                )
                return (real, desc)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 421-429 / 第 421-429 行

~~~~python
            elif cutlass.const_expr(tensor_name == "sfb"):
                # SFB: (n_pad, fake_k_pad, 1) -> offset fake_k_pad, atom-tile, expert-wise desc
                per_expert_shape = (shape[0], padded_size_i, c1)  # type: ignore[index]
                sf_layout = tile_atom_to_shape_SF(per_expert_shape, sf_vec_size)
                real = rewrite_tensor_shape(gmem_tensor_in_moe_view, sf_layout.shape)
                desc = tensormap_ptr_for_copy(
                    self.tensormap_ctor.get_desc_ptr("sfb", expert_idx)
                )
                return (real, desc)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 431-431 / 第 431-431 行

~~~~python
        raise ValueError("Invalid scenario or tensor name.")
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 433-443 / 第 433-443 行

~~~~python
    @cute.jit
    def prefetch_for_expert(self, expert_idx: Int32) -> None:
        if cutlass.const_expr(self.scenario == "2Dx3D"):
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("c", expert_idx))
        elif cutlass.const_expr(self.scenario == "2Dx2D"):
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("a", expert_idx))
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("b", expert_idx))
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("sfa", expert_idx))
            prefetch_tma_descriptor(self.tensormap_ctor.get_desc_ptr("sfb", expert_idx))
        else:
            raise ValueError("Invalid scenario.")
~~~~

**EN**: Marks `prefetch_for_expert` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 将 `prefetch_for_expert` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `abc.ABC` — used by this example / 供该示例使用
- `abc.abstractmethod` — used by this example / 供该示例使用
- `typing.Literal` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Union` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.typing.Pointer` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cutlass_dsl.Int32` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `dataclasses.dataclass` — used by this example / 供该示例使用
- `cutlass.utils.blockscaled_layout.tile_atom_to_shape_SF` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `blackwell.kernel.moe.moe_utils.OnlineTensormapDescCreator` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_utils.tensormap_ptr_for_copy` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_utils.compute_expert_token_range` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_utils.rewrite_tensor_shape` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_utils.prefetch_tma_descriptor` — used by this example / 供该示例使用
- `blackwell.kernel.moe.moe_persistent_scheduler.MoEWorkTileInfo` — used by this example / 供该示例使用
