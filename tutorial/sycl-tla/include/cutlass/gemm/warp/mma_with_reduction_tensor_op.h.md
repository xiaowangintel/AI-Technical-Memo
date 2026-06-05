# mma_with_reduction_tensor_op.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/warp/mma_with_reduction_tensor_op.h`
- **Purpose (EN):** Implements warp-scoped matrix multiply/iterator components.
- **用途 (CN):** 实现 warp 级矩阵乘加与迭代器组件。

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
32:     \brief Templates implementing warp-level matrix multiply-accumulate operations targeting
33:       Tensor Cores.
34: */
35: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 36-37
```cpp
36: #pragma once
37: 
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 38-41
```cpp
38: #include "cutlass/cutlass.h"
39: #include "cutlass/array.h"
40: #include "cutlass/platform/platform.h"
41: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具。

### Lines 42-45
```cpp
42: #include "cutlass/numeric_conversion.h"
43: #include "cutlass/numeric_types.h"
44: #include "cutlass/matrix_shape.h"
45: 
```
**EN:** Pulls in required dependencies such as numeric types/converters, core CUTLASS utilities.
**CN:** 引入所需依赖，例如 数值类型/转换器、CUTLASS 基础工具。

### Lines 46-52
```cpp
46: #include "cutlass/arch/memory_sm75.h"
47: #include "cutlass/arch/mma_sm75.h"
48: #include "cutlass/arch/mma_sm80.h"
49: 
50: #include "cutlass/gemm/gemm.h"
51: #include "cutlass/gemm/warp/mma.h"
52: 
```
**EN:** Pulls in required dependencies such as architecture intrinsics, core CUTLASS utilities, warp components.
**CN:** 引入所需依赖，例如 架构内建/指令封装、CUTLASS 基础工具、warp 组件。

### Lines 53-62
```cpp
53: #include "cutlass/gemm/warp/mma_tensor_op_policy.h"
54: #include "cutlass/gemm/warp/mma_tensor_op.h"
55: #include "cutlass/gemm/warp/mma_tensor_op_tile_iterator.h"
56: #include "cutlass/gemm/warp/mma_tensor_op_tile_iterator_sm80.h"
57: 
58: /////////////////////////////////////////////////////////////////////////////////////////////////
59: 
60: namespace cutlass {
61: namespace gemm {
62: namespace warp {
```
**EN:** Pulls in required dependencies such as warp components.
**CN:** 引入所需依赖，例如 warp 组件。

### Lines 63-72
```cpp
63: 
64: /////////////////////////////////////////////////////////////////////////////////////////////////
65: 
66: /// Structure to compute the matrix product targeting CUDA cores and SIMT math instructions.
67: template <
68:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
69:   typename Shape_,
70:   /// Data type of A elements
71:   typename ElementA_,
72:   /// Layout of A matrix (concept: MatrixLayout)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 73-82
```cpp
73:   typename LayoutA_,
74:   /// Data type of B elements
75:   typename ElementB_,
76:   /// Layout of B matrix (concept: MatrixLayout)
77:   typename LayoutB_,
78:   /// Element type of C matrix
79:   typename ElementC_,
80:   /// Layout of C matrix (concept: MatrixLayout)
81:   typename LayoutC_,
82:   /// Policy describing warp-level MmaTensorOp (concept: MmaTensorOp policy)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 83-86
```cpp
83:   typename Policy_,
84:   /// Reduce operand A or B along K dimension
85:   bool ReduceKForA_,
86:   /// Number of partitions along K dimension
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 87-91
```cpp
87:   int PartitionsK_ = 1,
88:   /// Store the accumulators in row major or column major.  Row major is used
89:   /// when output layout is interleaved.
90:   bool AccumulatorsInRowMajor = false,
91:   /// Used for partial specialization
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 92-101
```cpp
92:   typename Enable = bool
93: >
94: class MmaWithReductionTensorOp {
95: public:
96:   /// Shape of warp-level matrix operation (concept: GemmShape)
97:   using Shape = Shape_;
98: 
99:   /// Data type of multiplicand A
100:   using ElementA = ElementA_;
101: 
```
**EN:** Defines MmaWithReductionTensorOp, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MmaWithReductionTensorOp，用于封装策略、存储或算法行为的辅助类型。

### Lines 102-111
```cpp
102:   /// Layout of multiplicand A
103:   using LayoutA = LayoutA_;
104: 
105:   /// Data type of multiplicand B
106:   using ElementB = ElementB_;
107: 
108:   /// Layout of multiplicand B
109:   using LayoutB = LayoutB_;
110: 
111:   /// Data type of accumulator matrix C
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 112-122
```cpp
112:   using ElementC = ElementC_;
113: 
114:   /// Layout of accumulator matrix C
115:   using LayoutC = LayoutC_;
116: 
117:   /// Shape of the warp in units of thread (concept: MmaLanePolicySimt)
118:   using Policy = Policy_;
119: 
120:   /// Underlying matrix multiply operator (concept: arch::Mma)
121:   using ArchMmaOperator = typename Policy::Operator;
122: 
```
**EN:** Introduces local type aliases (ElementC, LayoutC, Policy, ArchMmaOperator) to simplify downstream template code.
**CN:** 引入本地类型别名（ElementC, LayoutC, Policy, ArchMmaOperator），简化后续模板代码。

### Lines 123-132
```cpp
123:   /// Indicates math operator
124:   using MathOperator = typename ArchMmaOperator::Operator;
125: 
126:   /// Architecture tag from underlying instruction
127:   using ArchTag = typename ArchMmaOperator::ArchTag;
128: 
129:   /// Indicates class of matrix operator
130:   using OperatorClass = arch::OpClassTensorOp;
131: 
132:   /// Shape of underlying instruction
```
**EN:** Defines of, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 of，用于封装策略、存储或算法行为的辅助类型。

### Lines 133-143
```cpp
133:   using InstructionShape = typename ArchMmaOperator::Shape;
134: 
135:   /// Complex transform on A operand
136:   static ComplexTransform const kTransformA = ComplexTransform::kNone;
137: 
138:   /// Complex transform on B operand
139:   static ComplexTransform const kTransformB = ComplexTransform::kNone;
140: 
141:   /// Number of threads participating in warp-level matrix product
142:   static int const kThreadCount = 32;
143: 
```
**EN:** Introduces local type aliases (InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（InstructionShape），简化后续模板代码。

### Lines 144-148
```cpp
144:   /// Number of partitions along K dimension
145:   static int const kPartitionsK = PartitionsK_;
146: 
147:   static bool const kReduceKForA = ReduceKForA_;
148: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 149-152
```cpp
149:   static_assert(platform::is_same<ElementA, cutlass::half_t>::value ||
150:                 platform::is_same<ElementA, cutlass::bfloat16_t>::value,
151:                 "ElementA needs to be fp16 or bf16.");
152: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 153-156
```cpp
153:   static_assert(platform::is_same<ElementB, cutlass::half_t>::value ||
154:                 platform::is_same<ElementB, cutlass::bfloat16_t>::value,
155:                 "ElementB needs to be fp16 or bf16.");
156: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 157-160
```cpp
157:   static_assert(platform::is_same<InstructionShape,
158:                                   cutlass::gemm::GemmShape<16, 8, 16>>::value,
159:                 "Only supports 16x8x16 tensor core instruction.");
160: 
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 161-166
```cpp
161:   static_assert(!AccumulatorsInRowMajor,
162:                 "Only calls tensor core instructions in column major.");
163: 
164: public:
165: 
166:   /// Iterates over the A operand in memory
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 167-175
```cpp
167:   using IteratorA = MmaTensorOpMultiplicandTileIterator<
168:      MatrixShape<Shape::kM, Shape::kK>, Operand::kA, ElementA, LayoutA,
169:      MatrixShape<ArchMmaOperator::Shape::kM, ArchMmaOperator::Shape::kK>,
170:      Policy::OpDelta::kRow, kThreadCount, kPartitionsK>;
171: 
172:   /// Storage for A tile
173:   using FragmentA = typename IteratorA::Fragment;
174: 
175:   /// Storage for transformed A tile
```
**EN:** Introduces local type aliases (IteratorA, FragmentA) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorA, FragmentA），简化后续模板代码。

### Lines 176-179
```cpp
176:   using TransformedFragmentA =
177:       Array<typename ArchMmaOperator::ElementA, FragmentA::kElements>;
178: 
179:   /// Iterates over the B operand in memory
```
**EN:** Introduces local type aliases (TransformedFragmentA) to simplify downstream template code.
**CN:** 引入本地类型别名（TransformedFragmentA），简化后续模板代码。

### Lines 180-188
```cpp
180:   using IteratorB = MmaTensorOpMultiplicandTileIterator<
181:       MatrixShape<Shape::kK, Shape::kN>, Operand::kB, ElementB, LayoutB,
182:       MatrixShape<ArchMmaOperator::Shape::kK, ArchMmaOperator::Shape::kN>,
183:       Policy::OpDelta::kRow, kThreadCount, kPartitionsK>;
184: 
185:   /// Storage for B tile
186:   using FragmentB = typename IteratorB::Fragment;
187: 
188:   /// Storage for transformed B tile
```
**EN:** Introduces local type aliases (IteratorB, FragmentB) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorB, FragmentB），简化后续模板代码。

### Lines 189-192
```cpp
189:   using TransformedFragmentB =
190:       Array<typename ArchMmaOperator::ElementB, FragmentB::kElements>;
191: 
192:   /// Iterates over the C operand in memory
```
**EN:** Introduces local type aliases (TransformedFragmentB) to simplify downstream template code.
**CN:** 引入本地类型别名（TransformedFragmentB），简化后续模板代码。

### Lines 193-200
```cpp
193:   using IteratorC = MmaTensorOpAccumulatorTileIterator<
194:      MatrixShape<Shape::kM, Shape::kN>, ElementC, LayoutC,
195:      typename ArchMmaOperator::Shape, typename Policy::OpDelta>;
196: 
197:   /// Storage for C tile
198:   using FragmentC = typename IteratorC::Fragment;
199: 
200:   /// Number of mma operations performed
```
**EN:** Introduces local type aliases (IteratorC, FragmentC) to simplify downstream template code.
**CN:** 引入本地类型别名（IteratorC, FragmentC），简化后续模板代码。

### Lines 201-210
```cpp
201:   using MmaIterations = MatrixShape<
202:     (Shape::kM + ArchMmaOperator::Shape::kM - 1) / ArchMmaOperator::Shape::kM,
203:     (Shape::kN + ArchMmaOperator::Shape::kN - 1) / ArchMmaOperator::Shape::kN
204:   >;
205: 
206:   using FragmentReduction = Array<ElementC, kReduceKForA ? (Shape::kM / 8) : (Shape::kN / 8)>;
207: 
208: public:
209: 
210:   /// Underlying matrix multiply operator (concept: arch::Mma)
```
**EN:** Introduces local type aliases (MmaIterations, FragmentReduction) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaIterations, FragmentReduction），简化后续模板代码。

### Lines 211-214
```cpp
211:   ArchMmaOperator mma;
212: 
213: public:
214: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 215-219
```cpp
215:   //
216:   // Methods
217:   //
218: 
219:   /// Ctor
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 220-223
```cpp
220:   CUTLASS_DEVICE
221:   MmaWithReductionTensorOp() {}
222: 
223:   /// Performs a warp-level matrix multiply-accumulate operation
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 224-232
```cpp
224:   CUTLASS_DEVICE
225:   void operator()(
226:     FragmentC &D,
227:     TransformedFragmentA const &A,
228:     TransformedFragmentB const &B,
229:     FragmentC const &C,
230:     FragmentReduction &gemm_k_reduction
231:   ) const {
232: 
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 233-238
```cpp
233:     using MmaOperandA = typename ArchMmaOperator::FragmentA;
234:     using MmaOperandB = typename ArchMmaOperator::FragmentB;
235:     using MmaOperandC = typename ArchMmaOperator::FragmentC;
236: 
237:     D = C;
238: 
```
**EN:** Introduces local type aliases (MmaOperandA, MmaOperandB, MmaOperandC) to simplify downstream template code.
**CN:** 引入本地类型别名（MmaOperandA, MmaOperandB, MmaOperandC），简化后续模板代码。

### Lines 239-242
```cpp
239:     [[maybe_unused]] MmaOperandA const *ptr_A = reinterpret_cast<MmaOperandA const *>(&A);
240:     [[maybe_unused]] MmaOperandB const *ptr_B = reinterpret_cast<MmaOperandB const *>(&B);
241:     [[maybe_unused]] MmaOperandC *ptr_D = reinterpret_cast<MmaOperandC *>(&D);
242: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 243-246
```cpp
243:     #if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 800)
244:       assert(0);
245:     #elif defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 800)
246:       // Serpentine visitation order maximizing reuse of Ra
```
**EN:** Applies conditional logic to select architecture paths, handle bounds, or guard special cases.
**CN:** 应用条件逻辑以选择架构路径、处理边界或保护特殊情况。

### Lines 247-249
```cpp
247:       CUTLASS_PRAGMA_UNROLL
248:       for (int m = 0; m < MmaIterations::kRow; ++m) {
249: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 250-254
```cpp
250:         CUTLASS_PRAGMA_UNROLL
251:         for (int n = 0; n < MmaIterations::kColumn; ++n) {
252: 
253:           int n_serpentine = ((m % 2) ? (MmaIterations::kColumn - 1 - n) : n);
254: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 255-259
```cpp
255:           mma(ptr_D[m + n_serpentine * MmaIterations::kRow],
256:               ptr_A[m],
257:               ptr_B[n_serpentine],
258:               ptr_D[m + n_serpentine * MmaIterations::kRow]);
259: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 260-268
```cpp
260:           if (!kReduceKForA && m == 0) {
261:             #if 0
262:             gemm_k_reduction[n_serpentine] += float(B[n_serpentine * 4]);
263:             gemm_k_reduction[n_serpentine] += float(B[n_serpentine * 4 + 1]);
264:             gemm_k_reduction[n_serpentine] += float(B[n_serpentine * 4 + 2]);
265:             gemm_k_reduction[n_serpentine] += float(B[n_serpentine * 4 + 3]);
266:             #else
267:             uint32_t const *tmp = reinterpret_cast<uint32_t const *>(&B);
268: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 269-280
```cpp
269:             if (platform::is_same<ElementB, cutlass::half_t>::value) {
270:               asm volatile(
271:                 "{\n\t"
272:                 " .reg .f16 low, high;\n\t"
273:                 " .reg .f32 tmp;\n\t"
274:                 " mov.b32 {low, high}, %1;\n\t"
275:                 " cvt.f32.f16 tmp, low;\n\t"
276:                 " add.f32 %0, tmp, %0;\n\t"
277:                 " cvt.f32.f16 tmp, high;\n\t"
278:                 " add.f32 %0, tmp, %0;\n\t"
279:                 " mov.b32 {low, high}, %2;\n\t"
280:                 " cvt.f32.f16 tmp, low;\n\t"
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 281-287
```cpp
281:                 " add.f32 %0, tmp, %0;\n\t"
282:                 " cvt.f32.f16 tmp, high;\n\t"
283:                 " add.f32 %0, tmp, %0;\n\t"
284:                 "}\n\t"
285:                 : "+f"(gemm_k_reduction[n_serpentine])
286:                 : "r"(tmp[n_serpentine * 2]), "r"(tmp[n_serpentine * 2 + 1]));
287:             } else if (platform::is_same<ElementB, cutlass::bfloat16_t>::value) {
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 288-299
```cpp
288:               asm volatile(
289:                 "{\n\t"
290:                 " .reg .f32 tmp;\n\t"
291:                 " shl.b32 tmp, %1, 16;\n\t"
292:                 " add.f32 %0, tmp, %0;\n\t"
293:                 " and.b32 tmp, %1, 0xffff0000;\n\t"
294:                 " add.f32 %0, tmp, %0;\n\t"
295:                 " shl.b32 tmp, %2, 16;\n\t"
296:                 " add.f32 %0, tmp, %0;\n\t"
297:                 " and.b32 tmp, %2, 0xffff0000;\n\t"
298:                 " add.f32 %0, tmp, %0;\n\t"
299:                 "}\n\t"
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 300-307
```cpp
300:                 : "+f"(gemm_k_reduction[n_serpentine])
301:               : "r"(tmp[n_serpentine * 2]), "r"(tmp[n_serpentine * 2 + 1]));
302:             } else {
303:                 assert(0);
304:             }
305:             #endif
306:           }
307: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 308-314
```cpp
308:           if (kReduceKForA && (n == 0)) {
309:             #if 0
310:             gemm_k_reduction[m * 2] += float(A[m * 8]);
311:             gemm_k_reduction[m * 2] += float(A[m * 8 + 1]);
312:             gemm_k_reduction[m * 2] += float(A[m * 8 + 4]);
313:             gemm_k_reduction[m * 2] += float(A[m * 8 + 5]);
314: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 315-321
```cpp
315:             gemm_k_reduction[m * 2 + 1] += float(A[m * 8 + 2]);
316:             gemm_k_reduction[m * 2 + 1] += float(A[m * 8 + 3]);
317:             gemm_k_reduction[m * 2 + 1] += float(A[m * 8 + 6]);
318:             gemm_k_reduction[m * 2 + 1] += float(A[m * 8 + 7]);
319:             #else
320:             uint32_t const *tmp = reinterpret_cast<uint32_t const *>(&A);
321: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 322-333
```cpp
322:             if (platform::is_same<ElementA, cutlass::half_t>::value) {
323:               asm volatile(
324:                 "{\n\t"
325:                 " .reg .f16 low, high;\n\t"
326:                 " .reg .f32 tmp;\n\t"
327:                 " mov.b32 {low, high}, %2;\n\t"
328:                 " cvt.f32.f16 tmp, low;\n\t"
329:                 " add.f32 %0, tmp, %0;\n\t"
330:                 " cvt.f32.f16 tmp, high;\n\t"
331:                 " add.f32 %0, tmp, %0;\n\t"
332:                 " mov.b32 {low, high}, %3;\n\t"
333:                 " cvt.f32.f16 tmp, low;\n\t"
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 334-345
```cpp
334:                 " add.f32 %1, tmp, %1;\n\t"
335:                 " cvt.f32.f16 tmp, high;\n\t"
336:                 " add.f32 %1, tmp, %1;\n\t"
337:                 " mov.b32 {low, high}, %4;\n\t"
338:                 " cvt.f32.f16 tmp, low;\n\t"
339:                 " add.f32 %0, tmp, %0;\n\t"
340:                 " cvt.f32.f16 tmp, high;\n\t"
341:                 " add.f32 %0, tmp, %0;\n\t"
342:                 " mov.b32 {low, high}, %5;\n\t"
343:                 " cvt.f32.f16 tmp, low;\n\t"
344:                 " add.f32 %1, tmp, %1;\n\t"
345:                 " cvt.f32.f16 tmp, high;\n\t"
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 346-352
```cpp
346:                 " add.f32 %1, tmp, %1;\n\t"
347:                 "}\n\t"
348:                 : "+f"(gemm_k_reduction[m * 2]), "+f"(gemm_k_reduction[m * 2 + 1])
349:                 : "r"(tmp[m * 4]), "r"(tmp[m * 4 + 1]),"r"(tmp[m * 4 + 2]), "r"(tmp[m * 4 + 3]));
350: 
351:             } else if (platform::is_same<ElementA, cutlass::bfloat16_t>::value) {
352: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 353-364
```cpp
353:               asm volatile(
354:                 "{\n\t"
355:                 " .reg .f32 tmp;\n\t"
356:                 " shl.b32 tmp, %2, 16;\n\t"
357:                 " add.f32 %0, tmp, %0;\n\t"
358:                 " and.b32 tmp, %2, 0xffff0000;\n\t"
359:                 " add.f32 %0, tmp, %0;\n\t"
360:                 " shl.b32 tmp, %3, 16;\n\t"
361:                 " add.f32 %1, tmp, %1;\n\t"
362:                 " and.b32 tmp, %3, 0xffff0000;\n\t"
363:                 " add.f32 %1, tmp, %1;\n\t"
364:                 " shl.b32 tmp, %4, 16;\n\t"
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 365-375
```cpp
365:                 " add.f32 %0, tmp, %0;\n\t"
366:                 " and.b32 tmp, %4, 0xffff0000;\n\t"
367:                 " add.f32 %0, tmp, %0;\n\t"
368:                 " shl.b32 tmp, %5, 16;\n\t"
369:                 " add.f32 %1, tmp, %1;\n\t"
370:                 " and.b32 tmp, %5, 0xffff0000;\n\t"
371:                 " add.f32 %1, tmp, %1;\n\t"
372:                 "}\n\t"
373:                 : "+f"(gemm_k_reduction[m * 2]), "+f"(gemm_k_reduction[m * 2 + 1])
374:                 : "r"(tmp[m * 4]), "r"(tmp[m * 4 + 1]),"r"(tmp[m * 4 + 2]), "r"(tmp[m * 4 + 3]));
375: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 376-382
```cpp
376:             } else {
377:               assert(0);
378:             }
379:             #endif
380:           }
381:         }
382:       }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 383-388
```cpp
383:     #else
384:       assert(0);
385:     #endif
386:   }
387: 
388:   /// Transform the mma operands to the required types
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 389-392
```cpp
389:   CUTLASS_DEVICE
390:   void transform(TransformedFragmentA &dst_A, TransformedFragmentB &dst_B,
391:                  FragmentA const &A, FragmentB const &B) const {
392: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 393-395
```cpp
393:     //
394:     // Define conversions from source type to instruction type
395:     //
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 396-407
```cpp
396:     FloatRoundStyle const kRoundA =
397:         PreferredRoundingMode<typename ArchMmaOperator::ElementA,
398:                               ElementA>::kRound;
399:     FloatRoundStyle const kRoundB =
400:         PreferredRoundingMode<typename ArchMmaOperator::ElementB,
401:                               ElementB>::kRound;
402:     #if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 800)
403:       detail::ConvertAndPack<typename ArchMmaOperator::ElementA, ElementA,
404:                             FragmentA::kElements, kRoundA>
405:           convert_A;
406:       NumericArrayConverter<typename ArchMmaOperator::ElementB, ElementB,
407:                             FragmentB::kElements / 2, kRoundB>
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 408-416
```cpp
408:           convert_B;
409:       Array<ElementB, FragmentB::kElements / 2> const *ptr_B =
410:           reinterpret_cast<Array<ElementB, FragmentB::kElements / 2> const *>(&B);
411:       Array<typename ArchMmaOperator::ElementB, FragmentB::kElements / 2> *
412:           ptr_dst_B = reinterpret_cast<Array<typename ArchMmaOperator::ElementB,
413:                                              FragmentB::kElements / 2> *>(&dst_B);
414: 
415:       dst_A = convert_A(A);
416: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 417-419
```cpp
417:       ptr_dst_B[0] = convert_B(ptr_B[0]);
418:       ptr_dst_B[1] = convert_B(ptr_B[1]);
419: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 420-431
```cpp
420:     #elif defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 800)
421:       detail::ConvertAndPack<typename ArchMmaOperator::ElementA, ElementA,
422:                             FragmentA::kElements / 2, kRoundA>
423:           convert_A;
424:       NumericArrayConverter<typename ArchMmaOperator::ElementB, ElementB,
425:                             FragmentB::kElements, kRoundB>
426:           convert_B;
427:       Array<ElementA, FragmentA::kElements / 2> const *ptr_A =
428:           reinterpret_cast<Array<ElementA, FragmentA::kElements / 2> const *>(&A);
429:       Array<typename ArchMmaOperator::ElementA, FragmentA::kElements / 2> *
430:           ptr_dst_A = reinterpret_cast<Array<typename ArchMmaOperator::ElementA,
431:                                              FragmentA::kElements / 2> *>(&dst_A);
```
**EN:** Applies conditional logic to select architecture paths, handle bounds, or guard special cases.
**CN:** 应用条件逻辑以选择架构路径、处理边界或保护特殊情况。

### Lines 432-434
```cpp
432: 
433:       dst_B = convert_B(B);
434: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 435-444
```cpp
435:       ptr_dst_A[0] = convert_A(ptr_A[0]);
436:       ptr_dst_A[1] = convert_A(ptr_A[1]);
437:     #else
438:       assert(0);
439:     #endif
440:   }
441: };
442: 
443: /////////////////////////////////////////////////////////////////////////////////////////////////
444: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 445-449
```cpp
445: } // namespace warp
446: } // namespace gemm
447: } // namespace cutlass
448: 
449: /////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Warp-level fragment movement and MMA sequencing  
  **CN:** warp 级 fragment 搬运与 MMA 时序
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Tensor Core instruction mapping  
  **CN:** Tensor Core 指令映射
- **EN:** SIMT execution policy  
  **CN:** SIMT 执行策略
- **EN:** Tile iterator abstractions  
  **CN:** Tile 迭代器抽象
- **EN:** Complex-valued multiply-accumulate support  
  **CN:** 复数乘加支持

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `MmaWithReductionTensorOp`, `of`, `operator`, `transform`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
