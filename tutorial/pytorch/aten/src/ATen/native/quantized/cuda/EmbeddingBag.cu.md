# EmbeddingBag.cu — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cuda/EmbeddingBag.cu`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU embedding lookup or embedding-bag accumulation logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU embedding 查找或 embedding-bag 累加逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/Parallel.h>
 5: #include <ATen/core/op_registration/op_registration.h>
 6: #include <ATen/cuda/CUDAContext.h>
 7: #include <ATen/native/TensorIterator.h>
 8: #include <ATen/quantized/Quantizer.h>
 9: #include <c10/cuda/CUDAGuard.h>
10: #include <torch/library.h>
11:
12: #ifndef AT_PER_OPERATOR_HEADERS
13: #include <ATen/Functions.h>
14: #include <ATen/NativeFunctions.h>
15: #else
16: #include <ATen/ops/arange.h>
17: #include <ATen/ops/empty.h>
18: #include <ATen/ops/empty_native.h>
19: #include <ATen/ops/resize_native.h>
20: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 22-40
```
22: namespace at::native {
23:
24: // BEGIN QUANTIZE HELPER FUNCTIONS
25: __device__ __forceinline__ float bfe(uint32_t val, uint32_t pos, uint32_t len) {
26: #ifdef USE_ROCM
27:   return *reinterpret_cast<float*>((val >> pos) && ((1u << len) - 1u ));
28: #else
29:   uint32_t ret;
30:   // Get the bit field of [pos, pos+len) bits from val:
31:   // (val >> pos) && ( (1u << len) - 1u )
32:   asm("bfe.u32 %0, %1, %2, %3;" : "=r"(ret) : "r"(val), "r"(pos), "r"(len));
33:   return __uint2float_rn(ret);
34: #endif
35: }
36:
37: // FMA with constant scale/bias for all 4 floats in fa
38: __forceinline__ __device__ float4
39: fma4sb(const float4 fa, const float fscale, const float fbias) {
40:   float4 res;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `bfe`, `fma4sb`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `bfe`, `fma4sb`，它们直接构成本文件的算子逻辑。

### Lines 41-63
```
41: #ifdef USE_ROCM
42:   res.x = fa.x * fscale + fbias;
43:   res.y = fa.y * fscale + fbias;
44:   res.z = fa.z * fscale + fbias;
45:   res.w = fa.w * fscale + fbias;
46: #else
47:   res.x = fmaf(fa.x, fscale, fbias);
48:   res.y = fmaf(fa.y, fscale, fbias);
49:   res.z = fmaf(fa.z, fscale, fbias);
50:   res.w = fmaf(fa.w, fscale, fbias);
51: #endif
52:   return res;
53: }
54:
55: template <uint8_t bits_per_dim>
56: __forceinline__ __device__ float4
57: dequantize_intx(uint32_t packedVals, float2 scale_bias, uint8_t offset_bits) {
58:   float4 res;
59:
60:   res.x = bfe(packedVals, offset_bits + (0 * bits_per_dim), bits_per_dim);
61:   res.y = bfe(packedVals, offset_bits + (1 * bits_per_dim), bits_per_dim);
62:   res.z = bfe(packedVals, offset_bits + (2 * bits_per_dim), bits_per_dim);
63:   res.w = bfe(packedVals, offset_bits + (3 * bits_per_dim), bits_per_dim);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `dequantize_intx`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `dequantize_intx`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 65-82
```
65:   return fma4sb(res, scale_bias.x, scale_bias.y);
66: }
67:
68: template <uint8_t bits_per_dim>
69: __forceinline__ __device__ void
70: accumulate_packed_intx(float4* acc, uint32_t packedVals, float2 scale_bias, float sample_weight) {
71:   constexpr uint8_t dims_per_byte = 8 / bits_per_dim;
72:   for (uint8_t i = 0; i < dims_per_byte; i++) {
73:     float4 res = dequantize_intx<bits_per_dim>(packedVals, scale_bias, 4 * bits_per_dim * i /* offset_bits */);
74:     // Accumulate in float32.
75:     acc[i].x += (res.x * sample_weight);
76:     acc[i].y += (res.y * sample_weight);
77:     acc[i].z += (res.z * sample_weight);
78:     acc[i].w += (res.w * sample_weight);
79:   }
80: }
81:
82: // END QUANTIZE HELPER FUNCTIONS
```
- EN: The main symbol in this range is `accumulate_packed_intx`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `accumulate_packed_intx`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 84-106
```
 84: // UN-OPTIMIZED kernel, doesn't even avoid warp divergence!
 85: template <typename index_t, uint8_t bits_per_dim>
 86: __global__ void embedding_bag_nbits_rowwise_offsets_kernel(
 87:     const PackedTensorAccessor64<uint8_t, 2, RestrictPtrTraits> weight,
 88:     const PackedTensorAccessor32<index_t, 1, RestrictPtrTraits> indices,
 89:     const PackedTensorAccessor32<index_t, 1, RestrictPtrTraits> offsets,
 90:     const bool /* pruned_weights */,
 91:     const PackedTensorAccessor32<float, 1, RestrictPtrTraits> per_sample_weights_,
 92:     const std::optional<Tensor>& compressed_indices_mapping,
 93:     const bool include_last_offset,
 94:     PackedTensorAccessor32<float, 2, RestrictPtrTraits> output) {
 95:   static_assert(bits_per_dim == 4 || bits_per_dim == 8, "the current embedding_bag_nbits_rowwise_offsets_kernel only has been tested for 4 and 8 bits per dim");
 96:   constexpr uint8_t dims_per_byte = 8 / bits_per_dim;
 97:   constexpr bool fp32_scale_bias = bits_per_dim == 8;
 98:
 99:   int32_t B = output.size(0);
100:   int32_t D = output.size(1);
101:   int32_t b_t = blockIdx.x * blockDim.y + threadIdx.y;
102:   if (b_t >= B * D) {
103:     return;
104:   }
105:   int32_t t = b_t / B;
106:   int32_t b = b_t % B;
```
- EN: The main symbol in this range is `embedding_bag_nbits_rowwise_offsets_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `embedding_bag_nbits_rowwise_offsets_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 108-129
```
108:   const int32_t D_bytes = weight.size(1);
109:
110:   bool use_per_sample = per_sample_weights_.size(0) > 0;
111:
112:   int64_t indices_start = offsets[t * B + b];
113:   int64_t indices_end;
114:   if (include_last_offset) {
115:     indices_end = offsets[t * B + b + 1];
116:   } else {
117:     indices_end = (t * B + b + 1) < offsets.size(0) ? offsets[t * B + b + 1]
118:                                                     : indices.size(0);
119:   }
120:
121:   int32_t L = indices_end - indices_start;
122:   const uint8_t* __restrict__ weights = &weight[0][0];
123:
124:   if (L == 0) {
125:     for (int32_t d = 0; d < D; d += 4) {
126:       *(float4*)(&output[b][d]) = make_float4(0, 0, 0, 0);
127:     }
128:     return;
129:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 132-151
```
132:   float4 accumulator[dims_per_byte];
133:   int32_t byte_offset = 0;
134:   for (int32_t d = 0; d < D; d += dims_per_byte * 4, byte_offset += 4) {
135:     for (int32_t i = 0; i < dims_per_byte; ++i) {
136:         accumulator[i] = make_float4(0, 0, 0, 0);
137:     }
138:     for (int32_t l = indices_start; l < indices_end; ++l) {
139:       int64_t idx = indices[l];
140:       float sample_weight = use_per_sample ? per_sample_weights_[l] : 1.0f;
141:       const uint8_t* __restrict__ row = &weights[idx * D_bytes];
142:       float2 scale_bias;
143:       if (fp32_scale_bias) {
144:         scale_bias = make_float2(
145:             reinterpret_cast<const float*>(&row[D_bytes - 8])[0],
146:             reinterpret_cast<const float*>(&row[D_bytes - 4])[0]);
147:       } else {
148:         scale_bias = make_float2(
149:             __half2float(reinterpret_cast<const __half*>(&row[D_bytes - 4])[0]),
150:             __half2float(reinterpret_cast<const __half*>(&row[D_bytes - 2])[0]));
151:       }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 153-176
```
153:       uint32_t v0 = reinterpret_cast<const uint32_t*>(&row[byte_offset])[0];
154:
155:       accumulate_packed_intx<bits_per_dim>(accumulator, v0, scale_bias, sample_weight);
156:     }
157:
158:
159:     for (int32_t i = 0; i < dims_per_byte; ++i) {
160:       *(float4*)(&output[b][d + (i * 4)]) = accumulator[i];
161:     }
162:   }
163: }
164:
165: inline at::Tensor create_empty_from(
166:     const at::Tensor& t,
167:     c10::ScalarType dtype) {
168:   return at::native::empty_cuda({0}, dtype, t.layout(), t.device(), false);
169: }
170:
171: Tensor qembeddingbag_byte_unpack(const Tensor& packed_weight) {
172:   const auto packed_weight_sizes = packed_weight.sizes();
173:   const auto col_dim = packed_weight_sizes.size() - 1;
174:   const int32_t input_rows = c10::size_to_dim_(col_dim, packed_weight_sizes);
175:   const int32_t input_columns = packed_weight_sizes[col_dim];
176:   const int32_t output_columns = input_columns - 2 * sizeof(float);
```
- EN: The main symbol in this range is `create_empty_from`, `qembeddingbag_byte_unpack`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `create_empty_from`, `qembeddingbag_byte_unpack`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 178-196
```
178:   std::vector<int64_t> output_shape = packed_weight_sizes.vec();
179:   output_shape[col_dim] = output_columns;
180:
181:   return at::empty(
182:       output_shape,
183:       packed_weight.options().dtype(kFloat),
184:       packed_weight.suggest_memory_format());
185: }
186:
187: template <typename IndexType, typename OffsetType>
188: at::Tensor& embedding_bag_byte_impl(
189:     at::Tensor& output,
190:     const at::Tensor& weight,
191:     const at::Tensor& indices,
192:     const at::Tensor& offsets,
193:     bool pruned_weights,
194:     const std::optional<at::Tensor>& per_sample_weights_,
195:     const std::optional<at::Tensor>& compressed_indices_mapping,
196:     bool include_last_offset,
```
- EN: The main symbol in this range is `embedding_bag_byte_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `embedding_bag_byte_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 197-216
```
197:     bool is_embedding_op) {
198:   TORCH_CHECK(weight.is_cuda());
199:   TORCH_CHECK(indices.is_cuda());
200:   TORCH_CHECK(offsets.is_cuda());
201:   TORCH_CHECK(indices.device() == weight.device())
202:   TORCH_CHECK(offsets.device() == weight.device());
203:   if (per_sample_weights_.has_value()) {
204:     TORCH_CHECK(per_sample_weights_.value().device() == weight.device());
205:   }
206:   if (compressed_indices_mapping.has_value()) {
207:     TORCH_CHECK(compressed_indices_mapping.value().device() == weight.device());
208:   }
209:
210:   TORCH_CHECK(weight.dtype() == at::kByte);
211:   TORCH_CHECK(weight.dim() == 2);
212:
213:   at::cuda::OptionalCUDAGuard device_guard;
214:   device_guard.set_index(weight.get_device());
215:
216:   const auto weight_sizes = weight.sizes();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 217-239
```
217:   const int64_t N = weight_sizes[0];
218:   const int D = weight_sizes[1] - 8; // NB: -8 to account for scale and bias
219:   const int64_t M = offsets.sizes()[0];
220:   TORCH_CHECK(D % 4 == 0);
221:   if(per_sample_weights_.has_value()) {
222:       TORCH_CHECK(per_sample_weights_.value().scalar_type() == at::kFloat,
223:               "Per sample weights expected scalar type ", at::kFloat, " but got ",
224:               per_sample_weights_.value().scalar_type());
225:   }
226:   TORCH_CHECK(
227:       !compressed_indices_mapping.has_value(),
228:       "Compressed indices mapping not yet implemented for embedding_bag_byte_rowwise_offsets_cuda");
229:
230:   const auto maxThreads = at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock;
231:
232:   int64_t output_size = include_last_offset ? M - 1 : M;
233:
234:   at::Tensor sample_weights;
235:   if (per_sample_weights_.has_value()) {
236:       sample_weights = per_sample_weights_.value();
237:   } else {
238:       sample_weights = create_empty_from(output, kFloat);
239:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 241-259
```
241:   const std::vector<int64_t> shape = {output_size, D};
242:   at::native::resize_(output, shape, std::nullopt);
243:   AT_DISPATCH_INDEX_TYPES(
244:       indices.scalar_type(), "embedding_bag_byte_rowwise_offsets_kernel", ([&] {
245:         embedding_bag_nbits_rowwise_offsets_kernel<index_t, 8><<<
246:             output_size,
247:             dim3(1, 1, 1),
248:             0,
249:             at::cuda::getCurrentCUDAStream()>>>(
250:             weight.packed_accessor64<uint8_t, 2, RestrictPtrTraits>(),
251:             indices.packed_accessor32<index_t, 1, RestrictPtrTraits>(),
252:             offsets.packed_accessor32<index_t, 1, RestrictPtrTraits>(),
253:             false /* pruned_weights */,
254:             sample_weights.packed_accessor32<float, 1, RestrictPtrTraits>(),
255:             compressed_indices_mapping,
256:             include_last_offset,
257:             output.packed_accessor32<float, 2, RestrictPtrTraits>());
258:         C10_CUDA_KERNEL_LAUNCH_CHECK();
259:       }));
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 261-279
```
261:   TORCH_CHECK(output.is_cuda());
262:
263:   return output;
264: }
265:
266: Tensor embedding_bag_byte_rowwise_offsets(
267:     const Tensor& weight,
268:     const Tensor& indices,
269:     const std::optional<Tensor>& offsets_in,
270:     const bool /* scale_grad_by_freq */,
271:     const int64_t /* mode */,
272:     bool pruned_weights,
273:     const std::optional<Tensor>& per_sample_weights_,
274:     const std::optional<Tensor>& compressed_indices_mapping,
275:     bool include_last_offset) {
276:   bool is_embedding_op = false;
277:   auto output = create_empty_from(weight, at::kFloat);
278:
279:   c10::MaybeOwned<at::Tensor> offsets;
```
- EN: The main symbol in this range is `embedding_bag_byte_rowwise_offsets`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `embedding_bag_byte_rowwise_offsets`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 280-299
```
280:   TORCH_CHECK(
281:       indices.dim() == 1 || indices.dim() == 2,
282:       "qembedding/qembedding_bag operator supports 1 or 2d indices, got ",
283:       indices.dim());
284:   // For embedding_bag operator with 2D indices, we set the offsets explicitly
285:   // here.
286:   if (indices.dim() == 2 && !is_embedding_op) {
287:     TORCH_CHECK(
288:         !offsets_in.has_value(),
289:         "embedding_bag_byte operator: input is 2D, then offsets has to be None, as input is treated is a mini-batch of fixed length sequences.");
290:
291:     offsets = c10::MaybeOwned<at::Tensor>::owned(at::arange(
292:         0, indices.numel(), indices.sizes()[1], indices.scalar_type()));
293:
294:   } else {
295:     TORCH_CHECK(
296:         offsets_in.has_value(),
297:         "embedding_bag_byte expects offsets to be set for 1D indices.");
298:     offsets = c10::MaybeOwned<at::Tensor>::borrowed(offsets_in.value());
299:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 301-319
```
301:   TORCH_CHECK(
302:       indices.scalar_type() == at::kInt || indices.scalar_type() == at::kLong,
303:       "Expect 32 or 64 bit indices, but found ",
304:       indices.scalar_type(),
305:       " instead.");
306:   TORCH_CHECK(
307:       offsets->scalar_type() == at::kInt || offsets->scalar_type() == at::kLong,
308:       "Expect 32 or 64 bit offsets, but found ",
309:       offsets->scalar_type(),
310:       " instead.");
311:   TORCH_CHECK(
312:       weight.is_contiguous() && indices.is_contiguous() &&
313:           offsets->is_contiguous(),
314:       "Expect weight, indices, and offsets to be contiguous.");
315:
316:   // Using helper function to support different type combination without the
317:   // need to cast, which can be additional performance overhead
318:   if (indices.scalar_type() == at::kInt && offsets->scalar_type() == at::kInt) {
319:     return embedding_bag_byte_impl<int, int>(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 320-339
```
320:         output,
321:         weight,
322:         indices,
323:         *offsets,
324:         pruned_weights,
325:         per_sample_weights_,
326:         compressed_indices_mapping,
327:         include_last_offset,
328:         is_embedding_op);
329:   } else if (
330:       indices.scalar_type() == at::kInt &&
331:       offsets->scalar_type() == at::kLong) {
332:     return embedding_bag_byte_impl<int, int64_t>(
333:         output,
334:         weight,
335:         indices,
336:         *offsets,
337:         pruned_weights,
338:         per_sample_weights_,
339:         compressed_indices_mapping,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 340-368
```
340:         include_last_offset,
341:         is_embedding_op);
342:   } else if (
343:       indices.scalar_type() == at::kLong &&
344:       offsets->scalar_type() == at::kInt) {
345:     return embedding_bag_byte_impl<int64_t, int>(
346:         output,
347:         weight,
348:         indices,
349:         *offsets,
350:         pruned_weights,
351:         per_sample_weights_,
352:         compressed_indices_mapping,
353:         include_last_offset,
354:         is_embedding_op);
355:   }
356:
357:   // default case given the TORCH_CHECK above
358:   return embedding_bag_byte_impl<int64_t, int64_t>(
359:       output,
360:       weight,
361:       indices,
362:       *offsets,
363:       pruned_weights,
364:       per_sample_weights_,
365:       compressed_indices_mapping,
366:       include_last_offset,
367:       is_embedding_op);
368: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 370-390
```
370: template <typename IndexType, typename OffsetType>
371: at::Tensor& embedding_bag_4bit_impl(
372:     at::Tensor& output,
373:     const at::Tensor& weight,
374:     const at::Tensor& indices,
375:     const at::Tensor& offsets,
376:     bool pruned_weights,
377:     const std::optional<at::Tensor>& per_sample_weights_,
378:     const std::optional<at::Tensor>& compressed_indices_mapping,
379:     bool include_last_offset) {
380:   TORCH_CHECK(weight.is_cuda());
381:   TORCH_CHECK(indices.is_cuda());
382:   TORCH_CHECK(offsets.is_cuda());
383:   TORCH_CHECK(indices.device() == weight.device())
384:   TORCH_CHECK(offsets.device() == weight.device());
385:   if (per_sample_weights_.has_value()) {
386:     TORCH_CHECK(per_sample_weights_.value().device() == weight.device());
387:   }
388:   if (compressed_indices_mapping.has_value()) {
389:     TORCH_CHECK(compressed_indices_mapping.value().device() == weight.device());
390:   }
```
- EN: The main symbol in this range is `embedding_bag_4bit_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `embedding_bag_4bit_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 392-410
```
392:   TORCH_CHECK(weight.dtype() == at::kByte);
393:   TORCH_CHECK(weight.dim() == 2);
394:
395:   at::cuda::OptionalCUDAGuard device_guard;
396:   device_guard.set_index(weight.get_device());
397:
398:   const auto weight_sizes = weight.sizes();
399:   const int64_t N = weight_sizes[0];
400:   const int D = 2*(weight_sizes[1] - 4); // NB: -4 to account for scale and bias @fp16
401:   const int64_t M = offsets.sizes()[0];
402:   TORCH_CHECK(D % 8 == 0);
403:   if(per_sample_weights_.has_value()) {
404:       TORCH_CHECK(per_sample_weights_.value().scalar_type() == at::kFloat,
405:               "Per sample weights expected scalar type ", at::kFloat, " but got ",
406:               per_sample_weights_.value().scalar_type());
407:   }
408:   TORCH_CHECK(
409:       !compressed_indices_mapping.has_value(),
410:       "Compressed indices mapping not yet implemented for embedding_bag_byte_rowwise_offsets_cuda");
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 412-430
```
412:   const auto maxThreads = at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock;
413:
414:   int64_t output_size = include_last_offset ? M - 1 : M;
415:
416:   at::Tensor sample_weights;
417:   if (per_sample_weights_.has_value()) {
418:       sample_weights = per_sample_weights_.value();
419:   } else {
420:       sample_weights = create_empty_from(output, kFloat);
421:   }
422:
423:   const std::vector<int64_t> shape = {output_size, D};
424:   at::native::resize_(output, shape, std::nullopt);
425:   AT_DISPATCH_INDEX_TYPES(
426:       indices.scalar_type(), "embedding_bag_4bit_rowwise_offsets_kernel", ([&] {
427:         embedding_bag_nbits_rowwise_offsets_kernel<index_t, 4><<<
428:             output_size,
429:             dim3(1, 1, 1),
430:             0,
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 431-458
```
431:             at::cuda::getCurrentCUDAStream()>>>(
432:             weight.packed_accessor64<uint8_t, 2, RestrictPtrTraits>(),
433:             indices.packed_accessor32<index_t, 1, RestrictPtrTraits>(),
434:             offsets.packed_accessor32<index_t, 1, RestrictPtrTraits>(),
435:             false /* pruned_weights */,
436:             sample_weights.packed_accessor32<float, 1, RestrictPtrTraits>(),
437:             compressed_indices_mapping,
438:             include_last_offset,
439:             output.packed_accessor32<float, 2, RestrictPtrTraits>());
440:         C10_CUDA_KERNEL_LAUNCH_CHECK();
441:       }));
442:
443:   TORCH_CHECK(output.is_cuda());
444:
445:   return output;
446: }
447:
448: Tensor embedding_bag_4bit_rowwise_offsets(
449:     const Tensor& weight,
450:     const Tensor& indices,
451:     const std::optional<Tensor>& offsets_in,
452:     const bool /* scale_grad_by_freq */,
453:     const int64_t /* mode */,
454:     bool pruned_weights,
455:     const std::optional<Tensor>& per_sample_weights_,
456:     const std::optional<Tensor>& compressed_indices_mapping,
457:     bool include_last_offset) {
458:   auto output = create_empty_from(weight, at::kFloat);
```
- EN: The main symbol in this range is `embedding_bag_4bit_rowwise_offsets`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `embedding_bag_4bit_rowwise_offsets`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 460-480
```
460:   c10::MaybeOwned<at::Tensor> offsets;
461:   TORCH_CHECK(
462:       indices.dim() == 1 || indices.dim() == 2,
463:       "qembedding/qembedding_bag operator supports 1 or 2d indices, got ",
464:       indices.dim());
465:
466:   // For embedding_bag operator with 2D indices, we need to set the offsets
467:   // explicitly here.
468:   if (indices.dim() == 2) {
469:     TORCH_CHECK(
470:         !offsets_in.has_value(),
471:         "embedding_bag_4bit operator: input is 2D, then offsets has to be None, as input is treated is a mini-batch of fixed length sequences.");
472:
473:     offsets = c10::MaybeOwned<at::Tensor>::owned(at::arange(
474:         0, indices.numel(), indices.sizes()[1], indices.scalar_type()));
475:   } else {
476:     TORCH_CHECK(
477:         offsets_in.has_value(),
478:         "embedding_bag_4bit operator expects offsets to be set for 1D indices.");
479:     offsets = c10::MaybeOwned<at::Tensor>::borrowed(offsets_in.value());
480:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 482-500
```
482:   TORCH_CHECK(
483:       indices.scalar_type() == at::kInt || indices.scalar_type() == at::kLong,
484:       "Expect 32 or 64 bit indices, but found ",
485:       indices.scalar_type(),
486:       " instead.");
487:   TORCH_CHECK(
488:       offsets->scalar_type() == at::kInt || offsets->scalar_type() == at::kLong,
489:       "Expect 32 or 64 bit offsets, but found ",
490:       offsets->scalar_type(),
491:       " instead.");
492:   TORCH_CHECK(
493:       weight.is_contiguous() && indices.is_contiguous() &&
494:           offsets->is_contiguous(),
495:       "Expect weight, indices, and offsets to be contiguous.");
496:
497:   if (indices.scalar_type() == at::kInt && offsets->scalar_type() == at::kInt) {
498:     return embedding_bag_4bit_impl<int, int>(
499:         output,
500:         weight,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 501-520
```
501:         indices,
502:         *offsets,
503:         pruned_weights,
504:         per_sample_weights_,
505:         compressed_indices_mapping,
506:         include_last_offset);
507:   } else if (
508:       indices.scalar_type() == at::kInt &&
509:       offsets->scalar_type() == at::kLong) {
510:     return embedding_bag_4bit_impl<int, int64_t>(
511:         output,
512:         weight,
513:         indices,
514:         *offsets,
515:         pruned_weights,
516:         per_sample_weights_,
517:         compressed_indices_mapping,
518:         include_last_offset);
519:   } else if (
520:       indices.scalar_type() == at::kLong &&
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 521-541
```
521:       offsets->scalar_type() == at::kInt) {
522:     return embedding_bag_4bit_impl<int64_t, int>(
523:         output,
524:         weight,
525:         indices,
526:         *offsets,
527:         pruned_weights,
528:         per_sample_weights_,
529:         compressed_indices_mapping,
530:         include_last_offset);
531:   }
532:   return embedding_bag_4bit_impl<int64_t, int64_t>(
533:       output,
534:       weight,
535:       indices,
536:       *offsets,
537:       pruned_weights,
538:       per_sample_weights_,
539:       compressed_indices_mapping,
540:       include_last_offset);
541: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 543-562
```
543: Tensor qembeddingbag_4bit_unpack(const Tensor& packed_weight) {
544:   int BIT_RATE = 4;
545:   const auto input_rows = packed_weight.size(0);
546:   const auto input_columns = packed_weight.size(1);
547:   const auto* input_data = packed_weight.const_data_ptr<uint8_t>();
548:   int NUM_ELEM_PER_BYTE = 8 / BIT_RATE;
549:
550:   // The last 4 bytes per row are two fp16 scale and zero_point.
551:   // The rest of input_columns is the number of values in the original row.
552:   std::vector<int64_t> output_dimensions = {
553:       input_rows,
554:       static_cast<std::int64_t>(input_columns - 2 * sizeof(at::Half)) *
555:           NUM_ELEM_PER_BYTE};
556:
557:   auto output = at::empty(
558:       output_dimensions,
559:       packed_weight.options().dtype(kFloat),
560:       packed_weight.suggest_memory_format());
561:   return output;
562: }
```
- EN: The main symbol in this range is `qembeddingbag_4bit_unpack`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qembeddingbag_4bit_unpack`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 564-579
```
564: TORCH_LIBRARY_IMPL(quantized, CUDA, m) {
565:   m.impl(
566:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_byte_unpack"),
567:       TORCH_FN(qembeddingbag_byte_unpack));
568:   m.impl(
569:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_byte_rowwise_offsets"),
570:       TORCH_FN(embedding_bag_byte_rowwise_offsets));
571:   m.impl(
572:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_4bit_unpack"),
573:       TORCH_FN(qembeddingbag_4bit_unpack));
574:   m.impl(
575:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_4bit_rowwise_offsets"),
576:       TORCH_FN(embedding_bag_4bit_rowwise_offsets));
577: }
578:
579: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/core/op_registration/op_registration.h`, `ATen/cuda/CUDAContext.h`, `ATen/native/TensorIterator.h`, `ATen/quantized/Quantizer.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/arange.h`
- c10 headers / c10 头文件: `c10/cuda/CUDAGuard.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Scalar`, `ScalarType`
