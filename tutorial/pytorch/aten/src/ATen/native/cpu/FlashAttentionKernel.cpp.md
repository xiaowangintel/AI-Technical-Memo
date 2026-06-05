# FlashAttentionKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/FlashAttentionKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Flash Attention Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Flash Attention Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3:
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/Parallel.h>
 6: #include <ATen/cpu/vec/vec.h>
 7: #include <ATen/cpu/vec/vec_half.h>
 8: #include <ATen/cpu/vec/functional.h>
 9: #include <ATen/native/CPUBlas.h>
10: #include <ATen/native/cpu/utils.h>
11: #include <ATen/native/transformers/attention.h>
12: #include <ATen/native/transformers/sdp_utils_cpp.h>
13: #include <c10/util/irange.h>
14:
15: #ifndef AT_PER_OPERATOR_HEADERS
16: #include <ATen/Functions.h>
17: #else
18: #include <ATen/ops/empty.h>
19: #endif
20: namespace at::native {
21:
22: namespace {
23:
24: // out = val * a + b
25: // is_b_stride_zero: If the stride of b is 0 (mask broadcasting case),
26: //                take b as a scalar pointer.
27: #if __GNUC__ == 11 && defined(__ARM_FEATURE_SVE)
28: template <typename T1, typename T2>
29: inline void _scale_attn_mask_fusion_kernel(
30:     T1* a,
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 31-60
```cpp
31:     T2* b,
32:     const int& size,
33:     T1* out,
34:     T1& val,
35:     bool is_b_stride_zero) {
36: #else
37: template <bool is_b_stride_zero, typename T1, typename T2>
38: inline void _scale_attn_mask_fusion_kernel(
39:     T1* a,
40:     T2* b,
41:     const int& size,
42:     T1* out,
43:     T1& val) {
44: #endif
45:   const auto vec_size1 = at::vec::Vectorized<T1>::size();
46:   const auto vec_size2 = at::vec::Vectorized<T2>::size();
47:   constexpr int64_t T1_n =
48:       (vec_size2 == vec_size1 * 2 && is_reduced_floating_point_v<T2>) ? 2 : 1;
49:   constexpr int64_t T2_n = 1;
50:   auto vec_scale = at::vec::VectorizedN<T1, T1_n>(val);
51:   int64_t i = 0;
52:   for (; i < size - (size % vec_size2); i += vec_size2) {
53:     auto a_n = at::vec::VectorizedN<T1, T1_n>::loadu(a + i);
54:     at::vec::VectorizedN<T2, T2_n> b_n;
55: #if __GNUC__ == 11 && defined(__ARM_FEATURE_SVE)
56:     if (is_b_stride_zero) {
57: #else
58:     if constexpr(is_b_stride_zero) {
59: #endif
60:       b_n = at::vec::VectorizedN<T2, T2_n>((T1)b[0]);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_scale_attn_mask_fusion_kernel`, `constexpr`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_scale_attn_mask_fusion_kernel`, `constexpr`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 61-98
```cpp
61:     } else {
62:       b_n = at::vec::VectorizedN<T2, T2_n>::loadu(b + i);
63:     }
64:     auto b_n_convert = at::vec::convert<T1, T1_n, T2, T2_n, true>(b_n);
65:     auto res = a_n * vec_scale + b_n_convert;
66:     res.store(out + i);
67:   }
68:   for (; i < size; i++) {
69:     auto tmp0 = a[i];
70:     T1 tmp1;
71: #if __GNUC__ == 11 && defined(__ARM_FEATURE_SVE)
72:     if (is_b_stride_zero) {
73: #else
74:     if constexpr(is_b_stride_zero) {
75: #endif
76:       tmp1 = (T1)b[0];
77:     } else {
78:       tmp1 = (T1)b[i];
79:     }
80:     out[i] = tmp0 * val + tmp1;
81:   }
82: }
83:
84: // 1) out = exp(a - val)
85: // 2) val = sum(out)
86: template <typename T1, typename T2>
87: inline void _exp_reduce_sum_fusion_kernel(
88:     T1* a,
89:     const int& size,
90:     T2* out,
91:     T1& val) {
92:   constexpr auto vec_size1 = vec::Vectorized<T1>::size();
93:   constexpr auto vec_size2 = vec::Vectorized<T2>::size();
94:   constexpr int64_t T1_n =
95:       (vec_size2 == vec_size1 * 2 && is_reduced_floating_point_v<T2>) ? 2 : 1;
96:   constexpr int64_t T2_n = 1;
97:   using Vec = vec::Vectorized<T1>;
98:   using VecN = vec::VectorizedN<T1, T1_n>;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `constexpr`, `_exp_reduce_sum_fusion_kernel`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `constexpr`, `_exp_reduce_sum_fusion_kernel`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 100-128
```cpp
100:   auto vec_max = VecN(val);
101:   auto vec_max_tail = Vec(val);
102:   T1 tmp_sum = 0;
103:   auto vec_tmp_sum = VecN(tmp_sum);
104:   auto vec_tmp_sum_tail = Vec(tmp_sum);
105:   const long vec_end_n = vec_size2 * (size / vec_size2);
106:   const long vec_end = vec_size1 * (size / vec_size1);
107:   auto exp_vec = [](const auto& v) {
108:     if constexpr (
109:         std::is_same_v<T1, float> &&
110:         (std::is_same_v<T2, at::BFloat16> || std::is_same_v<T2, at::Half>)) {
111:       return v.fexp_u20();
112:     } else {
113:       return v.exp_u20();
114:     }
115:   };
116:
117:   long i = 0;
118:   for (; i < vec_end_n; i += vec_size2) {
119:     auto tmp0 = VecN::loadu(a + i);
120:     auto tmp1 = tmp0 - vec_max;
121:     auto tmp2 = exp_vec(tmp1);
122:     vec_tmp_sum = vec_tmp_sum + tmp2;
123:     auto out_n = vec::convert<T2, T2_n, T1, T1_n, true>(tmp2);
124:     out_n.store(out + i);
125:   }
126:   for (; i < vec_end; i += vec_size1) {
127:     auto tmp0 = Vec::loadu(a + i);
128:     auto tmp1 = tmp0 - vec_max_tail;
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 129-158
```cpp
129:     auto tmp2 = exp_vec(tmp1);
130:     vec_tmp_sum_tail = vec_tmp_sum_tail + tmp2;
131:     _store(out + i, tmp2);
132:   }
133:   vec_tmp_sum[0] += vec_tmp_sum_tail;
134:   tmp_sum = vec::vec_reduce_all<T1>(
135:       [](vec::Vectorized<T1>& x, vec::Vectorized<T1>& y) {
136:         return x + y;
137:       },
138:       vec_tmp_sum);
139:   for (long i = vec_end; i < size; i++) {
140:     auto tmp0 = a[i];
141:     auto tmp1 = tmp0 - val;
142:     auto tmp2 = exp(tmp1);
143:     tmp_sum += tmp2;
144:     out[i] = tmp2;
145:   }
146:   val = tmp_sum;
147: }
148:
149: // 1) out = a * scale
150: // 2) max = max(out)
151: template <typename scalar_t>
152: inline void _mul_reduce_max_fusion_kernel(
153:     const scalar_t* a,
154:     const scalar_t& scale,
155:     const int& size,
156:     scalar_t* out,
157:     scalar_t& max) {
158:   using Vec = vec::Vectorized<scalar_t>;
```
- EN: The main symbol in this range is `vec_reduce_all<T1>`, `_mul_reduce_max_fusion_kernel`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `vec_reduce_all<T1>`, `_mul_reduce_max_fusion_kernel`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 159-196
```cpp
159:   using VecN = vec::VectorizedN<scalar_t, 2>;
160:   constexpr auto vec_size = Vec::size();
161:   constexpr auto vec_size_n = VecN::size();
162:   auto vec_scale = VecN(scale);
163:   scalar_t tmp_max = -std::numeric_limits<scalar_t>::infinity();
164:   auto vec_tmp_max = VecN(tmp_max);
165:   auto vec_tmp_max_tail = Vec(tmp_max);
166:   const long vec_end_n = vec_size_n * (size / vec_size_n);
167:   const long vec_end = vec_size * (size / vec_size);
168:   long i = 0;
169:   for (; i < vec_end_n; i += vec_size_n) {
170:     auto tmp0 = VecN::loadu(a + i);
171:     auto tmp1 = tmp0 * vec_scale;
172:     vec_tmp_max = vec::maximum(vec_tmp_max, tmp1);
173:     tmp1.store(out + i);
174:   }
175:   for (; i < vec_end; i += vec_size) {
176:     auto tmp0 = Vec::loadu(a + i);
177:     auto tmp1 = tmp0 * Vec(scale);
178:     vec_tmp_max_tail = vec::maximum(vec_tmp_max_tail, tmp1);
179:     _store(out + i, tmp1);
180:   }
181:   vec_tmp_max[0] = vec::maximum(vec_tmp_max[0], vec_tmp_max_tail);
182:   for (; i < size; i++) {
183:     auto tmp0 = a[i];
184:     auto tmp1 = tmp0 * scale;
185:     tmp_max = std::max(tmp_max, tmp1);
186:     out[i] = tmp1;
187:   }
188:   auto reduced_tmp_max = vec::vec_reduce_all<scalar_t>(
189:       [](vec::Vectorized<scalar_t>& x, vec::Vectorized<scalar_t>& y) {
190:         return vec::maximum(x, y);
191:       },
192:       vec_tmp_max);
193:   // Guard against Q*K^T being NaN
194:   max = std::isnan(reduced_tmp_max) ? std::numeric_limits<scalar_t>::quiet_NaN()
195:                                     : std::max(tmp_max, reduced_tmp_max);
196: }
```
- EN: The main symbol in this range is `vec_reduce_all<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `vec_reduce_all<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 198-226
```cpp
198: template <typename scalar_t>
199: inline scalar_t* conditional_data_ptr(scalar_t* ptr, scalar_t* ptr2) {
200:   TORCH_CHECK(ptr2 == nullptr);
201:   return ptr;
202: }
203:
204: template <typename scalar_t,
205:           typename std::enable_if_t<is_reduced_floating_point_v<scalar_t>, int> = 0>
206: inline scalar_t* conditional_data_ptr(float* ptr, scalar_t* ptr2) {
207:   return ptr2;
208: }
209:
210: template <typename scalar_t>
211: inline void fill_stub(scalar_t* data, scalar_t val, int64_t size) {
212:   using Vec = Vectorized<scalar_t>;
213:   Vec data_vec = Vec(val);
214:   int64_t d = 0;
215:   for (; d < size - (size % Vec::size()); d += Vec::size()) {
216:     data_vec.store(data + d);
217:   }
218:   #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
219:   # pragma unroll
220:   #endif
221:   for (; d < size; d++) {
222:     data[d] = val;
223:   }
224: }
225:
226: void reshape_attn_mask_to_4d(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `conditional_data_ptr`, `fill_stub`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `conditional_data_ptr`, `fill_stub`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 227-256
```cpp
227:     Tensor& attn_mask,
228:     int64_t batchSize,
229:     int64_t num_head,
230:     int64_t qSize,
231:     int64_t kvSize) {
232:   // Support mask shapes:
233:   // 2d: ({Q_seq_len, 1}  x {KV_seq_len, 1})
234:   // 4d: ({Batch, 1} x {Num_heads, 1} x {Q_seq_len, 1}  x {KV_seq_len, 1})
235:   // Guaranteed in check_attn_mask_shape
236:   int64_t attn_mask_size_0 = 1;
237:   int64_t attn_mask_size_1 = 1;
238:   if (attn_mask.dim() == 4) {
239:     if (attn_mask.size(0) == batchSize) {
240:       attn_mask_size_0 = batchSize;
241:     }
242:     if (attn_mask.size(1) == num_head) {
243:       attn_mask_size_1 = num_head;
244:     }
245:   }
246:   attn_mask = attn_mask
247:                 .view({attn_mask_size_0, attn_mask_size_1, attn_mask.size(-2), attn_mask.size(-1)})
248:                 .expand({attn_mask_size_0, attn_mask_size_1, qSize, kvSize});
249:   if (attn_mask.sym_stride(-1) != 1 && attn_mask.sym_stride(-1) != 0) {
250:     attn_mask = attn_mask.contiguous();
251:   }
252: }
253:
254: template <typename scalar_t>
255: inline void copy_value_with_pad(
256:     const scalar_t* value_ptr,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 257-286
```cpp
257:     scalar_t* dst_ptr,
258:     int64_t rows,
259:     int64_t cols,
260:     int64_t prows,
261:     int64_t pcols,
262:     int64_t ldi) {
263:   auto vec_size = at::vec::Vectorized<scalar_t>::size();
264:   int64_t i = 0;
265:   for (; i < rows; i++) {
266:     int64_t j = 0;
267:     for (; j < cols - (cols % vec_size); j += vec_size) {
268:       auto vec_v =
269:           at::vec::Vectorized<scalar_t>::loadu(value_ptr + i * ldi + j);
270:       vec_v.store(dst_ptr + i * pcols + j);
271:     }
272:
273:     if (j < cols) {
274:       auto vec_v = at::vec::Vectorized<scalar_t>::loadu(
275:           value_ptr + i * ldi + j, cols - j);
276:       vec_v.store(dst_ptr + i * pcols + j, cols - j);
277:     }
278:
279:     // col padding
280:     auto psize = pcols - cols;
281:     if (psize > 0) {
282:       auto zero_vec = at::vec::Vectorized<scalar_t>(0);
283:       int64_t pj = 0;
284:       for (; pj < psize - (psize % vec_size); pj += vec_size) {
285:         zero_vec.store(dst_ptr + i * pcols + cols + pj);
286:       }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 287-330
```cpp
287:       if (pj < psize) {
288:         zero_vec.store(dst_ptr + i * pcols + cols + pj, psize - pj);
289:       }
290:     }
291:   }
292:   // row padding
293:   for (; i < prows; i++) {
294:     auto zero_vec = at::vec::Vectorized<scalar_t>(0);
295:     int64_t j = 0;
296:     for (; j < pcols - (pcols % vec_size); j += vec_size) {
297:       zero_vec.store(dst_ptr + i * pcols + j);
298:     }
299:     if (j < pcols) {
300:       zero_vec.store(dst_ptr + i * pcols + j, pcols - j);
301:     }
302:
303:   }
304: }
305:
306: template <typename scalar_t>
307: inline void pad_remain_row_col_zero(
308:     scalar_t* value_ptr,
309:     int rows,
310:     int cols,
311:     int prows,
312:     int pcols,
313:     int ldi) {
314:   auto psize = pcols - cols;
315:   if (psize == 0 && prows == rows) {
316:     return;
317:   }
318:   auto vec_size = at::vec::Vectorized<scalar_t>::size();
319:   auto zero = at::vec::Vectorized<scalar_t>(0);
320:   if (psize > 0) {
321:     for (int i = 0; i < rows; i++) {
322:       int j = 0;
323:       for (; j < psize - (psize % vec_size); j += vec_size) {
324:         zero.store(value_ptr + i * ldi + cols + j);
325:       }
326:       if (j < psize) {
327:         zero.store(value_ptr + i * ldi + cols + j, psize - j);
328:       }
329:     }
330:   }
```
- EN: The main symbol in this range is `pad_remain_row_col_zero`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `pad_remain_row_col_zero`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 332-364
```cpp
332:   for (int i = rows; i < prows; i++) {
333:     int j = 0;
334:     for (; j < pcols - (pcols % vec_size); j += vec_size) {
335:       zero.store(value_ptr + i * ldi + j);
336:     }
337:     if (j < pcols) {
338:       zero.store(value_ptr + i * ldi + j, pcols - j);
339:     }
340:   }
341:
342: }
343:
344:
345: template <typename scalar_t, typename mask_t, int64_t q_split_size, int64_t kv_split_size, bool with_pack=false>
346: void cpu_flash_attention(
347:     const Tensor& output,
348:     const Tensor& logsumexp,
349:     const at::Tensor& q,
350:     const at::Tensor& k,
351:     const at::Tensor& v,
352:     double dropout_p,
353:     bool is_causal,
354:     std::optional<Tensor> attn_mask,
355:     std::optional<double> scale) {
356:   // Query (Batch x Num_heads    x Q_seq_len    x Dim_per_head)
357:   //    -> (Batch x Q_seq_len    x Num_heads    x Dim_per_head)
358:   // Key   (Batch x KV_num_heads x KV_seq_len   x Dim_per_head)
359:   //    -> (Batch x KV_seq_len   x KV_num_heads x Dim_per_head)
360:   // Value (Batch x KV_num_heads x KV_seq_len   x Dim_per_head)
361:   //    -> (Batch x KV_seq_len   x KV_num_heads x Dim_per_head)
362:   at::Tensor query = q.transpose(1, 2);
363:   at::Tensor key = k.transpose(1, 2);
364:   at::Tensor value = v.transpose(1, 2);
```
- EN: The main symbol in this range is `cpu_flash_attention`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_flash_attention`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 366-394
```cpp
366:   constexpr bool is_reduced_type = is_reduced_floating_point_v<scalar_t>;
367:   using accum_t = at::opmath_type<scalar_t>;
368:   using Vec = vec::Vectorized<accum_t>;
369:   accum_t scaling_factor =
370:       sdp::calculate_scale(query, scale).expect_float();
371:
372:   // Sizes
373:   TORCH_CHECK((query.size(3) == value.size(3)) && (key.size(3) == value.size(3)),
374:         "scaled_dot_product_attention_flash_attention: Q/K/V should have the same head size");
375:   int64_t batchSize = query.size(0);
376:   int64_t qSize = query.size(1);
377:   int64_t kvSize = value.size(1);
378:   int64_t num_head = query.size(2);
379:   int64_t kv_num_head = key.size(2);
380:   int64_t repeat_factor = num_head / kv_num_head;
381:   int64_t headSize = query.size(3);
382:
383:   bool has_attn_mask = attn_mask.has_value() && attn_mask.value().numel();
384:   if (has_attn_mask) {
385:     reshape_attn_mask_to_4d(attn_mask.value(), batchSize, num_head, qSize, kvSize);
386:   }
387:
388:   // Strides
389:   int64_t qStrideB = query.stride(0);
390:   int64_t qStrideM = query.stride(1);
391:   int64_t qStrideH = query.stride(2);
392:   int64_t kStrideB = key.stride(0);
393:   int64_t kStrideN = key.stride(1);
394:   int64_t kStrideH = key.stride(2);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 395-426
```cpp
395:   int64_t vStrideB = value.stride(0);
396:   int64_t vStrideN = value.stride(1);
397:   int64_t vStrideH = value.stride(2);
398:   int64_t oStrideB = output.stride(0);
399:   int64_t oStrideM = output.stride(1);
400:   int64_t oStrideH = output.stride(2);
401:   int64_t lStrideB = logsumexp.stride(0);
402:   int64_t lStrideM = logsumexp.stride(1);
403:   int64_t lStrideH = logsumexp.stride(2);
404:   int64_t mStrideB =
405:       (has_attn_mask && attn_mask.value().size(0) > 1)
406:       ? attn_mask.value().stride(0)
407:       : 0;
408:   int64_t mStrideH =
409:       (has_attn_mask && attn_mask.value().size(1) > 1)
410:       ? attn_mask.value().stride(1)
411:       : 0;
412:   int64_t mStrideM =
413:       (has_attn_mask && attn_mask.value().size(2) > 1)
414:       ? attn_mask.value().stride(2)
415:       : 0;
416:   int64_t mStrideN =
417:       (has_attn_mask && attn_mask.value().size(3) > 1)
418:       ? attn_mask.value().stride(3)
419:       : 0;
420:
421:   int64_t qSplitSize = q_split_size > qSize ? qSize : q_split_size;
422:   int64_t kvSplitSize = kv_split_size > kvSize ? kvSize : kv_split_size;
423:   int64_t qSlice = (qSize + qSplitSize - 1) / qSplitSize;
424:   int64_t kvSlice = (kvSize + kvSplitSize - 1) / kvSplitSize;
425:   int64_t kvTail = (kvSize - 1) % kvSplitSize + 1;
426:   int64_t num_thread = at::get_num_threads();
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 428-460
```cpp
428:   const auto dtype = query.scalar_type();
429:   const auto accumulate_dtype = toOpMathType(dtype);
430:
431:   // Whether pack is needed
432:   bool need_pack = false;
433:   if (with_pack) {
434:     // BFloat16 requires larger size as the fallback implementation
435:     // mkl_gemm_bf16bf16f32 is faster than mkl_gemm_f16f16f32
436:     int64_t thresh_size = (dtype == at::ScalarType::BFloat16) ? 64 : 16;
437:     need_pack = kvSize >= thresh_size && qSize >= thresh_size;
438:     // When the number of gemm is greater than the number of pack,
439:     // the pack overhead can be overlapped.
440:     if (need_pack) {
441:       double pack_size = batchSize * kv_num_head * kvSize * headSize;
442:       double qs_per_thread = (batchSize * num_head * qSlice + num_thread - 1) / num_thread;
443:       double gemm_size_per_thread = qs_per_thread * qSplitSize *
444:           (is_causal ? std::min(qSize, kvSize) : kvSize) * headSize;
445:       need_pack = gemm_size_per_thread / pack_size >= (dtype == at::ScalarType::BFloat16 ? 4 : 1);
446:     }
447:   }
448:
449:   // Pad is needed for packing when K is not even
450:   bool headSize_even = headSize % 2 == 0;
451:   int64_t eheadSize = need_pack && !headSize_even ? headSize + 1: headSize;
452:   int64_t ekvSplitSize = need_pack && (kvSplitSize % 2 != 0) ? kvSplitSize + 1 : kvSplitSize;
453:   int64_t ekvTail = need_pack && (kvTail % 2 != 0) ? kvTail + 1 : kvTail;
454:
455:   // Allocate per thread temp buf (accumulate type)
456:   int64_t size_per_thread =
457:       /* qk     */ qSplitSize * kvSplitSize +
458:       /* qk_max */ qSplitSize +
459:       /* qk_sum */ qSplitSize +
460:       /* dst    */ qSplitSize * headSize;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 462-498
```cpp
462:   at::Tensor buf = at::empty({num_thread, size_per_thread}, query.options().dtype(accumulate_dtype));
463:   at::Tensor buf_reduced = at::empty(
464:     {num_thread,
465:      qSplitSize,
466:      is_reduced_type ? ekvSplitSize : 0},
467:      query.options());
468:
469:   // Data ptrs
470:   const scalar_t* q_data = query.const_data_ptr<scalar_t>();
471:   const scalar_t* k_data = key.const_data_ptr<scalar_t>();
472:   const scalar_t* v_data = value.const_data_ptr<scalar_t>();
473:   mask_t* mask_data = has_attn_mask
474:       ? attn_mask.value().data_ptr<mask_t>()
475:       : nullptr;
476:   scalar_t* out_data = output.data_ptr<scalar_t>();
477:   accum_t* lse_data = logsumexp.data_ptr<accum_t>();
478:   accum_t* buf_data = buf.data_ptr<accum_t>();
479:   scalar_t* buf_reduced_data = is_reduced_type ? buf_reduced.data_ptr<scalar_t>() : nullptr;
480:
481:   // Buffer to store padding query and packing key/value
482:   scalar_t* key_reorder_ptr = nullptr;
483:   scalar_t* value_reorder_ptr = nullptr;
484:   scalar_t* query_padding_ptr = nullptr;
485:   int64_t kv_padding_size = (kvSize - 1) / kvSplitSize * ekvSplitSize + ekvTail;
486:   at::Tensor key_t_reorder;
487:   at::Tensor value_t_reorder;
488:   at::Tensor qeury_t_padding;
489:   if (need_pack) {
490:     key_t_reorder = at::empty(
491:       {batchSize, kv_num_head, eheadSize, kvSize},
492:       c10::CppTypeToScalarType<scalar_t>::value);
493:     value_t_reorder = at::empty(
494:       {batchSize, kv_num_head, kv_padding_size, headSize},
495:       c10::CppTypeToScalarType<scalar_t>::value);
496:     key_reorder_ptr = key_t_reorder.data_ptr<scalar_t>();
497:     value_reorder_ptr = value_t_reorder.data_ptr<scalar_t>();
498:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 500-530
```cpp
500:   if (!headSize_even && need_pack) {
501:     qeury_t_padding = at::empty(
502:       {num_thread, qSplitSize, eheadSize},
503:       c10::CppTypeToScalarType<scalar_t>::value);
504:     query_padding_ptr = qeury_t_padding.data_ptr<scalar_t>();
505:   }
506:
507:   // Reorder K, V
508:   if (need_pack) {
509:     at::Tensor tranpose_t_reorder = at::empty(
510:       {num_thread, kvSplitSize, headSize},
511:       c10::CppTypeToScalarType<scalar_t>::value);
512:     scalar_t* transpose_buffer_ptr = tranpose_t_reorder.data_ptr<scalar_t>();
513:     at::parallel_for(0, batchSize * kv_num_head * kvSlice, 1, [&](int64_t begin, int64_t end) {
514:         int ompIdx = at::get_thread_num();
515:         int64_t i = 0, kv_j = 0, l = 0, n = 0;
516:         scalar_t* transpose_ptr = transpose_buffer_ptr + ompIdx * kvSplitSize * headSize;
517:         at::native::data_index_init(begin, i, batchSize, kv_j, kv_num_head, l, kvSlice);
518:         for ([[maybe_unused]] auto z : c10::irange(begin, end)) {
519:           n = l * kvSplitSize;
520:           int64_t kvBlockSize = std::min(kvSplitSize, kvSize - n);
521:
522:           // transpose [kvBlockSize, headSize] -> [headSize, kvBlockSize]
523:           utils::transpose<uint16_t>(
524:               kvBlockSize,
525:               headSize,
526:               /* src_ptr */
527:               reinterpret_cast<const uint16_t*>(k_data + i * kStrideB + kv_j * kStrideH + n * kStrideN),
528:               /* ld_src */ kStrideN,
529:               /* dst */ reinterpret_cast<uint16_t*>(transpose_ptr),
530:               /* ld_dst */ kvBlockSize);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 532-569
```cpp
532:           // Pack [headSize, kvBlockSize]
533:           at::vec::pack_vnni2(
534:             /* src */ reinterpret_cast<const uint16_t*>(transpose_ptr),
535:             /* dst */ reinterpret_cast<uint16_t*>(key_reorder_ptr + i * kv_num_head * eheadSize * kvSize +
536:                     kv_j * eheadSize * kvSize + n * eheadSize),
537:             /* ld_src */ kvBlockSize,
538:             /* K */ headSize,
539:             /* N */ kvBlockSize);
540:
541:           // Pack [kvBlockSize, headSize]
542:           at::vec::pack_vnni2(
543:             /* src */ reinterpret_cast<const uint16_t*>(v_data + i * vStrideB + kv_j * vStrideH + n * vStrideN),
544:             /* dst */ reinterpret_cast<uint16_t*>(value_reorder_ptr +
545:                     i * kv_num_head * kv_padding_size * headSize +
546:                     kv_j * kv_padding_size * headSize + n * headSize),
547:             /* ld_src */ vStrideN,
548:             /* K */ kvBlockSize,
549:             /* N */ headSize);
550:
551:           // Move to the next query
552:           at::native::data_index_step(i, batchSize, kv_j, kv_num_head, l, kvSlice);
553:         }
554:       });
555:   }
556:
557:   at::parallel_for(0, batchSize * num_head * qSlice, 1, [&](int64_t begin, int64_t end) {
558:     int64_t i = 0, j = 0, k = 0;
559:     data_index_init(begin, i, batchSize, j, num_head, k, qSlice);
560:     int ompIdx = at::get_thread_num();
561:     accum_t* buf_ptr = buf_data + ompIdx * size_per_thread;
562:     accum_t* qk_data = buf_ptr;
563:     accum_t* qk_max_data = qk_data + qSplitSize * kvSplitSize;
564:     accum_t* qk_sum_data = qk_max_data + qSplitSize;
565:     accum_t* dst_data = qk_sum_data + qSplitSize;
566:     scalar_t* qk_reduced_data = is_reduced_type ? buf_reduced_data + ompIdx * qSplitSize * ekvSplitSize : nullptr;
567:     scalar_t* query_t_padding_ptr = (!headSize_even && need_pack)
568:             ? query_padding_ptr + ompIdx * qSplitSize * eheadSize
569:             : nullptr;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 571-599
```cpp
571:     for ([[maybe_unused]] auto z : c10::irange(begin, end)) {
572:       int64_t m = k * qSplitSize;
573:       int64_t qBlockSize = std::min(qSplitSize, qSize - m);
574:       int64_t kv_j = j / repeat_factor;
575:       // Initialize max and sum
576:       fill_stub(qk_max_data,
577:           -std::numeric_limits<accum_t>::infinity(), qBlockSize);
578:       fill_stub(qk_sum_data,
579:           static_cast<accum_t>(0), qBlockSize);
580:       int64_t num_keys = is_causal ? std::min(m + qBlockSize, kvSize) : kvSize;
581:       if (!headSize_even && need_pack) {
582:         // Pad query if headSize is not even
583:         // [qBlockSize, headSize] -> [qBlockSize, eheadSize]
584:         copy_value_with_pad<scalar_t>(
585:           q_data + i * qStrideB + j * qStrideH + m * qStrideM,
586:           query_t_padding_ptr,
587:           qBlockSize,
588:           headSize,
589:           qBlockSize,
590:           eheadSize,
591:           qStrideM
592:         );
593:       }
594:       for (int64_t n = 0; n < num_keys; n += kvSplitSize) {
595:         int64_t kvBlockSize = std::min(kvSplitSize, kvSize - n);
596:         int64_t ekvBlockSize = (need_pack && kvBlockSize % 2 != 0) ? kvBlockSize + 1 : kvBlockSize;
597:         // Calculate scale * q @ k.T
598:         if (need_pack) {
599:           if constexpr (is_reduced_floating_point_v<scalar_t>) {
```
- EN: The main symbol in this range is `constexpr`, `fill_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `constexpr`, `fill_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 600-629
```cpp
600:             cpublas::brgemm(
601:                 qBlockSize,
602:                 kvBlockSize,
603:                 eheadSize,
604:                 headSize_even ? qStrideM : eheadSize,
605:                 kvBlockSize,
606:                 kvBlockSize,
607:                 false,
608:                 !headSize_even
609:                     ? query_t_padding_ptr
610:                     : q_data + i * qStrideB + j * qStrideH + m * qStrideM,
611:                 key_reorder_ptr + i * kv_num_head * eheadSize * kvSize +
612:                     kv_j * eheadSize * kvSize + n * eheadSize,
613:                 qk_data);
614:           }
615:         } else {
616:           cpublas::gemm(
617:             TransposeType::Transpose,
618:             TransposeType::NoTranspose,
619:             kvBlockSize,
620:             qBlockSize,
621:             headSize,
622:             static_cast<accum_t>(1),
623:             k_data + i * kStrideB + kv_j * kStrideH +
624:                 n * kStrideN,
625:             kStrideN,
626:             q_data + i * qStrideB + j * qStrideH +
627:                 m * qStrideM,
628:             qStrideM,
629:             static_cast<accum_t>(0),
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 630-659
```cpp
630:             qk_data,
631:             kvBlockSize);
632:         }
633:         // Apply causal mask, fill unused with -inf
634:         if (is_causal && num_keys - n <= kvSplitSize) {
635:           for (const auto row : c10::irange(qBlockSize)) {
636:             int64_t last_col = m + row - n;
637:             accum_t* row_ptr = qk_data + row * kvBlockSize;
638:             fill_stub(row_ptr + last_col + 1,
639:                 -std::numeric_limits<accum_t>::infinity(),
640:                 kvBlockSize - last_col - 1);
641:           }
642:         }
643:         // Update attention weights with attention mask
644:         // And apply scaling factor
645:         // qk <- qk * scaling + attn_mask
646:         if (has_attn_mask) {
647:           for (int64_t row = 0; row < qBlockSize; ++row) {
648: #if __GNUC__ == 11 && defined(__ARM_FEATURE_SVE)
649:               _scale_attn_mask_fusion_kernel(
650:                 qk_data + row * kvBlockSize,
651:                 mask_data + i * mStrideB + j * mStrideH +
652:                     (m + row) * mStrideM + (mStrideN == 0 ? 0 : n),
653:                 kvBlockSize,
654:                 qk_data + row * kvBlockSize,
655:                 scaling_factor,
656:                 mStrideN == 0);
657: #else
658:               if (mStrideN == 0) {
659:                 _scale_attn_mask_fusion_kernel</*is_stride_0*/ true>(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `fill_stub`, `_scale_attn_mask_fusion_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `fill_stub`, `_scale_attn_mask_fusion_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 660-689
```cpp
660:                   qk_data + row * kvBlockSize,
661:                   mask_data + i * mStrideB + j * mStrideH +
662:                       (m + row) * mStrideM,
663:                   kvBlockSize,
664:                   qk_data + row * kvBlockSize,
665:                   scaling_factor);
666:               } else {
667:                 _scale_attn_mask_fusion_kernel</*is_stride_0*/ false>(
668:                   qk_data + row * kvBlockSize,
669:                   mask_data + i * mStrideB + j * mStrideH +
670:                       (m + row) * mStrideM + n,
671:                   kvBlockSize,
672:                   qk_data + row * kvBlockSize,
673:                   scaling_factor);
674:               }
675: #endif
676:           }
677:         }
678:         // Update coefficients with Softmax
679:         accum_t tmp_max = 0, tmp_sum = 0, exp_tmp = 0;
680:         for (int64_t row = 0; row < qBlockSize; ++row) {
681:           if (has_attn_mask) {
682:             // max per row
683:             tmp_max = at::vec::reduce_all<accum_t>(
684:                 [](Vec& x, Vec& y) { return at::vec::maximum(x, y); },
685:                 qk_data + row * kvBlockSize,
686:                 kvBlockSize);
687:           } else {
688:             // apply scaling factor and max per row in fusion
689:             _mul_reduce_max_fusion_kernel(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `reduce_all<accum_t>`, `_mul_reduce_max_fusion_kernel`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `reduce_all<accum_t>`, `_mul_reduce_max_fusion_kernel`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 690-719
```cpp
690:                 qk_data + row * kvBlockSize,
691:                 scaling_factor,
692:                 kvBlockSize,
693:                 qk_data + row * kvBlockSize,
694:                 tmp_max);
695:           }
696:           tmp_max = qk_max_data[row] > tmp_max ? qk_max_data[row] : tmp_max;
697:           if (tmp_max == -std::numeric_limits<accum_t>::infinity()) {
698:             // to avoid `nan = exp2f(-inf - (-inf))`
699:             fill_stub(conditional_data_ptr(qk_data, qk_reduced_data) + row * ekvBlockSize,
700:               static_cast<scalar_t>(0), kvBlockSize);
701:           } else {
702:             tmp_sum = tmp_max;
703:             // qk <- exp(qk - max) and sum per row
704:             _exp_reduce_sum_fusion_kernel(
705:                 qk_data + row * kvBlockSize, kvBlockSize,
706:                 conditional_data_ptr(qk_data, qk_reduced_data) + row * ekvBlockSize,
707:                 tmp_sum);
708:             // exp_tmp <- exp(max[row] - max)
709:             exp_tmp = std::exp(qk_max_data[row] - tmp_max);
710:             // sum[row] <- sum + exp_tmp * sum[row]
711:             qk_sum_data[row] = tmp_sum + exp_tmp * qk_sum_data[row];
712:             // max[row] <- max
713:             qk_max_data[row] = tmp_max;
714:             // dst <- dst * exp_tmp
715:             if (n > 0) {
716:               vec::map<accum_t>(
717:                 [exp_tmp](Vec x) { return x * Vec(exp_tmp); },
718:                 dst_data + row * headSize,
719:                 dst_data + row * headSize,
```
- EN: The main symbol in this range is `map<accum_t>`, `fill_stub`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `map<accum_t>`, `fill_stub`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 720-763
```cpp
720:                 headSize);
721:             }
722:           }
723:           if (need_pack && kvBlockSize % 2 != 0) {
724:             // Pad: [qSplitSize, kvBlockSize] -> [qSplitSize, kvBlockSize + 1]
725:             *(qk_reduced_data + row * (1 + kvBlockSize) + kvBlockSize) = scalar_t(0);
726:           }
727:         }
728:         // Calculate Softmax(q @ k.T) @ v
729:         if (need_pack) {
730:           int64_t psize = n / kvSplitSize * ekvSplitSize;
731:           if constexpr (is_reduced_floating_point_v<scalar_t>) {
732:             cpublas::brgemm(
733:                   qBlockSize,
734:                   headSize,
735:                   ekvBlockSize,
736:                   ekvBlockSize,
737:                   headSize,
738:                   headSize,
739:                   n > 0,
740:                   qk_reduced_data,
741:                   value_reorder_ptr +
742:                       i * kv_num_head * kv_padding_size * headSize +
743:                       kv_j * kv_padding_size * headSize + psize * headSize,
744:                   dst_data);
745:           }
746:         } else {
747:           cpublas::gemm(
748:             TransposeType::NoTranspose,
749:             TransposeType::NoTranspose,
750:             headSize,
751:             qBlockSize,
752:             kvBlockSize,
753:             static_cast<accum_t>(1),
754:             v_data + i * vStrideB + kv_j * vStrideH +
755:                 n * vStrideN,
756:             vStrideN,
757:             conditional_data_ptr(qk_data, qk_reduced_data),
758:             kvBlockSize,
759:             n == 0 ? static_cast<accum_t>(0) : static_cast<accum_t>(1),
760:             dst_data,
761:             headSize);
762:         }
763:       }
```
- EN: The main symbol in this range is `Softmax`, `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `Softmax`, `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 765-793
```cpp
765:       // dst <- dst / sum[row]
766:       // reorder MHA output with strides
767:       for (int64_t row = 0; row < qBlockSize; ++row) {
768:         // Row sums for full masked out rows are 0, we set them to 1
769:         // in order to avoid NaNs in the output and instead set fully
770:         // masked out rows to 0
771:         qk_max_data[row] = qk_max_data[row] == -std::numeric_limits<accum_t>::infinity() ? 0 : qk_max_data[row];
772:         qk_sum_data[row] = qk_sum_data[row] == 0 ? 1 : qk_sum_data[row];
773:         accum_t sum_reciprocal = 1 / qk_sum_data[row];
774:         vec::map<scalar_t>(
775:           [sum_reciprocal](Vec x) { return x * Vec(sum_reciprocal); },
776:           out_data + i * oStrideB + j * oStrideH + m * oStrideM + row * oStrideM,
777:           dst_data + row * headSize,
778:           headSize);
779:       }
780:       // Store logsumexp for backward
781:       accum_t* lse_ptr = lse_data + i * lStrideB + j * lStrideH + m * lStrideM;
782:       for (const auto row : c10::irange(qBlockSize)) {
783:         lse_ptr[row * lStrideM] = qk_max_data[row]
784:             + std::log(qk_sum_data[row]);
785:       }
786:       // Move to the next query
787:       data_index_step(i, batchSize, j, num_head, k, qSlice);
788:     }
789:     if (need_pack) {
790:       cpublas::brgemm_release();
791:     }
792:   });
793: }
```
- EN: The main symbol in this range is `map<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `map<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 795-828
```cpp
795: template <typename scalar_t, typename mask_t, int64_t q_split_size, int64_t kv_split_size>
796: void cpu_flash_attention_backward(
797:     const at::Tensor& grad_q,
798:     const at::Tensor& grad_k,
799:     const at::Tensor& grad_v,
800:     const at::Tensor& grad_out,
801:     const at::Tensor& query,
802:     const at::Tensor& key,
803:     const at::Tensor& value,
804:     const at::Tensor& out,
805:     const at::Tensor& logsumexp,
806:     double dropout_p,
807:     bool is_causal,
808:     std::optional<Tensor> attn_mask,
809:     std::optional<double> scale) {
810:   constexpr bool is_reduced_type = is_reduced_floating_point_v<scalar_t>;
811:   using accum_t = at::opmath_type<scalar_t>;
812:   using Vec = vec::Vectorized<accum_t>;
813:   accum_t scaling_factor =
814:       sdp::calculate_scale(query, scale).expect_float();
815:
816:   // Sizes
817:   TORCH_CHECK((query.size(3) == value.size(3)) && (key.size(3) == value.size(3)),
818:         "scaled_dot_product_attention_flash_attention_backward: Q/K/V should have the same head size");
819:   // Query (Batch x Q_seq_len  x Num_heads    x Dim_per_head)
820:   // Key   (Batch x KV_seq_len x KV_num_heads x Dim_per_head)
821:   // Value (Batch x KV_seq_len x KV_num_heads x Dim_per_head)
822:   int64_t batchSize = query.size(0);
823:   int64_t qSize = query.size(1);
824:   int64_t kvSize = value.size(1);
825:   int64_t num_head = query.size(2);
826:   int64_t kv_num_head = key.size(2);
827:   int64_t repeat_factor = num_head / kv_num_head;
828:   int64_t headSize = query.size(3);
```
- EN: The main symbol in this range is `cpu_flash_attention_backward`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `cpu_flash_attention_backward`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 830-866
```cpp
830:   bool has_attn_mask = attn_mask.has_value() && attn_mask.value().numel();
831:   if (has_attn_mask) {
832:     reshape_attn_mask_to_4d(attn_mask.value(), batchSize, num_head, qSize, kvSize);
833:   }
834:
835:   // Strides
836:   int64_t qStrideB = query.stride(0);
837:   int64_t qStrideM = query.stride(1);
838:   int64_t qStrideH = query.stride(2);
839:   int64_t kStrideB = key.stride(0);
840:   int64_t kStrideN = key.stride(1);
841:   int64_t kStrideH = key.stride(2);
842:   int64_t vStrideB = value.stride(0);
843:   int64_t vStrideN = value.stride(1);
844:   int64_t vStrideH = value.stride(2);
845:   int64_t oStrideB = out.stride(0);
846:   int64_t oStrideM = out.stride(1);
847:   int64_t oStrideH = out.stride(2);
848:   int64_t lStrideB = logsumexp.stride(0);
849:   int64_t lStrideM = logsumexp.stride(1);
850:   int64_t lStrideH = logsumexp.stride(2);
851:   int64_t mStrideB =
852:       (has_attn_mask && attn_mask.value().size(0) > 1)
853:       ? attn_mask.value().stride(0)
854:       : 0;
855:   int64_t mStrideH =
856:       (has_attn_mask && attn_mask.value().size(1) > 1)
857:       ? attn_mask.value().stride(1)
858:       : 0;
859:   int64_t mStrideM =
860:       (has_attn_mask && attn_mask.value().size(2) > 1)
861:       ? attn_mask.value().stride(2)
862:       : 0;
863:   int64_t mStrideN =
864:       (has_attn_mask && attn_mask.value().size(3) > 1)
865:       ? attn_mask.value().stride(3)
866:       : 0;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 868-899
```cpp
868:   int64_t grad_qStrideB = grad_q.stride(0);
869:   int64_t grad_qStrideM = grad_q.stride(1);
870:   int64_t grad_qStrideH = grad_q.stride(2);
871:   int64_t grad_kStrideB = grad_k.stride(0);
872:   int64_t grad_kStrideN = grad_k.stride(1);
873:   int64_t grad_kStrideH = grad_k.stride(2);
874:   int64_t grad_vStrideB = grad_v.stride(0);
875:   int64_t grad_vStrideN = grad_v.stride(1);
876:   int64_t grad_vStrideH = grad_v.stride(2);
877:   int64_t grad_oStrideB = grad_out.stride(0);
878:   int64_t grad_oStrideM = grad_out.stride(1);
879:   int64_t grad_oStrideH = grad_out.stride(2);
880:
881:   int64_t qSplitSize = q_split_size > qSize ? qSize : q_split_size;
882:   int64_t kvSplitSize = kv_split_size > kvSize ? kvSize : kv_split_size;
883:   int64_t num_thread = at::get_num_threads();
884:
885:   const auto dtype = query.scalar_type();
886:   const auto accumulate_dtype = toOpMathType(dtype);
887:
888:   // allocate per thread temp buf (accumulate type)
889:   int64_t size_per_thread =
890:       /* attn      */ qSplitSize * kvSplitSize +
891:       /* grad_attn */ qSplitSize * kvSplitSize;
892:
893:   at::Tensor buf = at::empty({num_thread, size_per_thread}, query.options().dtype(accumulate_dtype));
894:
895:   // allocate per thread temp buf_reduced (scalar type)
896:   // buf2 is only needed for bfloat16 and float16
897:   int64_t size_per_thread_reduced =
898:       /* attn_reduced      */ qSplitSize * kvSplitSize +
899:       /* grad_attn_reduced */ qSplitSize * kvSplitSize;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 901-929
```cpp
901:   at::Tensor buf_reduced = at::empty({num_thread, is_reduced_type ? size_per_thread_reduced : 0}, query.options());
902:
903:   scalar_t* grad_q_data = grad_q.data_ptr<scalar_t>();
904:   scalar_t* grad_k_data = grad_k.data_ptr<scalar_t>();
905:   scalar_t* grad_v_data = grad_v.data_ptr<scalar_t>();
906:   const scalar_t* grad_out_data = grad_out.const_data_ptr<scalar_t>();
907:   const scalar_t* q_data = query.const_data_ptr<scalar_t>();
908:   const scalar_t* k_data = key.const_data_ptr<scalar_t>();
909:   const scalar_t* v_data = value.const_data_ptr<scalar_t>();
910:   mask_t* mask_data = has_attn_mask
911:       ? attn_mask.value().data_ptr<mask_t>()
912:       : nullptr;
913:   const scalar_t* out_data = out.const_data_ptr<scalar_t>();
914:   const accum_t* lse_data = logsumexp.const_data_ptr<accum_t>();
915:   accum_t* buf_data = buf.data_ptr<accum_t>();
916:   scalar_t* buf_reduced_data = is_reduced_type ? buf_reduced.data_ptr<scalar_t>() : nullptr;
917:
918:   at::parallel_for(0, batchSize * kv_num_head, 1, [&](int64_t begin, int64_t end) {
919:     int64_t i = 0, kv_j = 0;
920:     data_index_init(begin, i, batchSize, kv_j, kv_num_head);
921:     int ompIdx = at::get_thread_num();
922:     accum_t* buf_ptr = buf_data + ompIdx * size_per_thread;
923:     accum_t* attn_data = buf_ptr;
924:     accum_t* grad_attn_data = attn_data + qSplitSize * kvSplitSize;
925:     scalar_t* buf_reduced_ptr = is_reduced_type ? buf_reduced_data + ompIdx * size_per_thread_reduced : nullptr;
926:     scalar_t* attn_reduced_data = is_reduced_type ? buf_reduced_ptr : nullptr;
927:     scalar_t* grad_attn_reduced_data = is_reduced_type ? attn_reduced_data + qSplitSize * kvSplitSize : nullptr;
928:
929:     at::Tensor dsum = at::empty({qSplitSize}, query.options().dtype(accumulate_dtype));
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 930-959
```cpp
930:     accum_t* dsum_data = dsum.data_ptr<accum_t>();
931:     for ([[maybe_unused]] auto z : c10::irange(begin, end)) {
932:       for (int64_t r = 0; r < repeat_factor; r++) {
933:         int64_t j = kv_j * repeat_factor + r;
934:         // rowsum of grad_out * out
935:         for (int64_t m = 0; m < qSize; m += qSplitSize) {
936:           int64_t qBlockSize = std::min(qSplitSize, qSize - m);
937:           // dsum <- rowsum(grad_out * out)
938:           for (const auto row : c10::irange(qBlockSize)) {
939:             *(dsum_data + row) = vec::map2_reduce_all<scalar_t>(
940:               [](Vec x, Vec y) { return x * y; },
941:               [](Vec x, Vec y) { return x + y; },
942:               grad_out_data + i * grad_oStrideB + j * grad_oStrideH + (m + row) * grad_oStrideM,
943:               out_data + i * oStrideB + j * oStrideH + (m + row) * oStrideM,
944:               headSize);
945:           }
946:           int64_t num_keys = is_causal ? std::min(m + qBlockSize, kvSize) : kvSize;
947:           for (int64_t n = 0; n < num_keys; n += kvSplitSize) {
948:             int64_t kvBlockSize = std::min(kvSplitSize, kvSize - n);
949:             // attn <- scale * q @ k.T
950:             cpublas::gemm(
951:               TransposeType::Transpose,
952:               TransposeType::NoTranspose,
953:               kvBlockSize,
954:               qBlockSize,
955:               headSize,
956:               scaling_factor,
957:               k_data + i * kStrideB + kv_j * kStrideH +
958:                   n * kStrideN,
959:               kStrideN,
```
- EN: The main symbol in this range is `rowsum`, `map2_reduce_all<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `rowsum`, `map2_reduce_all<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 960-989
```cpp
960:               q_data + i * qStrideB + j * qStrideH +
961:                   m * qStrideM,
962:               qStrideM,
963:               static_cast<accum_t>(0),
964:               attn_data,
965:               kvBlockSize);
966:             // attn <- attn + mask
967:             if (has_attn_mask) {
968:               accum_t one = accum_t(1);
969:               for (const auto row : c10::irange(qBlockSize)) {
970:   #if __GNUC__ == 11 && defined(__ARM_FEATURE_SVE)
971:                   _scale_attn_mask_fusion_kernel(
972:                     attn_data + row * kvBlockSize,
973:                     mask_data + i * mStrideB + j * mStrideH +
974:                         (m + row) * mStrideM + (mStrideN == 0 ? 0 : n),
975:                     kvBlockSize,
976:                     attn_data + row * kvBlockSize,
977:                     one,
978:                     mStrideN == 0);
979:   #else
980:                   if (mStrideN == 0) {
981:                     _scale_attn_mask_fusion_kernel</*is_stride_0*/ true>(
982:                       attn_data + row * kvBlockSize,
983:                       mask_data + i * mStrideB + j * mStrideH +
984:                           (m + row) * mStrideM,
985:                       kvBlockSize,
986:                       attn_data + row * kvBlockSize,
987:                       one);
988:                   } else {
989:                     _scale_attn_mask_fusion_kernel</*is_stride_0*/ false>(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_scale_attn_mask_fusion_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_scale_attn_mask_fusion_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 990-1019
```cpp
 990:                       attn_data + row * kvBlockSize,
 991:                       mask_data + i * mStrideB + j * mStrideH +
 992:                           (m + row) * mStrideM + n,
 993:                       kvBlockSize,
 994:                       attn_data + row * kvBlockSize,
 995:                       one);
 996:                   }
 997:   #endif
 998:               }
 999:             }
1000:             // restore self attention after softmax from logsumexp
1001:             // attn <- exp(attn - normalizer)
1002:             for (const auto row : c10::irange(qBlockSize)) {
1003:               accum_t normalizer = lse_data[i * lStrideB + j * lStrideH + (m + row) * lStrideM];
1004:               vec::map<accum_t>(
1005:                 [normalizer](Vec x) { return (x - Vec(normalizer)).exp(); },
1006:                 attn_data + row * kvBlockSize,
1007:                 attn_data + row * kvBlockSize,
1008:                 kvBlockSize);
1009:             }
1010:             // Apply causal mask, filled unused with 0
1011:             if (is_causal && num_keys - n <= kvSplitSize) {
1012:               for (const auto row : c10::irange(qBlockSize)) {
1013:                 int64_t last_col = m + row - n;
1014:                 accum_t* row_ptr = attn_data + row * kvBlockSize;
1015:                 fill_stub(row_ptr + last_col + 1, static_cast<accum_t>(0), kvBlockSize - last_col - 1);
1016:               }
1017:             }
1018:   #ifdef _MSC_VER
1019:             if (is_reduced_type) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `map<accum_t>`, `fill_stub`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `map<accum_t>`, `fill_stub`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 1020-1049
```cpp
1020:   #else
1021:             if constexpr (is_reduced_type) {
1022:   #endif
1023:               for (const auto row : c10::irange(qBlockSize)) {
1024:                 convert<accum_t, scalar_t>(
1025:                   attn_data + row * kvBlockSize,
1026:                   attn_reduced_data + row * kvBlockSize,
1027:                   kvBlockSize);
1028:               }
1029:             }
1030:             // grad_v <- grad_v + attn.T @ grad_out
1031:             cpublas::gemm(
1032:               TransposeType::NoTranspose,
1033:               TransposeType::Transpose,
1034:               headSize,
1035:               kvBlockSize,
1036:               qBlockSize,
1037:               static_cast<accum_t>(1),
1038:               grad_out_data + i * grad_oStrideB + j * grad_oStrideH +
1039:                   m * grad_oStrideM,
1040:               grad_oStrideM,
1041:               conditional_data_ptr(attn_data, attn_reduced_data),
1042:               kvBlockSize,
1043:               static_cast<accum_t>(1),
1044:               grad_v_data + i * grad_vStrideB + kv_j * grad_vStrideH +
1045:                   n * grad_vStrideN,
1046:               grad_vStrideN);
1047:             // grad_attn <- grad_out @ v.T
1048:             cpublas::gemm(
1049:               TransposeType::Transpose,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1050-1079
```cpp
1050:               TransposeType::NoTranspose,
1051:               kvBlockSize,
1052:               qBlockSize,
1053:               headSize,
1054:               static_cast<accum_t>(1),
1055:               v_data + i * vStrideB + kv_j * vStrideH +
1056:                   n * vStrideN,
1057:               vStrideN,
1058:               grad_out_data + i * grad_oStrideB + j * grad_oStrideH +
1059:                   m * grad_oStrideM,
1060:               grad_oStrideM,
1061:               static_cast<accum_t>(0),
1062:               grad_attn_data,
1063:               kvBlockSize);
1064:             // grad_attn <- attn * (grad_attn - dsum)
1065:             for (const auto row : c10::irange(qBlockSize)) {
1066:               accum_t d = *(dsum_data + row);
1067:               vec::map2<accum_t>(
1068:                 [d](Vec attn, Vec grad_attn) { return attn * (grad_attn - Vec(d)); },
1069:                 grad_attn_data + row * kvBlockSize,
1070:                 attn_data + row * kvBlockSize,
1071:                 grad_attn_data + row * kvBlockSize,
1072:                 kvBlockSize);
1073:             }
1074:   #ifdef _MSC_VER
1075:             if (is_reduced_type) {
1076:   #else
1077:             if constexpr (is_reduced_type) {
1078:   #endif
1079:               for (const auto row : c10::irange(qBlockSize)) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `map2<accum_t>`, `constexpr`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `map2<accum_t>`, `constexpr`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 1080-1109
```cpp
1080:                 convert<accum_t, scalar_t>(
1081:                   grad_attn_data + row * kvBlockSize,
1082:                   grad_attn_reduced_data + row * kvBlockSize,
1083:                   kvBlockSize);
1084:               }
1085:             }
1086:             // grad_q <- grad_q + scale * grad_attn @ k
1087:             cpublas::gemm(
1088:               TransposeType::NoTranspose,
1089:               TransposeType::NoTranspose,
1090:               headSize,
1091:               qBlockSize,
1092:               kvBlockSize,
1093:               scaling_factor,
1094:               k_data + i * kStrideB + kv_j * kStrideH +
1095:                   n * kStrideN,
1096:               kStrideN,
1097:               conditional_data_ptr(grad_attn_data, grad_attn_reduced_data),
1098:               kvBlockSize,
1099:               static_cast<accum_t>(1),
1100:               grad_q_data + i * grad_qStrideB + j * grad_qStrideH +
1101:                   m * grad_qStrideM,
1102:               grad_qStrideM);
1103:             // grad_k <- grad_k + scale * grad_attn.T @ q
1104:             cpublas::gemm(
1105:               TransposeType::NoTranspose,
1106:               TransposeType::Transpose,
1107:               headSize,
1108:               kvBlockSize,
1109:               qBlockSize,
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 1110-1142
```cpp
1110:               scaling_factor,
1111:               q_data + i * qStrideB + j * qStrideH +
1112:                   m * qStrideM,
1113:               qStrideM,
1114:               conditional_data_ptr(grad_attn_data, grad_attn_reduced_data),
1115:               kvBlockSize,
1116:               static_cast<accum_t>(1),
1117:               grad_k_data + i * grad_kStrideB + kv_j * grad_kStrideH +
1118:                   n * grad_kStrideN,
1119:               grad_kStrideN);
1120:           }
1121:         }
1122:       }
1123:       // Move to the next query
1124:       data_index_step(i, batchSize, kv_j, kv_num_head);
1125:     }
1126:   });
1127: }
1128:
1129: #define AT_DISPATCH_MASK_TYPES(TYPE, NAME, ...)            \
1130:   AT_DISPATCH_SWITCH(                                      \
1131:       TYPE,                                                \
1132:       NAME,                                                \
1133:       AT_PRIVATE_CASE_TYPE_USING_HINT(                     \
1134:           at::ScalarType::Bool, mask_t, __VA_ARGS__)       \
1135:       AT_PRIVATE_CASE_TYPE_USING_HINT(                     \
1136:           at::ScalarType::Float, mask_t, __VA_ARGS__)      \
1137:       AT_PRIVATE_CASE_TYPE_USING_HINT(                     \
1138:           at::ScalarType::Double, mask_t, __VA_ARGS__)     \
1139:       AT_PRIVATE_CASE_TYPE_USING_HINT(                     \
1140:           at::ScalarType::BFloat16, mask_t, __VA_ARGS__)   \
1141:       AT_PRIVATE_CASE_TYPE_USING_HINT(                     \
1142:           at::ScalarType::Half, mask_t, __VA_ARGS__))
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1144-1172
```cpp
1144: #define FLASH_ATTENTION_KERNEL(FNAME, PACK, TYPE1, TYPE2, SEQ1, SEQ2, ...)   \
1145:   if (PACK) {                                                      \
1146:     FNAME<TYPE1, TYPE2, SEQ1, SEQ2, true>(__VA_ARGS__);            \
1147:   } else {                                                         \
1148:     FNAME<TYPE1, TYPE2, SEQ1, SEQ2>(__VA_ARGS__);                  \
1149:   }
1150:
1151: void flash_attention_kernel_impl(
1152:     const Tensor& output,
1153:     const Tensor& logsumexp,
1154:     const at::Tensor& query,
1155:     const at::Tensor& key,
1156:     const at::Tensor& value,
1157:     double dropout_p,
1158:     bool is_causal,
1159:     std::optional<Tensor> attn_mask,
1160:     std::optional<double> scale) {
1161:   auto q_seq_len = query.size(2);
1162:
1163:   // When q_seq_len and k_seq_len are long enough,
1164:   // cpu_flash_attention with pack has better performance.
1165:   bool could_pack = ((query.scalar_type() == kHalf && cpublas::could_pack(kHalf)) ||
1166:                      (query.scalar_type() == kBFloat16 && cpublas::could_pack(kBFloat16)));
1167:
1168:   AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, query.scalar_type(), "flash_attention", [&] {
1169:     if (!attn_mask.has_value()) {
1170:       if (q_seq_len >= 768) {
1171:         FLASH_ATTENTION_KERNEL(cpu_flash_attention, could_pack, scalar_t, scalar_t, 256, 512,
1172:           output, logsumexp, query, key, value,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `flash_attention_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `flash_attention_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1173-1203
```cpp
1173:           dropout_p, is_causal, attn_mask, scale);
1174:       } else if (q_seq_len >= 192) {
1175:         FLASH_ATTENTION_KERNEL(cpu_flash_attention, could_pack, scalar_t, scalar_t, 64, 512,
1176:           output, logsumexp, query, key, value,
1177:           dropout_p, is_causal, attn_mask, scale);
1178:       } else {
1179:         FLASH_ATTENTION_KERNEL(cpu_flash_attention, could_pack, scalar_t, scalar_t, 32, 512,
1180:           output, logsumexp, query, key, value,
1181:           dropout_p, is_causal, attn_mask, scale);
1182:       }
1183:     } else {
1184:       AT_DISPATCH_MASK_TYPES(attn_mask.value().scalar_type(), "flash_attention_mask", [&]() {
1185:         if (q_seq_len >= 768) {
1186:           FLASH_ATTENTION_KERNEL(cpu_flash_attention, could_pack, scalar_t, mask_t, 256, 512,
1187:             output, logsumexp, query, key, value,
1188:             dropout_p, is_causal, attn_mask, scale);
1189:         } else if (q_seq_len >= 192) {
1190:           FLASH_ATTENTION_KERNEL(cpu_flash_attention, could_pack, scalar_t, mask_t, 64, 512,
1191:             output, logsumexp, query, key, value,
1192:             dropout_p, is_causal, attn_mask, scale);
1193:         } else {
1194:           FLASH_ATTENTION_KERNEL(cpu_flash_attention, could_pack, scalar_t, mask_t, 32, 512,
1195:             output, logsumexp, query, key, value,
1196:             dropout_p, is_causal, attn_mask, scale);
1197:         }
1198:       });
1199:     }
1200:   });
1201: }
1202:
1203: #undef FLASH_ATTENTION_KERNEL
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1205-1233
```cpp
1205: void flash_attention_backward_kernel_impl(
1206:     const at::Tensor& grad_q,
1207:     const at::Tensor& grad_k,
1208:     const at::Tensor& grad_v,
1209:     const at::Tensor& grad_out,
1210:     const at::Tensor& query,
1211:     const at::Tensor& key,
1212:     const at::Tensor& value,
1213:     const at::Tensor& out,
1214:     const at::Tensor& logsumexp,
1215:     double dropout_p,
1216:     bool is_causal,
1217:     std::optional<Tensor> attn_mask,
1218:     std::optional<double> scale) {
1219:   // make sure grad_out has no zero strides (broadcasted dimensions)
1220:   // since we are going to call gemm next
1221:   // zero stride in leading dimension would lead to slow impl for gemm
1222:   auto grad_out_contig = grad_out.contiguous();
1223:   auto q_seq_len = query.size(1);
1224:
1225:   AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, query.scalar_type(), "flash_attention_backward", [&] {
1226:     if (!attn_mask.has_value() || !attn_mask.value().defined()) {
1227:       using accum_t = at::opmath_type<scalar_t>;
1228:       if (q_seq_len >= 768) {
1229:         cpu_flash_attention_backward<scalar_t, accum_t, 256, 512>(
1230:           grad_q, grad_k, grad_v, grad_out_contig,
1231:           query, key, value, out, logsumexp,
1232:           dropout_p, is_causal, attn_mask, scale);
1233:       } else if (q_seq_len >= 192) {
```
- EN: The main symbol in this range is `flash_attention_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `flash_attention_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1234-1265
```cpp
1234:         cpu_flash_attention_backward<scalar_t, accum_t, 64, 512>(
1235:           grad_q, grad_k, grad_v, grad_out_contig,
1236:           query, key, value, out, logsumexp,
1237:           dropout_p, is_causal, attn_mask, scale);
1238:       } else {
1239:         cpu_flash_attention_backward<scalar_t, accum_t, 32, 512>(
1240:           grad_q, grad_k, grad_v, grad_out_contig,
1241:           query, key, value, out, logsumexp,
1242:           dropout_p, is_causal, attn_mask, scale);
1243:       }
1244:     } else {
1245:       AT_DISPATCH_MASK_TYPES(attn_mask.value().scalar_type(), "flash_attention_mask_backward", [&]() {
1246:         if (q_seq_len >= 768) {
1247:           cpu_flash_attention_backward<scalar_t, mask_t, 256, 512>(
1248:             grad_q, grad_k, grad_v, grad_out_contig,
1249:             query, key, value, out, logsumexp,
1250:             dropout_p, is_causal, attn_mask, scale);
1251:         } else if (q_seq_len >= 192) {
1252:           cpu_flash_attention_backward<scalar_t, mask_t, 64, 512>(
1253:             grad_q, grad_k, grad_v, grad_out_contig,
1254:             query, key, value, out, logsumexp,
1255:             dropout_p, is_causal, attn_mask, scale);
1256:         } else {
1257:           cpu_flash_attention_backward<scalar_t, mask_t, 32, 512>(
1258:             grad_q, grad_k, grad_v, grad_out_contig,
1259:             query, key, value, out, logsumexp,
1260:             dropout_p, is_causal, attn_mask, scale);
1261:         }
1262:       });
1263:     }
1264:   });
1265: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1267-1272
```cpp
1267: } // anonymous namespace
1268:
1269: ALSO_REGISTER_AVX512_DISPATCH(flash_attention_kernel, &flash_attention_kernel_impl)
1270: ALSO_REGISTER_AVX512_DISPATCH(flash_attention_backward_kernel, &flash_attention_backward_kernel_impl)
1271:
1272: } // at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Normalization statistics / 归一化统计
- Low-level memory access / 底层内存访问

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/cpu/vec/vec.h`, `ATen/cpu/vec/vec_half.h`, `ATen/cpu/vec/functional.h`, `ATen/native/CPUBlas.h`, `ATen/native/cpu/utils.h`, `ATen/native/transformers/attention.h`, `ATen/native/transformers/sdp_utils_cpp.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `Vectorized`, `parallel_for`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
