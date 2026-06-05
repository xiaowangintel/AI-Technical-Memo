# launch_policy.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/compat/launch_policy.hpp`
- Purpose (EN): Implements the compatibility layer that maps CUDA-like concepts onto SYCL/back-end abstractions for atomics, kernels, launch, memory, math, and device queries.
- 作用 (CN): 实现兼容层，把类 CUDA 概念映射到 SYCL/后端抽象，覆盖原子操作、内核、启动、内存、数学和设备查询。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21

```text
    1 | /***************************************************************************
    2 |  *
    3 |  *  Copyright (C) Codeplay Software Ltd.
    4 |  *  Copyright (C) 2025 Intel Corporation, All rights reserved.
    5 |  *
    6 |  *  Part of the LLVM Project, under the Apache License v2.0 with LLVM
    7 |  *  Exceptions. See https://llvm.org/LICENSE.txt for license information.
    8 |  *  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
    9 |  *
   10 |  *  Unless required by applicable law or agreed to in writing, software
   11 |  *  distributed under the License is distributed on an "AS IS" BASIS,
   12 |  *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   13 |  *  See the License for the specific language governing permissions and
   14 |  *  limitations under the License.
   15 |  *
   16 |  *  SYCL compatibility extension
   17 |  *
   18 |  *  launch.hpp
   19 |  *
   20 |  *  Description:
   21 |  *    launch functionality for the SYCL compatibility extension
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 22-25

```text
   22 |  **************************************************************************/
   24 | #pragma once
   25 | #pragma GCC system_header
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 27-37

```text
   27 | #include "sycl/ext/oneapi/experimental/enqueue_functions.hpp"
   28 | #include "sycl/ext/oneapi/properties/properties.hpp"
   29 | #include <sycl/event.hpp>
   30 | #include <sycl/nd_range.hpp>
   31 | #include <sycl/queue.hpp>
   32 | #include <sycl/range.hpp>
   34 | #include <cute/util/compat/defs.hpp>
   35 | #include <cute/util/compat/device.hpp>
   36 | #include <cute/util/compat/dims.hpp>
   37 | #include <cute/util/compat/traits.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `sycl/ext/oneapi/experimental/enqueue_functions.hpp`, `sycl/ext/oneapi/properties/properties.hpp`, `sycl/event.hpp`, `sycl/nd_range.hpp`, `sycl/queue.hpp`, and 5 more include(s).
**CN:** 通过引入 `sycl/ext/oneapi/experimental/enqueue_functions.hpp`, `sycl/ext/oneapi/properties/properties.hpp`, `sycl/event.hpp`, `sycl/nd_range.hpp`, `sycl/queue.hpp`，以及另外 5 个头文件 为该文件建立头文件依赖。

### Lines 39-40

```text
   39 | namespace compat {
   40 | namespace experimental {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 42-42

```text
   42 | namespace sycl_exp = sycl::ext::oneapi::experimental;
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 44-47

```text
   44 | // Wrapper for kernel sycl_exp::properties
   45 | template <typename Properties> struct kernel_properties {
   46 |   static_assert(sycl_exp::is_property_list_v<Properties>);
   47 |   using Props = Properties;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 49-50

```text
   49 |   template <typename... Props>
   50 |   kernel_properties(Props... properties) : props{properties...} {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 52-54

```text
   52 |   template <typename... Props>
   53 |   kernel_properties(sycl_exp::properties<Props...> properties)
   54 |       : props{properties} {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 56-57

```text
   56 |   Properties props;
   57 | };
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 59-61

```text
   59 | template <typename... Props, typename = std::enable_if_t<detail::are_all_props<Props...>::value, void>>
   60 | kernel_properties(Props... props)
   61 |     -> kernel_properties<decltype(sycl_exp::properties(props...))>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 63-65

```text
   63 | template <typename... Props>
   64 | kernel_properties(sycl_exp::properties<Props...> props)
   65 |     -> kernel_properties<sycl_exp::properties<Props...>>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 67-70

```text
   67 | // Wrapper for launch sycl_exp::properties
   68 | template <typename Properties> struct launch_properties {
   69 |   static_assert(sycl_exp::is_property_list_v<Properties>);
   70 |   using Props = Properties;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 72-73

```text
   72 |   template <typename... Props>
   73 |   launch_properties(Props... properties) : props{properties...} {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 75-77

```text
   75 |   template <typename... Props>
   76 |   launch_properties(sycl_exp::properties<Props...> properties)
   77 |       : props{properties} {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 79-80

```text
   79 |   Properties props;
   80 | };
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 82-84

```text
   82 | template <typename... Props, typename = std::enable_if_t<detail::are_all_props<Props...>::value, void>>
   83 | launch_properties(Props... props)
   84 |     -> launch_properties<decltype(sycl_exp::properties(props...))>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 86-88

```text
   86 | template <typename... Props>
   87 | launch_properties(sycl_exp::properties<Props...> props)
   88 |     -> launch_properties<sycl_exp::properties<Props...>>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 90-94

```text
   90 | // Wrapper for local memory size
   91 | struct local_mem_size {
   92 |   local_mem_size(size_t size = 0) : size{size} {};
   93 |   size_t size;
   94 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 96-104

```text
   96 | // launch_policy is constructed by the user & passed to `compat_exp::launch`
   97 | template <typename Range, typename KProps, typename LProps, bool LocalMem>
   98 | class launch_policy {
   99 |   static_assert(sycl_exp::is_property_list_v<KProps>);
  100 |   static_assert(sycl_exp::is_property_list_v<LProps>);
  101 |   static_assert(compat::detail::is_range_or_nd_range_v<Range>);
  102 |   static_assert(compat::detail::is_nd_range_v<Range> || !LocalMem,
  103 |                 "sycl::range kernel launches are incompatible with local "
  104 |                 "memory usage!");
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 106-110

```text
  106 | public:
  107 |   using KPropsT = KProps;
  108 |   using LPropsT = LProps;
  109 |   using RangeT = Range;
  110 |   static constexpr bool HasLocalMem = LocalMem;
```
**EN:** Introduces `KPropsT` as a shorthand alias so later template-heavy code can refer to this type or mapping more concisely.
**CN:** 引入 `KPropsT` 这一简写别名，使后续模板密集代码能够更简洁地引用该类型或映射。

### Lines 112-113

```text
  112 | private:
  113 |   launch_policy() = default;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 115-127

```text
  115 |   template <typename... Ts>
  116 |   launch_policy(Ts... ts)
  117 |       : _kernel_properties{detail::property_getter<
  118 |             kernel_properties, kernel_properties<KPropsT>, std::tuple<Ts...>>()(
  119 |             std::tuple<Ts...>(ts...))},
  120 |         _launch_properties{detail::property_getter<
  121 |             launch_properties, launch_properties<LPropsT>, std::tuple<Ts...>>()(
  122 |             std::tuple<Ts...>(ts...))},
  123 |         _local_mem_size{
  124 |             detail::local_mem_getter<local_mem_size, std::tuple<Ts...>>()(
  125 |                 std::tuple<Ts...>(ts...))} {
  126 |     check_variadic_args(ts...);
  127 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 129-137

```text
  129 |   template <typename... Ts> void check_variadic_args(Ts...) {
  130 |     static_assert(
  131 |         std::conjunction_v<std::disjunction<detail::is_kernel_properties<Ts>,
  132 |                                             detail::is_launch_properties<Ts>,
  133 |                                             detail::is_local_mem_size<Ts>>...>,
  134 |         "Received an unexpected argument to ctor. Did you forget to wrap "
  135 |         "in "
  136 |         "compat::kernel_properties, launch_properties, local_mem_size?");
  137 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 139-144

```text
  139 | public:
  140 |   template <typename... Ts>
  141 |   launch_policy(Range range, Ts... ts) : launch_policy(ts...) {
  142 |     _range = range;
  143 |     check_variadic_args(ts...);
  144 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 146-150

```text
  146 |   template <typename... Ts>
  147 |   launch_policy(dim3 global_range, Ts... ts) : launch_policy(ts...) {
  148 |     _range = Range{global_range};
  149 |     check_variadic_args(ts...);
  150 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 152-157

```text
  152 |   template <typename... Ts>
  153 |   launch_policy(dim3 global_range, dim3 local_range, Ts... ts)
  154 |       : launch_policy(ts...) {
  155 |     _range = Range{global_range * local_range, local_range};
  156 |     check_variadic_args(ts...);
  157 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 159-162

```text
  159 |   KProps get_kernel_properties() { return _kernel_properties.props; }
  160 |   LProps get_launch_properties() { return _launch_properties.props; }
  161 |   size_t get_local_mem_size() { return _local_mem_size.size; }
  162 |   Range get_range() { return _range; }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 164-169

```text
  164 | private:
  165 |   Range _range;
  166 |   kernel_properties<KProps> _kernel_properties;
  167 |   launch_properties<LProps> _launch_properties;
  168 |   local_mem_size _local_mem_size;
  169 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 171-176

```text
  171 | // Deduction guides for launch_policy
  172 | template <typename Range, typename... Ts>
  173 | launch_policy(Range, Ts...) -> launch_policy<
  174 |     Range, detail::properties_or_empty<kernel_properties, Ts...>,
  175 |     detail::properties_or_empty<launch_properties, Ts...>,
  176 |     detail::has_type<local_mem_size, std::tuple<Ts...>>::value>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 178-182

```text
  178 | template <int Dim, typename... Ts>
  179 | launch_policy(sycl::range<Dim>, sycl::range<Dim>, Ts...) -> launch_policy<
  180 |     sycl::nd_range<Dim>, detail::properties_or_empty<kernel_properties, Ts...>,
  181 |     detail::properties_or_empty<launch_properties, Ts...>,
  182 |     detail::has_type<local_mem_size, std::tuple<Ts...>>::value>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 184-188

```text
  184 | template <typename... Ts>
  185 | launch_policy(dim3, Ts...) -> launch_policy<
  186 |     sycl::range<3>, detail::properties_or_empty<kernel_properties, Ts...>,
  187 |     detail::properties_or_empty<launch_properties, Ts...>,
  188 |     detail::has_type<local_mem_size, std::tuple<Ts...>>::value>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 190-194

```text
  190 | template <typename... Ts>
  191 | launch_policy(dim3, dim3, Ts...) -> launch_policy<
  192 |     sycl::nd_range<3>, detail::properties_or_empty<kernel_properties, Ts...>,
  193 |     detail::properties_or_empty<launch_properties, Ts...>,
  194 |     detail::has_type<local_mem_size, std::tuple<Ts...>>::value>;
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 196-203

```text
  196 | namespace detail {
  197 | // Custom std::apply helpers to enable inlining
  198 | template <class F, class Tuple, size_t... Is>
  199 | __compat_inline__ constexpr void apply_expand(F &&f, Tuple &&t,
  200 |                                                   std::index_sequence<Is...>) {
  201 |   [[clang::always_inline]] std::forward<F>(f)(
  202 |       std::get<Is>(std::forward<Tuple>(t))...);
  203 | }
```
**EN:** Defines `F` as a reusable type-level building block in this header. It also leans heavily on compile-time specialization and provides tuple-like or typelist-oriented metaprogramming.
**CN:** 将 `F` 定义为本头文件中的可复用类型级构件。 它还大量依赖编译期特化并提供类 tuple 或 typelist 风格的元编程能力。

### Lines 205-210

```text
  205 | template <class F, class Tuple>
  206 | __compat_inline__ constexpr void apply_helper(F &&f, Tuple &&t) {
  207 |   apply_expand(
  208 |       std::forward<F>(f), std::forward<Tuple>(t),
  209 |       std::make_index_sequence<std::tuple_size_v<std::decay_t<Tuple>>>{});
  210 | }
```
**EN:** Defines `F` as a reusable type-level building block in this header. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 将 `F` 定义为本头文件中的可复用类型级构件。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 212-217

```text
  212 | template <auto F, typename Range, typename KProps, bool HasLocalMem,
  213 |           typename... Args>
  214 | struct KernelFunctor {
  215 |   KernelFunctor(KProps kernel_props, Args... args)
  216 |       : _kernel_properties{kernel_props},
  217 |         _argument_tuple(std::make_tuple(args...)) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 219-222

```text
  219 |   KernelFunctor(KProps kernel_props, sycl::local_accessor<char, 1> local_acc,
  220 |                 Args... args)
  221 |       : _kernel_properties{kernel_props}, _local_acc{local_acc},
  222 |         _argument_tuple(std::make_tuple(args...)) {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 224-224

```text
  224 |   auto get(sycl_exp::properties_tag) const { return _kernel_properties; }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 226-241

```text
  226 |   __compat_inline__ void
  227 |   operator()(compat::detail::range_to_item_t<Range>) const {
  228 |     if constexpr (HasLocalMem) {
  229 |       char *local_mem_ptr = static_cast<char *>(
  230 |           _local_acc.template get_multi_ptr<sycl::access::decorated::no>()
  231 |               .get());
  232 |       apply_helper(
  233 |           [lmem_ptr = local_mem_ptr](auto &&...args) {
  234 |             [[clang::always_inline]] F(args..., lmem_ptr);
  235 |           },
  236 |           _argument_tuple);
  237 |     } else {
  238 |       apply_helper([](auto &&...args) { [[clang::always_inline]] F(args...); },
  239 |                    _argument_tuple);
  240 |     }
  241 |   }
```
**EN:** Implements `operator`, a helper routine used by the surrounding CuTe abstractions. It also implements coordinate mapping or slicing behavior and leans heavily on compile-time specialization.
**CN:** 实现 `operator`，这是周围 CuTe 抽象所使用的辅助例程。 它还实现坐标映射或切片行为并大量依赖编译期特化。

### Lines 243-247

```text
  243 |   KProps _kernel_properties;
  244 |   std::tuple<Args...> _argument_tuple;
  245 |   std::conditional_t<HasLocalMem, sycl::local_accessor<char, 1>, std::monostate>
  246 |       _local_acc; // monostate for empty type
  247 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 249-257

```text
  249 | //====================================================================
  250 | // This helper function avoids 2 nested `if constexpr` in detail::launch
  251 | template <auto F, typename LaunchPolicy, typename... Args>
  252 | auto build_kernel_functor(LaunchPolicy launch_policy, Args... args) {
  253 |   return KernelFunctor<F, typename LaunchPolicy::RangeT,
  254 |                        typename LaunchPolicy::KPropsT,
  255 |                        LaunchPolicy::HasLocalMem, Args...>(
  256 |       launch_policy.get_kernel_properties(), args...);
  257 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 259-279

```text
  259 | template <auto F, typename LaunchPolicy, typename... Args>
  260 | auto build_kernel_functor(sycl::handler &cgh, LaunchPolicy launch_policy,
  261 |                           Args... args)
  262 |     -> KernelFunctor<F, typename LaunchPolicy::RangeT,
  263 |                      typename LaunchPolicy::KPropsT, LaunchPolicy::HasLocalMem,
  264 |                      Args...> {
  265 |   if constexpr (LaunchPolicy::HasLocalMem) {
  266 |     sycl::local_accessor<char, 1> local_memory(
  267 |         launch_policy.get_local_mem_size(), cgh);
  268 |     return KernelFunctor<F, typename LaunchPolicy::RangeT,
  269 |                          typename LaunchPolicy::KPropsT,
  270 |                          LaunchPolicy::HasLocalMem, Args...>(
  271 |         launch_policy.get_kernel_properties(), local_memory, args...);
  272 |   } else {
  273 |     return build_kernel_functor<F, LaunchPolicy, Args...>(launch_policy, args...);
  274 |   }
  275 | }
  277 | } // namespace detail
  278 | } // namespace experimental
  279 | } // namespace compat
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

## Key Concepts / 关键概念

- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `sycl/ext/oneapi/experimental/enqueue_functions.hpp`
  - `sycl/ext/oneapi/properties/properties.hpp`
  - `sycl/event.hpp`
  - `sycl/nd_range.hpp`
  - `sycl/queue.hpp`
  - `sycl/range.hpp`
  - `cute/util/compat/defs.hpp`
  - `cute/util/compat/device.hpp`
  - `cute/util/compat/dims.hpp`
  - `cute/util/compat/traits.hpp`
- Primary symbols / 主要符号: `kernel_properties`, `launch_properties`, `local_mem_size`, `KernelFunctor`, `launch_policy`, `F`, `Tuple`, `Props`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
