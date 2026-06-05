# SoftMaxKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/SoftMaxKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Soft Max Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Soft Max Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: #include <memory>
 2: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 3: #include <ATen/native/cpu/SoftmaxKernel.h>
 4:
 5: #include <ATen/native/cpu/LogSoftmaxKernelImpl.h>
 6:
 7: #include <algorithm>
 8: #include <iterator>
 9: #include <numeric>
10: #include <vector>
11:
12: #include <ATen/Dispatch.h>
13: #include <ATen/Parallel.h>
14: #include <ATen/TensorIterator.h>
15: #include <ATen/OpMathType.h>
16: #include <ATen/core/Tensor.h>
17: #include <ATen/cpu/vec/functional.h>
18: #include <ATen/cpu/vec/vec.h>
19: #include <c10/util/irange.h>
20:
21: // [Note AVX-SSE transitions] In general we avoid calls into cmath for code
22: // compiled with AVX/AVX2 This is because of SSE-AVX transitions and a bug in
23: // Glibc2.23 See https://bugs.launchpad.net/ubuntu/+source/glibc/+bug/1663280
24: //
25: // On grainsize: The grainsize is chosen to roughly get GRAIN_SIZE number of
26: // computations per task. Each task works across dim_size elements. 16 should be
27: // a very rough approximation of the number of computations per dim_size element
28: // by counting simple computations (*, +, -) as 1 and exp or log as 4.
29: //
30: // We use a chunk size such that it'd fit in L1D.
```
- EN: This range pulls in required headers, including `memory`, `ATen/native/cpu/SoftmaxKernel.h`, `ATen/native/cpu/LogSoftmaxKernelImpl.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `memory`, `ATen/native/cpu/SoftmaxKernel.h`, `ATen/native/cpu/LogSoftmaxKernelImpl.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 32-59
```cpp
32: namespace at::native {
33: namespace {
34: template <typename scalar_t>
35: inline void _vec_log_softmax_lastdim(
36:     const scalar_t* input_data_base,
37:     scalar_t* output_data_base,
38:     int64_t outer_size,
39:     int64_t dim_size) {
40:   const auto chunk_size = vec_log_softmax_lastdim_chunk_size<scalar_t>(
41:       at::internal::GRAIN_SIZE,
42:       outer_size,
43:       dim_size);
44:   // Note: grain_size value of 0
45:   // We don't change the number of OpenMP threads in the OpenMP thread-pool,
46:   // so some threads do useful work, while others don't.
47:   // We can simply use grain_size of 0 & rely upon invoke_parallel to distribute
48:   // work among threads in an equitable manner. We compute CHUNK_SIZE to ensure
49:   // each thread's computations would be efficient.
50:   parallel_for(0, outer_size, 0, [&](int64_t begin, int64_t end) {
51:     serial_vec_log_softmax_lastdim_range(
52:         input_data_base,
53:         output_data_base,
54:         dim_size,
55:         chunk_size,
56:         begin,
57:         end);
58:   });
59: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_vec_log_softmax_lastdim`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_vec_log_softmax_lastdim`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 61-93
```cpp
61: template<typename scalar_t>
62: inline typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
63: _vec_softmax_lastdim(
64:     const scalar_t* input_data_base,
65:     scalar_t* output_data_base,
66:     int64_t outer_size,
67:     int64_t dim_size) {
68:   using Vec = vec::Vectorized<scalar_t>;
69:   // See Note: grain_size value of 0
70:   parallel_for(0, outer_size, 0, [&](int64_t begin, int64_t end) {
71:     for (const auto i : c10::irange(begin, end)) {
72:       const scalar_t* input_data = input_data_base + i * dim_size;
73:       scalar_t* output_data = output_data_base + i * dim_size;
74:       scalar_t max_input = vec::reduce_all<scalar_t>(
75:           [](Vec& x, Vec& y) { return vec::maximum(x, y); },
76:           input_data,
77:           dim_size);
78:       vec::map(
79:           [max_input](Vec x) { return (x - Vec(max_input)).exp(); },
80:           output_data,
81:           input_data,
82:           dim_size);
83:       scalar_t tmp_sum = vec::reduce_all<scalar_t>(
84:           [](Vec x, Vec y) { return x + y; }, output_data, dim_size);
85:       tmp_sum = 1 / tmp_sum;
86:       vec::map(
87:           [tmp_sum](Vec x) { return x * Vec(tmp_sum); },
88:           output_data,
89:           output_data,
90:           dim_size);
91:     }
92:   });
93: }
```
- EN: The main symbol in this range is `_vec_softmax_lastdim`, `reduce_all<scalar_t>`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `_vec_softmax_lastdim`, `reduce_all<scalar_t>`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 95-129
```cpp
 95: template<typename scalar_t>
 96: inline typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
 97: _vec_softmax_lastdim(
 98:     const scalar_t* input_data_base,
 99:     scalar_t* output_data_base,
100:     int64_t outer_size,
101:     int64_t dim_size) {
102:   using Vec = vec::Vectorized<scalar_t>;
103:   using fVec = vec::Vectorized<float>;
104:   // See Note: grain_size value of 0
105:   parallel_for(0, outer_size, 0, [&](int64_t begin, int64_t end) {
106:     // thread local temp buffer.
107:     auto buffer = std::make_unique<float []>(dim_size);
108:     float* buffer_data = buffer.get();
109:
110:     for (const auto i : c10::irange(begin, end)) {
111:       const scalar_t* input_data = input_data_base + i * dim_size;
112:       scalar_t* output_data = output_data_base + i * dim_size;
113:       // reduce to max and cache float input data
114:       fVec max_fvec = fVec(-std::numeric_limits<float>::infinity());
115:       int64_t d0 = 0;
116:       for (; d0 < dim_size - (dim_size % Vec::size()); d0 += Vec::size()) {
117:         Vec data_vec = Vec::loadu(input_data + d0);
118:         auto [data_fvec0, data_fvec1] = vec::convert_to_float<scalar_t>(data_vec);
119:         max_fvec = vec::maximum(max_fvec, data_fvec0);
120:         max_fvec = vec::maximum(max_fvec, data_fvec1);
121:         data_fvec0.store(buffer_data + d0);
122:         data_fvec1.store(buffer_data + d0 + fVec::size());
123:       }
124:       float max_val = vec::vec_reduce_all([](fVec& x, fVec& y) { return vec::maximum(x, y); }, max_fvec);
125:       for (; d0 < dim_size; d0++) {
126:         float data_val = input_data[d0];
127:         max_val = std::max(max_val, data_val);
128:         buffer_data[d0] = data_val;
129:       }
```
- EN: The main symbol in this range is `_vec_softmax_lastdim`, `vec_reduce_all`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `_vec_softmax_lastdim`, `vec_reduce_all`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 131-159
```cpp
131:       // map (x - max).exp() and reduce to sum
132:       fVec sum_fvec = fVec(float(0));
133:       int64_t d1 = 0;
134:       for (; d1 < dim_size - (dim_size % fVec::size()); d1 += fVec::size()) {
135:         fVec data_fvec = (fVec::loadu(buffer_data + d1) - fVec(max_val)).exp();
136:         sum_fvec += data_fvec;
137:         data_fvec.store(buffer_data + d1);
138:       }
139:       float sum_val = vec::vec_reduce_all([](fVec& x, fVec& y) { return x + y; }, sum_fvec);
140:       for (; d1 < dim_size; d1++) {
141:         float data_val = std::exp(buffer_data[d1] - max_val);
142:         sum_val += data_val;
143:         buffer_data[d1] = data_val;
144:       }
145:
146:       sum_val = 1 / sum_val;
147:       int64_t d2 = 0;
148:       for (; d2 < dim_size - (dim_size % Vec::size()); d2 += Vec::size()) {
149:         fVec out_fvec0 = fVec::loadu(buffer_data + d2) * fVec(sum_val);
150:         fVec out_fvec1 = fVec::loadu(buffer_data + d2 + fVec::size()) * fVec(sum_val);
151:         Vec out_vec = vec::convert_from_float<scalar_t>(out_fvec0, out_fvec1);
152:         out_vec.store(output_data + d2);
153:       }
154:       for (; d2 < dim_size; d2++) {
155:         output_data[d2] = scalar_t(buffer_data[d2] * sum_val);
156:       }
157:     }
158:   });
159: }
```
- EN: The main symbol in this range is `vec_reduce_all`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `vec_reduce_all`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 161-189
```cpp
161: template <typename scalar_t, bool log_softmax>
162: inline void _vec_host_softmax_backward_lastdim(
163:     scalar_t* grad_input_data_base,
164:     const scalar_t* grad_data_base,
165:     const scalar_t* output_data_base,
166:     int64_t outer_size,
167:     int64_t dim_size) {
168:   using Vec = vec::Vectorized<at::opmath_type<scalar_t>>;
169:   // See Note: grain_size value of 0
170:   parallel_for(
171:       0,
172:       outer_size,
173:       0,
174:       [&](int64_t begin, int64_t end) {
175:         for (const auto i : c10::irange(begin, end)) {
176:           scalar_t* grad_input_data = grad_input_data_base + i * dim_size;
177:           const scalar_t* grad_data = grad_data_base + i * dim_size;
178:           const scalar_t* output_data = output_data_base + i * dim_size;
179:           if constexpr (log_softmax) {
180:             auto sum = vec::reduce_all<scalar_t>(
181:                 [](Vec& x, Vec& y) { return x + y; }, grad_data, dim_size);
182:             vec::map2(
183:                 [sum](Vec x, Vec y) { return x - ((y.exp()) * Vec(sum)); },
184:                 grad_input_data,
185:                 grad_data,
186:                 output_data,
187:                 dim_size);
188:           } else {
189:             auto sum = vec::map2_reduce_all<scalar_t>(
```
- EN: The main symbol in this range is `_vec_host_softmax_backward_lastdim`, `constexpr`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `_vec_host_softmax_backward_lastdim`, `constexpr`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 190-228
```cpp
190:                 [](Vec x, Vec y) { return x * y; },
191:                 [](Vec x, Vec y) { return x + y; },
192:                 grad_data,
193:                 output_data,
194:                 dim_size);
195:             vec::map2(
196:                 [sum](Vec x, Vec y) { return (x - Vec(sum)) * y; },
197:                 grad_input_data,
198:                 grad_data,
199:                 output_data,
200:                 dim_size);
201:           }
202:         }
203:       });
204: }
205:
206: template<typename scalar_t>
207: inline typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
208: _vec_softmax_backward(
209:     scalar_t* grad_input_data_base,
210:     const scalar_t* grad_output_data_base,
211:     const scalar_t* output_data_base,
212:     int64_t outer_size,
213:     int64_t inner_size,
214:     int64_t dim_size) {
215:   using Vec = vec::Vectorized<scalar_t>;
216:   int64_t outer_stride = dim_size * inner_size;
217:   int64_t BLOCK_SIZE = 128 * 1024;
218:   int64_t MAX_CHUNK_SIZE = std::max<int64_t>(
219:       BLOCK_SIZE / dim_size / sizeof(scalar_t), Vec::size());
220:   MAX_CHUNK_SIZE = MAX_CHUNK_SIZE / Vec::size() * Vec::size();
221:   int64_t CHUNK_SIZE = std::min<int64_t>(MAX_CHUNK_SIZE, inner_size);
222:   int64_t num_chunks = divup(inner_size, CHUNK_SIZE);
223:   // See Note: grain_size value of 0
224:   parallel_for(
225:       0, outer_size * num_chunks, 0, [&](int64_t begin, int64_t end) {
226:         // thread local temp buffer that holds vertical sum result
227:         auto buffer = std::make_unique<scalar_t[]>(CHUNK_SIZE);
228:         scalar_t* tmp_sum_data = buffer.get();
```
- EN: The main symbol in this range is `map2`, `_vec_softmax_backward`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `map2`, `_vec_softmax_backward`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 230-264
```cpp
230:         for (int64_t i = begin; i < end; i++) {
231:           int64_t outer_idx = i / num_chunks;
232:           int64_t k = i % num_chunks;
233:           int64_t inner_idx_begin = k * CHUNK_SIZE;
234:           int64_t size = std::min(CHUNK_SIZE, inner_size - inner_idx_begin);
235:
236:           // init
237:           Vec zero_vec = Vec(scalar_t(0));
238:           int64_t d0 = 0;
239:           for (; d0 < size - (size % Vec::size()); d0 += Vec::size()) {
240:             zero_vec.store(tmp_sum_data + d0);
241:           }
242:           for (; d0 < size; d0++) {
243:             tmp_sum_data[d0] = scalar_t(0);
244:           }
245:
246:           // compute sum of grad_output * output
247:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
248:             int64_t offset = outer_idx * outer_stride + dim_idx * inner_size +
249:                 inner_idx_begin;
250:             const scalar_t* grad_output_ptr = grad_output_data_base + offset;
251:             const scalar_t* output_ptr = output_data_base + offset;
252:
253:             int64_t d1 = 0;
254:             for (; d1 < size - (size % Vec::size()); d1 += Vec::size()) {
255:               Vec grad_output_vec = Vec::loadu(grad_output_ptr + d1);
256:               Vec output_vec = Vec::loadu(output_ptr + d1);
257:               Vec sum_vec = Vec::loadu(tmp_sum_data + d1);
258:               sum_vec += grad_output_vec * output_vec;
259:               sum_vec.store(tmp_sum_data + d1);
260:             }
261:             for (; d1 < size; d1++) {
262:               tmp_sum_data[d1] += grad_output_ptr[d1] * output_ptr[d1];
263:             }
264:           }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 266-294
```cpp
266:           // compute output * (grad_output - sum)
267:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
268:             int64_t offset = outer_idx * outer_stride + dim_idx * inner_size +
269:                 inner_idx_begin;
270:             const scalar_t* grad_output_ptr = grad_output_data_base + offset;
271:             const scalar_t* output_ptr = output_data_base + offset;
272:             scalar_t* grad_input_ptr = grad_input_data_base + offset;
273:
274:             int64_t d2 = 0;
275:             for (; d2 < size - (size % Vec::size()); d2 += Vec::size()) {
276:               Vec grad_output_vec = Vec::loadu(grad_output_ptr + d2);
277:               Vec output_vec = Vec::loadu(output_ptr + d2);
278:               Vec sum_vec = Vec::loadu(tmp_sum_data + d2);
279:               Vec grad_input_vec = output_vec * (grad_output_vec - sum_vec);
280:               grad_input_vec.store(grad_input_ptr + d2);
281:             }
282:             for (; d2 < size; d2++) {
283:               grad_input_ptr[d2] = output_ptr[d2] * (grad_output_ptr[d2] - tmp_sum_data[d2]);
284:             }
285:           }
286:         }
287:       });
288: }
289:
290: template<typename scalar_t>
291: inline typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
292: _vec_softmax_backward(
293:     scalar_t* grad_input_data_base,
294:     const scalar_t* grad_output_data_base,
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 295-326
```cpp
295:     const scalar_t* output_data_base,
296:     int64_t outer_size,
297:     int64_t inner_size,
298:     int64_t dim_size) {
299:   using Vec = vec::Vectorized<scalar_t>;
300:   using fVec = vec::Vectorized<float>;
301:   int64_t outer_stride = dim_size * inner_size;
302:   int64_t BLOCK_SIZE = 128 * 1024;
303:   int64_t MAX_CHUNK_SIZE = std::max<int64_t>(
304:       BLOCK_SIZE / dim_size / sizeof(scalar_t), Vec::size());
305:   MAX_CHUNK_SIZE = MAX_CHUNK_SIZE / Vec::size() * Vec::size();
306:   int64_t CHUNK_SIZE = std::min<int64_t>(MAX_CHUNK_SIZE, inner_size);
307:   int64_t num_chunks = divup(inner_size, CHUNK_SIZE);
308:   // See Note: grain_size value of 0
309:   parallel_for(
310:       0, outer_size * num_chunks, 0, [&](int64_t begin, int64_t end) {
311:         // thread local temp buffer that holds vertical sum result
312:         auto buffer = std::make_unique<float[]>(CHUNK_SIZE);
313:         float* tmp_sum_data = buffer.get();
314:
315:         // thread local buffer that holds grad_output and output data in float32
316:         auto grad_output_buffer = std::make_unique<float[]>(dim_size * CHUNK_SIZE);
317:         float* grad_output_buffer_data = grad_output_buffer.get();
318:
319:         auto output_buffer = std::make_unique<float[]>(dim_size * CHUNK_SIZE);
320:         float* output_buffer_data = output_buffer.get();
321:
322:         for (int64_t i = begin; i < end; i++) {
323:           int64_t outer_idx = i / num_chunks;
324:           int64_t k = i % num_chunks;
325:           int64_t inner_idx_begin = k * CHUNK_SIZE;
326:           int64_t size = std::min(CHUNK_SIZE, inner_size - inner_idx_begin);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 328-363
```cpp
328:           // init
329:           fVec zero_fvec = fVec(float(0));
330:           int64_t d0 = 0;
331:           for (; d0 < size - (size % Vec::size()); d0 += Vec::size()) {
332:             zero_fvec.store(tmp_sum_data + d0);
333:             zero_fvec.store(tmp_sum_data + d0 + fVec::size());
334:           }
335:           for (; d0 < size; d0++) {
336:             tmp_sum_data[d0] = float(0);
337:           }
338:
339:           // compute sum of grad_output * output
340:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
341:             int64_t offset = outer_idx * outer_stride + dim_idx * inner_size +
342:                 inner_idx_begin;
343:             const scalar_t* grad_output_ptr = grad_output_data_base + offset;
344:             const scalar_t* output_ptr = output_data_base + offset;
345:             float* grad_output_buffer_ptr =
346:                 grad_output_buffer_data + dim_idx * CHUNK_SIZE;
347:             float* output_buffer_ptr =
348:                 output_buffer_data + dim_idx * CHUNK_SIZE;
349:
350:             int64_t d1 = 0;
351:             for (; d1 < size - (size % Vec::size()); d1 += Vec::size()) {
352:               Vec grad_output_vec = Vec::loadu(grad_output_ptr + d1);
353:               auto [grad_output_fvec0, grad_output_fvec1] =
354:                   vec::convert_to_float<scalar_t>(grad_output_vec);
355:               Vec output_vec = Vec::loadu(output_ptr + d1);
356:               auto [output_fvec0, output_fvec1] =
357:                   vec::convert_to_float<scalar_t>(output_vec);
358:               fVec sum_fvec0 = fVec::loadu(tmp_sum_data + d1);
359:               fVec sum_fvec1 = fVec::loadu(tmp_sum_data + d1 + fVec::size());
360:               sum_fvec0 += grad_output_fvec0 * output_fvec0;
361:               sum_fvec1 += grad_output_fvec1 * output_fvec1;
362:               sum_fvec0.store(tmp_sum_data + d1);
363:               sum_fvec1.store(tmp_sum_data + d1 + fVec::size());
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 365-393
```cpp
365:               // cache the 'converted' float grad_output and output
366:               grad_output_fvec0.store(grad_output_buffer_ptr + d1);
367:               grad_output_fvec1.store(
368:                   grad_output_buffer_ptr + d1 + fVec::size());
369:               output_fvec0.store(output_buffer_ptr + d1);
370:               output_fvec1.store(output_buffer_ptr + d1 + fVec::size());
371:             }
372:             for (; d1 < size; d1++) {
373:               float grad_output_val = float(grad_output_ptr[d1]);
374:               float output_val = float(output_ptr[d1]);
375:               tmp_sum_data[d1] += grad_output_val * output_val;
376:               grad_output_buffer_ptr[d1] = grad_output_val;
377:               output_buffer_ptr[d1] = output_val;
378:             }
379:           }
380:
381:           // compute output * (grad_output - sum)
382:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
383:             scalar_t* grad_input_ptr = grad_input_data_base +
384:                 outer_idx * outer_stride + dim_idx * inner_size +
385:                 inner_idx_begin;
386:             float* grad_output_buffer_ptr =
387:                 grad_output_buffer_data + dim_idx * CHUNK_SIZE;
388:             float* output_buffer_ptr =
389:                 output_buffer_data + dim_idx * CHUNK_SIZE;
390:
391:             int64_t d2 = 0;
392:             for (; d2 < size - (size % Vec::size()); d2 += Vec::size()) {
393:               fVec sum_fvec0 = fVec::loadu(tmp_sum_data + d2);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 394-423
```cpp
394:               fVec sum_fvec1 = fVec::loadu(tmp_sum_data + d2 + fVec::size());
395:               fVec grad_output_fvec0 = fVec::loadu(grad_output_buffer_ptr + d2);
396:               fVec grad_output_fvec1 =
397:                   fVec::loadu(grad_output_buffer_ptr + d2 + fVec::size());
398:               fVec output_fvec0 = fVec::loadu(output_buffer_ptr + d2);
399:               fVec output_fvec1 =
400:                   fVec::loadu(output_buffer_ptr + d2 + fVec::size());
401:               fVec grad_input_fvec0 =
402:                   output_fvec0 * (grad_output_fvec0 - sum_fvec0);
403:               fVec grad_input_fvec1 =
404:                   output_fvec1 * (grad_output_fvec1 - sum_fvec1);
405:               Vec grad_input_vec =
406:                   vec::convert_from_float<scalar_t>(grad_input_fvec0, grad_input_fvec1);
407:               grad_input_vec.store(grad_input_ptr + d2);
408:             }
409:             for (; d2 < size; d2++) {
410:               grad_input_ptr[d2] = output_buffer_ptr[d2] * (grad_output_buffer_ptr[d2] - tmp_sum_data[d2]);
411:             }
412:           }
413:         }
414:       });
415: }
416:
417: template<typename scalar_t>
418: inline typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
419: _vec_log_softmax_backward(
420:     scalar_t* grad_input_data_base,
421:     const scalar_t* grad_output_data_base,
422:     const scalar_t* output_data_base,
423:     int64_t outer_size,
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 424-455
```cpp
424:     int64_t inner_size,
425:     int64_t dim_size) {
426:   using Vec = vec::Vectorized<scalar_t>;
427:   int64_t outer_stride = dim_size * inner_size;
428:   int64_t BLOCK_SIZE = 128 * 1024;
429:   int64_t MAX_CHUNK_SIZE = std::max<int64_t>(
430:       BLOCK_SIZE / dim_size / sizeof(scalar_t), Vec::size());
431:   MAX_CHUNK_SIZE = MAX_CHUNK_SIZE / Vec::size() * Vec::size();
432:   int64_t CHUNK_SIZE = std::min<int64_t>(MAX_CHUNK_SIZE, inner_size);
433:   int64_t num_chunks = divup(inner_size, CHUNK_SIZE);
434:   // See Note: grain_size value of 0
435:   parallel_for(
436:       0, outer_size * num_chunks, 0, [&](int64_t begin, int64_t end) {
437:         // thread local temp buffer that holds vertical sum result
438:         auto buffer = std::make_unique<scalar_t[]>(CHUNK_SIZE);
439:         scalar_t* tmp_sum_data = buffer.get();
440:
441:         for (int64_t i = begin; i < end; i++) {
442:           int64_t outer_idx = i / num_chunks;
443:           int64_t k = i % num_chunks;
444:           int64_t inner_idx_begin = k * CHUNK_SIZE;
445:           int64_t size = std::min(CHUNK_SIZE, inner_size - inner_idx_begin);
446:
447:           // init
448:           Vec zero_vec = Vec(scalar_t(0));
449:           int64_t d0 = 0;
450:           for (; d0 < size - (size % Vec::size()); d0 += Vec::size()) {
451:             zero_vec.store(tmp_sum_data + d0);
452:           }
453:           for (; d0 < size; d0++) {
454:             tmp_sum_data[d0] = scalar_t(0);
455:           }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 457-498
```cpp
457:           // compute sum of grad_output
458:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
459:             const scalar_t* grad_output_ptr = grad_output_data_base +
460:                 outer_idx * outer_stride + dim_idx * inner_size +
461:                 inner_idx_begin;
462:
463:             int64_t d1 = 0;
464:             for (; d1 < size - (size % Vec::size()); d1 += Vec::size()) {
465:               Vec grad_output_vec = Vec::loadu(grad_output_ptr + d1);
466:               Vec sum_vec = Vec::loadu(tmp_sum_data + d1);
467:               sum_vec += grad_output_vec;
468:               sum_vec.store(tmp_sum_data + d1);
469:             }
470:             for (; d1 < size; d1++) {
471:               tmp_sum_data[d1] += grad_output_ptr[d1];
472:             }
473:           }
474:
475:           // compute grad_output - output.exp() * sum
476:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
477:             int64_t offset = outer_idx * outer_stride + dim_idx * inner_size +
478:                 inner_idx_begin;
479:             const scalar_t* grad_output_ptr = grad_output_data_base + offset;
480:             const scalar_t* output_ptr = output_data_base + offset;
481:             scalar_t* grad_input_ptr = grad_input_data_base + offset;
482:
483:             int64_t d2 = 0;
484:             for (; d2 < size - (size % Vec::size()); d2 += Vec::size()) {
485:               Vec grad_output_vec = Vec::loadu(grad_output_ptr + d2);
486:               Vec output_vec = Vec::loadu(output_ptr + d2);
487:               Vec sum_vec = Vec::loadu(tmp_sum_data + d2);
488:               Vec grad_input_vec = grad_output_vec - output_vec.exp() * sum_vec;
489:               grad_input_vec.store(grad_input_ptr + d2);
490:             }
491:             for (; d2 < size; d2++) {
492:               grad_input_ptr[d2] = grad_output_ptr[d2] -
493:                   std::exp(output_ptr[d2]) * tmp_sum_data[d2];
494:             }
495:           }
496:         }
497:       });
498: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 500-533
```cpp
500: template<typename scalar_t>
501: inline typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
502: _vec_log_softmax_backward(
503:     scalar_t* grad_input_data_base,
504:     const scalar_t* grad_output_data_base,
505:     const scalar_t* output_data_base,
506:     int64_t outer_size,
507:     int64_t inner_size,
508:     int64_t dim_size) {
509:   using Vec = vec::Vectorized<scalar_t>;
510:   using fVec = vec::Vectorized<float>;
511:   int64_t outer_stride = dim_size * inner_size;
512:   int64_t BLOCK_SIZE = 128 * 1024;
513:   int64_t MAX_CHUNK_SIZE = std::max<int64_t>(
514:       BLOCK_SIZE / dim_size / sizeof(scalar_t), Vec::size());
515:   MAX_CHUNK_SIZE = MAX_CHUNK_SIZE / Vec::size() * Vec::size();
516:   int64_t CHUNK_SIZE = std::min<int64_t>(MAX_CHUNK_SIZE, inner_size);
517:   int64_t num_chunks = divup(inner_size, CHUNK_SIZE);
518:   // See Note: grain_size value of 0
519:   parallel_for(
520:       0, outer_size * num_chunks, 0, [&](int64_t begin, int64_t end) {
521:         // thread local temp buffer that holds vertical sum result
522:         auto buffer = std::make_unique<float[]>(CHUNK_SIZE);
523:         float* tmp_sum_data = buffer.get();
524:
525:         // thread local buffer that holds grad_output data in float32
526:         auto grad_output_buffer = std::make_unique<float[]>(dim_size * CHUNK_SIZE);
527:         float* grad_output_buffer_data = grad_output_buffer.get();
528:
529:         for (int64_t i = begin; i < end; i++) {
530:           int64_t outer_idx = i / num_chunks;
531:           int64_t k = i % num_chunks;
532:           int64_t inner_idx_begin = k * CHUNK_SIZE;
533:           int64_t size = std::min(CHUNK_SIZE, inner_size - inner_idx_begin);
```
- EN: The main symbol in this range is `_vec_log_softmax_backward`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `_vec_log_softmax_backward`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 535-564
```cpp
535:           // init
536:           fVec zero_fvec = fVec(float(0));
537:           int64_t d0 = 0;
538:           for (; d0 < size - (size % Vec::size()); d0 += Vec::size()) {
539:             zero_fvec.store(tmp_sum_data + d0);
540:             zero_fvec.store(tmp_sum_data + d0 + fVec::size());
541:           }
542:           for (; d0 < size; d0++) {
543:             tmp_sum_data[d0] = float(0);
544:           }
545:
546:           // compute sum of grad_output
547:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
548:             const scalar_t* grad_output_ptr = grad_output_data_base +
549:                 outer_idx * outer_stride + dim_idx * inner_size +
550:                 inner_idx_begin;
551:             float* grad_output_buffer_ptr =
552:                 grad_output_buffer_data + dim_idx * CHUNK_SIZE;
553:
554:             int64_t d1 = 0;
555:             for (; d1 < size - (size % Vec::size()); d1 += Vec::size()) {
556:               Vec grad_output_vec = Vec::loadu(grad_output_ptr + d1);
557:               auto [grad_output_fvec0, grad_output_fvec1] =
558:                   vec::convert_to_float<scalar_t>(grad_output_vec);
559:               fVec sum_fvec0 = fVec::loadu(tmp_sum_data + d1);
560:               fVec sum_fvec1 = fVec::loadu(tmp_sum_data + d1 + fVec::size());
561:               sum_fvec0 += grad_output_fvec0;
562:               sum_fvec1 += grad_output_fvec1;
563:               sum_fvec0.store(tmp_sum_data + d1);
564:               sum_fvec1.store(tmp_sum_data + d1 + fVec::size());
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 566-594
```cpp
566:               // cache the 'converted' float grad_output
567:               grad_output_fvec0.store(grad_output_buffer_ptr + d1);
568:               grad_output_fvec1.store(
569:                   grad_output_buffer_ptr + d1 + fVec::size());
570:             }
571:             for (; d1 < size; d1++) {
572:               float grad_output_val = float(grad_output_ptr[d1]);
573:               tmp_sum_data[d1] += grad_output_val;
574:               grad_output_buffer_ptr[d1] = grad_output_val;
575:             }
576:           }
577:
578:           // compute grad_output - output.exp() * sum
579:           for (int64_t dim_idx = 0; dim_idx < dim_size; dim_idx++) {
580:             int64_t offset = outer_idx * outer_stride + dim_idx * inner_size +
581:                 inner_idx_begin;
582:             const scalar_t* output_ptr = output_data_base + offset;
583:             scalar_t* grad_input_ptr = grad_input_data_base + offset;
584:             float* grad_output_buffer_ptr =
585:                 grad_output_buffer_data + dim_idx * CHUNK_SIZE;
586:
587:             int64_t d2 = 0;
588:             for (; d2 < size - (size % Vec::size()); d2 += Vec::size()) {
589:               Vec output_vec = Vec::loadu(output_ptr + d2);
590:               auto [output_fvec0, output_fvec1] =
591:                   vec::convert_to_float<scalar_t>(output_vec);
592:               fVec sum_fvec0 = fVec::loadu(tmp_sum_data + d2);
593:               fVec sum_fvec1 = fVec::loadu(tmp_sum_data + d2 + fVec::size());
594:               fVec grad_output_fvec0 = fVec::loadu(grad_output_buffer_ptr + d2);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 595-631
```cpp
595:               fVec grad_output_fvec1 =
596:                   fVec::loadu(grad_output_buffer_ptr + d2 + fVec::size());
597:               fVec grad_input_fvec0 =
598:                   grad_output_fvec0 - output_fvec0.exp() * sum_fvec0;
599:               fVec grad_input_fvec1 =
600:                   grad_output_fvec1 - output_fvec1.exp() * sum_fvec1;
601:               Vec grad_input_vec =
602:                   vec::convert_from_float<scalar_t>(grad_input_fvec0, grad_input_fvec1);
603:               grad_input_vec.store(grad_input_ptr + d2);
604:             }
605:             for (; d2 < size; d2++) {
606:               grad_input_ptr[d2] = grad_output_buffer_ptr[d2] -
607:                   std::exp(float(output_ptr[d2])) * tmp_sum_data[d2];
608:             }
609:           }
610:         }
611:       });
612: }
613:
614: template <typename scalar_t, bool LogSoftMax>
615: struct vec_host_softmax_lastdim {
616:   static void apply(const Tensor& output, const Tensor& input) {
617:     int64_t outer_size = 1;
618:     int64_t dim_size = input.size(input.ndimension() - 1);
619:     for (int64_t i = 0; i < input.ndimension() - 1; ++i)
620:       outer_size *= input.size(i);
621:     const scalar_t* input_data_base = input.const_data_ptr<scalar_t>();
622:     scalar_t* output_data_base = output.data_ptr<scalar_t>();
623:     if (LogSoftMax) {
624:       _vec_log_softmax_lastdim(
625:           input_data_base, output_data_base, outer_size, dim_size);
626:     } else {
627:       _vec_softmax_lastdim(
628:           input_data_base, output_data_base, outer_size, dim_size);
629:     }
630:   }
631: };
```
- EN: The main symbol in this range is `apply`, `vec_host_softmax_lastdim`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `apply`, `vec_host_softmax_lastdim`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 633-661
```cpp
633: template<typename scalar_t>
634: inline typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
635: _vec_softmax(
636:     const scalar_t* input_data_base,
637:     scalar_t* output_data_base,
638:     int64_t outer_size,
639:     int64_t inner_size,
640:     int64_t dim_size) {
641:   using Vec = vec::Vectorized<float>;
642:   using Vec16 = vec::Vectorized<scalar_t>;
643:   int64_t dim_stride = inner_size;
644:   int64_t outer_stride = dim_size * dim_stride;
645:   int vectorized_step = Vec16().size(); // Currently, we only support BFloat16/Half in this special implementation
646:   // See Note: grain_size value of 0
647:   parallel_for(
648:       0, outer_size * inner_size, 0, [&](int64_t begin, int64_t end) {
649:         int64_t idx = begin;
650:         std::vector<float> temp_vec_input(dim_size * vectorized_step);
651:         std::vector<float> temp_vec_output(dim_size * vectorized_step);
652:         float* temp_vec_input_data = temp_vec_input.data();
653:         float* temp_vec_output_data = temp_vec_output.data();
654:         while (idx < end) {
655:           int64_t outer_idx = idx / inner_size;
656:           int64_t inner_idx = idx % inner_size;
657:           if (((inner_idx + vectorized_step) <= inner_size) && ((idx + vectorized_step) <= end)) {
658:             // Vectorization
659:             const scalar_t* input_data =
660:                 input_data_base + outer_idx * outer_stride + inner_idx;
661:             scalar_t* output_data =
```
- EN: The main symbol in this range is `_vec_softmax`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `_vec_softmax`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 662-691
```cpp
662:                 output_data_base + outer_idx * outer_stride + inner_idx;
663:             // Step 1: Get max Score
664:             Vec16 max_vec_bf16 = Vec16::loadu(input_data);
665:             std::tuple<Vec, Vec> convert_result = vec::convert_to_float<scalar_t>(max_vec_bf16);
666:             Vec max_vec_o1 = std::get<0>(convert_result);
667:             Vec max_vec_o2 = std::get<1>(convert_result);
668:             std::get<0>(convert_result).store(temp_vec_input_data);
669:             std::get<1>(convert_result).store(temp_vec_input_data + Vec().size());
670:             for (const auto d : c10::irange(1, dim_size)) {
671:               Vec16 input_vec_bf16 = Vec16::loadu(input_data + d * dim_stride);
672:               convert_result = vec::convert_to_float<scalar_t>(input_vec_bf16);
673:               max_vec_o1 = vec::maximum(max_vec_o1, std::get<0>(convert_result));
674:               max_vec_o2 = vec::maximum(max_vec_o2, std::get<1>(convert_result));
675:               std::get<0>(convert_result).store(temp_vec_input_data + d*vectorized_step);
676:               std::get<1>(convert_result).store(temp_vec_input_data + d*vectorized_step + Vec().size());
677:             }
678:             // Step2: Calculate sum
679:             Vec sum_vec_o1 = Vec(0.0);
680:             Vec sum_vec_o2 = Vec(0.0);
681:             for (const auto d : c10::irange(dim_size)) {
682:               Vec output_vec_o1 = Vec::loadu(temp_vec_input_data + d*vectorized_step);
683:               Vec output_vec_o2 = Vec::loadu(temp_vec_input_data + d*vectorized_step + Vec().size());
684:               output_vec_o1 = (output_vec_o1 - max_vec_o1).exp();
685:               output_vec_o2 = (output_vec_o2 - max_vec_o2).exp();
686:               output_vec_o1.store(temp_vec_output_data + d*vectorized_step);
687:               output_vec_o2.store(temp_vec_output_data + d*vectorized_step + Vec().size());
688:
689:               sum_vec_o1 = sum_vec_o1 + output_vec_o1;
690:               sum_vec_o2 = sum_vec_o2 + output_vec_o2;
691:             }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 692-721
```cpp
692:             // Step3: Unify
693:             for (const auto d : c10::irange(dim_size)) {
694:               Vec output_vec_o1 = Vec::loadu(temp_vec_output_data + d*vectorized_step);
695:               Vec output_vec_o2 = Vec::loadu(temp_vec_output_data + d*vectorized_step + Vec().size());
696:               output_vec_o1 = output_vec_o1/sum_vec_o1;
697:               output_vec_o2 = output_vec_o2/sum_vec_o2;
698:               Vec16 output_vec_bf16 = vec::convert_from_float<scalar_t>(output_vec_o1, output_vec_o2);
699:               output_vec_bf16.store(output_data + d * dim_stride);
700:             }
701:             idx += vectorized_step;
702:           } else {
703:             // Tail case(Scalar): it is exactly same logic as host_softmax
704:             // inside aten/src/ATen/native/SoftMax.cpp. There are 2 kind of
705:             // cases which will fall through this part:
706:             // Case 1: For the idx at the end of total chunk for each thread, there are not enough numbers for parallelization.
707:             // Case 2: For the idx at the end of each inner_size inside thread, there are not enough numbers for parallelization.
708:             int64_t tail_number = ((idx+vectorized_step) > end) ? /*Case1*/ (end - idx) : /*Case2*/ (inner_size - inner_idx);
709:             for (const auto i : c10::irange(tail_number)) {
710:               outer_idx = (idx + i) / inner_size;
711:               inner_idx = (idx + i) % inner_size;
712:               const scalar_t* input_data =
713:                   input_data_base + outer_idx * outer_stride + inner_idx;
714:               scalar_t* output_data =
715:                   output_data_base + outer_idx * outer_stride + inner_idx;
716:               // Step1: Get max score
717:               float max_input = float(input_data[0]);
718:               for (const auto d : c10::irange(1, dim_size)) {
719:                 max_input = std::max(max_input, float(input_data[d * dim_stride]));
720:               }
721:               // Step2: Calculate the Sum
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 722-751
```cpp
722:               float sum_data = 0.0;
723:               float temp_output_data = 0.0;
724:               for (const auto d : c10::irange(dim_size)) {
725:                 temp_output_data = std::exp(input_data[d * dim_stride] - max_input);
726:                 sum_data += temp_output_data;
727:                 output_data[d * dim_stride] = scalar_t(temp_output_data);
728:               }
729:               // Step3: Unify
730:               for (const auto d : c10::irange(dim_size)) {
731:                 output_data[d * dim_stride] =
732:                     scalar_t(float(output_data[d * dim_stride])/sum_data);
733:               }
734:             }
735:             idx += tail_number;
736:           }
737:         }
738:       });
739: }
740:
741: template<typename scalar_t>
742: inline typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
743: _vec_softmax(
744:     const scalar_t* input_data_base,
745:     scalar_t* output_data_base,
746:     int64_t outer_size,
747:     int64_t inner_size,
748:     int64_t dim_size) {
749:   using Vec = vec::Vectorized<scalar_t>;
750:   int64_t dim_stride = inner_size;
751:   int64_t outer_stride = dim_size * dim_stride;
```
- EN: The main symbol in this range is `_vec_softmax`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `_vec_softmax`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 752-781
```cpp
752:   int vectorized_step = Vec().size();
753:   // See Note: grain_size value of 0
754:   parallel_for(
755:       0, outer_size * inner_size, 0, [&](int64_t begin, int64_t end) {
756:         int64_t idx = begin;
757:         while (idx < end) {
758:           int64_t outer_idx = idx / inner_size;
759:           int64_t inner_idx = idx % inner_size;
760:           if (((inner_idx + vectorized_step) <= inner_size) && ((idx + vectorized_step) <= end)) {
761:             // Vectorization
762:             const scalar_t* input_data =
763:                 input_data_base + outer_idx * outer_stride + inner_idx;
764:             scalar_t* output_data =
765:                 output_data_base + outer_idx * outer_stride + inner_idx;
766:             // Step 1: Get max Score
767:             Vec max_vec = Vec::loadu(input_data);
768:             for (const auto d : c10::irange(1, dim_size)) {
769:               Vec input_vec = Vec::loadu(input_data + d * dim_stride);
770:               max_vec = vec::maximum(max_vec, input_vec);
771:             }
772:             // Step2: Calculate sum
773:             Vec sum_vec = Vec(0.0);
774:             for (const auto d : c10::irange(dim_size)) {
775:               Vec output_vec =
776:                   (Vec::loadu(input_data + d * dim_stride) - max_vec).exp();
777:               output_vec.store(output_data + d * dim_stride);
778:               sum_vec = sum_vec + output_vec;
779:             }
780:             // Step3: Unify
781:             for (const auto d : c10::irange(dim_size)) {
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 782-823
```cpp
782:               Vec output_vec =
783:                   Vec::loadu(output_data + d * dim_stride) / sum_vec;
784:               output_vec.store(output_data + d * dim_stride);
785:             }
786:             idx += vectorized_step;
787:           } else {
788:             // Tail case(Scalar): it is exactly same logic as host_softmax
789:             // inside aten/src/ATen/native/SoftMax.cpp. There are 2 kind of
790:             // cases which will fall through this part:
791:             // Case 1: For the idx at the end of total chunk for each thread, there are not enough numbers for parallelization.
792:             // Case 2: For the idx at the end of each inner_size inside thread, there are not enough numbers for parallelization.
793:             int64_t tail_number = ((idx+vectorized_step) > end) ? /*Case1*/ (end - idx) : /*Case2*/ (inner_size - inner_idx);
794:             for (const auto i : c10::irange(tail_number)) {
795:               outer_idx = (idx + i) / inner_size;
796:               inner_idx = (idx + i) % inner_size;
797:               const scalar_t* input_data =
798:                   input_data_base + outer_idx * outer_stride + inner_idx;
799:               scalar_t* output_data =
800:                   output_data_base + outer_idx * outer_stride + inner_idx;
801:               // Step1: Get max score
802:               scalar_t max_input = input_data[0];
803:               for (const auto d : c10::irange(1, dim_size)) {
804:                 max_input = std::max(max_input, input_data[d * dim_stride]);
805:               }
806:               // Step2: Calculate the Sum
807:               scalar_t sum_data = 0;
808:               for (const auto d : c10::irange(dim_size)) {
809:                 output_data[d * dim_stride] =
810:                     std::exp(input_data[d * dim_stride] - max_input);
811:                 sum_data += output_data[d * dim_stride];
812:               }
813:               // Step3: Unify
814:               for (const auto d : c10::irange(dim_size)) {
815:                 output_data[d * dim_stride] =
816:                     output_data[d * dim_stride]/sum_data;
817:               }
818:             }
819:             idx += tail_number;
820:           }
821:         }
822:       });
823: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 825-861
```cpp
825: // NB: fast kernel for log_softmax when dim != -1
826: // input shape is normalized to {outer_size, dim_size, inner_size}
827: //
828: // The algorithm requires to load input tensor 3 times, to increase parallelism
829: // and cache hit rate, inner_size is blocked as:
830: //   inner_size: {CHUNK_SIZE, CHUNK_SIZE, ..., Remainder}
831: //
832: // Parallel on {outer_size, num_chunks} and do vertical reduction on each block of
833: // {dim_size, CHUNK_SIZE}, block size (128KB) selected to be L2 hit.
834: //
835: template<typename scalar_t>
836: inline typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
837: _vec_logsoftmax(
838:     const scalar_t* input_data_base,
839:     scalar_t* output_data_base,
840:     int64_t outer_size,
841:     int64_t inner_size,
842:     int64_t dim_size) {
843:   const auto [CHUNK_SIZE_binding, num_chunks_binding] = vec_logsoftmax_chunk_size_and_num_chunks<scalar_t>(
844:       inner_size, dim_size);
845:   // Work around "capturing a structured binding is not yet supported in OpenMP".
846:   const auto CHUNK_SIZE = CHUNK_SIZE_binding;
847:   const auto num_chunks = num_chunks_binding;
848:
849:   // See Note: grain_size value of 0
850:   at::parallel_for(0, outer_size * num_chunks, 0, [&](int64_t begin, int64_t end) {
851:     serial_vec_logsoftmax_range(
852:         input_data_base,
853:         output_data_base,
854:         inner_size,
855:         CHUNK_SIZE,
856:         num_chunks,
857:         dim_size,
858:         begin,
859:         end);
860:   });
861: }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Normalization-related state, scaling, or statistics are handled here.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里处理归一化相关的状态、缩放或统计量。

### Lines 863-891
```cpp
863: template<typename scalar_t>
864: inline typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
865: _vec_logsoftmax(
866:     const scalar_t* input_data_base,
867:     scalar_t* output_data_base,
868:     int64_t outer_size,
869:     int64_t inner_size,
870:     int64_t dim_size) {
871:   const auto [CHUNK_SIZE_binding, num_chunks_binding] = vec_logsoftmax_chunk_size_and_num_chunks<scalar_t>(
872:       inner_size, dim_size);
873:   // Work around "capturing a structured binding is not yet supported in OpenMP".
874:   const auto CHUNK_SIZE = CHUNK_SIZE_binding;
875:   const auto num_chunks = num_chunks_binding;
876:
877:   // See Note: grain_size value of 0
878:   at::parallel_for(0, outer_size * num_chunks, 0, [&](int64_t begin, int64_t end) {
879:     serial_vec_logsoftmax_range(
880:         input_data_base,
881:         output_data_base,
882:         inner_size,
883:         CHUNK_SIZE,
884:         num_chunks,
885:         dim_size,
886:         begin,
887:         end);
888:   });
889: }
890:
891: template <typename scalar_t, bool LogSoftMax>
```
- EN: The main symbol in this range is `_vec_logsoftmax`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 这一段的主要符号是 `_vec_logsoftmax`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 892-930
```cpp
892: struct vec_softmax {
893:   static void apply(const Tensor& output, const Tensor& input, int64_t dim) {
894:     int64_t outer_size = 1;
895:     int64_t dim_size = input.size(dim);
896:     int64_t inner_size = 1;
897:     for (const auto i : c10::irange(dim))outer_size *= input.size(i);
898:     for (int64_t i = dim + 1; i < input.dim(); ++i)
899:       inner_size *= input.size(i);
900:     const scalar_t* input_data_base = input.const_data_ptr<scalar_t>();
901:     scalar_t* output_data_base = output.data_ptr<scalar_t>();
902:     if (LogSoftMax) {
903:       _vec_logsoftmax(
904:           input_data_base, output_data_base, outer_size, inner_size, dim_size);
905:     } else {
906:       _vec_softmax(
907:           input_data_base, output_data_base, outer_size, inner_size, dim_size);
908:     }
909:   }
910: };
911:
912: template <typename scalar_t, bool LogSoftMax>
913: struct vec_host_softmax_backward_lastdim {
914:   static void
915:   apply(const Tensor& grad_input, const Tensor& grad, const Tensor& output) {
916:     int64_t outer_size = 1;
917:     int64_t dim_size = grad.size(grad.ndimension() - 1);
918:     for (int64_t i = 0; i < grad.ndimension() - 1; ++i)
919:       outer_size *= grad.size(i);
920:     scalar_t* grad_input_data_base = grad_input.mutable_data_ptr<scalar_t>();
921:     const scalar_t* grad_data_base = grad.const_data_ptr<scalar_t>();
922:     const scalar_t* output_data_base = output.const_data_ptr<scalar_t>();
923:     _vec_host_softmax_backward_lastdim<scalar_t, LogSoftMax>(
924:         grad_input_data_base,
925:         grad_data_base,
926:         output_data_base,
927:         outer_size,
928:         dim_size);
929:   }
930: };
```
- EN: The main symbol in this range is `apply`, `vec_softmax`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `apply`, `vec_softmax`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 932-969
```cpp
932: template <typename scalar_t, bool LogSoftMax>
933: struct vec_host_softmax_backward {
934:   static void apply(
935:       const Tensor& grad_input,
936:       const Tensor& grad,
937:       const Tensor& output,
938:       int64_t dim) {
939:     int64_t outer_size = 1;
940:     int64_t dim_size = grad.size(dim);
941:     int64_t inner_size = 1;
942:     for (const auto i : c10::irange(dim)) {
943:       outer_size *= grad.size(i);
944:     }
945:     for (int64_t i = dim + 1; i < grad.dim(); ++i) {
946:       inner_size *= grad.size(i);
947:     }
948:     scalar_t* grad_input_data_base = grad_input.mutable_data_ptr<scalar_t>();
949:     const scalar_t* grad_output_data_base = grad.const_data_ptr<scalar_t>();
950:     const scalar_t* output_data_base = output.const_data_ptr<scalar_t>();
951:     if (LogSoftMax) {
952:       _vec_log_softmax_backward<scalar_t>(
953:           grad_input_data_base,
954:           grad_output_data_base,
955:           output_data_base,
956:           outer_size,
957:           inner_size,
958:           dim_size);
959:     } else {
960:       _vec_softmax_backward<scalar_t>(
961:           grad_input_data_base,
962:           grad_output_data_base,
963:           output_data_base,
964:           outer_size,
965:           inner_size,
966:           dim_size);
967:     }
968:   }
969: };
```
- EN: The main symbol in this range is `apply`, `vec_host_softmax_backward`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `apply`, `vec_host_softmax_backward`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 971-999
```cpp
971: void softmax_lastdim_kernel_impl(
972:     const Tensor& result,
973:     const Tensor& self) {
974:   AT_DISPATCH_FLOATING_TYPES_AND2(
975:       at::ScalarType::BFloat16, at::ScalarType::Half, self.scalar_type(),
976:       "softmax_lastdim_kernel_impl",
977:       [&] { vec_host_softmax_lastdim<scalar_t, false>::apply(result, self); });
978: }
979:
980: void softmax_kernel_impl(const Tensor& result, const Tensor& self, int64_t dim) {
981:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::BFloat16, at::ScalarType::Half, self.scalar_type(),
982:     "softmax_kernel_impl",
983:     [&] { vec_softmax<scalar_t, false>::apply(result, self, dim); });
984: }
985:
986: void log_softmax_lastdim_kernel_impl(
987:     const Tensor& result,
988:     const Tensor& self) {
989:   AT_DISPATCH_FLOATING_TYPES_AND2(
990:       at::ScalarType::BFloat16, at::ScalarType::Half, self.scalar_type(),
991:       "log_softmax_lastdim_kernel_impl",
992:       [&] { vec_host_softmax_lastdim<scalar_t, true>::apply(result, self); });
993: }
994:
995: void log_softmax_kernel_impl(const Tensor& result, const Tensor& self, int64_t dim) {
996:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::BFloat16, at::ScalarType::Half, self.scalar_type(),
997:     "log_softmax_kernel_impl",
998:     [&] { vec_softmax<scalar_t, true>::apply(result, self, dim); });
999: }
```
- EN: The main symbol in this range is `softmax_lastdim_kernel_impl`, `softmax_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `softmax_lastdim_kernel_impl`, `softmax_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1001-1039
```cpp
1001: void softmax_backward_lastdim_kernel_impl(
1002:     const Tensor& grad_input,
1003:     const Tensor& grad,
1004:     const Tensor& output) {
1005:   AT_DISPATCH_FLOATING_TYPES_AND2(
1006:       at::ScalarType::BFloat16, at::ScalarType::Half, grad.scalar_type(),
1007:       "softmax_backward_lastdim_kernel_impl", [&] {
1008:         vec_host_softmax_backward_lastdim<scalar_t, false>::apply(
1009:             grad_input, grad, output);
1010:       });
1011: }
1012:
1013: void log_softmax_backward_lastdim_kernel_impl(
1014:     const Tensor& grad_input,
1015:     const Tensor& grad,
1016:     const Tensor& output) {
1017:   AT_DISPATCH_FLOATING_TYPES_AND2(
1018:       at::ScalarType::BFloat16, at::ScalarType::Half, grad.scalar_type(),
1019:       "log_softmax_backward_lastdim_kernel_impl", [&] {
1020:         vec_host_softmax_backward_lastdim<scalar_t, true>::apply(
1021:             grad_input, grad, output);
1022:       });
1023: }
1024:
1025: void softmax_backward_kernel_impl(
1026:     const Tensor& grad_input,
1027:     const Tensor& grad,
1028:     const Tensor& output,
1029:     int64_t dim) {
1030:   AT_DISPATCH_FLOATING_TYPES_AND2(
1031:       at::ScalarType::BFloat16,
1032:       at::ScalarType::Half,
1033:       grad.scalar_type(),
1034:       "softmax_backward_kernel_impl",
1035:       [&] {
1036:         vec_host_softmax_backward<scalar_t, false>::apply(
1037:             grad_input, grad, output, dim);
1038:       });
1039: }
```
- EN: The main symbol in this range is `softmax_backward_lastdim_kernel_impl`, `log_softmax_backward_lastdim_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `softmax_backward_lastdim_kernel_impl`, `log_softmax_backward_lastdim_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1041-1069
```cpp
1041: void log_softmax_backward_kernel_impl(
1042:     const Tensor& grad_input,
1043:     const Tensor& grad,
1044:     const Tensor& output,
1045:     int64_t dim) {
1046:   AT_DISPATCH_FLOATING_TYPES_AND2(
1047:       at::ScalarType::BFloat16,
1048:       at::ScalarType::Half,
1049:       grad.scalar_type(),
1050:       "log_softmax_backward_kernel_impl",
1051:       [&] {
1052:         vec_host_softmax_backward<scalar_t, true>::apply(
1053:             grad_input, grad, output, dim);
1054:       });
1055: }
1056:
1057: } // anonymous namespace
1058:
1059: ALSO_REGISTER_AVX512_DISPATCH(softmax_lastdim_kernel, &softmax_lastdim_kernel_impl)
1060: ALSO_REGISTER_AVX512_DISPATCH(log_softmax_lastdim_kernel, &log_softmax_lastdim_kernel_impl)
1061: ALSO_REGISTER_AVX512_DISPATCH(
1062:     softmax_backward_lastdim_kernel,
1063:     &softmax_backward_lastdim_kernel_impl)
1064: ALSO_REGISTER_AVX512_DISPATCH(
1065:     log_softmax_backward_lastdim_kernel,
1066:     &log_softmax_backward_lastdim_kernel_impl)
1067:
1068: ALSO_REGISTER_AVX512_DISPATCH(softmax_kernel, &softmax_kernel_impl)
1069: ALSO_REGISTER_AVX512_DISPATCH(log_softmax_kernel, &log_softmax_kernel_impl)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `log_softmax_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `log_softmax_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1070-1074
```cpp
1070: ALSO_REGISTER_AVX512_DISPATCH(softmax_backward_kernel, &softmax_backward_kernel_impl)
1071: ALSO_REGISTER_AVX512_DISPATCH(
1072:     log_softmax_backward_kernel,
1073:     &log_softmax_backward_kernel_impl)
1074: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/cpu/SoftmaxKernel.h`, `ATen/native/cpu/LogSoftmaxKernelImpl.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/TensorIterator.h`, `ATen/OpMathType.h`, `ATen/core/Tensor.h`, `ATen/cpu/vec/functional.h`, `ATen/cpu/vec/vec.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `memory`, `algorithm`, `iterator`, `numeric`, `vector`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Vectorized`, `parallel_for`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
