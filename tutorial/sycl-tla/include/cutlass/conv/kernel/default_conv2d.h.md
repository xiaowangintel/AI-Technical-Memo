# default_conv2d.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/kernel/default_conv2d.h`
- **Purpose (EN):** Default kernel-level implicit GEMM convolution definitions for threadblock-scoped epilogue.
- **用途 (CN):** 组合 默认 二维卷积 对应的内核级卷积逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
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
```
**EN:** Records the BSD-3-Clause license and redistribution conditions for the file.

**CN:** 记录该文件的 BSD-3-Clause 许可和再分发条件。

### Lines 32-35
```cpp
 32: /*! \file
 33:     \brief
 34:       Default kernel-level implicit GEMM convolution definitions for threadblock-scoped epilogue.
 35: */
```
**EN:** Documents the file's purpose, terminology, and high-level usage.

**CN:** 说明文件的用途、术语以及高层使用方式。

### Lines 37-37
```cpp
 37: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 39-56
```cpp
 39: #include "cutlass/cutlass.h"
 40: #include "cutlass/gemm/threadblock/default_mma.h"
 41: #include "cutlass/gemm/threadblock/threadblock_swizzle.h"
 42: #include "cutlass/conv/threadblock/threadblock_swizzle.h"
 43: #include "cutlass/epilogue/threadblock/default_epilogue_simt.h"
 44: #include "cutlass/epilogue/threadblock/default_epilogue_tensor_op.h"
 45: #include "cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h"
 46: #include "cutlass/epilogue/threadblock/default_epilogue_with_broadcast.h"
 47: #include "cutlass/epilogue/threadblock/default_epilogue_with_reduction.h"
 48: #include "cutlass/conv/convolution.h"
 49: #include "cutlass/conv/threadblock/conv2d_tile_iterator.h"
 50: #include "cutlass/conv/threadblock/implicit_gemm_pipelined.h"
 51: #include "cutlass/conv/threadblock/implicit_gemm_multistage.h"
 52: #include "cutlass/conv/threadblock/implicit_gemm_fprop_fusion_multistage.h"
 53: #include "cutlass/conv/threadblock/implicit_gemm_wgrad_fusion_multistage.h"
 54: #include "cutlass/conv/kernel/implicit_gemm_convolution.h"
 55: #include "cutlass/conv/kernel/implicit_gemm_convolution_fusion.h"
 56: #include "cutlass/conv/kernel/implicit_gemm_convolution_strided_dgrad.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `default_mma.h`, `threadblock_swizzle.h`, `threadblock_swizzle.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `default_mma.h`, `threadblock_swizzle.h`, `threadblock_swizzle.h`。

### Lines 60-62
```cpp
 60: namespace cutlass {
 61: namespace conv {
 62: namespace kernel {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 66-66
```cpp
 66: namespace detail {
```
**EN:** Opens the namespace scope `detail` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `detail`。

### Lines 68-83
```cpp
 68: template <
 69:   typename ArchTag,
 70:   typename Shape,
 71:   typename WarpMmaTensorOp,
 72:   int PartitionsK,
 73:   typename OutputOp
 74: >
 75: struct DefaultConvEpilogue {
 76:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueTensorOp<
 77:     Shape,
 78:     WarpMmaTensorOp,
 79:     PartitionsK,
 80:     OutputOp,
 81:     OutputOp::kCount
 82:   >::Epilogue;
 83: };
```
**EN:** Declares struct `DefaultConvEpilogue`, a default conv epilogue component in the convolution stack.

**CN:** 声明结构体 `DefaultConvEpilogue`，它是卷积栈中的 默认 conv 尾处理 组件。

### Lines 85-97
```cpp
 85: template <
 86:   typename Shape,
 87:   typename WarpMmaTensorOp,
 88:   int PartitionsK,
 89:   typename OutputOp
 90: >
 91: struct DefaultConvEpilogue<
 92:   arch::Sm70,
 93:   Shape,
 94:   WarpMmaTensorOp,
 95:   PartitionsK,
 96:   OutputOp
 97: > {
```
**EN:** Stores member state such as `Shape`, `WarpMmaTensorOp`, `PartitionsK`, `Sm70` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Shape`, `WarpMmaTensorOp`, `PartitionsK`, `Sm70` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 99-106
```cpp
 99:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueVoltaTensorOp<
100:     Shape,
101:     WarpMmaTensorOp,
102:     PartitionsK,
103:     OutputOp,
104:     OutputOp::kCount
105:   >::Epilogue;
106: };
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 109-136
```cpp
109: template <
110:   typename ArchTag,
111:   typename Shape,
112:   typename WarpMmaSimt,
113:   typename ElementOutput,
114:   typename ElementTensor,
115:   typename ElementVector,
116:   typename OutputOp,
117:   int ElementsPerAccess,
118:   typename PermuteDLayout = layout::NoPermute,
119:   conv::StrideSupport StrideSupport = conv::StrideSupport::kUnity,
120:   int Rank = 4
121: >
122: struct DefaultConvEpilogueWithBroadcastSimt {
123:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueWithBroadcastSimt<
124:     Shape,
125:     WarpMmaSimt,
126:     ElementOutput,
127:     ElementTensor,
128:     ElementVector,
129:     OutputOp,
130:     ElementsPerAccess,
131:     false,
132:     PermuteDLayout,
133:     StrideSupport,
134:     Rank
135:   >::Epilogue;
136: };
```
**EN:** Declares struct `DefaultConvEpilogueWithBroadcastSimt`, a default conv epilogue with broadcast simt component in the convolution stack.

**CN:** 声明结构体 `DefaultConvEpilogueWithBroadcastSimt`，它是卷积栈中的 默认 conv 尾处理 with 广播 simt 组件。

### Lines 138-158
```cpp
138: template <
139:   typename ArchTag,
140:   typename Shape,
141:   typename WarpMmaSimt,
142:   typename ElementOutput,
143:   typename ElementTensor,
144:   typename ElementVector,
145:   typename OutputOp,
146:   int ElementsPerAccess
147: >
148: struct DefaultConvEpilogueWithBroadcastSimtStridedDgrad {
149:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueWithBroadcastSimtStridedDgrad<
150:     Shape,
151:     WarpMmaSimt,
152:     ElementOutput,
153:     ElementTensor,
154:     ElementVector,
155:     OutputOp,
156:     ElementsPerAccess
157:   >::Epilogue;
158: };
```
**EN:** Declares struct `DefaultConvEpilogueWithBroadcastSimtStridedDgrad`, a default conv epilogue with broadcast simt strided data-gradient component in the convolution stack.

**CN:** 声明结构体 `DefaultConvEpilogueWithBroadcastSimtStridedDgrad`，它是卷积栈中的 默认 conv 尾处理 with 广播 simt 跨步 数据梯度 组件。

### Lines 160-182
```cpp
160: template <
161:   typename ArchTag,
162:   typename Shape,
163:   typename WarpMmaTensorOp,
164:   int PartitionsK,
165:   typename ElementOutput,
166:   typename ElementTensor,
167:   typename ElementVector,
168:   typename OutputOp,
169:   int ElementsPerAccess
170: >
171: struct DefaultConvEpilogueWithBroadcastTensorOp {
172:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueWithBroadcastTensorOp<
173:     Shape,
174:     WarpMmaTensorOp,
175:     PartitionsK,
176:     ElementOutput,
177:     ElementTensor,
178:     ElementVector,
179:     OutputOp,
180:     ElementsPerAccess
181:   >::Epilogue;
182: };
```
**EN:** Declares struct `DefaultConvEpilogueWithBroadcastTensorOp`, a default conv epilogue with broadcast tensor op component in the convolution stack.

**CN:** 声明结构体 `DefaultConvEpilogueWithBroadcastTensorOp`，它是卷积栈中的 默认 conv 尾处理 with 广播 tensor op 组件。

### Lines 184-215
```cpp
184: template <
185:   typename Shape,
186:   typename WarpMmaTensorOp,
187:   int PartitionsK,
188:   typename ElementOutput,
189:   typename ElementTensor,
190:   typename ElementVector,
191:   typename OutputOp,
192:   int ElementsPerAccess
193: >
194: struct DefaultConvEpilogueWithBroadcastTensorOp<
195:   arch::Sm70,
196:   Shape,
197:   WarpMmaTensorOp,
198:   PartitionsK,
199:   ElementOutput,
200:   ElementTensor,
201:   ElementVector,
202:   OutputOp,
203:   ElementsPerAccess
204:   > {
205:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueWithBroadcastVoltaTensorOp<
206:     Shape,
207:     WarpMmaTensorOp,
208:     PartitionsK,
209:     ElementOutput,
210:     ElementTensor,
211:     ElementVector,
212:     OutputOp,
213:     ElementsPerAccess
214:   >::Epilogue;
215: };
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 219-239
```cpp
219: template <
220:   typename ArchTag,
221:   typename Shape,
222:   typename WarpMmaTensorOp,
223:   int PartitionsK,
224:   typename ElementOutput,
225:   typename OutputOp,
226:   typename ReductionOp,
227:   int ElementsPerAccess
228: >
229: struct DefaultConvEpilogueWithReductionTensorOp {
230:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueWithReductionTensorOp<
231:     Shape,
232:     WarpMmaTensorOp,
233:     PartitionsK,
234:     ElementOutput,
235:     OutputOp,
236:     ReductionOp,
237:     ElementsPerAccess
238:   >::Epilogue;
239: };
```
**EN:** Declares struct `DefaultConvEpilogueWithReductionTensorOp`, a default conv epilogue with reduction tensor op component in the convolution stack.

**CN:** 声明结构体 `DefaultConvEpilogueWithReductionTensorOp`，它是卷积栈中的 默认 conv 尾处理 with 归约 tensor op 组件。

### Lines 241-269
```cpp
241: template <
242:   typename Shape,
243:   typename WarpMmaTensorOp,
244:   int PartitionsK,
245:   typename ElementOutput,
246:   typename OutputOp,
247:   typename ReductionOp,
248:   int ElementsPerAccess
249: >
250: struct DefaultConvEpilogueWithReductionTensorOp<
251:   arch::Sm70,
252:   Shape,
253:   WarpMmaTensorOp,
254:   PartitionsK,
255:   ElementOutput,
256:   OutputOp,
257:   ReductionOp,
258:   ElementsPerAccess
259:   > {
260:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueWithReductionVoltaTensorOp<
261:     Shape,
262:     WarpMmaTensorOp,
263:     PartitionsK,
264:     ElementOutput,
265:     OutputOp,
266:     ReductionOp,
267:     ElementsPerAccess
268:   >::Epilogue;
269: };
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 273-289
```cpp
273: // Defaults for strided Dgrad
274: template <
275:   typename ArchTag,
276:   typename Shape,
277:   typename WarpMmaTensorOp,
278:   int PartitionsK,
279:   typename OutputOp
280: >
281: struct DefaultConvEpilogueStridedDgrad {
282:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueTensorOpStridedDgrad<
283:     Shape,
284:     WarpMmaTensorOp,
285:     PartitionsK,
286:     OutputOp,
287:     OutputOp::kCount
288:   >::Epilogue;
289: };
```
**EN:** Declares struct `DefaultConvEpilogueStridedDgrad`. The nearby comment explains that it serves the surrounding default conv epilogue strided data-gradient logic.

**CN:** 声明结构体 `DefaultConvEpilogueStridedDgrad`，相邻注释说明它服务于周围的 默认 conv 尾处理 跨步 数据梯度 逻辑。

### Lines 291-303
```cpp
291: template <
292:   typename Shape,
293:   typename WarpMmaTensorOp,
294:   int PartitionsK,
295:   typename OutputOp
296: >
297: struct DefaultConvEpilogueStridedDgrad<
298:   arch::Sm70,
299:   Shape,
300:   WarpMmaTensorOp,
301:   PartitionsK,
302:   OutputOp
303: > {
```
**EN:** Stores member state such as `Shape`, `WarpMmaTensorOp`, `PartitionsK`, `Sm70` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Shape`, `WarpMmaTensorOp`, `PartitionsK`, `Sm70` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 305-312
```cpp
305:   using Epilogue = typename epilogue::threadblock::DefaultEpilogueVoltaTensorOpStridedDgrad<
306:     Shape,
307:     WarpMmaTensorOp,
308:     PartitionsK,
309:     OutputOp,
310:     OutputOp::kCount
311:   >::Epilogue;
312: };
```
**EN:** Introduces aliases such as `Epilogue` to keep the surrounding template code readable.

**CN:** 引入 `Epilogue` 等别名，以提升周围模板代码的可读性。

### Lines 314-314
```cpp
314: } // namespace detail
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 318-320
```cpp
318: } // namespace kernel
319: } // namespace conv
320: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Default kernel-level implicit GEMM convolution definitions for threadblock-scoped epilogue. **CN:** 核心作用：组合 默认 二维卷积 对应的内核级卷积逻辑。
- **EN:** Key exported symbols include `DefaultConvEpilogue`, `DefaultConvEpilogueWithBroadcastSimt`, `DefaultConvEpilogueWithBroadcastSimtStridedDgrad`, `DefaultConvEpilogueWithBroadcastTensorOp`, `DefaultConvEpilogueWithReductionTensorOp`, `DefaultConvEpilogueStridedDgrad`. **CN:** 关键导出符号包括 `DefaultConvEpilogue`, `DefaultConvEpilogueWithBroadcastSimt`, `DefaultConvEpilogueWithBroadcastSimtStridedDgrad`, `DefaultConvEpilogueWithBroadcastTensorOp`, `DefaultConvEpilogueWithReductionTensorOp`, `DefaultConvEpilogueStridedDgrad`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** MMA composition is central: the file either defines math operators directly or prepares data for them. **CN:** MMA 组合是核心：该文件要么直接定义数学算子，要么为其准备数据。
- **EN:** The implementation is architecture-aware and may specialize behavior for specific GPU generations. **CN:** 该实现具有架构感知能力，可能会针对特定 GPU 代际进行特化。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/gemm/threadblock/default_mma.h`
- `cutlass/gemm/threadblock/threadblock_swizzle.h`
- `cutlass/conv/threadblock/threadblock_swizzle.h`
- `cutlass/epilogue/threadblock/default_epilogue_simt.h`
- `cutlass/epilogue/threadblock/default_epilogue_tensor_op.h`
- `cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`
- `cutlass/epilogue/threadblock/default_epilogue_with_broadcast.h`
- `cutlass/epilogue/threadblock/default_epilogue_with_reduction.h`
- `cutlass/conv/convolution.h`
- `cutlass/conv/threadblock/conv2d_tile_iterator.h`
- `cutlass/conv/threadblock/implicit_gemm_pipelined.h`
- `cutlass/conv/threadblock/implicit_gemm_multistage.h`
- `cutlass/conv/threadblock/implicit_gemm_fprop_fusion_multistage.h`
- `cutlass/conv/threadblock/implicit_gemm_wgrad_fusion_multistage.h`
- `cutlass/conv/kernel/implicit_gemm_convolution.h`
- `cutlass/conv/kernel/implicit_gemm_convolution_fusion.h`
- `cutlass/conv/kernel/implicit_gemm_convolution_strided_dgrad.h`

### Internal Relationships / 内部关系
- **EN:** Relies on tensor or matrix layout helpers. **CN:** 依赖张量或矩阵布局辅助类型。
