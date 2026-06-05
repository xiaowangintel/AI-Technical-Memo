# default_gemm_planar_complex_universal.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/default_gemm_planar_complex_universal.h`
- **Purpose / 用途 (EN):** Defines the default kernel composition and specialization rules for GEMM planar complex universal. Briefly, the file comment says: Default kernel-level GEMM definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue.
- **Purpose / 用途 (CN):** 定义 GEMM planar complex universal 的默认内核组合与模板特化规则。 文件注释的简要说明是：Default kernel-level GEMM definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue。
- **Line count / 行数:** 352

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
  34 |       Default kernel-level GEMM definitions combine threadblock-scoped matrix multiply-add with
  35 |       the appropriate threadblock-scoped epilogue.
```
**EN:** This file-level comment names the header and introduces the high-level role of the kernel component that follows.
**CN:** 这一段文件级注释给出了头文件说明，并概括了后续内核组件的整体作用。

### Lines 37-39

```cpp
  37 |       Note, CUTLASS epilogues universally target row-major outputs. Column-major outputs are
  38 |       accommodated by exchanging A and B operands and assuming transposed layouts. Partial
  39 |       specializations here choose 'device::GemmTransposed' to implement this functionality.
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 43-43

```cpp
  43 | #pragma once
```
**EN:** This directive makes the header idempotent so repeated inclusion does not create duplicate definitions.
**CN:** 该指令让头文件只生效一次，避免重复包含导致的重复定义。

### Lines 45-45

```cpp
  45 | #include "cutlass/cutlass.h"
```
**EN:** This include block imports `cutlass/cutlass.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/cutlass.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 47-49

```cpp
  47 | #include "cutlass/complex.h"
  48 | #include "cutlass/layout/matrix.h"
  49 | #include "cutlass/numeric_types.h"
```
**EN:** This include block imports `cutlass/complex.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/complex.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 51-54

```cpp
  51 | #include "cutlass/gemm/kernel/gemm_planar_complex.h"
  52 | #include "cutlass/gemm/kernel/gemm_planar_complex_array.h"
  53 | #include "cutlass/gemm/kernel/default_gemm.h"
  54 | #include "cutlass/gemm/kernel/default_gemm_complex.h"
```
**EN:** This include block imports `cutlass/gemm/kernel/gemm_planar_complex.h`, `cutlass/gemm/kernel/gemm_planar_complex_array.h`, `cutlass/gemm/kernel/default_gemm.h`, `cutlass/gemm/kernel/default_gemm_complex.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/kernel/gemm_planar_complex.h`, `cutlass/gemm/kernel/gemm_planar_complex_array.h`, `cutlass/gemm/kernel/default_gemm.h`, `cutlass/gemm/kernel/default_gemm_complex.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 56-58

```cpp
  56 | #include "cutlass/epilogue/threadblock/default_epilogue_planar_complex.h"
  57 | #include "cutlass/gemm/threadblock/default_mma_planar_complex_pipelined.h"
  58 | #include "cutlass/gemm/threadblock/default_mma_planar_complex_multistage.h" 
```
**EN:** This include block imports `cutlass/epilogue/threadblock/default_epilogue_planar_complex.h`, `cutlass/gemm/threadblock/default_mma_planar_complex_pipelined.h`, `cutlass/gemm/threadblock/default_mma_planar_complex_multistage.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/epilogue/threadblock/default_epilogue_planar_complex.h`, `cutlass/gemm/threadblock/default_mma_planar_complex_pipelined.h`, `cutlass/gemm/threadblock/default_mma_planar_complex_multistage.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 62-64

```cpp
  62 | namespace cutlass {
  63 | namespace gemm {
  64 | namespace kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 68-87

```cpp
  68 | template <
  69 |     /// Element type for A matrix operand
  70 |     typename ElementA,
  71 |     /// Layout type for A matrix operand
  72 |     typename LayoutA,
  73 |     /// Complex elementwise transformation on A operand
  74 |     ComplexTransform TransformA,
  75 |     /// Access granularity of A matrix in units of elements
  76 |     int kAlignmentA,
  77 |     /// Element type for B matrix operand
  78 |     typename ElementB,
  79 |     /// Layout type for B matrix operand
  80 |     typename LayoutB,
  81 |     /// Complex elementwise transformation on B operand
  82 |     ComplexTransform TransformB,
  83 |     /// Access granularity of B matrix in units of elements
  84 |     int kAlignmentB,
  85 |     /// Element type for C and D matrix operands
  86 |     typename ElementC,
  87 |     /// Layout type for C and D matrix operands
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 88-107

```cpp
  88 |     typename LayoutC,
  89 |     /// Element type for internal accumulation
  90 |     typename ElementAccumulator,
  91 |     /// Operator class tag
  92 |     typename OperatorClass,
  93 |     /// Tag indicating architecture to tune for
  94 |     typename ArchTag,
  95 |     /// Threadblock-level tile size (concept: GemmShape)
  96 |     typename ThreadblockShape,
  97 |     /// Warp-level tile size (concept: GemmShape)
  98 |     typename WarpShape,
  99 |     /// Warp-level tile size (concept: GemmShape)
 100 |     typename InstructionShape,
 101 |     /// Epilogue output operator
 102 |     typename EpilogueOutputOp,
 103 |     /// Threadblock-level swizzling operator
 104 |     typename ThreadblockSwizzle,
 105 |     /// Number of stages used in the pipelined mainloop
 106 |     int Stages,
 107 |     /// Math operation performed by GEMM (e.g. arch::OpMultiplyAdd)
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 108-112

```cpp
 108 |     typename Operator,
 109 |     /// Conditional enabling to switch between stages
 110 |     typename Enable = void
 111 |   >
 112 | struct DefaultGemmPlanarComplexUniversal;
```
**EN:** This block declares or specializes `DefaultGemmPlanarComplexUniversal`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultGemmPlanarComplexUniversal`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 116-135

```cpp
 116 | /// Partial specialization for pipelined mainloop
 117 | template <
 118 |     /// Element type for A matrix operand
 119 |     typename ElementA,
 120 |     /// Layout type for A matrix operand
 121 |     typename LayoutA,
 122 |     /// Complex elementwise transformation on A operand
 123 |     ComplexTransform TransformA,
 124 |     /// Access granularity of A matrix in units of elements
 125 |     int kAlignmentA,
 126 |     /// Element type for B matrix operand
 127 |     typename ElementB,
 128 |     /// Layout type for B matrix operand
 129 |     typename LayoutB,
 130 |     /// Complex elementwise transformation on B operand
 131 |     ComplexTransform TransformB,
 132 |     /// Access granularity of B matrix in units of elements
 133 |     int kAlignmentB,
 134 |     /// Element type for C and D matrix operands
 135 |     typename ElementC,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 136-155

```cpp
 136 |     /// Layout type for C and D matrix operands
 137 |     typename LayoutC,
 138 |     /// Element type for internal accumulation
 139 |     typename ElementAccumulator,
 140 |     /// Operator class tag
 141 |     typename OperatorClass,
 142 |     /// Tag indicating architecture to tune for
 143 |     typename ArchTag,
 144 |     /// Threadblock-level tile size (concept: GemmShape)
 145 |     typename ThreadblockShape,
 146 |     /// Warp-level tile size (concept: GemmShape)
 147 |     typename WarpShape,
 148 |     /// Warp-level tile size (concept: GemmShape)
 149 |     typename InstructionShape,
 150 |     /// Epilogue output operator
 151 |     typename EpilogueOutputOp,
 152 |     /// Threadblock-level swizzling operator
 153 |     typename ThreadblockSwizzle,
 154 |     /// Number of stages used in the pipelined mainloop
 155 |     int Stages,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 156-175

```cpp
 156 |     /// Operation performed by GEMM
 157 |     typename Operator
 158 |   >
 159 | struct DefaultGemmPlanarComplexUniversal<
 160 |   ElementA,
 161 |   LayoutA,
 162 |   TransformA,
 163 |   kAlignmentA,
 164 |   ElementB,
 165 |   LayoutB,
 166 |   TransformB,
 167 |   kAlignmentB,
 168 |   ElementC,
 169 |   LayoutC,
 170 |   ElementAccumulator,
 171 |   OperatorClass,
 172 |   ArchTag,
 173 |   ThreadblockShape,
 174 |   WarpShape,
 175 |   InstructionShape,
```
**EN:** This block declares or specializes `DefaultGemmPlanarComplexUniversal`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultGemmPlanarComplexUniversal`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 176-181

```cpp
 176 |   EpilogueOutputOp,
 177 |   ThreadblockSwizzle,
 178 |   Stages,
 179 |   Operator,
 180 |   typename platform::enable_if<(Stages <= 2)>::type 
 181 | > {
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 183-202

```cpp
 183 |   /// Define planar complex valued variants instead
 184 |   using Mma = typename gemm::threadblock::DefaultMmaPlanarComplexPipelined<
 185 |     ElementA,
 186 |     LayoutA,
 187 |     kAlignmentA,
 188 |     ElementB,
 189 |     LayoutB,
 190 |     kAlignmentB,
 191 |     ElementAccumulator,
 192 |     LayoutC,
 193 |     OperatorClass,
 194 |     ArchTag,
 195 |     ThreadblockShape,
 196 |     WarpShape,
 197 |     InstructionShape,
 198 |     Stages,
 199 |     TransformA,
 200 |     TransformB,
 201 |     Operator
 202 |   >::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 204-213

```cpp
 204 |   /// Planar complex epilogue
 205 |   using Epilogue = typename epilogue::threadblock::DefaultEpiloguePlanarComplex<
 206 |     ThreadblockShape,
 207 |     typename Mma::Policy::Operator,
 208 |     OperatorClass,
 209 |     ArchTag,
 210 |     ThreadblockShape::kK / WarpShape::kK,
 211 |     EpilogueOutputOp,
 212 |     EpilogueOutputOp::kCount  
 213 |   >::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 215-220

```cpp
 215 |   /// Define the kernel in terms of the default kernel
 216 |   using GemmKernel = kernel::GemmPlanarComplex<
 217 |     Mma,
 218 |     Epilogue, 
 219 |     ThreadblockSwizzle
 220 |   >;
```
**EN:** This alias block derives concise type names `GemmKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `GemmKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 222-228

```cpp
 222 |   // Array variant
 223 |   using GemmArrayKernel = kernel::GemmPlanarComplexArray<
 224 |     Mma,
 225 |     Epilogue,
 226 |     ThreadblockSwizzle
 227 |   >;
 228 | };
```
**EN:** This alias block derives concise type names `GemmArrayKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `GemmArrayKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 232-251

```cpp
 232 | /// Partial specialization for multiple pipeline stages.
 233 | template <
 234 |     /// Element type for A matrix operand
 235 |     typename ElementA,
 236 |     /// Layout type for A matrix operand
 237 |     typename LayoutA,
 238 |     /// Complex elementwise transformation on A operand
 239 |     ComplexTransform TransformA,
 240 |     /// Access granularity of A matrix in units of elements
 241 |     int kAlignmentA,
 242 |     /// Element type for B matrix operand
 243 |     typename ElementB,
 244 |     /// Layout type for B matrix operand
 245 |     typename LayoutB,
 246 |     /// Complex elementwise transformation on B operand
 247 |     ComplexTransform TransformB,
 248 |     /// Access granularity of B matrix in units of elements
 249 |     int kAlignmentB,
 250 |     /// Element type for C and D matrix operands
 251 |     typename ElementC,
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 252-271

```cpp
 252 |     /// Layout type for C and D matrix operands
 253 |     typename LayoutC,
 254 |     /// Element type for internal accumulation
 255 |     typename ElementAccumulator,
 256 |     /// Operator class tag
 257 |     typename OperatorClass,
 258 |     /// Tag indicating architecture to tune for
 259 |     typename ArchTag,
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
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 272-291

```cpp
 272 |     /// Operation performed by GEMM
 273 |     typename Operator
 274 |   >
 275 | struct DefaultGemmPlanarComplexUniversal<
 276 |   ElementA,
 277 |   LayoutA,
 278 |   TransformA,
 279 |   kAlignmentA,
 280 |   ElementB,
 281 |   LayoutB,
 282 |   TransformB,
 283 |   kAlignmentB,
 284 |   ElementC,
 285 |   LayoutC,
 286 |   ElementAccumulator,
 287 |   OperatorClass,
 288 |   ArchTag,
 289 |   ThreadblockShape,
 290 |   WarpShape,
 291 |   InstructionShape,
```
**EN:** This block declares or specializes `DefaultGemmPlanarComplexUniversal`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DefaultGemmPlanarComplexUniversal`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 292-297

```cpp
 292 |   EpilogueOutputOp,
 293 |   ThreadblockSwizzle,
 294 |   Stages,
 295 |   Operator,
 296 |   typename platform::enable_if<(Stages > 2)>::type 
 297 | > {
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 299-318

```cpp
 299 |   /// Define planar complex valued variants instead
 300 |   using Mma = typename gemm::threadblock::DefaultMmaPlanarComplexMultistage<
 301 |     ElementA,
 302 |     LayoutA,
 303 |     kAlignmentA,
 304 |     ElementB,
 305 |     LayoutB,
 306 |     kAlignmentB,
 307 |     ElementAccumulator,
 308 |     LayoutC,
 309 |     OperatorClass,
 310 |     ArchTag,
 311 |     ThreadblockShape,
 312 |     WarpShape,
 313 |     InstructionShape,
 314 |     Stages,
 315 |     TransformA,
 316 |     TransformB,
 317 |     Operator
 318 |   >::ThreadblockMma;
```
**EN:** This alias block derives concise type names `Mma` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 320-329

```cpp
 320 |   /// Planar complex epilogue
 321 |   using Epilogue = typename epilogue::threadblock::DefaultEpiloguePlanarComplex<
 322 |     ThreadblockShape,
 323 |     typename Mma::Policy::Operator,
 324 |     OperatorClass,
 325 |     ArchTag,
 326 |     ThreadblockShape::kK / WarpShape::kK,
 327 |     EpilogueOutputOp,
 328 |     EpilogueOutputOp::kCount  
 329 |   >::Epilogue;
```
**EN:** This alias block derives concise type names `Epilogue` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Epilogue` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 331-336

```cpp
 331 |   /// Define the kernel in terms of the default kernel
 332 |   using GemmKernel = kernel::GemmPlanarComplex<
 333 |     Mma,
 334 |     Epilogue, 
 335 |     ThreadblockSwizzle
 336 |   >;
```
**EN:** This alias block derives concise type names `GemmKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `GemmKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 338-344

```cpp
 338 |   // Array variant
 339 |   using GemmArrayKernel = kernel::GemmPlanarComplexArray<
 340 |     Mma,
 341 |     Epilogue,
 342 |     ThreadblockSwizzle
 343 |   >;
 344 | };
```
**EN:** This alias block derives concise type names `GemmArrayKernel` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `GemmArrayKernel` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 348-350

```cpp
 348 | }  // namespace kernel
 349 | }  // namespace gemm
 350 | }  // namespace cutlass
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- SFINAE-based specialization / 基于 SFINAE 的特化
- Default kernel composition / 默认内核组合

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/cutlass.h`, `cutlass/complex.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, `cutlass/gemm/kernel/gemm_planar_complex.h`, `cutlass/gemm/kernel/gemm_planar_complex_array.h`, `cutlass/gemm/kernel/default_gemm.h`, `cutlass/gemm/kernel/default_gemm_complex.h`, `cutlass/epilogue/threadblock/default_epilogue_planar_complex.h`, `cutlass/gemm/threadblock/default_mma_planar_complex_pipelined.h`, `cutlass/gemm/threadblock/default_mma_planar_complex_multistage.h`
- **Subsystems / 子系统:** Epilogue/output pipeline / Epilogue 输出流水, Threadblock MMA building blocks / 线程块级 MMA 构件
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/gemm_planar_complex.h`, `cutlass/gemm/kernel/gemm_planar_complex_array.h`, `cutlass/gemm/kernel/default_gemm.h`, `cutlass/gemm/kernel/default_gemm_complex.h`
