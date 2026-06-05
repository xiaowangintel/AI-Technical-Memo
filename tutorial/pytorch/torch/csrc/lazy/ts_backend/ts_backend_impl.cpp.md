# ts_backend_impl.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/ts_backend/ts_backend_impl.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Defines lazy backend abstractions that isolate device-specific behavior.
  - CN: 定义 Lazy 后端抽象，用于隔离设备相关行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40
```cpp
 1 | #include <torch/csrc/lazy/ts_backend/ts_backend_impl.h>
 2 | 
 3 | #include <ATen/Functions.h>
 4 | #include <torch/csrc/lazy/backend/backend_device.h>
 5 | #include <torch/csrc/lazy/core/lazy_graph_executor.h>
 6 | #include <torch/csrc/lazy/generated/LazyNativeFunctions.h>
 7 | #include <torch/csrc/lazy/ts_backend/config.h>
 8 | #include <torch/csrc/lazy/ts_backend/ir_builder.h>
 9 | #include <torch/csrc/lazy/ts_backend/ts_eager_fallback.h>
10 | #include <torch/csrc/lazy/ts_backend/ts_lowering_context.h>
11 | #include <memory>
12 | 
13 | namespace at {
14 | // This function is defined in the codegenerated RegisterDispatchKey.cpp file.
15 | // For the TorchScript backend, we have a special case where the registration
16 | // does not happen immediately (at static initialization time), so that if an
17 | // external backend is loaded, it has a chance to register itself, and
18 | // TorchScript only registers itself if explicitly initialized
19 | extern TORCH_API void RegisterTorchScriptLazyNativeFunctions();
20 | extern TORCH_API void RegisterTorchScriptAutogradLazyNativeFunctions();
21 | } // namespace at
22 | 
23 | namespace torch::lazy {
24 | 
25 | struct TSBackendDeviceType : public BackendDeviceType {
26 |   TSBackendDeviceType() = delete;
27 |   TSBackendDeviceType(c10::DeviceType deviceType)
28 |       : BackendDeviceType((int8_t)deviceType) {
29 |     TORCH_CHECK(deviceType == at::kCPU || deviceType == at::kCUDA);
30 |   }
31 | 
32 |   std::string toString() const override {
33 |     return c10::DeviceTypeName((c10::DeviceType)type);
34 |   }
35 | 
36 |   c10::DeviceType c10Type() const {
37 |     return (c10::DeviceType)type;
38 |   }
39 | };
40 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/ts_backend/ts_backend_impl.h>`, `<ATen/Functions.h>`, `<torch/csrc/lazy/backend/backend_device.h>`, `<torch/csrc/lazy/core/lazy_graph_executor.h>` and system or third-party headers such as `<memory>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`at`, `torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `TSBackendDeviceType` that structure the state handled by this file. Implements routines such as `RegisterTorchScriptLazyNativeFunctions`, `RegisterTorchScriptAutogradLazyNativeFunctions`, `toString`, `c10Type` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/ts_backend/ts_backend_impl.h>`、`<ATen/Functions.h>`、`<torch/csrc/lazy/backend/backend_device.h>`、`<torch/csrc/lazy/core/lazy_graph_executor.h>`以及系统或第三方头文件，例如 `<memory>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`at`、`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `TSBackendDeviceType` 等数据抽象，用来组织本文件处理的状态。 实现了 `RegisterTorchScriptLazyNativeFunctions`、`RegisterTorchScriptAutogradLazyNativeFunctions`、`toString`、`c10Type` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 41-80
```cpp
41 | class TSBackendImpl : public torch::lazy::BackendImplInterface {
42 |  public:
43 |   TSBackendImpl() {
44 |     // TODO(whc) unify how all our flags are set and parsed as envs
45 |     static bool env_use_cuda = c10::utils::has_env("LTC_TS_CUDA");
46 |     auto type =
47 |         (env_use_cuda || FLAGS_torch_lazy_ts_cuda) ? at::kCUDA : at::kCPU;
48 |     default_device_type_ = std::make_shared<TSBackendDeviceType>(type);
49 |   }
50 | 
51 |   const IrBuilder* GetIrBuilder() const override {
52 |     static const IrBuilder* builder = new TorchScriptIrBuilder();
53 |     return builder;
54 |   }
55 | 
56 |   std::string CreateMetricReport() const override {
57 |     return "TSBackendImpl: N/A";
58 |   }
59 | 
60 |   std::unique_ptr<torch::lazy::LoweringContext> CreateLoweringContext(
61 |       const std::string& name,
62 |       torch::lazy::BackendDevice device,
63 |       c10::ArrayRef<const torch::lazy::Node*> post_order,
64 |       torch::lazy::Util::EmissionMap emit_status) const override {
65 |     return std::make_unique<torch::lazy::TSLoweringContext>(
66 |         name, device, post_order, emit_status);
67 |   }
68 | 
69 |   std::unique_ptr<torch::lazy::LoweringContext> CreateLoweringContext(
70 |       const std::string& name,
71 |       torch::lazy::BackendDevice device) const override {
72 |     return std::make_unique<torch::lazy::TSLoweringContext>(name, device);
73 |   }
74 | 
75 |   std::vector<std::string> GetCompilationDevices(
76 |       const std::string& device,
77 |       c10::ArrayRef<std::string> devices) const override {
78 |     return std::vector<std::string>(devices.begin(), devices.end());
79 |   }
80 | 
```
- EN: Defines or extends data abstractions such as `TSBackendImpl` that structure the state handled by this file. Implements routines such as `TSBackendImpl`, `GetIrBuilder`, `CreateMetricReport`, `CreateLoweringContext`, `GetCompilationDevices` that expose the key API or control flow of this region. Reads environment switches (`LTC_TS_CUDA`) to tune runtime behavior. At the statement level, this block returns computed state or forwards values to callers.
- CN: 定义或扩展了 `TSBackendImpl` 等数据抽象，用来组织本文件处理的状态。 实现了 `TSBackendImpl`、`GetIrBuilder`、`CreateMetricReport`、`CreateLoweringContext`、`GetCompilationDevices` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`LTC_TS_CUDA`）来调整运行时行为。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 81-114
```cpp
 81 |   at::Tensor MakeTensorFromComputationData(
 82 |       const torch::lazy::BackendDataPtr data,
 83 |       std::optional<at::ScalarType> logical_scalar_type) const override {
 84 |     const auto ts_data = std::static_pointer_cast<TSData>(data);
 85 |     return ts_data->data();
 86 |   }
 87 | 
 88 |   torch::lazy::BackendDataPtr MakeComputationDataFromTensor(
 89 |       const at::Tensor& tensor,
 90 |       const torch::lazy::Shape& shape,
 91 |       const torch::lazy::BackendDevice& device) const override {
 92 |     at::TensorOptions options = tensor.options().device(
 93 |         default_device_type_->c10Type(), device.ordinal());
 94 |     if (tensor.device().type() == default_device_type_->c10Type() &&
 95 |         default_device_type_->c10Type() == at::kCUDA) {
 96 |       return std::make_shared<TSData>(
 97 |           tensor.to(options, /*non_blocking=*/true), shape, device);
 98 |     } else if (tensor.device().type() == at::kCPU && tensor.numel() == 1) {
 99 |       // calling .item() on singleton cpu tensor is fast, and using fill is a
100 |       // safe, async way to copy cpu to cuda for a single value
101 |       auto device_tensor = at::full(tensor.sizes(), tensor.item(), options);
102 |       return std::make_shared<TSData>(device_tensor, shape, device);
103 |     } else {
104 |       return std::make_shared<TSData>(
105 |           tensor.to(options, /*non_blocking=*/false), shape, device);
106 |     }
107 |   }
108 | 
109 |   torch::lazy::BackendDataPtr MakeComputationDataFromScalar(
110 |       const at::Scalar& scalar,
111 |       const torch::lazy::BackendDevice& device) const override {
112 |     return std::make_shared<TSData>(scalar, device);
113 |   }
114 | 
```
- EN: Implements routines such as `MakeTensorFromComputationData`, `MakeComputationDataFromTensor`, `MakeComputationDataFromScalar` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `MakeTensorFromComputationData`、`MakeComputationDataFromTensor`、`MakeComputationDataFromScalar` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 115-152
```cpp
115 |   torch::lazy::BackendDataPtr GetComputationDataFromNode(
116 |       const Node* node) const override {
117 |     auto* device_data_node = DeviceData::Cast(node);
118 |     if (!device_data_node) {
119 |       return nullptr;
120 |     }
121 |     return device_data_node->data();
122 |   }
123 | 
124 |   std::string GetComputationBackendText(
125 |       const torch::lazy::ComputationPtr computation) const override {
126 |     auto ts_computation =
127 |         static_cast<torch::lazy::TSComputation*>(computation.get());
128 |     return ts_computation->graph()->toString();
129 |   }
130 | 
131 |   //////////////computation client interfaces///////////////////////
132 | 
133 |  public:
134 |   torch::lazy::BackendDataPtr CreateDataPlaceholder(
135 |       const torch::lazy::BackendDevice& device,
136 |       const torch::lazy::Shape& shape) const override;
137 | 
138 |   std::vector<torch::lazy::ComputationPtr> Compile(
139 |       std::vector<torch::lazy::ComputationPtr> instances) const override;
140 | 
141 |   std::vector<torch::lazy::BackendDataPtr> ExecuteComputation(
142 |       torch::lazy::ComputationPtr computation,
143 |       c10::ArrayRef<torch::lazy::BackendDataPtr> arguments,
144 |       const torch::lazy::BackendDevice& device) const override;
145 | 
146 |   std::shared_ptr<torch::lazy::BackendDeviceType> GetDefaultDeviceType()
147 |       const override {
148 |     return default_device_type_;
149 |   }
150 | 
151 |   at::DeviceType EagerFallbackDeviceType() const override;
152 | 
```
- EN: Implements routines such as `GetComputationDataFromNode`, `GetComputationBackendText`, `CreateDataPlaceholder`, `Compile`, `ExecuteComputation` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `GetComputationDataFromNode`、`GetComputationBackendText`、`CreateDataPlaceholder`、`Compile`、`ExecuteComputation` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 153-187
```cpp
153 |   void SetDefaultDeviceType(int8_t type) override {
154 |     default_device_type_ = std::make_shared<TSBackendDeviceType>(
155 |         static_cast<c10::DeviceType>(type));
156 |   }
157 | 
158 |   int64_t GetDefaultDeviceOrdinal() const override {
159 |     return default_device_ordinal_;
160 |   }
161 | 
162 |   void SetDefaultDeviceOrdinal(int64_t ordinal) override {
163 |     default_device_ordinal_ = ordinal;
164 |   }
165 | 
166 |   std::vector<torch::lazy::BackendDevice> GetBackendDevices() const override;
167 | 
168 |   torch::lazy::BackendDevice GetBackendDevice(
169 |       c10::Device device) const override;
170 | 
171 |   void SetRngSeed(size_t seed) const override {
172 |     LOG(FATAL) << "Not implemented yet.";
173 |   }
174 | 
175 |   // std::map<std::string, Metric> GetMetrics() const override { return {}; }
176 | 
177 |   // MemoryInfo GetMemoryInfo(const std::string& device) override {
178 |   //   LOG(FATAL) << "Not implemented yet.";
179 |   // }
180 | 
181 |   void PrepareToExit() const override;
182 | 
183 |  private:
184 |   std::shared_ptr<TSBackendDeviceType> default_device_type_;
185 |   int64_t default_device_ordinal_{0};
186 | };
187 | 
```
- EN: Implements routines such as `SetDefaultDeviceType`, `GetDefaultDeviceOrdinal`, `SetDefaultDeviceOrdinal`, `GetBackendDevices`, `GetBackendDevice` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `SetDefaultDeviceType`、`GetDefaultDeviceOrdinal`、`SetDefaultDeviceOrdinal`、`GetBackendDevices`、`GetBackendDevice` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 188-219
```cpp
188 | torch::lazy::BackendDataPtr TSBackendImpl::CreateDataPlaceholder(
189 |     const torch::lazy::BackendDevice& device,
190 |     const torch::lazy::Shape& shape) const {
191 |   return std::make_shared<TSData>(shape, device);
192 | }
193 | 
194 | std::vector<torch::lazy::ComputationPtr> TSBackendImpl::Compile(
195 |     std::vector<torch::lazy::ComputationPtr> instances) const {
196 |   for (const auto& instance : instances) {
197 |     auto ts_computation =
198 |         static_cast<torch::lazy::TSComputation*>(instance.get());
199 |     if (!ts_computation->in_mark_step) {
200 |       LOG(WARNING) << "Compile outside of mark step";
201 |     }
202 |   }
203 |   return instances;
204 | }
205 | 
206 | std::vector<torch::lazy::BackendDataPtr> TSBackendImpl::ExecuteComputation(
207 |     torch::lazy::ComputationPtr computation,
208 |     c10::ArrayRef<torch::lazy::BackendDataPtr> arguments,
209 |     const torch::lazy::BackendDevice& device) const {
210 |   auto ts_computation =
211 |       std::dynamic_pointer_cast<torch::lazy::TSComputation>(computation);
212 |   TORCH_CHECK(ts_computation, "Computation isn't TSComputation");
213 |   torch::jit::GraphExecutor& graph_executor = ts_computation->graph_executor();
214 |   std::vector<torch::jit::IValue> stack;
215 |   for (const auto& argument : arguments) {
216 |     const auto ts_data = std::static_pointer_cast<TSData>(argument);
217 |     const auto& scalar = ts_data->scalar;
218 |     if (scalar.has_value()) {
219 |       stack.emplace_back(scalar.value());
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 220-259
```cpp
220 |     } else {
221 |       // TODO(whc) should this check be made more general? it's written somewhat
222 |       // oddly
223 |       TORCH_CHECK(
224 |           static_cast<c10::DeviceType>(default_device_type_->type) !=
225 |               at::kCUDA ||
226 |           ts_data->data().device().type() == at::kCUDA);
227 |       stack.emplace_back(ts_data->data());
228 |     }
229 |   }
230 |   graph_executor.run(stack);
231 |   std::vector<torch::lazy::BackendDataPtr> results;
232 |   for (torch::jit::IValue component : stack) {
233 |     at::Tensor result = component.toTensor();
234 |     at::IntArrayRef result_sizes = result.sizes();
235 |     torch::lazy::Shape shape(
236 |         result.scalar_type(),
237 |         std::vector<int64_t>(result_sizes.begin(), result_sizes.end()));
238 |     results.push_back(std::make_shared<TSData>(result, shape, device));
239 |   }
240 |   return results;
241 | }
242 | 
243 | std::vector<torch::lazy::BackendDevice> TSBackendImpl::GetBackendDevices()
244 |     const {
245 |   std::vector<torch::lazy::BackendDevice> devices;
246 |   // TODO(whc) figure out how to query available devices from pytorch
247 |   devices.emplace_back(GetBackendDevice(c10::Device(c10::kCPU, 0)));
248 |   devices.emplace_back(GetBackendDevice(c10::Device(c10::kCUDA, 0)));
249 |   return devices;
250 | }
251 | 
252 | torch::lazy::BackendDevice TSBackendImpl::GetBackendDevice(
253 |     c10::Device device) const {
254 |   // Note, we ignore the device type specified by the c10::Device since it is
255 |   // expected to be a virtual device (lazy::), but we need to change this when
256 |   // we support lazy as a mode
257 |   return torch::lazy::BackendDevice(GetDefaultDeviceType(), device.index());
258 | }
259 | 
```
- EN: Implements routines such as `shape` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `shape` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 260-283
```cpp
260 | void TSBackendImpl::PrepareToExit() const {}
261 | 
262 | c10::DeviceType TSBackendImpl::EagerFallbackDeviceType() const {
263 |   // For TS backend, hardware device _is_ eager device
264 |   return (c10::DeviceType)GetDefaultDeviceType()->type;
265 | }
266 | 
267 | torch::lazy::BackendImplInterface* GetTSBackendImpl() {
268 |   static TSBackendImpl* ts_backend_impl = new TSBackendImpl();
269 |   return ts_backend_impl;
270 | }
271 | 
272 | void InitTorchScriptBackend() {
273 |   at::RegisterTorchScriptLazyNativeFunctions();
274 |   at::RegisterTorchScriptAutogradLazyNativeFunctions();
275 |   register_ts_ltc_eager_fallback();
276 |   static std::unique_ptr<BackendRegistrar> s_registrar;
277 |   s_registrar = std::make_unique<BackendRegistrar>(GetTSBackendImpl());
278 | 
279 |   static LazyGraphExecutor* executor = new LazyGraphExecutor();
280 |   LazyGraphExecutor::Register(executor);
281 | }
282 | 
283 | } // namespace torch::lazy
```
- EN: Implements routines such as `GetTSBackendImpl`, `InitTorchScriptBackend` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `GetTSBackendImpl`、`InitTorchScriptBackend` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `TSBackendImpl`, `TSBackendDeviceType`.
  - CN: `TSBackendImpl`、`TSBackendDeviceType`。
- **Important routines / 重要例程**
  - EN: `RegisterTorchScriptLazyNativeFunctions`, `RegisterTorchScriptAutogradLazyNativeFunctions`, `toString`, `c10Type`, `TSBackendImpl`, `GetIrBuilder`, `CreateMetricReport`, `CreateLoweringContext`.
  - CN: `RegisterTorchScriptLazyNativeFunctions`、`RegisterTorchScriptAutogradLazyNativeFunctions`、`toString`、`c10Type`、`TSBackendImpl`、`GetIrBuilder`、`CreateMetricReport`、`CreateLoweringContext`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `at`, `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `at`、`torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/ts_backend/ts_backend_impl.h>`, `<ATen/Functions.h>`, `<torch/csrc/lazy/backend/backend_device.h>`, `<torch/csrc/lazy/core/lazy_graph_executor.h>`, `<torch/csrc/lazy/generated/LazyNativeFunctions.h>`, `<torch/csrc/lazy/ts_backend/config.h>`, `<torch/csrc/lazy/ts_backend/ir_builder.h>`, `<torch/csrc/lazy/ts_backend/ts_eager_fallback.h>`, `<torch/csrc/lazy/ts_backend/ts_lowering_context.h>`
- External includes / 外部头文件: `<memory>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
