# tensor.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/tensor.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50
```cpp
 1 | #include <torch/csrc/lazy/core/config.h>
 2 | #include <torch/csrc/lazy/core/tensor.h>
 3 | 
 4 | #include <c10/util/irange.h>
 5 | #include <torch/csrc/lazy/core/helpers.h>
 6 | #include <torch/csrc/lazy/core/ir_builder.h>
 7 | #include <torch/csrc/lazy/core/ir_dump_util.h>
 8 | #include <torch/csrc/lazy/core/lazy_graph_executor.h>
 9 | #include <torch/csrc/lazy/core/metrics.h>
10 | #include <torch/csrc/lazy/core/tensor_impl.h>
11 | #include <torch/csrc/lazy/core/tensor_util.h>
12 | 
13 | #include <ATen/FunctionalTensorWrapper.h>
14 | 
15 | #include <utility>
16 | 
17 | namespace torch::lazy {
18 | namespace {
19 | LazyTensorPtr GetOrCreateLtcTensor(
20 |     const at::Tensor& tensor,
21 |     const BackendDevice& device) {
22 |   if (!tensor.defined()) {
23 |     return torch::lazy::LazyTensorPtr();
24 |   }
25 |   auto lazy_tensor = TryGetLtcTensor(tensor);
26 |   return lazy_tensor ? lazy_tensor : LazyTensor::Create(tensor, device);
27 | }
28 | } // namespace
29 | 
30 | LazyTensor::Data::~Data() {
31 |   LazyGraphExecutor::Get()->UnregisterTensor(this);
32 | }
33 | 
34 | LazyTensorPtr LazyTensor::Create(
35 |     const at::Tensor& tensor,
36 |     const BackendDevice& device) {
37 |   TORCH_CHECK(tensor.device().type() != at::kLazy);
38 |   LazyTensorPtr lazy_tensor =
39 |       c10::make_intrusive<LazyTensor>(LazyTensor(tensor, device));
40 |   LazyGraphExecutor::Get()->RegisterTensor(lazy_tensor->data());
41 |   return lazy_tensor;
42 | }
43 | 
44 | LazyTensorPtr LazyTensor::Create(Value ir_value, const BackendDevice& device) {
45 |   LazyTensorPtr lazy_tensor =
46 |       c10::make_intrusive<LazyTensor>(LazyTensor(std::move(ir_value), device));
47 |   LazyGraphExecutor::Get()->RegisterTensor(lazy_tensor->data());
48 |   return lazy_tensor;
49 | }
50 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/tensor.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/core/helpers.h>` and system or third-party headers such as `<utility>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Implements routines such as `GetOrCreateLtcTensor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/core/config.h>`、`<torch/csrc/lazy/core/tensor.h>`、`<c10/util/irange.h>`、`<torch/csrc/lazy/core/helpers.h>`以及系统或第三方头文件，例如 `<utility>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `GetOrCreateLtcTensor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 51-103
```cpp
 51 | LazyTensorPtr LazyTensor::Create(const BackendDataPtr& handle) {
 52 |   LazyTensorPtr lazy_tensor =
 53 |       c10::make_intrusive<LazyTensor>(LazyTensor(handle));
 54 |   LazyGraphExecutor::Get()->RegisterTensor(lazy_tensor->data());
 55 |   return lazy_tensor;
 56 | }
 57 | 
 58 | LazyTensorPtr LazyTensor::Create(std::shared_ptr<Data> data) {
 59 |   return c10::make_intrusive<LazyTensor>(LazyTensor(std::move(data)));
 60 | }
 61 | 
 62 | LazyTensor::LazyTensor(const at::Tensor& tensor, const BackendDevice& device)
 63 |     : LazyTensor(std::make_shared<Data>(tensor, device)) {}
 64 | 
 65 | LazyTensor::LazyTensor(const BackendDataPtr& handle)
 66 |     : LazyTensor(std::make_shared<Data>(handle, handle->device())) {}
 67 | 
 68 | LazyTensor::LazyTensor(Value ir_value, const BackendDevice& device)
 69 |     : LazyTensor(std::make_shared<Data>(std::move(ir_value), device)) {
 70 |   TryLimitGraphSize();
 71 | }
 72 | 
 73 | LazyTensor::LazyTensor(std::shared_ptr<Data> data) : data_(std::move(data)) {}
 74 | 
 75 | auto LazyTensor::data() const -> const std::shared_ptr<Data>& {
 76 |   TORCH_CHECK(data_ != nullptr, "Trying to access a null cursor");
 77 |   return data_;
 78 | }
 79 | 
 80 | int64_t LazyTensor::size(int64_t dim) const {
 81 |   auto tensor_shape = shape();
 82 |   auto rank = tensor_shape.Get().dim();
 83 |   auto dim_index = GetCanonicalDimensionIndex(dim, rank);
 84 |   return tensor_shape.Get().size(dim_index);
 85 | }
 86 | 
 87 | at::ScalarType LazyTensor::dtype() const {
 88 |   return shape().Get().scalar_type();
 89 | }
 90 | 
 91 | MaybeRef<Shape> LazyTensor::shape() const {
 92 |   if (data()->handle != nullptr) {
 93 |     return Shape(data()->handle->shape());
 94 |   }
 95 |   if (data()->ir_value) {
 96 |     // TODO(whc) remove shape from LazyTensor API too!
 97 |     return data()->ir_value.shape();
 98 |   }
 99 |   auto const& tensor_data = data()->tensor_data;
100 |   TORCH_CHECK(tensor_data);
101 |   return Shape(tensor_data->scalar_type(), ToI64Vector(tensor_data->sizes()));
102 | }
103 | 
```
- EN: Implements routines such as `shape`, `Shape`, `data` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 实现了 `shape`、`Shape`、`data` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 104-157
```cpp
104 | const BackendDevice& LazyTensor::GetDevice() const {
105 |   return data()->device;
106 | }
107 | 
108 | int64_t LazyTensor::GetUniqueId() const {
109 |   return data()->unique_id;
110 | }
111 | 
112 | BackendDataPtr LazyTensor::GetDataHandle() {
113 |   BackendDataPtr handle = CurrentDataHandle();
114 |   if (handle != nullptr) {
115 |     TORCH_CHECK(
116 |         handle->HasValue(),
117 |         "Trying to access data while an async operation is in flight: ",
118 |         handle->shape().to_string());
119 |     return handle;
120 |   }
121 | 
122 |   if (data()->ir_value) {
123 |     ApplyPendingGraph();
124 |   } else {
125 |     auto const& tensor_data = data()->tensor_data;
126 |     TORCH_CHECK(tensor_data.has_value());
127 |     data()->handle = TensorToDataHandle(*tensor_data, GetDevice());
128 |   }
129 | 
130 |   return data()->handle;
131 | }
132 | 
133 | BackendDataPtr LazyTensor::CurrentDataHandle() const {
134 |   return data()->handle;
135 | }
136 | 
137 | void LazyTensor::SetDataHandle(BackendDataPtr handle) {
138 |   SetDataHandle(std::move(handle), /*sync=*/true);
139 | }
140 | 
141 | void LazyTensor::SetDataHandle(BackendDataPtr handle, bool sync) {
142 |   data()->handle = std::move(handle);
143 |   // Assigning a device data should always clear the IR node, to allow graph
144 |   // trimming.
145 |   AssignIrValue(Value());
146 |   if (sync) {
147 |     data()->tensor_data = std::nullopt;
148 |   }
149 | }
150 | 
151 | void LazyTensor::SetIrValue(Value ir_value) {
152 |   data()->handle = nullptr;
153 |   data()->tensor_data = std::nullopt;
154 |   AssignIrValue(std::move(ir_value));
155 |   TryLimitGraphSize();
156 | }
157 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 158-210
```cpp
158 | void LazyTensor::SetInPlaceIrValue(Value ir_value) {
159 |   auto tensor_shape = shape();
160 |   if (tensor_shape.Get().scalar_type() != ir_value.shape().scalar_type()) {
161 |     ir_value =
162 |         MakeCast(ir_value, tensor_shape.Get().scalar_type(), std::nullopt);
163 |   }
164 |   SetIrValue(std::move(ir_value));
165 | }
166 | 
167 | void LazyTensor::AssignIrValue(Value ir_value) const {
168 |   data()->ir_value = std::move(ir_value);
169 |   data()->generation += 1;
170 | }
171 | 
172 | void LazyTensor::TryLimitGraphSize() {
173 |   if (data()->ir_value &&
174 |       LazyGraphExecutor::Get()->IncTrimCounter() %
175 |               FLAGS_torch_lazy_trim_graph_check_frequency ==
176 |           0) {
177 |     size_t graph_size = Util::GetGraphSize({data()->ir_value.node.get()});
178 |     if (static_cast<int64_t>(graph_size) > FLAGS_torch_lazy_trim_graph_size) {
179 |       TORCH_LAZY_COUNTER("TrimIrGraph", 1);
180 |       ApplyPendingGraph();
181 |     }
182 |   }
183 | }
184 | 
185 | Value LazyTensor::GetIrValue() const {
186 |   Value ir_value = CurrentIrValue();
187 |   if (ir_value) {
188 |     return ir_value;
189 |   }
190 |   BackendDataPtr handle = CurrentDataHandle();
191 |   if (handle != nullptr) {
192 |     // In case of tensor node, we do not clear the data when we set the IR
193 |     // node. This because we want further calls to GetIrValue() to fetch the
194 |     // same IR node, and not create new ones (even though the lowering context
195 |     // will still collapse them all into a single parameter op). So the call
196 |     // which wants the data will still find it, w/out having to fetch it via
197 |     // a computation client from-server call.
198 |     AssignIrValue(CreateTensorNode(handle, /*read_only=*/false));
199 |     return data()->ir_value;
200 |   }
201 |   std::optional<at::Tensor> tensor_data = CurrentTensorData();
202 |   TORCH_CHECK(tensor_data);
203 |   AssignIrValue(GetIrValueForTensor(*tensor_data, GetDevice()));
204 |   return data()->ir_value;
205 | }
206 | 
207 | Value LazyTensor::CurrentIrValue() const {
208 |   return data()->ir_value;
209 | }
210 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 211-258
```cpp
211 | void LazyTensor::SetTensorData(at::Tensor tensor_data) {
212 |   data()->tensor_data = std::move(tensor_data);
213 | }
214 | 
215 | std::optional<at::Tensor> LazyTensor::CurrentTensorData() const {
216 |   return data()->tensor_data;
217 | }
218 | 
219 | Value LazyTensor::GetIrValueForTensor(
220 |     const at::Tensor& tensor,
221 |     const BackendDevice& device) const {
222 |   BackendDataPtr data;
223 |   bool read_only = false;
224 |   if (tensor.dim() == 0 && tensor.numel() == 1) {
225 |     at::Scalar value = tensor.item();
226 |     if (IsSpecialScalar(value)) {
227 |       return MakeScalar(value, tensor.scalar_type());
228 |     }
229 |     data = LazyGraphExecutor::Get()->GetDeviceData(tensor.cpu(), device);
230 |     read_only = true;
231 |   } else {
232 |     TORCH_LAZY_TIMED("IrValueTensorToDataHandle");
233 |     data = TensorToDataHandle(tensor, device);
234 |   }
235 |   return CreateTensorNode(data, read_only);
236 | }
237 | 
238 | at::Tensor LazyTensor::ToTensor(bool detached) {
239 |   at::Tensor tensor;
240 |   std::optional<at::Tensor> tensor_data = CurrentTensorData();
241 |   if (!tensor_data) {
242 |     LazyGraphExecutor::Get()->DeviceBarrier(GetDevice());
243 |     // The GetDataHandle() call will trigger an ApplyPendingGraph() if an IR
244 |     // Node is available on the tensor.
245 |     std::vector<at::Tensor> tensors =
246 |         DataHandlesToTensors({GetDataHandle()}, dtype());
247 |     tensor = std::move(tensors.front());
248 |     if (!detached) {
249 |       SetTensorData(tensor);
250 |     }
251 |   } else {
252 |     tensor = *tensor_data;
253 |     if (detached) {
254 |       if (data()->ir_value || data()->handle != nullptr) {
255 |         // If we have other authoritative sources, just drop our reference and
256 |         // transfer it to the caller.
257 |         data()->tensor_data = std::nullopt;
258 |       } else {
```
- EN: Implements routines such as `MakeScalar`, `CreateTensorNode` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `MakeScalar`、`CreateTensorNode` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 259-313
```cpp
259 |         // Otherwise we need to make a copy to prevent the caller changing our
260 |         // version.
261 |         tensor = CopyTensor(tensor);
262 |       }
263 |     }
264 |   }
265 |   return tensor;
266 | }
267 | 
268 | void LazyTensor::ShallowCopyTo(const LazyTensorPtr& dest) const {
269 |   dest->SetIrValue(GetIrValue());
270 | }
271 | 
272 | void LazyTensor::SetTensor(at::Tensor tensor) {
273 |   SetTensorData(std::move(tensor));
274 |   data()->handle = nullptr;
275 |   AssignIrValue(Value());
276 | }
277 | 
278 | void LazyTensor::UpdateFromTensor(const at::Tensor& tensor, bool sync) {
279 |   if (sync) {
280 |     at::Tensor typed_tensor = CopyTensor(tensor, dtype(), /*copy=*/false);
281 |     SetIrValue(GetIrValueForTensor(typed_tensor, GetDevice()));
282 |   } else {
283 |     SetTensorData(tensor);
284 |     data()->handle = nullptr;
285 |     AssignIrValue(Value());
286 |   }
287 | }
288 | 
289 | void LazyTensor::UpdateFromTensorOut(const at::Tensor& tensor) {
290 |   UpdateFromTensor(tensor, /*sync=*/false);
291 | }
292 | 
293 | void LazyTensor::UpdateFromTensorOut(const LazyTensorPtr& tensor) {
294 |   SetIrValue(tensor->GetIrValue());
295 | }
296 | 
297 | Value LazyTensor::CreateTensorNode(const BackendDataPtr& data, bool read_only)
298 |     const {
299 |   data->SetInfo(std::make_shared<LazyGraphExecutor::DeviceDataInfo>(
300 |       GetUniqueId(), read_only));
301 |   return MakeDeviceData(data);
302 | }
303 | 
304 | std::vector<LazyTensorPtr> LazyTensor::MakeOutputTensors(
305 |     const NodePtr& node) const {
306 |   std::vector<LazyTensorPtr> tensors;
307 |   tensors.reserve(node->num_outputs());
308 |   for (const auto i : c10::irange(node->num_outputs())) {
309 |     tensors.push_back(Create(Value(node, i), GetDevice()));
310 |   }
311 |   return tensors;
312 | }
313 | 
```
- EN: Implements routines such as `MakeDeviceData` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `MakeDeviceData` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 314-368
```cpp
314 | LazyTensorPtr LazyTensor::CopyTensorToDevice(const BackendDevice& device) {
315 |   // TODO: This can be optimized.
316 |   return Create(ToTensor(/*detached=*/true), device);
317 | }
318 | 
319 | void LazyTensor::ApplyPendingGraph() {
320 |   LazyGraphExecutor::Get()->DeviceBarrier(GetDevice());
321 |   // This method is called to ensure that the tensor data is available on
322 |   // device, so that a call to CurrentDataHandle() returns a valid pointer.
323 |   if (CurrentDataHandle() == nullptr) {
324 |     std::vector<LazyTensorPtr> tensors(
325 |         {c10::make_intrusive<LazyTensor>(LazyTensor(*this))});
326 |     LazyGraphExecutor::Get()->SyncTensorsGraph(
327 |         &tensors,
328 |         {},
329 |         /*wait=*/true,
330 |         /*sync_ltc_data=*/false);
331 |   }
332 | }
333 | 
334 | int64_t LazyTensor::GetNextTensorId() {
335 |   static std::atomic<int64_t>* id_generator = new std::atomic<int64_t>(1);
336 |   return id_generator->fetch_add(1);
337 | }
338 | 
339 | torch::lazy::Value GetTensorList(at::ITensorListRef tensors) {
340 |   std::vector<Value> values;
341 |   for (const auto& t : tensors) {
342 |     auto* impl = dynamic_cast<LTCTensorImpl*>(t.unsafeGetTensorImpl());
343 |     TORCH_INTERNAL_ASSERT(
344 |         impl,
345 |         "GetTensorList only supports lists of valid tensors, but optional support could be added");
346 |     values.push_back(impl->tensor()->GetIrValue());
347 |   }
348 | 
349 |   return torch::lazy::Value(torch::lazy::MakeTensorList(values));
350 | }
351 | 
352 | LazyTensorPtr TryGetLtcTensor(const at::Tensor& tensor) {
353 |   auto* impl = dynamic_cast<LTCTensorImpl*>(
354 |       maybe_unwrap_functional(tensor).unsafeGetTensorImpl());
355 |   if (impl == nullptr) {
356 |     // return c10::make_intrusive<LazyTensor>();
357 |     return LazyTensorPtr();
358 |   }
359 |   return impl->tensor();
360 | }
361 | 
362 | LazyTensorPtr GetLtcTensor(const at::Tensor& tensor) {
363 |   auto lazy_tensor = TryGetLtcTensor(tensor);
364 |   TORCH_CHECK(
365 |       lazy_tensor, "Input tensor is not a lazy tensor: ", tensor.toString());
366 |   return lazy_tensor;
367 | }
368 | 
```
- EN: Implements routines such as `Create`, `GetTensorList`, `TryGetLtcTensor`, `LazyTensorPtr`, `GetLtcTensor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `Create`、`GetTensorList`、`TryGetLtcTensor`、`LazyTensorPtr`、`GetLtcTensor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 369-412
```cpp
369 | std::vector<LazyTensorPtr> GetLtcTensors(c10::ArrayRef<at::Tensor> tensors) {
370 |   std::vector<LazyTensorPtr> ltc_tensors;
371 |   ltc_tensors.reserve(tensors.size());
372 |   for (const auto& tensor : tensors) {
373 |     ltc_tensors.emplace_back(TryGetLtcTensor(tensor));
374 |   }
375 |   return ltc_tensors;
376 | }
377 | 
378 | LazyTensorPtr GetOrCreateLtcTensor(
379 |     const std::optional<at::Tensor>& tensor,
380 |     const BackendDevice& device) {
381 |   return GetOrCreateLtcTensor(tensor.value_or(at::Tensor()), device);
382 | }
383 | 
384 | LazyTensorPtr GetLtcTensorOrCreateForWrappedNumber(
385 |     const at::Tensor& tensor,
386 |     const BackendDevice& device) {
387 |   // TODO: There are places in core where a scalar is wrapped but not marked as
388 |   // wrapped.
389 |   return (tensor.unsafeGetTensorImpl()->is_wrapped_number() ||
390 |           (tensor.dim() == 0 && tensor.numel() == 1))
391 |       ? GetOrCreateLtcTensor(tensor, device)
392 |       : GetLtcTensor(tensor);
393 | }
394 | 
395 | at::Tensor CreateAtenFromLtcTensor(const LazyTensorPtr& ltc_tensor) {
396 |   return ltc_tensor ? at::Tensor(c10::make_intrusive<LTCTensorImpl>(ltc_tensor))
397 |                     : at::Tensor();
398 | }
399 | 
400 | at::Tensor CreateAtenFromLtcTensor(LazyTensor&& ltc_tensor) {
401 |   return at::Tensor(c10::make_intrusive<LTCTensorImpl>(std::move(ltc_tensor)));
402 | }
403 | 
404 | at::Tensor to_lazy_tensor(
405 |     const at::Tensor& self,
406 |     const c10::TensorOptions& options,
407 |     at::Device device,
408 |     bool non_blocking,
409 |     bool functionalize_output) {
410 |   TORCH_INTERNAL_ASSERT(self.device().type() != c10::kLazy);
411 |   TORCH_INTERNAL_ASSERT(device.type() == c10::kLazy);
412 | 
```
- EN: Implements routines such as `GetLtcTensors`, `GetOrCreateLtcTensor`, `GetLtcTensorOrCreateForWrappedNumber`, `CreateAtenFromLtcTensor`, `to_lazy_tensor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `GetLtcTensors`、`GetOrCreateLtcTensor`、`GetLtcTensorOrCreateForWrappedNumber`、`CreateAtenFromLtcTensor`、`to_lazy_tensor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 413-426
```cpp
413 |   auto eager_tensor =
414 |       self.to(options, /*non_blocking=*/non_blocking, /*copy=*/true);
415 |   auto lazy_self = torch::lazy::GetOrCreateLtcTensor(
416 |       eager_tensor, torch::lazy::atenDeviceToBackendDevice(device));
417 |   auto out = torch::lazy::CreateAtenFromLtcTensor(lazy_self);
418 |   if (functionalize_output) {
419 |     // See Note [Lazy Tensor Functionalization]
420 |     return at::functionalization::impl::to_functional_tensor(out);
421 |   } else {
422 |     return out;
423 |   }
424 | }
425 | 
426 | } // namespace torch::lazy
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `GetOrCreateLtcTensor`, `shape`, `Shape`, `data`, `MakeScalar`, `CreateTensorNode`, `MakeDeviceData`, `Create`.
  - CN: `GetOrCreateLtcTensor`、`shape`、`Shape`、`data`、`MakeScalar`、`CreateTensorNode`、`MakeDeviceData`、`Create`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/tensor.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/core/helpers.h>`, `<torch/csrc/lazy/core/ir_builder.h>`, `<torch/csrc/lazy/core/ir_dump_util.h>`, `<torch/csrc/lazy/core/lazy_graph_executor.h>`, `<torch/csrc/lazy/core/metrics.h>`, `<torch/csrc/lazy/core/tensor_impl.h>`, `<torch/csrc/lazy/core/tensor_util.h>`, `<ATen/FunctionalTensorWrapper.h>`
- External includes / 外部头文件: `<utility>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
