# qembeddingbag_unpack.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qembeddingbag_unpack.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU embedding lookup or embedding-bag accumulation logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU embedding 查找或 embedding-bag 累加逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Parallel.h>
 4: #include <ATen/native/quantized/cpu/EmbeddingPackedParams.h>
 5: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
 6: #include <ATen/native/quantized/cpu/qembeddingbag.h>
 7: #include <ATen/native/quantized/library.h>
 8: #include <c10/util/irange.h>
 9: #include <torch/library.h>
10:
11: #ifndef AT_PER_OPERATOR_HEADERS
12: #include <ATen/Functions.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Parallel.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Parallel.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 13-23
```cpp
13: #include <ATen/NativeFunctions.h>
14: #else
15: #include <ATen/ops/_empty_per_channel_affine_quantized.h>
16: #include <ATen/ops/empty.h>
17: #include <ATen/ops/from_blob.h>
18: #include <ATen/ops/resize_native.h>
19: #endif
20:
21: at::Tensor PackedEmbeddingBagWeight::unpack() {
22:   auto packed_weight = packed_w;
23:   at::Tensor weight_origin;
```
- EN: This range pulls in required headers, including `ATen/NativeFunctions.h`, `ATen/ops/_empty_per_channel_affine_quantized.h`, `ATen/ops/empty.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `unpack`, which contributes directly to this file's operator logic.
- CN: 这一段引入了所需头文件，例如 `ATen/NativeFunctions.h`, `ATen/ops/_empty_per_channel_affine_quantized.h`, `ATen/ops/empty.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `unpack`，它们直接构成本文件的算子逻辑。

### Lines 25-36
```cpp
25:   if (bit_rate_ == 8 || bit_rate_ == 4) {
26:     const auto input_rows = packed_weight.size(0);
27:     const auto input_columns = packed_weight.size(1);
28:     int scale_bias_bytes = 0;
29:     const auto num_elem_per_byte = 8 / bit_rate_;
30:     if (bit_rate_ == 8) {
31:       // The last 2 values are used to store the FP32 scale and zero_point
32:       // values per row.
33:       scale_bias_bytes = 8;
34:     } else {
35:       scale_bias_bytes = 4;
36:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 38-49
```cpp
38:     const auto* input = packed_weight.const_data_ptr<uint8_t>();
39:     // Calculate the output shape, accounting for the last n bytes to be used
40:     // for scale/bias rest of the entries are packed depending on the bit_width.
41:     std::vector<int64_t> output_shape = {
42:         input_rows,
43:         static_cast<std::int64_t>(input_columns - scale_bias_bytes) *
44:             num_elem_per_byte};
45:
46:     auto scales = at::from_blob(
47:         w_scale.data(), w_scale.size(), at::device(c10::kCPU).dtype(c10::kFloat));
48:     auto zero_points = at::from_blob(
49:         w_zp.data(), w_zp.size(), at::device(c10::kCPU).dtype(c10::kFloat));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 51-61
```cpp
51:     auto output_columns = output_shape[1];
52:     uint8_t* output_data = nullptr;
53:
54:     // Allocate output weight tensor based on the bit_width
55:     if (bit_rate_ == 8) {
56:       weight_origin = at::_empty_per_channel_affine_quantized(
57:           output_shape,
58:           scales.toType(c10::kFloat),
59:           zero_points.toType(c10::kFloat),
60:           0, // The output channel axis is 0
61:           at::device(c10::kCPU).dtype(c10::kQUInt8));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 62-74
```cpp
62:       output_data = static_cast<uint8_t*>(weight_origin.data_ptr());
63:     } else {
64:       // We create empty qtensor with the full output shape, and dtype set to
65:       // quint4x2 This will internally allocate appropriate storage bytes to
66:       // account for the packed nature of this dtype.
67:       weight_origin = at::_empty_per_channel_affine_quantized(
68:           output_shape,
69:           scales.toType(c10::kFloat),
70:           zero_points.toType(c10::kFloat),
71:           0, // The output channel axis is 0
72:           at::device(c10::kCPU).dtype(c10::kQUInt4x2));
73:       output_data = static_cast<uint8_t*>(weight_origin.data_ptr());
74:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 76-90
```cpp
76:     // Copy over the data from the packed weight to the output.
77:     // For sub-byte tensors this will copy the packed bytes over since the
78:     // sub_byte qtensors are expected to store data in packed format.
79:     at::parallel_for(0, input_rows, 1, [&](int32_t start_idx, int32_t end_idx) {
80:       for (const auto row : c10::irange(start_idx, end_idx)) {
81:         const std::uint8_t* input_row = input + row * input_columns;
82:         uint8_t* output_row =
83:             output_data + row * output_columns / num_elem_per_byte;
84:
85:         // output_columns
86:         for (const auto col : c10::irange(output_columns / num_elem_per_byte)) {
87:           output_row[col] = input_row[col];
88:         }
89:       }
90:     });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 92-102
```cpp
 92:     return weight_origin;
 93:   }
 94:   TORCH_INTERNAL_ASSERT(
 95:       false,
 96:       "We currently only support 8-bit and 4-bit quantization of embedding_bag.");
 97: }
 98:
 99: namespace at::native {
100:
101: Tensor& qembeddingbag_byte_unpack_out(Tensor& output, const Tensor& packed_weight) {
102:   // The "last" dimension of an N-Dimensioned batch of embedding bags is
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `qembeddingbag_byte_unpack_out`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `qembeddingbag_byte_unpack_out`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 103-114
```cpp
103:   // quantization channel. E.g. for a 2D embedding bag, this has
104:   // [ row, col ] dimensions, for batched of embedding bags, dimensions might be
105:   // [ batch, row, col ].
106:   //
107:   // Python Batched Embedding Example:
108:   // weights = torch.from_numpy((np.random.random_sample((
109:   //          2, 10, 3)).squeeze() + 1).astype(np.float32))
110:   // assert(weights.size() == torch.Size([2, 10, 3]))
111:   // # NOTE: 8 bytes (columns) are added due to fp32 zero_point and scales
112:   // packed_weights = torch.ops.quantized.embedding_bag_byte_prepack(weights)
113:   // assert(packed_weights.size() == torch.Size([2, 10, 11]))
114:   // unpacked_weights = torch.ops.quantized.embedding_bag_byte_unpack(packed_weights)
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 115-129
```cpp
115:   // assert(unpacked_weights.size() == torch.Size([2, 10, 3]))
116:   const auto packed_weight_sizes = packed_weight.sizes();
117:   const auto col_dim = packed_weight_sizes.size() - 1;
118:   const int64_t input_rows = c10::size_to_dim_(col_dim, packed_weight_sizes);
119:   const int32_t input_columns = packed_weight_sizes[col_dim];
120:   // The last 2 values are used to store the FP32 scale and zero_point values
121:   // per row.
122:   const int32_t output_columns = input_columns - 2 * sizeof(float);
123:   const auto* input_data = packed_weight.const_data_ptr<uint8_t>();
124:
125:   std::vector<int64_t> output_shape = packed_weight_sizes.vec();
126:   output_shape[col_dim] = output_columns;
127:   at::native::resize_(output, output_shape);
128:   auto output_contig = output.expect_contiguous();
129:   float* output_data = output_contig->data_ptr<float>();
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 131-144
```cpp
131: #ifdef USE_FBGEMM
132:   at::parallel_for(0, input_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
133:     fbgemm::Fused8BitRowwiseQuantizedSBFloatToFloatOrHalf<float>(
134:         input_data + start_idx * input_columns,
135:         end_idx - start_idx,
136:         input_columns,
137:         output_data + start_idx * output_columns);
138:   });
139: #else
140:   for (auto row : c10::irange(input_rows)) {
141:     const std::uint8_t* input_row = input_data + row * input_columns;
142:     const float* input_row_scale_zp =
143:         reinterpret_cast<const float*>(input_row + output_columns);
144:     float* output_row = output_data + row * output_columns;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 146-156
```cpp
146:     for (auto col : c10::irange(output_columns)) {
147:       output_row[col] =
148:           input_row[col] * input_row_scale_zp[0] + input_row_scale_zp[1];
149:     } // output_columns
150:   } // input_rows
151: #endif // USE_FBGEMM
152:   return output;
153: }
154:
155: namespace {
156: Tensor qembeddingbag_byte_unpack(const Tensor& packed_weight) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `qembeddingbag_byte_unpack`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `qembeddingbag_byte_unpack`，它们直接构成本文件的算子逻辑。

### Lines 157-171
```cpp
157:   at::Tensor output = at::empty(
158:       {},
159:       packed_weight.options().dtype(kFloat),
160:       packed_weight.suggest_memory_format());
161:   qembeddingbag_byte_unpack_out(output, packed_weight);
162:   return output;
163: }
164:
165: Tensor qembeddingbag_byte_unpack_meta(const Tensor& packed_weight) {
166:   const auto packed_weight_sizes = packed_weight.sym_sizes();
167:   const auto col_dim = packed_weight_sizes.size() - 1;
168:   const auto input_columns = packed_weight_sizes[col_dim];
169:   // The last 2 values are used to store the FP32 scale and zero_point values
170:   // per row.
171:   const auto output_columns = input_columns - 2 * c10::SymInt(sizeof(float));
```
- EN: The main symbol in this range is `qembeddingbag_byte_unpack_meta`, `qembeddingbag_byte_unpack_out`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qembeddingbag_byte_unpack_meta`, `qembeddingbag_byte_unpack_out`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 173-186
```cpp
173:   auto output_shape = packed_weight_sizes.vec();
174:   output_shape[col_dim] = output_columns;
175:
176:   at::SymDimVector output_shape_vec(output_shape);
177:   return at::empty_symint(output_shape_vec, packed_weight.options().dtype(kFloat), packed_weight.suggest_memory_format());
178: }
179:
180: Tensor _qembeddingbag_nbit_unpack_helper(
181:     const Tensor& packed_weight,
182:     int BIT_RATE) {
183:   const auto input_rows = packed_weight.size(0);
184:   const auto input_columns = packed_weight.size(1);
185:   const auto* input_data = packed_weight.const_data_ptr<uint8_t>();
186:   int NUM_ELEM_PER_BYTE = 8 / BIT_RATE;
```
- EN: The main symbol in this range is `_qembeddingbag_nbit_unpack_helper`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_qembeddingbag_nbit_unpack_helper`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 188-198
```cpp
188:   // The last 4 bytes per row are two fp16 scale and zero_point.
189:   // The rest of input_columns is the number of values in the original row.
190:   std::vector<int64_t> output_dimensions = {
191:       input_rows,
192:       static_cast<std::int64_t>(input_columns - 2 * sizeof(at::Half)) *
193:           NUM_ELEM_PER_BYTE};
194:
195:   auto output = at::empty(
196:       output_dimensions,
197:       packed_weight.options().dtype(kFloat),
198:       packed_weight.suggest_memory_format());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 199-210
```cpp
199:   float* output_data = output.data_ptr<float>();
200: #ifdef USE_FBGEMM
201:   at::parallel_for(0, input_rows, 1, [&](int64_t start_idx, int64_t end_idx) {
202:     fbgemm::FusedNBitRowwiseQuantizedSBHalfToFloatOrHalf<float>(
203:         BIT_RATE,
204:         input_data + start_idx * input_columns,
205:         end_idx - start_idx,
206:         input_columns,
207:         output_data + start_idx * output_dimensions[1]);
208:   });
209: #else
210:   auto output_columns = output_dimensions[1];
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 211-227
```cpp
211:   for (auto row : c10::irange(input_rows)) {
212:     float* output_row = output_data + row * output_columns;
213:     const std::uint8_t* input_row = input_data + row * input_columns;
214:     const at::Half* input_row_scale_zp = reinterpret_cast<const at::Half*>(
215:         input_row +
216:         (output_columns + NUM_ELEM_PER_BYTE - 1) / NUM_ELEM_PER_BYTE);
217:     float scale = input_row_scale_zp[0];
218:     float zero_point = input_row_scale_zp[1];
219:
220:     for (const auto col : c10::irange(output_columns)) {
221:       std::uint8_t quantized = input_row[col / NUM_ELEM_PER_BYTE];
222:       quantized >>= (col % NUM_ELEM_PER_BYTE) * BIT_RATE;
223:       quantized &= (1 << BIT_RATE) - 1;
224:       output_row[col] = scale * quantized + zero_point;
225:     } // output_columns
226:   } // input_rows
227: #endif // USE_FBGEMM
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 229-242
```cpp
229:   return output;
230: }
231:
232: // De-quantizes the result of the qembeddingbag_4bit_prepack operator.
233: // The input is expected to first have quantized values,
234: // then 2-byte fp16 scale and 2-byte zero_offset.
235: // The output is a matrix containing only the values, but de-quantized.
236: // De-quantization is performed by multiplying each value by its
237: // row's scale and zero_point parameters. The de-quantized values
238: // will thus not be exactly equal to the original, un-quantized
239: // floating point values.
240: Tensor qembeddingbag_4bit_unpack(const Tensor& packed_weight) {
241:   return _qembeddingbag_nbit_unpack_helper(packed_weight, 4 /*BIT_RATE*/);
242: }
```
- EN: The main symbol in this range is `qembeddingbag_4bit_unpack`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qembeddingbag_4bit_unpack`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 244-254
```cpp
244: // De-quantizes the result of the qembeddingbag_2bit_prepack operator.
245: // The input is expected to first have quantized values,
246: // then 2-byte fp16 scale and 2-byte zero_offset.
247: // The output is a matrix containing only the values, but de-quantized.
248: // De-quantization is performed by multiplying each value by its
249: // row's scale and zero_point parameters. The de-quantized values
250: // will thus not be exactly equal to the original, un-quantized
251: // floating point values.
252: Tensor qembeddingbag_2bit_unpack(const Tensor& packed_weight) {
253:   return _qembeddingbag_nbit_unpack_helper(packed_weight, 2 /*BIT_RATE*/);
254: }
```
- EN: The main symbol in this range is `qembeddingbag_2bit_unpack`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qembeddingbag_2bit_unpack`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 256-266
```cpp
256: class QEmbeddingUnpackWeights final {
257:  public:
258:   static at::Tensor run(
259:       const c10::intrusive_ptr<EmbeddingPackedParamsBase>& packed_weight) {
260:     return packed_weight->unpack();
261:   }
262: };
263:
264: TORCH_LIBRARY_IMPL(quantized, CPU, m) {
265:   m.impl(
266:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_byte_unpack"),
```
- EN: The main symbol in this range is `run`, `QEmbeddingUnpackWeights`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QEmbeddingUnpackWeights`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 267-282
```cpp
267:       qembeddingbag_byte_unpack);
268:   m.impl(
269:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_4bit_unpack"),
270:       qembeddingbag_4bit_unpack);
271:   m.impl(
272:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_2bit_unpack"),
273:       qembeddingbag_2bit_unpack);
274: }
275:
276: TORCH_LIBRARY_IMPL(quantized, CatchAll, m) {
277:   // Unpack the packed embedding_bag weights using TorchBind custom class.
278:   // TODO extend to support 4-bit qtensor.
279:   m.impl(
280:       TORCH_SELECTIVE_NAME("quantized::embedding_bag_unpack"),
281:       TORCH_FN(QEmbeddingUnpackWeights::run));
282: }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 284-291
```cpp
284: TORCH_LIBRARY_IMPL(quantized, Meta, m) {
285:   m.impl(
286:       "quantized::embedding_bag_byte_unpack",
287:       qembeddingbag_byte_unpack_meta);
288: }
289:
290: } // namespace
291: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

## Key Concepts / 关键概念

- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Parallel.h`, `ATen/native/quantized/cpu/EmbeddingPackedParams.h`, `ATen/native/quantized/cpu/fbgemm_utils.h`, `ATen/native/quantized/cpu/qembeddingbag.h`, `ATen/native/quantized/library.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_per_channel_affine_quantized.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`
- Key helper symbols / 关键辅助符号: `parallel_for`, `fbgemm`
