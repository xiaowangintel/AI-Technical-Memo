# default_multistage_mma_complex_core_sm80.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/threadblock/default_multistage_mma_complex_core_sm80.h`
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

### Lines 25-35
```cpp
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
31: /*! \file
32:     \brief Defines basic properties needed by CTA-level GEMMs assuming
33:    expectations about data layout of the global memory fragments, data types,
34:    and internal tile sizes.
35: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 36-46
```cpp
36:       Partial specializations for threadblock::Mma operations targeting TensorOp
37:    instructions.
38: */
39: 
40: #pragma once
41: 
42: #include "cutlass/array.h"
43: #include "cutlass/cutlass.h"
44: 
45: #include "cutlass/layout/tensor_op_multiplicand_sm75.h"
46: #include "cutlass/layout/tensor_op_multiplicand_sm80.h"
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 47-47
```cpp
47: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 48-54
```cpp
48: #include "cutlass/gemm/warp/mma_simt_policy.h"
49: #include "cutlass/gemm/warp/mma_simt.h"
50: #include "cutlass/gemm/warp/default_mma_complex_tensor_op.h"
51: #include "cutlass/gemm/warp/mma_tensor_op_tile_iterator_sm80.h"
52: 
53: #include "cutlass/gemm/threadblock/default_multistage_mma_complex_core.h"
54: 
```
**EN:** Pulls in required dependencies such as warp components, threadblock components.
**CN:** 引入所需依赖，例如 warp 组件、线程块组件。

### Lines 55-64
```cpp
55: #include "cutlass/matrix_shape.h"
56: #include "cutlass/numeric_types.h"
57: #include "cutlass/transform/pitch_linear_thread_map.h"
58: #include "cutlass/transform/threadblock/regular_tile_access_iterator_tensor_op.h"
59: #include "cutlass/transform/threadblock/regular_tile_access_iterator_tensor_op_sm80.h"
60: #include "cutlass/transform/threadblock/regular_tile_access_iterator_pitch_linear.h"
61: #include "cutlass/gemm/threadblock/mma_multistage.h"
62: 
63: ////////////////////////////////////////////////////////////////////////////////
64: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, numeric types/converters, threadblock components.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、数值类型/转换器、线程块组件。

### Lines 65-70
```cpp
65: namespace cutlass {
66: namespace gemm {
67: namespace threadblock {
68: 
69: ////////////////////////////////////////////////////////////////////////////////
70: 
```
**EN:** Enters namespace scope (cutlass::gemm::threadblock) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass::gemm::threadblock），组织 GEMM 抽象层。

### Lines 71-77
```cpp
71: /// Partial specialization for complex double-precision
72: ///
73: ///   A: column-major
74: ///   B: row-major
75: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
76: ///
77: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 78-88
```cpp
78: template <
79:     /// Shape of threadblock-scoped matrix multiply operator (concept:
80:     /// GemmShape)
81:     typename Shape_,
82:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
83:     typename WarpShape_,
84:     /// Shape of one matrix production operation (concept: GemmShape)
85:     typename InstructionShape_,
86:     /// Layout of accumulator
87:     typename LayoutC_,
88:     /// Number of stages
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 89-98
```cpp
89:     int Stages,
90:     /// Complex transformation on operand A
91:     ComplexTransform TransformA,
92:     /// Complex transformation on operand B
93:     ComplexTransform TransformB,
94:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
95:     typename Operator_,
96:     /// Cache operation of operand A
97:     cutlass::arch::CacheOperation::Kind CacheOpA,
98:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 99-109
```cpp
99:     cutlass::arch::CacheOperation::Kind CacheOpB>
100: struct DefaultMultistageMmaComplexCore<
101:     Shape_, WarpShape_, InstructionShape_, 
102:     complex<double>, layout::ColumnMajor,
103:     complex<double>, layout::RowMajor,
104:     complex<double>, LayoutC_, 
105:     arch::OpClassTensorOp,
106:     Stages,
107:     TransformA, TransformB,
108:     Operator_,
109:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 110-110
```cpp
110: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 111-122
```cpp
111:   using Shape = Shape_;
112:   using WarpShape = WarpShape_;
113:   using InstructionShape = InstructionShape_;
114:   using ElementA = complex<double>;
115:   using LayoutA = layout::ColumnMajor;
116:   using ElementB = complex<double>;
117:   using LayoutB = layout::RowMajor;
118:   using ElementC = complex<double>;
119:   using LayoutC = LayoutC_;
120:   static int const kStages = Stages;
121:   static ComplexTransform const kTransformA = TransformA;
122:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 123-127
```cpp
123:   using Operator = Operator_;
124:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
125:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
126: 
127:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 128-132
```cpp
128:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
129:                               Shape::kN / WarpShape::kN, 
130:                               Shape::kK / WarpShape::kK>;
131: 
132:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 133-136
```cpp
133:   static_assert(
134:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
135:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
136: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 137-146
```cpp
137:   static_assert(WarpCount::kCount > 1,
138:     "This specialization requires at least two warps.");
139: 
140:   /// Number of threads per warp
141:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
142: 
143:   /// Number of threads total
144:   static int const kThreads = WarpCount::kCount * kWarpSize;
145: 
146:   /// Size of a threadblock-scoped 128
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 147-148
```cpp
147:   static int const kAccessSizeInBits = 128;
148: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 149-156
```cpp
149:   //
150:   // Shared memory layouts
151:   //
152: 
153:   using SmemLayoutA = layout::ColumnMajorTensorOpMultiplicandCongruous128b;
154: 
155:   using SmemLayoutB = layout::RowMajorTensorOpMultiplicandCongruous128b;
156: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 157-161
```cpp
157:   //
158:   // Iterators to write to shared memory
159:   //
160: 
161:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 162-167
```cpp
162:   using IteratorThreadMapA = transform::PitchLinearWarpRakedThreadMap<
163:       layout::PitchLinearShape<Shape::kM, Shape::kK>, kThreads,
164:       layout::PitchLinearShape<8, 4>,
165:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
166: 
167:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 168-172
```cpp
168:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
169:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
170:       IteratorThreadMapA>;
171: 
172:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 173-178
```cpp
173:   using IteratorThreadMapB = transform::PitchLinearWarpRakedThreadMap<
174:       layout::PitchLinearShape<Shape::kN, Shape::kK>, kThreads,
175:       layout::PitchLinearShape<8, 4>,
176:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
177: 
178:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 179-182
```cpp
179:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
180:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
181:       IteratorThreadMapB>;
182: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 183-187
```cpp
183:   //
184:   // Warp-level matrix multiply operator
185:   //
186: 
187:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 188-196
```cpp
188:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
189:       WarpShape, InstructionShape, 
190:       ElementA, SmemLayoutA, 
191:       ElementB, SmemLayoutB,
192:       ElementC, LayoutC, 
193:       kTransformA, kTransformB,
194:       Operator>::Type;
195: 
196:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 197-201
```cpp
197:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
198:                                         MatrixShape<0, 0>, WarpCount::kK>;
199: };
200: 
201: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 202-208
```cpp
202: /// Partial specialization for complex double-precision
203: ///
204: ///   A: column-major
205: ///   B: row-major
206: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
207: ///
208: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 209-219
```cpp
209: template <
210:     /// Shape of threadblock-scoped matrix multiply operator (concept:
211:     /// GemmShape)
212:     typename Shape_,
213:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
214:     typename WarpShape_,
215:     /// Shape of one matrix production operation (concept: GemmShape)
216:     typename InstructionShape_,
217:     /// Layout of accumulator
218:     typename LayoutC_,
219:     /// Number of stages
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 220-229
```cpp
220:     int Stages,
221:     /// Complex transformation on operand A
222:     ComplexTransform TransformA,
223:     /// Complex transformation on operand B
224:     ComplexTransform TransformB,
225:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
226:     typename Operator_,
227:     /// Cache operation of operand A
228:     cutlass::arch::CacheOperation::Kind CacheOpA,
229:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 230-240
```cpp
230:     cutlass::arch::CacheOperation::Kind CacheOpB>
231: struct DefaultMultistageMmaComplexCore<
232:     Shape_, WarpShape_, InstructionShape_, 
233:     complex<double>, layout::ColumnMajor,
234:     complex<double>, layout::ColumnMajor,
235:     complex<double>, LayoutC_, 
236:     arch::OpClassTensorOp,
237:     Stages, 
238:     TransformA, TransformB,
239:     Operator_, 
240:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 241-241
```cpp
241: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 242-253
```cpp
242:   using Shape = Shape_;
243:   using WarpShape = WarpShape_;
244:   using InstructionShape = InstructionShape_;
245:   using ElementA = complex<double>;
246:   using LayoutA = layout::ColumnMajor;
247:   using ElementB = complex<double>;
248:   using LayoutB = layout::ColumnMajor;
249:   using ElementC = complex<double>;
250:   using LayoutC = LayoutC_;
251:   static int const kStages = Stages;
252:   using Operator = Operator_;
253:   static ComplexTransform const kTransformA = TransformA;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 254-258
```cpp
254:   static ComplexTransform const kTransformB = TransformB;
255:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
256:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
257: 
258:   /// Number of warps present
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 259-263
```cpp
259:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
260:                               Shape::kN / WarpShape::kN, 
261:                               Shape::kK / WarpShape::kK>;
262: 
263:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 264-267
```cpp
264:   static_assert(
265:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
266:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
267: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 268-277
```cpp
268:   static_assert(WarpCount::kCount > 1,
269:     "This specialization requires at least two warps.");
270: 
271:   /// Number of threads per warp
272:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
273: 
274:   /// Number of threads total
275:   static int const kThreads = WarpCount::kCount * kWarpSize;
276: 
277:   /// Size of a threadblock-scoped 128
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 278-279
```cpp
278:   static int const kAccessSizeInBits = 128;
279: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 280-283
```cpp
280:   //
281:   // Shared memory layouts
282:   //
283: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 284-286
```cpp
284:   using SmemLayoutA = layout::ColumnMajorTensorOpMultiplicandCongruous128b;
285:   using SmemLayoutB = layout::ColumnMajorTensorOpMultiplicandCrosswise128x4;
286: 
```
**EN:** Introduces local type aliases (SmemLayoutA, SmemLayoutB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemLayoutA, SmemLayoutB），简化后续模板代码。

### Lines 287-291
```cpp
287:   //
288:   // Iterators to write to shared memory
289:   //
290: 
291:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 292-297
```cpp
292:   using IteratorThreadMapA = transform::PitchLinearWarpRakedThreadMap<
293:       layout::PitchLinearShape<Shape::kM, Shape::kK>, kThreads,
294:       layout::PitchLinearShape<8, 4>,
295:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
296: 
297:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 298-302
```cpp
298:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
299:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
300:       IteratorThreadMapA>;
301: 
302:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 303-308
```cpp
303:   using IteratorThreadMapB = transform::PitchLinearWarpRakedThreadMap<
304:       layout::PitchLinearShape<Shape::kK, Shape::kN>, kThreads,
305:       layout::PitchLinearShape<8, 4>,
306:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
307: 
308:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 309-312
```cpp
309:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
310:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
311:       IteratorThreadMapB>;
312: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 313-317
```cpp
313:   //
314:   // Warp-level matrix multiply operator
315:   //
316: 
317:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 318-326
```cpp
318:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
319:       WarpShape, InstructionShape, 
320:       ElementA, SmemLayoutA, 
321:       ElementB, SmemLayoutB,
322:       ElementC, LayoutC, 
323:       kTransformA, kTransformB,
324:       Operator>::Type;
325: 
326:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 327-332
```cpp
327:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
328:                                         MatrixShape<0, 0>, WarpCount::kK>;
329: };
330: 
331: ////////////////////////////////////////////////////////////////////////////////
332: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 333-339
```cpp
333: /// Partial specialization for complex double-precision
334: ///
335: ///   A: row-major
336: ///   B: column-major
337: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
338: ///
339: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 340-350
```cpp
340: template <
341:     /// Shape of threadblock-scoped matrix multiply operator (concept:
342:     /// GemmShape)
343:     typename Shape_,
344:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
345:     typename WarpShape_,
346:     /// Shape of one matrix production operation (concept: GemmShape)
347:     typename InstructionShape_,
348:     /// Layout of accumulator
349:     typename LayoutC_,
350:     /// Number of stages
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 351-360
```cpp
351:     int Stages,
352:     /// Complex transformation on operand A
353:     ComplexTransform TransformA,
354:     /// Complex transformation on operand B
355:     ComplexTransform TransformB,
356:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
357:     typename Operator_,
358:     /// Cache operation of operand A
359:     cutlass::arch::CacheOperation::Kind CacheOpA,
360:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 361-371
```cpp
361:     cutlass::arch::CacheOperation::Kind CacheOpB>
362: struct DefaultMultistageMmaComplexCore<
363:     Shape_, WarpShape_, InstructionShape_, 
364:     complex<double>, layout::RowMajor,
365:     complex<double>, layout::ColumnMajor,
366:     complex<double>, LayoutC_, 
367:     arch::OpClassTensorOp,
368:     Stages,
369:     TransformA, TransformB,
370:     Operator_, 
371:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 372-372
```cpp
372: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 373-384
```cpp
373:   using Shape = Shape_;
374:   using WarpShape = WarpShape_;
375:   using InstructionShape = InstructionShape_;
376:   using ElementA = complex<double>;
377:   using LayoutA = layout::RowMajor;
378:   using ElementB = complex<double>;
379:   using LayoutB = layout::ColumnMajor;
380:   using ElementC = complex<double>;
381:   using LayoutC = LayoutC_;
382:   static int const kStages = Stages;
383:   static ComplexTransform const kTransformA = TransformA;
384:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 385-389
```cpp
385:   using Operator = Operator_;
386:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
387:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
388: 
389:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 390-394
```cpp
390:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
391:                               Shape::kN / WarpShape::kN, 
392:                               Shape::kK / WarpShape::kK>;
393: 
394:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 395-398
```cpp
395:   static_assert(
396:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
397:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
398:   
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 399-408
```cpp
399:   static_assert(WarpCount::kCount > 1,
400:     "This specialization requires at least two warps.");
401: 
402: 
403:   /// Number of threads per warp
404:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
405: 
406:   /// Number of threads total
407:   static int const kThreads = WarpCount::kCount * kWarpSize;
408: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 409-412
```cpp
409:   /// Size of a threadblock-scoped 128
410:   static int const kAccessSizeInBits = 128;
411: 
412: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 413-416
```cpp
413:   //
414:   // Shared memory layouts
415:   //
416: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 417-419
```cpp
417:   using SmemLayoutA = layout::RowMajorTensorOpMultiplicandCrosswise128x4;
418:   using SmemLayoutB = layout::ColumnMajorTensorOpMultiplicandCrosswise128x4;
419: 
```
**EN:** Introduces local type aliases (SmemLayoutA, SmemLayoutB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemLayoutA, SmemLayoutB），简化后续模板代码。

### Lines 420-424
```cpp
420:   //
421:   // Iterators to write to shared memory
422:   //
423: 
424:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 425-430
```cpp
425:   using IteratorThreadMapA = transform::PitchLinearWarpRakedThreadMap<
426:       layout::PitchLinearShape<Shape::kK, Shape::kM>, kThreads,
427:       layout::PitchLinearShape<8, 4>,
428:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
429: 
430:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 431-435
```cpp
431:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
432:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
433:       IteratorThreadMapA>;
434: 
435:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 436-441
```cpp
436:   using IteratorThreadMapB = transform::PitchLinearWarpRakedThreadMap<
437:       layout::PitchLinearShape<Shape::kK, Shape::kN>, kThreads,
438:       layout::PitchLinearShape<8, 4>,
439:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
440: 
441:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 442-445
```cpp
442:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
443:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
444:       IteratorThreadMapB>;
445: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 446-450
```cpp
446:   //
447:   // Warp-level matrix multiply operator
448:   //
449: 
450:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 451-459
```cpp
451:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
452:       WarpShape, InstructionShape, 
453:       ElementA, SmemLayoutA, 
454:       ElementB, SmemLayoutB,
455:       ElementC, LayoutC, 
456:       kTransformA, kTransformB,
457:       Operator>::Type;
458: 
459:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 460-464
```cpp
460:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
461:                                         MatrixShape<0, 0>, WarpCount::kK>;
462: };
463: 
464: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 465-471
```cpp
465: /// Partial specialization for complex double-precision
466: ///
467: ///   A: row-major
468: ///   B: row-major
469: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
470: ///
471: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 472-482
```cpp
472: template <
473:     /// Shape of threadblock-scoped matrix multiply operator (concept:
474:     /// GemmShape)
475:     typename Shape_,
476:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
477:     typename WarpShape_,
478:     /// Shape of one matrix production operation (concept: GemmShape)
479:     typename InstructionShape_,
480:     /// Layout of accumulator
481:     typename LayoutC_,
482:     /// Number of stages
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 483-492
```cpp
483:     int Stages,
484:     /// Complex transformation on operand A
485:     ComplexTransform TransformA,
486:     /// Complex transformation on operand B
487:     ComplexTransform TransformB,
488:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
489:     typename Operator_,    
490:     /// Cache operation of operand A
491:     cutlass::arch::CacheOperation::Kind CacheOpA,
492:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 493-503
```cpp
493:     cutlass::arch::CacheOperation::Kind CacheOpB>
494: struct DefaultMultistageMmaComplexCore<
495:     Shape_, WarpShape_, InstructionShape_, 
496:     complex<double>, layout::RowMajor,
497:     complex<double>, layout::RowMajor,
498:     complex<double>, LayoutC_, 
499:     arch::OpClassTensorOp,
500:     Stages, 
501:     TransformA, TransformB, 
502:     Operator_,
503:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 504-504
```cpp
504: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 505-516
```cpp
505:   using Shape = Shape_;
506:   using WarpShape = WarpShape_;
507:   using InstructionShape = InstructionShape_;
508:   using ElementA = complex<double>;
509:   using LayoutA = layout::RowMajor;
510:   using ElementB = complex<double>;
511:   using LayoutB = layout::RowMajor;
512:   using ElementC = complex<double>;
513:   using LayoutC = LayoutC_;
514:   static int const kStages = Stages;
515:   static ComplexTransform const kTransformA = TransformA;
516:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 517-521
```cpp
517:   using Operator = Operator_;
518:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
519:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
520: 
521:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 522-526
```cpp
522:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
523:                               Shape::kN / WarpShape::kN, 
524:                               Shape::kK / WarpShape::kK>;
525: 
526:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 527-530
```cpp
527:   static_assert(
528:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
529:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
530:   
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 531-540
```cpp
531:   static_assert(WarpCount::kCount > 1,
532:     "This specialization requires at least two warps.");
533: 
534: 
535:   /// Number of threads per warp
536:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
537: 
538:   /// Number of threads total
539:   static int const kThreads = WarpCount::kCount * kWarpSize;
540: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 541-544
```cpp
541:   /// Size of a threadblock-scoped 128
542:   static int const kAccessSizeInBits = 128;
543: 
544: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 545-548
```cpp
545:   //
546:   // Shared memory layouts
547:   //
548: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 549-551
```cpp
549:   using SmemLayoutA = layout::RowMajorTensorOpMultiplicandCrosswise128x4;
550:   using SmemLayoutB = layout::RowMajorTensorOpMultiplicandCongruous128b;
551: 
```
**EN:** Introduces local type aliases (SmemLayoutA, SmemLayoutB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemLayoutA, SmemLayoutB），简化后续模板代码。

### Lines 552-556
```cpp
552:   //
553:   // Iterators to write to shared memory
554:   //
555: 
556:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 557-562
```cpp
557:   using IteratorThreadMapA = transform::PitchLinearWarpRakedThreadMap<
558:       layout::PitchLinearShape<Shape::kK, Shape::kM>, kThreads,
559:       layout::PitchLinearShape<8, 4>,
560:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
561: 
562:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 563-567
```cpp
563:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
564:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
565:       IteratorThreadMapA>;
566: 
567:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 568-573
```cpp
568:   using IteratorThreadMapB = transform::PitchLinearWarpRakedThreadMap<
569:       layout::PitchLinearShape<Shape::kN, Shape::kK>, kThreads,
570:       layout::PitchLinearShape<8, 4>,
571:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
572: 
573:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 574-577
```cpp
574:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
575:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
576:       IteratorThreadMapB>;
577: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 578-582
```cpp
578:   //
579:   // Warp-level matrix multiply operator
580:   //
581: 
582:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 583-591
```cpp
583:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
584:       WarpShape, InstructionShape, 
585:       ElementA, SmemLayoutA, 
586:       ElementB, SmemLayoutB,
587:       ElementC, LayoutC, 
588:       kTransformA, kTransformB,
589:       Operator>::Type;
590: 
591:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 592-597
```cpp
592:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
593:                                         MatrixShape<0, 0>, WarpCount::kK>;
594: };
595: 
596: /////////////////////////////////////////////////////////////////////////////////////////////////
597: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 598-605
```cpp
598: /// Partial specialization for complex floating-point
599: ///
600: ///   A: column-major
601: ///   B: column-major
602: ///   Operator: arch::OpMultiplyAddComplex
603: ///   Math Instruction: mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32
604: ///
605: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 606-616
```cpp
606: template <
607:     /// Shape of threadblock-scoped matrix multiply operator (concept:
608:     /// GemmShape)
609:     typename Shape_,
610:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
611:     typename WarpShape_,
612:     /// Layout of accumulator
613:     typename LayoutC_,
614:     /// Number of stages
615:     int Stages,
616:     /// Complex transformation on operand A
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 617-624
```cpp
617:     ComplexTransform TransformA,
618:     /// Complex transformation on operand B
619:     ComplexTransform TransformB,
620:     /// Multiply-add operator (arch::OpMultiplyAddComplex)
621:     typename Operator_,
622:     /// Cache operation of operand A
623:     cutlass::arch::CacheOperation::Kind CacheOpA,
624:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 625-635
```cpp
625:     cutlass::arch::CacheOperation::Kind CacheOpB>
626: struct DefaultMultistageMmaComplexCore<
627:     Shape_, WarpShape_, GemmShape<16, 8, 8>, 
628:     complex<float>, layout::ColumnMajor,
629:     complex<float>, layout::ColumnMajor,
630:     complex<float>, LayoutC_, 
631:     arch::OpClassTensorOp,
632:     Stages,
633:     TransformA, TransformB,
634:     Operator_,
635:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 636-636
```cpp
636: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 637-648
```cpp
637:   using Shape = Shape_;
638:   using WarpShape = WarpShape_;
639:   using InstructionShape = GemmShape<16, 8, 8>;
640:   using ElementA = complex<float>;
641:   using LayoutA = layout::ColumnMajor;
642:   using ElementB = complex<float>;
643:   using LayoutB = layout::ColumnMajor;
644:   using ElementC = complex<float>;
645:   using LayoutC = LayoutC_;
646:   static int const kStages = Stages;
647:   static ComplexTransform const kTransformA = TransformA;
648:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 649-653
```cpp
649:   using Operator = Operator_;
650:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
651:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
652: 
653:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 654-658
```cpp
654:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
655:                               Shape::kN / WarpShape::kN, 
656:                               Shape::kK / WarpShape::kK>;
657: 
658:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 659-662
```cpp
659:   static_assert(
660:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
661:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
662: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 663-672
```cpp
663:   static_assert(WarpCount::kCount > 1,
664:     "This specialization requires at least two warps.");
665: 
666:   /// Number of threads per warp
667:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
668: 
669:   /// Number of threads total
670:   static int const kThreads = WarpCount::kCount * kWarpSize;
671: 
672:   /// Size of a threadblock-scoped
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 673-674
```cpp
673:   static int const kAccessSizeInBits = 64;
674: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 675-682
```cpp
675:   //
676:   // Shared memory layouts
677:   //
678: 
679:   using SmemLayoutA = layout::ColumnMajorTensorOpMultiplicandCongruous64b;
680: 
681:   using SmemLayoutB = layout::ColumnMajorTensorOpMultiplicand64bCrosswise;
682: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 683-687
```cpp
683:   //
684:   // Iterators to write to shared memory
685:   //
686: 
687:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 688-693
```cpp
688:   using IteratorThreadMapA = transform::PitchLinearWarpStripedThreadMap<
689:       layout::PitchLinearShape<Shape::kM, Shape::kK>, kThreads,
690:       layout::PitchLinearShape<16, 2>,
691:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
692: 
693:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 694-698
```cpp
694:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
695:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
696:       IteratorThreadMapA>;
697: 
698:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 699-704
```cpp
699:   using IteratorThreadMapB = transform::PitchLinearWarpRakedThreadMap<
700:       layout::PitchLinearShape<Shape::kK, Shape::kN>, kThreads,
701:       layout::PitchLinearShape<16, 2>,
702:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
703: 
704:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 705-708
```cpp
705:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
706:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
707:       IteratorThreadMapB>;
708: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 709-713
```cpp
709:   //
710:   // Warp-level matrix multiply operator
711:   //
712: 
713:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 714-722
```cpp
714:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
715:       WarpShape, InstructionShape, 
716:       ElementA, SmemLayoutA, 
717:       ElementB, SmemLayoutB,
718:       ElementC, LayoutC, 
719:       kTransformA, kTransformB,
720:       Operator>::Type;
721: 
722:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 723-727
```cpp
723:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
724:                                         MatrixShape<0, 0>, WarpCount::kK>;
725: };
726: 
727: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 728-735
```cpp
728: /// Partial specialization for complex floating-point
729: ///
730: ///   A: column-major
731: ///   B: row-major
732: ///   Operator: arch::OpMultiplyAddComplex
733: ///   Math Instruction: mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32
734: ///
735: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 736-746
```cpp
736: template <
737:     /// Shape of threadblock-scoped matrix multiply operator (concept:
738:     /// GemmShape)
739:     typename Shape_,
740:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
741:     typename WarpShape_,
742:     /// Layout of accumulator
743:     typename LayoutC_,
744:     /// Number of stages
745:     int Stages,
746:     /// Complex transformation on operand A
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 747-754
```cpp
747:     ComplexTransform TransformA,
748:     /// Complex transformation on operand B
749:     ComplexTransform TransformB,
750:     /// Multiply-add operator (arch::OpMultiplyAddComplex)
751:     typename Operator_,
752:     /// Cache operation of operand A
753:     cutlass::arch::CacheOperation::Kind CacheOpA,
754:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 755-765
```cpp
755:     cutlass::arch::CacheOperation::Kind CacheOpB>
756: struct DefaultMultistageMmaComplexCore<
757:     Shape_, WarpShape_, GemmShape<16, 8, 8>, 
758:     complex<float>, layout::ColumnMajor,
759:     complex<float>, layout::RowMajor,
760:     complex<float>, LayoutC_, 
761:     arch::OpClassTensorOp,
762:     Stages,
763:     TransformA, TransformB,
764:     Operator_,
765:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 766-766
```cpp
766: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 767-778
```cpp
767:   using Shape = Shape_;
768:   using WarpShape = WarpShape_;
769:   using InstructionShape = GemmShape<16, 8, 8>;
770:   using ElementA = complex<float>;
771:   using LayoutA = layout::ColumnMajor;
772:   using ElementB = complex<float>;
773:   using LayoutB = layout::RowMajor;
774:   using ElementC = complex<float>;
775:   using LayoutC = LayoutC_;
776:   static int const kStages = Stages;
777:   static ComplexTransform const kTransformA = TransformA;
778:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 779-783
```cpp
779:   using Operator = Operator_;
780:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
781:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
782: 
783:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 784-788
```cpp
784:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
785:                               Shape::kN / WarpShape::kN, 
786:                               Shape::kK / WarpShape::kK>;
787: 
788:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 789-792
```cpp
789:   static_assert(
790:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
791:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
792: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 793-802
```cpp
793:   static_assert(WarpCount::kCount > 1,
794:     "This specialization requires at least two warps.");
795: 
796:   /// Number of threads per warp
797:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
798: 
799:   /// Number of threads total
800:   static int const kThreads = WarpCount::kCount * kWarpSize;
801: 
802:   /// Size of a threadblock-scoped
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 803-804
```cpp
803:   static int const kAccessSizeInBits = 64;
804: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 805-812
```cpp
805:   //
806:   // Shared memory layouts
807:   //
808: 
809:   using SmemLayoutA = layout::ColumnMajorTensorOpMultiplicandCongruous64b;
810: 
811:   using SmemLayoutB = layout::RowMajorTensorOpMultiplicandCongruous64b;
812: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 813-817
```cpp
813:   //
814:   // Iterators to write to shared memory
815:   //
816: 
817:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 818-823
```cpp
818:   using IteratorThreadMapA = transform::PitchLinearWarpStripedThreadMap<
819:       layout::PitchLinearShape<Shape::kM, Shape::kK>, kThreads,
820:       layout::PitchLinearShape<16, 2>,
821:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
822: 
823:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 824-828
```cpp
824:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
825:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
826:       IteratorThreadMapA>;
827: 
828:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 829-834
```cpp
829:   using IteratorThreadMapB = transform::PitchLinearWarpStripedThreadMap<
830:       layout::PitchLinearShape<Shape::kN, Shape::kK>, kThreads,
831:       layout::PitchLinearShape<16, 2>,
832:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
833: 
834:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 835-838
```cpp
835:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
836:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
837:       IteratorThreadMapB>;
838: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 839-843
```cpp
839:   //
840:   // Warp-level matrix multiply operator
841:   //
842: 
843:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 844-852
```cpp
844:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
845:       WarpShape, InstructionShape, 
846:       ElementA, SmemLayoutA, 
847:       ElementB, SmemLayoutB,
848:       ElementC, LayoutC, 
849:       kTransformA, kTransformB,
850:       Operator>::Type;
851: 
852:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 853-858
```cpp
853:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
854:                                         MatrixShape<0, 0>, WarpCount::kK>;
855: };
856: 
857: ////////////////////////////////////////////////////////////////////////////////
858: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 859-866
```cpp
859: /// Partial specialization for complex floating-point
860: ///
861: ///   A: row-major
862: ///   B: column-major
863: ///   Operator: arch::OpMultiplyAddComplex
864: ///   Math Instruction: mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32
865: ///
866: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 867-877
```cpp
867: template <
868:     /// Shape of threadblock-scoped matrix multiply operator (concept:
869:     /// GemmShape)
870:     typename Shape_,
871:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
872:     typename WarpShape_,
873:     /// Layout of accumulator
874:     typename LayoutC_,
875:     /// Number of stages
876:     int Stages,
877:     /// Complex transformation on operand A
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 878-885
```cpp
878:     ComplexTransform TransformA,
879:     /// Complex transformation on operand B
880:     ComplexTransform TransformB,
881:     /// Multiply-add operator (arch::OpMultiplyAddComplex)
882:     typename Operator_,
883:     /// Cache operation of operand A
884:     cutlass::arch::CacheOperation::Kind CacheOpA,
885:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 886-896
```cpp
886:     cutlass::arch::CacheOperation::Kind CacheOpB>
887: struct DefaultMultistageMmaComplexCore<
888:     Shape_, WarpShape_, GemmShape<16, 8, 8>, 
889:     complex<float>, layout::RowMajor,
890:     complex<float>, layout::ColumnMajor,
891:     complex<float>, LayoutC_, 
892:     arch::OpClassTensorOp,
893:     Stages,
894:     TransformA, TransformB,
895:     Operator_,
896:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 897-897
```cpp
897: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 898-909
```cpp
898:   using Shape = Shape_;
899:   using WarpShape = WarpShape_;
900:   using InstructionShape = GemmShape<16, 8, 8>;
901:   using ElementA = complex<float>;
902:   using LayoutA = layout::RowMajor;
903:   using ElementB = complex<float>;
904:   using LayoutB = layout::ColumnMajor;
905:   using ElementC = complex<float>;
906:   using LayoutC = LayoutC_;
907:   static int const kStages = Stages;
908:   static ComplexTransform const kTransformA = TransformA;
909:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 910-914
```cpp
910:   using Operator = Operator_;
911:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
912:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
913: 
914:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 915-919
```cpp
915:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
916:                               Shape::kN / WarpShape::kN, 
917:                               Shape::kK / WarpShape::kK>;
918: 
919:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 920-923
```cpp
920:   static_assert(
921:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
922:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
923: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 924-933
```cpp
924:   static_assert(WarpCount::kCount > 1,
925:     "This specialization requires at least two warps.");
926: 
927:   /// Number of threads per warp
928:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
929: 
930:   /// Number of threads total
931:   static int const kThreads = WarpCount::kCount * kWarpSize;
932: 
933:   /// Size of a threadblock-scoped
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 934-935
```cpp
934:   static int const kAccessSizeInBits = 64;
935: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 936-943
```cpp
936:   //
937:   // Shared memory layouts
938:   //
939: 
940:   using SmemLayoutA = layout::RowMajorTensorOpMultiplicand64bCrosswise;
941: 
942:   using SmemLayoutB = layout::ColumnMajorTensorOpMultiplicand64bCrosswise;
943: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 944-948
```cpp
944:   //
945:   // Iterators to write to shared memory
946:   //
947: 
948:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 949-954
```cpp
949:   using IteratorThreadMapA = transform::PitchLinearWarpRakedThreadMap<
950:       layout::PitchLinearShape<Shape::kK, Shape::kM>, kThreads,
951:       layout::PitchLinearShape<16, 2>,
952:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
953: 
954:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 955-959
```cpp
955:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
956:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
957:       IteratorThreadMapA>;
958: 
959:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 960-965
```cpp
960:   using IteratorThreadMapB = transform::PitchLinearWarpRakedThreadMap<
961:       layout::PitchLinearShape<Shape::kK, Shape::kN>, kThreads,
962:       layout::PitchLinearShape<16, 2>,
963:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
964: 
965:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 966-969
```cpp
966:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
967:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
968:       IteratorThreadMapB>;
969:       
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 970-974
```cpp
970:   //
971:   // Warp-level matrix multiply operator
972:   //
973: 
974:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 975-983
```cpp
975:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
976:       WarpShape, InstructionShape, 
977:       ElementA, SmemLayoutA, 
978:       ElementB, SmemLayoutB,
979:       ElementC, LayoutC, 
980:       kTransformA, kTransformB,
981:       Operator>::Type;
982: 
983:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 984-989
```cpp
984:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
985:                                         MatrixShape<0, 0>, WarpCount::kK>;
986: };
987: 
988: ////////////////////////////////////////////////////////////////////////////////
989: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 990-997
```cpp
990: /// Partial specialization for complex floating-point
991: ///
992: ///   A: row-major
993: ///   B: row-major
994: ///   Operator: arch::OpMultiplyAddComplex
995: ///   Math Instruction: mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32
996: ///
997: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 998-1008
```cpp
998: template <
999:     /// Shape of threadblock-scoped matrix multiply operator (concept:
1000:     /// GemmShape)
1001:     typename Shape_,
1002:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
1003:     typename WarpShape_,
1004:     /// Layout of accumulator
1005:     typename LayoutC_,
1006:     /// Number of stages
1007:     int Stages,
1008:     /// Complex transformation on operand A
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1009-1016
```cpp
1009:     ComplexTransform TransformA,
1010:     /// Complex transformation on operand B
1011:     ComplexTransform TransformB,
1012:     /// Multiply-add operator (arch::OpMultiplyAddComplex)
1013:     typename Operator_,
1014:     /// Cache operation of operand A
1015:     cutlass::arch::CacheOperation::Kind CacheOpA,
1016:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1017-1027
```cpp
1017:     cutlass::arch::CacheOperation::Kind CacheOpB>
1018: struct DefaultMultistageMmaComplexCore<
1019:     Shape_, WarpShape_, GemmShape<16, 8, 8>, 
1020:     complex<float>, layout::RowMajor,
1021:     complex<float>, layout::RowMajor,
1022:     complex<float>, LayoutC_, 
1023:     arch::OpClassTensorOp,
1024:     Stages,
1025:     TransformA, TransformB,
1026:     Operator_,
1027:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 1028-1028
```cpp
1028: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 1029-1040
```cpp
1029:   using Shape = Shape_;
1030:   using WarpShape = WarpShape_;
1031:   using InstructionShape = GemmShape<16, 8, 8>;
1032:   using ElementA = complex<float>;
1033:   using LayoutA = layout::RowMajor;
1034:   using ElementB = complex<float>;
1035:   using LayoutB = layout::RowMajor;
1036:   using ElementC = complex<float>;
1037:   using LayoutC = LayoutC_;
1038:   static int const kStages = Stages;
1039:   static ComplexTransform const kTransformA = TransformA;
1040:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 1041-1045
```cpp
1041:   using Operator = Operator_;
1042:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
1043:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
1044: 
1045:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 1046-1050
```cpp
1046:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
1047:                               Shape::kN / WarpShape::kN, 
1048:                               Shape::kK / WarpShape::kK>;
1049: 
1050:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 1051-1054
```cpp
1051:   static_assert(
1052:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
1053:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
1054: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1055-1064
```cpp
1055:   static_assert(WarpCount::kCount > 1,
1056:     "This specialization requires at least two warps.");
1057: 
1058:   /// Number of threads per warp
1059:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
1060: 
1061:   /// Number of threads total
1062:   static int const kThreads = WarpCount::kCount * kWarpSize;
1063: 
1064:   /// Size of a threadblock-scoped
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1065-1066
```cpp
1065:   static int const kAccessSizeInBits = 64;
1066: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1067-1074
```cpp
1067:   //
1068:   // Shared memory layouts
1069:   //
1070: 
1071:   using SmemLayoutA = layout::RowMajorTensorOpMultiplicand64bCrosswise;
1072: 
1073:   using SmemLayoutB = layout::RowMajorTensorOpMultiplicandCongruous64b;
1074: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 1075-1079
```cpp
1075:   //
1076:   // Iterators to write to shared memory
1077:   //
1078: 
1079:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 1080-1085
```cpp
1080:   using IteratorThreadMapA = transform::PitchLinearWarpRakedThreadMap<
1081:       layout::PitchLinearShape<Shape::kK, Shape::kM>, kThreads,
1082:       layout::PitchLinearShape<16, 2>,
1083:       kAccessSizeInBits / sizeof_bits<ElementA>::value>;
1084: 
1085:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 1086-1090
```cpp
1086:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
1087:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 1,
1088:       IteratorThreadMapA>;
1089: 
1090:   /// ThreadMap of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 1091-1096
```cpp
1091:   using IteratorThreadMapB = transform::PitchLinearWarpStripedThreadMap<
1092:       layout::PitchLinearShape<Shape::kN, Shape::kK>, kThreads,
1093:       layout::PitchLinearShape<16, 2>,
1094:       kAccessSizeInBits / sizeof_bits<ElementB>::value>;
1095: 
1096:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 1097-1100
```cpp
1097:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
1098:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 0,
1099:       IteratorThreadMapB>;
1100:       
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 1101-1105
```cpp
1101:   //
1102:   // Warp-level matrix multiply operator
1103:   //
1104: 
1105:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 1106-1114
```cpp
1106:   using MmaTensorOp = typename cutlass::gemm::warp::DefaultMmaComplexTensorOp<
1107:       WarpShape, InstructionShape, 
1108:       ElementA, SmemLayoutA, 
1109:       ElementB, SmemLayoutB,
1110:       ElementC, LayoutC, 
1111:       kTransformA, kTransformB,
1112:       Operator>::Type;
1113: 
1114:   /// Policy used to define MmaPipelined
```
**EN:** Introduces local type aliases (MmaTensorOp) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaTensorOp），简化后续模板代码。

### Lines 1115-1120
```cpp
1115:   using MmaPolicy = MmaPolicy<MmaTensorOp, MatrixShape<0, 0>,
1116:                                         MatrixShape<0, 0>, WarpCount::kK>;
1117: };
1118: 
1119: ////////////////////////////////////////////////////////////////////////////////
1120: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 1121-1127
```cpp
1121: /// Partial specialization for complex SIMT operation
1122: ///
1123: ///   A: column-major
1124: ///   B: column-major
1125: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
1126: ///
1127: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1128-1132
```cpp
1128: template <
1129:     /// Shape of threadblock-scoped matrix multiply operator (concept:
1130:     /// GemmShape)
1131:     typename Shape_,
1132:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 1133-1143
```cpp
1133:     typename WarpShape_,
1134:     typename RealA,
1135:     typename RealB,
1136:     typename RealC,
1137:     /// Layout of accumulator
1138:     typename LayoutC_,
1139:     /// Number of stages
1140:     int Stages,
1141:     /// Complex transformation on operand A
1142:     ComplexTransform TransformA,
1143:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1144-1149
```cpp
1144:     ComplexTransform TransformB,
1145:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
1146:     typename Operator_,
1147:     /// Cache operation of operand A
1148:     cutlass::arch::CacheOperation::Kind CacheOpA,
1149:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1150-1160
```cpp
1150:     cutlass::arch::CacheOperation::Kind CacheOpB>
1151: struct DefaultMultistageMmaComplexCore<
1152:     Shape_, WarpShape_, GemmShape<1, 1, 1>, 
1153:     complex<RealA>, layout::ColumnMajor,
1154:     complex<RealB>, layout::ColumnMajor,
1155:     complex<RealC>, LayoutC_, 
1156:     arch::OpClassSimt,
1157:     Stages,
1158:     TransformA, TransformB,
1159:     Operator_,
1160:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 1161-1161
```cpp
1161: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 1162-1173
```cpp
1162:   using Shape = Shape_;
1163:   using WarpShape = WarpShape_;
1164:   using InstructionShape = GemmShape<1, 1, 1>;
1165:   using ElementA = complex<RealA>;
1166:   using LayoutA = layout::ColumnMajor;
1167:   using ElementB = complex<RealB>;
1168:   using LayoutB = layout::ColumnMajor;
1169:   using ElementC = complex<RealC>;
1170:   using LayoutC = LayoutC_;
1171:   static int const kStages = Stages;
1172:   static ComplexTransform const kTransformA = TransformA;
1173:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 1174-1178
```cpp
1174:   using Operator = Operator_;
1175:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
1176:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
1177: 
1178:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 1179-1183
```cpp
1179:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
1180:                               Shape::kN / WarpShape::kN, 
1181:                               Shape::kK / WarpShape::kK>;
1182: 
1183:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 1184-1187
```cpp
1184:   static_assert(
1185:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
1186:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
1187: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1188-1197
```cpp
1188:   static_assert(WarpCount::kCount > 1,
1189:     "This specialization requires at least two warps.");
1190: 
1191:   /// Number of threads per warp
1192:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
1193: 
1194:   /// Number of threads total
1195:   static int const kThreads = WarpCount::kCount * kWarpSize;
1196: 
1197:   /// Size of access
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1198-1202
```cpp
1198:   static int const kAccessSizeInBits = sizeof_bits<ElementA>::value;
1199: 
1200:   /// No vectorized accesses
1201:   static int const kElementsPerAccess = 1;
1202: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1203-1210
```cpp
1203:   //
1204:   // Shared memory layouts
1205:   //
1206: 
1207:   using SmemLayoutA = layout::ColumnMajor;
1208: 
1209:   using SmemLayoutB = layout::RowMajor;
1210: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1211-1215
```cpp
1211:   //
1212:   // Iterators to write to shared memory
1213:   //
1214: 
1215:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 1216-1222
```cpp
1216:   using IteratorThreadMapA = transform::PitchLinearStripminedThreadMap<
1217:     layout::PitchLinearShape<Shape::kM, Shape::kK>,
1218:     kThreads,
1219:     kElementsPerAccess
1220:   >;
1221: 
1222:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 1223-1227
```cpp
1223:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
1224:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 0,
1225:       IteratorThreadMapA>;
1226: 
1227:   /// Policy of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 1228-1237
```cpp
1228:   using IteratorThreadMapB = transform::PitchLinearStripminedThreadMap<
1229:     layout::PitchLinearShape<Shape::kK, Shape::kN>,
1230:     kThreads,
1231:     kElementsPerAccess
1232:   >;
1233: 
1234:   /// Transpose the ThreadMap of iterator B 
1235:   using SmemThreadMapB = transform::TransposePitchLinearThreadMapSimt<IteratorThreadMapB>;
1236: 
1237:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB, SmemThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB, SmemThreadMapB），简化后续模板代码。

### Lines 1238-1241
```cpp
1238:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
1239:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 1,
1240:       SmemThreadMapB>;
1241: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 1242-1246
```cpp
1242:   //
1243:   // Warp-level matrix multiply operator
1244:   //
1245: 
1246:   // Define the warp-level op
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1247-1258
```cpp
1247:   static const int WarpNumThreadsM = 4;
1248:   static const int WarpNumThreadsN = 8;
1249:   static_assert(!(WarpShape::kM % WarpNumThreadsM) && !(WarpShape::kN % WarpNumThreadsN),
1250:       "WarpShape must be divisible by ThreadTile shape.");
1251:   static const int ThreadTileM = WarpShape::kM / WarpNumThreadsM;
1252:   static const int ThreadTileN = WarpShape::kN / WarpNumThreadsN;
1253:   static const int LaneLayout = ThreadTileM > 4 && ThreadTileN > 4 ? 2 : 1;
1254:   static const int numElementsA = 128 / sizeof_bits<ElementA>::value;
1255:   static const int numElementsB = 128 / sizeof_bits<ElementB>::value;
1256:   static const int LaneM = cutlass::const_min(numElementsA, ThreadTileM);
1257:   static const int LaneN = cutlass::const_min(numElementsB, ThreadTileN);
1258:   // these should have max of thread tile also
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1259-1268
```cpp
1259:   using LaneMmaShape = cutlass::gemm::GemmShape<
1260:       LaneM,
1261:       LaneN,
1262:       1>;
1263:   using Policy = cutlass::gemm::warp::MmaSimtPolicy<
1264:       cutlass::MatrixShape<WarpNumThreadsM, WarpNumThreadsN>,   // WarpShape
1265:       cutlass::layout::RowMajorInterleaved<LaneLayout>,         // LaneLayout
1266:       LaneMmaShape
1267:   >;
1268: 
```
**EN:** Introduces local type aliases (LaneMmaShape, Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（LaneMmaShape, Policy），简化后续模板代码。

### Lines 1269-1280
```cpp
1269:   using MmaWarpSimt = cutlass::gemm::warp::MmaSimt<
1270:     WarpShape,    /// Size of the Gemm problem - concept: gemm::GemmShape<> 128, 128, 8
1271:     ElementA,     /// Data type of A elements
1272:     SmemLayoutA,  /// Layout of A matrix (concept: MatrixLayout)
1273:     ElementB,     /// Data type of B elements
1274:     SmemLayoutB,  /// Layout of B matrix (concept: MatrixLayout)
1275:     ElementC,     /// Element type of C matrix
1276:     LayoutC,      /// Layout of C matrix (concept: MatrixLayout)
1277:     Policy,       /// Policy describing warp-level MmaTensorOp (concept: MmaTensorOp policy)
1278:     1,            /// 1 partition along K dimension
1279:     kTransformA,  /// Transform for A
1280:     kTransformB   /// Transform for B
```
**EN:** Introduces local type aliases (MmaWarpSimt) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaWarpSimt），简化后续模板代码。

### Lines 1281-1283
```cpp
1281:     >;            /// Used for partial specialization
1282: 
1283:   /// Policy used to define MmaPipelined
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1284-1290
```cpp
1284:   using MmaPolicy = MmaPolicy<
1285:     MmaWarpSimt,
1286:     MatrixShape<0, 0>,
1287:     MatrixShape<0, Shape::kK / 32>,
1288:     WarpCount::kK>;
1289: };
1290: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 1291-1297
```cpp
1291: /// Partial specialization for complex SIMT operation
1292: ///
1293: ///   A: column-major
1294: ///   B: row-major
1295: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
1296: ///
1297: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1298-1302
```cpp
1298: template <
1299:     /// Shape of threadblock-scoped matrix multiply operator (concept:
1300:     /// GemmShape)
1301:     typename Shape_,
1302:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 1303-1313
```cpp
1303:     typename WarpShape_,
1304:     typename RealA,
1305:     typename RealB,
1306:     typename RealC,
1307:     /// Layout of accumulator
1308:     typename LayoutC_,
1309:     /// Number of stages
1310:     int Stages,
1311:     /// Complex transformation on operand A
1312:     ComplexTransform TransformA,
1313:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1314-1319
```cpp
1314:     ComplexTransform TransformB,
1315:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
1316:     typename Operator_,
1317:     /// Cache operation of operand A
1318:     cutlass::arch::CacheOperation::Kind CacheOpA,
1319:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1320-1330
```cpp
1320:     cutlass::arch::CacheOperation::Kind CacheOpB>
1321: struct DefaultMultistageMmaComplexCore<
1322:     Shape_, WarpShape_, GemmShape<1, 1, 1>, 
1323:     complex<RealA>, layout::ColumnMajor,
1324:     complex<RealB>, layout::RowMajor,
1325:     complex<RealC>, LayoutC_, 
1326:     arch::OpClassSimt,
1327:     Stages,
1328:     TransformA, TransformB,
1329:     Operator_,
1330:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 1331-1331
```cpp
1331: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 1332-1343
```cpp
1332:   using Shape = Shape_;
1333:   using WarpShape = WarpShape_;
1334:   using InstructionShape = GemmShape<1, 1, 1>;
1335:   using ElementA = complex<RealA>;
1336:   using LayoutA = layout::ColumnMajor;
1337:   using ElementB = complex<RealB>;
1338:   using LayoutB = layout::RowMajor;
1339:   using ElementC = complex<RealC>;
1340:   using LayoutC = LayoutC_;
1341:   static int const kStages = Stages;
1342:   static ComplexTransform const kTransformA = TransformA;
1343:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 1344-1348
```cpp
1344:   using Operator = Operator_;
1345:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
1346:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
1347: 
1348:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 1349-1353
```cpp
1349:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
1350:                               Shape::kN / WarpShape::kN, 
1351:                               Shape::kK / WarpShape::kK>;
1352: 
1353:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 1354-1357
```cpp
1354:   static_assert(
1355:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
1356:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
1357: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1358-1367
```cpp
1358:   static_assert(WarpCount::kCount > 1,
1359:     "This specialization requires at least two warps.");
1360: 
1361:   /// Number of threads per warp
1362:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
1363: 
1364:   /// Number of threads total
1365:   static int const kThreads = WarpCount::kCount * kWarpSize;
1366: 
1367:   /// Size of access
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1368-1372
```cpp
1368:   static int const kAccessSizeInBits = sizeof_bits<ElementA>::value;
1369: 
1370:   /// No vectorized accesses
1371:   static int const kElementsPerAccess = 1;
1372: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1373-1380
```cpp
1373:   //
1374:   // Shared memory layouts
1375:   //
1376: 
1377:   using SmemLayoutA = layout::ColumnMajor;
1378: 
1379:   using SmemLayoutB = layout::RowMajor;
1380: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1381-1385
```cpp
1381:   //
1382:   // Iterators to write to shared memory
1383:   //
1384: 
1385:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 1386-1392
```cpp
1386:   using IteratorThreadMapA = transform::PitchLinearStripminedThreadMap<
1387:     layout::PitchLinearShape<Shape::kM, Shape::kK>,
1388:     kThreads,
1389:     kElementsPerAccess
1390:   >;
1391: 
1392:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA），简化后续模板代码。

### Lines 1393-1397
```cpp
1393:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
1394:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 0,
1395:       IteratorThreadMapA>;
1396: 
1397:   /// Policy of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 1398-1404
```cpp
1398:   using IteratorThreadMapB = transform::PitchLinearStripminedThreadMap<
1399:     layout::PitchLinearShape<Shape::kN, Shape::kK>,
1400:     kThreads,
1401:     kElementsPerAccess
1402:   >;
1403: 
1404:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 1405-1408
```cpp
1405:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
1406:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 1,
1407:       IteratorThreadMapB>;
1408: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 1409-1413
```cpp
1409:   //
1410:   // Warp-level matrix multiply operator
1411:   //
1412: 
1413:   // Define the warp-level op
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1414-1425
```cpp
1414:   static const int WarpNumThreadsM = 4;
1415:   static const int WarpNumThreadsN = 8;
1416:   static_assert(!(WarpShape::kM % WarpNumThreadsM) && !(WarpShape::kN % WarpNumThreadsN),
1417:       "WarpShape must be divisible by ThreadTile shape.");
1418:   static const int ThreadTileM = WarpShape::kM / WarpNumThreadsM;
1419:   static const int ThreadTileN = WarpShape::kN / WarpNumThreadsN;
1420:   static const int LaneLayout = ThreadTileM > 4 && ThreadTileN > 4 ? 2 : 1;
1421:   static const int numElementsA = 128 / sizeof_bits<ElementA>::value;
1422:   static const int numElementsB = 128 / sizeof_bits<ElementB>::value;
1423:   static const int LaneM = cutlass::const_min(numElementsA, ThreadTileM);
1424:   static const int LaneN = cutlass::const_min(numElementsB, ThreadTileN);
1425:   // these should have max of thread tile also
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1426-1435
```cpp
1426:   using LaneMmaShape = cutlass::gemm::GemmShape<
1427:       LaneM,
1428:       LaneN,
1429:       1>;
1430:   using Policy = cutlass::gemm::warp::MmaSimtPolicy<
1431:       cutlass::MatrixShape<WarpNumThreadsM, WarpNumThreadsN>,   // WarpShape
1432:       cutlass::layout::RowMajorInterleaved<LaneLayout>,         // LaneLayout
1433:       LaneMmaShape
1434:   >;
1435: 
```
**EN:** Introduces local type aliases (LaneMmaShape, Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（LaneMmaShape, Policy），简化后续模板代码。

### Lines 1436-1447
```cpp
1436:   using MmaWarpSimt = cutlass::gemm::warp::MmaSimt<
1437:     WarpShape,    /// Size of the Gemm problem - concept: gemm::GemmShape<> 128, 128, 8
1438:     ElementA,     /// Data type of A elements
1439:     SmemLayoutA,  /// Layout of A matrix (concept: MatrixLayout)
1440:     ElementB,     /// Data type of B elements
1441:     SmemLayoutB,  /// Layout of B matrix (concept: MatrixLayout)
1442:     ElementC,     /// Element type of C matrix
1443:     LayoutC,      /// Layout of C matrix (concept: MatrixLayout)
1444:     Policy,       /// Policy describing warp-level MmaTensorOp (concept: MmaTensorOp policy)
1445:     1,            /// 1 partition along K dimension
1446:     kTransformA,  /// Transform for A
1447:     kTransformB   /// Transform for B
```
**EN:** Introduces local type aliases (MmaWarpSimt) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaWarpSimt），简化后续模板代码。

### Lines 1448-1450
```cpp
1448:     >;            /// Used for partial specialization
1449: 
1450:   /// Policy used to define MmaPipelined
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1451-1457
```cpp
1451:   using MmaPolicy = MmaPolicy<
1452:     MmaWarpSimt,
1453:     MatrixShape<0, 0>,
1454:     MatrixShape<0, 0>,    // or Shape::kK / 32
1455:     WarpCount::kK>;
1456: };
1457: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 1458-1464
```cpp
1458: /// Partial specialization for complex SIMT operation
1459: ///
1460: ///   A: row-major
1461: ///   B: column-major
1462: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
1463: ///
1464: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1465-1469
```cpp
1465: template <
1466:     /// Shape of threadblock-scoped matrix multiply operator (concept:
1467:     /// GemmShape)
1468:     typename Shape_,
1469:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 1470-1480
```cpp
1470:     typename WarpShape_,
1471:     typename RealA,
1472:     typename RealB,
1473:     typename RealC,
1474:     /// Layout of accumulator
1475:     typename LayoutC_,
1476:     /// Number of stages
1477:     int Stages,
1478:     /// Complex transformation on operand A
1479:     ComplexTransform TransformA,
1480:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1481-1486
```cpp
1481:     ComplexTransform TransformB,
1482:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
1483:     typename Operator_,
1484:     /// Cache operation of operand A
1485:     cutlass::arch::CacheOperation::Kind CacheOpA,
1486:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1487-1497
```cpp
1487:     cutlass::arch::CacheOperation::Kind CacheOpB>
1488: struct DefaultMultistageMmaComplexCore<
1489:     Shape_, WarpShape_, GemmShape<1, 1, 1>, 
1490:     complex<RealA>, layout::RowMajor,
1491:     complex<RealB>, layout::ColumnMajor,
1492:     complex<RealC>, LayoutC_, 
1493:     arch::OpClassSimt,
1494:     Stages,
1495:     TransformA, TransformB,
1496:     Operator_,
1497:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 1498-1498
```cpp
1498: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 1499-1510
```cpp
1499:   using Shape = Shape_;
1500:   using WarpShape = WarpShape_;
1501:   using InstructionShape = GemmShape<1, 1, 1>;
1502:   using ElementA = complex<RealA>;
1503:   using LayoutA = layout::RowMajor;
1504:   using ElementB = complex<RealB>;
1505:   using LayoutB = layout::ColumnMajor;
1506:   using ElementC = complex<RealC>;
1507:   using LayoutC = LayoutC_;
1508:   static int const kStages = Stages;
1509:   static ComplexTransform const kTransformA = TransformA;
1510:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 1511-1515
```cpp
1511:   using Operator = Operator_;
1512:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
1513:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
1514: 
1515:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 1516-1520
```cpp
1516:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
1517:                               Shape::kN / WarpShape::kN, 
1518:                               Shape::kK / WarpShape::kK>;
1519: 
1520:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 1521-1524
```cpp
1521:   static_assert(
1522:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
1523:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
1524: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1525-1534
```cpp
1525:   static_assert(WarpCount::kCount > 1,
1526:     "This specialization requires at least two warps.");
1527: 
1528:   /// Number of threads per warp
1529:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
1530: 
1531:   /// Number of threads total
1532:   static int const kThreads = WarpCount::kCount * kWarpSize;
1533: 
1534:   /// Size of access
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1535-1539
```cpp
1535:   static int const kAccessSizeInBits = sizeof_bits<ElementA>::value;
1536: 
1537:   /// No vectorized accesses
1538:   static int const kElementsPerAccess = 1;
1539: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1540-1547
```cpp
1540:   //
1541:   // Shared memory layouts
1542:   //
1543: 
1544:   using SmemLayoutA = layout::ColumnMajor;
1545: 
1546:   using SmemLayoutB = layout::RowMajor;
1547: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1548-1552
```cpp
1548:   //
1549:   // Iterators to write to shared memory
1550:   //
1551: 
1552:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 1553-1562
```cpp
1553:   using IteratorThreadMapA = transform::PitchLinearStripminedThreadMap<
1554:     layout::PitchLinearShape<Shape::kK, Shape::kM>,
1555:     kThreads,
1556:     kElementsPerAccess
1557:   >;
1558: 
1559:   /// Transpose the ThreadMap of iterator A
1560:   using SmemThreadMapA = transform::TransposePitchLinearThreadMapSimt<IteratorThreadMapA>;
1561: 
1562:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA, SmemThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA, SmemThreadMapA），简化后续模板代码。

### Lines 1563-1567
```cpp
1563:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
1564:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 0,
1565:       SmemThreadMapA>;
1566: 
1567:   /// Policy of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 1568-1577
```cpp
1568:   using IteratorThreadMapB = transform::PitchLinearStripminedThreadMap<
1569:     layout::PitchLinearShape<Shape::kK, Shape::kN>,
1570:     kThreads,
1571:     kElementsPerAccess
1572:   >;
1573: 
1574:   /// Transpose the ThreadMap of iterator B 
1575:   using SmemThreadMapB = transform::TransposePitchLinearThreadMapSimt<IteratorThreadMapB>;
1576: 
1577:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB, SmemThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB, SmemThreadMapB），简化后续模板代码。

### Lines 1578-1581
```cpp
1578:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
1579:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 1,
1580:       SmemThreadMapB>;
1581: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 1582-1586
```cpp
1582:   //
1583:   // Warp-level matrix multiply operator
1584:   //
1585: 
1586:   // Define the warp-level op
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1587-1598
```cpp
1587:   static const int WarpNumThreadsM = 4;
1588:   static const int WarpNumThreadsN = 8;
1589:   static_assert(!(WarpShape::kM % WarpNumThreadsM) && !(WarpShape::kN % WarpNumThreadsN),
1590:       "WarpShape must be divisible by ThreadTile shape.");
1591:   static const int ThreadTileM = WarpShape::kM / WarpNumThreadsM;
1592:   static const int ThreadTileN = WarpShape::kN / WarpNumThreadsN;
1593:   static const int LaneLayout = ThreadTileM > 4 && ThreadTileN > 4 ? 2 : 1;
1594:   static const int numElementsA = 128 / sizeof_bits<ElementA>::value;
1595:   static const int numElementsB = 128 / sizeof_bits<ElementB>::value;
1596:   static const int LaneM = cutlass::const_min(numElementsA, ThreadTileM);
1597:   static const int LaneN = cutlass::const_min(numElementsB, ThreadTileN);
1598:   // these should have max of thread tile also
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1599-1608
```cpp
1599:   using LaneMmaShape = cutlass::gemm::GemmShape<
1600:       LaneM,
1601:       LaneN,
1602:       1>;
1603:   using Policy = cutlass::gemm::warp::MmaSimtPolicy<
1604:       cutlass::MatrixShape<WarpNumThreadsM, WarpNumThreadsN>,   // WarpShape
1605:       cutlass::layout::RowMajorInterleaved<LaneLayout>,         // LaneLayout
1606:       LaneMmaShape
1607:   >;
1608: 
```
**EN:** Introduces local type aliases (LaneMmaShape, Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（LaneMmaShape, Policy），简化后续模板代码。

### Lines 1609-1620
```cpp
1609:   using MmaWarpSimt = cutlass::gemm::warp::MmaSimt<
1610:     WarpShape,    /// Size of the Gemm problem - concept: gemm::GemmShape<> 128, 128, 8
1611:     ElementA,     /// Data type of A elements
1612:     SmemLayoutA,  /// Layout of A matrix (concept: MatrixLayout)
1613:     ElementB,     /// Data type of B elements
1614:     SmemLayoutB,  /// Layout of B matrix (concept: MatrixLayout)
1615:     ElementC,     /// Element type of C matrix
1616:     LayoutC,      /// Layout of C matrix (concept: MatrixLayout)
1617:     Policy,       /// Policy describing warp-level MmaTensorOp (concept: MmaTensorOp policy)
1618:     1,            /// 1 partition along K dimension
1619:     kTransformA,  /// Transform for A
1620:     kTransformB   /// Transform for B
```
**EN:** Introduces local type aliases (MmaWarpSimt) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaWarpSimt），简化后续模板代码。

### Lines 1621-1623
```cpp
1621:     >;            /// Used for partial specialization
1622: 
1623:   /// Policy used to define MmaPipelined
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1624-1630
```cpp
1624:   using MmaPolicy = MmaPolicy<
1625:     MmaWarpSimt,
1626:     MatrixShape<Shape::kK / 32, 0>,
1627:     MatrixShape<0, Shape::kK / 32>,
1628:     WarpCount::kK>;
1629: };
1630: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 1631-1637
```cpp
1631: /// Partial specialization for complex SIMT operation
1632: ///
1633: ///   A: row-major
1634: ///   B: row-major
1635: ///   Operator: arch::OpMultiplyAddComplex or arch::OpMultiplyGaussianComplex
1636: ///
1637: /// This uses the default warp-level operator given tile sizes
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1638-1642
```cpp
1638: template <
1639:     /// Shape of threadblock-scoped matrix multiply operator (concept:
1640:     /// GemmShape)
1641:     typename Shape_,
1642:     /// Shape of warp-level matrix multiply operator (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 1643-1653
```cpp
1643:     typename WarpShape_,
1644:     typename RealA,
1645:     typename RealB,
1646:     typename RealC,
1647:     /// Layout of accumulator
1648:     typename LayoutC_,
1649:     /// Number of stages
1650:     int Stages,
1651:     /// Complex transformation on operand A
1652:     ComplexTransform TransformA,
1653:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1654-1659
```cpp
1654:     ComplexTransform TransformB,
1655:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
1656:     typename Operator_,
1657:     /// Cache operation of operand A
1658:     cutlass::arch::CacheOperation::Kind CacheOpA,
1659:     /// Cache operation of operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 1660-1670
```cpp
1660:     cutlass::arch::CacheOperation::Kind CacheOpB>
1661: struct DefaultMultistageMmaComplexCore<
1662:     Shape_, WarpShape_, GemmShape<1, 1, 1>, 
1663:     complex<RealA>, layout::RowMajor,
1664:     complex<RealB>, layout::RowMajor,
1665:     complex<RealC>, LayoutC_, 
1666:     arch::OpClassSimt,
1667:     Stages,
1668:     TransformA, TransformB,
1669:     Operator_,
1670:     CacheOpA, CacheOpB> {
```
**EN:** Defines DefaultMultistageMmaComplexCore, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageMmaComplexCore，用于封装策略、存储或算法行为的辅助类型。

### Lines 1671-1671
```cpp
1671: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 1672-1683
```cpp
1672:   using Shape = Shape_;
1673:   using WarpShape = WarpShape_;
1674:   using InstructionShape = GemmShape<1, 1, 1>;
1675:   using ElementA = complex<RealA>;
1676:   using LayoutA = layout::RowMajor;
1677:   using ElementB = complex<RealB>;
1678:   using LayoutB = layout::RowMajor;
1679:   using ElementC = complex<RealC>;
1680:   using LayoutC = LayoutC_;
1681:   static int const kStages = Stages;
1682:   static ComplexTransform const kTransformA = TransformA;
1683:   static ComplexTransform const kTransformB = TransformB;
```
**EN:** Introduces local type aliases (Shape, WarpShape, InstructionShape, ElementA) to simplify downstream template code.
**CN:** 引入本地类型别名（Shape, WarpShape, InstructionShape, ElementA），简化后续模板代码。

### Lines 1684-1688
```cpp
1684:   using Operator = Operator_;
1685:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = cutlass::arch::CacheOperation::Always;
1686:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = cutlass::arch::CacheOperation::Always;
1687: 
1688:   /// Number of warps present
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 1689-1693
```cpp
1689:   using WarpCount = GemmShape<Shape::kM / WarpShape::kM,
1690:                               Shape::kN / WarpShape::kN, 
1691:                               Shape::kK / WarpShape::kK>;
1692: 
1693:   // Divisility requirements
```
**EN:** Introduces local type aliases (WarpCount) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpCount），简化后续模板代码。

### Lines 1694-1697
```cpp
1694:   static_assert(
1695:       !(Shape::kM % WarpShape::kM) && !(Shape::kN % WarpShape::kN),
1696:       "Threadblock-scoped GEMM should be divisible by warp-scoped GEMM size.");
1697: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1698-1707
```cpp
1698:   static_assert(WarpCount::kCount > 1,
1699:     "This specialization requires at least two warps.");
1700: 
1701:   /// Number of threads per warp
1702:   static int const kWarpSize = warp::WarpSize<arch::OpClassTensorOp>::value;
1703: 
1704:   /// Number of threads total
1705:   static int const kThreads = WarpCount::kCount * kWarpSize;
1706: 
1707:   /// Size of access
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1708-1712
```cpp
1708:   static int const kAccessSizeInBits = sizeof_bits<ElementA>::value;
1709: 
1710:   /// No vectorized accesses
1711:   static int const kElementsPerAccess = 1;
1712: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1713-1720
```cpp
1713:   //
1714:   // Shared memory layouts
1715:   //
1716: 
1717:   using SmemLayoutA = layout::ColumnMajor;
1718: 
1719:   using SmemLayoutB = layout::RowMajor;
1720: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1721-1725
```cpp
1721:   //
1722:   // Iterators to write to shared memory
1723:   //
1724: 
1725:   /// ThreadMap of iterator A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 1726-1735
```cpp
1726:   using IteratorThreadMapA = transform::PitchLinearStripminedThreadMap<
1727:     layout::PitchLinearShape<Shape::kK, Shape::kM>,
1728:     kThreads,
1729:     kElementsPerAccess
1730:   >;
1731: 
1732:   /// Transpose the ThreadMap of iterator A
1733:   using SmemThreadMapA = transform::TransposePitchLinearThreadMapSimt<IteratorThreadMapA>;
1734: 
1735:   /// Shared memory iterator to A operand
```
**EN:** Introduces local type aliases (IteratorThreadMapA, SmemThreadMapA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapA, SmemThreadMapA），简化后续模板代码。

### Lines 1736-1740
```cpp
1736:   using SmemIteratorA = transform::threadblock::RegularTileAccessIterator<
1737:       MatrixShape<Shape::kM, Shape::kK>, ElementA, SmemLayoutA, 0,
1738:       SmemThreadMapA>;
1739: 
1740:   /// Policy of iterator B
```
**EN:** Introduces local type aliases (SmemIteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA），简化后续模板代码。

### Lines 1741-1747
```cpp
1741:   using IteratorThreadMapB = transform::PitchLinearStripminedThreadMap<
1742:     layout::PitchLinearShape<Shape::kN, Shape::kK>,
1743:     kThreads,
1744:     kElementsPerAccess
1745:   >;
1746: 
1747:   /// Shared memory iterator to B operand
```
**EN:** Introduces local type aliases (IteratorThreadMapB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorThreadMapB），简化后续模板代码。

### Lines 1748-1751
```cpp
1748:   using SmemIteratorB = transform::threadblock::RegularTileAccessIterator<
1749:       MatrixShape<Shape::kK, Shape::kN>, ElementB, SmemLayoutB, 1,
1750:       IteratorThreadMapB>;
1751: 
```
**EN:** Introduces local type aliases (SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorB），简化后续模板代码。

### Lines 1752-1756
```cpp
1752:   //
1753:   // Warp-level matrix multiply operator
1754:   //
1755: 
1756:   // Define the warp-level op
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1757-1768
```cpp
1757:   static const int WarpNumThreadsM = 4;
1758:   static const int WarpNumThreadsN = 8;
1759:   static_assert(!(WarpShape::kM % WarpNumThreadsM) && !(WarpShape::kN % WarpNumThreadsN),
1760:       "WarpShape must be divisible by ThreadTile shape.");
1761:   static const int ThreadTileM = WarpShape::kM / WarpNumThreadsM;
1762:   static const int ThreadTileN = WarpShape::kN / WarpNumThreadsN;
1763:   static const int LaneLayout = ThreadTileM > 4 && ThreadTileN > 4 ? 2 : 1;
1764:   static const int numElementsA = 128 / sizeof_bits<ElementA>::value;
1765:   static const int numElementsB = 128 / sizeof_bits<ElementB>::value;
1766:   static const int LaneM = cutlass::const_min(numElementsA, ThreadTileM);
1767:   static const int LaneN = cutlass::const_min(numElementsB, ThreadTileN);
1768:   // these should have max of thread tile also
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1769-1778
```cpp
1769:   using LaneMmaShape = cutlass::gemm::GemmShape<
1770:       LaneM,
1771:       LaneN,
1772:       1>;
1773:   using Policy = cutlass::gemm::warp::MmaSimtPolicy<
1774:       cutlass::MatrixShape<WarpNumThreadsM, WarpNumThreadsN>,   // WarpShape
1775:       cutlass::layout::RowMajorInterleaved<LaneLayout>,         // LaneLayout
1776:       LaneMmaShape
1777:   >;
1778: 
```
**EN:** Introduces local type aliases (LaneMmaShape, Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（LaneMmaShape, Policy），简化后续模板代码。

### Lines 1779-1790
```cpp
1779:   using MmaWarpSimt = cutlass::gemm::warp::MmaSimt<
1780:     WarpShape,    /// Size of the Gemm problem - concept: gemm::GemmShape<> 128, 128, 8
1781:     ElementA,     /// Data type of A elements
1782:     SmemLayoutA,  /// Layout of A matrix (concept: MatrixLayout)
1783:     ElementB,     /// Data type of B elements
1784:     SmemLayoutB,  /// Layout of B matrix (concept: MatrixLayout)
1785:     ElementC,     /// Element type of C matrix
1786:     LayoutC,      /// Layout of C matrix (concept: MatrixLayout)
1787:     Policy,       /// Policy describing warp-level MmaTensorOp (concept: MmaTensorOp policy)
1788:     1,            /// 1 partition along K dimension
1789:     kTransformA,  /// Transform for A
1790:     kTransformB   /// Transform for B
```
**EN:** Introduces local type aliases (MmaWarpSimt) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaWarpSimt），简化后续模板代码。

### Lines 1791-1793
```cpp
1791:     >;            /// Used for partial specialization
1792: 
1793:   /// Policy used to define MmaPipelined
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1794-1803
```cpp
1794:   using MmaPolicy = MmaPolicy<
1795:     MmaWarpSimt,
1796:     MatrixShape<Shape::kK / 32, 0>,
1797:     MatrixShape<0, 0>,    // or Shape::kK / 32
1798:     WarpCount::kK>;
1799: };
1800: 
1801: ////////////////////////////////////////////////////////////////////////////////
1802: 
1803: 
```
**EN:** Introduces local type aliases (MmaPolicy) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaPolicy），简化后续模板代码。

### Lines 1804-1808
```cpp
1804: }  // namespace threadblock
1805: }  // namespace gemm
1806: }  // namespace cutlass
1807: 
1808: ////////////////////////////////////////////////////////////////////////////////
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
- **EN:** SIMT execution policy  
  **CN:** SIMT 执行策略
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线
- **EN:** Multi-stage mainloop buffering  
  **CN:** 多阶段主循环缓冲

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `DefaultMultistageMmaComplexCore`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
