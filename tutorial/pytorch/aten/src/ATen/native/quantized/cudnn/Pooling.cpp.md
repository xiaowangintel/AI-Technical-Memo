# Pooling.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cudnn/Pooling.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU pooling kernels and pooled-output shape or reduction logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 池化 kernel，以及池化输出形状或归约逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
 1: #include <c10/util/Exception.h>
 2: #ifdef USE_CUDA
 3: #include <ATen/cuda/CUDAConfig.h>  // for the definition of AT_CUDNN_ENABLED
 4:
 5: #if AT_CUDNN_ENABLED()
 6: #include <ATen/cuda/Exceptions.h>
 7: #include <ATen/cudnn/Descriptors.h>
 8: #include <ATen/cudnn/Handle.h>
 9: #include <ATen/cudnn/Types.h>
10: #endif // AT_CUDNN_ENABLED
11: #endif // USE_CUDA
```
- EN: This range pulls in required headers, including `c10/util/Exception.h`, `ATen/cuda/CUDAConfig.h`, `ATen/cuda/Exceptions.h`. It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 这一段引入了所需头文件，例如 `c10/util/Exception.h`, `ATen/cuda/CUDAConfig.h`, `ATen/cuda/Exceptions.h`。 它还建立了编译期开关、特性选择或平台相关分支。

### Lines 13-21
```cpp
13: #include <ATen/ATen.h>
14: #include <ATen/native/Pool.h>
15: #include <ATen/native/TensorIterator.h>
16: #include <c10/core/QScheme.h>
17: #include <c10/core/ScalarType.h>
18: #include <c10/util/ArrayRef.h>
19: #include <torch/library.h>
20:
21: #include <vector>
```
- EN: This range pulls in required headers, including `ATen/ATen.h`, `ATen/native/Pool.h`, `ATen/native/TensorIterator.h`. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段引入了所需头文件，例如 `ATen/ATen.h`, `ATen/native/Pool.h`, `ATen/native/TensorIterator.h`。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 24-33
```cpp
24: namespace at::native {
25: namespace {
26: // TODO: This function is the same as that of Pooling.cpp. We should refactor this into quantized directory
27: // so that we don't need to duplicate the function
28: #ifdef USE_CUDA
29: #if AT_CUDNN_ENABLED()
30: void check_maxpool2d_params(
31:     IntArrayRef kernel_size,
32:     IntArrayRef stride,
33:     IntArrayRef padding,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 34-46
```cpp
34:     IntArrayRef dilation) {
35:   TORCH_CHECK(kernel_size.size() == 1 || kernel_size.size() == 2,
36:               "Expected 1d or 2d kernel size, got ", kernel_size.size());
37:   TORCH_CHECK(stride.empty() || stride.size() == 2,
38:               "Expected no strides or 2d strides, got", stride.size());
39:   TORCH_CHECK(padding.size() == 1 || padding.size() == 2,
40:               "Expected 1d or 2d padding, got ", padding.size());
41:   TORCH_CHECK(dilation.size() == 1 || dilation.size() == 2,
42:               "Expected 1d or 2d dilation, got ", dilation.size());
43: }
44: #endif
45: #endif
46: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。

### Lines 48-58
```cpp
48: // The current implementation of quantized cuda adaptive average pooling uses the following:
49: // dequant -> fp32 adaptive average pooling -> quant. This is the same numerically as
50: // quantized adaptive average pooling. This is not the ideal implementation, as we desire to
51: // operate on the quantized values directly.
52: // However, we are currently blocked on this as we are waiting for cudnn's 8.5.0 release, which is anticipated
53: // to support adaptive average pooling. When that support is made available, we will use it directly. TODO
54: Tensor adaptive_avg_pool2d_quantized_cuda(
55:     const at::Tensor& input,
56:     IntArrayRef output_size) {
57: // TODO: re-enable these cudnn preprocessors like quantized_max_pool2d_cudnn below when we implement this function with cudnn
58: #ifdef USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `adaptive_avg_pool2d_quantized_cuda`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `adaptive_avg_pool2d_quantized_cuda`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 59-69
```cpp
59: // #if AT_CUDNN_ENABLED()
60:     // TODO: limit this to per tensor quantized tensors for now, though should be easy to adapt
61:     // to per channel quantized tensors
62:     TORCH_CHECK(input.qscheme() == at::kPerTensorAffine, "adaptive_avg_pool2d_quantized_cuda oonly supports per tensor quantized tensors");
63:     auto input_fp32 = at::dequantize(input);
64:     auto result_fp32 = at::adaptive_avg_pool2d(input_fp32, output_size);
65:     return at::quantize_per_tensor(result_fp32, input.q_scale(), input.q_zero_point(), input.scalar_type());
66: #else // USE_CUDA
67:   TORCH_CHECK(false, "at::native::adaptive_avg_pool2d_quantized_cuda: ATen not compiled with USE_CUDA support");
68: #endif
69: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 71-81
```cpp
71: // Currently we support 4D and 3D input (qx) tensors, the latter of which is supported for
72: // legacy reasons. The first dimension of a 4D input tensor is the batch size.
73: // For a 3D tensor, there is no batch size dimension -- it can be viewed as a single batch.
74: // cudnn's 2D pooling operation requires the input and output to be 4D tensors, so we must cast
75: // any 3D tensors to 4D prior to using cudnn
76: // This implementation currently uses the v7 cudnn APIs as v8 cudnn APIs are not yet available for
77: // pooling operations.
78: // Consult https://docs.nvidia.com/deeplearning/cudnn/backend/latest/api/cudnn-ops-library.html#cudnnpoolingforward for
79: // documentation on the APIs
80: // Currently, it appears there is no cudnn support for dilated pooling -- we will
81: // submit a feature request for this with cudnn
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 82-93
```cpp
82: // TODO: ideally, we would like to use structured kernel support here so we do not have to repeat
83: // the input checks, however, that would require us to implement max_pool2d_with_indices_out_quantized_cuda
84: // based on how the dispatch table is currently constructed in native_functions.yaml. currently,
85: // there is no support for producing indices with cudnn max pooling, so until that becomes available, this cannot be done.
86: Tensor quantized_max_pool2d_cudnn(
87:     const Tensor& qx,
88:     IntArrayRef kernel_size,
89:     IntArrayRef stride,
90:     IntArrayRef padding,
91:     IntArrayRef dilation,
92:     bool ceil_mode) {
93: #ifdef USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `quantized_max_pool2d_cudnn`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `quantized_max_pool2d_cudnn`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 94-105
```cpp
 94: #if AT_CUDNN_ENABLED()
 95:   check_maxpool2d_params(
 96:       kernel_size,
 97:       stride,
 98:       padding,
 99:       dilation);
100:   if (stride.empty()) {
101:     stride = kernel_size;
102:   }
103:   auto ndim = qx.dim();
104:   TORCH_CHECK(
105:       ndim == 3 || ndim == 4, "Expecting the input tensor of rank 3 or 4.");
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 106-117
```cpp
106:   TORCH_CHECK(
107:       kernel_size.size() == 2,
108:       "quantized_max_pool2d_cudnn(): Expected kernel_size to be 2-dimensional: got ",
109:       kernel_size.size());
110:   TORCH_CHECK(
111:       stride.size() == 2,
112:       "quantized_max_pool2d_cudnn(): Expected stride to be 2-dimensional: got ",
113:       stride.size());
114:   TORCH_CHECK(
115:       dilation.size() == 2,
116:       "quantized_max_pool2d_cudnn(): Expected dilation to be 2-dimensional: got ",
117:       dilation.size());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 118-129
```cpp
118:   TORCH_CHECK(
119:       dilation[0] == 1 && dilation[1] == 1,
120:       "quantized_max_pool2d_cudnn(): Expected dilation=[1, 1] (cudnn does not currently support dilation[i] != 1), got",
121:       dilation);
122:   TORCH_CHECK(
123:       padding.size() == 2,
124:       "quantized_max_pool2d_cudnn(): Expected padding to be 2-dimensional: got ",
125:       padding.size());
126:
127:   auto input = qx;
128:   if (ndim == 4) {
129:     input = qx.to(MemoryFormat::ChannelsLast);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 130-141
```cpp
130:   } else { // 3D
131:     std::vector<int64_t> new_sizes{1, qx.size(0), qx.size(1), qx.size(2)};
132:     input = qx.view(new_sizes);
133:   }
134:   int batch_size = input.size(0);
135:   int64_t inC = input.size(1);
136:   int64_t inH = input.size(2);
137:   int64_t inW = input.size(3);
138:   // Check output dimensions.
139:   int64_t padH = padding[0];
140:   int64_t padW = padding[1];
141:   int64_t kH = kernel_size[0];
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 142-153
```cpp
142:   int64_t kW = kernel_size[1];
143:   int64_t strideH = stride[0];
144:   int64_t strideW = stride[1];
145:   TORCH_CHECK(
146:       kH > 0 && kW > 0,
147:       "qnnpack_maxpool2d(): kernel_size should be greater than zero.");
148:   TORCH_CHECK(
149:       strideH > 0 && strideW > 0,
150:       "qnnpack_maxpool2d(): strides should be greater than zero.");
151:   int64_t dilationH = dilation[0];
152:   int64_t dilationW = dilation[1];
153:   int64_t outC = inC;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 154-165
```cpp
154:   int64_t outH = pooling_output_shape(inH, kH, padH, strideH, dilationH, ceil_mode);
155:   int64_t outW = pooling_output_shape(inW, kW, padW, strideW, dilationW, ceil_mode);
156:   TORCH_CHECK(outH > 0 && outW > 0,
157:               "Given input size: (",
158:               inC, "x", inH, "x", inW,
159:               "). Calculated output size: (",
160:               outC, "x", outH, "x", outW,
161:               "). Output size is too small.");
162:
163:   std::vector<int64_t> output_shape;
164:   if (ndim == 3) {
165:     // cudnn requires 4D input and output for 2D pooling, so we prepend a dummy dimension
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 166-176
```cpp
166:     // whose size represents the batch size (1)
167:     output_shape = {1, outC, outH, outW};
168:   } else {
169:     output_shape = {batch_size, outC, outH, outW};
170:   }
171:   auto qy = at::_empty_affine_quantized(
172:       output_shape,
173:       at::device(at::kCUDA).dtype(at::ScalarType::QInt8),
174:       input.q_scale(),
175:       input.q_zero_point(),
176:       (ndim == 4 ? MemoryFormat::ChannelsLast : MemoryFormat::Contiguous));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 178-190
```cpp
178:   cudnnHandle_t handle = getCudnnHandle();
179:   cudnnPoolingDescriptor_t poolingDesc = nullptr;
180:   AT_CUDNN_CHECK_WITH_SHAPES(cudnnCreatePoolingDescriptor(&poolingDesc));
181:   AT_CUDNN_CHECK_WITH_SHAPES(cudnnSetPooling2dDescriptor(
182:       poolingDesc,
183:       CUDNN_POOLING_MAX_DETERMINISTIC,
184:       CUDNN_NOT_PROPAGATE_NAN,
185:       kernel_size[0], // kernel height
186:       kernel_size[1], // kernel width
187:       padding[0], // vertical padding
188:       padding[1], // horizontal padding
189:       stride[0], // vertical stride
190:       stride[1])); // horizontal stride
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 192-206
```cpp
192:   float one{1};
193:   float zero{0.0};
194:   TensorDescriptor xDesc;
195:   at::MemoryFormat memory_format = (ndim == 4 ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::Contiguous);
196:   xDesc.set(input, memory_format);
197:   TensorDescriptor yDesc;
198:   yDesc.set(qy, memory_format);
199:   cudnnPoolingForward(handle,
200:                       poolingDesc,
201:                       &one,
202:                       xDesc.desc(),
203:                       input.data_ptr<int8_t>(),
204:                       &zero,
205:                       yDesc.desc(),
206:                       qy.data_ptr<int8_t>());
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 208-217
```cpp
208:   // recall we casted our input and output to 4D if qx was 3D, so we recast it back to 3D prior to returning
209:   return (ndim == 3 ? qy.view(std::vector<int64_t>(output_shape.begin() + 1, output_shape.end())) : qy);
210: #else // AT_CUDNN_ENABLED()
211:   TORCH_CHECK(false, "at::native::quantized_max_pool2d_cudnn: ATen not compiled with cuDNN support");
212:   return Tensor{}; // never reached, placates the compiler
213: #endif // AT_CUDNN_ENABLED()
214: #else // USE_CUDA
215:   TORCH_CHECK(false, "at::native::quantized_max_pool2d_cudnn: ATen not compiled with USE_CUDA support");
216: #endif
217: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 219-229
```cpp
219: // Keep the registry in the anonymous namespace.
220: namespace {
221: template <uint32_t kSpatialDim>
222: class QMaxPool_arr_args final {
223:  public:
224:   static Tensor run(
225:       const Tensor& qx,
226:       std::vector<int64_t> kernel_size,
227:       std::vector<int64_t> stride,
228:       std::vector<int64_t> padding,
229:       std::vector<int64_t> dilation,
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `QMaxPool_arr_args`, which contributes directly to this file's operator logic. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `QMaxPool_arr_args`，它们直接构成本文件的算子逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 230-241
```cpp
230:       bool ceil_mode) {
231:     static_assert(kSpatialDim == 2, "quantized max pool is only valid for 2D");
232:     return quantized_max_pool2d_cudnn(qx, kernel_size, stride, padding,
233:                                     dilation, ceil_mode);
234:   }
235: };
236:
237: TORCH_LIBRARY_IMPL(quantized, QuantizedCUDA, m) {
238:   m.impl(TORCH_SELECTIVE_NAME("quantized::max_pool2d"), TORCH_FN(QMaxPool_arr_args<2>::run));
239: }
240:
241: } // namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 242-242
```cpp
242: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- TensorIterator traversal / TensorIterator 遍历
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/cuda/CUDAConfig.h`, `ATen/cuda/Exceptions.h`, `ATen/cudnn/Descriptors.h`, `ATen/cudnn/Handle.h`, `ATen/cudnn/Types.h`, `ATen/ATen.h`, `ATen/native/Pool.h`, `ATen/native/TensorIterator.h`
- c10 headers / c10 头文件: `c10/util/Exception.h`, `c10/core/QScheme.h`, `c10/core/ScalarType.h`, `c10/util/ArrayRef.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `vector`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Scalar`, `ScalarType`, `qnnpack`
