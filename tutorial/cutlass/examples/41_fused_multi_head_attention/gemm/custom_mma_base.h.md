# custom_mma_base.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm/custom_mma_base.h`
**Purpose / 用途**: Shared scaffold for FMHA threadblock MMA kernels, defining stage-aware shared storage, warp iterators, and common policy-derived aliases / FMHA threadblock MMA 内核的公共骨架，定义 stage 感知的共享内存、warp 迭代器和通用策略别名
---
## Line-by-Line Analysis / 逐行分析
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
/*! \file
    \brief Template for a double-buffered threadblock-scoped GEMM kernel.
*/

#pragma once

#include "cutlass/aligned_buffer.h"
#include "cutlass/arch/memory.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/threadblock/mma_base.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"
```
**EN**: Lines 1-45. The opening block is mostly legal boilerplate plus the dependency list. The included CUTLASS headers reveal the role of this file: it needs aligned shared-memory buffers, tensor references, GEMM shape utilities, and the base mma abstractions that derived kernels will reuse.
**CN**: 第1-45行：开头部分主要是许可证说明和依赖头文件列表。这里包含的 CUTLASS 头文件表明了本文件的职责：它需要对齐的共享内存缓冲区、TensorRef、GEMM 形状工具，以及供派生内核复用的基础 mma 抽象。
```cpp
////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace gemm {
namespace threadblock {

////////////////////////////////////////////////////////////////////////////////

/// Structure to compute the matrix product targeting CUDA cores and SIMT math
/// instructions.
template <
    /// Size of the Gemm problem - concept: gemm::GemmShape<>
    typename Shape_,
    /// Policy describing tuning details (concept: MmaPolicy)
    typename Policy_,
    /// Number of stages,
    int Stages,
    /// Used for partial specialization
    typename Enable = bool>
class CustomMmaBase {
 public:
  ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using Shape = Shape_;

  ///< Policy describing tuning details
  using Policy = Policy_;

  //
  // Dependent types
  //

  /// Warp-level Mma
  using Operator = typename Policy::Operator;

  /// Shape describing the overall GEMM computed from shared memory
  /// by each warp.
  using WarpGemm = typename Policy::Operator::Shape;

  /// Shape describing the number of warps filling the CTA
  using WarpCount = GemmShape<
      Shape::kM / WarpGemm::kM,
      Shape::kN / WarpGemm::kN,
      Shape::kK / WarpGemm::kK>;

  /// Number of warp-level GEMM oeprations
  static int const kWarpGemmIterations =
      (WarpGemm::kK / Operator::Policy::MmaShape::kK);

  /// Number of stages
  static int const kStages = Stages;

  //
```
**EN**: Lines 46-97. The primary template binds a threadblock tile shape, an MMA policy, and a compile-time stage count. From those inputs it derives the warp operator, the per-warp GEMM tile, the warp grid inside the CTA, and the number of warp-level K-groups each threadblock iteration must execute.
**CN**: 第46-97行：主模板把 threadblock tile 形状、MMA policy 和编译期 stage 数绑定在一起。基于这些输入，它推导出 warp 级算子、每个 warp 负责的 GEMM tile、CTA 内部的 warp 网格，以及每次 threadblock 迭代需要执行的 warp 级 K 分组数量。
```cpp
  // Nested structs
  //

  /// Shared storage object needed by threadblock-scoped GEMM
  template <typename Element, typename OperandShape, typename OperandLayout>
  struct OperandSharedStorage {
    AlignedBuffer<Element, OperandShape::kCount> buffer;
    using TensorRef = TensorRef<Element, OperandLayout>;

    CUTLASS_DEVICE
    static OperandLayout Layout() {
      return OperandLayout::packed({OperandShape::kRow, OperandShape::kColumn});
    }

    /// Returns a TensorRef to the operand
    CUTLASS_HOST_DEVICE
    TensorRef ref() {
      return TensorRef{buffer.data(), Layout()};
    }
  };
```
**EN**: Lines 98-117. OperandSharedStorage is a reusable wrapper for one shared-memory operand tile. It couples an AlignedBuffer with a packed layout constructor and a TensorRef factory, so both A and B operands can be described uniformly and handed to iterator types without duplicating boilerplate.
**CN**: 第98-117行：OperandSharedStorage 是对单个共享内存操作数 tile 的可复用封装。它把 AlignedBuffer、packed 布局构造函数以及 TensorRef 工厂组合在一起，使 A/B 两个操作数都能用统一方式描述，并交给迭代器类型使用，而无需重复模板样板代码。
```cpp

  /// Shape of the A matrix operand in shared memory
  using ShapeA = MatrixShape<
      Shape::kM + Policy::SmemPaddingA::kRow,
      Shape::kK * kStages + Policy::SmemPaddingA::kColumn>;

  /// Shape of the B matrix operand in shared memory
  using ShapeB = MatrixShape<
      Shape::kK * kStages + Policy::SmemPaddingB::kRow,
      Shape::kN + Policy::SmemPaddingB::kColumn>;

  using SharedStorageA = OperandSharedStorage<
      typename Operator::ElementA,
      ShapeA,
      typename Operator::LayoutA>;
  using SharedStorageB = OperandSharedStorage<
      typename Operator::ElementB,
      ShapeB,
      typename Operator::LayoutB>;
  using TensorRefA = typename SharedStorageA::TensorRef;
  using TensorRefB = typename SharedStorageB::TensorRef;

  struct SharedStorage {
    /// Buffer for A operand
    SharedStorageA operand_A;

    /// Buffer for B operand
    SharedStorageB operand_B;
  };
```
**EN**: Lines 118-146. These aliases materialize the shared-memory footprint for A and B. The K dimension is multiplied by kStages to reserve a circular-buffer window across stages, while policy-provided padding adjusts row/column extents to satisfy layout alignment or bank-conflict constraints. SharedStorage simply aggregates both operands.
**CN**: 第118-146行：这些类型别名具体化了 A 和 B 在共享内存中的占用。K 维乘以 kStages，用于为各 stage 预留循环缓冲窗口；而 policy 提供的 padding 会调整行/列范围，以满足布局对齐或避免 bank conflict 的需求。SharedStorage 只是把两个操作数组合在一起。
```cpp

 protected:
  //
  // Data members
  //

  /// Iterator to load a warp-scoped tile of A operand from shared memory
  typename Operator::IteratorA warp_tile_iterator_A_;

  /// Iterator to load a warp-scoped tile of B operand from shared memory
  typename Operator::IteratorB warp_tile_iterator_B_;

 public:
  /// Construct from tensor references
  CUTLASS_DEVICE
  CustomMmaBase(
      ///< Shared storage needed for internal use by threadblock-scoped GEMM
      SharedStorageA& shared_storageA,
      SharedStorageB& shared_storageB,
      ///< ID within the threadblock
      int thread_idx,
      ///< ID of warp
      int warp_idx,
      ///< ID of each thread within a warp
      int lane_idx)
      : warp_tile_iterator_A_(shared_storageA.ref(), lane_idx),
        warp_tile_iterator_B_(shared_storageB.ref(), lane_idx) {}
};

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace gemm
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Lines 147-182. The base class stores only the warp-level shared-memory iterators and initializes them from shared storage plus lane index. Notice that thread_idx and warp_idx are not used here yet: derived classes are responsible for mapping each warp to its tile offsets and for implementing the actual mainloop.
**CN**: 第147-182行：基类只保存 warp 级共享内存迭代器，并用共享内存引用和 lane 索引完成初始化。需要注意的是，这里还没有使用 thread_idx 和 warp_idx：具体由派生类负责把 warp 映射到各自的 tile 偏移，并实现真正的主循环。
---
## Key Concepts / 关键概念
- CustomMmaBase does not perform computation by itself; it centralizes storage and iterator wiring shared by multiple execution strategies. / CustomMmaBase 本身不执行计算；它集中管理多种执行策略共享的存储和迭代器布线。
- The shared-memory shapes explicitly encode stage buffering by expanding the K dimension and applying policy-controlled padding. / 共享内存形状通过扩展 K 维并应用 policy 控制的 padding，显式编码了 stage 缓冲机制。
- Derived kernels inherit a consistent view of warp tiling, shared storage, and warp iterators, which keeps higher-level scheduling code smaller. / 派生内核继承了一致的 warp 分块、共享存储和 warp 迭代器视图，从而让更高层的调度代码更精简。
## Dependencies / 依赖项
- `cutlass/gemm/threadblock/mma_base.h` — CUTLASS threadblock MMA base abstractions that this scaffold extends / 本骨架所扩展的 CUTLASS threadblock MMA 基类抽象
- `cutlass/aligned_buffer.h` — Defines aligned shared-memory buffers for operand tiles / 定义操作数 tile 使用的对齐共享内存缓冲区
- `cutlass/gemm/gemm.h` — Provides GEMM shape types and compile-time dimension helpers / 提供 GEMM 形状类型与编译期维度辅助工具
- `cutlass/matrix_shape.h` — Builds the padded shared-memory matrix extents for staged operands / 构建带 padding 的分级操作数共享内存矩阵范围
