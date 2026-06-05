# mma_traits_sm90_gmma_sparse.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/atom/mma_traits_sm90_gmma_sparse.hpp`
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

### Lines 30-32

```text
   30 |  **************************************************************************************************/
   32 | #pragma once
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 34-42

```text
   34 | #include <cute/pointer_sparse.hpp>             // cute::smem_sparse_ptr_flag
   35 | #include <cute/swizzle.hpp>                    // cute::Swizzle
   36 | #include <cute/tensor_impl.hpp>                // cute::Tensor
   37 | #include <cute/arch/mma_sm90_desc.hpp>         // cute::LayoutType
   38 | #include <cute/arch/mma_sm90_gmma_sparse.hpp>  // cute::SM90::SPARSE::GMMA_64x8x32_F16F16F16_SS, etc
   39 | #include <cute/atom/mma_traits_sm90_gmma.hpp>  // cute::GMMA::Layout_*
   40 | #include <cute/atom/mma_traits.hpp>            // cute::MMA_Traits
   41 | #include <cute/layout_composed.hpp>            // cute::ComposedLayout
   42 | #include <cute/numeric/integral_constant.hpp>  // cute::is_static
```
**EN:** Sets up the header dependencies for this file by importing `cute/pointer_sparse.hpp`, `cute/swizzle.hpp`, `cute/tensor_impl.hpp`, `cute/arch/mma_sm90_desc.hpp`, `cute/arch/mma_sm90_gmma_sparse.hpp`, and 4 more include(s).
**CN:** 通过引入 `cute/pointer_sparse.hpp`, `cute/swizzle.hpp`, `cute/tensor_impl.hpp`, `cute/arch/mma_sm90_desc.hpp`, `cute/arch/mma_sm90_gmma_sparse.hpp`，以及另外 4 个头文件 为该文件建立头文件依赖。

### Lines 44-44

```text
   44 | namespace cute {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 46-46

```text
   46 | namespace SM90::GMMA {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 48-64

```text
   48 | ///////////////////////////////////////////
   49 | // Common layouts for GMMA Shared Memory //
   50 | ///////////////////////////////////////////
   52 | // M|N-major layouts in units of Type and sparsity factor S
   53 | template <class Type, int S>
   54 | using Layout_MN_INTER_SpAtom = ComposedLayout<Swizzle<0,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   55 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_MN_INTER_Atom<Type>{}.layout_b()))>;
   56 | template <class Type, int S>
   57 | using Layout_MN_SW32_SpAtom  = ComposedLayout<Swizzle<1,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   58 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_MN_SW32_Atom<Type>{}.layout_b()))>;
   59 | template <class Type, int S>
   60 | using Layout_MN_SW64_SpAtom  = ComposedLayout<Swizzle<2,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   61 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_MN_SW64_Atom<Type>{}.layout_b()))>;
   62 | template <class Type, int S>
   63 | using Layout_MN_SW128_SpAtom = ComposedLayout<Swizzle<3,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   64 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_MN_SW128_Atom<Type>{}.layout_b()))>;
```
**EN:** Defines swizzle-related transformations that reorder indices to match hardware-friendly memory access patterns.
**CN:** 定义与 swizzle 相关的变换，通过重排索引来匹配更适合硬件的内存访问模式。

### Lines 66-78

```text
   66 | // K-major layouts in units of Type and sparsity factor S
   67 | template <class Type, int S>
   68 | using Layout_K_INTER_SpAtom = ComposedLayout<Swizzle<0,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   69 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_K_INTER_Atom<Type>{}.layout_b()))>;
   70 | template <class Type, int S>
   71 | using Layout_K_SW32_SpAtom  = ComposedLayout<Swizzle<1,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   72 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_K_SW32_Atom<Type>{}.layout_b()))>;
   73 | template <class Type, int S>
   74 | using Layout_K_SW64_SpAtom  = ComposedLayout<Swizzle<2,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   75 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_K_SW64_Atom<Type>{}.layout_b()))>;
   76 | template <class Type, int S>
   77 | using Layout_K_SW128_SpAtom = ComposedLayout<Swizzle<3,4,3>, smem_sparse_ptr_flag_bits<S,sizeof_bits_v<Type>>,
   78 |                                               decltype(blocked_product(Layout<Shape<_1,Int<S>>>{}, Layout_K_SW128_Atom<Type>{}.layout_b()))>;
```
**EN:** Defines swizzle-related transformations that reorder indices to match hardware-friendly memory access patterns.
**CN:** 定义与 swizzle 相关的变换，通过重排索引来匹配更适合硬件的内存访问模式。

### Lines 80-96

```text
   80 | // With GMMA::Major param
   81 | template <class Type, int S, GMMA::Major tnsp>
   82 | using Layout_INTER_SpAtom = typename conditional<tnsp == GMMA::Major::MN,
   83 |                                                  Layout_MN_INTER_SpAtom<Type,S>,
   84 |                                                  Layout_K_INTER_SpAtom<Type,S>>::type;
   85 | template <class Type, int S, GMMA::Major tnsp>
   86 | using Layout_SW32_SpAtom = typename conditional<tnsp == GMMA::Major::MN,
   87 |                                                 Layout_MN_SW32_SpAtom<Type,S>,
   88 |                                                 Layout_K_SW32_SpAtom<Type,S>>::type;
   89 | template <class Type, int S, GMMA::Major tnsp>
   90 | using Layout_SW64_SpAtom = typename conditional<tnsp == GMMA::Major::MN,
   91 |                                                 Layout_MN_SW64_SpAtom<Type,S>,
   92 |                                                 Layout_K_SW64_SpAtom<Type,S>>::type;
   93 | template <class Type, int S, GMMA::Major tnsp>
   94 | using Layout_SW128_SpAtom = typename conditional<tnsp == GMMA::Major::MN,
   95 |                                                  Layout_MN_SW128_SpAtom<Type,S>,
   96 |                                                  Layout_K_SW128_SpAtom<Type,S>>::type;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 98-105

```text
   98 | ///////////////////////////////////////////////////////////////////////////////
   99 | // Higher level GMMA Descriptor utilities
  100 | ///////////////////////////////////////////////////////////////////////////////
  102 | template <GMMA::Major>
  103 | struct sparse_smem_desc : DescriptorIterator {};
  105 | } // end namespace SM90::GMMA
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 107-122

```text
  107 | // Customization point for creating a cute::GMMAsparse_smem_desc Tensor
  108 | template <SM90::GMMA::Major MajorMode>
  109 | struct MakeTensor<SM90::GMMA::sparse_smem_desc<MajorMode>>
  110 | {
  111 |   // Note that this is the exact same as cute::GMMAsmem_desc above, plus additional static checks.
  112 |   template <class TEngine, class TLayout>
  113 |   CUTE_HOST_DEVICE constexpr auto
  114 |   operator()(Tensor<TEngine,TLayout> const& smem_tensor)
  115 |   {
  116 |     static_assert(is_smem<TEngine>::value, "Expected SMEM Tensor to construct a GMMA Desc Tensor");
  117 |     static_assert(is_sparse<typename TEngine::value_type>::value, "Expected sparse value_type.");
  118 |     static_assert(is_sparse_ptr<TEngine>::value, "Expected sparse iter.");
  119 |     return make_tensor(SM90::GMMA::DescriptorIterator{SM90::GMMA::make_gmma_desc<MajorMode>(tensor<0>(smem_tensor))},
  120 |                        replace<0>(recast<uint128_t const>(smem_tensor).layout(), Layout<_1,_0>{}));
  121 |   }
  122 | };
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 124-128

```text
  124 | ///////////////////////////////////////////////////////////////////////////////
  125 | //////////////////////////// MMA_TRAITS ///////////////////////////////////////
  126 | ///////////////////////////////////////////////////////////////////////////////
  128 | namespace SM90::GMMA {
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 130-140

```text
  130 | // Metadata layouts
  131 | using ELayout_64x64  = Layout<Shape <Shape <_2,   _2,_8, _4>, Shape <_32>>, 
  132 |                               Stride<Stride<_8,_2048,_1,_16>, Stride<_64>>>;
  134 | using ELayout_64x32  = Layout<Shape <Shape <   _2,_2,_8, _4>, Shape <_16,_2>>, 
  135 |                               Stride<Stride<_1024,_0,_1,_16>, Stride<_64,_8>>>;
  137 | using ELayout_64x16  = Layout<Shape <Shape <  _2,_2,_8, _4>, Shape < _8,_2>>, 
  138 |                               Stride<Stride<_512,_0,_1,_16>, Stride<_64,_8>>>;
  140 | } // namespace SM90::GMMA
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 142-144

```text
  142 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  144 | namespace SM90::GMMA::SPARSE {
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 146-163

```text
  146 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  148 | template <class MMAOp,
  149 |           class TD, class DLayout,
  150 |           class TA, class ALayout,
  151 |           class TB, class BLayout,
  152 |           class TC, class CLayout>
  153 | CUTE_HOST_DEVICE constexpr void
  154 | mma_unpack(MMA_Traits<MMAOp>   const& traits,
  155 |            Tensor<TD, DLayout>      & D,
  156 |            Tensor<TA, ALayout> const& A_zipped,
  157 |            Tensor<TB, BLayout> const& B,
  158 |            Tensor<TC, CLayout> const& C)
  159 | {
  160 |   static_assert(is_rmem_v<TD>, "Expected registers in MMA_Atom::call");
  161 |   static_assert(is_rmem_v<TA>, "Expected registers in MMA_Atom::call");
  162 |   static_assert(is_rmem_v<TB>, "Expected registers in MMA_Atom::call");
  163 |   static_assert(is_rmem_v<TC>, "Expected registers in MMA_Atom::call");
```
**EN:** Specializes `MMA_Traits` for `MMAOp`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `MMAOp` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 165-176

```text
  165 |   using DRegisters = typename MMAOp::DRegisters;
  166 |   using ARegisters = typename MMAOp::ARegisters;
  167 |   using ERegisters = typename MMAOp::ERegisters;
  168 |   using BRegisters = typename MMAOp::BRegisters;
  169 |   using CRegisters = typename MMAOp::CRegisters;
  171 |   // Register value types from the MMAOp register arrays
  172 |   using RegTypeD   = typename remove_extent<DRegisters>::type;
  173 |   using RegTypeA   = typename remove_extent<ARegisters>::type;
  174 |   using RegTypeE   = typename remove_extent<ERegisters>::type;
  175 |   using RegTypeB   = typename remove_extent<BRegisters>::type;
  176 |   using RegTypeC   = typename remove_extent<CRegisters>::type;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 178-181

```text
  178 |   constexpr int RegNumA = extent<ARegisters>::value;
  179 |   constexpr int RegNumE = extent<ERegisters>::value;
  180 |   constexpr int RegNumB = extent<BRegisters>::value;
  181 |   constexpr int RegNumC = extent<CRegisters>::value;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 183-186

```text
  183 |   auto [A, E] = unzip_tensor(A_zipped);
  184 |   Tensor rA   = recast<RegTypeA>(A);
  185 |   Tensor rE   = recast<RegTypeE>(E);
  186 |   Tensor rB   = recast<RegTypeB>(B);
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 188-190

```text
  188 |   CUTE_STATIC_ASSERT_V(size(rA) == Int<RegNumA>{});
  189 |   CUTE_STATIC_ASSERT_V(size(rE) == Int<RegNumE>{});
  190 |   CUTE_STATIC_ASSERT_V(size(rB) == Int<RegNumB>{});
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 192-194

```text
  192 |   static_assert(is_same<RegTypeD, void>::value, "GMMA DRegisters must have void type.");
  193 |   static_assert(is_same<typename TD::value_type, typename TC::value_type>::value, "GMMA C and D value_type must match.");
  194 |   static_assert(is_same<DLayout, CLayout>::value, "GMMA C and D layouts must match.");
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 196-196

```text
  196 |   Tensor rC = recast<RegTypeC>(D);  // NOTE: D and C are same, so use mutable D
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 198-198

```text
  198 |   CUTE_STATIC_ASSERT_V(size(rC) == Int<RegNumC>{});
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 200-206

```text
  200 |   detail::explode(MMAOp::fma,
  201 |                   rA, make_int_sequence<RegNumA>{},
  202 |                   rB, make_int_sequence<RegNumB>{},
  203 |                   rC, make_int_sequence<RegNumC>{},
  204 |                   rE, make_int_sequence<RegNumE>{},
  205 |                   &(traits.accumulate_), seq<0>{});
  206 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 208-210

```text
  208 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  210 | } // namespace SM90::SPARSE
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 212-221

```text
  212 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  214 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  215 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  216 | {
  217 |   using ValTypeD = half_t;
  218 |   using ValTypeA = sparse_elem<2, half_t>;
  219 |   using ValTypeE = sparse_elem<8, uint8_t>;
  220 |   using ValTypeB = half_t;
  221 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 223-231

```text
  223 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  224 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  226 |   using Shape_MNK = Shape<_64,_8,_32>;
  227 |   using ThrID   = Layout<_128>;
  228 |   using ALayout = GMMA::ABLayout< 64, 32>;
  229 |   using ELayout = GMMA::ELayout_64x32;
  230 |   using BLayout = GMMA::ABLayout<  8, 32>;
  231 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 233-234

```text
  233 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  234 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 236-245

```text
  236 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  238 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  239 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  240 | {
  241 |   using ValTypeD = half_t;
  242 |   using ValTypeA = sparse_elem<2, half_t>;
  243 |   using ValTypeE = sparse_elem<8, uint8_t>;
  244 |   using ValTypeB = half_t;
  245 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 247-254

```text
  247 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  249 |   using Shape_MNK = Shape<_64,_8,_32>;
  250 |   using ThrID   = Layout<_128>;
  251 |   using ALayout = GMMA::ALayout_64x32;
  252 |   using ELayout = GMMA::ELayout_64x32;
  253 |   using BLayout = GMMA::ABLayout<  8, 32>;
  254 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 256-257

```text
  256 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  257 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 259-268

```text
  259 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  261 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  262 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  263 | {
  264 |   using ValTypeD = half_t;
  265 |   using ValTypeA = sparse_elem<2, half_t>;
  266 |   using ValTypeE = sparse_elem<8, uint8_t>;
  267 |   using ValTypeB = half_t;
  268 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 270-278

```text
  270 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  271 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  273 |   using Shape_MNK = Shape<_64,_16,_32>;
  274 |   using ThrID   = Layout<_128>;
  275 |   using ALayout = GMMA::ABLayout< 64, 32>;
  276 |   using ELayout = GMMA::ELayout_64x32;
  277 |   using BLayout = GMMA::ABLayout< 16, 32>;
  278 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 280-281

```text
  280 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  281 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 283-292

```text
  283 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  285 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  286 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  287 | {
  288 |   using ValTypeD = half_t;
  289 |   using ValTypeA = sparse_elem<2, half_t>;
  290 |   using ValTypeE = sparse_elem<8, uint8_t>;
  291 |   using ValTypeB = half_t;
  292 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 294-301

```text
  294 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  296 |   using Shape_MNK = Shape<_64,_16,_32>;
  297 |   using ThrID   = Layout<_128>;
  298 |   using ALayout = GMMA::ALayout_64x32;
  299 |   using ELayout = GMMA::ELayout_64x32;
  300 |   using BLayout = GMMA::ABLayout< 16, 32>;
  301 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 303-304

```text
  303 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  304 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 306-315

```text
  306 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  308 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  309 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  310 | {
  311 |   using ValTypeD = half_t;
  312 |   using ValTypeA = sparse_elem<2, half_t>;
  313 |   using ValTypeE = sparse_elem<8, uint8_t>;
  314 |   using ValTypeB = half_t;
  315 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 317-325

```text
  317 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  318 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  320 |   using Shape_MNK = Shape<_64,_32,_32>;
  321 |   using ThrID   = Layout<_128>;
  322 |   using ALayout = GMMA::ABLayout< 64, 32>;
  323 |   using ELayout = GMMA::ELayout_64x32;
  324 |   using BLayout = GMMA::ABLayout< 32, 32>;
  325 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 327-328

```text
  327 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  328 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 330-339

```text
  330 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  332 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  333 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  334 | {
  335 |   using ValTypeD = half_t;
  336 |   using ValTypeA = sparse_elem<2, half_t>;
  337 |   using ValTypeE = sparse_elem<8, uint8_t>;
  338 |   using ValTypeB = half_t;
  339 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 341-348

```text
  341 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  343 |   using Shape_MNK = Shape<_64,_32,_32>;
  344 |   using ThrID   = Layout<_128>;
  345 |   using ALayout = GMMA::ALayout_64x32;
  346 |   using ELayout = GMMA::ELayout_64x32;
  347 |   using BLayout = GMMA::ABLayout< 32, 32>;
  348 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 350-351

```text
  350 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  351 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 353-362

```text
  353 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  355 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  356 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  357 | {
  358 |   using ValTypeD = half_t;
  359 |   using ValTypeA = sparse_elem<2, half_t>;
  360 |   using ValTypeE = sparse_elem<8, uint8_t>;
  361 |   using ValTypeB = half_t;
  362 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 364-372

```text
  364 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  365 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  367 |   using Shape_MNK = Shape<_64,_64,_32>;
  368 |   using ThrID   = Layout<_128>;
  369 |   using ALayout = GMMA::ABLayout< 64, 32>;
  370 |   using ELayout = GMMA::ELayout_64x32;
  371 |   using BLayout = GMMA::ABLayout< 64, 32>;
  372 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 374-375

```text
  374 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  375 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 377-386

```text
  377 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  379 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  380 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  381 | {
  382 |   using ValTypeD = half_t;
  383 |   using ValTypeA = sparse_elem<2, half_t>;
  384 |   using ValTypeE = sparse_elem<8, uint8_t>;
  385 |   using ValTypeB = half_t;
  386 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 388-395

```text
  388 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  390 |   using Shape_MNK = Shape<_64,_64,_32>;
  391 |   using ThrID   = Layout<_128>;
  392 |   using ALayout = GMMA::ALayout_64x32;
  393 |   using ELayout = GMMA::ELayout_64x32;
  394 |   using BLayout = GMMA::ABLayout< 64, 32>;
  395 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 397-398

```text
  397 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  398 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 400-409

```text
  400 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  402 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  403 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  404 | {
  405 |   using ValTypeD = half_t;
  406 |   using ValTypeA = sparse_elem<2, half_t>;
  407 |   using ValTypeE = sparse_elem<8, uint8_t>;
  408 |   using ValTypeB = half_t;
  409 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 411-419

```text
  411 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  412 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  414 |   using Shape_MNK = Shape<_64,_96,_32>;
  415 |   using ThrID   = Layout<_128>;
  416 |   using ALayout = GMMA::ABLayout< 64, 32>;
  417 |   using ELayout = GMMA::ELayout_64x32;
  418 |   using BLayout = GMMA::ABLayout< 96, 32>;
  419 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 421-422

```text
  421 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  422 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 424-433

```text
  424 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  426 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  427 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  428 | {
  429 |   using ValTypeD = half_t;
  430 |   using ValTypeA = sparse_elem<2, half_t>;
  431 |   using ValTypeE = sparse_elem<8, uint8_t>;
  432 |   using ValTypeB = half_t;
  433 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 435-442

```text
  435 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  437 |   using Shape_MNK = Shape<_64,_96,_32>;
  438 |   using ThrID   = Layout<_128>;
  439 |   using ALayout = GMMA::ALayout_64x32;
  440 |   using ELayout = GMMA::ELayout_64x32;
  441 |   using BLayout = GMMA::ABLayout< 96, 32>;
  442 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 444-445

```text
  444 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  445 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 447-456

```text
  447 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  449 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  450 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  451 | {
  452 |   using ValTypeD = half_t;
  453 |   using ValTypeA = sparse_elem<2, half_t>;
  454 |   using ValTypeE = sparse_elem<8, uint8_t>;
  455 |   using ValTypeB = half_t;
  456 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 458-466

```text
  458 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  459 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  461 |   using Shape_MNK = Shape<_64,_128,_32>;
  462 |   using ThrID   = Layout<_128>;
  463 |   using ALayout = GMMA::ABLayout< 64, 32>;
  464 |   using ELayout = GMMA::ELayout_64x32;
  465 |   using BLayout = GMMA::ABLayout<128, 32>;
  466 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 468-469

```text
  468 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  469 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 471-480

```text
  471 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  473 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  474 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  475 | {
  476 |   using ValTypeD = half_t;
  477 |   using ValTypeA = sparse_elem<2, half_t>;
  478 |   using ValTypeE = sparse_elem<8, uint8_t>;
  479 |   using ValTypeB = half_t;
  480 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 482-489

```text
  482 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  484 |   using Shape_MNK = Shape<_64,_128,_32>;
  485 |   using ThrID   = Layout<_128>;
  486 |   using ALayout = GMMA::ALayout_64x32;
  487 |   using ELayout = GMMA::ELayout_64x32;
  488 |   using BLayout = GMMA::ABLayout<128, 32>;
  489 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 491-492

```text
  491 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  492 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 494-503

```text
  494 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  496 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  497 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  498 | {
  499 |   using ValTypeD = half_t;
  500 |   using ValTypeA = sparse_elem<2, half_t>;
  501 |   using ValTypeE = sparse_elem<8, uint8_t>;
  502 |   using ValTypeB = half_t;
  503 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 505-513

```text
  505 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  506 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  508 |   using Shape_MNK = Shape<_64,_192,_32>;
  509 |   using ThrID   = Layout<_128>;
  510 |   using ALayout = GMMA::ABLayout< 64, 32>;
  511 |   using ELayout = GMMA::ELayout_64x32;
  512 |   using BLayout = GMMA::ABLayout<192, 32>;
  513 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 515-516

```text
  515 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  516 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 518-527

```text
  518 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  520 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  521 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  522 | {
  523 |   using ValTypeD = half_t;
  524 |   using ValTypeA = sparse_elem<2, half_t>;
  525 |   using ValTypeE = sparse_elem<8, uint8_t>;
  526 |   using ValTypeB = half_t;
  527 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 529-536

```text
  529 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  531 |   using Shape_MNK = Shape<_64,_192,_32>;
  532 |   using ThrID   = Layout<_128>;
  533 |   using ALayout = GMMA::ALayout_64x32;
  534 |   using ELayout = GMMA::ELayout_64x32;
  535 |   using BLayout = GMMA::ABLayout<192, 32>;
  536 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 538-539

```text
  538 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  539 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 541-550

```text
  541 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  543 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  544 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  545 | {
  546 |   using ValTypeD = half_t;
  547 |   using ValTypeA = sparse_elem<2, half_t>;
  548 |   using ValTypeE = sparse_elem<8, uint8_t>;
  549 |   using ValTypeB = half_t;
  550 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x32_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 552-560

```text
  552 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  553 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  555 |   using Shape_MNK = Shape<_64,_256,_32>;
  556 |   using ThrID   = Layout<_128>;
  557 |   using ALayout = GMMA::ABLayout< 64, 32>;
  558 |   using ELayout = GMMA::ELayout_64x32;
  559 |   using BLayout = GMMA::ABLayout<256, 32>;
  560 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 562-563

```text
  562 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  563 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 565-574

```text
  565 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  567 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  568 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  569 | {
  570 |   using ValTypeD = half_t;
  571 |   using ValTypeA = sparse_elem<2, half_t>;
  572 |   using ValTypeE = sparse_elem<8, uint8_t>;
  573 |   using ValTypeB = half_t;
  574 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x32_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 576-583

```text
  576 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  578 |   using Shape_MNK = Shape<_64,_256,_32>;
  579 |   using ThrID   = Layout<_128>;
  580 |   using ALayout = GMMA::ALayout_64x32;
  581 |   using ELayout = GMMA::ELayout_64x32;
  582 |   using BLayout = GMMA::ABLayout<256, 32>;
  583 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 585-586

```text
  585 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  586 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 588-597

```text
  588 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  590 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  591 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  592 | {
  593 |   using ValTypeD = float;
  594 |   using ValTypeA = sparse_elem<2, half_t>;
  595 |   using ValTypeE = sparse_elem<8, uint8_t>;
  596 |   using ValTypeB = half_t;
  597 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 599-607

```text
  599 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  600 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  602 |   using Shape_MNK = Shape<_64,_8,_32>;
  603 |   using ThrID   = Layout<_128>;
  604 |   using ALayout = GMMA::ABLayout< 64, 32>;
  605 |   using ELayout = GMMA::ELayout_64x32;
  606 |   using BLayout = GMMA::ABLayout<  8, 32>;
  607 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 609-610

```text
  609 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  610 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 612-621

```text
  612 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  614 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  615 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  616 | {
  617 |   using ValTypeD = float;
  618 |   using ValTypeA = sparse_elem<2, half_t>;
  619 |   using ValTypeE = sparse_elem<8, uint8_t>;
  620 |   using ValTypeB = half_t;
  621 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 623-630

```text
  623 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  625 |   using Shape_MNK = Shape<_64,_8,_32>;
  626 |   using ThrID   = Layout<_128>;
  627 |   using ALayout = GMMA::ALayout_64x32;
  628 |   using ELayout = GMMA::ELayout_64x32;
  629 |   using BLayout = GMMA::ABLayout<  8, 32>;
  630 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 632-633

```text
  632 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  633 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 635-644

```text
  635 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  637 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  638 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  639 | {
  640 |   using ValTypeD = float;
  641 |   using ValTypeA = sparse_elem<2, half_t>;
  642 |   using ValTypeE = sparse_elem<8, uint8_t>;
  643 |   using ValTypeB = half_t;
  644 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 646-654

```text
  646 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  647 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  649 |   using Shape_MNK = Shape<_64,_16,_32>;
  650 |   using ThrID   = Layout<_128>;
  651 |   using ALayout = GMMA::ABLayout< 64, 32>;
  652 |   using ELayout = GMMA::ELayout_64x32;
  653 |   using BLayout = GMMA::ABLayout< 16, 32>;
  654 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 656-657

```text
  656 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  657 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 659-668

```text
  659 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  661 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  662 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  663 | {
  664 |   using ValTypeD = float;
  665 |   using ValTypeA = sparse_elem<2, half_t>;
  666 |   using ValTypeE = sparse_elem<8, uint8_t>;
  667 |   using ValTypeB = half_t;
  668 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 670-677

```text
  670 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  672 |   using Shape_MNK = Shape<_64,_16,_32>;
  673 |   using ThrID   = Layout<_128>;
  674 |   using ALayout = GMMA::ALayout_64x32;
  675 |   using ELayout = GMMA::ELayout_64x32;
  676 |   using BLayout = GMMA::ABLayout< 16, 32>;
  677 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 679-680

```text
  679 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  680 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 682-691

```text
  682 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  684 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  685 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  686 | {
  687 |   using ValTypeD = float;
  688 |   using ValTypeA = sparse_elem<2, half_t>;
  689 |   using ValTypeE = sparse_elem<8, uint8_t>;
  690 |   using ValTypeB = half_t;
  691 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 693-701

```text
  693 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  694 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  696 |   using Shape_MNK = Shape<_64,_32,_32>;
  697 |   using ThrID   = Layout<_128>;
  698 |   using ALayout = GMMA::ABLayout< 64, 32>;
  699 |   using ELayout = GMMA::ELayout_64x32;
  700 |   using BLayout = GMMA::ABLayout< 32, 32>;
  701 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 703-704

```text
  703 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  704 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 706-715

```text
  706 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  708 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  709 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  710 | {
  711 |   using ValTypeD = float;
  712 |   using ValTypeA = sparse_elem<2, half_t>;
  713 |   using ValTypeE = sparse_elem<8, uint8_t>;
  714 |   using ValTypeB = half_t;
  715 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 717-724

```text
  717 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  719 |   using Shape_MNK = Shape<_64,_32,_32>;
  720 |   using ThrID   = Layout<_128>;
  721 |   using ALayout = GMMA::ALayout_64x32;
  722 |   using ELayout = GMMA::ELayout_64x32;
  723 |   using BLayout = GMMA::ABLayout< 32, 32>;
  724 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 726-727

```text
  726 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  727 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 729-738

```text
  729 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  731 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  732 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  733 | {
  734 |   using ValTypeD = float;
  735 |   using ValTypeA = sparse_elem<2, half_t>;
  736 |   using ValTypeE = sparse_elem<8, uint8_t>;
  737 |   using ValTypeB = half_t;
  738 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 740-748

```text
  740 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  741 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  743 |   using Shape_MNK = Shape<_64,_64,_32>;
  744 |   using ThrID   = Layout<_128>;
  745 |   using ALayout = GMMA::ABLayout< 64, 32>;
  746 |   using ELayout = GMMA::ELayout_64x32;
  747 |   using BLayout = GMMA::ABLayout< 64, 32>;
  748 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 750-751

```text
  750 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  751 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 753-762

```text
  753 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  755 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  756 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  757 | {
  758 |   using ValTypeD = float;
  759 |   using ValTypeA = sparse_elem<2, half_t>;
  760 |   using ValTypeE = sparse_elem<8, uint8_t>;
  761 |   using ValTypeB = half_t;
  762 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 764-771

```text
  764 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  766 |   using Shape_MNK = Shape<_64,_64,_32>;
  767 |   using ThrID   = Layout<_128>;
  768 |   using ALayout = GMMA::ALayout_64x32;
  769 |   using ELayout = GMMA::ELayout_64x32;
  770 |   using BLayout = GMMA::ABLayout< 64, 32>;
  771 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 773-774

```text
  773 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  774 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 776-785

```text
  776 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  778 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  779 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  780 | {
  781 |   using ValTypeD = float;
  782 |   using ValTypeA = sparse_elem<2, half_t>;
  783 |   using ValTypeE = sparse_elem<8, uint8_t>;
  784 |   using ValTypeB = half_t;
  785 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 787-795

```text
  787 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  788 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  790 |   using Shape_MNK = Shape<_64,_96,_32>;
  791 |   using ThrID   = Layout<_128>;
  792 |   using ALayout = GMMA::ABLayout< 64, 32>;
  793 |   using ELayout = GMMA::ELayout_64x32;
  794 |   using BLayout = GMMA::ABLayout< 96, 32>;
  795 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 797-798

```text
  797 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  798 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 800-809

```text
  800 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  802 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  803 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  804 | {
  805 |   using ValTypeD = float;
  806 |   using ValTypeA = sparse_elem<2, half_t>;
  807 |   using ValTypeE = sparse_elem<8, uint8_t>;
  808 |   using ValTypeB = half_t;
  809 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 811-818

```text
  811 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  813 |   using Shape_MNK = Shape<_64,_96,_32>;
  814 |   using ThrID   = Layout<_128>;
  815 |   using ALayout = GMMA::ALayout_64x32;
  816 |   using ELayout = GMMA::ELayout_64x32;
  817 |   using BLayout = GMMA::ABLayout< 96, 32>;
  818 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 820-821

```text
  820 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  821 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 823-832

```text
  823 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  825 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  826 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  827 | {
  828 |   using ValTypeD = float;
  829 |   using ValTypeA = sparse_elem<2, half_t>;
  830 |   using ValTypeE = sparse_elem<8, uint8_t>;
  831 |   using ValTypeB = half_t;
  832 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 834-842

```text
  834 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  835 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  837 |   using Shape_MNK = Shape<_64,_128,_32>;
  838 |   using ThrID   = Layout<_128>;
  839 |   using ALayout = GMMA::ABLayout< 64, 32>;
  840 |   using ELayout = GMMA::ELayout_64x32;
  841 |   using BLayout = GMMA::ABLayout<128, 32>;
  842 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 844-845

```text
  844 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  845 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 847-856

```text
  847 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  849 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  850 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  851 | {
  852 |   using ValTypeD = float;
  853 |   using ValTypeA = sparse_elem<2, half_t>;
  854 |   using ValTypeE = sparse_elem<8, uint8_t>;
  855 |   using ValTypeB = half_t;
  856 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 858-865

```text
  858 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  860 |   using Shape_MNK = Shape<_64,_128,_32>;
  861 |   using ThrID   = Layout<_128>;
  862 |   using ALayout = GMMA::ALayout_64x32;
  863 |   using ELayout = GMMA::ELayout_64x32;
  864 |   using BLayout = GMMA::ABLayout<128, 32>;
  865 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 867-868

```text
  867 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  868 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 870-879

```text
  870 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  872 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  873 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  874 | {
  875 |   using ValTypeD = float;
  876 |   using ValTypeA = sparse_elem<2, half_t>;
  877 |   using ValTypeE = sparse_elem<8, uint8_t>;
  878 |   using ValTypeB = half_t;
  879 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 881-889

```text
  881 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  882 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  884 |   using Shape_MNK = Shape<_64,_192,_32>;
  885 |   using ThrID   = Layout<_128>;
  886 |   using ALayout = GMMA::ABLayout< 64, 32>;
  887 |   using ELayout = GMMA::ELayout_64x32;
  888 |   using BLayout = GMMA::ABLayout<192, 32>;
  889 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 891-892

```text
  891 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  892 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 894-903

```text
  894 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  896 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  897 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  898 | {
  899 |   using ValTypeD = float;
  900 |   using ValTypeA = sparse_elem<2, half_t>;
  901 |   using ValTypeE = sparse_elem<8, uint8_t>;
  902 |   using ValTypeB = half_t;
  903 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 905-912

```text
  905 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  907 |   using Shape_MNK = Shape<_64,_192,_32>;
  908 |   using ThrID   = Layout<_128>;
  909 |   using ALayout = GMMA::ALayout_64x32;
  910 |   using ELayout = GMMA::ELayout_64x32;
  911 |   using BLayout = GMMA::ABLayout<192, 32>;
  912 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 914-915

```text
  914 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  915 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 917-926

```text
  917 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  919 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  920 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  921 | {
  922 |   using ValTypeD = float;
  923 |   using ValTypeA = sparse_elem<2, half_t>;
  924 |   using ValTypeE = sparse_elem<8, uint8_t>;
  925 |   using ValTypeB = half_t;
  926 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x32_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 928-936

```text
  928 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  929 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  931 |   using Shape_MNK = Shape<_64,_256,_32>;
  932 |   using ThrID   = Layout<_128>;
  933 |   using ALayout = GMMA::ABLayout< 64, 32>;
  934 |   using ELayout = GMMA::ELayout_64x32;
  935 |   using BLayout = GMMA::ABLayout<256, 32>;
  936 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 938-939

```text
  938 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  939 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 941-950

```text
  941 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  943 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  944 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  945 | {
  946 |   using ValTypeD = float;
  947 |   using ValTypeA = sparse_elem<2, half_t>;
  948 |   using ValTypeE = sparse_elem<8, uint8_t>;
  949 |   using ValTypeB = half_t;
  950 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x32_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 952-959

```text
  952 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  954 |   using Shape_MNK = Shape<_64,_256,_32>;
  955 |   using ThrID   = Layout<_128>;
  956 |   using ALayout = GMMA::ALayout_64x32;
  957 |   using ELayout = GMMA::ELayout_64x32;
  958 |   using BLayout = GMMA::ABLayout<256, 32>;
  959 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 961-962

```text
  961 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  962 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 964-973

```text
  964 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  966 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  967 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
  968 | {
  969 |   using ValTypeD = float;
  970 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
  971 |   using ValTypeE = sparse_elem<8, uint8_t>;
  972 |   using ValTypeB = bfloat16_t;
  973 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 975-983

```text
  975 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  976 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  978 |   using Shape_MNK = Shape<_64,_8,_32>;
  979 |   using ThrID   = Layout<_128>;
  980 |   using ALayout = GMMA::ABLayout< 64, 32>;
  981 |   using ELayout = GMMA::ELayout_64x32;
  982 |   using BLayout = GMMA::ABLayout<  8, 32>;
  983 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 985-986

```text
  985 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  986 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 988-997

```text
  988 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  990 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
  991 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
  992 | {
  993 |   using ValTypeD = float;
  994 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
  995 |   using ValTypeE = sparse_elem<8, uint8_t>;
  996 |   using ValTypeB = bfloat16_t;
  997 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 999-1006

```text
  999 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1001 |   using Shape_MNK = Shape<_64,_8,_32>;
 1002 |   using ThrID   = Layout<_128>;
 1003 |   using ALayout = GMMA::ALayout_64x32;
 1004 |   using ELayout = GMMA::ELayout_64x32;
 1005 |   using BLayout = GMMA::ABLayout<  8, 32>;
 1006 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1008-1009

```text
 1008 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1009 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1011-1020

```text
 1011 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1013 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1014 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1015 | {
 1016 |   using ValTypeD = float;
 1017 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1018 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1019 |   using ValTypeB = bfloat16_t;
 1020 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1022-1030

```text
 1022 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1023 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1025 |   using Shape_MNK = Shape<_64,_16,_32>;
 1026 |   using ThrID   = Layout<_128>;
 1027 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1028 |   using ELayout = GMMA::ELayout_64x32;
 1029 |   using BLayout = GMMA::ABLayout< 16, 32>;
 1030 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1032-1033

```text
 1032 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1033 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1035-1044

```text
 1035 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1037 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1038 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1039 | {
 1040 |   using ValTypeD = float;
 1041 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1042 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1043 |   using ValTypeB = bfloat16_t;
 1044 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1046-1053

```text
 1046 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1048 |   using Shape_MNK = Shape<_64,_16,_32>;
 1049 |   using ThrID   = Layout<_128>;
 1050 |   using ALayout = GMMA::ALayout_64x32;
 1051 |   using ELayout = GMMA::ELayout_64x32;
 1052 |   using BLayout = GMMA::ABLayout< 16, 32>;
 1053 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1055-1056

```text
 1055 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1056 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1058-1067

```text
 1058 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1060 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1061 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1062 | {
 1063 |   using ValTypeD = float;
 1064 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1065 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1066 |   using ValTypeB = bfloat16_t;
 1067 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1069-1077

```text
 1069 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1070 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1072 |   using Shape_MNK = Shape<_64,_32,_32>;
 1073 |   using ThrID   = Layout<_128>;
 1074 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1075 |   using ELayout = GMMA::ELayout_64x32;
 1076 |   using BLayout = GMMA::ABLayout< 32, 32>;
 1077 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1079-1080

```text
 1079 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1080 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1082-1091

```text
 1082 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1084 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1085 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1086 | {
 1087 |   using ValTypeD = float;
 1088 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1089 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1090 |   using ValTypeB = bfloat16_t;
 1091 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1093-1100

```text
 1093 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1095 |   using Shape_MNK = Shape<_64,_32,_32>;
 1096 |   using ThrID   = Layout<_128>;
 1097 |   using ALayout = GMMA::ALayout_64x32;
 1098 |   using ELayout = GMMA::ELayout_64x32;
 1099 |   using BLayout = GMMA::ABLayout< 32, 32>;
 1100 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1102-1103

```text
 1102 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1103 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1105-1114

```text
 1105 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1107 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1108 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1109 | {
 1110 |   using ValTypeD = float;
 1111 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1112 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1113 |   using ValTypeB = bfloat16_t;
 1114 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1116-1124

```text
 1116 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1117 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1119 |   using Shape_MNK = Shape<_64,_64,_32>;
 1120 |   using ThrID   = Layout<_128>;
 1121 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1122 |   using ELayout = GMMA::ELayout_64x32;
 1123 |   using BLayout = GMMA::ABLayout< 64, 32>;
 1124 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1126-1127

```text
 1126 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1127 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1129-1138

```text
 1129 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1131 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1132 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1133 | {
 1134 |   using ValTypeD = float;
 1135 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1136 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1137 |   using ValTypeB = bfloat16_t;
 1138 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1140-1147

```text
 1140 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1142 |   using Shape_MNK = Shape<_64,_64,_32>;
 1143 |   using ThrID   = Layout<_128>;
 1144 |   using ALayout = GMMA::ALayout_64x32;
 1145 |   using ELayout = GMMA::ELayout_64x32;
 1146 |   using BLayout = GMMA::ABLayout< 64, 32>;
 1147 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1149-1150

```text
 1149 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1150 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1152-1161

```text
 1152 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1154 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1155 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1156 | {
 1157 |   using ValTypeD = float;
 1158 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1159 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1160 |   using ValTypeB = bfloat16_t;
 1161 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1163-1171

```text
 1163 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1164 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1166 |   using Shape_MNK = Shape<_64,_96,_32>;
 1167 |   using ThrID   = Layout<_128>;
 1168 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1169 |   using ELayout = GMMA::ELayout_64x32;
 1170 |   using BLayout = GMMA::ABLayout< 96, 32>;
 1171 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1173-1174

```text
 1173 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1174 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1176-1185

```text
 1176 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1178 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1179 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1180 | {
 1181 |   using ValTypeD = float;
 1182 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1183 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1184 |   using ValTypeB = bfloat16_t;
 1185 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1187-1194

```text
 1187 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1189 |   using Shape_MNK = Shape<_64,_96,_32>;
 1190 |   using ThrID   = Layout<_128>;
 1191 |   using ALayout = GMMA::ALayout_64x32;
 1192 |   using ELayout = GMMA::ELayout_64x32;
 1193 |   using BLayout = GMMA::ABLayout< 96, 32>;
 1194 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1196-1197

```text
 1196 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1197 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1199-1208

```text
 1199 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1201 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1202 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1203 | {
 1204 |   using ValTypeD = float;
 1205 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1206 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1207 |   using ValTypeB = bfloat16_t;
 1208 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1210-1218

```text
 1210 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1211 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1213 |   using Shape_MNK = Shape<_64,_128,_32>;
 1214 |   using ThrID   = Layout<_128>;
 1215 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1216 |   using ELayout = GMMA::ELayout_64x32;
 1217 |   using BLayout = GMMA::ABLayout<128, 32>;
 1218 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1220-1221

```text
 1220 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1221 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1223-1232

```text
 1223 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1225 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1226 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1227 | {
 1228 |   using ValTypeD = float;
 1229 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1230 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1231 |   using ValTypeB = bfloat16_t;
 1232 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1234-1241

```text
 1234 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1236 |   using Shape_MNK = Shape<_64,_128,_32>;
 1237 |   using ThrID   = Layout<_128>;
 1238 |   using ALayout = GMMA::ALayout_64x32;
 1239 |   using ELayout = GMMA::ELayout_64x32;
 1240 |   using BLayout = GMMA::ABLayout<128, 32>;
 1241 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1243-1244

```text
 1243 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1244 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1246-1255

```text
 1246 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1248 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1249 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1250 | {
 1251 |   using ValTypeD = float;
 1252 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1253 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1254 |   using ValTypeB = bfloat16_t;
 1255 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1257-1265

```text
 1257 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1258 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1260 |   using Shape_MNK = Shape<_64,_192,_32>;
 1261 |   using ThrID   = Layout<_128>;
 1262 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1263 |   using ELayout = GMMA::ELayout_64x32;
 1264 |   using BLayout = GMMA::ABLayout<192, 32>;
 1265 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1267-1268

```text
 1267 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1268 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1270-1279

```text
 1270 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1272 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1273 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1274 | {
 1275 |   using ValTypeD = float;
 1276 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1277 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1278 |   using ValTypeB = bfloat16_t;
 1279 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1281-1288

```text
 1281 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1283 |   using Shape_MNK = Shape<_64,_192,_32>;
 1284 |   using ThrID   = Layout<_128>;
 1285 |   using ALayout = GMMA::ALayout_64x32;
 1286 |   using ELayout = GMMA::ELayout_64x32;
 1287 |   using BLayout = GMMA::ABLayout<192, 32>;
 1288 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1290-1291

```text
 1290 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1291 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1293-1302

```text
 1293 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1295 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1296 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1297 | {
 1298 |   using ValTypeD = float;
 1299 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1300 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1301 |   using ValTypeB = bfloat16_t;
 1302 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x32_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1304-1312

```text
 1304 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1305 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1307 |   using Shape_MNK = Shape<_64,_256,_32>;
 1308 |   using ThrID   = Layout<_128>;
 1309 |   using ALayout = GMMA::ABLayout< 64, 32>;
 1310 |   using ELayout = GMMA::ELayout_64x32;
 1311 |   using BLayout = GMMA::ABLayout<256, 32>;
 1312 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1314-1315

```text
 1314 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1315 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1317-1326

```text
 1317 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1319 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1320 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel>>
 1321 | {
 1322 |   using ValTypeD = float;
 1323 |   using ValTypeA = sparse_elem<2, bfloat16_t>;
 1324 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1325 |   using ValTypeB = bfloat16_t;
 1326 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x32_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1328-1335

```text
 1328 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1330 |   using Shape_MNK = Shape<_64,_256,_32>;
 1331 |   using ThrID   = Layout<_128>;
 1332 |   using ALayout = GMMA::ALayout_64x32;
 1333 |   using ELayout = GMMA::ELayout_64x32;
 1334 |   using BLayout = GMMA::ABLayout<256, 32>;
 1335 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1337-1338

```text
 1337 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1338 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1340-1349

```text
 1340 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1342 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1343 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1344 | {
 1345 |   using ValTypeD = float;
 1346 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1347 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1348 |   using ValTypeB = tfloat32_t;
 1349 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1351-1359

```text
 1351 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1352 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1354 |   using Shape_MNK = Shape<_64,_8,_16>;
 1355 |   using ThrID   = Layout<_128>;
 1356 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1357 |   using ELayout = GMMA::ELayout_64x16;
 1358 |   using BLayout = GMMA::ABLayout<  8, 16>;
 1359 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1361-1362

```text
 1361 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1362 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1364-1373

```text
 1364 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1366 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1367 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1368 | {
 1369 |   using ValTypeD = float;
 1370 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1371 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1372 |   using ValTypeB = tfloat32_t;
 1373 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1375-1382

```text
 1375 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1377 |   using Shape_MNK = Shape<_64,_8,_16>;
 1378 |   using ThrID   = Layout<_128>;
 1379 |   using ALayout = GMMA::ALayout_64x16;
 1380 |   using ELayout = GMMA::ELayout_64x16;
 1381 |   using BLayout = GMMA::ABLayout<  8, 16>;
 1382 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1384-1385

```text
 1384 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1385 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1387-1396

```text
 1387 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1389 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1390 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1391 | {
 1392 |   using ValTypeD = float;
 1393 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1394 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1395 |   using ValTypeB = tfloat32_t;
 1396 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1398-1406

```text
 1398 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1399 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1401 |   using Shape_MNK = Shape<_64,_16,_16>;
 1402 |   using ThrID   = Layout<_128>;
 1403 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1404 |   using ELayout = GMMA::ELayout_64x16;
 1405 |   using BLayout = GMMA::ABLayout< 16, 16>;
 1406 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1408-1409

```text
 1408 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1409 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1411-1420

```text
 1411 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1413 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1414 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1415 | {
 1416 |   using ValTypeD = float;
 1417 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1418 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1419 |   using ValTypeB = tfloat32_t;
 1420 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1422-1429

```text
 1422 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1424 |   using Shape_MNK = Shape<_64,_16,_16>;
 1425 |   using ThrID   = Layout<_128>;
 1426 |   using ALayout = GMMA::ALayout_64x16;
 1427 |   using ELayout = GMMA::ELayout_64x16;
 1428 |   using BLayout = GMMA::ABLayout< 16, 16>;
 1429 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1431-1432

```text
 1431 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1432 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1434-1443

```text
 1434 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1436 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1437 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1438 | {
 1439 |   using ValTypeD = float;
 1440 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1441 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1442 |   using ValTypeB = tfloat32_t;
 1443 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1445-1453

```text
 1445 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1446 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1448 |   using Shape_MNK = Shape<_64,_32,_16>;
 1449 |   using ThrID   = Layout<_128>;
 1450 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1451 |   using ELayout = GMMA::ELayout_64x16;
 1452 |   using BLayout = GMMA::ABLayout< 32, 16>;
 1453 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1455-1456

```text
 1455 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1456 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1458-1467

```text
 1458 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1460 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1461 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1462 | {
 1463 |   using ValTypeD = float;
 1464 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1465 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1466 |   using ValTypeB = tfloat32_t;
 1467 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1469-1476

```text
 1469 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1471 |   using Shape_MNK = Shape<_64,_32,_16>;
 1472 |   using ThrID   = Layout<_128>;
 1473 |   using ALayout = GMMA::ALayout_64x16;
 1474 |   using ELayout = GMMA::ELayout_64x16;
 1475 |   using BLayout = GMMA::ABLayout< 32, 16>;
 1476 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1478-1479

```text
 1478 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1479 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1481-1490

```text
 1481 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1483 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1484 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1485 | {
 1486 |   using ValTypeD = float;
 1487 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1488 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1489 |   using ValTypeB = tfloat32_t;
 1490 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1492-1500

```text
 1492 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1493 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1495 |   using Shape_MNK = Shape<_64,_64,_16>;
 1496 |   using ThrID   = Layout<_128>;
 1497 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1498 |   using ELayout = GMMA::ELayout_64x16;
 1499 |   using BLayout = GMMA::ABLayout< 64, 16>;
 1500 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1502-1503

```text
 1502 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1503 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1505-1514

```text
 1505 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1507 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1508 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1509 | {
 1510 |   using ValTypeD = float;
 1511 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1512 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1513 |   using ValTypeB = tfloat32_t;
 1514 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1516-1523

```text
 1516 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1518 |   using Shape_MNK = Shape<_64,_64,_16>;
 1519 |   using ThrID   = Layout<_128>;
 1520 |   using ALayout = GMMA::ALayout_64x16;
 1521 |   using ELayout = GMMA::ELayout_64x16;
 1522 |   using BLayout = GMMA::ABLayout< 64, 16>;
 1523 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1525-1526

```text
 1525 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1526 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1528-1537

```text
 1528 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1530 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1531 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1532 | {
 1533 |   using ValTypeD = float;
 1534 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1535 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1536 |   using ValTypeB = tfloat32_t;
 1537 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1539-1547

```text
 1539 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1540 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1542 |   using Shape_MNK = Shape<_64,_96,_16>;
 1543 |   using ThrID   = Layout<_128>;
 1544 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1545 |   using ELayout = GMMA::ELayout_64x16;
 1546 |   using BLayout = GMMA::ABLayout< 96, 16>;
 1547 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1549-1550

```text
 1549 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1550 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1552-1561

```text
 1552 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1554 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1555 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1556 | {
 1557 |   using ValTypeD = float;
 1558 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1559 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1560 |   using ValTypeB = tfloat32_t;
 1561 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1563-1570

```text
 1563 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1565 |   using Shape_MNK = Shape<_64,_96,_16>;
 1566 |   using ThrID   = Layout<_128>;
 1567 |   using ALayout = GMMA::ALayout_64x16;
 1568 |   using ELayout = GMMA::ELayout_64x16;
 1569 |   using BLayout = GMMA::ABLayout< 96, 16>;
 1570 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1572-1573

```text
 1572 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1573 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1575-1584

```text
 1575 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1577 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1578 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1579 | {
 1580 |   using ValTypeD = float;
 1581 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1582 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1583 |   using ValTypeB = tfloat32_t;
 1584 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1586-1594

```text
 1586 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1587 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1589 |   using Shape_MNK = Shape<_64,_128,_16>;
 1590 |   using ThrID   = Layout<_128>;
 1591 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1592 |   using ELayout = GMMA::ELayout_64x16;
 1593 |   using BLayout = GMMA::ABLayout<128, 16>;
 1594 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1596-1597

```text
 1596 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1597 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1599-1608

```text
 1599 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1601 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1602 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1603 | {
 1604 |   using ValTypeD = float;
 1605 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1606 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1607 |   using ValTypeB = tfloat32_t;
 1608 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1610-1617

```text
 1610 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1612 |   using Shape_MNK = Shape<_64,_128,_16>;
 1613 |   using ThrID   = Layout<_128>;
 1614 |   using ALayout = GMMA::ALayout_64x16;
 1615 |   using ELayout = GMMA::ELayout_64x16;
 1616 |   using BLayout = GMMA::ABLayout<128, 16>;
 1617 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1619-1620

```text
 1619 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1620 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1622-1631

```text
 1622 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1624 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1625 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1626 | {
 1627 |   using ValTypeD = float;
 1628 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1629 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1630 |   using ValTypeB = tfloat32_t;
 1631 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1633-1641

```text
 1633 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1634 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1636 |   using Shape_MNK = Shape<_64,_192,_16>;
 1637 |   using ThrID   = Layout<_128>;
 1638 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1639 |   using ELayout = GMMA::ELayout_64x16;
 1640 |   using BLayout = GMMA::ABLayout<192, 16>;
 1641 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1643-1644

```text
 1643 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1644 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1646-1655

```text
 1646 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1648 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1649 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1650 | {
 1651 |   using ValTypeD = float;
 1652 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1653 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1654 |   using ValTypeB = tfloat32_t;
 1655 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1657-1664

```text
 1657 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1659 |   using Shape_MNK = Shape<_64,_192,_16>;
 1660 |   using ThrID   = Layout<_128>;
 1661 |   using ALayout = GMMA::ALayout_64x16;
 1662 |   using ELayout = GMMA::ELayout_64x16;
 1663 |   using BLayout = GMMA::ABLayout<192, 16>;
 1664 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1666-1667

```text
 1666 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1667 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1669-1678

```text
 1669 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1671 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1672 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel>>
 1673 | {
 1674 |   using ValTypeD = float;
 1675 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1676 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1677 |   using ValTypeB = tfloat32_t;
 1678 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x16_F32TF32TF32_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1680-1688

```text
 1680 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1681 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1683 |   using Shape_MNK = Shape<_64,_256,_16>;
 1684 |   using ThrID   = Layout<_128>;
 1685 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1686 |   using ELayout = GMMA::ELayout_64x16;
 1687 |   using BLayout = GMMA::ABLayout<256, 16>;
 1688 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1690-1691

```text
 1690 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1691 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1693-1702

```text
 1693 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1695 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 1696 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel>>
 1697 | {
 1698 |   using ValTypeD = float;
 1699 |   using ValTypeA = sparse_elem<2, tfloat32_t>;
 1700 |   using ValTypeE = sparse_elem<4, uint8_t>;
 1701 |   using ValTypeB = tfloat32_t;
 1702 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x16_F32TF32TF32_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1704-1711

```text
 1704 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1706 |   using Shape_MNK = Shape<_64,_256,_16>;
 1707 |   using ThrID   = Layout<_128>;
 1708 |   using ALayout = GMMA::ALayout_64x16;
 1709 |   using ELayout = GMMA::ELayout_64x16;
 1710 |   using BLayout = GMMA::ABLayout<256, 16>;
 1711 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1713-1714

```text
 1713 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1714 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1716-1725

```text
 1716 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1718 | template <GMMA::SparseSel spsel>
 1719 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_SS_TN<spsel>>
 1720 | {
 1721 |   using ValTypeD = int32_t;
 1722 |   using ValTypeA = sparse_elem<2, int8_t>;
 1723 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1724 |   using ValTypeB = int8_t;
 1725 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1727-1735

```text
 1727 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1728 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1730 |   using Shape_MNK = Shape<_64,_8,_64>;
 1731 |   using ThrID   = Layout<_128>;
 1732 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1733 |   using ELayout = GMMA::ELayout_64x64;
 1734 |   using BLayout = GMMA::ABLayout<  8, 64>;
 1735 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1737-1738

```text
 1737 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1738 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1740-1749

```text
 1740 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1742 | template <GMMA::SparseSel spsel>
 1743 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_SS_TN_SATURATE<spsel>>
 1744 | {
 1745 |   using ValTypeD = int32_t;
 1746 |   using ValTypeA = sparse_elem<2, int8_t>;
 1747 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1748 |   using ValTypeB = int8_t;
 1749 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1751-1759

```text
 1751 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1752 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1754 |   using Shape_MNK = Shape<_64,_8,_64>;
 1755 |   using ThrID   = Layout<_128>;
 1756 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1757 |   using ELayout = GMMA::ELayout_64x64;
 1758 |   using BLayout = GMMA::ABLayout<  8, 64>;
 1759 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1761-1762

```text
 1761 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1762 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1764-1773

```text
 1764 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1766 | template <GMMA::SparseSel spsel>
 1767 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_SS_TN<spsel>>
 1768 | {
 1769 |   using ValTypeD = int32_t;
 1770 |   using ValTypeA = sparse_elem<2, int8_t>;
 1771 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1772 |   using ValTypeB = int8_t;
 1773 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1775-1783

```text
 1775 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1776 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1778 |   using Shape_MNK = Shape<_64,_16,_64>;
 1779 |   using ThrID   = Layout<_128>;
 1780 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1781 |   using ELayout = GMMA::ELayout_64x64;
 1782 |   using BLayout = GMMA::ABLayout< 16, 64>;
 1783 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1785-1786

```text
 1785 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1786 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1788-1797

```text
 1788 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1790 | template <GMMA::SparseSel spsel>
 1791 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_SS_TN_SATURATE<spsel>>
 1792 | {
 1793 |   using ValTypeD = int32_t;
 1794 |   using ValTypeA = sparse_elem<2, int8_t>;
 1795 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1796 |   using ValTypeB = int8_t;
 1797 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1799-1807

```text
 1799 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1800 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1802 |   using Shape_MNK = Shape<_64,_16,_64>;
 1803 |   using ThrID   = Layout<_128>;
 1804 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1805 |   using ELayout = GMMA::ELayout_64x64;
 1806 |   using BLayout = GMMA::ABLayout< 16, 64>;
 1807 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1809-1810

```text
 1809 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1810 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1812-1821

```text
 1812 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1814 | template <GMMA::SparseSel spsel>
 1815 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_SS_TN<spsel>>
 1816 | {
 1817 |   using ValTypeD = int32_t;
 1818 |   using ValTypeA = sparse_elem<2, int8_t>;
 1819 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1820 |   using ValTypeB = int8_t;
 1821 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1823-1831

```text
 1823 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1824 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1826 |   using Shape_MNK = Shape<_64,_32,_64>;
 1827 |   using ThrID   = Layout<_128>;
 1828 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1829 |   using ELayout = GMMA::ELayout_64x64;
 1830 |   using BLayout = GMMA::ABLayout< 32, 64>;
 1831 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1833-1834

```text
 1833 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1834 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1836-1845

```text
 1836 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1838 | template <GMMA::SparseSel spsel>
 1839 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_SS_TN_SATURATE<spsel>>
 1840 | {
 1841 |   using ValTypeD = int32_t;
 1842 |   using ValTypeA = sparse_elem<2, int8_t>;
 1843 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1844 |   using ValTypeB = int8_t;
 1845 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1847-1855

```text
 1847 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1848 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1850 |   using Shape_MNK = Shape<_64,_32,_64>;
 1851 |   using ThrID   = Layout<_128>;
 1852 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1853 |   using ELayout = GMMA::ELayout_64x64;
 1854 |   using BLayout = GMMA::ABLayout< 32, 64>;
 1855 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1857-1858

```text
 1857 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1858 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1860-1869

```text
 1860 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1862 | template <GMMA::SparseSel spsel>
 1863 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_SS_TN<spsel>>
 1864 | {
 1865 |   using ValTypeD = int32_t;
 1866 |   using ValTypeA = sparse_elem<2, int8_t>;
 1867 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1868 |   using ValTypeB = int8_t;
 1869 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1871-1879

```text
 1871 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1872 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1874 |   using Shape_MNK = Shape<_64,_64,_64>;
 1875 |   using ThrID   = Layout<_128>;
 1876 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1877 |   using ELayout = GMMA::ELayout_64x64;
 1878 |   using BLayout = GMMA::ABLayout< 64, 64>;
 1879 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1881-1882

```text
 1881 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1882 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1884-1893

```text
 1884 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1886 | template <GMMA::SparseSel spsel>
 1887 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_SS_TN_SATURATE<spsel>>
 1888 | {
 1889 |   using ValTypeD = int32_t;
 1890 |   using ValTypeA = sparse_elem<2, int8_t>;
 1891 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1892 |   using ValTypeB = int8_t;
 1893 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1895-1903

```text
 1895 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1896 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1898 |   using Shape_MNK = Shape<_64,_64,_64>;
 1899 |   using ThrID   = Layout<_128>;
 1900 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1901 |   using ELayout = GMMA::ELayout_64x64;
 1902 |   using BLayout = GMMA::ABLayout< 64, 64>;
 1903 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1905-1906

```text
 1905 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1906 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1908-1917

```text
 1908 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1910 | template <GMMA::SparseSel spsel>
 1911 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_SS_TN<spsel>>
 1912 | {
 1913 |   using ValTypeD = int32_t;
 1914 |   using ValTypeA = sparse_elem<2, int8_t>;
 1915 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1916 |   using ValTypeB = int8_t;
 1917 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1919-1927

```text
 1919 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1920 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1922 |   using Shape_MNK = Shape<_64,_96,_64>;
 1923 |   using ThrID   = Layout<_128>;
 1924 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1925 |   using ELayout = GMMA::ELayout_64x64;
 1926 |   using BLayout = GMMA::ABLayout< 96, 64>;
 1927 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1929-1930

```text
 1929 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1930 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1932-1941

```text
 1932 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1934 | template <GMMA::SparseSel spsel>
 1935 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_SS_TN_SATURATE<spsel>>
 1936 | {
 1937 |   using ValTypeD = int32_t;
 1938 |   using ValTypeA = sparse_elem<2, int8_t>;
 1939 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1940 |   using ValTypeB = int8_t;
 1941 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1943-1951

```text
 1943 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1944 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1946 |   using Shape_MNK = Shape<_64,_96,_64>;
 1947 |   using ThrID   = Layout<_128>;
 1948 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1949 |   using ELayout = GMMA::ELayout_64x64;
 1950 |   using BLayout = GMMA::ABLayout< 96, 64>;
 1951 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1953-1954

```text
 1953 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1954 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1956-1965

```text
 1956 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1958 | template <GMMA::SparseSel spsel>
 1959 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_SS_TN<spsel>>
 1960 | {
 1961 |   using ValTypeD = int32_t;
 1962 |   using ValTypeA = sparse_elem<2, int8_t>;
 1963 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1964 |   using ValTypeB = int8_t;
 1965 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1967-1975

```text
 1967 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1968 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1970 |   using Shape_MNK = Shape<_64,_128,_64>;
 1971 |   using ThrID   = Layout<_128>;
 1972 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1973 |   using ELayout = GMMA::ELayout_64x64;
 1974 |   using BLayout = GMMA::ABLayout<128, 64>;
 1975 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1977-1978

```text
 1977 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1978 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1980-1989

```text
 1980 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1982 | template <GMMA::SparseSel spsel>
 1983 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_SS_TN_SATURATE<spsel>>
 1984 | {
 1985 |   using ValTypeD = int32_t;
 1986 |   using ValTypeA = sparse_elem<2, int8_t>;
 1987 |   using ValTypeE = sparse_elem<8, uint8_t>;
 1988 |   using ValTypeB = int8_t;
 1989 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1991-1999

```text
 1991 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1992 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1994 |   using Shape_MNK = Shape<_64,_128,_64>;
 1995 |   using ThrID   = Layout<_128>;
 1996 |   using ALayout = GMMA::ABLayout< 64, 64>;
 1997 |   using ELayout = GMMA::ELayout_64x64;
 1998 |   using BLayout = GMMA::ABLayout<128, 64>;
 1999 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2001-2002

```text
 2001 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2002 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2004-2013

```text
 2004 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2006 | template <GMMA::SparseSel spsel>
 2007 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_SS_TN<spsel>>
 2008 | {
 2009 |   using ValTypeD = int32_t;
 2010 |   using ValTypeA = sparse_elem<2, int8_t>;
 2011 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2012 |   using ValTypeB = int8_t;
 2013 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2015-2023

```text
 2015 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2016 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2018 |   using Shape_MNK = Shape<_64,_192,_64>;
 2019 |   using ThrID   = Layout<_128>;
 2020 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2021 |   using ELayout = GMMA::ELayout_64x64;
 2022 |   using BLayout = GMMA::ABLayout<192, 64>;
 2023 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2025-2026

```text
 2025 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2026 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2028-2037

```text
 2028 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2030 | template <GMMA::SparseSel spsel>
 2031 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_SS_TN_SATURATE<spsel>>
 2032 | {
 2033 |   using ValTypeD = int32_t;
 2034 |   using ValTypeA = sparse_elem<2, int8_t>;
 2035 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2036 |   using ValTypeB = int8_t;
 2037 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2039-2047

```text
 2039 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2040 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2042 |   using Shape_MNK = Shape<_64,_192,_64>;
 2043 |   using ThrID   = Layout<_128>;
 2044 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2045 |   using ELayout = GMMA::ELayout_64x64;
 2046 |   using BLayout = GMMA::ABLayout<192, 64>;
 2047 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2049-2050

```text
 2049 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2050 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2052-2061

```text
 2052 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2054 | template <GMMA::SparseSel spsel>
 2055 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_SS_TN<spsel>>
 2056 | {
 2057 |   using ValTypeD = int32_t;
 2058 |   using ValTypeA = sparse_elem<2, int8_t>;
 2059 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2060 |   using ValTypeB = int8_t;
 2061 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2063-2071

```text
 2063 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2064 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2066 |   using Shape_MNK = Shape<_64,_256,_64>;
 2067 |   using ThrID   = Layout<_128>;
 2068 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2069 |   using ELayout = GMMA::ELayout_64x64;
 2070 |   using BLayout = GMMA::ABLayout<256, 64>;
 2071 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2073-2074

```text
 2073 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2074 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2076-2085

```text
 2076 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2078 | template <GMMA::SparseSel spsel>
 2079 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_SS_TN_SATURATE<spsel>>
 2080 | {
 2081 |   using ValTypeD = int32_t;
 2082 |   using ValTypeA = sparse_elem<2, int8_t>;
 2083 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2084 |   using ValTypeB = int8_t;
 2085 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2087-2095

```text
 2087 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2088 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2090 |   using Shape_MNK = Shape<_64,_256,_64>;
 2091 |   using ThrID   = Layout<_128>;
 2092 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2093 |   using ELayout = GMMA::ELayout_64x64;
 2094 |   using BLayout = GMMA::ABLayout<256, 64>;
 2095 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2097-2098

```text
 2097 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2098 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2100-2109

```text
 2100 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2102 | template <GMMA::SparseSel spsel>
 2103 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_RS_TN<spsel>>
 2104 | {
 2105 |   using ValTypeD = int32_t;
 2106 |   using ValTypeA = sparse_elem<2, int8_t>;
 2107 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2108 |   using ValTypeB = int8_t;
 2109 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2111-2118

```text
 2111 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2113 |   using Shape_MNK = Shape<_64,_8,_64>;
 2114 |   using ThrID   = Layout<_128>;
 2115 |   using ALayout = GMMA::ALayout_64x64;
 2116 |   using ELayout = GMMA::ELayout_64x64;
 2117 |   using BLayout = GMMA::ABLayout<  8, 64>;
 2118 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2120-2121

```text
 2120 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2121 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2123-2132

```text
 2123 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2125 | template <GMMA::SparseSel spsel>
 2126 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2127 | {
 2128 |   using ValTypeD = int32_t;
 2129 |   using ValTypeA = sparse_elem<2, int8_t>;
 2130 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2131 |   using ValTypeB = int8_t;
 2132 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2134-2141

```text
 2134 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2136 |   using Shape_MNK = Shape<_64,_8,_64>;
 2137 |   using ThrID   = Layout<_128>;
 2138 |   using ALayout = GMMA::ALayout_64x64;
 2139 |   using ELayout = GMMA::ELayout_64x64;
 2140 |   using BLayout = GMMA::ABLayout<  8, 64>;
 2141 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2143-2144

```text
 2143 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2144 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2146-2155

```text
 2146 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2148 | template <GMMA::SparseSel spsel>
 2149 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_RS_TN<spsel>>
 2150 | {
 2151 |   using ValTypeD = int32_t;
 2152 |   using ValTypeA = sparse_elem<2, int8_t>;
 2153 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2154 |   using ValTypeB = int8_t;
 2155 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2157-2164

```text
 2157 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2159 |   using Shape_MNK = Shape<_64,_16,_64>;
 2160 |   using ThrID   = Layout<_128>;
 2161 |   using ALayout = GMMA::ALayout_64x64;
 2162 |   using ELayout = GMMA::ELayout_64x64;
 2163 |   using BLayout = GMMA::ABLayout< 16, 64>;
 2164 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2166-2167

```text
 2166 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2167 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2169-2178

```text
 2169 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2171 | template <GMMA::SparseSel spsel>
 2172 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2173 | {
 2174 |   using ValTypeD = int32_t;
 2175 |   using ValTypeA = sparse_elem<2, int8_t>;
 2176 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2177 |   using ValTypeB = int8_t;
 2178 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2180-2187

```text
 2180 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2182 |   using Shape_MNK = Shape<_64,_16,_64>;
 2183 |   using ThrID   = Layout<_128>;
 2184 |   using ALayout = GMMA::ALayout_64x64;
 2185 |   using ELayout = GMMA::ELayout_64x64;
 2186 |   using BLayout = GMMA::ABLayout< 16, 64>;
 2187 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2189-2190

```text
 2189 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2190 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2192-2201

```text
 2192 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2194 | template <GMMA::SparseSel spsel>
 2195 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_RS_TN<spsel>>
 2196 | {
 2197 |   using ValTypeD = int32_t;
 2198 |   using ValTypeA = sparse_elem<2, int8_t>;
 2199 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2200 |   using ValTypeB = int8_t;
 2201 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2203-2210

```text
 2203 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2205 |   using Shape_MNK = Shape<_64,_32,_64>;
 2206 |   using ThrID   = Layout<_128>;
 2207 |   using ALayout = GMMA::ALayout_64x64;
 2208 |   using ELayout = GMMA::ELayout_64x64;
 2209 |   using BLayout = GMMA::ABLayout< 32, 64>;
 2210 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2212-2213

```text
 2212 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2213 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2215-2224

```text
 2215 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2217 | template <GMMA::SparseSel spsel>
 2218 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2219 | {
 2220 |   using ValTypeD = int32_t;
 2221 |   using ValTypeA = sparse_elem<2, int8_t>;
 2222 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2223 |   using ValTypeB = int8_t;
 2224 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2226-2233

```text
 2226 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2228 |   using Shape_MNK = Shape<_64,_32,_64>;
 2229 |   using ThrID   = Layout<_128>;
 2230 |   using ALayout = GMMA::ALayout_64x64;
 2231 |   using ELayout = GMMA::ELayout_64x64;
 2232 |   using BLayout = GMMA::ABLayout< 32, 64>;
 2233 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2235-2236

```text
 2235 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2236 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2238-2247

```text
 2238 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2240 | template <GMMA::SparseSel spsel>
 2241 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_RS_TN<spsel>>
 2242 | {
 2243 |   using ValTypeD = int32_t;
 2244 |   using ValTypeA = sparse_elem<2, int8_t>;
 2245 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2246 |   using ValTypeB = int8_t;
 2247 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2249-2256

```text
 2249 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2251 |   using Shape_MNK = Shape<_64,_64,_64>;
 2252 |   using ThrID   = Layout<_128>;
 2253 |   using ALayout = GMMA::ALayout_64x64;
 2254 |   using ELayout = GMMA::ELayout_64x64;
 2255 |   using BLayout = GMMA::ABLayout< 64, 64>;
 2256 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2258-2259

```text
 2258 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2259 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2261-2270

```text
 2261 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2263 | template <GMMA::SparseSel spsel>
 2264 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2265 | {
 2266 |   using ValTypeD = int32_t;
 2267 |   using ValTypeA = sparse_elem<2, int8_t>;
 2268 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2269 |   using ValTypeB = int8_t;
 2270 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2272-2279

```text
 2272 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2274 |   using Shape_MNK = Shape<_64,_64,_64>;
 2275 |   using ThrID   = Layout<_128>;
 2276 |   using ALayout = GMMA::ALayout_64x64;
 2277 |   using ELayout = GMMA::ELayout_64x64;
 2278 |   using BLayout = GMMA::ABLayout< 64, 64>;
 2279 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2281-2282

```text
 2281 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2282 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2284-2293

```text
 2284 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2286 | template <GMMA::SparseSel spsel>
 2287 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_RS_TN<spsel>>
 2288 | {
 2289 |   using ValTypeD = int32_t;
 2290 |   using ValTypeA = sparse_elem<2, int8_t>;
 2291 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2292 |   using ValTypeB = int8_t;
 2293 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2295-2302

```text
 2295 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2297 |   using Shape_MNK = Shape<_64,_96,_64>;
 2298 |   using ThrID   = Layout<_128>;
 2299 |   using ALayout = GMMA::ALayout_64x64;
 2300 |   using ELayout = GMMA::ELayout_64x64;
 2301 |   using BLayout = GMMA::ABLayout< 96, 64>;
 2302 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2304-2305

```text
 2304 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2305 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2307-2316

```text
 2307 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2309 | template <GMMA::SparseSel spsel>
 2310 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2311 | {
 2312 |   using ValTypeD = int32_t;
 2313 |   using ValTypeA = sparse_elem<2, int8_t>;
 2314 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2315 |   using ValTypeB = int8_t;
 2316 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2318-2325

```text
 2318 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2320 |   using Shape_MNK = Shape<_64,_96,_64>;
 2321 |   using ThrID   = Layout<_128>;
 2322 |   using ALayout = GMMA::ALayout_64x64;
 2323 |   using ELayout = GMMA::ELayout_64x64;
 2324 |   using BLayout = GMMA::ABLayout< 96, 64>;
 2325 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2327-2328

```text
 2327 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2328 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2330-2339

```text
 2330 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2332 | template <GMMA::SparseSel spsel>
 2333 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_RS_TN<spsel>>
 2334 | {
 2335 |   using ValTypeD = int32_t;
 2336 |   using ValTypeA = sparse_elem<2, int8_t>;
 2337 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2338 |   using ValTypeB = int8_t;
 2339 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2341-2348

```text
 2341 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2343 |   using Shape_MNK = Shape<_64,_128,_64>;
 2344 |   using ThrID   = Layout<_128>;
 2345 |   using ALayout = GMMA::ALayout_64x64;
 2346 |   using ELayout = GMMA::ELayout_64x64;
 2347 |   using BLayout = GMMA::ABLayout<128, 64>;
 2348 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2350-2351

```text
 2350 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2351 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2353-2362

```text
 2353 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2355 | template <GMMA::SparseSel spsel>
 2356 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2357 | {
 2358 |   using ValTypeD = int32_t;
 2359 |   using ValTypeA = sparse_elem<2, int8_t>;
 2360 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2361 |   using ValTypeB = int8_t;
 2362 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2364-2371

```text
 2364 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2366 |   using Shape_MNK = Shape<_64,_128,_64>;
 2367 |   using ThrID   = Layout<_128>;
 2368 |   using ALayout = GMMA::ALayout_64x64;
 2369 |   using ELayout = GMMA::ELayout_64x64;
 2370 |   using BLayout = GMMA::ABLayout<128, 64>;
 2371 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2373-2374

```text
 2373 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2374 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2376-2385

```text
 2376 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2378 | template <GMMA::SparseSel spsel>
 2379 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_RS_TN<spsel>>
 2380 | {
 2381 |   using ValTypeD = int32_t;
 2382 |   using ValTypeA = sparse_elem<2, int8_t>;
 2383 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2384 |   using ValTypeB = int8_t;
 2385 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2387-2394

```text
 2387 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2389 |   using Shape_MNK = Shape<_64,_192,_64>;
 2390 |   using ThrID   = Layout<_128>;
 2391 |   using ALayout = GMMA::ALayout_64x64;
 2392 |   using ELayout = GMMA::ELayout_64x64;
 2393 |   using BLayout = GMMA::ABLayout<192, 64>;
 2394 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2396-2397

```text
 2396 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2397 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2399-2408

```text
 2399 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2401 | template <GMMA::SparseSel spsel>
 2402 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2403 | {
 2404 |   using ValTypeD = int32_t;
 2405 |   using ValTypeA = sparse_elem<2, int8_t>;
 2406 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2407 |   using ValTypeB = int8_t;
 2408 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2410-2417

```text
 2410 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2412 |   using Shape_MNK = Shape<_64,_192,_64>;
 2413 |   using ThrID   = Layout<_128>;
 2414 |   using ALayout = GMMA::ALayout_64x64;
 2415 |   using ELayout = GMMA::ELayout_64x64;
 2416 |   using BLayout = GMMA::ABLayout<192, 64>;
 2417 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2419-2420

```text
 2419 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2420 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2422-2431

```text
 2422 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2424 | template <GMMA::SparseSel spsel>
 2425 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_RS_TN<spsel>>
 2426 | {
 2427 |   using ValTypeD = int32_t;
 2428 |   using ValTypeA = sparse_elem<2, int8_t>;
 2429 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2430 |   using ValTypeB = int8_t;
 2431 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2433-2440

```text
 2433 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2435 |   using Shape_MNK = Shape<_64,_256,_64>;
 2436 |   using ThrID   = Layout<_128>;
 2437 |   using ALayout = GMMA::ALayout_64x64;
 2438 |   using ELayout = GMMA::ELayout_64x64;
 2439 |   using BLayout = GMMA::ABLayout<256, 64>;
 2440 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2442-2443

```text
 2442 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2443 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2445-2454

```text
 2445 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2447 | template <GMMA::SparseSel spsel>
 2448 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_RS_TN_SATURATE<spsel>>
 2449 | {
 2450 |   using ValTypeD = int32_t;
 2451 |   using ValTypeA = sparse_elem<2, int8_t>;
 2452 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2453 |   using ValTypeB = int8_t;
 2454 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2456-2463

```text
 2456 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2458 |   using Shape_MNK = Shape<_64,_256,_64>;
 2459 |   using ThrID   = Layout<_128>;
 2460 |   using ALayout = GMMA::ALayout_64x64;
 2461 |   using ELayout = GMMA::ELayout_64x64;
 2462 |   using BLayout = GMMA::ABLayout<256, 64>;
 2463 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2465-2466

```text
 2465 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2466 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2468-2477

```text
 2468 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2470 | template <GMMA::SparseSel spsel>
 2471 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_SS_TN<spsel>>
 2472 | {
 2473 |   using ValTypeD = int32_t;
 2474 |   using ValTypeA = sparse_elem<2, int8_t>;
 2475 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2476 |   using ValTypeB = uint8_t;
 2477 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2479-2487

```text
 2479 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2480 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2482 |   using Shape_MNK = Shape<_64,_8,_64>;
 2483 |   using ThrID   = Layout<_128>;
 2484 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2485 |   using ELayout = GMMA::ELayout_64x64;
 2486 |   using BLayout = GMMA::ABLayout<  8, 64>;
 2487 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2489-2490

```text
 2489 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2490 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2492-2501

```text
 2492 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2494 | template <GMMA::SparseSel spsel>
 2495 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2496 | {
 2497 |   using ValTypeD = int32_t;
 2498 |   using ValTypeA = sparse_elem<2, int8_t>;
 2499 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2500 |   using ValTypeB = uint8_t;
 2501 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2503-2511

```text
 2503 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2504 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2506 |   using Shape_MNK = Shape<_64,_8,_64>;
 2507 |   using ThrID   = Layout<_128>;
 2508 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2509 |   using ELayout = GMMA::ELayout_64x64;
 2510 |   using BLayout = GMMA::ABLayout<  8, 64>;
 2511 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2513-2514

```text
 2513 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2514 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2516-2525

```text
 2516 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2518 | template <GMMA::SparseSel spsel>
 2519 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_SS_TN<spsel>>
 2520 | {
 2521 |   using ValTypeD = int32_t;
 2522 |   using ValTypeA = sparse_elem<2, int8_t>;
 2523 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2524 |   using ValTypeB = uint8_t;
 2525 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2527-2535

```text
 2527 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2528 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2530 |   using Shape_MNK = Shape<_64,_16,_64>;
 2531 |   using ThrID   = Layout<_128>;
 2532 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2533 |   using ELayout = GMMA::ELayout_64x64;
 2534 |   using BLayout = GMMA::ABLayout< 16, 64>;
 2535 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2537-2538

```text
 2537 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2538 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2540-2549

```text
 2540 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2542 | template <GMMA::SparseSel spsel>
 2543 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2544 | {
 2545 |   using ValTypeD = int32_t;
 2546 |   using ValTypeA = sparse_elem<2, int8_t>;
 2547 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2548 |   using ValTypeB = uint8_t;
 2549 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2551-2559

```text
 2551 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2552 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2554 |   using Shape_MNK = Shape<_64,_16,_64>;
 2555 |   using ThrID   = Layout<_128>;
 2556 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2557 |   using ELayout = GMMA::ELayout_64x64;
 2558 |   using BLayout = GMMA::ABLayout< 16, 64>;
 2559 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2561-2562

```text
 2561 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2562 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2564-2573

```text
 2564 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2566 | template <GMMA::SparseSel spsel>
 2567 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_SS_TN<spsel>>
 2568 | {
 2569 |   using ValTypeD = int32_t;
 2570 |   using ValTypeA = sparse_elem<2, int8_t>;
 2571 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2572 |   using ValTypeB = uint8_t;
 2573 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2575-2583

```text
 2575 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2576 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2578 |   using Shape_MNK = Shape<_64,_32,_64>;
 2579 |   using ThrID   = Layout<_128>;
 2580 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2581 |   using ELayout = GMMA::ELayout_64x64;
 2582 |   using BLayout = GMMA::ABLayout< 32, 64>;
 2583 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2585-2586

```text
 2585 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2586 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2588-2597

```text
 2588 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2590 | template <GMMA::SparseSel spsel>
 2591 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2592 | {
 2593 |   using ValTypeD = int32_t;
 2594 |   using ValTypeA = sparse_elem<2, int8_t>;
 2595 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2596 |   using ValTypeB = uint8_t;
 2597 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2599-2607

```text
 2599 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2600 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2602 |   using Shape_MNK = Shape<_64,_32,_64>;
 2603 |   using ThrID   = Layout<_128>;
 2604 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2605 |   using ELayout = GMMA::ELayout_64x64;
 2606 |   using BLayout = GMMA::ABLayout< 32, 64>;
 2607 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2609-2610

```text
 2609 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2610 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2612-2621

```text
 2612 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2614 | template <GMMA::SparseSel spsel>
 2615 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_SS_TN<spsel>>
 2616 | {
 2617 |   using ValTypeD = int32_t;
 2618 |   using ValTypeA = sparse_elem<2, int8_t>;
 2619 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2620 |   using ValTypeB = uint8_t;
 2621 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2623-2631

```text
 2623 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2624 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2626 |   using Shape_MNK = Shape<_64,_64,_64>;
 2627 |   using ThrID   = Layout<_128>;
 2628 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2629 |   using ELayout = GMMA::ELayout_64x64;
 2630 |   using BLayout = GMMA::ABLayout< 64, 64>;
 2631 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2633-2634

```text
 2633 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2634 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2636-2645

```text
 2636 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2638 | template <GMMA::SparseSel spsel>
 2639 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2640 | {
 2641 |   using ValTypeD = int32_t;
 2642 |   using ValTypeA = sparse_elem<2, int8_t>;
 2643 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2644 |   using ValTypeB = uint8_t;
 2645 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2647-2655

```text
 2647 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2648 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2650 |   using Shape_MNK = Shape<_64,_64,_64>;
 2651 |   using ThrID   = Layout<_128>;
 2652 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2653 |   using ELayout = GMMA::ELayout_64x64;
 2654 |   using BLayout = GMMA::ABLayout< 64, 64>;
 2655 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2657-2658

```text
 2657 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2658 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2660-2669

```text
 2660 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2662 | template <GMMA::SparseSel spsel>
 2663 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_SS_TN<spsel>>
 2664 | {
 2665 |   using ValTypeD = int32_t;
 2666 |   using ValTypeA = sparse_elem<2, int8_t>;
 2667 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2668 |   using ValTypeB = uint8_t;
 2669 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2671-2679

```text
 2671 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2672 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2674 |   using Shape_MNK = Shape<_64,_96,_64>;
 2675 |   using ThrID   = Layout<_128>;
 2676 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2677 |   using ELayout = GMMA::ELayout_64x64;
 2678 |   using BLayout = GMMA::ABLayout< 96, 64>;
 2679 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2681-2682

```text
 2681 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2682 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2684-2693

```text
 2684 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2686 | template <GMMA::SparseSel spsel>
 2687 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2688 | {
 2689 |   using ValTypeD = int32_t;
 2690 |   using ValTypeA = sparse_elem<2, int8_t>;
 2691 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2692 |   using ValTypeB = uint8_t;
 2693 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2695-2703

```text
 2695 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2696 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2698 |   using Shape_MNK = Shape<_64,_96,_64>;
 2699 |   using ThrID   = Layout<_128>;
 2700 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2701 |   using ELayout = GMMA::ELayout_64x64;
 2702 |   using BLayout = GMMA::ABLayout< 96, 64>;
 2703 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2705-2706

```text
 2705 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2706 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2708-2717

```text
 2708 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2710 | template <GMMA::SparseSel spsel>
 2711 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_SS_TN<spsel>>
 2712 | {
 2713 |   using ValTypeD = int32_t;
 2714 |   using ValTypeA = sparse_elem<2, int8_t>;
 2715 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2716 |   using ValTypeB = uint8_t;
 2717 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2719-2727

```text
 2719 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2720 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2722 |   using Shape_MNK = Shape<_64,_128,_64>;
 2723 |   using ThrID   = Layout<_128>;
 2724 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2725 |   using ELayout = GMMA::ELayout_64x64;
 2726 |   using BLayout = GMMA::ABLayout<128, 64>;
 2727 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2729-2730

```text
 2729 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2730 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2732-2741

```text
 2732 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2734 | template <GMMA::SparseSel spsel>
 2735 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2736 | {
 2737 |   using ValTypeD = int32_t;
 2738 |   using ValTypeA = sparse_elem<2, int8_t>;
 2739 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2740 |   using ValTypeB = uint8_t;
 2741 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2743-2751

```text
 2743 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2744 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2746 |   using Shape_MNK = Shape<_64,_128,_64>;
 2747 |   using ThrID   = Layout<_128>;
 2748 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2749 |   using ELayout = GMMA::ELayout_64x64;
 2750 |   using BLayout = GMMA::ABLayout<128, 64>;
 2751 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2753-2754

```text
 2753 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2754 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2756-2765

```text
 2756 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2758 | template <GMMA::SparseSel spsel>
 2759 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_SS_TN<spsel>>
 2760 | {
 2761 |   using ValTypeD = int32_t;
 2762 |   using ValTypeA = sparse_elem<2, int8_t>;
 2763 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2764 |   using ValTypeB = uint8_t;
 2765 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2767-2775

```text
 2767 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2768 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2770 |   using Shape_MNK = Shape<_64,_192,_64>;
 2771 |   using ThrID   = Layout<_128>;
 2772 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2773 |   using ELayout = GMMA::ELayout_64x64;
 2774 |   using BLayout = GMMA::ABLayout<192, 64>;
 2775 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2777-2778

```text
 2777 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2778 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2780-2789

```text
 2780 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2782 | template <GMMA::SparseSel spsel>
 2783 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2784 | {
 2785 |   using ValTypeD = int32_t;
 2786 |   using ValTypeA = sparse_elem<2, int8_t>;
 2787 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2788 |   using ValTypeB = uint8_t;
 2789 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2791-2799

```text
 2791 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2792 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2794 |   using Shape_MNK = Shape<_64,_192,_64>;
 2795 |   using ThrID   = Layout<_128>;
 2796 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2797 |   using ELayout = GMMA::ELayout_64x64;
 2798 |   using BLayout = GMMA::ABLayout<192, 64>;
 2799 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2801-2802

```text
 2801 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2802 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2804-2813

```text
 2804 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2806 | template <GMMA::SparseSel spsel>
 2807 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_SS_TN<spsel>>
 2808 | {
 2809 |   using ValTypeD = int32_t;
 2810 |   using ValTypeA = sparse_elem<2, int8_t>;
 2811 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2812 |   using ValTypeB = uint8_t;
 2813 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2815-2823

```text
 2815 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2816 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2818 |   using Shape_MNK = Shape<_64,_256,_64>;
 2819 |   using ThrID   = Layout<_128>;
 2820 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2821 |   using ELayout = GMMA::ELayout_64x64;
 2822 |   using BLayout = GMMA::ABLayout<256, 64>;
 2823 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2825-2826

```text
 2825 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2826 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2828-2837

```text
 2828 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2830 | template <GMMA::SparseSel spsel>
 2831 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_SS_TN_SATURATE<spsel>>
 2832 | {
 2833 |   using ValTypeD = int32_t;
 2834 |   using ValTypeA = sparse_elem<2, int8_t>;
 2835 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2836 |   using ValTypeB = uint8_t;
 2837 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2839-2847

```text
 2839 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2840 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2842 |   using Shape_MNK = Shape<_64,_256,_64>;
 2843 |   using ThrID   = Layout<_128>;
 2844 |   using ALayout = GMMA::ABLayout< 64, 64>;
 2845 |   using ELayout = GMMA::ELayout_64x64;
 2846 |   using BLayout = GMMA::ABLayout<256, 64>;
 2847 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2849-2850

```text
 2849 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2850 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2852-2861

```text
 2852 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2854 | template <GMMA::SparseSel spsel>
 2855 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_RS_TN<spsel>>
 2856 | {
 2857 |   using ValTypeD = int32_t;
 2858 |   using ValTypeA = sparse_elem<2, int8_t>;
 2859 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2860 |   using ValTypeB = uint8_t;
 2861 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2863-2870

```text
 2863 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2865 |   using Shape_MNK = Shape<_64,_8,_64>;
 2866 |   using ThrID   = Layout<_128>;
 2867 |   using ALayout = GMMA::ALayout_64x64;
 2868 |   using ELayout = GMMA::ELayout_64x64;
 2869 |   using BLayout = GMMA::ABLayout<  8, 64>;
 2870 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2872-2873

```text
 2872 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2873 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2875-2884

```text
 2875 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2877 | template <GMMA::SparseSel spsel>
 2878 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_RS_TN_SATURATE<spsel>>
 2879 | {
 2880 |   using ValTypeD = int32_t;
 2881 |   using ValTypeA = sparse_elem<2, int8_t>;
 2882 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2883 |   using ValTypeB = uint8_t;
 2884 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2886-2893

```text
 2886 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2888 |   using Shape_MNK = Shape<_64,_8,_64>;
 2889 |   using ThrID   = Layout<_128>;
 2890 |   using ALayout = GMMA::ALayout_64x64;
 2891 |   using ELayout = GMMA::ELayout_64x64;
 2892 |   using BLayout = GMMA::ABLayout<  8, 64>;
 2893 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2895-2896

```text
 2895 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2896 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2898-2907

```text
 2898 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2900 | template <GMMA::SparseSel spsel>
 2901 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_RS_TN<spsel>>
 2902 | {
 2903 |   using ValTypeD = int32_t;
 2904 |   using ValTypeA = sparse_elem<2, int8_t>;
 2905 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2906 |   using ValTypeB = uint8_t;
 2907 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2909-2916

```text
 2909 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2911 |   using Shape_MNK = Shape<_64,_16,_64>;
 2912 |   using ThrID   = Layout<_128>;
 2913 |   using ALayout = GMMA::ALayout_64x64;
 2914 |   using ELayout = GMMA::ELayout_64x64;
 2915 |   using BLayout = GMMA::ABLayout< 16, 64>;
 2916 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2918-2919

```text
 2918 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2919 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2921-2930

```text
 2921 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2923 | template <GMMA::SparseSel spsel>
 2924 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_RS_TN_SATURATE<spsel>>
 2925 | {
 2926 |   using ValTypeD = int32_t;
 2927 |   using ValTypeA = sparse_elem<2, int8_t>;
 2928 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2929 |   using ValTypeB = uint8_t;
 2930 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2932-2939

```text
 2932 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2934 |   using Shape_MNK = Shape<_64,_16,_64>;
 2935 |   using ThrID   = Layout<_128>;
 2936 |   using ALayout = GMMA::ALayout_64x64;
 2937 |   using ELayout = GMMA::ELayout_64x64;
 2938 |   using BLayout = GMMA::ABLayout< 16, 64>;
 2939 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2941-2942

```text
 2941 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2942 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2944-2953

```text
 2944 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2946 | template <GMMA::SparseSel spsel>
 2947 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_RS_TN<spsel>>
 2948 | {
 2949 |   using ValTypeD = int32_t;
 2950 |   using ValTypeA = sparse_elem<2, int8_t>;
 2951 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2952 |   using ValTypeB = uint8_t;
 2953 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2955-2962

```text
 2955 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2957 |   using Shape_MNK = Shape<_64,_32,_64>;
 2958 |   using ThrID   = Layout<_128>;
 2959 |   using ALayout = GMMA::ALayout_64x64;
 2960 |   using ELayout = GMMA::ELayout_64x64;
 2961 |   using BLayout = GMMA::ABLayout< 32, 64>;
 2962 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2964-2965

```text
 2964 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2965 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2967-2976

```text
 2967 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2969 | template <GMMA::SparseSel spsel>
 2970 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_RS_TN_SATURATE<spsel>>
 2971 | {
 2972 |   using ValTypeD = int32_t;
 2973 |   using ValTypeA = sparse_elem<2, int8_t>;
 2974 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2975 |   using ValTypeB = uint8_t;
 2976 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2978-2985

```text
 2978 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2980 |   using Shape_MNK = Shape<_64,_32,_64>;
 2981 |   using ThrID   = Layout<_128>;
 2982 |   using ALayout = GMMA::ALayout_64x64;
 2983 |   using ELayout = GMMA::ELayout_64x64;
 2984 |   using BLayout = GMMA::ABLayout< 32, 64>;
 2985 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2987-2988

```text
 2987 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2988 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2990-2999

```text
 2990 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2992 | template <GMMA::SparseSel spsel>
 2993 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_RS_TN<spsel>>
 2994 | {
 2995 |   using ValTypeD = int32_t;
 2996 |   using ValTypeA = sparse_elem<2, int8_t>;
 2997 |   using ValTypeE = sparse_elem<8, uint8_t>;
 2998 |   using ValTypeB = uint8_t;
 2999 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3001-3008

```text
 3001 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3003 |   using Shape_MNK = Shape<_64,_64,_64>;
 3004 |   using ThrID   = Layout<_128>;
 3005 |   using ALayout = GMMA::ALayout_64x64;
 3006 |   using ELayout = GMMA::ELayout_64x64;
 3007 |   using BLayout = GMMA::ABLayout< 64, 64>;
 3008 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3010-3011

```text
 3010 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3011 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3013-3022

```text
 3013 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3015 | template <GMMA::SparseSel spsel>
 3016 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_RS_TN_SATURATE<spsel>>
 3017 | {
 3018 |   using ValTypeD = int32_t;
 3019 |   using ValTypeA = sparse_elem<2, int8_t>;
 3020 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3021 |   using ValTypeB = uint8_t;
 3022 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3024-3031

```text
 3024 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3026 |   using Shape_MNK = Shape<_64,_64,_64>;
 3027 |   using ThrID   = Layout<_128>;
 3028 |   using ALayout = GMMA::ALayout_64x64;
 3029 |   using ELayout = GMMA::ELayout_64x64;
 3030 |   using BLayout = GMMA::ABLayout< 64, 64>;
 3031 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3033-3034

```text
 3033 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3034 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3036-3045

```text
 3036 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3038 | template <GMMA::SparseSel spsel>
 3039 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_RS_TN<spsel>>
 3040 | {
 3041 |   using ValTypeD = int32_t;
 3042 |   using ValTypeA = sparse_elem<2, int8_t>;
 3043 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3044 |   using ValTypeB = uint8_t;
 3045 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3047-3054

```text
 3047 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3049 |   using Shape_MNK = Shape<_64,_96,_64>;
 3050 |   using ThrID   = Layout<_128>;
 3051 |   using ALayout = GMMA::ALayout_64x64;
 3052 |   using ELayout = GMMA::ELayout_64x64;
 3053 |   using BLayout = GMMA::ABLayout< 96, 64>;
 3054 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3056-3057

```text
 3056 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3057 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3059-3068

```text
 3059 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3061 | template <GMMA::SparseSel spsel>
 3062 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_RS_TN_SATURATE<spsel>>
 3063 | {
 3064 |   using ValTypeD = int32_t;
 3065 |   using ValTypeA = sparse_elem<2, int8_t>;
 3066 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3067 |   using ValTypeB = uint8_t;
 3068 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3070-3077

```text
 3070 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3072 |   using Shape_MNK = Shape<_64,_96,_64>;
 3073 |   using ThrID   = Layout<_128>;
 3074 |   using ALayout = GMMA::ALayout_64x64;
 3075 |   using ELayout = GMMA::ELayout_64x64;
 3076 |   using BLayout = GMMA::ABLayout< 96, 64>;
 3077 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3079-3080

```text
 3079 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3080 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3082-3091

```text
 3082 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3084 | template <GMMA::SparseSel spsel>
 3085 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_RS_TN<spsel>>
 3086 | {
 3087 |   using ValTypeD = int32_t;
 3088 |   using ValTypeA = sparse_elem<2, int8_t>;
 3089 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3090 |   using ValTypeB = uint8_t;
 3091 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3093-3100

```text
 3093 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3095 |   using Shape_MNK = Shape<_64,_128,_64>;
 3096 |   using ThrID   = Layout<_128>;
 3097 |   using ALayout = GMMA::ALayout_64x64;
 3098 |   using ELayout = GMMA::ELayout_64x64;
 3099 |   using BLayout = GMMA::ABLayout<128, 64>;
 3100 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3102-3103

```text
 3102 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3103 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3105-3114

```text
 3105 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3107 | template <GMMA::SparseSel spsel>
 3108 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_RS_TN_SATURATE<spsel>>
 3109 | {
 3110 |   using ValTypeD = int32_t;
 3111 |   using ValTypeA = sparse_elem<2, int8_t>;
 3112 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3113 |   using ValTypeB = uint8_t;
 3114 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3116-3123

```text
 3116 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3118 |   using Shape_MNK = Shape<_64,_128,_64>;
 3119 |   using ThrID   = Layout<_128>;
 3120 |   using ALayout = GMMA::ALayout_64x64;
 3121 |   using ELayout = GMMA::ELayout_64x64;
 3122 |   using BLayout = GMMA::ABLayout<128, 64>;
 3123 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3125-3126

```text
 3125 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3126 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3128-3137

```text
 3128 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3130 | template <GMMA::SparseSel spsel>
 3131 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_RS_TN<spsel>>
 3132 | {
 3133 |   using ValTypeD = int32_t;
 3134 |   using ValTypeA = sparse_elem<2, int8_t>;
 3135 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3136 |   using ValTypeB = uint8_t;
 3137 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3139-3146

```text
 3139 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3141 |   using Shape_MNK = Shape<_64,_192,_64>;
 3142 |   using ThrID   = Layout<_128>;
 3143 |   using ALayout = GMMA::ALayout_64x64;
 3144 |   using ELayout = GMMA::ELayout_64x64;
 3145 |   using BLayout = GMMA::ABLayout<192, 64>;
 3146 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3148-3149

```text
 3148 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3149 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3151-3160

```text
 3151 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3153 | template <GMMA::SparseSel spsel>
 3154 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_RS_TN_SATURATE<spsel>>
 3155 | {
 3156 |   using ValTypeD = int32_t;
 3157 |   using ValTypeA = sparse_elem<2, int8_t>;
 3158 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3159 |   using ValTypeB = uint8_t;
 3160 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3162-3169

```text
 3162 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3164 |   using Shape_MNK = Shape<_64,_192,_64>;
 3165 |   using ThrID   = Layout<_128>;
 3166 |   using ALayout = GMMA::ALayout_64x64;
 3167 |   using ELayout = GMMA::ELayout_64x64;
 3168 |   using BLayout = GMMA::ABLayout<192, 64>;
 3169 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3171-3172

```text
 3171 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3172 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3174-3183

```text
 3174 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3176 | template <GMMA::SparseSel spsel>
 3177 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_RS_TN<spsel>>
 3178 | {
 3179 |   using ValTypeD = int32_t;
 3180 |   using ValTypeA = sparse_elem<2, int8_t>;
 3181 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3182 |   using ValTypeB = uint8_t;
 3183 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3185-3192

```text
 3185 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3187 |   using Shape_MNK = Shape<_64,_256,_64>;
 3188 |   using ThrID   = Layout<_128>;
 3189 |   using ALayout = GMMA::ALayout_64x64;
 3190 |   using ELayout = GMMA::ELayout_64x64;
 3191 |   using BLayout = GMMA::ABLayout<256, 64>;
 3192 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3194-3195

```text
 3194 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3195 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3197-3206

```text
 3197 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3199 | template <GMMA::SparseSel spsel>
 3200 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_RS_TN_SATURATE<spsel>>
 3201 | {
 3202 |   using ValTypeD = int32_t;
 3203 |   using ValTypeA = sparse_elem<2, int8_t>;
 3204 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3205 |   using ValTypeB = uint8_t;
 3206 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32S8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3208-3215

```text
 3208 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3210 |   using Shape_MNK = Shape<_64,_256,_64>;
 3211 |   using ThrID   = Layout<_128>;
 3212 |   using ALayout = GMMA::ALayout_64x64;
 3213 |   using ELayout = GMMA::ELayout_64x64;
 3214 |   using BLayout = GMMA::ABLayout<256, 64>;
 3215 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3217-3218

```text
 3217 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3218 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3220-3229

```text
 3220 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3222 | template <GMMA::SparseSel spsel>
 3223 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_SS_TN<spsel>>
 3224 | {
 3225 |   using ValTypeD = int32_t;
 3226 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3227 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3228 |   using ValTypeB = int8_t;
 3229 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3231-3239

```text
 3231 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3232 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3234 |   using Shape_MNK = Shape<_64,_8,_64>;
 3235 |   using ThrID   = Layout<_128>;
 3236 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3237 |   using ELayout = GMMA::ELayout_64x64;
 3238 |   using BLayout = GMMA::ABLayout<  8, 64>;
 3239 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3241-3242

```text
 3241 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3242 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3244-3253

```text
 3244 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3246 | template <GMMA::SparseSel spsel>
 3247 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3248 | {
 3249 |   using ValTypeD = int32_t;
 3250 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3251 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3252 |   using ValTypeB = int8_t;
 3253 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3255-3263

```text
 3255 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3256 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3258 |   using Shape_MNK = Shape<_64,_8,_64>;
 3259 |   using ThrID   = Layout<_128>;
 3260 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3261 |   using ELayout = GMMA::ELayout_64x64;
 3262 |   using BLayout = GMMA::ABLayout<  8, 64>;
 3263 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3265-3266

```text
 3265 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3266 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3268-3277

```text
 3268 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3270 | template <GMMA::SparseSel spsel>
 3271 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_SS_TN<spsel>>
 3272 | {
 3273 |   using ValTypeD = int32_t;
 3274 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3275 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3276 |   using ValTypeB = int8_t;
 3277 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3279-3287

```text
 3279 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3280 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3282 |   using Shape_MNK = Shape<_64,_16,_64>;
 3283 |   using ThrID   = Layout<_128>;
 3284 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3285 |   using ELayout = GMMA::ELayout_64x64;
 3286 |   using BLayout = GMMA::ABLayout< 16, 64>;
 3287 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3289-3290

```text
 3289 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3290 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3292-3301

```text
 3292 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3294 | template <GMMA::SparseSel spsel>
 3295 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3296 | {
 3297 |   using ValTypeD = int32_t;
 3298 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3299 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3300 |   using ValTypeB = int8_t;
 3301 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3303-3311

```text
 3303 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3304 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3306 |   using Shape_MNK = Shape<_64,_16,_64>;
 3307 |   using ThrID   = Layout<_128>;
 3308 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3309 |   using ELayout = GMMA::ELayout_64x64;
 3310 |   using BLayout = GMMA::ABLayout< 16, 64>;
 3311 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3313-3314

```text
 3313 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3314 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3316-3325

```text
 3316 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3318 | template <GMMA::SparseSel spsel>
 3319 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_SS_TN<spsel>>
 3320 | {
 3321 |   using ValTypeD = int32_t;
 3322 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3323 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3324 |   using ValTypeB = int8_t;
 3325 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3327-3335

```text
 3327 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3328 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3330 |   using Shape_MNK = Shape<_64,_32,_64>;
 3331 |   using ThrID   = Layout<_128>;
 3332 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3333 |   using ELayout = GMMA::ELayout_64x64;
 3334 |   using BLayout = GMMA::ABLayout< 32, 64>;
 3335 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3337-3338

```text
 3337 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3338 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3340-3349

```text
 3340 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3342 | template <GMMA::SparseSel spsel>
 3343 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3344 | {
 3345 |   using ValTypeD = int32_t;
 3346 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3347 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3348 |   using ValTypeB = int8_t;
 3349 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3351-3359

```text
 3351 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3352 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3354 |   using Shape_MNK = Shape<_64,_32,_64>;
 3355 |   using ThrID   = Layout<_128>;
 3356 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3357 |   using ELayout = GMMA::ELayout_64x64;
 3358 |   using BLayout = GMMA::ABLayout< 32, 64>;
 3359 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3361-3362

```text
 3361 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3362 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3364-3373

```text
 3364 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3366 | template <GMMA::SparseSel spsel>
 3367 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_SS_TN<spsel>>
 3368 | {
 3369 |   using ValTypeD = int32_t;
 3370 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3371 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3372 |   using ValTypeB = int8_t;
 3373 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3375-3383

```text
 3375 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3376 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3378 |   using Shape_MNK = Shape<_64,_64,_64>;
 3379 |   using ThrID   = Layout<_128>;
 3380 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3381 |   using ELayout = GMMA::ELayout_64x64;
 3382 |   using BLayout = GMMA::ABLayout< 64, 64>;
 3383 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3385-3386

```text
 3385 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3386 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3388-3397

```text
 3388 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3390 | template <GMMA::SparseSel spsel>
 3391 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3392 | {
 3393 |   using ValTypeD = int32_t;
 3394 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3395 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3396 |   using ValTypeB = int8_t;
 3397 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3399-3407

```text
 3399 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3400 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3402 |   using Shape_MNK = Shape<_64,_64,_64>;
 3403 |   using ThrID   = Layout<_128>;
 3404 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3405 |   using ELayout = GMMA::ELayout_64x64;
 3406 |   using BLayout = GMMA::ABLayout< 64, 64>;
 3407 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3409-3410

```text
 3409 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3410 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3412-3421

```text
 3412 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3414 | template <GMMA::SparseSel spsel>
 3415 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_SS_TN<spsel>>
 3416 | {
 3417 |   using ValTypeD = int32_t;
 3418 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3419 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3420 |   using ValTypeB = int8_t;
 3421 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3423-3431

```text
 3423 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3424 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3426 |   using Shape_MNK = Shape<_64,_96,_64>;
 3427 |   using ThrID   = Layout<_128>;
 3428 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3429 |   using ELayout = GMMA::ELayout_64x64;
 3430 |   using BLayout = GMMA::ABLayout< 96, 64>;
 3431 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3433-3434

```text
 3433 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3434 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3436-3445

```text
 3436 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3438 | template <GMMA::SparseSel spsel>
 3439 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3440 | {
 3441 |   using ValTypeD = int32_t;
 3442 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3443 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3444 |   using ValTypeB = int8_t;
 3445 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3447-3455

```text
 3447 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3448 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3450 |   using Shape_MNK = Shape<_64,_96,_64>;
 3451 |   using ThrID   = Layout<_128>;
 3452 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3453 |   using ELayout = GMMA::ELayout_64x64;
 3454 |   using BLayout = GMMA::ABLayout< 96, 64>;
 3455 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3457-3458

```text
 3457 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3458 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3460-3469

```text
 3460 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3462 | template <GMMA::SparseSel spsel>
 3463 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_SS_TN<spsel>>
 3464 | {
 3465 |   using ValTypeD = int32_t;
 3466 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3467 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3468 |   using ValTypeB = int8_t;
 3469 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3471-3479

```text
 3471 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3472 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3474 |   using Shape_MNK = Shape<_64,_128,_64>;
 3475 |   using ThrID   = Layout<_128>;
 3476 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3477 |   using ELayout = GMMA::ELayout_64x64;
 3478 |   using BLayout = GMMA::ABLayout<128, 64>;
 3479 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3481-3482

```text
 3481 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3482 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3484-3493

```text
 3484 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3486 | template <GMMA::SparseSel spsel>
 3487 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3488 | {
 3489 |   using ValTypeD = int32_t;
 3490 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3491 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3492 |   using ValTypeB = int8_t;
 3493 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3495-3503

```text
 3495 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3496 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3498 |   using Shape_MNK = Shape<_64,_128,_64>;
 3499 |   using ThrID   = Layout<_128>;
 3500 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3501 |   using ELayout = GMMA::ELayout_64x64;
 3502 |   using BLayout = GMMA::ABLayout<128, 64>;
 3503 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3505-3506

```text
 3505 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3506 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3508-3517

```text
 3508 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3510 | template <GMMA::SparseSel spsel>
 3511 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_SS_TN<spsel>>
 3512 | {
 3513 |   using ValTypeD = int32_t;
 3514 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3515 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3516 |   using ValTypeB = int8_t;
 3517 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3519-3527

```text
 3519 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3520 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3522 |   using Shape_MNK = Shape<_64,_192,_64>;
 3523 |   using ThrID   = Layout<_128>;
 3524 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3525 |   using ELayout = GMMA::ELayout_64x64;
 3526 |   using BLayout = GMMA::ABLayout<192, 64>;
 3527 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3529-3530

```text
 3529 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3530 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3532-3541

```text
 3532 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3534 | template <GMMA::SparseSel spsel>
 3535 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3536 | {
 3537 |   using ValTypeD = int32_t;
 3538 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3539 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3540 |   using ValTypeB = int8_t;
 3541 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3543-3551

```text
 3543 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3544 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3546 |   using Shape_MNK = Shape<_64,_192,_64>;
 3547 |   using ThrID   = Layout<_128>;
 3548 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3549 |   using ELayout = GMMA::ELayout_64x64;
 3550 |   using BLayout = GMMA::ABLayout<192, 64>;
 3551 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3553-3554

```text
 3553 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3554 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3556-3565

```text
 3556 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3558 | template <GMMA::SparseSel spsel>
 3559 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_SS_TN<spsel>>
 3560 | {
 3561 |   using ValTypeD = int32_t;
 3562 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3563 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3564 |   using ValTypeB = int8_t;
 3565 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3567-3575

```text
 3567 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3568 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3570 |   using Shape_MNK = Shape<_64,_256,_64>;
 3571 |   using ThrID   = Layout<_128>;
 3572 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3573 |   using ELayout = GMMA::ELayout_64x64;
 3574 |   using BLayout = GMMA::ABLayout<256, 64>;
 3575 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3577-3578

```text
 3577 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3578 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3580-3589

```text
 3580 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3582 | template <GMMA::SparseSel spsel>
 3583 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_SS_TN_SATURATE<spsel>>
 3584 | {
 3585 |   using ValTypeD = int32_t;
 3586 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3587 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3588 |   using ValTypeB = int8_t;
 3589 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3591-3599

```text
 3591 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3592 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3594 |   using Shape_MNK = Shape<_64,_256,_64>;
 3595 |   using ThrID   = Layout<_128>;
 3596 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3597 |   using ELayout = GMMA::ELayout_64x64;
 3598 |   using BLayout = GMMA::ABLayout<256, 64>;
 3599 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3601-3602

```text
 3601 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3602 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3604-3613

```text
 3604 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3606 | template <GMMA::SparseSel spsel>
 3607 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_RS_TN<spsel>>
 3608 | {
 3609 |   using ValTypeD = int32_t;
 3610 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3611 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3612 |   using ValTypeB = int8_t;
 3613 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3615-3622

```text
 3615 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3617 |   using Shape_MNK = Shape<_64,_8,_64>;
 3618 |   using ThrID   = Layout<_128>;
 3619 |   using ALayout = GMMA::ALayout_64x64;
 3620 |   using ELayout = GMMA::ELayout_64x64;
 3621 |   using BLayout = GMMA::ABLayout<  8, 64>;
 3622 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3624-3625

```text
 3624 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3625 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3627-3636

```text
 3627 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3629 | template <GMMA::SparseSel spsel>
 3630 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3631 | {
 3632 |   using ValTypeD = int32_t;
 3633 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3634 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3635 |   using ValTypeB = int8_t;
 3636 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3638-3645

```text
 3638 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3640 |   using Shape_MNK = Shape<_64,_8,_64>;
 3641 |   using ThrID   = Layout<_128>;
 3642 |   using ALayout = GMMA::ALayout_64x64;
 3643 |   using ELayout = GMMA::ELayout_64x64;
 3644 |   using BLayout = GMMA::ABLayout<  8, 64>;
 3645 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3647-3648

```text
 3647 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3648 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3650-3659

```text
 3650 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3652 | template <GMMA::SparseSel spsel>
 3653 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_RS_TN<spsel>>
 3654 | {
 3655 |   using ValTypeD = int32_t;
 3656 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3657 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3658 |   using ValTypeB = int8_t;
 3659 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3661-3668

```text
 3661 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3663 |   using Shape_MNK = Shape<_64,_16,_64>;
 3664 |   using ThrID   = Layout<_128>;
 3665 |   using ALayout = GMMA::ALayout_64x64;
 3666 |   using ELayout = GMMA::ELayout_64x64;
 3667 |   using BLayout = GMMA::ABLayout< 16, 64>;
 3668 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3670-3671

```text
 3670 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3671 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3673-3682

```text
 3673 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3675 | template <GMMA::SparseSel spsel>
 3676 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3677 | {
 3678 |   using ValTypeD = int32_t;
 3679 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3680 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3681 |   using ValTypeB = int8_t;
 3682 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3684-3691

```text
 3684 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3686 |   using Shape_MNK = Shape<_64,_16,_64>;
 3687 |   using ThrID   = Layout<_128>;
 3688 |   using ALayout = GMMA::ALayout_64x64;
 3689 |   using ELayout = GMMA::ELayout_64x64;
 3690 |   using BLayout = GMMA::ABLayout< 16, 64>;
 3691 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3693-3694

```text
 3693 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3694 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3696-3705

```text
 3696 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3698 | template <GMMA::SparseSel spsel>
 3699 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_RS_TN<spsel>>
 3700 | {
 3701 |   using ValTypeD = int32_t;
 3702 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3703 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3704 |   using ValTypeB = int8_t;
 3705 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3707-3714

```text
 3707 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3709 |   using Shape_MNK = Shape<_64,_32,_64>;
 3710 |   using ThrID   = Layout<_128>;
 3711 |   using ALayout = GMMA::ALayout_64x64;
 3712 |   using ELayout = GMMA::ELayout_64x64;
 3713 |   using BLayout = GMMA::ABLayout< 32, 64>;
 3714 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3716-3717

```text
 3716 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3717 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3719-3728

```text
 3719 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3721 | template <GMMA::SparseSel spsel>
 3722 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3723 | {
 3724 |   using ValTypeD = int32_t;
 3725 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3726 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3727 |   using ValTypeB = int8_t;
 3728 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3730-3737

```text
 3730 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3732 |   using Shape_MNK = Shape<_64,_32,_64>;
 3733 |   using ThrID   = Layout<_128>;
 3734 |   using ALayout = GMMA::ALayout_64x64;
 3735 |   using ELayout = GMMA::ELayout_64x64;
 3736 |   using BLayout = GMMA::ABLayout< 32, 64>;
 3737 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3739-3740

```text
 3739 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3740 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3742-3751

```text
 3742 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3744 | template <GMMA::SparseSel spsel>
 3745 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_RS_TN<spsel>>
 3746 | {
 3747 |   using ValTypeD = int32_t;
 3748 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3749 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3750 |   using ValTypeB = int8_t;
 3751 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3753-3760

```text
 3753 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3755 |   using Shape_MNK = Shape<_64,_64,_64>;
 3756 |   using ThrID   = Layout<_128>;
 3757 |   using ALayout = GMMA::ALayout_64x64;
 3758 |   using ELayout = GMMA::ELayout_64x64;
 3759 |   using BLayout = GMMA::ABLayout< 64, 64>;
 3760 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3762-3763

```text
 3762 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3763 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3765-3774

```text
 3765 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3767 | template <GMMA::SparseSel spsel>
 3768 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3769 | {
 3770 |   using ValTypeD = int32_t;
 3771 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3772 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3773 |   using ValTypeB = int8_t;
 3774 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3776-3783

```text
 3776 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3778 |   using Shape_MNK = Shape<_64,_64,_64>;
 3779 |   using ThrID   = Layout<_128>;
 3780 |   using ALayout = GMMA::ALayout_64x64;
 3781 |   using ELayout = GMMA::ELayout_64x64;
 3782 |   using BLayout = GMMA::ABLayout< 64, 64>;
 3783 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3785-3786

```text
 3785 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3786 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3788-3797

```text
 3788 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3790 | template <GMMA::SparseSel spsel>
 3791 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_RS_TN<spsel>>
 3792 | {
 3793 |   using ValTypeD = int32_t;
 3794 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3795 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3796 |   using ValTypeB = int8_t;
 3797 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3799-3806

```text
 3799 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3801 |   using Shape_MNK = Shape<_64,_96,_64>;
 3802 |   using ThrID   = Layout<_128>;
 3803 |   using ALayout = GMMA::ALayout_64x64;
 3804 |   using ELayout = GMMA::ELayout_64x64;
 3805 |   using BLayout = GMMA::ABLayout< 96, 64>;
 3806 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3808-3809

```text
 3808 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3809 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3811-3820

```text
 3811 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3813 | template <GMMA::SparseSel spsel>
 3814 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3815 | {
 3816 |   using ValTypeD = int32_t;
 3817 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3818 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3819 |   using ValTypeB = int8_t;
 3820 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3822-3829

```text
 3822 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3824 |   using Shape_MNK = Shape<_64,_96,_64>;
 3825 |   using ThrID   = Layout<_128>;
 3826 |   using ALayout = GMMA::ALayout_64x64;
 3827 |   using ELayout = GMMA::ELayout_64x64;
 3828 |   using BLayout = GMMA::ABLayout< 96, 64>;
 3829 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3831-3832

```text
 3831 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3832 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3834-3843

```text
 3834 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3836 | template <GMMA::SparseSel spsel>
 3837 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_RS_TN<spsel>>
 3838 | {
 3839 |   using ValTypeD = int32_t;
 3840 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3841 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3842 |   using ValTypeB = int8_t;
 3843 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3845-3852

```text
 3845 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3847 |   using Shape_MNK = Shape<_64,_128,_64>;
 3848 |   using ThrID   = Layout<_128>;
 3849 |   using ALayout = GMMA::ALayout_64x64;
 3850 |   using ELayout = GMMA::ELayout_64x64;
 3851 |   using BLayout = GMMA::ABLayout<128, 64>;
 3852 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3854-3855

```text
 3854 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3855 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3857-3866

```text
 3857 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3859 | template <GMMA::SparseSel spsel>
 3860 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3861 | {
 3862 |   using ValTypeD = int32_t;
 3863 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3864 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3865 |   using ValTypeB = int8_t;
 3866 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3868-3875

```text
 3868 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3870 |   using Shape_MNK = Shape<_64,_128,_64>;
 3871 |   using ThrID   = Layout<_128>;
 3872 |   using ALayout = GMMA::ALayout_64x64;
 3873 |   using ELayout = GMMA::ELayout_64x64;
 3874 |   using BLayout = GMMA::ABLayout<128, 64>;
 3875 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3877-3878

```text
 3877 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3878 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3880-3889

```text
 3880 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3882 | template <GMMA::SparseSel spsel>
 3883 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_RS_TN<spsel>>
 3884 | {
 3885 |   using ValTypeD = int32_t;
 3886 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3887 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3888 |   using ValTypeB = int8_t;
 3889 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3891-3898

```text
 3891 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3893 |   using Shape_MNK = Shape<_64,_192,_64>;
 3894 |   using ThrID   = Layout<_128>;
 3895 |   using ALayout = GMMA::ALayout_64x64;
 3896 |   using ELayout = GMMA::ELayout_64x64;
 3897 |   using BLayout = GMMA::ABLayout<192, 64>;
 3898 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3900-3901

```text
 3900 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3901 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3903-3912

```text
 3903 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3905 | template <GMMA::SparseSel spsel>
 3906 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3907 | {
 3908 |   using ValTypeD = int32_t;
 3909 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3910 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3911 |   using ValTypeB = int8_t;
 3912 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3914-3921

```text
 3914 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3916 |   using Shape_MNK = Shape<_64,_192,_64>;
 3917 |   using ThrID   = Layout<_128>;
 3918 |   using ALayout = GMMA::ALayout_64x64;
 3919 |   using ELayout = GMMA::ELayout_64x64;
 3920 |   using BLayout = GMMA::ABLayout<192, 64>;
 3921 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3923-3924

```text
 3923 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3924 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3926-3935

```text
 3926 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3928 | template <GMMA::SparseSel spsel>
 3929 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_RS_TN<spsel>>
 3930 | {
 3931 |   using ValTypeD = int32_t;
 3932 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3933 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3934 |   using ValTypeB = int8_t;
 3935 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3937-3944

```text
 3937 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3939 |   using Shape_MNK = Shape<_64,_256,_64>;
 3940 |   using ThrID   = Layout<_128>;
 3941 |   using ALayout = GMMA::ALayout_64x64;
 3942 |   using ELayout = GMMA::ELayout_64x64;
 3943 |   using BLayout = GMMA::ABLayout<256, 64>;
 3944 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3946-3947

```text
 3946 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3947 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3949-3958

```text
 3949 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3951 | template <GMMA::SparseSel spsel>
 3952 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_RS_TN_SATURATE<spsel>>
 3953 | {
 3954 |   using ValTypeD = int32_t;
 3955 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3956 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3957 |   using ValTypeB = int8_t;
 3958 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8S8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3960-3967

```text
 3960 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3962 |   using Shape_MNK = Shape<_64,_256,_64>;
 3963 |   using ThrID   = Layout<_128>;
 3964 |   using ALayout = GMMA::ALayout_64x64;
 3965 |   using ELayout = GMMA::ELayout_64x64;
 3966 |   using BLayout = GMMA::ABLayout<256, 64>;
 3967 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3969-3970

```text
 3969 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3970 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3972-3981

```text
 3972 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3974 | template <GMMA::SparseSel spsel>
 3975 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_SS_TN<spsel>>
 3976 | {
 3977 |   using ValTypeD = int32_t;
 3978 |   using ValTypeA = sparse_elem<2, uint8_t>;
 3979 |   using ValTypeE = sparse_elem<8, uint8_t>;
 3980 |   using ValTypeB = uint8_t;
 3981 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3983-3991

```text
 3983 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3984 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3986 |   using Shape_MNK = Shape<_64,_8,_64>;
 3987 |   using ThrID   = Layout<_128>;
 3988 |   using ALayout = GMMA::ABLayout< 64, 64>;
 3989 |   using ELayout = GMMA::ELayout_64x64;
 3990 |   using BLayout = GMMA::ABLayout<  8, 64>;
 3991 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3993-3994

```text
 3993 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3994 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3996-4005

```text
 3996 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3998 | template <GMMA::SparseSel spsel>
 3999 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4000 | {
 4001 |   using ValTypeD = int32_t;
 4002 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4003 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4004 |   using ValTypeB = uint8_t;
 4005 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4007-4015

```text
 4007 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4008 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4010 |   using Shape_MNK = Shape<_64,_8,_64>;
 4011 |   using ThrID   = Layout<_128>;
 4012 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4013 |   using ELayout = GMMA::ELayout_64x64;
 4014 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4015 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4017-4018

```text
 4017 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4018 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4020-4029

```text
 4020 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4022 | template <GMMA::SparseSel spsel>
 4023 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_SS_TN<spsel>>
 4024 | {
 4025 |   using ValTypeD = int32_t;
 4026 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4027 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4028 |   using ValTypeB = uint8_t;
 4029 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4031-4039

```text
 4031 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4032 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4034 |   using Shape_MNK = Shape<_64,_16,_64>;
 4035 |   using ThrID   = Layout<_128>;
 4036 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4037 |   using ELayout = GMMA::ELayout_64x64;
 4038 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4039 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4041-4042

```text
 4041 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4042 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4044-4053

```text
 4044 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4046 | template <GMMA::SparseSel spsel>
 4047 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4048 | {
 4049 |   using ValTypeD = int32_t;
 4050 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4051 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4052 |   using ValTypeB = uint8_t;
 4053 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4055-4063

```text
 4055 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4056 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4058 |   using Shape_MNK = Shape<_64,_16,_64>;
 4059 |   using ThrID   = Layout<_128>;
 4060 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4061 |   using ELayout = GMMA::ELayout_64x64;
 4062 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4063 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4065-4066

```text
 4065 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4066 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4068-4077

```text
 4068 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4070 | template <GMMA::SparseSel spsel>
 4071 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_SS_TN<spsel>>
 4072 | {
 4073 |   using ValTypeD = int32_t;
 4074 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4075 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4076 |   using ValTypeB = uint8_t;
 4077 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4079-4087

```text
 4079 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4080 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4082 |   using Shape_MNK = Shape<_64,_32,_64>;
 4083 |   using ThrID   = Layout<_128>;
 4084 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4085 |   using ELayout = GMMA::ELayout_64x64;
 4086 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4087 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4089-4090

```text
 4089 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4090 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4092-4101

```text
 4092 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4094 | template <GMMA::SparseSel spsel>
 4095 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4096 | {
 4097 |   using ValTypeD = int32_t;
 4098 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4099 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4100 |   using ValTypeB = uint8_t;
 4101 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4103-4111

```text
 4103 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4104 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4106 |   using Shape_MNK = Shape<_64,_32,_64>;
 4107 |   using ThrID   = Layout<_128>;
 4108 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4109 |   using ELayout = GMMA::ELayout_64x64;
 4110 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4111 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4113-4114

```text
 4113 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4114 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4116-4125

```text
 4116 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4118 | template <GMMA::SparseSel spsel>
 4119 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_SS_TN<spsel>>
 4120 | {
 4121 |   using ValTypeD = int32_t;
 4122 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4123 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4124 |   using ValTypeB = uint8_t;
 4125 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4127-4135

```text
 4127 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4128 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4130 |   using Shape_MNK = Shape<_64,_64,_64>;
 4131 |   using ThrID   = Layout<_128>;
 4132 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4133 |   using ELayout = GMMA::ELayout_64x64;
 4134 |   using BLayout = GMMA::ABLayout< 64, 64>;
 4135 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4137-4138

```text
 4137 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4138 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4140-4149

```text
 4140 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4142 | template <GMMA::SparseSel spsel>
 4143 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4144 | {
 4145 |   using ValTypeD = int32_t;
 4146 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4147 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4148 |   using ValTypeB = uint8_t;
 4149 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4151-4159

```text
 4151 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4152 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4154 |   using Shape_MNK = Shape<_64,_64,_64>;
 4155 |   using ThrID   = Layout<_128>;
 4156 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4157 |   using ELayout = GMMA::ELayout_64x64;
 4158 |   using BLayout = GMMA::ABLayout< 64, 64>;
 4159 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4161-4162

```text
 4161 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4162 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4164-4173

```text
 4164 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4166 | template <GMMA::SparseSel spsel>
 4167 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_SS_TN<spsel>>
 4168 | {
 4169 |   using ValTypeD = int32_t;
 4170 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4171 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4172 |   using ValTypeB = uint8_t;
 4173 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4175-4183

```text
 4175 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4176 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4178 |   using Shape_MNK = Shape<_64,_96,_64>;
 4179 |   using ThrID   = Layout<_128>;
 4180 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4181 |   using ELayout = GMMA::ELayout_64x64;
 4182 |   using BLayout = GMMA::ABLayout< 96, 64>;
 4183 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4185-4186

```text
 4185 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4186 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4188-4197

```text
 4188 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4190 | template <GMMA::SparseSel spsel>
 4191 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4192 | {
 4193 |   using ValTypeD = int32_t;
 4194 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4195 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4196 |   using ValTypeB = uint8_t;
 4197 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4199-4207

```text
 4199 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4200 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4202 |   using Shape_MNK = Shape<_64,_96,_64>;
 4203 |   using ThrID   = Layout<_128>;
 4204 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4205 |   using ELayout = GMMA::ELayout_64x64;
 4206 |   using BLayout = GMMA::ABLayout< 96, 64>;
 4207 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4209-4210

```text
 4209 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4210 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4212-4221

```text
 4212 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4214 | template <GMMA::SparseSel spsel>
 4215 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_SS_TN<spsel>>
 4216 | {
 4217 |   using ValTypeD = int32_t;
 4218 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4219 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4220 |   using ValTypeB = uint8_t;
 4221 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4223-4231

```text
 4223 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4224 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4226 |   using Shape_MNK = Shape<_64,_128,_64>;
 4227 |   using ThrID   = Layout<_128>;
 4228 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4229 |   using ELayout = GMMA::ELayout_64x64;
 4230 |   using BLayout = GMMA::ABLayout<128, 64>;
 4231 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4233-4234

```text
 4233 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4234 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4236-4245

```text
 4236 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4238 | template <GMMA::SparseSel spsel>
 4239 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4240 | {
 4241 |   using ValTypeD = int32_t;
 4242 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4243 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4244 |   using ValTypeB = uint8_t;
 4245 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4247-4255

```text
 4247 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4248 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4250 |   using Shape_MNK = Shape<_64,_128,_64>;
 4251 |   using ThrID   = Layout<_128>;
 4252 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4253 |   using ELayout = GMMA::ELayout_64x64;
 4254 |   using BLayout = GMMA::ABLayout<128, 64>;
 4255 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4257-4258

```text
 4257 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4258 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4260-4269

```text
 4260 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4262 | template <GMMA::SparseSel spsel>
 4263 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_SS_TN<spsel>>
 4264 | {
 4265 |   using ValTypeD = int32_t;
 4266 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4267 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4268 |   using ValTypeB = uint8_t;
 4269 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4271-4279

```text
 4271 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4272 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4274 |   using Shape_MNK = Shape<_64,_192,_64>;
 4275 |   using ThrID   = Layout<_128>;
 4276 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4277 |   using ELayout = GMMA::ELayout_64x64;
 4278 |   using BLayout = GMMA::ABLayout<192, 64>;
 4279 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4281-4282

```text
 4281 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4282 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4284-4293

```text
 4284 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4286 | template <GMMA::SparseSel spsel>
 4287 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4288 | {
 4289 |   using ValTypeD = int32_t;
 4290 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4291 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4292 |   using ValTypeB = uint8_t;
 4293 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4295-4303

```text
 4295 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4296 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4298 |   using Shape_MNK = Shape<_64,_192,_64>;
 4299 |   using ThrID   = Layout<_128>;
 4300 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4301 |   using ELayout = GMMA::ELayout_64x64;
 4302 |   using BLayout = GMMA::ABLayout<192, 64>;
 4303 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4305-4306

```text
 4305 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4306 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4308-4317

```text
 4308 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4310 | template <GMMA::SparseSel spsel>
 4311 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_SS_TN<spsel>>
 4312 | {
 4313 |   using ValTypeD = int32_t;
 4314 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4315 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4316 |   using ValTypeB = uint8_t;
 4317 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_SS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_SS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4319-4327

```text
 4319 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4320 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4322 |   using Shape_MNK = Shape<_64,_256,_64>;
 4323 |   using ThrID   = Layout<_128>;
 4324 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4325 |   using ELayout = GMMA::ELayout_64x64;
 4326 |   using BLayout = GMMA::ABLayout<256, 64>;
 4327 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4329-4330

```text
 4329 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4330 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4332-4341

```text
 4332 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4334 | template <GMMA::SparseSel spsel>
 4335 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_SS_TN_SATURATE<spsel>>
 4336 | {
 4337 |   using ValTypeD = int32_t;
 4338 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4339 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4340 |   using ValTypeB = uint8_t;
 4341 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_SS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_SS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4343-4351

```text
 4343 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4344 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4346 |   using Shape_MNK = Shape<_64,_256,_64>;
 4347 |   using ThrID   = Layout<_128>;
 4348 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4349 |   using ELayout = GMMA::ELayout_64x64;
 4350 |   using BLayout = GMMA::ABLayout<256, 64>;
 4351 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4353-4354

```text
 4353 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4354 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4356-4365

```text
 4356 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4358 | template <GMMA::SparseSel spsel>
 4359 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_RS_TN<spsel>>
 4360 | {
 4361 |   using ValTypeD = int32_t;
 4362 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4363 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4364 |   using ValTypeB = uint8_t;
 4365 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4367-4374

```text
 4367 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4369 |   using Shape_MNK = Shape<_64,_8,_64>;
 4370 |   using ThrID   = Layout<_128>;
 4371 |   using ALayout = GMMA::ALayout_64x64;
 4372 |   using ELayout = GMMA::ELayout_64x64;
 4373 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4374 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4376-4377

```text
 4376 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4377 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4379-4388

```text
 4379 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4381 | template <GMMA::SparseSel spsel>
 4382 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4383 | {
 4384 |   using ValTypeD = int32_t;
 4385 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4386 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4387 |   using ValTypeB = uint8_t;
 4388 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4390-4397

```text
 4390 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4392 |   using Shape_MNK = Shape<_64,_8,_64>;
 4393 |   using ThrID   = Layout<_128>;
 4394 |   using ALayout = GMMA::ALayout_64x64;
 4395 |   using ELayout = GMMA::ELayout_64x64;
 4396 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4397 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4399-4400

```text
 4399 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4400 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4402-4411

```text
 4402 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4404 | template <GMMA::SparseSel spsel>
 4405 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_RS_TN<spsel>>
 4406 | {
 4407 |   using ValTypeD = int32_t;
 4408 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4409 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4410 |   using ValTypeB = uint8_t;
 4411 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4413-4420

```text
 4413 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4415 |   using Shape_MNK = Shape<_64,_16,_64>;
 4416 |   using ThrID   = Layout<_128>;
 4417 |   using ALayout = GMMA::ALayout_64x64;
 4418 |   using ELayout = GMMA::ELayout_64x64;
 4419 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4420 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4422-4423

```text
 4422 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4423 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4425-4434

```text
 4425 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4427 | template <GMMA::SparseSel spsel>
 4428 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4429 | {
 4430 |   using ValTypeD = int32_t;
 4431 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4432 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4433 |   using ValTypeB = uint8_t;
 4434 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4436-4443

```text
 4436 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4438 |   using Shape_MNK = Shape<_64,_16,_64>;
 4439 |   using ThrID   = Layout<_128>;
 4440 |   using ALayout = GMMA::ALayout_64x64;
 4441 |   using ELayout = GMMA::ELayout_64x64;
 4442 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4443 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4445-4446

```text
 4445 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4446 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4448-4457

```text
 4448 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4450 | template <GMMA::SparseSel spsel>
 4451 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_RS_TN<spsel>>
 4452 | {
 4453 |   using ValTypeD = int32_t;
 4454 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4455 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4456 |   using ValTypeB = uint8_t;
 4457 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4459-4466

```text
 4459 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4461 |   using Shape_MNK = Shape<_64,_32,_64>;
 4462 |   using ThrID   = Layout<_128>;
 4463 |   using ALayout = GMMA::ALayout_64x64;
 4464 |   using ELayout = GMMA::ELayout_64x64;
 4465 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4466 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4468-4469

```text
 4468 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4469 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4471-4480

```text
 4471 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4473 | template <GMMA::SparseSel spsel>
 4474 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4475 | {
 4476 |   using ValTypeD = int32_t;
 4477 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4478 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4479 |   using ValTypeB = uint8_t;
 4480 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4482-4489

```text
 4482 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4484 |   using Shape_MNK = Shape<_64,_32,_64>;
 4485 |   using ThrID   = Layout<_128>;
 4486 |   using ALayout = GMMA::ALayout_64x64;
 4487 |   using ELayout = GMMA::ELayout_64x64;
 4488 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4489 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4491-4492

```text
 4491 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4492 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4494-4503

```text
 4494 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4496 | template <GMMA::SparseSel spsel>
 4497 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_RS_TN<spsel>>
 4498 | {
 4499 |   using ValTypeD = int32_t;
 4500 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4501 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4502 |   using ValTypeB = uint8_t;
 4503 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4505-4512

```text
 4505 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4507 |   using Shape_MNK = Shape<_64,_64,_64>;
 4508 |   using ThrID   = Layout<_128>;
 4509 |   using ALayout = GMMA::ALayout_64x64;
 4510 |   using ELayout = GMMA::ELayout_64x64;
 4511 |   using BLayout = GMMA::ABLayout< 64, 64>;
 4512 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4514-4515

```text
 4514 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4515 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4517-4526

```text
 4517 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4519 | template <GMMA::SparseSel spsel>
 4520 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4521 | {
 4522 |   using ValTypeD = int32_t;
 4523 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4524 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4525 |   using ValTypeB = uint8_t;
 4526 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4528-4535

```text
 4528 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4530 |   using Shape_MNK = Shape<_64,_64,_64>;
 4531 |   using ThrID   = Layout<_128>;
 4532 |   using ALayout = GMMA::ALayout_64x64;
 4533 |   using ELayout = GMMA::ELayout_64x64;
 4534 |   using BLayout = GMMA::ABLayout< 64, 64>;
 4535 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4537-4538

```text
 4537 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4538 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4540-4549

```text
 4540 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4542 | template <GMMA::SparseSel spsel>
 4543 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_RS_TN<spsel>>
 4544 | {
 4545 |   using ValTypeD = int32_t;
 4546 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4547 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4548 |   using ValTypeB = uint8_t;
 4549 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4551-4558

```text
 4551 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4553 |   using Shape_MNK = Shape<_64,_96,_64>;
 4554 |   using ThrID   = Layout<_128>;
 4555 |   using ALayout = GMMA::ALayout_64x64;
 4556 |   using ELayout = GMMA::ELayout_64x64;
 4557 |   using BLayout = GMMA::ABLayout< 96, 64>;
 4558 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4560-4561

```text
 4560 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4561 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4563-4572

```text
 4563 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4565 | template <GMMA::SparseSel spsel>
 4566 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4567 | {
 4568 |   using ValTypeD = int32_t;
 4569 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4570 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4571 |   using ValTypeB = uint8_t;
 4572 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4574-4581

```text
 4574 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4576 |   using Shape_MNK = Shape<_64,_96,_64>;
 4577 |   using ThrID   = Layout<_128>;
 4578 |   using ALayout = GMMA::ALayout_64x64;
 4579 |   using ELayout = GMMA::ELayout_64x64;
 4580 |   using BLayout = GMMA::ABLayout< 96, 64>;
 4581 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4583-4584

```text
 4583 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4584 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4586-4595

```text
 4586 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4588 | template <GMMA::SparseSel spsel>
 4589 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_RS_TN<spsel>>
 4590 | {
 4591 |   using ValTypeD = int32_t;
 4592 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4593 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4594 |   using ValTypeB = uint8_t;
 4595 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4597-4604

```text
 4597 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4599 |   using Shape_MNK = Shape<_64,_128,_64>;
 4600 |   using ThrID   = Layout<_128>;
 4601 |   using ALayout = GMMA::ALayout_64x64;
 4602 |   using ELayout = GMMA::ELayout_64x64;
 4603 |   using BLayout = GMMA::ABLayout<128, 64>;
 4604 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4606-4607

```text
 4606 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4607 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4609-4618

```text
 4609 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4611 | template <GMMA::SparseSel spsel>
 4612 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4613 | {
 4614 |   using ValTypeD = int32_t;
 4615 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4616 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4617 |   using ValTypeB = uint8_t;
 4618 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4620-4627

```text
 4620 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4622 |   using Shape_MNK = Shape<_64,_128,_64>;
 4623 |   using ThrID   = Layout<_128>;
 4624 |   using ALayout = GMMA::ALayout_64x64;
 4625 |   using ELayout = GMMA::ELayout_64x64;
 4626 |   using BLayout = GMMA::ABLayout<128, 64>;
 4627 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4629-4630

```text
 4629 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4630 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4632-4641

```text
 4632 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4634 | template <GMMA::SparseSel spsel>
 4635 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_RS_TN<spsel>>
 4636 | {
 4637 |   using ValTypeD = int32_t;
 4638 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4639 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4640 |   using ValTypeB = uint8_t;
 4641 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4643-4650

```text
 4643 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4645 |   using Shape_MNK = Shape<_64,_192,_64>;
 4646 |   using ThrID   = Layout<_128>;
 4647 |   using ALayout = GMMA::ALayout_64x64;
 4648 |   using ELayout = GMMA::ELayout_64x64;
 4649 |   using BLayout = GMMA::ABLayout<192, 64>;
 4650 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4652-4653

```text
 4652 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4653 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4655-4664

```text
 4655 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4657 | template <GMMA::SparseSel spsel>
 4658 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4659 | {
 4660 |   using ValTypeD = int32_t;
 4661 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4662 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4663 |   using ValTypeB = uint8_t;
 4664 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4666-4673

```text
 4666 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4668 |   using Shape_MNK = Shape<_64,_192,_64>;
 4669 |   using ThrID   = Layout<_128>;
 4670 |   using ALayout = GMMA::ALayout_64x64;
 4671 |   using ELayout = GMMA::ELayout_64x64;
 4672 |   using BLayout = GMMA::ABLayout<192, 64>;
 4673 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4675-4676

```text
 4675 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4676 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4678-4687

```text
 4678 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4680 | template <GMMA::SparseSel spsel>
 4681 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_RS_TN<spsel>>
 4682 | {
 4683 |   using ValTypeD = int32_t;
 4684 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4685 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4686 |   using ValTypeB = uint8_t;
 4687 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_RS_TN<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_RS_TN<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4689-4696

```text
 4689 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4691 |   using Shape_MNK = Shape<_64,_256,_64>;
 4692 |   using ThrID   = Layout<_128>;
 4693 |   using ALayout = GMMA::ALayout_64x64;
 4694 |   using ELayout = GMMA::ELayout_64x64;
 4695 |   using BLayout = GMMA::ABLayout<256, 64>;
 4696 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4698-4699

```text
 4698 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4699 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4701-4710

```text
 4701 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4703 | template <GMMA::SparseSel spsel>
 4704 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_RS_TN_SATURATE<spsel>>
 4705 | {
 4706 |   using ValTypeD = int32_t;
 4707 |   using ValTypeA = sparse_elem<2, uint8_t>;
 4708 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4709 |   using ValTypeB = uint8_t;
 4710 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_RS_TN_SATURATE<spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_S32U8U8_RS_TN_SATURATE<spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4712-4719

```text
 4712 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4714 |   using Shape_MNK = Shape<_64,_256,_64>;
 4715 |   using ThrID   = Layout<_128>;
 4716 |   using ALayout = GMMA::ALayout_64x64;
 4717 |   using ELayout = GMMA::ELayout_64x64;
 4718 |   using BLayout = GMMA::ABLayout<256, 64>;
 4719 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4721-4722

```text
 4721 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4722 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4724-4733

```text
 4724 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4726 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4727 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 4728 | {
 4729 |   using ValTypeD = half_t;
 4730 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4731 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4732 |   using ValTypeB = float_e4m3_t;
 4733 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4735-4743

```text
 4735 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4736 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4738 |   using Shape_MNK = Shape<_64,_8,_64>;
 4739 |   using ThrID   = Layout<_128>;
 4740 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4741 |   using ELayout = GMMA::ELayout_64x64;
 4742 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4743 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4745-4746

```text
 4745 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4746 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4748-4757

```text
 4748 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4750 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4751 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 4752 | {
 4753 |   using ValTypeD = half_t;
 4754 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4755 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4756 |   using ValTypeB = float_e4m3_t;
 4757 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4759-4766

```text
 4759 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4761 |   using Shape_MNK = Shape<_64,_8,_64>;
 4762 |   using ThrID   = Layout<_128>;
 4763 |   using ALayout = GMMA::ALayout_64x64;
 4764 |   using ELayout = GMMA::ELayout_64x64;
 4765 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4766 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4768-4769

```text
 4768 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4769 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4771-4780

```text
 4771 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4773 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4774 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 4775 | {
 4776 |   using ValTypeD = float;
 4777 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4778 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4779 |   using ValTypeB = float_e4m3_t;
 4780 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4782-4790

```text
 4782 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4783 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4785 |   using Shape_MNK = Shape<_64,_8,_64>;
 4786 |   using ThrID   = Layout<_128>;
 4787 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4788 |   using ELayout = GMMA::ELayout_64x64;
 4789 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4790 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4792-4793

```text
 4792 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4793 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4795-4804

```text
 4795 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4797 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4798 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 4799 | {
 4800 |   using ValTypeD = float;
 4801 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4802 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4803 |   using ValTypeB = float_e4m3_t;
 4804 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4806-4813

```text
 4806 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4808 |   using Shape_MNK = Shape<_64,_8,_64>;
 4809 |   using ThrID   = Layout<_128>;
 4810 |   using ALayout = GMMA::ALayout_64x64;
 4811 |   using ELayout = GMMA::ELayout_64x64;
 4812 |   using BLayout = GMMA::ABLayout<  8, 64>;
 4813 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4815-4816

```text
 4815 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4816 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4818-4827

```text
 4818 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4820 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4821 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 4822 | {
 4823 |   using ValTypeD = half_t;
 4824 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4825 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4826 |   using ValTypeB = float_e4m3_t;
 4827 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4829-4837

```text
 4829 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4830 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4832 |   using Shape_MNK = Shape<_64,_16,_64>;
 4833 |   using ThrID   = Layout<_128>;
 4834 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4835 |   using ELayout = GMMA::ELayout_64x64;
 4836 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4837 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4839-4840

```text
 4839 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4840 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4842-4851

```text
 4842 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4844 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4845 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 4846 | {
 4847 |   using ValTypeD = half_t;
 4848 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4849 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4850 |   using ValTypeB = float_e4m3_t;
 4851 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4853-4860

```text
 4853 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4855 |   using Shape_MNK = Shape<_64,_16,_64>;
 4856 |   using ThrID   = Layout<_128>;
 4857 |   using ALayout = GMMA::ALayout_64x64;
 4858 |   using ELayout = GMMA::ELayout_64x64;
 4859 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4860 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4862-4863

```text
 4862 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4863 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4865-4874

```text
 4865 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4867 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4868 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 4869 | {
 4870 |   using ValTypeD = float;
 4871 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4872 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4873 |   using ValTypeB = float_e4m3_t;
 4874 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4876-4884

```text
 4876 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4877 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4879 |   using Shape_MNK = Shape<_64,_16,_64>;
 4880 |   using ThrID   = Layout<_128>;
 4881 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4882 |   using ELayout = GMMA::ELayout_64x64;
 4883 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4884 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4886-4887

```text
 4886 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4887 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4889-4898

```text
 4889 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4891 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4892 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 4893 | {
 4894 |   using ValTypeD = float;
 4895 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4896 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4897 |   using ValTypeB = float_e4m3_t;
 4898 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4900-4907

```text
 4900 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4902 |   using Shape_MNK = Shape<_64,_16,_64>;
 4903 |   using ThrID   = Layout<_128>;
 4904 |   using ALayout = GMMA::ALayout_64x64;
 4905 |   using ELayout = GMMA::ELayout_64x64;
 4906 |   using BLayout = GMMA::ABLayout< 16, 64>;
 4907 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4909-4910

```text
 4909 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4910 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4912-4921

```text
 4912 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4914 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4915 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 4916 | {
 4917 |   using ValTypeD = half_t;
 4918 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4919 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4920 |   using ValTypeB = float_e4m3_t;
 4921 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4923-4931

```text
 4923 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4924 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4926 |   using Shape_MNK = Shape<_64,_32,_64>;
 4927 |   using ThrID   = Layout<_128>;
 4928 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4929 |   using ELayout = GMMA::ELayout_64x64;
 4930 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4931 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4933-4934

```text
 4933 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4934 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4936-4945

```text
 4936 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4938 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4939 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 4940 | {
 4941 |   using ValTypeD = half_t;
 4942 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4943 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4944 |   using ValTypeB = float_e4m3_t;
 4945 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4947-4954

```text
 4947 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4949 |   using Shape_MNK = Shape<_64,_32,_64>;
 4950 |   using ThrID   = Layout<_128>;
 4951 |   using ALayout = GMMA::ALayout_64x64;
 4952 |   using ELayout = GMMA::ELayout_64x64;
 4953 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4954 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4956-4957

```text
 4956 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4957 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4959-4968

```text
 4959 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4961 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4962 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 4963 | {
 4964 |   using ValTypeD = float;
 4965 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4966 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4967 |   using ValTypeB = float_e4m3_t;
 4968 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4970-4978

```text
 4970 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4971 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4973 |   using Shape_MNK = Shape<_64,_32,_64>;
 4974 |   using ThrID   = Layout<_128>;
 4975 |   using ALayout = GMMA::ABLayout< 64, 64>;
 4976 |   using ELayout = GMMA::ELayout_64x64;
 4977 |   using BLayout = GMMA::ABLayout< 32, 64>;
 4978 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4980-4981

```text
 4980 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4981 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4983-4992

```text
 4983 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4985 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 4986 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 4987 | {
 4988 |   using ValTypeD = float;
 4989 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 4990 |   using ValTypeE = sparse_elem<8, uint8_t>;
 4991 |   using ValTypeB = float_e4m3_t;
 4992 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4994-5001

```text
 4994 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4996 |   using Shape_MNK = Shape<_64,_32,_64>;
 4997 |   using ThrID   = Layout<_128>;
 4998 |   using ALayout = GMMA::ALayout_64x64;
 4999 |   using ELayout = GMMA::ELayout_64x64;
 5000 |   using BLayout = GMMA::ABLayout< 32, 64>;
 5001 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5003-5004

```text
 5003 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5004 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5006-5015

```text
 5006 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5008 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5009 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5010 | {
 5011 |   using ValTypeD = half_t;
 5012 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5013 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5014 |   using ValTypeB = float_e4m3_t;
 5015 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5017-5025

```text
 5017 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5018 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5020 |   using Shape_MNK = Shape<_64,_64,_64>;
 5021 |   using ThrID   = Layout<_128>;
 5022 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5023 |   using ELayout = GMMA::ELayout_64x64;
 5024 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5025 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5027-5028

```text
 5027 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5028 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5030-5039

```text
 5030 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5032 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5033 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5034 | {
 5035 |   using ValTypeD = half_t;
 5036 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5037 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5038 |   using ValTypeB = float_e4m3_t;
 5039 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5041-5048

```text
 5041 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5043 |   using Shape_MNK = Shape<_64,_64,_64>;
 5044 |   using ThrID   = Layout<_128>;
 5045 |   using ALayout = GMMA::ALayout_64x64;
 5046 |   using ELayout = GMMA::ELayout_64x64;
 5047 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5048 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5050-5051

```text
 5050 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5051 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5053-5062

```text
 5053 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5055 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5056 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5057 | {
 5058 |   using ValTypeD = float;
 5059 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5060 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5061 |   using ValTypeB = float_e4m3_t;
 5062 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5064-5072

```text
 5064 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5065 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5067 |   using Shape_MNK = Shape<_64,_64,_64>;
 5068 |   using ThrID   = Layout<_128>;
 5069 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5070 |   using ELayout = GMMA::ELayout_64x64;
 5071 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5072 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5074-5075

```text
 5074 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5075 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5077-5086

```text
 5077 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5079 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5080 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5081 | {
 5082 |   using ValTypeD = float;
 5083 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5084 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5085 |   using ValTypeB = float_e4m3_t;
 5086 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5088-5095

```text
 5088 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5090 |   using Shape_MNK = Shape<_64,_64,_64>;
 5091 |   using ThrID   = Layout<_128>;
 5092 |   using ALayout = GMMA::ALayout_64x64;
 5093 |   using ELayout = GMMA::ELayout_64x64;
 5094 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5095 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5097-5098

```text
 5097 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5098 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5100-5109

```text
 5100 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5102 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5103 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5104 | {
 5105 |   using ValTypeD = half_t;
 5106 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5107 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5108 |   using ValTypeB = float_e4m3_t;
 5109 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5111-5119

```text
 5111 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5112 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5114 |   using Shape_MNK = Shape<_64,_96,_64>;
 5115 |   using ThrID   = Layout<_128>;
 5116 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5117 |   using ELayout = GMMA::ELayout_64x64;
 5118 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5119 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5121-5122

```text
 5121 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5122 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5124-5133

```text
 5124 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5126 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5127 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5128 | {
 5129 |   using ValTypeD = half_t;
 5130 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5131 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5132 |   using ValTypeB = float_e4m3_t;
 5133 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5135-5142

```text
 5135 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5137 |   using Shape_MNK = Shape<_64,_96,_64>;
 5138 |   using ThrID   = Layout<_128>;
 5139 |   using ALayout = GMMA::ALayout_64x64;
 5140 |   using ELayout = GMMA::ELayout_64x64;
 5141 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5142 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5144-5145

```text
 5144 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5145 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5147-5156

```text
 5147 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5149 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5150 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5151 | {
 5152 |   using ValTypeD = float;
 5153 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5154 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5155 |   using ValTypeB = float_e4m3_t;
 5156 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5158-5166

```text
 5158 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5159 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5161 |   using Shape_MNK = Shape<_64,_96,_64>;
 5162 |   using ThrID   = Layout<_128>;
 5163 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5164 |   using ELayout = GMMA::ELayout_64x64;
 5165 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5166 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5168-5169

```text
 5168 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5169 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5171-5180

```text
 5171 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5173 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5174 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5175 | {
 5176 |   using ValTypeD = float;
 5177 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5178 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5179 |   using ValTypeB = float_e4m3_t;
 5180 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5182-5189

```text
 5182 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5184 |   using Shape_MNK = Shape<_64,_96,_64>;
 5185 |   using ThrID   = Layout<_128>;
 5186 |   using ALayout = GMMA::ALayout_64x64;
 5187 |   using ELayout = GMMA::ELayout_64x64;
 5188 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5189 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5191-5192

```text
 5191 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5192 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5194-5203

```text
 5194 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5196 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5197 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5198 | {
 5199 |   using ValTypeD = half_t;
 5200 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5201 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5202 |   using ValTypeB = float_e4m3_t;
 5203 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5205-5213

```text
 5205 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5206 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5208 |   using Shape_MNK = Shape<_64,_128,_64>;
 5209 |   using ThrID   = Layout<_128>;
 5210 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5211 |   using ELayout = GMMA::ELayout_64x64;
 5212 |   using BLayout = GMMA::ABLayout<128, 64>;
 5213 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5215-5216

```text
 5215 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5216 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5218-5227

```text
 5218 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5220 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5221 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5222 | {
 5223 |   using ValTypeD = half_t;
 5224 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5225 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5226 |   using ValTypeB = float_e4m3_t;
 5227 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5229-5236

```text
 5229 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5231 |   using Shape_MNK = Shape<_64,_128,_64>;
 5232 |   using ThrID   = Layout<_128>;
 5233 |   using ALayout = GMMA::ALayout_64x64;
 5234 |   using ELayout = GMMA::ELayout_64x64;
 5235 |   using BLayout = GMMA::ABLayout<128, 64>;
 5236 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5238-5239

```text
 5238 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5239 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5241-5250

```text
 5241 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5243 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5244 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5245 | {
 5246 |   using ValTypeD = float;
 5247 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5248 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5249 |   using ValTypeB = float_e4m3_t;
 5250 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5252-5260

```text
 5252 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5253 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5255 |   using Shape_MNK = Shape<_64,_128,_64>;
 5256 |   using ThrID   = Layout<_128>;
 5257 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5258 |   using ELayout = GMMA::ELayout_64x64;
 5259 |   using BLayout = GMMA::ABLayout<128, 64>;
 5260 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5262-5263

```text
 5262 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5263 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5265-5274

```text
 5265 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5267 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5268 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5269 | {
 5270 |   using ValTypeD = float;
 5271 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5272 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5273 |   using ValTypeB = float_e4m3_t;
 5274 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5276-5283

```text
 5276 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5278 |   using Shape_MNK = Shape<_64,_128,_64>;
 5279 |   using ThrID   = Layout<_128>;
 5280 |   using ALayout = GMMA::ALayout_64x64;
 5281 |   using ELayout = GMMA::ELayout_64x64;
 5282 |   using BLayout = GMMA::ABLayout<128, 64>;
 5283 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5285-5286

```text
 5285 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5286 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5288-5297

```text
 5288 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5290 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5291 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5292 | {
 5293 |   using ValTypeD = half_t;
 5294 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5295 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5296 |   using ValTypeB = float_e4m3_t;
 5297 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5299-5307

```text
 5299 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5300 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5302 |   using Shape_MNK = Shape<_64,_192,_64>;
 5303 |   using ThrID   = Layout<_128>;
 5304 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5305 |   using ELayout = GMMA::ELayout_64x64;
 5306 |   using BLayout = GMMA::ABLayout<192, 64>;
 5307 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5309-5310

```text
 5309 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5310 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5312-5321

```text
 5312 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5314 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5315 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5316 | {
 5317 |   using ValTypeD = half_t;
 5318 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5319 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5320 |   using ValTypeB = float_e4m3_t;
 5321 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5323-5330

```text
 5323 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5325 |   using Shape_MNK = Shape<_64,_192,_64>;
 5326 |   using ThrID   = Layout<_128>;
 5327 |   using ALayout = GMMA::ALayout_64x64;
 5328 |   using ELayout = GMMA::ELayout_64x64;
 5329 |   using BLayout = GMMA::ABLayout<192, 64>;
 5330 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5332-5333

```text
 5332 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5333 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5335-5344

```text
 5335 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5337 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5338 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5339 | {
 5340 |   using ValTypeD = float;
 5341 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5342 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5343 |   using ValTypeB = float_e4m3_t;
 5344 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5346-5354

```text
 5346 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5347 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5349 |   using Shape_MNK = Shape<_64,_192,_64>;
 5350 |   using ThrID   = Layout<_128>;
 5351 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5352 |   using ELayout = GMMA::ELayout_64x64;
 5353 |   using BLayout = GMMA::ABLayout<192, 64>;
 5354 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5356-5357

```text
 5356 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5357 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5359-5368

```text
 5359 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5361 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5362 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5363 | {
 5364 |   using ValTypeD = float;
 5365 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5366 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5367 |   using ValTypeB = float_e4m3_t;
 5368 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5370-5377

```text
 5370 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5372 |   using Shape_MNK = Shape<_64,_192,_64>;
 5373 |   using ThrID   = Layout<_128>;
 5374 |   using ALayout = GMMA::ALayout_64x64;
 5375 |   using ELayout = GMMA::ELayout_64x64;
 5376 |   using BLayout = GMMA::ABLayout<192, 64>;
 5377 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5379-5380

```text
 5379 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5380 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5382-5391

```text
 5382 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5384 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5385 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5386 | {
 5387 |   using ValTypeD = half_t;
 5388 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5389 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5390 |   using ValTypeB = float_e4m3_t;
 5391 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5393-5401

```text
 5393 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5394 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5396 |   using Shape_MNK = Shape<_64,_256,_64>;
 5397 |   using ThrID   = Layout<_128>;
 5398 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5399 |   using ELayout = GMMA::ELayout_64x64;
 5400 |   using BLayout = GMMA::ABLayout<256, 64>;
 5401 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5403-5404

```text
 5403 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5404 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5406-5415

```text
 5406 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5408 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5409 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5410 | {
 5411 |   using ValTypeD = half_t;
 5412 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5413 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5414 |   using ValTypeB = float_e4m3_t;
 5415 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5417-5424

```text
 5417 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5419 |   using Shape_MNK = Shape<_64,_256,_64>;
 5420 |   using ThrID   = Layout<_128>;
 5421 |   using ALayout = GMMA::ALayout_64x64;
 5422 |   using ELayout = GMMA::ELayout_64x64;
 5423 |   using BLayout = GMMA::ABLayout<256, 64>;
 5424 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5426-5427

```text
 5426 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5427 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5429-5438

```text
 5429 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5431 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5432 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel>>
 5433 | {
 5434 |   using ValTypeD = float;
 5435 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5436 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5437 |   using ValTypeB = float_e4m3_t;
 5438 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5440-5448

```text
 5440 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5441 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5443 |   using Shape_MNK = Shape<_64,_256,_64>;
 5444 |   using ThrID   = Layout<_128>;
 5445 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5446 |   using ELayout = GMMA::ELayout_64x64;
 5447 |   using BLayout = GMMA::ABLayout<256, 64>;
 5448 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5450-5451

```text
 5450 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5451 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5453-5462

```text
 5453 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5455 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5456 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel>>
 5457 | {
 5458 |   using ValTypeD = float;
 5459 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5460 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5461 |   using ValTypeB = float_e4m3_t;
 5462 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5464-5471

```text
 5464 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5466 |   using Shape_MNK = Shape<_64,_256,_64>;
 5467 |   using ThrID   = Layout<_128>;
 5468 |   using ALayout = GMMA::ALayout_64x64;
 5469 |   using ELayout = GMMA::ELayout_64x64;
 5470 |   using BLayout = GMMA::ABLayout<256, 64>;
 5471 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5473-5474

```text
 5473 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5474 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5476-5485

```text
 5476 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5478 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5479 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5480 | {
 5481 |   using ValTypeD = half_t;
 5482 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5483 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5484 |   using ValTypeB = float_e5m2_t;
 5485 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5487-5495

```text
 5487 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5488 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5490 |   using Shape_MNK = Shape<_64,_8,_64>;
 5491 |   using ThrID   = Layout<_128>;
 5492 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5493 |   using ELayout = GMMA::ELayout_64x64;
 5494 |   using BLayout = GMMA::ABLayout<  8, 64>;
 5495 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5497-5498

```text
 5497 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5498 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5500-5509

```text
 5500 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5502 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5503 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5504 | {
 5505 |   using ValTypeD = half_t;
 5506 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5507 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5508 |   using ValTypeB = float_e5m2_t;
 5509 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5511-5518

```text
 5511 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5513 |   using Shape_MNK = Shape<_64,_8,_64>;
 5514 |   using ThrID   = Layout<_128>;
 5515 |   using ALayout = GMMA::ALayout_64x64;
 5516 |   using ELayout = GMMA::ELayout_64x64;
 5517 |   using BLayout = GMMA::ABLayout<  8, 64>;
 5518 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5520-5521

```text
 5520 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5521 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5523-5532

```text
 5523 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5525 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5526 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5527 | {
 5528 |   using ValTypeD = float;
 5529 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5530 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5531 |   using ValTypeB = float_e5m2_t;
 5532 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5534-5542

```text
 5534 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5535 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5537 |   using Shape_MNK = Shape<_64,_8,_64>;
 5538 |   using ThrID   = Layout<_128>;
 5539 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5540 |   using ELayout = GMMA::ELayout_64x64;
 5541 |   using BLayout = GMMA::ABLayout<  8, 64>;
 5542 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5544-5545

```text
 5544 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5545 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5547-5556

```text
 5547 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5549 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5550 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5551 | {
 5552 |   using ValTypeD = float;
 5553 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5554 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5555 |   using ValTypeB = float_e5m2_t;
 5556 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5558-5565

```text
 5558 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5560 |   using Shape_MNK = Shape<_64,_8,_64>;
 5561 |   using ThrID   = Layout<_128>;
 5562 |   using ALayout = GMMA::ALayout_64x64;
 5563 |   using ELayout = GMMA::ELayout_64x64;
 5564 |   using BLayout = GMMA::ABLayout<  8, 64>;
 5565 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5567-5568

```text
 5567 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5568 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5570-5579

```text
 5570 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5572 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5573 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5574 | {
 5575 |   using ValTypeD = half_t;
 5576 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5577 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5578 |   using ValTypeB = float_e5m2_t;
 5579 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5581-5589

```text
 5581 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5582 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5584 |   using Shape_MNK = Shape<_64,_16,_64>;
 5585 |   using ThrID   = Layout<_128>;
 5586 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5587 |   using ELayout = GMMA::ELayout_64x64;
 5588 |   using BLayout = GMMA::ABLayout< 16, 64>;
 5589 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5591-5592

```text
 5591 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5592 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5594-5603

```text
 5594 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5596 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5597 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5598 | {
 5599 |   using ValTypeD = half_t;
 5600 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5601 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5602 |   using ValTypeB = float_e5m2_t;
 5603 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5605-5612

```text
 5605 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5607 |   using Shape_MNK = Shape<_64,_16,_64>;
 5608 |   using ThrID   = Layout<_128>;
 5609 |   using ALayout = GMMA::ALayout_64x64;
 5610 |   using ELayout = GMMA::ELayout_64x64;
 5611 |   using BLayout = GMMA::ABLayout< 16, 64>;
 5612 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5614-5615

```text
 5614 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5615 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5617-5626

```text
 5617 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5619 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5620 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5621 | {
 5622 |   using ValTypeD = float;
 5623 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5624 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5625 |   using ValTypeB = float_e5m2_t;
 5626 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5628-5636

```text
 5628 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5629 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5631 |   using Shape_MNK = Shape<_64,_16,_64>;
 5632 |   using ThrID   = Layout<_128>;
 5633 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5634 |   using ELayout = GMMA::ELayout_64x64;
 5635 |   using BLayout = GMMA::ABLayout< 16, 64>;
 5636 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5638-5639

```text
 5638 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5639 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5641-5650

```text
 5641 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5643 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5644 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5645 | {
 5646 |   using ValTypeD = float;
 5647 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5648 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5649 |   using ValTypeB = float_e5m2_t;
 5650 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5652-5659

```text
 5652 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5654 |   using Shape_MNK = Shape<_64,_16,_64>;
 5655 |   using ThrID   = Layout<_128>;
 5656 |   using ALayout = GMMA::ALayout_64x64;
 5657 |   using ELayout = GMMA::ELayout_64x64;
 5658 |   using BLayout = GMMA::ABLayout< 16, 64>;
 5659 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5661-5662

```text
 5661 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5662 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5664-5673

```text
 5664 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5666 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5667 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5668 | {
 5669 |   using ValTypeD = half_t;
 5670 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5671 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5672 |   using ValTypeB = float_e5m2_t;
 5673 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5675-5683

```text
 5675 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5676 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5678 |   using Shape_MNK = Shape<_64,_32,_64>;
 5679 |   using ThrID   = Layout<_128>;
 5680 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5681 |   using ELayout = GMMA::ELayout_64x64;
 5682 |   using BLayout = GMMA::ABLayout< 32, 64>;
 5683 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5685-5686

```text
 5685 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5686 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5688-5697

```text
 5688 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5690 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5691 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5692 | {
 5693 |   using ValTypeD = half_t;
 5694 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5695 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5696 |   using ValTypeB = float_e5m2_t;
 5697 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5699-5706

```text
 5699 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5701 |   using Shape_MNK = Shape<_64,_32,_64>;
 5702 |   using ThrID   = Layout<_128>;
 5703 |   using ALayout = GMMA::ALayout_64x64;
 5704 |   using ELayout = GMMA::ELayout_64x64;
 5705 |   using BLayout = GMMA::ABLayout< 32, 64>;
 5706 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5708-5709

```text
 5708 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5709 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5711-5720

```text
 5711 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5713 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5714 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5715 | {
 5716 |   using ValTypeD = float;
 5717 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5718 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5719 |   using ValTypeB = float_e5m2_t;
 5720 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5722-5730

```text
 5722 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5723 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5725 |   using Shape_MNK = Shape<_64,_32,_64>;
 5726 |   using ThrID   = Layout<_128>;
 5727 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5728 |   using ELayout = GMMA::ELayout_64x64;
 5729 |   using BLayout = GMMA::ABLayout< 32, 64>;
 5730 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5732-5733

```text
 5732 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5733 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5735-5744

```text
 5735 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5737 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5738 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5739 | {
 5740 |   using ValTypeD = float;
 5741 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5742 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5743 |   using ValTypeB = float_e5m2_t;
 5744 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5746-5753

```text
 5746 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5748 |   using Shape_MNK = Shape<_64,_32,_64>;
 5749 |   using ThrID   = Layout<_128>;
 5750 |   using ALayout = GMMA::ALayout_64x64;
 5751 |   using ELayout = GMMA::ELayout_64x64;
 5752 |   using BLayout = GMMA::ABLayout< 32, 64>;
 5753 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5755-5756

```text
 5755 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5756 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5758-5767

```text
 5758 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5760 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5761 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5762 | {
 5763 |   using ValTypeD = half_t;
 5764 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5765 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5766 |   using ValTypeB = float_e5m2_t;
 5767 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5769-5777

```text
 5769 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5770 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5772 |   using Shape_MNK = Shape<_64,_64,_64>;
 5773 |   using ThrID   = Layout<_128>;
 5774 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5775 |   using ELayout = GMMA::ELayout_64x64;
 5776 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5777 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5779-5780

```text
 5779 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5780 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5782-5791

```text
 5782 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5784 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5785 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5786 | {
 5787 |   using ValTypeD = half_t;
 5788 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5789 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5790 |   using ValTypeB = float_e5m2_t;
 5791 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5793-5800

```text
 5793 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5795 |   using Shape_MNK = Shape<_64,_64,_64>;
 5796 |   using ThrID   = Layout<_128>;
 5797 |   using ALayout = GMMA::ALayout_64x64;
 5798 |   using ELayout = GMMA::ELayout_64x64;
 5799 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5800 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5802-5803

```text
 5802 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5803 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5805-5814

```text
 5805 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5807 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5808 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5809 | {
 5810 |   using ValTypeD = float;
 5811 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5812 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5813 |   using ValTypeB = float_e5m2_t;
 5814 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5816-5824

```text
 5816 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5817 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5819 |   using Shape_MNK = Shape<_64,_64,_64>;
 5820 |   using ThrID   = Layout<_128>;
 5821 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5822 |   using ELayout = GMMA::ELayout_64x64;
 5823 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5824 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5826-5827

```text
 5826 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5827 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5829-5838

```text
 5829 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5831 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5832 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5833 | {
 5834 |   using ValTypeD = float;
 5835 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5836 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5837 |   using ValTypeB = float_e5m2_t;
 5838 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5840-5847

```text
 5840 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5842 |   using Shape_MNK = Shape<_64,_64,_64>;
 5843 |   using ThrID   = Layout<_128>;
 5844 |   using ALayout = GMMA::ALayout_64x64;
 5845 |   using ELayout = GMMA::ELayout_64x64;
 5846 |   using BLayout = GMMA::ABLayout< 64, 64>;
 5847 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5849-5850

```text
 5849 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5850 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5852-5861

```text
 5852 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5854 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5855 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5856 | {
 5857 |   using ValTypeD = half_t;
 5858 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5859 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5860 |   using ValTypeB = float_e5m2_t;
 5861 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5863-5871

```text
 5863 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5864 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5866 |   using Shape_MNK = Shape<_64,_96,_64>;
 5867 |   using ThrID   = Layout<_128>;
 5868 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5869 |   using ELayout = GMMA::ELayout_64x64;
 5870 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5871 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5873-5874

```text
 5873 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5874 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5876-5885

```text
 5876 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5878 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5879 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5880 | {
 5881 |   using ValTypeD = half_t;
 5882 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5883 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5884 |   using ValTypeB = float_e5m2_t;
 5885 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5887-5894

```text
 5887 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5889 |   using Shape_MNK = Shape<_64,_96,_64>;
 5890 |   using ThrID   = Layout<_128>;
 5891 |   using ALayout = GMMA::ALayout_64x64;
 5892 |   using ELayout = GMMA::ELayout_64x64;
 5893 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5894 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5896-5897

```text
 5896 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5897 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5899-5908

```text
 5899 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5901 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5902 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5903 | {
 5904 |   using ValTypeD = float;
 5905 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5906 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5907 |   using ValTypeB = float_e5m2_t;
 5908 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5910-5918

```text
 5910 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5911 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5913 |   using Shape_MNK = Shape<_64,_96,_64>;
 5914 |   using ThrID   = Layout<_128>;
 5915 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5916 |   using ELayout = GMMA::ELayout_64x64;
 5917 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5918 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5920-5921

```text
 5920 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5921 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5923-5932

```text
 5923 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5925 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5926 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5927 | {
 5928 |   using ValTypeD = float;
 5929 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5930 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5931 |   using ValTypeB = float_e5m2_t;
 5932 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5934-5941

```text
 5934 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5936 |   using Shape_MNK = Shape<_64,_96,_64>;
 5937 |   using ThrID   = Layout<_128>;
 5938 |   using ALayout = GMMA::ALayout_64x64;
 5939 |   using ELayout = GMMA::ELayout_64x64;
 5940 |   using BLayout = GMMA::ABLayout< 96, 64>;
 5941 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5943-5944

```text
 5943 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5944 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5946-5955

```text
 5946 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5948 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5949 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5950 | {
 5951 |   using ValTypeD = half_t;
 5952 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5953 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5954 |   using ValTypeB = float_e5m2_t;
 5955 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5957-5965

```text
 5957 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5958 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5960 |   using Shape_MNK = Shape<_64,_128,_64>;
 5961 |   using ThrID   = Layout<_128>;
 5962 |   using ALayout = GMMA::ABLayout< 64, 64>;
 5963 |   using ELayout = GMMA::ELayout_64x64;
 5964 |   using BLayout = GMMA::ABLayout<128, 64>;
 5965 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5967-5968

```text
 5967 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5968 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5970-5979

```text
 5970 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5972 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5973 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 5974 | {
 5975 |   using ValTypeD = half_t;
 5976 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 5977 |   using ValTypeE = sparse_elem<8, uint8_t>;
 5978 |   using ValTypeB = float_e5m2_t;
 5979 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5981-5988

```text
 5981 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5983 |   using Shape_MNK = Shape<_64,_128,_64>;
 5984 |   using ThrID   = Layout<_128>;
 5985 |   using ALayout = GMMA::ALayout_64x64;
 5986 |   using ELayout = GMMA::ELayout_64x64;
 5987 |   using BLayout = GMMA::ABLayout<128, 64>;
 5988 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5990-5991

```text
 5990 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5991 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5993-6002

```text
 5993 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5995 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 5996 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 5997 | {
 5998 |   using ValTypeD = float;
 5999 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6000 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6001 |   using ValTypeB = float_e5m2_t;
 6002 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6004-6012

```text
 6004 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6005 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6007 |   using Shape_MNK = Shape<_64,_128,_64>;
 6008 |   using ThrID   = Layout<_128>;
 6009 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6010 |   using ELayout = GMMA::ELayout_64x64;
 6011 |   using BLayout = GMMA::ABLayout<128, 64>;
 6012 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6014-6015

```text
 6014 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6015 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6017-6026

```text
 6017 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6019 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6020 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 6021 | {
 6022 |   using ValTypeD = float;
 6023 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6024 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6025 |   using ValTypeB = float_e5m2_t;
 6026 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6028-6035

```text
 6028 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6030 |   using Shape_MNK = Shape<_64,_128,_64>;
 6031 |   using ThrID   = Layout<_128>;
 6032 |   using ALayout = GMMA::ALayout_64x64;
 6033 |   using ELayout = GMMA::ELayout_64x64;
 6034 |   using BLayout = GMMA::ABLayout<128, 64>;
 6035 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6037-6038

```text
 6037 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6038 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6040-6049

```text
 6040 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6042 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6043 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 6044 | {
 6045 |   using ValTypeD = half_t;
 6046 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6047 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6048 |   using ValTypeB = float_e5m2_t;
 6049 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6051-6059

```text
 6051 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6052 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6054 |   using Shape_MNK = Shape<_64,_192,_64>;
 6055 |   using ThrID   = Layout<_128>;
 6056 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6057 |   using ELayout = GMMA::ELayout_64x64;
 6058 |   using BLayout = GMMA::ABLayout<192, 64>;
 6059 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6061-6062

```text
 6061 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6062 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6064-6073

```text
 6064 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6066 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6067 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 6068 | {
 6069 |   using ValTypeD = half_t;
 6070 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6071 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6072 |   using ValTypeB = float_e5m2_t;
 6073 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6075-6082

```text
 6075 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6077 |   using Shape_MNK = Shape<_64,_192,_64>;
 6078 |   using ThrID   = Layout<_128>;
 6079 |   using ALayout = GMMA::ALayout_64x64;
 6080 |   using ELayout = GMMA::ELayout_64x64;
 6081 |   using BLayout = GMMA::ABLayout<192, 64>;
 6082 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6084-6085

```text
 6084 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6085 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6087-6096

```text
 6087 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6089 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6090 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 6091 | {
 6092 |   using ValTypeD = float;
 6093 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6094 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6095 |   using ValTypeB = float_e5m2_t;
 6096 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6098-6106

```text
 6098 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6099 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6101 |   using Shape_MNK = Shape<_64,_192,_64>;
 6102 |   using ThrID   = Layout<_128>;
 6103 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6104 |   using ELayout = GMMA::ELayout_64x64;
 6105 |   using BLayout = GMMA::ABLayout<192, 64>;
 6106 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6108-6109

```text
 6108 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6109 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6111-6120

```text
 6111 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6113 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6114 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 6115 | {
 6116 |   using ValTypeD = float;
 6117 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6118 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6119 |   using ValTypeB = float_e5m2_t;
 6120 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6122-6129

```text
 6122 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6124 |   using Shape_MNK = Shape<_64,_192,_64>;
 6125 |   using ThrID   = Layout<_128>;
 6126 |   using ALayout = GMMA::ALayout_64x64;
 6127 |   using ELayout = GMMA::ELayout_64x64;
 6128 |   using BLayout = GMMA::ABLayout<192, 64>;
 6129 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6131-6132

```text
 6131 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6132 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6134-6143

```text
 6134 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6136 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6137 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 6138 | {
 6139 |   using ValTypeD = half_t;
 6140 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6141 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6142 |   using ValTypeB = float_e5m2_t;
 6143 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6145-6153

```text
 6145 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6146 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6148 |   using Shape_MNK = Shape<_64,_256,_64>;
 6149 |   using ThrID   = Layout<_128>;
 6150 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6151 |   using ELayout = GMMA::ELayout_64x64;
 6152 |   using BLayout = GMMA::ABLayout<256, 64>;
 6153 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6155-6156

```text
 6155 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6156 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6158-6167

```text
 6158 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6160 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6161 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 6162 | {
 6163 |   using ValTypeD = half_t;
 6164 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6165 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6166 |   using ValTypeB = float_e5m2_t;
 6167 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6169-6176

```text
 6169 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6171 |   using Shape_MNK = Shape<_64,_256,_64>;
 6172 |   using ThrID   = Layout<_128>;
 6173 |   using ALayout = GMMA::ALayout_64x64;
 6174 |   using ELayout = GMMA::ELayout_64x64;
 6175 |   using BLayout = GMMA::ABLayout<256, 64>;
 6176 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6178-6179

```text
 6178 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6179 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6181-6190

```text
 6181 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6183 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6184 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel>>
 6185 | {
 6186 |   using ValTypeD = float;
 6187 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6188 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6189 |   using ValTypeB = float_e5m2_t;
 6190 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6192-6200

```text
 6192 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6193 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6195 |   using Shape_MNK = Shape<_64,_256,_64>;
 6196 |   using ThrID   = Layout<_128>;
 6197 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6198 |   using ELayout = GMMA::ELayout_64x64;
 6199 |   using BLayout = GMMA::ABLayout<256, 64>;
 6200 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6202-6203

```text
 6202 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6203 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6205-6214

```text
 6205 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6207 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6208 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel>>
 6209 | {
 6210 |   using ValTypeD = float;
 6211 |   using ValTypeA = sparse_elem<2, float_e4m3_t>;
 6212 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6213 |   using ValTypeB = float_e5m2_t;
 6214 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E4M3E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6216-6223

```text
 6216 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6218 |   using Shape_MNK = Shape<_64,_256,_64>;
 6219 |   using ThrID   = Layout<_128>;
 6220 |   using ALayout = GMMA::ALayout_64x64;
 6221 |   using ELayout = GMMA::ELayout_64x64;
 6222 |   using BLayout = GMMA::ABLayout<256, 64>;
 6223 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6225-6226

```text
 6225 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6226 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6228-6237

```text
 6228 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6230 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6231 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6232 | {
 6233 |   using ValTypeD = half_t;
 6234 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6235 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6236 |   using ValTypeB = float_e4m3_t;
 6237 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6239-6247

```text
 6239 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6240 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6242 |   using Shape_MNK = Shape<_64,_8,_64>;
 6243 |   using ThrID   = Layout<_128>;
 6244 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6245 |   using ELayout = GMMA::ELayout_64x64;
 6246 |   using BLayout = GMMA::ABLayout<  8, 64>;
 6247 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6249-6250

```text
 6249 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6250 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6252-6261

```text
 6252 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6254 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6255 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6256 | {
 6257 |   using ValTypeD = half_t;
 6258 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6259 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6260 |   using ValTypeB = float_e4m3_t;
 6261 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6263-6270

```text
 6263 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6265 |   using Shape_MNK = Shape<_64,_8,_64>;
 6266 |   using ThrID   = Layout<_128>;
 6267 |   using ALayout = GMMA::ALayout_64x64;
 6268 |   using ELayout = GMMA::ELayout_64x64;
 6269 |   using BLayout = GMMA::ABLayout<  8, 64>;
 6270 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6272-6273

```text
 6272 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6273 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6275-6284

```text
 6275 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6277 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6278 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6279 | {
 6280 |   using ValTypeD = float;
 6281 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6282 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6283 |   using ValTypeB = float_e4m3_t;
 6284 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6286-6294

```text
 6286 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6287 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6289 |   using Shape_MNK = Shape<_64,_8,_64>;
 6290 |   using ThrID   = Layout<_128>;
 6291 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6292 |   using ELayout = GMMA::ELayout_64x64;
 6293 |   using BLayout = GMMA::ABLayout<  8, 64>;
 6294 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6296-6297

```text
 6296 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6297 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6299-6308

```text
 6299 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6301 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6302 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6303 | {
 6304 |   using ValTypeD = float;
 6305 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6306 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6307 |   using ValTypeB = float_e4m3_t;
 6308 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6310-6317

```text
 6310 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6312 |   using Shape_MNK = Shape<_64,_8,_64>;
 6313 |   using ThrID   = Layout<_128>;
 6314 |   using ALayout = GMMA::ALayout_64x64;
 6315 |   using ELayout = GMMA::ELayout_64x64;
 6316 |   using BLayout = GMMA::ABLayout<  8, 64>;
 6317 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6319-6320

```text
 6319 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6320 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6322-6331

```text
 6322 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6324 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6325 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6326 | {
 6327 |   using ValTypeD = half_t;
 6328 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6329 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6330 |   using ValTypeB = float_e4m3_t;
 6331 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6333-6341

```text
 6333 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6334 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6336 |   using Shape_MNK = Shape<_64,_16,_64>;
 6337 |   using ThrID   = Layout<_128>;
 6338 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6339 |   using ELayout = GMMA::ELayout_64x64;
 6340 |   using BLayout = GMMA::ABLayout< 16, 64>;
 6341 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6343-6344

```text
 6343 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6344 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6346-6355

```text
 6346 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6348 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6349 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6350 | {
 6351 |   using ValTypeD = half_t;
 6352 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6353 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6354 |   using ValTypeB = float_e4m3_t;
 6355 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6357-6364

```text
 6357 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6359 |   using Shape_MNK = Shape<_64,_16,_64>;
 6360 |   using ThrID   = Layout<_128>;
 6361 |   using ALayout = GMMA::ALayout_64x64;
 6362 |   using ELayout = GMMA::ELayout_64x64;
 6363 |   using BLayout = GMMA::ABLayout< 16, 64>;
 6364 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6366-6367

```text
 6366 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6367 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6369-6378

```text
 6369 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6371 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6372 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6373 | {
 6374 |   using ValTypeD = float;
 6375 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6376 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6377 |   using ValTypeB = float_e4m3_t;
 6378 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6380-6388

```text
 6380 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6381 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6383 |   using Shape_MNK = Shape<_64,_16,_64>;
 6384 |   using ThrID   = Layout<_128>;
 6385 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6386 |   using ELayout = GMMA::ELayout_64x64;
 6387 |   using BLayout = GMMA::ABLayout< 16, 64>;
 6388 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6390-6391

```text
 6390 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6391 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6393-6402

```text
 6393 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6395 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6396 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6397 | {
 6398 |   using ValTypeD = float;
 6399 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6400 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6401 |   using ValTypeB = float_e4m3_t;
 6402 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6404-6411

```text
 6404 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6406 |   using Shape_MNK = Shape<_64,_16,_64>;
 6407 |   using ThrID   = Layout<_128>;
 6408 |   using ALayout = GMMA::ALayout_64x64;
 6409 |   using ELayout = GMMA::ELayout_64x64;
 6410 |   using BLayout = GMMA::ABLayout< 16, 64>;
 6411 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6413-6414

```text
 6413 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6414 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6416-6425

```text
 6416 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6418 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6419 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6420 | {
 6421 |   using ValTypeD = half_t;
 6422 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6423 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6424 |   using ValTypeB = float_e4m3_t;
 6425 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6427-6435

```text
 6427 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6428 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6430 |   using Shape_MNK = Shape<_64,_32,_64>;
 6431 |   using ThrID   = Layout<_128>;
 6432 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6433 |   using ELayout = GMMA::ELayout_64x64;
 6434 |   using BLayout = GMMA::ABLayout< 32, 64>;
 6435 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6437-6438

```text
 6437 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6438 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6440-6449

```text
 6440 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6442 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6443 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6444 | {
 6445 |   using ValTypeD = half_t;
 6446 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6447 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6448 |   using ValTypeB = float_e4m3_t;
 6449 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6451-6458

```text
 6451 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6453 |   using Shape_MNK = Shape<_64,_32,_64>;
 6454 |   using ThrID   = Layout<_128>;
 6455 |   using ALayout = GMMA::ALayout_64x64;
 6456 |   using ELayout = GMMA::ELayout_64x64;
 6457 |   using BLayout = GMMA::ABLayout< 32, 64>;
 6458 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6460-6461

```text
 6460 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6461 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6463-6472

```text
 6463 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6465 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6466 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6467 | {
 6468 |   using ValTypeD = float;
 6469 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6470 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6471 |   using ValTypeB = float_e4m3_t;
 6472 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6474-6482

```text
 6474 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6475 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6477 |   using Shape_MNK = Shape<_64,_32,_64>;
 6478 |   using ThrID   = Layout<_128>;
 6479 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6480 |   using ELayout = GMMA::ELayout_64x64;
 6481 |   using BLayout = GMMA::ABLayout< 32, 64>;
 6482 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6484-6485

```text
 6484 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6485 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6487-6496

```text
 6487 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6489 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6490 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6491 | {
 6492 |   using ValTypeD = float;
 6493 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6494 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6495 |   using ValTypeB = float_e4m3_t;
 6496 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6498-6505

```text
 6498 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6500 |   using Shape_MNK = Shape<_64,_32,_64>;
 6501 |   using ThrID   = Layout<_128>;
 6502 |   using ALayout = GMMA::ALayout_64x64;
 6503 |   using ELayout = GMMA::ELayout_64x64;
 6504 |   using BLayout = GMMA::ABLayout< 32, 64>;
 6505 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6507-6508

```text
 6507 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6508 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6510-6519

```text
 6510 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6512 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6513 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6514 | {
 6515 |   using ValTypeD = half_t;
 6516 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6517 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6518 |   using ValTypeB = float_e4m3_t;
 6519 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6521-6529

```text
 6521 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6522 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6524 |   using Shape_MNK = Shape<_64,_64,_64>;
 6525 |   using ThrID   = Layout<_128>;
 6526 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6527 |   using ELayout = GMMA::ELayout_64x64;
 6528 |   using BLayout = GMMA::ABLayout< 64, 64>;
 6529 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6531-6532

```text
 6531 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6532 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6534-6543

```text
 6534 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6536 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6537 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6538 | {
 6539 |   using ValTypeD = half_t;
 6540 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6541 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6542 |   using ValTypeB = float_e4m3_t;
 6543 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6545-6552

```text
 6545 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6547 |   using Shape_MNK = Shape<_64,_64,_64>;
 6548 |   using ThrID   = Layout<_128>;
 6549 |   using ALayout = GMMA::ALayout_64x64;
 6550 |   using ELayout = GMMA::ELayout_64x64;
 6551 |   using BLayout = GMMA::ABLayout< 64, 64>;
 6552 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6554-6555

```text
 6554 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6555 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6557-6566

```text
 6557 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6559 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6560 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6561 | {
 6562 |   using ValTypeD = float;
 6563 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6564 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6565 |   using ValTypeB = float_e4m3_t;
 6566 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6568-6576

```text
 6568 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6569 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6571 |   using Shape_MNK = Shape<_64,_64,_64>;
 6572 |   using ThrID   = Layout<_128>;
 6573 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6574 |   using ELayout = GMMA::ELayout_64x64;
 6575 |   using BLayout = GMMA::ABLayout< 64, 64>;
 6576 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6578-6579

```text
 6578 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6579 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6581-6590

```text
 6581 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6583 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6584 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6585 | {
 6586 |   using ValTypeD = float;
 6587 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6588 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6589 |   using ValTypeB = float_e4m3_t;
 6590 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6592-6599

```text
 6592 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6594 |   using Shape_MNK = Shape<_64,_64,_64>;
 6595 |   using ThrID   = Layout<_128>;
 6596 |   using ALayout = GMMA::ALayout_64x64;
 6597 |   using ELayout = GMMA::ELayout_64x64;
 6598 |   using BLayout = GMMA::ABLayout< 64, 64>;
 6599 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6601-6602

```text
 6601 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6602 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6604-6613

```text
 6604 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6606 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6607 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6608 | {
 6609 |   using ValTypeD = half_t;
 6610 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6611 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6612 |   using ValTypeB = float_e4m3_t;
 6613 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6615-6623

```text
 6615 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6616 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6618 |   using Shape_MNK = Shape<_64,_96,_64>;
 6619 |   using ThrID   = Layout<_128>;
 6620 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6621 |   using ELayout = GMMA::ELayout_64x64;
 6622 |   using BLayout = GMMA::ABLayout< 96, 64>;
 6623 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6625-6626

```text
 6625 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6626 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6628-6637

```text
 6628 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6630 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6631 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6632 | {
 6633 |   using ValTypeD = half_t;
 6634 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6635 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6636 |   using ValTypeB = float_e4m3_t;
 6637 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6639-6646

```text
 6639 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6641 |   using Shape_MNK = Shape<_64,_96,_64>;
 6642 |   using ThrID   = Layout<_128>;
 6643 |   using ALayout = GMMA::ALayout_64x64;
 6644 |   using ELayout = GMMA::ELayout_64x64;
 6645 |   using BLayout = GMMA::ABLayout< 96, 64>;
 6646 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6648-6649

```text
 6648 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6649 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6651-6660

```text
 6651 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6653 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6654 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6655 | {
 6656 |   using ValTypeD = float;
 6657 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6658 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6659 |   using ValTypeB = float_e4m3_t;
 6660 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6662-6670

```text
 6662 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6663 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6665 |   using Shape_MNK = Shape<_64,_96,_64>;
 6666 |   using ThrID   = Layout<_128>;
 6667 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6668 |   using ELayout = GMMA::ELayout_64x64;
 6669 |   using BLayout = GMMA::ABLayout< 96, 64>;
 6670 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6672-6673

```text
 6672 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6673 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6675-6684

```text
 6675 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6677 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6678 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6679 | {
 6680 |   using ValTypeD = float;
 6681 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6682 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6683 |   using ValTypeB = float_e4m3_t;
 6684 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6686-6693

```text
 6686 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6688 |   using Shape_MNK = Shape<_64,_96,_64>;
 6689 |   using ThrID   = Layout<_128>;
 6690 |   using ALayout = GMMA::ALayout_64x64;
 6691 |   using ELayout = GMMA::ELayout_64x64;
 6692 |   using BLayout = GMMA::ABLayout< 96, 64>;
 6693 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6695-6696

```text
 6695 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6696 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6698-6707

```text
 6698 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6700 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6701 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6702 | {
 6703 |   using ValTypeD = half_t;
 6704 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6705 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6706 |   using ValTypeB = float_e4m3_t;
 6707 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6709-6717

```text
 6709 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6710 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6712 |   using Shape_MNK = Shape<_64,_128,_64>;
 6713 |   using ThrID   = Layout<_128>;
 6714 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6715 |   using ELayout = GMMA::ELayout_64x64;
 6716 |   using BLayout = GMMA::ABLayout<128, 64>;
 6717 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6719-6720

```text
 6719 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6720 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6722-6731

```text
 6722 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6724 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6725 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6726 | {
 6727 |   using ValTypeD = half_t;
 6728 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6729 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6730 |   using ValTypeB = float_e4m3_t;
 6731 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6733-6740

```text
 6733 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6735 |   using Shape_MNK = Shape<_64,_128,_64>;
 6736 |   using ThrID   = Layout<_128>;
 6737 |   using ALayout = GMMA::ALayout_64x64;
 6738 |   using ELayout = GMMA::ELayout_64x64;
 6739 |   using BLayout = GMMA::ABLayout<128, 64>;
 6740 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6742-6743

```text
 6742 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6743 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6745-6754

```text
 6745 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6747 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6748 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6749 | {
 6750 |   using ValTypeD = float;
 6751 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6752 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6753 |   using ValTypeB = float_e4m3_t;
 6754 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6756-6764

```text
 6756 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6757 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6759 |   using Shape_MNK = Shape<_64,_128,_64>;
 6760 |   using ThrID   = Layout<_128>;
 6761 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6762 |   using ELayout = GMMA::ELayout_64x64;
 6763 |   using BLayout = GMMA::ABLayout<128, 64>;
 6764 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6766-6767

```text
 6766 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6767 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6769-6778

```text
 6769 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6771 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6772 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6773 | {
 6774 |   using ValTypeD = float;
 6775 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6776 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6777 |   using ValTypeB = float_e4m3_t;
 6778 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6780-6787

```text
 6780 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6782 |   using Shape_MNK = Shape<_64,_128,_64>;
 6783 |   using ThrID   = Layout<_128>;
 6784 |   using ALayout = GMMA::ALayout_64x64;
 6785 |   using ELayout = GMMA::ELayout_64x64;
 6786 |   using BLayout = GMMA::ABLayout<128, 64>;
 6787 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6789-6790

```text
 6789 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6790 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6792-6801

```text
 6792 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6794 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6795 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6796 | {
 6797 |   using ValTypeD = half_t;
 6798 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6799 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6800 |   using ValTypeB = float_e4m3_t;
 6801 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6803-6811

```text
 6803 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6804 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6806 |   using Shape_MNK = Shape<_64,_192,_64>;
 6807 |   using ThrID   = Layout<_128>;
 6808 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6809 |   using ELayout = GMMA::ELayout_64x64;
 6810 |   using BLayout = GMMA::ABLayout<192, 64>;
 6811 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6813-6814

```text
 6813 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6814 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6816-6825

```text
 6816 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6818 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6819 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6820 | {
 6821 |   using ValTypeD = half_t;
 6822 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6823 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6824 |   using ValTypeB = float_e4m3_t;
 6825 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6827-6834

```text
 6827 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6829 |   using Shape_MNK = Shape<_64,_192,_64>;
 6830 |   using ThrID   = Layout<_128>;
 6831 |   using ALayout = GMMA::ALayout_64x64;
 6832 |   using ELayout = GMMA::ELayout_64x64;
 6833 |   using BLayout = GMMA::ABLayout<192, 64>;
 6834 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6836-6837

```text
 6836 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6837 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6839-6848

```text
 6839 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6841 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6842 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6843 | {
 6844 |   using ValTypeD = float;
 6845 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6846 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6847 |   using ValTypeB = float_e4m3_t;
 6848 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6850-6858

```text
 6850 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6851 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6853 |   using Shape_MNK = Shape<_64,_192,_64>;
 6854 |   using ThrID   = Layout<_128>;
 6855 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6856 |   using ELayout = GMMA::ELayout_64x64;
 6857 |   using BLayout = GMMA::ABLayout<192, 64>;
 6858 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6860-6861

```text
 6860 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6861 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6863-6872

```text
 6863 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6865 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6866 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6867 | {
 6868 |   using ValTypeD = float;
 6869 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6870 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6871 |   using ValTypeB = float_e4m3_t;
 6872 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6874-6881

```text
 6874 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6876 |   using Shape_MNK = Shape<_64,_192,_64>;
 6877 |   using ThrID   = Layout<_128>;
 6878 |   using ALayout = GMMA::ALayout_64x64;
 6879 |   using ELayout = GMMA::ELayout_64x64;
 6880 |   using BLayout = GMMA::ABLayout<192, 64>;
 6881 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6883-6884

```text
 6883 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6884 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6886-6895

```text
 6886 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6888 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6889 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6890 | {
 6891 |   using ValTypeD = half_t;
 6892 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6893 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6894 |   using ValTypeB = float_e4m3_t;
 6895 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6897-6905

```text
 6897 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6898 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6900 |   using Shape_MNK = Shape<_64,_256,_64>;
 6901 |   using ThrID   = Layout<_128>;
 6902 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6903 |   using ELayout = GMMA::ELayout_64x64;
 6904 |   using BLayout = GMMA::ABLayout<256, 64>;
 6905 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6907-6908

```text
 6907 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6908 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6910-6919

```text
 6910 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6912 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6913 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6914 | {
 6915 |   using ValTypeD = half_t;
 6916 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6917 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6918 |   using ValTypeB = float_e4m3_t;
 6919 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6921-6928

```text
 6921 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6923 |   using Shape_MNK = Shape<_64,_256,_64>;
 6924 |   using ThrID   = Layout<_128>;
 6925 |   using ALayout = GMMA::ALayout_64x64;
 6926 |   using ELayout = GMMA::ELayout_64x64;
 6927 |   using BLayout = GMMA::ABLayout<256, 64>;
 6928 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6930-6931

```text
 6930 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6931 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6933-6942

```text
 6933 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6935 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6936 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel>>
 6937 | {
 6938 |   using ValTypeD = float;
 6939 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6940 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6941 |   using ValTypeB = float_e4m3_t;
 6942 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E4M3_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6944-6952

```text
 6944 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6945 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6947 |   using Shape_MNK = Shape<_64,_256,_64>;
 6948 |   using ThrID   = Layout<_128>;
 6949 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6950 |   using ELayout = GMMA::ELayout_64x64;
 6951 |   using BLayout = GMMA::ABLayout<256, 64>;
 6952 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6954-6955

```text
 6954 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6955 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6957-6966

```text
 6957 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6959 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6960 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel>>
 6961 | {
 6962 |   using ValTypeD = float;
 6963 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6964 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6965 |   using ValTypeB = float_e4m3_t;
 6966 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E4M3_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6968-6975

```text
 6968 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6970 |   using Shape_MNK = Shape<_64,_256,_64>;
 6971 |   using ThrID   = Layout<_128>;
 6972 |   using ALayout = GMMA::ALayout_64x64;
 6973 |   using ELayout = GMMA::ELayout_64x64;
 6974 |   using BLayout = GMMA::ABLayout<256, 64>;
 6975 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6977-6978

```text
 6977 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6978 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6980-6989

```text
 6980 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6982 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 6983 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 6984 | {
 6985 |   using ValTypeD = half_t;
 6986 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 6987 |   using ValTypeE = sparse_elem<8, uint8_t>;
 6988 |   using ValTypeB = float_e5m2_t;
 6989 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6991-6999

```text
 6991 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6992 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6994 |   using Shape_MNK = Shape<_64,_8,_64>;
 6995 |   using ThrID   = Layout<_128>;
 6996 |   using ALayout = GMMA::ABLayout< 64, 64>;
 6997 |   using ELayout = GMMA::ELayout_64x64;
 6998 |   using BLayout = GMMA::ABLayout<  8, 64>;
 6999 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7001-7002

```text
 7001 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7002 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7004-7013

```text
 7004 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7006 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7007 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7008 | {
 7009 |   using ValTypeD = half_t;
 7010 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7011 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7012 |   using ValTypeB = float_e5m2_t;
 7013 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7015-7022

```text
 7015 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7017 |   using Shape_MNK = Shape<_64,_8,_64>;
 7018 |   using ThrID   = Layout<_128>;
 7019 |   using ALayout = GMMA::ALayout_64x64;
 7020 |   using ELayout = GMMA::ELayout_64x64;
 7021 |   using BLayout = GMMA::ABLayout<  8, 64>;
 7022 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7024-7025

```text
 7024 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7025 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7027-7036

```text
 7027 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7029 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7030 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7031 | {
 7032 |   using ValTypeD = float;
 7033 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7034 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7035 |   using ValTypeB = float_e5m2_t;
 7036 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7038-7046

```text
 7038 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7039 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7041 |   using Shape_MNK = Shape<_64,_8,_64>;
 7042 |   using ThrID   = Layout<_128>;
 7043 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7044 |   using ELayout = GMMA::ELayout_64x64;
 7045 |   using BLayout = GMMA::ABLayout<  8, 64>;
 7046 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7048-7049

```text
 7048 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7049 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7051-7060

```text
 7051 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7053 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7054 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7055 | {
 7056 |   using ValTypeD = float;
 7057 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7058 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7059 |   using ValTypeB = float_e5m2_t;
 7060 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x8x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7062-7069

```text
 7062 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7064 |   using Shape_MNK = Shape<_64,_8,_64>;
 7065 |   using ThrID   = Layout<_128>;
 7066 |   using ALayout = GMMA::ALayout_64x64;
 7067 |   using ELayout = GMMA::ELayout_64x64;
 7068 |   using BLayout = GMMA::ABLayout<  8, 64>;
 7069 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7071-7072

```text
 7071 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7072 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7074-7083

```text
 7074 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7076 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7077 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7078 | {
 7079 |   using ValTypeD = half_t;
 7080 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7081 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7082 |   using ValTypeB = float_e5m2_t;
 7083 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7085-7093

```text
 7085 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7086 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7088 |   using Shape_MNK = Shape<_64,_16,_64>;
 7089 |   using ThrID   = Layout<_128>;
 7090 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7091 |   using ELayout = GMMA::ELayout_64x64;
 7092 |   using BLayout = GMMA::ABLayout< 16, 64>;
 7093 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7095-7096

```text
 7095 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7096 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7098-7107

```text
 7098 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7100 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7101 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7102 | {
 7103 |   using ValTypeD = half_t;
 7104 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7105 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7106 |   using ValTypeB = float_e5m2_t;
 7107 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7109-7116

```text
 7109 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7111 |   using Shape_MNK = Shape<_64,_16,_64>;
 7112 |   using ThrID   = Layout<_128>;
 7113 |   using ALayout = GMMA::ALayout_64x64;
 7114 |   using ELayout = GMMA::ELayout_64x64;
 7115 |   using BLayout = GMMA::ABLayout< 16, 64>;
 7116 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7118-7119

```text
 7118 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7119 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7121-7130

```text
 7121 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7123 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7124 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7125 | {
 7126 |   using ValTypeD = float;
 7127 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7128 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7129 |   using ValTypeB = float_e5m2_t;
 7130 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7132-7140

```text
 7132 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7133 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7135 |   using Shape_MNK = Shape<_64,_16,_64>;
 7136 |   using ThrID   = Layout<_128>;
 7137 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7138 |   using ELayout = GMMA::ELayout_64x64;
 7139 |   using BLayout = GMMA::ABLayout< 16, 64>;
 7140 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7142-7143

```text
 7142 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7143 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7145-7154

```text
 7145 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7147 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7148 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7149 | {
 7150 |   using ValTypeD = float;
 7151 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7152 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7153 |   using ValTypeB = float_e5m2_t;
 7154 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x16x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7156-7163

```text
 7156 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7158 |   using Shape_MNK = Shape<_64,_16,_64>;
 7159 |   using ThrID   = Layout<_128>;
 7160 |   using ALayout = GMMA::ALayout_64x64;
 7161 |   using ELayout = GMMA::ELayout_64x64;
 7162 |   using BLayout = GMMA::ABLayout< 16, 64>;
 7163 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7165-7166

```text
 7165 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7166 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7168-7177

```text
 7168 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7170 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7171 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7172 | {
 7173 |   using ValTypeD = half_t;
 7174 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7175 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7176 |   using ValTypeB = float_e5m2_t;
 7177 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7179-7187

```text
 7179 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7180 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7182 |   using Shape_MNK = Shape<_64,_32,_64>;
 7183 |   using ThrID   = Layout<_128>;
 7184 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7185 |   using ELayout = GMMA::ELayout_64x64;
 7186 |   using BLayout = GMMA::ABLayout< 32, 64>;
 7187 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7189-7190

```text
 7189 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7190 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7192-7201

```text
 7192 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7194 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7195 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7196 | {
 7197 |   using ValTypeD = half_t;
 7198 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7199 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7200 |   using ValTypeB = float_e5m2_t;
 7201 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7203-7210

```text
 7203 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7205 |   using Shape_MNK = Shape<_64,_32,_64>;
 7206 |   using ThrID   = Layout<_128>;
 7207 |   using ALayout = GMMA::ALayout_64x64;
 7208 |   using ELayout = GMMA::ELayout_64x64;
 7209 |   using BLayout = GMMA::ABLayout< 32, 64>;
 7210 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7212-7213

```text
 7212 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7213 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7215-7224

```text
 7215 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7217 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7218 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7219 | {
 7220 |   using ValTypeD = float;
 7221 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7222 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7223 |   using ValTypeB = float_e5m2_t;
 7224 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7226-7234

```text
 7226 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7227 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7229 |   using Shape_MNK = Shape<_64,_32,_64>;
 7230 |   using ThrID   = Layout<_128>;
 7231 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7232 |   using ELayout = GMMA::ELayout_64x64;
 7233 |   using BLayout = GMMA::ABLayout< 32, 64>;
 7234 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7236-7237

```text
 7236 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7237 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7239-7248

```text
 7239 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7241 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7242 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7243 | {
 7244 |   using ValTypeD = float;
 7245 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7246 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7247 |   using ValTypeB = float_e5m2_t;
 7248 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x32x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7250-7257

```text
 7250 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7252 |   using Shape_MNK = Shape<_64,_32,_64>;
 7253 |   using ThrID   = Layout<_128>;
 7254 |   using ALayout = GMMA::ALayout_64x64;
 7255 |   using ELayout = GMMA::ELayout_64x64;
 7256 |   using BLayout = GMMA::ABLayout< 32, 64>;
 7257 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7259-7260

```text
 7259 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7260 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7262-7271

```text
 7262 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7264 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7265 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7266 | {
 7267 |   using ValTypeD = half_t;
 7268 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7269 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7270 |   using ValTypeB = float_e5m2_t;
 7271 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7273-7281

```text
 7273 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7274 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7276 |   using Shape_MNK = Shape<_64,_64,_64>;
 7277 |   using ThrID   = Layout<_128>;
 7278 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7279 |   using ELayout = GMMA::ELayout_64x64;
 7280 |   using BLayout = GMMA::ABLayout< 64, 64>;
 7281 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7283-7284

```text
 7283 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7284 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7286-7295

```text
 7286 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7288 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7289 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7290 | {
 7291 |   using ValTypeD = half_t;
 7292 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7293 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7294 |   using ValTypeB = float_e5m2_t;
 7295 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7297-7304

```text
 7297 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7299 |   using Shape_MNK = Shape<_64,_64,_64>;
 7300 |   using ThrID   = Layout<_128>;
 7301 |   using ALayout = GMMA::ALayout_64x64;
 7302 |   using ELayout = GMMA::ELayout_64x64;
 7303 |   using BLayout = GMMA::ABLayout< 64, 64>;
 7304 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7306-7307

```text
 7306 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7307 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7309-7318

```text
 7309 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7311 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7312 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7313 | {
 7314 |   using ValTypeD = float;
 7315 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7316 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7317 |   using ValTypeB = float_e5m2_t;
 7318 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7320-7328

```text
 7320 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7321 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7323 |   using Shape_MNK = Shape<_64,_64,_64>;
 7324 |   using ThrID   = Layout<_128>;
 7325 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7326 |   using ELayout = GMMA::ELayout_64x64;
 7327 |   using BLayout = GMMA::ABLayout< 64, 64>;
 7328 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7330-7331

```text
 7330 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7331 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7333-7342

```text
 7333 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7335 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7336 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7337 | {
 7338 |   using ValTypeD = float;
 7339 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7340 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7341 |   using ValTypeB = float_e5m2_t;
 7342 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x64x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7344-7351

```text
 7344 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7346 |   using Shape_MNK = Shape<_64,_64,_64>;
 7347 |   using ThrID   = Layout<_128>;
 7348 |   using ALayout = GMMA::ALayout_64x64;
 7349 |   using ELayout = GMMA::ELayout_64x64;
 7350 |   using BLayout = GMMA::ABLayout< 64, 64>;
 7351 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7353-7354

```text
 7353 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7354 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7356-7365

```text
 7356 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7358 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7359 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7360 | {
 7361 |   using ValTypeD = half_t;
 7362 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7363 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7364 |   using ValTypeB = float_e5m2_t;
 7365 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7367-7375

```text
 7367 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7368 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7370 |   using Shape_MNK = Shape<_64,_96,_64>;
 7371 |   using ThrID   = Layout<_128>;
 7372 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7373 |   using ELayout = GMMA::ELayout_64x64;
 7374 |   using BLayout = GMMA::ABLayout< 96, 64>;
 7375 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7377-7378

```text
 7377 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7378 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7380-7389

```text
 7380 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7382 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7383 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7384 | {
 7385 |   using ValTypeD = half_t;
 7386 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7387 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7388 |   using ValTypeB = float_e5m2_t;
 7389 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7391-7398

```text
 7391 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7393 |   using Shape_MNK = Shape<_64,_96,_64>;
 7394 |   using ThrID   = Layout<_128>;
 7395 |   using ALayout = GMMA::ALayout_64x64;
 7396 |   using ELayout = GMMA::ELayout_64x64;
 7397 |   using BLayout = GMMA::ABLayout< 96, 64>;
 7398 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7400-7401

```text
 7400 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7401 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7403-7412

```text
 7403 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7405 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7406 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7407 | {
 7408 |   using ValTypeD = float;
 7409 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7410 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7411 |   using ValTypeB = float_e5m2_t;
 7412 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7414-7422

```text
 7414 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7415 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7417 |   using Shape_MNK = Shape<_64,_96,_64>;
 7418 |   using ThrID   = Layout<_128>;
 7419 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7420 |   using ELayout = GMMA::ELayout_64x64;
 7421 |   using BLayout = GMMA::ABLayout< 96, 64>;
 7422 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7424-7425

```text
 7424 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7425 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7427-7436

```text
 7427 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7429 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7430 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7431 | {
 7432 |   using ValTypeD = float;
 7433 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7434 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7435 |   using ValTypeB = float_e5m2_t;
 7436 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x96x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7438-7445

```text
 7438 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7440 |   using Shape_MNK = Shape<_64,_96,_64>;
 7441 |   using ThrID   = Layout<_128>;
 7442 |   using ALayout = GMMA::ALayout_64x64;
 7443 |   using ELayout = GMMA::ELayout_64x64;
 7444 |   using BLayout = GMMA::ABLayout< 96, 64>;
 7445 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7447-7448

```text
 7447 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7448 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7450-7459

```text
 7450 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7452 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7453 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7454 | {
 7455 |   using ValTypeD = half_t;
 7456 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7457 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7458 |   using ValTypeB = float_e5m2_t;
 7459 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7461-7469

```text
 7461 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7462 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7464 |   using Shape_MNK = Shape<_64,_128,_64>;
 7465 |   using ThrID   = Layout<_128>;
 7466 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7467 |   using ELayout = GMMA::ELayout_64x64;
 7468 |   using BLayout = GMMA::ABLayout<128, 64>;
 7469 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7471-7472

```text
 7471 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7472 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7474-7483

```text
 7474 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7476 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7477 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7478 | {
 7479 |   using ValTypeD = half_t;
 7480 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7481 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7482 |   using ValTypeB = float_e5m2_t;
 7483 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7485-7492

```text
 7485 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7487 |   using Shape_MNK = Shape<_64,_128,_64>;
 7488 |   using ThrID   = Layout<_128>;
 7489 |   using ALayout = GMMA::ALayout_64x64;
 7490 |   using ELayout = GMMA::ELayout_64x64;
 7491 |   using BLayout = GMMA::ABLayout<128, 64>;
 7492 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7494-7495

```text
 7494 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7495 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7497-7506

```text
 7497 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7499 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7500 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7501 | {
 7502 |   using ValTypeD = float;
 7503 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7504 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7505 |   using ValTypeB = float_e5m2_t;
 7506 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7508-7516

```text
 7508 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7509 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7511 |   using Shape_MNK = Shape<_64,_128,_64>;
 7512 |   using ThrID   = Layout<_128>;
 7513 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7514 |   using ELayout = GMMA::ELayout_64x64;
 7515 |   using BLayout = GMMA::ABLayout<128, 64>;
 7516 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7518-7519

```text
 7518 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7519 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7521-7530

```text
 7521 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7523 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7524 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7525 | {
 7526 |   using ValTypeD = float;
 7527 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7528 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7529 |   using ValTypeB = float_e5m2_t;
 7530 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x128x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7532-7539

```text
 7532 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7534 |   using Shape_MNK = Shape<_64,_128,_64>;
 7535 |   using ThrID   = Layout<_128>;
 7536 |   using ALayout = GMMA::ALayout_64x64;
 7537 |   using ELayout = GMMA::ELayout_64x64;
 7538 |   using BLayout = GMMA::ABLayout<128, 64>;
 7539 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7541-7542

```text
 7541 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7542 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7544-7553

```text
 7544 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7546 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7547 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7548 | {
 7549 |   using ValTypeD = half_t;
 7550 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7551 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7552 |   using ValTypeB = float_e5m2_t;
 7553 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7555-7563

```text
 7555 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7556 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7558 |   using Shape_MNK = Shape<_64,_192,_64>;
 7559 |   using ThrID   = Layout<_128>;
 7560 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7561 |   using ELayout = GMMA::ELayout_64x64;
 7562 |   using BLayout = GMMA::ABLayout<192, 64>;
 7563 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7565-7566

```text
 7565 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7566 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7568-7577

```text
 7568 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7570 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7571 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7572 | {
 7573 |   using ValTypeD = half_t;
 7574 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7575 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7576 |   using ValTypeB = float_e5m2_t;
 7577 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7579-7586

```text
 7579 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7581 |   using Shape_MNK = Shape<_64,_192,_64>;
 7582 |   using ThrID   = Layout<_128>;
 7583 |   using ALayout = GMMA::ALayout_64x64;
 7584 |   using ELayout = GMMA::ELayout_64x64;
 7585 |   using BLayout = GMMA::ABLayout<192, 64>;
 7586 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7588-7589

```text
 7588 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7589 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7591-7600

```text
 7591 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7593 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7594 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7595 | {
 7596 |   using ValTypeD = float;
 7597 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7598 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7599 |   using ValTypeB = float_e5m2_t;
 7600 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7602-7610

```text
 7602 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7603 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7605 |   using Shape_MNK = Shape<_64,_192,_64>;
 7606 |   using ThrID   = Layout<_128>;
 7607 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7608 |   using ELayout = GMMA::ELayout_64x64;
 7609 |   using BLayout = GMMA::ABLayout<192, 64>;
 7610 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7612-7613

```text
 7612 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7613 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7615-7624

```text
 7615 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7617 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7618 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7619 | {
 7620 |   using ValTypeD = float;
 7621 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7622 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7623 |   using ValTypeB = float_e5m2_t;
 7624 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x192x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7626-7633

```text
 7626 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7628 |   using Shape_MNK = Shape<_64,_192,_64>;
 7629 |   using ThrID   = Layout<_128>;
 7630 |   using ALayout = GMMA::ALayout_64x64;
 7631 |   using ELayout = GMMA::ELayout_64x64;
 7632 |   using BLayout = GMMA::ABLayout<192, 64>;
 7633 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7635-7636

```text
 7635 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7636 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7638-7647

```text
 7638 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7640 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7641 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7642 | {
 7643 |   using ValTypeD = half_t;
 7644 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7645 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7646 |   using ValTypeB = float_e5m2_t;
 7647 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7649-7657

```text
 7649 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7650 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7652 |   using Shape_MNK = Shape<_64,_256,_64>;
 7653 |   using ThrID   = Layout<_128>;
 7654 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7655 |   using ELayout = GMMA::ELayout_64x64;
 7656 |   using BLayout = GMMA::ABLayout<256, 64>;
 7657 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7659-7660

```text
 7659 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7660 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7662-7671

```text
 7662 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7664 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7665 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7666 | {
 7667 |   using ValTypeD = half_t;
 7668 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7669 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7670 |   using ValTypeB = float_e5m2_t;
 7671 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F16E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7673-7680

```text
 7673 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7675 |   using Shape_MNK = Shape<_64,_256,_64>;
 7676 |   using ThrID   = Layout<_128>;
 7677 |   using ALayout = GMMA::ALayout_64x64;
 7678 |   using ELayout = GMMA::ELayout_64x64;
 7679 |   using BLayout = GMMA::ABLayout<256, 64>;
 7680 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7682-7683

```text
 7682 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7683 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7685-7694

```text
 7685 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7687 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7688 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel>>
 7689 | {
 7690 |   using ValTypeD = float;
 7691 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7692 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7693 |   using ValTypeB = float_e5m2_t;
 7694 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E5M2_SS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7696-7704

```text
 7696 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7697 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7699 |   using Shape_MNK = Shape<_64,_256,_64>;
 7700 |   using ThrID   = Layout<_128>;
 7701 |   using ALayout = GMMA::ABLayout< 64, 64>;
 7702 |   using ELayout = GMMA::ELayout_64x64;
 7703 |   using BLayout = GMMA::ABLayout<256, 64>;
 7704 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7706-7707

```text
 7706 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7707 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7709-7718

```text
 7709 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7711 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB, GMMA::SparseSel spsel>
 7712 | struct MMA_Traits<SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel>>
 7713 | {
 7714 |   using ValTypeD = float;
 7715 |   using ValTypeA = sparse_elem<2, float_e5m2_t>;
 7716 |   using ValTypeE = sparse_elem<8, uint8_t>;
 7717 |   using ValTypeB = float_e5m2_t;
 7718 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90::GMMA::SPARSE::GMMA_64x256x64_F32E5M2E5M2_RS_TN<scaleA, scaleB, spsel` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7720-7727

```text
 7720 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7722 |   using Shape_MNK = Shape<_64,_256,_64>;
 7723 |   using ThrID   = Layout<_128>;
 7724 |   using ALayout = GMMA::ALayout_64x64;
 7725 |   using ELayout = GMMA::ELayout_64x64;
 7726 |   using BLayout = GMMA::ABLayout<256, 64>;
 7727 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7729-7730

```text
 7729 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7730 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7732-7734

```text
 7732 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7734 | } // end namespace cute
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7736-7738

```text
 7736 | #if defined(CUTE_SM90_EXTENDED_MMA_SHAPES_ENABLED)
 7737 | #include "mma_traits_sm90_gmma_sparse_ext.hpp"
 7738 | #endif
```
**EN:** Sets up the header dependencies for this file by importing `mma_traits_sm90_gmma_sparse_ext.hpp`.
**CN:** 通过引入 `mma_traits_sm90_gmma_sparse_ext.hpp` 为该文件建立头文件依赖。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/pointer_sparse.hpp`
  - `cute/swizzle.hpp`
  - `cute/tensor_impl.hpp`
  - `cute/arch/mma_sm90_desc.hpp`
  - `cute/arch/mma_sm90_gmma_sparse.hpp`
  - `cute/atom/mma_traits_sm90_gmma.hpp`
  - `cute/atom/mma_traits.hpp`
  - `cute/layout_composed.hpp`
  - `cute/numeric/integral_constant.hpp`
  - `mma_traits_sm90_gmma_sparse_ext.hpp`
- Primary symbols / 主要符号: `sparse_smem_desc`, `MakeTensor`, `MMA_Traits`, `Type`, `TEngine`, `TLayout`, `MMAOp`, `TD`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
