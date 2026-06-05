# qlinear_prepack.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qlinear_prepack.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/Context.h>
 3: #include <ATen/core/Tensor.h>
 4: #include <ATen/cpp_custom_type_hack.h>
 5: #include <ATen/native/mkldnn/MKLDNNCommon.h>
 6: #include <ATen/native/quantized/PackedParams.h>
 7: #include <ATen/native/quantized/cpu/ACLUtils.h>
 8: #include <ATen/native/quantized/cpu/OnednnUtils.h>
 9: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
10: #include <ATen/native/quantized/cpu/QuantUtils.h>
11: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
12: #include <ATen/native/quantized/cpu/init_qnnpack.h>
13: #include <ATen/native/quantized/library.h>
14: #include <ATen/quantized/Quantizer.h>
15: #include <torch/custom_class.h>
16: #include <torch/library.h>
17:
18: #ifndef AT_PER_OPERATOR_HEADERS
19: #include <ATen/Functions.h>
20: #include <ATen/NativeFunctions.h>
```
- EN: This range pulls in required headers, including `ATen/Context.h`, `ATen/core/Tensor.h`, `ATen/cpp_custom_type_hack.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/Context.h`, `ATen/core/Tensor.h`, `ATen/cpp_custom_type_hack.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 21-40
```cpp
21: #else
22: #include <ATen/ops/_saturate_weight_to_fp16.h>
23: #include <ATen/ops/_saturate_weight_to_fp16_native.h>
24: #include <ATen/ops/_wrapped_linear_prepack_native.h>
25: #include <ATen/ops/_wrapped_quantized_linear_prepacked_native.h>
26: #include <ATen/ops/dequantize.h>
27: #include <ATen/ops/empty.h>
28: #include <ATen/ops/quantize_per_tensor.h>
29: #include <ATen/ops/zeros.h>
30: #endif
31:
32: #include <c10/util/irange.h>
33:
34: #include <algorithm>
35: #include <utility>
36: #include <vector>
37:
38: #ifdef USE_FBGEMM
39: namespace {
40: // Calculate the column offsets.
```
- EN: This range pulls in required headers, including `ATen/ops/_saturate_weight_to_fp16.h`, `ATen/ops/_saturate_weight_to_fp16_native.h`, `ATen/ops/_wrapped_linear_prepack_native.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/ops/_saturate_weight_to_fp16.h`, `ATen/ops/_saturate_weight_to_fp16_native.h`, `ATen/ops/_wrapped_linear_prepack_native.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 41-63
```cpp
41: // Note this includes the sum of the columns as well as the scalar term
42: // B_zero_point * K, whereas the row_offsets created by
43: // PackAWithQuantRowOffset is only the sum of the A rows.
44: void calc_col_offsets_transpose(
45:     int K,
46:     int N,
47:     const int8_t* Bint8,
48:     int32_t* B_zero_point,
49:     int32_t* col_offsets,
50:     c10::QScheme qtype) {
51:   for (const auto i : c10::irange(N)) {
52:     int32_t sum = 0;
53:     for (const auto j : c10::irange(K)) {
54:       sum += Bint8[i * K + j];
55:     }
56:     if (qtype == c10::kPerTensorAffine) {
57:       col_offsets[i] = sum - B_zero_point[0] * K;
58:     } else {
59:       col_offsets[i] = sum - B_zero_point[i] * K;
60:     }
61:   }
62: }
63: } // namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `calc_col_offsets_transpose`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `calc_col_offsets_transpose`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 65-83
```cpp
65: c10::intrusive_ptr<LinearPackedParamsBase> PackedLinearWeight::prepack(
66:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
67:     at::Tensor weight,
68:     std::optional<at::Tensor> bias) {
69:   TORCH_CHECK(
70:       weight.dim() == 2,
71:       "The weight tensor for quantized::linear_prepack (fbgemm) should"
72:       " be 2-dimensional.");
73:
74:   auto N = weight.size(0);
75:   auto K = weight.size(1);
76:
77:   // TODO: contiguous is called for further JIT optimizations.
78:   auto weight_contig = weight.contiguous();
79:   const auto qtype = weight.qscheme();
80:   std::vector<int32_t> weight_zero_points_int32(1, 0);
81:   if (qtype == c10::kPerTensorAffine) {
82:     weight_zero_points_int32[0] = {static_cast<int32_t>(weight.q_zero_point())};
83:   } else if (qtype == c10::kPerChannelAffine) {
```
- EN: The main symbol in this range is `prepack`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `prepack`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 84-110
```cpp
 84:     weight_zero_points_int32.resize(N, 0);
 85:     for (const auto i : c10::irange(N)) {
 86:       weight_zero_points_int32[i] =
 87:           weight.q_per_channel_zero_points()[i].item<int32_t>();
 88:     }
 89:   }
 90:   std::vector<float> weight_scales_float(1, 0.0);
 91:   if (qtype == c10::kPerTensorAffine) {
 92:     weight_scales_float[0] = {static_cast<float>(weight.q_scale())};
 93:   } else if (qtype == c10::kPerChannelAffine) {
 94:     weight_scales_float.resize(N, 0.0);
 95:     for (const auto i : c10::irange(N)) {
 96:       weight_scales_float[i] = weight.q_per_channel_scales()[i].item<float>();
 97:     }
 98:   }
 99:
100:   int8_t* weight_ptr_int8 =
101:       reinterpret_cast<int8_t*>(weight_contig.data_ptr<c10::qint8>());
102:
103:   std::vector<int32_t> col_offsets(N);
104:   calc_col_offsets_transpose(
105:       /*K=*/static_cast<int>(K),
106:       /*N=*/static_cast<int>(N),
107:       /*Bint8=*/weight_ptr_int8,
108:       /*B_zero_point=*/weight_zero_points_int32.data(),
109:       /*col_offsets=*/col_offsets.data(),
110:       /*qtype=*/qtype);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 112-137
```cpp
112:   std::optional<at::Tensor> bias_contig;
113:   if (bias.has_value()) {
114:     at::Tensor bias_vec = bias.value();
115:     TORCH_CHECK(bias_vec.dim() == 1, "bias should be a vector (1D Tensor)");
116:     TORCH_CHECK(
117:         bias_vec.size(0) == N,
118:         "bias should have N elements: " + std::to_string(N));
119:     bias_contig = bias->contiguous();
120:   }
121:   auto ret_ptr = c10::make_intrusive<PackedLinearWeight>(
122:       std::make_unique<fbgemm::PackBMatrix<int8_t>>(
123:           /*trans=*/fbgemm::matrix_op_t::Transpose,
124:           /*nRow=*/K,
125:           /*nCol=*/N,
126:           /*smat=*/weight_ptr_int8,
127:           /*ld=*/K,
128:           /*pmat=*/nullptr, // PackBMatrix manages ownership of pmat
129:           /*groups=*/1),
130:       bias_contig,
131:       col_offsets,
132:       weight_scales_float,
133:       weight_zero_points_int32,
134:       qtype);
135:   return ret_ptr;
136: }
137: #endif // USE_FBGEMM
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 139-165
```cpp
139: #ifdef USE_PYTORCH_QNNPACK
140: c10::intrusive_ptr<LinearPackedParamsBase> PackedLinearWeightsQnnp::prepack(
141:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
142:     at::Tensor weight,
143:     std::optional<at::Tensor> bias_in) {
144:   TORCH_CHECK(
145:       weight.dim() == 2,
146:       "quantized::linear_prepack (qnnpack): Weight tensor rank should be == 2");
147:
148:   int64_t rows_w = weight.size(0);
149:   at::Tensor bias_fp32;
150:   if (bias_in.has_value()) {
151:     bias_fp32 = bias_in.value();
152:   } else {
153:     bias_fp32 = at::zeros(rows_w, weight.options().dtype(at::kFloat));
154:   }
155:   TORCH_CHECK(
156:       !bias_fp32.defined() ||
157:           (bias_fp32.ndimension() == 1 && bias_fp32.size(0) == rows_w),
158:       "quantized::linear_prepack (qnnpack): Given weight of size ",
159:       weight.sizes(),
160:       ", expected bias to be 1-dimensional with ",
161:       rows_w,
162:       " elements",
163:       ", but got bias of size ",
164:       bias_fp32.sizes(),
165:       " instead");
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `prepack`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `prepack`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 167-186
```cpp
167:   at::Tensor weight_contig = weight.contiguous();
168:   auto [w_zero_points, w_scales] =
169:       make_zero_points_and_scales_tensor(weight_contig);
170:
171:   at::native::initQNNPACK();
172:
173:   // We set the pre-packed linear weights to nullptr below as we call pre-pack
174:   // during the first invocation of operator run. Refer to Linear.cpp for more
175:   // details. TODO Update to actually call pre-pack here once bias is removed
176:   // from pre-packing step.
177:   auto wt_ptr = c10::make_intrusive<PackedLinearWeightsQnnp>(
178:       nullptr,
179:       weight_contig, /* int8_t weight */
180:       bias_fp32.contiguous(), /* fp32 bias */
181:       std::nullopt, /* input_scale */
182:       w_scales,
183:       std::move(w_zero_points));
184:   return wt_ptr;
185: }
186: #endif // USE_PYTORCH_QNNPACK
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 188-206
```cpp
188: #ifdef USE_FBGEMM
189:
190: c10::intrusive_ptr<LinearPackedParamsBase> PackedLinearWeightFp16::prepack(
191:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
192:     at::Tensor weight,
193:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
194:     std::optional<at::Tensor> bias) {
195:
196:   weight = at::_saturate_weight_to_fp16(weight);
197:
198:   const int64_t K = weight.size(1);
199:   const int64_t N = weight.size(0);
200:   at::Tensor weight_contig = weight.contiguous();
201:   float* weight_contig_ptr = weight_contig.data_ptr<float>();
202:
203:   // TODO(mingzhe09088):
204:   // Consider using a functor here in PackedGemmMatrixFP16
205:   // Comments from (XQ): Not entirely sure this make_unique is safe.
206:   // make_unique is created with regular "new", and freed through
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `prepack`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `prepack`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 207-226
```cpp
207:   // TypeMetaData::deleteFn in this function. This is perfectly fine if the
208:   // tensors are created and freed within this translation unit. It might be
209:   // very problematic if that tensor flows across dll boundaries.
210:   auto ptr = c10::make_intrusive<PackedLinearWeightFp16>(
211:       std::make_unique<fbgemm::PackedGemmMatrixFP16>(
212:           fbgemm::matrix_op_t::Transpose, K, N, 1, weight_contig_ptr),
213:       bias);
214:   return ptr;
215: }
216: #endif // USE_FBGEMM
217:
218: #if AT_MKLDNN_ENABLED()
219: c10::intrusive_ptr<LinearPackedParamsBase> PackedLinearWeightsOnednn::prepack(
220:     at::Tensor weight,
221:     std::optional<at::Tensor> bias) {
222:   TORCH_CHECK(
223:       weight.dim() == 2,
224:       "The weight tensor for quantized::linear_prepack (onednn) should"
225:       " be 2-dimensional.");
226:   // Weight
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 227-252
```cpp
227:   std::vector<int64_t> dims = weight.sizes().vec();
228:   auto N = weight.size(0);
229:   std::vector<int32_t> wgt_zero_points;
230:   ideep::scale_t wgt_scales;
231:   const auto qtype = weight.qscheme();
232:   if (qtype == c10::kPerTensorAffine) {
233:     TORCH_CHECK(
234:         weight.q_zero_point() == 0,
235:         "quantized::linear_prepack: ONEDNN only supports symmetric quantization of weight,"
236:         " whose zero point must be 0, but got ", weight.q_zero_point());
237:     wgt_zero_points = std::vector<int32_t>(1, weight.q_zero_point());
238:     wgt_scales = ideep::scale_t(1, 1.0/weight.q_scale()); // Scales of ONEDNN and PyTorch are reciprocal
239:   } else if (qtype == c10::kPerChannelAffine) {
240:     wgt_zero_points.resize(N);
241:     wgt_scales.resize(N);
242:     for (int i = 0; i < N; ++i) {
243:       wgt_zero_points[i] = weight.q_per_channel_zero_points()[i].item<int32_t>();
244:       TORCH_CHECK(
245:           wgt_zero_points[i] == 0,
246:           "quantized::linear_prepack: ONEDNN only supports symmetric quantization of weight,"
247:           " whose zero point must be 0, but got ",  wgt_zero_points[i], ", at index ", i);
248:       wgt_scales[i] = 1.0f / weight.q_per_channel_scales()[i].item<float>(); // Scales of ONEDNN and PyTorch are reciprocal
249:     }
250:   } else {
251:     TORCH_CHECK(false, "Unsupported qscheme: ", toString(qtype));
252:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 254-272
```cpp
254:   // Prepack weight
255:   auto weight_copy = weight.clone();
256:   ideep::tensor wgt = ideep::tensor({dims, dnnl::memory::data_type::s8}, weight_copy.data_ptr());
257:   wgt.transpose_(0, 1); // ONEDNN requires transposed weight
258:   auto src_dims = ideep::dims(); // Unknown when prepacking
259:   ideep::attr_t op_attr;
260:   op_attr.set_zero_points_mask(DNNL_ARG_SRC, 0);
261:   auto w_desc = ideep::matmul_forward::expected_weights_desc(wgt.get_dims(), src_dims, dnnl::memory::data_type::s8,
262:                                                              dnnl::memory::data_type::u8, op_attr);
263:   ideep::tensor exp_wgt(w_desc);
264:   exp_wgt.feed_from(wgt);
265:   ideep::tensor * packed_weight_p = new ideep::tensor(std::move(exp_wgt));
266:   packed_weight_p->set_scale(wgt_scales);
267:   packed_weight_p->set_zero_point(wgt_zero_points);
268:   std::unique_ptr<ideep::tensor> weight_ptr(packed_weight_p);
269:   // Bias
270:   std::optional<ideep::tensor> onednn_bias{std::nullopt};
271:   if (bias.has_value()) {
272:     auto& b = bias.value();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 273-295
```cpp
273:     auto bias_size = b.sizes().vec();
274:     bias_size.insert(bias_size.begin(), 1);
275:     TORCH_CHECK(
276:         bias_size[1] == weight_ptr->get_dim(1),
277:         "bias should have N elements: ",
278:         std::to_string(weight_ptr->get_dim(1)),
279:         ", but got ", bias_size[1]);
280:     auto bias_desc = ideep::tensor::desc(bias_size, dnnl::memory::data_type::f32);
281:     ideep::tensor packed_bias;
282:     packed_bias.init(bias_desc, b.data_ptr());
283:     onednn_bias = std::optional<ideep::tensor>(packed_bias);
284:   }
285: #if AT_MKLDNN_ACL_ENABLED()
286:   if (qtype == c10::kPerTensorAffine) {
287:     return c10::make_intrusive<PackedLinearWeightsACL>(PackedLinearWeightsACL{
288:         std::move(weight_ptr), onednn_bias, weight, bias});
289:   }
290: #endif // #if AT_MKLDNN_ACL_ENABLED()
291:   auto ret_ptr =
292:       c10::make_intrusive<PackedLinearWeightsOnednn>(PackedLinearWeightsOnednn{
293:           std::move(weight_ptr), onednn_bias, weight, bias});
294:   return ret_ptr;
295: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 297-315
```cpp
297: static inline at::Tensor pack_weight_to_onednn_tensor(
298:     const at::Tensor& weight,
299:     std::optional<torch::List<int64_t>>& input_shape) {
300:   at::ScalarType weigh_dtype = weight.scalar_type();
301:   TORCH_CHECK(
302:     weigh_dtype == at::kChar || weigh_dtype == at::kFloat8_e4m3fn,
303:     "Weight should be of type int8 or float8_e4m3fn");
304:   bool is_fp8 = weigh_dtype == at::kFloat8_e4m3fn;
305: #if defined(__powerpc__)
306:   if (is_fp8){
307: #else
308:   if(is_fp8 && !cpuinfo_has_x86_amx_fp16()) {
309: #endif
310:     // oneDNN's fp8 requires AMX support
311:     // If AMX is not available, fall back to reference implementation
312:     // Transpose weight to align with behavior in oneDNN
313:     return weight.t();
314:   }
315:   std::vector<int64_t> w_dims = weight.sizes().vec();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `pack_weight_to_onednn_tensor`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `pack_weight_to_onednn_tensor`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 316-338
```cpp
316:   auto w_data_type = is_fp8
317:       ? dnnl::memory::data_type::f8_e4m3
318:       : dnnl::memory::data_type::s8;
319:   ideep::tensor wei = ideep::tensor({w_dims, w_data_type}, weight.data_ptr());
320:   wei.transpose_(0, 1); // oneDNN requires transposed weight
321:   ideep::dims input_dims = input_shape.has_value() ? input_shape.value().vec() : ideep::dims();
322:   ideep::attr_t op_attr;
323:   if (!is_fp8) {
324:     op_attr.set_zero_points_mask(DNNL_ARG_SRC, 0);
325:   }
326:   auto x_data_type = is_fp8
327:       ? dnnl::memory::data_type::f8_e4m3
328:       : dnnl::memory::data_type::u8;
329:   auto w_desc = ideep::matmul_forward::expected_weights_desc(
330:       wei.get_dims(), input_dims, w_data_type, x_data_type, op_attr);
331:   ideep::tensor expected_weight(w_desc);
332:   expected_weight.feed_from(wei);
333:   auto packed_weight = at::native::new_with_itensor_mkldnn(
334:       std::move(expected_weight),
335:       c10::optTypeMetaToScalarType(weight.options().dtype_opt()),
336:       weight.options().device_opt());
337:   return packed_weight;
338: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 340-358
```cpp
340: static inline at::Tensor pack_weight_to_fp16_onednn_tensor(
341:     at::Tensor& weight,
342:     std::optional<torch::List<int64_t>>& input_shape) {
343:   TORCH_CHECK(weight.scalar_type() == at::kHalf || weight.scalar_type() == at::kFloat, "Weight should be of type float or float16");
344:   weight = weight.scalar_type() == at::kHalf ? weight : at::_saturate_weight_to_fp16(weight);
345:   std::vector<int64_t> w_dims = weight.sizes().vec();
346:   auto weight_fp16 = weight.to(at::kHalf);
347:   ideep::tensor wei = ideep::tensor({w_dims, dnnl::memory::data_type::f16}, weight_fp16.data_ptr());
348:   auto expected_weight = wei.transpose(0, 1); // oneDNN requires transposed weight
349:   // Onednn does not support f32f16f32 matmul, so we need to convert weight to f32 before compute
350:   // Therefore, we just return weight in plain format
351:   auto packed_weight = at::native::new_with_itensor_mkldnn(
352:       std::move(expected_weight),
353:       c10::kHalf,
354:       weight.options().device_opt());
355:   return packed_weight;
356: }
357:
358: #endif // #if AT_MKLDNN_ENABLED()
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `pack_weight_to_fp16_onednn_tensor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `pack_weight_to_fp16_onednn_tensor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 360-381
```cpp
360: namespace at::native {
361:
362: at::Tensor _saturate_weight_to_fp16(const Tensor& weight) {
363:   Tensor weight_contig = weight.contiguous();
364:   float* weight_contig_ptr = weight_contig.data_ptr<float>();
365:   quant_utils::HandleWeightsSaturation(weight.size(0) * weight.size(1), weight_contig_ptr);
366:   return weight;
367: }
368:
369: template <class... Inputs>
370: static inline std::vector<c10::IValue> makeStack(Inputs&&... inputs) {
371:   return {std::forward<Inputs>(inputs)...};
372: }
373:
374: template <class... Args>
375: static inline std::vector<c10::IValue> callOpByHandle(
376:     const c10::OperatorHandle& op,
377:     Args... args) {
378:   auto stack = makeStack(std::forward<Args>(args)...);
379:   c10::Dispatcher::singleton().callBoxed(op, &stack);
380:   return stack;
381: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_saturate_weight_to_fp16`, `makeStack`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_saturate_weight_to_fp16`, `makeStack`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 383-404
```cpp
383: template <class... Args>
384: static inline std::vector<c10::IValue> callOpByName(
385:     const char* func_name,
386:     const char* overload_name,
387:     Args... args) {
388:   const std::optional<c10::OperatorHandle> op_handle =
389:       c10::Dispatcher::singleton().findSchema({func_name, overload_name});
390:   assert(op_handle.has_value());
391:   return callOpByHandle(op_handle.value(), std::forward<Args>(args)...);
392: }
393:
394: static at::Tensor wrapped_quantized_linear(
395:     at::Tensor input,
396:     const at::Tensor& input_scale,
397:     const at::Tensor& input_zero_point,
398:     const at::Tensor& weight,
399:     const at::Tensor& weight_scale,
400:     const at::Tensor& weight_zero_point,
401:     const at::Tensor& bias,
402:     const at::Tensor& output_scale,
403:     const at::Tensor& output_zero_point,
404:     [[maybe_unused]] const int64_t out_channel);
```
- EN: The main symbol in this range is `callOpByName`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `callOpByName`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 406-432
```cpp
406: at::Tensor wrapped_quantized_linear(
407:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
408:     at::Tensor input,
409:     const at::Tensor& input_scale,
410:     const at::Tensor& input_zero_point,
411:     const at::Tensor& weight,
412:     const at::Tensor& weight_scale,
413:     const at::Tensor& weight_zero_point,
414:     const at::Tensor& bias,
415:     const at::Tensor& output_scale,
416:     const at::Tensor& output_zero_point,
417:     [[maybe_unused]] const int64_t out_channel) {
418:   //This op does four things:
419:   // 1. Use quantize_per_tensor to quantize the input
420:   // 2. Use quantized::linear_prepack to prepack the weight and bias
421:   // 3. Use quantized::linear to do the int8 linear quantized computation
422:   // 4. Use dequantize to dequantize the result of quantized::linear
423:   // The reason we do this is because we want to have such wrapper op to
424:   // bypass the issue from torch.export
425: #ifdef USE_FBGEMM
426:   auto qw = at::quantize_per_tensor(
427:       weight, weight_scale, weight_zero_point, c10::ScalarType::QInt8);
428:   auto op = Dispatcher::singleton()
429:                 .findSchemaOrThrow("quantized::linear_prepack", "")
430:                 .typed<c10::intrusive_ptr<LinearPackedParamsBase>(
431:                     at::Tensor, std::optional<at::Tensor>)>();
432:   auto packed_params = op.call(qw, bias);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `wrapped_quantized_linear`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `wrapped_quantized_linear`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 434-460
```cpp
434:   auto qx = at::quantize_per_tensor(
435:       input, input_scale, input_zero_point, c10::ScalarType::QUInt8);
436:
437:   const auto scale_val = output_scale.item().toFloat();
438:   const auto zero_point_val = output_zero_point.item().toLong();
439:
440:   auto result = callOpByName(
441:       "quantized::linear", "", qx, packed_params, scale_val, zero_point_val);
442:
443:   return at::dequantize(result[0].toTensor());
444: #else // USE_FBGEMM
445:   TORCH_CHECK(
446:       false, "This PyTorch installation was not built with FBGEMM operators");
447: #endif // USE_FBGEMM
448: }
449:
450: static at::Tensor wrapped_quantized_linear_meta(
451:     at::Tensor input,
452:     [[maybe_unused]] const at::Tensor& input_scale,
453:     [[maybe_unused]] const at::Tensor& input_zero_point,
454:     const at::Tensor& weight,
455:     [[maybe_unused]] const at::Tensor& weight_scale,
456:     [[maybe_unused]] const at::Tensor& weight_zero_point,
457:     [[maybe_unused]] const at::Tensor& bias,
458:     [[maybe_unused]] const at::Tensor& output_scale,
459:     [[maybe_unused]] const at::Tensor& output_zero_point,
460:     [[maybe_unused]] const int64_t out_channel);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 462-483
```cpp
462: at::Tensor wrapped_quantized_linear_meta(
463:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
464:     at::Tensor input,
465:     [[maybe_unused]] const at::Tensor& input_scale,
466:     [[maybe_unused]] const at::Tensor& input_zero_point,
467:     const at::Tensor& weight,
468:     [[maybe_unused]] const at::Tensor& weight_scale,
469:     [[maybe_unused]] const at::Tensor& weight_zero_point,
470:     [[maybe_unused]] const at::Tensor& bias,
471:     [[maybe_unused]] const at::Tensor& output_scale,
472:     [[maybe_unused]] const at::Tensor& output_zero_point,
473:     [[maybe_unused]] const int64_t out_channel) {
474: #ifdef USE_FBGEMM
475:   const at::SymInt M = input.sym_size(0);
476:   const at::SymInt N = weight.sym_size(0);
477:   auto Y = at::empty_symint({M, N}, input.options().dtype(at::kFloat));
478:   return Y;
479: #else // USE_FBGEMM
480:   TORCH_CHECK(
481:       false, "This PyTorch installation was not built with FBGEMM operators");
482: #endif // USE_FBGEMM
483: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `wrapped_quantized_linear_meta`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `wrapped_quantized_linear_meta`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 485-505
```cpp
485: at::Tensor _wrapped_linear_prepack(const at::Tensor& weight,
486:     const at::Tensor& weight_scale,
487:     const at::Tensor& weight_zero_point,
488:     const at::Tensor& bias) {
489:   // This op does two things
490:   // 1. Use quantize_per_tensor to quantize the weight
491:   // 2. Use quantized::linear_prepack to prepack the weight and bias
492:   // The reason we do this is because we want to have such wrapper op to
493:   // save the quantized weight as constants for AOTI
494: #ifdef USE_FBGEMM
495:   TORCH_CHECK(
496:       weight.dim() == 2,
497:       "fbgemm weight packing only packs matrices not vectors.");
498:   auto qw = at::quantize_per_tensor(
499:       weight, weight_scale, weight_zero_point, c10::ScalarType::QInt8);
500:
501:   auto op = Dispatcher::singleton()
502:                 .findSchemaOrThrow("quantized::linear_prepack", "")
503:                 .typed<c10::intrusive_ptr<LinearPackedParamsBase>(
504:                     at::Tensor, std::optional<at::Tensor>)>();
505:   auto packed_params = op.call(qw, bias);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_wrapped_linear_prepack`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_wrapped_linear_prepack`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 507-525
```cpp
507:   auto unique_ptr_wrapper =
508:       std::make_unique<decltype(packed_params)>(std::move(packed_params));
509:   auto ret = cpp_custom_type_hack::create(
510:       std::move(unique_ptr_wrapper), weight.options());
511:   return ret;
512: #else // USE_FBGEMM
513:   TORCH_CHECK(
514:       false, "This PyTorch installation was not built with FBGEMM operators");
515: #endif // USE_FBGEMM
516: }
517:
518: at::Tensor _wrapped_quantized_linear_prepacked(const at::Tensor& input, const at::Tensor& input_scale,
519:     const at::Tensor& input_zero_point,
520:     const at::Tensor& packed_weight,
521:     const at::Tensor& output_scale,
522:     const at::Tensor& output_zero_point,
523:     [[maybe_unused]] const int64_t out_channel) {
524:   // This op is similar to wrapped_quantized_linear, but it takes the prepacked weight
525: #ifdef USE_FBGEMM
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_wrapped_quantized_linear_prepacked`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_wrapped_quantized_linear_prepacked`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 526-545
```cpp
526:   auto qx = at::quantize_per_tensor(
527:       input, input_scale, input_zero_point, c10::ScalarType::QUInt8);
528:   const auto scale_val = output_scale.item().toFloat();
529:   const auto zero_point_val = output_zero_point.item().toLong();
530:   auto packed_weight_ptr =
531:       // @lint-ignore CLANGTIDY facebook-hte-Deprecated
532:       cpp_custom_type_hack::cast<c10::intrusive_ptr<LinearPackedParamsBase>>(
533:           packed_weight);
534:   auto result = callOpByName(
535:       "quantized::linear", "", qx, packed_weight_ptr, scale_val, zero_point_val);
536:
537:   return at::dequantize(result[0].toTensor());
538: #else // USE_FBGEMM
539:   TORCH_CHECK(
540:       false, "This PyTorch installation was not built with FBGEMM operators");
541: #endif // USE_FBGEMM
542: }
543:
544: static at::Tensor _wrapped_linear_prepack_meta(const at::Tensor& weight,
545:     [[maybe_unused]] const at::Tensor& weight_scale,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 546-574
```cpp
546:     [[maybe_unused]] const at::Tensor& weight_zero_point,
547:     [[maybe_unused]] const at::Tensor& bias) {
548: #ifdef USE_FBGEMM
549:   TORCH_CHECK(
550:       weight.dim() == 2,
551:       "fbgemm weight packing only packs matrices not vectors.");
552:   const at::SymInt M = weight.sym_size(0);
553:   const at::SymInt N = weight.sym_size(1);
554:   auto Y = at::empty_symint({M, N}, weight.options().dtype(at::kFloat));
555:   return Y;
556: #else // USE_FBGEMM
557:   TORCH_CHECK(
558:       false, "This PyTorch installation was not built with FBGEMM operators");
559: #endif // USE_FBGEMM
560: }
561:
562: static at::Tensor _wrapped_quantized_linear_prepacked_meta(const at::Tensor& input,
563:     [[maybe_unused]] const at::Tensor& input_scale,
564:     [[maybe_unused]] const at::Tensor& input_zero_point,
565:     [[maybe_unused]] const at::Tensor& packed_weight,
566:     [[maybe_unused]] const at::Tensor& output_scale,
567:     [[maybe_unused]] const at::Tensor& output_zero_point,
568:     const int64_t out_channel) {
569: #ifdef USE_FBGEMM
570:   auto out_sizes = input.sym_sizes().vec();
571:   TORCH_CHECK(
572:         out_sizes.size() == 2,
573:         "The dimension of weight tensor should be equal to 2");
574:   out_sizes[out_sizes.size() - 1] = out_channel;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_wrapped_quantized_linear_prepacked_meta`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_wrapped_quantized_linear_prepacked_meta`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 576-594
```cpp
576:   return at::empty_symint(out_sizes, input.options());
577: #else // USE_FBGEMM
578:   TORCH_CHECK(
579:       false, "This PyTorch installation was not built with FBGEMM operators");
580: #endif // USE_FBGEMM
581: }
582:
583: namespace {
584:
585: class QLinearPackWeightInt8 final {
586:  public:
587:   static c10::intrusive_ptr<LinearPackedParamsBase> run(
588:       at::Tensor weight,
589:       std::optional<Tensor> bias) {
590:     auto& ctx = at::globalContext();
591:
592: #ifdef USE_FBGEMM
593:     if (ctx.qEngine() == at::QEngine::FBGEMM ||
594:         ctx.qEngine() == at::QEngine::X86) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `run`, `QLinearPackWeightInt8`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `run`, `QLinearPackWeightInt8`，它们直接构成本文件的算子逻辑。

### Lines 595-614
```cpp
595:       return PackedLinearWeight::prepack(std::move(weight), std::move(bias));
596:     }
597: #endif
598: #ifdef USE_PYTORCH_QNNPACK
599:     if (ctx.qEngine() == at::QEngine::QNNPACK) {
600:       return PackedLinearWeightsQnnp::prepack(
601:           std::move(weight), std::move(bias));
602:     }
603: #endif
604: #if AT_MKLDNN_ENABLED()
605:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
606:       return PackedLinearWeightsOnednn::prepack(std::move(weight), std::move(bias));
607:     }
608: #endif // #if AT_MKLDNN_ENABLED()
609:     TORCH_CHECK(
610:         false,
611:         "Didn't find engine for operation quantized::linear_prepack ",
612:         toString(ctx.qEngine()));
613:   }
614: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 616-634
```cpp
616: class QLinearPackWeightFp16 final {
617:  public:
618:   static c10::intrusive_ptr<LinearPackedParamsBase> run(
619:       at::Tensor weight,
620:       std::optional<Tensor> bias) {
621:     auto& ctx = at::globalContext();
622: #ifdef USE_FBGEMM
623:     // temporarily convert weight back to fp32, needs to be fixed
624:     // after fbgemm fixes the interface for their prepacking op (take fp16 input0
625:     weight = weight.to(ScalarType::Float);
626:     if (ctx.qEngine() == at::QEngine::FBGEMM ||
627:         ctx.qEngine() == at::QEngine::X86) {
628:       return PackedLinearWeightFp16::prepack(
629:           std::move(weight), std::move(bias));
630:     }
631: #endif // USE_FBGEMM
632: #ifdef USE_PYTORCH_QNNPACK
633:     if (ctx.qEngine() == at::QEngine::QNNPACK) {
634:       TORCH_CHECK(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run`, `QLinearPackWeightFp16`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run`, `QLinearPackWeightFp16`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 635-653
```cpp
635:           false,
636:           "quantized::linear_prepack_fp16 is currently "
637:           "not supported by QNNPACK");
638:     }
639: #endif // USE_PYTORCH_QNNPACK
640: #if AT_MKLDNN_ENABLED()
641:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
642:       TORCH_CHECK(
643:           false,
644:           "quantized::linear_prepack_fp16 is currently "
645:           "not supported by ONEDNN");
646:     }
647: #endif // #if AT_MKLDNN_ENABLED()
648:     TORCH_CHECK(
649:         false,
650:         "Didn't find engine for operation quantized::linear_prepack_fp16 ",
651:         toString(ctx.qEngine()));
652:   }
653: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 655-679
```cpp
655: class QLinearPackWeightInt8Legacy final {
656:  public:
657:   static Tensor run(
658:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
659:     [[maybe_unused]] at::Tensor weight,
660:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
661:     [[maybe_unused]] std::optional<Tensor> bias) {
662:     TORCH_CHECK(false,
663:         "This model uses an outdated version of quantized.linear_prepack. "
664:         "Please re-export your model using the newer definitions in torch.jit.quantized");
665:   }
666: };
667:
668: class QLinearPackWeightFp16Legacy final {
669:  public:
670:   static Tensor run(
671:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
672:     [[maybe_unused]] at::Tensor weight,
673:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
674:     [[maybe_unused]] std::optional<Tensor> bias) {
675:     TORCH_CHECK(false,
676:         "This model uses an outdated version of quantized.linear_prepack_fp16. "
677:         "Please re-export your model using the newer definitions in torch.jit.quantized");
678:   }
679: };
```
- EN: The main symbol in this range is `run`, `QLinearPackWeightInt8Legacy`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QLinearPackWeightInt8Legacy`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 681-699
```cpp
681: class QLinearPackWeightInt8Onednn final {
682:  public:
683:   static at::Tensor run(
684:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
685:     [[maybe_unused]] at::Tensor weight, // Not QTensor
686:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
687:     [[maybe_unused]] std::optional<torch::List<int64_t>> input_shape) {
688: #if AT_MKLDNN_ENABLED()
689:     return pack_weight_to_onednn_tensor(weight, input_shape);
690: #else
691:     TORCH_CHECK(false, "Unimplemented as onednn is not available.");
692: #endif
693:   }
694: };
695:
696: class QLinearPackWeightFp16Onednn final {
697:  public:
698:   static at::Tensor run(
699:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run`, `QLinearPackWeightInt8Onednn`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run`, `QLinearPackWeightInt8Onednn`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 700-721
```cpp
700:     [[maybe_unused]] at::Tensor weight, // Not QTensor
701:     // NOLINTNEXTLINE(performance-unnecessary-value-param)
702:     [[maybe_unused]] std::optional<torch::List<int64_t>> input_shape) {
703: #if AT_MKLDNN_ENABLED()
704:     return pack_weight_to_fp16_onednn_tensor(weight, input_shape);
705: #else
706:     TORCH_CHECK(false, "Unimplemented as onednn is not available.");
707: #endif
708:   }
709: };
710:
711: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
712:   register_linear_params();
713:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_prepack"), TORCH_FN(QLinearPackWeightInt8::run));
714:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_prepack_legacy"), TORCH_FN(QLinearPackWeightInt8Legacy::run));
715: }
716:
717: TORCH_LIBRARY_IMPL(quantized, CPU, m) {
718:   register_linear_params();
719:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_prepack_fp16"), TORCH_FN(QLinearPackWeightFp16::run));
720:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_prepack_fp16_legacy"), TORCH_FN(QLinearPackWeightFp16Legacy::run));
721: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 723-749
```cpp
723: TORCH_LIBRARY_IMPL(_quantized, QuantizedCPU, m) {
724:   register_linear_params();
725:   m.impl(TORCH_SELECTIVE_NAME("_quantized::linear_prepack"), TORCH_FN(QLinearPackWeightInt8::run));
726: }
727:
728: TORCH_LIBRARY_IMPL(_quantized, CPU, m) {
729:   register_linear_params();
730:   m.impl(TORCH_SELECTIVE_NAME("_quantized::linear_prepack_fp16"), TORCH_FN(QLinearPackWeightFp16::run));
731:   m.impl(TORCH_SELECTIVE_NAME("_quantized::linear_prepack_fp16_legacy"), TORCH_FN(QLinearPackWeightFp16Legacy::run));
732:   m.impl(TORCH_SELECTIVE_NAME("_quantized::wrapped_quantized_linear"), TORCH_FN(wrapped_quantized_linear));
733:   m.impl(
734:       TORCH_SELECTIVE_NAME("_quantized::_wrapped_linear_prepack"),
735:       _wrapped_linear_prepack);
736:   m.impl(
737:       TORCH_SELECTIVE_NAME("_quantized::_wrapped_quantized_linear_prepacked"),
738:       _wrapped_quantized_linear_prepacked);
739: }
740:
741: TORCH_LIBRARY_IMPL(_quantized, Meta, m) {
742:   m.impl(TORCH_SELECTIVE_NAME("_quantized::wrapped_quantized_linear"), TORCH_FN(wrapped_quantized_linear_meta));
743:   m.impl(
744:       TORCH_SELECTIVE_NAME("_quantized::_wrapped_linear_prepack"),
745:       _wrapped_linear_prepack_meta);
746:   m.impl(
747:       TORCH_SELECTIVE_NAME("_quantized::_wrapped_quantized_linear_prepacked"),
748:       _wrapped_quantized_linear_prepacked_meta);
749: }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 751-760
```cpp
751: TORCH_LIBRARY_IMPL(onednn, CPU, m) {
752:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_prepack"), TORCH_FN(QLinearPackWeightInt8Onednn::run));
753: }
754:
755: TORCH_LIBRARY_IMPL(onednn, CPU, m) {
756:   m.impl(TORCH_SELECTIVE_NAME("onednn::linear_prepack_fp16"), TORCH_FN(QLinearPackWeightFp16Onednn::run));
757: }
758:
759: } // namespace
760: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Context.h`, `ATen/core/Tensor.h`, `ATen/cpp_custom_type_hack.h`, `ATen/native/mkldnn/MKLDNNCommon.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/ACLUtils.h`, `ATen/native/quantized/cpu/OnednnUtils.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`, `ATen/native/quantized/cpu/QuantUtils.h`, `ATen/native/quantized/cpu/fbgemm_utils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/custom_class.h`, `torch/library.h`, `algorithm`, `utility`, `vector`
- Key helper symbols / 关键辅助符号: `Scalar`, `ScalarType`, `qnnpack`, `fbgemm`
