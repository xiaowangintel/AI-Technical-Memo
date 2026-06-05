# layout_composed.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/layout_composed.hpp`
- Purpose (EN): Defines helpers for composing layouts, shaping nested mappings, and building derived coordinate transforms from existing layouts.
- 作用 (CN): 定义布局组合辅助工具，用于塑造嵌套映射，并基于已有布局构建派生坐标变换。

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
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 33-35

```text
   33 | #include <cute/config.hpp>                     // CUTE_HOST_DEVICE, CUTE_GCC_UNREACHABLE
   34 | #include <cute/layout.hpp>                     // cute::tuple
   35 | #include <cute/numeric/integral_constant.hpp>  // cute::true_type, cute::false_type, cute::Int
```
**EN:** Sets up the header dependencies for this file by importing `cute/config.hpp`, `cute/layout.hpp`, `cute/numeric/integral_constant.hpp`.
**CN:** 通过引入 `cute/config.hpp`, `cute/layout.hpp`, `cute/numeric/integral_constant.hpp` 为该文件建立头文件依赖。

### Lines 37-50

```text
   37 | /* This implements a ComposedLayout of the form
   38 |  *   LayoutA o Offset o LayoutB
   39 |  * and is useful in cases where composition() does not or cannot apply to LayoutA and LayoutB.
   40 |  * For example, when the "divisibility condition" is violated in composition(LayoutA, LayoutB).
   41 |  *
   42 |  * This ComposedLayout provides similar functionality to Layout including tiling, partitioning,
   43 |  * coordinate-to-index mapping and layout manipulations, but is not considered a "normal" layout.
   44 |  * For example, this layout provides shape() and size() functions, but does not provide stride() functions.
   45 |  * Mostly, the similar functionality is accomplished by applying each operation to LayoutB only
   46 |  * as LayoutB defines the domain.
   47 |  */
   49 | namespace cute
   50 | {
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 52-191

```text
   52 | // A Layout of non-trivially composable functions: F o I o L
   53 | template <class LayoutA, class Offset, class LayoutB>
   54 | struct ComposedLayout : private cute::tuple<LayoutA, Offset, LayoutB>  // EBO for static layouts
   55 | {
   56 |   CUTE_HOST_DEVICE constexpr
   57 |   ComposedLayout(LayoutA const& layoutA = {},
   58 |                  Offset  const& offset  = {},
   59 |                  LayoutB const& layoutB = {})
   60 |       : cute::tuple<LayoutA, Offset, LayoutB>(layoutA, offset, layoutB)
   61 |   {}
   62 | 
   63 |   //
   64 |   // Accessors
   65 |   //
   66 | 
   67 |   static constexpr int rank  = LayoutB::rank;
   68 | 
   69 |   CUTE_HOST_DEVICE constexpr
   70 |   decltype(auto)
   71 |   layout_a() const {
   72 |     return get<0>(static_cast<cute::tuple<LayoutA, Offset, LayoutB> const&>(*this));
   73 |   }
   74 | 
   75 |   CUTE_HOST_DEVICE constexpr
   76 |   decltype(auto)
   77 |   offset() const {
   78 |     return get<1>(static_cast<cute::tuple<LayoutA, Offset, LayoutB> const&>(*this));
   79 |   }
   80 | 
   81 |   CUTE_HOST_DEVICE constexpr
   82 |   decltype(auto)
   83 |   layout_b() const {
   84 |     return get<2>(static_cast<cute::tuple<LayoutA, Offset, LayoutB> const&>(*this));
   85 |   }
   86 | 
   87 |   CUTE_HOST_DEVICE constexpr
   88 |   decltype(auto)
   89 |   layout() const {
   90 |     return *this;
   91 |   }
   92 | 
   93 |   CUTE_HOST_DEVICE constexpr
   94 |   decltype(auto)
   95 |   shape() const {
   96 |     return layout_b().shape();
   97 |   }
   98 | 
   99 |   // Doesn't really make sense to ask for the strides of this "layout"
  100 |   CUTE_HOST_DEVICE constexpr
  101 |   decltype(auto)
  102 |   stride() const = delete;
  103 | 
  104 |   //
  105 |   // Mappings
  106 |   //
  107 | 
  108 |   // Map a logical coordinate to a linear index (Coord has no Underscore slice operators)
  109 |   // OR
  110 |   // Slice the layout and return the sublayout (Coord has an Underscore slice op)
  111 |   template <class Coord>
  112 |   CUTE_HOST_DEVICE constexpr
  113 |   auto
  114 |   operator()(Coord const& coord) const {
  115 |     if constexpr (has_underscore<Coord>::value) {
  116 |       return slice(coord, *this);
  117 |     } else {
  118 |       return layout_a()(offset() + layout_b()(coord));    // (A o O o B)(c)
  119 |     }
  120 | 
  121 |     CUTE_GCC_UNREACHABLE;
  122 |   }
  123 | 
  124 |   // Convenience function for multi-dimensional coordinates
  125 |   template <class Coord0, class Coord1, class... Coords>
  126 |   CUTE_HOST_DEVICE constexpr
  127 |   auto
  128 |   operator()(Coord0 const& c0, Coord1 const& c1, Coords const&... cs) const {
  129 |     return operator()(make_coord(c0,c1,cs...));
  130 |   }
  131 | 
  132 |   //
  133 |   // Compose
  134 |   //
  135 | 
  136 |   template <class OtherLayout>
  137 |   CUTE_HOST_DEVICE constexpr
  138 |   auto
  139 |   compose(OtherLayout const& other) const {
  140 |     return composition(*this, other);
  141 |   }
  142 | 
  143 |   template <class... Layouts>
  144 |   CUTE_HOST_DEVICE constexpr
  145 |   auto
  146 |   compose(Layouts const&... layouts) const {
  147 |     return composition(*this, make_tile(layouts...));
  148 |   }
  149 | 
  150 |   template <class OtherShape>
  151 |   CUTE_HOST_DEVICE constexpr
  152 |   auto
  153 |   with_shape(OtherShape const& shape) const {
  154 |     return composition(*this, make_layout(shape));
  155 |   }
  156 | 
  157 |   template <class... Shapes>
  158 |   CUTE_HOST_DEVICE constexpr
  159 |   auto
  160 |   with_shape(Shapes const&... shapes) const {
  161 |     return composition(*this, make_layout(make_shape(shapes...)));
  162 |   }
  163 | 
  164 |   //
  165 |   // Tile
  166 |   //
  167 | 
  168 |   template <class OtherLayout>
  169 |   CUTE_HOST_DEVICE constexpr
  170 |   auto
  171 |   tile(OtherLayout const& other) const {
  172 |     return tiled_divide(*this, other);
  173 |   }
  174 | 
  175 |   template <class... Layouts>
  176 |   CUTE_HOST_DEVICE constexpr
  177 |   auto
  178 |   tile(Layouts const&... layouts) const {
  179 |     return tiled_divide(*this, make_tile(layouts...));
  180 |   }
  181 | 
  182 |   // Equality, return a static or dynamic boolean
  183 |   template <class... Args>
  184 |   CUTE_HOST_DEVICE constexpr
  185 |   auto
  186 |   operator==(ComposedLayout<Args...> const& other) const {
  187 |     return this->layout_a() == other.layout_a() &&
  188 |            this->layout_b() == other.layout_b() &&
  189 |            this->offset()   == other.offset();
  190 |   }
  191 | };
```
**EN:** Defines the main `Layout` interface, exposing shape/stride accessors, coordinate mapping, composition, and related layout algebra operations.
**CN:** 定义核心 `Layout` 接口，提供 shape/stride 访问、坐标映射、组合以及相关布局代数操作。

### Lines 193-194

```text
  193 | template <class A, class O, class B>
  194 | struct is_layout<ComposedLayout<A,O,B>> : true_type {};
```
**EN:** Defines `is_layout` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `is_layout` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 196-199

```text
  196 | template <class T>
  197 | struct is_composed_layout : false_type {};
  198 | template <class A, class O, class B>
  199 | struct is_composed_layout<ComposedLayout<A,O,B>> : true_type {};
```
**EN:** Defines `is_composed_layout` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `is_composed_layout` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 201-213

```text
  201 | //
  202 | // Constructors
  203 | //
  205 | template <class LayoutA, class Offset, class LayoutB>
  206 | CUTE_HOST_DEVICE constexpr
  207 | auto
  208 | make_composed_layout(LayoutA const& layoutA,
  209 |                      Offset  const& offset,
  210 |                      LayoutB const& layoutB)
  211 | {
  212 |   return ComposedLayout<LayoutA, Offset, LayoutB>{layoutA, offset, layoutB};
  213 | }
```
**EN:** Defines `LayoutA` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `LayoutA` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 215-226

```text
  215 | //
  216 | // Utilities
  217 | //
  219 | // Return the layout of a mode
  220 | template <int... Is, class A, class O, class B>
  221 | CUTE_HOST_DEVICE constexpr
  222 | decltype(auto)
  223 | layout(ComposedLayout<A,O,B> const& clayout)
  224 | {
  225 |   return composition(clayout.layout_a(), clayout.offset(), layout<Is...>(clayout.layout_b()));
  226 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 228-235

```text
  228 | // Return the shape of a mode
  229 | template <int... Is, class A, class O, class B>
  230 | CUTE_HOST_DEVICE constexpr
  231 | decltype(auto)
  232 | shape(ComposedLayout<A,O,B> const& layout)
  233 | {
  234 |   return shape<Is...>(layout.layout_b());
  235 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 237-241

```text
  237 | // Doesn't make sense to directly ask for the strides of this "layout"
  238 | template <int... Is, class Fn, class O, class Layout>
  239 | CUTE_HOST_DEVICE constexpr
  240 | decltype(auto)
  241 | stride(ComposedLayout<Fn,O,Layout> const& layout) = delete;
```
**EN:** Defines `Fn` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `Fn` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 243-250

```text
  243 | // Return the number of elements in a mode
  244 | template <int... Is, class A, class O, class B>
  245 | CUTE_HOST_DEVICE constexpr
  246 | decltype(auto)
  247 | size(ComposedLayout<A,O,B> const& layout)
  248 | {
  249 |   return size<Is...>(layout.layout_b());
  250 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 252-259

```text
  252 | // Return the number of modes
  253 | template <int... Is, class A, class O, class B>
  254 | CUTE_HOST_DEVICE constexpr
  255 | auto
  256 | rank(ComposedLayout<A,O,B> const& layout)
  257 | {
  258 |   return rank<Is...>(layout.layout_b());
  259 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 261-268

```text
  261 | // Return the depth of the layout
  262 | template <int... Is, class A, class O, class B>
  263 | CUTE_HOST_DEVICE constexpr
  264 | auto
  265 | depth(ComposedLayout<A,O,B> const& layout)
  266 | {
  267 |   return depth<Is...>(layout.layout_b());
  268 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 270-277

```text
  270 | // Return the codomain size of a mode
  271 | template <int... Is, class A, class O, class B>
  272 | CUTE_HOST_DEVICE constexpr
  273 | auto
  274 | cosize(ComposedLayout<A,O,B> const& layout)
  275 | {
  276 |   return cosize<Is...>(layout.layout_b());
  277 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 279-289

```text
  279 | //
  280 | // Operations to manipulate Layouts like a tuple of pairs
  281 | //
  283 | template <size_t I, class A, class O, class B>
  284 | CUTE_HOST_DEVICE constexpr
  285 | auto
  286 | get(ComposedLayout<A,O,B> const& a)
  287 | {
  288 |   return composition(a.layout_a(), a.offset(), get<I>(a.layout_b()));
  289 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 291-297

```text
  291 | template <int Begin, int End, class A, class O, class B>
  292 | CUTE_HOST_DEVICE constexpr
  293 | auto
  294 | take(ComposedLayout<A,O,B> const& a)
  295 | {
  296 |   return composition(a.layout_a(), a.offset(), take<Begin,End>(a.layout_b()));
  297 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 299-305

```text
  299 | template <class A, class O, class B>
  300 | CUTE_HOST_DEVICE constexpr
  301 | auto
  302 | flatten(ComposedLayout<A,O,B> const& a)
  303 | {
  304 |   return composition(a.layout_a(), a.offset(), flatten(a.layout_b()));
  305 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 307-313

```text
  307 | template <int N, class A, class O, class B, class X>
  308 | CUTE_HOST_DEVICE constexpr
  309 | auto
  310 | append(ComposedLayout<A,O,B> const& a, X const& x)
  311 | {
  312 |   return composition(a.layout_a(), a.offset(), append<N>(a.layout_b(), x));
  313 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 315-321

```text
  315 | template <int Begin, int End, class A, class O, class B>
  316 | CUTE_HOST_DEVICE constexpr
  317 | auto
  318 | group(ComposedLayout<A,O,B> const& a)
  319 | {
  320 |   return composition(a.layout_a(), a.offset(), group<Begin,End>(a.layout_b()));
  321 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 323-334

```text
  323 | //
  324 | // Slice a ComposedLayout
  325 | //
  327 | template <class Coord, class A, class O, class B>
  328 | CUTE_HOST_DEVICE constexpr
  329 | auto
  330 | slice_and_offset(Coord const& coord, ComposedLayout<A,O,B> const& layout)
  331 | {
  332 |   auto [slice, offset] = slice_and_offset(coord, layout.layout_b());
  333 |   return cute::make_tuple(ComposedLayout{layout.layout_a(), layout.offset() + offset, slice}, Int<0>{});
  334 | }
```
**EN:** Defines `Coord` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `Coord` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 336-342

```text
  336 | template <class Coord, class A, class O, class B>
  337 | CUTE_HOST_DEVICE constexpr
  338 | auto
  339 | slice(Coord const& coord, ComposedLayout<A,O,B> const& layout)
  340 | {
  341 |   return get<0>(slice_and_offset(coord, layout));
  342 | }
```
**EN:** Defines `Coord` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `Coord` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 344-352

```text
  344 | // Compute a pointer offset and (potentially modified) layout from a coordinate
  345 | // For composed layout tensors the offset is accumulated in the layout itself while pointer is not updated
  346 | template <class Coord, class A, class O, class B>
  347 | CUTE_HOST_DEVICE constexpr
  348 | auto
  349 | domain_offset(Coord const& coord, ComposedLayout<A,O,B> const& layout)
  350 | {
  351 |   return cute::make_tuple(ComposedLayout{layout.layout_a(), layout.offset() + layout.layout_b()(coord), layout.layout_b()}, Int<0>{});
  352 | }
```
**EN:** Defines `Coord` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `Coord` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 354-368

```text
  354 | //
  355 | // composition
  356 | //
  358 | template <class LayoutA,
  359 |           class Offset,
  360 |           class LayoutB>
  361 | CUTE_HOST_DEVICE constexpr
  362 | auto
  363 | composition(LayoutA const& layoutA,
  364 |             Offset  const& offset,
  365 |             LayoutB const& layoutB)
  366 | {
  367 |   return ComposedLayout<LayoutA, Offset, LayoutB>{layoutA, offset, layoutB};
  368 | }
```
**EN:** Defines `LayoutA` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `LayoutA` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 370-377

```text
  370 | template <class A, class O, class B, class Tiler>
  371 | CUTE_HOST_DEVICE constexpr
  372 | auto
  373 | composition(ComposedLayout<A,O,B> const& a,
  374 |             Tiler                 const& b)
  375 | {
  376 |   return composition(a.layout_a(), a.offset(), composition(a.layout_b(), b));
  377 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 379-389

```text
  379 | template <class ShapeA, class StrideA,
  380 |           class A, class O, class B>
  381 | CUTE_HOST_DEVICE constexpr
  382 | auto
  383 | composition(Layout<ShapeA,StrideA> const& a,
  384 |             ComposedLayout<A,O,B>  const& b)
  385 | {
  386 |   CUTE_STATIC_ASSERT_V(b.offset() == Int<0>{}, "Require offset == 0.");
  387 | 
  388 |   return composition(composition(a, b.layout_a()), b.layout_b());
  389 | }
```
**EN:** Defines `ShapeA` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `ShapeA` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 391-401

```text
  391 | //
  392 | // complement
  393 | //
  395 | template <class A, class O, class B, class CoTarget>
  396 | CUTE_HOST_DEVICE constexpr
  397 | auto
  398 | complement(ComposedLayout<A,O,B> const& layout, CoTarget const& cotarget)
  399 | {
  400 |   return complement(layout.layout_b(), cotarget);
  401 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 403-409

```text
  403 | template <class A, class O, class B>
  404 | CUTE_HOST_DEVICE constexpr
  405 | auto
  406 | complement(ComposedLayout<A,O,B> const& layout)
  407 | {
  408 |   return complement(layout, cosize(layout));
  409 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 411-421

```text
  411 | //
  412 | // inverse
  413 | //
  415 | template <class A, class O, class B>
  416 | CUTE_HOST_DEVICE constexpr
  417 | auto
  418 | right_inverse(ComposedLayout<A,O,B> const& layout)
  419 | {
  420 |   return composition(right_inverse(layout.layout_b()), right_inverse(layout.offset()), right_inverse(layout.layout_a()));
  421 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 423-429

```text
  423 | template <class A, class O, class B>
  424 | CUTE_HOST_DEVICE constexpr
  425 | auto
  426 | left_inverse(ComposedLayout<A,O,B> const& layout)
  427 | {
  428 |   return composition(left_inverse(layout.layout_b()), left_inverse(layout.offset()), left_inverse(layout.layout_a()));
  429 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 431-441

```text
  431 | //
  432 | // Other operations
  433 | //
  435 | template <class A, class O, class B>
  436 | CUTE_HOST_DEVICE constexpr
  437 | auto
  438 | zip(ComposedLayout<A,O,B> const& a)
  439 | {
  440 |   return composition(a.layout_a(), a.offset(), zip(a.layout_b()));
  441 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 443-452

```text
  443 | // Partitions
  445 | template <class A, class O, class B, class Tiler>
  446 | CUTE_HOST_DEVICE constexpr
  447 | auto
  448 | logical_divide(ComposedLayout<A,O,B> const& a,
  449 |                Tiler                 const& b)
  450 | {
  451 |   return composition(a.layout_a(), a.offset(), logical_divide(a.layout_b(), b));
  452 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 454-461

```text
  454 | template <class A, class O, class B, class Tiler>
  455 | CUTE_HOST_DEVICE constexpr
  456 | auto
  457 | tile_unzip(ComposedLayout<A,O,B> const& a,
  458 |            Tiler                 const& b)
  459 | {
  460 |   return composition(a.layout_a(), a.offset(), tile_unzip(a.layout_b(), b));
  461 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 463-470

```text
  463 | template <class A, class O, class B, class Tiler>
  464 | CUTE_HOST_DEVICE constexpr
  465 | auto
  466 | tiled_divide(ComposedLayout<A,O,B> const& a,
  467 |              Tiler                 const& b)
  468 | {
  469 |   return composition(a.layout_a(), a.offset(), tiled_divide(a.layout_b(), b));
  470 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 472-479

```text
  472 | template <class A, class O, class B, class Tiler>
  473 | CUTE_HOST_DEVICE constexpr
  474 | auto
  475 | zipped_divide(ComposedLayout<A,O,B> const& a,
  476 |               Tiler                 const& b)
  477 | {
  478 |   return composition(a.layout_a(), a.offset(), zipped_divide(a.layout_b(), b));
  479 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 481-488

```text
  481 | template <class A, class O, class B, class Tiler>
  482 | CUTE_HOST_DEVICE constexpr
  483 | auto
  484 | flat_divide(ComposedLayout<A,O,B> const& a,
  485 |             Tiler                 const& b)
  486 | {
  487 |   return composition(a.layout_a(), a.offset(), flat_divide(a.layout_b(), b));
  488 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 490-497

```text
  490 | template <class A, class O, class B, class Tiler>
  491 | CUTE_HOST_DEVICE constexpr
  492 | auto
  493 | logical_product(ComposedLayout<A,O,B> const& a,
  494 |                 Tiler                 const& b)
  495 | {
  496 |   return composition(a.layout_a(), a.offset(), logical_product(a.layout_b(), b));
  497 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 499-506

```text
  499 | template <class A, class O, class B, class Tiler>
  500 | CUTE_HOST_DEVICE constexpr
  501 | auto
  502 | zipped_product(ComposedLayout<A,O,B> const& a,
  503 |                Tiler                 const& b)
  504 | {
  505 |   return composition(a.layout_a(), a.offset(), zipped_product(a.layout_b(), b));
  506 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 508-515

```text
  508 | template <class A, class O, class B, class Tiler>
  509 | CUTE_HOST_DEVICE constexpr
  510 | auto
  511 | tiled_product(ComposedLayout<A,O,B> const& a,
  512 |               Tiler                 const& b)
  513 | {
  514 |   return composition(a.layout_a(), a.offset(), tiled_product(a.layout_b(), b));
  515 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 517-524

```text
  517 | template <class A, class O, class B, class Tiler>
  518 | CUTE_HOST_DEVICE constexpr
  519 | auto
  520 | flat_product(ComposedLayout<A,O,B> const& a,
  521 |              Tiler                 const& b)
  522 | {
  523 |   return composition(a.layout_a(), a.offset(), flat_product(a.layout_b(), b));
  524 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 526-533

```text
  526 | template <class A, class O, class B, class Tiler>
  527 | CUTE_HOST_DEVICE constexpr
  528 | auto
  529 | blocked_product(ComposedLayout<A,O,B> const& a,
  530 |                 Tiler                 const& b)
  531 | {
  532 |   return composition(a.layout_a(), a.offset(), blocked_product(a.layout_b(), b));
  533 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 535-542

```text
  535 | template <class A, class O, class B, class Tiler>
  536 | CUTE_HOST_DEVICE constexpr
  537 | auto
  538 | raked_product(ComposedLayout<A,O,B> const& a,
  539 |               Tiler                 const& b)
  540 | {
  541 |   return composition(a.layout_a(), a.offset(), raked_product(a.layout_b(), b));
  542 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 544-553

```text
  544 | template <class A, class O, class B,
  545 |           class Shape, class ModeOrder = GenColMajor>
  546 | CUTE_HOST_DEVICE constexpr
  547 | auto
  548 | tile_to_shape(ComposedLayout<A,O,B> const& layout,
  549 |               Shape                 const& trg_shape,
  550 |               ModeOrder             const& ord_shape = {})
  551 | {
  552 |   return composition(layout.layout_a(), layout.offset(), tile_to_shape(layout.layout_b(), trg_shape, ord_shape));
  553 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 555-562

```text
  555 | template <class A, class O, class B,
  556 |           class Shape>
  557 | CUTE_HOST_DEVICE constexpr
  558 | auto
  559 | filter(ComposedLayout<A,O,B> const& layout, Shape const& trg_profile)
  560 | {
  561 |   return composition(layout.layout_a(), layout.offset(), filter(layout.layout_b(), trg_profile));
  562 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 564-570

```text
  564 | template <class A, class O, class B>
  565 | CUTE_HOST_DEVICE constexpr
  566 | auto
  567 | coalesce(ComposedLayout<A,O,B> const& layout)
  568 | {
  569 |   return composition(layout.layout_a(), layout.offset(), coalesce(layout.layout_b()));
  570 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 572-578

```text
  572 | template <class A, class O, class B, class Shape>
  573 | CUTE_HOST_DEVICE constexpr
  574 | auto
  575 | coalesce(ComposedLayout<A,O,B> const& layout, Shape const& trg_profile)
  576 | {
  577 |   return composition(layout.layout_a(), layout.offset(), coalesce(layout.layout_b(), trg_profile));
  578 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 581-591

```text
  581 | //
  582 | // Upcast and Downcast
  583 | //
  585 | template <int N, class A, class O, class B>
  586 | CUTE_HOST_DEVICE constexpr
  587 | auto
  588 | upcast(ComposedLayout<A,O,B> const& layout)
  589 | {
  590 |   return composition(upcast<N>(layout.layout_a()), upcast<N>(layout.offset()), upcast<N>(layout.layout_b()));
  591 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 593-599

```text
  593 | template <int N, class A, class O, class B>
  594 | CUTE_HOST_DEVICE constexpr
  595 | auto
  596 | downcast(ComposedLayout<A,O,B> const& layout)
  597 | {
  598 |   return composition(downcast<N>(layout.layout_a()), downcast<N>(layout.offset()), downcast<N>(layout.layout_b()));
  599 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 602-622

```text
  602 | template <class OldType, class NewType,
  603 |           class A, class O, class B>
  604 | CUTE_HOST_DEVICE constexpr
  605 | auto
  606 | recast_layout(ComposedLayout<A,O,B> const& layout)
  607 | {
  608 |   using scale = decltype(trait_ratio(sizeof_bits<NewType>{}, sizeof_bits<OldType>{}));
  609 |   if constexpr (scale::num == 1 && scale::den == 1) {
  610 |     return layout;
  611 |   }
  612 |   else if constexpr (scale::num == 1) {
  613 |     return downcast<scale::den>(layout);
  614 |   }
  615 |   else if constexpr (scale::den == 1) {
  616 |     return upcast<scale::num>(layout);
  617 |   }
  618 |   else {
  619 |     return downcast<scale::den>(upcast<scale::num>(layout));
  620 |   }
  621 |   CUTE_GCC_UNREACHABLE;
  622 | }
```
**EN:** Defines `OldType` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `OldType` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 624-632

```text
  624 | template <class A, class O, class B>
  625 | CUTE_HOST_DEVICE constexpr
  626 | auto
  627 | max_alignment(ComposedLayout<A,O,B> const& layout)
  628 | {
  629 |   // Do not attempt for general ComposedLayouts
  630 |   //return gcd(max_alignment(layout.layout_a()), max_alignment(layout.offset()), max_alignment(layout.layout_b()));
  631 |   return Int<1>{};
  632 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 634-641

```text
  634 | template <class A, class O, class B>
  635 | CUTE_HOST_DEVICE constexpr
  636 | auto
  637 | nullspace(ComposedLayout<A,O,B> const& layout)
  638 | {
  639 |   // Do not attempt for general ComposedLayouts
  640 |   return Layout<_1,_0>{};
  641 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 643-651

```text
  643 | //
  644 | // Display utilities
  645 | //
  647 | template <class A, class O, class B>
  648 | CUTE_HOST_DEVICE void print(ComposedLayout<A,O,B> const& layout)
  649 | {
  650 |   print(layout.layout_a()); print(" o "); print(layout.offset()); print(" o "); print(layout.layout_b());
  651 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization and formats internal structures for diagnostics or visualization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化并为诊断或可视化格式化内部结构。

### Lines 653-661

```text
  653 | #if !defined(__CUDACC_RTC__)
  654 | template <class A, class O, class B>
  655 | CUTE_HOST std::ostream& operator<<(std::ostream& os, ComposedLayout<A,O,B> const& layout)
  656 | {
  657 |   return os << layout.layout_a() << " o " << layout.offset() << " o " << layout.layout_b();
  658 | }
  659 | #endif
  661 | } // end namespace cute
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

## Key Concepts / 关键概念

- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- Intel Xe-specific behavior / Intel Xe 特定行为
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/layout.hpp`
  - `cute/numeric/integral_constant.hpp`
- Primary symbols / 主要符号: `ComposedLayout`, `is_layout`, `is_composed_layout`, `LayoutA`, `Offset`, `LayoutB`, `Coord`, `Coord0`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
