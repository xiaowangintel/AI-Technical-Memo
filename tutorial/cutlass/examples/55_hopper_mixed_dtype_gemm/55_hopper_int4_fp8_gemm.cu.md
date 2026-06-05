# 55_hopper_int4_fp8_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/55_hopper_mixed_dtype_gemm/55_hopper_int4_fp8_gemm.cu`  
**Purpose / 用途**: This example demonstrates a Hopper mixed-dtype GEMM where narrow quantized weights and packed scaling metadata are fused into the math pipeline, while optional offline reordering improves the memory access pattern of the quantized operand. / 该示例展示了 Hopper 上的混合精度 GEMM：将窄量化权重与打包后的缩放元数据直接融合进计算流水线，同时可选的离线重排可改善量化操作数的访存模式。

---

## Line-by-Line Analysis / 逐行分析

The sections below cover the full source file in order. Each block includes original code and bilingual analysis.  
下面的各个小节按顺序覆盖整个源文件。每个代码块都附带原始代码与中英双语分析。

### Lines 1-88 | Motivation, constraints, and usage notes / 动机、约束与使用说明

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
33:     \brief Hopper GEMM example with different data types using CUTLASS 3.0 APIs for NVIDIA Hopper architecture
34: 
35:     This example shows how to perform INT4 x FP8 GEMM and scale up the INT4 weight during dequantization. It uses a look-up table to avoid the multiplications
36:     between INT4 and FP8. To trigger this method, use cutlass::Array<ElementScale, 8> as the scale type in the collective's arguments.
37: 
38:     However, this algorithm requires changes to the encoding of INT4 weights and scale factors. These changes must happen before launching the GEMM. See the helper functions
39:     `unify_quant_encoding`, `initialize_packed_scale` in the header `fp8_packed_scale.hpp` for details.
40: 
41:     In a nutshell, the positive values of INT4 weights need to be encoded in the same way as negative values except for the sign bit. For each scale factor,
42:     8 negative results (-8 x scale, -7 x scale, ... -1 x scale) are packed together, forming a cutlass::Array<ElementScale, 8> value.
43: 
44:     The narrower type always passes through the register file. Therefore, in cases where the narrower type is operand B, the collective will implicitly swap
45:     A and B in the main loop. However, as a result of this collective performing implicit swaps, it does not support TMA epilogues. Consequently, it is essential to consider this when constructing the epilogue,
46:     as illustrated in this example.
47: 
48:     Note that in this example, we explicitly swap A and B in order to use TMA epilogues. We do this since TMA epilogues are more performant on problem sizes of interest.
49: 
50:     As an additional optimization, we can reorder the narrow data type tensor such that elements read into register file by the same thread are contiguous in global and shared memory.
51:     This promotes vectorization of shared memory loads and removes additional instructions on the critical path. For example, when MMA is performed in FP8 data type, each thread reads
52:     4 groups of 4 elements that are logically contiguous in the same row (refer to https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#wgmma-64n32-a for thread-value layout).
53:     If the narrow type is INT4 and tensor is major in K dim, only 16 bits can be read at a time, leading to extra load instructions and suboptimal utilization of shared memory throughput.
54:     If we reorder the data offline to place all 16 elements read by a thread contiguously in memory, a single 64-bit load is sufficient. This reordering is often feasible when the quantized
55:     tensor is static (e.g. weight tensor of a NN layer at inference time). This example demonstrates how such a reordering can be performed and communicated to the kernel when the options.shuffle is set to true.
56: 
57:     It is expected that the scale's K dimension be scale_k = ceil_div(problem_k, group_size).
58: 
59:     Scales are always expected to be MN major. This means the fastest changing dimension must be M if A is scaled or N if B is scaled.
60: 
61:     If A is being scaled, the scales must have shape [M, scale_k],  while if B is scaled, it must have shape [N, scale_k].
62: 
63:     The implementation only supports "group-wise" scales. However, we can make it work for per-column scales by setting the group's size
64:     equal to the gemm problem K.
65: 
66:     Limitations:
67:       1) Only supports INT4 x { FP8, INT8, UINT8 }. The scales must be the same as mma Type. Scale with zero-point mode is not supported.
68:       2) The INT4 weights and scale factors have additional encoding requirements.
69:       3) The scales must be MN major. That means if A is scaled, it must be column major, but if B is scaled it must be row major.
70:       4) The scales must have the same layout and groupsize.
71:       5) The groupsize must be greater or equal to the tile shape k.
72:       6) Currently, TMA epilogues cannot be used when the narrow type is the B operand. This limitation arises because the implementation always swaps the
73:          operands to ensure that the narrow type passes through the register file, and TMA epilogues do not currently support implicit swap + transpose operations.
74:          We plan to address this limitation in the future. However, we address this in the example by explicitly swapping and transposing the operands.
75: 
76:     Optimizing suggestions:
77:       1) Use a small tile size, since the register pressure for this GEMM (and RS GEMM in general) is high (it uses a lot of register space).
78: 
79:     Examples:
80: 
81:       Runs the mixed input batched gemm (with batch size 2), converting B to the type of A (mode 0)
82:       $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_int4_fp8_gemm --m=2048 --n=2048 --k=2048 --l=2 --mode=0
83: 
84:       Runs the mixed input gemm, and applies a scaling factor to B before mma (mode 1). Applies a vector of scales to the entire
85:       matrix (group size is the same as the gemm k dimension).
86:       $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_int4_fp8_gemm --m=4096 --n=5120 --k=8192 --g=8192 --mode=1
87: */
88: 
```

- **EN**: The long opening comment is important to read carefully. It explains why INT4/FP8 mixed-dtype GEMM needs packed scales, why the narrow operand often travels through registers, why the sample manually swaps/transposes operands to keep TMA epilogues available, and when optional offline shuffle becomes beneficial.
- **CN**: 开头的大段注释非常关键。它解释了 INT4/FP8 混合 GEMM 为什么需要打包 scale、为什么窄操作数通常要经过寄存器通路、为什么示例要手动交换/转置操作数以保留 TMA epilogue，以及可选的离线 shuffle 何时有价值。

### Lines 89-194 | Headers, data types, and kernel configuration / 头文件、数据类型与 kernel 配置

```cpp
 89: #include <iostream>
 90: 
 91: #include "cutlass/cutlass.h"
 92: 
 93: #include "cute/tensor.hpp"
 94: #include "cutlass/tensor_ref.h"
 95: #include "cutlass/epilogue/collective/default_epilogue.hpp"
 96: #include "cutlass/epilogue/thread/linear_combination.h"
 97: #include "cutlass/gemm/dispatch_policy.hpp"
 98: #include "cutlass/gemm/collective/collective_builder.hpp"
 99: #include "cutlass/epilogue/collective/collective_builder.hpp"
100: #include "cutlass/gemm/device/gemm_universal_adapter.h"
101: #include "cutlass/gemm/kernel/gemm_universal.hpp"
102: 
103: #include "cutlass/util/command_line.h"
104: #include "cutlass/util/distribution.h"
105: #include "cutlass/util/host_tensor.h"
106: #include "cutlass/util/packed_stride.hpp"
107: #include "cutlass/util/tensor_view_io.h"
108: #include "cutlass/util/reference/device/tensor_fill.h"
109: #include "cutlass/util/reference/device/tensor_compare.h"
110: #include "cutlass/util/mixed_dtype_utils.hpp"
111: 
112: #include "helper.h"
113: #include "mixed_dtype_utils.hpp"
114: 
115: using namespace cute;
116: 
117: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
118: 
119: /////////////////////////////////////////////////////////////////////////////////////////////////
120: /// GEMM kernel configurations
121: /////////////////////////////////////////////////////////////////////////////////////////////////
122: using MmaType = cutlass::float_e4m3_t;
123: 
124: // Select quantization type via compile flag for this example
125: // templatized throughout code to enable instantiating both int4 and e2m1 versions in the same program
126: #if defined(CUTLASS_MIXED_DTYPE_E2M1)
127:   using QuantType = cutlass::float_e2m1_t;  // E2M1 (FP4)
128: #else
129:   using QuantType = cutlass::int4b_t;       // INT4 Two's Complement (default)
130: #endif
131: 
132: constexpr int TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;
133: 
134: // A matrix configuration
135: using         ElementA    = MmaType;                                        // Element type for A matrix operand
136: using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
137: constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)
138: 
139: // B matrix configuration
140: using         ElementB    = QuantType;                                      // Element type for B matrix operand
141: using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
142: constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)
143: 
144: // This example manually swaps and transposes, so keep transpose of input layouts
145: using LayoutA_Transpose = typename cutlass::layout::LayoutTranspose<LayoutA>::type;
146: using LayoutB_Transpose = typename cutlass::layout::LayoutTranspose<LayoutB>::type;
147: 
148: using StrideA = cutlass::detail::TagToStrideA_t<LayoutA>;
149: using StrideB = cutlass::detail::TagToStrideB_t<LayoutB>;
150: 
151: // Define the CuTe layout for reoredered quantized tensor B
152: // LayoutAtomQuant places values that will be read by the same thread in contiguous locations in global memory.
153: // It specifies the reordering within a single warp's fragment
154: using LayoutAtomQuant = decltype(cutlass::compute_memory_reordering_atom<MmaType>());
155: using LayoutB_Reordered = decltype(cute::tile_to_shape(LayoutAtomQuant{}, Layout<Shape<int,int,int>, StrideB>{}));
156: 
157: using ElementScale = MmaType;
158: using ElementZero = ElementScale; // only for verify
159: using LayoutScale = cutlass::layout::RowMajor;
160: 
161: // C/D matrix configuration
162: using         ElementC    = cutlass::half_t;                                // Element type for C and D matrix operands
163: using         LayoutC     = cutlass::layout::RowMajor;                      // Layout type for C and D matrix operands
164: constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)
165: 
166: // D matrix configuration
167: using         ElementD    = ElementC;
168: using         LayoutD     = LayoutC;
169: constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;
170: 
171: // Core kernel configurations
172: using ElementAccumulator  = float;                                          // Element type for internal accumulation
173: using ElementCompute      = float;                                          // Element type for epilogue computation
174: using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
175: using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
176: using TileShape           = Shape<_128,_128,cute::Int<TileShapeK>>;         // Threadblock-level tile size
177: using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
178: using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedCooperative;  // Kernel to launch based on the default setting in the Collective Builder
179: using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecializedCooperative;
180: using EpilogueTileType    = cutlass::epilogue::collective::EpilogueTileAuto;
181: 
182: using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
183:     cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
184:     TileShape, ClusterShape,
185:     EpilogueTileType,
186:     ElementAccumulator, ElementAccumulator,
187:     // Transpose layout of D here since we use explicit swap + transpose
188:     // the void type for C tells the builder to allocate 0 smem for the C matrix.
189:     // We can enable this if beta == 0 by changing ElementC to void below.
190:     ElementC, typename cutlass::layout::LayoutTranspose<LayoutC>::type, AlignmentC,
191:     ElementD, typename cutlass::layout::LayoutTranspose<LayoutD>::type, AlignmentD,
192:     EpilogueSchedule // This is the only epi supporting the required swap + transpose.
193:   >::CollectiveOp;
194: 
```

- **EN**: This section defines the MMA type, quantized operand type, layouts, scale element type, tile shape, and the specific CUTLASS kernel aliases used by the sample. A subtle but important detail is the introduction of transpose-layout aliases and reordered-layout atoms, because the kernel path is designed around manually arranged operand roles rather than the naive mathematical presentation.
- **CN**: 这一段定义了 MMA 类型、量化操作数类型、布局、scale 元素类型、tile 形状以及示例使用的 CUTLASS kernel 别名。一个很关键但容易忽略的细节是引入了转置布局别名和重排布局 atom，因为 kernel 路径是围绕“手动安排好的操作数角色”构建的，而不是简单照搬数学表达式。

### Lines 195-324 | Options, buffers, and global tensor metadata / 选项、缓冲区与全局张量元数据

```cpp
195: // =========================================================== MIXED INPUT WITH SCALES ===========================================================================
196: // The Scale information must get paired with the operand that will be scaled. In this example, B is scaled so we make a tuple of B's information and the scale information.
197: using CollectiveMainloopScaleOnly = typename cutlass::gemm::collective::CollectiveBuilder<
198:     ArchTag, OperatorClass,
199:     cute::tuple<ElementB, cutlass::Array<ElementScale, 8>>, LayoutB_Transpose, AlignmentB,
200:     ElementA, LayoutA_Transpose, AlignmentA,
201:     ElementAccumulator,
202:     TileShape, ClusterShape,
203:     cutlass::gemm::collective::StageCountAutoCarveout<
204:       static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
205:     >,
206:     KernelSchedule
207:   >::CollectiveOp;
208: 
209: using GemmKernelScaleOnly = cutlass::gemm::kernel::GemmUniversal<
210:     Shape<int,int,int,int>, // Indicates ProblemShape
211:     CollectiveMainloopScaleOnly,
212:     CollectiveEpilogue
213: >;
214: 
215: using CollectiveMainloopShuffled = typename cutlass::gemm::collective::CollectiveBuilder<
216:     ArchTag, OperatorClass,
217:     cute::tuple<ElementB, cutlass::Array<ElementScale, 8>>, LayoutB_Reordered, AlignmentB,
218:     ElementA, LayoutA_Transpose, AlignmentA,
219:     ElementAccumulator,
220:     TileShape, ClusterShape,
221:     cutlass::gemm::collective::StageCountAutoCarveout<
222:       static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
223:     >,
224:     KernelSchedule
225:   >::CollectiveOp;
226: 
227: using GemmKernelShuffled = cutlass::gemm::kernel::GemmUniversal<
228:     Shape<int,int,int,int>, // Indicates ProblemShape
229:     CollectiveMainloopShuffled,
230:     CollectiveEpilogue
231: >;
232: 
233: using GemmScaleOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleOnly>;
234: using GemmShuffled  = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelShuffled>;
235: 
236: using StrideC = typename GemmKernelScaleOnly::StrideC;
237: using StrideD = typename GemmKernelScaleOnly::StrideD;
238: 
239: using StrideC_ref = cutlass::detail::TagToStrideC_t<LayoutC>;
240: using StrideD_ref = cutlass::detail::TagToStrideC_t<LayoutD>;
241: 
242: //
243: // Data members
244: //
245: 
246: /// Initialization
247: StrideA stride_A;
248: StrideB stride_B;
249: StrideC stride_C;
250: StrideC_ref stride_C_ref;
251: StrideD stride_D;
252: StrideD_ref stride_D_ref;
253: uint64_t seed;
254: 
255: LayoutB_Reordered layout_B_reordered;
256: 
257: using StrideS = typename CollectiveMainloopScaleOnly::StrideScale;
258: using StrideS_ref = cutlass::detail::TagToStrideB_t<LayoutScale>;
259: StrideS stride_S;
260: StrideS_ref stride_S_ref;
261: 
262: cutlass::DeviceAllocation<ElementA> block_A;
263: cutlass::DeviceAllocation<ElementB> block_B;
264: cutlass::DeviceAllocation<ElementB> block_B_modified;
265: cutlass::DeviceAllocation<ElementA> block_B_dq;
266: cutlass::DeviceAllocation<ElementScale> block_scale;
267: cutlass::DeviceAllocation<cutlass::Array<ElementScale, 8>> block_scale_packed;
268: cutlass::DeviceAllocation<ElementZero> block_zero;
269: cutlass::DeviceAllocation<ElementC> block_C;
270: cutlass::DeviceAllocation<typename GemmScaleOnly::EpilogueOutputOp::ElementOutput> block_D;
271: cutlass::DeviceAllocation<typename GemmScaleOnly::EpilogueOutputOp::ElementOutput> block_ref_D;
272: 
273: #endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
274: 
275: /////////////////////////////////////////////////////////////////////////////////////////////////
276: /// Testbed utility types
277: /////////////////////////////////////////////////////////////////////////////////////////////////
278: 
279: // Command line options parsing
280: struct Options : MixedDtypeOptions {
281:   bool shuffle = true;
282: 
283:   // Parses the command line
284:   void parse(int argc, char const **args) {
285:     cutlass::CommandLine cmd(argc, args);
286:     cmd.get_cmd_line_argument("shuffle", shuffle);
287: 
288:     this->MixedDtypeOptions::parse(argc, args);
289: 
290:     mode = 1; // override the mode value to always be scale only mode
291:   }
292: 
293:   /// Prints the usage statement.
294:   std::ostream & print_usage(std::ostream &out) const {
295: 
296:     out << "55_hopper_int4_fp8_gemm\n\n"
297:       << "  Hopper Mixed Data Type GEMM using a Warp Specialized kernel.\n\n"
298:       << "Options:\n\n"
299:       << "  --help                      If specified, displays this usage statement\n\n"
300:       << "  --m=<int>                   Sets the M extent of the GEMM\n"
301:       << "  --n=<int>                   Sets the N extent of the GEMM\n"
302:       << "  --k=<int>                   Sets the K extent of the GEMM\n"
303:       << "  --l=<int>                   The number of independent gemm problems with mnk shape\n"
304:       << "  --g=<int>                   The size of each group for the scales. To broadcast a vector of scales or zeros, set the group size to K.\n"
305:       << "  --alpha=<f32>               Epilogue scalar alpha\n"
306:       << "  --beta=<f32>                Epilogue scalar beta\n\n"
307:       << "  --iterations=<int>          Number of profiling iterations to perform.\n\n"
308:       << "  --warmup=<int>              Number of warmup iterations to perform.\n\n"
309:       << "  --shuffle=<boolean>         Enable the offline layout swizzling.\n\n";
310: 
311:     out
312:       << "\n\nExamples:\n\n"
313:       << "$ " << "55_hopper_int4_fp8_gemm" << " --m=1024 --n=512 --k=1024 -g=1024 --l=10 --alpha=2 --beta=0.707 \n\n";
314: 
315:     return out;
316:   }
317: };
318: 
319: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
320: 
321: /////////////////////////////////////////////////////////////////////////////////////////////////
322: /// GEMM setup and evaluation
323: /////////////////////////////////////////////////////////////////////////////////////////////////
324: 
```

- **EN**: The file extends the shared mixed-dtype options with a `shuffle` flag and then declares the global buffers that hold original quantized weights, modified/reordered weights, dequantized references, per-group scales, packed scales, and outputs. The sample forces `mode = 1`, which means it concentrates on the scale-only pathway used by the fused kernel.
- **CN**: 该文件在共享 mixed-dtype 选项的基础上增加了 `shuffle` 开关，并声明保存原始量化权重、修改/重排后的权重、反量化参考值、逐组 scale、打包 scale 以及输出结果的全局缓冲区。示例固定 `mode = 1`，也就是专注于 fused kernel 使用的 scale-only 路径。

### Lines 325-407 | Initialization, packing, and optional offline shuffle / 初始化、打包与可选离线 shuffle

```cpp
325: /// Initialize operands to be used in the GEMM and reference GEMM
326: template <typename QuantType>
327: void initialize(Options const& options) {
328: 
329:   auto shape_B = cute::make_shape(options.n, options.k, options.l);
330:   int const scale_k = cutlass::ceil_div(options.k, options.g);
331:   stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
332:   stride_B = cutlass::make_cute_packed_stride(StrideB{}, shape_B);
333:   // Reverse stride here due to swap and transpose
334:   stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.n, options.m, options.l));
335:   stride_C_ref = cutlass::make_cute_packed_stride(StrideC_ref{}, cute::make_shape(options.m, options.n, options.l));
336:   // Reverse stride here due to swap and transpose
337:   stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.n, options.m, options.l));
338:   stride_D_ref = cutlass::make_cute_packed_stride(StrideD_ref{}, cute::make_shape(options.m, options.n, options.l));
339: 
340:   auto layout_B = make_layout(shape_B, stride_B);
341: 
342:   auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
343:   auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
344:   auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
345: 
346:   block_A.reset(a_coord.product());
347:   block_B.reset(b_coord.product());
348:   block_B_modified.reset(b_coord.product());
349:   block_B_dq.reset(b_coord.product());
350:   block_C.reset(c_coord.product());
351:   block_D.reset(c_coord.product());
352:   block_ref_D.reset(c_coord.product());
353: 
354:   block_scale.reset(scale_k * options.l * options.n);
355:   block_scale_packed.reset(scale_k * options.l * options.n);
356:   block_zero.reset(scale_k * options.l * options.n);
357: 
358:   // Initialize all base tensors
359:   initialize_tensor(block_A, seed + 2022);
360:   initialize_tensor(block_B, seed + 2021);
361:   initialize_tensor(block_C, seed + 2020);
362:   initialize_scale(block_scale, options);
363:   cutlass::pack_scale_fp8<ElementScale, QuantType>(block_scale.get(), block_scale_packed.get(), block_scale.size());
364:   initialize_zero(block_zero, options);
365: 
366:   // Compute dequantized reference for validation BEFORE formatting block_B
367:   auto shape_scale_zero = cute::make_shape(options.n, scale_k, options.l);
368:   stride_S = cutlass::make_cute_packed_stride(StrideS{}, cute::make_shape(options.n, scale_k, options.l));
369:   stride_S_ref = cutlass::make_cute_packed_stride(StrideS_ref{}, cute::make_shape(options.n, scale_k, options.l));
370:   auto layout_scale_zero = make_layout(shape_scale_zero, stride_S_ref);
371: 
372:   cudaStream_t stream = cudaStreamDefault;
373:   cutlass::dequantize(block_B_dq.get(), block_B.get(), layout_B, block_scale.get(), block_zero.get(), layout_scale_zero, options.g, stream);
374: 
375:   // Format B to separate buffer, preserving original block_B in case
376:   // it's needed by the application.
377: 
378:   #ifdef CUTLASS_MIXED_DTYPE_E2M1
379:     // E2M1: Copy to formatting buffer
380:     cutlass::device_memory::copy_device_to_device(block_B_modified.get(), block_B.get(), block_B.size());
381:   #else
382:     // INT4: Encode to formatting buffer
383:     cutlass::unified_encode_int4b(block_B.get(), block_B_modified.get(), block_B.size());
384:   #endif
385: 
386: 
387:   // original code
388:   // if constexpr (cutlass::platform::is_floating_point<QuantType>::value ||
389:   //              cute::is_same_v<QuantType, cutlass::float_e2m1_t>) {
390:   //  // E2M1: Copy to formatting buffer
391:   //  cutlass::device_memory::copy_device_to_device(block_B_modified.get(), block_B.get(), block_B.size());
392:   // } else {
393:   //  // INT4: Encode to formatting buffer
394:   //  cutlass::unified_encode_int4b(block_B.get(), block_B_modified.get(), block_B.size());
395:   // }
396: 
397:   if (options.shuffle) {
398:     // Repeat the reorder layout atom to tile the whole tensor shape
399:     layout_B_reordered = cute::tile_to_shape(LayoutAtomQuant{}, shape_B);
400:     cutlass::reorder_tensor(block_B_modified.get(), layout_B, layout_B_reordered);
401: 
402:     print("Quantized tensor layout: ");
403:     print(layout_B_reordered);
404:     print("\n");
405:   }
406: }
407: 
```

- **EN**: This is the heart of the sample setup. It computes scale shapes from `group_size`, allocates tensors, initializes data, packs groups of eight scales into `cutlass::Array<ElementScale, 8>`, explicitly encodes INT4 values with the unified encoding helper, and optionally performs an offline reorder so each thread later sees a contiguous fragment. That reorder is a memory-layout optimization, not a mathematical transformation.
- **CN**: 这是示例初始化阶段的核心。代码根据 `group_size` 计算 scale 形状，分配张量，初始化数据，把每 8 个 scale 打包为 `cutlass::Array<ElementScale, 8>`，再通过统一编码辅助函数显式编码 INT4，并可选地执行一次离线重排，使后续每个线程读取到的片段在内存中更连续。这个重排是访存布局优化，而不是数学意义上的变换。

### Lines 408-493 | Kernel arguments and reference verification / Kernel 参数与参考验证

```cpp
408: /// Populates a Gemm::Arguments structure from the given commandline options
409: /// Swap the A and B tensors, as well as problem shapes here.
410: template <typename Gemm>
411: typename Gemm::Arguments args_from_options(Options const& options)
412: {
413:   using Args = typename Gemm::Arguments;
414:   auto&& dB = [&]() {
415:     if constexpr (cute::is_same_v<Gemm, GemmShuffled>) { // offline swizzling is enabled.
416:       return layout_B_reordered;
417:     }
418:     else {
419:       return stride_B;
420:     }
421:   }();
422:   return Args {
423:     cutlass::gemm::GemmUniversalMode::kGemm,
424:     {options.n, options.m, options.k, options.l},
425:     {block_B_modified.get(), dB, block_A.get(), stride_A, block_scale_packed.get(), stride_S, options.g},
426:     {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
427:   };
428: }
429: 
430: bool verify(Options const& options) {
431:   //
432:   // Compute reference output
433:   //
434: 
435:   // In this example, we use the GPU default kernels as a reference (unfused scale).
436:   // This avoids numerical differences due to different accumulation order.
437: 
438:   // Again, due to numerical differences, we must use fast acc here when the mma type is
439:   // FP8 as the fused implementation only supports fast acc at the moment.
440:   constexpr bool IsFP8Input = cute::is_same_v<MmaType, cutlass::float_e4m3_t> || cute::is_same_v<MmaType, cutlass::float_e5m2_t>;
441:   using FP8Sched = cute::conditional_t<size<0>(TileShape{}) == 64, cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8FastAccum, cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8FastAccum>;
442:   using ScheduleRef = cute::conditional_t<IsFP8Input, FP8Sched, cutlass::gemm::collective::KernelScheduleAuto>;
443: 
444:   using CollectiveMainloopRef = typename cutlass::gemm::collective::CollectiveBuilder<
445:       ArchTag, OperatorClass,
446:       MmaType, LayoutA, AlignmentA,
447:       MmaType, LayoutB, AlignmentB,
448:       ElementAccumulator,
449:       TileShape, ClusterShape,
450:       cutlass::gemm::collective::StageCountAuto,
451:       ScheduleRef
452:     >::CollectiveOp;
453: 
454:   using CollectiveEpilogueRef = typename cutlass::epilogue::collective::CollectiveBuilder<
455:       cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
456:       TileShape, ClusterShape,
457:       cutlass::epilogue::collective::EpilogueTileAuto,
458:       ElementAccumulator, ElementAccumulator,
459:       ElementC, LayoutC, AlignmentC,
460:       ElementD, LayoutD, AlignmentD,
461:       cutlass::epilogue::NoSmemWarpSpecialized
462:     >::CollectiveOp;
463: 
464:   using GemmKernelRef = cutlass::gemm::kernel::GemmUniversal<
465:       Shape<int,int,int,int>, // Indicates ProblemShape
466:       CollectiveMainloopRef,
467:       CollectiveEpilogueRef
468:   >;
469: 
470:   using GemmRef = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelRef>;
471: 
472:   typename GemmRef::Arguments arguments{
473:     cutlass::gemm::GemmUniversalMode::kGemm,
474:     {options.m, options.n, options.k, options.l},
475:     {block_A.get(), stride_A, block_B_dq.get(), stride_B},
476:     {{options.alpha, options.beta}, block_C.get(), stride_C_ref, block_ref_D.get(), stride_D_ref}
477:   };
478: 
479:   // Run the gemm where the scaling is performed outside of the kernel.
480:   GemmRef gemm_ref;
481:   size_t workspace_size = GemmRef::get_workspace_size(arguments);
482:   cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
483:   CUTLASS_CHECK(gemm_ref.can_implement(arguments));
484:   CUTLASS_CHECK(gemm_ref.initialize(arguments, workspace.get()));
485:   CUTLASS_CHECK(gemm_ref.run());
486: 
487:   // compare_reference
488:   ElementD const epsilon(1e-2f);
489:   ElementD const non_zero_floor(1e-4f);
490:   bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_D.get(), block_D.get(), block_D.size(), epsilon, non_zero_floor);
491: 
492:   return passed;
493: }
```

- **EN**: The launch arguments intentionally swap A and B and also swap the M/N positions in the problem shape. This is how the sample forces the narrow quantized tensor through the intended path while keeping the fused epilogue viable. The verification path then runs a reference GEMM on an explicitly dequantized B tensor so correctness can be checked independently of the fused dequantization path.
- **CN**: 启动参数会刻意交换 A/B，同时也交换 problem shape 中的 M/N 位置。这样做是为了让窄量化张量走到期望的数据通路上，同时保留 fused epilogue 的可用性。验证路径随后会对显式反量化后的 B 张量执行参考 GEMM，从而把正确性检查与 fused 反量化路径分离开来。

### Lines 494-597 | Run flow, architecture guards, and mode selection / 运行流程、架构保护与模式选择

```cpp
494: 
495: /// Execute a given example GEMM computation
496: template <typename Gemm>
497: int run(Options &options)
498: {
499:   initialize<QuantType>(options);
500: 
501:   // Instantiate CUTLASS kernel depending on templates
502:   Gemm gemm;
503: 
504:   // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
505:   auto arguments = args_from_options<Gemm>(options);
506: 
507:   // Using the arguments, query for extra workspace required for matrix multiplication computation
508:   size_t workspace_size = Gemm::get_workspace_size(arguments);
509: 
510:   // Allocate workspace memory
511:   cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
512: 
513:   // Check if the problem size is supported or not
514:   CUTLASS_CHECK(gemm.can_implement(arguments));
515: 
516:   // Initialize CUTLASS kernel with arguments and workspace pointer
517:   CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
518: 
519:   // Correctness / Warmup iteration
520:   CUTLASS_CHECK(gemm.run());
521: 
522:   // Check if output from CUTLASS kernel and reference kernel are equal or not
523:   MixedDtypeResult result;
524:   result.passed = verify(options);
525:   mixed_dtype_profiling(gemm, options, result);
526:   std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
527:   if (!result.passed) {
528:     exit(-1);
529:   }
530: 
531:   return 0;
532: }
533: 
534: #endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
535: 
536: ///////////////////////////////////////////////////////////////////////////////////////////////////
537: 
538: int main(int argc, char const **args) {
539: 
540:   // CUTLASS must be compiled with CUDA 12.0 Toolkit to run this example
541:   // and must have compute capability at least 90.
542:   if (__CUDACC_VER_MAJOR__ < 12) {
543:     std::cerr << "This example requires CUDA 12 or newer.\n";
544:     // Returning zero so this test passes on older Toolkits. Its actions are no-op.
545:     return 0;
546:   }
547: 
548:   cudaDeviceProp props;
549:   int current_device_id;
550:   CUDA_CHECK(cudaGetDevice(&current_device_id));
551:   CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
552:   cudaError_t error = cudaGetDeviceProperties(&props, 0);
553:   if (props.major != 9 || props.minor != 0) {
554:     std::cerr
555:       << "This example requires a GPU of NVIDIA's Hopper Architecture (compute capability 90).\n";
556:     return 0;
557:   }
558: 
559: 
560: 
561: 
562:   //
563:   // Parse options
564:   //
565: 
566:   Options options;
567: 
568:   options.parse(argc, args);
569: 
570:   if (options.help) {
571:     options.print_usage(std::cout) << std::endl;
572:     return 0;
573:   }
574: 
575:   //
576:   // Evaluate CUTLASS kernels
577:   //
578: 
579: #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
580:   if (options.g == options.k) {
581:     std::cout << "Running in per-column scale mode." << std::endl;
582:   } else {
583:     std::cout << "Running in group scale mode." << std::endl;
584:   }
585:   if (options.shuffle) {
586:     std::cout << "Offline shuffle enabled." << std::endl;
587:     run<GemmShuffled>(options);
588:   } else {
589:     std::cout << "Offline shuffle disabled." << std::endl;
590:     run<GemmScaleOnly>(options);
591:   }
592: #endif
593: 
594:   return 0;
595: }
596: 
597: /////////////////////////////////////////////////////////////////////////////////////////////////
```

- **EN**: The tail of the file performs the standard CUTLASS launch lifecycle, selects the shuffled or non-shuffled kernel alias, prints whether the run corresponds to per-column or group scaling, and checks CUDA/SM90 requirements before entering the benchmark. It turns the example into a small laboratory for mixed-dtype layout experiments.
- **CN**: 文件尾部完成标准的 CUTLASS 启动生命周期，在 shuffle 与非 shuffle kernel 之间做选择，打印当前是 per-column scale 还是 group scale 模式，并在真正执行前检查 CUDA/SM90 前提。因此整个示例就像一个可直接试验混合精度布局方案的小型实验室。

---

## Key Concepts / 关键概念

- **EN**: Packed scale representation: eight scale values are grouped into one `cutlass::Array<ElementScale, 8>` object so the kernel consumes scaling metadata efficiently.
  **CN**: 打包 scale 表示：8 个 scale 会被组合成一个 `cutlass::Array<ElementScale, 8>`，便于 kernel 高效消费缩放元数据。
- **EN**: Offline shuffle changes only memory order, making the quantized operand friendlier for later per-thread vector loads.
  **CN**: 离线 shuffle 只改变内存顺序，使量化操作数更适合后续按线程进行向量化加载。
- **EN**: Manual operand swapping is used to preserve a favorable Hopper data path and keep TMA-based epilogues available.
  **CN**: 手动交换操作数是为了保留更理想的 Hopper 数据通路，并继续使用基于 TMA 的 epilogue。
- **EN**: Reference verification dequantizes outside the kernel so numerical comparison does not depend on the fused implementation details.
  **CN**: 参考验证会在 kernel 外部先反量化，这样数值比较就不依赖 fused 实现细节。

## Dependencies / 依赖项

- **`mixed_dtype_utils.hpp`**  
  **EN**: Provides helper logic for scale packing, INT4 encoding, and mixed-dtype setup.  
  **CN**: 提供 scale 打包、INT4 编码和混合精度初始化所需的辅助逻辑。
- **CUTLASS GEMM collective/device APIs**  
  **EN**: Used to build the fused mixed-dtype kernels and the reference kernel.  
  **CN**: 用于构建 fused 混合精度 kernel 和参考 kernel。
- **CuTe tensor/layout utilities**  
  **EN**: Define transposed and reordered layouts for quantized operands.  
  **CN**: 用于定义量化操作数的转置与重排布局。
- **`helper.h` and CUTLASS reference utilities**  
  **EN**: Support initialization, timing, and output comparison.  
  **CN**: 支持初始化、计时和输出比较。
