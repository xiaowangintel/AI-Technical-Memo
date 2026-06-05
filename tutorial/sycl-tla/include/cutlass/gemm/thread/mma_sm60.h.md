# mma_sm60.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/thread/mma_sm60.h`
- **Purpose (EN):** Implements per-thread matrix multiply helper operators.
- **用途 (CN):** 实现线程级矩阵乘法辅助算子。

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
32:     \brief Templates exposing architecture support for multiply-add operations
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

### Lines 37-46
```cpp
37: #include "cutlass/cutlass.h"
38: #include "cutlass/tensor_ref.h"
39: #include "cutlass/layout/matrix.h"
40: #include "cutlass/gemm/gemm.h"
41: #include "cutlass/gemm/thread/mma.h"
42: #include "cutlass/functional.h"
43: #include "cutlass/reduction/thread/reduce.h"
44: 
45: /////////////////////////////////////////////////////////////////////////////////////////////////
46: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, layout types.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、布局类型。

### Lines 47-57
```cpp
47: namespace cutlass {
48: namespace gemm {
49: namespace thread {
50: 
51: /////////////////////////////////////////////////////////////////////////////////////////////////
52: 
53: namespace detail {
54: 
55: /// Structure to compute the matrix product for HFMA
56: template <
57:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Enters namespace scope (cutlass::gemm::thread::detail) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass::gemm::thread::detail），组织 GEMM 抽象层。

### Lines 58-68
```cpp
58:   typename Shape,
59: 
60:   /// Layout of A matrix (concept: MatrixLayout)
61:   typename LayoutA,
62: 
63:   /// Layout of B matrix (concept: MatrixLayout)
64:   typename LayoutB,
65: 
66:   /// Layout of C matrix (concept: MatrixLayout)
67:   typename LayoutC,
68: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 69-69
```cpp
69:   /// Type of GEMM inner vs outer product
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 70-74
```cpp
70:   bool
71: >
72: struct Mma_HFMA2;
73: 
74: 
```
**EN:** Defines Mma_HFMA2, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Mma_HFMA2，用于封装策略、存储或算法行为的辅助类型。

### Lines 75-78
```cpp
75: /////////////////////////////
76: // Specialization for NNN  //
77: /////////////////////////////
78: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 79-88
```cpp
79: template <typename Shape_>
80: struct Mma_HFMA2 <
81:   Shape_,
82:   layout::ColumnMajor,
83:   layout::ColumnMajor,
84:   layout::ColumnMajor,
85:   true
86:   > {
87: 
88:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 89-99
```cpp
89:   using Shape = Shape_;
90: 
91:    /// A operand storage
92:   using FragmentA = Array<half_t, Shape::kMK>;
93: 
94:   /// B operand storage
95:   using FragmentB = Array<half_t, Shape::kKN>;
96: 
97:   /// C operand storage
98:   using FragmentC = Array<half_t, Shape::kMN>;
99: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 100-102
```cpp
100:   /// Underlying mathematical operator
101:   using Operator = arch::OpMultiplyAdd;
102: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 103-107
```cpp
103:   static_assert(
104:     !(Shape::kM % 2),
105:     "Mma_HFMA2 requires the M dimension to be divisible by 2."
106:   );
107: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 108-112
```cpp
108:   //
109:   // Methods
110:   //
111: 
112:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 113-122
```cpp
113:   CUTLASS_HOST_DEVICE
114:   void operator()(
115:     FragmentC & D,
116:     FragmentA const & A,
117:     FragmentB const & B,
118:     FragmentC const & C) {
119: 
120:     /// Initialize output with input
121:     D = C;
122: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 123-123
```cpp
123:     /// Use 1x1x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 124-134
```cpp
124:     using Mma = arch::Mma<
125:       gemm::GemmShape<2,1,1>,
126:       1,
127:       half_t,
128:       layout::ColumnMajor,
129:       half_t,
130:       layout::ColumnMajor,
131:       half_t,
132:       layout::ColumnMajor,
133:       arch::OpMultiplyAdd>;
134: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 135-140
```cpp
135:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
136:     Array<half_t, 2> const *ptr_A = reinterpret_cast<Array<half_t, 2> const *>(&A);
137:     Array<half_t, 1> const *ptr_B = reinterpret_cast<Array<half_t, 1> const *>(&B);
138: 
139:     Mma mma;
140: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 141-143
```cpp
141:     CUTLASS_PRAGMA_UNROLL
142:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
143: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 144-146
```cpp
144:       CUTLASS_PRAGMA_UNROLL
145:       for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
146: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 147-151
```cpp
147:         CUTLASS_PRAGMA_UNROLL
148:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
149: 
150:             Array<half_t, 2> tmp { ptr_D[n*Shape::kM/2 + m] };
151: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 152-161
```cpp
152:             mma(
153:                 tmp,
154:                 ptr_A[k*Shape::kM/2 + m],
155:                 ptr_B[n*Shape::kK + k],
156:                 tmp);
157: 
158:             ptr_D[n*Shape::kM/2 + m] = tmp;
159:         }
160:       }
161:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 162-164
```cpp
162:   }
163: };
164: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 165-168
```cpp
165: /////////////////////////////
166: // Specialization for NNT  //
167: /////////////////////////////
168: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 169-178
```cpp
169: template <typename Shape_>
170: struct Mma_HFMA2<
171:   Shape_,
172:   layout::ColumnMajor,
173:   layout::ColumnMajor,
174:   layout::RowMajor,
175:   true
176:   > {
177: 
178:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 179-189
```cpp
179:   using Shape = Shape_;
180: 
181:    /// A operand storage
182:   using FragmentA = Array<half_t, Shape::kMK>;
183: 
184:   /// B operand storage
185:   using FragmentB = Array<half_t, Shape::kKN>;
186: 
187:   /// C operand storage
188:   using FragmentC = Array<half_t, Shape::kMN>;
189: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 190-192
```cpp
190:   /// Underlying mathematical operator
191:   using Operator = arch::OpMultiplyAdd;
192: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 193-197
```cpp
193:   static_assert(
194:     !(Shape::kN % 2),
195:     "Mma_HFMA2 requires the N dimension to be divisible by 2."
196:   );
197: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 198-202
```cpp
198:   //
199:   // Methods
200:   //
201: 
202:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 203-212
```cpp
203:   CUTLASS_HOST_DEVICE
204:   void operator()(
205:     FragmentC & D,
206:     FragmentA const & A,
207:     FragmentB const & B,
208:     FragmentC const & C) {
209: 
210:     /// Initialize output with input
211:     D = C;
212: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 213-213
```cpp
213:     /// Use 1x2x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 214-224
```cpp
214:     using Mma = arch::Mma<
215:       gemm::GemmShape<1,2,1>,
216:       1,
217:       half_t,
218:       layout::ColumnMajor,
219:       half_t,
220:       layout::ColumnMajor,
221:       half_t,
222:       layout::RowMajor,
223:       arch::OpMultiplyAdd>;
224: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 225-230
```cpp
225:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
226:     Array<half_t, 1> const *ptr_A = reinterpret_cast<Array<half_t, 1> const *>(&A);
227:     Array<half_t, 2> const *ptr_B = reinterpret_cast<Array<half_t, 2> const *>(&B);
228: 
229:     Mma mma;
230: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 231-233
```cpp
231:     CUTLASS_PRAGMA_UNROLL
232:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
233: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 234-236
```cpp
234:         CUTLASS_PRAGMA_UNROLL
235:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
236: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 237-241
```cpp
237:           CUTLASS_PRAGMA_UNROLL
238:           for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
239: 
240:             Array<half_t, 2> tmp { ptr_D[m*Shape::kN/2 + n] };
241: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 242-245
```cpp
242:             Array<half_t, 2> tmp_B;
243:             tmp_B[0] = ptr_B->at(2*n*Shape::kK + k);
244:             tmp_B[1] = ptr_B->at((2*n+1)*Shape::kK + k);
245: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 246-255
```cpp
246:             mma(
247:                 tmp,
248:                 ptr_A[k*Shape::kM + m],
249:                 tmp_B,
250:                 tmp);
251: 
252:             ptr_D[m*Shape::kN/2 + n] = tmp;
253:         }
254:       }
255:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 256-259
```cpp
256:   }
257: };
258: 
259: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 260-263
```cpp
260: /////////////////////////////
261: // Specialization for NTN  //
262: /////////////////////////////
263: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 264-273
```cpp
264: template <typename Shape_>
265: struct Mma_HFMA2 <
266:   Shape_,
267:   layout::ColumnMajor,
268:   layout::RowMajor,
269:   layout::ColumnMajor,
270:   true
271:   > {
272: 
273:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 274-284
```cpp
274:   using Shape = Shape_;
275: 
276:   /// A operand storage
277:   using FragmentA = Array<half_t, Shape::kMK>;
278: 
279:   /// B operand storage
280:   using FragmentB = Array<half_t, Shape::kKN>;
281: 
282:   /// C operand storage
283:   using FragmentC = Array<half_t, Shape::kMN>;
284: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 285-287
```cpp
285:   /// Underlying mathematical operator
286:   using Operator = arch::OpMultiplyAdd;
287: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 288-292
```cpp
288:   static_assert(
289:     !(Shape::kM % 2),
290:     "Mma_HFMA2 requires the GEMM M dimension to be divisible by 2."
291:   );
292: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 293-297
```cpp
293:   //
294:   // Methods
295:   //
296: 
297:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 298-307
```cpp
298:   CUTLASS_HOST_DEVICE
299:   void operator()(
300:     FragmentC & D,
301:     FragmentA const & A,
302:     FragmentB const & B,
303:     FragmentC const & C) {
304: 
305:     /// Initialize output with input
306:     D = C;
307: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 308-318
```cpp
308:     using Mma = arch::Mma<
309:       gemm::GemmShape<2,1,1>,
310:       1,
311:       half_t,
312:       layout::ColumnMajor,
313:       half_t,
314:       layout::RowMajor,
315:       half_t,
316:       layout::ColumnMajor,
317:       arch::OpMultiplyAdd>;
318: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 319-324
```cpp
319:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
320:     Array<half_t, 2> const *ptr_A = reinterpret_cast<Array<half_t, 2> const *>(&A);
321:     Array<half_t, 1> const *ptr_B = reinterpret_cast<Array<half_t, 1> const *>(&B);
322: 
323:     Mma mma;
324: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 325-327
```cpp
325:     CUTLASS_PRAGMA_UNROLL
326:     for (int k = 0; k < Shape::kK / Mma::Shape::kK; ++k) {
327: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 328-330
```cpp
328:         CUTLASS_PRAGMA_UNROLL
329:         for (int m = 0; m < Shape::kM / Mma::Shape::kM; ++m) {
330: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 331-335
```cpp
331:           CUTLASS_PRAGMA_UNROLL
332:           for (int n = 0; n < Shape::kN / Mma::Shape::kN; ++n) {
333: 
334:           Array<half_t, 2> tmp { ptr_D[m + n * Shape::kM/2] };
335: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 336-345
```cpp
336:           mma(
337:             tmp,
338:             ptr_A[m + k * Shape::kM/2],
339:             ptr_B[k * Shape::kN + n],
340:             tmp);
341: 
342:           ptr_D[m + n * Shape::kM/2] = tmp;
343:         }
344:       }
345:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 346-348
```cpp
346:   }
347: };
348: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 349-352
```cpp
349: /////////////////////////////
350: // Specialization for NTT  //
351: /////////////////////////////
352: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 353-362
```cpp
353: template <typename Shape_>
354: struct Mma_HFMA2<
355:   Shape_,
356:   layout::ColumnMajor,
357:   layout::RowMajor,
358:   layout::RowMajor,
359:   true
360:   > {
361: 
362:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 363-373
```cpp
363:   using Shape = Shape_;
364: 
365:   /// A operand storage
366:   using FragmentA = Array<half_t, Shape::kMK>;
367: 
368:   /// B operand storage
369:   using FragmentB = Array<half_t, Shape::kKN>;
370: 
371:   /// C operand storage
372:   using FragmentC = Array<half_t, Shape::kMN>;
373: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 374-376
```cpp
374:   /// Underlying mathematical operator
375:   using Operator = arch::OpMultiplyAdd;
376: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 377-381
```cpp
377:   static_assert(
378:     !(Shape::kN % 2),
379:     "Mma_HFMA2 requires the N dimension to be divisible by 2."
380:   );
381: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 382-386
```cpp
382:   //
383:   // Methods
384:   //
385: 
386:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 387-396
```cpp
387:   CUTLASS_HOST_DEVICE
388:   void operator()(
389:     FragmentC & D,
390:     FragmentA const & A,
391:     FragmentB const & B,
392:     FragmentC const & C) {
393: 
394:     /// Initialize output with input
395:     D = C;
396: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 397-397
```cpp
397:     /// Use 1x2x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 398-408
```cpp
398:     using Mma = arch::Mma<
399:       gemm::GemmShape<1,2,1>,
400:       1,
401:       half_t,
402:       layout::ColumnMajor,
403:       half_t,
404:       layout::RowMajor,
405:       half_t,
406:       layout::RowMajor,
407:       arch::OpMultiplyAdd>;
408: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 409-414
```cpp
409:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
410:     Array<half_t, 1> const *ptr_A = reinterpret_cast<Array<half_t, 1> const *>(&A);
411:     Array<half_t, 2> const *ptr_B = reinterpret_cast<Array<half_t, 2> const *>(&B);
412: 
413:     Mma mma;
414: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 415-417
```cpp
415:     CUTLASS_PRAGMA_UNROLL
416:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
417: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 418-420
```cpp
418:         CUTLASS_PRAGMA_UNROLL
419:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
420: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 421-425
```cpp
421:           CUTLASS_PRAGMA_UNROLL
422:           for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
423: 
424:             Array<half_t, 2> tmp { ptr_D[m*Shape::kN/2 + n] };
425: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 426-435
```cpp
426:             mma(
427:                 tmp,
428:                 ptr_A[k*Shape::kM + m],
429:                 ptr_B[k*Shape::kN/2 + n],
430:                 tmp);
431: 
432:             ptr_D[m*Shape::kN/2 + n] = tmp;
433:         }
434:       }
435:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 436-439
```cpp
436:   }
437: };
438: 
439: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 440-443
```cpp
440: /////////////////////////////
441: // Specialization for TNN  //
442: /////////////////////////////
443: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 444-453
```cpp
444: template <typename Shape_>
445: struct Mma_HFMA2 <
446:   Shape_,
447:   layout::RowMajor,
448:   layout::ColumnMajor,
449:   layout::ColumnMajor,
450:   true
451:   > {
452: 
453:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 454-464
```cpp
454:   using Shape = Shape_;
455: 
456:   /// A operand storage
457:   using FragmentA = Array<half_t, Shape::kMK>;
458: 
459:   /// B operand storage
460:   using FragmentB = Array<half_t, Shape::kKN>;
461: 
462:   /// C operand storage
463:   using FragmentC = Array<half_t, Shape::kMN>;
464: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 465-467
```cpp
465:   /// Underlying mathematical operator
466:   using Operator = arch::OpMultiplyAdd;
467: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 468-472
```cpp
468:   static_assert(
469:     !(Shape::kM % 2),
470:     "Mma_HFMA2 requires the M dimension to be divisible by 2."
471:   );
472: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 473-477
```cpp
473:   //
474:   // Methods
475:   //
476: 
477:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 478-487
```cpp
478:   CUTLASS_HOST_DEVICE
479:   void operator()(
480:     FragmentC & D,
481:     FragmentA const & A,
482:     FragmentB const & B,
483:     FragmentC const & C) {
484: 
485:     /// Initialize output with input
486:     D = C;
487: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 488-488
```cpp
488:     /// Use 1x1x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 489-499
```cpp
489:     using Mma = arch::Mma<
490:       gemm::GemmShape<2,1,1>,
491:       1,
492:       half_t,
493:       layout::RowMajor,
494:       half_t,
495:       layout::ColumnMajor,
496:       half_t,
497:       layout::ColumnMajor,
498:       arch::OpMultiplyAdd>;
499: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 500-505
```cpp
500:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
501:     Array<half_t, 2> const *ptr_A = reinterpret_cast<Array<half_t, 2> const *>(&A);
502:     Array<half_t, 1> const *ptr_B = reinterpret_cast<Array<half_t, 1> const *>(&B);
503: 
504:     Mma mma;
505: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 506-508
```cpp
506:     CUTLASS_PRAGMA_UNROLL
507:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
508: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 509-511
```cpp
509:       CUTLASS_PRAGMA_UNROLL
510:       for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
511: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 512-516
```cpp
512:         CUTLASS_PRAGMA_UNROLL
513:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
514: 
515:             Array<half_t, 2> tmp { ptr_D[n*Shape::kM/2 + m] };
516: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 517-520
```cpp
517:             Array<half_t, 2> tmp_A;
518:             tmp_A[0] = ptr_A->at(2*m*Shape::kK + k);
519:             tmp_A[1] = ptr_A->at((2*m+1)*Shape::kK + k);
520: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 521-530
```cpp
521:             mma(
522:                 tmp,
523:                 tmp_A,
524:                 ptr_B[n*Shape::kK + k],
525:                 tmp);
526: 
527:             ptr_D[n*Shape::kM/2 + m] = tmp;
528:         }
529:       }
530:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 531-533
```cpp
531:   }
532: };
533: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 534-537
```cpp
534: /////////////////////////////
535: // Specialization for TNT  //
536: /////////////////////////////
537: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 538-547
```cpp
538: template <typename Shape_>
539: struct Mma_HFMA2 <
540:   Shape_,
541:   layout::RowMajor,
542:   layout::ColumnMajor,
543:   layout::RowMajor,
544:   true
545:   > {
546: 
547:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 548-558
```cpp
548:   using Shape = Shape_;
549: 
550:    /// A operand storage
551:   using FragmentA = Array<half_t, Shape::kMK>;
552: 
553:   /// B operand storage
554:   using FragmentB = Array<half_t, Shape::kKN>;
555: 
556:   /// C operand storage
557:   using FragmentC = Array<half_t, Shape::kMN>;
558: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 559-561
```cpp
559:   /// Underlying mathematical operator
560:   using Operator = arch::OpMultiplyAdd;
561: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 562-566
```cpp
562:   static_assert(
563:     !(Shape::kN % 2),
564:     "Mma_HFMA2 requires the N dimension to be divisible by 2."
565:   );
566: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 567-571
```cpp
567:   //
568:   // Methods
569:   //
570: 
571:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 572-581
```cpp
572:   CUTLASS_HOST_DEVICE
573:   void operator()(
574:     FragmentC & D,
575:     FragmentA const & A,
576:     FragmentB const & B,
577:     FragmentC const & C) {
578: 
579:     /// Initialize output with input
580:     D = C;
581: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 582-582
```cpp
582:     /// Use 1x2x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 583-593
```cpp
583:     using Mma = arch::Mma<
584:       gemm::GemmShape<1,2,1>,
585:       1,
586:       half_t,
587:       layout::RowMajor,
588:       half_t,
589:       layout::ColumnMajor,
590:       half_t,
591:       layout::RowMajor,
592:       arch::OpMultiplyAdd>;
593: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 594-599
```cpp
594:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
595:     Array<half_t, 1> const *ptr_A = reinterpret_cast<Array<half_t, 1> const *>(&A);
596:     Array<half_t, 2> const *ptr_B = reinterpret_cast<Array<half_t, 2> const *>(&B);
597: 
598:     Mma mma;
599: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 600-602
```cpp
600:     CUTLASS_PRAGMA_UNROLL
601:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
602: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 603-605
```cpp
603:         CUTLASS_PRAGMA_UNROLL
604:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
605: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 606-610
```cpp
606:           CUTLASS_PRAGMA_UNROLL
607:           for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
608: 
609:             Array<half_t, 2> tmp { ptr_D[m*Shape::kN/2 + n] };
610: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 611-614
```cpp
611:             Array<half_t, 2> tmp_B;
612:             tmp_B[0] = ptr_B->at(2*n*Shape::kK + k);
613:             tmp_B[1] = ptr_B->at((2*n+1)*Shape::kK + k);
614: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 615-624
```cpp
615:             mma(
616:                 tmp,
617:                 ptr_A[m*Shape::kK + k],
618:                 tmp_B,
619:                 tmp);
620: 
621:             ptr_D[m*Shape::kN/2 + n] = tmp;
622:         }
623:       }
624:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 625-627
```cpp
625:   }
626: };
627: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 628-631
```cpp
628: /////////////////////////////
629: // Specialization for TTN  //
630: /////////////////////////////
631: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 632-641
```cpp
632: template <typename Shape_>
633: struct Mma_HFMA2 <
634:   Shape_,
635:   layout::RowMajor,
636:   layout::RowMajor,
637:   layout::ColumnMajor,
638:   true
639:   > {
640: 
641:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 642-652
```cpp
642:   using Shape = Shape_;
643: 
644:    /// A operand storage
645:   using FragmentA = Array<half_t, Shape::kMK>;
646: 
647:   /// B operand storage
648:   using FragmentB = Array<half_t, Shape::kKN>;
649: 
650:   /// C operand storage
651:   using FragmentC = Array<half_t, Shape::kMN>;
652: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 653-655
```cpp
653:   /// Underlying mathematical operator
654:   using Operator = arch::OpMultiplyAdd;
655: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 656-660
```cpp
656:   static_assert(
657:     !(Shape::kM % 2),
658:     "Mma_HFMA2 requires the M dimension to be divisible by 2."
659:   );
660: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 661-665
```cpp
661:   //
662:   // Methods
663:   //
664: 
665:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 666-675
```cpp
666:   CUTLASS_HOST_DEVICE
667:   void operator()(
668:     FragmentC & D,
669:     FragmentA const & A,
670:     FragmentB const & B,
671:     FragmentC const & C) {
672: 
673:     /// Initialize output with input
674:     D = C;
675: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 676-676
```cpp
676:     /// Use 1x2x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 677-687
```cpp
677:     using Mma = arch::Mma<
678:       gemm::GemmShape<2,1,1>,
679:       1,
680:       half_t,
681:       layout::RowMajor,
682:       half_t,
683:       layout::RowMajor,
684:       half_t,
685:       layout::ColumnMajor,
686:       arch::OpMultiplyAdd>;
687: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 688-693
```cpp
688:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
689:     Array<half_t, 2> const *ptr_A = reinterpret_cast<Array<half_t, 2> const *>(&A);
690:     Array<half_t, 1> const *ptr_B = reinterpret_cast<Array<half_t, 1> const *>(&B);
691: 
692:     Mma mma;
693: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 694-696
```cpp
694:     CUTLASS_PRAGMA_UNROLL
695:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
696: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 697-699
```cpp
697:       CUTLASS_PRAGMA_UNROLL
698:       for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
699: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 700-704
```cpp
700:         CUTLASS_PRAGMA_UNROLL
701:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
702: 
703:             Array<half_t, 2> tmp { ptr_D[n*Shape::kM/2 + m] };
704: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 705-708
```cpp
705:             Array<half_t, 2> tmp_A;
706:             tmp_A[0] = ptr_A->at(2*m*Shape::kK + k);
707:             tmp_A[1] = ptr_A->at((2*m+1)*Shape::kK + k);
708: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 709-718
```cpp
709:             mma(
710:                 tmp,
711:                 tmp_A,
712:                 ptr_B[k*Shape::kN + n],
713:                 tmp);
714: 
715:             ptr_D[n*Shape::kM/2 + m] = tmp;
716:         }
717:       }
718:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 719-722
```cpp
719:   }
720: };
721: 
722: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 723-726
```cpp
723: /////////////////////////////
724: // Specialization for TTT  //
725: /////////////////////////////
726: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 727-736
```cpp
727: template <typename Shape_>
728: struct Mma_HFMA2<
729:   Shape_,
730:   layout::RowMajor,
731:   layout::RowMajor,
732:   layout::RowMajor,
733:   true
734:   > {
735: 
736:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 737-747
```cpp
737:   using Shape = Shape_;
738: 
739:   /// A operand storage
740:   using FragmentA = Array<half_t, Shape::kMK>;
741: 
742:   /// B operand storage
743:   using FragmentB = Array<half_t, Shape::kKN>;
744: 
745:   /// C operand storage
746:   using FragmentC = Array<half_t, Shape::kMN>;
747: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 748-750
```cpp
748:   /// Underlying mathematical operator
749:   using Operator = arch::OpMultiplyAdd;
750: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 751-755
```cpp
751:   static_assert(
752:     !(Shape::kN % 2),
753:     "Mma_HFMA2 requires the N dimension to be divisible by 2."
754:   );
755: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 756-760
```cpp
756:   //
757:   // Methods
758:   //
759: 
760:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 761-770
```cpp
761:   CUTLASS_HOST_DEVICE
762:   void operator()(
763:     FragmentC & D,
764:     FragmentA const & A,
765:     FragmentB const & B,
766:     FragmentC const & C) {
767: 
768:     /// Initialize output with input
769:     D = C;
770: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 771-771
```cpp
771:     /// Use 1x2x1 HFMA2 sequence for bulk of computation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 772-782
```cpp
772:     using Mma = arch::Mma<
773:       gemm::GemmShape<1,2,1>,
774:       1,
775:       half_t,
776:       layout::RowMajor,
777:       half_t,
778:       layout::RowMajor,
779:       half_t,
780:       layout::RowMajor,
781:       arch::OpMultiplyAdd>;
782: 
```
**EN:** Introduces local type aliases (Mma) to simplify downstream template code.
**CN:** 引入本地类型别名（Mma），简化后续模板代码。

### Lines 783-788
```cpp
783:     Array<half_t, 2> *ptr_D = reinterpret_cast<Array<half_t, 2> *>(&D);
784:     Array<half_t, 1> const *ptr_A = reinterpret_cast<Array<half_t, 1> const *>(&A);
785:     Array<half_t, 2> const *ptr_B = reinterpret_cast<Array<half_t, 2> const *>(&B);
786: 
787:     Mma mma;
788: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 789-791
```cpp
789:     CUTLASS_PRAGMA_UNROLL
790:     for(auto k=0; k <  Shape::kK / Mma::Shape::kK; k++){
791: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 792-794
```cpp
792:         CUTLASS_PRAGMA_UNROLL
793:         for(auto n=0; n < Shape::kN / Mma::Shape::kN; n++){
794: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 795-799
```cpp
795:           CUTLASS_PRAGMA_UNROLL
796:           for(auto m=0; m < Shape::kM / Mma::Shape::kM; m++){
797: 
798:             Array<half_t, 2> tmp { ptr_D[m*Shape::kN/2 + n] };
799: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 800-809
```cpp
800:             mma(
801:                 tmp,
802:                 ptr_A[m*Shape::kK + k],
803:                 ptr_B[k*Shape::kN/2 + n],
804:                 tmp);
805: 
806:             ptr_D[m*Shape::kN/2 + n] = tmp;
807:         }
808:       }
809:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 810-812
```cpp
810:   }
811: };
812: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 813-816
```cpp
813: /////////////////////////////////////////////////////////////////////
814: // Specialization for TNT + Inner Product  or 1x1x2K + LayoutC = T //
815: /////////////////////////////////////////////////////////////////////
816: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 817-826
```cpp
817: template <typename Shape_, typename LayoutA, typename LayoutB>
818: struct Mma_HFMA2<
819:   Shape_,
820:   LayoutA,
821:   LayoutB,
822:   layout::RowMajor,
823:   false
824:   > {
825: 
826:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 827-837
```cpp
827:   using Shape = Shape_;
828: 
829:   /// A operand storage
830:   using FragmentA = Array<half_t, Shape::kMK>;
831: 
832:   /// B operand storage
833:   using FragmentB = Array<half_t, Shape::kKN>;
834: 
835:   /// C operand storage
836:   using FragmentC = Array<half_t, Shape::kMN>;
837: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 838-840
```cpp
838:   /// Underlying mathematical operator
839:   using Operator = arch::OpMultiplyAdd;
840: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 841-845
```cpp
841:   static_assert(
842:     !(Shape::kK % 2),
843:     "Mma_HFMA2 requires the K dimension to be divisible by 2."
844:   );
845: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 846-850
```cpp
846:   //
847:   // Methods
848:   //
849: 
850:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 851-860
```cpp
851:   CUTLASS_HOST_DEVICE
852:   void operator()(
853:     FragmentC & D,
854:     FragmentA const & A,
855:     FragmentB const & B,
856:     FragmentC const & C) {
857: 
858:     /// Initialize output with input
859:     D = C;
860: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 861-863
```cpp
861:     /// Use 1x1x2 HFMA2 sequence for bulk of computation
862:     using GemmShape = gemm::GemmShape<1,1,2>;
863: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 864-868
```cpp
864:     Array<half_t, 1> *ptr_D = reinterpret_cast<Array<half_t, 1> *>(&D);
865:     Array<half_t, 2> const *ptr_A = reinterpret_cast<Array<half_t, 2> const *>(&A);
866:     Array<half_t, 2> const *ptr_B = reinterpret_cast<Array<half_t, 2> const *>(&B);
867: 
868:     // Inner product is calculated using MACs, followed by final reduction
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 869-871
```cpp
869:     multiply_add<Array<half_t, 2>> mac;
870:     cutlass::reduction::thread::Reduce< plus<half_t>, Array<half_t, 2> > reduce;
871: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 872-874
```cpp
872:     CUTLASS_PRAGMA_UNROLL
873:     for(auto n=0; n < Shape::kN / GemmShape::kN; n++){ 
874: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 875-877
```cpp
875:       CUTLASS_PRAGMA_UNROLL
876:       for(auto m=0; m < Shape::kM / GemmShape::kM; m++){
877: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 878-882
```cpp
878:         Array<half_t, 2> tmp_C;
879:         tmp_C.clear();
880:         Array<half_t, 1> *ptr_tmp_C = reinterpret_cast<Array<half_t, 1> *>(&tmp_C);
881:         ptr_tmp_C[0] = ptr_D[n*Shape::kM + m];
882: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 883-887
```cpp
883:         CUTLASS_PRAGMA_UNROLL
884:         for(auto k=0; k <  Shape::kK / GemmShape::kK; k++){ 
885:           tmp_C = mac(ptr_A[m*Shape::kK/2 + k], ptr_B[n*Shape::kK/2 + k], tmp_C);
886:         }
887: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 888-897
```cpp
888:         Array<half_t, 1> res;
889:         Array<half_t, 1> *ptr_res = &res;
890:         res = reduce(tmp_C);
891: 
892:         ptr_D[m*Shape::kN + n] = ptr_res[0];
893:       }
894:     }
895:   }
896: };
897: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 898-901
```cpp
898: /////////////////////////////////////////////////////////////////////
899: // Specialization for TNN + Inner Product  or 1x1x2K + LayoutC = N //
900: /////////////////////////////////////////////////////////////////////
901: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 902-911
```cpp
902: template <typename Shape_, typename LayoutA, typename LayoutB>
903: struct Mma_HFMA2<
904:   Shape_,
905:   LayoutA,
906:   LayoutB,
907:   layout::ColumnMajor,
908:   false
909:   > {
910: 
911:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Declares template parameters and begins the definition of Mma_HFMA2.
**CN:** 声明模板参数并开始定义 Mma_HFMA2。

### Lines 912-922
```cpp
912:   using Shape = Shape_;
913: 
914:   /// A operand storage
915:   using FragmentA = Array<half_t, Shape::kMK>;
916: 
917:   /// B operand storage
918:   using FragmentB = Array<half_t, Shape::kKN>;
919: 
920:   /// C operand storage
921:   using FragmentC = Array<half_t, Shape::kMN>;
922: 
```
**EN:** Introduces local type aliases (Shape, FragmentA, FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, FragmentA, FragmentB, FragmentC），简化后续模板代码。

### Lines 923-925
```cpp
923:   /// Underlying mathematical operator
924:   using Operator = arch::OpMultiplyAdd;
925: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 926-930
```cpp
926:   static_assert(
927:     !(Shape::kK % 2),
928:     "Mma_HFMA2 requires the K dimension to be divisible by 2."
929:   );
930: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 931-935
```cpp
931:   //
932:   // Methods
933:   //
934: 
935:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 936-945
```cpp
936:   CUTLASS_HOST_DEVICE
937:   void operator()(
938:     FragmentC & D,
939:     FragmentA const & A,
940:     FragmentB const & B,
941:     FragmentC const & C) {
942: 
943:     /// Initialize output with input
944:     D = C;
945: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 946-948
```cpp
946:     /// Use 1x1x2 HFMA2 sequence for bulk of computation
947:     using GemmShape= gemm::GemmShape<1,1,2>;
948: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 949-953
```cpp
949:     Array<half_t, 1> *ptr_D = reinterpret_cast<Array<half_t, 1> *>(&D);
950:     Array<half_t, 2> const *ptr_A = reinterpret_cast<Array<half_t, 2> const *>(&A);
951:     Array<half_t, 2> const *ptr_B = reinterpret_cast<Array<half_t, 2> const *>(&B);
952: 
953:     // Inner product is calculated using MACs, followed by final reduction
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 954-956
```cpp
954:     multiply_add<Array<half_t, 2>> mac;
955:     cutlass::reduction::thread::Reduce< plus<half_t>, Array<half_t, 2> > reduce;
956: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 957-959
```cpp
957:     CUTLASS_PRAGMA_UNROLL
958:     for(auto n=0; n < Shape::kN / GemmShape::kN; n++){ 
959: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 960-962
```cpp
960:       CUTLASS_PRAGMA_UNROLL
961:       for(auto m=0; m < Shape::kM / GemmShape::kM; m++){
962: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 963-967
```cpp
963:         Array<half_t, 2> tmp_C;
964:         tmp_C.clear();
965:         Array<half_t, 1> *ptr_tmp_C = reinterpret_cast<Array<half_t, 1> *>(&tmp_C);
966:         ptr_tmp_C[0] = ptr_D[n*Shape::kM + m];
967: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 968-974
```cpp
968:         CUTLASS_PRAGMA_UNROLL
969:         for(auto k=0; k <  Shape::kK / GemmShape::kK; k++){ 
970: 
971:           tmp_C = mac(ptr_A[m*Shape::kK/2 + k], ptr_B[n*Shape::kK/2 + k], tmp_C);
972: 
973:         }
974: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 975-984
```cpp
975:         Array<half_t, 1> res;
976:         Array<half_t, 1> *ptr_res = &res;
977:         res = reduce(tmp_C);
978: 
979:         ptr_D[n*Shape::kM + m] = ptr_res[0];
980:       }
981:     }
982:   }
983: };
984: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 985-991
```cpp
985: } // namespace detail
986: 
987: /////////////////////////////////////////////////////////////////////////////////////////////////
988: 
989: /// Structure to compute the matrix product
990: template <
991:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 992-1003
```cpp
992:   typename Shape_, typename LayoutA, typename LayoutB, typename LayoutC
993: >
994: struct Mma<
995:   Shape_,
996:   half_t,
997:   LayoutA,
998:   half_t,
999:   LayoutB,
1000:   half_t,
1001:   LayoutC,
1002:   arch::OpMultiplyAdd
1003:   > {
```
**EN:** Defines Mma, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Mma，用于封装策略、存储或算法行为的辅助类型。

### Lines 1004-1013
```cpp
1004: 
1005:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
1006:   using Shape = Shape_;
1007: 
1008:   /// Data type of operand A
1009:   using ElementA = half_t;
1010: 
1011:   /// Data type of operand B
1012:   using ElementB = half_t;
1013: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1014-1023
```cpp
1014:   /// Element type of operand C
1015:   using ElementC = half_t;
1016: 
1017:   /// Underlying mathematical operator
1018:   using Operator = arch::OpMultiplyAdd;
1019: 
1020:   /// A operand storage
1021:   using FragmentA = Array<ElementA, Shape::kMK>;
1022: 
1023:   /// B operand storage
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1024-1028
```cpp
1024:   using FragmentB = Array<ElementB, Shape::kKN>;
1025: 
1026:   /// C operand storage
1027:   using FragmentC = Array<ElementC, Shape::kMN>;
1028: 
```
**EN:** Introduces local type aliases (FragmentB, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（FragmentB, FragmentC），简化后续模板代码。

### Lines 1029-1033
```cpp
1029:   static bool const a_row_major = platform::is_same< LayoutA, layout::RowMajor>::value;
1030:   static bool const b_column_major = platform::is_same< LayoutB, layout::ColumnMajor>::value;
1031:   static bool const c_row_major = platform::is_same< LayoutC, layout::RowMajor>::value;
1032:   static bool const c_column_major = platform::is_same< LayoutC, layout::ColumnMajor>::value;
1033: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1034-1037
```cpp
1034:   static bool const m_mod2 = !(Shape::kM % 2);
1035:   static bool const n_mod2 = !(Shape::kN % 2);
1036:   static bool const k_mod2 = !(Shape::kK % 2);
1037: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1038-1043
```cpp
1038:   // HFMA based MMA optimizations are of 2 types :
1039:   // 1. Inner product 
1040:   // 2. Outer product
1041:   // It is chosen based on LayoutC (for outer product gemm) or
1042:   // Using LayoutA and LayoutB or shape=1x1x2K (for inner product gemms)
1043:   // If all fails, we choose the generic MMA
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1044-1047
```cpp
1044:   static bool const use_outer_prod = (c_column_major && m_mod2) || (c_row_major && n_mod2);
1045:   static bool const use_inner_prod = (a_row_major && b_column_major && k_mod2) || (Shape::kM==1 && Shape::kN==1 && k_mod2);
1046:   static bool const use_optimized =  (use_outer_prod || use_inner_prod);
1047: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1048-1052
```cpp
1048:   using ArchMmaOperator = typename platform::conditional< use_optimized, 
1049:     detail::Mma_HFMA2<Shape, LayoutA, LayoutB, LayoutC, use_outer_prod>, 
1050:     MmaGeneric <Shape, ElementA, LayoutA, ElementB, LayoutB, ElementC, LayoutC, Operator> 
1051:   >::type;
1052: 
```
**EN:** Introduces local type aliases (ArchMmaOperator) to simplify downstream template code.
**CN:** 引入本地类型别名（ArchMmaOperator），简化后续模板代码。

### Lines 1053-1057
```cpp
1053:   //
1054:   // Methods
1055:   //
1056: 
1057:   /// Computes a matrix product D = A * B + C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1058-1068
```cpp
1058:   CUTLASS_HOST_DEVICE
1059:   void operator()(
1060:     FragmentC & D,
1061:     FragmentA const & A,
1062:     FragmentB const & B,
1063:     FragmentC const & C) {
1064: 
1065:     ArchMmaOperator mma;
1066: 
1067:     mma(D, A, B, C);
1068: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 1069-1076
```cpp
1069:   }
1070: };
1071: 
1072: /////////////////////////////////////////////////////////////////////////////////////////////////
1073: 
1074: namespace detail {
1075: 
1076:   /// Determines whether to enable thread::Gemm<> specializations compatible with SM50
```
**EN:** Enters namespace scope (detail) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（detail），组织 GEMM 抽象层。

### Lines 1077-1079
```cpp
1077:   template <
1078:     typename LayoutA,
1079:     /// Layout of B matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 1080-1082
```cpp
1080:     typename LayoutB>
1081:   struct EnableMma_Crow_SM60 {
1082: 
```
**EN:** Defines EnableMma_Crow_SM60, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 EnableMma_Crow_SM60，用于封装策略、存储或算法行为的辅助类型。

### Lines 1083-1092
```cpp
1083:     static bool const kIsConventionalLayout =
1084:       (platform::is_same<LayoutA, layout::RowMajor>::value ||
1085:         platform::is_same<LayoutA, layout::ColumnMajor>::value) &&
1086:       (platform::is_same<LayoutB, layout::RowMajor>::value ||
1087:         platform::is_same<LayoutB, layout::ColumnMajor>::value);
1088: 
1089:     static bool const value = kIsConventionalLayout;
1090:   };
1091: };
1092: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1093-1097
```cpp
1093: /////////////////////////////////////////////////////////////////////////////////////////////////
1094: 
1095: /// Computes matrix product when C is row-major
1096: template <
1097:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1098-1109
```cpp
1098:   typename Shape_,
1099:   typename LayoutA_,
1100:   typename LayoutB_
1101: >
1102: struct Mma<
1103:   Shape_,
1104:   half_t,
1105:   LayoutA_,
1106:   half_t,
1107:   LayoutB_,
1108:   half_t,
1109:   layout::RowMajor,
```
**EN:** Defines Mma, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Mma，用于封装策略、存储或算法行为的辅助类型。

### Lines 1110-1115
```cpp
1110:   arch::OpMultiplyAdd,
1111:   typename platform::enable_if<detail::EnableMma_Crow_SM60<
1112:     LayoutA_,
1113:     LayoutB_
1114:     >::value>::type>{
1115: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1116-1124
```cpp
1116:   using Shape = Shape_;
1117:   using ElementA = half_t;
1118:   using LayoutA = LayoutA_;
1119:   using ElementB = half_t;
1120:   using LayoutB = LayoutB_;
1121:   using ElementC = half_t;
1122:   using LayoutC = layout::RowMajor;
1123:   using Operator = arch::OpMultiplyAdd;
1124: 
```
**EN:** Introduces local type aliases (Shape, ElementA, LayoutA, ElementB) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, ElementA, LayoutA, ElementB），简化后续模板代码。

### Lines 1125-1135
```cpp
1125:   using TransposeMma = Mma<
1126:     GemmShapeTranspose<Shape>,
1127:     half_t,
1128:     typename layout::LayoutTranspose<LayoutB>::type,
1129:     half_t,
1130:     typename layout::LayoutTranspose<LayoutA>::type,
1131:     half_t,
1132:     layout::ColumnMajor,
1133:     arch::OpMultiplyAdd,
1134:     bool>;
1135: 
```
**EN:** Introduces local type aliases (TransposeMma) to simplify downstream template code.
**CN:** 引入本地类型别名（TransposeMma），简化后续模板代码。

### Lines 1136-1141
```cpp
1136:   using FragmentA = Array<ElementA, Shape::kMK>;
1137:   using FragmentB = Array<ElementB, Shape::kKN>;
1138:   using FragmentC = Array<ElementC, Shape::kMN>;
1139: 
1140:   using ArchMmaOperator = typename TransposeMma::ArchMmaOperator;
1141: 
```
**EN:** Introduces local type aliases (FragmentA, FragmentB, FragmentC, ArchMmaOperator) to simplify downstream template code.
**CN:** 引入本地类型别名（FragmentA, FragmentB, FragmentC, ArchMmaOperator），简化后续模板代码。

### Lines 1142-1152
```cpp
1142:   CUTLASS_HOST_DEVICE
1143:   void operator()(
1144:     FragmentC & D,
1145:     FragmentA const & A,
1146:     FragmentB const & B,
1147:     FragmentC const & C) {
1148: 
1149:     TransposeMma mma;
1150: 
1151:     mma(D, B, A, C);
1152:   }
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 1153-1161
```cpp
1153: };
1154: 
1155: /////////////////////////////////////////////////////////////////////////////////////////////////
1156: 
1157: } // namespace thread
1158: } // namespace gemm
1159: } // namespace cutlass
1160: 
1161: /////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** Closes the current type or namespace scope.
**CN:** 结束当前类型或命名空间作用域。

## Key Concepts / 关键概念

- **EN:** Per-thread fragment math helpers  
  **CN:** 线程级 fragment 数学辅助
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `Mma_HFMA2`, `Mma`, `EnableMma_Crow_SM60`, `operator`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
