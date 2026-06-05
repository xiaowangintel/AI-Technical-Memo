# AffineQuantizer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/AffineQuantizer.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
 1: #include <ATen/native/quantized/AffineQuantizer.h>
 2:
 3:
 4: namespace at::native {
 5:
 6: DEFINE_DISPATCH(quantize_tensor_per_tensor_affine_stub);
 7: DEFINE_DISPATCH(quantize_tensor_per_channel_affine_stub);
 8: DEFINE_DISPATCH(quantize_tensor_per_channel_float_qparams_stub);
 9: DEFINE_DISPATCH(dequantize_tensor_per_tensor_affine_stub);
10: DEFINE_DISPATCH(dequantize_tensor_per_channel_affine_stub);
11: DEFINE_DISPATCH(dequantize_tensor_per_channel_float_qparams_stub);
12: DEFINE_DISPATCH(quantize_tensor_per_tensor_affine_sub_byte_stub);
13: DEFINE_DISPATCH(dequantize_tensor_per_tensor_affine_sub_byte_stub);
```
- EN: This range pulls in required headers, including `ATen/native/quantized/AffineQuantizer.h`. The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/native/quantized/AffineQuantizer.h`。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 15-28
```cpp
15: namespace {
16:
17: void checkRoundingMode(const char* fn_name) {
18:   // Disabling this warning message for now as it is printed incorrectly. Need
19:   // to fix
20:
21:   /*  TORCH_WARN_ONCE(
22:         std::fegetround() != FE_TONEAREST,
23:         fn_name,
24:         " current rounding mode is not set to round-to-nearest-ties-to-even
25:      (FE_TONEAREST). This will cause accuracy issues in quantized models.");
26:   */
27:   return;
28: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `checkRoundingMode`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `checkRoundingMode`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 30-44
```cpp
30: void checkFloatTensor(const char* fn_name, const Tensor& t) {
31:   TORCH_CHECK(
32:       t.scalar_type() == kFloat, fn_name, " expects a Float Tensor, got ",
33:       t.scalar_type());
34: }
35:
36: void checkSameDevice(
37:     const char* fn_name,
38:     const Tensor& t1,
39:     const Tensor& t2) {
40:   TORCH_CHECK(
41:       t1.device() == t2.device(),
42:       fn_name,
43:       " expects a quantized and float tensors to be on the same device.");
44: }
```
- EN: The main symbol in this range is `checkFloatTensor`, `checkSameDevice`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `checkFloatTensor`, `checkSameDevice`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 46-56
```cpp
46: template <typename T>
47: void checkQuantizedTensor(const char* fn_name, const Tensor& t) {
48:   TORCH_CHECK(t.is_quantized(), fn_name, " expects a quantized Tensor.");
49:   TORCH_CHECK(
50:       t.scalar_type() == caffe2::TypeMeta::Make<T>(),
51:       fn_name,
52:       " expects a ",
53:       caffe2::TypeMeta::Make<T>(),
54:       " Tensor, got ",
55:       t.scalar_type());
56: }
```
- EN: The main symbol in this range is `checkQuantizedTensor`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `checkQuantizedTensor`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 58-72
```cpp
58: template <typename T>
59: void checkZeroPoint(const char* fn_name, int64_t zero_point) {
60:   TORCH_CHECK(
61:       zero_point <= std::numeric_limits<T>::max(),
62:       fn_name,
63:       " zero_point ",
64:       zero_point,
65:       " is above upper bound.");
66:   TORCH_CHECK(
67:       zero_point >= std::numeric_limits<T>::min(),
68:       fn_name,
69:       " zero_point ",
70:       zero_point,
71:       " is below lower bound.");
72: }
```
- EN: The main symbol in this range is `checkZeroPoint`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `checkZeroPoint`，它们直接构成本文件的算子逻辑。

### Lines 74-84
```cpp
74: template <typename T>
75: void checkZeroPoints(const char* fn_name, const Tensor& zero_points) {
76:   auto zero_points_data = zero_points.data_ptr<int64_t>();
77:   for (const auto i : c10::irange(zero_points.numel())) {
78:     checkZeroPoint<T>(fn_name, zero_points_data[i]);
79:   }
80: }
81:
82: void checkSameSize(
83:     const char* fn_name,
84:     const Tensor& qt,
```
- EN: The main symbol in this range is `checkZeroPoints`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `checkZeroPoints`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 85-96
```cpp
85:     const Tensor& rt) {
86:   TORCH_CHECK(
87:       qt.sizes().equals(rt.sizes()),
88:       fn_name,
89:       " only works with Tensors with the same shape");
90: }
91:
92: void checkPerChannelParamsSize(
93:     const Tensor& rtensor,
94:     int64_t axis,
95:     const Tensor& scales,
96:     const Tensor& zero_points
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 97-107
```cpp
 97: ) {
 98:   int64_t channel = rtensor.size(axis);
 99:   TORCH_CHECK(
100:       channel == int64_t(scales.numel()),
101:       "length of scales must equal to channel, expected ", channel, " got, ", scales.numel());
102:   TORCH_CHECK(
103:       channel == int64_t(zero_points.numel()),
104:       "length of zero_points must equal to channel expected ", channel, " got, ", zero_points.numel());
105: }
106:
107: } // anonymous namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 109-119
```cpp
109: Tensor& quantize_tensor_per_tensor_affine(
110:     const Tensor& rtensor,
111:     Tensor& qtensor,
112:     double scale,
113:     int64_t zero_point) {
114:   static constexpr auto fn_name = "quantize_tensor_per_tensor_affine";
115:
116:   checkRoundingMode(fn_name);
117:   checkFloatTensor(fn_name, rtensor);
118:   checkSameDevice(fn_name, rtensor, qtensor);
119:   checkSameSize(fn_name, qtensor, rtensor);
```
- EN: The main symbol in this range is `quantize_tensor_per_tensor_affine`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `quantize_tensor_per_tensor_affine`，它们直接构成本文件的算子逻辑。

### Lines 121-136
```cpp
121:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(qtensor.scalar_type(), fn_name, [&]() {
122:     checkQuantizedTensor<scalar_t>(fn_name, qtensor);
123:     checkZeroPoint<underlying_t>(fn_name, zero_point);
124:   });
125:
126:   // Temporary solution to pack the tensor if dtype is torch.quint4x2
127:   // Can move this into the fbgemm::Quantize op.
128:   if (qtensor.scalar_type() == at::ScalarType::QUInt4x2 || qtensor.scalar_type() == at::ScalarType::QUInt2x4) {
129:     quantize_tensor_per_tensor_affine_sub_byte_stub(
130:         rtensor.device().type(), rtensor, qtensor, scale, zero_point);
131:   } else {
132:     quantize_tensor_per_tensor_affine_stub(
133:         rtensor.device().type(), rtensor, qtensor, scale, zero_point);
134:   }
135:   return qtensor;
136: }
```
- EN: The main symbol in this range is `quantize_tensor_per_tensor_affine_sub_byte_stub`, `quantize_tensor_per_tensor_affine_stub`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `quantize_tensor_per_tensor_affine_sub_byte_stub`, `quantize_tensor_per_tensor_affine_stub`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 138-149
```cpp
138: Tensor& quantize_tensor_per_channel_affine(
139:     const Tensor& rtensor,
140:     Tensor& qtensor,
141:     const Tensor& scales,
142:     Tensor zero_points,
143:     int64_t axis) {
144:   static constexpr auto fn_name = "quantize_tensor_per_channel_affine";
145:
146:   checkRoundingMode(fn_name);
147:   checkFloatTensor(fn_name, rtensor);
148:   checkSameDevice(fn_name, rtensor, qtensor);
149:   checkSameSize(fn_name, qtensor, rtensor);
```
- EN: The main symbol in this range is `quantize_tensor_per_channel_affine`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `quantize_tensor_per_channel_affine`，它们直接构成本文件的算子逻辑。

### Lines 151-161
```cpp
151:   AT_DISPATCH_QINT_TYPES(qtensor.scalar_type(), fn_name, [&]() {
152:     checkQuantizedTensor<scalar_t>(fn_name, qtensor);
153:     if (qtensor.device().type() != c10::DeviceType::CUDA &&
154:         qtensor.device().type() != c10::DeviceType::XPU &&
155:         qtensor.device().type() != c10::DeviceType::PrivateUse1) {
156:       checkZeroPoints<underlying_t>(fn_name, zero_points);
157:     }  // for cuda and privateuse1, this check will occur in the actual device function
158:   });
159:
160:   TORCH_CHECK(
161:       0 <= axis && axis < rtensor.dim(),
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 162-172
```cpp
162:       "Channel axis out of range in per channel affine quantization. Got: ",
163:       axis,
164:       "Expected: [0, ",
165:       rtensor.dim(),
166:       ")");
167:   checkPerChannelParamsSize(rtensor, axis, scales, zero_points);
168:
169:   quantize_tensor_per_channel_affine_stub(
170:       rtensor.device().type(), rtensor, qtensor, scales, zero_points, axis);
171:   return qtensor;
172: }
```
- EN: The main symbol in this range is `quantize_tensor_per_channel_affine_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `quantize_tensor_per_channel_affine_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 174-186
```cpp
174: Tensor& quantize_tensor_per_channel_float_qparams(
175:     const Tensor& rtensor,
176:     Tensor& qtensor,
177:     const Tensor& scales,
178:     const Tensor& zero_points,
179:     int64_t axis) {
180:   static constexpr auto fn_name =
181:       "quantize_tensor_per_channel_float_qparams";
182:
183:   checkRoundingMode(fn_name);
184:   checkFloatTensor(fn_name, rtensor);
185:   checkSameDevice(fn_name, rtensor, qtensor);
186:   checkSameSize(fn_name, qtensor, rtensor);
```
- EN: The main symbol in this range is `quantize_tensor_per_channel_float_qparams`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `quantize_tensor_per_channel_float_qparams`，它们直接构成本文件的算子逻辑。

### Lines 188-199
```cpp
188:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(qtensor.scalar_type(), fn_name, [&]() {
189:     checkQuantizedTensor<scalar_t>(fn_name, qtensor);
190:   });
191:
192:   TORCH_CHECK(
193:       0 <= axis && axis < rtensor.dim(),
194:       "Channel axis out of range in per channel float qparams quantization. Got: ",
195:       axis,
196:       "Expected: [0, ",
197:       rtensor.dim(),
198:       ")");
199:   checkPerChannelParamsSize(rtensor, axis, scales, zero_points);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 201-214
```cpp
201:   quantize_tensor_per_channel_float_qparams_stub(
202:       rtensor.device().type(), rtensor, qtensor, scales, zero_points, axis);
203:   return qtensor;
204: }
205:
206: Tensor& dequantize_tensor_per_tensor_affine(
207:     const Tensor& qtensor,
208:     Tensor& rtensor,
209:     double scale,
210:     int64_t zero_point) {
211:   static constexpr auto fn_name = "dequantize_tensor_per_tensor_affine";
212:   checkFloatTensor(fn_name, rtensor);
213:   checkSameDevice(fn_name, rtensor, qtensor);
214:   checkSameSize(fn_name, qtensor, rtensor);
```
- EN: The main symbol in this range is `dequantize_tensor_per_tensor_affine`, `quantize_tensor_per_channel_float_qparams_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `dequantize_tensor_per_tensor_affine`, `quantize_tensor_per_channel_float_qparams_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 216-229
```cpp
216:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(qtensor.scalar_type(), fn_name, [&]() {
217:     checkQuantizedTensor<scalar_t>(fn_name, qtensor);
218:     checkZeroPoint<underlying_t>(fn_name, zero_point);
219:   });
220:
221:   if (qtensor.scalar_type() == at::ScalarType::QUInt4x2 || qtensor.scalar_type() == at::ScalarType::QUInt2x4) {
222:     dequantize_tensor_per_tensor_affine_sub_byte_stub(
223:         qtensor.device().type(), qtensor, rtensor, scale, zero_point);
224:   } else {
225:     dequantize_tensor_per_tensor_affine_stub(
226:         qtensor.device().type(), qtensor, rtensor, scale, zero_point);
227:   }
228:   return rtensor;
229: }
```
- EN: The main symbol in this range is `dequantize_tensor_per_tensor_affine_sub_byte_stub`, `dequantize_tensor_per_tensor_affine_stub`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `dequantize_tensor_per_tensor_affine_sub_byte_stub`, `dequantize_tensor_per_tensor_affine_stub`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 231-241
```cpp
231: Tensor& dequantize_tensor_per_channel_affine(
232:     const Tensor& qtensor,
233:     Tensor& rtensor,
234:     const Tensor& scales,
235:     Tensor zero_points,
236:     int64_t axis) {
237:   static constexpr auto fn_name = "dequantize_tensor_per_channel_affine";
238:
239:   checkFloatTensor(fn_name, rtensor);
240:   checkSameDevice(fn_name, rtensor, qtensor);
241:   checkSameSize(fn_name, qtensor, rtensor);
```
- EN: The main symbol in this range is `dequantize_tensor_per_channel_affine`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `dequantize_tensor_per_channel_affine`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 243-253
```cpp
243:   AT_DISPATCH_QINT_TYPES(qtensor.scalar_type(), fn_name, [&]() {
244:     checkQuantizedTensor<scalar_t>(fn_name, qtensor);
245:     if(qtensor.device().type() != c10::DeviceType::CUDA &&
246:        qtensor.device().type() != c10::DeviceType::XPU &&
247:        qtensor.device().type() != c10::DeviceType::PrivateUse1){
248:       checkZeroPoints<underlying_t>(fn_name, zero_points);
249:     }  // for cuda and privateuse1, this check will occur in the actual device function
250:   });
251:
252:   TORCH_CHECK(
253:       0 <= axis && axis < qtensor.dim(),
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 254-264
```cpp
254:       "Channel axis out of range in per channel affine dequantization. Got:",
255:       axis,
256:       " Expected: [0, ",
257:       qtensor.dim(),
258:       ")");
259:   checkPerChannelParamsSize(rtensor, axis, scales, zero_points);
260:
261:   dequantize_tensor_per_channel_affine_stub(
262:       qtensor.device().type(), qtensor, rtensor, scales, zero_points, axis);
263:   return rtensor;
264: }
```
- EN: The main symbol in this range is `dequantize_tensor_per_channel_affine_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `dequantize_tensor_per_channel_affine_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 266-276
```cpp
266: Tensor& dequantize_tensor_per_channel_float_qparams(
267:     const Tensor& qtensor,
268:     Tensor& rtensor,
269:     const Tensor& scales,
270:     const Tensor& zero_points,
271:     int64_t axis) {
272:   static constexpr auto fn_name = "dequantize_tensor_per_channel_float_qparams";
273:
274:   checkFloatTensor(fn_name, rtensor);
275:   checkSameDevice(fn_name, rtensor, qtensor);
276:   checkSameSize(fn_name, qtensor, rtensor);
```
- EN: The main symbol in this range is `dequantize_tensor_per_channel_float_qparams`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `dequantize_tensor_per_channel_float_qparams`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 278-289
```cpp
278:   AT_DISPATCH_QINT_AND_SUB_BYTE_TYPES(qtensor.scalar_type(), fn_name, [&]() {
279:     checkQuantizedTensor<scalar_t>(fn_name, qtensor);
280:   });
281:
282:   TORCH_CHECK(
283:       0 <= axis && axis < qtensor.dim(),
284:       "Channel axis out of range in per channel float qparams dequantization. Got:",
285:       axis,
286:       " Expected: [0, ",
287:       qtensor.dim(),
288:       ")");
289:   checkPerChannelParamsSize(rtensor, axis, scales, zero_points);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 291-296
```cpp
291:   dequantize_tensor_per_channel_float_qparams_stub(
292:       qtensor.device().type(), qtensor, rtensor, scales, zero_points, axis);
293:   return rtensor;
294: }
295:
296: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `dequantize_tensor_per_channel_float_qparams_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `dequantize_tensor_per_channel_float_qparams_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/quantized/AffineQuantizer.h`
- Key helper symbols / 关键辅助符号: `Scalar`, `ScalarType`, `fbgemm`
