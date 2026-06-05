# FusedObsFakeQuant.cu — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cuda/FusedObsFakeQuant.cu`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/ceil_div.h>
 5: #include <ATen/native/cuda/Loops.cuh>
 6: #include <c10/cuda/CUDAGuard.h>
 7:
 8: #ifndef AT_PER_OPERATOR_HEADERS
 9: #include <ATen/Functions.h>
10: #else
11: #include <ATen/ops/aminmax.h>
12: #include <ATen/ops/_fake_quantize_per_tensor_affine_cachemask_tensor_qparams.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/ceil_div.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/ceil_div.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 13-24
```
13: #include <ATen/ops/fake_quantize_per_channel_affine.h>
14: #include <ATen/ops/fake_quantize_per_channel_affine_cachemask.h>
15: #include <ATen/ops/fake_quantize_per_tensor_affine.h>
16: #include <ATen/ops/fused_moving_avg_obs_fake_quant_native.h>
17: #include <ATen/ops/ones_like.h>
18: #endif
19:
20: #include <cmath>
21:
22: namespace at::native {
23:
24: namespace {
```
- EN: This range pulls in required headers, including `ATen/ops/fake_quantize_per_channel_affine.h`, `ATen/ops/fake_quantize_per_channel_affine_cachemask.h`, `ATen/ops/fake_quantize_per_tensor_affine.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/ops/fake_quantize_per_channel_affine.h`, `ATen/ops/fake_quantize_per_channel_affine_cachemask.h`, `ATen/ops/fake_quantize_per_tensor_affine.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 25-39
```
25: template <typename T>
26: __global__ void ChooseQuantizationParamsKernelImpl(
27:     const int64_t* fake_quant_on,
28:     const T* x_min,
29:     const T* x_max,
30:     int32_t qmin,
31:     int32_t qmax,
32:     int size,
33:     bool preserve_sparsity,
34:     float* scale,
35:     int32_t* zero_point) {
36:   int i = blockIdx.x * blockDim.x + threadIdx.x;
37:   if (i < size && *fake_quant_on == 1) {
38:     float min_val = x_min[i];
39:     float max_val = x_max[i];
```
- EN: The main symbol in this range is `ChooseQuantizationParamsKernelImpl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `ChooseQuantizationParamsKernelImpl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 41-55
```
41:     if (min_val < 0 && max_val > 0 && preserve_sparsity) {
42:       int symmetric_qmin = -((qmax - qmin) / 2 + 1);
43:       int symmetric_qmax = (qmax - qmin) / 2;
44:       double max_scale = std::max(
45:           fabs(min_val / symmetric_qmin), fabs(max_val / symmetric_qmax));
46:       min_val = max_scale * symmetric_qmin;
47:       max_val = max_scale * symmetric_qmax;
48:     }
49:
50:     // We extend the [min, max] interval to ensure that it contains 0.
51:     // Otherwise, we would not meet the requirement that 0 be an exactly
52:     // representable value.
53:     min_val = std::min(min_val, 0.f);
54:     max_val = std::max(max_val, 0.f);
55:     scale[i] = (static_cast<double>(max_val) - min_val) / (qmax - qmin);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 57-72
```
57:     // Moving this check outside this function would result in extra Device to
58:     // Host copy of the min and max val which would result in a perf hit.
59:     if (scale[i] == 0.0f || ::isinf(1.0f / scale[i])) {
60:       scale[i] = 0.1;
61:     }
62:
63:     double zero_point_from_min = qmin - min_val / static_cast<double>(scale[i]);
64:     double zero_point_from_max = qmax - max_val / static_cast<double>(scale[i]);
65:     double zero_point_from_min_error =
66:         std::abs(qmin) + std::abs(min_val / static_cast<double>(scale[i]));
67:     double zero_point_from_max_error =
68:         std::abs(qmax) + std::abs(max_val / static_cast<double>(scale[i]));
69:     double initial_zero_point =
70:         zero_point_from_min_error < zero_point_from_max_error
71:         ? zero_point_from_min
72:         : zero_point_from_max;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 74-84
```
74:     // Note: preserve_sparsity here means symmetric quantization.
75:     // for symmetric quantization, we force zero_point
76:     // to be a middle value between qmin and qmax.
77:     // If either min or max is 0, then we just use 0 as zero_point.
78:     if (min_val < 0 && max_val > 0 && preserve_sparsity) {
79:       initial_zero_point = static_cast<double>(qmin + qmax) / 2;
80:     }
81:     // Now we need to nudge the zero point to be an integer
82:     // (our zero points are integer, and this is motivated by the
83:     // requirement to be able to represent the real value "0" exactly as a
84:     // quantized value, which is required in multiple places, for example in
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 85-96
```
85:     // Im2col with zero padding).
86:     int32_t nudged_zero_point = 0;
87:     if (initial_zero_point < qmin) {
88:       nudged_zero_point = qmin;
89:     } else if (initial_zero_point > qmax) {
90:       nudged_zero_point = qmax;
91:     } else {
92:       nudged_zero_point = nearbyint(initial_zero_point);
93:     }
94:     zero_point[i] = nudged_zero_point;
95:   }
96: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 98-108
```
 98: __device__ inline bool isinf_device(float v) {
 99:   return ::isinf(v);
100: }
101: __device__ inline bool isinf_device(c10::BFloat16 v) {
102:   return ::isinf(static_cast<float>(v));
103: }
104: __device__ inline bool isinf_device(at::Half v) {
105:   return ::isinf(static_cast<float>(v));
106: }
107:
108: // CUDA kernel to compute Moving Average Min/Max of the tensor.
```
- EN: The main symbol in this range is `isinf_device`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `isinf_device`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 109-123
```
109: // It uses the running_min and running_max along with averaging const, c.
110: // The formula used to compute the new min/max is as follows
111: //
112: // running_min = (1 - c) * running_min + c * x_min, if running_min != inf
113: // running_min = x_min, if running_min == inf
114: template <typename T>
115: __global__ void MovingAverageMinMax(
116:     const int64_t* observer_on,
117:     const T* x_min,
118:     const T* x_max,
119:     T* running_min,
120:     T* running_max,
121:     const float averaging_const,
122:     const int size) {
123:   int i = blockIdx.x * blockDim.x + threadIdx.x;
```
- EN: The main symbol in this range is `MovingAverageMinMax`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `MovingAverageMinMax`，它们直接构成本文件的算子逻辑。

### Lines 125-138
```
125:   if (*observer_on == 1) {
126:     if (i < size) {
127:       T curr_min = x_min[i];
128:       T curr_max = x_max[i];
129:
130:       T averaging_const_t = static_cast<T>(averaging_const);
131:
132:       T adjusted_min = isinf_device(running_min[i]) ? curr_min
133:                                                     : (running_min[i]) +
134:               averaging_const_t * (curr_min - (running_min[i]));
135:
136:       T adjusted_max = isinf_device(running_max[i]) ? curr_max
137:                                                     : (running_max[i]) +
138:               averaging_const_t * (curr_max - (running_max[i]));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 140-155
```
140:       running_min[i] = adjusted_min;
141:       running_max[i] = adjusted_max;
142:     }
143:   }
144: }
145:
146: void _calculate_moving_average(
147:     const at::Tensor& x,
148:     const at::Tensor& observer_on,
149:     at::Tensor& running_min,
150:     at::Tensor& running_max,
151:     const float averaging_const,
152:     const int64_t size,
153:     bool per_row_fq) {
154:   at::cuda::OptionalCUDAGuard device_guard;
155:   device_guard.set_index(x.get_device());
```
- EN: The main symbol in this range is `_calculate_moving_average`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `_calculate_moving_average`，它们直接构成本文件的算子逻辑。

### Lines 157-169
```
157:   at::Tensor x_min, x_max;
158:
159:   int64_t* observer_on_data = observer_on.data_ptr<int64_t>();
160:   cudaStream_t cuda_stream = at::cuda::getCurrentCUDAStream();
161:
162:   if (per_row_fq) {
163:     std::tie(x_min, x_max) = at::aminmax(x, 1);
164:     int num_threads = std::min(size, (int64_t)512);
165:     const uint64_t num_blocks = ceil_div<uint64_t>(size, num_threads);
166:     AT_DISPATCH_FLOATING_TYPES_AND2(
167:         at::kBFloat16, at::kHalf, x.scalar_type(), "aminmax_kernel", [&] {
168:           scalar_t* x_min_data = x_min.data_ptr<scalar_t>();
169:           scalar_t* x_max_data = x_max.data_ptr<scalar_t>();
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 171-181
```
171:           scalar_t* running_min_data = running_min.data_ptr<scalar_t>();
172:           scalar_t* running_max_data = running_max.data_ptr<scalar_t>();
173:
174:           // Moving Average Min/Max observer for activations
175:           MovingAverageMinMax<<<num_blocks, num_threads, 0, cuda_stream>>>(
176:               observer_on_data,
177:               x_min_data,
178:               x_max_data,
179:               running_min_data,
180:               running_max_data,
181:               averaging_const,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 182-193
```
182:               size);
183:         });
184:     C10_CUDA_KERNEL_LAUNCH_CHECK();
185:   } else {
186:     std::tie(x_min, x_max) = at::aminmax(x);
187:     AT_DISPATCH_FLOATING_TYPES_AND2(
188:         at::kBFloat16, at::kHalf, x.scalar_type(), "aminmax_kernel", [&] {
189:           scalar_t* x_min_data = x_min.data_ptr<scalar_t>();
190:           scalar_t* x_max_data = x_max.data_ptr<scalar_t>();
191:
192:           scalar_t* running_min_data = running_min.data_ptr<scalar_t>();
193:           scalar_t* running_max_data = running_max.data_ptr<scalar_t>();
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 195-207
```
195:           // Moving Average Min/Max observer for activations
196:           MovingAverageMinMax<<<1, 1, 0, cuda_stream>>>(
197:               observer_on_data,
198:               x_min_data,
199:               x_max_data,
200:               running_min_data,
201:               running_max_data,
202:               averaging_const,
203:               1 /*size*/);
204:         });
205:     C10_CUDA_KERNEL_LAUNCH_CHECK();
206:   }
207: }
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 209-222
```
209: void _calc_moving_avg_qparams_helper(
210:     const at::Tensor& x,
211:     const at::Tensor fake_quant_on,
212:     at::Tensor& running_min,
213:     at::Tensor& running_max,
214:     float* scale_ptr,
215:     int32_t* zp_ptr,
216:     int32_t qmin,
217:     int32_t qmax,
218:     bool symmetric_quant,
219:     const int64_t size,
220:     bool per_row_fq = false) {
221:   at::cuda::OptionalCUDAGuard device_guard;
222:   device_guard.set_index(x.get_device());
```
- EN: The main symbol in this range is `_calc_moving_avg_qparams_helper`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `_calc_moving_avg_qparams_helper`，它们直接构成本文件的算子逻辑。

### Lines 224-234
```
224:   cudaStream_t cuda_stream = at::cuda::getCurrentCUDAStream();
225:   int64_t* fake_quant_on_data = fake_quant_on.data_ptr<int64_t>();
226:   if (per_row_fq) {
227:     AT_DISPATCH_FLOATING_TYPES_AND2(
228:         at::kBFloat16, at::kHalf, x.scalar_type(), "aminmax_kernel", [&] {
229:           scalar_t* running_min_data = running_min.data_ptr<scalar_t>();
230:           scalar_t* running_max_data = running_max.data_ptr<scalar_t>();
231:           int num_threads = std::min(size, (int64_t)512);
232:           const uint64_t num_blocks = ceil_div<uint64_t>(size, num_threads);
233:           ChooseQuantizationParamsKernelImpl<<<
234:               num_blocks,
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 235-246
```
235:               num_threads,
236:               0,
237:               cuda_stream>>>(
238:               fake_quant_on_data,
239:               running_min_data,
240:               running_max_data,
241:               qmin,
242:               qmax,
243:               size,
244:               symmetric_quant,
245:               scale_ptr,
246:               zp_ptr);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 247-258
```
247:         });
248:     C10_CUDA_KERNEL_LAUNCH_CHECK();
249:   } else {
250:     AT_DISPATCH_FLOATING_TYPES_AND2(
251:         at::kBFloat16, at::kHalf, x.scalar_type(), "aminmax_kernel", [&] {
252:           scalar_t* running_min_data = running_min.data_ptr<scalar_t>();
253:           scalar_t* running_max_data = running_max.data_ptr<scalar_t>();
254:           ChooseQuantizationParamsKernelImpl<<<1, 1, 0, cuda_stream>>>(
255:               fake_quant_on_data,
256:               running_min_data,
257:               running_max_data,
258:               qmin,
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 259-269
```
259:               qmax,
260:               1, // size
261:               symmetric_quant, // preserve_sparsity
262:               scale_ptr,
263:               zp_ptr);
264:         });
265:     C10_CUDA_KERNEL_LAUNCH_CHECK();
266:   }
267: }
268:
269: } // namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 271-281
```
271: std::tuple<at::Tensor, at::Tensor> fused_moving_avg_obs_fake_quant_cuda(
272:     const at::Tensor& x,
273:     const at::Tensor& observer_on,
274:     const at::Tensor& fake_quant_on,
275:     at::Tensor& running_min,
276:     at::Tensor& running_max,
277:     at::Tensor& scale,
278:     at::Tensor& zero_point,
279:     const double averaging_const,
280:     const int64_t qmin,
281:     const int64_t qmax,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 282-297
```
282:     const int64_t ch_axis,
283:     bool per_row_fq,
284:     bool symmetric_quant) {
285:   TORCH_CHECK(ch_axis < x.dim(), "Error in fused_moving_avg_obs_fake_quant_cpu: ch_axis must be < self.dim()");
286:   const auto x_contig = x.contiguous();
287:   // Calculate the size of the dimension we need to quantize over,
288:   // For per-channel quant we default to axis 0, since it is only for
289:   // weight quantization currently.
290:   int64_t size = 1;
291:   if (per_row_fq) {
292:     at::Tensor y = x;
293:     if (x.dim() != 2) {
294:       auto res = DimVector(x.sizes());
295:       std::iota(res.begin(), res.end(), 0);
296:       res[ch_axis] = 0;
297:       res[0] = ch_axis;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 299-309
```
299:       y = x.permute(res);
300:       y = y.flatten(1);
301:     }
302:     size = x.size(ch_axis);
303:     if (running_min.numel() == 0) {
304:       float inf = std::numeric_limits<float>::infinity();
305:       running_min.resize_(size).fill_(inf);
306:       running_max.resize_(size).fill_(-inf);
307:       scale.resize_(size);
308:       zero_point.resize_(size);
309:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 310-321
```
310:     _calculate_moving_average(
311:         y,
312:         observer_on.to(at::kLong),
313:         running_min,
314:         running_max,
315:         averaging_const,
316:         size,
317:         per_row_fq);
318:   } else {
319:     _calculate_moving_average(
320:         x_contig,
321:         observer_on.to(at::kLong),
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 322-333
```
322:         running_min,
323:         running_max,
324:         averaging_const,
325:         size,
326:         per_row_fq);
327:   }
328:
329:   float* scale_ptr = scale.data_ptr<float>();
330:   int32_t* zp_ptr = zero_point.data_ptr<int32_t>();
331:
332:   _calc_moving_avg_qparams_helper(
333:       x_contig,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 334-345
```
334:       fake_quant_on.to(at::kLong),
335:       running_min,
336:       running_max,
337:       scale_ptr,
338:       zp_ptr,
339:       qmin,
340:       qmax,
341:       symmetric_quant,
342:       size,
343:       per_row_fq);
344:
345:   if (per_row_fq) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 346-357
```
346:     if (fake_quant_on.item().toInt()) {
347:       return at::fake_quantize_per_channel_affine_cachemask(
348:           x, scale, zero_point, 0, qmin, qmax);
349:     } else {
350:       auto mask = at::ones_like(x, at::kBool, MemoryFormat::Preserve);
351:       return std::make_tuple(x.clone(), mask);
352:     }
353:   } else {
354:     return at::_fake_quantize_per_tensor_affine_cachemask_tensor_qparams(
355:         x, scale, zero_point, fake_quant_on.to(at::kLong), qmin, qmax);
356:   }
357: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 358-358
```
358: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Low-level memory access / 底层内存访问
- Namespace-scoped helper structure / 命名空间内辅助结构

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/ceil_div.h`, `ATen/native/cuda/Loops.cuh`, `ATen/Functions.h`, `ATen/ops/aminmax.h`, `ATen/ops/_fake_quantize_per_tensor_affine_cachemask_tensor_qparams.h`, `ATen/ops/fake_quantize_per_channel_affine.h`, `ATen/ops/fake_quantize_per_channel_affine_cachemask.h`, `ATen/ops/fake_quantize_per_tensor_affine.h`
- c10 headers / c10 头文件: `c10/cuda/CUDAGuard.h`
- Standard or third-party headers / 标准库或第三方头文件: `cmath`
- Key helper symbols / 关键辅助符号: `AT_DISPATCH_FLOATING_TYPES`
