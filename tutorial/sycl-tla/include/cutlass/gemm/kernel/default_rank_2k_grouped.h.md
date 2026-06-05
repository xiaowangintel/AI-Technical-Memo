# default_rank_2k_grouped.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/default_rank_2k_grouped.h`
- **Purpose / 用途 (EN):** Defines the default kernel composition and specialization rules for rank-2k grouped. Briefly, the file comment says: Default kernel-level grouped Rank2K.
- **Purpose / 用途 (CN):** 定义 rank-2k grouped 的默认内核组合与模板特化规则。 文件注释的简要说明是：Default kernel-level grouped Rank2K。
- **Line count / 行数:** 355

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
  34 |       Default kernel-level grouped Rank2K.
  35 | */
```
**EN:** This file-level comment names the header and introduces the high-level role of the kernel component that follows.
**CN:** 这一段文件级注释给出了头文件说明，并概括了后续内核组件的整体作用。

### Lines 37-37

```cpp
  37 | #pragma once
```
**EN:** This directive makes the header idempotent so repeated inclusion does not create duplicate definitions.
**CN:** 该指令让头文件只生效一次，避免重复包含导致的重复定义。

### Lines 39-39

```cpp
  39 | #include "cutlass/cutlass.h"
```
**EN:** This include block imports `cutlass/cutlass.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/cutlass.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 41-43

```cpp
  41 | #include "cutlass/complex.h"
  42 | #include "cutlass/layout/matrix.h"
  43 | #include "cutlass/numeric_types.h"
```
**EN:** This include block imports `cutlass/complex.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/complex.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 45-47

```cpp
  45 | #include "cutlass/gemm/kernel/rank_2k_transpose_operands.h"
  46 | #include "cutlass/gemm/kernel/default_rank_2k.h"
  47 | #include "cutlass/gemm/kernel/default_rank_2k_complex.h"
```
**EN:** This include block imports `cutlass/gemm/kernel/rank_2k_transpose_operands.h`, `cutlass/gemm/kernel/default_rank_2k.h`, `cutlass/gemm/kernel/default_rank_2k_complex.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/kernel/rank_2k_transpose_operands.h`, `cutlass/gemm/kernel/default_rank_2k.h`, `cutlass/gemm/kernel/default_rank_2k_complex.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 51-53

```cpp
  51 | namespace cutlass {
  52 | namespace gemm {
  53 | namespace kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 57-76

```cpp
  57 | template <
  58 |     /// Element type for A matrix operand
  59 |     typename ElementA,
  60 |     /// Layout type for A matrix operand
  61 |     typename LayoutA,
  62 |     /// Complex elementwise transformation on A operand
  63 |     ComplexTransform TransformA,
  64 |     /// Access granularity of A matrix in units of elements
  65 |     int kAlignmentA,
  66 |     /// Element type for B matrix operand
  67 |     typename ElementB,
  68 |     /// Layout type for B matrix operand
  69 |     typename LayoutB,
  70 |     /// Complex elementwise transformation on B operand
  71 |     ComplexTransform TransformB,
  72 |     /// Access granularity of B matrix in units of elements
  73 |     int kAlignmentB,
  74 |     /// Element type for C and D matrix operands
  75 |     typename ElementC,
  76 |     /// Layout type for C and D matrix operands
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 77-96

```cpp
  77 |     typename LayoutC,
  78 |     /// Fill Mode for C (kLower or kUpper)
  79 |     FillMode FillModeC,
  80 |     /// Element type for internal accumulation
  81 |     typename ElementAccumulator,
  82 |     /// Operator class tag
  83 |     typename OperatorClass,
  84 |     /// Tag indicating architecture to tune for
  85 |     typename ArchTag,
  86 |     /// Threadblock-level tile size (concept: GemmShape)
  87 |     typename ThreadblockShape,
  88 |     /// Warp-level tile size (concept: GemmShape)
  89 |     typename WarpShape,
  90 |     /// Warp-level tile size (concept: GemmShape)
  91 |     typename InstructionShape,
  92 |     /// Epilogue output operator
  93 |     typename EpilogueOutputOp,
  94 |     /// Threadblock-level swizzling operator
  95 |     typename ThreadblockSwizzle,
  96 |     /// Number of stages used in the pipelined mainloop
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 97-112

```cpp
  97 |     int Stages,
  98 |     /// Operation performed by GEMM
  99 |     typename Operator,
 100 |     /// Blas3 computation mode
 101 |     BlasMode BlasMode_ = BlasMode::kSymmetric,
 102 |     /// Whether the schedule of problems to visit has been precomputed
 103 |     GroupScheduleMode GroupScheduleMode_ = GroupScheduleMode::kDeviceOnly,
 104 |     ///
 105 |     typename Enable = void
 106 |     >
 107 | struct DefaultRank2KGrouped;
 108 | 
 109 | /////////////////////////////////////////////////////////////////////////////////////////////////
 110 | //
 111 | // Real-valued grouped Rank2K
 112 | //
```
**EN:** This block declares or specializes `DefaultRank2KGrouped`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KGrouped`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 114-133

```cpp
 114 | template <
 115 |     /// Element type for A matrix operand
 116 |     typename ElementA,
 117 |     /// Layout type for A matrix operand
 118 |     typename LayoutA,
 119 |     /// Complex elementwise transformation on A operand
 120 |     ComplexTransform TransformA,
 121 |     /// Access granularity of A matrix in units of elements
 122 |     int kAlignmentA,
 123 |     /// Element type for B matrix operand
 124 |     typename ElementB,
 125 |     /// Layout type for B matrix operand
 126 |     typename LayoutB,
 127 |     /// Complex elementwise transformation on B operand
 128 |     ComplexTransform TransformB,
 129 |     /// Access granularity of B matrix in units of elements
 130 |     int kAlignmentB,
 131 |     /// Element type for C and D matrix operands
 132 |     typename ElementC,
 133 |     /// Layout type for C and D matrix operands
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 134-153

```cpp
 134 |     typename LayoutC,
 135 |     /// Fill Mode for C (kLower or kUpper)
 136 |     FillMode FillModeC,
 137 |     /// Element type for internal accumulation
 138 |     typename ElementAccumulator,
 139 |     /// Operator class tag
 140 |     typename OperatorClass,
 141 |     /// Tag indicating architecture to tune for
 142 |     typename ArchTag,
 143 |     /// Threadblock-level tile size (concept: GemmShape)
 144 |     typename ThreadblockShape,
 145 |     /// Warp-level tile size (concept: GemmShape)
 146 |     typename WarpShape,
 147 |     /// Warp-level tile size (concept: GemmShape)
 148 |     typename InstructionShape,
 149 |     /// Epilogue output operator
 150 |     typename EpilogueOutputOp,
 151 |     /// Threadblock-level swizzling operator
 152 |     typename ThreadblockSwizzle,
 153 |     /// Number of stages used in the pipelined mainloop
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 154-171

```cpp
 154 |     int Stages,
 155 |     /// Operation performed by GEMM
 156 |     typename Operator,
 157 |     /// Blas3 computation mode
 158 |     BlasMode BlasMode_,
 159 |     /// Whether the schedule of problems to visit has been precomputed
 160 |     GroupScheduleMode GroupScheduleMode_
 161 |     >
 162 | struct DefaultRank2KGrouped<ElementA, LayoutA, TransformA, kAlignmentA,
 163 |           ElementB, LayoutB, TransformB, kAlignmentB,
 164 |           ElementC, LayoutC,
 165 |           FillModeC, ElementAccumulator, OperatorClass, ArchTag, ThreadblockShape,
 166 |           WarpShape, InstructionShape, EpilogueOutputOp,
 167 |           ThreadblockSwizzle, Stages, Operator, BlasMode_, GroupScheduleMode_,
 168 |           typename platform::enable_if< ! cutlass::is_complex<ElementAccumulator>::value>::type
 169 | > {
 170 |   // If true, we must construct a 'transposed-and-exchanged' Rank2K operator.
 171 |   static bool const kInternalTranspose = platform::is_same<LayoutC, layout::ColumnMajor>::value;
```
**EN:** This block declares or specializes `DefaultRank2KGrouped`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KGrouped`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 173-185

```cpp
 173 |   using MapArguments = kernel::detail::Rank2KMapArguments<
 174 |     ElementA,
 175 |     LayoutA,
 176 |     TransformA,
 177 |     kAlignmentA,
 178 |     ElementB,
 179 |     LayoutB,
 180 |     TransformB,
 181 |     kAlignmentB,
 182 |     LayoutC,
 183 |     FillModeC,
 184 |     kInternalTranspose
 185 |   >;
```
**EN:** This alias block derives concise type names `MapArguments` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `MapArguments` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 187-210

```cpp
 187 |   // Define the default grouped Rank2K kernel
 188 |   using DefaultRank2Kkernel = typename kernel::DefaultRank2K<
 189 |     typename MapArguments::ElementA,
 190 |     typename MapArguments::LayoutA,
 191 |     MapArguments::kAlignmentA,
 192 |     typename MapArguments::ElementB,
 193 |     typename MapArguments::LayoutB,
 194 |     MapArguments::kAlignmentB,
 195 |     ElementC,
 196 |     typename MapArguments::LayoutC,
 197 |     MapArguments::kFillModeC,
 198 |     ElementAccumulator,
 199 |     OperatorClass,
 200 |     ArchTag,
 201 |     ThreadblockShape,
 202 |     WarpShape,
 203 |     InstructionShape,
 204 |     EpilogueOutputOp,
 205 |     ThreadblockSwizzle,
 206 |     Stages,
 207 |     false,                  // SplitKSerial
 208 |     Operator,
 209 |     BlasMode_
 210 |   >::Rank2Kkernel;
```
**EN:** This alias block derives concise type names `DefaultRank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `DefaultRank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 212-230

```cpp
 212 |   /// Define the kernel in terms of the default kernel
 213 |   using Rank2Kkernel = kernel::Rank2KGrouped<
 214 |     typename DefaultRank2Kkernel::Mma1,
 215 |     typename DefaultRank2Kkernel::Mma2,
 216 |     typename DefaultRank2Kkernel::Epilogue,
 217 |     ThreadblockSwizzle,
 218 |     TransformA,
 219 |     TransformB,
 220 |     DefaultRank2Kkernel::kFillModeC,
 221 |     DefaultRank2Kkernel::kBlasMode,
 222 |     GroupScheduleMode_,
 223 |     kInternalTranspose
 224 |   >;
 225 | };
 226 | 
 227 | /////////////////////////////////////////////////////////////////////////////////////////////////
 228 | //
 229 | // Complex-valued grouped Rank2K
 230 | //
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 232-251

```cpp
 232 | template <
 233 |     /// Element type for A matrix operand
 234 |     typename ElementA,
 235 |     /// Layout type for A matrix operand
 236 |     typename LayoutA,
 237 |     /// Complex elementwise transformation on A operand
 238 |     ComplexTransform TransformA,
 239 |     /// Access granularity of A matrix in units of elements
 240 |     int kAlignmentA,
 241 |     /// Element type for B matrix operand
 242 |     typename ElementB,
 243 |     /// Layout type for B matrix operand
 244 |     typename LayoutB,
 245 |     /// Complex elementwise transformation on B operand
 246 |     ComplexTransform TransformB,
 247 |     /// Access granularity of B matrix in units of elements
 248 |     int kAlignmentB,
 249 |     /// Element type for C and D matrix operands
 250 |     typename ElementC,
 251 |     /// Layout type for C and D matrix operands
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 252-271

```cpp
 252 |     typename LayoutC,
 253 |     /// Fill Mode for C (kLower or kUpper)
 254 |     FillMode FillModeC,
 255 |     /// Element type for internal accumulation
 256 |     typename ElementAccumulator,
 257 |     /// Operator class tag
 258 |     typename OperatorClass,
 259 |     /// Tag indicating architecture to tune for
 260 |     typename ArchTag,
 261 |     /// Threadblock-level tile size (concept: GemmShape)
 262 |     typename ThreadblockShape,
 263 |     /// Warp-level tile size (concept: GemmShape)
 264 |     typename WarpShape,
 265 |     /// Warp-level tile size (concept: GemmShape)
 266 |     typename InstructionShape,
 267 |     /// Epilogue output operator
 268 |     typename EpilogueOutputOp,
 269 |     /// Threadblock-level swizzling operator
 270 |     typename ThreadblockSwizzle,
 271 |     /// Number of stages used in the pipelined mainloop
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 272-289

```cpp
 272 |     int Stages,
 273 |     /// Operation performed by GEMM
 274 |     typename Operator,
 275 |     /// Blas3 computation mode
 276 |     BlasMode BlasMode_,
 277 |     /// Whether the schedule of problems to visit has been precomputed
 278 |     GroupScheduleMode GroupScheduleMode_
 279 |     >
 280 | struct DefaultRank2KGrouped<ElementA, LayoutA, TransformA, kAlignmentA,
 281 |           ElementB, LayoutB, TransformB, kAlignmentB,
 282 |           ElementC, LayoutC,
 283 |           FillModeC, ElementAccumulator, OperatorClass, ArchTag, ThreadblockShape,
 284 |           WarpShape, InstructionShape, EpilogueOutputOp,
 285 |           ThreadblockSwizzle, Stages, Operator, BlasMode_, GroupScheduleMode_,
 286 |           typename platform::enable_if<cutlass::is_complex<ElementAccumulator>::value>::type
 287 | > {
 288 |   // If true, we must construct a 'transposed-and-exchanged' Rank2K operator.
 289 |   static bool const kInternalTranspose = platform::is_same<LayoutC, layout::ColumnMajor>::value;
```
**EN:** This block declares or specializes `DefaultRank2KGrouped`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultRank2KGrouped`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 291-303

```cpp
 291 |   using MapArguments = kernel::detail::Rank2KMapArguments<
 292 |     ElementA,
 293 |     LayoutA,
 294 |     TransformA,
 295 |     kAlignmentA,
 296 |     ElementB,
 297 |     LayoutB,
 298 |     TransformB,
 299 |     kAlignmentB,
 300 |     LayoutC,
 301 |     FillModeC,
 302 |     kInternalTranspose
 303 |   >;
```
**EN:** This alias block derives concise type names `MapArguments` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `MapArguments` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 305-328

```cpp
 305 |   // Define the default grouped Rank2K kernel
 306 |   using DefaultRank2Kkernel = typename kernel::DefaultRank2KComplex<
 307 |     typename MapArguments::ElementA,
 308 |     typename MapArguments::LayoutA,
 309 |     typename MapArguments::ElementB,
 310 |     typename MapArguments::LayoutB,
 311 |     ElementC,
 312 |     typename MapArguments::LayoutC,
 313 |     MapArguments::kFillModeC,
 314 |     ElementAccumulator,
 315 |     OperatorClass,
 316 |     ArchTag,
 317 |     ThreadblockShape,
 318 |     WarpShape,
 319 |     InstructionShape,
 320 |     EpilogueOutputOp,
 321 |     ThreadblockSwizzle,
 322 |     Stages,
 323 |     MapArguments::kTransformA,
 324 |     MapArguments::kTransformB,
 325 |     Operator,
 326 |     false,                  // SplitKSerial
 327 |     BlasMode_
 328 |   >::Rank2Kkernel;
```
**EN:** This alias block derives concise type names `DefaultRank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `DefaultRank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 330-347

```cpp
 330 |   /// Define the kernel in terms of the default kernel
 331 |   /// Pass through the user-provided TransformA and TransformB so as to
 332 |   /// correctly set public-facing TransformA and TransformB in kernel::Rank2KGrouped.
 333 |   /// This is needed because kernel::DefaultRank2KComplex may change TransformA and
 334 |   /// TransformB that become template arguments to Mma1 and Mma2.
 335 |   using Rank2Kkernel = kernel::Rank2KGrouped<
 336 |     typename DefaultRank2Kkernel::Mma1,
 337 |     typename DefaultRank2Kkernel::Mma2,
 338 |     typename DefaultRank2Kkernel::Epilogue,
 339 |     ThreadblockSwizzle,
 340 |     TransformA,
 341 |     TransformB,
 342 |     DefaultRank2Kkernel::kFillModeC,
 343 |     DefaultRank2Kkernel::kBlasMode,
 344 |     GroupScheduleMode_,
 345 |     kInternalTranspose
 346 |   >;
 347 | };
```
**EN:** This alias block derives concise type names `Rank2Kkernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Rank2Kkernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 351-353

```cpp
 351 | }  // namespace kernel
 352 | }  // namespace gemm
 353 | }  // namespace cutlass
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

## Key Concepts / 关键概念
- Default kernel composition / 默认内核组合
- Template metaprogramming / 模板元编程
- SFINAE-based specialization / 基于 SFINAE 的特化
- Grouped problem handling / 分组问题处理
- Rank-2k update / Rank-2k 更新
- Symmetric matrix multiply / 对称矩阵乘
- Runtime argument packing / 运行时参数打包

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/cutlass.h`, `cutlass/complex.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, `cutlass/gemm/kernel/rank_2k_transpose_operands.h`, `cutlass/gemm/kernel/default_rank_2k.h`, `cutlass/gemm/kernel/default_rank_2k_complex.h`
- **Subsystems / 子系统:** Grouped problem support / 分组问题支持
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/rank_2k_transpose_operands.h`, `cutlass/gemm/kernel/default_rank_2k.h`, `cutlass/gemm/kernel/default_rank_2k_complex.h`
