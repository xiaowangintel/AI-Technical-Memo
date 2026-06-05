# init.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/python/init.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Bridges C++ components to Python-facing APIs or bindings.
  - CN: 把 C++ 组件桥接到面向 Python 的 API 或绑定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-43
```cpp
 1 | #include <torch/csrc/lazy/python/init.h>
 2 | 
 3 | #include <ATen/FunctionalTensorWrapper.h>
 4 | #include <c10/core/Device.h>
 5 | #include <torch/csrc/jit/python/pybind.h>
 6 | #include <torch/csrc/lazy/backend/backend_device.h>
 7 | #include <torch/csrc/lazy/backend/backend_interface.h>
 8 | #include <torch/csrc/lazy/core/config.h>
 9 | #include <torch/csrc/lazy/core/debug_util.h>
10 | #include <torch/csrc/lazy/core/internal_ops/ltc_ops.h>
11 | #include <torch/csrc/lazy/core/ir_dump_util.h>
12 | #include <torch/csrc/lazy/core/lazy_graph_executor.h>
13 | #include <torch/csrc/lazy/core/metrics.h>
14 | #include <torch/csrc/lazy/core/trie.h>
15 | #include <torch/csrc/lazy/python/python_util.h>
16 | #if !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
17 | #include <torch/csrc/lazy/ts_backend/ts_backend_impl.h>
18 | #include <torch/csrc/lazy/ts_backend/ts_lowering_context.h>
19 | #endif // FBCODE_CAFFE2 || OVRSOURCE
20 | #include <string>
21 | #include <utility>
22 | #include <vector>
23 | 
24 | namespace torch::lazy {
25 | 
26 | // TODO(whc) backend 'device' related APIs are not very clear, this code could
27 | // be simplified but it should probably be done together with
28 | // designing/refactoring the overall approach to get/set of default eager/lazy
29 | // device types
30 | static torch::lazy::BackendDevice GetDeviceOrCurrent(
31 |     const std::string& device_str) {
32 |   if (device_str.empty()) {
33 |     getBackend()->GetDefaultDeviceType();
34 |     return torch::lazy::BackendDevice();
35 |   }
36 |   return torch::lazy::atenDeviceToBackendDevice(c10::Device(device_str));
37 | }
38 | 
39 | static std::ptrdiff_t GetTensorId(const at::Tensor& tensor) {
40 |   torch::lazy::LazyTensorPtr lazy_tensor = torch::lazy::TryGetLtcTensor(tensor);
41 |   return lazy_tensor->GetUniqueId();
42 | }
43 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/python/init.h>`, `<ATen/FunctionalTensorWrapper.h>`, `<c10/core/Device.h>`, `<torch/csrc/jit/python/pybind.h>` and system or third-party headers such as `<string>`, `<utility>`, `<vector>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Implements routines such as `GetDeviceOrCurrent`, `GetTensorId` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/python/init.h>`、`<ATen/FunctionalTensorWrapper.h>`、`<c10/core/Device.h>`、`<torch/csrc/jit/python/pybind.h>`以及系统或第三方头文件，例如 `<string>`、`<utility>`、`<vector>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `GetDeviceOrCurrent`、`GetTensorId` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 44-98
```cpp
44 | static std::string GetTensorsDump(
45 |     const std::vector<at::Tensor>& tensors,
46 |     const std::function<std::string(c10::ArrayRef<const torch::lazy::Node*>)>&
47 |         converter) {
48 |   std::vector<const torch::lazy::Node*> nodes;
49 |   std::vector<torch::lazy::Value> values;
50 |   for (auto& tensor : tensors) {
51 |     auto inner = at::functionalization::impl::from_functional_tensor(tensor);
52 |     torch::lazy::LazyTensorPtr lazy_tensor =
53 |         torch::lazy::TryGetLtcTensor(inner);
54 |     values.push_back(lazy_tensor->GetIrValue());
55 |     nodes.push_back(values.back().node.get());
56 |   }
57 |   return converter(nodes);
58 | }
59 | 
60 | static std::vector<torch::lazy::LazyTensorPtr> GetLtcTensors(
61 |     const std::vector<at::Tensor>& tensors,
62 |     bool want_all) {
63 |   std::vector<torch::lazy::LazyTensorPtr> lazy_tensors;
64 |   lazy_tensors.reserve(tensors.size());
65 |   if (want_all) {
66 |     for (auto& tensor : tensors) {
67 |       lazy_tensors.push_back(torch::lazy::TryGetLtcTensor(tensor));
68 |     }
69 |   } else {
70 |     for (auto& tensor : tensors) {
71 |       auto lazy_tensor = torch::lazy::TryGetLtcTensor(tensor);
72 |       if (lazy_tensor) {
73 |         lazy_tensors.push_back(lazy_tensor);
74 |       }
75 |     }
76 |   }
77 |   return lazy_tensors;
78 | }
79 | 
80 | static std::string GetTensorsBackendGraph(
81 |     const std::vector<at::Tensor>& tensors) {
82 |   std::vector<torch::lazy::LazyTensorPtr> lazy_tensors =
83 |       GetLtcTensors(tensors, /*want_all=*/false);
84 |   return torch::lazy::LazyGraphExecutor::Get()->DumpBackendComputation(
85 |       lazy_tensors);
86 | }
87 | 
88 | static void SyncTensors(
89 |     const std::vector<at::Tensor>& tensors,
90 |     const std::vector<std::string>& devices,
91 |     bool wait,
92 |     bool sync_ltc_data) {
93 |   std::vector<torch::lazy::LazyTensorPtr> lazy_tensors =
94 |       GetLtcTensors(tensors, /*want_all=*/false);
95 |   torch::lazy::LazyGraphExecutor::Get()->SyncTensorsGraph(
96 |       &lazy_tensors, devices, wait, sync_ltc_data);
97 | }
98 | 
```
- EN: Implements routines such as `GetTensorsDump`, `converter`, `GetLtcTensors`, `GetTensorsBackendGraph`, `SyncTensors` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `GetTensorsDump`、`converter`、`GetLtcTensors`、`GetTensorsBackendGraph`、`SyncTensors` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 99-145
```cpp
 99 | void initLazyBindings(PyObject* module) {
100 |   auto m = py::handle(module).cast<py::module>();
101 |   auto lazy = m.def_submodule("_lazy");
102 |   auto lazy_ts_backend = m.def_submodule("_lazy_ts_backend");
103 | 
104 |   lazy.def(
105 |       "_mark_step",
106 |       // TODO(whc) this API should probably change from vector<std::string> to
107 |       // vector<c10::device> but in a separate PR
108 |       [](const std::string& device_str,
109 |          const std::vector<std::string>& devices,
110 |          bool wait) {
111 |         pybind11::gil_scoped_release no_gil;
112 |         auto backend_device = GetDeviceOrCurrent(device_str);
113 |         torch::lazy::LazyGraphExecutor::Get()->SyncLiveTensorsGraph(
114 |             &backend_device, devices, wait);
115 |         torch::lazy::LazyGraphExecutor::Get()->MarkStep(backend_device);
116 |       },
117 |       py::arg("device") = "",
118 |       py::arg("devices"),
119 |       py::arg("wait") = true);
120 |   lazy.def(
121 |       "_wait_device_ops",
122 |       [](const std::vector<std::string>& devices) {
123 |         pybind11::gil_scoped_release no_gil;
124 |         // TODO: Add support of non-empty devices.
125 |         if (!devices.empty()) {
126 |           LOG(ERROR) << "Non-empty devices are not supported.";
127 |         }
128 |         torch::lazy::LazyGraphExecutor::Get()->WaitDeviceOps({});
129 |       },
130 |       py::arg("devices"));
131 |   lazy.def("_reset_metrics", []() {
132 |     torch::lazy::MetricsArena::Get()->ResetCounters();
133 |     torch::lazy::MetricsArena::Get()->ResetMetrics();
134 |   });
135 |   lazy.def("_counter_names", []() { return torch::lazy::GetCounterNames(); });
136 |   lazy.def(
137 |       "_metrics_report", []() { return torch::lazy::CreateMetricReport(); });
138 |   lazy.def("_counter_value", [](const std::string& name) -> py::object {
139 |     torch::lazy::CounterData* data = torch::lazy::GetCounter(name);
140 |     return data != nullptr ? py::cast<int64_t>(data->Value()) : py::none();
141 |   });
142 |   lazy.def("_get_tensor_id", [](const at::Tensor& tensor) {
143 |     return GetTensorId(tensor);
144 |   });
145 | 
```
- EN: Implements routines such as `initLazyBindings`, `GetTensorId` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `initLazyBindings`、`GetTensorId` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 146-190
```cpp
146 |   lazy.def(
147 |       "_get_tensors_text",
148 |       [](const std::vector<at::Tensor>& tensors) -> std::string {
149 |         auto converter = [](c10::ArrayRef<const torch::lazy::Node*> nodes) {
150 |           return torch::lazy::DumpUtil::ToText(nodes);
151 |         };
152 |         return GetTensorsDump(tensors, converter);
153 |       });
154 |   lazy.def(
155 |       "_get_tensors_dot",
156 |       [](const std::vector<at::Tensor>& tensors) -> std::string {
157 |         auto converter = [](c10::ArrayRef<const torch::lazy::Node*> nodes) {
158 |           return torch::lazy::DumpUtil::ToDot(nodes);
159 |         };
160 |         return GetTensorsDump(tensors, converter);
161 |       });
162 |   lazy.def(
163 |       "_get_tensors_backend",
164 |       [](const std::vector<at::Tensor>& tensors) -> std::string {
165 |         return GetTensorsBackendGraph(tensors);
166 |       });
167 |   lazy.def("_get_graph_hash", [](const std::vector<at::Tensor>& tensors) {
168 |     std::vector<LazyTensorPtr> xtensors;
169 |     xtensors.reserve(tensors.size());
170 |     for (auto& tensor : tensors) {
171 |       xtensors.emplace_back(TryGetLtcTensor(tensor));
172 |     }
173 |     auto hash = LazyGraphExecutor::Get()->GetGraphHash(xtensors);
174 |     std::string bin((const char*)&hash, sizeof(hash));
175 |     return py::bytes(bin);
176 |   });
177 |   lazy.def(
178 |       "_sync_multi",
179 |       [](const std::vector<at::Tensor>& tensors,
180 |          const std::vector<std::string>& devices,
181 |          bool wait,
182 |          bool sync_ltc_data) {
183 |         pybind11::gil_scoped_release no_gil;
184 |         SyncTensors(tensors, devices, wait, sync_ltc_data);
185 |       },
186 |       py::arg("tensors"),
187 |       py::arg("devices"),
188 |       py::arg("wait") = true,
189 |       py::arg("sync_ltc_data") = true);
190 | 
```
- EN: Implements routines such as `GetTensorsDump`, `GetTensorsBackendGraph`, `bin` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `GetTensorsDump`、`GetTensorsBackendGraph`、`bin` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 191-243
```cpp
191 |   lazy.def("_get_force_fallback", []() {
192 |     return torch::lazy::getLTCForceFallback();
193 |   });
194 |   lazy.def("_set_force_fallback", [](std::string newval) {
195 |     torch::lazy::getLTCForceFallback() = std::move(newval);
196 |   });
197 |   lazy.def("_clear_ir_cache", []() { TrieCache::Get()->Clear(); });
198 |   lazy.def("_dump_ir_cache", [](const std::string& filename) {
199 |     TrieCache::Get()->DumpToDotFile(filename);
200 |   });
201 |   lazy.def("_set_reuse_ir", [](bool val) { FLAGS_torch_lazy_reuse_ir = val; });
202 |   lazy.def("_set_symbolic_shape_mode", [](bool val) {
203 |     FLAGS_ltc_enable_symbolic_shapes = val;
204 |   });
205 |   lazy.def("_get_symbolic_shape_mode", []() {
206 |     return FLAGS_ltc_enable_symbolic_shapes;
207 |   });
208 |   lazy.def("_get_default_device_type", []() {
209 |     return getBackend()->GetDefaultDeviceType()->toString();
210 |   });
211 | 
212 |   lazy_ts_backend.def("_init", []() {
213 | #if !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
214 |     torch::lazy::InitTorchScriptBackend();
215 | #else
216 |       TORCH_CHECK(false, "TorchScript backend not yet supported in FBCODE/OVRSOURCE builds");
217 | #endif // !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
218 |   });
219 | 
220 |   /*
221 |    * Return tensor ids and tensors for DeviceData nodes.
222 |    * TODO(shunting) revisit this API for XLA
223 |    */
224 |   lazy_ts_backend.def(
225 |       "_get_tensors_ts_device_data_node",
226 |       [](const std::vector<at::Tensor>& tensors)
227 |           -> std::pair<std::vector<int64_t>, std::vector<at::IValue>> {
228 | #if !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
229 |         std::vector<const Node*> roots;
230 |         for (auto& tensor : tensors) {
231 |           auto xtensor = TryGetLtcTensor(tensor);
232 |           roots.push_back(xtensor->GetIrValue().node.get());
233 |         }
234 |         auto post_order = Util::ComputePostOrder(roots);
235 |         std::vector<int64_t> tensor_ids;
236 |         std::vector<at::IValue> ivalues;
237 | 
238 |         std::unordered_set<BackendData::Handle> data_handles_;
239 |         for (auto nodeptr : post_order) {
240 |           if (nodeptr->op() == *torch::lazy::ltc_device_data) {
241 |             const auto backend_data =
242 |                 getBackend()->GetComputationDataFromNode(nodeptr);
243 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `getBackend` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `getBackend` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 244-293
```cpp
244 |             auto infoptr = backend_data->info();
245 |             auto deviceDataInfoPtr =
246 |                 (torch::lazy::LazyGraphExecutor::DeviceDataInfo*)infoptr;
247 |             auto* tsDataPtr = (torch::lazy::TSData*)backend_data.get();
248 | 
249 |             // dedup DeviceData by handle
250 |             auto handle = tsDataPtr->GetHandle();
251 |             if (!data_handles_.insert(handle).second) {
252 |               continue;
253 |             }
254 |             tensor_ids.push_back(deviceDataInfoPtr->tensor_id);
255 |             /*
256 |              * If the TSData contains a tensor, then the tensor id will uniquely
257 |              * identify the tensor. We use that tensor id to find the tensor in
258 |              * other places: e.g. in the python forward method parameters.
259 |              *
260 |              * If the TSData contains a scalar, the tensor id itself is not
261 |              * important. We reuse the scalar value in future calls.
262 |              */
263 |             if (tsDataPtr->HasValue()) {
264 |               ivalues.emplace_back(tsDataPtr->data());
265 |             } else {
266 |               TORCH_CHECK(tsDataPtr->scalar.has_value());
267 |               ivalues.emplace_back(tsDataPtr->scalar.value());
268 |             }
269 |           }
270 |         }
271 |         return std::make_pair(tensor_ids, ivalues);
272 | #else
273 |         TORCH_CHECK(
274 |             false, "TorchScript backend not yet supported in FBCODE builds");
275 | #endif // !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
276 |       });
277 |   // TODO(shunting) revisit this part for XLA
278 |   lazy_ts_backend.def(
279 |       "_run_cached_graph",
280 |       [](const std::string& hash_str,
281 |          const std::vector<at::IValue>& graph_inputs) {
282 |         std::vector<at::Tensor> result;
283 | #if !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
284 |         TORCH_CHECK(hash_str.size() == sizeof(hash_t));
285 |         hash_t hash = *(hash_t*)(hash_str.c_str());
286 |         auto cachedComputation =
287 |             LazyGraphExecutor::Get()->GetComputationCache()->Get(hash);
288 |         TORCH_CHECK(
289 |             cachedComputation,
290 |             "Failed to get computation by hash. Maybe the entry get kicked out of the LRU cache"); // TODO implement a fallback mechanism, or make sure those entries never get kicked out
291 |         auto computationPtr =
292 |             (torch::lazy::TSComputation*)cachedComputation->computation.get();
293 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；构建后续执行依赖的容器状态。

### Lines 294-336
```cpp
294 |         std::vector<torch::jit::IValue> stack;
295 |         stack.reserve(graph_inputs.size());
296 |         for (const auto& arg : graph_inputs) {
297 |           stack.emplace_back(arg);
298 |         }
299 |         computationPtr->graph_executor().run(stack);
300 |         result.reserve(stack.size());
301 |         for (torch::jit::IValue elem : stack) {
302 |           result.push_back(elem.toTensor());
303 |         }
304 |         return result;
305 | #else
306 |         TORCH_CHECK(
307 |             false, "TorchScript backend not yet supported in FBCODE builds");
308 | #endif // !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
309 |       });
310 |   lazy_ts_backend.def("_get_latest_computation_graph", []() -> std::string {
311 | #if !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
312 |     auto computation = LazyGraphExecutor::Get()
313 |                            ->GetComputationCache()
314 |                            ->GetLatest()
315 |                            ->computation;
316 |     auto ts_computation = dynamic_cast<TSComputation*>(computation.get());
317 |     TORCH_CHECK(ts_computation, "Found non-TSComputation in cache");
318 |     return ts_computation->graph()->toString();
319 | #else
320 |     TORCH_CHECK(
321 |         false, "TorchScript backend not yet supported in FBCODE builds");
322 | #endif // !(defined(FBCODE_CAFFE2) || defined(OVRSOURCE))
323 |   });
324 | 
325 |   // GetPythonFramesFunction() has not ever worked with
326 |   // torchdeploy/multipy possibly because  // codespell:ignore multipy
327 |   // GetPythonFrames resolves to external cpython rather than embedded cpython.
328 |   // So far this problem has only been observed internally, so we will just
329 |   // block it off there.
330 | 
331 |   // When libtorch_python is loaded, we register the python frame getter
332 |   // otherwise, debug util simply omits python frames
333 |   GetPythonFramesFunction() = GetPythonFrames;
334 | }
335 | 
336 | } // namespace torch::lazy
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `GetDeviceOrCurrent`, `GetTensorId`, `GetTensorsDump`, `converter`, `GetLtcTensors`, `GetTensorsBackendGraph`, `SyncTensors`, `initLazyBindings`.
  - CN: `GetDeviceOrCurrent`、`GetTensorId`、`GetTensorsDump`、`converter`、`GetLtcTensors`、`GetTensorsBackendGraph`、`SyncTensors`、`initLazyBindings`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/python/init.h>`, `<ATen/FunctionalTensorWrapper.h>`, `<c10/core/Device.h>`, `<torch/csrc/jit/python/pybind.h>`, `<torch/csrc/lazy/backend/backend_device.h>`, `<torch/csrc/lazy/backend/backend_interface.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/debug_util.h>`, `<torch/csrc/lazy/core/internal_ops/ltc_ops.h>`, `<torch/csrc/lazy/core/ir_dump_util.h>`, `<torch/csrc/lazy/core/lazy_graph_executor.h>`, `<torch/csrc/lazy/core/metrics.h>`
- External includes / 外部头文件: `<string>`, `<utility>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
