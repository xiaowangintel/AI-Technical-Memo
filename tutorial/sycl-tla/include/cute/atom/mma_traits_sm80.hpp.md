# mma_traits_sm80.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/atom/mma_traits_sm80.hpp`
- Purpose (EN): Defines generic or architecture-specific `MMA_Traits` specializations that describe operand types, tile shapes, thread/value layouts, and fragment storage for MMA instructions.
- 作用 (CN): 定义通用或架构相关的 `MMA_Traits` 特化，用于描述 MMA 指令的操作数类型、tile 形状、线程/值布局以及 fragment 存储方式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```text
    1 | /***************************************************************************************************
    2 |  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 30-31

```text
   30 |  **************************************************************************************************/
   31 | #pragma once
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 33-36

```text
   33 | #include <cute/arch/mma_sm80.hpp>
   34 | #include <cute/atom/mma_traits.hpp>
   35 | #include <cute/layout.hpp>
   36 | #include <cute/numeric/numeric_types.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/arch/mma_sm80.hpp`, `cute/atom/mma_traits.hpp`, `cute/layout.hpp`, `cute/numeric/numeric_types.hpp`.
**CN:** 通过引入 `cute/arch/mma_sm80.hpp`, `cute/atom/mma_traits.hpp`, `cute/layout.hpp`, `cute/numeric/numeric_types.hpp` 为该文件建立头文件依赖。

### Lines 38-39

```text
   38 | namespace cute
   39 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 41-41

```text
   41 | namespace {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 43-56

```text
   43 | // (T32,V1) -> (M8,N8)
   44 | using SM80_8x4      = Layout<Shape <Shape < _4,_8>,_1>,
   45 |                              Stride<Stride< _8,_1>,_0>>;
   46 | // (T32,V2) -> (M8,N8)
   47 | using SM80_8x8_Row  = Layout<Shape <Shape < _4,_8>,_2>,
   48 |                              Stride<Stride<_16,_1>,_8>>;
   49 | // (T32,V4) -> (M8,N16)
   50 | using SM80_8x16_Row = Layout<Shape <Shape < _4,_8>,_4>,
   51 |                              Stride<Stride<_32,_1>,_8>>;
   52 | // (T32,V4) -> (M16,N8)
   53 | using SM80_16x8_Row = Layout<Shape <Shape < _4,_8>,Shape < _2,_2>>,
   54 |                              Stride<Stride<_32,_1>,Stride<_16,_8>>>;
   56 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 58-75

```text
   58 | ///////////////////////////////////////////////////////////////////////////////
   59 | //////////////////////// fp16 = fp16 * fp16 + fp16 ////////////////////////////
   60 | ///////////////////////////////////////////////////////////////////////////////
   62 | template <>
   63 | struct MMA_Traits<SM80_16x8x8_F16F16F16F16_TN>
   64 | {
   65 |   using ValTypeD = half_t;
   66 |   using ValTypeA = half_t;
   67 |   using ValTypeB = half_t;
   68 |   using ValTypeC = half_t;
   69 | 
   70 |   using Shape_MNK = Shape<_16,_8,_8>;
   71 |   using ThrID   = Layout<_32>;
   72 |   using ALayout = SM80_16x8_Row;
   73 |   using BLayout = SM80_8x8_Row;
   74 |   using CLayout = SM80_16x8_Row;
   75 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x8_F16F16F16F16_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x8_F16F16F16F16_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 77-92

```text
   77 | template <>
   78 | struct MMA_Traits<SM80_16x8x16_F16F16F16F16_TN>
   79 | {
   80 |   using ValTypeD = half_t;
   81 |   using ValTypeA = half_t;
   82 |   using ValTypeB = half_t;
   83 |   using ValTypeC = half_t;
   84 | 
   85 |   using Shape_MNK = Shape<_16,_8,_16>;
   86 |   using ThrID   = Layout<_32>;
   87 |   using ALayout = Layout<Shape <Shape < _4,_8>,Shape < _2,_2,  _2>>,
   88 |                          Stride<Stride<_32,_1>,Stride<_16,_8,_128>>>;
   89 |   using BLayout = Layout<Shape <Shape < _4,_8>,Shape <_2, _2>>,
   90 |                          Stride<Stride<_16,_1>,Stride<_8,_64>>>;
   91 |   using CLayout = SM80_16x8_Row;
   92 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_F16F16F16F16_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_F16F16F16F16_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 94-106

```text
   94 | ///////////////////////////////////////////////////////////////////////////////
   95 | //////////////////////// fp32 = fp16 * fp16 + fp32 ////////////////////////////
   96 | ///////////////////////////////////////////////////////////////////////////////
   98 | template <>
   99 | struct MMA_Traits<SM80_16x8x8_F32F16F16F32_TN>
  100 |      : MMA_Traits<SM80_16x8x8_F16F16F16F16_TN>
  101 | {
  102 |   using ValTypeD = float;
  103 |   using ValTypeA = half_t;
  104 |   using ValTypeB = half_t;
  105 |   using ValTypeC = float;
  106 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x8_F32F16F16F32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x8_F32F16F16F32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 108-116

```text
  108 | template <>
  109 | struct MMA_Traits<SM80_16x8x16_F32F16F16F32_TN>
  110 |      : MMA_Traits<SM80_16x8x16_F16F16F16F16_TN>
  111 | {
  112 |   using ValTypeD = float;
  113 |   using ValTypeA = half_t;
  114 |   using ValTypeB = half_t;
  115 |   using ValTypeC = float;
  116 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_F32F16F16F32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_F32F16F16F32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 118-130

```text
  118 | ///////////////////////////////////////////////////////////////////////////////
  119 | //////////////////////// fp32 = bf16 * bf16 + fp32 ////////////////////////////
  120 | ///////////////////////////////////////////////////////////////////////////////
  122 | template <>
  123 | struct MMA_Traits<SM80_16x8x8_F32BF16BF16F32_TN>
  124 |      : MMA_Traits<SM80_16x8x8_F16F16F16F16_TN>
  125 | {
  126 |   using ValTypeD = float;
  127 |   using ValTypeA = bfloat16_t;
  128 |   using ValTypeB = bfloat16_t;
  129 |   using ValTypeC = float;
  130 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x8_F32BF16BF16F32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x8_F32BF16BF16F32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 132-140

```text
  132 | template <>
  133 | struct MMA_Traits<SM80_16x8x16_F32BF16BF16F32_TN>
  134 |      : MMA_Traits<SM80_16x8x16_F16F16F16F16_TN>
  135 | {
  136 |   using ValTypeD = float;
  137 |   using ValTypeA = bfloat16_t;
  138 |   using ValTypeB = bfloat16_t;
  139 |   using ValTypeC = float;
  140 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_F32BF16BF16F32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_F32BF16BF16F32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 142-160

```text
  142 | ///////////////////////////////////////////////////////////////////////////////
  143 | //////////////////////// fp32 = tf32 * tf32 + fp32 ////////////////////////////
  144 | ///////////////////////////////////////////////////////////////////////////////
  146 | template <>
  147 | struct MMA_Traits<SM80_16x8x4_F32TF32TF32F32_TN>
  148 | {
  149 |   using ValTypeD = float;
  150 |   using ValTypeA = cutlass::tfloat32_t;
  151 |   using ValTypeB = cutlass::tfloat32_t;
  152 |   using ValTypeC = float;
  153 | 
  154 |   using Shape_MNK = Shape<_16,_8,_4>;
  155 |   using ThrID   = Layout<_32>;
  156 |   using ALayout = Layout<Shape <Shape < _4,_8>,_2>,
  157 |                          Stride<Stride<_16,_1>,_8>>;
  158 |   using BLayout = SM80_8x4;
  159 |   using CLayout = SM80_16x8_Row;
  160 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x4_F32TF32TF32F32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x4_F32TF32TF32F32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 162-177

```text
  162 | template <>
  163 | struct MMA_Traits<SM80_16x8x8_F32TF32TF32F32_TN>
  164 | {
  165 |   using ValTypeD = float;
  166 |   using ValTypeA = cutlass::tfloat32_t;
  167 |   using ValTypeB = cutlass::tfloat32_t;
  168 |   using ValTypeC = float;
  169 | 
  170 |   using Shape_MNK = Shape<_16,_8,_8>;
  171 |   using ThrID   = Layout<_32>;
  172 |   using ALayout = Layout<Shape <Shape < _4,_8>,Shape <_2, _2>>,
  173 |                          Stride<Stride<_16,_1>,Stride<_8,_64>>>;
  174 |   using BLayout = Layout<Shape <Shape <_4,_8>, _2>,
  175 |                          Stride<Stride<_8,_1>,_32>>;
  176 |   using CLayout = SM80_16x8_Row;
  177 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x8_F32TF32TF32F32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x8_F32TF32TF32F32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 179-196

```text
  179 | ///////////////////////////////////////////////////////////////////////////////
  180 | //////////////////////// fp64 = fp64 * fp64 + fp64 ////////////////////////////
  181 | ///////////////////////////////////////////////////////////////////////////////
  183 | template <>
  184 | struct MMA_Traits<SM80_8x8x4_F64F64F64F64_TN>
  185 | {
  186 |   using ValTypeD = double;
  187 |   using ValTypeA = double;
  188 |   using ValTypeB = double;
  189 |   using ValTypeC = double;
  190 | 
  191 |   using Shape_MNK = Shape<_8,_8,_4>;
  192 |   using ThrID   = Layout<_32>;
  193 |   using ALayout = SM80_8x4;
  194 |   using BLayout = SM80_8x4;
  195 |   using CLayout = SM80_8x8_Row;
  196 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x4_F64F64F64F64_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x4_F64F64F64F64_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 198-207

```text
  198 | // Custom complex fp64 MMA composed of 4 fp64 MMAs -- same layouts
  199 | template <>
  200 | struct MMA_Traits<SM80_8x8x4_C64C64C64C64_TN>
  201 |      : MMA_Traits<SM80_8x8x4_F64F64F64F64_TN>
  202 | {
  203 |   using ValTypeD = complex<double>;
  204 |   using ValTypeA = complex<double>;
  205 |   using ValTypeB = complex<double>;
  206 |   using ValTypeC = complex<double>;
  207 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x4_C64C64C64C64_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x4_C64C64C64C64_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 209-218

```text
  209 | // Custom complex fp64 MMA composed of 3 fp64 MMAs -- same layouts
  210 | template <>
  211 | struct MMA_Traits<SM80_8x8x4_GC64C64C64GC64_TN>
  212 |      : MMA_Traits<SM80_8x8x4_F64F64F64F64_TN>
  213 | {
  214 |   using ValTypeD = typename SM80_8x8x4_GC64C64C64GC64_TN::GaussComplex;
  215 |   using ValTypeA = complex<double>;
  216 |   using ValTypeB = complex<double>;
  217 |   using ValTypeC = typename SM80_8x8x4_GC64C64C64GC64_TN::GaussComplex;
  218 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x4_GC64C64C64GC64_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x4_GC64C64C64GC64_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 220-237

```text
  220 | ///////////////////////////////////////////////////////////////////////////////
  221 | /////////////////////////// s32 = s8 * s8 + s32 ///////////////////////////////
  222 | ///////////////////////////////////////////////////////////////////////////////
  224 | template <>
  225 | struct MMA_Traits<SM80_8x8x16_S32S8S8S32_TN>
  226 | {
  227 |   using ValTypeD = int32_t;
  228 |   using ValTypeA = int8_t;
  229 |   using ValTypeB = int8_t;
  230 |   using ValTypeC = int32_t;
  231 | 
  232 |   using Shape_MNK = Shape<_8,_8,_16>;
  233 |   using ThrID   = Layout<_32>;
  234 |   using ALayout = SM80_8x16_Row;
  235 |   using BLayout = SM80_8x16_Row;
  236 |   using CLayout = SM80_8x8_Row;
  237 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32S8S8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32S8S8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 239-241

```text
  239 | template <>
  240 | struct MMA_Traits<SM80_8x8x16_S32S8S8S32_TN_SATURATE>
  241 |      : MMA_Traits<SM80_8x8x16_S32S8S8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32S8S8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32S8S8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 243-257

```text
  243 | template <>
  244 | struct MMA_Traits<SM80_16x8x16_S32S8S8S32_TN>
  245 | {
  246 |   using ValTypeD = int32_t;
  247 |   using ValTypeA = int8_t;
  248 |   using ValTypeB = int8_t;
  249 |   using ValTypeC = int32_t;
  250 | 
  251 |   using Shape_MNK = Shape<_16,_8,_16>;
  252 |   using ThrID   = Layout<_32>;
  253 |   using ALayout = Layout<Shape <Shape < _4,_8>,Shape < _4,_2>>,
  254 |                          Stride<Stride<_64,_1>,Stride<_16,_8>>>;
  255 |   using BLayout = SM80_8x16_Row;
  256 |   using CLayout = SM80_16x8_Row;
  257 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32S8S8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32S8S8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 259-261

```text
  259 | template <>
  260 | struct MMA_Traits<SM80_16x8x16_S32S8S8S32_TN_SATURATE>
  261 |      : MMA_Traits<SM80_16x8x16_S32S8S8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32S8S8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32S8S8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 263-278

```text
  263 | template <>
  264 | struct MMA_Traits<SM80_16x8x32_S32S8S8S32_TN>
  265 | {
  266 |   using ValTypeD = int32_t;
  267 |   using ValTypeA = int8_t;
  268 |   using ValTypeB = int8_t;
  269 |   using ValTypeC = int32_t;
  270 | 
  271 |   using Shape_MNK = Shape<_16,_8,_32>;
  272 |   using ThrID   = Layout<_32>;
  273 |   using ALayout = Layout<Shape <Shape < _4,_8>,Shape < _4,_2,  _2>>,
  274 |                          Stride<Stride<_64,_1>,Stride<_16,_8,_256>>>;
  275 |   using BLayout = Layout<Shape <Shape < _4,_8>, Shape <_4,  _2>>,
  276 |                          Stride<Stride<_32,_1>, Stride<_8,_128>>>;
  277 |   using CLayout = SM80_16x8_Row;
  278 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S8S8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S8S8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 280-282

```text
  280 | template <>
  281 | struct MMA_Traits<SM80_16x8x32_S32S8S8S32_TN_SATURATE>
  282 |      : MMA_Traits<SM80_16x8x32_S32S8S8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S8S8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S8S8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 284-296

```text
  284 | ///////////////////////////////////////////////////////////////////////////////
  285 | /////////////////////////// s32 = s8 * u8 + s32 ///////////////////////////////
  286 | ///////////////////////////////////////////////////////////////////////////////
  288 | template <>
  289 | struct MMA_Traits<SM80_8x8x16_S32S8U8S32_TN>
  290 |      : MMA_Traits<SM80_8x8x16_S32S8S8S32_TN>
  291 | {
  292 |   using ValTypeD = int32_t;
  293 |   using ValTypeA = int8_t;
  294 |   using ValTypeB = uint8_t;
  295 |   using ValTypeC = int32_t;
  296 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32S8U8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32S8U8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 298-300

```text
  298 | template <>
  299 | struct MMA_Traits<SM80_8x8x16_S32S8U8S32_TN_SATURATE>
  300 |      : MMA_Traits<SM80_8x8x16_S32S8U8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32S8U8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32S8U8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 302-310

```text
  302 | template <>
  303 | struct MMA_Traits<SM80_16x8x16_S32S8U8S32_TN>
  304 |      : MMA_Traits<SM80_16x8x16_S32S8S8S32_TN>
  305 | {
  306 |   using ValTypeD = int32_t;
  307 |   using ValTypeA = int8_t;
  308 |   using ValTypeB = uint8_t;
  309 |   using ValTypeC = int32_t;
  310 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32S8U8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32S8U8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 312-314

```text
  312 | template <>
  313 | struct MMA_Traits<SM80_16x8x16_S32S8U8S32_TN_SATURATE>
  314 |      : MMA_Traits<SM80_16x8x16_S32S8U8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32S8U8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32S8U8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 316-324

```text
  316 | template <>
  317 | struct MMA_Traits<SM80_16x8x32_S32S8U8S32_TN>
  318 |      : MMA_Traits<SM80_16x8x32_S32S8S8S32_TN>
  319 | {
  320 |   using ValTypeD = int32_t;
  321 |   using ValTypeA = int8_t;
  322 |   using ValTypeB = uint8_t;
  323 |   using ValTypeC = int32_t;
  324 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S8U8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S8U8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 326-328

```text
  326 | template <>
  327 | struct MMA_Traits<SM80_16x8x32_S32S8U8S32_TN_SATURATE>
  328 |      : MMA_Traits<SM80_16x8x32_S32S8U8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S8U8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S8U8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 330-342

```text
  330 | ///////////////////////////////////////////////////////////////////////////////
  331 | /////////////////////////// s32 = u8 * s8 + s32 ///////////////////////////////
  332 | ///////////////////////////////////////////////////////////////////////////////
  334 | template <>
  335 | struct MMA_Traits<SM80_8x8x16_S32U8S8S32_TN>
  336 |      : MMA_Traits<SM80_8x8x16_S32S8S8S32_TN>
  337 | {
  338 |   using ValTypeD = int32_t;
  339 |   using ValTypeA = uint8_t;
  340 |   using ValTypeB = int8_t;
  341 |   using ValTypeC = int32_t;
  342 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32U8S8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32U8S8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 344-346

```text
  344 | template <>
  345 | struct MMA_Traits<SM80_8x8x16_S32U8S8S32_TN_SATURATE>
  346 |      : MMA_Traits<SM80_8x8x16_S32U8S8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32U8S8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32U8S8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 348-356

```text
  348 | template <>
  349 | struct MMA_Traits<SM80_16x8x16_S32U8S8S32_TN>
  350 |      : MMA_Traits<SM80_16x8x16_S32S8S8S32_TN>
  351 | {
  352 |   using ValTypeD = int32_t;
  353 |   using ValTypeA = uint8_t;
  354 |   using ValTypeB = int8_t;
  355 |   using ValTypeC = int32_t;
  356 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32U8S8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32U8S8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 358-360

```text
  358 | template <>
  359 | struct MMA_Traits<SM80_16x8x16_S32U8S8S32_TN_SATURATE>
  360 |      : MMA_Traits<SM80_16x8x16_S32U8S8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32U8S8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32U8S8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 362-370

```text
  362 | template <>
  363 | struct MMA_Traits<SM80_16x8x32_S32U8S8S32_TN>
  364 |      : MMA_Traits<SM80_16x8x32_S32S8S8S32_TN>
  365 | {
  366 |   using ValTypeD = int32_t;
  367 |   using ValTypeA = uint8_t;
  368 |   using ValTypeB = int8_t;
  369 |   using ValTypeC = int32_t;
  370 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U8S8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U8S8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 372-374

```text
  372 | template <>
  373 | struct MMA_Traits<SM80_16x8x32_S32U8S8S32_TN_SATURATE>
  374 |      : MMA_Traits<SM80_16x8x32_S32U8S8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U8S8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U8S8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 376-388

```text
  376 | ///////////////////////////////////////////////////////////////////////////////
  377 | /////////////////////////// s32 = u8 * u8 + s32 ///////////////////////////////
  378 | ///////////////////////////////////////////////////////////////////////////////
  380 | template <>
  381 | struct MMA_Traits<SM80_8x8x16_S32U8U8S32_TN>
  382 |      : MMA_Traits<SM80_8x8x16_S32S8S8S32_TN>
  383 | {
  384 |   using ValTypeD = int32_t;
  385 |   using ValTypeA = uint8_t;
  386 |   using ValTypeB = uint8_t;
  387 |   using ValTypeC = int32_t;
  388 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32U8U8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32U8U8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 390-392

```text
  390 | template <>
  391 | struct MMA_Traits<SM80_8x8x16_S32U8U8S32_TN_SATURATE>
  392 |      : MMA_Traits<SM80_8x8x16_S32U8U8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x16_S32U8U8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x16_S32U8U8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 394-402

```text
  394 | template <>
  395 | struct MMA_Traits<SM80_16x8x16_S32U8U8S32_TN>
  396 |      : MMA_Traits<SM80_16x8x16_S32S8S8S32_TN>
  397 | {
  398 |   using ValTypeD = int32_t;
  399 |   using ValTypeA = uint8_t;
  400 |   using ValTypeB = uint8_t;
  401 |   using ValTypeC = int32_t;
  402 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32U8U8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32U8U8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 404-406

```text
  404 | template <>
  405 | struct MMA_Traits<SM80_16x8x16_S32U8U8S32_TN_SATURATE>
  406 |      : MMA_Traits<SM80_16x8x16_S32U8U8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x16_S32U8U8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x16_S32U8U8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 408-416

```text
  408 | template <>
  409 | struct MMA_Traits<SM80_16x8x32_S32U8U8S32_TN>
  410 |      : MMA_Traits<SM80_16x8x32_S32S8S8S32_TN>
  411 | {
  412 |   using ValTypeD = int32_t;
  413 |   using ValTypeA = uint8_t;
  414 |   using ValTypeB = uint8_t;
  415 |   using ValTypeC = int32_t;
  416 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U8U8S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U8U8S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 418-420

```text
  418 | template <>
  419 | struct MMA_Traits<SM80_16x8x32_S32U8U8S32_TN_SATURATE>
  420 |      : MMA_Traits<SM80_16x8x32_S32U8U8S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U8U8S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U8U8S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 422-441

```text
  422 | ///////////////////////////////////////////////////////////////////////////////
  423 | /////////////////////////// s32 = s4 * s4 + s32 ///////////////////////////////
  424 | ///////////////////////////////////////////////////////////////////////////////
  426 | template <>
  427 | struct MMA_Traits<SM80_8x8x32_S32S4S4S32_TN> {
  428 |   using ValTypeD = int32_t;
  429 |   using ValTypeA = int4b_t;
  430 |   using ValTypeB = int4b_t;
  431 |   using ValTypeC = int32_t;
  432 | 
  433 |   using Shape_MNK = Shape<_8, _8, _32>;
  434 |   using ThrID = Layout<_32>;
  435 |   // (T32,V8) -> (M8,N32)
  436 |   using ALayout = Layout<Shape <Shape < _4, _8>, Shape <_8>>,
  437 |                          Stride<Stride<_64, _1>, Stride<_8>>>;
  438 |   using BLayout = Layout<Shape <Shape < _4, _8>, Shape <_8>>,
  439 |                          Stride<Stride<_64, _1>, Stride<_8>>>;
  440 |   using CLayout = SM80_8x8_Row;
  441 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32S4S4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32S4S4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 443-445

```text
  443 | template <>
  444 | struct MMA_Traits<SM80_8x8x32_S32S4S4S32_TN_SATURATE>
  445 |      : MMA_Traits<SM80_8x8x32_S32S4S4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32S4S4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32S4S4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 447-463

```text
  447 | template <>
  448 | struct MMA_Traits<SM80_16x8x32_S32S4S4S32_TN> {
  449 |   using ValTypeD = int32_t;
  450 |   using ValTypeA = int4b_t;
  451 |   using ValTypeB = int4b_t;
  452 |   using ValTypeC = int32_t;
  453 | 
  454 |   using Shape_MNK = Shape<_16, _8, _32>;
  455 |   using ThrID = Layout<_32>;
  456 |   // (T32,V16) -> (M16,N32)
  457 |   using ALayout = Layout<Shape <Shape <  _4, _8>, Shape < _8, _2>>,
  458 |                          Stride<Stride<_128, _1>, Stride<_16, _8>>>;
  459 |   // (T32,V8) -> (M8,N32)
  460 |   using BLayout = Layout<Shape <Shape < _4, _8>, Shape <_8>>,
  461 |                          Stride<Stride<_32, _1>, Stride<_8>>>;
  462 |   using CLayout = SM80_16x8_Row;
  463 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S4S4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S4S4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 465-467

```text
  465 | template <>
  466 | struct MMA_Traits<SM80_16x8x32_S32S4S4S32_TN_SATURATE>
  467 |      : MMA_Traits<SM80_16x8x32_S32S4S4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S4S4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S4S4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 469-485

```text
  469 | template <>
  470 | struct MMA_Traits<SM80_16x8x64_S32S4S4S32_TN> {
  471 |   using ValTypeD = int32_t;
  472 |   using ValTypeA = int4b_t;
  473 |   using ValTypeB = int4b_t;
  474 |   using ValTypeC = int32_t;
  475 | 
  476 |   using Shape_MNK = Shape<_16, _8, _64>;
  477 |   using ThrID = Layout<_32>;
  478 |   // (T32,V32) -> (M16,N64)
  479 |   using ALayout = Layout<Shape <Shape <  _4, _8>, Shape < _8, _2, _2>>,
  480 |                          Stride<Stride<_128, _1>, Stride<_16, _8, _512>>>;
  481 |   // (T32,V16) -> (M8,N64)
  482 |   using BLayout = Layout<Shape <Shape < _4, _8>, Shape <_8, _2>>,
  483 |                          Stride<Stride<_64, _1>, Stride<_8, _256>>>;
  484 |   using CLayout = SM80_16x8_Row;
  485 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32S4S4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32S4S4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 487-489

```text
  487 | template <>
  488 | struct MMA_Traits<SM80_16x8x64_S32S4S4S32_TN_SATURATE>
  489 |      : MMA_Traits<SM80_16x8x64_S32S4S4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32S4S4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32S4S4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 491-502

```text
  491 | ///////////////////////////////////////////////////////////////////////////////
  492 | /////////////////////////// s32 = s4 * u4 + s32 ///////////////////////////////
  493 | ///////////////////////////////////////////////////////////////////////////////
  495 | template <>
  496 | struct MMA_Traits<SM80_8x8x32_S32S4U4S32_TN>
  497 |      : MMA_Traits<SM80_8x8x32_S32S4S4S32_TN> {
  498 |   using ValTypeD = int32_t;
  499 |   using ValTypeA = int4b_t;
  500 |   using ValTypeB = uint4b_t;
  501 |   using ValTypeC = int32_t;
  502 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32S4U4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32S4U4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 504-506

```text
  504 | template <>
  505 | struct MMA_Traits<SM80_8x8x32_S32S4U4S32_TN_SATURATE>
  506 |      : MMA_Traits<SM80_8x8x32_S32S4U4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32S4U4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32S4U4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 508-515

```text
  508 | template <>
  509 | struct MMA_Traits<SM80_16x8x32_S32S4U4S32_TN>
  510 |     : MMA_Traits<SM80_16x8x32_S32S4S4S32_TN> {
  511 |   using ValTypeD = int32_t;
  512 |   using ValTypeA = int4b_t;
  513 |   using ValTypeB = uint4b_t;
  514 |   using ValTypeC = int32_t;
  515 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S4U4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S4U4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 517-527

```text
  517 | template <>
  518 | struct MMA_Traits<SM80_16x8x32_S32S4U4S32_TN_SATURATE>
  519 |      : MMA_Traits<SM80_16x8x32_S32S4U4S32_TN> {};
  520 | template <>
  521 | struct MMA_Traits<SM80_16x8x64_S32S4U4S32_TN>
  522 |      : MMA_Traits<SM80_16x8x64_S32S4S4S32_TN> {
  523 |   using ValTypeD = int32_t;
  524 |   using ValTypeA = int4b_t;
  525 |   using ValTypeB = uint4b_t;
  526 |   using ValTypeC = int32_t;
  527 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32S4U4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32S4U4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 529-531

```text
  529 | template <>
  530 | struct MMA_Traits<SM80_16x8x64_S32S4U4S32_TN_SATURATE>
  531 |      : MMA_Traits<SM80_16x8x64_S32S4U4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32S4U4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32S4U4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 533-544

```text
  533 | ///////////////////////////////////////////////////////////////////////////////
  534 | /////////////////////////// s32 = u4 * s4 + s32 ///////////////////////////////
  535 | ///////////////////////////////////////////////////////////////////////////////
  537 | template <>
  538 | struct MMA_Traits<SM80_8x8x32_S32U4S4S32_TN>
  539 |      : MMA_Traits<SM80_8x8x32_S32S4S4S32_TN> {
  540 |   using ValTypeD = int32_t;
  541 |   using ValTypeA = uint4b_t;
  542 |   using ValTypeB = int4b_t;
  543 |   using ValTypeC = int32_t;
  544 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32U4S4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32U4S4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 546-548

```text
  546 | template <>
  547 | struct MMA_Traits<SM80_8x8x32_S32U4S4S32_TN_SATURATE>
  548 |      : MMA_Traits<SM80_8x8x32_S32U4S4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32U4S4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32U4S4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 550-557

```text
  550 | template <>
  551 | struct MMA_Traits<SM80_16x8x32_S32U4S4S32_TN>
  552 |      : MMA_Traits<SM80_16x8x32_S32S4S4S32_TN> {
  553 |   using ValTypeD = int32_t;
  554 |   using ValTypeA = uint4b_t;
  555 |   using ValTypeB = int4b_t;
  556 |   using ValTypeC = int32_t;
  557 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U4S4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U4S4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 559-561

```text
  559 | template <>
  560 | struct MMA_Traits<SM80_16x8x32_S32U4S4S32_TN_SATURATE>
  561 |      : MMA_Traits<SM80_16x8x32_S32U4S4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U4S4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U4S4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 563-570

```text
  563 | template <>
  564 | struct MMA_Traits<SM80_16x8x64_S32U4S4S32_TN>
  565 |      : MMA_Traits<SM80_16x8x64_S32S4S4S32_TN> {
  566 |   using ValTypeD = int32_t;
  567 |   using ValTypeA = uint4b_t;
  568 |   using ValTypeB = int4b_t;
  569 |   using ValTypeC = int32_t;
  570 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32U4S4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32U4S4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 572-574

```text
  572 | template <>
  573 | struct MMA_Traits<SM80_16x8x64_S32U4S4S32_TN_SATURATE>
  574 |     : MMA_Traits<SM80_16x8x64_S32U4S4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32U4S4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32U4S4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 576-587

```text
  576 | ///////////////////////////////////////////////////////////////////////////////
  577 | /////////////////////////// s32 = u4 * u4 + s32 ///////////////////////////////
  578 | ///////////////////////////////////////////////////////////////////////////////
  580 | template <>
  581 | struct MMA_Traits<SM80_8x8x32_S32U4U4S32_TN>
  582 |      : MMA_Traits<SM80_8x8x32_S32S4S4S32_TN> {
  583 |   using ValTypeD = int32_t;
  584 |   using ValTypeA = uint4b_t;
  585 |   using ValTypeB = uint4b_t;
  586 |   using ValTypeC = int32_t;
  587 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32U4U4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32U4U4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 589-591

```text
  589 | template <>
  590 | struct MMA_Traits<SM80_8x8x32_S32U4U4S32_TN_SATURATE>
  591 |      : MMA_Traits<SM80_8x8x32_S32U4U4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x32_S32U4U4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x32_S32U4U4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 593-600

```text
  593 | template <>
  594 | struct MMA_Traits<SM80_16x8x32_S32U4U4S32_TN>
  595 |      : MMA_Traits<SM80_16x8x32_S32S4S4S32_TN> {
  596 |   using ValTypeD = int32_t;
  597 |   using ValTypeA = uint4b_t;
  598 |   using ValTypeB = uint4b_t;
  599 |   using ValTypeC = int32_t;
  600 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U4U4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U4U4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 602-604

```text
  602 | template <>
  603 | struct MMA_Traits<SM80_16x8x32_S32U4U4S32_TN_SATURATE>
  604 |      : MMA_Traits<SM80_16x8x32_S32U4U4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x32_S32U4U4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x32_S32U4U4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 606-613

```text
  606 | template <>
  607 | struct MMA_Traits<SM80_16x8x64_S32U4U4S32_TN>
  608 |      : MMA_Traits<SM80_16x8x64_S32S4S4S32_TN> {
  609 |   using ValTypeD = int32_t;
  610 |   using ValTypeA = uint4b_t;
  611 |   using ValTypeB = uint4b_t;
  612 |   using ValTypeC = int32_t;
  613 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32U4U4S32_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32U4U4S32_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 615-617

```text
  615 | template <>
  616 | struct MMA_Traits<SM80_16x8x64_S32U4U4S32_TN_SATURATE>
  617 |      : MMA_Traits<SM80_16x8x64_S32U4U4S32_TN> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x64_S32U4U4S32_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x64_S32U4U4S32_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 619-638

```text
  619 | ///////////////////////////////////////////////////////////////////////////////
  620 | /////////////////////////// s32 = b1 ^ b1 + s32 ///////////////////////////////
  621 | ///////////////////////////////////////////////////////////////////////////////
  623 | template <>
  624 | struct MMA_Traits<SM80_16x8x256_S32U1U1S32_TN_XORPOPC>
  625 | {
  626 |   using ValTypeD = int32_t;
  627 |   using ValTypeA = cute::uint1b_t;
  628 |   using ValTypeB = cute::uint1b_t;
  629 |   using ValTypeC = int32_t;
  630 | 
  631 |   using Shape_MNK = Shape<_16,_8,_256>;
  632 |   using ThrID   = Layout<_32>;
  633 |   using ALayout = Layout<Shape<Shape<_4,_8>,Shape<_32,_2,_2>>,
  634 |                        Stride<Stride<_512,_1>,Stride<_16,_8,_2048>>>;
  635 |   using BLayout = Layout<Shape<Shape <_4,_8>,Shape<_32,_2>>,
  636 |                          Stride<Stride<_256,_1>,Stride< _8,_1024>>>;
  637 |   using CLayout = SM80_16x8_Row;
  638 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x256_S32U1U1S32_TN_XORPOPC`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x256_S32U1U1S32_TN_XORPOPC` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 640-646

```text
  640 | ///////////////////////////////////////////////////////////////////////////////
  641 | /////////////////////////// s32 = b1 & b1 + s32 ///////////////////////////////
  642 | ///////////////////////////////////////////////////////////////////////////////
  644 | template <>
  645 | struct MMA_Traits<SM80_16x8x256_S32U1U1S32_TN_ANDPOPC>
  646 |      : MMA_Traits<SM80_16x8x256_S32U1U1S32_TN_XORPOPC> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x256_S32U1U1S32_TN_ANDPOPC`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x256_S32U1U1S32_TN_ANDPOPC` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 648-663

```text
  648 | template<>
  649 | struct MMA_Traits<SM80_8x8x128_S32U1U1S32_TN_XORPOPC>
  650 | {
  651 |   using ValTypeD = int32_t;
  652 |   using ValTypeA = cute::uint1b_t;
  653 |   using ValTypeB = cute::uint1b_t;
  654 |   using ValTypeC = int32_t;
  655 | 
  656 |   using Shape_MNK = Shape<_8,_8,_128>;
  657 |   using ThrID   = Layout<_32>;
  658 |   using ALayout = Layout<Shape<Shape<_4,_8>,_32>,
  659 |                          Stride<Stride<_256,_1>,_8>>;
  660 |   using BLayout = Layout<Shape<Shape<_4,_8>,_32>,
  661 |                          Stride<Stride<_256,_1>,_8>>;
  662 |   using CLayout = SM80_8x8_Row;
  663 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x128_S32U1U1S32_TN_XORPOPC`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x128_S32U1U1S32_TN_XORPOPC` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 665-667

```text
  665 | template <>
  666 | struct MMA_Traits<SM80_8x8x128_S32U1U1S32_TN_ANDPOPC>
  667 |       :MMA_Traits<SM80_8x8x128_S32U1U1S32_TN_XORPOPC> {};
```
**EN:** Specializes `MMA_Traits` for `SM80_8x8x128_S32U1U1S32_TN_ANDPOPC`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_8x8x128_S32U1U1S32_TN_ANDPOPC` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 669-684

```text
  669 | template<>
  670 | struct MMA_Traits<SM80_16x8x128_S32U1U1S32_TN_XORPOPC>
  671 | {
  672 |   using ValTypeD = int32_t;
  673 |   using ValTypeA = cute::uint1b_t;
  674 |   using ValTypeB = cute::uint1b_t;
  675 |   using ValTypeC = int32_t;
  676 | 
  677 |   using Shape_MNK = Shape<_16,_8,_128>;
  678 |   using ThrID   = Layout<_32>;
  679 |   using ALayout = Layout<Shape<Shape<_4,_8>,Shape<_32,_2>>,
  680 |                        Stride<Stride<_512,_1>,Stride<Stride<_16,_8>>>>;
  681 |   using BLayout = Layout<Shape <Shape<_4,_8>,_32>,
  682 |                          Stride<Stride<_256,_1>,_8>>;
  683 |   using CLayout = SM80_16x8_Row;
  684 | };
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x128_S32U1U1S32_TN_XORPOPC`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x128_S32U1U1S32_TN_XORPOPC` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 686-690

```text
  686 | template <>
  687 | struct MMA_Traits<SM80_16x8x128_S32U1U1S32_TN_ANDPOPC>
  688 |       :MMA_Traits<SM80_16x8x128_S32U1U1S32_TN_XORPOPC> {};
  690 | } // end namespace cute
```
**EN:** Specializes `MMA_Traits` for `SM80_16x8x128_S32U1U1S32_TN_ANDPOPC`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM80_16x8x128_S32U1U1S32_TN_ANDPOPC` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Compile-time numeric metaprogramming / 编译期数值元编程
- Intel Xe-specific behavior / Intel Xe 特定行为
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/arch/mma_sm80.hpp`
  - `cute/atom/mma_traits.hpp`
  - `cute/layout.hpp`
  - `cute/numeric/numeric_types.hpp`
- Primary symbols / 主要符号: `MMA_Traits`, `SM80_8x4`, `SM80_8x8_Row`, `SM80_8x16_Row`, `SM80_16x8_Row`, `ValTypeD`, `ValTypeA`, `ValTypeB`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
