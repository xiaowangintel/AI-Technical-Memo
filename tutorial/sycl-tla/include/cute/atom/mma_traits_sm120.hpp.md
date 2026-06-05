# mma_traits_sm120.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/atom/mma_traits_sm120.hpp`
- Purpose (EN): Defines generic or architecture-specific `MMA_Traits` specializations that describe operand types, tile shapes, thread/value layouts, and fragment storage for MMA instructions.
- 作用 (CN): 定义通用或架构相关的 `MMA_Traits` 特化，用于描述 MMA 指令的操作数类型、tile 形状、线程/值布局以及 fragment 存储方式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```text
    1 | /***************************************************************************************************
    2 |  * Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 30-32

```text
   30 |  **************************************************************************************************/
   32 | #pragma once
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 34-38

```text
   34 | #include <cute/arch/mma_sm120.hpp>
   35 | #include <cute/atom/mma_traits.hpp>
   36 | #include <cute/atom/mma_traits_sm80.hpp>
   37 | #include <cute/layout.hpp>
   38 | #include <cute/numeric/numeric_types.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/arch/mma_sm120.hpp`, `cute/atom/mma_traits.hpp`, `cute/atom/mma_traits_sm80.hpp`, `cute/layout.hpp`, `cute/numeric/numeric_types.hpp`.
**CN:** 通过引入 `cute/arch/mma_sm120.hpp`, `cute/atom/mma_traits.hpp`, `cute/atom/mma_traits_sm80.hpp`, `cute/layout.hpp`, `cute/numeric/numeric_types.hpp` 为该文件建立头文件依赖。

### Lines 40-41

```text
   40 | namespace cute
   41 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 43-43

```text
   43 | namespace SM120::BLOCKSCALED {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 45-115

```text
   45 | template <class MMAOp,
   46 |           class TD, class DLayout,
   47 |           class TA, class ALayout,
   48 |           class TB, class BLayout,
   49 |           class TC, class CLayout>
   50 | CUTE_HOST_DEVICE constexpr void
   51 | mma_unpack(MMA_Traits<MMAOp>   const& traits,
   52 |            Tensor<TD, DLayout>      & D,
   53 |            Tensor<TA, ALayout> const& A_zipped,
   54 |            Tensor<TB, BLayout> const& B_zipped,
   55 |            Tensor<TC, CLayout> const& C)
   56 | {
   57 |   static_assert(is_rmem<TD>::value, "Expected registers in MMA_Atom::call");
   58 |   static_assert(is_rmem<TA>::value, "Expected registers in MMA_Atom::call");
   59 |   static_assert(is_rmem<TB>::value, "Expected registers in MMA_Atom::call");
   60 |   static_assert(is_rmem<TC>::value, "Expected registers in MMA_Atom::call");
   61 | 
   62 |   // Register value types from the MMA_Operation register arrays
   63 |   using          RegTypeD = typename remove_extent<typename MMAOp::DRegisters>::type;
   64 |   using          RegTypeA = typename remove_extent<typename MMAOp::ARegisters>::type;
   65 |   using          RegTypeB = typename remove_extent<typename MMAOp::BRegisters>::type;
   66 |   using          RegTypeC = typename remove_extent<typename MMAOp::CRegisters>::type;
   67 |   using        RegTypeSFA = typename remove_extent<typename MMAOp::SFARegisters>::type;
   68 |   using        RegTypeSFB = typename remove_extent<typename MMAOp::SFBRegisters>::type;
   69 | 
   70 |   constexpr int   RegNumD = extent<typename MMAOp::DRegisters>::value;
   71 |   constexpr int   RegNumA = extent<typename MMAOp::ARegisters>::value;
   72 |   constexpr int   RegNumB = extent<typename MMAOp::BRegisters>::value;
   73 |   constexpr int   RegNumC = extent<typename MMAOp::CRegisters>::value;
   74 |   constexpr int RegNumSFA = extent<typename MMAOp::SFARegisters>::value;
   75 |   constexpr int RegNumSFB = extent<typename MMAOp::SFBRegisters>::value;
   76 | 
   77 |   auto  [A, SFA] = unzip_tensor(A_zipped);
   78 |   auto  [B, SFB] = unzip_tensor(B_zipped);
   79 |   
   80 |   using Shape_MNK = typename MMA_Traits<MMAOp>::Shape_MNK;
   81 |   constexpr int SFVecSize = MMA_Traits<MMAOp>::SFVecSize;
   82 |   
   83 |   // Assert logical size
   84 |   CUTE_STATIC_ASSERT_V(size(SFA) == size<2>(Shape_MNK{}));
   85 |   CUTE_STATIC_ASSERT_V(size(SFB) == size<2>(Shape_MNK{})); 
   86 | 
   87 |   // Assert physical size
   88 |   CUTE_STATIC_ASSERT(decltype(cosize(layout(SFA))){} == size<2>(Shape_MNK{}) / SFVecSize); 
   89 |   CUTE_STATIC_ASSERT(decltype(cosize(layout(SFB))){} == size<2>(Shape_MNK{}) / SFVecSize); 
   90 | 
   91 |   Tensor rA = recast<RegTypeA>(A);
   92 |   Tensor rB = recast<RegTypeB>(B);
   93 |   CUTE_STATIC_ASSERT_V(size(rA) == Int<RegNumA>{});
   94 |   CUTE_STATIC_ASSERT_V(size(rB) == Int<RegNumB>{});
   95 | 
   96 |   Tensor rD = recast<RegTypeD>(D);
   97 |   Tensor rC = recast<RegTypeC>(C);
   98 |   CUTE_STATIC_ASSERT_V(size(rD) == Int<RegNumD>{});
   99 |   CUTE_STATIC_ASSERT_V(size(rC) == Int<RegNumC>{});
  100 | 
  101 |   Tensor rSFA = recast<RegTypeSFA>(filter_zeros(SFA));
  102 |   Tensor rSFB = recast<RegTypeSFB>(filter_zeros(SFB));
  103 | 
  104 |   CUTE_STATIC_ASSERT_V(size(rSFA) == Int<RegNumSFA>{});
  105 |   CUTE_STATIC_ASSERT_V(size(rSFB) == Int<RegNumSFB>{});
  106 | 
  107 |   detail::explode(MMAOp::fma,
  108 |             rD,   make_int_sequence<RegNumD>{},
  109 |             rA,   make_int_sequence<RegNumA>{},
  110 |             rB,   make_int_sequence<RegNumB>{},
  111 |             rC,   make_int_sequence<RegNumC>{},
  112 |             rSFA, make_int_sequence<RegNumSFA>{},
  113 |             rSFB, make_int_sequence<RegNumSFB>{});
  114 | }
  115 | } // namespace SM120::BLOCKSCALED
```
**EN:** Specializes `MMA_Traits` for `MMAOp`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `MMAOp` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 117-130

```text
  117 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  119 | // MMA F8F6F4 16x8x32 TN
  120 | template <class a_type, class b_type, class c_type>
  121 | struct MMA_Traits<SM120_16x8x32_TN<a_type, b_type, c_type>>
  122 |      : MMA_Traits<SM80_16x8x32_S32S8S8S32_TN>
  123 | {
  124 |   // The MMA accepts 8-bit inputs regardless of the types for A and B
  125 |   using ValTypeA = uint8_t;
  126 |   using ValTypeB = uint8_t;
  127 | 
  128 |   using ValTypeD = c_type;
  129 |   using ValTypeC = c_type;
  130 | };
```
**EN:** Specializes `MMA_Traits` for `SM120_16x8x32_TN<a_type, b_type, c_type`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM120_16x8x32_TN<a_type, b_type, c_type` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 132-165

```text
  132 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  134 | // MMA MXF8F6F4 16x8x64 TN
  135 | template <class a_type, class b_type, class c_type, class sf_type, int VS>
  136 | struct MMA_Traits<SM120::BLOCKSCALED::SM120_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS>>
  137 | {
  138 |   // The MMA accepts 4-bit inputs regardless of the types for A and B
  139 |   using ValTypeA = uint4_t;
  140 |   using ValTypeB = uint4_t;
  141 | 
  142 |   using ValTypeD = c_type;
  143 |   using ValTypeC = c_type;
  144 | 
  145 |   using ValTypeSF = sf_type;
  146 |   constexpr static int SFVecSize = VS;
  147 | 
  148 |   using Shape_MNK = Shape<_16,_8,_64>;
  149 |   using ThrID     = Layout<_32>;
  150 | 
  151 |   // (T32,V32) -> (M16,K64)
  152 |   using ALayout   = Layout<Shape <Shape <  _4,_8>,Shape < _8,_2,  _2>>,
  153 |                            Stride<Stride<_128,_1>,Stride<_16,_8,_512>>>;
  154 |   // (T32,V16) -> (M16,K64)
  155 |   using BLayout   = Layout<Shape <Shape < _4,_8>,Shape <_8,  _2>>,
  156 |                            Stride<Stride<_64,_1>,Stride<_8,_256>>>;
  157 |   // (T32,V64) -> (M16,K64)
  158 |   using SFALayout = Layout<Shape <Shape <_2,_2,_8>,_64>,  // Effectively 16 threads due to the 2:0 mode
  159 |                            Stride<Stride<_8,_0,_1>,_16>>;
  160 |   // (T32,V64) -> (N8,K64)
  161 |   using SFBLayout = Layout<Shape <Shape <_4,_8>,_64>,     // Effectively 8 threads due to the 4:0 mode
  162 |                            Stride<Stride<_0,_1>, _8>>;
  163 |   // (T32,V4)  -> (M16,N8)
  164 |   using CLayout   = SM80_16x8_Row;
  165 | };
```
**EN:** Specializes `MMA_Traits` for `SM120::BLOCKSCALED::SM120_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM120::BLOCKSCALED::SM120_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 167-199

```text
  167 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  169 | // MMA MXF8F6F4 16x8x32 TN
  170 | template <class a_type, class b_type, class c_type, class sf_type, int VS>
  171 | struct MMA_Traits<SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<a_type, b_type, c_type, sf_type, VS>>
  172 | {
  173 |   using UnderlyingTraits = MMA_Traits<SM120_16x8x32_TN<a_type, b_type, c_type>>;
  174 | 
  175 |   // The MMA accepts 8-bit inputs regardless of the types for A and B
  176 |   using ValTypeA = typename UnderlyingTraits::ValTypeA;
  177 |   using ValTypeB = typename UnderlyingTraits::ValTypeB;
  178 | 
  179 |   using ValTypeD = typename UnderlyingTraits::ValTypeD;
  180 |   using ValTypeC = typename UnderlyingTraits::ValTypeC;
  181 | 
  182 |   using Shape_MNK = typename UnderlyingTraits::Shape_MNK;
  183 |   using ThrID     = typename UnderlyingTraits::ThrID;
  184 | 
  185 |   using ALayout   = typename UnderlyingTraits::ALayout;
  186 |   using BLayout   = typename UnderlyingTraits::BLayout;
  187 |   using CLayout   = typename UnderlyingTraits::CLayout;
  188 | 
  189 |   // Scaling factor
  190 |   using ValTypeSF = sf_type;
  191 |   constexpr static int SFVecSize = VS;
  192 | 
  193 |   // (T32,V32) -> (M16,K32)
  194 |   using SFALayout = Layout<Shape <Shape <_2,_2,_8>,_32>,  // Effectively 16 threads due to the 2:0 mode
  195 |                            Stride<Stride<_8,_0,_1>,_16>>;
  196 |   // (T32,V32) -> (N8,K32)
  197 |   using SFBLayout = Layout<Shape <Shape <_4,_8>,_32>,     // Effectively 8 threads due to the 4:0 mode
  198 |                            Stride<Stride<_0,_1>, _8>>;
  199 | };
```
**EN:** Specializes `MMA_Traits` for `SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<a_type, b_type, c_type, sf_type, VS`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<a_type, b_type, c_type, sf_type, VS` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 201-209

```text
  201 | // Transform if needed
  202 | template<class MMA_Op, class Tensor>
  203 | CUTLASS_DEVICE void
  204 | fp4_shift_A(MMA_Op const& op, Tensor&& tensor) {
  205 | }
  206 | template<class MMA_Op, class Tensor>
  207 | CUTLASS_DEVICE void
  208 | fp4_shift_B(MMA_Op const& op, Tensor&& tensor) {
  209 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 211-235

```text
  211 | // For SM120 MMA F8F6F4 input fp4, the operand A/B are load from ld.matrix. 
  212 | // ld.matrix b4x16_p64 places FP4 data at the first four bits in each
  213 | // eight-bit container, whereas MMA F8F6F4 expects the four-bit data to be in 
  214 | // the middle of the eight-bit container. Thus, e2m1 operands being fed
  215 | // to MMA F8F6F4 must be shifted left by two bits.
  216 | // 0b0000ABCD --> 0b00ABCD00
  217 | // NOTE: Same transformation is NOT needed for FP6 and FP8.
  218 | template<class AType, class BType, class... MMAArgs, class Tensor>
  219 | CUTLASS_DEVICE void
  220 | fp4_shift_A(SM120_16x8x32_TN<AType, BType, MMAArgs ...> const&, Tensor&& tensor) {
  221 |   using RegisterTypeA = typename remove_extent<typename
  222 |                         SM120_16x8x32_TN<AType, BType, MMAArgs ...>::ARegisters>::type;
  223 |   if constexpr (cute::is_same_v<AType, cutlass::float_e2m1_t>) {
  224 |     cute::transform(recast<RegisterTypeA>(tensor), [](RegisterTypeA& v){ return v << 2; });
  225 |   }
  226 | }
  227 | template<class AType, class BType, class... MMAArgs, class Tensor>
  228 | CUTLASS_DEVICE void
  229 | fp4_shift_B(SM120_16x8x32_TN<AType, BType, MMAArgs ...> const&, Tensor&& tensor) {
  230 |   using RegisterTypeB = typename remove_extent<typename
  231 |                         SM120_16x8x32_TN<AType, BType, MMAArgs ...>::BRegisters>::type;
  232 |   if constexpr (cute::is_same_v<BType, cutlass::float_e2m1_t>) {
  233 |     cute::transform(recast<RegisterTypeB>(tensor), [](RegisterTypeB& v){ return v << 2; });
  234 |   }
  235 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 237-237

```text
  237 | namespace SM120::BLOCKSCALED {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 239-262

```text
  239 | // Template function with scale factor needs to enmuerate types one by one, as template 
  240 | // arguments contatins two variadic lists, which cannot be deduced in one shot.
  241 | template<class AType, class BType, class CType, class SFType, int VS, class Tensor>
  242 | CUTLASS_DEVICE void
  243 | fp4_shift_A(SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<AType, BType, CType, SFType, VS> const&, Tensor&& tensor) {
  244 |   using RegisterTypeA = typename remove_extent<typename
  245 |                         SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<AType, BType, CType, SFType, VS>::ARegisters>::type;
  246 |   if constexpr (cute::is_same_v<AType, cutlass::float_e2m1_t>) {
  247 |     cute::transform(recast<RegisterTypeA>(tensor), [](RegisterTypeA& v){ return v << 2; });
  248 |   }
  249 | }
  250 | template<class AType, class BType, class CType, class SFType, int VS, class Tensor>
  251 | CUTLASS_DEVICE void
  252 | fp4_shift_B(SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<AType, BType, CType, SFType, VS> const&, Tensor&& tensor) {
  253 |   using RegisterTypeB = typename remove_extent<typename
  254 |                         SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<AType, BType, CType, SFType, VS>::BRegisters>::type;
  255 |   if constexpr (cute::is_same_v<BType, cutlass::float_e2m1_t>) {
  256 |     cute::transform(recast<RegisterTypeB>(tensor), [](RegisterTypeB& v){ return v << 2; });
  257 |   }
  258 | }
  260 | }
  262 | } // end namespace cute
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Intel Xe-specific behavior / Intel Xe 特定行为
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/arch/mma_sm120.hpp`
  - `cute/atom/mma_traits.hpp`
  - `cute/atom/mma_traits_sm80.hpp`
  - `cute/layout.hpp`
  - `cute/numeric/numeric_types.hpp`
- Primary symbols / 主要符号: `MMA_Traits`, `MMAOp`, `TD`, `DLayout`, `TA`, `ALayout`, `TB`, `BLayout`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
