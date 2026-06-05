# moe_utils.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/moe/moe_utils.py`  
**Purpose / 用途**: Online TMA Descriptor Construction Utilities. / 这是一个使用 CuTeDSL 实现 moe utils 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-44 / 第 29-44 行

~~~~python
"""
Online TMA Descriptor Construction Utilities.

Provides utilities for dynamically creating TMA descriptors at kernel runtime
based on runtime-provided information (problem sizes, pointers, etc.).

Key components:
- OnlineTensormapDescCreator: Simplified ABC for TMA descriptor builders (2 abstract methods)
- TensormapWorkspace: Helper for linear workspace layout of TMA descriptors
- MoEGroupedGemmTensormapConstructor: TMA descriptor constructor for MoE Grouped GEMM
- GeneralGroupedGemmTensormapConstructor: TMA descriptor constructor for general Grouped GEMM
- Pointer utility functions (ptr_offset_bytes, gmem_ptr_to_generic, etc.)
- tensormap_ptr_for_copy: Convert raw desc ptr to cute.copy-compatible type
- compute_expert_token_range: Compute per-expert token offset and count from offs
- rewrite_tensor_shape: Debug-friendly tensor shape rewrite utility
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 46-60 / 第 46-60 行

~~~~python
from abc import ABC, abstractmethod
from typing import Optional, Literal, Tuple, Union

import cutlass
import cutlass.cute as cute
from cutlass.cute.typing import AddressSpace, Pointer
from cutlass.cute.nvgpu import cpasync
from cutlass.cutlass_dsl import dsl_user_op, Int32
from cutlass._mlir import ir
from cutlass._mlir.dialects import llvm
from cutlass._mlir.dialects import cute as _cute_ir
from cutlass._mlir.dialects import cute_nvgpu as _cute_nvgpu_ir
from dataclasses import dataclass

from cutlass.utils.blockscaled_layout import tile_atom_to_shape_SF
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 62-62 / 第 62-62 行

~~~~python
TensormapDescBytes = 128
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 64-66 / 第 64-66 行

~~~~python
# =============================================================================
# Pointer Utilities
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 69-88 / 第 69-88 行

~~~~python
@dsl_user_op
@cute.jit
def spin_wait(
    ptr: Pointer, condition, fail_sleep_cycles: int = 100, *, loc=None, ip=None
) -> None:
    """
    Generic spin-wait.
    Example usage:
        # Wait until counter >= total_blocks
        spin_wait(counter_ptr, lambda x: x >= total_blocks, fail_sleep_cycles=100)

        # Wait until flag == 1
        spin_wait(flag_ptr, lambda x: x == 1)
    """
    current = cute.arch.load(ptr, ptr.dtype, cop="cg", loc=loc, ip=ip)
    while not condition(current):
        # Load with L1 cache bypass (ld.global.cg)
        if cutlass.const_expr(fail_sleep_cycles > 0):
            cute.arch.nanosleep(sleep_time=fail_sleep_cycles, loc=loc, ip=ip)
        current = cute.arch.load(ptr, ptr.dtype, cop="cg", loc=loc, ip=ip)
~~~~

**EN**: Marks `spin_wait` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 将 `spin_wait` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 91-116 / 第 91-116 行

~~~~python
@dsl_user_op
def gmem_ptr_to_generic(
    gmem_ptr: Pointer,
    *,
    loc: Optional[ir.Location] = None,
    ip: Optional[ir.InsertionPoint] = None,
) -> Pointer:
    if gmem_ptr.memspace != AddressSpace.gmem:
        raise ValueError(
            f"gmem_ptr_to_generic requires pointer in gmem address space, "
            f"got {gmem_ptr.memspace}"
        )
    # Get LLVM pointer and cast to generic address space
    llvm_ptr = gmem_ptr.to_llvm_ptr(loc=loc, ip=ip)
    generic_llvm_ptr = llvm.addrspacecast(
        llvm.PointerType.get(AddressSpace.generic), llvm_ptr, loc=loc, ip=ip
    )
    # Create a new cute.Pointer with generic address space, preserving alignment
    return cute.make_ptr(
        gmem_ptr.dtype,
        generic_llvm_ptr,
        AddressSpace.generic,
        assumed_align=gmem_ptr.alignment,
        loc=loc,
        ip=ip,
    )
~~~~

**EN**: Defines `gmem_ptr_to_generic` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 将 `gmem_ptr_to_generic` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 119-144 / 第 119-144 行

~~~~python
@dsl_user_op
def generic_ptr_to_gmem(
    generic_ptr: Pointer,
    *,
    loc: Optional[ir.Location] = None,
    ip: Optional[ir.InsertionPoint] = None,
) -> Pointer:
    if generic_ptr.memspace != AddressSpace.generic:
        raise ValueError(
            f"generic_ptr_to_gmem requires pointer in generic address space, "
            f"got {generic_ptr.memspace}"
        )
    # Get LLVM pointer and cast to gmem address space
    llvm_ptr = generic_ptr.to_llvm_ptr(loc=loc, ip=ip)
    gmem_llvm_ptr = llvm.addrspacecast(
        llvm.PointerType.get(AddressSpace.gmem), llvm_ptr, loc=loc, ip=ip
    )
    # Create a new cute.Pointer with gmem address space, preserving alignment
    return cute.make_ptr(
        generic_ptr.dtype,
        gmem_llvm_ptr,
        AddressSpace.gmem,
        assumed_align=generic_ptr.alignment,
        loc=loc,
        ip=ip,
    )
~~~~

**EN**: Defines `generic_ptr_to_gmem` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 将 `generic_ptr_to_gmem` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 147-171 / 第 147-171 行

~~~~python
@dsl_user_op
def prefetch_tma_descriptor(tma_desc_ptr: Pointer, *, loc=None, ip=None) -> None:
    """
    Prefetch a TMA descriptor from global memory.

    This function prefetches the TMA descriptor pointed to by tma_desc_ptr
    into the TMA descriptor cache. The pointer must be in generic or global
    address space. If a gmem pointer is passed, it will be automatically
    converted to generic address space.

    :param tma_desc_ptr: Pointer to the TMA descriptor in global or generic memory
    :type tma_desc_ptr: Pointer
    :raises ValueError: If pointer is not in generic or global address space
    """
    if tma_desc_ptr.memspace not in (AddressSpace.gmem, AddressSpace.generic):
        raise ValueError(
            f"prefetch_tma_descriptor requires pointer in gmem or generic address space, "
            f"got {tma_desc_ptr.memspace}"
        )
    # Convert gmem pointer to generic if needed
    if tma_desc_ptr.memspace == AddressSpace.gmem:
        tma_desc_ptr = gmem_ptr_to_generic(tma_desc_ptr, loc=loc, ip=ip)
    # Convert cute.Pointer to LLVM pointer for prefetch
    llvm_ptr = tma_desc_ptr.to_llvm_ptr(loc=loc, ip=ip)
    from cutlass.cute.arch.nvvm_wrappers import prefetch as nvvm_prefetch
~~~~

**EN**: Defines `prefetch_tma_descriptor` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 将 `prefetch_tma_descriptor` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 173-173 / 第 173-173 行

~~~~python
    nvvm_prefetch(llvm_ptr, tensormap=True, loc=loc, ip=ip)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Drops to low-level PTX/NVVM operations when the tutorial needs direct control over the generated GPU instructions.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 当教程需要直接控制生成的 GPU 指令时，下沉到低层 PTX/NVVM 操作。

### Lines 176-179 / 第 176-179 行

~~~~python
def ptr_offset_bytes(ptr: Pointer, byte_offset: int) -> Pointer:
    """Offset a pointer by a given number of bytes."""
    element_offset = byte_offset * 8 // ptr.dtype.width
    return ptr + element_offset
~~~~

**EN**: Defines `ptr_offset_bytes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `ptr_offset_bytes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

### Lines 182-201 / 第 182-201 行

~~~~python
@dsl_user_op
def tensormap_ptr_for_copy(raw_ptr: Pointer, *, loc=None, ip=None) -> Pointer:
    """
    Convert a raw TMA descriptor gmem pointer to the type expected by cute.copy.

    cute.copy requires the tma_desc_ptr to be in generic address space and
    recast to TmaDescriptorTiledType. This utility performs both conversions.

    :param raw_ptr: Raw pointer to TMA descriptor in gmem
    :type raw_ptr: Pointer
    :return: Pointer compatible with cute.copy's tma_desc_ptr parameter
    :rtype: Pointer
    """
    generic_ptr = gmem_ptr_to_generic(raw_ptr, loc=loc, ip=ip)
    tma_desc_ptr_ty = _cute_ir.PtrType.get(
        _cute_nvgpu_ir.TmaDescriptorTiledType.get(),
        generic_ptr.memspace,
        generic_ptr.alignment,
    )
    return _cute_ir.recast_iter(tma_desc_ptr_ty, generic_ptr.value)
~~~~

**EN**: Defines `tensormap_ptr_for_copy` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 将 `tensormap_ptr_for_copy` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 204-206 / 第 204-206 行

~~~~python
# =============================================================================
# MoE Utilities
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 209-230 / 第 209-230 行

~~~~python
@dsl_user_op
@cute.jit
def compute_expert_token_range(
    offs: cute.Tensor,
    expert_idx: Int32,
    *,
    loc=None,
    ip=None,
) -> Tuple[Int32, Int32]:
    """
    Compute token offset and count for a given expert from the cumsum offs tensor.

    :param offs: Cumulative sum tensor of token counts per expert, shape (experts,)
    :param expert_idx: Index of the expert
    :return: (token_offset, tokens_i) where token_offset is the start position
             and tokens_i is the number of tokens for this expert
    """
    token_offset = Int32(0)
    if expert_idx > Int32(0):
        token_offset = offs[expert_idx - 1]  # type: ignore[assignment]
    tokens_i = offs[expert_idx] - token_offset
    return token_offset, tokens_i
~~~~

**EN**: Marks `compute_expert_token_range` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `compute_expert_token_range` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 233-253 / 第 233-253 行

~~~~python
@dsl_user_op
def rewrite_tensor_shape(
    tensor: cute.Tensor,
    new_shape: Tuple,
    *,
    loc=None,
    ip=None,
) -> cute.Tensor:
    """
    Rewrite tensor shape while keeping the same stride and iterator.

    This is primarily for debug friendliness - shows the actual expert's shape
    instead of the fake global shape. No runtime overhead as it becomes
    dead code in non-debug builds.

    :param tensor: Source tensor whose stride and iterator to preserve
    :param new_shape: New shape to apply
    :return: New tensor with the given shape but original stride and iterator
    """
    new_layout = cute.make_layout(new_shape, stride=tensor.stride, loc=loc, ip=ip)
    return cute.make_tensor(tensor.iterator, new_layout, loc=loc, ip=ip)
~~~~

**EN**: Defines `rewrite_tensor_shape` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 将 `rewrite_tensor_shape` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 256-258 / 第 256-258 行

~~~~python
# =============================================================================
# TMA Descriptor Workspace Helper
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 261-280 / 第 261-280 行

~~~~python
class TensormapWorkspace:
    """
    Helper for linear workspace layout of TMA descriptors.

    Manages address calculation for a workspace buffer containing TMA descriptors
    organized as: for each executor (e.g., expert or group), a fixed set of
    named descriptor slots.

    Layout: [slot_0_exec_0, slot_1_exec_0, ..., slot_0_exec_1, slot_1_exec_1, ...]

    Example:
        # 2Dx3D MoE: only C is expert-wise
        workspace = TensormapWorkspace(workspace_ptr, ["c"])

        # 2Dx2D MoE: A and B are expert-wise
        workspace = TensormapWorkspace(workspace_ptr, ["a", "b"])

        # General grouped GEMM: all three tensors
        workspace = TensormapWorkspace(workspace_ptr, ["a", "b", "c"])
    """
~~~~

**EN**: Defines `TensormapWorkspace`, a reusable Python class that packages configuration and behavior for this example. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `TensormapWorkspace`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 282-290 / 第 282-290 行

~~~~python
    def __init__(self, workspace_ptr: Pointer, slot_names: list):
        """
        :param workspace_ptr: Pointer to the beginning of the workspace buffer
        :param slot_names: Ordered list of tensor names, defining the slot layout
                           per executor. e.g., ["a", "b", "c"]
        """
        self.workspace_ptr = workspace_ptr
        self._name_to_slot = {name: i for i, name in enumerate(slot_names)}
        self._slots_per_executor = len(slot_names)
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 292-312 / 第 292-312 行

~~~~python
    @cute.jit
    def get_ptr(self, tensor_name: str, executor_idx: Int32) -> Pointer:
        """
        Get the workspace pointer for a specific TMA descriptor.

        :param tensor_name: Name of the tensor (must be one of the slot_names)
        :param executor_idx: Index of the executor (e.g., group_idx or expert_idx)
        :return: Aligned pointer to the TMA descriptor in workspace
        """
        if cutlass.const_expr(tensor_name not in self._name_to_slot):
            raise ValueError(
                f"Invalid tensor_name '{tensor_name}', "
                f"expected one of {list(self._name_to_slot.keys())}"
            )
        slot = self._name_to_slot[tensor_name]
        byte_offset = (
            executor_idx * self._slots_per_executor + slot
        ) * TensormapDescBytes
        return ptr_offset_bytes(self.workspace_ptr, byte_offset).align(
            TensormapDescBytes
        )
~~~~

**EN**: Marks `get_ptr` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 将 `get_ptr` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 314-323 / 第 314-323 行

~~~~python
    @staticmethod
    def size_bytes(num_slots: int, num_executors: int) -> int:
        """
        Calculate workspace size in bytes.

        :param num_slots: Number of descriptor slots per executor
        :param num_executors: Total number of executors (e.g., expert_cnt or group_cnt)
        :return: Total workspace size in bytes
        """
        return num_slots * num_executors * TensormapDescBytes
~~~~

**EN**: Defines `size_bytes`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `size_bytes`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 326-328 / 第 326-328 行

~~~~python
# =============================================================================
# Online TMA Descriptor Creator (Abstract Base Class)
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 331-345 / 第 331-345 行

~~~~python
@dataclass(frozen=True)
class OnlineTensormapDescCreator(ABC):
    """
    Abstract base class for building TMA descriptors online (at kernel runtime).

    Subclasses store all needed parameters (both codegen-time configs and runtime
    values) as explicit instance attributes in __init__. No dict-based APIs.

    Subclasses must implement exactly 2 abstract methods:
    - construct_and_write: Build TMA descriptor(s) for one executor and write to workspace
    - get_desc_ptr: Return raw gmem pointer to a specific descriptor in workspace

    To convert the raw pointer for use with cute.copy, callers should use the
    standalone tensormap_ptr_for_copy() utility.
    """
~~~~

**EN**: Defines `OnlineTensormapDescCreator`, a reusable Python class that packages configuration and behavior for this example. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `OnlineTensormapDescCreator`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 347-359 / 第 347-359 行

~~~~python
    @abstractmethod
    def construct_and_write(self, executor_idx: Int32, dependency=None) -> None:
        """
        Build TMA descriptor(s) for one executor and write to workspace.

        :param executor_idx: Index of the executor (e.g., group_idx or expert_idx).
            Semantics may vary by subclass when ``dependency`` is provided.
        :param dependency: Optional pipeline consumer for inter-warp-group
            synchronization. When provided, the subclass decides when to wait
            (via ``dependency.wait_and_advance()``) and release. The subclass
            also decides how to interpret ``executor_idx`` in this mode.
        """
        ...
~~~~

**EN**: Defines `construct_and_write`, grouping related logic behind a named Python callable so the example stays modular and reusable. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `construct_and_write`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 361-370 / 第 361-370 行

~~~~python
    @abstractmethod
    def get_desc_ptr(self, tensor_name: str, executor_idx: Int32) -> Pointer:
        """
        Get the raw gmem pointer to a specific TMA descriptor in workspace.

        :param tensor_name: Name identifying which tensor's descriptor
        :param executor_idx: Index of the executor (e.g., group_idx or expert_idx)
        :return: Raw pointer (gmem) to the TMA descriptor
        """
        ...
~~~~

**EN**: Defines `get_desc_ptr`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `get_desc_ptr`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 373-375 / 第 373-375 行

~~~~python
# =============================================================================
# MoE Grouped GEMM Tensormap Constructor
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 378-392 / 第 378-392 行

~~~~python
class MoEGroupedGemmTensormapConstructor(OnlineTensormapDescCreator):
    """
    Tensormap descriptor constructor for MoE Grouped GEMM (expert-wise descriptors only).

    Non-expert-wise descriptors are passed directly at kernel launch.
    This class only handles:
    - 2Dx3D: C descriptors (expert-wise, to avoid write conflicts)
    - 2Dx2D: A and B descriptors (expert-wise, tokens is reduction axis)

    All parameters are stored as explicit instance attributes (no dicts).

    Workspace layout:
    - 2Dx3D: [C_0, C_1, ..., C_{n-1}]
    - 2Dx2D: [A_0, A_1, ..., A_{n-1}, B_0, B_1, ..., B_{n-1}]
    """
~~~~

**EN**: Defines `MoEGroupedGemmTensormapConstructor`, a reusable Python class that packages configuration and behavior for this example. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `MoEGroupedGemmTensormapConstructor`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 394-421 / 第 394-421 行

~~~~python
    def __init__(
        self,
        scenario: Literal["2Dx3D", "2Dx2D"],
        # Codegen-time configs
        a_dtype,
        b_dtype,
        c_dtype,
        a_smem_layout,
        b_smem_layout,
        epi_smem_layout,
        a_tma_op,
        b_tma_op,
        c_tma_op,
        tiled_mma,
        mma_tiler,
        cluster_layout_vmnk_shape,
        epi_tile,
        # Runtime params
        a_tensor: cute.Tensor,  # fake GEMM domain A
        b_tensor: cute.Tensor,  # fake GEMM domain B
        c_tensor: cute.Tensor,  # fake GEMM domain C
        offs: cute.Tensor,  # (experts,) cumsum
        workspace_ptr: Pointer,
    ) -> None:
        super().__init__()
        self.scenario = scenario
        # Codegen-time configs
        self.a_dtype = a_dtype
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 422-443 / 第 422-443 行

~~~~python
        self.b_dtype = b_dtype
        self.c_dtype = c_dtype
        self.a_smem_layout = a_smem_layout
        self.b_smem_layout = b_smem_layout
        self.epi_smem_layout = epi_smem_layout
        self.a_tma_op = a_tma_op
        self.b_tma_op = b_tma_op
        self.c_tma_op = c_tma_op
        self.tiled_mma = tiled_mma
        self.mma_tiler = mma_tiler
        self.cluster_layout_vmnk_shape = cluster_layout_vmnk_shape
        self.epi_tile = epi_tile
        # Runtime params
        self.a_tensor = a_tensor
        self.b_tensor = b_tensor
        self.c_tensor = c_tensor
        self.offs = offs
        # Workspace with scenario-specific slot layout
        if scenario == "2Dx3D":
            self.workspace = TensormapWorkspace(workspace_ptr, ["c"])
        else:
            self.workspace = TensormapWorkspace(workspace_ptr, ["a", "b"])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 445-451 / 第 445-451 行

~~~~python
    @staticmethod
    def get_workspace_size(scenario: Literal["2Dx3D", "2Dx2D"], expert_cnt: int) -> int:
        """Calculate workspace size in bytes for tensormap descriptors."""
        if scenario == "2Dx3D":
            return TensormapWorkspace.size_bytes(1, expert_cnt)  # only C
        else:
            return TensormapWorkspace.size_bytes(2, expert_cnt)  # A and B
~~~~

**EN**: Defines `get_workspace_size`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `get_workspace_size`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 453-455 / 第 453-455 行

~~~~python
    @cute.jit
    def get_desc_ptr(self, tensor_name: str, executor_idx: Int32) -> Pointer:
        return self.workspace.get_ptr(tensor_name, executor_idx)
~~~~

**EN**: Marks `get_desc_ptr` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Returns the constructed object or computed result to the caller.
**CN**: 将 `get_desc_ptr` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 把构造好的对象或计算结果返回给调用方。

### Lines 457-468 / 第 457-468 行

~~~~python
    @cute.jit
    def construct_and_write(self, executor_idx: Int32, dependency=None) -> None:
        """
        Create expert-wise tensormap descriptors for the given expert.

        - 2Dx3D: Creates C descriptor for this expert
        - 2Dx2D: Creates A and B descriptors for this expert
        """
        if cutlass.const_expr(self.scenario == "2Dx3D"):
            self._construct_c_desc_2dx3d(executor_idx)
        else:  # 2Dx2D
            self._construct_ab_descs_2dx2d(executor_idx)
~~~~

**EN**: Marks `construct_and_write` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `construct_and_write` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 470-477 / 第 470-477 行

~~~~python
    @cute.jit
    def _construct_c_desc_2dx3d(self, expert_idx: Int32) -> None:
        """
        2Dx3D: Create expert-wise C descriptor.
        C tensor: (fake_m, n, 1) = (tokens_sum, intermediate, 1)
        Slice fake_m -> (tokens_i, intermediate, 1) per expert.
        """
        token_offset, tokens_i = compute_expert_token_range(self.offs, expert_idx)
~~~~

**EN**: Marks `_construct_c_desc_2dx3d` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_construct_c_desc_2dx3d` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 479-481 / 第 479-481 行

~~~~python
        c_ptr = self.c_tensor.iterator
        c_stride = self.c_tensor.stride
        intermediate = self.c_tensor.shape[1]  # type: ignore[index]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 483-484 / 第 483-484 行

~~~~python
        c1 = cutlass.Int32(1)
        c0 = cutlass.Int32(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 486-491 / 第 486-491 行

~~~~python
        c_ptr_i = c_ptr + token_offset * c_stride[0]  # type: ignore[index]
        c_layout_i = cute.make_layout(
            (tokens_i, intermediate, c1),
            stride=(c_stride[0], c_stride[1], c0),  # type: ignore[index]
        )
        c_tensor_i = cute.make_tensor(c_ptr_i, c_layout_i)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 493-499 / 第 493-499 行

~~~~python
        tma_atom_c, _ = cpasync.make_tiled_tma_atom(
            self.c_tma_op,
            c_tensor_i,
            self.epi_smem_layout,
            self.epi_tile,
        )
        cpasync.copy_tensormap(tma_atom_c, self.get_desc_ptr("c", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 501-508 / 第 501-508 行

~~~~python
    @cute.jit
    def _construct_ab_descs_2dx2d(self, expert_idx: Int32) -> None:
        """
        2Dx2D: Create expert-wise A and B descriptors.
        A: (m, fake_k, 1) -> slice to (m, tokens_i, 1)
        B: (n, fake_k, 1) -> slice to (n, tokens_i, 1)
        """
        token_offset, tokens_i = compute_expert_token_range(self.offs, expert_idx)
~~~~

**EN**: Marks `_construct_ab_descs_2dx2d` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_construct_ab_descs_2dx2d` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 510-511 / 第 510-511 行

~~~~python
        c1 = cutlass.Int32(1)
        c0 = cutlass.Int32(0)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 513-513 / 第 513-513 行

~~~~python
        # A tensor: (m, fake_k, 1) -> (m, tokens_i, 1)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 514-516 / 第 514-516 行

~~~~python
        a_ptr = self.a_tensor.iterator
        a_stride = self.a_tensor.stride
        a_m = self.a_tensor.shape[0]  # type: ignore[index]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 518-523 / 第 518-523 行

~~~~python
        a_ptr_i = a_ptr + token_offset * a_stride[1]  # type: ignore[index]
        a_layout_i = cute.make_layout(
            (a_m, tokens_i, c1),
            stride=(a_stride[0], a_stride[1], c0),  # type: ignore[index]
        )
        a_tensor_i = cute.make_tensor(a_ptr_i, a_layout_i)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 525-533 / 第 525-533 行

~~~~python
        tma_atom_a, _ = cute.nvgpu.make_tiled_tma_atom_A(
            self.a_tma_op,
            a_tensor_i,
            self.a_smem_layout,
            self.mma_tiler,
            self.tiled_mma,
            self.cluster_layout_vmnk_shape,
        )
        cpasync.copy_tensormap(tma_atom_a, self.get_desc_ptr("a", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 535-535 / 第 535-535 行

~~~~python
        # B tensor: (n, fake_k, 1) -> (n, tokens_i, 1)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 536-538 / 第 536-538 行

~~~~python
        b_ptr = self.b_tensor.iterator
        b_stride = self.b_tensor.stride
        b_n = self.b_tensor.shape[0]  # type: ignore[index]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 540-545 / 第 540-545 行

~~~~python
        b_ptr_i = b_ptr + token_offset * b_stride[1]  # type: ignore[index]
        b_layout_i = cute.make_layout(
            (b_n, tokens_i, c1),
            stride=(b_stride[0], b_stride[1], c0),  # type: ignore[index]
        )
        b_tensor_i = cute.make_tensor(b_ptr_i, b_layout_i)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds explicit tensor layouts, so logical coordinates map cleanly onto CUDA memory addresses and traversal order.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建显式张量布局，使逻辑坐标能够清晰地映射到 CUDA 内存地址和遍历顺序。

### Lines 547-555 / 第 547-555 行

~~~~python
        tma_atom_b, _ = cute.nvgpu.make_tiled_tma_atom_B(
            self.b_tma_op,
            b_tensor_i,
            self.b_smem_layout,
            self.mma_tiler,
            self.tiled_mma,
            self.cluster_layout_vmnk_shape,
        )
        cpasync.copy_tensormap(tma_atom_b, self.get_desc_ptr("b", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 558-560 / 第 558-560 行

~~~~python
# =============================================================================
# MoE Scaled Grouped GEMM Tensormap Constructor
# =============================================================================
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 563-590 / 第 563-590 行

~~~~python
class MoEScaledGroupedGemmTensormapConstructor(OnlineTensormapDescCreator):
    """
    Tensormap descriptor constructor for MoE Scaled Grouped GEMM (block-scaled).

    .. py:attribute:: ChunkSize
        :value: 128

        Number of experts processed per chunk in the desc_init_kernel.
        Must match the warp-group width (4 warps × 32 threads).

    Extends MoEGroupedGemmTensormapConstructor with SFA/SFB descriptor support.

    Expert-wise descriptors only — non-expert-wise descriptors are passed
    directly at kernel launch.

    Workspace layout:
    - 2Dx3D: [C_0, C_1, ..., C_{n-1}]  (1 slot per expert)
    - 2Dx2D: [A_0, B_0, SFA_0, SFB_0, A_1, B_1, SFA_1, SFB_1, ...]  (4 slots per expert)

    :param scenario: "2Dx3D" or "2Dx2D"
    :param sf_vec_size: Scale factor vector size (32 for MXFP8/MXFP4, 16 for NVFP4)
    :param a_dtype: Data type for tensor A
    :param b_dtype: Data type for tensor B
    :param c_dtype: Data type for tensor C
    :param sf_dtype: Data type for scale factors (SFA/SFB)
    :param a_smem_layout: SMEM layout for A TMA
    :param b_smem_layout: SMEM layout for B TMA
    :param epi_smem_layout: SMEM layout for epilogue (C) TMA
~~~~

**EN**: Defines `MoEScaledGroupedGemmTensormapConstructor`, a reusable Python class that packages configuration and behavior for this example. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 定义 `MoEScaledGroupedGemmTensormapConstructor`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 591-614 / 第 591-614 行

~~~~python
    :param sfa_smem_layout: SMEM layout for SFA TMA
    :param sfb_smem_layout: SMEM layout for SFB TMA
    :param a_tma_op: TMA operation for A
    :param b_tma_op: TMA operation for B
    :param c_tma_op: TMA operation for C (S2G store or reduce)
    :param sfa_tma_op: TMA operation for SFA
    :param sfb_tma_op: TMA operation for SFB
    :param tiled_mma: TiledMma for A/B/SFA/C TMA atom construction
    :param tiled_mma_sfb: TiledMma for SFB (separate due to 2CTA replication)
    :param mma_tiler: MMA tiler shape (M, N, K)
    :param mma_tiler_sfb: MMA tiler shape for SFB
    :param cluster_layout_vmnk_shape: Cluster layout shape for A/B/SFA multicast
    :param cluster_layout_sfb_vmnk_shape: Cluster layout shape for SFB multicast
    :param epi_tile: Epilogue tile shape
    :param a_tensor: Fake GEMM domain A tensor
    :param b_tensor: Fake GEMM domain B tensor
    :param c_tensor: Fake GEMM domain C tensor
    :param sfa_tensor: Fake GEMM domain SFA tensor (atom-tiled layout)
    :param sfb_tensor: Fake GEMM domain SFB tensor (atom-tiled layout)
    :param offs: (experts,) cumsum offsets in data domain
    :param offs_padded: (experts,) cumsum offsets in padded scale domain
    :param workspace_ptr: Pointer to workspace for TMA descriptors
    :param expert_cnt: Total number of experts
    """
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 616-616 / 第 616-616 行

~~~~python
    ChunkSize = 128
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 618-645 / 第 618-645 行

~~~~python
    def __init__(
        self,
        scenario: Literal["2Dx3D", "2Dx2D"],
        sf_vec_size: int,
        # Codegen-time configs: dtypes
        a_dtype,
        b_dtype,
        c_dtype,
        sf_dtype,
        # Codegen-time configs: SMEM layouts
        a_smem_layout,
        b_smem_layout,
        epi_smem_layout,
        sfa_smem_layout,
        sfb_smem_layout,
        # Codegen-time configs: TMA ops
        a_tma_op,
        b_tma_op,
        c_tma_op,
        sfa_tma_op,
        sfb_tma_op,
        # Codegen-time configs: MMA / cluster / tile
        tiled_mma,
        tiled_mma_sfb,
        mma_tiler,
        mma_tiler_sfb,
        cluster_layout_vmnk_shape,
        cluster_layout_sfb_vmnk_shape,
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 646-673 / 第 646-673 行

~~~~python
        epi_tile,
        # Runtime params
        a_tensor: cute.Tensor,
        b_tensor: cute.Tensor,
        c_tensor: cute.Tensor,
        sfa_tensor: cute.Tensor,
        sfb_tensor: cute.Tensor,
        offs: cute.Tensor,
        offs_padded: cute.Tensor,
        workspace_ptr: Pointer,
        expert_cnt: Optional[Union[Int32, int]] = None,
    ) -> None:
        super().__init__()
        self.scenario = scenario
        self.sf_vec_size = sf_vec_size
        # Dtypes
        self.a_dtype = a_dtype
        self.b_dtype = b_dtype
        self.c_dtype = c_dtype
        self.sf_dtype = sf_dtype
        # SMEM layouts
        self.a_smem_layout = a_smem_layout
        self.b_smem_layout = b_smem_layout
        self.epi_smem_layout = epi_smem_layout
        self.sfa_smem_layout = sfa_smem_layout
        self.sfb_smem_layout = sfb_smem_layout
        # TMA ops
        self.a_tma_op = a_tma_op
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 674-699 / 第 674-699 行

~~~~python
        self.b_tma_op = b_tma_op
        self.c_tma_op = c_tma_op
        self.sfa_tma_op = sfa_tma_op
        self.sfb_tma_op = sfb_tma_op
        # MMA / cluster / tile
        self.tiled_mma = tiled_mma
        self.tiled_mma_sfb = tiled_mma_sfb
        self.mma_tiler = mma_tiler
        self.mma_tiler_sfb = mma_tiler_sfb
        self.cluster_layout_vmnk_shape = cluster_layout_vmnk_shape
        self.cluster_layout_sfb_vmnk_shape = cluster_layout_sfb_vmnk_shape
        self.epi_tile = epi_tile
        # Runtime params
        self.a_tensor = a_tensor
        self.b_tensor = b_tensor
        self.c_tensor = c_tensor
        self.sfa_tensor = sfa_tensor
        self.sfb_tensor = sfb_tensor
        self.offs = offs
        self.offs_padded = offs_padded
        self.expert_cnt = expert_cnt
        # Workspace with scenario-specific slot layout
        if scenario == "2Dx3D":
            self.workspace = TensormapWorkspace(workspace_ptr, ["c"])
        else:
            self.workspace = TensormapWorkspace(workspace_ptr, ["a", "b", "sfa", "sfb"])
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 701-707 / 第 701-707 行

~~~~python
    @staticmethod
    def get_workspace_size(scenario: Literal["2Dx3D", "2Dx2D"], expert_cnt: int) -> int:
        """Calculate workspace size in bytes for tensormap descriptors."""
        if scenario == "2Dx3D":
            return TensormapWorkspace.size_bytes(1, expert_cnt)  # C only
        else:
            return TensormapWorkspace.size_bytes(4, expert_cnt)  # A, B, SFA, SFB
~~~~

**EN**: Defines `get_workspace_size`, grouping related logic behind a named Python callable so the example stays modular and reusable. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Returns the constructed object or computed result to the caller.
**CN**: 定义 `get_workspace_size`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 把构造好的对象或计算结果返回给调用方。

### Lines 709-711 / 第 709-711 行

~~~~python
    @cute.jit
    def get_desc_ptr(self, tensor_name: str, executor_idx: Int32) -> Pointer:
        return self.workspace.get_ptr(tensor_name, executor_idx)
~~~~

**EN**: Marks `get_desc_ptr` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Returns the constructed object or computed result to the caller.
**CN**: 将 `get_desc_ptr` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 把构造好的对象或计算结果返回给调用方。

### Lines 713-739 / 第 713-739 行

~~~~python
    @cute.jit
    def construct_and_write(self, lane_in_group: Int32, dependency=None) -> None:
        """
        Create expert-wise tensormap descriptors for all experts.

        ``lane_in_group`` is the thread's position within its warp group
        (0..ChunkSize-1). The method loops internally over all experts in
        chunks of ``ChunkSize``, with two-phase pipeline synchronization
        per chunk.

        Per-chunk execution:

        1. Phase 1: Build descriptors that do NOT depend on ``offs_padded``
           (A/B for 2Dx2D, C for 2Dx3D). Overlaps with Group A's prefix sum.
        2. Barrier: ``consumer.wait_and_advance()`` — all threads participate.
        3. Phase 2: Build descriptors that depend on ``offs_padded``
           (SFA/SFB for 2Dx2D). Reads padded offsets from SMEM buffer.
        4. Release: ``handle.release()`` — all threads participate.

        :param lane_in_group: Thread's position within the warp group (0..127).
        :param dependency: ``(PipelineConsumer, smem_offs_padded)`` — the
            consumer for mbarrier sync, and the SMEM tensor of shape
            ``(ChunkSize + 1,)`` with layout ``[carry, offs_padded[0..127]]``.
        """
        consumer, smem_offs_padded = dependency
        assert self.expert_cnt is not None
        num_chunks = (self.expert_cnt + self.ChunkSize - 1) // self.ChunkSize
~~~~

**EN**: Marks `construct_and_write` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 将 `construct_and_write` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 741-744 / 第 741-744 行

~~~~python
        chunk_idx = cutlass.Int32(0)
        while chunk_idx < num_chunks:
            expert_idx = chunk_idx * self.ChunkSize + lane_in_group
            in_bounds = expert_idx < self.expert_cnt
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 746-746 / 第 746-746 行

~~~~python
            # Phase 1: non-dependent descriptors
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 747-751 / 第 747-751 行

~~~~python
            if in_bounds:
                if cutlass.const_expr(self.scenario == "2Dx2D"):
                    self._construct_ab_descs_2dx2d(expert_idx)
                else:
                    self._construct_c_desc_2dx3d(expert_idx)
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 753-753 / 第 753-753 行

~~~~python
            # All threads participate in barrier (fixed arrive count)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 754-754 / 第 754-754 行

~~~~python
            handle = consumer.wait_and_advance()
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Configures synchronization or a multistage pipeline so memory movement can overlap with computation safely.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 配置同步机制或多级流水线，使数据搬运能够与计算安全重叠。

### Lines 756-756 / 第 756-756 行

~~~~python
            # Phase 2: dependent descriptors (read padded offsets from SMEM)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 757-766 / 第 757-766 行

~~~~python
            if in_bounds:
                if cutlass.const_expr(self.scenario == "2Dx2D"):
                    # smem_offs_padded layout: [carry, chunk[0], ..., chunk[127]]
                    # padded_offset = smem[lane]     (prev expert's cumulative)
                    # padded_end    = smem[lane + 1] (this expert's cumulative)
                    padded_offset = smem_offs_padded[lane_in_group]
                    padded_size_i = smem_offs_padded[lane_in_group + 1] - padded_offset
                    self._construct_sf_descs_2dx2d_direct(
                        expert_idx, padded_offset, padded_size_i
                    )
~~~~

**EN**: Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 768-768 / 第 768-768 行

~~~~python
            # All threads release (fixed arrive count)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 769-769 / 第 769-769 行

~~~~python
            handle.release()
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 771-771 / 第 771-771 行

~~~~python
            chunk_idx += 1
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 773-775 / 第 773-775 行

~~~~python
    # -----------------------------------------------------------------
    # 2Dx3D: C descriptor (same as MoEGroupedGemmTensormapConstructor)
    # -----------------------------------------------------------------
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 777-784 / 第 777-784 行

~~~~python
    @cute.jit
    def _construct_c_desc_2dx3d(self, expert_idx: Int32) -> None:
        """
        2Dx3D: Create expert-wise C descriptor.
        C: (fake_m, n, 1) -> slice to (tokens_i, n, 1) per expert.
        """
        token_offset, tokens_i = compute_expert_token_range(self.offs, expert_idx)
        c1 = cutlass.Int32(1)
~~~~

**EN**: Marks `_construct_c_desc_2dx3d` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_construct_c_desc_2dx3d` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 786-787 / 第 786-787 行

~~~~python
        c_i = cute.domain_offset((token_offset, 0, 0), self.c_tensor)
        c_i = rewrite_tensor_shape(c_i, (tokens_i, self.c_tensor.shape[1], c1))  # type: ignore[index]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 789-795 / 第 789-795 行

~~~~python
        tma_atom_c, _ = cpasync.make_tiled_tma_atom(
            self.c_tma_op,
            c_i,
            self.epi_smem_layout,
            self.epi_tile,
        )
        cpasync.copy_tensormap(tma_atom_c, self.get_desc_ptr("c", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 797-799 / 第 797-799 行

~~~~python
    # -----------------------------------------------------------------
    # 2Dx2D: A, B descriptors (same as MoEGroupedGemmTensormapConstructor)
    # -----------------------------------------------------------------
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 801-809 / 第 801-809 行

~~~~python
    @cute.jit
    def _construct_ab_descs_2dx2d(self, expert_idx: Int32) -> None:
        """
        2Dx2D: Create expert-wise A and B descriptors.
        A: (m, fake_k, 1) -> slice to (m, tokens_i, 1)
        B: (n, fake_k, 1) -> slice to (n, tokens_i, 1)
        """
        token_offset, tokens_i = compute_expert_token_range(self.offs, expert_idx)
        c1 = cutlass.Int32(1)
~~~~

**EN**: Marks `_construct_ab_descs_2dx2d` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_construct_ab_descs_2dx2d` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 811-811 / 第 811-811 行

~~~~python
        # A: (m, fake_k, 1) -> domain_offset + rewrite shape
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 812-813 / 第 812-813 行

~~~~python
        a_i = cute.domain_offset((0, token_offset, 0), self.a_tensor)
        a_i = rewrite_tensor_shape(a_i, (self.a_tensor.shape[0], tokens_i, c1))  # type: ignore[index]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 815-823 / 第 815-823 行

~~~~python
        tma_atom_a, _ = cute.nvgpu.make_tiled_tma_atom_A(
            self.a_tma_op,
            a_i,
            self.a_smem_layout,
            self.mma_tiler,
            self.tiled_mma,
            self.cluster_layout_vmnk_shape,
        )
        cpasync.copy_tensormap(tma_atom_a, self.get_desc_ptr("a", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 825-825 / 第 825-825 行

~~~~python
        # B: (n, fake_k, 1) -> domain_offset + rewrite shape
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 826-827 / 第 826-827 行

~~~~python
        b_i = cute.domain_offset((0, token_offset, 0), self.b_tensor)
        b_i = rewrite_tensor_shape(b_i, (self.b_tensor.shape[0], tokens_i, c1))  # type: ignore[index]
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 829-837 / 第 829-837 行

~~~~python
        tma_atom_b, _ = cute.nvgpu.make_tiled_tma_atom_B(
            self.b_tma_op,
            b_i,
            self.b_smem_layout,
            self.mma_tiler,
            self.tiled_mma,
            self.cluster_layout_vmnk_shape,
        )
        cpasync.copy_tensormap(tma_atom_b, self.get_desc_ptr("b", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 839-841 / 第 839-841 行

~~~~python
    # -----------------------------------------------------------------
    # 2Dx2D: SFA, SFB descriptors (new for block-scaled)
    # -----------------------------------------------------------------
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 843-857 / 第 843-857 行

~~~~python
    @cute.jit
    def _construct_sf_descs_2dx2d_direct(
        self,
        expert_idx: Int32,
        padded_offset: Int32,
        padded_size_i: Int32,
    ) -> None:
        """
        2Dx2D: Create expert-wise SFA and SFB descriptors with pre-computed
        padded offset and size.

        This variant allows the caller to supply padded offsets from SMEM
        (in desc_init_kernel) instead of reading from ``self.offs_padded`` in GMEM.
        """
        c1 = cutlass.Int32(1)
~~~~

**EN**: Marks `_construct_sf_descs_2dx2d_direct` for JIT compilation, turning this callable into a specialized launcher or host-side wrapper for generated GPU code. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 将 `_construct_sf_descs_2dx2d_direct` 标记为 JIT 编译对象，使该可调用对象成为生成 GPU 代码的特化启动器或宿主端封装。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 859-876 / 第 859-876 行

~~~~python
        a_chunks_to_move = (
            padded_offset
            // self.sf_vec_size
            * cute.size(self.sfa_tensor, mode=[0])
            // 128
        )
        a_elems_to_move = (
            cute.size(self.sfa_tensor, mode=[0]) * padded_offset // self.sf_vec_size
        )
        b_chunks_to_move = (
            padded_offset
            // self.sf_vec_size
            * cute.size(self.sfb_tensor, mode=[0])
            // 128
        )
        b_elems_to_move = (
            cute.size(self.sfb_tensor, mode=[0]) * padded_offset // self.sf_vec_size
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 878-882 / 第 878-882 行

~~~~python
        per_expert_sfa_shape = (self.sfa_tensor.shape[0], padded_size_i, c1)  # type: ignore[index]
        sfa_layout_i = tile_atom_to_shape_SF(per_expert_sfa_shape, self.sf_vec_size)
        sfa_i = cute.make_tensor(
            self.sfa_tensor.iterator + a_elems_to_move, sfa_layout_i
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 884-893 / 第 884-893 行

~~~~python
        tma_atom_sfa, _ = cute.nvgpu.make_tiled_tma_atom_A(
            self.sfa_tma_op,
            sfa_i,
            self.sfa_smem_layout,
            self.mma_tiler,
            self.tiled_mma,
            self.cluster_layout_vmnk_shape,
            internal_type=cutlass.Uint64,
        )
        cpasync.copy_tensormap(tma_atom_sfa, self.get_desc_ptr("sfa", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 895-899 / 第 895-899 行

~~~~python
        per_expert_sfb_shape = (self.sfb_tensor.shape[0], padded_size_i, c1)  # type: ignore[index]
        sfb_layout_i = tile_atom_to_shape_SF(per_expert_sfb_shape, self.sf_vec_size)
        sfb_i = cute.make_tensor(
            self.sfb_tensor.iterator + b_elems_to_move, sfb_layout_i
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

### Lines 901-910 / 第 901-910 行

~~~~python
        tma_atom_sfb, _ = cute.nvgpu.make_tiled_tma_atom_B(
            self.sfb_tma_op,
            sfb_i,
            self.sfb_smem_layout,
            self.mma_tiler_sfb,
            self.tiled_mma_sfb,
            self.cluster_layout_sfb_vmnk_shape,
            internal_type=cutlass.Uint64,
        )
        cpasync.copy_tensormap(tma_atom_sfb, self.get_desc_ptr("sfb", expert_idx))
~~~~

**EN**: Sets up data movement primitives, which is how CuTeDSL describes copies between global memory, shared memory, registers, or TMA engines. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Works with mixture-of-experts routing or grouped GEMM structure, where scheduling and data packing are as important as math throughput.
**CN**: 设置数据搬运原语；CuTeDSL 通过这些原语描述全局内存、共享内存、寄存器或 TMA 引擎之间的拷贝。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 处理 MoE 路由或分组 GEMM 结构，此时调度与数据打包和数学吞吐同样重要。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Multistage pipeline and synchronization / 多级流水线与同步
- Tensor layout transforms and tiling / 张量布局变换与分块

## Dependencies / 依赖项

- `abc.ABC` — used by this example / 供该示例使用
- `abc.abstractmethod` — used by this example / 供该示例使用
- `typing.Optional` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Literal` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `typing.Union` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.typing.AddressSpace` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.typing.Pointer` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.cute.nvgpu.cpasync` — provides asynchronous copy and TMA primitives / 提供异步拷贝与 TMA 原语
- `cutlass.cutlass_dsl.dsl_user_op` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.Int32` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass._mlir.ir` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass._mlir.dialects.llvm` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass._mlir.dialects.cute` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass._mlir.dialects.cute_nvgpu` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `dataclasses.dataclass` — used by this example / 供该示例使用
- `cutlass.utils.blockscaled_layout.tile_atom_to_shape_SF` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.arch.nvvm_wrappers.prefetch` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
