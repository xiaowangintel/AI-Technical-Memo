# default_rank_2k.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/default_rank_2k.h`
- **Purpose / 用途 (EN):** Defines the default kernel composition and specialization rules for rank-2k. Briefly, the file comment says: Default kernel-level Rank2K definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue.
- **Purpose / 用途 (CN):** 定义 rank-2k 的默认内核组合与模板特化规则。 文件注释的简要说明是：Default kernel-level Rank2K definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue。
- **Line count / 行数:** 285

## Line-by-Line Analysis / 逐行分析
> The source is grouped into contiguous significant line ranges for readability. / 为了便于阅读，下面按连续的重要代码区间进行分析。

### Lines 1-30

```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3 |  * SPDX-License-Identifier: BSD-3-Clause
   4 |  *
   5 |  * Redistribution and use in source and binary forms, with or without
   6 |  * modification, are permitted provided that the following conditions are met:
   7 |  *
   8 |  * 1. Redistributions of source code must retain the above copyright notice, this
   9 |  * list of conditions and the following disclaimer.
  10 |  *
  11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12 |  * this list of conditions and the following disclaimer in the documentation
  13 |  * and/or other materials provided with the distribution.
  14 |  *
  15 |  * 3. Neither the name of the copyright holder nor the names of its
  16 |  * contributors may be used to endorse or promote products derived from
  17 |  * this software without specific prior written permission.
  18 |  *
  19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29 |  *
  30 |  **************************************************************************************************/
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 32-35

```cpp
  32 | /*! \file
  33 |     \brief 
  34 |       Default kernel-level Rank2K definitions combine threadblock-scoped matrix multiply-add with
  35 |       the appropriate threadblock-scoped epilogue.
```
**EN:** This file-level comment names the header and introduces the high-level role of the kernel component that follows.
**CN:** 这一段文件级注释给出了头文件说明，并概括了后续内核组件的整体作用。

### Lines 40-40

```cpp
  40 | #pragma once
```
**EN:** This directive makes the header idempotent so repeated inclusion does not create duplicate definitions.
**CN:** 该指令让头文件只生效一次，避免重复包含导致的重复定义。

### Lines 42-42

```cpp
  42 | #include "cutlass/blas3.h"
```
**EN:** This include block imports `cutlass/blas3.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/blas3.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 44-45

```cpp
  44 | #include "cutlass/layout/matrix.h"
  45 | #include "cutlass/arch/wmma.h"
```
**EN:** This include block imports `cutlass/layout/matrix.h`, `cutlass/arch/wmma.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/layout/matrix.h`, `cutlass/arch/wmma.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 47-48

```cpp
  47 | #include "cutlass/epilogue/threadblock/epilogue.h"
  48 | #include "cutlass/epilogue/thread/linear_combination.h"
```
**EN:** This include block imports `cutlass/epilogue/threadblock/epilogue.h`, `cutlass/epilogue/thread/linear_combination.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/epilogue/threadblock/epilogue.h`, `cutlass/epilogue/thread/linear_combination.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 50-57

```cpp
  50 | #include "cutlass/gemm/gemm.h"
  51 | #include "cutlass/gemm/kernel/rank_2k_universal.h"
  52 | #include "cutlass/gemm/threadblock/default_mma_core_sm75.h"
  53 | #include "cutlass/gemm/threadblock/default_mma_core_sm70.h"
  54 | #include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
  55 | #include "cutlass/gemm/threadblock/default_mma.h"
  56 | #include "cutlass/gemm/threadblock/default_mma_core_simt.h"
  57 | #include "cutlass/gemm/threadblock/threadblock_swizzle.h"
```
**EN:** This include block imports `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/rank_2k_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm75.h`, `cutlass/gemm/threadblock/default_mma_core_sm70.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, ... (+2 more), providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/rank_2k_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm75.h`, `cutlass/gemm/threadblock/default_mma_core_sm70.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, ... (+2 more)，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 59-62

```cpp
  59 | #include "cutlass/epilogue/threadblock/default_epilogue_tensor_op_blas3.h"
  60 | #include "cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h"
  61 | #include "cutlass/epilogue/threadblock/default_epilogue_simt.h"
  62 | #include "cutlass/transform/threadblock/predicated_tile_iterator.h"
```
**EN:** This include block imports `cutlass/epilogue/threadblock/default_epilogue_tensor_op_blas3.h`, `cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`, `cutlass/epilogue/threadblock/default_epilogue_simt.h`, `cutlass/transform/threadblock/predicated_tile_iterator.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/epilogue/threadblock/default_epilogue_tensor_op_blas3.h`, `cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`, `cutlass/epilogue/threadblock/default_epilogue_simt.h`, `cutlass/transform/threadblock/predicated_tile_iterator.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 64-66

```cpp
  64 | #if defined(CUTLASS_ARCH_WMMA_ENABLED)
  65 | #include "cutlass/epilogue/threadblock/default_epilogue_wmma_tensor_op.h"
  66 | #endif //CUTLASS_ARCH_WMMA_ENABLED
```
**EN:** This conditional-compilation block enables the following code only when the required architecture or backend feature is available.
**CN:** 这一段条件编译逻辑只在目标架构或后端特性可用时启用后续代码。

### Lines 71-73

```cpp
  71 | namespace cutlass {
  72 | namespace gemm {
  73 | namespace kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 77-106

```cpp
  77 | template <
  78 |     /// Element type for A matrix operand
  79 |     typename ElementA_,
  80 |     /// Layout type for A matrix operand
  81 |     typename LayoutA_,
  82 |     /// Access granularity of A matrix in units of elements
  83 |     int kAlignmentA,
  84 |     /// Element type for B matrix operand
  85 |     typename ElementB_,
  86 |     /// Layout type for B matrix operand
  87 |     typename LayoutB_,
  88 |     /// Access granularity of B matrix in units of elements
  89 |     int kAlignmentB,
  90 |     /// Element type for C and D matrix operands
  91 |     typename ElementC_,
  92 |     /// Layout type for C and D matrix operands
  93 |     typename LayoutC_,
  94 |     /// Fill Mode for C (kLower or kUpper)
  95 |     FillMode FillModeC_,
  96 |     /// Element type for internal accumulation
  97 |     typename ElementAccumulator,
  98 |     /// Operator class tag
  99 |     typename OperatorClass,
 100 |     /// Tag indicating architecture to tune for
 101 |     typename ArchTag,
 102 |     /// Threadblock-level tile size (concept: GemmShape)
 103 |     typename ThreadblockShape,
 104 |     /// Warp-level tile size (concept: GemmShape)
 105 |     typename WarpShape,
 106 |     /// Warp-level tile size (concept: GemmShape)
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 107-121

```cpp
 107 |     typename InstructionShape,
 108 |     /// Epilogue output operator
 109 |     typename EpilogueOutputOp,
 110 |     /// Threadblock-level swizzling operator
 111 |     typename ThreadblockSwizzle,
 112 |     /// Number of stages used in the pipelined mainloop
 113 |     int Stages,
 114 |     /// If true, kernel is configured to support serial reduction in the
 115 |     /// epilogue
 116 |     bool SplitKSerial,
 117 |     /// Operation performed by GEMM
 118 |     typename Operator,
 119 |     /// Blas3 computation mode
 120 |     BlasMode BlasMode_ = BlasMode::kSymmetric>
 121 | struct DefaultRank2K;
```
**EN:** This block declares or specializes `DefaultRank2K`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2K`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 125-154

```cpp
 125 | /// Partial specialization for Hopper Architecture
 126 | template <
 127 |     /// Element type for A matrix operand
 128 |     typename ElementA,
 129 |     /// Layout type for A matrix operand
 130 |     typename LayoutA,
 131 |     /// Access granularity of A matrix in units of elements
 132 |     int kAlignmentA,
 133 |     /// Element type for B matrix operand
 134 |     typename ElementB,
 135 |     /// Layout type for B matrix operand
 136 |     typename LayoutB,
 137 |     /// Access granularity of A matrix in units of elements
 138 |     int kAlignmentB,
 139 |     /// Element type for C and D matrix operands
 140 |     typename ElementC,
 141 |     /// Fill Mode for C (kLower or kUpper)
 142 |     FillMode FillModeC,
 143 |     /// Element type for internal accumulation
 144 |     typename ElementAccumulator,
 145 |     /// Threadblock-level tile size (concept: GemmShape)
 146 |     typename ThreadblockShape,
 147 |     /// Warp-level tile size (concept: GemmShape)
 148 |     typename WarpShape,
 149 |     /// Warp-level tile size (concept: GemmShape)
 150 |     typename InstructionShape,
 151 |     /// Epilogue output operator
 152 |     typename EpilogueOutputOp,
 153 |     /// Threadblock-level swizzling operator
 154 |     typename ThreadblockSwizzle,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 155-178

```cpp
 155 |     /// Number of stages used in the pipelined mainloop
 156 |     int Stages,
 157 |     /// If true, kernel is configured to support serial reduction in the
 158 |     /// epilogue
 159 |     bool SplitKSerial,
 160 |     /// Operation performed by GEMM
 161 |     typename Operator>
 162 | struct DefaultRank2K<
 163 |                     ElementA, LayoutA, kAlignmentA, 
 164 |                     ElementB, LayoutB, kAlignmentB, 
 165 |                     ElementC,layout::RowMajor, FillModeC, 
 166 |                     ElementAccumulator, arch::OpClassTensorOp, arch::Sm90, 
 167 |                     ThreadblockShape, WarpShape, InstructionShape,
 168 |                     EpilogueOutputOp, ThreadblockSwizzle, Stages, SplitKSerial,
 169 |                     Operator> {
 170 |   /// Define the threadblock-scoped matrix multiply-accumulate (A x BT)
 171 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMma<
 172 |       ElementA, LayoutA, 
 173 |       kAlignmentA, 
 174 |       ElementB, typename layout::LayoutTranspose<LayoutB>::type, 
 175 |       kAlignmentB,
 176 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm90,
 177 |       ThreadblockShape, WarpShape, InstructionShape, Stages,
 178 |       Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 180-188

```cpp
 180 |   /// Define the threadblock-scoped matrix multiply-accumulate (B x AT)
 181 |   using Mma2 = typename cutlass::gemm::threadblock::DefaultMma<
 182 |       ElementB, LayoutB, 
 183 |       kAlignmentB, 
 184 |       ElementA, typename layout::LayoutTranspose<LayoutA>::type, 
 185 |       kAlignmentA,
 186 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm90,
 187 |       ThreadblockShape, WarpShape, InstructionShape, Stages,
 188 |       Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 190-190

```cpp
 190 |   static const int kPartitionsK = ThreadblockShape::kK / WarpShape::kK;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 192-196

```cpp
 192 |   /// Define the epilogue
 193 |   using Epilogue =
 194 |       typename cutlass::epilogue::threadblock::DefaultEpilogueTensorOpBlas3<
 195 |           ThreadblockShape, typename Mma1::Operator, kPartitionsK, EpilogueOutputOp,
 196 |           EpilogueOutputOp::kCount, BlasMode::kSymmetric>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 198-200

```cpp
 198 |   /// Define the kernel-level Rank2K operator.
 199 |   using Rank2Kkernel = kernel::Rank2KUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, FillModeC, BlasMode::kSymmetric>;
 200 | };
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 204-233

```cpp
 204 | /// Partial specialization for Ampere Architecture
 205 | template <
 206 |     /// Element type for A matrix operand
 207 |     typename ElementA,
 208 |     /// Layout type for A matrix operand
 209 |     typename LayoutA,
 210 |     /// Access granularity of A matrix in units of elements
 211 |     int kAlignmentA,
 212 |     /// Element type for B matrix operand
 213 |     typename ElementB,
 214 |     /// Layout type for B matrix operand
 215 |     typename LayoutB,
 216 |     /// Access granularity of A matrix in units of elements
 217 |     int kAlignmentB,
 218 |     /// Element type for C and D matrix operands
 219 |     typename ElementC,
 220 |     /// Fill Mode for C (kLower or kUpper)
 221 |     FillMode FillModeC,
 222 |     /// Element type for internal accumulation
 223 |     typename ElementAccumulator,
 224 |     /// Threadblock-level tile size (concept: GemmShape)
 225 |     typename ThreadblockShape,
 226 |     /// Warp-level tile size (concept: GemmShape)
 227 |     typename WarpShape,
 228 |     /// Warp-level tile size (concept: GemmShape)
 229 |     typename InstructionShape,
 230 |     /// Epilogue output operator
 231 |     typename EpilogueOutputOp,
 232 |     /// Threadblock-level swizzling operator
 233 |     typename ThreadblockSwizzle,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 234-257

```cpp
 234 |     /// Number of stages used in the pipelined mainloop
 235 |     int Stages,
 236 |     /// If true, kernel is configured to support serial reduction in the
 237 |     /// epilogue
 238 |     bool SplitKSerial,
 239 |     /// Operation performed by GEMM
 240 |     typename Operator>
 241 | struct DefaultRank2K<
 242 |                     ElementA, LayoutA, kAlignmentA, 
 243 |                     ElementB, LayoutB, kAlignmentB, 
 244 |                     ElementC,layout::RowMajor, FillModeC, 
 245 |                     ElementAccumulator, arch::OpClassTensorOp, arch::Sm80, 
 246 |                     ThreadblockShape, WarpShape, InstructionShape,
 247 |                     EpilogueOutputOp, ThreadblockSwizzle, Stages, SplitKSerial,
 248 |                     Operator> {
 249 |   /// Define the threadblock-scoped matrix multiply-accumulate (A x BT)
 250 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMma<
 251 |       ElementA, LayoutA, 
 252 |       kAlignmentA, 
 253 |       ElementB, typename layout::LayoutTranspose<LayoutB>::type, 
 254 |       kAlignmentB,
 255 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80,
 256 |       ThreadblockShape, WarpShape, InstructionShape, Stages,
 257 |       Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 259-267

```cpp
 259 |   /// Define the threadblock-scoped matrix multiply-accumulate (B x AT)
 260 |   using Mma2 = typename cutlass::gemm::threadblock::DefaultMma<
 261 |       ElementB, LayoutB, 
 262 |       kAlignmentB, 
 263 |       ElementA, typename layout::LayoutTranspose<LayoutA>::type, 
 264 |       kAlignmentA,
 265 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80,
 266 |       ThreadblockShape, WarpShape, InstructionShape, Stages,
 267 |       Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 269-269

```cpp
 269 |   static const int kPartitionsK = ThreadblockShape::kK / WarpShape::kK;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 271-275

```cpp
 271 |   /// Define the epilogue
 272 |   using Epilogue =
 273 |       typename cutlass::epilogue::threadblock::DefaultEpilogueTensorOpBlas3<
 274 |           ThreadblockShape, typename Mma1::Operator, kPartitionsK, EpilogueOutputOp,
 275 |           EpilogueOutputOp::kCount, BlasMode::kSymmetric>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 277-280

```cpp
 277 |   /// Define the kernel-level Rank2K operator.
 278 |   using Rank2Kkernel = kernel::Rank2KUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, FillModeC, BlasMode::kSymmetric>;
 279 | };
 280 | ////////////////////////////////////////////////////////////////////////////////
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 283-285

```cpp
 283 | }  // namespace kernel
 284 | }  // namespace gemm
 285 | }  // namespace cutlass
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

## Key Concepts / 关键概念
- Default kernel composition / 默认内核组合
- Template metaprogramming / 模板元编程
- Rank-2k update / Rank-2k 更新
- Symmetric matrix multiply / 对称矩阵乘
- SM90 architecture tuning / SM90 架构调优

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/blas3.h`, `cutlass/layout/matrix.h`, `cutlass/arch/wmma.h`, `cutlass/epilogue/threadblock/epilogue.h`, `cutlass/epilogue/thread/linear_combination.h`, `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/rank_2k_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm75.h`, `cutlass/gemm/threadblock/default_mma_core_sm70.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_mma_core_simt.h`, ... (+6 more)
- **Subsystems / 子系统:** Epilogue/output pipeline / Epilogue 输出流水, Threadblock MMA building blocks / 线程块级 MMA 构件
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/rank_2k_universal.h`
