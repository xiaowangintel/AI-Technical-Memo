# UpSampleKernelAVXAntialias.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/UpSampleKernelAVXAntialias.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU resampling/interpolation kernels and coordinate mapping logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 重采样/插值 kernel 与坐标映射逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33
```cpp
 1: /*
 2: The Python Imaging Library (PIL) is
 3:
 4:     Copyright © 1997-2011 by Secret Labs AB
 5:     Copyright © 1995-2011 by Fredrik Lundh
 6:
 7: Pillow is the friendly PIL fork. It is
 8:
 9:     Copyright © 2010-2022 by Alex Clark and contributors
10:
11: Like PIL, Pillow is licensed under the open source HPND License
12: */
13:
14: // This code is heavily inspired from PILLOW-SIMD's implementation:
15: // https://github.com/uploadcare/pillow-simd/blob/simd/master/src/libImaging/Resample.c
16:
17: #pragma once
18: #ifdef CPU_CAPABILITY_AVX2
19: // TODO: This file only supports AVX2. We could split the AVX kernels into
20: // smaller logical blocks in order to port them into the Vec.h logic. This would
21: // allow to support other vectorization architectures and perhaps also support
22: // the non-vectorized fallback (we'd need to make sure it's not slower than the
23: // current fallback).
24:
25: #include <ATen/core/Tensor.h>
26: #include <ATen/cpu/vec/intrinsics.h>
27: #include <c10/util/irange.h>
28:
29: #ifndef AT_PER_OPERATOR_HEADERS
30: #include <ATen/Functions.h>
31: #else
32: #include <ATen/ops/empty.h>
33: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/cpu/vec/intrinsics.h`, `c10/util/irange.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/cpu/vec/intrinsics.h`, `c10/util/irange.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 36-67
```cpp
36: namespace {
37:
38: inline __m128i mm_cvtsi32_si128(const uint8_t* C10_RESTRICT ptr, bool i32_aligned) {
39:   int32_t v;
40:   if (i32_aligned) {
41:     v = *(const int32_t*)ptr;
42:   } else {
43:     std::memcpy(&v, ptr, 4);
44:   }
45:   return _mm_cvtsi32_si128(v);
46: }
47:
48: inline __m128i mm_cvtepu8_epi32(const uint8_t* C10_RESTRICT ptr, bool i32_aligned) {
49:   return _mm_cvtepu8_epi32(mm_cvtsi32_si128(ptr, i32_aligned));
50: }
51:
52: inline void _write_endline_rgb_as_uint32(
53:     uint8_t* C10_RESTRICT output,
54:     uint32_t data
55: ) {
56:   // data is (R G B X), output is (X1 X2 X3 | R1 B1 G1 R2 ...)
57:   // Here we explicitly set X as R1
58:   uint8_t* data_ptr = reinterpret_cast<uint8_t*>(&data);
59:   data_ptr[3] = output[3];
60:   std::memcpy(output, data_ptr, 4);
61: }
62:
63: at::Tensor unpack_rgb(const at::Tensor& packed_tensor) {
64:   // Convert a "packed" tensor (typically RGBRGBRGB if channels_last) into
65:   // RGBARGBARGBA format where A is hard-coded to 0. Each pixel is encoded
66:   // into as 32 bits. This generalizes to num_channels <= 4 and also works for
67:   // non-channels_last tensors.
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `mm_cvtsi32_si128`, `mm_cvtepu8_epi32`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `mm_cvtsi32_si128`, `mm_cvtepu8_epi32`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 69-97
```cpp
69:   const uint8_t* packed = (const uint8_t*)packed_tensor.const_data_ptr<uint8_t>();
70:   auto num_pixels = packed_tensor.size(1) * packed_tensor.size(2);
71:   auto num_channels = packed_tensor.size(0);
72:
73:   constexpr int rgba_size = 4;
74:   auto unpacked_tensor = at::empty({rgba_size, packed_tensor.size(1), packed_tensor.size(2)}, at::CPU(at::kByte));
75:   uint8_t* unpacked = (uint8_t*) unpacked_tensor.data_ptr<uint8_t>();
76:
77:   auto stride_i = packed_tensor.stride(2);
78:   auto stride_j = packed_tensor.stride(0);
79:
80:   for (const auto i : c10::irange(num_pixels)) {
81:     for (const auto j : c10::irange(rgba_size)) {
82:       unpacked[rgba_size * i + j] = (j < num_channels) ? packed[stride_i * i + stride_j * j] : 0;
83:     }
84:   }
85:   return unpacked_tensor;
86: }
87:
88: void pack_rgb(
89:     const at::Tensor& unpacked_tensor, // IN
90:     const at::Tensor& packed_tensor // OUT
91: ) {
92:   // Convert from unpacked channels last 3-channels or 4-channels tensor into original data layout.
93:
94:   uint8_t* unpacked = (uint8_t*)unpacked_tensor.data_ptr<uint8_t>();
95:   uint8_t* packed = (uint8_t*)packed_tensor.data_ptr<uint8_t>();
96:   auto num_pixels = packed_tensor.size(1) * packed_tensor.size(2);
97:   auto num_channels = packed_tensor.size(0);
```
- EN: The main symbol in this range is `pack_rgb`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `pack_rgb`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 99-131
```cpp
 99:   auto unpacked_increment = unpacked_tensor.size(0);
100:   auto packed_increment = packed_tensor.stride(2);
101:   auto packed_stride = packed_tensor.stride(0);
102:
103:   TORCH_INTERNAL_ASSERT(unpacked_increment == 3 || unpacked_increment == 4);
104:
105:   for ([[maybe_unused]] const auto i : c10::irange(num_pixels)) {
106:     for (const auto j : c10::irange(num_channels)) {
107:       packed[j * packed_stride] = unpacked[j];
108:     }
109:     unpacked += unpacked_increment;
110:     packed += packed_increment;
111:   }
112: }
113:
114: void ImagingResampleHorizontalConvolution8u4x(
115:     uint8_t* C10_RESTRICT lineOut0,
116:     uint8_t* C10_RESTRICT lineOut1,
117:     uint8_t* C10_RESTRICT lineOut2,
118:     uint8_t* C10_RESTRICT lineOut3,
119:     int64_t out_xsize,
120:     const uint8_t* C10_RESTRICT lineIn0,
121:     const uint8_t* C10_RESTRICT lineIn1,
122:     const uint8_t* C10_RESTRICT lineIn2,
123:     const uint8_t* C10_RESTRICT lineIn3,
124:     int64_t in_xsize,
125:     const int64_t* idx_ptr_xmin,
126:     const int64_t* idx_ptr_size,
127:     const int16_t* kk,
128:     int kmax,
129:     unsigned int coefs_precision,
130:     int64_t num_channels,
131:     bool is_last_line);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 133-162
```cpp
133: void ImagingResampleHorizontalConvolution8u(
134:     uint8_t* C10_RESTRICT lineOut,
135:     int64_t out_xsize,
136:     const uint8_t* C10_RESTRICT lineIn,
137:     int64_t in_xsize,
138:     const int64_t* idx_ptr_xmin,
139:     const int64_t* idx_ptr_size,
140:     const int16_t* kk,
141:     int kmax,
142:     unsigned int coefs_precision,
143:     int64_t num_channels,
144:     bool is_last_line);
145:
146: void ImagingResampleVerticalConvolution8u(
147:     uint8_t* C10_RESTRICT lineOut,
148:     const uint8_t* C10_RESTRICT lineIn,
149:     int64_t xsize,
150:     int64_t ids_min,
151:     int64_t ids_size,
152:     const int16_t* k,
153:     unsigned int coefs_precision,
154:     int64_t num_channels);
155:
156: template<int num_channels>
157: void ImagingResampleHorizontal(
158:     const at::Tensor & unpacked_output,
159:     const at::Tensor & unpacked_input,
160:     int ksize,
161:     const std::vector<at::Tensor>& horiz_indices_weights,
162:     unsigned int horiz_weights_precision) {
```
- EN: The main symbol in this range is `ImagingResampleHorizontal`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `ImagingResampleHorizontal`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 164-194
```cpp
164:   // Interpolation horizontal pass: we compute x-axis (image width) interpolation outputs.
165:
166:   // Input data is stored as
167:   //   input = [r[0], g[0], b[0], a[0], r[1], g[1], b[1], a[1], r[2], g[2], b[2], a[2], ...]
168:   // Weights are float values computed for each output pixel and rescaled to uint16:
169:   //   weights[i] = [w[i, 0], w[i, 1], ..., w[i, K-1]]
170:   // We want to compute the output as following:
171:   //   output = [oR[0], oG[0], oB[0], oA[0], oR[1], oG[1], oB[1], oA[1], ...]
172:   // where
173:   //   oR[yoffset + i] = r[yoffset + xmin[i]] * w[i, 0] + ... + r[yoffset + xmin[i] + K-1] * w[i, K-1]
174:   //   oG[yoffset + i] = g[yoffset + xmin[i]] * w[i, 0] + ... + g[yoffset + xmin[i] + K-1] * w[i, K-1]
175:   //   oB[yoffset + i] = b[yoffset + xmin[i]] * w[i, 0] + ... + b[yoffset + xmin[i] + K-1] * w[i, K-1]
176:   //
177:
178:   // TODO: we may want to merge that into the fallback code (currently called
179:   // basic_loop_separable_1d_horizontal<uint8_t>)
180:   // Although this may not be needed if / when we port all this code to use
181:   // Vec.h since this would potentially give us another fall-back implem
182:
183:   const int16_t* kk = (int16_t*)(horiz_indices_weights[3].const_data_ptr<double>());
184:
185:   auto xout = unpacked_output.size(2);
186:   auto yout = unpacked_output.size(1);
187:   auto xin = unpacked_input.size(2);
188:   TORCH_INTERNAL_ASSERT(num_channels == unpacked_input.size(0));
189:
190:   const int64_t* idx_ptr_xmin = horiz_indices_weights[0].const_data_ptr<int64_t>();
191:   const int64_t* idx_ptr_size = horiz_indices_weights[1].const_data_ptr<int64_t>();
192:
193:   uint8_t* unpacked_output_p = unpacked_output.data_ptr<uint8_t>();
194:   const uint8_t* unpacked_input_p = unpacked_input.const_data_ptr<uint8_t>();
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 196-233
```cpp
196:   int64_t yy = 0;
197:   auto xout_stride = xout * num_channels;
198:   auto xin_stride = xin * num_channels;
199:   for (; yy < yout - 3; yy += 4) {
200:     ImagingResampleHorizontalConvolution8u4x(
201:         unpacked_output_p + yy * xout_stride,
202:         unpacked_output_p + (yy + 1) * xout_stride,
203:         unpacked_output_p + (yy + 2) * xout_stride,
204:         unpacked_output_p + (yy + 3) * xout_stride,
205:         xout,
206:         unpacked_input_p + yy * xin_stride,
207:         unpacked_input_p + (yy + 1) * xin_stride,
208:         unpacked_input_p + (yy + 2) * xin_stride,
209:         unpacked_input_p + (yy + 3) * xin_stride,
210:         xin,
211:         idx_ptr_xmin,
212:         idx_ptr_size,
213:         kk,
214:         ksize,
215:         horiz_weights_precision,
216:         num_channels,
217:         yy + 3 == yout - 1);
218:   }
219:   for (; yy < yout; yy++) {
220:     ImagingResampleHorizontalConvolution8u(
221:         unpacked_output_p + yy * xout_stride,
222:         xout,
223:         unpacked_input_p + yy * xin_stride,
224:         xin,
225:         idx_ptr_xmin,
226:         idx_ptr_size,
227:         kk,
228:         ksize,
229:         horiz_weights_precision,
230:         num_channels,
231:         yy == yout - 1);
232:   }
233: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 235-264
```cpp
235: void ImagingResampleVertical(
236:     const at::Tensor & unpacked_output,
237:     const at::Tensor & unpacked_input,
238:     int ksize,
239:     const std::vector<at::Tensor>& vert_indices_weights,
240:     unsigned int vert_weights_precision) {
241:
242:   // Interpolation vertical pass: we compute y-axis interpolation outputs.
243:   // Input data is stored as
244:   //   input = [r[0], g[0], b[0], a[0], r[1], g[1], b[1], a[1], r[2], g[2], b[2], a[2], ...]
245:   // Weights are float values computed for each output pixel and rescaled to uint16:
246:   //   weights[i] = [w[i, 0], w[i, 1], ..., w[i, K-1]]
247:   // We want to compute the output as following:
248:   //   output = [oR[0], oG[0], oB[0], oA[0], oR[1], oG[1], oB[1], oA[1], ...]
249:   // where
250:   //   oR[xoffset + i] = r[xoffset + ymin[i]] * w[i, 0] + ... + r[xoffset + ymin[i] + (K-1) * xsize] * w[i, K-1]
251:   //   oG[xoffset + i] = g[xoffset + ymin[i]] * w[i, 0] + ... + g[xoffset + ymin[i] + (K-1) * xsize] * w[i, K-1]
252:   //   oB[xoffset + i] = b[xoffset + ymin[i]] * w[i, 0] + ... + b[xoffset + ymin[i] + (K-1) * xsize] * w[i, K-1]
253:
254:   // TODO: we may want to merge that into the fallback code (currently called
255:   // basic_loop_separable_1d_vertical<uint8_t>)
256:   // Although this may not be needed if / when we port all this code to use
257:   // Vec.h since this would potentially give us another fall-back implem
258:   const int16_t* kk = (int16_t*)(vert_indices_weights[3].const_data_ptr<double>());
259:
260:   const int64_t* idx_ptr_xmin = vert_indices_weights[0].const_data_ptr<int64_t>();
261:   const int64_t* idx_ptr_size = vert_indices_weights[1].const_data_ptr<int64_t>();
262:
263:   uint8_t* unpacked_output_p = unpacked_output.data_ptr<uint8_t>();
264:   const uint8_t* unpacked_input_p = unpacked_input.const_data_ptr<uint8_t>();
```
- EN: The main symbol in this range is `ImagingResampleVertical`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `ImagingResampleVertical`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 266-294
```cpp
266:   auto xout = unpacked_output.size(2);
267:   auto yout = unpacked_output.size(1);
268:   const auto num_channels = unpacked_input.size(0);
269:   TORCH_INTERNAL_ASSERT(num_channels == unpacked_output.size(0));
270:
271:   auto xout_stride = xout * num_channels;
272:   for (const auto yy : c10::irange(yout)) {
273:     const auto* k = &kk[yy * ksize];
274:     auto ids_min = idx_ptr_xmin[yy];
275:     auto ids_size = idx_ptr_size[yy];
276:     ImagingResampleVerticalConvolution8u(
277:         unpacked_output_p + yy * xout_stride,
278:         unpacked_input_p,
279:         xout,
280:         ids_min,
281:         ids_size,
282:         k,
283:         vert_weights_precision,
284:         num_channels);
285:   }
286: }
287:
288: // This is the only public entry point in this file.  It supports bilinear or bicubic
289: // mode for uint8 dtype when C <= 4, with or without antialias. The
290: // implem is based on PIL-SIMD.
291: // Its equivalent implementation (fallback) for when AVX isn't supported or when
292: // C > 4 is upsample_separable_Nd_kernel_impl()  There are a bunch of
293: // future improvement that can be done: look for the TODOs in this file.
294: // For details on how the weights are computed and how the multiplications are
```
- EN: The main symbol in this range is `upsample_separable_Nd_kernel_impl`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `upsample_separable_Nd_kernel_impl`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 295-331
```cpp
295: // run on int (instead of float weights), see
296: // [ Weights computation for uint8_t and multiplication trick ]
297: // For details on how the AVX kernels are implemented, see
298: // https://gist.github.com/NicolasHug/47c97d731f05eaad5694c173849b86f5
299: // See also [ Support for antialias=False as a subcase of antialias=True ] to
300: // learn more about how the antialias=False case is computed. The same holds
301: // here: all these kernels are general enough to handle an arbitrary number of
302: // weights, but when aa=False they could be optimized further.
303: template <typename scale_type, class F>
304: void upsample_avx_bilinear_bicubic_uint8(
305:     const at::Tensor& input_,
306:     const at::Tensor& output,
307:     bool align_corners,
308:     const scale_type& scales,
309:     bool antialias) {
310:   auto batch_size = input_.size(0);
311:   auto num_channels = input_.size(1);
312:   auto xin = input_.size(3);
313:   auto yin = input_.size(2);
314:   auto xout = output.size(3);
315:   auto yout = output.size(2);
316:
317:   if (xin == xout && yin == yout) {
318:     output.copy_(input_);
319:     return;
320:   }
321:
322:   at::Tensor input = input_;
323:   if (!(input.is_contiguous() || input.is_contiguous(at::MemoryFormat::ChannelsLast))) {
324:     // If input is not contiguous with memory format channels first or channels last,
325:     // we explicitly convert the input to contiguous channels last memory format.
326:     // This simplifies the rest of the code and let us assume that the format is only contiguous channels first or channels last,
327:     // Most tensors going through this `if` block won't need to go through unpacking, but those having C < 3 may
328:     // have to (this means 2 copies are made). We could avoid the extra copy by handling non-contiguous input
329:     // directly within unpack_rgb() and pack_rgb(), but initial attempts showed that this is fairly complex.
330:     input = input.contiguous(at::MemoryFormat::ChannelsLast);
331:   }
```
- EN: The main symbol in this range is `int`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `int`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 333-373
```cpp
333:   auto need_horizontal = xout != xin;
334:   auto need_vertical = yout != yin;
335:
336:   int ksize_horiz, ksize_vert;
337:   std::vector<at::Tensor> horiz_indices_weights, vert_indices_weights;
338:   unsigned int horiz_weights_precision, vert_weights_precision;
339:
340:   bool skip_unpacking = (num_channels == 3 || num_channels == 4) && input.is_contiguous(at::MemoryFormat::ChannelsLast);
341:   bool skip_packing = (num_channels == 3 || num_channels == 4) && output.is_contiguous(at::MemoryFormat::ChannelsLast);
342:
343:   if (need_horizontal) {
344:     int interp_dim = 3;
345:     auto stride = skip_unpacking ? num_channels : 4;
346:     std::tie(horiz_indices_weights, ksize_horiz, horiz_weights_precision) =
347:         F::compute_index_ranges_int16_weights(
348:             /*input_size=*/xin,
349:             /*output_size=*/xout,
350:             /*stride=*/stride,
351:             /*ndims=*/4,
352:             /*reshape_dim=*/interp_dim,
353:             /*align_corners=*/align_corners,
354:             /*opt_scale=*/scales[interp_dim - 2],
355:             /*antialias=*/antialias,
356:             /*align_i32=*/true);
357:   }
358:
359:   if (need_vertical) {
360:     int interp_dim = 2;
361:     auto stride = skip_unpacking ? num_channels * xout : 4 * xout;
362:     std::tie(vert_indices_weights, ksize_vert, vert_weights_precision) =
363:         F::compute_index_ranges_int16_weights(
364:             /*input_size=*/yin,
365:             /*output_size=*/yout,
366:             /*stride=*/stride,
367:             /*ndims=*/4,
368:             /*reshape_dim=*/interp_dim,
369:             /*align_corners=*/align_corners,
370:             /*opt_scale=*/scales[interp_dim - 2],
371:             /*antialias=*/antialias,
372:             /*align_i32=*/true);
373:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 375-414
```cpp
375:   at::Tensor buffer_horiz, buffer_vert;
376:   // Minor optimization: we can avoid allocating an extra buffer if we're performing
377:   // horizontal-only or vertical-only interpolation, and if the tensor doesn't
378:   // need repacking
379:   if (need_horizontal && (need_vertical || !skip_packing)) {
380:     auto c = skip_unpacking ? num_channels : 4;
381:     buffer_horiz = at::empty({c, yin, xout}, input.options());
382:   }
383:   if (need_vertical && !skip_packing) {
384:     auto c = skip_unpacking ? num_channels : 4;
385:     buffer_vert = at::empty({c, yout, xout}, input.options());
386:   }
387:
388:   for (const auto i : c10::irange(batch_size)) {
389:
390:     at::Tensor unpacked_input = skip_unpacking ? input[i] : unpack_rgb(input[i]);
391:     at::Tensor unpacked_output;
392:
393:     if (need_horizontal) {
394:       at::Tensor unpacked_output_temp = (need_vertical || !skip_packing) ? buffer_horiz : output[i];
395:
396:       if (skip_unpacking && num_channels == 3) {
397:         ImagingResampleHorizontal<3>(
398:           unpacked_output_temp,
399:           unpacked_input,
400:           ksize_horiz,
401:           horiz_indices_weights,
402:           horiz_weights_precision);
403:       } else {
404:         ImagingResampleHorizontal<4>(
405:             unpacked_output_temp,
406:             unpacked_input,
407:             ksize_horiz,
408:             horiz_indices_weights,
409:             horiz_weights_precision);
410:       }
411:       unpacked_output = unpacked_input = unpacked_output_temp;
412:     }
413:     if (need_vertical) {
414:       unpacked_output = skip_packing ? output[i] : buffer_vert;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 416-450
```cpp
416:       ImagingResampleVertical(
417:           unpacked_output,
418:           unpacked_input,
419:           ksize_vert,
420:           vert_indices_weights,
421:           vert_weights_precision
422:       );
423:     }
424:
425:     TORCH_INTERNAL_ASSERT(unpacked_output.defined());
426:
427:     if (!skip_packing) {
428:       pack_rgb(unpacked_output, output[i]);
429:     }
430:   }
431: }
432:
433: void ImagingResampleHorizontalConvolution8u4x(
434:     uint8_t* C10_RESTRICT lineOut0,
435:     uint8_t* C10_RESTRICT lineOut1,
436:     uint8_t* C10_RESTRICT lineOut2,
437:     uint8_t* C10_RESTRICT lineOut3,
438:     int64_t out_xsize,
439:     const uint8_t* C10_RESTRICT lineIn0,
440:     const uint8_t* C10_RESTRICT lineIn1,
441:     const uint8_t* C10_RESTRICT lineIn2,
442:     const uint8_t* C10_RESTRICT lineIn3,
443:     int64_t in_xsize,
444:     const int64_t* idx_ptr_xmin,
445:     const int64_t* idx_ptr_size,
446:     const int16_t* kk,
447:     int kmax,
448:     unsigned int coefs_precision,
449:     int64_t num_channels,
450:     bool is_last_line) {
```
- EN: The main symbol in this range is `ImagingResampleHorizontalConvolution8u4x`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `ImagingResampleHorizontalConvolution8u4x`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 452-482
```cpp
452:   // Interpolation horizontal pass processing together 4 vertical lines.
453:   // - Input data format is RGBA or RGB with R,G,B,A being uint8. In case of RGBA
454:   //   we can encode 4 values as a single uint32 value.
455:   // - We split the size of weight vector for a given output index as a sum:
456:   //   ids_size = num_blocks_4 * 4 + num_blocks_2 * 2 + num_blocks_1.
457:   // - We load and process 4 weights values in a loop ("block 4") then we process 2 weights values
458:   // in another loop ("block 2") and finally we process 1 weights value in the final loop ("block 1").
459:
460:   // Define shuffling masks (low/high) for num_channels 4 and 3
461:   // Mask low casts lower half of each lane to epi16 and reorder RGBARGBA -> RRGGBBAA:
462:   //   [r1 g1 b1 a1  r2 g2 b2 a2  ... | R1 G1 B1 A1  R2 G2 B2 A2 ... ] ->
463:   //   [r1 0 r2 0  g1 0 g2 0  b1 0 b2 0  a1 0 a2 0 | R1 0 R2 0  G1 0 G2 0  B1 0 B2 0  A1 0 A2 0]
464:   // Mask high casts upper half of each lane to epi16 and reorder RGBARGBA -> RRGGBBAA::
465:   //   [ ... r3 g3 b3 a3  r4 g4 b4 a4 | ... R3 G3 B3 A3  R4 G4 B4 A4 ] ->
466:   //   [r3 0 r4 0  g3 0 g4 0  b3 0 b4 0  a3 0 a4 0 | R3 0 R4 0  G3 0 G4 0  B3 0 B4 0  A3 0 A4 0]
467:
468:   const auto mask_low_c4 = _mm256_set_epi8(
469:       -1, 7, -1, 3, -1, 6, -1, 2, -1, 5, -1, 1, -1, 4, -1, 0,
470:       -1, 7, -1, 3, -1, 6, -1, 2, -1, 5, -1, 1, -1, 4, -1, 0);
471:   const auto mask_high_c4 = _mm256_set_epi8(
472:       -1, 15, -1, 11, -1, 14, -1, 10, -1, 13, -1, 9, -1, 12, -1, 8,
473:       -1, 15, -1, 11, -1, 14, -1, 10, -1, 13, -1, 9, -1, 12, -1, 8);
474:   const auto mask_low_c3 = _mm256_set_epi8(
475:       -1, -1, -1, -1, -1, 5, -1, 2, -1, 4, -1, 1, -1, 3, -1, 0,
476:       -1, -1, -1, -1, -1, 5, -1, 2, -1, 4, -1, 1, -1, 3, -1, 0);
477:   const auto mask_high_c3 = _mm256_set_epi8(
478:       -1, -1, -1, -1, -1, 11, -1, 8, -1, 10, -1, 7, -1, 9, -1, 6,
479:       -1, -1, -1, -1, -1, 11, -1, 8, -1, 10, -1, 7, -1, 9, -1, 6);
480:
481:   const auto mask_low = (num_channels == 3) ? mask_low_c3 : mask_low_c4;
482:   const auto mask_high = (num_channels == 3) ? mask_high_c3 : mask_high_c4;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 484-518
```cpp
484:   const auto stride = num_channels * sizeof(uint8_t);
485:
486:   TORCH_INTERNAL_ASSERT(stride == 3 || stride == 4);
487:
488:   // out_xsize = output width, out_x = output x index
489:   // ids_min is the input offset index corresponding to out_x
490:   // ids_size is the interpolation size for out_x
491:
492:   // Let's precompute ids_size limits for block 4 and block 2.
493:   //
494:   // In block 4 (4 means we process 4 weight values together), we read input data
495:   // with _mm_loadu_si128, i.e. 16 bytes, per one line:
496:   // lineIn0 + stride * (i + ids_min) + 16 <= lineIn0 + stride * (ids_size + ids_min)
497:   // --> i <= ids_size - 16.0 / stride
498:   // Strict boundary:
499:   // --> i < ids_size + 1 - int(ceil(16.0 / stride)) = ids_size - b4_delta
500:   // Soft boundary for reading inside the buffer except its boundaries:
501:   // --> i < ids_size + 1 - int(16.0 / stride) = ids_size - b4_delta_soft
502:   // RGBA: b4_delta = b4_delta_soft = 3
503:   // RGB : b4_delta = 5
504:   // RGB : b4_delta_soft = 4
505:   const auto b4_delta = (stride == 4) ? 3 : (is_last_line ? 5 : 4);
506:
507:   // In block 2 (2 means we process 2 weights values together), we read input data
508:   // with _mm_loadl_epi64, i.e. 8 bytes, per one line:
509:   // lineIn0 + stride * (i + ids_min) + 8 <= lineIn0 + stride * (ids_size + ids_min)
510:   // --> i <= ids_size - 8.0 / stride
511:   // Strict boundary:
512:   // --> i < ids_size + 1 - int(ceil(8.0 / stride)) = ids_size - b2_delta
513:   // Soft boundary for reading inside the buffer except its boundaries:
514:   // --> i < ids_size + 1 - int(8.0 / stride) = ids_size - b2_delta_soft
515:   // RGBA: b2_delta = b2_delta_soft = 1
516:   // RGB : b2_delta = 2
517:   // RGB : b2_delta_soft = 1
518:   const auto b2_delta = (stride == 4) ? 1 : (is_last_line ? 2 : 1);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 520-560
```cpp
520:   const auto max_out_x_strided = out_xsize * stride;
521:   const auto max_in_x_strided = in_xsize * stride;
522:
523:   const auto zero = _mm256_setzero_si256();
524:   const auto initial = _mm256_set1_epi32(1 << (coefs_precision - 1));
525:
526:   for (const auto out_x : c10::irange(out_xsize)) {
527:     const auto ids_min = idx_ptr_xmin[out_x];
528:     const auto ids_size = idx_ptr_size[out_x];
529:     const auto * k = &kk[out_x * kmax];
530:     int64_t i = 0;
531:
532:     auto sss0 = initial;
533:     auto sss1 = initial;
534:
535:     const auto * lineIn0_min = lineIn0 + ids_min;
536:     const auto * lineIn1_min = lineIn1 + ids_min;
537:     const auto * lineIn2_min = lineIn2 + ids_min;
538:     const auto * lineIn3_min = lineIn3 + ids_min;
539:
540:     // block 4
541:     for (; i < ids_size - b4_delta; i += 4) {
542:       // Load 4 values from weight vector
543:       // mmk0 = [wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ...]
544:       // mmk1 = [wl_2 wh_2 wl_3 wh_3  wl_2 wh_2 wl_3 wh_3  ...]
545:       const auto mmk0 = _mm256_set1_epi32(*(int32_t*)&k[i]);
546:       const auto mmk1 = _mm256_set1_epi32(*(int32_t*)&k[i + 2]);
547:
548:       // RGBA: Load 8 pixels (4 per line) from input lines 0 and 1:
549:       // source = [
550:       //   r0 g0 b0 a0  r1 g1 b1 a1  r2 g2 b2 a2  r3 g3 b3 a3
551:       //   R0 G0 B0 A0  R1 G1 B1 A1  R2 G2 B2 A2  R3 G3 B3 A3
552:       // ]
553:       // RGB: Load 10 pixels (5 per line)
554:       // source = [
555:       //   r0 g0 b0 r1  g1 b1 r2 g2  b2 r3 g3 b3  r4 g4 b4 r5
556:       //   R0 G0 B0 R1  G1 B1 R2 G2  B2 R3 G3 B3  R4 G4 B4 R5
557:       // ]
558:       auto source = _mm256_inserti128_si256(_mm256_castsi128_si256(
559:           _mm_loadu_si128((__m128i *) (lineIn0_min + stride * i))),
560:           _mm_loadu_si128((__m128i *) (lineIn1_min + stride * i)), 1);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 562-594
```cpp
562:       // Apply mask_low:
563:       // RGBA:
564:       //   [r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  a0 0 a1 0 | R0 0 R1 0  G0 0 G1 0  B0 0 B1 0  A0 0 A1 0]
565:       // RGB:
566:       //   [r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  0 0 0 0 | R0 0 R1 0  G0 0 G1 0  B0 0 B1 0  0 0 0 0]
567:       auto pix1 = _mm256_shuffle_epi8(source, mask_low);
568:       // Compute output value as C += w0 * C0 + w1 * C1 for each channel in 32-bit precision
569:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix1, mmk0));
570:
571:       // Apply mask_high:
572:       // RGBA:
573:       //   [r2 0 r3 0  g2 0 g3 0  b2 0 b3 0  a2 0 a3 0 | R2 0 R3 0  G2 0 G3 0  B2 0 B3 0  A2 0 A3 0]
574:       // RGB:
575:       //   [r2 0 r3 0  g2 0 g3 0  b2 0 b3 0  0 0 0 0 | R2 0 R3 0  G2 0 G3 0  B2 0 B3 0  0 0 0 0]
576:       auto pix2 = _mm256_shuffle_epi8(source, mask_high);
577:       // Compute output value as C += w2 * C2 + w3 * C3 for each channel in 32-bit precision
578:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix2, mmk1));
579:
580:       // Same as above to next lines 2 and 3:
581:       auto source2 = _mm256_inserti128_si256(_mm256_castsi128_si256(
582:           _mm_loadu_si128((__m128i *) (lineIn2_min + stride * i))),
583:           _mm_loadu_si128((__m128i *) (lineIn3_min + stride * i)), 1);
584:       auto pix3 = _mm256_shuffle_epi8(source2, mask_low);
585:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix3, mmk0));
586:       auto pix4 = _mm256_shuffle_epi8(source2, mask_high);
587:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix4, mmk1));
588:     }
589:
590:     // block 2
591:     for (; i < ids_size - b2_delta; i += 2) {
592:       // Load 2 values from weight vector
593:       // mmk = [wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ...]
594:       const auto mmk = _mm256_set1_epi32(*(int32_t*)&k[i]);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 596-630
```cpp
596:       // Load 4 pixels (2 per line) from input lines 0 and 1:
597:       // RGBA: source1 = [
598:       //   r0 g0 b0 a0  r1 g1 b1 a1  0 0 0 0  0 0 0 0
599:       //   R0 G0 B0 A0  R1 G1 B1 A1  0 0 0 0  0 0 0 0
600:       // ]
601:       // RGB: source1 = [
602:       //   r0 g0 b0 r1  g1 b1 r2  0 0 0 0  0 0 0 0
603:       //   R0 G0 B0 R1  G1 B1 R2  0 0 0 0  0 0 0 0
604:       // ]
605:       auto source1 = _mm256_inserti128_si256(_mm256_castsi128_si256(
606:           _mm_loadl_epi64((__m128i *) (lineIn0_min + stride * i))),
607:           _mm_loadl_epi64((__m128i *) (lineIn1_min + stride * i)), 1);
608:       // Apply mask_low:
609:       // RGBA:
610:       //   [r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  a0 0 a1 0 | R0 0 R1 0  G0 0 G1 0  B0 0 B1 0  A0 0 A1 0]
611:       // RGB:
612:       //   [r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  0 0 0 0 | R0 0 R1 0  G0 0 G1 0  B0 0 B1 0  0 0 0 0]
613:       auto pix1 = _mm256_shuffle_epi8(source1, mask_low);
614:       // Compute output value as C += w0 * C0 + w1 * C1 for each channel in 32-bit precision
615:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix1, mmk));
616:
617:       // Same as above for lines 2 and 3:
618:       auto source2 = _mm256_inserti128_si256(_mm256_castsi128_si256(
619:           _mm_loadl_epi64((__m128i *) (lineIn2_min + stride * i))),
620:           _mm_loadl_epi64((__m128i *) (lineIn3_min + stride * i)), 1);
621:       auto pix2 = _mm256_shuffle_epi8(source2, mask_low);
622:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix2, mmk));
623:     }
624:
625:     // block 1
626:     const auto i32_aligned = num_channels == 4;
627:     for (; i < ids_size - 1; i++) {
628:       // Load 1 value from weight vector
629:       // mmk = [wl_0 wh_0 0 0  wl_0 wh_0 0 0  ...]
630:       const auto mmk = _mm256_set1_epi32(k[i]);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 632-662
```cpp
632:       // Load 2 pixels (one per line) from input lines 0 and 1:
633:       // RGBA: pix1 = [
634:       //   r0 0 0 0  g0 0 0 0  b0 0 0 0  a0 0 0 0
635:       //   R0 0 0 0  G0 0 0 0  B0 0 0 0  A0 0 0 0
636:       // ]
637:       // RGB: pix1 = [
638:       //   r0 0 0 0  g0 0 0 0  b0 0 0 0  r1 0 0 0
639:       //   R0 0 0 0  G0 0 0 0  B0 0 0 0  R1 0 0 0
640:       // ]
641:       auto pix1 = _mm256_inserti128_si256(_mm256_castsi128_si256(
642:           mm_cvtepu8_epi32(lineIn0_min + stride * i, i32_aligned)),
643:           mm_cvtepu8_epi32(lineIn1_min + stride * i, i32_aligned), 1);
644:       // Compute output value as C += w0 * C0 for each channel in 32-bit precision
645:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix1, mmk));
646:
647:       // Same as above for lines 2 and 3
648:       auto pix2 = _mm256_inserti128_si256(_mm256_castsi128_si256(
649:           mm_cvtepu8_epi32(lineIn2_min + stride * i, i32_aligned)),
650:           mm_cvtepu8_epi32(lineIn3_min + stride * i, i32_aligned), 1);
651:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix2, mmk));
652:     }
653:
654:     if (i == ids_size - 1) {
655:       // last element
656:       auto mmk = _mm256_set1_epi32(k[i]);
657:       // For num_channels == 3 (3 bytes = one pixel) we tolerate to read 4 bytes
658:       // lines 0, 1 and 2 won't go out of allocated memory bounds
659:       auto pix = _mm256_inserti128_si256(_mm256_castsi128_si256(
660:           mm_cvtepu8_epi32(lineIn0_min + stride * i, i32_aligned)),
661:           mm_cvtepu8_epi32(lineIn1_min + stride * i, i32_aligned), 1);
662:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix, mmk));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 664-694
```cpp
664:       auto p0 = mm_cvtepu8_epi32(lineIn2_min + stride * i, i32_aligned);
665:       __m128i p1;
666:       if (num_channels == 3 && C10_UNLIKELY(is_last_line && ids_min + stride * i + 4 >= max_in_x_strided)) {
667:         uint8_t input[4];
668:         std::memcpy(input, lineIn3_min + stride * i, 3);
669:         p1 = mm_cvtepu8_epi32(input, true);
670:       } else {
671:         p1 = mm_cvtepu8_epi32(lineIn3_min + stride * i, i32_aligned);
672:       }
673:       auto pix2 = _mm256_inserti128_si256(_mm256_castsi128_si256(p0), p1, 1);
674:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix2, mmk));
675:     }
676:
677:     // Convert fixed point values back to integers (truncating)
678:     sss0 = _mm256_srai_epi32(sss0, coefs_precision);
679:     sss1 = _mm256_srai_epi32(sss1, coefs_precision);
680:     // Convert packed signed 32-bit integers to packed 16-bit integers using signed saturation
681:     // (a a a a b b b b c c c c d d d d) -> (a a b b c c d d 0 0 0 0 0 0 0 0)
682:     sss0 = _mm256_packs_epi32(sss0, zero);
683:     sss1 = _mm256_packs_epi32(sss1, zero);
684:     // Convert packed signed 16-bit integers to packed 8-bit integers using unsigned saturation
685:     // (a a b b c c d d) -> (a b c d 0 0 0 0)
686:     sss0 = _mm256_packus_epi16(sss0, zero);
687:     sss1 = _mm256_packus_epi16(sss1, zero);
688:
689:     // Write the output into single uint32
690:     // (a b c d) -> x_uint32
691:     auto o0 = _mm_cvtsi128_si32(_mm256_castsi256_si128(sss0));
692:     auto o1 = _mm_cvtsi128_si32(_mm256_extracti128_si256(sss0, 1));
693:     auto o2 = _mm_cvtsi128_si32(_mm256_castsi256_si128(sss1));
694:     auto o3 = _mm_cvtsi128_si32(_mm256_extracti128_si256(sss1, 1));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 696-737
```cpp
696:     const auto out_x_strided = stride * out_x;
697:
698:     if (num_channels == 3 && C10_UNLIKELY(out_x_strided + 4 >= max_out_x_strided)) {
699:       // Memcpy 4-bytes is faster than 3-bytes and this is a boundary case when we want to write
700:       // 4 bytes (R G B | X) to the output buffer (X1 X2 X3 | R1).
701:       // The 4th byte in the register (X) has a garbage value and 4th byte in the output buffer (R1) has a correct
702:       // value which was previously computed by another line. In other words, it means that we can not overwrite
703:       // it by simply writing 4 bytes from the register to the output. We'll do the following:
704:       //               v----------|
705:       // Output = [... X1 X2 X3 | R1 G1 B1 R2 ...]
706:       // First, we write R1 value to the 4th byte of (R G B | X) -> (R G B | R1)
707:       // Second, we write 4 bytes from the register to the output: (X1 X2 X3 | R1) -> (R G B | R1)
708:       // Output = [... R G B | R1 G1 B1 R2 ...]
709:
710:       _write_endline_rgb_as_uint32(lineOut0 + out_x_strided, o0);
711:       _write_endline_rgb_as_uint32(lineOut1 + out_x_strided, o1);
712:       _write_endline_rgb_as_uint32(lineOut2 + out_x_strided, o2);
713:
714:       if (C10_UNLIKELY(is_last_line)) {
715:         // When we handle the last line, we can not access the next 4 bytes
716:         // as they are out of memory bounds.
717:         std::memcpy(lineOut3 + out_x_strided, (uint8_t *) &o3, num_channels);
718:       } else {
719:         _write_endline_rgb_as_uint32(lineOut3 + out_x_strided, o3);
720:       }
721:     } else if (num_channels == 3) {
722:       // Memcpy 4-bytes is faster than 3-bytes and here
723:       // we simply write 4 bytes (... R G B X 0 0 0 0 0 ...) where X is a garbage value
724:       // that we will overwrite on the next iteration: (... R G B R G B X 0 0 ...)
725:       std::memcpy(lineOut0 + out_x_strided, (uint8_t *) &o0, 4);
726:       std::memcpy(lineOut1 + out_x_strided, (uint8_t *) &o1, 4);
727:       std::memcpy(lineOut2 + out_x_strided, (uint8_t *) &o2, 4);
728:       std::memcpy(lineOut3 + out_x_strided, (uint8_t *) &o3, 4);
729:     } else {
730:       // num_channels = 4 -> lineOutX + out_x_strided should be uint32 aligned
731:       *(uint32_t *)(lineOut0 + out_x_strided) = o0;
732:       *(uint32_t *)(lineOut1 + out_x_strided) = o1;
733:       *(uint32_t *)(lineOut2 + out_x_strided) = o2;
734:       *(uint32_t *)(lineOut3 + out_x_strided) = o3;
735:     }
736:   }
737: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 739-769
```cpp
739: void ImagingResampleHorizontalConvolution8u(
740:     uint8_t* C10_RESTRICT lineOut,
741:     int64_t out_xsize,
742:     const uint8_t* C10_RESTRICT lineIn,
743:     int64_t in_xsize,
744:     const int64_t* idx_ptr_xmin,
745:     const int64_t* idx_ptr_size,
746:     const int16_t* kk,
747:     int kmax,
748:     unsigned int coefs_precision,
749:     int64_t num_channels,
750:     bool is_last_line) {
751:
752:   // Interpolation horizontal pass processing only one vertical line.
753:   // - Input data format is RGBA or RGB with R,G,B,A being uint8. In case of RGBA
754:   //   we can encode 4 values as a single uint32 value.
755:   // - We split the size of weight vector for a given output index as a sum:
756:   //   ids_size = num_blocks_8 * 8 + num_blocks_4 * 4 + num_blocks_2 * 2 + num_blocks_1
757:   // - We load and process 8 weights values in a loop ("block 8") then 4 weights and 2 weights values in
758:   // in another loops ("block 4" and "block 2") and finally we process 1 weight value in the final loop ("block 1").
759:
760:   // Define various shuffling masks
761:   const auto kmask_low = _mm256_set_epi8(
762:       11, 10, 9, 8, 11, 10, 9, 8, 11, 10, 9, 8, 11, 10, 9, 8,
763:       3, 2, 1, 0, 3, 2, 1, 0, 3, 2, 1, 0, 3, 2, 1, 0);
764:   const auto kmask_high = _mm256_set_epi8(
765:       15, 14, 13, 12, 15, 14, 13, 12, 15, 14, 13, 12, 15, 14, 13, 12,
766:       7, 6, 5, 4, 7, 6, 5, 4, 7, 6, 5, 4, 7, 6, 5, 4);
767:   const auto kmask_hl = _mm256_set_epi8(
768:       7, 6, 5, 4, 7, 6, 5, 4, 7, 6, 5, 4, 7, 6, 5, 4,
769:       3, 2, 1, 0, 3, 2, 1, 0, 3, 2, 1, 0, 3, 2, 1, 0);
```
- EN: The main symbol in this range is `ImagingResampleHorizontalConvolution8u`, which contributes directly to this file's operator logic. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `ImagingResampleHorizontalConvolution8u`，它们直接构成本文件的算子逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 771-802
```cpp
771:   const auto mask_low_c4 = _mm256_set_epi8(
772:       -1, 7, -1, 3, -1, 6, -1, 2, -1, 5, -1, 1, -1, 4, -1, 0,
773:       -1, 7, -1, 3, -1, 6, -1, 2, -1, 5, -1, 1, -1, 4, -1, 0);
774:   const auto mask_high_c4 = _mm256_set_epi8(
775:       -1, 15, -1, 11, -1, 14, -1, 10, -1, 13, -1, 9, -1, 12, -1, 8,
776:       -1, 15, -1, 11, -1, 14, -1, 10, -1, 13, -1, 9, -1, 12, -1, 8);
777:   const auto mask_low_c3 = _mm256_set_epi8(
778:       -1, -1, -1, -1, -1, 5, -1, 2, -1, 4, -1, 1, -1, 3, -1, 0,
779:       -1, -1, -1, -1, -1, 5, -1, 2, -1, 4, -1, 1, -1, 3, -1, 0);
780:   const auto mask_high_c3 = _mm256_set_epi8(
781:       -1, -1, -1, -1, -1, 11, -1, 8, -1, 10, -1, 7, -1, 9, -1, 6,
782:       -1, -1, -1, -1, -1, 11, -1, 8, -1, 10, -1, 7, -1, 9, -1, 6);
783:   const auto mask_hl_c3 = _mm256_set_epi8(
784:       -1, -1, -1, -1, -1, 11, -1, 8, -1, 10, -1, 7, -1, 9, -1, 6,
785:       -1, -1, -1, -1, -1, 5, -1, 2, -1, 4, -1, 1, -1, 3, -1, 0);
786:   const auto mask_hl_c4 = _mm256_set_epi8(
787:       -1, 15, -1, 11, -1, 14, -1, 10, -1, 13, -1, 9, -1, 12, -1, 8,
788:       -1, 7, -1, 3, -1, 6, -1, 2, -1, 5, -1, 1, -1, 4, -1, 0);
789:
790:   const auto mask_low128_c3 = _mm_set_epi8(
791:       -1, -1, -1, -1, -1, 5, -1, 2, -1, 4, -1, 1, -1, 3, -1, 0);
792:   const auto mask_low128_c4 = _mm_set_epi8(
793:       -1, 7, -1, 3, -1, 6, -1, 2, -1, 5, -1, 1, -1, 4, -1, 0);
794:
795:   const auto mask_low = (num_channels == 3) ? mask_low_c3 : mask_low_c4;
796:   const auto mask_high = (num_channels == 3) ? mask_high_c3 : mask_high_c4;
797:   const auto mask_hl = (num_channels == 3) ? mask_hl_c3 : mask_hl_c4;
798:   const auto mask_low128 = (num_channels == 3) ? mask_low128_c3 : mask_low128_c4;
799:
800:   // out_xsize = output width, out_x = output x index
801:   // ids_min is the input offset index corresponding to out_x
802:   // ids_size is the interpolation size for out_x
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 804-835
```cpp
804:   const auto stride = num_channels * sizeof(uint8_t);
805:   const auto zero = _mm_setzero_si128();
806:
807:   TORCH_INTERNAL_ASSERT(stride == 3 || stride == 4);
808:
809:   // Let's precompute ids_size limits for block 8, block 4 and block 2
810:   //
811:   // In block 8 (8 means we process 8 weight values together), we read at
812:   // most 32 bytes input data (16 + 16 bytes for RGBA and 12 + 16 bytes for RGB)
813:   // lineIn + stride * (i + ids_min) + 32 <= lineIn + stride * (ids_size + ids_min)
814:   // --> i <= ids_size - 32.0 / stride
815:   // Strict boundary:
816:   // --> i < ids_size + 1 - int(ceil(32.0 / stride)) = ids_size - b8_delta
817:   // Soft boundary for reading inside the buffer except its boundaries:
818:   // --> i < ids_size + 1 - int(32.0 / stride) = ids_size - b8_delta_soft
819:   // RGBA: b8_delta = b8_delta_soft = 7
820:   // RGB : b8_delta = 10
821:   // RGB : b8_delta_soft = 9
822:   const auto b8_delta = (stride == 4) ? 7 : (is_last_line ? 10 : 9);
823:
824:   // In block 4 (4 means we process 4 weight values together), we read
825:   // 16 bytes of input data.
826:   // lineIn + stride * (i + ids_min) + 16 <= lineIn0 + stride * (ids_size + ids_min)
827:   // --> i <= ids_size - 16.0 / stride
828:   // Strict boundary:
829:   // --> i < ids_size + 1 - int(ceil(16.0 / stride)) = ids_size - b4_delta
830:   // Soft boundary for reading inside the buffer except its boundaries:
831:   // --> i < ids_size + 1 - int(16.0 / stride) = ids_size - b4_delta_soft
832:   // RGBA: b4_delta = b4_delta_soft = 3
833:   // RGB : b4_delta = 5
834:   // RGB : b4_delta_soft = 4
835:   const auto b4_delta = (stride == 4) ? 3 : (is_last_line ? 5 : 4);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 837-866
```cpp
837:   // In block 2 (2 means we process 2 weight values together), we read
838:   // 8 bytes of input data.
839:   // lineIn0 + stride * (i + ids_min) + 8 <= lineIn0 + stride * (ids_size + ids_min)
840:   // --> i <= ids_size - 8.0 / stride
841:   // Strict boundary:
842:   // --> i < ids_size + 1 - int(ceil(8.0 / stride)) = ids_size - b2_delta
843:   // Soft boundary for reading inside the buffer except its boundaries:
844:   // --> i < ids_size + 1 - int(8.0 / stride) = ids_size - b2_delta_soft
845:   // RGBA: b2_delta = b2_delta_soft = 1
846:   // RGB : b2_delta = 2
847:   // RGB : b2_delta_soft = 1
848:   const auto b2_delta = (stride == 4) ? 1 : (is_last_line ? 2 : 1);
849:
850:   const auto max_out_x_strided = out_xsize * stride;
851:   const auto max_in_x_strided = in_xsize * stride;
852:
853:   for (const auto out_x : c10::irange(out_xsize)) {
854:     __m128i sss;
855:     const auto ids_min = idx_ptr_xmin[out_x];
856:     const auto ids_size = idx_ptr_size[out_x];
857:     const auto * k = &kk[out_x * kmax];
858:     int64_t i = 0;
859:
860:     const auto * lineIn_min = lineIn + ids_min;
861:
862:     if (ids_size < 8) {
863:       sss = _mm_set1_epi32(1 << (coefs_precision - 1));
864:     } else {
865:       // Lower part will be added to higher, use only half of the error
866:       auto sss256 = _mm256_set1_epi32(1 << (coefs_precision - 2));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 868-910
```cpp
868:       // block 8
869:       for (; i < ids_size - b8_delta; i += 8) {
870:         // Load 8 values from weight vector
871:         auto tmp = _mm_loadu_si128((__m128i*)&k[i]);
872:         // ksource = [
873:         //    wl_0 wh_0 wl_1 wh_1  wl_2 wh_2 wl_3 wh_3  wl_4 wh_4 wl_5 wh_5  wl_6 wh_6 wl_7 wh_7
874:         //    wl_0 wh_0 wl_1 wh_1  wl_2 wh_2 wl_3 wh_3  wl_4 wh_4 wl_5 wh_5  wl_6 wh_6 wl_7 wh_7
875:         // ]
876:         auto ksource = _mm256_insertf128_si256(_mm256_castsi128_si256(tmp), tmp, 1);
877:
878:         // RGBA: Load 8 pixels from input:
879:         // source = [
880:         //    r0 g0 b0 a0  r1 g1 b1 a1  r2 g2 b2 a2  r3 g3 b3 a3
881:         //    r4 g4 b4 a4  r5 g5 b5 a5  r6 g6 b6 a6  r7 g7 b7 a7
882:         // ]
883:         // RGB: Load 10 pixels from input (however we can process only 8 pixels):
884:         // source = [
885:         //    r0 g0 b0 r1  g1 b1 r2 g2  b2 r3 g3 b3  r4 g4 b4 r5
886:         //    r4 g4 b4 r5  g5 b5 r6 g6  b6 r7 g7 b7  r8 g8 b8 r9
887:         // ]
888:         auto source = _mm256_inserti128_si256(_mm256_castsi128_si256(
889:             _mm_loadu_si128((__m128i *) (lineIn_min + stride * i))),
890:             _mm_loadu_si128((__m128i *) (lineIn_min + stride * (i + 4))), 1);
891:
892:         // Extract lower part of each lane, cast to epi16 and reorder RGBARGBA -> RRGGBBAA
893:         // RGBA: pix1 = [
894:         //   r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  a0 0 a1 0
895:         //   r4 0 r5 0  g4 0 g5 0  b4 0 b5 0  a4 0 a5 0
896:         // ]
897:         // RGB: pix1 = [
898:         //   r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  0 0 0 0
899:         //   r4 0 r5 0  g4 0 g5 0  b4 0 b5 0  0 0 0 0
900:         // ]
901:         auto pix1 = _mm256_shuffle_epi8(source, mask_low);
902:         // mmk1 = [
903:         //   wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ...  ...
904:         //   wl_4 wh_4 wl_5 wh_5  wl_4 wh_4 wl_5 wh_5  ...  ...
905:         // ]
906:         auto mmk1 = _mm256_shuffle_epi8(ksource, kmask_low);
907:         // Compute output value as
908:         //   C += w0 * C0 + w1 * C1
909:         //   C += w4 * C4 + w5 * C5 for each channel in 32-bit precision
910:         sss256 = _mm256_add_epi32(sss256, _mm256_madd_epi16(pix1, mmk1));
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 912-941
```cpp
912:         // Same as above for higher part of each lane
913:         auto pix2 = _mm256_shuffle_epi8(source, mask_high);
914:         auto mmk2 = _mm256_shuffle_epi8(ksource, kmask_high);
915:         // Compute output value as
916:         //    C += w2 * C2 + w3 * C3
917:         //    C += w6 * C6 + w7 * C7 for each channel in 32-bit precision
918:         sss256 = _mm256_add_epi32(sss256, _mm256_madd_epi16(pix2, mmk2));
919:       }
920:
921:       // block 4
922:       for (; i < ids_size - b4_delta; i += 4) {
923:         // Load 4 values from weight vector
924:         auto tmp = _mm_loadl_epi64((__m128i *) &k[i]);
925:         // ksource = [
926:         //    wl_0 wh_0 wl_1 wh_1  wl_2 wh_2 wl_3 wh_3  0 0 0 0  0 0 0 0
927:         //    wl_0 wh_0 wl_1 wh_1  wl_2 wh_2 wl_3 wh_3  0 0 0 0  0 0 0 0
928:         // ]
929:         auto ksource = _mm256_insertf128_si256(_mm256_castsi128_si256(tmp), tmp, 1);
930:
931:         // Load pixels from input line
932:         tmp = _mm_loadu_si128((__m128i *) (lineIn_min + stride * i));
933:         // RGBA: source = [
934:         //   r0 g0 b0 a0  r1 g1 b1 a1  r2 g2 b2 a2  r3 g3 b3 a3
935:         //   r0 g0 b0 a0  r1 g1 b1 a1  r2 g2 b2 a2  r3 g3 b3 a3
936:         // ]
937:         // RGB: source = [
938:         //   r0 g0 b0 r1  g1 b1 r2 g2  b2 r3 g3 b3  r4 g4 b4 r5
939:         //   r0 g0 b0 r1  g1 b1 r2 g2  b2 r3 g3 b3  r4 g4 b4 r5
940:         // ]
941:         auto source = _mm256_insertf128_si256(_mm256_castsi128_si256(tmp), tmp, 1);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 943-971
```cpp
943:         // Cast source to epi16 and reorder RGBARGBA -> RRGGBBAA
944:         // RGBA: pix = [
945:         //   r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  a0 0 a1 0
946:         //   r2 0 r3 0  g2 0 g3 0  b2 0 b3 0  a2 0 a3 0
947:         // ]
948:         // RGB: pix = [
949:         //   r0 0 r1 0  g0 0 g1 0  b0 0 b1 0  0 0 0 0
950:         //   r2 0 r3 0  g2 0 g3 0  b2 0 b3 0  0 0 0 0
951:         // ]
952:         auto pix = _mm256_shuffle_epi8(source, mask_hl);
953:         // mmk = [
954:         //   wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ... ...
955:         //   wl_2 wh_2 wl_3 wh_3  wl_2 wh_2 wl_3 wh_3  ... ...
956:         // ]
957:         auto mmk = _mm256_shuffle_epi8(ksource, kmask_hl);
958:         // Compute output value as
959:         //   C += w0 * C0 + w1 * C1
960:         //   C += w2 * C2 + w3 * C3 for each channel in 32-bit precision
961:         sss256 = _mm256_add_epi32(sss256, _mm256_madd_epi16(pix, mmk));
962:       }
963:
964:       // Sum results between the lanes
965:       sss = _mm_add_epi32(
966:           _mm256_extracti128_si256(sss256, 0),
967:           _mm256_extracti128_si256(sss256, 1));
968:     }
969:
970:     // block 2
971:     for (; i < ids_size - b2_delta; i += 2) {
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 972-1005
```cpp
 972:       // Load 2 values from weight vector
 973:       // mmk = [wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ...]
 974:       auto mmk = _mm_set1_epi32(*(int32_t*)&k[i]);
 975:       // Load pixels from input line
 976:       // RGBA: source = [
 977:       //   r0 g0 b0 a0  r1 g1 b1 a1  0 0 0 0  0 0 0 0
 978:       // ]
 979:       // RGB: source = [
 980:       //   r0 g0 b0 r1  g1 b1 r2 g2  0 0 0 0  0 0 0 0
 981:       // ]
 982:       auto source = _mm_loadl_epi64((__m128i *) (lineIn_min + stride * i));
 983:       // Cast source to epi16 and reorder RGBARGBA -> RRGGBBAA
 984:       auto pix = _mm_shuffle_epi8(source, mask_low128);
 985:       // Compute output value as C += w0 * C0 + w1 * C1 for each channel in 32-bit precision
 986:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
 987:     }
 988:
 989:     // block 1
 990:     const auto i32_aligned = num_channels == 4;
 991:     for (; i < ids_size - 1; i++) {
 992:       // Load 1 value from weight vector
 993:       // mmk = [wl_0 wh_0 0 0  wl_0 wh_0 0 0  ...]
 994:       auto mmk = _mm_set1_epi32(k[i]);
 995:       // Load one pixel from input line
 996:       // RGBA: pix = [
 997:       //   r0 0 0 0  g0 0 0 0  b0 0 0 0  a0 0 0 0
 998:       // ]
 999:       // RGB: pix = [
1000:       //   r0 0 0 0  g0 0 0 0  b0 0 0 0  r1 0 0 0
1001:       // ]
1002:       auto pix = mm_cvtepu8_epi32(lineIn_min + stride * i, i32_aligned);
1003:       // Compute output value as C += w0 * C0 for each channel in 32-bit precision
1004:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
1005:     }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1007-1035
```cpp
1007:     if (i == ids_size - 1) {
1008:       // last element
1009:       auto mmk = _mm_set1_epi32(k[i]);
1010:       __m128i pix;
1011:       auto p = lineIn_min + stride * i;
1012:       if (num_channels == 3 && C10_UNLIKELY(is_last_line && ids_min + stride * i + 4 >= max_in_x_strided)) {
1013:         uint8_t input[4];
1014:         std::memcpy(input, p, 3);
1015:         pix = mm_cvtepu8_epi32(input, true);
1016:       } else {
1017:         pix = mm_cvtepu8_epi32(p, i32_aligned);
1018:       }
1019:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
1020:     }
1021:
1022:     // Convert fixed point values back to integers (truncating)
1023:     sss = _mm_srai_epi32(sss, coefs_precision);
1024:     // Convert packed signed 32-bit integers to packed 16-bit integers using signed saturation
1025:     // (a a a a b b b b c c c c d d d d) -> (a a b b c c d d 0 0 0 0 0 0 0 0)
1026:     sss = _mm_packs_epi32(sss, zero);
1027:     // Convert packed signed 16-bit integers to packed 8-bit integers using unsigned saturation
1028:     // (a a b b c c d d) -> (a b c d 0 0 0 0)
1029:     sss = _mm_packus_epi16(sss, zero);
1030:     // Write the output into single uint32
1031:     // (a b c d) -> x_uint32
1032:     auto o = _mm_cvtsi128_si32(sss);
1033:     const auto out_x_strided = stride * out_x;
1034:     if (num_channels == 3 && C10_UNLIKELY(out_x_strided + 4 >= max_out_x_strided)) {
1035:       if (C10_UNLIKELY(is_last_line)) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1036-1072
```cpp
1036:         // When we handle the last line, we can not access the next 4 bytes
1037:         // as they are out of memory bounds.
1038:         std::memcpy(lineOut + out_x_strided, (uint8_t *) &o, 3);
1039:       } else {
1040:         // Memcpy 4-bytes is faster than 3-bytes and this is a boundary case when we want to write
1041:         // 4 bytes (R G B | X) to the output buffer (X1 X2 X3 | R1).
1042:         // The 4th byte in the register (X) has a garbage value and 4th byte in the output buffer (R1) has a correct
1043:         // value which was previously computed by another line. In other words, it means that we can not overwrite
1044:         // it by simply writing 4 bytes from the register to the output. We'll do the following:
1045:         //               v----------|
1046:         // Output = [... X1 X2 X3 | R1 G1 B1 R2 ...]
1047:         // First, we write R1 value to the 4th byte of (R G B | X) -> (R G B | R1)
1048:         // Second, we write 4 bytes from the register to the output: (X1 X2 X3 | R1) -> (R G B | R1)
1049:         // Output = [... R G B | R1 G1 B1 R2 ...]
1050:         _write_endline_rgb_as_uint32(lineOut + out_x_strided, o);
1051:       }
1052:     } else if (num_channels == 3) {
1053:       // Memcpy 4-bytes is faster than 3-bytes and here
1054:       // we simply write 4 bytes (... R G B X 0 0 0 0 0 ...) where X is a garbage value
1055:       // that we will overwrite on the next iteration: (... R G B R G B X 0 0 ...)
1056:       std::memcpy(lineOut + out_x_strided, (uint8_t *) &o, 4);
1057:     } else {
1058:       // num_channels = 4 -> lineOut + out_x_strided should be uint32 aligned
1059:       *(uint32_t *)(lineOut + out_x_strided) = o;
1060:     }
1061:   }
1062: }
1063:
1064: void ImagingResampleVerticalConvolution8u(
1065:     uint8_t* C10_RESTRICT lineOut,
1066:     const uint8_t* C10_RESTRICT lineIn,
1067:     int64_t xsize,
1068:     int64_t ids_min,
1069:     int64_t ids_size,
1070:     const int16_t* k,
1071:     unsigned int coefs_precision,
1072:     int64_t num_channels) {
```
- EN: The main symbol in this range is `ImagingResampleVerticalConvolution8u`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `ImagingResampleVerticalConvolution8u`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 1074-1103
```cpp
1074:   // Interpolation vertical pass processing one line.
1075:   // - We process x-axis data with blocks of 8, 2 and 1
1076:   // - We split the size of weight vector for a given output index as a sum: K = n * 2 + m.
1077:
1078:   // xsize = output width, also equals to input width
1079:   // ids_size = interpolation size
1080:   // ids_min = input y start index
1081:   const auto stride = num_channels * sizeof(uint8_t);
1082:
1083:   TORCH_INTERNAL_ASSERT(stride == 3 || stride == 4);
1084:
1085:   const int64_t data_size = xsize * stride;
1086:   const int64_t data_stride = stride;
1087:   constexpr auto vec_size = 256 / 8;
1088:
1089:   const auto initial = _mm_set1_epi32(1 << (coefs_precision - 1));
1090:   const auto initial_256 = _mm256_set1_epi32(1 << (coefs_precision - 1));
1091:   const auto zero = _mm_setzero_si128();
1092:   const auto zero_256 = _mm256_setzero_si256();
1093:
1094:   int64_t j = 0;
1095:   // block 8
1096:   const auto b8_usable_vec_stride = (vec_size / data_stride) * data_stride;
1097:   for (; j < data_size - vec_size; j += b8_usable_vec_stride) {
1098:     auto sss0 = initial_256;
1099:     auto sss1 = initial_256;
1100:     auto sss2 = initial_256;
1101:     auto sss3 = initial_256;
1102:     int64_t i = 0;
1103:     const auto * lineIn_min = lineIn + j + ids_min;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1105-1139
```cpp
1105:     for (; i < ids_size - 1; i += 2) {
1106:       // Load 2 values from weight vector
1107:       auto mmk = _mm256_set1_epi32(*(int32_t*)&k[i]);
1108:
1109:       // RGBA: Load 8 pixels per line
1110:       // source1 = [
1111:       //    r0 g0 b0 a0  r1 g1 b1 a1  r2 g2 b2 a2  r3 g3 b3 a3
1112:       //    r4 g4 b4 a4  r5 g5 b5 a5  r6 g6 b6 a6  r7 g7 b7 a7
1113:       // ]
1114:       // RGB: Load 10 pixels per line (however we can process only 8 pixels):
1115:       // source1 = [
1116:       //    r0 g0 b0 r1  g1 b1 r2 g2  b2 r3 g3 b3  r4 g4 b4 r5
1117:       //    r4 g4 b4 r5  g5 b5 r6 g6  b6 r7 g7 b7  r8 g8 b8 r9
1118:       // ]
1119:       auto source1 =
1120:           _mm256_loadu_si256((__m256i*)(lineIn_min + data_size * i));
1121:       auto source2 =
1122:           _mm256_loadu_si256((__m256i*)(lineIn_min + data_size * (i + 1)));
1123:
1124:       // Interleave source1 and source2 from the low half of each 128-bit lane
1125:       // and cast the result to epi16
1126:       // RGBA: pix1 = [
1127:       //    r0 0 R0 0  g0 0 G0 0  b0 0 B0 0  a0 0 A0 0
1128:       //    r1 0 R1 0  g1 0 G1 0  b1 0 B1 0  a1 0 A1 0
1129:       // ]
1130:       // RGB: pix1 = [
1131:       //    r0 0 R0 0  g0 0 G0 0  b0 0 B0 0  0 0 0 0
1132:       //    r1 0 R1 0  g1 0 G1 0  b1 0 B1 0  0 0 0 0
1133:       // ]
1134:       auto source_lo = _mm256_unpacklo_epi8(source1, source2);
1135:       auto pix1 = _mm256_unpacklo_epi8(source_lo, zero_256);
1136:       // Compute output value as
1137:       //   C += w0 * c0 + w1 * C0
1138:       //   C += w0 * c1 + w1 * C1 for each channel in 32-bit precision
1139:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix1, mmk));
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1141-1172
```cpp
1141:       // RGBA: pix2 = [
1142:       //    r2 0 R2 0  g2 0 G2 0  b2 0 B2 0  a2 0 A2 0
1143:       //    r3 0 R3 0  g3 0 G3 0  b3 0 B3 0  a3 0 A3 0
1144:       // ]
1145:       // RGB: pix2 = [
1146:       //    r2 0 R2 0  g2 0 G2 0  b2 0 B2 0  0 0 0 0
1147:       //    r3 0 R3 0  g3 0 G3 0  b3 0 B3 0  0 0 0 0
1148:       // ]
1149:       auto pix2 = _mm256_unpackhi_epi8(source_lo, zero_256);
1150:       // Compute output value as
1151:       //   C += w0 * c2 + w1 * C2
1152:       //   C += w0 * c3 + w1 * C3 for each channel in 32-bit precision
1153:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix2, mmk));
1154:
1155:       // Same as above for the high half of each 128-bit lane
1156:       auto source_hi = _mm256_unpackhi_epi8(source1, source2);
1157:       auto pix3 = _mm256_unpacklo_epi8(source_hi, zero_256);
1158:       sss2 = _mm256_add_epi32(sss2, _mm256_madd_epi16(pix3, mmk));
1159:       auto pix4 = _mm256_unpackhi_epi8(source_hi, zero_256);
1160:       sss3 = _mm256_add_epi32(sss3, _mm256_madd_epi16(pix4, mmk));
1161:     }
1162:     // Same processing as above but with a single weight value
1163:     for (; i < ids_size; i += 1) {
1164:       auto mmk = _mm256_set1_epi32(k[i]);
1165:
1166:       auto source1 = _mm256_loadu_si256((__m256i*)(lineIn_min + i * data_size));
1167:
1168:       auto source_lo = _mm256_unpacklo_epi8(source1, zero_256);
1169:       auto pix1 = _mm256_unpacklo_epi8(source_lo, zero_256);
1170:       sss0 = _mm256_add_epi32(sss0, _mm256_madd_epi16(pix1, mmk));
1171:       auto pix2 = _mm256_unpackhi_epi8(source_lo, zero_256);
1172:       sss1 = _mm256_add_epi32(sss1, _mm256_madd_epi16(pix2, mmk));
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1174-1204
```cpp
1174:       auto source_hi = _mm256_unpackhi_epi8(source1, zero_256);
1175:       auto pix3 = _mm256_unpacklo_epi8(source_hi, _mm256_setzero_si256());
1176:       sss2 = _mm256_add_epi32(sss2, _mm256_madd_epi16(pix3, mmk));
1177:       auto pix4 = _mm256_unpackhi_epi8(source_hi, _mm256_setzero_si256());
1178:       sss3 = _mm256_add_epi32(sss3, _mm256_madd_epi16(pix4, mmk));
1179:     }
1180:     // Convert fixed point values back to integers (truncating)
1181:     sss0 = _mm256_srai_epi32(sss0, coefs_precision);
1182:     sss1 = _mm256_srai_epi32(sss1, coefs_precision);
1183:     sss2 = _mm256_srai_epi32(sss2, coefs_precision);
1184:     sss3 = _mm256_srai_epi32(sss3, coefs_precision);
1185:     // Convert packed signed 32-bit integers to packed 16-bit integers using signed saturation
1186:     // (a a a a b b b b c c c c d d d d) -> (a a b b c c d d)
1187:     sss0 = _mm256_packs_epi32(sss0, sss1);
1188:     sss2 = _mm256_packs_epi32(sss2, sss3);
1189:     // Convert packed signed 16-bit integers to packed 8-bit integers using unsigned saturation
1190:     // (a a b b c c d d) -> (a b c d)
1191:     sss0 = _mm256_packus_epi16(sss0, sss2);
1192:
1193:     // Stores 32 bytes
1194:     _mm256_storeu_si256((__m256i*)(lineOut + j), sss0);
1195:   }
1196:
1197:   // TODO: Do we also need block 4 ???
1198:   // block 2
1199:   const auto b2_usable_vec_stride = (8 / data_stride) * data_stride;
1200:   for (; j < data_size - vec_size / 4; j += b2_usable_vec_stride) {
1201:     auto sss0 = initial;
1202:     auto sss1 = initial;
1203:     int64_t i = 0;
1204:     const auto * lineIn_min = lineIn + j + ids_min;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1206-1243
```cpp
1206:     for (; i < ids_size - 1; i += 2) {
1207:       // Load 2 values from weight vector
1208:       // mmk = [wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ... ]
1209:       auto mmk = _mm_set1_epi32(*(int32_t*)&k[i]);
1210:
1211:       // Load 2 pixels per line
1212:       // RGBA: source1 = [
1213:       //    r0 g0 b0 a0  r1 g1 b1 a1  0 0 0 0  0 0 0 0
1214:       // ]
1215:       // RGB: source1 = [
1216:       //    r0 g0 b0 r1  g1 b1 r2 g2  0 0 0 0  0 0 0 0
1217:       // ]
1218:       auto source1 = _mm_loadl_epi64((__m128i *) (lineIn_min + i * data_size));
1219:       auto source2 = _mm_loadl_epi64((__m128i *) (lineIn_min + (i + 1) * data_size));
1220:       // Interleave source1 and source2 and cast the result to epi16
1221:       // RGBA: pix = [
1222:       //    r0 0 R0 0  g0 0 G0 0  b0 0 B0 0  a0 0 A0 0
1223:       // ]
1224:       // RGB: pix = [
1225:       //    r0 0 R0 0  g0 0 G0 0  b0 0 B0 0  0 0 0 0
1226:       // ]
1227:       auto source = _mm_unpacklo_epi8(source1, source2);
1228:       auto pix = _mm_unpacklo_epi8(source, zero);
1229:       // Compute output value as C += w0 * c0 + w1 * C0 for each channel in 32-bit precision
1230:       sss0 = _mm_add_epi32(sss0, _mm_madd_epi16(pix, mmk));
1231:       // RGBA: pix = [
1232:       //    r1 0 R1 0  g1 0 G1 0  b1 0 B1 0  a1 0 A1 0
1233:       // ]
1234:       // RGB: pix = [
1235:       //    r1 0 R1 0  g1 0 G1 0  b1 0 B1 0  0 0 0 0
1236:       // ]
1237:       pix = _mm_unpackhi_epi8(source, zero);
1238:       // Compute output value as C += w0 * c1 + w1 * C1 for each channel in 32-bit precision
1239:       sss1 = _mm_add_epi32(sss1, _mm_madd_epi16(pix, mmk));
1240:     }
1241:     // Same processing as above but with a single weight value
1242:     for (; i < ids_size; i += 1) {
1243:       auto mmk = _mm_set1_epi32(k[i]);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1245-1277
```cpp
1245:       auto source1 = _mm_loadl_epi64((__m128i*) (lineIn_min + i * data_size));
1246:
1247:       auto source = _mm_unpacklo_epi8(source1, zero);
1248:       auto pix1 = _mm_unpacklo_epi8(source, zero);
1249:       sss0 = _mm_add_epi32(sss0, _mm_madd_epi16(pix1, mmk));
1250:       auto pix2 = _mm_unpackhi_epi8(source, zero);
1251:       sss1 = _mm_add_epi32(sss1, _mm_madd_epi16(pix2, mmk));
1252:     }
1253:     // Convert fixed point values back to integers (truncating)
1254:     sss0 = _mm_srai_epi32(sss0, coefs_precision);
1255:     sss1 = _mm_srai_epi32(sss1, coefs_precision);
1256:     // Convert packed signed 32-bit integers to packed 16-bit integers using signed saturation
1257:     // (a a a a b b b b c c c c d d d d) -> (a a b b c c d d)
1258:     sss0 = _mm_packs_epi32(sss0, sss1);
1259:     // Convert packed signed 16-bit integers to packed 8-bit integers using unsigned saturation
1260:     // (a a b b c c d d) -> (a b c d)
1261:     sss0 = _mm_packus_epi16(sss0, sss0);
1262:     // Store 2 pixels to the output
1263:     _mm_storel_epi64((__m128i*)(lineOut + j), sss0);
1264:   }
1265:
1266:   // block 1
1267:   const auto b1_usable_vec_stride = (4 / data_stride) * data_stride;
1268:   const auto i32_aligned = num_channels == 4;
1269:   for (; j < data_size - 4; j += b1_usable_vec_stride) {
1270:     auto sss = initial;
1271:     int64_t i = 0;
1272:     const auto * lineIn_min = lineIn + j + ids_min;
1273:
1274:     for (; i < ids_size - 1; i += 2) {
1275:       // Load 2 values from weight vector
1276:       // mmk = [wl_0 wh_0 wl_1 wh_1  wl_0 wh_0 wl_1 wh_1  ... ]
1277:       auto mmk = _mm_set1_epi32(*(int32_t*)&k[i]);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1279-1309
```cpp
1279:       // Load one pixel per line
1280:       // RGBA: source1 = [
1281:       //    r0 g0 b0 a0  0 0 0 0  0 0 0 0  0 0 0 0
1282:       // ]
1283:       // RGB: source1 = [
1284:       //    r0 g0 b0 r1  0 0 0 0  0 0 0 0  0 0 0 0
1285:       // ]
1286:       auto source1 = mm_cvtsi32_si128(lineIn_min + i * data_size, i32_aligned);
1287:       auto source2 = mm_cvtsi32_si128(lineIn_min + (i + 1) * data_size, i32_aligned);
1288:
1289:       // Interleave source1 and source2 and cast the result to epi16
1290:       // RGBA: pix = [
1291:       //    r0 0 R0 0  g0 0 G0 0  b0 0 B0 0  a0 0 A0 0
1292:       // ]
1293:       // RGB: pix = [
1294:       //    r0 0 R0 0  g0 0 G0 0  b0 0 B0 0  0 0 0 0
1295:       // ]
1296:       auto source = _mm_unpacklo_epi8(source1, source2);
1297:       auto pix = _mm_unpacklo_epi8(source, zero);
1298:       // Compute output value as C += w0 * c0 + w1 * C0 for each channel in 32-bit precision
1299:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
1300:     }
1301:
1302:     for (; i < ids_size; i++) {
1303:       auto mmk = _mm_set1_epi32(k[i]);
1304:       auto pix = mm_cvtepu8_epi32(lineIn_min + i * data_size, i32_aligned);
1305:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
1306:     }
1307:     sss = _mm_srai_epi32(sss, coefs_precision);
1308:     sss = _mm_packs_epi32(sss, zero);
1309:     sss = _mm_packus_epi16(sss, zero);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1311-1340
```cpp
1311:     auto o = _mm_cvtsi128_si32(sss);
1312:
1313:     // Here we write 4 bytes to the output even if num_channels < 4, e.g o = {r,g,b,X} for num_channels=3
1314:     // It is OK to write 4th byte (e.g. X) as on the next step we will overwrite it with new data.
1315:     // We also won't go out of bounds of lineOut memory allocation
1316:     std::memcpy(lineOut + j, (uint8_t *) &o, 4);
1317:   }
1318:
1319:   for (; j < data_size; j += data_stride) {
1320:     auto sss = initial;
1321:     int64_t i = 0;
1322:     const auto * lineIn_min = lineIn + j + ids_min;
1323:     // For RGBA we can use (ids_size - 1) as tighter limit but for RGB we can read outside memory boundary
1324:     // for the last remaining line
1325:     for (; i < ids_size - 2; i += 2) {
1326:       // Load two coefficients at once
1327:       auto mmk = _mm_set1_epi32(*(int32_t*)&k[i]);
1328:
1329:       // Load 2 lines
1330:       auto source1 = mm_cvtsi32_si128(lineIn_min + i * data_size, i32_aligned);
1331:       auto source2 = mm_cvtsi32_si128(lineIn_min + (i + 1) * data_size, i32_aligned);
1332:
1333:       auto source = _mm_unpacklo_epi8(source1, source2);
1334:       auto pix = _mm_unpacklo_epi8(source, zero);
1335:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
1336:     }
1337:
1338:     // Same processing as above but with a single weight value
1339:     for (; i < ids_size; i++) {
1340:       auto mmk = _mm_set1_epi32(k[i]);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1342-1373
```cpp
1342:       const uint8_t * p = lineIn_min + i * data_size;
1343:       __m128i pix;
1344:       // There is no much perf gain using more detailed condition like
1345:       // num_channels == 3 && ids_min + j + data_size * i + 4 >= in_max_size
1346:       // const int64_t in_max_size = data_size * in_ysize;
1347:       if (num_channels == 3) {
1348:         uint8_t input[4];
1349:         std::memcpy(input, p, 3);
1350:         pix = mm_cvtepu8_epi32(input, true);
1351:       } else {
1352:         pix = mm_cvtepu8_epi32(p, true);
1353:       }
1354:       sss = _mm_add_epi32(sss, _mm_madd_epi16(pix, mmk));
1355:     }
1356:
1357:     // Convert fixed point values back to integers (truncating)
1358:     sss = _mm_srai_epi32(sss, coefs_precision);
1359:     // Convert packed signed 32-bit integers to packed 16-bit integers using signed saturation
1360:     // (a a a a b b b b c c c c d d d d) -> (a a b b c c d d)
1361:     sss = _mm_packs_epi32(sss, zero);
1362:     // Convert packed signed 16-bit integers to packed 8-bit integers using unsigned saturation
1363:     // (a a b b c c d d) -> (a b c d)
1364:     sss = _mm_packus_epi16(sss, zero);
1365:     // Store one pixel to the output
1366:     auto o = _mm_cvtsi128_si32(sss);
1367:     if (num_channels == 3 && C10_UNLIKELY(j + 4 >= data_size)) {
1368:       std::memcpy(lineOut + j, (uint8_t *) &o, 3);
1369:     } else {
1370:       std::memcpy(lineOut + j, (uint8_t *) &o, 4);
1371:     }
1372:   }
1373: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1375-1376
```cpp
1375: } // anonymous namespace
1376: #endif // CPU_CAPABILITY_AVX2
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Low-level memory access / 底层内存访问
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/cpu/vec/intrinsics.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
