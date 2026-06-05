# mma_traits_sm120_sparse.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/atom/mma_traits_sm120_sparse.hpp`
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
   35 | #include <cute/arch/mma_sm120_sparse.hpp>
   36 | #include <cute/atom/mma_traits.hpp>
   37 | #include <cute/layout.hpp>
   38 | #include <cute/numeric/numeric_types.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/arch/mma_sm120.hpp`, `cute/arch/mma_sm120_sparse.hpp`, `cute/atom/mma_traits.hpp`, `cute/layout.hpp`, `cute/numeric/numeric_types.hpp`.
**CN:** 通过引入 `cute/arch/mma_sm120.hpp`, `cute/arch/mma_sm120_sparse.hpp`, `cute/atom/mma_traits.hpp`, `cute/layout.hpp`, `cute/numeric/numeric_types.hpp` 为该文件建立头文件依赖。

### Lines 40-41

```text
   40 | namespace cute
   41 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 43-43

```text
   43 | namespace {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 45-49

```text
   45 | // (T32,V4) -> (M16,N8)
   46 | using SM120_16x8_Row = Layout<Shape <Shape < _4,_8>,Shape < _2,_2>>,
   47 |                              Stride<Stride<_32,_1>,Stride<_16,_8>>>;
   49 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 51-122

```text
   51 | namespace SM120::BLOCKSCALED::SPARSE
   52 | {
   53 | 
   54 | // Unpack explode/mma call with sparse and block scalaring inputs.
   55 | template <class MMAOp,
   56 |           class TD, class DLayout,
   57 |           class TA, class ALayout,
   58 |           class TB, class BLayout,
   59 |           class TC, class CLayout>
   60 | CUTE_HOST_DEVICE constexpr void
   61 | mma_unpack(MMA_Traits<MMAOp>  const&,
   62 |           Tensor<TD, DLayout>      & D,
   63 |           Tensor<TA, ALayout> const& A,
   64 |           Tensor<TB, BLayout> const& B,
   65 |           Tensor<TC, CLayout> const& C)
   66 | {
   67 |   static_assert(is_rmem_v<TD>, "Expected registers in MMA_Atom::call");
   68 |   static_assert(is_rmem_v<TA>, "Expected registers in MMA_Atom::call");
   69 |   static_assert(is_rmem_v<TB>, "Expected registers in MMA_Atom::call");
   70 |   static_assert(is_rmem_v<TC>, "Expected registers in MMA_Atom::call");
   71 |   using         DRegisters = typename MMAOp::DRegisters;
   72 |   using         ARegisters = typename MMAOp::ARegisters;
   73 |   using         ERegisters = typename MMAOp::ERegisters;
   74 |   using         BRegisters = typename MMAOp::BRegisters;
   75 |   using         CRegisters = typename MMAOp::CRegisters;
   76 |   using         SFARegisters = typename MMAOp::SFARegisters;
   77 |   using         SFBRegisters = typename MMAOp::SFBRegisters;
   78 |   // Register value types from the MMAOp register arrays
   79 |   using         RegTypeD   = typename remove_extent<DRegisters>::type;
   80 |   using         RegTypeA   = typename remove_extent<ARegisters>::type;
   81 |   using         RegTypeE   = typename remove_extent<ERegisters>::type;
   82 |   using         RegTypeB   = typename remove_extent<BRegisters>::type;
   83 |   using         RegTypeC   = typename remove_extent<CRegisters>::type;
   84 |   using         RegTypeSFA = typename remove_extent<SFARegisters>::type;
   85 |   using         RegTypeSFB = typename remove_extent<SFBRegisters>::type;
   86 |   constexpr int RegNumD    = extent<DRegisters>::value;
   87 |   constexpr int RegNumA    = extent<ARegisters>::value;
   88 |   constexpr int RegNumE    = extent<ERegisters>::value;
   89 |   constexpr int RegNumB    = extent<BRegisters>::value;
   90 |   constexpr int RegNumC    = extent<CRegisters>::value;
   91 |   constexpr int RegNumSFA  = extent<SFARegisters>::value;
   92 |   constexpr int RegNumSFB  = extent<SFBRegisters>::value;
   93 | 
   94 |   auto  [tA, tSFA, tE] = unzip_tensor(A);
   95 |   auto  [tB, tSFB    ] = unzip_tensor(B);
   96 |   Tensor rA      = recast<RegTypeA>(tA);
   97 |   Tensor rE      = recast<RegTypeE>(tE);
   98 |   Tensor rB      = recast<RegTypeB>(tB);
   99 |   Tensor rD      = recast<RegTypeD>(D);
  100 |   Tensor rC      = recast<RegTypeC>(C);
  101 |   Tensor rSFA    = recast<RegTypeSFA>(tSFA);
  102 |   Tensor rSFB    = recast<RegTypeSFB>(tSFB);
  103 | 
  104 |   CUTE_STATIC_ASSERT_V(size(rA)   == Int<RegNumA>{});
  105 |   CUTE_STATIC_ASSERT_V(size(rE)   == Int<RegNumE>{});
  106 |   CUTE_STATIC_ASSERT_V(size(rB)   == Int<RegNumB>{});
  107 |   CUTE_STATIC_ASSERT_V(size(rD)   == Int<RegNumD>{});
  108 |   CUTE_STATIC_ASSERT_V(size(rC)   == Int<RegNumC>{});
  109 |   CUTE_STATIC_ASSERT_V(size(filter_zeros(rSFA)) == Int<RegNumSFA>{});
  110 |   CUTE_STATIC_ASSERT_V(size(filter_zeros(rSFB)) == Int<RegNumSFB>{});
  111 | 
  112 |   detail::explode(MMAOp::fma,
  113 |                   rD, make_int_sequence<RegNumD>{},
  114 |                   rA, make_int_sequence<RegNumA>{},
  115 |                   rB, make_int_sequence<RegNumB>{},
  116 |                   rC, make_int_sequence<RegNumC>{},
  117 |                   rE, make_int_sequence<RegNumE>{},
  118 |                   rSFA, make_int_sequence<RegNumSFA>{},
  119 |                   rSFB, make_int_sequence<RegNumSFB>{});
  120 | }
  121 | 
  122 | } // end namespace SM120::BLOCKSCALED::SPARSE
```
**EN:** Specializes `MMA_Traits` for `MMAOp`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `MMAOp` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 125-181

```text
  125 | namespace SM120::SPARSE
  126 | {
  127 | 
  128 | template <class MMAOp,
  129 |           class TD, class DLayout,
  130 |           class TA, class ALayout,
  131 |           class TB, class BLayout,
  132 |           class TC, class CLayout>
  133 | CUTE_HOST_DEVICE constexpr void
  134 | mma_unpack(MMA_Traits<MMAOp>  const&,
  135 |           Tensor<TD, DLayout>      & D,
  136 |           Tensor<TA, ALayout> const& A,
  137 |           Tensor<TB, BLayout> const& B,
  138 |           Tensor<TC, CLayout> const& C)
  139 | {
  140 |   static_assert(is_rmem_v<TD>, "Expected registers in MMA_Atom::call");
  141 |   static_assert(is_rmem_v<TA>, "Expected registers in MMA_Atom::call");
  142 |   static_assert(is_rmem_v<TB>, "Expected registers in MMA_Atom::call");
  143 |   static_assert(is_rmem_v<TC>, "Expected registers in MMA_Atom::call");
  144 |   using         DRegisters = typename MMAOp::DRegisters;
  145 |   using         ARegisters = typename MMAOp::ARegisters;
  146 |   using         ERegisters = typename MMAOp::ERegisters;
  147 |   using         BRegisters = typename MMAOp::BRegisters;
  148 |   using         CRegisters = typename MMAOp::CRegisters;
  149 |   // Register value types from the MMAOp register arrays
  150 |   using         RegTypeD   = typename remove_extent<DRegisters>::type;
  151 |   using         RegTypeA   = typename remove_extent<ARegisters>::type;
  152 |   using         RegTypeE   = typename remove_extent<ERegisters>::type;
  153 |   using         RegTypeB   = typename remove_extent<BRegisters>::type;
  154 |   using         RegTypeC   = typename remove_extent<CRegisters>::type;
  155 |   constexpr int RegNumD    = extent<DRegisters>::value;
  156 |   constexpr int RegNumA    = extent<ARegisters>::value;
  157 |   constexpr int RegNumE    = extent<ERegisters>::value;
  158 |   constexpr int RegNumB    = extent<BRegisters>::value;
  159 |   constexpr int RegNumC    = extent<CRegisters>::value;
  160 | 
  161 |   auto  [tA, tE] = unzip_tensor(A);
  162 |   Tensor rA      = recast<RegTypeA>(tA);
  163 |   Tensor rE      = recast<RegTypeE>(tE);
  164 |   Tensor rB      = recast<RegTypeB>(B);
  165 |   Tensor rD      = recast<RegTypeD>(D);
  166 |   Tensor rC      = recast<RegTypeC>(C);
  167 |   CUTE_STATIC_ASSERT_V(size(rA) == Int<RegNumA>{});
  168 |   CUTE_STATIC_ASSERT_V(size(rE) == Int<RegNumE>{});
  169 |   CUTE_STATIC_ASSERT_V(size(rB) == Int<RegNumB>{});
  170 |   CUTE_STATIC_ASSERT_V(size(rD) == Int<RegNumD>{});
  171 |   CUTE_STATIC_ASSERT_V(size(rC) == Int<RegNumC>{});
  172 | 
  173 |   detail::explode(MMAOp::fma,
  174 |                   rD, make_int_sequence<RegNumD>{},
  175 |                   rA, make_int_sequence<RegNumA>{},
  176 |                   rB, make_int_sequence<RegNumB>{},
  177 |                   rC, make_int_sequence<RegNumC>{},
  178 |                   rE, make_int_sequence<RegNumE>{});
  179 | }
  180 | 
  181 | } // end namespace SM120::SPARSE
```
**EN:** Specializes `MMA_Traits` for `MMAOp`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `MMAOp` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 183-210

```text
  183 | // sparse F8F6F4 without block-scaling
  184 | template <class a_type, class b_type, class c_type>
  185 | struct MMA_Traits<SM120::SPARSE::SM120_SPARSE_16x8x64_TN<a_type, b_type, c_type>>
  186 | {
  187 |   using ValTypeA = sparse_elem<2, a_type>;
  188 |   using ValTypeE = sparse_elem<8, uint8_t>;
  189 |   using ValTypeB = uint8_t;
  190 |   using FrgTypeA = sparse_elem<2, uint8_t>;
  191 |   using FrgTypeE = sparse_elem<8, uint8_t>;
  192 | 
  193 |   using ValTypeC = c_type;
  194 |   using ValTypeD = c_type;
  195 | 
  196 |   using Shape_MNK = Shape<_16, _8, _64>;
  197 |   using ThrID     = Layout<_32>;
  198 |   // (T32,V32) -> (M16,K64)
  199 |   using ALayout   = Layout<Shape <Shape <  _4,_8>,Shape < _8,_2,  _2>>,
  200 |                            Stride<Stride<_128,_1>,Stride<_16,_8,_512>>>;
  201 |   // (T32,V16) -> (N8,K64)
  202 |   using BLayout   = Layout<Shape <Shape < _4,_8>,Shape <_4,  _4>>,
  203 |                            Stride<Stride<_32,_1>,Stride<_8,_128>>>;
  204 |   // (T32,V4)  -> (M16,N8)
  205 |   using CLayout   = SM120_16x8_Row;
  206 | 
  207 |   // (T32, V32) -> (M16, K64) 
  208 |   using ELayout   = Layout<Shape <Shape <_2,  _2,_8>, _32>,
  209 |                            Stride<Stride<_8,_512,_1>,_16>>;
  210 | };
```
**EN:** Specializes `MMA_Traits` for `SM120::SPARSE::SM120_SPARSE_16x8x64_TN<a_type, b_type, c_type`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM120::SPARSE::SM120_SPARSE_16x8x64_TN<a_type, b_type, c_type` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 212-232

```text
  212 | // sparse MXF8F6F4 with block-scaling.
  213 | template <class a_type, class b_type, class c_type, class sf_type, int VS>
  214 | struct MMA_Traits<SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS>>
  215 |       : MMA_Traits<SM120::SPARSE::SM120_SPARSE_16x8x64_TN<a_type, b_type, c_type>>
  216 | {
  217 |   using ValTypeA = sparse_elem<2, a_type>;
  218 |   using ValTypeE = sparse_elem<8, uint8_t>;
  219 |   using ValTypeB = uint8_t;
  220 |   using FrgTypeA = sparse_elem<2, uint8_t>;
  221 |   using FrgTypeE = sparse_elem<8, uint8_t>;
  222 | 
  223 |   using ValTypeD = c_type;
  224 |   using ValTypeC = c_type;
  225 | 
  226 |   using ValTypeSF = sf_type;
  227 |   constexpr static int SFVecSize = VS;
  228 | 
  229 |   using UnderlyingSFTraits = MMA_Traits<SM120::BLOCKSCALED::SM120_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS>>;
  230 |   using SFALayout = typename UnderlyingSFTraits::SFALayout;
  231 |   using SFBLayout = typename UnderlyingSFTraits::SFBLayout;
  232 | };
```
**EN:** Specializes `MMA_Traits` for `SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x64_TN_VS<a_type, b_type, c_type, sf_type, VS` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 234-269

```text
  234 | template <class a_type, class b_type, class c_type, class sf_type, int VS> 
  235 | struct MMA_Traits<SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x128_TN_VS<a_type, b_type, c_type, sf_type, VS>>
  236 | {
  237 |   using ValTypeA = sparse_elem<4,  uint8_t>;
  238 |   using ValTypeE = sparse_elem<16, uint8_t>;
  239 |   using ValTypeB = uint4_t;
  240 |   using FrgTypeA = sparse_elem<4,  uint8_t>;
  241 |   using FrgTypeE = sparse_elem<16, uint8_t>;
  242 | 
  243 |   using ValTypeC = c_type;
  244 |   using ValTypeD = c_type;
  245 | 
  246 |   using ValTypeSF = sf_type;
  247 | 
  248 |   constexpr static int SFVecSize = VS;
  249 | 
  250 |   using Shape_MNK = Shape<_16, _8, _128>;
  251 |   using ThrID     = Layout<_32>;
  252 |   // (T32,V64) -> (M16,K128)
  253 |   using ALayout   = Layout<Shape <Shape <  _4,_8>,Shape <_16,_2,   _2>>,
  254 |                            Stride<Stride<_256,_1>,Stride<_16,_8,_1024>>>;
  255 |   // (T32,V32) -> (N8,K128)
  256 |   using BLayout   = Layout<Shape <Shape < _4,_8>,Shape <_8,  _4>>,
  257 |                            Stride<Stride<_64,_1>,Stride<_8,_256>>>;
  258 |   // (T32,V128) -> (M16,K128)
  259 |   using SFALayout = Layout<Shape <Shape <_2,_2,_8>,_128>,
  260 |                            Stride<Stride<_8,_0,_1>, _16>>;
  261 |   // (T32,V128) -> (N8,K128)
  262 |   using SFBLayout = Layout<Shape <Shape <_4,_8>,_128>,
  263 |                            Stride<Stride<_0,_1>,  _8>>;
  264 |   // (T32,V4)  -> (M16,N8)
  265 |   using CLayout   = SM120_16x8_Row;
  266 |   // (T32, V64) -> (M16, K128) 
  267 |   using ELayout   = Layout<Shape <Shape <_2,   _2,_8>, Shape< _64>>,
  268 |                            Stride<Stride<_8,_1024,_1>,Stride<_16>>>;
  269 | };
```
**EN:** Specializes `MMA_Traits` for `SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x128_TN_VS<a_type, b_type, c_type, sf_type, VS`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x128_TN_VS<a_type, b_type, c_type, sf_type, VS` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 271-271

```text
  271 | namespace SM120::SPARSE {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 273-299

```text
  273 | // For SM120 MMA F8F6F4 input fp4, the operand A/B are load from ld.matrix. 
  274 | // ld.matrix b4x16_p64 places FP4 data at the first four bits in each
  275 | // eight-bit container, whereas MMA F8F6F4 expects the four-bit data to be in 
  276 | // the middle of the eight-bit container. Thus, e2m1 operands being fed
  277 | // to MMA F8F6F4 must be shifted left by two bits.
  278 | // 0b0000ABCD --> 0b00ABCD00
  279 | // NOTE: Same transformation is NOT needed for FP6 and FP8.
  280 | template<class AType, class BType, class... MMAArgs, class Tensor>
  281 | CUTLASS_DEVICE void
  282 | fp4_shift_A(SM120_SPARSE_16x8x64_TN<AType, BType, MMAArgs ...> const&, Tensor&& tensor) {
  283 |   using RegisterTypeA = typename remove_extent<typename
  284 |                         SM120_SPARSE_16x8x64_TN<AType, BType, MMAArgs ...>::ARegisters>::type;
  285 |   if constexpr (cute::is_same_v<AType, cutlass::float_e2m1_t>) {
  286 |     cute::transform(recast<RegisterTypeA>(tensor), [](RegisterTypeA& v){ return v << 2; });
  287 |   }
  288 | }
  289 | template<class AType, class BType, class... MMAArgs, class Tensor>
  290 | CUTLASS_DEVICE void
  291 | fp4_shift_B(SM120_SPARSE_16x8x64_TN<AType, BType, MMAArgs ...> const&, Tensor&& tensor) {
  292 |   using RegisterTypeB = typename remove_extent<typename
  293 |                         SM120_SPARSE_16x8x64_TN<AType, BType, MMAArgs ...>::BRegisters>::type;
  294 |   if constexpr (cute::is_same_v<BType, cutlass::float_e2m1_t>) {
  295 |     cute::transform(recast<RegisterTypeB>(tensor), [](RegisterTypeB& v){ return v << 2; });
  296 |   }
  297 | }
  299 | } // end namespace SM120::SPARSE
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 301-301

```text
  301 | namespace SM120::BLOCKSCALED::SPARSE {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 303-326

```text
  303 | // Template function with scale factor needs to enmuerate types one by one, as template 
  304 | // arguments contatins two variadic lists, which cannot be deduced in one shot.
  305 | template<class AType, class BType, class CType, class SFType, int VS, class Tensor>
  306 | CUTLASS_DEVICE void
  307 | fp4_shift_A(SM120_SPARSE_16x8x64_TN_VS<AType, BType, CType, SFType, VS> const&, Tensor&& tensor) {
  308 |   using RegisterTypeA = typename remove_extent<typename
  309 |                         SM120_SPARSE_16x8x64_TN_VS<AType, BType, CType, SFType, VS>::ARegisters>::type;
  310 |   if constexpr (cute::is_same_v<AType, cutlass::float_e2m1_t>) {
  311 |     cute::transform(recast<RegisterTypeA>(tensor), [](RegisterTypeA& v){ return v << 2; });
  312 |   }
  313 | }
  314 | template<class AType, class BType, class CType, class SFType, int VS, class Tensor>
  315 | CUTLASS_DEVICE void
  316 | fp4_shift_B(SM120_SPARSE_16x8x64_TN_VS<AType, BType, CType, SFType, VS> const&, Tensor&& tensor) {
  317 |   using RegisterTypeB = typename remove_extent<typename
  318 |                         SM120_SPARSE_16x8x64_TN_VS<AType, BType, CType, SFType, VS>::BRegisters>::type;
  319 |   if constexpr (cute::is_same_v<BType, cutlass::float_e2m1_t>) {
  320 |     cute::transform(recast<RegisterTypeB>(tensor), [](RegisterTypeB& v){ return v << 2; });
  321 |   }
  322 | }
  324 | } // end namespace SM120::BLOCKSCALED::SPARSE
  326 | } // end namespace cute
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Intel Xe-specific behavior / Intel Xe 特定行为
- Sparse logical-to-physical mapping / 稀疏逻辑到物理映射

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/arch/mma_sm120.hpp`
  - `cute/arch/mma_sm120_sparse.hpp`
  - `cute/atom/mma_traits.hpp`
  - `cute/layout.hpp`
  - `cute/numeric/numeric_types.hpp`
- Primary symbols / 主要符号: `MMA_Traits`, `MMAOp`, `TD`, `DLayout`, `TA`, `ALayout`, `TB`, `BLayout`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
