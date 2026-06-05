# qlinear.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qlinear.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/Context.h>
 3: #include <ATen/Parallel.h>
 4: #include <ATen/TensorOperators.h>
 5: #include <ATen/core/Tensor.h>
 6: #include <ATen/core/List.h>
 7: #include <ATen/native/mkldnn/MKLDNNCommon.h>
 8: #include <ATen/native/quantized/PackedParams.h>
 9: #include <ATen/native/quantized/cpu/ACLUtils.h>
10: #include <ATen/native/quantized/cpu/OnednnUtils.h>
11: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
12: #include <ATen/native/quantized/cpu/QuantUtils.h>
13: #include <ATen/native/quantized/cpu/XnnpackUtils.h>
14: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
15: #include <ATen/native/quantized/cpu/qlinear.h>
16: #include <ATen/native/quantized/library.h>
17: #include <caffe2/utils/threadpool/pthreadpool-cpp.h>
18: #include <torch/library.h>
19:
20: #ifndef AT_PER_OPERATOR_HEADERS
21: #include <ATen/Functions.h>
22: #include <ATen/NativeFunctions.h>
23: #else
24: #include <ATen/ops/_empty_affine_quantized.h>         // for _empty_affine_q...
25: #include <ATen/ops/_empty_affine_quantized_native.h>  // for empty_affine_qu...
26: #include <ATen/ops/empty.h>                           // for empty
27: #include <ATen/ops/quantize_per_channel_native.h>     // for quantize_per_ch...
28: #include <ATen/ops/quantize_per_tensor_native.h>      // for quantize_per_te...
29: #include <ATen/ops/zeros.h>
30: #include <ATen/ops/_weight_int4pack_mm_for_cpu.h>
31: #include <ATen/ops/linear.h>
32: #include <ATen/ops/relu.h>
33: #include <ATen/ops/leaky_relu.h>
34: #include <ATen/ops/tanh.h>
35: #include <ATen/ops/gelu.h>
36: #include <ATen/ops/hardtanh.h>
37: #include <ATen/ops/hardswish.h>
38: #include <ATen/ops/sigmoid.h>
39: #endif
```
- EN: This range pulls in required headers, including `ATen/Context.h`, `ATen/Parallel.h`, `ATen/TensorOperators.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/Context.h`, `ATen/Parallel.h`, `ATen/TensorOperators.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 41-69
```cpp
41: #include <c10/util/irange.h>
42:
43: #include <algorithm>
44: #include <string>
45:
46: #ifdef USE_FBGEMM
47: template <bool ReluFused>
48: at::Tensor& PackedLinearWeight::apply_impl(
49:     const at::Tensor& input,
50:     double output_scale,
51:     int64_t output_zero_point,
52:     at::Tensor& output) {
53:   // uint8 * int8 -> uint8 (no quantization/dequantization)
54:
55:   // We make a strong guarantee that models using these operators will have
56:   // the same numerics across different machines. Therefore, we do not provide
57:   // a fallback path and rather fail loudly if we cannot run FBGEMM.
58:   TORCH_CHECK(
59:       fbgemm::fbgemmSupportedCPU(), "Your CPU does not support FBGEMM.");
60:   TORCH_CHECK(input.scalar_type() == c10::kQUInt8,
61:                 "Expected input data type ",
62:                 toString(c10::kQUInt8),
63:                 " but got ",
64:                 toString(input.scalar_type()));
65:
66:   // TODO: contiguous is called for further jit optimizations.
67:   auto input_contig = input.expect_contiguous();
68:   const auto* input_ptr =
69:       reinterpret_cast<uint8_t*>(input_contig->data_ptr<c10::quint8>());
```
- EN: This range pulls in required headers, including `c10/util/irange.h`, `algorithm`, `string`. It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic.
- CN: 这一段引入了所需头文件，例如 `c10/util/irange.h`, `algorithm`, `string`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。

### Lines 71-112
```cpp
 71:   TORCH_CHECK(
 72:       input.dim() >= 2,
 73:       "The dimension of input tensor should be larger than or equal to 2");
 74:   // C(output) = A(input) x B(weight), where C, A, B are M x N, M x K, K x N
 75:   // matrices, respectively.
 76:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
 77:   int64_t M = size_to_dim_(input.dim() - 1, input.sizes());
 78:
 79:   auto packB = w.get();
 80:
 81:   int64_t N = static_cast<int64_t>(packB->numCols());
 82:   int64_t K = input.sizes()[input.dim() - 1];
 83:   TORCH_CHECK(
 84:       K == static_cast<int64_t>(packB->numRows()),
 85:       "The number of rows in the packB should be equal to K: " +
 86:           std::to_string(K));
 87:
 88:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
 89:   float input_scale_float = input.q_scale();
 90:   int32_t input_zero_point_int32 = input.q_zero_point();
 91:
 92:   std::vector<float> output_multiplier_float(1, 0.0);
 93:   std::vector<float> act_times_w_scale(1, 0.0);
 94:   TORCH_CHECK(
 95:       w_scale.size() == w_zp.size(),
 96:       "Weight scales and zero points vectors should have the same size.");
 97:   if (q_scheme == c10::kPerTensorAffine) {
 98:     // Process the per tensor quantization.
 99:     act_times_w_scale[0] = (input_scale_float * w_scale[0]);
100:     output_multiplier_float[0] =
101:         act_times_w_scale[0] / static_cast<float>(output_scale);
102:   } else if (q_scheme == c10::kPerChannelAffine) {
103:     // Process the per channel quantization.
104:     output_multiplier_float.resize(N, 0.0);
105:     act_times_w_scale.resize(N, 1.0f);
106:     for (const auto i : c10::irange(N)) {
107:       act_times_w_scale[i] = (input_scale_float * w_scale[i]);
108:       output_multiplier_float[i] =
109:           act_times_w_scale[i] / static_cast<float>(output_scale);
110:     }
111:   }
112:   int32_t output_zero_point_int32 = static_cast<int32_t>(output_zero_point);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 114-142
```cpp
114:   const float* bias_ptr = nullptr;
115:   c10::MaybeOwned<at::Tensor> bias_contig;
116:   if (this->bias_.has_value()) {
117:     auto& bias = this->bias_.value();
118:     bias_contig = bias.expect_contiguous();
119:     TORCH_CHECK(bias_contig->dim() == 1, "bias should be a vector (1D Tensor)");
120:     TORCH_CHECK(
121:         bias_contig->sizes()[0] == N, "bias should have N elements: " + std::to_string(N));
122:     bias_ptr = reinterpret_cast<float*>(bias_contig->data_ptr<float>());
123:   }
124:
125:   // The resulting matrix here is 2-D, let's view it with the original
126:   // left hand dimensions of the input. Here are two examples:
127:   // 1. If the input tensor is {M, K}, the output tensor is {M, N}.
128:   // 2. If the input tensor is {b, M, K}, the output tensor is {b, M, N}.
129:   at::DimVector out_sizes(input.sizes());
130:   out_sizes.back() = N;
131:   // Resize output Tensor
132:   output.resize_(out_sizes);
133:
134:   // Allocate a buffer for fbgemmPacked to use
135:   auto buffer = at::empty(out_sizes, output.options().dtype(at::kInt));
136:
137:   auto output_data = reinterpret_cast<uint8_t*>(output.data_ptr<c10::quint8>());
138:
139:   int num_tasks = at::get_num_threads();
140:   at::parallel_for(0, num_tasks, 1, [&](int64_t begin, int64_t end) {
141:     for (const auto task_id : c10::irange(begin, end)) {
142:       // This operation does the following:
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 143-172
```cpp
143:       // 1) Creates a "row buffer" vector with offset values that must be
144:       //    added to the integer matrix multiplication operation to ensure
145:       //    correctness. This "row buffer" is also called the row offset, and
146:       //    it is needed when we use affine quantization for weights.
147:       // 2) Packs the resulting quantized matrix into vector-register and
148:       //    cache friendly tiles.
149:       //
150:       //  Note this is not executed eagerly, but rather within the
151:       //  fbgemmPacked call below.
152:       fbgemm::PackAWithRowOffset<uint8_t> packA(
153:           /*trans=*/fbgemm::matrix_op_t::NoTranspose,
154:           /*nRow=*/M,
155:           /*nCol=*/K,
156:           /*smat=*/input_ptr,
157:           /*ld=*/K,
158:           /*pmat=*/nullptr); // Currently, packA manages ownership of `pmat`.
159:                              // TODO: Consider a way to pre-allocate and reuse
160:                              // pmat buffer.
161:
162:       // ReQuantizeOutput requires pointers to the zero point values,
163:       // since in the case of rowwise quantization these will be arrays rather
164:       // than scalars. But in this case, we're doing whole-tensor quantization
165:       // so we just pass a pointer to the scale values (and internally
166:       // ReQuantizeOutput won't index past 0.
167:
168:       // This is the end of the pipeline, pass the resulting matrix through.
169:       fbgemm::DoNothing<> doNothingObj{};
170:
171:       if (q_scheme == c10::kPerTensorAffine) {
172:         // Process the per tensor quantization.
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 173-202
```cpp
173:         //
174:         // After the uint8 * int8 matrix multiplication is performed, this
175:         // operation does:
176:         //  1) Add in row and column offsets to the rows and columns,
177:         //  respectively.
178:         //  2) Add in the bias term.
179:         fbgemm::ReQuantizeOutput<
180:             ReluFused,
181:             fbgemm::QuantizationGranularity::TENSOR,
182:             float>
183:             outputProcObj(
184:                 doNothingObj,
185:                 output_multiplier_float.data(),
186:                 output_zero_point_int32,
187:                 input_zero_point_int32,
188:                 w_zp.data(),
189:                 packA.getRowOffsetBuffer(),
190:                 col_offsets.data(),
191:                 bias_ptr,
192:                 N, /* nCol */
193:                 1 /* groups */,
194:                 act_times_w_scale.data());
195:
196:         // Do the GEMM
197:         fbgemm::fbgemmPacked(
198:             /*packA=*/packA,
199:             /*packB=*/*packB,
200:             /*C=*/output_data,
201:             /*C_buffer=*/buffer.data_ptr<int32_t>(),
202:             /*ldc=*/N,
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 203-243
```cpp
203:             /*outProcess=*/outputProcObj,
204:             /*thread_id=*/task_id,
205:             /*num_threads=*/num_tasks);
206:       } else if (q_scheme == c10::kPerChannelAffine) {
207:         // Process the per channel quantization.
208:         //
209:         // After the uint8 * int8 matrix multiplication is performed, this
210:         // operation does:
211:         //  1) Add in row and column offsets to the rows and columns,
212:         //  respectively.
213:         //  2) Add in the bias term.
214:         fbgemm::ReQuantizeOutput<
215:             ReluFused,
216:             fbgemm::QuantizationGranularity::OUT_CHANNEL,
217:             float>
218:             outputProcObj(
219:                 doNothingObj,
220:                 output_multiplier_float.data(),
221:                 output_zero_point_int32,
222:                 input_zero_point_int32,
223:                 w_zp.data(),
224:                 packA.getRowOffsetBuffer(),
225:                 col_offsets.data(),
226:                 bias_ptr,
227:                 N, /*nCol=*/
228:                 1, /* groups*/
229:                 act_times_w_scale.data());
230:
231:         // Do the GEMM
232:         fbgemm::fbgemmPacked(
233:             /*packA=*/packA,
234:             /*packB=*/*packB,
235:             /*C=*/output_data,
236:             /*C_buffer=*/buffer.data_ptr<int32_t>(),
237:             /*ldc=*/N,
238:             /*outProcess=*/outputProcObj,
239:             /*thread_id=*/task_id,
240:             /*num_threads=*/num_tasks);
241:       }
242:     }
243:   });
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 245-273
```cpp
245:   return output;
246: }
247:
248: at::Tensor PackedLinearWeight::apply(
249:     at::Tensor input,
250:     double output_scale,
251:     int64_t output_zero_point) {
252:   // Allocate output Tensor
253:   auto output = at::_empty_affine_quantized(
254:       {0},
255:       at::device(c10::kCPU).dtype(c10::kQUInt8),
256:       output_scale,
257:       output_zero_point);
258:   apply_impl<false>(input, output_scale, output_zero_point, output);
259:   return output;
260: }
261:
262: at::Tensor PackedLinearWeight::apply_relu(
263:     at::Tensor input,
264:     double output_scale,
265:     int64_t output_zero_point) {
266:   auto output = at::_empty_affine_quantized(
267:       {0},
268:       at::device(c10::kCPU).dtype(c10::kQUInt8),
269:       output_scale,
270:       output_zero_point);
271:   apply_impl<true>(input, output_scale, output_zero_point, output);
272:   return output;
273: }
```
- EN: The main symbol in this range is `apply`, `apply_relu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply`, `apply_relu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 275-304
```cpp
275: at::Tensor& PackedLinearWeight::apply_out(
276:     const at::Tensor& input,
277:     double output_scale,
278:     int64_t output_zero_point,
279:     at::Tensor& output) {
280:   TORCH_CHECK(
281:       (output.device() == c10::kCPU) && (output.dtype() == c10::kQUInt8) &&
282:       (output.q_scale() == output_scale) &&
283:       (output.q_zero_point() == output_zero_point));
284:   return apply_impl<false>(input, output_scale, output_zero_point, output);
285: }
286:
287: at::Tensor& PackedLinearWeight::apply_relu_out(
288:     const at::Tensor& input,
289:     double output_scale,
290:     int64_t output_zero_point,
291:     at::Tensor& output) {
292:   TORCH_CHECK(
293:       (output.device() == c10::kCPU) && (output.dtype() == c10::kQUInt8) &&
294:       (output.q_scale() == output_scale) &&
295:       (output.q_zero_point() == output_zero_point));
296:   return apply_impl<true>(input, output_scale, output_zero_point, output);
297: }
298:
299: at::Tensor PackedLinearWeight::apply_with_input_q_dq_qweight_dq_output_fp32(
300:   at::Tensor input,
301:   double input_scale,
302:   int64_t input_zero_point) {
303:   TORCH_CHECK(!input.is_quantized(), "Input tensor for apply_with_input_q_dq_qweight_dq_output_fp32 is quantized; "
304:   "Expected input tensor in PackedLinearWeight::apply_with_input_q_dq_qweight_dq_output_fp32 to be full precision.");
```
- EN: The main symbol in this range is `apply_out`, `apply_relu_out`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_out`, `apply_relu_out`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 306-334
```cpp
306:   return apply_with_input_q_dq_qweight_dq_output_fp32_impl<false>(input, input_scale, input_zero_point);
307: }
308:
309: at::Tensor PackedLinearWeight::apply_with_input_q_dq_qweight_dq_relu_output_fp32(
310:   at::Tensor input,
311:   double input_scale,
312:   int64_t input_zero_point) {
313:   TORCH_CHECK(!input.is_quantized(), "Input tensor for apply_with_input_q_dq_qweight_dq_output_fp32 is quantized; "
314:   "Expected input tensor in PackedLinearWeight::apply_with_input_q_dq_qweight_dq_output_fp32 to be full precision.");
315:
316:   return apply_with_input_q_dq_qweight_dq_output_fp32_impl<true>(input, input_scale, input_zero_point);
317: }
318:
319:
320: template <bool ReluFused>
321: at::Tensor PackedLinearWeight::apply_with_input_q_dq_qweight_dq_output_fp32_impl(
322:     const at::Tensor& input,
323:     double input_scale,
324:     int64_t input_zero_point) {
325:   TORCH_CHECK(
326:       fbgemm::fbgemmSupportedCPU(), "Your CPU does not support FBGEMM.");
327:
328:   auto input_contig = input.expect_contiguous();
329:   const auto* input_ptr = input_contig->const_data_ptr<float>();
330:
331:   TORCH_CHECK(
332:       input.dim() >= 2,
333:       "The dimension of input tensor should be larger than or equal to 2");
334:   int64_t M = size_to_dim_(input.dim() - 1, input.sizes());
```
- EN: The main symbol in this range is `apply_with_input_q_dq_qweight_dq_relu_output_fp32`, `apply_with_input_q_dq_qweight_dq_output_fp32_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_with_input_q_dq_qweight_dq_relu_output_fp32`, `apply_with_input_q_dq_qweight_dq_output_fp32_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 336-371
```cpp
336:   auto packB = w.get();
337:
338:   int64_t N = static_cast<int64_t>(packB->numCols());
339:   int64_t K = input.sizes()[input.dim() - 1];
340:   TORCH_CHECK(
341:       K == static_cast<int64_t>(packB->numRows()),
342:       "The number of rows in the packB should be equal to K: " +
343:           std::to_string(K));
344:
345:   // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
346:   float input_scale_float = input_scale;
347:   int32_t input_zero_point_int32 = input_zero_point;
348:
349:   TORCH_CHECK(
350:       w_scale.size() == w_zp.size(),
351:       "Weight scales and zero points vectors should have the same size.");
352:
353:   const float* bias_ptr = nullptr;
354:   c10::MaybeOwned<at::Tensor> bias_contig;
355:   if (this->bias_.has_value()) {
356:     auto& bias = this->bias_.value();
357:     bias_contig = bias.expect_contiguous();
358:     TORCH_CHECK(bias_contig->dim() == 1, "bias should be a vector (1D Tensor)");
359:     TORCH_CHECK(
360:         bias_contig->sizes()[0] == N, "bias should have N elements: " + std::to_string(N));
361:     bias_ptr = bias_contig->data_ptr<float>();
362:   }
363:
364:   std::vector<int64_t> out_sizes = input.sizes().vec();
365:   out_sizes.back() = N;
366:   // Allocate output Tensor and a buffer for fbgemmPacked to use
367:   auto output = at::empty(out_sizes, input.options().dtype(at::kFloat));
368:   auto buffer = at::empty_like(
369:       output,
370:       output.options().dtype(at::kInt),
371:       LEGACY_CONTIGUOUS_MEMORY_FORMAT);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 373-407
```cpp
373:   auto output_data = output.data_ptr<float>();
374:
375:   int num_tasks = at::get_num_threads();
376:   at::parallel_for(0, num_tasks, 1, [&](int64_t begin, int64_t end) {
377:     fbgemm::PackAWithQuantRowOffset<uint8_t> packA(
378:         /*trans=*/fbgemm::matrix_op_t::NoTranspose,
379:         /*nRow=*/M,
380:         /*nCol=*/K,
381:         /*smat=*/input_ptr,
382:         /*ld=*/K,
383:         /*pmat=*/nullptr,
384:         /*scale=*/input_scale_float,
385:         /*zero_pt=*/input_zero_point_int32);
386:
387:     fbgemm::DoNothing<float, float> doNothingObj{};
388:     for (const auto task_id : c10::irange(begin, end)) {
389:       if (q_scheme == c10::kPerTensorAffine) {
390:         // Process the per tensor quantization.
391:         //
392:         // After the uint8 * int8 matrix multiplication is performed, this
393:         // operation does:
394:         //  1) Add in row and column offsets to the rows and columns,
395:         //  respectively.
396:         //  2) Add in the bias term.
397:         fbgemm::ReQuantizeForFloat<ReluFused>
398:             outputProcObj(
399:                 doNothingObj,
400:                 input_scale_float,
401:                 w_scale.data(),
402:                 input_zero_point_int32,
403:                 w_zp.data(),
404:                 packA.getRowOffsetBuffer(),
405:                 col_offsets.data(),
406:                 bias_ptr,
407:                 N /* nCol */);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 409-439
```cpp
409:         // Do the GEMM
410:         fbgemm::fbgemmPacked(
411:             /*packA=*/packA,
412:             /*packB=*/*packB,
413:             /*C=*/output_data,
414:             /*C_buffer=*/buffer.data_ptr<int32_t>(),
415:             /*ldc=*/N,
416:             /*outProcess=*/outputProcObj,
417:             /*thread_id=*/task_id,
418:             /*num_threads=*/num_tasks);
419:       } else if (q_scheme == c10::kPerChannelAffine) {
420:         // Process the per channel quantization.
421:         //
422:         // After the uint8 * int8 matrix multiplication is performed, this
423:         // operation does:
424:         //  1) Add in row and column offsets to the rows and columns,
425:         //  respectively.
426:         //  2) Add in the bias term.
427:         fbgemm::ReQuantizeForFloat<
428:             ReluFused,
429:             fbgemm::QuantizationGranularity::OUT_CHANNEL>
430:             outputProcObj(
431:                 doNothingObj,
432:                 input_scale_float,
433:                 w_scale.data(),
434:                 input_zero_point_int32,
435:                 w_zp.data(),
436:                 packA.getRowOffsetBuffer(),
437:                 col_offsets.data(),
438:                 bias_ptr,
439:                 N /* nCol */);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 441-470
```cpp
441:         // Do the GEMM
442:         fbgemm::fbgemmPacked(
443:             /*packA=*/packA,
444:             /*packB=*/*packB,
445:             /*C=*/output_data,
446:             /*C_buffer=*/buffer.data_ptr<int32_t>(),
447:             /*ldc=*/N,
448:             /*outProcess=*/outputProcObj,
449:             /*thread_id=*/task_id,
450:             /*num_threads=*/num_tasks);
451:       }
452:     }
453:   });
454:   return output;
455: }
456:
457: #endif // USE_FBGEMM
458:
459: #ifdef USE_PYTORCH_QNNPACK
460:
461: #ifdef USE_XNNPACK
462: // TODO: add per_channel support in the future when xnnp supports it
463: template <typename scalar_t, bool kReluFused>
464: at::Tensor PackedLinearWeightsQnnp::apply_impl_xnnp(
465:     const at::Tensor& input,
466:     double output_scale,
467:     int64_t output_zero_point) {
468:   using underlying_t = typename scalar_t::underlying;
469:
470:   std::lock_guard<std::mutex> lock(qnnp_mutex_);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_impl_xnnp`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_impl_xnnp`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 472-503
```cpp
472:   const std::string func_name = kReluFused ? "quantized::linear_relu (xnnpack)"
473:                                            : "quantized::linear (xnnpack)";
474:   TORCH_CHECK(
475:       input.dim() >= 2, func_name, ": Input tensor rank should be >= 2.");
476:   TORCH_CHECK(
477:       !per_channel(),
478:       func_name,
479:       ": xnnpack does not currently have per_channel support.");
480:
481:   const auto input_contig = input.contiguous();
482:   const auto input_scale = input_contig.q_scale();
483:
484:   const size_t rows_w = bias_.size(0);
485:   const size_t cols_w = input_contig.size(input_contig.dim() - 1);
486:
487:   auto status = xnn_status_invalid_state;
488:
489:   // Create an operator iff not already created
490:   if (!xnnp_linear_op ||
491:       (!this->input_scale.has_value() ||
492:        this->input_scale.value() != input_scale)) {
493:     // Update the input scale so we may cache the op
494:     this->input_scale = input_scale;
495:
496:     xnn_operator_t xnnp_op = nullptr;
497:
498:     const float* weight_scales_data = w_scales.const_data_ptr<float>();
499:
500:     // prepare weights
501:     underlying_t w_zp = static_cast<underlying_t>(
502:         orig_weight.q_zero_point() +
503:         (std::is_same_v<underlying_t, uint8_t> ? 128 : 0));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 505-547
```cpp
505:    at::Tensor xnnp_weight = at::_empty_affine_quantized(
506:         orig_weight.sizes(),
507:         c10::CppTypeToScalarType<scalar_t>::value,
508:         weight_scales_data[0],
509:         w_zp);
510:
511:     // copy from the original weight and take care of dtype change if necessary
512:     at::native::xnnp_utils::q8_copy_int8_weight_and_add_offset<scalar_t>(
513:         orig_weight, xnnp_weight);
514:
515:     // Original bias was float, so we requantize it here.
516:     at::Tensor qbias = quant_utils::QuantizeBias(false, bias_, orig_weight, input_scale);
517:
518:     // output limits
519:    auto output_min = kReluFused
520:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
521:         ? activationLimits<underlying_t>(output_scale, output_zero_point, Activation::RELU).first
522:         : std::numeric_limits<underlying_t>::min();
523:     auto output_max = kReluFused
524:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
525:         ? activationLimits<underlying_t>(output_scale, output_zero_point, Activation::RELU).second
526:         : std::numeric_limits<underlying_t>::max();
527:
528:     // Create an operator
529:     status = at::native::xnnp_utils::xnnp_create_fully_connected_nc(
530:         cols_w, /* input_channels */
531:         rows_w, /* output_channels */
532:         cols_w, /* input_stride */
533:         rows_w, /* output_stride */
534:         input_contig.q_zero_point(),
535:         input_contig.q_scale(),
536:         w_zp,
537:         weight_scales_data[0],
538:         reinterpret_cast<const underlying_t*>(
539:             xnnp_weight.template data_ptr<scalar_t>()),
540:         reinterpret_cast<int32_t*>(qbias.data_ptr<c10::qint32>()),
541:         output_zero_point,
542:         output_scale,
543:         output_min,
544:         output_max,
545:         0, /* flags */
546:         &xnnp_op);
547:     xnnp_linear_op = xnnpack_operator(xnnp_op);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 549-580
```cpp
549:     TORCH_CHECK(
550:         status == xnn_status_success,
551:         func_name,
552:         ": xnn create operator failed(",
553:         status,
554:         ")");
555:   }
556:
557:   /*
558:    * Allocate output Tensor and a buffer for XNNPACK to use
559:    * The resulting matrix here is 2-D, let's view it with the original
560:    * left hand dimensions of the input. Here are two examples:
561:    * 1. If the input tensor is {M, K}, the output tensor is {M, N}.
562:    * 2. If the input tensor is {b, M, K}, the output tensor is {b, M, N}.
563:    */
564:   std::vector<int64_t> out_sizes = input.sizes().vec();
565:   out_sizes.back() = static_cast<int64_t>(rows_w);
566:   at::Tensor output = at::native::empty_affine_quantized(
567:       out_sizes,
568:       c10::CppTypeToScalarType<scalar_t>::value,
569:       std::nullopt /* layout */,
570:       c10::kCPU,
571:       std::nullopt /* pin_memory */,
572:       output_scale,
573:       output_zero_point,
574:       input.suggest_memory_format());
575:
576:   // calculate batch_size
577:   size_t rows_input = 1;
578:   for (const auto i : c10::irange(input_contig.dim() - 1)) {
579:     rows_input *= input_contig.size(i);
580:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 582-613
```cpp
582:   // Reshape the operator
583:   status = at::native::xnnp_utils::xnnp_reshape_fully_connected_nc(
584:       xnnp_linear_op.get(),
585:       rows_input, /* batch_size */
586:       caffe2::pthreadpool_());
587:
588:   // Setup the operator
589:   status = at::native::xnnp_utils::xnnp_setup_fully_connected_nc(
590:       xnnp_linear_op.get(),
591:       reinterpret_cast<const underlying_t*>(
592:           input_contig.template data_ptr<scalar_t>()),
593:       reinterpret_cast<underlying_t*>(output.template data_ptr<scalar_t>())
594:     );
595:
596:   TORCH_CHECK(
597:       status == xnn_status_success,
598:       func_name,
599:       ": xnn setup operator failed(",
600:       status,
601:       ")");
602:
603:   // Run the operator
604:   status = xnn_run_operator(
605:       xnnp_linear_op.get(), // Linear op
606:       caffe2::pthreadpool_() // threadpool
607:   );
608:   TORCH_CHECK(
609:       status == xnn_status_success,
610:       func_name,
611:       ": xnn run operator failed(",
612:       status,
613:       ")");
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 615-647
```cpp
615:   return output;
616: }
617: #endif // USE_XNNPACK
618:
619: template <bool ReluFused>
620: at::Tensor PackedLinearWeightsQnnp::apply_impl(
621:     at::Tensor input,
622:     double output_scale,
623:     int64_t output_zero_point) {
624:   TORCH_CHECK(
625:       input.dim() >= 2,
626:       "quantized::linear(): Input tensor rank should be >= 2");
627:   TORCH_CHECK(input.scalar_type() == c10::kQUInt8,
628:                 "quantized::linear (qnnpack): Expected input data type ",
629:                 toString(c10::kQUInt8),
630:                 " but got ",
631:                 toString(input.scalar_type()));
632:
633:   auto input_contig = input.contiguous();
634:
635:   // Weight packing is not thread safe
636:   std::lock_guard<std::mutex> lock(qnnp_mutex_);
637:   auto packB = w.get();
638:   size_t rows_w = bias_.size(0);
639:   size_t cols_w = input_contig.size(input_contig.dim() - 1);
640:   auto input_scale = input_contig.q_scale();
641:
642:   if (!this->input_scale.has_value() ||
643:       this->input_scale.value() != input_scale) {
644:     // Get the original weight and adjust it to uint8 from int8
645:     auto weight_contig = orig_weight;
646:     auto bias_fp32 = bias_;
647:     int8_t* w_data = (int8_t*)weight_contig.data_ptr<c10::qint8>();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 649-686
```cpp
649:     float* weight_scales_data = w_scales.data_ptr<float>();
650:     // We calculate requant scale here as the vector holding the requant scale
651:     // is owned by this module. The pointer is then passed to qnnpack backend.
652:     generate_requantization_scales(
653:         // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
654:         w_scales, input_scale, output_scale, requantization_scales);
655:
656:     at::Tensor qnnp_weight = at::_empty_affine_quantized(
657:         weight_contig.sizes(),
658:         at::device(c10::kCPU).dtype(c10::kQUInt8),
659:         weight_scales_data[0],
660:         w_zero_points[0]);
661:     auto* qnnp_w_data = qnnp_weight.data_ptr<c10::quint8>();
662:     auto wt_numel = weight_contig.numel();
663:     for (const auto i : c10::irange(wt_numel)) {
664:       qnnp_w_data[i] = static_cast<c10::quint8>(w_data[i] + 128);
665:     }
666:     // Original bias was float, so we requantize it here.
667:     const bool is_per_channel = orig_weight.qscheme() == at::kPerChannelAffine;
668:     at::Tensor qbias = quant_utils::QuantizeBias(is_per_channel, bias_fp32, weight_contig, input_scale);
669:
670:     // Update the input scale to not pack again.
671:     this->input_scale = input_scale;
672:     w.reset();
673:     w = std::make_unique<qnnpack::PackBMatrix>(
674:         cols_w /* input_channels */,
675:         rows_w /* output_channels */,
676:         w_zero_points.data(),
677:         requantization_scales.data(),
678:         reinterpret_cast<uint8_t*>(qnnp_w_data),
679:         reinterpret_cast<int32_t*>(qbias.data_ptr<c10::qint32>()));
680:     packB = w.get();
681:     if (at::globalContext().releaseWeightsWhenPrepacking()) {
682:       // On mobile, we release the original weight by resetting the intrusive_ptr.
683:       // Calling unpack after this will throw an assertion.
684:       orig_weight.reset();
685:     }
686:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 688-716
```cpp
688:   size_t rows_input = 1;
689:   size_t cols_input = input_contig.size(input_contig.dim() - 1);
690:   for (const auto i : c10::irange(input_contig.dim() -1)) {
691:     rows_input *= input_contig.size(i);
692:   }
693:
694:   TORCH_CHECK(
695:       cols_input == cols_w,
696:       "quantized::linear(): input size does not match weight dimension 1 size: \
697:          got ",
698:       cols_input,
699:       " but expected ",
700:       cols_w);
701:
702:   // Allocate output Tensor and a buffer for QNNPACK to use
703:   // The resulting matrix here is 2-D, let's view it with the original
704:   // left hand dimensions of the input. Here are two examples:
705:   // 1. If the input tensor is {M, K}, the output tensor is {M, N}.
706:   // 2. If the input tensor is {b, M, K}, the output tensor is {b, M, N}.
707:   std::vector<int64_t> out_sizes = input.sizes().vec();
708:   out_sizes.back() = static_cast<long>(rows_w);
709:   at::Tensor output = at::_empty_affine_quantized(
710:       out_sizes,
711:       input.options(),
712:       output_scale,
713:       output_zero_point);
714:
715:   auto output_min = ReluFused
716:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 717-747
```cpp
717:       ? activationLimits<uint8_t>(output_scale, output_zero_point, Activation::RELU)
718:             .first
719:       : std::numeric_limits<uint8_t>::min();
720:   auto output_max = ReluFused
721:       // NOLINTNEXTLINE(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
722:       ? activationLimits<uint8_t>(output_scale, output_zero_point, Activation::RELU)
723:             .second
724:       : std::numeric_limits<uint8_t>::max();
725:   TORCH_INTERNAL_ASSERT(packB != nullptr, "Packed Weights are NULL");
726:   const pytorch_qnnp_status runStatus = qnnpack::qnnpackLinear(
727:       rows_input /* batch_size */,
728:       cols_input /* input_channels */,
729:       rows_w /* output_channels */,
730:       input_contig.q_zero_point(),
731:       w_zero_points.data(),
732:       requantization_scales.data(),
733:       output_zero_point,
734:       output_min,
735:       output_max,
736:       (uint8_t*)input_contig.data_ptr<c10::quint8>(),
737:       cols_input /* input_stride */,
738:       packB->getPackedWeights(),
739:       (uint8_t*)output.data_ptr<c10::quint8>(),
740:       rows_w /* output_stride */,
741:       // TODO (Ashkan): Disabling temporarily.
742:       // Throws a floating point exception with OSS pthreadpool.
743:       caffe2::pthreadpool_() /* threadpool */);
744:
745:   TORCH_INTERNAL_ASSERT(
746:       runStatus == pytorch_qnnp_status_success,
747:       "failed to run QNNPACK Linear operator");
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 749-782
```cpp
749:   return output;
750: }
751:
752: #ifdef USE_XNNPACK
753: static bool can_use_xnnp(c10::ScalarType dtype, bool per_channel) {
754:   if(!at::native::xnnpack::available()) {
755:     return false;
756:   }
757:
758:   bool supported_dtypes = dtype == c10::kQInt8;
759:   bool invalid_config = per_channel; /* xnnp does not currently support
760:                                         per-channel fully connected op */
761:   if (supported_dtypes && invalid_config) {
762:     /* don't want this to fall through to QNNPACK */
763:     TORCH_CHECK(
764:         false,
765:         "quantized::linear (xnnpack): Unsupported config for dtype KQInt8");
766:   }
767:   return supported_dtypes && !invalid_config;
768: }
769: #endif // USE_XNNPACK
770:
771: at::Tensor PackedLinearWeightsQnnp::apply(
772:     at::Tensor input,
773:     double output_scale,
774:     int64_t output_zero_point) {
775: #ifdef USE_XNNPACK
776:   if (can_use_xnnp(input.scalar_type(), per_channel())) {
777:     return apply_impl_xnnp<c10::qint8, false>(
778:         input, output_scale, output_zero_point);
779:   } /* fall through for unsupported types, configs, or shapes */
780: #endif // USE_XNNPACK
781:   return apply_impl<false>(std::move(input), output_scale, output_zero_point);
782: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `can_use_xnnp`, `apply`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `can_use_xnnp`, `apply`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 784-811
```cpp
784: at::Tensor PackedLinearWeightsQnnp::apply_relu(
785:     at::Tensor input,
786:     double output_scale,
787:     int64_t output_zero_point) {
788: #ifdef USE_XNNPACK
789:   if (can_use_xnnp(input.scalar_type(), per_channel())) {
790:     return apply_impl_xnnp<c10::qint8, true>(
791:         input, output_scale, output_zero_point);
792:   } /* fall through for unsupported types, configs, or shapes */
793: #endif // USE_XNNPACK
794:   return apply_impl<true>(std::move(input), output_scale, output_zero_point);
795: }
796:
797: #endif // USE_PYTORCH_QNNPACK
798:
799: #if AT_MKLDNN_ENABLED()
800: template <PostOps post_op>
801: at::Tensor PackedLinearWeightsOnednn::apply_impl(
802:     at::Tensor input,
803:     double output_scale,
804:     int64_t output_zero_point,
805:     torch::List<at::Scalar> post_op_args) {
806:   const int64_t dim = input.dim();
807:   TORCH_CHECK(
808:       dim != 0,
809:       "qlinear (ONEDNN): input dim should be at least 1, but got 0");
810:   TORCH_CHECK(input.scalar_type() == c10::ScalarType::QUInt8 || input.scalar_type() == c10::ScalarType::QInt8,
811:       "qlinear (ONEDNN): data type of input should be QUInt8 or QInt8.");
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_relu`, `apply_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_relu`, `apply_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 813-841
```cpp
813:   auto is_input_qint8 = input.scalar_type() == c10::ScalarType::QInt8;
814:   auto input_contig = input.expect_contiguous();
815:   auto& w = *weight_;
816:   auto K = input.size(dim - 1), M = input.numel() / K, N = w.get_dim(1);
817:   auto input_dims = {M, K};
818:   auto input_data_type = is_input_qint8 ? dnnl::memory::data_type::s8 : dnnl::memory::data_type::u8;
819:   auto input_desc = ideep::tensor::desc(input_dims, input_data_type);
820:   ideep::attr_t op_attr = ideep::attr_t();
821:   if (post_op == Relu) {
822:     op_attr = ideep::attr_t::fuse_relu();
823:   } else if (post_op == LeakyRelu) {
824:     op_attr = ideep::attr_t::fuse_relu(/*scale=*/1.0f, /*alpha=*/post_op_args.get(0).to<double>());
825:   } else if (post_op == Tanh) {
826:     op_attr = ideep::attr_t::fuse_tanh();
827:   }
828:   ideep::tensor x(input_desc, input_contig->data_ptr());
829:   auto dst_dims = {M, N};
830:   double input_scale = input.q_scale();
831:   int64_t input_zero_point = input.q_zero_point();
832:   const ideep::scale_t& src_scales = ideep::scale_t(1, 1.0/input_scale);
833:   const ideep::scale_t& weights_scales = w.get_scale();
834:   // Scales of ONEDNN and PyTorch are reciprocal
835:   const ideep::scale_t& dst_scales = ideep::scale_t(1, 1.0/output_scale);
836:   const ideep::zero_point_t& src_zero_point = ideep::zero_point_t(1, input_zero_point);
837:   const ideep::zero_point_t& dst_zero_point = ideep::zero_point_t(1, output_zero_point);
838:   // Compute: Use ideep::matmul_forward to support asymmetric quantization
839:   // Allocate output Tensor
840:   at::Tensor output = at::_empty_affine_quantized(
841:       dst_dims,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 842-871
```cpp
842:       at::device(c10::kCPU).dtype(is_input_qint8 ? c10::kQInt8 : c10::kQUInt8),
843:       output_scale,
844:       output_zero_point);
845:   if (output.numel() == 0) {
846:     return output;
847:   }
848:   auto output_ideep_data_type = is_input_qint8 ? ideep::tensor::data_type::s8 : ideep::tensor::data_type::u8;
849:   auto ideep_lowp_kind = is_input_qint8 ? ideep::s8s8 : ideep::u8s8;
850:   ideep::tensor y({dst_dims, output_ideep_data_type,
851:                    {output.strides().cbegin(), output.strides().cend()}},
852:                   output.data_ptr());
853:   bool with_bias = bias_.has_value();
854:   if (with_bias) {
855:     // Bias might be modified outside (e.g. by quantization bias correction).
856:     // If so, update the prepacked bias as well.
857:     if (bias_.value().get_data_handle() != orig_bias_.value().data_ptr()) {
858:       bias_.value().init(bias_.value().get_desc(), orig_bias_.value().data_ptr());
859:     }
860:   }
861:   const auto& b = with_bias ? bias_.value() : ideep::tensor();
862:   // Primitive cache is initialized when called for the first time
863:   // and won't be updated afterwards.
864:   int num_threads = at::get_num_threads();
865:   PrimitiveCacheKey cache_key = std::make_tuple(
866:       input_scale, input_zero_point, input_dims, output_scale, output_zero_point, num_threads, /*accum scale*/1.0, /*accum zero point*/0);
867:   c10::call_once(*cache_initialized_flag, [&](){
868:       LinearParams params;
869:       ideep::matmul_forward::prepare</*is_dynamic=*/false>(
870:           params, x, w, b, y,
871:           src_scales, weights_scales, dst_scales,
```
- EN: The main symbol in this range is `outside`, `call_once`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `outside`, `call_once`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 872-901
```cpp
872:           src_zero_point, dst_zero_point, 1.0f, 1.0f, op_attr,
873:           output_ideep_data_type,
874:           ideep_lowp_kind);
875:       get_cache() = LinearPrimitiveCache(cache_key, params);
876:       w = w.reorder_if_differ_in(params.pd.weights_desc());
877:   });
878:   if (get_cache().hit(cache_key)) {
879:     LinearParams& params = get_cache().get_param();
880:     ideep::matmul_forward::compute<false, false>(params, x, w, b, y);
881:   } else {
882:     ideep::matmul_forward::compute(x, w, b, y, src_scales, weights_scales,
883:                                    dst_scales, src_zero_point, dst_zero_point,
884:                                    1.0f, 1.0f, op_attr,
885:                                    output_ideep_data_type,
886:                                    ideep_lowp_kind);
887:   }
888:   auto out_sizes = input.sizes().vec();
889:   out_sizes.back() = N;
890:   if (output.sizes().vec() == out_sizes)
891:     return output;
892:   return output.reshape(out_sizes);
893: }
894:
895: at::Tensor PackedLinearWeightsOnednn::apply(
896:     at::Tensor input,
897:     double output_scale,
898:     int64_t output_zero_point) {
899:   return apply_impl<NoPostOp>(
900:       std::move(input), output_scale, output_zero_point);
901: }
```
- EN: The main symbol in this range is `apply`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `apply`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 903-931
```cpp
903: at::Tensor PackedLinearWeightsOnednn::apply_relu(
904:     at::Tensor input,
905:     double output_scale,
906:     int64_t output_zero_point) {
907:   return apply_impl<Relu>(
908:       std::move(input), output_scale, output_zero_point);
909: }
910:
911: at::Tensor PackedLinearWeightsOnednn:: apply_leaky_relu(
912:     at::Tensor input,
913:     double output_scale,
914:     int64_t output_zero_point,
915:     double negative_slope) {
916:   torch::List<at::Scalar> post_op_args =
917:       {at::Scalar(negative_slope)};
918:   return apply_impl<LeakyRelu>(
919:       std::move(input), output_scale, output_zero_point, post_op_args);
920: }
921:
922: at::Tensor PackedLinearWeightsOnednn:: apply_tanh(
923:     at::Tensor input,
924:     double output_scale,
925:     int64_t output_zero_point) {
926:   return apply_impl<Tanh>(
927:       std::move(input), output_scale, output_zero_point);
928: }
929:
930: static at::Tensor fp8_qlinear_onednn_ref(
931:     at::Tensor input,
```
- EN: The main symbol in this range is `apply_relu`, `apply_leaky_relu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_relu`, `apply_leaky_relu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 932-961
```cpp
932:     double input_scale,
933:     at::Tensor weight, // expect plain weight
934:     at::Tensor weight_scales,
935:     std::optional<at::Tensor> bias, // plain tensor
936:     double output_scale,
937:     std::optional<c10::ScalarType> output_dtype,
938:     std::optional<at::Tensor> other, // extra input for binary post-op
939:     double other_scale,
940:     const std::string_view& binary_post_op, // e.g. "none", "sum", "add"
941:     double binary_alpha,
942:     const std::string_view& unary_post_op, // e.g. "none", "relu"
943:     torch::List<std::optional<at::Scalar>>& unary_post_op_args,
944:     std::string_view& unary_post_op_algorithm) {
945:   TORCH_CHECK(
946:     input.scalar_type() == at::ScalarType::Float8_e4m3fn && weight.scalar_type() == at::ScalarType::Float8_e4m3fn,
947:     "FP8 qlinear: Unexpected dtype of input and weight:", input.scalar_type(), ", ", weight.scalar_type());
948:   const int64_t dim = input.dim();
949:   auto input_contig =
950:       dim == 2 ? input.contiguous() : input.reshape({-1, input.size(dim - 1)}).contiguous();
951:   auto N = weight.size(0);
952:   auto output_size = input.sizes().vec();
953:   output_size[dim - 1] = N;
954:   auto dqx = input_contig.to(at::kFloat) * input_scale;
955:   std::vector<int64_t> w_scales_new_shape(weight.dim(), 1);
956:   w_scales_new_shape[0] = -1;
957:   auto dqw = weight.to(at::kFloat) * weight_scales.reshape(w_scales_new_shape);
958:   auto y_f32 = at::linear(dqx, dqw);
959:   if (bias.has_value()) {
960:       y_f32 += bias.value().to(at::kFloat);
961:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 962-991
```cpp
962:   if (binary_post_op == "none") {
963:     if (unary_post_op == "relu") {
964:       at::relu_(y_f32);
965:     } else if (unary_post_op == "leaky_relu") {
966:       TORCH_CHECK(
967:           unary_post_op_args.size() == 1,
968:           "onednn qlinear: expect one argument for post op leaky_relu but got ", unary_post_op_args.size(), " args");
969:       auto element = unary_post_op_args.get(0);
970:       auto alpha = element.value().to<float>();
971:       at::leaky_relu_(y_f32, alpha);
972:     } else if (unary_post_op == "tanh") {
973:       at::tanh_(y_f32);
974:     } else if (unary_post_op == "gelu") {
975:       TORCH_CHECK(
976:           unary_post_op_algorithm == "none" || unary_post_op_algorithm == "tanh",
977:           "onednn qlinear: algorithm for post op gelu must be none or tanh but got ", unary_post_op_algorithm);
978:       at::gelu_(y_f32, unary_post_op_algorithm);
979:     } else if (unary_post_op == "hardtanh") {
980:       TORCH_CHECK(
981:           unary_post_op_args.size() == 2 &&
982:               unary_post_op_args.get(0).has_value() &&
983:               unary_post_op_args.get(1).has_value(),
984:           "hardtanh is expected to have two scalar input: min_val and max_val");
985:       auto lower_bound_value =
986:           unary_post_op_args.get(0).value().to<float>();
987:       auto upper_bound_value =
988:           unary_post_op_args.get(1).value().to<float>();
989:       at::hardtanh_(y_f32, lower_bound_value, upper_bound_value);
990:     } else if (unary_post_op == "hardswish") {
991:       at::hardswish_(y_f32);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 992-1021
```cpp
 992:     } else if (unary_post_op == "swish") {
 993:       y_f32 = y_f32 * at::sigmoid(y_f32);
 994:     } else {
 995:       TORCH_CHECK(
 996:           unary_post_op == "none",
 997:           "onednn qlinear: unsupported unary post op ", unary_post_op);
 998:     }
 999:   } else if (binary_post_op == "sum") {
1000:     TORCH_CHECK(other.has_value(), "onednn qlinear: the extra input is missing for post op sum");
1001:     auto x1 = other.value();
1002:     TORCH_CHECK(x1.sizes().vec() == output_size);
1003:     auto x1_f32 = x1.to(at::kFloat) * other_scale;
1004:     x1_f32 = x1_f32.view(y_f32.sizes());
1005:     if (unary_post_op == "none") {
1006:       y_f32.add_(x1_f32);
1007:     } else if (unary_post_op == "relu") {
1008:       y_f32.add_(x1_f32).relu_();
1009:     } else {
1010:       TORCH_CHECK(
1011:           false,
1012:           "onednn qlinear: unsupported unary post op ", unary_post_op, " with binary post op sum");
1013:     }
1014:     y_f32.div_(output_scale);
1015:     if (x1.scalar_type() == c10::kFloat8_e4m3fn) {
1016:       // Avoid NaN
1017:       y_f32.clamp_(-FP8E4M3_MAX, FP8E4M3_MAX);
1018:       // Align with oneDNN: convert fp32 to fp8 by fp32 -> fp16 -> fp8
1019:       y_f32 = y_f32.to(at::kHalf);
1020:     }
1021:     x1.copy_(y_f32.to(x1.scalar_type()).view(x1.sizes()));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 1022-1054
```cpp
1022:     return x1;
1023:   } else if (binary_post_op == "add") {
1024:     TORCH_CHECK(other.has_value(), "onednn qlinear: the extra input is missing for post op sum");
1025:     auto x1 = other.value();
1026:     TORCH_CHECK(x1.sizes().vec() == output_size);
1027:     auto x1_f32 = x1.to(at::kFloat) * other_scale;
1028:     x1_f32 = x1_f32.view(y_f32.sizes());
1029:     if (unary_post_op == "none") {
1030:       y_f32.add_(x1_f32);
1031:     } else if (unary_post_op == "relu") {
1032:       y_f32.add_(x1_f32).relu_();
1033:     } else {
1034:       TORCH_CHECK(
1035:           false,
1036:           "onednn qlinear: unsupported unary post op ", unary_post_op, " with binary post op add");
1037:     }
1038:   } else {
1039:     TORCH_CHECK(
1040:         false,
1041:         "onednn qlinear: unsupported binary post op ", binary_post_op);
1042:   }
1043:
1044:   y_f32.div_(output_scale);
1045:   y_f32 = y_f32.view(output_size);
1046:   auto out_dtype = output_dtype.has_value() ? output_dtype.value() : at::kFloat8_e4m3fn;
1047:   if (out_dtype == at::kFloat8_e4m3fn) {
1048:     // Avoid NaN
1049:     y_f32.clamp_(-FP8E4M3_MAX, FP8E4M3_MAX);
1050:     // Align with oneDNN: convert fp32 to fp8 by fp32 -> fp16 -> fp8
1051:     return y_f32.to(at::kHalf).to(out_dtype);
1052:   }
1053:   return y_f32.to(out_dtype);
1054: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 1056-1084
```cpp
1056: static at::Tensor linear_int8_with_onednn_weight(
1057:     at::Tensor input, // int8 CPU Tensor, not QTensor
1058:     double input_scale,
1059:     int64_t input_zero_point,
1060:     at::Tensor onednn_weight, // int8 tensor from MkldnnCPU
1061:     at::Tensor weight_scales,
1062:     at::Tensor weight_zero_points,
1063:     std::optional<at::Tensor> bias, // plain tensor
1064:     double output_scale,
1065:     int64_t output_zero_point,
1066:     std::optional<c10::ScalarType> output_dtype,
1067:     std::optional<at::Tensor> other, // extra input for binary post-op
1068:     double other_scale,
1069:     int64_t other_zero_point,
1070:     const std::string_view& binary_post_op, // e.g. "none", "sum", "add"
1071:     double binary_alpha,
1072:     const std::string_view& unary_post_op, // e.g. "none", "relu"
1073:     torch::List<std::optional<at::Scalar>>& unary_post_op_args,
1074:     std::string_view& unary_post_op_algorithm) {
1075:   using ideep::tensor;
1076:   const int64_t dim = input.dim();
1077:   TORCH_CHECK(input.scalar_type() == c10::ScalarType::Byte || input.scalar_type() == c10::ScalarType::Char || input.scalar_type() == c10::ScalarType::Float8_e4m3fn,
1078:       "qlinear with mkldnn tensor: data type of input should be uint8, int8 or float8_e4m3fn.");
1079:   TORCH_CHECK(onednn_weight.scalar_type() == c10::ScalarType::Char || onednn_weight.scalar_type() == c10::ScalarType::Float8_e4m3fn,
1080:       "qlinear with mkldnn tensor: data type of weight should be int8 or float8_e4m3fn.");
1081:   bool is_fp8 = false;
1082:   if (input.scalar_type() == c10::ScalarType::Float8_e4m3fn || onednn_weight.scalar_type() == c10::ScalarType::Float8_e4m3fn) {
1083:     TORCH_CHECK(
1084:         input.scalar_type() == c10::ScalarType::Float8_e4m3fn && onednn_weight.scalar_type() == c10::ScalarType::Float8_e4m3fn,
```
- EN: The main symbol in this range is `linear_int8_with_onednn_weight`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `linear_int8_with_onednn_weight`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1085-1114
```cpp
1085:         "qlinear with mkldnn tensor: data type of input and weight should be the same for fp8, but got ",
1086:         input.scalar_type(), " and ", onednn_weight.scalar_type());
1087:     is_fp8 = true;
1088:   }
1089:   TORCH_CHECK(
1090:       weight_scales.scalar_type() == c10::ScalarType::Float, "weight scales should be dtype c10::ScalarType::Float.");
1091:   TORCH_CHECK(
1092:       binary_alpha == 1.0f, "onednn qlinear: alpha != 1 for binary post op is not yet supported.");
1093:   bool fp32_output = output_dtype.has_value() && (output_dtype.value() == c10::kFloat);
1094:   bool bf16_output = output_dtype.has_value() && (output_dtype.value() == c10::kBFloat16);
1095:   if (fp32_output || bf16_output) {
1096:     TORCH_CHECK(
1097:         output_scale == 1.0f && output_zero_point == 0, "onednn qlinear: expect scale=1 and zero point=0 for fp32 output");
1098:   }
1099:   if (binary_post_op != "none") {
1100:     /* Supported cases for binary post op:
1101:       +-------------------+--------------+---------------+
1102:       | Extra input dtype | Output dtype | Post op       |
1103:       +-------------------+--------------+---------------+
1104:       | Fp32/bf16         | fp32/bf16    | sum           |
1105:       +-------------------+--------------+---------------+
1106:       | Fp32/bf16         | int8         | add           |
1107:       +-------------------+--------------+---------------+
1108:       | int8              | fp32/bf16    | not supported |
1109:       +-------------------+--------------+---------------+
1110:       | int8              | int8         | sum           |
1111:       +-------------------+--------------+---------------+
1112:     */
1113:     TORCH_CHECK(other.has_value(), "onednn qlinear: the extra input is missing for post op ", binary_post_op);
1114:     if (fp32_output || bf16_output) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1115-1147
```cpp
1115:       TORCH_CHECK(
1116:           other_scale == 1.0f && other_zero_point == 0,
1117:           "onednn qlinear: expect extra input scale = 1.0 and zero point = 0 when output dtype is ", output_dtype.value(),
1118:           ", but got ", other_scale, " and ", other_zero_point, ", respectively"
1119:       );
1120:     }
1121:     if (binary_post_op == "sum") {
1122:       auto expected_dtype = output_dtype.has_value() ? output_dtype.value() : input.scalar_type();
1123:       TORCH_CHECK(
1124:           other.value().scalar_type() == expected_dtype,
1125:           "onednn qlinear: the dtype of extra input for binary post op should be ", expected_dtype,
1126:           " (same as output dtype), but got ", other.value().scalar_type()
1127:       );
1128:     }
1129:   }
1130: #if defined(__powerpc__)
1131:   if (is_fp8) {
1132: #else
1133:   if(is_fp8 && !cpuinfo_has_x86_amx_fp16()) {
1134: #endif
1135:     // Fall back to ref impl on old platforms because not supported
1136:     // Transpose weight to align with behavior in oneDNN
1137:     return fp8_qlinear_onednn_ref(
1138:         input, input_scale, onednn_weight.t(), weight_scales, bias,
1139:         output_scale, output_dtype, other, other_scale,
1140:         binary_post_op, binary_alpha, unary_post_op,
1141:         unary_post_op_args, unary_post_op_algorithm);
1142:   }
1143:
1144:   // If the input has more than two dimensions, we will reshape it to a 2-dimensional form
1145:   // for calculation and subsequently reshape the output back.
1146:   auto input_contig =
1147:       dim == 2 ? input.contiguous() : input.reshape({-1, input.size(dim - 1)}).contiguous();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1149-1177
```cpp
1149:   auto src = at::native::itensor_from_tensor(input_contig);
1150:   int64_t K = input.size(dim - 1), M = input.numel() / K, N = onednn_weight.size(1);
1151:
1152:   auto output_size = input.sizes().vec();
1153:   output_size[dim - 1] = N;
1154:
1155:   bool with_bias = bias.has_value();
1156:
1157:   std::vector<int64_t> src_dims = {M, K};
1158:   std::vector<int64_t> dst_dims = {M, N};
1159:   auto out_dtype = output_dtype.has_value() ? output_dtype.value() : input.scalar_type();
1160:   at::Tensor output = binary_post_op == "sum" ?
1161:       other.value() :
1162:       at::empty(
1163:         dst_dims,
1164:         at::device(c10::kCPU)
1165:             .dtype(out_dtype)
1166:       );
1167:   if (output.numel() == 0) {
1168:     return output;
1169:   }
1170:   tensor dst = at::native::itensor_view_from_dense(output);
1171:   static tensor empty_tensor;
1172:   static tensor::desc empty_tensor_desc;
1173:   tensor src1 = binary_post_op == "add" ?
1174:       at::native::itensor_view_from_dense(other.value().reshape({-1, other.value().size(dim - 1)})) :
1175:       empty_tensor;
1176:
1177:   // Fast path with cache of params
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1178-1207
```cpp
1178:   static const char* env_var = std::getenv(CACHE_ONEDNN_CONTEXT_FLAG);
1179:   static const std::string cache_flag_str = env_var ? std::string(env_var) : "";
1180:   static const bool context_cache_enabled = !cache_flag_str.empty() && cache_flag_str == "1";
1181:   static std::unordered_map<int64_t, QlinearForwardParams> qlinear_forward_params_map;
1182:   int64_t weight_addr = at::native::data_ptr_from_mkldnn(onednn_weight);
1183:   if (context_cache_enabled) {
1184:     auto it = qlinear_forward_params_map.find(weight_addr);
1185:     if (it != qlinear_forward_params_map.end()) {
1186:       auto& params = it->second;
1187:       auto& args = params.args;
1188:       args[DNNL_ARG_SRC] = std::move(src);
1189:       args[DNNL_ARG_DST] = std::move(dst);
1190:       if (binary_post_op == "add") {
1191:         args[DNNL_ARG_ATTR_MULTIPLE_POST_OP(0) | DNNL_ARG_SRC_1] = std::move(src1);
1192:       }
1193:       params.primitive.execute(ideep::stream::default_stream(), args);
1194:       return dim == 2 ? output : output.resize_(output_size);
1195:     }
1196:   }
1197:
1198:   // Regular path
1199:   auto packed_weight = at::native::itensor_from_mkldnn(onednn_weight);
1200:   tensor onednn_bias;
1201:   if (with_bias) {
1202:     at::Tensor bias_val_float = bias.value();
1203:     if (bias_val_float.dim() == 1) {
1204:       auto b_reshape = bias_val_float.reshape({1, bias_val_float.size(0)});
1205:       onednn_bias = at::native::itensor_view_from_dense(b_reshape);
1206:     } else {
1207:       onednn_bias = at::native::itensor_view_from_dense(bias_val_float);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1208-1237
```cpp
1208:     }
1209:   }
1210:   // Create onednn primitive
1211:   auto src_dtype = at::native::get_mkldnn_dtype(input.scalar_type());
1212:   auto src_desc = tensor::desc(src_dims, src_dtype, ideep::format_tag::any);
1213:   auto weights_desc = packed_weight.get_desc();
1214:   auto dst_dtype = dst.get_data_type();
1215:   auto dst_desc = tensor::desc(dst_dims, dst_dtype, ideep::format_tag::any);
1216:   auto bias_desc = with_bias ?
1217:       tensor::desc(onednn_bias.get_dims(), onednn_bias.get_data_type(), ideep::format_tag::any) :
1218:       empty_tensor_desc;
1219:   // Get op attr for primitive
1220:   // Note: output_scale & output_zero_point are for re-quantization of the final output.
1221:   // And other_scale & other_zero_point are for dequantization of other.
1222:   auto other_desc = binary_post_op == "add" ? src1.get_desc() : empty_tensor_desc;
1223:   auto op_attr = onednn_utils::create_attr_by_post_op(
1224:     binary_post_op,
1225:     binary_alpha,
1226:     other_scale,
1227:     other_zero_point,
1228:     other_desc,
1229:     unary_post_op,
1230:     unary_post_op_args,
1231:     unary_post_op_algorithm
1232:   );
1233:   // Avoid NaN if output dtype is fp8
1234:   if (out_dtype == c10::kFloat8_e4m3fn) {
1235:     // To avoid NaN, we need to clamp the intermediate results (in fp32) to [-488, 488]
1236:     // before converting to fp8
1237:     auto post_ops = op_attr.get_post_ops();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 1238-1267
```cpp
1238:     post_ops.append_eltwise(dnnl::algorithm::eltwise_linear, 1.0/output_scale, 0.0);
1239:     post_ops.append_eltwise(dnnl::algorithm::eltwise_clip, -FP8E4M3_MAX, FP8E4M3_MAX);
1240:     op_attr.set_post_ops(post_ops);
1241:     output_scale = 1.0f;
1242:   }
1243:   if (input_scale != 1.0f) {
1244:     op_attr.set_scales_mask(DNNL_ARG_SRC, 0);
1245:   }
1246:   if (input_zero_point != 0) {
1247:     op_attr.set_zero_points_mask(DNNL_ARG_SRC, 0);
1248:   }
1249:   op_attr.set_scales_mask(DNNL_ARG_WEIGHTS, ideep::utils::op_scale_mask(weight_scales.numel()));
1250:   if (output_scale != 1.0f) {
1251:     op_attr.set_scales_mask(DNNL_ARG_DST, 0);
1252:   }
1253:   if (output_zero_point != 0) {
1254:     op_attr.set_zero_points_mask(DNNL_ARG_DST, 0);
1255:   }
1256:   op_attr.set_scratchpad_mode(dnnl::scratchpad_mode::user);
1257:   auto engine = ideep::engine::cpu_engine();
1258:   auto primitive_desc = with_bias ?
1259:       dnnl::matmul::primitive_desc(engine, src_desc, weights_desc, bias_desc, dst_desc, op_attr) :
1260:       dnnl::matmul::primitive_desc(engine, src_desc, weights_desc, dst_desc, op_attr);
1261:   auto primitive = dnnl::matmul(primitive_desc);
1262:
1263:   // Reorder weight if needed
1264:   auto expected_weight = packed_weight.reorder_if_differ_in(primitive_desc.weights_desc());
1265:
1266:   // Prepare args and execute primitive
1267:   tensor scratchpad(primitive_desc.scratchpad_desc());
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1268-1297
```cpp
1268:   ideep::exec_args args;
1269:   args.insert({DNNL_ARG_SRC, src});
1270:   args.insert({DNNL_ARG_WEIGHTS, expected_weight});
1271:   args.insert({DNNL_ARG_DST, dst});
1272:   args.insert({DNNL_ARG_SCRATCHPAD, scratchpad});
1273:   if (with_bias) {
1274:     args.insert({DNNL_ARG_BIAS, onednn_bias});
1275:   }
1276:   tensor src_scales_t = tensor(ideep::scale_t(1, input_scale));
1277:   tensor wei_scales_t = at::native::itensor_from_tensor(weight_scales);
1278:   tensor dst_scales_t = tensor(ideep::scale_t(1, output_scale));
1279:   tensor src_zp_t = tensor(ideep::zero_point_t(1, input_zero_point));
1280:   tensor dst_zp_t = tensor(ideep::zero_point_t(1, output_zero_point));
1281:   if (input_scale != 1.0f) {
1282:     args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_SRC, src_scales_t});
1283:   }
1284:   if (output_scale != 1.0f) {
1285:     args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_DST, dst_scales_t});
1286:   }
1287:   args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_WEIGHTS, wei_scales_t});
1288:   if (input_zero_point != 0) {
1289:     args.insert({DNNL_ARG_ATTR_ZERO_POINTS | DNNL_ARG_SRC, src_zp_t});
1290:   }
1291:   if (output_zero_point != 0) {
1292:     args.insert({DNNL_ARG_ATTR_ZERO_POINTS | DNNL_ARG_DST, dst_zp_t});
1293:   }
1294:   if (binary_post_op == "add") {
1295:     args.insert({DNNL_ARG_ATTR_MULTIPLE_POST_OP(0) | DNNL_ARG_SRC_1, src1});
1296:   }
1297:   primitive.execute(ideep::stream::default_stream(), args);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1298-1331
```cpp
1298:   // Update cache if needed
1299:   if (context_cache_enabled) {
1300:     QlinearForwardParams params;
1301:     params.primitive = primitive;
1302:     params.packed_weight = expected_weight;
1303:     // keep a copy rather than a view of weight scales
1304:     params.weight_scales = tensor(wei_scales_t.get_desc());
1305:     memcpy(params.weight_scales.get_data_handle(), wei_scales_t.get_data_handle(), wei_scales_t.get_desc().get_size());
1306:     params.src_scale = input_scale != 1.0f ? std::make_optional<tensor>(src_scales_t) : std::nullopt;
1307:     params.dst_scale = output_scale != 1.0f ? std::make_optional<tensor>(dst_scales_t) : std::nullopt;
1308:     params.src_zero_point = input_zero_point != 0 ? std::make_optional<tensor>(src_zp_t) : std::nullopt;
1309:     params.dst_zero_point = output_zero_point != 0 ? std::make_optional<tensor>(dst_zp_t) : std::nullopt;
1310:     params.bias = with_bias ? std::make_optional<tensor>(onednn_bias) : std::nullopt;
1311:     params.scratchpad = scratchpad;
1312:     params.init_args();
1313:     qlinear_forward_params_map[weight_addr] = params;
1314:   }
1315:   return dim == 2 ? output : output.resize_(output_size);
1316: }
1317:
1318: #if AT_MKLDNN_ACL_ENABLED()
1319:
1320: template <bool ReluFused>
1321: at::Tensor PackedLinearWeightsACL::apply_impl(
1322:     at::Tensor input,
1323:     double output_scale,
1324:     int64_t output_zero_point) {
1325:   const int64_t dim = input.dim();
1326:   TORCH_CHECK(
1327:       dim != 0, "qlinear (ACL): input dim should be at least 1, but got 0");
1328:   TORCH_CHECK(
1329:       input.scalar_type() == c10::ScalarType::QUInt8 ||
1330:           input.scalar_type() == c10::ScalarType::QInt8,
1331:       "qlinear (ACL): data type of input should be QUInt8 or QInt8.");
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1333-1364
```cpp
1333:   auto input_contig = input.expect_contiguous();
1334:
1335:   int64_t m = input.numel() / k_;
1336:   double input_scale = input.q_scale();
1337:   int64_t input_zero_point = input.q_zero_point();
1338:   auto is_input_qint8 = input.scalar_type() == c10::ScalarType::QInt8;
1339:   auto key = std::make_tuple(
1340:       m,
1341:       ReluFused,
1342:       static_cast<int64_t>(at::get_num_threads()),
1343:       input_scale,
1344:       input_zero_point,
1345:       output_scale,
1346:       output_zero_point,
1347:       is_input_qint8);
1348:
1349:   auto acl_gemm =
1350:       get_acl_quant_matmul<at::native::acl_utils::StaticQuantMatmul>(key);
1351:   if (acl_gemm) {
1352:     acl_gemm->src_q_tensor.allocator()->import_memory(input_contig->data_ptr());
1353:
1354:     auto dst_dims = {m, n_};
1355:     at::Tensor output = at::_empty_affine_quantized(
1356:         dst_dims,
1357:         at::device(c10::kCPU).dtype(
1358:             is_input_qint8 ? c10::kQInt8 : c10::kQUInt8),
1359:         output_scale,
1360:         output_zero_point);
1361:
1362:     if (output.numel() == 0) {
1363:       return output;
1364:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1366-1396
```cpp
1366:     acl_gemm->dst_q_tensor.allocator()->import_memory(output.data_ptr());
1367:
1368:     acl_gemm->gemm.run();
1369:
1370:     acl_gemm->src_q_tensor.allocator()->free();
1371:     acl_gemm->dst_q_tensor.allocator()->free();
1372:
1373:     auto out_sizes = input.sizes().vec();
1374:     out_sizes.back() = n_;
1375:
1376:     if (output.sizes().vec() == out_sizes)
1377:       return output;
1378:     return output.reshape(out_sizes);
1379:   }
1380:   // fallback to oneDNN in the unlikely scinario that ACL's validation fails
1381:   if (ReluFused) {
1382:     return PackedLinearWeightsOnednn::apply_relu(
1383:         input, output_scale, output_zero_point);
1384:   } else {
1385:     return PackedLinearWeightsOnednn::apply(
1386:         input, output_scale, output_zero_point);
1387:   }
1388: }
1389:
1390: at::Tensor PackedLinearWeightsACL::apply(
1391:     at::Tensor input,
1392:     double output_scale,
1393:     int64_t output_zero_point) {
1394:   return apply_impl</*ReluFused=*/false>(
1395:       std::move(input), output_scale, output_zero_point);
1396: }
```
- EN: The main symbol in this range is `apply`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `apply`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1398-1426
```cpp
1398: at::Tensor PackedLinearWeightsACL::apply_relu(
1399:     at::Tensor input,
1400:     double output_scale,
1401:     int64_t output_zero_point) {
1402:   return apply_impl</*ReluFused=*/true>(
1403:       std::move(input), output_scale, output_zero_point);
1404: }
1405:
1406: #endif // AT_MKLDNN_ACL_ENABLED()
1407: #endif // #if AT_MKLDNN_ENABLED()
1408:
1409: namespace at::native {
1410:
1411:   Tensor QLinearOnednn::run_pointwise_tensor(
1412:       Tensor act, // int8 CPU tensor, not QTensor
1413:       Tensor act_scale,
1414:       Tensor act_zero_point,
1415:       Tensor onednn_weight, // int8 tensor from MkldnnCPU
1416:       Tensor weight_scales,
1417:       Tensor weight_zero_points,
1418:       std::optional<Tensor> bias,
1419:       double output_scale,
1420:       int64_t output_zero_point,
1421:       std::optional<c10::ScalarType> output_dtype,
1422:       std::string_view post_op_name,
1423:       torch::List<std::optional<at::Scalar>> post_op_args,
1424:       std::string_view post_op_algorithm) {
1425: #if AT_MKLDNN_ENABLED()
1426:     // act_zero_point.numel() == 0 for symmetric quantization
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `apply_relu`, `run_pointwise_tensor`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `apply_relu`, `run_pointwise_tensor`，它们直接构成本文件的算子逻辑。

### Lines 1427-1456
```cpp
1427:     TORCH_CHECK(act_scale.numel() == 1 && act_zero_point.numel() <= 1,
1428:         "onednn int8 linear: act scale/zp size should be 1/<=1");
1429:     static std::optional<at::Tensor> other = std::nullopt;
1430:     constexpr std::string_view binary_post_op = "none";
1431:     int64_t act_zp = act_zero_point.numel() == 1 ? act_zero_point.item().toLong() : 0;
1432:     return linear_int8_with_onednn_weight(
1433:         act, act_scale.item().toDouble(), act_zp,
1434:         onednn_weight, weight_scales, weight_zero_points,
1435:         bias, output_scale, output_zero_point, output_dtype,
1436:         other, /*other scale*/1.0, /*other zp*/0,
1437:         binary_post_op, /*binary alpha*/1.0,
1438:         post_op_name, post_op_args, post_op_algorithm
1439:     );
1440: #endif
1441:     TORCH_CHECK(false, "Unimplemented (int8 linear with packed weight and bias)");
1442:   }
1443:
1444:   Tensor QLinearOnednn::run_pointwise_binary_tensor(
1445:       Tensor act, // int8 CPU tensor, not QTensor
1446:       Tensor act_scale,
1447:       Tensor act_zero_point,
1448:       Tensor onednn_weight, // int8 tensor from MkldnnCPU
1449:       Tensor weight_scales,
1450:       Tensor weight_zero_points,
1451:       std::optional<at::Tensor> other, // extra input for binary post-op
1452:       std::optional<Tensor> bias,
1453:       double output_scale,
1454:       int64_t output_zero_point,
1455:       std::optional<c10::ScalarType> output_dtype,
1456:       double other_scale,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1457-1489
```cpp
1457:       int64_t other_zero_point,
1458:       std::string_view binary_post_op, // e.g. "none", "sum", "add"
1459:       double binary_alpha,
1460:       std::string_view unary_post_op, // e.g. "none", "relu"
1461:       torch::List<std::optional<at::Scalar>> unary_post_op_args,
1462:       std::string_view unary_post_op_algorithm) {
1463: #if AT_MKLDNN_ENABLED()
1464:     // act_zero_point.numel() == 0 for symmetric quantization
1465:     TORCH_CHECK(act_scale.numel() == 1 && act_zero_point.numel() <= 1,
1466:         "onednn int8 linear: act scale/zp size should be 1/<=1");
1467:     int64_t act_zp = act_zero_point.numel() == 1 ? act_zero_point.item().toLong() : 0;
1468:     return linear_int8_with_onednn_weight(
1469:         act, act_scale.item().toDouble(), act_zp,
1470:         onednn_weight, weight_scales, weight_zero_points,
1471:         bias, output_scale, output_zero_point, output_dtype,
1472:         other, other_scale, other_zero_point,
1473:         binary_post_op, binary_alpha,
1474:         unary_post_op, unary_post_op_args, unary_post_op_algorithm
1475:     );
1476: #endif
1477:     TORCH_CHECK(false, "Unimplemented (int8 linear with packed weight and bias)");
1478:   }
1479:
1480:   Tensor _weight_int4pack_mm_cpu_tensor(
1481:       const Tensor& A,
1482:       const Tensor& B,
1483:       const Tensor& qGroupSize,
1484:       const Tensor& qScaleAndZeros) {
1485:     TORCH_CHECK(qGroupSize.numel() == 1, __func__, ": group size must be a scalar.");
1486:     TORCH_CHECK(qGroupSize.scalar_type() == c10::kLong, __func__, ": group size must be int64.");
1487:     int group_size = qGroupSize.item<int64_t>();
1488:     return at::_weight_int4pack_mm_for_cpu(A, B, group_size, qScaleAndZeros);
1489:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `_weight_int4pack_mm_cpu_tensor`, `_weight_int4pack_mm_for_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `_weight_int4pack_mm_cpu_tensor`, `_weight_int4pack_mm_for_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1492-1519
```cpp
1492: namespace {
1493:
1494: template <bool ReluFused>
1495: class QLinearInt8 final {
1496:  public:
1497:   static at::Tensor run(
1498:       at::Tensor input,
1499:       const c10::intrusive_ptr<LinearPackedParamsBase>& packed_weight,
1500:       double output_scale,
1501:       int64_t output_zero_point) {
1502:     if (ReluFused) {
1503:       return packed_weight->apply_relu(
1504:           std::move(input), output_scale, output_zero_point);
1505:     } else {
1506:       return packed_weight->apply(
1507:           std::move(input), output_scale, output_zero_point);
1508:     }
1509:   }
1510: };
1511:
1512: class QLinearLeakyReluInt8 final {
1513:  public:
1514:   static at::Tensor run(
1515:       at::Tensor input,
1516:       const c10::intrusive_ptr<LinearPackedParamsBase>& packed_weight,
1517:       double output_scale,
1518:       int64_t output_zero_point,
1519:       double negative_slope) {
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `run`, `QLinearInt8`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `run`, `QLinearInt8`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1520-1558
```cpp
1520: #if AT_MKLDNN_ENABLED() || !defined(STRIP_ERROR_MESSAGES)
1521:     auto& ctx = at::globalContext();
1522: #endif
1523: #if AT_MKLDNN_ENABLED()
1524:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
1525:       return dynamic_cast<PackedLinearWeightsOnednn*>(packed_weight.get())->apply_leaky_relu(
1526:           std::move(input), output_scale, output_zero_point, negative_slope);
1527:     }
1528: #endif
1529:     TORCH_CHECK(
1530:         false,
1531:         "Didn't find engine for operation quantized::linear_leaky_relu ",
1532:         toString(ctx.qEngine()));
1533:   }
1534: };
1535:
1536:
1537: class QLinearTanhInt8 final {
1538:  public:
1539:   static at::Tensor run(
1540:       at::Tensor input,
1541:       const c10::intrusive_ptr<LinearPackedParamsBase>& packed_weight,
1542:       double output_scale,
1543:       int64_t output_zero_point) {
1544: #if AT_MKLDNN_ENABLED() || !defined(STRIP_ERROR_MESSAGES)
1545:     auto& ctx = at::globalContext();
1546: #endif
1547: #if AT_MKLDNN_ENABLED()
1548:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
1549:       return dynamic_cast<PackedLinearWeightsOnednn*>(packed_weight.get())->apply_tanh(
1550:           std::move(input), output_scale, output_zero_point);
1551:     }
1552: #endif
1553:     TORCH_CHECK(
1554:         false,
1555:         "Didn't find engine for operation quantized::linear_tanh ",
1556:         toString(ctx.qEngine()));
1557:   }
1558: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run`, `QLinearTanhInt8`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run`, `QLinearTanhInt8`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 1560-1588
```cpp
1560: template <bool ReluFused>
1561: class QLinearInt8FusedQDQ final {
1562:  public:
1563:   static at::Tensor run(
1564:       at::Tensor input,
1565:       double input_scale,
1566:       int64_t input_zero_point,
1567:       const c10::intrusive_ptr<LinearPackedParamsBase>& packed_weight) {
1568:     if (ReluFused) {
1569:       return packed_weight->apply_with_input_q_dq_qweight_dq_relu_output_fp32(
1570:           std::move(input), input_scale, input_zero_point);
1571:     } else {
1572:       return packed_weight->apply_with_input_q_dq_qweight_dq_output_fp32(
1573:           std::move(input), input_scale, input_zero_point);
1574:     }
1575:   }
1576: };
1577:
1578: class QLinearOnednn final {
1579:  public:
1580:   static Tensor run_pointwise(
1581:       Tensor act, // int8 CPU tensor, not QTensor
1582:       double act_scale,
1583:       int64_t act_zero_point,
1584:       Tensor onednn_weight, // int8 tensor from MkldnnCPU
1585:       Tensor weight_scales,
1586:       Tensor weight_zero_points,
1587:       std::optional<Tensor> bias,
1588:       double output_scale,
```
- EN: The main symbol in this range is `run`, `QLinearInt8FusedQDQ`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `run`, `QLinearInt8FusedQDQ`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 1589-1618
```cpp
1589:       int64_t output_zero_point,
1590:       std::optional<c10::ScalarType> output_dtype,
1591:       std::string_view post_op_name,
1592:       torch::List<std::optional<at::Scalar>> post_op_args,
1593:       std::string_view post_op_algorithm) {
1594: #if AT_MKLDNN_ENABLED()
1595:     static std::optional<at::Tensor> other = std::nullopt;
1596:     static const std::string_view binary_post_op = "none";
1597:     return linear_int8_with_onednn_weight(
1598:         act, act_scale, act_zero_point,
1599:         onednn_weight, weight_scales, weight_zero_points,
1600:         bias, output_scale, output_zero_point, output_dtype,
1601:         other, /*other scale*/1.0, /*other zp*/0,
1602:         binary_post_op, /*binary alpha*/1.0,
1603:         post_op_name, post_op_args, post_op_algorithm
1604:     );
1605: #endif
1606:     TORCH_CHECK(false, "Unimplemented (int8 linear with packed weight and bias)");
1607:   }
1608:
1609:   static Tensor run_pointwise_binary(
1610:       Tensor act, // int8 CPU tensor, not QTensor
1611:       double act_scale,
1612:       int64_t act_zero_point,
1613:       Tensor onednn_weight, // int8 tensor from MkldnnCPU
1614:       Tensor weight_scales,
1615:       Tensor weight_zero_points,
1616:       std::optional<at::Tensor> other, // extra input for binary post-op
1617:       std::optional<Tensor> bias,
1618:       double output_scale,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1619-1648
```cpp
1619:       int64_t output_zero_point,
1620:       std::optional<c10::ScalarType> output_dtype,
1621:       double other_scale,
1622:       int64_t other_zero_point,
1623:       std::string_view binary_post_op, // e.g. "none", "sum", "add"
1624:       double binary_alpha,
1625:       std::string_view unary_post_op, // e.g. "none", "relu"
1626:       torch::List<std::optional<at::Scalar>> unary_post_op_args,
1627:       std::string_view unary_post_op_algorithm) {
1628: #if AT_MKLDNN_ENABLED()
1629:     return linear_int8_with_onednn_weight(
1630:         act, act_scale, act_zero_point,
1631:         onednn_weight, weight_scales, weight_zero_points,
1632:         bias, output_scale, output_zero_point, output_dtype,
1633:         other, other_scale, other_zero_point,
1634:         binary_post_op, binary_alpha,
1635:         unary_post_op, unary_post_op_args, unary_post_op_algorithm
1636:     );
1637: #endif
1638:     TORCH_CHECK(false, "Unimplemented (int8 linear with packed weight and bias)");
1639:   }
1640: };
1641:
1642: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
1643:   register_linear_params();
1644:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear"), TORCH_FN(QLinearInt8<false>::run));
1645:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_relu"), TORCH_FN(QLinearInt8<true>::run));
1646:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_leaky_relu"), TORCH_FN(QLinearLeakyReluInt8::run));
1647:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_tanh"), TORCH_FN(QLinearTanhInt8::run));
1648: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1650-1681
```cpp
1650: TORCH_LIBRARY_IMPL(_quantized, QuantizedCPU, m) {
1651:   register_linear_params();
1652:   m.impl(TORCH_SELECTIVE_NAME("_quantized::linear"), TORCH_FN(QLinearInt8<false>::run));
1653: }
1654:
1655: TORCH_LIBRARY_IMPL(quantized, CPU, m) {
1656:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_with_input_q_dq_qweight_dq_output_fp32"), TORCH_FN(QLinearInt8FusedQDQ<false>::run));
1657:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_with_input_q_dq_qweight_dq_relu_output_fp32"), TORCH_FN(QLinearInt8FusedQDQ<true>::run));
1658:   m.impl(TORCH_SELECTIVE_NAME("quantized::int4mm_packed_weight_cpu"), TORCH_FN(at::native::_weight_int4pack_mm_cpu_tensor));
1659: }
1660:
1661: TORCH_LIBRARY_IMPL(onednn, MkldnnCPU, m) {
1662:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise"),
1663:       TORCH_FN(QLinearOnednn::run_pointwise));
1664:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.tensor"),
1665:       TORCH_FN(at::native::QLinearOnednn::run_pointwise_tensor));
1666:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.binary"),
1667:       TORCH_FN(QLinearOnednn::run_pointwise_binary));
1668:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.binary_tensor"),
1669:       TORCH_FN(at::native::QLinearOnednn::run_pointwise_binary_tensor));
1670: }
1671:
1672: TORCH_LIBRARY_IMPL(onednn, CPU, m) {
1673:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise"),
1674:       TORCH_FN(QLinearOnednn::run_pointwise));
1675:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.tensor"),
1676:       TORCH_FN(at::native::QLinearOnednn::run_pointwise_tensor));
1677:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.binary"),
1678:       TORCH_FN(QLinearOnednn::run_pointwise_binary));
1679:   m.impl(TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.binary_tensor"),
1680:       TORCH_FN(at::native::QLinearOnednn::run_pointwise_binary_tensor));
1681: }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 1683-1684
```cpp
1683: } // namespace
1684: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Context.h`, `ATen/Parallel.h`, `ATen/TensorOperators.h`, `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/native/mkldnn/MKLDNNCommon.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/ACLUtils.h`, `ATen/native/quantized/cpu/OnednnUtils.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `caffe2/utils/threadpool/pthreadpool-cpp.h`, `torch/library.h`, `algorithm`, `string`
- Key helper symbols / 关键辅助符号: `parallel_for`, `Scalar`, `ScalarType`, `qnnpack`, `fbgemm`
