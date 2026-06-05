# default_multistage_trmm_complex.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/threadblock/default_multistage_trmm_complex.h`
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

### Lines 25-31
```cpp
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
31: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 32-39
```cpp
32: /*! \file
33:     \brief Template for a multistage GEMM kernel. Does not compute batching or support split-K.
34: 
35:   
36: */
37: 
38: #pragma once
39: 
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 40-49
```cpp
40: #include "cutlass/blas3.h"
41: #include "cutlass/arch/arch.h"
42: #include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
43: #include "cutlass/numeric_types.h"
44: #include "cutlass/transform/threadblock/predicated_tile_iterator_triangular_matrix.h"
45: #include "cutlass/gemm/threadblock/mma_blas3_multistage.h"
46: 
47: ////////////////////////////////////////////////////////////////////////////////
48: 
49: namespace cutlass {
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, architecture intrinsics, threadblock components, numeric types/converters.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、架构内建/指令封装、线程块组件、数值类型/转换器。

### Lines 50-60
```cpp
50: namespace gemm {
51: namespace threadblock {
52: 
53: ////////////////////////////////////////////////////////////////////////////////
54: 
55: template <
56:     /// Element type for A matrix operand
57:     typename ElementA_,
58:     /// Layout type for A matrix operand
59:     typename LayoutA_,
60:     /// Element type for B matrix operand
```
**EN:** Enters namespace scope (gemm::threadblock) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（gemm::threadblock），组织 GEMM 抽象层。

### Lines 61-70
```cpp
61:     typename ElementB_,
62:     /// Layout type for B matrix operand
63:     typename LayoutB_,
64:     /// Side Mode for the kernel
65:     SideMode kSideMode,
66:     /// Fill Mode for the triangular matrix
67:     FillMode kFillMode,
68:     /// Diag Type for the triangular matrix
69:     DiagType kDiagType,
70:     /// Element type for internal accumulation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 71-80
```cpp
71:     typename ElementAccumulator_,
72:     /// Layout type for C and D matrix operands
73:     typename LayoutC_,
74:     /// Operator class tag
75:     typename OperatorClass_,
76:     /// Tag indicating architecture to tune for
77:     typename ArchTag_,
78:     /// Threadblock-level tile size (concept: GemmShape)
79:     typename ThreadblockShape_,
80:     /// Warp-level tile size (concept: GemmShape)
```
**EN:** Defines tag, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tag，用于封装策略、存储或算法行为的辅助类型。

### Lines 81-90
```cpp
81:     typename WarpShape_,
82:     /// Instruction-level tile size (concept: GemmShape)
83:     typename InstructionShape_,
84:     /// Number of stages used in the pipelined mainloop
85:     int Stages,
86:     /// Complex transformation on operand A
87:     ComplexTransform TransformA = ComplexTransform::kNone,
88:     /// Complex transformation on operand B
89:     ComplexTransform TransformB = ComplexTransform::kNone,
90:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 91-92
```cpp
91:     typename Operator = arch::OpMultiplyAddComplex,
92:     /// Blas3 computation mode
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 93-95
```cpp
93:     BlasMode BlasMode_ = BlasMode::kTriangular,
94:     /// Store the accumulators in row major or column major.  Row major is used
95:     /// when output layout is interleaved.
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 96-105
```cpp
96:     bool AccumulatorsInRowMajor = false>
97: struct DefaultMultistageTrmmComplex;
98: 
99: ////////////////////////////////////////////////////////////////////////////////
100: 
101: /// Specialization for row-major output
102: template <
103:     /// Element type for A matrix operand
104:     typename ElementA,
105:     /// Layout type for A matrix operand
```
**EN:** Declares template parameters and begins the definition of DefaultMultistageTrmmComplex.
**CN:** 声明模板参数并开始定义 DefaultMultistageTrmmComplex。

### Lines 106-115
```cpp
106:     typename LayoutA,
107:     /// Element type for B matrix operand
108:     typename ElementB,
109:     /// Layout type for B matrix operand
110:     typename LayoutB,
111:     /// Side Mode for the kernel
112:     SideMode kSideMode,
113:     /// Fill Mode for the triangular matrix
114:     FillMode kFillMode,
115:     /// Diag Type for the triangular matrix
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 116-125
```cpp
116:     DiagType kDiagType,
117:     /// Element type for internal accumulation
118:     typename ElementAccumulator,
119:     /// Tag indicating architecture to tune for
120:     typename OperatorClass,
121:     /// Tag indicating architecture to tune for
122:     typename ArchTag,
123:     /// Threadblock-level tile size (concept: GemmShape)
124:     typename ThreadblockShape,
125:     /// Warp-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 126-135
```cpp
126:     typename WarpShape,
127:     /// Instruction-level tile size (concept: GemmShape)
128:     typename InstructionShape,
129:     /// Number of stages used in the multistage mainloop
130:     int Stages,
131:     /// Complex transformation on operand A
132:     ComplexTransform TransformA,
133:     /// Complex transformation on operand B
134:     ComplexTransform TransformB,
135:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 136-141
```cpp
136:     typename Operator>
137: struct DefaultMultistageTrmmComplex<ElementA, LayoutA, ElementB, LayoutB,
138:                             kSideMode, kFillMode, kDiagType,
139:                             ElementAccumulator, layout::RowMajor, OperatorClass, ArchTag, ThreadblockShape, WarpShape,
140:                             InstructionShape, Stages, TransformA, TransformB, Operator> {
141:   // Define the MmaCore components
```
**EN:** Defines DefaultMultistageTrmmComplex, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageTrmmComplex，用于封装策略、存储或算法行为的辅助类型。

### Lines 142-147
```cpp
142:   using MmaCore = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplexCore<
143:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA, 
144:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, OperatorClass,
145:       Stages, TransformA, TransformB, Operator>;
146: 
147:   // Define iterators over tiles from the A operand
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 148-157
```cpp
148:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
149:   using AccessTypeA = cutlass::Array<ElementA, ThreadMapA::kElementsPerAccess>;
150:   using IteratorA =
151:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
152:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
153:           ElementA, LayoutA, 1, ThreadMapA, 
154:           kSideMode, kFillMode, kDiagType, 
155:           AccessTypeA>;
156: 
157:   // Define iterators over tiles from the B operand
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 158-167
```cpp
158:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
159:   using AccessTypeB = cutlass::Array<ElementB, ThreadMapB::kElementsPerAccess>;
160:   using IteratorB =
161:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
162:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
163:           ElementB, LayoutB, 0, ThreadMapB, 
164:           kSideMode, FillMode::kFull, DiagType::kInvalid,
165:           AccessTypeB>;
166: 
167:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 168-177
```cpp
168:   using ThreadblockMma = cutlass::gemm::threadblock::MmaMultistage<
169:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
170:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
171:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
172:       typename MmaCore::MmaPolicy, Stages, SharedMemoryClearOption::kZfill>;
173: };
174: 
175: ////////////////////////////////////////////////////////////////////////////////
176: 
177: /// Specialization for row-major output and right-side mode
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 178-187
```cpp
178: template <
179:     /// Element type for A matrix operand
180:     typename ElementA,
181:     /// Layout type for A matrix operand
182:     typename LayoutA,
183:     /// Element type for B matrix operand
184:     typename ElementB,
185:     /// Layout type for B matrix operand
186:     typename LayoutB,
187:     /// Fill Mode for the triangular matrix
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 188-197
```cpp
188:     FillMode kFillMode,
189:     /// Diag Type for the triangular matrix
190:     DiagType kDiagType,
191:     /// Element type for internal accumulation
192:     typename ElementAccumulator,
193:     /// Tag indicating architecture to tune for
194:     typename OperatorClass,
195:     /// Tag indicating architecture to tune for
196:     typename ArchTag,
197:     /// Threadblock-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 198-207
```cpp
198:     typename ThreadblockShape,
199:     /// Warp-level tile size (concept: GemmShape)
200:     typename WarpShape,
201:     /// Instruction-level tile size (concept: GemmShape)
202:     typename InstructionShape,
203:     /// Number of stages used in the multistage mainloop
204:     int Stages,
205:     /// Complex transformation on operand A
206:     ComplexTransform TransformA,
207:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 208-209
```cpp
208:     ComplexTransform TransformB,
209:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 210-215
```cpp
210:     typename Operator>
211: struct DefaultMultistageTrmmComplex<ElementA, LayoutA, ElementB, LayoutB,
212:                             SideMode::kRight, kFillMode, kDiagType,
213:                             ElementAccumulator, layout::RowMajor, OperatorClass, ArchTag, ThreadblockShape, WarpShape,
214:                             InstructionShape, Stages, TransformA, TransformB, Operator> {
215:   // Define the MmaCore components
```
**EN:** Defines DefaultMultistageTrmmComplex, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageTrmmComplex，用于封装策略、存储或算法行为的辅助类型。

### Lines 216-221
```cpp
216:   using MmaCore = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplexCore<
217:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA, 
218:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, OperatorClass,
219:       Stages, TransformA, TransformB, Operator>;
220: 
221:   // Define iterators over tiles from the A operand
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 222-231
```cpp
222:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
223:   using AccessTypeA = cutlass::Array<ElementA, ThreadMapA::kElementsPerAccess>;
224:   using IteratorA =
225:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
226:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
227:           ElementA, LayoutA, 1, ThreadMapA, 
228:           SideMode::kRight, FillMode::kFull, DiagType::kInvalid, 
229:           AccessTypeA>;
230: 
231:   // Define iterators over tiles from the B operand
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 232-241
```cpp
232:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
233:   using AccessTypeB = cutlass::Array<ElementB, ThreadMapB::kElementsPerAccess>;
234:   using IteratorB =
235:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
236:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
237:           ElementB, LayoutB, 0, ThreadMapB, 
238:           SideMode::kRight, kFillMode, kDiagType,
239:           AccessTypeB>;
240: 
241:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 242-251
```cpp
242:   using ThreadblockMma = cutlass::gemm::threadblock::MmaMultistage<
243:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
244:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
245:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
246:       typename MmaCore::MmaPolicy, Stages, SharedMemoryClearOption::kZfill>;
247: };
248: 
249: ////////////////////////////////////////////////////////////////////////////////
250: 
251: /// Specialization for row-major output with unit diagonal
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 252-261
```cpp
252: template <
253:     /// Element type for A matrix operand
254:     typename ElementA,
255:     /// Layout type for A matrix operand
256:     typename LayoutA,
257:     /// Element type for B matrix operand
258:     typename ElementB,
259:     /// Layout type for B matrix operand
260:     typename LayoutB,
261:     /// Side Mode for the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 262-271
```cpp
262:     SideMode kSideMode,
263:     /// Fill Mode for the triangular matrix
264:     FillMode kFillMode,
265:     /// Element type for internal accumulation
266:     typename ElementAccumulator,
267:     /// Tag indicating architecture to tune for
268:     typename OperatorClass,
269:     /// Tag indicating architecture to tune for
270:     typename ArchTag,
271:     /// Threadblock-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 272-281
```cpp
272:     typename ThreadblockShape,
273:     /// Warp-level tile size (concept: GemmShape)
274:     typename WarpShape,
275:     /// Instruction-level tile size (concept: GemmShape)
276:     typename InstructionShape,
277:     /// Number of stages used in the multistage mainloop
278:     int Stages,
279:     /// Complex transformation on operand A
280:     ComplexTransform TransformA,
281:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 282-283
```cpp
282:     ComplexTransform TransformB,
283:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 284-289
```cpp
284:     typename Operator>
285: struct DefaultMultistageTrmmComplex<ElementA, LayoutA, ElementB, LayoutB,
286:                             kSideMode, kFillMode, DiagType::kUnit,
287:                             ElementAccumulator, layout::RowMajor, OperatorClass, ArchTag, ThreadblockShape, WarpShape,
288:                             InstructionShape, Stages, TransformA, TransformB, Operator> {
289:   // Define the MmaCore components
```
**EN:** Defines DefaultMultistageTrmmComplex, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageTrmmComplex，用于封装策略、存储或算法行为的辅助类型。

### Lines 290-295
```cpp
290:   using MmaCore = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplexCore<
291:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA, 
292:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, OperatorClass,
293:       Stages, TransformA, TransformB, Operator>;
294: 
295:   // Define iterators over tiles from the A operand
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 296-305
```cpp
296:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
297:   using AccessTypeA = cutlass::Array<ElementA, ThreadMapA::kElementsPerAccess>;
298:   using IteratorA =
299:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
300:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
301:           ElementA, LayoutA, 1, ThreadMapA, 
302:           kSideMode, kFillMode, DiagType::kUnit, 
303:           AccessTypeA>;
304: 
305:   // Define iterators over tiles from the B operand
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 306-315
```cpp
306:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
307:   using AccessTypeB = cutlass::Array<ElementB, ThreadMapB::kElementsPerAccess>;
308:   using IteratorB =
309:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
310:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
311:           ElementB, LayoutB, 0, ThreadMapB, 
312:           kSideMode, FillMode::kFull, DiagType::kInvalid,
313:           AccessTypeB>;
314: 
315:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 316-325
```cpp
316:   using ThreadblockMma = cutlass::gemm::threadblock::MmaBlas3Multistage<
317:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
318:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
319:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
320:       typename MmaCore::MmaPolicy, Stages, SharedMemoryClearOption::kZfill>;
321: };
322: 
323: ////////////////////////////////////////////////////////////////////////////////
324: 
325: /// Specialization for row-major output and right-side mode, unit diagonal
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 326-335
```cpp
326: template <
327:     /// Element type for A matrix operand
328:     typename ElementA,
329:     /// Layout type for A matrix operand
330:     typename LayoutA,
331:     /// Element type for B matrix operand
332:     typename ElementB,
333:     /// Layout type for B matrix operand
334:     typename LayoutB,
335:     /// Fill Mode for the triangular matrix
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 336-345
```cpp
336:     FillMode kFillMode,
337:     /// Element type for internal accumulation
338:     typename ElementAccumulator,
339:     /// Tag indicating architecture to tune for
340:     typename OperatorClass,
341:     /// Tag indicating architecture to tune for
342:     typename ArchTag,
343:     /// Threadblock-level tile size (concept: GemmShape)
344:     typename ThreadblockShape,
345:     /// Warp-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 346-355
```cpp
346:     typename WarpShape,
347:     /// Instruction-level tile size (concept: GemmShape)
348:     typename InstructionShape,
349:     /// Number of stages used in the multistage mainloop
350:     int Stages,
351:     /// Complex transformation on operand A
352:     ComplexTransform TransformA,
353:     /// Complex transformation on operand B
354:     ComplexTransform TransformB,
355:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 356-361
```cpp
356:     typename Operator>
357: struct DefaultMultistageTrmmComplex<ElementA, LayoutA, ElementB, LayoutB,
358:                             SideMode::kRight, kFillMode, DiagType::kUnit,
359:                             ElementAccumulator, layout::RowMajor, OperatorClass, ArchTag, ThreadblockShape, WarpShape,
360:                             InstructionShape, Stages, TransformA, TransformB, Operator> {
361:   // Define the MmaCore components
```
**EN:** Defines DefaultMultistageTrmmComplex, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageTrmmComplex，用于封装策略、存储或算法行为的辅助类型。

### Lines 362-367
```cpp
362:   using MmaCore = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplexCore<
363:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA, 
364:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, OperatorClass,
365:       Stages, TransformA, TransformB, Operator>;
366: 
367:   // Define iterators over tiles from the A operand
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 368-377
```cpp
368:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
369:   using AccessTypeA = cutlass::Array<ElementA, ThreadMapA::kElementsPerAccess>;
370:   using IteratorA =
371:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
372:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
373:           ElementA, LayoutA, 1, ThreadMapA, 
374:           SideMode::kRight, FillMode::kFull, DiagType::kInvalid, 
375:           AccessTypeA>;
376: 
377:   // Define iterators over tiles from the B operand
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 378-387
```cpp
378:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
379:   using AccessTypeB = cutlass::Array<ElementB, ThreadMapB::kElementsPerAccess>;
380:   using IteratorB =
381:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
382:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
383:           ElementB, LayoutB, 0, ThreadMapB, 
384:           SideMode::kRight, kFillMode, DiagType::kUnit,
385:           AccessTypeB>;
386: 
387:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 388-397
```cpp
388:   using ThreadblockMma = cutlass::gemm::threadblock::MmaBlas3Multistage<
389:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
390:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
391:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
392:       typename MmaCore::MmaPolicy, Stages, SharedMemoryClearOption::kZfill>;
393: };
394: 
395: 
396: ////////////////////////////////////////////////////////////////////////////////
397: 
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 398-408
```cpp
398: /// Specialization for row-major output (for TRMM where diagonal imag part is ignored - used by HEMM)
399: template <
400:     /// Element type for A matrix operand
401:     typename ElementA,
402:     /// Layout type for A matrix operand
403:     typename LayoutA,
404:     /// Element type for B matrix operand
405:     typename ElementB,
406:     /// Layout type for B matrix operand
407:     typename LayoutB,
408:     /// Side Mode for the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 409-418
```cpp
409:     SideMode kSideMode,
410:     /// Fill Mode for the triangular matrix
411:     FillMode kFillMode,
412:     /// Element type for internal accumulation
413:     typename ElementAccumulator,
414:     /// Tag indicating architecture to tune for
415:     typename OperatorClass,
416:     /// Tag indicating architecture to tune for
417:     typename ArchTag,
418:     /// Threadblock-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 419-428
```cpp
419:     typename ThreadblockShape,
420:     /// Warp-level tile size (concept: GemmShape)
421:     typename WarpShape,
422:     /// Instruction-level tile size (concept: GemmShape)
423:     typename InstructionShape,
424:     /// Number of stages used in the multistage mainloop
425:     int Stages,
426:     /// Complex transformation on operand A
427:     ComplexTransform TransformA,
428:     /// Complex transformation on operand B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 429-430
```cpp
429:     ComplexTransform TransformB,
430:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 431-437
```cpp
431:     typename Operator>
432: struct DefaultMultistageTrmmComplex<ElementA, LayoutA, ElementB, LayoutB,
433:                             kSideMode, kFillMode, DiagType::kNonUnit,
434:                             ElementAccumulator, layout::RowMajor, OperatorClass, ArchTag, ThreadblockShape, WarpShape,
435:                             InstructionShape, Stages, TransformA, TransformB, Operator, BlasMode::kHermitian> {
436: 
437:   // Define the MmaCore components
```
**EN:** Defines DefaultMultistageTrmmComplex, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageTrmmComplex，用于封装策略、存储或算法行为的辅助类型。

### Lines 438-442
```cpp
438:   using MmaCore = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplexCore<
439:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA, 
440:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, OperatorClass,
441:       Stages, TransformA, TransformB, Operator>;
442: 
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 443-445
```cpp
443:   // Define iterators over tiles from the A operand
444:   // PredicatedTileAccessIteratorTriangularMatrix only tracks diagonal elements,
445:   // when DiagType is kUnit
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 446-455
```cpp
446:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
447:   using AccessTypeA = cutlass::Array<ElementA, ThreadMapA::kElementsPerAccess>;
448:   using IteratorA =
449:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
450:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
451:           ElementA, LayoutA, 1, ThreadMapA, 
452:           kSideMode, kFillMode, DiagType::kUnit, 
453:           AccessTypeA>;
454: 
455:   // Define iterators over tiles from the B operand
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 456-465
```cpp
456:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
457:   using AccessTypeB = cutlass::Array<ElementB, ThreadMapB::kElementsPerAccess>;
458:   using IteratorB =
459:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
460:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
461:           ElementB, LayoutB, 0, ThreadMapB, 
462:           kSideMode, FillMode::kFull, DiagType::kInvalid,
463:           AccessTypeB>;
464: 
465:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 466-475
```cpp
466:   using ThreadblockMma = cutlass::gemm::threadblock::MmaBlas3Multistage<
467:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
468:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
469:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
470:       typename MmaCore::MmaPolicy, Stages, SharedMemoryClearOption::kZfill,
471:       BlasMode::kHermitian>;
472: };
473: 
474: ////////////////////////////////////////////////////////////////////////////////
475: 
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 476-486
```cpp
476: /// Specialization for row-major output and right-side mode (for TRMM where diagonal imag part is ignored - used by HEMM)
477: template <
478:     /// Element type for A matrix operand
479:     typename ElementA,
480:     /// Layout type for A matrix operand
481:     typename LayoutA,
482:     /// Element type for B matrix operand
483:     typename ElementB,
484:     /// Layout type for B matrix operand
485:     typename LayoutB,
486:     /// Fill Mode for the triangular matrix
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 487-496
```cpp
487:     FillMode kFillMode,
488:     /// Element type for internal accumulation
489:     typename ElementAccumulator,
490:     /// Tag indicating architecture to tune for
491:     typename OperatorClass,
492:     /// Tag indicating architecture to tune for
493:     typename ArchTag,
494:     /// Threadblock-level tile size (concept: GemmShape)
495:     typename ThreadblockShape,
496:     /// Warp-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 497-506
```cpp
497:     typename WarpShape,
498:     /// Instruction-level tile size (concept: GemmShape)
499:     typename InstructionShape,
500:     /// Number of stages used in the multistage mainloop
501:     int Stages,
502:     /// Complex transformation on operand A
503:     ComplexTransform TransformA,
504:     /// Complex transformation on operand B
505:     ComplexTransform TransformB,
506:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 507-513
```cpp
507:     typename Operator>
508: struct DefaultMultistageTrmmComplex<ElementA, LayoutA, ElementB, LayoutB,
509:                             SideMode::kRight, kFillMode, DiagType::kNonUnit,
510:                             ElementAccumulator, layout::RowMajor, OperatorClass, ArchTag, ThreadblockShape, WarpShape,
511:                             InstructionShape, Stages, TransformA, TransformB, Operator, BlasMode::kHermitian> {
512: 
513:   // Define the MmaCore components
```
**EN:** Defines DefaultMultistageTrmmComplex, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMultistageTrmmComplex，用于封装策略、存储或算法行为的辅助类型。

### Lines 514-519
```cpp
514:   using MmaCore = typename cutlass::gemm::threadblock::DefaultMultistageMmaComplexCore<
515:       ThreadblockShape, WarpShape, InstructionShape, ElementA, LayoutA, 
516:       ElementB, LayoutB, ElementAccumulator, layout::RowMajor, OperatorClass,
517:       Stages, TransformA, TransformB, Operator>;
518: 
519:   // Define iterators over tiles from the A operand
```
**EN:** Introduces local type aliases (MmaCore) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaCore），简化后续模板代码。

### Lines 520-528
```cpp
520:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
521:   using AccessTypeA = cutlass::Array<ElementA, ThreadMapA::kElementsPerAccess>;
522:   using IteratorA =
523:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
524:           cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
525:           ElementA, LayoutA, 1, ThreadMapA, 
526:           SideMode::kRight, FillMode::kFull, DiagType::kInvalid, 
527:           AccessTypeA>;
528: 
```
**EN:** Introduces local type aliases (ThreadMapA, AccessTypeA, IteratorA) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapA, AccessTypeA, IteratorA），简化后续模板代码。

### Lines 529-531
```cpp
529:   // Define iterators over tiles from the B operand
530:   // PredicatedTileAccessIteratorTriangularMatrix only tracks diagonal elements,
531:   // when DiagType is kUnit
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 532-541
```cpp
532:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
533:   using AccessTypeB = cutlass::Array<ElementB, ThreadMapB::kElementsPerAccess>;
534:   using IteratorB =
535:       cutlass::transform::threadblock::PredicatedTileAccessIteratorTriangularMatrix<
536:           cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
537:           ElementB, LayoutB, 0, ThreadMapB, 
538:           SideMode::kRight, kFillMode, DiagType::kUnit,
539:           AccessTypeB>;
540: 
541:   // Define the threadblock-scoped multistage matrix multiply
```
**EN:** Introduces local type aliases (ThreadMapB, AccessTypeB, IteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadMapB, AccessTypeB, IteratorB），简化后续模板代码。

### Lines 542-551
```cpp
542:   using ThreadblockMma = cutlass::gemm::threadblock::MmaBlas3Multistage<
543:       typename MmaCore::Shape, IteratorA, typename MmaCore::SmemIteratorA,
544:       MmaCore::kCacheOpA, IteratorB, typename MmaCore::SmemIteratorB,
545:       MmaCore::kCacheOpB, ElementAccumulator, layout::RowMajor,
546:       typename MmaCore::MmaPolicy, Stages, SharedMemoryClearOption::kZfill,
547:       BlasMode::kHermitian>;
548: };
549: 
550: ////////////////////////////////////////////////////////////////////////////////
551: 
```
**EN:** Introduces local type aliases (ThreadblockMma) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockMma），简化后续模板代码。

### Lines 552-556
```cpp
552: }  // namespace threadblock
553: }  // namespace gemm
554: }  // namespace cutlass
555: 
556: ////////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Threadblock-level tiling and shared memory orchestration  
  **CN:** 线程块级分块与共享内存编排
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线
- **EN:** Multi-stage mainloop buffering  
  **CN:** 多阶段主循环缓冲
- **EN:** Tile iterator abstractions  
  **CN:** Tile 迭代器抽象
- **EN:** Complex-valued multiply-accumulate support  
  **CN:** 复数乘加支持

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `tag`, `DefaultMultistageTrmmComplex`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
