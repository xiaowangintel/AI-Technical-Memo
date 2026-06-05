# convnd_problem_shape.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/convnd_problem_shape.hpp`
- **Purpose (EN):** This file contains definitions and utility functions for describing convolution problem shapes.
- **用途 (CN):** 通过 N 维卷积 问题 形状 建模更一般的卷积问题形状。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
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

### Lines 31-33
```cpp
 31: /*! \file
 32:     \brief This file contains definitions and utility functions for describing convolution problem shapes.
 33: */
```
**EN:** Documents the file's purpose, terminology, and high-level usage.

**CN:** 说明文件的用途、术语以及高层使用方式。

### Lines 34-34
```cpp
 34: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 36-38
```cpp
 36: #include "cutlass/cutlass.h"
 37: #include "cutlass/tensor_coord.h"
 38: #include "cutlass/conv/convolution.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `tensor_coord.h`, `convolution.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `tensor_coord.h`, `convolution.h`。

### Lines 40-40
```cpp
 40: #include "cute/container/array.hpp"
```
**EN:** Imports direct dependencies used later in the file, including `array.hpp`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `array.hpp`。

### Lines 42-44
```cpp
 42: #if ! defined(__CUDACC_RTC__)
 43: #include <initializer_list>
 44: #endif
```
**EN:** Imports direct dependencies used later in the file, including `initializer_list`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `initializer_list`。

### Lines 49-49
```cpp
 49: namespace cutlass::conv {
```
**EN:** Opens the namespace scope `cutlass::conv` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass::conv`。

### Lines 53-55
```cpp
 53: // Implements the user facing argument for all CUTLASS 3.x convolutions in a rank agnostic fashion.
 54: // All tensors are flat and by default treated as layout right (NDHWC, KTRSC, NZPQK)
 55: // Supports asymmetric padding, traversal strides, dilations, and all conv algorithm types.
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 56-60
```cpp
 56: template <
 57:   conv::Operator ConvOp_,
 58:   int NumSpatialDimensions_
 59: >
 60: struct ConvProblemShape {
```
**EN:** Declares struct `ConvProblemShape`, a conv problem shape component in the convolution stack.

**CN:** 声明结构体 `ConvProblemShape`，它是卷积栈中的 conv 问题 形状 组件。

### Lines 61-75
```cpp
 61:   //
 62:   // Alias types for members
 63:   //
 65:   static constexpr int RankS = NumSpatialDimensions_;
 66:   static constexpr int RankT = NumSpatialDimensions_ + 2;
 67:   static constexpr conv::Operator ConvOp = ConvOp_;
 68:   static constexpr int NumSpatialDimensions = NumSpatialDimensions_;
 69:   using SpatialExtent = cute::array<int, RankS>;
 70:   using TensorExtent  = cute::array<int, RankT>;
 71:   using TensorStride  = cute::array<int64_t, RankT>;
 72:   using ShapePadding = SpatialExtent;
 73:   using TraversalStride = SpatialExtent;
 74:   using ShapeDilation = SpatialExtent;
 75:   using Corner = SpatialExtent;
```
**EN:** Introduces aliases such as `SpatialExtent`, `TensorExtent`, `TensorStride`, `ShapePadding`, `TraversalStride`, `ShapeDilation` to keep the surrounding template code readable.

**CN:** 引入 `SpatialExtent`, `TensorExtent`, `TensorStride`, `ShapePadding`, `TraversalStride`, `ShapeDilation` 等别名，以提升周围模板代码的可读性。

### Lines 77-86
```cpp
 77:   //
 78:   // Members
 79:   //
 80:   cutlass::conv::Mode mode{};
 81:   TensorExtent shape_A{};
 82:   TensorStride stride_A{};
 83:   TensorExtent shape_B{};
 84:   TensorStride stride_B{};
 85:   TensorExtent shape_C{};
 86:   TensorStride stride_C{};
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 88-93
```cpp
 88:   // asymmetric padding, both upper and lower padding must be >= 0
 89:   ShapePadding lower_padding{};
 90:   ShapePadding upper_padding{};
 91:   TraversalStride traversal_stride{};
 92:   ShapeDilation dilation{};
 93:   int groups = 1;
```
**EN:** Stores member state such as `groups` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `groups` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 99-99
```cpp
 99:   ConvProblemShape() = default;
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 101-118
```cpp
101:   // Constructor accepts user facing arguments and computes to stores the corners as its internal state
102:   ConvProblemShape(
103:       conv::Mode mode,                                                     // convolution/cross-correlation
104:       TensorExtent shape_act,                                              // [n,d,h,w,c]
105:       TensorStride stride_act,                                             // [n,d,h,w,c]
106:       TensorExtent shape_flt,                                              // [k,t,r,s,c]
107:       TensorStride stride_flt,                                             // [k,t,r,s,c]
108:       ShapePadding lower_padding,                                          // [pad_d, pad_h, pad_w]
109:       ShapePadding upper_padding,                                          // [pad_d, pad_h, pad_w]
110:       TraversalStride tstride,                                             // [stride_d, stride_h, stride_w]
111:       ShapeDilation dilation,                                              // [dilation_d, dilation_h, dilation_w]
112:       int groups)
113:       : mode(mode)
114:       , lower_padding(lower_padding)
115:       , upper_padding(upper_padding)
116:       , traversal_stride(tstride)
117:       , dilation(dilation)
118:       , groups(groups) {
```
**EN:** Provides constructor-style initialization for `ConvProblemShape`.

**CN:** 为 `ConvProblemShape` 提供构造式初始化逻辑。

### Lines 120-122
```cpp
120:     auto [shape_xformed_act, stride_xformed_act] = calculate_xformed_act(shape_act, shape_flt);
121:     set_shape_stride_ABC(shape_act, stride_act, shape_flt, stride_flt, shape_xformed_act, stride_xformed_act);
122:   }
```
**EN:** Defines function `calculate_xformed_act` for this stage of the convolution workflow.

**CN:** 定义函数 `calculate_xformed_act`，服务于卷积工作流的这一阶段。

### Lines 124-142
```cpp
124:   // Allow user input of xformed activation stride to support non-packed strides.
125:   ConvProblemShape(
126:       conv::Mode mode,                                                     // convolution/cross-correlation
127:       TensorExtent shape_act,                                              // [n,d,h,w,c]
128:       TensorStride stride_act,                                             // [n,d,h,w,c]
129:       TensorExtent shape_flt,                                              // [k,t,r,s,c]
130:       TensorStride stride_flt,                                             // [k,t,r,s,c]
131:       TensorStride stride_xformed_act,                                     // [n,z,p,q,k]
132:       ShapePadding lower_padding,                                          // [pad_d, pad_h, pad_w]
133:       ShapePadding upper_padding,                                          // [pad_d, pad_h, pad_w]
134:       TraversalStride tstride,                                             // [stride_d, stride_h, stride_w]
135:       ShapeDilation dilation,                                              // [dilation_d, dilation_h, dilation_w]
136:       int groups)
137:       : mode(mode)
138:       , lower_padding(lower_padding)
139:       , upper_padding(upper_padding)
140:       , traversal_stride(tstride)
141:       , dilation(dilation)
142:       , groups(groups) {
```
**EN:** Provides constructor-style initialization for `ConvProblemShape`.

**CN:** 为 `ConvProblemShape` 提供构造式初始化逻辑。

### Lines 144-146
```cpp
144:     CUTLASS_ASSERT(stride_act[RankT - 1] == 1);
145:     CUTLASS_ASSERT(stride_flt[RankT - 1] == 1);
146:     CUTLASS_ASSERT(stride_xformed_act[RankT - 1] == 1);
```
**EN:** Provides constructor-style initialization for `CUTLASS_ASSERT`.

**CN:** 为 `CUTLASS_ASSERT` 提供构造式初始化逻辑。

### Lines 148-150
```cpp
148:     auto stride_act_packed = packed_stride_right_major(shape_act);
149:     auto stride_flt_packed = packed_stride_right_major(shape_flt);
150:     auto [shape_xformed_act, stride_xformed_act_packed] = calculate_xformed_act(shape_act, shape_flt);
```
**EN:** Defines function `packed_stride_right_major` for this stage of the convolution workflow.

**CN:** 定义函数 `packed_stride_right_major`，服务于卷积工作流的这一阶段。

### Lines 152-157
```cpp
152:     CUTLASS_PRAGMA_UNROLL
153:     for(int i = 0; i < RankT - 1; ++i) {
154:       CUTLASS_ASSERT(stride_act[i] >= stride_act_packed[i]);
155:       CUTLASS_ASSERT(stride_flt[i] >= stride_flt_packed[i]);
156:       CUTLASS_ASSERT(stride_xformed_act[i] >= stride_xformed_act_packed[i]);
157:     }
```
**EN:** Provides constructor-style initialization for `CUTLASS_ASSERT`.

**CN:** 为 `CUTLASS_ASSERT` 提供构造式初始化逻辑。

### Lines 159-160
```cpp
159:     set_shape_stride_ABC(shape_act, stride_act, shape_flt, stride_flt, shape_xformed_act, stride_xformed_act);
160:   }
```
**EN:** Defines function `set_shape_stride_ABC` for this stage of the convolution workflow.

**CN:** 定义函数 `set_shape_stride_ABC`，服务于卷积工作流的这一阶段。

### Lines 162-183
```cpp
162:   // Constructor accepts user facing arguments and presume packed tensor strides in canonical (CWHDN) order.
163:   ConvProblemShape(
164:       conv::Mode mode,
165:       TensorExtent shape_act,
166:       TensorExtent shape_flt,
167:       ShapePadding lower_padding,
168:       ShapePadding upper_padding,
169:       TraversalStride tstride,
170:       ShapeDilation dilation,
171:       int groups)
172:       : ConvProblemShape(
173:         mode,
174:         shape_act,
175:         packed_stride_right_major(shape_act),
176:         shape_flt,
177:         packed_stride_right_major(shape_flt),
178:         lower_padding,
179:         upper_padding,
180:         tstride,
181:         dilation,
182:         groups) {
183:     }
```
**EN:** Provides constructor-style initialization for `ConvProblemShape`.

**CN:** 为 `ConvProblemShape` 提供构造式初始化逻辑。

### Lines 185-185
```cpp
185: #if ! defined(__CUDACC_RTC__)
```
**EN:** Starts a conditional-compilation branch for architecture- or feature-specific code.

**CN:** 开始一个面向特定架构或特性的条件编译分支。

### Lines 186-199
```cpp
186:   // Constructor accepts user facing arguments and computes to stores the corners as its internal state
187:   ConvProblemShape(
188:       conv::Mode                     mode,
189:       std::initializer_list<int>     shape_act_,
190:       std::initializer_list<int64_t> stride_act_,
191:       std::initializer_list<int>     shape_flt_,
192:       std::initializer_list<int64_t> stride_flt_,
193:       std::initializer_list<int>     lower_padding_,
194:       std::initializer_list<int>     upper_padding_,
195:       std::initializer_list<int>     traversal_stride_,
196:       std::initializer_list<int>     dilation_,
197:       int groups)
198:       : mode(mode)
199:       , groups(groups) {
```
**EN:** Provides constructor-style initialization for `ConvProblemShape`.

**CN:** 为 `ConvProblemShape` 提供构造式初始化逻辑。

### Lines 201-204
```cpp
201:     TensorExtent shape_act{};
202:     TensorStride stride_act{};
203:     TensorExtent shape_flt{};
204:     TensorStride stride_flt{};
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 206-213
```cpp
206:     assert(shape_act_.size() == shape_act.size());
207:     assert(stride_act_.size() == stride_act.size());
208:     assert(shape_flt_.size() == shape_flt.size());
209:     assert(stride_flt_.size() == stride_flt.size());
210:     assert(lower_padding_.size() == lower_padding.size());
211:     assert(upper_padding_.size() == upper_padding.size());
212:     assert(traversal_stride_.size() == traversal_stride.size());
213:     assert(dilation_.size() == dilation.size());
```
**EN:** Defines function `assert` for this stage of the convolution workflow.

**CN:** 定义函数 `assert`，服务于卷积工作流的这一阶段。

### Lines 215-222
```cpp
215:     std::copy(shape_act_.begin(), shape_act_.end(), shape_act.begin());
216:     std::copy(stride_act_.begin(), stride_act_.end(), stride_act.begin());
217:     std::copy(shape_flt_.begin(), shape_flt_.end(), shape_flt.begin());
218:     std::copy(stride_flt_.begin(), stride_flt_.end(), stride_flt.begin());
219:     std::copy(lower_padding_.begin(), lower_padding_.end(), lower_padding.begin());
220:     std::copy(upper_padding_.begin(), upper_padding_.end(), upper_padding.begin());
221:     std::copy(traversal_stride_.begin(), traversal_stride_.end(), traversal_stride.begin());
222:     std::copy(dilation_.begin(), dilation_.end(), dilation.begin());
```
**EN:** Defines function `copy` for this stage of the convolution workflow.

**CN:** 定义函数 `copy`，服务于卷积工作流的这一阶段。

### Lines 224-226
```cpp
224:     auto [shape_xformed_act, stride_xformed_act] = calculate_xformed_act(shape_act, shape_flt);
225:     set_shape_stride_ABC(shape_act, stride_act, shape_flt, stride_flt, shape_xformed_act, stride_xformed_act);
226:   }
```
**EN:** Defines function `calculate_xformed_act` for this stage of the convolution workflow.

**CN:** 定义函数 `calculate_xformed_act`，服务于卷积工作流的这一阶段。

### Lines 228-247
```cpp
228:   // Allow user input of xformed activation stride to support non-packed strides.
229:   ConvProblemShape(
230:       conv::Mode                     mode,
231:       std::initializer_list<int>     shape_act_,
232:       std::initializer_list<int64_t> stride_act_,
233:       std::initializer_list<int>     shape_flt_,
234:       std::initializer_list<int64_t> stride_flt_,
235:       std::initializer_list<int64_t> stride_xformed_act_,
236:       std::initializer_list<int>     lower_padding_,
237:       std::initializer_list<int>     upper_padding_,
238:       std::initializer_list<int>     traversal_stride_,
239:       std::initializer_list<int>     dilation_,
240:       int groups)
241:       : mode(mode)
242:       , groups(groups) {
243:     TensorExtent shape_act{};
244:     TensorStride stride_act{};
245:     TensorExtent shape_flt{};
246:     TensorStride stride_flt{};
247:     TensorStride stride_xformed_act{};
```
**EN:** Provides constructor-style initialization for `ConvProblemShape`.

**CN:** 为 `ConvProblemShape` 提供构造式初始化逻辑。

### Lines 249-257
```cpp
249:     std::copy(shape_act_.begin(), shape_act_.end(), shape_act.begin());
250:     std::copy(stride_act_.begin(), stride_act_.end(), stride_act.begin());
251:     std::copy(shape_flt_.begin(), shape_flt_.end(), shape_flt.begin());
252:     std::copy(stride_flt_.begin(), stride_flt_.end(), stride_flt.begin());
253:     std::copy(stride_xformed_act_.begin(), stride_xformed_act_.end(), stride_xformed_act.begin());
254:     std::copy(lower_padding_.begin(), lower_padding_.end(), lower_padding.begin());
255:     std::copy(upper_padding_.begin(), upper_padding_.end(), upper_padding.begin());
256:     std::copy(traversal_stride_.begin(), traversal_stride_.end(), traversal_stride.begin());
257:     std::copy(dilation_.begin(), dilation_.end(), dilation.begin());
```
**EN:** Defines function `copy` for this stage of the convolution workflow.

**CN:** 定义函数 `copy`，服务于卷积工作流的这一阶段。

### Lines 259-261
```cpp
259:     CUTLASS_ASSERT(stride_act[RankT - 1] == 1);
260:     CUTLASS_ASSERT(stride_flt[RankT - 1] == 1);
261:     CUTLASS_ASSERT(stride_xformed_act[RankT - 1] == 1);
```
**EN:** Provides constructor-style initialization for `CUTLASS_ASSERT`.

**CN:** 为 `CUTLASS_ASSERT` 提供构造式初始化逻辑。

### Lines 263-265
```cpp
263:     auto stride_act_packed = packed_stride_right_major(shape_act);
264:     auto stride_flt_packed = packed_stride_right_major(shape_flt);
265:     auto [shape_xformed_act, stride_xformed_act_packed] = calculate_xformed_act(shape_act, shape_flt);
```
**EN:** Defines function `packed_stride_right_major` for this stage of the convolution workflow.

**CN:** 定义函数 `packed_stride_right_major`，服务于卷积工作流的这一阶段。

### Lines 267-272
```cpp
267:     CUTLASS_PRAGMA_UNROLL
268:     for(int i = 0; i < RankT - 1; ++i) {
269:       CUTLASS_ASSERT(stride_act[i] >= stride_act_packed[i]);
270:       CUTLASS_ASSERT(stride_flt[i] >= stride_flt_packed[i]);
271:       CUTLASS_ASSERT(stride_xformed_act[i] >= stride_xformed_act_packed[i]);
272:     }
```
**EN:** Provides constructor-style initialization for `CUTLASS_ASSERT`.

**CN:** 为 `CUTLASS_ASSERT` 提供构造式初始化逻辑。

### Lines 274-275
```cpp
274:     set_shape_stride_ABC(shape_act, stride_act, shape_flt, stride_flt, shape_xformed_act, stride_xformed_act);
275:   }
```
**EN:** Defines function `set_shape_stride_ABC` for this stage of the convolution workflow.

**CN:** 定义函数 `set_shape_stride_ABC`，服务于卷积工作流的这一阶段。

### Lines 277-292
```cpp
277:   // Constructor accepts user facing arguments and computes to stores the corners as its internal state
278:   ConvProblemShape(
279:       conv::Mode                     mode,
280:       std::initializer_list<int>     shape_act_,
281:       std::initializer_list<int>     shape_flt_,
282:       std::initializer_list<int>     lower_padding_,
283:       std::initializer_list<int>     upper_padding_,
284:       std::initializer_list<int>     traversal_stride_,
285:       std::initializer_list<int>     dilation_,
286:       int groups)
287:       : mode(mode)
288:       , groups(groups) {
289:     TensorExtent shape_act{};
290:     TensorStride stride_act{};
291:     TensorExtent shape_flt{};
292:     TensorStride stride_flt{};
```
**EN:** Provides constructor-style initialization for `ConvProblemShape`.

**CN:** 为 `ConvProblemShape` 提供构造式初始化逻辑。

### Lines 294-299
```cpp
294:     assert(shape_act_.size() == shape_act.size());
295:     assert(shape_flt_.size() == shape_flt.size());
296:     assert(lower_padding_.size() == lower_padding.size());
297:     assert(upper_padding_.size() == upper_padding.size());
298:     assert(traversal_stride_.size() == traversal_stride.size());
299:     assert(dilation_.size() == dilation.size());
```
**EN:** Defines function `assert` for this stage of the convolution workflow.

**CN:** 定义函数 `assert`，服务于卷积工作流的这一阶段。

### Lines 301-308
```cpp
301:     std::copy(shape_act_.begin(), shape_act_.end(), shape_act.begin());
302:     std::copy(shape_flt_.begin(), shape_flt_.end(), shape_flt.begin());
303:     std::copy(lower_padding_.begin(), lower_padding_.end(), lower_padding.begin());
304:     std::copy(upper_padding_.begin(), upper_padding_.end(), upper_padding.begin());
305:     std::copy(traversal_stride_.begin(), traversal_stride_.end(), traversal_stride.begin());
306:     std::copy(dilation_.begin(), dilation_.end(), dilation.begin());
307:     stride_act = packed_stride_right_major(shape_act);
308:     stride_flt = packed_stride_right_major(shape_flt);
```
**EN:** Defines function `copy` for this stage of the convolution workflow.

**CN:** 定义函数 `copy`，服务于卷积工作流的这一阶段。

### Lines 310-313
```cpp
310:     auto [shape_xformed_act, stride_xformed_act] = calculate_xformed_act(shape_act, shape_flt);
311:     set_shape_stride_ABC(shape_act, stride_act, shape_flt, stride_flt, shape_xformed_act, stride_xformed_act);
312:   }
313: #endif // not defined(__CUDACC_RTC__)
```
**EN:** Defines function `calculate_xformed_act` for this stage of the convolution workflow.

**CN:** 定义函数 `calculate_xformed_act`，服务于卷积工作流的这一阶段。

### Lines 315-322
```cpp
315:   // Set shape and stride of tensor A/B/C according to following table:
316:   // |              | Fprop  | Dgrad  | Wgrad |
317:   // | ------       | ------ | ------ | ------|
318:   // |   ShapeA     | NDHWC  | NZPQK  | NZPQK |
319:   // |   ShapeB     | KTRSC  | KTRSC  | NDHWC |
320:   // |   ShapeC     | NZPQK  | NDHWC  | KTRSC |
321:   //
322:   // Input comes from calculate_xformed_act, which does NOT depend on ConvOp.
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 323-332
```cpp
323:   CUTLASS_HOST_DEVICE
324:   constexpr void
325:   set_shape_stride_ABC(
326:     TensorExtent shape_act,
327:     TensorStride stride_act,
328:     TensorExtent shape_flt,
329:     TensorStride stride_flt,
330:     TensorExtent shape_xformed_act,
331:     TensorStride stride_xformed_act) {
332: #if defined(CUTLASS_DEBUG_TRACE_LEVEL) && (CUTLASS_DEBUG_TRACE_LEVEL > 1)
```
**EN:** Defines function `set_shape_stride_ABC` for this stage of the convolution workflow.

**CN:** 定义函数 `set_shape_stride_ABC`，服务于卷积工作流的这一阶段。

### Lines 333-356
```cpp
333:     printf("*** set_shape_stride_ABC ***");
334:     printf("\n  shape_act: ");
335:     print(shape_act);
336:     printf("\n  stride_act: ");
337:     print(stride_act);
338:     printf("\n  shape_flt: ");
339:     print(shape_flt);
340:     printf("\n  stride_flt: ");
341:     print(stride_flt);
342:     printf("\n  shape_xformed_act: ");
343:     print(shape_xformed_act);
344:     printf("\n  stride_xformed_act: ");
345:     print(stride_xformed_act);
346:     if constexpr (ConvOp == cutlass::conv::Operator::kFprop) {
347:       printf("\n  ConvOp: Fprop");
348:     }
349:     if constexpr (ConvOp == cutlass::conv::Operator::kDgrad) {
350:       printf("\n  ConvOp: Dgrad");
351:     }
352:     if constexpr (ConvOp == cutlass::conv::Operator::kWgrad) {
353:       printf("\n  ConvOp: Wgrad");
354:     }
355:     printf("\n");
356: #endif
```
**EN:** Defines function `constexpr` for this stage of the convolution workflow.

**CN:** 定义函数 `constexpr`，服务于卷积工作流的这一阶段。

### Lines 358-382
```cpp
358:     if constexpr (ConvOp == cutlass::conv::Operator::kFprop) {
359:       shape_A = shape_act;
360:       stride_A = stride_act;
361:       shape_B = shape_flt;
362:       stride_B = stride_flt;
363:       shape_C = shape_xformed_act;
364:       stride_C = stride_xformed_act;
365:     }
366:     else if constexpr (ConvOp == cutlass::conv::Operator::kDgrad) {
367:       shape_A = shape_xformed_act;
368:       stride_A = stride_xformed_act;
369:       shape_B = shape_flt;
370:       stride_B = stride_flt;
371:       shape_C = shape_act;
372:       stride_C = stride_act;
373:     }
374:     else if constexpr (ConvOp == cutlass::conv::Operator::kWgrad) {
375:       shape_A = shape_xformed_act;
376:       stride_A = stride_xformed_act;
377:       shape_B = shape_act;
378:       stride_B = stride_act;
379:       shape_C = shape_flt;
380:       stride_C = stride_flt;
381:     }
382: #if defined(CUTLASS_DEBUG_TRACE_LEVEL) && (CUTLASS_DEBUG_TRACE_LEVEL > 1)
```
**EN:** Defines function `constexpr` for this stage of the convolution workflow.

**CN:** 定义函数 `constexpr`，服务于卷积工作流的这一阶段。

### Lines 383-395
```cpp
383:     printf("\n  shape_A: ");
384:     print(shape_A);
385:     printf("\n  stride_A: ");
386:     print(stride_A);
387:     printf("\n  shape_B: ");
388:     print(shape_B);
389:     printf("\n  stride_B: ");
390:     print(stride_B);
391:     printf("\n  shape_C: ");
392:     print(shape_C);
393:     printf("\n  stride_C: ");
394:     print(stride_C);
395: #endif
```
**EN:** Defines function `printf` for this stage of the convolution workflow.

**CN:** 定义函数 `printf`，服务于卷积工作流的这一阶段。

### Lines 396-396
```cpp
396:   }
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 398-401
```cpp
398:   // Get A extents.
399:   // fprop: A extents array contains [N,D,H,W,C]. Turn that into ((W,H,D,N), (C))
400:   // dgrad: A extents array contains [N,Z,P,Q,K]. Turn that into ((Q,P,Z,N), (K))
401:   // wgrad: A extents array contains [N,Z,P,Q,K]. Turn that into ((K), (Q,P,Z,N))
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 402-406
```cpp
402:   CUTLASS_HOST_DEVICE
403:   constexpr auto
404:   get_shape_A() const {
405:     using cute::make_shape;
406:     using cute::take;
```
**EN:** Defines function `get_shape_A` for this stage of the convolution workflow.

**CN:** 定义函数 `get_shape_A`，服务于卷积工作流的这一阶段。

### Lines 408-413
```cpp
408:     if constexpr (ConvOp == conv::Operator::kFprop ||
409:                   ConvOp == conv::Operator::kDgrad) {
410:       return make_shape(
411:         cute::reverse(take<0, RankT - 1>(shape_A)),
412:         shape_A[RankT - 1]);
413:     }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 414-420
```cpp
414:     // For wgrad kernel, we need to linearize NZPQ for tensor A
415:     else if constexpr (ConvOp == conv::Operator::kWgrad) {
416:       return make_shape(
417:         shape_A[RankT - 1],
418:         cute::product(take<0, RankT - 1>(shape_A)));
419:     }
420:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 422-425
```cpp
422:   // Get B extents.
423:   // fprop: B extents array contains [K,T,R,S,C]. Turn that into ((K), (C,S,R,T))
424:   // dgrad: B extents array contains [K,T,R,S,C]. Turn that into ((C), (K,S,R,T))
425:   // wgrad: B extents array contains [N,D,H,W,C]. Turn that into ((C), (W,H,D,N))
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 426-431
```cpp
426:   CUTLASS_HOST_DEVICE
427:   constexpr auto
428:   get_shape_B() const {
429:     using cute::make_shape;
430:     using cute::reverse;
431:     using cute::take;
```
**EN:** Defines function `get_shape_B` for this stage of the convolution workflow.

**CN:** 定义函数 `get_shape_B`，服务于卷积工作流的这一阶段。

### Lines 433-443
```cpp
433:     if constexpr (ConvOp == conv::Operator::kFprop) {
434:       return make_shape(
435:         shape_B[0],
436:         reverse(take<1, RankT>(shape_B)));
437:     }
438:     else if constexpr (ConvOp == conv::Operator::kWgrad) {
439:       return make_shape(
440:         shape_B[RankT - 1],
441:         reverse(take<0, RankT - 1>(shape_B)));
442:     }
443:     else if constexpr (ConvOp == conv::Operator::kDgrad) {
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 444-451
```cpp
444:       // shape_B: [K,T,R,S,C], return: [(C),(K,S,R,T)]
445:       return make_shape(
446:         shape_B[RankT - 1],
447:         cute::insert<0>(
448:           reverse(take<1, RankT - 1>(shape_B)),
449:           shape_B[0]));
450:     }
451:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 453-456
```cpp
453:   // Get C extents.
454:   // fprop: C extents array contains [N,Z,P,Q,K]. Turn that into ((Q,P,Z,N), (K))
455:   // dgrad: C extents array contains [N,D,H,W,C]. Turn that into ((W,H,D,N), (C))
456:   // wgrad: C extents array contains [K,T,R,S,C]. Turn that into ((K), (C,S,R,T))
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 457-462
```cpp
457:   CUTLASS_HOST_DEVICE
458:   constexpr auto
459:   get_shape_C() const {
460:     using cute::make_shape;
461:     using cute::reverse;
462:     using cute::take;
```
**EN:** Defines function `get_shape_C` for this stage of the convolution workflow.

**CN:** 定义函数 `get_shape_C`，服务于卷积工作流的这一阶段。

### Lines 464-475
```cpp
464:     if constexpr (ConvOp == conv::Operator::kFprop ||
465:                   ConvOp == conv::Operator::kDgrad) {
466:       return make_shape(
467:         reverse(take<0, RankT - 1>(shape_C)),
468:         shape_C[RankT - 1]);
469:     }
470:     else if constexpr (ConvOp == conv::Operator::kWgrad) {
471:       return make_shape(
472:         shape_C[0],
473:         reverse(take<1, RankT>(shape_C)));
474:     }
475:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 477-487
```cpp
477:   // Static method that returns the canonical strides of tensors (layouts are right major and compact)
478:   CUTLASS_HOST_DEVICE
479:   static constexpr TensorStride
480:   packed_stride_right_major(TensorExtent const& extents) {
481:     TensorStride strides{};
482:     strides[RankT-1] = 1;
483:     cute::for_each(cute::make_rseq<RankT-1>{}, [&](auto i) {
484:       strides[i] = extents[i+1] * strides[i+1];
485:     });
486:     return strides;
487:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 489-498
```cpp
489:   // Static method that returns the packed logical size of any TensorExtent
490:   CUTLASS_HOST_DEVICE
491:   static constexpr size_t
492:   size(TensorExtent const& extents) {
493:     size_t size = 1;
494:     cute::for_each(cute::make_seq<RankT>{}, [&](auto i) {
495:       size *= extents[i];
496:     });
497:     return size;
498:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 500-504
```cpp
500:   CUTLASS_HOST_DEVICE
501:   constexpr size_t
502:   size_A() const {
503:     return shape_A[0] * stride_A[0];
504:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 506-510
```cpp
506:   CUTLASS_HOST_DEVICE
507:   constexpr size_t
508:   size_B() const {
509:     return shape_B[0] * stride_B[0];
510:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 512-516
```cpp
512:   CUTLASS_HOST_DEVICE
513:   constexpr size_t
514:   size_C() const {
515:     return shape_C[0] * stride_C[0];
516:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 518-522
```cpp
518:   // Equality operator
519:   CUTLASS_HOST_DEVICE
520:   bool operator==(ConvProblemShape<ConvOp, NumSpatialDimensions> const& rhs) const {
521:     using cute::for_each;
522:     using cute::make_seq;
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 524-524
```cpp
524:     bool is_equal = true;
```
**EN:** Stores member state such as `is_equal` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `is_equal` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 526-531
```cpp
526:     // Compare all tensor extents
527:     for_each(make_seq<RankT>{}, [&](auto i) {
528:       is_equal = is_equal
529:           && (shape_A[i] == rhs.shape_A[i])
530:           && (shape_B[i] == rhs.shape_B[i]);
531:     });
```
**EN:** Stores member state such as `is_equal` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `is_equal` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 533-540
```cpp
533:     // Compare all spatial extents
534:     for_each(make_seq<RankS>{}, [&](auto i) {
535:       is_equal = is_equal
536:           && (lower_padding[i] == rhs.lower_padding[i])
537:           && (upper_padding[i] == rhs.upper_padding[i])
538:           && (traversal_stride[i] == rhs.traversal_stride[i])
539:           && (dilation[i] == rhs.dilation[i]);
540:     });
```
**EN:** Stores member state such as `is_equal` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `is_equal` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 542-543
```cpp
542:     return is_equal;
543:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 545-549
```cpp
545:   /// Inequality operator
546:   CUTLASS_HOST_DEVICE
547:   bool operator!=(ConvProblemShape<ConvOp, NumSpatialDimensions> const &rhs) const {
548:     return !(*this == rhs);
549:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 551-555
```cpp
551: private:
552:   CUTLASS_HOST_DEVICE
553:   constexpr auto
554:   calculate_xformed_act(TensorExtent shape_act, TensorExtent shape_flt) {
555:     TensorExtent shape_xformed_act{};
```
**EN:** Defines function `calculate_xformed_act` for this stage of the convolution workflow.

**CN:** 定义函数 `calculate_xformed_act`，服务于卷积工作流的这一阶段。

### Lines 556-560
```cpp
556:     // calculate n,z,p,q,k.
557:     // a helper lambda to compute a single spatial extent of the nzpqk tensor
558:     auto nzpqk_extent = [](int act_ext, int filter_ext, int pad_total, int dilation, int tstride) {
559:       return 1 + (act_ext + pad_total - ((filter_ext -1) * dilation + 1)) / tstride;
560:     };
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 562-567
```cpp
562:     shape_xformed_act[0] = shape_act[0]; // Activation N extent
563:     cute::for_each(cute::make_seq<RankS>{}, [&](auto i) {
564:       shape_xformed_act[i+1] = nzpqk_extent(
565:           shape_act[i+1], shape_flt[i+1], upper_padding[i] + lower_padding[i], dilation[i], traversal_stride[i]);
566:       });
567:     shape_xformed_act[RankT-1] = shape_flt[0]; // Filter K extent
```
**EN:** Defines function `nzpqk_extent` for this stage of the convolution workflow.

**CN:** 定义函数 `nzpqk_extent`，服务于卷积工作流的这一阶段。

### Lines 569-569
```cpp
569:     TensorStride stride_xformed_act = packed_stride_right_major(shape_xformed_act);
```
**EN:** Defines function `packed_stride_right_major` for this stage of the convolution workflow.

**CN:** 定义函数 `packed_stride_right_major`，服务于卷积工作流的这一阶段。

### Lines 571-573
```cpp
571:     return cute::make_tuple(shape_xformed_act, stride_xformed_act);
572:   }
573: };
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 575-595
```cpp
575: template<
576:   conv::Operator ConvOp,
577:   int SpatialDim
578: >
579: void print(ConvProblemShape<ConvOp, SpatialDim> const& problem) {
580:   printf("ConvProblemShape with %d spatial dimensions implementing cutlass::conv::Operator::%d\n",
581:       SpatialDim, int(ConvOp));
582:   printf("\tTensorA: ");
583:       cute::print(problem.shape_A); printf(":");
584:       cute::print(problem.stride_A); printf("\n");
585:   printf("\tTensorB: ");
586:       cute::print(problem.shape_B); printf(":");
587:       cute::print(problem.stride_B); printf("\n");
588:   printf("\tTensorC: ");
589:       cute::print(problem.shape_C); printf(":");
590:       cute::print(problem.stride_C); printf("\n");
591:   printf("\tLower padding:     "); print(problem.lower_padding);       printf("\n");
592:   printf("\tUpper padding:     "); print(problem.upper_padding);       printf("\n");
593:   printf("\tTraversal strides: "); print(problem.traversal_stride);    printf("\n");
594:   printf("\tDilation:          "); print(problem.dilation);            printf("\n");
595: }
```
**EN:** Defines function `print` for this stage of the convolution workflow.

**CN:** 定义函数 `print`，服务于卷积工作流的这一阶段。

### Lines 599-599
```cpp
599: } // namespace cutlass::conv
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: This file contains definitions and utility functions for describing convolution problem shapes. **CN:** 核心作用：通过 N 维卷积 问题 形状 建模更一般的卷积问题形状。
- **EN:** Key exported symbols include `ConvProblemShape`, `SpatialExtent`, `TensorExtent`, `TensorStride`, `ShapePadding`, `TraversalStride`. **CN:** 关键导出符号包括 `ConvProblemShape`, `SpatialExtent`, `TensorExtent`, `TensorStride`, `ShapePadding`, `TraversalStride`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/tensor_coord.h`
- `cutlass/conv/convolution.h`
- `cute/container/array.hpp`
- `initializer_list`

### Internal Relationships / 内部关系
- **EN:** Uses CUTE metaprogramming and shape utilities. **CN:** 使用 CUTE 元编程与形状工具。
