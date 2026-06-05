# default_rank_2k_complex.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/default_rank_2k_complex.h`
- **Purpose / 用途 (EN):** Defines the default kernel composition and specialization rules for rank-2k complex. Briefly, the file comment says: Default kernel-level Rank2K definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue.
- **Purpose / 用途 (CN):** 定义 rank-2k complex 的默认内核组合与模板特化规则。 文件注释的简要说明是：Default kernel-level Rank2K definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue。
- **Line count / 行数:** 498

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

### Lines 50-55

```cpp
  50 | #include "cutlass/gemm/gemm.h"
  51 | #include "cutlass/gemm/kernel/rank_2k_universal.h"
  52 | #include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
  53 | #include "cutlass/gemm/threadblock/default_mma.h"
  54 | #include "cutlass/gemm/threadblock/default_multistage_mma_complex.h"
  55 | #include "cutlass/gemm/threadblock/threadblock_swizzle.h"
```
**EN:** This include block imports `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/rank_2k_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_multistage_mma_complex.h`, `cutlass/gemm/threadblock/threadblock_swizzle.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/rank_2k_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_multistage_mma_complex.h`, `cutlass/gemm/threadblock/threadblock_swizzle.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 57-58

```cpp
  57 | #include "cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op_blas3.h"
  58 | #include "cutlass/transform/threadblock/predicated_tile_iterator.h"
```
**EN:** This include block imports `cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op_blas3.h`, `cutlass/transform/threadblock/predicated_tile_iterator.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op_blas3.h`, `cutlass/transform/threadblock/predicated_tile_iterator.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 60-62

```cpp
  60 | #if defined(CUTLASS_ARCH_WMMA_ENABLED)
  61 | #include "cutlass/epilogue/threadblock/default_epilogue_wmma_tensor_op.h"
  62 | #endif //CUTLASS_ARCH_WMMA_ENABLED
```
**EN:** This conditional-compilation block enables the following code only when the required architecture or backend feature is available.
**CN:** 这一段条件编译逻辑只在目标架构或后端特性可用时启用后续代码。

### Lines 67-69

```cpp
  67 | namespace cutlass {
  68 | namespace gemm {
  69 | namespace kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 73-102

```cpp
  73 | template <
  74 |     /// Element type for A matrix operand
  75 |     typename ElementA_,
  76 |     /// Layout type for A matrix operand
  77 |     typename LayoutA_,
  78 |     /// Element type for B matrix operand
  79 |     typename ElementB_,
  80 |     /// Layout type for B matrix operand
  81 |     typename LayoutB_,
  82 |     /// Element type for C and D matrix operands
  83 |     typename ElementC_,
  84 |     /// Layout type for C and D matrix operands
  85 |     typename LayoutC_,
  86 |     /// Fill Mode for C (kLower or kUpper)
  87 |     FillMode FillModeC_,
  88 |     /// Element type for internal accumulation
  89 |     typename ElementAccumulator,
  90 |     /// Operator class tag
  91 |     typename OperatorClass,
  92 |     /// Tag indicating architecture to tune for
  93 |     typename ArchTag,
  94 |     /// Threadblock-level tile size (concept: GemmShape)
  95 |     typename ThreadblockShape,
  96 |     /// Warp-level tile size (concept: GemmShape)
  97 |     typename WarpShape,
  98 |     /// Warp-level tile size (concept: GemmShape)
  99 |     typename InstructionShape,
 100 |     /// Epilogue output operator
 101 |     typename EpilogueOutputOp,
 102 |     /// Threadblock-level swizzling operator
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 103-117

```cpp
 103 |     typename ThreadblockSwizzle,
 104 |     /// Number of stages used in the pipelined mainloop
 105 |     int Stages,
 106 |     /// Complex elementwise transformation on A operand
 107 |     ComplexTransform TransformA,
 108 |     /// Complex elementwise transformation on B operand
 109 |     ComplexTransform TransformB,
 110 |     /// Operation performed by GEMM
 111 |     typename Operator,
 112 |     /// If true, kernel is configured to support serial reduction in the
 113 |     /// epilogue
 114 |     bool SplitKSerial,
 115 |     /// Blas3 computation mode
 116 |     BlasMode BlasMode_ = BlasMode::kSymmetric>
 117 | struct DefaultRank2KComplex;
```
**EN:** This block declares or specializes `DefaultRank2KComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 120-121

```cpp
 120 | ////////////////////////////////////////////////////////////////////////////////
 121 | namespace detail {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 123-134

```cpp
 123 | template <
 124 |   /// Layout type for A matrix operand
 125 |   typename LayoutA_,
 126 |   /// Layout type for B matrix operand
 127 |   typename LayoutB_,
 128 |   /// Complex elementwise transformation 
 129 |   ComplexTransform TransformA,
 130 |   /// Complex elementwise transformation 
 131 |   ComplexTransform TransformB,
 132 |   /// Blas3 computation mode (symmetric/hermitian)
 133 |   BlasMode BlasMode_
 134 |   > struct Rank2KTransposedComplexTransform {
```
**EN:** This block declares or specializes `Rank2KTransposedComplexTransform`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `Rank2KTransposedComplexTransform`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 136-139

```cpp
 136 |   static ComplexTransform const kTransformA = TransformA;
 137 |   static ComplexTransform const kTransformB = TransformB;
 138 | 
 139 | };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 141-146

```cpp
 141 |   // partial specializations for HER2K CUBLAS_OP_N layout (ColumMajor)
 142 | template <>
 143 |   struct Rank2KTransposedComplexTransform <
 144 |   layout::ColumnMajor, layout::ColumnMajor, 
 145 |   ComplexTransform::kNone, ComplexTransform::kNone,
 146 |   BlasMode::kHermitian> {
```
**EN:** This block declares or specializes `Rank2KTransposedComplexTransform`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `Rank2KTransposedComplexTransform`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 148-151

```cpp
 148 |   static ComplexTransform const kTransformA = ComplexTransform::kConjugate;
 149 |   static ComplexTransform const kTransformB = ComplexTransform::kNone;
 150 | 
 151 | };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 153-158

```cpp
 153 |   // partial specializations for HER2K CUBLAS_OP_C layout (RowMajor + Complex conjugate) 
 154 | template <>
 155 |   struct Rank2KTransposedComplexTransform <
 156 |   layout::RowMajor, layout::RowMajor, 
 157 |   ComplexTransform::kConjugate, ComplexTransform::kConjugate,
 158 |   BlasMode::kHermitian> {
```
**EN:** This block declares or specializes `Rank2KTransposedComplexTransform`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `Rank2KTransposedComplexTransform`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 160-165

```cpp
 160 |   static ComplexTransform const kTransformA = ComplexTransform::kNone;
 161 |   static ComplexTransform const kTransformB = ComplexTransform::kConjugate;
 162 | 
 163 | };
 164 | 
 165 | }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 169-198

```cpp
 169 | /// Partial specialization for Hopper Architecture complex datatype (symmetric)
 170 | template <
 171 |     /// Element type for A matrix operand
 172 |     typename ElementA,
 173 |     /// Layout type for A matrix operand
 174 |     typename LayoutA,
 175 |     /// Element type for B matrix operand
 176 |     typename ElementB,
 177 |     /// Layout type for B matrix operand
 178 |     typename LayoutB,
 179 |     /// Element type for C and D matrix operands
 180 |     typename ElementC,
 181 |     /// Fill Mode for C (kLower or kUpper)
 182 |     FillMode FillModeC,
 183 |     /// Element type for internal accumulation
 184 |     typename ElementAccumulator,
 185 |     /// Threadblock-level tile size (concept: GemmShape)
 186 |     typename ThreadblockShape,
 187 |     /// Warp-level tile size (concept: GemmShape)
 188 |     typename WarpShape,
 189 |     /// Warp-level tile size (concept: GemmShape)
 190 |     typename InstructionShape,
 191 |     /// Epilogue output operator
 192 |     typename EpilogueOutputOp,
 193 |     /// Threadblock-level swizzling operator
 194 |     typename ThreadblockSwizzle,
 195 |     /// Number of stages used in the pipelined mainloop
 196 |     int Stages,
 197 |     /// Complex elementwise transformation on A operand
 198 |     ComplexTransform TransformA,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 199-211

```cpp
 199 |     /// Complex elementwise transformation on B operand
 200 |     ComplexTransform TransformB,
 201 |     /// Operation performed by GEMM
 202 |     typename Operator,
 203 |     /// If true, kernel is configured to support serial reduction in the
 204 |     /// epilogue
 205 |     bool SplitKSerial>
 206 | struct DefaultRank2KComplex<
 207 |   ElementA, LayoutA, ElementB, LayoutB, ElementC, 
 208 |   layout::RowMajor, FillModeC, ElementAccumulator, arch::OpClassTensorOp,
 209 |   arch::Sm90, ThreadblockShape, WarpShape, InstructionShape, 
 210 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 211 |   TransformA, TransformB, Operator, SplitKSerial, BlasMode::kSymmetric> {
```
**EN:** This block declares or specializes `DefaultRank2KComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 213-213

```cpp
 213 |   static BlasMode const kBlasMode = BlasMode::kSymmetric;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 215-221

```cpp
 215 |   /// Define the threadblock-scoped matrix multiply-accumulate (A x B^T)
 216 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 217 |       ElementA, LayoutA, 
 218 |       ElementB, typename layout::LayoutTranspose<LayoutB>::type, 
 219 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm90, 
 220 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 221 |       TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 223-229

```cpp
 223 |   /// Define the threadblock-scoped matrix multiply-accumulate (B x A^T)
 224 |   using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 225 |       ElementB, LayoutB, 
 226 |       ElementA, typename layout::LayoutTranspose<LayoutA>::type, 
 227 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm90, 
 228 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 229 |       TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 231-235

```cpp
 231 |   /// Define the epilogue
 232 |   using Epilogue =
 233 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOpBlas3<
 234 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 235 |           EpilogueOutputOp::kCount, Operator, kBlasMode>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 237-240

```cpp
 237 |   /// Define the kernel-level Rank2K operator.
 238 |   using Rank2Kkernel = kernel::Rank2KUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, FillModeC, kBlasMode>;
 239 | 
 240 | };
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 244-273

```cpp
 244 | /// Partial specialization for Hopper Architecture complex datatype (hermitian)
 245 | template <
 246 |     /// Element type for A matrix operand
 247 |     typename ElementA,
 248 |     /// Layout type for A matrix operand
 249 |     typename LayoutA,
 250 |     /// Element type for B matrix operand
 251 |     typename ElementB,
 252 |     /// Layout type for B matrix operand
 253 |     typename LayoutB,
 254 |     /// Element type for C and D matrix operands
 255 |     typename ElementC,
 256 |     /// Fill Mode for C (kLower or kUpper)
 257 |     FillMode FillModeC,
 258 |     /// Element type for internal accumulation
 259 |     typename ElementAccumulator,
 260 |     /// Threadblock-level tile size (concept: GemmShape)
 261 |     typename ThreadblockShape,
 262 |     /// Warp-level tile size (concept: GemmShape)
 263 |     typename WarpShape,
 264 |     /// Warp-level tile size (concept: GemmShape)
 265 |     typename InstructionShape,
 266 |     /// Epilogue output operator
 267 |     typename EpilogueOutputOp,
 268 |     /// Threadblock-level swizzling operator
 269 |     typename ThreadblockSwizzle,
 270 |     /// Number of stages used in the pipelined mainloop
 271 |     int Stages,
 272 |     /// Complex elementwise transformation on A operand
 273 |     ComplexTransform TransformA,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 274-286

```cpp
 274 |     /// Complex elementwise transformation on B operand
 275 |     ComplexTransform TransformB,
 276 |     /// Operation performed by GEMM
 277 |     typename Operator,
 278 |     /// If true, kernel is configured to support serial reduction in the
 279 |     /// epilogue
 280 |     bool SplitKSerial>
 281 | struct DefaultRank2KComplex<
 282 |   ElementA, LayoutA, ElementB, LayoutB, ElementC, 
 283 |   layout::RowMajor, FillModeC, ElementAccumulator, arch::OpClassTensorOp,
 284 |   arch::Sm90, ThreadblockShape, WarpShape, InstructionShape, 
 285 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 286 |   TransformA, TransformB, Operator, SplitKSerial, BlasMode::kHermitian> {
```
**EN:** This block declares or specializes `DefaultRank2KComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 288-288

```cpp
 288 |   static BlasMode const kBlasMode = BlasMode::kHermitian;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 290-292

```cpp
 290 |   // Complex transform for input A and B matrices (function on input layout)
 291 |   static ComplexTransform const kTransformA = TransformA;
 292 |   static ComplexTransform const kTransformB = TransformB;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 294-297

```cpp
 294 |   using TransposedComplexTransform = detail::Rank2KTransposedComplexTransform<
 295 |                                         LayoutA, LayoutB, 
 296 |                                         TransformA, TransformB,
 297 |                                         kBlasMode>;
```
**EN:** This alias block derives concise type names `TransposedComplexTransform` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `TransposedComplexTransform` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 299-301

```cpp
 299 |   // Complex transform on operandA and operandB (function of blas3 computation)
 300 |   static ComplexTransform const kTransformOperandA = TransposedComplexTransform::kTransformA;
 301 |   static ComplexTransform const kTransformOperandB = TransposedComplexTransform::kTransformB;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 303-309

```cpp
 303 |   /// Define the threadblock-scoped matrix multiply-accumulate (A x B^H)
 304 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 305 |       ElementA, LayoutA, 
 306 |       ElementB, typename layout::LayoutTranspose<LayoutB>::type, 
 307 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm90, 
 308 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 309 |       kTransformOperandA, kTransformOperandB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 311-317

```cpp
 311 |   /// Define the threadblock-scoped matrix multiply-accumulate (B x A^H)
 312 |   using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 313 |       ElementB, LayoutB, 
 314 |       ElementA, typename layout::LayoutTranspose<LayoutA>::type, 
 315 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm90, 
 316 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 317 |       kTransformOperandA, kTransformOperandB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 319-323

```cpp
 319 |   /// Define the epilogue
 320 |   using Epilogue =
 321 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOpBlas3<
 322 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 323 |           EpilogueOutputOp::kCount, Operator, kBlasMode>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 325-328

```cpp
 325 |   /// Define the kernel-level Rank2K operator.
 326 |   using Rank2Kkernel = kernel::Rank2KUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, FillModeC, kBlasMode>;
 327 | 
 328 | };
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 332-361

```cpp
 332 | /// Partial specialization for Ampere Architecture complex datatype (symmetric)
 333 | template <
 334 |     /// Element type for A matrix operand
 335 |     typename ElementA,
 336 |     /// Layout type for A matrix operand
 337 |     typename LayoutA,
 338 |     /// Element type for B matrix operand
 339 |     typename ElementB,
 340 |     /// Layout type for B matrix operand
 341 |     typename LayoutB,
 342 |     /// Element type for C and D matrix operands
 343 |     typename ElementC,
 344 |     /// Fill Mode for C (kLower or kUpper)
 345 |     FillMode FillModeC,
 346 |     /// Element type for internal accumulation
 347 |     typename ElementAccumulator,
 348 |     /// Threadblock-level tile size (concept: GemmShape)
 349 |     typename ThreadblockShape,
 350 |     /// Warp-level tile size (concept: GemmShape)
 351 |     typename WarpShape,
 352 |     /// Warp-level tile size (concept: GemmShape)
 353 |     typename InstructionShape,
 354 |     /// Epilogue output operator
 355 |     typename EpilogueOutputOp,
 356 |     /// Threadblock-level swizzling operator
 357 |     typename ThreadblockSwizzle,
 358 |     /// Number of stages used in the pipelined mainloop
 359 |     int Stages,
 360 |     /// Complex elementwise transformation on A operand
 361 |     ComplexTransform TransformA,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 362-374

```cpp
 362 |     /// Complex elementwise transformation on B operand
 363 |     ComplexTransform TransformB,
 364 |     /// Operation performed by GEMM
 365 |     typename Operator,
 366 |     /// If true, kernel is configured to support serial reduction in the
 367 |     /// epilogue
 368 |     bool SplitKSerial>
 369 | struct DefaultRank2KComplex<
 370 |   ElementA, LayoutA, ElementB, LayoutB, ElementC, 
 371 |   layout::RowMajor, FillModeC, ElementAccumulator, arch::OpClassTensorOp,
 372 |   arch::Sm80, ThreadblockShape, WarpShape, InstructionShape, 
 373 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 374 |   TransformA, TransformB, Operator, SplitKSerial, BlasMode::kSymmetric> {
```
**EN:** This block declares or specializes `DefaultRank2KComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 376-376

```cpp
 376 |   static BlasMode const kBlasMode = BlasMode::kSymmetric;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 378-384

```cpp
 378 |   /// Define the threadblock-scoped matrix multiply-accumulate (A x B^T)
 379 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 380 |       ElementA, LayoutA, 
 381 |       ElementB, typename layout::LayoutTranspose<LayoutB>::type, 
 382 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80, 
 383 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 384 |       TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 386-392

```cpp
 386 |   /// Define the threadblock-scoped matrix multiply-accumulate (B x A^T)
 387 |   using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 388 |       ElementB, LayoutB, 
 389 |       ElementA, typename layout::LayoutTranspose<LayoutA>::type, 
 390 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80, 
 391 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 392 |       TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 394-398

```cpp
 394 |   /// Define the epilogue
 395 |   using Epilogue =
 396 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOpBlas3<
 397 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 398 |           EpilogueOutputOp::kCount, Operator, kBlasMode>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 400-403

```cpp
 400 |   /// Define the kernel-level Rank2K operator.
 401 |   using Rank2Kkernel = kernel::Rank2KUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, FillModeC, kBlasMode>;
 402 | 
 403 | };
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 407-436

```cpp
 407 | /// Partial specialization for Ampere Architecture complex datatype (hermitian)
 408 | template <
 409 |     /// Element type for A matrix operand
 410 |     typename ElementA,
 411 |     /// Layout type for A matrix operand
 412 |     typename LayoutA,
 413 |     /// Element type for B matrix operand
 414 |     typename ElementB,
 415 |     /// Layout type for B matrix operand
 416 |     typename LayoutB,
 417 |     /// Element type for C and D matrix operands
 418 |     typename ElementC,
 419 |     /// Fill Mode for C (kLower or kUpper)
 420 |     FillMode FillModeC,
 421 |     /// Element type for internal accumulation
 422 |     typename ElementAccumulator,
 423 |     /// Threadblock-level tile size (concept: GemmShape)
 424 |     typename ThreadblockShape,
 425 |     /// Warp-level tile size (concept: GemmShape)
 426 |     typename WarpShape,
 427 |     /// Warp-level tile size (concept: GemmShape)
 428 |     typename InstructionShape,
 429 |     /// Epilogue output operator
 430 |     typename EpilogueOutputOp,
 431 |     /// Threadblock-level swizzling operator
 432 |     typename ThreadblockSwizzle,
 433 |     /// Number of stages used in the pipelined mainloop
 434 |     int Stages,
 435 |     /// Complex elementwise transformation on A operand
 436 |     ComplexTransform TransformA,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 437-449

```cpp
 437 |     /// Complex elementwise transformation on B operand
 438 |     ComplexTransform TransformB,
 439 |     /// Operation performed by GEMM
 440 |     typename Operator,
 441 |     /// If true, kernel is configured to support serial reduction in the
 442 |     /// epilogue
 443 |     bool SplitKSerial>
 444 | struct DefaultRank2KComplex<
 445 |   ElementA, LayoutA, ElementB, LayoutB, ElementC, 
 446 |   layout::RowMajor, FillModeC, ElementAccumulator, arch::OpClassTensorOp,
 447 |   arch::Sm80, ThreadblockShape, WarpShape, InstructionShape, 
 448 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 449 |   TransformA, TransformB, Operator, SplitKSerial, BlasMode::kHermitian> {
```
**EN:** This block declares or specializes `DefaultRank2KComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 451-451

```cpp
 451 |   static BlasMode const kBlasMode = BlasMode::kHermitian;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 453-455

```cpp
 453 |   // Complex transform for input A and B matrices (function on input layout)
 454 |   static ComplexTransform const kTransformA = TransformA;
 455 |   static ComplexTransform const kTransformB = TransformB;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 457-460

```cpp
 457 |   using TransposedComplexTransform = detail::Rank2KTransposedComplexTransform<
 458 |                                         LayoutA, LayoutB, 
 459 |                                         TransformA, TransformB,
 460 |                                         kBlasMode>;
```
**EN:** This alias block derives concise type names `TransposedComplexTransform` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `TransposedComplexTransform` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 462-464

```cpp
 462 |   // Complex transform on operandA and operandB (function of blas3 computation)
 463 |   static ComplexTransform const kTransformOperandA = TransposedComplexTransform::kTransformA;
 464 |   static ComplexTransform const kTransformOperandB = TransposedComplexTransform::kTransformB;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 466-472

```cpp
 466 |   /// Define the threadblock-scoped matrix multiply-accumulate (A x B^H)
 467 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 468 |       ElementA, LayoutA, 
 469 |       ElementB, typename layout::LayoutTranspose<LayoutB>::type, 
 470 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80, 
 471 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 472 |       kTransformOperandA, kTransformOperandB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 474-480

```cpp
 474 |   /// Define the threadblock-scoped matrix multiply-accumulate (B x A^H)
 475 |   using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplex<
 476 |       ElementB, LayoutB, 
 477 |       ElementA, typename layout::LayoutTranspose<LayoutA>::type, 
 478 |       ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80, 
 479 |       ThreadblockShape, WarpShape, InstructionShape, Stages, 
 480 |       kTransformOperandA, kTransformOperandB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 482-486

```cpp
 482 |   /// Define the epilogue
 483 |   using Epilogue =
 484 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOpBlas3<
 485 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 486 |           EpilogueOutputOp::kCount, Operator, kBlasMode>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 488-491

```cpp
 488 |   /// Define the kernel-level Rank2K operator.
 489 |   using Rank2Kkernel = kernel::Rank2KUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, FillModeC, kBlasMode>;
 490 | 
 491 | };
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 496-498

```cpp
 496 | }  // namespace kernel
 497 | }  // namespace gemm
 498 | }  // namespace cutlass
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
- **Included headers / 引入头文件:** `cutlass/blas3.h`, `cutlass/layout/matrix.h`, `cutlass/arch/wmma.h`, `cutlass/epilogue/threadblock/epilogue.h`, `cutlass/epilogue/thread/linear_combination.h`, `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/rank_2k_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_multistage_mma_complex.h`, `cutlass/gemm/threadblock/threadblock_swizzle.h`, `cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op_blas3.h`, ... (+2 more)
- **Subsystems / 子系统:** Epilogue/output pipeline / Epilogue 输出流水, Threadblock MMA building blocks / 线程块级 MMA 构件
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/rank_2k_universal.h`
