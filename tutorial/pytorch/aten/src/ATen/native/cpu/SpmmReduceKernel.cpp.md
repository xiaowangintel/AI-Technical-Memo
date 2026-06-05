# SpmmReduceKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/SpmmReduceKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Spmm Reduce Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Spmm Reduce Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/ExpandUtils.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/Parallel.h>
 6: #include <ATen/cpu/vec/functional.h>
 7: #include <ATen/cpu/vec/vec.h>
 8: #include <ATen/native/cpu/SpmmReduceKernel.h>
 9: #include <ATen/native/cpu/ReduceUtils.h>
10: #include <ATen/native/cpu/utils.h>
11: #include <c10/util/irange.h>
12: #include <ATen/OpMathType.h>
13:
14: #ifndef AT_PER_OPERATOR_HEADERS
15: #include <ATen/Functions.h>
16: #else
17: #include <ATen/ops/empty.h>
18: #include <ATen/ops/empty_native.h>
19: #include <ATen/ops/zeros.h>
20: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/ExpandUtils.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/ExpandUtils.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 22-42
```cpp
22: namespace at::native {
23:
24: namespace {
25:
26: template <typename scalar_t, typename index_t, ReductionType reduce>
27: inline void _update(at::opmath_type<scalar_t>* out_ptr, int64_t e, int64_t c, const scalar_t val, const scalar_t* other_data, int64_t K) {
28:   using opmath_t = at::opmath_type<scalar_t>;
29:   using Vec = vec::Vectorized<scalar_t>;
30:   using aVec = VecType<scalar_t>;
31:   constexpr int64_t kVecSize = Vec::size();
32:   constexpr int64_t kVLEN = kVecSize * 4;
33:
34:   int64_t k = 0;
35:   aVec val_vec = aVec((opmath_t)val);
36:   const scalar_t* other_ptr = other_data + c * K;
37:
38:   for (; k < K - (K % kVLEN); k += kVLEN) {
39:     aVec out_vec0 = aVec::loadu(out_ptr + k);
40:     aVec out_vec1 = aVec::loadu(out_ptr + k + kVecSize);
41:     aVec out_vec2 = aVec::loadu(out_ptr + k + kVecSize * 2);
42:     aVec out_vec3 = aVec::loadu(out_ptr + k + kVecSize * 3);
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_update`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_update`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 44-64
```cpp
44:     out_vec0 = update<aVec, reduce>(out_vec0, aVec::loadu(other_ptr + k) * val_vec);
45:     out_vec1 = update<aVec, reduce>(out_vec1, aVec::loadu(other_ptr + k + kVecSize) * val_vec);
46:     out_vec2 = update<aVec, reduce>(out_vec2, aVec::loadu(other_ptr + k + kVecSize * 2) * val_vec);
47:     out_vec3 = update<aVec, reduce>(out_vec3, aVec::loadu(other_ptr + k + kVecSize * 3) * val_vec);
48:
49:     out_vec0.store(out_ptr + k);
50:     out_vec1.store(out_ptr + k + kVecSize);
51:     out_vec2.store(out_ptr + k + kVecSize * 2);
52:     out_vec3.store(out_ptr + k + kVecSize * 3);
53:   }
54:   for (; k < K - (K % kVecSize); k += kVecSize) {
55:     aVec out_vec = aVec::loadu(out_ptr + k);
56:     out_vec = update<aVec, reduce>(out_vec, aVec::loadu(other_ptr + k) * val_vec);
57:     out_vec.store(out_ptr + k);
58:   }
59:   for (; k < K; k++) {
60:     opmath_t out_val = opmath_t(out_ptr[k]);
61:     out_val = update<opmath_t, reduce>(out_val, opmath_t(other_ptr[k]) * opmath_t(val));
62:     out_ptr[k] = out_val;
63:   }
64: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 66-86
```cpp
66: template <typename scalar_t, typename index_t, ReductionType reduce>
67: void spmm_reduce_kernel_impl(
68:     const Tensor& out,
69:     const Tensor& crow_indices,
70:     const Tensor& col_indices,
71:     const Tensor& values,
72:     const Tensor& other_) {
73:
74:   int64_t nnz = values.numel();
75:   if (nnz == 0) {
76:     return;
77:   }
78:
79:   auto other = other_.contiguous();
80:
81:   // access `crow_indices`, `col_indices` and `values` via TensorAccessor
82:   scalar_t* out_data = out.data_ptr<scalar_t>();
83:   auto csr_data = crow_indices.accessor<const index_t, 1>();
84:   auto col_data = col_indices.accessor<const index_t, 1>();
85:   auto val_data = values.accessor<const scalar_t, 1>();
86:   const scalar_t* other_data = other.const_data_ptr<scalar_t>();
```
- EN: The main symbol in this range is `spmm_reduce_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `spmm_reduce_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 88-106
```cpp
 88:   int64_t M = crow_indices.numel() - 1;
 89:   int64_t K = other.size(-1);
 90:
 91:   int num_threads = at::get_num_threads();
 92:   using opmath_t = at::opmath_type<scalar_t>;
 93:   Tensor buffer;
 94:   opmath_t* buffer_data = nullptr;
 95:   static constexpr bool need_acc = is_reduced_floating_point_v<scalar_t>;
 96:   if constexpr (need_acc) {
 97:     auto acc_type = at::toAccumulateType(out.scalar_type(), /*is_cuda=*/true);
 98:     buffer = at::zeros({num_threads, K}, out.options().dtype(acc_type));
 99:     buffer_data = buffer.data_ptr<opmath_t>();
100:   }
101:
102:   utils::parallel_sparse_csr(csr_data, M, nnz, [&](int64_t begin, int64_t end) {
103:     int tid = at::get_thread_num();
104:     TORCH_CHECK(tid < num_threads,
105:                 "expect thread id smaller than ", num_threads, ", got thread id ", tid);
106:     opmath_t* buffer_ptr = nullptr;
```
- EN: The main symbol in this range is `constexpr`, `parallel_sparse_csr`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `constexpr`, `parallel_sparse_csr`，它们直接构成本文件的算子逻辑。

### Lines 108-126
```cpp
108:     int64_t row_start = 0, row_end = 0;
109:     for (const auto m : c10::irange(begin, end)) {
110:       row_start = csr_data[m];
111:       row_end = csr_data[m + 1];
112:
113:       scalar_t* out_ptr = out_data + m * K;
114:       if constexpr (need_acc) {
115:         buffer_ptr = buffer_data + tid * K;
116:       } else {
117:         buffer_ptr = reinterpret_cast<opmath_t*>(out_ptr);
118:       }
119:
120:       // step 1: reinit the output row for reduce type 'amax' and 'amin'
121:       int64_t count = row_end - row_start;
122:       if (count != 0) {
123:         _init<scalar_t, reduce>(out_ptr, buffer_ptr, K, /*include_self*/false);
124:       }
125:
126:       // step 2: reduce, do blocking on rowwise to reduce write memory bandwidth
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 127-146
```cpp
127:       constexpr int64_t CHUNK_SIZE = 16;
128:       for (int64_t e0 = row_start; e0 < row_end; e0 += CHUNK_SIZE) {
129:         int64_t e1 = std::min(e0 + CHUNK_SIZE, row_end);
130:         for (const auto e : c10::irange(e0, e1)) {
131:           int64_t c = col_data[e];
132:           scalar_t val = val_data[e];
133:           _update<scalar_t, index_t, reduce>(buffer_ptr, e, c, val, other_data, K);
134:         }
135:       }
136:       if constexpr (need_acc) {
137:         if (count != 0) {
138:           vec::convert(buffer_ptr, out_ptr, K);
139:         }
140:       }
141:
142:       // step 3: finalize
143:       write<scalar_t, reduce>(out_ptr, count, K);
144:     }
145:   });
146: }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 148-169
```cpp
148: // update both val and arg, used for `amin` and `amax`
149: // it is a little troublesome to vectorize it since `scalar_t` and `index_t`
150: // might have different vector length, for example, each vector holds 8 floats
151: // and 4 int64_t.
152: template <typename scalar_t, typename index_t, ReductionType reduce>
153: inline void update_with_index(scalar_t *val, scalar_t new_val, index_t *arg, index_t new_arg) {
154:   if ((reduce == ReductionType::MIN && new_val < *val) ||
155:       (reduce == ReductionType::MAX && new_val > *val) ||
156:       at::_isnan<scalar_t>(new_val)) {
157:     *val = new_val;
158:     *arg = new_arg;
159:   }
160: }
161:
162: template <typename scalar_t, typename index_t, ReductionType reduce>
163: void spmm_reduce_arg_kernel_impl(
164:     const Tensor& out,
165:     const Tensor& arg_out,
166:     const Tensor& crow_indices,
167:     const Tensor& col_indices,
168:     const Tensor& values,
169:     const Tensor& other_) {
```
- EN: The main symbol in this range is `update_with_index`, `spmm_reduce_arg_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `update_with_index`, `spmm_reduce_arg_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 171-198
```cpp
171:   TORCH_CHECK(reduce == ReductionType::MAX || reduce == ReductionType::MIN);
172:   int64_t nnz = values.numel();
173:   if (nnz == 0) {
174:     return;
175:   }
176:
177:   auto other = other_.contiguous();
178:
179:   scalar_t* out_data = out.data_ptr<scalar_t>();
180:   index_t* arg_out_data = arg_out.data_ptr<index_t>();
181:   auto csr_data = crow_indices.accessor<const index_t, 1>();
182:   auto col_data = col_indices.accessor<const index_t, 1>();
183:   auto val_data = values.accessor<const scalar_t, 1>();
184:   const scalar_t* other_data = other.const_data_ptr<scalar_t>();
185:
186:   int64_t M = crow_indices.numel() - 1;
187:   int64_t K = other.size(-1);
188:
189:   int num_threads = at::get_num_threads();
190:   using opmath_t = at::opmath_type<scalar_t>;
191:   Tensor buffer;
192:   opmath_t* buffer_data = nullptr;
193:   static constexpr bool need_acc = is_reduced_floating_point_v<scalar_t>;
194:   if constexpr (need_acc) {
195:     auto acc_type = at::toAccumulateType(out.scalar_type(), /*is_cuda=*/true);
196:     buffer = at::zeros({num_threads, K}, out.options().dtype(acc_type));
197:     buffer_data = buffer.data_ptr<opmath_t>();
198:   }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 200-223
```cpp
200:   at::parallel_for(0, M, 1, [&](int64_t begin, int64_t end) {
201:     int tid = at::get_thread_num();
202:     TORCH_CHECK(tid < num_threads,
203:                 "expect thread id smaller than ", num_threads, ", got thread id ", tid);
204:     opmath_t* buffer_ptr = nullptr;
205:
206:     int64_t row_start = 0, row_end = 0, c = 0;
207:     for (const auto m : c10::irange(begin, end)) {
208:       row_start = csr_data[m];
209:       row_end = csr_data[m + 1];
210:
211:       scalar_t* out_ptr = out_data + m * K;
212:       index_t* arg_out_ptr = arg_out_data + m * K;
213:       if constexpr (need_acc) {
214:         buffer_ptr = buffer_data + tid * K;
215:       } else {
216:         buffer_ptr = reinterpret_cast<opmath_t*>(out_ptr);
217:       }
218:
219:       if (row_end != row_start) {
220:         _init<scalar_t, reduce>(out_ptr, buffer_ptr, K, /*include_self*/false);
221:         for (const auto e : c10::irange(row_start, row_end)) {
222:           c = col_data[e];
223:           opmath_t val = opmath_t(val_data[e]);
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 225-248
```cpp
225:           const scalar_t* other_ptr = other_data + c * K;
226:           for (const auto k : c10::irange(K)) {
227:             update_with_index<opmath_t, index_t, reduce>(
228:                 &buffer_ptr[k], opmath_t(val *  other_ptr[k]), &arg_out_ptr[k], index_t(e));
229:           };
230:         }
231:       }
232:       if constexpr (need_acc) {
233:         if (row_end != row_start) {
234:           vec::convert(buffer_ptr, out_ptr, K);
235:         }
236:       }
237:     }
238:   });
239: }
240:
241: template <typename scalar_t, typename index_t, ReductionType reduce>
242: void spmm_reduce_backward_input_kernel_impl(
243:     const Tensor& grad_self,
244:     const Tensor& grad_out_,
245:     const Tensor& crow_indices,
246:     const Tensor& col_indices,
247:     const Tensor& other_,
248:     const Tensor& row_indices) {
```
- EN: The main symbol in this range is `constexpr`, `spmm_reduce_backward_input_kernel_impl`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`, `spmm_reduce_backward_input_kernel_impl`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 250-271
```cpp
250:   int64_t nnz = grad_self._nnz();
251:   if (nnz == 0) {
252:     return;
253:   }
254:
255:   auto grad_out = grad_out_.contiguous();
256:   auto other = other_.contiguous();
257:
258:   auto values = grad_self.values();
259:   auto grad_values_data = values.accessor<scalar_t, 1>();
260:   const scalar_t* grad_out_data = grad_out.const_data_ptr<scalar_t>();
261:   auto crow_data = crow_indices.accessor<const index_t, 1>();
262:   auto col_data = col_indices.accessor<const index_t, 1>();
263:   const scalar_t* other_data = other.const_data_ptr<scalar_t>();
264:   auto row_data = row_indices.accessor<const index_t, 1>();
265:
266:   int64_t K = grad_out.size(1);
267:
268:   using Vec = vec::Vectorized<vec::vec_scalar_t<scalar_t>>;
269:   at::parallel_for(0, nnz, 1, [&](int64_t begin, int64_t end) {
270:     for (const auto i : c10::irange(begin, end)) {
271:       index_t row = row_data[i], col = col_data[i];
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 273-297
```cpp
273:       scalar_t val = vec::map2_reduce_all<scalar_t>(
274:           [](Vec x, Vec y) { return x * y; },
275:           [](Vec x, Vec y) { return x + y; },
276:           other_data + col * K,
277:           grad_out_data + row * K,
278:           K);
279:
280:       if (reduce == ReductionType::MEAN) {
281:         index_t row_start = crow_data[row], row_end = crow_data[row + 1];
282:         val /= (row_end - row_start);
283:       }
284:
285:       grad_values_data[i] = val;
286:     }
287:   });
288: }
289:
290: // backward for reduce type 'amax' or 'amin'
291: template <typename scalar_t, typename index_t>
292: void spmm_reduce_backward_input_arg_kernel_impl(
293:     const Tensor& grad_self,
294:     const Tensor& grad_out_,
295:     const Tensor& col_indices,
296:     const Tensor& other_,
297:     const Tensor& arg_out_) {
```
- EN: The main symbol in this range is `map2_reduce_all<scalar_t>`, `spmm_reduce_backward_input_arg_kernel_impl`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `map2_reduce_all<scalar_t>`, `spmm_reduce_backward_input_arg_kernel_impl`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 299-318
```cpp
299:   int64_t nnz = grad_self._nnz();
300:   if (nnz == 0) {
301:     return;
302:   }
303:
304:   auto grad_out = grad_out_.contiguous();
305:   auto other = other_.contiguous();
306:   auto arg_out = arg_out_.contiguous();
307:
308:   auto grad_values = grad_self.values();
309:   auto grad_values_data = grad_values.accessor<scalar_t, 1>();
310:   const scalar_t* grad_out_data = grad_out.const_data_ptr<scalar_t>();
311:   auto col_data = col_indices.accessor<const index_t, 1>();
312:   const scalar_t* other_data = other.const_data_ptr<scalar_t>();
313:   const index_t* arg_out_data = arg_out.const_data_ptr<index_t>();
314:
315:   int64_t M = grad_out.size(0);
316:   int64_t K = grad_out.size(1);
317:   auto grad = at::empty({M, K}, grad_out.options());
318:   scalar_t* grad_data = grad.mutable_data_ptr<scalar_t>();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 320-345
```cpp
320:   at::parallel_for(0, M, 1, [&](int64_t begin, int64_t end) {
321:     for (const auto m : c10::irange(begin, end)) {
322:       const scalar_t* grad_out_ptr = grad_out_data + m * K;
323:       scalar_t* grad_ptr = grad_data + m * K;
324:       const index_t* arg_out_ptr = arg_out_data + m * K;
325:
326:       for (const auto k : c10::irange(K)) {
327:         if (arg_out_ptr[k] == index_t(nnz)) {
328:           grad_ptr[k] = scalar_t(0);
329:         } else {
330:           // collect weight at max/min indices
331:           index_t col = col_data[arg_out_data[m * K + k]];
332:           grad_ptr[k] = other_data[col * K + k] * grad_out_ptr[k];
333:         }
334:       }
335:     }
336:   });
337:
338:   // scatter_add, consider to parallel this with atomic
339:   for (const auto i : c10::irange(M * K)) {
340:     index_t ind = arg_out_data[i];
341:     if (ind != index_t(nnz)) {
342:       grad_values_data[ind] += grad_data[i];
343:     }
344:   }
345: }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 347-373
```cpp
347: template <typename scalar_t, typename index_t>
348: void spmm_reduce_normalize_values_kernel_impl(
349:     const Tensor& normalized_values,
350:     const Tensor& values,
351:     const Tensor& crow_indices,
352:     const Tensor& row_indices) {
353:
354:   int64_t nnz = values.numel();
355:   if (nnz == 0) {
356:     return;
357:   }
358:
359:   auto normalized_values_data = normalized_values.accessor<scalar_t, 1>();
360:   auto values_data = values.accessor<scalar_t, 1>();
361:   auto crow_data = crow_indices.accessor<index_t, 1>();
362:   auto row_data = row_indices.accessor<index_t, 1>();
363:
364:   at::parallel_for(0, nnz, 1, [&](int64_t begin, int64_t end) {
365:     for (const auto i : c10::irange(begin, end)) {
366:       index_t row = row_data[i];
367:       index_t row_start = crow_data[row], row_end = crow_data[row + 1];
368:       // Note that when the row index row is listed in row_indices,
369:       // then crow_indices[row+1] > crow_indices[row] holds
370:       normalized_values_data[i] = values_data[i] / (row_end - row_start);
371:     }
372:   });
373: }
```
- EN: The main symbol in this range is `spmm_reduce_normalize_values_kernel_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `spmm_reduce_normalize_values_kernel_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 375-395
```cpp
375: template <typename scalar_t, typename index_t>
376: void spmm_reduce_backward_other_arg_kernel_impl(
377:     const Tensor& grad_other,
378:     const Tensor& grad_out_,
379:     const Tensor& col_indices,
380:     const Tensor& values,
381:     const Tensor& arg_out_) {
382:
383:   int64_t nnz = values.numel();
384:   if (nnz == 0) {
385:     return;
386:   }
387:
388:   auto grad_out = grad_out_.contiguous();
389:   auto arg_out = arg_out_.contiguous();
390:
391:   scalar_t* grad_other_data = grad_other.data_ptr<scalar_t>();
392:   const scalar_t* grad_out_data = grad_out.const_data_ptr<scalar_t>();
393:   auto col_data = col_indices.accessor<const index_t, 1>();
394:   auto values_data = values.accessor<const scalar_t, 1>();
395:   const index_t* arg_out_data = arg_out.const_data_ptr<index_t>();
```
- EN: The main symbol in this range is `spmm_reduce_backward_other_arg_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `spmm_reduce_backward_other_arg_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 397-416
```cpp
397:   int64_t M = grad_out.size(0);
398:   int64_t K = grad_out.size(1);
399:   auto grad = at::empty({M, K}, grad_out.options());
400:   scalar_t* grad_data = grad.mutable_data_ptr<scalar_t>();
401:
402:   at::parallel_for(0, M, 1, [&](int64_t begin, int64_t end) {
403:     for (const auto m : c10::irange(begin, end)) {
404:       const scalar_t* grad_out_ptr = grad_out_data + m * K;
405:       scalar_t* grad_ptr = grad_data + m * K;
406:       const index_t* arg_out_ptr = arg_out_data + m * K;
407:
408:       for (const auto k : c10::irange(K)) {
409:         if (arg_out_ptr[k] == index_t(nnz)) {
410:           grad_ptr[k] = scalar_t(0);
411:         } else {
412:           grad_ptr[k] = values_data[arg_out_ptr[k]] * grad_out_ptr[k];
413:         }
414:       }
415:     }
416:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 418-445
```cpp
418:   // scatter_add, consider to parallel this with atomic
419:   for (const auto m : c10::irange(M)) {
420:     for (const auto k : c10::irange(K)) {
421:       index_t ind = arg_out_data[m * K + k];
422:       if (ind != index_t(nnz)) {
423:         index_t col = col_data[ind];
424:         grad_other_data[col * K + k] += grad_data[m * K + k];
425:       }
426:     }
427:   }
428: }
429:
430: void spmm_reduce_kernel(
431:     const Tensor& out,
432:     const Tensor& crow_indices,
433:     const Tensor& col_indices,
434:     const Tensor& values,
435:     const Tensor& other,
436:     ReductionType reduce_op) {
437:     AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, values.scalar_type(), "spmm_reduce_kernel", [&]() {
438:       AT_DISPATCH_INDEX_TYPES(col_indices.scalar_type(), "spmm_reduce_indices", [&]() {
439:         AT_DISPATCH_REDUCTION_TYPES(reduce_op, [&]() {
440:           spmm_reduce_kernel_impl<scalar_t, index_t, reduce>(
441:               out, crow_indices, col_indices, values, other);
442:         });
443:       });
444:     });
445: }
```
- EN: The main symbol in this range is `spmm_reduce_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `spmm_reduce_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 447-465
```cpp
447: void spmm_reduce_arg_kernel(
448:     const Tensor& out,
449:     const Tensor& arg_out,
450:     const Tensor& crow_indices,
451:     const Tensor& col_indices,
452:     const Tensor& values,
453:     const Tensor& other,
454:     ReductionType reduce_op) {
455:   AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, values.scalar_type(), "spmm_reduce_kernel", [&]() {
456:     AT_DISPATCH_INDEX_TYPES(col_indices.scalar_type(), "spmm_reduce_indices", [&]() {
457:       AT_DISPATCH_REDUCTION_TYPES(reduce_op, [&]() {
458:         spmm_reduce_arg_kernel_impl<scalar_t, index_t, reduce>(
459:             out, arg_out, crow_indices, col_indices, values, other);
460:       });
461:     });
462:   });
463: }
464:
465: void spmm_reduce_backward_input_kernel(
```
- EN: The main symbol in this range is `spmm_reduce_arg_kernel`, `spmm_reduce_backward_input_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `spmm_reduce_arg_kernel`, `spmm_reduce_backward_input_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 466-485
```cpp
466:     const Tensor& grad_self,
467:     const Tensor& grad_out,
468:     const Tensor& crow_indices,
469:     const Tensor& col_indices,
470:     const Tensor& other,
471:     const Tensor& row_indices,
472:     ReductionType reduce_op) {
473:   TORCH_CHECK(reduce_op == ReductionType::SUM || reduce_op == ReductionType::MEAN);
474:   AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, other.scalar_type(), "spmm_reduce_backward_input_kernel", [&]() {
475:     AT_DISPATCH_INDEX_TYPES(col_indices.scalar_type(), "spmm_reduce_backward_input_indices", [&]() {
476:       AT_DISPATCH_REDUCTION_TYPES(reduce_op, [&]() {
477:         spmm_reduce_backward_input_kernel_impl<scalar_t, index_t, reduce>(
478:             grad_self, grad_out, crow_indices, col_indices, other, row_indices);
479:       });
480:     });
481:   });
482: }
483:
484: void spmm_reduce_backward_input_arg_kernel(
485:     const Tensor& grad_self,
```
- EN: The main symbol in this range is `spmm_reduce_backward_input_arg_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `spmm_reduce_backward_input_arg_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 486-511
```cpp
486:     const Tensor& grad_out,
487:     const Tensor& col_indices,
488:     const Tensor& other,
489:     const Tensor& arg_out,
490:     ReductionType reduce_op) {
491:   TORCH_CHECK(reduce_op == ReductionType::MAX || reduce_op == ReductionType::MIN);
492:   AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, other.scalar_type(), "spmm_reduce_backward_input_arg_kernel", [&]() {
493:     AT_DISPATCH_INDEX_TYPES(col_indices.scalar_type(), "spmm_reduce_backward_input_arg_indices", [&]() {
494:       spmm_reduce_backward_input_arg_kernel_impl<scalar_t, index_t>(
495:           grad_self, grad_out, col_indices, other, arg_out);
496:     });
497:   });
498: }
499:
500: void spmm_reduce_normalize_values_kernel(
501:     const Tensor& normalized_values,
502:     const Tensor& values,
503:     const Tensor& crow_indices,
504:     const Tensor& row_indices) {
505:   AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, values.scalar_type(), "spmm_reduce_normalize_values_kernel", [&]() {
506:     AT_DISPATCH_INDEX_TYPES(crow_indices.scalar_type(), "spmm_reduce_normalize_values_indices", [&]() {
507:       spmm_reduce_normalize_values_kernel_impl<scalar_t, index_t>(
508:           normalized_values, values, crow_indices, row_indices);
509:     });
510:   });
511: }
```
- EN: The main symbol in this range is `spmm_reduce_normalize_values_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `spmm_reduce_normalize_values_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里处理归一化相关的状态、缩放或统计量。

### Lines 513-535
```cpp
513: void spmm_reduce_backward_other_kernel(
514:     const Tensor& grad_other,
515:     const Tensor& grad_out,
516:     const Tensor& crow_indices,
517:     const Tensor& values,
518:     const Tensor& row_indices,
519:     const Tensor& ccol_indices,
520:     const Tensor& csr2csc,
521:     ReductionType reduce_op) {
522:   TORCH_CHECK(reduce_op == ReductionType::SUM || reduce_op == ReductionType::MEAN);
523:   // need to permute row_indices to CSC order
524:   auto row = row_indices.index_select(0, csr2csc);
525:
526:   Tensor val;
527:   if (reduce_op == ReductionType::MEAN) {
528:     // for reduce type "mean", need to normalize the values
529:     // with rowcount for each of the nonzero element.
530:     Tensor normalized_values = at::empty(values.sizes(), values.options());
531:     spmm_reduce_normalize_values_kernel(normalized_values, values, crow_indices, row_indices);
532:     val = normalized_values.index_select(0, csr2csc);
533:   } else {
534:     val = values.index_select(0, csr2csc);
535:   }
```
- EN: The main symbol in this range is `spmm_reduce_backward_other_kernel`, `spmm_reduce_normalize_values_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `spmm_reduce_backward_other_kernel`, `spmm_reduce_normalize_values_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里处理归一化相关的状态、缩放或统计量。

### Lines 537-556
```cpp
537:   spmm_reduce_kernel(grad_other, ccol_indices, row, val, grad_out, ReductionType::SUM);
538: }
539:
540: void spmm_reduce_backward_other_arg_kernel(
541:     const Tensor& grad_other,
542:     const Tensor& grad_out,
543:     const Tensor& col_indices,
544:     const Tensor& values,
545:     const Tensor& arg_out,
546:     ReductionType reduce_op) {
547:   TORCH_CHECK(reduce_op == ReductionType::MAX || reduce_op == ReductionType::MIN);
548:   AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, values.scalar_type(), "spmm_reduce_backward_other_arg_kernel", [&]() {
549:     AT_DISPATCH_INDEX_TYPES(col_indices.scalar_type(), "spmm_reduce_backward_other_arg_indices", [&]() {
550:       spmm_reduce_backward_other_arg_kernel_impl<scalar_t, index_t>(
551:           grad_other, grad_out, col_indices, values, arg_out);
552:     });
553:   });
554: }
555:
556: } // anonymous namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `spmm_reduce_backward_other_arg_kernel`, `spmm_reduce_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `spmm_reduce_backward_other_arg_kernel`, `spmm_reduce_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 558-565
```cpp
558: REGISTER_DISPATCH(spmm_reduce_stub, &spmm_reduce_kernel)
559: REGISTER_DISPATCH(spmm_reduce_arg_stub, &spmm_reduce_arg_kernel)
560: REGISTER_DISPATCH(spmm_reduce_backward_input_stub, &spmm_reduce_backward_input_kernel)
561: REGISTER_DISPATCH(spmm_reduce_backward_input_arg_stub, &spmm_reduce_backward_input_arg_kernel)
562: REGISTER_DISPATCH(spmm_reduce_backward_other_stub, &spmm_reduce_backward_other_kernel)
563: REGISTER_DISPATCH(spmm_reduce_backward_other_arg_stub, &spmm_reduce_backward_other_arg_kernel)
564:
565: } // at::native
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Normalization statistics / 归一化统计
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/ExpandUtils.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/cpu/vec/functional.h`, `ATen/cpu/vec/vec.h`, `ATen/native/cpu/SpmmReduceKernel.h`, `ATen/native/cpu/ReduceUtils.h`, `ATen/native/cpu/utils.h`, `ATen/OpMathType.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
