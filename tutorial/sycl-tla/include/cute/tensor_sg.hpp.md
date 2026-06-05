# tensor_sg.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/tensor_sg.hpp`
- Purpose (EN): Defines tensor views, tensor implementation details, subgroup-aware tensors, and tensor zipping/composition helpers.
- 作用 (CN): 定义张量视图、张量实现细节、子组感知张量以及张量拉链/组合辅助工具。

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
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 34-35

```text
   34 | #include <cute/tensor_impl.hpp>                // cute::Tensor
   35 | #include <cute/util/sycl_vec.hpp>              // intel::_SGSize
```
**EN:** Sets up the header dependencies for this file by importing `cute/tensor_impl.hpp`, `cute/util/sycl_vec.hpp`.
**CN:** 通过引入 `cute/tensor_impl.hpp`, `cute/util/sycl_vec.hpp` 为该文件建立头文件依赖。

### Lines 37-38

```text
   37 | namespace cute
   38 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 40-54

```text
   40 | //
   41 | // SubgroupTensor
   42 | //
   43 | // A SubgroupTensor represents a subgroup-scope tensor,
   44 | //   e.g. the result of a block 2D load, or an input/output matrix to DPAS.
   45 | //
   46 | // SubgroupTensor wraps a standard CuTe rmem tensor ("fragment") holding the current
   47 | //   work-item's slice of the tensor. It implicitly decays to this fragment, so it can
   48 | //   be used as a regular rmem Tensor.
   49 | //
   50 | // In addition, a SubgroupTensor holds a thread-value layout identifying logical coordinates
   51 | //   for each element of the tensor. The interpretation of the logical coordinates is user-defined,
   52 | //   Reorder operations use these logical coordinates to identify corresponding values in
   53 | //   the source and destination tensors.
   54 | //
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 56-91

```text
   56 | template <class Engine,
   57 |           class Layout,               // fragment coord -> V
   58 |           class SubgroupTVLayout>     // (T,V) -> coord in subgroup
   59 | struct SubgroupTensor : Tensor<Engine, Layout>
   60 | {
   61 |   using Base = Tensor<Engine, Layout>;
   62 | 
   63 |   using typename Base::iterator;
   64 |   using typename Base::value_type;
   65 |   using typename Base::element_type;
   66 |   using typename Base::reference;
   67 |   using typename Base::engine_type;
   68 |   using typename Base::layout_type;
   69 | 
   70 |   CUTE_HOST_DEVICE constexpr
   71 |   SubgroupTensor() {}
   72 | 
   73 |   CUTE_HOST_DEVICE constexpr explicit
   74 |   SubgroupTensor(Base const& base) {
   75 |     *this = static_cast<SubgroupTensor const&>(base);
   76 |   }
   77 | 
   78 |   static constexpr int rank = Layout::rank;
   79 | 
   80 |   CUTE_HOST_DEVICE constexpr
   81 |   decltype(auto)
   82 |   tensor() const {
   83 |     return *static_cast<const Base*>(this);
   84 |   }
   85 | 
   86 |   CUTE_HOST_DEVICE constexpr
   87 |   auto
   88 |   tv_layout() const {
   89 |     return SubgroupTVLayout{};
   90 |   }
   91 | };
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 93-96

```text
   93 | template <class T>
   94 | struct is_sg_tensor : false_type {};
   95 | template <class Engine, class Layout, class SubgroupTVLayout>
   96 | struct is_sg_tensor<SubgroupTensor<Engine,Layout,SubgroupTVLayout>> : true_type {};
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 98-99

```text
   98 | template <class Engine, class Layout, class SubgroupTVLayout>
   99 | struct is_tensor<SubgroupTensor<Engine,Layout,SubgroupTVLayout>> : true_type {};
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 101-114

```text
  101 | // Create a SubgroupTensor from its component parts:
  102 | //   a regular rmem Tensor and the subgroup-scope TV-layout.
  103 | template <class Engine,
  104 |           class Layout,
  105 |           class SubgroupTVLayout,
  106 |           __CUTE_REQUIRES(is_layout<SubgroupTVLayout>::value)>
  107 | CUTE_HOST_DEVICE
  108 | constexpr decltype(auto)
  109 | make_subgroup_tensor(Tensor<Engine,Layout>& tensor, SubgroupTVLayout const& tv_layout)
  110 | {
  111 |   static_assert(is_static_v<SubgroupTVLayout>, "Subgroup TV layout must be static");
  112 |   static_assert(is_rmem_v<Engine>, "Expected an rmem tensor");
  113 |   return make_subgroup_tensor(make_tensor(tensor.data(), tensor.layout()), tv_layout);
  114 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 116-127

```text
  116 | template <class Engine,
  117 |           class Layout,
  118 |           class SubgroupTVLayout,
  119 |           __CUTE_REQUIRES(is_layout<SubgroupTVLayout>::value)>
  120 | CUTE_HOST_DEVICE
  121 | constexpr decltype(auto)
  122 | make_subgroup_tensor(Tensor<Engine,Layout>&& tensor, SubgroupTVLayout const&)
  123 | {
  124 |   static_assert(is_static_v<SubgroupTVLayout>, "Subgroup TV layout must be static");
  125 |   static_assert(is_rmem_v<Engine>, "Expected an rmem tensor");
  126 |   return static_cast<SubgroupTensor<Engine,Layout,SubgroupTVLayout>&&>(tensor);
  127 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 129-142

```text
  129 | // Create a new owning SubgroupTensor with the given subgroup-level layout.
  130 | // Elements are assigned to threads following the normal Xe interleaved mapping
  131 | //   (i.e. work-item i gets elements i, i + 16, i + 32, ...)
  132 | template <typename T, class Shape, class Stride>
  133 | CUTE_HOST_DEVICE
  134 | constexpr auto
  135 | make_subgroup_tensor(Layout<Shape,Stride> const& sg_layout)
  136 | {
  137 |   using _SG = intel::_SGSize;
  138 |   auto ilayout = make_layout(make_shape(_SG{}, size(sg_layout) / _SG{}),
  139 |                              make_stride(_1{}, _16{}));
  140 |   auto sv_layout = sg_layout.compose(ilayout);
  141 |   return make_subgroup_tensor(make_fragment_like<T>(sv_layout(0,_)), sv_layout);
  142 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 144-152

```text
  144 | // Create a new owning SubgroupTensor with a subgroup-level layout, constructed
  145 | //   from the argument list with make_layout.
  146 | template <typename T, class... Args>
  147 | CUTE_HOST_DEVICE
  148 | constexpr auto
  149 | make_subgroup_tensor(Args const&... args)
  150 | {
  151 |   return make_subgroup_tensor<T>(make_layout(args...));
  152 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 155-171

```text
  155 | // Replicate a subgroup fragment in a given mode.
  156 | template <int Mode, int Expand, typename EngineIn, typename LayoutIn, typename TVLayoutIn>
  157 | CUTE_HOST_DEVICE
  158 | constexpr auto
  159 | expand_sg_fragment_helper(SubgroupTensor<EngineIn,LayoutIn,TVLayoutIn> const&)
  160 | {
  161 |   constexpr SubgroupTensor<EngineIn,LayoutIn,TVLayoutIn> frag;
  162 |   constexpr int ModeSize = get<Mode>(atuple_coshape(frag.tv_layout()));
  163 | 
  164 |   auto xlayout = append(frag.layout(),
  165 |                         Layout<C<Expand>, C<cosize_v<LayoutIn>>>{});
  166 |   auto xv_layout = append(get<1>(frag.tv_layout()),
  167 |                           make_layout(C<Expand>{}, C<ModeSize>{} * E<Mode>{}));
  168 |   auto xtv_layout = make_layout(get<0>(frag.tv_layout()), xv_layout);
  169 | 
  170 |   return make_subgroup_tensor(make_tensor<typename EngineIn::element_type>(xlayout), xtv_layout);
  171 | }
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 173-174

```text
  173 | template <typename SGTensor, int Mode, int Expand>
  174 | using expand_sg_fragment_t = decltype(expand_sg_fragment_helper<Mode, Expand>(SGTensor{}));
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

### Lines 176-188

```text
  176 | //
  177 | // Display utilities
  178 | //
  180 | template <class Engine, class Layout, class SubgroupTVLayout>
  181 | CUTE_HOST_DEVICE void print(SubgroupTensor<Engine,Layout,SubgroupTVLayout> const& tensor)
  182 | {
  183 |   print("SubgroupTensor\n");
  184 |   print("  Tensor:           "); print(static_cast<Tensor<Engine,Layout> const&>(tensor)); print("\n");
  185 |   print("  SubgroupTVLayout: "); print(SubgroupTVLayout{}); print("\n");
  186 | }
  188 | } // end namespace cute
```
**EN:** Defines tensor-facing helpers that build, transform, or zip tensor views on top of CuTe layouts and pointers.
**CN:** 定义面向张量的辅助代码，在 CuTe 布局与指针之上构建、变换或拉链组合张量视图。

## Key Concepts / 关键概念

- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/tensor_impl.hpp`
  - `cute/util/sycl_vec.hpp`
- Primary symbols / 主要符号: `SubgroupTensor`, `is_sg_tensor`, `is_tensor`, `Engine`, `Layout`, `SubgroupTVLayout`, `T`, `Shape`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
