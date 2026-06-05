# BinaryOps.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/BinaryOps.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU binary/comparison operator kernels and type-specific branches in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 二元/比较算子 kernel 与类型特化分支。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Context.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/ExpandUtils.h>
 6: #include <torch/library.h>
 7: #include <ATen/quantized/Quantizer.h>
 8: #include <ATen/native/quantized/cpu/ACLUtils.h>
 9: #include <ATen/native/quantized/cpu/BinaryOps.h>
10: #include <ATen/native/quantized/cpu/QuantizedOps.h>
11: #include <ATen/native/quantized/cpu/init_qnnpack.h>
12: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
13: #include <ATen/native/quantized/cpu/XnnpackUtils.h>
14: #include <caffe2/utils/threadpool/pthreadpool-cpp.h>
15:
16: #ifndef AT_PER_OPERATOR_HEADERS
17: #include <ATen/Functions.h>
18: #include <ATen/NativeFunctions.h>
19: #else
20: #include <ATen/ops/_empty_affine_quantized.h>
21: #include <ATen/ops/_empty_affine_quantized_native.h>
22: #include <ATen/ops/empty_like.h>
23: #include <ATen/ops/relu_native.h>
24: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 26-48
```cpp
26: #include <algorithm>
27: #include <utility>
28:
29: namespace at::native {
30:
31: DEFINE_DISPATCH(qadd_relu_stub);
32: DEFINE_DISPATCH(qadd_stub);
33: DEFINE_DISPATCH(qadd_scalar_relu_stub);
34: DEFINE_DISPATCH(qadd_scalar_stub);
35:
36: namespace {
37:
38: inline void check_inputs(const Tensor& qa, const Tensor& qb) {
39:   TORCH_CHECK(
40:       qa.qscheme() == kPerTensorAffine,
41:       "Only per tensor quantization is supported in Add.");
42:   TORCH_CHECK(
43:       qa.qscheme() == qb.qscheme(),
44:       "Both inputs to Add must have the same quantization scheme.");
45:   TORCH_CHECK(
46:       qa.scalar_type() == qb.scalar_type(),
47:       "Add operands should have same data type.");
48: }
```
- EN: This range pulls in required headers, including `algorithm`, `utility`. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `check_inputs`, which contributes directly to this file's operator logic.
- CN: 这一段引入了所需头文件，例如 `algorithm`, `utility`。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `check_inputs`，它们直接构成本文件的算子逻辑。

### Lines 50-68
```cpp
50: // Note: out is assumed to be the same size as self and other.
51: // Note: Addition is only supported when self, other, out are of the same dtype.
52: template <bool ReLUFused = false>
53: Tensor _add_out(Tensor& out, const Tensor& self, const Tensor& other) {
54:   if (ReLUFused) {
55:     qadd_relu_stub(self.device().type(), out, self, other);
56:   } else {
57:     qadd_stub(self.device().type(), out, self, other);
58:   }
59:   return out;
60: }
61:
62: template <bool ReLUFused = false>
63: Tensor _add_scalar_out(Tensor& out, const Tensor& self, const Scalar& other) {
64:   TORCH_CHECK(
65:       self.qscheme() == kPerTensorAffine,
66:       "Only per tensor affine is supported for now!!");
67:   // To implement tensor-scalar addition in quantized space, we simply
68:   // adjust the quantization parameters based on the following rules:
```
- EN: The main symbol in this range is `_add_out`, `_add_scalar_out`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `_add_out`, `_add_scalar_out`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 69-95
```cpp
69:   //
70:   // Let s = scale, z = zero point, c = other.toFloat(), c_q = round(c/s)
71:   // q_min = lowest representable value of scalar type
72:   // q_max = highest representable value of scalar type
73:   //
74:   // Let s' = the calculated scale or the output
75:   // z' = the calculated zero-point for the output
76:   //
77:   // If q_min > z - c_q
78:   //   s' = [(q_max - (z - c_q)]/[q_max - q_min] * s
79:   //   z' = q_min
80:   //   Xq' = at::requantize_from_int(Xq - z + c_q, s/s', z')
81:   // If q_max < z - c_q
82:   //   s' = [z - c_q -q_min]/[q_max - q_min] * s
83:   //   z' = q_max
84:   //   Xq' = at::requantize_from_int(Xq - z + c_q, s/s', z')
85:   // Else
86:   //   s' = s
87:   //   z' = z - c_q
88:
89:   AT_DISPATCH_QINT_TYPES(self.scalar_type(), "qadd_scalar", [&]() {
90:     double s = self.q_scale();
91:     int64_t z = self.q_zero_point();
92:     double c = other.toDouble();
93:     // NOLINTNEXTLINE(bugprone-signed-char-misuse)
94:     int64_t q_min = std::numeric_limits<underlying_t>::min();
95:     int64_t q_max = std::numeric_limits<underlying_t>::max();
```
- EN: The main symbol in this range is `toFloat`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `toFloat`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 97-115
```cpp
 97:     int64_t c_q = std::nearbyint(c / s);
 98:
 99:     double s_prime;
100:     int64_t z_prime;
101:
102:     if (q_min > z - c_q) {
103:       s_prime = (((double)q_max - (z - c_q))) / ((double)q_max - q_min) * s;
104:       z_prime = q_min;
105:       set_quantizer_(out, make_per_tensor_affine_quantizer(
106:           s_prime, z_prime, self.scalar_type()));
107:       if (ReLUFused) {
108:         qadd_scalar_relu_stub(self.device().type(), out, self, c_q);
109:       } else {
110:         qadd_scalar_stub(self.device().type(), out, self, c_q);
111:       }
112:     } else if (q_max < z - c_q) {
113:       s_prime = ((double)(z - c_q) - q_min) / ((double)q_max - q_min) * s;
114:       z_prime = q_max;
115:       set_quantizer_(out, make_per_tensor_affine_quantizer(
```
- EN: The main symbol in this range is `qadd_scalar_relu_stub`, `qadd_scalar_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `qadd_scalar_relu_stub`, `qadd_scalar_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 116-134
```cpp
116:           s_prime, z_prime, self.scalar_type()));
117:       if (ReLUFused) {
118:         qadd_scalar_relu_stub(self.device().type(), out, self, c_q);
119:       } else {
120:         qadd_scalar_stub(self.device().type(), out, self, c_q);
121:       }
122:     } else {
123:       s_prime = s;
124:       z_prime = z - c_q;
125:       out.copy_(self);
126:       set_quantizer_(out, make_per_tensor_affine_quantizer(
127:           s_prime, z_prime, self.scalar_type()));
128:       if (ReLUFused) {
129:         at::native::relu_quantized_cpu_(out);
130:       }
131:     }
132:   });
133:   return out;
134: }
```
- EN: The main symbol in this range is `qadd_scalar_relu_stub`, `qadd_scalar_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qadd_scalar_relu_stub`, `qadd_scalar_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 137-154
```cpp
137: #ifdef USE_PYTORCH_QNNPACK
138: template <bool ReLUFused = false>
139: Tensor qnnpack_add(Tensor qa, Tensor qb, double scale, int64_t zero_point) {
140:   TORCH_CHECK(qa.ndimension() > 0, "qnnpack_add(): Got empty input tensor.");
141:   TORCH_CHECK(qa.scalar_type() == c10::kQUInt8 && qb.scalar_type() == c10::kQUInt8,
142:                 "qnnpack_add(): Expected both input data types to be ",
143:                 toString(c10::kQUInt8),
144:                 " but got ",
145:                 toString(qa.scalar_type()),
146:                 " and ",
147:                 toString(qb.scalar_type()));
148:   Tensor qa_contig = qa.contiguous(qa.suggest_memory_format());
149:   // Reason for use qa's memory format for qb is that for the underlying
150:   // kernel can flatten all the dims and iterate over both the tensors.
151:   // In most cases, both qa and qb are in same memory format.
152:   // When they are not there is a copy overhead to make it contiguous
153:   // in qa's memory format.
154:   Tensor qb_contig = qb.contiguous(qa.suggest_memory_format());
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `qnnpack_add`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `qnnpack_add`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 156-175
```cpp
156:   const auto a_zero_point = qa_contig.q_zero_point();
157:   const auto b_zero_point = qb_contig.q_zero_point();
158:   const auto a_scale = qa_contig.q_scale();
159:   const auto b_scale = qb_contig.q_scale();
160:
161:   Tensor qy = at::native::empty_affine_quantized(
162:       qa_contig.sizes(),
163:       kQUInt8,
164:       std::nullopt /* layout */,
165:       kCPU,
166:       std::nullopt /* pin_memory */,
167:       scale,
168:       zero_point,
169:       qa.suggest_memory_format());
170:
171:   if (qa_contig.size(0) == 0) {
172:     return qy;
173:   }
174:
175:   initQNNPACK();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 177-202
```cpp
177:   pytorch_qnnp_operator_t qnnpack_operator{nullptr};
178:
179:   size_t num_elems = qa_contig.numel() / qa_contig.size(0);
180:   auto output_min = ReLUFused
181:       // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
182:       ? activationLimits<uint8_t>(scale, zero_point, Activation::RELU)
183:             .first
184:       : std::numeric_limits<uint8_t>::min();
185:   auto output_max = ReLUFused
186:       // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
187:       ? activationLimits<uint8_t>(scale, zero_point, Activation::RELU)
188:             .second
189:       : std::numeric_limits<uint8_t>::max();
190:   const pytorch_qnnp_status createStatus = pytorch_qnnp_create_add_nc_q8(
191:       num_elems /* input size */,
192:       a_zero_point /* a zero_point */,
193:       a_scale /* a scale */,
194:       b_zero_point /* b zero_point */,
195:       b_scale /* b scale */,
196:       static_cast<uint8_t>(zero_point) /* sum zero_point */,
197:       // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
198:       scale /* sum scale */,
199:       output_min /* output min */,
200:       output_max /* output max */,
201:       0 /* flags */,
202:       &qnnpack_operator);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 204-222
```cpp
204:   TORCH_INTERNAL_ASSERT(
205:       createStatus == pytorch_qnnp_status_success,
206:       "failed to create QNNPACK Add operator");
207:
208:   std::unique_ptr<pytorch_qnnp_operator, QnnpackOperatorDeleter>
209:       qnnpack_uniq_ptr(qnnpack_operator);
210:
211:   const pytorch_qnnp_status setupStatus = pytorch_qnnp_setup_add_nc_q8(
212:       qnnpack_operator /* add op */,
213:       qa_contig.size(0) /* batch size */,
214:       (uint8_t*)qa_contig.data_ptr<c10::quint8>() /* a data */,
215:       num_elems /* A stride */,
216:       (uint8_t*)qb_contig.data_ptr<c10::quint8>() /* b data */,
217:       num_elems /* B stride */,
218:       (uint8_t*)qy.data_ptr<c10::quint8>() /* output data */,
219:       num_elems /* sum stride */);
220:   TORCH_INTERNAL_ASSERT(
221:       setupStatus == pytorch_qnnp_status_success,
222:       "failed to setup QNNPACK Add operator");
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 224-242
```cpp
224:   pthreadpool_t threadpool = caffe2::pthreadpool_();
225:   const pytorch_qnnp_status runStatus =
226:       pytorch_qnnp_run_operator(qnnpack_operator, threadpool);
227:
228:   TORCH_INTERNAL_ASSERT(
229:       runStatus == pytorch_qnnp_status_success,
230:       "failed to run QNNPACK Add operator");
231:
232:   return qy;
233: }
234: #endif // USE_PYTORCH_QNNPACK
235:
236: #ifdef USE_XNNPACK
237: C10_ALWAYS_INLINE
238: enum xnn_status xnnp_define_q_tensor(const Tensor& tensor, MemoryFormat format, uint32_t& id, xnn_subgraph_t subgraph_ptr, uint32_t external_id, uint32_t flags){
239:   Tensor contig_tensor = tensor.contiguous(format);
240:   const auto tensor_shape = xnnp_utils::get_mem_format_aware_shape(contig_tensor);
241:   const int32_t zero_point = static_cast<int32_t>(contig_tensor.q_zero_point());
242:   const float scale = static_cast<float>(contig_tensor.q_scale());
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `xnnp_define_q_tensor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `xnnp_define_q_tensor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 244-262
```cpp
244:   return xnn_define_quantized_tensor_value(
245:     subgraph_ptr,
246:     xnn_datatype_qint8,
247:     zero_point,
248:     scale,
249:     tensor.ndimension(),
250:     tensor_shape.data(),
251:     nullptr,
252:     external_id,
253:     flags,
254:     &id);
255: }
256:
257: template <typename scalar_t, bool ReLUFused = false>
258: Tensor xnnp_add(Tensor qa, Tensor qb, double scale, int64_t zero_point) {
259:   TORCH_CHECK(qa.ndimension() > 0, __func__, ": Got empty input tensor.");
260:   TORCH_CHECK(at::native::xnnpack::available(), __func__, ": XNNPACK is not available")
261:
262:   // using qa memory format for qb to allow xnnpack kernel to flatten all the
```
- EN: The main symbol in this range is `xnnp_add`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `xnnp_add`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 263-286
```cpp
263:   // dims
264:   auto qa_mem_format = qa.suggest_memory_format();
265:   Tensor qa_contig = qa.contiguous(qa_mem_format);
266:   Tensor qb_contig = qb.contiguous(qa_mem_format);
267:   Tensor qy = at::native::empty_affine_quantized(
268:       at::infer_size_dimvector(qa_contig.sizes(), qb_contig.sizes()),
269:       qa.scalar_type(),
270:       std::nullopt /* layout */,
271:       kCPU,
272:       std::nullopt /* pin_memory */,
273:       scale,
274:       zero_point,
275:       qa_mem_format);
276:
277:   if (qa_contig.size(0) == 0) {
278:     return qy;
279:   }
280:
281:
282:   auto output_max = std::numeric_limits<float>::infinity();
283:   auto output_min = -std::numeric_limits<float>::infinity();
284:   if (ReLUFused) {
285:     output_min = 0;
286:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 288-313
```cpp
288:   // Create XNNPACK Subgraph
289:   xnn_subgraph_t subgraph_ptr = nullptr;
290:   auto status = xnn_create_subgraph(
291:     /*external_value_ids=*/3,
292:     /*flags=*/0,
293:     &subgraph_ptr);
294:   TORCH_CHECK(
295:       status == xnn_status_success,
296:       __func__, ": xnn create subgraph failed(", status,")!");
297:   std::unique_ptr<xnn_subgraph, decltype(&xnn_delete_subgraph)> subgraph(
298:       subgraph_ptr, &xnn_delete_subgraph);
299:
300:   uint32_t input0_id = XNN_INVALID_VALUE_ID, input1_id = XNN_INVALID_VALUE_ID, output_id = XNN_INVALID_VALUE_ID;
301:
302:   // Defining the quantized input 0
303:   status = xnnp_define_q_tensor(
304:     qa,
305:     qa_mem_format,
306:     input0_id,
307:     subgraph_ptr,
308:     0,
309:     XNN_VALUE_FLAG_EXTERNAL_INPUT
310:   );
311:   TORCH_CHECK(
312:       status == xnn_status_success && input0_id != XNN_INVALID_VALUE_ID,
313:       __func__, ": xnn define input 0 failed(", status,")!");
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 315-339
```cpp
315:   // Defining the quantized input 1
316:   status = xnnp_define_q_tensor(
317:     qb,
318:     qa_mem_format,
319:     input1_id,
320:     subgraph_ptr,
321:     1,
322:     XNN_VALUE_FLAG_EXTERNAL_INPUT
323:   );
324:   TORCH_CHECK(
325:       status == xnn_status_success && input1_id != XNN_INVALID_VALUE_ID,
326:       __func__, ": xnn define input 1 failed(", status,")!");
327:
328:   // Defining the quantized output
329:   status = xnnp_define_q_tensor(
330:     qy,
331:     qa_mem_format,
332:     output_id,
333:     subgraph_ptr,
334:     2,
335:     XNN_VALUE_FLAG_EXTERNAL_OUTPUT
336:   );
337:   TORCH_CHECK(
338:       status == xnn_status_success && output_id != XNN_INVALID_VALUE_ID,
339:       __func__, ": xnn define output failed(", status,")!");
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 341-364
```cpp
341:   const struct xnn_binary_params binary_params = {output_min, output_max};
342:   status = xnn_define_binary(
343:     subgraph_ptr,
344:     xnn_binary_add,
345:     &binary_params,
346:     input0_id,
347:     input1_id,
348:     output_id,
349:     0);
350:   TORCH_CHECK(
351:       status == xnn_status_success,
352:       __func__, ": xnn define binary add failed(", status,")!");
353:
354:   // create runtime
355:   xnn_runtime_t runtime_ptr = nullptr;
356:   status = xnn_create_runtime_v2(subgraph_ptr, caffe2::pthreadpool_(), 0, &runtime_ptr);
357:   TORCH_CHECK(
358:       status == xnn_status_success,
359:       __func__, ": xnn create runtime failed(", status,")!");
360:   TORCH_CHECK(
361:       runtime_ptr != nullptr,
362:       __func__, ": xnn create runtime failed because runtime_ptr is null");
363:   std::unique_ptr<xnn_runtime, decltype(&xnn_delete_runtime)> auto_runtime(
364:       runtime_ptr, &xnn_delete_runtime);
```
- EN: The main symbol in this range is `xnn_binary_params`, which contributes directly to this file's operator logic. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `xnn_binary_params`，它们直接构成本文件的算子逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 366-385
```cpp
366:   std::array<xnn_external_value, 3> external = {
367:     xnn_external_value{input0_id, reinterpret_cast<void*>(qa_contig.data_ptr<scalar_t>())},
368:     xnn_external_value{input1_id, reinterpret_cast<void*>(qb_contig.data_ptr<scalar_t>())},
369:     xnn_external_value{output_id, reinterpret_cast<void*>(qy.data_ptr<scalar_t>())}};
370:
371:   status = xnn_setup_runtime(
372:     runtime_ptr,
373:     external.size(),
374:     external.data());
375:   TORCH_CHECK(
376:       status == xnn_status_success,
377:       __func__, ": xnn setup runtime failed(", status,")!");
378:   status = xnn_invoke_runtime(runtime_ptr);
379:   TORCH_CHECK(
380:       status == xnn_status_success,
381:       __func__, ": xnn invoke runtime failed(", status,")!");
382:
383:   return qy;
384: }
385: #endif // USE_XNNPACK
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 387-408
```cpp
387: #if AT_MKLDNN_ACL_ENABLED()
388: Tensor acl_qadd(Tensor qa, Tensor qb, double scale, int64_t zero_point) {
389:   TORCH_CHECK(
390:       qa.qscheme() == kPerTensorAffine || qa.qscheme() == kPerTensorSymmetric,
391:       "Only per tensor quantization is supported in ACL quantized add.");
392:
393:   Tensor qa_contig = qa.contiguous(qa.suggest_memory_format());
394:   Tensor qb_contig = qb.contiguous(qa.suggest_memory_format());
395:   auto qa_mem_format = qa_contig.suggest_memory_format();
396:   Tensor dst = at::native::empty_affine_quantized(
397:       at::infer_size_dimvector(qa_contig.sizes(), qb_contig.sizes()),
398:       qa_contig.scalar_type(),
399:       std::nullopt /* layout */,
400:       kCPU,
401:       std::nullopt /* pin_memory */,
402:       scale,
403:       zero_point,
404:       qa_mem_format);
405:
406:   if (qb_contig.size(0) == 0) {
407:     return dst;
408:   }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 410-428
```cpp
410:   auto input_dims = qa_contig.sizes().vec();
411:   auto acl_dtype = dst.scalar_type() == kQInt8
412:       ? arm_compute::DataType::QASYMM8_SIGNED
413:       : arm_compute::DataType::QASYMM8;
414:   auto acl_add = std::make_shared<acl_utils::QuantAdd>(
415:       acl_dtype,
416:       input_dims,
417:       qa_contig.q_scale(),
418:       qa_contig.q_zero_point(),
419:       qb_contig.q_scale(),
420:       qb_contig.q_zero_point(),
421:       dst.q_scale(),
422:       dst.q_zero_point());
423:
424:   auto status = acl_add->validate();
425:   TORCH_CHECK(
426:       status.error_code() == arm_compute::ErrorCode::OK,
427:       "Arm Compute Library's Quantized Matmul Validation Failed: " +
428:           status.error_description());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 430-450
```cpp
430:   acl_add->configure();
431:
432:   acl_add->qa_tensor.allocator()->import_memory(qa_contig.data_ptr());
433:   acl_add->qb_tensor.allocator()->import_memory(qb_contig.data_ptr());
434:   acl_add->qdst_tensor.allocator()->import_memory(dst.data_ptr());
435:
436:   acl_add->q_add.run();
437:
438:   // this will not free memory, it will just tell ACL that we're no longer
439:   // using the pointer
440:   acl_add->qa_tensor.allocator()->free();
441:   acl_add->qb_tensor.allocator()->free();
442:   acl_add->qdst_tensor.allocator()->free();
443:
444:   return dst;
445: }
446: #endif // AT_MKLDNN_ACL_ENABLED()
447:
448: template <bool ReLUFused = false>
449: Tensor qadd(Tensor qa, Tensor qb, double scale, int64_t zero_point) {
450:   check_inputs(qa, qb);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 452-477
```cpp
452:   if (at::globalContext().qEngine() == at::QEngine::QNNPACK) {
453:     TORCH_CHECK(
454:         qa.scalar_type() == qb.scalar_type(),
455:         "Both inputs to qadd must have same type");
456:
457: #ifdef USE_XNNPACK
458:     if (qa.scalar_type() == kQInt8) {
459:           return xnnp_add<c10::qint8, ReLUFused>(qa, qb, scale, zero_point);
460:     }
461: #endif // USE_XNNPACK
462:
463: #ifdef USE_PYTORCH_QNNPACK
464:     if(qa.sizes() == qb.sizes() && /* qnnpack does not support broadcasting */
465:       qa.scalar_type() == kQUInt8) {
466:     return qnnpack_add<ReLUFused>(qa, qb, scale, zero_point);
467:     }
468: #endif // USE_PYTORCH_QNNPACK
469:   }
470:
471: #if AT_MKLDNN_ACL_ENABLED()
472:   if (!ReLUFused && qa.ndimension() > 0 && qa.sizes() == qb.sizes() &&
473:       qa.scalar_type() == qb.scalar_type() &&
474:       (qa.scalar_type() == kQInt8 || qa.scalar_type() == kQUInt8)) {
475:     return acl_qadd(qa, qb, scale, zero_point);
476:   }
477: #endif // AT_MKLDNN_ACL_ENABLED()
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 479-505
```cpp
479:   auto qc = at::_empty_affine_quantized(
480:       qa.sizes(),
481:       at::device(kCPU)
482:          .dtype(qa.scalar_type())
483:          .memory_format(qa.suggest_memory_format()),
484:       scale,
485:       zero_point,
486:       std::nullopt);
487:   return _add_out<ReLUFused>(qc, qa, qb);
488: }
489:
490: template <bool ReLUFused = false>
491: Tensor qadd_out(Tensor qa, Tensor qb, Tensor out) {
492:   check_inputs(qa, qb);
493:   check_inputs(qa, out);
494:   return _add_out<ReLUFused>(out, qa, qb);
495: }
496:
497:
498: template <bool ReLUFused = false>
499: Tensor qadd_scalar(Tensor qa, const Scalar& b) {
500:   TORCH_CHECK(qa.qscheme() == kPerTensorAffine ||
501:               qa.qscheme() == kPerTensorSymmetric,
502:               "Only per tensor quantization is supported in Add.");
503:   auto qc = at::empty_like(qa, qa.suggest_memory_format());
504:   return _add_scalar_out<ReLUFused>(qc, qa, b);
505: }
```
- EN: The main symbol in this range is `qadd_out`, `qadd_scalar`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qadd_out`, `qadd_scalar`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 507-528
```cpp
507: template <bool ReLUFused = false>
508: Tensor qadd_scalar2(Scalar b, Tensor qa) {
509:   TORCH_CHECK(qa.qscheme() == kPerTensorAffine ||
510:               qa.qscheme() == kPerTensorSymmetric,
511:               "Only per tensor quantization is supported in Add.");
512:   auto qc = at::empty_like(qa, qa.suggest_memory_format());
513:   return _add_scalar_out<ReLUFused>(qc, qa, b);
514: }
515:
516: template <bool ReLUFused = false>
517: Tensor qadd_scalar_out(Tensor qa, const Scalar& b, Tensor out) {
518:   check_inputs(qa, out);
519:   return _add_scalar_out<ReLUFused>(out, qa, b);
520: }
521:
522: // `torch.jit.trace` will trace Scalar as Tensor
523: // This can be removed after broadcast is supported and
524: // all variations of `quantized::add` is merged into `quantized::add`
525: template <bool ReLUFused = false>
526: Tensor qadd_scalar_tensor(Tensor qa, Tensor b) {
527:   return qadd_scalar(std::move(qa), b.item());
528: }
```
- EN: The main symbol in this range is `qadd_scalar2`, `qadd_scalar_out`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qadd_scalar2`, `qadd_scalar_out`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 530-548
```cpp
530: // `torch.jit.trace` will trace Scalar as Tensor
531: // This can be removed after broadcast is supported and
532: // all variations of `quantized::add` is merged into `quantized::add`
533: template <bool ReLUFused = false>
534: Tensor qadd_scalar_tensor_out(Tensor qa, Tensor b, Tensor out) {
535:   return qadd_scalar_out(std::move(qa), b.item(), std::move(out));
536: }
537:
538: DEFINE_DISPATCH(qadd_tensor_cpu_stub);
539: DEFINE_DISPATCH(qadd_relu_tensor_cpu_stub);
540: template <bool ReLUFused = false>
541: Tensor int8_add_tensor_onednn(
542:     const Tensor& self, double self_scale, int64_t self_zero_point,
543:     const Tensor& other, double other_scale, int64_t other_zero_point,
544:     double output_scale, int64_t output_zero_point, c10::ScalarType output_dtype) {
545:   // Both inputs should have the same shape and both in uint8 dtype.
546:   // If output_dtype is uint8, output is requantized with output scale/zero point.
547:   // Otherwise, output scale should be 1 and zero point 0.
548:   TORCH_CHECK(self.sizes() == other.sizes(),
```
- EN: The main symbol in this range is `qadd_scalar_tensor_out`, `int8_add_tensor_onednn`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qadd_scalar_tensor_out`, `int8_add_tensor_onednn`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 549-575
```cpp
549:               "Quantized add operands should have the same size.");
550:   TORCH_CHECK(self.scalar_type() == at::kByte && other.scalar_type() == at::kByte,
551:               "Quantized add operands should be of type uint8, but got ",
552:               self.scalar_type(), " and ", other.scalar_type());
553:   TORCH_CHECK(output_dtype == at::kByte || output_dtype == at::kFloat || output_dtype == at::kBFloat16 || output_dtype == at::kHalf,
554:               "Quantized add output should be of type uint8, float, bfloat16 or float16, but got ",
555:               output_dtype);
556:   if (output_dtype != at::kByte) {
557:     TORCH_CHECK(output_scale == 1.0 && output_zero_point == 0,
558:                 "Quantized add output scale and zero point should be 1 and 0 for "
559:                 "output_dtype ", output_dtype, ", but got scale = ",
560:                 output_scale, " and zero point = ", output_zero_point);
561:   }
562:   at::Tensor out = at::empty_like(self, self.options().dtype(output_dtype));
563:
564:
565:   if constexpr (ReLUFused) {
566:     qadd_relu_tensor_cpu_stub(
567:         self.device().type(), out, self, self_scale, self_zero_point,
568:         other, other_scale, other_zero_point,
569:         output_scale, output_zero_point);
570:   } else {
571:     qadd_tensor_cpu_stub(
572:         self.device().type(), out, self, self_scale, self_zero_point,
573:         other, other_scale, other_zero_point,
574:         output_scale, output_zero_point);
575:   }
```
- EN: The main symbol in this range is `constexpr`, `qadd_relu_tensor_cpu_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`, `qadd_relu_tensor_cpu_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 577-602
```cpp
577:   return out;
578: }
579:
580: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
581:   m.impl(TORCH_SELECTIVE_NAME("quantized::add"),                 TORCH_FN(qadd</*ReLUFused=*/false>));
582:   m.impl(TORCH_SELECTIVE_NAME("quantized::add.out"),             TORCH_FN(qadd_out</*ReLUFused=*/false>));
583:   m.impl(TORCH_SELECTIVE_NAME("quantized::add.Scalar"),          TORCH_FN(qadd_scalar</*ReLUFused=*/false>));
584:   m.impl(TORCH_SELECTIVE_NAME("quantized::add.Scalar2"),          TORCH_FN(qadd_scalar2</*ReLUFused=*/false>));
585:   m.impl(TORCH_SELECTIVE_NAME("quantized::add.Scalar_out"),      TORCH_FN(qadd_scalar_out</*ReLUFused=*/false>));
586:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu"),            TORCH_FN(qadd</*ReLUFused=*/true>));
587:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu.out"),        TORCH_FN(qadd_out</*ReLUFused=*/true>));
588:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu.Scalar"),     TORCH_FN(qadd_scalar</*ReLUFused=*/true>));
589:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu.Scalar2"),     TORCH_FN(qadd_scalar2</*ReLUFused=*/true>));
590:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu.Scalar_out"), TORCH_FN(qadd_scalar_out</*ReLUFused=*/true>));
591:   // deprecated functions, kept for backward compatibility
592:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_out"),             TORCH_FN(qadd_out</*ReLUFused=*/false>));
593:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu_out"),        TORCH_FN(qadd_out</*ReLUFused=*/true>));
594:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar"),          TORCH_FN(qadd_scalar</*ReLUFused=*/false>));
595:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar_relu"),     TORCH_FN(qadd_scalar</*ReLUFused=*/true>));
596:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar_out"),      TORCH_FN(qadd_scalar_out</*ReLUFused=*/false>));
597:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar_relu_out"), TORCH_FN(qadd_scalar_out</*ReLUFused=*/true>));
598:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar.Tensor"),   TORCH_FN(qadd_scalar_tensor</*ReLUFused=*/false>));
599:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar_relu.Tensor"), TORCH_FN(qadd_scalar_tensor</*ReLUFused=*/true>));
600:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar_out.Tensor"), TORCH_FN(qadd_scalar_tensor_out</*ReLUFused=*/false>));
601:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_scalar_relu_out.Tensor"), TORCH_FN(qadd_scalar_tensor_out</*ReLUFused=*/true>));
602: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 604-619
```cpp
604: TORCH_LIBRARY_IMPL(_quantized, QuantizedCPU, m) {
605:   m.impl(TORCH_SELECTIVE_NAME("_quantized::add"), TORCH_FN(qadd</*ReLUFused=*/false>));
606: }
607:
608: TORCH_LIBRARY_IMPL(onednn, CPU, m) {
609:   m.impl(TORCH_SELECTIVE_NAME("onednn::qadd.tensor"), TORCH_FN(int8_add_tensor_onednn<false>));
610:   m.impl(TORCH_SELECTIVE_NAME("onednn::qadd_relu.tensor"), TORCH_FN(int8_add_tensor_onednn<true>));
611: }
612:
613: }  // namespace
614:
615: Tensor quantized_add(Tensor qa, Tensor qb, double scale, int64_t zero_point){
616:   return qadd<false>(std::move(qa), std::move(qb), scale, zero_point);
617: }
618:
619: }  // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `quantized_add`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `quantized_add`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`, `ATen/ExpandUtils.h`, `ATen/quantized/Quantizer.h`, `ATen/native/quantized/cpu/ACLUtils.h`, `ATen/native/quantized/cpu/BinaryOps.h`, `ATen/native/quantized/cpu/QuantizedOps.h`, `ATen/native/quantized/cpu/init_qnnpack.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `caffe2/utils/threadpool/pthreadpool-cpp.h`, `algorithm`, `utility`
- Key helper symbols / 关键辅助符号: `Scalar`, `ScalarType`, `qnnpack`
