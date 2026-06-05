# 68_hopper_fp8_warp_specialized_grouped_gemm_with_blockwise_scaling.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/68_hopper_fp8_warp_specialized_grouped_gemm_with_blockwise_scaling/68_hopper_fp8_warp_specialized_grouped_gemm_with_blockwise_scaling.cu`  
**Purpose / 用途**: This example builds a Hopper SM90 grouped GEMM for FP8 tensors, adds blockwise scaling metadata for A and B, exposes raster-order and swizzle scheduler controls, and validates the kernel against a host-side reference path. / 该示例构建了一个面向 Hopper SM90 的 FP8 grouped GEMM，为 A/B 张量加入块级缩放元数据，暴露 raster order 与 swizzle 调度控制，并通过主机侧参考实现验证 kernel 结果。

---

## Line-by-Line Analysis / 逐行分析

The sections below cover the full source file in order. Each block includes original code and bilingual analysis.  
下面的各个小节按顺序覆盖整个源文件。每个代码块都附带原始代码与中英双语分析。

### Lines 1-59 | File overview and usage / 文件概览与用法

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
33:     \brief Grouped scale Hopper FP8 Grouped GEMM example using CUTLASS 3.0 APIs for NVIDIA Hopper architecture
34:     This example demonstrates a grouped scaled FP8 Grouped GEMM using the new CUTLASS 3.0.
35:     APIs on NVIDIA Hopper architecture. New features that will be showcased in this example are as follows:
36:     1. NVIDIA Hopper architecture introduces a new series of tensor core instructions (GMMA)
37:     which are more efficient than the Ampere tensor core instructions.
38:     2. NVIDIA Hopper architecture includes new Tensor Memory Accelerator (TMA) unit to transfer large
39:     blocks of data efficiently between global memory and shared memory. TMA also supports asynchronous
40:     copies between thread blocks in a cluster. This example also showcases on-the-fly modification of TMA
41:     descriptors to move between groups/problem_count (represented by groups).
42:     3. This example uses the Warp Specialized kernel design (see /media/docs/efficient_gemm.md for details).
43:     4. A simple way to tune the CTA rasterization direction and swizzle pattern of Hopper kernels. Both the
44:     CTA rasterization direction and swizzle pattern impact cross-CTA locality of accesses. By tuning we can
45:     improve performance.
46:     Examples:
47:       $ ./examples/68_hopper_fp8_warp_specialized_grouped_gemm_with_blockwise_scaling/68_hopper_fp8_warp_specialized_grouped_gemm_with_blockwise_scaling  \
48:         --m=2816 --n=3072 --k=16384 --save_aux=false --save_amax=false \
49:         --raster=h --swizzle=2 --benchmark=./test_benchmark.txt
50: 
51:       Where the test_benchmark.txt may look as such:
52:         0 256x512x128
53:         1 256x512x512
54:         2 512x256x128
55:         3 256x256x128
56:         4 256x512x1024
57:         5 1024x512x128 and so on
58: */
59: 
```

- **EN**: The opening comment explains what the sample demonstrates: Hopper warp-specialized grouped GEMM with FP8 inputs and blockwise scaling. It also records the required architecture/toolchain assumptions and gives example command lines so the rest of the file can focus on implementation details.
- **CN**: 开头注释说明了示例的目标：在 Hopper 上实现带块级缩放的 FP8 warp-specialized grouped GEMM。这里还写明了架构/工具链前提与示例命令，因此后续代码可以直接聚焦到实现细节。

### Lines 60-99 | Includes, aliases, and feature guards / 头文件、别名与特性开关

```cpp
60: #include <iostream>
61: #include <optional>
62: #include <fstream>
63: #include <sstream>
64: #include <vector>
65: #include <cfloat>
66: 
67: #include "cutlass/cutlass.h"
68: #include "cutlass/numeric_types.h"
69: 
70: #include "cute/tensor.hpp"
71: #include "cutlass/tensor_ref.h"
72: #include "cutlass/gemm/dispatch_policy.hpp"
73: #include "cutlass/gemm/collective/collective_builder.hpp"
74: #include "cutlass/gemm/device/gemm_universal_adapter.h"
75: #include "cutlass/gemm/kernel/gemm_universal.hpp"
76: #include "cutlass/gemm/kernel/tile_scheduler_params.h"
77: #include "cutlass/epilogue/dispatch_policy.hpp"
78: #include "cutlass/epilogue/collective/collective_builder.hpp"
79: 
80: #include "cutlass/util/command_line.h"
81: #include "cutlass/util/distribution.h"
82: #include "cutlass/util/host_tensor.h"
83: #include "cutlass/util/packed_stride.hpp"
84: #include "cutlass/util/tensor_view_io.h"
85: #include "cutlass/util/reference/host/tensor_fill.h"
86: #include "cutlass/util/reference/host/tensor_copy.h"
87: #include "cutlass/util/reference/host/tensor_compare.h"
88: #include "cutlass/util/reference/host/tensor_norm.h"
89: #include "cutlass/util/reference/device/tensor_fill.h"
90: #include "cutlass/util/reference/host/gett.hpp"
91: 
92: // Includes from examples directory
93: #include "helper.h"
94: #include "hopper_fp8_commandline.hpp"
95: 
96: using namespace cute;
97: 
98: using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int,int,int>>; // <M,N,K> per group
99: 
```

- **EN**: This block pulls in CUTLASS, CuTe, helper utilities, and the shared Hopper FP8 command-line parser. It also introduces the grouped problem-shape alias and the compile-time guard that keeps the example limited to SM90 features with modifiable TMA descriptors.
- **CN**: 这一段引入 CUTLASS、CuTe、辅助工具以及共享的 Hopper FP8 命令行解析头文件；同时定义 grouped problem shape 别名，并通过编译期开关把示例限制在支持可修改 TMA 描述符的 SM90 能力范围内。

### Lines 100-180 | Kernel data types and collective assembly / Kernel 数据类型与 collective 组装

```cpp
100: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
101: 
102: /////////////////////////////////////////////////////////////////////////////////////////////////
103: /// GEMM kernel configurations
104: /////////////////////////////////////////////////////////////////////////////////////////////////
105: 
106: // A matrix configuration
107: using         ElementA    = cutlass::float_e4m3_t;                          // Element type for A matrix operand
108: using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
109: constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)
110: 
111: // B matrix configuration
112: using         ElementB    = cutlass::float_e4m3_t;                          // Element type for B matrix operand
113: using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
114: constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)
115: 
116: // C matrix configuration
117: using         ElementC    = cutlass::float_e4m3_t;                          // Element type for C and D matrix operands
118: using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
119: constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)
120: 
121: // D matrix configuration
122: using         ElementD    = ElementC;
123: using         LayoutD     = LayoutC;
124: constexpr int AlignmentD  = AlignmentC;
125: 
126: // Core kernel configurations
127: using ElementAccumulator  = float;                                          // Element type for internal accumulation
128: using ElementBlockScale   = float;                                          // Element type for blockscaling during accumulation
129: using ElementCompute      = float;                                          // Element type for epilogue computation
130: 
131: using ArchTag       = cutlass::arch::Sm90;                          // Tag indicating the minimum SM that supports the intended feature
132: using OperatorClass = cutlass::arch::OpClassTensorOp;               // Operator class tag
133: using TileShape     = Shape<_128,_128,_128>;                        // Threadblock-level tile size
134: using ClusterShape  = Shape<_1,_2,_1>;                              // Shape of the threadblocks in a cluster
135: 
136: constexpr int ScaleGranularityM = 1;
137: constexpr int ScaleGranularityN = 128;
138: constexpr int ScaleGranularityK = 128;
139: 
140: constexpr int ScaleMsPerTile = size<0>(TileShape{}) / ScaleGranularityM;
141: constexpr int ScaleNsPerTile = size<1>(TileShape{}) / ScaleGranularityN;
142: 
143: using ScaleConfig   = cutlass::detail::Sm90BlockwiseScaleConfig<ScaleGranularityM, ScaleGranularityN, ScaleGranularityK>;
144: 
145: using LayoutSFA     = decltype(ScaleConfig::deduce_layoutSFA());    // Layout type for SFA matrix operand
146: using LayoutSFB     = decltype(ScaleConfig::deduce_layoutSFB());    // Layout type for SFB matrix operand
147: 
148: using KernelSchedule    = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise;
149: using EpilogueSchedule  = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
150: using EpilogueTileType  = cutlass::epilogue::collective::EpilogueTileAuto;
151: using FusionOperation   = cutlass::epilogue::fusion::LinearCombination<ElementC, ElementAccumulator>;
152: 
153: using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
154:     ArchTag, OperatorClass,
155:     TileShape, ClusterShape,
156:     EpilogueTileType,
157:     ElementAccumulator, ElementCompute,
158:     ElementC, LayoutC *, AlignmentC,
159:     ElementD, LayoutD *, AlignmentD,
160:     EpilogueSchedule,
161:     FusionOperation
162:   >::CollectiveOp;
163: 
164: using CollectiveMainloopWithGroupWiseScaling = typename cutlass::gemm::collective::CollectiveBuilder<
165:     ArchTag, OperatorClass,
166:     ElementA, cute::tuple<LayoutA *, LayoutSFA *>, AlignmentA,
167:     ElementB, cute::tuple<LayoutB *, LayoutSFB *>, AlignmentB,
168:     ElementAccumulator,
169:     TileShape, ClusterShape,
170:     cutlass::gemm::collective::StageCountAutoCarveout<
171:       static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
172:     >,
173:     KernelSchedule
174:   >::CollectiveOp;
175: 
176: using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
177:     ProblemShape,
178:     CollectiveMainloopWithGroupWiseScaling,
179:     CollectiveEpilogue
180:   >;
```

- **EN**: Here the example fixes the operand layouts, FP8 element types, accumulation type, tile shape, cluster shape, and block-scale granularity. The most important part is the use of `Sm90BlockwiseScaleConfig` plus tuple-based A/B operands so the mainloop consumes both matrix pointers and block-scale tensor pointers; the epilogue remains a standard linear-combination fusion op.
- **CN**: 这里确定了操作数布局、FP8 元素类型、累加类型、tile/cluster 形状以及块级缩放粒度。最关键的是通过 `Sm90BlockwiseScaleConfig` 与基于 tuple 的 A/B 操作数描述，让 mainloop 同时消费矩阵指针和 block-scale 张量指针；epilogue 则保持为标准的线性组合融合操作。

### Lines 181-246 | Device adapter, stride aliases, and global state / Device 适配器、stride 别名与全局状态

```cpp
181: 
182: using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
183: 
184: 
185: // Extract information from Gemm kernel.
186: using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
187: using ElementScalar     = typename EpilogueOutputOp::ElementScalar;
188: 
189: using StrideA = typename Gemm::GemmKernel::InternalStrideA;
190: using StrideB = typename Gemm::GemmKernel::InternalStrideB;
191: using StrideC = typename Gemm::GemmKernel::InternalStrideC;
192: using StrideD = typename Gemm::GemmKernel::InternalStrideD;
193: 
194: static_assert(cute::is_same_v<ElementAccumulator, ElementBlockScale>,
195:              "ElementAccumulator and ElementBlockScale should be same datatype");
196: 
197: /// Initialization
198: 
199: cutlass::DeviceAllocation<typename ProblemShape::UnderlyingProblemShape> problem_sizes;
200: 
201: std::vector<int64_t> offset_A;
202: std::vector<int64_t> offset_B;
203: std::vector<int64_t> offset_C;
204: std::vector<int64_t> offset_D;
205: std::vector<int64_t> offset_blockscale_A;
206: std::vector<int64_t> offset_blockscale_B;
207: 
208: std::vector<StrideA> stride_A_host;
209: std::vector<StrideB> stride_B_host;
210: std::vector<StrideC> stride_C_host;
211: std::vector<StrideD> stride_D_host;
212: std::vector<LayoutSFA> layout_SFA_host;
213: std::vector<LayoutSFB> layout_SFB_host;
214: 
215: std::vector<ElementAccumulator> alpha_host;
216: std::vector<ElementAccumulator> beta_host;
217: 
218: uint64_t seed;
219: 
220: cutlass::DeviceAllocation<ElementA> block_A;
221: cutlass::DeviceAllocation<ElementB> block_B;
222: cutlass::DeviceAllocation<ElementC> block_C;
223: cutlass::DeviceAllocation<ElementD> block_D;
224: cutlass::DeviceAllocation<ElementBlockScale> blockscale_block_A;
225: cutlass::DeviceAllocation<ElementBlockScale> blockscale_block_B;
226: 
227: cutlass::DeviceAllocation<const ElementA *> ptr_A;
228: cutlass::DeviceAllocation<const ElementB *> ptr_B;
229: cutlass::DeviceAllocation<const ElementC *> ptr_C;
230: cutlass::DeviceAllocation<ElementD *> ptr_D;
231: cutlass::DeviceAllocation<ElementD *> ptr_ref_D;
232: cutlass::DeviceAllocation<const ElementBlockScale *> ptr_blockscale_A;
233: cutlass::DeviceAllocation<const ElementBlockScale *> ptr_blockscale_B;
234: 
235: cutlass::DeviceAllocation<StrideA> stride_A;
236: cutlass::DeviceAllocation<StrideB> stride_B;
237: cutlass::DeviceAllocation<StrideC> stride_C;
238: cutlass::DeviceAllocation<StrideD> stride_D;
239: cutlass::DeviceAllocation<LayoutSFA> layout_SFA;
240: cutlass::DeviceAllocation<LayoutSFB> layout_SFB;
241: 
242: cutlass::DeviceAllocation<ElementAccumulator*> alpha_device;
243: cutlass::DeviceAllocation<ElementAccumulator*> beta_device;
244: cutlass::DeviceAllocation<ElementAccumulator> block_alpha;
245: cutlass::DeviceAllocation<ElementAccumulator> block_beta;
246: 
```

- **EN**: After the kernel type is built, the file derives stride/layout helper types and allocates host/device containers that hold problem sizes, per-group strides, per-group tensor pointers, and alpha/beta storage. This mirrors how grouped GEMM packages metadata for many independent problems into contiguous arrays.
- **CN**: 在 kernel 类型组装完成后，文件继续推导 stride/layout 辅助类型，并声明保存 problem size、逐组 stride、逐组张量指针以及 alpha/beta 的主机/设备容器。这正是 grouped GEMM 将多个独立问题打包成连续元数据数组的方式。

### Lines 247-311 | Result record and random initialization helper / 结果结构与随机初始化辅助函数

```cpp
247: #endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
248: 
249: /////////////////////////////////////////////////////////////////////////////////////////////////
250: /// Testbed utility types
251: /////////////////////////////////////////////////////////////////////////////////////////////////
252: 
253: /// Result structure
254: struct Result
255: {
256:   double avg_runtime_ms;
257:   double gflops;
258:   cutlass::Status status;
259:   cudaError_t error;
260:   bool passed;
261: 
262:   Result(
263:     double avg_runtime_ms = 0,
264:     double gflops = 0,
265:     cutlass::Status status = cutlass::Status::kSuccess,
266:     cudaError_t error = cudaSuccess)
267:   :
268:     avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(false)
269:   {}
270: 
271: };
272: 
273: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
274: 
275: /////////////////////////////////////////////////////////////////////////////////////////////////
276: /// GEMM setup and evaluation
277: /////////////////////////////////////////////////////////////////////////////////////////////////
278: 
279: /// Helper to initialize a block of device data
280: template <class Element, class ScopeMin = std::nullopt_t, class ScopeMax = std::nullopt_t>
281: bool initialize_block(
282:   cutlass::DeviceAllocation<Element>& block,
283:   uint64_t seed=2023,
284:   ScopeMin scope_min = std::nullopt, ScopeMax scope_max = std::nullopt) {
285: 
286:   double _scope_max, _scope_min;
287:   int bits_input = cutlass::sizeof_bits<Element>::value;
288:   if (bits_input == 1) {
289:     _scope_max = 2;
290:     _scope_min = 0;
291:   } else if (bits_input <= 8) {
292:     _scope_max = 2;
293:     _scope_min = -2;
294:   } else if (bits_input == 16) {
295:     _scope_max = 5;
296:     _scope_min = -5;
297:   } else {
298:     _scope_max = 8;
299:     _scope_min = -8;
300:   }
301:   if constexpr (!std::is_same_v<ScopeMax, std::nullopt_t>) {
302:     _scope_max = scope_max;
303:   }
304:   if constexpr (!std::is_same_v<ScopeMin, std::nullopt_t>) {
305:     _scope_min = scope_min;
306:   }
307:   cutlass::reference::device::BlockFillRandomUniform(
308:     block.get(), block.size(), seed, (Element) _scope_max, (Element) _scope_min, 0);
309: 
310:   return true;
311: }
```

- **EN**: The `Result` structure stores status, runtime, and throughput. The initialization helper chooses random ranges based on element width so the sample can fill FP8 tensors and scaling tensors with values that are large enough to exercise the kernel while remaining numerically reasonable for verification.
- **CN**: `Result` 结构体保存执行状态、运行时间和吞吐率。初始化辅助函数依据元素位宽选择随机范围，从而为 FP8 张量和缩放张量生成既能覆盖 kernel 路径、又便于数值验证的测试数据。

### Lines 312-383 | Grouped allocation and packed layout preparation / 分组分配与打包布局准备

```cpp
312: 
313: /// Allocates device-side data
314: template <typename OptionType>
315: void allocate(const OptionType &options) {
316: 
317:   int64_t total_elements_A = 0;
318:   int64_t total_elements_B = 0;
319:   int64_t total_elements_C = 0;
320:   int64_t total_elements_D = 0;
321:   int64_t total_elements_blockscale_A = 0;
322:   int64_t total_elements_blockscale_B = 0;
323: 
324:   offset_A.clear();
325:   offset_B.clear();
326:   offset_C.clear();
327:   offset_D.clear();
328:   offset_blockscale_A.clear();
329:   offset_blockscale_B.clear();
330:   stride_A_host.clear();
331:   stride_B_host.clear();
332:   stride_C_host.clear();
333:   stride_D_host.clear();
334: 
335:   for (int32_t i = 0; i < options.groups; ++i) {
336: 
337:     auto problem = options.problem_sizes_host.at(i);
338:     auto M = get<0>(problem);
339:     auto N = get<1>(problem);
340:     auto K = get<2>(problem);
341: 
342:     auto group_layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(M, N, K, 1));
343:     auto group_layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(M, N, K, 1));
344: 
345:     offset_A.push_back(total_elements_A);
346:     offset_B.push_back(total_elements_B);
347:     offset_C.push_back(total_elements_C);
348:     offset_D.push_back(total_elements_D);
349:     offset_blockscale_A.push_back(total_elements_blockscale_A);
350:     offset_blockscale_B.push_back(total_elements_blockscale_B);
351: 
352:     int64_t elements_A = M * K;
353:     int64_t elements_B = K * N;
354:     int64_t elements_C = M * N;
355:     int64_t elements_D = M * N;
356:     int64_t elements_blockscale_A = size(filter_zeros(group_layout_SFA));
357:     int64_t elements_blockscale_B = size(filter_zeros(group_layout_SFB));
358: 
359:     total_elements_A += elements_A;
360:     total_elements_B += elements_B;
361:     total_elements_C += elements_C;
362:     total_elements_D += elements_D;
363:     total_elements_blockscale_A += elements_blockscale_A;
364:     total_elements_blockscale_B += elements_blockscale_B;
365: 
366:     stride_A_host.push_back(cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1}));
367:     stride_B_host.push_back(cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1}));
368:     stride_C_host.push_back(cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1}));
369:     stride_D_host.push_back(cutlass::make_cute_packed_stride(StrideD{}, {M, N, 1}));
370:     layout_SFA_host.push_back(group_layout_SFA);
371:     layout_SFB_host.push_back(group_layout_SFB);
372: 
373:   }
374: 
375:   block_A.reset(total_elements_A);
376:   block_B.reset(total_elements_B);
377:   block_C.reset(total_elements_C);
378:   block_D.reset(total_elements_D);
379:   block_alpha.reset(options.groups);
380:   block_beta.reset(options.groups);
381:   blockscale_block_A.reset(total_elements_blockscale_A);
382:   blockscale_block_B.reset(total_elements_blockscale_B);
383: }
```

- **EN**: This section walks every group, computes packed offsets and strides, derives the block-scale layouts for A and B, and allocates the backing buffers. The important scheduler-adjacent detail is that all groups are flattened into pointer arrays plus metadata arrays, which is the representation the grouped kernel scheduler expects.
- **CN**: 这一段遍历所有 group，计算打包后的偏移与 stride，推导 A/B 的 block-scale 布局，并分配底层缓冲区。与调度密切相关的一点是：所有 group 最终都被压平成“指针数组 + 元数据数组”，这正是 grouped kernel scheduler 所要求的输入表示。

### Lines 384-492 | Per-group initialization and host-to-device metadata upload / 逐组初始化与元数据上传

```cpp
384: 
385: /// Initialize operands to be used in the GEMM and reference GEMM
386: template <typename OptionType>
387: void initialize(const OptionType &options) {
388: 
389:   problem_sizes.reset(options.groups);
390:   problem_sizes.copy_from_host(options.problem_sizes_host.data());
391: 
392:   std::vector<ElementA *> ptr_A_host(options.groups);
393:   std::vector<ElementB *> ptr_B_host(options.groups);
394:   std::vector<ElementC *> ptr_C_host(options.groups);
395:   std::vector<ElementD *> ptr_D_host(options.groups);
396:   std::vector<ElementAccumulator *> ptr_alpha_host(options.groups);
397:   std::vector<ElementAccumulator *> ptr_beta_host(options.groups);
398:   std::vector<ElementBlockScale *> ptr_blockscale_A_host(options.groups);
399:   std::vector<ElementBlockScale *> ptr_blockscale_B_host(options.groups);
400: 
401:   alpha_host.clear();
402:   beta_host.clear();
403: 
404:   for (int i = 0; i < options.groups; i++) {
405:     // If the current group's matrix has size 0, set the pointer to nullptr
406:     if (i < options.groups - 1 && offset_A.at(i) == offset_A.at(i + 1)) {
407:       ptr_A_host.at(i) = nullptr;
408:     } else {
409:       ptr_A_host.at(i) = block_A.get() + offset_A.at(i);
410:     }
411:     if (i < options.groups - 1 && offset_B.at(i) == offset_B.at(i + 1)) {
412:       ptr_B_host.at(i) = nullptr;
413:     } else {
414:       ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
415:     }
416:     if (i < options.groups - 1 && offset_C.at(i) == offset_C.at(i + 1)) {
417:       ptr_C_host.at(i) = nullptr;
418:     } else {
419:       ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
420:     }
421:     if (i < options.groups - 1 && offset_D.at(i) == offset_D.at(i + 1)) {
422:       ptr_D_host.at(i) = nullptr;
423:     } else {
424:       ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
425:     }
426:     if (i < options.groups - 1 && offset_blockscale_A.at(i) == offset_blockscale_A.at(i + 1)) {
427:       ptr_blockscale_A_host.at(i) = nullptr;
428:     } else {
429:       ptr_blockscale_A_host.at(i) = blockscale_block_A.get() + offset_blockscale_A.at(i);
430:     }
431:     if (i < options.groups - 1 && offset_blockscale_B.at(i) == offset_blockscale_B.at(i + 1)) {
432:       ptr_blockscale_B_host.at(i) = nullptr;
433:     } else {
434:       ptr_blockscale_B_host.at(i) = blockscale_block_B.get() + offset_blockscale_B.at(i);
435:     }
436:     alpha_host.push_back((options.alpha == FLT_MAX) ? static_cast<ElementAccumulator>((rand() % 5) + 1) : options.alpha);
437:     beta_host.push_back((options.beta == FLT_MAX) ? static_cast<ElementAccumulator>(rand() % 5) : options.beta);
438:     ptr_alpha_host.at(i) = block_alpha.get() + i;
439:     ptr_beta_host.at(i) = block_beta.get() + i;
440:   }
441: 
442:   ptr_A.reset(options.groups);
443:   ptr_A.copy_from_host(ptr_A_host.data());
444: 
445:   ptr_B.reset(options.groups);
446:   ptr_B.copy_from_host(ptr_B_host.data());
447: 
448:   ptr_C.reset(options.groups);
449:   ptr_C.copy_from_host(ptr_C_host.data());
450: 
451:   ptr_D.reset(options.groups);
452:   ptr_D.copy_from_host(ptr_D_host.data());
453: 
454:   ptr_blockscale_A.reset(options.groups);
455:   ptr_blockscale_A.copy_from_host(ptr_blockscale_A_host.data());
456: 
457:   ptr_blockscale_B.reset(options.groups);
458:   ptr_blockscale_B.copy_from_host(ptr_blockscale_B_host.data());
459: 
460:   stride_A.reset(options.groups);
461:   stride_A.copy_from_host(stride_A_host.data());
462: 
463:   stride_B.reset(options.groups);
464:   stride_B.copy_from_host(stride_B_host.data());
465: 
466:   stride_C.reset(options.groups);
467:   stride_C.copy_from_host(stride_C_host.data());
468: 
469:   stride_D.reset(options.groups);
470:   stride_D.copy_from_host(stride_D_host.data());
471: 
472:   layout_SFA.reset(options.groups);
473:   layout_SFA.copy_from_host(layout_SFA_host.data());
474: 
475:   layout_SFB.reset(options.groups);
476:   layout_SFB.copy_from_host(layout_SFB_host.data());
477: 
478:   alpha_device.reset(options.groups);
479:   alpha_device.copy_from_host(ptr_alpha_host.data());
480:   beta_device.reset(options.groups);
481:   beta_device.copy_from_host(ptr_beta_host.data());
482: 
483:   initialize_block(block_A, seed + 2022);
484:   initialize_block(block_B, seed + 2023);
485:   initialize_block(block_C, seed + 2024);
486:   initialize_block(blockscale_block_A, seed + 2025, -1, 1);
487:   initialize_block(blockscale_block_B, seed + 2026, -1, 1);
488: 
489:   block_alpha.copy_from_host(alpha_host.data());
490:   block_beta.copy_from_host(beta_host.data());
491: 
492: }
```

- **EN**: The file then materializes each group’s pointers, handles degenerate zero-sized problems with `nullptr`, fills alpha/beta either as scalars or per-group arrays, and copies the full metadata package to device memory. By the end of this block, the runtime has everything needed to launch grouped GEMM with heterogeneous shapes.
- **CN**: 随后代码为每个 group 生成实际指针，对零尺寸问题用 `nullptr` 处理，按“统一标量”或“逐组数组”方式填充 alpha/beta，并把完整的元数据包上传到设备端。执行完这一段后，运行时已经具备了发射异构 grouped GEMM 所需的一切信息。

### Lines 493-548 | Argument construction and scheduler controls / 参数构造与调度器控制

```cpp
493: 
494: /// Populates a Gemm::Arguments structure from the given commandline options
495: template<typename GemmArguments, typename OptionType>
496: GemmArguments args_from_options(const OptionType &options, bool host_problem_shapes_available = true)
497: {
498:   // Change device_id to another value if you are running on a machine with multiple GPUs and wish
499:   // to use a GPU other than that with device ID 0.
500:   int device_id = 0;
501:   cutlass::KernelHardwareInfo kernel_hw_info = cutlass::KernelHardwareInfo::make_kernel_hardware_info<typename Gemm::GemmKernel>(device_id);
502: 
503:   GemmArguments arguments{
504:     cutlass::gemm::GemmUniversalMode::kGrouped,
505:     {options.groups, problem_sizes.get(), host_problem_shapes_available ? options.problem_sizes_host.data() : (decltype(options.problem_sizes_host.data())) nullptr},
506:     {ptr_A.get(), stride_A.get(), ptr_B.get(), stride_B.get(),
507:      ptr_blockscale_A.get(), layout_SFA.get(),
508:      ptr_blockscale_B.get(), layout_SFB.get()
509:     },
510:     {
511:       {}, // epilogue.thread
512:       ptr_C.get(), stride_C.get(),
513:       ptr_D.get(), stride_D.get()
514:     },
515:     kernel_hw_info
516:   };
517: 
518:   auto &fusion_args = arguments.epilogue.thread;
519:   if (options.alpha != FLT_MAX && options.beta != FLT_MAX) {
520:     // If both alpha/beta are provided (via cmd line args) and are scalar, i.e., same alpha/beta applies to all batches.
521:     fusion_args.alpha = options.alpha;
522:     fusion_args.beta = options.beta;
523:     fusion_args.alpha_ptr = nullptr;
524:     fusion_args.beta_ptr = nullptr;
525:     fusion_args.alpha_ptr_array = nullptr;
526:     fusion_args.beta_ptr_array = nullptr;
527:     // Single alpha and beta for all groups
528:     fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 0};
529:     fusion_args.dBeta = {cute::_0{}, cute::_0{}, 0};
530:   }
531:   else {
532:     // If pointers to alpha/beta are provided, i.e., alpha/beta can differ between batches/groups.
533:     fusion_args.alpha = 0;
534:     fusion_args.beta = 0;
535:     fusion_args.alpha_ptr = nullptr;
536:     fusion_args.beta_ptr = nullptr;
537:     fusion_args.alpha_ptr_array = alpha_device.get();
538:     fusion_args.beta_ptr_array = beta_device.get();
539:     // One alpha and beta per each group
540:     fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 1};
541:     fusion_args.dBeta = {cute::_0{}, cute::_0{}, 1};
542:   }
543: 
544:   arguments.scheduler.raster_order = options.raster_order;
545:   // The tile scheduler will swizzle up to 8 and with the nearest multiple of 2 (i.e., 1, 2, 4, and 8)
546:   arguments.scheduler.max_swizzle_size = options.swizzle;
547: 
548:   return arguments;
```

- **EN**: This function builds `Gemm::Arguments` in grouped mode. It switches between scalar alpha/beta and per-group alpha/beta arrays, then explicitly programs `arguments.scheduler.raster_order` and `arguments.scheduler.max_swizzle_size`. Those two fields are the host-visible knobs that steer how the grouped tile scheduler traverses output space and how aggressively it swizzles tile order.
- **CN**: 该函数以 grouped 模式构造 `Gemm::Arguments`。它在“统一 alpha/beta 标量”和“逐组 alpha/beta 数组”之间切换，并显式设置 `arguments.scheduler.raster_order` 与 `arguments.scheduler.max_swizzle_size`。这两个字段就是主机侧可见的调度旋钮，用来控制 grouped tile scheduler 如何遍历输出空间，以及以多大力度对 tile 顺序做 swizzle。

### Lines 549-654 | Reference verification path / 参考验证路径

```cpp
549: }
550: 
551: template <typename OptionType>
552: bool verify(const OptionType &options) {
553: 
554:   //
555:   // Compute reference output
556:   //
557: 
558:   std::vector<ElementA> block_A_host(block_A.size());
559:   std::vector<ElementB> block_B_host(block_B.size());
560:   std::vector<ElementC> block_C_host(block_C.size());
561:   std::vector<ElementD> block_D_host_kernel(block_D.size());
562:   std::vector<ElementD> block_D_host_ref(block_D.size());
563:   std::vector<ElementBlockScale> blockscale_block_A_host(blockscale_block_A.size());
564:   std::vector<ElementBlockScale> blockscale_block_B_host(blockscale_block_B.size());
565: 
566:   block_A.copy_to_host(block_A_host.data());
567:   block_B.copy_to_host(block_B_host.data());
568:   block_C.copy_to_host(block_C_host.data());
569:   block_D.copy_to_host(block_D_host_kernel.data());
570:   blockscale_block_A.copy_to_host(blockscale_block_A_host.data());
571:   blockscale_block_B.copy_to_host(blockscale_block_B_host.data());
572: 
573:   bool passed = true;
574:   std::cout << "  Running host reference kernel - may run for a while for large problems." << std::endl;
575:   for (int group_idx = 0; group_idx < options.groups; group_idx++) {
576:     // Group scaling tensors shapes based `ScaleGranularityM`, CTA Block (TileShape) and GEMM Problem shape
577:     auto [m, n, k] = options.problem_sizes_host.at(group_idx);
578: 
579:     // Create instantiation for device reference gemm kernel
580:     auto A = cute::make_tensor(block_A_host.data() + offset_A.at(group_idx),
581:                               cute::make_layout(
582:                                   cute::make_shape(m, k, 1),
583:                                   stride_A_host.at(group_idx)
584:                                 )
585:                               );
586:     auto B = cute::make_tensor(block_B_host.data() + offset_B.at(group_idx),
587:                               cute::make_layout(
588:                                 cute::make_shape(n, k, 1),
589:                                 stride_B_host.at(group_idx)
590:                                 )
591:                               );
592:     auto C = cute::make_tensor(block_C_host.data() + offset_C.at(group_idx),
593:                               cute::make_layout(
594:                                   cute::make_shape(m, n, 1),
595:                                   stride_C_host.at(group_idx)
596:                                 )
597:                               );
598:     auto D = cute::make_tensor(block_D_host_ref.data() + offset_D.at(group_idx),
599:                               cute::make_layout(
600:                                   cute::make_shape(m, n, 1),
601:                                   stride_D_host.at(group_idx)
602:                                 )
603:                               );
604: 
605:     auto SFA = cute::make_tensor(blockscale_block_A_host.data() + offset_blockscale_A.at(group_idx),
606:                                  layout_SFA_host.at(group_idx));
607:     auto SFB = cute::make_tensor(blockscale_block_B_host.data() + offset_blockscale_B.at(group_idx),
608:                                  layout_SFB_host.at(group_idx));
609: 
610:     using unused_t = decltype(D);
611: 
612:     cutlass::reference::host::GettBlockScalingMainloopParams<
613:       ElementAccumulator,
614:       decltype(A),
615:       decltype(SFA),
616:       decltype(B),
617:       decltype(SFB)
618:     > mainloop_params{A, SFA, B, SFB};
619: 
620:     cutlass::reference::host::GettEpilogueParams<
621:         ElementScalar,
622:         ElementScalar,
623:         ElementAccumulator,
624:         ElementCompute,
625:         decltype(C),
626:         decltype(D)
627:     > epilogue_params;
628: 
629:     epilogue_params.C = C;
630:     epilogue_params.D = D;
631:     epilogue_params.alpha = alpha_host.at(group_idx);
632:     epilogue_params.beta = beta_host.at(group_idx);
633: 
634:     // get reference result
635:     cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
636: 
637:     // Check if output from CUTLASS kernel and reference kernel are equal or not
638:     auto this_group_passed = std::equal(
639:       // std::execution::par_unseq,
640:       block_D_host_ref.data() + offset_D.at(group_idx),
641:       block_D_host_ref.data() + offset_D.at(group_idx) + m * n,
642:       block_D_host_kernel.data() + offset_D.at(group_idx)
643:     );
644: 
645:     passed &= this_group_passed;
646: 
647: #if 0
648:     std::cout << "Group: " << group_idx << " M: " << m << " N: " << n << " K: " << k << " Status: " << this_group_passed << std::endl;
649: #endif
650: 
651:   }
652: 
653:   return passed;
654: }
```

- **EN**: For correctness, the example reconstructs each problem on the host, prepares block-scaling reference parameters, runs CUTLASS reference GEMM helpers, and compares the reference D tensor with the device result. This is what makes the sample trustworthy when experimenting with scheduler parameters or different groups.
- **CN**: 为保证正确性，示例在主机端重建每个问题，准备 block-scaling 参考参数，调用 CUTLASS 参考 GEMM 工具，然后将参考 D 张量与设备结果比较。正因为有这条路径，用户才能放心尝试不同的调度参数或分组配置。

### Lines 655-733 | Run flow, workspace, and profiling / 运行流程、workspace 与性能测试

```cpp
655: 
656: /// Execute a given example GEMM computation
657: template <typename OptionType>
658: int run(OptionType &options, bool host_problem_shapes_available = true)
659: {
660:   allocate(options);
661:   initialize(options);
662: 
663:   std::cout << "  Problem Sizes, Alpha, Beta " << std::endl;
664:   for (int32_t i = 0; i < options.groups; ++i) {
665:     std::cout << "    " << options.problem_sizes_host.at(i);
666:     std::cout << ", " << alpha_host.at(i) << ", " << beta_host.at(i) << std::endl;
667:   }
668:   std::cout << "  Groups      : " << options.groups  << std::endl;
669:   std::cout << "  Tile shape (M, N, K): " << size<0>(TileShape{}) << ", " << size<1>(TileShape{}) << ", " << size<2>(TileShape{}) << std::endl;
670:   std::cout << "  ScaleGranularityM: " << ScaleGranularityM << " (ScaleMsPerTile: " << ScaleMsPerTile << ")" << std::endl;
671:   std::cout << "  ScaleGranularityN: " << ScaleGranularityN << " (ScaleNsPerTile: " << ScaleNsPerTile << ")" << std::endl;
672:   std::string raster = "Heuristic";
673:   if (options.raster_order == RasterOrderOptions::AlongN) {
674:     raster = "Along N";
675:   }
676:   else if (options.raster_order == RasterOrderOptions::AlongM) {
677:     raster = "Along M";
678:   }
679:   std::cout << "  Rasterization: " << raster << " with a maximum CTA swizzle of " << options.swizzle << std::endl;
680: 
681:   // Instantiate CUTLASS kernel depending on templates
682:   Gemm gemm;
683: 
684:   // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
685:   auto arguments = args_from_options<typename Gemm::Arguments>(options, host_problem_shapes_available);
686: 
687:   // Using the arguments, query for extra workspace required for matrix multiplication computation
688:   size_t workspace_size = Gemm::get_workspace_size(arguments);
689: 
690:   // Allocate workspace memory
691:   cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
692: 
693:   // Check if the problem size is supported or not
694:   CUTLASS_CHECK(gemm.can_implement(arguments));
695: 
696:   // Initialize CUTLASS kernel with arguments and workspace pointer
697:   CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
698: 
699:   // Correctness / Warmup iteration
700:   CUTLASS_CHECK(gemm.run());
701: 
702:   // Check if output from CUTLASS kernel and reference kernel are equal or not
703:   Result result;
704:   result.passed = verify(options);
705: 
706:   std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
707: 
708:   if (!result.passed) {
709:    exit(-1);
710:   }
711: 
712:   // Run profiling loop
713:   if (options.iterations > 0) {
714:     GpuTimer timer;
715:     timer.start();
716:     for (int iter = 0; iter < options.iterations; ++iter) {
717:       CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
718:       CUTLASS_CHECK(gemm.run());
719:     }
720:     timer.stop();
721: 
722:     // Compute average runtime and GFLOPs.
723:     float elapsed_ms = timer.elapsed_millis();
724:     result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
725:     result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);
726: 
727:     std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
728:     std::cout << "  GFLOPS: " << result.gflops << std::endl;
729:     fflush(stdout);
730:   }
731: 
732:   return 0;
733: }
```

- **EN**: The `run()` path performs allocation, initialization, capability checks, workspace sizing, kernel initialization, kernel launch, validation, and timed profiling iterations. The grouped example uses the canonical CUTLASS flow: `get_workspace_size()`, `can_implement()`, `initialize()`, then `run()`.
- **CN**: `run()` 路径依次完成分配、初始化、能力检查、workspace 大小查询、kernel 初始化、kernel 启动、结果校验以及计时 profiling。这个 grouped 示例遵循 CUTLASS 的标准流程：`get_workspace_size()`、`can_implement()`、`initialize()`，最后再 `run()`。

### Lines 734-790 | Program entry and two launch modes / 程序入口与两种启动模式

```cpp
734: 
735: #endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
736: 
737: ///////////////////////////////////////////////////////////////////////////////////////////////////
738: 
739: int main(int argc, char const **args) {
740: 
741:   // CUTLASS must be compiled with CUDA 12.3 Toolkit to run this example
742:   // and must have compute capability at least 90.
743:   if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 3)) {
744:     std::cerr << "This example requires CUDA 12.3 or newer.\n";
745:     // Returning zero so this test passes on older Toolkits. Its actions are no-op.
746:     return 0;
747:   }
748: 
749:   cudaDeviceProp props;
750:   int current_device_id;
751:   CUDA_CHECK(cudaGetDevice(&current_device_id));
752:   CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
753:   cudaError_t error = cudaGetDeviceProperties(&props, 0);
754:   if (props.major != 9) {
755:     std::cerr
756:       << "This example requires a GPU of NVIDIA's Hopper Architecture or "
757:       << "later (compute capability 90 or greater).\n";
758:     return 0;
759:   }
760: 
761: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
762: 
763:   //
764:   // Parse options
765:   //
766: 
767:   Options<ProblemShape> options;
768: 
769:   options.parse(argc, args);
770: 
771:   if (options.help) {
772:     options.print_usage(std::cout) << std::endl;
773:     return 0;
774:   }
775: 
776:   //
777:   // Evaluate CUTLASS kernels
778:   //
779: 
780:   std::cout << "Running tests with host problem shapes:" << std::endl;
781:   run(options, true);
782:   std::cout << "Running tests without host problem shapes:" << std::endl;
783:   run(options, false);
784: 
785: #endif
786: 
787:   return 0;
788: }
789: 
790: /////////////////////////////////////////////////////////////////////////////////////////////////
```

- **EN**: The `main()` function enforces Hopper/CUDA requirements, parses command-line options, prints help when requested, and executes the sample twice: once with host problem-shape pointers available and once without them. That dual run demonstrates that the grouped scheduler can operate from device-resident metadata alone.
- **CN**: `main()` 先检查 Hopper/CUDA 前提，解析命令行参数，并在需要时打印帮助。随后示例会执行两次：一次提供 host problem-shape 指针，一次不提供。这样的双重运行展示了 grouped scheduler 可以仅依赖设备端元数据工作。

---

## Key Concepts / 关键概念

- **EN**: Grouped GEMM scheduler knobs: `raster_order` selects the major traversal direction, while `max_swizzle_size` controls tile-order swizzling for load balance and locality.
  **CN**: Grouped GEMM 调度旋钮：`raster_order` 选择主遍历方向，`max_swizzle_size` 控制 tile 顺序 swizzle，以平衡负载并改善局部性。
- **EN**: Blockwise FP8 scaling: the mainloop consumes matrix pointers together with scale-layout pointers so scaling happens as part of the math pipeline.
  **CN**: FP8 块级缩放：mainloop 同时接收矩阵指针和缩放布局指针，使缩放逻辑成为计算流水线的一部分。
- **EN**: Grouped metadata packing: problem sizes, strides, and tensor pointers are stored in arrays so one launch can cover many differently shaped GEMMs.
  **CN**: 分组元数据打包：problem size、stride 与张量指针都存入数组，从而让一次 launch 覆盖多个不同形状的 GEMM。
- **EN**: Reference validation is essential because scheduler settings change execution order but must not change numerically accepted results.
  **CN**: 参考验证非常关键，因为调度设置会改变执行顺序，但不能改变数值上可接受的结果。

## Dependencies / 依赖项

- **`cutlass/gemm/*`, `cutlass/epilogue/*`**  
  **EN**: Kernel, collective builder, grouped-GEMM argument, and epilogue fusion types.  
  **CN**: 提供 kernel、collective builder、grouped GEMM 参数以及 epilogue 融合类型。
- **`cute/*`**  
  **EN**: Shape/layout algebra used to define tiles, clusters, and scale layouts.  
  **CN**: 提供定义 tile、cluster 和缩放布局所需的 shape/layout 代数。
- **`helper.h` and `hopper_fp8_commandline.hpp`**  
  **EN**: Example-level helpers for error checking, timing, and FP8-specific command-line options.  
  **CN**: 提供示例层面的错误检查、计时和 FP8 命令行选项。
- **CUTLASS host/device reference utilities**  
  **EN**: Used to initialize tensors and verify grouped results.  
  **CN**: 用于初始化张量并验证 grouped 结果。
