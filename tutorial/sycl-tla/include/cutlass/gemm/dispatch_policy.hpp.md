# dispatch_policy.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/dispatch_policy.hpp`
- **Purpose (EN):** Declares dispatch policies that connect GEMM kernels to architecture-specific execution strategies.
- **用途 (CN):** 声明将 GEMM 内核连接到架构相关执行策略的调度策略。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
31: #pragma once
32: 
33: #include "cutlass/arch/arch.h"
34: #include "cutlass/gemm/gemm.h"
```
**EN:** Uses a single-inclusion guard so template definitions are not parsed twice.
**CN:** 使用单次包含保护，避免模板定义被重复解析。

### Lines 35-35
```cpp
35: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 36-45
```cpp
36: #include "cute/layout.hpp"
37: #include "cute/numeric/integral_constant.hpp" // cute::false_type
38: #include "cute/atom/copy_traits_sm100.hpp"
39: #include "cutlass/detail/collective/sm103_kernel_type.hpp"
40: //////////////////////////////////////////////////////////////////////////////
41: 
42: namespace cutlass::detail {
43: 
44: template <class T, template <int...> class U>
45: struct is_kernel_tag_of : cute::false_type {};
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, numeric types/converters.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、数值类型/转换器。

### Lines 46-49
```cpp
46: 
47: template <template <int...> class U, int... Args>
48: struct is_kernel_tag_of<U<Args...>, U> : cute::true_type {};
49: 
```
**EN:** Declares template parameters and begins the definition of U.
**CN:** 声明模板参数并开始定义 U。

### Lines 50-58
```cpp
50: template <class T, template <int...> class U>
51: constexpr bool is_kernel_tag_of_v = is_kernel_tag_of<T, U>::value;
52: 
53: template <class T, template <int,bool> class U>
54: struct is_asymmetric_dma_kernel_tag_of : cute::false_type {};
55: 
56: template <template <int, bool> class U, int I0, bool B0>
57: struct is_asymmetric_dma_kernel_tag_of<U<I0, B0>, U> : cute::true_type {};
58: 
```
**EN:** Declares template parameters and begins the definition of T.
**CN:** 声明模板参数并开始定义 T。

### Lines 59-69
```cpp
59: template <class T, template <int, bool> class U>
60: constexpr bool is_asymmetric_dma_kernel_tag_of_v = \
61:                               is_asymmetric_dma_kernel_tag_of<T, U>::value;
62: 
63: }
64: 
65: //////////////////////////////////////////////////////////////////////////////
66: 
67: namespace cutlass::gemm {
68: using namespace cute;
69: 
```
**EN:** Enters namespace scope (cutlass) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（cutlass），组织 GEMM 抽象层。

### Lines 70-73
```cpp
70: //////////////////////////////////////////////////////////////////////////////
71: 
72: namespace detail {
73: 
```
**EN:** Enters namespace scope (detail) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（detail），组织 GEMM 抽象层。

### Lines 74-83
```cpp
74: enum class KernelInputTransformType {
75:     FastF32,
76:     InterleavedComplexTF32,
77:     MixedInput
78: };
79: 
80: } // namespace detail
81: 
82: //////////////////////////////////////////////////////////////////////////////
83: 
```
**EN:** Defines KernelInputTransformType, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelInputTransformType，用于封装策略、存储或算法行为的辅助类型。

### Lines 84-90
```cpp
84: namespace kernel::detail {
85: 
86: // Has_SwapAB<T>::value will be true only if:
87: //   class T has member SwapAB and T::SwapAB is true
88: template <typename T, typename = void>
89: struct Has_SwapAB { static constexpr bool value = false; };
90: 
```
**EN:** Enters namespace scope (kernel) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（kernel），组织 GEMM 抽象层。

### Lines 91-94
```cpp
91: template <typename T>
92: struct Has_SwapAB <T, CUTE_STL_NAMESPACE::void_t<decltype(T::SwapAB)>>
93: { static constexpr bool value = T::SwapAB; };
94: 
```
**EN:** Declares template parameters and begins the definition of Has_SwapAB.
**CN:** 声明模板参数并开始定义 Has_SwapAB。

### Lines 95-101
```cpp
95: template <typename T>
96: static constexpr bool Has_SwapAB_v = Has_SwapAB<T>::value;
97: 
98: // additional producer warp role check for block scaling mainloop
99: template<typename T>
100: struct HasAuxiliaryLoad : cute::false_type{};
101: 
```
**EN:** Declares template parameters and begins the definition of HasAuxiliaryLoad.
**CN:** 声明模板参数并开始定义 HasAuxiliaryLoad。

### Lines 102-108
```cpp
102: template <typename T>
103: static constexpr bool HasAuxiliaryLoad_v = HasAuxiliaryLoad<T>::value;
104: 
105: } // namespace kernel::detail
106: 
107: //////////////////////////////////////////////////////////////////////////////
108: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 109-118
```cpp
109: //
110: // Kernel schedule policies (the base class tags, one for each kernel layer file)
111: //
112: struct KernelMultistage { };
113: struct KernelPtrArrayMultistage { };
114: struct KernelCpAsyncWarpSpecialized { };
115: struct KernelCpAsyncWarpSpecializedPingpong { };
116: struct KernelCpAsyncWarpSpecializedCooperative { };
117: struct KernelTma { };
118: struct KernelTmaWarpSpecialized { };
```
**EN:** Defines tags, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 tags，用于封装策略、存储或算法行为的辅助类型。

### Lines 119-121
```cpp
119: struct KernelTmaWarpSpecializedPingpong { 
120:   static constexpr int SchedulerPipelineStageCount = 0;
121: };
```
**EN:** Defines KernelTmaWarpSpecializedPingpong, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedPingpong，用于封装策略、存储或算法行为的辅助类型。

### Lines 122-131
```cpp
122: struct KernelTmaWarpSpecializedCooperative { 
123:   static constexpr int SchedulerPipelineStageCount = 0;
124: };
125: 
126: struct KernelPtrArrayTmaWarpSpecializedCooperative { };
127: struct KernelPtrArrayTmaWarpSpecializedPingpong { };
128: 
129: // FP8 related policies (including Blocked Scaled Accumulation)
130: struct KernelTmaWarpSpecializedCooperativeFP8Blockwise: KernelTmaWarpSpecializedCooperative { };
131: struct KernelTmaWarpSpecializedPingpongFP8Blockwise: KernelTmaWarpSpecializedPingpong { };
```
**EN:** Defines KernelTmaWarpSpecializedCooperative, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedCooperative，用于封装策略、存储或算法行为的辅助类型。

### Lines 132-134
```cpp
132: struct KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise: KernelPtrArrayTmaWarpSpecializedCooperative { };
133: struct KernelPtrArrayTmaWarpSpecializedPingpongFP8Blockwise: KernelPtrArrayTmaWarpSpecializedPingpong { };
134: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise，用于封装策略、存储或算法行为的辅助类型。

### Lines 135-144
```cpp
135: using KernelTmaWarpSpecializedCooperativeFP8BlockScaledAccum = KernelTmaWarpSpecializedCooperativeFP8Blockwise;
136: using KernelTmaWarpSpecializedPingpongFP8BlockScaledAccum = KernelTmaWarpSpecializedPingpongFP8Blockwise;
137: using KernelPtrArrayTmaWarpSpecializedCooperativeFP8BlockScaledAccum = KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise;
138: using KernelPtrArrayTmaWarpSpecializedPingpongFP8BlockScaledAccum = KernelPtrArrayTmaWarpSpecializedPingpongFP8Blockwise;
139: 
140: // Policies to opt into mixed type GEMMs
141: struct KernelTmaWarpSpecializedMixedInput : KernelTmaWarpSpecialized { };
142: struct KernelTmaWarpSpecializedPingpongMixedInput : KernelTmaWarpSpecializedPingpong { };
143: struct KernelTmaWarpSpecializedCooperativeMixedInput: KernelTmaWarpSpecializedCooperative { };
144: 
```
**EN:** Defines KernelTmaWarpSpecializedMixedInput, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedMixedInput，用于封装策略、存储或算法行为的辅助类型。

### Lines 145-149
```cpp
145: struct KernelXe { };
146: struct KernelXeCooperative { };
147: struct KernelXePtrArrayCooperative { };
148: //////////////////////////////////////////////////////////////////////////////
149: 
```
**EN:** Defines KernelXe, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelXe，用于封装策略、存储或算法行为的辅助类型。

### Lines 150-159
```cpp
150: //
151: // Builder dispatch policies (not a part of the main CUTLASS layers, simply used to opt into
152: // specific collective builder dispatches)
153: //
154: 
155: // FP8 related policies (including Fast Accumulation)
156: struct KernelTmaWarpSpecializedFP8FastAccum : KernelTmaWarpSpecialized { };
157: struct KernelTmaWarpSpecializedPingpongFP8FastAccum : KernelTmaWarpSpecializedPingpong { };
158: struct KernelTmaWarpSpecializedCooperativeFP8FastAccum: KernelTmaWarpSpecializedCooperative { };
159: struct KernelPtrArrayTmaWarpSpecializedCooperativeFP8FastAccum : KernelPtrArrayTmaWarpSpecializedCooperative { };
```
**EN:** Defines KernelTmaWarpSpecializedFP8FastAccum, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedFP8FastAccum，用于封装策略、存储或算法行为的辅助类型。

### Lines 160-169
```cpp
160: struct KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum : KernelPtrArrayTmaWarpSpecializedPingpong { };
161: 
162: //////////////////////////////////////////////////////////////////////////////
163: 
164: // Policies for dispatch of epilogue
165: struct EpilogueDefault { };
166: struct EpilogueTransposed { };
167: 
168: //////////////////////////////////////////////////////////////////////////////
169: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum，用于封装策略、存储或算法行为的辅助类型。

### Lines 170-174
```cpp
170: //
171: // Collective Mainloop Policies
172: //
173: 
174: // 2 stage pipeline through 1 stage in smem, 1 in rmem, WITHOUT predicated gmem loads
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 175-182
```cpp
175: struct MainloopSm70TwoStageUnpredicated {
176:   constexpr static int Stages = 2;
177:   using ArchTag = arch::Sm70;
178:   using Schedule = KernelMultistage;
179:   using ClusterShape = Shape<_1,_1,_1>;
180: };
181: 
182: // 2 stage pipeline through 1 stage in smem, 1 in rmem, with predicated gmem loads
```
**EN:** Defines MainloopSm70TwoStageUnpredicated, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MainloopSm70TwoStageUnpredicated，用于封装策略、存储或算法行为的辅助类型。

### Lines 183-190
```cpp
183: struct MainloopSm70TwoStage {
184:   constexpr static int Stages = 2;
185:   using ArchTag = arch::Sm70;
186:   using Schedule = KernelMultistage;
187:   using ClusterShape = Shape<_1,_1,_1>;
188: };
189: 
190: // n-buffer in smem (cp.async), pipelined with registers, WITHOUT predicated gmem loads
```
**EN:** Defines MainloopSm70TwoStage, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MainloopSm70TwoStage，用于封装策略、存储或算法行为的辅助类型。

### Lines 191-199
```cpp
191: template<int Stages_>
192: struct MainloopSm80CpAsyncUnpredicated {
193:   constexpr static int Stages = Stages_;
194:   using ArchTag = arch::Sm80;
195:   using Schedule = KernelMultistage;
196:   using ClusterShape = Shape<_1,_1,_1>;
197: };
198: 
199: // n-buffer in smem (cp.async), pipelined with registers, with predicated gmem loads
```
**EN:** Defines MainloopSm80CpAsyncUnpredicated, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MainloopSm80CpAsyncUnpredicated，用于封装策略、存储或算法行为的辅助类型。

### Lines 200-204
```cpp
200: template<
201:   int Stages_,
202:   class ClusterShape_ = Shape<_1,_1,_1>
203: >
204: struct MainloopSm80CpAsync {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 205-211
```cpp
205:   constexpr static int Stages = Stages_;
206:   using ArchTag = cute::conditional_t<(size(ClusterShape_{}) > 1), arch::Sm90, arch::Sm80>;
207:   using Schedule = KernelMultistage;
208:   using ClusterShape = ClusterShape_;
209: };
210: 
211: // n-buffer in smem (cp.async), pipelined with registers, with predicated gmem loads for SM100 Simt Ptr-Array
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 212-215
```cpp
212: template<int Stages_,
213:   class ClusterShape_ = Shape<_1,_1,_1>
214: >
215: struct MainloopSm80ArrayCpAsync {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 216-222
```cpp
216:   constexpr static int Stages = Stages_;
217:   using ArchTag = cute::conditional_t<(size(ClusterShape_{}) > 1), arch::Sm90, arch::Sm80>;
218:   using Schedule = KernelPtrArrayMultistage;
219:   using ClusterShape = ClusterShape_;
220: };
221: 
222: // n-buffer in smem (cp.async), pipelined with Hopper GMMA, with predicated gmem loads, warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 223-228
```cpp
223: template<
224:   int Stages_,
225:   class ClusterShape_ = Shape<_1,_1,_1>,
226:   class KernelSchedule = KernelCpAsyncWarpSpecialized
227: >
228: struct MainloopSm90CpAsyncGmmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 229-235
```cpp
229:   constexpr static int Stages = Stages_;
230:   using ClusterShape = ClusterShape_;
231:   using ArchTag = arch::Sm90;
232:   using Schedule = KernelSchedule;
233: };
234: 
235: // n-buffer in smem (cp.async), pipelined with Hopper GMMA, with predicated gmem loads, warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 236-241
```cpp
236: template<
237:   int Stages_,
238:   class ClusterShape_ = Shape<_1,_1,_1>,
239:   class KernelSchedule = KernelCpAsyncWarpSpecialized
240: >
241: struct MainloopSm90CpAsyncGmmaRmemAWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 242-248
```cpp
242:   constexpr static int Stages = Stages_;
243:   using ClusterShape = ClusterShape_;
244:   using ArchTag = arch::Sm90;
245:   using Schedule = KernelSchedule;
246: };
247: 
248: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, static schedule between TMA and GMMA
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 249-254
```cpp
249: template<
250:   int Stages_,
251:   class ClusterShape_ = Shape<_1,_1,_1>,
252:   int PipelineAsyncMmaStages_ = 1
253: >
254: struct MainloopSm90TmaGmma {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 255-262
```cpp
255:   constexpr static int Stages = Stages_;
256:   using ClusterShape = ClusterShape_;
257:   constexpr static int PipelineAsyncMmaStages = PipelineAsyncMmaStages_;
258:   using ArchTag = arch::Sm90;
259:   using Schedule = KernelTma;
260: };
261: 
262: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 263-268
```cpp
263: template<
264:   int Stages_,
265:   class ClusterShape_ = Shape<_1,_1,_1>,
266:   class KernelSchedule = KernelTmaWarpSpecializedCooperative
267: >
268: struct MainloopSm90TmaGmmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 269-276
```cpp
269:   constexpr static int Stages = Stages_;
270:   using ClusterShape = ClusterShape_;
271:   using ArchTag = arch::Sm90;
272:   using Schedule = KernelSchedule;
273: };
274: 
275: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule
276: // With GMMA's A data from registers.
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 277-282
```cpp
277: template<
278:   int Stages_,
279:   class ClusterShape_ = Shape<_1,_1,_1>,
280:   class KernelSchedule = KernelTmaWarpSpecialized
281: >
282: struct MainloopSm90TmaGmmaRmemAWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 283-292
```cpp
283:   constexpr static int Stages = Stages_;
284:   using ClusterShape = ClusterShape_;
285:   using ArchTag = arch::Sm90;
286:   using Schedule = KernelSchedule;
287:   static_assert(
288:     cute::is_same_v<Schedule, KernelTmaWarpSpecialized> ||
289:     cute::is_same_v<Schedule, KernelTmaWarpSpecializedPingpong> ||
290:     cute::is_same_v<Schedule, KernelTmaWarpSpecializedCooperative>,
291:     "KernelSchedule must be one of the warp specialized policies");
292: };
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 293-294
```cpp
293: 
294: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 295-300
```cpp
295: template<
296:   int Stages_,
297:   class ClusterShape_ = Shape<_1,_1,_1>,
298:   class KernelSchedule = KernelTmaWarpSpecialized
299: >
300: struct MainloopSm90TmaGmmaRmemAWarpSpecializedMixedInput {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 301-310
```cpp
301:   constexpr static int Stages = Stages_;
302:   using ClusterShape = ClusterShape_;
303:   using ArchTag = arch::Sm90;
304:   using Schedule = KernelSchedule;
305:   static_assert(
306:     cute::is_same_v<Schedule, KernelTmaWarpSpecialized> ||
307:     cute::is_same_v<Schedule, KernelTmaWarpSpecializedPingpong> ||
308:     cute::is_same_v<Schedule, KernelTmaWarpSpecializedCooperative>,
309:     "KernelSchedule must be one of the warp specialized policies");
310: };
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 311-313
```cpp
311: 
312: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule
313: // For FP8 kernels
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 314-320
```cpp
314: template<
315:   int Stages_,
316:   class ClusterShape_ = Shape<_1,_1,_1>,
317:   class KernelSchedule = KernelTmaWarpSpecialized
318: >
319: struct MainloopSm90TmaGmmaWarpSpecializedFP8
320:   : MainloopSm90TmaGmmaWarpSpecialized<Stages_, ClusterShape_, KernelSchedule> {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 321-330
```cpp
321:   static_assert(
322:     cute::is_same_v<KernelSchedule, KernelTmaWarpSpecialized> ||
323:     cute::is_same_v<KernelSchedule, KernelTmaWarpSpecializedPingpong> ||
324:     cute::is_same_v<KernelSchedule, KernelTmaWarpSpecializedCooperative>,
325:     "KernelSchedule must be one of the warp specialized policies");
326: };
327: 
328: 
329: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule
330: // For FP8 kernels with Blockwise (Software) Scaling
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 331-337
```cpp
331: template<
332:   int Stages_,
333:   class ClusterShape_ = Shape<_1,_1,_1>,
334:   class KernelSchedule = KernelTmaWarpSpecializedCooperativeFP8Blockwise
335: >
336: struct MainloopSm90TmaGmmaWarpSpecializedBlockwiseFP8
337:   : MainloopSm90TmaGmmaWarpSpecialized<Stages_, ClusterShape_, KernelSchedule> {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 338-344
```cpp
338:   static_assert(
339:     cute::is_same_v<KernelSchedule, KernelTmaWarpSpecializedCooperativeFP8Blockwise> ||
340:     cute::is_same_v<KernelSchedule, KernelTmaWarpSpecializedPingpongFP8Blockwise>,
341:     "KernelSchedule must be one of the warp specialized FP8 block scale policies");
342: };
343: 
344: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule for Ptr-Array and Grouped Gemm
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 345-350
```cpp
345: template<
346:   int Stages_,
347:   class ClusterShape_ = Shape<_1,_1,_1>,
348:   class KernelSchedule = KernelPtrArrayTmaWarpSpecializedCooperative
349: >
350: struct MainloopSm90ArrayTmaGmmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 351-360
```cpp
351:   constexpr static int Stages = Stages_;
352:   constexpr static int PipelineAsyncMmaStages = 1;
353:   using ClusterShape = ClusterShape_;
354:   using ArchTag = arch::Sm90;
355:   using Schedule = KernelSchedule;
356:   static_assert(
357:     cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, KernelSchedule> ||
358:     cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedPingpong, KernelSchedule>,
359:     "KernelSchedule must be one of the Ptr-Array or Grouped Gemm TMA Warp Specialized Cooperative or Pingpong policies");
360: };
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 361-363
```cpp
361: 
362: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule for Ptr-Array and Grouped Gemm
363: // For FP8 kernels
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 364-370
```cpp
364: template<
365:   int Stages_,
366:   class ClusterShape_ = Shape<_1,_1,_1>,
367:   class KernelSchedule = KernelPtrArrayTmaWarpSpecializedCooperative
368: >
369: struct MainloopSm90ArrayTmaGmmaWarpSpecializedFP8
370:   : MainloopSm90ArrayTmaGmmaWarpSpecialized<Stages_, ClusterShape_, KernelSchedule> {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 371-377
```cpp
371:   static_assert(
372:     cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, KernelSchedule> ||
373:     cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedPingpong, KernelSchedule>,
374:     "KernelSchedule must be one of the Ptr-Array or Grouped Gemm TMA Warp Specialized Cooperative or Pingpong policies");
375: };
376: 
377: // n-buffer in smem (Hopper TMA), pipelined with Hopper sparse GMMA and TMA, Warp specialized dynamic schedule
```
**EN:** This block focuses on sparse related implementation details.
**CN:** 该代码块聚焦于 稀疏处理 的实现细节。

### Lines 378-383
```cpp
378: template<
379:   int Stages_,
380:   class ClusterShape_ = Shape<_1,_1,_1>,
381:   class KernelSchedule = KernelTmaWarpSpecializedCooperative
382: >
383: struct MainloopSm90TmaGmmaWarpSpecializedSparse {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 384-390
```cpp
384:   constexpr static int Stages = Stages_;
385:   using ClusterShape = ClusterShape_;
386:   using ArchTag = arch::Sm90;
387:   using Schedule = KernelSchedule;
388: };
389: 
390: // For slow-accumulation sparse FP8 kernels
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 391-400
```cpp
391: template<
392:   int Stages,
393:   class ClusterShape = Shape<_1,_1,_1>,
394:   class KernelSchedule = KernelTmaWarpSpecializedCooperative
395: >
396: struct MainloopSm90TmaGmmaWarpSpecializedSparseFP8 
397:   : MainloopSm90TmaGmmaWarpSpecializedSparse<Stages, ClusterShape, KernelSchedule> {
398: };
399: 
400: // Mixed precision version n-buffer in rmem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule for Ptr-Array and Grouped Gemm
```
**EN:** Defines ClusterShape, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape，用于封装策略、存储或算法行为的辅助类型。

### Lines 401-406
```cpp
401: template<
402:   int Stages_,
403:   class ClusterShape_ = Shape<_1,_1,_1>,
404:   class KernelSchedule = KernelPtrArrayTmaWarpSpecializedCooperative
405: >
406: struct MainloopSm90ArrayTmaGmmaWarpSpecializedMixedInput {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 407-416
```cpp
407:   constexpr static int Stages = Stages_;
408:   using ClusterShape = ClusterShape_;
409:   using ArchTag = arch::Sm90;
410:   using Schedule = KernelSchedule;
411:   static_assert(
412:     cute::is_same_v<Schedule, KernelPtrArrayTmaWarpSpecializedCooperative> ||
413:     cute::is_same_v<Schedule, KernelPtrArrayTmaWarpSpecializedPingpong>,
414:     "KernelSchedule must be one of the Ptr-Array or Grouped Gemm TMA Warp Specialized Cooperative policies");
415: };
416: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 417-418
```cpp
417: // n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp specialized dynamic schedule
418: // For FP8 kernels with Block Scaling
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 419-425
```cpp
419: template<
420:   int Stages_,
421:   class ClusterShape_ = Shape<_1,_1,_1>,
422:   class KernelSchedule = KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise
423: >
424: struct MainloopSm90ArrayTmaGmmaWarpSpecializedBlockwise
425:   : MainloopSm90ArrayTmaGmmaWarpSpecialized<Stages_, ClusterShape_, KernelSchedule> {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 426-435
```cpp
426:   static_assert(
427:     cute::is_any_of_v<
428:       KernelSchedule,
429:       KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise,
430:       KernelPtrArrayTmaWarpSpecializedPingpongFP8Blockwise
431:     >,
432:     "KernelSchedule must be one of the warp specialized FP8 block scale policies");
433: };
434: 
435: //////////////////////////////////////////////////////////////////////////////
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 436-436
```cpp
436: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 437-441
```cpp
437: //
438: // Kernel Scheduler Tag
439: //
440: 
441: // Dense GEMM: SM100 tensor op policy that applies to both 1SM and 2SM MMA atoms
```
**EN:** This block focuses on tensor related implementation details.
**CN:** 该代码块聚焦于 Tensor Core/张量算子 的实现细节。

### Lines 442-446
```cpp
442: template<
443:   int SchedulerPipelineStageCount_,
444:   int AccumulatorPipelineStageCount_
445: >
446: struct KernelWarpSpecializedSm100 final {
```
**EN:** Defines KernelWarpSpecializedSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelWarpSpecializedSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 447-450
```cpp
447:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
448:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
449: };
450: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 451-455
```cpp
451: template<
452:   int SchedulerPipelineStageCount_,
453:   int AccumulatorPipelineStageCount_
454: >
455: struct KernelMixedTmaCpAsyncWarpSpecializedSm100 final {
```
**EN:** Defines KernelMixedTmaCpAsyncWarpSpecializedSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelMixedTmaCpAsyncWarpSpecializedSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 456-459
```cpp
456:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
457:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
458: };
459: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 460-464
```cpp
460: template<
461:   int SchedulerPipelineStageCount_,
462:   int AccumulatorPipelineStageCount_
463: >
464: struct KernelTmaWarpSpecializedSm100 final {
```
**EN:** Defines KernelTmaWarpSpecializedSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 465-469
```cpp
465:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
466:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
467: };
468: 
469: // Gemm with block scaling factors
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 470-474
```cpp
470: template<
471:   int SchedulerPipelineStageCount_,
472:   int AccumulatorPipelineStageCount_
473: >
474: struct KernelTmaWarpSpecializedBlockScaledSm100 final {
```
**EN:** Defines KernelTmaWarpSpecializedBlockScaledSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedBlockScaledSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 475-478
```cpp
475:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
476:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
477: };
478: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 479-483
```cpp
479: template<
480:   int SchedulerPipelineStageCount_,
481:   int AccumulatorPipelineStageCount_
482: >
483: struct KernelTmaWarpSpecializedMmaTransformSm100 final {
```
**EN:** Defines KernelTmaWarpSpecializedMmaTransformSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedMmaTransformSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 484-487
```cpp
484:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
485:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
486: };
487: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 488-492
```cpp
488: template<
489:   int SchedulerPipelineStageCount_,
490:   int AccumulatorPipelineStageCount_
491: >
492: struct KernelPtrArrayTmaWarpSpecializedMmaTransformSm100 final {
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedMmaTransformSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedMmaTransformSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 493-496
```cpp
493:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
494:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
495: };
496: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 497-501
```cpp
497: template<
498:   int SchedulerPipelineStageCount_,
499:   int AccumulatorPipelineStageCount_
500: >
501: struct KernelTmaWarpSpecializedBlockScaledSm103 final {
```
**EN:** Defines KernelTmaWarpSpecializedBlockScaledSm103, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedBlockScaledSm103，用于封装策略、存储或算法行为的辅助类型。

### Lines 502-505
```cpp
502:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
503:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
504: };
505: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 506-510
```cpp
506: template<
507:   int SchedulerPipelineStageCount_,
508:   int AccumulatorPipelineStageCount_
509: >
510: struct KernelPtrArrayTmaWarpSpecializedBlockScaledSm103 final {
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedBlockScaledSm103, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedBlockScaledSm103，用于封装策略、存储或算法行为的辅助类型。

### Lines 511-515
```cpp
511:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
512:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
513: };
514: 
515: // Sparse Gemm
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 516-520
```cpp
516: template<
517:   int SchedulerPipelineStageCount_,
518:   int AccumulatorPipelineStageCount_
519: >
520: struct KernelSparseTmaWarpSpecializedSm100 final {
```
**EN:** Defines KernelSparseTmaWarpSpecializedSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSparseTmaWarpSpecializedSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 521-525
```cpp
521:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
522:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
523: };
524: 
525: // Sparse Gemm with block scaling factors
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 526-530
```cpp
526: template<
527:   int SchedulerPipelineStageCount_,
528:   int AccumulatorPipelineStageCount_
529: >
530: struct KernelSparseTmaWarpSpecializedBlockScaledSm100 final {
```
**EN:** Defines KernelSparseTmaWarpSpecializedBlockScaledSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSparseTmaWarpSpecializedBlockScaledSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 531-535
```cpp
531:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
532:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
533: };
534: 
535: // InputTransform GEMM
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 536-540
```cpp
536: template<
537:   int SchedulerPipelineStageCount_,
538:   int AccumulatorPipelineStageCount_
539: >
540: struct KernelTmaWarpSpecializedInputTransformSm100 final {
```
**EN:** Defines KernelTmaWarpSpecializedInputTransformSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedInputTransformSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 541-545
```cpp
541:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
542:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
543: };
544: 
545: // InputTransform GEMM
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 546-550
```cpp
546: template<
547:   int SchedulerPipelineStageCount_,
548:   int AccumulatorPipelineStageCount_
549: >
550: struct KernelTmaWarpSpecializedMixedInputTransformSm100 final {
```
**EN:** Defines KernelTmaWarpSpecializedMixedInputTransformSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedMixedInputTransformSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 551-555
```cpp
551:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
552:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
553: };
554: 
555: // Ptr-Array Dense GEMM: SM100 tensor op policy that applies to both 1SM and 2SM MMA atoms
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 556-560
```cpp
556: template<
557:   int SchedulerPipelineStageCount_,
558:   int AccumulatorPipelineStageCount_
559: >
560: struct KernelPtrArrayTmaWarpSpecializedSm100 final {
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 561-565
```cpp
561:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
562:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
563: };
564: 
565: // Ptr-Array Block Scaled GEMM
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 566-570
```cpp
566: template<
567:   int SchedulerPipelineStageCount_,
568:   int AccumulatorPipelineStageCount_
569: >
570: struct KernelPtrArrayTmaWarpSpecializedBlockScaledSm100 final {
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedBlockScaledSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedBlockScaledSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 571-575
```cpp
571:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
572:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
573: };
574: 
575: // Ptr-Array InputTransform GEMM
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 576-580
```cpp
576: template<
577:   int SchedulerPipelineStageCount_,
578:   int AccumulatorPipelineStageCount_
579: >
580: struct KernelPtrArrayTmaWarpSpecializedInputTransformSm100 final {
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedInputTransformSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedInputTransformSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 581-586
```cpp
581:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
582:   static constexpr int AccumulatorPipelineStageCount = AccumulatorPipelineStageCount_;
583: };
584: 
585: 
586: // SM120 kernel schedules
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 587-591
```cpp
587: template<int SchedulerPipelineStageCount_>
588: struct KernelTmaWarpSpecializedCooperativeSm120 : KernelTmaWarpSpecializedCooperative { 
589:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
590: };
591: 
```
**EN:** Defines KernelTmaWarpSpecializedCooperativeSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedCooperativeSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 592-597
```cpp
592: template<int SchedulerPipelineStageCount_>
593: struct KernelTmaWarpSpecializedPingpongSm120 : KernelTmaWarpSpecializedPingpong { 
594:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
595: };
596: 
597: 
```
**EN:** Defines KernelTmaWarpSpecializedPingpongSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedPingpongSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 598-602
```cpp
598: template<int SchedulerPipelineStageCount_>
599: struct KernelTmaWarpSpecializedCooperativeBlockScaledSm120 : KernelTmaWarpSpecializedCooperative { 
600:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
601: };
602: 
```
**EN:** Defines KernelTmaWarpSpecializedCooperativeBlockScaledSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedCooperativeBlockScaledSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 603-608
```cpp
603: template<int SchedulerPipelineStageCount_>
604: struct KernelTmaWarpSpecializedPingpongBlockScaledSm120 : KernelTmaWarpSpecializedPingpong { 
605:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
606: };
607: 
608: // SM120 dense Ptr-array kernel schedules
```
**EN:** Defines KernelTmaWarpSpecializedPingpongBlockScaledSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedPingpongBlockScaledSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 609-613
```cpp
609: template<int SchedulerPipelineStageCount_>
610: struct KernelPtrArrayTmaWarpSpecializedCooperativeSm120 : KernelPtrArrayTmaWarpSpecializedCooperative {
611:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
612: };
613: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedCooperativeSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedCooperativeSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 614-618
```cpp
614: template<int SchedulerPipelineStageCount_>
615: struct KernelPtrArrayTmaWarpSpecializedPingpongSm120 : KernelPtrArrayTmaWarpSpecializedPingpong {
616:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
617: };
618: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedPingpongSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedPingpongSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 619-623
```cpp
619: template<int SchedulerPipelineStageCount_>
620: struct KernelPtrArrayTmaWarpSpecializedCooperativeBlockScaledSm120 : KernelPtrArrayTmaWarpSpecializedCooperative {
621:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
622: };
623: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedCooperativeBlockScaledSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedCooperativeBlockScaledSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 624-629
```cpp
624: template<int SchedulerPipelineStageCount_>
625: struct KernelPtrArrayTmaWarpSpecializedPingpongBlockScaledSm120 : KernelPtrArrayTmaWarpSpecializedPingpong {
626:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
627: };
628: 
629: // SM120 sparse kernel schedules
```
**EN:** Defines KernelPtrArrayTmaWarpSpecializedPingpongBlockScaledSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecializedPingpongBlockScaledSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 630-635
```cpp
630: template<int SchedulerPipelineStageCount_, bool isAsymmetric_>
631: struct KernelTmaWarpSpecializedCooperativeSparseSm120 {
632:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
633:   static constexpr bool isAsymmetric = isAsymmetric_;
634: };
635: 
```
**EN:** Defines KernelTmaWarpSpecializedCooperativeSparseSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedCooperativeSparseSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 636-642
```cpp
636: template<int SchedulerPipelineStageCount_, bool isAsymmetric_>
637: struct KernelTmaWarpSpecializedCooperativeSparseBlockScaledSm120 {
638:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
639:   static constexpr bool isAsymmetric = isAsymmetric_;
640: };
641: 
642: // SM120 blockwise kernel schedules
```
**EN:** Defines KernelTmaWarpSpecializedCooperativeSparseBlockScaledSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedCooperativeSparseBlockScaledSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 643-647
```cpp
643: template <int SchedulerPipelineStageCount_>
644: struct KernelTmaWarpSpecializedCooperativeBlockwiseScalingSm120 : KernelTmaWarpSpecializedCooperative {
645:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
646: };
647: 
```
**EN:** Declares template parameters and begins the definition of KernelTmaWarpSpecializedCooperativeBlockwiseScalingSm120.
**CN:** 声明模板参数并开始定义 KernelTmaWarpSpecializedCooperativeBlockwiseScalingSm120。

### Lines 648-652
```cpp
648: template <int SchedulerPipelineStageCount_>
649: struct KernelTmaWarpSpecializedPingpongBlockwiseScalingSm120 : KernelTmaWarpSpecializedPingpong {
650:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
651: };
652: 
```
**EN:** Declares template parameters and begins the definition of KernelTmaWarpSpecializedPingpongBlockwiseScalingSm120.
**CN:** 声明模板参数并开始定义 KernelTmaWarpSpecializedPingpongBlockwiseScalingSm120。

### Lines 653-657
```cpp
653: template <int SchedulerPipelineStageCount_>
654: struct KernelPtrArrayTmaWarpSpecializedCooperativeBlockwiseScalingSm120 : KernelPtrArrayTmaWarpSpecializedCooperative {
655:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
656: };
657: 
```
**EN:** Declares template parameters and begins the definition of KernelPtrArrayTmaWarpSpecializedCooperativeBlockwiseScalingSm120.
**CN:** 声明模板参数并开始定义 KernelPtrArrayTmaWarpSpecializedCooperativeBlockwiseScalingSm120。

### Lines 658-666
```cpp
658: template <int SchedulerPipelineStageCount_>
659: struct KernelPtrArrayTmaWarpSpecializedPingpongBlockwiseScalingSm120 : KernelPtrArrayTmaWarpSpecializedPingpong {
660:   static constexpr int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
661: };
662: 
663: // Auxiliary Load Tag.
664: 
665: namespace kernel::detail {
666: 
```
**EN:** Enters namespace scope (kernel) to organize the GEMM abstraction layer.
**CN:** 进入命名空间作用域（kernel），组织 GEMM 抽象层。

### Lines 667-678
```cpp
667: template<
668:   int Stages,
669:   class ClusterShape,
670:   class KernelSchedule
671: >
672: struct HasAuxiliaryLoad<
673:   MainloopSm90ArrayTmaGmmaWarpSpecializedBlockwise<
674:     Stages,
675:     ClusterShape,
676:     KernelSchedule
677:   >
678: > : cute::true_type{};
```
**EN:** Defines ClusterShape, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape，用于封装策略、存储或算法行为的辅助类型。

### Lines 679-679
```cpp
679: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 680-691
```cpp
680: template<
681:   int Stages,
682:   class ClusterShape,
683:   class KernelSchedule
684: >
685: struct HasAuxiliaryLoad<
686:   MainloopSm90TmaGmmaWarpSpecializedBlockwiseFP8<
687:     Stages,
688:     ClusterShape,
689:     KernelSchedule
690:   >
691: > : cute::true_type{};
```
**EN:** Defines ClusterShape, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape，用于封装策略、存储或算法行为的辅助类型。

### Lines 692-696
```cpp
692: 
693: } // namespace kernel::detail
694: 
695: //////////////////////////////////////////////////////////////////////////////
696: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 697-700
```cpp
697: //
698: // Collective Builder Tag Property
699: //
700: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 701-710
```cpp
701: ///////////////////////////////////////////////////////////////////////////////////////////////////////
702: //
703: //          SM100 Dispatch Policies
704: //
705: ///////////////////////////////////////////////////////////////////////////////////////////////////////
706: 
707: // Builder Tag Base Dispatch Policies
708: struct KernelSchedule1Sm {};
709: struct KernelSchedule2Sm {};
710: struct KernelScheduleSm100 {};
```
**EN:** Defines KernelSchedule1Sm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedule1Sm，用于封装策略、存储或算法行为的辅助类型。

### Lines 711-711
```cpp
711: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 712-716
```cpp
712: ///////////////////////////////////////////////////////////////////////////////////////////////////////
713: // SM100 Dense GEMM Dispatch Policies
714: ///////////////////////////////////////////////////////////////////////////////////////////////////////
715: struct KernelScheduleSm100DenseGemm : KernelScheduleSm100 {};   // Base policy
716: // Dense GEMM: Specialize for 1SM vs 2SM
```
**EN:** Defines KernelScheduleSm100DenseGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100DenseGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 717-721
```cpp
717: struct KernelTmaWarpSpecialized1SmSm100 final : KernelSchedule1Sm, KernelScheduleSm100DenseGemm {};  // Use for 1SM Dense GEMM Kernels for Collective Mainloop Builder
718: struct KernelTmaWarpSpecialized2SmSm100 final : KernelSchedule2Sm, KernelScheduleSm100DenseGemm {};  // Use for 2SM Dense GEMM Kernels for Collective Mainloop Builder
719: struct KernelWarpSpecialized1SmSm100    final : KernelSchedule1Sm, KernelScheduleSm100DenseGemm {};  // Use for 1SM Dense GEMM Kernels for Collective Mainloop Builder Without TMA
720: struct KernelMixedTmaCpAsyncWarpSpecialized1SmSm100 final : KernelSchedule1Sm, KernelScheduleSm100DenseGemm {};
721: 
```
**EN:** Defines KernelTmaWarpSpecialized1SmSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecialized1SmSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 722-730
```cpp
722: ///////////////////////////////////////////////////////////////////////////////////////////////////////
723: // SM100 Ptr-Array Dense GEMM Dispatch Policies
724: ///////////////////////////////////////////////////////////////////////////////////////////////////////
725: // Dense GEMM + (Ptr Array or Group GEMM)
726: struct KernelScheduleSm100PtrArrayDenseGemm : KernelScheduleSm100DenseGemm {};
727: // Ptr-Array Dense GEMM: Specialize for 1SM vs 2SM
728: struct KernelPtrArrayTmaWarpSpecialized1SmSm100 final : KernelSchedule1Sm, KernelScheduleSm100PtrArrayDenseGemm {};
729: struct KernelPtrArrayTmaWarpSpecialized2SmSm100 final : KernelSchedule2Sm, KernelScheduleSm100PtrArrayDenseGemm {};
730: 
```
**EN:** Defines KernelScheduleSm100PtrArrayDenseGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100PtrArrayDenseGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 731-740
```cpp
731: ///////////////////////////////////////////////////////////////////////////////////////////////////////
732: // SM100 Blockwise GEMM + Ptr-Array GEMM Dispatch Policies
733: ///////////////////////////////////////////////////////////////////////////////////////////////////////
734: struct KernelScheduleSm100Blockwise  : KernelScheduleSm100 {};
735: struct KernelTmaWarpSpecializedBlockwise1SmSm100 final : KernelSchedule1Sm, KernelScheduleSm100Blockwise {};
736: struct KernelTmaWarpSpecializedBlockwise2SmSm100 final : KernelSchedule2Sm, KernelScheduleSm100Blockwise {};
737: 
738: struct KernelScheduleSm100PtrArrayBlockwise  : KernelScheduleSm100Blockwise {};
739: struct KernelPtrArrayTmaWarpSpecializedBlockwise1SmSm100 final : KernelSchedule1Sm, KernelScheduleSm100PtrArrayBlockwise {};
740: struct KernelPtrArrayTmaWarpSpecializedBlockwise2SmSm100 final : KernelSchedule2Sm, KernelScheduleSm100PtrArrayBlockwise {};
```
**EN:** Defines KernelScheduleSm100Blockwise, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100Blockwise，用于封装策略、存储或算法行为的辅助类型。

### Lines 741-741
```cpp
741: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 742-749
```cpp
742: ///////////////////////////////////////////////////////////////////////////////////////////////////////
743: // SM100 Planar Complex GEMM Dispatch Policies
744: ///////////////////////////////////////////////////////////////////////////////////////////////////////
745: struct KernelScheduleSm100PlanarComplexGemm : KernelScheduleSm100{};
746: // Planar Complex GEMM: Specialize for 1SM vs 2SM
747: struct KernelTmaWarpSpecialized1SmPlanarComplexSm100 final : KernelSchedule1Sm, KernelScheduleSm100PlanarComplexGemm { };
748: struct KernelTmaWarpSpecialized2SmPlanarComplexSm100 final : KernelSchedule2Sm, KernelScheduleSm100PlanarComplexGemm { };
749: 
```
**EN:** Defines KernelScheduleSm100PlanarComplexGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100PlanarComplexGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 750-758
```cpp
750: ///////////////////////////////////////////////////////////////////////////////////////////////////////
751: // SM100 Ptr-Array Planar Complex GEMM Dispatch Policies
752: ///////////////////////////////////////////////////////////////////////////////////////////////////////
753: // Planar Complex GEMM + (Ptr Array or Group GEMM)
754: struct KernelScheduleSm100PtrArrayPlanarComplexGemm : KernelScheduleSm100PlanarComplexGemm {};
755: 
756: struct KernelPtrArrayTmaWarpSpecialized1SmPlanarComplexSm100 final : KernelSchedule1Sm, KernelScheduleSm100PtrArrayPlanarComplexGemm {};
757: struct KernelPtrArrayTmaWarpSpecialized2SmPlanarComplexSm100 final : KernelSchedule2Sm, KernelScheduleSm100PtrArrayPlanarComplexGemm {};
758: 
```
**EN:** Defines KernelScheduleSm100PtrArrayPlanarComplexGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100PtrArrayPlanarComplexGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 759-768
```cpp
759: ///////////////////////////////////////////////////////////////////////////////////////////////////////
760: // SM100 FastF32 (9xBF16) GEMM Dispatch Policies
761: ///////////////////////////////////////////////////////////////////////////////////////////////////////
762: struct KernelScheduleSm100FastFP32Gemm           : KernelScheduleSm100 {};
763: struct KernelTmaWarpSpecializedFastFP32SmemSm100 : KernelScheduleSm100FastFP32Gemm { };
764: // Dispatch policies without smem load the A operand from tmem
765: struct KernelTmaWarpSpecialized1SmFastFP32Sm100 final : KernelSchedule1Sm, KernelScheduleSm100FastFP32Gemm { };
766: struct KernelTmaWarpSpecialized2SmFastFP32Sm100 final : KernelSchedule2Sm, KernelScheduleSm100FastFP32Gemm { };
767: // Dispatch policies with smem load the A operand from smem
768: struct KernelTmaWarpSpecialized1SmFastFP32SmemSm100 final : KernelSchedule1Sm, KernelTmaWarpSpecializedFastFP32SmemSm100 { };
```
**EN:** Defines KernelScheduleSm100FastFP32Gemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100FastFP32Gemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 769-770
```cpp
769: struct KernelTmaWarpSpecialized2SmFastFP32SmemSm100 final : KernelSchedule2Sm, KernelTmaWarpSpecializedFastFP32SmemSm100 { };
770: 
```
**EN:** Defines KernelTmaWarpSpecialized2SmFastFP32SmemSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecialized2SmFastFP32SmemSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 771-780
```cpp
771: ///////////////////////////////////////////////////////////////////////////////////////////////////////
772: // SM100 Mixed Precision Input GEMM Dispatch Policies
773: ///////////////////////////////////////////////////////////////////////////////////////////////////////
774: struct KernelScheduleSm100MixedInputGemm           : KernelScheduleSm100 {};
775: struct KernelTmaWarpSpecializedMixedInputSmemSm100 : KernelScheduleSm100MixedInputGemm { };
776: struct KernelTmaWarpSpecialized1SmMixedInputSm100 final : KernelSchedule1Sm, KernelScheduleSm100MixedInputGemm { };
777: struct KernelTmaWarpSpecialized1SmMixedInputSmemSm100 final : KernelSchedule1Sm, KernelTmaWarpSpecializedMixedInputSmemSm100 { };
778: struct KernelTmaWarpSpecialized2SmMixedInputSm100 final : KernelSchedule2Sm, KernelScheduleSm100MixedInputGemm { };
779: struct KernelTmaWarpSpecialized2SmMixedInputSmemSm100 final : KernelSchedule2Sm, KernelTmaWarpSpecializedMixedInputSmemSm100 { };
780: 
```
**EN:** Defines KernelScheduleSm100MixedInputGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100MixedInputGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 781-790
```cpp
781: ///////////////////////////////////////////////////////////////////////////////////////////////////////
782: // SM100 Ptr-Array FastF32 (9xBF16) GEMM Dispatch Policies
783: ///////////////////////////////////////////////////////////////////////////////////////////////////////
784: // Ptr-Array Transform GEMM: Specialize for 1SM vs 2SM FastF32 GEMM
785: struct KernelScheduleSm100PtrArrayFastFP32Gemm           : KernelScheduleSm100FastFP32Gemm {};
786: struct KernelTmaWarpSpecializedPtrArrayFastFP32SmemSm100 : KernelScheduleSm100PtrArrayFastFP32Gemm { };
787: 
788: struct KernelPtrArrayTmaWarpSpecialized1SmFastFP32Sm100     final : KernelSchedule1Sm, KernelScheduleSm100PtrArrayFastFP32Gemm { };
789: struct KernelPtrArrayTmaWarpSpecialized2SmFastFP32Sm100     final : KernelSchedule2Sm, KernelScheduleSm100PtrArrayFastFP32Gemm { };
790: struct KernelPtrArrayTmaWarpSpecialized1SmFastFP32SmemSm100 final : KernelSchedule1Sm, KernelTmaWarpSpecializedPtrArrayFastFP32SmemSm100 { };
```
**EN:** Defines KernelScheduleSm100PtrArrayFastFP32Gemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm100PtrArrayFastFP32Gemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 791-792
```cpp
791: struct KernelPtrArrayTmaWarpSpecialized2SmFastFP32SmemSm100 final : KernelSchedule2Sm, KernelTmaWarpSpecializedPtrArrayFastFP32SmemSm100 { };
792: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecialized2SmFastFP32SmemSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecialized2SmFastFP32SmemSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 793-800
```cpp
793: ///////////////////////////////////////////////////////////////////////////////////////////////////////
794: // SM100 Sparse GEMM Dispatch Policies
795: ///////////////////////////////////////////////////////////////////////////////////////////////////////
796: struct KernelScheduleSparseGemmSm100 : KernelScheduleSm100 {};
797: // Sparse GEMM: Specialize for 1SM vs 2SM
798: struct KernelSparseTmaWarpSpecialized1SmSm100 final : KernelSchedule1Sm, KernelScheduleSparseGemmSm100 { };
799: struct KernelSparseTmaWarpSpecialized2SmSm100 final : KernelSchedule2Sm, KernelScheduleSparseGemmSm100 { };
800: 
```
**EN:** Defines KernelScheduleSparseGemmSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSparseGemmSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 801-810
```cpp
801: ///////////////////////////////////////////////////////////////////////////////////////////////////////
802: // SM100 BlockScaled Dense GEMM Dispatch Policies
803: ///////////////////////////////////////////////////////////////////////////////////////////////////////
804: struct KernelScheduleBlockScaledGemmSm100   : KernelScheduleSm100 {};                  
805: struct KernelScheduleMxNvf4Sm100            : KernelScheduleBlockScaledGemmSm100 {};
806: struct KernelScheduleMxf8f6f4Sm100          : KernelScheduleBlockScaledGemmSm100 {};
807: // Block Scaled Dense GEMM: Specialize for instruction type, scale factor vector size, and 1SM vs. 2SM
808: struct KernelTmaWarpSpecialized1SmBlockScaledSm100       final : KernelSchedule1Sm, KernelScheduleBlockScaledGemmSm100 { };
809: struct KernelTmaWarpSpecialized2SmBlockScaledSm100       final : KernelSchedule2Sm, KernelScheduleBlockScaledGemmSm100 { };
810: struct KernelTmaWarpSpecialized1SmNvf4Sm100              final : KernelSchedule1Sm, KernelScheduleMxNvf4Sm100 { };
```
**EN:** Defines KernelScheduleBlockScaledGemmSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleBlockScaledGemmSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 811-817
```cpp
811: struct KernelTmaWarpSpecialized2SmNvf4Sm100              final : KernelSchedule2Sm, KernelScheduleMxNvf4Sm100 { };
812: struct KernelTmaWarpSpecialized1SmMxf4Sm100              final : KernelSchedule1Sm, KernelScheduleMxNvf4Sm100 { };
813: struct KernelTmaWarpSpecialized2SmMxf4Sm100              final : KernelSchedule2Sm, KernelScheduleMxNvf4Sm100 { };
814: struct KernelTmaWarpSpecialized1SmMxf8f6f4Sm100          final : KernelSchedule1Sm, KernelScheduleMxf8f6f4Sm100 { };
815: struct KernelTmaWarpSpecialized2SmMxf8f6f4Sm100          final : KernelSchedule2Sm, KernelScheduleMxf8f6f4Sm100 { };
816: struct KernelMixedTmaCpAsyncWarpSpecialized1SmBlockScaledSm100 final : KernelSchedule1Sm, KernelScheduleBlockScaledGemmSm100 {};
817: 
```
**EN:** Defines KernelTmaWarpSpecialized2SmNvf4Sm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecialized2SmNvf4Sm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 818-827
```cpp
818: ///////////////////////////////////////////////////////////////////////////////////////////////////////
819: // SM100 BlockScaled Ptr Array Dense GEMM Dispatch Policies
820: ///////////////////////////////////////////////////////////////////////////////////////////////////////
821: // BlockScaled Dense GEMM + (Ptr Array or Group GEMM)
822: struct KernelSchedulePtrArrayBlockScaledGemmSm100   : KernelScheduleBlockScaledGemmSm100 {};
823: struct KernelSchedulePtrArrayMxNvf4Sm100            : KernelSchedulePtrArrayBlockScaledGemmSm100 {};
824: struct KernelSchedulePtrArrayMxf8f6f4Sm100          : KernelSchedulePtrArrayBlockScaledGemmSm100 {};
825: // Ptr-Array Block Scaled Dense GEMM: Specialize for instruction type, scale factor vector size, and 1SM vs. 2SM
826: struct KernelPtrArrayTmaWarpSpecialized1SmBlockScaledSm100       final : KernelSchedule1Sm, KernelSchedulePtrArrayBlockScaledGemmSm100 { };
827: struct KernelPtrArrayTmaWarpSpecialized2SmBlockScaledSm100       final : KernelSchedule2Sm, KernelSchedulePtrArrayBlockScaledGemmSm100 { };
```
**EN:** Defines KernelSchedulePtrArrayBlockScaledGemmSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedulePtrArrayBlockScaledGemmSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 828-833
```cpp
828: struct KernelPtrArrayTmaWarpSpecialized1SmNvf4Sm100              final : KernelSchedule1Sm, KernelSchedulePtrArrayMxNvf4Sm100 { };
829: struct KernelPtrArrayTmaWarpSpecialized2SmNvf4Sm100              final : KernelSchedule2Sm, KernelSchedulePtrArrayMxNvf4Sm100 { };
830: struct KernelPtrArrayTmaWarpSpecialized1SmMxf4Sm100              final : KernelSchedule1Sm, KernelSchedulePtrArrayMxNvf4Sm100 { };
831: struct KernelPtrArrayTmaWarpSpecialized2SmMxf4Sm100              final : KernelSchedule2Sm, KernelSchedulePtrArrayMxNvf4Sm100 { };
832: struct KernelPtrArrayTmaWarpSpecialized1SmMxf8f6f4Sm100          final : KernelSchedule1Sm, KernelSchedulePtrArrayMxf8f6f4Sm100 { };
833: struct KernelPtrArrayTmaWarpSpecialized2SmMxf8f6f4Sm100          final : KernelSchedule2Sm, KernelSchedulePtrArrayMxf8f6f4Sm100 { };
```
**EN:** Defines KernelPtrArrayTmaWarpSpecialized1SmNvf4Sm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecialized1SmNvf4Sm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 834-843
```cpp
834: ///////////////////////////////////////////////////////////////////////////////////////////////////////
835: // SM100 BlockScaled Sparse GEMM Dispatch Policies
836: ///////////////////////////////////////////////////////////////////////////////////////////////////////
837: struct KernelScheduleBlockScaledSparseGemmSm100 : KernelScheduleSm100 {};
838: struct KernelScheduleSparseMxNvf4Sm100          : KernelScheduleBlockScaledSparseGemmSm100 {};
839: struct KernelScheduleSparseMxf8f6f4Sm100        : KernelScheduleBlockScaledSparseGemmSm100 {};
840: // Block Scaled Sparse GEMM: Specialize for instruction type, scale factor vector size, and 1SM vs. 2SM
841: struct KernelSparseTmaWarpSpecialized1SmBlockScaledSm100 final : KernelSchedule1Sm, KernelScheduleBlockScaledSparseGemmSm100 {};
842: struct KernelSparseTmaWarpSpecialized2SmBlockScaledSm100 final : KernelSchedule2Sm, KernelScheduleBlockScaledSparseGemmSm100 {};
843: struct KernelSparseTmaWarpSpecialized1SmMxf8f6f4Sm100    final : KernelSchedule1Sm, KernelScheduleSparseMxf8f6f4Sm100 { };
```
**EN:** Defines KernelScheduleBlockScaledSparseGemmSm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleBlockScaledSparseGemmSm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 844-849
```cpp
844: struct KernelSparseTmaWarpSpecialized2SmMxf8f6f4Sm100    final : KernelSchedule2Sm, KernelScheduleSparseMxf8f6f4Sm100 { };
845: struct KernelSparseTmaWarpSpecialized1SmNvf4Sm100        final : KernelSchedule1Sm, KernelScheduleSparseMxNvf4Sm100 { };
846: struct KernelSparseTmaWarpSpecialized2SmNvf4Sm100        final : KernelSchedule2Sm, KernelScheduleSparseMxNvf4Sm100 { };
847: struct KernelSparseTmaWarpSpecialized1SmMxf4Sm100        final : KernelSchedule1Sm, KernelScheduleSparseMxNvf4Sm100 { };
848: struct KernelSparseTmaWarpSpecialized2SmMxf4Sm100        final : KernelSchedule2Sm, KernelScheduleSparseMxNvf4Sm100 { };
849: 
```
**EN:** Defines KernelSparseTmaWarpSpecialized2SmMxf8f6f4Sm100, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSparseTmaWarpSpecialized2SmMxf8f6f4Sm100，用于封装策略、存储或算法行为的辅助类型。

### Lines 850-859
```cpp
850: ///////////////////////////////////////////////////////////////////////////////////////////////////////
851: //
852: //          SM103 Dispatch Policies
853: //
854: ///////////////////////////////////////////////////////////////////////////////////////////////////////
855: 
856: struct KernelScheduleSm103 {};
857: struct KernelScheduleSm103BlockScaledGemm                  : KernelScheduleSm103 {};
858: struct KernelScheduleSm103BlockScaledMxNvf4UltraTmaPrefetch     : KernelScheduleSm103BlockScaledGemm {};
859: struct KernelScheduleSm103BlockScaledMxNvf4UltraDisablePrefetch : KernelScheduleSm103BlockScaledGemm {};
```
**EN:** Defines KernelScheduleSm103, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm103，用于封装策略、存储或算法行为的辅助类型。

### Lines 860-869
```cpp
860: 
861: // Blockscaled Gemm: Specialized for instruction type, scale factor vector size, and 1SM vs. 2SM
862: // These are the public dispatch policy name
863: struct KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch final : KernelSchedule1Sm, KernelScheduleSm103BlockScaledMxNvf4UltraTmaPrefetch { };
864: struct KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch final : KernelSchedule2Sm, KernelScheduleSm103BlockScaledMxNvf4UltraTmaPrefetch { };
865: struct KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch final : KernelSchedule1Sm, KernelScheduleSm103BlockScaledMxNvf4UltraTmaPrefetch { };
866: struct KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch final : KernelSchedule2Sm, KernelScheduleSm103BlockScaledMxNvf4UltraTmaPrefetch { };
867: 
868: struct KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch final : KernelSchedule1Sm, KernelScheduleSm103BlockScaledMxNvf4UltraDisablePrefetch { };
869: struct KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch final : KernelSchedule2Sm, KernelScheduleSm103BlockScaledMxNvf4UltraDisablePrefetch { };
```
**EN:** Defines KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch，用于封装策略、存储或算法行为的辅助类型。

### Lines 870-872
```cpp
870: struct KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch final : KernelSchedule1Sm, KernelScheduleSm103BlockScaledMxNvf4UltraDisablePrefetch { };
871: struct KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch final : KernelSchedule2Sm, KernelScheduleSm103BlockScaledMxNvf4UltraDisablePrefetch { };
872: 
```
**EN:** Defines KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch，用于封装策略、存储或算法行为的辅助类型。

### Lines 873-882
```cpp
873: using KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103 = KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch;
874: using KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103 = KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch;
875: using KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103 = KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch;
876: using KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103 = KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch;
877: 
878: 
879: struct KernelSchedulePtrArraySm103BlockScaledGemm                  : KernelScheduleSm103 {};
880: struct KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraTmaPrefetch     : KernelSchedulePtrArraySm103BlockScaledGemm {};
881: struct KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraDisablePrefetch : KernelSchedulePtrArraySm103BlockScaledGemm {};
882: 
```
**EN:** Defines KernelSchedulePtrArraySm103BlockScaledGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedulePtrArraySm103BlockScaledGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 883-892
```cpp
883: struct KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch final : KernelSchedule1Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraTmaPrefetch { };
884: struct KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch final : KernelSchedule2Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraTmaPrefetch { };
885: struct KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch final : KernelSchedule1Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraTmaPrefetch { };
886: struct KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103TmaPrefetch final : KernelSchedule2Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraTmaPrefetch { };
887: 
888: struct KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch final : KernelSchedule1Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraDisablePrefetch { };
889: struct KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch final : KernelSchedule2Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraDisablePrefetch { };
890: struct KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch final : KernelSchedule1Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraDisablePrefetch { };
891: struct KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch final : KernelSchedule2Sm, KernelSchedulePtrArraySm103BlockScaledMxNvf4UltraDisablePrefetch { };
892: 
```
**EN:** Defines KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103TmaPrefetch，用于封装策略、存储或算法行为的辅助类型。

### Lines 893-897
```cpp
893: using KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103 = KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch;
894: using KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103 = KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103DisablePrefetch;
895: using KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103 = KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch;
896: using KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103 = KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103DisablePrefetch;
897: 
```
**EN:** Introduces local type aliases (KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103, KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103, KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103, KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103) to simplify downstream template code.
**CN:** 引入本地类型别名（KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103, KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103, KernelPtrArrayTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs32Sm103, KernelPtrArrayTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs32Sm103），简化后续模板代码。

### Lines 898-907
```cpp
898: ///////////////////////////////////////////////////////////////////////////////////////////////////////
899: //
900: //          SM120 Dispatch Policies
901: //
902: ///////////////////////////////////////////////////////////////////////////////////////////////////////
903: 
904: // Builder Tag Base Dispatch Policies
905: struct KernelScheduleSm120 {};
906: struct KernelScheduleAcc2x4Sm120 {};
907: 
```
**EN:** Defines KernelScheduleSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 908-914
```cpp
908: ///////////////////////////////////////////////////////////////////////////////////////////////////////
909: // SM100 Dense GEMM Dispatch Policies
910: ///////////////////////////////////////////////////////////////////////////////////////////////////////
911: struct KernelScheduleSm120DenseGemm : KernelScheduleSm120 {};
912: // Dense GEMM: Specialize for instruction type
913: struct KernelScheduleF8f6f4Sm120 final : KernelScheduleSm120DenseGemm {};
914: 
```
**EN:** Defines KernelScheduleSm120DenseGemm, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSm120DenseGemm，用于封装策略、存储或算法行为的辅助类型。

### Lines 915-924
```cpp
915: ///////////////////////////////////////////////////////////////////////////////////////////////////////
916: // SM120 BlockScaled GEMM Dispatch Policies
917: ///////////////////////////////////////////////////////////////////////////////////////////////////////
918: struct KernelScheduleBlockScaledGemmSm120 : KernelScheduleSm120 {};
919: struct KernelScheduleMxf8f6f4Sm120        : KernelScheduleBlockScaledGemmSm120 {};
920: struct KernelScheduleMxNvf4Sm120          : KernelScheduleBlockScaledGemmSm120 {};
921: // Block Scaled GEMM: Specialize for instruction type, scale factor vector size.
922: struct KernelTmaWarpSpecializedNvf4Sm120             final : KernelScheduleMxNvf4Sm120, KernelTmaWarpSpecializedCooperative { };
923: struct KernelTmaWarpSpecializedPingpongNvf4Sm120     final : KernelScheduleMxNvf4Sm120, KernelTmaWarpSpecializedPingpong { };
924: struct KernelTmaWarpSpecializedMxf4Sm120             final : KernelScheduleMxNvf4Sm120, KernelTmaWarpSpecializedCooperative { };
```
**EN:** Defines KernelScheduleBlockScaledGemmSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleBlockScaledGemmSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 925-933
```cpp
925: struct KernelTmaWarpSpecializedPingpongMxf4Sm120     final : KernelScheduleMxNvf4Sm120, KernelTmaWarpSpecializedPingpong { };
926: struct KernelTmaWarpSpecializedMxf8f6f4Sm120         final : KernelScheduleMxf8f6f4Sm120, KernelTmaWarpSpecializedCooperative { };
927: struct KernelTmaWarpSpecializedPingpongMxf8f6f4Sm120 final : KernelScheduleMxf8f6f4Sm120, KernelTmaWarpSpecializedPingpong { };
928: // Blockwise Scaled GEMM
929: struct KernelScheduleSm120Blockwise: KernelScheduleSm120 { };
930: struct KernelTmaWarpSpecializedBlockwiseCooperativeSm120 final : KernelScheduleSm120Blockwise, KernelTmaWarpSpecializedCooperative { };
931: struct KernelTmaWarpSpecializedBlockwisePingpongSm120 final : KernelScheduleSm120Blockwise, KernelTmaWarpSpecializedPingpong { };
932: 
933: 
```
**EN:** Defines KernelTmaWarpSpecializedPingpongMxf4Sm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelTmaWarpSpecializedPingpongMxf4Sm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 934-940
```cpp
934: ///////////////////////////////////////////////////////////////////////////////////////////////////////
935: // SM120 Sparse GEMM Dispatch Policies
936: ///////////////////////////////////////////////////////////////////////////////////////////////////////
937: struct KernelScheduleSparseGemmSm120 : KernelScheduleSm120 {};
938: // Sparse GEMM: Specialize for instruction type
939: struct KernelScheduleSparseF8f6f4Sm120 final : KernelScheduleSparseGemmSm120 {};
940: 
```
**EN:** Defines KernelScheduleSparseGemmSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleSparseGemmSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 941-950
```cpp
941: ///////////////////////////////////////////////////////////////////////////////////////////////////////
942: // SM120 BlockScaled Sparse GEMM Dispatch Policies
943: ///////////////////////////////////////////////////////////////////////////////////////////////////////
944: struct KernelScheduleBlockScaledSparseGemmSm120 : KernelScheduleSm120 {};
945: struct KernelScheduleSparseMxNvf4Sm120          : KernelScheduleBlockScaledSparseGemmSm120 {};
946: struct KernelScheduleSparseMxf8f6f4Sm120        : KernelScheduleBlockScaledSparseGemmSm120 {};
947: // Block Scaled Sparse GEMM: Specialize for instruction type, scale factor vector size, Acc2x4
948: struct KernelSparseTmaWarpSpecializedNvf4Sm120           final : KernelScheduleSparseMxNvf4Sm120 { };
949: struct KernelSparseTmaWarpSpecializedMxf4Sm120           final : KernelScheduleSparseMxNvf4Sm120 { };
950: struct KernelSparseTmaWarpSpecializedMxf8f6f4Sm120       final : KernelScheduleSparseMxf8f6f4Sm120 { };
```
**EN:** Defines KernelScheduleBlockScaledSparseGemmSm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduleBlockScaledSparseGemmSm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 951-954
```cpp
951: struct KernelSparseTmaWarpSpecializedMxf8f6f4Acc2x4Sm120 final : KernelScheduleSparseMxf8f6f4Sm120, KernelScheduleAcc2x4Sm120 { };
952: 
953: //////////////////////////////////////////////////////////////////////////////
954: 
```
**EN:** Defines KernelSparseTmaWarpSpecializedMxf8f6f4Acc2x4Sm120, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSparseTmaWarpSpecializedMxf8f6f4Acc2x4Sm120，用于封装策略、存储或算法行为的辅助类型。

### Lines 955-959
```cpp
955: //
956: // Collective Mainloop Dispatch Policies
957: //
958: 
959: // n-buffer in smem, pipelined with Blackwell UMMA and CPASYNC, Warp specialized dynamic schedule
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 960-966
```cpp
960: template<
961:   int Stages_,
962:   int SchedulerPipelineStageCount_,
963:   int AccumulatorPipelineStageCount_,
964:   class ClusterShape_ = Shape<_1,_1,_1>
965: >
966: struct MainloopSm100UmmaCpAsyncWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 967-972
```cpp
967:   constexpr static int Stages = Stages_;
968:   using ClusterShape = ClusterShape_;
969:   using ArchTag = arch::Sm100;
970:   using Schedule = KernelWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
971: };
972: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 973-979
```cpp
973: template<
974:   int Stages_,
975:   int SchedulerPipelineStageCount_,
976:   int AccumulatorPipelineStageCount_,
977:   class ClusterShape_ = Shape<_1,_1,_1>
978: >
979: struct MainloopSm100UmmaMixedTmaCpAsyncWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 980-986
```cpp
980:   constexpr static int Stages = Stages_;
981:   using ClusterShape = ClusterShape_;
982:   using ArchTag = arch::Sm100;
983:   using Schedule = KernelMixedTmaCpAsyncWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
984:   constexpr static bool IsOverlappingAccum = false;
985: };
986: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 987-993
```cpp
987: template<
988:   int Stages_,
989:   int SchedulerPipelineStageCount_,
990:   int AccumulatorPipelineStageCount_,
991:   class ClusterShape_ = Shape<_1,_1,_1>
992: >
993: struct MainloopSm100UmmaMixedTmaCpAsyncWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 994-1001
```cpp
994:   constexpr static int Stages = Stages_;
995:   using ClusterShape = ClusterShape_;
996:   using ArchTag = arch::Sm100;
997:   using Schedule = KernelMixedTmaCpAsyncWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
998:   constexpr static bool IsOverlappingAccum = false;
999: };
1000: 
1001: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1002-1008
```cpp
1002: template<
1003:   int Stages_,
1004:   int SchedulerPipelineStageCount_,
1005:   int AccumulatorPipelineStageCount_,
1006:   class ClusterShape_ = Shape<_1,_1,_1>
1007: >
1008: struct MainloopSm100TmaUmmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1009-1016
```cpp
1009:   constexpr static int Stages = Stages_;
1010:   using ClusterShape = ClusterShape_;
1011:   using ArchTag = arch::Sm100;
1012:   using Schedule = KernelTmaWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1013:   constexpr static bool IsOverlappingAccum = false;
1014: };
1015: 
1016: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1017-1023
```cpp
1017: template<
1018:   int Stages_,
1019:   int SchedulerPipelineStageCount_,
1020:   int AccumulatorPipelineStageCount_,
1021:   class ClusterShape_ = Shape<_1,_1,_1>
1022: >
1023: struct MainloopSm100TmaUmmaWarpSpecializedBlockwiseScaling {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1024-1031
```cpp
1024:   constexpr static int Stages = Stages_;
1025:   using ClusterShape = ClusterShape_;
1026:   using ArchTag = arch::Sm100;
1027:   using Schedule = KernelTmaWarpSpecializedMmaTransformSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1028:   constexpr static bool IsOverlappingAccum = false;
1029: };
1030: 
1031: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1032-1038
```cpp
1032: template<
1033:   int Stages_,
1034:   int SchedulerPipelineStageCount_,
1035:   int AccumulatorPipelineStageCount_,
1036:   class ClusterShape_ = Shape<_1,_1,_1>
1037: >
1038: struct MainloopSm100ArrayTmaUmmaWarpSpecializedBlockwiseScaling {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1039-1046
```cpp
1039:   constexpr static int Stages = Stages_;
1040:   using ClusterShape = ClusterShape_;
1041:   using ArchTag = arch::Sm100;
1042:   using Schedule = KernelPtrArrayTmaWarpSpecializedMmaTransformSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1043:   constexpr static bool IsOverlappingAccum = false;
1044: };
1045: 
1046: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1047-1053
```cpp
1047: template<
1048:   int Stages_,
1049:   int SchedulerPipelineStageCount_,
1050:   int AccumulatorPipelineStageCount_,
1051:   class ClusterShape_ = Shape<_1,_1,_1>
1052: >
1053: struct MainloopSm100TmaUmmaWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1054-1060
```cpp
1054:   constexpr static int Stages = Stages_;
1055:   using ClusterShape = ClusterShape_;
1056:   using ArchTag = arch::Sm100;
1057:   constexpr static bool IsOverlappingAccum = AccumulatorPipelineStageCount_ == 1;
1058:   using Schedule = KernelTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1059: };
1060: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1061-1067
```cpp
1061: template<
1062:   int Stages_,
1063:   int SchedulerPipelineStageCount_,
1064:   int AccumulatorPipelineStageCount_,
1065:   class ClusterShape_ = Shape<_1,_1,_1>
1066: >
1067: struct MainloopSm100TmaUmmaWarpSpecializedSparse {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1068-1075
```cpp
1068:   constexpr static int Stages = Stages_;
1069:   constexpr static int MetadataS2TStages = 4;
1070:   using ClusterShape = ClusterShape_;
1071:   using ArchTag = arch::Sm100;
1072:   constexpr static bool IsOverlappingAccum = AccumulatorPipelineStageCount_ == 1;
1073:   using Schedule = KernelSparseTmaWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1074: };
1075: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1076-1082
```cpp
1076: template<
1077:   int Stages_,
1078:   int SchedulerPipelineStageCount_,
1079:   int AccumulatorPipelineStageCount_,
1080:   class ClusterShape_ = Shape<_1,_1,_1>
1081: >
1082: struct MainloopSm100TmaUmmaWarpSpecializedBlockScaledSparse {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1083-1092
```cpp
1083:   constexpr static int Stages = Stages_;
1084:   constexpr static int MetadataS2TStages = 4;
1085:   using ClusterShape = ClusterShape_;
1086:   using ArchTag = arch::Sm100;
1087:   constexpr static bool IsOverlappingAccum = AccumulatorPipelineStageCount_ == 1;
1088:   using Schedule = KernelSparseTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1089: };
1090: 
1091: // n-buffer in smem, pipelined with Blackwell Fast FP32 kernel with UMMA (HwScaled) and TMA,
1092: // Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1093-1095
```cpp
1093: template<
1094:   // Number of Pipeline stages for
1095:   // MainloopLoad <-> Conversion <-> MainLoad
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1096-1102
```cpp
1096:   int Load2TransformPipelineStageCount_,
1097:   // Number of Pipeline stages for
1098:   // MainloopLoad <-> Conversion <-> MainLoad
1099:   int Transform2MmaPipelineStageCount_,
1100:   // TileScheduler pipeline depth
1101:   int SchedulerPipelineStageCount_,
1102:   // Accmulator pipeline depth
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1103-1107
```cpp
1103:   int AccumulatorPipelineStageCount_,
1104:   // Number of MMA Bands to be computed in a single FastF32 MMA operation.
1105:   // For BF16 emulation, we have 3 compute matrices, with 9 MMAs forming 5 bands.
1106:   //    We can eliminate bands 4 and/or 5 (up to last 3 MMA operations).
1107:   //    Valid values are 3, 4, 5
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1108-1110
```cpp
1108:   int NumBandsToCompute_,
1109:   // Scaling factor for decomposed matrices (2^ScalingFactor)
1110:   // 8 for BF16, 11 for TF32
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1111-1116
```cpp
1111:   int ScalingFactor_,
1112:   // Number of UMMA instructions emulated a single stage
1113:   // Ex: Staged16 has 1 FastF32 MMA per stage
1114:   // Should be smaller than K-mode of a single ClusterTile
1115:   int AccPromotionInterval_,
1116:   // ClusterShape for the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1117-1119
```cpp
1117:   class ClusterShape_ = Shape<_1,_1,_1>,
1118:   // The TMEM_LOAD atom to be used for loading local accumulator
1119:   // from TMEM to registers
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1120-1131
```cpp
1120:   class AccumulatorCopyAtom_ = cute::SM100_TMEM_LOAD_32dp32b32x
1121: >
1122: struct MainloopSm100TmaUmmaWarpSpecializedFastF32 {
1123:   constexpr static int Load2TransformPipelineStageCount = Load2TransformPipelineStageCount_;
1124:   constexpr static int Transform2MmaPipelineStageCount = Transform2MmaPipelineStageCount_;
1125:   constexpr static int NumBandsToCompute = NumBandsToCompute_;
1126:   constexpr static int ScalingFactor = ScalingFactor_;
1127:   constexpr static int AccPromotionInterval = AccPromotionInterval_;
1128:   constexpr static detail::KernelInputTransformType InputTransformType = detail::KernelInputTransformType::FastF32;
1129:   using ClusterShape = ClusterShape_;
1130:   using AccumulatorCopyAtom = AccumulatorCopyAtom_;
1131:   using ArchTag = arch::Sm100;
```
**EN:** Defines AccumulatorCopyAtom_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 AccumulatorCopyAtom_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1132-1139
```cpp
1132:   using Schedule = KernelTmaWarpSpecializedInputTransformSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1133: 
1134:   // For backwards compatibility with GemmUniversalAdapter.
1135:   constexpr static int Stages = Load2TransformPipelineStageCount;
1136: };
1137: 
1138: 
1139: // n-buffer in smem, pipelined with Blackwell Mixed Input kernel with UMMA (HwScaled) and TMA,
```
**EN:** Introduces local type aliases (Schedule) to simplify downstream template code.
**CN:** 引入本地类型别名（Schedule），简化后续模板代码。

### Lines 1140-1142
```cpp
1140: template<
1141:   // Number of Pipeline stages for
1142:   // MainloopLoad <-> Conversion <-> MainLoad
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1143-1151
```cpp
1143:   int Load2TransformPipelineStageCount_,
1144:   // Number of Pipeline stages for
1145:   // MainloopLoad <-> Conversion <-> MainLoad
1146:   int Transform2MmaPipelineStageCount_,
1147:   // TileScheduler pipeline depth
1148:   int SchedulerPipelineStageCount_,
1149:   // Accmulator pipeline depth
1150:   int AccumulatorPipelineStageCount_,
1151:   // ClusterShape for the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1152-1162
```cpp
1152:   class ClusterShape_ = Shape<_1,_1,_1>
1153: >
1154: struct MainloopSm100TmaUmmaWarpSpecializedMixedInput {
1155:   constexpr static int Load2TransformPipelineStageCount = Load2TransformPipelineStageCount_;
1156:   constexpr static int Load2MmaPipelineStageCount = Load2TransformPipelineStageCount_;
1157:   constexpr static int Transform2MmaPipelineStageCount = Transform2MmaPipelineStageCount_;
1158:   constexpr static detail::KernelInputTransformType InputTransformType = detail::KernelInputTransformType::MixedInput;
1159:   using ClusterShape = ClusterShape_;
1160:   using ArchTag = arch::Sm100;
1161:   using Schedule = KernelTmaWarpSpecializedMixedInputTransformSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1162: 
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1163-1168
```cpp
1163:   // For backwards compatibility with GemmUniversalAdapter.
1164:   constexpr static int Stages = Load2TransformPipelineStageCount;
1165: };
1166: 
1167: 
1168: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1169-1175
```cpp
1169: template<
1170:   int Stages_,
1171:   int SchedulerPipelineStageCount_,
1172:   int AccumulatorPipelineStageCount_,
1173:   class ClusterShape_ = Shape<_1,_1,_1>
1174: >
1175: struct MainloopSm100ArrayTmaUmmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1176-1183
```cpp
1176:   constexpr static int Stages = Stages_;
1177:   using ClusterShape = ClusterShape_;
1178:   using ArchTag = arch::Sm100;
1179:   constexpr static bool IsOverlappingAccum = false;
1180:   using Schedule = KernelPtrArrayTmaWarpSpecializedSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1181: };
1182: 
1183: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1184-1190
```cpp
1184: template<
1185:   int Stages_,
1186:   int SchedulerPipelineStageCount_,
1187:   int AccumulatorPipelineStageCount_,
1188:   class ClusterShape_ = Shape<_1,_1,_1>
1189: >
1190: struct MainloopSm100ArrayTmaUmmaWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1191-1201
```cpp
1191:   constexpr static int Stages = Stages_;
1192:   using ClusterShape = ClusterShape_;
1193:   using ArchTag = arch::Sm100;
1194:   constexpr static bool IsOverlappingAccum = AccumulatorPipelineStageCount_ == 1;
1195:   using Schedule = KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1196: };
1197: 
1198: 
1199: 
1200: // n-buffer in smem, pipelined with Blackwell Fast FP32 kernel with UMMA (HwScaled) and TMA,
1201: // Warp specialized dynamic schedule
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1202-1204
```cpp
1202: template<
1203:   // Number of Pipeline stages for
1204:   // MainloopLoad <-> Conversion <-> MainLoad
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1205-1211
```cpp
1205:   int Load2TransformPipelineStageCount_,
1206:   // Number of Pipeline stages for
1207:   // MainloopLoad <-> Conversion <-> MainLoad
1208:   int Transform2MmaPipelineStageCount_,
1209:   // TileScheduler pipeline depth
1210:   int SchedulerPipelineStageCount_,
1211:   // Accmulator pipeline depth
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1212-1216
```cpp
1212:   int AccumulatorPipelineStageCount_,
1213:   // Number of MMA Bands to be computed in a single FastF32 MMA operation.
1214:   // For BF16 emulation, we have 3 compute matrices, with 9 MMAs forming 5 bands.
1215:   //    We can eliminate bands 4 and/or 5 (up to last 3 MMA operations).
1216:   //    Valid values are 3, 4, 5
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1217-1219
```cpp
1217:   int NumBandsToCompute_,
1218:   // Scaling factor for decomposed matrices (2^ScalingFactor)
1219:   // 8 for BF16, 11 for TF32
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1220-1225
```cpp
1220:   int ScalingFactor_,
1221:   // Number of UMMA instructions emulated a single stage
1222:   // Ex: Staged16 has 1 FastF32 MMA per stage
1223:   // Should be smaller than K-mode of a single ClusterTile
1224:   int AccPromotionInterval_,
1225:   // ClusterShape for the kernel
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1226-1228
```cpp
1226:   class ClusterShape_ = Shape<_1,_1,_1>,
1227:   // The TMEM_LOAD atom to be used for loading local accumulator
1228:   // from TMEM to registers
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1229-1240
```cpp
1229:   class AccumulatorCopyAtom_ = cute::SM100_TMEM_LOAD_32dp32b32x
1230: >
1231: struct MainloopSm100ArrayTmaUmmaWarpSpecializedFastF32 {
1232:   constexpr static int Load2TransformPipelineStageCount = Load2TransformPipelineStageCount_;
1233:   constexpr static int Transform2MmaPipelineStageCount = Transform2MmaPipelineStageCount_;
1234:   constexpr static int NumBandsToCompute = NumBandsToCompute_;
1235:   constexpr static int ScalingFactor = ScalingFactor_;
1236:   constexpr static int AccPromotionInterval = AccPromotionInterval_;
1237:   constexpr static detail::KernelInputTransformType InputTransformType = detail::KernelInputTransformType::FastF32;
1238:   using ClusterShape = ClusterShape_;
1239:   using AccumulatorCopyAtom = AccumulatorCopyAtom_;
1240:   using ArchTag = arch::Sm100;
```
**EN:** Defines AccumulatorCopyAtom_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 AccumulatorCopyAtom_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1241-1249
```cpp
1241:   using Schedule = KernelPtrArrayTmaWarpSpecializedInputTransformSm100<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1242: 
1243:   // For backwards compatibility with GemmUniversalAdapter.
1244:   constexpr static int Stages = Load2TransformPipelineStageCount;
1245: };
1246: 
1247: 
1248: #if defined(SYCL_INTEL_TARGET)
1249: 
```
**EN:** Introduces local type aliases (Schedule) to simplify downstream template code.
**CN:** 引入本地类型别名（Schedule），简化后续模板代码。

### Lines 1250-1252
```cpp
1250: // Specialization of the GEMM mainloop for Intel Xe architectures.
1251: // This version is tuned for operations using DPAS instructions with a subgroup size of 16.
1252: // Suitable for use with Intel Battlemage (Xe2) and PVC (Xe) architectures.
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1253-1261
```cpp
1253: template<int Stages_, class KernelSchedule = KernelXe>
1254: struct MainloopIntelXeXMX16 {
1255:   constexpr static int Stages = Stages_;
1256:   constexpr static int SubgroupSize = 16;
1257:   using ArchTag = arch::IntelXe;
1258:   using Schedule = KernelSchedule;
1259:   using ClusterShape = Shape<_1,_1,_1>;
1260: };
1261: 
```
**EN:** Defines KernelSchedule, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedule，用于封装策略、存储或算法行为的辅助类型。

### Lines 1262-1265
```cpp
1262: template<int Stages_, class KernelScheduler = KernelXePtrArrayCooperative>
1263: struct MainloopIntelXeXMX16Group : MainloopIntelXeXMX16<Stages_, KernelScheduler> {
1264: };
1265: 
```
**EN:** Defines KernelScheduler, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduler，用于封装策略、存储或算法行为的辅助类型。

### Lines 1266-1269
```cpp
1266: template<int Stages_, class KernelScheduler = KernelXePtrArrayCooperative>
1267: struct MainloopXeL1StagedGroup : MainloopIntelXeXMX16<Stages_, KernelScheduler> {
1268: };
1269: 
```
**EN:** Defines KernelScheduler, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduler，用于封装策略、存储或算法行为的辅助类型。

### Lines 1270-1273
```cpp
1270: template<int Stages_, class KernelScheduler = KernelXePtrArrayCooperative>
1271: struct MainloopIntelXeXMX16GroupMixedPrecision : MainloopIntelXeXMX16<Stages_, KernelScheduler> {
1272: };
1273: 
```
**EN:** Defines KernelScheduler, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduler，用于封装策略、存储或算法行为的辅助类型。

### Lines 1274-1277
```cpp
1274: template<int Stages_, class KernelSchedule = KernelXe>
1275: struct MainloopIntelXeXMX16MixedPrecision : MainloopIntelXeXMX16<Stages_, KernelSchedule> {
1276: };
1277: 
```
**EN:** Defines KernelSchedule, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedule，用于封装策略、存储或算法行为的辅助类型。

### Lines 1278-1281
```cpp
1278: template<int Stages_, class KernelSchedule = KernelXe>
1279: struct MainloopIntelW8A8 : MainloopIntelXeXMX16<Stages_, KernelSchedule> {
1280: };
1281: 
```
**EN:** Defines KernelSchedule, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedule，用于封装策略、存储或算法行为的辅助类型。

### Lines 1282-1285
```cpp
1282: template<int Stages_, class KernelScheduler = KernelXePtrArrayCooperative>
1283: struct MainloopIntelXeXMX16GroupFP8 : MainloopIntelXeXMX16<Stages_, KernelScheduler> {
1284: };
1285: 
```
**EN:** Defines KernelScheduler, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelScheduler，用于封装策略、存储或算法行为的辅助类型。

### Lines 1286-1291
```cpp
1286: template<int Stages_>
1287: struct MainloopIntelXeXMX16FP8Scaling : MainloopIntelXeXMX16<Stages_> {
1288: };
1289: 
1290: #endif
1291: 
```
**EN:** Defines MainloopIntelXeXMX16FP8Scaling, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MainloopIntelXeXMX16FP8Scaling，用于封装策略、存储或算法行为的辅助类型。

### Lines 1292-1299
```cpp
1292: #if defined(CUTLASS_ENABLE_SYCL)
1293: struct MainloopDeviceAgnostic {
1294:   using ArchTag = arch::Agnostic;
1295:   using ClusterShape = Shape<_1,_1,_1>;
1296:   using Schedule = KernelMultistage;
1297: };
1298: #endif
1299: 
```
**EN:** Defines MainloopDeviceAgnostic, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 MainloopDeviceAgnostic，用于封装策略、存储或算法行为的辅助类型。

### Lines 1300-1302
```cpp
1300: #if defined(CUTLASS_ENABLE_SYCL) 
1301: // Note: This dispatch policy is specifically added for CollectiveMma to support
1302: // the integration of new MMA atoms (XE_DPAS_TT) and copy atoms for Intel XE architecture
```
**EN:** Applies conditional logic to select architecture paths, handle bounds, or guard special cases.
**CN:** 应用条件逻辑以选择架构路径、处理边界或保护特殊情况。

### Lines 1303-1312
```cpp
1303: template<int Stages_, class KernelSchedule = KernelXe>
1304: struct MainloopXeL1Staged {
1305:   constexpr static int Stages = Stages_;
1306:   constexpr static int SubgroupSize = 16;
1307:   using ArchTag = arch::IntelXe;
1308:   using Schedule = KernelSchedule;
1309:   using ClusterShape = Shape<_1,_1,_1>;
1310: };
1311: #endif
1312: 
```
**EN:** Defines KernelSchedule, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 KernelSchedule，用于封装策略、存储或算法行为的辅助类型。

### Lines 1313-1313
```cpp
1313: // n-buffer in smem, pipelined with Blackwell UMMA and TMA, Warp specialized dynamic schedule
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1314-1322
```cpp
1314: template<
1315:   int LoadABPipelineStageCount_,
1316:   int LoadSFPipelineStageCount_,
1317:   int SchedulerPipelineStageCount_,
1318:   int AccumulatorPipelineStageCount_,
1319:   class ClusterShape_ = Shape<_1,_1,_1>,
1320:   cutlass::sm103::detail::KernelPrefetchType PrefetchType_ = cutlass::sm103::detail::KernelPrefetchType::TmaPrefetch
1321: >
1322: struct MainloopSm103TmaUmmaWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1323-1329
```cpp
1323:   constexpr static int LoadABPipelineStageCount = LoadABPipelineStageCount_;
1324:   constexpr static int LoadSFPipelineStageCount = LoadSFPipelineStageCount_;
1325:   using ClusterShape = ClusterShape_;
1326:   using ArchTag = arch::Sm103;
1327:   constexpr static bool IsOverlappingAccum = AccumulatorPipelineStageCount_ == 1;
1328:   using Schedule = KernelTmaWarpSpecializedBlockScaledSm103<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1329:   // For backwards compatibility with GemmUniversalAdapter.
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1330-1335
```cpp
1330:   constexpr static int Stages = LoadABPipelineStageCount;
1331:   constexpr static cutlass::sm103::detail::KernelPrefetchType PrefetchType = PrefetchType_;
1332: };
1333: 
1334: // Mainloop schedule for array-based TMA
1335: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1336-1344
```cpp
1336: template<
1337:   int LoadABPipelineStageCount_,
1338:   int LoadSFPipelineStageCount_,
1339:   int SchedulerPipelineStageCount_,
1340:   int AccumulatorPipelineStageCount_,
1341:   class ClusterShape_ = Shape<_1,_1,_1>,
1342:   cutlass::sm103::detail::KernelPrefetchType PrefetchType_ = cutlass::sm103::detail::KernelPrefetchType::TmaPrefetch
1343: >
1344: struct MainloopSm103ArrayTmaUmmaWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1345-1351
```cpp
1345:   constexpr static int LoadABPipelineStageCount = LoadABPipelineStageCount_;
1346:   constexpr static int LoadSFPipelineStageCount = LoadSFPipelineStageCount_;
1347:   using ClusterShape = ClusterShape_;
1348:   using ArchTag = arch::Sm103;
1349:   constexpr static bool IsOverlappingAccum = AccumulatorPipelineStageCount_ == 1;
1350:   using Schedule = KernelPtrArrayTmaWarpSpecializedBlockScaledSm103<SchedulerPipelineStageCount_, AccumulatorPipelineStageCount_>;
1351:   // For backwards compatibility with GemmUniversalAdapter.
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1352-1355
```cpp
1352:   constexpr static int Stages = LoadABPipelineStageCount;
1353:   constexpr static cutlass::sm103::detail::KernelPrefetchType PrefetchType = PrefetchType_;
1354: };
1355: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1356-1362
```cpp
1356: template<
1357:   int Stages_,
1358:   int SchedulerPipelineStageCount_,
1359:   class ClusterShape_,
1360:   class KernelSchedule_
1361: >
1362: struct MainloopSm120TmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1363-1369
```cpp
1363:   constexpr static int Stages = Stages_;
1364:   using ClusterShape = ClusterShape_;
1365:   using Schedule = KernelSchedule_;
1366:   constexpr static int PipelineAsyncMmaStages = 0;
1367:   using ArchTag = arch::Sm120;
1368: };
1369: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1370-1376
```cpp
1370: template<
1371:   int Stages_,
1372:   int SchedulerPipelineStageCount_,
1373:   class ClusterShape_,
1374:   class KernelSchedule_
1375: >
1376: struct MainloopSm120ArrayTmaWarpSpecialized {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1377-1386
```cpp
1377:   constexpr static int Stages = Stages_;
1378:   using ClusterShape = ClusterShape_;
1379:   using Schedule = KernelSchedule_;
1380:   constexpr static int PipelineAsyncMmaStages = 0;
1381:   using ArchTag = arch::Sm120;
1382:   static_assert(
1383:     cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, Schedule> ||
1384:     cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedPingpong, Schedule>,
1385:     "KernelSchedule must be one of the Ptr-Array or Grouped Gemm TMA Warp Specialized Cooperative or Pingpong policies");
1386: };
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1387-1388
```cpp
1387: 
1388: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 1389-1395
```cpp
1389: template<
1390:   int Stages_,
1391:   int SchedulerPipelineStageCount_,
1392:   class ClusterShape_,
1393:   class KernelSchedule_
1394: >
1395: struct MainloopSm120TmaWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1396-1403
```cpp
1396:   constexpr static int Stages = Stages_;
1397:   constexpr static int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
1398:   using ClusterShape = ClusterShape_;
1399:   using Schedule = KernelSchedule_;
1400:   constexpr static int PipelineAsyncMmaStages = 0;
1401:   using ArchTag = arch::Sm120;
1402: };
1403: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1404-1410
```cpp
1404: template<
1405:   int Stages_,
1406:   int SchedulerPipelineStageCount_,
1407:   class ClusterShape_,
1408:   class KernelSchedule_
1409: >
1410: struct MainloopSm120ArrayTmaWarpSpecializedBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1411-1417
```cpp
1411:   constexpr static int Stages = Stages_;
1412:   constexpr static int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
1413:   using ClusterShape = ClusterShape_;
1414:   constexpr static int PipelineAsyncMmaStages = 0;
1415:   using Schedule = KernelSchedule_;
1416:   using ArchTag = arch::Sm120;
1417: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1418-1423
```cpp
1418:   static_assert(cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, Schedule> ||
1419:                 cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedPingpong, Schedule>,
1420:                 "KernelSchedule must be one of the Ptr-Array or Grouped Gemm TMA Warp Specialized Cooperative or Pingpong policies.");
1421: };
1422: 
1423: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 1424-1431
```cpp
1424: template<
1425:   int StagesA_,
1426:   int StagesB_,
1427:   int StagesE_,
1428:   int SchedulerPipelineStageCount_,
1429:   class ClusterShape_ = Shape<_1,_1,_1>
1430: >
1431: struct MainloopSm120TmaWarpSpecializedSparse {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1432-1440
```cpp
1432:   constexpr static int StagesA = StagesA_;
1433:   constexpr static int StagesB = StagesB_;
1434:   constexpr static int StagesE = StagesE_;
1435:   constexpr static bool isAsymmetric = (StagesA != StagesB);
1436:   using ClusterShape = ClusterShape_;
1437:   using ArchTag = arch::Sm120;
1438:   using Schedule = KernelTmaWarpSpecializedCooperativeSparseSm120<SchedulerPipelineStageCount_, isAsymmetric>;
1439: };
1440: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1441-1448
```cpp
1441: template<
1442:   int StagesA_,
1443:   int StagesB_,
1444:   int StagesE_,
1445:   int SchedulerPipelineStageCount_,
1446:   class ClusterShape_ = Shape<_1,_1,_1>
1447: >
1448: struct MainloopSm120TmaWarpSpecializedSparseBlockScaled {
```
**EN:** Defines ClusterShape_, a helper type used to package policy, storage, or algorithm behavior.
**CN:** 定义 ClusterShape_，用于封装策略、存储或算法行为的辅助类型。

### Lines 1449-1457
```cpp
1449:   constexpr static int StagesA = StagesA_;
1450:   constexpr static int StagesB = StagesB_;
1451:   constexpr static int StagesE = StagesE_;
1452:   constexpr static bool isAsymmetric = (StagesA != StagesB);
1453:   using ClusterShape = ClusterShape_;
1454:   using ArchTag = arch::Sm120;
1455:   using Schedule = KernelTmaWarpSpecializedCooperativeSparseBlockScaledSm120<SchedulerPipelineStageCount_, isAsymmetric>;
1456: };
1457: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1458-1464
```cpp
1458: template <
1459:   int Stages_,
1460:   int SchedulerPipelineStageCount_,
1461:   class ClusterShape_,
1462:   class KernelSchedule_
1463: >
1464: struct MainloopSm120TmaWarpSpecializedBlockwiseScaling {
```
**EN:** Declares template parameters and begins the definition of ClusterShape_.
**CN:** 声明模板参数并开始定义 ClusterShape_。

### Lines 1465-1469
```cpp
1465:   constexpr static int Stages = Stages_;
1466:   constexpr static int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
1467:   using ClusterShape = ClusterShape_;
1468:   using Schedule = KernelSchedule_;
1469: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1470-1473
```cpp
1470:   constexpr static int PipelineAsyncMmaStages = 0;
1471:   using ArchTag = arch::Sm120;
1472: };
1473: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1474-1480
```cpp
1474: template <
1475:   int Stages_,
1476:   int SchedulerPipelineStageCount_,
1477:   class ClusterShape_,
1478:   class KernelSchedule_
1479: >
1480: struct MainloopSm120ArrayTmaWarpSpecializedBlockwiseScaling {
```
**EN:** Declares template parameters and begins the definition of ClusterShape_.
**CN:** 声明模板参数并开始定义 ClusterShape_。

### Lines 1481-1485
```cpp
1481:   constexpr static int Stages = Stages_;
1482:   constexpr static int SchedulerPipelineStageCount = SchedulerPipelineStageCount_;
1483:   using ClusterShape = ClusterShape_;
1484:   using Schedule = KernelSchedule_;
1485: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1486-1488
```cpp
1486:   constexpr static int PipelineAsyncMmaStages = 0;
1487:   using ArchTag = arch::Sm120;
1488: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 1489-1498
```cpp
1489:   static_assert(cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, Schedule> ||
1490:                 cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedPingpong, Schedule>, 
1491:                 "KernelSchedule must be one of the Ptr-Array or Grouped Gemm TMA Warp Specialized Cooperative or Pingpong policies.");
1492: };
1493: 
1494: 
1495: 
1496: //////////////////////////////////////////////////////////////////////////////
1497: 
1498: } // namespace cutlass::gemm
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

## Key Concepts / 关键概念

- **EN:** GEMM tiling and matrix multiplication  
  **CN:** GEMM 分块与矩阵乘法
- **EN:** SIMT execution policy  
  **CN:** SIMT 执行策略
- **EN:** Sparse operand handling  
  **CN:** 稀疏操作数处理
- **EN:** Asynchronous shared-memory staging  
  **CN:** 异步共享内存预取
- **EN:** Software pipelining across K iterations  
  **CN:** 跨 K 迭代的软件流水线
- **EN:** Multi-stage mainloop buffering  
  **CN:** 多阶段主循环缓冲

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `T`, `U`, `is_kernel_tag_of`, `is_asymmetric_dma_kernel_tag_of`, `KernelInputTransformType`, `Has_SwapAB`, `HasAuxiliaryLoad`, `tags`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
