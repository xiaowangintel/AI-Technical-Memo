# default_gemm_configuration.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/device/default_gemm_configuration.h`
- **Purpose (EN):** Defines default configuration helpers for selecting CUTLASS kernels.
- **用途 (CN):** 定义用于选择 CUTLASS 内核的默认配置辅助模板。

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
32:     \brief Definitions for GEMM structures
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

### Lines 37-42
```cpp
37: #include "cutlass/cutlass.h"
38: #include "cutlass/numeric_types.h"
39: #include "cutlass/arch/arch.h"
40: #include "cutlass/arch/mma.h"
41: #include "cutlass/arch/wmma.h"
42: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities, numeric types/converters, architecture intrinsics.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具、数值类型/转换器、架构内建/指令封装。

### Lines 43-52
```cpp
43: #include "cutlass/gemm/gemm.h"
44: #include "cutlass/epilogue/thread/linear_combination.h"
45: #include "cutlass/epilogue/thread/linear_combination_clamp.h"
46: 
47: ////////////////////////////////////////////////////////////////////////////////
48: 
49: namespace cutlass {
50: namespace gemm {
51: namespace device {
52: 
```
**EN:** Pulls in required dependencies such as core CUTLASS utilities.
**CN:** 引入所需依赖，例如 CUTLASS 基础工具。

### Lines 53-54
```cpp
53: ////////////////////////////////////////////////////////////////////////////////
54: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 55-64
```cpp
55: template <
56:   typename OperatorClass,
57:   typename ArchTag,
58:   typename ElementA, 
59:   typename ElementB, 
60:   typename ElementC,
61:   typename ElementAccumulator
62: >
63: struct DefaultGemmConfiguration;
64: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 65-66
```cpp
65: ////////////////////////////////////////////////////////////////////////////////
66: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 67-78
```cpp
67: template <
68:   typename ArchTag,
69:   typename ElementA, 
70:   typename ElementB, 
71:   typename ElementC, 
72:   typename ElementAccumulator>
73: struct DefaultGemmConfiguration<
74:   arch::OpClassSimt, 
75:   ArchTag,
76:   ElementA, 
77:   ElementB, 
78:   ElementC, 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 79-80
```cpp
79:   ElementAccumulator> {
80:   
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 81-87
```cpp
81:   static int const kAlignmentA = 1;
82:   static int const kAlignmentB = 1;
83:   using ThreadblockShape = GemmShape<128, 128, 8>;
84:   using WarpShape = GemmShape<32, 64, 8>;
85:   using InstructionShape = GemmShape<1, 1, 1>;
86:   static int const kStages = 2;
87: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 88-97
```cpp
88:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
89:     ElementC,
90:     1,
91:     ElementAccumulator,
92:     ElementAccumulator
93:   >;
94: 
95:   using Operator = arch::OpMultiplyAdd;
96: };
97: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 98-99
```cpp
98: ////////////////////////////////////////////////////////////////////////////////
99: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 100-104
```cpp
100: template < 
101:   typename ArchTag,
102:   typename ElementC>
103: struct DefaultGemmConfiguration<arch::OpClassSimt, ArchTag, int8_t, int8_t, ElementC, int32_t> {
104:   
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 105-111
```cpp
105:   static int const kAlignmentA = 4;
106:   static int const kAlignmentB = 4;
107:   using ThreadblockShape = GemmShape<128, 128, 32>;
108:   using WarpShape = GemmShape<32, 64, 32>;
109:   using InstructionShape = GemmShape<1, 1, 4>;
110:   static int const kStages = 2;
111: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 112-121
```cpp
112:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
113:     ElementC,
114:     1,
115:     int32_t,
116:     float
117:   >;
118: 
119:   using Operator = arch::OpMultiplyAdd;
120: };
121: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 122-123
```cpp
122: ////////////////////////////////////////////////////////////////////////////////
123: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 124-135
```cpp
124: template <
125:   typename ArchTag,
126:   typename ElementA, 
127:   typename ElementB, 
128:   typename ElementC, 
129:   typename ElementAccumulator>
130: struct DefaultGemmConfiguration<
131:   arch::OpClassWmmaTensorOp, 
132:   ArchTag,
133:   ElementA, 
134:   ElementB, 
135:   ElementC, 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 136-137
```cpp
136:   ElementAccumulator> {
137:   
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 138-142
```cpp
138:   static int const kAlignmentA = 128 / sizeof_bits<ElementA>::value;
139:   static int const kAlignmentB = 128 / sizeof_bits<ElementB>::value;
140: 
141:   static int const kStages = 2;
142:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 143-152
```cpp
143:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
144:     ElementC,
145:     128 / sizeof_bits<ElementC>::value,
146:     ElementAccumulator,
147:     ElementAccumulator
148:   >;
149: 
150:   using Operator = arch::OpMultiplyAdd;
151: };
152: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 153-154
```cpp
153: ////////////////////////////////////////////////////////////////////////////////
154: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 155-166
```cpp
155: template <
156:   typename ElementA, 
157:   typename ElementB, 
158:   typename ElementC, 
159:   typename ElementAccumulator>
160: struct DefaultGemmConfiguration<
161:   arch::OpClassTensorOp, 
162:   arch::Sm70,
163:   ElementA, 
164:   ElementB, 
165:   ElementC, 
166:   ElementAccumulator> {
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 167-167
```cpp
167:   
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 168-170
```cpp
168:   static int const kAlignmentA = 128 / sizeof_bits<ElementA>::value;
169:   static int const kAlignmentB = 128 / sizeof_bits<ElementB>::value;
170: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 171-175
```cpp
171:   using ThreadblockShape = GemmShape<128, 256, 32>;
172:   using WarpShape = GemmShape<64, 64, 32>;
173:   using InstructionShape = GemmShape<8, 8, 4>;
174:   static int const kStages = 2;
175:   
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 176-185
```cpp
176:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
177:     ElementC,
178:     128 / sizeof_bits<ElementC>::value,
179:     ElementAccumulator,
180:     ElementAccumulator
181:   >;
182: 
183:   using Operator = arch::OpMultiplyAdd;
184: };
185: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 186-187
```cpp
186: ////////////////////////////////////////////////////////////////////////////////
187: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 188-199
```cpp
188: template <
189:   typename ElementA, 
190:   typename ElementB, 
191:   typename ElementC, 
192:   typename ElementAccumulator>
193: struct DefaultGemmConfiguration<
194:   arch::OpClassTensorOp, 
195:   arch::Sm75,
196:   ElementA, 
197:   ElementB, 
198:   ElementC, 
199:   ElementAccumulator> {
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 200-200
```cpp
200: 
```
**EN:** Separates logical regions and improves readability.
**CN:** 分隔逻辑区域并提升可读性。

### Lines 201-207
```cpp
201:   static int const kAlignmentA = 128 / sizeof_bits<ElementA>::value;
202:   static int const kAlignmentB = 128 / sizeof_bits<ElementA>::value;
203:   using ThreadblockShape = GemmShape<128, 256, 32>;
204:   using WarpShape = GemmShape<64, 64, 32>;
205:   using InstructionShape = GemmShape<16, 8, 8>;
206:   static int const kStages = 2;
207: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 208-214
```cpp
208:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
209:     ElementC,
210:     128 / sizeof_bits<ElementC>::value,
211:     ElementAccumulator,
212:     ElementAccumulator
213:   >;
214: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp），简化后续模板代码。

### Lines 215-224
```cpp
215:   using Operator = typename platform::conditional<
216:       (platform::is_same<ElementA, int8_t>::value ||
217:        platform::is_same<ElementA, int4b_t>::value ||
218:        platform::is_same<ElementA, uint8_t>::value ||
219:        platform::is_same<ElementA, uint4b_t>::value),
220:       arch::OpMultiplyAddSaturate, arch::OpMultiplyAdd>::type;
221: };
222: 
223: ////////////////////////////////////////////////////////////////////////////////
224: 
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 225-234
```cpp
225: template < 
226:   typename ElementC>
227: struct DefaultGemmConfiguration<
228:   arch::OpClassTensorOp, 
229:   arch::Sm75, 
230:   int8_t, 
231:   int8_t, 
232:   ElementC, 
233:   int32_t> {
234:   
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 235-237
```cpp
235:   static int const kAlignmentA = 128 / sizeof_bits<int8_t>::value;
236:   static int const kAlignmentB = 128 / sizeof_bits<int8_t>::value;
237: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 238-242
```cpp
238:   using ThreadblockShape = GemmShape<128, 256, 64>;
239:   using WarpShape = GemmShape<64, 64, 64>;
240:   using InstructionShape = GemmShape<8, 8, 16>;
241:   static int const kStages = 2;
242: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 243-250
```cpp
243:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
244:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
245: 
246:   using Operator = arch::OpMultiplyAddSaturate;
247: };
248: 
249: ////////////////////////////////////////////////////////////////////////////////
250: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 251-260
```cpp
251: template < 
252:   typename ElementC>
253: struct DefaultGemmConfiguration<
254:   arch::OpClassTensorOp, 
255:   arch::Sm75, 
256:   int8_t, 
257:   uint8_t, 
258:   ElementC, 
259:   int32_t> {
260:   
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 261-263
```cpp
261:   static int const kAlignmentA = 128 / sizeof_bits<int8_t>::value;
262:   static int const kAlignmentB = 128 / sizeof_bits<uint8_t>::value;
263:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 264-268
```cpp
264:   using ThreadblockShape = GemmShape<128, 256, 64>;
265:   using WarpShape = GemmShape<64, 64, 64>;
266:   using InstructionShape = GemmShape<8, 8, 16>;
267:   static int const kStages = 2;
268: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 269-276
```cpp
269:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
270:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
271: 
272:   using Operator = arch::OpMultiplyAddSaturate;
273: };
274: 
275: ////////////////////////////////////////////////////////////////////////////////
276: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 277-286
```cpp
277: template < 
278:   typename ElementC>
279: struct DefaultGemmConfiguration<
280:   arch::OpClassTensorOp, 
281:   arch::Sm75, 
282:   uint8_t, 
283:   int8_t, 
284:   ElementC, 
285:   int32_t> {
286:   
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 287-289
```cpp
287:   static int const kAlignmentA = 128 / sizeof_bits<uint8_t>::value;
288:   static int const kAlignmentB = 128 / sizeof_bits<int8_t>::value;
289:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 290-294
```cpp
290:   using ThreadblockShape = GemmShape<128, 256, 64>;
291:   using WarpShape = GemmShape<64, 64, 64>;
292:   using InstructionShape = GemmShape<8, 8, 16>;
293:   static int const kStages = 2;
294: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 295-302
```cpp
295:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
296:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
297: 
298:   using Operator = arch::OpMultiplyAddSaturate;
299: };
300: 
301: ////////////////////////////////////////////////////////////////////////////////
302: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 303-312
```cpp
303: template < 
304:   typename ElementC>
305: struct DefaultGemmConfiguration<
306:   arch::OpClassTensorOp, 
307:   arch::Sm75, 
308:   uint8_t, 
309:   uint8_t, 
310:   ElementC, 
311:   int32_t> {
312:   
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 313-315
```cpp
313:   static int const kAlignmentA = 128 / sizeof_bits<uint8_t>::value;
314:   static int const kAlignmentB = 128 / sizeof_bits<uint8_t>::value;
315:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 316-320
```cpp
316:   using ThreadblockShape = GemmShape<128, 256, 64>;
317:   using WarpShape = GemmShape<64, 64, 64>;
318:   using InstructionShape = GemmShape<8, 8, 16>;
319:   static int const kStages = 2;
320: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 321-328
```cpp
321:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
322:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
323: 
324:   using Operator = arch::OpMultiplyAddSaturate;
325: };
326: 
327: ////////////////////////////////////////////////////////////////////////////////
328: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 329-338
```cpp
329: template < 
330:   typename ElementC>
331: struct DefaultGemmConfiguration<
332:   arch::OpClassTensorOp, 
333:   arch::Sm75, 
334:   int4b_t, 
335:   int4b_t, 
336:   ElementC, 
337:   int32_t> {
338:    
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 339-341
```cpp
339:   static int const kAlignmentA = 128 / sizeof_bits<int4b_t>::value;
340:   static int const kAlignmentB = 128 / sizeof_bits<int4b_t>::value;
341:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 342-346
```cpp
342:   using ThreadblockShape = GemmShape<128, 256, 128>;
343:   using WarpShape = GemmShape<64, 64, 128>;
344:   using InstructionShape = GemmShape<8, 8, 32>;
345:   static int const kStages = 2;
346: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 347-354
```cpp
347:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
348:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
349: 
350:   using Operator = arch::OpMultiplyAddSaturate;
351: };
352: 
353: ////////////////////////////////////////////////////////////////////////////////
354: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 355-364
```cpp
355: template < 
356:   typename ElementC>
357: struct DefaultGemmConfiguration<
358:   arch::OpClassTensorOp, 
359:   arch::Sm75, 
360:   int4b_t, 
361:   uint4b_t, 
362:   ElementC, 
363:   int32_t> {
364:     
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 365-367
```cpp
365:   static int const kAlignmentA = 128 / sizeof_bits<int4b_t>::value;
366:   static int const kAlignmentB = 128 / sizeof_bits<uint4b_t>::value;
367:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 368-372
```cpp
368:   using ThreadblockShape = GemmShape<128, 256, 128>;
369:   using WarpShape = GemmShape<64, 64, 128>;
370:   using InstructionShape = GemmShape<8, 8, 32>;
371:   static int const kStages = 2;
372: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 373-380
```cpp
373:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
374:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
375: 
376:   using Operator = arch::OpMultiplyAddSaturate;
377: };
378: 
379: ////////////////////////////////////////////////////////////////////////////////
380: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 381-390
```cpp
381: template < 
382:   typename ElementC>
383: struct DefaultGemmConfiguration<
384:   arch::OpClassTensorOp, 
385:   arch::Sm75, 
386:   uint4b_t, 
387:   int4b_t, 
388:   ElementC, 
389:   int32_t> {
390:   
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 391-393
```cpp
391:   static int const kAlignmentA = 128 / sizeof_bits<uint4b_t>::value;
392:   static int const kAlignmentB = 128 / sizeof_bits<int4b_t>::value;
393: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 394-398
```cpp
394:   using ThreadblockShape = GemmShape<128, 256, 128>;
395:   using WarpShape = GemmShape<64, 64, 128>;
396:   using InstructionShape = GemmShape<8, 8, 32>;
397:   static int const kStages = 2;
398: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 399-406
```cpp
399:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
400:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
401: 
402:   using Operator = arch::OpMultiplyAddSaturate;
403: };
404: 
405: ////////////////////////////////////////////////////////////////////////////////
406: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 407-416
```cpp
407: template < 
408:   typename ElementC>
409: struct DefaultGemmConfiguration<
410:   arch::OpClassTensorOp, 
411:   arch::Sm75, 
412:   uint4b_t, 
413:   uint4b_t, 
414:   ElementC, 
415:   int32_t> {
416:    
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 417-419
```cpp
417:   static int const kAlignmentA = 128 / sizeof_bits<uint4b_t>::value;
418:   static int const kAlignmentB = 128 / sizeof_bits<uint4b_t>::value;
419:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 420-424
```cpp
420:   using ThreadblockShape = GemmShape<128, 256, 128>;
421:   using WarpShape = GemmShape<64, 64, 128>;
422:   using InstructionShape = GemmShape<8, 8, 32>;
423:   static int const kStages = 2;
424: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 425-432
```cpp
425:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
426:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
427: 
428:   using Operator = arch::OpMultiplyAddSaturate;
429: };
430: 
431: ////////////////////////////////////////////////////////////////////////////////
432: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 433-442
```cpp
433: template < 
434:   typename ElementC>
435: struct DefaultGemmConfiguration<
436:   arch::OpClassTensorOp, 
437:   arch::Sm75, 
438:   uint1b_t, 
439:   uint1b_t, 
440:   ElementC, 
441:   int32_t> {
442:     
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 443-445
```cpp
443:   static int const kAlignmentA = 128 / sizeof_bits<uint1b_t>::value;
444:   static int const kAlignmentB = 128 / sizeof_bits<uint1b_t>::value;
445:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 446-450
```cpp
446:   using ThreadblockShape = GemmShape<128, 256, 512>;
447:   using WarpShape = GemmShape<64, 64, 512>;
448:   using InstructionShape = GemmShape<8, 8, 128>;
449:   static int const kStages = 2;
450: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 451-458
```cpp
451:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
452:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
453: 
454:   using Operator = arch::OpXorPopc;
455: };
456: 
457: ////////////////////////////////////////////////////////////////////////////////
458: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 459-463
```cpp
459: template <typename ElementA, typename ElementB, typename ElementC,
460:           typename ElementAccumulator>
461: struct DefaultGemmConfiguration<arch::OpClassTensorOp, arch::Sm80, ElementA,
462:                                 ElementB, ElementC, ElementAccumulator> {
463: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 464-466
```cpp
464:   static int const kAlignmentA = 128 / sizeof_bits<ElementA>::value;
465:   static int const kAlignmentB = 128 / sizeof_bits<ElementA>::value;
466:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 467-471
```cpp
467:   using ThreadblockShape = GemmShape<128, 256, 64>;
468:   using WarpShape = GemmShape<64, 64, 64>;
469:   using InstructionShape = GemmShape<16, 8, 16>;
470:   static int const kStages = 3;
471: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 472-475
```cpp
472:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
473:       ElementC, 128 / sizeof_bits<ElementC>::value, ElementAccumulator,
474:       ElementAccumulator>;
475: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp），简化后续模板代码。

### Lines 476-484
```cpp
476:   using Operator = typename platform::conditional<
477:       (platform::is_same<ElementA, int8_t>::value ||
478:        platform::is_same<ElementA, int4b_t>::value ||
479:        platform::is_same<ElementA, uint8_t>::value ||
480:        platform::is_same<ElementA, uint4b_t>::value),
481:       arch::OpMultiplyAddSaturate, arch::OpMultiplyAdd>::type;
482: };
483: 
484: ////////////////////////////////////////////////////////////////////////////////
```
**EN:** Introduces local type aliases (Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（Operator），简化后续模板代码。

### Lines 485-489
```cpp
485: template <typename ElementC,
486:           typename ElementAccumulator>
487: struct DefaultGemmConfiguration<arch::OpClassTensorOp, arch::Sm80, double,
488:                                 double, ElementC, ElementAccumulator> {
489: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 490-492
```cpp
490:   static int const kAlignmentA = 1;
491:   static int const kAlignmentB = 1;
492:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 493-497
```cpp
493:   using ThreadblockShape = GemmShape<128, 128, 16>;
494:   using WarpShape = GemmShape<32, 64, 16>;
495:   using InstructionShape = GemmShape<8, 8, 4>;
496:   static int const kStages = 3;
497: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 498-505
```cpp
498:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
499:       ElementC, 1, ElementAccumulator,
500:       ElementAccumulator>;
501: 
502:   using Operator = arch::OpMultiplyAdd;
503: };
504: 
505: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 506-515
```cpp
506: template <>
507: struct DefaultGemmConfiguration<
508:     arch::OpClassTensorOp, 
509:     arch::Sm80, 
510:     complex<double>,
511:     complex<double>, 
512:     complex<double>,
513:     complex<double>
514:   > {
515: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 516-518
```cpp
516:   static int const kAlignmentA = 1;
517:   static int const kAlignmentB = 1;
518:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 519-523
```cpp
519:   using ThreadblockShape = GemmShape<64, 64, 16>;
520:   using WarpShape = GemmShape<32, 32, 16>;
521:   using InstructionShape = GemmShape<8, 8, 4>;
522:   static int const kStages = 3;
523: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 524-532
```cpp
524:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
525:       complex<double>, 1, complex<double>,
526:       complex<double>>;
527: 
528:   using Operator = arch::OpMultiplyAddComplex;
529: };
530: 
531: ////////////////////////////////////////////////////////////////////////////////
532: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 533-542
```cpp
533: template < 
534:   typename ElementC>
535: struct DefaultGemmConfiguration<
536:   arch::OpClassTensorOp, 
537:   arch::Sm80, 
538:   int8_t, 
539:   int8_t, 
540:   ElementC, 
541:   int32_t> {
542:      
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 543-545
```cpp
543:   static int const kAlignmentA = 128 / sizeof_bits<int8_t>::value;
544:   static int const kAlignmentB = 128 / sizeof_bits<int8_t>::value;
545:  
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 546-550
```cpp
546:   using ThreadblockShape = GemmShape<128, 256, 64>;
547:   using WarpShape = GemmShape<64, 64, 64>;
548:   using InstructionShape = GemmShape<16, 8, 32>;
549:   static int const kStages = 3;
550: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 551-558
```cpp
551:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
552:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
553: 
554:   using Operator = arch::OpMultiplyAddSaturate;
555: };
556: 
557: ////////////////////////////////////////////////////////////////////////////////
558: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 559-568
```cpp
559: template < 
560:   typename ElementC>
561: struct DefaultGemmConfiguration<
562:   arch::OpClassTensorOp, 
563:   arch::Sm80, 
564:   int8_t, 
565:   uint8_t, 
566:   ElementC, 
567:   int32_t> {
568:       
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 569-571
```cpp
569:   static int const kAlignmentA = 128 / sizeof_bits<int8_t>::value;
570:   static int const kAlignmentB = 128 / sizeof_bits<uint8_t>::value;
571:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 572-576
```cpp
572:   using ThreadblockShape = GemmShape<128, 256, 64>;
573:   using WarpShape = GemmShape<64, 64, 64>;
574:   using InstructionShape = GemmShape<16, 8, 32>;
575:   static int const kStages = 3;
576: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 577-584
```cpp
577:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
578:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
579: 
580:   using Operator = arch::OpMultiplyAddSaturate;
581: };
582: 
583: ////////////////////////////////////////////////////////////////////////////////
584: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 585-594
```cpp
585: template < 
586:   typename ElementC>
587: struct DefaultGemmConfiguration<
588:   arch::OpClassTensorOp, 
589:   arch::Sm80, 
590:   uint8_t, 
591:   int8_t, 
592:   ElementC, 
593:   int32_t> {
594:       
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 595-597
```cpp
595:   static int const kAlignmentA = 128 / sizeof_bits<uint8_t>::value;
596:   static int const kAlignmentB = 128 / sizeof_bits<int8_t>::value;
597:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 598-602
```cpp
598:   using ThreadblockShape = GemmShape<128, 256, 64>;
599:   using WarpShape = GemmShape<64, 64, 64>;
600:   using InstructionShape = GemmShape<16, 8, 32>;
601:   static int const kStages = 3;
602: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 603-610
```cpp
603:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
604:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
605: 
606:   using Operator = arch::OpMultiplyAddSaturate;
607: };
608: 
609: ////////////////////////////////////////////////////////////////////////////////
610: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 611-620
```cpp
611: template < 
612:   typename ElementC>
613: struct DefaultGemmConfiguration<
614:   arch::OpClassTensorOp, 
615:   arch::Sm80, 
616:   uint8_t, 
617:   uint8_t, 
618:   ElementC, 
619:   int32_t> {
620:       
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 621-623
```cpp
621:   static int const kAlignmentA = 128 / sizeof_bits<uint8_t>::value;
622:   static int const kAlignmentB = 128 / sizeof_bits<uint8_t>::value;
623:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 624-628
```cpp
624:   using ThreadblockShape = GemmShape<128, 256, 64>;
625:   using WarpShape = GemmShape<64, 64, 64>;
626:   using InstructionShape = GemmShape<16, 8, 32>;
627:   static int const kStages = 3;
628: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 629-636
```cpp
629:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
630:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
631: 
632:   using Operator = arch::OpMultiplyAddSaturate;
633: };
634: 
635: ////////////////////////////////////////////////////////////////////////////////
636: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 637-646
```cpp
637: template < 
638:   typename ElementC>
639: struct DefaultGemmConfiguration<
640:   arch::OpClassTensorOp, 
641:   arch::Sm80, 
642:   int4b_t, 
643:   int4b_t, 
644:   ElementC, 
645:   int32_t> {
646:       
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 647-649
```cpp
647:   static int const kAlignmentA = 128 / sizeof_bits<int4b_t>::value;
648:   static int const kAlignmentB = 128 / sizeof_bits<int4b_t>::value;
649:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 650-654
```cpp
650:   using ThreadblockShape = GemmShape<128, 256, 128>;
651:   using WarpShape = GemmShape<64, 64, 128>;
652:   using InstructionShape = GemmShape<16, 8, 64>;
653:   static int const kStages = 3;
654: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 655-662
```cpp
655:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
656:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
657: 
658:   using Operator = arch::OpMultiplyAddSaturate;
659: };
660: 
661: ////////////////////////////////////////////////////////////////////////////////
662: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 663-672
```cpp
663: template < 
664:   typename ElementC>
665: struct DefaultGemmConfiguration<
666:   arch::OpClassTensorOp, 
667:   arch::Sm80, 
668:   int4b_t, 
669:   uint4b_t, 
670:   ElementC, 
671:   int32_t> {
672:        
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 673-675
```cpp
673:   static int const kAlignmentA = 128 / sizeof_bits<int4b_t>::value;
674:   static int const kAlignmentB = 128 / sizeof_bits<uint4b_t>::value;
675:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 676-680
```cpp
676:   using ThreadblockShape = GemmShape<128, 256, 128>;
677:   using WarpShape = GemmShape<64, 64, 128>;
678:   using InstructionShape = GemmShape<16, 8, 64>;
679:   static int const kStages = 3;
680: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 681-688
```cpp
681:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
682:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
683: 
684:   using Operator = arch::OpMultiplyAddSaturate;
685: };
686: 
687: ////////////////////////////////////////////////////////////////////////////////
688: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 689-698
```cpp
689: template < 
690:   typename ElementC>
691: struct DefaultGemmConfiguration<
692:   arch::OpClassTensorOp, 
693:   arch::Sm80, 
694:   uint4b_t, 
695:   int4b_t, 
696:   ElementC, 
697:   int32_t> {
698:        
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 699-701
```cpp
699:   static int const kAlignmentA = 128 / sizeof_bits<uint4b_t>::value;
700:   static int const kAlignmentB = 128 / sizeof_bits<int4b_t>::value;
701:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 702-706
```cpp
702:   using ThreadblockShape = GemmShape<128, 256, 128>;
703:   using WarpShape = GemmShape<64, 64, 128>;
704:   using InstructionShape = GemmShape<16, 8, 64>;
705:   static int const kStages = 3;
706: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 707-714
```cpp
707:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
708:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
709: 
710:   using Operator = arch::OpMultiplyAddSaturate;
711: };
712: 
713: ////////////////////////////////////////////////////////////////////////////////
714: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 715-724
```cpp
715: template < 
716:   typename ElementC>
717: struct DefaultGemmConfiguration<
718:   arch::OpClassTensorOp, 
719:   arch::Sm80, 
720:   uint4b_t, 
721:   uint4b_t, 
722:   ElementC, 
723:   int32_t> {
724:        
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 725-727
```cpp
725:   static int const kAlignmentA = 128 / sizeof_bits<uint4b_t>::value;
726:   static int const kAlignmentB = 128 / sizeof_bits<uint4b_t>::value;
727:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 728-732
```cpp
728:   using ThreadblockShape = GemmShape<128, 256, 128>;
729:   using WarpShape = GemmShape<64, 64, 128>;
730:   using InstructionShape = GemmShape<16, 8, 64>;
731:   static int const kStages = 3;
732: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 733-740
```cpp
733:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
734:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
735: 
736:   using Operator = arch::OpMultiplyAddSaturate;
737: };
738: 
739: ////////////////////////////////////////////////////////////////////////////////
740: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 741-750
```cpp
741: template < 
742:   typename ElementC>
743: struct DefaultGemmConfiguration<
744:   arch::OpClassTensorOp, 
745:   arch::Sm80, 
746:   uint1b_t, 
747:   uint1b_t, 
748:   ElementC, 
749:   int32_t> {
750:        
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 751-753
```cpp
751:   static int const kAlignmentA = 128 / sizeof_bits<uint1b_t>::value;
752:   static int const kAlignmentB = 128 / sizeof_bits<uint1b_t>::value;
753:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 754-758
```cpp
754:   using ThreadblockShape = GemmShape<128, 256, 512>;
755:   using WarpShape = GemmShape<64, 64, 512>;
756:   using InstructionShape = GemmShape<16, 8, 256>;
757:   static int const kStages = 3;
758: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 759-766
```cpp
759:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
760:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
761: 
762:   using Operator = arch::OpMultiplyAdd;
763: };
764: 
765: ////////////////////////////////////////////////////////////////////////////////
766: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 767-776
```cpp
767: template <
768:   typename ElementC>
769: struct DefaultGemmConfiguration<
770:   arch::OpClassTensorOp,
771:   arch::Sm80,
772:   int4b_t,
773:   int8_t,
774:   ElementC,
775:   int32_t> {
776: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 777-779
```cpp
777:   static int const kAlignmentA = 128 / sizeof_bits<int4b_t>::value;
778:   static int const kAlignmentB = 128 / sizeof_bits<int8_t>::value;
779: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 780-784
```cpp
780:   using ThreadblockShape = GemmShape<128, 256, 64>;
781:   using WarpShape = GemmShape<64, 64, 64>;
782:   using InstructionShape = GemmShape<16, 8, 32>;
783:   static int const kStages = 3;
784: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 785-792
```cpp
785:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
786:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
787: 
788:   using Operator = arch::OpMultiplyAddSaturate;
789: };
790: 
791: ////////////////////////////////////////////////////////////////////////////////
792: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 793-802
```cpp
793: template <
794:   typename ElementC>
795: struct DefaultGemmConfiguration<
796:   arch::OpClassTensorOp,
797:   arch::Sm80,
798:   int8_t,
799:   int4b_t,
800:   ElementC,
801:   int32_t> {
802: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 803-805
```cpp
803:   static int const kAlignmentA = 128 / sizeof_bits<int8_t>::value;
804:   static int const kAlignmentB = 128 / sizeof_bits<int4b_t>::value;
805: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 806-810
```cpp
806:   using ThreadblockShape = GemmShape<128, 256, 64>;
807:   using WarpShape = GemmShape<64, 64, 64>;
808:   using InstructionShape = GemmShape<16, 8, 32>;
809:   static int const kStages = 3;
810: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 811-819
```cpp
811:   using EpilogueOutputOp = epilogue::thread::LinearCombinationClamp<
812:       ElementC, 128 / sizeof_bits<ElementC>::value, int32_t, float>;
813: 
814:   using Operator = arch::OpMultiplyAddSaturate;
815: };
816: 
817: ////////////////////////////////////////////////////////////////////////////////
818: 
819: /// Base configuration for all {fe4m3, fe5m2} x {fe4m3, fe5m2} combinations on SM89
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 820-825
```cpp
820: template <
821:   typename ElementA,
822:   typename ElementB,
823:   typename ElementC,
824:   typename ElementAccumulator>
825: struct DefaultGemmConfigurationSm89F8 {
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfigurationSm89F8.
**CN:** 声明模板参数并开始定义 DefaultGemmConfigurationSm89F8。

### Lines 826-832
```cpp
826:   static_assert((platform::is_same<ElementA, cutlass::float_e4m3_t>::value ||
827:                  platform::is_same<ElementA, cutlass::float_e5m2_t>::value),
828:                 "ElementA must be of type float_e4m3_t or float_e5m2_t");
829:   static_assert((platform::is_same<ElementB, cutlass::float_e4m3_t>::value ||
830:                  platform::is_same<ElementB, cutlass::float_e5m2_t>::value),
831:                 "ElementB must be of type float_e4m3_t or float_e5m2_t");
832: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 833-835
```cpp
833:   static int const kAlignmentA = 128 / sizeof_bits<ElementA>::value;
834:   static int const kAlignmentB = 128 / sizeof_bits<ElementB>::value;
835: 
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 836-840
```cpp
836:   using ThreadblockShape = GemmShape<128, 256, 64>;
837:   using WarpShape = GemmShape<64, 64, 64>;
838:   using InstructionShape = GemmShape<16, 8, 32>;
839:   static int const kStages = 3;
840: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 841-848
```cpp
841:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
842:       ElementC, 128 / sizeof_bits<ElementC>::value, ElementAccumulator,
843:       ElementAccumulator>;
844: 
845:   using Operator = arch::OpMultiplyAdd;
846: };
847: 
848: /// Partial specialization for SM89 fe4m3 x fe4m3
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 849-860
```cpp
849: template <typename ElementC, typename ElementAccumulator>
850: struct DefaultGemmConfiguration<
851:   arch::OpClassTensorOp,
852:   arch::Sm89,
853:   cutlass::float_e4m3_t,
854:   cutlass::float_e4m3_t,
855:   ElementC,
856:   ElementAccumulator> : DefaultGemmConfigurationSm89F8<
857:                             cutlass::float_e4m3_t,
858:                             cutlass::float_e4m3_t,
859:                             ElementC,
860:                             ElementAccumulator> {};
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 861-862
```cpp
861: 
862: /// Partial specialization for SM89 fe4m3 x fe5m2
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 863-874
```cpp
863: template <typename ElementC, typename ElementAccumulator>
864: struct DefaultGemmConfiguration<
865:   arch::OpClassTensorOp,
866:   arch::Sm89,
867:   cutlass::float_e4m3_t,
868:   cutlass::float_e5m2_t,
869:   ElementC,
870:   ElementAccumulator> : DefaultGemmConfigurationSm89F8<
871:                             cutlass::float_e4m3_t,
872:                             cutlass::float_e5m2_t,
873:                             ElementC,
874:                             ElementAccumulator> {};
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 875-876
```cpp
875: 
876: /// Partial specialization for SM89 fe5m2 x fe4m3
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 877-888
```cpp
877: template <typename ElementC, typename ElementAccumulator>
878: struct DefaultGemmConfiguration<
879:   arch::OpClassTensorOp,
880:   arch::Sm89,
881:   cutlass::float_e5m2_t,
882:   cutlass::float_e4m3_t,
883:   ElementC,
884:   ElementAccumulator> : DefaultGemmConfigurationSm89F8<
885:                             cutlass::float_e5m2_t,
886:                             cutlass::float_e4m3_t,
887:                             ElementC,
888:                             ElementAccumulator> {};
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 889-890
```cpp
889: 
890: /// Partial specialization for SM89 fe5m2 x fe5m2
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 891-902
```cpp
891: template <typename ElementC, typename ElementAccumulator>
892: struct DefaultGemmConfiguration<
893:   arch::OpClassTensorOp,
894:   arch::Sm89,
895:   cutlass::float_e5m2_t,
896:   cutlass::float_e5m2_t,
897:   ElementC,
898:   ElementAccumulator> : DefaultGemmConfigurationSm89F8<
899:                             cutlass::float_e5m2_t,
900:                             cutlass::float_e5m2_t,
901:                             ElementC,
902:                             ElementAccumulator> {};
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 903-905
```cpp
903: 
904: ////////////////////////////////////////////////////////////////////////////////
905: 
```
**EN:** Provides supporting declarations that connect this header to the larger CUTLASS GEMM hierarchy.
**CN:** 提供支撑性声明，使该头文件接入更大的 CUTLASS GEMM 层次结构。

### Lines 906-910
```cpp
906: template <typename ElementC,
907:           typename ElementAccumulator>
908: struct DefaultGemmConfiguration<arch::OpClassTensorOp, arch::Sm90, double,
909:                                 double, ElementC, ElementAccumulator> {
910: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 911-913
```cpp
911:   static int const kAlignmentA = 1;
912:   static int const kAlignmentB = 1;
913:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 914-918
```cpp
914:   using ThreadblockShape = GemmShape<128, 256, 64>;
915:   using WarpShape = GemmShape<64, 64, 64>;
916:   using InstructionShape = GemmShape<16, 8, 4>;
917:   static int const kStages = 3;
918: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 919-925
```cpp
919:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
920:       ElementC, 1, ElementAccumulator,
921:       ElementAccumulator>;
922: 
923:   using Operator = arch::OpMultiplyAdd;
924: };
925: 
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 926-935
```cpp
926: template <>
927: struct DefaultGemmConfiguration<
928:     arch::OpClassTensorOp, 
929:     arch::Sm90, 
930:     complex<double>,
931:     complex<double>, 
932:     complex<double>,
933:     complex<double>
934:   > {
935: 
```
**EN:** Declares template parameters and begins the definition of DefaultGemmConfiguration.
**CN:** 声明模板参数并开始定义 DefaultGemmConfiguration。

### Lines 936-938
```cpp
936:   static int const kAlignmentA = 1;
937:   static int const kAlignmentB = 1;
938:   
```
**EN:** Defines compile-time constants or traits that specialize kernel behavior without runtime overhead.
**CN:** 定义编译期常量或 traits，在不增加运行时开销的情况下特化内核行为。

### Lines 939-943
```cpp
939:   using ThreadblockShape = GemmShape<64, 64, 16>;
940:   using WarpShape = GemmShape<32, 32, 16>;
941:   using InstructionShape = GemmShape<16, 8, 4>;
942:   static int const kStages = 3;
943: 
```
**EN:** Introduces local type aliases (ThreadblockShape, WarpShape, InstructionShape) to simplify downstream template code.
**CN:** 引入本地类型别名（ThreadblockShape, WarpShape, InstructionShape），简化后续模板代码。

### Lines 944-953
```cpp
944:   using EpilogueOutputOp = epilogue::thread::LinearCombination<
945:       complex<double>, 1, complex<double>,
946:       complex<double>>;
947: 
948:   using Operator = arch::OpMultiplyAddComplex;
949: };
950: 
951: } // namespace device
952: } // namespace gemm
953: } // namespace cutlass
```
**EN:** Introduces local type aliases (EpilogueOutputOp, Operator) to simplify downstream template code.
**CN:** 引入本地类型别名（EpilogueOutputOp, Operator），简化后续模板代码。

### Lines 954-955
```cpp
954: 
955: ////////////////////////////////////////////////////////////////////////////////
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
- **EN:** WMMA programming model  
  **CN:** WMMA 编程模型
- **EN:** SIMT execution policy  
  **CN:** SIMT 执行策略
- **EN:** Epilogue/output transformation  
  **CN:** Epilogue/输出变换

## Dependencies / 依赖关系

- **Direct includes / 直接包含:** none / 无
- **Key symbols / 关键符号:** `DefaultGemmConfiguration`, `DefaultGemmConfigurationSm89F8`
- **Relationship / 关系:** This header participates in the CUTLASS GEMM hierarchy and is consumed together with neighboring policy, iterator, and kernel headers.  
  **关系 (CN):** 该头文件属于 CUTLASS GEMM 分层体系，会与相邻的策略、迭代器和内核头文件协同使用。
