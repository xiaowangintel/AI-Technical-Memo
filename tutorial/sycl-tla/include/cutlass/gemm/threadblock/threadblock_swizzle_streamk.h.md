# threadblock_swizzle_streamk.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/threadblock/threadblock_swizzle_streamk.h`
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
32:     \brief Implements streamk threadblock mapping blockIdx to GEMM problems.
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

### Lines 37-45
```cpp
37: #include "cutlass/cutlass.h"
38: #include "cutlass/fast_math.h"
39: #include "cutlass/layout/matrix.h"
40: #include "cutlass/platform/platform.h"
41: #include "cutlass/gemm/gemm_enumerated_types.h"
42: #include "cutlass/conv/conv2d_problem_size.h"
43: #include "cutlass/conv/conv3d_problem_size.h"
44: #include "cutlass/gemm/threadblock/index_remat.h"
45: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, layout types, threadblock components.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、布局类型、线程块组件。

### Lines 46-55
```cpp
46: #if !defined(__CUDACC_RTC__)
47: #include <iostream>
48: #include "cutlass/core_io.h"
49: #include "cutlass/trace.h"
50: #endif
51: 
52: 
53: /////////////////////////////////////////////////////////////////////////////////////////////////
54: 
55: namespace cutlass {
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具。

### Lines 56-66
```cpp
56: namespace gemm {
57: namespace threadblock {
58: 
59: /////////////////////////////////////////////////////////////////////////////////////////////////
60: 
61: /// Threadblock mapping control for GEMMs
62: struct ThreadblockSwizzleStreamK {
63: 
64:   /// Advertise StreamkFeature
65:   using StreamkFeature = void;
66: 
```
**EN:** Enters namespace scope (gemm::threadblock) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（gemm::threadblock），组织 GEMM 抽象层。

### Lines 67-73
```cpp
67: 
68:   /// Kernel traits
69:   template <typename GemmKernel>
70:   struct KernelTraits {};
71: 
72: 
73:   /// Reduction strategy
```
**EN:** Declares template parameters and begins the definition of KernelTraits.
**CN:** 声明模板参数并开始定义 KernelTraits。

### Lines 74-79
```cpp
74:   enum ReductionStrategy
75:   {
76:     kNone,      // Data-parallel strategy (no seams, fixup, etc.)
77: 
78:     kAtomic,    // Non-deterministic reduction of SK-block partials using atomic aggregation in L2
79: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 80-89
```cpp
80:     kMixed,     // Deterministic reduction of SK-block partials employing either:
81:                 //   (a) A separate wave of reduction thread blocks" (for scenarios with lots of
82:                 //       SK-blocks per SK-tile)
83:                 //   (b) Turnstile-ordered atomic aggregation in L2 (for scenarios with few
84:                 //       SK-blocks per SK-tile)
85:   };
86: 
87:   static ReductionStrategy const kReductionStrategy = kMixed;
88: 
89: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 90-99
```cpp
90:   //
91:   // Heuristics
92:   //
93: 
94:   /// Data-parallel wave-quantization efficiency threshold (above which we go data-parallel)
95:   static float constexpr kDpEfficiencyThreshold = 0.92f;
96: 
97:   /// Minimum number of MAC-iterations per streamk block
98:   static int const kMinItersPerSkBlock = 2;
99: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 100-109
```cpp
100:   /// Height in CTAs of a grid rasterization cohort
101:   static int const kCohortCtasM = 8;
102: 
103:   /// Width in CTAs of a grid rasterization cohort
104:   static int const kCohortCtasN = 4;
105: 
106:   /// Number of CTAs per cohort
107:   static int const kCtasPerCohort = kCohortCtasN * kCohortCtasM;
108: 
109:   /// Cost-equivalent number of SM-iterations for fixup I/O
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 110-113
```cpp
110:   static int const kFixupStartupIterEquiv = 10;
111:   static int const kFixupPeerIterEquiv = 3;
112: 
113: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 114-122
```cpp
114:   //
115:   // Member state
116:   //
117: 
118: 
119:   /// The 3D value-extents of the GEMM computation volume (m,n,k)
120:   GemmCoord problem_size;
121: 
122:   /// Div/mod accelerators
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 123-133
```cpp
123:   FastDivmod div_mod_tiled_shape_m;
124:   FastDivmod div_mod_tiled_shape_n;
125:   FastDivmod div_mod_tiled_cohort_shape_n;
126:   FastDivmod div_mod_iters_per_tile;
127: 
128:   /// Whether to perform cohort CTA rasterization
129:   bool cohort_raster;
130: 
131:   // Whether to pad and remap block indices
132:   bool remap_block_indices;
133: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 134-139
```cpp
134:   /// CTA occupancy per SM
135:   int sm_occupancy;
136: 
137:   /// Number of SMs for dispatch heuristics to load-balance using Stream-K CTAs (wave size)
138:   int avail_sms;
139: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 140-145
```cpp
140:   int dp_blocks;                            /// Number of data-parallel thread blocks in the grid
141:   int dp_first_wave_tiles;                  /// Number of output tiles each CTA in the first DP wave will produce
142: 
143:   /// Number of reduction blocks in the grid
144:   int reduction_blocks;
145: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 146-151
```cpp
146:   int sk_waves;
147:   int sk_tiles;
148:   int sk_big_blocks_per_region;
149:   int sk_iters_per_region;
150: 
151:   /// Div/mod accelerators
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 152-161
```cpp
152:   FastDivmod div_mod_sk_iters_per_normal_block;
153:   FastDivmod div_mod_sk_iters_per_big_block;
154:   FastDivmod div_mod_sk_iters_per_region;
155:   FastDivmod div_mod_sk_regions;                      //!! used in block map
156:   FastDivmod div_mod_sk_blocks_per_region;            //!! used in block map
157: 
158:   /// The batch count
159:   int batch_count;
160: 
161: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 162-169
```cpp
162:   //
163:   // Host+device interface
164:   //
165: 
166:   /// Constructor
167:   ThreadblockSwizzleStreamK() = default;
168: 
169:   /// Returns the GEMM volume in thread block tiles
```
**EN:** This block focuses on swizzle related implementation details.
**CN:** 该代码块聚焦于 swizzle 映射 的实现细节。

### Lines 170-179
```cpp
170:   CUTLASS_HOST_DEVICE
171:   GemmCoord tiled_shape() const
172:   {
173:     return GemmCoord(
174:         static_cast<int>(div_mod_tiled_shape_m),
175:         static_cast<int>(div_mod_tiled_shape_n),
176:         batch_count);
177:   }
178: 
179:   /// Number of iterations per output tile
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 180-186
```cpp
180:   CUTLASS_HOST_DEVICE
181:   int iters_per_tile() const
182:   {
183:     return static_cast<int>(div_mod_iters_per_tile);
184:   }
185: 
186:   /// Number of iterations for normal SK-blocks
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 187-193
```cpp
187:   CUTLASS_HOST_DEVICE
188:   int sk_iters_per_normal_block() const
189:   {
190:     return static_cast<int>(div_mod_sk_iters_per_normal_block);
191:   }
192: 
193:   /// Number of SK regions
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 194-200
```cpp
194:   CUTLASS_HOST_DEVICE
195:   int sk_regions() const
196:   {
197:     return static_cast<int>(div_mod_sk_regions);
198:   }
199: 
200:   /// Number of SK blocks per region (splitting factor)
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 201-207
```cpp
201:   CUTLASS_HOST_DEVICE
202:   int sk_blocks_per_region() const
203:   {
204:     return static_cast<int>(div_mod_sk_blocks_per_region);
205:   }
206: 
207: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 208-212
```cpp
208:   //
209:   // Host-side interface
210:   //
211: 
212:   /// Debug print
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 213-224
```cpp
213:   void Print()
214:   {
215: #ifndef __CUDA_ARCH__
216:     auto tiles = tiled_shape().mn().product();
217:     std::cout <<
218:         "problem_size: (" << problem_size.m() << "," << problem_size.n() << ")" <<
219:         ", tiled_shape: (" << tiled_shape().m() << "," << tiled_shape().n() << ")" <<
220:         ", tiles: " << tiles <<
221:         ", dp_tiles: " << tiles - sk_tiles <<
222:         ", sk_tiles: " << sk_tiles <<
223:         ", iters_per_tile: " << iters_per_tile() <<
224:         ", reduction_blocks: " << reduction_blocks <<
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 225-236
```cpp
225:         ", dp_blocks: " << dp_blocks <<
226:         ", dp_waves: " << dp_blocks / avail_sms <<
227:         ", dp_first_wave_tiles: " << dp_first_wave_tiles <<
228:         ", sk_blocks_per_region: " << sk_blocks_per_region() <<
229:         ", sk_regions: " << sk_regions() <<
230:         ", sk_waves: " << sk_waves <<
231:         ", sk_iters_per_normal_block: " << sk_iters_per_normal_block() <<
232:         ", sk_big_blocks_per_region: " << sk_big_blocks_per_region <<
233:         ", remap_block_indices: " << remap_block_indices <<
234:         ", cohort_raster: " << cohort_raster <<
235:         ", sm_occupancy: " << sm_occupancy <<
236:         ", avail_sms: " << avail_sms <<
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 237-243
```cpp
237:         ", num_blocks: " << get_num_blocks() <<
238:         "\n\n";
239: #endif
240:   }
241: 
242: 
243:   // Compute sk_blocks to dispatch for a given number of sk_tiles
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 244-252
```cpp
244:   static void get_sk_blocks(
245:     int &sk_blocks,     /// [out]
246:     int &savings_iters, /// [out]
247:     int sk_tiles,
248:     int iters_per_tile,
249:     int avail_sms,
250:     int max_sk_occupancy,
251:     bool allow_partial_wave)
252:   {
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 253-255
```cpp
253:     savings_iters = INT_MIN;
254:     sk_blocks = 0;
255: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 256-261
```cpp
256:     if (sk_tiles == 0) {
257:       return;
258:     }
259: 
260:     int sk_iters = sk_tiles * iters_per_tile;
261: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 262-264
```cpp
262:     int dp_equiv_waves = (sk_tiles + avail_sms - 1) / avail_sms;
263:     int dp_equiv_iters = iters_per_tile * dp_equiv_waves;
264: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 265-267
```cpp
265:     int min_sk_blocks = (allow_partial_wave) ? fast_min(avail_sms, sk_tiles + 1) : avail_sms;
266:     int max_sk_blocks = fast_min(avail_sms * max_sk_occupancy, sk_iters / kMinItersPerSkBlock);
267: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 268-277
```cpp
268:     for (int trial_sk_blocks = min_sk_blocks; trial_sk_blocks <= max_sk_blocks; ++trial_sk_blocks)
269:     {
270:       int sk_waves = (trial_sk_blocks + avail_sms - 1) / avail_sms;
271:       int max_sk_iters_per_block = (sk_iters + trial_sk_blocks - 1) / trial_sk_blocks;
272:       int sk_iter_equiv = max_sk_iters_per_block * sk_waves;
273: 
274:       int num_peers = ((trial_sk_blocks + sk_tiles - 1) / sk_tiles) + 1;        // add one for alignment skew
275: 
276:       float iter_cost = 0.02f * float(num_peers) * float(sk_iter_equiv);
277: 
```
**EN:** Runs an explicit loop, typically unrolled, to transform fragments or iterate across tiles.
**CN:** 执行显式循环（通常可展开），以变换片段或遍历 tile。

### Lines 278-287
```cpp
278:       if (trial_sk_blocks % sk_tiles == 0)
279:       {
280:         // aligned
281:         num_peers = (trial_sk_blocks / sk_tiles);
282: 
283:         iter_cost = 0.0f;
284:       }
285: 
286:       float peer_cost = 2.0f * float(num_peers);
287: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 288-293
```cpp
288:       float base_cost = 2.0f * float(sk_waves);
289: 
290:       int fixup_iter_equiv = int(base_cost + iter_cost + peer_cost);
291: 
292:       int trial_savings_iters = dp_equiv_iters - sk_iter_equiv - fixup_iter_equiv;
293: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 294-302
```cpp
294:       if (trial_savings_iters >= savings_iters) {
295:           savings_iters = trial_savings_iters;
296:           sk_blocks = trial_sk_blocks;
297:       }
298:     }
299:   }
300: 
301: 
302:   /// Determine the populations of DP and SK blocks to invoke for the given number of output tiles
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 303-310
```cpp
303:   static void get_blocks(
304:     int &dp_tiles,      /// [out]
305:     int &sk_blocks,     /// [out]
306:     int output_tiles,
307:     int iters_per_tile,
308:     int avail_sms,
309:     int sm_occupancy)
310:   {
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 311-314
```cpp
311:     int full_waves = output_tiles / avail_sms;
312:     int full_wave_tiles = full_waves * avail_sms;
313:     int partial_wave_tiles = output_tiles - full_wave_tiles;
314: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 315-318
```cpp
315:     int score = -1;
316:     dp_tiles = output_tiles;
317:     sk_blocks = 0;
318: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 319-324
```cpp
319:     if (partial_wave_tiles == 0)
320:     {
321:       // Perfect quantization
322:       return;
323:     }
324: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 325-333
```cpp
325:     if (full_waves < sm_occupancy)
326:     {
327:         // We're less than full GPU occupancy
328: 
329:         // Form the SK wave from the partial wave to get us up to full GPU occupancy
330:         int max_sk_occupancy = sm_occupancy - full_waves;
331: 
332:         dp_tiles = full_wave_tiles;
333: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 334-344
```cpp
334:         get_sk_blocks(
335:           sk_blocks,
336:           score,
337:           partial_wave_tiles,
338:           iters_per_tile,
339:           avail_sms,
340:           max_sk_occupancy,
341:           true);                 // we can run with less than a full wave of SK-blocks
342: 
343:         if (score < 0) {
344:           // not profitable
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 345-353
```cpp
345:           sk_blocks = 0;
346:           dp_tiles = output_tiles;
347:         }
348: 
349:         return;
350:     }
351: 
352:     // We're at (or greater) than GPU occupancy
353: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 354-361
```cpp
354:     if ((sm_occupancy > 1 ) && (full_waves % sm_occupancy == sm_occupancy - 1))
355:     {
356:         // If occupancy is more than one CTA per SM, form the SK wave from the partial
357:         // wave to get us to full GPU occupancy
358:         int max_sk_occupancy = 1;
359: 
360:         dp_tiles = full_wave_tiles;
361: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 362-370
```cpp
362:         get_sk_blocks(
363:           sk_blocks,
364:           score,
365:           partial_wave_tiles,
366:           iters_per_tile,
367:           avail_sms,
368:           max_sk_occupancy,
369:           true);                 // we can run with less than a full wave of SK-blocks
370: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 371-381
```cpp
371:         if (score >= 0) {
372:             return;
373:         }
374:     }
375: 
376:     // Form the SK wave by combining the last full wave and the partial wave
377:     // We're less than full GPU occupancy
378:     dp_tiles = full_wave_tiles - avail_sms;
379: 
380:     int max_sk_occupancy = sm_occupancy - ((full_waves - 1) % sm_occupancy);
381: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 382-392
```cpp
382:     get_sk_blocks(
383:       sk_blocks,
384:       score,
385:       partial_wave_tiles + avail_sms,
386:       iters_per_tile,
387:       avail_sms,
388:       max_sk_occupancy,
389:       false);                 // we cannot run with less than a full wave of SK-blocks
390: 
391:     if (score < 0) {
392:       // not profitable
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 393-399
```cpp
393:       sk_blocks = 0;
394:       dp_tiles = output_tiles;
395:     }
396: 
397:   }
398: 
399:   /// Constructor: *Gemm* problem size (m, n, k)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 400-411
```cpp
400:   ThreadblockSwizzleStreamK(
401:     GemmUniversalMode const mode_,
402:     GemmCoord const problem_size_,
403:     GemmCoord const tile_size_,
404:     int const batch_split_,                        /// Either (mode == GemmUniversalMode::kBatched) the batch count, or (mode == GemmUniversalMode::kGemm) the tile-splitting factor (1 defaults to StreamK, >1 emulates Split-K)
405:     int const sm_occupancy_,
406:     int const device_sms_,
407:     int const avail_sms_,                          /// The number of SMs that StreamK dispatch heuristics will attempt to load-balance across (-1 defaults to device width, 1 implies classic data-parallel scheduling)
408:     size_t const element_A_bytes_,
409:     size_t const element_B_bytes_,
410:     size_t const element_C_bytes_,
411:     int const epilogue_acc_fragments_)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 412-423
```cpp
412:   :
413:     problem_size(problem_size_),
414:     batch_count((mode_ == GemmUniversalMode::kBatched || mode_ == GemmUniversalMode::kArray) ? batch_split_ : 1),
415:     reduction_blocks(0),
416:     dp_blocks(0),
417:     dp_first_wave_tiles(1),     // Default: one tile per DP-block in the first wave of DP blocks
418:     sk_tiles(0),
419:     sk_big_blocks_per_region(0),
420:     sk_iters_per_region(0),
421:     sk_waves(0),
422:     sm_occupancy(sm_occupancy_),
423:     remap_block_indices(false),
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 424-430
```cpp
424:     avail_sms(fast_max(1, avail_sms_)),
425:     cohort_raster(false)
426:   {
427:     int gpu_occupancy = device_sms_ * sm_occupancy;
428:     int iters_per_tile = (problem_size.k() + tile_size_.k() - 1) / tile_size_.k();
429:     int sk_iters_per_normal_block = 0;
430: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 431-433
```cpp
431:     int sk_regions = 1;              // Default: a single region of iteration space (across all SK tiles)
432:     int sk_blocks_per_region = 0;
433: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 434-438
```cpp
434:     GemmCoord tiled_shape(
435:       (problem_size.m() + tile_size_.m() - 1) / tile_size_.m(),
436:       (problem_size.n() + tile_size_.n() - 1) / tile_size_.n(),
437:       batch_count);
438: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 439-447
```cpp
439:     size_t problem_bytes =
440:               (element_C_bytes_ * problem_size.m() * problem_size.n()) +
441:               (element_A_bytes_ * problem_size.m() * problem_size.k()) +
442:               (element_B_bytes_ * problem_size.k() * problem_size.n());
443: 
444:     size_t problem_flops = size_t(problem_size.m()) * size_t(problem_size.n()) * size_t(problem_size.k()) * 2;
445: 
446:     [[maybe_unused]] float flops_per_byte = float(problem_flops) / float(problem_bytes);
447: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 448-451
```cpp
448:     int output_tiles = tiled_shape.m() * tiled_shape.n();
449:     int waves = (output_tiles + avail_sms - 1) / avail_sms;
450:     [[maybe_unused]] float dp_efficiency = float(output_tiles) / float(waves * avail_sms);
451: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 452-456
```cpp
452:     //
453:     // Determine dispatch composition of DP-tiles and SK-blocks
454:     //
455: 
456:     // Start with a DP-only configuration
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 457-460
```cpp
457:     int dp_tiles = output_tiles;    // Number of data-parallel tiles
458:     int sk_blocks = 0;              // Number of thread blocks to produce the remaining SK tiles
459: 
460:     // Only kGemm mode allows for SK load balancing
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 461-466
```cpp
461:     if (mode_ == GemmUniversalMode::kGemm)
462:     {
463:       int split_factor = batch_split_;
464:       if (split_factor > 1)
465:       {
466:         // Split-K override
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 467-469
```cpp
467:         dp_tiles = 0;
468:         sk_blocks = output_tiles * split_factor;
469:       }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 470-473
```cpp
470:       else if ((kReductionStrategy != kNone) &&   // Load-balancing strategy statically enabled
471:         (avail_sms > 1))                         // Plurality of SMs to load balance across
472:       {
473:         // Use heuristics
```
**EN:** Applies conditional logic to select architecture paths, handle bounds, or guard special cases.
**CN:** 应用条件逻辑以选择架构路径、处理边界或保护特殊情况。

### Lines 474-483
```cpp
474:         get_blocks(
475:           dp_tiles,      /// [out]
476:           sk_blocks,     /// [out]
477:           output_tiles,
478:           iters_per_tile,
479:           avail_sms,
480:           sm_occupancy);
481:       }
482:     }
483: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 484-487
```cpp
484:     sk_tiles = output_tiles - dp_tiles;
485: 
486: 
487:     // Compute SK block iteration details
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 488-491
```cpp
488:     if (sk_blocks > 0)
489:     {
490:       sk_waves = (sk_blocks + avail_sms - 1) / avail_sms;
491: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 492-494
```cpp
492:       int sk_iters = sk_tiles * iters_per_tile;
493:       sk_blocks = fast_min(sk_blocks, sk_iters);
494: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 495-498
```cpp
495:       sk_iters_per_normal_block = sk_iters / sk_blocks;
496:       int extra_sk_iters = sk_iters - (sk_iters_per_normal_block * sk_blocks);
497:       int sk_big_blocks = extra_sk_iters;
498: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 499-504
```cpp
499:       if ((sk_blocks > sk_tiles) && (sk_blocks % sk_tiles == 0))
500:       {
501:         // Split-K decomposition
502:         sk_regions = sk_tiles;
503:       }
504: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 505-508
```cpp
505:       sk_blocks_per_region = sk_blocks / sk_regions;
506:       sk_big_blocks_per_region = sk_big_blocks / sk_regions;
507:       sk_iters_per_region = sk_iters / sk_regions;
508: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 509-516
```cpp
509:       // Use a separate reduction wave when all of:
510:       // - Non-atomic reduction stratgy
511:       // - The number of SK waves won't fully occupy the GPU (Otherwise we don't have
512:       //   a strong-scaling case for more parallel reduction)
513:       // - More than three peers working on an SK tile.  (This occurs when the ratio of
514:       //   SK-blocks to SK-tiles > 2, as a single tile may be covered by four SK-blocks,
515:       //   e.g.:[partial-block | block | block | partial-block] ).  With three or
516:       //   less peers, the two non-finishing SK-blocks are not expected to contend.
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 517-525
```cpp
517:       if ((kReductionStrategy == kMixed) &&
518:           (sk_waves < sm_occupancy) &&
519:           (sk_blocks > 2 * sk_tiles))
520:       {
521:         // Launch a reduction block for every accumulator fragment in each SK-tile
522:         reduction_blocks = sk_tiles * epilogue_acc_fragments_;
523: 
524:       }
525: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 526-529
```cpp
526:       // When we have a multi-occupancy kernel and at least two waves of active blocks (where
527:       // at least one wave is SK blocks), we need to (1) dispatch at least four waves, and (2)
528:       // remap the block indices so that we can reliably spread the SK blocks evenly across the
529:       // device's first SM occupancy valence. Also see get_num_blocks() and get_block_idx().
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 530-535
```cpp
530:       remap_block_indices = (
531:           (sm_occupancy > 1) &&
532:           (device_sms_ == avail_sms) &&
533:           (get_num_active_blocks() > avail_sms * 2));
534: 
535:       // Initialize fast div/mod members related to SK
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 536-542
```cpp
536:       div_mod_sk_iters_per_normal_block = FastDivmod(sk_iters_per_normal_block);
537:       div_mod_sk_iters_per_big_block = FastDivmod(sk_iters_per_normal_block + 1);
538:       div_mod_sk_iters_per_region = FastDivmod(sk_iters_per_region);
539:       div_mod_sk_regions = FastDivmod(sk_regions);
540:       div_mod_sk_blocks_per_region = FastDivmod(sk_blocks_per_region);
541:     }
542: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 543-548
```cpp
543:     //
544:     // Compute DP blocks
545:     //
546: 
547:     dp_blocks = dp_tiles;
548: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 549-556
```cpp
549:     cutlass::gemm::GemmCoord tiled_cohort_shape(
550:         (tiled_shape.m() + kCohortCtasM - 1) / kCohortCtasM,
551:         (tiled_shape.n() + kCohortCtasN - 1) / kCohortCtasN,
552:         tiled_shape.k());
553:     int cohort_blocks = (tiled_cohort_shape.m() * tiled_cohort_shape.n()) * kCtasPerCohort;
554:     float cohort_efficiency = float(dp_blocks) / float(cohort_blocks);
555: 
556:     // Check if the SK tiles would be in cohorts that are in-bounds
```
**EN:** Applies conditional logic to select architecture paths, handle bounds, or guard special cases.
**CN:** 应用条件逻辑以选择架构路径、处理边界或保护特殊情况。

### Lines 557-566
```cpp
557:     bool sk_in_range = true;
558:     if (sk_tiles > 0)
559:     {
560:       int last_sk_tile = sk_tiles - 1;
561:       int cohort_tile_idx = last_sk_tile / kCtasPerCohort;
562:       int cohort_grid_m = cohort_tile_idx / tiled_cohort_shape.n();
563:       int cohort_grid_n = (cohort_grid_m > 0) ?
564:         tiled_cohort_shape.n() - 1 :
565:         cohort_tile_idx % tiled_cohort_shape.n();
566: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 567-575
```cpp
567:       if ((((cohort_grid_m + 1) * kCohortCtasM) >= tiled_shape.m()) ||
568:           (((cohort_grid_n + 1) * kCohortCtasN) >= tiled_shape.n()))
569:       {
570:         sk_in_range = false;
571:       }
572: 
573:     }
574: 
575:     // Decide if we're going to be doing cohort raster
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 576-579
```cpp
576:     if (sk_in_range &&
577:         (dp_blocks >= gpu_occupancy * 2) &&
578:         (cohort_efficiency > 0.85f))
579:     {
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 580-582
```cpp
580:       cohort_raster = true;
581:       dp_blocks = cohort_blocks;
582:     }
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 583-586
```cpp
583:     else if (sk_waves > 0)
584:     {
585:       // Update semi-persistence of first DP wave to ensure full grid wavesets
586:       // (Only applies when there's an SK component and we're not doing blocked cohort rasterization)
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 587-590
```cpp
587:       int dp_tile_waves = (dp_tiles + avail_sms - 1) / avail_sms;
588:       int full_dp_tile_waves = dp_tiles / avail_sms;
589:       int waveset_excess = (sk_waves + dp_tile_waves) % sm_occupancy;
590: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 591-598
```cpp
591:       if (dp_first_wave_tiles + waveset_excess <= full_dp_tile_waves)
592:       {
593:         dp_first_wave_tiles += waveset_excess;
594:         dp_blocks -= (waveset_excess * avail_sms);
595:       }
596:     }
597: 
598:     // Setup fast-div/mod for device-side usage
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 599-606
```cpp
599:     div_mod_tiled_shape_m = FastDivmod(tiled_shape.m());
600:     div_mod_tiled_shape_n = FastDivmod(tiled_shape.n());
601:     div_mod_tiled_cohort_shape_n = FastDivmod(tiled_cohort_shape.n());
602:     div_mod_iters_per_tile = FastDivmod(iters_per_tile);
603: 
604:   }
605: 
606:   /// Number of blocks performing useful work
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 607-612
```cpp
607:   int get_num_active_blocks() const
608:   {
609:     return (sk_waves * avail_sms) + dp_blocks + reduction_blocks;
610:   }
611: 
612:   /// Obtains number of threadblocks per GEMM
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 613-623
```cpp
613:   int get_num_blocks() const
614:   {
615:     int active_blocks = get_num_active_blocks();
616:     if (remap_block_indices)
617:     {
618:       // Add padding blocks if we are performing remapping in order to dispatch a grid of at least four waves
619:       return fast_max(active_blocks, avail_sms * 4);
620:     }
621: 
622:     return active_blocks;
623:   }
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 624-626
```cpp
624: 
625: 
626:   /// Obtains grid extents in CTAs
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 627-632
```cpp
627:   dim3 get_grid_dims() const
628:   {
629:     return dim3(get_num_blocks(), 1, batch_count);
630:   }
631: 
632: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 633-637
```cpp
633:   //
634:   // Device-side interface
635:   //
636: 
637:   /// Obtains number of threadblocks per GEMM
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 638-644
```cpp
638:   CUTLASS_DEVICE
639:   int device_num_blocks() const
640:   {
641:     return GridDimX();
642:   }
643: 
644:   /// Obtains tile index for the given sk iteration
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 645-652
```cpp
645:   CUTLASS_DEVICE
646:   int get_sk_tile_idx(int iter) const
647:   {
648:     int tile_idx = div_mod_iters_per_tile.div(iter);
649:     return tile_idx;
650:   }
651: 
652:   /// Obtains the batch index
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 653-659
```cpp
653:   CUTLASS_DEVICE
654:   int get_batch_idx() const
655:   {
656:     return RematerializeBlockIdxZ();
657:   }
658: 
659:   /// Obtains the calling threadblock's tiled coordinates for the given tile index
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 660-667
```cpp
660:   CUTLASS_DEVICE
661:   GemmCoord get_tile_offset(int tile_idx) const
662:   {
663:     int m, n;
664: 
665:     // row-major raster
666:     div_mod_tiled_shape_n(m, n, tile_idx);
667: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 668-673
```cpp
668:     if (tiled_shape().m() < tiled_shape().n())
669:     {
670:       // column-major raster
671:       div_mod_tiled_shape_m(n, m, tile_idx);
672:     }
673: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 674-676
```cpp
674:     if (cohort_raster)
675:     {
676:       // tiled cohort raster
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 677-680
```cpp
677:       int cohort_tile_idx = tile_idx / kCtasPerCohort;
678:       int cohort_grid_m, cohort_grid_n;
679:       div_mod_tiled_cohort_shape_n(cohort_grid_m, cohort_grid_n, cohort_tile_idx);
680: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 681-684
```cpp
681:       int block_idx_cohort = tile_idx % kCtasPerCohort;
682:       int block_cohort_m = block_idx_cohort / kCohortCtasN;
683:       int block_cohort_n = block_idx_cohort % kCohortCtasN;
684: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 685-692
```cpp
685:       m = (cohort_grid_m * kCohortCtasM) + block_cohort_m;
686:       n = (cohort_grid_n * kCohortCtasN) + block_cohort_n;
687:     }
688: 
689:     return GemmCoord(m, n, get_batch_idx());
690:   }
691: 
692:   /// Obtains the calling threadblock's tiled coordinates for the given tile index (row-major rasterization)
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 693-696
```cpp
693:   CUTLASS_DEVICE
694:   GemmCoord get_tile_offset_row_major(int tile_idx) const
695:   {
696:     // row-major raster
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 697-702
```cpp
697:     int m, n;
698:     div_mod_tiled_shape_n(m, n, tile_idx);
699:     return GemmCoord(m, n, get_batch_idx());
700:   }
701: 
702:   /// Obtains calling threadblock's linear threadblock index
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 703-709
```cpp
703:   CUTLASS_DEVICE
704:   int get_block_idx() const
705:   {
706:     int block_idx = RematerializeBlockIdxX();
707: 
708:     // Remap the block indices for the first two waves of thread blocks if
709:     // we have multi-occupancy and the grid constitutes four or more waves
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 710-718
```cpp
710:     if (remap_block_indices && (block_idx < avail_sms * 2))
711:     {
712:       int dest_sm = block_idx / 2;
713:       int dest_wave = block_idx % 2;
714:       int remapped_block_idx = dest_sm + (dest_wave * avail_sms);
715:       block_idx = remapped_block_idx;
716:     }
717: 
718:     // Remap block indices to interleave SK regions to limit intra-region waiting
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 719-728
```cpp
719:     if (block_idx < sk_regions() * sk_blocks_per_region())
720:     {
721:       int block_in_region;
722:       int region;
723:       div_mod_sk_regions(block_in_region, region, block_idx);
724:       block_idx = (region * sk_blocks_per_region()) + block_in_region;
725:     }
726: 
727:     return block_idx;
728:   }
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 729-731
```cpp
729: 
730: 
731:   /// Obtains calling linear threadblock index of the first block to work on the given tile
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 732-738
```cpp
732:   CUTLASS_DEVICE
733:   int get_sk_block_idx(int iter) const
734:   {
735:     int region_idx;
736:     int iter_in_region;
737:     div_mod_sk_iters_per_region(region_idx, iter_in_region, iter);
738: 
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 739-741
```cpp
739:     int big_block_iters = (sk_big_blocks_per_region * sk_iters_per_normal_block()) + sk_big_blocks_per_region;   // number of iterations in the region's big blocks
740:     int normal_block_iters = iter_in_region - big_block_iters;                                                 // number of iterations in the region's normal blocks
741: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 742-744
```cpp
742:     int big_block_idx_in_region = div_mod_sk_iters_per_big_block.div(iter_in_region);
743:     int normal_block_idx_in_region = sk_big_blocks_per_region + div_mod_sk_iters_per_normal_block.div(normal_block_iters);
744: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 745-754
```cpp
745:     int block_idx_in_region = (big_block_idx_in_region < sk_big_blocks_per_region) ?
746:         big_block_idx_in_region :
747:         normal_block_idx_in_region;
748: 
749:     int owning_block_idx = (sk_blocks_per_region() * region_idx) + block_idx_in_region;
750: 
751:     return owning_block_idx;
752:   }
753: 
754:   /// Obtains iteration extends for the given SK block index
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 755-760
```cpp
755:   CUTLASS_DEVICE
756:   void get_iter_extents(
757:       int sk_block_idx,
758:       int &block_iter_begin,
759:       int &block_iter_end) const
760:   {
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 761-767
```cpp
761:     int region_idx;
762:     int block_idx_in_region;
763:     div_mod_sk_blocks_per_region(region_idx, block_idx_in_region, sk_block_idx);
764: 
765:     block_iter_begin = (region_idx * sk_iters_per_region) + (block_idx_in_region * sk_iters_per_normal_block());
766: 
767:     // Adjust extents for the first "num_big_blocks" blocks that get one extra iteration
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 768-770
```cpp
768:     int block_iters = sk_iters_per_normal_block();
769:     if (block_idx_in_region < sk_big_blocks_per_region) {
770:       // This is a +1 iteration block
```
**EN:** Defines an inline device/host routine that advances the pipeline, iterator, or math operation.
**CN:** 定义内联的设备/主机例程，用于推进流水线、迭代器或数学操作。

### Lines 771-780
```cpp
771:       block_iter_begin += block_idx_in_region;
772:       block_iters++;
773:     } else {
774:       // This is a regular block
775:       block_iter_begin += sk_big_blocks_per_region;
776:     }
777:     block_iter_end = block_iter_begin + block_iters;
778:   }
779: 
780: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 781-781
```cpp
781:   /// Obtains calling linear threadblock index of the first block to work on the given tile
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 782-789
```cpp
782:   CUTLASS_DEVICE
783:   int get_first_block_idx(int tile_idx, int block_idx) const
784:   {
785:     if (tile_idx >= sk_tiles) {
786:       // DP tile
787:       return block_idx;
788:     }
789: 
```
**EN:** Implements a small inline routine and returns computed fragments, iterators, or status values.
**CN:** 实现一个小型内联例程，并返回计算结果、迭代器或状态值。

### Lines 790-799
```cpp
790:     int iter = tile_idx * iters_per_tile();
791:     return get_sk_block_idx(iter);
792:   }
793: 
794: };
795: 
796: /////////////////////////////////////////////////////////////////////////////////////////////////
797: 
798: } // namespace threadblock
799: } // namespace gemm
```
**EN:** Returns the value produced by the helper logic in this block.
**CN:** 返回该代码块中辅助逻辑产生的结果。

### Lines 800-801
```cpp
800: } // namespace cutlass
801: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** Threadblock-level tiling and shared memory orchestration  
  **CN:** 线程块级分块与共享内存编排
- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** Epilogue/output transformation  
  **CN:** Epilogue/输出变换
- **EN:** Threadblock swizzle mapping  
  **CN:** 线程块 swizzle 映射
- **EN:** Split-K decomposition  
  **CN:** Split-K 拆分

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `ThreadblockSwizzleStreamK`, `KernelTraits`, `tiled_shape`, `iters_per_tile`, `sk_iters_per_normal_block`, `sk_regions`, `region`, `Print`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
