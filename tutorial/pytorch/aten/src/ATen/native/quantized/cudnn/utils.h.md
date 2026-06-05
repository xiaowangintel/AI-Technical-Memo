# utils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cudnn/utils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: #pragma once
 2: /*
 3: This file contains some of the auxiliary functions used by both Conv.cpp & Linear.cpp (introduced in a later PR)
 4: */
 5:
 6: #ifdef USE_CUDA
 7: #include <ATen/cuda/CUDAConfig.h>  // for the definition of AT_CUDNN_ENABLED
 8:
 9: #if AT_CUDNN_ENABLED()
10:
11: #include <ATen/cudnn/Types.h>
12: #include <ATen/Tensor.h>
13: #include <ATen/native/quantized/PackedParams.h>
14: #include <c10/core/QScheme.h>
15: #include <c10/util/ArrayRef.h>
16: #include <c10/util/Exception.h>
```
- EN: This range pulls in required headers, including `ATen/cuda/CUDAConfig.h`, `ATen/cudnn/Types.h`, `ATen/Tensor.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/cuda/CUDAConfig.h`, `ATen/cudnn/Types.h`, `ATen/Tensor.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 18-28
```cpp
18: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wsuggest-override")
19: #include <cudnn_frontend.h>
20: C10_DIAGNOSTIC_POP()
21:
22: #ifndef AT_PER_OPERATOR_HEADERS
23: #include <ATen/Functions.h>
24: #else
25: #include <ATen/ops/empty.h>
26: #endif
27:
28: struct PackedLinearWeightCudnn : public LinearPackedParamsBase {
```
- EN: This range pulls in required headers, including `cudnn_frontend.h`, `ATen/Functions.h`, `ATen/ops/empty.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `PackedLinearWeightCudnn`, which contributes directly to this file's operator logic.
- CN: 这一段引入了所需头文件，例如 `cudnn_frontend.h`, `ATen/Functions.h`, `ATen/ops/empty.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `PackedLinearWeightCudnn`，它们直接构成本文件的算子逻辑。

### Lines 29-44
```cpp
29:   PackedLinearWeightCudnn(
30:       at::Tensor orig_weight,
31:       std::optional<at::Tensor> bias,
32:       c10::QScheme q_scheme)
33:       : orig_weight(std::move(orig_weight)),
34:         bias_(std::move(bias)),
35:         q_scheme(std::move(q_scheme)) {}
36:
37:   at::Tensor apply(
38:       at::Tensor input,
39:       double output_scale,
40:       int64_t output_zero_point) override;
41:   at::Tensor apply_relu(
42:       at::Tensor input,
43:       double output_scale,
44:       int64_t output_zero_point) override;
```
- EN: The main symbol in this range is `PackedLinearWeightCudnn`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `PackedLinearWeightCudnn`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 46-57
```cpp
46:   at::Tensor apply_dynamic(at::Tensor input, bool reduce_range = false) override {
47:     TORCH_CHECK(false, "apply_dynamic is not implemented for this packed parameter type");
48:   }
49:   at::Tensor apply_dynamic_relu(at::Tensor input, bool reduce_range = false) override {
50:     TORCH_CHECK(false, "apply_dynamic_relu is not implemented for this packed parameter type");
51:   }
52:
53:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
54:
55:   std::optional<at::Tensor> bias() override {
56:     return bias_;
57:   }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 59-72
```cpp
59:   static c10::intrusive_ptr<LinearPackedParamsBase> prepack(
60:       at::Tensor weight,
61:       std::optional<at::Tensor> bias);
62:
63:  private:
64:   at::Tensor orig_weight;
65:   std::optional<at::Tensor> bias_;
66:   c10::QScheme q_scheme;
67:
68:   template <bool ReluFused>
69:   at::Tensor apply_impl(
70:       const at::Tensor& input,
71:       double output_scale,
72:       int64_t output_zero_point);
```
- EN: The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 74-84
```cpp
74:   template <bool ReluFused>
75:   void apply_impl_helper(
76:       const at::Tensor& quantized_output,
77:       const at::Tensor& input,
78:       double output_scale);
79: };
80:
81: template <int kSpatialDim = 2>
82: struct PackedConvWeightCudnn : public ConvPackedParamsBase<kSpatialDim> {
83:   PackedConvWeightCudnn(
84:       at::Tensor orig_weight,
```
- EN: The main symbol in this range is `PackedConvWeightCudnn`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `PackedConvWeightCudnn`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 85-96
```cpp
85:       std::optional<at::Tensor> bias,
86:       torch::List<int64_t> stride,
87:       torch::List<int64_t> padding,
88:       torch::List<int64_t> output_padding,
89:       torch::List<int64_t> dilation,
90:       int64_t groups,
91:       bool transpose,
92:       c10::QScheme q_scheme,
93:       int64_t output_channels)
94:       : maybe_padded_weight_(std::move(orig_weight)),
95:         bias_(std::move(bias)),
96:         stride_(stride),
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 97-108
```cpp
 97:         padding_(padding),
 98:         output_padding_(output_padding),
 99:         dilation_(dilation),
100:         groups_(groups),
101:         transpose_(transpose),
102:         q_scheme_(q_scheme),
103:         num_unpadded_output_channels_(output_channels) {} // output channels needs to be stored when we have to pad this dimension
104:
105:   at::Tensor apply(
106:       const at::Tensor& input,
107:       double output_scale,
108:       int64_t output_zero_point) override;
```
- EN: The main symbol in this range is `padding_`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `padding_`，它们直接构成本文件的算子逻辑。

### Lines 110-125
```cpp
110:   at::Tensor apply_relu(
111:       const at::Tensor& input,
112:       double output_scale,
113:       int64_t output_zero_point) override;
114:
115:   at::Tensor apply_dynamic(
116:     const at::Tensor& input,
117:     bool reduce_range) override {
118:     TORCH_CHECK(false, "apply_dynamic is currently not reported");
119:   }
120:
121:   at::Tensor apply_dynamic_relu(
122:     const at::Tensor& input,
123:     bool reduce_range) {
124:     TORCH_CHECK(false, "apply_dynamic_relu is currently not reported");
125:   }
```
- EN: The main symbol in this range is `apply_dynamic_relu`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `apply_dynamic_relu`，它们直接构成本文件的算子逻辑。

### Lines 127-137
```cpp
127:   std::tuple<at::Tensor, std::optional<at::Tensor>> unpack() override;
128:
129:   static c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> prepack(
130:       at::Tensor weight,
131:       std::optional<at::Tensor> bias,
132:       torch::List<int64_t> stride,
133:       torch::List<int64_t> padding,
134:       torch::List<int64_t> output_padding,
135:       torch::List<int64_t> dilation,
136:       int64_t groups,
137:       bool transpose);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 139-151
```cpp
139:   const float* GetBiasData(at::Tensor* bias);
140:
141:   torch::List<int64_t> stride() const override {
142:     return stride_;
143:   }
144:
145:   torch::List<int64_t> padding() const override {
146:     return padding_;
147:   }
148:
149:   torch::List<int64_t> output_padding() const override {
150:     return output_padding_;
151:   }
```
- EN: The main symbol in this range is `stride`, `padding`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `stride`, `padding`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 153-163
```cpp
153:   torch::List<int64_t> dilation() const override {
154:     return dilation_;
155:   }
156:
157:   int64_t groups() const override {
158:     return groups_;
159:   }
160:
161:   bool transpose() const override {
162:     return transpose_;
163:   }
```
- EN: The main symbol in this range is `dilation`, `groups`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `dilation`, `groups`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 165-179
```cpp
165:  private:
166:   // cudnn v8.4.0 expects conv2d's int8 weight tensor's input and output channels to be a multiple of 4. if it is not
167:   // we need to explicitly pad it to a multiple of 4 ourselves as cudnn does not currently support padding, hence the naming
168:   // convention "maybe"_padded_weight.
169:   // TODO: when and if cudnn enables padding in their operators, we can remove padding on our end and rename this to orig_weight_
170:   at::Tensor maybe_padded_weight_;
171:   std::optional<at::Tensor> bias_;
172:   torch::List<int64_t> stride_;
173:   torch::List<int64_t> padding_;
174:   torch::List<int64_t> output_padding_;
175:   torch::List<int64_t> dilation_;
176:   int64_t groups_;
177:   bool transpose_;
178:   c10::QScheme q_scheme_;
179:   int64_t num_unpadded_output_channels_;
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 181-192
```cpp
181:   template <bool ReluFused>
182:   at::Tensor apply_impl(
183:       const at::Tensor& input,
184:       double output_scale,
185:       int64_t output_zero_point);
186:
187:   template <bool ReluFused>
188:   void apply_impl_helper(
189:       const at::Tensor& quantized_output,
190:       const at::Tensor& input,
191:       double output_scale);
192: };
```
- EN: The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 194-205
```cpp
194: namespace cudnn_utils {
195:
196: // TODO: we can remove this function when cuDNN enables pass by value support for
197: // pointwise multiplication operations. the only reason why we need this right now is
198: // we use broadcasting scalar multiplication in conv, linear, and add ops, and cuDNN requires
199: // the scalar to be a scalar tensor with the same number of dimensions (num_dim) as the tensor we're multiplying to
200: inline at::Tensor getRequantMultiplierTensor(double requant_multiplier, uint8_t num_dim) {
201:   at::SmallVector<int64_t, 4> requantize_multiplier_tensor_size(num_dim, 1);
202:   at::Tensor requantize_multiplier_tensor = at::empty(requantize_multiplier_tensor_size, at::device(at::kCUDA).dtype(at::kFloat));
203:   requantize_multiplier_tensor.fill_(requant_multiplier);
204:   return requantize_multiplier_tensor;
205: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `dimensions`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `dimensions`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 207-217
```cpp
207: inline uint8_t getAlignment(const at::Tensor &t) {
208:   // alignment are in bytes
209:   uint8_t alignment = 1;
210:   uintptr_t address = reinterpret_cast<uintptr_t>(t.data_ptr());
211:   for (; alignment < 16; alignment *= 2) {
212:     if (address % (alignment * 2)) {
213:       return alignment;
214:     }
215:   }
216:   return alignment;
217: }
```
- EN: The main symbol in this range is `getAlignment`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `getAlignment`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 219-229
```cpp
219: // For the two getTensorDescriptor functions, there is a is_virtual parameter. This parameter is used to set the cudnn
220: // tensor as virtual or not. Setting the tensor as virtual is expected to have some performance benefits as the cudnn
221: // backend cudnn will no longer directly save to the tensor, allowing us to omit this tensor from the variant pack.
222: // See third_party/cudnn_frontend/samples/fusion_sample.cpp for other examples
223:
224: inline cudnn_frontend::Tensor getTensorDescriptor(const at::Tensor &t, int64_t id, uint8_t alignment, bool is_virtual = false) {
225:   auto shape = t.sizes();
226:   auto strides = t.strides();
227:   if (is_virtual) {
228:     return cudnn_frontend::TensorBuilder()
229:       .setDim(shape.size(), shape.data())
```
- EN: The main symbol in this range is `getTensorDescriptor`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `getTensorDescriptor`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 230-244
```cpp
230:       .setStrides(strides.size(), strides.data())
231:       .setId(id)
232:       .setAlignment(alignment)
233:       .setVirtual()
234:       .setDataType(at::native::getCudnnDataType(t))
235:       .build();
236:   }
237:   return cudnn_frontend::TensorBuilder()
238:     .setDim(shape.size(), shape.data())
239:     .setStrides(strides.size(), strides.data())
240:     .setId(id)
241:     .setAlignment(alignment)
242:     .setDataType(at::native::getCudnnDataType(t))
243:     .build();
244: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 246-256
```cpp
246: inline cudnn_frontend::Tensor getTensorDescriptor(const c10::IntArrayRef& shape, const c10::IntArrayRef& strides, cudnnDataType_t cudnn_dtype, int64_t id, uint8_t alignment, bool is_virtual = false) {
247:   if (is_virtual) {
248:     return cudnn_frontend::TensorBuilder()
249:       .setDim(shape.size(), shape.data())
250:       .setStrides(strides.size(), strides.data())
251:       .setId(id)
252:       .setAlignment(alignment)
253:       .setVirtual()
254:       .setDataType(cudnn_dtype)
255:       .build();
256:   }
```
- EN: The main symbol in this range is `getTensorDescriptor`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `getTensorDescriptor`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 257-273
```cpp
257:   return cudnn_frontend::TensorBuilder()
258:     .setDim(shape.size(), shape.data())
259:     .setStrides(strides.size(), strides.data())
260:     .setId(id)
261:     .setAlignment(alignment)
262:     .setDataType(cudnn_dtype)
263:     .build();
264: }
265:
266: // TODO: there is a table from input dtype to operator dtype, we can derive
267: // the operator dtype based on input dtype
268: inline cudnn_frontend::PointWiseDesc_v8 getPointWiseMulDescriptor(cudnnDataType_t dataType) {
269:   return cudnn_frontend::PointWiseDescBuilder()
270:     .setMode(cudnnPointwiseMode_t::CUDNN_POINTWISE_MUL)
271:     .setMathPrecision(dataType)
272:     .build();
273: }
```
- EN: The main symbol in this range is `getPointWiseMulDescriptor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `getPointWiseMulDescriptor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 275-285
```cpp
275: // TODO: there is a table from input dtype to operator dtype, we can derive
276: // the operator dtype based on input dtype
277: inline cudnn_frontend::PointWiseDesc_v8 getPointWiseAddDescriptor(cudnnDataType_t dataType) {
278:   return cudnn_frontend::PointWiseDescBuilder()
279:     .setMode(cudnnPointwiseMode_t::CUDNN_POINTWISE_ADD)
280:     .setMathPrecision(dataType)
281:     .build();
282: }
283:
284: // TODO: there is a table from input dtype to operator dtype, we can derive
285: // the operator dtype based on input dtype
```
- EN: The main symbol in this range is `getPointWiseAddDescriptor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `getPointWiseAddDescriptor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 286-297
```cpp
286: inline cudnn_frontend::PointWiseDesc_v8 getPointWiseReluDescriptor(cudnnDataType_t dataType) {
287:   return cudnn_frontend::PointWiseDescBuilder()
288:     .setMode(cudnnPointwiseMode_t::CUDNN_POINTWISE_RELU_FWD)
289:     .setMathPrecision(dataType)
290:     .build();
291: }
292:
293:
294: inline void filterEngineConfigs(
295:   cudnn_frontend::EngineConfigList &from,
296:   cudnn_frontend::EngineConfigList &to,
297:   bool deterministic, bool allow_tf32, c10::ScalarType scalar_type)
```
- EN: The main symbol in this range is `getPointWiseReluDescriptor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `getPointWiseReluDescriptor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 298-310
```cpp
298: {
299:   auto filter = [=](cudnnBackendDescriptor_t c) {
300:     if (deterministic) {
301:       if (cudnn_frontend::hasNumericalNote<CUDNN_NUMERICAL_NOTE_NONDETERMINISTIC>(c)) return true;
302:     }
303:     if (scalar_type == at::kFloat || scalar_type == at::kChar || !allow_tf32) {
304:       if (cudnn_frontend::hasNumericalNote<CUDNN_NUMERICAL_NOTE_DOWN_CONVERT_INPUTS>(c)) return true;
305:       if (cudnn_frontend::hasNumericalNote<CUDNN_NUMERICAL_NOTE_TENSOR_CORE>(c)) return true;
306:     }
307:     return false;
308:   };
309:   cudnn_frontend::filter(from, to, filter);
310: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 312-315
```cpp
312: } // cudnn_utils
313:
314: #endif  // AT_CUDNN_ENABLED
315: #endif  // USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/cuda/CUDAConfig.h`, `ATen/cudnn/Types.h`, `ATen/Tensor.h`, `ATen/native/quantized/PackedParams.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/core/QScheme.h`, `c10/util/ArrayRef.h`, `c10/util/Exception.h`
- Standard or third-party headers / 标准库或第三方头文件: `cudnn_frontend.h`
- Key helper symbols / 关键辅助符号: `SmallVector`, `Scalar`, `ScalarType`
