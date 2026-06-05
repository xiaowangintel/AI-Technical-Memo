# GridSamplerKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/GridSamplerKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Grid Sampler Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Grid Sampler Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: #define TORCH_ASSERT_NO_OPERATORS
 2: #include <ATen/native/GridSampler.h>
 3: #include <ATen/native/cpu/GridSamplerKernel.h>
 4: #include <ATen/core/TensorBase.h>
 5: #include <ATen/Dispatch.h>
 6: #include <ATen/Parallel.h>
 7: #include <ATen/TensorGeometry.h>
 8: #include <ATen/TensorIterator.h>
 9: #include <ATen/cpu/vec/vec.h>
10: #include <c10/util/irange.h>
11:
12: #include <algorithm>
13: #include <cstring>
14:
15: namespace at::native { namespace {
16:
17: // fixes segfaults for GCC >= 12 on some AArch64 cpus https://github.com/pytorch/pytorch/issues/157626
18: #if defined(__GNUC__) && __GNUC__ >= 12 && defined(__aarch64__)
19: #pragma GCC push_options
20: #pragma GCC optimize ("no-strict-aliasing")
21: #endif
22:
23: /**  NOTE [ Grid Sample CPU Kernels ]
24:  *
25:  *   Implementation of vectorized grid sample CPU kernels is divided into three
26:  *   parts. More detailed description exist after this paragraph, but on a high
27:  *   level, they are
28:  *   1. `ComputeLocation` struct
29:  *      + Computes the interpolation location basing on padding mode.
30:  *   2. `ApplyGridSample` struct
```
- EN: This range pulls in required headers, including `ATen/native/GridSampler.h`, `ATen/native/cpu/GridSamplerKernel.h`, `ATen/core/TensorBase.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/native/GridSampler.h`, `ATen/native/cpu/GridSamplerKernel.h`, `ATen/core/TensorBase.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 31-60
```cpp
31:  *      + Owns N (# spatial dims) `ComputeLocation` structs, and uses them to
32:  *        compute the interpolation locations.
33:  *      + Interpolates the values and writes to output.
34:  *   3. `grid_sample_2d_grid_slice_iterator` function
35:  *      + Iterates over a slice of the grid tensor based on the geometry by the
36:  *        spatial ordering, i.e., the first iteration will process grid values
37:  *           grid[n, 0, 0, :], grid[n, 0, 1, :], grid[n, 0, 2, :], ...
38:  *        (Recall that, e.g., 2D grid has shape [N x H x W x 2], so grid[n, ...]
39:  *         is a slice, and grid[n, h, w, :] contains the values for a single
40:  *         output spatial location.)
41:  *      + Applies a given operator at each iteration, so we can use the same
42:  *        pattern for forward and backward.
43:  *
44:  *   Putting everything together, we have, e.g., the forward kernel implemented
45:  *   as
46:  *
47:  *      // `ApplyGridSample` struct that processes grid values, extracts and
48:  *      // interpolates input values, and write to output.
49:  *      ApplyGridSample<scalar_t, 2, interp, padding> grid_sample(input_accessor);
50:  *
51:  *      // For each slice, we call `grid_sample_2d_grid_slice_iterator` with
52:  *      //   1. the grid slice, and
53:  *      //   2. a lambda that takes in
54:  *      //      i.   location vectors (x and y for 2D) extracted from grid
55:  *      //      ii.  `spatial_offset` as the spatial offset of these vectors
56:  *      //           from the beginning of this slice.
57:  *      //      iii. `len` as the number of valid locations in the vectors.
58:  *      //           (There might not be enough near boundary.)
59:  *      for (const auto n : c10::irange(input_accessor.size(0))) {
60:  *        grid_sample_2d_grid_slice_iterator(
```
- EN: The main symbol in this range is `vectors`, `that`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `vectors`, `that`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 61-90
```cpp
61:  *          grid_accessor[n],
62:  *          [&](const Vectorized<scalar_t>& grid_x,
63:  *              const Vectorized<scalar_t>& grid_y,
64:  *              int64_t spatial_offset, int64_t len) {
65:  *            grid_sample.forward(out_accessor[n], input_accessor[n],
66:  *                                spatial_offset, grid_x, grid_y, len);
67:  *          });
68:  *      }
69:  *
70:  *   Now we talk about details of each of these three parts:
71:  *
72:  *   1. `ComputeLocation` struct
73:  *      Transforms grid values into interpolation locations of the input tensor
74:  *      for a particular spatial dimension, based on the size of that dimension
75:  *      in input tensor, and the padding mode.
76:  *
77:  *        template<typename scalar_t, GridSamplerPadding padding>
78:  *        struct ComputeLocation {
79:  *          using Vec = Vectorized<scalar_t>;
80:  *
81:  *          // ctor
82:  *          ComputeLocation(int64_t size);
83:  *
84:  *          // Given grid values `in`, return the interpolation locations after
85:  *          // un-normalization and padding mechanism (elementwise).
86:  *          Vec apply(const Vec &in) const;
87:  *
88:  *          // Similar to `apply`, but also returns `d apply(in) / d in`
89:  *          // (elementwise).
90:  *          // this is often used in gradient computation.
```
- EN: The main symbol in this range is `ComputeLocation`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `ComputeLocation`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 91-120
```cpp
 91:  *          std::pair<Vec, Vec> apply_get_grad(const Vec &in) const;
 92:  *        };
 93:  *
 94:  *   2. `ApplyGridSample` struct
 95:  *      Owns N `ComputeLocation` structs, where N is the number of spatial
 96:  *      dimensions. Given N input grid vectors (one for each spatial dimension)
 97:  *      and spatial offset, it gets the interpolation locations from
 98:  *      `ComputeLocation`s, applies interpolation procedure, and then writes to
 99:  *      the output (or grad_input & grad_grid in backward).
100:  *
101:  *        template<typename scalar_t, int spatial_dim,
102:  *                 GridSamplerInterpolation interp,
103:  *                 GridSamplerPadding padding>
104:  *        struct ApplyGridSample {
105:  *
106:  *          // ctor
107:  *          ApplyGridSample(const TensorAccessor<scalar_t, 4>& input);
108:  *
109:  *          // Applies grid sampling (forward) procedure:
110:  *          //   1. computes interpolation locations from grid values `grid_x`
111:  *          //      and `grid_y`,
112:  *          //   2. interpolates output values using the locations and input
113:  *          //      data in `inp_slice`, and
114:  *          //   3. writes the first `len` values in the interpolated vector to
115:  *          //      `out_slice` with spatial offset being `offset`.
116:  *          //
117:  *          // This assumes that `grid_x` and `grid_y` all contain valid grid
118:  *          // values \in [-1, 1], even at indices greater than `len`.
119:  *          //
120:  *          // The `*_slice` argument names mean samples within a batch (i.e.,
```
- EN: The main symbol in this range is `ApplyGridSample`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `ApplyGridSample`，它们直接构成本文件的算子逻辑。

### Lines 121-163
```cpp
121:  *          // with the batch dimension sliced out).
122:  *          void forward(TensorAccessor<scalar_t, 3>& out_slice,
123:  *                       const TensorAccessor<scalar_t, 3>& inp_slice,
124:  *                       int64_t offset, const Vec& grid_x, const Vec& grid_y,
125:  *                       int64_t len) const;
126:  *
127:  *          // Applies grid sampling (backward) procedure. Arguments semantics
128:  *          // and strategy are similar to those of `forward`, with the
129:  *          // exception that `backward` has branches based on whether `input`
130:  *          // requires gradient (passed in as a template parameter). The
131:  *          // TensorAccessor for the input gradient is also given as a
132:  *          // pointer instead of reference, so that it can be null if the
133:  *          // gradient is not calculated.
134:  *          template <bool input_requires_grad>
135:  *          void backward(TensorAccessor<scalar_t, 3>* gInp_slice_ptr,
136:  *                        TensorAccessor<scalar_t, 3>& gGrid_slice,
137:  *                        const TensorAccessor<scalar_t, 3>& gOut_slice,
138:  *                        const TensorAccessor<scalar_t, 3>& inp_slice,
139:  *                        int64_t offset, const Vec& grid_x, const Vec& grid_y,
140:  *                        int64_t len) const;
141:  *        };
142:  *
143:  *   3. `grid_sample_2d_grid_slice_iterator` function
144:  *      Among the tensors we work with, we know that the output tensors are
145:  *      contiguous (i.e., `output` in forward, and `grad_input` & `grad_grid` in
146:  *      backward), we need to randomly read `input` anyways, and `grad_output`
147:  *      usually comes from autograd and is often contiguous. So we base our
148:  *      iterating strategy on the geometry of grid.
149:  *      `grid_sample_2d_grid_slice_iterator` function provides an abstraction to
150:  *      efficiently iterates through a `grid` slice (without batch dimension).
151:  *      See comments of that function on the specific cases and strategies used.
152:  *
153:  *        template<typename scalar_t, typename ApplyFn>
154:  *        void grid_sample_2d_grid_slice_iterator(
155:  *          const TensorAccessor<scalar_t, 3>& grid_slice,
156:  *          const ApplyFn &apply_fn);
157:  *
158:  *      `apply_fn` is a function/lambda that takes in
159:  *           i.   location vectors (x and y for 2D) extracted from grid
160:  *           ii.  `spatial_offset` as the spatial offset of these vectors
161:  *                from the beginning of this slice.
162:  *           iii. `len` as the number of valid locations in the vectors.
163:  *                (There might not be enough near boundary.)
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 165-194
```cpp
165:  *       It should be callable as if it has declaration:
166:  *          void apply_fn(const Vectorized<scalar_t>& grid_x,
167:  *                        const Vectorized<scalar_t>& grid_y,
168:  *                        int64_t spatial_offset, int64_t len);
169:  *
170:  *      `apply_fn` will be called multiple times, and together cover the entire
171:  *      output spatial space.
172:  *
173:  *  Now you should be able to understand everything about the implementation of
174:  *  2D forward kernel shown at the beginning of this note.
175:  *
176:  **/
177:
178:
179: using at::native::detail::GridSamplerInterpolation;
180: using at::native::detail::GridSamplerPadding;
181: using namespace at::vec;
182:
183:
184: // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ComputeLocation ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
185: // Struct to compute interpolation location from grid values, and to apply
186: // padding mechanism (e.g., reflection).
187: // See NOTE [ Grid Sample CPU Kernels ] for details.
188:
189: template<typename scalar_t, bool align_corners>
190: struct ComputeLocationBase;
191:
192: template<typename scalar_t>
193: struct ComputeLocationBase<scalar_t, /*align_corners=*/true> {
194:   using Vec = Vectorized<scalar_t>;
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `ComputeLocationBase`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `ComputeLocationBase`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 196-235
```cpp
196:   // values are clipped to between 0 and max_val
197:   const scalar_t max_val;
198:   // unnormalization scaling factor
199:   const scalar_t scaling_factor;
200:   // reflection parameters: reflected coordinates land in [low, low+span] inclusive
201:   const scalar_t low; // only used when align_corners=False
202:   const scalar_t twice_span;
203:   // if the reflecting span is empty, all reflected coords are set to 0
204:   const bool empty;
205:
206:   ComputeLocationBase(int64_t size)
207:     : max_val(static_cast<scalar_t>(size - 1))
208:     , scaling_factor(static_cast<scalar_t>(size - 1) / 2)
209:     , low(static_cast<scalar_t>(0))
210:     , twice_span(static_cast<scalar_t>(size - 1) * 2)
211:     , empty(size <= 1) {}
212:
213:   inline Vec unnormalize(const Vec &in) const {
214:     return (in + Vec(1)) * Vec(scaling_factor);
215:   }
216:
217:   inline Vec clip_coordinates(const Vec &in) const {
218:     // Invert order of clamp_min operands in order to clamp Nans to zero
219:     return clamp_max(Vec(max_val), clamp_min(Vec(0), in));
220:   }
221:
222:   // same as clip_coordinates but also returns the gradient multiplier
223:   inline std::pair<Vec, Vec> clip_coordinates_get_grad(const Vec &in) const {
224:     using int_t = int_same_size_t<scalar_t>;
225:     auto bounded_lo = maximum(in, Vec(0));
226:     // Integral type equality comparison is very very fast because it just looks
227:     // at the bits. Casting is free too. So we use the following pattern instead
228:     // of comparison + blendv.
229:     // Note that it is important for the gradient calculation that borders
230:     // are considered out of bounds.
231:     auto in_bound_lo = cast<scalar_t>(cast<int_t>(bounded_lo) != cast<int_t>(Vec(0)));
232:     auto res = minimum(bounded_lo, Vec(max_val));
233:     auto in_bound_hi = cast<scalar_t>(cast<int_t>(res) != cast<int_t>(Vec(max_val)));
234:     return std::make_pair(res, in_bound_lo & in_bound_hi);
235:   }
```
- EN: The main symbol in this range is `ComputeLocationBase`, `unnormalize`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `ComputeLocationBase`, `unnormalize`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 237-265
```cpp
237:   inline Vec reflect_coordinates(const Vec &in) const {
238:     if (empty) {
239:       return Vec(0);
240:     }
241:     Vec twice_span_vec(twice_span);
242:     auto abs_in = in.abs();
243:     auto fdouble_flips = abs_in / twice_span_vec;
244:     auto double_flips = fdouble_flips.trunc();
245:     auto extra = abs_in - double_flips * twice_span_vec;
246:     // Now we need to test if extra > max_val to find out if another flip is
247:     // needed. The following comparison does that and returns the correct
248:     // flipped value.
249:     return minimum(extra, twice_span_vec - extra);
250:   }
251:
252:   // same as reflect_coordinates but also returns the gradient multiplier
253:   inline std::pair<Vec, Vec> reflect_coordinates_get_grad(const Vec &in) const {
254:     if (empty) {
255:       return std::make_pair(Vec(0), Vec(0));
256:     }
257:     Vec twice_span_vec(twice_span);
258:     auto neg_in = in < Vec(0);
259:     auto abs_in = in.abs();
260:     auto fdouble_flips = abs_in / twice_span_vec;
261:     auto double_flips = fdouble_flips.trunc();
262:
263:     auto extra = abs_in - double_flips * twice_span_vec;
264:     auto reflected_extra = twice_span_vec - extra;
265:     auto one_more_flip = extra > reflected_extra;
```
- EN: The main symbol in this range is `reflect_coordinates`, `reflect_coordinates_get_grad`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `reflect_coordinates`, `reflect_coordinates_get_grad`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 267-297
```cpp
267:     return std::make_pair(
268:       Vec::blendv(extra, reflected_extra, one_more_flip),
269:       Vec::blendv(Vec(1), Vec(-1), one_more_flip ^ neg_in)
270:     );
271:   }
272: };
273:
274: template<typename scalar_t>
275: struct ComputeLocationBase<scalar_t, /*align_corners=*/false> {
276:   using Vec = Vectorized<scalar_t>;
277:
278:   // values are clipped to between 0 and max_val
279:   const scalar_t max_val;
280:   // unnormalization scaling factor
281:   const scalar_t scaling_factor;
282:   // reflection parameters: reflected coordinates land in [low, low+span] inclusive
283:   const scalar_t low;
284:   const scalar_t twice_span;
285:   // if the reflecting span is empty, all reflected coords are set to 0
286:   const bool empty; // only used when align_corners=True
287:
288:   ComputeLocationBase(int64_t size)
289:     : max_val(static_cast<scalar_t>(size - 1))
290:     , scaling_factor(static_cast<scalar_t>(size) / 2)
291:     , low(static_cast<scalar_t>(-0.5))
292:     , twice_span(static_cast<scalar_t>(size) * 2)
293:     , empty(size <= 0) {}
294:
295:   inline Vec unnormalize(const Vec &in) const {
296:     return (in + Vec(static_cast<scalar_t>(1))) * Vec(scaling_factor) - Vec(static_cast<scalar_t>(0.5));
297:   }
```
- EN: The main symbol in this range is `ComputeLocationBase`, `unnormalize`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `ComputeLocationBase`, `unnormalize`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 299-331
```cpp
299:   inline Vec clip_coordinates(const Vec &in) const {
300:     // Invert order of clamp_min operands in order to clamp Nans to zero
301:     return clamp_max(Vec(max_val), clamp_min(Vec(0), in));
302:   }
303:
304:   // same as clip_coordinates but also returns the gradient multiplier
305:   inline std::pair<Vec, Vec> clip_coordinates_get_grad(const Vec &in) const {
306:     using int_t = int_same_size_t<scalar_t>;
307:     auto bounded_lo = maximum(in, Vec(0));
308:     // Integral type equality comparison is very very fast because it just looks
309:     // at the bits. Casting is free too. So we use the following pattern instead
310:     // of comparison + blendv.
311:     // Note that it is important for the gradient calculation that borders
312:     // are considered out of bounds.
313:     auto in_bound_lo = cast<scalar_t>(cast<int_t>(bounded_lo) != cast<int_t>(Vec(0)));
314:     auto res = minimum(bounded_lo, Vec(max_val));
315:     auto in_bound_hi = cast<scalar_t>(cast<int_t>(res) != cast<int_t>(Vec(max_val)));
316:     return std::make_pair(res, in_bound_lo & in_bound_hi);
317:   }
318:
319:   inline Vec reflect_coordinates(const Vec &in) const {
320:     Vec twice_span_vec(twice_span), low_vec(low);
321:     // Since reflection is around low and low+span, subtract low before
322:     // the reflection, and then add it back at the end.
323:     auto abs_in = (in - low_vec).abs();
324:     auto fdouble_flips = abs_in / twice_span_vec;
325:     auto double_flips = fdouble_flips.trunc();
326:     auto extra = abs_in - double_flips * twice_span_vec;
327:     // Now we need to test if extra > max_val to find out if another flip is
328:     // needed. The following comparison does that and returns the correct
329:     // flipped value.
330:     return minimum(extra, twice_span_vec - extra) + low_vec;
331:   }
```
- EN: The main symbol in this range is `clip_coordinates`, `clip_coordinates_get_grad`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `clip_coordinates`, `clip_coordinates_get_grad`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 333-361
```cpp
333:   // same as reflect_coordinates but also returns the gradient multiplier
334:   inline std::pair<Vec, Vec> reflect_coordinates_get_grad(const Vec &in) const {
335:     Vec twice_span_vec(twice_span), low_vec(low);
336:     Vec in_minus_low = in - low_vec;
337:     auto neg_in = in_minus_low < Vec(0);
338:     auto abs_in = in_minus_low.abs();
339:     auto fdouble_flips = abs_in / twice_span_vec;
340:     auto double_flips = fdouble_flips.trunc();
341:
342:     auto extra = abs_in - double_flips * twice_span_vec;
343:     auto reflected_extra = twice_span_vec - extra;
344:     auto one_more_flip = extra > reflected_extra;
345:
346:     return std::make_pair(
347:       Vec::blendv(extra, reflected_extra, one_more_flip) + low_vec,
348:       Vec::blendv(Vec(1), Vec(-1), one_more_flip ^ neg_in)
349:     );
350:   }
351: };
352:
353: template<typename scalar_t, GridSamplerPadding padding, bool align_corners>
354: struct ComputeLocation;
355:
356: template<typename scalar_t, bool align_corners>
357: struct ComputeLocation<scalar_t, GridSamplerPadding::Zeros, align_corners>
358:   : ComputeLocationBase<scalar_t, align_corners> {
359:   using Vec = Vectorized<scalar_t>;
360:   using ComputeLocationBase<scalar_t, align_corners>::unnormalize;
361:   using ComputeLocationBase<scalar_t, align_corners>::scaling_factor;
```
- EN: The main symbol in this range is `reflect_coordinates_get_grad`, `ComputeLocation`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `reflect_coordinates_get_grad`, `ComputeLocation`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 363-391
```cpp
363:   using ComputeLocationBase<scalar_t, align_corners>::ComputeLocationBase;
364:
365:   inline Vec apply(const Vec &in) const {
366:     return unnormalize(in);
367:   }
368:
369:   inline Vec compute_coordinates(const Vec &in) const {
370:     return in;
371:   }
372:
373:   inline std::pair<Vec, Vec> apply_get_grad(const Vec &in) const {
374:     return std::make_pair(unnormalize(in), Vec(scaling_factor));
375:   }
376: };
377:
378: template<typename scalar_t, bool align_corners>
379: struct ComputeLocation<scalar_t, GridSamplerPadding::Border, align_corners>
380:   : ComputeLocationBase<scalar_t, align_corners> {
381:   using Vec = Vectorized<scalar_t>;
382:   using ComputeLocationBase<scalar_t, align_corners>::unnormalize;
383:   using ComputeLocationBase<scalar_t, align_corners>::clip_coordinates;
384:   using ComputeLocationBase<scalar_t, align_corners>::clip_coordinates_get_grad;
385:   using ComputeLocationBase<scalar_t, align_corners>::scaling_factor;
386:
387:   using ComputeLocationBase<scalar_t, align_corners>::ComputeLocationBase;
388:
389:   inline Vec apply(const Vec &in) const {
390:     return clip_coordinates(unnormalize(in));
391:   }
```
- EN: The main symbol in this range is `apply`, `compute_coordinates`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `apply`, `compute_coordinates`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 393-426
```cpp
393:   inline Vec compute_coordinates(const Vec &in) const {
394:     return clip_coordinates(in);
395:   }
396:
397:   inline std::pair<Vec, Vec> apply_get_grad(const Vec &in) const {
398:     auto [res, grad_clip] = clip_coordinates_get_grad(unnormalize(in));
399:     return std::make_pair(res, grad_clip & Vec(scaling_factor));
400:   }
401: };
402:
403: template<typename scalar_t, bool align_corners>
404: struct ComputeLocation<scalar_t, GridSamplerPadding::Reflection, align_corners>
405:   : ComputeLocationBase<scalar_t, align_corners> {
406:   using Vec = Vectorized<scalar_t>;
407:   using ComputeLocationBase<scalar_t, align_corners>::unnormalize;
408:   using ComputeLocationBase<scalar_t, align_corners>::clip_coordinates;
409:   using ComputeLocationBase<scalar_t, align_corners>::clip_coordinates_get_grad;
410:   using ComputeLocationBase<scalar_t, align_corners>::reflect_coordinates;
411:   using ComputeLocationBase<scalar_t, align_corners>::reflect_coordinates_get_grad;
412:   using ComputeLocationBase<scalar_t, align_corners>::scaling_factor;
413:
414:   using ComputeLocationBase<scalar_t, align_corners>::ComputeLocationBase;
415:
416:   inline Vec apply(const Vec &in) const {
417:     auto res = reflect_coordinates(unnormalize(in));
418:     res = clip_coordinates(res);
419:     return res;
420:   }
421:
422:   inline Vec compute_coordinates(const Vec &in) const {
423:     auto res = reflect_coordinates(in);
424:     res = clip_coordinates(res);
425:     return res;
426:   }
```
- EN: The main symbol in this range is `compute_coordinates`, `apply_get_grad`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `compute_coordinates`, `apply_get_grad`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 428-456
```cpp
428:   inline std::pair<Vec, Vec> apply_get_grad(const Vec &in) const {
429:     auto [res, grad_refl] = reflect_coordinates_get_grad(unnormalize(in));
430:     Vec grad(scaling_factor);
431:     grad = grad_refl * grad;
432:     auto [res2, grad_clip] = clip_coordinates_get_grad(res);
433:     grad = grad_clip & grad;
434:     return std::make_pair(res2, grad);
435:   }
436: };
437:
438: // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ApplyGridSample ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
439: // Struct to apply grid sample (reading from input, interpolate, and write to
440: // output).
441: // See NOTE [ Grid Sample CPU Kernels ] for details.
442:
443: template<typename scalar_t>
444: inline void
445: mask_scatter_add(const scalar_t *src, scalar_t* base_addr,
446:                  const int_same_size_t<scalar_t> *offsets,
447:                  const int_same_size_t<scalar_t> *mask, int64_t len) {
448:   #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
449:   # pragma unroll
450:   #endif
451:   for (const auto i : c10::irange(len)) {
452:     if (mask[i] & 0x01) {
453:       base_addr[offsets[i]] += src[i];
454:     }
455:   }
456: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_get_grad`, `sample`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_get_grad`, `sample`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 458-489
```cpp
458: template<typename scalar_t, int spatial_dim,
459:          GridSamplerInterpolation interp,
460:          GridSamplerPadding padding,
461:          bool align_corners>
462: struct ApplyGridSample;
463:
464: template<typename scalar_t, GridSamplerPadding padding, bool align_corners>
465: struct ApplyGridSample<scalar_t, 2, GridSamplerInterpolation::Bilinear,
466:                        padding, align_corners> {
467:   using Vec = Vectorized<scalar_t>;
468:   using integer_t = int_same_size_t<scalar_t>;
469:   using iVec = Vectorized<integer_t>;
470:
471:   const int64_t inp_H;
472:   const int64_t inp_W;
473:   const int64_t inp_sH;
474:   const int64_t inp_sW;
475:   const int64_t C;
476:   const int64_t inp_sC;
477:   const ComputeLocation<scalar_t, padding, align_corners> compute_H;
478:   const ComputeLocation<scalar_t, padding, align_corners> compute_W;
479:   const bool must_in_bound = padding != GridSamplerPadding::Zeros;
480:
481:   ApplyGridSample(const TensorAccessor<const scalar_t, 4>& input)
482:     : inp_H(input.size(2))
483:     , inp_W(input.size(3))
484:     , inp_sH(input.stride(2))
485:     , inp_sW(input.stride(3))
486:     , C(input.size(1))
487:     , inp_sC(input.stride(1))
488:     , compute_H(input.size(2))
489:     , compute_W(input.size(3)) {}
```
- EN: The main symbol in this range is `ApplyGridSample`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `ApplyGridSample`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 491-520
```cpp
491:   inline std::tuple<
492:     Vec, Vec, Vec, Vec,       // distances to 4 sides
493:     Vec, Vec, Vec, Vec,       // interpolation weights wrt 4 corners
494:     Vec, Vec, Vec, Vec,       // in_bound masks
495:     iVec, iVec                // y_n and x_w
496:   >
497:   compute_interp_params(const Vec& x, const Vec& y) const {
498:     // get NE, NW, SE, SW pixel values from (x, y)
499:     // assuming we get exact integer representation and just use scalar_t
500:     // if we don't, the weights will be garbage anyways.
501:     auto x_w = x.floor();
502:     auto y_n = y.floor();
503:
504:     // get distances to each side
505:     auto w = x - x_w;
506:     auto e = Vec(1) - w;
507:     auto n = y - y_n;
508:     auto s = Vec(1) - n;
509:
510:     // get interpolation weights for each neighbor
511:     // e.g., for the nw corner, the weight is `dist_to_south * dist_to_east`.
512:     auto nw = s * e;
513:     auto ne = s * w;
514:     auto sw = n * e;
515:     auto se = n * w;
516:
517:     auto i_x_w = convert_to_int_of_same_size(x_w);
518:     auto i_y_n = convert_to_int_of_same_size(y_n);
519:     auto i_x_e = i_x_w + iVec(1);
520:     auto i_y_s = i_y_n + iVec(1);
```
- EN: The main symbol in this range is `compute_interp_params`, which contributes directly to this file's operator logic. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `compute_interp_params`，它们直接构成本文件的算子逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 522-551
```cpp
522:     // Use int comparison because it is much faster than float comp with AVX2
523:     // (latency 1 cyc vs. 4 cyc on skylake)
524:     // Avoid using the le and ge because those are not implemented in AVX2 and
525:     // are actually simulated using multiple instructions.
526:     auto w_mask = must_in_bound ? iVec(-1)  // true = all ones
527:                                 : (i_x_w > iVec(-1)) & (i_x_w < iVec(inp_W));
528:     auto n_mask = must_in_bound ? iVec(-1)  // true = all ones
529:                                 : (i_y_n > iVec(-1)) & (i_y_n < iVec(inp_H));
530:     auto e_mask = must_in_bound ? (i_x_e < iVec(inp_W))
531:                                 : (i_x_e > iVec(-1)) & (i_x_e < iVec(inp_W));
532:     auto s_mask = must_in_bound ? (i_y_s < iVec(inp_H))
533:                                 : (i_y_s > iVec(-1)) & (i_y_s < iVec(inp_H));
534:     auto nw_mask = cast<scalar_t>(must_in_bound ? iVec(-1) : (w_mask & n_mask));
535:     auto ne_mask = cast<scalar_t>(e_mask & n_mask);
536:     auto sw_mask = cast<scalar_t>(w_mask & s_mask);
537:     auto se_mask = cast<scalar_t>(e_mask & s_mask);
538:
539:     return std::make_tuple(
540:       n, s, w, e,
541:       nw, ne, sw, se,
542:       nw_mask, ne_mask, sw_mask, se_mask,
543:       i_y_n, i_x_w);
544:   }
545:
546:   inline void forward(TensorAccessor<scalar_t, 3>& out_slice,
547:                       const TensorAccessor<const scalar_t, 3>& inp_slice,
548:                       int64_t offset, const Vec& grid_x, const Vec& grid_y,
549:                       int64_t len) const {
550:     auto x = compute_W.apply(grid_x);
551:     auto y = compute_H.apply(grid_y);
```
- EN: The main symbol in this range is `forward`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `forward`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 553-587
```cpp
553:     auto interp_params = compute_interp_params(x, y);
554:
555:     auto nw = std::get<4>(interp_params);
556:     auto ne = std::get<5>(interp_params);
557:     auto sw = std::get<6>(interp_params);
558:     auto se = std::get<7>(interp_params);
559:
560:     auto nw_mask = std::get<8>(interp_params);
561:     auto ne_mask = std::get<9>(interp_params);
562:     auto sw_mask = std::get<10>(interp_params);
563:     auto se_mask = std::get<11>(interp_params);
564:
565:     auto i_y_n = std::get<12>(interp_params);
566:     auto i_x_w = std::get<13>(interp_params);
567:
568:     auto i_nw_offset = i_y_n * iVec(inp_sH) + i_x_w * iVec(inp_sW);
569:     auto i_ne_offset = i_nw_offset + iVec(inp_sW);
570:     auto i_sw_offset = i_nw_offset + iVec(inp_sH);
571:     auto i_se_offset = i_sw_offset + iVec(inp_sW);
572:
573:     #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
574:     # pragma unroll
575:     #endif
576:     for (const auto c : c10::irange(C)) {
577:       auto inp_slice_C_ptr = inp_slice[c].data();
578:
579:       // mask_gather zeros out the mask, so we need to make copies
580:       Vec nw_mask_copy = nw_mask;
581:       Vec ne_mask_copy = ne_mask;
582:       Vec sw_mask_copy = sw_mask;
583:       Vec se_mask_copy = se_mask;
584:       auto nw_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_nw_offset, nw_mask_copy);
585:       auto ne_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_ne_offset, ne_mask_copy);
586:       auto sw_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_sw_offset, sw_mask_copy);
587:       auto se_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_se_offset, se_mask_copy);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 589-624
```cpp
589:       auto interpolated = (nw_val * nw) + (ne_val * ne) + (sw_val * sw) + (se_val * se);
590:       interpolated.store(out_slice[c].data() + offset, len);
591:     }
592:   }
593:
594:   template<bool input_requires_grad>
595:   inline void backward(TensorAccessor<scalar_t, 3>* gInp_slice_ptr,
596:                        TensorAccessor<scalar_t, 3>& gGrid_slice,
597:                        const TensorAccessor<const scalar_t, 3>& gOut_slice,
598:                        const TensorAccessor<const scalar_t, 3>& inp_slice,
599:                        int64_t offset, const Vec& grid_x, const Vec& grid_y,
600:                        int64_t len) const {
601:     auto [x, gx_mult] = compute_W.apply_get_grad(grid_x);
602:     auto [y, gy_mult] = compute_H.apply_get_grad(grid_y);
603:
604:     auto [
605:       n, s, w, e, nw, ne, sw, se, nw_mask, ne_mask, sw_mask, se_mask,
606:       i_y_n, i_x_w] = compute_interp_params(x, y);
607:
608:     auto i_nw_offset = i_y_n * iVec(inp_sH) + i_x_w * iVec(inp_sW);
609:     auto i_ne_offset = i_nw_offset + iVec(inp_sW);
610:     auto i_sw_offset = i_nw_offset + iVec(inp_sH);
611:     auto i_se_offset = i_sw_offset + iVec(inp_sW);
612:
613:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
614:     integer_t i_nw_mask_arr[iVec::size()];
615:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
616:     integer_t i_ne_mask_arr[iVec::size()];
617:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
618:     integer_t i_sw_mask_arr[iVec::size()];
619:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
620:     integer_t i_se_mask_arr[iVec::size()];
621:     nw_mask.store(i_nw_mask_arr);
622:     ne_mask.store(i_ne_mask_arr);
623:     sw_mask.store(i_sw_mask_arr);
624:     se_mask.store(i_se_mask_arr);
```
- EN: The main symbol in this range is `backward`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `backward`，它们直接构成本文件的算子逻辑。

### Lines 626-656
```cpp
626:     // i_gInp_*_offset_arr and gInp_corner_arr variables below are unnecessary
627:     // when input_requires_grad is false (they are only used within the
628:     // if-blocks), but required to make the code well-formed.
629:
630:     // When reading input values, we used mask_gather. Unfortunately, there is
631:     // no mask_scatter_add (the backward of mask_gather) in Intel intrinsics.
632:     // So we store the necessary vectors to temporary arrays and use the helper
633:     // mask_scatter_add defined above.
634:
635:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
636:     integer_t i_gInp_nw_offset_arr[iVec::size()];
637:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
638:     integer_t i_gInp_ne_offset_arr[iVec::size()];
639:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
640:     integer_t i_gInp_sw_offset_arr[iVec::size()];
641:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
642:     integer_t i_gInp_se_offset_arr[iVec::size()];
643:     if (input_requires_grad) {
644:       auto i_gInp_nw_offset = i_y_n * iVec(inp_W) + i_x_w;
645:       auto i_gInp_ne_offset = i_gInp_nw_offset + iVec(1);
646:       auto i_gInp_sw_offset = i_gInp_nw_offset + iVec(inp_W);
647:       auto i_gInp_se_offset = i_gInp_sw_offset + iVec(1);
648:
649:       i_gInp_nw_offset.store(i_gInp_nw_offset_arr);
650:       i_gInp_ne_offset.store(i_gInp_ne_offset_arr);
651:       i_gInp_sw_offset.store(i_gInp_sw_offset_arr);
652:       i_gInp_se_offset.store(i_gInp_se_offset_arr);
653:     }
654:
655:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
656:     scalar_t gInp_corner_arr[Vec::size()];
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 658-688
```cpp
658:     auto gx = Vec(0), gy = Vec(0);
659:     #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
660:     # pragma unroll
661:     #endif
662:     for (const auto c : c10::irange(C)) {
663:       auto inp_slice_C_ptr = inp_slice[c].data();
664:       auto gOut = Vec::loadu(gOut_slice[c].data() + offset, len);
665:
666:       if (input_requires_grad) {
667:         TORCH_INTERNAL_ASSERT(gInp_slice_ptr);
668:         auto gInp_slice_C_ptr = (*gInp_slice_ptr)[c].data();
669:
670:         (nw * gOut).store(gInp_corner_arr);
671:         mask_scatter_add(gInp_corner_arr, gInp_slice_C_ptr, i_gInp_nw_offset_arr, i_nw_mask_arr, len);
672:         (ne * gOut).store(gInp_corner_arr);
673:         mask_scatter_add(gInp_corner_arr, gInp_slice_C_ptr, i_gInp_ne_offset_arr, i_ne_mask_arr, len);
674:         (sw * gOut).store(gInp_corner_arr);
675:         mask_scatter_add(gInp_corner_arr, gInp_slice_C_ptr, i_gInp_sw_offset_arr, i_sw_mask_arr, len);
676:         (se * gOut).store(gInp_corner_arr);
677:         mask_scatter_add(gInp_corner_arr, gInp_slice_C_ptr, i_gInp_se_offset_arr, i_se_mask_arr, len);
678:       }
679:
680:       // mask_gather zeros out the mask, so we need to make copies
681:       Vec nw_mask_copy = nw_mask;
682:       Vec ne_mask_copy = ne_mask;
683:       Vec sw_mask_copy = sw_mask;
684:       Vec se_mask_copy = se_mask;
685:       auto nw_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_nw_offset, nw_mask_copy);
686:       auto ne_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_ne_offset, ne_mask_copy);
687:       auto sw_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_sw_offset, sw_mask_copy);
688:       auto se_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_C_ptr, i_se_offset, se_mask_copy);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 690-722
```cpp
690:       gx = gx + ((ne_val - nw_val) * s + (se_val - sw_val) * n) * gOut;
691:       gy = gy + ((sw_val - nw_val) * e + (se_val - ne_val) * w) * gOut;
692:     }
693:
694:     gx = gx * gx_mult;
695:     gy = gy * gy_mult;
696:
697:     constexpr int64_t step = Vec::size();
698:     auto interleaved_gGrid = interleave2(gx, gy);
699:     auto gGrid_ptr = gGrid_slice.data() + offset * 2;
700:     std::get<0>(interleaved_gGrid).store(gGrid_ptr,
701:                                          std::min(len * 2, step));
702:     std::get<1>(interleaved_gGrid).store(gGrid_ptr + step,
703:                                          std::max(static_cast<int64_t>(0), len * 2 - step));
704:   }
705: };
706:
707: template<typename scalar_t, GridSamplerPadding padding, bool align_corners>
708: struct ApplyGridSample<scalar_t, 2, GridSamplerInterpolation::Nearest,
709:                        padding, align_corners> {
710:   using Vec = Vectorized<scalar_t>;
711:   using integer_t = int_same_size_t<scalar_t>;
712:   using iVec = Vectorized<integer_t>;
713:
714:   const int64_t inp_H;
715:   const int64_t inp_W;
716:   const int64_t inp_sH;
717:   const int64_t inp_sW;
718:   const int64_t C;
719:   const int64_t inp_sC;
720:   const ComputeLocation<scalar_t, padding, align_corners> compute_H;
721:   const ComputeLocation<scalar_t, padding, align_corners> compute_W;
722:   const bool must_in_bound = padding != GridSamplerPadding::Zeros;
```
- EN: The main symbol in this range is `ApplyGridSample`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `ApplyGridSample`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 724-752
```cpp
724:   ApplyGridSample(const TensorAccessor<const scalar_t, 4>& input)
725:     : inp_H(input.size(2))
726:     , inp_W(input.size(3))
727:     , inp_sH(input.stride(2))
728:     , inp_sW(input.stride(3))
729:     , C(input.size(1))
730:     , inp_sC(input.stride(1))
731:     , compute_H(input.size(2))
732:     , compute_W(input.size(3)) {}
733:
734:   inline void forward(TensorAccessor<scalar_t, 3>& out_slice,
735:                       const TensorAccessor<const scalar_t, 3>& inp_slice,
736:                       int64_t offset, const Vec& grid_x, const Vec& grid_y,
737:                       int64_t len) const {
738:     auto x = compute_W.apply(grid_x);
739:     auto y = compute_H.apply(grid_y);
740:
741:     auto x_nearest = x.round();
742:     auto y_nearest = y.round();
743:
744:     auto i_x_nearest = convert_to_int_of_same_size(x_nearest);
745:     auto i_y_nearest = convert_to_int_of_same_size(y_nearest);
746:
747:     auto i_mask = must_in_bound ? iVec(-1)
748:                                 : (i_x_nearest > iVec(-1)) & (i_x_nearest < iVec(inp_W)) &
749:                                   (i_y_nearest > iVec(-1)) & (i_y_nearest < iVec(inp_H));
750:     auto mask = cast<scalar_t>(i_mask);
751:
752:     auto i_offset = i_y_nearest * iVec(inp_sH) + i_x_nearest * iVec(inp_sW);
```
- EN: The main symbol in this range is `ApplyGridSample`, `forward`, which contributes directly to this file's operator logic. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `ApplyGridSample`, `forward`，它们直接构成本文件的算子逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 754-783
```cpp
754:     auto out_ptr = out_slice.data() + offset;
755:     auto out_sC = out_slice.stride(0);
756:     auto inp_slice_ptr = inp_slice.data();
757:     #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
758:     # pragma unroll
759:     #endif
760:     for (int64_t c = 0; c < C; ++c, out_ptr += out_sC, inp_slice_ptr += inp_sC) {
761:       // mask_gather zeros out the mask, so we need to make a copy
762:       auto mask_copy = mask;
763:       auto inp_val = mask_gather<sizeof(scalar_t)>(Vec(0), inp_slice_ptr, i_offset, mask_copy);
764:       inp_val.store(static_cast<void*>(out_ptr), len);
765:     }
766:   }
767:
768:   template<bool input_requires_grad>
769:   inline void backward(TensorAccessor<scalar_t, 3>* gInp_slice_ptr,
770:                        TensorAccessor<scalar_t, 3>& gGrid_slice,
771:                        const TensorAccessor<const scalar_t, 3>& gOut_slice,
772:                        const TensorAccessor<const scalar_t, 3>& /*inp_slice*/,
773:                        int64_t offset, const Vec& grid_x, const Vec& grid_y,
774:                        int64_t len) const {
775:     if (input_requires_grad) {
776:       auto x = compute_W.apply(grid_x);
777:       auto y = compute_H.apply(grid_y);
778:
779:       auto x_nearest = x.round();
780:       auto y_nearest = y.round();
781:
782:       auto i_x_nearest = convert_to_int_of_same_size(x_nearest);
783:       auto i_y_nearest = convert_to_int_of_same_size(y_nearest);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `backward`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `backward`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 785-820
```cpp
785:       auto i_mask = must_in_bound ? iVec(-1)
786:                                   : (i_x_nearest > iVec(-1)) & (i_x_nearest < iVec(inp_W)) &
787:                                     (i_y_nearest > iVec(-1)) & (i_y_nearest < iVec(inp_H));
788:
789:       auto i_gInp_offset = i_y_nearest * iVec(inp_W) + i_x_nearest;  // gInp is contiguous
790:
791:       // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
792:       integer_t mask_arr[iVec::size()];
793:       i_mask.store(mask_arr);
794:       // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
795:       integer_t gInp_offset_arr[iVec::size()];
796:       i_gInp_offset.store(gInp_offset_arr);
797:
798:       #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
799:       # pragma unroll
800:       #endif
801:       for (const auto c : c10::irange(C)) {
802:         mask_scatter_add(gOut_slice[c].data() + offset, (*gInp_slice_ptr)[c].data(),
803:                         gInp_offset_arr, mask_arr, len);
804:       }
805:     }
806:
807:     // grid has zero 0 gradient in Nearest mode
808:     auto gGrid_ptr = gGrid_slice.data() + offset * 2;
809:     std::memset(gGrid_ptr, 0, sizeof(scalar_t) * len * 2);
810:   }
811: };
812:
813: // Use bicubic convolution algorithm. Based on
814: // https://en.wikipedia.org/wiki/Bicubic_interpolation#Bicubic_convolution_algorithm
815: template<typename scalar_t, GridSamplerPadding padding, bool align_corners>
816: struct ApplyGridSample<scalar_t, 2, GridSamplerInterpolation::Bicubic,
817:                        padding, align_corners> {
818:   using Vec = Vectorized<scalar_t>;
819:   using integer_t = int_same_size_t<scalar_t>;
820:   using iVec = Vectorized<integer_t>;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `ApplyGridSample`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `ApplyGridSample`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 822-858
```cpp
822:   const int64_t inp_H;
823:   const int64_t inp_W;
824:   const int64_t inp_sH;
825:   const int64_t inp_sW;
826:   const int64_t C;
827:   const int64_t inp_sC;
828:   const ComputeLocation<scalar_t, padding, align_corners> compute_H;
829:   const ComputeLocation<scalar_t, padding, align_corners> compute_W;
830:   const bool must_in_bound = padding != GridSamplerPadding::Zeros;
831:
832:   // constant used in cubic convolution
833:   // could be -0.5 or -0.75, use the same value in UpSampleBicubic2d.h
834:   const Vec A = Vec(static_cast<scalar_t>(-0.75));
835:
836:   ApplyGridSample(const TensorAccessor<const scalar_t, 4>& input)
837:     : inp_H(input.size(2))
838:     , inp_W(input.size(3))
839:     , inp_sH(input.stride(2))
840:     , inp_sW(input.stride(3))
841:     , C(input.size(1))
842:     , inp_sC(input.stride(1))
843:     , compute_H(input.size(2))
844:     , compute_W(input.size(3)) {}
845:
846:   // Calculate the cubic convolution coefficient
847:   // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
848:   inline void get_cubic_coefficients(Vec (&coeffs)[4], const Vec& tx) const {
849:     Vec x;
850:     x = tx + Vec(1);  // 1 < x = |-1 - tx| < 2
851:     coeffs[0] = ((A * x - Vec(5) * A) * x + Vec(8) * A) * x - Vec(4) * A;
852:     x = tx;           // x = |0 - tx| <= 1
853:     coeffs[1] = ((A + Vec(2)) * x - (A + Vec(3))) * x * x + Vec(1);
854:     x = Vec(1) - tx;  // x = |1 - tx| <= 1
855:     coeffs[2] = ((A + Vec(2)) * x - (A + Vec(3))) * x * x + Vec(1);
856:     x = Vec(2) - tx;  // 1 < x = |2 - tx| < 2
857:     coeffs[3] = ((A * x - Vec(5) * A) * x + Vec(8) * A) * x - Vec(4) * A;
858:   }
```
- EN: The main symbol in this range is `ApplyGridSample`, which contributes directly to this file's operator logic. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `ApplyGridSample`，它们直接构成本文件的算子逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 860-889
```cpp
860:   // Calculate the differential of the cubic convolution, i.e. `d coeff / d x`
861:   // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
862:   inline void get_cubic_coefficients_grad(Vec (&coeffs)[4], const Vec& tx) const {
863:     Vec x;
864:     x = Vec(-1) - tx; // 1 < x = |-1 - tx| < 2
865:     coeffs[0] = (Vec(-3) * A * x - Vec(10) * A ) * x - Vec(8) * A;
866:     x = Vec(0) - tx;  // x = |0 - tx| <= 1
867:     coeffs[1] = (Vec(-3) * (A + Vec(2)) * x - Vec(2) * (A + Vec(3))) * x;
868:     x = Vec(1) - tx;  // x = |1 - tx| <= 1
869:     coeffs[2] = (Vec(3) * (A + Vec(2)) * x - Vec(2) * (A + Vec(3))) * x;
870:     x = Vec(2) - tx;  // 1 < x = |2 - tx| < 2
871:     coeffs[3] = (Vec(3) * A * x - Vec(10) * A) * x + Vec(8) * A;
872:   }
873:
874:   inline Vec get_value_bounded(const scalar_t* data, const Vec& x, const Vec& y) const {
875:     auto ix = convert_to_int_of_same_size(compute_W.compute_coordinates(x));
876:     auto iy = convert_to_int_of_same_size(compute_H.compute_coordinates(y));
877:
878:     auto mask_x = must_in_bound ? iVec(-1) : (ix > iVec(-1)) & (ix < iVec(inp_W));
879:     auto mask_y = must_in_bound ? iVec(-1) : (iy > iVec(-1)) & (iy < iVec(inp_H));
880:     auto mask = cast<scalar_t>(mask_x & mask_y);
881:
882:     auto offset = iy * iVec(inp_sH) + ix * iVec(inp_sW);
883:
884:     auto val = mask_gather<sizeof(scalar_t)>(Vec(0), data, offset, mask);
885:     return val;
886:   }
887:
888:   inline void add_value_bounded(scalar_t* data, int64_t len, const Vec& x, const Vec&y,
889:                                const Vec& delta) const {
```
- EN: The main symbol in this range is `get_value_bounded`, `add_value_bounded`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `get_value_bounded`, `add_value_bounded`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 891-920
```cpp
891:     auto ix = convert_to_int_of_same_size(compute_W.compute_coordinates(x));
892:     auto iy = convert_to_int_of_same_size(compute_H.compute_coordinates(y));
893:
894:     auto mask_x = must_in_bound ? iVec(-1) : (ix > iVec(-1)) & (ix < iVec(inp_W));
895:     auto mask_y = must_in_bound ? iVec(-1) : (iy > iVec(-1)) & (iy < iVec(inp_H));
896:     auto mask = cast<scalar_t>(mask_x & mask_y);
897:
898:     auto i_gInp_offset = iy * iVec(inp_W) + ix;
899:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
900:     integer_t i_gInp_offset_arr[iVec::size()];
901:     i_gInp_offset.store(i_gInp_offset_arr);
902:
903:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
904:     integer_t mask_arr[iVec::size()];
905:     mask.store(mask_arr);
906:
907:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
908:     scalar_t gInp_corner_arr[Vec::size()];
909:     delta.store(gInp_corner_arr);
910:
911:     mask_scatter_add(gInp_corner_arr, data, i_gInp_offset_arr, mask_arr, len);
912:   }
913:
914:   inline void forward(TensorAccessor<scalar_t, 3>& out_slice,
915:                       const TensorAccessor<const scalar_t, 3>& inp_slice,
916:                       int64_t offset, const Vec& grid_x, const Vec& grid_y,
917:                       int64_t len) const {
918:
919:     auto x = compute_W.unnormalize(grid_x);
920:     auto y = compute_H.unnormalize(grid_y);
```
- EN: The main symbol in this range is `forward`, which contributes directly to this file's operator logic. The math and shape handling relate to convolution-style operators. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `forward`，它们直接构成本文件的算子逻辑。 这里的计算与形状处理与卷积类算子相关。 这里处理归一化相关的状态、缩放或统计量。

### Lines 922-954
```cpp
922:     auto ix = x.floor();
923:     auto iy = y.floor();
924:
925:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
926:     Vec coeff_x[4];
927:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
928:     Vec coeff_y[4];
929:     get_cubic_coefficients(coeff_x, x - ix);
930:     get_cubic_coefficients(coeff_y, y - iy);
931:
932:     #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
933:     # pragma unroll
934:     #endif
935:     for (const auto c : c10::irange(C)) {
936:       auto inp_slice_C_ptr = inp_slice[c].data();
937:
938:       // Interpolate the 4 values in the x direction
939:       // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
940:       Vec interp_x[4];
941:       for (const auto i : c10::irange(4)) {
942:         interp_x[i] =
943:           coeff_x[0] * get_value_bounded(inp_slice_C_ptr, ix - Vec(1), iy + Vec(-1 + i)) +
944:           coeff_x[1] * get_value_bounded(inp_slice_C_ptr, ix + Vec(0), iy + Vec(-1 + i)) +
945:           coeff_x[2] * get_value_bounded(inp_slice_C_ptr, ix + Vec(1), iy + Vec(-1 + i)) +
946:           coeff_x[3] * get_value_bounded(inp_slice_C_ptr, ix + Vec(2), iy + Vec(-1 + i));
947:       }
948:
949:       // Interpolate the 4 values in the y direction
950:       auto interpolated = coeff_y[0] * interp_x[0] + coeff_y[1] * interp_x[1] +
951:                           coeff_y[2] * interp_x[2] + coeff_y[3] * interp_x[3];
952:       interpolated.store(out_slice[c].data() + offset, len);
953:     }
954:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 956-991
```cpp
956:   template<bool input_requires_grad>
957:   inline void backward(TensorAccessor<scalar_t, 3>* gInp_slice_ptr,
958:                       TensorAccessor<scalar_t, 3>& gGrid_slice,
959:                       const TensorAccessor<const scalar_t, 3>& gOut_slice,
960:                       const TensorAccessor<const scalar_t, 3>& inp_slice,
961:                       int64_t offset, const Vec& grid_x, const Vec& grid_y,
962:                       int64_t len) const {
963:     Vec x = compute_W.unnormalize(grid_x);
964:     Vec y = compute_H.unnormalize(grid_y);
965:     Vec gx_mult = Vec(compute_W.scaling_factor);
966:     Vec gy_mult = Vec(compute_H.scaling_factor);
967:
968:     auto ix = x.floor();
969:     auto iy = y.floor();
970:
971:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
972:     Vec coeff_x[4];
973:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
974:     Vec coeff_y[4];
975:     get_cubic_coefficients(coeff_x, x - ix);
976:     get_cubic_coefficients(coeff_y, y - iy);
977:
978:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
979:     Vec coeff_x_grad[4];
980:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
981:     Vec coeff_y_grad[4];
982:     get_cubic_coefficients_grad(coeff_x_grad, x - ix);
983:     get_cubic_coefficients_grad(coeff_y_grad, y - iy);
984:
985:     auto gx = Vec(0), gy = Vec(0);
986:     #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
987:     # pragma unroll
988:     #endif
989:     for (const auto c : c10::irange(C)) {
990:       auto inp_slice_C_ptr = inp_slice[c].data();
991:       auto gOut = Vec::loadu(gOut_slice[c].data() + offset, len);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `backward`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `backward`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 993-1021
```cpp
 993:       for (const auto i : c10::irange(4)) {
 994:         for (const auto j : c10::irange(4)) {
 995:           auto xx = ix + Vec(-1 + i);
 996:           auto yy = iy + Vec(-1 + j);
 997:
 998:           if (input_requires_grad) {
 999:             auto gInp_slice_C_ptr = (*gInp_slice_ptr)[c].data();
1000:             add_value_bounded(gInp_slice_C_ptr, len, xx, yy, gOut * coeff_x[i] * coeff_y[j]);
1001:           }
1002:
1003:           auto val = get_value_bounded(inp_slice_C_ptr, xx, yy);
1004:           gx = gx - val * gOut * coeff_x_grad[i] * coeff_y[j];
1005:           gy = gy - val * gOut * coeff_y_grad[j] * coeff_x[i];
1006:         }
1007:       }
1008:     }
1009:
1010:     gx = gx * gx_mult;
1011:     gy = gy * gy_mult;
1012:
1013:     constexpr int64_t step = Vec::size();
1014:     auto interleaved_gGrid = interleave2(gx, gy);
1015:     auto gGrid_ptr = gGrid_slice.data() + offset * 2;
1016:     std::get<0>(interleaved_gGrid).store(gGrid_ptr,
1017:                                          std::min(len * 2, step));
1018:     std::get<1>(interleaved_gGrid).store(gGrid_ptr + step,
1019:                                          std::max(static_cast<int64_t>(0), len * 2 - step));
1020:   }
1021: };
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1023-1065
```cpp
1023: #if defined(__GNUC__) && __GNUC__ >= 12 && defined(__aarch64__)
1024: #pragma GCC pop_options
1025: #endif
1026:
1027: // ~~~~~~~~~~~~~~~~~~ grid_sample_2d_grid_slice_iterator ~~~~~~~~~~~~~~~~~~~~~~
1028: // Function to apply a vectorized function on a grid slice tensor (without batch
1029: // dimension).
1030: // See NOTE [ Grid Sample CPU Kernels ] for details.
1031:
1032: template<typename scalar_t, typename ApplyFn>
1033: inline void grid_sample_2d_grid_slice_iterator(
1034:     const TensorAccessor<const scalar_t, 3>& grid_slice, const ApplyFn &apply_fn) {
1035:   int64_t out_H = grid_slice.size(0);
1036:   int64_t out_W = grid_slice.size(1);
1037:   int64_t grid_sH = grid_slice.stride(0);
1038:   int64_t grid_sW = grid_slice.stride(1);
1039:   int64_t grid_sCoor = grid_slice.stride(2);
1040:   auto grid_ptr = grid_slice.data();
1041:
1042:   using Vec = Vectorized<scalar_t>;
1043:   using iVec = Vectorized<int_same_size_t<scalar_t>>;
1044:   constexpr int64_t step = Vec::size();
1045:
1046:   // Loop over each output pixel in grid.
1047:   // We consider the following three cases (after slicing out the batch
1048:   // dimension).
1049:   // See detailed discussions under each if-case.
1050:
1051:   if (at::geometry_is_contiguous({out_H, out_W, 2}, {grid_sH, grid_sW, grid_sCoor})) {
1052:     // Case 1:
1053:     // Grid is contiguous.
1054:     // Strategy: Sequentially load two vectors at the same time, and get,
1055:     //           e.g.,  {x0, y0, x1, y1}, {x2, y2, x3, y3}. Then we use
1056:     //           at::vec::deinterleave2 to get x and y vectors.
1057:     auto total_size = out_H * out_W;
1058:     for (int64_t spatial_offset = 0; spatial_offset < total_size; spatial_offset += step) {
1059:       auto grid_offset = spatial_offset * 2;
1060:       auto len = std::min(step, total_size - spatial_offset);
1061:       auto vec1 = Vec::loadu(grid_ptr + grid_offset,
1062:                              std::min(step, len * 2));
1063:       auto vec2 = Vec::loadu(grid_ptr + grid_offset + step,
1064:                              std::max(static_cast<int64_t>(0), len * 2 - step));
1065:       auto [x, y] = deinterleave2(vec1, vec2);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1067-1097
```cpp
1067:       // make sure that x and y are valid grid sample locations
1068:       if (len < step) {
1069:         x = Vec::set(Vec(0), x, len);
1070:         y = Vec::set(Vec(0), y, len);
1071:       }
1072:       apply_fn(x, y, spatial_offset, len);
1073:     }
1074:   } else if (grid_sW == 1 || out_W == 1) {
1075:     // Case 2:
1076:     // The W dimension is contiguous.
1077:     // This can be common, e.g., grid is from a conv net output of shape
1078:     // [N, 2, H, W].
1079:     // Strategy: Divide into two contiguous slices each of shape [H, W], and
1080:     //           each containing x and y vectors. So we sequentially load a
1081:     //           vector from each of them to get x and y vector
1082:
1083:     // Function to apply along a contiguous W dimension (or flattened H x W).
1084:     auto line_fn = [&](const scalar_t *grid_ptr_x, const scalar_t *grid_ptr_y,
1085:                        int64_t out_base_offset, int64_t total_size) {
1086:       for (int64_t i = 0; i < total_size; i += step) {
1087:         auto len = std::min(step, total_size - i);
1088:         auto x = Vec::loadu(grid_ptr_x + i, len);
1089:         auto y = Vec::loadu(grid_ptr_y + i, len);
1090:         // make sure that x and y are valid grid sample locations
1091:         if (len < step) {
1092:           x = Vec::set(Vec(0), x, len);
1093:           y = Vec::set(Vec(0), y, len);
1094:         }
1095:         apply_fn(x, y, out_base_offset + i, len);
1096:       }
1097:     };
```
- EN: The main symbol in this range is `dimension`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `dimension`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1099-1136
```cpp
1099:     if (at::geometry_is_contiguous({out_H, out_W}, {grid_sH, grid_sW})) {
1100:       // If [H, W] is contiguous, apply line_fn once.
1101:       line_fn(grid_ptr, grid_ptr + grid_sCoor, 0, out_H * out_W);
1102:     } else {
1103:       // If only [W] is contiguous, apply line_fn once for each h slice.
1104:       auto grid_ptr_NH = grid_ptr;
1105:       for (const auto h : c10::irange(out_H)) {
1106:         line_fn(grid_ptr_NH, grid_ptr_NH + grid_sCoor, h * out_W, out_W);
1107:         grid_ptr_NH += grid_sH;
1108:       }
1109:     }
1110:   } else {
1111:     // Case 3:
1112:     // General case.
1113:     // Strategy: Do a for-loop over H, for each W slice, use
1114:     //           at::vec::gather to load the x and y vectors.
1115:     int64_t spatial_offset = 0;
1116:     const int64_t i_offset_delta = grid_sW * step;
1117:
1118:     #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
1119:     # pragma unroll
1120:     #endif
1121:     for (const auto h : c10::irange(out_H)) {
1122:       auto grid_ptr_x = grid_ptr + h * grid_sH;
1123:       auto grid_ptr_y = grid_ptr_x + grid_sCoor;
1124:       auto i_offsets = iVec::arange(0, grid_sW);
1125:       #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
1126:       # pragma unroll
1127:       #endif
1128:       for (int64_t w = 0; w < out_W; w += step) {
1129:         auto len = std::min(step, out_W - w);
1130:         if (len < step) {
1131:           // prevents illegal memory access, sets the exceeding offsets to zero
1132:           i_offsets = iVec::set(iVec(0), i_offsets, len);
1133:         }
1134:         apply_fn(vec::gather<sizeof(scalar_t)>(grid_ptr_x, i_offsets),
1135:                  vec::gather<sizeof(scalar_t)>(grid_ptr_y, i_offsets),
1136:                  spatial_offset, len);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1138-1166
```cpp
1138:         grid_ptr_x += i_offset_delta;
1139:         grid_ptr_y += i_offset_delta;
1140:         spatial_offset += len;
1141:       }
1142:     }
1143:   }
1144: }
1145:
1146: // ~~~~~~~~~~~~~~~~~~~~~~~~~ Grid Sample Kernels ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
1147: // Use the structs & functions defined above to calculate grid sample forward
1148: // and backward.
1149: // See NOTE [ Grid Sample CPU Kernels ] for details.
1150:
1151: void grid_sampler_2d_cpu_kernel_impl(
1152:     const TensorBase &output, const TensorBase &input, const TensorBase &grid,
1153:     int64_t interpolation_mode, int64_t padding_mode, bool align_corners) {
1154:   auto N = input.size(0);
1155:   auto H = grid.size(1);
1156:   auto W = grid.size(2);
1157:   auto spatial_size = H * W;
1158:   auto grain_size = spatial_size == 0 ? (N + 1)
1159:                                       : at::divup(at::internal::GRAIN_SIZE, spatial_size * 4 /* 2d * 2 tensors*/);
1160:   if (output.numel() == 0) {
1161:          return;
1162:   }
1163:
1164: #define HANDLE_CASE(interp, padding, align_corners)                            \
1165:   case padding: {                                                              \
1166:     ApplyGridSample<scalar_t, 2, interp, padding, align_corners>               \
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `grid_sampler_2d_cpu_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `grid_sampler_2d_cpu_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1167-1196
```cpp
1167:     grid_sample(inp_acc);                                                      \
1168:     parallel_for(0, N, grain_size, [&](int64_t begin, int64_t end) {           \
1169:       for (const auto n : c10::irange(begin, end)) {                           \
1170:         auto out_slice = out_acc[n];                                           \
1171:         auto inp_slice = inp_acc[n];                                           \
1172:         grid_sample_2d_grid_slice_iterator(                                    \
1173:           grid_acc[n],                                                         \
1174:           [&](const Vectorized<scalar_t>& grid_x, const Vectorized<scalar_t>& grid_y,  \
1175:               int64_t spatial_offset, int64_t len) {                           \
1176:             grid_sample.forward(out_slice, inp_slice, spatial_offset,          \
1177:                                 grid_x, grid_y, len);                          \
1178:           });                                                                  \
1179:         }                                                                      \
1180:       });                                                                      \
1181:     return;                                                                    \
1182:   }
1183:
1184: #define HANDLE_INTERP(interp, align_corners)                                   \
1185:   case interp: {                                                               \
1186:     switch (static_cast<GridSamplerPadding>(padding_mode)) {                   \
1187:       HANDLE_CASE(interp, GridSamplerPadding::Zeros, align_corners);           \
1188:       HANDLE_CASE(interp, GridSamplerPadding::Border, align_corners);          \
1189:       HANDLE_CASE(interp, GridSamplerPadding::Reflection, align_corners);      \
1190:     }                                                                          \
1191:     return;                                                                    \
1192:   }
1193:
1194:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(), "grid_sampler_2d_cpu_kernel_impl", [&] {
1195:     auto out_acc = output.accessor<scalar_t, 4>();
1196:     auto inp_acc = input.accessor<const scalar_t, 4>();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `grid_sample_2d_grid_slice_iterator`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `grid_sample_2d_grid_slice_iterator`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1197-1232
```cpp
1197:     auto grid_acc = grid.accessor<const scalar_t, 4>();
1198:     if (align_corners) {
1199:       switch (static_cast<GridSamplerInterpolation>(interpolation_mode)) {
1200:         HANDLE_INTERP(GridSamplerInterpolation::Bilinear, true);
1201:         HANDLE_INTERP(GridSamplerInterpolation::Nearest, true);
1202:         HANDLE_INTERP(GridSamplerInterpolation::Bicubic, true);
1203:       }
1204:     } else {
1205:       switch (static_cast<GridSamplerInterpolation>(interpolation_mode)) {
1206:         HANDLE_INTERP(GridSamplerInterpolation::Bilinear, false);
1207:         HANDLE_INTERP(GridSamplerInterpolation::Nearest, false);
1208:         HANDLE_INTERP(GridSamplerInterpolation::Bicubic, false);
1209:       }
1210:     }
1211:   });
1212: #undef HANDLE_CASE
1213: #undef HANDLE_INTERP
1214: }
1215:
1216: void grid_sampler_2d_backward_cpu_kernel_impl(
1217:     const TensorBase &grad_input,
1218:     const TensorBase &grad_grid,
1219:     const TensorBase &grad_output_,
1220:     const TensorBase &input,
1221:     const TensorBase &grid,
1222:     int64_t interpolation_mode,
1223:     int64_t padding_mode,
1224:     bool align_corners,
1225:     std::array<bool,2> output_mask) {
1226:   if (grad_output_.numel() == 0) {
1227:     grad_grid.zero_();
1228:     return;
1229:   }
1230:   // grad_output should be contiguous most of time. Ensuring that it is
1231:   // contiguous can greatly simplify this code.
1232:   auto grad_output = grad_output_.contiguous();
```
- EN: The main symbol in this range is `grid_sampler_2d_backward_cpu_kernel_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `grid_sampler_2d_backward_cpu_kernel_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1234-1270
```cpp
1234:   // If `input` gradient is not required, we skip computing it -- not needing to create
1235:   // the tensor to hold the gradient can markedly increase performance. (`grid` gradient
1236:   // is always computed.)
1237:   auto input_requires_grad = output_mask[0];
1238:
1239:   auto N = input.size(0);
1240:   auto spatial_size = grid.size(1) * grid.size(2);
1241:   auto grain_size = spatial_size == 0 ? (N + 1)
1242:                                       : at::divup(at::internal::GRAIN_SIZE, spatial_size * 10 /* 2d * 5 tensors*/);
1243:
1244: #define GINP_SLICE_PTR_true auto gInp_slice = gInp_acc[n]; auto gInp_slice_ptr = &gInp_slice;
1245: #define GINP_SLICE_PTR_false TensorAccessor<scalar_t, 3>* gInp_slice_ptr = nullptr;
1246: #define GINP_SLICE_PTR(input_requires_grad) GINP_SLICE_PTR_##input_requires_grad
1247:
1248: #define HANDLE_CASE(interp, padding, align_corners, input_requires_grad)         \
1249:   case padding: {                                                                \
1250:     ApplyGridSample<scalar_t, 2, interp, padding, align_corners>                 \
1251:     grid_sample(inp_acc);                                                        \
1252:     parallel_for(0, N, grain_size, [&](int64_t begin, int64_t end) {             \
1253:       for (const auto n : c10::irange(begin, end)) {                             \
1254:         GINP_SLICE_PTR(input_requires_grad)                                      \
1255:         auto gGrid_slice = gGrid_acc[n];                                         \
1256:         auto gOut_slice = gOut_acc[n];                                           \
1257:         auto inp_slice = inp_acc[n];                                             \
1258:         grid_sample_2d_grid_slice_iterator(                                      \
1259:           grid_acc[n],                                                           \
1260:           [&](const Vectorized<scalar_t>& grid_x, const Vectorized<scalar_t>& grid_y,    \
1261:               int64_t spatial_offset, int64_t len) {                             \
1262:             grid_sample.backward<input_requires_grad>(gInp_slice_ptr, gGrid_slice,       \
1263:                                                       gOut_slice, inp_slice,     \
1264:                                                       spatial_offset, grid_x,    \
1265:                                                       grid_y, len);              \
1266:           });                                                                    \
1267:       }                                                                          \
1268:     });                                                                          \
1269:     return;                                                                      \
1270:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `grid_sample_2d_grid_slice_iterator`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `grid_sample_2d_grid_slice_iterator`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 1272-1300
```cpp
1272: #define HANDLE_INTERP(interp, align_corners, input_requires_grad)           \
1273:   case interp: {                                                            \
1274:     switch (static_cast<GridSamplerPadding>(padding_mode)) {                \
1275:       HANDLE_CASE(interp, GridSamplerPadding::Zeros, align_corners, input_requires_grad);      \
1276:       HANDLE_CASE(interp, GridSamplerPadding::Border, align_corners, input_requires_grad);     \
1277:       HANDLE_CASE(interp, GridSamplerPadding::Reflection, align_corners, input_requires_grad); \
1278:     }                                                                       \
1279:     return;                                                                 \
1280:   }
1281:
1282:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(), "grid_sampler_2d_backward_cpu_kernel_impl", [&] {
1283:     auto gGrid_acc = grad_grid.accessor<scalar_t, 4>();
1284:     auto inp_acc = input.accessor<const scalar_t, 4>();
1285:     auto grid_acc = grid.accessor<const scalar_t, 4>();
1286:     auto gOut_acc = grad_output.accessor<const scalar_t, 4>();
1287:     if (input_requires_grad) {
1288:       auto gInp_acc = grad_input.accessor<scalar_t, 4>();
1289:       if (align_corners) {
1290:         switch (static_cast<GridSamplerInterpolation>(interpolation_mode)) {
1291:           HANDLE_INTERP(GridSamplerInterpolation::Bilinear, true, true);
1292:           HANDLE_INTERP(GridSamplerInterpolation::Nearest, true, true);
1293:           HANDLE_INTERP(GridSamplerInterpolation::Bicubic, true, true);
1294:         }
1295:       } else {
1296:         switch (static_cast<GridSamplerInterpolation>(interpolation_mode)) {
1297:           HANDLE_INTERP(GridSamplerInterpolation::Bilinear, false, true);
1298:           HANDLE_INTERP(GridSamplerInterpolation::Nearest, false, true);
1299:           HANDLE_INTERP(GridSamplerInterpolation::Bicubic, false, true);
1300:         }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1301-1329
```cpp
1301:       }
1302:     } else {
1303:       if (align_corners) {
1304:         switch (static_cast<GridSamplerInterpolation>(interpolation_mode)) {
1305:           HANDLE_INTERP(GridSamplerInterpolation::Bilinear, true, false);
1306:           HANDLE_INTERP(GridSamplerInterpolation::Nearest, true, false);
1307:           HANDLE_INTERP(GridSamplerInterpolation::Bicubic, true, false);
1308:         }
1309:       } else {
1310:         switch (static_cast<GridSamplerInterpolation>(interpolation_mode)) {
1311:           HANDLE_INTERP(GridSamplerInterpolation::Bilinear, false, false);
1312:           HANDLE_INTERP(GridSamplerInterpolation::Nearest, false, false);
1313:           HANDLE_INTERP(GridSamplerInterpolation::Bicubic, false, false);
1314:         }
1315:       }
1316:
1317:     }
1318:   });
1319: #undef HANDLE_CASE
1320: #undef HANDLE_INTERP
1321: }
1322:
1323: }
1324:
1325: REGISTER_DISPATCH(grid_sampler_2d_cpu_kernel, &grid_sampler_2d_cpu_kernel_impl)
1326: REGISTER_DISPATCH(grid_sampler_2d_backward_cpu_kernel, &grid_sampler_2d_backward_cpu_kernel_impl)
1327:
1328:
1329: }  // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Normalization statistics / 归一化统计

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/GridSampler.h`, `ATen/native/cpu/GridSamplerKernel.h`, `ATen/core/TensorBase.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/TensorGeometry.h`, `ATen/TensorIterator.h`, `ATen/cpu/vec/vec.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `algorithm`, `cstring`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `TensorBase`, `cpu_kernel`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`
