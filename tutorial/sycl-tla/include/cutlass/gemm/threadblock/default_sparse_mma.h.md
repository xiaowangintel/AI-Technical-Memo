# default_sparse_mma.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/threadblock/default_sparse_mma.h`
- **Purpose (EN):** Defines default configuration helpers for selecting CUTLASS kernels.
- **用途 (CN):** 定义用于选择 CUTLASS 内核的默认配置辅助模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 25-34
```cpp
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
31: /*! \file
32:     \brief Template for a pipelined GEMM kernel. Does not compute batching or support split-K.
33: */
34: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 35-36
```cpp
35: #pragma once
36: 
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 37-41
```cpp
37: #include "cutlass/cutlass.h"
38: #include "cutlass/numeric_types.h"
39: #include "cutlass/arch/arch.h"
40: #include "cutlass/arch/wmma.h"
41: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, numeric types/converters, architecture intrinsics.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、数值类型/转换器、架构内建/指令封装。

### Lines 42-52
```cpp
42: #include "cutlass/layout/matrix.h"
43: #include "cutlass/transform/threadblock/predicated_tile_iterator.h"
44: #include "cutlass/transform/threadblock/predicated_tile_iterator_2dthreadtile.h"
45: #include "cutlass/gemm/threadblock/default_mma_core_sm70.h"
46: #include "cutlass/gemm/threadblock/default_mma_core_sm75.h"
47: #include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
48: #include "cutlass/gemm/threadblock/default_mma_core_sparse_sm80.h"
49: #if defined(CUTLASS_ARCH_WMMA_ENABLED)
50: #include "cutlass/gemm/threadblock/default_mma_core_wmma.h"
51: #endif //CUTLASS_ARCH_WMMA_ENABLED
52: 
```
**EN:** Pulls in required dependencies such as layout types, threadblock components.
**CN:** 引入所需依赖，例如 布局类型、线程块组件。

### Lines 53-62
```cpp
53: ////////////////////////////////////////////////////////////////////////////////
54: 
55: namespace cutlass {
56: namespace gemm {
57: namespace threadblock {
58: 
59: ////////////////////////////////////////////////////////////////////////////////
60: 
61: template <
62:     /// Element type for A matrix operand
```
**EN:** Enters namespace scope (cutlass::gemm::threadblock) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass::gemm::threadblock），组织 GEMM 抽象层。

### Lines 63-72
```cpp
63:     typename ElementA_,
64:     /// Layout type for A matrix operand
65:     typename LayoutA_,
66:     /// Access granularity of A matrix in units of elements
67:     int kAlignmentA,
68:     /// Element type for B matrix operand
69:     typename ElementB_,
70:     /// Layout type for B matrix operand
71:     typename LayoutB_,
72:     /// Access granularity of B matrix in units of elements
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 73-82
```cpp
73:     int kAlignmentB,
74:     /// Element type for internal accumulation
75:     typename ElementAccumulator_,
76:     /// Layout type for C and D matrix operands
77:     typename LayoutC_,
78:     /// Operator class tag
79:     typename OperatorClass_,
80:     /// Tag indicating architecture to tune for
81:     typename ArchTag_,
82:     /// Threadblock-level tile size (concept: GemmShape)
```
**EN:** Defines tag, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tag，用于封装策略、存储或算法行为的辅助类型。

### Lines 83-90
```cpp
83:     typename ThreadblockShape_,
84:     /// Warp-level tile size (concept: GemmShape)
85:     typename WarpShape_,
86:     /// Instruction-level tile size (concept: GemmShape)
87:     typename InstructionShape_,
88:     /// Number of stages used in the pipelined mainloop
89:     int Stages,
90:     /// Operation performed by GEMM
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 91-93
```cpp
91:     typename Operator,
92:     /// Store the accumulators in row major or column major.  Row major is used
93:     /// when output layout is interleaved.
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 94-104
```cpp
94:     bool AccumulatorsInRowMajor = false
95:     >
96: struct DefaultSparseMma;
97: 
98: ////////////////////////////////////////////////////////////////////////////////
99: 
100: /// Specialization for row-major output (OperatorClass TensorOp)
101: template <
102:     /// Element type for A matrix operand
103:     typename ElementA,
104:     /// Layout type for A matrix operand
```
**EN:** Declares template parameters and begins the definition of DefaultSparseMma.
**CN:** 声明模板参数并开始定义 DefaultSparseMma。

### Lines 105-114
```cpp
105:     typename LayoutA,
106:     /// Access granularity of A matrix in units of elements
107:     int kAlignmentA,
108:     /// Element type for B matrix operand
109:     typename ElementB,
110:     /// Layout type for B matrix operand
111:     typename LayoutB,
112:     /// Access granularity of B matrix in units of elements
113:     int kAlignmentB,
114:     /// Element type for internal accumulation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 115-124
```cpp
115:     typename ElementAccumulator,
116:     /// Tag indicating architecture to tune for
117:     typename ArchTag,
118:     /// Threadblock-level tile size (concept: GemmShape)
119:     typename ThreadblockShape,
120:     /// Warp-level tile size (concept: GemmShape)
121:     typename WarpShape,
122:     /// Instruction-level tile size (concept: GemmShape)
123:     typename InstructionShape,
124:     /// Number of stages used in the multistage mainloop
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 125-126
```cpp
125:     int Stages,
126:     /// Operation performed by GEMM
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 127-132
```cpp
127:     typename Operator
128:     >
129: struct DefaultSparseMma<ElementA, LayoutA, kAlignmentA, ElementB, LayoutB,
130:                   kAlignmentB, ElementAccumulator, layout::RowMajor,
131:                   arch::OpClassTensorOp, ArchTag, ThreadblockShape, WarpShape,
132:                   InstructionShape, Stages, Operator, false> {
```
**EN:** Defines DefaultSparseMma, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultSparseMma，用于封装策略、存储或算法行为的辅助类型。

### Lines 133-137
```cpp
133:   static cutlass::arch::CacheOperation::Kind const CacheOpA =
134:       ((sizeof_bits<ElementA>::value * kAlignmentA) == 128)
135:           ? cutlass::arch::CacheOperation::Global
136:           : cutlass::arch::CacheOperation::Always;
137: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 138-144
```cpp
138:   static cutlass::arch::CacheOperation::Kind const CacheOpB =
139:       ((sizeof_bits<ElementB>::value * kAlignmentB) == 128)
140:           ? cutlass::arch::CacheOperation::Global
141:           : cutlass::arch::CacheOperation::Always;
142:   
143: 
144:   // Define the MmaCore components
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 145-152
```cpp
145:   using MmaCore = typename cutlass::gemm::threadblock::DefaultSparseMmaCore<
146:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA,
147:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp,
148:       Stages, Operator, false, CacheOpA, CacheOpB>;
149: 
150:   static int const kSparse = MmaCore::kSparse;
151: 
152:   // Define iterators over tiles from the A operand
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 153-160
```cpp
153:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
154:   using AccessTypeA = cutlass::Array<ElementA, kAlignmentA>;
155:   using IteratorA =
156:       cutlass::transform::threadblock::PredicatedTileAccessIterator<
157:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK / kSparse>,
158:           ElementA, LayoutA, 1, ThreadMapA, AccessTypeA>;
159: 
160:   // Define iterators over tiles from the B operand
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 161-168
```cpp
161:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
162:   using AccessTypeB = cutlass::Array<ElementB, kAlignmentB>;
163:   using IteratorB =
164:       cutlass::transform::threadblock::PredicatedTileAccessIterator<
165:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
166:           ElementB, LayoutB, 0, ThreadMapB, AccessTypeB>;
167: 
168:   // Define iterators over tiles from the E operand
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 169-180
```cpp
169:   using ElementE = typename MmaCore::ElementE;
170:   using LayoutE = typename MmaCore::GmemLayoutE;
171:   using ThreadMapE = typename MmaCore::IteratorThreadMapE;
172:   using AccessTypeE =
173:       cutlass::Array<ElementE, 128 / sizeof_bits<ElementE>::value>;
174:   using IteratorE =
175:       cutlass::transform::threadblock::PredicatedTileAccessIterator<
176:           cutlass::MatrixShape<ThreadblockShape::kM,
177:                                ThreadblockShape::kK / kSparse /
178:                                    MmaCore::kElementsPerElementE>,
179:           ElementE, LayoutE, 1, ThreadMapE, AccessTypeE>;
180: 
```
**EN:** Introduces local type aliases (ElementE, LayoutE, ThreadMapE, AccessTypeE) to simplify downstream template code.
**CN:** 引入本地类型别名（ElementE, LayoutE, ThreadMapE, AccessTypeE），简化后续模板代码。

### Lines 181-181
```cpp
181:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 182-191
```cpp
182:   using ThreadblockMma = cutlass::gemm::threadblock::SparseMmaMultistage<
183:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
184:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
185:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
186:       IteratorE, typename MmaCore::SmemIteratorE, MmaCore::kCacheOpE,
187:       typename MmaCore::MmaPolicy, Stages>;
188: };
189: 
190: ////////////////////////////////////////////////////////////////////////////////
191: 
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 192-196
```cpp
192: } // namespace threadblock
193: } // namespace gemm
194: } // namespace cutlass 
195: 
196: ////////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Threadblock-level tiling and shared memory orchestration  
  **CN:** 线程块级分块与共享内存编排
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Tensor Core instruction mapping  
  **CN:** Tensor Core 指令映射
- **EN:** WMMA programming model  
  **CN:** WMMA 编程模型
- **EN:** Sparse operand handling  
  **CN:** 稀疏操作数处理
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `tag`, `DefaultSparseMma`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
