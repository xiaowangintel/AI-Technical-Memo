# qembeddingbag_prepack.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qembeddingbag_prepack.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU embedding lookup or embedding-bag accumulation logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU embedding 查找或 embedding-bag 累加逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/native/quantized/cpu/qembeddingbag_prepack.h>
 3:
 4: #include <ATen/Parallel.h>
 5: #include <ATen/Utils.h>
 6: #include <ATen/core/Tensor.h>
 7: #include <ATen/core/custom_class.h>
 8: #include <ATen/native/quantized/cpu/EmbeddingPackedParams.h>
 9: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
10: #include <ATen/native/quantized/library.h>
11: #include <c10/core/ScalarType.h>
12: #include <torch/library.h>
13:
14: #ifndef AT_PER_OPERATOR_HEADERS
15: #include <ATen/Functions.h>
16: #include <ATen/NativeFunctions.h>
17: #else
18: #include <ATen/ops/choose_qparams_optimized.h>
19: #include <ATen/ops/empty.h>
20: #include <ATen/ops/resize_native.h>
21: #endif
```
- EN: This range pulls in required headers, including `ATen/native/quantized/cpu/qembeddingbag_prepack.h`, `ATen/Parallel.h`, `ATen/Utils.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/native/quantized/cpu/qembeddingbag_prepack.h`, `ATen/Parallel.h`, `ATen/Utils.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 23-44
```cpp
23: #include <c10/util/irange.h>
24:
25: #include <utility>
26:
27: /*
28:  * Prepack function for embedding_bag weights.
29:  * This function expects a per-row quantized weight tensor
30:  * with a floating point scale and zero_point value.
31:  * zero point is set to be (-Xmin/scale)
32:  * To prepack the weights we store the scale and bias (where bias is Xmin)
33:  * for each row along with the quantized weights.
34:  */
35: c10::intrusive_ptr<EmbeddingPackedParamsBase> PackedEmbeddingBagWeight::prepack(
36:     const at::Tensor& qweight) {
37:   static constexpr int64_t version = 1;
38:   TORCH_CHECK(
39:       qweight.dim() == 2,
40:       "quantized::embedding_bag_prepack weight tensor rank should be 2");
41:   TORCH_CHECK(
42:       qweight.scalar_type() == c10::kQUInt8 ||
43:           qweight.scalar_type() == c10::kQUInt4x2,
44:       "qembedding_bag_prepack currently only supports quint8 and quint4x2 weights");
```
- EN: This range pulls in required headers, including `c10/util/irange.h`, `utility`. The main symbol in this range is `be`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `c10/util/irange.h`, `utility`。 这一段的主要符号是 `be`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 46-68
```cpp
46:   at::Tensor weight_contig =
47:       qweight.contiguous(qweight.suggest_memory_format());
48:
49:   int bit_width = 0, scale_bias_bytes = 0;
50:   uint8_t* weight_data = static_cast<uint8_t*>(weight_contig.data_ptr());
51:   if (qweight.scalar_type() == c10::kQUInt8) {
52:     bit_width = 8;
53:     scale_bias_bytes =
54:         sizeof(float) * 2; // extra 8 bytes to store FP scale and bias per row.
55:   } else {
56:     bit_width = 4;
57:     scale_bias_bytes = sizeof(at::Half) *
58:         2; // extra 4 bytes to store at::Half scale and bias per row.
59:   }
60:   const auto num_elem_per_byte = 8 / bit_width;
61:
62:   int64_t embedding_rows = qweight.size(0);
63:   int64_t embedding_cols = qweight.size(1);
64:   const auto qtype = qweight.qscheme();
65:   TORCH_CHECK(
66:       qtype == c10::kPerChannelAffineFloatQParams,
67:       "Expect embedding_bag weights to be quantized using kPerChannelAffineFloatQParams");
68:   std::vector<float> weight_bias(embedding_rows);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 70-93
```cpp
70:   const auto& channel_scales = qweight.q_per_channel_scales();
71:   const auto& channel_zero_points = qweight.q_per_channel_zero_points();
72:   std::vector<float> weight_scales(
73:       channel_scales.data_ptr<float>(),
74:       channel_scales.data_ptr<float>() + embedding_rows);
75:   std::vector<float> weight_zero_points(
76:       channel_zero_points.data_ptr<float>(),
77:       channel_zero_points.data_ptr<float>() + embedding_rows);
78:
79:   for (const auto i : c10::irange(embedding_rows)) {
80:     // As of now weight_zero_points and weight_scales are initialized with
81:     // the size of embedding_rows. Hence, this linter is a false positive.
82:     // However, if this assumption changes in the future, we need to
83:     // ensure that the bounds are checked.
84:     // NOLINTNEXTLINE(facebook-hte-LocalUncheckedArrayBounds)
85:     weight_bias[i] = weight_zero_points[i] * weight_scales[i] * -1;
86:   }
87:
88:   std::vector<int64_t> output_shape = {
89:       embedding_rows,
90:       static_cast<std::int64_t>(
91:           (embedding_cols + num_elem_per_byte - 1) / num_elem_per_byte +
92:           scale_bias_bytes)}; // extra bytes to store scale and bias per row.
93:   size_t output_columns = output_shape[1];
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 95-113
```cpp
 95:   // Allocate output packed weights.
 96:   at::Tensor output = at::empty(
 97:       output_shape,
 98:       weight_contig.options().dtype(at::kByte),
 99:       weight_contig.suggest_memory_format());
100:   auto* output_data = output.data_ptr<uint8_t>();
101:
102:   if (bit_width == 8) {
103:     at::parallel_for(
104:         0, embedding_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
105:           for (const auto row : c10::irange(start_idx, end_idx)) {
106:             const uint8_t* input_row = weight_data + row * embedding_cols;
107:             std::uint8_t* output_row = output_data + row * output_columns;
108:             auto output_row_scale_bias = output_row + embedding_cols;
109:             // don't use float* to avoid unaligned address access
110:             std::memcpy(
111:                 output_row_scale_bias, &(weight_scales[row]), sizeof(float));
112:             std::memcpy(
113:                 output_row_scale_bias + sizeof(float),
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 114-139
```cpp
114:                 &(weight_bias[row]),
115:                 sizeof(float));
116:             for (const auto col : c10::irange(embedding_cols)) {
117:               output_row[col] = input_row[col];
118:             }
119:           }
120:         });
121:   } else {
122:     // Re-calculate the number of embedding_cols, to account for values packed
123:     // in a byte.
124:     embedding_cols =
125:         (embedding_cols + num_elem_per_byte - 1) / num_elem_per_byte;
126:     at::parallel_for(
127:         0, embedding_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
128:           for (const auto row : c10::irange(start_idx, end_idx)) {
129:             const uint8_t* input_row = weight_data + row * embedding_cols;
130:             std::uint8_t* output_row = output_data + row * output_columns;
131:             auto output_row_scale_bias = output_row + embedding_cols;
132:             at::Half weight_scale = weight_scales[row];
133:             at::Half weight_bias_half = weight_bias[row];
134:             // don't use at::Half* to avoid unaligned address access
135:             std::memcpy(output_row_scale_bias, &weight_scale, sizeof(at::Half));
136:             std::memcpy(
137:                 output_row_scale_bias + sizeof(at::Half),
138:                 &weight_bias_half,
139:                 sizeof(at::Half));
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 141-159
```cpp
141:             for (const auto col : c10::irange(embedding_cols)) {
142:               // The weight values have already been packed, so here we just
143:               // store it in the output tensor.
144:               output_row[col] = input_row[col];
145:             }
146:           }
147:         });
148:   }
149:
150:   auto packed_ptr = c10::make_intrusive<PackedEmbeddingBagWeight>(
151:       output,
152:       std::move(weight_scales),
153:       std::move(weight_zero_points),
154:       bit_width,
155:       qtype,
156:       version);
157:
158:   return packed_ptr;
159: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 161-180
```cpp
161: #ifdef USE_FBGEMM
162: namespace {
163: /// Number of columns in the rowwise min/max buffer passed to the quantization function(s)
164: constexpr int kRowwiseMinMaxNumCols = 2;
165:
166: bool _validate_rowwise_min_max(
167:   const at::Tensor& weight,
168:   const std::optional<at::Tensor>& rowwise_min_max_opt) {
169:   const auto is_valid_rowwise_min_max = rowwise_min_max_opt.has_value();
170:
171:   if (is_valid_rowwise_min_max) {
172:       TORCH_CHECK(
173:         (rowwise_min_max_opt->dim() == 2 &&
174:         rowwise_min_max_opt->size(0) == weight.size(0) &&
175:         rowwise_min_max_opt->size(1) == kRowwiseMinMaxNumCols),
176:         "'rowwise_min_max' must be a 2D tensor with shape [num_rows(weight), 2].");
177:   }
178:
179:   return is_valid_rowwise_min_max;
180: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_validate_rowwise_min_max`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_validate_rowwise_min_max`，它们直接构成本文件的算子逻辑。

### Lines 182-200
```cpp
182: auto _get_rowwise_min_max_contig(
183:   const std::optional<at::Tensor>& rowwise_min_max_opt) {
184:     return rowwise_min_max_opt.has_value()
185:       ? rowwise_min_max_opt->expect_contiguous(rowwise_min_max_opt->suggest_memory_format())
186:       : at::borrow_from_optional_tensor(rowwise_min_max_opt);
187: }
188: }
189: #endif // USE_FBGEMM
190:
191: namespace at::native {
192:
193: // Note - This is a temporary pack function for embedding bag which quantizes
194: // and packs the float weight tensor. In the next step it will be replaced by a
195: // quantize and pack function once we support FP scale and FP zero_point
196: //
197: // The optional rowwise_min_max argument is to support callers to pass in the min/max
198: // values of the weight tensor. If the rowwise_min_max is not provided, the min/max
199: // values will be computed from the weight tensor.
200: //
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_get_rowwise_min_max_contig`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_get_rowwise_min_max_contig`，它们直接构成本文件的算子逻辑。

### Lines 201-220
```cpp
201: // Python example examining a packed 8bit zero_point and scale:
202: //
203: // >> x = torch.from_numpy(np.array([[[10, 20], [30, 40]],[[50, 60], [70, 80]]],
204: // dtype=np.float32))
205: // >> x_packed = torch.ops.quantized.embedding_bag_byte_prepack(x)
206: //
207: // # Pull out and examine packed scales, zero_points and values
208: // >> zero_points = x_packed[:,:,-4:].numpy()
209: // >> scales = x_packed[:,:,-8:-4].numpy()
210: // >> values = x_packed[:,:,:-8].numpy()
211: //
212: // >> zero_points
213: // array([[[  0,   0,  32,  65],
214: //        [  0,   0, 240,  65]],
215: //
216: //       [[  0,   0,  72,  66],
217: //        [  0,   0, 140,  66]]], dtype=uint8)
218: //
219: // >> scales
220: // array([[[161, 160,  32,  61],
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 221-240
```cpp
221: //        [161, 160,  32,  61]],
222: //
223: //       [[161, 160,  32,  61],
224: //        [161, 160,  32,  61]]], dtype=uint8)
225: // >> values
226: // array([[[  0, 255],
227: //        [  0, 255]],
228: //
229: //       [[  0, 255],
230: //        [  0, 255]]], dtype=uint8)
231: //
232: // # Convert 4 byte packed scales and zero_points to float
233: // # and apply against values in order to recover unquantized values.
234: // def bytes2float(arr):
235: //    packed_hex = bytearray(arr)
236: //    return struct.unpack('f', packed_hex)
237: //
238: // >> float_zero_points = np.apply_along_axis(bytes2float, 2, zero_points)
239: // >> float_zero_points
240: // array([[[10.],
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 241-260
```cpp
241: //         [30.]],
242: //
243: //        [[50.],
244: //         [70.]]])
245: // >> float_scales = np.apply_along_axis(bytes2float, 2, scales)
246: // >> float_scales
247: // array([[[0.03921569],
248: //        [0.03921569]],
249: //
250: //       [[0.03921569],
251: //        [0.03921569]]])
252: // >> values *  float_scales + float_zero_points
253: // array([[[10.        , 20.00000035],
254: //         [30.        , 40.00000035]],
255: //
256: //        [[50.        , 60.00000035],
257: //         [70.        , 80.00000035]]])
258: Tensor& qembeddingbag_byte_prepack_out(
259:     Tensor& output,
260:     const Tensor& weight,
```
- EN: The main symbol in this range is `qembeddingbag_byte_prepack_out`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `qembeddingbag_byte_prepack_out`，它们直接构成本文件的算子逻辑。

### Lines 261-287
```cpp
261:     const std::optional<Tensor>& rowwise_min_max_opt) {
262:   // The "last" dimension of an N-Dimensioned batch of embedding bags is
263:   // quantization channel. E.g. for a 2D embedding bag, this has
264:   // [ row, col ] dimensions, for batched of embedding bags, dimensions might be
265:   // [ batch, row, col ].
266:   //
267:   // Python Batched Embedding Example:
268:   // weights = torch.from_numpy((np.random.random_sample((
269:   //          2, 10, 3)).squeeze() + 1).astype(np.float32))
270:   // assert(weights.size() == torch.Size([2, 10, 3]))
271:   // # NOTE: 8 bytes (columns) are added due to fp32 zero_point and scales
272:   // packed_weights = torch.ops.quantized.embedding_bag_byte_prepack(weights)
273:   // assert(packed_weights.size() == torch.Size([2, 10, 11]))
274:
275:   TORCH_CHECK(
276:       weight.scalar_type() == at::ScalarType::Float ||
277:           weight.scalar_type() == at::ScalarType::Half,
278:       "'embedding_bag_byte_prepack' only support float32 or float16.");
279:
280:   const auto weight_sizes = weight.sizes();
281:   const auto cols_dim = weight_sizes.size() - 1;
282:   const int64_t embedding_rows = c10::size_to_dim_(static_cast<int>(cols_dim), weight_sizes);
283:   const int32_t embedding_cols = static_cast<int32_t>(weight_sizes[cols_dim]);
284:   // Add 8 bytes per column to store FP32 scale and zero_point per row.
285:   const int32_t output_columns = static_cast<int32_t>(embedding_cols + 2 * sizeof(float));
286:   const auto weight_contig =
287:       weight.expect_contiguous(weight.suggest_memory_format());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 289-307
```cpp
289:   // Adjust output dimensions to account for FP32 scale and zero_points.
290:   std::vector<int64_t> output_shape = weight_sizes.vec();
291:   output_shape.at(cols_dim) = output_columns;
292:   at::native::resize_(output, output_shape, std::nullopt);
293:   auto* output_data = output.data_ptr<uint8_t>();
294:
295: #ifdef USE_FBGEMM
296:   // Move these outside of the ifdef when we support non-FBGEMM flow.
297:   const auto is_valid_rowwise_min_max = _validate_rowwise_min_max(weight, rowwise_min_max_opt);
298:   const auto rowwise_min_max_contig = _get_rowwise_min_max_contig(rowwise_min_max_opt);
299:
300:   if (weight_contig->scalar_type() == at::ScalarType::Half) {
301:     const auto weight_data =
302:         static_cast<fbgemm::float16*>(weight_contig->data_ptr());
303:     const auto rowwise_min_max_data = is_valid_rowwise_min_max
304:         ? static_cast<fbgemm::float16*>(rowwise_min_max_contig->data_ptr())
305:         : nullptr;
306:     at::parallel_for(
307:         0, embedding_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 308-329
```cpp
308:           fbgemm::FloatOrHalfToFused8BitRowwiseQuantizedSBFloat<
309:               fbgemm::float16>(
310:               weight_data + start_idx * embedding_cols,
311:               end_idx - start_idx,
312:               embedding_cols,
313:               output_data + start_idx * output_columns,
314:               (is_valid_rowwise_min_max ? (rowwise_min_max_data + start_idx * kRowwiseMinMaxNumCols) : nullptr));
315:         });
316:   } else {
317:     const auto weight_data = weight_contig->data_ptr<float>();
318:     const auto rowwise_min_max_data =
319:         is_valid_rowwise_min_max ? rowwise_min_max_contig->data_ptr<float>() : nullptr;
320:     at::parallel_for(
321:         0, embedding_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
322:           fbgemm::FloatOrHalfToFused8BitRowwiseQuantizedSBFloat<float>(
323:               weight_data + start_idx * embedding_cols,
324:               end_idx - start_idx,
325:               embedding_cols,
326:               output_data + start_idx * output_columns,
327:               (is_valid_rowwise_min_max ? (rowwise_min_max_data + start_idx * kRowwiseMinMaxNumCols) : nullptr));
328:         });
329:   }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 331-358
```cpp
331: #else
332:   const Tensor& float_weight =
333:       weight_contig->scalar_type() == at::ScalarType::Half
334:       ? weight_contig->to(at::ScalarType::Float)
335:       : *weight_contig;
336:   const auto weight_data = float_weight.data_ptr<float>();
337:   constexpr float kEpsilon = 1e-8f;
338:   for (auto row : c10::irange(embedding_rows)) {
339:     const float* input_row = weight_data + row * embedding_cols;
340:     std::uint8_t* output_row = output_data + row * output_columns;
341:     float* output_row_scale_zp =
342:         reinterpret_cast<float*>(output_row + embedding_cols);
343:
344:     float minimum_element =
345:         *std::min_element(input_row, input_row + embedding_cols);
346:     float maximum_element =
347:         *std::max_element(input_row, input_row + embedding_cols);
348:     float range = maximum_element - minimum_element;
349:
350:     output_row_scale_zp[0] = range / 255.0f;
351:     output_row_scale_zp[1] = minimum_element;
352:     const auto inverse_scale = 255.0f / (range + kEpsilon);
353:     for (auto col : c10::irange(embedding_cols)) {
354:       output_row[col] =
355:           lrintf((input_row[col] - minimum_element) * inverse_scale);
356:     } // embedding_cols
357:   } // embedding_rows
358: #endif // USE_FBGEMM
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 360-378
```cpp
360:   return output;
361: }
362:
363: Tensor qembeddingbag_byte_prepack(const Tensor& weight) {
364:   const auto weight_contig =
365:       weight.expect_contiguous(weight.suggest_memory_format());
366:   Tensor output = at::detail::empty_cpu(
367:       {0},
368:       at::kByte,
369:       weight_contig->layout(),
370:       weight_contig->device(),
371:       std::nullopt,
372:       std::nullopt);
373:   qembeddingbag_byte_prepack_out(output, weight);
374:   return output;
375: }
376:
377: static Tensor qembeddingbag_byte_prepack_with_rowwise_min_max(
378:     const Tensor& weight,
```
- EN: The main symbol in this range is `qembeddingbag_byte_prepack`, `empty_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qembeddingbag_byte_prepack`, `empty_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 379-404
```cpp
379:     const Tensor& rowwise_min_max) {
380:   const auto weight_contig =
381:       weight.expect_contiguous(weight.suggest_memory_format());
382:   Tensor output = at::detail::empty_cpu(
383:       {0},
384:       at::kByte,
385:       weight_contig->layout(),
386:       weight_contig->device(),
387:       std::nullopt,
388:       std::nullopt);
389:   qembeddingbag_byte_prepack_out(output, weight, rowwise_min_max);
390:   return output;
391: }
392:
393: Tensor qembeddingbag_byte_prepack_meta(const Tensor& weight) {
394:   const auto weight_contig =
395:       weight.expect_contiguous(weight.suggest_memory_format());
396:   TORCH_CHECK(
397:       weight.scalar_type() == at::ScalarType::Float ||
398:           weight.scalar_type() == at::ScalarType::Half,
399:       "'embedding_bag_byte_prepack' only support float32 or float16.");
400:   const auto weight_sizes = weight.sym_sizes();
401:   const auto cols_dim = weight.ndimension() - 1;
402:   const auto& embedding_cols = weight_sizes[cols_dim];
403:   // Add 8 bytes per column to store FP32 scale and zero_point per row.
404:   const auto output_columns = embedding_cols + 2 * c10::SymInt(sizeof(float));
```
- EN: The main symbol in this range is `qembeddingbag_byte_prepack_meta`, `empty_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qembeddingbag_byte_prepack_meta`, `empty_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 406-431
```cpp
406:   // Adjust output dimensions to account for FP32 scale and zero_points.
407:   auto output_shape = weight_sizes.vec();
408:   output_shape.at(cols_dim) = output_columns;
409:   at::SymDimVector output_shape_vec(output_shape);
410:
411:   return at::empty_symint(
412:       output_shape_vec,
413:       weight.options().dtype(weight.scalar_type()),
414:       weight.suggest_memory_format());
415: }
416:
417: namespace {
418:
419: // TODO: Extend support to N-D batched embeddings, similar to
420: // qembeddingbag_byte_prepack
421: Tensor _qembeddingbag_nbit_prepack_helper(
422:     const Tensor& weight,
423:     int bit_width,
424:     const bool optimized_qparams,
425:     const int64_t nbins,
426:     const double ratio,
427:     const std::optional<Tensor>& rowwise_min_max_opt = std::nullopt) {
428:   TORCH_CHECK(
429:       weight.scalar_type() == at::ScalarType::Float ||
430:           weight.scalar_type() == at::ScalarType::Half,
431:       "'qembeddingbag_nbit_prepack' only support float32 or float16.");
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_qembeddingbag_nbit_prepack_helper`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_qembeddingbag_nbit_prepack_helper`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 433-451
```cpp
433:   int64_t embedding_rows = weight.size(0);
434:   int64_t embedding_cols = weight.size(1);
435:
436:   Tensor weight_contig = weight.contiguous(weight.suggest_memory_format());
437:
438:   TORCH_CHECK(
439:       bit_width == 4 || bit_width == 2,
440:       "bit_width must be either 2 or 4 to use 'qembeddingbag_nbit_prepack'."
441:       "For 8bit, consider using 'embedding_bag_byte_prepack'.");
442:
443:   int NUM_ELEM_PER_BYTE = 8 / bit_width;
444:   TORCH_CHECK(
445:       weight_contig.size(weight.dim() - 1) % NUM_ELEM_PER_BYTE == 0,
446:       "qembeddingbag_",
447:       std::to_string(bit_width),
448:       "bit_prepack only works for the number of columns a multiple of ",
449:       std::to_string(NUM_ELEM_PER_BYTE));
450:
451:   // The "fused" representation stores the scale and bias with the
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 452-471
```cpp
452:   // row-wise quantized data in one tensor.
453:   // Since we represent the scale and bias in 16-bit float, we'll use the
454:   // last 4 bytes of each row for scale (2 bytes) and bias (2 bytes).
455:   // | ... quantized data ... | scale | bias |
456:   // |    number_of_columns   |  2B   |  2B  |
457:   std::vector<int64_t> output_shape = {
458:       embedding_rows,
459:       static_cast<std::int64_t>(
460:           (embedding_cols + NUM_ELEM_PER_BYTE - 1) / NUM_ELEM_PER_BYTE +
461:           2 * sizeof(at::Half))};
462:   auto output = at::empty(
463:       output_shape,
464:       weight_contig.options().dtype(at::kByte),
465:       weight_contig.suggest_memory_format());
466:   auto* output_data = output.data_ptr<uint8_t>();
467:
468: #ifdef USE_FBGEMM
469:   // Move these outside of the ifdef when we support non-FBGEMM flow.
470:   const auto is_valid_rowwise_min_max = _validate_rowwise_min_max(weight, rowwise_min_max_opt);
471:   const auto rowwise_min_max_contig = _get_rowwise_min_max_contig(rowwise_min_max_opt);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 473-491
```cpp
473:   if (!optimized_qparams) {
474:     if (weight_contig.scalar_type() == at::ScalarType::Half) {
475:       const auto weight_data =
476:           static_cast<fbgemm::float16*>(weight_contig.data_ptr());
477:       const auto rowwise_min_max_data = is_valid_rowwise_min_max
478:           ? static_cast<fbgemm::float16*>(rowwise_min_max_contig->data_ptr())
479:           : nullptr;
480:       at::parallel_for(
481:           0, embedding_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
482:             fbgemm::FloatOrHalfToFusedNBitRowwiseQuantizedSBHalf<
483:                 fbgemm::float16>(
484:                 bit_width,
485:                 weight_data + start_idx * embedding_cols,
486:                 end_idx - start_idx,
487:                 static_cast<int>(embedding_cols),
488:                 output_data + start_idx * output_shape[1],
489:                 (is_valid_rowwise_min_max ? (rowwise_min_max_data + start_idx * kRowwiseMinMaxNumCols) : nullptr));
490:           });
491:     } else {
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 492-516
```cpp
492:       const auto weight_data = weight_contig.data_ptr<float>();
493:       const auto rowwise_min_max_data =
494:           is_valid_rowwise_min_max ? rowwise_min_max_contig->data_ptr<float>() : nullptr;
495:       at::parallel_for(
496:           0, embedding_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
497:             fbgemm::FloatOrHalfToFusedNBitRowwiseQuantizedSBHalf<float>(
498:                 bit_width,
499:                 weight_data + start_idx * embedding_cols,
500:                 end_idx - start_idx,
501:                 static_cast<int>(embedding_cols),
502:                 output_data + start_idx * output_shape[1],
503:                 (is_valid_rowwise_min_max ? (rowwise_min_max_data + start_idx * kRowwiseMinMaxNumCols) : nullptr));
504:           });
505:     }
506:   } else {
507: #endif // USE_FBGEMM
508:     const auto output_columns = output.size(output.dim() - 1);
509:     const auto float_weight =
510:         weight_contig.scalar_type() == at::ScalarType::Half
511:         ? weight_contig.to(at::ScalarType::Float)
512:         : std::move(weight_contig);
513:     const auto weight_data = float_weight.data_ptr<float>();
514:     for (const auto row : c10::irange(embedding_rows)) {
515:       const float* input_row = weight_data + row * embedding_cols;
516:       std::uint8_t* output_row = output_data + row * output_columns;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 518-536
```cpp
518:       float Xmin = std::numeric_limits<float>::quiet_NaN(), Xmax = std::numeric_limits<float>::quiet_NaN();
519:       if (optimized_qparams) {
520:         auto [xmax_tensor, xmin_tensor] = at::choose_qparams_optimized(
521:             float_weight[row], embedding_cols, nbins, ratio, bit_width);
522:         TORCH_CHECK(
523:             xmax_tensor.numel() == 1 && xmin_tensor.numel() == 1,
524:             "Expected choose_qparams_optimized to return min/max tensors of size 1");
525:         Xmax = xmax_tensor.item<float>();
526:         Xmin = xmin_tensor.item<float>();
527:       } else {
528:         Xmin = *std::min_element(input_row, input_row + embedding_cols);
529:         Xmax = *std::max_element(input_row, input_row + embedding_cols);
530:       }
531:       Xmin = static_cast<at::Half>(Xmin);
532:       float range = Xmax - Xmin;
533:       // Set scale to 1.0f for the corner case of Xmax == Xmin .
534:       // Any non-zero scale would work because during quantization
535:       // (X - Xmin) / scale will be 0 for all X unless scale is 0.
536:       // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 537-556
```cpp
537:       at::Half scale = range == 0 ? 1.0f : range / ((1 << bit_width) - 1);
538:       float inverse_scale = scale == 0 ? 1.0f : 1.0f / scale;
539:       if (scale == 0 || std::isinf(inverse_scale)) {
540:         // Corner case handling when Xmax == Xmin
541:         // Any scale would work because X - Xmin will be 0 for all X
542:         scale = 1.0f;
543:         inverse_scale = 1.0f;
544:       }
545:       // Update the scale and zero_point of each row.
546:       at::Half* output_row_scale_zp = reinterpret_cast<at::Half*>(
547:           output_row +
548:           (embedding_cols + NUM_ELEM_PER_BYTE - 1) / NUM_ELEM_PER_BYTE);
549:
550:       output_row_scale_zp[0] = scale;
551:       output_row_scale_zp[1] = Xmin;
552:
553:       // Pack the weight values.
554:       for (const auto col : c10::irange(embedding_cols)) {
555:         float X = input_row[col];
556:         std::uint8_t quantized = std::max(
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 557-575
```cpp
557:             0,
558:             std::min<int>(
559:                 static_cast<int>(lrintf((X - Xmin) * inverse_scale)), (1 << bit_width) - 1));
560:         // We pack 2 4-bit values in a byte. Index 0 is packed in the lower
561:         // 4-bits and index 1 is packed in the upper 4-bits.
562:         if (col % NUM_ELEM_PER_BYTE == 0) {
563:           output_row[col / NUM_ELEM_PER_BYTE] = quantized;
564:         } else {
565:           output_row[col / NUM_ELEM_PER_BYTE] |=
566:               (quantized << ((col % NUM_ELEM_PER_BYTE) * bit_width));
567:         }
568:       } // embedding_cols
569:     } // embedding_rows
570: #ifdef USE_FBGEMM
571:   }
572: #endif // USE_FBGEMM
573:
574:   return output;
575: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 577-601
```cpp
577: // Applies 4-bit row-wise quantization by determining the range
578: // (maximum - minimum) and bias (minimum value) of each row in the input
579: // matrix, and then scaling each element to an 2-bit number between 0 and
580: // 15.
581: // To later de-quantize values, the scale (range / 15) and zero_point
582: // are stored alongside the data. More precisely, each row first has quantized
583: // values, and then 2-byte fp16 scale and 2-byte zero_offset.
584: Tensor qembeddingbag_4bit_prepack(
585:     const Tensor& weight,
586:     const bool optimized_qparams,
587:     const int64_t nbins,
588:     const double ratio) {
589:   return _qembeddingbag_nbit_prepack_helper(
590:       weight, 4 /*bit_width*/, optimized_qparams, nbins, ratio);
591: }
592:
593: Tensor qembeddingbag_4bit_prepack_with_rowwise_min_max(
594:     const Tensor& weight,
595:     const Tensor& rowwise_min_max,
596:     const bool optimized_qparams,
597:     const int64_t nbins,
598:     const double ratio) {
599:   return _qembeddingbag_nbit_prepack_helper(
600:       weight, 4 /*bit_width*/, optimized_qparams, nbins, ratio, rowwise_min_max);
601: }
```
- EN: The main symbol in this range is `bias`, `qembeddingbag_4bit_prepack_with_rowwise_min_max`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `bias`, `qembeddingbag_4bit_prepack_with_rowwise_min_max`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 603-628
```cpp
603: // Applies 2-bit row-wise quantization by determining the range
604: // (maximum - minimum) and bias (minimum value) of each row in the input
605: // matrix, and then scaling each element to an 2-bit number between 0 and
606: // 3.
607: // To later de-quantize values, the scale (range / 3) and zero_point
608: // are stored alongside the data. More precisely, each row first has quantized
609: // values, and then 2-byte fp16 scale and 2-byte zero_offset.
610: // TODO() - Add 2Bit Embedding Lookup operator.
611: Tensor qembeddingbag_2bit_prepack(
612:     const Tensor& weight,
613:     const bool optimized_qparams,
614:     const int64_t nbins,
615:     const double ratio) {
616:   return _qembeddingbag_nbit_prepack_helper(
617:       weight, 2 /*bit_width*/, optimized_qparams, nbins, ratio);
618: }
619:
620: Tensor qembeddingbag_2bit_prepack_with_rowwise_min_max(
621:     const Tensor& weight,
622:     const Tensor& rowwise_min_max,
623:     const bool optimized_qparams,
624:     const int64_t nbins,
625:     const double ratio) {
626:   return _qembeddingbag_nbit_prepack_helper(
627:       weight, 2 /*bit_width*/, optimized_qparams, nbins, ratio, rowwise_min_max);
628: }
```
- EN: The main symbol in this range is `bias`, `qembeddingbag_2bit_prepack_with_rowwise_min_max`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `bias`, `qembeddingbag_2bit_prepack_with_rowwise_min_max`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 630-656
```cpp
630: class QEmbeddingPackWeights final {
631:  public:
632:   static c10::intrusive_ptr<EmbeddingPackedParamsBase> run(const at::Tensor& weight) {
633:     return PackedEmbeddingBagWeight::prepack(weight);
634:   }
635: };
636:
637: TORCH_LIBRARY_IMPL(quantized, CPU, m) {
638:   m.impl(
639:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_byte_prepack"),
640:       TORCH_FN(qembeddingbag_byte_prepack));
641:   m.impl(
642:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_byte_prepack_with_rowwise_min_max"),
643:       TORCH_FN(qembeddingbag_byte_prepack_with_rowwise_min_max));
644:   m.impl(
645:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_4bit_prepack"),
646:       TORCH_FN(qembeddingbag_4bit_prepack));
647:   m.impl(
648:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_4bit_prepack_with_rowwise_min_max"),
649:       TORCH_FN(qembeddingbag_4bit_prepack_with_rowwise_min_max));
650:   m.impl(
651:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_2bit_prepack"),
652:       TORCH_FN(qembeddingbag_2bit_prepack));
653:   m.impl(
654:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_2bit_prepack_with_rowwise_min_max"),
655:       TORCH_FN(qembeddingbag_2bit_prepack_with_rowwise_min_max));
656: }
```
- EN: The main symbol in this range is `run`, `QEmbeddingPackWeights`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QEmbeddingPackWeights`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 658-670
```cpp
658: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
659:   m.impl(
660:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_prepack"),
661:       TORCH_FN(QEmbeddingPackWeights::run));
662: }
663:
664: TORCH_LIBRARY_IMPL(quantized, Meta, m) {
665:   m.impl(
666:       "quantized::embedding_bag_byte_prepack", qembeddingbag_byte_prepack_meta);
667: }
668:
669: } // namespace
670: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

## Key Concepts / 关键概念

- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/quantized/cpu/qembeddingbag_prepack.h`, `ATen/Parallel.h`, `ATen/Utils.h`, `ATen/core/Tensor.h`, `ATen/core/custom_class.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`, `ATen/native/quantized/cpu/fbgemm_utils.h`, `ATen/native/quantized/library.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`
- c10 headers / c10 头文件: `c10/core/ScalarType.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `utility`
- Key helper symbols / 关键辅助符号: `parallel_for`, `Scalar`, `ScalarType`, `fbgemm`
