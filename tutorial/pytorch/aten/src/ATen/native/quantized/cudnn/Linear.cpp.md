# Linear.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cudnn/Linear.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
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
11: #include <ATen/cudnn/Types.h>
12: #include <ATen/native/quantized/cudnn/utils.h>
```
- EN: This range pulls in required headers, including `ATen/cuda/CUDAConfig.h`, `c10/util/ArrayRef.h`, `ATen/ATen.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/cuda/CUDAConfig.h`, `c10/util/ArrayRef.h`, `ATen/ATen.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 13-25
```cpp
13: #include <ATen/native/quantized/PackedParams.h>
14: #include <ATen/native/utils/ParamsHash.h>
15: #include <ATen/TensorUtils.h>
16: #include <c10/core/ScalarType.h>
17: #include <c10/cuda/CUDACachingAllocator.h>
18: #include <c10/cuda/CUDAFunctions.h>
19: #include <cudnn_frontend.h>
20: #include <torch/library.h>
21:
22: #include <iostream>
23: #include <unordered_map>
24:
25: int register_linear_params();
```
- EN: This range pulls in required headers, including `ATen/native/quantized/PackedParams.h`, `ATen/native/utils/ParamsHash.h`, `ATen/TensorUtils.h`. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/native/quantized/PackedParams.h`, `ATen/native/utils/ParamsHash.h`, `ATen/TensorUtils.h`。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 27-37
```cpp
27: // TODO: there is a table from input dtype and weight dtype to operator dtype,
28: // we can derive the operator dtype based on input dtype
29: cudnn_frontend::MatMulDesc_v8 getLinearDescriptor(cudnnDataType_t dataType) {
30:   return cudnn_frontend::MatMulDescBuilder()
31:     .setMathPrecision(dataType)
32:     .build();
33: }
34:
35: // FIXME: make this thread-safe by reusing the benchmark cache in Conv_v7.cpp
36: namespace {
37: // we currently set the maximum number of input dimensions to 5
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `getLinearDescriptor`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `getLinearDescriptor`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 38-49
```cpp
38: // this can be increased, if necessary
39: constexpr uint8_t max_num_input_dim = 5;
40: struct LinearParams {
41:   c10::DeviceIndex device_id;
42:   cudnnDataType_t dataType;
43:   int64_t input_size[max_num_input_dim];
44:   uint8_t input_dim;
45:   at::MemoryFormat memory_format;
46:   int64_t weight_size[2];
47:   bool deterministic;
48:   bool allow_tf32;
49: };
```
- EN: The main symbol in this range is `LinearParams`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `LinearParams`，它们直接构成本文件的算子逻辑。

### Lines 50-61
```cpp
50: struct CacheKey {
51:   LinearParams params;
52:   uint8_t input_alignment;
53:   uint8_t weight_alignment;
54:   uint8_t output_alignment;
55:   // default to -1 when no bias
56:   int8_t bias_alignment;
57:   bool kReluFused;
58: };
59: void setLinearParams(
60:     LinearParams* params, const at::Tensor& input, const at::Tensor& weight,
61:     bool deterministic, bool allow_tf32) {
```
- EN: The main symbol in this range is `setLinearParams`, `CacheKey`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `setLinearParams`, `CacheKey`，它们直接构成本文件的算子逻辑。

### Lines 62-73
```cpp
62:   // operator datatype needs to be int32 for int8 matmul, but we can
63:   // set the datatype for output tensor to int32 or fp32
64:   memset(params, 0, sizeof(LinearParams));
65:   params->device_id = at::cuda::current_device();
66:   params->dataType = CUDNN_DATA_INT32;
67:   params->input_dim = input.dim();
68:   params->memory_format = input.suggest_memory_format();
69:   for (int i = 0; i < params->input_dim; ++i) {
70:     params->input_size[i] = input.sizes()[i];
71:   }
72:   for (int i = 0; i < 2; ++i) {
73:     params->weight_size[i] = weight.sizes()[i];
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 74-85
```cpp
74:   }
75:   params->deterministic = deterministic;
76:   params->allow_tf32 = allow_tf32;
77: }
78: std::unordered_map<CacheKey, cudnn_frontend::ExecutionPlan, at::native::ParamsHash<CacheKey>, at::native::ParamsEqual<CacheKey>> execution_plan_cache;
79: }
80: // TODO: we can use cudnn_frontend::ExecutionPlanCache when it supports caching
81: // multiple operators
82: // reference: https://github.com/NVIDIA/cudnn-frontend/blob/main/samples/legacy_samples/conv_sample.cpp#L295
83: //static cudnn_frontend::ExecutionPlanCache plan_cache("sample_cache");
84:
85: // currently we only support int8 symmetric (zero_point = 0 for inputs and output) quantized linear op
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 86-97
```cpp
86: // We implement relu(act_int8 * transpose(w_int8) + [bias_fp32/(act_scale * w_scale] ) * ( act_scale * w_scale / out_scale )
87: // which requires 5 cudnn ops (1 matmul, 2 multiplication, 1 add, and 1 relu ops)
88: // matmul op: linear_op
89: // Multiplication ops: rhs_mult_op, requant_op
90: // Addition op: add_op
91: // Relu op: relu_op
92: template <bool kReluFused>
93: void PackedLinearWeightCudnn::apply_impl_helper(const at::Tensor& quantized_output, const at::Tensor& input, double output_scale) {
94:   if (quantized_output.numel() == 0) {
95:     return;
96:   }
97:   auto act_scale = input.q_scale();
```
- EN: The main symbol in this range is `relu`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `relu`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 98-114
```cpp
 98:   auto weight_scale = orig_weight.q_scale();
 99:   auto requantize_multiplier = act_scale * weight_scale / output_scale;
100:   at::Tensor requantize_multiplier_tensor = cudnn_utils::getRequantMultiplierTensor(requantize_multiplier, quantized_output.dim());
101:   std::optional<at::Tensor> bias_multiplier_tensor;
102:   std::optional<at::Tensor> broadcasted_bias;
103:   if (bias_.has_value()) {
104:     // the input bias is a 1-D tensor whose size is the same as the size of the last dimension of quantized_output
105:     // we need to add trailing dimensions in order to properly broadcast bias, otherwise broadcast_to will fail.
106:     // the number of trailing dimensions is quantized_output.dim() - 2. We also prepend a leading dimension for clarity
107:     std::vector<int64_t> new_size(quantized_output.dim(), 1);
108:     new_size.back() = bias_.value().size(0);
109:     broadcasted_bias = bias_.value().clone().reshape(new_size);
110:     broadcasted_bias.value() = broadcasted_bias.value().broadcast_to(quantized_output.sizes()).contiguous();
111:     bias_multiplier_tensor = at::empty(quantized_output.sizes(), at::device(at::kCUDA).dtype(at::kFloat));
112:     auto bias_multiplier = 1.0 / (act_scale * weight_scale);
113:     bias_multiplier_tensor.value().fill_(bias_multiplier);
114:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 116-125
```cpp
116:   cudnnHandle_t handle = at::native::getCudnnHandle();
117:   CacheKey key{};
118:   // memset is needed here because there is implicit packing added for CacheKey, and this can result in uninitialized padded values that are
119:   // used for hashing (see how at::native::ParamsHash is defined). without memset, we can potentially come across a situation where two
120:   // CacheKey objects have the same user defined parameters, but
121:   // different padded values, resulting in different hash outputs.
122:   memset(&key, 0, sizeof(key));
123:   bool deterministic{true};
124:   bool allow_tf32{false};
125:   setLinearParams(&key.params, input, orig_weight, deterministic, allow_tf32);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 127-137
```cpp
127:   key.input_alignment = cudnn_utils::getAlignment(input);
128:   key.output_alignment = cudnn_utils::getAlignment(quantized_output);
129:   key.weight_alignment = cudnn_utils::getAlignment(orig_weight);
130:   if (bias_.has_value()) {
131:     key.bias_alignment = static_cast<int8_t>(cudnn_utils::getAlignment(broadcasted_bias.value()));
132:   } else {
133:     key.bias_alignment = -1;
134:   }
135:   key.kReluFused = kReluFused;
136:   // the matmul operation is input * transpose(weight), so we will work with the transposed weight
137:   auto weight_transposed = transpose(orig_weight, 0, 1);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 138-149
```cpp
138:   // cudnn expects tensors to be at least 3D. weight_transposed is currently 2D. we will create a 3D view
139:   // by prepending a leading dummy dimension (cudnn expects leading dimensions to be the dummy dimensions)
140:   std::vector<int64_t> new_sizes(3, 1);
141:   new_sizes.back() = weight_transposed.size(1);
142:   new_sizes[1] = weight_transposed.size(0);
143:   weight_transposed = weight_transposed.view(new_sizes);
144:
145:   auto run = [&](const cudnn_frontend::ExecutionPlan& plan_desc) {
146:     auto workspace_size = plan_desc.getWorkspaceSize();
147:     auto workspace_ptr = c10::cuda::CUDACachingAllocator::get()->allocate(workspace_size);
148:     at::SmallVector<void *, 8> data_ptrs;
149:     at::SmallVector<int64_t, 8> uids;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 150-165
```cpp
150:     data_ptrs = {input.data_ptr<int8_t>(), weight_transposed.data_ptr<int8_t>(),
151:                  requantize_multiplier_tensor.data_ptr(), quantized_output.data_ptr<int8_t>()};
152:     uids = {'x', 'w', 's', 'r'};
153:     if (bias_.has_value()) {
154:       data_ptrs.insert(data_ptrs.end(), {broadcasted_bias.value().data_ptr(), bias_multiplier_tensor.value().data_ptr(),
155:                                          broadcasted_bias.value().data_ptr(), broadcasted_bias.value().data_ptr()});
156:       uids.insert(uids.end(), {'b', 'c', 'd', 'n'});
157:     }
158:     auto variantPack = cudnn_frontend::VariantPackBuilder()
159:       .setWorkspacePointer(workspace_size ? workspace_ptr.get() : nullptr)
160:       .setDataPointers(static_cast<int64_t>(uids.size()), data_ptrs.data())
161:       .setUids(static_cast<int64_t>(uids.size()), uids.data())
162:       .build();
163:     auto variant_pack_desc = variantPack.get_raw_desc();
164:     AT_CUDNN_CHECK(cudnnBackendExecute(handle, plan_desc.get_raw_desc(), variant_pack_desc));
165:   };
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 167-177
```cpp
167:   auto search = execution_plan_cache.find(key);
168:   if (search != execution_plan_cache.end()) {
169:     cudnn_frontend::ExecutionPlan plan_desc = search->second;
170:     run(plan_desc);
171:     return;
172:   }
173:
174:   // linear_op computes act_int8 * transpose(w_int8) (matrix multiplication)
175:   // where act_int8 and w_int8 are the input and weight variables, resp.
176:   // output is a fp32 tensor
177:   auto linear_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_MATMUL_DESCRIPTOR)
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 178-191
```cpp
178:       .setaMatDesc(cudnn_utils::getTensorDescriptor(input.sizes(), input.strides(), CUDNN_DATA_INT8, 'x', key.input_alignment))
179:       .setbMatDesc(cudnn_utils::getTensorDescriptor(weight_transposed.sizes(), weight_transposed.strides(), CUDNN_DATA_INT8, 'w', key.weight_alignment))
180:       // for virtual tensors, the alignment is not used, so we can just put an arbitrary value here, e.g., key.output_alignment
181:       .setcMatDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_FLOAT, 'y', key.output_alignment, true))
182:       .setmatmulDesc(getLinearDescriptor(key.params.dataType))
183:       .build();
184:   // std::cout << "operator:" << linear_op.describe() << std::endl;
185:
186:   std::optional<cudnn_frontend::Operation> bias_mult_op;
187:   std::optional<cudnn_frontend::Operation> sum_linear_bias_op;
188:   if (bias_.has_value()) {
189:     // we can't directly assign bias_mult_op because operator= is deleted for cudnn_frontend::Operation;
190:     // alternatively, I think we can use std::unique_ptr and dynamically allocate these builder ops
191:     // but here, we chose to do it statically. std::optional<T>::emplace() enables this approach
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 193-204
```cpp
193:     // bias_mult_op computes bias_fp32 / (act_scale * w_scale) or bias_fp32 * (1 / (act_scale * w_scale))
194:     // where bias_multiplier = (1 / (act_scale * w_scale))
195:     // output is a fp32 tensor
196:     // we use inplace operation here where the output is assigned to the input
197:     bias_mult_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
198:       .setxDesc(cudnn_utils::getTensorDescriptor(broadcasted_bias.value(), 'b', cudnn_utils::getAlignment(broadcasted_bias.value())))
199:       .setbDesc(cudnn_utils::getTensorDescriptor(bias_multiplier_tensor.value(), 'c', cudnn_utils::getAlignment(bias_multiplier_tensor.value())))
200:       // TODO: I think we should be able to make this a virtual tensor, but we would need cudnn to support
201:       // setbdesc(ManagedOpaqueDescriptor const &raw_tensor) first
202:       .setyDesc(cudnn_utils::getTensorDescriptor(broadcasted_bias.value(), 'd', cudnn_utils::getAlignment(broadcasted_bias.value())))
203:       .setpwDesc(cudnn_utils::getPointWiseMulDescriptor(at::native::getCudnnDataType(bias_multiplier_tensor.value())))
204:       .build());
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 206-220
```cpp
206:     // computes (act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)])
207:     // where the 1st and 2nd summands is output of linear op and broadcasted_bias, resp.
208:     // output is a fp32 tensor
209:     // we use inplace operation here where the output is assigned to the input
210:     sum_linear_bias_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
211:       .setxDesc(linear_op.getOutputTensor())
212:       // TODO: An additional entry for broadcasted_bias in the uid-data_ptr pairing
213:       // appears to be needed in the current version of cudnn (8.4.0). Without it, some
214:       // test cases are failing. NVIDIA is currently investigating this issue.
215:       // When this issue is fixed, we can change 'n' back to 'd' and remove the additional entry in uid and data_ptrs in variant pack above
216:       .setbDesc(cudnn_utils::getTensorDescriptor(broadcasted_bias.value(), 'n', cudnn_utils::getAlignment(broadcasted_bias.value())))
217:       .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_FLOAT, 'e', key.output_alignment, true))
218:       .setpwDesc(cudnn_utils::getPointWiseAddDescriptor(at::native::getCudnnDataType(broadcasted_bias.value())))
219:       .build());
220:   }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 222-235
```cpp
222:   // relu_op computes relu(act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]
223:   // or relu(act_int8 * w_int8) if bias is not present.
224:   // output is a fp32 tensor
225:   std::optional<cudnn_frontend::Operation> relu_op;
226:   std::shared_ptr<cudnn_frontend::OpaqueBackendPointer> tensor2requant_ptr = bias_.has_value() ? sum_linear_bias_op.value().getOutputTensor() : linear_op.getOutputTensor();
227:   if constexpr (kReluFused) {
228:     // we use inplace operation here where the output is assigned to the input
229:     relu_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
230:       .setxDesc(tensor2requant_ptr)
231:       // for virtual tensors, the alignment is not used, so we can just put an arbitrary value here, e.g., key.output_alignment
232:       .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_FLOAT, 'f', key.output_alignment, true))
233:       .setpwDesc(cudnn_utils::getPointWiseReluDescriptor(CUDNN_DATA_FLOAT))
234:       .build());
235:   }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 237-246
```cpp
237:   // requant_op computes relu(act_int8 * w_int8 + [bias_fp32/(act_scale * w_scale)]) / (out_scale / (act_scale * w_scale))
238:   // or relu(act_int8 * w_int8) / (out_scale / (act_scale * w_scale))) if bias is not present.
239:   // output is a fp32 tensor
240:   auto requant_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
241:     .setxDesc(kReluFused ? relu_op.value().getOutputTensor() : tensor2requant_ptr)
242:     .setbDesc(cudnn_utils::getTensorDescriptor(requantize_multiplier_tensor, 's', cudnn_utils::getAlignment(requantize_multiplier_tensor)))
243:     .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_INT8, 'r', key.output_alignment))
244:     .setpwDesc(cudnn_utils::getPointWiseMulDescriptor(at::native::getCudnnDataType(requantize_multiplier_tensor)))
245:     .build();
246:   // // std::cout << "operator:" << requant_op.describe() << std::endl;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 248-262
```cpp
248:   std::vector<cudnn_frontend::Operation const *> ops{&linear_op};
249:   if (bias_.has_value()) {
250:     ops.emplace_back(&(bias_mult_op.value()));
251:     ops.emplace_back(&(sum_linear_bias_op.value()));
252:   }
253:   if constexpr (kReluFused) {
254:     ops.emplace_back(&(relu_op.value()));
255:   }
256:   ops.emplace_back(&requant_op);
257:
258:   auto opGraph = cudnn_frontend::OperationGraphBuilder()
259:       .setHandle(handle)
260:       .setOperationGraph(static_cast<int64_t>(ops.size()), ops.data())
261:       .build();
262:   // std::cout << "opGraph: " << opGraph.describe() << std::endl;
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 264-274
```cpp
264:   auto heuristics = cudnn_frontend::EngineHeuristicsBuilder()
265:       .setOperationGraph(opGraph)
266:       .setHeurMode(CUDNN_HEUR_MODE_INSTANT)
267:       .build();
268:   auto fallback = cudnn_frontend::EngineFallbackListBuilder()
269:                     .setOperationGraph(opGraph)
270:                     .setOperation(CUDNN_BACKEND_OPERATION_MATMUL_DESCRIPTOR)
271:                     .build();
272:
273:   auto& engine_configs = heuristics.getEngineConfig(heuristics.getEngineConfigCount());
274:   auto& fallback_list = fallback.getFallbackList();
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 276-290
```cpp
276:   cudnn_frontend::EngineConfigList filtered_configs;
277:   cudnn_utils::filterEngineConfigs(engine_configs, filtered_configs, deterministic, allow_tf32, at::kChar);
278:   cudnn_utils::filterEngineConfigs(fallback_list, filtered_configs, deterministic, allow_tf32, at::kChar);
279:
280:   for (auto &cfg : engine_configs) {
281:     try {
282:       auto plan = cudnn_frontend::ExecutionPlanBuilder()
283:         .setHandle(handle)
284:         .setEngineConfig(cfg)
285:         .build();
286:       run(plan);
287:       execution_plan_cache.emplace(key, plan);
288:       return;
289:     } catch (cudnn_frontend::cudnnException &e) {std::cout << "cudnn error:" << e.what() << '\n';} catch(c10::CuDNNError &e) { std::cout << "other error" << e.what() << '\n';}
290:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 292-302
```cpp
292:   TORCH_CHECK(false, "Unable to find an engine to execute this computation Quantized Linear Cudnn");
293: }
294:
295: // output Tensor will be a clampped int8 Tensor
296: // both act and weight will be int8 Tensor
297: // Numerics are the same as conv (see aten/src/ATen/native/quantized/Conv.cpp):
298: template <bool kReluFused>
299: at::Tensor PackedLinearWeightCudnn::apply_impl(
300:     const at::Tensor& act,
301:     double output_scale,
302:     int64_t output_zero_point) {
```
- EN: The main symbol in this range is `conv`, `apply_impl`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `conv`, `apply_impl`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 303-314
```cpp
303:   std::vector<int64_t> original_output_shape{act.sizes().vec()}; // 2D
304:   original_output_shape.back() = orig_weight.size(0); // output channels
305:   // cudnn expects tensors to be at least 3D. we will prepend a dummy dimension for quantized_output
306:   std::vector<int64_t> output_shape(3, 1);
307:   output_shape[1] = original_output_shape[0];
308:   output_shape[2] = original_output_shape[1];
309:   at::Tensor quantized_output = at::_empty_affine_quantized(
310:       output_shape,
311:       at::device(at::kCUDA).dtype(at::ScalarType::QInt8),
312:       output_scale,
313:       output_zero_point);
314:   // cudnn expects tensors to be at least 3D. act is currently 2D. We will create a 3D view
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 315-329
```cpp
315:   std::vector<int64_t> new_sizes(3, 1);
316:   // cudnn expects leading dimensions to be the dummy dimensions
317:   new_sizes.back() = act.sizes().back();
318:   new_sizes[1] = act.size(0);
319:   apply_impl_helper<kReluFused>(
320:       quantized_output, act.view(new_sizes), output_scale);
321:   return quantized_output.view(original_output_shape);
322: }
323:
324: at::Tensor PackedLinearWeightCudnn::apply(
325:     at::Tensor input,
326:     double output_scale,
327:     int64_t output_zero_point) {
328:   return apply_impl<false>(input, output_scale, output_zero_point);
329: }
```
- EN: The main symbol in this range is `apply`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `apply`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 331-340
```cpp
331: at::Tensor PackedLinearWeightCudnn::apply_relu(
332:     at::Tensor input,
333:     double output_scale,
334:     int64_t output_zero_point) {
335:   return apply_impl<true>(input, output_scale, output_zero_point);
336: }
337:
338:
339: namespace at::native {
340: namespace {
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `apply_relu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `apply_relu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 342-357
```cpp
342: template <bool kReluFused>
343: class QLinearInt8 final {
344:  public:
345:   static at::Tensor run(
346:       at::Tensor act,
347:       const c10::intrusive_ptr<LinearPackedParamsBase>& packed_weight,
348:       double output_scale,
349:       int64_t output_zero_point) {
350:     // TODO: check all zero_points are zero/all tensors are symmetrically quantized
351:     if constexpr (kReluFused) {
352:       return packed_weight->apply_relu(std::move(act), output_scale, output_zero_point);
353:     } else {
354:       return packed_weight->apply(std::move(act), output_scale, output_zero_point);
355:     }
356:   }
357: };
```
- EN: The main symbol in this range is `run`, `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `run`, `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 359-369
```cpp
359: TORCH_LIBRARY_IMPL(quantized, QuantizedCUDA, m) {
360:   register_linear_params();
361:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear"), QLinearInt8<false>::run);
362:   m.impl(TORCH_SELECTIVE_NAME("quantized::linear_relu"), QLinearInt8<true>::run);
363: }
364:
365: } // namespace
366: } // namespace at::native
367:
368:
369: #endif  // AT_CUDNN_ENABLED
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 370-370
```cpp
370: #endif  // USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/cuda/CUDAConfig.h`, `ATen/ATen.h`, `ATen/cuda/Exceptions.h`, `ATen/cudnn/Handle.h`, `ATen/cudnn/Types.h`, `ATen/native/quantized/cudnn/utils.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/utils/ParamsHash.h`, `ATen/TensorUtils.h`
- c10 headers / c10 头文件: `c10/util/ArrayRef.h`, `c10/core/ScalarType.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/cuda/CUDAFunctions.h`
- Standard or third-party headers / 标准库或第三方头文件: `cudnn_frontend.h`, `torch/library.h`, `iostream`, `unordered_map`
- Key helper symbols / 关键辅助符号: `SmallVector`, `Scalar`, `ScalarType`
