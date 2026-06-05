# qconv_unpack.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/qconv_unpack.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU convolution helpers, packed-parameter handling, or output-shape logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 卷积辅助逻辑、打包参数处理或输出形状计算。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
 1: /*
 2: The dispatch registrations at the end of this file applies to fbgemm, qnnpack, and cudnn backends.
 3: The correct unpack backend function is determined using runtime polymorphism through the packed_weight pointer,
 4: which is of type intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> and points to either a PackedConvWeightsQnnp,
 5: PackedConvWeights (Fbgemm), or PackedConvWeightsCudnn at runtime, which all inherit from ConvPackedParamsBase.
 6: The implementations for the unpack functions can be found in /cpu/qconv_unpack_impl.cpp, for fbgemm&qnnpack
 7: and /cudnn/ConvUnpackImpl.cpp, for cudnn.
 8: */
 9:
10: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
11: #include <tuple>
```
- EN: This range pulls in required headers, including `tuple`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `tuple`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 13-22
```cpp
13: #include <ATen/core/Tensor.h>
14: #include <ATen/core/List.h>
15: #include <ATen/core/ivalue.h>
16: #include <torch/library.h>
17: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
18: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
19: #include <ATen/native/quantized/cpu/OnednnUtils.h>
20: #include <ATen/native/quantized/cpu/QuantUtils.h>
21: #include <ATen/native/quantized/PackedParams.h>
22: #include <ATen/native/quantized/library.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/core/ivalue.h`. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/core/ivalue.h`。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 24-34
```cpp
24: #ifndef AT_PER_OPERATOR_HEADERS
25: #include <ATen/Functions.h>
26: #else
27: #include <ATen/ops/_empty_affine_quantized.h>
28: #include <ATen/ops/_empty_per_channel_affine_quantized.h>
29: #include <ATen/ops/from_blob.h>
30: #endif
31:
32:
33: namespace at::native {
34: namespace {
```
- EN: This range pulls in required headers, including `ATen/Functions.h`, `ATen/ops/_empty_affine_quantized.h`, `ATen/ops/_empty_per_channel_affine_quantized.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/Functions.h`, `ATen/ops/_empty_affine_quantized.h`, `ATen/ops/_empty_per_channel_affine_quantized.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 36-48
```cpp
36: /*
37:  * QConvPackWeightInt8 expects its input tensor to be in shape
38:  * [output_channels, kernel_height, kernel_width, input_channels/Groups]
39:  * Therefore, the unpacking of packed weight tensor using QConvUnpackWeightsInt8
40:  * results in a tensor of the same shape.
41:  */
42:
43: template <int kSpatialDim = 2>
44: class QConvUnpackWeightsInt8 final {
45:  public:
46:   static std::tuple<at::Tensor, std::optional<at::Tensor>> run(
47:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
48:     auto& ctx = at::globalContext();
```
- EN: The main symbol in this range is `run`, `QConvUnpackWeightsInt8`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `run`, `QConvUnpackWeightsInt8`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 50-65
```cpp
50: #ifdef USE_FBGEMM
51:     if (ctx.qEngine() == at::QEngine::FBGEMM ||
52:         ctx.qEngine() == at::QEngine::X86) {
53:       return packed_weight->unpack();
54:     }
55: #endif
56:
57: #ifdef USE_PYTORCH_QNNPACK
58:     if (ctx.qEngine() == at::QEngine::QNNPACK) {
59:       TORCH_CHECK(
60:           kSpatialDim == 2,
61:           "quantized::conv2d_unpack (qnnpack): QNNPACK only supports Conv2d "
62:           "now.");
63:       return packed_weight->unpack();
64:     }
65: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 67-78
```cpp
67: #if AT_MKLDNN_ENABLED()
68:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
69:       return packed_weight->unpack();
70:     }
71: #endif
72:
73:     TORCH_CHECK(
74:         false,
75:         "Didn't find engine for operation quantized::conv2d_unpack ",
76:         toString(ctx.qEngine()));
77:   }
78: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 80-94
```cpp
80: class QConv1dUnpackWeightsInt8 final {
81:  public:
82:   static std::tuple<at::Tensor, std::optional<at::Tensor>> run(
83:       const c10::intrusive_ptr<ConvPackedParamsBase<2>>& packed_weight) {
84:     auto& ctx = at::globalContext();
85:     at::Tensor weight;
86:     std::optional<at::Tensor> bias;
87: #ifdef USE_FBGEMM
88:     if (ctx.qEngine() == at::QEngine::FBGEMM ||
89:         ctx.qEngine() == at::QEngine::X86) {
90:       std::tie(weight, bias) = packed_weight->unpack();
91:       weight = weight.squeeze_(quant_utils::kConv1dSqueezeDim + 2);
92:       return std::tuple<at::Tensor, std::optional<at::Tensor>>(weight, bias);
93:     }
94: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `run`, `QConv1dUnpackWeightsInt8`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `run`, `QConv1dUnpackWeightsInt8`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 96-106
```cpp
 96: #ifdef USE_PYTORCH_QNNPACK
 97:     if (ctx.qEngine() == at::QEngine::QNNPACK) {
 98:       std::tie(weight, bias) = packed_weight->unpack();
 99:       at::Tensor new_weight = weight.clone();
100:       new_weight = new_weight.squeeze_(quant_utils::kConv1dSqueezeDim + 2);
101:       return std::tuple<at::Tensor, std::optional<at::Tensor>>(new_weight, bias);
102:     }
103: #endif
104:
105: #if AT_MKLDNN_ENABLED()
106:     if (ctx.qEngine() == at::QEngine::ONEDNN) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 107-119
```cpp
107:       std::tie(weight, bias) = packed_weight->unpack();
108:       at::Tensor new_weight = weight.clone();
109:       new_weight.squeeze_(quant_utils::kConv1dSqueezeDim + 2);
110:       return std::tuple<at::Tensor, std::optional<at::Tensor>>(new_weight, bias);
111:     }
112: #endif
113:
114:     TORCH_CHECK(
115:         false,
116:         "Didn't find engine for operation quantized::conv1d_unpack ",
117:         toString(ctx.qEngine()));
118:   }
119: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 121-131
```cpp
121: template <int kSpatialDim = 2>
122: class QConvStride final {
123:  public:
124:   static torch::List<int64_t> run(
125:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
126:     return packed_weight->stride();
127:   }
128: };
129:
130: template <int kSpatialDim = 2>
131: class QConvPadding final {
```
- EN: The main symbol in this range is `run`, `QConvStride`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QConvStride`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 132-146
```cpp
132:  public:
133:   static torch::List<int64_t> run(
134:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
135:     return packed_weight->padding();
136:   }
137: };
138:
139: template <int kSpatialDim = 2>
140: class QConvOutputPadding final {
141:  public:
142:   static torch::List<int64_t> run(
143:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
144:     return packed_weight->output_padding();
145:   }
146: };
```
- EN: The main symbol in this range is `run`, `QConvOutputPadding`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QConvOutputPadding`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 148-158
```cpp
148: template <int kSpatialDim = 2>
149: class QConvDilation final {
150:  public:
151:   static torch::List<int64_t> run(
152:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
153:     return packed_weight->dilation();
154:   }
155: };
156:
157: template <int kSpatialDim = 2>
158: class QConvGroups final {
```
- EN: The main symbol in this range is `run`, `QConvDilation`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QConvDilation`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 159-173
```cpp
159:  public:
160:   static int64_t run(
161:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
162:     return packed_weight->groups();
163:   }
164: };
165:
166: template <int kSpatialDim = 2>
167: class QConvTranspose final {
168:  public:
169:   static int64_t run(
170:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight) {
171:     return packed_weight->transpose();
172:   }
173: };
```
- EN: The main symbol in this range is `run`, `QConvTranspose`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QConvTranspose`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 175-190
```cpp
175: IValue
176: unpack_quantized_prepacked_sizes_conv2d(const IValue& ivalue) {
177:   auto params = ivalue.toCustomClass<ConvPackedParamsBase<2>>();
178:   auto [weight, bias] = params->unpack();
179:   at::OptionalIntArrayRef bias_sizes = std::nullopt;
180:   if (bias && bias->defined()) {
181:     bias_sizes = bias->sizes();
182:   }
183:   return IValue(std::make_tuple(
184:       weight.sizes(),
185:       bias_sizes,
186:       params->stride(),
187:       params->padding(),
188:       params->dilation(),
189:       params->groups()));
190: }
```
- EN: The main symbol in this range is `unpack_quantized_prepacked_sizes_conv2d`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `unpack_quantized_prepacked_sizes_conv2d`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 192-201
```cpp
192: TORCH_LIBRARY_IMPL(quantized, CatchAll, m) {
193:   register_conv_params<2>();
194:   register_conv_params<3>();
195:   // conv_unpack is deprecated, please use conv2d_unpack for 2D conv.
196:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_unpack"), TORCH_FN(QConvUnpackWeightsInt8<2>::run));
197:   // We use  conv2d_unpack to be consistent with conv3d_unpack
198:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv1d_unpack"), TORCH_FN(QConv1dUnpackWeightsInt8::run));
199:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_unpack"), TORCH_FN(QConvUnpackWeightsInt8<2>::run));
200:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_unpack_sizes"), TORCH_FN(unpack_quantized_prepacked_sizes_conv2d));
201:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_unpack"), TORCH_FN(QConvUnpackWeightsInt8<3>::run));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 203-215
```cpp
203:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_stride"), TORCH_FN(QConvStride<2>::run));
204:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_padding"), TORCH_FN(QConvPadding<2>::run));
205:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_output_padding"), TORCH_FN(QConvOutputPadding<2>::run));
206:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_dilation"), TORCH_FN(QConvDilation<2>::run));
207:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_groups"), TORCH_FN(QConvGroups<2>::run));
208:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_transpose"), TORCH_FN(QConvTranspose<2>::run));
209:
210:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_stride"), TORCH_FN(QConvStride<3>::run));
211:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_padding"), TORCH_FN(QConvPadding<3>::run));
212:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_output_padding"), TORCH_FN(QConvOutputPadding<3>::run));
213:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_dilation"), TORCH_FN(QConvDilation<3>::run));
214:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_groups"), TORCH_FN(QConvGroups<3>::run));
215:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv3d_transpose"), TORCH_FN(QConvTranspose<3>::run));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 217-227
```cpp
217:   // ConvTranspose is the same, however, we want to have different name.
218:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose1d_unpack"), TORCH_FN(QConv1dUnpackWeightsInt8::run));
219:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_unpack"), TORCH_FN(QConvUnpackWeightsInt8<2>::run));
220:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_unpack"), TORCH_FN(QConvUnpackWeightsInt8<3>::run));
221:
222:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_stride"), TORCH_FN(QConvStride<2>::run));
223:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_padding"), TORCH_FN(QConvPadding<2>::run));
224:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_output_padding"), TORCH_FN(QConvOutputPadding<2>::run));
225:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_dilation"), TORCH_FN(QConvDilation<2>::run));
226:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_groups"), TORCH_FN(QConvGroups<2>::run));
227:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose2d_transpose"), TORCH_FN(QConvTranspose<2>::run));
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 228-237
```cpp
228:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_stride"), TORCH_FN(QConvStride<3>::run));
229:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_padding"), TORCH_FN(QConvPadding<3>::run));
230:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_output_padding"), TORCH_FN(QConvOutputPadding<3>::run));
231:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_dilation"), TORCH_FN(QConvDilation<3>::run));
232:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_groups"), TORCH_FN(QConvGroups<3>::run));
233:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv_transpose3d_transpose"), TORCH_FN(QConvTranspose<3>::run));
234: }
235:
236: } // namespace
237: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/core/ivalue.h`, `ATen/native/quantized/cpu/fbgemm_utils.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`, `ATen/native/quantized/cpu/OnednnUtils.h`, `ATen/native/quantized/cpu/QuantUtils.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/library.h`, `ATen/Functions.h`
- Standard or third-party headers / 标准库或第三方头文件: `tuple`, `torch/library.h`
- Key helper symbols / 关键辅助符号: `qnnpack`, `fbgemm`
