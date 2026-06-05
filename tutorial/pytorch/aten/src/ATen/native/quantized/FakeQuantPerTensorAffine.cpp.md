# FakeQuantPerTensorAffine.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/FakeQuantPerTensorAffine.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
 1: #include <ATen/ATen.h>
 2: #include <ATen/Dispatch.h>
 3: #include <ATen/NativeFunctions.h>
 4: #include <ATen/native/TensorIterator.h>
 5: #include <ATen/native/cpu/Loops.h>
 6: #include <ATen/native/quantized/FakeQuantAffine.h>
 7:
 8: // FakeQuantize Op for PerTensorAffine quantization scheme.
 9:
10: namespace at::native {
11:
12: // Use REGISTER_DISPATCH to run CPU and CUDA backend.
13: DEFINE_DISPATCH(fake_quant_tensor_cachemask_stub);
14: DEFINE_DISPATCH(fake_quant_grad_learnable_tensor_stub);
15: DEFINE_DISPATCH(fake_quant_tensor_cachemask_tensor_qparams_stub);
```
- EN: This range pulls in required headers, including `ATen/ATen.h`, `ATen/Dispatch.h`, `ATen/NativeFunctions.h`. The code enters or documents the namespace scope used by ATen native CPU operators. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/ATen.h`, `ATen/Dispatch.h`, `ATen/NativeFunctions.h`。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 17-28
```cpp
17: /* Fake-quantizes the 'inputs' tensor.
18:
19: Args:
20:   self: Forward input tensor.
21:   dY: Backward input tensor (_backward op only).
22:   scale: scale of per tensor affine quantization
23:   zero_point: zero_point of per tensor affine quantization
24:   quant_min: minimum quantized value
25:   quant_max: maximum quantized value
26:
27: Returns:
28:   Quantized tensor (double dtype).
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 30-40
```cpp
30: */
31: Tensor fake_quantize_per_tensor_affine(
32:     const Tensor& self,
33:     double scale,
34:     int64_t zero_point,
35:     int64_t quant_min,
36:     int64_t quant_max) {
37:   auto res = at::fake_quantize_per_tensor_affine_cachemask(
38:       self, scale, zero_point, quant_min, quant_max);
39:   return std::get<0>(std::move(res));
40: }
```
- EN: The main symbol in this range is `fake_quantize_per_tensor_affine`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fake_quantize_per_tensor_affine`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 42-53
```cpp
42: Tensor fake_quantize_per_tensor_affine(
43:     const Tensor& self,
44:     const Tensor& scale,
45:     const Tensor& zero_point,
46:     int64_t quant_min,
47:     int64_t quant_max) {
48:   auto res = at::_fake_quantize_per_tensor_affine_cachemask_tensor_qparams(
49:       self, scale, zero_point, at::ones(1, self.options().dtype(at::kLong)), quant_min, quant_max);
50:   return std::get<0>(std::move(res));
51: }
52:
53: /* Fake-quantizes the 'inputs' tensor, saving a mask for the backward pass.
```
- EN: The main symbol in this range is `fake_quantize_per_tensor_affine`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fake_quantize_per_tensor_affine`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 55-65
```cpp
55: This is numerically equivalent to `fake_quantize_per_tensor_affine`,
56: but has a lower memory overhead in the backward pass.
57:
58: Args:
59:   self: Forward input tensor.
60:   scale: scale of per tensor affine quantization
61:   zero_point: zero_point of per tensor affine quantization
62:   quant_min: minimum quantized value
63:   quant_max: maximum quantized value
64:
65: Returns:
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 66-81
```cpp
66:   Quantized tensor (double dtype).
67:   Mask (bool dtype).
68: */
69: std::tuple<Tensor, Tensor> fake_quantize_per_tensor_affine_cachemask(
70:     const Tensor& self,
71:     double scale,
72:     int64_t zero_point,
73:     int64_t quant_min,
74:     int64_t quant_max) {
75:   TORCH_CHECK(
76:       quant_min <= quant_max,
77:       "`quant_min` should be less than or \
78:         equal to `quant_max`.");
79:   TORCH_CHECK(
80:       zero_point >= quant_min && zero_point <= quant_max,
81:       "`zero_point` must be between `quant_min` and `quant_max`.");
```
- EN: The main symbol in this range is `tensor`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `tensor`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 83-93
```cpp
83:   auto Y = at::empty_like(self, self.options(), MemoryFormat::Preserve);
84:   auto mask = at::empty_like(self, at::kBool, MemoryFormat::Preserve);
85:   fake_quant_tensor_cachemask_stub(
86:       self.device().type(), Y, mask, self, scale, zero_point, quant_min, quant_max);
87:   // TODO(future, optional): look into packing the mask further (BoolTensor uses
88:   //   1 byte per element, we only need 1 bit per element).
89:   return std::make_tuple(std::move(Y), std::move(mask));
90: }
91:
92: std::tuple<Tensor, Tensor> _fake_quantize_per_tensor_affine_cachemask_tensor_qparams(
93:     const Tensor& self,
```
- EN: The main symbol in this range is `fake_quant_tensor_cachemask_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fake_quant_tensor_cachemask_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 94-110
```cpp
 94:     const Tensor& scale,
 95:     const Tensor& zero_point,
 96:     const Tensor& fake_quant_enabled,
 97:     int64_t quant_min,
 98:     int64_t quant_max) {
 99:   TORCH_CHECK(
100:       quant_min <= quant_max,
101:       "`quant_min` should be less than or \
102:         equal to `quant_max`.");
103:   auto Y = at::empty_like(self, self.options(), MemoryFormat::Preserve);
104:   auto mask = at::empty_like(self, at::kBool, MemoryFormat::Preserve);
105:   fake_quant_tensor_cachemask_tensor_qparams_stub(
106:       self.device().type(), Y, mask, self, scale, zero_point, fake_quant_enabled, quant_min, quant_max);
107:   // TODO(future, optional): look into packing the mask further (BoolTensor uses
108:   //   1 byte per element, we only need 1 bit per element).
109:   return std::make_tuple(std::move(Y), std::move(mask));
110: }
```
- EN: The main symbol in this range is `fake_quant_tensor_cachemask_tensor_qparams_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fake_quant_tensor_cachemask_tensor_qparams_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 112-122
```cpp
112: /* Backward path to fake-quantize the 'inputs' tensor, with mask.
113:
114: Args:
115:   dY: output grad.
116:   mask: mask tensor from the forward pass.
117:
118: Returns:
119:   dX (input grad).
120: */
121: Tensor fake_quantize_per_tensor_affine_cachemask_backward(
122:     const Tensor& dY,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 123-134
```cpp
123:     const Tensor& mask) {
124:   TORCH_CHECK(mask.scalar_type() == ScalarType::Bool);
125:   TORCH_CHECK(mask.sym_numel() == dY.sym_numel(),
126:       "`mask` and `dY` are not the same size: ",
127:       "`mask` is size ", mask.sym_numel(), " and `dY` is size ", dY.sym_numel());
128:   if (dY.sym_numel() <= 0) {
129:     return dY;
130:   }
131:   // Note: no additional kernels needed, since mask is pre-computed
132:   // and we can use the existing tensor multiplication kernels.
133:   return dY * mask;
134: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 136-146
```cpp
136: static int64_t _get_zero_point_from_tensor(
137:     const Tensor& zero_point,
138:     int64_t quant_min,
139:     int64_t quant_max,
140:     bool is_forward) {
141:   float zero_point_fp = zero_point[0].item<float>();
142:   zero_point_fp = is_forward ? std::nearbyint(zero_point_fp) : zero_point_fp + 0.5f;
143:   float zero_point_clamped = std::min(std::max(zero_point_fp, static_cast<float>(quant_min)),
144:                                        static_cast<float>(quant_max));
145:   return static_cast<int64_t>(zero_point_clamped);
146: }
```
- EN: The main symbol in this range is `_get_zero_point_from_tensor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `_get_zero_point_from_tensor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 148-159
```cpp
148: Tensor _fake_quantize_learnable_per_tensor_affine(
149:     const Tensor& self,
150:     const Tensor& scale,
151:     const Tensor& zero_point,
152:     int64_t quant_min,
153:     int64_t quant_max,
154:     double grad_factor) {
155:   float scale_val = scale[0].item<float>();
156:   int64_t zero_point_val = native::_get_zero_point_from_tensor(zero_point, quant_min, quant_max, true);
157:   return native::fake_quantize_per_tensor_affine(
158:     self, scale_val, zero_point_val, quant_min, quant_max);
159: }
```
- EN: The main symbol in this range is `_fake_quantize_learnable_per_tensor_affine`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `_fake_quantize_learnable_per_tensor_affine`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 161-171
```cpp
161: std::tuple<Tensor, Tensor, Tensor> _fake_quantize_learnable_per_tensor_affine_backward(
162:     const Tensor& dY,
163:     const Tensor& X,
164:     const Tensor& scale,
165:     const Tensor& zero_point,
166:     int64_t quant_min,
167:     int64_t quant_max,
168:     double grad_factor) {
169:   /* The gradients for scale and zero point are calculated as below:
170:      Let Xfq be the fake quantized version of X.
171:      Let Xq be the quantized version of X (clamped at qmin and qmax).
```
- EN: The main symbol in this range is `_fake_quantize_learnable_per_tensor_affine_backward`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_fake_quantize_learnable_per_tensor_affine_backward`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 172-186
```cpp
172:      Let Delta and z be the scale and the zero point.
173:      :math:
174:       \frac{d\Delta }{dx} =
175:         \begin{cases}
176:           q_{\min} - z& \text{ if } X_q= q_{\min} \\
177:           q_{\max} - z& \text{ if } X_q= q_{\max} \\
178:           (X_{fq} - X) / \Delta & \text{ else }
179:         \end{cases}
180:
181:       \frac{dz }{dx} =
182:         \begin{cases}
183:           -\Delta& \text{ if } X_q= q_{\min} \text{ or } X_q = q_{\max} \\
184:           0 & \text{ else }
185:         \end{cases}
186:   */
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 188-197
```cpp
188:   bool is_bfloat16 = (X.scalar_type() == at::kBFloat16);
189:
190:   at::Tensor X_ = is_bfloat16 ? X.to(ScalarType::Float) : X;
191:   at::Tensor dY_ = is_bfloat16 ? dY.to(ScalarType::Float) : dY;
192:   at::Tensor scale_ = is_bfloat16 ? scale.to(ScalarType::Float) : scale;
193:   at::Tensor zero_point_ = is_bfloat16 ? zero_point.to(ScalarType::Float) : zero_point;
194:
195:   float scale_val = scale_[0].item<float>();
196:   float inv_scale_val = 1.0f / scale_val;
197:   int64_t zero_point_val = native::_get_zero_point_from_tensor(zero_point_, quant_min, quant_max, false);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 199-213
```cpp
199:   TORCH_CHECK(dY_.scalar_type() == ScalarType::Float);
200:   TORCH_CHECK(X_.scalar_type() == ScalarType::Float);
201:   TORCH_CHECK(scale_.scalar_type() == ScalarType::Float);
202:   TORCH_CHECK(zero_point_.scalar_type() == ScalarType::Float);
203:   TORCH_CHECK(X_.numel() == dY_.numel(), "`X` and `dY` are not the same size");
204:   TORCH_CHECK(
205:       quant_min <= 0 && quant_max >= 0,
206:       "`quant_min` should be less than or \
207:         equal to `quant_max`, and the quantization range should include 0.");
208:   TORCH_CHECK(
209:       zero_point_val >= quant_min && zero_point_val <= quant_max,
210:       "`zero_point` must be between `quant_min` and `quant_max`.");
211:   if (X_.numel() <= 0) {
212:     return std::make_tuple(X, scale, zero_point);
213:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 215-225
```cpp
215:   auto dX = at::empty_like(X_, X_.options(), MemoryFormat::Preserve);
216:   auto dScale_vec = at::empty_like(X_, X_.options(), MemoryFormat::Preserve);
217:   auto dZeroPoint_vec = at::empty_like(X_, X_.options(), MemoryFormat::Preserve);
218:
219:   auto iter = TensorIteratorConfig()
220:     .add_output(dX)
221:     .add_output(dScale_vec)
222:     .add_output(dZeroPoint_vec)
223:     .add_input(X_)
224:     .add_input(dY_)
225:     .build();
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 227-237
```cpp
227:   fake_quant_grad_learnable_tensor_stub(
228:     X_.device().type(), iter, scale_val, inv_scale_val, zero_point_val, quant_min, quant_max, grad_factor);
229:
230:   // The total sums over the scale and zero point gradient vectors are what will be returned in the end.
231:   auto dScale = dScale_vec.sum().unsqueeze(0).to(scale_.device());
232:   auto dZeroPoint = dZeroPoint_vec.sum().unsqueeze(0).to(zero_point_.device());
233:
234:   return std::make_tuple(std::move(dX), std::move(dScale), std::move(dZeroPoint));
235: }
236:
237: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `fake_quant_grad_learnable_tensor_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `fake_quant_grad_learnable_tensor_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

## Key Concepts / 关键概念

- TensorIterator traversal / TensorIterator 遍历
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/ATen.h`, `ATen/Dispatch.h`, `ATen/NativeFunctions.h`, `ATen/native/TensorIterator.h`, `ATen/native/cpu/Loops.h`, `ATen/native/quantized/FakeQuantAffine.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `REGISTER_DISPATCH`, `Scalar`, `ScalarType`
