# QuantizedOpKernels.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/kernels/QuantizedOpKernels.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-72
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/core/List.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/Parallel.h>
 6: #include <ATen/native/Activation.h>
 7: #include <ATen/native/TopKImpl.h>
 8: #include <ATen/native/TensorIterator.h>
 9: #include <ATen/native/UpSample.h>
10: #include <ATen/native/cpu/IndexKernelUtils.h>
11: #include <ATen/native/cpu/Loops.h>
12: #include <ATen/native/quantized/AffineQuantizer.h>
13: #include <ATen/native/quantized/FakeQuantAffine.h>
14: #include <ATen/native/quantized/IndexKernel.h>
15: #include <ATen/native/quantized/cpu/QuantizedOps.h>
16: #include <ATen/native/cpu/utils.h>
17: #include <c10/util/irange.h>
18: #include <c10/util/Unroll.h>
19:
20: #ifndef AT_PER_OPERATOR_HEADERS
21: #include <ATen/Functions.h>
22: #else
23: #include <ATen/ops/_empty_affine_quantized.h>
24: #include <ATen/ops/empty.h>
25: #endif
26:
27: #include <cmath>
28: #ifdef USE_FBGEMM
29: #include <fbgemm/QuantUtils.h>
30: #endif
31: #ifdef _OPENMP
32: #include <omp.h>
33: #endif
34: #if defined(__ARM_NEON__) || defined(__aarch64__)
35: #include <ATen/quantized/Quantizer.h>
36: #include <arm_neon.h>
37: #endif
38:
39:
40: // NOLINTBEGIN(*-c-arrays)
41: namespace at::native {
42: namespace {
43:
44: void check_tensor_memory_format(const Tensor& ref, const Tensor& other) {
45:   TORCH_CHECK(
46:       ref.is_contiguous(ref.suggest_memory_format()),
47:       "Quantized tensor should be contiguous");
48:   TORCH_CHECK(
49:       other.is_contiguous(ref.suggest_memory_format()),
50:       "Float tensor should be contiguous "
51:       "in same memory format as quantized tensor");
52: }
53:
54: // ****************** HEY YOU! YES YOU! Read this! ********************
55: //
56: // Please read the README.md in this directory before editing this file
57:
58: template <bool ReLUFused = false>
59: Tensor qcat_nhwc_kernel(
60:     const MaterializedITensorListRef& qxs,
61:     int64_t dim,
62:     double scale,
63:     int64_t zero_point) {
64:   const at::Tensor& qx0 = qxs[0];
65:   int64_t C_out = 0;
66:   std::vector<int64_t> Cs_in;
67:   // Prefix sum of input channels for fast indexing
68:   std::vector<int64_t> Cs_sum;
69:   std::vector<double> scales;
70:   std::vector<int64_t> zero_pts;
71:   std::vector<void*> data_ptrs;
72:   std::vector<bool> is_fast_path;
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 74-143
```cpp
 74:   for (const at::Tensor& qx : qxs) {
 75:     TORCH_CHECK(
 76:         qx.dim() == qx0.dim(),
 77:         "Tensors must have the same number of dimensions: got ",
 78:         qx.dim(),
 79:         " and ",
 80:         qx0.dim());
 81: #define CHECK_DIM(d)                                            \
 82:   TORCH_CHECK(                                                  \
 83:       qx.size(d) == qx0.size(d),                                \
 84:       "Sizes of tensors must match expect in dimension 1. Got", \
 85:       qx.size(d),                                               \
 86:       " and ",                                                  \
 87:       qx0.size(d));
 88:     CHECK_DIM(0);
 89:     CHECK_DIM(2);
 90:     CHECK_DIM(3);
 91:     TORCH_CHECK(
 92:         qx.scalar_type() == qx0.scalar_type(),
 93:         "Expected object of scalar type ",
 94:         toString(qx0.scalar_type()),
 95:         " but got scalar type ",
 96:         toString(qx.scalar_type()));
 97:     Cs_in.push_back(qx.size(1));
 98:     Cs_sum.push_back(C_out);
 99:     C_out += qx.size(1);
100:     scales.push_back(qx.q_scale());
101:     zero_pts.push_back(qx.q_zero_point());
102:     data_ptrs.push_back(qx.data_ptr());
103:     is_fast_path.push_back(
104:         qx.q_scale() == scale &&
105:         qx.q_zero_point() == zero_point);
106:   }
107:
108:   const int64_t N = qx0.size(0);
109:   const int64_t H = qx0.size(2);
110:   const int64_t W = qx0.size(3);
111:   float inv_scale = static_cast<float>(1.0 / scale);
112:
113:   auto output = at::_empty_affine_quantized(
114:       {N, C_out, H, W},
115:       qx0.options().memory_format(MemoryFormat::ChannelsLast),
116:       scale,
117:       zero_point,
118:       std::nullopt);
119:
120:   // N, H, and W are explicitly captured here because there's a bug in GCC5
121:   // and clang5 which causes an internal compiler error if they're not
122:   AT_DISPATCH_QINT_TYPES(output.scalar_type(), "qcat_nhwc", [&, N, H, W]() {
123:     using Vec = Vectorized<scalar_t>;
124:     at::parallel_for(0, N * H * W, 0, [&](int64_t begin, int64_t end) {
125:       for (const auto i : c10::irange(begin, end)) {
126:         // loop over input tensors
127:         for (const auto tidx : c10::irange(Cs_in.size())) {
128:           scalar_t::underlying* optr =
129:               reinterpret_cast<scalar_t::underlying*>(output.data_ptr()) +
130:               i * C_out + Cs_sum[tidx];
131:
132:           auto curr_C = Cs_in[tidx];
133:           float curr_scale = scales[tidx];
134:           int64_t curr_zero_pt = zero_pts[tidx];
135:
136:           scalar_t::underlying* iptr =
137:               reinterpret_cast<scalar_t::underlying*>(data_ptrs[tidx]) +
138:               i * curr_C;
139:
140:           if (is_fast_path[tidx] && !ReLUFused) {
141:             std::memcpy(optr, iptr, curr_C * sizeof(typename scalar_t::underlying));
142:             continue;
143:           }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 运行时分派会根据数据类型或量化标量类别选择专用实现。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 145-216
```cpp
145:           constexpr auto VLEN = Vec::size();
146:           int64_t c = 0;
147:
148:           // Vectorized loop
149:           if (c + VLEN <= curr_C) {
150:             auto curr_scale_vec = Vectorized<float>(curr_scale);
151:             auto curr_zero_pt_vec = Vectorized<float>(curr_zero_pt);
152:             auto scale_neg_zp_premul = curr_scale_vec * curr_zero_pt_vec.neg();
153:             for (; c + VLEN <= curr_C; c += VLEN) {
154:               auto inp_vec = Vec::loadu(iptr + c);
155:               auto float_values = inp_vec.dequantize(
156:                   curr_scale_vec, curr_zero_pt_vec, scale_neg_zp_premul);
157:               Vec::float_vec_return_type retvals;
158:               for (int i = 0; i < Vec::float_num_vecs(); ++i) {
159:                 if constexpr (ReLUFused) {
160:                   retvals[i] =
161:                       vec::maximum(float_values[i], Vectorized<float>(0.0f));
162:                 } else {
163:                   retvals[i] = float_values[i];
164:                 }
165:               }
166:               auto quantized =
167:                   Vec::quantize(retvals, scale, zero_point, inv_scale);
168:               quantized.store(optr + c);
169:             }
170:           }
171:
172:           // Vectorized loop for channel between 8 and 32 (avx2)
173:           constexpr auto kVLEN = Vectorized<float>::size();
174:           int64_t elem_size = curr_C - c;
175:           if ((VLEN == 4 * kVLEN) && elem_size >= kVLEN) {
176:             auto curr_scale_vec = Vectorized<float>(curr_scale);
177:             auto curr_zero_pt_vec = Vectorized<float>(curr_zero_pt);
178:             auto scale_neg_zp_premul = curr_scale_vec * curr_zero_pt_vec.neg();
179:             int64_t vec_num = elem_size / kVLEN;
180:             std::array<typename scalar_t::underlying, VLEN> buf_in{};
181:             memcpy(buf_in.data(), iptr + c, vec_num * kVLEN);
182:             auto inp_vec = Vec::loadu(buf_in.data());
183:             auto float_values = inp_vec.dequantize(
184:                 curr_scale_vec, curr_zero_pt_vec, scale_neg_zp_premul);
185:             Vec::float_vec_return_type retvals;
186:             for (int i = 0; i < vec_num; ++i) {
187:               if constexpr (ReLUFused) {
188:                 retvals[i] =
189:                     vec::maximum(float_values[i], Vectorized<float>(0.0f));
190:               } else {
191:                 retvals[i] = float_values[i];
192:               }
193:             }
194:             auto quantized =
195:                 Vec::quantize(retvals, scale, zero_point, inv_scale);
196:             quantized.store(optr + c, vec_num * kVLEN);
197:             c += vec_num * kVLEN;
198:           }
199:
200:           // Scalar loop
201:           for (; c < curr_C; ++c) {
202:             auto float_val = at::native::dequantize_val(
203:                 curr_scale,
204:                 curr_zero_pt,
205:                 reinterpret_cast<scalar_t*>(iptr)[c]);
206:             if constexpr (ReLUFused) {
207:               float_val = std::max(0.0f, float_val);
208:             }
209:             optr[c] = at::native::quantize_val<scalar_t>(
210:                           scale, zero_point, float_val)
211:                           .val_;
212:           } // for c
213:         } // for tidx
214:       } // for i
215:     });
216:   });
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 218-297
```cpp
218:   return output;
219: }
220:
221: // horizontal sum over a range of uint8_t
222: int64_t hsum(const uint8_t* A, int len) {
223:   int64_t row_sum = 0;
224:   int i = 0;
225:
226: #ifdef CPU_CAPABILITY_AVX2
227:   __m256i sum_v = _mm256_setzero_si256();
228:   __m256i one_epi16_v = _mm256_set1_epi16(1);
229:   __m256i one_epi8_v = _mm256_set1_epi8(1);
230:   // vectorized
231:   for (; i < len / 32 * 32; i += 32) {
232:     __m256i src_v = _mm256_loadu_si256(reinterpret_cast<__m256i const*>(A + i));
233:     sum_v = _mm256_add_epi32(
234:       sum_v,
235:       _mm256_madd_epi16(
236:         // first argument is unsigned, second is signed
237:         _mm256_maddubs_epi16(src_v, one_epi8_v),
238:       one_epi16_v)
239:     );
240:   }
241:
242:   alignas(64) int32_t temp[8];
243:   _mm256_store_si256(reinterpret_cast<__m256i*>(temp), sum_v);
244:   for (const auto k : c10::irange(8)) {
245:     row_sum += temp[k];
246:   }
247: #elif defined(CPU_CAPABILITY_AVX512)
248:   __m512i sum_v = _mm512_setzero_si512();
249:   __m512i one_epi16_v = _mm512_set1_epi16(1);
250:   __m512i one_epi8_v = _mm512_set1_epi8(1);
251:   // vectorized
252:   for (; i < len / 64 * 64; i += 64) {
253:     __m512i src_v = _mm512_loadu_si512(reinterpret_cast<__m512i const*>(A + i));
254:     sum_v = _mm512_add_epi32(
255:       sum_v,
256:       _mm512_madd_epi16(
257:         // first argument is unsigned, second is signed
258:         _mm512_maddubs_epi16(src_v, one_epi8_v),
259:       one_epi16_v)
260:     );
261:   }
262:
263:   alignas(64) int32_t temp[16];
264:   _mm512_store_si512(reinterpret_cast<__m512i*>(temp), sum_v);
265:   for (const auto k : c10::irange(16)) {
266:     row_sum += temp[k];
267:   }
268: #endif // CPU_CAPABILITY_AVX2 or CPU_CAPABILITY_AVX512
269:
270:   // scalar
271:   for (; i < len; ++i) {
272:     row_sum += A[i];
273:   }
274:
275:   return row_sum;
276: }
277:
278: // horizontal sum over a range of int8_t
279: int64_t hsum(const int8_t* A, int len) {
280:   int64_t row_sum = 0;
281:   int i = 0;
282:
283: #ifdef CPU_CAPABILITY_AVX2
284:   __m256i sum_v = _mm256_setzero_si256();
285:   __m256i one_epi16_v = _mm256_set1_epi16(1);
286:   __m256i one_epi8_v = _mm256_set1_epi8(1);
287:   // vectorized
288:   for (; i < len / 32 * 32; i += 32) {
289:     __m256i src_v = _mm256_loadu_si256(reinterpret_cast<__m256i const*>(A + i));
290:     sum_v = _mm256_add_epi32(
291:       sum_v,
292:       _mm256_madd_epi16(
293:         // first argument is unsigned, second is signed
294:         _mm256_maddubs_epi16(one_epi8_v, src_v),
295:       one_epi16_v)
296:     );
297:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `hsum`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `hsum`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 299-373
```cpp
299:   alignas(64) int32_t temp[8];
300:   _mm256_store_si256(reinterpret_cast<__m256i*>(temp), sum_v);
301:   for (const auto k : c10::irange(8)) {
302:     row_sum += temp[k];
303:   }
304: #elif defined(CPU_CAPABILITY_AVX512)
305:   __m512i sum_v = _mm512_setzero_si512();
306:   __m512i one_epi16_v = _mm512_set1_epi16(1);
307:   __m512i one_epi8_v = _mm512_set1_epi8(1);
308:   // vectorized
309:   for (; i < len / 64 * 64; i += 64) {
310:     __m512i src_v = _mm512_loadu_si512(reinterpret_cast<__m512i const*>(A + i));
311:     sum_v = _mm512_add_epi32(
312:       sum_v,
313:       _mm512_madd_epi16(
314:         // first argument is unsigned, second is signed
315:         _mm512_maddubs_epi16(one_epi8_v, src_v),
316:       one_epi16_v)
317:     );
318:   }
319:
320:   alignas(64) int32_t temp[16];
321:   _mm512_store_si512(reinterpret_cast<__m512i*>(temp), sum_v);
322:   for (const auto k : c10::irange(16)) {
323:     row_sum += temp[k];
324:   }
325: #endif // CPU_CAPABILITY_AVX2 or CPU_CAPABILITY_AVX512
326:
327:   // scalar
328:   for (; i < len; ++i) {
329:     row_sum += A[i];
330:   }
331:
332:   return row_sum;
333: }
334:
335: // horizontal sum over a range of int32_t
336: int64_t hsum(const int32_t* A, int len) {
337:   int64_t row_sum = 0;
338:   int i = 0;
339:
340: #ifdef CPU_CAPABILITY_AVX2
341:   __m256i sum_epi64 = _mm256_setzero_si256();
342:   // vectorized
343:   for (; i < len / 8 * 8; i += 8) {
344:     __m256i src_epi32 = _mm256_loadu_si256(reinterpret_cast<__m256i const*>(A + i));
345:     // widen
346:     __m128i src_lo_epi32 = _mm256_castsi256_si128(src_epi32);
347:     __m128i src_hi_epi32 = _mm256_extracti128_si256(src_epi32, 1);
348:     __m256i src_lo_epi64 = _mm256_cvtepi32_epi64(src_lo_epi32);
349:     __m256i src_hi_epi64 = _mm256_cvtepi32_epi64(src_hi_epi32);
350:     // add
351:     sum_epi64 = _mm256_add_epi64(sum_epi64, src_lo_epi64);
352:     sum_epi64 = _mm256_add_epi64(sum_epi64, src_hi_epi64);
353:   }
354:
355:   alignas(64) int64_t temp[4];
356:   _mm256_store_si256(reinterpret_cast<__m256i*>(temp), sum_epi64);
357:   for (const auto k : c10::irange(4)) {
358:     row_sum += temp[k];
359:   }
360: #elif defined(CPU_CAPABILITY_AVX512)
361:   __m512i sum_epi64 = _mm512_setzero_si512();
362:   // vectorized
363:   for (; i < len / 16 * 16; i += 16) {
364:     __m512i src_epi32 = _mm512_loadu_si512(reinterpret_cast<__m512i const*>(A + i));
365:     // widen
366:     __m256i src_lo_epi32 = _mm512_castsi512_si256(src_epi32);
367:     __m256i src_hi_epi32 = _mm512_extracti32x8_epi32(src_epi32, 1);
368:     __m512i src_lo_epi64 = _mm512_cvtepi32_epi64(src_lo_epi32);
369:     __m512i src_hi_epi64 = _mm512_cvtepi32_epi64(src_hi_epi32);
370:     // add
371:     sum_epi64 = _mm512_add_epi64(sum_epi64, src_lo_epi64);
372:     sum_epi64 = _mm512_add_epi64(sum_epi64, src_hi_epi64);
373:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `hsum`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `hsum`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 375-454
```cpp
375:   alignas(64) int64_t temp[8];
376:   _mm512_store_si512(reinterpret_cast<__m512i*>(temp), sum_epi64);
377:   for (const auto k : c10::irange(8)) {
378:     row_sum += temp[k];
379:   }
380: #endif // CPU_CAPABILITY_AVX2 or CPU_CAPABILITY_AVX512
381:
382:   // scalar
383:   for (; i < len; ++i) {
384:     row_sum += A[i];
385:   }
386:
387:   return row_sum;
388: }
389:
390: // horizontal sum of squares over a range of uint8_t
391: int64_t hsum_sq(const uint8_t* A, int len) {
392:   int64_t row_sum = 0;
393:   int i = 0;
394:
395: #ifdef CPU_CAPABILITY_AVX2
396:   // vectorized
397:   __m256i sum_v_epu32 = _mm256_setzero_si256();
398:   alignas(64) int32_t temp[8];
399:   int overflow_threshold = 262144; // 2147483647(max of int32)/(256*256)*8 = 262144
400:   int loop = len / overflow_threshold + 1;
401:   for(int j=0; j<=loop; j++){
402:     for (; ((i < overflow_threshold * j) && (i < len / 16 * 16)); i += 16) {
403:       // (i15, ..., i0)
404:       __m128i src_epu8 = _mm_loadu_si128(reinterpret_cast<__m128i const*>(A + i));
405:       __m256i src_epu16 = _mm256_cvtepu8_epi16(src_epu8);
406:       // (i15 ^ 2, ..., i0 ^ 2)
407:       __m256i sq_epu16 = _mm256_mullo_epi16(src_epu16, src_epu16);
408:       // (i7 ^ 2, ..., i0 ^ 2)
409:       __m128i sq_lo_epu16 = _mm256_castsi256_si128(sq_epu16);
410:       // (i15 ^ 2, ..., i8 ^ 2)
411:       __m128i sq_hi_epu16 = _mm256_extractf128_si256(sq_epu16, 1);
412:       // widen to epu32
413:       __m256i sq_lo_epu32 = _mm256_cvtepu16_epi32(sq_lo_epu16);
414:       __m256i sq_hi_epu32 = _mm256_cvtepu16_epi32(sq_hi_epu16);
415:       // add to running sum
416:       sum_v_epu32 = _mm256_add_epi32(sum_v_epu32, sq_lo_epu32);
417:       sum_v_epu32 = _mm256_add_epi32(sum_v_epu32, sq_hi_epu32);
418:     }
419:     _mm256_store_si256(reinterpret_cast<__m256i*>(temp), sum_v_epu32);
420:     for (const auto k : c10::irange(8)) {
421:       row_sum += temp[k];
422:     }
423:     sum_v_epu32 = _mm256_setzero_si256();
424:   }
425: #elif defined(CPU_CAPABILITY_AVX512)
426:   __m512i sum_v_epu32 = _mm512_setzero_si512();
427:   alignas(64) int32_t temp[16];
428:   int overflow_threshold = 262144; // 2147483647(max of int32)/(512*512)*8 = 262144
429:   int loop = len / overflow_threshold + 1;
430:   for(int j=0; j<=loop; j++){
431:     for (; ((i < overflow_threshold * j) && (i < len / 32 * 32)); i += 32) {
432:       // (i31, ..., i0)
433:       __m256i src_epu8 = _mm256_loadu_si256(reinterpret_cast<__m256i const*>(A + i));
434:       __m512i src_epu16 = _mm512_cvtepu8_epi16(src_epu8);
435:       // (i31 ^ 2, ..., i0 ^ 2)
436:       __m512i sq_epu16 = _mm512_mullo_epi16(src_epu16, src_epu16);
437:       // (i15 ^ 2, ..., i0 ^ 2)
438:       __m256i sq_lo_epu16 = _mm512_castsi512_si256(sq_epu16);
439:       // (i31 ^ 2, ..., i16 ^ 2)
440:       __m256i sq_hi_epu16 = _mm512_extracti32x8_epi32(sq_epu16, 1);
441:       // widen to epu32
442:       __m512i sq_lo_epu32 = _mm512_cvtepu16_epi32(sq_lo_epu16);
443:       __m512i sq_hi_epu32 = _mm512_cvtepu16_epi32(sq_hi_epu16);
444:       // add to running sum
445:       sum_v_epu32 = _mm512_add_epi32(sum_v_epu32, sq_lo_epu32);
446:       sum_v_epu32 = _mm512_add_epi32(sum_v_epu32, sq_hi_epu32);
447:     }
448:     _mm512_store_si512(reinterpret_cast<__m512i*>(temp), sum_v_epu32);
449:     for (const auto k : c10::irange(16)) {
450:       row_sum += temp[k];
451:     }
452:     sum_v_epu32 = _mm512_setzero_si512();
453:   }
454: #endif // CPU_CAPABILITY_AVX2 or CPU_CAPABILITY_AVX512
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `hsum_sq`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `hsum_sq`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 456-528
```cpp
456:   // scalar
457:   for (; i < len; ++i) {
458:     row_sum += A[i] * A[i];
459:   }
460:
461:   return row_sum;
462: }
463:
464: // horizontal sum of squares over a range of int8_t
465: int64_t hsum_sq(const int8_t* A, int len) {
466:   int64_t row_sum = 0;
467:   int i = 0;
468:
469: #ifdef CPU_CAPABILITY_AVX2
470:   // vectorized
471:   __m256i sum_v_epi32 = _mm256_setzero_si256();
472:   alignas(64) int32_t temp[8];
473:
474:   int overflow_threshold = 1048576; //2147483647/(128*128)*8 = 1048576
475:   int loop = len / overflow_threshold + 1;
476:
477:   for(int j=0; j<=loop; j++){
478:     for (; ((i < overflow_threshold * j) && (i < len / 16 * 16)); i += 16) {
479:       // (i15, ..., i0)
480:       __m128i src_epi8 = _mm_loadu_si128(reinterpret_cast<__m128i const*>(A + i));
481:       __m256i src_epi16 = _mm256_cvtepi8_epi16(src_epi8);
482:       // (i15 ^ 2, ..., i0 ^ 2)
483:       __m256i sq_epi16 = _mm256_mullo_epi16(src_epi16, src_epi16);
484:       // (i7 ^ 2, ..., i0 ^ 2)
485:       __m128i sq_lo_epi16 = _mm256_castsi256_si128(sq_epi16);
486:       // (i15 ^ 2, ..., i8 ^ 2)
487:       __m128i sq_hi_epi16 = _mm256_extractf128_si256(sq_epi16, 1);
488:       // widen to epi32
489:       __m256i sq_lo_epi32 = _mm256_cvtepi16_epi32(sq_lo_epi16);
490:       __m256i sq_hi_epi32 = _mm256_cvtepi16_epi32(sq_hi_epi16);
491:       // add to running sum
492:       sum_v_epi32 = _mm256_add_epi32(sum_v_epi32, sq_lo_epi32);
493:       sum_v_epi32 = _mm256_add_epi32(sum_v_epi32, sq_hi_epi32);
494:     }
495:     _mm256_store_si256(reinterpret_cast<__m256i*>(temp), sum_v_epi32);
496:
497:     for (const auto k : c10::irange(8)) {
498:       row_sum += temp[k];
499:     }
500:     sum_v_epi32 = _mm256_setzero_si256();
501:   }
502: #elif defined(CPU_CAPABILITY_AVX512)
503:   // vectorized
504:   __m512i sum_v_epi32 = _mm512_setzero_si512();
505:   alignas(64) int32_t temp[16];
506:
507:   int overflow_threshold = 1048576; //2147483647/(256*256)*8 = 1048576
508:   int loop = len / overflow_threshold + 1;
509:
510:   for(int j=0; j<=loop; j++){
511:     for (; ((i < overflow_threshold * j) && (i < len / 32 * 32)); i += 32) {
512:       // (i31, ..., i0)
513:       __m256i src_epi8 = _mm256_loadu_si256(reinterpret_cast<__m256i const*>(A + i));
514:       __m512i src_epi16 = _mm512_cvtepi8_epi16(src_epi8);
515:       // (i31 ^ 2, ..., i0 ^ 2)
516:       __m512i sq_epi16 = _mm512_mullo_epi16(src_epi16, src_epi16);
517:       // (i15 ^ 2, ..., i0 ^ 2)
518:       __m256i sq_lo_epi16 = _mm512_castsi512_si256(sq_epi16);
519:       // (i31 ^ 2, ..., i16 ^ 2)
520:       __m256i sq_hi_epi16 = _mm512_extracti32x8_epi32(sq_epi16, 1);
521:       // widen to epi32
522:       __m512i sq_lo_epi32 = _mm512_cvtepi16_epi32(sq_lo_epi16);
523:       __m512i sq_hi_epi32 = _mm512_cvtepi16_epi32(sq_hi_epi16);
524:       // add to running sum
525:       sum_v_epi32 = _mm512_add_epi32(sum_v_epi32, sq_lo_epi32);
526:       sum_v_epi32 = _mm512_add_epi32(sum_v_epi32, sq_hi_epi32);
527:     }
528:     _mm512_store_si512(reinterpret_cast<__m512i*>(temp), sum_v_epi32);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `hsum_sq`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `hsum_sq`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 530-609
```cpp
530:     for (const auto k : c10::irange(16)) {
531:       row_sum += temp[k];
532:     }
533:     sum_v_epi32 = _mm512_setzero_si512();
534:   }
535: #endif // CPU_CAPABILITY_AVX2 or CPU_CAPABILITY_AVX512
536:
537:   // scalar
538:   for (; i < len; ++i) {
539:     row_sum += A[i] * A[i];
540:   }
541:
542:   return row_sum;
543: }
544:
545: // horizontal sum os squares over a range of int32_t
546: // floats throughout are necessary to prevent overflow
547: float hsum_sq(const int32_t* A, int len) {
548:   float row_sum = 0;
549:   int i = 0;
550:
551: #ifdef CPU_CAPABILITY_AVX2
552:   __m256 sum_ps = _mm256_setzero_ps();
553:   // vectorized
554:   for (; i < len / 8 * 8; i += 8) {
555:     __m256i src_epi32 = _mm256_loadu_si256(reinterpret_cast<__m256i const*>(A + i));
556:     __m256 src_ps = _mm256_cvtepi32_ps(src_epi32);
557:     sum_ps = _mm256_add_ps(sum_ps, _mm256_mul_ps(src_ps, src_ps));
558:   }
559:
560:   alignas(64) float temp[8];
561:   _mm256_store_ps(temp, sum_ps);
562:   for (const auto k : c10::irange(8)) {
563:     row_sum += temp[k];
564:   }
565: #elif defined(CPU_CAPABILITY_AVX512)
566:   __m512 sum_ps = _mm512_setzero_ps();
567:   // vectorized
568:   for (; i < len / 16 * 16; i += 16) {
569:     __m512i src_epi32 = _mm512_loadu_si512(reinterpret_cast<__m512i const*>(A + i));
570:     __m512 src_ps = _mm512_cvtepi32_ps(src_epi32);
571:     sum_ps = _mm512_add_ps(sum_ps, _mm512_mul_ps(src_ps, src_ps));
572:   }
573:
574:   alignas(64) float temp[16];
575:   _mm512_store_ps(temp, sum_ps);
576:   for (const auto k : c10::irange(16)) {
577:     row_sum += temp[k];
578:   }
579: #endif // CPU_CAPABILITY_AVX2 or CPU_CAPABILITY_AVX512
580:
581:   // scalar
582:   for (; i < len; ++i) {
583:     int64_t cur = static_cast<int64_t>(A[i]);
584:     row_sum += (float)cur * (float)cur;
585:   }
586:
587:   return row_sum;
588: }
589:
590: void qrelu_kernel(const Tensor& qx, Tensor& qy) {
591:   const auto zero_point = qx.q_zero_point();
592:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qrelu", [&]() {
593:     qy = at::_empty_affine_quantized(
594:         qx.sizes(),
595:         at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
596:         qx.q_scale(),
597:         qx.q_zero_point(),
598:         std::nullopt);
599:     using Vec = Vectorized<scalar_t>;
600:     auto zero_point_vec = Vec(scalar_t(zero_point));
601:     auto iter = TensorIterator::unary_op(qy, qx);
602:     cpu_kernel_vec(
603:         iter,
604:         [&](scalar_t value) -> scalar_t {
605:           return scalar_t(std::max<underlying_t>(value.val_, zero_point));
606:         },
607:         [&](Vec value) -> Vec { return value.relu(zero_point_vec); });
608:   });
609: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `hsum_sq`, `qrelu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `hsum_sq`, `qrelu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 611-690
```cpp
611: void leaky_qrelu_out_kernel(Tensor& out, const Tensor& qx,
612:                                    const Scalar& negval_) {
613:   int64_t i_zp = qx.q_zero_point();
614:   float i_scale = static_cast<float>(qx.q_scale());
615:
616:   int64_t o_zp = out.q_zero_point();
617:   float o_scale = static_cast<float>(out.q_scale());
618:   float o_inv_scale = 1.0f / o_scale;
619:
620:   float negval = negval_.to<float>();
621:
622:   AT_DISPATCH_QINT_TYPES(out.scalar_type(), "leaky_qrelu", [&] {
623:     using Vec = Vectorized<float>;  // Naive implementation uses dequant/quant loop.
624:     using qVec = Vectorized<scalar_t>;
625:     Vec zero_vec = Vec(0.0f);
626:     Vec one_vec = Vec(1.0f);
627:
628:     Vec i_scale_vec = Vec(i_scale);
629:     Vec i_zp_vec = Vec(i_zp);
630:     Vec i_scale_zp_neg_premul_vec = i_scale_vec * i_zp_vec.neg();
631:
632:     Vec negval_vec = Vec(negval);
633:
634:     auto iter = TensorIterator::unary_op(out, qx);
635:
636:     cpu_kernel_vec(
637:         iter,
638:         [&](scalar_t value_qx) -> scalar_t {
639:           auto value_dx = at::native::dequantize_val(i_scale, i_zp, value_qx);
640:           auto value_dy = value_dx > 0 ? value_dx : value_dx * negval;
641:           return at::native::quantize_val<scalar_t>(o_scale, o_zp, value_dy);
642:         },
643:         [&](qVec qx_vec) -> qVec {
644:           /* Vectorized implementation creates a multiplicand vector, which has
645:            * "alpha" for all negative dx values and ones-vector for all
646:            * positive values of dx. The multiplicand then is multiplied by the
647:            * input.
648:            */
649:           auto dx_vec_vec = qx_vec.dequantize(i_scale_vec, i_zp_vec,
650:                                               i_scale_zp_neg_premul_vec);
651:           for (auto & dx_vec : dx_vec_vec) {
652:             const auto multiplicand = Vec::blendv(negval_vec, one_vec,
653:                                                   dx_vec > zero_vec);
654:             dx_vec *= multiplicand;
655:           }
656:           return qVec::quantize(dx_vec_vec, o_scale, o_zp, o_inv_scale);
657:         });
658:   });
659: }
660:
661: void qprelu_out_kernel(Tensor& out,
662:                               const Tensor& qx,
663:                               const Tensor& qw) {
664:   int32_t i_zp = static_cast<int32_t>(qx.q_zero_point());
665:   float i_scale = static_cast<float>(qx.q_scale());
666:
667:   int32_t w_zp = static_cast<int32_t>(qw.q_zero_point());
668:   float w_scale = static_cast<float>(qw.q_scale());
669:
670:   int32_t o_zp = static_cast<int32_t>(out.q_zero_point());
671:   float o_scale = static_cast<float>(out.q_scale());
672:   float o_inv_scale = 1.0f / o_scale;
673:
674:   float multiplier = i_scale * w_scale * o_inv_scale;
675:
676:   int64_t input_ndim = qx.dim();
677:   TORCH_CHECK(input_ndim > 0, "qprelu: zero-dim input tensor is not allowed.");
678:
679:   // This logic is present in at::prelu and repeated here, as this path can be
680:   // hit via quantized::prelu, which is registered under quantized/cpu/qprelu.cpu
681:   auto qw_nd = qw;
682:   if (input_ndim != qw_nd.dim()) {
683:     DimVector dim_w(input_ndim, 1);
684:     if (input_ndim > 1) {
685:       dim_w[1] = qw.numel();
686:     }
687:     // This will always be a view in CPU/CUDA, but some backends
688:     // like MKLDNN do not support views
689:     qw_nd = qw_nd.reshape(dim_w);
690:   }
```
- EN: The main symbol in this range is `leaky_qrelu_out_kernel`, `qprelu_out_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `leaky_qrelu_out_kernel`, `qprelu_out_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 692-760
```cpp
692:   auto iter = TensorIteratorConfig()
693:     .add_output(out)
694:     .add_input(qx)
695:     .add_input(qw_nd)
696:     .build();
697:
698:   AT_DISPATCH_QINT_TYPES(out.scalar_type(), "qprelu", [&] {
699:     using qVec = Vectorized<scalar_t>;
700:     qVec i_zp_vec = qVec(static_cast<scalar_t>(i_zp));
701:     qVec w_zp_vec = qVec(static_cast<scalar_t>(w_zp));
702:
703:     // Quantized one as weight
704:     auto qw_one = at::native::quantize_val<scalar_t>(w_scale, w_zp, 1.0f);
705:     qVec vec_qw_one = qVec(qw_one);
706:     auto vec_qw_one_sub_zp = vec_qw_one.widening_subtract(w_zp_vec)[0];
707:     int32_t qw_one_sub_zp = qw_one.val_ - w_zp;
708:
709:     cpu_kernel_vec(
710:       iter,
711:       [=](scalar_t val_qx, scalar_t val_qw) -> scalar_t {
712:         int32_t qx_pos = std::max(static_cast<int32_t>(val_qx.val_), i_zp);
713:         int32_t qx_neg = std::min(static_cast<int32_t>(val_qx.val_), i_zp);
714:         int32_t qx_pos_sub_zp = qx_pos - i_zp;
715:         int32_t qx_neg_sub_zp = qx_neg - i_zp;
716:         int32_t qw_sub_zp = val_qw.val_ - w_zp;
717:         auto qy_sub_zp = qx_pos_sub_zp * qw_one_sub_zp + qx_neg_sub_zp * qw_sub_zp;
718:         return at::native::requantize_from_int<scalar_t>(
719:             multiplier, o_zp, qy_sub_zp);
720:       },
721:       [=](qVec vec_qx, qVec vec_qw) -> qVec {
722:         auto vec_qx_pos = vec_qx.maximum(i_zp_vec);
723:         auto vec_qx_neg = vec_qx.minimum(i_zp_vec);
724:         qVec::int_vec_return_type qx_pos_sub_zp = vec_qx_pos.widening_subtract(i_zp_vec);
725:         qVec::int_vec_return_type qx_neg_sub_zp = vec_qx_neg.widening_subtract(i_zp_vec);
726:         qVec::int_vec_return_type qw_sub_zp = vec_qw.widening_subtract(w_zp_vec);
727:         qVec::int_vec_return_type qy_sub_zp;
728:         for (const auto i : c10::irange(qVec::int_num_vecs())) {
729:           qy_sub_zp[i] = qx_pos_sub_zp[i] * vec_qw_one_sub_zp + qx_neg_sub_zp[i] * qw_sub_zp[i];
730:         }
731:         return qVec::requantize_from_int(qy_sub_zp, multiplier, o_zp);
732:       });
733:   });
734:
735: }
736:
737: void qgelu_kernel(const Tensor& qx, Tensor& qy, GeluType approximate) {
738:   int64_t zero_point = qx.q_zero_point();
739:   float scale = static_cast<float>(qx.q_scale());
740:   auto scale_vec = Vectorized<float>(scale);
741:   auto zero_point_vec = Vectorized<float>(zero_point);
742:   auto scale_neg_zp_premul_vec = scale_vec * zero_point_vec.neg();
743:   int64_t output_zero_point = zero_point;
744:   float output_scale = scale;
745:   float inv_output_scale = 1.0 / output_scale;
746:   const auto kAlphaVec = Vectorized<float>(M_SQRT1_2);
747:   const auto kBetaVec = Vectorized<float>(M_SQRT2 * M_2_SQRTPI * 0.5);
748:   const auto kKappaVec = Vectorized<float>(0.044715);
749:   const auto kOneVec = Vectorized<float>(1);
750:   const auto kPointFiveVec = Vectorized<float>(0.5);
751:
752:   if (approximate == GeluType::Tanh) {
753:     AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qgelu", [&]() {
754:       qy = at::_empty_affine_quantized(
755:           qx.sizes(),
756:           at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
757:           output_scale,
758:           output_zero_point,
759:           std::nullopt);
760:       auto iter = TensorIterator::unary_op(qy, qx);
```
- EN: The main symbol in this range is `qgelu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qgelu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 762-830
```cpp
762:       using Vec = Vectorized<scalar_t>;
763:       cpu_kernel_vec(
764:           iter,
765:           [&](scalar_t value_qx) -> scalar_t {
766:             const auto value_dx =
767:                 at::native::dequantize_val(scale, zero_point, value_qx);
768:
769:             const auto kBeta = M_SQRT2 * M_2_SQRTPI * 0.5;
770:             const auto kKappa = 0.044715;
771:             const auto x_cube = value_dx * value_dx * value_dx;
772:             const auto inner = kBeta * (value_dx + kKappa * x_cube);
773:             const auto value_dy = 0.5 * value_dx * (1.0 + std::tanh(inner));
774:
775:             return at::native::quantize_val<scalar_t>(
776:                 output_scale, output_zero_point, value_dy);
777:           },
778:           [&](Vec value_qx) -> Vec {
779:             auto value_dx = value_qx.dequantize(
780:                 scale_vec, zero_point_vec, scale_neg_zp_premul_vec);
781:             for (auto & value : value_dx) {
782:               auto value_cube = value * value * value;
783:               auto inner = kBetaVec * (value + kKappaVec * value_cube);
784:               value = kPointFiveVec * value * (kOneVec + inner.tanh());
785:             }
786:             return Vec::quantize(
787:                 value_dx, output_scale, output_zero_point, inv_output_scale);
788:           });
789:     });
790:   } else {
791:     AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qgelu", [&]() {
792:       qy = at::_empty_affine_quantized(
793:           qx.sizes(),
794:           at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
795:           output_scale,
796:           output_zero_point,
797:           std::nullopt);
798:       auto iter = TensorIterator::unary_op(qy, qx);
799:
800:       using Vec = Vectorized<scalar_t>;
801:       cpu_kernel_vec(
802:           iter,
803:           [&](scalar_t value_qx) -> scalar_t {
804:             const auto value_dx =
805:                 at::native::dequantize_val(scale, zero_point, value_qx);
806:             const auto value_dy =
807:                 value_dx * 0.5 * (1 + std::erf(value_dx * M_SQRT1_2));
808:             return at::native::quantize_val<scalar_t>(
809:                 output_scale, output_zero_point, value_dy);
810:           },
811:           [&](Vec value_qx) -> Vec {
812:             auto value_dx = value_qx.dequantize(
813:                 scale_vec, zero_point_vec, scale_neg_zp_premul_vec);
814:             for (auto & value : value_dx) {
815:               value = value * kPointFiveVec * (kOneVec + (value * kAlphaVec).erf());
816:             }
817:             return Vec::quantize(
818:                 value_dx, output_scale, output_zero_point, inv_output_scale);
819:           });
820:     });
821:   }
822: }
823:
824:
825: void qsigmoid_kernel(
826:     const Tensor& qx, Tensor& qy, double output_scale, int64_t output_zero_point ) {
827:   int64_t zero_point = qx.q_zero_point();
828:   float scale = static_cast<float>(qx.q_scale());
829:   auto scale_vec = Vectorized<float>(scale);
830:   auto zero_point_vec = Vectorized<float>(zero_point);
```
- EN: The main symbol in this range is `qsigmoid_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qsigmoid_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 832-903
```cpp
832:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qsigmoid", [&]() {
833:     float inv_output_scale = 1.0 / output_scale;
834:
835:     qy = at::_empty_affine_quantized(
836:         qx.sizes(),
837:         at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
838:         output_scale,
839:         output_zero_point,
840:         std::nullopt);
841:     auto iter = TensorIterator::unary_op(qy, qx);
842:
843:     using Vec = Vectorized<scalar_t>;
844:     cpu_kernel_vec(
845:         iter,
846:         [&](scalar_t value_qx) -> scalar_t {
847:           const auto value_dx =
848:               at::native::dequantize_val(scale, zero_point, value_qx);
849:           const auto value_dy = 1.0f / (1.0 + std::exp((-value_dx)));
850:           return at::native::quantize_val<scalar_t>(
851:               output_scale, output_zero_point, value_dy);
852:         },
853:         [&](Vec value_qx) -> Vec {
854:           auto value_dx = value_qx.dequantize(scale_vec, zero_point_vec);
855:           for (auto & value : value_dx) {
856:             value = value.neg();
857:             value = value.exp();
858:             value = Vectorized<float>(1.0f) + value;
859:             value = value.reciprocal();
860:           }
861:           return Vec::quantize(
862:               value_dx, output_scale, output_zero_point, inv_output_scale);
863:         });
864:   });
865: }
866:
867: void qhardsigmoid_kernel(const Tensor& qx, Tensor& qy) {
868:   int64_t zero_point = qx.q_zero_point();
869:   float scale = static_cast<float>(qx.q_scale());
870:   auto scale_vec = Vectorized<float>(scale);
871:   auto zero_point_vec = Vectorized<float>(zero_point);
872:   auto scale_neg_zp_premul_vec = scale_vec * zero_point_vec.neg();
873:
874:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qhardsigmoid", [&]() {
875:
876:     // - Output scale is set to 1.0 / 2^(BIT_NUM)
877:     float output_scale = 0.00390625;  // 1.0 / 2^8
878:     if (SCALAR_TYPE == at::kQInt32) {
879:       output_scale = 2.3283064365386963e-10;  // 1.0 / 2^32
880:     }
881:     float inv_output_scale = 1.0 / output_scale;
882:
883:     // The default zero-point is zero.  As a one-off optimization for
884:     // kQInt8, we set the zero-point to -128 to maximize precision in the
885:     // [0, 1] output range. kQInt32 can be handled in a future PR if needed.
886:     int64_t output_zero_point = 0;
887:     if (SCALAR_TYPE == at::kQInt8) {
888:       output_zero_point = -128;
889:     }
890:
891:     qy = at::_empty_affine_quantized(
892:         qx.sizes(),
893:         at::device(kCPU).dtype(SCALAR_TYPE),
894:         output_scale,
895:         output_zero_point,
896:         qx.suggest_memory_format());
897:     auto iter = TensorIterator::unary_op(qy, qx);
898:
899:     using qVec = Vectorized<scalar_t>;
900:     using fVec = Vectorized<float>;
901:     fVec kZeroVec(0.0f);
902:     fVec kThreeVec(3.0f);
903:     fVec kSixVec(6.0f);
```
- EN: The main symbol in this range is `qhardsigmoid_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qhardsigmoid_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 905-989
```cpp
905:     // Naive implementation: uses dequantize/execute/quantize routine
906:     cpu_kernel_vec(
907:         iter,
908:         [&](scalar_t qx) -> scalar_t {
909:           auto x = at::native::dequantize_val(scale, zero_point, qx);
910:           const auto y = std::min(std::max(x + 3.0f, 0.0f), 6.0f) / 6.0f;
911:           return at::native::quantize_val<scalar_t>(
912:               output_scale, output_zero_point, y);
913:         },
914:         [&](qVec value_qx) -> qVec {
915:           auto value_dx = value_qx.dequantize(
916:               scale_vec, zero_point_vec, scale_neg_zp_premul_vec);
917:           for (auto & value : value_dx) {
918:             value =
919:                 vec::minimum(
920:                     vec::maximum(value + kThreeVec, kZeroVec),
921:                     kSixVec) /
922:                 kSixVec;
923:           }
924:           return qVec::quantize(
925:               value_dx, output_scale, output_zero_point, inv_output_scale);
926:         });
927:   });
928: }
929:
930: void qclamp_kernel(
931:     const Tensor& qx,
932:     const Scalar& min_scalar,
933:     const Scalar& max_scalar,
934:     Tensor& qy) {
935:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qclamp", [&]() {
936:     qy = at::_empty_affine_quantized(
937:         qx.sizes(),
938:         at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
939:         qx.q_scale(),
940:         qx.q_zero_point(),
941:         std::nullopt);
942:     using Vec = Vectorized<scalar_t>;
943:     auto iter = TensorIterator::unary_op(qy, qx);
944:     auto min = min_scalar.to<float>();
945:     auto max = max_scalar.to<float>();
946:     scalar_t min_q = at::native::quantize_val<scalar_t>(
947:         qx.q_scale(), qx.q_zero_point(), min);
948:     scalar_t max_q = at::native::quantize_val<scalar_t>(
949:         qx.q_scale(), qx.q_zero_point(), max);
950:     auto min_vec = Vec(min_q);
951:     auto max_vec = Vec(max_q);
952:     cpu_kernel_vec(
953:         iter,
954:         [&](scalar_t value) -> scalar_t {
955:           underlying_t min_clamped =
956:               std::max<underlying_t>(value.val_, min_q.val_);
957:           return scalar_t(std::min<underlying_t>(min_clamped, max_q.val_));
958:         },
959:         [&](Vec val) -> Vec {
960:           auto min_clamped = val.maximum(min_vec);
961:           return min_clamped.minimum(max_vec);
962:         });
963:   });
964: }
965:
966: void qclamp_min_kernel(const Tensor& qx, const Scalar& min_scalar, Tensor& qy) {
967:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qclamp", [&]() {
968:     qy = at::_empty_affine_quantized(
969:         qx.sizes(),
970:         at::device(kCPU)
971:             .dtype(SCALAR_TYPE)
972:             .memory_format(qx.suggest_memory_format()),
973:         qx.q_scale(),
974:         qx.q_zero_point(),
975:         std::nullopt);
976:     using Vec = Vectorized<scalar_t>;
977:     auto iter = TensorIterator::unary_op(qy, qx);
978:     auto min = min_scalar.to<float>();
979:     scalar_t min_q = at::native::quantize_val<scalar_t>(
980:         qx.q_scale(), qx.q_zero_point(), min);
981:     auto min_vec = Vec(min_q);
982:     cpu_kernel_vec(
983:         iter,
984:         [&](scalar_t value) -> scalar_t {
985:           return scalar_t(std::max<underlying_t>(value.val_, min_q.val_));
986:         },
987:         [&](Vec val) -> Vec { return val.maximum(min_vec); });
988:   });
989: }
```
- EN: The main symbol in this range is `qclamp_kernel`, `qclamp_min_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qclamp_kernel`, `qclamp_min_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 991-1082
```cpp
 991: void qclamp_max_kernel(const Tensor& qx, const Scalar& max_scalar, Tensor& qy) {
 992:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qclamp", [&]() {
 993:     qy = at::_empty_affine_quantized(
 994:         qx.sizes(),
 995:         at::device(kCPU)
 996:             .dtype(SCALAR_TYPE)
 997:             .memory_format(qx.suggest_memory_format()),
 998:         qx.q_scale(),
 999:         qx.q_zero_point(),
1000:         std::nullopt);
1001:     using Vec = Vectorized<scalar_t>;
1002:     auto iter = TensorIterator::unary_op(qy, qx);
1003:     auto max = max_scalar.to<float>();
1004:     scalar_t max_q = at::native::quantize_val<scalar_t>(
1005:         qx.q_scale(), qx.q_zero_point(), max);
1006:     auto max_vec = Vec(max_q);
1007:     cpu_kernel_vec(
1008:         iter,
1009:         [&](scalar_t value) -> scalar_t {
1010:           return scalar_t(std::min<underlying_t>(value.val_, max_q.val_));
1011:         },
1012:         [&](Vec val) -> Vec { return val.minimum(max_vec); });
1013:   });
1014: }
1015:
1016: void qthreshold_kernel(
1017:   // TODO: For future tasks, since output quantization parameters are set equal to
1018:   // the input ones, it might make sense to implement this completely in the
1019:   // quantized domain.
1020:    const Tensor& qx,
1021:    const Scalar& threshold_scalar,
1022:    const Scalar& value_scalar,
1023:    Tensor& qy) {
1024:
1025:   // defines input and output scales and zero_points
1026:   int64_t input_zero_point = qx.q_zero_point();
1027:   float input_scale = static_cast<float>(qx.q_scale());
1028:   int64_t output_zero_point = qy.q_zero_point();
1029:   float output_scale = static_cast<float>(qy.q_scale());
1030:   float inv_output_scale = static_cast<float>(1.0 / output_scale);
1031:
1032:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qthreshold", [&]() {
1033:     qy = at::_empty_affine_quantized(
1034:       qx.sizes(),
1035:       at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
1036:       qx.q_scale(),
1037:       qx.q_zero_point(),
1038:       std::nullopt);
1039:
1040:     // vectorized
1041:     using Vec = Vectorized<float>;
1042:     using qVec = Vectorized<scalar_t>;
1043:     // defines the iterator
1044:     auto iter = TensorIterator::unary_op(qy, qx);
1045:     // defines the vectorized versions
1046:     Vec input_scale_vec = Vec(input_scale);
1047:     Vec input_zero_point_vec = Vec(input_zero_point);
1048:     Vec input_scale_neg_zp_premul_vec = input_scale_vec * input_zero_point_vec.neg();
1049:     // defines the floating-point versions of threshold and value
1050:     float threshold_float = threshold_scalar.to<float>();
1051:     float value_float = value_scalar.to<float>();
1052:     Vec threshold_vec = Vec(threshold_float);
1053:     Vec value_vec = Vec(value_float);
1054:
1055:     // Naive implementation: uses dequantize/execute/quantize routine
1056:     cpu_kernel_vec(
1057:         iter,
1058:         [&](scalar_t value_qx) -> scalar_t {
1059:           // dequantize
1060:           const auto x = at::native::dequantize_val(input_scale, input_zero_point, value_qx);
1061:           // Applies the Threshold operation
1062:           const auto y = x > threshold_float ? x : value_float;
1063:           // quantize
1064:           return at::native::quantize_val<scalar_t>(output_scale, output_zero_point, y);
1065:         },
1066:         [&](qVec value_qx) -> qVec {
1067:           // dequantize
1068:           auto dx_vec = value_qx.dequantize(
1069:             input_scale_vec, input_zero_point_vec, input_scale_neg_zp_premul_vec);
1070:           for (auto & value : dx_vec) {
1071:             // check if any elements are below threshold
1072:             const auto cmp_to_threshold = value > threshold_vec;
1073:             if (cmp_to_threshold.zero_mask()) {
1074:               // blend
1075:               value = Vec::blendv(value_vec, value, cmp_to_threshold);
1076:             }
1077:           }
1078:           // quantize
1079:           return qVec::quantize(dx_vec, output_scale, output_zero_point, inv_output_scale);
1080:         });
1081:   });
1082: }
```
- EN: The main symbol in this range is `qclamp_max_kernel`, `qthreshold_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qclamp_max_kernel`, `qthreshold_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1085-1154
```cpp
1085: void qhardswish_kernel(const Tensor& qx, Tensor& qy) {
1086:   const auto i_scale = qx.q_scale();
1087:   const auto i_zero_point = qx.q_zero_point();
1088:
1089:   const auto o_scale = qy.q_scale();
1090:   const auto o_zero_point = qy.q_zero_point();
1091:   const float o_inv_scale = static_cast<float>(1.0 / o_scale);
1092:
1093:   using fVec = Vectorized<float>;
1094:   fVec i_scale_vec(i_scale);
1095:   fVec i_zero_point_vec(i_zero_point);
1096:   fVec i_scale_neg_zp_premul_vec = i_scale_vec * i_zero_point_vec.neg();
1097:   fVec zero_vec(0.0f);
1098:   fVec three_vec(3.0f);
1099:   fVec six_vec(6.0f);
1100:
1101:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qhardswish", [&]() {
1102:     using qVec = Vectorized<scalar_t>;
1103:     auto iter = TensorIterator::unary_op(qy, qx);
1104:     cpu_kernel_vec(
1105:         iter,
1106:         [&](scalar_t value) -> scalar_t {
1107:           const auto x =
1108:               at::native::dequantize_val(i_scale, i_zero_point, value);
1109:           const auto y = x * std::min(std::max(x + 3.0f, 0.0f), 6.0f) / 6.0f;
1110:           return at::native::quantize_val<scalar_t>(o_scale, o_zero_point, y);
1111:         },
1112:         [&](qVec value) -> qVec {
1113:           auto value_dx = value.dequantize(i_scale_vec, i_zero_point_vec,
1114:                                            i_scale_neg_zp_premul_vec);
1115:           for (auto & value : value_dx) {
1116:             value = value * vec::minimum(
1117:               vec::maximum(value + three_vec, zero_vec),
1118:               six_vec
1119:             ) / six_vec;
1120:           }
1121:           return qVec::quantize(value_dx, o_scale, o_zero_point, o_inv_scale);
1122:         });
1123:   });
1124: }
1125:
1126:
1127: void qtanh_kernel(const Tensor& qx, Tensor& qy) {
1128:   int64_t zero_point = qx.q_zero_point();
1129:   float scale = static_cast<float>(qx.q_scale());
1130:   auto scale_vec = Vectorized<float>(scale);
1131:   auto zero_point_vec = Vectorized<float>(zero_point);
1132:   auto scale_neg_zp_premul_vec = scale_vec * zero_point_vec.neg();
1133:
1134:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qtanh", [&]() {
1135:     // Naive implementation: uses dequantize/execute/quantize routine
1136:     // - Output scale is set to 2.0 / 2^(BIT_NUM)
1137:     // - For signed types output zero point is set to 0
1138:     // - For unsigned types output zero point is set to (qmax + qmin) / 2.0
1139:     float output_scale = 0.0078125;  // 2.0 / 512
1140:     int64_t output_zero_point = 0;
1141:     if (SCALAR_TYPE == at::kQInt32) {
1142:       output_scale = 4.656612873077393e-10;  // 2.0 / 2^32
1143:     } else if (SCALAR_TYPE == at::kQUInt8) {
1144:       output_zero_point = 128;
1145:     }
1146:     float inv_output_scale = 1.0 / output_scale;
1147:
1148:     qy = at::_empty_affine_quantized(
1149:         qx.sizes(),
1150:         at::device(kCPU).dtype(SCALAR_TYPE).memory_format(qx.suggest_memory_format()),
1151:         output_scale,
1152:         output_zero_point,
1153:         std::nullopt);
1154:     auto iter = TensorIterator::unary_op(qy, qx);
```
- EN: The main symbol in this range is `qhardswish_kernel`, `qtanh_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qhardswish_kernel`, `qtanh_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1156-1228
```cpp
1156:     using Vec = Vectorized<scalar_t>;
1157:     cpu_kernel_vec(
1158:         iter,
1159:         [&](scalar_t value_qx) -> scalar_t {
1160:           const auto value_dx =
1161:               at::native::dequantize_val(scale, zero_point, value_qx);
1162:           return at::native::quantize_val<scalar_t>(
1163:               output_scale, output_zero_point, std::tanh(value_dx));
1164:         },
1165:         [&](Vec value_qx) -> Vec {
1166:           const auto value_dx = value_qx.dequantize(
1167:               scale_vec, zero_point_vec, scale_neg_zp_premul_vec);
1168:           Vec::float_vec_return_type retvals;
1169:           for (const auto idx : c10::irange(Vec::float_num_vecs())) {
1170:             retvals[idx] = value_dx[idx].tanh();
1171:           }
1172:           return Vec::quantize(
1173:               retvals, output_scale, output_zero_point, inv_output_scale);
1174:         });
1175:   });
1176: }
1177:
1178: void qelu_kernel(
1179:     const Tensor& qx,
1180:     const Scalar& alpha,
1181:     const Scalar& scale,
1182:     const Scalar& input_scale,
1183:     Tensor& qy) {
1184:   // scale and input_scale arguments refer to a generalized ELU formula
1185:   // if x >= 0, ELU(x) = x * scale
1186:   // if x <= 0, ELU(x) = (exp(x * input_scale) - 1) * scale
1187:   // in the normal ELU formula, both are equal to 1
1188:   // they are NOT related to the quantization scale term
1189:
1190:   int64_t i_zp = qx.q_zero_point();
1191:   float i_scale = static_cast<float>(qx.q_scale());
1192:
1193:   // In a future PR, we can improve on output scale and zero_point
1194:   // selection.
1195:   int64_t o_zp = qy.q_zero_point();
1196:   float o_scale = static_cast<float>(qy.q_scale());
1197:   float inv_o_scale = static_cast<float>(1.0 / o_scale);
1198:
1199:   float alpha_float = alpha.to<float>();
1200:   float scale_coef = scale.to<float>();
1201:   float input_scale_coef = input_scale.to<float>();
1202:
1203:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qelu_kernel", [&] {
1204:
1205:     auto iter = TensorIterator::unary_op(qy, qx);
1206:
1207:     // vectorized
1208:     using Vec = Vectorized<float>;
1209:     using qVec = Vectorized<scalar_t>;
1210:
1211:     Vec zero_vec = Vec(0.0f);
1212:     Vec one_vec = Vec(1.0f);
1213:     Vec alpha_vec = Vec(alpha_float);
1214:     Vec scale_coef_vec = Vec(scale_coef);
1215:     Vec input_scale_coef_vec = Vec(input_scale_coef);
1216:     Vec i_scale_vec = Vec(i_scale);
1217:     Vec i_zero_point_vec = Vec(i_zp);
1218:     Vec i_scale_neg_zp_premul_vec = i_scale_vec * i_zero_point_vec.neg();
1219:
1220:     cpu_kernel_vec(
1221:       iter,
1222:       [&](scalar_t value_qx) -> scalar_t {
1223:         // dequantize
1224:         const auto x = at::native::dequantize_val(i_scale, i_zp, value_qx);
1225:         // ELU
1226:         const auto y = x >= 0
1227:           ? x * scale_coef
1228:           : (std::expm1(x * input_scale_coef) * alpha_float * scale_coef);
```
- EN: The main symbol in this range is `qelu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `qelu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1230-1320
```cpp
1230:         // quantize
1231:         return at::native::quantize_val<scalar_t>(o_scale, o_zp, y);
1232:       },
1233:       [&](qVec value_qx) -> qVec {
1234:         // dequantize
1235:         auto dx_vec_vec = value_qx.dequantize(i_scale_vec, i_zero_point_vec,
1236:                                             i_scale_neg_zp_premul_vec);
1237:         for (auto & value : dx_vec_vec) {
1238:           // quickly check if any elements are below zero
1239:           const auto cmp_to_zero = value > zero_vec;
1240:
1241:           if (cmp_to_zero.zero_mask()) {
1242:
1243:             Vec dx_vec_copy_neg_elu = value * one_vec;
1244:             // calculate the negative part of ELU on the copy
1245:             dx_vec_copy_neg_elu = dx_vec_copy_neg_elu * input_scale_coef_vec;
1246:             dx_vec_copy_neg_elu = dx_vec_copy_neg_elu.expm1();
1247:             dx_vec_copy_neg_elu = dx_vec_copy_neg_elu * alpha_vec;
1248:             // blend
1249:             value = Vec::blendv(dx_vec_copy_neg_elu, value,
1250:                                         value > zero_vec);
1251:           }
1252:
1253:           value = value * scale_coef_vec;
1254:         }
1255:         // quantize
1256:         return qVec::quantize(dx_vec_vec, o_scale, o_zp, inv_o_scale);
1257:       }
1258:     );
1259:
1260:   });
1261: }
1262:
1263: // Note: out is assumed to be the same size as self and other.
1264: // Note: Addition is only supported when self and out are of the same dtype.
1265: // Note: other is already assumed to be in int32, i.e., it's
1266: // round(float/self_scale)
1267: template <bool ReLUFused = false>
1268: void qadd_scalar_kernel(Tensor& out, const Tensor& self, const Scalar& other) {
1269:   int64_t zero_point = out.q_zero_point();
1270:   float scale = static_cast<float>(out.q_scale());
1271:   float inv_scale = 1.0f / scale;
1272:   int64_t self_zero_point = self.q_zero_point();
1273:   float self_scale = static_cast<float>(self.q_scale());
1274:
1275:   float multiplier = self_scale * inv_scale;
1276:
1277:   AT_DISPATCH_QINT_TYPES(self.scalar_type(), "qadd_scalar", [&]() {
1278:     using Vec = Vectorized<scalar_t>;
1279:     auto iter = TensorIterator::unary_op(out, self);
1280:     auto other_val = other.to<int32_t>();
1281:     auto other_vec = Vectorized<c10::qint32>(static_cast<c10::qint32>(other_val));
1282:     cpu_kernel_vec(
1283:         iter,
1284:         [&](scalar_t a) -> scalar_t {
1285:           int32_t a_sub_z = static_cast<int32_t>(a.val_) -
1286:               static_cast<int32_t>(self_zero_point);
1287:           int32_t c = a_sub_z + other_val;
1288:           scalar_t res = at::native::requantize_from_int<scalar_t>(
1289:               multiplier, zero_point, c);
1290:           if constexpr (ReLUFused) {
1291:             res.val_ = std::max<scalar_t::underlying>(res.val_, zero_point);
1292:           }
1293:           return res;
1294:         },
1295:         [&](Vec a) -> Vec {
1296:           Vec::int_vec_return_type a_sub_z =
1297:               a.widening_subtract(Vec(static_cast<scalar_t>(self_zero_point)));
1298:           Vec::int_vec_return_type c;
1299:           for (const auto i : c10::irange(Vec::int_num_vecs())) {
1300:             c[i] = a_sub_z[i] + other_vec;
1301:           }
1302:           Vec rv = Vec::requantize_from_int(c, multiplier, zero_point);
1303:           if constexpr (ReLUFused) {
1304:             rv = rv.maximum(Vec(static_cast<scalar_t>(zero_point)));
1305:           }
1306:           return rv;
1307:         });
1308:   });
1309: }
1310: // Note: out is assumed to be the same size as self and other.
1311: // Note: Addition is only supported when self, other, out are of the same dtype.
1312: template <bool ReLUFused = false>
1313: void qadd_kernel(Tensor& out, const Tensor& self, const Tensor& other) {
1314:   int64_t zero_point = out.q_zero_point();
1315:   float scale = static_cast<float>(out.q_scale());
1316:   float inv_scale = 1.0f / scale;
1317:   int64_t self_zero_point = self.q_zero_point();
1318:   float self_scale = static_cast<float>(self.q_scale());
1319:   int64_t other_zero_point = other.q_zero_point();
1320:   float other_scale = static_cast<float>(other.q_scale());
```
- EN: The main symbol in this range is `round`, `constexpr`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `round`, `constexpr`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1322-1390
```cpp
1322:   // Broadcast out the parameters here to amortize out that cost across
1323:   // loop iterations.
1324:   // TODO: we can optimize dequantization by doing a premultiplication
1325:   // of the zero point by scale and doing FMA on scale*x_q - (scale*zero_point)
1326:   auto self_zero_point_vec = Vectorized<float>(self_zero_point);
1327:   auto self_scale_vec = Vectorized<float>(self_scale);
1328:   auto other_zero_point_vec = Vectorized<float>(other_zero_point);
1329:   auto other_scale_vec = Vectorized<float>(other_scale);
1330:
1331:   auto self_scale_neg_zp_premul_vec = self_scale_vec * self_zero_point_vec.neg();
1332:   auto other_scale_zp_premul_vec = other_scale_vec * other_zero_point_vec.neg();
1333:
1334:   auto iter = TensorIterator::borrowing_binary_op(out, self, other);
1335:
1336:   AT_DISPATCH_QINT_TYPES(out.scalar_type(), "qadd", [&]() {
1337:     using Vec = Vectorized<scalar_t>;
1338:     cpu_kernel_vec(
1339:         iter,
1340:         [&](scalar_t a, scalar_t b) -> scalar_t {
1341:           const auto da =
1342:               at::native::dequantize_val(self_scale, self_zero_point, a);
1343:           const auto db =
1344:               at::native::dequantize_val(other_scale, other_zero_point, b);
1345:           float c = da + db;
1346:           if (ReLUFused) {
1347:             c = std::max<float>(c, 0.0);
1348:           }
1349:           return at::native::quantize_val<scalar_t>(scale, zero_point, c);
1350:         },
1351:         [&](Vec a, Vec b) -> Vec {
1352:           const auto da = a.dequantize(
1353:               self_scale_vec, self_zero_point_vec, self_scale_neg_zp_premul_vec);
1354:           const auto db = b.dequantize(
1355:               other_scale_vec, other_zero_point_vec, other_scale_zp_premul_vec);
1356:           Vec::float_vec_return_type retvals;
1357:           for (const auto i : c10::irange(Vec::float_num_vecs())) {
1358:             auto c = da[i] + db[i];
1359:             if constexpr (ReLUFused) {
1360:               c = vec::maximum(c, Vectorized<float>(0.0f));
1361:             }
1362:             retvals[i] = c;
1363:           }
1364:           // TODO: fbgemm::Quantize doesn't support taking in the
1365:           // pre-broadcasted parameters. We might be able to save some cycles by
1366:           // enabling that in the API.
1367:           // TODO: specialize fbgemm::Quantize for a single vector and make it
1368:           // inlineable. This could help with interleaving as suggested by the
1369:           // TensorIterator implementations
1370:           auto rv = Vec::quantize(retvals, scale, zero_point, inv_scale);
1371:           return rv;
1372:         });
1373:   });
1374: }
1375:
1376: // Note: out is assumed to be the same size as self and other.
1377: // Note: Multiplication is only supported when self, other, out are of the same
1378: // dtype.
1379: template <bool ReLUFused = false>
1380: void qmul_kernel(Tensor& out, const Tensor& self, const Tensor& other) {
1381:   int64_t zero_point = out.q_zero_point();
1382:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
1383:   float scale = out.q_scale();
1384:   float inv_scale = 1.0f / scale;
1385:   int64_t self_zero_point = self.q_zero_point();
1386:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
1387:   float self_scale = self.q_scale();
1388:   int64_t other_zero_point = other.q_zero_point();
1389:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
1390:   float other_scale = other.q_scale();
```
- EN: The main symbol in this range is `constexpr`, `qmul_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `constexpr`, `qmul_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1392-1468
```cpp
1392:   float multiplier = self_scale * other_scale * inv_scale;
1393:
1394:   auto iter = TensorIterator::borrowing_binary_op(out, self, other);
1395:
1396:   AT_DISPATCH_QINT_TYPES(out.scalar_type(), "qmul", [&]() {
1397:     using Vec = Vectorized<scalar_t>;
1398:     cpu_kernel_vec(
1399:         iter,
1400:         [&](scalar_t a, scalar_t b) -> scalar_t {
1401:           int32_t a_sub_z = static_cast<int32_t>(a.val_) -
1402:               static_cast<int32_t>(self_zero_point);
1403:           int32_t b_sub_z = static_cast<int32_t>(b.val_) -
1404:               static_cast<int32_t>(other_zero_point);
1405:           int32_t c = a_sub_z * b_sub_z;
1406:           scalar_t res = at::native::requantize_from_int<scalar_t>(
1407:               multiplier, zero_point, c);
1408:           if constexpr (ReLUFused) {
1409:             res.val_ = std::max<scalar_t::underlying>(res.val_, zero_point);
1410:           }
1411:           return res;
1412:         },
1413:         [&](Vec a, Vec b) -> Vec {
1414:           Vec::int_vec_return_type a_sub_zp =
1415:               a.widening_subtract(Vec(static_cast<scalar_t>(self_zero_point)));
1416:           Vec::int_vec_return_type b_sub_zp =
1417:               b.widening_subtract(Vec(static_cast<scalar_t>(other_zero_point)));
1418:           Vec::int_vec_return_type c;
1419:           for (const auto i : c10::irange(Vec::int_num_vecs())) {
1420:             c[i] = a_sub_zp[i] * b_sub_zp[i];
1421:           }
1422:           Vec rv = Vec::requantize_from_int(c, multiplier, zero_point);
1423:           if constexpr (ReLUFused) {
1424:             rv = rv.maximum(Vec(static_cast<scalar_t>(zero_point)));
1425:           }
1426:           return rv;
1427:         });
1428:   });
1429: }
1430:
1431: template <typename scalar_t, typename scalar_t_underlying>
1432: void _qmaxpool_2d_nhwc_kernel(
1433:     const Tensor& qx,
1434:     int64_t iC, // input/output channels
1435:     int64_t iH,
1436:     int64_t iW, // input sizes
1437:     int64_t oH,
1438:     int64_t oW, // output sizes
1439:     int64_t kH,
1440:     int64_t kW, // kernel size
1441:     int64_t sH,
1442:     int64_t sW, // strides
1443:     int64_t pH,
1444:     int64_t pW, // padding
1445:     int64_t dH,
1446:     int64_t dW, // dilation
1447:     Tensor& qy) {
1448:     scalar_t* idata = static_cast<scalar_t*>(qx.data_ptr());
1449:     scalar_t* odata = static_cast<scalar_t*>(qy.data_ptr());
1450:
1451:     int64_t nBatch = qx.size(0);
1452:     at::parallel_for(0, nBatch * oH * oW, 0, [&](int64_t begin, int64_t end) {
1453:       int64_t b{0}, row{0}, col{0};
1454:       data_index_init(begin, b, nBatch, row, oH, col, oW);
1455:
1456:       for (const auto i : c10::irange(begin, end)) {
1457:         auto* i_p = reinterpret_cast<scalar_t_underlying*>(idata + b * iW * iH * iC);
1458:         auto* o_p = reinterpret_cast<scalar_t_underlying*>(odata + i * iC);
1459:
1460:         // Loop over reduction block
1461:         int64_t h_start = row * sH - pH;
1462:         int64_t w_start = col * sW - pW;
1463:         int64_t h_end = std::min(h_start + (kH - 1) * dH + 1, iH);
1464:         int64_t w_end = std::min(w_start + (kW - 1) * dW + 1, iW);
1465:         while (h_start < 0)
1466:           h_start += dH;
1467:         while (w_start < 0)
1468:           w_start += dW;
```
- EN: The main symbol in this range is `constexpr`, `_qmaxpool_2d_nhwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `constexpr`, `_qmaxpool_2d_nhwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1470-1557
```cpp
1470:         int64_t c = 0;
1471:
1472:         // Interleaved vector loop 4x
1473:         constexpr auto vec_width = Vectorized<scalar_t>::size();
1474:         for (; c + 4 * vec_width <= iC; c += 4 * vec_width) {
1475:           Vectorized<scalar_t> acc{
1476:               scalar_t(std::numeric_limits<scalar_t_underlying>::lowest())};
1477:           // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
1478:           Vectorized<scalar_t> accs[4] = {acc, acc, acc, acc};
1479:           int64_t tcntr = 0;
1480:           int64_t x, y;
1481:           for (y = h_start; y < h_end; y += dH) {
1482:             for (x = w_start; x < w_end; x += dW) {
1483:               for (const auto i : c10::irange(4)) {
1484:                 tcntr = y * iW + x;
1485:                 auto vals = Vectorized<scalar_t>::loadu(
1486:                     i_p + tcntr * iC + c + Vectorized<scalar_t>::size() * i);
1487:                 accs[i] = vec::maximum(accs[i], vals);
1488:               }
1489:             } // for x
1490:           } // for y
1491:           for (const auto i : c10::irange(4)) {
1492:             accs[i].store(o_p + c + Vectorized<scalar_t>::size() * i);
1493:           }
1494:         } // for c
1495:
1496:         // Vector loop
1497:         for (; c + vec_width <= iC; c += vec_width) {
1498:           Vectorized<scalar_t> acc{
1499:               scalar_t(std::numeric_limits<scalar_t_underlying>::lowest())};
1500:           int64_t tcntr = 0;
1501:           int64_t x, y;
1502:           for (y = h_start; y < h_end; y += dH) {
1503:             for (x = w_start; x < w_end; x += dW) {
1504:               tcntr = y * iW + x;
1505:               auto vals = Vectorized<scalar_t>::loadu(i_p + tcntr * iC + c);
1506:               acc = vec::maximum(acc, vals);
1507:             } // for x
1508:           } // for y
1509:           acc.store(o_p + c);
1510:         } // for c
1511:
1512:         for (; c < iC; ++c) {
1513:           auto max_val = std::numeric_limits<scalar_t_underlying>::lowest();
1514:           int64_t tcntr = 0;
1515:           int64_t x, y;
1516:           for (y = h_start; y < h_end; y += dH) {
1517:             for (x = w_start; x < w_end; x += dW) {
1518:               tcntr = y * iW + x;
1519:               auto val = *(i_p + tcntr * iC + c);
1520:               max_val = std::max(max_val, val);
1521:             } // for x
1522:           } // for y
1523:
1524:           o_p[c] = max_val;
1525:         } // for c
1526:
1527:         data_index_step(b, nBatch, row, oH, col, oW);
1528:       }
1529:     });
1530: }
1531:
1532: void qmaxpool_2d_nhwc_kernel(
1533:     const Tensor& qx,
1534:     int64_t iC, // input/output channels
1535:     int64_t iH,
1536:     int64_t iW, // input sizes
1537:     int64_t oH,
1538:     int64_t oW, // output sizes
1539:     int64_t kH,
1540:     int64_t kW, // kernel size
1541:     int64_t sH,
1542:     int64_t sW, // strides
1543:     int64_t pH,
1544:     int64_t pW, // padding
1545:     int64_t dH,
1546:     int64_t dW, // dilation
1547:     Tensor& qy) {
1548:   if (qx.scalar_type() == ScalarType::Byte) {
1549:     AT_DISPATCH_INTEGRAL_TYPES(qx.scalar_type(), "max_pool2d_nhwc", [&]() {
1550:       _qmaxpool_2d_nhwc_kernel<scalar_t, scalar_t>(qx, iC, iH, iW, oH, oW, kH, kW, sH, sW, pH, pW, dH, dW, qy);
1551:     });
1552:   } else {
1553:     AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "max_pool2d_nhwc", [&]() {
1554:       _qmaxpool_2d_nhwc_kernel<scalar_t, scalar_t::underlying>(qx, iC, iH, iW, oH, oW, kH, kW, sH, sW, pH, pW, dH, dW, qy);
1555:     });
1556:   }
1557: }
```
- EN: The main symbol in this range is `qmaxpool_2d_nhwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `qmaxpool_2d_nhwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 1559-1644
```cpp
1559: void qmaxpool_3d_nthwc_kernel(
1560:     const Tensor& qx,
1561:     int64_t iC, // input/output channels
1562:     int64_t iT,
1563:     int64_t iH,
1564:     int64_t iW, // input sizes
1565:     int64_t oT,
1566:     int64_t oH,
1567:     int64_t oW, // output sizes
1568:     int64_t kT,
1569:     int64_t kH,
1570:     int64_t kW, // kernel size
1571:     int64_t sT,
1572:     int64_t sH,
1573:     int64_t sW, // strides
1574:     int64_t pT,
1575:     int64_t pH,
1576:     int64_t pW, // padding
1577:     int64_t dT,
1578:     int64_t dH,
1579:     int64_t dW, // dilation
1580:     Tensor& qy) {
1581:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "max_pool3d_nthwc", [&]() {
1582:     scalar_t* idata = static_cast<scalar_t*>(qx.data_ptr());
1583:     scalar_t* odata = static_cast<scalar_t*>(qy.data_ptr());
1584:     int64_t nBatch = qx.size(0);
1585:     at::parallel_for(0, nBatch * oT * oH * oW, 0, [&](int64_t begin, int64_t end) {
1586:       int64_t b{0}, time{0}, row{0}, col{0};
1587:
1588:       data_index_init(begin, b, nBatch, time, oT, row, oH, col, oW);
1589:
1590:       for (const auto i : c10::irange(begin, end)) {
1591:         auto* i_p = reinterpret_cast<scalar_t::underlying*>(idata + b * iT * iW * iH * iC);
1592:         auto* o_p = reinterpret_cast<scalar_t::underlying*>(odata + i * iC);
1593:
1594:         // Loop over reduction block
1595:         int64_t t_start = time * sT - pT;
1596:         int64_t h_start = row * sH - pH;
1597:         int64_t w_start = col * sW - pW;
1598:         int64_t t_end = std::min(t_start + (kT - 1) * dT + 1, iT);
1599:         int64_t h_end = std::min(h_start + (kH - 1) * dH + 1, iH);
1600:         int64_t w_end = std::min(w_start + (kW - 1) * dW + 1, iW);
1601:         while (t_start < 0)
1602:           t_start += dT;
1603:         while (h_start < 0)
1604:           h_start += dH;
1605:         while (w_start < 0)
1606:           w_start += dW;
1607:
1608:         int64_t c = 0;
1609:         constexpr auto vec_width = Vectorized<scalar_t>::size();
1610:         // Vector loop
1611:         for (; c + vec_width <= iC; c += vec_width) {
1612:           Vectorized<scalar_t> acc{
1613:               scalar_t(std::numeric_limits<scalar_t::underlying>::lowest())};
1614:           int64_t tcntr = 0;
1615:           int64_t t, x, y;
1616:           for (t = t_start; t < t_end; t += dT) {
1617:             for (y = h_start; y < h_end; y += dH) {
1618:               for (x = w_start; x < w_end; x += dW) {
1619:                 tcntr = t * iH * iW + y * iW + x;
1620:                 auto vals = Vectorized<scalar_t>::loadu(i_p + tcntr * iC + c);
1621:                 acc = vec::maximum(acc, vals);
1622:               } // for x
1623:             } // for y
1624:           } // for t
1625:           acc.store(o_p + c);
1626:         } // for c
1627:
1628:         for (; c < iC; ++c) {
1629:           auto max_val = std::numeric_limits<scalar_t::underlying>::lowest();
1630:           int64_t tcntr = 0;
1631:           int64_t t, x, y;
1632:           for (t = t_start; t < t_end; t += dT) {
1633:             for (y = h_start; y < h_end; y += dH) {
1634:               for (x = w_start; x < w_end; x += dW) {
1635:                 tcntr = t * iH * iW + y * iW + x;
1636:                 auto val = *(i_p + tcntr * iC + c);
1637:                 max_val = std::max(max_val, val);
1638:               } // for x
1639:             } // for y
1640:           } // for t
1641:           o_p[c] = max_val;
1642:         } // for c
1643:         data_index_step(b, nBatch, time, oT, row, oH, col, oW);
1644:       }
```
- EN: The main symbol in this range is `qmaxpool_3d_nthwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 这一段的主要符号是 `qmaxpool_3d_nthwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 1646-1734
```cpp
1646:     });
1647:
1648:   });
1649: }
1650:
1651: template <typename T>
1652: void do_avg_pool_nhwc_on_AVX_n(
1653:     const typename T::underlying* i_p,
1654:     typename T::underlying* o_p,
1655:     int& c_start,
1656:     int input_zero_point_m_size,
1657:     int output_zero_point,
1658:     float multiplier,
1659:     int dstart,
1660:     int dend,
1661:     int hstart,
1662:     int hend,
1663:     int wstart,
1664:     int wend,
1665:     int dsize,
1666:     int hsize,
1667:     int wsize,
1668:     int csize) {
1669: #if (defined(CPU_CAPABILITY_AVX2) || defined(CPU_CAPABILITY_AVX512)) && !defined(_MSC_VER)
1670:   // buffer for channel accumulator, used to interchange channel-loop
1671:   // to inner-most, so that memory access of the input tensor data is
1672:   // continuous.
1673: #ifdef CPU_CAPABILITY_AVX2
1674:   constexpr int cb_size = 16;
1675: #else
1676:   constexpr int cb_size = 8;
1677: #endif
1678:   constexpr int vec_width = Vectorized<T>::size() / 4;
1679:   constexpr int cb_step = cb_size * vec_width;
1680:   Vectorized<int32_t> acc_buffer[cb_size];
1681:   Vectorized<float> acc_buffer_fp[cb_size];
1682:
1683: #ifdef CPU_CAPABILITY_AVX2
1684:   if (vec_width == 8) {
1685: #else
1686:   if (vec_width == 16) {
1687: #endif
1688:     for (int c = c_start; c < csize; c += cb_step) {
1689:       int cend = std::min(cb_size, (csize - c) / vec_width);
1690:       // initialize loop
1691:       for (const auto ic : c10::irange(cend)) {
1692:         acc_buffer[ic] = Vectorized<int32_t>(input_zero_point_m_size);
1693:       }
1694:       // compute loop
1695:       for (const auto id : c10::irange(dstart, dend)) {
1696:         for (const auto ih : c10::irange(hstart, hend)) {
1697:           for (const auto iw : c10::irange(wstart, wend)) {
1698:             const int i_idx =
1699:                 (id * wsize * hsize + ih * wsize + iw) *
1700:                     csize +
1701:                 c;
1702:             for (const auto ic : c10::irange(cend)) {
1703:               auto vals = vec::convert_to_int32<typename T::underlying>(
1704:                   i_p + i_idx + ic * vec_width);
1705:               acc_buffer[ic] = acc_buffer[ic] + vals;
1706:             }
1707:           }
1708:         }
1709:       }
1710:       // convert int32 accumulative to fp32
1711:       vec::convert((int*)acc_buffer, (float*)acc_buffer_fp, cend * vec_width);
1712:
1713:       // first quantize using AVX2 or AVX512 using 32 lanes, then 8, finally falls
1714:       // back to single
1715: #ifdef CPU_CAPABILITY_AVX2
1716:       QuantizeAvx2<typename T::underlying>(
1717:           (float*)acc_buffer_fp,
1718:           o_p + c,
1719:           cend * vec_width,
1720:           multiplier,
1721:           output_zero_point);
1722: #else
1723:       QuantizeAvx512<typename T::underlying>(
1724:           (float*)acc_buffer_fp,
1725:           o_p + c,
1726:           cend * vec_width,
1727:           multiplier,
1728:           output_zero_point);
1729: #endif
1730:     }
1731:     c_start = csize / vec_width * vec_width;
1732:   }
1733: #endif
1734: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `do_avg_pool_nhwc_on_AVX_n`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `do_avg_pool_nhwc_on_AVX_n`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 1736-1808
```cpp
1736: template <typename T>
1737: void do_avg_pool_on_AVX_n(
1738:     typename T::underlying* i_p,
1739:     typename T::underlying* o_p,
1740:     int64_t& c,
1741:     int64_t channel_size,
1742:     int64_t channel_multiplier,
1743:     int32_t input_zero_point_m_size,
1744:     int32_t output_zero_point,
1745:     float multiplier,
1746:     int64_t dstart,
1747:     int64_t dend,
1748:     int64_t hstart,
1749:     int64_t hend,
1750:     int64_t wstart,
1751:     int64_t wend,
1752:     int64_t stride_C,
1753:     int64_t stride_D,
1754:     int64_t stride_H,
1755:     int64_t stride_W) {
1756: #if (defined(CPU_CAPABILITY_AVX2) || defined(CPU_CAPABILITY_AVX512)) && !defined(_MSC_VER)
1757:   constexpr int vec_width = Vectorized<T>::size() / 4;
1758: #ifdef CPU_CAPABILITY_AVX2
1759:   if (vec_width == 8) {
1760: #else
1761:   if (vec_width == 16) {
1762: #endif
1763:     for (; c + vec_width <= channel_size; c += vec_width) {
1764:       int64_t tcntr = 0;
1765:
1766:       Vectorized<int32_t> acc(input_zero_point_m_size);
1767:       for (const auto id : c10::irange(dstart, dend)) {
1768:         for (const auto ih : c10::irange(hstart, hend)) {
1769:           for (const auto iw : c10::irange(wstart, wend)) {
1770:             tcntr = id * stride_D + ih * stride_H + iw * stride_W;
1771:             auto vals = vec::convert_to_int32<typename T::underlying>(
1772:                 i_p + tcntr * channel_multiplier + c * stride_C);
1773:             acc = acc + vals;
1774:           }
1775:         }
1776:       }
1777:       int32_t acc_int[vec_width];
1778:       float acc_fp[vec_width];
1779:       acc.store(acc_int);
1780:       vec::convert(acc_int, acc_fp, vec_width);
1781:       at::native::quantize_vec<T>(
1782:           1.0f / multiplier,
1783:           output_zero_point,
1784:           acc_fp,
1785:           reinterpret_cast<T*>(o_p + c),
1786:           vec_width);
1787:     }
1788:   }
1789: #endif
1790: }
1791:
1792: template <typename T>
1793: void _qadaptive_avg_pool_kernel(
1794:     const Tensor& qx,
1795:     Tensor& qy,
1796:     int64_t nBatch,
1797:     int64_t sizeC,
1798:     int64_t isizeD,  // Set to 1 for 2d
1799:     int64_t isizeH,
1800:     int64_t isizeW,
1801:     int64_t osizeD,  // Set to 1 for 2d
1802:     int64_t osizeH,
1803:     int64_t osizeW,
1804:     int64_t istrideB,
1805:     int64_t istrideC,
1806:     int64_t istrideD,  // Set to 1 for 2d
1807:     int64_t istrideH,
1808:     int64_t istrideW) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `do_avg_pool_on_AVX_n`, `_qadaptive_avg_pool_kernel`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `do_avg_pool_on_AVX_n`, `_qadaptive_avg_pool_kernel`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 1810-1901
```cpp
1810:   T* idata = static_cast<T*>(qx.data_ptr());
1811:   T* odata = static_cast<T*>(qy.data_ptr());
1812:
1813:   const float input_scale = qx.q_scale();
1814:   const float output_scale = qy.q_scale();
1815:   const int input_zero_point = qx.q_zero_point();
1816:   const int output_zero_point = qy.q_zero_point();
1817:
1818:   at::parallel_for(0, nBatch, 0, [&](int64_t batch_start, int64_t batch_end) {
1819:     for (const auto b : c10::irange(batch_start, batch_end)) {
1820:       auto* i_p = reinterpret_cast<typename T::underlying*>(
1821:           idata + b * istrideB);
1822:
1823:       for (const auto od : c10::irange(osizeD)) {
1824:         int istartD = (int)std::floor((float)(od * isizeD) / osizeD);
1825:         int iendD = (int)std::ceil((float)((od + 1) * isizeD) / osizeD);
1826:         int kD = iendD - istartD;
1827:         for (const auto oh : c10::irange(osizeH)) {
1828:           int istartH = (int)std::floor((float)(oh * isizeH) / osizeH);
1829:           int iendH = (int)std::ceil((float)((oh + 1) * isizeH) / osizeH);
1830:           int kH = iendH - istartH;
1831:           for (const auto ow : c10::irange(osizeW)) {
1832:             auto* o_p = reinterpret_cast<typename T::underlying*>(
1833:                 odata +
1834:                 b * osizeD * osizeH * osizeW * sizeC +
1835:                 od * osizeH * osizeW * sizeC +
1836:                 oh * osizeW * sizeC +
1837:                 ow * sizeC);
1838:             int istartW = (int)std::floor((float)(ow * isizeW) / osizeW);
1839:             int iendW = (int)std::ceil((float)((ow + 1) * isizeW) / osizeW);
1840:             int kW = iendW - istartW;
1841:             int size = kD * kH * kW;
1842:             float multiplier = input_scale / output_scale / size;
1843:             int input_zero_point_m_size = -input_zero_point * size;
1844:             int64_t c = 0;
1845:             // For int8 or uint8quantization, we implicitly use int32 as
1846:             // accumulation Or else, it will go to the slow path
1847:             // TODO: support 16bit, 32bit, and etc.
1848:             auto* internal_i_p = i_p +
1849:                                 istartD * istrideD +
1850:                                 istartH * istrideH +
1851:                                 istartW * istrideW;
1852:
1853:             // Note: If AVX is not available, `do_avg_pool_on_AVX_n is a noop.
1854:             //       In that case, the following loop takes over
1855:             // TODO: more vectorization with loop interleaving
1856:             do_avg_pool_on_AVX_n<T>(
1857:                 internal_i_p,
1858:                 o_p,
1859:                 c,
1860:                 sizeC,
1861:                 1,
1862:                 input_zero_point_m_size,
1863:                 output_zero_point,
1864:                 multiplier,
1865:                 0,
1866:                 kD,
1867:                 0,
1868:                 kH,
1869:                 0,
1870:                 kW,
1871:                 istrideC,
1872:                 istrideD,
1873:                 istrideH,
1874:                 istrideW);
1875:             // 1) The following loop handles the remaining channels
1876:             // 2) It also handles the Non-AVX2 path
1877:             for (; c < sizeC; ++c) {
1878:               int32_t acc_int32 = input_zero_point_m_size;
1879:               int64_t tcntr = 0;
1880:               for (const auto id : c10::irange(kD)) {
1881:                 for (const auto ih : c10::irange(kH)) {
1882:                   for (const auto iw : c10::irange(kW)) {
1883:                     tcntr = id * istrideD +
1884:                         ih * istrideH +
1885:                         iw * istrideW;
1886:                     auto val = *(internal_i_p + tcntr + c * istrideC);
1887:                     acc_int32 += val;
1888:                   }
1889:                 }
1890:               }
1891:               // clamp
1892:               o_p[c] = at::native::quantize_val<T>(1.0f / multiplier,
1893:                                                           output_zero_point,
1894:                                                           acc_int32).val_;
1895:             } // c
1896:           } // oh
1897:         } // ow
1898:       } // od
1899:     }
1900:   });
1901: }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1903-1972
```cpp
1903: void qadaptive_avg_pool2d_nhwc_kernel(
1904:     const Tensor& qx,
1905:     Tensor& qy,
1906:     int64_t nBatch,
1907:     int64_t sizeC,
1908:     int64_t isizeH,
1909:     int64_t isizeW,
1910:     int64_t osizeH,
1911:     int64_t osizeW,
1912:     int64_t istrideB,
1913:     int64_t istrideC,
1914:     int64_t istrideH,
1915:     int64_t istrideW) {
1916:     AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "adaptive_avg_pool2d_nhwc", [&]() {
1917:         _qadaptive_avg_pool_kernel<scalar_t>(
1918:           qx,
1919:           qy,
1920:           nBatch,
1921:           sizeC,
1922:           /*isizeD=*/1,
1923:           isizeH,
1924:           isizeW,
1925:           /*osizeD=*/1,
1926:           osizeH,
1927:           osizeW,
1928:           istrideB,
1929:           istrideC,
1930:           /*istrideD=*/1,
1931:           istrideH,
1932:           istrideW);
1933:       }
1934:     );
1935: }
1936:
1937: void qadaptive_avg_pool3d_ndhwc_kernel(
1938:     const Tensor& qx,
1939:     Tensor& qy,
1940:     int64_t nBatch,
1941:     int64_t sizeC,
1942:     int64_t isizeD,
1943:     int64_t isizeH,
1944:     int64_t isizeW,
1945:     int64_t osizeD,
1946:     int64_t osizeH,
1947:     int64_t osizeW,
1948:     int64_t istrideB,
1949:     int64_t istrideC,
1950:     int64_t istrideD,
1951:     int64_t istrideH,
1952:     int64_t istrideW) {
1953:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "adaptive_avg_pool3d_ndhwc", [&]() {
1954:     _qadaptive_avg_pool_kernel<scalar_t>(
1955:       qx,
1956:       qy,
1957:       nBatch,
1958:       sizeC,
1959:       isizeD,
1960:       isizeH,
1961:       isizeW,
1962:       osizeD,
1963:       osizeH,
1964:       osizeW,
1965:       istrideB,
1966:       istrideC,
1967:       istrideD,
1968:       istrideH,
1969:       istrideW);
1970:     }
1971:   );
1972: }
```
- EN: The main symbol in this range is `qadaptive_avg_pool2d_nhwc_kernel`, `qadaptive_avg_pool3d_ndhwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qadaptive_avg_pool2d_nhwc_kernel`, `qadaptive_avg_pool3d_ndhwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1974-2043
```cpp
1974: template <typename T>
1975: void _qavg_pool_nhwc_kernel(
1976:     const Tensor& qx,
1977:     Tensor& qy,
1978:     int64_t nBatch,
1979:     int64_t nInputPlane,
1980:     int64_t inputWidth,
1981:     int64_t inputHeight,
1982:     int64_t inputDepth,
1983:     int64_t outputWidth,
1984:     int64_t outputHeight,
1985:     int64_t outputDepth,
1986:     int kW,
1987:     int kH,
1988:     int kD,
1989:     int dW,
1990:     int dH,
1991:     int dD,
1992:     int padW,
1993:     int padH,
1994:     int padD,
1995:     bool count_include_pad,
1996:     std::optional<int64_t> divisor_override) {
1997:   T* idata = static_cast<T*>(qx.data_ptr());
1998:   T* odata = static_cast<T*>(qy.data_ptr());
1999:   int strideC = 1;
2000:   int strideW = strideC * nInputPlane;
2001:   int istrideH = strideW * inputWidth;
2002:   int istrideD = istrideH * inputHeight;
2003:   int istrideB = istrideD * inputDepth;
2004:
2005:   // lift these operations outside the loop to reduce access overheads
2006:   float input_scale = qx.q_scale();
2007:   float output_scale = qy.q_scale();
2008:   int input_zero_point = qx.q_zero_point();
2009:   int output_zero_point = qy.q_zero_point();
2010:   int64_t divisor_override_factor =
2011:       divisor_override.has_value() ? divisor_override.value() : 0;
2012:
2013:   at::parallel_for(0, nBatch * outputDepth * outputHeight * outputWidth, 0, [&](int64_t begin, int64_t end) {
2014:     int64_t b{0}, od{0}, oh{0}, ow{0};
2015:     data_index_init(begin, b, nBatch, od, outputDepth, oh, outputHeight, ow, outputWidth);
2016:
2017:     for (const auto i : c10::irange(begin, end)) {
2018:       auto* i_p = reinterpret_cast<typename T::underlying*>(idata + b * istrideB);
2019:       auto* o_p = reinterpret_cast<typename T::underlying*>(odata + i * strideW);
2020:       int dstart = od * dD - padD;
2021:       int hstart = oh * dH - padH;
2022:       int wstart = ow * dW - padW;
2023:
2024:       int dend = std::min(dstart + kD, (int)inputDepth + padD);
2025:       int hend = std::min(hstart + kH, (int)inputHeight + padH);
2026:       int wend = std::min(wstart + kW, (int)inputWidth + padW);
2027:       int pool_size = (dend - dstart) * (hend - hstart) * (wend - wstart);
2028:
2029:       dstart = std::max(dstart, 0);
2030:       hstart = std::max(hstart, 0);
2031:       wstart = std::max(wstart, 0);
2032:       dend = std::min(dend, (int)inputDepth);
2033:       hend = std::min(hend, (int)inputHeight);
2034:       wend = std::min(wend, (int)inputWidth);
2035:
2036:       int size = (dend - dstart) * (hend - hstart) * (wend - wstart);
2037:       int divide_size = count_include_pad ? pool_size : size;
2038:       int divide_factor =
2039:           divisor_override_factor ? divisor_override_factor : divide_size;
2040:       float multiplier = input_scale / output_scale  / divide_factor;
2041:       int input_zero_point_m_size = -input_zero_point * size;
2042:
2043:       int c_start = 0;
```
- EN: The main symbol in this range is `_qavg_pool_nhwc_kernel`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `_qavg_pool_nhwc_kernel`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 2045-2133
```cpp
2045:       // For int8 quantization, we implicitly use int32 as accumulation
2046:       // Or else, it will go to the slow path
2047:       // TODO: support 16bit, 32bit, and etc.
2048:       do_avg_pool_nhwc_on_AVX_n<T>(
2049:           i_p,
2050:           o_p,
2051:           c_start,
2052:           input_zero_point_m_size,
2053:           output_zero_point,
2054:           multiplier,
2055:           dstart,
2056:           dend,
2057:           hstart,
2058:           hend,
2059:           wstart,
2060:           wend,
2061:           inputDepth,
2062:           inputHeight,
2063:           inputWidth,
2064:           nInputPlane);
2065:
2066:       // 1) The following loop handles the remaining channels
2067:       // 2) It also handles the Non-AVX2 path
2068:       for (const auto c: c10::irange(c_start, nInputPlane)) {
2069:         int32_t acc_int32 = input_zero_point_m_size;
2070:         for (const auto id : c10::irange(dstart, dend)) {
2071:           for (const auto ih : c10::irange(hstart, hend)) {
2072:             for (const auto iw : c10::irange(wstart, wend)) {
2073:               auto val =
2074:                   *(i_p + id * istrideD + ih * istrideH + iw * strideW +
2075:                   c * strideC);
2076:               acc_int32 += val;
2077:             }
2078:           }
2079:        }
2080:        double acc_fp = acc_int32 * 1.0;
2081:        // clamp
2082:        o_p[c] = at::native::quantize_val<T>(
2083:            1.0f / multiplier, output_zero_point, acc_fp)
2084:            .val_;
2085:       } // c
2086:
2087:       data_index_step(b, nBatch, od, outputDepth, oh, outputHeight, ow, outputWidth);
2088:     }
2089:   });
2090: }
2091:
2092: void qavg_pool2d_nhwc_kernel(
2093:     const Tensor& qx,
2094:     Tensor& qy,
2095:     int64_t b,
2096:     int64_t nInputPlane,
2097:     int64_t inputWidth,
2098:     int64_t inputHeight,
2099:     int64_t outputWidth,
2100:     int64_t outputHeight,
2101:     int kW,
2102:     int kH,
2103:     int dW,
2104:     int dH,
2105:     int padW,
2106:     int padH,
2107:     bool count_include_pad,
2108:     std::optional<int64_t> divisor_override) {
2109:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "avg_pool2d_nhwc", [&]() {
2110:     _qavg_pool_nhwc_kernel<scalar_t>(
2111:       qx,
2112:       qy,
2113:       b,
2114:       nInputPlane,
2115:       inputWidth,
2116:       inputHeight,
2117:       1,
2118:       outputWidth,
2119:       outputHeight,
2120:       1,
2121:       kW,
2122:       kH,
2123:       1,
2124:       dW,
2125:       dH,
2126:       1,
2127:       padW,
2128:       padH,
2129:       0,
2130:       count_include_pad,
2131:       divisor_override);
2132:   });
2133: }
```
- EN: The main symbol in this range is `qavg_pool2d_nhwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `qavg_pool2d_nhwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 2135-2203
```cpp
2135: void qavg_pool3d_nhwc_kernel(
2136:     const Tensor& qx,
2137:     Tensor& qy,
2138:     int64_t b,
2139:     int64_t nInputPlane,
2140:     int64_t inputWidth,
2141:     int64_t inputHeight,
2142:     int64_t inputDepth,
2143:     int64_t outputWidth,
2144:     int64_t outputHeight,
2145:     int64_t outputDepth,
2146:     int kW,
2147:     int kH,
2148:     int kD,
2149:     int dW,
2150:     int dH,
2151:     int dD,
2152:     int padW,
2153:     int padH,
2154:     int padD,
2155:     bool count_include_pad,
2156:     std::optional<int64_t> divisor_override) {
2157:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "avg_pool3d_nhwc", [&]() {
2158:     _qavg_pool_nhwc_kernel<scalar_t>(
2159:       qx,
2160:       qy,
2161:       b,
2162:       nInputPlane,
2163:       inputWidth,
2164:       inputHeight,
2165:       inputDepth,
2166:       outputWidth,
2167:       outputHeight,
2168:       outputDepth,
2169:       kW,
2170:       kH,
2171:       kD,
2172:       dW,
2173:       dH,
2174:       dD,
2175:       padW,
2176:       padH,
2177:       padD,
2178:       count_include_pad,
2179:       divisor_override);
2180:   });
2181: }
2182:
2183: template <typename T>
2184: int64_t do_quantized_bilinear_on_AVX_n(
2185:     const typename T::underlying*& pos1,
2186:     typename T::underlying*& pos2,
2187:     int64_t input_width,
2188:     int64_t output_height,
2189:     int64_t output_width,
2190:     int64_t channels,
2191:     int32_t output_zero_point,
2192:     int32_t input_zero_point,
2193:     float inverse_scale,
2194:     const float h0lambda,
2195:     const float h1lambda,
2196:     const float w0lambda,
2197:     const float w1lambda,
2198:     const int64_t h1p,
2199:     const int64_t w1p) {
2200:   int64_t c = 0;
2201: #if (defined(CPU_CAPABILITY_AVX2) || defined(CPU_CAPABILITY_AVX512)) && !defined(_MSC_VER)
2202:   constexpr auto vec_width = Vectorized<T>::size() / 4;
2203: #ifdef CPU_CAPABILITY_AVX2
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `qavg_pool3d_nhwc_kernel`, `do_quantized_bilinear_on_AVX_n`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `qavg_pool3d_nhwc_kernel`, `do_quantized_bilinear_on_AVX_n`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 2204-2275
```cpp
2204:   if (vec_width == 8) {
2205: #else
2206:   if (vec_width == 16) {
2207: #endif
2208:     for (; c + vec_width <= channels; c += vec_width) {
2209:       Vectorized<float> pos1_fp_v[4];
2210:       Vectorized<int32_t> pos1_int_v[4];
2211:       pos1_int_v[0] = vec::convert_to_int32<typename T::underlying>(pos1);
2212:       pos1_int_v[1] = vec::convert_to_int32<typename T::underlying>(
2213:           pos1 + w1p * channels);
2214:       pos1_int_v[2] = vec::convert_to_int32<typename T::underlying>(
2215:           pos1 + h1p * input_width * channels);
2216:       pos1_int_v[3] = vec::convert_to_int32<typename T::underlying>(
2217:           pos1 + (h1p * input_width + w1p) * channels);
2218:       for (const auto i : c10::irange(4)) {
2219:         int32_t pos1_int[vec_width];
2220:         float pos1_fp[vec_width];
2221:         pos1_int_v[i].store(pos1_int);
2222:         vec::convert(pos1_int, pos1_fp, vec_width);
2223:         pos1_fp_v[i] = Vectorized<float>::loadu(pos1_fp, 8);
2224:       }
2225:       Vectorized<float> h0lambda_v(h0lambda);
2226:       Vectorized<float> h1lambda_v(h1lambda);
2227:       Vectorized<float> w0lambda_v(w0lambda);
2228:       Vectorized<float> w1lambda_v(w1lambda);
2229:       Vectorized<float> input_zero_point_v(input_zero_point);
2230:       Vectorized<float> result =
2231:           h0lambda_v * (w0lambda_v * pos1_fp_v[0] + w1lambda_v * pos1_fp_v[1]) +
2232:           h1lambda_v * (w0lambda_v * pos1_fp_v[2] + w1lambda_v * pos1_fp_v[3]) -
2233:           input_zero_point_v;
2234:       float result_fp[vec_width];
2235:       result.store(result_fp);
2236:       at::native::quantize_vec<T>(
2237:           inverse_scale,
2238:           output_zero_point,
2239:           result_fp,
2240:           reinterpret_cast<T*>(pos2),
2241:           vec_width);
2242:       pos1 += vec_width;
2243:       pos2 += vec_width;
2244:     }
2245:   }
2246: #endif
2247:   return c;
2248: }
2249:
2250: void qupsample_bilinear2d_nhwc_kernel(
2251:     Tensor& output,
2252:     const Tensor& input,
2253:     int64_t input_height,
2254:     int64_t input_width,
2255:     int64_t output_height,
2256:     int64_t output_width,
2257:     int64_t nbatch,
2258:     int64_t channels,
2259:     bool align_corners,
2260:     std::optional<double> scales_h,
2261:     std::optional<double> scales_w) {
2262:   AT_DISPATCH_QINT_TYPES(input.scalar_type(), "upsample_bilinear2d_nhwc", [&]() {
2263:     auto* idata = static_cast<scalar_t*>(input.data_ptr());
2264:     auto* odata = static_cast<scalar_t*>(output.data_ptr());
2265:     float inverse_scale = output.q_scale() / input.q_scale();
2266:     const auto rheight = area_pixel_compute_scale<float>(
2267:         input_height, output_height, align_corners, scales_h);
2268:     const auto rwidth = area_pixel_compute_scale<float>(
2269:         input_width, output_width, align_corners, scales_w);
2270:
2271:     auto input_q_zero_point = input.q_zero_point();
2272:     auto output_q_zero_point = output.q_zero_point();
2273:     at::parallel_for(0, nbatch * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
2274:       int64_t b{0}, h2{0}, w2{0};
2275:       data_index_init(begin, b, nbatch, h2, output_height, w2, output_width);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `qupsample_bilinear2d_nhwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `qupsample_bilinear2d_nhwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 2277-2361
```cpp
2277:       for ([[maybe_unused]] const auto i : c10::irange(begin, end)) {
2278:         auto* i_p = reinterpret_cast<typename scalar_t::underlying*>(
2279:             idata + b * input_height * input_width * channels);
2280:         auto* o_p = reinterpret_cast<typename scalar_t::underlying*>(
2281:             odata + b * output_height * output_width * channels);
2282:
2283:         const auto h1r = area_pixel_compute_source_index<float>(
2284:             rheight, h2, align_corners, /*cubic=*/false);
2285:
2286:         const int64_t h1 = h1r;
2287:         const int64_t h1p = (h1 < input_height - 1) ? 1 : 0;
2288:         const float h1lambda = h1r - h1;
2289:         const float h0lambda = static_cast<float>(1.) - h1lambda;
2290:
2291:         const auto w1r = area_pixel_compute_source_index<float>(
2292:             rwidth, w2, align_corners, /*cubic=*/false);
2293:         const int64_t w1 = w1r;
2294:         const int64_t w1p = (w1 < input_width - 1) ? 1 : 0;
2295:
2296:         const float w1lambda = w1r - w1;
2297:         const float w0lambda = static_cast<float>(1.) - w1lambda;
2298:
2299:         int64_t c = 0;
2300:         // We use float32 to do the computation
2301:         const typename scalar_t::underlying* pos1 =
2302:             i_p + (h1 * input_width + w1) * channels;
2303:         typename scalar_t::underlying* pos2 =
2304:             o_p + (h2 * output_width + w2) * channels;
2305:         // We have to isolate this function out because the VS does not
2306:         // expand the macro correctly.
2307:         c = do_quantized_bilinear_on_AVX_n<scalar_t>(
2308:             pos1,
2309:             pos2,
2310:             input_width,
2311:             output_height,
2312:             output_width,
2313:             channels,
2314:             output_q_zero_point,
2315:             input_q_zero_point,
2316:             inverse_scale,
2317:             h0lambda,
2318:             h1lambda,
2319:             w0lambda,
2320:             w1lambda,
2321:             h1p,
2322:             w1p);
2323:         // 1) The following loop handles the remaining channels
2324:         // 2) It also handles the Non-AVX2 path
2325:         for (; c < channels; ++c) {
2326:           float result = h0lambda *
2327:                   (w0lambda * pos1[0] + w1lambda * pos1[w1p * channels]) +
2328:               h1lambda *
2329:                   (w0lambda * pos1[h1p * input_width * channels] +
2330:                    w1lambda * pos1[(h1p * input_width + w1p) * channels]);
2331:           pos2[0] = at::native::quantize_val<scalar_t>(
2332:                         inverse_scale,
2333:                         output_q_zero_point,
2334:                         result - input_q_zero_point)
2335:                         .val_;
2336:           pos1 += 1;
2337:           pos2 += 1;
2338:         } // c
2339:
2340:         data_index_step(b, nbatch, h2, output_height, w2, output_width);
2341:       }
2342:     });
2343:   });
2344: }
2345:
2346: void qtopk_kernel(Tensor& values,
2347:     Tensor& indices,
2348:     const Tensor& self,
2349:     int64_t k,
2350:     int64_t dim,
2351:     bool largest,
2352:     bool sorted) {
2353:   auto sizes = self.sizes();
2354:   auto iter = TensorIteratorConfig()
2355:     .check_all_same_dtype(false)
2356:     .resize_outputs(false)
2357:     .declare_static_shape(sizes, /*squash_dims=*/dim)
2358:     .add_output(values)
2359:     .add_output(indices)
2360:     .add_input(self)
2361:     .build();
```
- EN: The main symbol in this range is `qtopk_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `qtopk_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 2363-2433
```cpp
2363:   auto mode_values_stride = values.strides()[dim];
2364:   auto mode_indices_stride = indices.strides()[dim];
2365:   auto tmp_values_stride = self.strides()[dim];
2366:   // If sizes is empty, the tensor is scalar. This prevents accessing an empty array.
2367:   auto dim_size = sizes.empty() ? 1 : sizes[dim];
2368:
2369:   AT_DISPATCH_QINT_TYPES(self.scalar_type(), "qtopk_cpu", [&] {
2370:     auto loop = [&](char** data, const int64_t* strides, int64_t n) {
2371:       using underlying_t = typename scalar_t::underlying;
2372:       static_assert(sizeof(scalar_t) == sizeof(underlying_t), "");
2373:       return topk_impl_loop<underlying_t, underlying_t>(
2374:           mode_values_stride, mode_indices_stride, tmp_values_stride,
2375:           k, dim_size, largest, sorted, data, strides, n);
2376:     };
2377:
2378:     int64_t grain_size = internal::GRAIN_SIZE / std::max(int64_t{1}, sizes[dim]);
2379:     iter.for_each(loop, /*grain_size=*/grain_size);
2380:   });
2381: }
2382:
2383: template <typename T, bool ReluFused>
2384: inline void do_bn_compute(
2385:     typename T::underlying* X_ptr,
2386:     typename T::underlying* Y_ptr,
2387:     Vectorized<float> & fake_scale,
2388:     Vectorized<float> & in_zp_vec,
2389:     Vectorized<float> & scale_neg_zp_premul,
2390:     int64_t out_zero_point,
2391:     Vectorized<T> & out_zero_point_v,
2392:     float*  alpha,
2393:     float* beta,
2394:     int64_t vec_num,
2395:     int64_t kVLen
2396: ) {
2397:   using Vec = Vectorized<T>;
2398:   auto vals_q = Vec::loadu(X_ptr);
2399:   // Fake scale of 1.0 here, should not affect performance (FMA in place of sub)
2400:   auto vals_dq = vals_q.dequantize(fake_scale, in_zp_vec, scale_neg_zp_premul);
2401:   for (const auto idx : c10::irange(vec_num)) {
2402:     auto alpha_v = Vectorized<float>::loadu(alpha + idx * kVLen);
2403:     auto beta_v = Vectorized<float>::loadu(beta + idx * kVLen);
2404:     vals_dq[idx] = vec::fmadd(alpha_v, vals_dq[idx], beta_v);
2405:   }
2406:   auto outputs_q = Vec::quantize(vals_dq, /*scale=*/1.0f, out_zero_point, /*inverse_scale=*/1.0f);
2407:   // Fake scale again
2408:   if constexpr (ReluFused) {
2409:     outputs_q = outputs_q.maximum(out_zero_point_v);
2410:   }
2411:   outputs_q.store(Y_ptr, vec_num * kVLen);
2412: }
2413:
2414: template <bool ReluFused>
2415: void q_batch_norm_kernel(
2416:     int64_t N,
2417:     int64_t C,
2418:     int64_t HxW,
2419:     int64_t in_zero_point,
2420:     int64_t out_zero_point,
2421:     const Tensor& input,
2422:     const Tensor& a,
2423:     const Tensor& b,
2424:     Tensor& output) {
2425:
2426:   AT_DISPATCH_QINT_TYPES(input.scalar_type(), "qbatch_norm", [&]() {
2427:     float* alpha = a.data_ptr<float>();
2428:     float* beta = b.data_ptr<float>();
2429:     auto minimum = std::numeric_limits<scalar_t::underlying>::lowest();
2430:     auto maximum = std::numeric_limits<scalar_t::underlying>::max();
2431:     scalar_t::underlying* X =
2432:         reinterpret_cast<scalar_t::underlying*>(input.data_ptr());
2433:     scalar_t::underlying* Y = reinterpret_cast<scalar_t::underlying*>(output.data_ptr());
```
- EN: The main symbol in this range is `do_bn_compute`, `constexpr`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `do_bn_compute`, `constexpr`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 2435-2514
```cpp
2435:     constexpr int kVLen = Vectorized<float>::size();
2436:     const int64_t outer_size = N * HxW;
2437:     using Vec = Vectorized<scalar_t>;
2438:     // Hoisted variables
2439:     auto in_zp_vec = Vectorized<float>(static_cast<float>(in_zero_point));
2440:     auto fake_scale = Vectorized<float>(1.0f);
2441:     auto scale_neg_zp_premul = fake_scale * in_zp_vec.neg();
2442:     auto out_zero_point_v = Vec(scalar_t(out_zero_point));
2443:     const auto lanes = static_cast<int64_t>(Vec::float_num_vecs() * kVLen);
2444:     at::parallel_for(0, outer_size, 0, [&](int64_t begin, int64_t end) {
2445:       for (const auto i : c10::irange(begin, end)) {
2446:         auto* X_ptr = reinterpret_cast<typename scalar_t::underlying*>(X + i * C);
2447:         auto* Y_ptr = reinterpret_cast<typename scalar_t::underlying*>(Y + i * C);
2448:         int64_t ch = 0;
2449:
2450:         for(; ch + lanes <= C; ch += lanes) {
2451:           do_bn_compute<scalar_t, ReluFused>(
2452:             X_ptr + ch,
2453:             Y_ptr + ch,
2454:             fake_scale,
2455:             in_zp_vec,
2456:             scale_neg_zp_premul,
2457:             out_zero_point,
2458:             out_zero_point_v,
2459:             alpha + ch,
2460:             beta + ch,
2461:             Vec::float_num_vecs(),
2462:             kVLen
2463:           );
2464:         }
2465:
2466:         // for channel between 8 and 32, still use 32 width for performance
2467:         // Benchmark shows it is faster than doing 8 channels each time
2468:         int64_t elem_size = C - ch;
2469:         if ((lanes == 32) && elem_size >= kVLen) {
2470:           int64_t vec_num = elem_size / kVLen;
2471:           std::vector<typename scalar_t::underlying> buf_in(lanes);
2472:           memcpy(buf_in.data(), X_ptr + ch, vec_num * kVLen); // 3 cycles
2473:           do_bn_compute<scalar_t, ReluFused>(
2474:             buf_in.data(),
2475:             Y_ptr + ch,
2476:             fake_scale,
2477:             in_zp_vec,
2478:             scale_neg_zp_premul,
2479:             out_zero_point,
2480:             out_zero_point_v,
2481:             alpha + ch,
2482:             beta + ch,
2483:             vec_num,
2484:             kVLen
2485:           );
2486:           ch += vec_num * kVLen;
2487:         }
2488:         // for channels less than 8
2489:         for (; ch < C; ++ch) {
2490:           long quantized_down = out_zero_point +
2491:               lrintf(alpha[ch] * (X_ptr[ch] - in_zero_point) +
2492:                           beta[ch]);
2493:           if constexpr (ReluFused) { // static if
2494:             quantized_down = std::max<long>(quantized_down, out_zero_point);
2495:           }
2496:           Y_ptr[ch] = std::min<long>(
2497:               std::max<long>(quantized_down, minimum), maximum);
2498:         }
2499:       }
2500:     });
2501:   });
2502: }
2503:
2504: template <typename T>
2505: void q_batch_norm_cpu_kernel_impl(
2506:     int64_t N,
2507:     int64_t C,
2508:     int64_t HxW,
2509:     int64_t in_zero_point,
2510:     int64_t out_zero_point,
2511:     const uint8_t* in_ptr,
2512:     const float* alpha_ptr,
2513:     const float* beta_ptr,
2514:     T* out_ptr) {
```
- EN: The main symbol in this range is `constexpr`, `q_batch_norm_cpu_kernel_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `constexpr`, `q_batch_norm_cpu_kernel_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 2516-2597
```cpp
2516:   int q_min = 0;
2517:   int q_max = 255;
2518:   const int64_t outer_size = N * HxW;
2519:
2520: #if defined(CPU_CAPABILITY_AVX512)
2521:   constexpr int kVLen = 16;
2522:   static constexpr int num_vecs = sizeof(float) / sizeof(uint8_t);
2523:   auto in_zp_vec = _mm512_set1_ps((float)in_zero_point);
2524:   auto fake_scale = _mm512_set1_ps(1.0f);
2525:   auto scale_neg_zp_premul = _mm512_xor_ps(_mm512_set1_ps(-0.f), in_zp_vec);
2526:   auto out_zero_point_v = _mm512_set1_epi32((int)out_zero_point);
2527:   constexpr auto lanes = static_cast<int64_t>(num_vecs * kVLen);
2528:   __m512i v_q_max = _mm512_set1_epi32(q_max);
2529:   __m512i v_q_min = _mm512_set1_epi32(q_min);
2530:
2531:   auto load_convert_u8_to_f32_512bit = [&](const uint8_t* src, __m512* dst) {
2532:     // Step 1: Load 512 bits
2533:     __m512i raw = _mm512_loadu_si512(src);
2534:
2535:     // Step 2: Extract two 256-bit chunks
2536:     __m256i v0 = _mm512_extracti64x4_epi64(raw, 0); // bytes 0–31
2537:     __m256i v1 = _mm512_extracti64x4_epi64(raw, 1); // bytes 32–63
2538:
2539:     // Step 3: Process each 256-bit chunk
2540:     // --- Expand uint8_t -> uint16_t ---
2541:     __m256i u16lo0 = _mm256_cvtepu8_epi16(_mm256_extracti128_si256(v0, 0));
2542:     __m256i u16hi0 = _mm256_cvtepu8_epi16(_mm256_extracti128_si256(v0, 1));
2543:     __m256i u16lo1 = _mm256_cvtepu8_epi16(_mm256_extracti128_si256(v1, 0));
2544:     __m256i u16hi1 = _mm256_cvtepu8_epi16(_mm256_extracti128_si256(v1, 1));
2545:     // --- Expand to uint32_t and convert to float ---
2546:     dst[0] = _mm512_cvtepi32_ps(_mm512_cvtepu16_epi32(u16lo0));
2547:     dst[1] = _mm512_cvtepi32_ps(_mm512_cvtepu16_epi32(u16hi0));
2548:     dst[2] = _mm512_cvtepi32_ps(_mm512_cvtepu16_epi32(u16lo1));
2549:     dst[3] = _mm512_cvtepi32_ps(_mm512_cvtepu16_epi32(u16hi1));
2550:   };
2551:
2552:   auto load_convert_u8_to_f32_128bit = [&](const uint8_t* src) {
2553:     // --- Load and expand uint8_t -> uint16_t ---
2554:     __m256i v_u16 = _mm256_cvtepu8_epi16(_mm_loadu_si128((__m128i*)src));
2555:     // --- Expand to uint32_t and convert to float ---
2556:     return _mm512_cvtepi32_ps(_mm512_cvtepu16_epi32(v_u16));
2557:   };
2558:
2559:   auto store_output = [&](__m512 out, T* out_addr) {
2560:     if constexpr (std::is_same<T, float>::value) {
2561:       _mm512_storeu_ps(out_addr, out);
2562:     } else if constexpr (std::is_same<T, at::BFloat16>::value) {
2563:       __m256i out_bf16 = cvtfp32_bf16(out);
2564:       _mm256_storeu_si256((__m256i*)out_addr, out_bf16);
2565:     } else if constexpr (std::is_same<T, at::Half>::value) {
2566:       __m256i out_f16 = cvtfp32_fp16(out);
2567:       _mm256_storeu_si256((__m256i*)out_addr, out_f16);
2568:     } else { //  T == uint8, requantization needed
2569:       __m512i out_i32 = _mm512_cvtps_epi32(out);
2570:       out_i32 = _mm512_add_epi32(out_i32, out_zero_point_v);
2571:       out_i32 = _mm512_min_epi32(out_i32, v_q_max);
2572:       out_i32 = _mm512_max_epi32(out_i32, v_q_min);
2573:       __m128i out_i8 = _mm512_cvtepi32_epi8(out_i32);
2574:       _mm_storeu_si128((__m128i*)out_addr, out_i8);
2575:     }
2576:   };
2577: #endif
2578:
2579:   at::parallel_for(0, outer_size, 0, [&](int64_t begin, int64_t end) {
2580:     for (const auto i : c10::irange(begin, end)) {
2581:       auto* X_ptr = in_ptr + i * C;
2582:       auto* Y_ptr = out_ptr + i * C;
2583:       int64_t ch = 0;
2584:
2585: #if defined(CPU_CAPABILITY_AVX512)
2586:       __m512 vals_dq[num_vecs];
2587:       for(; ch + lanes <= C; ch += lanes) {
2588:         // load 64 values of input then dequantize them
2589:         load_convert_u8_to_f32_512bit(X_ptr + ch, vals_dq);
2590:         for (const auto idx : c10::irange(num_vecs)) {
2591:           vals_dq[idx] = _mm512_fmadd_ps(fake_scale, vals_dq[idx], scale_neg_zp_premul);
2592:           auto alpha_v = _mm512_loadu_ps(alpha_ptr + ch + idx * kVLen);
2593:           auto beta_v = _mm512_loadu_ps(beta_ptr + ch + idx * kVLen);
2594:           vals_dq[idx] = _mm512_fmadd_ps(alpha_v, vals_dq[idx], beta_v);
2595:           store_output(vals_dq[idx], Y_ptr + ch + idx * kVLen);
2596:         }
2597:       }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 2599-2672
```cpp
2599:       // for channel between 16 and 64
2600:       int64_t elem_size = C - ch;
2601:       if (elem_size >= kVLen) {
2602:         int64_t vec_num = elem_size / kVLen;
2603:         for (const auto idx : c10::irange(vec_num)) {
2604:           __m512 val_dq = load_convert_u8_to_f32_128bit(X_ptr + ch + idx * kVLen);
2605:           val_dq = _mm512_fmadd_ps(fake_scale, val_dq, scale_neg_zp_premul);
2606:           auto alpha_v = _mm512_loadu_ps(alpha_ptr + ch + idx * kVLen);
2607:           auto beta_v = _mm512_loadu_ps(beta_ptr + ch + idx * kVLen);
2608:           val_dq = _mm512_fmadd_ps(alpha_v, val_dq, beta_v);
2609:           store_output(val_dq, Y_ptr + ch + idx * kVLen);
2610:         }
2611:         ch += vec_num * kVLen;
2612:       }
2613: #endif
2614:       // for channels less than 16
2615:       for (; ch < C; ++ch) {
2616:         float y_val_f = alpha_ptr[ch] * (X_ptr[ch] - in_zero_point) +
2617:                         beta_ptr[ch];
2618:         if constexpr (std::is_same<T, float>::value) {
2619:           Y_ptr[ch] = y_val_f;
2620:         } else if constexpr (std::is_same<T, at::BFloat16>::value) {
2621:           Y_ptr[ch] = (at::BFloat16)y_val_f;
2622:         } else if constexpr (std::is_same<T, at::Half>::value) {
2623:           Y_ptr[ch] = (at::Half)y_val_f;
2624:         } else { //  T == uint8, requantization needed
2625:           long quantized_down = out_zero_point + lrintf(y_val_f);
2626:           Y_ptr[ch] = std::min<long>(
2627:               std::max<long>(quantized_down, q_min), q_max);
2628:         }
2629:       }
2630:     }
2631:   });
2632: }
2633:
2634: void q_batch_norm_cpu_kernel(
2635:     int64_t N,
2636:     int64_t C,
2637:     int64_t HxW,
2638:     int64_t in_zero_point,
2639:     int64_t out_zero_point,
2640:     const Tensor& input,
2641:     const Tensor& a,
2642:     const Tensor& b,
2643:     Tensor& output) {
2644:   auto in_ptr = input.const_data_ptr<uint8_t>();
2645:   float* alpha_ptr = a.data_ptr<float>();
2646:   float* beta_ptr = b.data_ptr<float>();
2647:   AT_DISPATCH_FLOATING_TYPES_AND3(
2648:       at::ScalarType::BFloat16, at::ScalarType::Half, at::ScalarType::Byte, output.scalar_type(), "int8_batch_norm2d_cpu", [&] {
2649:         auto out_ptr = output.data_ptr<scalar_t>();
2650:         q_batch_norm_cpu_kernel_impl<scalar_t>(
2651:             N, C, HxW, in_zero_point, out_zero_point, in_ptr, alpha_ptr, beta_ptr, out_ptr);
2652:       });
2653: }
2654:
2655: void _fake_quantize_tensor_helper(
2656:   Tensor& output,
2657:   Tensor& mask,
2658:   const Tensor& input,
2659:   int fake_quant_on,
2660:   float sc,
2661:   int64_t z_point,
2662:   int64_t quant_min,
2663:   int64_t quant_max) {
2664:
2665:   float inv_scale = 1.0f / sc;
2666:
2667:   auto iter_combined = TensorIteratorConfig()
2668:     .check_all_same_dtype(false)
2669:     .add_output(output)
2670:     .add_output(mask)
2671:     .add_input(input)
2672:     .build();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `constexpr`, `q_batch_norm_cpu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `constexpr`, `q_batch_norm_cpu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 2674-2742
```cpp
2674:   if (at::isReducedFloatingType(input.scalar_type())) {
2675:     AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "fake_quantize_tensor_cachemask_kernel_type_handling", [&]() {
2676:       iter_combined.for_each([&](char** data, const int64_t* strides, int64_t n) {
2677:         for (const auto i : c10::irange(n)) {
2678:           scalar_t* output_val = (scalar_t*)(data[0] + i * strides[0]);
2679:           bool* mask_val = (bool*)(data[1] + i * strides[1]);
2680:           scalar_t* input_val = (scalar_t*)(data[2] + i * strides[2]);
2681:
2682:           if (fake_quant_on) {
2683:             auto qval_f = z_point + std::nearbyint(*input_val * inv_scale);
2684:             const auto qval = static_cast<int64_t>(std::fmin(std::fmax(qval_f, quant_min), quant_max));
2685:             *output_val = (qval - z_point) * sc;
2686:             *mask_val = ((quant_min <= qval_f) && (qval_f <= quant_max));
2687:           } else {
2688:             *output_val = *input_val;
2689:             *mask_val = 1;
2690:           }
2691:         }
2692:       });
2693:     });
2694:   } else {
2695:     AT_DISPATCH_FLOATING_TYPES_AND_HALF(input.scalar_type(), "fake_quantize_tensor_cachemask_kernel_type_handling", [&] {
2696:       iter_combined.for_each([&](char** data, const int64_t* strides, int64_t n) {
2697:         for (const auto i : c10::irange(n)) {
2698:           scalar_t* output_val = (scalar_t*)(data[0] + i * strides[0]);
2699:           bool* mask_val = (bool*)(data[1] + i * strides[1]);
2700:           scalar_t* input_val = (scalar_t*)(data[2] + i * strides[2]);
2701:
2702:           if (fake_quant_on) {
2703:             auto qval_f = z_point + std::nearbyint(*input_val * inv_scale);
2704:             const auto qval = static_cast<int64_t>(std::fmin(std::fmax(qval_f, quant_min), quant_max));
2705:             *output_val = (qval - z_point) * sc;
2706:             *mask_val = ((quant_min <= qval_f) && (qval_f <= quant_max));
2707:           } else {
2708:             *output_val = *input_val;
2709:             *mask_val = 1;
2710:           }
2711:         }
2712:       });
2713:     });
2714:   }
2715: }
2716:
2717: void fake_quantize_tensor_cachemask_kernel(
2718:     Tensor& output,
2719:     Tensor& mask,
2720:     const Tensor& input,
2721:     float sc,
2722:     int64_t z_point,
2723:     int64_t quant_min,
2724:     int64_t quant_max) {
2725:   _fake_quantize_tensor_helper(output, mask, input, 1, sc, z_point, quant_min, quant_max);
2726: }
2727:
2728: void fake_quantize_tensor_cachemask_tensor_qparams_kernel(
2729:     Tensor& output,
2730:     Tensor& mask,
2731:     const Tensor& input,
2732:     const Tensor& sc,
2733:     const Tensor& z_point,
2734:     const Tensor& fake_quant_enabled,
2735:     int64_t quant_min,
2736:     int64_t quant_max) {
2737:   _fake_quantize_tensor_helper(output, mask, input, fake_quant_enabled.item().toInt(), sc.item().toFloat(), z_point.item().toInt(), quant_min, quant_max);
2738: }
2739:
2740: void fake_quantize_learnable_tensor_grad_kernel_cpu(
2741:     TensorIterator& iter,
2742:     float scale,
```
- EN: The main symbol in this range is `for_each`, `fake_quantize_tensor_cachemask_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `for_each`, `fake_quantize_tensor_cachemask_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 2743-2812
```cpp
2743:     float inv_scale,
2744:     int64_t zero_point,
2745:     int64_t quant_min,
2746:     int64_t quant_max,
2747:     float grad_factor) {
2748:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2749:   float dscale_small = quant_min - zero_point;
2750:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2751:   float dscale_big = quant_max - zero_point;
2752:   iter.for_each([&](char** data, const int64_t* strides, int64_t n) {
2753:     /*  When a for_each call is made on a TensorIterator with multiple inputs and outputs,
2754:         the order they are accessed follows the order they are built within the iterator.
2755:         For example, if an iterator is built in the following order:
2756:         auto iter = TensorIteratorConfig().
2757:           .add_output(firstOutput)
2758:           .add_output(secondOutput)
2759:           .add_input(firstInput)
2760:           .add_input(secondInput)
2761:           .build()
2762:         data will contain 4 pointers to pointers to values in the following order:
2763:         firstOutput, secondOutput, firstInput, secondInput.
2764:         Proper pointer referencing and dereferencing, along with the usage of strides
2765:         (to move onto different elements), can allow accessing of the input and assignment
2766:         to the right output.
2767:     */
2768:     for (const auto i : c10::irange(n)) {
2769:       float* dXOutput = (float*)(data[0] + i * strides[0]);
2770:       float* dScaleOutput = (float*)(data[1] + i * strides[1]);
2771:       float* dZeroPointOutput = (float*)(data[2] + i * strides[2]);
2772:       float* XInput = (float*)(data[3] + i * strides[3]);
2773:       float* dYInput = (float*)(data[4] + i * strides[4]);
2774:       // Calculate gradients for X.
2775:       int64_t xqi = std::nearbyint(zero_point + (*XInput) * inv_scale);
2776:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2777:       *dXOutput = (*dYInput) * (xqi >= quant_min && xqi <= quant_max);
2778:       // Calculate gradients for scale and zero point.
2779:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2780:       float xfqi = static_cast<float>((std::max(std::min(xqi, quant_max), quant_min) - zero_point) * scale);
2781:       // Calculate gradients according to the gradient of the clamp function.
2782:       if (xqi < quant_min || xqi > quant_max) {
2783:         *dZeroPointOutput = (*dYInput) * (-1) * scale * grad_factor;
2784:         *dScaleOutput = ((xqi < quant_min) ? ((*dYInput) * dscale_small) : ((*dYInput) * dscale_big)) * grad_factor;
2785:       } else {
2786:         *dZeroPointOutput = 0;
2787:         *dScaleOutput = (*dYInput) * (xfqi - (*XInput)) * inv_scale * grad_factor;
2788:       }
2789:     }
2790:   });
2791: }
2792:
2793: template <typename SelfType>
2794: void _fake_quant_per_channel_cachemask_cpu_helper(
2795:     TensorIterator& iter,
2796:     TensorIterator& iter_mask,
2797:     const int64_t quant_min,
2798:     const int64_t quant_max) {
2799:
2800:   const auto& zero_point_dtype = iter.input_dtype(2);
2801:
2802:   if(at::isFloatingType(zero_point_dtype)){
2803:     // When zero_point is float, quantize mirroring affine quantizer equation
2804:     // Xq = Round(Xf * inv_scale + zero_point)
2805:     // where zero_point is in float.
2806:     AT_DISPATCH_FLOATING_TYPES_AND_HALF(zero_point_dtype, "fake_quantize_channel_cachemask_cpu_zero_point_handling", [&] {
2807:       // write mask
2808:       cpu_kernel(iter_mask, [=](SelfType self, float scale, scalar_t zero_point) -> bool {
2809:         float inv_scale = 1.0f / scale;
2810:         const auto qval = std::lrintf(zero_point + (self * inv_scale));
2811:         return ((quant_min <= qval) && (qval <= quant_max));
2812:       });
```
- EN: The main symbol in this range is `for_each`, `TensorIteratorConfig`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `for_each`, `TensorIteratorConfig`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 2814-2891
```cpp
2814:       // write fake_quant
2815:       cpu_kernel(iter, [=](SelfType self, float scale, scalar_t zero_point) -> SelfType {
2816:         float inv_scale = 1.0f / scale;
2817:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2818:         return (std::fmin(
2819:                     std::fmax(
2820:                         std::lrintf(zero_point + self * inv_scale),
2821:                         quant_min),
2822:                     quant_max) -
2823:                 zero_point) *
2824:             scale;
2825:       });
2826:     });
2827:
2828:   } else {
2829:       // write mask
2830:       cpu_kernel(iter_mask, [=](SelfType self, float scale, int32_t zero_point) -> bool {
2831:         float inv_scale = 1.0f / scale;
2832:         const auto qval = static_cast<int64_t>(zero_point + std::nearbyint(self * inv_scale));
2833:         return ((quant_min <= qval) && (qval <= quant_max));
2834:       });
2835:
2836:       // write fake_quant
2837:       cpu_kernel(iter, [=](SelfType self, float scale, int32_t zero_point) -> SelfType {
2838:         float inv_scale = 1.0f / scale;
2839:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2840:         return (std::fmin(
2841:                     std::fmax(
2842:                         static_cast<int64_t>(
2843:                             zero_point + std::nearbyint(self * inv_scale)),
2844:                         quant_min),
2845:                     quant_max) -
2846:                 zero_point) *
2847:             scale;
2848:       });
2849:   }
2850:
2851: }
2852:
2853:
2854: void fake_quant_per_channel_cachemask_cpu(
2855:     TensorIterator& iter,
2856:     TensorIterator& iter_mask,
2857:     int64_t quant_min,
2858:     int64_t quant_max) {
2859:   // TODO(future, optional): read once, write twice.  Not done at the moment
2860:   //   for simplicity, as we do not expect this to be a bottleneck.
2861:
2862:   if (at::isReducedFloatingType(iter.dtype())) {
2863:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "fake_quantize_channel_cachemask_cpu_type_handling", [&]() {
2864:       _fake_quant_per_channel_cachemask_cpu_helper<scalar_t>(iter, iter_mask, quant_min, quant_max);
2865:     });
2866:   } else {
2867:     AT_DISPATCH_FLOATING_TYPES_AND_HALF(iter.dtype(), "fake_quantize_channel_cachemask_cpu_type_handling", [&] {
2868:       _fake_quant_per_channel_cachemask_cpu_helper<scalar_t>(iter, iter_mask, quant_min, quant_max);
2869:     });
2870:   }
2871: }
2872:
2873:
2874: void fake_quantize_learnable_channel_grad_kernel_cpu(
2875:     TensorIterator& iter,
2876:     int64_t quant_min,
2877:     int64_t quant_max,
2878:     float grad_factor) {
2879:   iter.for_each([&](char** data, const int64_t* strides, int64_t n) {
2880:     /*  To see how the input and outputs are referenced and assigned,
2881:         please see the implementation of
2882:         fake_quantize_learnable_tensor_grad_kernel_cpu.
2883:     */
2884:     for (const auto i : c10::irange(n)) {
2885:       float* dx_output = (float*)(data[0] + i * strides[0]);
2886:       float* dscale_output = (float*)(data[1] + i * strides[1]);
2887:       float* dzero_point_output = (float*)(data[2] + i * strides[2]);
2888:       float* x_input = (float*)(data[3] + i * strides[3]);
2889:       float* dy_input = (float*)(data[4] + i * strides[4]);
2890:       float* scale_input = (float*)(data[5] + i * strides[5]);
2891:       float* zero_point_input = (float*)(data[6] + i * strides[6]);
```
- EN: The main symbol in this range is `fake_quant_per_channel_cachemask_cpu`, `fake_quantize_learnable_channel_grad_kernel_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `fake_quant_per_channel_cachemask_cpu`, `fake_quantize_learnable_channel_grad_kernel_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 2893-2968
```cpp
2893:       float inv_scale = 1.0f / (*scale_input);
2894:       float dscale_small = quant_min - (*zero_point_input);
2895:       float dscale_big = quant_max - (*zero_point_input);
2896:       // Calculate gradients for X.
2897:       int64_t xqi = std::nearbyint((*zero_point_input) + (*x_input) * inv_scale);
2898:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
2899:       *dx_output = (*dy_input) * (xqi >= quant_min && xqi <= quant_max);
2900:       // Calculate gradients for scale and zero point.
2901:       float xfqi = ((std::max(std::min(xqi, quant_max), quant_min) - (*zero_point_input)) * (*scale_input));
2902:       if (xqi < quant_min || xqi > quant_max) {
2903:         *dzero_point_output = (*dy_input) * (-1) * (*scale_input) * grad_factor;
2904:         *dscale_output = ((xqi < quant_min) ? ((*dy_input) * dscale_small) : ((*dy_input) * dscale_big)) * grad_factor;
2905:       } else {
2906:         *dzero_point_output = 0;
2907:         *dscale_output = (*dy_input) * (xfqi - (*x_input)) * inv_scale * grad_factor;
2908:       }
2909:     }
2910:   });
2911: }
2912:
2913: // Assumes X is composed of M groups of N elements. Normalizes each of the
2914: // groups and optionally applies affine scaling. Useful for LayerNorm,
2915: // GroupNorm, InstanceNorm.
2916: void quantized_normalize_kernel(
2917:     const Tensor& X, // input tensor
2918:     const Tensor& gamma, // weight (optional)
2919:     const Tensor& beta, // bias (optional)
2920:     bool affine_per_channel, // scaling applied elementwise if false, per channel if true
2921:     int num_channels, // only used if affine_per_channel is set
2922:     int num_groups, // only used if affine_per_channel is set
2923:     int64_t M, // number of groups
2924:     int64_t N, // number of elements in each group
2925:     double eps,
2926:     Tensor* Y) {
2927:   AT_DISPATCH_QINT_TYPES(X.scalar_type(), "quantized_layer_norm_kernel_impl_cpu", [&]() {
2928:     using qVec = vec::Vectorized<scalar_t>;
2929:     using fVec = vec::Vectorized<float>;
2930:
2931:     TORCH_INTERNAL_ASSERT(X.numel() == M * N, "Unexpected num elements in X");
2932:     TORCH_INTERNAL_ASSERT(
2933:         !gamma.defined() ||
2934:         (!affine_per_channel && gamma.numel() == N) ||
2935:         (affine_per_channel && gamma.numel() == num_channels),
2936:         "Unexpected size of gamma");
2937:     TORCH_INTERNAL_ASSERT(
2938:         !beta.defined() ||
2939:         (!affine_per_channel && beta.numel() == N) ||
2940:         (affine_per_channel && beta.numel() == num_channels),
2941:         "Unexpected size of beta");
2942:
2943:     scalar_t* X_data = X.data_ptr<scalar_t>();
2944:     const float* gamma_data = gamma.defined() ? gamma.const_data_ptr<float>() : nullptr;
2945:     const float* beta_data = beta.defined() ? beta.const_data_ptr<float>() : nullptr;
2946:     scalar_t* Y_data = Y->data_ptr<scalar_t>();
2947:     const bool gamma_null = gamma_data == nullptr;
2948:     const bool beta_null = beta_data == nullptr;
2949:     int64_t x_zp = X.q_zero_point();
2950:     float x_scale = X.q_scale();
2951:     fVec x_zp_vec(x_zp);
2952:     fVec one_vec(1.0f);
2953:     fVec zero_vec(0.0f);
2954:     float x_fake_scale = 1.0f;
2955:     fVec x_fake_scale_vec(x_fake_scale);
2956:     fVec x_fake_scale_zp_neg_premul_vec = x_fake_scale_vec * x_zp_vec.neg();
2957:     int64_t y_zp = Y->q_zero_point();
2958:     float y_scale = Y->q_scale();
2959:     float y_inv_scale = 1.0f / y_scale;
2960:
2961:     constexpr int kFloatVLen = fVec::size();
2962:     int64_t kIntVLen = kFloatVLen * qVec::float_num_vecs();
2963:     int64_t kNumIntVecInLayer = N / kIntVLen;
2964:     int64_t kNonVecRemInLayer = N % kIntVLen;
2965:     int channels_per_group = num_channels / num_groups;
2966:     int64_t NPerChannel = N / channels_per_group;
2967:     int64_t kNumIntVecInChannel = NPerChannel / kIntVLen;
2968:     int64_t kNonVecRemInChannel = NPerChannel % kIntVLen;
```
- EN: The main symbol in this range is `quantized_normalize_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `quantized_normalize_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 2970-3043
```cpp
2970:     at::parallel_for(0, M, 1, [&](int64_t start, int64_t end) {
2971:       for (const auto i : c10::irange(start, end)) {
2972:
2973:         scalar_t* X_ptr = X_data + i * N;
2974:         scalar_t* Y_ptr = Y_data + i * N;
2975:
2976:         // First pass: calculate mean and variance.
2977:
2978:         scalar_t::underlying* X_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(X_ptr);
2979:         auto l_sum_shifted = hsum(X_ptr_underlying, N);
2980:         auto l_sum_sq_shifted = hsum_sq(X_ptr_underlying, N);
2981:         float l_mean_shifted_div_scale_x = static_cast<float>(l_sum_shifted) / N;
2982:         // mean(dqX) / scale_x
2983:         float layer_mean_div_scale_x = l_mean_shifted_div_scale_x - x_zp;
2984:         // var(dqX) / scale_x^2
2985:         float layer_var_div_scale_x_sq =
2986:           std::max(static_cast<float>(l_sum_sq_shifted) / N -
2987:               l_mean_shifted_div_scale_x * l_mean_shifted_div_scale_x, 0.0f);
2988:         // scale_x / sqrt(var(dqX) + eps)
2989:         float scale_x_div_layer_std = x_scale /
2990:           std::sqrt(layer_var_div_scale_x_sq * x_scale * x_scale + eps);
2991:         fVec layer_mean_div_scale_xVec(layer_mean_div_scale_x);
2992:         fVec scale_x_div_layer_stdVec(scale_x_div_layer_std);
2993:
2994:         // Second pass: normalize
2995:
2996:         // TODO replace with TensorIterator implementation once #33166 is fixed.
2997:         if (affine_per_channel) {
2998:
2999:           // if scaling per channel, scaling parameters can be pre-multiplied
3000:           // with normalization parameters
3001:           for (const auto chIdx : c10::irange(channels_per_group)) {
3002:             int scalingIdx = (i * channels_per_group + chIdx) % (num_channels);
3003:             float gamma = gamma_null ? 1.0f : gamma_data[scalingIdx];
3004:             // scale_x / layer_std * gamma
3005:             float gamma_p = scale_x_div_layer_std * gamma;
3006:             float beta = beta_null ? 0.0f : beta_data[scalingIdx];
3007:             fVec gamma_p_vec(gamma_p);
3008:             fVec beta_vec(beta);
3009:
3010:             int64_t chStartIdx = chIdx * NPerChannel;
3011:             int64_t chEndIdx = chStartIdx + NPerChannel;
3012:
3013:             for (const auto vecIdx : c10::irange(kNumIntVecInChannel)) {
3014:               int64_t vecStartIdx = chStartIdx + vecIdx * kIntVLen;
3015:               auto qXVec = qVec::loadu(X_ptr + vecStartIdx);
3016:               auto dqXVec = qXVec.dequantize(x_fake_scale_vec, x_zp_vec,
3017:                   x_fake_scale_zp_neg_premul_vec);
3018:               for (auto &dq : dqXVec) {
3019:                 dq =
3020:                   (dq - layer_mean_div_scale_xVec) *
3021:                     gamma_p_vec + beta_vec;
3022:               }
3023:               qVec::quantize(dqXVec, y_scale, y_zp, y_inv_scale)
3024:                 .store(Y_ptr + vecStartIdx);
3025:             }
3026:
3027:             // Remainder
3028:             if (kNonVecRemInChannel > 0) {
3029:               int64_t remIdx = chEndIdx - kNonVecRemInChannel;
3030:               auto qXVec = qVec::loadu(X_ptr + remIdx, kNonVecRemInChannel);
3031:               auto dqXVec = qXVec.dequantize(x_fake_scale_vec, x_zp_vec,
3032:                     x_fake_scale_zp_neg_premul_vec);
3033:               int validDqvecLen = (kNonVecRemInChannel - 1) / fVec::size() + 1;
3034:               for (int i = 0; i < validDqvecLen; ++i) {
3035:                 auto &dq = dqXVec[i];
3036:                 dq =
3037:                   (dq - layer_mean_div_scale_xVec) *
3038:                     gamma_p_vec + beta_vec;
3039:               }
3040:               qVec::quantize(dqXVec, y_scale, y_zp, y_inv_scale)
3041:                 .store(Y_ptr + remIdx, kNonVecRemInChannel);
3042:             }
3043:           } // chIdx
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 3045-3113
```cpp
3045:         } else {
3046:
3047:           for (const auto vecIdx : c10::irange(kNumIntVecInLayer)) {
3048:             int64_t vecStartIdx = vecIdx * kIntVLen;
3049:             auto qXVec = qVec::loadu(X_ptr + vecStartIdx);
3050:             auto dqXVec = qXVec.dequantize(x_fake_scale_vec, x_zp_vec,
3051:                 x_fake_scale_zp_neg_premul_vec);
3052:             for (const auto dqXVecIdx : c10::irange(dqXVec.size())) {
3053:               int64_t vecVecStartIdx = vecStartIdx + dqXVecIdx * kFloatVLen;
3054:               auto gammaVec = gamma_null
3055:                 ? one_vec
3056:                 : fVec::loadu(gamma_data + vecVecStartIdx);
3057:               auto betaVec = beta_null
3058:                 ? zero_vec
3059:                 : fVec::loadu(beta_data + vecVecStartIdx);
3060:               dqXVec[dqXVecIdx] =
3061:                 (dqXVec[dqXVecIdx] - layer_mean_div_scale_xVec) *
3062:                   scale_x_div_layer_stdVec * gammaVec + betaVec;
3063:               qVec::quantize(dqXVec, y_scale, y_zp, y_inv_scale)
3064:                 .store(Y_ptr + vecStartIdx);
3065:             }
3066:           }
3067:           for (int64_t remIdx = N - kNonVecRemInLayer; remIdx < N; remIdx++) {
3068:             const float gamma_v = gamma_null ? 1.0f : gamma_data[remIdx];
3069:             const float beta_v = beta_null ? 0.0f : beta_data[remIdx];
3070:             auto qXVal = X_ptr[remIdx];
3071:             float dqXVal = at::native::dequantize_val(x_fake_scale, x_zp, qXVal);
3072:             float dqY =
3073:               ((dqXVal - layer_mean_div_scale_x) * scale_x_div_layer_std) * gamma_v + beta_v;
3074:             Y_ptr[remIdx] = at::native::quantize_val<scalar_t>(y_scale, y_zp, dqY);
3075:           }
3076:         }
3077:       }
3078:     }); // parallel_for
3079:
3080:   });
3081: }
3082:
3083: void qmean_inner_dim_kernel(
3084:     const Tensor& self,
3085:     OptionalIntArrayRef opt_dim,
3086:     bool keepdim,
3087:     std::optional<ScalarType> opt_dtype,
3088:     Tensor& result) {
3089:   // 'opt_dtype' should be none or equal to that of input
3090:   ScalarType dtype = self.scalar_type();
3091:   auto in_dims = self.sizes().vec();
3092:   auto out_dims = in_dims;
3093:   bool is_all_reduce = !opt_dim.has_value() || opt_dim.value().empty();
3094:   size_t num_dims_to_squeeze = is_all_reduce ? self.dim() : opt_dim.value().size();
3095:   int64_t M = 1; // Num of groups
3096:   int64_t N = 1; // Num of elements to take average of in each group
3097:   for (size_t i = 0; i < in_dims.size() - num_dims_to_squeeze; ++i) {
3098:     M *= in_dims[i];
3099:   }
3100:   for (size_t i = 0; i < num_dims_to_squeeze; ++i) {
3101:     auto idx = out_dims.size() - 1 - i;
3102:     N *= out_dims[idx];
3103:     out_dims[idx] = 1;
3104:   }
3105:   if (!keepdim) {
3106:     out_dims.erase(out_dims.end() - num_dims_to_squeeze, out_dims.end());
3107:   }
3108:   result = at::_empty_affine_quantized(
3109:       out_dims,
3110:       at::device(kCPU).dtype(dtype).memory_format(self.suggest_memory_format()),
3111:       self.q_scale(),
3112:       self.q_zero_point(),
3113:       std::nullopt);
```
- EN: The main symbol in this range is `qmean_inner_dim_kernel`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `qmean_inner_dim_kernel`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 3115-3197
```cpp
3115:   AT_DISPATCH_QINT_TYPES(self.scalar_type(), "quantized_mean_kernel_impl_cpu", [&]() {
3116:     scalar_t* X_data = self.data_ptr<scalar_t>();
3117:     scalar_t* Y_data = result.data_ptr<scalar_t>();
3118:
3119:     at::parallel_for(0, M, 1, [&](int64_t start, int64_t end) {
3120:       for (const auto i : c10::irange(start, end)) {
3121:         scalar_t* X_ptr = X_data + i * N;
3122:         scalar_t* Y_ptr = Y_data + i;
3123:         scalar_t::underlying* X_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(X_ptr);
3124:         scalar_t::underlying* Y_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(Y_ptr);
3125:         auto x_sum = hsum(X_ptr_underlying, N);
3126:         float y_float = static_cast<float>(x_sum) / N;
3127:         *Y_ptr_underlying = std::nearbyint(y_float);
3128:       }
3129:     });
3130:   });
3131: }
3132:
3133: void qstd_inner_dim_kernel(
3134:     const Tensor& self,
3135:     OptionalIntArrayRef dim,
3136:     const std::optional<Scalar>& correction_opt,
3137:     bool keepdim,
3138:     Tensor& result) {
3139:   ScalarType dtype = self.scalar_type();
3140:   auto in_dims = self.sizes().vec();
3141:   auto out_dims = in_dims;
3142:   size_t num_dims_to_squeeze = dim.has_value() && !dim.value().empty() ?
3143:                                dim.value().size() :
3144:                                self.dim();
3145:   int64_t M = 1; // Num of groups
3146:   int64_t N = 1; // Num of elements to take std of in each group
3147:   for (size_t i = 0; i < in_dims.size() - num_dims_to_squeeze; ++i) {
3148:     M *= in_dims[i];
3149:   }
3150:   for (size_t i = 0; i < num_dims_to_squeeze; ++i) {
3151:     auto idx = out_dims.size() - 1 - i;
3152:     N *= out_dims[idx];
3153:     out_dims[idx] = 1;
3154:   }
3155:   if (!keepdim) {
3156:     out_dims.erase(out_dims.end() - num_dims_to_squeeze, out_dims.end());
3157:   }
3158:   const auto correction = correction_opt.value_or(1).toDouble();
3159:   double den = std::max(N - correction, 0.0); // Denominator when computing mean and deviation
3160:   auto x_scale = self.q_scale();
3161:   auto x_zp = self.q_zero_point();
3162:   result = at::_empty_affine_quantized(
3163:       out_dims,
3164:       at::device(kCPU).dtype(dtype).memory_format(self.suggest_memory_format()),
3165:       x_scale,
3166:       x_zp,
3167:       std::nullopt);
3168:
3169:   AT_DISPATCH_QINT_TYPES(self.scalar_type(), "quantized_std_kernel_impl_cpu", [&]() {
3170:     scalar_t* X_data = self.data_ptr<scalar_t>();
3171:     scalar_t* Y_data = result.data_ptr<scalar_t>();
3172:
3173:     at::parallel_for(0, M, 1, [&](int64_t start, int64_t end) {
3174:       for (const auto i : c10::irange(start, end)) {
3175:         scalar_t* X_ptr = X_data + i * N;
3176:         scalar_t* Y_ptr = Y_data + i;
3177:         scalar_t::underlying* X_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(X_ptr);
3178:         scalar_t::underlying* Y_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(Y_ptr);
3179:         auto x_sum_shifted = hsum(X_ptr_underlying, N);
3180:         auto x_sum_sq_shifted = hsum_sq(X_ptr_underlying, N);
3181:         // Use double for intermediate variables to avoid accuracy issue
3182:         // Mean with zero point
3183:         double x_mean_shifted_div_scale_x = static_cast<double>(x_sum_shifted) / N;
3184:         double x_mean_unbiased_shifted_div_scale_x = static_cast<double>(x_sum_shifted) / den;
3185:         // variance / x_scale^2
3186:         double x_var_div_scale_x_sq =
3187:             std::max(static_cast<double>(x_sum_sq_shifted) / den -
3188:                 2 * x_mean_shifted_div_scale_x * x_mean_unbiased_shifted_div_scale_x +
3189:                 x_mean_shifted_div_scale_x * x_mean_shifted_div_scale_x * N / den, (double)0.0);
3190:         double y_float = std::sqrt(x_var_div_scale_x_sq) * x_scale;
3191:         *Y_ptr_underlying = at::native::quantize_val<scalar_t>(
3192:                             x_scale, x_zp, y_float)
3193:                             .val_;
3194:       }
3195:     });
3196:   });
3197: }
```
- EN: The main symbol in this range is `qstd_inner_dim_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 这一段的主要符号是 `qstd_inner_dim_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 3199-3280
```cpp
3199: // For group norm of channels_last input
3200: void quantized_groupnorm_nhwc_kernel(
3201:     const Tensor& X, // input tensor
3202:     const Tensor& gamma, // weight (optional)
3203:     const Tensor& beta, // bias (optional)
3204:     bool affine_per_channel, // must be true for group/instance norm
3205:     int num_channels, // only used if affine_per_channel is set
3206:     int num_groups, // only used if affine_per_channel is set
3207:     int64_t M, // number of groups = Bs * G
3208:     int64_t N, // number of elements in each group = C * H * W / G
3209:     double eps,
3210:     Tensor* Y) {
3211:   AT_DISPATCH_QINT_TYPES(X.scalar_type(), "quantized_norm_nhwc_kernel_impl_cpu", [&]() {
3212:     using qVec = vec::Vectorized<scalar_t>;
3213:     using fVec = vec::Vectorized<float>;
3214:
3215:     int64_t G = num_groups;
3216:     int64_t Bs = M / G;
3217:     int64_t C = num_channels;
3218:
3219:     TORCH_INTERNAL_ASSERT(X.numel() == M * N, "Unexpected num elements in X");
3220:     TORCH_INTERNAL_ASSERT(
3221:         !gamma.defined() ||
3222:         (!affine_per_channel && gamma.numel() == N) ||
3223:         (affine_per_channel && gamma.numel() == C),
3224:         "Unexpected size of gamma");
3225:     TORCH_INTERNAL_ASSERT(
3226:         !beta.defined() ||
3227:         (!affine_per_channel && beta.numel() == N) ||
3228:         (affine_per_channel && beta.numel() == C),
3229:         "Unexpected size of beta");
3230:
3231:     scalar_t* X_data = X.data_ptr<scalar_t>();
3232:     const float* gamma_data = gamma.defined() ? gamma.const_data_ptr<float>() : nullptr;
3233:     const float* beta_data = beta.defined() ? beta.const_data_ptr<float>() : nullptr;
3234:     scalar_t* Y_data = Y->data_ptr<scalar_t>();
3235:     const bool gamma_null = gamma_data == nullptr;
3236:     const bool beta_null = beta_data == nullptr;
3237:     int64_t x_zp = X.q_zero_point();
3238:     float x_scale = X.q_scale();
3239:     fVec x_zp_vec(x_zp);
3240:     fVec one_vec(1.0f);
3241:     fVec zero_vec(0.0f);
3242:     float x_fake_scale = 1.0f;
3243:     fVec x_fake_scale_vec(x_fake_scale);
3244:     fVec x_fake_scale_zp_neg_premul_vec = x_fake_scale_vec * x_zp_vec.neg();
3245:     int64_t y_zp = Y->q_zero_point();
3246:     float y_scale = Y->q_scale();
3247:     float y_inv_scale = 1.0f / y_scale;
3248:
3249:     constexpr int kFloatVLen = fVec::size();
3250:     int64_t kIntVLen = kFloatVLen * qVec::float_num_vecs();
3251:     int64_t channels_per_group = C / G;
3252:     int64_t HxW = N / channels_per_group;
3253:     int64_t kNumIntVecInHxW = channels_per_group / kIntVLen;
3254:     int64_t kNonVecRemInHxW = channels_per_group % kIntVLen;
3255:     int64_t kNumIntVecOnChannel = C / kIntVLen;
3256:     int64_t kNonVecRemOnChannel = C % kIntVLen;
3257:
3258:     // Buffer for x and x^2
3259:     Tensor buffer = at::empty({M, 2 * channels_per_group}, X.options().dtype(at::kFloat));
3260:     float* buffer_data = buffer.mutable_data_ptr<float>();
3261:
3262:     // We can parallel in the following 2 impls:
3263:     //
3264:     // impl-1: parallel on N * G. Only need one omp session but memory access
3265:     //   per thread is non-contiguous.
3266:     //
3267:     // impl-2: parallel on N * HxW. Memory access per thread is contiguous,
3268:     //   but requires help of extra temp buffer of size {T, N, 2C}.
3269:     //
3270:     // Generally impl-2 has better performance when HxW is large enough
3271:     // The threshold is found by tests.
3272:     constexpr int64_t feature_map_threshold = 512;
3273:     if (HxW < feature_map_threshold) {
3274:       // Impl-1: Parallel for each group
3275:       //
3276:       // Parallel for each group, M = Bs * G
3277:       at::parallel_for(0, M, 1, [&](int64_t begin, int64_t end) {
3278:         int64_t n{0} /* batch index */, g{0} /* group index in each batch */;
3279:         data_index_init(begin, n, N, g, G);
3280:         for (const auto grpIdx : c10::irange(begin, end)) { // For each group
```
- EN: The main symbol in this range is `quantized_groupnorm_nhwc_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 这一段的主要符号是 `quantized_groupnorm_nhwc_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 3282-3356
```cpp
3282:           // Step 1: calculate mean and variance.
3283:           int64_t l_sum_shifted = 0;
3284:           int64_t l_sum_sq_shifted = 0;
3285:           for (const auto hw : c10::irange(HxW)) {
3286:             scalar_t* X_ptr = X_data + n * N * G + g * channels_per_group + hw * C;
3287:             scalar_t::underlying* X_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(X_ptr);
3288:             l_sum_shifted += hsum(X_ptr_underlying, channels_per_group);
3289:             l_sum_sq_shifted += hsum_sq(X_ptr_underlying, channels_per_group);
3290:           }
3291:
3292:           // mean(dqX) / scale_x + x_zp
3293:           float l_mean_shifted_div_scale_x = static_cast<float>(l_sum_shifted) / N;
3294:           // mean(dqX) / scale_x
3295:           float layer_mean_div_scale_x = l_mean_shifted_div_scale_x - x_zp;
3296:           // var(dqX) / scale_x^2
3297:           float layer_var_div_scale_x_sq =
3298:             std::max(static_cast<float>(l_sum_sq_shifted) / N -
3299:                 l_mean_shifted_div_scale_x * l_mean_shifted_div_scale_x, 0.0f);
3300:           // scale_x / sqrt(var(dqX) + eps)
3301:           float scale_x_div_layer_std = x_scale /
3302:             std::sqrt(layer_var_div_scale_x_sq * x_scale * x_scale + eps);
3303:
3304:           // Step 2: calculate scale and bias
3305:           float* scale_ptr = buffer_data + grpIdx * 2 * channels_per_group;
3306:           float* bias_ptr = scale_ptr + channels_per_group;
3307:           for (const auto d : c10::irange(channels_per_group)) {
3308:             const int64_t chIdx = g * channels_per_group + d;
3309:             scale_ptr[d] = scale_x_div_layer_std * (gamma_null ? 1.0f : gamma_data[chIdx]);
3310:             bias_ptr[d] = -scale_ptr[d] * layer_mean_div_scale_x + (beta_null ? 0.0f : beta_data[chIdx]);
3311:           }
3312:
3313:           // Step 3: applying scale and bias
3314:           for (const auto hwIdx : c10::irange(HxW)) {
3315:             const scalar_t* X_ptr = X_data + n * N * G + g * channels_per_group + hwIdx * C;
3316:             scalar_t* Y_ptr = Y_data + n * N * G + g * channels_per_group + hwIdx * C;
3317:             // vectorized
3318:             for (const auto vecIdx : c10::irange(kNumIntVecInHxW)) {
3319:               int64_t vecStartIdx = vecIdx * kIntVLen;
3320:               auto qXVec = qVec::loadu(X_ptr + vecStartIdx);
3321:               auto dqXVec = qXVec.dequantize(x_fake_scale_vec, x_zp_vec,
3322:                     x_fake_scale_zp_neg_premul_vec);
3323:               for (size_t fvecIdx = 0; fvecIdx < dqXVec.size(); ++fvecIdx) {
3324:                 auto scaleVec = fVec::loadu(scale_ptr + vecStartIdx + fvecIdx * kFloatVLen);
3325:                 auto biasVec = fVec::loadu(bias_ptr + vecStartIdx + fvecIdx * kFloatVLen);
3326:                 dqXVec[fvecIdx] = dqXVec[fvecIdx] * scaleVec + biasVec;
3327:               }
3328:               qVec::quantize(dqXVec, y_scale, y_zp, y_inv_scale)
3329:                   .store(Y_ptr + vecStartIdx);
3330:             }
3331:             // Remaining scalar
3332:             for (int64_t remIdx = kNumIntVecInHxW * kIntVLen;
3333:                  remIdx < kNonVecRemInHxW + kNumIntVecInHxW * kIntVLen;
3334:                  ++remIdx) {
3335:               auto qXVal = X_ptr[remIdx];
3336:               float dqXVal = at::native::dequantize_val(x_fake_scale, x_zp, qXVal);
3337:               float dqY = dqXVal * scale_ptr[remIdx] + bias_ptr[remIdx];
3338:               Y_ptr[remIdx] = at::native::quantize_val<scalar_t>(y_scale, y_zp, dqY);
3339:             }
3340:           } // loop over HxW
3341:
3342:           data_index_step(n, N, g, G);
3343:         } // for each group
3344:       }); // parallel_for
3345:     } else { // HxW > feature_map_threshold
3346:       // impl-2: parallel on Bs * HxW.
3347:       //
3348:       // Buffer for x and x^2
3349:       // To avoid thread conflict, we use a temp buffer of {T, Bs, 2*C}
3350:       int num_threads = at::get_num_threads();
3351:       Tensor buffer = at::empty({num_threads, Bs, 2 * C}, X.options().dtype(at::kFloat)).zero_();
3352:       float* buffer_data = buffer.mutable_data_ptr<float>();
3353:       Tensor mean = at::empty(M, X.options().dtype(at::kFloat));
3354:       float* mean_data = mean.mutable_data_ptr<float>();
3355:       Tensor rstd = at::empty(M, X.options().dtype(at::kFloat));
3356:       float* rstd_data = rstd.mutable_data_ptr<float>();
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 3358-3426
```cpp
3358:       // Step 1: Accumulate on C dimension
3359:       at::parallel_for(0, Bs * HxW, 1, [&](int64_t begin, int64_t end) {
3360:         int tid = at::get_thread_num();
3361:         float* buffer_ptr = buffer_data + tid * Bs * 2 * C;
3362:
3363:         int64_t n{0} /* batch index */, m{0} /* HxW index */;
3364:         data_index_init(begin, n, Bs, m, HxW);
3365:         for (const auto nhwIdx : c10::irange(begin, end)) {
3366:           float* mean_ptr = buffer_ptr + n * 2 * C;
3367:           float* rstd_ptr = mean_ptr + C;
3368:           scalar_t* X_ptr = X_data + nhwIdx * C;
3369:           scalar_t::underlying* X_ptr_underlying = reinterpret_cast<scalar_t::underlying*>(X_ptr);
3370:           for (int chIdx = 0; chIdx < C; ++chIdx) {
3371:             auto x = X_ptr_underlying[chIdx];
3372:             mean_ptr[chIdx] += x;
3373:             rstd_ptr[chIdx] += x * x;
3374:           }
3375:           data_index_step(n, Bs, m, HxW);
3376:         }
3377:       });
3378:
3379:       // Step 2: Calculate mean and rstd
3380:       for (const auto n : c10::irange(Bs)) {
3381:         for (const auto g : c10::irange(G)) {
3382:           float mean_val{0}, rstd_val{0};
3383:           for (const auto t : c10::irange(num_threads)) {
3384:             float* buffer_ptr = buffer_data + t * Bs * 2 * C + n * 2 * C;
3385:             for (const auto d : c10::irange(channels_per_group)) {
3386:               mean_val += buffer_ptr[g * channels_per_group + d];
3387:               rstd_val += buffer_ptr[g * channels_per_group + d + C];
3388:             } // for d
3389:           } // for t
3390:
3391:           // mean / scale_x + x_zp
3392:           float l_mean_shifted_div_scale_x = mean_val / N;
3393:           // mean / scale_x
3394:           float layer_mean_div_scale_x = l_mean_shifted_div_scale_x - x_zp;
3395:           // var / scale_x^2
3396:           float layer_var_div_scale_x_sq =
3397:               std::max(rstd_val / N -
3398:               l_mean_shifted_div_scale_x * l_mean_shifted_div_scale_x, 0.0f);
3399:           // scale_x / sqrt(var + eps)
3400:           float scale_x_div_layer_std = x_scale /
3401:               std::sqrt(layer_var_div_scale_x_sq * x_scale * x_scale + eps);
3402:           mean_data[n * G + g] = layer_mean_div_scale_x;
3403:           rstd_data[n * G + g] = scale_x_div_layer_std;
3404:
3405:         } // for g
3406:       } // for n
3407:
3408:       // Step 3: Calculate scale and bias
3409:       //
3410:       // We could fuse step 3 and 4 into a single session but this way is better:
3411:       //   a. D might be too small for vectorization;
3412:       //   b. Avoid duplicate calculation of scale/bias, each HxW plain share the same scale/bias
3413:       //
3414:       for (const auto n : c10::irange(Bs)) {
3415:         for (const auto g : c10::irange(G)) {
3416:           float* scale_ptr = buffer_data + n * 2 * C;
3417:           float* bias_ptr = scale_ptr + C;
3418:           float mean_val = mean_data[n * G + g];
3419:           float rstd_val = rstd_data[n * G + g];
3420:           for (const auto d : c10::irange(channels_per_group)) {
3421:             const int64_t chIdx = g * channels_per_group + d;
3422:             scale_ptr[chIdx] = rstd_val * (gamma_null ? 1.0f : gamma_data[chIdx]);
3423:             bias_ptr[chIdx] = -scale_ptr[chIdx] * mean_val + (beta_null ? 0.0f : beta_data[chIdx]);
3424:           } // for d
3425:         } // for g
3426:       } // for n
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 3428-3502
```cpp
3428:       // step-4: apply scale and bias
3429:       //
3430:       // Parallel on all the outer dimensions of Bs and HxW
3431:       // and vectorize on C.
3432:       //
3433:       at::parallel_for(0, Bs * HxW, 1, [&](int64_t begin, int64_t end) {
3434:         int64_t n{0}, m{0};
3435:         data_index_init(begin, n, Bs, m, HxW);
3436:         for (const auto nhwIdx : c10::irange(begin, end)) {
3437:           const scalar_t* X_ptr = X_data + nhwIdx * C;
3438:           scalar_t* Y_ptr = Y_data + nhwIdx * C;
3439:           float* scale_ptr = buffer_data + n * 2 * C;
3440:           float* bias_ptr = scale_ptr + C;
3441:           // Vectorized
3442:           for (const auto vecIdx : c10::irange(kNumIntVecOnChannel)) {
3443:             int64_t vecStartIdx = vecIdx * kIntVLen;
3444:             auto qXVec = qVec::loadu(X_ptr + vecStartIdx);
3445:             auto dqXVec = qXVec.dequantize(x_fake_scale_vec, x_zp_vec,
3446:                   x_fake_scale_zp_neg_premul_vec);
3447:             for (size_t fvecIdx = 0; fvecIdx < dqXVec.size(); ++fvecIdx) {
3448:               auto scaleVec = fVec::loadu(scale_ptr + vecStartIdx + fvecIdx * kFloatVLen);
3449:               auto biasVec = fVec::loadu(bias_ptr + vecStartIdx + fvecIdx * kFloatVLen);
3450:               dqXVec[fvecIdx] = dqXVec[fvecIdx] * scaleVec + biasVec;
3451:             }
3452:             qVec::quantize(dqXVec, y_scale, y_zp, y_inv_scale)
3453:                 .store(Y_ptr + vecStartIdx);
3454:           }
3455:           // Remaining scalar
3456:           for (int64_t remIdx = kNumIntVecOnChannel * kIntVLen;
3457:                remIdx < kNonVecRemOnChannel + kNumIntVecOnChannel * kIntVLen;
3458:                ++remIdx) {
3459:             auto qXVal = X_ptr[remIdx];
3460:             float dqXVal = at::native::dequantize_val(x_fake_scale, x_zp, qXVal);
3461:             float dqY = dqXVal * scale_ptr[remIdx] + bias_ptr[remIdx];
3462:             Y_ptr[remIdx] = at::native::quantize_val<scalar_t>(y_scale, y_zp, dqY);
3463:           }
3464:
3465:           data_index_step(n, Bs, m, HxW);
3466:         } // for idx on nhw
3467:       }); // parallel_for on nhw
3468:
3469:     } // if HxW > feature_map_threshold
3470:
3471:   }); // AT_DISPATCH_QINT_TYPES
3472: }
3473:
3474: #ifdef USE_FBGEMM
3475: void quantize_tensor_per_tensor_affine_cpu(
3476:     const Tensor& rtensor,
3477:     Tensor& qtensor,
3478:     double scale,
3479:     int64_t zero_point) {
3480:   AT_DISPATCH_QINT_TYPES(
3481:       qtensor.scalar_type(), "quantize_tensor_per_tensor_affine_cpu", [&]() {
3482:         check_tensor_memory_format(rtensor, qtensor);
3483:         const float* rd = rtensor.const_data_ptr<float>();
3484:         auto qd = reinterpret_cast<underlying_t*>(qtensor.data_ptr<scalar_t>());
3485:         fbgemm::TensorQuantizationParams qparams{};
3486:         qparams.scale = scale;
3487:         qparams.zero_point = zero_point;
3488:         qparams.precision = CHAR_BIT * sizeof(underlying_t);
3489:         int num_tasks = at::get_num_threads();
3490:         at::parallel_for(0, num_tasks, 1, [&](int64_t begin, int64_t end) {
3491:           for (const auto task_id : c10::irange(begin, end)) {
3492:             fbgemm::Quantize<underlying_t, false /*LEGACY*/>(
3493:                 rd, /*src=*/
3494:                 qd, /*dst=*/
3495:                 rtensor.numel(), /*len*/
3496:                 qparams, /*qparams=*/
3497:                 task_id, /*thread_id*/
3498:                 num_tasks /*num_threads*/);
3499:           }
3500:         });
3501:       });
3502: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `quantize_tensor_per_tensor_affine_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `quantize_tensor_per_tensor_affine_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 3504-3572
```cpp
3504: void dequantize_tensor_per_tensor_affine_cpu(
3505:     const Tensor& qtensor,
3506:     Tensor& rtensor,
3507:     double scale,
3508:     int64_t zero_point) {
3509:   AT_DISPATCH_QINT_TYPES(
3510:       qtensor.scalar_type(), "dequantize_tensor_per_tensor_affine_cpu", [&]() {
3511:         check_tensor_memory_format(qtensor, rtensor);
3512:         const auto* qd =
3513:             reinterpret_cast<const underlying_t*>(qtensor.data_ptr<scalar_t>());
3514:         fbgemm::TensorQuantizationParams qparams{};
3515:         qparams.scale = scale;
3516:         qparams.zero_point = zero_point;
3517:         qparams.precision = CHAR_BIT * sizeof(underlying_t);
3518:         float* rd = rtensor.data_ptr<float>();
3519:         int num_tasks = at::get_num_threads();
3520:         at::parallel_for(0, num_tasks, 1, [&](int64_t begin, int64_t end) {
3521:           for (const auto task_id : c10::irange(begin, end)) {
3522:             fbgemm::Dequantize<underlying_t>(
3523:                 qd, /*src=*/
3524:                 rd, /*dst=*/
3525:                 qtensor.numel(), /*len=*/
3526:                 qparams, /*qparams=*/
3527:                 task_id, /*thread_id*/
3528:                 num_tasks /*num_threads*/);
3529:           }
3530:         });
3531:       });
3532: }
3533: #else // USE_FBGEMM
3534:
3535: #if defined(__ARM_NEON__) || defined(__aarch64__)
3536:
3537: constexpr static int PARALLEL_THRESHOLD = 1 << 20;
3538:
3539: // Generic template defaults to naive quantize implementation
3540: template <typename T>
3541: void quantize_tensor_arm(
3542:     const float* __restrict__ in,
3543:     T* __restrict__ out,
3544:     const int64_t N,
3545:     const float scale,
3546:     const int32_t zero_point) {
3547:   for (const auto i : c10::irange(N)) {
3548:     out[i] = at::native::quantize_val<T>(scale, zero_point, in[i]);
3549:   }
3550: }
3551:
3552: namespace quantize_tensor_arm_intrinsics {
3553: template <typename Tx8>
3554: C10_ALWAYS_INLINE Tx8 vqmov(int16x8_t vraw);
3555:
3556: template <>
3557: C10_ALWAYS_INLINE uint8x8_t vqmov<uint8x8_t>(int16x8_t vraw) {
3558:   return vqmovun_s16(vraw);
3559: }
3560:
3561: template <>
3562: C10_ALWAYS_INLINE int8x8_t vqmov<int8x8_t>(int16x8_t vraw) {
3563:   return vqmovn_s16(vraw);
3564: }
3565:
3566: template <typename T, typename Tx8>
3567: C10_ALWAYS_INLINE void vst1(T* out, Tx8 vout);
3568:
3569: template <>
3570: C10_ALWAYS_INLINE void vst1<uint8_t, uint8x8_t>(uint8_t* out, uint8x8_t vout) {
3571:   vst1_u8(out, vout);
3572: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `dequantize_tensor_per_tensor_affine_cpu`, `quantize_tensor_arm`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `dequantize_tensor_per_tensor_affine_cpu`, `quantize_tensor_arm`，它们直接构成本文件的算子逻辑。

### Lines 3574-3659
```cpp
3574: template <>
3575: C10_ALWAYS_INLINE void vst1<int8_t, int8x8_t>(int8_t* out, int8x8_t vout) {
3576:   vst1_s8(out, vout);
3577: }
3578: } // namespace quantize_tensor_arm_intrinsics
3579:
3580: // Specialized implementation from caffe2::Int8Quantize.
3581: // There may be slight accuracy difference between this and implementation of
3582: // quantize_val
3583: // TODO Update quantize_tensor_arm implementation to follow quantize_val,
3584: // i.e. f = Round(value/scale + zero_point)
3585: // TODO Make quantize_tensor_arm work for int32 datatype too.
3586: template <typename scalar_t, typename underlying_t, typename underlying_x8_t>
3587: void quantize_tensor_arm_q8(
3588:     const float* __restrict__ in,
3589:     scalar_t* __restrict__ out,
3590:     const int64_t N,
3591:     const float scale,
3592:     const int32_t zero_point) {
3593:   const float inv_scale = 1.0f / scale;
3594:   uint32_t i = 0;
3595:   underlying_t* out_underlying = reinterpret_cast<underlying_t*>(out);
3596:   const float32x4_t vinv_scale = vdupq_n_f32(inv_scale);
3597: #if defined(__ARM_NEON__)
3598:   // magic float and magic int to take care of rounding
3599:   // int magic_round(float f): interpret_int32(f + 12582912.0f) - 0x4B400000
3600:   // Some detail:
3601:   // 12582912.0f is 2**23 + 2**22. The trick is based on the fact that when you
3602:   // add a small number to a large number, the result rounds to the precision of
3603:   // the least significant bit of the large number. For IEEE-754
3604:   // single-precision number mantissa has 23 bits, and adding 2**23 would cause
3605:   // rounding to the nearest even integer. The we cast to int and subtract the
3606:   // same number (0x4B400000 is the integer representation of 12582912.0f) to
3607:   // get only the mantissa. This works if -2**22 < x < 2**22, but preserves the
3608:   // sign for negative numbers.
3609:   const int32x4_t voffset = vdupq_n_s32(zero_point - 0x4B400000);
3610:   const float32x4_t vmagic_float = vdupq_n_f32(12582912.0f);
3611:   for (i = 0; i + 8 <= N; i += 8) {
3612:     const float32x4_t vin0123 = vld1q_f32(in);
3613:     in += 4;
3614:     const float32x4_t vin4567 = vld1q_f32(in);
3615:     in += 4;
3616:     const int32x4_t vraw0123 = vaddq_s32(
3617:         voffset,
3618:         vreinterpretq_s32_f32(
3619:             vaddq_f32(vmagic_float, vmulq_f32(vin0123, vinv_scale))));
3620:     const int32x4_t vraw4567 = vaddq_s32(
3621:         voffset,
3622:         vreinterpretq_s32_f32(
3623:             vaddq_f32(vmagic_float, vmulq_f32(vin4567, vinv_scale))));
3624:     const int16x8_t vraw01234567 =
3625:         vcombine_s16(vqmovn_s32(vraw0123), vqmovn_s32(vraw4567));
3626:     const underlying_x8_t vout01234567 =
3627:         quantize_tensor_arm_intrinsics::vqmov<underlying_x8_t>(vraw01234567);
3628:     quantize_tensor_arm_intrinsics::vst1<underlying_t, underlying_x8_t>(
3629:         out_underlying, vout01234567);
3630:     out_underlying += 8;
3631:   }
3632:   for (; i < N; ++i) {
3633:     (*out_underlying++) =
3634:         at::native::quantize_val_arm<underlying_t>(scale, zero_point, (*in++));
3635:   }
3636: #else
3637:   const int16x8_t vzero_point = vdupq_n_s16((int16_t)(uint16_t)zero_point);
3638:   for (i = 0; i + 8 <= N; i += 8) {
3639:     const float32x4_t vin0123 = vld1q_f32(in);
3640:     in += 4;
3641:     const float32x4_t vin4567 = vld1q_f32(in);
3642:     in += 4;
3643:     const int32x4_t v0123_rounded = vcvtnq_s32_f32(vmulq_f32(vin0123, vinv_scale));
3644:     const int32x4_t v4567_rounded = vcvtnq_s32_f32(vmulq_f32(vin4567, vinv_scale));
3645:     const int16x8_t v01234567_packed = vqaddq_s16(
3646:         vqmovn_high_s32(vqmovn_s32(v0123_rounded), v4567_rounded), vzero_point);
3647:     const underlying_x8_t vout01234567 =
3648:         quantize_tensor_arm_intrinsics::vqmov<underlying_x8_t>(
3649:             v01234567_packed);
3650:     quantize_tensor_arm_intrinsics::vst1<underlying_t, underlying_x8_t>(
3651:         out_underlying, vout01234567);
3652:     out_underlying += 8;
3653:   }
3654:   for (; i < N; ++i) {
3655:     (*out_underlying++) =
3656:         at::native::quantize_val_arm<underlying_t>(scale, zero_point, (*in++));
3657:   }
3658: #endif
3659: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `int8x8_t>`, `Round`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `int8x8_t>`, `Round`，它们直接构成本文件的算子逻辑。

### Lines 3661-3730
```cpp
3661: template <>
3662: void quantize_tensor_arm<c10::quint8>(
3663:     const float* __restrict__ in,
3664:     c10::quint8* __restrict__ out,
3665:     const int64_t N,
3666:     const float scale,
3667:     const int32_t zero_point) {
3668:   quantize_tensor_arm_q8<c10::quint8, uint8_t, uint8x8_t>(
3669:       in, out, N, scale, zero_point);
3670: }
3671:
3672: template <>
3673: void quantize_tensor_arm<c10::qint8>(
3674:     const float* __restrict__ in,
3675:     c10::qint8* __restrict__ out,
3676:     const int64_t N,
3677:     const float scale,
3678:     const int32_t zero_point) {
3679:   quantize_tensor_arm_q8<c10::qint8, int8_t, int8x8_t>(
3680:       in, out, N, scale, zero_point);
3681: }
3682:
3683: #if defined(__aarch64__)
3684: #define VMOVL_HIGH_U8(x) vmovl_high_u8(x)
3685: #define VMOVL_HIGH_S8(x) vmovl_high_s8(x)
3686: #define VMOVL_HIGH_U16(x) vmovl_high_u16(x)
3687: #define VMOVL_HIGH_S16(x) vmovl_high_s16(x)
3688: #else // vmovl_high intrinsic not supported
3689: #define VMOVL_HIGH_U8(x) vmovl_u8(vget_high_u8(x))
3690: #define VMOVL_HIGH_S8(x) vmovl_s8(vget_high_s8(x))
3691: #define VMOVL_HIGH_U16(x) vmovl_u16(vget_high_u16(x))
3692: #define VMOVL_HIGH_S16(x) vmovl_s16(vget_high_s16(x))
3693: #endif
3694:
3695: // Generic template defaults to naive dequantize implementation
3696: template <typename T>
3697: void dequantize_tensor_arm(
3698:     const T* __restrict__ in,
3699:     float* __restrict__ out,
3700:     const int64_t N,
3701:     const float scale,
3702:     const int32_t zero_point) {
3703:   for (int i = 0; i < N; ++i) {
3704:     out[i] = dequantize_val<T>(scale, zero_point, in[i]);
3705:   }
3706: }
3707:
3708: template <>
3709: void dequantize_tensor_arm<c10::qint8>(
3710:     const c10::qint8* __restrict__ in,
3711:     float* __restrict__ out,
3712:     const int64_t N,
3713:     const float scale,
3714:     const int32_t zero_point) {
3715:   const int8_t* in_underlying = reinterpret_cast<const int8_t*>(in);
3716:
3717:   const float32x4_t scale_fp32x4 = vdupq_n_f32(scale);
3718:   // Zero point is restricted to be in bounds of a signed 8 bit integer
3719:   const int8x8_t zero_point_s8x8 = vget_low_s8(vdupq_n_s8(static_cast<int8_t>(zero_point)));
3720:
3721:   int i;
3722:   for (i = 0; i + 16 <= N; i += 16) {
3723:     const int8x16_t vin_s8 = vld1q_s8(in_underlying);
3724:
3725:     // Extract upper or lower values to int16x8 and subtract zero point
3726:     // Each input element and the zero point are restricted to be in bounds of
3727:     // a signed 8 bit integer, so the difference will fit in a signed 16 bit
3728:     // integer
3729:     const int16x8_t minus_zp_low_s16 = vsubl_s8(vget_low_s8(vin_s8), zero_point_s8x8); // 0 ... 7
3730:     const int16x8_t minus_zp_high_s16 = vsubl_s8(vget_high_s8(vin_s8), zero_point_s8x8); // 8 ... 15
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `quint8>`, `qint8>`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `quint8>`, `qint8>`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 3732-3832
```cpp
3732:     const int32x4_t minus_zp_low_low = vmovl_s16(vget_low_s16(minus_zp_low_s16)); // 0 ... 3
3733:     const int32x4_t minus_zp_low_high = VMOVL_HIGH_S16(minus_zp_low_s16); // 4 ... 7
3734:     const int32x4_t minus_zp_high_low = vmovl_s16(vget_low_s16(minus_zp_high_s16)); // 8 ... 11
3735:     const int32x4_t minus_zp_high_high = VMOVL_HIGH_S16(minus_zp_high_s16); // 12 ... 15
3736:
3737:     // Store            * scale   int32->fp32
3738:     vst1q_f32(out,      vmulq_f32(vcvtq_f32_s32(minus_zp_low_low), scale_fp32x4));
3739:     vst1q_f32(out + 4,  vmulq_f32(vcvtq_f32_s32(minus_zp_low_high), scale_fp32x4));
3740:     vst1q_f32(out + 8,  vmulq_f32(vcvtq_f32_s32(minus_zp_high_low), scale_fp32x4));
3741:     vst1q_f32(out + 12, vmulq_f32(vcvtq_f32_s32(minus_zp_high_high), scale_fp32x4));
3742:
3743:     out += 16;
3744:     in += 16;
3745:     in_underlying += 16;
3746:   }
3747:
3748:   for (; i < N; ++i) { // use default dequantize for remaining vals
3749:     (*out++) = dequantize_val<c10::qint8>(scale, zero_point, (*in++));
3750:   }
3751: }
3752:
3753: template <>
3754: void dequantize_tensor_arm<c10::quint8>(
3755:     const c10::quint8* __restrict__ in,
3756:     float* __restrict__ out,
3757:     const int64_t N,
3758:     const float scale,
3759:     const int32_t zero_point) {
3760:   const uint8_t* in_underlying = reinterpret_cast<const uint8_t*>(in);
3761:
3762:   const float32x4_t scale_fp32x4 = vdupq_n_f32(scale);
3763:   // Zero point is restricted to be in bounds of an unsigned 8 bit integer
3764:   const uint8x8_t zero_point_u8x8 = vget_low_u8(vdupq_n_u8(static_cast<uint8_t>(zero_point)));
3765:
3766:   int i;
3767:   for (i = 0; i + 16 <= N; i += 16) {
3768:     const uint8x16_t vin_u8 = vld1q_u8(in_underlying);
3769:
3770:     // Extract upper or lower values to uint16x8 and subtract zero point
3771:     // Each input element and the zero point are restricted to be in bounds of
3772:     // an unsigned 8 bit integer, so the difference will fit in a signed 16 bit
3773:     // integer
3774:     const int16x8_t minus_zp_low_s16 = vreinterpretq_s16_u16(vsubl_u8(vget_low_u8(vin_u8), zero_point_u8x8)); // 0 ... 7
3775:     const int16x8_t minus_zp_high_s16 = vreinterpretq_s16_u16(vsubl_u8(vget_high_u8(vin_u8), zero_point_u8x8)); // 8 ... 15
3776:
3777:     const int32x4_t minus_zp_low_low = vmovl_s16(vget_low_s16(minus_zp_low_s16)); // 0 ... 3
3778:     const int32x4_t minus_zp_low_high = VMOVL_HIGH_S16(minus_zp_low_s16); // 4 ... 7
3779:     const int32x4_t minus_zp_high_low = vmovl_s16(vget_low_s16(minus_zp_high_s16)); // 8 ... 11
3780:     const int32x4_t minus_zp_high_high = VMOVL_HIGH_S16(minus_zp_high_s16); // 12 ... 15
3781:
3782:     // Store            * scale   int32->fp32
3783:     vst1q_f32(out,      vmulq_f32(vcvtq_f32_s32(minus_zp_low_low), scale_fp32x4));
3784:     vst1q_f32(out + 4,  vmulq_f32(vcvtq_f32_s32(minus_zp_low_high), scale_fp32x4));
3785:     vst1q_f32(out + 8,  vmulq_f32(vcvtq_f32_s32(minus_zp_high_low), scale_fp32x4));
3786:     vst1q_f32(out + 12, vmulq_f32(vcvtq_f32_s32(minus_zp_high_high), scale_fp32x4));
3787:
3788:     out += 16;
3789:     in += 16;
3790:     in_underlying += 16;
3791:   }
3792:
3793:   for (; i < N; ++i) { // use default dequantize for remaining vals
3794:     (*out++) = dequantize_val<c10::quint8>(scale, zero_point, (*in++));
3795:   }
3796: }
3797:
3798: #endif // defined(__ARM_NEON__) || defined(__aarch64__)
3799:
3800: void quantize_tensor_per_tensor_affine_cpu(
3801:     const Tensor& rtensor,
3802:     Tensor& qtensor,
3803:     double scale,
3804:     int64_t zero_point) {
3805:   check_tensor_memory_format(rtensor, qtensor);
3806:   const float* rdata = rtensor.const_data_ptr<float>();
3807:   int numel = rtensor.numel();
3808: #if defined(__ARM_NEON__) || defined(__aarch64__)
3809:   AT_DISPATCH_QINT_TYPES(
3810:       qtensor.scalar_type(), "quantize_tensor_per_tensor_affine_cpu", [&]() {
3811:         scalar_t* qdata = qtensor.data_ptr<scalar_t>();
3812:         auto quantize_range = [&](int64_t begin, int64_t end) {
3813:           quantize_tensor_arm<scalar_t>(
3814:             rdata + begin, qdata + begin, end - begin, scale, zero_point);
3815:         };
3816:         if (numel >= PARALLEL_THRESHOLD) {
3817:           at::parallel_for(0, numel, 1, quantize_range);
3818:         } else {
3819:           quantize_range(0, numel);
3820:         }
3821:       });
3822: #else
3823:   // Fallback path
3824:   AT_DISPATCH_QINT_TYPES(
3825:       qtensor.scalar_type(), "quantize_tensor_per_tensor_affine_cpu", [&]() {
3826:         scalar_t* qdata = qtensor.data_ptr<scalar_t>();
3827:         for (const auto i : c10::irange(numel)) {
3828:           qdata[i] = quantize_val<scalar_t>(scale, zero_point, rdata[i]);
3829:         }
3830:       });
3831: #endif // defined(__ARM_NEON__) || defined(__aarch64__)
3832: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `quint8>`, `quantize_tensor_per_tensor_affine_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `quint8>`, `quantize_tensor_per_tensor_affine_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 3834-3920
```cpp
3834: void dequantize_tensor_per_tensor_affine_cpu(
3835:     const Tensor& qtensor,
3836:     Tensor& rtensor,
3837:     double scale,
3838:     int64_t zero_point) {
3839:   check_tensor_memory_format(qtensor, rtensor);
3840:   float* rdata = rtensor.data_ptr<float>();
3841:   int numel = qtensor.numel();
3842: #if defined(__ARM_NEON__) || defined(__aarch64__)
3843:   AT_DISPATCH_QINT_TYPES(
3844:       qtensor.scalar_type(), "dequantize_tensor_per_tensor_affine_cpu", [&]() {
3845:         const scalar_t* qdata = qtensor.const_data_ptr<scalar_t>();
3846:         auto dequantize_range = [&](int64_t begin, int64_t end) {
3847:           dequantize_tensor_arm<scalar_t>(
3848:             qdata + begin, rdata + begin, end - begin, scale, zero_point);
3849:         };
3850:         if (numel >= PARALLEL_THRESHOLD) {
3851:           at::parallel_for(0, numel, 1, dequantize_range);
3852:         } else {
3853:           dequantize_range(0, numel);
3854:         }
3855:       });
3856: #else
3857:   // Fallback path
3858:   AT_DISPATCH_QINT_TYPES(
3859:       qtensor.scalar_type(), "dequantize_tensor_per_tensor_affine_cpu", [&]() {
3860:         const scalar_t* qdata = qtensor.const_data_ptr<scalar_t>();
3861:         for (const auto i : c10::irange(numel)) {
3862:           rdata[i] = dequantize_val<scalar_t>(scale, zero_point, qdata[i]);
3863:         }
3864:       });
3865: #endif // defined(__ARM_NEON__) || defined(__aarch64__)
3866: }
3867: #endif // USE_FBGEMM
3868:
3869: // TODO: add fbgemm for per channel
3870: // Generic template defaults to naive quantize implementation
3871: template <typename T>
3872: void quantize_tensor_per_channel_impl(
3873:     const Tensor& rtensor,
3874:     Tensor& qtensor,
3875:     const Tensor& scales,
3876:     const Tensor& zero_points,
3877:     int64_t axis) {
3878:   // TODO: channels last kernel can be made faster.
3879:   // For contiguous tensors, e.g. NCHW, arbitrary axis can be used.
3880:   // For channels_last/3d however axis == 0 or 1.
3881:   // Since current implementation on channels_last format does not
3882:   // cover per channel quant with arbitrary axis value, it is better
3883:   // to check and fail.
3884:   int64_t batches = size_to_dim_(axis, rtensor.sizes());
3885:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
3886:   int64_t elements_per_channel = size_from_dim_(axis + 1, rtensor.sizes());
3887:   int64_t channels = rtensor.size(axis);
3888:   auto scales_data = scales.data_ptr<double>();
3889:   auto zero_points_data = zero_points.data_ptr<int64_t>();
3890:   const float* in = rtensor.const_data_ptr<float>();
3891:   auto out = qtensor.data_ptr<T>();
3892:   if (axis == 1 &&
3893:       (rtensor.is_contiguous(MemoryFormat::ChannelsLast) ||
3894:        rtensor.is_contiguous(MemoryFormat::ChannelsLast3d))) {
3895:     // This code handles per channel quant when axis = 1 and
3896:     // channels_last contig.
3897:     // If axis = 0 and channels_last contig, implementation for channels
3898:     // first (NCHW) works.
3899:     for (const auto b : c10::irange(batches)) {
3900:       for (const auto e : c10::irange(elements_per_channel)) {
3901:         for (const auto c : c10::irange(channels)) {
3902:           auto i = b * channels * elements_per_channel + e * channels + c;
3903:           out[i] = at::native::quantize_val<T>(
3904:               scales_data[c], zero_points_data[c], in[i]);
3905:         }
3906:       }
3907:     }
3908:   } else {
3909:     for (const auto b : c10::irange(batches)) {
3910:       for (const auto c : c10::irange(channels)) {
3911:         for (const auto e : c10::irange(elements_per_channel)) {
3912:           auto i = b * channels * elements_per_channel +
3913:               c * elements_per_channel + e;
3914:           out[i] = at::native::quantize_val<T>(
3915:               scales_data[c], zero_points_data[c], in[i]);
3916:         }
3917:       }
3918:     }
3919:   }
3920: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `dequantize_tensor_per_tensor_affine_cpu`, `quantize_tensor_per_channel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `dequantize_tensor_per_tensor_affine_cpu`, `quantize_tensor_per_channel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 3922-3990
```cpp
3922: #if defined(__ARM_NEON__) || defined(__aarch64__)
3923: // Specialized implementation from caffe2::Int8Quantize.
3924: // There may be slight accuracy difference between this and implementation of
3925: // quantize_val
3926: // TODO Update quantize_tensor_per_channel_impl implementation to follow
3927: // quantize_val, i.e. f = Round(value/scale + zero_point)
3928: // TODO Make quantize_tensor_per_channel_impl work for other datatypes too
3929: // (int8, int32).
3930: template <>
3931: void quantize_tensor_per_channel_impl<c10::quint8>(
3932:     const Tensor& rtensor,
3933:     Tensor& qtensor,
3934:     const Tensor& scales,
3935:     const Tensor& zero_points,
3936:     int64_t axis) {
3937:   int64_t batches = size_to_dim_(axis, rtensor.sizes());
3938:   int64_t elements_per_channel = size_from_dim_(axis + 1, rtensor.sizes());
3939:   int64_t channels = rtensor.size(axis);
3940:   auto scales_data = scales.data_ptr<double>();
3941:   auto zero_points_data = zero_points.data_ptr<int64_t>();
3942:   const float* in = rtensor.const_data_ptr<float>();
3943:   auto out = (uint8_t*)qtensor.data_ptr<c10::quint8>();
3944: #if defined(__ARM_NEON__)
3945:   // magic float and magic int to take care of rounding
3946:   // int magic_round(float f): interpret_int32(f + 12582912.0f) - 0x4B400000
3947:   // Some detail:
3948:   // 12582912.0f is 2**23 + 2**22. The trick is based on the fact that when you
3949:   // add a small number to a large number, the result rounds to the precision of
3950:   // the least significant bit of the large number. For IEEE-754
3951:   // single-precision number mantissa has 23 bits, and adding 2**23 would cause
3952:   // rounding to the nearest even integer. The we cast to int and subtract the
3953:   // same number (0x4B400000 is the integer representation of 12582912.0f) to
3954:   // get only the mantissa. This works if -2**22 < x < 2**22, but preserves the
3955:   // sign for negative numbers.
3956:   const float32x4_t vmagic_float = vdupq_n_f32(12582912.0f);
3957:   // Copy reciprocal of scales (double) into float array
3958:   // Copy zero_points with magic int (int64_t) into int32_t array
3959:   std::vector<float> inv_scales(channels);
3960:   std::vector<int32_t> zero_points_int32t(channels);
3961:   for (const auto i : c10::irange(channels)) {
3962:     inv_scales[i] = 1.0f / (float)scales_data[i];
3963:     zero_points_int32t[i] = (int32_t)(uint32_t)zero_points_data[i] - 0x4B400000;
3964:   }
3965:   if (axis == 1 &&
3966:       (rtensor.is_contiguous(MemoryFormat::ChannelsLast) ||
3967:        rtensor.is_contiguous(MemoryFormat::ChannelsLast3d))) {
3968:     // This code handles per channel quant when axis = 1 and
3969:     // channels_last contig.
3970:     // If axis = 0 and channels_last contig, implementation for channels
3971:     // first (NCHW) works.
3972:     for ([[maybe_unused]] const auto b : c10::irange(batches)) {
3973:       for ([[maybe_unused]] const auto e : c10::irange(elements_per_channel)) {
3974:         uint32_t c = 0;
3975:         while (c + 8 < channels) {
3976:           const int32x4_t voffset0123 = vld1q_s32(&zero_points_int32t[c]);
3977:           const float32x4_t vinv_scale0123 = vld1q_f32(&inv_scales[c]);
3978:           c += 4;
3979:           const int32x4_t voffset4567 = vld1q_s32(&zero_points_int32t[c]);
3980:           const float32x4_t vinv_scale4567 = vld1q_f32(&inv_scales[c]);
3981:           c += 4;
3982:           const float32x4_t vin0123 = vld1q_f32(in);
3983:           in += 4;
3984:           const float32x4_t vin4567 = vld1q_f32(in);
3985:           in += 4;
3986:           const int32x4_t vraw0123 = vaddq_s32(
3987:               voffset0123,
3988:               vreinterpretq_s32_f32(
3989:                   vaddq_f32(vmagic_float, vmulq_f32(vin0123, vinv_scale0123))));
3990:           const int32x4_t vraw4567 = vaddq_s32(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `first`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `first`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 3991-4060
```cpp
3991:               voffset4567,
3992:               vreinterpretq_s32_f32(
3993:                   vaddq_f32(vmagic_float, vmulq_f32(vin4567, vinv_scale4567))));
3994:           const int16x8_t vraw01234567 =
3995:               vcombine_s16(vqmovn_s32(vraw0123), vqmovn_s32(vraw4567));
3996:           const uint8x8_t vout01234567 = vqmovun_s16(vraw01234567);
3997:           vst1_u8(out, vout01234567);
3998:           out += 8;
3999:         }
4000:         for (; c < channels; ++c) {
4001:           (*out++) = at::native::quantize_val_arm<uint8_t>(
4002:               scales_data[c], zero_points_data[c], (*in++));
4003:         }
4004:       }
4005:     }
4006:   } else {
4007:     for ([[maybe_unused]] const auto b : c10::irange(batches)) {
4008:       for (const auto c : c10::irange(channels)) {
4009:         uint32_t e = 0;
4010:         const int32x4_t voffset = vdupq_n_s32(zero_points_int32t[c]);
4011:         const float32x4_t vinv_scale = vdupq_n_f32(inv_scales[c]);
4012:         for (; e + 8 < elements_per_channel; e += 8) {
4013:           const float32x4_t vin0123 = vld1q_f32(in);
4014:           in += 4;
4015:           const float32x4_t vin4567 = vld1q_f32(in);
4016:           in += 4;
4017:           const int32x4_t vraw0123 = vaddq_s32(
4018:               voffset,
4019:               vreinterpretq_s32_f32(
4020:                   vaddq_f32(vmagic_float, vmulq_f32(vin0123, vinv_scale))));
4021:           const int32x4_t vraw4567 = vaddq_s32(
4022:               voffset,
4023:               vreinterpretq_s32_f32(
4024:                   vaddq_f32(vmagic_float, vmulq_f32(vin4567, vinv_scale))));
4025:           const int16x8_t vraw01234567 =
4026:               vcombine_s16(vqmovn_s32(vraw0123), vqmovn_s32(vraw4567));
4027:           const uint8x8_t vout01234567 = vqmovun_s16(vraw01234567);
4028:           vst1_u8(out, vout01234567);
4029:           out += 8;
4030:         }
4031:         for (; e < elements_per_channel; ++e) {
4032:           (*out++) = at::native::quantize_val_arm<uint8_t>(
4033:               scales_data[c], zero_points_data[c], (*in++));
4034:         }
4035:       }
4036:     }
4037:   }
4038: #else // defined(__ARM_NEON__)
4039:   // Copy scales (double) into float array
4040:   // Copy zero_points (int64_t) into int16_t array
4041:   std::vector<float> inv_scales(channels);
4042:   std::vector<int16_t> zero_points_int16t(channels);
4043:   for (const auto i : c10::irange(channels)) {
4044:     inv_scales[i] = 1.0f / (float)scales_data[i];
4045:     zero_points_int16t[i] = (int16_t)(uint16_t)zero_points_data[i];
4046:   }
4047:   if (axis == 1 &&
4048:       (rtensor.is_contiguous(MemoryFormat::ChannelsLast) ||
4049:        rtensor.is_contiguous(MemoryFormat::ChannelsLast3d))) {
4050:     // This code handles per channel quant when axis = 1 and
4051:     // channels_last contig.
4052:     // If axis = 0 and channels_last contig, implementation for channels
4053:     // first (NCHW) works.
4054:     for ([[maybe_unused]] const auto b : c10::irange(batches)) {
4055:       for ([[maybe_unused]] const auto e : c10::irange(elements_per_channel)) {
4056:         uint32_t c = 0;
4057:         while (c + 8 < channels) {
4058:           const int16x8_t vzero_point = vld1q_s16(&zero_points_int16t[c]);
4059:           const float32x4_t vinv_scale0123 = vld1q_f32(&inv_scales[c]);
4060:           c += 4;
```
- EN: The main symbol in this range is `first`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `first`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 4061-4133
```cpp
4061:           const float32x4_t vinv_scale4567 = vld1q_f32(&inv_scales[c]);
4062:           c += 4;
4063:           const float32x4_t vin0123 = vld1q_f32(in);
4064:           in += 4;
4065:           const float32x4_t vin4567 = vld1q_f32(in);
4066:           in += 4;
4067:           const int32x4_t v0123_rounded =
4068:               vcvtnq_s32_f32(vmulq_f32(vin0123, vinv_scale0123));
4069:           const int32x4_t v4567_rounded =
4070:               vcvtnq_s32_f32(vmulq_f32(vin4567, vinv_scale4567));
4071:           const int16x8_t v01234567_packed = vqaddq_s16(
4072:               vqmovn_high_s32(vqmovn_s32(v0123_rounded), v4567_rounded),
4073:               vzero_point);
4074:           const uint8x8_t vout01234567 = vqmovun_s16(v01234567_packed);
4075:           vst1_u8(out, vout01234567);
4076:           out += 8;
4077:         }
4078:         for (; c < channels; ++c) {
4079:           (*out++) = at::native::quantize_val_arm<uint8_t>(
4080:               scales_data[c], zero_points_data[c], (*in++));
4081:         }
4082:       }
4083:     }
4084:   } else {
4085:     for ([[maybe_unused]] const auto b : c10::irange(batches)) {
4086:       for ([[maybe_unused]] const auto c : c10::irange(channels)) {
4087:         uint32_t e = 0;
4088:         const int16x8_t vzero_point = vdupq_n_s16(zero_points_int16t[c]);
4089:         const float32x4_t vinv_scale = vdupq_n_f32(inv_scales[c]);
4090:         for (; e + 8 < elements_per_channel; e += 8) {
4091:           const float32x4_t vin0123 = vld1q_f32(in);
4092:           in += 4;
4093:           const float32x4_t vin4567 = vld1q_f32(in);
4094:           in += 4;
4095:           const int32x4_t v0123_rounded =
4096:               vcvtnq_s32_f32(vmulq_f32(vin0123, vinv_scale));
4097:           const int32x4_t v4567_rounded =
4098:               vcvtnq_s32_f32(vmulq_f32(vin4567, vinv_scale));
4099:           const int16x8_t v01234567_packed = vqaddq_s16(
4100:               vqmovn_high_s32(vqmovn_s32(v0123_rounded), v4567_rounded),
4101:               vzero_point);
4102:           const uint8x8_t vout01234567 = vqmovun_s16(v01234567_packed);
4103:           vst1_u8(out, vout01234567);
4104:           out += 8;
4105:         }
4106:         for (; e < elements_per_channel; ++e) {
4107:           (*out++) = at::native::quantize_val_arm<uint8_t>(
4108:               scales_data[c], zero_points_data[c], (*in++));
4109:         }
4110:       }
4111:     }
4112:   }
4113: #endif // defined(__ARM_NEON__)
4114: }
4115: #endif // defined(__ARM_NEON__) || defined(__aarch64__)
4116:
4117: void quantize_tensor_per_channel_affine_cpu(
4118:     const Tensor& rtensor,
4119:     Tensor& qtensor,
4120:     const Tensor& scales,
4121:     const Tensor& zero_points,
4122:     int64_t axis) {
4123:   TORCH_CHECK(
4124:       rtensor.is_contiguous() || (axis <= 1),
4125:       "If tensor is channels_last contig then per channel quantization "
4126:       "is supported only for axis = 0 or 1.");
4127:   AT_DISPATCH_QINT_TYPES(
4128:       qtensor.scalar_type(), "quantize_tensor_per_channel_affine_cpu", [&]() {
4129:         check_tensor_memory_format(rtensor, qtensor);
4130:         quantize_tensor_per_channel_impl<scalar_t>(
4131:             rtensor, qtensor, scales, zero_points, axis);
4132:       });
4133: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `quantize_tensor_per_channel_affine_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `quantize_tensor_per_channel_affine_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 4135-4210
```cpp
4135: template<typename T, typename N, typename Q>
4136: void dequantize_per_channel_affine_kernel(
4137:       const Tensor& qtensor,
4138:       Tensor& rtensor,
4139:       const Tensor& scales,
4140:       const Tensor& zero_points,
4141:       int64_t axis,
4142:       int bit_width=8) {
4143:
4144:   // For contiguous tensors, e.g. NCHW, arbitrary axis can be used.
4145:   // For channels_last/3d however axis == 0 or 1.
4146:   // Since current implementation on channels_last format does not
4147:   // cover per channel quant with arbitrary axis value, it is better
4148:   // to check and fail.
4149:   TORCH_CHECK(rtensor.is_contiguous() || (axis <=1),
4150:       "If tensor is channels_last contig then per channel quantization "
4151:       "is supported only for axis = 0 or 1.");
4152:   int64_t batches = size_to_dim_(axis, rtensor.sizes());
4153:   int64_t elements_per_channel =
4154:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
4155:       size_from_dim_(axis + 1, rtensor.sizes());
4156:   int64_t channel = rtensor.size(axis);
4157:   auto scales_data = scales.data_ptr<T>();
4158:   auto zero_points_data = zero_points.data_ptr<N>();
4159:   check_tensor_memory_format(qtensor, rtensor);
4160:   const auto* qd = qtensor.const_data_ptr<Q>();
4161:   float* rd = rtensor.data_ptr<float>();
4162:   const auto elem_per_byte = 8 / bit_width;
4163:   if (axis == 1 && (rtensor.is_contiguous(MemoryFormat::ChannelsLast) ||
4164:       rtensor.is_contiguous(MemoryFormat::ChannelsLast3d))) {
4165:     for (const auto b : c10::irange(batches)) {
4166:       for (const auto e : c10::irange(elements_per_channel)) {
4167:         for (const auto c : c10::irange(channel)) {
4168:           auto i = b * channel * elements_per_channel + e * channel + c;
4169:           // We need to convert the qint8 value to float to ensure the
4170:           // subtraction subexpression returns a float
4171:           auto qvalue = qd[i / elem_per_byte].val_;
4172:           if (bit_width < 8) {
4173:             qvalue >>= (i % elem_per_byte) * bit_width;
4174:             qvalue &= (1 << bit_width) - 1;
4175:           }
4176:           rd[i] = (static_cast<float>(qvalue) - zero_points_data[c]) * scales_data[c];
4177:         }
4178:       }
4179:     }
4180:   } else {
4181:     for (const auto b : c10::irange(batches)) {
4182:       for (const auto c : c10::irange(channel)) {
4183:         for (const auto e : c10::irange(elements_per_channel)) {
4184:           auto i = b * channel * elements_per_channel +
4185:               c * elements_per_channel + e;
4186:           // We need to convert the qint8 value to float to ensure the
4187:           // subtraction subexpression returns a float
4188:           auto qvalue = qd[i / elem_per_byte].val_;
4189:           if (bit_width < 8) {
4190:             qvalue >>= (i % elem_per_byte) * bit_width;
4191:             qvalue &= (1 << bit_width) - 1;
4192:           }
4193:           rd[i] = (static_cast<float>(qvalue) - zero_points_data[c]) * scales_data[c];
4194:         }
4195:       }
4196:     }
4197:   }
4198: }
4199:
4200: void dequantize_tensor_per_channel_affine_cpu(
4201:     const Tensor& qtensor,
4202:     Tensor& rtensor,
4203:     const Tensor& scales,
4204:     const Tensor& zero_points,
4205:     int64_t axis) {
4206:   AT_DISPATCH_QINT_TYPES(
4207:       qtensor.scalar_type(), "dequantize_tensor_per_channel_affine_cpu", [&]() {
4208:         dequantize_per_channel_affine_kernel<double, int64_t, scalar_t>(qtensor, rtensor, scales, zero_points, axis);
4209:       });
4210: }
```
- EN: The main symbol in this range is `dequantize_per_channel_affine_kernel`, `dequantize_tensor_per_channel_affine_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `dequantize_per_channel_affine_kernel`, `dequantize_tensor_per_channel_affine_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 4212-4286
```cpp
4212: // quantize stubs for floating point scale and zero_point.
4213: void quantize_tensor_per_channel_float_qparams_cpu(
4214:     const Tensor& rtensor,
4215:     Tensor& qtensor,
4216:     const Tensor& scales,
4217:     const Tensor& zero_points,
4218:     int64_t axis) {
4219:   // For contiguous tensors, e.g. NCHW, arbitrary axis can be used.
4220:   // For channels_last/3d however axis == 0 or 1.
4221:   // Since current implementation on channels_last format does not
4222:   // cover per channel quant with arbitrary axis value, it is better
4223:   // to check and fail.
4224:   TORCH_CHECK(rtensor.is_contiguous() || (axis <=1),
4225:       "If tensor is channels_last contig then per channel quantization "
4226:       "is supported only for axis = 0 or 1.");
4227:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(
4228:       qtensor.scalar_type(), "quantize_tensor_per_channel_float_qparams_cpu", [&]() {
4229:         int64_t batches = size_to_dim_(axis, rtensor.sizes());
4230:         int64_t elements_per_channel =
4231:             size_from_dim_(axis + 1, rtensor.sizes());
4232:         int64_t channel = rtensor.size(axis);
4233:         auto scales_data = scales.data_ptr<float>();
4234:         auto zero_points_data = zero_points.data_ptr<float>();
4235:         check_tensor_memory_format(rtensor, qtensor);
4236:         const float* rdata = rtensor.const_data_ptr<float>();
4237:         auto qdata = reinterpret_cast<underlying_t*>(qtensor.data_ptr<scalar_t>());
4238:         const auto elem_per_byte = CHAR_BIT / bit_width;
4239:         int qvalue = 0;
4240:         if (axis == 1 && (rtensor.is_contiguous(MemoryFormat::ChannelsLast) ||
4241:             rtensor.is_contiguous(MemoryFormat::ChannelsLast3d))) {
4242:           for (const auto b : c10::irange(batches)) {
4243:             for (const auto e : c10::irange(elements_per_channel)) {
4244:               for (const auto c : c10::irange(channel)) {
4245:                 auto i = b * channel * elements_per_channel + e * channel + c;
4246:                 qvalue = quantize_val_float_qparams(
4247:                     scales_data[c], zero_points_data[c], rdata[i], quant_min, quant_max);
4248:                 if (i % elem_per_byte == 0) {
4249:                   qdata[i / elem_per_byte] = static_cast<underlying_t>(qvalue);
4250:                 } else {
4251:                   qdata[i / elem_per_byte] |= static_cast<underlying_t>(qvalue << ((i % elem_per_byte) * bit_width));
4252:                 }
4253:               }
4254:             }
4255:           }
4256:         } else {
4257:           for (const auto b : c10::irange(batches)) {
4258:             for (const auto c : c10::irange(channel)) {
4259:               for (const auto e : c10::irange(elements_per_channel)) {
4260:                 auto i = b * channel * elements_per_channel +
4261:                     c * elements_per_channel + e;
4262:                 qvalue = quantize_val_float_qparams(
4263:                     scales_data[c], zero_points_data[c], rdata[i], quant_min, quant_max);
4264:                 if (i % elem_per_byte == 0) {
4265:                   qdata[i / elem_per_byte] = static_cast<underlying_t>(qvalue);
4266:                 } else {
4267:                   qdata[i / elem_per_byte] |= static_cast<underlying_t>(qvalue << ((i % elem_per_byte) * bit_width));
4268:                 }
4269:               }
4270:             }
4271:           }
4272:         }
4273:       });
4274: }
4275:
4276: void dequantize_tensor_per_channel_float_qparams_cpu(
4277:     const Tensor& qtensor,
4278:     Tensor& rtensor,
4279:     const Tensor& scales,
4280:     const Tensor& zero_points,
4281:     int64_t axis) {
4282:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(
4283:       qtensor.scalar_type(), "dequantize_tensor_per_channel_float_qparams_cpu", [&]() {
4284:         dequantize_per_channel_affine_kernel<float, float, scalar_t>(qtensor, rtensor, scales, zero_points, axis, bit_width);
4285:       });
4286: }
```
- EN: The main symbol in this range is `quantize_tensor_per_channel_float_qparams_cpu`, `dequantize_tensor_per_channel_float_qparams_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `quantize_tensor_per_channel_float_qparams_cpu`, `dequantize_tensor_per_channel_float_qparams_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 4288-4356
```cpp
4288: void quantize_tensor_per_tensor_affine_sub_byte_cpu(
4289:     const Tensor& rtensor,
4290:     Tensor& qtensor,
4291:     float scale,
4292:     float zero_point) {
4293:   // TODO Use fbgemm kernel to pack values
4294:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(
4295:     qtensor.scalar_type(), "quantize_tensor_per_tensor_affine_sub_byte_cpu", [&]() {
4296:       check_tensor_memory_format(rtensor, qtensor);
4297:       const float* const rdata = rtensor.const_data_ptr<float>();
4298:       auto qdata = reinterpret_cast<underlying_t*>(qtensor.data_ptr<scalar_t>());
4299:       auto numel = rtensor.numel();
4300:       const auto elem_per_byte = CHAR_BIT / bit_width;
4301:       for (const auto i : c10::irange(numel)) {
4302:         float inv_scale = scale == 0 ? 1.0f : 1.0f / scale;
4303:         int64_t qvalue = lrintf(std::nearbyint(rdata[i] * inv_scale) + zero_point);
4304:         qvalue = std::max(quant_min, std::min(qvalue, quant_max));
4305:
4306:         // We pack sub_byte values and align them to a byte.
4307:         // Eg. for 4-bits Index 0 is packed in the lower 4-bits
4308:         // and index 1 is packed in the upper 4-bits.
4309:         if (i % elem_per_byte == 0) {
4310:           qdata[i / elem_per_byte] = static_cast<underlying_t>(qvalue);
4311:         } else {
4312:           qdata[i / elem_per_byte] |= static_cast<underlying_t>(qvalue << ((i % elem_per_byte) * bit_width));
4313:         }
4314:       } // for numel
4315:     });
4316: }
4317:
4318: void dequantize_tensor_per_tensor_affine_sub_byte_cpu(
4319:     const Tensor& qtensor,
4320:     Tensor& rtensor,
4321:     float scale,
4322:     float zero_point) {
4323:   // TODO Use fbgemm kernel to pack values
4324:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(
4325:     qtensor.scalar_type(), "dequantize_tensor_per_tensor_affine_sub_byte_cpu", [&]() {
4326:       check_tensor_memory_format(rtensor, qtensor);
4327:       auto rdata = rtensor.data_ptr<float>();
4328:       const underlying_t* qdata = reinterpret_cast<const underlying_t*>(qtensor.const_data_ptr<scalar_t>());
4329:       auto numel = rtensor.numel();
4330:       const auto elem_per_byte = CHAR_BIT / bit_width;
4331:
4332:       for (const auto i : c10::irange(numel)) {
4333:         underlying_t qvalue = qdata[i / elem_per_byte];
4334:         qvalue >>= (i % elem_per_byte) * bit_width;
4335:         qvalue &= (1 << bit_width) - 1;
4336:         rdata[i] = (static_cast<float>(qvalue) - zero_point) * scale;
4337:       }
4338:   });
4339: }
4340:
4341: // This function expects quantized_val input to already be quantized
4342: template <typename scalar_t>
4343: void cpu_masked_fill_kernel_quantized_cpu(TensorIterator& iter, scalar_t quantized_val) {
4344:   auto loop = [&](char** data, const int64_t* strides, int64_t n) {
4345:     char* dst = data[0];
4346:     char* mask = data[1];
4347:     for (const auto i : c10::irange(n)) {
4348:       bool mask_value = *reinterpret_cast<bool*>(mask + strides[1] * i);
4349:
4350:       if (mask_value) {
4351:         *(scalar_t*)(dst + strides[0] * i) = quantized_val;
4352:       }
4353:     }
4354:   };
4355:   iter.for_each(loop);
4356: }
```
- EN: The main symbol in this range is `quantize_tensor_per_tensor_affine_sub_byte_cpu`, `dequantize_tensor_per_tensor_affine_sub_byte_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `quantize_tensor_per_tensor_affine_sub_byte_cpu`, `dequantize_tensor_per_tensor_affine_sub_byte_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 4358-4426
```cpp
4358: void masked_fill_kernel_quantized_cpu(TensorIterator& iter, const Scalar& value, double scale, int zero_point) {
4359:   AT_DISPATCH_QINT_TYPES(iter.dtype(), "masked_fill", [&] {
4360:     float float_val = value.to<float>();
4361:     auto quantized_val = quantize_val<scalar_t>(scale, zero_point, float_val);
4362:     auto mask_dtype = iter.input_dtype(0);
4363:     TORCH_CHECK(mask_dtype == ScalarType::Bool, "masked_fill only supports boolean masks, "
4364:       "but got mask with dtype ", mask_dtype);
4365:     cpu_masked_fill_kernel_quantized_cpu<scalar_t>(iter, quantized_val);
4366:   });
4367: }
4368:
4369: // currently, we do not support accumulate=True for quantized tensors. We throw an exception in _index_put_impl_quantized_cpu_
4370: void index_put_kernel_quantized_cpu(TensorIterator& iter, IntArrayRef index_size, IntArrayRef index_stride, bool accumulate, double scale, int zero_point) {
4371:   // NOTE: duplicate indices are only supported if accumulate is true.
4372:   AT_DISPATCH_QINT_TYPES(iter.dtype(), "index_put", [&] {
4373:     // See Note [Enabling Deterministic Operations]
4374:     // Parallel cpu_index_kernel with accumulation is nondeterministic, so we
4375:     // must enable serial execution if deterministic algorithms are enabled.
4376:     const bool is_deterministic = at::globalContext().deterministicAlgorithms();
4377:     at::native::cpu_index_kernel<scalar_t>(iter, index_size, index_stride, [scale, zero_point](char* dst, char* src, int64_t offset) {
4378:       *(scalar_t*)(dst + offset) = quantize_val<scalar_t>(scale, zero_point, *(float*)src);
4379:     }, /*serial_execution=*/is_deterministic);
4380:   });
4381: }
4382:
4383: template<typename T>
4384: void _qmul_tensor_cpu_impl(
4385:     T* out_ptr,
4386:     int64_t size,
4387:     const uint8_t* x_ptr,
4388:     double x_scale,
4389:     int64_t x_zero_point,
4390:     const uint8_t* y_ptr,
4391:     double y_scale,
4392:     int64_t y_zero_point,
4393:     double output_scale,
4394:     int64_t output_zero_point) {
4395:   float multiplier = x_scale * y_scale / output_scale;
4396:   auto compute_with_scalar = [&](int idx) {
4397:     uint8_t x_data = *(x_ptr + idx);
4398:     uint8_t y_data = *(y_ptr + idx);
4399:     int32_t x_val = static_cast<int32_t>(x_data) - x_zero_point;
4400:     int32_t y_val = static_cast<int32_t>(y_data) - y_zero_point;
4401:     int32_t out_val = x_val * y_val;
4402:     float out_val_f = (float)out_val * multiplier;
4403:     if constexpr (std::is_same<T, float>::value) {
4404:       *(out_ptr + idx) = out_val_f;
4405:     } else if constexpr (std::is_same<T, at::BFloat16>::value) {
4406:       *(out_ptr + idx) = at::BFloat16(out_val_f);
4407:     } else if constexpr (std::is_same<T, at::Half>::value) {
4408:       *(out_ptr + idx) = at::Half(out_val_f);
4409:     } else { //  T == uint8, requantization needed
4410:       out_val_f = std::round(out_val_f);
4411:       int32_t out_val_i32 = (int32_t)out_val_f + output_zero_point;
4412:       out_val_i32 = std::min(255, std::max(0, out_val_i32));
4413:       *(out_ptr + idx) = static_cast<uint8_t>(out_val_i32);
4414:     }
4415:   };
4416: #if defined(CPU_CAPABILITY_AVX512)
4417:   int64_t size_rem = size % 16;
4418:   int64_t size_com = size - size_rem;
4419:   int64_t steps = size_com / 16;
4420:   __m512 vs = _mm512_set1_ps(multiplier);
4421:   __m512i vza = _mm512_set1_epi32(x_zero_point);
4422:   __m512i vzb = _mm512_set1_epi32(y_zero_point);
4423:   __m512i vzc = _mm512_set1_epi32(output_zero_point);
4424:   __m512i v255 = _mm512_set1_epi32(255);
4425:   __m512i v0 = _mm512_set1_epi32(0);
4426:   at::parallel_for(0, steps, 1, [&](int64_t start, int64_t end) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `masked_fill_kernel_quantized_cpu`, `index_put_kernel_quantized_cpu`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `masked_fill_kernel_quantized_cpu`, `index_put_kernel_quantized_cpu`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 4427-4496
```cpp
4427:     for (const auto d : c10::irange(start, end)) {
4428:       auto x_data = x_ptr + d * 16;
4429:       auto y_data = y_ptr + d * 16;
4430:       auto out_data = out_ptr + d * 16;
4431:       __m128i va = _mm_loadu_si128((__m128i*)x_data);
4432:       __m128i vb = _mm_loadu_si128((__m128i*)y_data);
4433:       __m512i va_i32 = _mm512_cvtepi8_epi32(va);
4434:       __m512i vb_i32 = _mm512_cvtepi8_epi32(vb);
4435:       va_i32 = _mm512_sub_epi32(va_i32, vza);
4436:       vb_i32 = _mm512_sub_epi32(vb_i32, vzb);
4437:       __m512i vc = _mm512_mullo_epi32(va_i32, vb_i32);
4438:       __m512 vc_f = _mm512_cvtepi32_ps(vc);
4439:       vc_f = _mm512_mul_ps(vc_f, vs);
4440:       if constexpr (std::is_same<T, float>::value) {
4441:         _mm512_storeu_ps(out_data, vc_f);
4442:       } else if constexpr (std::is_same<T, at::BFloat16>::value) {
4443:         __m256i vc_bf16 = cvtfp32_bf16(vc_f);
4444:         _mm256_storeu_si256((__m256i*)out_data, vc_bf16);
4445:       } else if constexpr (std::is_same<T, at::Half>::value) {
4446:         __m256i vc_f16 = cvtfp32_fp16(vc_f);
4447:         _mm256_storeu_si256((__m256i*)out_data, vc_f16);
4448:       } else { //  T == uint8, requantization needed
4449:         __m512i vc_i32 = _mm512_cvtps_epi32(vc_f);
4450:         vc_i32 = _mm512_add_epi32(vc_i32, vzc);
4451:         vc_i32 = _mm512_min_epi32(vc_i32, v255);
4452:         vc_i32 = _mm512_max_epi32(vc_i32, v0);
4453:         __m128i vc_i8 = _mm512_cvtepi32_epi8(vc_i32);
4454:         _mm_storeu_si128((__m128i*)out_data, vc_i8);
4455:       }
4456:     }
4457:   });
4458:   if (size_rem > 0) {
4459:     for (const auto d : c10::irange(size_rem)) {
4460:       compute_with_scalar(size_com + d);
4461:     }
4462:   }
4463: #else
4464:   at::parallel_for(0, size, 1, [&](int64_t start, int64_t end) {
4465:     for (const auto d : c10::irange(start, end)) {
4466:       compute_with_scalar(d);
4467:     }
4468:   });
4469: #endif
4470: }
4471:
4472: void qmul_tensor_cpu_kernel(
4473:     Tensor& out,
4474:     const Tensor& qx,
4475:     double qx_scale,
4476:     int64_t qx_zero_point,
4477:     const Tensor& qy,
4478:     double qy_scale,
4479:     int64_t qy_zero_point,
4480:     double output_scale,
4481:     int64_t output_zero_point) {
4482:   auto qx_ptr = qx.const_data_ptr<uint8_t>();
4483:   auto qy_ptr = qy.const_data_ptr<uint8_t>();
4484:   int64_t size = qx.numel();
4485:   TORCH_CHECK(
4486:       size == qy.numel() && size == out.numel(),
4487:       "qmul_cpu: Expect qx, qy and out to have the same number of elements");
4488:   AT_DISPATCH_FLOATING_TYPES_AND3(
4489:       at::ScalarType::BFloat16, at::ScalarType::Half, at::ScalarType::Byte, out.scalar_type(), "int8_mul_cpu", [&] {
4490:         auto out_ptr = out.data_ptr<scalar_t>();
4491:         _qmul_tensor_cpu_impl<scalar_t>(
4492:             out_ptr, size, qx_ptr, qx_scale, qx_zero_point, qy_ptr, qy_scale, qy_zero_point, output_scale, output_zero_point);
4493:       });
4494: }
4495:
4496: template<typename T, bool ReLUFused>
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `constexpr`, `qmul_tensor_cpu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `constexpr`, `qmul_tensor_cpu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 4497-4596
```cpp
4497: void _qadd_tensor_cpu_impl(
4498:     T* out_ptr,
4499:     int64_t size,
4500:     const uint8_t* x_ptr,
4501:     double x_scale,
4502:     int64_t x_zero_point,
4503:     const uint8_t* y_ptr,
4504:     double y_scale,
4505:     int64_t y_zero_point,
4506:     double output_scale,
4507:     int64_t output_zero_point) {
4508:   float inv_output_scale = 1.0 / output_scale;
4509:   auto compute_with_scalar = [&](int idx) {
4510:     uint8_t x_data = *(x_ptr + idx);
4511:     uint8_t y_data = *(y_ptr + idx);
4512:     int32_t x_val = static_cast<int32_t>(x_data) - x_zero_point;
4513:     int32_t y_val = static_cast<int32_t>(y_data) - y_zero_point;
4514:     float x_val_f = static_cast<float>(x_val) * x_scale;
4515:     float y_val_f = static_cast<float>(y_val) * y_scale;
4516:     float out_val_f = x_val_f + y_val_f;
4517:     if constexpr (ReLUFused) {
4518:       out_val_f = std::max(out_val_f, 0.f);
4519:     }
4520:     if constexpr (std::is_same<T, float>::value) {
4521:       *(out_ptr + idx) = out_val_f;
4522:     } else if constexpr (std::is_same<T, at::BFloat16>::value) {
4523:       *(out_ptr + idx) = at::BFloat16(out_val_f);
4524:     } else if constexpr (std::is_same<T, at::Half>::value) {
4525:       *(out_ptr + idx) = at::Half(out_val_f);
4526:     } else { //  T == uint8, requantization needed
4527:       out_val_f = std::round(out_val_f * inv_output_scale);
4528:       int32_t out_val_i32 = (int32_t)out_val_f + output_zero_point;
4529:       out_val_i32 = std::min(255, std::max(0, out_val_i32));
4530:       *(out_ptr + idx) = static_cast<uint8_t>(out_val_i32);
4531:     }
4532:   };
4533: #if defined(CPU_CAPABILITY_AVX512)
4534:   int64_t size_rem = size % 16;
4535:   int64_t size_com = size - size_rem;
4536:   int64_t steps = size_com / 16;
4537:   __m512 vsa = _mm512_set1_ps(x_scale);
4538:   __m512 vsb = _mm512_set1_ps(y_scale);
4539:   __m512 vsc = _mm512_set1_ps(inv_output_scale);
4540:   __m512i vza = _mm512_set1_epi32(x_zero_point);
4541:   __m512i vzb = _mm512_set1_epi32(y_zero_point);
4542:   __m512i vzc = _mm512_set1_epi32(output_zero_point);
4543:   __m512i v255 = _mm512_set1_epi32(255);
4544:   __m512i v0 = _mm512_set1_epi32(0);
4545:   __m512 v0f = _mm512_set1_ps(0);
4546:   at::parallel_for(0, steps, 1, [&](int64_t start, int64_t end) {
4547:     for (const auto d : c10::irange(start, end)) {
4548:       auto x_data = x_ptr + d * 16;
4549:       auto y_data = y_ptr + d * 16;
4550:       auto out_data = out_ptr + d * 16;
4551:       __m128i va = _mm_loadu_si128((__m128i*)x_data);
4552:       __m128i vb = _mm_loadu_si128((__m128i*)y_data);
4553:       __m512i va_i32 = _mm512_cvtepi8_epi32(va);
4554:       __m512i vb_i32 = _mm512_cvtepi8_epi32(vb);
4555:       va_i32 = _mm512_sub_epi32(va_i32, vza);
4556:       vb_i32 = _mm512_sub_epi32(vb_i32, vzb);
4557:       __m512 va_f = _mm512_cvtepi32_ps(va_i32);
4558:       __m512 vb_f = _mm512_cvtepi32_ps(vb_i32);
4559:       va_f = _mm512_mul_ps(va_f, vsa);
4560:       vb_f = _mm512_mul_ps(vb_f, vsb);
4561:       __m512 vc_f = _mm512_add_ps(va_f, vb_f);
4562:       if constexpr (ReLUFused) {
4563:         vc_f = _mm512_max_ps(vc_f, v0f);
4564:       }
4565:       if constexpr (std::is_same<T, float>::value) {
4566:         _mm512_storeu_ps(out_data, vc_f);
4567:       } else if constexpr (std::is_same<T, at::BFloat16>::value) {
4568:         __m256i vc_bf16 = cvtfp32_bf16(vc_f);
4569:         _mm256_storeu_si256((__m256i*)out_data, vc_bf16);
4570:       } else if constexpr (std::is_same<T, at::Half>::value) {
4571:         __m256i vc_f16 = cvtfp32_fp16(vc_f);
4572:         _mm256_storeu_si256((__m256i*)out_data, vc_f16);
4573:       } else { //  T == uint8, requantization needed
4574:         vc_f = _mm512_mul_ps(vc_f, vsc);
4575:         __m512i vc_i32 = _mm512_cvtps_epi32(vc_f);
4576:         vc_i32 = _mm512_add_epi32(vc_i32, vzc);
4577:         vc_i32 = _mm512_min_epi32(vc_i32, v255);
4578:         vc_i32 = _mm512_max_epi32(vc_i32, v0);
4579:         __m128i vc_i8 = _mm512_cvtepi32_epi8(vc_i32);
4580:         _mm_storeu_si128((__m128i*)out_data, vc_i8);
4581:       }
4582:     }
4583:   });
4584:   if (size_rem > 0) {
4585:     for (const auto d : c10::irange(size_rem)) {
4586:       compute_with_scalar(size_com + d);
4587:     }
4588:   }
4589: #else
4590:   at::parallel_for(0, size, 1, [&](int64_t start, int64_t end) {
4591:     for (const auto d : c10::irange(start, end)) {
4592:       compute_with_scalar(d);
4593:     }
4594:   });
4595: #endif
4596: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_qadd_tensor_cpu_impl`, `constexpr`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_qadd_tensor_cpu_impl`, `constexpr`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 4598-4667
```cpp
4598: template <bool ReLUFused>
4599: void qadd_tensor_cpu_kernel(
4600:     Tensor& out,
4601:     const Tensor& qx,
4602:     double qx_scale,
4603:     int64_t qx_zero_point,
4604:     const Tensor& qy,
4605:     double qy_scale,
4606:     int64_t qy_zero_point,
4607:     double output_scale,
4608:     int64_t output_zero_point) {
4609:   auto qx_ptr = qx.const_data_ptr<uint8_t>();
4610:   auto qy_ptr = qy.const_data_ptr<uint8_t>();
4611:   int64_t size = qx.numel();
4612:   TORCH_CHECK(
4613:       size == qy.numel() && size == out.numel(),
4614:       "qadd_cpu: Expect qx, qy and out to have the same number of elements");
4615:   AT_DISPATCH_FLOATING_TYPES_AND3(
4616:       at::ScalarType::BFloat16, at::ScalarType::Half, at::ScalarType::Byte, out.scalar_type(), "int8_add_cpu", [&] {
4617:         auto out_ptr = out.data_ptr<scalar_t>();
4618:         _qadd_tensor_cpu_impl<scalar_t, ReLUFused>(
4619:             out_ptr, size, qx_ptr, qx_scale, qx_zero_point, qy_ptr, qy_scale, qy_zero_point, output_scale, output_zero_point);
4620:       });
4621: }
4622: } // anonymous namespace
4623:
4624: // Some quantization tests are flaky on Windows with AVX512. If --continue-through-error
4625: // is used, only one fails. But if the failing test is skipped, another one fails.
4626: // If the second test is also skipped, a third one fails.
4627: // So, until Quantization support for Windows is fixed for AVX512,
4628: // AVX2 kernels would be used instead. Ref: GH 56992.
4629: #if defined(_WIN32)
4630: REGISTER_DISPATCH(dequantize_tensor_per_channel_affine_stub,
4631:                   &dequantize_tensor_per_channel_affine_cpu)
4632: REGISTER_DISPATCH(dequantize_tensor_per_channel_float_qparams_stub,
4633:                   &dequantize_tensor_per_channel_float_qparams_cpu)
4634: REGISTER_DISPATCH(fake_quant_per_channel_cachemask_stub,
4635:                   &fake_quant_per_channel_cachemask_cpu)
4636: REGISTER_DISPATCH(qavg_pool2d_nhwc_stub, &qavg_pool2d_nhwc_kernel)
4637: REGISTER_DISPATCH(qavg_pool3d_nhwc_stub, &qavg_pool3d_nhwc_kernel)
4638: #else
4639: // These kernels are dispatched to AVX512
4640: ALSO_REGISTER_AVX512_DISPATCH(dequantize_tensor_per_channel_affine_stub,
4641:                   &dequantize_tensor_per_channel_affine_cpu)
4642: ALSO_REGISTER_AVX512_DISPATCH(dequantize_tensor_per_channel_float_qparams_stub,
4643:                   &dequantize_tensor_per_channel_float_qparams_cpu)
4644: ALSO_REGISTER_AVX512_DISPATCH(fake_quant_per_channel_cachemask_stub,
4645:                   &fake_quant_per_channel_cachemask_cpu)
4646: ALSO_REGISTER_AVX512_DISPATCH(qavg_pool2d_nhwc_stub, &qavg_pool2d_nhwc_kernel)
4647: ALSO_REGISTER_AVX512_DISPATCH(qavg_pool3d_nhwc_stub, &qavg_pool3d_nhwc_kernel)
4648: #endif // CPU_CAPABILITY_AVX512 && _WIN32
4649:
4650: // The kernels below are dispatched to AVX2 because they don't perform as well
4651: // with AVX512. We might revisit this decision in the near future.
4652: REGISTER_DISPATCH(dequantize_tensor_per_tensor_affine_stub,
4653:                   &dequantize_tensor_per_tensor_affine_cpu)
4654: REGISTER_DISPATCH(fake_quant_grad_learnable_tensor_stub,
4655:                   &fake_quantize_learnable_tensor_grad_kernel_cpu)
4656: REGISTER_DISPATCH(fake_quant_tensor_cachemask_stub,
4657:                   &fake_quantize_tensor_cachemask_kernel)
4658: REGISTER_DISPATCH(fake_quant_tensor_cachemask_tensor_qparams_stub,
4659:                   &fake_quantize_tensor_cachemask_tensor_qparams_kernel)
4660: REGISTER_DISPATCH(qadaptive_avg_pool2d_nhwc_stub,
4661:                   &qadaptive_avg_pool2d_nhwc_kernel)
4662: REGISTER_DISPATCH(qadaptive_avg_pool3d_ndhwc_stub,
4663:                   &qadaptive_avg_pool3d_ndhwc_kernel)
4664: REGISTER_DISPATCH(qadd_relu_stub, &qadd_kernel<true>)
4665: REGISTER_DISPATCH(qadd_scalar_relu_stub, &qadd_scalar_kernel<true>)
4666: REGISTER_DISPATCH(qadd_scalar_stub, &qadd_scalar_kernel<false>)
4667: REGISTER_DISPATCH(qadd_stub, &qadd_kernel<false>)
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `qadd_tensor_cpu_kernel`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `qadd_tensor_cpu_kernel`，它们直接构成本文件的算子逻辑。

### Lines 4669-4725
```cpp
4669: REGISTER_DISPATCH(qbatch_norm_relu_stub, &q_batch_norm_kernel<true>)
4670: REGISTER_DISPATCH(qbatch_norm_stub, &q_batch_norm_kernel<false>)
4671: REGISTER_DISPATCH(qcat_nhwc_stub, &qcat_nhwc_kernel<false>)
4672: REGISTER_DISPATCH(qcat_relu_nhwc_stub, &qcat_nhwc_kernel<true>)
4673: REGISTER_DISPATCH(qclamp_stub, &qclamp_kernel)
4674: REGISTER_DISPATCH(qclamp_min_stub, &qclamp_min_kernel)
4675: REGISTER_DISPATCH(qclamp_max_stub, &qclamp_max_kernel)
4676: REGISTER_DISPATCH(qelu_stub, &qelu_kernel)
4677: REGISTER_DISPATCH(qhardsigmoid_stub, &qhardsigmoid_kernel)
4678: REGISTER_DISPATCH(qhardswish_stub, &qhardswish_kernel)
4679: REGISTER_DISPATCH(qmaxpool_2d_nhwc_stub, &qmaxpool_2d_nhwc_kernel)
4680: REGISTER_DISPATCH(qmaxpool_3d_nthwc_stub, &qmaxpool_3d_nthwc_kernel)
4681: REGISTER_DISPATCH(qmul_relu_stub, &qmul_kernel<true>)
4682: REGISTER_DISPATCH(qmul_stub, &qmul_kernel<false>)
4683: REGISTER_DISPATCH(qrelu_leaky_stub, &leaky_qrelu_out_kernel)
4684: REGISTER_DISPATCH(qrelu_stub, &qrelu_kernel)
4685: REGISTER_DISPATCH(qprelu_stub, &qprelu_out_kernel)
4686: REGISTER_DISPATCH(qgelu_stub, &qgelu_kernel)
4687: REGISTER_DISPATCH(qsigmoid_stub, &qsigmoid_kernel)
4688: REGISTER_DISPATCH(qtanh_stub, &qtanh_kernel)
4689: REGISTER_DISPATCH(qthreshold_stub, &qthreshold_kernel)
4690: REGISTER_DISPATCH(qtopk_stub, &qtopk_kernel)
4691: REGISTER_DISPATCH(fake_quant_grad_learnable_channel_stub,
4692:                   &fake_quantize_learnable_channel_grad_kernel_cpu)
4693: REGISTER_DISPATCH(
4694:     quantize_tensor_per_tensor_affine_stub,
4695:     &quantize_tensor_per_tensor_affine_cpu)
4696: REGISTER_DISPATCH(
4697:     quantize_tensor_per_channel_affine_stub,
4698:     &quantize_tensor_per_channel_affine_cpu)
4699: REGISTER_DISPATCH(
4700:     quantize_tensor_per_channel_float_qparams_stub,
4701:     &quantize_tensor_per_channel_float_qparams_cpu)
4702: REGISTER_DISPATCH(quantized_normalize_stub, &quantized_normalize_kernel)
4703: REGISTER_DISPATCH(quantized_groupnorm_nhwc_stub, &quantized_groupnorm_nhwc_kernel)
4704: REGISTER_DISPATCH(qupsample_bilinear2d_nhwc_stub,
4705:                   &qupsample_bilinear2d_nhwc_kernel)
4706: REGISTER_DISPATCH(
4707:     quantize_tensor_per_tensor_affine_sub_byte_stub,
4708:     &quantize_tensor_per_tensor_affine_sub_byte_cpu)
4709: REGISTER_DISPATCH(
4710:     dequantize_tensor_per_tensor_affine_sub_byte_stub,
4711:     &dequantize_tensor_per_tensor_affine_sub_byte_cpu)
4712: REGISTER_DISPATCH(
4713:     masked_fill_kernel_quantized_stub,
4714:     &masked_fill_kernel_quantized_cpu)
4715: REGISTER_DISPATCH(
4716:     index_put_kernel_quantized_stub,
4717:     &index_put_kernel_quantized_cpu)
4718: REGISTER_DISPATCH(qmean_inner_dim_stub, &qmean_inner_dim_kernel)
4719: REGISTER_DISPATCH(qstd_inner_dim_stub, &qstd_inner_dim_kernel)
4720: ALSO_REGISTER_AVX512_DISPATCH(qmul_tensor_cpu_stub, &qmul_tensor_cpu_kernel)
4721: ALSO_REGISTER_AVX512_DISPATCH(qadd_tensor_cpu_stub, &qadd_tensor_cpu_kernel<false>)
4722: ALSO_REGISTER_AVX512_DISPATCH(qadd_relu_tensor_cpu_stub, &qadd_tensor_cpu_kernel<true>)
4723: ALSO_REGISTER_AVX512_DISPATCH(qbatch_norm_cpu_stub, &q_batch_norm_cpu_kernel)
4724: } // namespace at::native
4725: // NOLINTEND(*-c-arrays)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/native/Activation.h`, `ATen/native/TopKImpl.h`, `ATen/native/TensorIterator.h`, `ATen/native/UpSample.h`, `ATen/native/cpu/IndexKernelUtils.h`, `ATen/native/cpu/Loops.h`
- c10 headers / c10 头文件: `c10/util/irange.h`, `c10/util/Unroll.h`
- Standard or third-party headers / 标准库或第三方头文件: `cmath`, `fbgemm/QuantUtils.h`, `omp.h`, `arm_neon.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `cpu_kernel`, `cpu_kernel_vec`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `AT_DISPATCH_REDUCED_FLOATING_TYPES`, `Scalar`, `ScalarType`
