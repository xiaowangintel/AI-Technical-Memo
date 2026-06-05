# 57_hopper_grouped_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/57_hopper_grouped_gemm/57_hopper_grouped_gemm.cu`  
**Purpose / 用途**: This sample compares Hopper grouped GEMM schedule variants, shows how grouped problems are generated or loaded from benchmark files, and demonstrates the full CUTLASS grouped-launch path including per-group alpha/beta support and correctness verification. / 该示例对比了 Hopper grouped GEMM 的多种调度方案，展示如何随机生成或从 benchmark 文件加载 grouped 问题，并演示带逐组 alpha/beta 支持和正确性验证的完整 CUTLASS grouped 启动流程。

---

## Line-by-Line Analysis / 逐行分析

The sections below cover the full source file in order. Each block includes original code and bilingual analysis.  
下面的各个小节按顺序覆盖整个源文件。每个代码块都附带原始代码与中英双语分析。

### Lines 1-61 | Overview, motivation, and usage examples / 概览、动机与使用示例

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
33:     \brief Hopper Grouped GEMM example using CUTLASS 3 APIs for NVIDIA Hopper architecture.
34: 
35:     This example demonstrates an implementation of Grouped GEMM using a TMA + GMMA
36:     warp-specialized cooperative kernel.
37:     For this example all scheduling work is performed on the device.
38:     The new feature showcased in this example is on-the-fly modification of TMA descriptors
39:     to move between groups/problem_count (represented by groups).
40: 
41:     To run this example:
42: 
43:       $ ./examples/57_hopper_grouped_gemm/57_hopper_grouped_gemm --m=2048 --n=2048 --k=2048 --groups=10
44: 
45:       The above example command makes all 10 groups to be sized at the given m, n, k sizes.
46:       Skipping any of the problem dimensions randomizes it across the different groups.
47:       Same applies for alpha and beta values that are randomized across the different groups.
48: 
49:     To run this example for a set of problems using the benchmark option:
50: 
51:       $ ./examples/57_hopper_grouped_gemm/57_hopper_grouped_gemm --benchmark=./test_benchmark.txt
52: 
53:       Where the test_benchmark.txt may look as such:
54:         0 256x512x128
55:         1 256x512x512
56:         2 512x256x128
57:         3 256x256x128
58:         4 256x512x1024
59:         5 1024x512x128 and so on
60: */
61: 
```

- **EN**: The file comment frames the sample as a Hopper grouped GEMM demonstration using device-side scheduling. It emphasizes that all work assignment happens on the GPU and gives command examples for both synthetic workloads and benchmark-driven runs.
- **CN**: 文件注释把本示例定位为 Hopper grouped GEMM 的 device-side scheduling 演示。它强调任务分配完全在 GPU 端完成，并给出了随机负载和 benchmark 驱动两种运行示例。

### Lines 62-91 | Includes and support utilities / 头文件与支持工具

```cpp
62: #include <iostream>
63: #include <fstream>
64: #include <sstream>
65: #include <vector>
66: #include <cfloat>
67: 
68: #include "cutlass/cutlass.h"
69: 
70: #include "cute/tensor.hpp"
71: #include "cutlass/tensor_ref.h"
72: #include "cutlass/epilogue/collective/default_epilogue.hpp"
73: #include "cutlass/epilogue/thread/linear_combination.h"
74: #include "cutlass/gemm/dispatch_policy.hpp"
75: #include "cutlass/gemm/group_array_problem_shape.hpp"
76: #include "cutlass/gemm/collective/collective_builder.hpp"
77: #include "cutlass/epilogue/collective/collective_builder.hpp"
78: #include "cutlass/gemm/device/gemm_universal_adapter.h"
79: #include "cutlass/gemm/kernel/gemm_universal.hpp"
80: 
81: #include "cutlass/util/command_line.h"
82: #include "cutlass/util/distribution.h"
83: #include "cutlass/util/host_tensor.h"
84: #include "cutlass/util/packed_stride.hpp"
85: #include "cutlass/util/tensor_view_io.h"
86: #include "cutlass/util/reference/device/gemm.h"
87: #include "cutlass/util/reference/device/tensor_compare.h"
88: #include "cutlass/util/reference/device/tensor_fill.h"
89: 
90: #include "helper.h"
91: 
```

- **EN**: This short block imports CUTLASS/CuTe building blocks, grouped GEMM adapters, command-line parsing, and reference helpers. It establishes everything needed to instantiate multiple schedule variants from the same host harness.
- **CN**: 这一小段引入 CUTLASS/CuTe 基础组件、grouped GEMM 适配器、命令行解析和参考实现工具，为后续在同一套 host harness 中实例化多种调度变体做好准备。

### Lines 92-180 | Problem types and schedule-specific kernel definitions / 问题类型与按调度区分的 kernel 定义

```cpp
 92: using namespace cute;
 93: using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int,int,int>>; // <M,N,K> per group
 94: using ElementA = cutlass::float_e4m3_t;                                    // Element type for A matrix operand
 95: using ElementB = cutlass::float_e5m2_t;                                    // Element type for B matrix operand
 96: using ElementC = cutlass::half_t;                                          // Element type for C and D matrix operands
 97: 
 98: #if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
 99: 
100: /////////////////////////////////////////////////////////////////////////////////////////////////
101: /// GEMM kernel configurations
102: /////////////////////////////////////////////////////////////////////////////////////////////////
103: 
104: // A matrix configuration
105: using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
106: constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Alignment of A matrix in units of elements (up to 16 bytes)
107: 
108: // B matrix configuration
109: using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
110: constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Alignment of B matrix in units of elements (up to 16 bytes)
111: 
112: // C/D matrix configuration
113: using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
114: constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Alignment of C matrix in units of elements (up to 16 bytes)
115: 
116: // Core kernel configurations
117: using ElementAccumulator  = float;                                          // Element type for internal accumulation
118: using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
119: using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
120: using StageCountType = cutlass::gemm::collective::StageCountAuto;           // Stage count maximized based on the tile size
121: 
122: // Different configs for pingpong/cooperative
123: struct CooperativeConfig {
124:   using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8FastAccum;
125:   using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
126:   using TileShape           = Shape<_256,_128,_128>;
127:   using ClusterShape        = Shape<_1,_2,_1>;
128: };
129: 
130: struct PingpongConfig {
131:   using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
132:   using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
133:   using TileShape           = Shape<_128,_128,_128>;
134:   using ClusterShape        = Shape<_2,_1,_1>;
135: };
136: 
137: template <typename ScheduleConfig>
138: struct GemmGivenSchedule {
139:   using TileShape           = typename ScheduleConfig::TileShape;                   // Threadblock-level tile size
140:   using ClusterShape        = typename ScheduleConfig::ClusterShape;                // Shape of the threadblocks in a cluster
141:   using KernelSchedule      = typename ScheduleConfig::KernelSchedule;              // Kernel to launch
142:   using EpilogueSchedule    = typename ScheduleConfig::EpilogueSchedule;            // Epilogue to launch
143: 
144:   using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
145:     cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
146:     TileShape, ClusterShape,
147:     cutlass::epilogue::collective::EpilogueTileAuto,
148:     ElementAccumulator, ElementAccumulator,
149:     ElementC, LayoutC *, AlignmentC,
150:     ElementC, LayoutC *, AlignmentC,
151:     EpilogueSchedule,
152:     cutlass::epilogue::fusion::LinearCombination<ElementC, ElementAccumulator>
153:   >::CollectiveOp;
154: 
155: using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
156:     ArchTag, OperatorClass,
157:     ElementA, LayoutA *, AlignmentA,
158:     ElementB, LayoutB *, AlignmentB,
159:     ElementAccumulator,
160:     TileShape, ClusterShape,
161:     cutlass::gemm::collective::StageCountAutoCarveout<
162:       static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
163:     KernelSchedule
164:   >::CollectiveOp;
165: 
166:   using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
167:       ProblemShape,
168:       CollectiveMainloop,
169:       CollectiveEpilogue
170:   >;
171: 
172:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
173: };
174: 
175: using GemmKernel = GemmGivenSchedule<CooperativeConfig>::GemmKernel;
176: using Gemm = GemmGivenSchedule<CooperativeConfig>::Gemm;
177: 
178: using GemmKernelPingpong = GemmGivenSchedule<PingpongConfig>::GemmKernel;
179: using GemmPingpong = GemmGivenSchedule<PingpongConfig>::Gemm;
180: 
```

- **EN**: The example defines operand types and then introduces two schedule configurations. `CooperativeConfig` uses the cooperative warp-specialized FP8-fast-accum schedule with a larger 256x128x128 tile and a `1x2x1` cluster, while `PingpongConfig` uses the pingpong schedule with a 128x128x128 tile and a `2x1x1` cluster. `GemmGivenSchedule` turns any such config into a complete grouped GEMM type.
- **CN**: 示例先定义操作数类型，然后给出两套调度配置。`CooperativeConfig` 使用 cooperative 的 warp-specialized FP8 fast-accum 调度，配合更大的 256x128x128 tile 和 `1x2x1` cluster；`PingpongConfig` 则使用 pingpong 调度，配合 128x128x128 tile 与 `2x1x1` cluster。`GemmGivenSchedule` 会把这类配置统一组装成完整的 grouped GEMM 类型。

### Lines 181-236 | Reference kernel aliases and global storage / 参考 kernel 别名与全局存储

```cpp
181: // Reference device GEMM implementation type
182: using DeviceGemmReference = cutlass::reference::device::Gemm<
183:   ElementA,
184:   LayoutA,
185:   ElementB,
186:   LayoutB,
187:   ElementC,
188:   LayoutC,
189:   ElementAccumulator,
190:   ElementAccumulator>;
191: 
192: using StrideA = typename Gemm::GemmKernel::InternalStrideA;
193: using StrideB = typename Gemm::GemmKernel::InternalStrideB;
194: using StrideC = typename Gemm::GemmKernel::InternalStrideC;
195: using StrideD = typename Gemm::GemmKernel::InternalStrideD;
196: 
197: // Host-side allocations
198: std::vector<int64_t> offset_A;
199: std::vector<int64_t> offset_B;
200: std::vector<int64_t> offset_C;
201: std::vector<int64_t> offset_D;
202: 
203: std::vector<StrideA> stride_A_host;
204: std::vector<StrideB> stride_B_host;
205: std::vector<StrideC> stride_C_host;
206: std::vector<StrideD> stride_D_host;
207: 
208: std::vector<ElementAccumulator> alpha_host;
209: std::vector<ElementAccumulator> beta_host;
210: 
211: // Device-side allocations
212: cutlass::DeviceAllocation<typename ProblemShape::UnderlyingProblemShape> problem_sizes;
213: 
214: cutlass::DeviceAllocation<typename Gemm::ElementA> block_A;
215: cutlass::DeviceAllocation<typename Gemm::ElementB> block_B;
216: cutlass::DeviceAllocation<typename Gemm::ElementC> block_C;
217: cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_D;
218: cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_ref_D;
219: 
220: cutlass::DeviceAllocation<const typename Gemm::ElementA *> ptr_A;
221: cutlass::DeviceAllocation<const typename Gemm::ElementB *> ptr_B;
222: cutlass::DeviceAllocation<const typename Gemm::ElementC *> ptr_C;
223: cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_D;
224: cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_ref_D;
225: 
226: cutlass::DeviceAllocation<StrideA> stride_A;
227: cutlass::DeviceAllocation<StrideB> stride_B;
228: cutlass::DeviceAllocation<StrideC> stride_C;
229: cutlass::DeviceAllocation<StrideD> stride_D;
230: 
231: // Note, this is an array of pointers to alpha and beta scaling values per group
232: cutlass::DeviceAllocation<ElementAccumulator*> alpha_device;
233: cutlass::DeviceAllocation<ElementAccumulator*> beta_device;
234: cutlass::DeviceAllocation<ElementAccumulator> block_alpha;
235: cutlass::DeviceAllocation<ElementAccumulator> block_beta;
236: 
```

- **EN**: After schedule construction, the file declares the device reference GEMM type and the host/device arrays used to store shapes, strides, pointers, and outputs. The structure mirrors the runtime contract expected by grouped GEMM launches.
- **CN**: 在调度类型构建完成后，文件声明了 device 参考 GEMM 类型，以及保存 shape、stride、指针和输出的主机/设备数组。这一结构与 grouped GEMM 启动所要求的运行时契约完全对应。

### Lines 237-384 | Command-line `Options` for grouped workloads / 用于 grouped 工作负载的命令行 `Options`

```cpp
237: #endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
238: 
239: /////////////////////////////////////////////////////////////////////////////////////////////////
240: /// Testbed utility types
241: /////////////////////////////////////////////////////////////////////////////////////////////////
242: 
243: // Command line options parsing
244: struct Options {
245: 
246:   bool help = false;
247: 
248:   float alpha = FLT_MAX;
249:   float beta  = FLT_MAX;
250:   int iterations = 10;
251:   int m = 1024, n = 2048, k = 512, groups = 10;
252:   std::string benchmark_path;
253:   std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host;
254:   int const tma_alignment_bits = 128;
255:   int const alignment = tma_alignment_bits / cutlass::sizeof_bits<ElementA>::value;
256: 
257:   // Parses the command line
258:   void parse(int argc, char const **args) {
259:     cutlass::CommandLine cmd(argc, args);
260: 
261:     if (cmd.check_cmd_line_flag("help")) {
262:       help = true;
263:       return;
264:     }
265: 
266:     cmd.get_cmd_line_argument("m", m);
267:     cmd.get_cmd_line_argument("n", n);
268:     cmd.get_cmd_line_argument("k", k);
269:     cmd.get_cmd_line_argument("groups", groups);
270:     cmd.get_cmd_line_argument("alpha", alpha, FLT_MAX);
271:     cmd.get_cmd_line_argument("beta",  beta,  FLT_MAX);
272:     cmd.get_cmd_line_argument("iterations", iterations);
273:     cmd.get_cmd_line_argument("benchmark", benchmark_path);
274: 
275:     // Decide how to initialize the problems
276:     if (!benchmark_path.empty()) {
277:       if (!benchmark_problems()) {
278:         problem_sizes_host.clear();
279:         return;
280:       }
281:     }
282:     else {
283:       randomize_problems(cmd);
284:     }
285:   }
286: 
287:   void randomize_problems(cutlass::CommandLine &cmd) {
288:     int cmd_line_m = -1, cmd_line_n = -1, cmd_line_k = -1;
289:     cmd.get_cmd_line_argument("m", cmd_line_m);
290:     cmd.get_cmd_line_argument("n", cmd_line_n);
291:     cmd.get_cmd_line_argument("k", cmd_line_k);
292: 
293:     problem_sizes_host.reserve(groups);
294: 
295:     for (int i = groups; i > 0; i--) {
296:       int m = cmd_line_m;
297:       int n = cmd_line_n;
298:       int k = cmd_line_k;
299:       if (m < 0) {
300:         m = alignment * ((rand() % 64));
301:       }
302:       if (n < 0) {
303:         n = alignment * ((rand() % 64));
304:       }
305:       if (k < 0) {
306:         k = alignment * ((rand() % 64));
307:       }
308:       problem_sizes_host.push_back({m, n, k});
309:     }
310:   }
311: 
312:   /// Load a benchmark
313:   bool benchmark_problems() {
314:     std::ifstream file(benchmark_path);
315:     if (!file.good()) {
316:       return false;
317:     }
318: 
319:     while (file.good()) {
320: 
321:       int idx = -1;
322:       std::string extent_str;
323: 
324:       file >> idx >> extent_str;
325: 
326:       if (idx < 0 || extent_str.empty()) {
327:         break;
328:       }
329: 
330:       cutlass::gemm::GemmCoord extent;
331:       std::vector<std::string> tokens;
332: 
333:       cutlass::CommandLine::tokenize(tokens, extent_str, 'x');
334: 
335:       for (int i = 0; i < int(tokens.size()); ++i) {
336:         extent.at(i) = std::atoi(tokens.at(i).c_str());
337:       }
338:       problem_sizes_host.push_back({extent.m(), extent.n(), extent.k()});
339:     }
340:     groups = static_cast<int>(problem_sizes_host.size());
341: 
342:     return true;
343:   }
344: 
345:   /// Prints the usage statement.
346:   std::ostream & print_usage(std::ostream &out) const {
347: 
348:     out << "57_hopper_grouped_gemm\n\n"
349:       << "  Hopper FP8 Grouped GEMM using a Warp Specialized kernel.\n\n"
350:       << "Options:\n\n"
351:       << "  --help                      If specified, displays this usage statement\n\n"
352:       << "  --m=<int>                   Sets the M extent of the GEMM for all groups\n"
353:       << "  --n=<int>                   Sets the N extent of the GEMM for all groups\n"
354:       << "  --k=<int>                   Sets the K extent of the GEMM for all groups\n"
355:       << "  --groups=<int>              Sets the number of individual GEMM problems for Grouped GEMM\n"
356:       << "  --alpha=<f32>               Epilogue scalar alpha\n"
357:       << "  --beta=<f32>                Epilogue scalar beta\n\n"
358:       << "  --iterations=<int>          Number of profiling iterations to perform\n\n"
359:       << "  --benchmark=<str>           Executes a benchmark problem size.\n";
360: 
361:     out
362:       << "\n\nExamples:\n\n"
363:       << "$ " << "57_hopper_grouped_gemm" << " --m=1024 --n=512 --k=1024 --groups=10 --alpha=2 --beta=0.707 \n\n";
364: 
365:     return out;
366:   }
367: 
368:   /// Compute performance in GFLOP/s
369:   double gflops(double runtime_s, std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host) const
370:   {
371:     // Number of real-valued multiply-adds
372:     uint64_t fmas = uint64_t();
373: 
374:     for (auto const & problem : problem_sizes_host) {
375:       fmas += static_cast<uint64_t>(get<0>(problem)) *
376:               static_cast<uint64_t>(get<1>(problem)) *
377:               static_cast<uint64_t>(get<2>(problem));
378:     }
379:     // Two flops per multiply-add
380:     uint64_t flop = uint64_t(2) * uint64_t(fmas);
381:     double gflop = double(flop) / double(1.0e9);
382:     return gflop / runtime_s;
383:   }
384: };
```

- **EN**: The options parser supports fixed M/N/K extents, group count, optional scalar alpha/beta, iteration count, and a benchmark file. It also computes total GFLOPs across all groups, which is the correct throughput metric for grouped workloads because each group can have a different shape.
- **CN**: `Options` 解析器支持固定的 M/N/K、group 数量、可选的统一 alpha/beta、迭代次数以及 benchmark 文件。它还会计算所有 group 的总 GFLOPs，这才是 grouped 负载下正确的吞吐指标，因为每个 group 的形状都可能不同。

### Lines 385-426 | Result container and random-fill helper / 结果容器与随机填充辅助函数

```cpp
385: 
386: /// Result structure
387: struct Result
388: {
389:   double avg_runtime_ms = 0.0;
390:   double gflops = 0.0;
391:   cutlass::Status status = cutlass::Status::kSuccess;
392:   cudaError_t error = cudaSuccess;
393:   bool passed = false;
394: };
395: 
396: #if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
397: 
398: /////////////////////////////////////////////////////////////////////////////////////////////////
399: /// GEMM setup and evaluation
400: /////////////////////////////////////////////////////////////////////////////////////////////////
401: 
402: /// Helper to initialize a block of device data
403: template <class Element>
404: bool initialize_block(
405:   cutlass::DeviceAllocation<Element>& block,
406:   uint64_t seed=2023) {
407: 
408:   Element scope_max, scope_min;
409:   int bits_input = cutlass::sizeof_bits<Element>::value;
410: 
411:   if (bits_input == 1) {
412:     scope_max = static_cast<Element>(2);
413:     scope_min = static_cast<Element>(0);
414:   } else if (bits_input <= 8) {
415:     scope_max = static_cast<Element>(2);
416:     scope_min = static_cast<Element>(-2);
417:   } else {
418:     scope_max = static_cast<Element>(8);
419:     scope_min = static_cast<Element>(-8);
420:   }
421: 
422:   cutlass::reference::device::BlockFillRandomUniform(
423:     block.get(), block.size(), seed, scope_max, scope_min, 0);
424: 
425:   return true;
426: }
```

- **EN**: This block mirrors other samples: a lightweight result record plus a data-initialization helper that chooses ranges based on element precision so the benchmark starts from realistic tensor contents.
- **CN**: 这一段延续了其他示例的模式：先定义轻量级结果结构，再提供一个依据元素精度选择数值范围的数据初始化函数，使 benchmark 从较真实的张量内容开始。

### Lines 427-471 | Per-group offset computation and allocation / 逐组偏移计算与分配

```cpp
427: 
428: /// Allocates device-side data
429: void allocate(const Options &options) {
430:   int64_t total_elements_A = 0;
431:   int64_t total_elements_B = 0;
432:   int64_t total_elements_C = 0;
433:   int64_t total_elements_D = 0;
434: 
435:   for (int32_t i = 0; i < options.groups; ++i) {
436: 
437:     auto problem = options.problem_sizes_host.at(i);
438:     auto M = get<0>(problem);
439:     auto N = get<1>(problem);
440:     auto K = get<2>(problem);
441: 
442:     offset_A.push_back(total_elements_A);
443:     offset_B.push_back(total_elements_B);
444:     offset_C.push_back(total_elements_C);
445:     offset_D.push_back(total_elements_D);
446: 
447:     int64_t elements_A = M * K;
448:     int64_t elements_B = K * N;
449:     int64_t elements_C = M * N;
450:     int64_t elements_D = M * N;
451: 
452:     total_elements_A += elements_A;
453:     total_elements_B += elements_B;
454:     total_elements_C += elements_C;
455:     total_elements_D += elements_D;
456: 
457:     stride_A_host.push_back(cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1}));
458:     stride_B_host.push_back(cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1}));
459:     stride_C_host.push_back(cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1}));
460:     stride_D_host.push_back(cutlass::make_cute_packed_stride(StrideD{}, {M, N, 1}));
461: 
462:   }
463: 
464:   block_A.reset(total_elements_A);
465:   block_B.reset(total_elements_B);
466:   block_C.reset(total_elements_C);
467:   block_D.reset(total_elements_D);
468:   block_ref_D.reset(total_elements_D);
469:   block_alpha.reset(options.groups);
470:   block_beta.reset(options.groups);
471: }
```

- **EN**: Each group receives packed offsets into the contiguous A/B/C/D backing stores. The function allocates storage for tensor data, the reference output, and per-group alpha/beta arrays so later code can choose either scalar or group-specific epilogues.
- **CN**: 每个 group 都会在连续的 A/B/C/D 存储区中获得自己的打包偏移。该函数分配张量数据、参考输出以及逐组 alpha/beta 数组，从而让后续代码可以在统一标量 epilogue 和逐组 epilogue 之间切换。

### Lines 472-556 | Initialization of problem metadata and tensor pointers / 问题元数据与张量指针初始化

```cpp
472: 
473: /// Initialize operands to be used in the GEMM and reference GEMM
474: void initialize(const Options &options) {
475: 
476:   uint64_t seed = 2020;
477: 
478:   problem_sizes.reset(options.groups);
479:   problem_sizes.copy_from_host(options.problem_sizes_host.data());
480: 
481:   //
482:   // Assign pointers
483:   //
484: 
485:   std::vector<ElementA *> ptr_A_host(options.groups);
486:   std::vector<ElementB *> ptr_B_host(options.groups);
487:   std::vector<ElementC *> ptr_C_host(options.groups);
488:   std::vector<ElementC *> ptr_D_host(options.groups);
489:   std::vector<ElementAccumulator *> ptr_alpha_host(options.groups);
490:   std::vector<ElementAccumulator *> ptr_beta_host(options.groups);
491: 
492:   for (int32_t i = 0; i < options.groups; ++i) {
493:     // If the current group's matrix has size 0, set the pointer to nullptr
494:     if (i < options.groups - 1 && offset_A.at(i) == offset_A.at(i + 1)) {
495:       ptr_A_host.at(i) = nullptr;
496:     } else {
497:       ptr_A_host.at(i) = block_A.get() + offset_A.at(i);
498:     }
499:     if (i < options.groups - 1 && offset_B.at(i) == offset_B.at(i + 1)) {
500:       ptr_B_host.at(i) = nullptr;
501:     } else {
502:       ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
503:     }
504:     if (i < options.groups - 1 && offset_C.at(i) == offset_C.at(i + 1)) {
505:       ptr_C_host.at(i) = nullptr;
506:     } else {
507:       ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
508:     }
509:     if (i < options.groups - 1 && offset_D.at(i) == offset_D.at(i + 1)) {
510:       ptr_D_host.at(i) = nullptr;
511:     } else {
512:       ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
513:     }
514:     alpha_host.push_back((options.alpha == FLT_MAX) ? static_cast<ElementAccumulator>((rand() % 5) + 1) : options.alpha);
515:     beta_host.push_back((options.beta == FLT_MAX) ? static_cast<ElementAccumulator>(rand() % 5) : options.beta);
516:     ptr_alpha_host.at(i) = block_alpha.get() + i;
517:     ptr_beta_host.at(i) = block_beta.get() + i;
518:   }
519: 
520:   ptr_A.reset(options.groups);
521:   ptr_A.copy_from_host(ptr_A_host.data());
522: 
523:   ptr_B.reset(options.groups);
524:   ptr_B.copy_from_host(ptr_B_host.data());
525: 
526:   ptr_C.reset(options.groups);
527:   ptr_C.copy_from_host(ptr_C_host.data());
528: 
529:   ptr_D.reset(options.groups);
530:   ptr_D.copy_from_host(ptr_D_host.data());
531: 
532:   stride_A.reset(options.groups);
533:   stride_A.copy_from_host(stride_A_host.data());
534: 
535:   stride_B.reset(options.groups);
536:   stride_B.copy_from_host(stride_B_host.data());
537: 
538:   stride_C.reset(options.groups);
539:   stride_C.copy_from_host(stride_C_host.data());
540: 
541:   stride_D.reset(options.groups);
542:   stride_D.copy_from_host(stride_D_host.data());
543: 
544:   alpha_device.reset(options.groups);
545:   alpha_device.copy_from_host(ptr_alpha_host.data());
546:   beta_device.reset(options.groups);
547:   beta_device.copy_from_host(ptr_beta_host.data());
548: 
549:   initialize_block(block_A, seed + 2021);
550:   initialize_block(block_B, seed + 2022);
551:   initialize_block(block_C, seed + 2023);
552:   initialize_block(block_D, seed + 2024);
553:   block_alpha.copy_from_host(alpha_host.data());
554:   block_beta.copy_from_host(beta_host.data());
555: }
556: 
```

- **EN**: The initialization path copies problem shapes to the GPU, constructs per-group pointers, emits `nullptr` for empty problems, fills tensors, and uploads alpha/beta data. This is the practical host-side preparation step for device-side grouped scheduling.
- **CN**: 初始化路径会把 problem shape 拷贝到 GPU，构造逐组指针，对空问题写入 `nullptr`，填充张量并上传 alpha/beta 数据。这正是 device-side grouped scheduling 在 host 端需要完成的准备工作。

### Lines 557-614 | Building grouped arguments / 构造 grouped 参数

```cpp
557: /// Populates a Gemm::Arguments structure from the given commandline options
558: template <typename GemmT>
559: typename GemmT::Arguments args_from_options(const Options &options, bool host_problem_shapes_available = true)
560: {
561:   // Change device_id to another value if you are running on a machine with multiple GPUs and wish
562:   // to use a GPU other than that with device ID 0.
563:   int device_id = 0;
564:   cutlass::KernelHardwareInfo kernel_hw_info = cutlass::KernelHardwareInfo::make_kernel_hardware_info<Gemm::GemmKernel>(device_id);
565: 
566:   typename GemmT::Arguments arguments;
567:   decltype(arguments.epilogue.thread) fusion_args;
568: 
569:   if (options.alpha != FLT_MAX && options.beta != FLT_MAX) {
570:     // If both alpha/beta are provided (via cmd line args) and are scalar, i.e., same alpha/beta applies to all batches.
571:     fusion_args.alpha = options.alpha;
572:     fusion_args.beta = options.beta;
573:     fusion_args.alpha_ptr = nullptr;
574:     fusion_args.beta_ptr = nullptr;
575:     fusion_args.alpha_ptr_array = nullptr;
576:     fusion_args.beta_ptr_array = nullptr;
577:     // Single alpha and beta for all groups
578:     fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 0};
579:     fusion_args.dBeta = {cute::_0{}, cute::_0{}, 0};
580:   }
581:   else {
582:     // If pointers to alpha/beta are provided, i.e., alpha/beta can differ between batches/groups.
583:     fusion_args.alpha = 0;
584:     fusion_args.beta = 0;
585:     fusion_args.alpha_ptr = nullptr;
586:     fusion_args.beta_ptr = nullptr;
587:     fusion_args.alpha_ptr_array = alpha_device.get();
588:     fusion_args.beta_ptr_array = beta_device.get();
589:     // One alpha and beta per each group
590:     fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 1};
591:     fusion_args.dBeta = {cute::_0{}, cute::_0{}, 1};
592:   }
593: 
594:   if (host_problem_shapes_available) {
595:     arguments = typename GemmT::Arguments {
596:       cutlass::gemm::GemmUniversalMode::kGrouped,
597:       {options.groups, problem_sizes.get(), options.problem_sizes_host.data()},
598:       {ptr_A.get(), stride_A.get(), ptr_B.get(), stride_B.get()},
599:       {fusion_args, ptr_C.get(), stride_C.get(), ptr_D.get(), stride_D.get()},
600:       kernel_hw_info
601:     };
602:   }
603:   else {
604:     arguments = typename GemmT::Arguments {
605:       cutlass::gemm::GemmUniversalMode::kGrouped,
606:       {options.groups, problem_sizes.get(), nullptr},
607:       {ptr_A.get(), stride_A.get(), ptr_B.get(), stride_B.get()},
608:       {fusion_args, ptr_C.get(), stride_C.get(), ptr_D.get(), stride_D.get()},
609:       kernel_hw_info
610:     };
611:   }
612: 
613:   return arguments;
614: }
```

- **EN**: This function packages the launch arguments for either schedule variant. The key idea is that the grouped kernel interface is stable even though the schedule changes: only the instantiated `GemmT` type differs, while the host code still passes grouped mode, problem-shape arrays, pointer arrays, stride arrays, and epilogue fusion parameters.
- **CN**: 该函数为不同调度变体打包启动参数。关键点在于：即使 schedule 改变，grouped kernel 接口仍然稳定——变化的只是实例化出来的 `GemmT` 类型，而 host 侧依旧传入 grouped mode、problem-shape 数组、指针数组、stride 数组以及 epilogue 融合参数。

### Lines 615-654 | Device-reference verification / 设备参考实现验证

```cpp
615: 
616: bool verify(const Options &options) {
617:   bool passed = true;
618:   for (int32_t i = 0; i < options.groups; ++i) {
619:     auto problem = options.problem_sizes_host.at(i);
620:     auto M = get<0>(problem);
621:     auto N = get<1>(problem);
622:     auto K = get<2>(problem);
623:     cutlass::TensorRef ref_A(block_A.get() + offset_A.at(i), Gemm::LayoutA::packed({M, K}));
624:     cutlass::TensorRef ref_B(block_B.get() + offset_B.at(i), Gemm::LayoutB::packed({K, N}));
625:     cutlass::TensorRef ref_C(block_C.get() + offset_C.at(i), Gemm::LayoutC::packed({M, N}));
626:     cutlass::TensorRef ref_D(block_ref_D.get() + offset_D.at(i), Gemm::LayoutD::packed({M, N}));
627: 
628:     //
629:     // Compute reference output
630:     //
631: 
632:     // Create instantiation for device reference gemm kernel
633:     DeviceGemmReference gemm_reference;
634: 
635:     // Launch device reference gemm kernel
636:     gemm_reference(
637:       {M, N, K},
638:       ElementAccumulator(alpha_host.at(i)),
639:       ref_A,
640:       ref_B,
641:       ElementAccumulator(beta_host.at(i)),
642:       ref_C,
643:       ref_D);
644: 
645:     // Wait for kernel to finish
646:     CUDA_CHECK(cudaDeviceSynchronize());
647: 
648:     // Check if output from CUTLASS kernel and reference kernel are equal or not
649:     passed &= cutlass::reference::device::BlockCompareEqual(block_ref_D.get() + offset_D.at(i), block_D.get() + offset_D.at(i), M * N);
650:     #if 0
651:     std::cout << "Group: " << i << " Status: " << passed << std::endl;
652:     #endif
653:   }
654:   return passed;
```

- **EN**: To validate correctness, the code launches a reference GEMM for every group and compares the resulting D tensor with the grouped-kernel output. This decouples algorithmic correctness from schedule choice.
- **CN**: 为验证正确性，代码会为每个 group 启动参考 GEMM，并将得到的 D 张量与 grouped kernel 的输出进行比较。这样就把“算法正确性”与“调度选择”清晰地分离开来。

### Lines 655-723 | End-to-end run flow / 端到端运行流程

```cpp
655: }
656: 
657: /// Execute a given example GEMM computation
658: template <typename GemmT>
659: int run(Options &options, bool host_problem_shapes_available = true)
660: {
661:   allocate(options);
662:   initialize(options);
663: 
664:   std::cout << "  Problem Sizes, Alpha, Beta " << std::endl;
665:   for (int32_t i = 0; i < options.groups; ++i) {
666:     std::cout << "    " << options.problem_sizes_host.at(i);
667:     std::cout << ", "   << alpha_host.at(i) << ", " << beta_host.at(i) << std::endl;
668:   }
669:   std::cout << "  Groups      : " << options.groups  << std::endl;
670: 
671:   // Instantiate CUTLASS kernel depending on templates
672:   GemmT gemm;
673: 
674:   // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
675:   auto arguments = args_from_options<GemmT>(options, host_problem_shapes_available);
676: 
677:   // Using the arguments, query for extra workspace required for matrix multiplication computation
678:   size_t workspace_size = GemmT::get_workspace_size(arguments);
679: 
680:   // Allocate workspace memory
681:   cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
682: 
683:   // Check if the problem size is supported or not
684:   CUTLASS_CHECK(gemm.can_implement(arguments));
685: 
686:   // Initialize CUTLASS kernel with arguments and workspace pointer
687:   CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
688: 
689:   // Correctness / Warmup iteration
690:   CUTLASS_CHECK(gemm.run());
691: 
692:   // Check if output from CUTLASS kernel and reference kernel are equal or not
693:   Result result;
694:   result.passed = verify(options);
695: 
696:   std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
697: 
698:   if (!result.passed) {
699:     exit(-1);
700:   }
701: 
702:   // Run profiling loop
703:   if (options.iterations > 0)
704:   {
705:     GpuTimer timer;
706:     timer.start();
707:     for (int iter = 0; iter < options.iterations; ++iter) {
708:       CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
709:       CUTLASS_CHECK(gemm.run());
710:     }
711:     timer.stop();
712: 
713:     // Compute average setup and runtime and GFLOPs.
714:     float elapsed_ms       = timer.elapsed_millis();
715:     result.avg_runtime_ms  = double(elapsed_ms) / double(options.iterations);
716:     result.gflops          = options.gflops(result.avg_runtime_ms / 1000.0, options.problem_sizes_host);
717: 
718:     std::cout << "  Avg runtime : " << result.avg_runtime_ms << " ms" << std::endl;
719:     std::cout << "  TFLOPS      : " << result.gflops / 1000.0 << std::endl;
720:   }
721: 
722:   return 0;
723: }
```

- **EN**: The `run()` function allocates data, initializes metadata, constructs the selected grouped kernel, queries workspace, performs capability checks, launches once for correctness, then profiles multiple iterations. It is the clearest summary of the CUTLASS grouped-GEMM runtime lifecycle.
- **CN**: `run()` 函数负责分配数据、初始化元数据、构造选定的 grouped kernel、查询 workspace、执行能力检查、先跑一次正确性验证，再进行多轮 profiling。它最完整地展示了 CUTLASS grouped GEMM 的运行时生命周期。

### Lines 724-783 | Program entry and schedule comparison / 程序入口与调度对比

```cpp
724: 
725: #endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
726: 
727: ///////////////////////////////////////////////////////////////////////////////////////////////////
728: 
729: int main(int argc, char const **args) {
730: 
731:   // CUTLASS must be compiled with CUDA 12.3 Toolkit to run this example
732:   if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 3)) {
733:     std::cerr << "This example requires CUDA 12.3 or newer.\n";
734:     // Returning zero so this test passes on older Toolkits. Its actions are no-op.
735:     return 0;
736:   }
737: 
738:   cudaDeviceProp props;
739:   int current_device_id;
740:   CUDA_CHECK(cudaGetDevice(&current_device_id));
741:   CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
742:   cudaError_t error = cudaGetDeviceProperties(&props, 0);
743:   if (props.major != 9 || props.minor != 0) {
744:     std::cerr
745:       << "This example requires a GPU of NVIDIA's Hopper Architecture (compute capability 90).\n";
746:     return 0;
747:   }
748: 
749: 
750: 
751: 
752:   //
753:   // Parse options
754:   //
755: 
756:   Options options;
757: 
758:   options.parse(argc, args);
759: 
760:   if (options.help) {
761:     options.print_usage(std::cout) << std::endl;
762:     return 0;
763:   }
764: 
765:   //
766:   // Evaluate CUTLASS kernels
767:   //
768: 
769: #if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
770:   std::cout << "\n*** Cooperative schedule ***" << std::endl;
771:   run<Gemm>(options);
772:   std::cout << "\n*** Cooperative schedule (host problem shapes unavailable) ***" << std::endl;
773:   run<Gemm>(options, false /*host_problem_shapes_available*/);
774:   std::cout << "\n*** Pingpong schedule ***" << std::endl;
775:   run<GemmPingpong>(options);
776:   std::cout << "\n*** Pingpong schedule (host problem shapes unavailable) ***" << std::endl;
777:   run<GemmPingpong>(options, false /*host_problem_shapes_available*/);
778: #endif
779: 
780:   return 0;
781: }
782: 
783: /////////////////////////////////////////////////////////////////////////////////////////////////
```

- **EN**: The entry point checks Hopper requirements, parses options, and then runs both cooperative and pingpong schedules with and without host problem-shape arrays. This makes the example a direct schedule-comparison harness rather than a single fixed benchmark.
- **CN**: 程序入口先检查 Hopper 前提并解析参数，然后分别在“有/无 host problem-shape 数组”两种模式下运行 cooperative 和 pingpong 调度。因此这个示例本质上是一个可直接比较调度差异的实验 harness，而不是单一固定 benchmark。

---

## Key Concepts / 关键概念

- **EN**: Cooperative vs. pingpong grouped schedulers: both are warp-specialized Hopper schedules, but they use different tile/cluster organizations and different kernel/epilogue schedule tags.
  **CN**: Cooperative 与 pingpong grouped scheduler：二者都是 Hopper 的 warp-specialized 调度，但采用不同的 tile/cluster 组织和不同的 kernel/epilogue schedule tag。
- **EN**: Device-side grouped scheduling means the GPU consumes arrays of problems, pointers, and strides and decides which group/tile to execute.
  **CN**: Device-side grouped scheduling 表示 GPU 直接消费 problem、指针和 stride 数组，并自行决定执行哪个 group/tile。
- **EN**: A stable host harness can benchmark multiple schedule variants simply by swapping the instantiated `Gemm` type.
  **CN**: 通过替换实例化的 `Gemm` 类型，同一套稳定的 host harness 就能对比多个调度变体。
- **EN**: Grouped throughput must aggregate work over all groups rather than using a single `m*n*k` formula.
  **CN**: grouped 吞吐必须对所有 group 的工作量求和，而不能只套用单个 `m*n*k` 公式。

## Dependencies / 依赖项

- **`cutlass/gemm/*`, `cutlass/epilogue/*`**  
  **EN**: Grouped GEMM kernel builders, schedule tags, and the universal adapter.  
  **CN**: 提供 grouped GEMM kernel builder、schedule tag 和通用适配器。
- **`cutlass/util/command_line.h`**  
  **EN**: Parses benchmark parameters and benchmark-file paths.  
  **CN**: 负责解析 benchmark 参数与 benchmark 文件路径。
- **CUTLASS device reference GEMM utilities**  
  **EN**: Used to validate grouped outputs on the GPU.  
  **CN**: 用于在 GPU 上验证 grouped 输出。
- **`helper.h`**  
  **EN**: Provides CUDA error checking and timing helpers shared by examples.  
  **CN**: 提供示例共享的 CUDA 错误检查与计时辅助。
