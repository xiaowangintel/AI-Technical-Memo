# symm.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/reference/host/symm.h`
- **Purpose (EN):** This file declares symmetric matrix multiply for host-side reference utilities.
- **目的 (CN):** 该文件声明了面向主机端参考工具的对称矩阵乘法逻辑。
- **Brief / 简述:** Reference implementation for SYMM update in host-side code.

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
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
13:  * and/or other materials provided with the distribution.
14:  *
15:  * 3. Neither the name of the copyright holder nor the names of its
16:  * contributors may be used to endorse or promote products derived from
17:  * this software without specific prior written permission.
18:  *
19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-30
```cpp
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 31-36
```cpp
31: /*! \file
32:     \brief Reference implementation for SYMM update in host-side code.
33:     
34:     
35: 
36: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 38-38
```cpp
38: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 40-41
```cpp
40: #include "cutlass/blas3.h"
41: #include "cutlass/numeric_conversion.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/blas3.h`, `cutlass/numeric_conversion.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/blas3.h`, `cutlass/numeric_conversion.h`。

### Lines 43-47
```cpp
43: #include "cutlass/tensor_view.h"
44: #include "cutlass/gemm/gemm.h"
45: #include "cutlass/arch/mma.h"
46: #include "cutlass/util/host_tensor.h"
47: #include "cutlass/util/reference/host/gemm.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/tensor_view.h`, `cutlass/gemm/gemm.h`, `cutlass/arch/mma.h`, `cutlass/util/host_tensor.h`, `cutlass/util/reference/host/gemm.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/tensor_view.h`, `cutlass/gemm/gemm.h`, `cutlass/arch/mma.h`, `cutlass/util/host_tensor.h`, `cutlass/util/reference/host/gemm.h`。

### Lines 49-51
```cpp
49: namespace cutlass {
50: namespace reference {
51: namespace host {
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 53-53
```cpp
53: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 55-56
```cpp
55: /// Computes a general matrix product among matrices (tensors of rank=2) pointed to by TensorRef
56: /// objects.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 57-70
```cpp
57: template <
58:   typename ElementA,
59:   typename LayoutA,
60:   SideMode SideModeA,
61:   FillMode FillModeA,
62:   typename ElementB,
63:   typename LayoutB,
64:   typename ElementC,
65:   typename LayoutC,
66:   typename ScalarType,
67:   typename ComputeType,
68:   typename InnerProductOp = multiply_add<ComputeType>,
69:   typename ConvertOp = NumericConverter<ElementC, ScalarType>
70: >
```
- **EN:** Declares or updates local/member state such as `InnerProductOp`, `ConvertOp`.
- **CN:** 声明或更新局部/成员状态，例如 `InnerProductOp`, `ConvertOp`。

### Lines 71-79
```cpp
71: void compute_symm(
72:   gemm::GemmCoord problem_size,
73:   ScalarType alpha,
74:   TensorRef<ElementA, LayoutA> tensor_a,
75:   TensorRef<ElementB, LayoutB> tensor_b,
76:   ScalarType beta,
77:   TensorRef<ElementC, LayoutC> tensor_c,
78:   TensorRef<ElementC, LayoutC> tensor_d,
79:   ComputeType initial_accum) {
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 81-85
```cpp
81:   static_assert(
82:     LayoutA::kRank == 2 &&
83:     LayoutB::kRank == 2 &&
84:     LayoutC::kRank == 2, 
85:     "Tensors must be of rank 2");
```
- **EN:** Declares or updates local/member state such as `kRank`.
- **CN:** 声明或更新局部/成员状态，例如 `kRank`。

### Lines 87-88
```cpp
87:   static_assert(SideModeA != SideMode::kInvalid
88:                 , "Side Mode can either be Left or Right.");
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 90-93
```cpp
90:   static_assert(
91:     FillModeA == FillMode::kLower || 
92:     FillModeA == FillMode::kUpper, 
93:     "Fill Mode can either be Lower or Upper.");
```
- **EN:** Declares or updates local/member state such as `FillModeA`.
- **CN:** 声明或更新局部/成员状态，例如 `FillModeA`。

### Lines 95-96
```cpp
95:   using CompareOp_w_diag =  typename TrMatrixCompareOp<FillModeA, DiagType::kNonUnit>::Type;
96:   using CompareOp_wo_diag = typename TrMatrixCompareOp<FillModeA, DiagType::kZero>::Type;
```
- **EN:** Introduces namespace imports, aliases, or shorthand names used by later code.
- **CN:** 引入后续代码会使用的命名空间、别名或简写名称。

### Lines 98-98
```cpp
98:   // Note: batch is ignored.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 99-100
```cpp
99:   int const M = problem_size.m();
100:   int const N = problem_size.n();
```
- **EN:** Implements `m` and coordinates helper calls such as `n`.
- **CN:** 实现 `m`，并协调调用 `n` 等辅助逻辑。

### Lines 101-101
```cpp
101:   // Assuming correct k-dimension value is passed
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 102-102
```cpp
102:   int const K = problem_size.k();
```
- **EN:** Implements `k` for this file's main component.
- **CN:** 为该文件的核心组件实现 `k`。

### Lines 104-104
```cpp
104:   // Blocking necessary to speedup reference implementation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 105-106
```cpp
105:   int const Mblock = 16;
106:   int const Nblock = 16;
```
- **EN:** Declares or updates local/member state such as `Mblock`, `Nblock`.
- **CN:** 声明或更新局部/成员状态，例如 `Mblock`, `Nblock`。

### Lines 108-111
```cpp
108:   ConvertOp convert_op;
109:   InnerProductOp inner_product_op;
110:   CompareOp_w_diag compare_op_1;
111:   CompareOp_wo_diag compare_op_2;
```
- **EN:** Declares or updates local/member state such as `convert_op`, `inner_product_op`, `compare_op_1`, `compare_op_2`.
- **CN:** 声明或更新局部/成员状态，例如 `convert_op`, `inner_product_op`, `compare_op_1`, `compare_op_2`。

### Lines 113-114
```cpp
113:   for (int row_block = 0; row_block < M; row_block += Mblock) {
114:     for (int col_block = 0; col_block < N; col_block += Nblock) {
```
- **EN:** Declares or updates local/member state such as `row_block`, `M`, `col_block`, `N`.
- **CN:** 声明或更新局部/成员状态，例如 `row_block`, `M`, `col_block`, `N`。

### Lines 116-116
```cpp
116:       ComputeType accum[Mblock][Nblock];
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 118-122
```cpp
118:       for (int j = 0; j < Nblock; j++) {
119:         for (int i = 0; i < Mblock; i++) {
120:           accum[i][j] = initial_accum;
121:         }
122:       }
```
- **EN:** Declares or updates local/member state such as `j`, `Nblock`, `i`, `Mblock`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `Nblock`, `i`, `Mblock`。

### Lines 124-128
```cpp
124:       for (int k_block = 0; k_block < K; ++k_block) {
125:         for (int j = 0; j < Nblock; j++) {
126:           for (int i = 0; i < Mblock; i++) {
127:             int row = row_block + i;
128:             int col = col_block + j;
```
- **EN:** Declares or updates local/member state such as `k_block`, `K`, `j`, `Nblock`.
- **CN:** 声明或更新局部/成员状态，例如 `k_block`, `K`, `j`, `Nblock`。

### Lines 130-134
```cpp
130:             if (row < M && col < N) {
131:               ElementA a_1 = ElementA();
132:               ElementB b_1 = ElementB();
133:               ElementA a_2 = ElementA();
134:               ElementB b_2 = ElementB();
```
- **EN:** Declares or updates local/member state such as `a_1`, `b_1`, `a_2`, `b_2`.
- **CN:** 声明或更新局部/成员状态，例如 `a_1`, `b_1`, `a_2`, `b_2`。

### Lines 136-136
```cpp
136:               // A x B or B x A (with diagonal)
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 137-145
```cpp
137:               if (SideModeA == SideMode::kLeft) {
138:                 a_1 = (compare_op_1(row, k_block)) ? 
139:                       (tensor_a.at(MatrixCoord(row, k_block))) : ElementA();
140:                 b_1 = tensor_b.at(MatrixCoord(k_block, col));
141:               } else if (SideModeA == SideMode::kRight) {
142:                 a_1 = tensor_b.at(MatrixCoord(row, k_block));
143:                 b_1 = (compare_op_1(k_block, col)) ? 
144:                       tensor_a.at(MatrixCoord(k_block, col)) : ElementA();
145:               }
```
- **EN:** Declares or updates local/member state such as `SideModeA`, `a_1`, `b_1`.
- **CN:** 声明或更新局部/成员状态，例如 `SideModeA`, `a_1`, `b_1`。

### Lines 147-148
```cpp
147:               ComputeType compute_a_1(cast_if_scalar<ComputeType>(a_1));
148:               ComputeType compute_b_1(cast_if_scalar<ComputeType>(b_1));
```
- **EN:** Implements `compute_a_1` and coordinates helper calls such as `compute_b_1`.
- **CN:** 实现 `compute_a_1`，并协调调用 `compute_b_1` 等辅助逻辑。

### Lines 150-150
```cpp
150:               accum[i][j] = inner_product_op(compute_a_1, compute_b_1, accum[i][j]);
```
- **EN:** Implements `inner_product_op` for this file's main component.
- **CN:** 为该文件的核心组件实现 `inner_product_op`。

### Lines 152-152
```cpp
152:               // A^T x B or B x A^T (without diagonal)
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 153-161
```cpp
153:               if (SideModeA == SideMode::kLeft) {
154:                 a_2 = (compare_op_2(k_block, row)) ? 
155:                       (tensor_a.at(MatrixCoord(k_block, row))) : ElementA();
156:                 b_2 = tensor_b.at(MatrixCoord(k_block, col));
157:               } else if (SideModeA == SideMode::kRight) {
158:                 a_2 = tensor_b.at(MatrixCoord(row, k_block));
159:                 b_2 = (compare_op_2(col, k_block)) ? 
160:                       tensor_a.at(MatrixCoord(col, k_block)) : ElementA();
161:               }
```
- **EN:** Declares or updates local/member state such as `SideModeA`, `a_2`, `b_2`.
- **CN:** 声明或更新局部/成员状态，例如 `SideModeA`, `a_2`, `b_2`。

### Lines 163-164
```cpp
163:               ComputeType compute_a_2(cast_if_scalar<ComputeType>(a_2));
164:               ComputeType compute_b_2(cast_if_scalar<ComputeType>(b_2));
```
- **EN:** Implements `compute_a_2` and coordinates helper calls such as `compute_b_2`.
- **CN:** 实现 `compute_a_2`，并协调调用 `compute_b_2` 等辅助逻辑。

### Lines 166-170
```cpp
166:               accum[i][j] = inner_product_op(compute_a_2, compute_b_2, accum[i][j]);
167:             }
168:           }
169:         }
170:       }
```
- **EN:** Implements `inner_product_op` for this file's main component.
- **CN:** 为该文件的核心组件实现 `inner_product_op`。

### Lines 172-175
```cpp
172:       for (int j = 0; j < Nblock; j++) {
173:         for (int i = 0; i < Mblock; i++) {
174:           int row = row_block + i;
175:           int col = col_block + j;
```
- **EN:** Declares or updates local/member state such as `j`, `Nblock`, `i`, `Mblock`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `Nblock`, `i`, `Mblock`。

### Lines 177-177
```cpp
177:           MatrixCoord coord = MatrixCoord(row, col);
```
- **EN:** Implements `MatrixCoord` for this file's main component.
- **CN:** 为该文件的核心组件实现 `MatrixCoord`。

### Lines 179-188
```cpp
179:           if (row < M && col < N) {
180:             tensor_d.at(coord) = convert_op(
181:               alpha * ScalarType(accum[i][j]) +
182:               beta * ScalarType(tensor_c.at(coord)));
183:           }
184:         }
185:       }
186:     }
187:   }
188: }
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 190-190
```cpp
190: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 192-193
```cpp
192: /// Computes a general Symm update (tensors of rank=2) pointed to by TensorRef
193: /// objects.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 194-207
```cpp
194: template <
195:   typename ElementA,
196:   typename LayoutA,
197:   SideMode SideModeA,
198:   FillMode FillModeA,
199:   typename ElementB,
200:   typename LayoutB,
201:   typename ElementC,
202:   typename LayoutC,
203:   typename ScalarType,
204:   typename ComputeType,
205:   typename InnerProductOp = multiply_add<ComputeType>,
206:   typename ConvertOp = NumericConverter<ElementC, ScalarType>
207: >
```
- **EN:** Declares or updates local/member state such as `InnerProductOp`, `ConvertOp`.
- **CN:** 声明或更新局部/成员状态，例如 `InnerProductOp`, `ConvertOp`。

### Lines 208-215
```cpp
208: void compute_symm(
209:   gemm::GemmCoord problem_size,
210:   ScalarType alpha,
211:   TensorRef<ElementA, LayoutA> tensor_a,
212:   TensorRef<ElementB, LayoutB> tensor_b,
213:   ScalarType beta,
214:   TensorRef<ElementC, LayoutC> tensor_c,
215:   ComputeType initial_accum) {
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 216-220
```cpp
216:   compute_symm<ElementA, LayoutA, SideModeA, FillModeA, ElementB, LayoutB, ElementC, LayoutC,
217:                ScalarType, ComputeType, InnerProductOp, ConvertOp>(
218:       problem_size, alpha, tensor_a, tensor_b, beta, tensor_c, tensor_c,
219:       initial_accum);
220: }
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 222-222
```cpp
222: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 224-237
```cpp
224: template <
225:   typename ElementA,
226:   typename LayoutA,
227:   SideMode SideModeA,
228:   FillMode FillModeA,
229:   typename ElementB,
230:   typename LayoutB,
231:   typename ElementC,
232:   typename LayoutC,
233:   typename ScalarType,
234:   typename ComputeType,
235:   typename InnerProductOp = cutlass::arch::OpMultiplyAdd
236: >
237: struct Symm;
```
- **EN:** Introduces `Symm`, a type used to support symmetric matrix multiply.
- **CN:** 引入 `Symm`，即一个用于支持对称矩阵乘法的类型。

### Lines 239-239
```cpp
239: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 241-241
```cpp
241: /// Partial specialization for multiply-add
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 242-248
```cpp
242: template <typename ElementA, typename LayoutA, 
243:           SideMode SideModeA, FillMode FillModeA,
244:           typename ElementB, typename LayoutB, 
245:           typename ElementC, typename LayoutC,
246:           typename ScalarType, typename ComputeType>
247: struct Symm<ElementA, LayoutA, SideModeA, FillModeA, ElementB, LayoutB, ElementC, LayoutC, ScalarType,
248:             ComputeType, arch::OpMultiplyAdd> {
```
- **EN:** Introduces `Symm`, a type used to support symmetric matrix multiply.
- **CN:** 引入 `Symm`，即一个用于支持对称矩阵乘法的类型。

### Lines 250-257
```cpp
250:   void operator()(gemm::GemmCoord problem_size, ScalarType alpha,
251:                   TensorRef<ElementA, LayoutA> tensor_a,
252:                   TensorRef<ElementB, LayoutB> tensor_b, ScalarType beta,
253:                   TensorRef<ElementC, LayoutC> tensor_c,
254:                   ComputeType initial_accum = ComputeType(0)) {
255:     static_assert(
256:         LayoutA::kRank == 2 && LayoutB::kRank == 2 && LayoutC::kRank == 2,
257:         "Tensors must be of rank 2");
```
- **EN:** Declares or updates local/member state such as `initial_accum`, `kRank`.
- **CN:** 声明或更新局部/成员状态，例如 `initial_accum`, `kRank`。

### Lines 259-262
```cpp
259:     compute_symm<ElementA, LayoutA, SideModeA, FillModeA, ElementB, LayoutB, ElementC, LayoutC,
260:                  ScalarType, ComputeType, multiply_add<ComputeType>>(
261:         problem_size, alpha, tensor_a, tensor_b, beta, tensor_c, initial_accum);
262:   }
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 264-272
```cpp
264:   void operator()(gemm::GemmCoord problem_size, ScalarType alpha,
265:                   TensorRef<ElementA, LayoutA> tensor_a,
266:                   TensorRef<ElementB, LayoutB> tensor_b, ScalarType beta,
267:                   TensorRef<ElementC, LayoutC> tensor_c,
268:                   TensorRef<ElementC, LayoutC> tensor_d,
269:                   ComputeType initial_accum = ComputeType(0)) {
270:     static_assert(
271:         LayoutA::kRank == 2 && LayoutB::kRank == 2 && LayoutC::kRank == 2,
272:         "Tensors must be of rank 2");
```
- **EN:** Declares or updates local/member state such as `initial_accum`, `kRank`.
- **CN:** 声明或更新局部/成员状态，例如 `initial_accum`, `kRank`。

### Lines 274-278
```cpp
274:     compute_symm<ElementA, LayoutA, SideModeA, FillModeA, ElementB, LayoutB, ElementC, LayoutC,
275:                  ScalarType, ComputeType, multiply_add<ComputeType>>(
276:         problem_size, alpha, tensor_a, tensor_b, beta, tensor_c, tensor_d, initial_accum);
277:   }
278: };
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

### Lines 281-281
```cpp
281: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 283-285
```cpp
283: } // namespace host
284: } // namespace reference
285: } // namespace cutlass
```
- **EN:** Supporting logic for the symmetric matrix multiply implementation.
- **CN:** 对称矩阵乘法实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **Structured type design / 结构化类型设计**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**
- **GEMM specialization / GEMM 特化**
- **Convolution support / 卷积支持**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/blas3.h`, `cutlass/numeric_conversion.h`, `cutlass/tensor_view.h`, `cutlass/gemm/gemm.h`, `cutlass/arch/mma.h`, `cutlass/util/host_tensor.h`, `cutlass/util/reference/host/gemm.h`
