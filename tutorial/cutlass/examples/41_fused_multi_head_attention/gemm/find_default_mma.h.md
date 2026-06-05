# find_default_mma.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm/find_default_mma.h`
**Purpose / 用途**: Customized default MMA selector that preserves CUTLASS behavior except for FMHA-specific FastF32 cases needing different stage counts / 定制默认 MMA 选择器，除 FMHA 特定的 FastF32 stage 数场景外保持 CUTLASS 原有行为
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
    \brief Cutlass provides helper template functions to figure out the right
   datastructures to instantiate to run a GEMM with various parameters (see
   `cutlass/gemm/threadblock/default_mma.h`). However, due to template
   instantiation priority rules, it will only create an MmaMultiStage with
   kStages=3 (otherwise creates an MmePipelined - which is not compatible with
   FastF32). kStages=3 uses too much shared memory and we want to use kStages=2,
   so we just copy-pasted some code from `default_mma.h` and
   `default_mma_core.h` files and wrapped this template to allow our usecase.

    This is really only for the FastF32 case - aka using TensorCores with fp32.
*/

#pragma once

#include "cutlass/gemm/threadblock/default_mma.h"
#include "cutlass/gemm/threadblock/default_mma_core_simt.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm70.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm75.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
```
**EN**: Lines 1-52. The file comment explains the motivation very explicitly: CUTLASS template priority selects a three-stage MmaMultistage for the FastF32 path, but that consumes too much shared memory for this use case. The includes pull in the default-mma selection logic and the architecture-specific mma-core building blocks needed to reconstruct a customized choice.
**CN**: 第1-52行：文件注释非常明确地解释了动机：对 FastF32 路径，CUTLASS 的模板优先级会选出三级 MmaMultistage，但这在当前场景下占用过多共享内存。后面的 include 则引入默认 mma 选择逻辑，以及重建自定义选择所需的各架构 mma-core 构件。
```cpp
namespace cutlass {
namespace gemm {
namespace threadblock {

template <
    /// Element type for A matrix operand
    typename ElementA,
    /// Layout type for A matrix operand
    typename LayoutA,
    /// Access granularity of A matrix in units of elements
    int kAlignmentA,
    /// Element type for B matrix operand
    typename ElementB,
    /// Layout type for B matrix operand
    typename LayoutB,
    /// Access granularity of B matrix in units of elements
    int kAlignmentB,
    /// Element type for internal accumulation
    typename ElementAccumulator,
    /// Layout type for C and D matrix operand
    typename LayoutC,
    /// Operator class tag
    typename OperatorClass,
    /// Tag indicating architecture to tune for
    typename ArchTag,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape,
    /// Instruction-level tile size (concept: GemmShape)
    typename InstructionShape,
    /// Number of stages used in the pipelined mainloop
    int Stages,
    /// Operation performed by GEMM
    typename Operator,
    typename Enable_ = void>
struct FindDefaultMma {
  static constexpr bool AccumulatorsInRowMajor = false;
  static constexpr SharedMemoryClearOption SharedMemoryClear =
      SharedMemoryClearOption::kNone;
  using DefaultMma = cutlass::gemm::threadblock::DefaultMma<
      ElementA,
      LayoutA,
      kAlignmentA,
      ElementB,
      LayoutB,
      kAlignmentB,
      ElementAccumulator,
      LayoutC,
      OperatorClass,
      ArchTag,
      ThreadblockShape,
      WarpShape,
      InstructionShape,
      Stages,
      Operator,
      AccumulatorsInRowMajor,
      SharedMemoryClear>;
};
```
**EN**: Lines 53-112. The primary FindDefaultMma template is just a pass-through wrapper. It fixes two auxiliary knobs—AccumulatorsInRowMajor = false and SharedMemoryClear = kNone—and then aliases CUTLASS DefaultMma with all user-supplied operand/layout/shape/operator parameters unchanged.
**CN**: 第53-112行：主模板 FindDefaultMma 只是一个透传包装。它固定了两个辅助参数——AccumulatorsInRowMajor = false 和 SharedMemoryClear = kNone——然后在不改变用户提供的操作数/布局/形状/算子参数的情况下，直接别名到 CUTLASS 的 DefaultMma。
```cpp
/// Specialization for sm80 / FastF32 / multistage with kStages=2
template <
    typename ElementA_,
    /// Layout type for A matrix operand
    typename LayoutA_,
    /// Access granularity of A matrix in units of elements
    int kAlignmentA,
    typename ElementB_,
    /// Layout type for B matrix operand
    typename LayoutB_,
    /// Access granularity of B matrix in units of elements
    int kAlignmentB,
    typename ElementAccumulator,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape,
    /// Instruction-level tile size (concept: GemmShape)
    typename InstructionShape,
    int kStages,
    typename Operator>
struct FindDefaultMma<
    ElementA_,
    LayoutA_,
    kAlignmentA,
    ElementB_,
    LayoutB_,
    kAlignmentB,
    ElementAccumulator,
    layout::RowMajor,
    arch::OpClassTensorOp,
    arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    kStages,
    Operator,
    typename cutlass::platform::enable_if<(kAlignmentA > 1)>::type> {
  using LayoutC = layout::RowMajor;
  using OperatorClass = arch::OpClassTensorOp;
  using ArchTag = arch::Sm80;

  using DefaultMma_ = cutlass::gemm::threadblock::DefaultMma<
      ElementA_,
      LayoutA_,
      kAlignmentA,
      ElementB_,
      LayoutB_,
      kAlignmentB,
      ElementAccumulator,
      LayoutC,
      OperatorClass,
      ArchTag,
      ThreadblockShape,
      WarpShape,
      InstructionShape,
      3,
      Operator>;
  struct DefaultMma : DefaultMma_ {
    using MmaCore_ = typename DefaultMma_::MmaCore;
    // Define the threadblock-scoped multistage matrix multiply
    using ThreadblockMma = cutlass::gemm::threadblock::MmaMultistage<
        typename MmaCore_::Shape,
        typename DefaultMma_::IteratorA,
        typename MmaCore_::SmemIteratorA,
        MmaCore_::kCacheOpA,
        typename DefaultMma_::IteratorB,
        typename MmaCore_::SmemIteratorB,
        MmaCore_::kCacheOpB,
        ElementAccumulator,
        LayoutC,
        typename MmaCore_::MmaPolicy,
        kStages>;
  };
};

} // namespace threadblock
} // namespace gemm
} // namespace cutlass
```
**EN**: Lines 113-191. The specialization targets exactly the problematic path: row-major output, TensorOp class, SM80 architecture, and aligned A accesses (kAlignmentA > 1). It first asks CUTLASS for the usual default configuration but forces that query to use 3 stages so it can reuse the standard MmaCore, iterators, and policy. It then derives a nested DefaultMma type whose ThreadblockMma is replaced with MmaMultistage parameterized by the caller-provided kStages, effectively preserving CUTLASS default selection while overriding only the threadblock mainloop depth.
**CN**: 第113-191行：这个特化精确命中了有问题的路径：RowMajor 输出、TensorOp 类、SM80 架构，以及对齐的 A 访问（kAlignmentA > 1）。它先向 CUTLASS 请求一个“正常”的默认配置，但强制查询使用 3 stages，以便复用标准的 MmaCore、迭代器和 policy。然后它派生出一个嵌套的 DefaultMma 类型，把其中的 ThreadblockMma 替换为使用调用方 kStages 参数的 MmaMultistage，从而在保留 CUTLASS 默认选择结果的同时，仅覆写 threadblock 主循环深度。
---
## Key Concepts / 关键概念
- Most configurations are untouched; only one architecture/operator/layout/alignment corner case is overridden. / 绝大多数配置都保持不变；只有一个架构/算子/布局/对齐条件交汇的特殊分支被覆写。
- The specialization reuses CUTLASS DefaultMma machinery as much as possible and swaps only the ThreadblockMma type. / 该特化尽可能复用 CUTLASS DefaultMma 的既有机制，只替换了 ThreadblockMma 类型。
- This file is the bridge between architecture-specific default selection and the custom multistage pipeline needed by the fused attention kernel. / 这个文件把架构相关的默认选择逻辑，与融合注意力内核所需的自定义 multistage 流水连接了起来。
## Dependencies / 依赖项
- `cutlass/gemm/threadblock/default_mma.h` — Provides CUTLASS baseline threadblock MMA selection logic / 提供 CUTLASS 基线 threadblock MMA 选择逻辑
- `cutlass/gemm/threadblock/default_mma_core_sm80.h` — Supplies SM80 tensor-core MMA core types needed for the FastF32 override / 提供 FastF32 覆写所需的 SM80 Tensor Core MMA Core 类型
- `cutlass/gemm/threadblock/default_mma_core_sm75.h` — Keeps earlier Tensor Core architectures on the standard CUTLASS path / 让较早的 Tensor Core 架构继续沿用 CUTLASS 标准路径
- `cutlass/gemm/threadblock/default_mma_core_simt.h` — Retains SIMT fallback support when tensor-core specializations do not apply / 在 Tensor Core 特化不适用时保留 SIMT 回退支持
