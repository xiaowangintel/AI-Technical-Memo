# default_mma_complex_tensor_op.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/warp/default_mma_complex_tensor_op.h`
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
32:     \brief Default warp-level GEMM operators selected by data type, size, and layouts of operands.
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
38: #include "cutlass/gemm/warp/mma_complex_tensor_op.h"
39: #include "cutlass/gemm/warp/mma_complex_tensor_op_fast_f32.h"
40: #include "cutlass/gemm/warp/mma_gaussian_complex_tensor_op.h"
41: #include "cutlass/layout/tensor_op_multiplicand_sm80.h"
42: 
43: namespace cutlass {
44: namespace gemm {
45: namespace warp {
46: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, warp components, layout types.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、warp 组件、布局类型。

### Lines 47-56
```cpp
47: /////////////////////////////////////////////////////////////////////////////////////////////////
48: 
49: template <
50:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
51:     typename WarpShape_,
52:     /// Shape of one matrix production operation (concept: GemmShape)
53:     typename InstructionShape_,
54:     /// Data type of A elements
55:     typename ElementA_,
56:     /// Layout of A matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 57-66
```cpp
57:     typename LayoutA_,
58:     /// Data type of B elements
59:     typename ElementB_,
60:     /// Layout of B matrix (concept: MatrixLayout)
61:     typename LayoutB_,
62:     /// Element type of C matrix
63:     typename ElementC_,
64:     /// Layout of C matrix (concept: MatrixLayout)
65:     typename LayoutC_,
66:     /// Complex transform on A operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 67-70
```cpp
67:     ComplexTransform TransformA = ComplexTransform::kNone,
68:     /// Complex transform on B operand
69:     ComplexTransform TransformB = ComplexTransform::kNone,
70:     /// Multiply-add operator (arch::OpMultiplyAddComplex, arch::OpMultiplyGaussianComplex)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 71-75
```cpp
71:     typename Operator_ = arch::OpMultiplyAddComplex>
72: struct DefaultMmaComplexTensorOp;
73: 
74: /////////////////////////////////////////////////////////////////////////////////////////////////
75: 
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 76-86
```cpp
76: /// Partial specialization for complex<T>*complex<T> case
77: //  4 real-valued mma operations
78: //  A = (ar + j ai), B (br +j bi), D = AB
79: //  D = dr + j di = (ar*br - ai*bi) + j (ar*bi + ai*br) 
80: /////////////////////////////////////////////////////////////////////////////////////////////////
81: template <
82:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
83:     typename WarpShape_,
84:     /// Shape of one matrix production operation (concept: GemmShape)
85:     typename InstructionShape_,
86:     /// Real-valued underlying type of complex-valued A operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 87-96
```cpp
87:     typename RealElementA,
88:     /// Layout of A matrix (concept: MatrixLayout)
89:     typename LayoutA,
90:     /// Real-valued underlying type of complex-valued B operand
91:     typename RealElementB,
92:     /// Layout of B matrix (concept: MatrixLayout)
93:     typename LayoutB,
94:     /// Real-valued underlying type of complex-valued C operand
95:     typename RealElementC,
96:     /// Layout of C matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 97-100
```cpp
97:     typename LayoutC,
98:     /// Complex transform on A operand
99:     ComplexTransform TransformA,
100:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 101-112
```cpp
101:     ComplexTransform TransformB>
102: struct DefaultMmaComplexTensorOp<
103:     WarpShape_,
104:     InstructionShape_,
105:     complex<RealElementA>,
106:     LayoutA,
107:     complex<RealElementB>,
108:     LayoutB,
109:     complex<RealElementC>,
110:     LayoutC,
111:     TransformA,
112:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 113-114
```cpp
113:     arch::OpMultiplyAddComplex> {
114: 
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 115-126
```cpp
115:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
116:       cutlass::arch::Mma<
117:         InstructionShape_, 
118:         32, 
119:         RealElementA,
120:         cutlass::layout::RowMajor,
121:         RealElementB,
122:         cutlass::layout::ColumnMajor,
123:         RealElementC,
124:         cutlass::layout::RowMajor, 
125:         arch::OpMultiplyAdd>,
126:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 127-129
```cpp
127:     >;
128: 
129:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 130-141
```cpp
130:   using Type = cutlass::gemm::warp::MmaComplexTensorOp<
131:     WarpShape_,
132:     complex<RealElementA>,
133:     LayoutA,
134:     complex<RealElementB>,
135:     LayoutB,
136:     complex<RealElementC>,
137:     LayoutC, 
138:     Policy,
139:     TransformA,
140:     TransformB>;
141: };
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 142-144
```cpp
142: 
143: /////////////////////////////////////////////////////////////////////////////////////////////////
144: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 145-154
```cpp
145: /// Partial specialization for complex<T>*complex<T> case using GaussianComplex operation
146: //  3 real-valued mma operations
147: //  A  = (ar + j ai), B = (br +j bi), D = AB
148: //  P1 = (ar + ai) * br, P2 = - ar * (br - bi), P3 = ai * (br + bi) 
149: //  D  = dr + j di = (P1 - P3) + j (P1 + P2)
150: /////////////////////////////////////////////////////////////////////////////////////////////////
151: template <
152:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
153:     typename WarpShape_,
154:     /// Shape of one matrix production operation (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 155-164
```cpp
155:     typename InstructionShape_,
156:     /// Real-valued underlying type of complex-valued A operand
157:     typename RealElementA,
158:     /// Layout of A matrix (concept: MatrixLayout)
159:     typename LayoutA,
160:     /// Real-valued underlying type of complex-valued B operand
161:     typename RealElementB,
162:     /// Layout of B matrix (concept: MatrixLayout)
163:     typename LayoutB,
164:     /// Real-valued underlying type of complex-valued C operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 165-170
```cpp
165:     typename RealElementC,
166:     /// Layout of C matrix (concept: MatrixLayout)
167:     typename LayoutC,
168:     /// Complex transform on A operand
169:     ComplexTransform TransformA,
170:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 171-182
```cpp
171:     ComplexTransform TransformB>
172: struct DefaultMmaComplexTensorOp<
173:     WarpShape_,
174:     InstructionShape_,
175:     complex<RealElementA>,
176:     LayoutA,
177:     complex<RealElementB>,
178:     LayoutB,
179:     complex<RealElementC>,
180:     LayoutC,
181:     TransformA,
182:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 183-184
```cpp
183:     arch::OpMultiplyAddGaussianComplex> {
184: 
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 185-196
```cpp
185:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
186:       cutlass::arch::Mma<
187:         InstructionShape_, 
188:         32, 
189:         RealElementA,
190:         cutlass::layout::RowMajor,
191:         RealElementB,
192:         cutlass::layout::ColumnMajor,
193:         RealElementC,
194:         cutlass::layout::RowMajor, 
195:         arch::OpMultiplyAdd>,
196:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 197-199
```cpp
197:     >;
198: 
199:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 200-211
```cpp
200:   using Type = cutlass::gemm::warp::MmaGaussianComplexTensorOp<
201:     WarpShape_,
202:     complex<RealElementA>,
203:     LayoutA,
204:     complex<RealElementB>,
205:     LayoutB,
206:     complex<RealElementC>,
207:     LayoutC, 
208:     Policy,
209:     TransformA,
210:     TransformB>;
211: };
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 212-213
```cpp
212: /////////////////////////////////////////////////////////////////////////////////////////////////
213: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 214-224
```cpp
214: /////////////////////////////////////////////////////////////////////////////////////////////////
215: /// Partial specialization - input and output types are complex<float>*complex<float> 
216: //  Use TF32 tensor operation internally
217: //  4 real-valued mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32 operations on TF32 
218: //  A = (ar + j ai), B (br +j bi), D = AB
219: //  D = dr + j di = (ar*br - ai*bi) + j (ar*bi + ai*br) 
220: /////////////////////////////////////////////////////////////////////////////////////////////////
221: template <
222:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
223:     typename WarpShape_,
224:     /// Shape of one matrix production operation (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 225-234
```cpp
225:     typename InstructionShape_,
226:     /// Layout of A matrix (concept: MatrixLayout)
227:     typename LayoutA,
228:     /// Layout of B matrix (concept: MatrixLayout)
229:     typename LayoutB,
230:     /// Layout of C matrix (concept: MatrixLayout)
231:     typename LayoutC,
232:     /// Complex transform on A operand
233:     ComplexTransform TransformA,
234:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 235-246
```cpp
235:     ComplexTransform TransformB>
236: struct DefaultMmaComplexTensorOp<
237:     WarpShape_,
238:     InstructionShape_,
239:     complex<float>,
240:     LayoutA,
241:     complex<float>,
242:     LayoutB,
243:     complex<float>,
244:     LayoutC,
245:     TransformA,
246:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 247-249
```cpp
247:     arch::OpMultiplyAddComplex> {
248: 
249:   // Complex floating point tensor operation use mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32 mma instruction
```
**EN:** This block focuses on tensor, complex related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、复数处理 的实现细节。

### Lines 250-261
```cpp
250:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
251:       cutlass::arch::Mma<
252:         InstructionShape_, 
253:         32, 
254:         tfloat32_t,
255:         cutlass::layout::RowMajor,
256:         tfloat32_t,
257:         cutlass::layout::ColumnMajor,
258:         float,
259:         cutlass::layout::RowMajor, 
260:         arch::OpMultiplyAdd>,
261:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 262-264
```cpp
262:     >;
263: 
264:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 265-276
```cpp
265:   using Type = cutlass::gemm::warp::MmaComplexTensorOp<
266:     WarpShape_,
267:     complex<float>,
268:     LayoutA,
269:     complex<float>,
270:     LayoutB,
271:     complex<float>,
272:     LayoutC, 
273:     Policy,
274:     TransformA,
275:     TransformB>;
276: };
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 277-277
```cpp
277: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 278-288
```cpp
278: /////////////////////////////////////////////////////////////////////////////////////////////////
279: /// Partial specialization - input and output types are complex<float>*complex<float> 
280: //  Use BF16 tensor operation internally
281: //  4 real-valued mma.sync.aligned.m16n8k8.f32.bf16.bf16.f32 operations on BF16
282: //  A = (ar + j ai), B (br +j bi), D = AB
283: //  D = dr + j di = (ar*br - ai*bi) + j (ar*bi + ai*br) 
284: /////////////////////////////////////////////////////////////////////////////////////////////////
285: template <
286:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
287:     typename WarpShape_,
288:     /// Shape of one matrix production operation (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 289-298
```cpp
289:     typename InstructionShape_,
290:     /// Layout of A matrix (concept: MatrixLayout)
291:     typename LayoutA,
292:     /// Layout of B matrix (concept: MatrixLayout)
293:     typename LayoutB,
294:     /// Layout of C matrix (concept: MatrixLayout)
295:     typename LayoutC,
296:     /// Complex transform on A operand
297:     ComplexTransform TransformA,
298:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 299-310
```cpp
299:     ComplexTransform TransformB>
300: struct DefaultMmaComplexTensorOp<
301:     WarpShape_,
302:     InstructionShape_,
303:     complex<float>,
304:     LayoutA,
305:     complex<float>,
306:     LayoutB,
307:     complex<float>,
308:     LayoutC,
309:     TransformA,
310:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 311-313
```cpp
311:     arch::OpMultiplyAddFastBF16> {
312: 
313:   // Complex floating point tensor operation use mma.sync.aligned.m16n8k8.f32.bf16.bf16.f32 mma instruction
```
**EN:** This block focuses on tensor, complex related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、复数处理 的实现细节。

### Lines 314-325
```cpp
314:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
315:       cutlass::arch::Mma<
316:         InstructionShape_, 
317:         32, 
318:         bfloat16_t,
319:         cutlass::layout::RowMajor,
320:         bfloat16_t,
321:         cutlass::layout::ColumnMajor,
322:         float,
323:         cutlass::layout::RowMajor, 
324:         arch::OpMultiplyAdd>,
325:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 326-328
```cpp
326:     >;
327: 
328:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 329-340
```cpp
329:   using Type = cutlass::gemm::warp::MmaComplexTensorOp<
330:     WarpShape_,
331:     complex<float>,
332:     LayoutA,
333:     complex<float>,
334:     LayoutB,
335:     complex<float>,
336:     LayoutC, 
337:     Policy,
338:     TransformA,
339:     TransformB>;
340: };
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 341-341
```cpp
341: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 342-352
```cpp
342: /////////////////////////////////////////////////////////////////////////////////////////////////
343: /// Partial specialization - input and output types are complex<float>*complex<float> 
344: //  Use F16 tensor operation internally
345: //  4 real-valued mma.sync.aligned.m16n8k8.f32.f16.f16.f32 operations on F16
346: //  A = (ar + j ai), B (br +j bi), D = AB
347: //  D = dr + j di = (ar*br - ai*bi) + j (ar*bi + ai*br) 
348: /////////////////////////////////////////////////////////////////////////////////////////////////
349: template <
350:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
351:     typename WarpShape_,
352:     /// Shape of one matrix production operation (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 353-362
```cpp
353:     typename InstructionShape_,
354:     /// Layout of A matrix (concept: MatrixLayout)
355:     typename LayoutA,
356:     /// Layout of B matrix (concept: MatrixLayout)
357:     typename LayoutB,
358:     /// Layout of C matrix (concept: MatrixLayout)
359:     typename LayoutC,
360:     /// Complex transform on A operand
361:     ComplexTransform TransformA,
362:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 363-374
```cpp
363:     ComplexTransform TransformB>
364: struct DefaultMmaComplexTensorOp<
365:     WarpShape_,
366:     InstructionShape_,
367:     complex<float>,
368:     LayoutA,
369:     complex<float>,
370:     LayoutB,
371:     complex<float>,
372:     LayoutC,
373:     TransformA,
374:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 375-377
```cpp
375:     arch::OpMultiplyAddFastF16> {
376: 
377:   // Complex floating point tensor operation use mma.sync.aligned.m16n8k8.f32.f16.f16.f32 mma instruction
```
**EN:** This block focuses on tensor, complex related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、复数处理 的实现细节。

### Lines 378-389
```cpp
378:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
379:       cutlass::arch::Mma<
380:         InstructionShape_, 
381:         32, 
382:         half_t,
383:         cutlass::layout::RowMajor,
384:         half_t,
385:         cutlass::layout::ColumnMajor,
386:         float,
387:         cutlass::layout::RowMajor, 
388:         arch::OpMultiplyAdd>,
389:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 390-392
```cpp
390:     >;
391: 
392:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 393-404
```cpp
393:   using Type = cutlass::gemm::warp::MmaComplexTensorOp<
394:     WarpShape_,
395:     complex<float>,
396:     LayoutA,
397:     complex<float>,
398:     LayoutB,
399:     complex<float>,
400:     LayoutC, 
401:     Policy,
402:     TransformA,
403:     TransformB>;
404: };
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 405-405
```cpp
405: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 406-415
```cpp
406: /////////////////////////////////////////////////////////////////////////////////////////////////
407: /// 3xTF32 or 4xTF32 (fast and accurate complex<float> operation)
408: /// Partial specialization - input and output types are complex<float> * complex<float> 
409: //  Use 3xTF32 or 4xTF32 tensor operation internally
410: //  4 real-valued mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32 operations on TF32 
411: //  A = (ar + j ai), B (br +j bi), D = AB
412: //  D = dr + j di = 3x[(ar*br - ai*bi) + j (ar*bi + ai*br)]
413: /////////////////////////////////////////////////////////////////////////////////////////////////
414: /////////////////////////////////////////////////////////////////////////////////////////////////
415: 
```
**EN:** This block focuses on tensor, complex related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、复数处理 的实现细节。

### Lines 416-425
```cpp
416: template <
417:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
418:     typename WarpShape_,
419:     /// Shape of one matrix production operation (concept: GemmShape)
420:     typename InstructionShape_,
421:     /// Layout of A matrix (concept: MatrixLayout)
422:     typename LayoutA,
423:     /// Layout of B matrix (concept: MatrixLayout)
424:     typename LayoutB,
425:     /// Layout of C matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 426-429
```cpp
426:     typename LayoutC,
427:     /// Complex transform on A operand
428:     ComplexTransform TransformA,
429:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 430-441
```cpp
430:     ComplexTransform TransformB>
431: struct DefaultMmaComplexTensorOp<
432:     WarpShape_,
433:     InstructionShape_,
434:     complex<float>,
435:     LayoutA,
436:     complex<float>,
437:     LayoutB,
438:     complex<float>,
439:     LayoutC,
440:     TransformA,
441:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 442-444
```cpp
442:     arch::OpMultiplyAddComplexFastF32> {
443: 
444:   // Complex floating point tensor operation use mma.sync.aligned.m16n8k8.f32.tf32.tf32.f32 mma instruction
```
**EN:** This block focuses on tensor, complex related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、复数处理 的实现细节。

### Lines 445-456
```cpp
445:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
446:       cutlass::arch::Mma<
447:         InstructionShape_, 
448:         32, 
449:         tfloat32_t,
450:         cutlass::layout::RowMajor,
451:         tfloat32_t,
452:         cutlass::layout::ColumnMajor,
453:         float,
454:         cutlass::layout::RowMajor, 
455:         arch::OpMultiplyAdd>,
456:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 457-459
```cpp
457:     >;
458: 
459:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 460-471
```cpp
460:   using Type = cutlass::gemm::warp::MmaComplexTensorOpFastF32<
461:     WarpShape_,
462:     complex<float>,
463:     LayoutA,
464:     complex<float>,
465:     LayoutB,
466:     complex<float>,
467:     LayoutC, 
468:     Policy,
469:     TransformA,
470:     TransformB>;
471: };
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 472-474
```cpp
472: 
473: /////////////////////////////////////////////////////////////////////////////////////////////////
474: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 475-485
```cpp
475: /// Partial specialization for complex<double>*complex<double> case
476: //  4 real-valued mma.sync.aligned.m16n8k4.f64.f64.f64.f64 operations
477: //  A = (ar + j ai), B (br +j bi), D = AB
478: //  D = dr + j di = (ar*br - ai*bi) + j (ar*bi + ai*br) 
479: /////////////////////////////////////////////////////////////////////////////////////////////////
480: template <
481:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
482:     typename WarpShape_,
483:     /// Real-valued underlying type of complex-valued A operand
484:     typename RealElementA,
485:     /// Layout of A matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 486-495
```cpp
486:     typename LayoutA,
487:     /// Real-valued underlying type of complex-valued B operand
488:     typename RealElementB,
489:     /// Layout of B matrix (concept: MatrixLayout)
490:     typename LayoutB,
491:     /// Real-valued underlying type of complex-valued C operand
492:     typename RealElementC,
493:     /// Layout of C matrix (concept: MatrixLayout)
494:     typename LayoutC,
495:     /// Complex transform on A operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 496-497
```cpp
496:     ComplexTransform TransformA,
497:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 498-509
```cpp
498:     ComplexTransform TransformB>
499: struct DefaultMmaComplexTensorOp<
500:     WarpShape_,
501:     GemmShape<16, 8, 4>,
502:     complex<RealElementA>,
503:     LayoutA,
504:     complex<RealElementB>,
505:     LayoutB,
506:     complex<RealElementC>,
507:     LayoutC,
508:     TransformA,
509:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 510-511
```cpp
510:     arch::OpMultiplyAddComplex> {
511: 
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 512-523
```cpp
512:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
513:       cutlass::arch::Mma<
514:         GemmShape<16, 8, 4>,
515:         32, 
516:         RealElementA,
517:         cutlass::layout::RowMajor,
518:         RealElementB,
519:         cutlass::layout::ColumnMajor,
520:         RealElementC,
521:         cutlass::layout::RowMajor, 
522:         arch::OpMultiplyAdd>,
523:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 524-526
```cpp
524:     >;
525: 
526:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 527-538
```cpp
527:   using Type = cutlass::gemm::warp::MmaComplexTensorOp<
528:     WarpShape_,
529:     complex<RealElementA>,
530:     LayoutA,
531:     complex<RealElementB>,
532:     LayoutB,
533:     complex<RealElementC>,
534:     LayoutC, 
535:     Policy,
536:     TransformA,
537:     TransformB,
538:     true>;
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 539-540
```cpp
539: };
540: 
```
**EN:** Closes the current type or namespace scope.
**CN:** 结束当前类型或命名空间作用域。

### Lines 541-551
```cpp
541: /////////////////////////////////////////////////////////////////////////////////////////////////
542: /// Partial specialization for complex<T>*complex<T> case using GaussianComplex operation
543: //  3 real-valued mma.sync.aligned.m16n8k4.f64.f64.f64.f64 operations 
544: //  A  = (ar + j ai), B = (br +j bi), D = AB
545: //  P1 = (ar + ai) * br, P2 = - ar * (br - bi), P3 = ai * (br + bi) 
546: //  D  = dr + j di = (P1 - P3) + j (P1 + P2)
547: /////////////////////////////////////////////////////////////////////////////////////////////////
548: template <
549:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
550:     typename WarpShape_,
551:     /// Real-valued underlying type of complex-valued A operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 552-561
```cpp
552:     typename RealElementA,
553:     /// Layout of A matrix (concept: MatrixLayout)
554:     typename LayoutA,
555:     /// Real-valued underlying type of complex-valued B operand
556:     typename RealElementB,
557:     /// Layout of B matrix (concept: MatrixLayout)
558:     typename LayoutB,
559:     /// Real-valued underlying type of complex-valued C operand
560:     typename RealElementC,
561:     /// Layout of C matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 562-565
```cpp
562:     typename LayoutC,
563:     /// Complex transform on A operand
564:     ComplexTransform TransformA,
565:     /// Complex transform on B operand
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 566-577
```cpp
566:     ComplexTransform TransformB>
567: struct DefaultMmaComplexTensorOp<
568:     WarpShape_,
569:     GemmShape<16, 8, 4>,
570:     complex<RealElementA>,
571:     LayoutA,
572:     complex<RealElementB>,
573:     LayoutB,
574:     complex<RealElementC>,
575:     LayoutC,
576:     TransformA,
577:     TransformB,
```
**EN:** Defines DefaultMmaComplexTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 DefaultMmaComplexTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 578-579
```cpp
578:     arch::OpMultiplyAddGaussianComplex> {
579: 
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 580-591
```cpp
580:   using Policy = cutlass::gemm::warp::MmaTensorOpPolicy<
581:       cutlass::arch::Mma<
582:         GemmShape<16, 8, 4>,
583:         32, 
584:         RealElementA,
585:         cutlass::layout::RowMajor,
586:         RealElementB,
587:         cutlass::layout::ColumnMajor,
588:         RealElementC,
589:         cutlass::layout::RowMajor, 
590:         arch::OpMultiplyAdd>,
591:       cutlass::MatrixShape<1, 1>
```
**EN:** Introduces local type aliases (Policy) to simplify downstream template code.
**CN:** 引入本地类型别名（Policy），简化后续模板代码。

### Lines 592-594
```cpp
592:     >;
593: 
594:   // Define the warp-level tensor op
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 595-606
```cpp
595:   using Type = cutlass::gemm::warp::MmaGaussianComplexTensorOp<
596:     WarpShape_,
597:     complex<RealElementA>,
598:     LayoutA,
599:     complex<RealElementB>,
600:     LayoutB,
601:     complex<RealElementC>,
602:     LayoutC, 
603:     Policy,
604:     TransformA,
605:     TransformB,
606:     true>;
```
**EN:** Introduces local type aliases (Type) to simplify downstream template code.
**CN:** 引入本地类型别名（Type），简化后续模板代码。

### Lines 607-612
```cpp
607: };
608: /////////////////////////////////////////////////////////////////////////////////////////////////
609: 
610: } // namespace warp
611: } // namespace gemm
612: } // namespace cutlass
```
**EN:** Closes the current type or namespace scope.
**CN:** 结束当前类型或命名空间作用域。

## Key Concepts / 关键概念

- **EN:** Warp-level fragment movement and MMA sequencing  
  **CN:** warp 级 fragment 搬运与 MMA 时序
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Tensor Core instruction mapping  
  **CN:** Tensor Core 指令映射
- **EN:** Complex-valued multiply-accumulate support  
  **CN:** 复数乘加支持

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `DefaultMmaComplexTensorOp`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
