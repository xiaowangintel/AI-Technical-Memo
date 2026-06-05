# ts_eager_fallback.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/ts_backend/ts_eager_fallback.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Defines lazy backend abstractions that isolate device-specific behavior.
  - CN: 定义 Lazy 后端抽象，用于隔离设备相关行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-41
```cpp
 1 | #include <torch/csrc/lazy/ts_backend/ts_eager_fallback.h>
 2 | 
 3 | #include <ATen/FunctionalTensorWrapper.h>
 4 | #include <ATen/Functions.h>
 5 | #include <ATen/core/boxing/KernelFunction.h>
 6 | #include <ATen/native/CPUFallback.h>
 7 | #include <torch/csrc/lazy/backend/backend_interface.h>
 8 | #include <torch/csrc/lazy/core/config.h>
 9 | #include <torch/csrc/lazy/core/metrics.h>
10 | #include <torch/csrc/lazy/core/tensor.h>
11 | #include <torch/library.h>
12 | #include <sstream>
13 | #include <unordered_map>
14 | 
15 | namespace torch::lazy {
16 | namespace {
17 | 
18 | std::vector<at::Tensor> _to_eager(
19 |     at::TensorList tensors,
20 |     c10::DeviceType device_type) {
21 |   switch (device_type) {
22 |     case at::kCPU: {
23 |       return at::_to_cpu(tensors);
24 |     }
25 |     default: {
26 |       std::vector<at::Tensor> eager_tensors;
27 |       for (const auto& t : tensors) {
28 |         c10::TensorOptions options = t.options().device(device_type);
29 |         at::Tensor eager_tensor = t.to(
30 |             options,
31 |             /*non_blocking*/ false,
32 |             /*copy*/ false);
33 |         eager_tensors.push_back(eager_tensor);
34 |       }
35 |       return eager_tensors;
36 |     }
37 |   }
38 | }
39 | 
40 | // convenience helper for converting tensors to cpu
41 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/ts_backend/ts_eager_fallback.h>`, `<ATen/FunctionalTensorWrapper.h>`, `<ATen/Functions.h>`, `<ATen/core/boxing/KernelFunction.h>` and system or third-party headers such as `<sstream>`, `<unordered_map>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Implements routines such as `_to_eager` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/ts_backend/ts_eager_fallback.h>`、`<ATen/FunctionalTensorWrapper.h>`、`<ATen/Functions.h>`、`<ATen/core/boxing/KernelFunction.h>`以及系统或第三方头文件，例如 `<sstream>`、`<unordered_map>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `_to_eager` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 42-89
```cpp
42 | std::vector<at::Tensor> to_eager(
43 |     const at::TensorList& tensors,
44 |     c10::DeviceType device_type) {
45 |   // We can't just call _to_eager() on the entire list of Tensors because it
46 |   // will break on undefined tensors. Separate out undefined tensors first.
47 |   std::vector<at::Tensor> eager_tensors(tensors.size());
48 |   std::vector<at::Tensor> valid_tensors;
49 |   std::vector<bool> to_translate(tensors.size());
50 |   for (size_t i = 0; i < tensors.size(); ++i) {
51 |     const at::Tensor& tensor = tensors[i];
52 |     // Explicitly handling undefined tensors here instead of letting `_to_eager`
53 |     // handle it. Otherwise, we'd need to require all backends with their own
54 |     // implementation of _to_eager to properly handle undefined tensors.
55 |     if (tensor.defined()) {
56 |       to_translate[i] = true;
57 |       valid_tensors.push_back(tensor);
58 |     } else {
59 |       eager_tensors[i] = tensor;
60 |     }
61 |   }
62 |   auto eager_valid_tensors = _to_eager(valid_tensors, device_type);
63 |   for (size_t i = 0, defined_pos = 0; i < tensors.size(); ++i) {
64 |     if (to_translate[i]) {
65 |       eager_tensors[i] = std::move(eager_valid_tensors[defined_pos++]);
66 |     }
67 |   }
68 |   return eager_tensors;
69 | }
70 | 
71 | std::vector<std::optional<at::Tensor>> to_eager(
72 |     const std::vector<std::optional<at::Tensor>>& tensors,
73 |     c10::DeviceType device_type) {
74 |   // We can't just call _to_eager() on the entire list of Tensors because it
75 |   // will break on undefined tensors. Separate out undefined tensors first.
76 |   std::vector<std::optional<at::Tensor>> eager_tensors(tensors.size());
77 |   std::vector<at::Tensor> valid_tensors;
78 |   std::vector<bool> to_translate(tensors.size());
79 |   for (size_t i = 0; i < tensors.size(); ++i) {
80 |     const std::optional<at::Tensor>& tensor = tensors[i];
81 |     // Explicitly handling undefined tensors here instead of letting `_to_eager`
82 |     // handle it. Otherwise, we'd need to require all backends with their own
83 |     // implementation of _to_eager to properly handle undefined tensors.
84 |     if (tensor.has_value() && tensor->defined()) {
85 |       to_translate[i] = true;
86 |       valid_tensors.push_back(*tensor);
87 |     } else {
88 |       eager_tensors[i] = tensor;
89 |     }
```
- EN: Implements routines such as `to_eager`, `eager_tensors`, `to_translate` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `to_eager`、`eager_tensors`、`to_translate` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 90-144
```cpp
 90 |   }
 91 |   auto eager_valid_tensors = _to_eager(valid_tensors, device_type);
 92 |   for (size_t i = 0, defined_pos = 0; i < tensors.size(); ++i) {
 93 |     if (to_translate[i]) {
 94 |       eager_tensors[i] = std::move(eager_valid_tensors[defined_pos++]);
 95 |     }
 96 |   }
 97 |   return eager_tensors;
 98 | }
 99 | 
100 | c10::DispatchKey dispatch_key(c10::DeviceType device_type) {
101 |   switch (device_type) {
102 |     case at::kCPU: {
103 |       return c10::DispatchKey::CPU;
104 |     }
105 |     case at::kCUDA: {
106 |       return c10::DispatchKey::CUDA;
107 |     }
108 |     default: {
109 |       TORCH_CHECK(false, "Unsupported device type: ", device_type);
110 |     }
111 |   }
112 | }
113 | 
114 | std::optional<c10::Device> compute_target_device(
115 |     std::vector<at::Tensor>& t_args,
116 |     const std::vector<c10::List<at::Tensor>>& tlist_args,
117 |     const std::vector<c10::List<std::optional<at::Tensor>>>& opt_tlist_args) {
118 |   // Decide what device to move the output tensor(s) to.
119 |   // The current convention is that we use the first tensor arg to pick the
120 |   // device Barring that, we take the first tensor from a TensorList arg.
121 |   if (!t_args.empty()) {
122 |     return t_args[0].device();
123 |   } else {
124 |     // We need to loop through all of the (potentially multiple) TensorList
125 |     // arguments In case, e.g. the first one is empty but the second is not.
126 |     for (auto& tens_list : tlist_args) {
127 |       for (const auto i : c10::irange(tens_list.size())) {
128 |         return tens_list.get(i).device();
129 |       }
130 |     }
131 |     for (auto& tens_list : opt_tlist_args) {
132 |       for (const auto i : c10::irange(tens_list.size())) {
133 |         auto const& e = tens_list.get(i);
134 |         if (e.has_value()) {
135 |           return e->device();
136 |         }
137 |       }
138 |     }
139 |   }
140 |   return std::nullopt;
141 | }
142 | 
143 | } // namespace
144 | 
```
- EN: Implements routines such as `dispatch_key`, `compute_target_device` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `dispatch_key`、`compute_target_device` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 145-196
```cpp
145 | static std::unordered_map<std::string, ::torch::lazy::Counter*>
146 |     _eager_fallback_counters;
147 | 
148 | bool force_eager_fallback(c10::Symbol op) {
149 |   auto force_str = getLTCForceFallback();
150 |   if (!force_str.empty()) {
151 |     static auto force_sym = c10::Symbol::fromQualString(std::string(force_str));
152 |     if (op == force_sym) {
153 |       return true;
154 |     }
155 |   }
156 |   if (op == at::aten::nonzero) {
157 |     // When symbolic shape mode is not enabled, the nonzero shape function
158 |     // returns an incorrect result.
159 |     return !symbolicShapeEnabled();
160 |   }
161 | 
162 |   return false;
163 | }
164 | 
165 | void ltc_eager_fallback(
166 |     const c10::OperatorHandle& op,
167 |     torch::jit::Stack* stack) {
168 |   // TODO(whc) this FN_TRACK thing hasn't been used so far in LTC iirc but could
169 |   // land/re-enable it LTC_FN_TRACK(3);;
170 |   const auto name = c10::toString(op.operator_name());
171 | 
172 |   // Manually applying the TORCH_LAZY_COUNTER macro.
173 |   // We need to do it ourselves and explicitly keep a mapping of counters
174 |   // because this boxed fallback kernel is used by multiple operators,
175 |   // and the macro stamps out a static Counter object with a fixed name
176 |   // at the code location that it was called.
177 |   if (_eager_fallback_counters.find(name) == _eager_fallback_counters.end()) {
178 |     _eager_fallback_counters[name] = new ::torch::lazy::Counter(name);
179 |   }
180 |   _eager_fallback_counters[name]->AddValue(1);
181 | 
182 |   auto& args = op.schema().arguments();
183 |   auto arguments = torch::jit::last(stack, args.size());
184 | 
185 |   // Log each tensor argument.
186 |   for (const auto& ivalue : arguments) {
187 |     if (ivalue.isTensor()) {
188 |       VLOG(3) << ivalue.toTensor().toString();
189 |     }
190 |   }
191 | 
192 |   // Call the actual boxed CPU fallback.
193 |   ts_eager_fallback(
194 |       op, stack, torch::lazy::getBackend()->EagerFallbackDeviceType());
195 | }
196 | 
```
- EN: Implements routines such as `force_eager_fallback`, `ltc_eager_fallback` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `force_eager_fallback`、`ltc_eager_fallback` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 197-248
```cpp
197 | void register_ts_ltc_eager_fallback() {
198 |   static auto m = MAKE_TORCH_LIBRARY_IMPL(_, Lazy);
199 |   // Most backends use TORCH_LIBRARY_* macros which perform their dispatcher
200 |   // registrations at static library init time, but the lazy Torchscript backend
201 |   // does not since it is built in the main torch lib but not always used.
202 |   // In particular, if another external backend wants to register itself to the
203 |   // same key (Lazy), Torchscript backend must not be initialized.
204 |   m.fallback(torch::CppFunction::makeFromBoxedFunction<&ltc_eager_fallback>());
205 | }
206 | 
207 | void ts_eager_fallback(
208 |     const c10::OperatorHandle& op,
209 |     torch::jit::Stack* stack,
210 |     c10::DeviceType device_type) {
211 |   auto& schema_args = op.schema().arguments();
212 |   const auto num_arguments = schema_args.size();
213 |   auto arguments = torch::jit::last(stack, num_arguments);
214 |   const auto arguments_begin = stack->size() - num_arguments;
215 | 
216 |   std::vector<at::Tensor> tensor_args;
217 |   std::vector<size_t> tensor_args_indices;
218 | 
219 |   std::vector<c10::List<at::Tensor>> tensorlist_args;
220 |   std::vector<c10::List<std::optional<at::Tensor>>> opt_tensorlist_args;
221 | 
222 |   // Step 1: Convert all non-eager tensor inputs into eager tensors and put them
223 |   // on the stack at the correct indices.
224 |   for (size_t idx = 0; idx < arguments.size(); ++idx) {
225 |     const auto& ivalue = arguments[idx];
226 |     if (ivalue.isTensor()) {
227 |       tensor_args.push_back(ivalue.toTensor());
228 |       tensor_args_indices.push_back(idx);
229 |     } else if (ivalue.isTensorList()) {
230 |       // Note: we copy each TensorList argument to eager individually out of
231 |       // convenience, but XLA would benefit from materializing all tensor and
232 |       // TensorList args onto the CPU at the same time. We can improve this if
233 |       // we need better perf for XLA's CPU fallbacks.
234 |       auto eager_ivalue = c10::IValue(c10::List<at::Tensor>(
235 |           to_eager(ivalue.toTensorVector(), device_type)));
236 |       (*stack)[arguments_begin + idx] = std::move(eager_ivalue);
237 |       tensorlist_args.push_back(ivalue.toTensorList());
238 |     } else if (ivalue.isOptionalTensorList()) {
239 |       auto eager_ivalue = c10::IValue(c10::List<std::optional<at::Tensor>>(
240 |           to_eager(ivalue.toOptionalTensorVector(), device_type)));
241 |       (*stack)[arguments_begin + idx] = std::move(eager_ivalue);
242 |       opt_tensorlist_args.push_back(ivalue.toOptionalTensorList());
243 |     }
244 |   }
245 |   // XLA requires all of the tensor arguments to be gathered up and converted to
246 |   // CPU together.
247 |   auto eager_tensors = to_eager(tensor_args, device_type);
248 | 
```
- EN: Implements routines such as `register_ts_ltc_eager_fallback`, `ts_eager_fallback` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `register_ts_ltc_eager_fallback`、`ts_eager_fallback` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；使用移动语义来避免不必要的拷贝。

### Lines 249-290
```cpp
249 |   for (const auto i : c10::irange(tensor_args_indices.size())) {
250 |     auto idx = tensor_args_indices[i];
251 |     (*stack)[arguments_begin + idx] = c10::IValue(eager_tensors[i]);
252 |   }
253 | 
254 |   // Step 2: Call the underlying eager implementation of the operator
255 |   op.redispatchBoxed(c10::DispatchKeySet(dispatch_key(device_type)), stack);
256 | 
257 |   // Step 3: We need to take special care to handle mutable aliases properly:
258 |   // If any input tensors are mutable aliases, we need to directly copy the
259 |   // updated data on the eager tensors back to the original inputs.
260 |   for (const auto i : c10::irange(tensor_args_indices.size())) {
261 |     auto tensor_idx = tensor_args_indices[i];
262 |     const auto alias_info = schema_args[tensor_idx].alias_info();
263 |     if (alias_info != nullptr && alias_info->isWrite()) {
264 |       at::_copy_from_and_resize(eager_tensors[i], tensor_args[i]);
265 |     }
266 |   }
267 | 
268 |   // Step 4: Convert any eager output tensors back to the original input device.
269 |   // For mutable alias'd outputs, we also need to take special care
270 |   // to move the ORIGINAL input tensor back onto the stack, in place of
271 |   // the temporary eager output tensor that we created.
272 |   //
273 |   // Note [Eager Fallback Does Not Handle View Operators]
274 |   // Also note that we are incapable of handling immutable aliases properly.
275 |   // Why?
276 |   // Schemas with an immutable alias'd tensor outputs correspond to view
277 |   // operators. For example, the `view_as` schema from native_functions.yaml:
278 |   // `view_as(Tensor(a) self, Tensor other) -> Tensor(a)`
279 |   // We can't handle these ops properly, because view ops are supposed to return
280 |   // a NEW tensor that shares the SAME storage as the original tensor.
281 |   // However, the new tensor that we created cannot share the same storage,
282 |   // since it lives on the eager CPU / CUDA device and the original tensor lives
283 |   // on a different device. Because of that, we warn if someone attempts to call
284 |   // the eager fallback on a view operator (this is to maintain BC for view ops
285 |   // for XLA that fall back to CPU).
286 |   const auto& schema_returns = op.schema().returns();
287 |   const auto& num_returns = schema_returns.size();
288 |   auto returns = torch::jit::last(stack, num_returns);
289 |   const auto returns_begin = stack->size() - num_returns;
290 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 291-338
```cpp
291 |   for (const auto idx : c10::irange(returns.size())) {
292 |     if (returns[idx].isTensor()) {
293 |       const auto& return_tens = returns[idx].toTensor();
294 |       if (return_tens.defined()) {
295 |         const auto alias_info = schema_returns[idx].alias_info();
296 |         if (alias_info != nullptr && alias_info->isWrite()) {
297 |           // Case (1): mutable alias case. Move the input ivalue directly onto
298 |           // the stack in place of the existing eager output tensor.
299 |           bool found_alias = false;
300 |           // We could store some extra metadata on the function schema to avoid
301 |           // the loop here if we need to improve perf.
302 |           for (const auto i : c10::irange(tensor_args_indices.size())) {
303 |             auto input_tensor_idx = tensor_args_indices[i];
304 |             const auto& input_tensor = eager_tensors[i];
305 |             const auto input_alias_info =
306 |                 schema_args[input_tensor_idx].alias_info();
307 |             if (input_tensor.defined() && input_alias_info != nullptr &&
308 |                 *alias_info == *input_alias_info) {
309 |               // We've found the original input tensor that aliases with the
310 |               // current output. Wrap it in an IValue and put it directly on the
311 |               // stack.
312 |               (*stack)[returns_begin + idx] = c10::IValue(tensor_args[i]);
313 |               found_alias = true;
314 |               break;
315 |             }
316 |           }
317 |           TORCH_CHECK(
318 |               found_alias,
319 |               "The operator ",
320 |               op.schema().operator_name(),
321 |               " appears to have invalid alias information. ",
322 |               "Found a return tensor argument with a mismatched "
323 |               "mutable alias: ",
324 |               schema_returns[idx]);
325 |         } else {
326 |           std::optional<c10::Device> tgt_device = compute_target_device(
327 |               tensor_args, tensorlist_args, opt_tensorlist_args);
328 |           if (alias_info != nullptr && !alias_info->isWrite()) {
329 |             // immutable alias (view) case: Warn here, since we're copying and
330 |             // not creating a view.
331 |             // If this operator is needed, the backend should provide a kernel
332 |             // for it.
333 |             // See Note [Eager Fallback Does Not Handle View Operators]
334 |             std::stringstream dev_str;
335 |             if (tgt_device) {
336 |               dev_str << *tgt_device;
337 |             } else {
338 |               dev_str << "<none>";
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 339-371
```cpp
339 |             }
340 |             // We should never hit this for a view op,
341 |             // because LazyTensor should provide a lowering for the
342 |             // corresponding view_copy operator. The functionalization pass will
343 |             // take care of calling the view_copy operator instead of the view.
344 |             TORCH_CHECK(
345 |                 false,
346 |                 "The operator ",
347 |                 op.schema().operator_name(),
348 |                 " appears to be a view operator, ",
349 |                 "but it has no implementation for the backend \"",
350 |                 dev_str.str(),
351 |                 "\". View operators don't support ",
352 |                 "falling back to run on the eager, since the tensor's "
353 |                 "storage cannot be shared across devices.");
354 |           }
355 |           // Case (2): copy case. Copy the eager output tensor to the original
356 |           // device.
357 | 
358 |           // We technically  might not have a target device, e.g. if you call
359 |           // torch.cat() with an empty list In that case, we shouldn't have any
360 |           // tensors to schlep across devices anyway.
361 |           if (tgt_device) {
362 |             (*stack)[returns_begin + idx] =
363 |                 c10::IValue(returns[idx].toTensor().to(*tgt_device));
364 |           }
365 |         }
366 |       }
367 |     }
368 |   }
369 | }
370 | 
371 | } // namespace torch::lazy
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `_to_eager`, `to_eager`, `eager_tensors`, `to_translate`, `dispatch_key`, `compute_target_device`, `force_eager_fallback`, `ltc_eager_fallback`.
  - CN: `_to_eager`、`to_eager`、`eager_tensors`、`to_translate`、`dispatch_key`、`compute_target_device`、`force_eager_fallback`、`ltc_eager_fallback`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/ts_backend/ts_eager_fallback.h>`, `<ATen/FunctionalTensorWrapper.h>`, `<ATen/Functions.h>`, `<ATen/core/boxing/KernelFunction.h>`, `<ATen/native/CPUFallback.h>`, `<torch/csrc/lazy/backend/backend_interface.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/metrics.h>`, `<torch/csrc/lazy/core/tensor.h>`, `<torch/library.h>`
- External includes / 外部头文件: `<sstream>`, `<unordered_map>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
