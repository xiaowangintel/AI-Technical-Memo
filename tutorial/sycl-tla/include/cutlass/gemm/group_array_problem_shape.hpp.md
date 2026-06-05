# group_array_problem_shape.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/group_array_problem_shape.hpp`
- **Purpose (EN):** Defines grouped problem-shape utilities for batched or grouped GEMM scheduling.
- **用途 (CN):** 定义用于批量或分组 GEMM 调度的问题形状工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
3:  * SPDX-License-Identifier: BSD-3-Clause
4:  *
5:  * Redistribution and use in source and binary forms, with or without
6:  * modification, are permitted provided that the following conditions are met:
7:  *
8:  * 1. Redistributions of source code must retain the above copyright notice, this
9:  * list of conditions and the following disclaimer.
10:  *
11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
12:  * this list of conditions and the following disclaimer in the documentation
```
**EN:** Introduces the license header and legal reuse conditions for this header.
**CN:** 说明该头文件的许可证声明与复用条件。

### Lines 13-24
```cpp
13:  * and/or other materials provided with the distribution.
14:  *
15:  * 3. Neither the name of the copyright holder nor the names of its
16:  * contributors may be used to endorse or promote products derived from
17:  * this software without specific prior written permission.
18:  *
19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
```
**EN:** Introduces the license header and legal reuse conditions for this header.
**CN:** 说明该头文件的许可证声明与复用条件。

### Lines 25-35
```cpp
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
31: /*! \file
32:     \brief This file contains definitions and utility functions for describing problem shapes 
33:            for 3.x Ptr-Array GEMMs and Grouped GEMMs.
34: */
35: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 36-46
```cpp
36: #pragma once
37: 
38: #include "cutlass/cutlass.h"
39: #include "cutlass/tensor_coord.h"
40: 
41: #include "cute/container/array.hpp"
42: 
43: #if ! defined(__CUDACC_RTC__)
44: #include <initializer_list>
45: #endif
46: 
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 47-52
```cpp
47: ////////////////////////////////////////////////////////////////////////////////////////////////////
48: 
49: namespace cutlass::gemm {
50: 
51: ////////////////////////////////////////////////////////////////////////////////////////////////////
52: 
```
**EN:** Enters namespace scope (cutlass) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass），组织 GEMM 抽象层。

### Lines 53-59
```cpp
53: template <class ProblemShape_>
54: struct GroupProblemShape {
55:   using UnderlyingProblemShape = ProblemShape_;
56:   int32_t num_groups = 1;
57:   UnderlyingProblemShape* problem_shapes = nullptr;
58:   UnderlyingProblemShape const* host_problem_shapes = nullptr;
59: 
```
**EN:** Declares template parameters and begins the definition of ProblemShape_.
**CN:** 声明模板参数并开始定义 ProblemShape_。

### Lines 60-62
```cpp
60:   CUTLASS_HOST_DEVICE
61:   int32_t groups() const { return num_groups; }
62: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 63-68
```cpp
63:   CUTLASS_HOST_DEVICE
64:   UnderlyingProblemShape const
65:   get_problem_shape(int32_t group_idx) const {
66:     return problem_shapes[group_idx];
67:   }
68: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 69-74
```cpp
69:   CUTLASS_HOST_DEVICE
70:   UnderlyingProblemShape const
71:   get_host_problem_shape(int32_t group_idx) const {
72:     return host_problem_shapes != nullptr ? host_problem_shapes[group_idx] : UnderlyingProblemShape{};
73:   }
74: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 75-81
```cpp
75:   CUTLASS_HOST_DEVICE
76:   bool
77:   is_host_problem_shape_available() const {
78:     return host_problem_shapes != nullptr;
79:   }
80: };
81: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 82-86
```cpp
82: template <class ProblemShape_, class MaxProblemShape_>
83: struct MoEProblemShape {
84:   using UnderlyingProblemShape = ProblemShape_;
85:   using MaxProblemShape = MaxProblemShape_;
86: 
```
**EN:** Declares template parameters and begins the definition of ProblemShape_.
**CN:** 声明模板参数并开始定义 ProblemShape_。

### Lines 87-91
```cpp
87:   UnderlyingProblemShape problem_shape;
88:   MaxProblemShape max_problem_shape;
89: };
90: 
91: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 92-96
```cpp
92: template <class ProblemShape_>
93: class ArrayProblemShape {
94: public:
95:   using UnderlyingProblemShape = ProblemShape_;
96: 
```
**EN:** Declares template parameters and begins the definition of ProblemShape_.
**CN:** 声明模板参数并开始定义 ProblemShape_。

### Lines 97-103
```cpp
97:   ArrayProblemShape() = default;
98:   ArrayProblemShape(UnderlyingProblemShape ps) : problem_shape_(ps) {}
99: 
100:   // Num of groups for Ptr-Array GEMM always remain one, just the number of batches (l) can vary
101:   // This is just to maintain uniformity with GroupProblemShape
102:   constexpr int32_t groups() const { return 1; }
103: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 104-106
```cpp
104:   UnderlyingProblemShape* problem_shapes() const {
105:     return &problem_shape_;
106:   }
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 107-111
```cpp
107:   UnderlyingProblemShape const* host_problem_shapes() const {
108:     return &problem_shape_;
109:   }
110: 
111:   // This is just to maintain uniformity with GroupProblemShape
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 112-117
```cpp
112:   CUTLASS_HOST_DEVICE
113:   UnderlyingProblemShape const
114:   get_problem_shape(int32_t /* unused */ = 0) const {
115:     return problem_shape_;
116:   }
117: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 118-123
```cpp
118:   CUTLASS_HOST_DEVICE
119:   UnderlyingProblemShape const
120:   get_host_problem_shape(int32_t /* unused */ = 0) const {
121:     return problem_shape_;
122:   }
123: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 124-133
```cpp
124:   CUTLASS_HOST_DEVICE
125:   bool
126:   is_host_problem_shape_available() const {
127:     return true;
128:   }
129: private:
130:   UnderlyingProblemShape problem_shape_{};
131: };
132: 
133: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 134-143
```cpp
134: namespace detail {
135:   
136: template<class T>
137: struct is_moe_problem_shape : cute::false_type {};
138: template<class T, class U>
139: struct is_moe_problem_shape<cutlass::gemm::MoEProblemShape<T,U>> : cute::true_type {}; 
140: 
141: }
142: 
143: } // namespace cutlass::gemm 
```
**EN:** Enters namespace scope (detail) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（detail），组织 GEMM 抽象层。

## Key Concepts / 关键概念

- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Grouped problem scheduling  
  **CN:** 分组问题调度

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `ProblemShape_`, `GroupProblemShape`, `MaxProblemShape_`, `MoEProblemShape`, `ArrayProblemShape`, `T`, `is_moe_problem_shape`, `U`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
