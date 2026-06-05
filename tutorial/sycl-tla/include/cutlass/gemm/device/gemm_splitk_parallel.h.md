# gemm_splitk_parallel.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/device/gemm_splitk_parallel.h`
- **Purpose (EN):** Implements a device-level GEMM-family interface callable from host code.
- **用途 (CN):** 实现可由主机代码调用的设备级 GEMM 系列接口。

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
32:     \brief Template for GEMM performing a reduction over K partitions in parallel.
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
38: #include "cutlass/numeric_types.h"
39: #include "cutlass/arch/arch.h"
40: #include "cutlass/device_kernel.h"
41: 
42: #include "cutlass/gemm/threadblock/threadblock_swizzle.h"
43: #include "cutlass/gemm/kernel/gemm.h"
44: 
45: #include "cutlass/gemm/kernel/default_gemm_splitk_parallel.h"
46: #include "cutlass/gemm/device/default_gemm_configuration.h"
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, numeric types/converters, architecture intrinsics, device wrappers.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、数值类型/转换器、架构内建/指令封装、设备级封装。

### Lines 47-47
```cpp
47: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 48-57
```cpp
48: #include "cutlass/epilogue/thread/conversion_op.h"
49: #include "cutlass/reduction/kernel/reduce_split_k.h"
50: #include "cutlass/reduction/thread/reduction_operators.h"
51: 
52: ////////////////////////////////////////////////////////////////////////////////
53: 
54: namespace cutlass {
55: namespace gemm {
56: namespace device {
57: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, kernel adapters.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、kernel 适配层。

### Lines 58-59
```cpp
58: ////////////////////////////////////////////////////////////////////////////////
59: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 60-62
```cpp
60: /*! 
61:   Gemm device-level operator performing parallel reduction over the K partition.
62: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 63-73
```cpp
63: */
64: template <
65:     /// Element type for A matrix operand
66:     typename ElementA_,
67:     /// Layout type for A matrix operand
68:     typename LayoutA_,
69:     /// Element type for B matrix operand
70:     typename ElementB_,
71:     /// Layout type for B matrix operand
72:     typename LayoutB_,
73:     /// Element type for C and D matrix operands
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 74-79
```cpp
74:     typename ElementC_,
75:     /// Layout type for C and D matrix operands
76:     typename LayoutC_,
77:     /// Element type for internal accumulation
78:     typename ElementAccumulator_ = ElementC_,
79:     /// Operator class tag
```
**EN:** Defines tag, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tag，用于封装策略、存储或算法行为的辅助类型。

### Lines 80-85
```cpp
80:     typename OperatorClass_ = arch::OpClassSimt,
81:     /// Tag indicating architecture to tune for.  This is the minimum SM that
82:       /// supports the intended feature. The device kernel can be built
83:       /// targeting any SM larger than this number.
84:     typename ArchTag_ = arch::Sm70,
85:     /// Threadblock-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 86-89
```cpp
86:     typename ThreadblockShape_ = typename DefaultGemmConfiguration<
87:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
88:         ElementAccumulator_>::ThreadblockShape,
89:     /// Warp-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 90-93
```cpp
90:     typename WarpShape_ = typename DefaultGemmConfiguration<
91:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
92:         ElementAccumulator_>::WarpShape,
93:     /// Instruction-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 94-97
```cpp
94:     typename InstructionShape_ = typename DefaultGemmConfiguration<
95:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
96:         ElementAccumulator_>::InstructionShape,
97:     /// Epilogue output operator
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 98-101
```cpp
98:     typename EpilogueOutputOp_ = typename DefaultGemmConfiguration<
99:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
100:         ElementAccumulator_>::EpilogueOutputOp,
101:     /// Epilogue output operator
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 102-108
```cpp
102:     typename ConvertScaledOp_ = cutlass::epilogue::thread::Convert<
103:         ElementAccumulator_,
104:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
105:                                  ElementAccumulator_,
106:                                  ElementAccumulator_>::EpilogueOutputOp::kCount,
107:         ElementAccumulator_>,
108:     /// Reduction operator
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 109-112
```cpp
109:     typename ReductionOp_ = cutlass::reduction::thread::ReduceAdd<
110:         ElementAccumulator_, typename EpilogueOutputOp_::ElementAccumulator,
111:         EpilogueOutputOp_::kCount>,
112:     /// Threadblock-level swizzling operator
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 113-115
```cpp
113:     typename ThreadblockSwizzle_ =
114:         threadblock::GemmSplitKHorizontalThreadblockSwizzle,
115:     /// Number of stages used in the pipelined mainloop
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 116-119
```cpp
116:     int Stages =
117:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
118:                                  ElementC_, ElementAccumulator_>::kStages,
119:     /// Access granularity of A matrix in units of elements
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 120-123
```cpp
120:     int kAlignmentA =
121:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
122:                                  ElementC_, ElementAccumulator_>::kAlignmentA,
123:     /// Access granularity of B matrix in units of elements
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 124-127
```cpp
124:     int kAlignmentB =
125:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
126:                                  ElementC_, ElementAccumulator_>::kAlignmentB,
127:     /// Operation performed by GEMM
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 128-133
```cpp
128:     typename Operator_ = typename DefaultGemmConfiguration<
129:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
130:         ElementAccumulator_>::Operator>
131: class GemmSplitKParallel {
132:  public:
133: 
```
**EN:** Defines GemmSplitKParallel, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 GemmSplitKParallel，用于封装策略、存储或算法行为的辅助类型。

### Lines 134-145
```cpp
134:   using ElementA = ElementA_;
135:   using LayoutA = LayoutA_;
136:   using ElementB = ElementB_;
137:   using LayoutB = LayoutB_;
138:   using ElementC = ElementC_;
139:   using LayoutC = LayoutC_;
140:   using ElementAccumulator = ElementAccumulator_;
141:   using OperatorClass = OperatorClass_;
142:   using ArchTag = ArchTag_;
143:   using ThreadblockShape = ThreadblockShape_;
144:   using WarpShape = WarpShape_;
145:   using InstructionShape = InstructionShape_;
```
**EN:** Introduces local type aliases (ElementA, LayoutA, ElementB, LayoutB) to simplify downstream template code.
**CN:** 引入本地类型别名（ElementA, LayoutA, ElementB, LayoutB），简化后续模板代码。

### Lines 146-153
```cpp
146:   using ConvertScaledOp = ConvertScaledOp_;
147:   using EpilogueOutputOp = EpilogueOutputOp_;
148:   using ReductionOp = ReductionOp_;
149:   using ThreadblockSwizzle = ThreadblockSwizzle_;
150:   using Operator = Operator_;
151:   static int const kStages = Stages;
152: 
153:   /// GEMM kernel 
```
**EN:** Introduces local type aliases (ConvertScaledOp, EpilogueOutputOp, ReductionOp, ThreadblockSwizzle) to simplify downstream template code.
**CN:** 引入本地类型别名（ConvertScaledOp, EpilogueOutputOp, ReductionOp, ThreadblockSwizzle），简化后续模板代码。

### Lines 154-165
```cpp
154:   using GemmKernel = typename kernel::DefaultGemmSplitKParallel<
155:     ElementA,
156:     LayoutA,
157:     kAlignmentA,
158:     ElementB,
159:     LayoutB,
160:     kAlignmentB,
161:     ElementAccumulator,
162:     LayoutC,
163:     ElementAccumulator,
164:     OperatorClass,
165:     ArchTag,
```
**EN:** Introduces local type aliases (GemmKernel) to simplify downstream template code.
**CN:** 引入本地类型别名（GemmKernel），简化后续模板代码。

### Lines 166-175
```cpp
166:     ThreadblockShape,
167:     WarpShape,
168:     InstructionShape,
169:     ConvertScaledOp,
170:     ThreadblockSwizzle,
171:     kStages,
172:     Operator
173:   >::GemmKernel;
174: 
175:   /// Reduction kernel
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 176-181
```cpp
176:   using ReductionKernel = cutlass::reduction::kernel::ReduceSplitK<
177:     cutlass::MatrixShape<4, 32 * EpilogueOutputOp::kCount>,
178:     EpilogueOutputOp,
179:     ReductionOp
180:   >;
181: 
```
**EN:** Introduces local type aliases (ReductionKernel) to simplify downstream template code.
**CN:** 引入本地类型别名（ReductionKernel），简化后续模板代码。

### Lines 182-188
```cpp
182:   //
183:   //
184:   //
185: 
186:   /// Argument structure
187:   struct Arguments {
188: 
```
**EN:** Defines Arguments, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Arguments，用于封装策略、存储或算法行为的辅助类型。

### Lines 189-192
```cpp
189:     //
190:     // Data members
191:     //
192: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 193-202
```cpp
193:     GemmCoord problem_size;
194:     TensorRef<ElementA const, LayoutA> ref_A;
195:     TensorRef<ElementB const, LayoutB> ref_B;
196:     TensorRef<ElementC const, LayoutC> ref_C;
197:     TensorRef<ElementC, LayoutC> ref_D;
198:     typename EpilogueOutputOp::Params epilogue;
199:     int split_k_slices;
200:     typename ConvertScaledOp::Params convert;
201:     typename ReductionOp::Params reduction;
202: 
```
**EN:** This block focuses on tensor, epilogue related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、epilogue 输出阶段 的实现细节。

### Lines 203-207
```cpp
203:     //
204:     // Methods
205:     //
206: 
207:     /// Default ctor
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 208-211
```cpp
208:     CUTLASS_HOST_DEVICE
209:     Arguments() { }
210: 
211:     /// Constructs an Arguments structure 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 212-223
```cpp
212:     CUTLASS_HOST_DEVICE
213:     Arguments(
214:       GemmCoord problem_size_,
215:       TensorRef<ElementA const, LayoutA> ref_A_,
216:       TensorRef<ElementB const, LayoutB> ref_B_,
217:       TensorRef<ElementC const, LayoutC> ref_C_,
218:       TensorRef<ElementC, LayoutC> ref_D_,
219:       typename EpilogueOutputOp::Params epilogue_ = 
220:         typename EpilogueOutputOp::Params(),
221:       int split_k_slices = 1,
222:       typename ConvertScaledOp::Params convert_ = 
223:         typename ConvertScaledOp::Params(),
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 224-235
```cpp
224:       typename ReductionOp::Params reduction_ =
225:         typename ReductionOp::Params()
226:     ):
227:       problem_size(problem_size_),
228:       ref_A(ref_A_),
229:       ref_B(ref_B_),
230:       ref_C(ref_C_),
231:       ref_D(ref_D_),
232:       epilogue(epilogue_),
233:       split_k_slices(split_k_slices),
234:       convert(convert_),
235:       reduction(reduction_) { }
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 236-245
```cpp
236:   };
237: 
238: private:
239: 
240:   /// Kernel parameters object
241:   typename GemmKernel::Params gemm_params_;
242: 
243:   /// Reduction kernel parameters object
244:   typename ReductionKernel::Params reduction_params_;
245: 
```
**EN:** Closes the current type or namespace scope.
**CN:** 结束当前类型或命名空间作用域。

### Lines 246-251
```cpp
246: public:
247: 
248:   /// Constructs the GEMM.
249:   GemmSplitKParallel() { }
250: 
251:   /// Determines whether the GEMM can execute the given problem.
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 252-261
```cpp
252:   static Status can_implement(Arguments const &args) {
253:     return Status::kSuccess;
254:   }
255: 
256:   /// Gets the workspace size
257:   static size_t get_workspace_size(Arguments const &args) {
258:     
259:     // Determine grid shape
260:     ThreadblockSwizzle threadblock_swizzle;
261: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 262-272
```cpp
262:     cutlass::gemm::GemmCoord grid_shape = threadblock_swizzle.get_tiled_shape(
263:       args.problem_size, 
264:       {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
265:       args.split_k_slices);
266: 
267:     return sizeof(ElementAccumulator_) * size_t(args.problem_size.m()) * size_t(args.problem_size.n()) * grid_shape.k();
268:   }
269: 
270:   /// Initializes GEMM state from arguments.
271:   Status initialize(Arguments const &args, void *workspace) {
272: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 273-275
```cpp
273:     // Determine grid shape
274:     ThreadblockSwizzle threadblock_swizzle;
275: 
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 276-281
```cpp
276:     cutlass::gemm::GemmCoord grid_shape = threadblock_swizzle.get_tiled_shape(
277:       args.problem_size, 
278:       {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
279:       args.split_k_slices);
280: 
281:     // Define a reference to the workspace - this is an aligned region in device memory.
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 282-285
```cpp
282:     if (!workspace) {
283:       return Status::kErrorWorkspaceNull;
284:     }
285:     
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 286-292
```cpp
286:     TensorRef<ElementAccumulator_, layout::RowMajor> ref_workspace(
287:       static_cast<ElementAccumulator_ *>(workspace), 
288:       args.problem_size.n());
289: 
290:     int64_t partition_stride = int64_t(args.problem_size.m()) * int64_t(args.problem_size.n());
291: 
292:     // Initialize the Params structure
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 293-302
```cpp
293:     gemm_params_ = typename GemmKernel::Params{
294:       args.problem_size,
295:       grid_shape,
296:       args.ref_A.non_const_ref(),
297:       args.ref_B.non_const_ref(),
298:       ref_workspace,
299:       args.convert,
300:       partition_stride
301:     };
302: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 303-312
```cpp
303:     reduction_params_ = typename ReductionKernel::Params(
304:       args.problem_size.mn(),
305:       grid_shape.k(),
306:       partition_stride,
307:       ref_workspace,
308:       args.ref_D,
309:       args.ref_C.non_const_ref(),
310:       args.epilogue
311:     );
312: 
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 313-318
```cpp
313:     return Status::kSuccess;
314:   }
315: 
316:   /// Lightweight update given a subset of arguments
317:   Status update(Arguments const &args, void *workspace = nullptr) {
318: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 319-322
```cpp
319:     if (!workspace) {
320:       return Status::kErrorWorkspaceNull;
321:     }
322: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 323-326
```cpp
323:     gemm_params_.ref_A.reset(args.ref_A.data());
324:     gemm_params_.ref_B.reset(args.ref_B.data());
325:     gemm_params_.ref_D.reset(workspace);     
326: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 327-335
```cpp
327:     reduction_params_.ref_D.reset(args.ref_D.data());
328:     reduction_params_.ref_C.reset(args.ref_C.data());
329: 
330:     return Status::kSuccess;
331:   }
332: 
333:   /// Runs the kernel using initialized state.
334:   Status run(cudaStream_t stream = nullptr) {
335: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 336-341
```cpp
336:     //
337:     // Launch GEMM kernel
338:     //
339: 
340:     ThreadblockSwizzle threadblock_swizzle;
341: 
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 342-346
```cpp
342:     dim3 grid = threadblock_swizzle.get_grid_shape(gemm_params_.grid_tiled_shape);
343:     dim3 block(GemmKernel::kThreadCount, 1, 1);
344: 
345:     cudaError_t result;
346: 
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 347-349
```cpp
347:     int smem_size = int(sizeof(typename GemmKernel::SharedStorage));
348:     if (smem_size >= (48 << 10)) {
349: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 350-354
```cpp
350:       result = cudaFuncSetAttribute(
351:         Kernel<GemmKernel>,
352:         cudaFuncAttributeMaxDynamicSharedMemorySize,
353:         smem_size);
354: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 355-359
```cpp
355:       if (result != cudaSuccess) {
356:         return Status::kErrorInternal;
357:       }
358:     }
359: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 360-364
```cpp
360:     cutlass::arch::synclog_setup();
361:     Kernel<GemmKernel><<<grid, block, smem_size, stream>>>(gemm_params_);
362: 
363:     result = cudaGetLastError();
364: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 365-368
```cpp
365:     if (result != cudaSuccess) {
366:       return Status::kErrorInternal;
367:     }
368: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 369-372
```cpp
369:     //
370:     // Launch reduction kernel
371:     //
372: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 373-379
```cpp
373:     block = ReductionKernel::block_shape();
374:     grid = ReductionKernel::grid_shape(gemm_params_.problem_size.mn());
375: 
376:     Kernel<ReductionKernel><<< grid, block, 0, stream >>>(reduction_params_);
377: 
378:     result = cudaGetLastError();
379: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 380-387
```cpp
380:     if (result != cudaSuccess) {
381:       return Status::kErrorInternal;
382:     }
383: 
384:     return result == cudaSuccess ? Status::kSuccess : Status::kErrorInternal;
385:   }
386: 
387:   /// Runs the kernel using initialized state.
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 388-392
```cpp
388:   Status operator()(cudaStream_t stream = nullptr) {
389:     return run(stream);
390:   }
391: 
392:   /// Runs the kernel using initialized state.
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 393-399
```cpp
393:   Status operator()(
394:     Arguments const &args, 
395:     void *workspace = nullptr, 
396:     cudaStream_t stream = nullptr) {
397:     
398:     Status status = initialize(args, workspace);
399:     
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 400-409
```cpp
400:     if (status == Status::kSuccess) {
401:       status = run(stream);
402:     }
403: 
404:     return status;
405:   }
406: };
407: 
408: ////////////////////////////////////////////////////////////////////////////////
409: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 410-420
```cpp
410: /// Partial specialization for column-major output
411: template <
412:     /// Element type for A matrix operand
413:     typename ElementA_,
414:     /// Layout type for A matrix operand
415:     typename LayoutA_,
416:     /// Element type for B matrix operand
417:     typename ElementB_,
418:     /// Layout type for B matrix operand
419:     typename LayoutB_,
420:     /// Element type for C and D matrix operands
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 421-424
```cpp
421:     typename ElementC_,
422:     /// Element type for internal accumulation
423:     typename ElementAccumulator_,
424:     /// Operator class tag
```
**EN:** Defines tag, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tag，用于封装策略、存储或算法行为的辅助类型。

### Lines 425-434
```cpp
425:     typename OperatorClass_,
426:     /// Tag indicating architecture to tune for.  This is the minimum SM that
427:       /// supports the intended feature. The device kernel can be built
428:       /// targeting any SM larger than this number.
429:     typename ArchTag_,
430:     /// Threadblock-level tile size (concept: GemmShape)
431:     typename ThreadblockShape_,
432:     /// Warp-level tile size (concept: GemmShape)
433:     typename WarpShape_,
434:     /// Instruction-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 435-444
```cpp
435:     typename InstructionShape_,
436:     /// Epilogue output operator
437:     typename EpilogueOutputOp_,
438:     /// Epilogue output operator
439:     typename ConvertScaledOp_,
440:     /// Reduction operator
441:     typename ReductionOp_,
442:     /// Threadblock-level swizzling operator
443:     typename ThreadblockSwizzle_,
444:     /// Number of stages used in the pipelined mainloop
```
**EN:** This block focuses on swizzle, epilogue related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射、epilogue 输出阶段 的实现细节。

### Lines 445-446
```cpp
445:     int Stages, int kAlignmentA, int kAlignmentB,
446:     /// Operation performed by GEMM
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 447-455
```cpp
447:     typename Operator_>
448: class GemmSplitKParallel<ElementA_, LayoutA_, ElementB_, LayoutB_, ElementC_,
449:                          layout::ColumnMajor, ElementAccumulator_,
450:                          OperatorClass_, ArchTag_, ThreadblockShape_,
451:                          WarpShape_, InstructionShape_, EpilogueOutputOp_,
452:                          ConvertScaledOp_, ReductionOp_, ThreadblockSwizzle_,
453:                          Stages, kAlignmentA, kAlignmentB, Operator_> {
454:  public:
455: 
```
**EN:** Defines GemmSplitKParallel, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 GemmSplitKParallel，用于封装策略、存储或算法行为的辅助类型。

### Lines 456-467
```cpp
456:   using ElementA = ElementA_;
457:   using LayoutA = LayoutA_;
458:   using ElementB = ElementB_;
459:   using LayoutB = LayoutB_;
460:   using ElementC = ElementC_;
461:   using LayoutC = layout::ColumnMajor;
462:   using ElementAccumulator = ElementAccumulator_;
463:   using OperatorClass = OperatorClass_;
464:   using ArchTag = ArchTag_;
465:   using ThreadblockShape = ThreadblockShape_;
466:   using WarpShape = WarpShape_;
467:   using InstructionShape = InstructionShape_;
```
**EN:** Introduces local type aliases (ElementA, LayoutA, ElementB, LayoutB) to simplify downstream template code.
**CN:** 引入本地类型别名（ElementA, LayoutA, ElementB, LayoutB），简化后续模板代码。

### Lines 468-474
```cpp
468:   using ConvertScaledOp = ConvertScaledOp_;
469:   using EpilogueOutputOp = EpilogueOutputOp_;
470:   using ReductionOp = ReductionOp_;
471:   using ThreadblockSwizzle = ThreadblockSwizzle_;
472:   using Operator = Operator_;
473:   static int const kStages = Stages;
474: 
```
**EN:** Introduces local type aliases (ConvertScaledOp, EpilogueOutputOp, ReductionOp, ThreadblockSwizzle) to simplify downstream template code.
**CN:** 引入本地类型别名（ConvertScaledOp, EpilogueOutputOp, ReductionOp, ThreadblockSwizzle），简化后续模板代码。

### Lines 475-486
```cpp
475:   using UnderlyingOperator = GemmSplitKParallel< 
476:     ElementB,
477:     typename layout::LayoutTranspose<LayoutB>::type,
478:     ElementA,
479:     typename layout::LayoutTranspose<LayoutA>::type,
480:     ElementC,
481:     layout::RowMajor,    
482:     ElementAccumulator,
483:     OperatorClass,
484:     ArchTag,
485:     ThreadblockShape,
486:     WarpShape,
```
**EN:** Introduces local type aliases (UnderlyingOperator) to simplify downstream template code.
**CN:** 引入本地类型别名（UnderlyingOperator），简化后续模板代码。

### Lines 487-497
```cpp
487:     InstructionShape,
488:     EpilogueOutputOp,
489:     ConvertScaledOp,
490:     ReductionOp,
491:     ThreadblockSwizzle,
492:     Stages,
493:     kAlignmentA,
494:     kAlignmentB,
495:     Operator
496:   >;
497: 
```
**EN:** This block focuses on swizzle, epilogue related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射、epilogue 输出阶段 的实现细节。

### Lines 498-504
```cpp
498:   using UnderlyingArguments = typename UnderlyingOperator::Arguments;
499:   using GemmKernel = typename UnderlyingOperator::GemmKernel;
500:   using ReductionKernel = typename UnderlyingOperator::ReductionKernel;
501: 
502:   /// Argument structure
503:   struct Arguments {
504: 
```
**EN:** Defines Arguments, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Arguments，用于封装策略、存储或算法行为的辅助类型。

### Lines 505-508
```cpp
505:     //
506:     // Data members
507:     //
508: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 509-518
```cpp
509:     GemmCoord problem_size;
510:     TensorRef<ElementA const, LayoutA> ref_A;
511:     TensorRef<ElementB const, LayoutB> ref_B;
512:     TensorRef<ElementC const, LayoutC> ref_C;
513:     TensorRef<ElementC, LayoutC> ref_D;
514:     typename EpilogueOutputOp::Params epilogue;
515:     int split_k_slices;
516:     typename ConvertScaledOp::Params convert;
517:     typename ReductionOp::Params reduction;
518: 
```
**EN:** This block focuses on tensor, epilogue related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子、epilogue 输出阶段 的实现细节。

### Lines 519-523
```cpp
519:     //
520:     // Methods
521:     //
522: 
523:     /// Default ctor
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 524-527
```cpp
524:     CUTLASS_HOST_DEVICE
525:     Arguments() { }
526: 
527:     /// Constructs an Arguments structure 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 528-539
```cpp
528:     CUTLASS_HOST_DEVICE
529:     Arguments(
530:       GemmCoord problem_size_,
531:       TensorRef<ElementA const, LayoutA> ref_A_,
532:       TensorRef<ElementB const, LayoutB> ref_B_,
533:       TensorRef<ElementC const, LayoutC> ref_C_,
534:       TensorRef<ElementC, LayoutC> ref_D_,
535:       typename EpilogueOutputOp::Params epilogue_ = 
536:         typename EpilogueOutputOp::Params(),
537:       int split_k_slices = 1,
538:       typename ConvertScaledOp::Params convert_ = 
539:         typename ConvertScaledOp::Params(),
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 540-551
```cpp
540:       typename ReductionOp::Params reduction_ =
541:         typename ReductionOp::Params()
542:     ):
543:       problem_size(problem_size_),
544:       ref_A(ref_A_),
545:       ref_B(ref_B_),
546:       ref_C(ref_C_),
547:       ref_D(ref_D_),
548:       epilogue(epilogue_),
549:       split_k_slices(split_k_slices),
550:       convert(convert_),
551:       reduction(reduction_) { }
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 552-561
```cpp
552:   };
553: 
554: private:
555: 
556:   /// Kernel parameters object
557:   UnderlyingOperator underlying_operator_;
558: 
559: public:
560: 
561:   /// Constructs the GEMM.
```
**EN:** Closes the current type or namespace scope.
**CN:** 结束当前类型或命名空间作用域。

### Lines 562-564
```cpp
562:   GemmSplitKParallel() { }
563: 
564:   /// Helper to construct a transposed equivalent for the underlying GEMM operator
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 565-576
```cpp
565:   static UnderlyingArguments to_underlying_arguments(Arguments const &args) {
566:     return UnderlyingArguments(
567:       {args.problem_size.n(), args.problem_size.m(), args.problem_size.k()},
568:       {args.ref_B.data(), args.ref_B.stride(0)},
569:       {args.ref_A.data(), args.ref_A.stride(0)},
570:       {args.ref_C.data(), args.ref_C.stride(0)},
571:       {args.ref_D.data(), args.ref_D.stride(0)},
572:       args.epilogue,
573:       args.split_k_slices,
574:       args.convert,
575:       args.reduction
576:     );
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 577-587
```cpp
577:   }
578: 
579:   /// Determines whether the GEMM can execute the given problem.
580:   static Status can_implement(Arguments const &args) {
581: 
582:     return UnderlyingOperator::can_implement(to_underlying_arguments(args));
583:   }
584: 
585:   /// Gets the workspace size
586:   static size_t get_workspace_size(Arguments const &args) {
587:     
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 588-597
```cpp
588:     return UnderlyingOperator::get_workspace_size(to_underlying_arguments(args));
589:   }
590: 
591:   /// Initializes GEMM state from arguments.
592:   Status initialize(Arguments const &args, void *workspace) {
593: 
594:     return underlying_operator_.initialize(to_underlying_arguments(args), workspace);
595:   }
596: 
597:   /// Lightweight update given a subset of arguments
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 598-607
```cpp
598:   Status update(Arguments const &args, void *workspace = nullptr) {
599: 
600:     return underlying_operator_.update(to_underlying_arguments(args), workspace);
601:   }
602: 
603:   /// Runs the kernel using initialized state.
604:   Status run(cudaStream_t stream = nullptr) {
605: 
606:     return underlying_operator_.run(stream);
607:   }
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 608-609
```cpp
608: 
609:   /// Runs the kernel using initialized state.
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 610-614
```cpp
610:   Status operator()(cudaStream_t stream = nullptr) {
611:     return run(stream);
612:   }
613: 
614:   /// Runs the kernel using initialized state.
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 615-621
```cpp
615:   Status operator()(
616:     Arguments const &args, 
617:     void *workspace = nullptr, 
618:     cudaStream_t stream = nullptr) {
619:     
620:     Status status = initialize(args, workspace, stream);
621:     
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 622-631
```cpp
622:     if (status == Status::kSuccess) {
623:       status = run(stream);
624:     }
625: 
626:     return status;
627:   }
628: };
629: 
630: ////////////////////////////////////////////////////////////////////////////////
631: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 632-636
```cpp
632: } // namespace device
633: } // namespace gemm
634: } // namespace cutlass
635: 
636: ////////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Host-facing device operator wrappers  
  **CN:** 面向主机的设备算子封装
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** SIMT execution policy  
  **CN:** SIMT 执行策略
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线
- **EN:** Epilogue/output transformation  
  **CN:** Epilogue/输出变换
- **EN:** Threadblock swizzle mapping  
  **CN:** 线程块 swizzle 映射

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `tag`, `GemmSplitKParallel`, `Arguments`, `can_implement`, `get_workspace_size`, `initialize`, `update`, `run`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
