# AvgPoolKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/AvgPoolKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU pooling kernels and pooled-output shape or reduction logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 池化 kernel，以及池化输出形状或归约逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/Parallel.h>
 5: #include <ATen/OpMathType.h>
 6: #include <ATen/cpu/vec/vec.h>
 7: #include <ATen/cpu/vec/functional.h>
 8: #include <ATen/native/Pool.h>
 9: #include <ATen/native/cpu/utils.h>
10: #include <c10/util/irange.h>
11:
12: namespace at::native {
13:
14: namespace {
15:
16: template <typename scalar_t>
17: void cpu_avg_pool2d(
18:     const Tensor& output_,
19:     const Tensor& input_,
20:     int64_t kW, int64_t kH,
21:     int64_t dW, int64_t dH,
22:     int64_t padW, int64_t padH,
23:     bool count_include_pad,
24:     std::optional<int64_t> divisor_override) {
25:   using acc_t = at::opmath_type<scalar_t>;
26:
27:   auto input = input_.contiguous();
28:   auto output = output_.contiguous();
29:
30:   auto input_data = input.const_data_ptr<scalar_t>();
31:   auto output_data = output.data_ptr<scalar_t>();
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 33-64
```cpp
33:   int64_t numel = output.numel();
34:   int64_t ndim = input.ndimension();
35:   // treat batch size and channels as one dimension
36:   int64_t channels = ndim == 3 ? input.size(0) : input.size(0) * input.size(1);
37:   int64_t input_height = input.size(-2);
38:   int64_t input_width = input.size(-1);
39:   int64_t output_height = output.size(-2);
40:   int64_t output_width = output.size(-1);
41:
42:   // parallel on dim N, C, H, W
43:   at::parallel_for(0, numel, 0, [&](int64_t begin, int64_t end) {
44:     int64_t c = 0;
45:     int64_t oh = 0;
46:     int64_t ow = 0;
47:     data_index_init(begin, c, channels, oh, output_height, ow, output_width);
48:
49:     for (const auto i : c10::irange(begin, end)) {
50:       output_data[i] = static_cast<scalar_t>(0);
51:
52:       // local pointers
53:       const scalar_t* input_ptr = input_data + c * input_height * input_width;
54:
55:       // compute the mean of the input image...
56:       int64_t ih0 = oh * dH - padH;
57:       int64_t iw0 = ow * dW - padW;
58:       int64_t ih1 = std::min(ih0 + kH, input_height + padH);
59:       int64_t iw1 = std::min(iw0 + kW, input_width + padW);
60:       int64_t pool_size = (ih1 - ih0) * (iw1 - iw0);
61:       ih0 = std::max(ih0, (int64_t) 0);
62:       iw0 = std::max(iw0, (int64_t) 0);
63:       ih1 = std::min(ih1, input_height);
64:       iw1 = std::min(iw1, input_width);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 66-95
```cpp
66:       if (ih0 >= ih1 || iw0 >= iw1) {
67:         // move on to next output index
68:         data_index_step(c, channels, oh, output_height, ow, output_width);
69:         continue;
70:       }
71:
72:       acc_t sum = 0;
73:
74:       int64_t divide_factor = 0;
75:       if (divisor_override.has_value()) {
76:         divide_factor = divisor_override.value();
77:       } else {
78:         if(count_include_pad) {
79:           divide_factor = pool_size;
80:         } else {
81:           divide_factor = (ih1 - ih0) * (iw1 - iw0);
82:         }
83:       }
84:
85:       for (const auto ih : c10::irange(ih0, ih1)) {
86:         for (const auto iw : c10::irange(iw0, iw1)) {
87:           sum += input_ptr[ih * input_width + iw];
88:         }
89:       }
90:       output_data[i] += scalar_t(sum / divide_factor);
91:
92:       // move on to next output index
93:       data_index_step(c, channels, oh, output_height, ow, output_width);
94:     }
95:   });
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 97-126
```cpp
 97:   if (!output_.is_contiguous()) {
 98:     output_.copy_(output);
 99:   }
100: }
101:
102: template <typename scalar_t,
103:           std::enable_if_t<!is_reduced_floating_point<scalar_t>::value, int> = 0>
104: void cpu_avg_pool2d_channels_last(
105:     const Tensor& output_,
106:     const Tensor& input_,
107:     int64_t kW, int64_t kH,
108:     int64_t dW, int64_t dH,
109:     int64_t padW, int64_t padH,
110:     bool count_include_pad,
111:     std::optional<int64_t> divisor_override) {
112:   TORCH_CHECK(input_.ndimension() == 4,
113:               "2d average pooling with channels last format supports tensors with 4 dims");
114:   auto memory_format = at::MemoryFormat::ChannelsLast;
115:   auto input = input_.contiguous(memory_format);
116:   auto output = output_.contiguous(memory_format);
117:
118:   auto input_data = input.const_data_ptr<scalar_t>();
119:   auto output_data = output.data_ptr<scalar_t>();
120:
121:   int64_t nbatch = input.size(0);
122:   int64_t channels = input.size(1);
123:   int64_t input_height = input.size(2);
124:   int64_t input_width = input.size(3);
125:   int64_t output_height = output.size(2);
126:   int64_t output_width = output.size(3);
```
- EN: The main symbol in this range is `cpu_avg_pool2d_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_avg_pool2d_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 128-159
```cpp
128:   using Vec = vec::Vectorized<scalar_t>;
129:   // parallel on dim N, H, W
130:   at::parallel_for(0, nbatch * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
131:     int64_t n = 0;
132:     int64_t oh = 0;
133:     int64_t ow = 0;
134:     data_index_init(begin, n, nbatch, oh, output_height, ow, output_width);
135:
136:     int64_t size = channels;
137:     int64_t len = size - (size % Vec::size());
138:     for (const auto i : c10::irange(begin, end)) {
139:       // compute the mean of the input image...
140:       int64_t ih0 = oh * dH - padH;
141:       int64_t iw0 = ow * dW - padW;
142:       int64_t ih1 = std::min(ih0 + kH, input_height + padH);
143:       int64_t iw1 = std::min(iw0 + kW, input_width + padW);
144:       int64_t pool_size = (ih1 - ih0) * (iw1 - iw0);
145:       ih0 = std::max(ih0, (int64_t) 0);
146:       iw0 = std::max(iw0, (int64_t) 0);
147:       ih1 = std::min(ih1, input_height);
148:       iw1 = std::min(iw1, input_width);
149:
150:       int64_t divide_factor = 0;
151:       if (divisor_override.has_value()) {
152:         divide_factor = divisor_override.value();
153:       } else {
154:         if(count_include_pad) {
155:           divide_factor = pool_size;
156:         } else {
157:           divide_factor = (ih1 - ih0) * (iw1 - iw0);
158:         }
159:       }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 161-194
```cpp
161:       scalar_t* out = output_data + i * channels;
162:
163:       // Pass I: zero the out lane
164:       int64_t d1 = 0;
165:       for (; d1 < len; d1 += Vec::size()) {
166:         Vec out_vec = Vec(scalar_t(0));
167:         out_vec.store(out + d1);
168:       }
169:       for (; d1 < size; d1++) {
170:         out[d1] = scalar_t(0);
171:       }
172:
173:       if (ih0 >= ih1 || iw0 >= iw1) {
174:         // move on to next output index
175:         data_index_step(n, nbatch, oh, output_height, ow, output_width);
176:         continue;
177:       }
178:
179:       // Pass II: compute local sum
180:       for (const auto ih : c10::irange(ih0, ih1)) {
181:         for (const auto iw : c10::irange(iw0, iw1)) {
182:           const scalar_t* in = input_data + n * input_height * input_width * channels +
183:               ih * input_width * channels + iw * channels;
184:
185:           int64_t d2 = 0;
186:           for (; d2 < len; d2 += Vec::size()) {
187:             Vec out_vec = Vec::loadu(out + d2) + Vec::loadu(in + d2);
188:             out_vec.store(out + d2);
189:           }
190:           for (; d2 < size; d2++) {
191:             out[d2] += in[d2];
192:           }
193:         }
194:       }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 196-230
```cpp
196:       // Pass III: compute local average
197:       int64_t d3 = 0;
198:       for (; d3 < len; d3 += Vec::size()) {
199:         Vec out_vec = Vec::loadu(out + d3) / Vec(scalar_t(divide_factor));
200:         out_vec.store(out + d3);
201:       }
202:       for (; d3 < size; d3++) {
203:         out[d3] = out[d3] / divide_factor;
204:       }
205:
206:       // move on to next output index
207:       data_index_step(n, nbatch, oh, output_height, ow, output_width);
208:     }
209:   });
210:
211:   if (!output_.is_contiguous(memory_format)) {
212:     output_.copy_(output);
213:   }
214: }
215:
216: template <typename scalar_t,
217:           std::enable_if_t<is_reduced_floating_point<scalar_t>::value, int> = 0>
218: void cpu_avg_pool2d_channels_last(
219:     const Tensor& output_,
220:     const Tensor& input_,
221:     int64_t kW, int64_t kH,
222:     int64_t dW, int64_t dH,
223:     int64_t padW, int64_t padH,
224:     bool count_include_pad,
225:     std::optional<int64_t> divisor_override) {
226:   TORCH_CHECK(input_.ndimension() == 4,
227:               "2d average pooling with channels last format supports tensors with 4 dims");
228:   auto memory_format = at::MemoryFormat::ChannelsLast;
229:   auto input = input_.contiguous(memory_format);
230:   auto output = output_.contiguous(memory_format);
```
- EN: The main symbol in this range is `cpu_avg_pool2d_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_avg_pool2d_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 232-267
```cpp
232:   auto input_data = input.const_data_ptr<scalar_t>();
233:   auto output_data = output.data_ptr<scalar_t>();
234:
235:   int64_t nbatch = input.size(0);
236:   int64_t channels = input.size(1);
237:   int64_t input_height = input.size(2);
238:   int64_t input_width = input.size(3);
239:   int64_t output_height = output.size(2);
240:   int64_t output_width = output.size(3);
241:
242:   using bVec = vec::Vectorized<scalar_t>;
243:   using fVec = vec::Vectorized<float>;
244:   // parallel on dim N, H, W
245:   at::parallel_for(0, nbatch * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
246:     int64_t n = 0;
247:     int64_t oh = 0;
248:     int64_t ow = 0;
249:     data_index_init(begin, n, nbatch, oh, output_height, ow, output_width);
250:
251:     // temp buffer for sum, use float as accumulation type
252:     // can't reuse output buffer to store sum since it is BFloat16/Half
253:     auto sum_arr = std::make_unique<float []>(channels);
254:     float* sum = sum_arr.get();
255:
256:     int64_t size = channels;
257:     for (const auto i : c10::irange(begin, end)) {
258:       // compute the mean of the input image...
259:       int64_t ih0 = oh * dH - padH;
260:       int64_t iw0 = ow * dW - padW;
261:       int64_t ih1 = std::min(ih0 + kH, input_height + padH);
262:       int64_t iw1 = std::min(iw0 + kW, input_width + padW);
263:       int64_t pool_size = (ih1 - ih0) * (iw1 - iw0);
264:       ih0 = std::max(ih0, (int64_t) 0);
265:       iw0 = std::max(iw0, (int64_t) 0);
266:       ih1 = std::min(ih1, input_height);
267:       iw1 = std::min(iw1, input_width);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 269-301
```cpp
269:       int64_t divide_factor = 0;
270:       if (divisor_override.has_value()) {
271:         divide_factor = divisor_override.value();
272:       } else {
273:         if(count_include_pad) {
274:           divide_factor = pool_size;
275:         } else {
276:           divide_factor = (ih1 - ih0) * (iw1 - iw0);
277:         }
278:       }
279:
280:       scalar_t* out = output_data + i * channels;
281:
282:       // Pass I: zero the out lane
283:       int64_t d1 = 0;
284:       for (; d1 < size - (size % fVec::size()); d1 += fVec::size()) {
285:         fVec sum_fvec = fVec(float(0));
286:         sum_fvec.store(sum + d1);
287:       }
288:       for (; d1 < size; d1++) {
289:         sum[d1] = float(0);
290:       }
291:
292:       if (ih0 >= ih1 || iw0 >= iw1) {
293:         // since we are not directly using output as the accumulation buffer,
294:         // in case the kernel window is out of range, need to zero the output buffer here.
295:         for (int64_t k = 0; k < size; k++) {
296:           out[k] = 0;
297:         }
298:         // move on to next output index
299:         data_index_step(n, nbatch, oh, output_height, ow, output_width);
300:         continue;
301:       }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 303-336
```cpp
303:       // Pass II: compute local sum
304:       for (const auto ih : c10::irange(ih0, ih1)) {
305:         for (const auto iw : c10::irange(iw0, iw1)) {
306:           const scalar_t* in = input_data + n * input_height * input_width * channels +
307:               ih * input_width * channels + iw * channels;
308:
309:           int64_t d2 = 0;
310:           for (; d2 < size - (size % bVec::size()); d2 += bVec::size()) {
311:             bVec data_bvec = bVec::loadu(in + d2);
312:             auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
313:
314:             fVec sum_fvec0 = fVec::loadu(sum + d2) + data_fvec0;
315:             fVec sum_fvec1 = fVec::loadu(sum + d2 + fVec::size()) + data_fvec1;
316:             sum_fvec0.store(sum + d2);
317:             sum_fvec1.store(sum + d2 + fVec::size());
318:           }
319:           for (; d2 < size; d2++) {
320:             sum[d2] += float(in[d2]);
321:           }
322:         }
323:       }
324:
325:       // Pass III: compute local average
326:       int64_t d3 = 0;
327:       for (; d3 < size - (size % bVec::size()); d3 += bVec::size()) {
328:         fVec out_fvec0 = fVec::loadu(sum + d3) / fVec(float(divide_factor));
329:         fVec out_fvec1 = fVec::loadu(sum + d3 + fVec::size()) / fVec(float(divide_factor));
330:
331:         bVec out_bvec = convert_from_float<scalar_t>(out_fvec0, out_fvec1);
332:         out_bvec.store(out + d3);
333:       }
334:       for (; d3 < size; d3++) {
335:         out[d3] = scalar_t(sum[d3] / divide_factor);
336:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 338-369
```cpp
338:       // move on to next output index
339:       data_index_step(n, nbatch, oh, output_height, ow, output_width);
340:     }
341:   });
342:
343:   if (!output_.is_contiguous(memory_format)) {
344:     output_.copy_(output);
345:   }
346: }
347:
348: template <typename scalar_t>
349: void cpu_avg_pool2d_backward(
350:     const Tensor& grad_input_,
351:     const Tensor& grad_output_,
352:     int kW, int kH,
353:     int dW, int dH,
354:     int padW, int padH,
355:     bool count_include_pad,
356:     std::optional<int64_t> divisor_override) {
357:   auto grad_output = grad_output_.contiguous();
358:   auto grad_input = grad_input_.contiguous();
359:
360:   auto grad_output_data = grad_output.const_data_ptr<scalar_t>();
361:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
362:
363:   int64_t ndim = grad_output.ndimension();
364:   // treat batch size and channels as one dimension
365:   int64_t channels = ndim == 3 ? grad_output.size(0) : grad_output.size(0) * grad_output.size(1);
366:   int64_t input_height = grad_input.size(-2);
367:   int64_t input_width = grad_input.size(-1);
368:   int64_t output_height = grad_output.size(-2);
369:   int64_t output_width = grad_output.size(-1);
```
- EN: The main symbol in this range is `cpu_avg_pool2d_backward`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_avg_pool2d_backward`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 371-409
```cpp
371:   // parallel on dim of N, C
372:   at::parallel_for(0, channels, 0, [&](int64_t begin, int64_t end) {
373:     for (const auto c : c10::irange(begin, end)) {
374:       scalar_t* grad_input_ptr = grad_input_data + c * input_height * input_width;
375:       const scalar_t* grad_output_ptr = grad_output_data + c * output_height * output_width;
376:
377:       for (const auto oh : c10::irange(output_height)) {
378:         for (const auto ow : c10::irange(output_width)) {
379:           int64_t ih0 = oh * dH - padH;
380:           int64_t iw0 = ow * dW - padW;
381:           int64_t ih1 = std::min(ih0 + kH, input_height + padH);
382:           int64_t iw1 = std::min(iw0 + kW, input_width + padW);
383:           int64_t pool_size = (ih1 - ih0) * (iw1 - iw0);
384:           ih0 = std::max(ih0, (int64_t) 0);
385:           iw0 = std::max(iw0, (int64_t) 0);
386:           ih1 = std::min(ih1, input_height);
387:           iw1 = std::min(iw1, input_width);
388:
389:           int64_t divide_factor = 0;
390:           if (divisor_override.has_value()) {
391:             divide_factor = divisor_override.value();
392:           } else {
393:             if(count_include_pad) {
394:               divide_factor = pool_size;
395:             } else {
396:               divide_factor = (ih1 - ih0) * (iw1 - iw0);
397:             }
398:           }
399:
400:           scalar_t grad_delta = grad_output_ptr[oh * output_width + ow] / divide_factor;
401:           for (const auto ih : c10::irange(ih0, ih1)) {
402:             for (const auto iw : c10::irange(iw0, iw1)) {
403:               grad_input_ptr[ih * input_width + iw] += grad_delta;
404:             }
405:           }
406:         }
407:       }
408:     }
409:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 411-444
```cpp
411:   if (!grad_input_.is_contiguous()) {
412:     grad_input_.copy_(grad_input);
413:   }
414: }
415:
416: template <typename scalar_t>
417: void cpu_avg_pool2d_backward_channels_last(
418:     const Tensor& grad_input_,
419:     const Tensor& grad_output_,
420:     int kW, int kH,
421:     int dW, int dH,
422:     int padW, int padH,
423:     bool count_include_pad,
424:     std::optional<int64_t> divisor_override) {
425:   auto memory_format = at::MemoryFormat::ChannelsLast;
426:   auto grad_input = grad_input_.contiguous(memory_format);
427:   auto grad_output = grad_output_.contiguous(memory_format);
428:
429:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
430:   auto grad_output_data = grad_output.const_data_ptr<scalar_t>();
431:
432:   int64_t nbatch = grad_input.size(0);
433:   int64_t channels = grad_input.size(1);
434:   int64_t input_height = grad_input.size(2);
435:   int64_t input_width = grad_input.size(3);
436:   int64_t output_height = grad_output.size(2);
437:   int64_t output_width = grad_output.size(3);
438:
439:   using Vec = vec::Vectorized<scalar_t>;
440:   // parallel on dim N
441:   at::parallel_for(0, nbatch, 0, [&](int64_t begin, int64_t end) {
442:     for (const auto n : c10::irange(begin, end)) {
443:       scalar_t* grad_input_ptr = grad_input_data + n * input_height * input_width * channels;
444:       const scalar_t* grad_output_ptr = grad_output_data + n * output_height * output_width * channels;
```
- EN: The main symbol in this range is `cpu_avg_pool2d_backward_channels_last`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `cpu_avg_pool2d_backward_channels_last`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 446-474
```cpp
446:       for (const auto oh : c10::irange(output_height)) {
447:         for (const auto ow : c10::irange(output_width)) {
448:           int64_t ih0 = oh * dH - padH;
449:           int64_t iw0 = ow * dW - padW;
450:           int64_t ih1 = std::min(ih0 + kH, input_height + padH);
451:           int64_t iw1 = std::min(iw0 + kW, input_width + padW);
452:           int64_t pool_size = (ih1 - ih0) * (iw1 - iw0);
453:           ih0 = std::max(ih0, (int64_t) 0);
454:           iw0 = std::max(iw0, (int64_t) 0);
455:           ih1 = std::min(ih1, input_height);
456:           iw1 = std::min(iw1, input_width);
457:
458:           int64_t divide_factor = 0;
459:           if (divisor_override.has_value()) {
460:             divide_factor = divisor_override.value();
461:           } else {
462:             if(count_include_pad) {
463:               divide_factor = pool_size;
464:             } else {
465:               divide_factor = (ih1 - ih0) * (iw1 - iw0);
466:             }
467:           }
468:
469:           const scalar_t* gout = grad_output_ptr + oh * output_width * channels + ow * channels;
470:           int64_t size = channels;
471:           int64_t len = size - (size % Vec::size());
472:           for (const auto ih : c10::irange(ih0, ih1)) {
473:             for (const auto iw : c10::irange(iw0, iw1)) {
474:               scalar_t* gin = grad_input_ptr + ih * input_width * channels + iw * channels;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 476-504
```cpp
476:               int64_t d = 0;
477:               for (; d < len; d += Vec::size()) {
478:                 Vec gin_vec = Vec::loadu(gin + d) + Vec::loadu(gout + d) / Vec(scalar_t(divide_factor));
479:                 gin_vec.store(gin + d);
480:               }
481:               for (; d < size; d++) {
482:                 gin[d] += gout[d] / divide_factor;
483:               }
484:             }
485:           }
486:         }
487:       }
488:     }
489:   });
490:
491:   if (!grad_input_.is_contiguous(memory_format)) {
492:     grad_input_.copy_(grad_input);
493:   }
494: }
495:
496: void avg_pool2d_kernel_impl(
497:     const Tensor& output,
498:     const Tensor& input,
499:     int64_t kW, int64_t kH,
500:     int64_t dW, int64_t dH,
501:     int64_t padW, int64_t padH,
502:     bool count_include_pad,
503:     std::optional<int64_t> divisor_override) {
504:   switch (input.suggest_memory_format()) {
```
- EN: The main symbol in this range is `avg_pool2d_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `avg_pool2d_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 505-546
```cpp
505:     case at::MemoryFormat::Contiguous: {
506:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, input.scalar_type(), "avg_pool2d", [&] {
507:         cpu_avg_pool2d<scalar_t>(output, input, kW, kH, dW, dH, padW, padH, count_include_pad, divisor_override);
508:       });
509:       break;
510:     }
511:     case at::MemoryFormat::ChannelsLast: {
512:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, input.scalar_type(), "avg_pool2d_channels_last", [&] {
513:         cpu_avg_pool2d_channels_last<scalar_t>(output, input, kW, kH, dW, dH, padW, padH, count_include_pad, divisor_override);
514:       });
515:       break;
516:     }
517:     default:
518:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
519:   }
520: }
521:
522: void avg_pool2d_backward_kernel_impl(
523:     const Tensor& grad_input,
524:     const Tensor& grad_output,
525:     int kW, int kH,
526:     int dW, int dH,
527:     int padW, int padH,
528:     bool count_include_pad,
529:     std::optional<int64_t> divisor_override) {
530:   switch (grad_output.suggest_memory_format()) {
531:     case at::MemoryFormat::Contiguous: {
532:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, grad_output.scalar_type(), "avg_pool2d_backward", [&] {
533:         cpu_avg_pool2d_backward<scalar_t>(grad_input, grad_output, kW, kH, dW, dH, padW, padH, count_include_pad, divisor_override);
534:       });
535:       break;
536:     }
537:     case at::MemoryFormat::ChannelsLast: {
538:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, grad_output.scalar_type(), "avg_pool2d_backward_channels_last", [&] {
539:         cpu_avg_pool2d_backward_channels_last<scalar_t>(grad_input, grad_output, kW, kH, dW, dH, padW, padH, count_include_pad, divisor_override);
540:       });
541:       break;
542:     }
543:     default:
544:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
545:   }
546: }
```
- EN: The main symbol in this range is `avg_pool2d_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `avg_pool2d_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 549-583
```cpp
549: template <typename scalar_t>
550: void cpu_avg_pool3d(
551:     const Tensor& output_,
552:     const Tensor& input_,
553:     int64_t kW, int64_t kH, int64_t kD,
554:     int64_t dW, int64_t dH, int64_t dD,
555:     int64_t padW, int64_t padH, int64_t padD,
556:     bool count_include_pad,
557:     std::optional<int64_t> divisor_override) {
558:   using acc_t = at::opmath_type<scalar_t>;
559:
560:   auto input = input_.contiguous();
561:   auto output = output_.contiguous();
562:
563:   auto input_data = input.data_ptr<scalar_t>();
564:   auto output_data = output.data_ptr<scalar_t>();
565:
566:   int64_t numel = output.numel();
567:   int64_t ndim = input.ndimension();
568:   // treat batch size and channels as one dimension
569:   int64_t channels = ndim == 4 ? input.size(0) : input.size(0) * input.size(1);
570:   int64_t input_depth = input.size(-3);
571:   int64_t input_height = input.size(-2);
572:   int64_t input_width = input.size(-1);
573:   int64_t output_depth = output.size(-3);
574:   int64_t output_height = output.size(-2);
575:   int64_t output_width = output.size(-1);
576:
577:   // parallel on dim N, C, D, H, W
578:   at::parallel_for(0, numel, 0, [&](int64_t begin, int64_t end) {
579:     int64_t c = 0;
580:     int64_t od = 0;
581:     int64_t oh = 0;
582:     int64_t ow = 0;
583:     data_index_init(begin, c, channels, od, output_depth, oh, output_height, ow, output_width);
```
- EN: The main symbol in this range is `cpu_avg_pool3d`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_avg_pool3d`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 585-623
```cpp
585:     for (const auto i : c10::irange(begin, end)) {
586:       output_data[i] = static_cast<scalar_t>(0);
587:
588:       // local pointers
589:       scalar_t* input_ptr = input_data + c * input_depth * input_height * input_width;
590:
591:       // compute the mean of the input image...
592:       int64_t id0 = od * dD - padD;
593:       int64_t ih0 = oh * dH - padH;
594:       int64_t iw0 = ow * dW - padW;
595:       int64_t id1 = std::min(id0 + kD, input_depth + padD);
596:       int64_t ih1 = std::min(ih0 + kH, input_height + padH);
597:       int64_t iw1 = std::min(iw0 + kW, input_width + padW);
598:       int64_t pool_size = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
599:       id0 = std::max(id0, (int64_t) 0);
600:       ih0 = std::max(ih0, (int64_t) 0);
601:       iw0 = std::max(iw0, (int64_t) 0);
602:       id1 = std::min(id1, input_depth);
603:       ih1 = std::min(ih1, input_height);
604:       iw1 = std::min(iw1, input_width);
605:
606:       if (id0 >= id1 || ih0 >= ih1 || iw0 >= iw1) {
607:         // move on to next output index
608:         data_index_step(c, channels, od, output_depth, oh, output_height, ow, output_width);
609:         continue;
610:       }
611:
612:       acc_t sum = 0;
613:
614:       int64_t divide_factor = 0;
615:       if (divisor_override.has_value()) {
616:         divide_factor = divisor_override.value();
617:       } else {
618:         if(count_include_pad) {
619:           divide_factor = pool_size;
620:         } else {
621:           divide_factor = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
622:         }
623:       }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 625-658
```cpp
625:       for (const auto id : c10::irange(id0, id1)) {
626:         for (const auto ih : c10::irange(ih0, ih1)) {
627:           for (const auto iw : c10::irange(iw0, iw1)) {
628:             sum += input_ptr[id * input_height * input_width + ih * input_width + iw];
629:           }
630:         }
631:       }
632:       output_data[i] += scalar_t(sum / divide_factor);
633:
634:       // move on to next output index
635:       data_index_step(c, channels, od, output_depth, oh, output_height, ow, output_width);
636:     }
637:   });
638:
639:   if (!output_.is_contiguous()) {
640:     output_.copy_(output);
641:   }
642: }
643:
644: template <typename scalar_t,
645:           std::enable_if_t<!is_reduced_floating_point<scalar_t>::value, int> = 0>
646: void cpu_avg_pool3d_channels_last(
647:     const Tensor& output_,
648:     const Tensor& input_,
649:     int64_t kW, int64_t kH, int64_t kD,
650:     int64_t dW, int64_t dH, int64_t dD,
651:     int64_t padW, int64_t padH, int64_t padD,
652:     bool count_include_pad,
653:     std::optional<int64_t> divisor_override) {
654:   TORCH_CHECK(input_.ndimension() == 5,
655:               "3d average pooling with channels last format supports tensors with 5 dims");
656:   auto memory_format = at::MemoryFormat::ChannelsLast3d;
657:   auto input = input_.contiguous(memory_format);
658:   auto output = output_.contiguous(memory_format);
```
- EN: The main symbol in this range is `cpu_avg_pool3d_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_avg_pool3d_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 660-697
```cpp
660:   auto input_data = input.data_ptr<scalar_t>();
661:   auto output_data = output.data_ptr<scalar_t>();
662:
663:   int64_t nbatch = input.size(0);
664:   int64_t channels = input.size(1);
665:   int64_t input_depth = input.size(2);
666:   int64_t input_height = input.size(3);
667:   int64_t input_width = input.size(4);
668:   int64_t output_depth = output.size(2);
669:   int64_t output_height = output.size(3);
670:   int64_t output_width = output.size(4);
671:
672:   using Vec = vec::Vectorized<scalar_t>;
673:   // parallel on dim N, H, W
674:   at::parallel_for(0, nbatch * output_depth * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
675:     int64_t n = 0;
676:     int64_t od = 0;
677:     int64_t oh = 0;
678:     int64_t ow = 0;
679:     data_index_init(begin, n, nbatch, od, output_depth, oh, output_height, ow, output_width);
680:
681:     int64_t size = channels;
682:     int64_t len = size - (size % Vec::size());
683:     for (const auto i : c10::irange(begin, end)) {
684:       // compute the mean of the input image...
685:       int64_t id0 = od * dD - padD;
686:       int64_t ih0 = oh * dH - padH;
687:       int64_t iw0 = ow * dW - padW;
688:       int64_t id1 = std::min(id0 + kD, input_depth + padD);
689:       int64_t ih1 = std::min(ih0 + kH, input_height + padH);
690:       int64_t iw1 = std::min(iw0 + kW, input_width + padW);
691:       int64_t pool_size = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
692:       id0 = std::max(id0, (int64_t) 0);
693:       ih0 = std::max(ih0, (int64_t) 0);
694:       iw0 = std::max(iw0, (int64_t) 0);
695:       id1 = std::min(id1, input_depth);
696:       ih1 = std::min(ih1, input_height);
697:       iw1 = std::min(iw1, input_width);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 699-733
```cpp
699:       int64_t divide_factor = 0;
700:       if (divisor_override.has_value()) {
701:         divide_factor = divisor_override.value();
702:       } else {
703:         if(count_include_pad) {
704:           divide_factor = pool_size;
705:         } else {
706:           divide_factor = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
707:         }
708:       }
709:
710:       scalar_t* out = output_data + i * channels;
711:
712:       // Pass I: zero the out lane
713:       int64_t d1 = 0;
714:       for (; d1 < len; d1 += Vec::size()) {
715:         Vec out_vec = Vec(scalar_t(0));
716:         out_vec.store(out + d1);
717:       }
718:       for (; d1 < size; d1++) {
719:         out[d1] = scalar_t(0);
720:       }
721:
722:       if (id0 >= id1 || ih0 >= ih1 || iw0 >= iw1) {
723:         // move on to next output index
724:         data_index_step(n, nbatch, od, output_depth, oh, output_height, ow, output_width);
725:         continue;
726:       }
727:
728:       // Pass II: compute local sum
729:       for (const auto id : c10::irange(id0, id1)) {
730:         for (const auto ih : c10::irange(ih0, ih1)) {
731:           for (const auto iw : c10::irange(iw0, iw1)) {
732:             scalar_t* in = input_data + n * input_depth * input_height * input_width * channels +
733:                 id * input_height * input_width * channels + ih * input_width * channels + iw * channels;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 735-765
```cpp
735:             int64_t d2 = 0;
736:             for (; d2 < len; d2 += Vec::size()) {
737:               Vec out_vec = Vec::loadu(out + d2) + Vec::loadu(in + d2);
738:               out_vec.store(out + d2);
739:             }
740:             for (; d2 < size; d2++) {
741:               out[d2] += in[d2];
742:             }
743:           }
744:         }
745:       }
746:
747:       // Pass III: compute local average
748:       int64_t d3 = 0;
749:       for (; d3 < len; d3 += Vec::size()) {
750:         Vec out_vec = Vec::loadu(out + d3) / Vec(scalar_t(divide_factor));
751:         out_vec.store(out + d3);
752:       }
753:       for (; d3 < size; d3++) {
754:         out[d3] = out[d3] / divide_factor;
755:       }
756:
757:       // move on to next output index
758:       data_index_step(n, nbatch, od, output_depth, oh, output_height, ow, output_width);
759:     }
760:   });
761:
762:   if (!output_.is_contiguous(memory_format)) {
763:     output_.copy_(output);
764:   }
765: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 767-803
```cpp
767: template <typename scalar_t,
768:           std::enable_if_t<is_reduced_floating_point<scalar_t>::value, int> = 0>
769: void cpu_avg_pool3d_channels_last(
770:     const Tensor& output_,
771:     const Tensor& input_,
772:     int64_t kW, int64_t kH, int64_t kD,
773:     int64_t dW, int64_t dH, int64_t dD,
774:     int64_t padW, int64_t padH, int64_t padD,
775:     bool count_include_pad,
776:     std::optional<int64_t> divisor_override) {
777:   TORCH_CHECK(input_.ndimension() == 5,
778:               "3d average pooling with channels last format supports tensors with 5 dims");
779:   auto memory_format = at::MemoryFormat::ChannelsLast3d;
780:   auto input = input_.contiguous(memory_format);
781:   auto output = output_.contiguous(memory_format);
782:
783:   auto input_data = input.data_ptr<BFloat16>();
784:   auto output_data = output.data_ptr<BFloat16>();
785:
786:   int64_t nbatch = input.size(0);
787:   int64_t channels = input.size(1);
788:   int64_t input_depth = input.size(2);
789:   int64_t input_height = input.size(3);
790:   int64_t input_width = input.size(4);
791:   int64_t output_depth = output.size(2);
792:   int64_t output_height = output.size(3);
793:   int64_t output_width = output.size(4);
794:
795:   using bVec = vec::Vectorized<BFloat16>;
796:   using fVec = vec::Vectorized<float>;
797:   // parallel on dim N, H, W
798:   at::parallel_for(0, nbatch * output_depth * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
799:     int64_t n = 0;
800:     int64_t od = 0;
801:     int64_t oh = 0;
802:     int64_t ow = 0;
803:     data_index_init(begin, n, nbatch, od, output_depth, oh, output_height, ow, output_width);
```
- EN: The main symbol in this range is `cpu_avg_pool3d_channels_last`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `cpu_avg_pool3d_channels_last`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 805-836
```cpp
805:     // temp buffer for sum, use float as accumulation type
806:     // can't reuse output buffer to store sum since it is BFloat16
807:     auto sum_arr = std::make_unique<float []>(channels);
808:     float* sum = sum_arr.get();
809:
810:     int64_t size = channels;
811:     for (const auto i : c10::irange(begin, end)) {
812:       // compute the mean of the input image...
813:       int64_t id0 = od * dD - padD;
814:       int64_t ih0 = oh * dH - padH;
815:       int64_t iw0 = ow * dW - padW;
816:       int64_t id1 = std::min(id0 + kD, input_depth + padD);
817:       int64_t ih1 = std::min(ih0 + kH, input_height + padH);
818:       int64_t iw1 = std::min(iw0 + kW, input_width + padW);
819:       int64_t pool_size = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
820:       id0 = std::max(id0, (int64_t) 0);
821:       ih0 = std::max(ih0, (int64_t) 0);
822:       iw0 = std::max(iw0, (int64_t) 0);
823:       id1 = std::min(id1, input_depth);
824:       ih1 = std::min(ih1, input_height);
825:       iw1 = std::min(iw1, input_width);
826:
827:       int64_t divide_factor = 0;
828:       if (divisor_override.has_value()) {
829:         divide_factor = divisor_override.value();
830:       } else {
831:         if(count_include_pad) {
832:           divide_factor = pool_size;
833:         } else {
834:           divide_factor = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
835:         }
836:       }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 838-866
```cpp
838:       BFloat16* out = output_data + i * channels;
839:
840:       // Pass I: zero the out lane
841:       int64_t d1 = 0;
842:       for (; d1 < size - (size % fVec::size()); d1 += fVec::size()) {
843:         fVec sum_fvec = fVec(float(0));
844:         sum_fvec.store(sum + d1);
845:       }
846:       for (; d1 < size; d1++) {
847:         sum[d1] = float(0);
848:       }
849:
850:       if (id0 >= id1 || ih0 >= ih1 || iw0 >= iw1) {
851:         // since we are not directly using output as the accumulation buffer,
852:         // in case the kernel window is out of range, need to zero the output buffer here.
853:         for (int64_t k = 0; k < size; k++) {
854:           out[k] = 0;
855:         }
856:         // move on to next output index
857:         data_index_step(n, nbatch, od, output_depth, oh, output_height, ow, output_width);
858:         continue;
859:       }
860:
861:       // Pass II: compute local sum
862:       for (const auto id : c10::irange(id0, id1)) {
863:         for (const auto ih : c10::irange(ih0, ih1)) {
864:           for (const auto iw : c10::irange(iw0, iw1)) {
865:             BFloat16* in = input_data + n * input_depth * input_height * input_width * channels +
866:                 id * input_height * input_width * channels + ih * input_width * channels + iw * channels;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 868-896
```cpp
868:             int64_t d2 = 0;
869:             for (; d2 < size - (size % bVec::size()); d2 += bVec::size()) {
870:               bVec data_bvec = bVec::loadu(in + d2);
871:               auto [data_fvec0, data_fvec1] = convert_bfloat16_float(data_bvec);
872:
873:               fVec sum_fvec0 = fVec::loadu(sum + d2) + data_fvec0;
874:               fVec sum_fvec1 = fVec::loadu(sum + d2 + fVec::size()) + data_fvec1;
875:               sum_fvec0.store(sum + d2);
876:               sum_fvec1.store(sum + d2 + fVec::size());
877:             }
878:             for (; d2 < size; d2++) {
879:               sum[d2] += float(in[d2]);
880:             }
881:           }
882:         }
883:       }
884:
885:       // Pass III: compute local average
886:       int64_t d3 = 0;
887:       for (; d3 < size - (size % bVec::size()); d3 += bVec::size()) {
888:         fVec out_fvec0 = fVec::loadu(sum + d3) / fVec(float(divide_factor));
889:         fVec out_fvec1 = fVec::loadu(sum + d3 + fVec::size()) / fVec(float(divide_factor));
890:
891:         bVec out_bvec = convert_float_bfloat16(out_fvec0, out_fvec1);
892:         out_bvec.store(out + d3);
893:       }
894:       for (; d3 < size; d3++) {
895:         out[d3] = BFloat16(sum[d3] / divide_factor);
896:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 898-931
```cpp
898:       // move on to next output index
899:       data_index_step(n, nbatch, od, output_depth, oh, output_height, ow, output_width);
900:     }
901:   });
902:
903:   if (!output_.is_contiguous(memory_format)) {
904:     output_.copy_(output);
905:   }
906: }
907:
908: template <typename scalar_t>
909: void cpu_avg_pool3d_backward(
910:     const Tensor& grad_input_,
911:     const Tensor& grad_output_,
912:     int kW, int kH, int kD,
913:     int dW, int dH, int dD,
914:     int padW, int padH, int padD,
915:     bool count_include_pad,
916:     std::optional<int64_t> divisor_override) {
917:   auto grad_output = grad_output_.contiguous();
918:   auto grad_input = grad_input_.contiguous();
919:
920:   auto grad_output_data = grad_output.data_ptr<scalar_t>();
921:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
922:
923:   int64_t ndim = grad_output.ndimension();
924:   // treat batch size and channels as one dimension
925:   int64_t channels = ndim == 4 ? grad_output.size(0) : grad_output.size(0) * grad_output.size(1);
926:   int64_t input_depth = grad_input.size(-3);
927:   int64_t input_height = grad_input.size(-2);
928:   int64_t input_width = grad_input.size(-1);
929:   int64_t output_depth = grad_output.size(-3);
930:   int64_t output_height = grad_output.size(-2);
931:   int64_t output_width = grad_output.size(-1);
```
- EN: The main symbol in this range is `cpu_avg_pool3d_backward`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `cpu_avg_pool3d_backward`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 933-964
```cpp
933:   // parallel on dim of N, C
934:   at::parallel_for(0, channels, 0, [&](int64_t begin, int64_t end) {
935:     for (const auto c : c10::irange(begin, end)) {
936:       scalar_t* grad_input_ptr = grad_input_data + c * input_depth * input_height * input_width;
937:       scalar_t* grad_output_ptr = grad_output_data + c * output_depth * output_height * output_width;
938:
939:       for (const auto od : c10::irange(output_depth)) {
940:         for (const auto oh : c10::irange(output_height)) {
941:           for (const auto ow : c10::irange(output_width)) {
942:             int64_t id0 = od * dD - padD;
943:             int64_t ih0 = oh * dH - padH;
944:             int64_t iw0 = ow * dW - padW;
945:             int64_t id1 = std::min(id0 + kD, input_depth + padD);
946:             int64_t ih1 = std::min(ih0 + kH, input_height + padH);
947:             int64_t iw1 = std::min(iw0 + kW, input_width + padW);
948:             int64_t pool_size = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
949:             id0 = std::max(id0, (int64_t) 0);
950:             ih0 = std::max(ih0, (int64_t) 0);
951:             iw0 = std::max(iw0, (int64_t) 0);
952:             ih1 = std::min(ih1, input_height);
953:             iw1 = std::min(iw1, input_width);
954:
955:             int64_t divide_factor = 0;
956:             if (divisor_override.has_value()) {
957:               divide_factor = divisor_override.value();
958:             } else {
959:               if(count_include_pad) {
960:                 divide_factor = pool_size;
961:               } else {
962:                 divide_factor = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
963:               }
964:             }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 966-996
```cpp
966:             scalar_t grad_delta = grad_output_ptr[od * output_height * output_width + oh * output_width + ow] / divide_factor;
967:             for (const auto id : c10::irange(id0, id1)) {
968:               for (const auto ih : c10::irange(ih0, ih1)) {
969:                 for (const auto iw : c10::irange(iw0, iw1)) {
970:                   grad_input_ptr[id * input_height * input_width + ih * input_width + iw] += grad_delta;
971:                 }
972:               }
973:             }
974:           }
975:         }
976:       }
977:     }
978:   });
979:
980:   if (!grad_input_.is_contiguous()) {
981:     grad_input_.copy_(grad_input);
982:   }
983: }
984:
985: template <typename scalar_t>
986: void cpu_avg_pool3d_backward_channels_last(
987:     const Tensor& grad_input_,
988:     const Tensor& grad_output_,
989:     int kW, int kH, int kD,
990:     int dW, int dH, int dD,
991:     int padW, int padH, int padD,
992:     bool count_include_pad,
993:     std::optional<int64_t> divisor_override) {
994:   auto memory_format = at::MemoryFormat::ChannelsLast3d;
995:   auto grad_input = grad_input_.contiguous(memory_format);
996:   auto grad_output = grad_output_.contiguous(memory_format);
```
- EN: The main symbol in this range is `cpu_avg_pool3d_backward_channels_last`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_avg_pool3d_backward_channels_last`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 998-1032
```cpp
 998:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
 999:   auto grad_output_data = grad_output.data_ptr<scalar_t>();
1000:
1001:   int64_t nbatch = grad_input.size(0);
1002:   int64_t channels = grad_input.size(1);
1003:   int64_t input_depth = grad_input.size(2);
1004:   int64_t input_height = grad_input.size(3);
1005:   int64_t input_width = grad_input.size(4);
1006:   int64_t output_depth = grad_output.size(2);
1007:   int64_t output_height = grad_output.size(3);
1008:   int64_t output_width = grad_output.size(4);
1009:
1010:   using Vec = vec::Vectorized<scalar_t>;
1011:   // parallel on dim N
1012:   at::parallel_for(0, nbatch, 0, [&](int64_t begin, int64_t end) {
1013:     for (const auto n : c10::irange(begin, end)) {
1014:       scalar_t* grad_input_ptr = grad_input_data + n * input_depth * input_height * input_width * channels;
1015:       scalar_t* grad_output_ptr = grad_output_data + n * output_height * output_width * channels;
1016:
1017:       for (const auto od : c10::irange(output_depth)) {
1018:         for (const auto oh : c10::irange(output_height)) {
1019:           for (const auto ow : c10::irange(output_width)) {
1020:             int64_t id0 = od * dD - padD;
1021:             int64_t ih0 = oh * dH - padH;
1022:             int64_t iw0 = ow * dW - padW;
1023:             int64_t id1 = std::min(id0 + kD, input_depth + padD);
1024:             int64_t ih1 = std::min(ih0 + kH, input_height + padH);
1025:             int64_t iw1 = std::min(iw0 + kW, input_width + padW);
1026:             int64_t pool_size = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
1027:             id0 = std::max(id0, (int64_t) 0);
1028:             ih0 = std::max(ih0, (int64_t) 0);
1029:             iw0 = std::max(iw0, (int64_t) 0);
1030:             id1 = std::min(id1, input_depth);
1031:             ih1 = std::min(ih1, input_height);
1032:             iw1 = std::min(iw1, input_width);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1034-1068
```cpp
1034:             int64_t divide_factor = 0;
1035:             if (divisor_override.has_value()) {
1036:               divide_factor = divisor_override.value();
1037:             } else {
1038:               if(count_include_pad) {
1039:                 divide_factor = pool_size;
1040:               } else {
1041:                 divide_factor = (id1 - id0) * (ih1 - ih0) * (iw1 - iw0);
1042:               }
1043:             }
1044:
1045:             scalar_t* gout = grad_output_ptr + od * output_height * output_width * channels + oh * output_width * channels + ow * channels;
1046:             int64_t size = channels;
1047:             int64_t len = size - (size % Vec::size());
1048:             for (const auto id : c10::irange(id0, id1)) {
1049:               for (const auto ih : c10::irange(ih0, ih1)) {
1050:                 for (const auto iw : c10::irange(iw0, iw1)) {
1051:                   scalar_t* gin = grad_input_ptr + id * input_height * input_width * channels + ih * input_width * channels + iw * channels;
1052:
1053:                   int64_t d = 0;
1054:                   for (; d < len; d += Vec::size()) {
1055:                     Vec gin_vec = Vec::loadu(gin + d) + Vec::loadu(gout + d) / Vec(scalar_t(divide_factor));
1056:                     gin_vec.store(gin + d);
1057:                   }
1058:                   for (; d < size; d++) {
1059:                     gin[d] += gout[d] / divide_factor;
1060:                   }
1061:                 }
1062:               }
1063:             }
1064:           }
1065:         }
1066:       }
1067:     }
1068:   });
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 1070-1101
```cpp
1070:   if (!grad_input_.is_contiguous(memory_format)) {
1071:     grad_input_.copy_(grad_input);
1072:   }
1073: }
1074:
1075:
1076:
1077: void avg_pool3d_kernel_impl(
1078:     const Tensor& output,
1079:     const Tensor& input,
1080:     int64_t kW, int64_t kH, int64_t kD,
1081:     int64_t dW, int64_t dH, int64_t dD,
1082:     int64_t padW, int64_t padH, int64_t padD,
1083:     bool count_include_pad,
1084:     std::optional<int64_t> divisor_override) {
1085:   switch (input.suggest_memory_format()) {
1086:     case at::MemoryFormat::Contiguous: {
1087:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, input.scalar_type(), "avg_pool3d", [&] {
1088:         cpu_avg_pool3d<scalar_t>(output, input, kW, kH, kD, dW, dH, dD, padW, padH, padD, count_include_pad, divisor_override);
1089:       });
1090:       break;
1091:     }
1092:     case at::MemoryFormat::ChannelsLast: {
1093:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, input.scalar_type(), "avg_pool3d_channels_last", [&] {
1094:         cpu_avg_pool3d_channels_last<scalar_t>(output, input, kW, kH, kD, dW, dH, dD, padW, padH, padD, count_include_pad, divisor_override);
1095:       });
1096:       break;
1097:     }
1098:     default:
1099:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
1100:   }
1101: }
```
- EN: The main symbol in this range is `avg_pool3d_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `avg_pool3d_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1104-1131
```cpp
1104: void avg_pool3d_backward_kernel_impl(
1105:     const Tensor& grad_input,
1106:     const Tensor& grad_output,
1107:     int kW, int kH, int kD,
1108:     int dW, int dH, int dD,
1109:     int padW, int padH, int padD,
1110:     bool count_include_pad,
1111:     std::optional<int64_t> divisor_override) {
1112:   switch (grad_output.suggest_memory_format()) {
1113:     case at::MemoryFormat::Contiguous: {
1114:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, grad_output.scalar_type(), "avg_pool3d_backward", [&] {
1115:         cpu_avg_pool3d_backward<scalar_t>(grad_input, grad_output, kW, kH, kD, dW, dH, dD, padW, padH, padD, count_include_pad, divisor_override);
1116:       });
1117:       break;
1118:     }
1119:     case at::MemoryFormat::ChannelsLast3d: {
1120:       AT_DISPATCH_FLOATING_TYPES_AND3(kLong, kBFloat16, kHalf, grad_output.scalar_type(), "avg_pool3d_backward_channels_last", [&] {
1121:         cpu_avg_pool3d_backward_channels_last<scalar_t>(grad_input, grad_output, kW, kH, kD, dW, dH, dD, padW, padH, padD, count_include_pad, divisor_override);
1122:       });
1123:       break;
1124:     }
1125:     default:
1126:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
1127:   }
1128: }
1129:
1130:
1131: } // anonymous namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `avg_pool3d_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `avg_pool3d_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1133-1138
```cpp
1133: REGISTER_DISPATCH(avg_pool2d_kernel, &avg_pool2d_kernel_impl)
1134: REGISTER_DISPATCH(avg_pool2d_backward_kernel, &avg_pool2d_backward_kernel_impl)
1135: REGISTER_DISPATCH(avg_pool3d_kernel, &avg_pool3d_kernel_impl)
1136: REGISTER_DISPATCH(avg_pool3d_backward_kernel, &avg_pool3d_backward_kernel_impl)
1137:
1138: } // at::native
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/OpMathType.h`, `ATen/cpu/vec/vec.h`, `ATen/cpu/vec/functional.h`, `ATen/native/Pool.h`, `ATen/native/cpu/utils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`
