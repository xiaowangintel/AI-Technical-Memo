# fbgemm_utils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/fbgemm_utils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: #pragma once
 2:
 3: #include <ATen/Tensor.h>
 4: #include <ATen/native/quantized/PackedParams.h>
 5: #include <ATen/native/quantized/cpu/EmbeddingPackedParams.h>
 6: #include <c10/core/QScheme.h>
 7: #include <c10/util/irange.h>
 8:
 9: #ifdef USE_FBGEMM
10: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wextra-semi")
11: #include <fbgemm/Fbgemm.h>
12: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Winconsistent-missing-destructor-override")
13: #include <fbgemm/FbgemmFP16.h>
14: C10_DIAGNOSTIC_POP()
15: #include <fbgemm/QuantUtils.h>
16: C10_DIAGNOSTIC_POP()
17:
18: // The struct for the packed weight matrix (PackBMatrix) and the corresponding
19: // column offsets used for the fully connect layer, which are both prepared in
20: // the prepacking step to save the computations in the inference. Note the
```
- EN: This range pulls in required headers, including `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 21-46
```cpp
21: // column offsets include the sum of the B columns as well as the scalar term
22: // B_zero_point * K, whereas the row offsets created by
23: // PackAWithQuantRowOffset/PackAWithIm2Col/PackAWithRowOffset are only the sum
24: // of the A rows. The column offsets are needed for the asymmetric quantization
25: // (affine quantization) of input matrix.
26: // Note that in JIT mode we can think of a way to fuse col_offsets with bias.
27: struct TORCH_API PackedLinearWeight : public LinearPackedParamsBase {
28:   PackedLinearWeight(
29:       std::unique_ptr<fbgemm::PackBMatrix<int8_t>> w,
30:       std::optional<at::Tensor> bias,
31:       std::vector<int32_t> col_offsets,
32:       std::vector<float> w_scale,
33:       std::vector<int32_t> w_zp,
34:       c10::QScheme q_scheme)
35:       : w(std::move(w)),
36:         bias_(std::move(bias)),
37:         col_offsets(std::move(col_offsets)),
38:         w_scale(std::move(w_scale)),
39:         w_zp(std::move(w_zp)),
40:         q_scheme(std::move(q_scheme)) {}
41:   std::unique_ptr<fbgemm::PackBMatrix<int8_t>> w;
42:   std::optional<at::Tensor> bias_;
43:   std::vector<int32_t> col_offsets;
44:   std::vector<float> w_scale;
45:   std::vector<int32_t> w_zp;
46:   c10::QScheme q_scheme;
```
- EN: The main symbol in this range is `PackedLinearWeight`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `PackedLinearWeight`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 48-68
```cpp
48:   at::Tensor apply(
49:       at::Tensor input,
50:       double output_scale,
51:       int64_t output_zero_point) override;
52:
53:   at::Tensor apply_relu(
54:       at::Tensor input,
55:       double output_scale,
56:       int64_t output_zero_point) override;
57:
58:   at::Tensor& apply_out(
59:       const at::Tensor& input,
60:       double output_scale,
61:       int64_t output_zero_point,
62:       at::Tensor& output) override;
63:
64:   at::Tensor& apply_relu_out(
65:       const at::Tensor& input,
66:       double output_scale,
67:       int64_t output_zero_point,
68:       at::Tensor& output) override;
```
- EN: The main symbol in this range is `apply_out`, `apply_relu_out`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `apply_out`, `apply_relu_out`，它们直接构成本文件的算子逻辑。

### Lines 70-90
```cpp
70:   at::Tensor apply_with_input_q_dq_qweight_dq_output_fp32(
71:       at::Tensor input,
72:       double input_scale,
73:       int64_t input_zero_point) override;
74:
75:   at::Tensor apply_with_input_q_dq_qweight_dq_relu_output_fp32(
76:       at::Tensor input,
77:       double input_scale,
78:       int64_t input_zero_point) override;
79:
80:   at::Tensor apply_dynamic(at::Tensor input, bool reduce_range = false)
81:       override;
82:
83:   at::Tensor apply_dynamic_relu(at::Tensor input, bool reduce_range = false)
84:       override;
85:
86:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
87:
88:   std::optional<at::Tensor> bias() override {
89:     return bias_;
90:   }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 92-112
```cpp
 92:   static c10::intrusive_ptr<LinearPackedParamsBase> prepack(
 93:       at::Tensor weight,
 94:       std::optional<at::Tensor> bias);
 95:
 96:  private:
 97:   template <bool ReluFused>
 98:   at::Tensor& apply_impl(
 99:       const at::Tensor& input,
100:       double output_scale,
101:       int64_t output_zero_point,
102:       at::Tensor& output);
103:
104:   template <bool ReluFused>
105:   at::Tensor apply_with_input_q_dq_qweight_dq_output_fp32_impl(
106:       const at::Tensor& input,
107:       double input_scale,
108:       int64_t input_zero_point);
109:
110:   template <bool ReluFused>
111:   at::Tensor apply_dynamic_impl(at::Tensor input, bool reduce_range = false);
112: };
```
- EN: The main symbol in this range is `apply_impl`, `apply_with_input_q_dq_qweight_dq_output_fp32_impl`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_impl`, `apply_with_input_q_dq_qweight_dq_output_fp32_impl`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 114-134
```cpp
114: struct TORCH_API PackedLinearWeightFp16 : public LinearPackedParamsBase {
115:   PackedLinearWeightFp16(
116:       std::unique_ptr<fbgemm::PackedGemmMatrixFP16> w,
117:       std::optional<at::Tensor> bias)
118:       : w(std::move(w)), bias_(std::move(bias)) {}
119:
120:   std::unique_ptr<fbgemm::PackedGemmMatrixFP16> w;
121:   std::optional<at::Tensor> bias_;
122:
123:   at::Tensor apply(
124:       at::Tensor /*input*/,
125:       double /*output_scale*/,
126:       int64_t /*output_zero_point*/) override {
127:     TORCH_INTERNAL_ASSERT(false);
128:   }
129:   at::Tensor apply_relu(
130:       at::Tensor /*input*/,
131:       double /*output_scale*/,
132:       int64_t /*output_zero_point*/) override {
133:     TORCH_INTERNAL_ASSERT(false);
134:   }
```
- EN: The main symbol in this range is `PackedLinearWeightFp16`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `PackedLinearWeightFp16`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 136-154
```cpp
136:   at::Tensor apply_dynamic(at::Tensor input, bool reduce_range = false)
137:       override;
138:   at::Tensor apply_dynamic_relu(at::Tensor input, bool reduce_range = false)
139:       override;
140:
141:   at::Tensor& apply_dynamic_out(
142:       const at::Tensor& input,
143:       at::Tensor& output,
144:       bool reduce_range = false) override;
145:   at::Tensor& apply_dynamic_relu_out(
146:       const at::Tensor& input,
147:       at::Tensor& output,
148:       bool reduce_range = false) override;
149:
150:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
151:
152:   std::optional<at::Tensor> bias() override {
153:     return bias_;
154:   }
```
- EN: The main symbol in this range is `apply_dynamic_out`, `apply_dynamic_relu_out`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `apply_dynamic_out`, `apply_dynamic_relu_out`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 156-174
```cpp
156:   static c10::intrusive_ptr<LinearPackedParamsBase> prepack(
157:       at::Tensor weight,
158:       std::optional<at::Tensor> bias);
159:
160:   void set_bias(std::optional<at::Tensor> bias) override;
161:
162:  private:
163:   template <bool ReluFused>
164:   at::Tensor& apply_dynamic_impl(const at::Tensor& input, at::Tensor& output);
165: };
166:
167: template <int kSpatialDim = 2>
168: struct TORCH_API PackedConvWeight : public ConvPackedParamsBase<kSpatialDim> {
169:   PackedConvWeight(
170:       std::unique_ptr<fbgemm::PackWeightsForConv<kSpatialDim>> w,
171:       std::optional<at::Tensor> bias,
172:       torch::List<int64_t> stride,
173:       torch::List<int64_t> padding,
174:       torch::List<int64_t> output_padding,
```
- EN: The main symbol in this range is `apply_dynamic_impl`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `apply_dynamic_impl`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 175-195
```cpp
175:       torch::List<int64_t> dilation,
176:       int64_t groups,
177:       uint8_t transpose,
178:       std::vector<int32_t> col_offsets,
179:       std::vector<int64_t> kernel,
180:       std::vector<float> w_scale,
181:       std::vector<int32_t> w_zp,
182:       c10::QScheme q_scheme)
183:       : w(std::move(w)),
184:         bias(std::move(bias)),
185:         stride_(std::move(stride)),
186:         padding_(std::move(padding)),
187:         output_padding_(std::move(output_padding)),
188:         dilation_(std::move(dilation)),
189:         groups_(groups),
190:         transpose_(transpose),
191:         col_offsets(std::move(col_offsets)),
192:         kernel(std::move(kernel)),
193:         w_scale(std::move(w_scale)),
194:         w_zp(std::move(w_zp)),
195:         q_scheme(q_scheme) {}
```
- EN: The main symbol in this range is `w`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `w`，它们直接构成本文件的算子逻辑。

### Lines 197-219
```cpp
197:   std::unique_ptr<fbgemm::PackWeightsForConv<kSpatialDim>> w;
198:   std::optional<at::Tensor> bias;
199:   torch::List<int64_t> stride_;
200:   torch::List<int64_t> padding_;
201:   torch::List<int64_t> output_padding_;
202:   torch::List<int64_t> dilation_;
203:   int64_t groups_;
204:   uint8_t transpose_;
205:   std::vector<int32_t> col_offsets;
206:   std::vector<int64_t> kernel;
207:   std::vector<float> w_scale;
208:   std::vector<int32_t> w_zp;
209:   c10::QScheme q_scheme;
210:
211:   at::Tensor apply(
212:       const at::Tensor& input,
213:       double output_scale,
214:       int64_t output_zero_point) override;
215:
216:   at::Tensor apply_relu(
217:       const at::Tensor& input,
218:       double output_scale,
219:       int64_t output_zero_point) override;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 221-243
```cpp
221:   at::Tensor apply_dynamic(
222:     const at::Tensor& input,
223:     bool reduce_range) override;
224:
225:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
226:
227:   static c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> prepack(
228:       at::Tensor weight,
229:       std::optional<at::Tensor> bias,
230:       torch::List<int64_t> stride,
231:       torch::List<int64_t> padding,
232:       torch::List<int64_t> output_padding,
233:       torch::List<int64_t> dilation,
234:       int64_t groups,
235:       bool transpose);
236:
237:   const float* GetBiasData(at::Tensor* bias);
238:
239:   void GetQuantizationParams(
240:       float act_scale,
241:       float out_scale,
242:       std::vector<float>* output_multiplier_float,
243:       std::vector<float>* act_times_w_scale);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 245-263
```cpp
245:   torch::List<int64_t> stride() const override {
246:     return stride_;
247:   }
248:
249:   torch::List<int64_t> padding() const override {
250:     return padding_;
251:   }
252:
253:   torch::List<int64_t> output_padding() const override {
254:     return output_padding_;
255:   }
256:
257:   torch::List<int64_t> dilation() const override {
258:     return dilation_;
259:   }
260:
261:   int64_t groups() const override {
262:     return groups_;
263:   }
```
- EN: The main symbol in this range is `stride`, `padding`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `stride`, `padding`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 265-285
```cpp
265:   bool transpose() const override {
266:     return (bool)transpose_;
267:   }
268:
269:  private:
270:   template <bool ReluFused>
271:   at::Tensor apply_impl(
272:       const at::Tensor& input,
273:       double output_scale,
274:       int64_t output_zero_point);
275: };
276:
277: // PackWeight: Convert the weight from uint8 to int8.
278: inline void convert_uint8_int8(
279:     int len,
280:     const uint8_t* src_uint8,
281:     int8_t* dst_int8) {
282:   for (const auto i : c10::irange(len)) {
283:     dst_int8[i] = static_cast<int8_t>(static_cast<int32_t>(src_uint8[i]) - 128);
284:   }
285: }
```
- EN: The main symbol in this range is `convert_uint8_int8`, `apply_impl`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `convert_uint8_int8`, `apply_impl`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 287-312
```cpp
287: // UnpackWeight: Convert the weight from int8 to uint8.
288: inline void convert_int8_uint8(
289:     int len,
290:     const int8_t* src_int8,
291:     uint8_t* dst_uint8) {
292:   for (const auto i : c10::irange(len)) {
293:     dst_uint8[i] =
294:         static_cast<uint8_t>(static_cast<int32_t>(src_int8[i]) + 128);
295:   }
296: }
297:
298: namespace at::native::fbgemm_utils {
299:
300: template <int kSpatialDim = 2>
301: fbgemm::conv_param_t<kSpatialDim> MakeFbgemmConvParam(
302:     int N,
303:     int C,
304:     int M,
305:     const std::vector<int>& image_shape,
306:     int groups,
307:     const std::vector<int>& kernels,
308:     const std::vector<int>& strides,
309:     const std::vector<int>& pads,
310:     const std::vector<int>& dilations,
311:     const std::vector<int>& output_padding = std::vector<int>(kSpatialDim, 0),
312:     bool transposed = false);
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `convert_int8_uint8`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `convert_int8_uint8`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 314-339
```cpp
314: // TODO: Remove functions below when ChannelsLast3d is ready.
315: Tensor MakeStridedQTensorCPU(
316:     const IntArrayRef& sizes,
317:     const IntArrayRef& strides,
318:     const TensorOptions& options,
319:     QuantizerPtr quantizer);
320:
321: Tensor MakeEmptyAffineQuantizedChannelsLast3dTensor(
322:     int64_t N,
323:     int64_t C,
324:     int64_t D,
325:     int64_t H,
326:     int64_t W,
327:     const TensorOptions& options,
328:     double scale,
329:     int64_t zero_point);
330:
331: Tensor MakeEmptyPerChannelAffineQuantizedChannelsLast3dTensor(
332:     int64_t N,
333:     int64_t C,
334:     int64_t D,
335:     int64_t H,
336:     int64_t W,
337:     const TensorOptions& options,
338:     const Tensor& scales,
339:     const Tensor& zero_points);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 341-359
```cpp
341: Tensor ConvertToChannelsLast3dTensor(const Tensor& src);
342:
343: template <int kSpatialDim = 2>
344: Tensor TransposeConvTensorUnpackConversion(const Tensor& src, int groups);
345:
346: template <int kSpatialDim>
347: Tensor ConvertConvWeightsToChannelLastTensor(
348:     const at::Tensor& src,
349:     int groups,
350:     bool transpose);
351: } // at::native::namespace fbgemm_utils
352:
353: #endif // USE_FBGEMM
354:
355: struct TORCH_API PackedEmbeddingBagWeight : public EmbeddingPackedParamsBase {
356:   PackedEmbeddingBagWeight(
357:       at::Tensor packed_w,
358:       std::vector<float> w_scale,
359:       std::vector<float> w_zp,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 360-379
```cpp
360:       int64_t bit_rate,
361:       c10::QScheme q_scheme,
362:       int64_t version)
363:       : packed_w(std::move(packed_w)),
364:         w_scale(std::move(w_scale)),
365:         w_zp(std::move(w_zp)),
366:         bit_rate_(bit_rate),
367:         q_scheme(q_scheme),
368:         version_(version) {
369:     if (!this->packed_w.is_contiguous()) {
370:       this->packed_w = this->packed_w.contiguous();
371:     }
372:   }
373:
374:   at::Tensor packed_w;
375:   std::vector<float> w_scale;
376:   std::vector<float> w_zp;
377:   int64_t bit_rate_;
378:   c10::QScheme q_scheme;
379:   int64_t version_;
```
- EN: The main symbol in this range is `packed_w`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `packed_w`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 381-400
```cpp
381:   at::Tensor unpack() override;
382:   static c10::intrusive_ptr<EmbeddingPackedParamsBase> prepack(
383:       const at::Tensor& weight);
384:
385:   int64_t bit_rate() const override {
386:     return bit_rate_;
387:   }
388:
389:   int64_t version() const override {
390:     return version_;
391:   }
392:
393:   at::Tensor embeddingbag_byte(
394:       const at::Tensor& indices,
395:       const std::optional<at::Tensor>& offsets,
396:       bool pruned_weights,
397:       const std::optional<at::Tensor>& per_sample_weights_,
398:       const std::optional<at::Tensor>& compressed_indices_mapping,
399:       bool include_last_offset,
400:       bool is_embedding_op) override;
```
- EN: The main symbol in this range is `bit_rate`, `version`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `bit_rate`, `version`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 402-410
```cpp
402:   at::Tensor embeddingbag_4bit(
403:       const at::Tensor& indices,
404:       const std::optional<at::Tensor>& offsets,
405:       bool pruned_weights,
406:       const std::optional<at::Tensor>& per_sample_weights_,
407:       const std::optional<at::Tensor>& compressed_indices_mapping,
408:       bool include_last_offset,
409:       bool is_embedding_op) override;
410: };
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Shape/container bookkeeping / 形状与容器管理
- Namespace-scoped helper structure / 命名空间内辅助结构

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`
- c10 headers / c10 头文件: `c10/core/QScheme.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `fbgemm/Fbgemm.h`, `fbgemm/FbgemmFP16.h`, `fbgemm/QuantUtils.h`
- Key helper symbols / 关键辅助符号: `fbgemm`
