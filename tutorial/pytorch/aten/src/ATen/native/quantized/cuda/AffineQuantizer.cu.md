# AffineQuantizer.cu — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cuda/AffineQuantizer.cu`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/TensorIterator.h>
 4: #include <ATen/native/quantized/AffineQuantizer.h>
 5: #include <cmath>
 6: #include <ATen/native/cuda/Loops.cuh>
 7:
 8: #ifndef AT_PER_OPERATOR_HEADERS
 9: #include <ATen/Functions.h>
10: #include <ATen/NativeFunctions.h>
11: #else
12: #include <ATen/ops/_unsafe_view_native.h>
13: #include <ATen/ops/any.h>
14: #include <ATen/ops/gt.h>
15: #include <ATen/ops/lt.h>
16: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/TensorIterator.h`, `ATen/native/quantized/AffineQuantizer.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/TensorIterator.h`, `ATen/native/quantized/AffineQuantizer.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 18-28
```
18: namespace at::native {
19: namespace {
20:
21: template <typename T>
22: void check_zero_points_cuda(
23:     const std::string& fn_name,
24:     const Tensor& zero_points) {
25:   constexpr int64_t qmin = std::numeric_limits<T>::min();
26:   constexpr int64_t qmax = std::numeric_limits<T>::max();
27:   auto zp_within_upper = at::any(at::gt(zero_points, qmax)).item().equal(false);
28:   auto zp_within_lower = at::any(at::lt(zero_points, qmin)).item().equal(false);
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `check_zero_points_cuda`, which contributes directly to this file's operator logic.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `check_zero_points_cuda`，它们直接构成本文件的算子逻辑。

### Lines 29-40
```
29:   TORCH_CHECK(
30:     zp_within_lower,
31:     fn_name,
32:     "zero_point is below lower bound.");
33:   TORCH_CHECK(
34:     zp_within_upper,
35:     fn_name,
36:     "zero_point is above upper bound.");
37: }
38:
39: void quantize_tensor_per_tensor_affine_cuda(
40:     const Tensor& rtensor,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 41-52
```
41:     Tensor& qtensor,
42:     double scale,
43:     int64_t zero_point) {
44:   AT_DISPATCH_QINT_TYPES(
45:       qtensor.scalar_type(), "quantize_tensor_per_tensor_affine_cuda", [&]() {
46:         constexpr int64_t qmin = std::numeric_limits<underlying_t>::min();
47:         constexpr int64_t qmax = std::numeric_limits<underlying_t>::max();
48:
49:         auto iter = TensorIteratorConfig()
50:                         .check_all_same_dtype(false)
51:                         .add_output(qtensor)
52:                         .add_input(rtensor)
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 53-66
```
53:                         .add_input(qtensor)
54:                         .build();
55:         gpu_kernel(
56:             iter,
57:             [=] GPU_LAMBDA(float raw_val, scalar_t quantized_val) -> scalar_t {
58:               int64_t qvalue =
59:                   static_cast<int64_t>(std::nearbyint(raw_val / scale) + zero_point);
60:               qvalue = std::max<int64_t>(qvalue, qmin);
61:               qvalue = std::min<int64_t>(qvalue, qmax);
62:               quantized_val.val_ = qvalue;
63:               return quantized_val;
64:             });
65:       });
66: }
```
- EN: The main symbol in this range is `gpu_kernel`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `gpu_kernel`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 68-78
```
68: void dequantize_tensor_per_tensor_affine_cuda(
69:     const Tensor& qtensor,
70:     Tensor& rtensor,
71:     double scale,
72:     int64_t zero_point) {
73:   AT_DISPATCH_QINT_TYPES(
74:       qtensor.scalar_type(), "dequantize_tensor_per_tensor_affine_cuda", [&]() {
75:         auto iter = TensorIteratorConfig()
76:                         .check_all_same_dtype(false)
77:                         .add_output(rtensor)
78:                         .add_input(qtensor)
```
- EN: The main symbol in this range is `dequantize_tensor_per_tensor_affine_cuda`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `dequantize_tensor_per_tensor_affine_cuda`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 79-94
```
79:                         .build();
80:         gpu_kernel(iter, [=] GPU_LAMBDA(scalar_t value) -> float {
81:           return (static_cast<float>(value.val_) - zero_point) * scale;
82:         });
83:       });
84: }
85:
86: void quantize_tensor_per_channel_affine_cuda(
87:     const Tensor& rtensor,
88:     Tensor& qtensor,
89:     const Tensor& scales,
90:     const Tensor& zero_points,
91:     int64_t axis) {
92:   static constexpr auto fn_name = "quantize_tensor_per_channel_affine_cuda";
93:   std::vector<int64_t> expected_shape(rtensor.dim(), 1);
94:   expected_shape[axis] = rtensor.size(axis);
```
- EN: The main symbol in this range is `quantize_tensor_per_channel_affine_cuda`, `gpu_kernel`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `quantize_tensor_per_channel_affine_cuda`, `gpu_kernel`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 96-106
```
 96:   auto shaped_scales = native::_unsafe_view(scales, expected_shape);
 97:   auto shaped_zero_points = native::_unsafe_view(zero_points, expected_shape);
 98:
 99:   auto iter = TensorIteratorConfig()
100:                   .check_all_same_dtype(false)
101:                   .add_output(qtensor)
102:                   .add_input(rtensor)
103:                   .add_input(qtensor)
104:                   .add_input(shaped_scales)
105:                   .add_input(shaped_zero_points)
106:                   .build();
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 108-117
```
108:   AT_DISPATCH_QINT_TYPES(
109:     qtensor.scalar_type(), fn_name, [&]() {
110:       check_zero_points_cuda<underlying_t>(fn_name, zero_points);
111:
112:       constexpr int64_t qmin = std::numeric_limits<underlying_t>::min();
113:       constexpr int64_t qmax = std::numeric_limits<underlying_t>::max();
114:       // trying to match _quantize_per_channel_ref_nd in test_quantized_tensor.py
115:       gpu_kernel(
116:           iter,
117:           [=] GPU_LAMBDA(float raw_val, scalar_t quantized_val, double scale, int64_t zero_point) -> scalar_t {
```
- EN: The main symbol in this range is `gpu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `gpu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 119-129
```
119:             int64_t qvalue =
120:                 static_cast<int64_t>(std::nearbyint(raw_val/scale) + zero_point);
121:             qvalue = std::max<int64_t>(qvalue, qmin);
122:             qvalue = std::min<int64_t>(qvalue, qmax);
123:             quantized_val.val_ = qvalue;
124:             return quantized_val;
125:           });
126:     });
127: }
128:
129: void dequantize_tensor_per_channel_affine_cuda(
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 130-146
```
130:     const Tensor& qtensor,
131:     Tensor& rtensor,
132:     const Tensor& scales,
133:     const Tensor& zero_points,
134:     int64_t axis) {
135:   static constexpr auto fn_name = "dequantize_tensor_per_channel_affine_cuda";
136:   std::vector<int64_t> expected_shape(rtensor.dim(), 1);
137:   expected_shape[axis] = rtensor.size(axis);
138:
139:   auto shaped_scales = native::_unsafe_view(scales, expected_shape);
140:   auto shaped_zero_points = native::_unsafe_view(zero_points, expected_shape);
141:
142:   AT_DISPATCH_QINT_TYPES(
143:       qtensor.scalar_type(),
144:       fn_name,
145:       [&]() {
146:         check_zero_points_cuda<underlying_t>(fn_name, zero_points);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 148-163
```
148:         auto iter = TensorIteratorConfig()
149:                         .check_all_same_dtype(false)
150:                         .add_output(rtensor)
151:                         .add_input(qtensor)
152:                         .add_input(shaped_scales)
153:                         .add_input(shaped_zero_points)
154:                         .build();
155:
156:         gpu_kernel(
157:             iter,
158:             [=] GPU_LAMBDA(
159:                 scalar_t value, double scale, int64_t zero_point) -> float {
160:               return static_cast<float>(value.val_ - zero_point) * scale;
161:             });
162:       });
163: }
```
- EN: The main symbol in this range is `gpu_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `gpu_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 165-176
```
165: void quantize_tensor_per_channel_float_qparams_cuda(
166:     const Tensor& rtensor,
167:     Tensor& qtensor,
168:     const Tensor& scales,
169:     const Tensor& zero_points,
170:     int64_t axis) {
171:   static constexpr auto fn_name = "quantize_tensor_per_channel_float_qparams_cuda";
172:   std::vector<int64_t> expected_shape(rtensor.dim(), 1);
173:   expected_shape[axis] = rtensor.size(axis);
174:
175:   auto shaped_scales = native::_unsafe_view(scales, expected_shape);
176:   auto shaped_zero_points = native::_unsafe_view(zero_points, expected_shape);
```
- EN: The main symbol in this range is `quantize_tensor_per_channel_float_qparams_cuda`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `quantize_tensor_per_channel_float_qparams_cuda`，它们直接构成本文件的算子逻辑。

### Lines 178-191
```
178:   auto iter = TensorIteratorConfig()
179:                   .check_all_same_dtype(false)
180:                   .add_output(qtensor)
181:                   .add_input(rtensor)
182:                   .add_input(qtensor)
183:                   .add_input(shaped_scales)
184:                   .add_input(shaped_zero_points)
185:                   .build();
186:
187:   AT_DISPATCH_QINT_TYPES(
188:       qtensor.scalar_type(),
189:       fn_name,
190:       [&]() {
191:         check_zero_points_cuda<underlying_t>(fn_name, zero_points);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 193-203
```
193:         constexpr int64_t qmin = std::numeric_limits<underlying_t>::min();
194:         constexpr int64_t qmax = std::numeric_limits<underlying_t>::max();
195:         // trying to match _quantize_per_channel_ref_nd in
196:         gpu_kernel(
197:             iter,
198:             [=] GPU_LAMBDA(
199:                 float raw_val,
200:                 scalar_t quantized_val,
201:                 float scale,
202:                 float zero_point) -> scalar_t {
203:               float inv_scale = 1.0f / scale;
```
- EN: The main symbol in this range is `gpu_kernel`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `gpu_kernel`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 204-215
```
204:               int64_t qvalue = lrintf(raw_val * inv_scale + zero_point);
205:               qvalue = std::max<int64_t>(qvalue, qmin);
206:               qvalue = std::min<int64_t>(qvalue, qmax);
207:               quantized_val.val_ = qvalue;
208:               return quantized_val;
209:             });
210:       });
211: }
212:
213: void dequantize_tensor_per_channel_float_qparams_cuda(
214:     const Tensor& qtensor,
215:     Tensor& rtensor,
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 216-230
```
216:     const Tensor& scales,
217:     const Tensor& zero_points,
218:     int64_t axis) {
219:   static constexpr auto fn_name = "dequantize_tensor_per_channel_float_qparams_cuda";
220:   std::vector<int64_t> expected_shape(rtensor.dim(), 1);
221:   expected_shape[axis] = rtensor.size(axis);
222:
223:   auto shaped_scales = native::_unsafe_view(scales, expected_shape);
224:   auto shaped_zero_points = native::_unsafe_view(zero_points, expected_shape);
225:
226:   AT_DISPATCH_QINT_TYPES(
227:       qtensor.scalar_type(),
228:       fn_name,
229:       [&]() {
230:         check_zero_points_cuda<underlying_t>(fn_name, zero_points);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 232-247
```
232:         auto iter = TensorIteratorConfig()
233:                         .check_all_same_dtype(false)
234:                         .add_output(rtensor)
235:                         .add_input(qtensor)
236:                         .add_input(shaped_scales)
237:                         .add_input(shaped_zero_points)
238:                         .build();
239:
240:         gpu_kernel(
241:             iter,
242:             [=] GPU_LAMBDA(
243:                 scalar_t value, float scale, float zero_point) -> float {
244:               return (static_cast<float>(value.val_) - zero_point) * scale;
245:             });
246:       });
247: }
```
- EN: The main symbol in this range is `gpu_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `gpu_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 249-259
```
249: } // anonymous namespace
250:
251: REGISTER_DISPATCH(
252:     quantize_tensor_per_tensor_affine_stub,
253:     &quantize_tensor_per_tensor_affine_cuda);
254: REGISTER_DISPATCH(
255:     dequantize_tensor_per_tensor_affine_stub,
256:     &dequantize_tensor_per_tensor_affine_cuda);
257: REGISTER_DISPATCH(
258:     quantize_tensor_per_channel_affine_stub,
259:     &quantize_tensor_per_channel_affine_cuda);
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 260-269
```
260: REGISTER_DISPATCH(
261:     dequantize_tensor_per_channel_affine_stub,
262:     &dequantize_tensor_per_channel_affine_cuda);
263: REGISTER_DISPATCH(
264:     quantize_tensor_per_channel_float_qparams_stub,
265:     &quantize_tensor_per_channel_float_qparams_cuda);
266: REGISTER_DISPATCH(
267:     dequantize_tensor_per_channel_float_qparams_stub,
268:     &dequantize_tensor_per_channel_float_qparams_cuda);
269: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/TensorIterator.h`, `ATen/native/quantized/AffineQuantizer.h`, `ATen/native/cuda/Loops.cuh`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_unsafe_view_native.h`, `ATen/ops/any.h`, `ATen/ops/gt.h`, `ATen/ops/lt.h`
- Standard or third-party headers / 标准库或第三方头文件: `cmath`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `REGISTER_DISPATCH`
