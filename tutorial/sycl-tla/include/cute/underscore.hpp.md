# underscore.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/underscore.hpp`
- Purpose (EN): Provides the underscore placeholder and related slicing markers used throughout CuTe layout and tensor APIs.
- 作用 (CN): 提供下划线占位符及相关切片标记，供 CuTe 的布局与张量 API 广泛使用。

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
   33 | #include <cute/config.hpp>                     // CUTE_INLINE_CONSTANT, CUTE_HOST_DEVICE
   34 | #include <cute/container/tuple.hpp>            // cute::is_tuple
   35 | #include <cute/numeric/integral_constant.hpp>  // cute::false_type, cute::true_type
```
**EN:** Sets up the header dependencies for this file by importing `cute/config.hpp`, `cute/container/tuple.hpp`, `cute/numeric/integral_constant.hpp`.
**CN:** 通过引入 `cute/config.hpp`, `cute/container/tuple.hpp`, `cute/numeric/integral_constant.hpp` 为该文件建立头文件依赖。

### Lines 37-38

```text
   37 | namespace cute
   38 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 40-41

```text
   40 | // For slicing
   41 | struct Underscore : Int<0> {};
```
**EN:** Defines `Underscore` as a reusable type-level building block in this header.
**CN:** 将 `Underscore` 定义为本头文件中的可复用类型级构件。

### Lines 43-43

```text
   43 | CUTE_INLINE_CONSTANT Underscore _;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 45-46

```text
   45 | // Convenient alias
   46 | using X = Underscore;
```
**EN:** Introduces `X` as a shorthand alias so later template-heavy code can refer to this type or mapping more concisely.
**CN:** 引入 `X` 这一简写别名，使后续模板密集代码能够更简洁地引用该类型或映射。

### Lines 48-50

```text
   48 | // Treat Underscore as an integral like integral_constant
   49 | template <>
   50 | struct is_integral<Underscore> : true_type {};
```
**EN:** Defines `is_integral` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `is_integral` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 52-55

```text
   52 | template <class T>
   53 | struct is_underscore : false_type {};
   54 | template <>
   55 | struct is_underscore<Underscore> : true_type {};
```
**EN:** Defines `is_underscore` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization.
**CN:** 将 `is_underscore` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化。

### Lines 57-67

```text
   57 | // Tuple trait for detecting static member element
   58 | template <class Tuple, class Elem, class Enable = void>
   59 | struct has_elem : false_type {};
   60 | template <class Elem>
   61 | struct has_elem<Elem, Elem> : true_type {};
   62 | template <class Tuple, class Elem>
   63 | struct has_elem<Tuple, Elem, enable_if_t<is_tuple<Tuple>::value> >
   64 |     : has_elem<Tuple, Elem, tuple_seq<Tuple> > {};
   65 | template <class Tuple, class Elem, int... Is>
   66 | struct has_elem<Tuple, Elem, seq<Is...>>
   67 |     : disjunction<has_elem<tuple_element_t<Is, Tuple>, Elem>...> {};
```
**EN:** Defines `has_elem` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization and provides tuple-like or typelist-oriented metaprogramming.
**CN:** 将 `has_elem` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化并提供类 tuple 或 typelist 风格的元编程能力。

### Lines 69-79

```text
   69 | // Tuple trait for detecting static member element
   70 | template <class Tuple, class Elem, class Enable = void>
   71 | struct all_elem : false_type {};
   72 | template <class Elem>
   73 | struct all_elem<Elem, Elem> : true_type {};
   74 | template <class Tuple, class Elem>
   75 | struct all_elem<Tuple, Elem, enable_if_t<is_tuple<Tuple>::value> >
   76 |     : all_elem<Tuple, Elem, tuple_seq<Tuple> > {};
   77 | template <class Tuple, class Elem, int... Is>
   78 | struct all_elem<Tuple, Elem, seq<Is...>>
   79 |     : conjunction<all_elem<tuple_element_t<Is, Tuple>, Elem>...> {};
```
**EN:** Defines `all_elem` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization and provides tuple-like or typelist-oriented metaprogramming.
**CN:** 将 `all_elem` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化并提供类 tuple 或 typelist 风格的元编程能力。

### Lines 81-92

```text
   81 | // Tuple trait for detecting Underscore member
   82 | template <class Tuple>
   83 | using has_underscore = has_elem<Tuple, Underscore>;
   85 | template <class Tuple>
   86 | using all_underscore = all_elem<Tuple, Underscore>;
   88 | template <class Tuple>
   89 | using has_int1 = has_elem<Tuple, Int<1>>;
   91 | template <class Tuple>
   92 | using has_int0 = has_elem<Tuple, Int<0>>;
```
**EN:** Introduces `Tuple` as a shorthand alias so later template-heavy code can refer to this type or mapping more concisely.
**CN:** 引入 `Tuple` 这一简写别名，使后续模板密集代码能够更简洁地引用该类型或映射。

### Lines 94-98

```text
   94 | //
   95 | // Slice keeps only the elements of Tuple B that are paired with an Underscore
   96 | //
   98 | namespace detail {
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 100-117

```text
  100 | template <class A, class B>
  101 | CUTE_HOST_DEVICE constexpr
  102 | auto
  103 | lift_slice(A const& a, B const& b)
  104 | {
  105 |   if constexpr (is_tuple<A>::value) {
  106 |     static_assert(tuple_size<A>::value == tuple_size<B>::value, "Mismatched Ranks");
  107 |     return filter_tuple(a, b, [](auto const& x, auto const& y) { return lift_slice(x,y); });
  108 |   } else if constexpr (is_underscore<A>::value) {
  109 |     return cute::tuple<B>{b};
  110 |   } else {
  111 |     return cute::tuple<>{};
  112 |   }
  113 | 
  114 |   CUTE_GCC_UNREACHABLE;
  115 | }
  117 | } // end namespace detail
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 119-135

```text
  119 | // Entry point overrides the lifting so that slice(_,b) == b
  120 | template <class A, class B>
  121 | CUTE_HOST_DEVICE constexpr
  122 | auto
  123 | slice(A const& a, B const& b)
  124 | {
  125 |   if constexpr (is_tuple<A>::value) {
  126 |     static_assert(tuple_size<A>::value == tuple_size<B>::value, "Mismatched Ranks");
  127 |     return filter_tuple(a, b, [](auto const& x, auto const& y) { return detail::lift_slice(x,y); });
  128 |   } else if constexpr (is_underscore<A>::value) {
  129 |     return b;
  130 |   } else {
  131 |     return cute::tuple<>{};
  132 |   }
  133 | 
  134 |   CUTE_GCC_UNREACHABLE;
  135 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also implements coordinate mapping or slicing behavior and acts as a factory/helper for constructing derived objects.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还实现坐标映射或切片行为并充当构造派生对象的工厂/辅助函数。

### Lines 137-141

```text
  137 | //
  138 | // Dice keeps only the elements of Tuple B that are paired with an Int
  139 | //
  141 | namespace detail {
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 143-160

```text
  143 | template <class A, class B>
  144 | CUTE_HOST_DEVICE constexpr
  145 | auto
  146 | lift_dice(A const& a, B const& b)
  147 | {
  148 |   if constexpr (is_tuple<A>::value) {
  149 |     static_assert(tuple_size<A>::value == tuple_size<B>::value, "Mismatched Ranks");
  150 |     return filter_tuple(a, b, [](auto const& x, auto const& y) { return lift_dice(x,y); });
  151 |   } else if constexpr (is_underscore<A>::value) {
  152 |     return cute::tuple<>{};
  153 |   } else {
  154 |     return cute::tuple<B>{b};
  155 |   }
  156 | 
  157 |   CUTE_GCC_UNREACHABLE;
  158 | }
  160 | } // end namespace detail
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 162-178

```text
  162 | // Entry point overrides the lifting so that dice(1,b) == b
  163 | template <class A, class B>
  164 | CUTE_HOST_DEVICE constexpr
  165 | auto
  166 | dice(A const& a, B const& b)
  167 | {
  168 |   if constexpr (is_tuple<A>::value) {
  169 |     static_assert(tuple_size<A>::value == tuple_size<B>::value, "Mismatched Ranks");
  170 |     return filter_tuple(a, b, [](auto const& x, auto const& y) { return detail::lift_dice(x,y); });
  171 |   } else if constexpr (is_underscore<A>::value) {
  172 |     return cute::tuple<>{};
  173 |   } else {
  174 |     return b;
  175 |   }
  176 | 
  177 |   CUTE_GCC_UNREACHABLE;
  178 | }
```
**EN:** Defines `A` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `A` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 180-186

```text
  180 | //
  181 | // Display utilities
  182 | //
  184 | CUTE_HOST_DEVICE void print(Underscore const&) {
  185 |   printf("_");
  186 | }
```
**EN:** Implements `print`, a helper routine used by the surrounding CuTe abstractions. It also formats internal structures for diagnostics or visualization.
**CN:** 实现 `print`，这是周围 CuTe 抽象所使用的辅助例程。 它还为诊断或可视化格式化内部结构。

### Lines 188-194

```text
  188 | #if !defined(__CUDACC_RTC__)
  189 | CUTE_HOST std::ostream& operator<<(std::ostream& os, Underscore const&) {
  190 |   return os << "_";
  191 | }
  192 | #endif
  194 | } // end namespace cute
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

## Key Concepts / 关键概念

- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Intel Xe-specific behavior / Intel Xe 特定行为
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/container/tuple.hpp`
  - `cute/numeric/integral_constant.hpp`
- Primary symbols / 主要符号: `Underscore`, `is_integral`, `is_underscore`, `has_elem`, `all_elem`, `T`, `Tuple`, `Elem`
- Dependency role / 依赖角色: Provides foundational template utilities that many higher-level CuTe headers build upon. / 提供基础模板工具，许多更高层的 CuTe 头文件都建立在其之上。
