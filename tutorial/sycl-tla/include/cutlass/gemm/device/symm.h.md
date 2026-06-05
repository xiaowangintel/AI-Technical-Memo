# symm.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/device/symm.h`
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
32:     \brief Template for a pipelined SYMM and HEMM kernels. Does not compute batching or support split-K.
33: 
34:   
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 35-38
```cpp
35: */
36: 
37: #pragma once
38: 
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 39-48
```cpp
39: #include "cutlass/blas3.h"
40: #include "cutlass/arch/arch.h"
41: #include "cutlass/device_kernel.h"
42: 
43: #include "cutlass/gemm/threadblock/threadblock_swizzle.h"
44: #include "cutlass/gemm/kernel/symm_universal.h"
45: 
46: #include "cutlass/gemm/kernel/default_symm_universal.h"
47: #include "cutlass/gemm/device/default_gemm_configuration.h"
48: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, architecture intrinsics, device wrappers, threadblock components.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、架构内建/指令封装、设备级封装、线程块组件。

### Lines 49-58
```cpp
49: ////////////////////////////////////////////////////////////////////////////////
50: 
51: namespace cutlass {
52: namespace gemm {
53: namespace device {
54: 
55: /////////////////////////////////////////////////////////////////////////////////////////////////
56: 
57: template <
58:     /// Element type for A matrix operand
```
**EN:** Enters namespace scope (cutlass::gemm::device) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass::gemm::device），组织 GEMM 抽象层。

### Lines 59-68
```cpp
59:     typename ElementA_,
60:     /// Layout type for A matrix operand
61:     typename LayoutA_,
62:     /// Side Mode for A (kLeft or kRight)
63:     SideMode SideModeA,
64:     /// Fill Mode for A (kLower or kUpper)
65:     FillMode FillModeA,
66:     /// Element type for B matrix operand
67:     typename ElementB_,
68:     /// Layout type for B matrix operand
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 69-78
```cpp
69:     typename LayoutB_,
70:     /// Element type for C and D matrix operands
71:     typename ElementC_,
72:     /// Layout type for C and D matrix operands
73:     typename LayoutC_,
74:     /// Element type for internal accumulation
75:     typename ElementAccumulator_ = ElementC_,
76:     /// Operator class tag
77:     typename OperatorClass_ = arch::OpClassTensorOp,
78:     /// Tag indicating architecture to tune for
```
**EN:** Defines tag, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tag，用于封装策略、存储或算法行为的辅助类型。

### Lines 79-80
```cpp
79:     typename ArchTag_ = arch::Sm80,
80:     /// Threadblock-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 81-84
```cpp
81:     typename ThreadblockShape_ = typename DefaultGemmConfiguration<
82:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
83:         ElementAccumulator_>::ThreadblockShape,
84:     /// Warp-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 85-88
```cpp
85:     typename WarpShape_ = typename DefaultGemmConfiguration<
86:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
87:         ElementAccumulator_>::WarpShape,
88:     /// Instruction-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 89-92
```cpp
89:     typename InstructionShape_ = typename DefaultGemmConfiguration<
90:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
91:         ElementAccumulator_>::InstructionShape,
92:     /// Epilogue output operator
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 93-102
```cpp
93:     typename EpilogueOutputOp_ = epilogue::thread::LinearCombination<
94:       ElementC_,
95:       128 / sizeof_bits<ElementC_>::value,
96:       ElementAccumulator_,
97:       ElementAccumulator_,
98:       epilogue::thread::ScaleType::OnlyAlphaScaling
99:     >,
100:     /// Threadblock-level swizzling operator
101:     typename ThreadblockSwizzle_ = threadblock::GemmIdentityThreadblockSwizzle<>,
102:     /// Number of stages used in the pipelined mainloop
```
**EN:** This block focuses on swizzle, epilogue related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射、epilogue 输出阶段 的实现细节。

### Lines 103-106
```cpp
103:     int Stages =
104:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
105:                                  ElementC_, ElementAccumulator_>::kStages,
106:     /// Access granularity of A matrix in units of elements
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 107-110
```cpp
107:     int AlignmentA =
108:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
109:                                  ElementC_, ElementAccumulator_>::kAlignmentA,
110:     /// Access granularity of B matrix in units of elements
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 111-116
```cpp
111:     int AlignmentB =
112:         DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
113:                                  ElementC_, ElementAccumulator_>::kAlignmentB,
114:     /// If true, kernel supports split-K with serial reduction
115:     bool SplitKSerial = false,
116:     /// Operation performed by SYMM
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 117-120
```cpp
117:     typename Operator_ = typename DefaultGemmConfiguration<
118:         OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
119:         ElementAccumulator_>::Operator,
120:     /// Blas3 computation mode (symmetric/hermitian)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 121-124
```cpp
121:     BlasMode BlasMode_ = BlasMode::kSymmetric>
122: class Symm {
123:  public:
124: 
```
**EN:** Defines Symm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Symm，用于封装策略、存储或算法行为的辅助类型。

### Lines 125-136
```cpp
125:   using ElementA = ElementA_;
126:   using LayoutA = LayoutA_;
127:   using ElementAKernel = typename platform::conditional<(SideModeA == SideMode::kRight), ElementB_, ElementA_>::type;
128:   using LayoutAKernel = typename platform::conditional<(SideModeA == SideMode::kRight), LayoutB_, LayoutA_>::type;
129:   using ElementB = ElementB_;
130:   using LayoutB = LayoutB_;
131:   using ElementBKernel = typename platform::conditional<(SideModeA == SideMode::kRight), ElementA_, ElementB_>::type;
132:   using LayoutBKernel = typename platform::conditional<(SideModeA == SideMode::kRight), LayoutA_, LayoutB_>::type;
133:   using ElementC = ElementC_;
134:   using LayoutC = LayoutC_;
135:   using ElementAccumulator = ElementAccumulator_;
136:   using OperatorClass = OperatorClass_;
```
**EN:** Introduces local type aliases (ElementA, LayoutA, ElementAKernel, LayoutAKernel) to simplify downstream template code.
**CN:** 引入本地类型别名（ElementA, LayoutA, ElementAKernel, LayoutAKernel），简化后续模板代码。

### Lines 137-148
```cpp
137:   using ArchTag = ArchTag_;
138:   using ThreadblockShape = ThreadblockShape_;
139:   using WarpShape = WarpShape_;
140:   using InstructionShape = InstructionShape_;
141:   using EpilogueOutputOp = EpilogueOutputOp_;
142:   using ThreadblockSwizzle = ThreadblockSwizzle_;
143:   using Operator = Operator_;
144:   static SideMode const kSideModeA = SideModeA;
145:   static FillMode const kFillModeA = FillModeA;
146:   static int const kStages = Stages;
147:   static int const kAlignmentA = AlignmentA;
148:   static int const kAlignmentAKernel = (SideModeA == SideMode::kRight) ? AlignmentB : AlignmentA;
```
**EN:** Introduces local type aliases (ArchTag, ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ArchTag, ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 149-155
```cpp
149:   static int const kAlignmentB = AlignmentB;
150:   static int const kAlignmentBKernel = (SideModeA == SideMode::kRight) ? AlignmentA : AlignmentB;
151:   static int const kAlignmentC = EpilogueOutputOp::kCount;
152:   static bool const kSplitKSerial = SplitKSerial;
153:   static BlasMode const kBlasMode = BlasMode_;
154: 
155:   // static asserts for symm update kernel
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 156-159
```cpp
156:   static_assert(platform::is_same<LayoutA, LayoutB>::value,
157:     "SYMM update operator support same layouts for operand A and B");
158: 
159:   /// Define the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 160-171
```cpp
160:   using SymmKernel = typename kernel::DefaultSymmUniversal<
161:     ElementAKernel,
162:     LayoutAKernel,
163:     kSideModeA,
164:     kFillModeA,
165:     kAlignmentAKernel,
166:     ElementBKernel,
167:     LayoutBKernel,
168:     kAlignmentBKernel,
169:     ElementC,
170:     LayoutC,
171:     ElementAccumulator,
```
**EN:** Introduces local type aliases (SymmKernel) to simplify downstream template code.
**CN:** 引入本地类型别名（SymmKernel），简化后续模板代码。

### Lines 172-183
```cpp
172:     OperatorClass,
173:     ArchTag,
174:     ThreadblockShape,
175:     WarpShape,
176:     InstructionShape,
177:     EpilogueOutputOp,
178:     ThreadblockSwizzle,
179:     kStages,
180:     kSplitKSerial,
181:     Operator,
182:     kBlasMode
183:   >::SymmKernel;
```
**EN:** This block focuses on swizzle, epilogue related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射、epilogue 输出阶段 的实现细节。

### Lines 184-189
```cpp
184:   
185:   using Arguments = typename SymmKernel::Arguments;
186: 
187: private:
188: 
189:   /// Kernel parameters object
```
**EN:** Introduces local type aliases (Arguments) to simplify downstream template code.
**CN:** 引入本地类型别名（Arguments），简化后续模板代码。

### Lines 190-198
```cpp
190:   typename SymmKernel::Params params_;
191: public:
192: 
193:   /// Constructs the SYMM.
194:   Symm() { }
195: 
196:   /// Determines whether the SYMM can execute the given problem.
197:   static Status can_implement(Arguments const &args) {
198: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 199-204
```cpp
199:     if (!kSplitKSerial && args.batch_count > 1) {
200:       return Status::kErrorInvalidProblem;
201:     }
202: 
203:     Status status = SymmKernel::can_implement(args);
204: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 205-208
```cpp
205:     if (SideModeA == SideMode::kInvalid) {
206:       return Status::kErrorInvalidProblem;
207:     }
208:    
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 209-212
```cpp
209:     if (FillModeA != FillMode::kLower && FillModeA != FillMode::kUpper) {
210:       return Status::kErrorInvalidProblem;
211:     }
212: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 213-222
```cpp
213:     if (status != Status::kSuccess) {
214:       return status;
215:     }
216: 
217:     return Status::kSuccess;
218:   }
219: 
220:   /// Gets the workspace size
221:   static size_t get_workspace_size(Arguments const &args) {
222:     
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 223-227
```cpp
223:     size_t bytes = 0;
224: 
225:     // Determine grid shape
226:     ThreadblockSwizzle threadblock_swizzle;
227: 
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 228-237
```cpp
228:     cutlass::gemm::GemmCoord tiled_shape = threadblock_swizzle.get_tiled_shape(
229:       args.problem_size, 
230:       {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
231:       args.batch_count);
232:     
233:     if (kSplitKSerial && args.batch_count > 1) {
234: 
235:       bytes += sizeof(int) * size_t(tiled_shape.m()) * size_t(tiled_shape.n());
236:     }
237: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 238-246
```cpp
238:     return bytes;
239:   }
240: 
241:   /// Initializes SYMM state from arguments.
242:   Status initialize(Arguments const &args, void *workspace = nullptr, cudaStream_t stream = nullptr) {
243:     
244:     // Determine grid shape
245:     ThreadblockSwizzle threadblock_swizzle;
246: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 247-251
```cpp
247:     cutlass::gemm::GemmCoord grid_tiled_shape = threadblock_swizzle.get_tiled_shape(
248:       args.problem_size, 
249:       {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
250:       args.batch_count);
251: 
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 252-261
```cpp
252:     if (kSplitKSerial) {
253:       if (args.batch_count > 1) {
254:         if (!workspace) {
255:           return Status::kErrorWorkspaceNull;
256:         }
257: 
258:         size_t bytes = get_workspace_size(args);
259:       
260:         cudaError_t result = cudaMemsetAsync(workspace, 0, bytes, stream);
261: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 262-268
```cpp
262:         if (result != cudaSuccess) {
263:           return Status::kErrorInternal;
264:         }
265:       }
266:     }
267:     else {
268: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 269-278
```cpp
269:       if (args.batch_count > 1) {
270:         return Status::kErrorInvalidProblem;
271:       }
272:     }
273:     
274:     int gemm_k_size = args.problem_size.k();
275: 
276:    // Swapping argument for A and B, if A was on the right side (problem size doesn't need to change here).
277:     if (kSideModeA == SideMode::kRight) {
278:       // Initialize the Params structure
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 279-288
```cpp
279:       params_ = typename SymmKernel::Params{
280:         args.swapped_matrices(),
281:         grid_tiled_shape,
282:         gemm_k_size,
283:         static_cast<int *>(workspace)
284:       };
285: 
286:       return Status::kSuccess;
287:     }
288: 
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 289-289
```cpp
289:     // Initialize the Params structure
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 290-299
```cpp
290:     params_ = typename SymmKernel::Params{
291:       args,
292:       grid_tiled_shape,
293:       gemm_k_size,
294:       static_cast<int *>(workspace)
295:     };
296:     
297:     return Status::kSuccess;
298:   }
299: 
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 300-302
```cpp
300:   /// Lightweight update given a subset of arguments
301:   Status update(Arguments const &args, void *workspace = nullptr) {
302:     
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 303-310
```cpp
303:     if (kSplitKSerial && args.batch_count > 1) {  
304:       if (!workspace) {
305:         return Status::kErrorWorkspaceNull;
306:       }
307:     }
308: 
309:     size_t workspace_bytes = get_workspace_size(args);
310: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 311-320
```cpp
311:     if (workspace_bytes && !workspace) {
312:       return Status::kErrorWorkspaceNull;
313:     }
314: 
315:     params_.update(args, workspace);
316: 
317:     return Status::kSuccess;
318:   }
319: 
320:   /// Runs the kernel using initialized state.
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 321-324
```cpp
321:   Status run(cudaStream_t stream = nullptr) {
322: 
323:     ThreadblockSwizzle threadblock_swizzle;
324: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 325-329
```cpp
325:     dim3 grid = threadblock_swizzle.get_grid_shape(params_.grid_tiled_shape);
326:     dim3 block(SymmKernel::kThreadCount, 1, 1);
327: 
328:     int smem_size = int(sizeof(typename SymmKernel::SharedStorage));
329: 
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 330-334
```cpp
330:     if (smem_size >= (48 << 10)) {
331:       cudaError_t result = cudaFuncSetAttribute(Kernel<SymmKernel>,
332:                                     cudaFuncAttributeMaxDynamicSharedMemorySize,
333:                                     smem_size);
334: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 335-339
```cpp
335:       if (result != cudaSuccess) {
336:         return Status::kErrorInternal;
337:       }
338:     }
339: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 340-348
```cpp
340:     cutlass::arch::synclog_setup();
341:     cutlass::Kernel<SymmKernel><<<grid, block, smem_size, stream>>>(params_);
342: 
343:     cudaError_t result = cudaGetLastError();
344: 
345:     return result == cudaSuccess ? Status::kSuccess : Status::kErrorInternal;
346:   }
347: 
348:   /// Runs the kernel using initialized state.
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 349-353
```cpp
349:   Status operator()(cudaStream_t stream = nullptr) {
350:     return run(stream);
351:   }
352: 
353:   /// Runs the kernel using initialized state.
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 354-360
```cpp
354:   Status operator()(
355:     Arguments const &args, 
356:     void *workspace = nullptr, 
357:     cudaStream_t stream = nullptr) {
358:     
359:     Status status = initialize(args, workspace);
360:     
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 361-369
```cpp
361:     if (status == Status::kSuccess) {
362:       status = run(stream);
363:     }
364: 
365:     return status;
366:   }
367: };
368: ////////////////////////////////////////////////////////////////////////////////
369: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 370-378
```cpp
370: /********************************************************************************************************
371:   SYMM/HEMM has 4 combinations based on Layouts {RowMajor, ColumnMajor} x Side mode {LeftSide, RightSide}
372:   In templates and arguments to cutlass kernel, `matrix A` is always symmetric/hermitian, and `matrix B` is rectangular. 
373:   (adhering to the cuBLAS convention)
374: 
375:   Although, cuBLAS SYMM/HEMM only supports ColumnMajor layouts for all matrices (A, B, C/D).
376: 
377:   For the mainloop and symm kernel, `A` and `B` points to left-side and right-side matrices, respectively.
378:   
```
**EN:** Introduces the license header and legal reuse conditions for this header.
**CN:** 说明该头文件的许可证声明与复用条件。

### Lines 379-381
```cpp
379:   Thus, for LeftSide mode `A` and `B` points to `matrix A` and `matrix B`, respectively. While for 
380:   the RightSide mode `A` and `B` points to `matrix B` and `matrix A`, respectively. 
381:   
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 382-387
```cpp
382:   Additionally, CUTLASS GEMM epilogue is always RowMajor, and ColumnMajor output is achieved by 
383:   transposing the GEMM problem. Thus, ColumnMajor output layout for SYMM/HEMM requires:
384:    - Transposing `matrix A` and `matrix B` layouts
385:    - Swapping problem size m and n values
386:    - Swapping LeftSide and RightSide mode
387:   
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 388-390
```cpp
388:   RowMajor output:    D = matrix A x matrix B
389:   ColumnMajor output: D = matrix A x matrix B -> Transpose (D) = Transpose(matrix B) x Transpose(matrix A)
390: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 391-398
```cpp
391:   {RowMajor, ColumnMajor} x Side Mode {LeftSide, RightSide} 4 cases:
392:     1.  LeftSide mode and RowMajor output (default template)
393:     2.  LeftSide mode and ColumnMajor output 
394:     3.  RightSide mode and RowMajor output
395:     4.  RightSide mode and ColumnMajor output
396:   
397:   Mapping ColumnMajor output layout cases 2 and 4 to RowMajor efficient epilogue implementation:
398:   
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 399-404
```cpp
399:   Case 2 -> Case 3:
400:       D_col = matrix A x matrix B (LeftSide mode) 
401:    => Transpose(D_col) = Transpose(matrix B) x Transpose(matrix A) (RightSide mode)
402: 
403:   swap pointers for `A` and `B` call GEMM mainloop with RowMajor efficient-epilogue
404: 
```
**EN:** This block focuses on epilogue related implementation details.
**CN:** 该代码块聚焦于 epilogue 输出阶段 的实现细节。

### Lines 405-408
```cpp
405:   Case 4 -> Case 1:
406:       D_col = matrix B x matrix A (RightSide mode) 
407:    => Transpose(D_col) = Transpose(matrix A) x Transpose(matrix B) (LeftSide mode)
408: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 409-418
```cpp
409:    call GEMM mainloop for with RowMajor efficient-epilogue
410: ********************************************************************************************************/
411: 
412: /// Partial specialization for column-major output exchanges problem size and operand.
413: template <
414:     /// Element type for A matrix operand
415:     typename ElementA_,
416:     /// Layout type for A matrix operand
417:     typename LayoutA_,
418:     /// Side Mode for A (kLeft or kRight)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 419-428
```cpp
419:     SideMode SideModeA,
420:     /// Fill Mode for A (kLower or kUpper)
421:     FillMode FillModeA,
422:     /// Element type for B matrix operand
423:     typename ElementB_,
424:     /// Layout type for B matrix operand
425:     typename LayoutB_,
426:     /// Element type for C and D matrix operands
427:     typename ElementC_,
428:     /// Element type for internal accumulation
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 429-430
```cpp
429:     typename ElementAccumulator_,
430:     /// Operator class tag
```
**EN:** Defines tag, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tag，用于封装策略、存储或算法行为的辅助类型。

### Lines 431-440
```cpp
431:     typename OperatorClass_,
432:     /// Tag indicating architecture to tune for.  This is the minimum SM that
433:     /// supports the intended feature. The device kernel can be built
434:     /// targeting any SM larger than this number.
435:     typename ArchTag_,
436:     /// Threadblock-level tile size (concept: GemmShape)
437:     typename ThreadblockShape_,
438:     /// Warp-level tile size (concept: GemmShape)
439:     typename WarpShape_,
440:     /// Instruction-level tile size (concept: GemmShape)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 441-450
```cpp
441:     typename InstructionShape_,
442:     /// Epilogue output operator
443:     typename EpilogueOutputOp_,
444:     /// Threadblock-level swizzling operator
445:     typename ThreadblockSwizzle_,
446:     /// Number of stages used in the pipelined mainloop
447:     int Stages,
448:     /// Access granularity of A matrix in units of elements
449:     int AlignmentA,
450:     /// Access granularity of B matrix in units of elements
```
**EN:** This block focuses on swizzle, epilogue related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射、epilogue 输出阶段 的实现细节。

### Lines 451-456
```cpp
451:     int AlignmentB,
452:     /// If true, kernel supports split-K with serial reduction
453:     bool SplitKSerial,
454:     /// Operation performed by Symm update kernel
455:     typename Operator_,
456:     /// Blas3 computation mode (symmetric/hermitian)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 457-466
```cpp
457:     BlasMode BlasMode_
458:     >
459: class Symm<ElementA_, LayoutA_, SideModeA, FillModeA, ElementB_, LayoutB_, ElementC_,
460:            layout::ColumnMajor,  // partially specialized on LayoutC
461:            ElementAccumulator_, OperatorClass_, ArchTag_, ThreadblockShape_,
462:            WarpShape_, InstructionShape_, EpilogueOutputOp_,
463:            ThreadblockSwizzle_, Stages, AlignmentA, AlignmentB,
464:            SplitKSerial, Operator_, BlasMode_> {
465:  public:
466: 
```
**EN:** Defines Symm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Symm，用于封装策略、存储或算法行为的辅助类型。

### Lines 467-478
```cpp
467:   using ElementA = ElementA_;
468:   using LayoutA = LayoutA_;
469:   using ElementB = ElementB_;
470:   using LayoutB = LayoutB_;
471:   using ElementC = ElementC_;
472:   using LayoutC = layout::ColumnMajor;
473:   using ElementAccumulator = ElementAccumulator_;
474:   using OperatorClass = OperatorClass_;
475:   using ArchTag = ArchTag_;
476:   using ThreadblockShape = ThreadblockShape_;
477:   using WarpShape = WarpShape_;
478:   using InstructionShape = InstructionShape_;
```
**EN:** Introduces local type aliases (ElementA, LayoutA, ElementB, LayoutB) to simplify downstream template code.
**CN:** 引入本地类型别名（ElementA, LayoutA, ElementB, LayoutB），简化后续模板代码。

### Lines 479-490
```cpp
479:   using EpilogueOutputOp = EpilogueOutputOp_;
480:   using ThreadblockSwizzle = ThreadblockSwizzle_;
481:   using Operator = Operator_;
482:   static SideMode const kSideModeA = SideModeA;
483:   static FillMode const kFillModeA = FillModeA;
484:   static int const kStages = Stages;
485:   static int const kAlignmentA = AlignmentA;
486:   static int const kAlignmentB = AlignmentB;
487:   static int const kAlignmentC = EpilogueOutputOp::kCount;
488:   static bool const kSplitKSerial = SplitKSerial;
489:   static BlasMode const kBlasMode = BlasMode_;
490:   
```
**EN:** Introduces local type aliases (EpilogueOutputOp, ThreadblockSwizzle, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, ThreadblockSwizzle, Operator），简化后续模板代码。

### Lines 491-491
```cpp
491:   /// Define the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 492-503
```cpp
492:   using UnderlyingOperator = typename cutlass::gemm::device::Symm<
493:     ElementA,
494:     typename layout::LayoutTranspose<LayoutA>::type,
495:     InvertSideMode<kSideModeA>::mode,
496:     InvertFillMode<kFillModeA>::mode,
497:     ElementB,
498:     typename layout::LayoutTranspose<LayoutB>::type, 
499:     ElementC,
500:     layout::RowMajor,
501:     ElementAccumulator,
502:     OperatorClass,
503:     ArchTag,
```
**EN:** Introduces local type aliases (UnderlyingOperator) to simplify downstream template code.
**CN:** 引入本地类型别名（UnderlyingOperator），简化后续模板代码。

### Lines 504-515
```cpp
504:     ThreadblockShape,
505:     WarpShape,
506:     InstructionShape,
507:     EpilogueOutputOp,
508:     ThreadblockSwizzle,
509:     kStages,
510:     kAlignmentA,
511:     kAlignmentB,
512:     kSplitKSerial,
513:     Operator,
514:     kBlasMode
515:   >;
```
**EN:** This block focuses on swizzle, epilogue related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射、epilogue 输出阶段 的实现细节。

### Lines 516-518
```cpp
516:   
517: 
518:   /// Argument structure
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 519-528
```cpp
519:   using Arguments = typename UnderlyingOperator::Arguments;
520:   using SymmKernel = typename UnderlyingOperator::SymmKernel;
521: 
522: private:
523: 
524:   UnderlyingOperator underlying_operator_;
525: 
526: public:
527: 
528:   /// Constructs the Symm.
```
**EN:** Introduces local type aliases (Arguments, SymmKernel) to simplify downstream template code.
**CN:** 引入本地类型别名（Arguments, SymmKernel），简化后续模板代码。

### Lines 529-531
```cpp
529:   Symm() { }
530: 
531:   /// Helper to construct a transposed equivalent for the underlying SYMM operator
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 532-541
```cpp
532:   static Arguments to_underlying_arguments(Arguments const &args) {
533:     return args.transposed_problem_size();
534:   }
535: 
536:   /// Determines whether the Symm can execute the given problem.
537:   static Status can_implement(Arguments const &args) {
538: 
539:     return UnderlyingOperator::can_implement(to_underlying_arguments(args));
540:   }
541: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 542-548
```cpp
542:   /// Gets the workspace size
543:   static size_t get_workspace_size(Arguments const &args) {
544:     
545:     return UnderlyingOperator::get_workspace_size(to_underlying_arguments(args));
546:   }
547: 
548:   /// Computes the grid shape
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 549-553
```cpp
549:   static dim3 get_grid_shape(Arguments const &args) { 
550:     return UnderlyingOperator::get_grid_shape(to_underlying_arguments(args));
551:   }
552: 
553:   /// Computes the maximum number of active blocks per multiprocessor
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 554-563
```cpp
554:   static int maximum_active_blocks(int smem_capacity = -1) {
555:     return UnderlyingOperator::maximum_active_blocks(smem_capacity);
556:   }
557: 
558:   /// Initializes Symm state from arguments.
559:   Status initialize(Arguments const &args, void *workspace = nullptr, cudaStream_t stream = nullptr) {
560: 
561:     return underlying_operator_.initialize(to_underlying_arguments(args), workspace, stream);
562:   }
563: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 564-574
```cpp
564:   /// Lightweight update given a subset of arguments
565:   Status update(Arguments const &args, void *workspace = nullptr) {
566: 
567:     return underlying_operator_.update(to_underlying_arguments(args), workspace);
568:   }
569: 
570:   /// Runs the kernel using initialized state.
571:   Status run(cudaStream_t stream = nullptr) {
572: 
573:     return underlying_operator_.run(stream);
574:   }
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 575-576
```cpp
575: 
576:   /// Runs the kernel using initialized state.
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 577-581
```cpp
577:   Status operator()(cudaStream_t stream = nullptr) {
578:     return run(stream);
579:   }
580: 
581:   /// Runs the kernel using initialized state.
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 582-588
```cpp
582:   Status operator()(
583:     Arguments const &args, 
584:     void *workspace = nullptr, 
585:     cudaStream_t stream = nullptr) {
586:     
587:     Status status = initialize(args, workspace, stream);
588:     
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 589-598
```cpp
589:     if (status == Status::kSuccess) {
590:       status = run(stream);
591:     }
592: 
593:     return status;
594:   }
595: };
596: 
597: ////////////////////////////////////////////////////////////////////////////////
598: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 599-603
```cpp
599: } // namespace device
600: } // namespace Symm
601: } // namespace cutlass
602: 
603: ////////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Host-facing device operator wrappers  
  **CN:** 面向主机的设备算子封装
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Tensor Core instruction mapping  
  **CN:** Tensor Core 指令映射
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线
- **EN:** Epilogue/output transformation  
  **CN:** Epilogue/输出变换
- **EN:** Threadblock swizzle mapping  
  **CN:** 线程块 swizzle 映射

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `tag`, `Symm`, `can_implement`, `get_workspace_size`, `initialize`, `side`, `update`, `run`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
