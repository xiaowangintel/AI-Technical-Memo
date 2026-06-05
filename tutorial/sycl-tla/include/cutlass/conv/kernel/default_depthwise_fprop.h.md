# default_depthwise_fprop.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/kernel/default_depthwise_fprop.h`
- **Purpose (EN):** Default kernel-level Depthwise implicit GEMM convolution definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue.
- **用途 (CN):** 组合 默认 深度卷积 前向传播 对应的内核级卷积逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
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
**EN:** Records the BSD-3-Clause license and redistribution conditions for the file.

**CN:** 记录该文件的 BSD-3-Clause 许可和再分发条件。

### Lines 32-36
```cpp
 32: /*! \file
 33:     \brief 
 34:     Default kernel-level Depthwise implicit GEMM convolution definitions combine threadblock-scoped 
 35:       matrix multiply-add with the appropriate threadblock-scoped epilogue.  
 36: */
```
**EN:** Documents the file's purpose, terminology, and high-level usage.

**CN:** 说明文件的用途、术语以及高层使用方式。

### Lines 38-38
```cpp
 38: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 40-42
```cpp
 40: #include "cutlass/cutlass.h"
 41: #include "cutlass/conv/kernel/default_conv2d.h"
 42: #include "cutlass/conv/kernel/direct_convolution.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `default_conv2d.h`, `direct_convolution.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `default_conv2d.h`, `direct_convolution.h`。

### Lines 44-44
```cpp
 44: #include "cutlass/conv/threadblock/depthwise_mma_core_with_lane_access_size.h"
```
**EN:** Imports direct dependencies used later in the file, including `depthwise_mma_core_with_lane_access_size.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `depthwise_mma_core_with_lane_access_size.h`。

### Lines 46-48
```cpp
 46: #include "cutlass/conv/threadblock/conv2d_fprop_activation_tile_access_iterator_analytic.h"
 47: #include "cutlass/conv/threadblock/conv2d_fprop_filter_tile_access_iterator_analytic.h"
 48: #include "cutlass/conv/threadblock/depthwise_fprop_pipelined.h"
```
**EN:** Imports direct dependencies used later in the file, including `conv2d_fprop_activation_tile_access_iterator_analytic.h`, `conv2d_fprop_filter_tile_access_iterator_analytic.h`, `depthwise_fprop_pipelined.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `conv2d_fprop_activation_tile_access_iterator_analytic.h`, `conv2d_fprop_filter_tile_access_iterator_analytic.h`, `depthwise_fprop_pipelined.h`。

### Lines 50-52
```cpp
 50: // Direct Conv Related Header files
 51: #include "cutlass/conv/threadblock/depthwise_fprop_activation_tile_access_iterator_direct_conv_optimized.h"
 52: #include "cutlass/conv/threadblock/depthwise_fprop_activation_tile_access_iterator_direct_conv_fixed_stride_dilation.h"
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 54-55
```cpp
 54: #include "cutlass/conv/threadblock/depthwise_fprop_filter_tile_access_iterator_direct_conv_optimized.h"
 55: #include "cutlass/conv/threadblock/depthwise_fprop_direct_conv_multistage.h"
```
**EN:** Imports direct dependencies used later in the file, including `depthwise_fprop_filter_tile_access_iterator_direct_conv_optimized.h`, `depthwise_fprop_direct_conv_multistage.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `depthwise_fprop_filter_tile_access_iterator_direct_conv_optimized.h`, `depthwise_fprop_direct_conv_multistage.h`。

### Lines 59-61
```cpp
 59: namespace cutlass {
 60: namespace conv {
 61: namespace kernel {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 64-88
```cpp
 64: /// Defines a kernel for DepthwiseFprop
 65: template <
 66:   typename ElementA,
 67:   typename LayoutA,
 68:   typename ElementB,
 69:   typename LayoutB,
 70:   typename ElementC,
 71:   typename LayoutC,
 72:   typename ElementAccumulator,
 73:   typename OperatorClass,
 74:   typename ArchTag,
 75:   typename ThreadblockShape,
 76:   typename WarpShape,
 77:   typename InstructionShape,
 78:   typename EpilogueOutputOp,
 79:   typename ThreadblockSwizzle,
 80:   int Stages,
 81:   typename MathOperatorTag,
 82:   conv::IteratorAlgorithm IteratorAlgorithm = IteratorAlgorithm::kAnalytic,
 83:   conv::StrideSupport StrideSupport = StrideSupport::kUnity,
 84:   /// Access granularity of A matrix in units of elements
 85:   int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value,
 86:   /// Access granularity of B matrix in units of elements
 87:   int AlignmentB = cutlass::sizeof_bits<ElementB>::value / cutlass::sizeof_bits<ElementB>::value
 88: > struct DefaultDepthwiseFprop;
```
**EN:** Stores member state such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 91-121
```cpp
 91: /// Defines a kernel for DepthwiseFprop with direct convolution algorithm
 92: template <
 93:   typename ElementA,
 94:   typename LayoutA,
 95:   typename ElementB,
 96:   typename LayoutB,
 97:   typename ElementC,
 98:   typename LayoutC,
 99:   typename ElementAccumulator,
100:   typename OperatorClass,
101:   typename ArchTag,
102:   typename ThreadblockShape,
103:   typename ThreadBlockOutputShape,
104:   typename FilterShape,
105:   typename WarpShape,
106:   typename InstructionShape,
107:   typename EpilogueOutputOp,
108:   typename ThreadblockSwizzle,
109:   int Stages,
110:   typename MathOperatorTag,
111:   conv::IteratorAlgorithm IteratorAlgorithm = IteratorAlgorithm::kAnalytic,
112:   conv::StrideSupport StrideSupport = StrideSupport::kUnity,
113:   // MatrixShape<Height, Width>
114:   typename StrideShape = cutlass::MatrixShape<-1, -1>,
115:   // MatrixShape< Height, Width> 
116:   typename DilationShape =  cutlass::MatrixShape<-1, -1>, 
117:   /// Access granularity of A matrix in units of elements
118:   int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value,
119:   /// Access granularity of B matrix in units of elements
120:   int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value
121: > struct DefaultDepthwiseDirect2dConvFprop;
```
**EN:** Stores member state such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 124-167
```cpp
124: //                            OpClassSimt convolutions
126: /// Defines a kernel for Depthwise specialization for Analytic IteratorAlgorithm
127: template <
128:   typename ElementA,
129:   typename LayoutA,
130:   typename ElementB,
131:   typename LayoutB,
132:   typename ElementC,
133:   typename LayoutC,
134:   typename ElementAccumulator,
135:   typename ArchTag,
136:   typename ThreadblockShape,
137:   typename WarpShape,
138:   typename InstructionShape,
139:   typename EpilogueOutputOp,
140:   typename ThreadblockSwizzle,
141:   typename MathOperatorTag,
142:   conv::StrideSupport StrideSupport,
143:   int AlignmentA,
144:   int AlignmentB
145: >
146: struct DefaultDepthwiseFprop <
147:   ElementA,
148:   LayoutA,
149:   ElementB,
150:   LayoutB,
151:   ElementC,
152:   LayoutC,
153:   ElementAccumulator,
154:   arch::OpClassSimt,
155:   ArchTag,
156:   ThreadblockShape,
157:   WarpShape,
158:   InstructionShape,
159:   EpilogueOutputOp,
160:   ThreadblockSwizzle,
161:   2,
162:   MathOperatorTag, //   cutlass::arch::OpMultiplyAdd
163:   IteratorAlgorithm::kAnalytic,
164:   StrideSupport,
165:   AlignmentA,
166:   AlignmentB
167: > {
```
**EN:** Stores member state such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 169-184
```cpp
169:   // Define the core components from GEMM
170:   using MmaCore = typename cutlass::conv::threadblock::DepthwiseMmaCoreWithLaneAccessSize<
171:       ThreadblockShape,
172:       WarpShape,
173:       InstructionShape,
174:       ElementA,
175:       layout::RowMajor,
176:       ElementB,
177:       layout::ColumnMajor,
178:       ElementAccumulator,
179:       layout::RowMajor,
180:       arch::OpClassSimt,
181:       128,
182:       sizeof_bits<ElementB>::value,
183:       2,
184:       MathOperatorTag>;
```
**EN:** Introduces aliases such as `MmaCore` to keep the surrounding template code readable.

**CN:** 引入 `MmaCore` 等别名，以提升周围模板代码的可读性。

### Lines 186-195
```cpp
186:   // Define iterators over tiles from the A operand
187:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
188:   using IteratorA =
189:     cutlass::conv::threadblock::TileIterator<
190:       cutlass::conv::threadblock::Conv2dFpropActivationTileAccessIteratorAnalytic<
191:         cutlass::MatrixShape<ThreadblockShape::kM, ThreadblockShape::kK>,
192:         ElementA, LayoutA,
193:         ThreadMapA
194:       >
195:     >;
```
**EN:** Introduces aliases such as `ThreadMapA`, `IteratorA` to keep the surrounding template code readable.

**CN:** 引入 `ThreadMapA`, `IteratorA` 等别名，以提升周围模板代码的可读性。

### Lines 197-197
```cpp
197:   using SmemIteratorA = typename MmaCore::SmemIteratorA;
```
**EN:** Introduces aliases such as `SmemIteratorA` to keep the surrounding template code readable.

**CN:** 引入 `SmemIteratorA` 等别名，以提升周围模板代码的可读性。

### Lines 199-211
```cpp
199:   // Define iterators over tiles from the B operand
200:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
201:   using AccessTypeB = cutlass::AlignedArray<ElementB, AlignmentB>;
202:   using IteratorB =
203:     cutlass::conv::threadblock::TileIterator<
204:       cutlass::conv::threadblock::Conv2dFpropFilterTileAccessIteratorAnalytic<
205:         cutlass::MatrixShape<ThreadblockShape::kK, ThreadblockShape::kN>,
206:         ElementB, LayoutB,
207:         ThreadMapB,
208:         AccessTypeB,
209:         cutlass::conv::GroupMode::kDepthwise
210:       >
211:     >;
```
**EN:** Introduces aliases such as `ThreadMapB`, `AccessTypeB`, `IteratorB` to keep the surrounding template code readable.

**CN:** 引入 `ThreadMapB`, `AccessTypeB`, `IteratorB` 等别名，以提升周围模板代码的可读性。

### Lines 213-213
```cpp
213:   using SmemIteratorB = typename MmaCore::SmemIteratorB;
```
**EN:** Introduces aliases such as `SmemIteratorB` to keep the surrounding template code readable.

**CN:** 引入 `SmemIteratorB` 等别名，以提升周围模板代码的可读性。

### Lines 215-217
```cpp
215:   // Warp-level GEMM components
216:   using WarpMmaSimtOp = typename MmaCore::MmaWarpSimt;
217:   using MmaPolicy = typename MmaCore::MmaPolicy;
```
**EN:** Introduces aliases such as `WarpMmaSimtOp`, `MmaPolicy` to keep the surrounding template code readable.

**CN:** 引入 `WarpMmaSimtOp`, `MmaPolicy` 等别名，以提升周围模板代码的可读性。

### Lines 219-229
```cpp
219:   // Define the Mma
220:   using Mma = threadblock::DepthwiseFpropPipelined<
221:     ThreadblockShape,
222:     IteratorA,
223:     SmemIteratorA,
224:     IteratorB,
225:     SmemIteratorB,
226:     ElementC,
227:     LayoutC,
228:     MmaPolicy
229:   >;
```
**EN:** Introduces aliases such as `Mma` to keep the surrounding template code readable.

**CN:** 引入 `Mma` 等别名，以提升周围模板代码的可读性。

### Lines 231-237
```cpp
231:   // Define the epilogue
232:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueSimt<
233:     ThreadblockShape,
234:     WarpMmaSimtOp,
235:     EpilogueOutputOp,
236:     EpilogueOutputOp::kCount
237:   >::Epilogue;
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 239-248
```cpp
239:   // Define the kernel
240:   using Kernel = cutlass::conv::kernel::ImplicitGemmConvolution<
241:     Mma,
242:     Epilogue,
243:     ThreadblockSwizzle,
244:     conv::Operator::kFprop,
245:     Conv2dProblemSize,
246:     cutlass::conv::GroupMode::kDepthwise
247:   >;
248: };
```
**EN:** Introduces aliases such as `Kernel` to keep the surrounding template code readable.

**CN:** 引入 `Kernel` 等别名，以提升周围模板代码的可读性。

### Lines 251-302
```cpp
251: /// Defines a kernel for Depthwise specialization for direct 2d conv implementation, 
252: /// multiple stage pipeline, and SIMT-based mainloop
253: template <
254:   typename ElementA,
255:   typename LayoutA,
256:   typename ElementB,
257:   typename LayoutB,
258:   typename ElementC,
259:   typename LayoutC,
260:   typename ElementAccumulator,
261:   typename ArchTag,
262:   typename ThreadblockShape,
263:   typename ThreadBlockOutputShape,
264:   typename FilterShape,
265:   typename WarpShape,
266:   typename InstructionShape,
267:   typename EpilogueOutputOp,
268:   typename ThreadblockSwizzle,
269:   int Stages,
270:   typename MathOperatorTag,
271:   conv::StrideSupport StrideSupport,
272:   typename StrideShape,
273:   typename DilationShape,
274:   int AlignmentA,
275:   int AlignmentB
276: >
277: struct DefaultDepthwiseDirect2dConvFprop <
278:   ElementA,
279:   LayoutA,
280:   ElementB,
281:   LayoutB,
282:   ElementC,
283:   LayoutC,
284:   ElementAccumulator,
285:   arch::OpClassSimt,
286:   ArchTag,
287:   ThreadblockShape,
288:   ThreadBlockOutputShape,
289:   FilterShape,
290:   WarpShape,
291:   InstructionShape,
292:   EpilogueOutputOp,
293:   ThreadblockSwizzle,
294:   Stages,
295:   MathOperatorTag,
296:   IteratorAlgorithm::kOptimized,
297:   StrideSupport,
298:   StrideShape,
299:   DilationShape,
300:   AlignmentA,
301:   AlignmentB
302: > {
```
**EN:** Stores member state such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 303-310
```cpp
303:   // One warp handles the entrie groups per cta.
304:   static_assert(ThreadblockShape::kN == WarpShape::kN,
305:                 "ThreadblockShape::kN should be same as WarpShape::kN ");
306:   static_assert(ThreadblockShape::kK == FilterShape::kCount && WarpShape::kK == FilterShape::kCount,
307:                 "ThreadblockShape::kK and WarpShape::kK should be same as filter size");
308:   static_assert(ThreadblockShape::kM % WarpShape::kM == 0,
309:                 "ThreadblockShape::kM must be divisible by WarpShape shape::kM");
310:   static_assert(ThreadBlockOutputShape::kN, "ThreadBlockOutputShape::kN should be 1");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 312-329
```cpp
312:   // Define the core components from GEMM
313:   using MmaCore = typename cutlass::conv::threadblock::DepthwiseDirectConvMmaCoreWithLaneAccessSize<
314:       ThreadblockShape,
315:       ThreadBlockOutputShape,
316:       FilterShape,
317:       WarpShape,
318:       InstructionShape,
319:       ElementA,
320:       layout::RowMajor,
321:       ElementB,
322:       layout::ColumnMajor,
323:       ElementAccumulator,
324:       layout::RowMajor,
325:       arch::OpClassSimt,
326:       128,
327:       128,
328:       Stages,
329:       MathOperatorTag>;
```
**EN:** Introduces aliases such as `MmaCore` to keep the surrounding template code readable.

**CN:** 引入 `MmaCore` 等别名，以提升周围模板代码的可读性。

### Lines 331-339
```cpp
331:   // Define iterators over tiles from the A operand
332:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
333:   using IteratorA =
334:     cutlass::conv::threadblock::DepthwiseFpropActivationDirect2dConvTileAccessIteratorOptimized<
335:       cutlass::MatrixShape<ThreadblockShape::kM,ThreadblockShape::kN>, // < outputShape:KMNK, groups per cta>
336:       ThreadBlockOutputShape,
337:       ElementA, LayoutA,
338:       ThreadMapA
339:     >;
```
**EN:** Introduces aliases such as `ThreadMapA`, `IteratorA` to keep the surrounding template code readable.

**CN:** 引入 `ThreadMapA`, `IteratorA` 等别名，以提升周围模板代码的可读性。

### Lines 341-341
```cpp
341:   using SmemIteratorA = typename MmaCore::SmemIteratorA;
```
**EN:** Introduces aliases such as `SmemIteratorA` to keep the surrounding template code readable.

**CN:** 引入 `SmemIteratorA` 等别名，以提升周围模板代码的可读性。

### Lines 343-351
```cpp
343:   // Define iterators over tiles from the B operand
344:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
345:   using AccessTypeB = cutlass::AlignedArray<ElementB, AlignmentB>;
346:   using IteratorB =
347:       cutlass::conv::threadblock::DepthwiseFpropFilterDirectConvTileAccessIteratorOptimized<
348:         cutlass::MatrixShape<ThreadblockShape::kN, FilterShape::kCount>,
349:         ElementB, LayoutB,
350:         ThreadMapB
351:       >;
```
**EN:** Introduces aliases such as `ThreadMapB`, `AccessTypeB`, `IteratorB` to keep the surrounding template code readable.

**CN:** 引入 `ThreadMapB`, `AccessTypeB`, `IteratorB` 等别名，以提升周围模板代码的可读性。

### Lines 353-353
```cpp
353:   using SmemIteratorB = typename MmaCore::SmemIteratorB;
```
**EN:** Introduces aliases such as `SmemIteratorB` to keep the surrounding template code readable.

**CN:** 引入 `SmemIteratorB` 等别名，以提升周围模板代码的可读性。

### Lines 355-362
```cpp
355:   // Warp-level GEMM components
356:   using WarpMmaSimtOp = typename MmaCore::MmaWarpSimt;
357:   using MmaPolicy = typename MmaCore::MmaPolicy;
358:   using ThreadOutputShape = typename MmaCore::ThreadOutputShape;
359:   static cutlass::arch::CacheOperation::Kind const CacheOpA =
360:       ((sizeof_bits<ElementA>::value * AlignmentA) == 128)
361:           ? cutlass::arch::CacheOperation::Global
362:           : cutlass::arch::CacheOperation::Always;
```
**EN:** Introduces aliases such as `WarpMmaSimtOp`, `MmaPolicy`, `ThreadOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `WarpMmaSimtOp`, `MmaPolicy`, `ThreadOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 364-367
```cpp
364:   static cutlass::arch::CacheOperation::Kind const CacheOpB =
365:       ((sizeof_bits<ElementB>::value * AlignmentB) == 128)
366:           ? cutlass::arch::CacheOperation::Global
367:           : cutlass::arch::CacheOperation::Always;
```
**EN:** Stores member state such as `CacheOpB`, `Always` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `CacheOpB`, `Always` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 369-377
```cpp
369:   // Define the epilogue
370:   using Epilogue = typename epilogue::threadblock::DefaultDirectConvEpilogueSimt<
371:     ThreadblockShape, // < outputShape:KMNK, groups per cta>
372:     WarpMmaSimtOp,
373:     EpilogueOutputOp,
374:     EpilogueOutputOp::kCount,
375:     ThreadOutputShape,
376:     ThreadBlockOutputShape
377:   >::Epilogue;
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 379-391
```cpp
379:   // Define the Mma
380:   using Mma = threadblock::DepthwiseFpropDirectConvMultipleStage<
381:     ThreadblockShape,
382:     IteratorA,
383:     SmemIteratorA,
384:     CacheOpA,
385:     IteratorB,
386:     SmemIteratorB,
387:     CacheOpB,
388:     MmaPolicy,
389:     Stages,
390:     Epilogue
391:   >;
```
**EN:** Introduces aliases such as `Mma` to keep the surrounding template code readable.

**CN:** 引入 `Mma` 等别名，以提升周围模板代码的可读性。

### Lines 393-403
```cpp
393:   // Define the kernel
394:   using Kernel = cutlass::conv::kernel::DirectConvolution<
395:     Mma,
396:     Epilogue,
397:     ThreadblockSwizzle,
398:     conv::Operator::kFprop,
399:     Conv2dProblemSize,
400:     cutlass::conv::GroupMode::kDepthwise,
401:     ThreadBlockOutputShape
402:   >;
403: };
```
**EN:** Introduces aliases such as `Kernel` to keep the surrounding template code readable.

**CN:** 引入 `Kernel` 等别名，以提升周围模板代码的可读性。

### Lines 406-457
```cpp
406: /// Defines a kernel for Depthwise specialization for direct 2d conv implementation, 
407: /// multiple stage pipeline, and SIMT-based mainloop
408: template <
409:   typename ElementA,
410:   typename LayoutA,
411:   typename ElementB,
412:   typename LayoutB,
413:   typename ElementC,
414:   typename LayoutC,
415:   typename ElementAccumulator,
416:   typename ArchTag,
417:   typename ThreadblockShape,
418:   typename ThreadBlockOutputShape,
419:   typename FilterShape,
420:   typename WarpShape,
421:   typename InstructionShape,
422:   typename EpilogueOutputOp,
423:   typename ThreadblockSwizzle,
424:   int Stages,
425:   typename MathOperatorTag,
426:   conv::StrideSupport StrideSupport,
427:   typename StrideShape,
428:   typename DilationShape,
429:   int AlignmentA,
430:   int AlignmentB
431: >
432: struct DefaultDepthwiseDirect2dConvFprop <
433:   ElementA,
434:   LayoutA,
435:   ElementB,
436:   LayoutB,
437:   ElementC,
438:   LayoutC,
439:   ElementAccumulator,
440:   arch::OpClassSimt,
441:   ArchTag,
442:   ThreadblockShape,
443:   ThreadBlockOutputShape,
444:   FilterShape,
445:   WarpShape,
446:   InstructionShape,
447:   EpilogueOutputOp,
448:   ThreadblockSwizzle,
449:   Stages,
450:   MathOperatorTag,
451:   IteratorAlgorithm::kFixedStrideDilation,
452:   StrideSupport,
453:   StrideShape,
454:   DilationShape,
455:   AlignmentA,
456:   AlignmentB
457: > {
```
**EN:** Stores member state such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 461-468
```cpp
461:   // One warp handles the entrie groups per cta.
462:   static_assert(ThreadblockShape::kN == WarpShape::kN,
463:                 "ThreadblockShape::kN should be same as WarpShape::kN ");
464:   static_assert(ThreadblockShape::kK == FilterShape::kCount && WarpShape::kK == FilterShape::kCount,
465:                 "ThreadblockShape::kK and WarpShape::kK should be same as filter size");
466:   static_assert(ThreadblockShape::kM % WarpShape::kM == 0,
467:                 "ThreadblockShape::kM must be divisible by WarpShape shape::kM");
468:   static_assert(ThreadBlockOutputShape::kN, "ThreadBlockOutputShape::kN should be 1");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 470-471
```cpp
470:   static_assert(StrideShape::kRow >= 0 && StrideShape::kColumn >= 0, "Stride should be fixed");
471:   static_assert(DilationShape::kRow >= 0 && DilationShape::kColumn >= 0, "Stride should be fixed");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 473-475
```cpp
473:   // Activations loaded by threadblock
474:   static int const ActivationShapeH = (ThreadBlockOutputShape::kH - 1) * StrideShape::kRow +
475:                              (FilterShape::kRow - 1) * DilationShape::kRow + 1;
```
**EN:** Defines compile-time constants such as compile-time constants that parameterize later logic.

**CN:** 定义 compile-time constants 等编译期常量，用来参数化后续逻辑。

### Lines 477-478
```cpp
477:   static int const ActivationShapeW = (ThreadBlockOutputShape::kW - 1) * StrideShape::kColumn +
478:                              (FilterShape::kColumn - 1) * DilationShape::kColumn + 1;
```
**EN:** Defines compile-time constants such as compile-time constants that parameterize later logic.

**CN:** 定义 compile-time constants 等编译期常量，用来参数化后续逻辑。

### Lines 480-481
```cpp
480:   using ActivationShape =
481:       cutlass::conv::TensorNHWCShape<1, ActivationShapeH, ActivationShapeW, ThreadblockShape::kN >;
```
**EN:** Introduces aliases such as `ActivationShape` to keep the surrounding template code readable.

**CN:** 引入 `ActivationShape` 等别名，以提升周围模板代码的可读性。

### Lines 483-504
```cpp
483:   // Define the core components from GEMM
484:   using MmaCore = typename cutlass::conv::threadblock::DepthwiseDirectConvMmaCoreWithLaneAccessSize<
485:       ThreadblockShape,
486:       ThreadBlockOutputShape,
487:       FilterShape,
488:       WarpShape,
489:       InstructionShape,
490:       ElementA,
491:       layout::RowMajor,
492:       ElementB,
493:       layout::ColumnMajor,
494:       ElementAccumulator,
495:       layout::RowMajor,
496:       arch::OpClassSimt,
497:       128,
498:       128,
499:       Stages,
500:       MathOperatorTag,
501:       IteratorAlgorithm::kFixedStrideDilation,
502:       StrideShape,
503:       DilationShape,
504:       ActivationShape>;
```
**EN:** Introduces aliases such as `MmaCore` to keep the surrounding template code readable.

**CN:** 引入 `MmaCore` 等别名，以提升周围模板代码的可读性。

### Lines 506-517
```cpp
506:   // Define iterators over tiles from the A operand
507:   using ThreadMapA = typename MmaCore::IteratorThreadMapA;
508:   using IteratorA =
509:     cutlass::conv::threadblock::DepthwiseFpropActivationDirect2dConvTileAccessIteratorFixedStrideDilation<
510:       cutlass::MatrixShape<ThreadblockShape::kM,ThreadblockShape::kN>, // < outputShape:KMNK, groups per cta>
511:       ThreadBlockOutputShape,
512:       StrideShape,
513:       DilationShape,
514:       ActivationShape,
515:       ElementA, LayoutA,
516:       ThreadMapA
517:     >;
```
**EN:** Introduces aliases such as `ThreadMapA`, `IteratorA` to keep the surrounding template code readable.

**CN:** 引入 `ThreadMapA`, `IteratorA` 等别名，以提升周围模板代码的可读性。

### Lines 519-519
```cpp
519:   using SmemIteratorA = typename MmaCore::SmemIteratorA;
```
**EN:** Introduces aliases such as `SmemIteratorA` to keep the surrounding template code readable.

**CN:** 引入 `SmemIteratorA` 等别名，以提升周围模板代码的可读性。

### Lines 521-529
```cpp
521:   // Define iterators over tiles from the B operand
522:   using ThreadMapB = typename MmaCore::IteratorThreadMapB;
523:   using AccessTypeB = cutlass::AlignedArray<ElementB, AlignmentB>;
524:   using IteratorB =
525:       cutlass::conv::threadblock::DepthwiseFpropFilterDirectConvTileAccessIteratorOptimized<
526:         cutlass::MatrixShape<ThreadblockShape::kN, FilterShape::kCount>,
527:         ElementB, LayoutB,
528:         ThreadMapB
529:       >;
```
**EN:** Introduces aliases such as `ThreadMapB`, `AccessTypeB`, `IteratorB` to keep the surrounding template code readable.

**CN:** 引入 `ThreadMapB`, `AccessTypeB`, `IteratorB` 等别名，以提升周围模板代码的可读性。

### Lines 531-531
```cpp
531:   using SmemIteratorB = typename MmaCore::SmemIteratorB;
```
**EN:** Introduces aliases such as `SmemIteratorB` to keep the surrounding template code readable.

**CN:** 引入 `SmemIteratorB` 等别名，以提升周围模板代码的可读性。

### Lines 533-540
```cpp
533:   // Warp-level GEMM components
534:   using WarpMmaSimtOp = typename MmaCore::MmaWarpSimt;
535:   using MmaPolicy = typename MmaCore::MmaPolicy;
536:   using ThreadOutputShape = typename MmaCore::ThreadOutputShape;
537:   static cutlass::arch::CacheOperation::Kind const CacheOpA =
538:       ((sizeof_bits<ElementA>::value * AlignmentA) == 128)
539:           ? cutlass::arch::CacheOperation::Global
540:           : cutlass::arch::CacheOperation::Always;
```
**EN:** Introduces aliases such as `WarpMmaSimtOp`, `MmaPolicy`, `ThreadOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `WarpMmaSimtOp`, `MmaPolicy`, `ThreadOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 542-545
```cpp
542:   static cutlass::arch::CacheOperation::Kind const CacheOpB =
543:       ((sizeof_bits<ElementB>::value * AlignmentB) == 128)
544:           ? cutlass::arch::CacheOperation::Global
545:           : cutlass::arch::CacheOperation::Always;
```
**EN:** Stores member state such as `CacheOpB`, `Always` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `CacheOpB`, `Always` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 547-555
```cpp
547:   // Define the epilogue
548:   using Epilogue = typename epilogue::threadblock::DefaultDirectConvEpilogueSimt<
549:     ThreadblockShape, // < outputShape:KMNK, groups per cta>
550:     WarpMmaSimtOp,
551:     EpilogueOutputOp,
552:     EpilogueOutputOp::kCount,
553:     ThreadOutputShape,
554:     ThreadBlockOutputShape
555:   >::Epilogue;
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 557-570
```cpp
557:   // Define the Mma
558:   using Mma = threadblock::DepthwiseFpropDirectConvMultipleStage<
559:     ThreadblockShape,
560:     IteratorA,
561:     SmemIteratorA,
562:     CacheOpA,
563:     IteratorB,
564:     SmemIteratorB,
565:     CacheOpB,
566:     MmaPolicy,
567:     Stages,
568:     Epilogue,
569:     IteratorAlgorithm::kFixedStrideDilation
570:   >;
```
**EN:** Introduces aliases such as `Mma` to keep the surrounding template code readable.

**CN:** 引入 `Mma` 等别名，以提升周围模板代码的可读性。

### Lines 572-582
```cpp
572:   // Define the kernel
573:   using Kernel = cutlass::conv::kernel::DirectConvolution<
574:     Mma,
575:     Epilogue,
576:     ThreadblockSwizzle,
577:     conv::Operator::kFprop,
578:     Conv2dProblemSize,
579:     cutlass::conv::GroupMode::kDepthwise,
580:     ThreadBlockOutputShape
581:   >;
582: };
```
**EN:** Introduces aliases such as `Kernel` to keep the surrounding template code readable.

**CN:** 引入 `Kernel` 等别名，以提升周围模板代码的可读性。

### Lines 584-586
```cpp
584: } // namespace kernel
585: } // namespace conv
586: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Default kernel-level Depthwise implicit GEMM convolution definitions combine threadblock-scoped matrix multiply-add with the appropriate threadblock-scoped epilogue. **CN:** 核心作用：组合 默认 深度卷积 前向传播 对应的内核级卷积逻辑。
- **EN:** Key exported symbols include `DefaultDepthwiseFprop`, `DefaultDepthwiseDirect2dConvFprop`, `MmaCore`, `ThreadMapA`, `IteratorA`, `SmemIteratorA`. **CN:** 关键导出符号包括 `DefaultDepthwiseFprop`, `DefaultDepthwiseDirect2dConvFprop`, `MmaCore`, `ThreadMapA`, `IteratorA`, `SmemIteratorA`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** MMA composition is central: the file either defines math operators directly or prepares data for them. **CN:** MMA 组合是核心：该文件要么直接定义数学算子，要么为其准备数据。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/conv/kernel/default_conv2d.h`
- `cutlass/conv/kernel/direct_convolution.h`
- `cutlass/conv/threadblock/depthwise_mma_core_with_lane_access_size.h`
- `cutlass/conv/threadblock/conv2d_fprop_activation_tile_access_iterator_analytic.h`
- `cutlass/conv/threadblock/conv2d_fprop_filter_tile_access_iterator_analytic.h`
- `cutlass/conv/threadblock/depthwise_fprop_pipelined.h`
- `cutlass/conv/threadblock/depthwise_fprop_activation_tile_access_iterator_direct_conv_optimized.h`
- `cutlass/conv/threadblock/depthwise_fprop_activation_tile_access_iterator_direct_conv_fixed_stride_dilation.h`
- `cutlass/conv/threadblock/depthwise_fprop_filter_tile_access_iterator_direct_conv_optimized.h`
- `cutlass/conv/threadblock/depthwise_fprop_direct_conv_multistage.h`

### Internal Relationships / 内部关系
- **EN:** Depends on architecture-specific intrinsics or tags. **CN:** 依赖体系结构相关的内建操作或标签。
- **EN:** Relies on tensor or matrix layout helpers. **CN:** 依赖张量或矩阵布局辅助类型。
- **EN:** Collaborates with threadblock-scoped convolution components. **CN:** 与线程块级卷积组件协同工作。
