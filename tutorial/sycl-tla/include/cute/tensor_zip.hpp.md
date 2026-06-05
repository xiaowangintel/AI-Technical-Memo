# tensor_zip.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/tensor_zip.hpp`
- Purpose (EN): Defines tensor views, tensor implementation details, subgroup-aware tensors, and tensor zipping/composition helpers.
- 作用 (CN): 定义张量视图、张量实现细节、子组感知张量以及张量拉链/组合辅助工具。

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
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 34-36

```text
   34 | #include <cute/config.hpp>           // CUTE_HOST_DEVICE
   35 | #include <cute/tensor_impl.hpp>      // cute::Tensor
   36 | #include <cute/container/tuple.hpp>  // cute::tuple
```
**EN:** Sets up the header dependencies for this file by importing `cute/config.hpp`, `cute/tensor_impl.hpp`, `cute/container/tuple.hpp`.
**CN:** 通过引入 `cute/config.hpp`, `cute/tensor_impl.hpp`, `cute/container/tuple.hpp` 为该文件建立头文件依赖。

### Lines 38-39

```text
   38 | namespace cute
   39 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 41-87

```text
   41 | // A tuple of Iterators that can be offset asymmetrically
   42 | // Note that this only accepts op+(tuple<Index...>) and op[tuple<Index...>]
   43 | //   where each iterator will be offset by its respective index only.
   44 | // READ-ONLY for now until cute::tuple can be constructed with references.
   45 | template <class... Iters>
   46 | struct ZipIterator
   47 | {
   48 |   using value_type   = cute::tuple<iter_value_t<Iters>...>;
   49 |   using element_type = cute::tuple<iter_element_t<Iters>...>;
   50 |   // NOTE: cute::tuple does not support constructions with references at the moment.
   51 |   //       Consider fixes and/or an implementation of std::forward_as_tuple.
   52 |   //       For now, use a cute::tuple of value_types instead, which makes this Iterator READ-ONLY.
   53 |   //using reference    = cute::tuple<iter_reference_t<Iters>...>;
   54 |   using reference  = value_type;
   55 | 
   56 |   ZipIterator() = delete;
   57 | 
   58 |   CUTE_HOST_DEVICE constexpr
   59 |   ZipIterator(Iters... iters)
   60 |     : iters_(iters...)
   61 |   {}
   62 | 
   63 |   CUTE_HOST_DEVICE constexpr
   64 |   ZipIterator(cute::tuple<Iters...> const& iters)
   65 |     : iters_(iters)
   66 |   {}
   67 | 
   68 |   CUTE_HOST_DEVICE constexpr
   69 |   reference operator*() const {
   70 |     return cute::apply(iters_, [](auto&&... args) { return reference(*args...); });
   71 |   }
   72 | 
   73 |   template <class... Index>
   74 |   CUTE_HOST_DEVICE constexpr
   75 |   ZipIterator operator+(cute::tuple<Index...> const& idxs) const {
   76 |     static_assert(sizeof...(Index) == sizeof...(Iters), "Expect same number of offsets as iterators.");
   77 |     return cute::transform(iters_, idxs, [](auto&& iter, auto&& idx) { return iter + idx; });
   78 |   }
   79 | 
   80 |   template <class... Index>
   81 |   CUTE_HOST_DEVICE constexpr
   82 |   reference operator[](cute::tuple<Index...> const& idxs) const {
   83 |     return *(*this + idxs);
   84 |   }
   85 | 
   86 |   cute::tuple<Iters...> iters_;
   87 | };
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 89-99

```text
   89 | //------------------------------------------------------------------------------
   90 | // type traits
   92 | template <class... Iters>
   93 | struct is_rmem<ZipIterator<Iters...>> : conjunction<is_rmem<Iters>...> {};
   94 | template <class... Iters>
   95 | struct is_smem<ZipIterator<Iters...>> : conjunction<is_smem<Iters>...> {};
   96 | template <class... Iters>
   97 | struct is_gmem<ZipIterator<Iters...>> : conjunction<is_gmem<Iters>...> {};
   98 | template <class... Iters>                                                   
   99 | struct is_tmem<ZipIterator<Iters...>> : conjunction<is_tmem<Iters>...> {};  
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 101-147

```text
  101 | // A tuple of Layouts that operates on each Layout symmetrically
  102 | // The Layouts need to have compatible shapes and ranks.
  103 | // The ZipLayout presents the intersection of the domain of its component Layouts.
  104 | //   E.g. all Layouts accept 1D coords and ZipLayout does as well.
  105 | // The ZipLayout returns the union of the codomain of its component Layouts.
  106 | //   E.g. all Layouts return an integer so ZipLayout returns a tuple of integers.
  107 | template <class... Layouts>
  108 | struct ZipLayout
  109 | {
  110 |   static constexpr int rank = (int(0) | ... | Layouts::rank);
  111 | 
  112 |   static_assert((is_layout<Layouts>::value && ...), "All template parameters must be layouts");
  113 |   static_assert(((Layouts::rank == rank) && ...),   "All layouts must have the same rank");
  114 | 
  115 |   CUTE_HOST_DEVICE constexpr
  116 |   ZipLayout(Layouts const&... layouts)
  117 |     : layouts_(layouts...)
  118 |   {}
  119 | 
  120 |   CUTE_HOST_DEVICE constexpr
  121 |   ZipLayout(cute::tuple<Layouts...> const& layouts)
  122 |     : layouts_(layouts)
  123 |   {}
  124 | 
  125 |   template <class Coord>
  126 |   CUTE_HOST_DEVICE constexpr
  127 |   auto
  128 |   operator()(Coord const& coord) const {
  129 |     if constexpr (has_underscore<Coord>::value) {
  130 |       return ZipLayout(cute::transform(layouts_, [&] (auto layout) { return layout(coord); }));
  131 |     } else {
  132 |       return cute::transform(layouts_, [&] (auto layout) { return layout(coord); });
  133 |     }
  134 | 
  135 |     CUTE_GCC_UNREACHABLE;
  136 |   }
  137 | 
  138 |   // op() convenience function for multi-dimensional coordinates
  139 |   template <class Coord0, class Coord1, class... Coords>
  140 |   CUTE_HOST_DEVICE constexpr
  141 |   decltype(auto)
  142 |   operator()(Coord0 const& c0, Coord1 const& c1, Coords const&... cs) const {
  143 |     return operator()(make_coord(c0,c1,cs...));
  144 |   }
  145 | 
  146 |   cute::tuple<Layouts...> layouts_;
  147 | };
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 149-150

```text
  149 | template <class... Layouts>
  150 | struct is_layout<ZipLayout<Layouts...>> : true_type {};
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 152-163

```text
  152 | //
  153 | // make_zip_tensor and unzip_tensor
  154 | //
  156 | template <class... Engines, class... Layouts>
  157 | CUTE_HOST_DEVICE constexpr
  158 | auto
  159 | make_zip_tensor(Tensor<Engines,Layouts> const&... tensors)
  160 | {
  161 |   return make_tensor(ZipIterator(tensors.data()...),
  162 |                      ZipLayout(tensors.layout()...));
  163 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 165-172

```text
  165 | template <class Engine, class Layout>
  166 | CUTE_HOST_DEVICE constexpr
  167 | auto
  168 | unzip_tensor(Tensor<Engine,Layout> const& tensor)
  169 | {
  170 |   return cute::transform(tensor.data().iters_, tensor.layout().layouts_,
  171 |                          [](auto iter, auto layout) { return make_tensor(iter, layout); });
  172 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 174-184

```text
  174 | //
  175 | // Utilities
  176 | //
  178 | template <int... Is, class... Layouts>
  179 | CUTE_HOST_DEVICE constexpr
  180 | auto
  181 | rank(ZipLayout<Layouts...> const& layouts)
  182 | {
  183 |   return rank<Is...>(get<0>(layouts.layouts_));
  184 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 186-192

```text
  186 | template <int... Is, class... Layouts>
  187 | CUTE_HOST_DEVICE constexpr
  188 | auto
  189 | size(ZipLayout<Layouts...> const& layouts)
  190 | {
  191 |   return size<Is...>(get<0>(layouts.layouts_));
  192 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 194-206

```text
  194 | //
  195 | // Manipulation
  196 | //
  198 | // Extend each component layout to rank-N by appending Layout @a x.
  199 | template <int N, class... Layouts, class ShapeX = _1, class StrideX = _0>
  200 | CUTE_HOST_DEVICE constexpr
  201 | auto
  202 | append(ZipLayout<Layouts...>  const& layouts,
  203 |        Layout<ShapeX,StrideX> const& x = {})
  204 | {
  205 |   return ZipLayout(cute::transform(layouts.layouts_, [&](auto t){ return append<N>(t, x); }));
  206 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 208-216

```text
  208 | // Extend each component layout to rank-N by prepending Layout @a x.
  209 | template <int N, class... Layouts, class ShapeX = _1, class StrideX = _0>
  210 | CUTE_HOST_DEVICE constexpr
  211 | auto
  212 | prepend(ZipLayout<Layouts...>  const& layouts,
  213 |         Layout<ShapeX,StrideX> const& x = {})
  214 | {
  215 |   return ZipLayout(cute::transform(layouts.layouts_, [&](auto t){ return prepend<N>(t, x); }));
  216 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 218-225

```text
  218 | template <class... Layouts, class Tiler>
  219 | CUTE_HOST_DEVICE constexpr
  220 | auto
  221 | logical_divide(ZipLayout<Layouts...> const& layouts,
  222 |                Tiler                 const& tiler)
  223 | {
  224 |   return ZipLayout(cute::transform(layouts.layouts_, [&](auto t){ return logical_divide(t, tiler); }));
  225 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 227-234

```text
  227 | template <class... Layouts, class Tiler>
  228 | CUTE_HOST_DEVICE constexpr
  229 | auto
  230 | zipped_divide(ZipLayout<Layouts...> const& layouts,
  231 |               Tiler                 const& tiler)
  232 | {
  233 |   return ZipLayout(cute::transform(layouts.layouts_, [&](auto t){ return zipped_divide(t, tiler); }));
  234 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 236-246

```text
  236 | // Return <SlicedZipLayout, ZipOffsets> by calling slice_and_offset and all component layouts.
  237 | template <class Coord, class... Layouts>
  238 | CUTE_HOST_DEVICE constexpr
  239 | auto
  240 | slice_and_offset(Coord const& c, ZipLayout<Layouts...> const& layouts)
  241 | {
  242 |   auto result = cute::zip(cute::transform(layouts.layouts_, [&c](auto const& layout) { return slice_and_offset(c, layout); }));
  243 |   return cute::make_tuple(ZipLayout(get<0>(result)), get<1>(result));
  244 | }
  246 | } // end namespace cute
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

## Key Concepts / 关键概念

- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Intel Xe-specific behavior / Intel Xe 特定行为

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/tensor_impl.hpp`
  - `cute/container/tuple.hpp`
- Primary symbols / 主要符号: `ZipIterator`, `is_rmem`, `is_smem`, `is_gmem`, `is_tmem`, `ZipLayout`, `is_layout`, `Coord`
- Dependency role / 依赖角色: Builds on tuple/shape/stride primitives and is used by tensor, tiling, copy, and compute abstractions that need coordinate mappings. / 构建于 tuple/shape/stride 原语之上，供需要坐标映射的张量、平铺、拷贝和计算抽象使用。
