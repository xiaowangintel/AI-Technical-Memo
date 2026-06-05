# default_fmha_grouped.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/default_fmha_grouped.h`
**Purpose / 用途**: Compile-time assembly of the grouped FMHA forward kernel. It turns architecture, datatype, alignment, and tile-shape choices into two cooperating GEMM building blocks: MM0 for `Q @ K^T` and MM1 for `Attn @ V`. / 该文件负责在编译期组装 grouped FMHA 前向内核，把架构、数据类型、对齐方式和 tile 形状等选择拼成两个协作的 GEMM 构件：MM0 用于 `Q @ K^T`，MM1 用于 `Attn @ V`。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-64 / 第 1-64 行
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
    \brief
      Default kernel-level GEMM definitions combine threadblock-scoped matrix multiply-add with
      the appropriate threadblock-scoped epilogue.

      Note, CUTLASS epilogues universally target row-major outputs. Column-major outputs are
      accommodated by exchanging A and B operands and assuming transposed layouts. Partial
      specializations here choose 'device::GemmTransposed' to implement this functionality.

*/

#pragma once

#include "cutlass/cutlass.h"

#include "cutlass/complex.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/numeric_types.h"

#include "fmha_grouped.h"
#include "gemm_kernel_utils.h"
#include "gemm/custom_mma.h"
#include "gemm/find_default_mma.h"
#include "gemm/mma_from_smem.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace gemm {
namespace kernel {

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: License, includes, and namespace setup for FMHA kernel construction.
**CN**: 许可证、头文件依赖以及 FMHA 内核组装所需的命名空间设置。

### Lines 65-93 / 第 65-93 行
```cpp
template <
    // The datatype of Q/K/V
    typename scalar_t_,
    // Architecture we are targeting (eg `cutlass::arch::Sm80`)
    typename ArchTag_,
    // If Q/K/V are correctly aligned in memory and we can run a fast kernel
    bool isAligned_,
    int kQueriesPerBlock,
    int kKeysPerBlock,
    int kMaxK = (int)cutlass::platform::numeric_limits<uint32_t>::max(),
    GroupScheduleMode GroupScheduleMode_ = GroupScheduleMode::kDeviceOnly
    >
struct DefaultFMHAGrouped {
  using scalar_t = scalar_t_;
  using accum_t = float;
  using output_t = scalar_t;

  // Accumulator between 2 iterations
  // Using `accum_t` improves perf on f16 at the cost of
  // numerical errors
  using output_accum_t = accum_t;

  using ArchTag = ArchTag_;
  static bool const kIsAligned = isAligned_;
  static bool const kSingleValueIteration = kMaxK <= kKeysPerBlock;
  static constexpr bool kIsHalf = cutlass::sizeof_bits<scalar_t>::value == 16;
  static int const kWarpSize = 32;
  static int const kNumWarpsPerBlock = kQueriesPerBlock * kKeysPerBlock / (kWarpSize * kWarpSize);
```
**EN**: Template parameters and top-level traits: datatype, architecture tag, alignment mode, tile sizes, and grouped schedule mode.
**CN**: 模板参数与顶层 traits：数据类型、架构标签、对齐模式、tile 大小以及 grouped 调度模式。

### Lines 94-181 / 第 94-181 行
```cpp
  struct MM0 {
    /*
      In this first matmul, we compute a block of `Q @ K.T`.
      While the calculation result is still hot in registers, we update
      `mi`, `m_prime`, `s_prime` in shared-memory, and then store this value
      into a shared-memory ("AccumulatorSharedStorage") that is used later as
      operand A for the second matmul (see MM1)
    */

    using GemmType = gemm_kernel_utils::DefaultGemmType<ArchTag, scalar_t>;
    using OpClass = typename GemmType::OpClass;

    using ElementA = scalar_t;
    using ElementB = scalar_t;
    using ElementC = scalar_t;
    using ElementAccumulator = accum_t;

    using LayoutA = cutlass::layout::RowMajor;
    using LayoutB = cutlass::layout::ColumnMajor;
    using LayoutC = cutlass::layout::RowMajor;

    using DefaultConfig =
        typename cutlass::gemm::device::DefaultGemmConfiguration<
            OpClass,
            ArchTag,
            ElementA,
            ElementB,
            ElementC,
            ElementAccumulator
            >;

    static int const kAlignmentA =
        kIsAligned ? DefaultConfig::kAlignmentA : GemmType::kMinimumAlignment;
    static int const kAlignmentB =
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment;

    using ThreadblockShape = cutlass::gemm::GemmShape<kQueriesPerBlock, kKeysPerBlock, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using InstructionShape = typename GemmType::InstructionShape;

    static int const kStages = DefaultConfig::kStages;
    using Operator = typename GemmType::Operator;

    using DefaultMma = typename cutlass::gemm::threadblock::FindDefaultMma<
        ElementA,
        LayoutA,
        kAlignmentA,
        ElementB,
        LayoutB,
        kAlignmentB,
        ElementAccumulator,
        LayoutC,
        OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        InstructionShape,
        ArchTag::kMinComputeCapability >= 80 && kIsHalf
            ? 4
            : DefaultConfig::kStages,
        Operator
        >::DefaultMma;

    using MmaCore = typename DefaultMma::MmaCore;
    using IteratorA = typename DefaultMma::IteratorA;
    using IteratorB = typename DefaultMma::IteratorB;
    using DefaultThreadblockMma = typename DefaultMma::ThreadblockMma;
    using Mma = typename cutlass::platform::conditional<
        kSingleValueIteration,
        typename MakeCustomMma<DefaultThreadblockMma, kMaxK>::Mma,
        DefaultThreadblockMma>::type;
    using AccumLambdaIterator = typename DefaultMmaAccumLambdaIterator<
        typename Mma::Operator::IteratorC,
        ElementAccumulator,
        kWarpSize>::Iterator;

    static_assert(MmaCore::WarpCount::kCount == kNumWarpsPerBlock, "");

    // Epilogue to store to shared-memory in a format that we can use later for
    // the second matmul
    using B2bGemm = typename cutlass::gemm::threadblock::B2bGemm<
        typename Mma::Operator::IteratorC,
        typename Mma::Operator,
        scalar_t,
        WarpShape,
        ThreadblockShape>;
    using AccumulatorSharedStorage = typename B2bGemm::AccumulatorSharedStorage;
  };
```
**EN**: `MM0` definition for `Q @ K^T`, including default GEMM dispatch, alignment rules, threadblock/warp/instruction shapes, and shared-memory accumulator storage.
**CN**: `MM0` 定义：负责 `Q @ K^T`，包含默认 GEMM 派发、对齐规则、threadblock/warp/instruction 形状，以及累加器共享内存存储。

### Lines 182-278 / 第 182-278 行
```cpp

  struct MM1 {
    /*
      Second matmul: perform `attn @ V` where `attn` is the attention (not
      normalized) and stored in shared memory
    */

    using GemmType = typename MM0::GemmType;
    using OpClass = typename GemmType::OpClass;

    using ElementA = scalar_t;
    using ElementB = scalar_t;
    using ElementC = output_accum_t;
    using ElementAccumulator = accum_t;

    using LayoutA = cutlass::layout::RowMajor;
    using LayoutB = cutlass::layout::RowMajor;
    using LayoutC = cutlass::layout::RowMajor;

    using DefaultConfig =
        typename cutlass::gemm::device::DefaultGemmConfiguration<
            OpClass,
            ArchTag,
            ElementA,
            ElementB,
            ElementC,
            ElementAccumulator
            >;

    static int const kAlignmentA = DefaultConfig::kAlignmentA;
    static int const kAlignmentB =
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment;

    using ThreadblockShape = typename MM0::ThreadblockShape;
    using WarpShape = typename MM0::WarpShape;
    using InstructionShape = typename MM0::InstructionShape;

    using EpilogueOutputOp = typename DefaultConfig::EpilogueOutputOp;

    static int const kStages = DefaultConfig::kStages;
    using Operator = typename GemmType::Operator;

    using ThreadblockSwizzle = void; // Swizzling is unused
    static bool const kSplitKSerial = false;

    using DefaultGemm = cutlass::gemm::kernel::DefaultGemm<
        ElementA,
        LayoutA,
        kAlignmentA,
        ElementB,
        LayoutB,
        kAlignmentB,
        ElementC,
        LayoutC,
        ElementAccumulator,
        OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        InstructionShape,
        EpilogueOutputOp,
        ThreadblockSwizzle,
        ArchTag::kMinComputeCapability >= 80 && kIsHalf
            ? 4
            : DefaultConfig::kStages,
        kSplitKSerial,
        Operator>;

    using WarpIteratorA = typename cutlass::gemm::threadblock::
    DefaultWarpIteratorAFromSharedMemory<
        typename DefaultGemm::Mma::Policy::Operator::Shape, // WarpShape
        typename DefaultGemm::Mma::Policy::Operator::InstructionShape,
        typename DefaultGemm::Mma::Policy::Operator::IteratorA,
        typename DefaultGemm::Mma::Policy>::WarpIterator;

    using DefaultMmaFromSmem =
        typename cutlass::gemm::threadblock::DefaultMmaFromSharedMemory<
            typename DefaultGemm::Mma,
            MM0::AccumulatorSharedStorage::Shape::kN,  // kMaxK
            WarpIteratorA,
            false>; // kScaleOperandA

    using Mma = typename DefaultMmaFromSmem::Mma;
    using IteratorB = typename Mma::IteratorB;
    using WarpCount = typename Mma::WarpCount;
    static_assert(WarpCount::kCount == kNumWarpsPerBlock, "");

    using DefaultEpilogue = typename DefaultGemm::Epilogue;
    using OutputTileIterator =
        typename cutlass::epilogue::threadblock::PredicatedTileIterator<
            typename DefaultEpilogue::OutputTileIterator::ThreadMap,
            output_t>;
    using OutputTileIteratorAccum =
        typename cutlass::epilogue::threadblock::PredicatedTileIterator<
            typename DefaultEpilogue::OutputTileIterator::ThreadMap,
            output_accum_t>;
  };
```
**EN**: `MM1` definition for `Attn @ V`, including shared-memory operand-A loading, second GEMM dispatch, and output iterators.
**CN**: `MM1` 定义：负责 `Attn @ V`，包含从共享内存读取操作数 A、第二段 GEMM 派发以及输出迭代器。

### Lines 279-290 / 第 279-290 行
```cpp

/// Define the kernel in terms of the default kernel
  using FMHAKernel = kernel::FMHAGrouped<
    MM0,
    MM1,
    scalar_t,
    accum_t,
    output_t,
    output_accum_t,
    kSingleValueIteration,
    GroupScheduleMode_
  >;
```
**EN**: Final `FMHAKernel` alias wiring both GEMM stages into the grouped runtime kernel.
**CN**: 最终的 `FMHAKernel` 别名，把两段 GEMM 组装为 grouped 运行时内核。

### Lines 291-299 / 第 291-299 行
```cpp
};

/////////////////////////////////////////////////////////////////////////////////////////////////

}  // namespace kernel
}  // namespace gemm
}  // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Namespace closure.
**CN**: 命名空间收尾。

---
## Key Concepts / 关键概念
- **EN:** The file expresses FMHA as a two-stage back-to-back GEMM pipeline: score generation first, value aggregation second.
  **CN:** 该文件把 FMHA 表达为“两段背靠背 GEMM”流水：先生成分数，再做 value 聚合。
- **EN:** `MM0::B2bGemm::AccumulatorSharedStorage` is the hand-off buffer between stages. The score tile stays in shared memory instead of being written to global memory.
  **CN:** `MM0::B2bGemm::AccumulatorSharedStorage` 是两段之间的交接缓冲区，分数 tile 保留在共享内存中，而不是写回全局内存。
- **EN:** `FindDefaultMma`, `DefaultGemmConfiguration`, and `DefaultMmaFromSharedMemory` show standard CUTLASS kernel composition: choose the MMA core, then derive iterators, stages, and epilogues from it.
  **CN:** `FindDefaultMma`、`DefaultGemmConfiguration` 与 `DefaultMmaFromSharedMemory` 展示了典型的 CUTLASS 内核拼装方式：先选择 MMA 核心，再从中推导 iterator、stage 与 epilogue。
- **EN:** Alignment and architecture tags decide whether the kernel can use faster tensor-core paths or must fall back to more conservative access patterns.
  **CN:** 对齐状态与架构标签决定内核能否使用更快的 Tensor Core 路径，或退回到更保守的访存方式。
- **EN:** `default_fmha_grouped.h` does not execute FMHA itself; it provides the concrete MM0/MM1 types consumed by `fmha_grouped.h`.
  **CN:** `default_fmha_grouped.h` 本身不执行 FMHA，而是为 `fmha_grouped.h` 提供具体的 MM0 / MM1 类型。
- **EN:** The epilogue types chosen here determine how online softmax state and output rescaling are later attached to MM1 writeback.
  **CN:** 这里确定的 epilogue 相关类型，会直接影响后续 MM1 写回阶段如何挂接在线 softmax 状态与输出重标定。
## Dependencies / 依赖项
- `"cutlass/cutlass.h"` — CUTLASS core definitions and portability macros / CUTLASS 核心定义与可移植性宏
- `"cutlass/complex.h"` — CUTLASS complex-number helpers and related numeric traits / CUTLASS 复数辅助类型与相关数值 traits
- `"cutlass/layout/matrix.h"` — matrix layout tags and coordinate helpers / 矩阵布局标签与坐标辅助类型
- `"cutlass/numeric_types.h"` — CUTLASS scalar and numeric utility types / CUTLASS 标量与数值工具类型
- `"fmha_grouped.h"` — the grouped FMHA runtime kernel assembled by this configuration layer / 由该配置层拼装的 grouped FMHA 运行时内核
- `"gemm_kernel_utils.h"` — FMHA GEMM dispatch, validation, and compile-time utility helpers / FMHA GEMM 派发、校验与编译期工具辅助
- `"gemm/custom_mma.h"` — custom MMA path for bounded-K FMHA specializations / 面向有界 K 特化的自定义 MMA 路径
- `"gemm/find_default_mma.h"` — helper for selecting the default CUTLASS threadblock MMA / 用于选择默认 CUTLASS threadblock MMA 的辅助模板
- `"gemm/mma_from_smem.h"` — adapters that rebuild MMA pipelines with operand A from shared memory / 把操作数 A 改为来自共享内存的 MMA 适配层
