# AdaptiveMaxPoolKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/AdaptiveMaxPoolKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU pooling kernels and pooled-output shape or reduction logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 池化 kernel，以及池化输出形状或归约逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3:
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/native/AdaptivePooling.h>
 6: #include <ATen/Parallel.h>
 7: #include <ATen/cpu/vec/vec.h>
 8: #include <ATen/cpu/vec/functional.h>
 9: #include <ATen/native/cpu/utils.h>
10: #include <c10/util/irange.h>
11: #include <ATen/OpMathType.h>
12:
13: namespace at::native {
14:
15: namespace {
16:
17: template <typename scalar_t, typename accscalar_t>
18: void cpu_adaptive_max_pool2d(
19:     const Tensor& output_,
20:     const Tensor& indices_,
21:     const Tensor& input_,
22:     IntArrayRef output_size) {
23:   auto input = input_.contiguous();
24:   auto output = output_.contiguous();
25:   auto indices = indices_.contiguous();
26:
27:   auto input_data = input.const_data_ptr<scalar_t>();
28:   auto output_data = output.data_ptr<scalar_t>();
29:   auto indices_data = indices.data_ptr<int64_t>();
30:
31:   int64_t ndim = input.ndimension();
32:   // treat batch size and channels as one dimension
33:   int64_t channels = ndim == 3 ? input.size(0) : input.size(0) * input.size(1);
34:   int64_t input_height = input.size(-2);
35:   int64_t input_width = input.size(-1);
36:   int64_t output_height = output_size[0];
37:   int64_t output_width = output_size[1];
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/native/AdaptivePooling.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/native/AdaptivePooling.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 39-74
```cpp
39:   // parallel on dim of N, C
40:   at::parallel_for(0, channels, 0, [&](int64_t begin, int64_t end) {
41:     for (const auto c : c10::irange(begin, end)) {
42:       const scalar_t* input_ptr = input_data + c * input_height * input_width;
43:       scalar_t* output_ptr = output_data + c * output_height * output_width;
44:       int64_t* indices_ptr = indices_data + c * output_height * output_width;
45:
46:       for (const auto oh : c10::irange(output_height)) {
47:         int64_t ih0 = start_index(oh, output_height, input_height);
48:         int64_t ih1 = end_index(oh, output_height, input_height);
49:
50:         for (const auto ow : c10::irange(output_width)) {
51:           int64_t iw0 = start_index(ow, output_width, input_width);
52:           int64_t iw1 = end_index(ow, output_width, input_width);
53:
54:           // compute local max
55:           int64_t maxindex = ih0 * input_width + iw0;
56:           accscalar_t maxval = -std::numeric_limits<accscalar_t>::infinity();
57:           for (int64_t ih = ih0; ih < ih1; ih ++) {
58:             for (int64_t iw = iw0; iw < iw1; iw ++) {
59:               int64_t index = ih * input_width + iw;
60:               scalar_t val = input_ptr[index];
61:               if ((val > maxval) || std::isnan(val)) {
62:                 maxval = val;
63:                 maxindex = index;
64:               }
65:             }
66:           }
67:
68:           // set output to local max and store location of max
69:           output_ptr[oh * output_width + ow] = maxval;
70:           indices_ptr[oh * output_width + ow] = maxindex;
71:         }
72:       }
73:     }
74:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 76-107
```cpp
 76:   if (!output_.is_contiguous()) {
 77:     output_.copy_(output);
 78:   }
 79:   if (!indices_.is_contiguous()) {
 80:     indices_.copy_(indices);
 81:   }
 82: }
 83:
 84: template <typename scalar_t>
 85: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
 86: cpu_adaptive_max_pool2d_channels_last(
 87:     const Tensor& output_,
 88:     const Tensor& indices_,
 89:     const Tensor& input_,
 90:     IntArrayRef output_size) {
 91:   TORCH_CHECK(input_.ndimension() == 4,
 92:               "2d adaptive max pooling with channels last format supports tensors with 4 dims");
 93:   auto memory_format = at::MemoryFormat::ChannelsLast;
 94:   auto input = input_.contiguous(memory_format);
 95:   auto output = output_.contiguous(memory_format);
 96:   auto indices = indices_.contiguous(memory_format);
 97:
 98:   auto input_data = input.const_data_ptr<scalar_t>();
 99:   auto output_data = output.data_ptr<scalar_t>();
100:   auto indices_data = indices.data_ptr<int64_t>();
101:
102:   int64_t nbatch = input.size(0);
103:   int64_t channels = input.size(1);
104:   int64_t input_height = input.size(2);
105:   int64_t input_width = input.size(3);
106:   int64_t output_height = output_size[0];
107:   int64_t output_width = output_size[1];
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool2d_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool2d_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 109-137
```cpp
109:   using Vec = vec::Vectorized<scalar_t>;
110:   using integer_t = vec::int_same_size_t<scalar_t>;
111:   using iVec = vec::Vectorized<integer_t>;
112:   // for the convenience of vectorization, use integer of the same size of scalar_t,
113:   //   e.g. int32_t for float, int64_t for double
114:   // need to make sure doesn't overflow
115:   TORCH_CHECK(input_height * input_width <= std::numeric_limits<integer_t>::max());
116:
117:   // parallel on dim of N, H, W
118:   at::parallel_for(0, nbatch * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
119:     int64_t n = 0;
120:     int64_t oh = 0;
121:     int64_t ow = 0;
122:     data_index_init(begin, n, nbatch, oh, output_height, ow, output_width);
123:
124:     int64_t size = channels;
125:     int64_t len = size - (size % Vec::size());
126:     // temp buffer holding index with integer_t
127:     auto index_buffer = std::make_unique<integer_t []>(len);
128:
129:     for (const auto i : c10::irange(begin, end)) {
130:       int64_t ih0 = start_index(oh, output_height, input_height);
131:       int64_t ih1 = end_index(oh, output_height, input_height);
132:
133:       int64_t iw0 = start_index(ow, output_width, input_width);
134:       int64_t iw1 = end_index(ow, output_width, input_width);
135:
136:       scalar_t* out = output_data + i * channels;
137:       int64_t* ind = indices_data + i * channels;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 139-168
```cpp
139:       // Pass I: init out lane
140:       iVec index0_vec = iVec(ih0 * input_width + iw0);
141:       Vec out_vec = Vec(-std::numeric_limits<scalar_t>::infinity());
142:       int64_t d1 = 0;
143:       for (; d1 < len; d1 += Vec::size()) {
144:         index0_vec.store(index_buffer.get() + d1);
145:         out_vec.store(out + d1);
146:       }
147:       for (; d1 < size; d1++) {
148:         ind[d1] = ih0 * input_width + iw0;
149:         out[d1] = -std::numeric_limits<scalar_t>::infinity();
150:       }
151:       // Pass II: compute local max
152:       for (int64_t ih = ih0; ih < ih1; ih ++) {
153:         for (int64_t iw = iw0; iw < iw1; iw ++) {
154:           const scalar_t* in = input_data + n * input_height * input_width * channels +
155:               ih * input_width * channels + iw * channels;
156:
157:           int64_t d2 = 0;
158:           for (; d2 < len; d2 += Vec::size()) {
159:             iVec index_vec = iVec(ih * input_width + iw);
160:             Vec val_vec = Vec::loadu(in + d2);
161:             iVec maxindex_vec = iVec::loadu(index_buffer.get() + d2);
162:             Vec maxval_vec = Vec::loadu(out + d2);
163:
164:             // true = all ones, false = all zeros
165:             Vec mask = (val_vec > maxval_vec) | val_vec.isnan();
166:             iVec imask = vec::cast<integer_t>(mask);
167:             Vec out_vec = Vec::blendv(maxval_vec, val_vec, mask);
168:             iVec ind_vec = iVec::blendv(maxindex_vec, index_vec, imask);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 170-199
```cpp
170:             out_vec.store(out + d2);
171:             ind_vec.store(index_buffer.get() + d2);
172:           }
173:           for (; d2 < size; d2++) {
174:             int64_t index = ih * input_width + iw;
175:             scalar_t val = in[d2];
176:             int64_t maxindex = ind[d2];
177:             scalar_t maxval = out[d2];
178:
179:             bool mask = (val > maxval) || std::isnan(val);
180:             out[d2] = mask ? val : maxval;
181:             ind[d2] = mask ? index : maxindex;
182:           }
183:         }
184:       }
185:       // convert indice data type
186:       vec::convert<integer_t, int64_t>(index_buffer.get(), ind, len);
187:
188:       // move on to next output index
189:       data_index_step(n, nbatch, oh, output_height, ow, output_width);
190:     }
191:   });
192:
193:   if (!output_.is_contiguous(memory_format)) {
194:     output_.copy_(output);
195:   }
196:   if (!indices_.is_contiguous(memory_format)) {
197:     indices_.copy_(indices);
198:   }
199: }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 201-230
```cpp
201: template <typename scalar_t>
202: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
203: cpu_adaptive_max_pool2d_channels_last(
204:     const Tensor& output_,
205:     const Tensor& indices_,
206:     const Tensor& input_,
207:     IntArrayRef output_size) {
208:   TORCH_CHECK(input_.ndimension() == 4,
209:               "2d adaptive max pooling with channels last format supports tensors with 4 dims");
210:   auto memory_format = at::MemoryFormat::ChannelsLast;
211:   auto input = input_.contiguous(memory_format);
212:   auto output = output_.contiguous(memory_format);
213:   auto indices = indices_.contiguous(memory_format);
214:
215:   auto input_data = input.const_data_ptr<scalar_t>();
216:   auto output_data = output.data_ptr<scalar_t>();
217:   auto indices_data = indices.data_ptr<int64_t>();
218:
219:   int64_t nbatch = input.size(0);
220:   int64_t channels = input.size(1);
221:   int64_t input_height = input.size(2);
222:   int64_t input_width = input.size(3);
223:   int64_t output_height = output_size[0];
224:   int64_t output_width = output_size[1];
225:
226:   using bVec = vec::Vectorized<scalar_t>;
227:   using fVec = vec::Vectorized<float>;
228:   using iVec = vec::Vectorized<int32_t>;
229:   // need to make sure doesn't overflow
230:   TORCH_CHECK(input_height * input_width <= std::numeric_limits<int32_t>::max());
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool2d_channels_last`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool2d_channels_last`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 232-273
```cpp
232:   // parallel on dim of N, H, W
233:   at::parallel_for(0, nbatch * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
234:     int64_t n = 0;
235:     int64_t oh = 0;
236:     int64_t ow = 0;
237:     data_index_init(begin, n, nbatch, oh, output_height, ow, output_width);
238:
239:     int64_t size = channels;
240:     int64_t len = size - (size % bVec::size());
241:     // temp buffer holding index with integer_t
242:     auto index_buffer = std::make_unique<int32_t []>(len);
243:     // temp buffer holding max value with float
244:     auto max_arr = std::make_unique<float []>(size);
245:     float* max = max_arr.get();
246:
247:     for (const auto i : c10::irange(begin, end)) {
248:       int64_t ih0 = start_index(oh, output_height, input_height);
249:       int64_t ih1 = end_index(oh, output_height, input_height);
250:
251:       int64_t iw0 = start_index(ow, output_width, input_width);
252:       int64_t iw1 = end_index(ow, output_width, input_width);
253:
254:       scalar_t* out = output_data + i * channels;
255:       int64_t* ind = indices_data + i * channels;
256:
257:       // Pass I: init out lane
258:       iVec index0_ivec = iVec(ih0 * input_width + iw0);
259:       fVec max_fvec = fVec(-std::numeric_limits<float>::infinity());
260:       int64_t d1 = 0;
261:       for (; d1 < len; d1 += fVec::size()) {
262:         index0_ivec.store(index_buffer.get() + d1);
263:         max_fvec.store(max + d1);
264:       }
265:       for (; d1 < size; d1++) {
266:         ind[d1] = ih0 * input_width + iw0;
267:         max[d1] = -std::numeric_limits<float>::infinity();
268:       }
269:       // Pass II: compute local max
270:       for (int64_t ih = ih0; ih < ih1; ih ++) {
271:         for (int64_t iw = iw0; iw < iw1; iw ++) {
272:           const scalar_t* in = input_data + n * input_height * input_width * channels +
273:               ih * input_width * channels + iw * channels;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 275-306
```cpp
275:           int64_t d2 = 0;
276:           for (; d2 < len; d2 += bVec::size()) {
277:             iVec index_ivec = iVec(ih * input_width + iw);
278:             bVec val_bvec = bVec::loadu(in + d2);
279:             auto [val_fvec0, val_fvec1] = convert_to_float<scalar_t>(val_bvec);
280:
281:             iVec maxindex_ivec0 = iVec::loadu(index_buffer.get() + d2);
282:             iVec maxindex_ivec1 = iVec::loadu(index_buffer.get() + d2 + iVec::size());
283:             fVec maxval_fvec0 = fVec::loadu(max + d2);
284:             fVec maxval_fvec1 = fVec::loadu(max + d2 + fVec::size());
285:
286:             // true = all ones, false = all zeros
287:             fVec mask0 = (val_fvec0 > maxval_fvec0) | val_fvec0.isnan();
288:             fVec mask1 = (val_fvec1 > maxval_fvec1) | val_fvec1.isnan();
289:             iVec imask0 = vec::cast<int32_t>(mask0);
290:             iVec imask1 = vec::cast<int32_t>(mask1);
291:
292:             fVec max_fvec0 = fVec::blendv(maxval_fvec0, val_fvec0, mask0);
293:             fVec max_fvec1 = fVec::blendv(maxval_fvec1, val_fvec1, mask1);
294:             iVec ind_ivec0 = iVec::blendv(maxindex_ivec0, index_ivec, imask0);
295:             iVec ind_ivec1 = iVec::blendv(maxindex_ivec1, index_ivec, imask1);
296:
297:             max_fvec0.store(max + d2);
298:             max_fvec1.store(max + d2 + fVec::size());
299:             ind_ivec0.store(index_buffer.get() + d2);
300:             ind_ivec1.store(index_buffer.get() + d2 + iVec::size());
301:           }
302:           for (; d2 < size; d2++) {
303:             int64_t index = ih * input_width + iw;
304:             float val = float(in[d2]);
305:             int64_t maxindex = ind[d2];
306:             float maxval = max[d2];
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 308-339
```cpp
308:             bool mask = (val > maxval) || std::isnan(val);
309:             max[d2] = mask ? val : maxval;
310:             ind[d2] = mask ? index : maxindex;
311:           }
312:         }
313:       }
314:       // Pass III: convert max values from float to bfloat16/Half
315:       int64_t d3 = 0;
316:       for (; d3 < len; d3 += bVec::size()) {
317:         fVec max_fvec0 = fVec::loadu(max + d3);
318:         fVec max_fvec1 = fVec::loadu(max + d3 + fVec::size());
319:         bVec max_bvec = convert_from_float<scalar_t>(max_fvec0, max_fvec1);
320:         max_bvec.store(out + d3);
321:       }
322:       for (; d3 < size; d3++) {
323:         out[d3] = scalar_t(max[d3]);
324:       }
325:       // convert indice data type
326:       vec::convert<int32_t, int64_t>(index_buffer.get(), ind, len);
327:
328:       // move on to next output index
329:       data_index_step(n, nbatch, oh, output_height, ow, output_width);
330:     }
331:   });
332:
333:   if (!output_.is_contiguous(memory_format)) {
334:     output_.copy_(output);
335:   }
336:   if (!indices_.is_contiguous(memory_format)) {
337:     indices_.copy_(indices);
338:   }
339: }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 341-373
```cpp
341: template <typename scalar_t>
342: void cpu_adaptive_max_pool2d_backward(
343:     const Tensor& grad_input_,
344:     const Tensor& grad_output_,
345:     const Tensor& indices_) {
346:   auto grad_output = grad_output_.contiguous();
347:   auto indices = indices_.contiguous();
348:   auto grad_input = grad_input_.contiguous();
349:
350:   auto grad_output_data = grad_output.const_data_ptr<scalar_t>();
351:   auto indices_data = indices.const_data_ptr<int64_t>();
352:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
353:
354:   int64_t ndim = grad_output.ndimension();
355:   // treat batch size and channels as one dimension
356:   int64_t channels = ndim == 3 ? grad_output.size(0) : grad_output.size(0) * grad_output.size(1);
357:   int64_t input_height = grad_input.size(-2);
358:   int64_t input_width = grad_input.size(-1);
359:   int64_t output_height = grad_output.size(-2);
360:   int64_t output_width = grad_output.size(-1);
361:
362:   // parallel on dim of N, C
363:   at::parallel_for(0, channels, 0, [&](int64_t begin, int64_t end) {
364:     for (const auto c : c10::irange(begin, end)) {
365:       scalar_t* grad_input_ptr = grad_input_data + c * input_height * input_width;
366:       const scalar_t* grad_output_ptr = grad_output_data + c * output_height * output_width;
367:       const int64_t* indices_ptr = indices_data + c * output_height * output_width;
368:
369:       for (const auto oh : c10::irange(output_height)) {
370:         for (const auto ow : c10::irange(output_width)) {
371:           // retrieve position of max
372:           int64_t index = oh * output_width + ow;
373:           int64_t maxindex = indices_ptr[index];
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool2d_backward`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool2d_backward`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 375-408
```cpp
375:           // update gradient
376:           grad_input_ptr[maxindex] += grad_output_ptr[index];
377:         }
378:       }
379:     }
380:   });
381:
382:   if (!grad_input_.is_contiguous()) {
383:     grad_input_.copy_(grad_input);
384:   }
385: }
386:
387: template <typename scalar_t>
388: void cpu_adaptive_max_pool2d_backward_channels_last(
389:     const Tensor& grad_input_,
390:     const Tensor& grad_output_,
391:     const Tensor& indices_) {
392:   TORCH_CHECK(grad_output_.ndimension() == 4,
393:               "2d adaptive max pooling backward with channels last format supports tensors with 4 dims.");
394:   auto memory_format = at::MemoryFormat::ChannelsLast;
395:   auto grad_input = grad_input_.contiguous(memory_format);
396:   auto grad_output = grad_output_.contiguous(memory_format);
397:   auto indices = indices_.contiguous(memory_format);
398:
399:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
400:   auto grad_output_data = grad_output.const_data_ptr<scalar_t>();
401:   auto indices_data = indices.const_data_ptr<int64_t>();
402:
403:   int64_t nbatch = grad_input.size(0);
404:   int64_t channels = grad_input.size(1);
405:   int64_t input_height = grad_input.size(2);
406:   int64_t input_width = grad_input.size(3);
407:   int64_t output_height = grad_output.size(2);
408:   int64_t output_width = grad_output.size(3);
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool2d_backward_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool2d_backward_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 410-438
```cpp
410:   // parallel on dim N
411:   at::parallel_for(0, nbatch, 0, [&](int64_t begin, int64_t end) {
412:     for (const auto n : c10::irange(begin, end)) {
413:       scalar_t* grad_input_ptr = grad_input_data + n * input_height * input_width * channels;
414:       const scalar_t* grad_output_ptr = grad_output_data + n * output_height * output_width * channels;
415:       const int64_t* indices_ptr = indices_data + n * output_height * output_width * channels;
416:
417:       for (const auto oh : c10::irange(output_height)) {
418:         for (const auto ow : c10::irange(output_width)) {
419:           const scalar_t* gout = grad_output_ptr + oh * output_width * channels + ow * channels;
420:           const int64_t* ind = indices_ptr + oh * output_width * channels + ow * channels;
421:           // TODO: gcc vectorization
422:           for (const auto c : c10::irange(channels)) {
423:             int64_t maxindex = ind[c];
424:             grad_input_ptr[maxindex * channels + c] += gout[c];
425:           }
426:         }
427:       }
428:     }
429:   });
430:
431:   if (!grad_input_.is_contiguous(memory_format)) {
432:     grad_input_.copy_(grad_input);
433:   }
434: }
435:
436: void adaptive_max_pool2d_kernel_impl(
437:     const Tensor& output,
438:     const Tensor& indices,
```
- EN: The main symbol in this range is `adaptive_max_pool2d_kernel_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `adaptive_max_pool2d_kernel_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 439-481
```cpp
439:     const Tensor& input,
440:     IntArrayRef output_size) {
441:   switch (input.suggest_memory_format()) {
442:     case at::MemoryFormat::Contiguous: {
443:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "adaptive_max_pool2d", [&] {
444:         using param_t = at::opmath_type<scalar_t>;
445:         cpu_adaptive_max_pool2d<scalar_t, /*accscalar_t*/param_t>(output, indices, input, output_size);
446:       });
447:       break;
448:     }
449:     case at::MemoryFormat::ChannelsLast: {
450:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "adaptive_max_pool2d_channels_last", [&]{
451:         cpu_adaptive_max_pool2d_channels_last<scalar_t>(output, indices, input, output_size);
452:       });
453:       break;
454:     }
455:     default:
456:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
457:   }
458: }
459:
460: void adaptive_max_pool2d_backward_kernel_impl(
461:     const Tensor& grad_input,
462:     const Tensor& grad_output,
463:     const Tensor& indices) {
464:   // can't use grad_output memory format to switch here since grad_output might be NC11
465:   switch (grad_input.suggest_memory_format()) {
466:     case at::MemoryFormat::Contiguous: {
467:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, grad_output.scalar_type(), "adaptive_max_pool2d_backward", [&] {
468:         cpu_adaptive_max_pool2d_backward<scalar_t>(grad_input, grad_output, indices);
469:       });
470:       break;
471:     }
472:     case at::MemoryFormat::ChannelsLast: {
473:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, grad_output.scalar_type(), "adaptive_max_pool2d_backward_channels_last", [&]{
474:         cpu_adaptive_max_pool2d_backward_channels_last<scalar_t>(grad_input, grad_output, indices);
475:       });
476:       break;
477:     }
478:     default:
479:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
480:   }
481: }
```
- EN: The main symbol in this range is `adaptive_max_pool2d_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `adaptive_max_pool2d_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 483-512
```cpp
483: template <typename scalar_t, typename accscalar_t>
484: void cpu_adaptive_max_pool3d(
485:     const Tensor& output_,
486:     const Tensor& indices_,
487:     const Tensor& input_,
488:     IntArrayRef output_size) {
489:   auto input = input_.contiguous();
490:   auto output = output_.contiguous();
491:   auto indices = indices_.contiguous();
492:
493:   auto input_data = input.data_ptr<scalar_t>();
494:   auto output_data = output.data_ptr<scalar_t>();
495:   auto indices_data = indices.data_ptr<int64_t>();
496:
497:   int64_t ndim = input.ndimension();
498:   // treat batch size and channels as one dimension
499:   int64_t channels = ndim == 4 ? input.size(0) : input.size(0) * input.size(1);
500:   int64_t input_depth = input.size(-3);
501:   int64_t input_height = input.size(-2);
502:   int64_t input_width = input.size(-1);
503:   int64_t output_depth = output_size[0];
504:   int64_t output_height = output_size[1];
505:   int64_t output_width = output_size[2];
506:
507:   // parallel on dim of N, C
508:   at::parallel_for(0, channels, 0, [&](int64_t begin, int64_t end) {
509:     for (const auto c : c10::irange(begin, end)) {
510:       scalar_t* input_ptr = input_data + c * input_depth * input_height * input_width;
511:       scalar_t* output_ptr = output_data + c * output_depth * output_height * output_width;
512:       int64_t* indices_ptr = indices_data + c * output_depth * output_height * output_width;
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool3d`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool3d`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 514-548
```cpp
514:       for (const auto od : c10::irange(output_depth)) {
515:         int64_t id0 = start_index(od, output_depth, input_depth);
516:         int64_t id1 = end_index(od, output_depth, input_depth);
517:           for (const auto oh : c10::irange(output_height)) {
518:             int64_t ih0 = start_index(oh, output_height, input_height);
519:             int64_t ih1 = end_index(oh, output_height, input_height);
520:
521:           for (const auto ow : c10::irange(output_width)) {
522:             int64_t iw0 = start_index(ow, output_width, input_width);
523:             int64_t iw1 = end_index(ow, output_width, input_width);
524:
525:             // compute local max
526:             int64_t maxindex = id0 * input_height * input_width + ih0 * input_width + iw0;
527:             accscalar_t maxval = -std::numeric_limits<accscalar_t>::infinity();
528:             for (int64_t id = id0; id < id1; id ++) {
529:               for (int64_t ih = ih0; ih < ih1; ih ++) {
530:                 for (int64_t iw = iw0; iw < iw1; iw ++) {
531:                   int64_t index = id * input_height * input_width + ih * input_width + iw;
532:                   scalar_t val = input_ptr[index];
533:                   if ((val > maxval) || std::isnan(val)) {
534:                     maxval = val;
535:                     maxindex = index;
536:                   }
537:                 }
538:               }
539:             }
540:
541:             // set output to local max and store location of max
542:             output_ptr[od * output_height * output_width + oh * output_width + ow] = maxval;
543:             indices_ptr[od * output_height * output_width + oh * output_width + ow] = maxindex;
544:           }
545:         }
546:       }
547:     }
548:   });
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 550-583
```cpp
550:   if (!output_.is_contiguous()) {
551:     output_.copy_(output);
552:   }
553:   if (!indices_.is_contiguous()) {
554:     indices_.copy_(indices);
555:   }
556: }
557:
558: template <typename scalar_t>
559: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
560: cpu_adaptive_max_pool3d_channels_last(
561:     const Tensor& output_,
562:     const Tensor& indices_,
563:     const Tensor& input_,
564:     IntArrayRef output_size) {
565:   TORCH_CHECK(input_.ndimension() == 5,
566:               "3d adaptive max pooling with channels last format supports tensors with 5 dims");
567:   auto memory_format = at::MemoryFormat::ChannelsLast3d;
568:   auto input = input_.contiguous(memory_format);
569:   auto output = output_.contiguous(memory_format);
570:   auto indices = indices_.contiguous(memory_format);
571:
572:   auto input_data = input.data_ptr<scalar_t>();
573:   auto output_data = output.data_ptr<scalar_t>();
574:   auto indices_data = indices.data_ptr<int64_t>();
575:
576:   int64_t nbatch = input.size(0);
577:   int64_t channels = input.size(1);
578:   int64_t input_depth = input.size(2);
579:   int64_t input_height = input.size(3);
580:   int64_t input_width = input.size(4);
581:   int64_t output_depth = output_size[0];
582:   int64_t output_height = output_size[1];
583:   int64_t output_width = output_size[2];
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool3d_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool3d_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 585-614
```cpp
585:   using Vec = vec::Vectorized<scalar_t>;
586:   using integer_t = vec::int_same_size_t<scalar_t>;
587:   using iVec = vec::Vectorized<integer_t>;
588:   // for the convenience of vectorization, use integer of the same size of scalar_t,
589:   //   e.g. int32_t for float, int64_t for double
590:   // need to make sure doesn't overflow
591:   TORCH_CHECK(input_height * input_width <= std::numeric_limits<integer_t>::max());
592:
593:   // parallel on dim of N, H, W
594:   at::parallel_for(0, nbatch * output_depth * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
595:     int64_t n = 0;
596:     int64_t od = 0;
597:     int64_t oh = 0;
598:     int64_t ow = 0;
599:     data_index_init(begin, n, nbatch, od, output_depth, oh, output_height, ow, output_width);
600:
601:     int64_t size = channels;
602:     int64_t len = size - (size % Vec::size());
603:     // temp buffer holding index with integer_t
604:     auto index_buffer = std::make_unique<integer_t []>(len);
605:
606:     for (const auto i : c10::irange(begin, end)) {
607:       int64_t id0 = start_index(od, output_depth, input_depth);
608:       int64_t id1 = end_index(od, output_depth, input_depth);
609:
610:       int64_t ih0 = start_index(oh, output_height, input_height);
611:       int64_t ih1 = end_index(oh, output_height, input_height);
612:
613:       int64_t iw0 = start_index(ow, output_width, input_width);
614:       int64_t iw1 = end_index(ow, output_width, input_width);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 616-649
```cpp
616:       scalar_t* out = output_data + i * channels;
617:       int64_t* ind = indices_data + i * channels;
618:
619:       // Pass I: init out lane
620:       iVec index0_vec = iVec(id0 * input_height * input_width + ih0 * input_width + iw0);
621:       Vec out_vec = Vec(-std::numeric_limits<scalar_t>::infinity());
622:       int64_t d1 = 0;
623:       for (; d1 < len; d1 += Vec::size()) {
624:         index0_vec.store(index_buffer.get() + d1);
625:         out_vec.store(out + d1);
626:       }
627:       for (; d1 < size; d1++) {
628:         ind[d1] = id0 * input_height * input_width + ih0 * input_width + iw0;
629:         out[d1] = -std::numeric_limits<scalar_t>::infinity();
630:       }
631:       // Pass II: compute local max
632:       for (int64_t id = id0; id < id1; id ++) {
633:         for (int64_t ih = ih0; ih < ih1; ih ++) {
634:           for (int64_t iw = iw0; iw < iw1; iw ++) {
635:             scalar_t* in = input_data + n * input_depth * input_height * input_width * channels +
636:                 id * input_height * input_width * channels + ih * input_width * channels + iw * channels;
637:
638:             int64_t d2 = 0;
639:             for (; d2 < len; d2 += Vec::size()) {
640:               iVec index_vec = iVec(id * input_height * input_width + ih * input_width + iw);
641:               Vec val_vec = Vec::loadu(in + d2);
642:               iVec maxindex_vec = iVec::loadu(index_buffer.get() + d2);
643:               Vec maxval_vec = Vec::loadu(out + d2);
644:
645:               // true = all ones, false = all zeros
646:               Vec mask = (val_vec > maxval_vec) | val_vec.isnan();
647:               iVec imask = vec::cast<integer_t>(mask);
648:               Vec out_vec = Vec::blendv(maxval_vec, val_vec, mask);
649:               iVec ind_vec = iVec::blendv(maxindex_vec, index_vec, imask);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 651-681
```cpp
651:               out_vec.store(out + d2);
652:               ind_vec.store(index_buffer.get() + d2);
653:             }
654:             for (; d2 < size; d2++) {
655:               int64_t index = id * input_height * input_width + ih * input_width + iw;
656:               scalar_t val = in[d2];
657:               int64_t maxindex = ind[d2];
658:               scalar_t maxval = out[d2];
659:
660:               bool mask = (val > maxval) || std::isnan(val);
661:               out[d2] = mask ? val : maxval;
662:               ind[d2] = mask ? index : maxindex;
663:             }
664:           }
665:         }
666:       }
667:       // convert indice data type
668:       vec::convert<integer_t, int64_t>(index_buffer.get(), ind, len);
669:
670:       // move on to next output index
671:       data_index_step(n, nbatch, od, output_depth, oh, output_height, ow, output_width);
672:     }
673:   });
674:
675:   if (!output_.is_contiguous(memory_format)) {
676:     output_.copy_(output);
677:   }
678:   if (!indices_.is_contiguous(memory_format)) {
679:     indices_.copy_(indices);
680:   }
681: }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 683-714
```cpp
683: template <typename scalar_t>
684: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
685: cpu_adaptive_max_pool3d_channels_last(
686:     const Tensor& output_,
687:     const Tensor& indices_,
688:     const Tensor& input_,
689:     IntArrayRef output_size) {
690:   TORCH_CHECK(input_.ndimension() == 5,
691:               "3d adaptive max pooling with channels last format supports tensors with 5 dims");
692:   auto memory_format = at::MemoryFormat::ChannelsLast3d;
693:   auto input = input_.contiguous(memory_format);
694:   auto output = output_.contiguous(memory_format);
695:   auto indices = indices_.contiguous(memory_format);
696:
697:   auto input_data = input.data_ptr<BFloat16>();
698:   auto output_data = output.data_ptr<BFloat16>();
699:   auto indices_data = indices.data_ptr<int64_t>();
700:
701:   int64_t nbatch = input.size(0);
702:   int64_t channels = input.size(1);
703:   int64_t input_depth = input.size(2);
704:   int64_t input_height = input.size(3);
705:   int64_t input_width = input.size(4);
706:   int64_t output_depth = output_size[0];
707:   int64_t output_height = output_size[1];
708:   int64_t output_width = output_size[2];
709:
710:   using bVec = vec::Vectorized<BFloat16>;
711:   using fVec = vec::Vectorized<float>;
712:   using iVec = vec::Vectorized<int32_t>;
713:   // need to make sure doesn't overflow
714:   TORCH_CHECK(input_height * input_width <= std::numeric_limits<int32_t>::max());
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool3d_channels_last`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool3d_channels_last`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 716-743
```cpp
716:   // parallel on dim of N, H, W
717:   at::parallel_for(0, nbatch * output_depth * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
718:     int64_t n = 0;
719:     int64_t od = 0;
720:     int64_t oh = 0;
721:     int64_t ow = 0;
722:     data_index_init(begin, n, nbatch, od, output_depth, oh, output_height, ow, output_width);
723:
724:     int64_t size = channels;
725:     int64_t len = size - (size % bVec::size());
726:     // temp buffer holding index with integer_t
727:     auto index_buffer = std::make_unique<int32_t []>(len);
728:     // temp buffer holding max value with float
729:     auto max_arr = std::make_unique<float []>(size);
730:     float* max = max_arr.get();
731:
732:     for (const auto i : c10::irange(begin, end)) {
733:       int64_t id0 = start_index(od, output_depth, input_depth);
734:       int64_t id1 = end_index(od, output_depth, input_depth);
735:
736:       int64_t ih0 = start_index(oh, output_height, input_height);
737:       int64_t ih1 = end_index(oh, output_height, input_height);
738:
739:       int64_t iw0 = start_index(ow, output_width, input_width);
740:       int64_t iw1 = end_index(ow, output_width, input_width);
741:
742:       BFloat16* out = output_data + i * channels;
743:       int64_t* ind = indices_data + i * channels;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 745-773
```cpp
745:       // Pass I: init out lane
746:       iVec index0_ivec = iVec(id0 * input_height * input_width + ih0 * input_width + iw0);
747:       fVec max_fvec = fVec(-std::numeric_limits<float>::infinity());
748:       int64_t d1 = 0;
749:       for (; d1 < len; d1 += fVec::size()) {
750:         index0_ivec.store(index_buffer.get() + d1);
751:         max_fvec.store(max + d1);
752:       }
753:       for (; d1 < size; d1++) {
754:         ind[d1] = id0 * input_height * input_width + ih0 * input_width + iw0;
755:         max[d1] = -std::numeric_limits<float>::infinity();
756:       }
757:       // Pass II: compute local max
758:       for (int64_t id = id0; id < id1; id ++) {
759:         for (int64_t ih = ih0; ih < ih1; ih ++) {
760:           for (int64_t iw = iw0; iw < iw1; iw ++) {
761:             BFloat16* in = input_data + n * input_depth * input_height * input_width * channels +
762:                 id * input_height * input_width * channels + ih * input_width * channels + iw * channels;
763:
764:             int64_t d2 = 0;
765:             for (; d2 < len; d2 += bVec::size()) {
766:               iVec index_ivec = iVec(id * input_height * input_width + ih * input_width + iw);
767:               bVec val_bvec = bVec::loadu(in + d2);
768:               auto [val_fvec0, val_fvec1] = convert_bfloat16_float(val_bvec);
769:
770:               iVec maxindex_ivec0 = iVec::loadu(index_buffer.get() + d2);
771:               iVec maxindex_ivec1 = iVec::loadu(index_buffer.get() + d2 + iVec::size());
772:               fVec maxval_fvec0 = fVec::loadu(max + d2);
773:               fVec maxval_fvec1 = fVec::loadu(max + d2 + fVec::size());
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 775-816
```cpp
775:               // true = all ones, false = all zeros
776:               fVec mask0 = (val_fvec0 > maxval_fvec0) | val_fvec0.isnan();
777:               fVec mask1 = (val_fvec1 > maxval_fvec1) | val_fvec1.isnan();
778:               iVec imask0 = vec::cast<int32_t>(mask0);
779:               iVec imask1 = vec::cast<int32_t>(mask1);
780:
781:               fVec max_fvec0 = fVec::blendv(maxval_fvec0, val_fvec0, mask0);
782:               fVec max_fvec1 = fVec::blendv(maxval_fvec1, val_fvec1, mask1);
783:               iVec ind_ivec0 = iVec::blendv(maxindex_ivec0, index_ivec, imask0);
784:               iVec ind_ivec1 = iVec::blendv(maxindex_ivec1, index_ivec, imask1);
785:
786:               max_fvec0.store(max + d2);
787:               max_fvec1.store(max + d2 + fVec::size());
788:               ind_ivec0.store(index_buffer.get() + d2);
789:               ind_ivec1.store(index_buffer.get() + d2 + iVec::size());
790:             }
791:             for (; d2 < size; d2++) {
792:               int64_t index = id * input_height * input_width + ih * input_width + iw;
793:               float val = float(in[d2]);
794:               int64_t maxindex = ind[d2];
795:               float maxval = max[d2];
796:
797:               bool mask = (val > maxval) || std::isnan(val);
798:               max[d2] = mask ? val : maxval;
799:               ind[d2] = mask ? index : maxindex;
800:             }
801:           }
802:         }
803:       }
804:       // Pass III: convert max values from float to bfloat16
805:       int64_t d3 = 0;
806:       for (; d3 < len; d3 += bVec::size()) {
807:         fVec max_fvec0 = fVec::loadu(max + d3);
808:         fVec max_fvec1 = fVec::loadu(max + d3 + fVec::size());
809:         bVec max_bvec = convert_float_bfloat16(max_fvec0, max_fvec1);
810:         max_bvec.store(out + d3);
811:       }
812:       for (; d3 < size; d3++) {
813:         out[d3] = BFloat16(max[d3]);
814:       }
815:       // convert indice data type
816:       vec::convert<int32_t, int64_t>(index_buffer.get(), ind, len);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 818-852
```cpp
818:       // move on to next output index
819:       data_index_step(n, nbatch, od, output_depth, oh, output_height, ow, output_width);
820:     }
821:   });
822:
823:   if (!output_.is_contiguous(memory_format)) {
824:     output_.copy_(output);
825:   }
826:   if (!indices_.is_contiguous(memory_format)) {
827:     indices_.copy_(indices);
828:   }
829: }
830:
831: template <typename scalar_t>
832: void cpu_adaptive_max_pool3d_backward(
833:     const Tensor& grad_input_,
834:     const Tensor& grad_output_,
835:     const Tensor& indices_) {
836:   auto grad_output = grad_output_.contiguous();
837:   auto indices = indices_.contiguous();
838:   auto grad_input = grad_input_.contiguous();
839:
840:   auto grad_output_data = grad_output.data_ptr<scalar_t>();
841:   auto indices_data = indices.data_ptr<int64_t>();
842:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
843:
844:   int64_t ndim = grad_output.ndimension();
845:   // treat batch size and channels as one dimension
846:   int64_t channels = ndim == 3 ? grad_output.size(0) : grad_output.size(0) * grad_output.size(1);
847:   int64_t input_depth = grad_input.size(-3);
848:   int64_t input_height = grad_input.size(-2);
849:   int64_t input_width = grad_input.size(-1);
850:   int64_t output_depth = grad_output.size(-3);
851:   int64_t output_height = grad_output.size(-2);
852:   int64_t output_width = grad_output.size(-1);
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool3d_backward`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool3d_backward`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 854-891
```cpp
854:   // parallel on dim of N, C
855:   at::parallel_for(0, channels, 0, [&](int64_t begin, int64_t end) {
856:     for (const auto c : c10::irange(begin, end)) {
857:       scalar_t* grad_input_ptr = grad_input_data + c * input_depth * input_height * input_width;
858:       scalar_t* grad_output_ptr = grad_output_data + c * output_depth * output_height * output_width;
859:       int64_t* indices_ptr = indices_data + c * output_depth * output_height * output_width;
860:
861:       for (const auto od : c10::irange(output_depth)) {
862:         for (const auto oh : c10::irange(output_height)) {
863:           for (const auto ow : c10::irange(output_width)) {
864:             // retrieve position of max
865:             int64_t index = od * output_height * output_width + oh * output_width + ow;
866:             int64_t maxindex = indices_ptr[index];
867:
868:             // update gradient
869:             grad_input_ptr[maxindex] += grad_output_ptr[index];
870:           }
871:         }
872:       }
873:     }
874:   });
875:
876:   if (!grad_input_.is_contiguous()) {
877:     grad_input_.copy_(grad_input);
878:   }
879: }
880:
881: template <typename scalar_t>
882: void cpu_adaptive_max_pool3d_backward_channels_last(
883:     const Tensor& grad_input_,
884:     const Tensor& grad_output_,
885:     const Tensor& indices_) {
886:   TORCH_CHECK(grad_output_.ndimension() == 5,
887:               "3d adaptive max pooling backward with channels last format supports tensors with 5 dims.");
888:   auto memory_format = at::MemoryFormat::ChannelsLast3d;
889:   auto grad_input = grad_input_.contiguous(memory_format);
890:   auto grad_output = grad_output_.contiguous(memory_format);
891:   auto indices = indices_.contiguous(memory_format);
```
- EN: The main symbol in this range is `cpu_adaptive_max_pool3d_backward_channels_last`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `cpu_adaptive_max_pool3d_backward_channels_last`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 893-927
```cpp
893:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
894:   auto grad_output_data = grad_output.data_ptr<scalar_t>();
895:   auto indices_data = indices.data_ptr<int64_t>();
896:
897:   int64_t nbatch = grad_input.size(0);
898:   int64_t channels = grad_input.size(1);
899:   int64_t input_depth = grad_input.size(2);
900:   int64_t input_height = grad_input.size(3);
901:   int64_t input_width = grad_input.size(4);
902:   int64_t output_depth = grad_output.size(2);
903:   int64_t output_height = grad_output.size(3);
904:   int64_t output_width = grad_output.size(4);
905:
906:   // parallel on dim N
907:   at::parallel_for(0, nbatch, 0, [&](int64_t begin, int64_t end) {
908:     for (const auto n : c10::irange(begin, end)) {
909:       scalar_t* grad_input_ptr = grad_input_data + n * input_depth * input_height * input_width * channels;
910:       scalar_t* grad_output_ptr = grad_output_data + n * output_depth * output_height * output_width * channels;
911:       int64_t* indices_ptr = indices_data + n * output_depth * output_height * output_width * channels;
912:
913:       for (const auto od : c10::irange(output_depth)) {
914:         for (const auto oh : c10::irange(output_height)) {
915:           for (const auto ow : c10::irange(output_width)) {
916:             scalar_t* gout = grad_output_ptr + od * output_height * output_width * channels + oh * output_width * channels + ow * channels;
917:             int64_t* ind = indices_ptr + od * output_height * output_width * channels + oh * output_width * channels + ow * channels;
918:             // TODO: gcc vectorization
919:             for (const auto c : c10::irange(channels)) {
920:               int64_t maxindex = ind[c];
921:               grad_input_ptr[maxindex * channels + c] += gout[c];
922:             }
923:           }
924:         }
925:       }
926:     }
927:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 929-956
```cpp
929:   if (!grad_input_.is_contiguous(memory_format)) {
930:     grad_input_.copy_(grad_input);
931:   }
932: }
933:
934: void adaptive_max_pool3d_kernel_impl(
935:     const Tensor& output,
936:     const Tensor& indices,
937:     const Tensor& input,
938:     IntArrayRef output_size) {
939:   switch (input.suggest_memory_format()) {
940:     case at::MemoryFormat::Contiguous: {
941:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "adaptive_max_pool3d", [&] {
942:         using param_t = at::opmath_type<scalar_t>;
943:         cpu_adaptive_max_pool3d<scalar_t, /*accscalar_t*/param_t>(output, indices, input, output_size);
944:       });
945:       break;
946:     }
947:     case at::MemoryFormat::ChannelsLast3d: {
948:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "adaptive_max_pool3d_channels_last", [&]{
949:         cpu_adaptive_max_pool3d_channels_last<scalar_t>(output, indices, input, output_size);
950:       });
951:       break;
952:     }
953:     default:
954:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
955:   }
956: }
```
- EN: The main symbol in this range is `adaptive_max_pool3d_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `adaptive_max_pool3d_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 958-986
```cpp
958: void adaptive_max_pool3d_backward_kernel_impl(
959:     const Tensor& grad_input,
960:     const Tensor& grad_output,
961:     const Tensor& indices) {
962:   // can't use grad_output memory format to switch here since grad_output might be NC11
963:   switch (grad_input.suggest_memory_format()) {
964:     case at::MemoryFormat::Contiguous: {
965:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, grad_output.scalar_type(), "adaptive_max_pool3d_backward", [&] {
966:         cpu_adaptive_max_pool3d_backward<scalar_t>(grad_input, grad_output, indices);
967:       });
968:       break;
969:     }
970:     case at::MemoryFormat::ChannelsLast3d: {
971:       AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, grad_output.scalar_type(), "adaptive_max_pool3d_backward_channels_last", [&]{
972:         cpu_adaptive_max_pool3d_backward_channels_last<scalar_t>(grad_input, grad_output, indices);
973:       });
974:       break;
975:     }
976:     default:
977:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
978:   }
979: }
980:
981: } // anonymous namespace
982:
983: REGISTER_DISPATCH(adaptive_max_pool2d_kernel, &adaptive_max_pool2d_kernel_impl)
984: REGISTER_DISPATCH(adaptive_max_pool2d_backward_kernel, &adaptive_max_pool2d_backward_kernel_impl)
985: REGISTER_DISPATCH(adaptive_max_pool3d_kernel, &adaptive_max_pool3d_kernel_impl)
986: REGISTER_DISPATCH(adaptive_max_pool3d_backward_kernel, &adaptive_max_pool3d_backward_kernel_impl)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `adaptive_max_pool3d_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `adaptive_max_pool3d_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 988-988
```cpp
988: } // at::native
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/native/AdaptivePooling.h`, `ATen/Parallel.h`, `ATen/cpu/vec/vec.h`, `ATen/cpu/vec/functional.h`, `ATen/native/cpu/utils.h`, `ATen/OpMathType.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
