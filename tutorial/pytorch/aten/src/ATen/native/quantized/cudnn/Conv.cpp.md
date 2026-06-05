# Conv.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cudnn/Conv.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU convolution helpers, packed-parameter handling, or output-shape logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 卷积辅助逻辑、打包参数处理或输出形状计算。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: #ifdef USE_CUDA
 2: #include <ATen/cuda/CUDAConfig.h>  // for the definition of AT_CUDNN_ENABLED
 3:
 4: #if AT_CUDNN_ENABLED()
 5:
 6: #include <c10/util/ArrayRef.h>
 7:
 8: #include <ATen/ATen.h>
 9: #include <ATen/cuda/Exceptions.h>
10: #include <ATen/cudnn/Handle.h>
11: #include <ATen/native/cudnn/ConvShared.h>
12: #include <ATen/native/quantized/cudnn/utils.h>
13: #include <ATen/native/quantized/library.h>
14: #include <ATen/native/quantized/ConvUtils.h>
15: #include <ATen/native/quantized/PackedParams.h>
16: #include <ATen/native/utils/ParamsHash.h>
17: #include <ATen/TensorUtils.h>
18: #include <c10/cuda/CUDACachingAllocator.h>
19: #include <cudnn_frontend.h>
20: #include <torch/library.h>
```
- EN: This range pulls in required headers, including `ATen/cuda/CUDAConfig.h`, `c10/util/ArrayRef.h`, `ATen/ATen.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/cuda/CUDAConfig.h`, `c10/util/ArrayRef.h`, `ATen/ATen.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 22-39
```cpp
22: #include <iostream>
23: #include <unordered_map>
24: #include <vector>
25:
26: // TODO: there is a table from input dtype and weight dtype to operator qdtype,
27: // we can derive the operator dtype based on input dtype
28: cudnn_frontend::ConvDesc_v8 getConvDescriptor(cudnnDataType_t dataType, c10::IntArrayRef padding, c10::IntArrayRef stride, c10::IntArrayRef dilation) {
29:   int64_t convDim = static_cast<int64_t>(stride.size());
30:   return cudnn_frontend::ConvDescBuilder()
31:     .setDataType(dataType)
32:     .setMathMode(CUDNN_CROSS_CORRELATION)
33:     .setNDims(convDim)
34:     .setStrides(convDim, stride.data())
35:     .setPrePadding(convDim, padding.data())
36:     .setPostPadding(convDim, padding.data())
37:     .setDilation(convDim, dilation.data())
38:     .build();
39: }
```
- EN: This range pulls in required headers, including `iostream`, `unordered_map`, `vector`. The main symbol in this range is `getConvDescriptor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段引入了所需头文件，例如 `iostream`, `unordered_map`, `vector`。 这一段的主要符号是 `getConvDescriptor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 41-66
```cpp
41: // FIXME: make this thread-safe by reusing the benchmark cache in Conv_v7.cpp
42: namespace {
43: struct CacheKey {
44:   at::native::ConvolutionParams params;
45:   uint8_t input_alignment;
46:   uint8_t weight_alignment;
47:   uint8_t output_alignment;
48:   // default to -1 when no bias
49:   int8_t bias_alignment;
50:   bool kReluFused;
51: };
52: std::unordered_map<CacheKey, cudnn_frontend::ExecutionPlan, at::native::ParamsHash<CacheKey>, at::native::ParamsEqual<CacheKey>> execution_plan_cache;
53: } // anonymous namespace
54: // TODO: we can use cudnn_frontend::ExecutionPlanCache when it supports caching
55: // multiple operators
56: // reference: https://github.com/NVIDIA/cudnn-frontend/blob/main/samples/legacy_samples/conv_sample.cpp#L295
57: //static cudnn_frontend::ExecutionPlanCache plan_cache("sample_cache");
58:
59: // the parameter quantized_output is a quantized tensor
60: template <int kSpatialDim>
61: template <bool kReluFused>
62: void PackedConvWeightCudnn<kSpatialDim>::apply_impl_helper(const at::Tensor& quantized_output, const at::Tensor& input, double output_scale) {
63:   auto act_scale = input.q_scale();
64:   auto weight_scale = maybe_padded_weight_.q_scale();
65:   auto requantize_multiplier = act_scale * weight_scale / output_scale;
66:   at::Tensor requantize_multiplier_tensor = cudnn_utils::getRequantMultiplierTensor(requantize_multiplier, kSpatialDim + 2);
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `apply_impl_helper`, `CacheKey`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `apply_impl_helper`, `CacheKey`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 68-86
```cpp
68:   std::optional<at::Tensor> bias_multiplier_tensor;
69:   std::optional<at::Tensor> broadcasted_bias;
70:   if (bias_.has_value()) {
71:     // the input bias is a 1-D tensor whose size is the same as the size of the second dimension of quantized_output.
72:     // we need to add trailing dimensions in order to properly broadcast bias, otherwise broadcast_to will fail.
73:     // the number of trailing dimensions is quantized_output.dim() - 2, so the new size of the broadcast_bias
74:     // becomes quantized_output.dim() - 2 + 1. nothing needs to be done for the leading dimensions
75:     std::vector<int64_t> new_size(quantized_output.dim() - 1, 1);
76:     new_size[0] = bias_.value().size(0);
77:     broadcasted_bias = bias_.value().reshape(new_size);
78:     broadcasted_bias.value() = broadcasted_bias.value().broadcast_to(quantized_output.sizes());
79:     broadcasted_bias.value() = broadcasted_bias.value().to(c10::MemoryFormat::ChannelsLast);
80:     bias_multiplier_tensor = at::empty(quantized_output.sizes(), at::device(at::kCUDA).dtype(at::kFloat), at::MemoryFormat::ChannelsLast);
81:     auto bias_multiplier = 1.0 / (act_scale * weight_scale);
82:     bias_multiplier_tensor.value().fill_(bias_multiplier);
83:   }
84:
85:   cudnnHandle_t handle = at::native::getCudnnHandle();
86:   CacheKey key{};
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 87-110
```cpp
 87:   // memset is needed here because there is implicit packing added for CacheKey, and this can result in uninitialized padded values that are
 88:   // used for hashing (see how at::native::ParamsHash is defined). without memset, we can potentially come across a situation where two
 89:   // CacheKey objects have the same user defined parameters, but
 90:   // different padded values, resulting in different hash outputs.
 91:   memset(&key, 0, sizeof(key));
 92:   bool deterministic{true};
 93:   bool allow_tf32{false};
 94:   auto padding_vec = padding_.vec();
 95:   auto stride_vec = stride_.vec();
 96:   auto dilation_vec = dilation_.vec();
 97:   setConvolutionParams(&key.params, input, maybe_padded_weight_, padding_vec, stride_vec, dilation_vec, groups_, deterministic, allow_tf32, input.suggest_memory_format());
 98:
 99:   // operator datatype needs to be int32 for int8 convolution, but we can
100:   // set the datatype for output tensor to int32 or fp32
101:   key.params.dataType = CUDNN_DATA_INT32;
102:   key.input_alignment = cudnn_utils::getAlignment(input);
103:   key.output_alignment = cudnn_utils::getAlignment(quantized_output);
104:   key.weight_alignment = cudnn_utils::getAlignment(maybe_padded_weight_);
105:   if (bias_.has_value()) {
106:     key.bias_alignment = static_cast<int8_t>(cudnn_utils::getAlignment(broadcasted_bias.value()));
107:   } else {
108:     key.bias_alignment = -1;
109:   }
110:   key.kReluFused = kReluFused;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 112-132
```cpp
112:   auto run = [&](const cudnn_frontend::ExecutionPlan& plan_desc) {
113:     auto workspace_size = plan_desc.getWorkspaceSize();
114:     auto workspace_ptr = c10::cuda::CUDACachingAllocator::get()->allocate(workspace_size);
115:     at::SmallVector<void *, 7> data_ptrs;
116:     at::SmallVector<int64_t, 7> uids;
117:     data_ptrs = {input.data_ptr<int8_t>(), maybe_padded_weight_.data_ptr<int8_t>(),
118:                  requantize_multiplier_tensor.data_ptr(), quantized_output.data_ptr<int8_t>()};
119:     uids = {'x', 'w', 's', 'r'};
120:     if (bias_.has_value()) {
121:       data_ptrs.insert(data_ptrs.end(), {broadcasted_bias.value().data_ptr(), bias_multiplier_tensor.value().data_ptr(),
122:                                          broadcasted_bias.value().data_ptr()});
123:       uids.insert(uids.end(), {'b', 'c', 'd'});
124:     }
125:     auto variantPack = cudnn_frontend::VariantPackBuilder()
126:       .setWorkspacePointer(workspace_size ? workspace_ptr.get() : nullptr)
127:       .setDataPointers(static_cast<int64_t>(uids.size()), data_ptrs.data())
128:       .setUids(static_cast<int64_t>(uids.size()), uids.data())
129:       .build();
130:     auto variant_pack_desc = variantPack.get_raw_desc();
131:     AT_CUDNN_CHECK(cudnnBackendExecute(handle, plan_desc.get_raw_desc(), variant_pack_desc));
132:   };
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 134-157
```cpp
134:   auto search = execution_plan_cache.find(key);
135:   if (search != execution_plan_cache.end()) {
136:     cudnn_frontend::ExecutionPlan plan_desc = search->second;
137:     run(plan_desc);
138:     return;
139:   }
140:   // conv_op computes act_fp32 * w_fp32 (matrix multiplication)
141:   // where act_fp32 and w_fp32 are the input and weight variables, resp.
142:   // output is a fp32 tensor
143:   auto conv_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_CONVOLUTION_FORWARD_DESCRIPTOR)
144:       .setxDesc(cudnn_utils::getTensorDescriptor(input.sizes(), input.strides(), CUDNN_DATA_INT8, 'x', key.input_alignment))
145:       // for virtual tensors, the alignment is not used, so we can just put an arbitrary value here, e.g., key.output_alignment
146:       .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_FLOAT, 'y', key.output_alignment, true))
147:       .setwDesc(cudnn_utils::getTensorDescriptor(maybe_padded_weight_.sizes(), maybe_padded_weight_.strides(), CUDNN_DATA_INT8, 'w', key.weight_alignment))
148:       .setcDesc(getConvDescriptor(key.params.dataType, padding_vec, stride_vec, dilation_vec))
149:       .build();
150:   // std::cout << "operator:" << conv_op.describe() << std::endl;
151:
152:   std::optional<cudnn_frontend::Operation> bias_mult_op;
153:   std::optional<cudnn_frontend::Operation> sum_conv_bias_op;
154:   if (bias_.has_value()) {
155:     // we can't directly assign bias_mult_op because operator= is deleted for cudnn_frontend::Operation;
156:     // alternatively, I think we can use std::unique_ptr and dynamically allocate these builder ops
157:     // but here, we chose to do it statically. std::optional<T>::emplace() enables this approach
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 159-181
```cpp
159:     // bias_mult_op computes bias_fp32 / (act_scale * w_scale) or bias_fp32 * (1 / (act_scale * w_scale))
160:     // where bias_multiplier = (1 / (act_scale * w_scale))
161:     // output is a fp32 tensor
162:     // we use inplace operation here where the output is assigned to the input
163:     bias_mult_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
164:       .setxDesc(cudnn_utils::getTensorDescriptor(broadcasted_bias.value(), 'b', cudnn_utils::getAlignment(broadcasted_bias.value())))
165:       .setbDesc(cudnn_utils::getTensorDescriptor(bias_multiplier_tensor.value(), 'c', cudnn_utils::getAlignment(bias_multiplier_tensor.value())))
166:       .setyDesc(cudnn_utils::getTensorDescriptor(broadcasted_bias.value(), 'd', cudnn_utils::getAlignment(broadcasted_bias.value())))
167:       .setpwDesc(cudnn_utils::getPointWiseMulDescriptor(at::native::getCudnnDataType(bias_multiplier_tensor.value())))
168:       .build());
169:
170:     // computes (act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)])
171:     // where the 1st and 2nd summands is output of conv_op and broadcasted_bias, resp.
172:     // output is a fp32 tensor
173:     // we use inplace operation here where the output is assigned to the input
174:     sum_conv_bias_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
175:       .setxDesc(conv_op.getOutputTensor())
176:       .setbDesc(cudnn_utils::getTensorDescriptor(broadcasted_bias.value(), 'd', cudnn_utils::getAlignment(broadcasted_bias.value())))
177:       // for virtual tensors, the alignment is not used, so we can just put an arbitrary value here, e.g., key.output_alignment
178:       .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_FLOAT, 'e', key.output_alignment, true))
179:       .setpwDesc(cudnn_utils::getPointWiseAddDescriptor(at::native::getCudnnDataType(broadcasted_bias.value())))
180:       .build());
181:   }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 183-207
```cpp
183:   // relu_op computes relu(act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]
184:   // or relu(act_int8 * w_int8) if bias is not present.
185:   // output is a fp32 tensor
186:   std::optional<cudnn_frontend::Operation> relu_op;
187:   std::shared_ptr<cudnn_frontend::OpaqueBackendPointer> tensor2requant_ptr = bias_.has_value() ? sum_conv_bias_op.value().getOutputTensor() : conv_op.getOutputTensor();
188:   if (kReluFused) {
189:     // we use inplace operation here where the output is assigned to the input
190:     relu_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
191:       .setxDesc(tensor2requant_ptr)
192:       // for virtual tensors, the alignment is not used, so we can just put an arbitrary value here, e.g., key.output_alignment
193:       .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_FLOAT, 'f', key.output_alignment, true))
194:       .setpwDesc(cudnn_utils::getPointWiseReluDescriptor(CUDNN_DATA_FLOAT))
195:       .build());
196:   }
197:
198:   // relu_op computes relu(act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]) / (out_scale / (act_scale * w_scale))
199:   // or relu(act_int8 * w_int8) / (out_scale / (act_scale * w_scale))) if bias is not present.
200:   // output is a fp32 tensor
201:   auto requant_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
202:     .setxDesc(kReluFused ? relu_op.value().getOutputTensor() : tensor2requant_ptr)
203:     .setbDesc(cudnn_utils::getTensorDescriptor(requantize_multiplier_tensor, 's', cudnn_utils::getAlignment(requantize_multiplier_tensor)))
204:     .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_INT8, 'r', key.output_alignment))
205:     .setpwDesc(cudnn_utils::getPointWiseMulDescriptor(at::native::getCudnnDataType(requantize_multiplier_tensor)))
206:     .build();
207:   // std::cout << "operator:" << requant_op.describe() << std::endl;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 209-232
```cpp
209:   std::vector<cudnn_frontend::Operation const *> ops{&conv_op};
210:   if (bias_.has_value()) {
211:     ops.emplace_back(&(bias_mult_op.value()));
212:     ops.emplace_back(&(sum_conv_bias_op.value()));
213:   }
214:   if (kReluFused) {
215:     ops.emplace_back(&(relu_op.value()));
216:   }
217:   ops.emplace_back(&requant_op);
218:
219:   auto opGraph = cudnn_frontend::OperationGraphBuilder()
220:       .setHandle(handle)
221:       .setOperationGraph(static_cast<int64_t>(ops.size()), ops.data())
222:       .build();
223:   // std::cout << "opGraph: " << opGraph.describe() << std::endl;
224:
225:   auto heuristics = cudnn_frontend::EngineHeuristicsBuilder()
226:       .setOperationGraph(opGraph)
227:       .setHeurMode(CUDNN_HEUR_MODE_INSTANT)
228:       .build();
229:   auto fallback = cudnn_frontend::EngineFallbackListBuilder()
230:                     .setOperationGraph(opGraph)
231:                     .setOperation(CUDNN_BACKEND_OPERATION_CONVOLUTION_FORWARD_DESCRIPTOR)
232:                     .build();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 234-254
```cpp
234:   auto& engine_configs = heuristics.getEngineConfig(heuristics.getEngineConfigCount());
235:   auto& fallback_list = fallback.getFallbackList();
236:
237:   cudnn_frontend::EngineConfigList filtered_configs;
238:   cudnn_utils::filterEngineConfigs(engine_configs, filtered_configs, deterministic, allow_tf32, at::kChar);
239:   cudnn_utils::filterEngineConfigs(fallback_list, filtered_configs, deterministic, allow_tf32, at::kChar);
240:
241:   for (auto &cfg : engine_configs) {
242:     try {
243:       auto plan = cudnn_frontend::ExecutionPlanBuilder()
244:         .setHandle(handle)
245:         .setEngineConfig(cfg)
246:         .build();
247:       run(plan);
248:       execution_plan_cache.emplace(key, plan);
249:       return;
250:     } catch (cudnn_frontend::cudnnException &e) {std::cout << "cudnn error:" << e.what() << '\n';} catch(c10::CuDNNError &e) { std::cout << "other error" << e.what() << '\n';}
251:   }
252:
253:   TORCH_CHECK(false, "Unable to find an engine to execute this computation in Quantized Conv2D Cudnn");
254: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 256-274
```cpp
256: //
257: // output Tensor will be a clampped int8 Tensor
258: // both act and weight will be int8 Tensor
259: /*
260: Numerics:
261: out_fp32 = conv_fp32(act_fp32, w_fp32, …)
262:                     = act_fp32 * w_fp32 + bias_fp32
263: act_int8 = act_fp32 / act_scale + act_zero_point
264: w_int8 = w_fp32 / w_scale + w_zero_point
265: out_int8 = out_fp32 / out_scale + out_zero_point
266: out_int8 = (act_fp32 * w_fp32 + [bias_fp32]) / out_scale + out_zero_point
267:               = (act_int8 - act_zero_point) * act_scale * (w_int8 - w_zero_point) * w_scale / out_scale + out_zero_point + [bias_fp32 / out_scale]
268:              = (act_int8 * w_int8 - act_int8 * w_zero_point - act_zero_point * w_int8 + act_zero_point * w_zero_point) * act_scale * w_scale / out_scale + out_zero_point + [bias_fp32 / out_scale]
269:              = (if both act and weight are symmetrically quantized, int8, then act_zero_point = w_zero_point = 0)
270:              = (act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]) * act_scale * w_scale / out_scale
271:              = (act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]) / (out_scale / (act_scale * w_scale))
272:              = requantize((act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]), out_scale / (act_scale * w_scale))
273: */
274: template <int kSpatialDim>
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 275-293
```cpp
275: template <bool kReluFused>
276: at::Tensor PackedConvWeightCudnn<kSpatialDim>::apply_impl(
277:     const at::Tensor& act,
278:     double output_scale,
279:     int64_t output_zero_point) {
280:   const auto batch_size = kSpatialDim == 2 ? act.size(0) : 1;
281:   const auto num_input_channels = act.size(kSpatialDim - 1);
282:   const auto H = act.size(kSpatialDim);
283:   const auto W = act.size(kSpatialDim + 1);
284:   const auto num_output_channels = maybe_padded_weight_.size(0); // output channels
285:   std::vector<int64_t> kernel_size = {maybe_padded_weight_.size(2), maybe_padded_weight_.size(3)};
286:   auto output_shape = at::native::quantized::MakeConvOutputShape<kSpatialDim>(batch_size, num_output_channels, {H, W},
287:   kernel_size, stride_, padding_, dilation_);
288:   at::Tensor quantized_output = at::_empty_affine_quantized(
289:       output_shape,
290:       at::device(at::kCUDA).dtype(at::ScalarType::QInt8),
291:       output_scale,
292:       output_zero_point,
293:       at::MemoryFormat::ChannelsLast);
```
- EN: The main symbol in this range is `apply_impl`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `apply_impl`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 295-313
```cpp
295:   // cudnn v8.4.0 expects conv2d's int8 activation tensor's input channels to be a multiple of 4. if it is not
296:   // we need to explicitly pad it to a multiple of 4 ourselves as cudnn does not currently support padding.
297:   // TODO: when and if cudnn enables padding in their operators, we can remove padding on our end;
298:   // currently, limit padding support to groups=1 (ungrouped conv)
299:   // TODO: implement this for groups > 1; should be straightforward since we're only padding a single dimension
300:   auto act_maybe_padded = act;
301:   if (num_input_channels % 4 != 0) {
302:     int8_t num_slices = 4 - num_input_channels % 4; // number of slices we need to pad
303:     act_maybe_padded = at::pad(act, {0, 0, 0, 0, 0, num_slices, 0, 0}, "constant", 0);
304:   }
305:   apply_impl_helper<kReluFused>(
306:       quantized_output, act_maybe_padded.to(c10::MemoryFormat::ChannelsLast), output_scale);
307:
308:   // need to return sliced tensor if output_channels was padded
309:   if (num_unpadded_output_channels_ != maybe_padded_weight_.size(0)) {
310:     return quantized_output.slice(1, 0, num_unpadded_output_channels_);
311:   }
312:   return quantized_output;
313: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 315-334
```cpp
315: template <int kSpatialDim>
316: at::Tensor PackedConvWeightCudnn<kSpatialDim>::apply(
317:     const at::Tensor& input,
318:     double output_scale,
319:     int64_t output_zero_point) {
320:   return apply_impl<false>(input, output_scale, output_zero_point);
321: }
322:
323: template <int kSpatialDim>
324: at::Tensor PackedConvWeightCudnn<kSpatialDim>::apply_relu(
325:     const at::Tensor& input,
326:     double output_scale,
327:     int64_t output_zero_point) {
328:   return apply_impl<true>(input, output_scale, output_zero_point);
329: }
330:
331: template at::Tensor PackedConvWeightCudnn<2>::apply(
332:     const at::Tensor& act,
333:     double output_scale,
334:     int64_t output_zero_point);
```
- EN: The main symbol in this range is `apply`, `apply_relu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply`, `apply_relu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 336-354
```cpp
336: template at::Tensor PackedConvWeightCudnn<2>::apply_relu(
337:     const at::Tensor& act,
338:     double output_scale,
339:     int64_t output_zero_point);
340:
341: namespace at:: native {
342: namespace {
343:
344: template <bool kReluFused>
345: class QConv1dInt8 final {
346:  public:
347:   static Tensor run(
348:       Tensor act,
349:       const c10::intrusive_ptr<ConvPackedParamsBase<2>>& packed_weight,
350:       double output_scale,
351:       int64_t output_zero_point) {
352:     at::Tensor output;
353:     // we currently use conv2d kernel for conv1d by making the input and weight tensors
354:     // 4D rather than 3D. we add a dummy width dimension of size 1
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `run`, `QConv1dInt8`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `run`, `QConv1dInt8`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 355-374
```cpp
355:     // N, C, L -> N, C, 1, L
356:     act = act.unsqueeze(-2);
357:     if (kReluFused) {
358:       output = packed_weight->apply_relu(act, output_scale, output_zero_point);
359:     } else {
360:       output = packed_weight->apply(act, output_scale, output_zero_point);
361:     }
362:     // N, C, 1, L -> N, C, L
363:     return output.squeeze_(-2);
364:   }
365: };
366:
367: template <int kSpatialDim, bool kReluFused>
368: class QConvInt8 final {
369:  public:
370:   static at::Tensor run(
371:       at::Tensor act,
372:       const c10::intrusive_ptr<ConvPackedParamsBase<kSpatialDim>>& packed_weight,
373:       double output_scale,
374:       int64_t output_zero_point) {
```
- EN: The main symbol in this range is `run`, `QConvInt8`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `run`, `QConvInt8`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 375-398
```cpp
375:     TORCH_CHECK(kSpatialDim == 1 || kSpatialDim == 2, "Error in quantized cudnn conv2d operator: "
376:                 "Expected kSpatialDim == 1 || kSpatialDim == 2; received kSpatialDim=", kSpatialDim);
377:     // TODO: check all zero_points are zero/all tensors are symmetrically quantized
378:     if (kReluFused) {
379:       return packed_weight->apply_relu(act, output_scale, output_zero_point);
380:     } else {
381:       return packed_weight->apply(act, output_scale, output_zero_point);
382:     }
383:   }
384: };
385:
386: TORCH_LIBRARY_IMPL(quantized, QuantizedCUDA, m) {
387:   // the cpu conv1d doesn't use the quantized::conv1d*.new variant for packed weights. instead it just uses
388:   // quantized::conv1d for packed weights (see quantized/library.cpp).
389:   // this is inconsistent with what has been done for conv2d where new variants use packed weights, and
390:   // old variant does not. we adopt this inconsistency for now to be consistent with QuantizedCPU's conv1d
391:   // and will eventually deprecate the old variants
392:   register_conv_params<2>();
393:   register_conv_params<3>();
394:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv1d"), QConv1dInt8<false>::run);
395:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv1d_relu"), QConv1dInt8<true>::run);
396:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d.new"), QConvInt8<2, false>::run);
397:   m.impl(TORCH_SELECTIVE_NAME("quantized::conv2d_relu.new"), QConvInt8<2, true>::run);
398: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 400-405
```cpp
400: } // anonymous namespace
401: } // namespace at::native
402:
403:
404: #endif  // AT_CUDNN_ENABLED
405: #endif  // USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/cuda/CUDAConfig.h`, `ATen/ATen.h`, `ATen/cuda/Exceptions.h`, `ATen/cudnn/Handle.h`, `ATen/native/cudnn/ConvShared.h`, `ATen/native/quantized/cudnn/utils.h`, `ATen/native/quantized/library.h`, `ATen/native/quantized/ConvUtils.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/utils/ParamsHash.h`
- c10 headers / c10 头文件: `c10/util/ArrayRef.h`, `c10/cuda/CUDACachingAllocator.h`
- Standard or third-party headers / 标准库或第三方头文件: `cudnn_frontend.h`, `torch/library.h`, `iostream`, `unordered_map`, `vector`
- Key helper symbols / 关键辅助符号: `SmallVector`, `Scalar`, `ScalarType`
