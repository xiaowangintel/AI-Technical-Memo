# FakeQuantPerChannelAffine.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/FakeQuantPerChannelAffine.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #include <ATen/ATen.h>
 2: #include <ATen/Dispatch.h>
 3: #include <ATen/NativeFunctions.h>
 4: #include <ATen/native/TensorIterator.h>
 5: #include <ATen/native/cpu/Loops.h>
 6: #include <ATen/native/quantized/FakeQuantAffine.h>
 7:
 8: #include <c10/util/irange.h>
 9:
10: // FakeQuantize Op for PerChannelAffine quantization scheme.
11:
12: namespace at::native {
```
- EN: This range pulls in required headers, including `ATen/ATen.h`, `ATen/Dispatch.h`, `ATen/NativeFunctions.h`. The code enters or documents the namespace scope used by ATen native CPU operators. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/ATen.h`, `ATen/Dispatch.h`, `ATen/NativeFunctions.h`。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 14-28
```cpp
14: // Use REGISTER_DISPATCH to run CPU and CUDA backend.
15: DEFINE_DISPATCH(fake_quant_per_channel_cachemask_stub);
16: DEFINE_DISPATCH(fake_quant_grad_learnable_channel_stub);
17:
18: /* Per channel fake-quantizes the 'inputs' tensor.
19: Args:
20:   X: Forward input tensor.
21:   dY: Backward input tensor (_backward op only).
22:   scale: scale of per channel affine quantization
23:   zero_point: zero_point of per channel affine quantization
24:   axis: int specifying the axis to be quantized
25:   quant_min: minimum quantized value
26:   quant_max: maximum quantized value
27: Returns:
28:   Fake quantized tensor (double dtype).
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 30-42
```cpp
30: */
31:
32: Tensor fake_quantize_per_channel_affine(
33:     const Tensor& self,
34:     const Tensor& scale,
35:     const Tensor& zero_point,
36:     int64_t axis,
37:     int64_t quant_min,
38:     int64_t quant_max) {
39:   auto res = at::fake_quantize_per_channel_affine_cachemask(
40:       self, scale, zero_point, axis, quant_min, quant_max);
41:   return std::get<0>(std::move(res));
42: }
```
- EN: The main symbol in this range is `fake_quantize_per_channel_affine`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fake_quantize_per_channel_affine`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 44-54
```cpp
44: std::tuple<Tensor, Tensor> fake_quantize_per_channel_affine_cachemask(
45:     const Tensor& self,
46:     const Tensor& scale,
47:     const Tensor& zero_point,
48:     int64_t axis,
49:     int64_t quant_min,
50:     int64_t quant_max) {
51:   TORCH_CHECK(scale.scalar_type() == ScalarType::Float || scale.scalar_type() == at::kBFloat16,
52:               "Scale must be Float or BFloat16, found ", scale.scalar_type());
53:   TORCH_CHECK(zero_point.scalar_type() == ScalarType::Int || zero_point.scalar_type() == ScalarType::Float || zero_point.scalar_type() == ScalarType::Half,
54:               "Zero-point must be Int32, Float or Half, found ", zero_point.scalar_type());
```
- EN: The main symbol in this range is `fake_quantize_per_channel_affine_cachemask`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `fake_quantize_per_channel_affine_cachemask`，它们直接构成本文件的算子逻辑。

### Lines 55-67
```cpp
55:   TORCH_CHECK(scale.dim() == 1, "scale should be a 1-D tensor");
56:   TORCH_CHECK(zero_point.dim() == 1, "zero point should be a 1-D tensor");
57:   TORCH_CHECK(
58:       scale.numel() == zero_point.numel(),
59:       "scale and zero-point need to have the same dimensions");
60:   TORCH_CHECK(
61:       scale.numel() == self.size(axis),
62:       "dimensions of scale and zero-point are not consistent with input tensor")
63:
64:   TORCH_CHECK(
65:       quant_min <= quant_max,
66:       "`quant_min` should be less than or \
67:         equal to `quant_max`.");
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 69-80
```cpp
69:   if(!at::isFloatingType(zero_point.scalar_type())){
70:       TORCH_CHECK(
71:           at::min(zero_point).item().toInt() >= quant_min &&
72:               at::max(zero_point).item().toInt() <= quant_max,
73:           "`zero_point` must be between `quant_min` and `quant_max`.");
74:   }
75:   TORCH_CHECK(
76:       axis >= 0 && axis <= self.dim(),
77:       "`axis` must be between 0 and number of dimensions of input");
78:
79:   auto Y = at::empty_like(self, self.options(), MemoryFormat::Preserve);
80:   auto mask = at::empty_like(self, at::kBool, MemoryFormat::Preserve);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 82-91
```cpp
82:   c10::DimVector expected_shape(self.dim(), 1);
83:   expected_shape[axis] = self.size(axis);
84:
85:   TensorIterator iter = TensorIteratorConfig()
86:     .check_all_same_dtype(false)
87:     .add_output(Y)
88:     .add_input(self)
89:     .add_owned_input(native::_unsafe_view(scale, expected_shape))
90:     .add_owned_input(native::_unsafe_view(zero_point, expected_shape))
91:     .build();
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 93-107
```cpp
 93:   // TODO(future, optional): read once, write twice.  Not done at the moment
 94:   //   for simplicity, as we do not expect this to be a bottleneck.
 95:   TensorIterator iter_mask = TensorIteratorConfig()
 96:     .check_all_same_dtype(false)
 97:     .add_output(mask)
 98:     .add_input(self)
 99:     .add_owned_input(native::_unsafe_view(scale, expected_shape))
100:     .add_owned_input(native::_unsafe_view(zero_point, expected_shape))
101:     .build();
102:
103:   // TODO(future, optional): look into packing the mask further (BoolTensor uses
104:   //   1 byte per element, we only need 1 bit per element).
105:   fake_quant_per_channel_cachemask_stub(iter.device_type(), iter, iter_mask, quant_min, quant_max);
106:   return std::make_tuple(Y, mask);
107: }
```
- EN: The main symbol in this range is `fake_quant_per_channel_cachemask_stub`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fake_quant_per_channel_cachemask_stub`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 109-119
```cpp
109: /* Backward path to fake-quantize the 'inputs' tensor per channel, with mask.
110:
111: Args:
112:   dY: output grad.
113:   mask: mask tensor from the forward pass.
114:
115: Returns:
116:   dX (input grad).
117: */
118: Tensor fake_quantize_per_channel_affine_cachemask_backward(
119:     const Tensor& dY,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 120-131
```cpp
120:     const Tensor& mask) {
121:   TORCH_CHECK(mask.scalar_type() == ScalarType::Bool);
122:   TORCH_CHECK(mask.numel() == dY.numel(),
123:       "`mask` and `dY` are not the same size: ",
124:       "`mask` is size ", mask.numel(), " and `dY` is size ", dY.numel());
125:   if (dY.numel() <= 0) {
126:     return dY;
127:   }
128:   // Note: no additional kernels needed, since mask is pre-computed
129:   // and we can use the existing tensor multiplication kernels.
130:   return dY * mask;
131: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 133-143
```cpp
133: static Tensor _get_rounded_zero_point(
134:     const Tensor& zero_point,
135:     int64_t quant_min,
136:     int64_t quant_max) {
137:   // This assumes the per channel zero point vector is single-dimensioned.
138:   return zero_point.round().clamp_(quant_min, quant_max);
139: }
140:
141: Tensor _fake_quantize_learnable_per_channel_affine(
142:     const Tensor& self,
143:     const Tensor& scale,
```
- EN: The main symbol in this range is `_get_rounded_zero_point`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `_get_rounded_zero_point`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 144-155
```cpp
144:     const Tensor& zero_point,
145:     int64_t axis,
146:     int64_t quant_min,
147:     int64_t quant_max,
148:     double grad_factor) {
149:   Tensor zero_point_rounded = _get_rounded_zero_point(zero_point, quant_min, quant_max).to(at::kInt);
150:   return native::fake_quantize_per_channel_affine(
151:     self, scale, zero_point_rounded, axis, quant_min, quant_max);
152: }
153:
154: std::tuple<Tensor, Tensor, Tensor> _fake_quantize_learnable_per_channel_affine_backward(
155:     const Tensor& dY,
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 156-167
```cpp
156:     const Tensor& X,
157:     const Tensor& scale,
158:     const Tensor& zero_point,
159:     int64_t axis,
160:     int64_t quant_min,
161:     int64_t quant_max,
162:     double grad_factor) {
163:   /* The gradients for scale and zero point are calculated as below:
164:      Let Xfq be the fake quantized version of X.
165:      Let Xq be the quantized version of X (clamped at qmin and qmax).
166:      Let Delta and z be the scale and the zero point.
167:      :math:
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 168-179
```cpp
168:       \frac{d\Delta }{dx} =
169:         \begin{cases}
170:           q_{\min} - z& \text{ if } X_q= q_{\min} \\
171:           q_{\max} - z& \text{ if } X_q= q_{\max} \\
172:           (X_{fq} - X) / \Delta & \text{ else }
173:         \end{cases}
174:
175:       \frac{dz }{dx} =
176:         \begin{cases}
177:           -\Delta& \text{ if } X_q= q_{\min} \text{ or } X_q = q_{\max} \\
178:           0 & \text{ else }
179:         \end{cases}
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 180-192
```cpp
180:   */
181:   bool is_bfloat16 = (X.scalar_type() == at::kBFloat16);
182:   at::Tensor X_ = is_bfloat16 ? X.to(ScalarType::Float) : X;
183:   at::Tensor dY_ = is_bfloat16 ? dY.to(ScalarType::Float) : dY;
184:   at::Tensor scale_ = is_bfloat16 ? scale.to(ScalarType::Float) : scale;
185:   at::Tensor zero_point_ = is_bfloat16 ? zero_point.to(ScalarType::Float) : zero_point;
186:
187:   auto zero_point_rounded = _get_rounded_zero_point(zero_point_, quant_min, quant_max);
188:
189:   TORCH_CHECK(dY_.scalar_type() == ScalarType::Float);
190:   TORCH_CHECK(X_.scalar_type() == ScalarType::Float);
191:   TORCH_CHECK(scale_.scalar_type() == ScalarType::Float);
192:   TORCH_CHECK(zero_point_.scalar_type() == ScalarType::Float);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 194-205
```cpp
194:   TORCH_CHECK(X_.sizes() == dY_.sizes(), "`X` and `dY` are not the same size");
195:   TORCH_CHECK(
196:       quant_min <= 0 && quant_max >= 0,
197:       "Expecting `quant_min` <= 0 and `quant_max` >= 0");
198:   TORCH_CHECK(scale_.dim() == 1, "scale should be a 1-D tensor");
199:   TORCH_CHECK(zero_point_.dim() == 1, "zero point should be a 1-D tensor");
200:   TORCH_CHECK(
201:       scale_.numel() == zero_point_.numel(),
202:       "scale and zero-point need to have the same dimensions");
203:   TORCH_CHECK(
204:       scale_.numel() == X_.size(axis),
205:       "dimensions of scale and zero-point are not consistent with input tensor")
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 207-218
```cpp
207:   TORCH_CHECK(
208:       at::min(zero_point_rounded).item().toLong() >= quant_min &&
209:           at::max(zero_point_rounded).item().toLong() <= quant_max,
210:       "`zero_point` must be between `quant_min` and `quant_max`.");
211:
212:   TORCH_CHECK(
213:       axis >= 0 && axis < X_.dim(),
214:       "`axis` must be between 0 and number of dimensions of input");
215:
216:   if (X_.numel() <= 0) {
217:     return std::make_tuple(X, scale, zero_point);
218:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 220-233
```cpp
220:   auto dX = at::empty_like(X_, X_.options(), MemoryFormat::Preserve);
221:   auto dScale_vec = at::empty_like(X_, X_.options(), MemoryFormat::Preserve);
222:   auto dZeroPoint_vec = at::empty_like(X_, X_.options(), MemoryFormat::Preserve);
223:   auto numDimensions = X_.ndimension();
224:
225:   // Create an axis mask for vectorizing and reshaping the scale and zero point tensors
226:   // into the same shapes as X along the channel axis.
227:   c10::DimVector axis_mask(numDimensions);
228:   for (const auto i : c10::irange(numDimensions)) {
229:     axis_mask[i] = (i == axis) ? X_.size(axis) : 1;
230:   }
231:   auto X_shape = X_.sizes();
232:   auto scale_vectorized = scale_.reshape(at::IntArrayRef(axis_mask.data(), numDimensions)).expand(X_shape);
233:   auto zero_point_vectorized = zero_point_rounded.reshape(at::IntArrayRef(axis_mask.data(), numDimensions)).expand(X_shape);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 235-246
```cpp
235:   auto iter = TensorIteratorConfig()
236:     .add_output(dX)
237:     .add_output(dScale_vec)
238:     .add_output(dZeroPoint_vec)
239:     .add_input(X_)
240:     .add_input(dY_)
241:     .add_input(scale_vectorized)
242:     .add_input(zero_point_vectorized)
243:     .build();
244:
245:   fake_quant_grad_learnable_channel_stub(
246:     X_.device().type(), iter, quant_min, quant_max, grad_factor);
```
- EN: The main symbol in this range is `fake_quant_grad_learnable_channel_stub`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `fake_quant_grad_learnable_channel_stub`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 248-258
```cpp
248:   auto numElements = X_.ndimension() - 1;
249:
250:   // Create a collection of axes that include all but the channel axis for
251:   // reduction when summing over the dScale and dZeroPoint tensors.
252:   c10::DimVector axis_for_reduction(numElements);
253:   for (const auto i : c10::irange(axis)) {
254:     axis_for_reduction[i] = i;
255:   }
256:   for (const auto i : c10::irange(axis, numElements)) {
257:     axis_for_reduction[i] = i + 1;
258:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 260-265
```cpp
260:   auto dScale = dScale_vec.sum(at::IntArrayRef(axis_for_reduction.data(), numElements));
261:   auto dZeroPoint = dZeroPoint_vec.sum(at::IntArrayRef(axis_for_reduction.data(), numElements));
262:
263:   return std::make_tuple(std::move(dX), std::move(dScale), std::move(dZeroPoint));
264: }
265: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 该代码块还会组装或返回外围算子代码所需的结果。

## Key Concepts / 关键概念

- TensorIterator traversal / TensorIterator 遍历
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/ATen.h`, `ATen/Dispatch.h`, `ATen/NativeFunctions.h`, `ATen/native/TensorIterator.h`, `ATen/native/cpu/Loops.h`, `ATen/native/quantized/FakeQuantAffine.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `REGISTER_DISPATCH`, `Scalar`, `ScalarType`
