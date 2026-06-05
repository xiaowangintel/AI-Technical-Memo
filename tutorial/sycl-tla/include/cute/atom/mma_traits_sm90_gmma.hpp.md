# mma_traits_sm90_gmma.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/atom/mma_traits_sm90_gmma.hpp`
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

### Lines 33-41

```text
   33 | #include <cute/pointer_flagged.hpp>            // cute::smem_ptr_flag
   34 | #include <cute/pointer_sparse.hpp>             // cute::smem_sparse_ptr_flag
   35 | #include <cute/swizzle.hpp>                    // cute::Swizzle
   36 | #include <cute/tensor_impl.hpp>                // cute::Tensor
   37 | #include <cute/arch/mma_sm90_desc.hpp>         // cute::LayoutType
   38 | #include <cute/arch/mma_sm90_gmma.hpp>         // cute::SM90_64x8x16_F16F16F16_SS, etc
   39 | #include <cute/atom/mma_traits.hpp>            // cute::MMA_Traits
   40 | #include <cute/layout_composed.hpp>            // cute::ComposedLayout
   41 | #include <cute/numeric/integral_constant.hpp>  // cute::is_static
```
**EN:** Sets up the header dependencies for this file by importing `cute/pointer_flagged.hpp`, `cute/pointer_sparse.hpp`, `cute/swizzle.hpp`, `cute/tensor_impl.hpp`, `cute/arch/mma_sm90_desc.hpp`, and 4 more include(s).
**CN:** 通过引入 `cute/pointer_flagged.hpp`, `cute/pointer_sparse.hpp`, `cute/swizzle.hpp`, `cute/tensor_impl.hpp`, `cute/arch/mma_sm90_desc.hpp`，以及另外 4 个头文件 为该文件建立头文件依赖。

### Lines 43-43

```text
   43 | namespace cute {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 45-66

```text
   45 | // Fence between the async destination accumulators of GMMA & source for their dependent use
   46 | template <class Engine, class Layout>
   47 | CUTE_HOST_DEVICE
   48 | void
   49 | warpgroup_fence_operand(Tensor<Engine, Layout>& frg) {
   50 |   CUTE_STATIC_ASSERT(is_static<Layout>::value);
   51 |   if constexpr (is_same_v<typename Engine::value_type, float>) {
   52 |     auto f32_frg = recast<float>(frg);
   53 |     CUTE_UNROLL
   54 |     for (int i = 0; i < size(f32_frg); ++i) {
   55 |       warpgroup_fence_operand(f32_frg(i));
   56 |     }
   57 |   }
   58 |   else {
   59 |     CUTE_STATIC_ASSERT(is_rmem<Engine>::value);
   60 |     auto u32_frg = recast<uint32_t>(frg);
   61 |     CUTE_UNROLL
   62 |     for (int i = 0; i < size(u32_frg); ++i) {
   63 |       warpgroup_fence_operand(u32_frg(i));
   64 |     }
   65 |   }
   66 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 68-68

```text
   68 | namespace SM90::GMMA {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 70-84

```text
   70 | ///////////////////////////////////////////
   71 | // Common layouts for GMMA Shared Memory //
   72 | ///////////////////////////////////////////
   74 | // M|N-major GMMA layouts in units of bits
   75 | using Layout_MN_INTER_Atom_Bits = ComposedLayout<Swizzle<0,4,3>, smem_ptr_flag, Layout<Shape< _128,_8>,Stride<_1, _128>>>;
   76 | using Layout_MN_SW32_Atom_Bits  = ComposedLayout<Swizzle<1,4,3>, smem_ptr_flag, Layout<Shape< _256,_8>,Stride<_1, _256>>>;
   77 | using Layout_MN_SW64_Atom_Bits  = ComposedLayout<Swizzle<2,4,3>, smem_ptr_flag, Layout<Shape< _512,_8>,Stride<_1, _512>>>;
   78 | using Layout_MN_SW128_Atom_Bits = ComposedLayout<Swizzle<3,4,3>, smem_ptr_flag, Layout<Shape<_1024,_8>,Stride<_1,_1024>>>;
   80 | // K-major GMMA layouts in units of bits
   81 | using Layout_K_INTER_Atom_Bits  = ComposedLayout<Swizzle<0,4,3>, smem_ptr_flag, Layout<Shape<_8, _128>,Stride< _128,_1>>>;
   82 | using Layout_K_SW32_Atom_Bits   = ComposedLayout<Swizzle<1,4,3>, smem_ptr_flag, Layout<Shape<_8, _256>,Stride< _256,_1>>>;
   83 | using Layout_K_SW64_Atom_Bits   = ComposedLayout<Swizzle<2,4,3>, smem_ptr_flag, Layout<Shape<_8, _512>,Stride< _512,_1>>>;
   84 | using Layout_K_SW128_Atom_Bits  = ComposedLayout<Swizzle<3,4,3>, smem_ptr_flag, Layout<Shape<_8,_1024>,Stride<_1024,_1>>>;
```
**EN:** Defines shared-memory pointer wrappers and constructors, including overloads that compose with swizzle-aware addressing.
**CN:** 定义共享内存指针包装器与构造函数，并包含可与 swizzle 感知寻址组合的重载。

### Lines 86-94

```text
   86 | // M|N-major layouts in units of Type
   87 | template <class Type>
   88 | using Layout_MN_INTER_Atom = decltype(upcast<sizeof_bits<Type>::value>(Layout_MN_INTER_Atom_Bits{}));
   89 | template <class Type>
   90 | using Layout_MN_SW32_Atom  = decltype(upcast<sizeof_bits<Type>::value>(Layout_MN_SW32_Atom_Bits{}));
   91 | template <class Type>
   92 | using Layout_MN_SW64_Atom  = decltype(upcast<sizeof_bits<Type>::value>(Layout_MN_SW64_Atom_Bits{}));
   93 | template <class Type>
   94 | using Layout_MN_SW128_Atom = decltype(upcast<sizeof_bits<Type>::value>(Layout_MN_SW128_Atom_Bits{}));
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 96-104

```text
   96 | // K-major layouts in units of Type
   97 | template <class Type>
   98 | using Layout_K_INTER_Atom = decltype(upcast<sizeof_bits<Type>::value>(Layout_K_INTER_Atom_Bits{}));
   99 | template <class Type>
  100 | using Layout_K_SW32_Atom  = decltype(upcast<sizeof_bits<Type>::value>(Layout_K_SW32_Atom_Bits{}));
  101 | template <class Type>
  102 | using Layout_K_SW64_Atom  = decltype(upcast<sizeof_bits<Type>::value>(Layout_K_SW64_Atom_Bits{}));
  103 | template <class Type>
  104 | using Layout_K_SW128_Atom = decltype(upcast<sizeof_bits<Type>::value>(Layout_K_SW128_Atom_Bits{}));
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 106-122

```text
  106 | // With GMMA::Major param
  107 | template <class Type, Major tnsp>
  108 | using Layout_INTER_Atom = typename conditional<tnsp == Major::MN,
  109 |                                                Layout_MN_INTER_Atom<Type>,
  110 |                                                Layout_K_INTER_Atom<Type>>::type;
  111 | template <class Type, Major tnsp>
  112 | using Layout_SW32_Atom = typename conditional<tnsp == Major::MN,
  113 |                                               Layout_MN_SW32_Atom<Type>,
  114 |                                               Layout_K_SW32_Atom<Type>>::type;
  115 | template <class Type, Major tnsp>
  116 | using Layout_SW64_Atom = typename conditional<tnsp == Major::MN,
  117 |                                               Layout_MN_SW64_Atom<Type>,
  118 |                                               Layout_K_SW64_Atom<Type>>::type;
  119 | template <class Type, Major tnsp>
  120 | using Layout_SW128_Atom = typename conditional<tnsp == Major::MN,
  121 |                                                Layout_MN_SW128_Atom<Type>,
  122 |                                                Layout_K_SW128_Atom<Type>>::type;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 124-134

```text
  124 | //
  125 | // Tensor (position-dependent swizzle) to LayoutType utility
  126 | //
  128 | template <class Engine, class Shape, class Stride>
  129 | CUTE_HOST_DEVICE constexpr
  130 | LayoutType
  131 | layout_type(Tensor<Engine, Layout<Shape,Stride>> const&)
  132 | {
  133 |   static_assert(is_same<uint128_t, typename Engine::value_type>::value,
  134 |                 "Expected uint128_t type in LayoutType conversion.");
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 136-139

```text
  136 |   using Swizzle = get_swizzle_t<Engine>;
  137 |   constexpr int B = Swizzle::num_bits;
  138 |   constexpr int M = Swizzle::num_base;
  139 |   constexpr int S = Swizzle::num_shft;
```
**EN:** Defines swizzle-related transformations that reorder indices to match hardware-friendly memory access patterns.
**CN:** 定义与 swizzle 相关的变换，通过重排索引来匹配更适合硬件的内存访问模式。

### Lines 141-143

```text
  141 |   static_assert(M == 4,           "Unsupported layout swizzle");
  142 |   static_assert(0 <= B && B <= 3, "Unsupported layout swizzle");
  143 |   static_assert(S == 3,           "Unsupported layout swizzle");
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 145-152

```text
  145 |   switch (B) {
  146 |     case 0: return LayoutType::INTERLEAVE;
  147 |     case 1: return LayoutType::B32;
  148 |     case 2: return LayoutType::B64;
  149 |     case 3: return LayoutType::B128;
  150 |   }
  151 |   return LayoutType::INTERLEAVE;  // ERROR
  152 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 154-203

```text
  154 | ///////////////////////////////////////////////////////////////////////////////
  155 | // Construction method for GMMA Descriptors
  156 | ///////////////////////////////////////////////////////////////////////////////
  158 | /**
  159 | * ///////////////////////////////
  160 | * // make_gmma_desc<Major::MN> //
  161 | * ///////////////////////////////
  162 | * Each GmmaDescriptor Major-MN describes a canonical layout of the form
  163 | *
  164 | * LayoutType::INTERLEAVE   : Swizzle<0,4,3> o smem_ptr o ((T,1,m),(8,k)):((1,T,SBO),(1T,LBO))
  165 | * LayoutType::B32          : Swizzle<1,4,3> o smem_ptr o ((T,2,m),(8,k)):((1,T,LBO),(2T,SBO))
  166 | * LayoutType::B64          : Swizzle<2,4,3> o smem_ptr o ((T,4,m),(8,k)):((1,T,LBO),(4T,SBO))
  167 | * LayoutType::B128         : Swizzle<3,4,3> o smem_ptr o ((T,8,m),(8,k)):((1,T,LBO),(8T,SBO))
  168 | *
  169 | * where
  170 | *   T  : sizeof(uint128_t) / sizeof(value_type)
  171 | *   m  : integer in [1,16] corresponding to GMMA shape
  172 | *   k  : integer in [1,32] corresponding to GMMA shape
  173 | *   SBO: stride byte offset
  174 | *   LBO: leading byte offset
  175 | *
  176 | * See GMMA::Layout_MN_XXX_Atom<value_type> for building canonical GmmaDescriptor Major-MN layouts.
  177 | * For example,
  178 | *   auto smem_layout = tile_to_shape(Layout_MN_SW128_Atom<value_type>{}, Shape<_128,_64>{});
  179 | * is guaranteed to be accepted by make_gmma_desc<Major::MN> for appropriate value_type.
  180 | *
  181 | * //////////////////////////////
  182 | * // make_gmma_desc<Major::K> //
  183 | * //////////////////////////////
  184 | * Each GmmaDescriptor Major-K describes a canonical layout of the form
  185 | *
  186 | * LayoutType::INTERLEAVE : Swizzle<0,4,3> o smem_ptr o ((8,m),(T,2)):((1T,SBO),(1,LBO))
  187 | * LayoutType::B32        : Swizzle<1,4,3> o smem_ptr o ((8,m),(T,2)):((2T,SBO),(1, T ))
  188 | * LayoutType::B64        : Swizzle<2,4,3> o smem_ptr o ((8,m),(T,2)):((4T,SBO),(1, T ))
  189 | * LayoutType::B128       : Swizzle<3,4,3> o smem_ptr o ((8,m),(T,2)):((8T,SBO),(1, T ))
  190 | *
  191 | * See GMMA::Layout_K_XXX_Atom<value_type> for building canonical GmmaDescriptor Major-K layouts.
  192 | * For example,
  193 | *   auto smem_layout = tile_to_shape(Layout_K_SW128_Atom<value_type>{}, Shape<_128,_64>{});
  194 | * is guaranteed to be accepted by make_gmma_desc<Major::K> for appropriate value_type.
  195 | */
  196 | template <Major MajorMode, class TEngine, class TLayout>
  197 | CUTE_HOST_DEVICE constexpr
  198 | GmmaDescriptor
  199 | make_gmma_desc(Tensor<TEngine,TLayout> const& tensor)
  200 | {
  201 |   static_assert(is_smem<TEngine>::value, "GMMA Descriptors can only be constructed on smem.");
  202 |   static_assert(TLayout::rank == 2, "GMMA Descriptors can only be constructed on rank-2 tensors.");
  203 |   using value_type = typename TEngine::value_type;
```
**EN:** Defines shared-memory pointer wrappers and constructors, including overloads that compose with swizzle-aware addressing.
**CN:** 定义共享内存指针包装器与构造函数，并包含可与 swizzle 感知寻址组合的重载。

### Lines 205-205

```text
  205 |   Tensor u128_tensor = recast<uint128_t const>(tensor);
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 207-208

```text
  207 |   // Result
  208 |   GmmaDescriptor desc;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 210-212

```text
  210 |   // Layout type
  211 |   constexpr LayoutType LAYOUT_TYPE = layout_type(u128_tensor);
  212 |   desc.bitfield.layout_type_ = uint8_t(LAYOUT_TYPE);
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 214-216

```text
  214 |   // Start address (4LSB not included)
  215 |   uint32_t start_address = cast_smem_ptr_to_uint(raw_pointer_cast(u128_tensor.data()));
  216 |   desc.bitfield.start_address_ = static_cast<uint16_t>(start_address >> 4);
```
**EN:** Defines shared-memory pointer wrappers and constructors, including overloads that compose with swizzle-aware addressing.
**CN:** 定义共享内存指针包装器与构造函数，并包含可与 swizzle 感知寻址组合的重载。

### Lines 218-219

```text
  218 |   constexpr uint8_t base_offset = 0;
  219 |   desc.bitfield.base_offset_ = base_offset;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 221-225

```text
  221 |   // LayoutType meta
  222 |   constexpr int W = LAYOUT_TYPE == LayoutType::INTERLEAVE ? 1 :
  223 |                     LAYOUT_TYPE == LayoutType::B32        ? 2 :
  224 |                     LAYOUT_TYPE == LayoutType::B64        ? 4 :
  225 |                     LAYOUT_TYPE == LayoutType::B128       ? 8 : -1;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 227-297

```text
  227 |   if constexpr (MajorMode == Major::MN)
  228 |   {
  229 |     /* In units of uint128_t, each GmmaDescriptor Major-MN describes a canonical layout of the form
  230 |      *
  231 |      * LayoutType::INTERLEAVE         : Swizzle<0,4,3> o smem_ptr o ((1,n),(8,k)):((X,SBO),(1,LBO))
  232 |      * LayoutType::B32                : Swizzle<1,4,3> o smem_ptr o ((2,n),(8,k)):((1,LBO),(2,SBO))
  233 |      * LayoutType::B64                : Swizzle<2,4,3> o smem_ptr o ((4,n),(8,k)):((1,LBO),(4,SBO))
  234 |      * LayoutType::B128               : Swizzle<3,4,3> o smem_ptr o ((8,n),(8,k)):((1,LBO),(8,SBO))
  235 |      */
  236 |     static_assert(size<1>(u128_tensor) == Int<(256 / cute::sizeof_bits<value_type>::value)>{} || // A and B in dense MMA
  237 |                   size<1>(u128_tensor) == Int<(128 / cute::sizeof_bits<value_type>::value)>{} || // A in sparse MMA
  238 |                   size<1>(u128_tensor) == Int<(512 / cute::sizeof_bits<value_type>::value)>{},   // B in sparse MMA
  239 |                          "Not a canonical GMMA_MN Layout: Expected K-size 256/sizeof_bits<T> for dense or (128|512)/sizeof_bits<T> for sparse.");
  240 | 
  241 |     // Construct the canonical GMMA T Layout with shape ((W,n),(8,2))
  242 |     Layout canonical_layout = logical_divide(layout(u128_tensor), Tile<Layout<Int<W>,_1>,Layout<Int<8>,_1>>{});
  243 | 
  244 |     // Check profile of canonical
  245 |     CUTE_STATIC_ASSERT_V(congruent(canonical_layout, Shape<Shape<_1,_1>,Shape<_1,_1>>{}), "Not a canonical GMMA_MN Layout: Expected profile failure.");
  246 |     // Check canonical mode strides
  247 |     constexpr uint32_t stride_00 = stride<0,0>(canonical_layout);
  248 |     constexpr uint32_t expected_stride_00 = LAYOUT_TYPE == LayoutType::INTERLEAVE ? stride<0,0>(canonical_layout) : 1;
  249 |     static_assert(stride_00 == expected_stride_00, "Not a canonical GMMA_MN Layout: Expected stride failure.");
  250 |     constexpr uint32_t stride_10 = stride<1,0>(canonical_layout);
  251 |     constexpr uint32_t expected_stride_10 = W;
  252 |     static_assert(stride_10 == expected_stride_10, "Not a canonical GMMA_MN Layout: Expected stride failure.");
  253 | 
  254 |     // stride dimension byte offset and leading dimension byte offset (4LSB not included == uint128_t units)
  255 |     constexpr uint32_t stride_01 = stride<0,1>(canonical_layout);
  256 |     constexpr uint32_t stride_11 = stride<1,1>(canonical_layout);
  257 | 
  258 |     desc.bitfield.stride_byte_offset_  = (LAYOUT_TYPE == LayoutType::INTERLEAVE) ? stride_01 : stride_11;
  259 |     desc.bitfield.leading_byte_offset_ = (LAYOUT_TYPE == LayoutType::INTERLEAVE) ? stride_11 : stride_01;
  260 |   }
  261 |   else if constexpr (MajorMode == Major::K)
  262 |   {
  263 |     /* In units of uint128_t, each GmmaDescriptor Major-K describes a canonical layout of the form
  264 |      *
  265 |      * LayoutType::INTERLEAVE    : Swizzle<0,4,3> o smem_ptr o ((8,n),2):((1,SBO),LBO)
  266 |      * LayoutType::B32           : Swizzle<1,4,3> o smem_ptr o ((8,n),2):((2,SBO),1)
  267 |      * LayoutType::B64           : Swizzle<2,4,3> o smem_ptr o ((8,n),2):((4,SBO),1)
  268 |      * LayoutType::B128          : Swizzle<3,4,3> o smem_ptr o ((8,n),2):((8,SBO),1)
  269 |      */
  270 |     CUTE_STATIC_ASSERT_V(size<0>(u128_tensor) % Int<8>{} == Int<0>{},          // N|M size
  271 |                          "Not a canonical GMMA_K Layout: Expected MN-size multiple of 8.");
  272 |     CUTE_STATIC_ASSERT_V(size<1>(u128_tensor) == Int<2>{} || size<1>(u128_tensor) == Int<4>{},      // K   size
  273 |                          "Not a canonical GMMA_K Layout: Expected K-size 2 for dense or 4 for sparse (in units of uint128_t).");
  274 | 
  275 |     // Construct the canonical GMMA N Layout with shape ((8,n),(2,1))
  276 |     Layout canonical_layout = logical_divide(layout(u128_tensor), Tile<Layout<_8,_1>,Layout<_2,_1>>{});
  277 | 
  278 |     // Check profile of canonical
  279 |     CUTE_STATIC_ASSERT_V(congruent(canonical_layout, Shape<Shape<_1,_1>,Shape<_1,_1>>{}), "Not a canonical GMMA_K Layout: Expected profile failure.");
  280 |     // Check canonical mode strides
  281 |     constexpr uint32_t stride_00 = stride<0,0>(canonical_layout);
  282 |     constexpr uint32_t expected_stride_00 = W;
  283 |     static_assert(stride_00 == expected_stride_00, "Not a canonical GMMA_K Layout: Expected stride failure.");
  284 |     constexpr uint32_t stride_10 = stride<1,0>(canonical_layout);
  285 |     constexpr uint32_t expected_stride_10 = (LAYOUT_TYPE == LayoutType::INTERLEAVE) ? stride<1,0>(canonical_layout) : 1;
  286 |     static_assert(stride_10 == expected_stride_10, "Not a canonical GMMA_K Layout: Expected stride failure.");
  287 | 
  288 |     // stride dimension byte offset and leading dimension byte offset (4LSB not included == uint128_t units)
  289 |     constexpr uint32_t stride_01 = stride<0,1>(canonical_layout);
  290 | 
  291 |     desc.bitfield.stride_byte_offset_  = stride_01;
  292 |     desc.bitfield.leading_byte_offset_ = stride_10;
  293 |   } else {
  294 |     static_assert(MajorMode != Major::MN && MajorMode != Major::K, "Unrecognized MajorMode!");
  295 |   }
  296 |   return desc;
  297 | }
```
**EN:** Defines shared-memory pointer wrappers and constructors, including overloads that compose with swizzle-aware addressing.
**CN:** 定义共享内存指针包装器与构造函数，并包含可与 swizzle 感知寻址组合的重载。

### Lines 299-307

```text
  299 | ///////////////////////////////////////////////////////////////////////////////
  300 | // Higher level GMMA Descriptor utilities
  301 | ///////////////////////////////////////////////////////////////////////////////
  303 | struct DescriptorIterator
  304 | {
  305 |   using reference    = GmmaDescriptor;
  306 |   using element_type = GmmaDescriptor;
  307 |   using value_type   = GmmaDescriptor;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 309-309

```text
  309 |   GmmaDescriptor desc_;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 311-313

```text
  311 |   // Dereference returns the GmmaDescriptor
  312 |   CUTE_HOST_DEVICE constexpr
  313 |   reference operator*() const { return desc_; }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 315-318

```text
  315 |   // Advance and return a new GmmaDescriptor
  316 |   template <class Index>
  317 |   CUTE_HOST_DEVICE constexpr
  318 |   reference operator[](Index const& i) const { return *(*this + i); }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 320-331

```text
  320 |   // Return an advanced iterator
  321 |   template <class Index>
  322 |   CUTE_HOST_DEVICE constexpr
  323 |   DescriptorIterator operator+(Index const& offset) const
  324 |   {
  325 |     // Use 32bit calculation rather than 64 bit calculation as we only update the part of desc
  326 |     GmmaDescriptor ret;
  327 |     ret.reg32_[0] = desc_.reg32_[0] + uint32_t(offset);
  328 |     ret.reg32_[1] = desc_.reg32_[1];
  329 |     return { ret };
  330 |   }
  331 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 333-338

```text
  333 | template <class T>
  334 | CUTE_HOST_DEVICE constexpr
  335 | GmmaDescriptor
  336 | raw_pointer_cast(DescriptorIterator const& ptr) {
  337 |   return ptr.desc_;
  338 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 340-347

```text
  340 | // Recast a DescriptorIterator Tensor to uint64_t, it's RegType in mma_unpack
  341 | template <class NewT>
  342 | CUTE_HOST_DEVICE constexpr
  343 | DescriptorIterator
  344 | recast_ptr(DescriptorIterator const& iter) {
  345 |   static_assert(is_same<NewT, uint64_t>::value, "Can only cast GmmaDescriptorIterator to uint64_t.");
  346 |   return iter;  // Do nothing, it will still dereference to GmmaDescriptor and decay to uint64_t
  347 | }
```
**EN:** Implements `recast_ptr`, preserving sparse or sub-byte semantics when converting a raw address to a CuTe-aware iterator or pointer type.
**CN:** 实现 `recast_ptr`，在把原始地址转换为 CuTe 感知的迭代器或指针类型时保留稀疏或子字节语义。

### Lines 349-352

```text
  349 | CUTE_HOST_DEVICE void
  350 | print(DescriptorIterator) {
  351 |   printf("GMMA::DescriptorIterator");
  352 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 354-359

```text
  354 | // The GMMA Traits below have custom fragment type flags for their smem desc tensors.
  355 | // These flags specialize a MakeTensor customization point to correctly make the fragment that is desired.
  356 | template <Major>
  357 | struct smem_desc : DescriptorIterator {};
  359 | } // end namespace SM90::GMMA
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 361-373

```text
  361 | // Customization point for creating a GMMA::smem_desc Tensor
  362 | template <SM90::GMMA::Major MajorMode>
  363 | struct MakeTensor<SM90::GMMA::smem_desc<MajorMode>>
  364 | {
  365 |   template <class TEngine, class TLayout>
  366 |   CUTE_HOST_DEVICE constexpr auto
  367 |   operator()(Tensor<TEngine,TLayout> const& smem_tensor)
  368 |   {
  369 |     static_assert(is_smem<TEngine>::value, "Expected SMEM Tensor to construct a GMMA Desc Tensor");
  370 |     return make_tensor(SM90::GMMA::DescriptorIterator{SM90::GMMA::make_gmma_desc<MajorMode>(tensor<0>(smem_tensor))},
  371 |                        replace<0>(recast<uint128_t const>(smem_tensor).layout(), Layout<_1,_0>{}));
  372 |   }
  373 | };
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 375-379

```text
  375 | ///////////////////////////////////////////////////////////////////////////////
  376 | //////////////////////////// MMA_TRAITS ///////////////////////////////////////
  377 | ///////////////////////////////////////////////////////////////////////////////
  379 | namespace SM90::GMMA {
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 381-401

```text
  381 | //
  382 | // Specialized mma_unpack implementation for SM90 GMMA instructions
  383 | //
  385 | template <class MMA_Op, class... MMA_Args,
  386 |           class TD, class DLayout,
  387 |           class TA, class ALayout,
  388 |           class TB, class BLayout,
  389 |           class TC, class CLayout>
  390 | CUTE_HOST_DEVICE constexpr
  391 | void
  392 | mma_unpack(MMA_Traits<MMA_Op, MMA_Args...> const& traits,
  393 |            Tensor<TD, DLayout>      & D,
  394 |            Tensor<TA, ALayout> const& A,
  395 |            Tensor<TB, BLayout> const& B,
  396 |            Tensor<TC, CLayout> const& C)
  397 | {
  398 |   static_assert(is_rmem<TD>::value, "Expected registers in MMA_Atom::call");
  399 |   static_assert(is_rmem<TA>::value, "Expected registers in MMA_Atom::call");
  400 |   static_assert(is_rmem<TB>::value, "Expected registers in MMA_Atom::call");
  401 |   static_assert(is_rmem<TC>::value, "Expected registers in MMA_Atom::call");
```
**EN:** Specializes `MMA_Traits` for `MMA_Op, MMA_Args...`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `MMA_Op, MMA_Args...` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 403-406

```text
  403 |   // Register value types from the MMA_Operation register arrays
  404 |   using RegTypeA = typename remove_extent<typename MMA_Op::ARegisters>::type;
  405 |   using RegTypeB = typename remove_extent<typename MMA_Op::BRegisters>::type;
  406 |   using RegTypeC = typename remove_extent<typename MMA_Op::CRegisters>::type;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 408-411

```text
  408 |   // SM90 GMMA take three arguments rather than four, try to assert C and D are aliased
  409 |   static_assert(is_same<typename TD::value_type, typename TC::value_type>::value, "GMMA C and D value_type must match.");
  410 |   static_assert(is_same<DLayout, CLayout>::value, "GMMA C and D layouts must match.");
  411 |   // assert((void*)&C == (void*)&D);
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 413-415

```text
  413 |   Tensor rA = recast<RegTypeA>(A);
  414 |   Tensor rB = recast<RegTypeB>(B);
  415 |   Tensor rC = recast<RegTypeC>(D);  // NOTE: D and C are same, so use mutable D
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 417-419

```text
  417 |   constexpr int RegNumA = extent<typename MMA_Op::ARegisters>::value;
  418 |   constexpr int RegNumB = extent<typename MMA_Op::BRegisters>::value;
  419 |   constexpr int RegNumC = extent<typename MMA_Op::CRegisters>::value;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 421-423

```text
  421 |   CUTE_STATIC_ASSERT_V(size(rA) == Int<RegNumA>{});
  422 |   CUTE_STATIC_ASSERT_V(size(rB) == Int<RegNumB>{});
  423 |   CUTE_STATIC_ASSERT_V(size(rC) == Int<RegNumC>{});
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 425-430

```text
  425 |   detail::explode(MMA_Op::fma,
  426 |                   rA, make_int_sequence<RegNumA>{},
  427 |                   rB, make_int_sequence<RegNumB>{},
  428 |                   rC, make_int_sequence<RegNumC>{},
  429 |                   &(traits.accumulate_), seq<0>{});
  430 | }
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 432-467

```text
  432 | // Accumulator layouts
  433 | template<int N>
  434 | using CLayout_64xN   = Layout<Shape <Shape <  _4,_8, _4>,Shape < _2,_2,Int<N/8>>>,
  435 |                               Stride<Stride<_128,_1,_16>,Stride<_64,_8,   _512>>>;
  437 | using CLayout_64x8   = CLayout_64xN<  8>;
  438 | using CLayout_64x16  = CLayout_64xN< 16>;
  439 | using CLayout_64x32  = CLayout_64xN< 32>;
  440 | using CLayout_64x64  = CLayout_64xN< 64>;
  441 | using CLayout_64x96  = CLayout_64xN< 96>;
  442 | using CLayout_64x128 = CLayout_64xN<128>;
  443 | using CLayout_64x192 = CLayout_64xN<192>;
  444 | using CLayout_64x256 = CLayout_64xN<256>;
  446 | // Register source layout for 32-bit value types
  447 | using ALayout_64x8   = Layout<Shape <Shape <  _4,_8, _4>,Shape <    _2,  _2>>,
  448 |                               Stride<Stride< _64,_1,_16>,Stride<    _8,_256>>>;
  450 | // Register source layout for 16-bit (sparse 32-bit) value types
  451 | using ALayout_64x16  = Layout<Shape <Shape <  _4,_8, _4>,Shape < _2,_2,  _2>>,
  452 |                               Stride<Stride<_128,_1,_16>,Stride<_64,_8,_512>>>;
  454 | // Register source layout for 8-bit (sparse 16-bit) value types
  455 | using ALayout_64x32  = Layout<Shape <Shape <  _4,_8, _4>,Shape < _4,_2,   _2>>,
  456 |                               Stride<Stride<_256,_1,_16>,Stride<_64,_8,_1024>>>;
  458 | // Register source layout for sparse 8-bit value types
  459 | using ALayout_64x64  = Layout<Shape <Shape <  _4,_8, _4>,Shape < _8,_2,   _2>>,
  460 |                               Stride<Stride<_512,_1,_16>,Stride<_64,_8,_2048>>>;
  462 | // Shared memory source layouts for any value type
  463 | template <int M, int K>
  464 | using ABLayout       = Layout<Shape <_128,Shape <Int<M>,Int<K>>>,
  465 |                               Stride<  _0,Stride<    _1,Int<M>>>>;
  467 | } // end namespace SM90::GMMA
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 469-485

```text
  469 | using namespace SM90;
  471 | template <
  472 |   GMMA::Major tnspA,
  473 |   GMMA::Major tnspB,
  474 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  475 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  476 | >
  477 | using SM90_64x8x16_F16F16F16_SS = SM90::GMMA::MMA_64x8x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  479 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  480 | struct MMA_Traits<SM90_64x8x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  481 | {
  482 |   using ValTypeD = half_t;
  483 |   using ValTypeA = half_t;
  484 |   using ValTypeB = half_t;
  485 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 487-494

```text
  487 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  488 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  490 |   using Shape_MNK = Shape<_64,_8,_16>;
  491 |   using ThrID   = Layout<_128>;
  492 |   using ALayout = GMMA::ABLayout< 64, 16>;
  493 |   using BLayout = GMMA::ABLayout<  8, 16>;
  494 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 496-497

```text
  496 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  497 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 499-515

```text
  499 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  501 | template <
  502 |   GMMA::Major tnspA,
  503 |   GMMA::Major tnspB,
  504 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  505 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  506 | >
  507 | using SM90_64x8x16_F16F16F16_RS = SM90::GMMA::MMA_64x8x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  509 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  510 | struct MMA_Traits<SM90_64x8x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  511 | {
  512 |   using ValTypeD = half_t;
  513 |   using ValTypeA = half_t;
  514 |   using ValTypeB = half_t;
  515 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 517-523

```text
  517 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  519 |   using Shape_MNK = Shape<_64,_8,_16>;
  520 |   using ThrID   = Layout<_128>;
  521 |   using ALayout = GMMA::ALayout_64x16;
  522 |   using BLayout = GMMA::ABLayout<  8, 16>;
  523 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 525-526

```text
  525 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  526 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 528-544

```text
  528 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  530 | template <
  531 |   GMMA::Major tnspA,
  532 |   GMMA::Major tnspB,
  533 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  534 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  535 | >
  536 | using SM90_64x16x16_F16F16F16_SS = SM90::GMMA::MMA_64x16x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  538 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  539 | struct MMA_Traits<SM90_64x16x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  540 | {
  541 |   using ValTypeD = half_t;
  542 |   using ValTypeA = half_t;
  543 |   using ValTypeB = half_t;
  544 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 546-553

```text
  546 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  547 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  549 |   using Shape_MNK = Shape<_64,_16,_16>;
  550 |   using ThrID   = Layout<_128>;
  551 |   using ALayout = GMMA::ABLayout< 64, 16>;
  552 |   using BLayout = GMMA::ABLayout< 16, 16>;
  553 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 555-556

```text
  555 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  556 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 558-574

```text
  558 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  560 | template <
  561 |   GMMA::Major tnspA,
  562 |   GMMA::Major tnspB,
  563 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  564 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  565 | >
  566 | using SM90_64x16x16_F16F16F16_RS = SM90::GMMA::MMA_64x16x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  568 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  569 | struct MMA_Traits<SM90_64x16x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  570 | {
  571 |   using ValTypeD = half_t;
  572 |   using ValTypeA = half_t;
  573 |   using ValTypeB = half_t;
  574 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 576-582

```text
  576 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  578 |   using Shape_MNK = Shape<_64,_16,_16>;
  579 |   using ThrID   = Layout<_128>;
  580 |   using ALayout = GMMA::ALayout_64x16;
  581 |   using BLayout = GMMA::ABLayout< 16, 16>;
  582 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 584-585

```text
  584 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  585 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 587-603

```text
  587 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  589 | template <
  590 |   GMMA::Major tnspA,
  591 |   GMMA::Major tnspB,
  592 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  593 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  594 | >
  595 | using SM90_64x32x16_F16F16F16_SS = SM90::GMMA::MMA_64x32x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  597 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  598 | struct MMA_Traits<SM90_64x32x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  599 | {
  600 |   using ValTypeD = half_t;
  601 |   using ValTypeA = half_t;
  602 |   using ValTypeB = half_t;
  603 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 605-612

```text
  605 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  606 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  608 |   using Shape_MNK = Shape<_64,_32,_16>;
  609 |   using ThrID   = Layout<_128>;
  610 |   using ALayout = GMMA::ABLayout< 64, 16>;
  611 |   using BLayout = GMMA::ABLayout< 32, 16>;
  612 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 614-615

```text
  614 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  615 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 617-633

```text
  617 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  619 | template <
  620 |   GMMA::Major tnspA,
  621 |   GMMA::Major tnspB,
  622 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  623 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  624 | >
  625 | using SM90_64x32x16_F16F16F16_RS = SM90::GMMA::MMA_64x32x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  627 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  628 | struct MMA_Traits<SM90_64x32x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  629 | {
  630 |   using ValTypeD = half_t;
  631 |   using ValTypeA = half_t;
  632 |   using ValTypeB = half_t;
  633 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 635-641

```text
  635 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  637 |   using Shape_MNK = Shape<_64,_32,_16>;
  638 |   using ThrID   = Layout<_128>;
  639 |   using ALayout = GMMA::ALayout_64x16;
  640 |   using BLayout = GMMA::ABLayout< 32, 16>;
  641 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 643-644

```text
  643 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  644 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 646-662

```text
  646 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  648 | template <
  649 |   GMMA::Major tnspA,
  650 |   GMMA::Major tnspB,
  651 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  652 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  653 | >
  654 | using SM90_64x64x16_F16F16F16_SS = SM90::GMMA::MMA_64x64x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  656 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  657 | struct MMA_Traits<SM90_64x64x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  658 | {
  659 |   using ValTypeD = half_t;
  660 |   using ValTypeA = half_t;
  661 |   using ValTypeB = half_t;
  662 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 664-671

```text
  664 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  665 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  667 |   using Shape_MNK = Shape<_64,_64,_16>;
  668 |   using ThrID   = Layout<_128>;
  669 |   using ALayout = GMMA::ABLayout< 64, 16>;
  670 |   using BLayout = GMMA::ABLayout< 64, 16>;
  671 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 673-674

```text
  673 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  674 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 676-692

```text
  676 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  678 | template <
  679 |   GMMA::Major tnspA,
  680 |   GMMA::Major tnspB,
  681 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  682 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  683 | >
  684 | using SM90_64x64x16_F16F16F16_RS = SM90::GMMA::MMA_64x64x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  686 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  687 | struct MMA_Traits<SM90_64x64x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  688 | {
  689 |   using ValTypeD = half_t;
  690 |   using ValTypeA = half_t;
  691 |   using ValTypeB = half_t;
  692 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 694-700

```text
  694 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  696 |   using Shape_MNK = Shape<_64,_64,_16>;
  697 |   using ThrID   = Layout<_128>;
  698 |   using ALayout = GMMA::ALayout_64x16;
  699 |   using BLayout = GMMA::ABLayout< 64, 16>;
  700 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 702-703

```text
  702 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  703 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 705-721

```text
  705 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  707 | template <
  708 |   GMMA::Major tnspA,
  709 |   GMMA::Major tnspB,
  710 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  711 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  712 | >
  713 | using SM90_64x96x16_F16F16F16_SS = SM90::GMMA::MMA_64x96x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  715 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  716 | struct MMA_Traits<SM90_64x96x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  717 | {
  718 |   using ValTypeD = half_t;
  719 |   using ValTypeA = half_t;
  720 |   using ValTypeB = half_t;
  721 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 723-730

```text
  723 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  724 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  726 |   using Shape_MNK = Shape<_64,_96,_16>;
  727 |   using ThrID   = Layout<_128>;
  728 |   using ALayout = GMMA::ABLayout< 64, 16>;
  729 |   using BLayout = GMMA::ABLayout< 96, 16>;
  730 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 732-733

```text
  732 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  733 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 735-751

```text
  735 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  737 | template <
  738 |   GMMA::Major tnspA,
  739 |   GMMA::Major tnspB,
  740 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  741 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  742 | >
  743 | using SM90_64x96x16_F16F16F16_RS = SM90::GMMA::MMA_64x96x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  745 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  746 | struct MMA_Traits<SM90_64x96x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  747 | {
  748 |   using ValTypeD = half_t;
  749 |   using ValTypeA = half_t;
  750 |   using ValTypeB = half_t;
  751 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 753-759

```text
  753 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  755 |   using Shape_MNK = Shape<_64,_96,_16>;
  756 |   using ThrID   = Layout<_128>;
  757 |   using ALayout = GMMA::ALayout_64x16;
  758 |   using BLayout = GMMA::ABLayout< 96, 16>;
  759 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 761-762

```text
  761 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  762 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 764-780

```text
  764 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  766 | template <
  767 |   GMMA::Major tnspA,
  768 |   GMMA::Major tnspB,
  769 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  770 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  771 | >
  772 | using SM90_64x128x16_F16F16F16_SS = SM90::GMMA::MMA_64x128x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  774 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  775 | struct MMA_Traits<SM90_64x128x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  776 | {
  777 |   using ValTypeD = half_t;
  778 |   using ValTypeA = half_t;
  779 |   using ValTypeB = half_t;
  780 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 782-789

```text
  782 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  783 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  785 |   using Shape_MNK = Shape<_64,_128,_16>;
  786 |   using ThrID   = Layout<_128>;
  787 |   using ALayout = GMMA::ABLayout< 64, 16>;
  788 |   using BLayout = GMMA::ABLayout<128, 16>;
  789 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 791-792

```text
  791 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  792 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 794-810

```text
  794 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  796 | template <
  797 |   GMMA::Major tnspA,
  798 |   GMMA::Major tnspB,
  799 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  800 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  801 | >
  802 | using SM90_64x128x16_F16F16F16_RS = SM90::GMMA::MMA_64x128x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  804 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  805 | struct MMA_Traits<SM90_64x128x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  806 | {
  807 |   using ValTypeD = half_t;
  808 |   using ValTypeA = half_t;
  809 |   using ValTypeB = half_t;
  810 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 812-818

```text
  812 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  814 |   using Shape_MNK = Shape<_64,_128,_16>;
  815 |   using ThrID   = Layout<_128>;
  816 |   using ALayout = GMMA::ALayout_64x16;
  817 |   using BLayout = GMMA::ABLayout<128, 16>;
  818 |   using CLayout = GMMA::CLayout_64x128;
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

### Lines 823-839

```text
  823 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  825 | template <
  826 |   GMMA::Major tnspA,
  827 |   GMMA::Major tnspB,
  828 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  829 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  830 | >
  831 | using SM90_64x192x16_F16F16F16_SS = SM90::GMMA::MMA_64x192x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  833 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  834 | struct MMA_Traits<SM90_64x192x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  835 | {
  836 |   using ValTypeD = half_t;
  837 |   using ValTypeA = half_t;
  838 |   using ValTypeB = half_t;
  839 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 841-848

```text
  841 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  842 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  844 |   using Shape_MNK = Shape<_64,_192,_16>;
  845 |   using ThrID   = Layout<_128>;
  846 |   using ALayout = GMMA::ABLayout< 64, 16>;
  847 |   using BLayout = GMMA::ABLayout<192, 16>;
  848 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 850-851

```text
  850 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  851 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 853-869

```text
  853 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  855 | template <
  856 |   GMMA::Major tnspA,
  857 |   GMMA::Major tnspB,
  858 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  859 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  860 | >
  861 | using SM90_64x192x16_F16F16F16_RS = SM90::GMMA::MMA_64x192x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  863 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  864 | struct MMA_Traits<SM90_64x192x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  865 | {
  866 |   using ValTypeD = half_t;
  867 |   using ValTypeA = half_t;
  868 |   using ValTypeB = half_t;
  869 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 871-877

```text
  871 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  873 |   using Shape_MNK = Shape<_64,_192,_16>;
  874 |   using ThrID   = Layout<_128>;
  875 |   using ALayout = GMMA::ALayout_64x16;
  876 |   using BLayout = GMMA::ABLayout<192, 16>;
  877 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 879-880

```text
  879 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  880 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 882-898

```text
  882 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  884 | template <
  885 |   GMMA::Major tnspA,
  886 |   GMMA::Major tnspB,
  887 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  888 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  889 | >
  890 | using SM90_64x256x16_F16F16F16_SS = SM90::GMMA::MMA_64x256x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  892 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  893 | struct MMA_Traits<SM90_64x256x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  894 | {
  895 |   using ValTypeD = half_t;
  896 |   using ValTypeA = half_t;
  897 |   using ValTypeB = half_t;
  898 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x16_F16F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 900-907

```text
  900 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  901 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  903 |   using Shape_MNK = Shape<_64,_256,_16>;
  904 |   using ThrID   = Layout<_128>;
  905 |   using ALayout = GMMA::ABLayout< 64, 16>;
  906 |   using BLayout = GMMA::ABLayout<256, 16>;
  907 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 909-910

```text
  909 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  910 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 912-928

```text
  912 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  914 | template <
  915 |   GMMA::Major tnspA,
  916 |   GMMA::Major tnspB,
  917 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  918 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  919 | >
  920 | using SM90_64x256x16_F16F16F16_RS = SM90::GMMA::MMA_64x256x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  922 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  923 | struct MMA_Traits<SM90_64x256x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  924 | {
  925 |   using ValTypeD = half_t;
  926 |   using ValTypeA = half_t;
  927 |   using ValTypeB = half_t;
  928 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x16_F16F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 930-936

```text
  930 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  932 |   using Shape_MNK = Shape<_64,_256,_16>;
  933 |   using ThrID   = Layout<_128>;
  934 |   using ALayout = GMMA::ALayout_64x16;
  935 |   using BLayout = GMMA::ABLayout<256, 16>;
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

### Lines 941-957

```text
  941 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  943 | template <
  944 |   GMMA::Major tnspA,
  945 |   GMMA::Major tnspB,
  946 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  947 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  948 | >
  949 | using SM90_64x8x16_F32F16F16_SS = SM90::GMMA::MMA_64x8x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
  951 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  952 | struct MMA_Traits<SM90_64x8x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
  953 | {
  954 |   using ValTypeD = float;
  955 |   using ValTypeA = half_t;
  956 |   using ValTypeB = half_t;
  957 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 959-966

```text
  959 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
  960 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  962 |   using Shape_MNK = Shape<_64,_8,_16>;
  963 |   using ThrID   = Layout<_128>;
  964 |   using ALayout = GMMA::ABLayout< 64, 16>;
  965 |   using BLayout = GMMA::ABLayout<  8, 16>;
  966 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 968-969

```text
  968 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  969 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 971-987

```text
  971 | ////////////////////////////////////////////////////////////////////////////////////////////////////
  973 | template <
  974 |   GMMA::Major tnspA,
  975 |   GMMA::Major tnspB,
  976 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
  977 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
  978 | >
  979 | using SM90_64x8x16_F32F16F16_RS = SM90::GMMA::MMA_64x8x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
  981 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
  982 | struct MMA_Traits<SM90_64x8x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
  983 | {
  984 |   using ValTypeD = float;
  985 |   using ValTypeA = half_t;
  986 |   using ValTypeB = half_t;
  987 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 989-995

```text
  989 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
  991 |   using Shape_MNK = Shape<_64,_8,_16>;
  992 |   using ThrID   = Layout<_128>;
  993 |   using ALayout = GMMA::ALayout_64x16;
  994 |   using BLayout = GMMA::ABLayout<  8, 16>;
  995 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 997-998

```text
  997 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
  998 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1000-1016

```text
 1000 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1002 | template <
 1003 |   GMMA::Major tnspA,
 1004 |   GMMA::Major tnspB,
 1005 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1006 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1007 | >
 1008 | using SM90_64x16x16_F32F16F16_SS = SM90::GMMA::MMA_64x16x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1010 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1011 | struct MMA_Traits<SM90_64x16x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1012 | {
 1013 |   using ValTypeD = float;
 1014 |   using ValTypeA = half_t;
 1015 |   using ValTypeB = half_t;
 1016 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1018-1025

```text
 1018 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1019 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1021 |   using Shape_MNK = Shape<_64,_16,_16>;
 1022 |   using ThrID   = Layout<_128>;
 1023 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1024 |   using BLayout = GMMA::ABLayout< 16, 16>;
 1025 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1027-1028

```text
 1027 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1028 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1030-1046

```text
 1030 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1032 | template <
 1033 |   GMMA::Major tnspA,
 1034 |   GMMA::Major tnspB,
 1035 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1036 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1037 | >
 1038 | using SM90_64x16x16_F32F16F16_RS = SM90::GMMA::MMA_64x16x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1040 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1041 | struct MMA_Traits<SM90_64x16x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1042 | {
 1043 |   using ValTypeD = float;
 1044 |   using ValTypeA = half_t;
 1045 |   using ValTypeB = half_t;
 1046 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1048-1054

```text
 1048 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1050 |   using Shape_MNK = Shape<_64,_16,_16>;
 1051 |   using ThrID   = Layout<_128>;
 1052 |   using ALayout = GMMA::ALayout_64x16;
 1053 |   using BLayout = GMMA::ABLayout< 16, 16>;
 1054 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1056-1057

```text
 1056 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1057 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1059-1075

```text
 1059 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1061 | template <
 1062 |   GMMA::Major tnspA,
 1063 |   GMMA::Major tnspB,
 1064 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1065 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1066 | >
 1067 | using SM90_64x32x16_F32F16F16_SS = SM90::GMMA::MMA_64x32x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1069 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1070 | struct MMA_Traits<SM90_64x32x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1071 | {
 1072 |   using ValTypeD = float;
 1073 |   using ValTypeA = half_t;
 1074 |   using ValTypeB = half_t;
 1075 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1077-1084

```text
 1077 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1078 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1080 |   using Shape_MNK = Shape<_64,_32,_16>;
 1081 |   using ThrID   = Layout<_128>;
 1082 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1083 |   using BLayout = GMMA::ABLayout< 32, 16>;
 1084 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1086-1087

```text
 1086 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1087 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1089-1105

```text
 1089 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1091 | template <
 1092 |   GMMA::Major tnspA,
 1093 |   GMMA::Major tnspB,
 1094 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1095 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1096 | >
 1097 | using SM90_64x32x16_F32F16F16_RS = SM90::GMMA::MMA_64x32x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1099 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1100 | struct MMA_Traits<SM90_64x32x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1101 | {
 1102 |   using ValTypeD = float;
 1103 |   using ValTypeA = half_t;
 1104 |   using ValTypeB = half_t;
 1105 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1107-1113

```text
 1107 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1109 |   using Shape_MNK = Shape<_64,_32,_16>;
 1110 |   using ThrID   = Layout<_128>;
 1111 |   using ALayout = GMMA::ALayout_64x16;
 1112 |   using BLayout = GMMA::ABLayout< 32, 16>;
 1113 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1115-1116

```text
 1115 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1116 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1118-1134

```text
 1118 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1120 | template <
 1121 |   GMMA::Major tnspA,
 1122 |   GMMA::Major tnspB,
 1123 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1124 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1125 | >
 1126 | using SM90_64x64x16_F32F16F16_SS = SM90::GMMA::MMA_64x64x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1128 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1129 | struct MMA_Traits<SM90_64x64x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1130 | {
 1131 |   using ValTypeD = float;
 1132 |   using ValTypeA = half_t;
 1133 |   using ValTypeB = half_t;
 1134 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1136-1143

```text
 1136 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1137 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1139 |   using Shape_MNK = Shape<_64,_64,_16>;
 1140 |   using ThrID   = Layout<_128>;
 1141 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1142 |   using BLayout = GMMA::ABLayout< 64, 16>;
 1143 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1145-1146

```text
 1145 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1146 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1148-1164

```text
 1148 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1150 | template <
 1151 |   GMMA::Major tnspA,
 1152 |   GMMA::Major tnspB,
 1153 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1154 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1155 | >
 1156 | using SM90_64x64x16_F32F16F16_RS = SM90::GMMA::MMA_64x64x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1158 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1159 | struct MMA_Traits<SM90_64x64x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1160 | {
 1161 |   using ValTypeD = float;
 1162 |   using ValTypeA = half_t;
 1163 |   using ValTypeB = half_t;
 1164 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1166-1172

```text
 1166 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1168 |   using Shape_MNK = Shape<_64,_64,_16>;
 1169 |   using ThrID   = Layout<_128>;
 1170 |   using ALayout = GMMA::ALayout_64x16;
 1171 |   using BLayout = GMMA::ABLayout< 64, 16>;
 1172 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1174-1175

```text
 1174 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1175 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1177-1193

```text
 1177 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1179 | template <
 1180 |   GMMA::Major tnspA,
 1181 |   GMMA::Major tnspB,
 1182 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1183 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1184 | >
 1185 | using SM90_64x96x16_F32F16F16_SS = SM90::GMMA::MMA_64x96x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1187 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1188 | struct MMA_Traits<SM90_64x96x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1189 | {
 1190 |   using ValTypeD = float;
 1191 |   using ValTypeA = half_t;
 1192 |   using ValTypeB = half_t;
 1193 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1195-1202

```text
 1195 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1196 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1198 |   using Shape_MNK = Shape<_64,_96,_16>;
 1199 |   using ThrID   = Layout<_128>;
 1200 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1201 |   using BLayout = GMMA::ABLayout< 96, 16>;
 1202 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1204-1205

```text
 1204 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1205 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1207-1223

```text
 1207 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1209 | template <
 1210 |   GMMA::Major tnspA,
 1211 |   GMMA::Major tnspB,
 1212 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1213 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1214 | >
 1215 | using SM90_64x96x16_F32F16F16_RS = SM90::GMMA::MMA_64x96x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1217 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1218 | struct MMA_Traits<SM90_64x96x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1219 | {
 1220 |   using ValTypeD = float;
 1221 |   using ValTypeA = half_t;
 1222 |   using ValTypeB = half_t;
 1223 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1225-1231

```text
 1225 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1227 |   using Shape_MNK = Shape<_64,_96,_16>;
 1228 |   using ThrID   = Layout<_128>;
 1229 |   using ALayout = GMMA::ALayout_64x16;
 1230 |   using BLayout = GMMA::ABLayout< 96, 16>;
 1231 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1233-1234

```text
 1233 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1234 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1236-1252

```text
 1236 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1238 | template <
 1239 |   GMMA::Major tnspA,
 1240 |   GMMA::Major tnspB,
 1241 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1242 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1243 | >
 1244 | using SM90_64x128x16_F32F16F16_SS = SM90::GMMA::MMA_64x128x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1246 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1247 | struct MMA_Traits<SM90_64x128x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1248 | {
 1249 |   using ValTypeD = float;
 1250 |   using ValTypeA = half_t;
 1251 |   using ValTypeB = half_t;
 1252 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1254-1261

```text
 1254 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1255 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1257 |   using Shape_MNK = Shape<_64,_128,_16>;
 1258 |   using ThrID   = Layout<_128>;
 1259 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1260 |   using BLayout = GMMA::ABLayout<128, 16>;
 1261 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1263-1264

```text
 1263 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1264 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1266-1282

```text
 1266 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1268 | template <
 1269 |   GMMA::Major tnspA,
 1270 |   GMMA::Major tnspB,
 1271 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1272 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1273 | >
 1274 | using SM90_64x128x16_F32F16F16_RS = SM90::GMMA::MMA_64x128x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1276 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1277 | struct MMA_Traits<SM90_64x128x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1278 | {
 1279 |   using ValTypeD = float;
 1280 |   using ValTypeA = half_t;
 1281 |   using ValTypeB = half_t;
 1282 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1284-1290

```text
 1284 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1286 |   using Shape_MNK = Shape<_64,_128,_16>;
 1287 |   using ThrID   = Layout<_128>;
 1288 |   using ALayout = GMMA::ALayout_64x16;
 1289 |   using BLayout = GMMA::ABLayout<128, 16>;
 1290 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1292-1293

```text
 1292 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1293 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1295-1311

```text
 1295 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1297 | template <
 1298 |   GMMA::Major tnspA,
 1299 |   GMMA::Major tnspB,
 1300 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1301 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1302 | >
 1303 | using SM90_64x192x16_F32F16F16_SS = SM90::GMMA::MMA_64x192x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1305 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1306 | struct MMA_Traits<SM90_64x192x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1307 | {
 1308 |   using ValTypeD = float;
 1309 |   using ValTypeA = half_t;
 1310 |   using ValTypeB = half_t;
 1311 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1313-1320

```text
 1313 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1314 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1316 |   using Shape_MNK = Shape<_64,_192,_16>;
 1317 |   using ThrID   = Layout<_128>;
 1318 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1319 |   using BLayout = GMMA::ABLayout<192, 16>;
 1320 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1322-1323

```text
 1322 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1323 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1325-1341

```text
 1325 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1327 | template <
 1328 |   GMMA::Major tnspA,
 1329 |   GMMA::Major tnspB,
 1330 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1331 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1332 | >
 1333 | using SM90_64x192x16_F32F16F16_RS = SM90::GMMA::MMA_64x192x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1335 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1336 | struct MMA_Traits<SM90_64x192x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1337 | {
 1338 |   using ValTypeD = float;
 1339 |   using ValTypeA = half_t;
 1340 |   using ValTypeB = half_t;
 1341 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1343-1349

```text
 1343 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1345 |   using Shape_MNK = Shape<_64,_192,_16>;
 1346 |   using ThrID   = Layout<_128>;
 1347 |   using ALayout = GMMA::ALayout_64x16;
 1348 |   using BLayout = GMMA::ABLayout<192, 16>;
 1349 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1351-1352

```text
 1351 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1352 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1354-1370

```text
 1354 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1356 | template <
 1357 |   GMMA::Major tnspA,
 1358 |   GMMA::Major tnspB,
 1359 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1360 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1361 | >
 1362 | using SM90_64x256x16_F32F16F16_SS = SM90::GMMA::MMA_64x256x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>;
 1364 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1365 | struct MMA_Traits<SM90_64x256x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB>>
 1366 | {
 1367 |   using ValTypeD = float;
 1368 |   using ValTypeA = half_t;
 1369 |   using ValTypeB = half_t;
 1370 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x16_F32F16F16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1372-1379

```text
 1372 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1373 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1375 |   using Shape_MNK = Shape<_64,_256,_16>;
 1376 |   using ThrID   = Layout<_128>;
 1377 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1378 |   using BLayout = GMMA::ABLayout<256, 16>;
 1379 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1381-1382

```text
 1381 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1382 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1384-1400

```text
 1384 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1386 | template <
 1387 |   GMMA::Major tnspA,
 1388 |   GMMA::Major tnspB,
 1389 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1390 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1391 | >
 1392 | using SM90_64x256x16_F32F16F16_RS = SM90::GMMA::MMA_64x256x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>;
 1394 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1395 | struct MMA_Traits<SM90_64x256x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB>>
 1396 | {
 1397 |   using ValTypeD = float;
 1398 |   using ValTypeA = half_t;
 1399 |   using ValTypeB = half_t;
 1400 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x16_F32F16F16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1402-1408

```text
 1402 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1404 |   using Shape_MNK = Shape<_64,_256,_16>;
 1405 |   using ThrID   = Layout<_128>;
 1406 |   using ALayout = GMMA::ALayout_64x16;
 1407 |   using BLayout = GMMA::ABLayout<256, 16>;
 1408 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1410-1411

```text
 1410 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1411 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1413-1429

```text
 1413 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1415 | template <
 1416 |   GMMA::Major tnspA,
 1417 |   GMMA::Major tnspB,
 1418 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1419 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1420 | >
 1421 | using SM90_64x8x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x8x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1423 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1424 | struct MMA_Traits<SM90_64x8x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1425 | {
 1426 |   using ValTypeD = float;
 1427 |   using ValTypeA = bfloat16_t;
 1428 |   using ValTypeB = bfloat16_t;
 1429 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1431-1438

```text
 1431 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1432 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1434 |   using Shape_MNK = Shape<_64,_8,_16>;
 1435 |   using ThrID   = Layout<_128>;
 1436 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1437 |   using BLayout = GMMA::ABLayout<  8, 16>;
 1438 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1440-1441

```text
 1440 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1441 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1443-1459

```text
 1443 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1445 | template <
 1446 |   GMMA::Major tnspA,
 1447 |   GMMA::Major tnspB,
 1448 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1449 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1450 | >
 1451 | using SM90_64x8x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x8x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1453 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1454 | struct MMA_Traits<SM90_64x8x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1455 | {
 1456 |   using ValTypeD = float;
 1457 |   using ValTypeA = bfloat16_t;
 1458 |   using ValTypeB = bfloat16_t;
 1459 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1461-1467

```text
 1461 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1463 |   using Shape_MNK = Shape<_64,_8,_16>;
 1464 |   using ThrID   = Layout<_128>;
 1465 |   using ALayout = GMMA::ALayout_64x16;
 1466 |   using BLayout = GMMA::ABLayout<  8, 16>;
 1467 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1469-1470

```text
 1469 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1470 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1472-1488

```text
 1472 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1474 | template <
 1475 |   GMMA::Major tnspA,
 1476 |   GMMA::Major tnspB,
 1477 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1478 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1479 | >
 1480 | using SM90_64x16x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x16x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1482 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1483 | struct MMA_Traits<SM90_64x16x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1484 | {
 1485 |   using ValTypeD = float;
 1486 |   using ValTypeA = bfloat16_t;
 1487 |   using ValTypeB = bfloat16_t;
 1488 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1490-1497

```text
 1490 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1491 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1493 |   using Shape_MNK = Shape<_64,_16,_16>;
 1494 |   using ThrID   = Layout<_128>;
 1495 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1496 |   using BLayout = GMMA::ABLayout< 16, 16>;
 1497 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1499-1500

```text
 1499 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1500 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1502-1518

```text
 1502 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1504 | template <
 1505 |   GMMA::Major tnspA,
 1506 |   GMMA::Major tnspB,
 1507 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1508 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1509 | >
 1510 | using SM90_64x16x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x16x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1512 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1513 | struct MMA_Traits<SM90_64x16x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1514 | {
 1515 |   using ValTypeD = float;
 1516 |   using ValTypeA = bfloat16_t;
 1517 |   using ValTypeB = bfloat16_t;
 1518 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1520-1526

```text
 1520 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1522 |   using Shape_MNK = Shape<_64,_16,_16>;
 1523 |   using ThrID   = Layout<_128>;
 1524 |   using ALayout = GMMA::ALayout_64x16;
 1525 |   using BLayout = GMMA::ABLayout< 16, 16>;
 1526 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1528-1529

```text
 1528 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1529 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1531-1547

```text
 1531 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1533 | template <
 1534 |   GMMA::Major tnspA,
 1535 |   GMMA::Major tnspB,
 1536 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1537 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1538 | >
 1539 | using SM90_64x32x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x32x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1541 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1542 | struct MMA_Traits<SM90_64x32x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1543 | {
 1544 |   using ValTypeD = float;
 1545 |   using ValTypeA = bfloat16_t;
 1546 |   using ValTypeB = bfloat16_t;
 1547 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1549-1556

```text
 1549 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1550 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1552 |   using Shape_MNK = Shape<_64,_32,_16>;
 1553 |   using ThrID   = Layout<_128>;
 1554 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1555 |   using BLayout = GMMA::ABLayout< 32, 16>;
 1556 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1558-1559

```text
 1558 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1559 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1561-1577

```text
 1561 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1563 | template <
 1564 |   GMMA::Major tnspA,
 1565 |   GMMA::Major tnspB,
 1566 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1567 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1568 | >
 1569 | using SM90_64x32x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x32x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1571 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1572 | struct MMA_Traits<SM90_64x32x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1573 | {
 1574 |   using ValTypeD = float;
 1575 |   using ValTypeA = bfloat16_t;
 1576 |   using ValTypeB = bfloat16_t;
 1577 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1579-1585

```text
 1579 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1581 |   using Shape_MNK = Shape<_64,_32,_16>;
 1582 |   using ThrID   = Layout<_128>;
 1583 |   using ALayout = GMMA::ALayout_64x16;
 1584 |   using BLayout = GMMA::ABLayout< 32, 16>;
 1585 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1587-1588

```text
 1587 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1588 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1590-1606

```text
 1590 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1592 | template <
 1593 |   GMMA::Major tnspA,
 1594 |   GMMA::Major tnspB,
 1595 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1596 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1597 | >
 1598 | using SM90_64x64x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x64x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1600 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1601 | struct MMA_Traits<SM90_64x64x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1602 | {
 1603 |   using ValTypeD = float;
 1604 |   using ValTypeA = bfloat16_t;
 1605 |   using ValTypeB = bfloat16_t;
 1606 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1608-1615

```text
 1608 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1609 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1611 |   using Shape_MNK = Shape<_64,_64,_16>;
 1612 |   using ThrID   = Layout<_128>;
 1613 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1614 |   using BLayout = GMMA::ABLayout< 64, 16>;
 1615 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1617-1618

```text
 1617 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1618 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1620-1636

```text
 1620 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1622 | template <
 1623 |   GMMA::Major tnspA,
 1624 |   GMMA::Major tnspB,
 1625 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1626 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1627 | >
 1628 | using SM90_64x64x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x64x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1630 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1631 | struct MMA_Traits<SM90_64x64x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1632 | {
 1633 |   using ValTypeD = float;
 1634 |   using ValTypeA = bfloat16_t;
 1635 |   using ValTypeB = bfloat16_t;
 1636 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1638-1644

```text
 1638 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1640 |   using Shape_MNK = Shape<_64,_64,_16>;
 1641 |   using ThrID   = Layout<_128>;
 1642 |   using ALayout = GMMA::ALayout_64x16;
 1643 |   using BLayout = GMMA::ABLayout< 64, 16>;
 1644 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1646-1647

```text
 1646 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1647 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1649-1665

```text
 1649 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1651 | template <
 1652 |   GMMA::Major tnspA,
 1653 |   GMMA::Major tnspB,
 1654 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1655 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1656 | >
 1657 | using SM90_64x96x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x96x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1659 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1660 | struct MMA_Traits<SM90_64x96x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1661 | {
 1662 |   using ValTypeD = float;
 1663 |   using ValTypeA = bfloat16_t;
 1664 |   using ValTypeB = bfloat16_t;
 1665 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1667-1674

```text
 1667 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1668 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1670 |   using Shape_MNK = Shape<_64,_96,_16>;
 1671 |   using ThrID   = Layout<_128>;
 1672 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1673 |   using BLayout = GMMA::ABLayout< 96, 16>;
 1674 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1676-1677

```text
 1676 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1677 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1679-1695

```text
 1679 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1681 | template <
 1682 |   GMMA::Major tnspA,
 1683 |   GMMA::Major tnspB,
 1684 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1685 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1686 | >
 1687 | using SM90_64x96x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x96x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1689 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1690 | struct MMA_Traits<SM90_64x96x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1691 | {
 1692 |   using ValTypeD = float;
 1693 |   using ValTypeA = bfloat16_t;
 1694 |   using ValTypeB = bfloat16_t;
 1695 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1697-1703

```text
 1697 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1699 |   using Shape_MNK = Shape<_64,_96,_16>;
 1700 |   using ThrID   = Layout<_128>;
 1701 |   using ALayout = GMMA::ALayout_64x16;
 1702 |   using BLayout = GMMA::ABLayout< 96, 16>;
 1703 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1705-1706

```text
 1705 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1706 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1708-1724

```text
 1708 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1710 | template <
 1711 |   GMMA::Major tnspA,
 1712 |   GMMA::Major tnspB,
 1713 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1714 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1715 | >
 1716 | using SM90_64x128x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x128x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1718 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1719 | struct MMA_Traits<SM90_64x128x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1720 | {
 1721 |   using ValTypeD = float;
 1722 |   using ValTypeA = bfloat16_t;
 1723 |   using ValTypeB = bfloat16_t;
 1724 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1726-1733

```text
 1726 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1727 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1729 |   using Shape_MNK = Shape<_64,_128,_16>;
 1730 |   using ThrID   = Layout<_128>;
 1731 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1732 |   using BLayout = GMMA::ABLayout<128, 16>;
 1733 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1735-1736

```text
 1735 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1736 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1738-1754

```text
 1738 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1740 | template <
 1741 |   GMMA::Major tnspA,
 1742 |   GMMA::Major tnspB,
 1743 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1744 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1745 | >
 1746 | using SM90_64x128x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x128x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1748 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1749 | struct MMA_Traits<SM90_64x128x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1750 | {
 1751 |   using ValTypeD = float;
 1752 |   using ValTypeA = bfloat16_t;
 1753 |   using ValTypeB = bfloat16_t;
 1754 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1756-1762

```text
 1756 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1758 |   using Shape_MNK = Shape<_64,_128,_16>;
 1759 |   using ThrID   = Layout<_128>;
 1760 |   using ALayout = GMMA::ALayout_64x16;
 1761 |   using BLayout = GMMA::ABLayout<128, 16>;
 1762 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1764-1765

```text
 1764 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1765 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1767-1783

```text
 1767 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1769 | template <
 1770 |   GMMA::Major tnspA,
 1771 |   GMMA::Major tnspB,
 1772 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1773 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1774 | >
 1775 | using SM90_64x192x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x192x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1777 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1778 | struct MMA_Traits<SM90_64x192x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1779 | {
 1780 |   using ValTypeD = float;
 1781 |   using ValTypeA = bfloat16_t;
 1782 |   using ValTypeB = bfloat16_t;
 1783 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1785-1792

```text
 1785 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1786 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1788 |   using Shape_MNK = Shape<_64,_192,_16>;
 1789 |   using ThrID   = Layout<_128>;
 1790 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1791 |   using BLayout = GMMA::ABLayout<192, 16>;
 1792 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1794-1795

```text
 1794 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1795 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1797-1813

```text
 1797 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1799 | template <
 1800 |   GMMA::Major tnspA,
 1801 |   GMMA::Major tnspB,
 1802 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1803 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1804 | >
 1805 | using SM90_64x192x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x192x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1807 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1808 | struct MMA_Traits<SM90_64x192x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1809 | {
 1810 |   using ValTypeD = float;
 1811 |   using ValTypeA = bfloat16_t;
 1812 |   using ValTypeB = bfloat16_t;
 1813 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1815-1821

```text
 1815 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1817 |   using Shape_MNK = Shape<_64,_192,_16>;
 1818 |   using ThrID   = Layout<_128>;
 1819 |   using ALayout = GMMA::ALayout_64x16;
 1820 |   using BLayout = GMMA::ABLayout<192, 16>;
 1821 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1823-1824

```text
 1823 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1824 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1826-1842

```text
 1826 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1828 | template <
 1829 |   GMMA::Major tnspA,
 1830 |   GMMA::Major tnspB,
 1831 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1832 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1833 | >
 1834 | using SM90_64x256x16_F32BF16BF16_SS = SM90::GMMA::MMA_64x256x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>;
 1836 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1837 | struct MMA_Traits<SM90_64x256x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB>>
 1838 | {
 1839 |   using ValTypeD = float;
 1840 |   using ValTypeA = bfloat16_t;
 1841 |   using ValTypeB = bfloat16_t;
 1842 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x16_F32BF16BF16_SS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1844-1851

```text
 1844 |   using FrgTypeA = GMMA::smem_desc<tnspA>;
 1845 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1847 |   using Shape_MNK = Shape<_64,_256,_16>;
 1848 |   using ThrID   = Layout<_128>;
 1849 |   using ALayout = GMMA::ABLayout< 64, 16>;
 1850 |   using BLayout = GMMA::ABLayout<256, 16>;
 1851 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1853-1854

```text
 1853 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1854 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1856-1872

```text
 1856 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1858 | template <
 1859 |   GMMA::Major tnspA,
 1860 |   GMMA::Major tnspB,
 1861 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1862 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1863 | >
 1864 | using SM90_64x256x16_F32BF16BF16_RS = SM90::GMMA::MMA_64x256x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>;
 1866 | template <GMMA::Major tnspA, GMMA::Major tnspB, GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1867 | struct MMA_Traits<SM90_64x256x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB>>
 1868 | {
 1869 |   using ValTypeD = float;
 1870 |   using ValTypeA = bfloat16_t;
 1871 |   using ValTypeB = bfloat16_t;
 1872 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x16_F32BF16BF16_RS<tnspA, tnspB, scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1874-1880

```text
 1874 |   using FrgTypeB = GMMA::smem_desc<tnspB>;
 1876 |   using Shape_MNK = Shape<_64,_256,_16>;
 1877 |   using ThrID   = Layout<_128>;
 1878 |   using ALayout = GMMA::ALayout_64x16;
 1879 |   using BLayout = GMMA::ABLayout<256, 16>;
 1880 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1882-1883

```text
 1882 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1883 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1885-1899

```text
 1885 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1887 | template <
 1888 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1889 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1890 | >
 1891 | using SM90_64x8x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x8x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 1893 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1894 | struct MMA_Traits<SM90_64x8x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 1895 | {
 1896 |   using ValTypeD = float;
 1897 |   using ValTypeA = tfloat32_t;
 1898 |   using ValTypeB = tfloat32_t;
 1899 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1901-1908

```text
 1901 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1902 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1904 |   using Shape_MNK = Shape<_64,_8,_8>;
 1905 |   using ThrID   = Layout<_128>;
 1906 |   using ALayout = GMMA::ABLayout< 64,  8>;
 1907 |   using BLayout = GMMA::ABLayout<  8,  8>;
 1908 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1910-1911

```text
 1910 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1911 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1913-1927

```text
 1913 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1915 | template <
 1916 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1917 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1918 | >
 1919 | using SM90_64x8x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x8x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 1921 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1922 | struct MMA_Traits<SM90_64x8x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 1923 | {
 1924 |   using ValTypeD = float;
 1925 |   using ValTypeA = tfloat32_t;
 1926 |   using ValTypeB = tfloat32_t;
 1927 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1929-1935

```text
 1929 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1931 |   using Shape_MNK = Shape<_64,_8,_8>;
 1932 |   using ThrID   = Layout<_128>;
 1933 |   using ALayout = GMMA::ALayout_64x8;
 1934 |   using BLayout = GMMA::ABLayout<  8,  8>;
 1935 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1937-1938

```text
 1937 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1938 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1940-1954

```text
 1940 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1942 | template <
 1943 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1944 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1945 | >
 1946 | using SM90_64x16x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x16x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 1948 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1949 | struct MMA_Traits<SM90_64x16x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 1950 | {
 1951 |   using ValTypeD = float;
 1952 |   using ValTypeA = tfloat32_t;
 1953 |   using ValTypeB = tfloat32_t;
 1954 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1956-1963

```text
 1956 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 1957 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1959 |   using Shape_MNK = Shape<_64,_16,_8>;
 1960 |   using ThrID   = Layout<_128>;
 1961 |   using ALayout = GMMA::ABLayout< 64,  8>;
 1962 |   using BLayout = GMMA::ABLayout< 16,  8>;
 1963 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1965-1966

```text
 1965 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1966 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1968-1982

```text
 1968 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1970 | template <
 1971 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1972 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 1973 | >
 1974 | using SM90_64x16x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x16x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 1976 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 1977 | struct MMA_Traits<SM90_64x16x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 1978 | {
 1979 |   using ValTypeD = float;
 1980 |   using ValTypeA = tfloat32_t;
 1981 |   using ValTypeB = tfloat32_t;
 1982 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 1984-1990

```text
 1984 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 1986 |   using Shape_MNK = Shape<_64,_16,_8>;
 1987 |   using ThrID   = Layout<_128>;
 1988 |   using ALayout = GMMA::ALayout_64x8;
 1989 |   using BLayout = GMMA::ABLayout< 16,  8>;
 1990 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1992-1993

```text
 1992 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 1993 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 1995-2009

```text
 1995 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 1997 | template <
 1998 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 1999 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2000 | >
 2001 | using SM90_64x32x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x32x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 2003 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2004 | struct MMA_Traits<SM90_64x32x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 2005 | {
 2006 |   using ValTypeD = float;
 2007 |   using ValTypeA = tfloat32_t;
 2008 |   using ValTypeB = tfloat32_t;
 2009 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2011-2018

```text
 2011 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2012 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2014 |   using Shape_MNK = Shape<_64,_32,_8>;
 2015 |   using ThrID   = Layout<_128>;
 2016 |   using ALayout = GMMA::ABLayout< 64,  8>;
 2017 |   using BLayout = GMMA::ABLayout< 32,  8>;
 2018 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2020-2021

```text
 2020 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2021 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2023-2037

```text
 2023 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2025 | template <
 2026 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2027 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2028 | >
 2029 | using SM90_64x32x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x32x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 2031 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2032 | struct MMA_Traits<SM90_64x32x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 2033 | {
 2034 |   using ValTypeD = float;
 2035 |   using ValTypeA = tfloat32_t;
 2036 |   using ValTypeB = tfloat32_t;
 2037 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2039-2045

```text
 2039 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2041 |   using Shape_MNK = Shape<_64,_32,_8>;
 2042 |   using ThrID   = Layout<_128>;
 2043 |   using ALayout = GMMA::ALayout_64x8;
 2044 |   using BLayout = GMMA::ABLayout< 32,  8>;
 2045 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2047-2048

```text
 2047 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2048 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2050-2064

```text
 2050 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2052 | template <
 2053 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2054 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2055 | >
 2056 | using SM90_64x64x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x64x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 2058 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2059 | struct MMA_Traits<SM90_64x64x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 2060 | {
 2061 |   using ValTypeD = float;
 2062 |   using ValTypeA = tfloat32_t;
 2063 |   using ValTypeB = tfloat32_t;
 2064 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2066-2073

```text
 2066 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2067 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2069 |   using Shape_MNK = Shape<_64,_64,_8>;
 2070 |   using ThrID   = Layout<_128>;
 2071 |   using ALayout = GMMA::ABLayout< 64,  8>;
 2072 |   using BLayout = GMMA::ABLayout< 64,  8>;
 2073 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2075-2076

```text
 2075 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2076 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2078-2092

```text
 2078 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2080 | template <
 2081 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2082 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2083 | >
 2084 | using SM90_64x64x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x64x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 2086 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2087 | struct MMA_Traits<SM90_64x64x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 2088 | {
 2089 |   using ValTypeD = float;
 2090 |   using ValTypeA = tfloat32_t;
 2091 |   using ValTypeB = tfloat32_t;
 2092 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2094-2100

```text
 2094 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2096 |   using Shape_MNK = Shape<_64,_64,_8>;
 2097 |   using ThrID   = Layout<_128>;
 2098 |   using ALayout = GMMA::ALayout_64x8;
 2099 |   using BLayout = GMMA::ABLayout< 64,  8>;
 2100 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2102-2103

```text
 2102 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2103 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2105-2119

```text
 2105 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2107 | template <
 2108 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2109 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2110 | >
 2111 | using SM90_64x96x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x96x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 2113 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2114 | struct MMA_Traits<SM90_64x96x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 2115 | {
 2116 |   using ValTypeD = float;
 2117 |   using ValTypeA = tfloat32_t;
 2118 |   using ValTypeB = tfloat32_t;
 2119 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2121-2128

```text
 2121 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2122 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2124 |   using Shape_MNK = Shape<_64,_96,_8>;
 2125 |   using ThrID   = Layout<_128>;
 2126 |   using ALayout = GMMA::ABLayout< 64,  8>;
 2127 |   using BLayout = GMMA::ABLayout< 96,  8>;
 2128 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2130-2131

```text
 2130 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2131 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2133-2147

```text
 2133 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2135 | template <
 2136 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2137 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2138 | >
 2139 | using SM90_64x96x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x96x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 2141 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2142 | struct MMA_Traits<SM90_64x96x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 2143 | {
 2144 |   using ValTypeD = float;
 2145 |   using ValTypeA = tfloat32_t;
 2146 |   using ValTypeB = tfloat32_t;
 2147 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2149-2155

```text
 2149 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2151 |   using Shape_MNK = Shape<_64,_96,_8>;
 2152 |   using ThrID   = Layout<_128>;
 2153 |   using ALayout = GMMA::ALayout_64x8;
 2154 |   using BLayout = GMMA::ABLayout< 96,  8>;
 2155 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2157-2158

```text
 2157 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2158 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2160-2174

```text
 2160 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2162 | template <
 2163 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2164 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2165 | >
 2166 | using SM90_64x128x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x128x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 2168 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2169 | struct MMA_Traits<SM90_64x128x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 2170 | {
 2171 |   using ValTypeD = float;
 2172 |   using ValTypeA = tfloat32_t;
 2173 |   using ValTypeB = tfloat32_t;
 2174 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2176-2183

```text
 2176 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2177 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2179 |   using Shape_MNK = Shape<_64,_128,_8>;
 2180 |   using ThrID   = Layout<_128>;
 2181 |   using ALayout = GMMA::ABLayout< 64,  8>;
 2182 |   using BLayout = GMMA::ABLayout<128,  8>;
 2183 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2185-2186

```text
 2185 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2186 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2188-2202

```text
 2188 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2190 | template <
 2191 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2192 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2193 | >
 2194 | using SM90_64x128x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x128x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 2196 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2197 | struct MMA_Traits<SM90_64x128x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 2198 | {
 2199 |   using ValTypeD = float;
 2200 |   using ValTypeA = tfloat32_t;
 2201 |   using ValTypeB = tfloat32_t;
 2202 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2204-2210

```text
 2204 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2206 |   using Shape_MNK = Shape<_64,_128,_8>;
 2207 |   using ThrID   = Layout<_128>;
 2208 |   using ALayout = GMMA::ALayout_64x8;
 2209 |   using BLayout = GMMA::ABLayout<128,  8>;
 2210 |   using CLayout = GMMA::CLayout_64x128;
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

### Lines 2215-2229

```text
 2215 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2217 | template <
 2218 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2219 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2220 | >
 2221 | using SM90_64x192x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x192x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 2223 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2224 | struct MMA_Traits<SM90_64x192x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 2225 | {
 2226 |   using ValTypeD = float;
 2227 |   using ValTypeA = tfloat32_t;
 2228 |   using ValTypeB = tfloat32_t;
 2229 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2231-2238

```text
 2231 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2232 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2234 |   using Shape_MNK = Shape<_64,_192,_8>;
 2235 |   using ThrID   = Layout<_128>;
 2236 |   using ALayout = GMMA::ABLayout< 64,  8>;
 2237 |   using BLayout = GMMA::ABLayout<192,  8>;
 2238 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2240-2241

```text
 2240 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2241 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2243-2257

```text
 2243 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2245 | template <
 2246 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2247 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2248 | >
 2249 | using SM90_64x192x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x192x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 2251 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2252 | struct MMA_Traits<SM90_64x192x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 2253 | {
 2254 |   using ValTypeD = float;
 2255 |   using ValTypeA = tfloat32_t;
 2256 |   using ValTypeB = tfloat32_t;
 2257 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2259-2265

```text
 2259 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2261 |   using Shape_MNK = Shape<_64,_192,_8>;
 2262 |   using ThrID   = Layout<_128>;
 2263 |   using ALayout = GMMA::ALayout_64x8;
 2264 |   using BLayout = GMMA::ABLayout<192,  8>;
 2265 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2267-2268

```text
 2267 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2268 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2270-2284

```text
 2270 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2272 | template <
 2273 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2274 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2275 | >
 2276 | using SM90_64x256x8_F32TF32TF32_SS_TN = SM90::GMMA::MMA_64x256x8_F32TF32TF32_SS_TN<scaleA, scaleB>;
 2278 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2279 | struct MMA_Traits<SM90_64x256x8_F32TF32TF32_SS_TN<scaleA, scaleB>>
 2280 | {
 2281 |   using ValTypeD = float;
 2282 |   using ValTypeA = tfloat32_t;
 2283 |   using ValTypeB = tfloat32_t;
 2284 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x8_F32TF32TF32_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x8_F32TF32TF32_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2286-2293

```text
 2286 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2287 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2289 |   using Shape_MNK = Shape<_64,_256,_8>;
 2290 |   using ThrID   = Layout<_128>;
 2291 |   using ALayout = GMMA::ABLayout< 64,  8>;
 2292 |   using BLayout = GMMA::ABLayout<256,  8>;
 2293 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2295-2296

```text
 2295 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2296 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2298-2312

```text
 2298 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2300 | template <
 2301 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 2302 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 2303 | >
 2304 | using SM90_64x256x8_F32TF32TF32_RS_TN = SM90::GMMA::MMA_64x256x8_F32TF32TF32_RS_TN<scaleA, scaleB>;
 2306 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 2307 | struct MMA_Traits<SM90_64x256x8_F32TF32TF32_RS_TN<scaleA, scaleB>>
 2308 | {
 2309 |   using ValTypeD = float;
 2310 |   using ValTypeA = tfloat32_t;
 2311 |   using ValTypeB = tfloat32_t;
 2312 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x8_F32TF32TF32_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x8_F32TF32TF32_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2314-2320

```text
 2314 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2316 |   using Shape_MNK = Shape<_64,_256,_8>;
 2317 |   using ThrID   = Layout<_128>;
 2318 |   using ALayout = GMMA::ALayout_64x8;
 2319 |   using BLayout = GMMA::ABLayout<256,  8>;
 2320 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2322-2323

```text
 2322 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2323 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2325-2336

```text
 2325 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2328 | using SM90_64x8x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x8x32_S32S8S8_SS_TN;
 2330 | template <>
 2331 | struct MMA_Traits<SM90_64x8x32_S32S8S8_SS_TN>
 2332 | {
 2333 |   using ValTypeD = int32_t;
 2334 |   using ValTypeA = int8_t;
 2335 |   using ValTypeB = int8_t;
 2336 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2338-2345

```text
 2338 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2339 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2341 |   using Shape_MNK = Shape<_64,_8,_32>;
 2342 |   using ThrID   = Layout<_128>;
 2343 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2344 |   using BLayout = GMMA::ABLayout<  8, 32>;
 2345 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2347-2348

```text
 2347 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2348 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2350-2361

```text
 2350 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2353 | using SM90_64x8x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32S8S8_SS_TN_SATURATE;
 2355 | template <>
 2356 | struct MMA_Traits<SM90_64x8x32_S32S8S8_SS_TN_SATURATE>
 2357 | {
 2358 |   using ValTypeD = int32_t;
 2359 |   using ValTypeA = int8_t;
 2360 |   using ValTypeB = int8_t;
 2361 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2363-2370

```text
 2363 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2364 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2366 |   using Shape_MNK = Shape<_64,_8,_32>;
 2367 |   using ThrID   = Layout<_128>;
 2368 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2369 |   using BLayout = GMMA::ABLayout<  8, 32>;
 2370 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2372-2373

```text
 2372 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2373 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2375-2386

```text
 2375 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2378 | using SM90_64x16x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x16x32_S32S8S8_SS_TN;
 2380 | template <>
 2381 | struct MMA_Traits<SM90_64x16x32_S32S8S8_SS_TN>
 2382 | {
 2383 |   using ValTypeD = int32_t;
 2384 |   using ValTypeA = int8_t;
 2385 |   using ValTypeB = int8_t;
 2386 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2388-2395

```text
 2388 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2389 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2391 |   using Shape_MNK = Shape<_64,_16,_32>;
 2392 |   using ThrID   = Layout<_128>;
 2393 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2394 |   using BLayout = GMMA::ABLayout< 16, 32>;
 2395 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2397-2398

```text
 2397 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2398 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2400-2411

```text
 2400 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2403 | using SM90_64x16x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32S8S8_SS_TN_SATURATE;
 2405 | template <>
 2406 | struct MMA_Traits<SM90_64x16x32_S32S8S8_SS_TN_SATURATE>
 2407 | {
 2408 |   using ValTypeD = int32_t;
 2409 |   using ValTypeA = int8_t;
 2410 |   using ValTypeB = int8_t;
 2411 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2413-2420

```text
 2413 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2414 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2416 |   using Shape_MNK = Shape<_64,_16,_32>;
 2417 |   using ThrID   = Layout<_128>;
 2418 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2419 |   using BLayout = GMMA::ABLayout< 16, 32>;
 2420 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2422-2423

```text
 2422 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2423 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2425-2436

```text
 2425 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2428 | using SM90_64x32x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x32x32_S32S8S8_SS_TN;
 2430 | template <>
 2431 | struct MMA_Traits<SM90_64x32x32_S32S8S8_SS_TN>
 2432 | {
 2433 |   using ValTypeD = int32_t;
 2434 |   using ValTypeA = int8_t;
 2435 |   using ValTypeB = int8_t;
 2436 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2438-2445

```text
 2438 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2439 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2441 |   using Shape_MNK = Shape<_64,_32,_32>;
 2442 |   using ThrID   = Layout<_128>;
 2443 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2444 |   using BLayout = GMMA::ABLayout< 32, 32>;
 2445 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2447-2448

```text
 2447 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2448 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2450-2461

```text
 2450 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2453 | using SM90_64x32x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32S8S8_SS_TN_SATURATE;
 2455 | template <>
 2456 | struct MMA_Traits<SM90_64x32x32_S32S8S8_SS_TN_SATURATE>
 2457 | {
 2458 |   using ValTypeD = int32_t;
 2459 |   using ValTypeA = int8_t;
 2460 |   using ValTypeB = int8_t;
 2461 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2463-2470

```text
 2463 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2464 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2466 |   using Shape_MNK = Shape<_64,_32,_32>;
 2467 |   using ThrID   = Layout<_128>;
 2468 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2469 |   using BLayout = GMMA::ABLayout< 32, 32>;
 2470 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2472-2473

```text
 2472 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2473 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2475-2486

```text
 2475 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2478 | using SM90_64x64x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x64x32_S32S8S8_SS_TN;
 2480 | template <>
 2481 | struct MMA_Traits<SM90_64x64x32_S32S8S8_SS_TN>
 2482 | {
 2483 |   using ValTypeD = int32_t;
 2484 |   using ValTypeA = int8_t;
 2485 |   using ValTypeB = int8_t;
 2486 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2488-2495

```text
 2488 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2489 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2491 |   using Shape_MNK = Shape<_64,_64,_32>;
 2492 |   using ThrID   = Layout<_128>;
 2493 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2494 |   using BLayout = GMMA::ABLayout< 64, 32>;
 2495 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2497-2498

```text
 2497 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2498 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2500-2511

```text
 2500 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2503 | using SM90_64x64x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32S8S8_SS_TN_SATURATE;
 2505 | template <>
 2506 | struct MMA_Traits<SM90_64x64x32_S32S8S8_SS_TN_SATURATE>
 2507 | {
 2508 |   using ValTypeD = int32_t;
 2509 |   using ValTypeA = int8_t;
 2510 |   using ValTypeB = int8_t;
 2511 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2513-2520

```text
 2513 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2514 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2516 |   using Shape_MNK = Shape<_64,_64,_32>;
 2517 |   using ThrID   = Layout<_128>;
 2518 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2519 |   using BLayout = GMMA::ABLayout< 64, 32>;
 2520 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2522-2523

```text
 2522 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2523 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2525-2536

```text
 2525 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2528 | using SM90_64x96x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x96x32_S32S8S8_SS_TN;
 2530 | template <>
 2531 | struct MMA_Traits<SM90_64x96x32_S32S8S8_SS_TN>
 2532 | {
 2533 |   using ValTypeD = int32_t;
 2534 |   using ValTypeA = int8_t;
 2535 |   using ValTypeB = int8_t;
 2536 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2538-2545

```text
 2538 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2539 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2541 |   using Shape_MNK = Shape<_64,_96,_32>;
 2542 |   using ThrID   = Layout<_128>;
 2543 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2544 |   using BLayout = GMMA::ABLayout< 96, 32>;
 2545 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2547-2548

```text
 2547 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2548 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2550-2561

```text
 2550 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2553 | using SM90_64x96x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32S8S8_SS_TN_SATURATE;
 2555 | template <>
 2556 | struct MMA_Traits<SM90_64x96x32_S32S8S8_SS_TN_SATURATE>
 2557 | {
 2558 |   using ValTypeD = int32_t;
 2559 |   using ValTypeA = int8_t;
 2560 |   using ValTypeB = int8_t;
 2561 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2563-2570

```text
 2563 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2564 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2566 |   using Shape_MNK = Shape<_64,_96,_32>;
 2567 |   using ThrID   = Layout<_128>;
 2568 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2569 |   using BLayout = GMMA::ABLayout< 96, 32>;
 2570 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2572-2573

```text
 2572 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2573 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2575-2586

```text
 2575 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2578 | using SM90_64x128x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x128x32_S32S8S8_SS_TN;
 2580 | template <>
 2581 | struct MMA_Traits<SM90_64x128x32_S32S8S8_SS_TN>
 2582 | {
 2583 |   using ValTypeD = int32_t;
 2584 |   using ValTypeA = int8_t;
 2585 |   using ValTypeB = int8_t;
 2586 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2588-2595

```text
 2588 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2589 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2591 |   using Shape_MNK = Shape<_64,_128,_32>;
 2592 |   using ThrID   = Layout<_128>;
 2593 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2594 |   using BLayout = GMMA::ABLayout<128, 32>;
 2595 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2597-2598

```text
 2597 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2598 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2600-2611

```text
 2600 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2603 | using SM90_64x128x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32S8S8_SS_TN_SATURATE;
 2605 | template <>
 2606 | struct MMA_Traits<SM90_64x128x32_S32S8S8_SS_TN_SATURATE>
 2607 | {
 2608 |   using ValTypeD = int32_t;
 2609 |   using ValTypeA = int8_t;
 2610 |   using ValTypeB = int8_t;
 2611 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2613-2620

```text
 2613 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2614 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2616 |   using Shape_MNK = Shape<_64,_128,_32>;
 2617 |   using ThrID   = Layout<_128>;
 2618 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2619 |   using BLayout = GMMA::ABLayout<128, 32>;
 2620 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2622-2623

```text
 2622 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2623 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2625-2636

```text
 2625 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2628 | using SM90_64x192x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x192x32_S32S8S8_SS_TN;
 2630 | template <>
 2631 | struct MMA_Traits<SM90_64x192x32_S32S8S8_SS_TN>
 2632 | {
 2633 |   using ValTypeD = int32_t;
 2634 |   using ValTypeA = int8_t;
 2635 |   using ValTypeB = int8_t;
 2636 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2638-2645

```text
 2638 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2639 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2641 |   using Shape_MNK = Shape<_64,_192,_32>;
 2642 |   using ThrID   = Layout<_128>;
 2643 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2644 |   using BLayout = GMMA::ABLayout<192, 32>;
 2645 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2647-2648

```text
 2647 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2648 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2650-2661

```text
 2650 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2653 | using SM90_64x192x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32S8S8_SS_TN_SATURATE;
 2655 | template <>
 2656 | struct MMA_Traits<SM90_64x192x32_S32S8S8_SS_TN_SATURATE>
 2657 | {
 2658 |   using ValTypeD = int32_t;
 2659 |   using ValTypeA = int8_t;
 2660 |   using ValTypeB = int8_t;
 2661 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2663-2670

```text
 2663 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2664 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2666 |   using Shape_MNK = Shape<_64,_192,_32>;
 2667 |   using ThrID   = Layout<_128>;
 2668 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2669 |   using BLayout = GMMA::ABLayout<192, 32>;
 2670 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2672-2673

```text
 2672 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2673 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2675-2686

```text
 2675 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2678 | using SM90_64x256x32_S32S8S8_SS_TN = SM90::GMMA::MMA_64x256x32_S32S8S8_SS_TN;
 2680 | template <>
 2681 | struct MMA_Traits<SM90_64x256x32_S32S8S8_SS_TN>
 2682 | {
 2683 |   using ValTypeD = int32_t;
 2684 |   using ValTypeA = int8_t;
 2685 |   using ValTypeB = int8_t;
 2686 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2688-2695

```text
 2688 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2689 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2691 |   using Shape_MNK = Shape<_64,_256,_32>;
 2692 |   using ThrID   = Layout<_128>;
 2693 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2694 |   using BLayout = GMMA::ABLayout<256, 32>;
 2695 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2697-2698

```text
 2697 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2698 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2700-2711

```text
 2700 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2703 | using SM90_64x256x32_S32S8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32S8S8_SS_TN_SATURATE;
 2705 | template <>
 2706 | struct MMA_Traits<SM90_64x256x32_S32S8S8_SS_TN_SATURATE>
 2707 | {
 2708 |   using ValTypeD = int32_t;
 2709 |   using ValTypeA = int8_t;
 2710 |   using ValTypeB = int8_t;
 2711 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2713-2720

```text
 2713 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 2714 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2716 |   using Shape_MNK = Shape<_64,_256,_32>;
 2717 |   using ThrID   = Layout<_128>;
 2718 |   using ALayout = GMMA::ABLayout< 64, 32>;
 2719 |   using BLayout = GMMA::ABLayout<256, 32>;
 2720 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2722-2723

```text
 2722 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2723 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2725-2736

```text
 2725 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2728 | using SM90_64x8x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x8x32_S32S8S8_RS_TN;
 2730 | template <>
 2731 | struct MMA_Traits<SM90_64x8x32_S32S8S8_RS_TN>
 2732 | {
 2733 |   using ValTypeD = int32_t;
 2734 |   using ValTypeA = int8_t;
 2735 |   using ValTypeB = int8_t;
 2736 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2738-2744

```text
 2738 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2740 |   using Shape_MNK = Shape<_64,_8,_32>;
 2741 |   using ThrID   = Layout<_128>;
 2742 |   using ALayout = GMMA::ALayout_64x32;
 2743 |   using BLayout = GMMA::ABLayout<  8, 32>;
 2744 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2746-2747

```text
 2746 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2747 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2749-2760

```text
 2749 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2752 | using SM90_64x8x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32S8S8_RS_TN_SATURATE;
 2754 | template <>
 2755 | struct MMA_Traits<SM90_64x8x32_S32S8S8_RS_TN_SATURATE>
 2756 | {
 2757 |   using ValTypeD = int32_t;
 2758 |   using ValTypeA = int8_t;
 2759 |   using ValTypeB = int8_t;
 2760 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2762-2768

```text
 2762 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2764 |   using Shape_MNK = Shape<_64,_8,_32>;
 2765 |   using ThrID   = Layout<_128>;
 2766 |   using ALayout = GMMA::ALayout_64x32;
 2767 |   using BLayout = GMMA::ABLayout<  8, 32>;
 2768 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2770-2771

```text
 2770 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2771 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2773-2784

```text
 2773 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2776 | using SM90_64x16x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x16x32_S32S8S8_RS_TN;
 2778 | template <>
 2779 | struct MMA_Traits<SM90_64x16x32_S32S8S8_RS_TN>
 2780 | {
 2781 |   using ValTypeD = int32_t;
 2782 |   using ValTypeA = int8_t;
 2783 |   using ValTypeB = int8_t;
 2784 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2786-2792

```text
 2786 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2788 |   using Shape_MNK = Shape<_64,_16,_32>;
 2789 |   using ThrID   = Layout<_128>;
 2790 |   using ALayout = GMMA::ALayout_64x32;
 2791 |   using BLayout = GMMA::ABLayout< 16, 32>;
 2792 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2794-2795

```text
 2794 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2795 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2797-2808

```text
 2797 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2800 | using SM90_64x16x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32S8S8_RS_TN_SATURATE;
 2802 | template <>
 2803 | struct MMA_Traits<SM90_64x16x32_S32S8S8_RS_TN_SATURATE>
 2804 | {
 2805 |   using ValTypeD = int32_t;
 2806 |   using ValTypeA = int8_t;
 2807 |   using ValTypeB = int8_t;
 2808 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2810-2816

```text
 2810 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2812 |   using Shape_MNK = Shape<_64,_16,_32>;
 2813 |   using ThrID   = Layout<_128>;
 2814 |   using ALayout = GMMA::ALayout_64x32;
 2815 |   using BLayout = GMMA::ABLayout< 16, 32>;
 2816 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2818-2819

```text
 2818 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2819 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2821-2832

```text
 2821 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2824 | using SM90_64x32x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x32x32_S32S8S8_RS_TN;
 2826 | template <>
 2827 | struct MMA_Traits<SM90_64x32x32_S32S8S8_RS_TN>
 2828 | {
 2829 |   using ValTypeD = int32_t;
 2830 |   using ValTypeA = int8_t;
 2831 |   using ValTypeB = int8_t;
 2832 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2834-2840

```text
 2834 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2836 |   using Shape_MNK = Shape<_64,_32,_32>;
 2837 |   using ThrID   = Layout<_128>;
 2838 |   using ALayout = GMMA::ALayout_64x32;
 2839 |   using BLayout = GMMA::ABLayout< 32, 32>;
 2840 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2842-2843

```text
 2842 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2843 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2845-2856

```text
 2845 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2848 | using SM90_64x32x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32S8S8_RS_TN_SATURATE;
 2850 | template <>
 2851 | struct MMA_Traits<SM90_64x32x32_S32S8S8_RS_TN_SATURATE>
 2852 | {
 2853 |   using ValTypeD = int32_t;
 2854 |   using ValTypeA = int8_t;
 2855 |   using ValTypeB = int8_t;
 2856 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2858-2864

```text
 2858 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2860 |   using Shape_MNK = Shape<_64,_32,_32>;
 2861 |   using ThrID   = Layout<_128>;
 2862 |   using ALayout = GMMA::ALayout_64x32;
 2863 |   using BLayout = GMMA::ABLayout< 32, 32>;
 2864 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2866-2867

```text
 2866 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2867 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2869-2880

```text
 2869 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2872 | using SM90_64x64x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x64x32_S32S8S8_RS_TN;
 2874 | template <>
 2875 | struct MMA_Traits<SM90_64x64x32_S32S8S8_RS_TN>
 2876 | {
 2877 |   using ValTypeD = int32_t;
 2878 |   using ValTypeA = int8_t;
 2879 |   using ValTypeB = int8_t;
 2880 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2882-2888

```text
 2882 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2884 |   using Shape_MNK = Shape<_64,_64,_32>;
 2885 |   using ThrID   = Layout<_128>;
 2886 |   using ALayout = GMMA::ALayout_64x32;
 2887 |   using BLayout = GMMA::ABLayout< 64, 32>;
 2888 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2890-2891

```text
 2890 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2891 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2893-2904

```text
 2893 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2896 | using SM90_64x64x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32S8S8_RS_TN_SATURATE;
 2898 | template <>
 2899 | struct MMA_Traits<SM90_64x64x32_S32S8S8_RS_TN_SATURATE>
 2900 | {
 2901 |   using ValTypeD = int32_t;
 2902 |   using ValTypeA = int8_t;
 2903 |   using ValTypeB = int8_t;
 2904 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2906-2912

```text
 2906 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2908 |   using Shape_MNK = Shape<_64,_64,_32>;
 2909 |   using ThrID   = Layout<_128>;
 2910 |   using ALayout = GMMA::ALayout_64x32;
 2911 |   using BLayout = GMMA::ABLayout< 64, 32>;
 2912 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2914-2915

```text
 2914 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2915 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2917-2928

```text
 2917 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2920 | using SM90_64x96x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x96x32_S32S8S8_RS_TN;
 2922 | template <>
 2923 | struct MMA_Traits<SM90_64x96x32_S32S8S8_RS_TN>
 2924 | {
 2925 |   using ValTypeD = int32_t;
 2926 |   using ValTypeA = int8_t;
 2927 |   using ValTypeB = int8_t;
 2928 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2930-2936

```text
 2930 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2932 |   using Shape_MNK = Shape<_64,_96,_32>;
 2933 |   using ThrID   = Layout<_128>;
 2934 |   using ALayout = GMMA::ALayout_64x32;
 2935 |   using BLayout = GMMA::ABLayout< 96, 32>;
 2936 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2938-2939

```text
 2938 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2939 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2941-2952

```text
 2941 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2944 | using SM90_64x96x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32S8S8_RS_TN_SATURATE;
 2946 | template <>
 2947 | struct MMA_Traits<SM90_64x96x32_S32S8S8_RS_TN_SATURATE>
 2948 | {
 2949 |   using ValTypeD = int32_t;
 2950 |   using ValTypeA = int8_t;
 2951 |   using ValTypeB = int8_t;
 2952 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2954-2960

```text
 2954 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2956 |   using Shape_MNK = Shape<_64,_96,_32>;
 2957 |   using ThrID   = Layout<_128>;
 2958 |   using ALayout = GMMA::ALayout_64x32;
 2959 |   using BLayout = GMMA::ABLayout< 96, 32>;
 2960 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2962-2963

```text
 2962 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2963 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2965-2976

```text
 2965 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2968 | using SM90_64x128x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x128x32_S32S8S8_RS_TN;
 2970 | template <>
 2971 | struct MMA_Traits<SM90_64x128x32_S32S8S8_RS_TN>
 2972 | {
 2973 |   using ValTypeD = int32_t;
 2974 |   using ValTypeA = int8_t;
 2975 |   using ValTypeB = int8_t;
 2976 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 2978-2984

```text
 2978 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 2980 |   using Shape_MNK = Shape<_64,_128,_32>;
 2981 |   using ThrID   = Layout<_128>;
 2982 |   using ALayout = GMMA::ALayout_64x32;
 2983 |   using BLayout = GMMA::ABLayout<128, 32>;
 2984 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2986-2987

```text
 2986 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 2987 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 2989-3000

```text
 2989 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 2992 | using SM90_64x128x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32S8S8_RS_TN_SATURATE;
 2994 | template <>
 2995 | struct MMA_Traits<SM90_64x128x32_S32S8S8_RS_TN_SATURATE>
 2996 | {
 2997 |   using ValTypeD = int32_t;
 2998 |   using ValTypeA = int8_t;
 2999 |   using ValTypeB = int8_t;
 3000 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3002-3008

```text
 3002 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3004 |   using Shape_MNK = Shape<_64,_128,_32>;
 3005 |   using ThrID   = Layout<_128>;
 3006 |   using ALayout = GMMA::ALayout_64x32;
 3007 |   using BLayout = GMMA::ABLayout<128, 32>;
 3008 |   using CLayout = GMMA::CLayout_64x128;
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

### Lines 3013-3024

```text
 3013 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3016 | using SM90_64x192x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x192x32_S32S8S8_RS_TN;
 3018 | template <>
 3019 | struct MMA_Traits<SM90_64x192x32_S32S8S8_RS_TN>
 3020 | {
 3021 |   using ValTypeD = int32_t;
 3022 |   using ValTypeA = int8_t;
 3023 |   using ValTypeB = int8_t;
 3024 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3026-3032

```text
 3026 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3028 |   using Shape_MNK = Shape<_64,_192,_32>;
 3029 |   using ThrID   = Layout<_128>;
 3030 |   using ALayout = GMMA::ALayout_64x32;
 3031 |   using BLayout = GMMA::ABLayout<192, 32>;
 3032 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3034-3035

```text
 3034 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3035 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3037-3048

```text
 3037 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3040 | using SM90_64x192x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32S8S8_RS_TN_SATURATE;
 3042 | template <>
 3043 | struct MMA_Traits<SM90_64x192x32_S32S8S8_RS_TN_SATURATE>
 3044 | {
 3045 |   using ValTypeD = int32_t;
 3046 |   using ValTypeA = int8_t;
 3047 |   using ValTypeB = int8_t;
 3048 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3050-3056

```text
 3050 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3052 |   using Shape_MNK = Shape<_64,_192,_32>;
 3053 |   using ThrID   = Layout<_128>;
 3054 |   using ALayout = GMMA::ALayout_64x32;
 3055 |   using BLayout = GMMA::ABLayout<192, 32>;
 3056 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3058-3059

```text
 3058 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3059 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3061-3072

```text
 3061 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3064 | using SM90_64x256x32_S32S8S8_RS_TN = SM90::GMMA::MMA_64x256x32_S32S8S8_RS_TN;
 3066 | template <>
 3067 | struct MMA_Traits<SM90_64x256x32_S32S8S8_RS_TN>
 3068 | {
 3069 |   using ValTypeD = int32_t;
 3070 |   using ValTypeA = int8_t;
 3071 |   using ValTypeB = int8_t;
 3072 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3074-3080

```text
 3074 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3076 |   using Shape_MNK = Shape<_64,_256,_32>;
 3077 |   using ThrID   = Layout<_128>;
 3078 |   using ALayout = GMMA::ALayout_64x32;
 3079 |   using BLayout = GMMA::ABLayout<256, 32>;
 3080 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3082-3083

```text
 3082 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3083 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3085-3096

```text
 3085 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3088 | using SM90_64x256x32_S32S8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32S8S8_RS_TN_SATURATE;
 3090 | template <>
 3091 | struct MMA_Traits<SM90_64x256x32_S32S8S8_RS_TN_SATURATE>
 3092 | {
 3093 |   using ValTypeD = int32_t;
 3094 |   using ValTypeA = int8_t;
 3095 |   using ValTypeB = int8_t;
 3096 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3098-3104

```text
 3098 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3100 |   using Shape_MNK = Shape<_64,_256,_32>;
 3101 |   using ThrID   = Layout<_128>;
 3102 |   using ALayout = GMMA::ALayout_64x32;
 3103 |   using BLayout = GMMA::ABLayout<256, 32>;
 3104 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3106-3107

```text
 3106 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3107 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3109-3120

```text
 3109 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3112 | using SM90_64x8x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x8x32_S32S8U8_SS_TN;
 3114 | template <>
 3115 | struct MMA_Traits<SM90_64x8x32_S32S8U8_SS_TN>
 3116 | {
 3117 |   using ValTypeD = int32_t;
 3118 |   using ValTypeA = int8_t;
 3119 |   using ValTypeB = uint8_t;
 3120 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3122-3129

```text
 3122 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3123 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3125 |   using Shape_MNK = Shape<_64,_8,_32>;
 3126 |   using ThrID   = Layout<_128>;
 3127 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3128 |   using BLayout = GMMA::ABLayout<  8, 32>;
 3129 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3131-3132

```text
 3131 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3132 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3134-3145

```text
 3134 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3137 | using SM90_64x8x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32S8U8_SS_TN_SATURATE;
 3139 | template <>
 3140 | struct MMA_Traits<SM90_64x8x32_S32S8U8_SS_TN_SATURATE>
 3141 | {
 3142 |   using ValTypeD = int32_t;
 3143 |   using ValTypeA = int8_t;
 3144 |   using ValTypeB = uint8_t;
 3145 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3147-3154

```text
 3147 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3148 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3150 |   using Shape_MNK = Shape<_64,_8,_32>;
 3151 |   using ThrID   = Layout<_128>;
 3152 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3153 |   using BLayout = GMMA::ABLayout<  8, 32>;
 3154 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3156-3157

```text
 3156 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3157 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3159-3170

```text
 3159 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3162 | using SM90_64x16x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x16x32_S32S8U8_SS_TN;
 3164 | template <>
 3165 | struct MMA_Traits<SM90_64x16x32_S32S8U8_SS_TN>
 3166 | {
 3167 |   using ValTypeD = int32_t;
 3168 |   using ValTypeA = int8_t;
 3169 |   using ValTypeB = uint8_t;
 3170 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3172-3179

```text
 3172 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3173 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3175 |   using Shape_MNK = Shape<_64,_16,_32>;
 3176 |   using ThrID   = Layout<_128>;
 3177 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3178 |   using BLayout = GMMA::ABLayout< 16, 32>;
 3179 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3181-3182

```text
 3181 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3182 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3184-3195

```text
 3184 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3187 | using SM90_64x16x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32S8U8_SS_TN_SATURATE;
 3189 | template <>
 3190 | struct MMA_Traits<SM90_64x16x32_S32S8U8_SS_TN_SATURATE>
 3191 | {
 3192 |   using ValTypeD = int32_t;
 3193 |   using ValTypeA = int8_t;
 3194 |   using ValTypeB = uint8_t;
 3195 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3197-3204

```text
 3197 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3198 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3200 |   using Shape_MNK = Shape<_64,_16,_32>;
 3201 |   using ThrID   = Layout<_128>;
 3202 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3203 |   using BLayout = GMMA::ABLayout< 16, 32>;
 3204 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3206-3207

```text
 3206 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3207 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3209-3220

```text
 3209 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3212 | using SM90_64x32x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x32x32_S32S8U8_SS_TN;
 3214 | template <>
 3215 | struct MMA_Traits<SM90_64x32x32_S32S8U8_SS_TN>
 3216 | {
 3217 |   using ValTypeD = int32_t;
 3218 |   using ValTypeA = int8_t;
 3219 |   using ValTypeB = uint8_t;
 3220 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3222-3229

```text
 3222 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3223 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3225 |   using Shape_MNK = Shape<_64,_32,_32>;
 3226 |   using ThrID   = Layout<_128>;
 3227 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3228 |   using BLayout = GMMA::ABLayout< 32, 32>;
 3229 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3231-3232

```text
 3231 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3232 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3234-3245

```text
 3234 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3237 | using SM90_64x32x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32S8U8_SS_TN_SATURATE;
 3239 | template <>
 3240 | struct MMA_Traits<SM90_64x32x32_S32S8U8_SS_TN_SATURATE>
 3241 | {
 3242 |   using ValTypeD = int32_t;
 3243 |   using ValTypeA = int8_t;
 3244 |   using ValTypeB = uint8_t;
 3245 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3247-3254

```text
 3247 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3248 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3250 |   using Shape_MNK = Shape<_64,_32,_32>;
 3251 |   using ThrID   = Layout<_128>;
 3252 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3253 |   using BLayout = GMMA::ABLayout< 32, 32>;
 3254 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3256-3257

```text
 3256 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3257 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3259-3270

```text
 3259 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3262 | using SM90_64x64x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x64x32_S32S8U8_SS_TN;
 3264 | template <>
 3265 | struct MMA_Traits<SM90_64x64x32_S32S8U8_SS_TN>
 3266 | {
 3267 |   using ValTypeD = int32_t;
 3268 |   using ValTypeA = int8_t;
 3269 |   using ValTypeB = uint8_t;
 3270 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3272-3279

```text
 3272 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3273 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3275 |   using Shape_MNK = Shape<_64,_64,_32>;
 3276 |   using ThrID   = Layout<_128>;
 3277 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3278 |   using BLayout = GMMA::ABLayout< 64, 32>;
 3279 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3281-3282

```text
 3281 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3282 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3284-3295

```text
 3284 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3287 | using SM90_64x64x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32S8U8_SS_TN_SATURATE;
 3289 | template <>
 3290 | struct MMA_Traits<SM90_64x64x32_S32S8U8_SS_TN_SATURATE>
 3291 | {
 3292 |   using ValTypeD = int32_t;
 3293 |   using ValTypeA = int8_t;
 3294 |   using ValTypeB = uint8_t;
 3295 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3297-3304

```text
 3297 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3298 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3300 |   using Shape_MNK = Shape<_64,_64,_32>;
 3301 |   using ThrID   = Layout<_128>;
 3302 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3303 |   using BLayout = GMMA::ABLayout< 64, 32>;
 3304 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3306-3307

```text
 3306 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3307 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3309-3320

```text
 3309 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3312 | using SM90_64x96x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x96x32_S32S8U8_SS_TN;
 3314 | template <>
 3315 | struct MMA_Traits<SM90_64x96x32_S32S8U8_SS_TN>
 3316 | {
 3317 |   using ValTypeD = int32_t;
 3318 |   using ValTypeA = int8_t;
 3319 |   using ValTypeB = uint8_t;
 3320 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3322-3329

```text
 3322 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3323 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3325 |   using Shape_MNK = Shape<_64,_96,_32>;
 3326 |   using ThrID   = Layout<_128>;
 3327 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3328 |   using BLayout = GMMA::ABLayout< 96, 32>;
 3329 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3331-3332

```text
 3331 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3332 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3334-3345

```text
 3334 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3337 | using SM90_64x96x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32S8U8_SS_TN_SATURATE;
 3339 | template <>
 3340 | struct MMA_Traits<SM90_64x96x32_S32S8U8_SS_TN_SATURATE>
 3341 | {
 3342 |   using ValTypeD = int32_t;
 3343 |   using ValTypeA = int8_t;
 3344 |   using ValTypeB = uint8_t;
 3345 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3347-3354

```text
 3347 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3348 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3350 |   using Shape_MNK = Shape<_64,_96,_32>;
 3351 |   using ThrID   = Layout<_128>;
 3352 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3353 |   using BLayout = GMMA::ABLayout< 96, 32>;
 3354 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3356-3357

```text
 3356 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3357 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3359-3370

```text
 3359 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3362 | using SM90_64x128x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x128x32_S32S8U8_SS_TN;
 3364 | template <>
 3365 | struct MMA_Traits<SM90_64x128x32_S32S8U8_SS_TN>
 3366 | {
 3367 |   using ValTypeD = int32_t;
 3368 |   using ValTypeA = int8_t;
 3369 |   using ValTypeB = uint8_t;
 3370 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3372-3379

```text
 3372 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3373 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3375 |   using Shape_MNK = Shape<_64,_128,_32>;
 3376 |   using ThrID   = Layout<_128>;
 3377 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3378 |   using BLayout = GMMA::ABLayout<128, 32>;
 3379 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3381-3382

```text
 3381 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3382 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3384-3395

```text
 3384 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3387 | using SM90_64x128x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32S8U8_SS_TN_SATURATE;
 3389 | template <>
 3390 | struct MMA_Traits<SM90_64x128x32_S32S8U8_SS_TN_SATURATE>
 3391 | {
 3392 |   using ValTypeD = int32_t;
 3393 |   using ValTypeA = int8_t;
 3394 |   using ValTypeB = uint8_t;
 3395 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3397-3404

```text
 3397 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3398 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3400 |   using Shape_MNK = Shape<_64,_128,_32>;
 3401 |   using ThrID   = Layout<_128>;
 3402 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3403 |   using BLayout = GMMA::ABLayout<128, 32>;
 3404 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3406-3407

```text
 3406 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3407 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3409-3420

```text
 3409 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3412 | using SM90_64x192x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x192x32_S32S8U8_SS_TN;
 3414 | template <>
 3415 | struct MMA_Traits<SM90_64x192x32_S32S8U8_SS_TN>
 3416 | {
 3417 |   using ValTypeD = int32_t;
 3418 |   using ValTypeA = int8_t;
 3419 |   using ValTypeB = uint8_t;
 3420 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3422-3429

```text
 3422 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3423 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3425 |   using Shape_MNK = Shape<_64,_192,_32>;
 3426 |   using ThrID   = Layout<_128>;
 3427 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3428 |   using BLayout = GMMA::ABLayout<192, 32>;
 3429 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3431-3432

```text
 3431 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3432 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3434-3445

```text
 3434 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3437 | using SM90_64x192x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32S8U8_SS_TN_SATURATE;
 3439 | template <>
 3440 | struct MMA_Traits<SM90_64x192x32_S32S8U8_SS_TN_SATURATE>
 3441 | {
 3442 |   using ValTypeD = int32_t;
 3443 |   using ValTypeA = int8_t;
 3444 |   using ValTypeB = uint8_t;
 3445 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3447-3454

```text
 3447 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3448 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3450 |   using Shape_MNK = Shape<_64,_192,_32>;
 3451 |   using ThrID   = Layout<_128>;
 3452 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3453 |   using BLayout = GMMA::ABLayout<192, 32>;
 3454 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3456-3457

```text
 3456 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3457 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3459-3470

```text
 3459 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3462 | using SM90_64x256x32_S32S8U8_SS_TN = SM90::GMMA::MMA_64x256x32_S32S8U8_SS_TN;
 3464 | template <>
 3465 | struct MMA_Traits<SM90_64x256x32_S32S8U8_SS_TN>
 3466 | {
 3467 |   using ValTypeD = int32_t;
 3468 |   using ValTypeA = int8_t;
 3469 |   using ValTypeB = uint8_t;
 3470 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3472-3479

```text
 3472 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3473 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3475 |   using Shape_MNK = Shape<_64,_256,_32>;
 3476 |   using ThrID   = Layout<_128>;
 3477 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3478 |   using BLayout = GMMA::ABLayout<256, 32>;
 3479 |   using CLayout = GMMA::CLayout_64x256;
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

### Lines 3484-3495

```text
 3484 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3487 | using SM90_64x256x32_S32S8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32S8U8_SS_TN_SATURATE;
 3489 | template <>
 3490 | struct MMA_Traits<SM90_64x256x32_S32S8U8_SS_TN_SATURATE>
 3491 | {
 3492 |   using ValTypeD = int32_t;
 3493 |   using ValTypeA = int8_t;
 3494 |   using ValTypeB = uint8_t;
 3495 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3497-3504

```text
 3497 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3498 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3500 |   using Shape_MNK = Shape<_64,_256,_32>;
 3501 |   using ThrID   = Layout<_128>;
 3502 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3503 |   using BLayout = GMMA::ABLayout<256, 32>;
 3504 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3506-3507

```text
 3506 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3507 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3509-3520

```text
 3509 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3512 | using SM90_64x8x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x8x32_S32S8U8_RS_TN;
 3514 | template <>
 3515 | struct MMA_Traits<SM90_64x8x32_S32S8U8_RS_TN>
 3516 | {
 3517 |   using ValTypeD = int32_t;
 3518 |   using ValTypeA = int8_t;
 3519 |   using ValTypeB = uint8_t;
 3520 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3522-3528

```text
 3522 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3524 |   using Shape_MNK = Shape<_64,_8,_32>;
 3525 |   using ThrID   = Layout<_128>;
 3526 |   using ALayout = GMMA::ALayout_64x32;
 3527 |   using BLayout = GMMA::ABLayout<  8, 32>;
 3528 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3530-3531

```text
 3530 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3531 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3533-3544

```text
 3533 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3536 | using SM90_64x8x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32S8U8_RS_TN_SATURATE;
 3538 | template <>
 3539 | struct MMA_Traits<SM90_64x8x32_S32S8U8_RS_TN_SATURATE>
 3540 | {
 3541 |   using ValTypeD = int32_t;
 3542 |   using ValTypeA = int8_t;
 3543 |   using ValTypeB = uint8_t;
 3544 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3546-3552

```text
 3546 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3548 |   using Shape_MNK = Shape<_64,_8,_32>;
 3549 |   using ThrID   = Layout<_128>;
 3550 |   using ALayout = GMMA::ALayout_64x32;
 3551 |   using BLayout = GMMA::ABLayout<  8, 32>;
 3552 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3554-3555

```text
 3554 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3555 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3557-3568

```text
 3557 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3560 | using SM90_64x16x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x16x32_S32S8U8_RS_TN;
 3562 | template <>
 3563 | struct MMA_Traits<SM90_64x16x32_S32S8U8_RS_TN>
 3564 | {
 3565 |   using ValTypeD = int32_t;
 3566 |   using ValTypeA = int8_t;
 3567 |   using ValTypeB = uint8_t;
 3568 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3570-3576

```text
 3570 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3572 |   using Shape_MNK = Shape<_64,_16,_32>;
 3573 |   using ThrID   = Layout<_128>;
 3574 |   using ALayout = GMMA::ALayout_64x32;
 3575 |   using BLayout = GMMA::ABLayout< 16, 32>;
 3576 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3578-3579

```text
 3578 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3579 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3581-3592

```text
 3581 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3584 | using SM90_64x16x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32S8U8_RS_TN_SATURATE;
 3586 | template <>
 3587 | struct MMA_Traits<SM90_64x16x32_S32S8U8_RS_TN_SATURATE>
 3588 | {
 3589 |   using ValTypeD = int32_t;
 3590 |   using ValTypeA = int8_t;
 3591 |   using ValTypeB = uint8_t;
 3592 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3594-3600

```text
 3594 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3596 |   using Shape_MNK = Shape<_64,_16,_32>;
 3597 |   using ThrID   = Layout<_128>;
 3598 |   using ALayout = GMMA::ALayout_64x32;
 3599 |   using BLayout = GMMA::ABLayout< 16, 32>;
 3600 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3602-3603

```text
 3602 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3603 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3605-3616

```text
 3605 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3608 | using SM90_64x32x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x32x32_S32S8U8_RS_TN;
 3610 | template <>
 3611 | struct MMA_Traits<SM90_64x32x32_S32S8U8_RS_TN>
 3612 | {
 3613 |   using ValTypeD = int32_t;
 3614 |   using ValTypeA = int8_t;
 3615 |   using ValTypeB = uint8_t;
 3616 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3618-3624

```text
 3618 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3620 |   using Shape_MNK = Shape<_64,_32,_32>;
 3621 |   using ThrID   = Layout<_128>;
 3622 |   using ALayout = GMMA::ALayout_64x32;
 3623 |   using BLayout = GMMA::ABLayout< 32, 32>;
 3624 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3626-3627

```text
 3626 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3627 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3629-3640

```text
 3629 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3632 | using SM90_64x32x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32S8U8_RS_TN_SATURATE;
 3634 | template <>
 3635 | struct MMA_Traits<SM90_64x32x32_S32S8U8_RS_TN_SATURATE>
 3636 | {
 3637 |   using ValTypeD = int32_t;
 3638 |   using ValTypeA = int8_t;
 3639 |   using ValTypeB = uint8_t;
 3640 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3642-3648

```text
 3642 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3644 |   using Shape_MNK = Shape<_64,_32,_32>;
 3645 |   using ThrID   = Layout<_128>;
 3646 |   using ALayout = GMMA::ALayout_64x32;
 3647 |   using BLayout = GMMA::ABLayout< 32, 32>;
 3648 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3650-3651

```text
 3650 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3651 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3653-3664

```text
 3653 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3656 | using SM90_64x64x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x64x32_S32S8U8_RS_TN;
 3658 | template <>
 3659 | struct MMA_Traits<SM90_64x64x32_S32S8U8_RS_TN>
 3660 | {
 3661 |   using ValTypeD = int32_t;
 3662 |   using ValTypeA = int8_t;
 3663 |   using ValTypeB = uint8_t;
 3664 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3666-3672

```text
 3666 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3668 |   using Shape_MNK = Shape<_64,_64,_32>;
 3669 |   using ThrID   = Layout<_128>;
 3670 |   using ALayout = GMMA::ALayout_64x32;
 3671 |   using BLayout = GMMA::ABLayout< 64, 32>;
 3672 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3674-3675

```text
 3674 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3675 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3677-3688

```text
 3677 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3680 | using SM90_64x64x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32S8U8_RS_TN_SATURATE;
 3682 | template <>
 3683 | struct MMA_Traits<SM90_64x64x32_S32S8U8_RS_TN_SATURATE>
 3684 | {
 3685 |   using ValTypeD = int32_t;
 3686 |   using ValTypeA = int8_t;
 3687 |   using ValTypeB = uint8_t;
 3688 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3690-3696

```text
 3690 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3692 |   using Shape_MNK = Shape<_64,_64,_32>;
 3693 |   using ThrID   = Layout<_128>;
 3694 |   using ALayout = GMMA::ALayout_64x32;
 3695 |   using BLayout = GMMA::ABLayout< 64, 32>;
 3696 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3698-3699

```text
 3698 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3699 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3701-3712

```text
 3701 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3704 | using SM90_64x96x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x96x32_S32S8U8_RS_TN;
 3706 | template <>
 3707 | struct MMA_Traits<SM90_64x96x32_S32S8U8_RS_TN>
 3708 | {
 3709 |   using ValTypeD = int32_t;
 3710 |   using ValTypeA = int8_t;
 3711 |   using ValTypeB = uint8_t;
 3712 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3714-3720

```text
 3714 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3716 |   using Shape_MNK = Shape<_64,_96,_32>;
 3717 |   using ThrID   = Layout<_128>;
 3718 |   using ALayout = GMMA::ALayout_64x32;
 3719 |   using BLayout = GMMA::ABLayout< 96, 32>;
 3720 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3722-3723

```text
 3722 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3723 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3725-3736

```text
 3725 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3728 | using SM90_64x96x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32S8U8_RS_TN_SATURATE;
 3730 | template <>
 3731 | struct MMA_Traits<SM90_64x96x32_S32S8U8_RS_TN_SATURATE>
 3732 | {
 3733 |   using ValTypeD = int32_t;
 3734 |   using ValTypeA = int8_t;
 3735 |   using ValTypeB = uint8_t;
 3736 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3738-3744

```text
 3738 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3740 |   using Shape_MNK = Shape<_64,_96,_32>;
 3741 |   using ThrID   = Layout<_128>;
 3742 |   using ALayout = GMMA::ALayout_64x32;
 3743 |   using BLayout = GMMA::ABLayout< 96, 32>;
 3744 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3746-3747

```text
 3746 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3747 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3749-3760

```text
 3749 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3752 | using SM90_64x128x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x128x32_S32S8U8_RS_TN;
 3754 | template <>
 3755 | struct MMA_Traits<SM90_64x128x32_S32S8U8_RS_TN>
 3756 | {
 3757 |   using ValTypeD = int32_t;
 3758 |   using ValTypeA = int8_t;
 3759 |   using ValTypeB = uint8_t;
 3760 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3762-3768

```text
 3762 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3764 |   using Shape_MNK = Shape<_64,_128,_32>;
 3765 |   using ThrID   = Layout<_128>;
 3766 |   using ALayout = GMMA::ALayout_64x32;
 3767 |   using BLayout = GMMA::ABLayout<128, 32>;
 3768 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3770-3771

```text
 3770 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3771 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3773-3784

```text
 3773 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3776 | using SM90_64x128x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32S8U8_RS_TN_SATURATE;
 3778 | template <>
 3779 | struct MMA_Traits<SM90_64x128x32_S32S8U8_RS_TN_SATURATE>
 3780 | {
 3781 |   using ValTypeD = int32_t;
 3782 |   using ValTypeA = int8_t;
 3783 |   using ValTypeB = uint8_t;
 3784 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3786-3792

```text
 3786 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3788 |   using Shape_MNK = Shape<_64,_128,_32>;
 3789 |   using ThrID   = Layout<_128>;
 3790 |   using ALayout = GMMA::ALayout_64x32;
 3791 |   using BLayout = GMMA::ABLayout<128, 32>;
 3792 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3794-3795

```text
 3794 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3795 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3797-3808

```text
 3797 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3800 | using SM90_64x192x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x192x32_S32S8U8_RS_TN;
 3802 | template <>
 3803 | struct MMA_Traits<SM90_64x192x32_S32S8U8_RS_TN>
 3804 | {
 3805 |   using ValTypeD = int32_t;
 3806 |   using ValTypeA = int8_t;
 3807 |   using ValTypeB = uint8_t;
 3808 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3810-3816

```text
 3810 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3812 |   using Shape_MNK = Shape<_64,_192,_32>;
 3813 |   using ThrID   = Layout<_128>;
 3814 |   using ALayout = GMMA::ALayout_64x32;
 3815 |   using BLayout = GMMA::ABLayout<192, 32>;
 3816 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3818-3819

```text
 3818 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3819 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3821-3832

```text
 3821 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3824 | using SM90_64x192x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32S8U8_RS_TN_SATURATE;
 3826 | template <>
 3827 | struct MMA_Traits<SM90_64x192x32_S32S8U8_RS_TN_SATURATE>
 3828 | {
 3829 |   using ValTypeD = int32_t;
 3830 |   using ValTypeA = int8_t;
 3831 |   using ValTypeB = uint8_t;
 3832 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3834-3840

```text
 3834 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3836 |   using Shape_MNK = Shape<_64,_192,_32>;
 3837 |   using ThrID   = Layout<_128>;
 3838 |   using ALayout = GMMA::ALayout_64x32;
 3839 |   using BLayout = GMMA::ABLayout<192, 32>;
 3840 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3842-3843

```text
 3842 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3843 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3845-3856

```text
 3845 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3848 | using SM90_64x256x32_S32S8U8_RS_TN = SM90::GMMA::MMA_64x256x32_S32S8U8_RS_TN;
 3850 | template <>
 3851 | struct MMA_Traits<SM90_64x256x32_S32S8U8_RS_TN>
 3852 | {
 3853 |   using ValTypeD = int32_t;
 3854 |   using ValTypeA = int8_t;
 3855 |   using ValTypeB = uint8_t;
 3856 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3858-3864

```text
 3858 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3860 |   using Shape_MNK = Shape<_64,_256,_32>;
 3861 |   using ThrID   = Layout<_128>;
 3862 |   using ALayout = GMMA::ALayout_64x32;
 3863 |   using BLayout = GMMA::ABLayout<256, 32>;
 3864 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3866-3867

```text
 3866 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3867 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3869-3880

```text
 3869 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3872 | using SM90_64x256x32_S32S8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32S8U8_RS_TN_SATURATE;
 3874 | template <>
 3875 | struct MMA_Traits<SM90_64x256x32_S32S8U8_RS_TN_SATURATE>
 3876 | {
 3877 |   using ValTypeD = int32_t;
 3878 |   using ValTypeA = int8_t;
 3879 |   using ValTypeB = uint8_t;
 3880 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32S8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32S8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3882-3888

```text
 3882 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3884 |   using Shape_MNK = Shape<_64,_256,_32>;
 3885 |   using ThrID   = Layout<_128>;
 3886 |   using ALayout = GMMA::ALayout_64x32;
 3887 |   using BLayout = GMMA::ABLayout<256, 32>;
 3888 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3890-3891

```text
 3890 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3891 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3893-3904

```text
 3893 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3896 | using SM90_64x8x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x8x32_S32U8S8_SS_TN;
 3898 | template <>
 3899 | struct MMA_Traits<SM90_64x8x32_S32U8S8_SS_TN>
 3900 | {
 3901 |   using ValTypeD = int32_t;
 3902 |   using ValTypeA = uint8_t;
 3903 |   using ValTypeB = int8_t;
 3904 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3906-3913

```text
 3906 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3907 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3909 |   using Shape_MNK = Shape<_64,_8,_32>;
 3910 |   using ThrID   = Layout<_128>;
 3911 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3912 |   using BLayout = GMMA::ABLayout<  8, 32>;
 3913 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3915-3916

```text
 3915 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3916 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3918-3929

```text
 3918 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3921 | using SM90_64x8x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32U8S8_SS_TN_SATURATE;
 3923 | template <>
 3924 | struct MMA_Traits<SM90_64x8x32_S32U8S8_SS_TN_SATURATE>
 3925 | {
 3926 |   using ValTypeD = int32_t;
 3927 |   using ValTypeA = uint8_t;
 3928 |   using ValTypeB = int8_t;
 3929 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3931-3938

```text
 3931 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3932 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3934 |   using Shape_MNK = Shape<_64,_8,_32>;
 3935 |   using ThrID   = Layout<_128>;
 3936 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3937 |   using BLayout = GMMA::ABLayout<  8, 32>;
 3938 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3940-3941

```text
 3940 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3941 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3943-3954

```text
 3943 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3946 | using SM90_64x16x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x16x32_S32U8S8_SS_TN;
 3948 | template <>
 3949 | struct MMA_Traits<SM90_64x16x32_S32U8S8_SS_TN>
 3950 | {
 3951 |   using ValTypeD = int32_t;
 3952 |   using ValTypeA = uint8_t;
 3953 |   using ValTypeB = int8_t;
 3954 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3956-3963

```text
 3956 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3957 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3959 |   using Shape_MNK = Shape<_64,_16,_32>;
 3960 |   using ThrID   = Layout<_128>;
 3961 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3962 |   using BLayout = GMMA::ABLayout< 16, 32>;
 3963 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3965-3966

```text
 3965 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3966 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3968-3979

```text
 3968 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3971 | using SM90_64x16x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32U8S8_SS_TN_SATURATE;
 3973 | template <>
 3974 | struct MMA_Traits<SM90_64x16x32_S32U8S8_SS_TN_SATURATE>
 3975 | {
 3976 |   using ValTypeD = int32_t;
 3977 |   using ValTypeA = uint8_t;
 3978 |   using ValTypeB = int8_t;
 3979 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 3981-3988

```text
 3981 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 3982 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 3984 |   using Shape_MNK = Shape<_64,_16,_32>;
 3985 |   using ThrID   = Layout<_128>;
 3986 |   using ALayout = GMMA::ABLayout< 64, 32>;
 3987 |   using BLayout = GMMA::ABLayout< 16, 32>;
 3988 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3990-3991

```text
 3990 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 3991 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 3993-4004

```text
 3993 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 3996 | using SM90_64x32x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x32x32_S32U8S8_SS_TN;
 3998 | template <>
 3999 | struct MMA_Traits<SM90_64x32x32_S32U8S8_SS_TN>
 4000 | {
 4001 |   using ValTypeD = int32_t;
 4002 |   using ValTypeA = uint8_t;
 4003 |   using ValTypeB = int8_t;
 4004 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4006-4013

```text
 4006 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4007 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4009 |   using Shape_MNK = Shape<_64,_32,_32>;
 4010 |   using ThrID   = Layout<_128>;
 4011 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4012 |   using BLayout = GMMA::ABLayout< 32, 32>;
 4013 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4015-4016

```text
 4015 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4016 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4018-4029

```text
 4018 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4021 | using SM90_64x32x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32U8S8_SS_TN_SATURATE;
 4023 | template <>
 4024 | struct MMA_Traits<SM90_64x32x32_S32U8S8_SS_TN_SATURATE>
 4025 | {
 4026 |   using ValTypeD = int32_t;
 4027 |   using ValTypeA = uint8_t;
 4028 |   using ValTypeB = int8_t;
 4029 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4031-4038

```text
 4031 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4032 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4034 |   using Shape_MNK = Shape<_64,_32,_32>;
 4035 |   using ThrID   = Layout<_128>;
 4036 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4037 |   using BLayout = GMMA::ABLayout< 32, 32>;
 4038 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4040-4041

```text
 4040 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4041 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4043-4054

```text
 4043 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4046 | using SM90_64x64x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x64x32_S32U8S8_SS_TN;
 4048 | template <>
 4049 | struct MMA_Traits<SM90_64x64x32_S32U8S8_SS_TN>
 4050 | {
 4051 |   using ValTypeD = int32_t;
 4052 |   using ValTypeA = uint8_t;
 4053 |   using ValTypeB = int8_t;
 4054 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4056-4063

```text
 4056 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4057 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4059 |   using Shape_MNK = Shape<_64,_64,_32>;
 4060 |   using ThrID   = Layout<_128>;
 4061 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4062 |   using BLayout = GMMA::ABLayout< 64, 32>;
 4063 |   using CLayout = GMMA::CLayout_64x64;
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

### Lines 4068-4079

```text
 4068 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4071 | using SM90_64x64x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32U8S8_SS_TN_SATURATE;
 4073 | template <>
 4074 | struct MMA_Traits<SM90_64x64x32_S32U8S8_SS_TN_SATURATE>
 4075 | {
 4076 |   using ValTypeD = int32_t;
 4077 |   using ValTypeA = uint8_t;
 4078 |   using ValTypeB = int8_t;
 4079 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4081-4088

```text
 4081 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4082 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4084 |   using Shape_MNK = Shape<_64,_64,_32>;
 4085 |   using ThrID   = Layout<_128>;
 4086 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4087 |   using BLayout = GMMA::ABLayout< 64, 32>;
 4088 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4090-4091

```text
 4090 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4091 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4093-4104

```text
 4093 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4096 | using SM90_64x96x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x96x32_S32U8S8_SS_TN;
 4098 | template <>
 4099 | struct MMA_Traits<SM90_64x96x32_S32U8S8_SS_TN>
 4100 | {
 4101 |   using ValTypeD = int32_t;
 4102 |   using ValTypeA = uint8_t;
 4103 |   using ValTypeB = int8_t;
 4104 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4106-4113

```text
 4106 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4107 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4109 |   using Shape_MNK = Shape<_64,_96,_32>;
 4110 |   using ThrID   = Layout<_128>;
 4111 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4112 |   using BLayout = GMMA::ABLayout< 96, 32>;
 4113 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4115-4116

```text
 4115 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4116 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4118-4129

```text
 4118 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4121 | using SM90_64x96x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32U8S8_SS_TN_SATURATE;
 4123 | template <>
 4124 | struct MMA_Traits<SM90_64x96x32_S32U8S8_SS_TN_SATURATE>
 4125 | {
 4126 |   using ValTypeD = int32_t;
 4127 |   using ValTypeA = uint8_t;
 4128 |   using ValTypeB = int8_t;
 4129 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4131-4138

```text
 4131 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4132 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4134 |   using Shape_MNK = Shape<_64,_96,_32>;
 4135 |   using ThrID   = Layout<_128>;
 4136 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4137 |   using BLayout = GMMA::ABLayout< 96, 32>;
 4138 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4140-4141

```text
 4140 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4141 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4143-4154

```text
 4143 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4146 | using SM90_64x128x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x128x32_S32U8S8_SS_TN;
 4148 | template <>
 4149 | struct MMA_Traits<SM90_64x128x32_S32U8S8_SS_TN>
 4150 | {
 4151 |   using ValTypeD = int32_t;
 4152 |   using ValTypeA = uint8_t;
 4153 |   using ValTypeB = int8_t;
 4154 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4156-4163

```text
 4156 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4157 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4159 |   using Shape_MNK = Shape<_64,_128,_32>;
 4160 |   using ThrID   = Layout<_128>;
 4161 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4162 |   using BLayout = GMMA::ABLayout<128, 32>;
 4163 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4165-4166

```text
 4165 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4166 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4168-4179

```text
 4168 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4171 | using SM90_64x128x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32U8S8_SS_TN_SATURATE;
 4173 | template <>
 4174 | struct MMA_Traits<SM90_64x128x32_S32U8S8_SS_TN_SATURATE>
 4175 | {
 4176 |   using ValTypeD = int32_t;
 4177 |   using ValTypeA = uint8_t;
 4178 |   using ValTypeB = int8_t;
 4179 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4181-4188

```text
 4181 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4182 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4184 |   using Shape_MNK = Shape<_64,_128,_32>;
 4185 |   using ThrID   = Layout<_128>;
 4186 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4187 |   using BLayout = GMMA::ABLayout<128, 32>;
 4188 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4190-4191

```text
 4190 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4191 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4193-4204

```text
 4193 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4196 | using SM90_64x192x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x192x32_S32U8S8_SS_TN;
 4198 | template <>
 4199 | struct MMA_Traits<SM90_64x192x32_S32U8S8_SS_TN>
 4200 | {
 4201 |   using ValTypeD = int32_t;
 4202 |   using ValTypeA = uint8_t;
 4203 |   using ValTypeB = int8_t;
 4204 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4206-4213

```text
 4206 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4207 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4209 |   using Shape_MNK = Shape<_64,_192,_32>;
 4210 |   using ThrID   = Layout<_128>;
 4211 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4212 |   using BLayout = GMMA::ABLayout<192, 32>;
 4213 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4215-4216

```text
 4215 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4216 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4218-4229

```text
 4218 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4221 | using SM90_64x192x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32U8S8_SS_TN_SATURATE;
 4223 | template <>
 4224 | struct MMA_Traits<SM90_64x192x32_S32U8S8_SS_TN_SATURATE>
 4225 | {
 4226 |   using ValTypeD = int32_t;
 4227 |   using ValTypeA = uint8_t;
 4228 |   using ValTypeB = int8_t;
 4229 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4231-4238

```text
 4231 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4232 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4234 |   using Shape_MNK = Shape<_64,_192,_32>;
 4235 |   using ThrID   = Layout<_128>;
 4236 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4237 |   using BLayout = GMMA::ABLayout<192, 32>;
 4238 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4240-4241

```text
 4240 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4241 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4243-4254

```text
 4243 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4246 | using SM90_64x256x32_S32U8S8_SS_TN = SM90::GMMA::MMA_64x256x32_S32U8S8_SS_TN;
 4248 | template <>
 4249 | struct MMA_Traits<SM90_64x256x32_S32U8S8_SS_TN>
 4250 | {
 4251 |   using ValTypeD = int32_t;
 4252 |   using ValTypeA = uint8_t;
 4253 |   using ValTypeB = int8_t;
 4254 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8S8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8S8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4256-4263

```text
 4256 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4257 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4259 |   using Shape_MNK = Shape<_64,_256,_32>;
 4260 |   using ThrID   = Layout<_128>;
 4261 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4262 |   using BLayout = GMMA::ABLayout<256, 32>;
 4263 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4265-4266

```text
 4265 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4266 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4268-4279

```text
 4268 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4271 | using SM90_64x256x32_S32U8S8_SS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32U8S8_SS_TN_SATURATE;
 4273 | template <>
 4274 | struct MMA_Traits<SM90_64x256x32_S32U8S8_SS_TN_SATURATE>
 4275 | {
 4276 |   using ValTypeD = int32_t;
 4277 |   using ValTypeA = uint8_t;
 4278 |   using ValTypeB = int8_t;
 4279 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8S8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8S8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4281-4288

```text
 4281 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4282 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4284 |   using Shape_MNK = Shape<_64,_256,_32>;
 4285 |   using ThrID   = Layout<_128>;
 4286 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4287 |   using BLayout = GMMA::ABLayout<256, 32>;
 4288 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4290-4291

```text
 4290 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4291 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4293-4304

```text
 4293 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4296 | using SM90_64x8x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x8x32_S32U8S8_RS_TN;
 4298 | template <>
 4299 | struct MMA_Traits<SM90_64x8x32_S32U8S8_RS_TN>
 4300 | {
 4301 |   using ValTypeD = int32_t;
 4302 |   using ValTypeA = uint8_t;
 4303 |   using ValTypeB = int8_t;
 4304 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4306-4312

```text
 4306 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4308 |   using Shape_MNK = Shape<_64,_8,_32>;
 4309 |   using ThrID   = Layout<_128>;
 4310 |   using ALayout = GMMA::ALayout_64x32;
 4311 |   using BLayout = GMMA::ABLayout<  8, 32>;
 4312 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4314-4315

```text
 4314 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4315 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4317-4328

```text
 4317 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4320 | using SM90_64x8x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32U8S8_RS_TN_SATURATE;
 4322 | template <>
 4323 | struct MMA_Traits<SM90_64x8x32_S32U8S8_RS_TN_SATURATE>
 4324 | {
 4325 |   using ValTypeD = int32_t;
 4326 |   using ValTypeA = uint8_t;
 4327 |   using ValTypeB = int8_t;
 4328 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4330-4336

```text
 4330 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4332 |   using Shape_MNK = Shape<_64,_8,_32>;
 4333 |   using ThrID   = Layout<_128>;
 4334 |   using ALayout = GMMA::ALayout_64x32;
 4335 |   using BLayout = GMMA::ABLayout<  8, 32>;
 4336 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4338-4339

```text
 4338 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4339 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4341-4352

```text
 4341 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4344 | using SM90_64x16x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x16x32_S32U8S8_RS_TN;
 4346 | template <>
 4347 | struct MMA_Traits<SM90_64x16x32_S32U8S8_RS_TN>
 4348 | {
 4349 |   using ValTypeD = int32_t;
 4350 |   using ValTypeA = uint8_t;
 4351 |   using ValTypeB = int8_t;
 4352 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4354-4360

```text
 4354 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4356 |   using Shape_MNK = Shape<_64,_16,_32>;
 4357 |   using ThrID   = Layout<_128>;
 4358 |   using ALayout = GMMA::ALayout_64x32;
 4359 |   using BLayout = GMMA::ABLayout< 16, 32>;
 4360 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4362-4363

```text
 4362 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4363 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4365-4376

```text
 4365 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4368 | using SM90_64x16x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32U8S8_RS_TN_SATURATE;
 4370 | template <>
 4371 | struct MMA_Traits<SM90_64x16x32_S32U8S8_RS_TN_SATURATE>
 4372 | {
 4373 |   using ValTypeD = int32_t;
 4374 |   using ValTypeA = uint8_t;
 4375 |   using ValTypeB = int8_t;
 4376 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4378-4384

```text
 4378 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4380 |   using Shape_MNK = Shape<_64,_16,_32>;
 4381 |   using ThrID   = Layout<_128>;
 4382 |   using ALayout = GMMA::ALayout_64x32;
 4383 |   using BLayout = GMMA::ABLayout< 16, 32>;
 4384 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4386-4387

```text
 4386 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4387 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4389-4400

```text
 4389 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4392 | using SM90_64x32x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x32x32_S32U8S8_RS_TN;
 4394 | template <>
 4395 | struct MMA_Traits<SM90_64x32x32_S32U8S8_RS_TN>
 4396 | {
 4397 |   using ValTypeD = int32_t;
 4398 |   using ValTypeA = uint8_t;
 4399 |   using ValTypeB = int8_t;
 4400 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4402-4408

```text
 4402 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4404 |   using Shape_MNK = Shape<_64,_32,_32>;
 4405 |   using ThrID   = Layout<_128>;
 4406 |   using ALayout = GMMA::ALayout_64x32;
 4407 |   using BLayout = GMMA::ABLayout< 32, 32>;
 4408 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4410-4411

```text
 4410 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4411 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4413-4424

```text
 4413 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4416 | using SM90_64x32x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32U8S8_RS_TN_SATURATE;
 4418 | template <>
 4419 | struct MMA_Traits<SM90_64x32x32_S32U8S8_RS_TN_SATURATE>
 4420 | {
 4421 |   using ValTypeD = int32_t;
 4422 |   using ValTypeA = uint8_t;
 4423 |   using ValTypeB = int8_t;
 4424 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4426-4432

```text
 4426 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4428 |   using Shape_MNK = Shape<_64,_32,_32>;
 4429 |   using ThrID   = Layout<_128>;
 4430 |   using ALayout = GMMA::ALayout_64x32;
 4431 |   using BLayout = GMMA::ABLayout< 32, 32>;
 4432 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4434-4435

```text
 4434 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4435 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4437-4448

```text
 4437 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4440 | using SM90_64x64x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x64x32_S32U8S8_RS_TN;
 4442 | template <>
 4443 | struct MMA_Traits<SM90_64x64x32_S32U8S8_RS_TN>
 4444 | {
 4445 |   using ValTypeD = int32_t;
 4446 |   using ValTypeA = uint8_t;
 4447 |   using ValTypeB = int8_t;
 4448 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4450-4456

```text
 4450 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4452 |   using Shape_MNK = Shape<_64,_64,_32>;
 4453 |   using ThrID   = Layout<_128>;
 4454 |   using ALayout = GMMA::ALayout_64x32;
 4455 |   using BLayout = GMMA::ABLayout< 64, 32>;
 4456 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4458-4459

```text
 4458 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4459 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4461-4472

```text
 4461 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4464 | using SM90_64x64x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32U8S8_RS_TN_SATURATE;
 4466 | template <>
 4467 | struct MMA_Traits<SM90_64x64x32_S32U8S8_RS_TN_SATURATE>
 4468 | {
 4469 |   using ValTypeD = int32_t;
 4470 |   using ValTypeA = uint8_t;
 4471 |   using ValTypeB = int8_t;
 4472 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4474-4480

```text
 4474 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4476 |   using Shape_MNK = Shape<_64,_64,_32>;
 4477 |   using ThrID   = Layout<_128>;
 4478 |   using ALayout = GMMA::ALayout_64x32;
 4479 |   using BLayout = GMMA::ABLayout< 64, 32>;
 4480 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4482-4483

```text
 4482 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4483 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4485-4496

```text
 4485 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4488 | using SM90_64x96x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x96x32_S32U8S8_RS_TN;
 4490 | template <>
 4491 | struct MMA_Traits<SM90_64x96x32_S32U8S8_RS_TN>
 4492 | {
 4493 |   using ValTypeD = int32_t;
 4494 |   using ValTypeA = uint8_t;
 4495 |   using ValTypeB = int8_t;
 4496 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4498-4504

```text
 4498 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4500 |   using Shape_MNK = Shape<_64,_96,_32>;
 4501 |   using ThrID   = Layout<_128>;
 4502 |   using ALayout = GMMA::ALayout_64x32;
 4503 |   using BLayout = GMMA::ABLayout< 96, 32>;
 4504 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4506-4507

```text
 4506 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4507 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4509-4520

```text
 4509 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4512 | using SM90_64x96x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32U8S8_RS_TN_SATURATE;
 4514 | template <>
 4515 | struct MMA_Traits<SM90_64x96x32_S32U8S8_RS_TN_SATURATE>
 4516 | {
 4517 |   using ValTypeD = int32_t;
 4518 |   using ValTypeA = uint8_t;
 4519 |   using ValTypeB = int8_t;
 4520 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4522-4528

```text
 4522 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4524 |   using Shape_MNK = Shape<_64,_96,_32>;
 4525 |   using ThrID   = Layout<_128>;
 4526 |   using ALayout = GMMA::ALayout_64x32;
 4527 |   using BLayout = GMMA::ABLayout< 96, 32>;
 4528 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4530-4531

```text
 4530 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4531 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4533-4544

```text
 4533 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4536 | using SM90_64x128x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x128x32_S32U8S8_RS_TN;
 4538 | template <>
 4539 | struct MMA_Traits<SM90_64x128x32_S32U8S8_RS_TN>
 4540 | {
 4541 |   using ValTypeD = int32_t;
 4542 |   using ValTypeA = uint8_t;
 4543 |   using ValTypeB = int8_t;
 4544 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4546-4552

```text
 4546 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4548 |   using Shape_MNK = Shape<_64,_128,_32>;
 4549 |   using ThrID   = Layout<_128>;
 4550 |   using ALayout = GMMA::ALayout_64x32;
 4551 |   using BLayout = GMMA::ABLayout<128, 32>;
 4552 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4554-4555

```text
 4554 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4555 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4557-4568

```text
 4557 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4560 | using SM90_64x128x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32U8S8_RS_TN_SATURATE;
 4562 | template <>
 4563 | struct MMA_Traits<SM90_64x128x32_S32U8S8_RS_TN_SATURATE>
 4564 | {
 4565 |   using ValTypeD = int32_t;
 4566 |   using ValTypeA = uint8_t;
 4567 |   using ValTypeB = int8_t;
 4568 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4570-4576

```text
 4570 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4572 |   using Shape_MNK = Shape<_64,_128,_32>;
 4573 |   using ThrID   = Layout<_128>;
 4574 |   using ALayout = GMMA::ALayout_64x32;
 4575 |   using BLayout = GMMA::ABLayout<128, 32>;
 4576 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4578-4579

```text
 4578 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4579 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4581-4592

```text
 4581 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4584 | using SM90_64x192x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x192x32_S32U8S8_RS_TN;
 4586 | template <>
 4587 | struct MMA_Traits<SM90_64x192x32_S32U8S8_RS_TN>
 4588 | {
 4589 |   using ValTypeD = int32_t;
 4590 |   using ValTypeA = uint8_t;
 4591 |   using ValTypeB = int8_t;
 4592 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4594-4600

```text
 4594 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4596 |   using Shape_MNK = Shape<_64,_192,_32>;
 4597 |   using ThrID   = Layout<_128>;
 4598 |   using ALayout = GMMA::ALayout_64x32;
 4599 |   using BLayout = GMMA::ABLayout<192, 32>;
 4600 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4602-4603

```text
 4602 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4603 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4605-4616

```text
 4605 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4608 | using SM90_64x192x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32U8S8_RS_TN_SATURATE;
 4610 | template <>
 4611 | struct MMA_Traits<SM90_64x192x32_S32U8S8_RS_TN_SATURATE>
 4612 | {
 4613 |   using ValTypeD = int32_t;
 4614 |   using ValTypeA = uint8_t;
 4615 |   using ValTypeB = int8_t;
 4616 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4618-4624

```text
 4618 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4620 |   using Shape_MNK = Shape<_64,_192,_32>;
 4621 |   using ThrID   = Layout<_128>;
 4622 |   using ALayout = GMMA::ALayout_64x32;
 4623 |   using BLayout = GMMA::ABLayout<192, 32>;
 4624 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4626-4627

```text
 4626 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4627 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4629-4640

```text
 4629 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4632 | using SM90_64x256x32_S32U8S8_RS_TN = SM90::GMMA::MMA_64x256x32_S32U8S8_RS_TN;
 4634 | template <>
 4635 | struct MMA_Traits<SM90_64x256x32_S32U8S8_RS_TN>
 4636 | {
 4637 |   using ValTypeD = int32_t;
 4638 |   using ValTypeA = uint8_t;
 4639 |   using ValTypeB = int8_t;
 4640 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8S8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8S8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4642-4648

```text
 4642 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4644 |   using Shape_MNK = Shape<_64,_256,_32>;
 4645 |   using ThrID   = Layout<_128>;
 4646 |   using ALayout = GMMA::ALayout_64x32;
 4647 |   using BLayout = GMMA::ABLayout<256, 32>;
 4648 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4650-4651

```text
 4650 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4651 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4653-4664

```text
 4653 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4656 | using SM90_64x256x32_S32U8S8_RS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32U8S8_RS_TN_SATURATE;
 4658 | template <>
 4659 | struct MMA_Traits<SM90_64x256x32_S32U8S8_RS_TN_SATURATE>
 4660 | {
 4661 |   using ValTypeD = int32_t;
 4662 |   using ValTypeA = uint8_t;
 4663 |   using ValTypeB = int8_t;
 4664 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8S8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8S8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4666-4672

```text
 4666 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4668 |   using Shape_MNK = Shape<_64,_256,_32>;
 4669 |   using ThrID   = Layout<_128>;
 4670 |   using ALayout = GMMA::ALayout_64x32;
 4671 |   using BLayout = GMMA::ABLayout<256, 32>;
 4672 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4674-4675

```text
 4674 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4675 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4677-4688

```text
 4677 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4680 | using SM90_64x8x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x8x32_S32U8U8_SS_TN;
 4682 | template <>
 4683 | struct MMA_Traits<SM90_64x8x32_S32U8U8_SS_TN>
 4684 | {
 4685 |   using ValTypeD = int32_t;
 4686 |   using ValTypeA = uint8_t;
 4687 |   using ValTypeB = uint8_t;
 4688 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4690-4697

```text
 4690 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4691 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4693 |   using Shape_MNK = Shape<_64,_8,_32>;
 4694 |   using ThrID   = Layout<_128>;
 4695 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4696 |   using BLayout = GMMA::ABLayout<  8, 32>;
 4697 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4699-4700

```text
 4699 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4700 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4702-4713

```text
 4702 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4705 | using SM90_64x8x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32U8U8_SS_TN_SATURATE;
 4707 | template <>
 4708 | struct MMA_Traits<SM90_64x8x32_S32U8U8_SS_TN_SATURATE>
 4709 | {
 4710 |   using ValTypeD = int32_t;
 4711 |   using ValTypeA = uint8_t;
 4712 |   using ValTypeB = uint8_t;
 4713 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4715-4722

```text
 4715 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4716 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4718 |   using Shape_MNK = Shape<_64,_8,_32>;
 4719 |   using ThrID   = Layout<_128>;
 4720 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4721 |   using BLayout = GMMA::ABLayout<  8, 32>;
 4722 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4724-4725

```text
 4724 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4725 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4727-4738

```text
 4727 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4730 | using SM90_64x16x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x16x32_S32U8U8_SS_TN;
 4732 | template <>
 4733 | struct MMA_Traits<SM90_64x16x32_S32U8U8_SS_TN>
 4734 | {
 4735 |   using ValTypeD = int32_t;
 4736 |   using ValTypeA = uint8_t;
 4737 |   using ValTypeB = uint8_t;
 4738 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4740-4747

```text
 4740 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4741 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4743 |   using Shape_MNK = Shape<_64,_16,_32>;
 4744 |   using ThrID   = Layout<_128>;
 4745 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4746 |   using BLayout = GMMA::ABLayout< 16, 32>;
 4747 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4749-4750

```text
 4749 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4750 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4752-4763

```text
 4752 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4755 | using SM90_64x16x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32U8U8_SS_TN_SATURATE;
 4757 | template <>
 4758 | struct MMA_Traits<SM90_64x16x32_S32U8U8_SS_TN_SATURATE>
 4759 | {
 4760 |   using ValTypeD = int32_t;
 4761 |   using ValTypeA = uint8_t;
 4762 |   using ValTypeB = uint8_t;
 4763 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4765-4772

```text
 4765 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4766 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4768 |   using Shape_MNK = Shape<_64,_16,_32>;
 4769 |   using ThrID   = Layout<_128>;
 4770 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4771 |   using BLayout = GMMA::ABLayout< 16, 32>;
 4772 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4774-4775

```text
 4774 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4775 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4777-4788

```text
 4777 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4780 | using SM90_64x32x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x32x32_S32U8U8_SS_TN;
 4782 | template <>
 4783 | struct MMA_Traits<SM90_64x32x32_S32U8U8_SS_TN>
 4784 | {
 4785 |   using ValTypeD = int32_t;
 4786 |   using ValTypeA = uint8_t;
 4787 |   using ValTypeB = uint8_t;
 4788 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4790-4797

```text
 4790 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4791 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4793 |   using Shape_MNK = Shape<_64,_32,_32>;
 4794 |   using ThrID   = Layout<_128>;
 4795 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4796 |   using BLayout = GMMA::ABLayout< 32, 32>;
 4797 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4799-4800

```text
 4799 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4800 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4802-4813

```text
 4802 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4805 | using SM90_64x32x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32U8U8_SS_TN_SATURATE;
 4807 | template <>
 4808 | struct MMA_Traits<SM90_64x32x32_S32U8U8_SS_TN_SATURATE>
 4809 | {
 4810 |   using ValTypeD = int32_t;
 4811 |   using ValTypeA = uint8_t;
 4812 |   using ValTypeB = uint8_t;
 4813 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4815-4822

```text
 4815 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4816 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4818 |   using Shape_MNK = Shape<_64,_32,_32>;
 4819 |   using ThrID   = Layout<_128>;
 4820 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4821 |   using BLayout = GMMA::ABLayout< 32, 32>;
 4822 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4824-4825

```text
 4824 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4825 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4827-4838

```text
 4827 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4830 | using SM90_64x64x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x64x32_S32U8U8_SS_TN;
 4832 | template <>
 4833 | struct MMA_Traits<SM90_64x64x32_S32U8U8_SS_TN>
 4834 | {
 4835 |   using ValTypeD = int32_t;
 4836 |   using ValTypeA = uint8_t;
 4837 |   using ValTypeB = uint8_t;
 4838 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4840-4847

```text
 4840 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4841 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4843 |   using Shape_MNK = Shape<_64,_64,_32>;
 4844 |   using ThrID   = Layout<_128>;
 4845 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4846 |   using BLayout = GMMA::ABLayout< 64, 32>;
 4847 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4849-4850

```text
 4849 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4850 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4852-4863

```text
 4852 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4855 | using SM90_64x64x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32U8U8_SS_TN_SATURATE;
 4857 | template <>
 4858 | struct MMA_Traits<SM90_64x64x32_S32U8U8_SS_TN_SATURATE>
 4859 | {
 4860 |   using ValTypeD = int32_t;
 4861 |   using ValTypeA = uint8_t;
 4862 |   using ValTypeB = uint8_t;
 4863 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4865-4872

```text
 4865 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4866 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4868 |   using Shape_MNK = Shape<_64,_64,_32>;
 4869 |   using ThrID   = Layout<_128>;
 4870 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4871 |   using BLayout = GMMA::ABLayout< 64, 32>;
 4872 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4874-4875

```text
 4874 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4875 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4877-4888

```text
 4877 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4880 | using SM90_64x96x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x96x32_S32U8U8_SS_TN;
 4882 | template <>
 4883 | struct MMA_Traits<SM90_64x96x32_S32U8U8_SS_TN>
 4884 | {
 4885 |   using ValTypeD = int32_t;
 4886 |   using ValTypeA = uint8_t;
 4887 |   using ValTypeB = uint8_t;
 4888 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4890-4897

```text
 4890 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4891 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4893 |   using Shape_MNK = Shape<_64,_96,_32>;
 4894 |   using ThrID   = Layout<_128>;
 4895 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4896 |   using BLayout = GMMA::ABLayout< 96, 32>;
 4897 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4899-4900

```text
 4899 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4900 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4902-4913

```text
 4902 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4905 | using SM90_64x96x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32U8U8_SS_TN_SATURATE;
 4907 | template <>
 4908 | struct MMA_Traits<SM90_64x96x32_S32U8U8_SS_TN_SATURATE>
 4909 | {
 4910 |   using ValTypeD = int32_t;
 4911 |   using ValTypeA = uint8_t;
 4912 |   using ValTypeB = uint8_t;
 4913 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4915-4922

```text
 4915 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4916 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4918 |   using Shape_MNK = Shape<_64,_96,_32>;
 4919 |   using ThrID   = Layout<_128>;
 4920 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4921 |   using BLayout = GMMA::ABLayout< 96, 32>;
 4922 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4924-4925

```text
 4924 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4925 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4927-4938

```text
 4927 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4930 | using SM90_64x128x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x128x32_S32U8U8_SS_TN;
 4932 | template <>
 4933 | struct MMA_Traits<SM90_64x128x32_S32U8U8_SS_TN>
 4934 | {
 4935 |   using ValTypeD = int32_t;
 4936 |   using ValTypeA = uint8_t;
 4937 |   using ValTypeB = uint8_t;
 4938 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4940-4947

```text
 4940 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4941 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4943 |   using Shape_MNK = Shape<_64,_128,_32>;
 4944 |   using ThrID   = Layout<_128>;
 4945 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4946 |   using BLayout = GMMA::ABLayout<128, 32>;
 4947 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4949-4950

```text
 4949 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4950 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4952-4963

```text
 4952 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4955 | using SM90_64x128x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32U8U8_SS_TN_SATURATE;
 4957 | template <>
 4958 | struct MMA_Traits<SM90_64x128x32_S32U8U8_SS_TN_SATURATE>
 4959 | {
 4960 |   using ValTypeD = int32_t;
 4961 |   using ValTypeA = uint8_t;
 4962 |   using ValTypeB = uint8_t;
 4963 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4965-4972

```text
 4965 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4966 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4968 |   using Shape_MNK = Shape<_64,_128,_32>;
 4969 |   using ThrID   = Layout<_128>;
 4970 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4971 |   using BLayout = GMMA::ABLayout<128, 32>;
 4972 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4974-4975

```text
 4974 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 4975 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4977-4988

```text
 4977 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 4980 | using SM90_64x192x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x192x32_S32U8U8_SS_TN;
 4982 | template <>
 4983 | struct MMA_Traits<SM90_64x192x32_S32U8U8_SS_TN>
 4984 | {
 4985 |   using ValTypeD = int32_t;
 4986 |   using ValTypeA = uint8_t;
 4987 |   using ValTypeB = uint8_t;
 4988 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 4990-4997

```text
 4990 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 4991 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 4993 |   using Shape_MNK = Shape<_64,_192,_32>;
 4994 |   using ThrID   = Layout<_128>;
 4995 |   using ALayout = GMMA::ABLayout< 64, 32>;
 4996 |   using BLayout = GMMA::ABLayout<192, 32>;
 4997 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 4999-5000

```text
 4999 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5000 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5002-5013

```text
 5002 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5005 | using SM90_64x192x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32U8U8_SS_TN_SATURATE;
 5007 | template <>
 5008 | struct MMA_Traits<SM90_64x192x32_S32U8U8_SS_TN_SATURATE>
 5009 | {
 5010 |   using ValTypeD = int32_t;
 5011 |   using ValTypeA = uint8_t;
 5012 |   using ValTypeB = uint8_t;
 5013 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5015-5022

```text
 5015 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5016 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5018 |   using Shape_MNK = Shape<_64,_192,_32>;
 5019 |   using ThrID   = Layout<_128>;
 5020 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5021 |   using BLayout = GMMA::ABLayout<192, 32>;
 5022 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5024-5025

```text
 5024 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5025 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5027-5038

```text
 5027 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5030 | using SM90_64x256x32_S32U8U8_SS_TN = SM90::GMMA::MMA_64x256x32_S32U8U8_SS_TN;
 5032 | template <>
 5033 | struct MMA_Traits<SM90_64x256x32_S32U8U8_SS_TN>
 5034 | {
 5035 |   using ValTypeD = int32_t;
 5036 |   using ValTypeA = uint8_t;
 5037 |   using ValTypeB = uint8_t;
 5038 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8U8_SS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8U8_SS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5040-5047

```text
 5040 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5041 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5043 |   using Shape_MNK = Shape<_64,_256,_32>;
 5044 |   using ThrID   = Layout<_128>;
 5045 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5046 |   using BLayout = GMMA::ABLayout<256, 32>;
 5047 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5049-5050

```text
 5049 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5050 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5052-5063

```text
 5052 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5055 | using SM90_64x256x32_S32U8U8_SS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32U8U8_SS_TN_SATURATE;
 5057 | template <>
 5058 | struct MMA_Traits<SM90_64x256x32_S32U8U8_SS_TN_SATURATE>
 5059 | {
 5060 |   using ValTypeD = int32_t;
 5061 |   using ValTypeA = uint8_t;
 5062 |   using ValTypeB = uint8_t;
 5063 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8U8_SS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8U8_SS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5065-5072

```text
 5065 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5066 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5068 |   using Shape_MNK = Shape<_64,_256,_32>;
 5069 |   using ThrID   = Layout<_128>;
 5070 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5071 |   using BLayout = GMMA::ABLayout<256, 32>;
 5072 |   using CLayout = GMMA::CLayout_64x256;
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

### Lines 5077-5088

```text
 5077 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5080 | using SM90_64x8x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x8x32_S32U8U8_RS_TN;
 5082 | template <>
 5083 | struct MMA_Traits<SM90_64x8x32_S32U8U8_RS_TN>
 5084 | {
 5085 |   using ValTypeD = int32_t;
 5086 |   using ValTypeA = uint8_t;
 5087 |   using ValTypeB = uint8_t;
 5088 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5090-5096

```text
 5090 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5092 |   using Shape_MNK = Shape<_64,_8,_32>;
 5093 |   using ThrID   = Layout<_128>;
 5094 |   using ALayout = GMMA::ALayout_64x32;
 5095 |   using BLayout = GMMA::ABLayout<  8, 32>;
 5096 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5098-5099

```text
 5098 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5099 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5101-5112

```text
 5101 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5104 | using SM90_64x8x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x8x32_S32U8U8_RS_TN_SATURATE;
 5106 | template <>
 5107 | struct MMA_Traits<SM90_64x8x32_S32U8U8_RS_TN_SATURATE>
 5108 | {
 5109 |   using ValTypeD = int32_t;
 5110 |   using ValTypeA = uint8_t;
 5111 |   using ValTypeB = uint8_t;
 5112 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5114-5120

```text
 5114 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5116 |   using Shape_MNK = Shape<_64,_8,_32>;
 5117 |   using ThrID   = Layout<_128>;
 5118 |   using ALayout = GMMA::ALayout_64x32;
 5119 |   using BLayout = GMMA::ABLayout<  8, 32>;
 5120 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5122-5123

```text
 5122 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5123 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5125-5136

```text
 5125 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5128 | using SM90_64x16x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x16x32_S32U8U8_RS_TN;
 5130 | template <>
 5131 | struct MMA_Traits<SM90_64x16x32_S32U8U8_RS_TN>
 5132 | {
 5133 |   using ValTypeD = int32_t;
 5134 |   using ValTypeA = uint8_t;
 5135 |   using ValTypeB = uint8_t;
 5136 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5138-5144

```text
 5138 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5140 |   using Shape_MNK = Shape<_64,_16,_32>;
 5141 |   using ThrID   = Layout<_128>;
 5142 |   using ALayout = GMMA::ALayout_64x32;
 5143 |   using BLayout = GMMA::ABLayout< 16, 32>;
 5144 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5146-5147

```text
 5146 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5147 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5149-5160

```text
 5149 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5152 | using SM90_64x16x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x16x32_S32U8U8_RS_TN_SATURATE;
 5154 | template <>
 5155 | struct MMA_Traits<SM90_64x16x32_S32U8U8_RS_TN_SATURATE>
 5156 | {
 5157 |   using ValTypeD = int32_t;
 5158 |   using ValTypeA = uint8_t;
 5159 |   using ValTypeB = uint8_t;
 5160 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5162-5168

```text
 5162 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5164 |   using Shape_MNK = Shape<_64,_16,_32>;
 5165 |   using ThrID   = Layout<_128>;
 5166 |   using ALayout = GMMA::ALayout_64x32;
 5167 |   using BLayout = GMMA::ABLayout< 16, 32>;
 5168 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5170-5171

```text
 5170 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5171 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5173-5184

```text
 5173 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5176 | using SM90_64x32x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x32x32_S32U8U8_RS_TN;
 5178 | template <>
 5179 | struct MMA_Traits<SM90_64x32x32_S32U8U8_RS_TN>
 5180 | {
 5181 |   using ValTypeD = int32_t;
 5182 |   using ValTypeA = uint8_t;
 5183 |   using ValTypeB = uint8_t;
 5184 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5186-5192

```text
 5186 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5188 |   using Shape_MNK = Shape<_64,_32,_32>;
 5189 |   using ThrID   = Layout<_128>;
 5190 |   using ALayout = GMMA::ALayout_64x32;
 5191 |   using BLayout = GMMA::ABLayout< 32, 32>;
 5192 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5194-5195

```text
 5194 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5195 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5197-5208

```text
 5197 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5200 | using SM90_64x32x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x32x32_S32U8U8_RS_TN_SATURATE;
 5202 | template <>
 5203 | struct MMA_Traits<SM90_64x32x32_S32U8U8_RS_TN_SATURATE>
 5204 | {
 5205 |   using ValTypeD = int32_t;
 5206 |   using ValTypeA = uint8_t;
 5207 |   using ValTypeB = uint8_t;
 5208 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5210-5216

```text
 5210 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5212 |   using Shape_MNK = Shape<_64,_32,_32>;
 5213 |   using ThrID   = Layout<_128>;
 5214 |   using ALayout = GMMA::ALayout_64x32;
 5215 |   using BLayout = GMMA::ABLayout< 32, 32>;
 5216 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5218-5219

```text
 5218 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5219 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5221-5232

```text
 5221 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5224 | using SM90_64x64x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x64x32_S32U8U8_RS_TN;
 5226 | template <>
 5227 | struct MMA_Traits<SM90_64x64x32_S32U8U8_RS_TN>
 5228 | {
 5229 |   using ValTypeD = int32_t;
 5230 |   using ValTypeA = uint8_t;
 5231 |   using ValTypeB = uint8_t;
 5232 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5234-5240

```text
 5234 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5236 |   using Shape_MNK = Shape<_64,_64,_32>;
 5237 |   using ThrID   = Layout<_128>;
 5238 |   using ALayout = GMMA::ALayout_64x32;
 5239 |   using BLayout = GMMA::ABLayout< 64, 32>;
 5240 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5242-5243

```text
 5242 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5243 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5245-5256

```text
 5245 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5248 | using SM90_64x64x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x64x32_S32U8U8_RS_TN_SATURATE;
 5250 | template <>
 5251 | struct MMA_Traits<SM90_64x64x32_S32U8U8_RS_TN_SATURATE>
 5252 | {
 5253 |   using ValTypeD = int32_t;
 5254 |   using ValTypeA = uint8_t;
 5255 |   using ValTypeB = uint8_t;
 5256 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5258-5264

```text
 5258 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5260 |   using Shape_MNK = Shape<_64,_64,_32>;
 5261 |   using ThrID   = Layout<_128>;
 5262 |   using ALayout = GMMA::ALayout_64x32;
 5263 |   using BLayout = GMMA::ABLayout< 64, 32>;
 5264 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5266-5267

```text
 5266 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5267 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5269-5280

```text
 5269 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5272 | using SM90_64x96x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x96x32_S32U8U8_RS_TN;
 5274 | template <>
 5275 | struct MMA_Traits<SM90_64x96x32_S32U8U8_RS_TN>
 5276 | {
 5277 |   using ValTypeD = int32_t;
 5278 |   using ValTypeA = uint8_t;
 5279 |   using ValTypeB = uint8_t;
 5280 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5282-5288

```text
 5282 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5284 |   using Shape_MNK = Shape<_64,_96,_32>;
 5285 |   using ThrID   = Layout<_128>;
 5286 |   using ALayout = GMMA::ALayout_64x32;
 5287 |   using BLayout = GMMA::ABLayout< 96, 32>;
 5288 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5290-5291

```text
 5290 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5291 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5293-5304

```text
 5293 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5296 | using SM90_64x96x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x96x32_S32U8U8_RS_TN_SATURATE;
 5298 | template <>
 5299 | struct MMA_Traits<SM90_64x96x32_S32U8U8_RS_TN_SATURATE>
 5300 | {
 5301 |   using ValTypeD = int32_t;
 5302 |   using ValTypeA = uint8_t;
 5303 |   using ValTypeB = uint8_t;
 5304 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5306-5312

```text
 5306 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5308 |   using Shape_MNK = Shape<_64,_96,_32>;
 5309 |   using ThrID   = Layout<_128>;
 5310 |   using ALayout = GMMA::ALayout_64x32;
 5311 |   using BLayout = GMMA::ABLayout< 96, 32>;
 5312 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5314-5315

```text
 5314 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5315 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5317-5328

```text
 5317 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5320 | using SM90_64x128x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x128x32_S32U8U8_RS_TN;
 5322 | template <>
 5323 | struct MMA_Traits<SM90_64x128x32_S32U8U8_RS_TN>
 5324 | {
 5325 |   using ValTypeD = int32_t;
 5326 |   using ValTypeA = uint8_t;
 5327 |   using ValTypeB = uint8_t;
 5328 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5330-5336

```text
 5330 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5332 |   using Shape_MNK = Shape<_64,_128,_32>;
 5333 |   using ThrID   = Layout<_128>;
 5334 |   using ALayout = GMMA::ALayout_64x32;
 5335 |   using BLayout = GMMA::ABLayout<128, 32>;
 5336 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5338-5339

```text
 5338 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5339 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5341-5352

```text
 5341 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5344 | using SM90_64x128x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x128x32_S32U8U8_RS_TN_SATURATE;
 5346 | template <>
 5347 | struct MMA_Traits<SM90_64x128x32_S32U8U8_RS_TN_SATURATE>
 5348 | {
 5349 |   using ValTypeD = int32_t;
 5350 |   using ValTypeA = uint8_t;
 5351 |   using ValTypeB = uint8_t;
 5352 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5354-5360

```text
 5354 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5356 |   using Shape_MNK = Shape<_64,_128,_32>;
 5357 |   using ThrID   = Layout<_128>;
 5358 |   using ALayout = GMMA::ALayout_64x32;
 5359 |   using BLayout = GMMA::ABLayout<128, 32>;
 5360 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5362-5363

```text
 5362 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5363 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5365-5376

```text
 5365 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5368 | using SM90_64x192x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x192x32_S32U8U8_RS_TN;
 5370 | template <>
 5371 | struct MMA_Traits<SM90_64x192x32_S32U8U8_RS_TN>
 5372 | {
 5373 |   using ValTypeD = int32_t;
 5374 |   using ValTypeA = uint8_t;
 5375 |   using ValTypeB = uint8_t;
 5376 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5378-5384

```text
 5378 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5380 |   using Shape_MNK = Shape<_64,_192,_32>;
 5381 |   using ThrID   = Layout<_128>;
 5382 |   using ALayout = GMMA::ALayout_64x32;
 5383 |   using BLayout = GMMA::ABLayout<192, 32>;
 5384 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5386-5387

```text
 5386 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5387 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5389-5400

```text
 5389 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5392 | using SM90_64x192x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x192x32_S32U8U8_RS_TN_SATURATE;
 5394 | template <>
 5395 | struct MMA_Traits<SM90_64x192x32_S32U8U8_RS_TN_SATURATE>
 5396 | {
 5397 |   using ValTypeD = int32_t;
 5398 |   using ValTypeA = uint8_t;
 5399 |   using ValTypeB = uint8_t;
 5400 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5402-5408

```text
 5402 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5404 |   using Shape_MNK = Shape<_64,_192,_32>;
 5405 |   using ThrID   = Layout<_128>;
 5406 |   using ALayout = GMMA::ALayout_64x32;
 5407 |   using BLayout = GMMA::ABLayout<192, 32>;
 5408 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5410-5411

```text
 5410 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5411 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5413-5424

```text
 5413 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5416 | using SM90_64x256x32_S32U8U8_RS_TN = SM90::GMMA::MMA_64x256x32_S32U8U8_RS_TN;
 5418 | template <>
 5419 | struct MMA_Traits<SM90_64x256x32_S32U8U8_RS_TN>
 5420 | {
 5421 |   using ValTypeD = int32_t;
 5422 |   using ValTypeA = uint8_t;
 5423 |   using ValTypeB = uint8_t;
 5424 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8U8_RS_TN`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8U8_RS_TN` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5426-5432

```text
 5426 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5428 |   using Shape_MNK = Shape<_64,_256,_32>;
 5429 |   using ThrID   = Layout<_128>;
 5430 |   using ALayout = GMMA::ALayout_64x32;
 5431 |   using BLayout = GMMA::ABLayout<256, 32>;
 5432 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5434-5435

```text
 5434 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5435 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5437-5448

```text
 5437 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5440 | using SM90_64x256x32_S32U8U8_RS_TN_SATURATE = SM90::GMMA::MMA_64x256x32_S32U8U8_RS_TN_SATURATE;
 5442 | template <>
 5443 | struct MMA_Traits<SM90_64x256x32_S32U8U8_RS_TN_SATURATE>
 5444 | {
 5445 |   using ValTypeD = int32_t;
 5446 |   using ValTypeA = uint8_t;
 5447 |   using ValTypeB = uint8_t;
 5448 |   using ValTypeC = int32_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_S32U8U8_RS_TN_SATURATE`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_S32U8U8_RS_TN_SATURATE` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5450-5456

```text
 5450 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5452 |   using Shape_MNK = Shape<_64,_256,_32>;
 5453 |   using ThrID   = Layout<_128>;
 5454 |   using ALayout = GMMA::ALayout_64x32;
 5455 |   using BLayout = GMMA::ABLayout<256, 32>;
 5456 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5458-5459

```text
 5458 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5459 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5461-5475

```text
 5461 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5463 | template <
 5464 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5465 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5466 | >
 5467 | using SM90_64x8x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x8x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 5469 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5470 | struct MMA_Traits<SM90_64x8x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 5471 | {
 5472 |   using ValTypeD = half_t;
 5473 |   using ValTypeA = float_e4m3_t;
 5474 |   using ValTypeB = float_e4m3_t;
 5475 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5477-5484

```text
 5477 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5478 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5480 |   using Shape_MNK = Shape<_64,_8,_32>;
 5481 |   using ThrID   = Layout<_128>;
 5482 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5483 |   using BLayout = GMMA::ABLayout<  8, 32>;
 5484 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5486-5487

```text
 5486 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5487 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5489-5503

```text
 5489 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5491 | template <
 5492 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5493 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5494 | >
 5495 | using SM90_64x8x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x8x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 5497 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5498 | struct MMA_Traits<SM90_64x8x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 5499 | {
 5500 |   using ValTypeD = half_t;
 5501 |   using ValTypeA = float_e4m3_t;
 5502 |   using ValTypeB = float_e4m3_t;
 5503 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5505-5511

```text
 5505 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5507 |   using Shape_MNK = Shape<_64,_8,_32>;
 5508 |   using ThrID   = Layout<_128>;
 5509 |   using ALayout = GMMA::ALayout_64x32;
 5510 |   using BLayout = GMMA::ABLayout<  8, 32>;
 5511 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5513-5514

```text
 5513 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5514 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5516-5530

```text
 5516 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5518 | template <
 5519 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5520 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5521 | >
 5522 | using SM90_64x8x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x8x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 5524 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5525 | struct MMA_Traits<SM90_64x8x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 5526 | {
 5527 |   using ValTypeD = float;
 5528 |   using ValTypeA = float_e4m3_t;
 5529 |   using ValTypeB = float_e4m3_t;
 5530 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5532-5539

```text
 5532 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5533 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5535 |   using Shape_MNK = Shape<_64,_8,_32>;
 5536 |   using ThrID   = Layout<_128>;
 5537 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5538 |   using BLayout = GMMA::ABLayout<  8, 32>;
 5539 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5541-5542

```text
 5541 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5542 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5544-5558

```text
 5544 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5546 | template <
 5547 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5548 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5549 | >
 5550 | using SM90_64x8x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x8x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 5552 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5553 | struct MMA_Traits<SM90_64x8x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 5554 | {
 5555 |   using ValTypeD = float;
 5556 |   using ValTypeA = float_e4m3_t;
 5557 |   using ValTypeB = float_e4m3_t;
 5558 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5560-5566

```text
 5560 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5562 |   using Shape_MNK = Shape<_64,_8,_32>;
 5563 |   using ThrID   = Layout<_128>;
 5564 |   using ALayout = GMMA::ALayout_64x32;
 5565 |   using BLayout = GMMA::ABLayout<  8, 32>;
 5566 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5568-5569

```text
 5568 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5569 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5571-5585

```text
 5571 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5573 | template <
 5574 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5575 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5576 | >
 5577 | using SM90_64x16x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x16x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 5579 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5580 | struct MMA_Traits<SM90_64x16x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 5581 | {
 5582 |   using ValTypeD = half_t;
 5583 |   using ValTypeA = float_e4m3_t;
 5584 |   using ValTypeB = float_e4m3_t;
 5585 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5587-5594

```text
 5587 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5588 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5590 |   using Shape_MNK = Shape<_64,_16,_32>;
 5591 |   using ThrID   = Layout<_128>;
 5592 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5593 |   using BLayout = GMMA::ABLayout< 16, 32>;
 5594 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5596-5597

```text
 5596 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5597 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5599-5613

```text
 5599 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5601 | template <
 5602 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5603 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5604 | >
 5605 | using SM90_64x16x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x16x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 5607 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5608 | struct MMA_Traits<SM90_64x16x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 5609 | {
 5610 |   using ValTypeD = half_t;
 5611 |   using ValTypeA = float_e4m3_t;
 5612 |   using ValTypeB = float_e4m3_t;
 5613 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5615-5621

```text
 5615 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5617 |   using Shape_MNK = Shape<_64,_16,_32>;
 5618 |   using ThrID   = Layout<_128>;
 5619 |   using ALayout = GMMA::ALayout_64x32;
 5620 |   using BLayout = GMMA::ABLayout< 16, 32>;
 5621 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5623-5624

```text
 5623 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5624 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5626-5640

```text
 5626 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5628 | template <
 5629 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5630 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5631 | >
 5632 | using SM90_64x16x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x16x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 5634 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5635 | struct MMA_Traits<SM90_64x16x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 5636 | {
 5637 |   using ValTypeD = float;
 5638 |   using ValTypeA = float_e4m3_t;
 5639 |   using ValTypeB = float_e4m3_t;
 5640 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5642-5649

```text
 5642 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5643 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5645 |   using Shape_MNK = Shape<_64,_16,_32>;
 5646 |   using ThrID   = Layout<_128>;
 5647 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5648 |   using BLayout = GMMA::ABLayout< 16, 32>;
 5649 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5651-5652

```text
 5651 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5652 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5654-5668

```text
 5654 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5656 | template <
 5657 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5658 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5659 | >
 5660 | using SM90_64x16x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x16x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 5662 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5663 | struct MMA_Traits<SM90_64x16x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 5664 | {
 5665 |   using ValTypeD = float;
 5666 |   using ValTypeA = float_e4m3_t;
 5667 |   using ValTypeB = float_e4m3_t;
 5668 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5670-5676

```text
 5670 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5672 |   using Shape_MNK = Shape<_64,_16,_32>;
 5673 |   using ThrID   = Layout<_128>;
 5674 |   using ALayout = GMMA::ALayout_64x32;
 5675 |   using BLayout = GMMA::ABLayout< 16, 32>;
 5676 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5678-5679

```text
 5678 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5679 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5681-5695

```text
 5681 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5683 | template <
 5684 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5685 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5686 | >
 5687 | using SM90_64x32x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x32x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 5689 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5690 | struct MMA_Traits<SM90_64x32x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 5691 | {
 5692 |   using ValTypeD = half_t;
 5693 |   using ValTypeA = float_e4m3_t;
 5694 |   using ValTypeB = float_e4m3_t;
 5695 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5697-5704

```text
 5697 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5698 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5700 |   using Shape_MNK = Shape<_64,_32,_32>;
 5701 |   using ThrID   = Layout<_128>;
 5702 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5703 |   using BLayout = GMMA::ABLayout< 32, 32>;
 5704 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5706-5707

```text
 5706 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5707 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5709-5723

```text
 5709 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5711 | template <
 5712 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5713 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5714 | >
 5715 | using SM90_64x32x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x32x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 5717 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5718 | struct MMA_Traits<SM90_64x32x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 5719 | {
 5720 |   using ValTypeD = half_t;
 5721 |   using ValTypeA = float_e4m3_t;
 5722 |   using ValTypeB = float_e4m3_t;
 5723 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5725-5731

```text
 5725 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5727 |   using Shape_MNK = Shape<_64,_32,_32>;
 5728 |   using ThrID   = Layout<_128>;
 5729 |   using ALayout = GMMA::ALayout_64x32;
 5730 |   using BLayout = GMMA::ABLayout< 32, 32>;
 5731 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5733-5734

```text
 5733 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5734 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5736-5750

```text
 5736 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5738 | template <
 5739 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5740 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5741 | >
 5742 | using SM90_64x32x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x32x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 5744 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5745 | struct MMA_Traits<SM90_64x32x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 5746 | {
 5747 |   using ValTypeD = float;
 5748 |   using ValTypeA = float_e4m3_t;
 5749 |   using ValTypeB = float_e4m3_t;
 5750 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5752-5759

```text
 5752 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5753 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5755 |   using Shape_MNK = Shape<_64,_32,_32>;
 5756 |   using ThrID   = Layout<_128>;
 5757 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5758 |   using BLayout = GMMA::ABLayout< 32, 32>;
 5759 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5761-5762

```text
 5761 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5762 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5764-5778

```text
 5764 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5766 | template <
 5767 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5768 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5769 | >
 5770 | using SM90_64x32x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x32x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 5772 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5773 | struct MMA_Traits<SM90_64x32x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 5774 | {
 5775 |   using ValTypeD = float;
 5776 |   using ValTypeA = float_e4m3_t;
 5777 |   using ValTypeB = float_e4m3_t;
 5778 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5780-5786

```text
 5780 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5782 |   using Shape_MNK = Shape<_64,_32,_32>;
 5783 |   using ThrID   = Layout<_128>;
 5784 |   using ALayout = GMMA::ALayout_64x32;
 5785 |   using BLayout = GMMA::ABLayout< 32, 32>;
 5786 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5788-5789

```text
 5788 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5789 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5791-5805

```text
 5791 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5793 | template <
 5794 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5795 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5796 | >
 5797 | using SM90_64x64x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x64x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 5799 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5800 | struct MMA_Traits<SM90_64x64x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 5801 | {
 5802 |   using ValTypeD = half_t;
 5803 |   using ValTypeA = float_e4m3_t;
 5804 |   using ValTypeB = float_e4m3_t;
 5805 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5807-5814

```text
 5807 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5808 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5810 |   using Shape_MNK = Shape<_64,_64,_32>;
 5811 |   using ThrID   = Layout<_128>;
 5812 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5813 |   using BLayout = GMMA::ABLayout< 64, 32>;
 5814 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5816-5817

```text
 5816 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5817 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5819-5833

```text
 5819 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5821 | template <
 5822 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5823 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5824 | >
 5825 | using SM90_64x64x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x64x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 5827 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5828 | struct MMA_Traits<SM90_64x64x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 5829 | {
 5830 |   using ValTypeD = half_t;
 5831 |   using ValTypeA = float_e4m3_t;
 5832 |   using ValTypeB = float_e4m3_t;
 5833 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5835-5841

```text
 5835 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5837 |   using Shape_MNK = Shape<_64,_64,_32>;
 5838 |   using ThrID   = Layout<_128>;
 5839 |   using ALayout = GMMA::ALayout_64x32;
 5840 |   using BLayout = GMMA::ABLayout< 64, 32>;
 5841 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5843-5844

```text
 5843 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5844 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5846-5860

```text
 5846 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5848 | template <
 5849 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5850 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5851 | >
 5852 | using SM90_64x64x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x64x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 5854 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5855 | struct MMA_Traits<SM90_64x64x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 5856 | {
 5857 |   using ValTypeD = float;
 5858 |   using ValTypeA = float_e4m3_t;
 5859 |   using ValTypeB = float_e4m3_t;
 5860 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5862-5869

```text
 5862 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5863 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5865 |   using Shape_MNK = Shape<_64,_64,_32>;
 5866 |   using ThrID   = Layout<_128>;
 5867 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5868 |   using BLayout = GMMA::ABLayout< 64, 32>;
 5869 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5871-5872

```text
 5871 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5872 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5874-5888

```text
 5874 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5876 | template <
 5877 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5878 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5879 | >
 5880 | using SM90_64x64x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x64x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 5882 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5883 | struct MMA_Traits<SM90_64x64x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 5884 | {
 5885 |   using ValTypeD = float;
 5886 |   using ValTypeA = float_e4m3_t;
 5887 |   using ValTypeB = float_e4m3_t;
 5888 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5890-5896

```text
 5890 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5892 |   using Shape_MNK = Shape<_64,_64,_32>;
 5893 |   using ThrID   = Layout<_128>;
 5894 |   using ALayout = GMMA::ALayout_64x32;
 5895 |   using BLayout = GMMA::ABLayout< 64, 32>;
 5896 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5898-5899

```text
 5898 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5899 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5901-5915

```text
 5901 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5903 | template <
 5904 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5905 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5906 | >
 5907 | using SM90_64x96x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x96x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 5909 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5910 | struct MMA_Traits<SM90_64x96x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 5911 | {
 5912 |   using ValTypeD = half_t;
 5913 |   using ValTypeA = float_e4m3_t;
 5914 |   using ValTypeB = float_e4m3_t;
 5915 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5917-5924

```text
 5917 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5918 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5920 |   using Shape_MNK = Shape<_64,_96,_32>;
 5921 |   using ThrID   = Layout<_128>;
 5922 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5923 |   using BLayout = GMMA::ABLayout< 96, 32>;
 5924 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5926-5927

```text
 5926 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5927 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5929-5943

```text
 5929 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5931 | template <
 5932 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5933 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5934 | >
 5935 | using SM90_64x96x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x96x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 5937 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5938 | struct MMA_Traits<SM90_64x96x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 5939 | {
 5940 |   using ValTypeD = half_t;
 5941 |   using ValTypeA = float_e4m3_t;
 5942 |   using ValTypeB = float_e4m3_t;
 5943 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5945-5951

```text
 5945 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5947 |   using Shape_MNK = Shape<_64,_96,_32>;
 5948 |   using ThrID   = Layout<_128>;
 5949 |   using ALayout = GMMA::ALayout_64x32;
 5950 |   using BLayout = GMMA::ABLayout< 96, 32>;
 5951 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5953-5954

```text
 5953 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5954 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5956-5970

```text
 5956 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5958 | template <
 5959 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5960 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5961 | >
 5962 | using SM90_64x96x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x96x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 5964 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5965 | struct MMA_Traits<SM90_64x96x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 5966 | {
 5967 |   using ValTypeD = float;
 5968 |   using ValTypeA = float_e4m3_t;
 5969 |   using ValTypeB = float_e4m3_t;
 5970 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 5972-5979

```text
 5972 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 5973 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 5975 |   using Shape_MNK = Shape<_64,_96,_32>;
 5976 |   using ThrID   = Layout<_128>;
 5977 |   using ALayout = GMMA::ABLayout< 64, 32>;
 5978 |   using BLayout = GMMA::ABLayout< 96, 32>;
 5979 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5981-5982

```text
 5981 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 5982 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 5984-5998

```text
 5984 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 5986 | template <
 5987 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 5988 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 5989 | >
 5990 | using SM90_64x96x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x96x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 5992 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 5993 | struct MMA_Traits<SM90_64x96x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 5994 | {
 5995 |   using ValTypeD = float;
 5996 |   using ValTypeA = float_e4m3_t;
 5997 |   using ValTypeB = float_e4m3_t;
 5998 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6000-6006

```text
 6000 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6002 |   using Shape_MNK = Shape<_64,_96,_32>;
 6003 |   using ThrID   = Layout<_128>;
 6004 |   using ALayout = GMMA::ALayout_64x32;
 6005 |   using BLayout = GMMA::ABLayout< 96, 32>;
 6006 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6008-6009

```text
 6008 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6009 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6011-6025

```text
 6011 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6013 | template <
 6014 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6015 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6016 | >
 6017 | using SM90_64x128x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x128x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 6019 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6020 | struct MMA_Traits<SM90_64x128x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 6021 | {
 6022 |   using ValTypeD = half_t;
 6023 |   using ValTypeA = float_e4m3_t;
 6024 |   using ValTypeB = float_e4m3_t;
 6025 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6027-6034

```text
 6027 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6028 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6030 |   using Shape_MNK = Shape<_64,_128,_32>;
 6031 |   using ThrID   = Layout<_128>;
 6032 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6033 |   using BLayout = GMMA::ABLayout<128, 32>;
 6034 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6036-6037

```text
 6036 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6037 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6039-6053

```text
 6039 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6041 | template <
 6042 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6043 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6044 | >
 6045 | using SM90_64x128x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x128x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 6047 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6048 | struct MMA_Traits<SM90_64x128x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 6049 | {
 6050 |   using ValTypeD = half_t;
 6051 |   using ValTypeA = float_e4m3_t;
 6052 |   using ValTypeB = float_e4m3_t;
 6053 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6055-6061

```text
 6055 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6057 |   using Shape_MNK = Shape<_64,_128,_32>;
 6058 |   using ThrID   = Layout<_128>;
 6059 |   using ALayout = GMMA::ALayout_64x32;
 6060 |   using BLayout = GMMA::ABLayout<128, 32>;
 6061 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6063-6064

```text
 6063 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6064 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6066-6080

```text
 6066 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6068 | template <
 6069 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6070 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6071 | >
 6072 | using SM90_64x128x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x128x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 6074 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6075 | struct MMA_Traits<SM90_64x128x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 6076 | {
 6077 |   using ValTypeD = float;
 6078 |   using ValTypeA = float_e4m3_t;
 6079 |   using ValTypeB = float_e4m3_t;
 6080 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6082-6089

```text
 6082 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6083 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6085 |   using Shape_MNK = Shape<_64,_128,_32>;
 6086 |   using ThrID   = Layout<_128>;
 6087 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6088 |   using BLayout = GMMA::ABLayout<128, 32>;
 6089 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6091-6092

```text
 6091 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6092 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6094-6108

```text
 6094 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6096 | template <
 6097 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6098 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6099 | >
 6100 | using SM90_64x128x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x128x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 6102 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6103 | struct MMA_Traits<SM90_64x128x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 6104 | {
 6105 |   using ValTypeD = float;
 6106 |   using ValTypeA = float_e4m3_t;
 6107 |   using ValTypeB = float_e4m3_t;
 6108 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6110-6116

```text
 6110 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6112 |   using Shape_MNK = Shape<_64,_128,_32>;
 6113 |   using ThrID   = Layout<_128>;
 6114 |   using ALayout = GMMA::ALayout_64x32;
 6115 |   using BLayout = GMMA::ABLayout<128, 32>;
 6116 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6118-6119

```text
 6118 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6119 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6121-6135

```text
 6121 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6123 | template <
 6124 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6125 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6126 | >
 6127 | using SM90_64x192x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x192x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 6129 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6130 | struct MMA_Traits<SM90_64x192x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 6131 | {
 6132 |   using ValTypeD = half_t;
 6133 |   using ValTypeA = float_e4m3_t;
 6134 |   using ValTypeB = float_e4m3_t;
 6135 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6137-6144

```text
 6137 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6138 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6140 |   using Shape_MNK = Shape<_64,_192,_32>;
 6141 |   using ThrID   = Layout<_128>;
 6142 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6143 |   using BLayout = GMMA::ABLayout<192, 32>;
 6144 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6146-6147

```text
 6146 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6147 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6149-6163

```text
 6149 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6151 | template <
 6152 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6153 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6154 | >
 6155 | using SM90_64x192x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x192x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 6157 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6158 | struct MMA_Traits<SM90_64x192x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 6159 | {
 6160 |   using ValTypeD = half_t;
 6161 |   using ValTypeA = float_e4m3_t;
 6162 |   using ValTypeB = float_e4m3_t;
 6163 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6165-6171

```text
 6165 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6167 |   using Shape_MNK = Shape<_64,_192,_32>;
 6168 |   using ThrID   = Layout<_128>;
 6169 |   using ALayout = GMMA::ALayout_64x32;
 6170 |   using BLayout = GMMA::ABLayout<192, 32>;
 6171 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6173-6174

```text
 6173 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6174 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6176-6190

```text
 6176 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6178 | template <
 6179 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6180 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6181 | >
 6182 | using SM90_64x192x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x192x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 6184 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6185 | struct MMA_Traits<SM90_64x192x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 6186 | {
 6187 |   using ValTypeD = float;
 6188 |   using ValTypeA = float_e4m3_t;
 6189 |   using ValTypeB = float_e4m3_t;
 6190 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6192-6199

```text
 6192 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6193 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6195 |   using Shape_MNK = Shape<_64,_192,_32>;
 6196 |   using ThrID   = Layout<_128>;
 6197 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6198 |   using BLayout = GMMA::ABLayout<192, 32>;
 6199 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6201-6202

```text
 6201 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6202 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6204-6218

```text
 6204 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6206 | template <
 6207 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6208 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6209 | >
 6210 | using SM90_64x192x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x192x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 6212 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6213 | struct MMA_Traits<SM90_64x192x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 6214 | {
 6215 |   using ValTypeD = float;
 6216 |   using ValTypeA = float_e4m3_t;
 6217 |   using ValTypeB = float_e4m3_t;
 6218 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6220-6226

```text
 6220 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6222 |   using Shape_MNK = Shape<_64,_192,_32>;
 6223 |   using ThrID   = Layout<_128>;
 6224 |   using ALayout = GMMA::ALayout_64x32;
 6225 |   using BLayout = GMMA::ABLayout<192, 32>;
 6226 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6228-6229

```text
 6228 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6229 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6231-6245

```text
 6231 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6233 | template <
 6234 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6235 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6236 | >
 6237 | using SM90_64x256x32_F16E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x256x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>;
 6239 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6240 | struct MMA_Traits<SM90_64x256x32_F16E4M3E4M3_SS_TN<scaleA, scaleB>>
 6241 | {
 6242 |   using ValTypeD = half_t;
 6243 |   using ValTypeA = float_e4m3_t;
 6244 |   using ValTypeB = float_e4m3_t;
 6245 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6247-6254

```text
 6247 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6248 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6250 |   using Shape_MNK = Shape<_64,_256,_32>;
 6251 |   using ThrID   = Layout<_128>;
 6252 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6253 |   using BLayout = GMMA::ABLayout<256, 32>;
 6254 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6256-6257

```text
 6256 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6257 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6259-6273

```text
 6259 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6261 | template <
 6262 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6263 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6264 | >
 6265 | using SM90_64x256x32_F16E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x256x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>;
 6267 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6268 | struct MMA_Traits<SM90_64x256x32_F16E4M3E4M3_RS_TN<scaleA, scaleB>>
 6269 | {
 6270 |   using ValTypeD = half_t;
 6271 |   using ValTypeA = float_e4m3_t;
 6272 |   using ValTypeB = float_e4m3_t;
 6273 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6275-6281

```text
 6275 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6277 |   using Shape_MNK = Shape<_64,_256,_32>;
 6278 |   using ThrID   = Layout<_128>;
 6279 |   using ALayout = GMMA::ALayout_64x32;
 6280 |   using BLayout = GMMA::ABLayout<256, 32>;
 6281 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6283-6284

```text
 6283 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6284 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6286-6300

```text
 6286 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6288 | template <
 6289 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6290 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6291 | >
 6292 | using SM90_64x256x32_F32E4M3E4M3_SS_TN = SM90::GMMA::MMA_64x256x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>;
 6294 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6295 | struct MMA_Traits<SM90_64x256x32_F32E4M3E4M3_SS_TN<scaleA, scaleB>>
 6296 | {
 6297 |   using ValTypeD = float;
 6298 |   using ValTypeA = float_e4m3_t;
 6299 |   using ValTypeB = float_e4m3_t;
 6300 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E4M3E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E4M3E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6302-6309

```text
 6302 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6303 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6305 |   using Shape_MNK = Shape<_64,_256,_32>;
 6306 |   using ThrID   = Layout<_128>;
 6307 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6308 |   using BLayout = GMMA::ABLayout<256, 32>;
 6309 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6311-6312

```text
 6311 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6312 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6314-6328

```text
 6314 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6316 | template <
 6317 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6318 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6319 | >
 6320 | using SM90_64x256x32_F32E4M3E4M3_RS_TN = SM90::GMMA::MMA_64x256x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>;
 6322 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6323 | struct MMA_Traits<SM90_64x256x32_F32E4M3E4M3_RS_TN<scaleA, scaleB>>
 6324 | {
 6325 |   using ValTypeD = float;
 6326 |   using ValTypeA = float_e4m3_t;
 6327 |   using ValTypeB = float_e4m3_t;
 6328 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E4M3E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E4M3E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6330-6336

```text
 6330 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6332 |   using Shape_MNK = Shape<_64,_256,_32>;
 6333 |   using ThrID   = Layout<_128>;
 6334 |   using ALayout = GMMA::ALayout_64x32;
 6335 |   using BLayout = GMMA::ABLayout<256, 32>;
 6336 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6338-6339

```text
 6338 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6339 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6341-6355

```text
 6341 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6343 | template <
 6344 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6345 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6346 | >
 6347 | using SM90_64x8x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x8x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 6349 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6350 | struct MMA_Traits<SM90_64x8x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 6351 | {
 6352 |   using ValTypeD = half_t;
 6353 |   using ValTypeA = float_e4m3_t;
 6354 |   using ValTypeB = float_e5m2_t;
 6355 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6357-6364

```text
 6357 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6358 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6360 |   using Shape_MNK = Shape<_64,_8,_32>;
 6361 |   using ThrID   = Layout<_128>;
 6362 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6363 |   using BLayout = GMMA::ABLayout<  8, 32>;
 6364 |   using CLayout = GMMA::CLayout_64x8;
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

### Lines 6369-6383

```text
 6369 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6371 | template <
 6372 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6373 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6374 | >
 6375 | using SM90_64x8x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x8x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 6377 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6378 | struct MMA_Traits<SM90_64x8x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 6379 | {
 6380 |   using ValTypeD = half_t;
 6381 |   using ValTypeA = float_e4m3_t;
 6382 |   using ValTypeB = float_e5m2_t;
 6383 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6385-6391

```text
 6385 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6387 |   using Shape_MNK = Shape<_64,_8,_32>;
 6388 |   using ThrID   = Layout<_128>;
 6389 |   using ALayout = GMMA::ALayout_64x32;
 6390 |   using BLayout = GMMA::ABLayout<  8, 32>;
 6391 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6393-6394

```text
 6393 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6394 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6396-6410

```text
 6396 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6398 | template <
 6399 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6400 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6401 | >
 6402 | using SM90_64x8x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x8x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 6404 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6405 | struct MMA_Traits<SM90_64x8x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 6406 | {
 6407 |   using ValTypeD = float;
 6408 |   using ValTypeA = float_e4m3_t;
 6409 |   using ValTypeB = float_e5m2_t;
 6410 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6412-6419

```text
 6412 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6413 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6415 |   using Shape_MNK = Shape<_64,_8,_32>;
 6416 |   using ThrID   = Layout<_128>;
 6417 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6418 |   using BLayout = GMMA::ABLayout<  8, 32>;
 6419 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6421-6422

```text
 6421 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6422 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6424-6438

```text
 6424 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6426 | template <
 6427 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6428 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6429 | >
 6430 | using SM90_64x8x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x8x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 6432 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6433 | struct MMA_Traits<SM90_64x8x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 6434 | {
 6435 |   using ValTypeD = float;
 6436 |   using ValTypeA = float_e4m3_t;
 6437 |   using ValTypeB = float_e5m2_t;
 6438 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6440-6446

```text
 6440 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6442 |   using Shape_MNK = Shape<_64,_8,_32>;
 6443 |   using ThrID   = Layout<_128>;
 6444 |   using ALayout = GMMA::ALayout_64x32;
 6445 |   using BLayout = GMMA::ABLayout<  8, 32>;
 6446 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6448-6449

```text
 6448 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6449 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6451-6465

```text
 6451 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6453 | template <
 6454 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6455 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6456 | >
 6457 | using SM90_64x16x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x16x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 6459 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6460 | struct MMA_Traits<SM90_64x16x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 6461 | {
 6462 |   using ValTypeD = half_t;
 6463 |   using ValTypeA = float_e4m3_t;
 6464 |   using ValTypeB = float_e5m2_t;
 6465 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6467-6474

```text
 6467 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6468 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6470 |   using Shape_MNK = Shape<_64,_16,_32>;
 6471 |   using ThrID   = Layout<_128>;
 6472 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6473 |   using BLayout = GMMA::ABLayout< 16, 32>;
 6474 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6476-6477

```text
 6476 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6477 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6479-6493

```text
 6479 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6481 | template <
 6482 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6483 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6484 | >
 6485 | using SM90_64x16x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x16x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 6487 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6488 | struct MMA_Traits<SM90_64x16x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 6489 | {
 6490 |   using ValTypeD = half_t;
 6491 |   using ValTypeA = float_e4m3_t;
 6492 |   using ValTypeB = float_e5m2_t;
 6493 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6495-6501

```text
 6495 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6497 |   using Shape_MNK = Shape<_64,_16,_32>;
 6498 |   using ThrID   = Layout<_128>;
 6499 |   using ALayout = GMMA::ALayout_64x32;
 6500 |   using BLayout = GMMA::ABLayout< 16, 32>;
 6501 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6503-6504

```text
 6503 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6504 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6506-6520

```text
 6506 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6508 | template <
 6509 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6510 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6511 | >
 6512 | using SM90_64x16x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x16x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 6514 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6515 | struct MMA_Traits<SM90_64x16x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 6516 | {
 6517 |   using ValTypeD = float;
 6518 |   using ValTypeA = float_e4m3_t;
 6519 |   using ValTypeB = float_e5m2_t;
 6520 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6522-6529

```text
 6522 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6523 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6525 |   using Shape_MNK = Shape<_64,_16,_32>;
 6526 |   using ThrID   = Layout<_128>;
 6527 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6528 |   using BLayout = GMMA::ABLayout< 16, 32>;
 6529 |   using CLayout = GMMA::CLayout_64x16;
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

### Lines 6534-6548

```text
 6534 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6536 | template <
 6537 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6538 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6539 | >
 6540 | using SM90_64x16x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x16x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 6542 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6543 | struct MMA_Traits<SM90_64x16x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 6544 | {
 6545 |   using ValTypeD = float;
 6546 |   using ValTypeA = float_e4m3_t;
 6547 |   using ValTypeB = float_e5m2_t;
 6548 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6550-6556

```text
 6550 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6552 |   using Shape_MNK = Shape<_64,_16,_32>;
 6553 |   using ThrID   = Layout<_128>;
 6554 |   using ALayout = GMMA::ALayout_64x32;
 6555 |   using BLayout = GMMA::ABLayout< 16, 32>;
 6556 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6558-6559

```text
 6558 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6559 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6561-6575

```text
 6561 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6563 | template <
 6564 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6565 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6566 | >
 6567 | using SM90_64x32x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x32x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 6569 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6570 | struct MMA_Traits<SM90_64x32x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 6571 | {
 6572 |   using ValTypeD = half_t;
 6573 |   using ValTypeA = float_e4m3_t;
 6574 |   using ValTypeB = float_e5m2_t;
 6575 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6577-6584

```text
 6577 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6578 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6580 |   using Shape_MNK = Shape<_64,_32,_32>;
 6581 |   using ThrID   = Layout<_128>;
 6582 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6583 |   using BLayout = GMMA::ABLayout< 32, 32>;
 6584 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6586-6587

```text
 6586 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6587 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6589-6603

```text
 6589 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6591 | template <
 6592 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6593 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6594 | >
 6595 | using SM90_64x32x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x32x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 6597 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6598 | struct MMA_Traits<SM90_64x32x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 6599 | {
 6600 |   using ValTypeD = half_t;
 6601 |   using ValTypeA = float_e4m3_t;
 6602 |   using ValTypeB = float_e5m2_t;
 6603 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6605-6611

```text
 6605 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6607 |   using Shape_MNK = Shape<_64,_32,_32>;
 6608 |   using ThrID   = Layout<_128>;
 6609 |   using ALayout = GMMA::ALayout_64x32;
 6610 |   using BLayout = GMMA::ABLayout< 32, 32>;
 6611 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6613-6614

```text
 6613 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6614 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6616-6630

```text
 6616 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6618 | template <
 6619 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6620 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6621 | >
 6622 | using SM90_64x32x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x32x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 6624 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6625 | struct MMA_Traits<SM90_64x32x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 6626 | {
 6627 |   using ValTypeD = float;
 6628 |   using ValTypeA = float_e4m3_t;
 6629 |   using ValTypeB = float_e5m2_t;
 6630 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6632-6639

```text
 6632 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6633 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6635 |   using Shape_MNK = Shape<_64,_32,_32>;
 6636 |   using ThrID   = Layout<_128>;
 6637 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6638 |   using BLayout = GMMA::ABLayout< 32, 32>;
 6639 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6641-6642

```text
 6641 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6642 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6644-6658

```text
 6644 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6646 | template <
 6647 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6648 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6649 | >
 6650 | using SM90_64x32x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x32x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 6652 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6653 | struct MMA_Traits<SM90_64x32x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 6654 | {
 6655 |   using ValTypeD = float;
 6656 |   using ValTypeA = float_e4m3_t;
 6657 |   using ValTypeB = float_e5m2_t;
 6658 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6660-6666

```text
 6660 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6662 |   using Shape_MNK = Shape<_64,_32,_32>;
 6663 |   using ThrID   = Layout<_128>;
 6664 |   using ALayout = GMMA::ALayout_64x32;
 6665 |   using BLayout = GMMA::ABLayout< 32, 32>;
 6666 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6668-6669

```text
 6668 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6669 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6671-6685

```text
 6671 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6673 | template <
 6674 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6675 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6676 | >
 6677 | using SM90_64x64x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x64x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 6679 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6680 | struct MMA_Traits<SM90_64x64x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 6681 | {
 6682 |   using ValTypeD = half_t;
 6683 |   using ValTypeA = float_e4m3_t;
 6684 |   using ValTypeB = float_e5m2_t;
 6685 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6687-6694

```text
 6687 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6688 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6690 |   using Shape_MNK = Shape<_64,_64,_32>;
 6691 |   using ThrID   = Layout<_128>;
 6692 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6693 |   using BLayout = GMMA::ABLayout< 64, 32>;
 6694 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6696-6697

```text
 6696 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6697 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6699-6713

```text
 6699 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6701 | template <
 6702 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6703 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6704 | >
 6705 | using SM90_64x64x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x64x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 6707 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6708 | struct MMA_Traits<SM90_64x64x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 6709 | {
 6710 |   using ValTypeD = half_t;
 6711 |   using ValTypeA = float_e4m3_t;
 6712 |   using ValTypeB = float_e5m2_t;
 6713 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6715-6721

```text
 6715 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6717 |   using Shape_MNK = Shape<_64,_64,_32>;
 6718 |   using ThrID   = Layout<_128>;
 6719 |   using ALayout = GMMA::ALayout_64x32;
 6720 |   using BLayout = GMMA::ABLayout< 64, 32>;
 6721 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6723-6724

```text
 6723 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6724 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6726-6740

```text
 6726 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6728 | template <
 6729 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6730 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6731 | >
 6732 | using SM90_64x64x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x64x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 6734 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6735 | struct MMA_Traits<SM90_64x64x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 6736 | {
 6737 |   using ValTypeD = float;
 6738 |   using ValTypeA = float_e4m3_t;
 6739 |   using ValTypeB = float_e5m2_t;
 6740 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6742-6749

```text
 6742 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6743 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6745 |   using Shape_MNK = Shape<_64,_64,_32>;
 6746 |   using ThrID   = Layout<_128>;
 6747 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6748 |   using BLayout = GMMA::ABLayout< 64, 32>;
 6749 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6751-6752

```text
 6751 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6752 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6754-6768

```text
 6754 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6756 | template <
 6757 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6758 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6759 | >
 6760 | using SM90_64x64x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x64x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 6762 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6763 | struct MMA_Traits<SM90_64x64x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 6764 | {
 6765 |   using ValTypeD = float;
 6766 |   using ValTypeA = float_e4m3_t;
 6767 |   using ValTypeB = float_e5m2_t;
 6768 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6770-6776

```text
 6770 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6772 |   using Shape_MNK = Shape<_64,_64,_32>;
 6773 |   using ThrID   = Layout<_128>;
 6774 |   using ALayout = GMMA::ALayout_64x32;
 6775 |   using BLayout = GMMA::ABLayout< 64, 32>;
 6776 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6778-6779

```text
 6778 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6779 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6781-6795

```text
 6781 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6783 | template <
 6784 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6785 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6786 | >
 6787 | using SM90_64x96x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x96x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 6789 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6790 | struct MMA_Traits<SM90_64x96x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 6791 | {
 6792 |   using ValTypeD = half_t;
 6793 |   using ValTypeA = float_e4m3_t;
 6794 |   using ValTypeB = float_e5m2_t;
 6795 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6797-6804

```text
 6797 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6798 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6800 |   using Shape_MNK = Shape<_64,_96,_32>;
 6801 |   using ThrID   = Layout<_128>;
 6802 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6803 |   using BLayout = GMMA::ABLayout< 96, 32>;
 6804 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6806-6807

```text
 6806 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6807 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6809-6823

```text
 6809 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6811 | template <
 6812 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6813 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6814 | >
 6815 | using SM90_64x96x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x96x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 6817 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6818 | struct MMA_Traits<SM90_64x96x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 6819 | {
 6820 |   using ValTypeD = half_t;
 6821 |   using ValTypeA = float_e4m3_t;
 6822 |   using ValTypeB = float_e5m2_t;
 6823 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6825-6831

```text
 6825 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6827 |   using Shape_MNK = Shape<_64,_96,_32>;
 6828 |   using ThrID   = Layout<_128>;
 6829 |   using ALayout = GMMA::ALayout_64x32;
 6830 |   using BLayout = GMMA::ABLayout< 96, 32>;
 6831 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6833-6834

```text
 6833 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6834 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6836-6850

```text
 6836 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6838 | template <
 6839 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6840 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6841 | >
 6842 | using SM90_64x96x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x96x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 6844 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6845 | struct MMA_Traits<SM90_64x96x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 6846 | {
 6847 |   using ValTypeD = float;
 6848 |   using ValTypeA = float_e4m3_t;
 6849 |   using ValTypeB = float_e5m2_t;
 6850 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6852-6859

```text
 6852 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6853 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6855 |   using Shape_MNK = Shape<_64,_96,_32>;
 6856 |   using ThrID   = Layout<_128>;
 6857 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6858 |   using BLayout = GMMA::ABLayout< 96, 32>;
 6859 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6861-6862

```text
 6861 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6862 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6864-6878

```text
 6864 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6866 | template <
 6867 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6868 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6869 | >
 6870 | using SM90_64x96x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x96x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 6872 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6873 | struct MMA_Traits<SM90_64x96x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 6874 | {
 6875 |   using ValTypeD = float;
 6876 |   using ValTypeA = float_e4m3_t;
 6877 |   using ValTypeB = float_e5m2_t;
 6878 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6880-6886

```text
 6880 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6882 |   using Shape_MNK = Shape<_64,_96,_32>;
 6883 |   using ThrID   = Layout<_128>;
 6884 |   using ALayout = GMMA::ALayout_64x32;
 6885 |   using BLayout = GMMA::ABLayout< 96, 32>;
 6886 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6888-6889

```text
 6888 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6889 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6891-6905

```text
 6891 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6893 | template <
 6894 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6895 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6896 | >
 6897 | using SM90_64x128x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x128x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 6899 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6900 | struct MMA_Traits<SM90_64x128x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 6901 | {
 6902 |   using ValTypeD = half_t;
 6903 |   using ValTypeA = float_e4m3_t;
 6904 |   using ValTypeB = float_e5m2_t;
 6905 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6907-6914

```text
 6907 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6908 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6910 |   using Shape_MNK = Shape<_64,_128,_32>;
 6911 |   using ThrID   = Layout<_128>;
 6912 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6913 |   using BLayout = GMMA::ABLayout<128, 32>;
 6914 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6916-6917

```text
 6916 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6917 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6919-6933

```text
 6919 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6921 | template <
 6922 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6923 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6924 | >
 6925 | using SM90_64x128x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x128x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 6927 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6928 | struct MMA_Traits<SM90_64x128x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 6929 | {
 6930 |   using ValTypeD = half_t;
 6931 |   using ValTypeA = float_e4m3_t;
 6932 |   using ValTypeB = float_e5m2_t;
 6933 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6935-6941

```text
 6935 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6937 |   using Shape_MNK = Shape<_64,_128,_32>;
 6938 |   using ThrID   = Layout<_128>;
 6939 |   using ALayout = GMMA::ALayout_64x32;
 6940 |   using BLayout = GMMA::ABLayout<128, 32>;
 6941 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6943-6944

```text
 6943 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6944 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6946-6960

```text
 6946 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6948 | template <
 6949 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6950 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6951 | >
 6952 | using SM90_64x128x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x128x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 6954 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6955 | struct MMA_Traits<SM90_64x128x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 6956 | {
 6957 |   using ValTypeD = float;
 6958 |   using ValTypeA = float_e4m3_t;
 6959 |   using ValTypeB = float_e5m2_t;
 6960 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6962-6969

```text
 6962 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 6963 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6965 |   using Shape_MNK = Shape<_64,_128,_32>;
 6966 |   using ThrID   = Layout<_128>;
 6967 |   using ALayout = GMMA::ABLayout< 64, 32>;
 6968 |   using BLayout = GMMA::ABLayout<128, 32>;
 6969 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6971-6972

```text
 6971 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6972 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6974-6988

```text
 6974 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 6976 | template <
 6977 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 6978 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 6979 | >
 6980 | using SM90_64x128x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x128x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 6982 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 6983 | struct MMA_Traits<SM90_64x128x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 6984 | {
 6985 |   using ValTypeD = float;
 6986 |   using ValTypeA = float_e4m3_t;
 6987 |   using ValTypeB = float_e5m2_t;
 6988 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 6990-6996

```text
 6990 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 6992 |   using Shape_MNK = Shape<_64,_128,_32>;
 6993 |   using ThrID   = Layout<_128>;
 6994 |   using ALayout = GMMA::ALayout_64x32;
 6995 |   using BLayout = GMMA::ABLayout<128, 32>;
 6996 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 6998-6999

```text
 6998 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 6999 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7001-7015

```text
 7001 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7003 | template <
 7004 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7005 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7006 | >
 7007 | using SM90_64x192x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x192x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 7009 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7010 | struct MMA_Traits<SM90_64x192x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 7011 | {
 7012 |   using ValTypeD = half_t;
 7013 |   using ValTypeA = float_e4m3_t;
 7014 |   using ValTypeB = float_e5m2_t;
 7015 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7017-7024

```text
 7017 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7018 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7020 |   using Shape_MNK = Shape<_64,_192,_32>;
 7021 |   using ThrID   = Layout<_128>;
 7022 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7023 |   using BLayout = GMMA::ABLayout<192, 32>;
 7024 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7026-7027

```text
 7026 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7027 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7029-7043

```text
 7029 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7031 | template <
 7032 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7033 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7034 | >
 7035 | using SM90_64x192x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x192x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 7037 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7038 | struct MMA_Traits<SM90_64x192x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 7039 | {
 7040 |   using ValTypeD = half_t;
 7041 |   using ValTypeA = float_e4m3_t;
 7042 |   using ValTypeB = float_e5m2_t;
 7043 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7045-7051

```text
 7045 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7047 |   using Shape_MNK = Shape<_64,_192,_32>;
 7048 |   using ThrID   = Layout<_128>;
 7049 |   using ALayout = GMMA::ALayout_64x32;
 7050 |   using BLayout = GMMA::ABLayout<192, 32>;
 7051 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7053-7054

```text
 7053 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7054 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7056-7070

```text
 7056 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7058 | template <
 7059 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7060 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7061 | >
 7062 | using SM90_64x192x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x192x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 7064 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7065 | struct MMA_Traits<SM90_64x192x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 7066 | {
 7067 |   using ValTypeD = float;
 7068 |   using ValTypeA = float_e4m3_t;
 7069 |   using ValTypeB = float_e5m2_t;
 7070 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7072-7079

```text
 7072 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7073 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7075 |   using Shape_MNK = Shape<_64,_192,_32>;
 7076 |   using ThrID   = Layout<_128>;
 7077 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7078 |   using BLayout = GMMA::ABLayout<192, 32>;
 7079 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7081-7082

```text
 7081 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7082 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7084-7098

```text
 7084 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7086 | template <
 7087 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7088 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7089 | >
 7090 | using SM90_64x192x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x192x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 7092 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7093 | struct MMA_Traits<SM90_64x192x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 7094 | {
 7095 |   using ValTypeD = float;
 7096 |   using ValTypeA = float_e4m3_t;
 7097 |   using ValTypeB = float_e5m2_t;
 7098 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7100-7106

```text
 7100 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7102 |   using Shape_MNK = Shape<_64,_192,_32>;
 7103 |   using ThrID   = Layout<_128>;
 7104 |   using ALayout = GMMA::ALayout_64x32;
 7105 |   using BLayout = GMMA::ABLayout<192, 32>;
 7106 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7108-7109

```text
 7108 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7109 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7111-7125

```text
 7111 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7113 | template <
 7114 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7115 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7116 | >
 7117 | using SM90_64x256x32_F16E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x256x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>;
 7119 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7120 | struct MMA_Traits<SM90_64x256x32_F16E4M3E5M2_SS_TN<scaleA, scaleB>>
 7121 | {
 7122 |   using ValTypeD = half_t;
 7123 |   using ValTypeA = float_e4m3_t;
 7124 |   using ValTypeB = float_e5m2_t;
 7125 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7127-7134

```text
 7127 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7128 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7130 |   using Shape_MNK = Shape<_64,_256,_32>;
 7131 |   using ThrID   = Layout<_128>;
 7132 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7133 |   using BLayout = GMMA::ABLayout<256, 32>;
 7134 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7136-7137

```text
 7136 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7137 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7139-7153

```text
 7139 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7141 | template <
 7142 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7143 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7144 | >
 7145 | using SM90_64x256x32_F16E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x256x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>;
 7147 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7148 | struct MMA_Traits<SM90_64x256x32_F16E4M3E5M2_RS_TN<scaleA, scaleB>>
 7149 | {
 7150 |   using ValTypeD = half_t;
 7151 |   using ValTypeA = float_e4m3_t;
 7152 |   using ValTypeB = float_e5m2_t;
 7153 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7155-7161

```text
 7155 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7157 |   using Shape_MNK = Shape<_64,_256,_32>;
 7158 |   using ThrID   = Layout<_128>;
 7159 |   using ALayout = GMMA::ALayout_64x32;
 7160 |   using BLayout = GMMA::ABLayout<256, 32>;
 7161 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7163-7164

```text
 7163 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7164 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7166-7180

```text
 7166 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7168 | template <
 7169 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7170 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7171 | >
 7172 | using SM90_64x256x32_F32E4M3E5M2_SS_TN = SM90::GMMA::MMA_64x256x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>;
 7174 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7175 | struct MMA_Traits<SM90_64x256x32_F32E4M3E5M2_SS_TN<scaleA, scaleB>>
 7176 | {
 7177 |   using ValTypeD = float;
 7178 |   using ValTypeA = float_e4m3_t;
 7179 |   using ValTypeB = float_e5m2_t;
 7180 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E4M3E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E4M3E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7182-7189

```text
 7182 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7183 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7185 |   using Shape_MNK = Shape<_64,_256,_32>;
 7186 |   using ThrID   = Layout<_128>;
 7187 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7188 |   using BLayout = GMMA::ABLayout<256, 32>;
 7189 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7191-7192

```text
 7191 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7192 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7194-7208

```text
 7194 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7196 | template <
 7197 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7198 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7199 | >
 7200 | using SM90_64x256x32_F32E4M3E5M2_RS_TN = SM90::GMMA::MMA_64x256x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>;
 7202 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7203 | struct MMA_Traits<SM90_64x256x32_F32E4M3E5M2_RS_TN<scaleA, scaleB>>
 7204 | {
 7205 |   using ValTypeD = float;
 7206 |   using ValTypeA = float_e4m3_t;
 7207 |   using ValTypeB = float_e5m2_t;
 7208 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E4M3E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E4M3E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7210-7216

```text
 7210 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7212 |   using Shape_MNK = Shape<_64,_256,_32>;
 7213 |   using ThrID   = Layout<_128>;
 7214 |   using ALayout = GMMA::ALayout_64x32;
 7215 |   using BLayout = GMMA::ABLayout<256, 32>;
 7216 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7218-7219

```text
 7218 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7219 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7221-7235

```text
 7221 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7223 | template <
 7224 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7225 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7226 | >
 7227 | using SM90_64x8x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x8x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7229 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7230 | struct MMA_Traits<SM90_64x8x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7231 | {
 7232 |   using ValTypeD = half_t;
 7233 |   using ValTypeA = float_e5m2_t;
 7234 |   using ValTypeB = float_e4m3_t;
 7235 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7237-7244

```text
 7237 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7238 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7240 |   using Shape_MNK = Shape<_64,_8,_32>;
 7241 |   using ThrID   = Layout<_128>;
 7242 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7243 |   using BLayout = GMMA::ABLayout<  8, 32>;
 7244 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7246-7247

```text
 7246 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7247 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7249-7263

```text
 7249 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7251 | template <
 7252 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7253 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7254 | >
 7255 | using SM90_64x8x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x8x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7257 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7258 | struct MMA_Traits<SM90_64x8x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7259 | {
 7260 |   using ValTypeD = half_t;
 7261 |   using ValTypeA = float_e5m2_t;
 7262 |   using ValTypeB = float_e4m3_t;
 7263 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7265-7271

```text
 7265 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7267 |   using Shape_MNK = Shape<_64,_8,_32>;
 7268 |   using ThrID   = Layout<_128>;
 7269 |   using ALayout = GMMA::ALayout_64x32;
 7270 |   using BLayout = GMMA::ABLayout<  8, 32>;
 7271 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7273-7274

```text
 7273 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7274 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7276-7290

```text
 7276 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7278 | template <
 7279 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7280 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7281 | >
 7282 | using SM90_64x8x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x8x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7284 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7285 | struct MMA_Traits<SM90_64x8x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7286 | {
 7287 |   using ValTypeD = float;
 7288 |   using ValTypeA = float_e5m2_t;
 7289 |   using ValTypeB = float_e4m3_t;
 7290 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7292-7299

```text
 7292 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7293 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7295 |   using Shape_MNK = Shape<_64,_8,_32>;
 7296 |   using ThrID   = Layout<_128>;
 7297 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7298 |   using BLayout = GMMA::ABLayout<  8, 32>;
 7299 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7301-7302

```text
 7301 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7302 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7304-7318

```text
 7304 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7306 | template <
 7307 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7308 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7309 | >
 7310 | using SM90_64x8x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x8x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7312 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7313 | struct MMA_Traits<SM90_64x8x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7314 | {
 7315 |   using ValTypeD = float;
 7316 |   using ValTypeA = float_e5m2_t;
 7317 |   using ValTypeB = float_e4m3_t;
 7318 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7320-7326

```text
 7320 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7322 |   using Shape_MNK = Shape<_64,_8,_32>;
 7323 |   using ThrID   = Layout<_128>;
 7324 |   using ALayout = GMMA::ALayout_64x32;
 7325 |   using BLayout = GMMA::ABLayout<  8, 32>;
 7326 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7328-7329

```text
 7328 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7329 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7331-7345

```text
 7331 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7333 | template <
 7334 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7335 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7336 | >
 7337 | using SM90_64x16x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x16x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7339 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7340 | struct MMA_Traits<SM90_64x16x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7341 | {
 7342 |   using ValTypeD = half_t;
 7343 |   using ValTypeA = float_e5m2_t;
 7344 |   using ValTypeB = float_e4m3_t;
 7345 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7347-7354

```text
 7347 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7348 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7350 |   using Shape_MNK = Shape<_64,_16,_32>;
 7351 |   using ThrID   = Layout<_128>;
 7352 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7353 |   using BLayout = GMMA::ABLayout< 16, 32>;
 7354 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7356-7357

```text
 7356 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7357 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7359-7373

```text
 7359 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7361 | template <
 7362 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7363 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7364 | >
 7365 | using SM90_64x16x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x16x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7367 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7368 | struct MMA_Traits<SM90_64x16x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7369 | {
 7370 |   using ValTypeD = half_t;
 7371 |   using ValTypeA = float_e5m2_t;
 7372 |   using ValTypeB = float_e4m3_t;
 7373 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7375-7381

```text
 7375 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7377 |   using Shape_MNK = Shape<_64,_16,_32>;
 7378 |   using ThrID   = Layout<_128>;
 7379 |   using ALayout = GMMA::ALayout_64x32;
 7380 |   using BLayout = GMMA::ABLayout< 16, 32>;
 7381 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7383-7384

```text
 7383 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7384 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7386-7400

```text
 7386 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7388 | template <
 7389 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7390 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7391 | >
 7392 | using SM90_64x16x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x16x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7394 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7395 | struct MMA_Traits<SM90_64x16x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7396 | {
 7397 |   using ValTypeD = float;
 7398 |   using ValTypeA = float_e5m2_t;
 7399 |   using ValTypeB = float_e4m3_t;
 7400 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7402-7409

```text
 7402 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7403 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7405 |   using Shape_MNK = Shape<_64,_16,_32>;
 7406 |   using ThrID   = Layout<_128>;
 7407 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7408 |   using BLayout = GMMA::ABLayout< 16, 32>;
 7409 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7411-7412

```text
 7411 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7412 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7414-7428

```text
 7414 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7416 | template <
 7417 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7418 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7419 | >
 7420 | using SM90_64x16x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x16x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7422 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7423 | struct MMA_Traits<SM90_64x16x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7424 | {
 7425 |   using ValTypeD = float;
 7426 |   using ValTypeA = float_e5m2_t;
 7427 |   using ValTypeB = float_e4m3_t;
 7428 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7430-7436

```text
 7430 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7432 |   using Shape_MNK = Shape<_64,_16,_32>;
 7433 |   using ThrID   = Layout<_128>;
 7434 |   using ALayout = GMMA::ALayout_64x32;
 7435 |   using BLayout = GMMA::ABLayout< 16, 32>;
 7436 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7438-7439

```text
 7438 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7439 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7441-7455

```text
 7441 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7443 | template <
 7444 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7445 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7446 | >
 7447 | using SM90_64x32x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x32x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7449 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7450 | struct MMA_Traits<SM90_64x32x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7451 | {
 7452 |   using ValTypeD = half_t;
 7453 |   using ValTypeA = float_e5m2_t;
 7454 |   using ValTypeB = float_e4m3_t;
 7455 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7457-7464

```text
 7457 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7458 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7460 |   using Shape_MNK = Shape<_64,_32,_32>;
 7461 |   using ThrID   = Layout<_128>;
 7462 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7463 |   using BLayout = GMMA::ABLayout< 32, 32>;
 7464 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7466-7467

```text
 7466 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7467 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7469-7483

```text
 7469 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7471 | template <
 7472 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7473 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7474 | >
 7475 | using SM90_64x32x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x32x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7477 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7478 | struct MMA_Traits<SM90_64x32x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7479 | {
 7480 |   using ValTypeD = half_t;
 7481 |   using ValTypeA = float_e5m2_t;
 7482 |   using ValTypeB = float_e4m3_t;
 7483 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7485-7491

```text
 7485 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7487 |   using Shape_MNK = Shape<_64,_32,_32>;
 7488 |   using ThrID   = Layout<_128>;
 7489 |   using ALayout = GMMA::ALayout_64x32;
 7490 |   using BLayout = GMMA::ABLayout< 32, 32>;
 7491 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7493-7494

```text
 7493 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7494 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7496-7510

```text
 7496 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7498 | template <
 7499 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7500 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7501 | >
 7502 | using SM90_64x32x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x32x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7504 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7505 | struct MMA_Traits<SM90_64x32x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7506 | {
 7507 |   using ValTypeD = float;
 7508 |   using ValTypeA = float_e5m2_t;
 7509 |   using ValTypeB = float_e4m3_t;
 7510 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7512-7519

```text
 7512 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7513 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7515 |   using Shape_MNK = Shape<_64,_32,_32>;
 7516 |   using ThrID   = Layout<_128>;
 7517 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7518 |   using BLayout = GMMA::ABLayout< 32, 32>;
 7519 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7521-7522

```text
 7521 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7522 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7524-7538

```text
 7524 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7526 | template <
 7527 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7528 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7529 | >
 7530 | using SM90_64x32x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x32x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7532 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7533 | struct MMA_Traits<SM90_64x32x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7534 | {
 7535 |   using ValTypeD = float;
 7536 |   using ValTypeA = float_e5m2_t;
 7537 |   using ValTypeB = float_e4m3_t;
 7538 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7540-7546

```text
 7540 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7542 |   using Shape_MNK = Shape<_64,_32,_32>;
 7543 |   using ThrID   = Layout<_128>;
 7544 |   using ALayout = GMMA::ALayout_64x32;
 7545 |   using BLayout = GMMA::ABLayout< 32, 32>;
 7546 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7548-7549

```text
 7548 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7549 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7551-7565

```text
 7551 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7553 | template <
 7554 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7555 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7556 | >
 7557 | using SM90_64x64x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x64x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7559 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7560 | struct MMA_Traits<SM90_64x64x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7561 | {
 7562 |   using ValTypeD = half_t;
 7563 |   using ValTypeA = float_e5m2_t;
 7564 |   using ValTypeB = float_e4m3_t;
 7565 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7567-7574

```text
 7567 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7568 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7570 |   using Shape_MNK = Shape<_64,_64,_32>;
 7571 |   using ThrID   = Layout<_128>;
 7572 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7573 |   using BLayout = GMMA::ABLayout< 64, 32>;
 7574 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7576-7577

```text
 7576 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7577 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7579-7593

```text
 7579 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7581 | template <
 7582 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7583 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7584 | >
 7585 | using SM90_64x64x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x64x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7587 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7588 | struct MMA_Traits<SM90_64x64x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7589 | {
 7590 |   using ValTypeD = half_t;
 7591 |   using ValTypeA = float_e5m2_t;
 7592 |   using ValTypeB = float_e4m3_t;
 7593 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7595-7601

```text
 7595 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7597 |   using Shape_MNK = Shape<_64,_64,_32>;
 7598 |   using ThrID   = Layout<_128>;
 7599 |   using ALayout = GMMA::ALayout_64x32;
 7600 |   using BLayout = GMMA::ABLayout< 64, 32>;
 7601 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7603-7604

```text
 7603 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7604 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7606-7620

```text
 7606 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7608 | template <
 7609 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7610 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7611 | >
 7612 | using SM90_64x64x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x64x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7614 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7615 | struct MMA_Traits<SM90_64x64x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7616 | {
 7617 |   using ValTypeD = float;
 7618 |   using ValTypeA = float_e5m2_t;
 7619 |   using ValTypeB = float_e4m3_t;
 7620 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7622-7629

```text
 7622 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7623 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7625 |   using Shape_MNK = Shape<_64,_64,_32>;
 7626 |   using ThrID   = Layout<_128>;
 7627 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7628 |   using BLayout = GMMA::ABLayout< 64, 32>;
 7629 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7631-7632

```text
 7631 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7632 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7634-7648

```text
 7634 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7636 | template <
 7637 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7638 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7639 | >
 7640 | using SM90_64x64x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x64x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7642 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7643 | struct MMA_Traits<SM90_64x64x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7644 | {
 7645 |   using ValTypeD = float;
 7646 |   using ValTypeA = float_e5m2_t;
 7647 |   using ValTypeB = float_e4m3_t;
 7648 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7650-7656

```text
 7650 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7652 |   using Shape_MNK = Shape<_64,_64,_32>;
 7653 |   using ThrID   = Layout<_128>;
 7654 |   using ALayout = GMMA::ALayout_64x32;
 7655 |   using BLayout = GMMA::ABLayout< 64, 32>;
 7656 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7658-7659

```text
 7658 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7659 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7661-7675

```text
 7661 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7663 | template <
 7664 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7665 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7666 | >
 7667 | using SM90_64x96x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x96x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7669 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7670 | struct MMA_Traits<SM90_64x96x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7671 | {
 7672 |   using ValTypeD = half_t;
 7673 |   using ValTypeA = float_e5m2_t;
 7674 |   using ValTypeB = float_e4m3_t;
 7675 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7677-7684

```text
 7677 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7678 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7680 |   using Shape_MNK = Shape<_64,_96,_32>;
 7681 |   using ThrID   = Layout<_128>;
 7682 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7683 |   using BLayout = GMMA::ABLayout< 96, 32>;
 7684 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7686-7687

```text
 7686 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7687 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7689-7703

```text
 7689 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7691 | template <
 7692 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7693 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7694 | >
 7695 | using SM90_64x96x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x96x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7697 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7698 | struct MMA_Traits<SM90_64x96x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7699 | {
 7700 |   using ValTypeD = half_t;
 7701 |   using ValTypeA = float_e5m2_t;
 7702 |   using ValTypeB = float_e4m3_t;
 7703 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7705-7711

```text
 7705 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7707 |   using Shape_MNK = Shape<_64,_96,_32>;
 7708 |   using ThrID   = Layout<_128>;
 7709 |   using ALayout = GMMA::ALayout_64x32;
 7710 |   using BLayout = GMMA::ABLayout< 96, 32>;
 7711 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7713-7714

```text
 7713 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7714 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7716-7730

```text
 7716 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7718 | template <
 7719 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7720 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7721 | >
 7722 | using SM90_64x96x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x96x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7724 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7725 | struct MMA_Traits<SM90_64x96x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7726 | {
 7727 |   using ValTypeD = float;
 7728 |   using ValTypeA = float_e5m2_t;
 7729 |   using ValTypeB = float_e4m3_t;
 7730 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7732-7739

```text
 7732 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7733 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7735 |   using Shape_MNK = Shape<_64,_96,_32>;
 7736 |   using ThrID   = Layout<_128>;
 7737 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7738 |   using BLayout = GMMA::ABLayout< 96, 32>;
 7739 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7741-7742

```text
 7741 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7742 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7744-7758

```text
 7744 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7746 | template <
 7747 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7748 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7749 | >
 7750 | using SM90_64x96x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x96x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7752 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7753 | struct MMA_Traits<SM90_64x96x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7754 | {
 7755 |   using ValTypeD = float;
 7756 |   using ValTypeA = float_e5m2_t;
 7757 |   using ValTypeB = float_e4m3_t;
 7758 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7760-7766

```text
 7760 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7762 |   using Shape_MNK = Shape<_64,_96,_32>;
 7763 |   using ThrID   = Layout<_128>;
 7764 |   using ALayout = GMMA::ALayout_64x32;
 7765 |   using BLayout = GMMA::ABLayout< 96, 32>;
 7766 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7768-7769

```text
 7768 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7769 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7771-7785

```text
 7771 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7773 | template <
 7774 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7775 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7776 | >
 7777 | using SM90_64x128x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x128x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7779 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7780 | struct MMA_Traits<SM90_64x128x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7781 | {
 7782 |   using ValTypeD = half_t;
 7783 |   using ValTypeA = float_e5m2_t;
 7784 |   using ValTypeB = float_e4m3_t;
 7785 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7787-7794

```text
 7787 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7788 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7790 |   using Shape_MNK = Shape<_64,_128,_32>;
 7791 |   using ThrID   = Layout<_128>;
 7792 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7793 |   using BLayout = GMMA::ABLayout<128, 32>;
 7794 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7796-7797

```text
 7796 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7797 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7799-7813

```text
 7799 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7801 | template <
 7802 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7803 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7804 | >
 7805 | using SM90_64x128x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x128x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7807 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7808 | struct MMA_Traits<SM90_64x128x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7809 | {
 7810 |   using ValTypeD = half_t;
 7811 |   using ValTypeA = float_e5m2_t;
 7812 |   using ValTypeB = float_e4m3_t;
 7813 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7815-7821

```text
 7815 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7817 |   using Shape_MNK = Shape<_64,_128,_32>;
 7818 |   using ThrID   = Layout<_128>;
 7819 |   using ALayout = GMMA::ALayout_64x32;
 7820 |   using BLayout = GMMA::ABLayout<128, 32>;
 7821 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7823-7824

```text
 7823 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7824 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7826-7840

```text
 7826 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7828 | template <
 7829 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7830 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7831 | >
 7832 | using SM90_64x128x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x128x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7834 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7835 | struct MMA_Traits<SM90_64x128x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7836 | {
 7837 |   using ValTypeD = float;
 7838 |   using ValTypeA = float_e5m2_t;
 7839 |   using ValTypeB = float_e4m3_t;
 7840 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7842-7849

```text
 7842 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7843 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7845 |   using Shape_MNK = Shape<_64,_128,_32>;
 7846 |   using ThrID   = Layout<_128>;
 7847 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7848 |   using BLayout = GMMA::ABLayout<128, 32>;
 7849 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7851-7852

```text
 7851 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7852 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7854-7868

```text
 7854 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7856 | template <
 7857 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7858 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7859 | >
 7860 | using SM90_64x128x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x128x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7862 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7863 | struct MMA_Traits<SM90_64x128x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7864 | {
 7865 |   using ValTypeD = float;
 7866 |   using ValTypeA = float_e5m2_t;
 7867 |   using ValTypeB = float_e4m3_t;
 7868 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7870-7876

```text
 7870 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7872 |   using Shape_MNK = Shape<_64,_128,_32>;
 7873 |   using ThrID   = Layout<_128>;
 7874 |   using ALayout = GMMA::ALayout_64x32;
 7875 |   using BLayout = GMMA::ABLayout<128, 32>;
 7876 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7878-7879

```text
 7878 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7879 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7881-7895

```text
 7881 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7883 | template <
 7884 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7885 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7886 | >
 7887 | using SM90_64x192x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x192x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7889 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7890 | struct MMA_Traits<SM90_64x192x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 7891 | {
 7892 |   using ValTypeD = half_t;
 7893 |   using ValTypeA = float_e5m2_t;
 7894 |   using ValTypeB = float_e4m3_t;
 7895 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7897-7904

```text
 7897 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7898 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7900 |   using Shape_MNK = Shape<_64,_192,_32>;
 7901 |   using ThrID   = Layout<_128>;
 7902 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7903 |   using BLayout = GMMA::ABLayout<192, 32>;
 7904 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7906-7907

```text
 7906 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7907 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7909-7923

```text
 7909 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7911 | template <
 7912 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7913 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7914 | >
 7915 | using SM90_64x192x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x192x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 7917 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7918 | struct MMA_Traits<SM90_64x192x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 7919 | {
 7920 |   using ValTypeD = half_t;
 7921 |   using ValTypeA = float_e5m2_t;
 7922 |   using ValTypeB = float_e4m3_t;
 7923 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7925-7931

```text
 7925 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7927 |   using Shape_MNK = Shape<_64,_192,_32>;
 7928 |   using ThrID   = Layout<_128>;
 7929 |   using ALayout = GMMA::ALayout_64x32;
 7930 |   using BLayout = GMMA::ABLayout<192, 32>;
 7931 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7933-7934

```text
 7933 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7934 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7936-7950

```text
 7936 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7938 | template <
 7939 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7940 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7941 | >
 7942 | using SM90_64x192x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x192x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 7944 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7945 | struct MMA_Traits<SM90_64x192x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 7946 | {
 7947 |   using ValTypeD = float;
 7948 |   using ValTypeA = float_e5m2_t;
 7949 |   using ValTypeB = float_e4m3_t;
 7950 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7952-7959

```text
 7952 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 7953 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7955 |   using Shape_MNK = Shape<_64,_192,_32>;
 7956 |   using ThrID   = Layout<_128>;
 7957 |   using ALayout = GMMA::ABLayout< 64, 32>;
 7958 |   using BLayout = GMMA::ABLayout<192, 32>;
 7959 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7961-7962

```text
 7961 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7962 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7964-7978

```text
 7964 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7966 | template <
 7967 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7968 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7969 | >
 7970 | using SM90_64x192x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x192x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 7972 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 7973 | struct MMA_Traits<SM90_64x192x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 7974 | {
 7975 |   using ValTypeD = float;
 7976 |   using ValTypeA = float_e5m2_t;
 7977 |   using ValTypeB = float_e4m3_t;
 7978 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 7980-7986

```text
 7980 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 7982 |   using Shape_MNK = Shape<_64,_192,_32>;
 7983 |   using ThrID   = Layout<_128>;
 7984 |   using ALayout = GMMA::ALayout_64x32;
 7985 |   using BLayout = GMMA::ABLayout<192, 32>;
 7986 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7988-7989

```text
 7988 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 7989 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 7991-8005

```text
 7991 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 7993 | template <
 7994 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 7995 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 7996 | >
 7997 | using SM90_64x256x32_F16E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x256x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>;
 7999 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8000 | struct MMA_Traits<SM90_64x256x32_F16E5M2E4M3_SS_TN<scaleA, scaleB>>
 8001 | {
 8002 |   using ValTypeD = half_t;
 8003 |   using ValTypeA = float_e5m2_t;
 8004 |   using ValTypeB = float_e4m3_t;
 8005 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8007-8014

```text
 8007 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8008 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8010 |   using Shape_MNK = Shape<_64,_256,_32>;
 8011 |   using ThrID   = Layout<_128>;
 8012 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8013 |   using BLayout = GMMA::ABLayout<256, 32>;
 8014 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8016-8017

```text
 8016 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8017 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8019-8033

```text
 8019 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8021 | template <
 8022 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8023 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8024 | >
 8025 | using SM90_64x256x32_F16E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x256x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>;
 8027 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8028 | struct MMA_Traits<SM90_64x256x32_F16E5M2E4M3_RS_TN<scaleA, scaleB>>
 8029 | {
 8030 |   using ValTypeD = half_t;
 8031 |   using ValTypeA = float_e5m2_t;
 8032 |   using ValTypeB = float_e4m3_t;
 8033 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8035-8041

```text
 8035 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8037 |   using Shape_MNK = Shape<_64,_256,_32>;
 8038 |   using ThrID   = Layout<_128>;
 8039 |   using ALayout = GMMA::ALayout_64x32;
 8040 |   using BLayout = GMMA::ABLayout<256, 32>;
 8041 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8043-8044

```text
 8043 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8044 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8046-8060

```text
 8046 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8048 | template <
 8049 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8050 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8051 | >
 8052 | using SM90_64x256x32_F32E5M2E4M3_SS_TN = SM90::GMMA::MMA_64x256x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>;
 8054 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8055 | struct MMA_Traits<SM90_64x256x32_F32E5M2E4M3_SS_TN<scaleA, scaleB>>
 8056 | {
 8057 |   using ValTypeD = float;
 8058 |   using ValTypeA = float_e5m2_t;
 8059 |   using ValTypeB = float_e4m3_t;
 8060 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E5M2E4M3_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E5M2E4M3_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8062-8069

```text
 8062 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8063 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8065 |   using Shape_MNK = Shape<_64,_256,_32>;
 8066 |   using ThrID   = Layout<_128>;
 8067 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8068 |   using BLayout = GMMA::ABLayout<256, 32>;
 8069 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8071-8072

```text
 8071 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8072 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8074-8088

```text
 8074 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8076 | template <
 8077 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8078 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8079 | >
 8080 | using SM90_64x256x32_F32E5M2E4M3_RS_TN = SM90::GMMA::MMA_64x256x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>;
 8082 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8083 | struct MMA_Traits<SM90_64x256x32_F32E5M2E4M3_RS_TN<scaleA, scaleB>>
 8084 | {
 8085 |   using ValTypeD = float;
 8086 |   using ValTypeA = float_e5m2_t;
 8087 |   using ValTypeB = float_e4m3_t;
 8088 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E5M2E4M3_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E5M2E4M3_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8090-8096

```text
 8090 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8092 |   using Shape_MNK = Shape<_64,_256,_32>;
 8093 |   using ThrID   = Layout<_128>;
 8094 |   using ALayout = GMMA::ALayout_64x32;
 8095 |   using BLayout = GMMA::ABLayout<256, 32>;
 8096 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8098-8099

```text
 8098 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8099 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8101-8115

```text
 8101 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8103 | template <
 8104 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8105 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8106 | >
 8107 | using SM90_64x8x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x8x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8109 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8110 | struct MMA_Traits<SM90_64x8x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8111 | {
 8112 |   using ValTypeD = half_t;
 8113 |   using ValTypeA = float_e5m2_t;
 8114 |   using ValTypeB = float_e5m2_t;
 8115 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8117-8124

```text
 8117 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8118 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8120 |   using Shape_MNK = Shape<_64,_8,_32>;
 8121 |   using ThrID   = Layout<_128>;
 8122 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8123 |   using BLayout = GMMA::ABLayout<  8, 32>;
 8124 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8126-8127

```text
 8126 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8127 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8129-8143

```text
 8129 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8131 | template <
 8132 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8133 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8134 | >
 8135 | using SM90_64x8x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x8x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8137 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8138 | struct MMA_Traits<SM90_64x8x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8139 | {
 8140 |   using ValTypeD = half_t;
 8141 |   using ValTypeA = float_e5m2_t;
 8142 |   using ValTypeB = float_e5m2_t;
 8143 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8145-8151

```text
 8145 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8147 |   using Shape_MNK = Shape<_64,_8,_32>;
 8148 |   using ThrID   = Layout<_128>;
 8149 |   using ALayout = GMMA::ALayout_64x32;
 8150 |   using BLayout = GMMA::ABLayout<  8, 32>;
 8151 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8153-8154

```text
 8153 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8154 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8156-8170

```text
 8156 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8158 | template <
 8159 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8160 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8161 | >
 8162 | using SM90_64x8x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x8x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8164 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8165 | struct MMA_Traits<SM90_64x8x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8166 | {
 8167 |   using ValTypeD = float;
 8168 |   using ValTypeA = float_e5m2_t;
 8169 |   using ValTypeB = float_e5m2_t;
 8170 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8172-8179

```text
 8172 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8173 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8175 |   using Shape_MNK = Shape<_64,_8,_32>;
 8176 |   using ThrID   = Layout<_128>;
 8177 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8178 |   using BLayout = GMMA::ABLayout<  8, 32>;
 8179 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8181-8182

```text
 8181 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8182 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8184-8198

```text
 8184 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8186 | template <
 8187 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8188 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8189 | >
 8190 | using SM90_64x8x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x8x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8192 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8193 | struct MMA_Traits<SM90_64x8x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8194 | {
 8195 |   using ValTypeD = float;
 8196 |   using ValTypeA = float_e5m2_t;
 8197 |   using ValTypeB = float_e5m2_t;
 8198 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x8x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x8x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8200-8206

```text
 8200 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8202 |   using Shape_MNK = Shape<_64,_8,_32>;
 8203 |   using ThrID   = Layout<_128>;
 8204 |   using ALayout = GMMA::ALayout_64x32;
 8205 |   using BLayout = GMMA::ABLayout<  8, 32>;
 8206 |   using CLayout = GMMA::CLayout_64x8;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8208-8209

```text
 8208 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8209 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8211-8225

```text
 8211 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8213 | template <
 8214 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8215 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8216 | >
 8217 | using SM90_64x16x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x16x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8219 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8220 | struct MMA_Traits<SM90_64x16x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8221 | {
 8222 |   using ValTypeD = half_t;
 8223 |   using ValTypeA = float_e5m2_t;
 8224 |   using ValTypeB = float_e5m2_t;
 8225 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8227-8234

```text
 8227 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8228 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8230 |   using Shape_MNK = Shape<_64,_16,_32>;
 8231 |   using ThrID   = Layout<_128>;
 8232 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8233 |   using BLayout = GMMA::ABLayout< 16, 32>;
 8234 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8236-8237

```text
 8236 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8237 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8239-8253

```text
 8239 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8241 | template <
 8242 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8243 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8244 | >
 8245 | using SM90_64x16x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x16x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8247 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8248 | struct MMA_Traits<SM90_64x16x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8249 | {
 8250 |   using ValTypeD = half_t;
 8251 |   using ValTypeA = float_e5m2_t;
 8252 |   using ValTypeB = float_e5m2_t;
 8253 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8255-8261

```text
 8255 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8257 |   using Shape_MNK = Shape<_64,_16,_32>;
 8258 |   using ThrID   = Layout<_128>;
 8259 |   using ALayout = GMMA::ALayout_64x32;
 8260 |   using BLayout = GMMA::ABLayout< 16, 32>;
 8261 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8263-8264

```text
 8263 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8264 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8266-8280

```text
 8266 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8268 | template <
 8269 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8270 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8271 | >
 8272 | using SM90_64x16x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x16x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8274 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8275 | struct MMA_Traits<SM90_64x16x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8276 | {
 8277 |   using ValTypeD = float;
 8278 |   using ValTypeA = float_e5m2_t;
 8279 |   using ValTypeB = float_e5m2_t;
 8280 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8282-8289

```text
 8282 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8283 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8285 |   using Shape_MNK = Shape<_64,_16,_32>;
 8286 |   using ThrID   = Layout<_128>;
 8287 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8288 |   using BLayout = GMMA::ABLayout< 16, 32>;
 8289 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8291-8292

```text
 8291 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8292 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8294-8308

```text
 8294 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8296 | template <
 8297 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8298 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8299 | >
 8300 | using SM90_64x16x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x16x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8302 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8303 | struct MMA_Traits<SM90_64x16x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8304 | {
 8305 |   using ValTypeD = float;
 8306 |   using ValTypeA = float_e5m2_t;
 8307 |   using ValTypeB = float_e5m2_t;
 8308 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x16x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x16x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8310-8316

```text
 8310 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8312 |   using Shape_MNK = Shape<_64,_16,_32>;
 8313 |   using ThrID   = Layout<_128>;
 8314 |   using ALayout = GMMA::ALayout_64x32;
 8315 |   using BLayout = GMMA::ABLayout< 16, 32>;
 8316 |   using CLayout = GMMA::CLayout_64x16;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8318-8319

```text
 8318 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8319 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8321-8335

```text
 8321 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8323 | template <
 8324 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8325 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8326 | >
 8327 | using SM90_64x32x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x32x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8329 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8330 | struct MMA_Traits<SM90_64x32x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8331 | {
 8332 |   using ValTypeD = half_t;
 8333 |   using ValTypeA = float_e5m2_t;
 8334 |   using ValTypeB = float_e5m2_t;
 8335 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8337-8344

```text
 8337 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8338 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8340 |   using Shape_MNK = Shape<_64,_32,_32>;
 8341 |   using ThrID   = Layout<_128>;
 8342 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8343 |   using BLayout = GMMA::ABLayout< 32, 32>;
 8344 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8346-8347

```text
 8346 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8347 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8349-8363

```text
 8349 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8351 | template <
 8352 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8353 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8354 | >
 8355 | using SM90_64x32x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x32x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8357 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8358 | struct MMA_Traits<SM90_64x32x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8359 | {
 8360 |   using ValTypeD = half_t;
 8361 |   using ValTypeA = float_e5m2_t;
 8362 |   using ValTypeB = float_e5m2_t;
 8363 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8365-8371

```text
 8365 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8367 |   using Shape_MNK = Shape<_64,_32,_32>;
 8368 |   using ThrID   = Layout<_128>;
 8369 |   using ALayout = GMMA::ALayout_64x32;
 8370 |   using BLayout = GMMA::ABLayout< 32, 32>;
 8371 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8373-8374

```text
 8373 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8374 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8376-8390

```text
 8376 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8378 | template <
 8379 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8380 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8381 | >
 8382 | using SM90_64x32x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x32x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8384 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8385 | struct MMA_Traits<SM90_64x32x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8386 | {
 8387 |   using ValTypeD = float;
 8388 |   using ValTypeA = float_e5m2_t;
 8389 |   using ValTypeB = float_e5m2_t;
 8390 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8392-8399

```text
 8392 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8393 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8395 |   using Shape_MNK = Shape<_64,_32,_32>;
 8396 |   using ThrID   = Layout<_128>;
 8397 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8398 |   using BLayout = GMMA::ABLayout< 32, 32>;
 8399 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8401-8402

```text
 8401 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8402 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8404-8418

```text
 8404 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8406 | template <
 8407 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8408 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8409 | >
 8410 | using SM90_64x32x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x32x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8412 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8413 | struct MMA_Traits<SM90_64x32x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8414 | {
 8415 |   using ValTypeD = float;
 8416 |   using ValTypeA = float_e5m2_t;
 8417 |   using ValTypeB = float_e5m2_t;
 8418 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x32x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x32x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8420-8426

```text
 8420 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8422 |   using Shape_MNK = Shape<_64,_32,_32>;
 8423 |   using ThrID   = Layout<_128>;
 8424 |   using ALayout = GMMA::ALayout_64x32;
 8425 |   using BLayout = GMMA::ABLayout< 32, 32>;
 8426 |   using CLayout = GMMA::CLayout_64x32;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8428-8429

```text
 8428 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8429 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8431-8445

```text
 8431 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8433 | template <
 8434 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8435 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8436 | >
 8437 | using SM90_64x64x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x64x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8439 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8440 | struct MMA_Traits<SM90_64x64x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8441 | {
 8442 |   using ValTypeD = half_t;
 8443 |   using ValTypeA = float_e5m2_t;
 8444 |   using ValTypeB = float_e5m2_t;
 8445 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8447-8454

```text
 8447 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8448 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8450 |   using Shape_MNK = Shape<_64,_64,_32>;
 8451 |   using ThrID   = Layout<_128>;
 8452 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8453 |   using BLayout = GMMA::ABLayout< 64, 32>;
 8454 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8456-8457

```text
 8456 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8457 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8459-8473

```text
 8459 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8461 | template <
 8462 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8463 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8464 | >
 8465 | using SM90_64x64x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x64x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8467 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8468 | struct MMA_Traits<SM90_64x64x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8469 | {
 8470 |   using ValTypeD = half_t;
 8471 |   using ValTypeA = float_e5m2_t;
 8472 |   using ValTypeB = float_e5m2_t;
 8473 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8475-8481

```text
 8475 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8477 |   using Shape_MNK = Shape<_64,_64,_32>;
 8478 |   using ThrID   = Layout<_128>;
 8479 |   using ALayout = GMMA::ALayout_64x32;
 8480 |   using BLayout = GMMA::ABLayout< 64, 32>;
 8481 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8483-8484

```text
 8483 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8484 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8486-8500

```text
 8486 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8488 | template <
 8489 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8490 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8491 | >
 8492 | using SM90_64x64x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x64x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8494 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8495 | struct MMA_Traits<SM90_64x64x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8496 | {
 8497 |   using ValTypeD = float;
 8498 |   using ValTypeA = float_e5m2_t;
 8499 |   using ValTypeB = float_e5m2_t;
 8500 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8502-8509

```text
 8502 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8503 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8505 |   using Shape_MNK = Shape<_64,_64,_32>;
 8506 |   using ThrID   = Layout<_128>;
 8507 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8508 |   using BLayout = GMMA::ABLayout< 64, 32>;
 8509 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8511-8512

```text
 8511 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8512 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8514-8528

```text
 8514 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8516 | template <
 8517 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8518 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8519 | >
 8520 | using SM90_64x64x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x64x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8522 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8523 | struct MMA_Traits<SM90_64x64x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8524 | {
 8525 |   using ValTypeD = float;
 8526 |   using ValTypeA = float_e5m2_t;
 8527 |   using ValTypeB = float_e5m2_t;
 8528 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x64x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x64x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8530-8536

```text
 8530 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8532 |   using Shape_MNK = Shape<_64,_64,_32>;
 8533 |   using ThrID   = Layout<_128>;
 8534 |   using ALayout = GMMA::ALayout_64x32;
 8535 |   using BLayout = GMMA::ABLayout< 64, 32>;
 8536 |   using CLayout = GMMA::CLayout_64x64;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8538-8539

```text
 8538 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8539 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8541-8555

```text
 8541 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8543 | template <
 8544 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8545 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8546 | >
 8547 | using SM90_64x96x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x96x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8549 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8550 | struct MMA_Traits<SM90_64x96x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8551 | {
 8552 |   using ValTypeD = half_t;
 8553 |   using ValTypeA = float_e5m2_t;
 8554 |   using ValTypeB = float_e5m2_t;
 8555 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8557-8564

```text
 8557 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8558 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8560 |   using Shape_MNK = Shape<_64,_96,_32>;
 8561 |   using ThrID   = Layout<_128>;
 8562 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8563 |   using BLayout = GMMA::ABLayout< 96, 32>;
 8564 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8566-8567

```text
 8566 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8567 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8569-8583

```text
 8569 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8571 | template <
 8572 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8573 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8574 | >
 8575 | using SM90_64x96x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x96x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8577 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8578 | struct MMA_Traits<SM90_64x96x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8579 | {
 8580 |   using ValTypeD = half_t;
 8581 |   using ValTypeA = float_e5m2_t;
 8582 |   using ValTypeB = float_e5m2_t;
 8583 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8585-8591

```text
 8585 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8587 |   using Shape_MNK = Shape<_64,_96,_32>;
 8588 |   using ThrID   = Layout<_128>;
 8589 |   using ALayout = GMMA::ALayout_64x32;
 8590 |   using BLayout = GMMA::ABLayout< 96, 32>;
 8591 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8593-8594

```text
 8593 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8594 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8596-8610

```text
 8596 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8598 | template <
 8599 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8600 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8601 | >
 8602 | using SM90_64x96x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x96x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8604 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8605 | struct MMA_Traits<SM90_64x96x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8606 | {
 8607 |   using ValTypeD = float;
 8608 |   using ValTypeA = float_e5m2_t;
 8609 |   using ValTypeB = float_e5m2_t;
 8610 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8612-8619

```text
 8612 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8613 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8615 |   using Shape_MNK = Shape<_64,_96,_32>;
 8616 |   using ThrID   = Layout<_128>;
 8617 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8618 |   using BLayout = GMMA::ABLayout< 96, 32>;
 8619 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8621-8622

```text
 8621 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8622 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8624-8638

```text
 8624 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8626 | template <
 8627 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8628 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8629 | >
 8630 | using SM90_64x96x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x96x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8632 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8633 | struct MMA_Traits<SM90_64x96x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8634 | {
 8635 |   using ValTypeD = float;
 8636 |   using ValTypeA = float_e5m2_t;
 8637 |   using ValTypeB = float_e5m2_t;
 8638 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x96x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x96x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8640-8646

```text
 8640 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8642 |   using Shape_MNK = Shape<_64,_96,_32>;
 8643 |   using ThrID   = Layout<_128>;
 8644 |   using ALayout = GMMA::ALayout_64x32;
 8645 |   using BLayout = GMMA::ABLayout< 96, 32>;
 8646 |   using CLayout = GMMA::CLayout_64x96;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8648-8649

```text
 8648 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8649 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8651-8665

```text
 8651 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8653 | template <
 8654 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8655 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8656 | >
 8657 | using SM90_64x128x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x128x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8659 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8660 | struct MMA_Traits<SM90_64x128x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8661 | {
 8662 |   using ValTypeD = half_t;
 8663 |   using ValTypeA = float_e5m2_t;
 8664 |   using ValTypeB = float_e5m2_t;
 8665 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8667-8674

```text
 8667 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8668 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8670 |   using Shape_MNK = Shape<_64,_128,_32>;
 8671 |   using ThrID   = Layout<_128>;
 8672 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8673 |   using BLayout = GMMA::ABLayout<128, 32>;
 8674 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8676-8677

```text
 8676 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8677 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8679-8693

```text
 8679 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8681 | template <
 8682 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8683 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8684 | >
 8685 | using SM90_64x128x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x128x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8687 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8688 | struct MMA_Traits<SM90_64x128x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8689 | {
 8690 |   using ValTypeD = half_t;
 8691 |   using ValTypeA = float_e5m2_t;
 8692 |   using ValTypeB = float_e5m2_t;
 8693 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8695-8701

```text
 8695 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8697 |   using Shape_MNK = Shape<_64,_128,_32>;
 8698 |   using ThrID   = Layout<_128>;
 8699 |   using ALayout = GMMA::ALayout_64x32;
 8700 |   using BLayout = GMMA::ABLayout<128, 32>;
 8701 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8703-8704

```text
 8703 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8704 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8706-8720

```text
 8706 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8708 | template <
 8709 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8710 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8711 | >
 8712 | using SM90_64x128x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x128x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8714 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8715 | struct MMA_Traits<SM90_64x128x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8716 | {
 8717 |   using ValTypeD = float;
 8718 |   using ValTypeA = float_e5m2_t;
 8719 |   using ValTypeB = float_e5m2_t;
 8720 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8722-8729

```text
 8722 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8723 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8725 |   using Shape_MNK = Shape<_64,_128,_32>;
 8726 |   using ThrID   = Layout<_128>;
 8727 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8728 |   using BLayout = GMMA::ABLayout<128, 32>;
 8729 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8731-8732

```text
 8731 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8732 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8734-8748

```text
 8734 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8736 | template <
 8737 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8738 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8739 | >
 8740 | using SM90_64x128x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x128x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8742 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8743 | struct MMA_Traits<SM90_64x128x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8744 | {
 8745 |   using ValTypeD = float;
 8746 |   using ValTypeA = float_e5m2_t;
 8747 |   using ValTypeB = float_e5m2_t;
 8748 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x128x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x128x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8750-8756

```text
 8750 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8752 |   using Shape_MNK = Shape<_64,_128,_32>;
 8753 |   using ThrID   = Layout<_128>;
 8754 |   using ALayout = GMMA::ALayout_64x32;
 8755 |   using BLayout = GMMA::ABLayout<128, 32>;
 8756 |   using CLayout = GMMA::CLayout_64x128;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8758-8759

```text
 8758 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8759 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8761-8775

```text
 8761 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8763 | template <
 8764 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8765 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8766 | >
 8767 | using SM90_64x192x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x192x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8769 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8770 | struct MMA_Traits<SM90_64x192x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8771 | {
 8772 |   using ValTypeD = half_t;
 8773 |   using ValTypeA = float_e5m2_t;
 8774 |   using ValTypeB = float_e5m2_t;
 8775 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8777-8784

```text
 8777 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8778 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8780 |   using Shape_MNK = Shape<_64,_192,_32>;
 8781 |   using ThrID   = Layout<_128>;
 8782 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8783 |   using BLayout = GMMA::ABLayout<192, 32>;
 8784 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8786-8787

```text
 8786 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8787 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8789-8803

```text
 8789 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8791 | template <
 8792 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8793 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8794 | >
 8795 | using SM90_64x192x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x192x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8797 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8798 | struct MMA_Traits<SM90_64x192x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8799 | {
 8800 |   using ValTypeD = half_t;
 8801 |   using ValTypeA = float_e5m2_t;
 8802 |   using ValTypeB = float_e5m2_t;
 8803 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8805-8811

```text
 8805 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8807 |   using Shape_MNK = Shape<_64,_192,_32>;
 8808 |   using ThrID   = Layout<_128>;
 8809 |   using ALayout = GMMA::ALayout_64x32;
 8810 |   using BLayout = GMMA::ABLayout<192, 32>;
 8811 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8813-8814

```text
 8813 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8814 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8816-8830

```text
 8816 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8818 | template <
 8819 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8820 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8821 | >
 8822 | using SM90_64x192x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x192x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8824 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8825 | struct MMA_Traits<SM90_64x192x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8826 | {
 8827 |   using ValTypeD = float;
 8828 |   using ValTypeA = float_e5m2_t;
 8829 |   using ValTypeB = float_e5m2_t;
 8830 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8832-8839

```text
 8832 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8833 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8835 |   using Shape_MNK = Shape<_64,_192,_32>;
 8836 |   using ThrID   = Layout<_128>;
 8837 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8838 |   using BLayout = GMMA::ABLayout<192, 32>;
 8839 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8841-8842

```text
 8841 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8842 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8844-8858

```text
 8844 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8846 | template <
 8847 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8848 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8849 | >
 8850 | using SM90_64x192x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x192x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8852 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8853 | struct MMA_Traits<SM90_64x192x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8854 | {
 8855 |   using ValTypeD = float;
 8856 |   using ValTypeA = float_e5m2_t;
 8857 |   using ValTypeB = float_e5m2_t;
 8858 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x192x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x192x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8860-8866

```text
 8860 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8862 |   using Shape_MNK = Shape<_64,_192,_32>;
 8863 |   using ThrID   = Layout<_128>;
 8864 |   using ALayout = GMMA::ALayout_64x32;
 8865 |   using BLayout = GMMA::ABLayout<192, 32>;
 8866 |   using CLayout = GMMA::CLayout_64x192;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8868-8869

```text
 8868 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8869 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8871-8885

```text
 8871 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8873 | template <
 8874 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8875 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8876 | >
 8877 | using SM90_64x256x32_F16E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x256x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>;
 8879 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8880 | struct MMA_Traits<SM90_64x256x32_F16E5M2E5M2_SS_TN<scaleA, scaleB>>
 8881 | {
 8882 |   using ValTypeD = half_t;
 8883 |   using ValTypeA = float_e5m2_t;
 8884 |   using ValTypeB = float_e5m2_t;
 8885 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8887-8894

```text
 8887 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8888 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8890 |   using Shape_MNK = Shape<_64,_256,_32>;
 8891 |   using ThrID   = Layout<_128>;
 8892 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8893 |   using BLayout = GMMA::ABLayout<256, 32>;
 8894 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8896-8897

```text
 8896 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8897 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8899-8913

```text
 8899 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8901 | template <
 8902 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8903 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8904 | >
 8905 | using SM90_64x256x32_F16E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x256x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>;
 8907 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8908 | struct MMA_Traits<SM90_64x256x32_F16E5M2E5M2_RS_TN<scaleA, scaleB>>
 8909 | {
 8910 |   using ValTypeD = half_t;
 8911 |   using ValTypeA = float_e5m2_t;
 8912 |   using ValTypeB = float_e5m2_t;
 8913 |   using ValTypeC = half_t;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F16E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F16E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8915-8921

```text
 8915 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8917 |   using Shape_MNK = Shape<_64,_256,_32>;
 8918 |   using ThrID   = Layout<_128>;
 8919 |   using ALayout = GMMA::ALayout_64x32;
 8920 |   using BLayout = GMMA::ABLayout<256, 32>;
 8921 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8923-8924

```text
 8923 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8924 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8926-8940

```text
 8926 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8928 | template <
 8929 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8930 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8931 | >
 8932 | using SM90_64x256x32_F32E5M2E5M2_SS_TN = SM90::GMMA::MMA_64x256x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>;
 8934 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8935 | struct MMA_Traits<SM90_64x256x32_F32E5M2E5M2_SS_TN<scaleA, scaleB>>
 8936 | {
 8937 |   using ValTypeD = float;
 8938 |   using ValTypeA = float_e5m2_t;
 8939 |   using ValTypeB = float_e5m2_t;
 8940 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E5M2E5M2_SS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E5M2E5M2_SS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8942-8949

```text
 8942 |   using FrgTypeA = GMMA::smem_desc<GMMA::Major::K>;
 8943 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8945 |   using Shape_MNK = Shape<_64,_256,_32>;
 8946 |   using ThrID   = Layout<_128>;
 8947 |   using ALayout = GMMA::ABLayout< 64, 32>;
 8948 |   using BLayout = GMMA::ABLayout<256, 32>;
 8949 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8951-8952

```text
 8951 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8952 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8954-8968

```text
 8954 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8956 | template <
 8957 |   GMMA::ScaleIn  scaleA = GMMA::ScaleIn::One,
 8958 |   GMMA::ScaleIn  scaleB = GMMA::ScaleIn::One
 8959 | >
 8960 | using SM90_64x256x32_F32E5M2E5M2_RS_TN = SM90::GMMA::MMA_64x256x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>;
 8962 | template <GMMA::ScaleIn scaleA, GMMA::ScaleIn scaleB>
 8963 | struct MMA_Traits<SM90_64x256x32_F32E5M2E5M2_RS_TN<scaleA, scaleB>>
 8964 | {
 8965 |   using ValTypeD = float;
 8966 |   using ValTypeA = float_e5m2_t;
 8967 |   using ValTypeB = float_e5m2_t;
 8968 |   using ValTypeC = float;
```
**EN:** Specializes `MMA_Traits` for `SM90_64x256x32_F32E5M2E5M2_RS_TN<scaleA, scaleB`, encoding the operand types, tile shape, fragment representation, and layout metadata needed by higher-level MMA wrappers.
**CN:** 为 `SM90_64x256x32_F32E5M2E5M2_RS_TN<scaleA, scaleB` 特化 `MMA_Traits`，编码更高层 MMA 包装器所需的操作数类型、tile 形状、fragment 表示以及布局元数据。

### Lines 8970-8976

```text
 8970 |   using FrgTypeB = GMMA::smem_desc<GMMA::Major::K>;
 8972 |   using Shape_MNK = Shape<_64,_256,_32>;
 8973 |   using ThrID   = Layout<_128>;
 8974 |   using ALayout = GMMA::ALayout_64x32;
 8975 |   using BLayout = GMMA::ABLayout<256, 32>;
 8976 |   using CLayout = GMMA::CLayout_64x256;
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8978-8979

```text
 8978 |   GMMA::ScaleOut accumulate_ = GMMA::ScaleOut::One;
 8979 | };
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8981-8983

```text
 8981 | ////////////////////////////////////////////////////////////////////////////////////////////////////
 8983 | } // end namespace cute
```
**EN:** Defines trait-oriented helpers that let later code select optimized implementations through template specialization.
**CN:** 定义面向 traits 的辅助工具，使后续代码能够通过模板特化选择优化实现。

### Lines 8985-8987

```text
 8985 | #if defined(CUTE_SM90_EXTENDED_MMA_SHAPES_ENABLED)
 8986 | #include "mma_traits_sm90_gmma_ext.hpp"
 8987 | #endif
```
**EN:** Sets up the header dependencies for this file by importing `mma_traits_sm90_gmma_ext.hpp`.
**CN:** 通过引入 `mma_traits_sm90_gmma_ext.hpp` 为该文件建立头文件依赖。

## Key Concepts / 关键概念

- Hardware MMA traits and instruction mapping / 硬件 MMA traits 与指令映射
- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/pointer_flagged.hpp`
  - `cute/pointer_sparse.hpp`
  - `cute/swizzle.hpp`
  - `cute/tensor_impl.hpp`
  - `cute/arch/mma_sm90_desc.hpp`
  - `cute/arch/mma_sm90_gmma.hpp`
  - `cute/atom/mma_traits.hpp`
  - `cute/layout_composed.hpp`
  - `cute/numeric/integral_constant.hpp`
  - `mma_traits_sm90_gmma_ext.hpp`
- Primary symbols / 主要符号: `DescriptorIterator`, `smem_desc`, `MakeTensor`, `MMA_Traits`, `Engine`, `Layout`, `Type`, `Shape`
- Dependency role / 依赖角色: Depends on low-level architecture instruction wrappers and is consumed by higher-level `MMA_Atom`, tiling, and kernel-building code. / 依赖底层架构指令包装器，并被更高层的 `MMA_Atom`、平铺逻辑和内核构建代码消费。
