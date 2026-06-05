# default_symm_complex.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/default_symm_complex.h`
- **Purpose / 用途 (EN):** Defines the default kernel composition and specialization rules for SYMM complex. Briefly, the file comment says: Default kernel-level SYMM/HEMM definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue.
- **Purpose / 用途 (CN):** 定义 SYMM complex 的默认内核组合与模板特化规则。 文件注释的简要说明是：Default kernel-level SYMM/HEMM definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue。
- **Line count / 行数:** 508

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
  34 |       Default kernel-level SYMM/HEMM definitions combine threadblock-scoped matrix multiply-add with
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

### Lines 50-56

```cpp
  50 | #include "cutlass/gemm/gemm.h"
  51 | #include "cutlass/gemm/kernel/symm_universal.h"
  52 | #include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
  53 | #include "cutlass/gemm/threadblock/default_mma.h"
  54 | #include "cutlass/gemm/threadblock/default_multistage_trmm_complex.h"
  55 | #include "cutlass/gemm/threadblock/default_multistage_mma_complex.h"
  56 | #include "cutlass/gemm/threadblock/threadblock_swizzle.h"
```
**EN:** This include block imports `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/symm_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_multistage_trmm_complex.h`, `cutlass/gemm/threadblock/default_multistage_mma_complex.h`, ... (+1 more), providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/symm_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_multistage_trmm_complex.h`, `cutlass/gemm/threadblock/default_multistage_mma_complex.h`, ... (+1 more)，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 58-59

```cpp
  58 | #include "cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op.h"
  59 | #include "cutlass/transform/threadblock/predicated_tile_iterator.h"
```
**EN:** This include block imports `cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op.h`, `cutlass/transform/threadblock/predicated_tile_iterator.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/epilogue/threadblock/default_epilogue_complex_tensor_op.h`, `cutlass/transform/threadblock/predicated_tile_iterator.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 61-63

```cpp
  61 | #if defined(CUTLASS_ARCH_WMMA_ENABLED)
  62 | #include "cutlass/epilogue/threadblock/default_epilogue_wmma_tensor_op.h"
  63 | #endif //CUTLASS_ARCH_WMMA_ENABLED
```
**EN:** This conditional-compilation block enables the following code only when the required architecture or backend feature is available.
**CN:** 这一段条件编译逻辑只在目标架构或后端特性可用时启用后续代码。

### Lines 68-70

```cpp
  68 | namespace cutlass {
  69 | namespace gemm {
  70 | namespace kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 74-103

```cpp
  74 | template <
  75 |     /// Element type for A matrix operand
  76 |     typename ElementA_,
  77 |     /// Layout type for A matrix operand
  78 |     typename LayoutA_,
  79 |     /// Side Mode for A (kLeft or kRight)
  80 |     SideMode kSideModeA,
  81 |     /// Fill Mode for A (kLower or kUpper)
  82 |     FillMode kFillModeA,
  83 |     /// Element type for B matrix operand
  84 |     typename ElementB_,
  85 |     /// Layout type for B matrix operand
  86 |     typename LayoutB_,
  87 |     /// Element type for C and D matrix operands
  88 |     typename ElementC_,
  89 |     /// Layout type for C and D matrix operands
  90 |     typename LayoutC_,
  91 |     /// Element type for internal accumulation
  92 |     typename ElementAccumulator,
  93 |     /// Operator class tag
  94 |     typename OperatorClass,
  95 |     /// Tag indicating architecture to tune for
  96 |     typename ArchTag,
  97 |     /// Threadblock-level tile size (concept: GemmShape)
  98 |     typename ThreadblockShape,
  99 |     /// Warp-level tile size (concept: GemmShape)
 100 |     typename WarpShape,
 101 |     /// Warp-level tile size (concept: GemmShape)
 102 |     typename InstructionShape,
 103 |     /// Epilogue output operator
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 104-116

```cpp
 104 |     typename EpilogueOutputOp,
 105 |     /// Threadblock-level swizzling operator
 106 |     typename ThreadblockSwizzle,
 107 |     /// Number of stages used in the pipelined mainloop
 108 |     int Stages,
 109 |     /// Operation performed by GEMM
 110 |     typename Operator,
 111 |     /// If true, kernel is configured to support serial reduction in the
 112 |     /// epilogue
 113 |     bool SplitKSerial,
 114 |     /// Blas3 computation mode
 115 |     BlasMode BlasMode_ = BlasMode::kSymmetric>
 116 | struct DefaultSymmComplex;
```
**EN:** This block declares or specializes `DefaultSymmComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultSymmComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 120-149

```cpp
 120 | /// Partial specialization for Hopper Architecture complex datatype (symmetric)
 121 | template <
 122 |     /// Element type for A matrix operand
 123 |     typename ElementA,
 124 |     /// Layout type for A matrix operand
 125 |     typename LayoutA,
 126 |     /// Side Mode for A (kLeft or kRight)
 127 |     SideMode kSideModeA,
 128 |     /// Fill Mode for A (kLower or kUpper)
 129 |     FillMode kFillModeA,
 130 |     /// Element type for B matrix operand
 131 |     typename ElementB,
 132 |     /// Layout type for B matrix operand
 133 |     typename LayoutB,
 134 |     /// Element type for C and D matrix operands
 135 |     typename ElementC,
 136 |     /// Element type for internal accumulation
 137 |     typename ElementAccumulator,
 138 |     /// Threadblock-level tile size (concept: GemmShape)
 139 |     typename ThreadblockShape,
 140 |     /// Warp-level tile size (concept: GemmShape)
 141 |     typename WarpShape,
 142 |     /// Warp-level tile size (concept: GemmShape)
 143 |     typename InstructionShape,
 144 |     /// Epilogue output operator
 145 |     typename EpilogueOutputOp,
 146 |     /// Threadblock-level swizzling operator
 147 |     typename ThreadblockSwizzle,
 148 |     /// Number of stages used in the pipelined mainloop
 149 |     int Stages,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 150-160

```cpp
 150 |     /// Operation performed by GEMM
 151 |     typename Operator,
 152 |     /// If true, kernel is configured to support serial reduction in the
 153 |     /// epilogue
 154 |     bool SplitKSerial>
 155 | struct DefaultSymmComplex<
 156 |   ElementA, LayoutA, kSideModeA, kFillModeA, ElementB, LayoutB, ElementC, 
 157 |   layout::RowMajor, ElementAccumulator, arch::OpClassTensorOp,
 158 |   arch::Sm90, ThreadblockShape, WarpShape, InstructionShape, 
 159 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 160 |   Operator, SplitKSerial, BlasMode::kSymmetric> {
```
**EN:** This block declares or specializes `DefaultSymmComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultSymmComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 162-165

```cpp
 162 |   static BlasMode const kBlasMode = BlasMode::kSymmetric;
 163 |   // Complex Transform don't apply to A or B for SYMM
 164 |   static ComplexTransform const TransformA = ComplexTransform::kNone; 
 165 |   static ComplexTransform const TransformB = ComplexTransform::kNone; 
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 167-177

```cpp
 167 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 168 |   /// TRMM - with diagonal: alpha * A * B or alpha * B * A
 169 | 	static const DiagType kDiagTypeMma1 = DiagType::kNonUnit;
 170 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 171 |       ElementA, LayoutA, 
 172 |       ElementB, LayoutB, 
 173 |       kSideModeA, kFillModeA, kDiagTypeMma1, 
 174 |       ElementAccumulator, layout::RowMajor, 
 175 |       arch::OpClassTensorOp, arch::Sm90,
 176 |       ThreadblockShape, WarpShape, InstructionShape,
 177 |       Stages, TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 179-199

```cpp
 179 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 180 |   /// TRMM - withOUT diagonal: alpha * AT * B or alpha * B * AT
 181 | 	static const DiagType kDiagTypeMma2 = DiagType::kZero;
 182 |   using LayoutAMma2 = typename platform::conditional<
 183 |                                 (kSideModeA == SideMode::kLeft), 
 184 |                                 typename layout::LayoutTranspose<LayoutA>::type, 
 185 |                                 LayoutA
 186 |                               >::type;
 187 |   using LayoutBMma2 = typename platform::conditional<
 188 |                                 (kSideModeA == SideMode::kLeft), 
 189 |                                 LayoutB, 
 190 |                                 typename layout::LayoutTranspose<LayoutB>::type
 191 |                               >::type; 
 192 | 	using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 193 | 			ElementA, LayoutAMma2, 
 194 | 			ElementB, LayoutBMma2, 
 195 | 			kSideModeA, InvertFillMode<kFillModeA>::mode, kDiagTypeMma2, 
 196 | 			ElementAccumulator, layout::RowMajor, 
 197 | 			arch::OpClassTensorOp, arch::Sm90,
 198 | 			ThreadblockShape, WarpShape, InstructionShape,
 199 | 			Stages, TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `LayoutAMma2`, `LayoutBMma2`, `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `LayoutAMma2`, `LayoutBMma2`, `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 201-205

```cpp
 201 |   /// Define the epilogue
 202 |   using Epilogue =
 203 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOp<
 204 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 205 |           EpilogueOutputOp::kCount, Operator>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 207-210

```cpp
 207 |   /// Define the kernel-level Symm operator.
 208 |   using SymmKernel = kernel::SymmUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, kSideModeA, kFillModeA>;
 209 | 
 210 | };
```
**EN:** This alias block derives concise type names `SymmKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `SymmKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 214-243

```cpp
 214 | /// Partial specialization for Hopper Architecture complex datatype (hermitian)
 215 | template <
 216 |     /// Element type for A matrix operand
 217 |     typename ElementA,
 218 |     /// Layout type for A matrix operand
 219 |     typename LayoutA,
 220 |     /// Side Mode for A (kLeft or kRight)
 221 |     SideMode kSideModeA,
 222 |     /// Fill Mode for A (kLower or kUpper)
 223 |     FillMode kFillModeA,
 224 |     /// Element type for B matrix operand
 225 |     typename ElementB,
 226 |     /// Layout type for B matrix operand
 227 |     typename LayoutB,
 228 |     /// Element type for C and D matrix operands
 229 |     typename ElementC,
 230 |     /// Element type for internal accumulation
 231 |     typename ElementAccumulator,
 232 |     /// Threadblock-level tile size (concept: GemmShape)
 233 |     typename ThreadblockShape,
 234 |     /// Warp-level tile size (concept: GemmShape)
 235 |     typename WarpShape,
 236 |     /// Warp-level tile size (concept: GemmShape)
 237 |     typename InstructionShape,
 238 |     /// Epilogue output operator
 239 |     typename EpilogueOutputOp,
 240 |     /// Threadblock-level swizzling operator
 241 |     typename ThreadblockSwizzle,
 242 |     /// Number of stages used in the pipelined mainloop
 243 |     int Stages,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 244-254

```cpp
 244 |     /// Operation performed by GEMM
 245 |     typename Operator,
 246 |     /// If true, kernel is configured to support serial reduction in the
 247 |     /// epilogue
 248 |     bool SplitKSerial>
 249 | struct DefaultSymmComplex<
 250 |   ElementA, LayoutA, kSideModeA, kFillModeA, ElementB, LayoutB, ElementC, 
 251 |   layout::RowMajor, ElementAccumulator, arch::OpClassTensorOp,
 252 |   arch::Sm90, ThreadblockShape, WarpShape, InstructionShape, 
 253 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 254 |   Operator, SplitKSerial, BlasMode::kHermitian> {
```
**EN:** This block declares or specializes `DefaultSymmComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultSymmComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 256-256

```cpp
 256 |   static BlasMode const kBlasMode = BlasMode::kHermitian;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 259-271

```cpp
 259 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 260 |   /// TRMM - with diagonal: alpha * A * B or alpha * B * A
 261 | 	static const DiagType kDiagTypeMma1 = DiagType::kNonUnit;
 262 |   static ComplexTransform const TransformAMma1 = ComplexTransform::kNone; 
 263 |   static ComplexTransform const TransformBMma1 = ComplexTransform::kNone; 
 264 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 265 |       ElementA, LayoutA, 
 266 |       ElementB, LayoutB, 
 267 |       kSideModeA, kFillModeA, kDiagTypeMma1, 
 268 |       ElementAccumulator, layout::RowMajor, 
 269 |       arch::OpClassTensorOp, arch::Sm90,
 270 |       ThreadblockShape, WarpShape, InstructionShape,
 271 |       Stages, TransformAMma1, TransformBMma1, Operator, BlasMode::kHermitian>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 273-289

```cpp
 273 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 274 |   /// TRMM - withOUT diagonal - with conjugate transpose: alpha * AT * B or alpha * B * AT
 275 | 	static const DiagType kDiagTypeMma2 = DiagType::kZero;
 276 |   using LayoutAMma2 = typename platform::conditional<
 277 |                                 (kSideModeA == SideMode::kLeft), 
 278 |                                 typename layout::LayoutTranspose<LayoutA>::type, 
 279 |                                 LayoutA
 280 |                               >::type;
 281 |   using LayoutBMma2 = typename platform::conditional<
 282 |                                 (kSideModeA == SideMode::kLeft), 
 283 |                                 LayoutB, 
 284 |                                 typename layout::LayoutTranspose<LayoutB>::type
 285 |                               >::type;
 286 |   static ComplexTransform const TransformAMma2 = (kSideModeA == SideMode::kLeft) ? 
 287 |                                               ComplexTransform::kConjugate : ComplexTransform::kNone;
 288 |   static ComplexTransform const TransformBMma2 = (kSideModeA == SideMode::kLeft) ? 
 289 |                                               ComplexTransform::kNone : ComplexTransform::kConjugate;
```
**EN:** This alias block derives concise type names `LayoutAMma2`, `LayoutBMma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `LayoutAMma2`, `LayoutBMma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 291-298

```cpp
 291 | 	using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 292 | 			ElementA, LayoutAMma2, 
 293 | 			ElementB, LayoutBMma2, 
 294 | 			kSideModeA, InvertFillMode<kFillModeA>::mode, kDiagTypeMma2, 
 295 | 			ElementAccumulator, layout::RowMajor, 
 296 | 			arch::OpClassTensorOp, arch::Sm90,
 297 | 			ThreadblockShape, WarpShape, InstructionShape,
 298 | 			Stages, TransformAMma2, TransformBMma2, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 300-304

```cpp
 300 |   /// Define the epilogue
 301 |   using Epilogue =
 302 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOp<
 303 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 304 |           EpilogueOutputOp::kCount, Operator>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 306-309

```cpp
 306 |   /// Define the kernel-level Symm operator.
 307 |   using SymmKernel = kernel::SymmUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, kSideModeA, kFillModeA>;
 308 | 
 309 | };
```
**EN:** This alias block derives concise type names `SymmKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `SymmKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 313-342

```cpp
 313 | /// Partial specialization for Ampere Architecture complex datatype (symmetric)
 314 | template <
 315 |     /// Element type for A matrix operand
 316 |     typename ElementA,
 317 |     /// Layout type for A matrix operand
 318 |     typename LayoutA,
 319 |     /// Side Mode for A (kLeft or kRight)
 320 |     SideMode kSideModeA,
 321 |     /// Fill Mode for A (kLower or kUpper)
 322 |     FillMode kFillModeA,
 323 |     /// Element type for B matrix operand
 324 |     typename ElementB,
 325 |     /// Layout type for B matrix operand
 326 |     typename LayoutB,
 327 |     /// Element type for C and D matrix operands
 328 |     typename ElementC,
 329 |     /// Element type for internal accumulation
 330 |     typename ElementAccumulator,
 331 |     /// Threadblock-level tile size (concept: GemmShape)
 332 |     typename ThreadblockShape,
 333 |     /// Warp-level tile size (concept: GemmShape)
 334 |     typename WarpShape,
 335 |     /// Warp-level tile size (concept: GemmShape)
 336 |     typename InstructionShape,
 337 |     /// Epilogue output operator
 338 |     typename EpilogueOutputOp,
 339 |     /// Threadblock-level swizzling operator
 340 |     typename ThreadblockSwizzle,
 341 |     /// Number of stages used in the pipelined mainloop
 342 |     int Stages,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 343-353

```cpp
 343 |     /// Operation performed by GEMM
 344 |     typename Operator,
 345 |     /// If true, kernel is configured to support serial reduction in the
 346 |     /// epilogue
 347 |     bool SplitKSerial>
 348 | struct DefaultSymmComplex<
 349 |   ElementA, LayoutA, kSideModeA, kFillModeA, ElementB, LayoutB, ElementC, 
 350 |   layout::RowMajor, ElementAccumulator, arch::OpClassTensorOp,
 351 |   arch::Sm80, ThreadblockShape, WarpShape, InstructionShape, 
 352 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 353 |   Operator, SplitKSerial, BlasMode::kSymmetric> {
```
**EN:** This block declares or specializes `DefaultSymmComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultSymmComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 355-358

```cpp
 355 |   static BlasMode const kBlasMode = BlasMode::kSymmetric;
 356 |   // Complex Transform don't apply to A or B for SYMM
 357 |   static ComplexTransform const TransformA = ComplexTransform::kNone; 
 358 |   static ComplexTransform const TransformB = ComplexTransform::kNone; 
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 360-370

```cpp
 360 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 361 |   /// TRMM - with diagonal: alpha * A * B or alpha * B * A
 362 | 	static const DiagType kDiagTypeMma1 = DiagType::kNonUnit;
 363 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 364 |       ElementA, LayoutA, 
 365 |       ElementB, LayoutB, 
 366 |       kSideModeA, kFillModeA, kDiagTypeMma1, 
 367 |       ElementAccumulator, layout::RowMajor, 
 368 |       arch::OpClassTensorOp, arch::Sm80,
 369 |       ThreadblockShape, WarpShape, InstructionShape,
 370 |       Stages, TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 372-392

```cpp
 372 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 373 |   /// TRMM - withOUT diagonal: alpha * AT * B or alpha * B * AT
 374 | 	static const DiagType kDiagTypeMma2 = DiagType::kZero;
 375 |   using LayoutAMma2 = typename platform::conditional<
 376 |                                 (kSideModeA == SideMode::kLeft), 
 377 |                                 typename layout::LayoutTranspose<LayoutA>::type, 
 378 |                                 LayoutA
 379 |                               >::type;
 380 |   using LayoutBMma2 = typename platform::conditional<
 381 |                                 (kSideModeA == SideMode::kLeft), 
 382 |                                 LayoutB, 
 383 |                                 typename layout::LayoutTranspose<LayoutB>::type
 384 |                               >::type; 
 385 | 	using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 386 | 			ElementA, LayoutAMma2, 
 387 | 			ElementB, LayoutBMma2, 
 388 | 			kSideModeA, InvertFillMode<kFillModeA>::mode, kDiagTypeMma2, 
 389 | 			ElementAccumulator, layout::RowMajor, 
 390 | 			arch::OpClassTensorOp, arch::Sm80,
 391 | 			ThreadblockShape, WarpShape, InstructionShape,
 392 | 			Stages, TransformA, TransformB, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `LayoutAMma2`, `LayoutBMma2`, `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `LayoutAMma2`, `LayoutBMma2`, `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 394-398

```cpp
 394 |   /// Define the epilogue
 395 |   using Epilogue =
 396 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOp<
 397 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 398 |           EpilogueOutputOp::kCount, Operator>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 400-403

```cpp
 400 |   /// Define the kernel-level Symm operator.
 401 |   using SymmKernel = kernel::SymmUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, kSideModeA, kFillModeA>;
 402 | 
 403 | };
```
**EN:** This alias block derives concise type names `SymmKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `SymmKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 407-436

```cpp
 407 | /// Partial specialization for Ampere Architecture complex datatype (hermitian)
 408 | template <
 409 |     /// Element type for A matrix operand
 410 |     typename ElementA,
 411 |     /// Layout type for A matrix operand
 412 |     typename LayoutA,
 413 |     /// Side Mode for A (kLeft or kRight)
 414 |     SideMode kSideModeA,
 415 |     /// Fill Mode for A (kLower or kUpper)
 416 |     FillMode kFillModeA,
 417 |     /// Element type for B matrix operand
 418 |     typename ElementB,
 419 |     /// Layout type for B matrix operand
 420 |     typename LayoutB,
 421 |     /// Element type for C and D matrix operands
 422 |     typename ElementC,
 423 |     /// Element type for internal accumulation
 424 |     typename ElementAccumulator,
 425 |     /// Threadblock-level tile size (concept: GemmShape)
 426 |     typename ThreadblockShape,
 427 |     /// Warp-level tile size (concept: GemmShape)
 428 |     typename WarpShape,
 429 |     /// Warp-level tile size (concept: GemmShape)
 430 |     typename InstructionShape,
 431 |     /// Epilogue output operator
 432 |     typename EpilogueOutputOp,
 433 |     /// Threadblock-level swizzling operator
 434 |     typename ThreadblockSwizzle,
 435 |     /// Number of stages used in the pipelined mainloop
 436 |     int Stages,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 437-447

```cpp
 437 |     /// Operation performed by GEMM
 438 |     typename Operator,
 439 |     /// If true, kernel is configured to support serial reduction in the
 440 |     /// epilogue
 441 |     bool SplitKSerial>
 442 | struct DefaultSymmComplex<
 443 |   ElementA, LayoutA, kSideModeA, kFillModeA, ElementB, LayoutB, ElementC, 
 444 |   layout::RowMajor, ElementAccumulator, arch::OpClassTensorOp,
 445 |   arch::Sm80, ThreadblockShape, WarpShape, InstructionShape, 
 446 |   EpilogueOutputOp, ThreadblockSwizzle, Stages, 
 447 |   Operator, SplitKSerial, BlasMode::kHermitian> {
```
**EN:** This block declares or specializes `DefaultSymmComplex`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultSymmComplex`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 449-449

```cpp
 449 |   static BlasMode const kBlasMode = BlasMode::kHermitian;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 452-464

```cpp
 452 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 453 |   /// TRMM - with diagonal: alpha * A * B or alpha * B * A
 454 | 	static const DiagType kDiagTypeMma1 = DiagType::kNonUnit;
 455 |   static ComplexTransform const TransformAMma1 = ComplexTransform::kNone; 
 456 |   static ComplexTransform const TransformBMma1 = ComplexTransform::kNone; 
 457 |   using Mma1 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 458 |       ElementA, LayoutA, 
 459 |       ElementB, LayoutB, 
 460 |       kSideModeA, kFillModeA, kDiagTypeMma1, 
 461 |       ElementAccumulator, layout::RowMajor, 
 462 |       arch::OpClassTensorOp, arch::Sm80,
 463 |       ThreadblockShape, WarpShape, InstructionShape,
 464 |       Stages, TransformAMma1, TransformBMma1, Operator, BlasMode::kHermitian>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma1` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma1` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 466-482

```cpp
 466 |   /// Define the threadblock-scoped triagular matrix multiply-accumulate
 467 |   /// TRMM - withOUT diagonal - with conjugate transpose: alpha * AT * B or alpha * B * AT
 468 | 	static const DiagType kDiagTypeMma2 = DiagType::kZero;
 469 |   using LayoutAMma2 = typename platform::conditional<
 470 |                                 (kSideModeA == SideMode::kLeft), 
 471 |                                 typename layout::LayoutTranspose<LayoutA>::type, 
 472 |                                 LayoutA
 473 |                               >::type;
 474 |   using LayoutBMma2 = typename platform::conditional<
 475 |                                 (kSideModeA == SideMode::kLeft), 
 476 |                                 LayoutB, 
 477 |                                 typename layout::LayoutTranspose<LayoutB>::type
 478 |                               >::type;
 479 |   static ComplexTransform const TransformAMma2 = (kSideModeA == SideMode::kLeft) ? 
 480 |                                               ComplexTransform::kConjugate : ComplexTransform::kNone;
 481 |   static ComplexTransform const TransformBMma2 = (kSideModeA == SideMode::kLeft) ? 
 482 |                                               ComplexTransform::kNone : ComplexTransform::kConjugate;
```
**EN:** This alias block derives concise type names `LayoutAMma2`, `LayoutBMma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `LayoutAMma2`, `LayoutBMma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 484-491

```cpp
 484 | 	using Mma2 = typename cutlass::gemm::threadblock::DefaultMultistageTrmmComplex<
 485 | 			ElementA, LayoutAMma2, 
 486 | 			ElementB, LayoutBMma2, 
 487 | 			kSideModeA, InvertFillMode<kFillModeA>::mode, kDiagTypeMma2, 
 488 | 			ElementAccumulator, layout::RowMajor, 
 489 | 			arch::OpClassTensorOp, arch::Sm80,
 490 | 			ThreadblockShape, WarpShape, InstructionShape,
 491 | 			Stages, TransformAMma2, TransformBMma2, Operator>::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma2` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma2` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 493-497

```cpp
 493 |   /// Define the epilogue
 494 |   using Epilogue =
 495 |       typename cutlass::epilogue::threadblock::DefaultEpilogueComplexTensorOp<
 496 |           ThreadblockShape, typename Mma1::Operator, 1, EpilogueOutputOp,
 497 |           EpilogueOutputOp::kCount, Operator>::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 499-502

```cpp
 499 |   /// Define the kernel-level Symm operator.
 500 |   using SymmKernel = kernel::SymmUniversal<Mma1, Mma2, Epilogue, ThreadblockSwizzle, kSideModeA, kFillModeA>;
 501 | 
 502 | };
```
**EN:** This alias block derives concise type names `SymmKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `SymmKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 506-508

```cpp
 506 | }  // namespace kernel
 507 | }  // namespace gemm
 508 | }  // namespace cutlass
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

## Key Concepts / 关键概念
- Default kernel composition / 默认内核组合
- Template metaprogramming / 模板元编程
- Symmetric matrix multiply / 对称矩阵乘
- Triangular matrix multiply / 三角矩阵乘
- SM90 architecture tuning / SM90 架构调优

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/blas3.h`, `cutlass/layout/matrix.h`, `cutlass/arch/wmma.h`, `cutlass/epilogue/threadblock/epilogue.h`, `cutlass/epilogue/thread/linear_combination.h`, `cutlass/gemm/gemm.h`, `cutlass/gemm/kernel/symm_universal.h`, `cutlass/gemm/threadblock/default_mma_core_sm80.h`, `cutlass/gemm/threadblock/default_mma.h`, `cutlass/gemm/threadblock/default_multistage_trmm_complex.h`, `cutlass/gemm/threadblock/default_multistage_mma_complex.h`, `cutlass/gemm/threadblock/threadblock_swizzle.h`, ... (+3 more)
- **Subsystems / 子系统:** Epilogue/output pipeline / Epilogue 输出流水, Threadblock MMA building blocks / 线程块级 MMA 构件
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/symm_universal.h`
