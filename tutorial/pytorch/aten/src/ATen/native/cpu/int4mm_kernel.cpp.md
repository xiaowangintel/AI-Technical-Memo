# int4mm_kernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/int4mm_kernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for int4mm kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 int4mm kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: #include <type_traits>
 2: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 3: #include <ATen/core/Tensor.h>
 4:
 5: #include <ATen/Dispatch.h>
 6: #include <ATen/Parallel.h>
 7: #include <ATen/cpu/vec/functional.h>
 8: #include <ATen/cpu/vec/vec.h>
 9: #include <ATen/native/cpu/int_mm_kernel.h>
10: #include <ATen/native/cpu/utils.h>
11: #include <cmath>
12: #include <c10/util/Unroll.h>
13: #include <c10/util/irange.h>
14:
15: #ifndef AT_PER_OPERATOR_HEADERS
16: #include <ATen/Functions.h>
17: #else
18: #include <ATen/ops/cat.h>
19: #endif
20:
21: #if AT_KLEIDIAI_ENABLED()
22: #include <ATen/native/kleidiai/kai_kernels.h>
23: #include <cpuinfo.h>
24: #endif
25:
26: #if (defined(_WIN32) || defined(_WIN64))
27: #define RESTRICT __restrict
28: #else
29: #define RESTRICT __restrict__
30: #endif
```
- EN: This range pulls in required headers, including `type_traits`, `ATen/core/Tensor.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `type_traits`, `ATen/core/Tensor.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 32-73
```cpp
32: namespace at::native {
33:
34: namespace {
35:
36: inline bool is_block_start(int index, int BLOCK_SIZE) {
37:   return !(index & (BLOCK_SIZE -1));
38: }
39:
40: #if (defined(CPU_CAPABILITY_AVX512) || defined(CPU_CAPABILITY_AVX2)) && !defined(_MSC_VER)
41: // convert 16x int4 to int8, handle 64 bits at a time
42: // used in avx2 and avx512
43: inline __m128i convert_int4_to_int8(const uint8_t* data) {
44:   __m128i tmp = _mm_loadu_si64((const __m128i*)data);
45:   __m128i bytes = _mm_cvtepu8_epi16(tmp);
46:   const __m128i lowMask = _mm_set1_epi8(0xF);
47:   __m128i high = _mm_andnot_si128(lowMask, bytes);
48:   __m128i low = _mm_and_si128(lowMask, bytes);
49:   high = _mm_slli_epi16(high, 4);
50:   bytes = _mm_or_si128(low, high);
51:   return bytes;
52: }
53: #endif
54:
55: #if defined(CPU_CAPABILITY_AVX512) && !defined(_MSC_VER)
56:
57: // A block : {BLOCK_M, BLOCK_K}, lda = K
58: // B block : {BLOCK_K, BLOCK_N / 2}, ldb = BLOCK_N / 2
59: // C block : {BLOCK_M, BLOCK_N}, ldc = N
60: //
61: // ScaleAndZeros block : {1, BLOCK_N, 2}
62: //
63: template <int BLOCK_M, int BLOCK_N>
64: inline void tinygemm_kernel(
65:     const BFloat16* RESTRICT A,
66:     const uint8_t* RESTRICT B,
67:     const BFloat16* RESTRICT ScaleAndZeros,
68:     BFloat16* RESTRICT C,
69:     int lda,
70:     int ldb,
71:     int ldc,
72:     int K,
73:     int BLOCK_K) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `is_block_start`, `tinygemm_kernel`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `is_block_start`, `tinygemm_kernel`，它们直接构成本文件的算子逻辑。

### Lines 75-107
```cpp
 75:   constexpr int ROWS = BLOCK_M;
 76:   constexpr int COLS = BLOCK_N / 16;
 77:
 78:   const int PREFETCH_SIZE_K = 16 * 4;
 79:   const int PREFETCH_SIZE_KB = (PREFETCH_SIZE_K + BLOCK_K - 1) / BLOCK_K;
 80:
 81:   // number of blocks on K
 82:   const int KB = K / BLOCK_K;
 83:
 84:   __m512 va;
 85:   __m512 vb[COLS];
 86:   __m512 vc[ROWS * COLS];
 87:   __m512 scale[COLS];
 88:   __m512 zero[COLS];
 89:
 90:   // Lookup table to de-quantize int4 values to bf16.
 91:   // Values are dequantized as truly int4 [-8, 7] range;
 92:   //
 93:   // dequant = (bf16(int4_value) * bf16_scale) + bf16_zero
 94:   //
 95:   static const __m512 lut = _mm512_set_ps(
 96:       7.0f, 6.0f, 5.0f, 4.0f,
 97:       3.0f, 2.0f, 1.0f, 0.0f,
 98:       -1.0f, -2.0f, -3.0f, -4.0f,
 99:       -5.0f, -6.0f, -7.0f, -8.0f);
100:
101:   // index for transpose
102:   static const __m512i idx1 = _mm512_set_epi32(
103:       30, 28, 26, 24, 22, 20, 18, 16,
104:       14, 12, 10, 8, 6, 4, 2, 0);
105:   static const __m512i idx2 = _mm512_set_epi32(
106:       31, 29, 27, 25, 23, 21, 19, 17,
107:       15, 13, 11, 9, 7, 5, 3, 1);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 109-139
```cpp
109:   // load scale and zero point
110:   auto load_scale_and_zeros = [&](int i, int _kb) {
111:     // load 2x bfloat16 vector
112:     __m512i t = _mm512_loadu_si512((__m512i*)(ScaleAndZeros + _kb * ldc * 2 + 32 * i));
113:     if (_kb + PREFETCH_SIZE_KB < KB) {
114:       _mm_prefetch(ScaleAndZeros + (_kb + PREFETCH_SIZE_KB) * ldc * 2 + 32 * i, _MM_HINT_T0);
115:     }
116:
117:     // convert to 2x f32 vector
118:     __m512 a, b;
119:     vec::cvtbf16_fp32(t, a, b);
120:
121:     // transpose scale_and_zero from {16, 2} to {2, 16}
122:     // inputs:
123:     //   a: {s0, z0, s1, z1, ..., s7, z7}
124:     //   b: {s8, z8, s9, z9, ..., s15, z15}
125:     // output:
126:     //   scale: {s0, s1, s2, ..., s15}
127:     //   zero:  {z0, z1, z2, ..., z15}
128:     scale[i] = _mm512_mask_permutex2var_ps(a, 0xffff, idx1, b);
129:     zero[i] = _mm512_mask_permutex2var_ps(a, 0xffff, idx2, b);
130:   };
131:
132:   auto loadc = [&](auto i) {
133:     vc[i] = _mm512_setzero_ps();
134:   };
135:   c10::ForcedUnroll<ROWS * COLS>{}(loadc);
136:
137:   auto compute = [&, COLS](auto i, int k) {
138:     constexpr  int row = i / COLS;
139:     constexpr  int col = i % COLS;
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 141-177
```cpp
141:     if constexpr (col == 0) {
142:       float aa = static_cast<float>(A[row * lda + k]);
143:       if (k + PREFETCH_SIZE_K < K) {
144:         _mm_prefetch(A + row * lda + k + PREFETCH_SIZE_K, _MM_HINT_T0);
145:       }
146:       va = _mm512_set1_ps(aa);
147:     }
148:
149:     if constexpr (row == 0) {
150:       if constexpr (COLS == 4) {
151:         // when BLOCK_N = 64, handle each row at a time
152:         // to reduce de-quantize overhead.
153:         if constexpr (col == 0) {
154:           __m256i b4 = _mm256_loadu_si256((__m256i*)(B + k * ldb));
155:           if (k + PREFETCH_SIZE_K < K) {
156:             _mm_prefetch(B + (k + PREFETCH_SIZE_K) * ldb, _MM_HINT_T0);
157:           }
158:
159:           __m512i b32 = _mm512_cvtepu8_epi32(_mm256_castsi256_si128(b4));
160:           vb[0] = _mm512_permutexvar_ps(b32, lut);
161:           vb[0] = _mm512_fmadd_ps(vb[0], scale[0], zero[0]);
162:           vb[2] = _mm512_permutexvar_ps(_mm512_srli_epi32(b32, 4), lut);
163:           vb[2] = _mm512_fmadd_ps(vb[2], scale[2], zero[2]);
164:
165:           b32 = _mm512_cvtepu8_epi32(_mm256_extracti128_si256(b4, 1));
166:           vb[1] = _mm512_permutexvar_ps(b32, lut);
167:           vb[1] = _mm512_fmadd_ps(vb[1], scale[1], zero[1]);
168:           vb[3] = _mm512_permutexvar_ps(_mm512_srli_epi32(b32, 4), lut);
169:           vb[3] = _mm512_fmadd_ps(vb[3], scale[3], zero[3]);
170:         }
171:       } else {
172:         __m128i b8 = convert_int4_to_int8(B + k * ldb + col * 8);
173:         __m512i b32 = _mm512_cvtepu8_epi32(b8);
174:         vb[col] = _mm512_permutexvar_ps(b32, lut);
175:         vb[col] = _mm512_fmadd_ps(vb[col], scale[col], zero[col]);
176:       }
177:     }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 179-209
```cpp
179:     constexpr int idx = row * COLS + col;
180:     vc[idx] = _mm512_fmadd_ps(va, vb[col], vc[idx]);
181:   };
182:
183:   for (int k = 0, kb = 0; k < K; ++k) {
184:     if (is_block_start(k, BLOCK_K)) {
185:       c10::ForcedUnroll<COLS>{}(load_scale_and_zeros, kb++);
186:     }
187:     c10::ForcedUnroll<ROWS * COLS>{}(compute, k);
188:   }
189:
190:   //store to C
191:   auto storec = [&, COLS](auto i) {
192:     constexpr int row = i / COLS;
193:     constexpr int col = i % COLS;
194:     if constexpr (COLS == 4) {
195:       // when BLOCK_N = 64, handle each row at a time
196:       // to reduce `cvtfp32_bf16` overhead.
197:       if constexpr (col == 0) {
198:         __m512i c01 = vec::cvtfp32_bf16(vc[row * 4 + 0], vc[row * 4 + 1]);
199:         __m512i c23 = vec::cvtfp32_bf16(vc[row * 4 + 2], vc[row * 4 + 3]);
200:         _mm512_storeu_si512((__m512i*)(C + row * ldc + 0 * 32), c01);
201:         _mm512_storeu_si512((__m512i*)(C + row * ldc + 1 * 32), c23);
202:       }
203:     } else {
204:       __m256i ci = vec::cvtfp32_bf16(vc[i]);
205:       _mm256_storeu_si256((__m256i*)(C + row * ldc + col * 16), ci);
206:     }
207:   };
208:   c10::ForcedUnroll<ROWS * COLS>{}(storec);
209: }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 211-240
```cpp
211: #elif defined(CPU_CAPABILITY_AVX2) && !defined(_MSC_VER)
212:
213: template <int BLOCK_M, int BLOCK_N>
214: inline void tinygemm_kernel(
215:     const BFloat16* RESTRICT A,
216:     const uint8_t* RESTRICT B,
217:     const BFloat16* RESTRICT ScaleAndZeros,
218:     BFloat16* RESTRICT C,
219:     int lda,
220:     int ldb,
221:     int ldc,
222:     int K,
223:     int BLOCK_K) {
224:
225:   constexpr int ROWS = BLOCK_M;
226:   constexpr int COLS = BLOCK_N / 8;
227:
228:   const int PREFETCH_SIZE_K = 16 * 4;
229:   const int PREFETCH_SIZE_KB = (PREFETCH_SIZE_K + BLOCK_K - 1) / BLOCK_K;
230:
231:   // number of blocks on K
232:   const int KB = K / BLOCK_K;
233:
234:   __m256 va;
235:   __m256 vb[COLS];
236:   __m256 vc[ROWS * COLS];
237:   __m256 scale[COLS];
238:   __m256 zero[COLS];
239:
240:   static const __m256i idx1 = _mm256_setr_epi32(0, 2, 4, 6, 1, 3, 5, 7);
```
- EN: The main symbol in this range is `tinygemm_kernel`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `tinygemm_kernel`，它们直接构成本文件的算子逻辑。

### Lines 242-271
```cpp
242:   // offset to shift from range [0, 15] to [-8, 7]
243:   const __m256 offset = _mm256_set1_ps(-8.0f);
244:
245:   // load scale and zero point
246:   auto load_scale_and_zeros = [&](int i, int _kb) {
247:     // load 2x bfloat16 vector
248:     __m256i t = _mm256_loadu_si256((__m256i*)(ScaleAndZeros + _kb * ldc * 2 + 16 * i));
249:     if (_kb + PREFETCH_SIZE_KB < KB) {
250:       _mm_prefetch(ScaleAndZeros + (_kb + PREFETCH_SIZE_KB) * ldc * 2 + 16 * i, _MM_HINT_T0);
251:     }
252:
253:     // convert to 2x f32 vector
254:     __m256 a, b;
255:     vec::cvtbf16_fp32(t, a, b);
256:
257:     // transpose scale_and_zero from {8, 2} to {2, 8}
258:     // inputs:
259:     //   a: {s0, z0, s1, z1, s2, z2, s3, z3}
260:     //   b: {s4, z4, s5, z5, s6, z6, s7, z7}
261:     // output:
262:     //   scale: {s0, s1, s2, s3, s4, s5, s6, s7}
263:     //   zero:  {z0, z1, z2, z3, z4, z5, z6, z7}
264:     a = _mm256_permutevar8x32_ps(a, idx1);
265:     b = _mm256_permutevar8x32_ps(b, idx1);
266:     scale[i] = _mm256_permute2f128_ps(a, b, 0b0100000);
267:     zero[i] = _mm256_permute2f128_ps(a, b, 0b0110001);
268:
269:     // zero = -8 * scale + zero
270:     zero[i] = _mm256_fmadd_ps(scale[i], offset, zero[i]);
271:   };
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 273-304
```cpp
273:   auto loadc = [&](auto i) {
274:     vc[i] = _mm256_setzero_ps();
275:   };
276:   c10::ForcedUnroll<ROWS * COLS>{}(loadc);
277:
278:   auto compute = [&, COLS](auto i, int k) {
279:     constexpr int row = i / COLS;
280:     constexpr int col = i % COLS;
281:
282:     if constexpr (col == 0) {
283:       float aa = static_cast<float>(A[row * lda + k]);
284:       if (k + PREFETCH_SIZE_K < K) {
285:         _mm_prefetch(A + row * lda + k + PREFETCH_SIZE_K, _MM_HINT_T0);
286:       }
287:       va = _mm256_set1_ps(aa);
288:     }
289:
290:     if constexpr (row == 0) {
291:       if constexpr (COLS == 4) {
292:         // when BLOCK_N = 32, handle each row at a time
293:         if constexpr (col == 0) {
294:           __m256i mask = _mm256_set1_epi32(0xF);
295:           __m128i b4 = _mm_loadu_si128((__m128i*)(B + k * ldb));
296:           if (k + PREFETCH_SIZE_K < K) {
297:             _mm_prefetch(B + (k + PREFETCH_SIZE_K) * ldb, _MM_HINT_T0);
298:           }
299:
300:           __m256i b32 = _mm256_cvtepu8_epi32(b4);
301:           vb[0] = _mm256_cvtepi32_ps(_mm256_and_si256(b32, mask));
302:           vb[0] = _mm256_fmadd_ps(vb[0], scale[0], zero[0]);
303:           vb[2] = _mm256_cvtepi32_ps(_mm256_srli_epi32(b32, 4));
304:           vb[2] = _mm256_fmadd_ps(vb[2], scale[2], zero[2]);
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 306-338
```cpp
306:           b32 = _mm256_cvtepu8_epi32(_mm_shuffle_epi32(b4, _MM_SHUFFLE(3, 2, 3, 2)));
307:           vb[1] = _mm256_cvtepi32_ps(_mm256_and_si256(b32, mask));
308:           vb[1] = _mm256_fmadd_ps(vb[1], scale[1], zero[1]);
309:           vb[3] = _mm256_cvtepi32_ps(_mm256_srli_epi32(b32, 4));
310:           vb[3] = _mm256_fmadd_ps(vb[3], scale[3], zero[3]);
311:         }
312:       } else {
313:         if constexpr (col % 2 == 0) {
314:           // de-quantize per 64 bits (16x int4)
315:           __m128i b8 = convert_int4_to_int8(B + k * ldb + col * 4);
316:           __m128i b8_val0 = _mm_set1_epi64x(_mm_extract_epi64(b8, 0));
317:           __m128i b8_val1 = _mm_set1_epi64x(_mm_extract_epi64(b8, 1));
318:           if (k + PREFETCH_SIZE_K < K) {
319:             _mm_prefetch(B + (k + PREFETCH_SIZE_K) * ldb + col * 4, _MM_HINT_T0);
320:           }
321:
322:           vb[col] = _mm256_cvtepi32_ps(_mm256_cvtepu8_epi32(b8_val0));
323:           vb[col] = _mm256_fmadd_ps(vb[col], scale[col], zero[col]);
324:           vb[col + 1] = _mm256_cvtepi32_ps(_mm256_cvtepu8_epi32(b8_val1));
325:           vb[col + 1] = _mm256_fmadd_ps(vb[col + 1], scale[col + 1], zero[col + 1]);
326:         }
327:       }
328:     }
329:
330:     constexpr int idx = row * COLS + col;
331:     vc[idx] = _mm256_fmadd_ps(va, vb[col], vc[idx]);
332:   };
333:   for (int k = 0, kb = 0; k < K; ++k) {
334:     if (is_block_start(k, BLOCK_K)) {
335:         c10::ForcedUnroll<COLS>{}(load_scale_and_zeros, kb++);
336:     }
337:     c10::ForcedUnroll<ROWS * COLS>{}(compute, k);
338:   }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 340-374
```cpp
340:   // store to C
341:   auto storec = [&](auto i) {
342:     constexpr int row = i / COLS;
343:     constexpr int col = i % COLS;
344:     if constexpr (col % 2 == 0) {
345:       __m256i ci = vec::cvtfp32_bf16(vc[row * COLS + col], vc[row * COLS + col + 1]);
346:       _mm256_storeu_si256((__m256i*)(C + row * ldc + col * 8), ci);
347:     }
348:   };
349:   c10::ForcedUnroll<ROWS * COLS>{}(storec);
350: }
351:
352: #endif
353:
354: #if !defined(C10_MOBILE) && defined(__aarch64__)
355: #include <arm_neon.h>
356:
357: inline float32x4x2_t load_as_float32x4x2(const Half* ptr) {
358:   float16x4x2_t f16_val = vld2_f16(reinterpret_cast<const float16_t *>(ptr));
359:   auto val_low = vcvt_f32_f16(f16_val.val[0]);
360:   auto val_high = vcvt_f32_f16(f16_val.val[1]);
361:   return {val_low, val_high};
362: }
363:
364: inline void store_float32x4(Half* ptr, float32x4_t val) {
365:     vst1_f16(reinterpret_cast<float16_t*>(ptr), vcvt_f16_f32(val));
366: }
367:
368: inline float32x4x2_t load_as_float32x4x2(const BFloat16* ptr) {
369:   int32x4_t shift = vdupq_n_s32(16);
370:   uint16x4x2_t u16_val = vld2_u16(reinterpret_cast<const uint16_t *>(ptr));
371:   uint32x4_t int_low = vmovl_u16(u16_val.val[0]);
372:   uint32x4_t int_high = vmovl_u16(u16_val.val[1]);
373:   return {vreinterpretq_f32_u32(vshlq_u32(int_low, shift)), vreinterpretq_f32_u32(vshlq_u32(int_high, shift))};
374: }
```
- EN: This range pulls in required headers, including `arm_neon.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `constexpr`, `store_float32x4`, which contributes directly to this file's operator logic.
- CN: 这一段引入了所需头文件，例如 `arm_neon.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `constexpr`, `store_float32x4`，它们直接构成本文件的算子逻辑。

### Lines 376-404
```cpp
376: inline void store_float32x4(BFloat16* ptr, float32x4_t val) {
377:     int32x4_t shift = vdupq_n_s32(-16);
378:     uint32x4_t uint32_val = vshlq_u32(vreinterpretq_u32_f32(val), shift);
379:     vst1_u16(reinterpret_cast<uint16_t*>(ptr), vmovn_u32(uint32_val));
380: }
381:
382: inline float32x4x2_t load_as_float32x4x2(const float* ptr) {
383:   return vld2q_f32(ptr);
384: }
385:
386: inline void store_float32x4(float* ptr, float32x4_t val) {
387:     vst1q_f32(ptr, val);
388: }
389:
390: template <int BLOCK_M, int BLOCK_N, typename T>
391: inline void tinygemm_kernel_(
392:     const T* RESTRICT A,
393:     const uint8_t* RESTRICT B,
394:     const T* RESTRICT ScaleAndZeros,
395:     T* RESTRICT C,
396:     int lda,
397:     int ldb,
398:     int ldc,
399:     int K,
400:     int BLOCK_K) {
401:   int16_t shift_vals[4] = {0, -4, -8, -12};
402:   int16x4_t shifts = vld1_s16(shift_vals);
403:   int16x4_t offs = vdup_n_s16(8);
404:   uint16x4_t mask = vdup_n_u16(0x0F);
```
- EN: The main symbol in this range is `store_float32x4`, `load_as_float32x4x2`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `store_float32x4`, `load_as_float32x4x2`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 405-436
```cpp
405:   for (const auto m : c10::irange(BLOCK_M)) {
406:     for (int n = 0; n < BLOCK_N; n+= 16) {
407:       float32x4_t c_val[4];
408:       float32x4_t scales[4], zeros[4];
409:       c10::ForcedUnroll<4>{}([&](auto i) {
410:           c_val[i] = vdupq_n_f32(0.0);
411:       });
412:       for (const auto k : c10::irange(K)) {
413:         const auto a_val = vdupq_n_f32(static_cast<float>(A[m * lda + k]));
414:         if (is_block_start(k, BLOCK_K)) {
415:           int kb = k / BLOCK_K;
416:           c10::ForcedUnroll<4>{}([&](auto i) {
417:             auto scales_and_zeros = load_as_float32x4x2(ScaleAndZeros + kb * ldc * 2 + n * 2 + i * 8);
418:             scales[i] = scales_and_zeros.val[0];
419:             zeros[i] = scales_and_zeros.val[1];
420:           });
421:         }
422:         c10::ForcedUnroll<4>{}([&](auto i) {
423:           uint16_t b_pack = reinterpret_cast<const uint16_t*>(B + k * ldb + n / 2)[i];
424:           uint16x4_t b_masked = vand_u16(vshl_u16(vdup_n_u16(b_pack), shifts), mask);
425:           int16x4_t b_ints = vsub_s16(vreinterpret_s16_u16(b_masked), offs);
426:           float32x4_t b_vals = vcvtq_f32_s32(vmovl_s16(b_ints));
427:           b_vals = vaddq_f32(zeros[i], vmulq_f32(scales[i], b_vals));
428:           c_val[i] = vfmaq_f32(c_val[i], b_vals, a_val);
429:         });
430:       }
431:       c10::ForcedUnroll<4>{}([&](auto i) {
432:         store_float32x4(C + m * ldc + n + i * 4, c_val[i]);
433:       });
434:     }
435:   }
436: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 438-479
```cpp
438: template <int BLOCK_M, int BLOCK_N>
439: inline void tinygemm_kernel(
440:     const Half* RESTRICT A,
441:     const uint8_t* RESTRICT B,
442:     const Half* RESTRICT ScaleAndZeros,
443:     Half* RESTRICT C,
444:     int lda,
445:     int ldb,
446:     int ldc,
447:     int K,
448:     int BLOCK_K) {
449:   tinygemm_kernel_<BLOCK_M, BLOCK_N>(A, B, ScaleAndZeros, C, lda, ldb, ldc, K, BLOCK_K);
450: }
451:
452: template <int BLOCK_M, int BLOCK_N>
453: inline void tinygemm_kernel(
454:     const BFloat16* RESTRICT A,
455:     const uint8_t* RESTRICT B,
456:     const BFloat16* RESTRICT ScaleAndZeros,
457:     BFloat16* RESTRICT C,
458:     int lda,
459:     int ldb,
460:     int ldc,
461:     int K,
462:     int BLOCK_K) {
463:   tinygemm_kernel_<BLOCK_M, BLOCK_N>(A, B, ScaleAndZeros, C, lda, ldb, ldc, K, BLOCK_K);
464: }
465:
466: template <int BLOCK_M, int BLOCK_N>
467: inline void tinygemm_kernel(
468:     const float* RESTRICT A,
469:     const uint8_t* RESTRICT B,
470:     const float* RESTRICT ScaleAndZeros,
471:     float* RESTRICT C,
472:     int lda,
473:     int ldb,
474:     int ldc,
475:     int K,
476:     int BLOCK_K) {
477:   tinygemm_kernel_<BLOCK_M, BLOCK_N>(A, B, ScaleAndZeros, C, lda, ldb, ldc, K, BLOCK_K);
478: }
479: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `tinygemm_kernel`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `tinygemm_kernel`，它们直接构成本文件的算子逻辑。

### Lines 481-521
```cpp
481: template<int BLOCK_N>
482: inline float convert_int4_to_float(const uint8_t* b, int n) {
483:   static constexpr float lut[16] = {
484:     -8.0f, -7.0f, -6.0f, -5.0f,
485:     -4.0f, -3.0f, -2.0f, -1.0f,
486:     0.0f, 1.0f, 2.0f, 3.0f,
487:     4.0f, 5.0f, 6.0f, 7.0f
488:   };
489:   int index;
490: #if defined(CPU_CAPABILITY_AVX512) && !defined(_MSC_VER)
491:   if constexpr (BLOCK_N == 64) {
492:     const int nb = n/BLOCK_N;
493:     n -= nb*BLOCK_N;
494:     if (n < 32) {
495:       auto val = b[nb * BLOCK_N / 2 + n];
496:       index = val & 0x0f;
497:     } else {
498:       auto val = b[nb * BLOCK_N / 2 + (n - 32)];
499:       index = val >> 4;
500:     }
501:   } else
502: #elif defined(CPU_CAPABILITY_AVX2) && !defined(_MSC_VER)
503:   if constexpr (BLOCK_N == 32) {
504:     const int nb = n/BLOCK_N;
505:     n -= nb*BLOCK_N;
506:     if (n < 16) {
507:       auto val = b[nb * BLOCK_N / 2 + n];
508:       index = val & 0x0f;
509:     } else {
510:       auto val = b[nb * BLOCK_N / 2 + (n - 16)];
511:       index = val >> 4;
512:     }
513:   } else
514: #endif
515:   {
516:     const auto is_even = (n & 1) == 0;
517:     auto val = b[n/2];
518:     index = is_even ? (val & 0x0F) : (val >> 4);
519:   }
520:   return lut[index];
521: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `convert_int4_to_float`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `convert_int4_to_float`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 523-552
```cpp
523: // non-vectorized version
524: template <int BLOCK_M, int BLOCK_N, typename T>
525: inline void tinygemm_kernel(
526:     const T* RESTRICT A,
527:     const uint8_t* RESTRICT B,
528:     const T* RESTRICT ScaleAndZeros,
529:     T* RESTRICT C,
530:     int lda,
531:     int ldb,
532:     int ldc,
533:     int K,
534:     int BLOCK_K) {
535:
536:   for (const auto m : c10::irange(BLOCK_M)) {
537:     for (const auto n : c10::irange(BLOCK_N)) {
538:       float c_val = 0;
539:       for (const auto k : c10::irange(K)) {
540:         int kb = k / BLOCK_K;
541:         const auto scale = static_cast<float>(ScaleAndZeros[kb * ldc * 2 + n * 2]);
542:         const auto zero = static_cast<float>(ScaleAndZeros[kb * ldc * 2 + n * 2 + 1]);
543:         const auto a_val = static_cast<float>(A[m * lda + k]);
544:         float b_val = convert_int4_to_float<BLOCK_N>(B + k *ldb, n);
545:         b_val = b_val * scale + zero;
546:
547:         c_val += a_val * b_val;
548:       }
549:       C[m * ldc + n] = c_val;
550:     }
551:   }
552: }
```
- EN: The main symbol in this range is `tinygemm_kernel`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `tinygemm_kernel`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 555-582
```cpp
555: #define LAUNCH_TINYGEMM_KERNEL(MB_SIZE, NB_SIZE)                 \
556:   tinygemm_kernel<MB_SIZE, NB_SIZE>(                             \
557:       A_ptr, B_ptr, S_ptr, C_ptr,                                \
558:       K, NB_SIZE / 2, N, K, BLOCK_K);
559:
560: #define LAUNCH_TINYGEMM_NB_SIZE(MB_SIZE)                         \
561:   switch (nb_size) {                                             \
562:     case 16:                                                     \
563:       LAUNCH_TINYGEMM_KERNEL(MB_SIZE, 16);                       \
564:       break;                                                     \
565:     case 32:                                                     \
566:       LAUNCH_TINYGEMM_KERNEL(MB_SIZE, 32);                       \
567:       break;                                                     \
568:     case 48:                                                     \
569:       LAUNCH_TINYGEMM_KERNEL(MB_SIZE, 48);                       \
570:       break;                                                     \
571:     case 64:                                                     \
572:       LAUNCH_TINYGEMM_KERNEL(MB_SIZE, 64);                       \
573:       break;                                                     \
574:     default:                                                     \
575:       TORCH_CHECK(false, "Unsupported n block size: ", nb_size); \
576:       break;                                                     \
577:   }
578:
579: // NB: int4 weight pack (with BLOCK_N 64)
580: //   weight (int32): {N/64, 64, K}
581: //   packed (uint8): {N/64, K, 32}
582: //
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 583-619
```cpp
583: // 1. avx512 packed format:
584: //   When N is 64, to do 256-bit unpacking at a time, we pack Lane0 with Lane2,
585: //   Lane1 with Lane3 since we can only do shift on a 128-bit basis.
586: //
587: //   weight:
588: //     [Lane0] N0...15:  {a00, a01, a02, ...}
589: //     [Lane1] N16...31: {a10, a11, a12, ...}
590: //     [Lane2] N32...47: {a20, a21, a22, ...}
591: //     [Lane3] N48...63: {a30, a31, a32, ...}
592: //
593: //  packed:
594: //     [Lane02] N0...31:  {a20|a00, a21|a01, a22|a02, ...}
595: //     [Lane13] N32...63: {a30|a10, a31|a11, a32|a12, ...}
596: //
597: //  Note: when N is 16, 32 or 48, pack with 64-bit format.
598: //
599: // 2. avx2 packed format:
600: //   When N is 32, to do 128-bit unpacking at a time.
601: //
602: //   weight:
603: //     [Lane0] N0...15:  { a0,  a1,  a2, ...}
604: //     [Lane1] N16...32: {a16, a17, a18, ...}
605: //
606: //  packed:
607: //    [Lane01] N0...32: {a16|a0, a17|a1, a18|a2, ...}
608: //
609: //  Note: When N is 16, pack with 64-bit format
610: //
611: // 3 non-vectorized packed format:
612: //   Do 64-bit unpacking at a time.
613: //
614: //   weight: {a0, a1, a2, a3, ..., a14, a15}
615: //   packed: {a1|a0, a3, a2, ..., a15|a14}
616: //
617: void weight_to_int4pack_kernel(
618:     const Tensor& weight_packed,
619:     const Tensor& weight) {
```
- EN: The main symbol in this range is `weight_to_int4pack_kernel`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `weight_to_int4pack_kernel`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 621-657
```cpp
621:   auto weight_packed_data = reinterpret_cast<uint8_t*>(weight_packed.data_ptr());
622:   const auto weight_data = weight.const_data_ptr<int32_t>();
623:
624:   int N = weight.size(0);
625:   int K = weight.size(1);
626:
627:   // 64 for avx512 and 32 for avx2/non-vectorized
628:   constexpr int BLOCK_N = vec::Vectorized<float>::size() * 4;
629:   const int NB =  (N + BLOCK_N - 1) / BLOCK_N;
630:
631:   // parallel on NB blocks
632:   at::parallel_for(0, NB, 0, [&](int begin, int end) {
633:     for (const auto i : c10::irange(begin, end)) {
634:       int nb_size = std::min(BLOCK_N, N - i * BLOCK_N);
635:
636:       const int32_t* src = weight_data + i * BLOCK_N * K;
637:       uint8_t* dst = weight_packed_data + i * K * BLOCK_N / 2;
638:       for (const auto k : c10::irange(K)) {
639: #if defined(CPU_CAPABILITY_AVX512) && !defined(_MSC_VER)
640:         if (nb_size == BLOCK_N) {
641:           for (const auto d : c10::irange(16)) {
642:             int32_t val0 = src[(d +  0) * K + k];
643:             int32_t val1 = src[(d + 16) * K + k];
644:             int32_t val2 = src[(d + 32) * K + k];
645:             int32_t val3 = src[(d + 48) * K + k];
646:
647:             uint8_t packed02 = ((uint8_t)val2 << 4) | ((uint8_t)val0);
648:             uint8_t packed13 = ((uint8_t)val3 << 4) | ((uint8_t)val1);
649:
650:             dst[k * 32 + d] = packed02;
651:             dst[k * 32 + 16 + d] = packed13;
652:           }
653:         } else {
654:           // for nb_size 16, 32, 48
655:           for (int n = 0; n < nb_size; n += 2) {
656:             int32_t val0 = src[n * K + k];
657:             int32_t val1 = src[n * K + K + k];
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 659-695
```cpp
659:             uint8_t packed = ((uint8_t)val1 << 4) | ((uint8_t)val0);
660:             dst[k * nb_size / 2 + n / 2] = packed;
661:           }
662:         }
663: #elif defined(CPU_CAPABILITY_AVX2) && !defined(_MSC_VER)
664:         if (nb_size == BLOCK_N) {
665:           // for nb_size 32
666:           for (const auto d : c10::irange(16)) {
667:             int32_t val0 = src[(d + 0) * K + k];
668:             int32_t val1 = src[(d + 16) * K + k];
669:
670:             uint8_t packed01 = ((uint8_t)val1 << 4) | ((uint8_t)val0);
671:             dst[k * 16 + d] = packed01;
672:           }
673:         } else {
674:           // for nb_size 16
675:           for (int n = 0; n < nb_size; n += 2) {
676:             int32_t val0 = src[n * K + k];
677:             int32_t val1 = src[n * K + K + k];
678:
679:             uint8_t packed = ((uint8_t)val1 << 4) | ((uint8_t)val0);
680:             dst[k * nb_size / 2 + n / 2] = packed;
681:           }
682:         }
683: #else
684:         for (int n = 0; n < nb_size; n += 2) {
685:           int32_t val0 = src[n * K + k];
686:           int32_t val1 = src[n * K + K + k];
687:
688:           uint8_t packed = ((uint8_t)val1 << 4) | ((uint8_t)val0);
689:           dst[k * nb_size / 2 + n / 2] = packed;
690:         }
691: #endif
692:       }
693:     }
694:   });
695: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 697-725
```cpp
697: template<typename T>
698: void int4pack_mm_kernel_(
699:     const Tensor& C,
700:     const Tensor& A,
701:     const Tensor& B,
702:     int qGroupSize,
703:     const Tensor& qScaleAndZeros) {
704:
705:   const auto* A_data = A.const_data_ptr<T>();
706:   const auto* B_data = reinterpret_cast<const uint8_t*>(B.const_data_ptr());
707:   auto* C_data = C.data_ptr<T>();
708:   const auto* S_data = qScaleAndZeros.const_data_ptr<T>();
709:
710:   int M = A.size(0);
711:   int N = B.size(0);
712:   int K = A.size(1);
713:
714:   constexpr int BLOCK_M = 4;
715:   // 64 for avx512 and 32 for avx2/non-vectorized
716:   constexpr int BLOCK_N = vec::Vectorized<float>::size() * 4;
717:   // 32, 64, 128, 256
718:   const int BLOCK_K = qGroupSize;
719:
720:   const int MB = (M + BLOCK_M - 1) / BLOCK_M;
721:   const int NB = (N + BLOCK_N - 1) / BLOCK_N;
722:
723:   at::parallel_for(0, MB * NB, 0, [&](int begin, int end) {
724:     int mb{0}, nb{0};
725:     data_index_init(begin, mb, MB, nb, NB);
```
- EN: The main symbol in this range is `int4pack_mm_kernel_`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `int4pack_mm_kernel_`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 727-759
```cpp
727:     for ([[maybe_unused]] const auto i : c10::irange(begin, end)) {
728:       int mb_start = mb * BLOCK_M;
729:       int mb_size = std::min(BLOCK_M, M - mb_start);
730:       int nb_start = nb * BLOCK_N;
731:       int nb_size = std::min(BLOCK_N, N - nb_start);
732:
733:       const auto* A_ptr = A_data + mb_start * K;
734:       const auto* B_ptr = B_data + nb_start * K / 2;
735:       const auto* S_ptr = S_data + nb_start * 2;
736:       auto* C_ptr = C_data + mb_start * N + nb_start;
737:
738:       switch (mb_size) {
739:         case 1:
740:           LAUNCH_TINYGEMM_NB_SIZE(1);
741:           break;
742:         case 2:
743:           LAUNCH_TINYGEMM_NB_SIZE(2);
744:           break;
745:         case 3:
746:           LAUNCH_TINYGEMM_NB_SIZE(3);
747:           break;
748:         case 4:
749:           LAUNCH_TINYGEMM_NB_SIZE(4);
750:           break;
751:         default:
752:           TORCH_CHECK(false, "Unsupported m block size: ", mb_size);
753:       }
754:
755:       // move to the next index
756:       data_index_step(mb, MB, nb, NB);
757:     }
758:   });
759: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 761-794
```cpp
761: void int4pack_mm_kernel(
762:     const Tensor& C,
763:     const Tensor& A,
764:     const Tensor& B,
765:     int qGroupSize,
766:     const Tensor& qScaleAndZeros) {
767:   if (C.scalar_type() == kBFloat16) {
768:     int4pack_mm_kernel_<BFloat16>(C, A, B, qGroupSize, qScaleAndZeros);
769:   } else if (C.scalar_type() == kHalf) {
770:     int4pack_mm_kernel_<Half>(C, A, B, qGroupSize, qScaleAndZeros);
771:   } else {
772:     int4pack_mm_kernel_<float>(C, A, B, qGroupSize, qScaleAndZeros);
773:   }
774: }
775:
776: #if AT_KLEIDIAI_ENABLED()
777: bool can_use_kleidiai(
778:     const at::Tensor& scales_zeros,
779:     const int64_t K,
780:     const int64_t block_size) {
781:   bool ret = false;
782:   if (cpuinfo_has_arm_neon_dot()) {
783:     // The Groupwise kernel requires BFloat16 Scales and Channelwise kernel
784:     // requires Float32 Scales. If not provided, we will use fallback
785:     // implementation.
786:     if ((block_size == K && scales_zeros.dtype() == at::kFloat) ||
787:         ((block_size < K && !(block_size % 32) && !(K % block_size)) &&
788:          scales_zeros.dtype() == at::kBFloat16)) {
789:       ret = true;
790:     }
791:   }
792:   return ret;
793: }
794: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `int4pack_mm_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `int4pack_mm_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 796-829
```cpp
796: static void ref_dyn_quant_matmul_4bit_channelwise_kernel_bf16(
797:     size_t m,
798:     size_t n,
799:     size_t k,
800:     const uint16_t* lhs_bf16,
801:     const uint8_t* rhs_qs4cx,
802:     const float* rhs_scales,
803:     uint16_t* dst_bf16,
804:     float scalar_min,
805:     float scalar_max,
806:     const float* bias) {
807:   // Roundup lambda for internal stride calculations
808:   auto roundup = [](size_t a, size_t b) { return ((a + b - 1) / b) * b; };
809:
810:   // Cast bfloat16 to float32 inline
811:   auto cast_bf16_to_f32 = [](uint16_t bf16_val) {
812:     uint32_t tmp = static_cast<uint32_t>(bf16_val) << 16;
813:     float f;
814:     std::memcpy(&f, &tmp, sizeof(f));
815:     return f;
816:   };
817:
818:   // Cast float32 to bfloat16 inline
819:   auto cast_f32_to_bf16 = [](float f) {
820:     uint32_t bits;
821:     std::memcpy(&bits, &f, sizeof(bits));
822:     return static_cast<uint16_t>(bits >> 16);
823:   };
824:
825:   // Quantization pack lambda (channelwise QA8DX)
826:   auto quant_pack_8bit_channelwise =
827:       [&](size_t M, size_t K, const uint16_t* src_bf16, int8_t* dst_qa8dx) {
828:         constexpr int8_t kI8Min = std::numeric_limits<std::int8_t>::lowest();
829:         constexpr int8_t kI8Max = std::numeric_limits<std::int8_t>::max();
```
- EN: The main symbol in this range is `ref_dyn_quant_matmul_4bit_channelwise_kernel_bf16`, `lambda`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `ref_dyn_quant_matmul_4bit_channelwise_kernel_bf16`, `lambda`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 831-871
```cpp
831:         const size_t dst_stride =
832:             K * sizeof(int8_t) + sizeof(float) + sizeof(int32_t);
833:         for (size_t i = 0; i < M; ++i) {
834:           const uint16_t* row_ptr = src_bf16 + i * K;
835:           // find min/max
836:           float mn = FLT_MAX, mx = -FLT_MAX;
837:           for (size_t j = 0; j < K; ++j) {
838:             float v = cast_bf16_to_f32(row_ptr[j]);
839:             mn = std::min(mn, v);
840:             mx = std::max(mx, v);
841:           }
842:           float rmin = std::min(0.0f, mn);
843:           float rmax = std::max(0.0f, mx);
844:           constexpr float qmin = static_cast<float>(kI8Min);
845:           constexpr float qmax = static_cast<float>(kI8Max);
846:           float scale = (rmin == rmax) ? 1.f : (qmax - qmin) / (rmax - rmin);
847:           float recip = scale ? 1.0f / scale : 0.0f;
848:           int32_t zp;
849:           float des_min = rmin * scale;
850:           float des_max = rmax * scale;
851:           float err_min = qmin + des_min;
852:           float err_max = qmax + des_max;
853:           float zp_f =
854:               (err_min + err_max) > 0 ? qmin - des_min : qmax - des_max;
855:           zp_f = std::clamp(zp_f, qmin, qmax);
856:           zp = std::lrintf(zp_f);
857:           int8_t* out_ptr = dst_qa8dx + i * dst_stride;
858:           // store header
859:           *reinterpret_cast<float*>(out_ptr) = recip;
860:           *reinterpret_cast<int32_t*>(out_ptr + sizeof(float)) = -zp;
861:           out_ptr += sizeof(float) + sizeof(int32_t);
862:           // quantize
863:           for (size_t j = 0; j < K; ++j) {
864:             float v = cast_bf16_to_f32(row_ptr[j]);
865:             int32_t q = static_cast<int32_t>(std::round(v * scale)) + zp;
866:             q = std::clamp(
867:                 q, static_cast<int32_t>(kI8Min), static_cast<int32_t>(kI8Max));
868:             *out_ptr++ = static_cast<int8_t>(q);
869:           }
870:         }
871:       };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 873-911
```cpp
873:   // MatMul lambda (MXN x MXK -> MNXK BF16)
874:   auto matmul_kernel = [&](size_t M,
875:                            size_t N,
876:                            size_t K,
877:                            const int8_t* lhs,
878:                            const uint8_t* rhs,
879:                            const float* scales,
880:                            uint16_t* dst,
881:                            float lo,
882:                            float hi) {
883:     const size_t lhs_stride =
884:         K * sizeof(int8_t) + sizeof(float) + sizeof(int32_t);
885:     const size_t rhs_stride = roundup(K, 2) / 2;
886:     for (size_t i = 0; i < M; ++i) {
887:       const int8_t* lhs_row = lhs + i * lhs_stride;
888:       for (size_t j = 0; j < N; ++j) {
889:         int32_t acc = 0;
890:         const int8_t* lptr = lhs_row;
891:         const uint8_t* rptr = rhs + j * rhs_stride;
892:         float lhs_scale = *reinterpret_cast<const float*>(lptr);
893:         int32_t lhs_off =
894:             *reinterpret_cast<const int32_t*>(lptr + sizeof(float));
895:         lptr += sizeof(float) + sizeof(int32_t);
896:         for (size_t t = 0; t < K; ++t) {
897:           int32_t lv = static_cast<int32_t>(lptr[t]);
898:           uint8_t bv = rptr[t / 2];
899:           int32_t rv = ((t & 1) == 0) ? (static_cast<int32_t>(bv & 0xF) - 8)
900:                                       : (static_cast<int32_t>(bv >> 4) - 8);
901:           acc += lv * rv + lhs_off * rv;
902:         }
903:         float res = static_cast<float>(acc) * scales[j] * lhs_scale;
904:         if (bias) {
905:           res += bias[j];
906:         }
907:         res = std::clamp(res, lo, hi);
908:         *dst++ = cast_f32_to_bf16(res);
909:       }
910:     }
911:   };
```
- EN: The main symbol in this range is `lambda`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `lambda`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 913-941
```cpp
913:   // allocate and run
914:   std::unique_ptr<int8_t[]> packed(
915:       new int8_t[m * (k * sizeof(int8_t) + sizeof(float) + sizeof(int32_t))]);
916:   quant_pack_8bit_channelwise(m, k, lhs_bf16, packed.get());
917:   matmul_kernel(
918:       m,
919:       n,
920:       k,
921:       packed.get(),
922:       rhs_qs4cx,
923:       rhs_scales,
924:       dst_bf16,
925:       scalar_min,
926:       scalar_max);
927: }
928:
929: /**
930:  * The Int4 quantized weights must be represented as a uint8 tensor
931:  * For matrix multiplication with a weight shape of (N x K)
932:  * the shape of the 4-bit quantized weights is [N, K/groupsize, groupsize/2].
933:  *
934:  * For KleidiAI weight packing, the scales, biases, and Int4 quantized
935:  * weights are packed into a single `packed_weights` structure, optimized for
936:  * Arm instructions.
937:  *
938:  * In the fallback reference kernel, no special packing is required for
939:  * Int4 quantized weights.
940:  *
941:  * The Groupwise kernel requires BFloat16 Scales and Channelwise kernel requires
```
- EN: The main symbol in this range is `matmul_kernel`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `matmul_kernel`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 942-972
```cpp
942:  * Float32 Scales. If not provided, we will use fallback implementation.
943:  */
944: void dyn_quant_pack_4bit_weight_kernel(
945:     Tensor& packed_weights,
946:     const Tensor& weights,
947:     const Tensor& scales_zeros,
948:     const std::optional<Tensor>& bias,
949:     const int64_t N,
950:     const int64_t K,
951:     const int64_t block_size) {
952: #if AT_KLEIDIAI_ENABLED()
953:   if (can_use_kleidiai(scales_zeros, K, block_size)) {
954:     const int64_t weight_packed_size =
955:         kleidiai::kai_pack_rhs_int4_size(N, K, block_size, weights.scalar_type());
956:     packed_weights.resize_({weight_packed_size});
957:     kleidiai::kai_pack_int4_rhs(
958:         packed_weights, weights, scales_zeros, bias, N, K, block_size);
959:   } else
960: #endif
961:   {
962:     packed_weights = packed_weights.to(kFloat);
963:     auto weight_reshaped = weights.reshape({-1}).to(kFloat);
964:     auto scales_zeros_reshaped = scales_zeros.reshape({-1}).to(kFloat);
965:     std::vector<at::Tensor> tensors_to_cat = {weight_reshaped, scales_zeros_reshaped};
966:     if (bias.has_value()) {
967:       tensors_to_cat.push_back(bias.value().view({-1}).to(kFloat));
968:     }
969:     auto res = at::cat(tensors_to_cat, 0);
970:     packed_weights.resize_(res.sizes()).copy_(res);
971:   }
972: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `dyn_quant_pack_4bit_weight_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `dyn_quant_pack_4bit_weight_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 974-1003
```cpp
 974: void ref_dyn_quant_matmul_4bit_channelwise_kernel(
 975:     size_t m,
 976:     size_t n,
 977:     size_t k,
 978:     const float* lhs_f32,
 979:     const uint8_t* rhs_qs4cx,
 980:     const float* rhs_scales_f32,
 981:     float* dst_f32,
 982:     float scalar_min,
 983:     float scalar_max,
 984:     const float* bias) {
 985:   const size_t input_size_8bit = m * (k + sizeof(int32_t) + sizeof(float));
 986:
 987:   auto lhs_qa8dx_buffer = std::make_unique<uint8_t[]>(input_size_8bit);
 988:   uint8_t* lhs_qa8dx = lhs_qa8dx_buffer.get();
 989:
 990:   // Lambda for quantizing the fp32 input to 8 bit symmetric and pack it in
 991:   // required format for matmul
 992:   auto input_quant_pack_8bit_channelwise =
 993:       [&](size_t m, size_t k, const float* lhs_f32, int8_t* lhs_qa8dx) {
 994:         constexpr int8_t kI8Min = std::numeric_limits<std::int8_t>::lowest();
 995:         constexpr int8_t kI8Max = std::numeric_limits<std::int8_t>::max();
 996:
 997:         const size_t dst_stride =
 998:             (k * sizeof(int8_t) + sizeof(float) + sizeof(int32_t));
 999:
1000:         const size_t lhs_qa8dx_stride = k;
1001:
1002:         for (size_t m_idx = 0; m_idx < m; ++m_idx) {
1003:           const float* src_ptr = lhs_f32 + m_idx * lhs_qa8dx_stride;
```
- EN: The main symbol in this range is `ref_dyn_quant_matmul_4bit_channelwise_kernel`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `ref_dyn_quant_matmul_4bit_channelwise_kernel`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1005-1033
```cpp
1005:           float max0 = -FLT_MAX;
1006:           float min0 = FLT_MAX;
1007:
1008:           // Find min/max for each channel
1009:           for (size_t k_idx = 0; k_idx < k; ++k_idx) {
1010:             const float src0_0 = src_ptr[k_idx];
1011:
1012:             max0 = std::max(src0_0, max0);
1013:             min0 = std::min(src0_0, min0);
1014:           }
1015:
1016:           // Maximum/minimum int8 values
1017:           constexpr float qmin = static_cast<float>(kI8Min);
1018:           constexpr float qmax = static_cast<float>(kI8Max);
1019:
1020:           const float rmin0 = std::min(0.0f, min0);
1021:           const float rmax0 = std::max(0.0f, max0);
1022:
1023:           const float scale0 =
1024:               rmin0 == rmax0 ? 1.f : (qmax - qmin) / (rmax0 - rmin0);
1025:
1026:           // Reciprocal to quantize
1027:           const float recip_scale0 = scale0 ? 1.0f / scale0 : 0.0f;
1028:
1029:           const float descaled_min0 = rmin0 * scale0;
1030:           const float descaled_max0 = rmax0 * scale0;
1031:
1032:           const float zero_point_from_min_error0 = qmin + descaled_min0;
1033:           const float zero_point_from_max_error0 = qmax + descaled_max0;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1035-1068
```cpp
1035:           float zero_point0 =
1036:               zero_point_from_min_error0 + zero_point_from_max_error0 > 0
1037:               ? qmin - descaled_min0
1038:               : qmax - descaled_max0;
1039:
1040:           zero_point0 = std::max(zero_point0, qmin);
1041:           zero_point0 = std::min(zero_point0, qmax);
1042:
1043:           // Round to nearest integer
1044:           const int32_t nudged_zero_point0 = std::lrintf(zero_point0);
1045:
1046:           int8_t* dst_ptr = lhs_qa8dx + m_idx * dst_stride;
1047:
1048:           // LHS offset at the beginning of the row
1049:           *((float*)dst_ptr) = recip_scale0;
1050:           dst_ptr += sizeof(float);
1051:           *((int32_t*)dst_ptr) = -nudged_zero_point0;
1052:           dst_ptr += sizeof(int32_t);
1053:
1054:           // Quantize the channels
1055:           for (size_t k_idx = 0; k_idx < k; ++k_idx) {
1056:             const float src0_0 = src_ptr[k_idx];
1057:
1058:             // Scale the values
1059:             int32_t v0_s32 = (int32_t)(std::round(src0_0 * scale0));
1060:
1061:             v0_s32 = v0_s32 + nudged_zero_point0;
1062:             v0_s32 = std::max(v0_s32, static_cast<int32_t>(kI8Min));
1063:             v0_s32 = std::min(v0_s32, static_cast<int32_t>(kI8Max));
1064:             dst_ptr[0] = (int8_t)v0_s32;
1065:             dst_ptr += sizeof(int8_t);
1066:           }
1067:         }
1068:       };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1070-1098
```cpp
1070:   // Dynamically Quantize the float32 input to 8 bit asymmetric
1071:   input_quant_pack_8bit_channelwise(m, k, lhs_f32, (int8_t*)lhs_qa8dx);
1072:
1073:   const size_t lhs_stride =
1074:       k * sizeof(int8_t) + sizeof(float) + sizeof(int32_t);
1075:
1076:   const size_t rhs_qs4cx_stride = ((((k + 2 - 1) / 2) * 2) / 2);
1077:
1078:   for (size_t m_idx = 0; m_idx < m; ++m_idx) {
1079:     const int8_t* lhs_ptr_start = (int8_t*)lhs_qa8dx + m_idx * lhs_stride;
1080:
1081:     for (size_t n_idx = 0; n_idx < n; ++n_idx) {
1082:       // Main f32 accumulator
1083:       int32_t iacc = 0;
1084:
1085:       const int8_t* lhs_ptr = lhs_ptr_start;
1086:       const uint8_t* rhs_ptr = rhs_qs4cx + n_idx * rhs_qs4cx_stride;
1087:
1088:       // Get the LHS quantization parameters stored at the
1089:       // beginning of each row
1090:       const float lhs_scale = *(const float*)lhs_ptr;
1091:       lhs_ptr += sizeof(float);
1092:
1093:       const int32_t lhs_offset = *(const int32_t*)lhs_ptr;
1094:       lhs_ptr += sizeof(int32_t);
1095:
1096:       for (size_t k_idx = 0; k_idx < k; ++k_idx) {
1097:         // Get the LHS values
1098:         const int32_t lhs_v0 = (int32_t)lhs_ptr[0];
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1100-1129
```cpp
1100:         // Get the RHS values
1101:         const uint8_t rhs_byte = rhs_ptr[0];
1102:
1103:         // Unpack the RHS values
1104:         int32_t rhs_v0 = 0;
1105:         if ((k_idx % 2) == 0) {
1106:           rhs_v0 = (((int32_t)(rhs_byte & 0x0F)) - 8);
1107:         } else {
1108:           rhs_v0 = (((int32_t)(rhs_byte >> 4)) - 8);
1109:         }
1110:
1111:         iacc += lhs_v0 * rhs_v0;
1112:         iacc += lhs_offset * rhs_v0;
1113:
1114:         lhs_ptr += 1;
1115:
1116:         // Increment only when k_idx is not a multiple of 2
1117:         rhs_ptr += k_idx % 2;
1118:       }
1119:
1120:       // Get the RHS scale
1121:       const float rhs_scale = rhs_scales_f32[n_idx];
1122:
1123:       float main_acc = iacc * rhs_scale;
1124:
1125:       main_acc = main_acc * lhs_scale;
1126:
1127:       if (bias) {
1128:         main_acc += bias[n_idx];
1129:       }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1131-1159
```cpp
1131:       // Clamp (min-max) operation
1132:       main_acc = std::max(main_acc, scalar_min);
1133:       main_acc = std::min(main_acc, scalar_max);
1134:
1135:       dst_f32[0] = main_acc;
1136:       dst_f32 += 1;
1137:     }
1138:   }
1139: }
1140:
1141: void ref_dyn_quant_matmul_4bit_groupwise_kernel(
1142:     size_t m,
1143:     size_t n,
1144:     size_t k,
1145:     size_t bl,
1146:     const float* lhs_f32,
1147:     const uint8_t* rhs_qs4c32,
1148:     const float* rhs_scales_fp32,
1149:     float* dst_f32,
1150:     float scalar_min,
1151:     float scalar_max,
1152:     const float* bias) {
1153:   // Lambda for LHS quantization
1154:   auto lhs_quant_pack = [&](size_t m,
1155:                             size_t k,
1156:                             const float* lhs_f32,
1157:                             int8_t* lhs_qa8dx) {
1158:     constexpr int8_t kI8Min = std::numeric_limits<std::int8_t>::lowest();
1159:     constexpr int8_t kI8Max = std::numeric_limits<std::int8_t>::max();
```
- EN: The main symbol in this range is `ref_dyn_quant_matmul_4bit_groupwise_kernel`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `ref_dyn_quant_matmul_4bit_groupwise_kernel`，它们直接构成本文件的算子逻辑。

### Lines 1161-1190
```cpp
1161:     const size_t dst_stride =
1162:         (k * sizeof(int8_t) + sizeof(float) + sizeof(int32_t));
1163:
1164:     for (size_t row_idx = 0; row_idx < m; ++row_idx) {
1165:       const float* src_ptr = lhs_f32 + row_idx * k;
1166:
1167:       float max0 = -FLT_MAX;
1168:       float min0 = FLT_MAX;
1169:
1170:       for (size_t k_idx = 0; k_idx < k; ++k_idx) {
1171:         const float src0_0 = src_ptr[k_idx];
1172:         max0 = std::max(src0_0, max0);
1173:         min0 = std::min(src0_0, min0);
1174:       }
1175:
1176:       constexpr float qmin = static_cast<float>(kI8Min);
1177:       constexpr float qmax = static_cast<float>(kI8Max);
1178:
1179:       const float rmin0 = std::min(0.0f, min0);
1180:       const float rmax0 = std::max(0.0f, max0);
1181:       const float scale0 =
1182:           (rmin0 == rmax0) ? 1.f : (qmax - qmin) / (rmax0 - rmin0);
1183:       const float recip_scale0 = scale0 ? 1.0f / scale0 : 0.0f;
1184:
1185:       const float descaled_min0 = rmin0 * scale0;
1186:       const float descaled_max0 = rmax0 * scale0;
1187:
1188:       float zero_point0 = (qmin + descaled_min0 + qmax + descaled_max0 > 0)
1189:           ? qmin - descaled_min0
1190:           : qmax - descaled_max0;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1192-1223
```cpp
1192:       zero_point0 = std::max(zero_point0, qmin);
1193:       zero_point0 = std::min(zero_point0, qmax);
1194:       const int32_t nudged_zero_point0 = std::lrintf(zero_point0);
1195:
1196:       int8_t* dst_ptr = lhs_qa8dx + row_idx * dst_stride;
1197:
1198:       *((float*)dst_ptr) = recip_scale0;
1199:       dst_ptr += sizeof(float);
1200:       *((int32_t*)dst_ptr) = -nudged_zero_point0;
1201:       dst_ptr += sizeof(int32_t);
1202:
1203:       for (size_t k_idx = 0; k_idx < k; ++k_idx) {
1204:         const float src0_0 = src_ptr[k_idx];
1205:         int32_t v0_s32 = (int32_t)(std::round(src0_0 * scale0));
1206:         v0_s32 = std::max(
1207:             std::min(v0_s32 + nudged_zero_point0, static_cast<int32_t>(kI8Max)),
1208:             static_cast<int32_t>(kI8Min));
1209:         dst_ptr[0] = (int8_t)v0_s32;
1210:         dst_ptr += sizeof(int8_t);
1211:       }
1212:     }
1213:   };
1214:
1215:   auto lhs_qa8dx_buffer = std::make_unique<int8_t[]>(
1216:       m * (k + sizeof(float) + sizeof(int32_t))); // Allocate for LHS
1217:   int8_t* lhs_qa8dx = lhs_qa8dx_buffer.get();
1218:   // Quantize and pack LHS
1219:   lhs_quant_pack(m, k, lhs_f32, lhs_qa8dx);
1220:
1221:   const size_t num_blocks_row = (((k + bl - 1) / bl) * bl) / bl;
1222:   const size_t lhs_stride = k + sizeof(float) + sizeof(int32_t);
1223:   const size_t rhs_stride = (((k + 2 - 1) / 2) * 2) / 2;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1225-1255
```cpp
1225:   for (size_t row_idx = 0; row_idx < m; ++row_idx) {
1226:     const int8_t* lhs_ptr_start = lhs_qa8dx + row_idx * lhs_stride;
1227:
1228:     for (size_t col_idx = 0; col_idx < n; ++col_idx) {
1229:       float main_acc = 0.0f;
1230:       const int8_t* lhs_ptr = lhs_ptr_start;
1231:       const uint8_t* rhs_ptr = rhs_qs4c32 + col_idx * rhs_stride;
1232:
1233:       const float lhs_scale = *(const float*)lhs_ptr;
1234:       lhs_ptr += sizeof(float);
1235:       const int32_t lhs_offset = *(const int32_t*)lhs_ptr;
1236:       lhs_ptr += sizeof(int32_t);
1237:
1238:       for (size_t block_idx = 0; block_idx < num_blocks_row; ++block_idx) {
1239:         const float rhs_scale =
1240:             rhs_scales_fp32[block_idx + col_idx * num_blocks_row];
1241:         int32_t iacc = 0;
1242:
1243:         for (size_t i = 0; i < bl; ++i) {
1244:           const size_t k_idx = block_idx * bl + i;
1245:           if (k_idx >= k) {
1246:             break;
1247:           }
1248:
1249:           const int32_t lhs_v0 = (int32_t)lhs_ptr[0];
1250:           const uint8_t rhs_byte = rhs_ptr[0];
1251:           int32_t rhs_v0 = (k_idx % 2 == 0) ? (((int32_t)(rhs_byte & 0x0F)) - 8)
1252:                                             : (((int32_t)(rhs_byte >> 4)) - 8);
1253:
1254:           iacc += lhs_v0 * rhs_v0;
1255:           iacc += lhs_offset * rhs_v0;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1257-1285
```cpp
1257:           lhs_ptr += 1;
1258:           rhs_ptr += (k_idx % 2);
1259:         }
1260:
1261:         main_acc += iacc * rhs_scale;
1262:       }
1263:
1264:       main_acc = main_acc * lhs_scale;
1265:
1266:       if (bias) {
1267:         main_acc += bias[col_idx];
1268:       }
1269:
1270:       main_acc = std::max(main_acc, scalar_min);
1271:       main_acc = std::min(main_acc, scalar_max);
1272:
1273:       dst_f32[0] = main_acc;
1274:       dst_f32 += 1;
1275:     }
1276:   }
1277: }
1278:
1279: /**
1280:  * Dynamic INT4 weight-only MatMul with per-row input quantization.
1281:  *
1282:  * Execution Flow:
1283:  *
1284:  *   (INT4 Weights + FP Scales [+ optional Bias])
1285:  *
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1286-1322
```cpp
1286:  *    Input (FP32 or BF16)         Packed Weight Buffer
1287:  *           |                             |
1288:  *    Row-wise Quantization (INT8)         |
1289:  *           |                             |
1290:  *     INT8 Input Activation      INT4 Quantized Weights + Scales
1291:  *                  \             /
1292:  *                   \           /
1293:  *              Quantized Matrix Multiply
1294:  *                     |
1295:  *              Output Tensor (BF16 or FP32)
1296:  *
1297:  * Notes:
1298:  *   - Groupwise kernels expect BF16 scales
1299:  *   - Channelwise kernels expect FP32 scales
1300:  *   - Bias is currently unsupported in fallback path
1301:  */
1302: void dyn_quant_matmul_4bit_kernel(
1303:     const Tensor& output,
1304:     const Tensor& inp,
1305:     const Tensor& packed_weights,
1306:     const int64_t M,
1307:     const int64_t N,
1308:     const int64_t K,
1309:     const int64_t block_size) {
1310: #if AT_KLEIDIAI_ENABLED()
1311:   const int64_t weight_packed_size =
1312:       kleidiai::kai_pack_rhs_int4_size(N, K, block_size, inp.scalar_type());
1313:   if (weight_packed_size == packed_weights.numel()) {
1314:     // KleidiAI interface internally handles the Channelwise and groupwise
1315:     // distinction
1316:     kleidiai::kai_quant_pack_lhs_int4_mm(output, inp, packed_weights, M, N, K, block_size);
1317:   } else
1318: #endif
1319:   {
1320:     {
1321:     void* input = inp.data_ptr();
1322:     void* dst = output.data_ptr();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `Input`, `dyn_quant_matmul_4bit_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `Input`, `dyn_quant_matmul_4bit_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1324-1352
```cpp
1324:     // Extract weights, sclaes and biases form from packed tensor
1325:     const int weights_elements = N * K / 2;
1326:     const int scale_elements = N * (K / block_size);
1327:     TORCH_CHECK(packed_weights.numel() >=  (weights_elements + scale_elements), "Invalid packed weight tensor size");
1328:
1329:     auto extracted_weights = packed_weights.narrow(0, 0, weights_elements).to(kByte);
1330:     auto extracted_scales_and_bias = packed_weights.narrow(0, weights_elements, packed_weights.size(0) - weights_elements).to(kFloat);
1331:     auto float32_scales = extracted_scales_and_bias.narrow(0, 0, scale_elements);
1332:
1333:     int bias_elements = packed_weights.numel() - (weights_elements + scale_elements);
1334:     float* weight_scales = float32_scales.data_ptr<float>();
1335:
1336:     void* bias_data = nullptr;
1337:     if (bias_elements) {
1338:         auto float32_bias = extracted_scales_and_bias.narrow(0, scale_elements, bias_elements);
1339:         TORCH_CHECK(float32_bias.size(0) == N, "Expected bias length to match output dimension");
1340:         bias_data = float32_bias.data_ptr();
1341:
1342:     }
1343:     // 2 elements of 4 bit weights are packed into 1 uint8 packet
1344:     uint8_t* weights_4bit = reinterpret_cast<uint8_t*>(extracted_weights.data_ptr());
1345:
1346:     // Dispatch to reference kernels
1347:     if (inp.scalar_type() == at::kBFloat16) {
1348:         // BF16 input, BF16 output
1349:         constexpr float BF16_MAX = 3.38953139e+38f;
1350:         constexpr float BF16_MIN = -BF16_MAX;
1351:         if (block_size == K) {
1352:             ref_dyn_quant_matmul_4bit_channelwise_kernel_bf16(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1353-1384
```cpp
1353:                 M, N, K,
1354:                 (uint16_t*)input, weights_4bit, weight_scales,
1355:                 (uint16_t*)dst, BF16_MIN, BF16_MAX, (float*)bias_data);
1356:         } else {
1357:             TORCH_CHECK(false, "Unsupported block size for BF16 fallback");
1358:         }
1359:     } else if (inp.scalar_type() == at::kFloat) {
1360:         // FP32 input, FP32 output
1361:         if (block_size == K) {
1362:             ref_dyn_quant_matmul_4bit_channelwise_kernel(
1363:                 M, N, K,
1364:                 (float*)input, weights_4bit, weight_scales,
1365:                 (float*)dst, -FLT_MAX, FLT_MAX, (float*)bias_data);
1366:         } else if (!(block_size % 32) && !(K % block_size)) {
1367:             ref_dyn_quant_matmul_4bit_groupwise_kernel(
1368:                 M, N, K, block_size,
1369:                 (float*)input, weights_4bit, weight_scales,
1370:                 (float*)dst, -FLT_MAX, FLT_MAX, (float*)bias_data);
1371:         } else {
1372:             TORCH_CHECK(false, "Unsupported block size for FP32 fallback");
1373:         }
1374:     } else {
1375:         TORCH_CHECK(false, "Unsupported input/output dtype combination for int4mm kernel");
1376:     }
1377: }
1378: }
1379: } // anonymous namespace
1380: }
1381: ALSO_REGISTER_AVX512_DISPATCH(weight_to_int4pack_stub, &weight_to_int4pack_kernel)
1382: ALSO_REGISTER_AVX512_DISPATCH(int4pack_mm_stub, &int4pack_mm_kernel)
1383: REGISTER_DISPATCH(dyn_quant_pack_4bit_weight_stub, &dyn_quant_pack_4bit_weight_kernel)
1384: REGISTER_DISPATCH(dyn_quant_matmul_4bit_stub, &dyn_quant_matmul_4bit_kernel)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `ref_dyn_quant_matmul_4bit_channelwise_kernel`, `ref_dyn_quant_matmul_4bit_groupwise_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `ref_dyn_quant_matmul_4bit_channelwise_kernel`, `ref_dyn_quant_matmul_4bit_groupwise_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1386-1386
```cpp
1386: } // at::native
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/cpu/vec/functional.h`, `ATen/cpu/vec/vec.h`, `ATen/native/cpu/int_mm_kernel.h`, `ATen/native/cpu/utils.h`, `ATen/Functions.h`, `ATen/ops/cat.h`, `ATen/native/kleidiai/kai_kernels.h`
- c10 headers / c10 头文件: `c10/util/Unroll.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `type_traits`, `cmath`, `cpuinfo.h`, `arm_neon.h`
- Key helper symbols / 关键辅助符号: `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`
