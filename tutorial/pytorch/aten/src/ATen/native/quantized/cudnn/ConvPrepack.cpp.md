# ConvPrepack.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cudnn/ConvPrepack.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU convolution helpers, packed-parameter handling, or output-shape logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 卷积辅助逻辑、打包参数处理或输出形状计算。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
 1: #ifdef USE_CUDA
 2: #include <ATen/cuda/CUDAConfig.h>  // for the definition of AT_CUDNN_ENABLED
 3:
 4: #if AT_CUDNN_ENABLED()
 5:
 6: #include <ATen/ATen.h>
 7: #include <torch/library.h>
 8: #include <ATen/native/quantized/cpu/QuantUtils.h>
 9: #include <ATen/native/quantized/cudnn/utils.h>
10: #include <ATen/native/quantized/library.h>
11: #include <ATen/native/quantized/PackedParams.h>
12: #include <ATen/quantized/Quantizer.h>
13: #include <c10/core/QScheme.h>
14: #include <c10/util/irange.h>
```
- EN: This range pulls in required headers, including `ATen/cuda/CUDAConfig.h`, `ATen/ATen.h`, `torch/library.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/cuda/CUDAConfig.h`, `ATen/ATen.h`, `torch/library.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 16-26
```cpp
16: #include <utility>
17:
18: template <int kSpatialDim>
19: c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> PackedConvWeightCudnn<
20:     kSpatialDim>::
21:     prepack(
22:         at::Tensor weight,
23:         std::optional<at::Tensor> bias,
24:         torch::List<int64_t> stride,
25:         torch::List<int64_t> padding,
26:         torch::List<int64_t> output_padding,
```
- EN: This range pulls in required headers, including `utility`. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段引入了所需头文件，例如 `utility`。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 27-38
```cpp
27:         torch::List<int64_t> dilation,
28:         int64_t groups,
29:         bool transpose) {
30:   // TODO: need to check out to implement groups for conv operator in Conv.cpp
31:   TORCH_CHECK(groups == 1, "Quantized cudnn conv2d is currently limited to groups = 1; received groups =", groups);
32:   TORCH_CHECK(weight.qscheme() == c10::kPerTensorAffine, "Unsupported qscheme: ", toString(weight.qscheme()));
33:   TORCH_CHECK(
34:       kSpatialDim == 2,  // 1D is packed as 2d, hence we don't need other checks
35:       "cuDNN packing only supports 2D convolution.");
36:   TORCH_CHECK(
37:       weight.ndimension() == kSpatialDim + 2,
38:       "Weights are expected to have ",
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 39-50
```cpp
39:       kSpatialDim + 2,
40:       " dimensions");
41:   TORCH_CHECK(
42:       stride.size() == kSpatialDim,
43:       "stride should contain ",
44:       kSpatialDim,
45:       " elements for ",
46:       kSpatialDim,
47:       "D convolution.");
48:   TORCH_CHECK(
49:       padding.size() == kSpatialDim,
50:       "quantized::conv_prepack (cudnn): Specify front/top/left padding only. "
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 51-62
```cpp
51:       "end/bottom/right padding assumed to be equal to front/top/left");
52:   TORCH_CHECK(
53:       !transpose || output_padding.size() == kSpatialDim,
54:       "quantized::conv_prepack: Specify top/left output padding "
55:       "only. bottom/right padding assumed to be equal to top/left");
56:   TORCH_CHECK(
57:       dilation.size() == kSpatialDim,
58:       "quantized::conv_prepack (cudnn): dilation should contain ",
59:       kSpatialDim,
60:       " elements for ",
61:       kSpatialDim,
62:       "D convolution.");
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 63-74
```cpp
63:   TORCH_CHECK(!transpose, "cudNN quantized conv prepack expects transpose = false")
64:   const auto num_unpadded_output_channels = weight.size(0);
65:   const auto qtype = weight.qscheme();
66:   if (bias.has_value()) {
67:     TORCH_CHECK(bias.value().dim() == 1, "bias should be a vector (1D Tensor)");
68:     TORCH_CHECK(
69:         bias.value().size(0) == num_unpadded_output_channels,
70:         "bias should have K elements: " + std::to_string(num_unpadded_output_channels));
71:     // TODO: we create a broadcasted_bias tensor later so I think we don't need to make this contiguous here.
72:     // we will revisit this when nvidia adds proper support for broadcasting
73:     // bias_contig = bias->contiguous();
74:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 76-91
```cpp
76:   // cudnn v8.4.0 expects conv2d's int8 weight tensor's input and output channels to be a multiple of 4. if it is not
77:   // we need to explicitly pad it to a multiple of 4 ourselves as cudnn does not currently support padding.
78:   // TODO: when and if cudnn enables padding in their operators, we can remove padding on our end;
79:   // currently, limit padding support to groups=1 (ungrouped conv)
80:   // TODO: implement this for groups > 1
81:   auto num_input_channels = weight.size(1);
82:   auto num_output_slices2pad = (4 - num_unpadded_output_channels % 4) % 4;
83:   auto num_input_slices2pad = (4 - num_input_channels % 4) % 4;
84:   if (num_output_slices2pad != 0 || num_input_slices2pad != 0) {
85:     // the second argument is an initializer list of padded values. there are 2 values for each dimension.
86:     // refer to https://pytorch.org/docs/stable/generated/torch.nn.functional.pad.html for more details
87:     weight = at::pad(weight, {0, 0, 0, 0, 0, num_input_slices2pad, 0, num_output_slices2pad}, "constant", 0);
88:     if (bias.has_value()) {
89:       bias.value() = at::pad(bias.value(), {0, num_output_slices2pad}, "constant", 0);
90:     }
91:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 93-105
```cpp
 93:   auto ret_ptr = c10::make_intrusive<PackedConvWeightCudnn<kSpatialDim>>(
 94:           weight.to(c10::MemoryFormat::ChannelsLast), // TODO: this assumes 2D I think. make it more general?
 95:           std::move(bias),
 96:           stride,
 97:           padding,
 98:           output_padding,
 99:           dilation,
100:           groups,
101:           transpose,
102:           qtype,
103:           num_unpadded_output_channels);
104:   return ret_ptr;
105: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 107-118
```cpp
107: template
108: c10::intrusive_ptr<ConvPackedParamsBase<2>> PackedConvWeightCudnn<
109:     2>::
110:     prepack(
111:         at::Tensor weight,
112:         std::optional<at::Tensor> bias_in,
113:         torch::List<int64_t> stride,
114:         torch::List<int64_t> padding,
115:         torch::List<int64_t> output_padding,
116:         torch::List<int64_t> dilation,
117:         int64_t groups,
118:         bool transpose);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 121-130
```cpp
121: namespace at::native {
122: namespace {
123:
124: template <int kSpatialDim = 2>
125: class QConvPackWeightInt8Cudnn final {
126:  public:
127:   static c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> run_conv(
128:       Tensor weight,
129:       std::optional<Tensor> bias,
130:       torch::List<int64_t> stride,
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `QConvPackWeightInt8Cudnn`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `QConvPackWeightInt8Cudnn`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 131-141
```cpp
131:       torch::List<int64_t> padding,
132:       torch::List<int64_t> dilation,
133:       int64_t groups) {
134:     torch::List<int64_t> output_padding;
135:     output_padding.reserve(kSpatialDim);
136:     for ([[maybe_unused]] const auto idx : c10::irange(kSpatialDim)) {
137:       output_padding.push_back(0);
138:     }
139:     return _run(weight, bias, stride, padding, output_padding, dilation, groups,
140:                 /*transpose=*/false);
141:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 143-157
```cpp
143:  private:
144:   static c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>> _run(
145:       Tensor weight,
146:       std::optional<Tensor> bias,
147:       torch::List<int64_t> stride,
148:       torch::List<int64_t> padding,
149:       torch::List<int64_t> output_padding,
150:       torch::List<int64_t> dilation,
151:       int64_t groups,
152:       bool transpose) {
153:     return PackedConvWeightCudnn<kSpatialDim>::prepack(
154:         weight, bias, stride, padding, output_padding, dilation, groups,
155:         transpose);
156:   }
157: };
```
- EN: The main symbol in this range is `_run`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_run`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 159-171
```cpp
159: class QConv1dPackWeightInt8Cudnn final {
160:  public:
161:   static c10::intrusive_ptr<ConvPackedParamsBase<2>> run_conv(
162:       Tensor weight,
163:       std::optional<Tensor> bias,
164:       torch::List<int64_t> stride,
165:       torch::List<int64_t> padding,
166:       torch::List<int64_t> dilation,
167:       int64_t groups) {
168:     const torch::List<int64_t> output_padding({0});
169:     return _run(std::move(weight), std::move(bias), stride, padding, output_padding, dilation, groups,
170:                 /*transpose=*/false);
171:   }
```
- EN: The main symbol in this range is `run_conv`, `QConv1dPackWeightInt8Cudnn`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run_conv`, `QConv1dPackWeightInt8Cudnn`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 173-183
```cpp
173:  private:
174:   static c10::intrusive_ptr<ConvPackedParamsBase<2>> _run(
175:       Tensor weight,
176:       std::optional<Tensor> bias,
177:       torch::List<int64_t> stride,
178:       torch::List<int64_t> padding,
179:       torch::List<int64_t> output_padding,
180:       torch::List<int64_t> dilation,
181:       int64_t groups,
182:       bool transpose) {
183:     if (weight.dim() == 3) {
```
- EN: The main symbol in this range is `_run`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_run`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 184-198
```cpp
184:       // we currently use conv2d kernel for conv1d by making the input and weight tensors
185:       // 4D rather than 3D. we add a dummy width dimension of size 1
186:       // out channels, in channels / groups, L -> out channels, in channels / groups, 1, L
187:       weight = weight.unsqueeze(-2);
188:     }
189:     stride = quant_utils::MakeArgForConv1d(stride, 1);
190:     padding = quant_utils::MakeArgForConv1d(padding, 0);
191:     output_padding = quant_utils::MakeArgForConv1d(output_padding, 0);
192:     dilation = quant_utils::MakeArgForConv1d(dilation, 1);
193:
194:     return PackedConvWeightCudnn<2>::prepack(
195:         weight, std::move(bias), stride, padding, output_padding, dilation, groups,
196:         transpose);
197:   }
198: };
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 200-210
```cpp
200: TORCH_LIBRARY_IMPL(quantized, QuantizedCUDA, m) {
201:   register_conv_params<2>();
202:   register_conv_params<3>();
203:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv1d_prepack"), TORCH_FN(QConv1dPackWeightInt8Cudnn::run_conv));
204:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_prepack"), TORCH_FN(QConvPackWeightInt8Cudnn<2>::run_conv));
205: }
206:
207: } // namespace
208: } // namespace at::native
209:
210: #endif  // AT_CUDNN_ENABLED
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 211-211
```cpp
211: #endif  // USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/cuda/CUDAConfig.h`, `ATen/ATen.h`, `ATen/native/quantized/cpu/QuantUtils.h`, `ATen/native/quantized/cudnn/utils.h`, `ATen/native/quantized/library.h`, `ATen/native/quantized/PackedParams.h`, `ATen/quantized/Quantizer.h`
- c10 headers / c10 头文件: `c10/core/QScheme.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `utility`
