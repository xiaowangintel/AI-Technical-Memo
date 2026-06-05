# conv2d_problem_size.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/conv2d_problem_size.h`
- **Purpose (EN):** This file contains definitions and utility functions for describing convolution problem sizes.
- **用途 (CN):** 定义围绕 二维卷积 问题 规模 的卷积问题规模描述与辅助映射。

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

### Lines 31-32
```cpp
 31: /*! \file
 32:     \brief This file contains definitions and utility functions for describing convolution problem sizes.
```
**EN:** Documents the file's purpose, terminology, and high-level usage.

**CN:** 说明文件的用途、术语以及高层使用方式。

### Lines 34-46
```cpp
 34:   Conv2dProblem desciption:
 35:     activation (NHWC), 
 36:     filter (KRSC), 
 37:     output (NPQK), 
 38:     pading (pad_h, pad_w),
 39:     stride (stride_h, stride_w),
 40:     dilation (dilation_h, dilation_w).
 42:   Free functions to map:
 43:     Map tensor extents (Conv2d -> ImplicitGemm)      : implicit_gemm_tensor_[a|b|c]_extent(ConvolutionOperator)
 44:     Map tensor sizes (Conv2d -> ImplicitGemm)        : implicit_gemm_tensor_[a|b|c]_size(ConvolutionOperator)
 45:     Map tensor problem sizes (Conv2d -> ImplicitGemm): implicit_gemm_problem_size(ConvolutionOperator)
 46: */
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 48-48
```cpp
 48: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 50-56
```cpp
 50: #include "cutlass/cutlass.h"
 51: #include "cutlass/tensor_coord.h"
 52: #include "cutlass/fast_math.h"
 53: #include "cutlass/gemm/gemm_enumerated_types.h"
 54: #include "cutlass/matrix_coord.h"
 55: #include "cutlass/conv/convolution.h"
 56: #include "cutlass/functional.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `tensor_coord.h`, `fast_math.h`, `gemm_enumerated_types.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `tensor_coord.h`, `fast_math.h`, `gemm_enumerated_types.h`。

### Lines 58-59
```cpp
 58: namespace cutlass {
 59: namespace conv {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 63-64
```cpp
 63: /// Problem size structure
 64: struct Conv2dProblemSize {
```
**EN:** Declares struct `Conv2dProblemSize`. The nearby comment explains that it serves the surrounding 2D convolution problem size logic.

**CN:** 声明结构体 `Conv2dProblemSize`，相邻注释说明它服务于周围的 二维卷积 问题 规模 逻辑。

### Lines 66-71
```cpp
 66:   // Conv2d strictly problem size parameters
 67:   int N, H, W, C, P, Q, K, R, S;
 68:   int pad_h, pad_w;
 69:   int stride_h, stride_w;
 70:   int dilation_h, dilation_w;
 71:   Mode mode;
```
**EN:** Stores member state such as `N`, `pad_h`, `stride_h`, `dilation_h`, `mode` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `N`, `pad_h`, `stride_h`, `dilation_h`, `mode` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 73-75
```cpp
 73:   // Conv2d implementation-related parameters 
 74:   int split_k_slices;
 75:   int groups;
```
**EN:** Stores member state such as `split_k_slices`, `groups` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `split_k_slices`, `groups` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 81-86
```cpp
 81: public:
 82:   CUTLASS_HOST_DEVICE
 83:   Conv2dProblemSize():
 84:     N(0), H(0), W(0), C(0), P(0), Q(0), K(0), R(0), S(0),
 85:     pad_h(0), pad_w(0), stride_h(1), stride_w(1), dilation_h(1), dilation_w(1),
 86:     mode(Mode::kConvolution), split_k_slices(1), groups(1) { }
```
**EN:** Provides constructor-style initialization for `Conv2dProblemSize`.

**CN:** 为 `Conv2dProblemSize` 提供构造式初始化逻辑。

### Lines 88-104
```cpp
 88:   /// Constructor for default padding, stride, dilation, and split-K
 89:   CUTLASS_HOST_DEVICE
 90:   Conv2dProblemSize(
 91:     int N,
 92:     int H,
 93:     int W,
 94:     int C,
 95:     int P,
 96:     int Q,
 97:     int K,
 98:     int R,
 99:     int S,
100:     Mode mode
101:   ): 
102:     N(N), H(H), W(W), C(C), P(P), Q(Q), K(K), R(R), S(S),
103:     pad_h(R / 2), pad_w(S / 2), stride_h(1), stride_w(1), dilation_h(1), dilation_w(1),
104:     mode(mode), split_k_slices(1), groups (1) { }
```
**EN:** Provides constructor-style initialization for `Conv2dProblemSize`.

**CN:** 为 `Conv2dProblemSize` 提供构造式初始化逻辑。

### Lines 106-131
```cpp
106:   /// Constructor
107:   CUTLASS_HOST_DEVICE
108:   Conv2dProblemSize(
109:     int N,
110:     int H,
111:     int W,
112:     int C,
113:     int K,
114:     int R,
115:     int S,
116:     int P,
117:     int Q,
118:     int pad_h,
119:     int pad_w,
120:     int stride_h,
121:     int stride_w,
122:     int dilation_h,
123:     int dilation_w,
124:     Mode mode,
125:     int split_k_slices = 1,
126:     int groups = 1
127:   ):
128:     N(N), H(H), W(W), C(C), P(P), Q(Q), K(K), R(R), S(S),
129:     pad_h(pad_h), pad_w(pad_w), stride_h(stride_h), stride_w(stride_w),
130:     dilation_h(dilation_h), dilation_w(dilation_w), 
131:     mode(mode), split_k_slices(split_k_slices), groups (groups) { }
```
**EN:** Provides constructor-style initialization for `Conv2dProblemSize`.

**CN:** 为 `Conv2dProblemSize` 提供构造式初始化逻辑。

### Lines 133-153
```cpp
133:   /// Constructs convolution problem size from cutlass Tensor4DCoord and MatrixCoord 
134:   // set user-defined output size and sets P and Q (include all data members in ctor)
135:   CUTLASS_HOST_DEVICE
136:   Conv2dProblemSize(
137:     cutlass::Tensor4DCoord input_size,    // NHWC
138:     cutlass::Tensor4DCoord filter_size,   // KRSC
139:     cutlass::Tensor4DCoord padding,       // pad_h, _, pad_w, _
140:     cutlass::MatrixCoord stride,          // stride_h, stride_w
141:     cutlass::MatrixCoord dilation,        // dilation_h, dilation_w
142:     cutlass::Tensor4DCoord output_size,   // NPQK
143:     cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation,
144:     int split_k_slices = 1,
145:     int groups = 1
146:   ):
147:     N(input_size.n()), H(input_size.h()), W(input_size.w()), C(input_size.c()),
148:     P(output_size.h()), Q(output_size.w()),
149:     K(filter_size.n()), R(filter_size.h()), S(filter_size.w()),
150:     pad_h(padding[0]), pad_w(padding[2]),
151:     stride_h(stride.row()), stride_w(stride.column()),
152:     dilation_h(dilation.row()), dilation_w(dilation.column()),
153:     mode(mode), split_k_slices(split_k_slices), groups(groups) {}
```
**EN:** Provides constructor-style initialization for `Conv2dProblemSize`.

**CN:** 为 `Conv2dProblemSize` 提供构造式初始化逻辑。

### Lines 155-173
```cpp
155:   /// Constructs convolution problem size from cutlass Tensor4DCoord and MatrixCoord 
156:   // computes output size and sets P and Q (skip output from ctor arguments)
157:   CUTLASS_HOST_DEVICE  
158:   Conv2dProblemSize(
159:     cutlass::Tensor4DCoord input_size,   // NHWC
160:     cutlass::Tensor4DCoord filter_size,  // KRSC
161:     cutlass::Tensor4DCoord padding,      // pad_h, upper_pad_h, pad_w, upper_pad_w
162:     cutlass::MatrixCoord stride,         // stride_h, stride_w
163:     cutlass::MatrixCoord dilation,       // dilation_h, dilation_w
164:     cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation,
165:     int split_k_slices = 1,
166:     int groups = 1
167:   ):
168:     N(input_size.n()), H(input_size.h()), W(input_size.w()), C(input_size.c()),
169:     K(filter_size.n()), R(filter_size.h()), S(filter_size.w()),
170:     pad_h(padding[0]), pad_w(padding[2]),
171:     stride_h(stride.row()), stride_w(stride.column()),
172:     dilation_h(dilation.row()), dilation_w(dilation.column()),
173:     mode(mode), split_k_slices(split_k_slices), groups(groups) {
```
**EN:** Provides constructor-style initialization for `Conv2dProblemSize`.

**CN:** 为 `Conv2dProblemSize` 提供构造式初始化逻辑。

### Lines 174-177
```cpp
174:       // set output P and Q
175:       P = ((H + pad_h + padding[1] - R * dilation_h) / stride_h) + 1;
176:       Q = ((W + pad_w + padding[3] - S * dilation_w) / stride_w) + 1;
177:     }
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 179-195
```cpp
179:   /// Constructs convolution problem size from cutlass Tensor4DCoord and MatrixCoord 
180:   // set user-defined output size and sets P and Q (skip padding, striding, and dilation)
181:   CUTLASS_HOST_DEVICE
182:   Conv2dProblemSize(
183:     cutlass::Tensor4DCoord input_size,    // NHWC
184:     cutlass::Tensor4DCoord filter_size,   // KRSC
185:     cutlass::Tensor4DCoord output_size,   // NPQK
186:     cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation,
187:     int split_k_slices = 1,
188:     int groups = 1
189:   ):
190:     N(input_size.n()), H(input_size.h()), W(input_size.w()), C(input_size.c()),
191:     P(output_size.h()), Q(output_size.w()),
192:     K(filter_size.n()), R(filter_size.h()), S(filter_size.w()),
193:     pad_h(R / 2), pad_w(S / 2), stride_h(1), stride_w(1),
194:     dilation_h(1), dilation_w(1),
195:     mode(mode), split_k_slices(split_k_slices), groups(groups) {}
```
**EN:** Provides constructor-style initialization for `Conv2dProblemSize`.

**CN:** 为 `Conv2dProblemSize` 提供构造式初始化逻辑。

### Lines 197-203
```cpp
197:   // Reset covolution mode in the problem
198:   CUTLASS_HOST_DEVICE
199:   Conv2dProblemSize reset_mode(cutlass::conv::Mode mode_) {
200:     Conv2dProblemSize tmp(*this);
201:     tmp.mode = mode_; 
202:     return tmp; 
203:   }
```
**EN:** Returns a modified copy with one runtime field changed.

**CN:** 返回一个修改了单个运行时字段的副本。

### Lines 205-211
```cpp
205:   // Reset covolution mode in the problem
206:   CUTLASS_HOST_DEVICE
207:   Conv2dProblemSize reset_split_k_slices(int split_k_slices_) {
208:     Conv2dProblemSize tmp(*this);
209:     tmp.split_k_slices = split_k_slices_; 
210:     return tmp; 
211:   }
```
**EN:** Returns a modified copy with one runtime field changed.

**CN:** 返回一个修改了单个运行时字段的副本。

### Lines 213-224
```cpp
213:   /// Equality operator (ignores mode and split_k_slice)
214:   CUTLASS_HOST_DEVICE
215:   bool operator==(Conv2dProblemSize const &conv) const {
216:     return (
217:       (N == conv.N) && (H == conv.H) && (W == conv.W) && (C == conv.C) &&
218:       (K == conv.K) && (R == conv.R) && (S == conv.S) &&
219:       (P == conv.P) && (Q == conv.Q) &&
220:       (pad_h == conv.pad_h) && (pad_w == conv.pad_w) &&
221:       (stride_h == conv.stride_h) && (stride_w == conv.stride_w) &&
222:       (dilation_h == conv.dilation_h) && (dilation_w == conv.dilation_w)
223:     );  
224:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 226-230
```cpp
226:   /// Inequality operator
227:   CUTLASS_HOST_DEVICE
228:   bool operator!=(Conv2dProblemSize const &rhs) const {
229:     return !(*this == rhs);
230:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 232-234
```cpp
232:   /// Returns activation extent as Tensor4DCoord
233:   CUTLASS_HOST_DEVICE
234:   cutlass::Tensor4DCoord activation_extent() const {
```
**EN:** Defines function `activation_extent` for this stage of the convolution workflow.

**CN:** 定义函数 `activation_extent`，服务于卷积工作流的这一阶段。

### Lines 236-237
```cpp
236:     return cutlass::Tensor4DCoord ({N, H, W, C});
237:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 239-241
```cpp
239:   /// Returns filter extent as Tensor4DCoord
240:   CUTLASS_HOST_DEVICE
241:   cutlass::Tensor4DCoord filter_extent(bool is_deconv = false) const {
```
**EN:** Defines function `filter_extent` for this stage of the convolution workflow.

**CN:** 定义函数 `filter_extent`，服务于卷积工作流的这一阶段。

### Lines 243-245
```cpp
243:     return is_deconv ? cutlass::Tensor4DCoord ({C, R, S, K / groups})
244:         : cutlass::Tensor4DCoord ({K, R, S, C / groups});
245:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 247-249
```cpp
247:   /// Returns output extent as Tensor4DCoord
248:   CUTLASS_HOST_DEVICE
249:   cutlass::Tensor4DCoord output_extent() const {
```
**EN:** Defines function `output_extent` for this stage of the convolution workflow.

**CN:** 定义函数 `output_extent`，服务于卷积工作流的这一阶段。

### Lines 251-252
```cpp
251:     return cutlass::Tensor4DCoord ({N, P, Q, K});
252:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 254-256
```cpp
254:   /// Returns activation size in number of elements
255:   CUTLASS_HOST_DEVICE
256:   int64_t activation_size() const {
```
**EN:** Defines function `activation_size` for this stage of the convolution workflow.

**CN:** 定义函数 `activation_size`，服务于卷积工作流的这一阶段。

### Lines 258-260
```cpp
258:     return static_cast<int64_t>(N) * static_cast<int64_t>(H) *
259:            static_cast<int64_t>(W) * static_cast<int64_t>(C);
260:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 262-264
```cpp
262:   /// Returns filter size in number of elements
263:   CUTLASS_HOST_DEVICE
264:   int64_t filter_size() const {
```
**EN:** Defines function `filter_size` for this stage of the convolution workflow.

**CN:** 定义函数 `filter_size`，服务于卷积工作流的这一阶段。

### Lines 266-269
```cpp
266:     return static_cast<int64_t>(K) * static_cast<int64_t>(R) *
267:            static_cast<int64_t>(S) * static_cast<int64_t>(C) /
268:            static_cast<int64_t>(groups);
269:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 271-273
```cpp
271:   /// Returns output size in number of elements
272:   CUTLASS_HOST_DEVICE
273:   int64_t output_size() const {
```
**EN:** Defines function `output_size` for this stage of the convolution workflow.

**CN:** 定义函数 `output_size`，服务于卷积工作流的这一阶段。

### Lines 275-277
```cpp
275:     return static_cast<int64_t>(N) * static_cast<int64_t>(P) *
276:            static_cast<int64_t>(Q) * static_cast<int64_t>(K);
277:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 279-281
```cpp
279:   /// Returns padding as Tensor4DCoord
280:   CUTLASS_HOST_DEVICE
281:   cutlass::Tensor4DCoord padding() const {
```
**EN:** Defines function `padding` for this stage of the convolution workflow.

**CN:** 定义函数 `padding`，服务于卷积工作流的这一阶段。

### Lines 283-284
```cpp
283:     return cutlass::Tensor4DCoord ({pad_h, pad_h, pad_w, pad_w});
284:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 286-288
```cpp
286:   /// Returns stride as MatrixCoord
287:   CUTLASS_HOST_DEVICE
288:   cutlass::MatrixCoord stride() const {
```
**EN:** Defines function `stride` for this stage of the convolution workflow.

**CN:** 定义函数 `stride`，服务于卷积工作流的这一阶段。

### Lines 290-291
```cpp
290:     return cutlass::MatrixCoord ({stride_h, stride_w});
291:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 293-295
```cpp
293:   /// Returns dilation as MatrixCoord
294:   CUTLASS_HOST_DEVICE
295:   cutlass::MatrixCoord dilation() const {
```
**EN:** Defines function `dilation` for this stage of the convolution workflow.

**CN:** 定义函数 `dilation`，服务于卷积工作流的这一阶段。

### Lines 297-298
```cpp
297:     return cutlass::MatrixCoord ({dilation_h, dilation_w});
298:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 301-307
```cpp
301:   //        Methods used for strided dgrad implementation
303:   /// Number of filter r positions to accumulate in gemm-k dim
304:   CUTLASS_HOST_DEVICE
305:   int num_gemm_k_filter_r(int r) const {
306:     return ((R - r + stride_h - 1) / stride_h);
307:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 309-313
```cpp
309:   /// Number of filter s positions to accumulate in gemm-k dim
310:   CUTLASS_HOST_DEVICE
311:   int num_gemm_k_filter_s(int s) const {
312:     return ((S - s + stride_w - 1) / stride_w);
313:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 315-320
```cpp
315:   /// Number of filter positions to accumulate in gemm-k dim
316:   CUTLASS_HOST_DEVICE
317:   int num_gemm_k_filter_positions(int r, int s) const {
318:     return num_gemm_k_filter_r(r) * num_gemm_k_filter_s(s);
319:   }
320: };
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 323-330
```cpp
323: //                                  ImplicitGemm helper functions                                 //
326: /// Determine the problem size of the implicit GEMM operation
327: CUTLASS_HOST_DEVICE
328: cutlass::gemm::GemmCoord implicit_gemm_problem_size(
329:   Operator conv_operator, 
330:   Conv2dProblemSize const &problem_size) {
```
**EN:** Defines function `implicit_gemm_problem_size` for this stage of the convolution workflow.

**CN:** 定义函数 `implicit_gemm_problem_size`，服务于卷积工作流的这一阶段。

### Lines 331-356
```cpp
331:   // Compute problem size
332:   switch (conv_operator) {
333:   case Operator::kFprop:
334:     return gemm::GemmCoord(
335:       problem_size.N * problem_size.P * problem_size.Q,
336:       problem_size.K,
337:       problem_size.R * problem_size.S * problem_size.C / problem_size.groups
338:     );
339:   case Operator::kDeconv:
340:   case Operator::kDgrad:
341:     return gemm::GemmCoord(
342:       problem_size.N * problem_size.H * problem_size.W,
343:       problem_size.C,
344:       problem_size.R * problem_size.S * problem_size.K
345:     );
346:   case Operator::kWgrad:
347:     return gemm::GemmCoord(
348:       problem_size.K,
349:       problem_size.R * problem_size.S * problem_size.C,
350:       problem_size.N * problem_size.P * problem_size.Q
351:     );
352:   default:
353:     break;
354:   }
355:   return gemm::GemmCoord();
356: }
```
**EN:** Provides constructor-style initialization for `GemmCoord`.

**CN:** 为 `GemmCoord` 提供构造式初始化逻辑。

### Lines 358-366
```cpp
358: // Determine the number of gemm_k iterations for conv2d problem using implicit gemm algorithm
359: CUTLASS_HOST_DEVICE
360: int implicit_gemm_k_iterations(
361:   Operator conv_operator, 
362:   int threadblock_K, 
363:   Conv2dProblemSize const &problem_size,
364:   IteratorAlgorithm algorithm = IteratorAlgorithm::kAnalytic,
365:   GroupMode group_mode = GroupMode::kNone,
366:   int threadblock_N = 0) {
```
**EN:** Defines function `implicit_gemm_k_iterations` for this stage of the convolution workflow.

**CN:** 定义函数 `implicit_gemm_k_iterations`，服务于卷积工作流的这一阶段。

### Lines 368-368
```cpp
368:   int iterations = 0;
```
**EN:** Stores member state such as `iterations` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterations` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 370-370
```cpp
370:   if (group_mode == GroupMode::kNone) {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 372-372
```cpp
372:     if (algorithm == IteratorAlgorithm::kFixedChannels) {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 374-378
```cpp
374:       int positions_per_iteration = threadblock_K / problem_size.C;
375:       switch (conv_operator) {
376:       case Operator::kFprop:
377:         iterations = (problem_size.R * problem_size.S + positions_per_iteration - 1 ) / positions_per_iteration;
378:         break;
```
**EN:** Stores member state such as `positions_per_iteration`, `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `positions_per_iteration`, `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 380-384
```cpp
380:       default:
381:         break;
382:       }
383:     }
384:     else if (algorithm == IteratorAlgorithm::kFewChannels) {
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 386-389
```cpp
386:       switch (conv_operator) {
387:       case Operator::kFprop:
388:         iterations = (problem_size.R * problem_size.S * problem_size.C + threadblock_K - 1 ) / threadblock_K;
389:         break;
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 391-396
```cpp
391:       default:
392:         break;
393:       }
394:     }
395:     else {
396:       int elements_per_split_k_slice = 0;
```
**EN:** Stores member state such as `break`, `elements_per_split_k_slice` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break`, `elements_per_split_k_slice` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 398-402
```cpp
398:       switch (conv_operator) {
399:       case Operator::kFprop:
400:         elements_per_split_k_slice = (problem_size.C + problem_size.split_k_slices - 1) / problem_size.split_k_slices;
401:         iterations = problem_size.R * problem_size.S * ((elements_per_split_k_slice + threadblock_K - 1) / threadblock_K);
402:         break;
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 404-408
```cpp
404:       case Operator::kDeconv:
405:       case Operator::kDgrad:
406:         elements_per_split_k_slice = (problem_size.K + problem_size.split_k_slices - 1) / problem_size.split_k_slices;
407:         iterations = problem_size.R * problem_size.S * ((elements_per_split_k_slice + threadblock_K - 1) / threadblock_K);
408:         break;
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 410-413
```cpp
410:       case Operator::kWgrad:
411:         elements_per_split_k_slice = (problem_size.N * problem_size.P * problem_size.Q + problem_size.split_k_slices - 1) / problem_size.split_k_slices;
412:         iterations = (elements_per_split_k_slice + threadblock_K - 1) / threadblock_K;
413:         break;
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 415-418
```cpp
415:       default:
416:         break;
417:       }
418:     }
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 420-421
```cpp
420:   } else if (group_mode == GroupMode::kDepthwise) {
421:     int channels_per_cta = threadblock_N;
```
**EN:** Stores member state such as `channels_per_cta` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `channels_per_cta` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 423-428
```cpp
423:     if (algorithm == IteratorAlgorithm::kAnalytic) {
424:       switch (conv_operator) {
425:         case Operator::kFprop:
426:           iterations = problem_size.R * problem_size.S *
427:                        ((channels_per_cta + threadblock_K - 1) / threadblock_K);
428:           break;
```
**EN:** Stores member state such as `iterations`, `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterations`, `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 430-434
```cpp
430:         default:
431:           break;
432:       }
433:     }
434:   } else {  // Group conv
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 436-437
```cpp
436:     int channels_per_group = problem_size.C / problem_size.groups;
437:     int k_per_group = problem_size.K / problem_size.groups;
```
**EN:** Stores member state such as `channels_per_group`, `k_per_group` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `channels_per_group`, `k_per_group` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 439-442
```cpp
439:     if (algorithm == IteratorAlgorithm::kAnalytic) {
440:       switch (conv_operator) {
441:         case Operator::kFprop:
442:           iterations = problem_size.R * problem_size.S * ((channels_per_group + threadblock_K - 1) / threadblock_K);
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 443-449
```cpp
443:           // In group conv, if k_per_group < threadblock_N, one Threadblock will calculate multiple groups
444:           if (problem_size.groups != 1) {
445:             if (k_per_group < threadblock_N) {
446:               iterations *= threadblock_N / k_per_group;
447:             }
448:           }
449:           break;
```
**EN:** Stores member state such as `k_per_group`, `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `k_per_group`, `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 451-454
```cpp
451:         default:
452:           break;
453:       }
454:     } else if (algorithm == IteratorAlgorithm::kOptimized) {
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 455-460
```cpp
455:       // Current optimized iterator only support GroupMode::kSingleGroup
456:       if (group_mode == GroupMode::kSingleGroup) {
457:         switch (conv_operator) {
458:           case Operator::kFprop:
459:             iterations = problem_size.R * problem_size.S * ((channels_per_group + threadblock_K - 1) / threadblock_K);
460:             break;
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 462-466
```cpp
462:           default:
463:             break;
464:         }
465:       }
466:     }
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 468-468
```cpp
468:   }
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 470-471
```cpp
470:   return iterations;
471: }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 474-482
```cpp
474: template <int N = 1, int Output_P = 1, int Output_Q = 1>
475: CUTLASS_HOST_DEVICE
476: int depthwise_gemm_k_iterations(
477:   Operator conv_operator, 
478:   int threadblock_K, 
479:   Conv2dProblemSize const &problem_size,
480:   IteratorAlgorithm algorithm = IteratorAlgorithm::kAnalytic,
481:   GroupMode group_mode = GroupMode::kNone,
482:   int threadblock_N = 0) {
```
**EN:** Defines function `depthwise_gemm_k_iterations` for this stage of the convolution workflow.

**CN:** 定义函数 `depthwise_gemm_k_iterations`，服务于卷积工作流的这一阶段。

### Lines 484-486
```cpp
484:     int n =  problem_size.N;
485:     int p = (problem_size.P + Output_P - 1) /  Output_P;
486:     int q = (problem_size.Q + Output_Q - 1) /  Output_Q;
```
**EN:** Stores member state such as `n` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `n` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 488-490
```cpp
488:     int iterations = (n * p * q + problem_size.split_k_slices - 1) / problem_size.split_k_slices;
489:     return iterations;
490: }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 493-497
```cpp
493: CUTLASS_HOST_DEVICE
494: int implicit_gemm_k_iterations_per_channel(
495:     Operator conv_operator,
496:     Conv2dProblemSize const &problem_size,
497:     IteratorAlgorithm algorithm = IteratorAlgorithm::kAnalytic) {
```
**EN:** Defines function `implicit_gemm_k_iterations_per_channel` for this stage of the convolution workflow.

**CN:** 定义函数 `implicit_gemm_k_iterations_per_channel`，服务于卷积工作流的这一阶段。

### Lines 499-504
```cpp
499:   int iterations = 0; //0 means not applicable
500:   if (algorithm == IteratorAlgorithm::kAnalytic || algorithm == IteratorAlgorithm::kOptimized) {
501:     switch (conv_operator) {
502:       case Operator::kFprop:
503:         iterations = problem_size.R * problem_size.S;
504:         break;
```
**EN:** Stores member state such as `iterations`, `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterations`, `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 506-509
```cpp
506:       case Operator::kDeconv:
507:       case Operator::kDgrad:
508:         iterations = problem_size.R * problem_size.S;
509:         break;
```
**EN:** Stores member state such as `iterations`, `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterations`, `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 511-516
```cpp
511:       default:
512:         break;
513:     }
514:   }
515:   return iterations;
516: }
```
**EN:** Stores member state such as `break` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `break` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 519-534
```cpp
519: //  Mapping function (ImplicitGemm A, B, C -> Conv Activation, Filter, Output)
521: /// Returns ImplicitGemm tensor A extent as Tensor4DCoord
522: CUTLASS_HOST_DEVICE
523: cutlass::Tensor4DCoord implicit_gemm_tensor_a_extent(
524:   Operator conv_operator,
525:   Conv2dProblemSize const &problem_size) {
526:   switch (conv_operator) {
527:     case cutlass::conv::Operator::kFprop: return problem_size.activation_extent();
528:     case cutlass::conv::Operator::kDeconv:
529:     case cutlass::conv::Operator::kDgrad: return problem_size.output_extent();
530:     case cutlass::conv::Operator::kWgrad: return problem_size.output_extent();
531:     default : break;
532:   }
533:   return cutlass::Tensor4DCoord();
534: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 536-549
```cpp
536: /// Returns ImplicitGemm tensor B extent as Tensor4DCoord
537: CUTLASS_HOST_DEVICE
538: cutlass::Tensor4DCoord implicit_gemm_tensor_b_extent(
539:   Operator conv_operator,
540:   Conv2dProblemSize const &problem_size) {
541:   switch (conv_operator) {
542:     case cutlass::conv::Operator::kFprop: return problem_size.filter_extent();
543:     case cutlass::conv::Operator::kDeconv: return problem_size.filter_extent(true);
544:     case cutlass::conv::Operator::kDgrad: return problem_size.filter_extent();
545:     case cutlass::conv::Operator::kWgrad: return problem_size.activation_extent();
546:     default : break;
547:   }
548:   return cutlass::Tensor4DCoord();
549: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 551-564
```cpp
551: /// Returns ImplicitGemm tensor C extent as Tensor4DCoord
552: CUTLASS_HOST_DEVICE
553: cutlass::Tensor4DCoord implicit_gemm_tensor_c_extent(
554:   Operator conv_operator,
555:   Conv2dProblemSize const &problem_size) {
556:   switch (conv_operator) {
557:     case cutlass::conv::Operator::kFprop: return problem_size.output_extent();
558:     case cutlass::conv::Operator::kDeconv:
559:     case cutlass::conv::Operator::kDgrad: return problem_size.activation_extent();
560:     case cutlass::conv::Operator::kWgrad: return problem_size.filter_extent();
561:     default : break;
562:   }
563:   return cutlass::Tensor4DCoord();
564: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 566-579
```cpp
566: /// Returns ImplicitGemm tensor A size in number of elements
567: CUTLASS_HOST_DEVICE
568: int64_t implicit_gemm_tensor_a_size(
569:   Operator conv_operator,
570:   Conv2dProblemSize const &problem_size) {
571:   switch (conv_operator) {
572:     case cutlass::conv::Operator::kFprop: return problem_size.activation_size();
573:     case cutlass::conv::Operator::kDeconv:
574:     case cutlass::conv::Operator::kDgrad: return problem_size.output_size();
575:     case cutlass::conv::Operator::kWgrad: return problem_size.output_size();
576:     default : break;
577:   }
578:   return 0;
579: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 581-594
```cpp
581: /// Returns ImplicitGemm tensor B size in number of elements
582: CUTLASS_HOST_DEVICE
583: int64_t implicit_gemm_tensor_b_size(
584:   Operator conv_operator,
585:   Conv2dProblemSize const &problem_size) {
586:   switch (conv_operator) {
587:     case cutlass::conv::Operator::kFprop: return problem_size.filter_size();
588:     case cutlass::conv::Operator::kDeconv:
589:     case cutlass::conv::Operator::kDgrad: return problem_size.filter_size();
590:     case cutlass::conv::Operator::kWgrad: return problem_size.activation_size();
591:     default : break;
592:   }
593:   return 0;
594: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 596-609
```cpp
596: /// Returns ImplicitGemm tensor C size in number of elements
597: CUTLASS_HOST_DEVICE
598: int64_t implicit_gemm_tensor_c_size(
599:   Operator conv_operator,
600:   Conv2dProblemSize const &problem_size) {
601:   switch (conv_operator) {
602:     case cutlass::conv::Operator::kFprop: return problem_size.output_size();
603:     case cutlass::conv::Operator::kDeconv:
604:     case cutlass::conv::Operator::kDgrad: return problem_size.activation_size();
605:     case cutlass::conv::Operator::kWgrad: return problem_size.filter_size();
606:     default : break;
607:   }
608:   return 0;
609: }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 614-620
```cpp
614: //                                  Strided dgrad helper functions                                 //
616: // Returns number of CTAs tile M to cover valid MMAs per starting filter postion
617: CUTLASS_HOST_DEVICE
618: int strided_dgrad_tile_m_per_filter(
619:   Conv2dProblemSize const &problem_size,
620:   int tile_size_m) {
```
**EN:** Defines function `strided_dgrad_tile_m_per_filter` for this stage of the convolution workflow.

**CN:** 定义函数 `strided_dgrad_tile_m_per_filter`，服务于卷积工作流的这一阶段。

### Lines 622-625
```cpp
622:   // Compute NHW rows in Dx output that needs MMA per starting filter position
623:   int rows_h_per_filter = (problem_size.H + problem_size.stride_h - 1) / problem_size.stride_h;
624:   int rows_w_per_filter = (problem_size.W + problem_size.stride_w - 1) / problem_size.stride_w;
625:   int rows_nhw_per_filter = problem_size.N * rows_h_per_filter * rows_w_per_filter;
```
**EN:** Stores member state such as `rows_nhw_per_filter` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `rows_nhw_per_filter` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 627-628
```cpp
627:   // Number of CTAs tile M to cover valid MMAs per starting filter postion
628:   int tile_m_per_filter = (rows_nhw_per_filter + tile_size_m - 1) / tile_size_m;
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 630-631
```cpp
630:   return tile_m_per_filter;
631: }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 633-639
```cpp
633: // Computes starting Dx coord (h, w) for given starting filter postion
634: CUTLASS_HOST_DEVICE
635: void strided_dgrad_starting_coords(
636:   Conv2dProblemSize const &problem_size,
637:   FastDivmod const &stride_h_divmod, FastDivmod const &stride_w_divmod,
638:   int r, int s,
639:   int &start_h, int &start_w) {
```
**EN:** Defines function `strided_dgrad_starting_coords` for this stage of the convolution workflow.

**CN:** 定义函数 `strided_dgrad_starting_coords`，服务于卷积工作流的这一阶段。

### Lines 641-642
```cpp
641:   // function locals for remainder by fast divmod
642:   int pad_h_rem_, pad_w_rem_;
```
**EN:** Stores member state such as `pad_h_rem_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `pad_h_rem_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 644-647
```cpp
644:   // start_h  = std::abs(problem_size.stride_h - ((problem_size.pad_h % problem_size.stride_h) - r)) % problem_size.stride_h;
645:   stride_h_divmod.divmod(pad_h_rem_, problem_size.pad_h);
646:   int r_ = absolute_value(problem_size.stride_h - (pad_h_rem_ - r));
647:   stride_h_divmod.divmod(start_h, r_);
```
**EN:** Defines function `divmod` for this stage of the convolution workflow.

**CN:** 定义函数 `divmod`，服务于卷积工作流的这一阶段。

### Lines 649-653
```cpp
649:   //start_w  = std::abs(problem_size.stride_w - ((problem_size.pad_w % problem_size.stride_w) - s)) % problem_size.stride_w;
650:   stride_w_divmod.divmod(pad_w_rem_, problem_size.pad_w);
651:   int s_ = absolute_value(problem_size.stride_w - (pad_w_rem_ - s));
652:   stride_w_divmod.divmod(start_w, s_);
653: }
```
**EN:** Defines function `divmod` for this stage of the convolution workflow.

**CN:** 定义函数 `divmod`，服务于卷积工作流的这一阶段。

### Lines 655-656
```cpp
655: } // namespace conv
656: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: This file contains definitions and utility functions for describing convolution problem sizes. **CN:** 核心作用：定义围绕 二维卷积 问题 规模 的卷积问题规模描述与辅助映射。
- **EN:** Key exported symbols include `Conv2dProblemSize`. **CN:** 关键导出符号包括 `Conv2dProblemSize`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** Precomputed parameter objects reduce runtime address arithmetic in hot loops. **CN:** 预计算参数对象可以减少热点循环中的运行时地址计算。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/tensor_coord.h`
- `cutlass/fast_math.h`
- `cutlass/gemm/gemm_enumerated_types.h`
- `cutlass/matrix_coord.h`
- `cutlass/conv/convolution.h`
- `cutlass/functional.h`

### Internal Relationships / 内部关系
- **EN:** Builds on CUTLASS GEMM shapes, policies, or operators. **CN:** 构建在 CUTLASS GEMM 的形状、策略或算子之上。
