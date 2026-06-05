# device_nhwc_pooling.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/device_nhwc_pooling.h`
- **Purpose (EN):** This file declares device nhwc pooling for general CUTLASS utility support.
- **目的 (CN):** 该文件声明了面向CUTLASS 通用工具支持的device nhwc pooling逻辑。
- **Brief / 简述:** cuda kernels to do avg/max pooling on a device memory tensor with NHWC layout.

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /******************************************************************************
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
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-30
```cpp
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  ******************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 32-32
```cpp
32: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 34-37
```cpp
34: /**
35:  * \file
36:  * \brief cuda kernels to do avg/max pooling on a device memory tensor with NHWC layout.
37:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 39-45
```cpp
39: #include "cutlass/cutlass.h"
40: #include "cutlass/layout/tensor.h"
41: #include "cutlass/numeric_types.h"
42: #include "cutlass/tensor_coord.h"
43: #include "cutlass/tensor_ref.h"
44: #include "device_utils.h"
45: #include <cfloat>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`。

### Lines 47-47
```cpp
47: namespace cutlass {
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 49-51
```cpp
49: /** \brief interface to do avg/max pooling on a device memory tensor with NHWC layout.
50:  * \tparam T: data type
51:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 52-61
```cpp
52: template <typename T>
53: void pooling_nhwc(cutlass::Tensor4DCoord input_tensor_size,
54:                   cutlass::Tensor4DCoord filter_tensor_size,
55:                   cutlass::Tensor4DCoord output_tensor_size,
56:                   cutlass::MatrixCoord padding,
57:                   cutlass::MatrixCoord stride,
58:                   TensorRef<T, layout::TensorNHWC> ref_input,
59:                   TensorRef<T, layout::TensorNHWC> ref_output,
60:                   int poolingType, //0 for avg pooling ; 1 for max pooling
61:                   cudaStream_t stream);
```
- **EN:** Declares or updates local/member state such as `pooling`.
- **CN:** 声明或更新局部/成员状态，例如 `pooling`。

### Lines 63-64
```cpp
63: /** get the output size of pooling
64:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 65-68
```cpp
65: inline int getOutputSize(int H_W, int padding, int kernel_size, int stride)
66: {
67:     return (H_W + 2 * padding - kernel_size) / stride + 1;
68: }
```
- **EN:** Implements `getOutputSize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `getOutputSize`。

### Lines 70-79
```cpp
70: /**
71:  * input is [N, H, W, C]
72:  * assume stride == kernel_size
73:  * output_h = (H + 2*padding_H - kernel_H)/stride_H
74:  * output_w = (W + 2*padding_W - kernel_W)/stride_W
75:  * output is [N, output_h, output_w, C]
76:  * grid(N, output_h, output_w)
77:  * block(min(C, 256)) :
78:  * each block deals with C elements of output when each thread deals with ((C + 255)/256 element of output)
79: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 80-93
```cpp
80: template<typename T, bool IS_AVG_POOLING>
81: __global__ void pooling_nhwc_element1_kernel(T* output,
82:                                              const T* input,
83:                                              const int N,
84:                                              const int H,
85:                                              const int W,
86:                                              const int C,
87:                                              const int output_H,
88:                                              const int output_W,
89:                                              const int kernel_H,
90:                                              const int kernel_W,
91:                                              const int stride_H,
92:                                              const int stride_W,
93:                                              const int padding_H,
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 94-99
```cpp
94:                                              const int padding_W)
95: {
96:   const int tid = threadIdx.x;
97:   const int n_idx = blockIdx.x;
98:   const int output_h_idx = blockIdx.y;
99:   const int output_w_idx = blockIdx.z;
```
- **EN:** Declares or updates local/member state such as `tid`, `x`, `n_idx`, `output_h_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `tid`, `x`, `n_idx`, `output_h_idx`。

### Lines 101-104
```cpp
101:   int h_start_idx = output_h_idx * stride_H - padding_H;
102:   int h_end_idx = h_start_idx + kernel_H;
103:   h_start_idx = (h_start_idx < 0) ? 0 : h_start_idx;
104:   h_end_idx = h_end_idx > H ? H : h_end_idx;
```
- **EN:** Declares or updates local/member state such as `h_start_idx`, `padding_H`, `h_end_idx`, `kernel_H`.
- **CN:** 声明或更新局部/成员状态，例如 `h_start_idx`, `padding_H`, `h_end_idx`, `kernel_H`。

### Lines 106-109
```cpp
106:   int w_start_idx = output_w_idx * stride_W - padding_W;
107:   int w_end_idx = w_start_idx + kernel_W;
108:   w_start_idx = (w_start_idx < 0) ? 0 : w_start_idx;
109:   w_end_idx = w_end_idx > W ? W : w_end_idx;
```
- **EN:** Declares or updates local/member state such as `w_start_idx`, `padding_W`, `w_end_idx`, `kernel_W`.
- **CN:** 声明或更新局部/成员状态，例如 `w_start_idx`, `padding_W`, `w_end_idx`, `kernel_W`。

### Lines 111-124
```cpp
111:   input += n_idx * H * W * C;
112:   output += ((n_idx * output_H + output_h_idx) * output_W + output_w_idx) * C;
113:   const int kernel_size2 = kernel_H * kernel_W;
114:   for (int c_idx = tid; c_idx < C; c_idx += blockDim.x) {
115:     float pooling;
116:     if (IS_AVG_POOLING){
117:       pooling = 0.0f;
118:     }
119:     else{
120:       pooling = -FLT_MAX;
121:     }
122:     for (int h = h_start_idx; h < h_end_idx; h++) {
123:       for (int w = w_start_idx; w < w_end_idx; w++) {
124:         const int idx = (h * W + w) * C;
```
- **EN:** Declares or updates local/member state such as `C`, `kernel_size2`, `kernel_W`, `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `C`, `kernel_size2`, `kernel_W`, `c_idx`。

### Lines 125-128
```cpp
125:         const float tmp = static_cast<float>(input[idx + c_idx]);
126:         if (IS_AVG_POOLING){
127:           pooling = pooling + tmp;
128:         }
```
- **EN:** Declares or updates local/member state such as `tmp`, `pooling`.
- **CN:** 声明或更新局部/成员状态，例如 `tmp`, `pooling`。

### Lines 129-133
```cpp
129:         else{
130:           pooling = pooling > tmp ? pooling : tmp;
131:         }
132:       }
133:     }
```
- **EN:** Declares or updates local/member state such as `pooling`, `tmp`.
- **CN:** 声明或更新局部/成员状态，例如 `pooling`, `tmp`。

### Lines 135-144
```cpp
135:     T output_val;
136:     if (IS_AVG_POOLING){
137:       output_val = T(pooling/kernel_size2);
138:     }
139:     else{
140:       output_val = T(pooling);
141:     }
142:     output[c_idx] = output_val;
143:   }
144: }
```
- **EN:** Declares or updates local/member state such as `output_val`.
- **CN:** 声明或更新局部/成员状态，例如 `output_val`。

### Lines 146-159
```cpp
146: template<typename T2, typename T, bool IS_AVG_POOLING>
147: __global__ void pooling_nhwc_element2_kernel(T2* output,
148:                                              const T2* input,
149:                                              const int N,
150:                                              const int H,
151:                                              const int W,
152:                                              const int C,
153:                                              const int output_H,
154:                                              const int output_W,
155:                                              const int kernel_H,
156:                                              const int kernel_W,
157:                                              const int stride_H,
158:                                              const int stride_W,
159:                                              const int padding_H,
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 160-165
```cpp
160:                                              const int padding_W)
161: {
162:   const int tid = threadIdx.x;
163:   const int n_idx = blockIdx.x;
164:   const int output_h_idx = blockIdx.y;
165:   const int output_w_idx = blockIdx.z;
```
- **EN:** Declares or updates local/member state such as `tid`, `x`, `n_idx`, `output_h_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `tid`, `x`, `n_idx`, `output_h_idx`。

### Lines 167-170
```cpp
167:   int h_start_idx = output_h_idx * stride_H - padding_H;
168:   int h_end_idx = h_start_idx + kernel_H;
169:   h_start_idx = (h_start_idx < 0) ? 0 : h_start_idx;
170:   h_end_idx = h_end_idx > H ? H : h_end_idx;
```
- **EN:** Declares or updates local/member state such as `h_start_idx`, `padding_H`, `h_end_idx`, `kernel_H`.
- **CN:** 声明或更新局部/成员状态，例如 `h_start_idx`, `padding_H`, `h_end_idx`, `kernel_H`。

### Lines 172-175
```cpp
172:   int w_start_idx = output_w_idx * stride_W - padding_W;
173:   int w_end_idx = w_start_idx + kernel_W;
174:   w_start_idx = (w_start_idx < 0) ? 0 : w_start_idx;
175:   w_end_idx = w_end_idx > W ? W : w_end_idx;
```
- **EN:** Declares or updates local/member state such as `w_start_idx`, `padding_W`, `w_end_idx`, `kernel_W`.
- **CN:** 声明或更新局部/成员状态，例如 `w_start_idx`, `padding_W`, `w_end_idx`, `kernel_W`。

### Lines 177-190
```cpp
177:   input += n_idx * H * W * C;
178:   output += ((n_idx * output_H + output_h_idx) * output_W + output_w_idx) * C;
179:   const int kernel_size2 = kernel_H * kernel_W;
180:   for (int c_idx = tid; c_idx < C; c_idx += blockDim.x) {
181:     float2 pooling;
182:     if (IS_AVG_POOLING) { 
183:       pooling = {0.0f, 0.0f};
184:     }
185:     else {
186:       pooling = {-FLT_MAX, -FLT_MAX};
187:     }
188:     for (int h = h_start_idx; h < h_end_idx; h++) {
189:       for (int w = w_start_idx; w < w_end_idx; w++) {
190:         const int idx = (h * W + w) * C;
```
- **EN:** Declares or updates local/member state such as `C`, `kernel_size2`, `kernel_W`, `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `C`, `kernel_size2`, `kernel_W`, `c_idx`。

### Lines 191-196
```cpp
191:         const T2 tmp = input[idx + c_idx];
192:         const float2 tmp_flt2 = {static_cast<float>(tmp.x), static_cast<float>(tmp.y)};
193:         if (IS_AVG_POOLING) {
194:           pooling.x += tmp_flt2.x;
195:           pooling.y += tmp_flt2.y;
196:         }
```
- **EN:** Declares or updates local/member state such as `tmp`, `tmp_flt2`, `x`, `y`.
- **CN:** 声明或更新局部/成员状态，例如 `tmp`, `tmp_flt2`, `x`, `y`。

### Lines 197-202
```cpp
197:         else {
198:           pooling.x = pooling.x > tmp_flt2.x ? pooling.x : tmp_flt2.x;
199:           pooling.y = pooling.y > tmp_flt2.y ? pooling.y : tmp_flt2.y;
200:         }
201:       }
202:     }
```
- **EN:** Declares or updates local/member state such as `x`, `y`.
- **CN:** 声明或更新局部/成员状态，例如 `x`, `y`。

### Lines 204-215
```cpp
204:     T2 output_val;
205:     if (IS_AVG_POOLING) {
206:       output_val.x = T(pooling.x/kernel_size2);
207:       output_val.y = T(pooling.y/kernel_size2);
208:     }
209:     else {
210:       output_val.x = T(pooling.x);
211:       output_val.y = T(pooling.y);
212:     }
213:     output[c_idx] = output_val;
214:   }
215: }
```
- **EN:** Declares or updates local/member state such as `output_val`, `x`, `y`.
- **CN:** 声明或更新局部/成员状态，例如 `output_val`, `x`, `y`。

### Lines 217-222
```cpp
217: /**
218:  * output [N, 1, 1, C]
219:  * input [N, H, W, C]
220:  * grid(C, N)
221:  * block(block_size) -- each block deals with H*W/block_size elements;
222: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 223-236
```cpp
223: template<typename T, bool IS_AVG_POOLING>
224: __global__ void pooling_nxhTo1x1_element1_kernel(
225:     T* output, const T* input, const int N, const int HW, const int C)
226: {
227:     const int c_idx = blockIdx.x;
228:     const int n_idx = blockIdx.y;
229:     float pooling[1];
230:     if (IS_AVG_POOLING) {
231:       pooling[0] = 0.0f;
232:     }
233:     else {
234:       pooling[0] = -FLT_MAX;
235:     }
236:     const size_t input_offset = n_idx * HW * C + c_idx;
```
- **EN:** Implements `pooling_nxhTo1x1_element1_kernel` for this file's main component.
- **CN:** 为该文件的核心组件实现 `pooling_nxhTo1x1_element1_kernel`。

### Lines 237-240
```cpp
237:     input += input_offset;
238:     const size_t output_offset = n_idx * C + c_idx;
239:     output += output_offset;
240:     int tid = threadIdx.x;
```
- **EN:** Declares or updates local/member state such as `input_offset`, `output_offset`, `c_idx`, `tid`.
- **CN:** 声明或更新局部/成员状态，例如 `input_offset`, `output_offset`, `c_idx`, `tid`。

### Lines 242-255
```cpp
242:     for (int index = tid; index < HW; index += blockDim.x) {
243:         float val = static_cast<float>(input[index * C]);
244:         if (IS_AVG_POOLING) {
245:           pooling[0] += val;
246:         }
247:         else {
248:           pooling[0] = pooling[0] > val ? pooling[0] : val;
249:         }
250:     }
251:     if (blockDim.x <= 32) {
252:         if (IS_AVG_POOLING) {
253:           warpReduceSum<float, 1>(pooling);
254:         }
255:         else {
```
- **EN:** Declares or updates local/member state such as `index`, `tid`, `HW`, `val`.
- **CN:** 声明或更新局部/成员状态，例如 `index`, `tid`, `HW`, `val`。

### Lines 256-259
```cpp
256:           warpReduceMax<float, 1>(pooling);
257:         }
258:     }
259:     else {
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 260-273
```cpp
260:         if (IS_AVG_POOLING) {
261:           blockReduceSum<float, 1>(pooling);
262:         }
263:         else {
264:           blockReduceMax<float, 1>(pooling);
265:         }
266:     }
267:     __syncthreads();
268:     if (threadIdx.x == 0) {
269:         T output_val;
270:         if (IS_AVG_POOLING) {
271:           output_val = T(pooling[0] / HW);
272:         }
273:         else {
```
- **EN:** Declares or updates local/member state such as `x`, `output_val`.
- **CN:** 声明或更新局部/成员状态，例如 `x`, `output_val`。

### Lines 274-277
```cpp
274:           output_val = T(pooling[0]);
275:         }
276:         output[0] = output_val;
277:     }
```
- **EN:** Implements `T` for this file's main component.
- **CN:** 为该文件的核心组件实现 `T`。

### Lines 278-278
```cpp
278: }
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 281-286
```cpp
281: /**
282:  * output [N, 1, 1, C]
283:  * input [N, H, W, C]
284:  * grid(C/2, N)
285:  * block(block_size) -- each thread deals with H*W/block_size * 2 elements;
286: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 287-300
```cpp
287: template<typename T2, typename T, bool IS_AVG_POOLING>
288: __global__ void pooling_nxhTo1x1_element2_kernel(
289:     T2* output, const T2* input, const int N, const int HW, const int C)
290: {
291:     const int c_idx = blockIdx.x;
292:     const int n_idx = blockIdx.y;
293:     float pooling[2];
294:     if (IS_AVG_POOLING) {
295:       pooling[0] = pooling[1] = 0.0f;
296:     }
297:     else {
298:       pooling[0] = pooling[1] = -FLT_MAX;
299:     }
300:     const int C_2 = C / 2;
```
- **EN:** Implements `pooling_nxhTo1x1_element2_kernel` for this file's main component.
- **CN:** 为该文件的核心组件实现 `pooling_nxhTo1x1_element2_kernel`。

### Lines 301-305
```cpp
301:     const size_t input_offset = n_idx * HW * C_2 + c_idx;
302:     input += input_offset;
303:     const size_t output_offset = n_idx * C_2 + c_idx;
304:     output += output_offset;
305:     int tid = threadIdx.x;
```
- **EN:** Declares or updates local/member state such as `input_offset`, `c_idx`, `output_offset`, `tid`.
- **CN:** 声明或更新局部/成员状态，例如 `input_offset`, `c_idx`, `output_offset`, `tid`。

### Lines 307-320
```cpp
307:     for (int index = tid; index < HW; index += blockDim.x) {
308:         T2 val = input[index * C_2];
309:         float2 val_flt2 = {static_cast<float>(val.x), static_cast<float>(val.y)};
310:         if (IS_AVG_POOLING) {
311:           pooling[0] += val_flt2.x;
312:           pooling[1] += val_flt2.y;
313:         }
314:         else {
315:           pooling[0] = pooling[0] > val_flt2.x ? pooling[0] : val_flt2.x;
316:           pooling[1] = pooling[1] > val_flt2.y ? pooling[1] : val_flt2.y;
317:         }
318:     }
319:     if (blockDim.x <= 32) {
320:         if (IS_AVG_POOLING) {
```
- **EN:** Declares or updates local/member state such as `index`, `tid`, `HW`, `val`.
- **CN:** 声明或更新局部/成员状态，例如 `index`, `tid`, `HW`, `val`。

### Lines 321-325
```cpp
321:           warpReduceSum<float, 2>(pooling);
322:         }
323:         else {
324:           warpReduceMax<float, 2>(pooling);
325:         }
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 326-339
```cpp
326:     }
327:     else {
328:         if (IS_AVG_POOLING) {
329:           blockReduceSum<float, 2>(pooling);
330:         }
331:         else {
332:           blockReduceMax<float, 2>(pooling);
333:         }
334:     }
335:     __syncthreads();
336:     if (threadIdx.x == 0) {
337:         T2 output_val;
338:         if (IS_AVG_POOLING) {
339:           output_val.x = T(pooling[0] / HW);
```
- **EN:** Declares or updates local/member state such as `x`, `output_val`.
- **CN:** 声明或更新局部/成员状态，例如 `x`, `output_val`。

### Lines 340-345
```cpp
340:           output_val.y = T(pooling[1] / HW);
341:         }
342:         else {
343:           output_val.x = T(pooling[0]);
344:           output_val.y = T(pooling[1]);
345:         }
```
- **EN:** Implements `T` for this file's main component.
- **CN:** 为该文件的核心组件实现 `T`。

### Lines 346-348
```cpp
346:         output[0] = output_val;
347:     }
348: }
```
- **EN:** Declares or updates local/member state such as `output_val`.
- **CN:** 声明或更新局部/成员状态，例如 `output_val`。

### Lines 350-359
```cpp
350: template <typename T>
351: void pooling_nhwc(cutlass::Tensor4DCoord input_tensor_size,
352:                   cutlass::Tensor4DCoord filter_tensor_size,
353:                   cutlass::Tensor4DCoord output_tensor_size,
354:                   cutlass::Tensor4DCoord padding,
355:                   cutlass::MatrixCoord stride,
356:                   TensorRef<T, layout::TensorNHWC> ref_input,
357:                   TensorRef<T, layout::TensorNHWC> ref_output,
358:                   int poolingType, //0 for avg pooling ; 1 for max pooling
359:                   cudaStream_t stream) {
```
- **EN:** Declares or updates local/member state such as `pooling`.
- **CN:** 声明或更新局部/成员状态，例如 `pooling`。

### Lines 361-362
```cpp
361:   assert(input_tensor_size.n() == output_tensor_size.n() &&
362:          input_tensor_size.c() == output_tensor_size.c());
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 364-373
```cpp
364:   const int N = input_tensor_size.n();
365:   const int H = input_tensor_size.h();
366:   const int W = input_tensor_size.w();
367:   const int C = input_tensor_size.c();
368:   const int padding_H = padding.h();
369:   const int padding_W = padding.w();
370:   const int kernel_H = filter_tensor_size.h();
371:   const int kernel_W = filter_tensor_size.w();
372:   const int stride_H = stride.row();
373:   const int stride_W = stride.column();
```
- **EN:** Implements `n` and coordinates helper calls such as `h`, `w`, `c`.
- **CN:** 实现 `n`，并协调调用 `h`, `w`, `c` 等辅助逻辑。

### Lines 375-376
```cpp
375:   const int output_H = getOutputSize(H, padding_H, kernel_H, stride_H);
376:   const int output_W = getOutputSize(W, padding_W, kernel_W, stride_W);
```
- **EN:** Implements `getOutputSize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `getOutputSize`。

### Lines 378-379
```cpp
378:   assert(output_tensor_size.h() == output_H &&
379:          output_tensor_size.w() == output_W);
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 381-394
```cpp
381:   if (C % 2 != 0) {
382:     if ((H == kernel_H && padding_H == 0) && (W == kernel_W && padding_W == 0)) {
383:       dim3 grid(C, N);
384:       dim3 block(256);
385:       if (H*W < block.x){
386:         block.x = (H*W + 31)/32*32;
387:       } 
388:       if (poolingType == 0) {
389:         pooling_nxhTo1x1_element1_kernel<T, true><<<grid, block, 0, stream>>>(
390:           ref_output.data(),
391:           ref_input.data(),
392:           N,
393:           H*W,
394:           C);
```
- **EN:** Declares or updates local/member state such as `H`, `padding_H`, `W`, `padding_W`.
- **CN:** 声明或更新局部/成员状态，例如 `H`, `padding_H`, `W`, `padding_W`。

### Lines 395-403
```cpp
395:       } // if (poolingType == 0)
396:       else {
397:         pooling_nxhTo1x1_element1_kernel<T, false><<<grid, block, 0, stream>>>(
398:           ref_output.data(),
399:           ref_input.data(),
400:           N,
401:           H*W,
402:           C);
403:       }
```
- **EN:** Declares or updates local/member state such as `poolingType`.
- **CN:** 声明或更新局部/成员状态，例如 `poolingType`。

### Lines 404-417
```cpp
404:     } // if ((H == kernel_H && padding_H == 0) && (W == kernel_W && padding_W == 0))
405:     else {
406:       dim3 grid(N, output_H, output_W);
407:       dim3 block(256);
408:       if (C < block.x) {
409:         block.x = C;
410:       }
411:       if (poolingType == 0) {
412:         pooling_nhwc_element1_kernel<T, true><<<grid, block, 0, stream>>>(
413:           ref_output.data(), 
414:           ref_input.data(),
415:           N,
416:           H,
417:           W,
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`, `data`.
- **CN:** 实现 `grid`，并协调调用 `block`, `data` 等辅助逻辑。

### Lines 418-428
```cpp
418:           C,
419:           output_H,
420:           output_W,
421:           kernel_H,
422:           kernel_W,
423:           stride_H,
424:           stride_W,
425:           padding_H,
426:           padding_W);
427:       } // if (poolingType == 0)
428:       else {
```
- **EN:** Declares or updates local/member state such as `poolingType`.
- **CN:** 声明或更新局部/成员状态，例如 `poolingType`。

### Lines 429-442
```cpp
429:         pooling_nhwc_element1_kernel<T, false><<<grid, block, 0, stream>>>(
430:           ref_output.data(),
431:           ref_input.data(),
432:           N,
433:           H,
434:           W,
435:           C,
436:           output_H,
437:           output_W,
438:           kernel_H,
439:           kernel_W,
440:           stride_H,
441:           stride_W,
442:           padding_H,
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 443-447
```cpp
443:           padding_W);
444:       }
445:     }
446:   } // if (C % 2 != 0))
447:   else {
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 448-461
```cpp
448:     if ((H == kernel_H && padding_H == 0) && (W == kernel_W && padding_W == 0)) {
449:       dim3 grid(C/2, N);
450:       dim3 block(256);
451:       if (H*W < block.x){
452:         block.x = (H*W + 31)/32*32;
453:       }
454:       if (poolingType == 0) {
455:         if (std::is_same<T, float>::value) {
456:           pooling_nxhTo1x1_element2_kernel<float2, float, true><<<grid, block, 0, stream>>>(
457:             (float2*)(ref_output.data()),
458:             (const float2*)(ref_input.data()),
459:             N,
460:             H*W,
461:             C);
```
- **EN:** Declares or updates local/member state such as `H`, `padding_H`, `W`, `padding_W`.
- **CN:** 声明或更新局部/成员状态，例如 `H`, `padding_H`, `W`, `padding_W`。

### Lines 462-470
```cpp
462:         } // if (std::is_same<T, float>::value)
463:         else {
464:           pooling_nxhTo1x1_element2_kernel<half2, half, true><<<grid, block, 0, stream>>>(
465:             (half2*)(ref_output.data()),
466:             (const half2*)(ref_input.data()),
467:             N,
468:             H*W,
469:             C);
470:         }
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 471-484
```cpp
471:       } // if (poolingType == 0)
472:       else {
473:         if (std::is_same<T, float>::value) {
474:           pooling_nxhTo1x1_element2_kernel<float2, float, false><<<grid, block, 0, stream>>>(
475:             (float2*)(ref_output.data()),
476:             (const float2*)(ref_input.data()),
477:             N,
478:             H*W,
479:             C);
480:         } // if (std::is_same<T, float>::value)
481:         else {
482:           pooling_nxhTo1x1_element2_kernel<half2, half, false><<<grid, block, 0, stream>>>(
483:             (half2*)(ref_output.data()),
484:             (const half2*)(ref_input.data()),
```
- **EN:** Declares or updates local/member state such as `poolingType`.
- **CN:** 声明或更新局部/成员状态，例如 `poolingType`。

### Lines 485-488
```cpp
485:             N,
486:             H*W,
487:             C);
488:         }
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 489-502
```cpp
489:       }
490:     } // if ((H == kernel_H && padding_H == 0) && (W == kernel_W && padding_W == 0))
491:     else {
492:       dim3 grid(N, output_H, output_W);
493:       dim3 block(256);
494:       if (C/2 < block.x) {
495:         block.x = C/2;
496:       }
497:       if (poolingType == 0) {
498:         if (std::is_same<T, float>::value) {
499:           pooling_nhwc_element2_kernel<float2, float, true><<<grid, block, 0, stream>>>(
500:             (float2*)(ref_output.data()),
501:             (const float2*)(ref_input.data()),
502:             N,
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`, `data`.
- **CN:** 实现 `grid`，并协调调用 `block`, `data` 等辅助逻辑。

### Lines 503-515
```cpp
503:             H,
504:             W,
505:             C/2,
506:             output_H,
507:             output_W,
508:             kernel_H,
509:             kernel_W,
510:             stride_H,
511:             stride_W,
512:             padding_H,
513:             padding_W);
514:         } // if (std::is_same<T, float>::value)
515:         else {
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 516-529
```cpp
516:           pooling_nhwc_element2_kernel<half2, half, true><<<grid, block, 0, stream>>>(
517:             (half2*)(ref_output.data()),
518:             (const half2*)(ref_input.data()),
519:             N,
520:             H,
521:             W,
522:             C/2,
523:             output_H,
524:             output_W,
525:             kernel_H,
526:             kernel_W,
527:             stride_H,
528:             stride_W,
529:             padding_H,
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 530-533
```cpp
530:             padding_W);
531:         }
532:       } // if (poolingType == 0)
533:       else {
```
- **EN:** Declares or updates local/member state such as `poolingType`.
- **CN:** 声明或更新局部/成员状态，例如 `poolingType`。

### Lines 534-547
```cpp
534:         if (std::is_same<T, float>::value) {
535:           pooling_nhwc_element2_kernel<float2, float, false><<<grid, block, 0, stream>>>(
536:             (float2*)(ref_output.data()),
537:             (const float2*)(ref_input.data()),
538:             N,
539:             H,
540:             W,
541:             C/2,
542:             output_H,
543:             output_W,
544:             kernel_H,
545:             kernel_W,
546:             stride_H,
547:             stride_W,
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 548-551
```cpp
548:             padding_H,
549:             padding_W);
550:         } // if (std::is_same<T, float>::value)
551:         else {
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 552-565
```cpp
552:           pooling_nhwc_element2_kernel<half2, half, false><<<grid, block, 0, stream>>>(
553:             (half2*)(ref_output.data()),
554:             (const half2*)(ref_input.data()),
555:             N,
556:             H,
557:             W,
558:             C/2,
559:             output_H,
560:             output_W,
561:             kernel_H,
562:             kernel_W,
563:             stride_H,
564:             stride_W,
565:             padding_H,
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 566-569
```cpp
566:             padding_W);
567:         }
568:       }
569:     }
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 570-571
```cpp
570:   }
571: }
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

### Lines 573-573
```cpp
573: } //namespace cutlass
```
- **EN:** Supporting logic for the device nhwc pooling implementation.
- **CN:** device nhwc pooling实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Tensor manipulation / 张量处理**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`
- **External headers / 外部头文件:** `device_utils.h`, `cfloat`
- **Runtime/backends / 运行时与后端:** `CUDA`
