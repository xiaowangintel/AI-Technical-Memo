# launch.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/compat/launch.hpp`
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

### Lines 27-36

```text
   27 | #include <sycl/accessor.hpp>
   28 | #include <sycl/event.hpp>
   29 | #include <sycl/nd_range.hpp>
   30 | #include <sycl/queue.hpp>
   31 | #include <sycl/range.hpp>
   32 | #include <sycl/reduction.hpp>
   34 | #include <cute/util/compat/device.hpp>
   35 | #include <cute/util/compat/dims.hpp>
   36 | #include <cute/util/compat/launch_policy.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `sycl/accessor.hpp`, `sycl/event.hpp`, `sycl/nd_range.hpp`, `sycl/queue.hpp`, `sycl/range.hpp`, and 4 more include(s).
**CN:** 通过引入 `sycl/accessor.hpp`, `sycl/event.hpp`, `sycl/nd_range.hpp`, `sycl/queue.hpp`, `sycl/range.hpp`，以及另外 4 个头文件 为该文件建立头文件依赖。

### Lines 38-38

```text
   38 | namespace compat {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 40-40

```text
   40 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 42-45

```text
   42 | template <typename R, typename... Types>
   43 | constexpr size_t getArgumentCount(R (*f)(Types...)) {
   44 |   return sizeof...(Types);
   45 | }
```
**EN:** Implements `getArgumentCount`, a helper routine used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 实现 `getArgumentCount`，这是周围 CuTe 抽象所使用的辅助例程。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 47-58

```text
   47 | template <int Dim>
   48 | sycl::nd_range<3> transform_nd_range(const sycl::nd_range<Dim> &range) {
   49 |   sycl::range<Dim> global_range = range.get_global_range();
   50 |   sycl::range<Dim> local_range = range.get_local_range();
   51 |   if constexpr (Dim == 3) {
   52 |     return range;
   53 |   } else if constexpr (Dim == 2) {
   54 |     return sycl::nd_range<3>{{1, global_range[0], global_range[1]},
   55 |                              {1, local_range[0], local_range[1]}};
   56 |   }
   57 |   return sycl::nd_range<3>{{1, 1, global_range[0]}, {1, 1, local_range[0]}};
   58 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并大量依赖编译期特化。

### Lines 60-67

```text
   60 | template <auto F, class N, typename... Args>
   61 | std::enable_if_t<std::is_invocable_v<decltype(F), Args...>, sycl::event>
   62 | launch(const sycl::nd_range<3> &range, sycl::queue q, Args... args) {
   63 |   static_assert(detail::getArgumentCount(F) == sizeof...(args),
   64 |                 "Wrong number of arguments to SYCL kernel");
   65 |   static_assert(
   66 |       std::is_same<std::invoke_result_t<decltype(F), Args...>, void>::value,
   67 |       "SYCL kernels should return void");
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 69-73

```text
   69 |   return q.parallel_for<N>(
   70 |       range, [=](sycl::nd_item<3>) { [[clang::always_inline]] F(args...); });
   71 | }
   73 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and bridges to SYCL execution or group abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并桥接到 SYCL 执行模型或 group 抽象。

### Lines 75-77

```text
   75 | template <int Dim>
   76 | inline sycl::nd_range<Dim> compute_nd_range(sycl::range<Dim> global_size_in,
   77 |                                             sycl::range<Dim> work_group_size) {
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 79-85

```text
   79 |   if (global_size_in.size() == 0 || work_group_size.size() == 0) {
   80 |     throw std::invalid_argument("Global or local size is zero!");
   81 |   }
   82 |   for (size_t i = 0; i < Dim; ++i) {
   83 |     if (global_size_in[i] < work_group_size[i])
   84 |       throw std::invalid_argument("Work group size larger than global size");
   85 |   }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 87-91

```text
   87 |   auto global_size =
   88 |       ((global_size_in + work_group_size - 1) / work_group_size) *
   89 |       work_group_size;
   90 |   return {global_size, work_group_size};
   91 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 93-96

```text
   93 | inline sycl::nd_range<1> compute_nd_range(int global_size_in,
   94 |                                           int work_group_size) {
   95 |   return compute_nd_range<1>(global_size_in, work_group_size);
   96 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 98-102

```text
   98 | template <auto F, class N=sycl::detail::auto_name, int Dim, typename... Args>
   99 | std::enable_if_t<std::is_invocable_v<decltype(F), Args...>, sycl::event>
  100 | launch(const sycl::nd_range<Dim> &range, sycl::queue q, Args... args) {
  101 |   return detail::launch<F, N>(detail::transform_nd_range<Dim>(range), q, args...);
  102 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 104-108

```text
  104 | template <auto F, class N=sycl::detail::auto_name, int Dim, typename... Args>
  105 | std::enable_if_t<std::is_invocable_v<decltype(F), Args...>, sycl::event>
  106 | launch(const sycl::nd_range<Dim> &range, Args... args) {
  107 |   return launch<F, N>(range, get_default_queue(), args...);
  108 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 110-115

```text
  110 | // Alternative launch through dim3 objects
  111 | template <auto F, class N=sycl::detail::auto_name, typename... Args>
  112 | std::enable_if_t<std::is_invocable_v<decltype(F), Args...>, sycl::event>
  113 | launch(const dim3 &grid, const dim3 &threads, sycl::queue q, Args... args) {
  114 |   return launch<F, N>(sycl::nd_range<3>{grid * threads, threads}, q, args...);
  115 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 117-123

```text
  117 | template <auto F, class N=sycl::detail::auto_name, typename... Args>
  118 | std::enable_if_t<std::is_invocable_v<decltype(F), Args...>, sycl::event>
  119 | launch(const dim3 &grid, const dim3 &threads, Args... args) {
  120 |   return launch<F, N>(grid, threads, get_default_queue(), args...);
  121 | }
  123 | } // namespace compat
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 125-125

```text
  125 | namespace compat::experimental {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 127-127

```text
  127 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 129-133

```text
  129 | template <auto F, class N = sycl::detail::auto_name, bool EventNeeded = true, typename LaunchPolicy, typename... Args>
  130 | auto launch(LaunchPolicy launch_policy, sycl::queue q, Args... args) {
  131 |   static_assert(compat::args_compatible<LaunchPolicy, F, Args...>,
  132 |                 "Mismatch between device function signature and supplied "
  133 |                 "arguments. Have you correctly handled local memory/char*?");
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 135-136

```text
  135 |   sycl_exp::launch_config config(launch_policy.get_range(),
  136 |                                  launch_policy.get_launch_properties());
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 138-169

```text
  138 |   if constexpr (EventNeeded) {
  139 |     return sycl_exp::submit_with_event(q, [&](sycl::handler &cgh) {
  140 |       auto KernelFunctor = build_kernel_functor<F>(cgh, launch_policy, args...);
  141 |       if constexpr (compat::detail::is_range_v<typename LaunchPolicy::RangeT>) {
  142 |         sycl_exp::parallel_for<N>(cgh, config, KernelFunctor);
  143 |       } else {
  144 |         static_assert(compat::detail::is_nd_range_v<typename LaunchPolicy::RangeT>);
  145 |         sycl_exp::nd_launch<N>(cgh, config, KernelFunctor);
  146 |       }
  147 |     });
  148 |   } else if constexpr (LaunchPolicy::HasLocalMem && launch_policy.get_local_mem_size() != 0) {
  149 |     sycl_exp::submit(q, [&](sycl::handler &cgh) {
  150 |       auto KernelFunctor = build_kernel_functor<F>(cgh, launch_policy, args...);
  151 |       if constexpr (compat::detail::is_range_v<typename LaunchPolicy::RangeT>) {
  152 |         sycl_exp::parallel_for<N>(cgh, config, KernelFunctor);
  153 |       } else {
  154 |         static_assert(compat::detail::is_nd_range_v<typename LaunchPolicy::RangeT>);
  155 |         sycl_exp::nd_launch<N>(cgh, config, KernelFunctor);
  156 |       }
  157 |     });
  158 |   } else {
  159 |     auto KernelFunctor = build_kernel_functor<F>(launch_policy, args...);
  160 |     if constexpr (compat::detail::is_range_v<typename LaunchPolicy::RangeT>) {
  161 |       sycl_exp::parallel_for<N>(q, config, KernelFunctor);
  162 |     } else {
  163 |       static_assert(compat::detail::is_nd_range_v<typename LaunchPolicy::RangeT>);
  164 |       sycl_exp::nd_launch<N>(q, config, KernelFunctor);
  165 |     }
  166 |   }
  167 | }
  169 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 172-176

```text
  172 | template <auto F, class N = sycl::detail::auto_name, bool EventNeeded = true, typename LaunchPolicy, typename... Args>
  173 | auto launch(LaunchPolicy launch_policy, sycl::queue q, Args... args) {
  174 |   static_assert(detail::is_launch_policy_v<LaunchPolicy>);
  175 |   return detail::launch<F, N, EventNeeded>(launch_policy, q, args...);
  176 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 178-184

```text
  178 | template <auto F, class N = sycl::detail::auto_name, bool EventNeeded = true, typename LaunchPolicy, typename... Args>
  179 | auto launch(LaunchPolicy launch_policy, Args... args) {
  180 |   static_assert(detail::is_launch_policy_v<LaunchPolicy>);
  181 |   return launch<F, N, EventNeeded>(launch_policy, get_default_queue(), args...);
  182 | }
  184 | } // namespace compat::experimental
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

## Key Concepts / 关键概念

- Compile-time numeric metaprogramming / 编译期数值元编程
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Header-only template specialization patterns / 纯头文件模板特化模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `sycl/accessor.hpp`
  - `sycl/event.hpp`
  - `sycl/nd_range.hpp`
  - `sycl/queue.hpp`
  - `sycl/range.hpp`
  - `sycl/reduction.hpp`
  - `cute/util/compat/device.hpp`
  - `cute/util/compat/dims.hpp`
  - `cute/util/compat/launch_policy.hpp`
- Primary symbols / 主要符号: `N`, `getArgumentCount`, `launch`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
