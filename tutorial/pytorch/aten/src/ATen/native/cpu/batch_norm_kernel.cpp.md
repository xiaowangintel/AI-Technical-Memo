# batch_norm_kernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/batch_norm_kernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU normalization kernels, statistics updates, and scaling rules in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 归一化 kernel、统计量更新与缩放规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-35
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/native/batch_norm.h>
 3:
 4: #include <ATen/core/Tensor.h>
 5: #include <ATen/AccumulateType.h>
 6: #include <ATen/Dispatch.h>
 7: #include <ATen/Parallel.h>
 8: #include <ATen/native/TensorIterator.h>
 9: #include <ATen/native/cpu/Loops.h>
10: #include <ATen/native/cpu/utils.h>
11: #include <ATen/native/cpu/mixed_data_type.h>
12: #include <ATen/cpu/vec/functional.h>
13: #include <ATen/cpu/vec/vec.h>
14: #include <c10/util/irange.h>
15: #include <ATen/OpMathType.h>
16:
17: #ifndef AT_PER_OPERATOR_HEADERS
18: #include <ATen/Functions.h>
19: #else
20: #include <ATen/ops/empty.h>
21: #include <ATen/ops/ones.h>
22: #include <ATen/ops/zeros.h>
23: #endif
24:
25: namespace at::native {
26: namespace {
27:
28: using namespace vec;
29:
30: template<typename param_t, typename opmath_t>
31: void batch_norm_cpu_collect_linear_and_constant_terms(
32:     opmath_t* alpha, opmath_t* beta, int64_t n_channel,
33:     const Tensor& weight /* optional */, const Tensor& bias /* optional */,
34:     const Tensor& save_mean, const Tensor& save_invstd,
35:     const Tensor& running_mean, const Tensor& running_var, bool train, double eps) {
```
- EN: This range pulls in required headers, including `ATen/native/batch_norm.h`, `ATen/core/Tensor.h`, `ATen/AccumulateType.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/native/batch_norm.h`, `ATen/core/Tensor.h`, `ATen/AccumulateType.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 37-70
```cpp
37:   const param_t* weight_data = weight.defined() ? weight.const_data_ptr<param_t>() : nullptr;
38:   const param_t* bias_data = bias.defined() ? bias.const_data_ptr<param_t>() : nullptr;
39:
40:   auto save_mean_a = conditional_accessor_1d<const param_t>(save_mean);
41:   auto save_invstd_a = conditional_accessor_1d<const param_t>(save_invstd);
42:   auto running_mean_a = conditional_accessor_1d<const param_t>(running_mean);
43:   auto running_var_a = conditional_accessor_1d<const param_t>(running_var);
44:
45:   /// Collect the linear and constant terms regarding the input.
46:   /// output(n, c, h, w)
47:   ///     = (input(n, c, h, w) - mean(c)) / sqrt(var(c) + eps) * weight(c)
48:   ///         + bias(c)
49:   ///     = input(n, c, h, w) * inv_var(c) * weight(c)
50:   ///         - mean(c) * inv_var(c) * weight(c) + bias(c),
51:   /// where inv_var(c) = 1 / sqrt(var(c) + eps).
52:   /// So the linear term, alpha(c) = inv_var(c) * weight(c),
53:   ///   the constant term beta(c) = bias(c) - mean(c) * inv_var(c) * weight(c)
54:   /// Note that this is only a good idea if (input_size >> c), in degenerate
55:   /// cases where image_size == 1 && batch_size == 1, it is slow.
56:   for (const auto c : c10::irange(n_channel)) {
57:     opmath_t mean, invstd;
58:     if (train) {
59:       mean = save_mean_a[c];
60:       invstd = save_invstd_a[c];
61:     } else {
62:       mean = running_mean_a[c];
63:       invstd = 1 / std::sqrt(running_var_a[c] + static_cast<opmath_t>(eps));
64:     }
65:     param_t weight_v = weight_data ? weight_data[c] : param_t(1);
66:     param_t bias_v = bias_data ? bias_data[c] : param_t(0);
67:     alpha[c] = invstd * weight_v;
68:     beta[c] = bias_v - mean * alpha[c];
69:   }
70: }
```
- EN: The main symbol in this range is `output`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `output`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 72-102
```cpp
 72: /// A fast path for CPU inference and training forward when all tensors are contiguous.
 73: template<typename scalar_t>
 74: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
 75: batch_norm_cpu_contiguous_impl(Tensor& output, const Tensor& input,
 76:     const Tensor& weight, const Tensor& bias, const Tensor& save_mean, const Tensor& save_invstd,
 77:     const Tensor& running_mean, const Tensor& running_var, bool train, double eps) {
 78:
 79:   using Vec = Vectorized<scalar_t>;
 80:   int64_t n_batch = input.size(0);
 81:   int64_t n_channel = input.size(1);
 82:   int64_t image_size = input.numel() / n_batch / n_channel;
 83:
 84:   Tensor alpha = at::empty({n_channel}, input.options());
 85:   Tensor beta = at::empty({n_channel}, input.options());
 86:   scalar_t* alpha_data = alpha.mutable_data_ptr<scalar_t>();
 87:   scalar_t* beta_data = beta.data_ptr<scalar_t>();
 88:
 89:   batch_norm_cpu_collect_linear_and_constant_terms<scalar_t, scalar_t>(
 90:      alpha_data, beta_data, n_channel, weight, bias,
 91:      save_mean, save_invstd, running_mean, running_var, train, eps);
 92:
 93:   scalar_t* output_data = output.data_ptr<scalar_t>();
 94:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
 95:
 96:   // Apply the linear terms to the input,
 97:   // output(n, c, h, w) = input(n, c, h, w) * alpha(c) + beta(c)
 98:   const int64_t loop_size = image_size - (image_size % Vec::size());
 99:   at::parallel_for(0, n_batch * n_channel, 1, [&](int64_t begin, int64_t end) {
100:     int64_t n = 0;
101:     int64_t c = 0;
102:     data_index_init(begin, n, n_batch, c, n_channel);
```
- EN: The main symbol in this range is `batch_norm_cpu_contiguous_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `batch_norm_cpu_contiguous_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 104-134
```cpp
104:     for (const auto i : c10::irange(begin, end)) {
105:       const Vec alpha_vec(alpha_data[c]);
106:       const Vec beta_vec(beta_data[c]);
107:       int64_t offset = i * image_size;
108:       int64_t d = 0;
109:       for (; d < loop_size; d += Vec::size()) {
110:         Vec data_vec = Vec::loadu(input_data + offset + d);
111:         Vec output_vec = data_vec * alpha_vec + beta_vec;
112:         output_vec.store(output_data + offset + d);
113:       }
114:       if (image_size - d > 0) {
115:         Vec data_vec = Vec::loadu(input_data + offset + d, image_size - d);
116:         Vec output_vec = data_vec * alpha_vec + beta_vec;
117:         output_vec.store(output_data + offset + d, image_size - d);
118:       }
119:       // move on to next index
120:       data_index_step(n, n_batch, c, n_channel);
121:     }
122:   });
123: }
124:
125: template <typename scalar_t>
126: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
127: batch_norm_cpu_channels_last_impl(Tensor& output, const Tensor& input,
128:     const Tensor& weight, const Tensor& bias, const Tensor& save_mean, const Tensor& save_invstd,
129:     const Tensor& running_mean, const Tensor& running_var, bool train, double eps) {
130:
131:   using Vec = Vectorized<scalar_t>;
132:   int64_t n_batch = input.size(0);
133:   int64_t n_channel = input.size(1);
134:   int64_t image_size = input.numel() / n_batch / n_channel;
```
- EN: The main symbol in this range is `batch_norm_cpu_channels_last_impl`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `batch_norm_cpu_channels_last_impl`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 136-173
```cpp
136:   Tensor alpha = at::empty({n_channel}, input.options());
137:   Tensor beta = at::empty({n_channel}, input.options());
138:   scalar_t* alpha_data = alpha.mutable_data_ptr<scalar_t>();
139:   scalar_t* beta_data = beta.data_ptr<scalar_t>();
140:
141:   batch_norm_cpu_collect_linear_and_constant_terms<scalar_t, scalar_t>(
142:       alpha_data, beta_data, n_channel, weight, bias,
143:       save_mean, save_invstd, running_mean, running_var, train, eps);
144:
145:   scalar_t* output_data = output.data_ptr<scalar_t>();
146:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
147:
148:   // Apply the linear terms to the input,
149:   // output(n, c, h, w) = input(n, c, h, w) * alpha(c) + beta(c)
150:   const int64_t loop_size = n_channel - (n_channel % Vec::size());
151:   at::parallel_for(0, n_batch * image_size, 1, [&](int64_t begin, int64_t end) {
152:     for (const auto i : c10::irange(begin, end)) {
153:       int64_t offset = i * n_channel;
154:       int64_t d = 0;
155:       // vectorize on channel dimension, for normal batch_norm input size,
156:       // alpha/beta should fit in L1 cache, otherwise consider blocking.
157:       for (; d < loop_size; d += Vec::size()) {
158:         Vec alpha_vec = Vec::loadu(alpha_data + d);
159:         Vec beta_vec = Vec::loadu(beta_data + d);
160:         Vec data_vec = Vec::loadu(input_data + offset + d);
161:         Vec output_vec = data_vec * alpha_vec + beta_vec;
162:         output_vec.store(output_data + offset + d);
163:       }
164:       if (n_channel - d > 0) {
165:         Vec alpha_vec = Vec::loadu(alpha_data + d, n_channel - d);
166:         Vec beta_vec = Vec::loadu(beta_data + d, n_channel - d);
167:         Vec data_vec = Vec::loadu(input_data + offset + d, n_channel - d);
168:         Vec output_vec = data_vec * alpha_vec + beta_vec;
169:         output_vec.store(output_data + offset + d, n_channel - d);
170:       }
171:     }
172:   });
173: }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 175-204
```cpp
175: template <typename scalar_t>
176: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
177: batch_norm_cpu_collect_stats_contiguous_impl(
178:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
179:
180:   // keep acc_type as opmath_type will use float type when scalar_t==float
181:   // while acc_type uses double for float.
182:   using accscalar_t = at::acc_type<scalar_t, false>;
183:   int64_t n_batch = input.size(0);
184:   int64_t n_channel = input.size(1);
185:   int64_t image_size = input.numel() / n_batch / n_channel;
186:   int64_t N = input.numel() / n_channel;
187:
188:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
189:   scalar_t* mean_data = mean.data_ptr<scalar_t>();
190:   scalar_t* var_sum_data = var_sum.data_ptr<scalar_t>();
191:
192:   // parallel dim reduce on 'channel'
193:   at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
194:     for (const auto c : c10::irange(begin, end)) {
195:       // compute mean per input
196:       accscalar_t sum = 0;
197:       for (const auto n : c10::irange(n_batch)) {
198:         for (const auto i : c10::irange(image_size)) {
199:           auto offset = n * n_channel * image_size + c * image_size + i;
200:           sum += input_data[offset];
201:         }
202:       }
203:       scalar_t mean = sum / N;
204:       mean_data[c] = mean;
```
- EN: The main symbol in this range is `batch_norm_cpu_collect_stats_contiguous_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `batch_norm_cpu_collect_stats_contiguous_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 206-234
```cpp
206:       // compute variance per input
207:       accscalar_t _var_sum = 0;
208:       for (const auto n : c10::irange(n_batch)) {
209:         for (const auto i : c10::irange(image_size)) {
210:           auto offset = n * n_channel * image_size + c * image_size + i;
211:           auto x = input_data[offset];
212:           _var_sum += (x - mean) * (x - mean);
213:         }
214:       }
215:       var_sum_data[c] = _var_sum;
216:     }
217:   });
218: }
219:
220: template <typename scalar_t>
221: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
222: batch_norm_cpu_collect_stats_channels_last_impl(
223:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
224:
225:   using Vec = Vectorized<scalar_t>;
226:   // keep acc_type as opmath_type will use float type when scalar_t==float
227:   // while acc_type uses double for float.
228:   using accscalar_t = at::acc_type<scalar_t, false>;
229:   int64_t n_channel = input.size(1);
230:   int64_t N = input.numel() / n_channel;
231:
232:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
233:   scalar_t* mean_data = mean.data_ptr<scalar_t>();
234:   scalar_t* var_sum_data = var_sum.data_ptr<scalar_t>();
```
- EN: The main symbol in this range is `batch_norm_cpu_collect_stats_channels_last_impl`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `batch_norm_cpu_collect_stats_channels_last_impl`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 236-267
```cpp
236:   // Typical vertical reduce from shape of {NHW, C} to {C}.
237:   // Apply two path parallel reduction when NHW > max_threads:
238:   // First path: allocate an immediate buffer of size {max_threads, C}, parallel along dim0,
239:   //    {NHW, C} => {max_threads, C}
240:   //
241:   // Second path: parallel along dim1 of the immediate buffer,
242:   //    {max_threads, C} => {C}
243:   //
244:   // Normal size of C should fit in L1, otherwise consider blocking on C.
245:   //
246:   int num_threads = at::get_num_threads();
247:
248:   if (N > num_threads) {
249:     Tensor buffer = at::zeros({num_threads, n_channel}, input.options());
250:     scalar_t* buffer_data = buffer.data_ptr<scalar_t>();
251:
252:     // compute mean per input
253:     at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
254:       int tid = at::get_thread_num();
255:       TORCH_CHECK(tid < num_threads,
256:                   "expect thread id smaller than ", num_threads, ", got thread id ", tid);
257:       scalar_t* buffer_ptr = buffer_data + tid * n_channel;
258:       for (const auto i : c10::irange(begin, end)) {
259:         const scalar_t* x_ptr = input_data + i * n_channel;
260:         vec::map2<scalar_t>(
261:             [](Vec x, Vec y) { return x + y; },
262:             buffer_ptr,
263:             x_ptr,
264:             buffer_ptr,
265:             n_channel);
266:       }
267:     });
```
- EN: The main symbol in this range is `map2<scalar_t>`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `map2<scalar_t>`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 269-296
```cpp
269:     at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
270:       for (const auto c : c10::irange(begin, end)) {
271:         accscalar_t sum = 0;
272:         for (const auto t : c10::irange(num_threads)) {
273:           sum += buffer_data[t * n_channel + c];
274:         }
275:         scalar_t mean = sum / N;
276:         mean_data[c] = mean;
277:       }
278:     });
279:
280:     // compute variance per input, reuse the immediate buffer
281:     buffer.zero_();
282:     at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
283:       int tid = at::get_thread_num();
284:       TORCH_CHECK(tid < num_threads, "expect thread id smaller than ", num_threads, ", got thread id ", tid);
285:       scalar_t* buffer_ptr = buffer_data + tid * n_channel;
286:       for (const auto i : c10::irange(begin, end)) {
287:         const scalar_t* x_ptr = input_data + i * n_channel;
288:         vec::map3<scalar_t>(
289:             [](Vec x, Vec y, Vec mean) { return y + (x - mean) * (x - mean); },
290:             buffer_ptr,
291:             x_ptr,
292:             buffer_ptr,
293:             mean_data,
294:             n_channel);
295:       }
296:     });
```
- EN: The main symbol in this range is `map3<scalar_t>`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `map3<scalar_t>`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 298-326
```cpp
298:     at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
299:       for (const auto c : c10::irange(begin, end)) {
300:         accscalar_t _var_sum = 0;
301:         for (const auto t : c10::irange(num_threads)) {
302:           _var_sum += buffer_data[t * n_channel + c];
303:         }
304:         var_sum_data[c] = _var_sum;
305:       }
306:     });
307:   } else {
308:     // Vertical reduce from shape of {NHW, C} to {C} when NHW <= max_threads.
309:     // We'll use two methods, Method 1 and Method 2.
310:     //
311:     // Method 1: when TILE_SIZE < C <= THRESHOLD, parallel on C
312:     //    {NHW, C} => {C}
313:     //
314:     // Method 2: when C <= TILE_SIZE or C > THRESHOLD, tile and vectorize on C, C is tiled as:
315:     //    C: {TILE_SIZE, TILE_SIZE, ..., Remainder}
316:     // parallel on tiles, vectorized vertical reduce on each tile
317:     //    {NHW, TILE_SIZE} => {TILE_SIZE}
318:     //
319:     // The optimal THRESHOLD to tile was found empirically.
320:     // When C > THRESHOLD, C is large enough that the benefit from tiling and vectorization outweigh the synchronization overhead.
321:     // When C <= TILE_SIZE, the problem size is small enough (C <= TILE_SIZE && NHW <= max_threads) that it's better to launch single thread with vectorization than C threads without vectorization.
322:     //
323:     // When num_threads == 1, always use Method 2 as there is no synchronization overhead.
324:     //
325:     int64_t TILE_SIZE = 16;
326:     int64_t THRESHOLD = 2048;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 328-356
```cpp
328:     // Method 2: parallel on tiles of C, vectorized vertical reduce on each tile
329:     if (num_threads == 1 || (n_channel <= TILE_SIZE || n_channel > THRESHOLD)) {
330:       // compute mean per input
331:       mean.zero_();
332:       at::parallel_for(0, (n_channel + TILE_SIZE - 1) / TILE_SIZE, 1, [&](int64_t tile_idx_begin, int64_t tile_idx_end) {
333:         for (int64_t tile_idx = tile_idx_begin; tile_idx < tile_idx_end; tile_idx++) {
334:           int64_t jj_begin = tile_idx * TILE_SIZE;
335:           int64_t jj_end = std::min(jj_begin + TILE_SIZE, n_channel);
336:           scalar_t* mean_ptr = mean_data + jj_begin;
337:           for (const auto i : c10::irange(N)) {
338:             const scalar_t* x_ptr = input_data + (i * n_channel + jj_begin);
339:             vec::map2<scalar_t>(
340:               [](Vec x, Vec y) { return x + y; },
341:               mean_ptr,
342:               x_ptr,
343:               mean_ptr,
344:               jj_end - jj_begin);
345:           }
346:           vec::map<scalar_t>(
347:             [N](Vec x) { return x / Vec(N); },
348:             mean_ptr,
349:             mean_ptr,
350:             jj_end - jj_begin);
351:         }
352:       });
353:
354:       // compute variance per input
355:       var_sum.zero_();
356:       at::parallel_for(0, (n_channel + TILE_SIZE - 1) / TILE_SIZE, 1, [&](int64_t tile_idx_begin, int64_t tile_idx_end) {
```
- EN: The main symbol in this range is `map2<scalar_t>`, `map<scalar_t>`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `map2<scalar_t>`, `map<scalar_t>`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 357-387
```cpp
357:         for (int64_t tile_idx = tile_idx_begin; tile_idx < tile_idx_end; tile_idx++) {
358:           int64_t jj_begin = tile_idx * TILE_SIZE;
359:           int64_t jj_end = std::min(jj_begin + TILE_SIZE, n_channel);
360:           scalar_t* var_sum_ptr = var_sum_data + jj_begin;
361:           scalar_t* mean_ptr = mean_data + jj_begin;
362:           for (const auto i : c10::irange(N)) {
363:             const scalar_t* x_ptr = input_data + (i * n_channel + jj_begin);
364:             vec::map3<scalar_t>(
365:               [](Vec x, Vec y, Vec mean) { return y + (x - mean) * (x - mean); },
366:               var_sum_ptr,
367:               x_ptr,
368:               var_sum_ptr,
369:               mean_ptr,
370:               jj_end - jj_begin);
371:           }
372:         }
373:       });
374:     }
375:     // Method 1: parallel on C, vertical reduce
376:     else {
377:       // compute mean per input
378:       at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
379:         for (const auto c : c10::irange(begin, end)) {
380:           accscalar_t sum = 0;
381:           for (const auto t : c10::irange(N)) {
382:             sum += input_data[t * n_channel + c];
383:           }
384:           scalar_t mean = sum / N;
385:           mean_data[c] = mean;
386:         }
387:       });
```
- EN: The main symbol in this range is `map3<scalar_t>`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `map3<scalar_t>`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 389-417
```cpp
389:       // compute variance per input
390:       at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
391:         for (const auto c : c10::irange(begin, end)) {
392:           accscalar_t _var_sum = 0;
393:           for (const auto t : c10::irange(N)) {
394:             _var_sum += (input_data[t * n_channel + c] - mean_data[c]) * (input_data[t * n_channel + c] - mean_data[c]);
395:           }
396:           var_sum_data[c] = _var_sum;
397:         }
398:       });
399:     }
400:   }
401: }
402:
403: template <typename scalar_t>
404: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
405: batch_norm_cpu_backward_contiguous_impl(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
406:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
407:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
408:     bool train, double eps) {
409:
410:   using Vec = Vectorized<scalar_t>;
411:   // keep acc_type as opmath_type will use float type when scalar_t==float
412:   // while acc_type uses double for float.
413:   using accscalar_t = at::acc_type<scalar_t, false>;
414:   int64_t n_batch = input.size(0);
415:   int64_t n_channel = input.size(1);
416:   int64_t image_size = input.numel() / n_batch / n_channel;
417:   int64_t N = input.numel() / n_channel;
```
- EN: The main symbol in this range is `batch_norm_cpu_backward_contiguous_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `batch_norm_cpu_backward_contiguous_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 419-447
```cpp
419:   const scalar_t* grad_output_data = grad_output.const_data_ptr<scalar_t>();
420:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
421:
422:   scalar_t* grad_input_data = grad_input.defined() ? grad_input.mutable_data_ptr<scalar_t>() : nullptr;
423:   scalar_t* grad_weight_data = grad_weight.defined() ? grad_weight.data_ptr<scalar_t>() : nullptr;
424:   scalar_t* grad_bias_data = grad_bias.defined() ? grad_bias.data_ptr<scalar_t>() : nullptr;
425:   const bool grad_input_null = grad_input_data == nullptr;
426:   const bool grad_weight_null = grad_weight_data == nullptr;
427:   const bool grad_bias_null = grad_bias_data == nullptr;
428:
429:   auto weight_a = conditional_accessor_1d<const scalar_t>(weight);
430:   auto save_mean_a = conditional_accessor_1d<const scalar_t>(save_mean);
431:   auto save_invstd_a = conditional_accessor_1d<const scalar_t>(save_invstd);
432:   auto running_mean_a = conditional_accessor_1d<const scalar_t>(running_mean);
433:   auto running_var_a = conditional_accessor_1d<const scalar_t>(running_var);
434:
435:   // parallel dim reduce on 'channel'
436:   at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
437:     for (const auto c : c10::irange(begin, end)) {
438:       scalar_t w = weight.defined() ? weight_a[c] : 1;
439:
440:       scalar_t mean, invstd;
441:       if (train) {
442:         mean = save_mean_a[c];
443:         invstd = save_invstd_a[c];
444:       } else {
445:         mean = running_mean_a[c];
446:         invstd = 1 / std::sqrt(running_var_a[c] + eps);
447:       }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 449-480
```cpp
449:       // reduce over grad_output in feature plane
450:       // compute 1) sum; 2) dot product of Q(X) and dY.
451:       // fuse into a single loop to reuse dY
452:       //
453:       accscalar_t sum = 0;
454:       accscalar_t dotp = 0;
455:       for (const auto n : c10::irange(n_batch)) {
456:         const scalar_t* x_ptr = input_data + n * n_channel * image_size + c * image_size;
457:         const scalar_t* dy_ptr = grad_output_data + n * n_channel * image_size + c * image_size;
458:
459:         sum += vec::reduce_all<scalar_t>(
460:             [](Vec& x, Vec& y) { return x + y; },
461:             dy_ptr,
462:             image_size);
463:
464:         dotp += vec::map2_reduce_all<scalar_t>(
465:             [mean](Vec x, Vec dy) { return (x - Vec(mean)) * dy; },
466:             [](Vec x, Vec y) { return x + y; },
467:             x_ptr,
468:             dy_ptr,
469:             image_size);
470:       }
471:
472:       if (!grad_input_null) {
473:         if (train) {
474:           scalar_t k = (scalar_t) dotp * invstd * invstd / N;
475:           scalar_t grad_mean = sum / N;
476:
477:           for (const auto n : c10::irange(n_batch)) {
478:             const scalar_t* x_ptr = input_data + n * n_channel * image_size + c * image_size;
479:             scalar_t* dx_ptr = grad_input_data + n * n_channel * image_size + c * image_size;
480:             const scalar_t* dy_ptr = grad_output_data + n * n_channel * image_size + c * image_size;
```
- EN: The main symbol in this range is `reduce_all<scalar_t>`, `map2_reduce_all<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `reduce_all<scalar_t>`, `map2_reduce_all<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 482-513
```cpp
482:             // Scalar math:
483:             // for (const auto j : c10::irange(image_size)) {
484:             //   scalar_t dx = (x_ptr[j] - mean) * k;
485:             //   dx_ptr[j] = (dy_ptr[j] - grad_mean - dx) * invstd * w;
486:             // }
487:             vec::map2<scalar_t>(
488:                 [=](Vec x, Vec dy) {
489:                   Vec dx = (x - Vec(mean)) * Vec(k);
490:                   return (dy - Vec(grad_mean) - dx) * Vec(invstd) * Vec(w);
491:                 },
492:                 dx_ptr,
493:                 x_ptr,
494:                 dy_ptr,
495:                 image_size);
496:           }
497:         } else { // evaluation mode
498:           for (const auto n : c10::irange(n_batch)) {
499:             scalar_t* dx_ptr = grad_input_data + n * n_channel * image_size + c * image_size;
500:             const scalar_t* dy_ptr = grad_output_data + n * n_channel * image_size + c * image_size;
501:
502:             // Scalar math:
503:             // for (const auto j : c10::irange(image_size)) {
504:             //   dx_ptr[j] = dy_ptr[j] * invstd * w;
505:             // }
506:             vec::map<scalar_t>(
507:                 [=](Vec dy) { return dy * Vec(invstd) * Vec(w); },
508:                 dx_ptr,
509:                 dy_ptr,
510:                 image_size);
511:           }
512:         }
513:       }
```
- EN: The main symbol in this range is `map2<scalar_t>`, `map<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `map2<scalar_t>`, `map<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 515-545
```cpp
515:       if (!grad_weight_null) {
516:         grad_weight_data[c] = dotp * invstd;
517:       }
518:
519:       if (!grad_bias_null) {
520:         grad_bias_data[c] = sum;
521:       }
522:     }
523:   });
524: }
525:
526: template <typename scalar_t>
527: typename std::enable_if_t<std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
528: batch_norm_cpu_backward_channels_last_impl(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
529:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
530:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
531:     bool train, double eps) {
532:
533:   using Vec = Vectorized<scalar_t>;
534:   // keep acc_type as opmath_type will use float type when scalar_t==float
535:   // while acc_type uses double for float.
536:   using accscalar_t = at::acc_type<scalar_t, false>;
537:   int64_t n_channel = input.size(1);
538:   int64_t N = input.numel() / n_channel;
539:
540:   const scalar_t* grad_output_data = grad_output.const_data_ptr<scalar_t>();
541:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
542:
543:   scalar_t* grad_input_data = grad_input.defined() ? grad_input.mutable_data_ptr<scalar_t>() : nullptr;
544:   scalar_t* grad_weight_data = grad_weight.defined() ? grad_weight.data_ptr<scalar_t>() : nullptr;
545:   scalar_t* grad_bias_data = grad_bias.defined() ? grad_bias.data_ptr<scalar_t>() : nullptr;
```
- EN: The main symbol in this range is `batch_norm_cpu_backward_channels_last_impl`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `batch_norm_cpu_backward_channels_last_impl`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 547-581
```cpp
547:   const scalar_t* save_mean_data = conditional_data_ptr<const scalar_t>(save_mean);
548:   scalar_t* save_invstd_data = conditional_data_ptr<scalar_t>(save_invstd);
549:   const scalar_t* running_mean_data = conditional_data_ptr<const scalar_t>(running_mean);
550:   const scalar_t* running_var_data = conditional_data_ptr<const scalar_t>(running_var);
551:
552:   Tensor weight_ = weight.defined() ? weight : at::ones({n_channel}, input.options());
553:   const scalar_t* weight_data = weight_.const_data_ptr<scalar_t>();
554:
555:   const scalar_t* mean_ptr = nullptr;
556:   scalar_t* invstd_ptr = nullptr;
557:   Tensor invstd = at::empty({0}, input.options());
558:   if (train) {
559:     mean_ptr = save_mean_data;
560:     invstd_ptr = save_invstd_data;
561:   } else {
562:     mean_ptr = running_mean_data;
563:
564:     invstd.resize_({n_channel});
565:     invstd_ptr = invstd.data_ptr<scalar_t>();
566:     for (const auto c : c10::irange(n_channel)) {
567:       invstd_ptr[c] = 1 / std::sqrt(running_var_data[c] + eps);
568:     }
569:   }
570:
571:   // Typical vertical reduce from shape of {NHW, C} to {C}.
572:   // Apply two path parallel reduction:
573:   // First path: allocate an immediate buffer of size {2, max_threads, C}, parallel along dim0,
574:   //    sum = buffer[0], dotp = buffer[2]
575:   //
576:   // Second path: parallel along dim1 of the immediate buffer.
577:   //
578:   int num_threads = at::get_num_threads();
579:   Tensor buffer = at::zeros({2, num_threads, n_channel}, input.options());
580:   scalar_t* sum_data = buffer.data_ptr<scalar_t>();
581:   scalar_t* dotp_data = sum_data + num_threads * n_channel;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 583-620
```cpp
583:   // compute sum and dotp per feature plain,
584:   // fuse into a single loop to reuse grad_output in L1.
585:   at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
586:     int tid = at::get_thread_num();
587:     TORCH_CHECK(tid < num_threads, "expect thread id smaller than ", num_threads, ", got thread id ", tid);
588:     scalar_t* sum_ptr = sum_data + tid * n_channel;
589:     scalar_t* dotp_ptr = dotp_data + tid * n_channel;
590:     for (const auto i : c10::irange(begin, end)) {
591:       const scalar_t* x_ptr = input_data + i * n_channel;
592:       const scalar_t* dy_ptr = grad_output_data + i * n_channel;
593:
594:       vec::map2<scalar_t>(
595:           [](Vec sum, Vec dy) { return sum + dy; },
596:           sum_ptr,
597:           sum_ptr,
598:           dy_ptr,
599:           n_channel);
600:
601:       vec::map4<scalar_t>(
602:           [](Vec dotp, Vec x, Vec mean, Vec dy) { return dotp + (x - mean) * dy; },
603:           dotp_ptr,
604:           dotp_ptr,
605:           x_ptr,
606:           mean_ptr,
607:           dy_ptr,
608:           n_channel);
609:     }
610:   });
611:
612:   at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
613:     for (const auto c : c10::irange(begin, end)) {
614:       // store the final result of sum and dotp in the 1st lane of immediate buffer,
615:       // so that we won't need to allocate anther buffer to store the temp values.
616:       accscalar_t _sum = 0;
617:       for (const auto t : c10::irange(num_threads)) {
618:         _sum += sum_data[t * n_channel + c];
619:       }
620:       sum_data[/* 0 * n_channel + */c] = _sum;
```
- EN: The main symbol in this range is `map2<scalar_t>`, `map4<scalar_t>`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `map2<scalar_t>`, `map4<scalar_t>`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 622-650
```cpp
622:       accscalar_t _dotp = 0;
623:       for (const auto t : c10::irange(num_threads)) {
624:         _dotp += dotp_data[t * n_channel + c];
625:       }
626:       dotp_data[/* 0 * n_channel + */c] = _dotp;
627:     }
628:   });
629:
630:   // compute grad_input
631:   const int64_t loop_size = n_channel - (n_channel % Vec::size());
632:   if (grad_input.defined()) {
633:     at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
634:       for (const auto i : c10::irange(begin, end)) {
635:         scalar_t* dx_ptr = grad_input_data + i * n_channel;
636:         const scalar_t* x_ptr = input_data + i * n_channel;
637:         const scalar_t* dy_ptr = grad_output_data + i * n_channel;
638:         if (train) {
639:           int64_t d = 0;
640:           for (; d < loop_size; d += Vec::size()) {
641:             Vec x = Vec::loadu(x_ptr + d);
642:             Vec mean = Vec::loadu(mean_ptr + d);
643:             Vec dotp = Vec::loadu(dotp_data + d);
644:             Vec invstd = Vec::loadu(invstd_ptr + d);
645:             Vec k = dotp * invstd * invstd / Vec(N);
646:             Vec dx = (x - mean) * k;
647:             Vec dy = Vec::loadu(dy_ptr + d);
648:             Vec grad_mean = Vec::loadu(sum_data + d) / Vec(N);
649:             Vec w = Vec::loadu(weight_data + d);
650:             dx = (dy - grad_mean - dx) * invstd * w;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 651-685
```cpp
651:             dx.store(dx_ptr + d);
652:           }
653:           if (n_channel - d > 0) {
654:             Vec x = Vec::loadu(x_ptr + d, n_channel - d);
655:             Vec mean = Vec::loadu(mean_ptr + d, n_channel - d);
656:             Vec dotp = Vec::loadu(dotp_data + d, n_channel - d);
657:             Vec invstd = Vec::loadu(invstd_ptr + d, n_channel - d);
658:             Vec k = dotp * invstd * invstd / Vec(N);
659:             Vec dx = (x - mean) * k;
660:             Vec dy = Vec::loadu(dy_ptr + d, n_channel - d);
661:             Vec grad_mean = Vec::loadu(sum_data + d, n_channel - d) / Vec(N);
662:             Vec w = Vec::loadu(weight_data + d, n_channel - d);
663:             dx = (dy - grad_mean - dx) * invstd * w;
664:             dx.store(dx_ptr + d, n_channel - d);
665:           }
666:         } else { // evaluation mode
667:           int64_t d = 0;
668:           for (; d < loop_size; d += Vec::size()) {
669:             Vec dy = Vec::loadu(dy_ptr + d);
670:             Vec invstd = Vec::loadu(invstd_ptr + d);
671:             Vec w = Vec::loadu(weight_data + d);
672:             Vec dx = dy * invstd * w;
673:             dx.store(dx_ptr + d);
674:           }
675:           if (n_channel - d > 0) {
676:             Vec dy = Vec::loadu(dy_ptr + d, n_channel - d);
677:             Vec invstd = Vec::loadu(invstd_ptr + d, n_channel - d);
678:             Vec w = Vec::loadu(weight_data + d, n_channel - d);
679:             Vec dx = dy * invstd * w;
680:             dx.store(dx_ptr + d, n_channel - d);
681:           }
682:         }
683:       }
684:     });
685:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 687-718
```cpp
687:   if (grad_weight.defined()) {
688:     // grad_weight = dotp * invstd
689:     vec::map2<scalar_t>(
690:         [](Vec dotp, Vec invstd) { return dotp * invstd; },
691:         grad_weight_data,
692:         dotp_data,
693:         invstd_ptr,
694:         n_channel);
695:   }
696:
697:   // grad_bias = sum
698:   if (grad_bias.defined()) {
699:     vec::map<scalar_t>(
700:         [](Vec sum) { return sum; },
701:         grad_bias_data,
702:         sum_data,
703:         n_channel);
704:   }
705: }
706:
707: /// bfloat16/Half kernels
708: template<typename scalar_t>
709: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
710: batch_norm_cpu_contiguous_impl(Tensor& output, const Tensor& input,
711:     const Tensor& weight, const Tensor& bias, const Tensor& save_mean, const Tensor& save_invstd,
712:     const Tensor& running_mean, const Tensor& running_var, bool train, double eps) {
713:   using opmath_t = at::opmath_type<scalar_t>;
714:   using bVec = Vectorized<scalar_t>;
715:   using fVec = Vectorized<opmath_t>;
716:   int64_t n_batch = input.size(0);
717:   int64_t n_channel = input.size(1);
718:   int64_t image_size = input.numel() / n_batch / n_channel;
```
- EN: The main symbol in this range is `map2<scalar_t>`, `map<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `map2<scalar_t>`, `map<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 720-756
```cpp
720:   // use float as acc type
721:   Tensor alpha = at::empty({n_channel}, input.options().dtype(kFloat));
722:   Tensor beta = at::empty({n_channel}, input.options().dtype(kFloat));
723:   opmath_t* alpha_data = alpha.mutable_data_ptr<opmath_t>();
724:   opmath_t* beta_data = beta.data_ptr<opmath_t>();
725:
726:   const bool mixed_type = is_mixed_type(input, weight, bias, save_mean, save_invstd, running_mean, running_var);
727:   if (mixed_type) {
728:     batch_norm_cpu_collect_linear_and_constant_terms<opmath_t, opmath_t>(
729:         alpha_data, beta_data, n_channel, weight, bias,
730:         save_mean, save_invstd, running_mean, running_var, train, eps);
731:   } else {
732:     batch_norm_cpu_collect_linear_and_constant_terms<scalar_t, opmath_t>(
733:         alpha_data, beta_data, n_channel, weight, bias,
734:         save_mean, save_invstd, running_mean, running_var, train, eps);
735:   }
736:
737:   scalar_t* output_data = output.data_ptr<scalar_t>();
738:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
739:
740:   const int64_t loop_size = image_size - (image_size % bVec::size());
741:   at::parallel_for(0, n_batch * n_channel, 1, [&](int64_t begin, int64_t end) {
742:     int64_t n = 0;
743:     int64_t c = 0;
744:     data_index_init(begin, n, n_batch, c, n_channel);
745:
746:     for (const auto i : c10::irange(begin, end)) {
747:       const scalar_t* input_ptr = input_data + i * image_size;
748:       scalar_t* output_ptr = output_data + i * image_size;
749:       const opmath_t alpha_val = alpha_data[c];
750:       const opmath_t beta_val = beta_data[c];
751:       const fVec alpha_fvec(alpha_val);
752:       const fVec beta_fvec(beta_val);
753:       int64_t d = 0;
754:       for (; d < loop_size; d += bVec::size()) {
755:         bVec data_bvec = bVec::loadu(input_ptr + d);
756:         auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 758-787
```cpp
758:         fVec out_fvec0 = data_fvec0 * alpha_fvec + beta_fvec;
759:         fVec out_fvec1 = data_fvec1 * alpha_fvec + beta_fvec;
760:         bVec out_bvec = convert_from_float<scalar_t>(out_fvec0, out_fvec1);
761:         out_bvec.store(output_ptr + d);
762:       }
763:       for (; d < image_size; d++) {
764:         output_ptr[d] = scalar_t(opmath_t(input_ptr[d]) * alpha_val + beta_val);
765:       }
766:       // move on to next index
767:       data_index_step(n, n_batch, c, n_channel);
768:     }
769:   });
770: }
771:
772: template <typename scalar_t>
773: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
774: batch_norm_cpu_channels_last_impl(Tensor& output, const Tensor& input,
775:     const Tensor& weight, const Tensor& bias, const Tensor& save_mean, const Tensor& save_invstd,
776:     const Tensor& running_mean, const Tensor& running_var, bool train, double eps) {
777:   using opmath_t = at::opmath_type<scalar_t>;
778:   using bVec = Vectorized<scalar_t>;
779:   using fVec = Vectorized<opmath_t>;
780:   int64_t n_batch = input.size(0);
781:   int64_t n_channel = input.size(1);
782:   int64_t image_size = input.numel() / n_batch / n_channel;
783:
784:   Tensor alpha = at::empty({n_channel}, input.options().dtype(kFloat));
785:   Tensor beta = at::empty({n_channel}, input.options().dtype(kFloat));
786:   opmath_t* alpha_data = alpha.mutable_data_ptr<opmath_t>();
787:   opmath_t* beta_data = beta.data_ptr<opmath_t>();
```
- EN: The main symbol in this range is `batch_norm_cpu_channels_last_impl`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `batch_norm_cpu_channels_last_impl`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 789-827
```cpp
789:   const bool mixed_type = is_mixed_type(input, weight, bias, save_mean, save_invstd, running_mean, running_var);
790:   if (mixed_type) {
791:     batch_norm_cpu_collect_linear_and_constant_terms<opmath_t, opmath_t>(
792:         alpha_data, beta_data, n_channel, weight, bias,
793:         save_mean, save_invstd, running_mean, running_var, train, eps);
794:   } else {
795:     batch_norm_cpu_collect_linear_and_constant_terms<scalar_t, opmath_t>(
796:         alpha_data, beta_data, n_channel, weight, bias,
797:         save_mean, save_invstd, running_mean, running_var, train, eps);
798:   }
799:
800:   scalar_t* output_data = output.data_ptr<scalar_t>();
801:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
802:
803:   const int64_t loop_size = n_channel - (n_channel % bVec::size());
804:   at::parallel_for(0, n_batch * image_size, 1, [&](int64_t begin, int64_t end) {
805:     for (const auto i : c10::irange(begin, end)) {
806:       const scalar_t* input_ptr = input_data + i * n_channel;
807:       scalar_t* output_ptr = output_data + i * n_channel;
808:       int64_t d = 0;
809:       for (; d < loop_size; d += bVec::size()) {
810:         fVec alpha_fvec0 = fVec::loadu(alpha_data + d);
811:         fVec alpha_fvec1 = fVec::loadu(alpha_data + d + fVec::size());
812:         fVec beta_fvec0 = fVec::loadu(beta_data + d);
813:         fVec beta_fvec1 = fVec::loadu(beta_data + d + fVec::size());
814:         bVec data_bvec = bVec::loadu(input_ptr + d);
815:         auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
816:
817:         fVec out_fvec0 = data_fvec0 * alpha_fvec0 + beta_fvec0;
818:         fVec out_fvec1 = data_fvec1 * alpha_fvec1 + beta_fvec1;
819:         bVec out_bvec = convert_from_float<scalar_t>(out_fvec0, out_fvec1);
820:         out_bvec.store(output_ptr + d);
821:       }
822:       for (; d < n_channel; d++) {
823:         output_ptr[d] = scalar_t(opmath_t(input_ptr[d]) * alpha_data[d] + beta_data[d]);
824:       }
825:     }
826:   });
827: }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 829-864
```cpp
829: template <typename scalar_t, typename param_t>
830: inline void batch_norm_cpu_collect_stats_contiguous_internal(
831:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
832:   using opmath_t = at::opmath_type<scalar_t>;
833:   using bVec = Vectorized<scalar_t>;
834:   using fVec = Vectorized<opmath_t>;
835:   int64_t n_batch = input.size(0);
836:   int64_t n_channel = input.size(1);
837:   int64_t image_size = input.numel() / n_batch / n_channel;
838:   int64_t N = input.numel() / n_channel;
839:
840:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
841:   param_t* mean_data = mean.data_ptr<param_t>();
842:   param_t* var_sum_data = var_sum.data_ptr<param_t>();
843:
844:   at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
845:     for (const auto c : c10::irange(begin, end)) {
846:       opmath_t sum_val = opmath_t(0);
847:       fVec sum_fvec = fVec(opmath_t(0));
848:       for (int64_t n = 0; n < n_batch; n++) {
849:         const scalar_t* input_ptr = input_data + n * n_channel * image_size + c * image_size;
850:         int64_t d = 0;
851:         for (; d < image_size - (image_size % bVec::size()); d += bVec::size()) {
852:           bVec data_bvec = bVec::loadu(input_ptr + d);
853:           auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
854:           sum_fvec += data_fvec0;
855:           sum_fvec += data_fvec1;
856:         }
857:         for (; d < image_size; d++) {
858:           sum_val += opmath_t(input_ptr[d]);
859:         }
860:       }
861:       // TODO: use fast version
862:       sum_val += vec_reduce_all([](fVec& x, fVec& y) { return x + y; }, sum_fvec, fVec::size());
863:       opmath_t mean_val = sum_val / N;
864:       mean_data[c] = param_t(mean_val);
```
- EN: The main symbol in this range is `batch_norm_cpu_collect_stats_contiguous_internal`, `vec_reduce_all`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `batch_norm_cpu_collect_stats_contiguous_internal`, `vec_reduce_all`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 866-900
```cpp
866:       opmath_t var_val = opmath_t(0);
867:       fVec var_fvec = fVec(opmath_t(0));
868:       fVec mean_fvec = fVec(mean_val);
869:       for (int64_t n = 0; n < n_batch; n++) {
870:         const scalar_t* input_ptr = input_data + n * n_channel * image_size + c * image_size;
871:         int64_t d = 0;
872:         for (; d < image_size - (image_size % bVec::size()); d += bVec::size()) {
873:           bVec data_bvec = bVec::loadu(input_ptr + d);
874:           auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
875:           var_fvec += (data_fvec0 - mean_fvec) * (data_fvec0 - mean_fvec);
876:           var_fvec += (data_fvec1 - mean_fvec) * (data_fvec1 - mean_fvec);
877:         }
878:         for (; d < image_size; d++) {
879:           opmath_t data_val = input_ptr[d];
880:           var_val += (data_val - mean_val) * (data_val - mean_val);
881:         }
882:       }
883:       // TODO: use fast version
884:       var_val += vec_reduce_all([](fVec& x, fVec& y) { return x + y; }, var_fvec, fVec::size());
885:       var_sum_data[c] = param_t(var_val);
886:     }
887:   });
888: }
889:
890: template <typename scalar_t>
891: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
892: batch_norm_cpu_collect_stats_contiguous_impl(
893:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
894:   const bool mixed_type = is_mixed_type(input, mean, var_sum);
895:   if (mixed_type) {
896:     batch_norm_cpu_collect_stats_contiguous_internal<scalar_t, at::opmath_type<scalar_t>>(mean, var_sum, input);
897:   } else {
898:     batch_norm_cpu_collect_stats_contiguous_internal<scalar_t, scalar_t>(mean, var_sum, input);
899:   }
900: }
```
- EN: The main symbol in this range is `vec_reduce_all`, `batch_norm_cpu_collect_stats_contiguous_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `vec_reduce_all`, `batch_norm_cpu_collect_stats_contiguous_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 902-938
```cpp
902: template <typename scalar_t, typename param_t>
903: inline void batch_norm_cpu_collect_stats_channels_last_internal(
904:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
905:   using opmath_t = at::opmath_type<scalar_t>;
906:   using bVec = Vectorized<scalar_t>;
907:   using fVec = Vectorized<opmath_t>;
908:   int64_t n_channel = input.size(1);
909:   int64_t N = input.numel() / n_channel;
910:
911:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
912:   param_t* mean_data = mean.data_ptr<param_t>();
913:   param_t* var_sum_data = var_sum.data_ptr<param_t>();
914:
915:   int num_threads = at::get_num_threads();
916:   Tensor buffer = at::zeros({num_threads, n_channel}, input.options().dtype(kFloat));
917:   opmath_t* buffer_data = buffer.data_ptr<opmath_t>();
918:
919:   at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
920:     int tid = at::get_thread_num();
921:     TORCH_CHECK(tid < num_threads, "expect thread id smaller than ", num_threads, ", got thread id ", tid);
922:     opmath_t* buffer_ptr = buffer_data + tid * n_channel;
923:     for (const auto i : c10::irange(begin, end)) {
924:       const scalar_t* input_ptr = input_data + i * n_channel;
925:       int64_t d = 0;
926:       for (; d < n_channel - (n_channel % bVec::size()); d += bVec::size()) {
927:         bVec data_bvec = bVec::loadu(input_ptr + d);
928:         auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
929:         fVec sum_fvec0 = fVec::loadu(buffer_ptr + d) + data_fvec0;
930:         fVec sum_fvec1 = fVec::loadu(buffer_ptr + d + fVec::size()) + data_fvec1;
931:         sum_fvec0.store(buffer_ptr + d);
932:         sum_fvec1.store(buffer_ptr + d + fVec::size());
933:       }
934:       for (; d < n_channel; d++) {
935:         buffer_ptr[d] += input_ptr[d];
936:       }
937:     }
938:   });
```
- EN: The main symbol in this range is `batch_norm_cpu_collect_stats_channels_last_internal`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `batch_norm_cpu_collect_stats_channels_last_internal`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 940-973
```cpp
940:   for (const auto c : c10::irange(n_channel)) {
941:     opmath_t sum = 0;
942:     for (const auto t : c10::irange(num_threads)) {
943:       sum += buffer_data[t * n_channel + c];
944:     }
945:     mean_data[c] = param_t(sum / N);
946:   }
947:
948:   buffer.zero_();
949:   at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
950:     int tid = at::get_thread_num();
951:     TORCH_CHECK(tid < num_threads, "expect thread id smaller than ", num_threads, ", got thread id ", tid);
952:     opmath_t* buffer_ptr = buffer_data + tid * n_channel;
953:     for (const auto i : c10::irange(begin, end)) {
954:       const scalar_t* input_ptr = input_data + i * n_channel;
955:       int64_t d = 0;
956:       for (; d < n_channel - (n_channel % bVec::size()); d += bVec::size()) {
957:         bVec data_bvec = bVec::loadu(input_ptr + d);
958:         auto [data_fvec0, data_fvec1] = convert_to_float<scalar_t>(data_bvec);
959:         auto [mean_fvec0, mean_fvec1] = load2f(mean_data + d);
960:         fVec var_fvec0 = fVec::loadu(buffer_ptr + d);
961:         fVec var_fvec1 = fVec::loadu(buffer_ptr + d + fVec::size());
962:         var_fvec0 += (data_fvec0 - mean_fvec0) * (data_fvec0 - mean_fvec0);
963:         var_fvec1 += (data_fvec1 - mean_fvec1) * (data_fvec1 - mean_fvec1);
964:         var_fvec0.store(buffer_ptr + d);
965:         var_fvec1.store(buffer_ptr + d + fVec::size());
966:       }
967:       for (; d < n_channel; d++) {
968:         opmath_t data_val = opmath_t(input_ptr[d]);
969:         opmath_t mean_val = opmath_t(mean_data[d]);
970:         buffer_ptr[d] += (data_val - mean_val) * (data_val - mean_val);
971:       }
972:     }
973:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 975-1007
```cpp
 975:   for (const auto c : c10::irange(n_channel)) {
 976:     opmath_t _var_sum = 0;
 977:     for (const auto t : c10::irange(num_threads)) {
 978:       _var_sum += buffer_data[t * n_channel + c];
 979:     }
 980:     var_sum_data[c] = param_t(_var_sum);
 981:   }
 982: }
 983:
 984: template <typename scalar_t>
 985: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
 986: batch_norm_cpu_collect_stats_channels_last_impl(
 987:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
 988:   const bool mixed_type = is_mixed_type(input, mean, var_sum);
 989:   if (mixed_type) {
 990:     batch_norm_cpu_collect_stats_channels_last_internal<scalar_t, at::opmath_type<scalar_t>>(mean, var_sum, input);
 991:   } else {
 992:     batch_norm_cpu_collect_stats_channels_last_internal<scalar_t, scalar_t>(mean, var_sum, input);
 993:   }
 994: }
 995:
 996: template <typename scalar_t, typename param_t>
 997: void batch_norm_cpu_backward_contiguous_internal(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
 998:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
 999:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
1000:     bool train, double eps) {
1001:   using opmath_t = at::opmath_type<scalar_t>;
1002:   using bVec = Vectorized<scalar_t>;
1003:   using fVec = Vectorized<opmath_t>;
1004:   int64_t n_batch = input.size(0);
1005:   int64_t n_channel = input.size(1);
1006:   int64_t image_size = input.numel() / n_batch / n_channel;
1007:   int64_t N = input.numel() / n_channel;
```
- EN: The main symbol in this range is `batch_norm_cpu_collect_stats_channels_last_impl`, `batch_norm_cpu_backward_contiguous_internal`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `batch_norm_cpu_collect_stats_channels_last_impl`, `batch_norm_cpu_backward_contiguous_internal`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1009-1037
```cpp
1009:   const scalar_t* grad_output_data = grad_output.const_data_ptr<scalar_t>();
1010:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
1011:
1012:   scalar_t* grad_input_data = grad_input.defined() ? grad_input.mutable_data_ptr<scalar_t>() : nullptr;
1013:   param_t* grad_weight_data = grad_weight.defined() ? grad_weight.data_ptr<param_t>() : nullptr;
1014:   param_t* grad_bias_data = grad_bias.defined() ? grad_bias.data_ptr<param_t>() : nullptr;
1015:   const bool grad_input_null = grad_input_data == nullptr;
1016:   const bool grad_weight_null = grad_weight_data == nullptr;
1017:   const bool grad_bias_null = grad_bias_data == nullptr;
1018:
1019:   auto weight_a = conditional_accessor_1d<const param_t>(weight);
1020:   auto save_mean_a = conditional_accessor_1d<const param_t>(save_mean);
1021:   auto save_invstd_a = conditional_accessor_1d<const param_t>(save_invstd);
1022:   auto running_mean_a = conditional_accessor_1d<const param_t>(running_mean);
1023:   auto running_var_a = conditional_accessor_1d<const param_t>(running_var);
1024:
1025:   // parallel dim reduce on 'channel'
1026:   at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
1027:     for (const auto c : c10::irange(begin, end)) {
1028:       opmath_t w = weight.defined() ? opmath_t(weight_a[c]) : 1;
1029:
1030:       opmath_t mean, invstd;
1031:       if (train) {
1032:         mean = save_mean_a[c];
1033:         invstd = save_invstd_a[c];
1034:       } else {
1035:         mean = running_mean_a[c];
1036:         invstd = 1 / std::sqrt(running_var_a[c] + eps);
1037:       }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1039-1067
```cpp
1039:       // compute 1) sum; 2) dot product of Q(X) and dY.
1040:       opmath_t sum{0}, dotp{0};
1041:       fVec sum_fvec{0}, dotp_fvec{0};
1042:       for (const auto n : c10::irange(n_batch)) {
1043:         const scalar_t* x_ptr = input_data + n * n_channel * image_size + c * image_size;
1044:         const scalar_t* dy_ptr = grad_output_data + n * n_channel * image_size + c * image_size;
1045:
1046:         int64_t d = 0;
1047:         for (; d < image_size - (image_size % bVec::size()); d += bVec::size()) {
1048:           bVec dy_bvec = bVec::loadu(dy_ptr + d);
1049:           auto [dy_fvec0, dy_fvec1] = convert_to_float<scalar_t>(dy_bvec);
1050:           sum_fvec += dy_fvec0;
1051:           sum_fvec += dy_fvec1;
1052:
1053:           bVec x_bvec = bVec::loadu(x_ptr + d);
1054:           auto [x_fvec0, x_fvec1] = convert_to_float<scalar_t>(x_bvec);
1055:           dotp_fvec += (x_fvec0 - fVec(mean)) * dy_fvec0;
1056:           dotp_fvec += (x_fvec1 - fVec(mean)) * dy_fvec1;
1057:         }
1058:         for (; d < image_size; d++) {
1059:           sum += opmath_t(dy_ptr[d]);
1060:           dotp += (opmath_t(x_ptr[d]) - mean) * opmath_t(dy_ptr[d]);
1061:         }
1062:       }
1063:       // TODO: use fast version
1064:       sum += vec_reduce_all([](fVec& x, fVec& y) { return x + y; }, sum_fvec, fVec::size());
1065:       dotp += vec_reduce_all([](fVec& x, fVec& y) { return x + y; }, dotp_fvec, fVec::size());
1066:
1067:       if (!grad_input_null) {
```
- EN: The main symbol in this range is `vec_reduce_all`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `vec_reduce_all`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1068-1102
```cpp
1068:         if (train) {
1069:           opmath_t k = dotp * invstd * invstd / N;
1070:           opmath_t grad_mean = sum / N;
1071:           for (const auto n : c10::irange(n_batch)) {
1072:             const scalar_t* x_ptr = input_data + n * n_channel * image_size + c * image_size;
1073:             scalar_t* dx_ptr = grad_input_data + n * n_channel * image_size + c * image_size;
1074:             const scalar_t* dy_ptr = grad_output_data + n * n_channel * image_size + c * image_size;
1075:             vec::map2(
1076:                 [=](fVec x, fVec dy) {
1077:                   fVec dx = (x - fVec(mean)) * fVec(k);
1078:                   return (dy - fVec(grad_mean) - dx) * fVec(invstd) * fVec(w);
1079:                 },
1080:                 dx_ptr, x_ptr, dy_ptr, image_size);
1081:           }
1082:         } else { // evaluation mode
1083:           for (const auto n : c10::irange(n_batch)) {
1084:             scalar_t* dx_ptr = grad_input_data + n * n_channel * image_size + c * image_size;
1085:             const scalar_t* dy_ptr = grad_output_data + n * n_channel * image_size + c * image_size;
1086:             vec::map(
1087:                 [=](fVec dy) { return dy * fVec(invstd) * fVec(w); },
1088:                 dx_ptr, dy_ptr, image_size);
1089:           }
1090:         }
1091:       }
1092:
1093:       if (!grad_weight_null) {
1094:         grad_weight_data[c] = param_t(dotp * invstd);
1095:       }
1096:
1097:       if (!grad_bias_null) {
1098:         grad_bias_data[c] = param_t(sum);
1099:       }
1100:     }
1101:   });
1102: }
```
- EN: The main symbol in this range is `map2`, `map`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `map2`, `map`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1104-1132
```cpp
1104: template <typename scalar_t>
1105: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
1106: batch_norm_cpu_backward_contiguous_impl(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
1107:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
1108:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
1109:     bool train, double eps) {
1110:   const bool mixed_type = is_mixed_type(input, weight, running_mean, running_var, save_mean, save_invstd);
1111:   if (mixed_type) {
1112:     batch_norm_cpu_backward_contiguous_internal<scalar_t, at::opmath_type<scalar_t>>(grad_input, grad_weight, grad_bias,
1113:         grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1114:   } else {
1115:     batch_norm_cpu_backward_contiguous_internal<scalar_t, scalar_t>(grad_input, grad_weight, grad_bias,
1116:         grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1117:   }
1118: }
1119:
1120: template <typename scalar_t, typename param_t>
1121: void batch_norm_cpu_backward_channels_last_internal(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
1122:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
1123:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
1124:     bool train, double eps) {
1125:   using opmath_t = at::opmath_type<scalar_t>;
1126:   using bVec = Vectorized<scalar_t>;
1127:   using fVec = Vectorized<opmath_t>;
1128:   int64_t n_channel = input.size(1);
1129:   int64_t N = input.numel() / n_channel;
1130:
1131:   const scalar_t* grad_output_data = grad_output.const_data_ptr<scalar_t>();
1132:   const scalar_t* input_data = input.const_data_ptr<scalar_t>();
```
- EN: The main symbol in this range is `batch_norm_cpu_backward_contiguous_impl`, `batch_norm_cpu_backward_channels_last_internal`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `batch_norm_cpu_backward_contiguous_impl`, `batch_norm_cpu_backward_channels_last_internal`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1134-1163
```cpp
1134:   scalar_t* grad_input_data = grad_input.defined() ? grad_input.mutable_data_ptr<scalar_t>() : nullptr;
1135:   param_t* grad_weight_data = grad_weight.defined() ? grad_weight.data_ptr<param_t>() : nullptr;
1136:   param_t* grad_bias_data = grad_bias.defined() ? grad_bias.data_ptr<param_t>() : nullptr;
1137:
1138:   auto weight_a = conditional_accessor_1d<const param_t>(weight);
1139:   auto save_mean_a = conditional_accessor_1d<const param_t>(save_mean);
1140:   auto save_invstd_a = conditional_accessor_1d<const param_t>(save_invstd);
1141:   auto running_mean_a = conditional_accessor_1d<const param_t>(running_mean);
1142:   auto running_var_a = conditional_accessor_1d<const param_t>(running_var);
1143:
1144:   // use float as acc type
1145:   bool weight_defined = weight.defined();
1146:   Tensor weight_f = at::empty({n_channel}, input.options().dtype(kFloat));
1147:   Tensor mean = at::empty({n_channel}, input.options().dtype(kFloat));
1148:   Tensor invstd = at::empty({n_channel}, input.options().dtype(kFloat));
1149:   opmath_t* weight_data = weight_f.data_ptr<opmath_t>();
1150:   opmath_t* mean_data = mean.data_ptr<opmath_t>();
1151:   opmath_t* invstd_data = invstd.data_ptr<opmath_t>();
1152:
1153:   for (const auto c : c10::irange(n_channel)) {
1154:     weight_data[c] = weight_defined ? opmath_t(weight_a[c]) : 1;
1155:
1156:     if (train) {
1157:       mean_data[c] = save_mean_a[c];
1158:       invstd_data[c] = save_invstd_a[c];
1159:     } else {
1160:       mean_data[c] = running_mean_a[c];
1161:       invstd_data[c] = 1 / std::sqrt(running_var_a[c] + eps);
1162:     }
1163:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1165-1207
```cpp
1165:   int num_threads = at::get_num_threads();
1166:   Tensor buffer = at::zeros({2, num_threads, n_channel}, input.options().dtype(kFloat));
1167:   opmath_t* sum_data = buffer.data_ptr<opmath_t>();
1168:   opmath_t* dotp_data = sum_data + num_threads * n_channel;
1169:
1170:   at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
1171:     int tid = at::get_thread_num();
1172:     TORCH_CHECK(tid < num_threads, "expect thread id smaller than ", num_threads, ", got thread id ", tid);
1173:     opmath_t* sum_ptr = sum_data + tid * n_channel;
1174:     opmath_t* dotp_ptr = dotp_data + tid * n_channel;
1175:     for (const auto i : c10::irange(begin, end)) {
1176:       const scalar_t* x_ptr = input_data + i * n_channel;
1177:       const scalar_t* dy_ptr = grad_output_data + i * n_channel;
1178:
1179:       int64_t d = 0;
1180:       for(; d < n_channel - (n_channel % bVec::size()); d += bVec::size()) {
1181:         bVec dy_bvec = bVec::loadu(dy_ptr + d);
1182:         auto [dy_fvec0, dy_fvec1] = convert_to_float<scalar_t>(dy_bvec);
1183:         fVec sum_fvec0 = dy_fvec0 + fVec::loadu(sum_ptr + d);
1184:         fVec sum_fvec1 = dy_fvec1 + fVec::loadu(sum_ptr + d + fVec::size());
1185:         sum_fvec0.store(sum_ptr + d);
1186:         sum_fvec1.store(sum_ptr + d + fVec::size());
1187:
1188:         bVec x_bvec = bVec::loadu(x_ptr + d);
1189:         auto [x_fvec0, x_fvec1] = convert_to_float<scalar_t>(x_bvec);
1190:         fVec mean_fvec0 = fVec::loadu(mean_data + d);
1191:         fVec mean_fvec1 = fVec::loadu(mean_data + d + fVec::size());
1192:         fVec dotp_fvec0 = fVec::loadu(dotp_ptr + d);
1193:         fVec dotp_fvec1 = fVec::loadu(dotp_ptr + d + fVec::size());
1194:         dotp_fvec0 += (x_fvec0 - mean_fvec0) * dy_fvec0;
1195:         dotp_fvec1 += (x_fvec1 - mean_fvec1) * dy_fvec1;
1196:         dotp_fvec0.store(dotp_ptr + d);
1197:         dotp_fvec1.store(dotp_ptr + d + fVec::size());
1198:       }
1199:       for (; d < n_channel; d++) {
1200:         opmath_t dy_val = dy_ptr[d];
1201:         opmath_t x_val = x_ptr[d];
1202:         opmath_t mean_val = mean_data[d];
1203:         sum_ptr[d] += dy_val;
1204:         dotp_ptr[d] += (x_val - mean_val) * dy_val;
1205:       }
1206:     }
1207:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 1209-1237
```cpp
1209:   at::parallel_for(0, n_channel, 1, [&](int64_t begin, int64_t end) {
1210:     for (const auto c : c10::irange(begin, end)) {
1211:       // store the final result of sum and dotp in the 1st lane of immediate buffer,
1212:       // so that we won't need to allocate anther buffer to store the temp values.
1213:       opmath_t _sum = 0;
1214:       for (const auto t : c10::irange(num_threads)) {
1215:         _sum += sum_data[t * n_channel + c];
1216:       }
1217:       sum_data[/* 0 * n_channel + */c] = _sum;
1218:
1219:       opmath_t _dotp = 0;
1220:       for (const auto t : c10::irange(num_threads)) {
1221:         _dotp += dotp_data[t * n_channel + c];
1222:       }
1223:       dotp_data[/* 0 * n_channel + */c] = _dotp;
1224:     }
1225:   });
1226:
1227:   // compute grad_input
1228:   if (grad_input.defined()) {
1229:     at::parallel_for(0, N, 1, [&](int64_t begin, int64_t end) {
1230:       for (const auto i : c10::irange(begin, end)) {
1231:         scalar_t* dx_ptr = grad_input_data + i * n_channel;
1232:         const scalar_t* x_ptr = input_data + i * n_channel;
1233:         const scalar_t* dy_ptr = grad_output_data + i * n_channel;
1234:         if (train) {
1235:           int64_t d = 0;
1236:           for (; d < n_channel - (n_channel % bVec::size()); d += bVec::size()) {
1237:             bVec x_bvec = bVec::loadu(x_ptr + d);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1238-1267
```cpp
1238:             auto [x_fvec0, x_fvec1] = convert_to_float<scalar_t>(x_bvec);
1239:             fVec mean_fvec0 = fVec::loadu(mean_data + d);
1240:             fVec mean_fvec1 = fVec::loadu(mean_data + d + fVec::size());
1241:             fVec dotp_fvec0 = fVec::loadu(dotp_data + d);
1242:             fVec dotp_fvec1 = fVec::loadu(dotp_data + d + fVec::size());
1243:             fVec invstd_fvec0 = fVec::loadu(invstd_data + d);
1244:             fVec invstd_fvec1 = fVec::loadu(invstd_data + d + fVec::size());
1245:             fVec k_fvec0 = dotp_fvec0 * invstd_fvec0 * invstd_fvec0 / fVec(N);
1246:             fVec k_fvec1 = dotp_fvec1 * invstd_fvec1 * invstd_fvec1 / fVec(N);
1247:             fVec dx_fvec0 = (x_fvec0 - mean_fvec0) * k_fvec0;
1248:             fVec dx_fvec1 = (x_fvec1 - mean_fvec1) * k_fvec1;
1249:             bVec dy_bvec = bVec::loadu(dy_ptr + d);
1250:             auto [dy_fvec0, dy_fvec1] = convert_to_float<scalar_t>(dy_bvec);
1251:             fVec grad_mean_fvec0 = fVec::loadu(sum_data + d) / fVec(N);
1252:             fVec grad_mean_fvec1 = fVec::loadu(sum_data + d + fVec::size()) / fVec(N);
1253:             fVec w_fvec0 = fVec::loadu(weight_data + d);
1254:             fVec w_fvec1 = fVec::loadu(weight_data + d + fVec::size());
1255:             dx_fvec0 = (dy_fvec0 - grad_mean_fvec0 - dx_fvec0) * invstd_fvec0 * w_fvec0;
1256:             dx_fvec1 = (dy_fvec1 - grad_mean_fvec1 - dx_fvec1) * invstd_fvec1 * w_fvec1;
1257:             bVec dx_bvec = convert_from_float<scalar_t>(dx_fvec0, dx_fvec1);
1258:             dx_bvec.store(dx_ptr + d);
1259:           }
1260:           for (; d < n_channel; d++) {
1261:             opmath_t x_val = x_ptr[d];
1262:             opmath_t mean_val = mean_data[d];
1263:             opmath_t dotp_val = dotp_data[d];
1264:             opmath_t invstd_val = invstd_data[d];
1265:             opmath_t k_val = dotp_val * invstd_val * invstd_val / N;
1266:             opmath_t dx_val = (x_val - mean_val) * k_val;
1267:             opmath_t dy_val = dy_ptr[d];
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 1268-1297
```cpp
1268:             opmath_t grad_mean_val = sum_data[d] / N;
1269:             opmath_t w_val = weight_data[d];
1270:             dx_val = (dy_val - grad_mean_val - dx_val) * invstd_val * w_val;
1271:             dx_ptr[d] = scalar_t(dx_val);
1272:           }
1273:         } else { // evaluation mode
1274:           int64_t d = 0;
1275:           for (; d < n_channel - (n_channel % bVec::size()); d += bVec::size()) {
1276:             bVec dy_bvec = bVec::loadu(dy_ptr + d);
1277:             auto [dy_fvec0, dy_fvec1] = convert_to_float<scalar_t>(dy_bvec);
1278:             fVec invstd_fvec0 = fVec::loadu(invstd_data + d);
1279:             fVec invstd_fvec1 = fVec::loadu(invstd_data + d + fVec::size());
1280:             fVec w_fvec0 = fVec::loadu(weight_data + d);
1281:             fVec w_fvec1 = fVec::loadu(weight_data + d + fVec::size());
1282:             fVec dx_fvec0 = dy_fvec0 * invstd_fvec0 * w_fvec0;
1283:             fVec dx_fvec1 = dy_fvec1 * invstd_fvec1 * w_fvec1;
1284:             bVec dx_bvec = convert_from_float<scalar_t>(dx_fvec0, dx_fvec1);
1285:             dx_bvec.store(dx_ptr + d);
1286:           }
1287:           for (; d < n_channel; d++) {
1288:             opmath_t dy_val = dy_ptr[d];
1289:             opmath_t invstd_val = invstd_data[d];
1290:             opmath_t w_val = weight_data[d];
1291:             opmath_t dx_val = dy_val * invstd_val * w_val;
1292:             dx_ptr[d] = scalar_t(dx_val);
1293:           }
1294:         }
1295:       }
1296:     });
1297:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 1299-1326
```cpp
1299:   if (grad_weight.defined()) {
1300:     for (const auto c : c10::irange(n_channel)) {
1301:       grad_weight_data[c] = param_t(dotp_data[c] * invstd_data[c]);
1302:     }
1303:   }
1304:
1305:   if (grad_bias.defined()) {
1306:     for (const auto c : c10::irange(n_channel)) {
1307:       grad_bias_data[c] = param_t(sum_data[c]);
1308:     }
1309:   }
1310: }
1311:
1312: template <typename scalar_t>
1313: typename std::enable_if_t<!std::is_same_v<scalar_t, at::opmath_type<scalar_t>>, void>
1314: batch_norm_cpu_backward_channels_last_impl(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
1315:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
1316:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
1317:     bool train, double eps) {
1318:   const bool mixed_type = is_mixed_type(input, weight, running_mean, running_var, save_mean, save_invstd);
1319:   if (mixed_type) {
1320:     batch_norm_cpu_backward_channels_last_internal<scalar_t, at::opmath_type<scalar_t>>(grad_input, grad_weight, grad_bias,
1321:         grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1322:   } else {
1323:     batch_norm_cpu_backward_channels_last_internal<scalar_t, scalar_t>(grad_input, grad_weight, grad_bias,
1324:         grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1325:   }
1326: }
```
- EN: The main symbol in this range is `batch_norm_cpu_backward_channels_last_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `batch_norm_cpu_backward_channels_last_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1328-1370
```cpp
1328: void batch_norm_cpu_kernel(Tensor& output, const Tensor& input,
1329:     const Tensor& weight, const Tensor& bias, const Tensor& save_mean,  const Tensor& save_invstd,
1330:     const Tensor& running_mean, const Tensor& running_var, bool train, double eps) {
1331:   int64_t image_size = input.numel() / input.size(0) / input.size(1);
1332:   if (input.is_contiguous()) { // NC11 is also channels last
1333:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "batch_norm_cpu_contiguous", [&] {
1334:       if (image_size == 1) {
1335:         batch_norm_cpu_channels_last_impl<scalar_t>(output, input, weight, bias,
1336:             save_mean, save_invstd, running_mean, running_var, train, eps);
1337:       } else {
1338:         batch_norm_cpu_contiguous_impl<scalar_t>(output, input, weight, bias,
1339:             save_mean, save_invstd, running_mean, running_var, train, eps);
1340:       }
1341:     });
1342:   } else if (input.is_contiguous(at::MemoryFormat::ChannelsLast) || input.is_contiguous(at::MemoryFormat::ChannelsLast3d)) {
1343:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "batch_norm_cpu_channels_last", [&] {
1344:       batch_norm_cpu_channels_last_impl<scalar_t>(output, input, weight, bias,
1345:           save_mean, save_invstd, running_mean, running_var, train, eps);
1346:     });
1347:   } else {
1348:     TORCH_CHECK(false, "batch_norm_cpu_kernel: expecting input to be contiguous.");
1349:   }
1350: }
1351:
1352: void batch_norm_cpu_collect_stats_kernel(
1353:     Tensor& mean, Tensor& var_sum, const Tensor& input) {
1354:   int64_t image_size = input.numel() / input.size(0) / input.size(1);
1355:   if (input.is_contiguous()) {
1356:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "batch_norm_cpu_collect_stats_contiguous", [&] {
1357:       if (image_size == 1) { // NC11 is also channels last
1358:         batch_norm_cpu_collect_stats_channels_last_impl<scalar_t>(mean, var_sum, input);
1359:       } else {
1360:         batch_norm_cpu_collect_stats_contiguous_impl<scalar_t>(mean, var_sum, input);
1361:       }
1362:     });
1363:   } else if (input.is_contiguous(at::MemoryFormat::ChannelsLast) || input.is_contiguous(at::MemoryFormat::ChannelsLast3d)) {
1364:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "batch_norm_cpu_collect_stats_channels_last", [&] {
1365:       batch_norm_cpu_collect_stats_channels_last_impl<scalar_t>(mean, var_sum, input);
1366:     });
1367:   } else {
1368:     TORCH_CHECK(false, "batch_norm_cpu_collect_stats_kernel: expecting input to be contiguous.");
1369:   }
1370: }
```
- EN: The main symbol in this range is `batch_norm_cpu_kernel`, `batch_norm_cpu_collect_stats_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `batch_norm_cpu_kernel`, `batch_norm_cpu_collect_stats_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1372-1401
```cpp
1372: void batch_norm_cpu_backward_kernel(Tensor& grad_input, Tensor& grad_weight, Tensor& grad_bias,
1373:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
1374:     const Tensor& running_mean, const Tensor& running_var, const Tensor& save_mean, const Tensor& save_invstd,
1375:     bool train, double eps) {
1376:   int64_t image_size = input.numel() / input.size(0) / input.size(1);
1377:   if (input.is_contiguous()) {
1378:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "batch_norm_cpu_backward_contiguous", [&] {
1379:       if (image_size == 1) { // NC11 is also channels last
1380:         batch_norm_cpu_backward_channels_last_impl<scalar_t>(grad_input, grad_weight, grad_bias,
1381:             grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1382:       } else {
1383:         batch_norm_cpu_backward_contiguous_impl<scalar_t>(grad_input, grad_weight, grad_bias,
1384:             grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1385:       }
1386:     });
1387:   } else if (input.is_contiguous(at::MemoryFormat::ChannelsLast) || input.is_contiguous(at::MemoryFormat::ChannelsLast3d)) {
1388:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, input.scalar_type(), "batch_norm_cpu_backward_channels_last", [&] {
1389:       batch_norm_cpu_backward_channels_last_impl<scalar_t>(grad_input, grad_weight, grad_bias,
1390:           grad_output, input, weight, running_mean, running_var, save_mean, save_invstd, train, eps);
1391:     });
1392:   } else {
1393:     TORCH_CHECK(false, "batch_norm_cpu_backward_kernel: expecting input to be contiguous.");
1394:   }
1395: }
1396:
1397: }// anonymous namespace
1398:
1399: REGISTER_DISPATCH(batch_norm_cpu_stub, &batch_norm_cpu_kernel)
1400: REGISTER_DISPATCH(batch_norm_cpu_collect_stats_stub, &batch_norm_cpu_collect_stats_kernel)
1401: REGISTER_DISPATCH(batch_norm_cpu_backward_stub, &batch_norm_cpu_backward_kernel)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `batch_norm_cpu_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `batch_norm_cpu_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1403-1403
```cpp
1403: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Normalization statistics / 归一化统计

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/batch_norm.h`, `ATen/core/Tensor.h`, `ATen/AccumulateType.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/native/TensorIterator.h`, `ATen/native/cpu/Loops.h`, `ATen/native/cpu/utils.h`, `ATen/native/cpu/mixed_data_type.h`, `ATen/cpu/vec/functional.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `cpu_kernel`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
