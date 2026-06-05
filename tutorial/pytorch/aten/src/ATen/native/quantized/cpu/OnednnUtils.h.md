# OnednnUtils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/OnednnUtils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```cpp
 1: #pragma once
 2:
 3: #include <ATen/Config.h>
 4: #if AT_MKLDNN_ENABLED()
 5: #include <ATen/Tensor.h>
 6: #include <ATen/native/quantized/PackedParams.h>
 7: #include <ideep.hpp>
 8: #if !defined(__powerpc__)
 9: #include <cpuinfo.h>
10: #endif
11:
12: #include <c10/util/CallOnce.h>
13:
14: using PrimitiveCacheKey = std::tuple<
15:     double, // input_scale
16:     int64_t, // input_zero_point
17:     std::vector<int64_t>, // input_shape
18:     double, // output_scale
19:     int64_t, // output_zero_point
20:     int64_t, // OMP_number_of_threads
21:     double, // accum_scale
22:     int64_t>; // accum_zero_point
```
- EN: This range pulls in required headers, including `ATen/Config.h`, `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/Config.h`, `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 24-48
```cpp
24: enum CacheKeyIndex {
25:   InputScale,
26:   InputZeroPoint,
27:   InputShape,
28:   OutputScale,
29:   OutputZeroPoint,
30:   NumOfThreads,
31: };
32:
33: // Base class of primitive cache
34: struct PrimitiveCache {
35:   PrimitiveCacheKey key;
36:
37:   bool hit(const PrimitiveCacheKey& key) {
38:     return this->key == key;
39:   }
40: };
41:
42: using LinearParams = ideep::matmul_forward_params;
43: using Conv = dnnl::convolution_forward;
44: using ConvDesc = dnnl::convolution_forward::primitive_desc;
45: using ConvParams = ideep::convolution_forward_params;
46: using Deconv = dnnl::deconvolution_forward;
47: using DeconvDesc = dnnl::deconvolution_forward::primitive_desc;
48: using DeconvParams = ideep::deconv_forward_params;
```
- EN: The main symbol in this range is `hit`, `of`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `hit`, `of`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 50-71
```cpp
50: struct LinearPrimitiveCache : PrimitiveCache {
51:   LinearPrimitiveCache() = default;
52:
53:   LinearPrimitiveCache(
54:       const PrimitiveCacheKey& key,
55:       const LinearParams& param) {
56:     this->key = key;
57:     this->param = param;
58:   }
59:
60:   LinearParams param;
61:
62:   // For dynamic qlinear, scale and zero point
63:   // are set at execution time. So we only need to compare
64:   // the rest part of key.
65:   bool hit_dynamic(const PrimitiveCacheKey& new_key) {
66:     auto const& cached_input_shape = std::get<InputShape>(this->key);
67:     auto const& new_input_shape = std::get<InputShape>(new_key);
68:     return (
69:         cached_input_shape == new_input_shape &&
70:         std::get<NumOfThreads>(this->key) == std::get<NumOfThreads>(new_key));
71:   }
```
- EN: The main symbol in this range is `LinearPrimitiveCache`, `hit_dynamic`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `LinearPrimitiveCache`, `hit_dynamic`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 73-93
```cpp
73:   LinearParams& get_param() {
74:     return param;
75:   }
76: };
77:
78: struct ConvPrimitiveCache : PrimitiveCache {
79:   ConvPrimitiveCache() = default;
80:
81:   ConvPrimitiveCache(
82:       const PrimitiveCacheKey& key,
83:       const ConvParams& params) {
84:     this->key = key;
85:     this->params = params;
86:   }
87:
88:   ConvParams params;
89:
90:   ConvParams& get_params() {
91:     return params;
92:   }
93: };
```
- EN: The main symbol in this range is `get_param`, `ConvPrimitiveCache`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `get_param`, `ConvPrimitiveCache`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 95-118
```cpp
 95: struct DeconvPrimitiveCache : PrimitiveCache {
 96:   DeconvPrimitiveCache() = default;
 97:
 98:   DeconvPrimitiveCache(
 99:       const PrimitiveCacheKey& key,
100:       const DeconvParams& params) {
101:     this->key = key;
102:     this->params = params;
103:   }
104:
105:   DeconvParams params;
106:
107:   DeconvParams& get_params() {
108:     return params;
109:   }
110: };
111:
112: enum PostOps {
113:   NoPostOp,
114:   Relu,
115:   LeakyRelu,
116:   Tanh,
117:   Gelu
118: };
```
- EN: The main symbol in this range is `DeconvPrimitiveCache`, `get_params`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `DeconvPrimitiveCache`, `get_params`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 121-145
```cpp
121: struct PackedLinearWeightsOnednn : public LinearPackedParamsBase {
122:   PackedLinearWeightsOnednn(
123:       std::unique_ptr<ideep::tensor> weight,
124:       std::optional<ideep::tensor> bias,
125:       at::Tensor orig_weight,
126:       std::optional<at::Tensor> orig_bias)
127:       : weight_(std::move(weight)),
128:         bias_(std::move(bias)),
129:         orig_weight_(std::move(orig_weight)),
130:         orig_bias_(std::move(orig_bias)) {
131:     cache_initialized_flag = std::make_unique<c10::once_flag>();
132:   }
133:   std::unique_ptr<ideep::tensor> weight_;
134:   std::optional<ideep::tensor> bias_;
135:   at::Tensor orig_weight_;
136:   std::optional<at::Tensor> orig_bias_;
137:
138:   at::Tensor apply(
139:       at::Tensor input,
140:       double output_scale,
141:       int64_t output_zero_point) override;
142:   at::Tensor apply_relu(
143:       at::Tensor input,
144:       double output_scale,
145:       int64_t output_zero_point) override;
```
- EN: The main symbol in this range is `PackedLinearWeightsOnednn`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `PackedLinearWeightsOnednn`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 147-165
```cpp
147:   at::Tensor apply_dynamic(at::Tensor input, bool reduce_range=false) override;
148:   at::Tensor apply_dynamic_relu(at::Tensor input, bool reduce_range=false) override;
149:
150:   at::Tensor apply_leaky_relu(
151:       at::Tensor input,
152:       double output_scale,
153:       int64_t output_zero_point,
154:       double negative_slope);
155:
156:   at::Tensor apply_tanh(
157:       at::Tensor input,
158:       double output_scale,
159:       int64_t output_zero_point);
160:
161:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
162:
163:   std::optional<at::Tensor> bias() override {
164:     return orig_bias_;
165:   }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 167-188
```cpp
167:   static c10::intrusive_ptr<LinearPackedParamsBase> prepack(
168:       at::Tensor weight,
169:       std::optional<at::Tensor> bias);
170:
171:  private:
172:   LinearPrimitiveCache prim_cache;
173:   std::unique_ptr<c10::once_flag> cache_initialized_flag;
174:
175:   template <PostOps post_op>
176:   at::Tensor apply_impl(
177:       at::Tensor input,
178:       double output_scale,
179:       int64_t output_zero_point,
180:       torch::List<at::Scalar> post_op_args = torch::List<at::Scalar>());
181:
182:   template <bool ReluFused>
183:   at::Tensor apply_dynamic_impl(at::Tensor input, bool reduce_range=false);
184:
185:   LinearPrimitiveCache& get_cache() {
186:     return prim_cache;
187:   }
188: };
```
- EN: The main symbol in this range is `get_cache`, `apply_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `get_cache`, `apply_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 190-214
```cpp
190: template <int kSpatialDim = 2>
191: struct PackedConvWeightsOnednn : public ConvPackedParamsBase<kSpatialDim> {
192:   PackedConvWeightsOnednn(
193:       std::unique_ptr<ideep::tensor> weight,
194:       std::optional<ideep::tensor> bias,
195:       at::Tensor orig_weight,
196:       std::optional<at::Tensor> orig_bias,
197:       torch::List<int64_t> stride,
198:       torch::List<int64_t> padding,
199:       torch::List<int64_t> output_padding,
200:       torch::List<int64_t> dilation,
201:       int64_t groups,
202:       uint8_t transpose)
203:       : weight_(std::move(weight)),
204:         bias_(std::move(bias)),
205:         orig_weight_(std::move(orig_weight)),
206:         orig_bias_(std::move(orig_bias)),
207:         stride_(std::move(stride)),
208:         padding_(std::move(padding)),
209:         output_padding_(std::move(output_padding)),
210:         dilation_(std::move(dilation)),
211:         groups_(groups),
212:         transpose_(transpose) {
213:     cache_initialized_flag = std::make_unique<c10::once_flag>();
214:   }
```
- EN: The main symbol in this range is `PackedConvWeightsOnednn`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `PackedConvWeightsOnednn`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 216-235
```cpp
216:   std::unique_ptr<ideep::tensor> weight_;
217:   std::optional<ideep::tensor> bias_;
218:   at::Tensor orig_weight_;
219:   std::optional<at::Tensor> orig_bias_;
220:   torch::List<int64_t> stride_;
221:   torch::List<int64_t> padding_;
222:   torch::List<int64_t> output_padding_;
223:   torch::List<int64_t> dilation_;
224:   int64_t groups_;
225:   uint8_t transpose_;
226:
227:   at::Tensor apply(
228:       const at::Tensor& input,
229:       double output_scale,
230:       int64_t output_zero_point) override;
231:
232:   at::Tensor apply_relu(
233:       const at::Tensor& input,
234:       double output_scale,
235:       int64_t output_zero_point) override;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 237-263
```cpp
237:   at::Tensor apply_dynamic(
238:       const at::Tensor& input,
239:       bool reduce_range) override;
240:
241:   at::Tensor apply_add(
242:       const at::Tensor& input,
243:       const at::Tensor& accum,
244:       double output_scale,
245:       int64_t output_zero_point);
246:
247:   at::Tensor apply_add_relu(
248:       const at::Tensor& input,
249:       const at::Tensor& accum,
250:       double output_scale,
251:       int64_t output_zero_point);
252:
253:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
254:
255:   static c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> prepack(
256:       at::Tensor weight,
257:       std::optional<at::Tensor> bias,
258:       torch::List<int64_t> stride,
259:       torch::List<int64_t> padding,
260:       torch::List<int64_t> output_padding,
261:       torch::List<int64_t> dilation,
262:       int64_t groups,
263:       bool transpose);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 265-283
```cpp
265:   torch::List<int64_t> stride() const override {
266:     return stride_;
267:   }
268:
269:   torch::List<int64_t> padding() const override {
270:     return padding_;
271:   }
272:
273:   torch::List<int64_t> output_padding() const override {
274:     return output_padding_;
275:   }
276:
277:   torch::List<int64_t> dilation() const override {
278:     return dilation_;
279:   }
280:
281:   int64_t groups() const override {
282:     return groups_;
283:   }
```
- EN: The main symbol in this range is `stride`, `padding`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `stride`, `padding`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 285-304
```cpp
285:   bool transpose() const override {
286:     return (bool)transpose_;
287:   }
288:
289:  private:
290:   ConvPrimitiveCache conv_prim_cache;
291:   DeconvPrimitiveCache deconv_prim_cache;
292:   std::unique_ptr<c10::once_flag> cache_initialized_flag;
293:
294:   template <bool ReluFused>
295:   at::Tensor apply_impl(
296:       const at::Tensor& input,
297:       const std::optional<at::Tensor>& accum,
298:       double output_scale,
299:       int64_t output_zero_point);
300:
301:   ConvPrimitiveCache& get_conv_cache() {
302:     assert(!transpose());
303:     return conv_prim_cache;
304:   }
```
- EN: The main symbol in this range is `get_conv_cache`, `apply_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `get_conv_cache`, `apply_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 306-324
```cpp
306:   DeconvPrimitiveCache& get_deconv_cache() {
307:     assert(transpose());
308:     return deconv_prim_cache;
309:   }
310: };
311:
312: namespace onednn_utils {
313:
314: inline ideep::attr_t create_attr_by_post_op(
315:     const std::string_view& binary_post_op,
316:     double binary_alpha,
317:     double input1_scale,
318:     int64_t input1_zero_point,
319:     const ideep::tensor::desc& input1_desc,
320:     const std::string_view& unary_post_op,
321:     const torch::List<std::optional<at::Scalar>>& unary_post_op_args,
322:     const std::string_view& unary_post_op_algorithm) {
323:   using ideep::tensor;
324:   if (binary_post_op == "none") {
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `get_deconv_cache`, `create_attr_by_post_op`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `get_deconv_cache`, `create_attr_by_post_op`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 325-344
```cpp
325:     if (unary_post_op == "relu") {
326:       return ideep::attr_t::fuse_relu();
327:     } else if (unary_post_op == "leaky_relu") {
328:       TORCH_CHECK(
329:           unary_post_op_args.size() == 1,
330:           "onednn qlinear: expect one argument for post op leaky_relu but got ", unary_post_op_args.size(), " args");
331:       auto alpha = unary_post_op_args[0].value().to<float>();
332:       return ideep::attr_t::fuse_relu_v2(alpha);
333:     } else if (unary_post_op == "tanh") {
334:       return ideep::attr_t::fuse_tanh();
335:     } else if (unary_post_op == "gelu") {
336:       TORCH_CHECK(
337:           unary_post_op_algorithm == "none" || unary_post_op_algorithm == "tanh",
338:           "onednn qlinear: algorithm for post op gelu must be none or tanh but got ", unary_post_op_algorithm);
339:       auto post_algorithm = unary_post_op_algorithm == "none" ?
340:         dnnl::algorithm::eltwise_gelu_erf :
341:         dnnl::algorithm::eltwise_gelu_tanh;
342:       return ideep::attr_t::fuse_gelu_v2(0.f, 0.f, post_algorithm);
343:     } else if (unary_post_op == "hardtanh") {
344:       TORCH_CHECK(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 345-364
```cpp
345:           unary_post_op_args.size() == 2 &&
346:               unary_post_op_args[0].has_value() &&
347:               unary_post_op_args[1].has_value(),
348:           "hardtanh is expected to have two scalar input: min_val and max_val");
349:       auto lower_bound_value =
350:           unary_post_op_args[0].value().to<float>();
351:       auto upper_bound_value =
352:           unary_post_op_args[1].value().to<float>();
353:       return ideep::attr_t::fuse_clamp(lower_bound_value, upper_bound_value);
354:     } else if (unary_post_op == "hardswish") {
355:       return ideep::attr_t::fuse_hardswish();
356:     } else if (unary_post_op == "swish") {
357:       return ideep::attr_t::fuse_swish();
358:     } else {
359:       TORCH_CHECK(
360:           unary_post_op == "none",
361:           "onednn qlinear: unsupported unary post op ", unary_post_op);
362:     }
363:   } else if (binary_post_op == "sum") {
364:     if (unary_post_op == "none") {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 365-392
```cpp
365:       return ideep::attr_t::fuse_sum(input1_scale, input1_zero_point);
366:     } else if (unary_post_op == "relu") {
367:       return ideep::attr_t::residual_with_sum_zero_point(input1_scale, input1_zero_point);
368:     } else {
369:       TORCH_CHECK(
370:           false,
371:           "onednn qlinear: unsupported unary post op ", unary_post_op, " with binary post op sum");
372:     }
373:   } else if (binary_post_op == "add") {
374:     if (unary_post_op == "none") {
375:       return ideep::attr_t::fuse_binary(ideep::algorithm::binary_add, input1_desc);
376:     } else if (unary_post_op == "relu") {
377:       ideep::post_ops po;
378:       po.append_binary(ideep::algorithm::binary_add, input1_desc);
379:       po.append_eltwise(ideep::algorithm::eltwise_relu, 0, 0);
380:       return ideep::attr_t::attr_post_ops(po);
381:     } else {
382:       TORCH_CHECK(
383:           false,
384:           "onednn qlinear: unsupported unary post op ", unary_post_op, " with binary post op add");
385:     }
386:   } else {
387:     TORCH_CHECK(
388:         false,
389:         "onednn qlinear: unsupported binary post op ", binary_post_op);
390:   }
391:   return ideep::attr_t();
392: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 394-421
```cpp
394: // ONEDNN requires symmetric quantization of weight
395: // Use this util function to check.
396: inline bool is_weight_symmetric_quant(
397:       const at::Tensor& weight,
398:       bool is_transposed_conv) {
399:   bool is_symmetric = true;
400:   const auto qtype = weight.qscheme();
401:   if (qtype == c10::kPerTensorAffine) {
402:     is_symmetric &= (weight.q_zero_point() == 0);
403:   } else if (qtype == c10::kPerChannelAffine) {
404:     if (is_transposed_conv) {
405:       // This case is currently not supported in PyTorch
406:       // but we do not want to raise an error in this util function.
407:       is_symmetric = false;
408:     } else {
409:       auto output_channels = weight.size(0);
410:       for (int i = 0; i < output_channels; ++i) {
411:         auto zp = weight.q_per_channel_zero_points()[i].item<int32_t>();
412:         is_symmetric &= (zp == 0);
413:       }
414:     }
415:   } else {
416:     // This case is currently not supported in PyTorch
417:       // but we do not want to raise an error in this util function.
418:     is_symmetric = false;
419:   }
420:   return is_symmetric;
421: }
```
- EN: The main symbol in this range is `is_weight_symmetric_quant`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `is_weight_symmetric_quant`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 423-448
```cpp
423: // When qengine is x86, use this util func to check if onednn kernel
424: // is preferred than fbgemm's to get better performance.
425: inline bool should_use_onednn_quant(
426:     const at::Tensor& weight,
427:     bool is_transposed_conv,
428:     int groups,
429:     torch::List<int64_t> output_padding) {
430:   // Performance of onednn is only validated on Linux right now.
431:   // Also, the heuristics for dispatching are based on perf data on Linux.
432:   // So, for x86 qengine, we always use fbgemm kernels if OS is not Linux.
433:   // TODO Support more OSs.
434: #if !defined(__linux__)
435:   return false;
436: #else
437: #if defined(__powerpc__)
438:   constexpr auto vnni_available = true;
439: #else
440:   const auto vnni_available = cpuinfo_has_x86_avx512vnni();
441: #endif
442:   bool w_sym_quant =
443:       is_weight_symmetric_quant(weight, is_transposed_conv);
444:   bool opad_all_zero =
445:       std::all_of(output_padding.begin(), output_padding.end(), [](int i) { return i==0; });
446:   return vnni_available && (groups <= 100) && w_sym_quant && opad_all_zero;
447: #endif
448: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `should_use_onednn_quant`, `all_of`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `should_use_onednn_quant`, `all_of`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 450-468
```cpp
450: } // onednn_utils
451:
452: at::Tensor _qconv_prepack_onednn(
453:     at::Tensor weight, // from CPU backend instead of QuantizedCPU
454:     at::Tensor weight_scales, // Weight zero points must be 0 for onednn
455:     double input_scale,
456:     int64_t input_zero_point,
457:     torch::List<int64_t> stride,
458:     torch::List<int64_t> padding,
459:     torch::List<int64_t> dilation,
460:     int64_t groups,
461:     std::optional<torch::List<int64_t>> input_shape=std::nullopt);
462:
463: #define FP8E4M3_MAX 448.0
464:
465: #define CACHE_ONEDNN_CONTEXT_FLAG "ONEDNN_CACHE_CONTEXT_UNSAFE"
466: #if IDEEP_PREREQ(3, 9, 0, 0)
467: #define ONEDNN_FP8_QCONV_SUPPORTED
468: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 470-488
```cpp
470: struct QlinearForwardParams {
471:   dnnl::matmul primitive;
472:   ideep::exec_args args;
473:   ideep::tensor packed_weight;
474:   ideep::tensor weight_scales;
475:   std::optional<ideep::tensor> src_scale;
476:   std::optional<ideep::tensor> src_zero_point;
477:   std::optional<ideep::tensor> dst_scale;
478:   std::optional<ideep::tensor> dst_zero_point;
479:   std::optional<ideep::tensor> bias;
480:   ideep::tensor scratchpad;
481:
482:   void init_args() {
483:     args.insert({DNNL_ARG_WEIGHTS, packed_weight});
484:     args.insert({DNNL_ARG_SCRATCHPAD, scratchpad});
485:     if (bias.has_value()) {
486:       args.insert({DNNL_ARG_BIAS, bias.value()});
487:     }
488:     if (src_scale.has_value()) {
```
- EN: The main symbol in this range is `init_args`, `QlinearForwardParams`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `init_args`, `QlinearForwardParams`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 489-504
```cpp
489:       args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_SRC, src_scale.value()});
490:     }
491:     if (dst_scale.has_value()) {
492:       args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_DST, dst_scale.value()});
493:     }
494:     args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_WEIGHTS, weight_scales});
495:     if (src_zero_point.has_value()) {
496:       args.insert({DNNL_ARG_ATTR_ZERO_POINTS | DNNL_ARG_SRC, src_zero_point.value()});
497:     }
498:     if (dst_zero_point.has_value()) {
499:       args.insert({DNNL_ARG_ATTR_ZERO_POINTS | DNNL_ARG_DST, dst_zero_point.value()});
500:     }
501:   }
502: };
503:
504: #endif // #if AT_MKLDNN_ENABLED()
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Config.h`, `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`
- c10 headers / c10 头文件: `c10/util/CallOnce.h`
- Standard or third-party headers / 标准库或第三方头文件: `ideep.hpp`, `cpuinfo.h`
- Key helper symbols / 关键辅助符号: `Scalar`, `fbgemm`
