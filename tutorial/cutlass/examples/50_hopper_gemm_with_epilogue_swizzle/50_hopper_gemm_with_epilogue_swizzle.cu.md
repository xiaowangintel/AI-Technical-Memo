# 50_hopper_gemm_with_epilogue_swizzle.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/50_hopper_gemm_with_epilogue_swizzle/50_hopper_gemm_with_epilogue_swizzle.cu`  
**Purpose / 用途**: This sample assembles a Hopper GEMM manually so it can demonstrate a custom epilogue swizzle: the output tile is staged through a swizzled shared-memory layout before vectorized stores write the final tensor. / 该示例手工组装了一个 Hopper GEMM，以便展示自定义 epilogue swizzle：输出 tile 会先经过带 swizzle 的共享内存布局，再通过向量化 store 写回最终张量。

---

## Line-by-Line Analysis / 逐行分析

The sections below cover the full source file in order. Each block includes original code and bilingual analysis.  
下面的各个小节按顺序覆盖整个源文件。每个代码块都附带原始代码与中英双语分析。

### Lines 1-60 | File overview and epilogue motivation / 文件概览与 epilogue 动机

```cpp
 1: /***************************************************************************************************
 2:  * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
31: 
32: /*! \file
33:     \brief Hopper GEMM example to create a GEMM kernel with custom Collectives
34: 
35:     The following example shows how to assemble a custom GEMM kernel that spells out the Collectives
36:     directly instead of using a builder and, in the process, instance a more efficient Epilogue
37:     (from `cutlass/epilogue/collective/sm70_epilogue_vectorized.hpp`) instead of using the default epilogue.
38: 
39:     The GemmUniversal API takes 3 main template arguments:
40:       (1) the problem shape / extents
41:       (2) the collective mainloop type
42:       (3) the collective epilogue type
43: 
44:     While the collecive mainloop can be stamped out using a CollectiveBuilder interface, it is
45:     possible to build a custom collective mainloop directly as well. Furthermore, since epilogues
46:     do not yet have a builder interface, this example shows how to instantiate a more-efficient
47:     epilogue alongside the collective mainloop.
48: 
49:     Note: there are several ways to implement the GEMM epilogue in Hopper - each with its own set
50:     of trade-offs. So it is recommended that users look at the options available under
51:     cutlass/epilogue/collective and evaluate for their particular scenario.
52: 
53:     Please refer to examples 48, 49 to learn more about kernel schedules and other CuTe examples
54:     present in `test/unit/cute` to famialiarize with the basics of CuTe.
55: 
56:     Examples:
57: 
58:       $ ./examples/50_hopper_gemm_with_epilogue_swizzle/50_hopper_gemm_with_epilogue_swizzle
59: */
60: 
```

- **EN**: The introductory comment explains that the sample does not rely only on high-level builders; instead it assembles a custom mainloop and epilogue to study a more efficient Hopper write-back path. The central theme is not the GEMM math itself but the output staging strategy.
- **CN**: 开头注释说明该示例并不只依赖高层 builder，而是手工组装 mainloop 与 epilogue，以研究一种更高效的 Hopper 写回路径。核心关注点并不是 GEMM 数学本身，而是输出的 staging 策略。

### Lines 61-141 | Includes and basic runtime options / 头文件与基础运行选项

```cpp
 61: #include <iostream>
 62: 
 63: #include "cutlass/cutlass.h"
 64: 
 65: #include "cute/tensor.hpp"
 66: #include "cutlass/util/command_line.h"
 67: #include "cutlass/tensor_ref.h"
 68: #include "cutlass/epilogue/collective/collective_epilogue.hpp"
 69: #include "cutlass/epilogue/thread/linear_combination.h"
 70: #include "cutlass/gemm/dispatch_policy.hpp"
 71: #include "cutlass/gemm/collective/collective_builder.hpp"
 72: #include "cutlass/gemm/device/gemm_universal_adapter.h"
 73: #include "cutlass/gemm/kernel/gemm_universal.hpp"
 74: #include "cutlass/gemm/dispatch_policy.hpp"
 75: 
 76: #include "cutlass/util/command_line.h"
 77: #include "cutlass/util/distribution.h"
 78: #include "cutlass/util/host_tensor.h"
 79: #include "cutlass/util/packed_stride.hpp"
 80: #include "cutlass/util/tensor_view_io.h"
 81: #include "cutlass/util/reference/device/gemm_complex.h"
 82: #include "cutlass/util/reference/device/tensor_compare.h"
 83: #include "cutlass/util/reference/device/tensor_fill.h"
 84: 
 85: using namespace cute;
 86: 
 87: ///////////////////////////////////////////////////////////////////////////////////////////////////
 88: 
 89: // Command line options parsing
 90: struct Options {
 91: 
 92:   bool help;
 93:   bool error;
 94: 
 95:   int m, n, k, l;
 96:   int alpha, beta;
 97: 
 98:   Options():
 99:     help(false),
100:     error(false),
101:     m(2048), n(2048), k(2048), l(1),
102:     alpha(1), beta(0)
103:   { }
104: 
105:   // Parses the command line
106:   void parse(int argc, char const **args) {
107:     cutlass::CommandLine cmd(argc, args);
108: 
109:     if (cmd.check_cmd_line_flag("help")) {
110:       help = true;
111:       return;
112:     }
113: 
114:     cmd.get_cmd_line_argument("m", m, 2048);
115:     cmd.get_cmd_line_argument("n", n, 2048);
116:     cmd.get_cmd_line_argument("k", k, 2048);
117:     cmd.get_cmd_line_argument("l", l, 1);
118:     cmd.get_cmd_line_argument("alpha", alpha, 1);
119:     cmd.get_cmd_line_argument("beta", beta, 0);
120:   }
121: 
122:   /// Prints the usage statement.
123:   std::ostream & print_usage(std::ostream &out) const {
124: 
125:     out << "50_hopper_gemm_with_epilogue_swizzle\n\n"
126:       << "Hopper GEMM Example with Epilogue Swizzle.\n\n"
127:       << "Options:\n\n"
128:       << "  --help                      If specified, displays this usage statement\n\n"
129:       << "  --m=<int>                   Sets the M extent of the GEMM\n"
130:       << "  --n=<int>                   Sets the N extent of the GEMM\n"
131:       << "  --k=<int>                   Sets the K extent of the GEMM\n"
132:       << "  --l=<int>                   Sets the L extent (batch count) of the GEMM\n"
133:       << "  --alpha=<s32>               Epilogue scalar alpha\n"
134:       << "  --beta=<s32>                Epilogue scalar beta\n\n";
135: 
136:     return out;
137:   }
138: };
139: 
140: ///////////////////////////////////////////////////////////////////////////////////////////////////
141: 
```

- **EN**: This block introduces the required CUTLASS pieces and defines a small `Options` structure for M/N/K/L and alpha/beta. The sample keeps runtime controls intentionally small because the interesting customization happens at compile time in the kernel assembly below.
- **CN**: 这一段引入所需的 CUTLASS 组件，并定义一个简洁的 `Options` 结构来控制 M/N/K/L 和 alpha/beta。运行时选项保持得很精简，因为真正有意思的定制都发生在下方的编译期 kernel 组装过程中。

### Lines 142-169 | Tensor initialization helper / 张量初始化辅助函数

```cpp
142: /// Helper to initialize a block of device data
143: template <class Element>
144: bool initialize_block(
145:   cutlass::DeviceAllocation<Element>& block,
146:   uint64_t seed=2023) {
147: 
148:   Element scope_max, scope_min;
149:   int bits_input = cutlass::sizeof_bits<Element>::value;
150: 
151:   if (bits_input == 1) {
152:     scope_max = 2;
153:     scope_min = 0;
154:   } else if (bits_input <= 8) {
155:     scope_max = 2;
156:     scope_min = -2;
157:   } else {
158:     scope_max = 8;
159:     scope_min = -8;
160:   }
161: 
162:   cutlass::reference::device::BlockFillRandomUniform(
163:     block.get(), block.size(), seed, scope_max, scope_min, 0);
164: 
165:   return true;
166: }
167: 
168: ///////////////////////////////////////////////////////////////////////////////////////////////////
169: 
```

- **EN**: The helper chooses fill ranges based on the element bit width so int8 and wider types receive sensible pseudo-random test values before benchmarking and verification.
- **CN**: 该辅助函数会依据元素位宽选择填充值范围，让 int8 及更宽的数据类型在 benchmark 和验证之前得到合理的伪随机测试数据。

### Lines 170-280 | Example runner state, initialization, and verification / 示例运行器状态、初始化与验证

```cpp
170: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
171: 
172: // Wrapper to run and verify a GEMM.
173: template <
174:   class Gemm
175: >
176: struct ExampleRunner {
177: 
178:   using StrideA = typename Gemm::GemmKernel::StrideA;
179:   using StrideB = typename Gemm::GemmKernel::StrideB;
180:   using StrideC = typename Gemm::GemmKernel::StrideC;
181:   using StrideD = typename Gemm::GemmKernel::StrideD;
182: 
183:   using LayoutA = typename Gemm::LayoutA;
184:   using LayoutB = typename Gemm::LayoutB;
185:   using LayoutC = typename Gemm::LayoutC;
186:   using LayoutD = typename Gemm::LayoutD;
187: 
188:   using ElementA = typename Gemm::ElementA;
189:   using ElementB = typename Gemm::ElementB;
190:   using ElementAcc = typename Gemm::ElementAccumulator;
191: 
192:   using CollectiveEpilogue = typename Gemm::CollectiveEpilogue;
193:   using ElementC = typename Gemm::ElementC;
194:   using ElementOutput = typename CollectiveEpilogue::ElementOutput;
195:   using ElementCompute = typename CollectiveEpilogue::ElementCompute;
196:   using ElementAccumulator = typename CollectiveEpilogue::ElementAccumulator;
197: 
198:   using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
199: 
200:   //
201:   // Data members
202:   //
203: 
204:   /// Initialization
205:   StrideA stride_A;
206:   StrideB stride_B;
207:   StrideC stride_C;
208:   StrideD stride_D;
209:   uint64_t seed = 0;
210: 
211:   cutlass::DeviceAllocation<ElementA> block_A;
212:   cutlass::DeviceAllocation<ElementB> block_B;
213:   cutlass::DeviceAllocation<ElementC> block_C;
214:   cutlass::DeviceAllocation<ElementOutput> block_D;
215:   cutlass::DeviceAllocation<ElementOutput> block_ref_D;
216: 
217:   //
218:   // Methods
219:   //
220: 
221:   bool verify(const ProblemShapeType& problem_size, int32_t alpha, int32_t beta) {
222:     auto [M, N, K, L] = problem_size;
223: 
224:     cutlass::TensorRef ref_A(block_A.get(), LayoutA::packed({M, K}));
225:     cutlass::TensorRef ref_B(block_B.get(), LayoutB::packed({K, N}));
226:     cutlass::TensorRef ref_C(block_C.get(), LayoutC::packed({M, N}));
227:     cutlass::TensorRef ref_D(block_ref_D.get(), LayoutD::packed({M, N}));
228: 
229:     cutlass::reference::device::GemmComplex(
230:           {M, N, K},
231:           ElementCompute(alpha),
232:           ref_A,
233:           cutlass::ComplexTransform::kNone,
234:           ref_B,
235:           cutlass::ComplexTransform::kNone,
236:           ElementCompute(beta),
237:           ref_C,
238:           ref_D,
239:           ElementAccumulator(0),
240:           L,     // batch_count
241:           M * K, // batch_stride_A
242:           K * N, // batch_stride_B
243:           M * N, // batch_stride_C
244:           M * N  // batch_stride_D
245:         );
246: 
247:     cudaError_t result = cudaDeviceSynchronize();
248:     if (result != cudaSuccess) {
249:       std::cerr << "Reference kernel failed. Last CUDA error: "
250:                 << cudaGetErrorString(result) << std::endl;
251:       return false;
252:     }
253: 
254:     // Check if output from CUTLASS kernel and reference kernel are equal or not
255:     bool passed = cutlass::reference::device::BlockCompareEqual(block_ref_D.get(), block_D.get(), block_D.size());
256: 
257:     return passed;
258:   }
259: 
260:   /// Initialize operands to be used in the GEMM and reference GEMM
261:   void initialize(const ProblemShapeType& problem_size) {
262:     auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
263:     auto [M, N, K, L] = problem_shape_MNKL;
264: 
265:     stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
266:     stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
267:     stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(M, N, L));
268:     stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(M, N, L));
269: 
270:     block_A.reset(M * K * L);
271:     block_B.reset(K * N * L);
272:     block_C.reset(M * N * L);
273:     block_D.reset(M * N * L);
274:     block_ref_D.reset(M * N * L);
275: 
276:     initialize_block(block_A, seed + 2023);
277:     initialize_block(block_B, seed + 2022);
278:     initialize_block(block_C, seed + 2021);
279:   }
280: 
```

- **EN**: The `ExampleRunner` template owns device buffers, builds packed strides from the chosen layouts, and provides both initialization and verification logic. Verification is separate from performance measurement, which makes it easy to reason about correctness before studying the swizzled epilogue performance.
- **CN**: `ExampleRunner` 模板负责管理设备缓冲区，根据布局构造打包 stride，并提供初始化与验证逻辑。验证过程与性能测试分离，因此可以先确认正确性，再研究 swizzled epilogue 的性能收益。

### Lines 281-342 | Runner execution wrapper / 运行器执行封装

```cpp
281:   bool run(const Options& options, const cutlass::KernelHardwareInfo& hw_info) {
282:     ProblemShapeType problem_size = ProblemShapeType{options.m, options.n, options.k, options.l};
283: 
284:     initialize(problem_size);
285: 
286:     typename Gemm::GemmKernel::Arguments arguments{
287:       cutlass::gemm::GemmUniversalMode::kGemm,
288:       problem_size,
289:       {block_A.get(), stride_A, block_B.get(), stride_B},
290:       {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D},
291:       hw_info
292:     };
293: 
294:     Gemm gemm_op;
295: 
296:     size_t workspace_size = Gemm::get_workspace_size(arguments);
297:     cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
298: 
299:     cutlass::Status status = gemm_op.can_implement(arguments);
300:     if (status != cutlass::Status::kSuccess) {
301:       std::cerr << "This kernel is not supported. Last CUDA error is: "
302:                 << cudaGetErrorString(cudaGetLastError()) << std::endl;
303:       return false;
304:     }
305: 
306:     status = gemm_op.initialize(arguments, workspace.get());
307:     if (status != cutlass::Status::kSuccess) {
308:       std::cerr << "Failed to initialize the CUTLASS kernel. Last CUDA error is: "
309:                 << cudaGetErrorString(cudaGetLastError()) << std::endl;
310:       return false;
311:     }
312: 
313:     // Run the GEMM
314:     status = gemm_op.run();
315:     if (status != cutlass::Status::kSuccess) {
316:       std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
317:                 << cudaGetErrorString(cudaGetLastError()) << std::endl;
318:       return false;
319:     }
320: 
321:     cudaError_t result = cudaDeviceSynchronize();
322:     if (result != cudaSuccess) {
323:       std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
324:                 << cudaGetErrorString(result) << std::endl;
325:       return false;
326:     }
327: 
328:     // Verify that the result is correct
329:     bool passed = verify(problem_size, options.alpha, options.beta);
330:     if (!passed) {
331:       std::cerr << "Reference check failed" << std::endl;
332:     }
333: 
334:     return passed;
335:   }
336: 
337: };
338: 
339: #endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
340: 
341: ///////////////////////////////////////////////////////////////////////////////////////////////////
342: 
```

- **EN**: This portion builds the runtime argument object, requests workspace, launches the kernel, synchronizes, and calls verification. It is a compact illustration of the normal CUTLASS device-adapter workflow for a single GEMM.
- **CN**: 这一段负责构造运行时参数对象、申请 workspace、启动 kernel、同步并调用验证，是单个 GEMM 场景下 CUTLASS device adapter 工作流的精炼示例。

### Lines 343-476 | Architecture checks and mainloop assembly / 架构检查与 mainloop 组装

```cpp
343: int main(int argc, char const **args) {
344: 
345:   cudaDeviceProp props;
346: 
347:   cudaError_t error = cudaGetDeviceProperties(&props, 0);
348:   if (error != cudaSuccess) {
349:     std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
350:     return -1;
351:   }
352: 
353:   if (__CUDACC_VER_MAJOR__ < 12 || props.major < 9) {
354:     std::cout
355:       << "This example requires a GPU of NVIDIA's Hopper Architecture or "
356:       << "later (compute capability 90 or greater) and CUDA 12.0 or greater.\n";
357:       return 0;
358:   }
359: 
360:   else if (__CUDACC_VER_MAJOR__ < 12 || props.major != 9 || props.minor != 0) {
361:     std::cout
362:       << "This example requires a GPU of NVIDIA's Hopper Architecture "
363:       << "(compute capability 90) and CUDA 12.0 or greater.\n";
364:     return 0;
365:   }
366: 
367: 
368:   //
369:   // Parse options
370:   //
371: 
372:   Options options;
373: 
374:   options.parse(argc, args);
375: 
376:   if (options.help) {
377:     options.print_usage(std::cout) << std::endl;
378:     return 0;
379:   }
380: 
381:   if (options.error) {
382:     std::cerr << "Aborting execution." << std::endl;
383:     return -1;
384:   }
385: 
386: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
387: 
388:   //
389:   // Run examples
390:   //
391: 
392:   // The KernelHardwareInfo struct holds the number of SMs on the GPU with a given device ID. This
393:   // information is used by the underlying kernel.
394:   cutlass::KernelHardwareInfo hw_info;
395: 
396:   // Change device_id to another value if you are running on a machine with multiple GPUs and wish
397:   // to use a GPU other than that with device ID 0.
398:   hw_info.device_id = 0;
399:   hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
400: 
401:   bool passed;
402: 
403:   // Problem configuration
404:   using ElementA = int8_t;
405:   using ElementB = int8_t;
406:   using ElementAcc = int32_t;
407:   using ElementOutput = int8_t;
408: 
409:   // Note : Only TN WGMMA Gemm is supported currently in 3.0
410:   using LayoutA = cutlass::layout::RowMajor;
411:   using LayoutB = cutlass::layout::ColumnMajor;
412:   using LayoutC = cutlass::layout::ColumnMajor;
413:   using LayoutD = cutlass::layout::ColumnMajor;
414: 
415:   // Tiling configuration selection
416:   using TileShape = Shape<_128,_64,_128>;
417: 
418:   // Choosing a thread block cluster larger than 1 allows us to Multicast data across thread blocks
419:   using ClusterShape = Shape<_1,_2,_1>;
420: 
421:   //
422:   // Assembling the CollectiveMainloop type
423:   //
424: 
425:   // Pipeline Depth to be used i.e number of A, B buffers in shared memory
426:   constexpr int PipelineStages = 8;
427: 
428:   // Let's choose a Warp-Specialized Mainloop implementation which uses TMA
429:   // Note : This requires / assumes the tensors to be 16B aligned
430:   using DispatchPolicy = cutlass::gemm::MainloopSm90TmaGmmaWarpSpecialized<PipelineStages, ClusterShape,
431:                            cutlass::gemm::KernelTmaWarpSpecialized>;
432: 
433:   // TN => K Major for both A & B
434:   static constexpr cute::GMMA::Major GmmaMajorA = cute::GMMA::Major::K;
435:   static constexpr cute::GMMA::Major GmmaMajorB = cute::GMMA::Major::K;
436: 
437:   // We use the SS op selector as both A, B operands are read directly from SMEM (for TN WGMMA)
438:   using TiledMma = decltype(cute::make_tiled_mma(cute::GMMA::ss_op_selector<
439:       ElementA, ElementB, ElementAcc, TileShape, GmmaMajorA, GmmaMajorB>()));
440: 
441:   // A loads can be optimized with multicast if cluster-n > 1
442:   using GmemTiledCopyA = std::conditional< cute::size(shape<1>(ClusterShape{})) == 1,
443:                            cute::SM90_TMA_LOAD,
444:                            cute::SM90_TMA_LOAD_MULTICAST>::type;
445: 
446:   // B loads can be optimized with multicast if cluster-m > 1
447:   using GmemTiledCopyB = std::conditional< cute::size(shape<0>(ClusterShape{})) == 1,
448:                            cute::SM90_TMA_LOAD,
449:                            cute::SM90_TMA_LOAD_MULTICAST>::type;
450: 
451:   using SmemLayoutAtomA = decltype(cutlass::gemm::collective::detail::ss_smem_selector<
452:       GmmaMajorA, ElementA, decltype(cute::get<0>(TileShape{})), decltype(cute::get<2>(TileShape{}))
453:     >());
454: 
455:   using SmemLayoutAtomB = decltype(cutlass::gemm::collective::detail::ss_smem_selector<
456:       GmmaMajorB, ElementB, decltype(cute::get<1>(TileShape{})), decltype(cute::get<2>(TileShape{}))
457:     >());
458: 
459:   using CollectiveMainloop = cutlass::gemm::collective::CollectiveMma<
460:       DispatchPolicy,
461:       TileShape,
462:       ElementA,
463:       cutlass::gemm::TagToStrideA_t<LayoutA>,
464:       ElementB,
465:       cutlass::gemm::TagToStrideB_t<LayoutB>,
466:       TiledMma,
467:       GmemTiledCopyA,
468:       SmemLayoutAtomA,
469:       void, // Does not need a SmemCopyAtom, since A is read directly from SMEM
470:       cute::identity,
471:       GmemTiledCopyB,
472:       SmemLayoutAtomB,
473:       void, // Does not need a SmemCopyAtom, since B is read directly from SMEM
474:       cute::identity
475:     >;
476: 
```

- **EN**: The entry point checks CUDA/Hopper availability, fixes the problem data types and layouts, selects a `1x2x1` cluster for multicast opportunities, defines the TMA+WGMMA dispatch policy, and constructs the `CollectiveMainloop`. By the time this block ends, the math pipeline and its shared-memory staging for A/B are fully defined.
- **CN**: 程序入口先检查 CUDA/Hopper 是否可用，然后确定问题的数据类型和布局，选择 `1x2x1` cluster 以利用 multicast 机会，定义 TMA+WGMMA dispatch policy，并构造 `CollectiveMainloop`。到这一段结束时，主计算流水线以及 A/B 的共享内存 staging 都已经确定。

### Lines 477-510 | Custom epilogue swizzle layout / 自定义 epilogue swizzle 布局

```cpp
477:   //
478:   // Assembling the Collective Epilogue Type
479:   //
480: 
481:   // Break the 128 along TILE_M into chunks of 32, to get a 128B leading dimension
482:   using PreSwizzleLayout = Layout< Shape< Shape <_32,_4   >,_64>,
483:                                    Stride<Stride< _1,_2048>,_32>>;
484: 
485:   // 128 threads loading 16 elements each (to get vectorized global stores)
486:   using TileShapeS2R = Shape<_128,_16>;
487: 
488:   // Layout to ensure bank-conflict free loads & stores
489:   using SmemLayout = ComposedLayout<
490:                        Swizzle<3,4,3>,
491:                        smem_ptr_flag_bits<sizeof_bits<ElementAcc>::value>,
492:                        PreSwizzleLayout>;
493: 
494:   // Tiled copy from Smem to Registers
495:   // Note : CuTe will vectorize this copy if the tiling + swizzling above were right
496:   using TiledCopyS2R = TiledCopy<
497:                          Copy_Atom<DefaultCopy, ElementAcc>,
498:                          Layout< Shape<_128,_16>,
499:                                  Stride<_16,_1>>,
500:                          TileShapeS2R>;
501: 
502:   using Epilogue = cutlass::epilogue::collective::detail::Sm90TmaWarpSpecializedAdapter<
503:     cutlass::epilogue::collective::Epilogue<
504:       cutlass::gemm::TagToStrideC_t<LayoutC>,
505:       cutlass::gemm::TagToStrideC_t<LayoutD>,
506:       cutlass::epilogue::thread::LinearCombination<int32_t, 1, int32_t, int32_t>,
507:       SmemLayout,
508:       Copy_Atom<DefaultCopy, ElementAcc>,
509:       TiledCopyS2R,
510:       Copy_Atom<DefaultCopy, ElementOutput>>>;
```

- **EN**: This is the most important block in the file. `PreSwizzleLayout` first reshapes the 128-row M tile into 32-row chunks so the leading dimension becomes 128 bytes. `Swizzle<3,4,3>` is then composed with shared-memory pointer flag bits to produce a bank-conflict-friendly `SmemLayout`. Finally, `TiledCopyS2R` defines a 128-thread by 16-element copy pattern so the epilogue can read the staged tile from shared memory and emit vectorized global stores. In short, epilogue swizzle changes the physical write-back order without changing the mathematical output.
- **CN**: 这是全文件最关键的一段。`PreSwizzleLayout` 先把 M 方向的 128 行 tile 重排成若干 32 行块，使 leading dimension 变成 128 字节；随后再把 `Swizzle<3,4,3>` 与共享内存指针标志位组合成更利于避免 bank conflict 的 `SmemLayout`；最后 `TiledCopyS2R` 规定了 128 线程 × 每线程 16 元素的拷贝模式，使 epilogue 可以从共享内存取出已 staging 的 tile，并进行向量化全局写回。简而言之，epilogue swizzle 改变的是物理写回顺序，而不是数学输出结果。

### Lines 511-535 | Final kernel type and launch / 最终 kernel 类型与启动

```cpp
511: 
512:   //
513:   // Assembling the GemmKernel
514:   //
515: 
516:   using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
517:       Shape<int,int,int,int>,
518:       CollectiveMainloop,
519:       Epilogue
520:   >;
521: 
522:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
523: 
524:   ExampleRunner<Gemm> runner;
525: 
526:   passed = runner.run(options, hw_info);
527: 
528:   std::cout << "WGMMA GEMM with Epilogue Swizzle : " << (passed ? "Passed" : "Failed") << std::endl;
529: 
530: #endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
531: 
532:   return 0;
533: }
534: 
535: /////////////////////////////////////////////////////////////////////////////////////////////////
```

- **EN**: The last lines bind the custom epilogue to the mainloop inside a universal GEMM adapter and run the sample through `ExampleRunner`. This closes the example by showing that a bespoke epilogue can still integrate cleanly with the standard CUTLASS launcher pattern.
- **CN**: 最后几行把自定义 epilogue 与 mainloop 绑定进通用 GEMM 适配器，并通过 `ExampleRunner` 执行示例。这说明即便使用了定制 epilogue，也依然可以顺畅地接入标准 CUTLASS 启动流程。

---

## Key Concepts / 关键概念

- **EN**: Epilogue swizzle reorganizes the output tile in shared memory to improve store efficiency and avoid shared-memory bank conflicts.
  **CN**: Epilogue swizzle 会在共享内存中重组输出 tile，从而提升写回效率并避免 shared memory bank conflict。
- **EN**: `PreSwizzleLayout` controls the coarse tile reshaping, while `Swizzle<3,4,3>` adds the fine-grained address permutation.
  **CN**: `PreSwizzleLayout` 负责较粗粒度的 tile 重塑，而 `Swizzle<3,4,3>` 负责更细粒度的地址置换。
- **EN**: The vectorized `TiledCopyS2R` path lets the epilogue convert a carefully staged shared-memory tile into efficient global writes.
  **CN**: 向量化的 `TiledCopyS2R` 路径使 epilogue 能把精心 staging 的共享内存 tile 转换为高效的全局写操作。
- **EN**: The mainloop and epilogue are assembled separately, which highlights how CUTLASS lets advanced users replace just one phase of the pipeline.
  **CN**: mainloop 与 epilogue 是分开组装的，这正体现了 CUTLASS 允许高级用户只替换流水线中的某一阶段。

## Dependencies / 依赖项

- **CUTLASS collective mainloop and epilogue APIs**  
  **EN**: Provide the low-level pieces used to assemble a custom Hopper GEMM.  
  **CN**: 提供组装自定义 Hopper GEMM 所需的低层 mainloop/epilogue 组件。
- **CuTe layout primitives such as `Layout`, `Shape`, `Stride`, and `Swizzle`**  
  **EN**: Describe the shared-memory swizzle used by the epilogue.  
  **CN**: 用于描述 epilogue 采用的共享内存 swizzle 布局。
- **CUTLASS reference utilities**  
  **EN**: Initialize tensors and verify the kernel output.  
  **CN**: 用于初始化张量并验证 kernel 输出。
- **`helper.h`**  
  **EN**: Supplies CUDA error checks and timing helpers shared by examples.  
  **CN**: 提供示例共享的 CUDA 错误检查与计时辅助。
