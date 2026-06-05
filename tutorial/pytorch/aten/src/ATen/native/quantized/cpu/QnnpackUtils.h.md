# QnnpackUtils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/QnnpackUtils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
 1: #pragma once
 2:
 3: #ifdef USE_PYTORCH_QNNPACK
 4: #include <ATen/core/Tensor.h>
 5: #include <c10/util/irange.h>
 6: #include <pytorch_qnnpack.h>
 7: #include <qnnpack_func.h>
 8: #include <ATen/native/quantized/cpu/XnnpackUtils.h>
 9: #include <ATen/native/quantized/PackedParams.h>
10: #include <ATen/native/utils/Factory.h>
11:
12: #ifndef AT_PER_OPERATOR_HEADERS
13: #include <ATen/Functions.h>
14: #else
15: #include <ATen/ops/empty.h>
16: #endif
17:
18: #include <utility>
19: inline int kPaddingChannels = 8;
20: struct QnnpackOperatorDeleter {
21:   void operator()(pytorch_qnnp_operator_t op) {
22:     pytorch_qnnp_delete_operator(op);
23:   }
24: };
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `c10/util/irange.h`, `pytorch_qnnpack.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `QnnpackOperatorDeleter`, which contributes directly to this file's operator logic.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `c10/util/irange.h`, `pytorch_qnnpack.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `QnnpackOperatorDeleter`，它们直接构成本文件的算子逻辑。

### Lines 26-44
```cpp
26: // PackedWeight struct for QNNPACK stores the original Weight and Bias as
27: // QNNPACK currently does not support an unpack function.
28: // For PyTorch Mobile, once the model is scripted and serialized we don't need
29: // to call unpack, so we can save some memory by checking for this case and free
30: // the original weights after packing.
31: // Input scale is set to null in pre-pack step. QNNPACK needs bias quantized
32: // with input scale which is available at runtime in pytorch. During runtime if
33: // input scale value changes then we requantize bias with the updated scale. For
34: // inference we expect the graph to be static so the input scale should not
35: // change across consecutive inference calls.
36: struct PackedLinearWeightsQnnp : public LinearPackedParamsBase {
37:   PackedLinearWeightsQnnp(
38:       std::unique_ptr<qnnpack::PackBMatrix> w,
39:       at::Tensor orig_weight,
40:       at::Tensor bias,
41:       std::optional<double> input_scale,
42:       at::Tensor w_scales,
43:       std::vector<uint8_t>&& w_zps)
44:       : w(std::move(w)),
```
- EN: The main symbol in this range is `PackedLinearWeightsQnnp`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `PackedLinearWeightsQnnp`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 45-65
```cpp
45:         orig_weight(std::move(orig_weight)),
46:         bias_(at::native::mobile::allocate_padded_contiguous_if_needed(
47:             bias, bias.suggest_memory_format())),
48:         per_channel_(this->orig_weight.qscheme() == at::kPerChannelAffine),
49:         input_scale(std::move(input_scale)),
50:         w_scales(std::move(w_scales)),
51:         w_zero_points(std::move(w_zps)),
52:         q_scheme(this->orig_weight.qscheme()) {
53:     weight_sizes = this->orig_weight.sizes().vec();
54:   }
55:
56:   std::unique_ptr<qnnpack::PackBMatrix> w;
57:   at::Tensor orig_weight;
58:   at::Tensor bias_;
59:   bool per_channel_;
60:   std::optional<double> input_scale;
61:   at::Tensor w_scales;
62:   std::vector<uint8_t> w_zero_points;
63:   std::vector<float> requantization_scales;
64:   std::vector<int64_t> weight_sizes;
65:   c10::QScheme q_scheme;
```
- EN: The main symbol in this range is `orig_weight`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `orig_weight`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 67-87
```cpp
67:   at::Tensor apply(
68:       at::Tensor input,
69:       double output_scale,
70:       int64_t output_zero_point) override;
71:   at::Tensor apply_relu(
72:       at::Tensor input,
73:       double output_scale,
74:       int64_t output_zero_point) override;
75:
76:   at::Tensor apply_dynamic(at::Tensor input, bool reduce_range=false) override;
77:   at::Tensor apply_dynamic_relu(at::Tensor input, bool reduce_range=false) override;
78:
79:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
80:
81:   std::optional<at::Tensor> bias() override {
82:     return bias_;
83:   }
84:
85:   static c10::intrusive_ptr<LinearPackedParamsBase> prepack(
86:       at::Tensor weight,
87:       std::optional<at::Tensor> bias);
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 89-110
```cpp
 89:   bool per_channel() const {
 90:     return per_channel_;
 91:   }
 92:
 93:  private:
 94:   std::mutex qnnp_mutex_;
 95:
 96: #ifdef USE_XNNPACK
 97:   xnnpack_operator xnnp_linear_op;
 98:
 99:   template <typename scalar_t, bool kReluFused>
100:   at::Tensor apply_impl_xnnp(
101:       const at::Tensor& input,
102:       double output_scale,
103:       int64_t output_zero_point);
104: #endif // USE_XNNPACK
105:
106:   template <bool ReluFused>
107:   at::Tensor apply_impl(
108:       at::Tensor input,
109:       double output_scale,
110:       int64_t output_zero_point);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `per_channel`, `apply_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `per_channel`, `apply_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 112-130
```cpp
112:   template <bool ReluFused>
113:   at::Tensor apply_dynamic_impl(at::Tensor input, bool reduce_range);
114: };
115:
116: template <int kSpatialDim = 2>
117: struct PackedConvWeightsQnnp : public ConvPackedParamsBase<kSpatialDim> {
118:   PackedConvWeightsQnnp(
119:       std::unique_ptr<qnnpack::PrePackConvWeights> w,
120:       at::Tensor orig_weight,
121:       at::Tensor bias,
122:       torch::List<int64_t> stride,
123:       torch::List<int64_t> padding,
124:       torch::List<int64_t> output_padding,
125:       torch::List<int64_t> dilation,
126:       int64_t groups,
127:       bool transpose,
128:       std::optional<double> input_scale,
129:       std::vector<int64_t> kernel,
130:       at::Tensor w_scale,
```
- EN: The main symbol in this range is `apply_dynamic_impl`, `PackedConvWeightsQnnp`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `apply_dynamic_impl`, `PackedConvWeightsQnnp`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 131-150
```cpp
131:       std::vector<uint8_t>&& w_zps,
132:       bool is_per_channel)
133:       : w(std::move(w)),
134:         orig_weight(std::move(orig_weight)),
135:         bias(std::move(bias)),
136:         stride_(std::move(stride)),
137:         padding_(std::move(padding)),
138:         output_padding_(std::move(output_padding)),
139:         dilation_(std::move(dilation)),
140:         groups_(groups),
141:         transpose_(transpose),
142:         is_per_channel_(is_per_channel),
143:         input_scale(input_scale),
144:         kernel_(std::move(kernel)),
145:         w_scales(std::move(w_scale)),
146:         w_zero_points(std::move(w_zps)) {
147:     const bool any_padding = std::any_of(
148:         padding_.begin(), padding_.end(), [](const auto& e) { return e != 0; });
149:     const size_t kernel_size =
150:         std::accumulate(kernel_.begin(), kernel_.end(), 1, std::multiplies<>());
```
- EN: The main symbol in this range is `w`, `any_of`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `w`, `any_of`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 152-176
```cpp
152:     const size_t group_input_channels = transpose
153:         ? this->orig_weight.size(0) / groups
154:         : this->orig_weight.size(1);
155:     const size_t group_output_channels = transpose
156:         ? this->orig_weight.size(1)
157:         : this->orig_weight.size(0) / groups;
158:
159:     const size_t kernel_depth = kSpatialDim == 3 ? kernel_[0] : 1;
160:     const size_t kernel_height = kernel_[kSpatialDim - 2];
161:     const size_t kernel_width = kernel_[kSpatialDim - 1];
162:
163:     pytorch_qnnp_ukernel_type ukernel_type;
164:     if (transpose_) {
165:       ukernel_type = pytorch_qnnp_ukernel_type_conv;
166:     } else {
167:       ukernel_type = pytorch_qnnp_ukernel_type_none;
168:
169:       const bool has_depthwise_dimensions =
170:           (kSpatialDim == 2 &&
171:            ((kernel_height == 3 && kernel_width == 3) ||
172:             (kernel_height == 5 && kernel_width == 5))) ||
173:           (kSpatialDim == 3 && kernel_height == 3 && kernel_width == 3 &&
174:            kernel_depth == 3);
175:       const bool has_depthwise_grouping =
176:           group_input_channels == 1 && group_output_channels == 1 && groups > 1;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 178-198
```cpp
178:       if (has_depthwise_dimensions && has_depthwise_grouping) {
179:         ukernel_type = pytorch_qnnp_ukernel_type_dwconv;
180:       } else if (
181:           kernel_size == 1 &&
182:           std::all_of(
183:               stride_.begin(),
184:               stride_.end(),
185:               [](const auto& e) { return e == 1; }) &&
186:           !any_padding) {
187:         ukernel_type = group_input_channels >= SIZE_MAX
188:             ? pytorch_qnnp_ukernel_type_xzp_gemm
189:             : pytorch_qnnp_ukernel_type_gemm;
190:       } else {
191:         ukernel_type = pytorch_qnnp_ukernel_type_conv;
192:       }
193:     }
194:
195:     if (is_per_channel && ukernel_type == pytorch_qnnp_ukernel_type_xzp_gemm) {
196:       TORCH_INTERNAL_ASSERT(
197:           false, "Per channel quantized weights are not supported for XZP kernels");
198:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 200-218
```cpp
200:     pytorch_qnnp_operator_t convolution{nullptr};
201:     // Initially all the params are set to zero.
202:     convolution = static_cast<pytorch_qnnp_operator_t>(
203:         calloc(1, sizeof(struct pytorch_qnnp_operator)));
204:     if (convolution == nullptr) {
205:       TORCH_INTERNAL_ASSERT(
206:           false, "failed to allocate %zu bytes for pytorch_qnnp_operator structure",
207:           sizeof(struct pytorch_qnnp_operator));
208:     }
209:
210:     convolution_op =
211:         std::unique_ptr<pytorch_qnnp_operator, QnnpackOperatorDeleter>(
212:             convolution);
213:
214:     // NOLINTNEXTLINE(clang-analyzer-core.NullDereference)
215:     convolution->ukernel_type = ukernel_type;
216:     convolution->groups = groups;
217:     convolution->group_input_channels = group_input_channels;
218:     convolution->group_output_channels = group_output_channels;
```
- EN: The main symbol in this range is `pytorch_qnnp_operator`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `pytorch_qnnp_operator`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 219-238
```cpp
219:     convolution->kernel_depth = kernel_depth;
220:     convolution->kernel_height = kernel_height;
221:     convolution->kernel_width = kernel_width;
222:     convolution->stride_depth = kSpatialDim == 3 ? stride_[0] : 1;
223:     convolution->stride_height = stride_[kSpatialDim - 2];
224:     convolution->stride_width = stride_[kSpatialDim - 1];
225:     convolution->dilation_depth = kSpatialDim == 3 ? dilation_[0] : 1;
226:     convolution->dilation_height = dilation_[kSpatialDim - 2];
227:     convolution->dilation_width = dilation_[kSpatialDim - 1];
228:     convolution->input_padding_height = padding_[kSpatialDim - 2];
229:     convolution->input_padding_width = padding_[kSpatialDim - 1];
230:     convolution->input_padding_depth = kSpatialDim == 3 ? padding_[0] : 0;
231:     convolution->per_channel = is_per_channel_;
232:     convolution->transpose = transpose_;
233:
234:     const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
235:     const size_t k_stride = (group_input_channels + (kr - 1)) & -kr;
236:
237:     size_t zero_size = sizeof(uint8_t) * k_stride;
238:     size_t zero_offset = 0;
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 240-258
```cpp
240:     if (transpose_) {
241:       convolution->adjustment_width = output_padding_[1];
242:       convolution->adjustment_height = output_padding_[0];
243:       if (group_input_channels < 8) {
244:         zero_size += 8;
245:         zero_offset = 8;
246:       }
247:     } else {
248:       zero_buffer_size = 0;
249:       if (any_padding) {
250:         zero_size = 0;
251:         zero_offset = 0;
252:         if (ukernel_type == pytorch_qnnp_ukernel_type_dwconv) {
253:           const uint32_t cr = pytorch_qnnp_params.q8dw9.cr;
254:           const size_t group_stride = (groups + (cr - 1)) & -cr;
255:           if (groups >= 8) {
256:             zero_size = sizeof(uint8_t) * group_stride;
257:             zero_offset = 0;
258:           } else {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 259-278
```cpp
259:             zero_size = sizeof(uint8_t) * group_stride + 8;
260:             zero_offset = sizeof(uint8_t) * 8;
261:           }
262:         } else if (
263:             ukernel_type == pytorch_qnnp_ukernel_type_conv ||
264:             ukernel_type == pytorch_qnnp_ukernel_type_gemm) {
265:           if (group_input_channels >= 8) {
266:             zero_size = sizeof(uint8_t) * k_stride;
267:             zero_offset = 0;
268:           } else {
269:             zero_size = sizeof(uint8_t) * k_stride + 8;
270:             zero_offset = 8;
271:           }
272:         }
273:       }
274:     }
275:
276:     // NOLINTNEXTLINE(clang-analyzer-optin.portability.UnixAPI)
277:     void* zero_buffer = malloc(zero_size);
278:     if (zero_buffer == nullptr) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 279-298
```cpp
279:       pytorch_qnnp_delete_operator(convolution);
280:       TORCH_INTERNAL_ASSERT(
281:           false, "failed to allocate %zu bytes for zero padding",
282:           zero_size);
283:     }
284:     // Need to set to input zero point
285:     // memset(zero_buffer, input_zero_point, zero_size);
286:     zero_buffer_size = zero_size;
287:     convolution->zero_buffer = zero_buffer;
288:     convolution->zero_pointer = (void*)((uintptr_t)zero_buffer + zero_offset);
289:   }
290:
291:   std::unique_ptr<pytorch_qnnp_operator, QnnpackOperatorDeleter> convolution_op;
292:   #ifdef USE_XNNPACK
293:   xnnpack_operator xnnp_convolution_op;
294:   #endif  // USE_XNNPACK
295:   std::unique_ptr<qnnpack::PrePackConvWeights> w;
296:   at::Tensor orig_weight;
297:   at::Tensor bias;
298:   torch::List<int64_t> stride_;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 299-320
```cpp
299:   torch::List<int64_t> padding_;
300:   torch::List<int64_t> output_padding_;
301:   torch::List<int64_t> dilation_;
302:   int64_t groups_;
303:   bool transpose_;
304:   bool is_per_channel_;
305:   std::optional<double> input_scale;
306:   std::vector<int64_t> kernel_;
307:   at::Tensor w_scales;
308:   std::vector<uint8_t> w_zero_points;
309:   std::vector<float> requantization_scales;
310:   size_t zero_buffer_size;
311:
312:   at::Tensor apply(
313:       const at::Tensor& input,
314:       double output_scale,
315:       int64_t output_zero_point) override;
316:
317:   at::Tensor apply_relu(
318:       const at::Tensor& input,
319:       double output_scale,
320:       int64_t output_zero_point) override;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 322-340
```cpp
322:   at::Tensor apply_dynamic(
323:       const at::Tensor& input,
324:       bool reduce_range=false) override;
325:
326:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
327:
328:   static c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> prepack(
329:       at::Tensor weight,
330:       std::optional<at::Tensor> bias,
331:       torch::List<int64_t> stride,
332:       torch::List<int64_t> padding,
333:       torch::List<int64_t> output_padding,
334:       torch::List<int64_t> dilation,
335:       int64_t groups,
336:       bool transpose);
337:
338:   torch::List<int64_t> stride() const override {
339:     return stride_;
340:   }
```
- EN: The main symbol in this range is `stride`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `stride`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 342-360
```cpp
342:   torch::List<int64_t> padding() const override {
343:     return padding_;
344:   }
345:
346:   torch::List<int64_t> output_padding() const override {
347:     return output_padding_;
348:   }
349:
350:   torch::List<int64_t> dilation() const override {
351:     return dilation_;
352:   }
353:
354:   int64_t groups() const override {
355:     return groups_;
356:   }
357:
358:   bool transpose() const override {
359:     return transpose_;
360:   }
```
- EN: The main symbol in this range is `padding`, `output_padding`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `padding`, `output_padding`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 362-381
```cpp
362:   bool per_channel() const {
363:     return is_per_channel_;
364:   }
365:
366:  private:
367:   std::mutex qnnp_mutex_;
368:   template <bool ReluFused>
369:   at::Tensor apply_impl(
370:       const at::Tensor& input,
371:       double output_scale,
372:       int64_t output_zero_point);
373:
374: #ifdef USE_XNNPACK
375:   template <typename scalar_t, bool ReluFused>
376:   at::Tensor apply_impl_xnnp(
377:       const at::Tensor& input,
378:       double output_scale,
379:       int64_t output_zero_point);
380: #endif // USE_XNNPACK
381: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `per_channel`, `apply_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `per_channel`, `apply_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 383-401
```cpp
383: enum class Activation : uint8_t { NONE = 0, RELU = 1 };
384:
385: template<typename T>
386: inline T QuantizeValue(float scale, int32_t zero_point, float value) {
387:   const int32_t qmin = std::numeric_limits<T>::min();
388:   const int32_t qmax = std::numeric_limits<T>::max();
389:   auto r = zero_point + static_cast<int32_t>(std::nearbyint(value / scale));
390:   r = std::max(r, qmin);
391:   r = std::min(r, qmax);
392:   return static_cast<T>(r);
393: }
394:
395: template<typename T>
396: inline std::pair<T, T> activationLimits(
397:     float scale,
398:     int32_t zero_point,
399:     Activation Ac) {
400:   switch (Ac) {
401:     case Activation::NONE:
```
- EN: The main symbol in this range is `QuantizeValue`, `activationLimits`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `QuantizeValue`, `activationLimits`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 402-425
```cpp
402:       return {std::numeric_limits<T>::min(),
403:               std::numeric_limits<T>::max()};
404:     case Activation::RELU:
405:       return {QuantizeValue<T>(scale, zero_point, 0.0),
406:               std::numeric_limits<T>::max()};
407:     default:
408: #ifdef _MSC_VER
409:       __assume(0);
410: #else
411:       __builtin_unreachable();
412: #endif
413:   }
414: }
415:
416: namespace at::native::qnnp_avgpool_helper {
417: Tensor qnnpack_avg_pool2d(
418:     Tensor input,
419:     IntArrayRef kernel_size,
420:     IntArrayRef stride,
421:     IntArrayRef padding,
422:     bool ceil_mode,
423:     bool count_include_pad,
424:     std::optional<int64_t> divisor_override);
425: } // namespace at::native::qnnp_avgpool_helper
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 427-450
```cpp
427: namespace {
428: [[maybe_unused]] std::vector<float> generate_requantization_scales(
429:     const at::Tensor& weight_scales,
430:     const float input_scale,
431:     const float output_scale,
432:     std::vector<float>& requant_scales) {
433:   // Since weight scale is allocated with padding
434:   // weight_scales.numel() gives us padded num elements.
435:   const auto num_output_channels_padded = weight_scales.numel();
436:   float *const weight_scales_data = weight_scales.data_ptr<float>();
437:   if (static_cast<int64_t>(requant_scales.size()) < num_output_channels_padded) {
438:     requant_scales.resize(num_output_channels_padded);
439:   }
440:   for (const auto i : c10::irange(num_output_channels_padded)) {
441:     const auto inverse_output_scale = 1.f /output_scale;
442:     requant_scales[i] = (weight_scales_data[i] * input_scale) * inverse_output_scale;
443:     TORCH_CHECK(
444:         (requant_scales[i] > 0.0f && std::isnormal(requant_scales[i])),
445:         "failed to create op with requantization scale: ",
446:         requant_scales[i],
447:         ": requantization scale must be finite and positive");
448:   }
449:   return requant_scales;
450: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `generate_requantization_scales`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `generate_requantization_scales`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 452-470
```cpp
452: [[maybe_unused]] std::pair<std::vector<uint8_t>, at::Tensor>
453: make_zero_points_and_scales_tensor(
454:     const at::Tensor& weight_contig,
455:     bool transpose = false,
456:     uint32_t groups = 1) {
457:   const int out_ch_idx = transpose ? 1 : 0;
458:   const auto num_output_channels = weight_contig.size(out_ch_idx) * (transpose ? groups : 1);
459:   // Add 8 to account for buffering needed by QNNPACK.
460:   const auto num_output_channels_padded = num_output_channels + kPaddingChannels;
461:   const auto qtype = weight_contig.qscheme();
462:   std::vector<uint8_t> weight_zp(num_output_channels_padded, 0);
463:   // Adjust weight zero point, similar to weight data.
464:   if (qtype == at::kPerTensorAffine) {
465:     for (const auto i : c10::irange(num_output_channels)) {
466:       weight_zp[i] = (uint8_t)(weight_contig.q_zero_point() + 128);
467:     }
468:   } else if (qtype == at::kPerChannelAffine) {
469:     TORCH_CHECK(
470:         weight_contig.q_per_channel_zero_points().scalar_type() == at::kLong,
```
- EN: The main symbol in this range is `make_zero_points_and_scales_tensor`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `make_zero_points_and_scales_tensor`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 471-490
```cpp
471:         "Per channel zero points dtype must be long int.");
472:     const int64_t* per_channel_zero_points =
473:       weight_contig.q_per_channel_zero_points().data_ptr<int64_t>();
474:     for (const auto i : c10::irange(num_output_channels)) {
475:       weight_zp[i] = (uint8_t)(per_channel_zero_points[i] + 128);
476:     }
477:   } else {
478:     TORCH_INTERNAL_ASSERT(false, "Unsupported quantization scheme.");
479:   }
480:   at:: Tensor weight_scales =
481:     at::empty(
482:         {num_output_channels_padded},
483:         at::device(at::kCPU).dtype(at::kFloat));
484:   float *const weight_scales_data = weight_scales.data_ptr<float>();
485:   if (qtype == at::kPerTensorAffine) {
486:     for (const auto i : c10::irange(num_output_channels)) {
487:       weight_scales_data[i] = weight_contig.q_scale();
488:     }
489:   } else if (qtype == at::kPerChannelAffine) {
490:     TORCH_CHECK(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 491-508
```cpp
491:         weight_contig.q_per_channel_scales().scalar_type() == at::kDouble,
492:         "Per channel scales dtype must be double.");
493:     const double *const per_channel_scales =
494:       weight_contig.q_per_channel_scales().data_ptr<double>();
495:     for (const auto i : c10::irange(num_output_channels)) {
496:       weight_scales_data[i] = static_cast<float>(per_channel_scales[i]);
497:     }
498:   } else {
499:     TORCH_INTERNAL_ASSERT(false, "Unsupported quantization scheme.");
500:   }
501:   for (const auto i : c10::irange(num_output_channels, num_output_channels_padded)) {
502:     weight_scales_data[i] = 1.f;
503:   }
504:   return {weight_zp, weight_scales};
505: }
506: } // namespace
507:
508: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Normalization statistics / 归一化统计
- Low-level memory access / 底层内存访问
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/native/quantized/cpu/XnnpackUtils.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/utils/Factory.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `pytorch_qnnpack.h`, `qnnpack_func.h`, `utility`
- Key helper symbols / 关键辅助符号: `qnnpack`
