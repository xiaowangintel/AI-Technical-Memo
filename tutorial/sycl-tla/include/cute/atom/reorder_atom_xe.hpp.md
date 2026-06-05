# reorder_atom_xe.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/atom/reorder_atom_xe.hpp`
- Purpose (EN): Defines reorder atoms that describe data-layout permutations for architecture-aware data movement and tensor reordering.
- 作用 (CN): 定义重排原子，描述面向架构的数据布局置换，用于数据搬运和张量重排。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```text
    1 | /***************************************************************************************************
    2 | * Copyright (C) 2025 Intel Corporation, All rights reserved.
    3 | * SPDX-License-Identifier: BSD-3-Clause
    4 | *
    5 | * Redistribution and use in source and binary forms, with or without
    6 | * modification, are permitted provided that the following conditions are met:
    7 | *
    8 | * 1. Redistributions of source code must retain the above copyright notice, this
    9 | * list of conditions and the following disclaimer.
   10 | *
   11 | * 2. Redistributions in binary form must reproduce the above copyright notice,
   12 | * this list of conditions and the following disclaimer in the documentation
   13 | * and/or other materials provided with the distribution.
   14 | *
   15 | * 3. Neither the name of the copyright holder nor the names of its
   16 | * contributors may be used to endorse or promote products derived from
   17 | * this software without specific prior written permission.
   18 | *
   19 | * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
   20 | * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
   21 | * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
   22 | * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
   23 | * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
   24 | * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
   25 | * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
   26 | * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
   27 | * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
   28 | * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
   29 | *
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 30-32

```text
   30 | **************************************************************************************************/
   32 | #pragma once
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 34-34

```text
   34 | #include <cute/util/sycl_vec.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/util/sycl_vec.hpp`.
**CN:** 通过引入 `cute/util/sycl_vec.hpp` 为该文件建立头文件依赖。

### Lines 36-37

```text
   36 | namespace cute
   37 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 39-43

```text
   39 | // Reorder kinds.
   40 | //   "U" -> unit stride in registers
   41 | //   "V" -> VNNI format in registers
   42 | // e.g. UV means "unit to VNNI"
   43 | enum class ReorderKind : int { UU_Universal, UU, UV, VU, VV, Generic };
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 45-48

```text
   45 | template <ReorderKind kind, typename SrcType, typename DstType>
   46 | struct Xe_Reorder {
   47 |   using Unimplemented = void;
   48 | };
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 50-54

```text
   50 | // Check for the existence of an optimized reorder sequence.
   51 | template <ReorderKind kind, typename SrcType, typename DstType>
   52 | constexpr bool has_xe_optimized_reorder_impl(char) { return true; }
   53 | template <ReorderKind kind, typename SrcType, typename DstType, typename V = typename Xe_Reorder<kind, SrcType, DstType>::Unimplemented>
   54 | constexpr bool has_xe_optimized_reorder_impl(int) { return false; }
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 56-59

```text
   56 | template <ReorderKind kind, typename SrcType, typename DstType>
   57 | constexpr bool has_xe_optimized_reorder() {
   58 |   return has_xe_optimized_reorder_impl<kind, SrcType, DstType>(0);
   59 | }
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 61-114

```text
   61 | // Classify a subgroup-scope reorder.
   62 | template <int SV, int DV, class ReorderLayout>
   63 | constexpr ReorderKind classify_xe_reorder()
   64 | {
   65 |   constexpr int R = rank(ReorderLayout{});
   66 |   using Size0   = decltype(size<0>(ReorderLayout{}));
   67 |   using Stride0 = decltype(stride<0>(ReorderLayout{}));
   68 |   using _SV = Int<SV>;
   69 |   using _DV = Int<DV>;
   70 | 
   71 |   constexpr int VL = 16 * cute::max(SV, DV);
   72 |   if (is_constant_v<1, Stride0>) {
   73 |     // Unit stride -> unit stride. Require whole GRFs for both src and dst.
   74 |     if constexpr (is_constant_v<0, decltype(Size0{} % Int<VL>{})>) {
   75 |       return ReorderKind::UU;
   76 |     }
   77 | 
   78 |     // Fallback unit->unit when we do not have full GRFs.
   79 |     if constexpr (is_constant_v<0, decltype(Size0{} % _16{})>) {
   80 |       return ReorderKind::UU_Universal;
   81 |     }
   82 |   }
   83 | 
   84 |   // Check for VNNI reorders.
   85 |   // Fundamental assumption: values associated with a single VNNI block are contiguous in val space
   86 |   //   in both src and dst (even if only one of those is in VNNI format). All others take the generic path.
   87 |   if constexpr (R >= 2) {
   88 |     constexpr auto Modes01 = take<0,2>(ReorderLayout{});
   89 | 
   90 |     // Check for unit <-> VNNI reorders.
   91 |     //   unit->VNNI:  (_16, _DV, ...):(_DV, _1, ...)
   92 |     //   VNNI->unit:  (_SV, _16, ...):(_16, _1, ...)
   93 |     if constexpr (Modes01 == Layout<Shape<_16, _DV>, Stride<_DV, _1>>{}) {
   94 |       return ReorderKind::UV;
   95 |     }
   96 |     if constexpr (Modes01 == Layout<Shape<_SV, _16>, Stride<_16, _1>>{}) {
   97 |       return ReorderKind::VU;
   98 |     }
   99 | 
  100 |     // Check for VNNI -> VNNI reorders.
  101 |     //  SV > DV:  (_DV, _SV/DV, _16, ...):(_1, _DV*16, _DV, ...)
  102 |     //  DV > SV:  (_SV, _16, _DV/SV, ...):(_1, _SV*16, _SV, ...)
  103 |     if constexpr (R >= 3 && SV != DV) {
  104 |       constexpr auto Modes012 = take<0,3>(ReorderLayout());
  105 |       if constexpr (SV > DV && Modes012 == Layout<Shape<_DV, Int<SV/DV>, _16>, Stride<_1, Int<DV*16>, _DV>>{}) {
  106 |         return ReorderKind::VV;
  107 |       } else if constexpr (DV > SV && Modes012 == Layout<Shape<_SV, _16, Int<DV/SV>>, Stride<_1, Int<SV*16>, _SV>>{}) {
  108 |         return ReorderKind::VV;
  109 |       }
  110 |     }
  111 |   }
  112 | 
  113 |   return ReorderKind::Generic;
  114 | }
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 116-141

```text
  116 | template <class SType, class DType,
  117 |           class SLayout, class DLayout>
  118 | auto choose_xe_reorder_impl(SLayout const& slayout,   // (src thr, src val) -> coord
  119 |                             DLayout const& dlayout) { // (dst thr, dst val) -> coord
  120 |   // Calculate data transformation, interleaving WI-owned values:
  121 |   //  (thr0,val0) ... (thr15,val0), (thr0,val1), ..., (thr15,val1), ...
  122 |   auto rlayout = coalesce(composition(right_inverse(dlayout), slayout));          // src index -> dst index
  123 | 
  124 |   // Classify reorder type.
  125 |   constexpr auto SV = 32 / sizeof_bits_v<SType>;  // src elements per 32-bit channel
  126 |   constexpr auto DV = 32 / sizeof_bits_v<DType>;  // dst elements per 32-bit channel
  127 |   constexpr auto rclass = classify_xe_reorder<SV, DV, decltype(rlayout)>();
  128 | 
  129 |   if constexpr (has_xe_optimized_reorder<rclass, SType, DType>())
  130 |     return Xe_Reorder<rclass, SType, DType>{};
  131 |   else if constexpr (rclass == ReorderKind::UU_Universal)
  132 |     return Universal_Reorder_UU<SType, DType>{};
  133 |   else if constexpr (is_subbyte_v<SType>)
  134 |     return ReorderDispatchConvertRelayout{};
  135 |   else if constexpr (is_subbyte_v<DType>)
  136 |     return ReorderDispatchRelayoutConvert{};
  137 |   else if constexpr (!is_same_v<remove_cv_t<SType>, remove_cv_t<DType>>)
  138 |     return ReorderDispatchConvertRelayout{};
  139 |   else
  140 |     return ReorderDispatchXeGeneric{};
  141 | }
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 144-169

```text
  144 | // Copy a strided vector to a strided vector in GRF.
  145 | //   src and dst must each fit within a single register.
  146 | template <int simd, int sstride, int dstride, int sidx, int didx,
  147 |           class SEngine, class SLayoutWI,
  148 |           class DEngine, class DLayoutWI>
  149 | CUTE_HOST_DEVICE
  150 | void
  151 | reorder_span(Tensor<SEngine,SLayoutWI> const& src,
  152 |              Tensor<DEngine,DLayoutWI> &      dst)
  153 | {
  154 |   using namespace intel;
  155 |   using ValType = typename SEngine::element_type;
  156 |   using StorageType = storage_vector_t<ValType, 32>;
  157 |   constexpr int grf_elems = 64 / sizeof(ValType);
  158 |   const auto& sv = *recast_ptr<StorageType>(src.data() + ((sidx / grf_elems) * (grf_elems / sg_size)));
  159 |   auto&       dv = *recast_ptr<StorageType>(dst.data() + ((didx / grf_elems) * (grf_elems / sg_size)));
  160 |   constexpr auto soff = sidx % grf_elems;
  161 |   constexpr auto doff = didx % grf_elems;
  162 | #ifdef __SYCL_DEVICE_ONLY__
  163 |   asm (
  164 |     "mov (M1_NM, %2) %0(0,%5)<%3> %1(0,%6)<%4;1,0>"
  165 |     : "+rw"(dv)
  166 |     : "rw"(sv), "P"(simd), "P"(dstride), "P"(sstride), "P"(doff), "P"(soff)
  167 |   );
  168 | #endif
  169 | }
```
**EN:** Implements `recast_ptr`, preserving sparse or sub-byte semantics when converting a raw address to a CuTe-aware iterator or pointer type.
**CN:** 实现 `recast_ptr`，在把原始地址转换为 CuTe 感知的迭代器或指针类型时保留稀疏或子字节语义。

### Lines 171-239

```text
  171 | // Generic Xe reorders, supporting arbitrary layout changes, but not type conversions.
  172 | template <class SEngine, class SLayoutWI, class SLayout,
  173 |           class DEngine, class DLayoutWI, class DLayout>
  174 | CUTE_HOST_DEVICE
  175 | void
  176 | reorder_impl(ReorderDispatchXeGeneric  const&,
  177 |              Tensor<SEngine,SLayoutWI> const& src,       // WI fragment
  178 |              Tensor<DEngine,DLayoutWI> &      dst,       // WI fragment
  179 |              SLayout                   const&,           // (src thr, src val) -> coord
  180 |              DLayout                   const&)           // (dst thr, dst val) -> coord
  181 | {
  182 |   using SrcType = typename SEngine::element_type;
  183 |   using DstType = typename DEngine::element_type;
  184 |   static_assert(is_same_v<SrcType, DstType>, "No type conversions allowed on this path");
  185 | 
  186 |   static constexpr SLayout slayout{};
  187 |   static constexpr DLayout dlayout{};
  188 |   static constexpr auto rlayout = coalesce(composition(right_inverse(dlayout), slayout));      // src index -> dst index
  189 |   static constexpr auto ilayout = coalesce(composition(right_inverse(slayout), dlayout));      // dst index -> src index
  190 | 
  191 |   // Check for broadcast cases. This path allows a single src element to be copied
  192 |   //   to multiple dst elements (useful for grouped quantization cases).
  193 |   // Broadcast in (flattened) mode 0 requires special handling.
  194 |   static constexpr bool has_broadcast = (size(DLayoutWI{}) > size(SLayoutWI{}));
  195 |   static constexpr bool mode0_broadcast = has_broadcast && (stride<0>(ilayout) == _0{});
  196 | 
  197 |   // Decide whether to stride on src or dst, depending on which allows a longer vector length.
  198 |   static constexpr int elems_per_grf = 64 / sizeof(SrcType);
  199 |   static constexpr auto dstride = stride<0>(rlayout);
  200 |   static constexpr int sstride = mode0_broadcast ? 1 : stride<0>(ilayout);
  201 |   static constexpr int ds_vl = cute::min(32, cute::min(shape<0>(rlayout), elems_per_grf / dstride));
  202 |   static constexpr int ss_vl = cute::min(32, cute::min(shape<0>(ilayout), elems_per_grf / sstride));
  203 | 
  204 |   // Make dst live, to prevent compiler from inserting its own initialization.
  205 | #ifdef __SYCL_DEVICE_ONLY__
  206 |   using StorageType = intel::storage_vector_t<DstType, 32>;
  207 | 
  208 |   CUTE_UNROLL
  209 |   for (int i = 0; i < dst.size(); i += 4 / sizeof(DstType)) {
  210 |     auto &dv = *recast_ptr<StorageType>(dst.data() + i);
  211 |     asm("" : "=rw"(dv));
  212 |   }
  213 | #endif
  214 | 
  215 |   if constexpr (mode0_broadcast) {
  216 |     // Stride on dst, with mode-0 broadcast.
  217 |     for_each(make_seq<size<0>(ilayout)>{}, [&](auto j) {
  218 |       for_each(make_seq<size(SLayout{}) / ds_vl>{}, [&](auto i) {
  219 |         constexpr auto sidx = i * ds_vl;
  220 |         constexpr auto didx = rlayout(sidx) + j;
  221 |         reorder_span<ds_vl, 1, dstride, sidx, didx>(src, dst);
  222 |       });
  223 |     });
  224 |   } else if constexpr (ss_vl >= ds_vl || has_broadcast) {
  225 |     // Stride on src. For simplicity, take 1 GRF at a time.
  226 |     for_each(make_seq<size(SLayout{}) / ss_vl>{}, [&](auto i) {
  227 |       constexpr auto didx = i * ss_vl;
  228 |       constexpr auto sidx = ilayout(didx);
  229 |       reorder_span<ss_vl, sstride, 1, sidx, didx>(src, dst);
  230 |     });
  231 |   } else {
  232 |     // Stride on dst.
  233 |     for_each(make_seq<size(SLayout{}) / ds_vl>{}, [&](auto i) {
  234 |       constexpr auto sidx = i * ds_vl;
  235 |       constexpr auto didx = rlayout(sidx);
  236 |       reorder_span<ds_vl, 1, dstride, sidx, didx>(src, dst);
  237 |     });
  238 |   }
  239 | }
```
**EN:** Implements `recast_ptr`, preserving sparse or sub-byte semantics when converting a raw address to a CuTe-aware iterator or pointer type.
**CN:** 实现 `recast_ptr`，在把原始地址转换为 CuTe 感知的迭代器或指针类型时保留稀疏或子字节语义。

### Lines 241-257

```text
  241 | //
  242 | // Display utilities
  243 | //
  244 | CUTE_HOST_DEVICE
  245 | void
  246 | print(ReorderKind kind) {
  247 | #define CASE(x) if (kind == ReorderKind::x) print(#x);
  248 |   CASE(UU_Universal)
  249 |   CASE(UU)
  250 |   CASE(UV)
  251 |   CASE(VU)
  252 |   CASE(VV)
  253 |   CASE(Generic)
  254 | #undef CASE
  255 | }
  257 | } // end namespace cute
```
**EN:** Defines a reorder-related building block used to permute data layout between logical and hardware-preferred organizations.
**CN:** 定义与重排相关的构件，用于在逻辑布局与硬件偏好的组织方式之间进行置换。

### Lines 260-260

```text
  260 | #include <cute/arch/reorder_xe.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/arch/reorder_xe.hpp`.
**CN:** 通过引入 `cute/arch/reorder_xe.hpp` 为该文件建立头文件依赖。

## Key Concepts / 关键概念

- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- Intel Xe-specific behavior / Intel Xe 特定行为
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/util/sycl_vec.hpp`
  - `cute/arch/reorder_xe.hpp`
- Primary symbols / 主要符号: `Xe_Reorder`, `ReorderKind`, `ReorderLayout`, `SType`, `DType`, `SLayout`, `DLayout`, `SEngine`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
