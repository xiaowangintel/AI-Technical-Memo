# UpSampleKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/UpSampleKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU resampling/interpolation kernels and coordinate mapping logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 重采样/插值 kernel 与坐标映射逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Context.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/Parallel.h>
 6: #include <ATen/TensorIterator.h>
 7: #include <ATen/cpu/vec/vec.h>
 8: #include <ATen/native/UpSample.h>
 9: #include <ATen/native/cpu/utils.h>
10: #include <c10/util/irange.h>
11: #include <ATen/native/cpu/UpSampleKernelAVXAntialias.h>
12: #include <ATen/native/cpu/UpSampleKernelNEONAntialias.h>
13:
14: #ifndef AT_PER_OPERATOR_HEADERS
15: #include <ATen/Functions.h>
16: #else
17: #include <ATen/ops/empty.h>
18: #include <ATen/ops/empty_native.h>
19: #include <ATen/ops/ones.h>
20: #endif
21:
22: namespace at::native {
23: namespace {
24:
25: using scale_t = std::vector<std::optional<double>>;
26:
27: // Naming conventions used in this file:
28: //
29: // - "non_separable": All spatial dimensions are interpolated in a single
30: //   TensorIterator pass, using the recursive InterpolateNonSeparable struct.
31: //   Non-separable is always multi-dimensional (Nd).
32: //
33: // - "separable": The multi-dimensional interpolation is decomposed into a
34: //   sequence of 1d passes (one per spatial dimension). The entry point is Nd
35: //   (it loops over dims), and inner functions are 1d.
36: //
37: //   - "1d": Refers to processing a single spatial dimension within the
38: //     separable approach. Not to be confused with 1d interpolation (e.g.
39: //     linear); it means one dimension of a multi-dimensional separable
40: //     decomposition.
41: //
42: //   - "Nd": Templated on out_ndims (1, 2, or 3) for the separable approach.
43:
44:
45: // ---- Non-separable interpolation ----
46: //
47: // Used by: nearest, linear, bilinear (float), cubic (float), trilinear.
48: // Processes all spatial dims in a single TensorIterator pass via the recursive
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 49-96
```cpp
49: // InterpolateNonSeparable struct.
50: //
51: // Call chain:
52: //   upsample_non_separable_Nd_kernel_impl
53: //     -> upsample_non_separable
54: //       -> basic_loop_non_separable
55: //         -> interpolate_non_separable
56: //           -> InterpolateNonSeparable (recursive struct)
57: //
58: // Helper structs and methods for upsample_non_separable
59: //
60: // Interpolation structure to compute output value in n-dimensional case.
61: // - recursively compute interpolated output for each dimension
62: // - we rely a lot on compiler's code optimization such that implemented operations
63: //   can be automatically factorized and vectorized using SSE and AVX2
64: template <int n, typename scalar_t, typename opmath_t, typename index_t, int interp_size>
65: struct InterpolateNonSeparable {
66:     static inline opmath_t eval(char* src, char** data, const int64_t* strides, int64_t i) {
67:       index_t ids = *(index_t*)&data[0][i * strides[0]];
68:       opmath_t wts = *(scalar_t*)&data[1][i * strides[1]];
69:       opmath_t t = InterpolateNonSeparable<n - 1, scalar_t, opmath_t, index_t, interp_size>::eval(src + ids, &data[2 * interp_size], &strides[2 * interp_size], i);
70:       opmath_t output = t * wts;
71:       for (const auto j : c10::irange(1, interp_size)) {
72:         ids = *(index_t*)&data[2 * j + 0][i * strides[2 * j + 0]];
73:         wts = *(scalar_t*)&data[2 * j + 1][i * strides[2 * j + 1]];
74:         t = InterpolateNonSeparable<n - 1, scalar_t, opmath_t, index_t, interp_size>::eval(src + ids, &data[2 * interp_size], &strides[2 * interp_size], i);
75:         output += t * wts;
76:       }
77:       return output;
78:   }
79: };
80:
81: template <typename scalar_t, typename opmath_t, typename index_t, int interp_size>
82: struct InterpolateNonSeparable<1, scalar_t, opmath_t, index_t, interp_size> {
83:     static inline opmath_t eval(char* src, char** data, const int64_t* strides, int64_t i) {
84:       index_t ids = *(index_t*)&data[0][i * strides[0]];
85:       opmath_t wts = *(scalar_t*)&data[1][i * strides[1]];
86:       opmath_t t = *(scalar_t *)&src[ids];
87:       opmath_t output = t * wts;
88:       for (const auto j : c10::irange(1, interp_size)) {
89:         ids = *(index_t*)&data[2 * j + 0][i * strides[2 * j + 0]];
90:         wts = *(scalar_t*)&data[2 * j + 1][i * strides[2 * j + 1]];
91:         t = *(scalar_t *)&src[ids];
92:         output += t * wts;
93:       }
94:       return output;
95:     }
96: };
```
- EN: The main symbol in this range is `eval`, `InterpolateNonSeparable`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `eval`, `InterpolateNonSeparable`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 98-145
```cpp
 98: template <int n, typename scalar_t, typename opmath_t, typename index_t>
 99: struct InterpolateNonSeparable<n, scalar_t, opmath_t, index_t, 1> {
100:     static inline opmath_t eval(char* src, char** data, const int64_t* strides, int64_t i) {
101:       index_t ids = *(index_t*)&data[0][i * strides[0]];
102:       return InterpolateNonSeparable<n - 1, scalar_t, opmath_t, index_t, 1>::eval(src + ids, &data[2], &strides[2], i);
103:   }
104: };
105:
106: template <typename scalar_t, typename opmath_t, typename index_t>
107: struct InterpolateNonSeparable<1, scalar_t, opmath_t, index_t, 1> {
108:     static inline opmath_t eval(char* src, char** data, const int64_t* strides, int64_t i) {
109:       index_t ids = *(index_t*)&data[0][i * strides[0]];
110:       return *(scalar_t *)&src[ids];
111:     }
112: };
113:
114: // There is an unexpected 2x slowdown for upsample_trilinear3d channels_first
115: // for both 1 and 6 threads. We have to specialize this case as below:
116: // Once the issue is fixed we can keep generic implementation and remove:
117: // struct InterpolateNonSeparable<n, scalar_t, index_t, 2> and
118: // struct InterpolateNonSeparable<1, scalar_t, index_t, 2>
119: template <int n, typename scalar_t, typename opmath_t, typename index_t>
120: struct InterpolateNonSeparable<n, scalar_t, opmath_t, index_t, 2> {
121:     static inline opmath_t eval(char* src, char** data, const int64_t* strides, int64_t i) {
122:         index_t i0 = *(index_t*)&data[0][i * strides[0]];
123:         index_t i1 = *(index_t*)&data[2][i * strides[2]];
124:         opmath_t w0 = *(scalar_t *)&data[1][i * strides[1]];
125:         opmath_t w1 = *(scalar_t *)&data[3][i * strides[3]];
126:
127:         opmath_t t0 = InterpolateNonSeparable<n - 1, scalar_t, opmath_t, index_t, 2>::eval(src + i0, &data[4], &strides[4], i);
128:         opmath_t t1 = InterpolateNonSeparable<n - 1, scalar_t, opmath_t, index_t, 2>::eval(src + i1, &data[4], &strides[4], i);
129:
130:         return t0 * w0 + t1 * w1;
131:   }
132: };
133:
134: template <typename scalar_t, typename opmath_t, typename index_t>
135: struct InterpolateNonSeparable<1, scalar_t, opmath_t, index_t, 2> {
136:     static inline opmath_t eval(char* src, char** data, const int64_t* strides, int64_t i) {
137:         index_t i0 = *(index_t*)&data[0][i * strides[0]];
138:         index_t i1 = *(index_t*)&data[2][i * strides[2]];
139:         opmath_t w0 = *(scalar_t *)&data[1][i * strides[1]];
140:         opmath_t w1 = *(scalar_t *)&data[3][i * strides[3]];
141:         opmath_t t0 = *(scalar_t *)&src[i0];
142:         opmath_t t1 = *(scalar_t *)&src[i1];
143:         return t0 * w0 + t1 * w1;
144:     }
145: };
```
- EN: The main symbol in this range is `eval`, `InterpolateNonSeparable`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `eval`, `InterpolateNonSeparable`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 147-200
```cpp
147: template <int n, typename scalar_t, typename index_t, int interp_size>
148: inline scalar_t interpolate_non_separable(char* src, char** data, const int64_t* strides, int64_t i) {
149:   using opmath_t = at::opmath_type<scalar_t>;
150:   return InterpolateNonSeparable<n, scalar_t, opmath_t, index_t, interp_size>::eval(src, data, strides, i);
151: }
152:
153: // ---- Separable interpolation ----
154: //
155: // Used by: bilinear/bicubic with antialias=True, and bilinear/bicubic uint8
156: // (as fallback when AVX isn't supported).
157: // Processes one spatial dimension at a time. The outer loop over dimensions
158: // is in upsample_separable_Nd_kernel_impl.
159: //
160: // Call chain:
161: //   upsample_separable_Nd_kernel_impl  (loops over dims)
162: //     -> upsample_separable_1d
163: //       -> basic_loop_separable_1d_horizontal  (for last spatial dim, i.e. W)
164: //         -> interpolate_separable_1d
165: //       -> basic_loop_separable_1d_vertical  (for other spatial dims, e.g. H, D)
166: //         -> interpolate_separable_1d_zero_strides
167:
168: template <typename scalar_t, typename index_t>
169: inline scalar_t interpolate_separable_1d_zero_strides(
170:     char* src,
171:     char** data,
172:     const index_t ids_stride) {
173:   const index_t ids_min = *(index_t*)&data[0][0];
174:   const index_t ids_size = *(index_t*)&data[1][0];
175:
176:   char* src_min = src + ids_min;
177:
178:   scalar_t t = *(scalar_t*)&src_min[0];
179:   index_t wts_idx = *(index_t*)&data[4][0];
180:   scalar_t* wts_ptr = (scalar_t*)&data[3][wts_idx];
181:   scalar_t wts = wts_ptr[0];
182:
183:   scalar_t output = t * wts;
184:   for (const auto j : c10::irange(1, ids_size)) {
185:     wts = wts_ptr[j];
186:     t = *(scalar_t*)&src_min[j * ids_stride];
187:     output += t * wts;
188:   }
189:   return output;
190: }
191:
192: template <typename scalar_t, typename index_t>
193: inline scalar_t interpolate_separable_1d(
194:     char* src,
195:     char** data,
196:     const int64_t* strides,
197:     int64_t i,
198:     const index_t ids_stride) {
199:   index_t ids_min = *(index_t*)&data[0][i * strides[0]];
200:   index_t ids_size = *(index_t*)&data[1][i * strides[1]];
```
- EN: The main symbol in this range is `interpolate_non_separable`, `upsample_separable_Nd_kernel_impl`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `interpolate_non_separable`, `upsample_separable_Nd_kernel_impl`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 202-248
```cpp
202:   char* src_min = src + ids_min;
203:
204:   scalar_t t = *(scalar_t*)&src_min[0];
205:   index_t wts_idx = *(index_t*)&data[4][i * strides[4]];
206:   scalar_t* wts_ptr = (scalar_t*)&data[3][wts_idx];
207:   scalar_t wts = wts_ptr[0];
208:
209:   scalar_t output = t * wts;
210:   for (const auto j : c10::irange(1, ids_size)) {
211:     wts = wts_ptr[j];
212:     t = *(scalar_t*)&src_min[j * ids_stride];
213:     output += t * wts;
214:   }
215:   return output;
216: }
217:
218: template<int m>
219: inline bool is_zero_stride(const int64_t* strides) {
220:   bool output = strides[0] == 0;
221:   for (const auto i : c10::irange(1, m)) {
222:     output &= (strides[i] == 0);
223:   }
224:   return output;
225: }
226:
227: template <typename scalar_t, typename index_t, int interp_size>
228: inline bool is_contiguous_stride(const int64_t* strides) {
229:   bool output = (strides[0] == sizeof(index_t)) && (strides[1] == sizeof(scalar_t));
230:   for (int i=2; i<2 * interp_size; i+=2) {
231:     output &= (strides[i] == sizeof(index_t)) && (strides[i + 1] == sizeof(scalar_t));
232:   }
233:   return output;
234: }
235:
236: // Helper class to recursively check if all input strides corresponding to interpolated dimensions
237: // are equal zero except on a single dimension.
238: //
239: // Note: "zero-stride" and "non-zero stride" here refer to the strides of the
240: // pre-computed indices and weights tensors (extra TensorIterator inputs), NOT
241: // to the contiguity of the input or output data tensors.
242: //
243: // Inputs: array of strides of size N, non_zero_stride_dim which can be -1, 0, 1, 2, ...
244: //   if non_zero_stride_dim, we check that all strides are equal zero, otherwise
245: //   4 strides corresponding to the strides for index_0, weight_0, index_1 and weight_1 for non_zero_stride_dim
246: //   dimension should be non zero.
247: //
248: // Unit check of the recursion is to verify whether 4 strides for one interpolated dimension are either zero,
```
- EN: The main symbol in this range is `is_zero_stride`, `is_contiguous_stride`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `is_zero_stride`, `is_contiguous_stride`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 249-305
```cpp
249: // see method is_zero_stride, or (sizeof(index_t), sizeof(scalar_t), sizeof(index_t), sizeof(scalar_t)), see
250: // method is_contiguous_stride.
251: //
252: // In practice, we have the following cases:
253: // - for ND, float32, channel first, strides are
254: //         dimN-1,              dim1,           dim0
255: //         i0, w0, i1, w1, ..., i0, w0, i1, w1, i0, w0, i1, w1
256: // strides=(0,  0,  0,  0, ...,  0,  0,  0,  0,  4,  4,  4,  4)
257: //
258: // if size dim0 is 1 then its strides are 0 and dim1 strides are equal 4
259: //
260: // - for ND, float32, channel last, strides are
261: //         dimN-1,         dimN-2,             dim0
262: //         i0, w0, i1, w1, i0, w0, i1, w1, ... i0, w0, i1, w1
263: // strides=(0,  0,  0,  0,  0,  0,  0,  0, ..., 0,  0,  0,  0)
264: //
265: // Using these methods we can hint the compiler to factorize constant indices and weights
266: // in upsample_non_separable
267: template <int N, int non_zero_stride_dim, typename scalar_t, typename index_t, int interp_size>
268: struct CheckAlmostAllZeroStrides {
269:   static inline bool eval(const int64_t* strides) {
270:     // N is dim index: N -> dim0, N-1 -> dim1, ...
271:     // non_zero_stride_dim should be out_dims - dim
272:     bool output = false;
273:     if constexpr (N == non_zero_stride_dim) {
274:       output = is_contiguous_stride<scalar_t, index_t, interp_size>(strides);
275:     } else {
276:       output = is_zero_stride<2 * interp_size>(strides);
277:     }
278:     return output &&
279:       CheckAlmostAllZeroStrides<N - 1, non_zero_stride_dim, scalar_t, index_t, interp_size>::eval(
280:         &strides[2 * interp_size]);
281:   }
282: };
283:
284: template <int non_zero_stride_dim, typename scalar_t, typename index_t, int interp_size>
285: struct CheckAlmostAllZeroStrides<0, non_zero_stride_dim, scalar_t, index_t, interp_size> {
286:   static inline bool eval(const int64_t* /*strides*/) {
287:     return true;
288:   }
289: };
290:
291: template <int n, int s, typename scalar_t, typename index_t, int interp_size>
292: inline bool check_almost_all_zero_stride(const int64_t* strides) {
293:   return CheckAlmostAllZeroStrides<n, s, scalar_t, index_t, interp_size>::eval(strides);
294: }
295:
296: // Helper method to compute interpolation for nearest, linear, cubic modes
297: template <typename scalar_t, typename index_t, int out_ndims, int interp_size>
298: inline void basic_loop_non_separable(char** data, const int64_t* strides, int64_t n) {
299:   char* dst = data[0];
300:   char* src = data[1];
301:   for (const auto i : c10::irange(n)) {
302:     *(scalar_t*)&dst[i * strides[0]] = interpolate_non_separable<out_ndims, scalar_t, index_t, interp_size>(
303:         src + i * strides[1], &data[2], &strides[2], i);
304:   }
305: }
```
- EN: The main symbol in this range is `eval`, `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `eval`, `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 307-361
```cpp
307: template <typename scalar_t>
308: inline void basic_loop_separable_1d_vertical(
309:     char** data,
310:     const int64_t* strides,
311:     int64_t n,
312:     unsigned int weights_precision) {
313:   char* dst = data[0];
314:   char* src = data[1];
315:   // index stride is constant for the given dimension
316:   const int64_t ids_stride = *(int64_t*)&data[2 + 2][0];
317:
318:   for (const auto i : c10::irange(n)) {
319:     *(scalar_t*)&dst[i * strides[0]] =
320:         interpolate_separable_1d_zero_strides<scalar_t, int64_t>(
321:             src + i * strides[1], &data[2], ids_stride);
322:   }
323: }
324:
325: template <>
326: inline void basic_loop_separable_1d_vertical<uint8_t>(
327:     char** data,
328:     const int64_t* strides,
329:     int64_t n,
330:     unsigned int weights_precision) {
331:   // See Note [ Weights computation for uint8_t and multiplication trick ]
332:   char* dst = data[0];
333:   char* src = data[1];
334:
335:   // index stride is constant for the given dimension
336:   const int64_t ids_stride = *(int64_t*)&data[2 + 2][0];
337:   const int64_t ids_size = *(int64_t*)&data[2 + 1][0];
338:   const int64_t ids_min = *(int64_t*)&data[2 + 0][0];
339:
340:   int64_t i = 0;
341:
342:   for (; i<n; i++) {
343:
344:     char* src_min = src + i * strides[1] + ids_min;
345:
346:     uint8_t t = *(uint8_t*)&src_min[0];
347:     int64_t wts_idx = *(int64_t*)&data[2 + 4][0];
348:     int16_t* wts_ptr = (int16_t*)&data[2 + 3][wts_idx];
349:     int16_t wts = wts_ptr[0];
350:
351:     // Intermediate computations are using integer type
352:     int output = 1 << (weights_precision - 1);  // accounts for the +0.5 part
353:     output += t * wts;
354:     for (const auto j : c10::irange(1, ids_size)) {
355:       wts = wts_ptr[j];
356:       t = *(uint8_t*)&src_min[j * ids_stride];
357:       output += t * wts;
358:     }
359:     *(uint8_t*)&dst[i * strides[0]] = (uint8_t)std::clamp(output >> weights_precision, 0, 255);
360:   }
361: }
```
- EN: The main symbol in this range is `basic_loop_separable_1d_vertical`, `basic_loop_separable_1d_vertical<uint8_t>`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `basic_loop_separable_1d_vertical`, `basic_loop_separable_1d_vertical<uint8_t>`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 363-409
```cpp
363: template <typename scalar_t>
364: inline void basic_loop_separable_1d_horizontal(
365:     char** data,
366:     const int64_t* strides,
367:     int64_t n,
368:     unsigned int weights_precision) {
369:   char* dst = data[0];
370:   char* src = data[1];
371:   // index stride is constant for the given dimension
372:   const int64_t ids_stride = *(int64_t*)&data[2 + 2][0];
373:
374:   if (strides[1] == 0) {
375:     for (const auto i : c10::irange(n)) {
376:       *(scalar_t*)&dst[i * strides[0]] =
377:           interpolate_separable_1d<scalar_t, int64_t>(
378:               src, &data[2], &strides[2], i, ids_stride);
379:     }
380:   } else {
381:     for (const auto i : c10::irange(n)) {
382:       *(scalar_t*)&dst[i * strides[0]] =
383:           interpolate_separable_1d<scalar_t, int64_t>(
384:               src + i * strides[1], &data[2], &strides[2], i, ids_stride);
385:     }
386:   }
387: }
388:
389: template <>
390: inline void basic_loop_separable_1d_horizontal<uint8_t>(
391:     char** data,
392:     const int64_t* strides,
393:     int64_t n,
394:     unsigned int weights_precision) {
395:   // See Note [ Weights computation for uint8_t and multiplication trick ]
396:   char* dst = data[0];
397:   char* src = data[1];
398:   // index stride is constant for the given dimension
399:   const int64_t ids_stride = *(int64_t*)&data[2 + 2][0];
400:
401:   int64_t i = 0;
402:
403:   // Here we are implementing data interpolation within the same line (vs between the lines)
404:   // output[x, y] = input[xmin[x], y] * W[x] + input[xmin[x] + 1, y] * W[x + 1] + ... + input[xmin[x] + xsize, y] * W[x + xsize]
405:
406:   for (; i<n; i++) {
407:
408:     int64_t ids_min = *(int64_t*)&data[2 + 0][i * strides[2 + 0]];
409:     int64_t ids_size = *(int64_t*)&data[2 + 1][i * strides[2 + 1]];
```
- EN: The main symbol in this range is `basic_loop_separable_1d_horizontal`, `basic_loop_separable_1d_horizontal<uint8_t>`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `basic_loop_separable_1d_horizontal`, `basic_loop_separable_1d_horizontal<uint8_t>`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 411-463
```cpp
411:     char* src_min = src + i * strides[1] + ids_min;
412:
413:     uint8_t t = *(uint8_t*)&src_min[0];
414:     int64_t wts_idx = *(int64_t*)&data[2 + 4][i * strides[2 + 4]];
415:     int16_t* wts_ptr = (int16_t*)&data[2 + 3][wts_idx];
416:     int16_t wts = wts_ptr[0];
417:
418:     // Intermediate computations are using integer type
419:     int output = 1 << (weights_precision - 1);  // accounts for the +0.5 part
420:     output += t * wts;
421:     for (const auto j : c10::irange(1, ids_size)) {
422:       wts = wts_ptr[j];
423:       t = *(uint8_t*)&src_min[j * ids_stride];
424:       output += t * wts;
425:     }
426:     *(uint8_t*)&dst[i * strides[0]] = (uint8_t)std::clamp(output >> weights_precision, 0, 255);
427:   }
428: }
429:
430: // Generic upsampling computation method using TensorIterator for Nd case.
431: // Supports: nearest, linear, cubic modes with interp_size template argument: 1, 2, 4
432: //
433: // Single loop function for 1d, 2d and 3d cases and modes
434: // For N dimensions, output value up to Di dimension can be computed as
435: //
436: // output_i[a] = interpolate(output_{i+1}[a], w_{i+1}[a], output_{i+1}[a+1], w_{i+1}[a+1], ...)
437: // with
438: // output_DN[a] = interpolate(input_DN[a], w_DN[a], input_DN[a+1], w_DN[a+1], ...)
439: // and i - dimension index and a - linear index for spatial coordinates
440: //
441: // The recursive call is implemented with the InterpolateNonSeparable struct using template for
442: // the loop unrolling on compile time.
443: template <typename scalar_t, int out_ndims, int interp_size>
444: void upsample_non_separable(at::TensorIterator& iter)
445: {
446:   auto loop = [&](char** data, const int64_t* strides, int64_t n) {
447:     // special-cases to let the compiler apply compile-time input-specific optimizations
448:     if ((strides[0] == sizeof(scalar_t) && (strides[1] == 0) &&
449:         // NOLINTNEXTLINE(bugprone-branch-clone)
450:         check_almost_all_zero_stride<out_ndims, 1, scalar_t, int64_t, interp_size>(&strides[2]))) {
451:       // contiguous channels-first case
452:       basic_loop_non_separable<scalar_t, int64_t, out_ndims, interp_size>(data, strides, n);
453:     } else if ((strides[0] == sizeof(scalar_t) && (strides[1] == sizeof(scalar_t)) &&
454:                check_almost_all_zero_stride<out_ndims, -1, scalar_t, int64_t, interp_size>(&strides[2]))) {
455:       // contiguous channels-last case
456:       basic_loop_non_separable<scalar_t, int64_t, out_ndims, interp_size>(data, strides, n);
457:     } else {
458:       // fallback
459:       basic_loop_non_separable<scalar_t, int64_t, out_ndims, interp_size>(data, strides, n);
460:     }
461:   };
462:   iter.for_each(loop);
463: }
```
- EN: The main symbol in this range is `interpolate`, `using`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `interpolate`, `using`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 465-513
```cpp
465: template <typename scalar_t, typename scale_type, nearest_idx_fn_t nearest_idx_fn>
466: void upsample_nearest_channels_last(
467:     const Tensor& output_,
468:     const Tensor& input_,
469:     const scale_type& scales) {
470:   TORCH_CHECK(input_.dtype() == output_.dtype(), "expected dtype ", input_.dtype(),
471:               " for `output` but got dtype ", output_.dtype());
472:
473:   auto input_sizes = input_.sizes().vec();
474:   auto output_sizes = output_.sizes().vec();
475:   auto ndim = input_sizes.size();
476:   TORCH_CHECK(ndim >=4 && ndim <= 5, "Upsample with NHWC format supports tensors with 4 or 5 dims.")
477:
478:   auto channels_last_memory_format = ndim == 4 ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::ChannelsLast3d;
479:   auto input = input_.contiguous(channels_last_memory_format);
480:   auto output = output_.contiguous(channels_last_memory_format);
481:
482:   auto input_data = input.const_data_ptr<scalar_t>();
483:   auto output_data = output.data_ptr<scalar_t>();
484:
485:   int64_t num_batches =  input_sizes[0];
486:   int64_t channels =  input_sizes[1];
487:   int64_t input_depth = (ndim == 5) ? input_sizes[2] : 1;
488:   int64_t output_depth = (ndim == 5) ? output_sizes[2] : 1;
489:   int64_t input_height = input_sizes[ndim - 2];
490:   int64_t output_height = output_sizes[ndim - 2];
491:   int64_t input_width = input_sizes[ndim - 1];
492:   int64_t output_width = output_sizes[ndim - 1];
493:   int64_t numel = output.numel();
494:
495:   TORCH_CHECK(channels > 0, "expected input and output channels greater than 0 but got ", channels);
496:
497:   using Vec = vec::Vectorized<scalar_t>;
498:   auto copy = [](scalar_t* out, const scalar_t* in, int64_t size) {
499:     int64_t d = 0;
500:     for (; d < size - (size % Vec::size()); d += Vec::size()) {
501:       Vec out_vec = Vec::loadu(in + d);
502:       out_vec.store(out + d);
503:     }
504:     for (; d < size; d++) {
505:       out[d] = in[d];
506:     }
507:   };
508:
509:   auto loop2d = [&](int64_t begin, int64_t end) {
510:     int64_t n = 0;
511:     int64_t oh = 0;
512:     int64_t ow = 0;
513:     data_index_init(begin, n, num_batches, oh, output_height, ow, output_width);
```
- EN: The main symbol in this range is `upsample_nearest_channels_last`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `upsample_nearest_channels_last`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 515-563
```cpp
515:     for (const auto i : c10::irange(begin, end)) {
516:       int64_t ih = nearest_idx_fn(oh, input_height, output_height, scales[0]);
517:       int64_t iw = nearest_idx_fn(ow, input_width, output_width, scales[1]);
518:       scalar_t* output_ptr = output_data + i * channels;
519:       const scalar_t* input_ptr = input_data + n * input_height * input_width * channels +
520:           ih * input_width * channels + iw * channels;
521:       copy(output_ptr, input_ptr, channels);
522:       data_index_step(n, num_batches, oh, output_height, ow, output_width);
523:     }
524:   };
525:
526:   auto loop3d = [&](int64_t begin, int64_t end) {
527:     int64_t n = 0;
528:     int64_t od = 0;
529:     int64_t oh = 0;
530:     int64_t ow = 0;
531:     data_index_init(begin, n, num_batches, od, output_depth, oh, output_height, ow, output_width);
532:
533:     for (const auto i : c10::irange(begin, end)) {
534:       int64_t id = nearest_idx_fn(od, input_depth, output_depth, scales[0]);
535:       int64_t ih = nearest_idx_fn(oh, input_height, output_height, scales[1]);
536:       int64_t iw = nearest_idx_fn(ow, input_width, output_width, scales[2]);
537:       scalar_t* output_ptr = output_data + i * channels;
538:       const scalar_t* input_ptr = input_data + n * input_depth * input_height * input_width * channels +
539:           id * input_height * input_width * channels +
540:           ih * input_width * channels + iw * channels;
541:       copy(output_ptr, input_ptr, channels);
542:       data_index_step(n, num_batches, od, output_depth, oh, output_height, ow, output_width);
543:     }
544:   };
545:
546:   if (ndim == 4) {
547:     // upsample nearest 2d
548:     at::parallel_for(0, numel / channels, at::internal::GRAIN_SIZE / channels, loop2d);
549:   } else {
550:     // upsample nearest 3d
551:     TORCH_INTERNAL_ASSERT(ndim == 5);
552:     at::parallel_for(0, numel / channels, at::internal::GRAIN_SIZE / channels, loop3d);
553:   }
554:
555:   if (!output_.is_contiguous(channels_last_memory_format)) {
556:     output_.copy_(output);
557:   }
558: }
559:
560: template <typename scalar_t, typename accscalar_t>
561: inline VecType<scalar_t> interpolate(const scalar_t* t, accscalar_t w) {
562:   return VecType<scalar_t>::loadu(t) * VecType<scalar_t>(w);
563: }
```
- EN: The main symbol in this range is `interpolate`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `interpolate`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 565-614
```cpp
565: template <typename scalar_t, typename accscalar_t, typename... Args>
566: inline VecType<scalar_t> interpolate(const scalar_t* t, accscalar_t w, Args... args) {
567:   return VecType<scalar_t>::loadu(t) * VecType<scalar_t>(w) + interpolate(args...);
568: }
569:
570: template <typename scalar_t, typename scale_type>
571: void upsample_linear_channels_last(
572:     const Tensor& output_,
573:     const Tensor& input_,
574:     bool align_corners,
575:     const scale_type& scales) {
576:   TORCH_CHECK(input_.dtype() == output_.dtype(), "expected dtype ", input_.dtype(),
577:               " for `output` but got dtype ", output_.dtype());
578:
579:   auto input_sizes = input_.sizes().vec();
580:   auto output_sizes = output_.sizes().vec();
581:   auto ndim = input_sizes.size();
582:   TORCH_CHECK(ndim >=4 && ndim <= 5, "Upsample with NHWC format supports tensors with 4 or 5 dims.")
583:
584:   auto channels_last_memory_format = ndim == 4 ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::ChannelsLast3d;
585:   auto input = input_.contiguous(channels_last_memory_format);
586:   auto output = output_.contiguous(channels_last_memory_format);
587:
588:   auto input_data = input.const_data_ptr<scalar_t>();
589:   auto output_data = output.data_ptr<scalar_t>();
590:
591:   int64_t num_batches =  input_sizes[0];
592:   int64_t channels =  input_sizes[1];
593:   int64_t input_depth = (ndim == 5) ? input_sizes[2] : 1;
594:   int64_t output_depth = (ndim == 5) ? output_sizes[2] : 1;
595:   int64_t input_height = input_sizes[ndim - 2];
596:   int64_t output_height = output_sizes[ndim - 2];
597:   int64_t input_width = input_sizes[ndim - 1];
598:   int64_t output_width = output_sizes[ndim - 1];
599:
600:   TORCH_CHECK(channels > 0, "expected input and output channels greater than 0 but got ", channels);
601:   int64_t output_slice_size = output_depth * output_height * output_width * channels;
602:
603:   using opmath_t = at::opmath_type<scalar_t>;
604:   using Vec = vec::Vectorized<scalar_t>;
605:   auto loop2d = [&](int64_t begin, int64_t end) {
606:     const auto height_scale = area_pixel_compute_scale<opmath_t>(
607:         input_height, output_height, align_corners, scales[0]);
608:     const auto width_scale = area_pixel_compute_scale<opmath_t>(
609:         input_width, output_width, align_corners, scales[1]);
610:
611:     auto input_indexr = [=](int64_t n, int64_t h, int64_t w) {
612:       return input_data + n * input_height * input_width * channels +
613:           h * input_width * channels + w * channels;
614:     };
```
- EN: The main symbol in this range is `interpolate`, `upsample_linear_channels_last`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `interpolate`, `upsample_linear_channels_last`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 616-663
```cpp
616:     int64_t ih0 = 0, ih1 = 0, iw0 = 0, iw1 = 0;
617:     opmath_t h0lambda, h1lambda, w0lambda, w1lambda;
618:     for (const auto n : c10::irange(begin, end)) {
619:       for (const auto oh : c10::irange(output_height)) {
620:         compute_source_index_and_lambda(
621:             ih0, ih1, h0lambda, h1lambda, height_scale, oh, input_height, output_height, align_corners);
622:         for (const auto ow : c10::irange(output_width)) {
623:           compute_source_index_and_lambda(
624:               iw0, iw1, w0lambda, w1lambda, width_scale, ow, input_width, output_width, align_corners);
625:
626:           scalar_t* out = output_data + n * output_slice_size +
627:               oh * output_width * channels + ow * channels;
628:           const scalar_t* i00 = input_indexr(n, ih0, iw0);
629:           const scalar_t* i01 = input_indexr(n, ih0, iw1);
630:           const scalar_t* i10 = input_indexr(n, ih1, iw0);
631:           const scalar_t* i11 = input_indexr(n, ih1, iw1);
632:           opmath_t w00 = h0lambda * w0lambda;
633:           opmath_t w01 = h0lambda * w1lambda;
634:           opmath_t w10 = h1lambda * w0lambda;
635:           opmath_t w11 = h1lambda * w1lambda;
636:
637:           int64_t size = channels;
638:           int64_t d = 0;
639:           for (; d < size - (size % Vec::size()); d += Vec::size()) {
640:             auto out_vec = interpolate(i00 + d, w00, i01 + d, w01, i10 + d, w10, i11 + d, w11);
641:             out_vec.store(out + d);
642:           }
643:           for (; d < size; d++) {
644:             out[d] = i00[d] * w00 + i01[d] * w01 + i10[d] * w10 + i11[d] * w11;
645:           }
646:         }
647:       }
648:     }
649:   };
650:
651:   auto loop3d = [&](int64_t begin, int64_t end) {
652:     const auto depth_scale = area_pixel_compute_scale<opmath_t>(
653:         input_depth, output_depth, align_corners, scales[0]);
654:     const auto height_scale = area_pixel_compute_scale<opmath_t>(
655:         input_height, output_height, align_corners, scales[1]);
656:     const auto width_scale = area_pixel_compute_scale<opmath_t>(
657:         input_width, output_width, align_corners, scales[2]);
658:
659:     auto input_indexr = [=](int64_t n, int64_t d, int64_t h, int64_t w) {
660:       return input_data + n * input_depth * input_height * input_width * channels +
661:           d * input_height * input_width * channels +
662:           h * input_width * channels + w * channels;
663:     };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 665-715
```cpp
665:     int64_t id0 = 0, id1 = 0, ih0 = 0, ih1 = 0, iw0 = 0, iw1 = 0;
666:     opmath_t d0lambda, d1lambda, h0lambda, h1lambda, w0lambda, w1lambda;
667:     for (const auto n : c10::irange(begin, end)) {
668:       for (const auto od : c10::irange(output_depth)) {
669:         compute_source_index_and_lambda(
670:             id0, id1, d0lambda, d1lambda, depth_scale, od, input_depth, output_depth, align_corners);
671:         for (const auto oh : c10::irange(output_height)) {
672:           compute_source_index_and_lambda(
673:               ih0, ih1, h0lambda, h1lambda, height_scale, oh, input_height, output_height, align_corners);
674:           for (const auto ow : c10::irange(output_width)) {
675:             compute_source_index_and_lambda(
676:                 iw0, iw1, w0lambda, w1lambda, width_scale, ow, input_width, output_width, align_corners);
677:
678:             scalar_t* out = output_data + n * output_slice_size +
679:                 od * output_height * output_width * channels +
680:                 oh * output_width * channels + ow * channels;
681:             const scalar_t* i000 = input_indexr(n, id0, ih0, iw0);
682:             const scalar_t* i001 = input_indexr(n, id0, ih0, iw1);
683:             const scalar_t* i010 = input_indexr(n, id0, ih1, iw0);
684:             const scalar_t* i011 = input_indexr(n, id0, ih1, iw1);
685:             const scalar_t* i100 = input_indexr(n, id1, ih0, iw0);
686:             const scalar_t* i101 = input_indexr(n, id1, ih0, iw1);
687:             const scalar_t* i110 = input_indexr(n, id1, ih1, iw0);
688:             const scalar_t* i111 = input_indexr(n, id1, ih1, iw1);
689:             opmath_t w000 = d0lambda * h0lambda * w0lambda;
690:             opmath_t w001 = d0lambda * h0lambda * w1lambda;
691:             opmath_t w010 = d0lambda * h1lambda * w0lambda;
692:             opmath_t w011 = d0lambda * h1lambda * w1lambda;
693:             opmath_t w100 = d1lambda * h0lambda * w0lambda;
694:             opmath_t w101 = d1lambda * h0lambda * w1lambda;
695:             opmath_t w110 = d1lambda * h1lambda * w0lambda;
696:             opmath_t w111 = d1lambda * h1lambda * w1lambda;
697:
698:             int64_t size = channels;
699:             int64_t d = 0;
700:             for (; d < size - (size % Vec::size()); d += Vec::size()) {
701:               auto out_vec = interpolate(
702:                   i000 + d, w000, i001 + d, w001, i010 + d, w010, i011 + d, w011,
703:                   i100 + d, w100, i101 + d, w101, i110 + d, w110, i111 + d, w111);
704:               out_vec.store(out + d);
705:             }
706:             for (; d < size; d++) {
707:               out[d] =
708:                   i000[d] * w000 + i001[d] * w001 + i010[d] * w010 + i011[d] * w011 +
709:                   i100[d] * w100 + i101[d] * w101 + i110[d] * w110 + i111[d] * w111;
710:             }
711:           }
712:         }
713:       }
714:     }
715:   };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 717-769
```cpp
717:   if (ndim == 4) {
718:     // bilinear 2d
719:     at::parallel_for(0, num_batches, at::internal::GRAIN_SIZE / output_slice_size / 4, loop2d);
720:   } else {
721:     // trilinear 3d
722:     TORCH_INTERNAL_ASSERT(ndim == 5);
723:     at::parallel_for(0, num_batches, at::internal::GRAIN_SIZE / output_slice_size / 8, loop3d);
724:   }
725:
726:   if (!output_.is_contiguous(channels_last_memory_format)) {
727:     output_.copy_(output);
728:   }
729: }
730:
731: // Helper structs to use with upsample_non_separable_Nd_kernel_impl
732: struct HelperInterpBase {
733:
734:   static inline void init_indices_weights(
735:     at::ScalarType output_type,
736:     std::vector<Tensor> & output, int64_t output_size, int64_t ndims,
737:     int64_t reshape_dim, int interp_size
738:   ) {
739:
740:     auto new_shape = std::vector<int64_t>(ndims, 1);
741:     new_shape[reshape_dim] = output_size;
742:
743:     for ([[maybe_unused]] const auto j : c10::irange(interp_size)) {
744:       output.emplace_back(
745:           empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<int64_t>())));
746:       output.emplace_back(empty(new_shape, at::device(kCPU).dtype(output_type)));
747:     }
748:   }
749:
750:   // This is a helper function for _compute_index_ranges_weights method that computes
751:   // source two int64 scalars index min and size and a list weights (of size max_interp_size)
752:   // for interpolation with antialiasing=true mode. It returns the maximal weights value
753:   template <typename scalar_t, typename aa_filter_fn_t>
754:   static inline scalar_t _compute_indices_min_size_weights_aa(
755:     const int64_t i, const int64_t input_size, const scalar_t scale, const scalar_t support,
756:     scalar_t* wt_ptr, const int64_t max_interp_size, aa_filter_fn_t filter_fn,
757:     int64_t& xmin, int64_t& xsize
758:   ) {
759:
760:     scalar_t center = scale * (i + 0.5);
761:     scalar_t total_w = 0.0;
762:     scalar_t invscale = (scale >= 1.0) ? 1.0 / scale : 1.0;
763:     xmin = std::max(
764:         static_cast<int64_t>(center - support + 0.5), static_cast<int64_t>(0));
765:     xsize = std::min(
766:         static_cast<int64_t>(center + support + 0.5), input_size) - xmin;
767:     // There are rare cases when due to precision xsize can be larger than max_interp_size by one.
768:     // We have to clip the value
769:     xsize = std::clamp(xsize, static_cast<int64_t>(0), max_interp_size);
```
- EN: The main symbol in this range is `init_indices_weights`, `weights`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `init_indices_weights`, `weights`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 771-824
```cpp
771:     int64_t j = 0;
772:     for (; j < xsize; j++) {
773:       scalar_t w = filter_fn((j + xmin - center + 0.5) * invscale);
774:       wt_ptr[j] = w;
775:       total_w += w;
776:     }
777:
778:     scalar_t wt_max = 0.0;
779:     if (total_w != 0.0) {
780:       for (j = 0; j < xsize; j++) {
781:         wt_ptr[j] /= total_w;
782:         wt_max = std::max(wt_max, wt_ptr[j]);
783:       }
784:     }
785:
786:     for (; j < max_interp_size; j++) {
787:       wt_ptr[j] = static_cast<scalar_t>(0.0);
788:     }
789:     return wt_max;
790:   }
791:
792:   // This is a helper function for _compute_index_ranges_weights method that computes
793:   // source two int64 scalars index min and size and a list weights (of size max_interp_size)
794:   // for interpolation with antialiasing=false mode. It returns the maximal weights value.
795:   // This function is templated with scalar_t for type of scale and weights but is only used for
796:   // bilinear/bicubic modes on uint8 input and antialiasing=false (in this case scalar_t is double).
797:   // For float input types we are using upsample_non_separable_Nd_kernel_impl and compute_indices_weights methods
798:   template <typename scalar_t, typename aa_filter_fn_t>
799:   static inline scalar_t _compute_indices_min_size_weights(
800:     const int64_t i, const int64_t input_size, const scalar_t scale,
801:     scalar_t* wt_ptr, const int64_t max_interp_size, aa_filter_fn_t filter_fn,
802:     bool align_corners, int64_t& index_min, int64_t& index_size
803:   ) {
804:     // Notes. We do not use opmath_t in this method as f16 and other smaller float types are not routed here.
805:     // Typical usage of this method is with scalar_t = double when computing indices and weights for uint8 input
806:     // The code below partly adapts indices and lambda computation from compute_indices_weights method and
807:     // index_min/index_size from _compute_indices_min_size_weights_aa
808:
809:     bool cubic = max_interp_size > 2;
810:     const auto real_input_index = area_pixel_compute_source_index<scalar_t>(
811:         scale, i, align_corners, /*cubic=*/cubic);
812:
813:     scalar_t lambda;
814:     int64_t input_index = 0;
815:     guard_index_and_lambda(real_input_index, input_size, input_index, lambda);
816:
817:     const auto support = static_cast<int64_t>(max_interp_size * 0.5);
818:     const auto unbound_index_min = input_index - support + 1;
819:     const auto unbound_index_max = input_index + support + 1;
820:     index_min = std::max(unbound_index_min, static_cast<int64_t>(0));
821:     index_size = std::min(unbound_index_max, input_size) - index_min;
822:     // There are rare cases when due to precision xsize can be larger than max_interp_size by one.
823:     // We have to clip the value
824:     index_size = std::clamp(index_size, static_cast<int64_t>(0), max_interp_size);
```
- EN: The main symbol in this range is `weights`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `weights`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 826-881
```cpp
826:     // Below the weights are computed using filter_fn and accumulating values for indices being out of bounds
827:     // For example, for bicubic mode for output index i = 0, we have input_index = -1,
828:     // then we have unbound_index_min = -2 and unbound_index_max = 1 => unbounded input indices are [-2, -1, 0, 1] and
829:     // valid input indices will be [0, 1]
830:     // For unbounded input indices we compute four non-zero weights values [w0, w1, w2, w3] and as only two weights can
831:     // be used with valid input indcies, we accumulate values in the following way: [w0 + w1 + w2, w3, 0.0, 0.0]
832:     // This is equivalent to the float path which would compute indices as [0, 0, 0, 1] and weights as [w0, w1, w2, s3].
833:     // A similar accumulation should done for unbounded indices larger than input size.
834:     auto w_index = 0;
835:     scalar_t wt_max = 0.0;
836:     for (const auto j : c10::irange(max_interp_size)) {
837:       // initialize weights value as we will accumulate below
838:       wt_ptr[j] = 0.0;
839:
840:       scalar_t w = filter_fn(static_cast<scalar_t>(j + 1 - support) - lambda);
841:       if (unbound_index_min + j <= 0) {
842:         w_index = 0;
843:       } else if (unbound_index_min + j >= input_size - 1) {
844:         w_index = index_size - 1;
845:       }
846:       wt_ptr[w_index] += w;
847:       wt_max = std::max(wt_max, wt_ptr[w_index]);
848:       w_index++;
849:     }
850:
851:     return wt_max;
852:   }
853:
854:   // Note [ Support for antialias=False as a subcase of antialias=True ]
855:   // This function was originally written with the hard assumption that
856:   // antialias=True and it was later extended to support antialias=False.
857:   // The only difference between aa and no-aa is in how the
858:   // weights and indices are computed (and their number). In aa their number is
859:   // variable but with no-aa, they're fixed to interp_size. The same "filters"
860:   // can be used otherwise. HOWEVER, support for antialias=False here may not be
861:   // optimally optimized: the code assumes an arbitrary number of weights and
862:   // indices, but this can be optimized further when aa=False since we know
863:   // their actual dimensions.
864:   template <typename scalar_t, typename aa_filter_fn_t, int weight_index_stride=sizeof(scalar_t)>
865:   static inline std::tuple<std::vector<Tensor>, int, scalar_t> _compute_index_ranges_weights(
866:     int64_t input_size, int64_t output_size, int64_t stride, int64_t ndims,
867:     int64_t reshape_dim, scalar_t scale,
868:     int interp_size, aa_filter_fn_t aa_filter_fn, bool antialias, bool align_corners
869:   ) {
870:
871:     std::vector<Tensor> output;
872:
873:     scalar_t support;
874:     int max_interp_size = 0;
875:     if (antialias) {
876:         support = (scale >= 1.0) ? (interp_size * 0.5) * scale : interp_size * 0.5;
877:         max_interp_size = (int) std::ceil(support) * 2 + 1;
878:     } else {
879:         support = interp_size * 0.5;
880:         max_interp_size = interp_size;
881:     }
```
- EN: The main symbol in this range is `computed`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `computed`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 883-941
```cpp
883:     auto new_shape = std::vector<int64_t>(ndims, 1);
884:     new_shape[reshape_dim] = output_size;
885:
886:     // Bounds approach as in PIL: xmin/xmax
887:     output.emplace_back(
888:         empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<int64_t>())));
889:     output.emplace_back(
890:         empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<int64_t>())));
891:     output.emplace_back(
892:         empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<int64_t>())));
893:
894:     {
895:       // Weights
896:       new_shape[reshape_dim] = output_size * max_interp_size;
897:       auto wts = empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<scalar_t>()));
898:       auto strides = wts.strides().vec();
899:       strides[reshape_dim] = 0;
900:       new_shape[reshape_dim] = output_size;
901:       wts = wts.as_strided(new_shape, strides);
902:       output.emplace_back(wts);
903:       // Weights indices
904:       output.emplace_back(
905:           empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<int64_t>())));
906:     }
907:
908:     int64_t* idx_ptr_xmin = output[0].data_ptr<int64_t>();
909:     int64_t* idx_ptr_size = output[1].data_ptr<int64_t>();
910:     int64_t* idx_ptr_stride = output[2].data_ptr<int64_t>();
911:     scalar_t* wt_ptr = output[3].data_ptr<scalar_t>();
912:     int64_t* wt_idx_ptr = output[4].data_ptr<int64_t>();
913:
914:     scalar_t wt_max = 0.0;
915:     for (const auto i : c10::irange(output_size)) {
916:       int64_t xmin = 0, xsize = 0;
917:       scalar_t wt_max_i;
918:       if (antialias) {
919:         wt_max_i = HelperInterpBase::_compute_indices_min_size_weights_aa(
920:             i,
921:             input_size,
922:             scale,
923:             support,
924:             wt_ptr + i * max_interp_size,
925:             max_interp_size,
926:             aa_filter_fn,
927:             xmin,
928:             xsize);
929:       } else {
930:         wt_max_i = HelperInterpBase::_compute_indices_min_size_weights(
931:             i,
932:             input_size,
933:             scale,
934:             wt_ptr + i * max_interp_size,
935:             max_interp_size,
936:             aa_filter_fn,
937:             align_corners,
938:             xmin,
939:             xsize);
940:       }
941:       wt_max = std::max(wt_max, wt_max_i);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 943-997
```cpp
943:       idx_ptr_xmin[i] = xmin * stride;
944:       idx_ptr_size[i] = xsize;
945:       idx_ptr_stride[i] = stride;
946:       wt_idx_ptr[i] = i * max_interp_size * weight_index_stride;
947:     }
948:     return {std::move(output), max_interp_size, wt_max};
949:   }
950:
951:   /*
952:   NOTE [ Weights computation for uint8_t and multiplication trick ]
953:   When the input/output dtype is uint8_t, we still compute the interpolation
954:   weights as double, but then convert them to int16 via some conversion logic
955:   detailed below. This allows us to compute all interpolation operation (sum of
956:   multiplications) as ints instead of floats. The result is converted back into
957:   uint8 in basic_loop_separable_1d_horizontal<uint8_t> (and vertical)
958:
959:   In essence the idea is to avoid a multiplication between a float (the
960:   weight) and an int (the pixel value) and instead run a multiplication between
961:   2 ints:
962:
963:   ```py
964:   COEF_PREC = 16
965:
966:   def mul(a:float, b:int) -> Tuple[float, int]:
967:     # return a * b, round(a * b)
968:     actual = a * b
969:
970:     assert a > 0  # I'm lazy
971:     int_a = floor(0.5 + a * (1 << COEF_PREC))
972:     with_trick = ((int_a * b) + (1 << (COEF_PREC - 1))) >> COEF_PREC
973:
974:     return actual, with_trick  # round(actual) == with_trick!!
975:   ```
976:
977:   Here's how it works:
978:   N == COEFF_PREC
979:   1 << N == 2**N
980:   floor(0.5 + x) == round(x)
981:
982:   So the operation is something like
983:
984:   int_a = round(a * 2**N)  -- let's just say it's `a * 2**N` for simplicity
985:
986:   res = ((int_a * b) + (1 << (N - 1))) >> N
987:       = ((a * 2**N * b + 2**(N - 1)) / 2**N
988:       = a * b + 0.5
989:       = round(a * b)
990:       = what we wanted
991:   */
992:   template <typename aa_filter_fn_t>
993:   static inline std::tuple<std::vector<Tensor>, int, unsigned int> _compute_index_ranges_int16_weights(
994:     int64_t input_size, int64_t output_size, int64_t stride, int64_t ndims,
995:     int64_t reshape_dim, bool align_corners, const std::optional<double>& opt_scale,
996:     int interp_size, aa_filter_fn_t aa_filter_fn, bool antialias, bool align_i32=false
997:   ) {
```
- EN: The main symbol in this range is `operation`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `operation`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 999-1046
```cpp
 999:     double scale = area_pixel_compute_scale<double>(
1000:         input_size, output_size, align_corners, opt_scale);
1001:
1002:     auto [indices_weights, aligned_interp_size, wt_max] = HelperInterpBase::_compute_index_ranges_weights<double, aa_filter_fn_t, sizeof(int16_t)>(
1003:         input_size, output_size, stride, ndims, reshape_dim, scale, interp_size, aa_filter_fn, antialias, align_corners);
1004:     interp_size = aligned_interp_size;
1005:
1006:     // Rescale float weights to int16 and compute weights precision
1007:     auto weights_f64 = indices_weights[3];
1008:     double * data_f64 = weights_f64. template data_ptr<double>();
1009:
1010:     unsigned int weights_precision = 0;
1011:     for (weights_precision = 0; weights_precision < 22; ++weights_precision) {
1012:         int next_value = (int) (0.5 + wt_max * (1 << (weights_precision + 1)));
1013:         if (next_value >= (1 << 15))
1014:             break;
1015:     }
1016:
1017:     // Rescale float values to int16
1018:     int16_t * data_i16 = (int16_t *) data_f64;
1019:
1020:     if (align_i32) {
1021:       // We should respect int32 alignment as we will load int16 data as int32
1022:       // See ImagingResampleHorizontalConvolution8u4x, mmk0 = _mm256_set1_epi32(*(int32_t*)&k[x]);
1023:       // compute aligned_interp_size = nearest pair value to interp_size
1024:       while (aligned_interp_size % sizeof(int32_t) != 0) {
1025:         aligned_interp_size += 1;
1026:       }
1027:       // assert that we won't go out of bounds
1028:       TORCH_INTERNAL_ASSERT(aligned_interp_size * sizeof(int16_t) < interp_size * sizeof(double));
1029:     }
1030:
1031:     for (const auto j : c10::irange(output_size)) {
1032:       for (const auto k : c10::irange(interp_size)) {
1033:         double v = data_f64[j * interp_size + k] * (1 << weights_precision);
1034:         data_i16[j * aligned_interp_size + k] = (v < 0) ? (int) (-0.5 + v) : (int) (0.5 + v);
1035:       }
1036:     }
1037:
1038:     return {std::move(indices_weights), aligned_interp_size, weights_precision};
1039:   }
1040: };
1041:
1042: struct HelperInterpNearest : public HelperInterpBase {
1043:   // This structure implements outdated and buggy method to compute indices
1044:   // for nearest neighbours interpolation
1045:   // We keep this structure for BC and consider as deprecated.
1046:   // See HelperInterpNearestExact as replacement
```
- EN: The main symbol in this range is `HelperInterpNearest`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `HelperInterpNearest`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1048-1110
```cpp
1048:   static constexpr int interp_size = 1;
1049:
1050:   static inline void init_indices_weights(
1051:     at::ScalarType output_type,
1052:     std::vector<Tensor> & output, int64_t output_size, int64_t ndims,
1053:     int64_t reshape_dim, int interp_size
1054:   ) {
1055:     auto new_shape = std::vector<int64_t>(ndims, 1);
1056:     new_shape[reshape_dim] = output_size;
1057:
1058:     for ([[maybe_unused]] const auto j : c10::irange(interp_size)) {
1059:       output.emplace_back(
1060:           empty(new_shape, at::device(kCPU).dtype(c10::CppTypeToScalarType<int64_t>())));
1061:       // Defines weights for consistency, but not used
1062:       output.emplace_back(at::ones(new_shape, at::device(kCPU).dtype(output_type)));
1063:     }
1064:   }
1065:
1066:   // Compute nearest mode indices and weights for each interpolated dimension
1067:   // indices_weights = {
1068:   //      {indices_0, 1.0, },  // dim -n
1069:   //      {indices_0, 1.0, },  // dim -(n-1)
1070:   //      ...
1071:   //      {indices_0, 1.0, },  // dim -1
1072:   // }
1073:   // Indices and weights are reshaped as (1, 1, ..., N, ..., 1, 1) to
1074:   // fit input/output tensors.
1075:   // Indices are already containing the strides to optimize the computations
1076:   static inline std::vector<Tensor> compute_indices_weights(
1077:     at::ScalarType scalar_type,
1078:     int64_t input_size, int64_t output_size, int64_t stride, int64_t ndims,
1079:     int64_t reshape_dim, bool align_corners, const std::optional<double>& opt_scale
1080:   ) {
1081:
1082:     TORCH_INTERNAL_ASSERT(!align_corners);
1083:     std::vector<Tensor> output;
1084:     HelperInterpNearest::init_indices_weights(
1085:       scalar_type, output, output_size, ndims, reshape_dim, HelperInterpNearest::interp_size);
1086:
1087:     AT_DISPATCH_FLOATING_TYPES_AND2(
1088:       kBFloat16, kHalf, scalar_type, "compute_indices_weights_nearest", [&] {
1089:         using opmath_t = at::opmath_type<scalar_t>;
1090:         opmath_t scale = area_pixel_compute_scale<opmath_t>(input_size, output_size, align_corners, opt_scale);
1091:
1092:         auto input_index_ptr = output[0].data_ptr<int64_t>();
1093:         int64_t input_index;
1094:
1095:         // Indices are computed as following:
1096:         // scale = 1.0 * isize / osize
1097:         // index_f32 = (output_index) * scale
1098:         // input_index = floor(index_f32)
1099:         // Same as OpenCV INTER_NEAREST
1100:         for (const auto i : c10::irange(output_size)) {
1101:           const auto real_input_index =
1102:               area_pixel_compute_source_index<opmath_t>(
1103:                   scale, i, /*align_corners=*/true, /*cubic=*/false);
1104:           input_index = static_cast<int64_t>(floorf(real_input_index));
1105:           input_index_ptr[i] = static_cast<int64_t>(std::min(input_index, input_size - 1)) * stride;
1106:         }
1107:       }
1108:     );
1109:     return output;
1110:   }
```
- EN: The main symbol in this range is `init_indices_weights`, `as`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `init_indices_weights`, `as`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1112-1161
```cpp
1112: };
1113:
1114: struct HelperInterpNearestExact : public HelperInterpNearest {
1115:
1116:   // Compute nearest mode indices and weights for each interpolated dimension
1117:   // indices_weights = {
1118:   //      {indices_0, 1.0, },  // dim -n
1119:   //      {indices_0, 1.0, },  // dim -(n-1)
1120:   //      ...
1121:   //      {indices_0, 1.0, },  // dim -1
1122:   // }
1123:   // Indices and weights are reshaped as (1, 1, ..., N, ..., 1, 1) to
1124:   // fit input/output tensors.
1125:   // Indices are already containing the strides to optimize the computations
1126:   static inline std::vector<Tensor> compute_indices_weights(
1127:     at::ScalarType scalar_type,
1128:     int64_t input_size, int64_t output_size, int64_t stride, int64_t ndims,
1129:     int64_t reshape_dim, bool align_corners, const std::optional<double>& opt_scale
1130:   ) {
1131:
1132:     TORCH_INTERNAL_ASSERT(!align_corners);
1133:     std::vector<Tensor> output;
1134:     HelperInterpNearest::init_indices_weights(
1135:       scalar_type, output, output_size, ndims, reshape_dim, HelperInterpNearest::interp_size);
1136:
1137:     AT_DISPATCH_FLOATING_TYPES_AND2(
1138:       kBFloat16, kHalf, scalar_type, "compute_indices_weights_nearest", [&] {
1139:         using opmath_t = at::opmath_type<scalar_t>;
1140:         opmath_t scale = area_pixel_compute_scale<opmath_t>(input_size, output_size, align_corners, opt_scale);
1141:
1142:         auto input_index_ptr = output[0].data_ptr<int64_t>();
1143:         int64_t input_index;
1144:
1145:         // Indices should be computed as following:
1146:         // scale = 1.0 * isize / osize
1147:         // index_f32 = (output_index + 0.5) * scale - 0.5
1148:         // input_index = round(index_f32)
1149:         // Same as Pillow and Scikit-Image/Scipy ndi.zoom
1150:         for (const auto i : c10::irange(output_size)) {
1151:           const auto real_input_index =
1152:               area_pixel_compute_source_index<opmath_t>(
1153:                   scale, i, /*align_corners=*/align_corners, /*cubic=*/false);
1154:           input_index = static_cast<int64_t>(floorf(real_input_index + 0.5));
1155:           input_index_ptr[i] = static_cast<int64_t>(std::min(input_index, input_size - 1)) * stride;
1156:         }
1157:       }
1158:     );
1159:     return output;
1160:   }
1161: };
```
- EN: The main symbol in this range is `as`, `round`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `as`, `round`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1163-1212
```cpp
1163: struct HelperInterpLinear : public HelperInterpBase {
1164:
1165:   static constexpr int interp_size = 2;
1166:
1167:   // Compute indices and weights for each interpolated dimension
1168:   // indices_weights = {
1169:   //      {indices_0, weights_0, indices_1, weights_1},  // dim -n
1170:   //      {indices_0, weights_0, indices_1, weights_1},  // dim -(n-1)
1171:   //      ...
1172:   //      {indices_0, weights_0, indices_1, weights_1},  // dim -1
1173:   // }
1174:   // Indices and weights are reshaped as (1, 1, ..., N, ..., 1, 1) to
1175:   // fit input/output tensors.
1176:   // Indices are already containing the strides to optimize the computations
1177:   static inline std::vector<Tensor> compute_indices_weights(
1178:     at::ScalarType scalar_type,
1179:     int64_t input_size, int64_t output_size, int64_t stride, int64_t ndims, int64_t reshape_dim,
1180:     bool align_corners, const std::optional<double>& opt_scale
1181:   ) {
1182:     std::vector<Tensor> output;
1183:     HelperInterpLinear::init_indices_weights(
1184:       scalar_type, output, output_size, ndims, reshape_dim, HelperInterpLinear::interp_size);
1185:     AT_DISPATCH_FLOATING_TYPES_AND2(
1186:       kBFloat16, kHalf, scalar_type, "compute_indices_weights_linear", [&] {
1187:         using opmath_t = at::opmath_type<scalar_t>;
1188:         opmath_t scale = area_pixel_compute_scale<opmath_t>(input_size, output_size, align_corners, opt_scale);
1189:
1190:         auto input_index0_ptr = output[0].data_ptr<int64_t>();
1191:         auto lambda0_ptr = output[1].data_ptr<scalar_t>();
1192:         auto input_index1_ptr = output[2].data_ptr<int64_t>();
1193:         auto lambda1_ptr = output[3].data_ptr<scalar_t>();
1194:
1195:         for (const auto i : c10::irange(output_size)) {
1196:
1197:           compute_source_index_and_lambda<scalar_t, opmath_t>(
1198:             input_index0_ptr[i], input_index1_ptr[i],
1199:             lambda0_ptr[i], lambda1_ptr[i],
1200:             scale, i, input_size, output_size, align_corners
1201:           );
1202:           // put stride into indices
1203:           // index values correspond to input indices (0, 1, 2, 3, ...)
1204:           // when multiplied by input stride, maximum possible value
1205:           // input_size[dim-1] * input_size[dim-2] * ... for the given dimension.
1206:           input_index0_ptr[i] *= stride;
1207:           input_index1_ptr[i] *= stride;
1208:         }
1209:       }
1210:     );
1211:     return output;
1212:   }
```
- EN: The main symbol in this range is `as`, `HelperInterpLinear`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `as`, `HelperInterpLinear`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1214-1261
```cpp
1214:   // taken from
1215:   // https://github.com/python-pillow/Pillow/blob/6812205f18ca4ef54372e87e1a13ce4a859434df/
1216:   // src/libImaging/Resample.c#L20-L29
1217:   template<typename scalar_t>
1218:   static inline scalar_t aa_filter(scalar_t x) {
1219:     x = std::abs(x);
1220:     if (x < 1.0) {
1221:       return 1.0 - x;
1222:     }
1223:     return 0.0;
1224:   }
1225:
1226:   static inline std::vector<Tensor> compute_index_ranges_weights(
1227:     at::ScalarType scalar_type,
1228:     int64_t input_size,
1229:     int64_t output_size,
1230:     int64_t stride,
1231:     int64_t ndims,
1232:     int64_t reshape_dim,
1233:     bool align_corners,
1234:     const std::optional<double>& opt_scale,
1235:     bool antialias
1236:   ) {
1237:
1238:     std::vector<Tensor> indices_weights;
1239:     AT_DISPATCH_FLOATING_TYPES(
1240:       scalar_type, "compute_index_ranges_weights", [&] {
1241:
1242:         scalar_t scale = area_pixel_compute_scale<scalar_t>(
1243:             input_size, output_size, align_corners, opt_scale);
1244:
1245:         auto interp_size = HelperInterpLinear::interp_size;
1246:
1247:         indices_weights = std::get<0>(HelperInterpLinear::_compute_index_ranges_weights<scalar_t>(
1248:             input_size,
1249:             output_size,
1250:             stride,
1251:             ndims,
1252:             reshape_dim,
1253:             scale,
1254:             interp_size,
1255:             &HelperInterpLinear::aa_filter<scalar_t>,
1256:             /*antialias=*/antialias,
1257:             /*align_corners=*/align_corners));
1258:       }
1259:     );
1260:     return indices_weights;
1261:   }
```
- EN: The main symbol in this range is `aa_filter`, `compute_index_ranges_weights`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `aa_filter`, `compute_index_ranges_weights`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1263-1309
```cpp
1263:   static inline std::tuple<std::vector<Tensor>, int, unsigned int> compute_index_ranges_int16_weights(
1264:     int64_t input_size,
1265:     int64_t output_size,
1266:     int64_t stride,
1267:     int64_t ndims,
1268:     int64_t reshape_dim,
1269:     bool align_corners,
1270:     const std::optional<double>& opt_scale,
1271:     bool antialias,
1272:     bool align_i32=false
1273:   ) {
1274:
1275:     auto interp_size = HelperInterpLinear::interp_size;
1276:     auto fn = HelperInterpLinear::aa_filter<double>;
1277:     return HelperInterpLinear::_compute_index_ranges_int16_weights(
1278:         input_size, output_size, stride, ndims, reshape_dim,
1279:         align_corners, opt_scale, interp_size, fn, antialias, align_i32);
1280:   }
1281: };
1282:
1283: struct HelperInterpCubic : public HelperInterpBase {
1284:
1285:   static constexpr int interp_size = 4;
1286:
1287:   // Compute indices and weights for each interpolated dimension
1288:   // indices_weights = {
1289:   //      {indices_0, weights_0, indices_1, weights_1, ..., indices_3, weights_3},  // dim -n
1290:   //      {indices_0, weights_0, indices_1, weights_1, ..., indices_3, weights_3},  // dim -(n-1)
1291:   //      ...
1292:   //      {indices_0, weights_0, indices_1, weights_1, ..., indices_3, weights_3},  // dim -1
1293:   // }
1294:   // Indices and weights are reshaped as (1, 1, ..., N, ..., 1, 1) to
1295:   // fit input/output tensors.
1296:   // Indices are already containing the strides to optimize the computations
1297:   static inline std::vector<Tensor> compute_indices_weights(
1298:     at::ScalarType scalar_type,
1299:     int64_t input_size, int64_t output_size, int64_t stride, int64_t ndims, int64_t reshape_dim,
1300:     bool align_corners, const std::optional<double>& opt_scale
1301:   ) {
1302:     std::vector<Tensor> output;
1303:     HelperInterpCubic::init_indices_weights(
1304:       scalar_type, output, output_size, ndims, reshape_dim, HelperInterpCubic::interp_size);
1305:
1306:     AT_DISPATCH_FLOATING_TYPES_AND2(
1307:       kBFloat16, kHalf, scalar_type, "compute_indices_weights_cubic", [&] {
1308:         using opmath_t = at::opmath_type<scalar_t>;
1309:         opmath_t scale = area_pixel_compute_scale<opmath_t>(input_size, output_size, align_corners, opt_scale);
```
- EN: The main symbol in this range is `compute_index_ranges_int16_weights`, `as`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `compute_index_ranges_int16_weights`, `as`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1311-1357
```cpp
1311:         int64_t input_index;
1312:         int64_t zero = static_cast<int64_t>(0);
1313:         // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
1314:         opmath_t coeffs[4];
1315:
1316:         int64_t * idx_ptr;
1317:         scalar_t * wt_ptr;
1318:         for (const auto i : c10::irange(output_size)) {
1319:           const auto real_input_index =
1320:               area_pixel_compute_source_index<opmath_t>(
1321:                   scale, i, align_corners, /*cubic=*/true);
1322:           opmath_t lambda;
1323:           guard_index_and_lambda(real_input_index, input_size, input_index, lambda);
1324:           get_cubic_upsample_coefficients<opmath_t>(coeffs, lambda);
1325:
1326:           for (const auto j : c10::irange(interp_size)) {
1327:             idx_ptr = output[2 * j + 0].data_ptr<int64_t>();
1328:             idx_ptr[i] = static_cast<int64_t>(std::max(std::min(input_index + j - 1, input_size - 1), zero)) * stride;
1329:             wt_ptr = output[2 * j + 1].data_ptr<scalar_t>();
1330:             wt_ptr[i] = coeffs[j];
1331:           }
1332:         }
1333:       }
1334:     );
1335:     return output;
1336:   }
1337:
1338:   // taken from
1339:   // https://github.com/python-pillow/Pillow/blob/6812205f18ca4ef54372e87e1a13ce4a859434df/
1340:   // src/libImaging/Resample.c#L46-L62
1341:   template<typename scalar_t, bool use_keys_cubic=true>
1342:   static inline scalar_t aa_filter(scalar_t x) {
1343:     // https://en.wikipedia.org/wiki/Bicubic_interpolation#Bicubic_convolution_algorithm
1344:     // a = -0.5 was proposed by R. Keys in "Cubic convolution interpolation for digital image processing"
1345:     // We are using -0.5 for bicubic, antialiasing=true (compatibility with PIL)
1346:     // and using -0.75 for bicubic, antialiasing=false (compatibility with Opencv)
1347:     constexpr scalar_t a = use_keys_cubic ? -0.5 : -0.75;
1348:
1349:     x = std::abs(x);
1350:     if (x < 1.0) {
1351:         return cubic_convolution1(x, a);
1352:     }
1353:     if (x < 2.0) {
1354:         return cubic_convolution2(x, a);
1355:     }
1356:     return 0.0;
1357:   }
```
- EN: The main symbol in this range is `aa_filter`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `aa_filter`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1359-1406
```cpp
1359:   static inline std::vector<Tensor> compute_index_ranges_weights(
1360:     at::ScalarType scalar_type,
1361:     int64_t input_size,
1362:     int64_t output_size,
1363:     int64_t stride,
1364:     int64_t ndims,
1365:     int64_t reshape_dim,
1366:     bool align_corners,
1367:     const std::optional<double>& opt_scale,
1368:     bool antialias
1369:   ) {
1370:
1371:     std::vector<Tensor> indices_weights;
1372:     AT_DISPATCH_FLOATING_TYPES(
1373:       scalar_type, "compute_index_ranges_weights", [&] {
1374:
1375:         scalar_t scale = area_pixel_compute_scale<scalar_t>(
1376:             input_size, output_size, align_corners, opt_scale);
1377:
1378:         auto interp_size = HelperInterpCubic::interp_size;
1379:
1380:         indices_weights = std::get<0>(HelperInterpCubic::_compute_index_ranges_weights<scalar_t>(
1381:             input_size,
1382:             output_size,
1383:             stride,
1384:             ndims,
1385:             reshape_dim,
1386:             scale,
1387:             interp_size,
1388:             &HelperInterpCubic::aa_filter<scalar_t>,
1389:             /*antialias=*/antialias,
1390:             /*align_corners=*/align_corners));
1391:       }
1392:     );
1393:     return indices_weights;
1394:   }
1395:
1396:   static inline std::tuple<std::vector<Tensor>, int, unsigned int> compute_index_ranges_int16_weights(
1397:     int64_t input_size,
1398:     int64_t output_size,
1399:     int64_t stride,
1400:     int64_t ndims,
1401:     int64_t reshape_dim,
1402:     bool align_corners,
1403:     const std::optional<double>& opt_scale,
1404:     bool antialias,
1405:     bool align_i32=false
1406:   ) {
```
- EN: The main symbol in this range is `compute_index_ranges_weights`, `compute_index_ranges_int16_weights`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `compute_index_ranges_weights`, `compute_index_ranges_int16_weights`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1408-1454
```cpp
1408:     auto interp_size = HelperInterpCubic::interp_size;
1409:     // We have to use the -0.75 constant when aa is False so that this uint8
1410:     // path is as close as possible to float results.
1411:     auto fn = antialias ? HelperInterpCubic::aa_filter<double, true> : HelperInterpCubic::aa_filter<double, false>;
1412:     return HelperInterpCubic::_compute_index_ranges_int16_weights(
1413:         input_size, output_size, stride, ndims, reshape_dim,
1414:         align_corners, opt_scale, interp_size, fn, antialias, align_i32);
1415:   }
1416:
1417: };
1418:
1419: struct HelperInterpLanczos : public HelperInterpBase {
1420:
1421:   static constexpr int interp_size = 6;
1422:
1423:   // Taken from
1424:   // https://github.com/python-pillow/Pillow/blob/8004234d879254cc354935ad42fbb51b1700925e/src/libImaging/Resample.c#L64-L86
1425:   template<typename scalar_t>
1426:   static inline scalar_t sinc_filter(scalar_t x) {
1427:     if (x == 0.0) {
1428:       return 1.0;
1429:     }
1430:     x *= c10::pi<scalar_t>;
1431:     return std::sin(x) / x;
1432:   }
1433:
1434:   template<typename scalar_t>
1435:   static inline scalar_t aa_filter(scalar_t x) {
1436:     // Lanczos-3 filter: sinc(x) * sinc(x/3) for |x| < 3
1437:     x = std::abs(x);
1438:     if (x < 3.0) {
1439:       return sinc_filter(x) * sinc_filter(x / 3.0);
1440:     }
1441:     return 0.0;
1442:   }
1443:
1444:   static inline std::vector<Tensor> compute_index_ranges_weights(
1445:     at::ScalarType scalar_type,
1446:     int64_t input_size,
1447:     int64_t output_size,
1448:     int64_t stride,
1449:     int64_t ndims,
1450:     int64_t reshape_dim,
1451:     bool align_corners,
1452:     const std::optional<double>& opt_scale,
1453:     bool antialias
1454:   ) {
```
- EN: The main symbol in this range is `sinc_filter`, `aa_filter`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `sinc_filter`, `aa_filter`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1456-1515
```cpp
1456:     std::vector<Tensor> indices_weights;
1457:     AT_DISPATCH_FLOATING_TYPES(
1458:       scalar_type, "compute_index_ranges_weights", [&] {
1459:
1460:         scalar_t scale = area_pixel_compute_scale<scalar_t>(
1461:             input_size, output_size, align_corners, opt_scale);
1462:
1463:         auto interp_size = HelperInterpLanczos::interp_size;
1464:
1465:         indices_weights = std::get<0>(HelperInterpLanczos::_compute_index_ranges_weights<scalar_t>(
1466:             input_size,
1467:             output_size,
1468:             stride,
1469:             ndims,
1470:             reshape_dim,
1471:             scale,
1472:             interp_size,
1473:             &HelperInterpLanczos::aa_filter<scalar_t>,
1474:             /*antialias=*/antialias,
1475:             /*align_corners=*/align_corners));
1476:       }
1477:     );
1478:     return indices_weights;
1479:   }
1480:
1481:   static inline std::tuple<std::vector<Tensor>, int, unsigned int> compute_index_ranges_int16_weights(
1482:     int64_t input_size,
1483:     int64_t output_size,
1484:     int64_t stride,
1485:     int64_t ndims,
1486:     int64_t reshape_dim,
1487:     bool align_corners,
1488:     const std::optional<double>& opt_scale,
1489:     bool antialias,
1490:     bool align_i32=false
1491:   ) {
1492:
1493:     auto interp_size = HelperInterpLanczos::interp_size;
1494:     auto fn = HelperInterpLanczos::aa_filter<double>;
1495:     return HelperInterpLanczos::_compute_index_ranges_int16_weights(
1496:         input_size, output_size, stride, ndims, reshape_dim,
1497:         align_corners, opt_scale, interp_size, fn, antialias, align_i32);
1498:   }
1499:
1500: };
1501:
1502: // Generic upsampling interpolation kernel for N-d case.
1503: // Input is assumed to be like NCHW, NCL, NCKHW - interpolated spatial dimension
1504: // are those from the end up to batch size N and number of channels C.
1505: //
1506: // Internally, it uses TensorIterator to optimize the computations.
1507: // - out_ndims is the number of interpolated dims: 1, 2, 3
1508: // - scale_type is template type for scales, typically std::optional<double>
1509: // - template<typename> class F is one of the above structs to compute indices and weights
1510: template <int out_ndims, typename scale_type, class F>
1511: void upsample_non_separable_Nd_kernel_impl(
1512:     const Tensor& output,
1513:     const Tensor& input,
1514:     bool align_corners,
1515:     const scale_type& scales) {
```
- EN: The main symbol in this range is `compute_index_ranges_int16_weights`, `upsample_non_separable_Nd_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `compute_index_ranges_int16_weights`, `upsample_non_separable_Nd_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1518-1565
```cpp
1518:   // input can be NCHW, NCL or NCKHW
1519:   auto shape = input.sizes().vec();
1520:   auto strides = input.strides().vec();
1521:   auto oshape = output.sizes();
1522:
1523:   TORCH_INTERNAL_ASSERT(
1524:     shape.size() == oshape.size() && shape.size() == 2 + out_ndims
1525:   );
1526:   TORCH_INTERNAL_ASSERT(strides.size() == 2 + out_ndims);
1527:
1528:   for (const auto i : c10::irange(out_ndims)) {
1529:     shape[i + 2] = oshape[i + 2];
1530:     strides[i + 2] = 0;
1531:   }
1532:   auto restrided_input = input.as_strided(shape, strides);
1533:
1534:
1535:   constexpr int interp_size = F::interp_size;
1536:   auto input_scalar_type = input.scalar_type();
1537:   if ((interp_size == 1 && input_scalar_type == at::ScalarType::Byte)) {
1538:     // nearest also supports uint8 tensor, but we have to use float
1539:     // with compute_indices_weights
1540:     input_scalar_type = at::ScalarType::Float;
1541:   }
1542:
1543:   std::vector<std::vector<Tensor>> indices_weights;
1544:   indices_weights.reserve(out_ndims);
1545:   for (const auto i : c10::irange(out_ndims)) {
1546:     indices_weights.emplace_back(
1547:       F::compute_indices_weights(
1548:         input_scalar_type, input.size(i + 2), oshape[i + 2],
1549:         input.stride(i + 2) * input.element_size(),
1550:         input.dim(), i + 2, align_corners, scales[i]
1551:       )
1552:     );
1553:   }
1554:
1555:   TensorIteratorConfig config;
1556:   config.check_all_same_dtype(false)
1557:     .declare_static_dtype_and_device(input.scalar_type(), input.device())
1558:     .add_output(output)
1559:     .add_const_input(restrided_input);
1560:
1561:   for (auto & idx_weight: indices_weights) {
1562:     for (auto& tensor : idx_weight) {
1563:       config.add_const_input(tensor);
1564:     }
1565:   }
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1567-1614
```cpp
1567:   auto iter = config.build();
1568:
1569:   if (interp_size > 1) {
1570:     // Nearest also supports uint8 tensor, so need to handle it separately
1571:     // Dispatch name should be "upsample_non_separable" but we keep the old
1572:     // name for internal BC.
1573:     AT_DISPATCH_FLOATING_TYPES_AND2(
1574:         kBFloat16, kHalf, iter.dtype(), "upsample_generic_Nd", [&] {
1575:         // MSVC can not catch constexpr int interp_size here
1576:         constexpr int mode = F::interp_size;
1577:         upsample_non_separable<scalar_t, out_ndims, mode>(iter);
1578:     });
1579:   } else {
1580:     // Dispatch name should be "upsample_non_separable" but we keep the old
1581:     // name for internal BC.
1582:     AT_DISPATCH_FLOATING_TYPES_AND3(kByte, kBFloat16, kHalf,
1583:         iter.dtype(), "upsample_generic_Nd", [&] {
1584:         constexpr int mode = F::interp_size;
1585:         upsample_non_separable<scalar_t, out_ndims, mode>(iter);
1586:     });
1587:   }
1588: }
1589:
1590: template <int out_ndims, typename scale_type, class F, bool is_horizontal>
1591: void upsample_separable_1d(
1592:     const Tensor& output,
1593:     const Tensor& input,
1594:     int interp_dim,
1595:     bool align_corners,
1596:     const scale_type& scales,
1597:     bool antialias) {
1598:
1599:   // input can be NCHW, NCL or NCKHW
1600:   auto shape = input.sizes().vec();
1601:   auto strides = input.strides().vec();
1602:   auto oshape = output.sizes();
1603:
1604:   TORCH_INTERNAL_ASSERT(
1605:       shape.size() == oshape.size() && shape.size() == 2 + out_ndims);
1606:   TORCH_INTERNAL_ASSERT(strides.size() == 2 + out_ndims);
1607:
1608:   for (const auto i : c10::irange(out_ndims)) {
1609:     shape[i + 2] = oshape[i + 2];
1610:   }
1611:   strides[interp_dim] = 0;
1612:   auto restrided_input = input.as_strided(shape, strides);
1613:
1614:   auto input_scalar_type = input.scalar_type();
```
- EN: The main symbol in this range is `upsample_separable_1d`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `upsample_separable_1d`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1616-1668
```cpp
1616:   std::vector<Tensor> indices_weights;
1617:   unsigned int weights_precision = 0;
1618:
1619:   if (input_scalar_type == at::kByte) {
1620:     // This is a special branch to provide uint8 dtype support for bilinear, bicubic and lanczos modes only
1621:     TORCH_INTERNAL_ASSERT(F::interp_size == 2 || F::interp_size == 4 || F::interp_size == 6);
1622:     int unused = 0;
1623:     std::tie(indices_weights, unused, weights_precision) =
1624:       F::compute_index_ranges_int16_weights(
1625:         input.size(interp_dim), oshape[interp_dim],
1626:         input.stride(interp_dim) * input.element_size(),
1627:         input.dim(), interp_dim, align_corners, scales[interp_dim - 2],
1628:         antialias);
1629:     TORCH_INTERNAL_ASSERT(weights_precision > 0);
1630:   } else {
1631:     indices_weights =
1632:       F::compute_index_ranges_weights(
1633:         input_scalar_type, input.size(interp_dim), oshape[interp_dim],
1634:         input.stride(interp_dim) * input.element_size(),
1635:         input.dim(), interp_dim, align_corners, scales[interp_dim - 2],
1636:         antialias);
1637:   }
1638:
1639:   TensorIteratorConfig config;
1640:   config.check_all_same_dtype(false)
1641:       .declare_static_dtype_and_device(input.scalar_type(), input.device())
1642:       .add_output(output)
1643:       .add_const_input(restrided_input);
1644:
1645:   for (auto& tensor : indices_weights) {
1646:     config.add_const_input(tensor);
1647:   }
1648:
1649:   auto iter = config.build();
1650:
1651:   // Dispatch name should be "upsample_separable_1d" but we keep the old
1652:   // name for internal BC.
1653:   AT_DISPATCH_FLOATING_TYPES_AND(
1654:       at::ScalarType::Byte, iter.dtype(), "upsample_generic_Nd_aa", [&] {
1655:         auto loop = [&](char** data, const int64_t* strides, int64_t n) {
1656:           if constexpr (is_horizontal) {
1657:             // Strides are : X 0 | 8 8 8 0 8  (Channels first)
1658:             // Strides are : X X | 0 0 0 0 0  (Channels last)
1659:             basic_loop_separable_1d_horizontal<scalar_t>(data, strides, n, weights_precision);
1660:           } else {
1661:             // Strides are : X Y | 0 0 0 0 0 (Channels first)
1662:             // Strides are : X X | 0 0 0 0 0 (Channels last)
1663:             basic_loop_separable_1d_vertical<scalar_t>(data, strides, n, weights_precision);
1664:           }
1665:         };
1666:         iter.for_each(loop);
1667:       });
1668: }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1670-1723
```cpp
1670: // Generic separable upsampling interpolation kernel for N-d case with anti-aliasing.
1671: // It also supports antialias=False iff
1672: // (dtype == uint8 and mode in ("bilinear", "bicubic")): this is used as
1673: // fallback in these settings when AVX isn't supported.
1674: template <int out_ndims, typename scale_type, class F>
1675: void upsample_separable_Nd_kernel_impl(
1676:     const Tensor& output,
1677:     const Tensor& input,
1678:     bool align_corners,
1679:     const scale_type& scales,
1680:     bool antialias) {
1681:
1682:   auto output_shape = output.sizes();
1683:   auto input_shape = input.sizes();
1684:   auto temp_oshape = input_shape.vec();
1685:
1686:   if (output_shape == input_shape) {
1687:     output.copy_(input);
1688:     return;
1689:   }
1690:
1691:   at::Tensor temp_output, temp_input = input;
1692:
1693:   int interp_dim = 0;
1694:   // Precompute the number of 1d resize ops
1695:   // to avoid copying temporary buffer to output
1696:   int num_1d_ops = 0;
1697:   for (const auto i : c10::irange(out_ndims)) {
1698:     interp_dim = 2 + out_ndims - 1 - i;
1699:     if (output_shape[interp_dim] != input_shape[interp_dim]) {
1700:       num_1d_ops += 1;
1701:     }
1702:   }
1703:
1704:   // Horizontal resampling (last spatial dim, i.e. W)
1705:   interp_dim = 2 + out_ndims - 1;
1706:   if (output_shape[interp_dim] != input_shape[interp_dim]) {
1707:
1708:     num_1d_ops -= 1;
1709:     if (num_1d_ops > 0) {
1710:       temp_oshape[interp_dim] = output_shape[interp_dim];
1711:       temp_output = at::empty(temp_oshape, input.options());
1712:     } else {
1713:       temp_output = output;
1714:     }
1715:
1716:     upsample_separable_1d<
1717:         out_ndims,
1718:         scale_t,
1719:         F,
1720:         true>(
1721:         temp_output, temp_input, interp_dim, align_corners, scales, antialias);
1722:     temp_input = temp_output;
1723:   }
```
- EN: The main symbol in this range is `in`, `upsample_separable_Nd_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `in`, `upsample_separable_Nd_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1725-1779
```cpp
1725:   // Vertical resampling (remaining spatial dims, e.g. H, D)
1726:   for (const auto i : c10::irange(1, out_ndims)) {
1727:     interp_dim = 2 + out_ndims - 1 - i;
1728:     if (output_shape[interp_dim] != input_shape[interp_dim]) {
1729:
1730:       num_1d_ops -= 1;
1731:       if (num_1d_ops > 0) {
1732:         temp_oshape[interp_dim] = output_shape[interp_dim];
1733:         temp_output = at::empty(temp_oshape, input.options());
1734:       } else {
1735:         temp_output = output;
1736:       }
1737:
1738:       upsample_separable_1d<
1739:           out_ndims,
1740:           scale_t,
1741:           F,
1742:           false>(
1743:           temp_output, temp_input, interp_dim, align_corners, scales, antialias);
1744:       temp_input = temp_output;
1745:     }
1746:   }
1747: }
1748:
1749: void upsample_nearest1d_kernel_impl(
1750:     const Tensor& output,
1751:     const Tensor& input,
1752:     std::optional<double> scales_w) {
1753:   upsample_non_separable_Nd_kernel_impl<1, scale_t, HelperInterpNearest>(
1754:       output, input, false, {scales_w});
1755: }
1756:
1757: void _upsample_nearest_exact1d_kernel_impl(
1758:     const Tensor& output,
1759:     const Tensor& input,
1760:     std::optional<double> scales_w) {
1761:   upsample_non_separable_Nd_kernel_impl<1, scale_t, HelperInterpNearestExact>(
1762:     output, input, false, {scales_w});
1763: }
1764:
1765: int _use_channels_last_kernel_2d(
1766:     const Tensor& output,
1767:     const Tensor& input) {
1768:       // This condition is used to know whether we should dispatch to a
1769:       // channels-last-optimized kernel, or to the more general
1770:       // upsample_non_separable_Nd_kernel_impl(). For now, the channels-last kernels
1771:       // are only optimized for channels_last and when C >= 4 (shape = NCHW).
1772:       // For a very wide range of use-cases (typically image or mask resizing
1773:       // where we have C < 4), using upsample_non_separable_Nd_kernel_impl() is
1774:       // actually faster. On top of that, benchmarks showed that this also
1775:       // depends on the *output* size (output_H + output_W), for both
1776:       // upsampling and downsampling. The current 128 threshold was determined
1777:       // through benchmarks.
1778:       return ((input.is_contiguous(at::MemoryFormat::ChannelsLast)) && (input.size(1) > 3)) || ((output.size(-2) + output.size(-1)) <= 128);
1779: }
```
- EN: The main symbol in this range is `resampling`, `upsample_nearest1d_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `resampling`, `upsample_nearest1d_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1781-1837
```cpp
1781: int _use_channels_last_kernel_3d(
1782:     // Similar to _use_channels_last_kernel_2d() but for 3d resampling (e.g. videos)
1783:     // Note that unlike the 2d case, this is not subject to small output size
1784:     // overhead - hence the absence of the 128 threshold in the condition.
1785:     const Tensor& output,
1786:     const Tensor& input) {
1787:       return ((input.is_contiguous(at::MemoryFormat::ChannelsLast3d)) && (input.size(1) > 3));
1788: }
1789:
1790:
1791: void upsample_nearest2d_kernel_impl(
1792:     const Tensor& output,
1793:     const Tensor& input,
1794:     std::optional<double> scales_h,
1795:     std::optional<double> scales_w) {
1796:   if (_use_channels_last_kernel_2d(output, input)) {
1797:     AT_DISPATCH_FLOATING_TYPES_AND3(kByte, kBFloat16, kHalf,
1798:         input.scalar_type(), "upsample_nearest2d_channels_last", [&] {
1799:       upsample_nearest_channels_last<scalar_t, scale_t, nearest_idx>(output, input, {scales_h, scales_w});
1800:     });
1801:   } else {
1802:     upsample_non_separable_Nd_kernel_impl<2, scale_t, HelperInterpNearest>(
1803:       output, input, false, {scales_h, scales_w});
1804:   }
1805: }
1806:
1807: void _upsample_nearest_exact2d_kernel_impl(
1808:     const Tensor& output,
1809:     const Tensor& input,
1810:     std::optional<double> scales_h,
1811:     std::optional<double> scales_w) {
1812:   if (_use_channels_last_kernel_2d(output, input)) {
1813:     AT_DISPATCH_FLOATING_TYPES_AND3(kByte, kBFloat16, kHalf, input.scalar_type(), "upsample_nearest2d_channels_last", [&] {
1814:       upsample_nearest_channels_last<scalar_t, scale_t, nearest_exact_idx>(output, input, {scales_h, scales_w});
1815:     });
1816:   } else {
1817:     upsample_non_separable_Nd_kernel_impl<2, scale_t, HelperInterpNearestExact>(
1818:       output, input, false, {scales_h, scales_w});
1819:   }
1820: }
1821:
1822: void upsample_nearest3d_kernel_impl(
1823:     const Tensor& output,
1824:     const Tensor& input,
1825:     std::optional<double> scales_d,
1826:     std::optional<double> scales_h,
1827:     std::optional<double> scales_w) {
1828:   if (_use_channels_last_kernel_3d(output, input)) {
1829:     AT_DISPATCH_FLOATING_TYPES_AND3(kByte, kBFloat16, kHalf,
1830:         input.scalar_type(), "upsample_nearest3d_channels_last", [&] {
1831:       upsample_nearest_channels_last<scalar_t, scale_t, nearest_idx>(output, input, {scales_d, scales_h, scales_w});
1832:     });
1833:   } else {
1834:     upsample_non_separable_Nd_kernel_impl<3, scale_t, HelperInterpNearest>(
1835:       output, input, false, {scales_d, scales_h, scales_w});
1836:   }
1837: }
```
- EN: The main symbol in this range is `_use_channels_last_kernel_3d`, `upsample_nearest2d_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `_use_channels_last_kernel_3d`, `upsample_nearest2d_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1839-1885
```cpp
1839: void _upsample_nearest_exact3d_kernel_impl(
1840:     const Tensor& output,
1841:     const Tensor& input,
1842:     std::optional<double> scales_d,
1843:     std::optional<double> scales_h,
1844:     std::optional<double> scales_w) {
1845:   if (_use_channels_last_kernel_3d(output, input)) {
1846:     AT_DISPATCH_FLOATING_TYPES_AND3(kByte, kBFloat16, kHalf, input.scalar_type(), "upsample_nearest3d_channels_last", [&] {
1847:       upsample_nearest_channels_last<scalar_t, scale_t, nearest_exact_idx>(output, input, {scales_d, scales_h, scales_w});
1848:     });
1849:   } else {
1850:     upsample_non_separable_Nd_kernel_impl<3, scale_t, HelperInterpNearestExact>(
1851:       output, input, false, {scales_d, scales_h, scales_w});
1852:   }
1853: }
1854:
1855: void upsample_linear1d_kernel_impl(
1856:     const Tensor& output,
1857:     const Tensor& input,
1858:     bool align_corners,
1859:     std::optional<double> scales_w) {
1860:   upsample_non_separable_Nd_kernel_impl<1, scale_t, HelperInterpLinear>(
1861:     output, input, align_corners, {scales_w});
1862: }
1863:
1864:
1865: void upsample_bilinear2d_kernel_impl_float(
1866:     const Tensor& output,
1867:     const Tensor& input,
1868:     bool align_corners,
1869:     std::optional<double> scales_h,
1870:     std::optional<double> scales_w) {
1871:
1872:   // See note above about _use_channels_last_kernel_2d(output, input). The extra cond is present
1873:   // because benchmarks showed that with only 1 thread, images (C == 3) were
1874:   // slightly faster with the channels-last kernel than with the generic one.
1875:   // That's not the case for masks though (C == 1), which strongly benefit from
1876:   // using the generic kernel.
1877:   if ((_use_channels_last_kernel_2d(output, input)) || (at::get_num_threads() == 1 && input.size(1) == 3)) {
1878:     AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, input.scalar_type(), "upsample_bilinear2d_channels_last", [&] {
1879:       upsample_linear_channels_last<scalar_t, scale_t>(output, input, align_corners, {scales_h, scales_w});
1880:     });
1881:   } else {
1882:     upsample_non_separable_Nd_kernel_impl<2, scale_t, HelperInterpLinear>(
1883:       output, input, align_corners, {scales_h, scales_w});
1884:   }
1885: }
```
- EN: The main symbol in this range is `_upsample_nearest_exact3d_kernel_impl`, `upsample_linear1d_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `_upsample_nearest_exact3d_kernel_impl`, `upsample_linear1d_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1887-1944
```cpp
1887: void upsample_bilinear2d_kernel_impl(
1888:     const Tensor& output,
1889:     const Tensor& input,
1890:     bool align_corners,
1891:     std::optional<double> scales_h,
1892:     std::optional<double> scales_w) {
1893:
1894:   if (input.dtype() == at::kByte) {
1895:     #ifdef CPU_CAPABILITY_AVX2
1896:       if (input.size(1) <= 4) {
1897:         return upsample_avx_bilinear_bicubic_uint8<scale_t, HelperInterpLinear>(input,
1898:           output, align_corners, {scales_h, scales_w},
1899:           /*antialias=*/false);
1900:       }
1901:     #elif defined(__aarch64__)
1902:       if (input.size(1) == 3
1903:           && input.is_contiguous(at::MemoryFormat::ChannelsLast)
1904:           && output.is_contiguous(at::MemoryFormat::ChannelsLast)) {
1905:         return upsample_neon_bilinear_bicubic_uint8<scale_t, HelperInterpLinear>(
1906:           input, output, align_corners, {scales_h, scales_w},
1907:           /*antialias=*/false);
1908:       }
1909:     #endif  // CPU_CAPABILITY_AVX2
1910:     return upsample_separable_Nd_kernel_impl<2, scale_t, HelperInterpLinear>(
1911:       output, input, align_corners, {scales_h, scales_w},
1912:       /*antialias=*/false);
1913:   }
1914:   return upsample_bilinear2d_kernel_impl_float(output, input, align_corners, scales_h, scales_w);
1915: }
1916:
1917:
1918: void upsample_bilinear2d_aa_kernel_impl(
1919:     const Tensor& output,
1920:     const Tensor& input,
1921:     bool align_corners,
1922:     std::optional<double> scales_h,
1923:     std::optional<double> scales_w) {
1924:   if (input.dtype() == at::kByte) {
1925:     #ifdef CPU_CAPABILITY_AVX2
1926:       if (input.size(1) <= 4) {
1927:         return upsample_avx_bilinear_bicubic_uint8<scale_t, HelperInterpLinear>(
1928:           input, output, align_corners, {scales_h, scales_w},
1929:           /*antialias=*/true);
1930:       }
1931:     #elif defined(__aarch64__)
1932:       if (input.size(1) == 3
1933:           && input.is_contiguous(at::MemoryFormat::ChannelsLast)
1934:           && output.is_contiguous(at::MemoryFormat::ChannelsLast)) {
1935:         return upsample_neon_bilinear_bicubic_uint8<scale_t, HelperInterpLinear>(
1936:           input, output, align_corners, {scales_h, scales_w},
1937:           /*antialias=*/true);
1938:       }
1939:     #endif  // CPU_CAPABILITY_AVX2
1940:   }
1941:   return upsample_separable_Nd_kernel_impl<2, scale_t, HelperInterpLinear>(
1942:     output, input, align_corners, {scales_h, scales_w},
1943:     /*antialias=*/true);
1944: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `upsample_bilinear2d_kernel_impl`, `upsample_bilinear2d_aa_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `upsample_bilinear2d_kernel_impl`, `upsample_bilinear2d_aa_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1946-1992
```cpp
1946: void upsample_trilinear3d_kernel_impl(
1947:     const Tensor& output,
1948:     const Tensor& input,
1949:     bool align_corners,
1950:     std::optional<double> scales_d,
1951:     std::optional<double> scales_h,
1952:     std::optional<double> scales_w) {
1953:   if ((_use_channels_last_kernel_3d(output, input))) {
1954:     AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, input.scalar_type(), "upsample_trilinear3d_channels_last", [&] {
1955:       upsample_linear_channels_last<scalar_t, scale_t>(output, input, align_corners, {scales_d, scales_h, scales_w});
1956:     });
1957:   } else {
1958:     upsample_non_separable_Nd_kernel_impl<3, scale_t, HelperInterpLinear>(
1959:       output, input, align_corners, {scales_d, scales_h, scales_w});
1960:   }
1961: }
1962:
1963: void upsample_bicubic2d_kernel_impl(
1964:     const Tensor& output,
1965:     const Tensor& input,
1966:     bool align_corners,
1967:     std::optional<double> scales_h,
1968:     std::optional<double> scales_w) {
1969:
1970:   if (input.dtype() == at::kByte) {
1971:     #ifdef CPU_CAPABILITY_AVX2
1972:       if (input.size(1) <= 4) {
1973:         return upsample_avx_bilinear_bicubic_uint8<scale_t, HelperInterpCubic>(input,
1974:           output, align_corners, {scales_h, scales_w},
1975:           /*antialias=*/false);
1976:       }
1977:     #elif defined(__aarch64__)
1978:       if (input.size(1) == 3
1979:           && input.is_contiguous(at::MemoryFormat::ChannelsLast)
1980:           && output.is_contiguous(at::MemoryFormat::ChannelsLast)) {
1981:         return upsample_neon_bilinear_bicubic_uint8<scale_t, HelperInterpCubic>(
1982:           input, output, align_corners, {scales_h, scales_w},
1983:           /*antialias=*/false);
1984:       }
1985:     #endif  // CPU_CAPABILITY_AVX2
1986:     return upsample_separable_Nd_kernel_impl<2, scale_t, HelperInterpCubic>(
1987:       output, input, align_corners, {scales_h, scales_w},
1988:       /*antialias=*/false);
1989:   }
1990:   return upsample_non_separable_Nd_kernel_impl<2, scale_t, HelperInterpCubic>(
1991:     output, input, align_corners, {scales_h, scales_w});
1992: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `upsample_trilinear3d_kernel_impl`, `upsample_bicubic2d_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `upsample_trilinear3d_kernel_impl`, `upsample_bicubic2d_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1994-2050
```cpp
1994: void upsample_bicubic2d_aa_kernel_impl(
1995:     const Tensor& output,
1996:     const Tensor& input,
1997:     bool align_corners,
1998:     std::optional<double> scales_h,
1999:     std::optional<double> scales_w) {
2000:
2001:   if (input.dtype() == at::kByte) {
2002:     #ifdef CPU_CAPABILITY_AVX2
2003:       if (input.size(1) <= 4) {
2004:         return upsample_avx_bilinear_bicubic_uint8<scale_t, HelperInterpCubic>(
2005:           input, output, align_corners, {scales_h, scales_w},
2006:           /*antialias=*/true);
2007:       }
2008:     #elif defined(__aarch64__)
2009:       if (input.size(1) == 3
2010:           && input.is_contiguous(at::MemoryFormat::ChannelsLast)
2011:           && output.is_contiguous(at::MemoryFormat::ChannelsLast)) {
2012:         return upsample_neon_bilinear_bicubic_uint8<scale_t, HelperInterpCubic>(
2013:           input, output, align_corners, {scales_h, scales_w},
2014:           /*antialias=*/true);
2015:       }
2016:     #endif  // CPU_CAPABILITY_AVX2
2017:   }
2018:   return upsample_separable_Nd_kernel_impl<2, scale_t, HelperInterpCubic>(
2019:     output, input, align_corners, {scales_h, scales_w},
2020:     /*antialias=*/true);
2021: }
2022:
2023: void upsample_lanczos2d_aa_kernel_impl(
2024:     const Tensor& output,
2025:     const Tensor& input,
2026:     bool align_corners,
2027:     std::optional<double> scales_h,
2028:     std::optional<double> scales_w) {
2029:
2030:   if (input.dtype() == at::kByte) {
2031:     #ifdef CPU_CAPABILITY_AVX2
2032:       if (input.size(1) <= 4) {
2033:         return upsample_avx_bilinear_bicubic_uint8<scale_t, HelperInterpLanczos>(
2034:           input, output, align_corners, {scales_h, scales_w},
2035:           /*antialias=*/true);
2036:       }
2037:     #elif defined(__aarch64__)
2038:       if (input.size(1) == 3
2039:           && input.is_contiguous(at::MemoryFormat::ChannelsLast)
2040:           && output.is_contiguous(at::MemoryFormat::ChannelsLast)) {
2041:         return upsample_neon_bilinear_bicubic_uint8<scale_t, HelperInterpLanczos>(
2042:           input, output, align_corners, {scales_h, scales_w},
2043:           /*antialias=*/true);
2044:       }
2045:     #endif  // CPU_CAPABILITY_AVX2
2046:   }
2047:   return upsample_separable_Nd_kernel_impl<2, scale_t, HelperInterpLanczos>(
2048:     output, input, align_corners, {scales_h, scales_w},
2049:     /*antialias=*/true);
2050: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `upsample_bicubic2d_aa_kernel_impl`, `upsample_lanczos2d_aa_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `upsample_bicubic2d_aa_kernel_impl`, `upsample_lanczos2d_aa_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 2052-2100
```cpp
2052: template <
2053:     typename scalar_t,
2054:     typename scale_type,
2055:     class F>
2056: void upsample_separable_Nd_backward_aa(
2057:     const Tensor& grad_input_,
2058:     const Tensor& grad_output_,
2059:     bool align_corners,
2060:     const scale_type& scales) {
2061:   TORCH_CHECK(grad_input_.dtype() == grad_output_.dtype(), "expected dtype ", grad_output_.dtype(),
2062:               " for `grad_input` but got dtype ", grad_input_.dtype());
2063:
2064:   auto grad_output = grad_output_.contiguous();
2065:   auto grad_input = grad_input_.contiguous();
2066:
2067:   auto grad_output_data = grad_output.const_data_ptr<scalar_t>();
2068:   auto grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
2069:   auto input_sizes = grad_input.sizes().vec();
2070:   auto output_sizes = grad_output.sizes().vec();
2071:   auto ndim = input_sizes.size();
2072:
2073:   // treat nbatch and channels as one dimension
2074:   int64_t channels = input_sizes[0] * input_sizes[1];
2075:   int64_t output_depth = (ndim == 5) ? output_sizes[2] : 1;
2076:   int64_t input_height = (ndim >= 4) ? input_sizes[ndim - 2] : 1;
2077:   int64_t output_height = (ndim >= 4) ? output_sizes[ndim - 2] : 1;
2078:   int64_t input_width = input_sizes[ndim - 1];
2079:   int64_t output_width = output_sizes[ndim - 1];
2080:
2081:   int64_t output_slice_size = output_depth * output_height * output_width;
2082:   int interp_size = F::interp_size;
2083:
2084:   auto loop2d = [&](int64_t begin, int64_t end) {
2085:     const scalar_t height_scale = area_pixel_compute_scale<scalar_t>(
2086:         input_height, output_height, align_corners, scales[0]);
2087:     const scalar_t width_scale = area_pixel_compute_scale<scalar_t>(
2088:         input_width, output_width, align_corners, scales[1]);
2089:
2090:     auto input_indexr = [=](int64_t c, int64_t h, int64_t w) {
2091:       return grad_input_data + c * input_height * input_width +
2092:           h * input_width + w;
2093:     };
2094:
2095:     const scalar_t support_h = (height_scale >= 1.0)
2096:         ? (interp_size * 0.5) * height_scale
2097:         : interp_size * 0.5;
2098:     const scalar_t support_w = (width_scale >= 1.0)
2099:         ? (interp_size * 0.5) * width_scale
2100:         : interp_size * 0.5;
```
- EN: The main symbol in this range is `upsample_separable_Nd_backward_aa`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `upsample_separable_Nd_backward_aa`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 2102-2151
```cpp
2102:     const int interp_height = (int)ceilf(support_h) * 2 + 1;
2103:     const int interp_width = (int)ceilf(support_w) * 2 + 1;
2104:
2105:     std::vector<scalar_t> wx(interp_width, 0.0);
2106:     std::vector<scalar_t> wy(interp_height, 0.0);
2107:
2108:     int64_t xmin = 0, ymin = 0;
2109:     int64_t xsize = 0, ysize = 0;
2110:
2111:     typedef scalar_t (*aa_filter_fn_t)(scalar_t);
2112:     aa_filter_fn_t filter_fn = &F::aa_filter;
2113:
2114:     for (const auto oh : c10::irange(output_height)) {
2115:       F::_compute_indices_min_size_weights_aa(
2116:           oh,
2117:           input_height,
2118:           height_scale,
2119:           support_h,
2120:           wy.data(),
2121:           interp_height,
2122:           filter_fn,
2123:           ymin,
2124:           ysize);
2125:
2126:       for (const auto ow : c10::irange(output_width)) {
2127:         F::_compute_indices_min_size_weights_aa(
2128:             ow,
2129:             input_width,
2130:             width_scale,
2131:             support_w,
2132:             wx.data(),
2133:             interp_width,
2134:             filter_fn,
2135:             xmin,
2136:             xsize);
2137:
2138:         for (const auto c : c10::irange(begin, end)) {
2139:           scalar_t grad_output_value =
2140:               grad_output_data[c * output_slice_size + oh * output_width + ow];
2141:
2142:           for (const auto y : c10::irange(ysize)) {
2143:             for (const auto x : c10::irange(xsize)) {
2144:               *input_indexr(c, ymin + y, xmin + x) +=
2145:                   wx[x] * wy[y] * grad_output_value;
2146:             }
2147:           }
2148:         }
2149:       }
2150:     }
2151:   };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 2153-2203
```cpp
2153:   if (ndim == 4) {
2154:     // upsample bilinear 2d
2155:     at::parallel_for(
2156:         0, channels, at::internal::GRAIN_SIZE / output_slice_size / 4, loop2d);
2157:   } else {
2158:     TORCH_CHECK(false, "Unsupported tensor ndim");
2159:   }
2160:
2161:   if (!grad_input_.is_contiguous()) {
2162:     grad_input_.copy_(grad_input);
2163:   }
2164: }
2165:
2166: void upsample_bilinear2d_aa_backward_kernel_impl(
2167:     const Tensor& grad_input,
2168:     const Tensor& grad_output,
2169:     bool align_corners,
2170:     std::optional<double> scales_h,
2171:     std::optional<double> scales_w) {
2172:   AT_DISPATCH_FLOATING_TYPES(
2173:       grad_output.scalar_type(), "upsample_bilinear2d_aa_backward_cpu", [&] {
2174:         upsample_separable_Nd_backward_aa<scalar_t, scale_t, HelperInterpLinear>(
2175:             grad_input, grad_output, align_corners, {scales_h, scales_w});
2176:       });
2177: }
2178:
2179: void upsample_bicubic2d_aa_backward_kernel_impl(
2180:     const Tensor& grad_input,
2181:     const Tensor& grad_output,
2182:     bool align_corners,
2183:     std::optional<double> scales_h,
2184:     std::optional<double> scales_w) {
2185:   AT_DISPATCH_FLOATING_TYPES(
2186:       grad_output.scalar_type(), "upsample_bicubic2d_aa_backward_cpu", [&] {
2187:         upsample_separable_Nd_backward_aa<scalar_t, scale_t, HelperInterpCubic>(
2188:             grad_input, grad_output, align_corners, {scales_h, scales_w});
2189:       });
2190: }
2191:
2192: void upsample_lanczos2d_aa_backward_kernel_impl(
2193:     const Tensor& grad_input,
2194:     const Tensor& grad_output,
2195:     bool align_corners,
2196:     std::optional<double> scales_h,
2197:     std::optional<double> scales_w) {
2198:   AT_DISPATCH_FLOATING_TYPES(
2199:       grad_output.scalar_type(), "upsample_lanczos2d_aa_backward_cpu", [&] {
2200:         upsample_separable_Nd_backward_aa<scalar_t, scale_t, HelperInterpLanczos>(
2201:             grad_input, grad_output, align_corners, {scales_h, scales_w});
2202:       });
2203: }
```
- EN: The main symbol in this range is `upsample_bilinear2d_aa_backward_kernel_impl`, `upsample_bicubic2d_aa_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 这一段的主要符号是 `upsample_bilinear2d_aa_backward_kernel_impl`, `upsample_bicubic2d_aa_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 2205-2226
```cpp
2205: } // anonymous namespace
2206:
2207: REGISTER_DISPATCH(upsample_nearest1d_kernel, &upsample_nearest1d_kernel_impl)
2208: REGISTER_DISPATCH(_upsample_nearest_exact1d_kernel, &_upsample_nearest_exact1d_kernel_impl)
2209: REGISTER_DISPATCH(upsample_nearest2d_kernel, &upsample_nearest2d_kernel_impl)
2210: REGISTER_DISPATCH(_upsample_nearest_exact2d_kernel, &_upsample_nearest_exact2d_kernel_impl)
2211: REGISTER_DISPATCH(upsample_nearest3d_kernel, &upsample_nearest3d_kernel_impl)
2212: REGISTER_DISPATCH(_upsample_nearest_exact3d_kernel, &_upsample_nearest_exact3d_kernel_impl)
2213:
2214: REGISTER_DISPATCH(upsample_linear1d_kernel, &upsample_linear1d_kernel_impl)
2215: REGISTER_DISPATCH(upsample_bilinear2d_kernel, &upsample_bilinear2d_kernel_impl)
2216: REGISTER_DISPATCH(_upsample_bilinear2d_aa_kernel, &upsample_bilinear2d_aa_kernel_impl)
2217: REGISTER_DISPATCH(_upsample_bilinear2d_aa_backward_kernel, &upsample_bilinear2d_aa_backward_kernel_impl)
2218: REGISTER_DISPATCH(upsample_trilinear3d_kernel, &upsample_trilinear3d_kernel_impl)
2219:
2220: REGISTER_DISPATCH(upsample_bicubic2d_kernel, &upsample_bicubic2d_kernel_impl)
2221: REGISTER_DISPATCH(_upsample_bicubic2d_aa_kernel, &upsample_bicubic2d_aa_kernel_impl)
2222: REGISTER_DISPATCH(_upsample_bicubic2d_aa_backward_kernel, &upsample_bicubic2d_aa_backward_kernel_impl)
2223:
2224: REGISTER_DISPATCH(_upsample_lanczos2d_aa_kernel, &upsample_lanczos2d_aa_kernel_impl)
2225: REGISTER_DISPATCH(_upsample_lanczos2d_aa_backward_kernel, &upsample_lanczos2d_aa_backward_kernel_impl)
2226: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/TensorIterator.h`, `ATen/cpu/vec/vec.h`, `ATen/native/UpSample.h`, `ATen/native/cpu/utils.h`, `ATen/native/cpu/UpSampleKernelAVXAntialias.h`, `ATen/native/cpu/UpSampleKernelNEONAntialias.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
