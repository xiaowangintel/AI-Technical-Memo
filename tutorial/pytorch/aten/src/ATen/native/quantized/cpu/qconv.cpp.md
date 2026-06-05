# qconv.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qconv.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU convolution helpers, packed-parameter handling, or output-shape logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 卷积辅助逻辑、打包参数处理或输出形状计算。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <algorithm>
 3: #include <cmath>
 4: #include <string>
 5: #include <vector>
 6:
 7: #include <ATen/core/Tensor.h>
 8: #include <ATen/core/List.h>
 9: #include <ATen/Context.h>
10: #include <ATen/Parallel.h>
11: #include <ATen/TensorOperators.h>
12: #include <ATen/SmallVector.h>
13: #include <ATen/native/quantized/PackedParams.h>
14: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
15: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
16: #include <ATen/native/quantized/cpu/XnnpackUtils.h>
17: #include <ATen/native/quantized/cpu/OnednnUtils.h>
18: #include <ATen/native/quantized/ConvUtils.h>
19: #include <ATen/native/quantized/cpu/QuantUtils.h>
20: #include <ATen/native/quantized/cpu/qconv.h>
21: #include <caffe2/utils/threadpool/pthreadpool-cpp.h>
22: #include <torch/library.h>
23: #include <ATen/quantized/Quantizer.h>
24: #include <ATen/native/mkldnn/MKLDNNCommon.h>
25:
26: #ifndef AT_PER_OPERATOR_HEADERS
27: #include <ATen/Functions.h>
28: #include <ATen/NativeFunctions.h>
29: #else
30: #include <ATen/ops/_empty_affine_quantized.h>
31: #include <ATen/ops/_empty_affine_quantized_native.h>
32: #include <ATen/ops/_empty_per_channel_affine_quantized_native.h>
33: #include <ATen/ops/empty.h>
34: #include <ATen/ops/quantize_per_channel_native.h>
35: #include <ATen/ops/quantize_per_tensor_native.h>
36: #include <ATen/ops/zeros.h>
37: #include <ATen/ops/convolution.h>
38: #include <ATen/ops/linear.h>
39: #include <ATen/ops/relu.h>
40: #include <ATen/ops/leaky_relu.h>
41: #include <ATen/ops/tanh.h>
42: #include <ATen/ops/gelu.h>
43: #include <ATen/ops/hardtanh.h>
44: #include <ATen/ops/hardswish.h>
45: #include <ATen/ops/sigmoid.h>
46: #endif
47:
48: #include <c10/util/irange.h>
```
- EN: This range pulls in required headers, including `algorithm`, `cmath`, `string`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `algorithm`, `cmath`, `string`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 50-105
```cpp
 50: namespace {
 51: // To have a sanity check for maximum matrix size.
 52: constexpr int64_t kReasonableMaxDim = 1000000;
 53: } // namespace
 54:
 55: template <int kSpatialDim = 2>
 56: static bool ConvDimChecks(
 57:     int64_t act_dims,
 58:     int64_t stride_dims,
 59:     int64_t padding_dims,
 60:     int64_t output_padding_dims,
 61:     int64_t dilation_dims,
 62:     std::string func_name,
 63:     bool transpose = false) {
 64:   TORCH_CHECK(
 65:       act_dims == kSpatialDim + 2,
 66:       func_name,
 67:       kSpatialDim,
 68:       "d(): Expected activation tensor to have ",
 69:       kSpatialDim + 2,
 70:       " dimensions, got ",
 71:       act_dims);
 72:   TORCH_CHECK(
 73:       stride_dims == kSpatialDim,
 74:       func_name,
 75:       kSpatialDim,
 76:       "d(): Expected stride tensor to have ",
 77:       kSpatialDim,
 78:       " dimensions, got ",
 79:       stride_dims);
 80:   TORCH_CHECK(
 81:       padding_dims == kSpatialDim,
 82:       func_name,
 83:       kSpatialDim,
 84:       "d(): Expected padding tensor to have ",
 85:       kSpatialDim,
 86:       " dimensions, got ",
 87:       padding_dims);
 88:   TORCH_CHECK(
 89:       !transpose || (output_padding_dims == kSpatialDim),
 90:       func_name,
 91:       kSpatialDim,
 92:       "d(): Expected output padding tensor to have ",
 93:       kSpatialDim,
 94:       " dimensions, got ",
 95:       output_padding_dims);
 96:   TORCH_CHECK(
 97:       dilation_dims == kSpatialDim,
 98:       func_name,
 99:       kSpatialDim,
100:       "d(): Expected dilation tensor to have ",
101:       kSpatialDim,
102:       " dimensions, got ",
103:       dilation_dims);
104:   return true;
105: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `ConvDimChecks`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `ConvDimChecks`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 107-155
```cpp
107: static inline int64_t compute_deconv_shape(int64_t input,
108:                                     int64_t kernel,
109:                                     int64_t stride,
110:                                     int64_t input_padding,
111:                                     int64_t output_padding,
112:                                     int64_t dilation) {
113:   int64_t out = (input - 1) * stride - 2 * input_padding
114:                 + dilation * (kernel - 1) + output_padding + 1;
115:   return out;
116: }
117:
118: template <int64_t kSpatialDim>
119: static at::SmallVector<int64_t, kSpatialDim + 2> MakeDeConvOutputShape(
120:     int64_t N, int64_t M,
121:     const std::vector<int64_t>& input_shape,
122:     const std::vector<int64_t>& kernel,
123:     const torch::List<int64_t>& stride,
124:     const torch::List<int64_t>& input_padding,
125:     const torch::List<int64_t>& output_padding,
126:     const torch::List<int64_t>& dilation) {
127:   at::SmallVector<int64_t, kSpatialDim + 2> output_shape;
128:   output_shape.resize(kSpatialDim + 2);
129:   output_shape[0] = N;  // Batch size
130:   output_shape[1] = M;  // Output channels
131:   for (const auto idx : c10::irange(kSpatialDim)) {
132:     output_shape[idx + 2] = compute_deconv_shape(input_shape[idx],
133:                                                  kernel[idx],
134:                                                  stride[idx],
135:                                                  input_padding[idx],
136:                                                  output_padding[idx],
137:                                                  dilation[idx]);
138:     TORCH_CHECK(output_shape[idx + 2] > 0,
139:                 "Output dimension is zero for ", idx, " axis;"
140:                 " kernel: ", kernel[idx],
141:                 ", stride: ", stride[idx],
142:                 ", input padding: ", input_padding[idx],
143:                 ", output padding: ", output_padding[idx],
144:                 ", dilation: ", dilation[idx])
145:     TORCH_CHECK(output_shape[idx + 2] < kReasonableMaxDim,
146:                 "Output dimension is beyond reasonable maximum for ", idx,
147:                 " axis;"
148:                 " kernel: ", kernel[idx],
149:                 ", stride: ", stride[idx],
150:                 ", input padding: ", input_padding[idx],
151:                 ", output padding: ", output_padding[idx],
152:                 ", dilation: ", dilation[idx]);
153:   }
154:   return output_shape;
155: }
```
- EN: The main symbol in this range is `compute_deconv_shape`, `MakeDeConvOutputShape`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `compute_deconv_shape`, `MakeDeConvOutputShape`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 157-204
```cpp
157: #ifdef USE_FBGEMM
158:
159: template <int kSpatialDim = 2>
160: at::SmallVector<int64_t, kSpatialDim + 2> MakeConvOutputShape(
161:     int N,
162:     int M,
163:     const std::array<int, kSpatialDim>& output_image_shape);
164:
165: template <>
166: at::SmallVector<int64_t, 4> MakeConvOutputShape<2>(
167:     int N,
168:     int M,
169:     const std::array<int, 2>& output_image_shape) {
170:   return {N, M, output_image_shape[0], output_image_shape[1]};
171: }
172:
173: template <>
174: at::SmallVector<int64_t, 5> MakeConvOutputShape<3>(
175:     int N,
176:     int M,
177:     const std::array<int, 3>& output_image_shape) {
178:   return {N,
179:           M,
180:           output_image_shape[0],
181:           output_image_shape[1],
182:           output_image_shape[2]};
183: }
184:
185: #endif // USE_FBGEMM
186:
187: #ifdef USE_PYTORCH_QNNPACK
188:
189: template <size_t kSpatialDim>
190: static std::array<int64_t, kSpatialDim> MakeInputShape(
191:     int64_t D,
192:     int64_t H,
193:     int64_t W);
194:
195: template <>
196: std::array<int64_t, 2> MakeInputShape(int64_t /*D*/, int64_t H, int64_t W) {
197:   return {H, W};
198: }
199: template <>
200: std::array<int64_t, 3> MakeInputShape(int64_t D, int64_t H, int64_t W) {
201:   return {D, H, W};
202: }
203:
204: #endif // USE_PYTORCH_QNNPACK
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `MakeConvOutputShape<2>`, `MakeConvOutputShape<3>`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `MakeConvOutputShape<2>`, `MakeConvOutputShape<3>`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 206-252
```cpp
206: #ifdef USE_FBGEMM
207: template <int kSpatialDim>
208: const float* PackedConvWeight<kSpatialDim>::GetBiasData(at::Tensor* bias_ptr) {
209:   const float* bias_data = nullptr;
210:   if (bias.has_value()) {
211:     *bias_ptr = bias.value();
212:     TORCH_CHECK(
213:         bias_ptr->dtype() == at::kFloat,
214:         "[QConv3D] The 'bias' tensor must have 'torch.float' dtype");
215:     *bias_ptr = bias_ptr->contiguous();
216:     TORCH_CHECK(bias_ptr->dim() == 1, "bias should be a vector (1D Tensor)");
217:     const int M = w->outputChannels();
218:     TORCH_CHECK(bias_ptr->size(0) == M, "bias should have ", M, " elements.");
219:     bias_data = bias_ptr->data_ptr<float>();
220:   }
221:   return bias_data;
222: }
223:
224: template <int kSpatialDim>
225: void PackedConvWeight<kSpatialDim>::GetQuantizationParams(
226:     float act_scale,
227:     float out_scale,
228:     std::vector<float>* output_multiplier_float,
229:     std::vector<float>* act_times_w_scale) {
230:   if (q_scheme == c10::kPerTensorAffine) {
231:     *act_times_w_scale = {(act_scale * w_scale[0])};
232:     *output_multiplier_float = {act_times_w_scale->front() / out_scale};
233:   } else if (q_scheme == c10::kPerChannelAffine) {
234:     const int M = w->outputChannels();
235:     output_multiplier_float->resize(M);
236:     act_times_w_scale->resize(M);
237:     for (const auto i : c10::irange(M)) {
238:       act_times_w_scale->at(i) = (act_scale * w_scale[i]);
239:       output_multiplier_float->at(i) = act_times_w_scale->at(i) / out_scale;
240:     }
241:   } else {
242:     TORCH_CHECK(false, "[QConv", kSpatialDim, "D] Unknown quantization scheme");
243:   }
244: }
245:
246: template <int kSpatialDim>
247: at::Tensor PackedConvWeight<kSpatialDim>::apply(
248:     const at::Tensor& input,
249:     double output_scale,
250:     int64_t output_zero_point) {
251:   return apply_impl<false>(input, output_scale, output_zero_point);
252: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `GetBiasData`, `GetQuantizationParams`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `GetBiasData`, `GetQuantizationParams`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 254-303
```cpp
254: template <int kSpatialDim>
255: at::Tensor PackedConvWeight<kSpatialDim>::apply_relu(
256:     const at::Tensor& input,
257:     double output_scale,
258:     int64_t output_zero_point) {
259:   return apply_impl<true>(input, output_scale, output_zero_point);
260: }
261:
262: template <int kSpatialDim>
263: template <bool kReluFused>
264: at::Tensor PackedConvWeight<kSpatialDim>::apply_impl(
265:     const at::Tensor& act,
266:     double output_scale,
267:     int64_t output_zero_point) {
268:   // Quantized kernels are all written with NHWC (channels last) layout in
269:   // mind. Ideally, we'd be compatible with conv2d behavior and preserve the
270:   // inputs layout as is (doing necessary upconversions).
271:   //
272:   // However, to be more robust, for now we just force output layout to always
273:   // be NHWC (channels last), thus opportunistically improving perf.
274:   //
275:   // This might change when full memory format support lands
276:   // See https://github.com/pytorch/pytorch/issues/23403
277:   const std::string func_name = transpose() ? "quantized::conv_transpose"
278:                                             : "quantized::conv";
279:   TORCH_CHECK(
280:       fbgemm::fbgemmSupportedCPU(), "Your CPU does not support FBGEMM.");
281:   TORCH_CHECK(act.scalar_type() == c10::kQUInt8,
282:                 func_name,
283:                 "(FBGEMM): Expected activation data type ",
284:                 toString(c10::kQUInt8),
285:                 " but got ",
286:                 toString(act.scalar_type()));
287:
288:   ConvDimChecks<kSpatialDim>(
289:       act.ndimension(), stride().size(), padding().size(),
290:       output_padding().size(), dilation().size(), func_name, transpose());
291:
292:   const int N = act.size(0);
293:   const int C = act.size(1);
294:   const int D = kSpatialDim == 2 ? 1 : act.size(2);
295:   const int H = act.size(kSpatialDim);
296:   const int W = act.size(kSpatialDim + 1);
297:
298:   const at::Tensor act_ndhwc = kSpatialDim == 2
299:       ? act.contiguous(c10::MemoryFormat::ChannelsLast)
300:       : at::native::fbgemm_utils::ConvertToChannelsLast3dTensor(act);
301:   const uint8_t* act_data =
302:       reinterpret_cast<uint8_t*>(act_ndhwc.data_ptr<c10::quint8>());
303:   auto* pack_w = w.get();
```
- EN: The main symbol in this range is `apply_relu`, `apply_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_relu`, `apply_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 305-351
```cpp
305:   const int M = pack_w->outputChannels();
306:   const int kernel_d = kSpatialDim == 2 ? 1 : kernel[0];
307:   const int kernel_h = kernel[kSpatialDim - 2];
308:   const int kernel_w = kernel[kSpatialDim - 1];
309:   const int pad_d = kSpatialDim == 2 ? 0 : padding_[0];
310:   const int pad_h = padding_[kSpatialDim - 2];
311:   const int pad_w = padding_[kSpatialDim - 1];
312:   const int stride_d = kSpatialDim == 2 ? 1 : stride_[0];
313:   const int stride_h = stride_[kSpatialDim - 2];
314:   const int stride_w = stride_[kSpatialDim - 1];
315:   const int dilation_d = kSpatialDim == 2 ? 1 : dilation_[0];
316:   const int dilation_h = dilation_[kSpatialDim - 2];
317:   const int dilation_w = dilation_[kSpatialDim - 1];
318:   const int output_padding_d = kSpatialDim == 2 ? 0 : output_padding_[0];
319:   const int output_padding_h = output_padding_[kSpatialDim - 2];
320:   const int output_padding_w = output_padding_[kSpatialDim - 1];
321:
322:   if (kSpatialDim == 2) {
323:     TORCH_CHECK(
324:         C == pack_w->inputChannels(),
325:         "[QConv2D] Given groups=",
326:         groups_,
327:         ", weight of size ",
328:         M,
329:         ", ",
330:         kernel_h,
331:         ", ",
332:         kernel_w,
333:         ", ",
334:         pack_w->inputChannels(),
335:         ", expected input (NCHW) ",
336:         N,
337:         ", ",
338:         C,
339:         ", ",
340:         H,
341:         ", ",
342:         W,
343:         " to have ",
344:         pack_w->inputChannels(),
345:         " channels, but got ",
346:         C,
347:         " channels instead");
348:   } else {
349:     TORCH_CHECK(
350:         C == pack_w->inputChannels(),
351:         "[QConv3D] Given groups=",
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 352-401
```cpp
352:         groups_,
353:         ", weight of size ",
354:         M,
355:         ", ",
356:         kernel_d,
357:         ", ",
358:         kernel_h,
359:         ", ",
360:         kernel_w,
361:         ", ",
362:         pack_w->inputChannels(),
363:         ", expected input (NCDHW) ",
364:         N,
365:         ", ",
366:         C,
367:         ", ",
368:         D,
369:         ", ",
370:         H,
371:         ", ",
372:         W,
373:         " to have ",
374:         pack_w->inputChannels(),
375:         " channels, but got ",
376:         C,
377:         " channels instead");
378:   }
379:
380:   fbgemm::conv_param_t<kSpatialDim> conv_p =
381:       at::native::fbgemm_utils::MakeFbgemmConvParam<kSpatialDim>(
382:           N, // Batch size
383:           C, // Number of input channels
384:           M, // Number of output channels
385:           kSpatialDim == 2 ? std::vector<int>{H, W} : std::vector<int>{D, H, W},
386:           groups_,
387:           kSpatialDim == 2 ? std::vector<int>{kernel_h, kernel_w}
388:                            : std::vector<int>{kernel_d, kernel_h, kernel_w},
389:           kSpatialDim == 2 ? std::vector<int>{stride_h, stride_w}
390:                            : std::vector<int>{stride_d, stride_h, stride_w},
391:           kSpatialDim == 2 ? std::vector<int>{pad_h, pad_w}
392:                            : std::vector<int>{pad_d, pad_h, pad_w},
393:           kSpatialDim == 2
394:               ? std::vector<int>{dilation_h, dilation_w}
395:               : std::vector<int>{dilation_d, dilation_h, dilation_w},
396:           kSpatialDim == 2
397:               ? std::vector<int>{output_padding_h, output_padding_w}
398:               : std::vector<int>{output_padding_d,
399:                                  output_padding_h,
400:                                  output_padding_w},
401:           transpose());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 403-449
```cpp
403:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
404:   const float act_scale = act.q_scale();
405:   const int32_t act_zero_point = act.q_zero_point();
406:
407:   at::Tensor bias;
408:   const float* bias_data = GetBiasData(&bias);
409:
410:   TORCH_CHECK(
411:       w_scale.size() == w_zp.size(),
412:       "Weight scales and zero points vectors should have the same size.");
413:   std::vector<float> output_multiplier_float;
414:   std::vector<float> act_times_w_scale;
415:   GetQuantizationParams(
416:       act_scale, output_scale, &output_multiplier_float, &act_times_w_scale);
417:
418:   at::SmallVector<int64_t, kSpatialDim + 2> output_shape;
419:   if (transpose()) {
420:     output_shape = MakeDeConvOutputShape<kSpatialDim>(
421:         N,
422:         M,
423:         kSpatialDim == 2 ? std::vector<int64_t>{H, W} : std::vector<int64_t>{D, H, W},
424:         kernel,
425:         stride(),
426:         padding(),
427:         output_padding(),
428:         dilation());
429:
430:     // if use direct convolution implementation, compute the col_offsets
431:     // of the weight matrix at model initialization stage.
432:     // We need to know the shape of output matrix
433:     // to compute col_offsets for direct convolution.
434:     // Hence it cannot be called from inside weight packing function
435:     // like other quantized conv implementation
436:     if (pack_w->getPackedWForDirectconv().get() &&
437:         pack_w->getPackedWForDirectconv().get()->is_first_call()) {
438:           pack_w->getPackedWForDirectconv().get()->col_offsets_with_zero_pt_s8acc32_DirectConvT(
439:               conv_p,
440:               w_zp.data(),
441:               col_offsets,
442:               M);
443:     }
444:   } else {
445:     output_shape = MakeConvOutputShape<kSpatialDim>(N, M, conv_p.OUT_DIM);
446:   }
447:   if (N > 0) {
448:     TORCH_CHECK(
449:         std::all_of(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 450-497
```cpp
450:             output_shape.begin(),
451:             output_shape.end(),
452:             [](int64_t i) { return i > 0; }),
453:         "[QConv",
454:         kSpatialDim,
455:         "D] each dimension of output tensor should be greater than 0");
456:   }
457:   at::Tensor output = kSpatialDim == 2
458:       ? at::_empty_affine_quantized(
459:             output_shape,
460:             at::device(c10::kCPU)
461:                 .dtype(c10::kQUInt8)
462:                 .memory_format(c10::MemoryFormat::ChannelsLast),
463:             output_scale,
464:             output_zero_point,
465:             std::nullopt)
466:       : at::native::fbgemm_utils::MakeEmptyAffineQuantizedChannelsLast3dTensor(
467:             output_shape[0],
468:             output_shape[1],
469:             output_shape[2],
470:             output_shape[3],
471:             output_shape[4],
472:             at::device(c10::kCPU).dtype(c10::kQUInt8),
473:             output_scale,
474:             output_zero_point);
475:   at::Tensor buffer =
476:       at::empty(output.sizes(), output.options().dtype(c10::kInt));
477:   const int num_tasks = at::get_num_threads();
478:   at::parallel_for(0, num_tasks, 1, [&](int64_t begin, int64_t end) {
479:     fbgemm::DoNothing<> kNoOpObj{};
480:     for (const auto task_id : c10::irange(begin, end)) {
481:       if (q_scheme == c10::kPerTensorAffine) {
482:         fbgemm::ReQuantizeOutput<
483:             kReluFused,
484:             fbgemm::QuantizationGranularity::TENSOR,
485:             float>
486:             output_proc_obj(
487:                 kNoOpObj,
488:                 output_multiplier_float.data(),
489:                 output_zero_point,
490:                 act_zero_point,
491:                 w_zp.data(),
492:                 nullptr, /* row offset buffer */
493:                 col_offsets.data(),
494:                 bias_data,
495:                 M,
496:                 groups_,
497:                 act_times_w_scale.data());
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 498-549
```cpp
498:         fbgemm::fbgemmConv<decltype(output_proc_obj), kSpatialDim, int32_t>(
499:             conv_p,
500:             act_data,
501:             *pack_w,
502:             reinterpret_cast<uint8_t*>(output.data_ptr<c10::quint8>()),
503:             buffer.data_ptr<int32_t>(),
504:             output_proc_obj,
505:             task_id /* thread_id*/,
506:             num_tasks /* num_threads */);
507:       } else if (q_scheme == c10::kPerChannelAffine) {
508:         fbgemm::ReQuantizeOutput<
509:             kReluFused,
510:             fbgemm::QuantizationGranularity::OUT_CHANNEL,
511:             float>
512:             output_proc_obj(
513:                 kNoOpObj,
514:                 output_multiplier_float.data(),
515:                 output_zero_point,
516:                 act_zero_point,
517:                 w_zp.data(),
518:                 nullptr, /* row offset buffer */
519:                 col_offsets.data(),
520:                 bias_data,
521:                 M,
522:                 groups_,
523:                 act_times_w_scale.data());
524:
525:         fbgemm::fbgemmConv<decltype(output_proc_obj), kSpatialDim, int32_t>(
526:             conv_p,
527:             act_data,
528:             *pack_w,
529:             reinterpret_cast<uint8_t*>(output.data_ptr<c10::quint8>()),
530:             buffer.data_ptr<int32_t>(),
531:             output_proc_obj,
532:             task_id /* thread_id*/,
533:             num_tasks /* num_threads */);
534:       }
535:     }
536:   });
537:
538:   return output;
539: }
540:
541: template at::Tensor PackedConvWeight<2>::apply(
542:     const at::Tensor& act,
543:     double output_scale,
544:     int64_t output_zero_point);
545:
546: template at::Tensor PackedConvWeight<2>::apply_relu(
547:     const at::Tensor& act,
548:     double output_scale,
549:     int64_t output_zero_point);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 551-620
```cpp
551: template at::Tensor PackedConvWeight<3>::apply(
552:     const at::Tensor& act,
553:     double output_scale,
554:     int64_t output_zero_point);
555:
556: template at::Tensor PackedConvWeight<3>::apply_relu(
557:     const at::Tensor& act,
558:     double output_scale,
559:     int64_t output_zero_point);
560:
561: template at::Tensor PackedConvWeight<2>::apply_impl<false>(
562:     const at::Tensor& act,
563:     double output_scale,
564:     int64_t output_zero_point);
565:
566: template at::Tensor PackedConvWeight<3>::apply_impl<false>(
567:   const at::Tensor& act,
568:   double output_scale,
569:   int64_t output_zero_point);
570:
571: #endif // USE_FBGEMM
572:
573: #ifdef USE_PYTORCH_QNNPACK
574:
575: #ifdef USE_XNNPACK
576: template <int kSpatialDim>
577: template <typename scalar_t, bool kReluFused>
578: at::Tensor PackedConvWeightsQnnp<kSpatialDim>::apply_impl_xnnp(
579:     const at::Tensor& act, double output_scale, int64_t output_zero_point) {
580:   using underlying_t = typename scalar_t::underlying;
581:
582:   std::lock_guard<std::mutex> lock(qnnp_mutex_);
583:
584:   const std::string func_name = transpose()
585:       ? "quantized::conv_transpose (xnnpack)"
586:       : "quantized::conv (xnnpack)";
587:   TORCH_CHECK(
588:       kSpatialDim == 2,
589:       func_name, ": xnnpack does not currently support 3d convolution.");
590:
591:   /*
592:    * NB:
593:    * [de]conv_prepack prepares weights (values, scale, and zero_points) ahead of
594:    * time during prepack() call assuming the activation will be uint8_t. But it
595:    * may not always be the case. A solution may involve making prepack routine
596:    * aware of the input qdtype. But currently all the pieces are not ready to
597:    * pass that model level info to the prepack function. So, for now, here in
598:    * this function we have to massage weights if we learn the input qdtype is
599:    * not uint8_t. This involves copying and converting uint8_t to int8_t
600:    * whenever necessary. To add to that, since XNNPACK, as of writing this,
601:    * doesn't support per_channel weights for quint8_t, we add following assert
602:    * makes sure we don't run into that case. Also take shortcuts when processing
603:    * weights, which means we have to revisit and fix some weight massging logic
604:    * when we enable the missing feature in XNNPACK.
605:    *
606:    * Table below summarizes how the weights are handled,
607:    *
608:    * .-------------------------------------------------------------------------.
609:    * | input_qdtype |              uint8_t            |            int8_t      |
610:    * | per_channel  |       yes       |       no      |      yes     |    no   |
611:    * |-------------------------------------------------------------------------|
612:    * | zero_points  | at::zeros()*    | orig_zp + 128 | at:zeros()** | orig_zp |
613:    * | scale        |            dtype = float, no changes needed              |
614:    * | values       |        always processed before passing to XNNPACK        |
615:    * .-------------------------------------------------------------------------.
616:    *
617:    * Notes: * - zero_points for uint8_t + per_channel: no support in xnnpack, need
618:    * to fix when support is added. ** - zero_points for int8_t: symmetric
619:    * quantization means XNNPACK will ignore kernel zero point(s).
620:    */
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_impl_xnnp`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_impl_xnnp`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 622-689
```cpp
622:   if constexpr (std::is_same_v<underlying_t, c10::quint8>) {
623:     TORCH_CHECK(!per_channel(),
624:       func_name, ": xnnpack does not currently have per_channel support with activation dtype of c10::quint8."
625:     );
626:   }
627:
628:   // More checks
629:   ConvDimChecks<kSpatialDim>(
630:       act.ndimension(),
631:       stride().size(),
632:       padding().size(),
633:       output_padding().size(),
634:       dilation().size(),
635:       func_name,
636:       transpose());
637:
638:   const int64_t N = act.size(0);
639:   const int64_t H = act.size(2);
640:   const int64_t W = act.size(3);
641:   const int64_t D = 1;
642:   const int64_t M = bias.size(0);
643:
644:   const auto act_nhwc = act.contiguous(c10::MemoryFormat::ChannelsLast);
645:   const auto act_input_scale = act_nhwc.q_scale();
646:
647:   auto status = xnn_status_invalid_state;
648:
649:   // Create an operator iff necessary
650:   if (!xnnp_convolution_op ||
651:       (!input_scale.has_value() || input_scale.value() != act_input_scale)) {
652:     xnn_operator_t xnnp_op = nullptr;
653:
654:     // Update the input scale so we may cache the op
655:     input_scale = act_input_scale;
656:
657:     // create an empty tensor for packing the weights
658:     const at::Tensor weight_contig =
659:         orig_weight.contiguous(c10::MemoryFormat::ChannelsLast);
660:     const float* w_scales_data = w_scales.const_data_ptr<float>();
661:     underlying_t w_zp = 0;
662:     at::Tensor weight_tensor;
663:
664:     if (!per_channel()) {
665:       w_zp = static_cast<underlying_t>(
666:           weight_contig.q_zero_point() +
667:           (std::is_same_v<underlying_t, uint8_t> ? 128 : 0));
668:
669:       weight_tensor = at::native::empty_affine_quantized(
670:           weight_contig.sizes(),
671:           c10::CppTypeToScalarType<scalar_t>::value,
672:           std::nullopt /* layout */,
673:           c10::kCPU,
674:           std::nullopt /* pin_memory */,
675:           w_scales_data[0],
676:           w_zp,
677:           c10::MemoryFormat::ChannelsLast);
678:     } else { /* per_channel */
679:       weight_tensor = at::native::empty_per_channel_affine_quantized(
680:           weight_contig.sizes(),
681:           w_scales,
682:           at::zeros(w_scales.sizes(), at::kInt), /* see comment above about w_zp */
683:           weight_contig.q_per_channel_axis(),
684:           c10::CppTypeToScalarType<scalar_t>::value,
685:           std::nullopt /* layout */,
686:           c10::kCPU,
687:           std::nullopt /* pin_memory */,
688:           c10::MemoryFormat::ChannelsLast);
689:     }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 691-742
```cpp
691:     // copy from the original weight and take care of dtype change if necessary
692:     at::native::xnnp_utils::q8_copy_int8_weight_and_add_offset<scalar_t>(
693:         weight_contig, weight_tensor);
694:     const at::Tensor xnnp_weight =
695:         at::native::xnnp_utils::convert_conv_weights_to_channel_last_tensor<
696:             kSpatialDim>(weight_tensor, groups(), transpose());
697:
698:     auto output_min = kReluFused
699:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
700:         ? activationLimits<underlying_t>(output_scale, output_zero_point, Activation::RELU).first
701:         : std::numeric_limits<underlying_t>::min();
702:     auto output_max = kReluFused
703:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
704:         ? activationLimits<underlying_t>(output_scale, output_zero_point, Activation::RELU).second
705:         : std::numeric_limits<underlying_t>::max();
706:
707:
708:     // Original bias was float, so we requantize it here.
709:     at::Tensor qbias = quant_utils::QuantizeBias(per_channel(), bias, weight_contig, act_input_scale);
710:
711:     status = at::native::xnnp_utils::xnnp_create_convolution2d_nhwc(
712:         padding()[0],
713:         padding()[1],
714:         padding()[0],
715:         padding()[1],
716:         kernel_[0],
717:         kernel_[1],
718:         stride()[0],
719:         stride()[1],
720:         dilation()[0],
721:         dilation()[1],
722:         groups(),
723:         !transpose() ? orig_weight.size(1) : orig_weight.size(0) / groups(),
724:         !transpose() ? orig_weight.size(0) / groups() : orig_weight.size(1),
725:         !transpose() ? orig_weight.size(1) * groups() : orig_weight.size(0),
726:         !transpose() ? orig_weight.size(0) : orig_weight.size(1) * groups(),
727:         act_nhwc.q_zero_point(),
728:         act_input_scale,
729:         w_zp, /* will be ignored for Q[SC]8, see comment
730:                 above about w_zp*/
731:         w_scales_data,
732:         reinterpret_cast<const underlying_t*>(
733:             xnnp_weight.template data_ptr<scalar_t>()),
734:         reinterpret_cast<int32_t*>(qbias.template data_ptr<c10::qint32>()),
735:         output_zero_point,
736:         output_scale,
737:         output_min,
738:         output_max,
739:         0,
740:         &xnnp_op,
741:         per_channel(),
742:         transpose());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 744-793
```cpp
744:     xnnp_convolution_op = xnnpack_operator(xnnp_op);
745:     TORCH_CHECK(
746:         status == xnn_status_success,
747:         func_name,
748:         ": xnn create operator failed(",
749:         status,
750:         ")");
751:   }
752:
753:   at::SmallVector<int64_t, kSpatialDim + 2> output_shape;
754:   const auto input_shape = MakeInputShape<kSpatialDim>(D, H, W);
755:   if (transpose()) {
756:     output_shape = MakeDeConvOutputShape<kSpatialDim>(
757:         N, M, {H, W}, kernel_, stride(), padding(), output_padding(), dilation());
758:   } else {
759:     output_shape = at::native::quantized::MakeConvOutputShape<kSpatialDim>(
760:         N, M, input_shape, kernel_, stride(), padding(), dilation());
761:   }
762:
763:   if (act_nhwc.numel() > 0) {
764:     TORCH_CHECK(
765:         std::all_of(
766:             output_shape.begin(),
767:             output_shape.end(),
768:             [](int64_t i) { return i > 0; }),
769:         func_name, ": ", kSpatialDim, "d (xnnpack): each dimension of output tensor should be greater than 0.")
770:   }
771:
772:   // Allocate output Tensor and a buffer for XNNPACK to use
773:   at::Tensor output = at::native::empty_affine_quantized(
774:       output_shape,
775:       c10::CppTypeToScalarType<scalar_t>::value,
776:       std::nullopt /* layout */,
777:       c10::kCPU,
778:       std::nullopt /* pin_memory */,
779:       output_scale,
780:       output_zero_point,
781:       c10::MemoryFormat::ChannelsLast);
782:
783:   // Reshape the operator
784:   status = at::native::xnnp_utils::xnnp_reshape_convolution2d_nhwc(
785:       xnnp_convolution_op.get(),
786:       N,
787:       H,
788:       W,
789:       caffe2::pthreadpool_(),
790:       per_channel(),
791:       transpose(),
792:       output_padding()[0],
793:       output_padding()[1]);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 795-856
```cpp
795:   TORCH_CHECK(
796:       status == xnn_status_success,
797:       func_name,
798:       ": xnn setup operator failed(",
799:       status,
800:       ")");
801:
802:   // Setup the operator
803:   status = at::native::xnnp_utils::xnnp_setup_convolution2d_nhwc(
804:       xnnp_convolution_op.get(),
805:       reinterpret_cast<const underlying_t*>(act_nhwc.template data_ptr<scalar_t>()),
806:       reinterpret_cast<underlying_t*>(output.template data_ptr<scalar_t>()),
807:       per_channel(),
808:       transpose());
809:
810:   TORCH_CHECK(
811:       status == xnn_status_success,
812:       func_name,
813:       ": xnn setup operator failed(",
814:       status,
815:       ")");
816:
817:   // Run the operator
818:   status = xnn_run_operator(
819:       xnnp_convolution_op.get(), /* xnn_operator_t op */
820:       caffe2::pthreadpool_()); /* pthreadpool_t threadpool */
821:
822:   TORCH_CHECK(
823:       status == xnn_status_success,
824:       func_name,
825:       ": xnn run operator failed(",
826:       status,
827:       ")");
828:
829:   return output;
830: }
831:
832: #endif // USE_XNNPACK
833:
834: template <int kSpatialDim>
835: template <bool kReluFused>
836: at::Tensor PackedConvWeightsQnnp<kSpatialDim>::apply_impl(
837:     const at::Tensor& act,
838:     double output_scale,
839:     int64_t output_zero_point) {
840:   // QNNPack is not thread safe
841:   std::lock_guard<std::mutex> lock(qnnp_mutex_);
842:   const std::string func_name = transpose() ? "quantized::conv_transpose"
843:                                             : "quantized::conv";
844:   TORCH_CHECK(!(kReluFused && transpose()),
845:               kSpatialDim == 2,
846:               func_name, kSpatialDim,
847:               "d (qnnpack): ConvTranspose cannot be fused with ReLU.");
848:   TORCH_CHECK(act.scalar_type() == c10::kQUInt8,
849:               func_name,
850:               "(qnnpack): Expected activation data type ",
851:               toString(c10::kQUInt8),
852:               " but got ",
853:               toString(act.scalar_type()));
854:   ConvDimChecks<kSpatialDim>(
855:       act.ndimension(), stride().size(), padding().size(),
856:       output_padding().size(), dilation().size(), func_name, transpose());
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 858-908
```cpp
858:   auto* pack_w = w.get();
859:
860:   // TODO Can be replaced with packB->getOutputChannels() when update pre-pack
861:   // to actually do the packing.
862:   const int out_ch_idx = transpose() ? 1 : 0;
863:   const auto out_ch = bias.size(0);
864:   // inputs are in semantic NCHW format
865:   const int N = act.size(0);
866:   const int C = act.size(1);
867:   const int D = kSpatialDim == 3 ? act.size(2) : 1;
868:   const int H = act.size(kSpatialDim);
869:   const int W = act.size(kSpatialDim + 1);
870:   const int M = out_ch; // output channels
871:
872:   const auto channels_last = kSpatialDim == 2
873:       ? c10::MemoryFormat::ChannelsLast
874:       : c10::MemoryFormat::ChannelsLast3d;
875:   const at::Tensor act_ndhwc = act.contiguous(channels_last);
876:
877:   auto output_min = kReluFused
878:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
879:       ? activationLimits<uint8_t>(output_scale, output_zero_point, Activation::RELU)
880:             .first
881:       : std::numeric_limits<uint8_t>::min();
882:   auto output_max = kReluFused
883:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
884:       ? activationLimits<uint8_t>(output_scale, output_zero_point, Activation::RELU)
885:             .second
886:       : std::numeric_limits<uint8_t>::max();
887:
888:   double act_input_scale = act_ndhwc.q_scale();
889:
890:   // Re-quantizing the bias based on input scale and weight scale.
891:   if (!input_scale.has_value() || input_scale.value() != act_input_scale) {
892:     TORCH_CHECK(M == (transpose() ? groups() : 1) * orig_weight.size(out_ch_idx),
893:         "Output channel size of weight and bias must match.");
894:     TORCH_CHECK(C == (transpose() ? 1 : groups()) * orig_weight.size(1 - out_ch_idx),
895:         "Input channel size of weight and bias must match.");
896:
897:     // Get the original weight and adjust it to uint8 from int8
898:     auto weight_contig = orig_weight.contiguous(channels_last);
899:     auto bias_fp32 = bias;
900:     int8_t* w_data =
901:         reinterpret_cast<int8_t*>(weight_contig.template data_ptr<c10::qint8>());
902:
903:     float* weight_scales_data = w_scales.data_ptr<float>();
904:     // We calculate requant scale here as the vector holding the requant scale
905:     // is owned by this module. The pointer is then passed to qnnpack backend.
906:     generate_requantization_scales(
907:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
908:         w_scales, act_input_scale, output_scale, requantization_scales);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 910-968
```cpp
910:     // TODO Kimish, we are allocating affine_quantized regardless of per channel or not.
911:     // This allocation is actually used only for packing weight and thus will be freed.
912:     // Still we should be consistent. Fix this.
913:     at::Tensor qnnp_weight = at::_empty_affine_quantized(
914:         weight_contig.sizes(),
915:         at::device(c10::kCPU).dtype(c10::kQUInt8).memory_format(channels_last),
916:         weight_scales_data[0],
917:         w_zero_points[0],
918:         std::nullopt);
919:     auto* qnnp_w_data = qnnp_weight.template data_ptr<c10::quint8>();
920:     auto wt_numel = weight_contig.numel();
921:     for (const auto i : c10::irange(wt_numel)) {
922:       qnnp_w_data[i] = static_cast<c10::quint8>(w_data[i] + 128);
923:     }
924:     // Original bias was float, so we requantize it here.
925:     at::Tensor qbias = quant_utils::QuantizeBias(convolution_op->per_channel, bias_fp32, weight_contig, act_input_scale);
926:
927:     // Update the input scale to not pack again.
928:     input_scale = act_input_scale;
929:     w.reset();
930:     w = std::make_unique<qnnpack::PrePackConvWeights>(
931:         convolution_op.get(),
932:         w_zero_points.data(),
933:         reinterpret_cast<uint8_t*>(qnnp_w_data),
934:         reinterpret_cast<int32_t*>(qbias.template data_ptr<c10::qint32>()));
935:     pack_w = w.get();
936:     if (at::globalContext().releaseWeightsWhenPrepacking()) {
937:         // On mobile, we release the original weight by resetting the intrusive_ptr.
938:         // Calling unpack after this will throw an assertion.
939:         orig_weight.reset();
940:     }
941:
942:     // Set padding buffer to zero point. This can only be done if we want
943:     // to do it only once.
944:     if (zero_buffer_size) {
945:       memset(
946:           convolution_op->zero_buffer,
947:           act_ndhwc.q_zero_point(),
948:           zero_buffer_size);
949:     }
950:   }
951:
952:   TORCH_INTERNAL_ASSERT(pack_w != nullptr, "Packed Weights are NULL");
953:   at::SmallVector<int64_t, kSpatialDim + 2> output_shape;
954:   const auto input_shape = MakeInputShape<kSpatialDim>(D, H, W);
955:   if (transpose()) {
956:     output_shape = MakeDeConvOutputShape<kSpatialDim>(
957:         N,
958:         M,
959:         kSpatialDim == 2 ? std::vector<int64_t>{H, W} : std::vector<int64_t>{D, H, W},
960:         kernel_,
961:         stride(),
962:         padding(),
963:         output_padding(),
964:         dilation());
965:   } else {
966:     output_shape = at::native::quantized::MakeConvOutputShape<kSpatialDim>(
967:         N, M, input_shape, kernel_, stride(), padding(), dilation());
968:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 970-1027
```cpp
 970:   if (act_ndhwc.numel() > 0) {
 971:     TORCH_CHECK(
 972:         std::all_of(
 973:             output_shape.begin(),
 974:             output_shape.end(),
 975:             [](int64_t i) { return i > 0; }),
 976:         func_name,
 977:         kSpatialDim,
 978:         "d (qnnpack): each dimension of output tensor should "
 979:         "be greater than 0.")
 980:   }
 981:
 982:   // Allocate output Tensor and a buffer for QNNPACK to use
 983:   at::Tensor output = at::native::empty_affine_quantized(
 984:       output_shape,
 985:       c10::kQUInt8,
 986:       std::nullopt /* layout */,
 987:       c10::kCPU,
 988:       std::nullopt /* pin_memory */,
 989:       output_scale,
 990:       output_zero_point,
 991:       channels_last);
 992:
 993:   pytorch_qnnp_status run_status{};
 994:   if (transpose()) {
 995:     run_status = qnnpack::qnnpackDeConv(
 996:         convolution_op.get(),
 997:         pack_w->getPackedWeights(),
 998:         N,
 999:         H,
1000:         W,
1001:         act_ndhwc.q_zero_point(),
1002:         reinterpret_cast<uint8_t*>(act_ndhwc.template data_ptr<c10::quint8>()),
1003:         w_zero_points.data(),
1004:         requantization_scales.data(),
1005:         output.q_zero_point(),
1006:         output_min,
1007:         output_max,
1008:         reinterpret_cast<uint8_t*>(output.template data_ptr<c10::quint8>()),
1009:         caffe2::pthreadpool_());
1010:   } else {
1011:     run_status = qnnpack::qnnpackConv(
1012:         convolution_op.get(),
1013:         pack_w->getPackedWeights(),
1014:         N,
1015:         D,
1016:         H,
1017:         W,
1018:         act_ndhwc.q_zero_point(),
1019:         reinterpret_cast<uint8_t*>(act_ndhwc.template data_ptr<c10::quint8>()),
1020:         w_zero_points.data(),
1021:         requantization_scales.data(),
1022:         output.q_zero_point(),
1023:         output_min,
1024:         output_max,
1025:         reinterpret_cast<uint8_t*>(output.template data_ptr<c10::quint8>()),
1026:         caffe2::pthreadpool_());
1027:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1029-1075
```cpp
1029:   TORCH_INTERNAL_ASSERT(
1030:       run_status == pytorch_qnnp_status_success,
1031:       "failed to run quantized::conv2d (qnnpack) operator");
1032:
1033:   return output;
1034: }
1035:
1036: #ifdef USE_XNNPACK
1037: static bool can_use_xnnp(
1038:     c10::ScalarType dtype,
1039:     int kSpatialDim,
1040:     bool per_channel,
1041:     bool transpose) {
1042:   if (!at::native::xnnpack::available()) {
1043:     return false;
1044:   }
1045:   bool supported_dtypes = dtype == c10::kQInt8;
1046:   bool invalid_config =
1047:       (kSpatialDim != 2 /* No support for 3d convolution */
1048:         || (dtype == c10::kQInt8 && transpose &&
1049:             per_channel)); /* int8_t deconv does not support per-channel */
1050:   if (supported_dtypes && invalid_config) {
1051:     /* don't want this to fall through to QNNPACK */
1052:     const std::string func_name =
1053:         transpose ? "quantized::conv_transpose" : "quantized::conv";
1054:     TORCH_CHECK(
1055:         false,
1056:         func_name,
1057:         " (xnnpack): Unsupported conv config for dtype KQInt8");
1058:   }
1059:   return supported_dtypes && !invalid_config;
1060: }
1061: #endif  // USE_XNNPACK
1062:
1063: template <int kSpatialDim>
1064: at::Tensor PackedConvWeightsQnnp<kSpatialDim>::apply(
1065:     const at::Tensor& input,
1066:     double output_scale,
1067:     int64_t output_zero_point) {
1068: #ifdef USE_XNNPACK
1069:   if (can_use_xnnp(input.scalar_type(), kSpatialDim, per_channel(), transpose())) {
1070:     return apply_impl_xnnp<c10::qint8, false>(
1071:         input, output_scale, output_zero_point);
1072:   } /* fall through for unsupported types, configs, or shapes */
1073: #endif // USE_XNNPACK
1074:   return apply_impl<false>(input, output_scale, output_zero_point);
1075: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `can_use_xnnp`, `apply`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `can_use_xnnp`, `apply`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1077-1130
```cpp
1077: template <int kSpatialDim>
1078: at::Tensor PackedConvWeightsQnnp<kSpatialDim>::apply_relu(
1079:     const at::Tensor& input,
1080:     double output_scale,
1081:     int64_t output_zero_point) {
1082: #ifdef USE_XNNPACK
1083:   if (can_use_xnnp(input.scalar_type(), kSpatialDim, per_channel(), transpose())) {
1084:     return apply_impl_xnnp<c10::qint8, true>(
1085:         input, output_scale, output_zero_point);
1086:   } /* fall through for unsupported types, configs, or shapes */
1087: #endif // USE_XNNPACK
1088:   return apply_impl<true>(input, output_scale, output_zero_point);
1089: }
1090:
1091: template at::Tensor PackedConvWeightsQnnp<2>::apply(
1092:     const at::Tensor& act,
1093:     double output_scale,
1094:     int64_t output_zero_point);
1095:
1096: template at::Tensor PackedConvWeightsQnnp<2>::apply_relu(
1097:     const at::Tensor& act,
1098:     double output_scale,
1099:     int64_t output_zero_point);
1100:
1101: template at::Tensor PackedConvWeightsQnnp<3>::apply(
1102:     const at::Tensor& act,
1103:     double output_scale,
1104:     int64_t output_zero_point);
1105:
1106: template at::Tensor PackedConvWeightsQnnp<3>::apply_relu(
1107:     const at::Tensor& act,
1108:     double output_scale,
1109:     int64_t output_zero_point);
1110:
1111: template at::Tensor PackedConvWeightsQnnp<2>::apply_impl<false>(
1112:     const at::Tensor& act,
1113:     double output_scale,
1114:     int64_t output_zero_point);
1115:
1116: template at::Tensor PackedConvWeightsQnnp<3>::apply_impl<false>(
1117:   const at::Tensor& act,
1118:   double output_scale,
1119:   int64_t output_zero_point);
1120:
1121: #endif // USE_PYTORCH_QNNPACK
1122:
1123: #if AT_MKLDNN_ENABLED()
1124: template <int kSpatialDim>
1125: at::Tensor PackedConvWeightsOnednn<kSpatialDim>::apply(
1126:     const at::Tensor& input,
1127:     double output_scale,
1128:     int64_t output_zero_point) {
1129:   return apply_impl<false>(input, std::nullopt, output_scale, output_zero_point);
1130: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_relu`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_relu`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1132-1182
```cpp
1132: template <int kSpatialDim>
1133: at::Tensor PackedConvWeightsOnednn<kSpatialDim>::apply_relu(
1134:     const at::Tensor& input,
1135:     double output_scale,
1136:     int64_t output_zero_point) {
1137:   return apply_impl<true>(input, std::nullopt, output_scale, output_zero_point);
1138: }
1139:
1140: template <int kSpatialDim>
1141: at::Tensor PackedConvWeightsOnednn<kSpatialDim>::apply_add(
1142:     const at::Tensor& input,
1143:     const at::Tensor& accum,
1144:     double output_scale,
1145:     int64_t output_zero_point) {
1146:   TORCH_CHECK(kSpatialDim == 2, " Currently, only conv2d with add is supported.");
1147:   return apply_impl<false>(input, accum, output_scale, output_zero_point);
1148: }
1149:
1150: template <int kSpatialDim>
1151: at::Tensor PackedConvWeightsOnednn<kSpatialDim>::apply_add_relu(
1152:     const at::Tensor& input,
1153:     const at::Tensor& accum,
1154:     double output_scale,
1155:     int64_t output_zero_point) {
1156:   TORCH_CHECK(kSpatialDim == 2, " Currently, only conv2d add relu is supported.");
1157:   return apply_impl<true>(input, accum, output_scale, output_zero_point);
1158: }
1159:
1160: template <int kSpatialDim>
1161: template <bool kReluFused>
1162: at::Tensor PackedConvWeightsOnednn<kSpatialDim>::apply_impl(
1163:     const at::Tensor& act,
1164:     const std::optional<at::Tensor>& accum,
1165:     double output_scale,
1166:     int64_t output_zero_point) {
1167:   std::string func_name = "quantized::conv";
1168:   if (transpose()) {
1169:     func_name += "_transpose";
1170:   }
1171:   func_name += std::to_string(kSpatialDim) + "d";
1172:
1173:   // has_accum: extra input besides the conv to do conv add fusion.
1174:   bool has_accum = accum.has_value() ? true : false;
1175:   if (has_accum) {
1176:     auto& ctx = at::globalContext();
1177:     func_name += "_add";
1178:     TORCH_CHECK(
1179:       !transpose(),
1180:       "Didn't support transposed conv for conv with add ",
1181:       c10::toString(ctx.qEngine()));
1182:   }
```
- EN: The main symbol in this range is `apply_relu`, `apply_add`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `apply_relu`, `apply_add`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1184-1230
```cpp
1184:   if (kReluFused) {
1185:     func_name += "_relu";
1186:   }
1187:   ConvDimChecks<kSpatialDim>(
1188:       act.ndimension(), stride().size(), padding().size(),
1189:       output_padding().size(), dilation().size(), func_name, transpose());
1190:   TORCH_CHECK(act.scalar_type() == c10::ScalarType::QUInt8,
1191:       func_name, " (ONEDNN): data type of input should be QUint8.");
1192:
1193:   // src
1194:   auto act_contig = act.contiguous(kSpatialDim == 2 ? c10::MemoryFormat::ChannelsLast : c10::MemoryFormat::ChannelsLast3d);
1195:   auto src_dims = act_contig.sizes().vec();
1196:   auto src_data_type = dnnl::memory::data_type::u8;
1197:   auto src_desc = ideep::tensor::desc(src_dims, src_data_type,
1198:       kSpatialDim == 2 ? ideep::format_tag::nhwc : ideep::format_tag::ndhwc);
1199:   ideep::tensor src(src_desc, act_contig.data_ptr());
1200:   // weights & bias
1201:   ideep::tensor& weights = *(weight_);
1202:   bool with_bias = bias_.has_value();
1203:   const auto& kernel_size = weights.get_dims();
1204:   // dst
1205:   const std::vector<int64_t>& input_size = src.get_dims();
1206:   std::vector<int64_t> output_sizes;
1207:   if (transpose()) {
1208:     // Prepacked weight format: [o, i, ...]
1209:     const int N = act.size(0); // batch size
1210:     const int C = act.size(1); // input channels
1211:     const int M = weights.get_dim(0); // output channels
1212:     const int D = kSpatialDim == 2 ? 1 : act.size(2); // input depth
1213:     const int H = act.size(kSpatialDim); // input height
1214:     const int W = act.size(kSpatialDim + 1); // input width
1215:     const int KH = weights.get_dim(kSpatialDim); // kernel height
1216:     const int KW = weights.get_dim(kSpatialDim + 1); // kernel width
1217:     const int KD = kSpatialDim == 2 ? 1 : weights.get_dim(2); // kernel depth
1218:     TORCH_CHECK(C == groups() * weights.get_dim(1), // weight: [o, i, ...]
1219:                 func_name, " (ONEDNN): input channel number should be ",
1220:                 groups() * weights.get_dim(1), ", but got ", C);
1221:     auto output_shape = MakeDeConvOutputShape<kSpatialDim>(
1222:         N,
1223:         M,
1224:         kSpatialDim == 2 ? std::vector<int64_t>{H, W} : std::vector<int64_t>{D, H, W},
1225:         kSpatialDim == 2 ? std::vector<int64_t>{KH, KW} : std::vector<int64_t>{KD, KH, KW},
1226:         stride(),
1227:         padding(),
1228:         output_padding(),
1229:         dilation());
1230:     output_sizes = c10::IntArrayRef(output_shape).vec();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1231-1290
```cpp
1231:   } else {
1232:     output_sizes = at::native::conv_output_size(input_size, kernel_size, padding().vec(), stride().vec(), dilation().vec());
1233:   }
1234:   ideep::dims dst_dims = ideep::dims({output_sizes.cbegin(), output_sizes.cend()});
1235:   at::Tensor output = at::_empty_affine_quantized(
1236:       dst_dims,
1237:       at::device(c10::kCPU)
1238:           .dtype(c10::kQUInt8)
1239:           .memory_format(kSpatialDim == 2 ?
1240:               c10::MemoryFormat::ChannelsLast :
1241:               c10::MemoryFormat::ChannelsLast3d),
1242:       output_scale,
1243:       output_zero_point,
1244:       std::nullopt);
1245:   if (output.numel() == 0) {
1246:     return output;
1247:   }
1248:   ideep::tensor dst;
1249:   at::Tensor accum_contig;
1250:   if (has_accum) {
1251:     auto dst_desc = ideep::tensor::desc(dst_dims, src_data_type,
1252:         kSpatialDim == 2 ? ideep::format_tag::nhwc : ideep::format_tag::ndhwc);
1253:     accum_contig = accum.value().contiguous(kSpatialDim == 2 ? c10::MemoryFormat::ChannelsLast : c10::MemoryFormat::ChannelsLast3d);
1254:     TORCH_CHECK(accum_contig.dtype() == output.dtype(), "The output tensor should have same dtype as the accum tensor.");
1255:     // When fused with sum, the dst tensor will share the data ptr as the accum tensor.
1256:     dst.init(dst_desc, accum_contig.data_ptr());
1257:   } else {
1258:     dst = ideep::tensor({dst_dims, ideep::tensor::data_type::u8, {output.strides().cbegin(), output.strides().cend()}},
1259:                       output.data_ptr());
1260:   }
1261:
1262:   // Parameters
1263:   const ideep::dims& strides = stride().vec();
1264:   const ideep::dims& dilates = dilation().vec();
1265:   const ideep::dims& padding_l = padding().vec();
1266:   const ideep::dims& padding_r = padding().vec();
1267:   double input_scale = act.q_scale();
1268:   int64_t input_zp = act.q_zero_point();
1269:   // Scales of ONEDNN and PyTorch are reciprocal
1270:   const ideep::scale_t& src_scales = ideep::scale_t(1, 1.0/input_scale);
1271:   const ideep::scale_t& weights_scales = weights.get_scale();
1272:   double inv_output_scale = 1.0/output_scale;
1273:   const ideep::zero_point_t src_zero_points = ideep::zero_point_t(1, input_zp);
1274:   const ideep::zero_point_t dst_zero_points = ideep::zero_point_t(1, output_zero_point);
1275:
1276:   ideep::attr_t op_attr;
1277:   float sum_scale = has_accum ? accum.value().q_scale() : 1.0;
1278:   int32_t sum_zero_point = has_accum ? accum.value().q_zero_point() : 0;
1279:   if (has_accum) {
1280:     // Just tells we have these post op, the actual value such as scale and zero point will be set later.
1281:     op_attr = kReluFused ? ideep::attr_t::residual_with_sum_zero_point() : ideep::attr_t::fuse_sum();
1282:     const ideep::scale_t accum_scale = ideep::scale_t(1, 1.0/sum_scale);
1283:     const ideep::zero_point_t accum_zero_points = ideep::zero_point_t(1, sum_zero_point);
1284:     // Set the dst scale and zero point with the value of accum.
1285:     // The true scale and zero point is stored in ideep::scale_t(scale_size, inv_output_scale) and dst_zero_points.
1286:     dst.set_scale(accum_scale);
1287:     dst.set_zero_point(accum_zero_points);
1288:   } else if (kReluFused) {
1289:     op_attr = ideep::attr_t::fuse_relu();
1290:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1292-1338
```cpp
1292:   // Bias might be modified outside (e.g. by quantization bias correction).
1293:   // If so, update the prepacked bias as well.
1294:   if (with_bias && bias_.value().get_data_handle() != orig_bias_.value().data_ptr()) {
1295:     bias_.value().init(bias_.value().get_desc(), orig_bias_.value().data_ptr());
1296:   }
1297:   const auto& b = with_bias ? bias_.value() : ideep::tensor();
1298:   int num_threads = at::get_num_threads();
1299:   if (transpose()) {
1300:     // Primitive cache is initialized when called for the first time
1301:     // and won't be updated afterwards.
1302:     PrimitiveCacheKey cache_key = std::make_tuple(
1303:         input_scale, input_zp, src_dims, output_scale, output_zero_point, num_threads, sum_scale, sum_zero_point);
1304:     c10::call_once(*cache_initialized_flag, [&](){
1305:         DeconvParams params;
1306:         ideep::convolution_transpose_forward::prepare(
1307:             params, src, weights, b, dst_dims, dst,
1308:             strides, padding_l, padding_r, dilates, groups(),
1309:             src_scales, weights_scales, ideep::scale_t(1, inv_output_scale),
1310:             src_zero_points, dst_zero_points, op_attr,
1311:             dnnl::algorithm::deconvolution_direct,
1312:             dnnl::prop_kind::forward_inference,
1313:             ideep::u8s8, ideep::engine::cpu_engine());
1314:         get_deconv_cache() = DeconvPrimitiveCache(cache_key, params);
1315:         auto expected_weight_desc = ideep::tensor::desc(params.pd.weights_desc(), groups());
1316:         weights = weights.reorder_if_differ_in(expected_weight_desc);
1317:     });
1318:     if (get_deconv_cache().hit(cache_key)) {
1319:       DeconvParams& params = get_deconv_cache().get_params();
1320:       ideep::convolution_transpose_forward::compute<false, false>(
1321:           params, src, weights, b, dst);
1322:     } else {
1323:       ideep::convolution_transpose_forward::compute(
1324:           src, weights, b, dst_dims, dst,
1325:           strides, padding_l, padding_r, dilates,
1326:           groups(), src_scales, weights_scales,
1327:           ideep::scale_t(1, inv_output_scale),
1328:           src_zero_points, dst_zero_points, op_attr,
1329:           dnnl::algorithm::deconvolution_direct,
1330:           dnnl::prop_kind::forward_inference,
1331:           ideep::u8s8, ideep::engine::cpu_engine());
1332:     }
1333:   } else {  // not transposed
1334:     PrimitiveCacheKey cache_key = std::make_tuple(
1335:         input_scale, input_zp, src_dims, output_scale, output_zero_point, num_threads, sum_scale, sum_zero_point);
1336:     c10::call_once(*cache_initialized_flag, [&](){
1337:         ConvParams params;
1338:         ideep::convolution_forward::prepare(
```
- EN: The main symbol in this range is `outside`, `call_once`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `outside`, `call_once`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1339-1389
```cpp
1339:             params, src, weights, b, dst_dims, dst,
1340:             strides, dilates, padding_l, padding_r, groups(),
1341:             src_scales, weights_scales, ideep::scale_t(1, inv_output_scale),
1342:             src_zero_points, dst_zero_points,
1343:             op_attr, dnnl::algorithm::convolution_direct,
1344:             dnnl::prop_kind::forward_inference,
1345:             ideep::u8s8, ideep::engine::cpu_engine());
1346:         get_conv_cache() = ConvPrimitiveCache(cache_key, params);
1347:         auto expected_weight_desc = ideep::tensor::desc(params.pd.weights_desc(), groups());
1348:         weights = weights.reorder_if_differ_in(expected_weight_desc);
1349:     });
1350:     // If hit, use cached data. If miss, fall back to normal path.
1351:     if (get_conv_cache().hit(cache_key)) {
1352:       auto& params = get_conv_cache().get_params();
1353:       ideep::convolution_forward::compute<false, false>(params, src, weights, b, dst);
1354:     } else {
1355:       ideep::convolution_forward::compute(
1356:           src, weights, b, dst_dims, dst,
1357:           strides, dilates, padding_l, padding_r, groups(),
1358:           src_scales, weights_scales, ideep::scale_t(1, inv_output_scale),
1359:           src_zero_points, dst_zero_points, op_attr,
1360:           dnnl::algorithm::convolution_direct,
1361:           dnnl::prop_kind::forward_inference,
1362:           ideep::u8s8, ideep::engine::cpu_engine());
1363:     }
1364:   }
1365:   if (has_accum) {
1366:     // When fused with sum, the accum tensor share the data ptr as dst tensor as the output.
1367:     // Reset output's scale and zero point into accum_contig.
1368:     set_quantizer_(accum_contig, at::make_per_tensor_affine_quantizer(
1369:         output_scale, output_zero_point, accum_contig.scalar_type()));
1370:     return accum_contig;
1371:   } else {
1372:     return output;
1373:   }
1374: }
1375:
1376: template at::Tensor PackedConvWeightsOnednn<2>::apply(
1377:     const at::Tensor& act,
1378:     double output_scale,
1379:     int64_t output_zero_point);
1380:
1381: template at::Tensor PackedConvWeightsOnednn<2>::apply_relu(
1382:     const at::Tensor& act,
1383:     double output_scale,
1384:     int64_t output_zero_point);
1385:
1386: template at::Tensor PackedConvWeightsOnednn<3>::apply(
1387:     const at::Tensor& act,
1388:     double output_scale,
1389:     int64_t output_zero_point);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1391-1437
```cpp
1391: template at::Tensor PackedConvWeightsOnednn<3>::apply_relu(
1392:     const at::Tensor& act,
1393:     double output_scale,
1394:     int64_t output_zero_point);
1395:
1396: static at::Tensor _fp8_convolution_onednn_ref(
1397:     at::Tensor act, // contains quantized values but not QTensor
1398:     double act_scale,
1399:     at::Tensor weight, // MKLDNN tensor with quantized values
1400:     at::Tensor weight_scales,
1401:     std::optional<at::Tensor> bias, // Bias is not packed into MKLDNN tensor
1402:     torch::List<int64_t> stride,
1403:     torch::List<int64_t> padding,
1404:     torch::List<int64_t> dilation,
1405:     int64_t groups,
1406:     double output_scale,
1407:     std::optional<at::Tensor> accum, // accum to fused with conv add
1408:     double accum_scale,
1409:     std::optional<c10::ScalarType> output_dtype,
1410:     std::optional<std::string_view> binary_attr,
1411:     std::optional<at::Scalar> binary_alpha,
1412:     std::optional<std::string_view> unary_attr,
1413:     torch::List<std::optional<at::Scalar>> unary_scalars,
1414:     std::optional<std::string_view> unary_algorithm) {
1415:   TORCH_CHECK(
1416:     act.scalar_type() == at::ScalarType::Float8_e4m3fn && weight.scalar_type() == at::ScalarType::Float8_e4m3fn,
1417:     "FP8 qconv: Unexpected dtype of input and weight:", act.scalar_type(), ", ", weight.scalar_type());
1418:   int kSpatialDim = act.dim() - 2;
1419:   // conv1d is converted to conv2d before calling this function
1420:   TORCH_CHECK(kSpatialDim != 1, "Expect 2D or 3D convolution, but got 1D convolution.");
1421:   auto act_contig = act.contiguous(kSpatialDim == 2 ?
1422:                                    c10::MemoryFormat::ChannelsLast :
1423:                                    c10::MemoryFormat::ChannelsLast3d);
1424:   auto dqx = act_contig.to(at::kFloat) * act_scale;
1425:   std::vector<int64_t> w_scales_new_shape(weight.dim(), 1);
1426:   w_scales_new_shape[0] = -1;
1427:   auto dqw = weight.to(at::kFloat) * weight_scales.reshape(w_scales_new_shape);
1428:   auto output_padding = std::vector<int64_t>(kSpatialDim, 0);
1429:   auto bias_float = bias.has_value() ? bias.value().to(at::kFloat) : bias;
1430:   auto y_f32 = at::convolution(
1431:     dqx, dqw, bias_float, stride.vec(), padding.vec(), dilation.vec(), /* transposed */false, output_padding, groups
1432:   );
1433:   if (!binary_attr.has_value() || binary_attr == "none") {
1434:     if (unary_attr == "relu") {
1435:       at::relu_(y_f32);
1436:     } else if (unary_attr == "leaky_relu") {
1437:       TORCH_CHECK(
```
- EN: The main symbol in this range is `_fp8_convolution_onednn_ref`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_fp8_convolution_onednn_ref`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1438-1500
```cpp
1438:           unary_scalars.size() == 1,
1439:           "onednn qconv: expect one argument for post op leaky_relu but got ", unary_scalars.size(), " args");
1440:       auto element = unary_scalars.get(0);
1441:       auto alpha = element.value().to<float>();
1442:       at::leaky_relu_(y_f32, alpha);
1443:     } else if (unary_attr == "tanh") {
1444:       at::tanh_(y_f32);
1445:     } else if (unary_attr == "gelu") {
1446:       TORCH_CHECK(
1447:           unary_algorithm == "none" || unary_algorithm == "tanh",
1448:           "onednn qconv: algorithm for post op gelu must be none or tanh but got ", unary_algorithm);
1449:       at::gelu_(y_f32, unary_algorithm.value());
1450:     } else if (unary_attr == "hardtanh") {
1451:       TORCH_CHECK(
1452:           unary_scalars.size() == 2 &&
1453:               unary_scalars.get(0).has_value() &&
1454:               unary_scalars.get(1).has_value(),
1455:           "hardtanh is expected to have two scalar input: min_val and max_val");
1456:       auto lower_bound_value =
1457:           unary_scalars.get(0).value().to<float>();
1458:       auto upper_bound_value =
1459:           unary_scalars.get(1).value().to<float>();
1460:       at::hardtanh_(y_f32, lower_bound_value, upper_bound_value);
1461:     } else if (unary_attr == "hardswish") {
1462:       at::hardswish_(y_f32);
1463:     } else if (unary_attr == "swish") {
1464:       y_f32 = y_f32 * at::sigmoid(y_f32);
1465:     } else {
1466:       TORCH_CHECK(
1467:           !unary_attr.has_value() || unary_attr == "none",
1468:           "onednn qconv: unsupported unary post op ", unary_attr);
1469:     }
1470:   } else if (binary_attr == "sum") {
1471:     TORCH_CHECK(accum.has_value(), "onednn qconv: the extra input is missing for post op sum");
1472:     auto x1 = accum.value();
1473:     TORCH_CHECK(x1.sizes() == y_f32.sizes());
1474:     auto x1_f32 = x1.to(at::kFloat) * accum_scale;
1475:     x1_f32 = x1_f32.view(y_f32.sizes());
1476:     if (!unary_attr.has_value() || unary_attr == "none") {
1477:       y_f32.add_(x1_f32);
1478:     } else if (unary_attr == "relu") {
1479:       y_f32.add_(x1_f32).relu_();
1480:     } else {
1481:       TORCH_CHECK(
1482:           false,
1483:           "onednn qconv: unsupported unary post op ", unary_attr, " with binary post op sum");
1484:     }
1485:     y_f32.div_(output_scale);
1486:     if (x1.scalar_type() == at::kFloat8_e4m3fn) {
1487:       // Avoid NaN
1488:       y_f32.clamp_(-FP8E4M3_MAX, FP8E4M3_MAX);
1489:       // Align with oneDNN: convert fp32 to fp8 by fp32 -> fp16 -> fp8
1490:       y_f32 = y_f32.to(at::kHalf);
1491:     }
1492:     x1.copy_(y_f32.to(x1.scalar_type()).view(x1.sizes()));
1493:     // Return a copy: custom ops must not return tensors that alias inputs.
1494:     // The accum buffer has already been mutated in-place above.
1495:     return x1.clone();
1496:   } else {
1497:     TORCH_CHECK(
1498:         false,
1499:         "onednn qconv: unsupported binary post op ", binary_attr);
1500:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 1502-1555
```cpp
1502:   y_f32.div_(output_scale);
1503:   auto out_dtype = output_dtype.has_value() ? output_dtype.value() : at::kFloat8_e4m3fn;
1504:   if (out_dtype == at::kFloat8_e4m3fn) {
1505:     // Avoid NaN
1506:     y_f32.clamp_(-FP8E4M3_MAX, FP8E4M3_MAX);
1507:     // Align with oneDNN: convert fp32 to fp8 by fp32 -> fp16 -> fp8
1508:     return y_f32.to(at::kHalf).to(out_dtype);
1509:   }
1510:   return y_f32.to(out_dtype);
1511: }
1512:
1513: static at::Tensor _quantized_convolution_onednn(
1514:     at::Tensor act, // contains quantized values but not QTensor
1515:     double act_scale,
1516:     int64_t act_zero_point,
1517:     at::Tensor weight, // MKLDNN tensor with quantized values
1518:     at::Tensor weight_scales,
1519:     at::Tensor weight_zero_points,
1520:     std::optional<at::Tensor> bias, // Bias is not packed into MKLDNN tensor
1521:     torch::List<int64_t> stride,
1522:     torch::List<int64_t> padding,
1523:     torch::List<int64_t> dilation,
1524:     bool transposed,
1525:     int64_t groups,
1526:     double output_scale,
1527:     int64_t output_zero_point,
1528:     std::optional<at::Tensor> accum, // accum to fused with conv add
1529:     double accum_scale,
1530:     int64_t accum_zero_point,
1531:     std::optional<c10::ScalarType> output_dtype,
1532:     std::optional<std::string_view> binary_attr,
1533:     std::optional<at::Scalar> binary_alpha,
1534:     std::optional<std::string_view> unary_attr,
1535:     torch::List<std::optional<at::Scalar>> unary_scalars,
1536:     std::optional<std::string_view> unary_algorithm) {
1537:   using ideep::tensor;
1538:   /*********************************/
1539:   /*          Checks               */
1540:   /*********************************/
1541:   // Due the constant folding inside Inductor freeze,
1542:   // https://github.com/pytorch/pytorch/blob/b99d605a3070de35677cc43f0196c2f2e807b822/torch/ao/quantization/fx/_decomposed.py#L62-L63
1543:   // inv_scale = 1.0 / scale will be folded.
1544:   // So, we can only get inv_scale from quant node which is used as
1545:   // output_scale of this op.
1546:   bool fp32_output = output_dtype.has_value() && (output_dtype.value() == c10::kFloat);
1547:   bool bfloat16_output = output_dtype.has_value() && (output_dtype.value() == c10::kBFloat16);
1548:   if (fp32_output || bfloat16_output) {
1549:     // When fp32 or bf16 output, oneDNN expects op_attr doesn't set_scales and set_zero_points.
1550:     // So, we will use default output_scale as 1.0 and output_zero_point as 0, since
1551:     // when output_scale is 1.0, we will skip invoking of op_attr.set_scales in ideep;
1552:     // when output_zero_point is 0, we will skip invoking of op_attr.set_zero_points in ideep.
1553:     TORCH_CHECK(output_scale == 1.0,  " (ONEDNN): fp32 or bf16 output, output_scale must be 1.0.");
1554:     TORCH_CHECK(output_zero_point == 0,  " (ONEDNN): fp32 or bf16 output, output_zero_point must be 0");
1555:   }
```
- EN: The main symbol in this range is `_quantized_convolution_onednn`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `_quantized_convolution_onednn`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1557-1603
```cpp
1557:   int kSpatialDim = act.dim() - 2;
1558:   bool is_1d = (1 == kSpatialDim);
1559:
1560:   bool has_binary_post_op = binary_attr.has_value() && binary_attr.value() != "none";
1561:   bool has_unary_post_op = unary_attr.has_value() && unary_attr.value() != "none";
1562:   // has_accum_postop_sum: extra input besides the conv to do conv post op sum fusion.
1563:   bool has_accum_postop_sum = has_binary_post_op && binary_attr.value() == "sum";
1564:
1565:   if (has_accum_postop_sum) {
1566:     TORCH_CHECK(accum.has_value(), "For post op sum, accum tensor should not be empty.");
1567:     TORCH_CHECK(
1568:       accum.value().is_contiguous(
1569:         kSpatialDim == 2
1570:         ? c10::MemoryFormat::ChannelsLast
1571:         : c10::MemoryFormat::ChannelsLast3d
1572:       ),
1573:       "For post op sum, accum tensor must be contiguous."
1574:     );
1575:     if (fp32_output || bfloat16_output) {
1576:       TORCH_CHECK(accum_scale == 1.0,  " (ONEDNN): fp32 or bf16 output, accum_scale must be 1.0.");
1577:       TORCH_CHECK(accum_zero_point == 0,  " (ONEDNN): fp32 or bf16 output, accum_zero_point must be 0");
1578:       TORCH_CHECK((accum.value().scalar_type() == c10::kFloat) || (accum.value().scalar_type() == c10::kBFloat16), "The accum tensor should be KFloat or KBFloat.");
1579:     }
1580:   }
1581:
1582:   std::string func_name = "quantized::packed_weights_conv";
1583:   func_name += std::to_string(kSpatialDim) + "d";
1584:   if (has_binary_post_op) {
1585:     func_name += binary_attr.value().data();
1586:   }
1587:   if (has_unary_post_op) {
1588:     func_name += unary_attr.value().data();
1589:   }
1590:
1591:   if (kSpatialDim == 1) {
1592:     kSpatialDim += 1;
1593:   }
1594:   if (transposed) {
1595:     TORCH_CHECK(
1596:       false,
1597:       func_name, ": to support transposed convolution."
1598:     );
1599:   }
1600:   if (is_1d) {
1601:     // N, C, L -> N, C, 1, L
1602:     act = act.unsqueeze(quant_utils::kConv1dSqueezeDim + 2);
1603:     stride = quant_utils::MakeArgForConv1d(stride, 1);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1604-1651
```cpp
1604:     padding = quant_utils::MakeArgForConv1d(padding, 0);
1605:     dilation = quant_utils::MakeArgForConv1d(dilation, 1);
1606:   }
1607:   auto act_dtype = act.scalar_type();
1608:   TORCH_CHECK(
1609:     act_dtype == c10::ScalarType::Byte || act_dtype == c10::ScalarType::Float8_e4m3fn,
1610:     func_name, ": Input tensor should have uint8 (unsigned char) or fp8 data type");
1611:   TORCH_CHECK(
1612:     weight.scalar_type() == c10::ScalarType::Char || weight.scalar_type() == c10::ScalarType::Float8_e4m3fn,
1613:     func_name, ": Weight tensor should have int8 (char) or fp8 data type");
1614:   TORCH_CHECK(
1615:     weight.ndimension() == kSpatialDim + 2,
1616:     func_name, ": Weights are expected to have ", kSpatialDim + 2, " dimensions");
1617:   TORCH_CHECK(
1618:     stride.size() == (decltype(stride.size()))kSpatialDim,
1619:     func_name, ": stride should contain ", kSpatialDim, " elements for ",
1620:     kSpatialDim, "D convolution.");
1621:   TORCH_CHECK(
1622:     padding.size() == (decltype(padding.size()))kSpatialDim,
1623:     func_name, ": Specify front/top/left padding only. "
1624:     "end/bottom/right padding assumed to be equal to front/top/left");
1625:   TORCH_CHECK(
1626:     dilation.size() == (decltype(dilation.size()))kSpatialDim,
1627:     func_name, ": dilation should contain ", kSpatialDim, " elements for ",
1628:     kSpatialDim, "D convolution.");
1629:   bool is_fp8 = weight.scalar_type() == c10::ScalarType::Float8_e4m3fn;
1630: #ifdef ONEDNN_FP8_QCONV_SUPPORTED
1631:   if (is_fp8 && !cpuinfo_has_x86_amx_fp16()) {
1632: #else
1633:   if (is_fp8) {
1634: #endif
1635:     TORCH_CHECK(act_dtype == c10::ScalarType::Float8_e4m3fn,
1636:       func_name, ": expect input tensor to have fp8 data type, but got ", act_dtype);
1637:     TORCH_CHECK(act_zero_point == 0,
1638:       func_name, ": fp8 input should not have zero point.");
1639:     // the current version of oneDNN does not fp8 conv yet
1640:     // TODO(weiwen) Refine this part when oneDNN supports fp8 conv
1641:     auto out = _fp8_convolution_onednn_ref(
1642:         act, act_scale, weight, weight_scales,
1643:         bias, stride, padding, dilation, groups,
1644:         output_scale, accum, accum_scale,
1645:         output_dtype, binary_attr, binary_alpha, unary_attr,
1646:         unary_scalars, unary_algorithm);
1647:     if (is_1d) {
1648:       out.squeeze_(quant_utils::kConv1dSqueezeDim + 2);
1649:     }
1650:     return out;
1651:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1653-1700
```cpp
1653:   TORCH_CHECK(
1654:     weight.is_mkldnn(),
1655:     func_name, ": Weight should be prepacked as an MKLDNN tensor"
1656:   );
1657:
1658:   // Parameters
1659: #if IDEEP_PREREQ(3, 1, 0, 1)
1660:   // 1. If the weight scale generated by observer should with dtype float32
1661:   // https://github.com/pytorch/pytorch/blob/d2c24eca8a60c56b31ca967a44d5cc4522802aa6/torch/ao/quantization/observer.py#L323
1662:   // 2. If the weight scale got from the quantized tensor, like did in the UT. It's with dtype of double.
1663:   // https://github.com/pytorch/pytorch/blob/d2fa3f608b5e4f582a8aaf752f10efe4ca72a7d0/aten/src/ATen/quantized/Quantizer.cpp#L69
1664:   TORCH_CHECK(
1665:     weight_scales.scalar_type() == c10::ScalarType::Double || weight_scales.scalar_type() == c10::ScalarType::Float,
1666:     "weight_scales should be with data type Double or float");
1667:   if (weight_scales.scalar_type() == c10::ScalarType::Double) {
1668:     // For case 2, we will convert it from double to float, since ideep::scale_t is alias of std::vector<float>
1669:     weight_scales = weight_scales.to(c10::ScalarType::Float);
1670:   }
1671:   TORCH_CHECK(
1672:     weight_scales.ndimension() == 0 ||
1673:     (weight_scales.strides().size() == 1 || weight_scales.stride(0) == 1),
1674:     "weight_scales should be scalar tensor or contiguous 1D tensor.");
1675:   ideep::scale_t weights_scales(weight_scales.data_ptr<float>(), weight_scales.data_ptr<float>()+weight_scales.numel());
1676: #elif IDEEP_PREREQ(3, 1, 0, 0)
1677:   // TODO (leslie): optimize the performance here:
1678:   // 1. Remove the reciprocal of weight scale, we have done the reciprocal of weight scale back in Ideep:
1679:   // https://github.com/intel/ideep/blob/3c90e365526e19c110371d23831678a7e9d4353d/include/ideep/operators/conv.hpp#L163-L168
1680:   // 2. Remove 2 memory copies of weight_scales:
1681:   //   2.1 Input of weights_scales is PyTorch Dense tensor, we convert it to vector<float>
1682:   //   2.2 OneDNN stream submit convert weights_scales from vector to ideep::tensor
1683:   //   https://github.com/intel/ideep/blob/3c90e365526e19c110371d23831678a7e9d4353d/include/ideep/operators/conv.hpp#L1855-L1860
1684:   // We should be able to directly convert weights_scales from PyTorch Dense Tensor to IDeep Tensor which can share same data ptr.
1685:   ideep::scale_t weights_scales(weight_scales.numel());
1686:   if (weight_scales.ndimension() == 0) {
1687:     // Weight is quant per tensor, then weight_scales will be a scalar Tensor
1688:     weights_scales[0] = 1.0 / weight_scales.item().toDouble(); // Scales of ONEDNN and PyTorch are reciprocal
1689:   } else {
1690:     // Weight is quant per channel
1691:     for (int i = 0; i < weight_scales.numel(); ++i) {
1692:       weights_scales[i] = 1.0 / weight_scales[i].item().toDouble();
1693:     }
1694:   }
1695: #else
1696:   TORCH_CHECK(false, "Unexpected IDeep version to do qconv calculation.");
1697: #endif
1698:
1699:   const ideep::zero_point_t src_zero_points = ideep::zero_point_t(1, act_zero_point);
1700:   const ideep::zero_point_t dst_zero_points = ideep::zero_point_t(1, output_zero_point);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 1702-1748
```cpp
1702:   // Weight
1703:   auto packed_weight = at::native::itensor_from_mkldnn(weight);
1704:
1705:   // Bias
1706:   ideep::tensor onednn_bias;
1707:   const int output_channels = weight.size(0);
1708:   bool with_bias = bias.has_value();
1709:
1710:   at::Tensor bias_val_float;
1711:   if (with_bias) {
1712:     // For int8-mixed-bf16, we will also use float32 bias
1713:     bias_val_float = bias.value().to(at::kFloat);
1714:     TORCH_CHECK(bias_val_float.dim() == 1, "bias should be a vector (1D Tensor)");
1715:     TORCH_CHECK(
1716:         bias_val_float.size(0) == output_channels,
1717:         "bias should have K elements: " + std::to_string(output_channels));
1718:     auto bias_desc = ideep::tensor::desc(bias_val_float.sizes().vec(), dnnl::memory::data_type::f32);
1719:     onednn_bias.init(bias_desc, bias_val_float.data_ptr());
1720:   }
1721:
1722:   const auto& expected_bias = with_bias ? onednn_bias : ideep::tensor();
1723:
1724:   /*********************************/
1725:   /*        Computation            */
1726:   /*********************************/
1727:   // src
1728:   auto act_contig = act.contiguous(kSpatialDim == 2 ?
1729:                                    c10::MemoryFormat::ChannelsLast :
1730:                                    c10::MemoryFormat::ChannelsLast3d);
1731:   auto src_dims = act_contig.sizes().vec();
1732:   auto src_data_type = at::native::get_mkldnn_dtype(act.scalar_type());
1733:   auto src_desc = ideep::tensor::desc(src_dims, src_data_type,
1734:       kSpatialDim == 2 ? ideep::format_tag::nhwc : ideep::format_tag::ndhwc);
1735:   ideep::tensor src;
1736:   src.init(src_desc, act_contig.data_ptr());
1737:   // dst
1738:   const std::vector<int64_t>& input_size = src.get_dims();
1739:   const auto& kernel_size = packed_weight.get_dims();
1740:   std::vector<int64_t> output_sizes;
1741:   output_sizes = at::native::conv_output_size(input_size, kernel_size, padding.vec(), stride.vec(), dilation.vec());
1742:   ideep::dims dst_dims = ideep::dims({output_sizes.cbegin(), output_sizes.cend()});
1743:   // Output is not a quantized tensor but data type is uint8
1744:   auto out_dtype = output_dtype.has_value() ? output_dtype.value() : act_dtype;
1745:   at::Tensor output = has_accum_postop_sum ?
1746:     accum.value() :
1747:     at::empty(
1748:       dst_dims,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1749-1796
```cpp
1749:       at::device(c10::kCPU)
1750:           .dtype(out_dtype)
1751:           .memory_format(kSpatialDim == 2 ?
1752:               c10::MemoryFormat::ChannelsLast :
1753:               c10::MemoryFormat::ChannelsLast3d)
1754:     );
1755:   if (output.numel() == 0) {
1756:     // When has_accum_postop_sum, output aliases accum (the input). Custom ops
1757:     // must not return tensors that alias inputs, so return a copy.
1758:     return has_accum_postop_sum ? output.clone() : output;
1759:   }
1760:   ideep::tensor dst = at::native::itensor_view_from_dense(output);
1761:   static ideep::tensor::desc dummy_accum_desc;
1762:   ideep::attr_t op_attr = onednn_utils::create_attr_by_post_op(
1763:     binary_attr.has_value() ? binary_attr.value() : "none",
1764:     binary_alpha.has_value() ? binary_alpha.value().to<double>() : 1.0,
1765:     accum_scale,
1766:     accum_zero_point,
1767:     dummy_accum_desc,
1768:     unary_attr.has_value() ? unary_attr.value() : "none",
1769:     unary_scalars,
1770:     unary_algorithm.has_value() ? unary_algorithm.value() : ""
1771:   );
1772:   // Avoid NaN if output dtype is fp8
1773:   if (out_dtype == c10::kFloat8_e4m3fn) {
1774:     // To avoid NaN, we need to clamp the intermediate results (in fp32) to [-488, 488]
1775:     // before converting to fp8
1776:     auto post_ops = op_attr.get_post_ops();
1777:     post_ops.append_eltwise(dnnl::algorithm::eltwise_linear, 1.0/output_scale, 0.0);
1778:     post_ops.append_eltwise(dnnl::algorithm::eltwise_clip, -FP8E4M3_MAX, FP8E4M3_MAX);
1779:     op_attr.set_post_ops(post_ops);
1780:     output_scale = 1.0f;
1781:   }
1782:
1783: #if IDEEP_PREREQ(3, 1, 0, 0)
1784:   // Use oneDNN's APIs instead of prepare/compute from ideep to reduce integration overhead.
1785:   // The functions from ideep are heavy because they have complex data structures for unified API
1786:   // oneDNN version >= 3.1.0 is required.
1787:   auto weight_grouped = packed_weight.make_grouped_weights(groups, /* is_deconv */false);
1788:   auto weights_desc = tensor::desc(weight_grouped.get_dims(), packed_weight.get_data_type(), ideep::format_tag::any);
1789:   if (groups > 1) {
1790:     weights_desc = weights_desc.to_grouped(groups);
1791:   }
1792:   auto dst_desc = dst.get_desc();
1793:   auto bias_desc = with_bias ?
1794:       tensor::desc(expected_bias.get_dims(), ideep::data_type::f32, ideep::format_tag::any) :
1795:       tensor::desc();
1796:   if (act_scale != 1.0f) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1797-1861
```cpp
1797:     op_attr.set_scales_mask(DNNL_ARG_SRC, 0);
1798:   }
1799:   if (act_zero_point != 0) {
1800:     op_attr.set_zero_points_mask(DNNL_ARG_SRC, 0);
1801:   }
1802:   int oc_per_group = weight_grouped.get_dim(0) / groups;
1803:   int wei_scale_mask = ideep::utils::conv_weight_scale_mask(weight_scales.numel(), oc_per_group, groups, false);
1804:   op_attr.set_scales_mask(DNNL_ARG_WEIGHTS, wei_scale_mask);
1805:   if (output_scale != 1.0f) {
1806:     op_attr.set_scales_mask(DNNL_ARG_DST, 0);
1807:   }
1808:   if (output_zero_point != 0) {
1809:     op_attr.set_zero_points_mask(DNNL_ARG_DST, 0);
1810:   }
1811:   op_attr.set_scratchpad_mode(dnnl::scratchpad_mode::user);
1812:   auto engine = ideep::engine::cpu_engine();
1813:   auto dilates_dnnl = ideep::utils::get_compatible_dilates(dilation.vec());
1814:   auto primitive_desc = with_bias ?
1815:       dnnl::convolution_forward::primitive_desc(
1816:         engine, dnnl::prop_kind::forward_inference, dnnl::algorithm::convolution_direct,
1817:         src_desc, weights_desc, bias_desc, dst_desc,
1818:         stride.vec(), dilates_dnnl, padding.vec(), padding.vec(), op_attr
1819:       ) :
1820:       dnnl::convolution_forward::primitive_desc(
1821:         engine, dnnl::prop_kind::forward_inference, dnnl::algorithm::convolution_direct,
1822:         src_desc, weights_desc, dst_desc,
1823:         stride.vec(), dilates_dnnl, padding.vec(), padding.vec(), op_attr
1824:       );
1825:   auto primitive = dnnl::convolution_forward(primitive_desc);
1826:
1827:   // Reorder weight if needed
1828:   auto expected_weight = weight_grouped.reorder_if_differ_in(primitive_desc.weights_desc());
1829:
1830:   // Prepare args and execute primitive
1831:   tensor scratchpad(primitive_desc.scratchpad_desc());
1832:   ideep::exec_args args;
1833:   args.insert({DNNL_ARG_SRC, src});
1834:   args.insert({DNNL_ARG_WEIGHTS, expected_weight});
1835:   args.insert({DNNL_ARG_DST, dst});
1836:   args.insert({DNNL_ARG_SCRATCHPAD, scratchpad});
1837:   if (with_bias) {
1838:     args.insert({DNNL_ARG_BIAS, expected_bias});
1839:   }
1840:   tensor src_scales_t = tensor(ideep::scale_t(1, act_scale));
1841:   tensor wei_scales_t = tensor(weights_scales);
1842:   tensor dst_scales_t = tensor(ideep::scale_t(1, output_scale));
1843:   tensor src_zp_t = tensor(ideep::zero_point_t(1, act_zero_point));
1844:   tensor dst_zp_t = tensor(ideep::zero_point_t(1, output_zero_point));
1845:   if (act_scale != 1.0f) {
1846:     args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_SRC, src_scales_t});
1847:   }
1848:   if (output_scale != 1.0f) {
1849:     args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_DST, dst_scales_t});
1850:   }
1851:   args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_WEIGHTS, wei_scales_t});
1852:   if (act_zero_point != 0) {
1853:     args.insert({DNNL_ARG_ATTR_ZERO_POINTS | DNNL_ARG_SRC, src_zp_t});
1854:   }
1855:   if (output_zero_point != 0) {
1856:     args.insert({DNNL_ARG_ATTR_ZERO_POINTS | DNNL_ARG_DST, dst_zp_t});
1857:   }
1858:   primitive.execute(ideep::stream::default_stream(), args);
1859: #else
1860:   // Scales of ONEDNN and PyTorch are reciprocal
1861:   const ideep::scale_t& src_scales = ideep::scale_t(1, 1.0 / act_scale);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 1863-1925
```cpp
1863:   // set accum scale/zero point to dst
1864:   if (has_accum_postop_sum) {
1865:     const ideep::scale_t accum_ideep_scale = ideep::scale_t(1, 1.0/accum_scale);
1866:     const ideep::zero_point_t accum_ideep_zero_points = ideep::zero_point_t(1, accum_zero_point);
1867:     // Set the dst scale and zero point with the value of accum.
1868:     // The true scale and zero point is stored in ideep::scale_t(scale_size, output_scale) and dst_zero_points.
1869:     dst.set_scale(accum_ideep_scale);
1870:     dst.set_zero_point(accum_ideep_zero_points);
1871:   }
1872:
1873:   // Weight Reorder
1874:   ConvParams params;
1875:   ideep::convolution_forward::prepare(
1876:       params, src, packed_weight, expected_bias, dst_dims, dst,
1877:       stride.vec(), dilation.vec(), padding.vec(), padding.vec(), groups,
1878:       src_scales, weights_scales, ideep::scale_t(1, 1.0f / output_scale),
1879:       src_zero_points, dst_zero_points,
1880:       op_attr, dnnl::algorithm::convolution_direct,
1881:       dnnl::prop_kind::forward_inference,
1882:       ideep::u8s8, ideep::engine::cpu_engine());
1883:   auto expected_weight_desc = ideep::tensor::desc(params.pd.weights_desc(), groups);
1884:   ideep::tensor expected_weight = packed_weight.reorder_if_differ_in(expected_weight_desc);
1885:
1886:   // Computation
1887:   ideep::convolution_forward::compute<false, false>(params, src, expected_weight, expected_bias, dst);
1888: #endif
1889:
1890:   if (is_1d) {
1891:     output.squeeze_(quant_utils::kConv1dSqueezeDim + 2);
1892:   }
1893:   if (has_accum_postop_sum) {
1894:     // When has_accum_postop_sum, output aliases accum (the input) — see
1895:     // assignment above. Return a copy: custom ops must not return tensors
1896:     // that alias inputs.
1897:     return output.clone();
1898:   } else {
1899:     return output;
1900:   }
1901: }
1902:
1903: #endif // #if AT_MKLDNN_ENABLED()
1904:
1905: namespace at::native {
1906:
1907:   at::Tensor QConvoneDNN::run_pointwise(
1908:       at::Tensor act, // contains quantized values but not QTensor
1909:       double act_scale,
1910:       int64_t act_zero_point,
1911:       at::Tensor weight, // contains quantized values but not QTensor
1912:       at::Tensor weight_scales,
1913:       at::Tensor weight_zero_points,
1914:       std::optional<at::Tensor> bias,
1915:       torch::List<int64_t> stride,
1916:       torch::List<int64_t> padding,
1917:       torch::List<int64_t> dilation,
1918:       int64_t groups,
1919:       double output_scale,
1920:       int64_t output_zero_point,
1921:       std::optional<c10::ScalarType> output_dtype,
1922:       std::string_view attr,
1923:       torch::List<std::optional<at::Scalar>> scalars,
1924:       std::optional<std::string_view> algorithm) {
1925: #if AT_MKLDNN_ENABLED()
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `run_pointwise`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `run_pointwise`，它们直接构成本文件的算子逻辑。

### Lines 1927-1981
```cpp
1927:     std::vector<std::string> supported_postop = {
1928:       "none"
1929:     };
1930:     if (act.dim() == 3) {
1931:       // Conv1D post op
1932:       supported_postop.emplace_back("relu");
1933:     } else if (act.dim() == 4) {
1934:       // Conv2D post op
1935:       supported_postop.emplace_back("relu");
1936:       supported_postop.emplace_back("hardtanh");
1937:       supported_postop.emplace_back("hardswish");
1938:       supported_postop.emplace_back("swish");
1939:     }
1940:     TORCH_CHECK(
1941:       std::find(supported_postop.begin(), supported_postop.end(), attr) != supported_postop.end(),
1942:       "Unsupported post op ",
1943:       attr,
1944:       " for quantized pointwise conv",
1945:       act.dim()-2,
1946:       "d.")
1947:     return _quantized_convolution_onednn(
1948:         act, act_scale, act_zero_point,
1949:         weight, weight_scales, weight_zero_points,
1950:         bias, stride, padding, dilation, /*transposed*/false,
1951:         groups, output_scale, output_zero_point,
1952:         /*accum*/std::nullopt, /*accum_scale*/0.0, /*accum_zero_point*/0,
1953:         /*output_dtype*/output_dtype, /*binary_attr*/std::nullopt, /*binary_alpha*/std::nullopt,
1954:         /*unary_attr*/attr, /*unary_scalars*/scalars, /*unary_algorithm*/algorithm
1955:     );
1956: #else
1957:     TORCH_CHECK(false, "Unimplemented as onednn is not available.")
1958: #endif
1959:   }
1960:
1961:   at::Tensor QConvoneDNN::run_pointwise_tensor(
1962:       at::Tensor act, // contains quantized values but not QTensor
1963:       at::Tensor act_scale,
1964:       at::Tensor act_zero_point,
1965:       at::Tensor weight, // contains quantized values but not QTensor
1966:       at::Tensor weight_scales,
1967:       at::Tensor weight_zero_points,
1968:       std::optional<at::Tensor> bias,
1969:       torch::List<int64_t> stride,
1970:       torch::List<int64_t> padding,
1971:       torch::List<int64_t> dilation,
1972:       int64_t groups,
1973:       double output_scale,
1974:       int64_t output_zero_point,
1975:       std::optional<c10::ScalarType> output_dtype,
1976:       std::string_view attr,
1977:       torch::List<std::optional<at::Scalar>> scalars,
1978:       std::optional<std::string_view> algorithm) {
1979: #if AT_MKLDNN_ENABLED()
1980:     TORCH_CHECK(act_scale.numel() == 1 && act_zero_point.numel() == 1,
1981:         "onednn int8 linear: act scale/zp size should be 1");
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run_pointwise_tensor`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run_pointwise_tensor`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1983-2048
```cpp
1983:     return run_pointwise(
1984:         act, act_scale.item().toDouble(), act_zero_point.item().toLong(),
1985:         weight, weight_scales, weight_zero_points,
1986:         bias, stride, padding, dilation,
1987:         groups, output_scale, output_zero_point,
1988:         /*output_dtype*/output_dtype,
1989:         /*unary_attr*/attr, /*unary_scalars*/scalars, /*unary_algorithm*/algorithm
1990:     );
1991: #else
1992:     TORCH_CHECK(false, "Unimplemented as onednn is not available.")
1993: #endif
1994:   }
1995:
1996:
1997:   at::Tensor QConvoneDNN::run_pointwise_binary(
1998:       at::Tensor act, // contains quantized values but not QTensor
1999:       double act_scale,
2000:       int64_t act_zero_point,
2001:       at::Tensor weight, // contains quantized values but not QTensor
2002:       at::Tensor weight_scales,
2003:       at::Tensor weight_zero_points,
2004:       at::Tensor accum, // contains quantized values but not QTensor
2005:       std::optional<at::Tensor> bias,
2006:       torch::List<int64_t> stride,
2007:       torch::List<int64_t> padding,
2008:       torch::List<int64_t> dilation,
2009:       int64_t groups,
2010:       double output_scale,
2011:       int64_t output_zero_point,
2012:       std::optional<c10::ScalarType> output_dtype,
2013:       double accum_scale,
2014:       int64_t accum_zero_point,
2015:       std::string_view binary_attr,
2016:       std::optional<at::Scalar> alpha,
2017:       std::optional<std::string_view> unary_attr,
2018:       torch::List<std::optional<at::Scalar>> unary_scalars,
2019:       std::optional<std::string_view> unary_algorithm) {
2020: #if AT_MKLDNN_ENABLED()
2021:     // Conv2D post op check
2022:     TORCH_CHECK(
2023:       act.dim() == 4 && binary_attr == "sum" && (
2024:         !unary_attr.has_value() ||
2025:         (unary_attr.has_value() &&
2026:           (
2027:             unary_attr.value() == "none" || unary_attr.value() == "relu"
2028:           )
2029:         )
2030:       ),
2031:       "post_op sum or post_op sum_relu is supported for quantized pointwise conv2d. Got binary_post_op: ",
2032:       binary_attr,
2033:       " unary_post_op: ",
2034:       unary_attr.has_value() ? unary_attr.value() : "none",
2035:       ".")
2036:     return _quantized_convolution_onednn(
2037:         act, act_scale, act_zero_point,
2038:         weight, weight_scales, weight_zero_points,
2039:         bias, stride, padding, dilation, /*transposed*/false,
2040:         groups, output_scale, output_zero_point,
2041:         accum, accum_scale, accum_zero_point,
2042:         /*output_dtype*/output_dtype, binary_attr, alpha,
2043:         unary_attr, unary_scalars, unary_algorithm
2044:     );
2045: #else
2046:     TORCH_CHECK(false, "Unimplemented as onednn is not available.")
2047: #endif
2048:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run_pointwise_binary`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run_pointwise_binary`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 2050-2096
```cpp
2050:   at::Tensor QConvoneDNN::run_pointwise_binary_tensor(
2051:       at::Tensor act, // contains quantized values but not QTensor
2052:       at::Tensor act_scale,
2053:       at::Tensor act_zero_point,
2054:       at::Tensor weight, // contains quantized values but not QTensor
2055:       at::Tensor weight_scales,
2056:       at::Tensor weight_zero_points,
2057:       at::Tensor accum, // contains quantized values but not QTensor
2058:       std::optional<at::Tensor> bias,
2059:       torch::List<int64_t> stride,
2060:       torch::List<int64_t> padding,
2061:       torch::List<int64_t> dilation,
2062:       int64_t groups,
2063:       double output_scale,
2064:       int64_t output_zero_point,
2065:       std::optional<c10::ScalarType> output_dtype,
2066:       double accum_scale,
2067:       int64_t accum_zero_point,
2068:       std::string_view binary_attr,
2069:       std::optional<at::Scalar> alpha,
2070:       std::optional<std::string_view> unary_attr,
2071:       torch::List<std::optional<at::Scalar>> unary_scalars,
2072:       std::optional<std::string_view> unary_algorithm) {
2073:
2074:     TORCH_CHECK(act_scale.numel() == 1 && act_zero_point.numel() == 1,
2075:         "onednn int8 linear: act scale/zp size should be 1");
2076:     return run_pointwise_binary(
2077:       act, act_scale.item().toDouble(), act_zero_point.item().toLong(),
2078:       weight, weight_scales, weight_zero_points, accum, bias,
2079:       stride, padding, dilation, groups,
2080:       output_scale, output_zero_point, output_dtype, accum_scale, accum_zero_point,
2081:       binary_attr, alpha, unary_attr, unary_scalars, unary_algorithm
2082:     );
2083: }
2084:
2085:
2086: namespace {
2087:
2088: /*
2089:  * FBGEMM uses vpmaddubsw instruction to multiply activations (uint8_t) and
2090:  * weights (int8_t).
2091:  *
2092:  * https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=_mm256_maddubs_epi16&expand=3284,3530&ig_expand=4236
2093:  *
2094:  * vpmaddubsw operates on a vector of activations and a vector of
2095:  * weights. If these vectors are
2096:  *
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `run_pointwise_binary_tensor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `run_pointwise_binary_tensor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 2097-2161
```cpp
2097:  *    A (uint8_t) = a0, a1, a2, a3 ...
2098:  *
2099:  * and
2100:  *
2101:  *    B (int8_t)  = b0, b1, b2, b3 ...
2102:  *
2103:  * the result of this instruction is an int16_t vector with values
2104:  *
2105:  *    C (int16_t) = a0*b0 + a1*b1, a2*b2 + a3*b3 ...
2106:  *
2107:  * For large values of A and/or B the result (a0*b0 + a1*b1) might not fit into
2108:  * an int16_t number. So the instruction saturates them to max (or min) possible
2109:  * value of an int16_t number. Such behavior is expected for the
2110:  * implementation below.
2111:  *
2112:  * For example, a0 = 255, a1 = 255, b0 = 127 and b1 = 127 the actual result
2113:  * 64770 overflows for an int16_t number (-32768, 32767) so the returned result
2114:  * is 32767.
2115:  *
2116:  */
2117: template <int kSpatialDim, bool kReluFused>
2118: class QConvInt8 final {
2119:  public:
2120:   static Tensor run(
2121:       Tensor act,
2122:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight,
2123:       double output_scale,
2124:       int64_t output_zero_point) {
2125:     if (kReluFused) {
2126:       return packed_weight->apply_relu(act, output_scale, output_zero_point);
2127:     } else {
2128:       return packed_weight->apply(act, output_scale, output_zero_point);
2129:     }
2130:   }
2131: };
2132:
2133: template <int kSpatialDim, bool kReluFused>
2134: class QConvAddInt8 final {
2135:  public:
2136:   static Tensor run(
2137:       Tensor act,
2138:       Tensor accum,
2139:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight,
2140:       double output_scale,
2141:       int64_t output_zero_point) {
2142: #if AT_MKLDNN_ENABLED() || !defined(STRIP_ERROR_MESSAGES)
2143:     auto& ctx = at::globalContext();
2144: #endif
2145: #if AT_MKLDNN_ENABLED()
2146:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
2147:       if (kReluFused) {
2148:         return dynamic_cast<PackedConvWeightsOnednn<kSpatialDim>*>(packed_weight.get())->apply_add_relu(
2149:           act, accum, output_scale, output_zero_point);
2150:       } else {
2151:         return dynamic_cast<PackedConvWeightsOnednn<kSpatialDim>*>(packed_weight.get())->apply_add(
2152:           act, accum, output_scale, output_zero_point);
2153:       }
2154:     }
2155: #endif
2156:     TORCH_CHECK(
2157:     false,
2158:     "Didn't find engine for operation quantized::conv2d_add.",
2159:     toString(ctx.qEngine()));
2160:   }
2161: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run`, `QConvInt8`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run`, `QConvInt8`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 2163-2211
```cpp
2163: template <bool kReluFused>
2164: class QConv1dInt8 final {
2165:  public:
2166:   static Tensor run(
2167:       Tensor act,
2168:       const c10::intrusive_ptr<ConvPackedParamsBase<2>>& packed_weight,
2169:       double output_scale,
2170:       int64_t output_zero_point) {
2171:     at::Tensor output;
2172:     // N, C, L -> N, C, 1, L
2173:     act = act.unsqueeze(quant_utils::kConv1dSqueezeDim + 2);
2174:     if (kReluFused) {
2175:       output = packed_weight->apply_relu(act, output_scale, output_zero_point);
2176:     } else {
2177:       output = packed_weight->apply(act, output_scale, output_zero_point);
2178:     }
2179:     // N, C, 1, L -> N, C, L
2180:     return output.squeeze_(quant_utils::kConv1dSqueezeDim + 2);
2181:   }
2182: };
2183:
2184: // kernel for maintaining backward compatibility
2185: template <int kSpatialDim, bool kReluFused>
2186: class QConvInt8ForBC final {
2187:  public:
2188:   static Tensor run(
2189:       Tensor act,
2190:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight,
2191:       torch::List<int64_t> /*stride*/,
2192:       torch::List<int64_t> /*padding*/,
2193:       torch::List<int64_t> /*dilation*/,
2194:       int64_t /*groups*/,
2195:       double output_scale,
2196:       int64_t output_zero_point) {
2197:     if (kReluFused) {
2198:       TORCH_WARN_ONCE(
2199:           "Arguments [stride, padding, dilation, groups] in ops.quantized.conv" +
2200:               std::to_string(kSpatialDim),
2201:           "d_relu, have been removed, please update your model to remove these arguments.");
2202:       return packed_weight->apply_relu(act, output_scale, output_zero_point);
2203:     } else {
2204:       TORCH_WARN_ONCE(
2205:           "Arguments [stride, padding, dilation, groups] in ops.quantized.conv",
2206:           std::to_string(kSpatialDim),
2207:           "d, have been removed, please update your model to remove these arguments.");
2208:       return packed_weight->apply(act, output_scale, output_zero_point);
2209:     }
2210:   }
2211: };
```
- EN: The main symbol in this range is `run`, `QConv1dInt8`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `run`, `QConv1dInt8`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 2213-2264
```cpp
2213: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
2214:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv1d"),          QConv1dInt8<false>::run);
2215:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv1d_relu"),     QConv1dInt8<true>::run);
2216:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d.new"),      QConvInt8<2, false>::run);
2217:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_relu.new"), QConvInt8<2, true>::run);
2218:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_add"),      QConvAddInt8<2, false>::run);
2219:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_add_relu"), QConvAddInt8<2, true>::run);
2220:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d.new"),      QConvInt8<3, false>::run);
2221:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_relu.new"), QConvInt8<3, true>::run);
2222:   // for backward compatibility
2223:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d"), QConvInt8ForBC<2, false>::run);
2224:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_relu"), QConvInt8ForBC<2, true>::run);
2225:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d"), QConvInt8ForBC<3, false>::run);
2226:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_relu"), QConvInt8ForBC<3, true>::run);
2227:
2228:   // transpose
2229:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose1d"),  QConv1dInt8<false>::run);
2230:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d"),  QConvInt8<2, false>::run);
2231:   m.impl(
2232:       TORCH_SELECTIVE_NAME("quantized::conv_transpose3d"),
2233:       QConvInt8<3, false>::run);
2234: }
2235:
2236: TORCH_LIBRARY_IMPL(_quantized, QuantizedCPU, m) {
2237:   m.impl(TORCH_SELECTIVE_NAME("_quantized::conv2d"),      QConvInt8<2, false>::run);
2238:   m.impl(TORCH_SELECTIVE_NAME("_quantized::conv2d_relu"), QConvInt8<2, true>::run);
2239:
2240:   // transpose
2241:   m.impl(TORCH_SELECTIVE_NAME("_quantized::conv_transpose1d"),  QConv1dInt8<false>::run);
2242:   m.impl(TORCH_SELECTIVE_NAME("_quantized::conv_transpose2d"),  QConvInt8<2, false>::run);
2243: }
2244:
2245: TORCH_LIBRARY_IMPL(onednn, MkldnnCPU, m) {
2246:   // Conv1D/2D/3D with unary postop
2247:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv1d_pointwise"), at::native::QConvoneDNN::run_pointwise);
2248:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise"), at::native::QConvoneDNN::run_pointwise);
2249:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.tensor"), at::native::QConvoneDNN::run_pointwise_tensor);
2250:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv3d_pointwise"), at::native::QConvoneDNN::run_pointwise);
2251:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv_pointwise"), at::native::QConvoneDNN::run_pointwise);
2252:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv_pointwise.tensor"), at::native::QConvoneDNN::run_pointwise_tensor);
2253:
2254:   // Conv2D with binary postop
2255:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.binary"), at::native::QConvoneDNN::run_pointwise_binary);
2256:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.binary_tensor"), at::native::QConvoneDNN::run_pointwise_binary_tensor);
2257: }
2258:
2259: TORCH_LIBRARY_IMPL(onednn, CPU, m) {
2260:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv_pointwise"), at::native::QConvoneDNN::run_pointwise);
2261:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv_pointwise.tensor"), at::native::QConvoneDNN::run_pointwise_tensor);
2262:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.binary"), at::native::QConvoneDNN::run_pointwise_binary);
2263:   m.impl(TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.binary_tensor"), at::native::QConvoneDNN::run_pointwise_binary_tensor);
2264: }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 2266-2267
```cpp
2266: } // namespace
2267: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Normalization statistics / 归一化统计
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Context.h`, `ATen/Parallel.h`, `ATen/TensorOperators.h`, `ATen/SmallVector.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/fbgemm_utils.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`, `ATen/native/quantized/cpu/XnnpackUtils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `algorithm`, `cmath`, `string`, `vector`, `caffe2/utils/threadpool/pthreadpool-cpp.h`, `torch/library.h`
- Key helper symbols / 关键辅助符号: `parallel_for`, `SmallVector`, `Scalar`, `ScalarType`, `qnnpack`, `fbgemm`
