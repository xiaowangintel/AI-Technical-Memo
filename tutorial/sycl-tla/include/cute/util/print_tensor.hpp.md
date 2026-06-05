# print_tensor.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/print_tensor.hpp`
- Purpose (EN): Implements text, LaTeX, SVG, and tensor-oriented printers used to visualize CuTe layouts and tensor structures.
- 作用 (CN): 实现文本、LaTeX、SVG 以及张量专用打印器，用于可视化 CuTe 布局和张量结构。

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
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 33-36

```text
   33 | #include <cute/config.hpp>           // CUTE_HOST_DEVICE
   35 | #include <cute/layout.hpp>
   36 | #include <cute/tensor_impl.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `cute/config.hpp`, `cute/layout.hpp`, `cute/tensor_impl.hpp`.
**CN:** 通过引入 `cute/config.hpp`, `cute/layout.hpp`, `cute/tensor_impl.hpp` 为该文件建立头文件依赖。

### Lines 38-39

```text
   38 | namespace cute
   39 | {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 41-77

```text
   41 | ////////////////////////////////
   42 | // Layout 2D to Console table //
   43 | ////////////////////////////////
   45 | template <class Layout>
   46 | CUTE_HOST_DEVICE
   47 | void
   48 | print_layout(Layout const& layout)  // (m,n) -> idx
   49 | {
   50 |   CUTE_STATIC_ASSERT_V(rank(layout) == Int<2>{});
   51 | 
   52 |   int idx_width = num_digits(cosize(layout)) + 2;
   53 |   const char* delim = "+-----------------------";
   54 | 
   55 |   print(layout); print("\n");
   56 | 
   57 |   // Column indices
   58 |   print("    ");
   59 |   for (int n = 0; n < size<1>(layout); ++n) { printf("  %*d ", idx_width-2, n); }
   60 |   printf("\n");
   61 | 
   62 |   // Print out A m-by-n
   63 |   for (int m = 0; m < size<0>(layout); ++m) {
   64 |     // Header
   65 |     print("    ");
   66 |     for (int n = 0; n < size<1>(layout); ++n) { printf("%.*s", idx_width+1, delim); }
   67 |     printf("+\n");
   68 |     // Values
   69 |     printf("%2d  ", m);  // Row indices
   70 |     for (int n = 0; n < size<1>(layout); ++n) { printf("| %*d ", idx_width-2, int(layout(m,n))); }
   71 |     printf("|\n");
   72 |   }
   73 |   // Footer
   74 |   print("    ");
   75 |   for (int n = 0; n < size<1>(layout); ++n) { printf("%.*s", idx_width+1, delim); }
   76 |   printf("+\n");
   77 | }
```
**EN:** Defines the main `Layout` interface, exposing shape/stride accessors, coordinate mapping, composition, and related layout algebra operations.
**CN:** 定义核心 `Layout` 接口，提供 shape/stride 访问、坐标映射、组合以及相关布局代数操作。

### Lines 79-86

```text
   79 | // Capture and cast smem_ptr_flag Layouts to offset-0 layouts
   80 | template <class SwizzleFn, int B, class Layout>
   81 | CUTE_HOST_DEVICE
   82 | void
   83 | print_layout(ComposedLayout<SwizzleFn,smem_ptr_flag_bits<B>,Layout> const& layout)
   84 | {
   85 |   print_layout(as_position_independent_swizzle_layout(layout));
   86 | }
```
**EN:** Defines shared-memory pointer wrappers and constructors, including overloads that compose with swizzle-aware addressing.
**CN:** 定义共享内存指针包装器与构造函数，并包含可与 swizzle 感知寻址组合的重载。

### Lines 88-133

```text
   88 | ////////////////////////////////
   89 | // Tensor 1D,2D,3D,4D Console //
   90 | ////////////////////////////////
   92 | template <class Engine, class Layout>
   93 | CUTE_HOST_DEVICE
   94 | void
   95 | print_tensor(Tensor<Engine,Layout> const& tensor, bool print_type = true)
   96 | {
   97 |   if (print_type) {
   98 |     print(tensor); print(":\n");
   99 |   }
  100 | 
  101 |   if constexpr (Layout::rank == 1)
  102 |   {
  103 |     for (int m = 0; m < size(tensor); ++m) {
  104 |       pretty_print(tensor(m));
  105 |       printf("\n");
  106 |     }
  107 |   } else
  108 |   if constexpr (Layout::rank == 2)
  109 |   {
  110 |     for (int m = 0; m < size<0>(tensor); ++m) {
  111 |       for (int n = 0; n < size<1>(tensor); ++n) {
  112 |         pretty_print(tensor(m,n));
  113 |       }
  114 |       printf("\n");
  115 |     }
  116 |   } else
  117 |   if constexpr (Layout::rank == 3)
  118 |   {
  119 |     print_tensor(tensor(_,_,0), false);
  120 |     for (int k = 1; k < size<2>(tensor); ++k) {
  121 |       for (int i = 0; i < 5*size<1>(tensor); ++i) { print("-"); } print("\n");
  122 |       print_tensor(tensor(_,_,k), false);
  123 |     }
  124 |   } else
  125 |   if constexpr (Layout::rank == 4)
  126 |   {
  127 |     print_tensor(tensor(_,_,_,0), false);
  128 |     for (int p = 1; p < size<3>(tensor); ++p) {
  129 |       for (int i = 0; i < 5*size<1>(tensor); ++i) { print("="); } print("\n");
  130 |       print_tensor(tensor(_,_,_,p), false);
  131 |     }
  132 |   }
  133 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 135-176

```text
  135 | #if !defined(__CUDACC_RTC__)
  136 | template <class Engine, class Layout>
  137 | CUTE_HOST
  138 | std::ostream&
  139 | print_tensor_os(std::ostream& os, Tensor<Engine,Layout> const& tensor)
  140 | {
  141 |   int digits = 9;
  142 | 
  143 |   if constexpr (Layout::rank == 1)
  144 |   {
  145 |     for (int m = 0; m < size(tensor); ++m) {
  146 |       os << std::setw(digits) << tensor(m) << std::endl;
  147 |     }
  148 |   } else
  149 |   if constexpr (Layout::rank == 2)
  150 |   {
  151 |     for (int m = 0; m < size<0>(tensor); ++m) {
  152 |       for (int n = 0; n < size<1>(tensor); ++n) {
  153 |         os << std::setw(digits) << tensor(m,n);
  154 |       }
  155 |       os << std::endl;
  156 |     }
  157 |   } else
  158 |   if constexpr (Layout::rank == 3)
  159 |   {
  160 |     print_tensor_os(os, tensor(_,_,0));
  161 |     for (int k = 1; k < size<2>(tensor); ++k) {
  162 |       for (int i = 0; i < digits*size<1>(tensor); ++i) { os << "-"; } os << std::endl;
  163 |       print_tensor_os(os, tensor(_,_,k));
  164 |     }
  165 |   } else
  166 |   if constexpr (Layout::rank == 4)
  167 |   {
  168 |     print_tensor_os(os, tensor(_,_,_,0));
  169 |     for (int p = 1; p < size<3>(tensor); ++p) {
  170 |       for (int i = 0; i < digits*size<1>(tensor); ++i) { os << "="; } os << std::endl;
  171 |       print_tensor_os(os, tensor(_,_,_,p));
  172 |     }
  173 |   }
  174 | 
  175 |   return os;
  176 | }
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

### Lines 178-188

```text
  178 | template <class Engine, class Layout>
  179 | CUTE_HOST
  180 | std::ostream&
  181 | operator<<(std::ostream& os, Tensor<Engine,Layout> const& tensor)
  182 | {
  183 |   os << tensor.layout() << std::endl;
  184 |   return print_tensor_os(os, tensor);
  185 | }
  186 | #endif // !defined(__CUDACC_RTC__)
  188 | } // end namespace cute
```
**EN:** Formats CuTe objects for human-readable inspection, often emitting structured text or visualization-oriented output.
**CN:** 把 CuTe 对象格式化为便于人工检查的输出，通常生成结构化文本或面向可视化的结果。

## Key Concepts / 关键概念

- Layout algebra and coordinate linearization / 布局代数与坐标线性化
- Tensor views, tiling, and partitioning / 张量视图、平铺与分块
- Memory-space-aware pointer adaptation / 感知内存空间的指针适配
- Compile-time numeric metaprogramming / 编译期数值元编程
- Intel Xe-specific behavior / Intel Xe 特定行为
- Data swizzling and remapped access patterns / 数据重排与重映射访问模式

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `cute/config.hpp`
  - `cute/layout.hpp`
  - `cute/tensor_impl.hpp`
- Primary symbols / 主要符号: `Layout`, `SwizzleFn`, `Engine`, `print_layout`, `print_tensor`
- Dependency role / 依赖角色: Builds on tuple/shape/stride primitives and is used by tensor, tiling, copy, and compute abstractions that need coordinate mappings. / 构建于 tuple/shape/stride 原语之上，供需要坐标映射的张量、平铺、拷贝和计算抽象使用。
