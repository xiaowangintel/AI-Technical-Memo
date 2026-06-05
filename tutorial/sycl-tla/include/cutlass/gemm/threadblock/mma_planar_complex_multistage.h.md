# mma_planar_complex_multistage.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/threadblock/mma_planar_complex_multistage.h`
- **Purpose (EN):** Implements threadblock-scoped GEMM building blocks and pipeline logic.
- **用途 (CN):** 实现线程块级 GEMM 构件与流水线逻辑。

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
32:     \brief Template for a double-buffered threadblock-scoped GEMM kernel.
33: */
34: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 35-38
```cpp
35: #pragma once
36: 
37: #include "cutlass/cutlass.h"
38: 
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 39-48
```cpp
39: #include "cutlass/aligned_buffer.h"
40: #include "cutlass/arch/memory.h"
41: #include "cutlass/array.h"
42: #include "cutlass/array_planar_complex.h"
43: #include "cutlass/functional.h"
44: #include "cutlass/matrix_shape.h"
45: #include "cutlass/numeric_types.h"
46: 
47: #include "cutlass/gemm/gemm.h"
48: #include "cutlass/gemm/threadblock/mma_planar_complex_base.h"
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, architecture intrinsics, numeric types/converters, threadblock components.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、架构内建/指令封装、数值类型/转换器、线程块组件。

### Lines 49-59
```cpp
49: 
50: /////////////////////////////////////////////////////////////////////////////////////////////////
51: 
52: namespace cutlass {
53: namespace gemm {
54: namespace threadblock {
55: 
56: /////////////////////////////////////////////////////////////////////////////////////////////////
57: 
58: /// Structure to compute the matrix product targeting CUDA cores and SIMT math
59: /// instructions.
```
**EN:** Enters namespace scope (cutlass::gemm::threadblock) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass::gemm::threadblock），组织 GEMM 抽象层。

### Lines 60-61
```cpp
60: template <
61:     /// Size of the Gemm problem - concept: gemm::GemmShape<>
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 62-65
```cpp
62:     typename Shape_,
63:     /// Iterates over tiles of A operand in global memory
64:     //  (concept: ReadableTileIterator | ForwardTileIterator |
65:     //  MaskedTileIterator)
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 66-70
```cpp
66:     typename IteratorA_,
67:     /// Iterates over tiles of A operand in shared memory
68:     /// (concept: WriteableTileIterator | RandomAccessTileIterator)
69:     typename SmemIteratorA_,
70:     /// Cache operation for operand A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 71-74
```cpp
71:     cutlass::arch::CacheOperation::Kind CacheOpA,
72:     /// Iterates over tiles of B operand in global memory
73:     //  (concept: ReadableTileIterator | ForwardTileIterator |
74:     //  MaskedTileIterator)
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 75-85
```cpp
75:     typename IteratorB_,
76:     /// Iterates over tiles of B operand in shared memory
77:     /// (concept: WriteableTileIterator | RandomAccessTileIterator)
78:     typename SmemIteratorB_,
79:     /// Cache operation for operand B
80:     cutlass::arch::CacheOperation::Kind CacheOpB,
81:     /// Data type of accumulator matrix
82:     typename ElementC_,
83:     /// Data type of accumulator matrix
84:     typename LayoutC_,
85:     /// Policy describing tuning details (concept: MmaPolicy)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 86-91
```cpp
86:     typename Policy_,
87:     /// Number of stages,
88:     int Stages,
89:     /// Transformation applied to A
90:     ComplexTransform TransformA = ComplexTransform::kNone,
91:     /// Transformation applied to B
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 92-102
```cpp
92:     ComplexTransform TransformB = ComplexTransform::kNone
93: >
94: class MmaPlanarComplexMultistage : 
95:   public MmaPlanarComplexBase<Shape_, Policy_, Stages> {
96: public:
97:   ///< Base class
98:   using Base = MmaPlanarComplexBase<Shape_, Policy_, Stages>;
99: 
100:   ///< Size of the Gemm problem - concept: gemm::GemmShape<>
101:   using Shape = Shape_;
102: 
```
**EN:** Defines MmaPlanarComplexMultistage, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MmaPlanarComplexMultistage，用于封装策略、存储或算法行为的辅助类型。

### Lines 103-112
```cpp
103:   ///< Iterates over tiles of A operand in global memory
104:   using IteratorA = IteratorA_;
105: 
106:   ///< Iterates over tiles of B operand in global memory
107:   using IteratorB = IteratorB_;
108: 
109:   ///< Data type of accumulator matrix
110:   using ElementC = ElementC_;
111: 
112:   ///< Layout of accumulator matrix
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 113-120
```cpp
113:   using LayoutC = LayoutC_;
114: 
115:   ///< Policy describing tuning details
116:   using Policy = Policy_;
117: 
118:   ///< Architecture tag
119:   using ArchTag = arch::Sm80;
120: 
```
**EN:** Introduces local type aliases (LayoutC, Policy, ArchTag) to simplify downstream template code.
**CN:** 引入本地类型别名（LayoutC, Policy, ArchTag），简化后续模板代码。

### Lines 121-123
```cpp
121:   using SmemIteratorA = SmemIteratorA_;
122:   using SmemIteratorB = SmemIteratorB_;
123: 
```
**EN:** Introduces local type aliases (SmemIteratorA, SmemIteratorB) to simplify downstream template code.
**CN:** 引入本地类型别名（SmemIteratorA, SmemIteratorB），简化后续模板代码。

### Lines 124-132
```cpp
124:   static cutlass::arch::CacheOperation::Kind const kCacheOpA = CacheOpA;
125:   static cutlass::arch::CacheOperation::Kind const kCacheOpB = CacheOpB;
126: 
127:   /// Transformation applied to A
128:   static ComplexTransform const kTransformA = TransformA;
129: 
130:   /// Transformation applied to B
131:   static ComplexTransform const kTransformB = TransformB;
132: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 133-137
```cpp
133:   //
134:   // Dependent types
135:   //
136: 
137:   /// Fragment of accumulator tile
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 138-148
```cpp
138:   using FragmentC = ArrayPlanarComplex<
139:     typename Policy::Operator::FragmentC::Element,
140:     Policy::Operator::FragmentC::kElements
141:   >;
142: 
143:   /// Warp-level Mma
144:   using Operator = typename Policy::Operator;
145: 
146:   /// Internal structure exposed for introspection.
147:   struct Detail {
148: 
```
**EN:** Defines Detail, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 Detail，用于封装策略、存储或算法行为的辅助类型。

### Lines 149-153
```cpp
149:     static_assert(Base::kWarpGemmIterations > 1,
150:                   "The pipelined structure requires at least two warp-level "
151:                   "GEMM operations.");
152: 
153:     /// Number of cp.async instructions to load one stage of operand A
```
**EN:** This block focuses on cp.async related implementation details.
**CN:** 该代码块聚焦于 cp.async 异步拷贝 的实现细节。

### Lines 154-157
```cpp
154:     static int const TBLoadIterationsA =
155:         IteratorA::ThreadMap::Iterations::kCount;
156: 
157:     /// Number of cp.async instructions to load one stage of operand B
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 158-163
```cpp
158:     static int const TBLoadIterationsB =
159:         IteratorB::ThreadMap::Iterations::kCount;
160: 
161:     /// Number of stages
162:     static int const kStages = Stages;
163: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 164-166
```cpp
164:     static int const kAccessesPerGroupA =
165:         (TBLoadIterationsA + Base::kWarpGemmIterations - 1) / Base::kWarpGemmIterations;
166: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 167-172
```cpp
167:     static int const kAccessesPerGroupB =
168:         (TBLoadIterationsB + Base::kWarpGemmIterations - 1) / Base::kWarpGemmIterations;
169:   };
170: 
171:  private:
172: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 173-177
```cpp
173:   using WarpFragmentA = typename Operator::FragmentA;
174:   using WarpFragmentB = typename Operator::FragmentB;
175: 
176:  private:
177: 
```
**EN:** Introduces local type aliases (WarpFragmentA, WarpFragmentB) to simplify downstream template code.
**CN:** 引入本地类型别名（WarpFragmentA, WarpFragmentB），简化后续模板代码。

### Lines 178-187
```cpp
178:   //
179:   // Data members
180:   //
181: 
182:   /// Iterator to write threadblock-scoped tile of A operand to shared memory
183:   SmemIteratorA smem_iterator_A_;
184: 
185:   /// Iterator to write threadblock-scoped tile of B operand to shared memory
186:   SmemIteratorB smem_iterator_B_;
187: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 188-190
```cpp
188: public:
189: 
190:   /// Construct from tensor references
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 191-199
```cpp
191:   CUTLASS_DEVICE
192:   MmaPlanarComplexMultistage(
193:       ///< Shared storage needed for internal use by threadblock-scoped GEMM
194:       typename Base::SharedStorage &shared_storage,
195:       ///< ID within the threadblock
196:       int thread_idx,
197:       ///< ID of warp
198:       int warp_idx,
199:       ///< ID of each thread within a warp
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 200-205
```cpp
200:       int lane_idx
201:     ):
202:       Base(shared_storage, thread_idx, warp_idx, lane_idx),
203:       smem_iterator_A_(shared_storage.operand_A_ref(), thread_idx),
204:       smem_iterator_B_(shared_storage.operand_B_ref(), thread_idx)
205:   {
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 206-211
```cpp
206:     // Compute warp location within threadblock tile by mapping the warp_id to
207:     // three coordinates:
208:     //   _m: the warp's position within the threadblock along the M dimension
209:     //   _n: the warp's position within the threadblock along the N dimension
210:     //   _k: the warp's position within the threadblock along the K dimension
211: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 212-214
```cpp
212:     int warp_idx_mn = warp_idx % (Base::WarpCount::kM * Base::WarpCount::kN);
213:     int warp_idx_k = warp_idx / (Base::WarpCount::kM * Base::WarpCount::kN);
214: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 215-218
```cpp
215:     int warp_idx_m = warp_idx_mn % Base::WarpCount::kM;
216:     int warp_idx_n = warp_idx_mn / Base::WarpCount::kM;
217: 
218:     // Add per-warp offsets in units of warp-level tiles
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 219-224
```cpp
219:     this->warp_tile_iterator_A_.add_tile_offset({warp_idx_m, Base::kWarpGemmIterations * warp_idx_k});
220:     this->warp_tile_iterator_B_.add_tile_offset({Base::kWarpGemmIterations * warp_idx_k, warp_idx_n});
221:   }
222: 
223: private:
224: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 225-229
```cpp
225:   CUTLASS_DEVICE
226:   void copy_tiles_and_advance(
227:     IteratorA &iterator_A_real,
228:     IteratorA &iterator_A_imag,
229:     
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 230-232
```cpp
230:     IteratorB &iterator_B_real, 
231:     IteratorB &iterator_B_imag, 
232:     
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 233-235
```cpp
233:     int group_start_A = 0, 
234:     int group_start_B = 0) {
235: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 236-240
```cpp
236:     iterator_A_real.set_iteration_index(group_start_A * IteratorA::kAccessesPerVector);
237:     iterator_A_imag.set_iteration_index(group_start_A * IteratorA::kAccessesPerVector);
238:     this->smem_iterator_A_.set_iteration_index(group_start_A);
239: 
240:     // Load for operand A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 241-243
```cpp
241:     CUTLASS_PRAGMA_UNROLL
242:     for (int j = 0; j < Detail::kAccessesPerGroupA; ++j) {
243:         
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 244-246
```cpp
244:       typename IteratorA::AccessType *dst_ptr = 
245:         reinterpret_cast<typename IteratorA::AccessType *>(this->smem_iterator_A_.get());
246:           
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 247-250
```cpp
247:       int const kSrcBytes = 
248:         sizeof_bits<typename IteratorA::Element>::value * 
249:         IteratorA::ThreadMap::kElementsPerAccess / IteratorA::kAccessesPerVector / 8;
250: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 251-253
```cpp
251:       CUTLASS_PRAGMA_UNROLL
252:       for (int v = 0; v < IteratorA::kAccessesPerVector; ++v) {
253: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 254-256
```cpp
254:         auto gmem_ptr_real = iterator_A_real.get();
255:         auto gmem_ptr_imag = iterator_A_imag.get();
256: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 257-266
```cpp
257:         bool pred_guard = iterator_A_real.valid();
258:         cutlass::arch::cp_async<kSrcBytes, kCacheOpA>(
259:             dst_ptr + v,
260:             gmem_ptr_real,
261:             pred_guard);
262:         cutlass::arch::cp_async<kSrcBytes, kCacheOpA>(
263:             dst_ptr + v + (Base::SharedStorage::kImaginaryStrideA / IteratorA::ThreadMap::kElementsPerAccess),
264:             reinterpret_cast<char const *>(gmem_ptr_imag),
265:             pred_guard);
266: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 267-273
```cpp
267:         ++iterator_A_real;
268:         ++iterator_A_imag;
269:       }
270: 
271:       ++this->smem_iterator_A_;
272:     }
273: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 274-278
```cpp
274:     iterator_B_real.set_iteration_index(group_start_B * IteratorB::kAccessesPerVector);
275:     iterator_B_imag.set_iteration_index(group_start_B * IteratorB::kAccessesPerVector);
276:     this->smem_iterator_B_.set_iteration_index(group_start_B);
277: 
278:     // Load for operand B
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 279-283
```cpp
279:     CUTLASS_PRAGMA_UNROLL
280:     for (int j = 0; j < Detail::kAccessesPerGroupB; ++j) {
281:       typename IteratorB::AccessType *dst_ptr = 
282:         reinterpret_cast<typename IteratorB::AccessType *>(this->smem_iterator_B_.get());
283:       
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 284-287
```cpp
284:       int const kSrcBytes = 
285:         sizeof_bits<typename IteratorB::Element>::value * 
286:         IteratorB::ThreadMap::kElementsPerAccess / IteratorB::kAccessesPerVector / 8;
287: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 288-292
```cpp
288:       CUTLASS_PRAGMA_UNROLL
289:       for (int v = 0; v < IteratorB::kAccessesPerVector; ++v) {
290:         auto gmem_ptr_real = iterator_B_real.get();
291:         auto gmem_ptr_imag = iterator_B_imag.get();
292: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 293-302
```cpp
293:         bool pred_guard = iterator_B_real.valid();
294:         cutlass::arch::cp_async<kSrcBytes, kCacheOpB>(
295:             dst_ptr + v,
296:             gmem_ptr_real,
297:             pred_guard);
298:         cutlass::arch::cp_async<kSrcBytes, kCacheOpB>(
299:             dst_ptr + v + (Base::SharedStorage::kImaginaryStrideB / IteratorB::ThreadMap::kElementsPerAccess),
300:             reinterpret_cast<char const *>(gmem_ptr_imag),
301:             pred_guard);
302: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 303-309
```cpp
303:         ++iterator_B_real;
304:         ++iterator_B_imag;
305:       }
306:       ++this->smem_iterator_B_;
307:     }
308:   }
309: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 310-320
```cpp
310:   CUTLASS_DEVICE
311:   void warp_mma_planar_complex(
312:     Operator & warp_mma, 
313:     FragmentC &accum,
314:     WarpFragmentA const & real_A, 
315:     WarpFragmentA const & imag_A, 
316:     WarpFragmentB const & real_B, 
317:     WarpFragmentB const & imag_B) {
318: 
319:     cutlass::negate<Array<typename WarpFragmentB::Element, WarpFragmentB::kElements>> neg_op_B;
320: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 321-325
```cpp
321:     WarpFragmentB neg_real_B = neg_op_B(real_B);
322:     WarpFragmentB neg_imag_B = neg_op_B(imag_B);
323: 
324:     warp_mma(accum.real, real_A, real_B, accum.real);  
325: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 326-328
```cpp
326:     if (kTransformB == ComplexTransform::kNone) {
327:       warp_mma(accum.imag, real_A, imag_B, accum.imag);
328:     }
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 329-332
```cpp
329:     else {
330:       warp_mma(accum.imag, real_A, neg_imag_B, accum.imag);
331:     }
332: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 333-335
```cpp
333:     if (kTransformA == ComplexTransform::kNone) {
334:       warp_mma(accum.imag, imag_A, real_B, accum.imag);
335:     }
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 336-339
```cpp
336:     else {
337:       warp_mma(accum.imag, imag_A, neg_real_B, accum.imag);
338:     }
339: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 340-342
```cpp
340:     if (kTransformA == ComplexTransform::kNone ^ kTransformB == ComplexTransform::kNone) {
341:       warp_mma(accum.real, imag_A, imag_B, accum.real);
342:     }
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 343-350
```cpp
343:     else {
344:       warp_mma(accum.real, imag_A, neg_imag_B, accum.real);
345:     }
346:   }
347: 
348: public:
349:   
350:   /// Perform a threadblock-scoped matrix multiply-accumulate
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 351-361
```cpp
351:   CUTLASS_DEVICE
352:   void operator()(
353:       ///< problem size of GEMM
354:       int gemm_k_iterations,
355:       ///< destination accumulator tile
356:       FragmentC &accum,
357:       ///< iterator over A operand in global memory
358:       IteratorA iterator_A_real,
359:       ///< iterator over A operand in global memory
360:       IteratorA iterator_A_imag,
361:       ///< iterator over B operand in global memory
```
**EN:** Implements the functor call operator that performs the core compute or data-movement step.
**CN:** 实现函数对象调用运算符，执行核心计算或数据搬运步骤。

### Lines 362-367
```cpp
362:       IteratorB iterator_B_real,
363:       ///< iterator over B operand in global memory
364:       IteratorB iterator_B_imag,
365:       ///< initial value of accumulator
366:       FragmentC const &src_accum) {
367: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 368-372
```cpp
368:     //
369:     // Prologue
370:     //
371: 
372:     // Issue several complete stages
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 373-376
```cpp
373:     CUTLASS_PRAGMA_UNROLL
374:     for (int stage = 0; stage < Base::kStages - 1;
375:          ++stage, --gemm_k_iterations) {
376: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 377-381
```cpp
377:       iterator_A_real.clear_mask(gemm_k_iterations == 0);
378:       iterator_A_imag.clear_mask(gemm_k_iterations == 0);
379:       iterator_B_real.clear_mask(gemm_k_iterations == 0);
380:       iterator_B_imag.clear_mask(gemm_k_iterations == 0);
381: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 382-387
```cpp
382:       iterator_A_real.set_iteration_index(0);
383:       iterator_A_imag.set_iteration_index(0);
384: 
385:       this->smem_iterator_A_.set_iteration_index(0);
386: 
387:       // Load for operand A
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 388-390
```cpp
388:       CUTLASS_PRAGMA_UNROLL
389:       for (int j = 0; j < Detail::TBLoadIterationsA; ++j) {
390: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 391-393
```cpp
391:         typename IteratorA::AccessType *dst_ptr = 
392:           reinterpret_cast<typename IteratorA::AccessType *>(this->smem_iterator_A_.get());
393: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 394-396
```cpp
394:         CUTLASS_PRAGMA_UNROLL
395:         for (int v = 0; v < IteratorA::kAccessesPerVector; ++v) {
396: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 397-402
```cpp
397:           int const kSrcBytes = 
398:             sizeof_bits<typename IteratorA::Element>::value * 
399:             IteratorA::ThreadMap::kElementsPerAccess / IteratorA::kAccessesPerVector / 8;
400: 
401:           bool pred_guard = iterator_A_real.valid();
402: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 403-405
```cpp
403:           auto src_ptr_real = iterator_A_real.get();
404:           auto src_ptr_imag = iterator_A_imag.get();
405: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 406-408
```cpp
406:           cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA>(
407:               dst_ptr + v, src_ptr_real, pred_guard);
408: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 409-415
```cpp
409:           cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA>(
410:               dst_ptr + v +
411:                   Base::SharedStorage::kImaginaryStrideA /
412:                       IteratorA::ThreadMap::kElementsPerAccess,
413:               reinterpret_cast<char const *>(src_ptr_imag),
414:               pred_guard);
415: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 416-422
```cpp
416:           ++iterator_A_real;
417:           ++iterator_A_imag;
418:         }
419: 
420:         ++this->smem_iterator_A_;
421:       }
422: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 423-428
```cpp
423:       iterator_B_real.set_iteration_index(0);
424:       iterator_B_imag.set_iteration_index(0);
425: 
426:       this->smem_iterator_B_.set_iteration_index(0);
427: 
428:       // Load for operand B
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 429-431
```cpp
429:       CUTLASS_PRAGMA_UNROLL
430:       for (int j = 0; j < Detail::TBLoadIterationsB; ++j) {
431: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 432-434
```cpp
432:         typename IteratorB::AccessType *dst_ptr = 
433:           reinterpret_cast<typename IteratorB::AccessType *>(this->smem_iterator_B_.get());
434: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 435-437
```cpp
435:         CUTLASS_PRAGMA_UNROLL
436:         for (int v = 0; v < IteratorB::kAccessesPerVector; ++v) {
437: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 438-443
```cpp
438:           int const kSrcBytes = 
439:             sizeof_bits<typename IteratorB::Element>::value * 
440:             IteratorB::ThreadMap::kElementsPerAccess / IteratorB::kAccessesPerVector / 8;
441: 
442:           bool pred_guard = iterator_B_real.valid();
443: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 444-446
```cpp
444:           auto src_ptr_real = iterator_B_real.get();
445:           auto src_ptr_imag = iterator_B_imag.get();
446: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 447-449
```cpp
447:           cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
448:             dst_ptr + v, src_ptr_real, pred_guard);
449: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 450-456
```cpp
450:           cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
451:               dst_ptr + v +
452:                   Base::SharedStorage::kImaginaryStrideB /
453:                       IteratorB::ThreadMap::kElementsPerAccess,
454:               reinterpret_cast<char const *>(src_ptr_imag),
455:               pred_guard);
456: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 457-464
```cpp
457:           ++iterator_B_real;
458:           ++iterator_B_imag;
459:         }
460: 
461:         ++this->smem_iterator_B_;
462:       }
463: 
464:       // Move to the next stage
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 465-467
```cpp
465:       iterator_A_real.add_tile_offset({0, 1});
466:       iterator_A_imag.add_tile_offset({0, 1});
467: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 468-470
```cpp
468:       iterator_B_real.add_tile_offset({1, 0});
469:       iterator_B_imag.add_tile_offset({1, 0});
470: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 471-480
```cpp
471:       this->smem_iterator_A_.add_tile_offset({0, 1});
472:       this->smem_iterator_B_.add_tile_offset({1, 0});
473: 
474:       // Inserts a memory fence between stages of cp.async instructions
475:       cutlass::arch::cp_async_fence();
476:     }
477: 
478:     // Perform accumulation in the 'd' output operand
479:     accum = src_accum;
480: 
```
**EN:** This block focuses on cp.async, iterator related implementation details.
**CN:** 该代码块聚焦于 cp.async 异步拷贝、迭代器逻辑 的实现细节。

### Lines 481-481
```cpp
481:     // Blocks until all but kStages-2 cp.async stages have committed.
```
**EN:** This block focuses on cp.async related implementation details.
**CN:** 该代码块聚焦于 cp.async 异步拷贝 的实现细节。

### Lines 482-487
```cpp
482:     cutlass::arch::cp_async_wait<Base::kStages - 2>();
483:     __syncthreads();
484: 
485:     // Pair of fragments used to overlap shared memory loads and math
486:     // instructions
487: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 488-490
```cpp
488:     WarpFragmentA warp_frag_real_A[2];
489:     WarpFragmentA warp_frag_imag_A[2];
490: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 491-493
```cpp
491:     WarpFragmentB warp_frag_real_B[2];
492:     WarpFragmentB warp_frag_imag_B[2];
493: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 494-496
```cpp
494:     this->warp_tile_iterator_A_.set_kgroup_index(0);
495:     this->warp_tile_iterator_B_.set_kgroup_index(0);
496: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 497-499
```cpp
497:     this->warp_tile_iterator_A_.load(warp_frag_real_A[0]);
498:     this->warp_tile_iterator_A_.load_with_pointer_offset(warp_frag_imag_A[0], Base::SharedStorage::kImaginaryStrideA);
499: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 500-502
```cpp
500:     this->warp_tile_iterator_B_.load(warp_frag_real_B[0]);
501:     this->warp_tile_iterator_B_.load_with_pointer_offset(warp_frag_imag_B[0], Base::SharedStorage::kImaginaryStrideB);
502: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 503-505
```cpp
503:     ++this->warp_tile_iterator_A_;
504:     ++this->warp_tile_iterator_B_;
505: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 506-515
```cpp
506:     iterator_A_real.clear_mask(gemm_k_iterations == 0);
507:     iterator_A_imag.clear_mask(gemm_k_iterations == 0);
508:     iterator_B_real.clear_mask(gemm_k_iterations == 0);
509:     iterator_B_imag.clear_mask(gemm_k_iterations == 0);
510: 
511:     // Start issuing the first group of the next stage outside of the mainloop
512:     copy_tiles_and_advance(iterator_A_real, iterator_A_imag, iterator_B_real, iterator_B_imag);
513: 
514:     Operator warp_mma;
515: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 516-518
```cpp
516:     int smem_write_stage_idx = Base::kStages - 1;
517:     int smem_read_stage_idx = 0;
518: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 519-522
```cpp
519:     //
520:     // Mainloop
521:     //
522: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 523-530
```cpp
523:     CUTLASS_GEMM_LOOP
524:     for (; gemm_k_iterations > (-Base::kStages + 1);) {
525:       //
526:       // Loop over GEMM K dimension
527:       //
528: 
529:       // Computes a warp-level GEMM on data held in shared memory
530:       // Each "warp_mma_k" refers to a warp-level matrix multiply-accumulate
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 531-537
```cpp
531:       CUTLASS_PRAGMA_UNROLL
532:       for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations;
533:            ++warp_mma_k) {
534: 
535:         // Load warp-level tiles from shared memory, wrapping to k offset if
536:         // this is the last group as the case may be.
537: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 538-540
```cpp
538:         this->warp_tile_iterator_A_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations);
539:         this->warp_tile_iterator_B_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations);
540:         
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 541-543
```cpp
541:         this->warp_tile_iterator_A_.load(warp_frag_real_A[(warp_mma_k + 1) % 2]);
542:         this->warp_tile_iterator_A_.load_with_pointer_offset(warp_frag_imag_A[(warp_mma_k + 1) % 2], Base::SharedStorage::kImaginaryStrideA);
543:         
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 544-546
```cpp
544:         this->warp_tile_iterator_B_.load(warp_frag_real_B[(warp_mma_k + 1) % 2]);
545:         this->warp_tile_iterator_B_.load_with_pointer_offset(warp_frag_imag_B[(warp_mma_k + 1) % 2], Base::SharedStorage::kImaginaryStrideB);
546: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 547-552
```cpp
547:         ++this->warp_tile_iterator_A_;
548:         ++this->warp_tile_iterator_B_;
549: 
550:         // Issue global->shared copies for the next stage
551:         int group_start_iteration_A, group_start_iteration_B;
552: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 553-556
```cpp
553:         if (warp_mma_k + 1 == Base::kWarpGemmIterations) {
554:           group_start_iteration_A = 0;
555:           group_start_iteration_B = 0;
556:         }
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 557-561
```cpp
557:         else {
558:           group_start_iteration_A = (warp_mma_k + 1) * Detail::kAccessesPerGroupA;
559:           group_start_iteration_B = (warp_mma_k + 1) * Detail::kAccessesPerGroupB;
560:         }
561:     
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 562-571
```cpp
562:         copy_tiles_and_advance(
563:           iterator_A_real, 
564:           iterator_A_imag,
565:           iterator_B_real, 
566:           iterator_B_imag,
567:           group_start_iteration_A, 
568:           group_start_iteration_B);
569: 
570:         if (warp_mma_k + 2 == Base::kWarpGemmIterations) {
571:           // Inserts a memory fence between stages of cp.async instructions
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 572-574
```cpp
572:           cutlass::arch::cp_async_fence();
573: 
574:           // Blocks until all but kStages-2 cp.async stages have committed.
```
**EN:** This block focuses on cp.async related implementation details.
**CN:** 该代码块聚焦于 cp.async 异步拷贝 的实现细节。

### Lines 575-578
```cpp
575:           arch::cp_async_wait<Base::kStages - 2>();
576:           __syncthreads();
577: 
578:           // Move to the next stage
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 579-581
```cpp
579:           iterator_A_real.add_tile_offset({0, 1});
580:           iterator_A_imag.add_tile_offset({0, 1});
581:           
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 582-584
```cpp
582:           iterator_B_real.add_tile_offset({1, 0});
583:           iterator_B_imag.add_tile_offset({1, 0});
584: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 585-589
```cpp
585:           this->smem_iterator_A_.add_tile_offset({0, 1});
586:           this->smem_iterator_B_.add_tile_offset({1, 0});
587: 
588:           // Add negative offsets to return iterators to the 'start' of the
589:           // circular buffer in shared memory
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 590-599
```cpp
590:           if (smem_write_stage_idx == (Base::kStages - 1)) {
591:             this->smem_iterator_A_.add_tile_offset({0, -Base::kStages});
592:             this->smem_iterator_B_.add_tile_offset({-Base::kStages, 0});
593:             smem_write_stage_idx = 0;
594:           } else {
595:             ++smem_write_stage_idx;
596:           }
597: 
598:           if (smem_read_stage_idx == (Base::kStages - 1)) {
599: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 600-603
```cpp
600:             this->warp_tile_iterator_A_.add_tile_offset(
601:                 {0, -Base::kStages * Policy::kPartitionsK *
602:                         Base::kWarpGemmIterations});
603: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 604-612
```cpp
604:             this->warp_tile_iterator_B_.add_tile_offset(
605:                 {-Base::kStages * Policy::kPartitionsK *
606:                      Base::kWarpGemmIterations,
607:                  0});
608:             smem_read_stage_idx = 0;
609:           } else {
610:             ++smem_read_stage_idx;
611:           }
612: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 613-619
```cpp
613:           --gemm_k_iterations;
614:           iterator_A_real.clear_mask(gemm_k_iterations == 0);
615:           iterator_A_imag.clear_mask(gemm_k_iterations == 0);
616:           iterator_B_real.clear_mask(gemm_k_iterations == 0);
617:           iterator_B_imag.clear_mask(gemm_k_iterations == 0);
618:         }
619: 
```
**EN:** This block focuses on iterator related implementation details.
**CN:** 该代码块聚焦于 迭代器逻辑 的实现细节。

### Lines 620-629
```cpp
620:         warp_mma_planar_complex(
621:           warp_mma, 
622:           accum, 
623:           warp_frag_real_A[warp_mma_k % 2], 
624:           warp_frag_imag_A[warp_mma_k % 2],
625:           warp_frag_real_B[warp_mma_k % 2], 
626:           warp_frag_imag_B[warp_mma_k % 2]);
627:       }
628: 
629:     }
```
**EN:** This block focuses on complex related implementation details.
**CN:** 该代码块聚焦于 复数处理 的实现细节。

### Lines 630-632
```cpp
630: 
631: 
632:     // Commit and drain all pending and predicated cp.async pnz from the GEMM mainloop
```
**EN:** This block focuses on cp.async related implementation details.
**CN:** 该代码块聚焦于 cp.async 异步拷贝 的实现细节。

### Lines 633-641
```cpp
633:     cutlass::arch::cp_async_fence();
634:     cutlass::arch::cp_async_wait<0>();
635:     __syncthreads();
636: 
637:   }
638: };
639: 
640: /////////////////////////////////////////////////////////////////////////////////////////////////
641: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 642-646
```cpp
642: }  // namespace threadblock
643: }  // namespace gemm
644: }  // namespace cutlass
645: 
646: /////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Threadblock-level tiling and shared memory orchestration  
  **CN:** 线程块级分块与共享内存编排
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** SIMT execution policy  
  **CN:** SIMT 执行策略
- **EN:** Asynchronous shared-memory staging  
  **CN:** 异步共享内存预取
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线
- **EN:** Multi-stage mainloop buffering  
  **CN:** 多阶段主循环缓冲

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `MmaPlanarComplexMultistage`, `using`, `Detail`, `copy_tiles_and_advance`, `warp_mma_planar_complex`, `operator`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
