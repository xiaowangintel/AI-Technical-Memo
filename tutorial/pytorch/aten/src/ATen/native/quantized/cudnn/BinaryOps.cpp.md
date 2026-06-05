# BinaryOps.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cudnn/BinaryOps.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU binary/comparison operator kernels and type-specific branches in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 二元/比较算子 kernel 与类型特化分支。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
 1: #ifdef USE_CUDA
 2: #include <ATen/cuda/CUDAConfig.h>  // for the definition of AT_CUDNN_ENABLED
 3:
 4: #if AT_CUDNN_ENABLED()
 5:
 6: #include <ATen/core/TensorBase.h>
 7: #include <ATen/core/TensorBody.h>
 8: #include <ATen/cuda/Exceptions.h>
 9: #include <ATen/cudnn/Handle.h>
10: #include <ATen/native/quantized/cudnn/utils.h>
11: #include <ATen/native/utils/ParamsHash.h>
12: #include <ATen/TensorUtils.h>
13: #include <c10/core/MemoryFormat.h>
14: #include <c10/core/QScheme.h>
15: #include <c10/cuda/CUDAFunctions.h>
16: #include <c10/util/ArrayRef.h>
17: #include <torch/library.h>
```
- EN: This range pulls in required headers, including `ATen/cuda/CUDAConfig.h`, `ATen/core/TensorBase.h`, `ATen/core/TensorBody.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/cuda/CUDAConfig.h`, `ATen/core/TensorBase.h`, `ATen/core/TensorBody.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 19-29
```cpp
19: #ifndef AT_PER_OPERATOR_HEADERS
20: #include <ATen/Functions.h>
21: #else
22: #include <ATen/ops/empty.h>
23: #include <ATen/ops/_empty_affine_quantized.h>
24: #endif
25:
26: #include <unordered_map>
27:
28:
29: namespace at::native {
```
- EN: This range pulls in required headers, including `ATen/Functions.h`, `ATen/ops/empty.h`, `ATen/ops/_empty_affine_quantized.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/Functions.h`, `ATen/ops/empty.h`, `ATen/ops/_empty_affine_quantized.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 30-41
```cpp
30: namespace {
31: constexpr uint8_t max_num_input_dim = 5;
32: struct AddParams {
33:   c10::DeviceIndex device_id;
34:   int64_t input_a_size[max_num_input_dim];
35:   int64_t input_b_size[max_num_input_dim];
36:   uint8_t input_dim; // we currently assume both inputs are given as the same size (i.e., no broadcasting)
37:   at::MemoryFormat memory_format;
38:   bool deterministic;
39:   bool allow_tf32;
40: };
41: struct CacheKey {
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `AddParams`, `CacheKey`, which contributes directly to this file's operator logic.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `AddParams`, `CacheKey`，它们直接构成本文件的算子逻辑。

### Lines 42-53
```cpp
42:   AddParams params;
43:   uint8_t input_a_alignment;
44:   uint8_t input_b_alignment;
45:   uint8_t output_alignment;
46:   bool kReluFused;
47: };
48: void setAddParams(
49:     AddParams* params, const at::Tensor& input_a, const at::Tensor& input_b,
50:     bool deterministic, bool allow_tf32) {
51:   memset(params, 0, sizeof(AddParams));
52:   params->device_id = at::cuda::current_device();
53:   params->input_dim = input_a.dim();
```
- EN: The main symbol in this range is `setAddParams`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `setAddParams`，它们直接构成本文件的算子逻辑。

### Lines 54-65
```cpp
54:   params->memory_format = input_a.suggest_memory_format();
55:   for (int i = 0; i < params->input_dim; ++i) {
56:     params->input_a_size[i] = input_a.sizes()[i];
57:     params->input_b_size[i] = input_b.sizes()[i];
58:   }
59:   params->deterministic = deterministic;
60:   params->allow_tf32 = allow_tf32;
61: }
62: // FIXME: make this thread-safe by reusing the benchmark cache in Conv_v7.cpp
63: // we currently set the maximum number of input dimensions to 5
64: // this can be increased, if necessary
65: std::unordered_map<CacheKey, cudnn_frontend::ManagedOpaqueDescriptor, at::native::ParamsHash<CacheKey>, at::native::ParamsEqual<CacheKey>> execution_plan_cache;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 67-79
```cpp
67: // TODO: this is also in BinaryOps.cpp and some other cpp files in quantized/cpu/. I think we should
68: // move everything into a utilities file in quantized/ directory later.
69: inline void check_inputs(const Tensor& qa, const Tensor& qb) {
70:   TORCH_CHECK(
71:       qa.qscheme() == kPerTensorAffine,
72:       "Only per tensor quantization is supported in Add.");
73:   TORCH_CHECK(
74:       qa.qscheme() == qb.qscheme(),
75:       "Both inputs to Add must have the same quantization scheme.");
76:   TORCH_CHECK(
77:       qa.scalar_type() == qb.scalar_type(),
78:       "Add operands should have same data type.");
79: }
```
- EN: The main symbol in this range is `check_inputs`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `check_inputs`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 81-93
```cpp
81: // currently we only support int8 symmetric (zero_point = 0 for inputs and output) quantized add
82: // We implement relu ( (a_int8 + b_int8 * ( b_scale/a_scale) ) ) * ( a_scale / out_scale )
83: // which requires 4 cudnn ops (2 multiplication, 1 addition, and 1 relu ops)
84: // Multiplication ops: rhs_mult_op, requant_op
85: // Addition op: add_op
86: // Relu op: relu_op
87: template <bool kReluFused = false>
88: Tensor add(Tensor qa, Tensor qb, double output_scale, int64_t output_zero_point) {
89:   if (qa.numel() == 0) {
90:     return Tensor{};
91:   }
92:   // TODO: add shape checking when broadcasted add is supported. For now we assume the input tensors are the same shape
93:   TORCH_CHECK(qa.sizes() == qb.sizes(), "Quantized cudnn add currently expects both input tensors to be the same shape");
```
- EN: The main symbol in this range is `symmetric`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `symmetric`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 95-105
```cpp
 95:   check_inputs(qa, qb);
 96:
 97:   // cudnn expects tensors to be at least 3D. So we will prepend dummy dimensions if the input tensors are not at least 3D
 98:   auto orig_sizes = qa.sizes().vec();
 99:   if (qa.dim() < 3) {
100:     std::vector<int64_t> new_sizes(3, 1);
101:     // cudnn expects leading dimensions to be the dummy dimensions
102:     new_sizes.back() = qa.sizes().back();
103:     if (qa.dim() == 2) {
104:       new_sizes[1] = qa.size(0);
105:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 106-120
```cpp
106:     qa = qa.view(new_sizes);
107:     qb = qb.view(new_sizes);
108:   } else if (qa.dim() == 4) {
109:     qa = qa.contiguous(c10::MemoryFormat::ChannelsLast);
110:     qb = qb.contiguous(c10::MemoryFormat::ChannelsLast);
111:   }
112:
113:   auto memory_format = qa.dim() == 4 ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::Contiguous;
114:   at::Tensor add_output = at::empty(qa.sizes(), at::device(at::kCUDA).dtype(at::kFloat), memory_format);
115:   at::Tensor quantized_output = at::_empty_affine_quantized(qa.sizes(), at::device(at::kCUDA).dtype(at::ScalarType::QInt8),
116:                                                             output_scale, output_zero_point, memory_format);
117:   double requantize_multiplier = qa.q_scale() / output_scale;
118:   at::Tensor requantize_multiplier_tensor = cudnn_utils::getRequantMultiplierTensor(requantize_multiplier, quantized_output.dim());
119:   at::Tensor rhs_multiplier_tensor = at::empty(quantized_output.sizes(), at::device(at::kCUDA).dtype(at::kFloat), memory_format);
120:   rhs_multiplier_tensor.fill_(qb.q_scale() / qa.q_scale());
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 122-135
```cpp
122:   cudnnHandle_t handle = at::native::getCudnnHandle();
123:   CacheKey key{};
124:   // memset is needed here because there is implicit packing added for CacheKey, and this can result in uninitialized padded values that are
125:   // used for hashing (see how at::native::ParamsHash is defined). Without memset, we can potentially come across a situation where two
126:   // CacheKey objects have the same user defined parameters, but
127:   // different padded values, resulting in different hash outputs.
128:   memset(&key, 0, sizeof(key));
129:   bool deterministic{true};
130:   bool allow_tf32{false};
131:   setAddParams(&key.params, qa, qb, deterministic, allow_tf32);
132:   key.kReluFused = kReluFused;
133:   key.input_a_alignment = cudnn_utils::getAlignment(qa);
134:   key.input_b_alignment = cudnn_utils::getAlignment(qb);
135:   key.output_alignment = cudnn_utils::getAlignment(add_output);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 137-151
```cpp
137:   auto run = [&](const cudnn_frontend::ManagedOpaqueDescriptor& plan_desc) {
138:     auto workspace_size = 0;
139:     auto workspace = at::empty({workspace_size}, qa.options().dtype(at::kByte));
140:     std::vector<void *> data_ptrs;
141:     std::vector<int64_t> uids;
142:     data_ptrs.reserve(8);
143:     uids.reserve(8);
144:     data_ptrs = {qb.data_ptr<int8_t>(), rhs_multiplier_tensor.data_ptr(), add_output.data_ptr(),
145:                  qa.data_ptr<int8_t>(), add_output.data_ptr(), requantize_multiplier_tensor.data_ptr(),
146:                  quantized_output.data_ptr<int8_t>()};
147:     uids = {'b', 'm', 'c', 'a', 'p', 'r', 'q'};
148:     if constexpr (kReluFused) {
149:         data_ptrs.emplace_back(add_output.data_ptr()),
150:         uids.emplace_back('f');
151:     }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 153-167
```cpp
153:     auto variantPack = cudnn_frontend::VariantPackBuilder()
154:       .setWorkspacePointer(workspace.data_ptr())
155:       .setDataPointers(static_cast<int64_t>(uids.size()), data_ptrs.data())
156:       .setUids(static_cast<int64_t>(uids.size()), uids.data())
157:       .build();
158:     auto variant_pack_desc = variantPack.get_raw_desc();
159:     AT_CUDNN_CHECK(cudnnBackendExecute(handle, plan_desc->get_backend_descriptor(), variant_pack_desc));
160:   };
161:
162:   auto search = execution_plan_cache.find(key);
163:   if (search != execution_plan_cache.end()) {
164:     cudnn_frontend::ManagedOpaqueDescriptor plan_desc = search->second;
165:     run(plan_desc);
166:     return quantized_output.view(orig_sizes);
167:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 169-184
```cpp
169:   // computes qb_int8 * ( qb_scale/qa_scale )
170:   auto rhs_mult_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
171:       .setxDesc(cudnn_utils::getTensorDescriptor(qb.sizes(), qb.strides(), CUDNN_DATA_INT8, 'b', key.input_b_alignment))
172:       .setbDesc(cudnn_utils::getTensorDescriptor(rhs_multiplier_tensor, 'm', cudnn_utils::getAlignment(rhs_multiplier_tensor)))
173:       .setyDesc(cudnn_utils::getTensorDescriptor(add_output, 'c', key.output_alignment))
174:       .setpwDesc(cudnn_utils::getPointWiseMulDescriptor(at::native::getCudnnDataType(add_output)))
175:       .build();
176:
177:   // add_op computes (qa_int8 + qb_int8 * ( qb_scale/qa_scale ) )
178:   // add_output is a fp32 tensor for accumulation purposes
179:   auto add_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
180:       .setxDesc(rhs_mult_op.getOutputTensor())
181:       .setbDesc(cudnn_utils::getTensorDescriptor(qa.sizes(), qa.strides(), CUDNN_DATA_INT8, 'a', key.input_a_alignment))
182:       .setyDesc(cudnn_utils::getTensorDescriptor(add_output, 'p', key.output_alignment))
183:       .setpwDesc(cudnn_utils::getPointWiseAddDescriptor(at::native::getCudnnDataType(add_output)))
184:       .build();
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 186-197
```cpp
186:   // relu_op computes
187:   // relu( (qa_int8 + qb_int8 * ( qb_scale/qa_scale ) )  )
188:   // output is a fp32 tensor
189:   std::optional<cudnn_frontend::Operation> relu_op;
190:   if constexpr (kReluFused) {
191:     // we use inplace operation here where the output is assigned to the input
192:     relu_op.emplace(cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
193:       .setxDesc(add_op.getOutputTensor())
194:       .setyDesc(cudnn_utils::getTensorDescriptor(add_output, 'f', key.output_alignment))
195:       .setpwDesc(cudnn_utils::getPointWiseReluDescriptor(at::native::getCudnnDataType(add_output)))
196:       .build());
197:   }
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。

### Lines 199-212
```cpp
199:   // requant_op computes
200:   // (a_int8 + b_int8 * ( b_scale/a_scale) ) * a_scale / out_scale
201:   auto requant_op = cudnn_frontend::OperationBuilder(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
202:     .setxDesc(kReluFused ? relu_op.value().getOutputTensor() : add_op.getOutputTensor())
203:     .setbDesc(cudnn_utils::getTensorDescriptor(requantize_multiplier_tensor, 'r', cudnn_utils::getAlignment(requantize_multiplier_tensor)))
204:     .setyDesc(cudnn_utils::getTensorDescriptor(quantized_output.sizes(), quantized_output.strides(), CUDNN_DATA_INT8, 'q', cudnn_utils::getAlignment(quantized_output)))
205:     .setpwDesc(cudnn_utils::getPointWiseMulDescriptor(at::native::getCudnnDataType(requantize_multiplier_tensor)))
206:     .build();
207:
208:   std::vector<cudnn_frontend::Operation const *> ops{&rhs_mult_op, &add_op};
209:   if constexpr (kReluFused) {
210:     ops.emplace_back(&(relu_op.value()));
211:   }
212:   ops.emplace_back(&requant_op);
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 214-227
```cpp
214:   auto opGraph = cudnn_frontend::OperationGraphBuilder()
215:       .setHandle(handle)
216:       .setOperationGraph(static_cast<int64_t>(ops.size()), ops.data())
217:       .build();
218:   // std::cout << "opGraph: " << opGraph.describe() << std::endl;
219:
220:   auto heuristics = cudnn_frontend::EngineHeuristicsBuilder()
221:       .setOperationGraph(opGraph)
222:       .setHeurMode(CUDNN_HEUR_MODE_INSTANT)
223:       .build();
224:   auto fallback = cudnn_frontend::EngineFallbackListBuilder()
225:                     .setOperationGraph(opGraph)
226:                     .setOperation(CUDNN_BACKEND_OPERATION_POINTWISE_DESCRIPTOR)
227:                     .build();
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 229-239
```cpp
229:   auto& engine_configs = heuristics.getEngineConfig(heuristics.getEngineConfigCount());
230:   auto& fallback_list = fallback.getFallbackList();
231:
232:   cudnn_frontend::EngineConfigList filtered_configs;
233:   cudnn_utils::filterEngineConfigs(engine_configs, filtered_configs, deterministic, allow_tf32, at::kChar);
234:   cudnn_utils::filterEngineConfigs(fallback_list, filtered_configs, deterministic, allow_tf32, at::kChar);
235:   for (auto &cfg : engine_configs) {
236:     try {
237:       auto plan = cudnn_frontend::ExecutionPlanBuilder()
238:         .setHandle(handle)
239:         .setEngineConfig(cfg)
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 240-254
```cpp
240:         .build();
241:       auto plan_desc = plan.get_desc();
242:       run(plan_desc);
243:       execution_plan_cache[key] = plan_desc;
244:       return quantized_output.view(orig_sizes);
245:     } catch (cudnn_frontend::cudnnException &e) {std::cout << "cudnn error:" << e.what() << '\n';} catch(c10::CuDNNError &e) { std::cout << "other error" << e.what() << '\n';}
246:   }
247:
248:   TORCH_CHECK(false, "Unable to find an engine to execute this computation in Quantized Add Cudnn");
249: }
250:
251: TORCH_LIBRARY_IMPL(quantized, QuantizedCUDA, m) {
252:   m.impl(TORCH_SELECTIVE_NAME("quantized::add"), TORCH_FN(add</*ReLUFused=*/false>));
253:   m.impl(TORCH_SELECTIVE_NAME("quantized::add_relu"), TORCH_FN(add</*ReLUFused=*/true>));
254: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 256-260
```cpp
256: } // namespace
257: } // namespace at::native
258:
259: #endif  // AT_CUDNN_ENABLED
260: #endif  // USE_CUDA
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/cuda/CUDAConfig.h`, `ATen/core/TensorBase.h`, `ATen/core/TensorBody.h`, `ATen/cuda/Exceptions.h`, `ATen/cudnn/Handle.h`, `ATen/native/quantized/cudnn/utils.h`, `ATen/native/utils/ParamsHash.h`, `ATen/TensorUtils.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/core/MemoryFormat.h`, `c10/core/QScheme.h`, `c10/cuda/CUDAFunctions.h`, `c10/util/ArrayRef.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `unordered_map`
- Key helper symbols / 关键辅助符号: `TensorBase`, `Scalar`, `ScalarType`
