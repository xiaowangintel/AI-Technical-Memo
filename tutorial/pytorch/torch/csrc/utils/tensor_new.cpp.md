# tensor_new.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/tensor_new.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-96
```cpp
 1 | #include <torch/csrc/python_headers.h>
 2 | #include <torch/csrc/utils/tensor_new.h>
 3 | 
 4 | #include <pybind11/pybind11.h>
 5 | #include <torch/csrc/DynamicTypes.h>
 6 | #include <torch/csrc/Exceptions.h>
 7 | #include <torch/csrc/Size.h>
 8 | #include <torch/csrc/autograd/generated/variable_factories.h>
 9 | #include <torch/csrc/autograd/variable.h>
10 | #include <torch/csrc/utils/device_lazy_init.h>
11 | #include <torch/csrc/utils/numpy_stub.h>
12 | #include <torch/csrc/utils/pybind.h>
13 | #include <torch/csrc/utils/python_arg_parser.h>
14 | #include <torch/csrc/utils/python_numbers.h>
15 | #include <torch/csrc/utils/python_scalars.h>
16 | #include <torch/csrc/utils/python_strings.h>
17 | #include <torch/csrc/utils/tensor_numpy.h>
18 | 
19 | #include <ATen/ATen.h>
20 | #include <ATen/DLConvertor.h>
21 | #include <ATen/InitialTensorOptions.h>
22 | #include <ATen/NamedTensorUtils.h>
23 | #include <ATen/NativeFunctions.h>
24 | #include <ATen/SparseCsrTensorUtils.h>
25 | #include <ATen/TracerMode.h>
26 | #include <ATen/dlpack.h>
27 | #include <c10/core/Backend.h>
28 | #include <c10/core/DispatchKeySet.h>
29 | #include <c10/core/Layout.h>
30 | #include <c10/util/Exception.h>
31 | #include <c10/util/irange.h>
32 | #include <optional>
33 | 
34 | #include <stdexcept>
35 | #include <vector>
36 | 
37 | using at::Device;
38 | using at::IntArrayRef;
39 | using at::kInt;
40 | using at::kLong;
41 | using at::ScalarType;
42 | using at::Storage;
43 | using at::Tensor;
44 | using at::TensorOptions;
45 | using std::optional;
46 | 
47 | namespace torch::utils {
48 | namespace {
49 | const int MAX_DIMS = 128;
50 | 
51 | thread_local bool kOnlyLiftCPUTensors = false;
52 | 
53 | TensorOptions build_options(
54 |     c10::TensorOptions options,
55 |     at::ScalarType scalar_type,
56 |     const std::optional<Device>& device = std::nullopt) {
57 |   options = options.dtype(scalar_type);
58 |   if (device.has_value()) {
59 |     return options.device(device);
60 |   }
61 |   return options;
62 | }
63 | 
64 | // NB: It appears there is some consistency invariant between options and
65 | // device, where if device is non-empty, its type must be consistent with the
66 | // device type in options.
67 | // TODO: Refactor this so we just pass everything in via options
68 | 
69 | Tensor new_with_sizes(
70 |     c10::TensorOptions options,
71 |     at::ScalarType scalar_type,
72 |     const std::optional<Device>& device,
73 |     c10::SymIntArrayRef sizes) {
74 |   maybe_initialize_device(options.device());
75 |   pybind11::gil_scoped_release no_gil;
76 |   return at::empty_symint(sizes, build_options(options, scalar_type, device));
77 | }
78 | 
79 | Tensor new_with_storage(
80 |     c10::TensorOptions options,
81 |     at::ScalarType scalar_type,
82 |     Storage storage) {
83 |   auto tensor = at::empty({}, build_options(options, scalar_type));
84 |   tensor.set_(std::move(storage));
85 |   return tensor;
86 | }
87 | 
88 | std::vector<int64_t> compute_sizes(PyObject* seq, ScalarType scalar_type) {
89 |   bool is_storage = isStorage(seq);
90 |   std::vector<int64_t> sizes;
91 |   // Note that after the first iteration, obj is the only thing that keeps
92 |   // the seq raw pointer alive.
93 |   THPObjectPtr obj;
94 |   while (PySequence_Check(seq)) {
95 |     auto length = PySequence_Length(seq);
96 |     if (length < 0)
```
- EN: Brings in project headers such as `<torch/csrc/python_headers.h>`, `<torch/csrc/utils/tensor_new.h>`, `<torch/csrc/DynamicTypes.h>`, `<torch/csrc/Exceptions.h>` and system or third-party headers such as `<pybind11/pybind11.h>`, `<optional>`, `<stdexcept>`, `<vector>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::utils`) so ownership matches the PyTorch subsystem layout. Implements routines such as `build_options`, `new_with_sizes`, `new_with_storage`, `compute_sizes` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/python_headers.h>`、`<torch/csrc/utils/tensor_new.h>`、`<torch/csrc/DynamicTypes.h>`、`<torch/csrc/Exceptions.h>`以及系统或第三方头文件，例如 `<pybind11/pybind11.h>`、`<optional>`、`<stdexcept>`、`<vector>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::utils`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `build_options`、`new_with_sizes`、`new_with_storage`、`compute_sizes` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 97-192
```cpp
 97 |       throw python_error();
 98 |     if (is_storage) {
 99 |       length /= static_cast<int64_t>(elementSize(scalar_type));
100 |     }
101 |     sizes.push_back(length);
102 |     TORCH_CHECK_VALUE(
103 |         sizes.size() <= MAX_DIMS,
104 |         "too many dimensions '",
105 |         Py_TYPE(seq)->tp_name,
106 |         "'");
107 |     if (length == 0)
108 |       break;
109 |     PyObject* new_obj = PySequence_GetItem(seq, 0);
110 |     // This line uses seq so we must NOT override obj before this line
111 |     TORCH_CHECK_VALUE(
112 |         new_obj,
113 |         "could not determine the shape of object type '",
114 |         Py_TYPE(seq)->tp_name,
115 |         "'");
116 |     obj = THPObjectPtr(new_obj);
117 |     seq = obj.get();
118 |   }
119 | 
120 |   return sizes;
121 | }
122 | 
123 | ScalarType infer_scalar_type(PyObject* obj) {
124 |   if (torch::is_symint(obj)) {
125 |     return ScalarType::Long;
126 |   }
127 |   if (torch::is_symfloat(obj)) {
128 |     return torch::tensors::get_default_scalar_type();
129 |   }
130 | #ifdef USE_NUMPY
131 |   if (is_numpy_available()) {
132 |     if (PyArray_Check(obj)) {
133 |       return numpy_dtype_to_aten(PyArray_TYPE((PyArrayObject*)obj));
134 |     }
135 |     if (PyArray_CheckScalar(obj)) {
136 |       THPObjectPtr arr(PyArray_FromScalar(obj, nullptr));
137 |       return numpy_dtype_to_aten(PyArray_TYPE((PyArrayObject*)arr.get()));
138 |     }
139 |   }
140 | #endif
141 |   if (PyFloat_Check(obj)) {
142 |     // this is always guaranteed to be a floating-point type, and makes it more
143 |     // convenient to write e.g. torch.tensor(0.) than torch.tensor(0.,
144 |     // dtype=torch.Tensor.dtype).
145 |     return torch::tensors::get_default_scalar_type();
146 |   }
147 |   if (THPUtils_checkLong(obj)) {
148 |     return ScalarType::Long;
149 |   }
150 |   if (PyBool_Check(obj)) {
151 |     return ScalarType::Bool;
152 |   }
153 |   if (PyComplex_Check(obj)) {
154 |     switch (torch::tensors::get_default_scalar_type()) {
155 |       case ScalarType::Float:
156 |         return ScalarType::ComplexFloat;
157 |       case ScalarType::Double:
158 |         return ScalarType::ComplexDouble;
159 |       case ScalarType::Half:
160 |         return ScalarType::ComplexHalf;
161 |       default:
162 |         TORCH_CHECK(false, "invalid default scalar type for complex");
163 |     }
164 |   }
165 |   if (THPVariable_Check(obj)) {
166 |     const auto& var = THPVariable_Unpack(obj);
167 |     return var.scalar_type();
168 |   }
169 |   TORCH_CHECK_TYPE(
170 |       !THPUtils_checkString(obj),
171 |       "new(): invalid data type '",
172 |       Py_TYPE(obj)->tp_name,
173 |       "'");
174 |   if (PySequence_Check(obj)) {
175 |     auto length = PySequence_Length(obj);
176 |     if (length < 0)
177 |       throw python_error();
178 |     // match NumPy semantics, except use default tensor type instead of double.
179 |     if (length == 0)
180 |       return torch::tensors::get_default_scalar_type();
181 |     ScalarType scalarType{};
182 |     for (const auto i : c10::irange(length)) {
183 |       THPObjectPtr handle(PySequence_GetItem(obj, i));
184 |       if (!handle)
185 |         throw python_error();
186 |       auto cur_item = handle.get();
187 |       TORCH_CHECK_TYPE(
188 |           cur_item != obj, "new(): self-referential lists are incompatible");
189 |       ScalarType item_scalarType = infer_scalar_type(cur_item);
190 |       scalarType = (i > 0) ? at::promoteTypes(scalarType, item_scalarType)
191 |                            : item_scalarType;
192 |       if (scalarType == ScalarType::ComplexDouble) {
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `python_error`, `infer_scalar_type`, `numpy_dtype_to_aten`, `arr`, `TORCH_CHECK` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `python_error`、`infer_scalar_type`、`numpy_dtype_to_aten`、`arr`、`TORCH_CHECK` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 193-288
```cpp
193 |         // this won't change (unless we hit undefined, but that will fail
194 |         // later).
195 |         return scalarType;
196 |       }
197 |     }
198 |     return scalarType;
199 |   }
200 |   TORCH_CHECK(false, "Could not infer dtype of ", Py_TYPE(obj)->tp_name);
201 | }
202 | 
203 | void recursive_store(
204 |     char* data,
205 |     IntArrayRef sizes,
206 |     IntArrayRef strides,
207 |     int64_t dim,
208 |     ScalarType scalarType,
209 |     size_t elementSize,
210 |     PyObject* obj) {
211 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(data != nullptr);
212 | 
213 |   int64_t ndim = static_cast<int64_t>(sizes.size());
214 |   bool is_symfloat = torch::is_symfloat(obj);
215 |   bool is_symint = torch::is_symint(obj);
216 |   if (dim == ndim) {
217 |     if (is_symfloat) {
218 |       auto new_obj = py::reinterpret_borrow<py::object>(obj);
219 |       auto val = new_obj.cast<c10::SymFloat>();
220 |       const double double_val = val.guard_float(__FILE__, __LINE__);
221 |       obj = Py_BuildValue("d", double_val);
222 |     }
223 |     if (is_symint) {
224 |       auto new_obj = py::reinterpret_borrow<py::object>(obj);
225 |       auto val = new_obj.cast<c10::SymInt>();
226 |       const int64_t int_val = val.guard_int(__FILE__, __LINE__);
227 |       obj = Py_BuildValue("i", int_val);
228 |     }
229 |     torch::utils::store_scalar(data, scalarType, obj);
230 |     return;
231 |   }
232 | 
233 |   auto n = sizes[dim];
234 |   auto seq = THPObjectPtr(PySequence_Fast(obj, "not a sequence"));
235 |   if (!seq)
236 |     throw python_error();
237 |   // NOLINTNEXTLINE(bugprone-branch-clone)
238 |   auto seq_size = PySequence_Fast_GET_SIZE(seq.get());
239 |   TORCH_CHECK_VALUE(
240 |       seq_size == n,
241 |       "expected sequence of length ",
242 |       n,
243 |       " at dim ",
244 |       dim,
245 |       " (got ",
246 |       seq_size,
247 |       ")");
248 | 
249 |   PyObject** items = PySequence_Fast_ITEMS(seq.get());
250 |   for (const auto i : c10::irange(n)) {
251 | #ifdef USE_NUMPY
252 |     if (is_numpy_available() && PyArray_Check(items[i])) {
253 |       TORCH_WARN_ONCE(
254 |           "Creating a tensor from a list of numpy.ndarrays is extremely slow. "
255 |           "Please consider converting the list to a single numpy.ndarray with "
256 |           "numpy.array() before converting to a tensor.");
257 |     }
258 | #endif
259 |     recursive_store(
260 |         data, sizes, strides, dim + 1, scalarType, elementSize, items[i]);
261 |     data += strides[dim] * elementSize;
262 |   }
263 | }
264 | 
265 | Tensor internal_new_from_data(
266 |     c10::TensorOptions options,
267 |     at::ScalarType scalar_type,
268 |     std::optional<Device> device_opt,
269 |     PyObject* data,
270 |     bool copy_variables,
271 |     bool copy_numpy,
272 |     bool type_inference,
273 |     bool pin_memory = false) {
274 |   TORCH_CHECK_TYPE(
275 |       !THPUtils_checkString(data),
276 |       "new(): invalid data type '",
277 |       Py_TYPE(data)->tp_name,
278 |       "'");
279 | 
280 |   if (THPVariable_Check(data)) {
281 |     TORCH_CHECK(!pin_memory, "Can't pin tensor constructed from a variable");
282 |     // TODO: use MaybeOwned
283 |     auto var = THPVariable_Unpack(data);
284 |     if (copy_variables) {
285 |       var = var.detach();
286 |     }
287 |     // infer the scalar type and device type; it's not expected to infer the
288 |     // layout since these constructors are defined per-layout-type (e.g. tensor
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `recursive_store`, `python_error`, `internal_new_from_data` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `recursive_store`、`python_error`、`internal_new_from_data` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 289-384
```cpp
289 |     // vs sparse_coo_tensor).
290 |     const auto& inferred_scalar_type =
291 |         type_inference ? var.scalar_type() : scalar_type;
292 |     auto device = device_opt.has_value() ? *device_opt : var.device();
293 |     pybind11::gil_scoped_release no_gil;
294 |     maybe_initialize_device(device);
295 |     return var.to(
296 |         device,
297 |         inferred_scalar_type,
298 |         /*non_blocking=*/false,
299 |         /*copy=*/copy_variables);
300 |   }
301 | 
302 | #ifdef USE_NUMPY
303 |   if (PyObject_HasAttrString(data, "__cuda_array_interface__")) {
304 |     TORCH_CHECK(
305 |         !pin_memory,
306 |         "Can't pin tensor constructed from __cuda_array_interface__");
307 |     auto tensor = tensor_from_cuda_array_interface(data, device_opt);
308 |     const auto& inferred_scalar_type =
309 |         type_inference ? tensor.scalar_type() : scalar_type;
310 | 
311 |     // Device preference is:
312 |     //  - explicitly user specified device in `device_opt`
313 |     //      - either by setting device='...'
314 |     //      - or setting torch.set_default_device(...)
315 |     //  - device of already constructed tensor
316 |     // This prevents an unnecessary device -> host copy when the tensor is
317 |     // already on the device, while respecting a default device and allows the
318 |     // user to overwrite the behavior explicitly.
319 |     at::Device device = device_opt.has_value() ? *device_opt : tensor.device();
320 | 
321 |     pybind11::gil_scoped_release no_gil;
322 |     maybe_initialize_device(device);
323 |     return tensor.to(
324 |         device,
325 |         inferred_scalar_type,
326 |         /*non_blocking=*/false,
327 |         /*copy=*/copy_numpy);
328 |   }
329 | 
330 |   if (is_numpy_available() && PyArray_Check(data)) {
331 |     TORCH_CHECK(!pin_memory, "Can't pin tensor constructed from numpy");
332 |     auto tensor =
333 |         tensor_from_numpy(data, /*warn_if_not_writeable=*/!copy_numpy);
334 |     const auto& inferred_scalar_type =
335 |         type_inference ? tensor.scalar_type() : scalar_type;
336 |     auto device = device_opt.has_value() ? *device_opt : options.device();
337 |     pybind11::gil_scoped_release no_gil;
338 |     maybe_initialize_device(device);
339 |     return tensor.to(
340 |         device,
341 |         inferred_scalar_type,
342 |         /*non_blocking=*/false,
343 |         /*copy=*/copy_numpy);
344 |   }
345 | #endif
346 | 
347 |   if (PyObject_HasAttrString(data, "__dlpack__")) {
348 |     py::object tensor_o =
349 |         py::module::import("torch").attr("utils").attr("dlpack").attr(
350 |             "from_dlpack")(py::handle(data));
351 |     Tensor tensor = py::cast<Tensor>(tensor_o);
352 |     const auto& inferred_scalar_type =
353 |         type_inference ? tensor.scalar_type() : scalar_type;
354 |     auto device = device_opt.has_value() ? *device_opt : tensor.device();
355 |     pybind11::gil_scoped_release no_gil;
356 |     maybe_initialize_device(device);
357 |     return tensor.to(
358 |         device,
359 |         inferred_scalar_type,
360 |         /*non_blocking=*/false,
361 |         /*copy=*/copy_variables);
362 |   }
363 | 
364 |   auto device = device_opt.has_value() ? *device_opt : options.device();
365 | 
366 |   auto sizes = compute_sizes(data, scalar_type);
367 | 
368 |   ScalarType inferred_scalar_type =
369 |       type_inference ? infer_scalar_type(data) : scalar_type;
370 |   // This exists to prevent us from tracing the call to empty().  The actual
371 |   // autograd code doesn't really matter, because requires_grad is always false
372 |   // here.
373 |   // What are the semantics of tensor_new()?
374 |   // We manually construct a tensor and place on it on the correct device with
375 |   // empty() and to(). We then have to "lift" the newly constructed tensor in
376 |   // some cases, like when we're performing a functorch transform or running
377 |   // functionalization. The exclude guards are all to ensure that extra logic
378 |   // doesn't run when we're constructing the raw tensor.
379 |   Tensor tensor;
380 |   {
381 |     at::AutoDispatchBelowADInplaceOrView guard;
382 |     c10::impl::ExcludeDispatchKeyGuard torchdispatchmode_guard(
383 |         c10::DispatchKey::Python);
384 |     c10::impl::ExcludeDispatchKeyGuard torchdispatchmode_snapshot_guard(
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `torchdispatchmode_guard` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `torchdispatchmode_guard` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 385-480
```cpp
385 |         c10::DispatchKey::PythonTLSSnapshot);
386 |     // functorch uses FuncTorchDynamicLayerBackMode as a mode key to wrap all
387 |     // tensors returned from operators in special TensorWrapper tensor extension
388 |     c10::impl::ExcludeDispatchKeyGuard functorch_front_guard(
389 |         c10::DispatchKey::FuncTorchDynamicLayerFrontMode);
390 |     c10::impl::ExcludeDispatchKeyGuard functorch_back_guard(
391 |         c10::DispatchKey::FuncTorchDynamicLayerBackMode);
392 |     // We disable Fake and DeferredInit handlers for similar reasons as
393 |     // functorch.
394 |     c10::impl::ExcludeDispatchKeyGuard fake_and_deferred_init_guard(
395 |         c10::DispatchKeySet{
396 |             c10::DispatchKey::Fake, c10::DispatchKey::DeferredInit});
397 |     // Note [Functionalization <> torch.Tensor constructor]
398 |     // Functionalization "lifts" the newly constructed tensor into a wrapper
399 |     // using aten::lift().
400 |     c10::impl::ExcludeDispatchKeyGuard functionalize_guard(
401 |         c10::DispatchKey::Functionalize);
402 |     {
403 |       // Tracing should probably also use the "lift" operator to add the tensor
404 |       // to a trace, but it's technically BC-breaking to do that, since we
405 |       // currently trace .to() calls.
406 |       at::tracer::impl::NoTracerDispatchMode tracer_guard;
407 | 
408 |       if (isStorage(data)) {
409 |         auto [storage, storage_scalar_type, is_typed_storage] =
410 |             createStorageGetType(data);
411 | 
412 |         TORCH_CHECK(
413 |             !is_typed_storage || storage_scalar_type == scalar_type,
414 |             "Expected a Storage of type ",
415 |             scalar_type,
416 |             " or an UntypedStorage, but got ",
417 |             storage_scalar_type);
418 |         tensor = at::empty(
419 |             {0}, // sizes. Storage will be set later.
420 |             at::initialTensorOptions()
421 |                 .dtype(
422 |                     is_typed_storage ? storage_scalar_type
423 |                                      : inferred_scalar_type)
424 |                 .pinned_memory(pin_memory)
425 |                 .device(storage.device()));
426 |         tensor.set_(storage);
427 | 
428 |       } else {
429 |         TensorOptions opts =
430 |             at::initialTensorOptions().dtype(inferred_scalar_type);
431 | 
432 |         // If the device is Meta, take the shortcut. We don't want to allocate
433 |         // an empty CPU tensor which would break our contract for meta tensors.
434 |         if (device == at::kMeta) {
435 |           return at::empty(sizes, opts.device(device));
436 |         }
437 |         tensor = at::empty(sizes, opts.pinned_memory(pin_memory));
438 |         if (c10::multiply_integers(tensor.sizes()) != 0) {
439 |           recursive_store(
440 |               (char*)tensor.data_ptr(),
441 |               tensor.sizes(),
442 |               tensor.strides(),
443 |               0,
444 |               inferred_scalar_type,
445 |               tensor.dtype().itemsize(),
446 |               data);
447 |         }
448 |       }
449 |     }
450 |     pybind11::gil_scoped_release no_gil;
451 |     maybe_initialize_device(device);
452 |     // However, it is VERY important that we trace the to() call here (even
453 |     // though the reason this is important is a hack).  Without *some* factory
454 |     // function call that is traced at construction time, we will consider
455 |     // a tensor constant as originating from "outside" the trace, and if you
456 |     // try to return it directly we will fail with the error saying no
457 |     // "no observable data dependence".  In an ideal world, we wouldn't trace
458 |     // a to() call but I need to think harder about what exactly we should trace
459 |     // in this case.
460 |     if (only_lift_cpu_tensors()) {
461 |       tensor = tensor.to(
462 |           inferred_scalar_type, /*non_blocking=*/false, /*copy=*/false);
463 | 
464 |     } else {
465 |       tensor = tensor.to(
466 |           device, inferred_scalar_type, /*non_blocking=*/false, /*copy=*/false);
467 |     }
468 |   }
469 | 
470 |   // torch.jit.trace will continue to trace out `.to()` instead of `.lift()`,
471 |   // since changing it is BC-breaking.
472 |   at::tracer::impl::NoTracerDispatchMode tracer_guard;
473 |   {
474 |     // lift has no autograd implementation, so we need to make sure we don't try
475 |     // to dispatch to it.
476 |     // TODO: arguably it should have an autograd implementation that noops
477 |     at::AutoDispatchBelowADInplaceOrView guard;
478 |     tensor = at::lift_fresh(tensor);
479 |   }
480 |   if (only_lift_cpu_tensors() && device.type() != DeviceType::CPU) {
```
- EN: Implements routines such as `functorch_front_guard`, `functorch_back_guard`, `functionalize_guard` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `functorch_front_guard`、`functorch_back_guard`、`functionalize_guard` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 481-572
```cpp
481 |     if (!device.has_index() &&
482 |         !torch::utils::is_device_initialized(device.type())) {
483 |       // Infer device 0 to avoid device init
484 |       device = c10::Device(device.type(), 0);
485 |     }
486 |     tensor = tensor.to(device, /*non_blocking=*/false, /*copy=*/false);
487 |   }
488 |   return tensor;
489 | }
490 | 
491 | Tensor new_from_data_copy(
492 |     c10::TensorOptions options,
493 |     at::ScalarType scalar_type,
494 |     std::optional<Device> device,
495 |     PyObject* data) {
496 |   return internal_new_from_data(
497 |       options,
498 |       scalar_type,
499 |       device,
500 |       data,
501 |       /*copy_variables=*/true,
502 |       /*copy_numpy=*/true,
503 |       /*type_inference=*/false);
504 | }
505 | 
506 | Tensor legacy_new_from_sequence(
507 |     c10::TensorOptions options,
508 |     at::ScalarType scalar_type,
509 |     std::optional<Device> device,
510 |     PyObject* data) {
511 |   TORCH_CHECK_TYPE(
512 |       PySequence_Check(data),
513 |       "new(): data must be a sequence (got ",
514 |       Py_TYPE(data)->tp_name,
515 |       ")");
516 |   return internal_new_from_data(
517 |       options,
518 |       scalar_type,
519 |       device,
520 |       data,
521 |       /*copy_variables=*/false,
522 |       /*copy_numpy=*/false,
523 |       /*type_inference=*/false);
524 | }
525 | 
526 | // "base" here refers to the Tensor type on which the function was invoked,
527 | // e.g.: in x.new(y), 'x' is the base.
528 | // TODO: Rewrite this using dispatchKeyToTensorOptions
529 | void check_base_legacy_new(
530 |     c10::DispatchKey dispatch_key,
531 |     at::Layout expected_layout) {
532 |   if (expected_layout == c10::kStrided) {
533 |     constexpr c10::DispatchKeySet expected_key_set({
534 |         c10::DispatchKey::CPU,
535 |         c10::DispatchKey::CUDA,
536 |         c10::DispatchKey::HIP,
537 |         c10::DispatchKey::XLA,
538 |         c10::DispatchKey::Lazy,
539 |         c10::DispatchKey::IPU,
540 |         c10::DispatchKey::XPU,
541 |         c10::DispatchKey::HPU,
542 |         c10::DispatchKey::MPS,
543 |         c10::DispatchKey::Meta,
544 |         c10::DispatchKey::PrivateUse1,
545 |     });
546 |     TORCH_CHECK(
547 |         expected_key_set.has(dispatch_key),
548 |         "new(): expected key in ",
549 |         expected_key_set,
550 |         " but got: ",
551 |         dispatch_key);
552 |   } else if (expected_layout == c10::kSparse) {
553 |     // NOTE: no sparse XLA or Lazy
554 |     constexpr c10::DispatchKeySet expected_key_set({
555 |         c10::DispatchKey::SparseCPU,
556 |         c10::DispatchKey::SparseCUDA,
557 |         c10::DispatchKey::SparseHIP,
558 |         c10::DispatchKey::SparseXPU,
559 |         c10::DispatchKey::SparseMPS,
560 |         c10::DispatchKey::SparsePrivateUse1,
561 |     });
562 |     TORCH_CHECK(
563 |         expected_key_set.has(dispatch_key),
564 |         "new(): expected key in ",
565 |         expected_key_set,
566 |         " but got: ",
567 |         dispatch_key);
568 |   } else {
569 |     TORCH_INTERNAL_ASSERT(false, "unexpected layout");
570 |   }
571 | }
572 | 
```
- EN: Implements routines such as `new_from_data_copy`, `internal_new_from_data`, `legacy_new_from_sequence`, `check_base_legacy_new` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `new_from_data_copy`、`internal_new_from_data`、`legacy_new_from_sequence`、`check_base_legacy_new` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 573-668
```cpp
573 | // TODO: Make this accept options instead of dispatch key
574 | void check_legacy_ctor_device(
575 |     c10::DispatchKey dispatch_key,
576 |     std::optional<Device> device) {
577 |   if (device.has_value()) {
578 |     TORCH_CHECK(
579 |         dispatchKeyToDeviceType(dispatch_key) == device.value().type(),
580 |         "legacy constructor expects device type: ",
581 |         dispatchKeyToDeviceType(dispatch_key),
582 |         " but device type: ",
583 |         device.value().type(),
584 |         " was passed");
585 |   }
586 | }
587 | 
588 | std::optional<Device> device_or_from_dispatch_key(
589 |     std::optional<Device> device,
590 |     c10::DispatchKey dispatch_key) {
591 |   if (device.has_value()) {
592 |     return device;
593 |   } else {
594 |     return Device(dispatchKeyToDeviceType(dispatch_key));
595 |   }
596 | }
597 | 
598 | enum class CtorOrNew {
599 |   BASE_CTOR,
600 |   CTOR,
601 |   NEW,
602 | };
603 | 
604 | Tensor legacy_sparse_tensor_generic_ctor_new(
605 |     c10::DispatchKey dispatch_key,
606 |     at::ScalarType scalar_type,
607 |     PyObject* args,
608 |     PyObject* kwargs,
609 |     CtorOrNew ctor_or_new) {
610 |   auto options = dispatchKeyToTensorOptions(dispatch_key);
611 |   static PythonArgParser parser({
612 |       "new(*, Device? device=None)",
613 |       "new(*, int64_t cdata)|hidden",
614 |       "new(Tensor indices, Tensor values, *, Device? device=None)",
615 |       "new(Tensor indices, Tensor values, IntArrayRef size, *, Device? device=None)",
616 |       "new(SymIntArrayRef size, *, Device? device=None)",
617 |   });
618 |   if (ctor_or_new == CtorOrNew::NEW)
619 |     check_base_legacy_new(dispatch_key, c10::kSparse);
620 |   ParsedArgs<4> parsed_args;
621 |   auto r = parser.parse(args, kwargs, parsed_args);
622 |   if (r.idx == 0) {
623 |     if (ctor_or_new == CtorOrNew::CTOR) {
624 |       TORCH_WARN_ONCE(
625 |           "torch.sparse.SparseTensor() is deprecated."
626 |           "  Please use torch.sparse_coo_tensor((0,), dtype=).");
627 |     }
628 |     auto deviceOptional = r.deviceOptional(0);
629 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
630 |     return at::empty({0}, build_options(options, scalar_type, deviceOptional));
631 |   } else if (r.idx == 1) {
632 |     if (ctor_or_new == CtorOrNew::CTOR) {
633 |       TORCH_WARN_ONCE(
634 |           "torch.sparse.SparseTensor(cdata=x._cdata) is deprecated."
635 |           "  Please use torch.sparse_coo_tensor(x._indices(), x._values(), x.shape).");
636 |     }
637 |     // NOLINTNEXTLINE(performance-no-int-to-ptr)
638 |     auto cdata = reinterpret_cast<void*>(r.toInt64(0));
639 |     return at::unsafeTensorFromTH(cdata, true);
640 |   } else if (r.idx == 2) {
641 |     if (ctor_or_new == CtorOrNew::CTOR) {
642 |       TORCH_WARN_ONCE(
643 |           "torch.sparse.SparseTensor(indices, values, *, device=) is deprecated."
644 |           "  Please use torch.sparse_coo_tensor(indices, values, dtype=, device=).");
645 |     }
646 |     // Note: this signature doesn't have a dtype, even though it has a device;
647 |     // it probably shouldn't have a device (we should infer it).
648 |     auto deviceOptional = r.deviceOptional(2);
649 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
650 |     at::OptionalDeviceGuard device_guard(deviceOptional);
651 |     return at::sparse_coo_tensor(r.tensor(0), r.tensor(1));
652 |   } else if (r.idx == 3) {
653 |     if (ctor_or_new == CtorOrNew::CTOR) {
654 |       TORCH_WARN_ONCE(
655 |           "torch.sparse.SparseTensor(indices, values, shape, *, device=) is deprecated."
656 |           "  Please use torch.sparse_coo_tensor(indices, values, shape, dtype=, device=).");
657 |     }
658 |     // Note: this signature doesn't have a dtype, even though it has a device;
659 |     // it probably shouldn't have a device (we should infer it).
660 |     auto deviceOptional = r.deviceOptional(3);
661 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
662 |     at::OptionalDeviceGuard device_guard(deviceOptional);
663 |     return at::sparse_coo_tensor(r.tensor(0), r.tensor(1), r.intlist(2));
664 |   } else if (r.idx == 4) {
665 |     PyObject* arg = r.pyobject(0);
666 |     auto deviceOptional = r.deviceOptional(1);
667 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
668 |     if (!THPSize_Check(arg) && PyTuple_GET_SIZE(args) >= 1 &&
```
- EN: Defines or extends data abstractions such as `CtorOrNew` that structure the state handled by this file. Implements routines such as `check_legacy_ctor_device`, `device_or_from_dispatch_key`, `Device`, `legacy_sparse_tensor_generic_ctor_new`, `device_guard` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 定义或扩展了 `CtorOrNew` 等数据抽象，用来组织本文件处理的状态。 实现了 `check_legacy_ctor_device`、`device_or_from_dispatch_key`、`Device`、`legacy_sparse_tensor_generic_ctor_new`、`device_guard` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 669-764
```cpp
669 |         arg == PyTuple_GET_ITEM(args, 0)) {
670 |       // new(sequence) binds to this signature but should be treated differently
671 |       // unless the sequences is a torch.Size
672 |       if (ctor_or_new == CtorOrNew::CTOR) {
673 |         TORCH_CHECK_TYPE(
674 |             false,
675 |             "torch.sparse.SparseTensor(sequence) only accepts sizes.  Please use torch.sparse_coo_tensor() "
676 |             "or construct a strided tensor and convert it to sparse via to_sparse.");
677 |       } else {
678 |         TORCH_CHECK_TYPE(
679 |             false,
680 |             "SparseTensor.new(sequence) only accepts sizes.  Please use torch.sparse_coo_tensor() "
681 |             "or construct a strided tensor and convert it to sparse via to_sparse.");
682 |       }
683 |     }
684 |     if (ctor_or_new == CtorOrNew::CTOR) {
685 |       TORCH_WARN_ONCE(
686 |           "torch.sparse.SparseTensor(shape, *, device=) is deprecated."
687 |           "  Please use torch.sparse_coo_tensor(shape, dtype=, device=).");
688 |     }
689 |     return new_with_sizes(
690 |         options, scalar_type, deviceOptional, r.symintlist(0));
691 |   }
692 |   TORCH_CHECK(false, "new(): invalid arguments");
693 | }
694 | 
695 | // NB: device_idx here is NOT a DeviceIndex, but index into PythonArgs
696 | c10::TensorOptions typeIdWithDefault(
697 |     PythonArgs& r,
698 |     int64_t device_idx,
699 |     c10::DispatchKey dispatch_key) {
700 |   auto options = dispatchKeyToTensorOptions(dispatch_key);
701 |   if (!r.isNone(static_cast<int>(device_idx))) {
702 |     // TODO: This line doesn't seem to be exercised at all in tests
703 |     options = options.device(r.device(static_cast<int>(device_idx)).type());
704 |   }
705 |   return options;
706 | }
707 | 
708 | } // namespace
709 | 
710 | static Tensor legacy_tensor_generic_ctor_new(
711 |     c10::DispatchKey dispatch_key,
712 |     at::ScalarType scalar_type,
713 |     PyObject* args,
714 |     PyObject* kwargs,
715 |     CtorOrNew ctor_or_new) {
716 |   auto options = dispatchKeyToTensorOptions(dispatch_key);
717 |   static PythonArgParser parser({
718 |       "new(*, Device? device=None)",
719 |       "new(Storage storage)",
720 |       "new(*, int64_t cdata)|hidden",
721 |       // This constructor is no longer legacy, it will also be usable for
722 |       // subclass initialization
723 |       "new(Tensor other)",
724 |       "new(Tensor other, *, Device? device=None)|hidden", // prevent Tensor
725 |                                                           // matching with
726 |                                                           // IntArrayRef,
727 |                                                           // PyObject*
728 |       "new(SymIntArrayRef size, *, Device? device=None)",
729 |       "new(PyObject* data, *, Device? device=None)",
730 |   });
731 | 
732 |   if (isSparse(dispatchKeyToBackend(dispatch_key))) {
733 |     return legacy_sparse_tensor_generic_ctor_new(
734 |         dispatch_key, scalar_type, args, kwargs, ctor_or_new);
735 |   }
736 | 
737 |   if (ctor_or_new == CtorOrNew::NEW)
738 |     check_base_legacy_new(dispatch_key, c10::kStrided);
739 | 
740 |   ParsedArgs<2> parsed_args;
741 |   auto r = parser.parse(args, kwargs, parsed_args);
742 |   if (r.idx == 0) {
743 |     auto deviceOptional = r.deviceOptional(0);
744 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
745 |     at::OptionalDeviceGuard device_guard(deviceOptional);
746 |     return at::empty({0}, build_options(options, scalar_type));
747 |   } else if (r.idx == 1) {
748 |     at::ScalarType storage_scalar_type{at::ScalarType::Undefined};
749 |     bool is_typed_storage = false;
750 |     at::Storage storage = r.storage(0, storage_scalar_type, is_typed_storage);
751 |     if (storage_scalar_type != at::ScalarType::Undefined && is_typed_storage) {
752 |       TORCH_CHECK(
753 |           storage_scalar_type == scalar_type,
754 |           "Expected a Storage of type ",
755 |           scalar_type,
756 |           " or an UntypedStorage, but got type ",
757 |           storage_scalar_type,
758 |           " for argument 1 'storage'");
759 |     }
760 |     return new_with_storage(options, scalar_type, storage);
761 |   } else if (r.idx == 2) {
762 |     // NOLINTNEXTLINE(performance-no-int-to-ptr)
763 |     auto cdata = reinterpret_cast<void*>(r.toInt64(0));
764 |     return at::unsafeTensorFromTH(cdata, true);
```
- EN: Implements routines such as `new_with_sizes`, `typeIdWithDefault`, `legacy_tensor_generic_ctor_new`, `legacy_sparse_tensor_generic_ctor_new`, `device_guard` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `new_with_sizes`、`typeIdWithDefault`、`legacy_tensor_generic_ctor_new`、`legacy_sparse_tensor_generic_ctor_new`、`device_guard` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 765-860
```cpp
765 |   } else if (r.idx == 3) {
766 |     const auto& other = r.tensor(0);
767 |     // BASE_CTOR (aka torch.Tensor) is now relaxed to accept any
768 |     // dtype; previously it was "float" biased
769 |     if (ctor_or_new != CtorOrNew::BASE_CTOR) {
770 |       options = options.dtype(scalar_type);
771 |       TORCH_CHECK_TYPE(
772 |           other.options().type_equal(options),
773 |           "expected ",
774 |           options,
775 |           " (got ",
776 |           other.options(),
777 |           ")");
778 |     }
779 |     return other.alias();
780 |   } else if (r.idx == 4) {
781 |     if (ctor_or_new == CtorOrNew::CTOR || ctor_or_new == CtorOrNew::BASE_CTOR) {
782 |       TORCH_CHECK(
783 |           false,
784 |           "Legacy tensor constructor of the form torch.Tensor(tensor, device=device) "
785 |           "is not supported.  Use torch.tensor(...) or torch.as_tensor(...) instead.");
786 |     } else {
787 |       TORCH_CHECK(
788 |           false,
789 |           "Legacy tensor new of the form tensor.new(tensor, device=device) "
790 |           "is not supported.  Use torch.as_tensor(...) instead.");
791 |     }
792 |   } else if (r.idx == 5) {
793 |     PyObject* arg = r.pyobject(0);
794 |     auto deviceOptional = r.deviceOptional(1);
795 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
796 |     if (!THPSize_Check(arg) && PyTuple_GET_SIZE(args) >= 1 &&
797 |         arg == PyTuple_GET_ITEM(args, 0)) {
798 |       // new(sequence) binds to this signature but should be treated differently
799 |       // unless the sequences is a torch.Size
800 |       return legacy_new_from_sequence(
801 |           options, scalar_type, deviceOptional, r.pyobject(0));
802 |     }
803 |     return new_with_sizes(
804 |         options, scalar_type, deviceOptional, r.symintlist(0));
805 |   } else if (r.idx == 6) {
806 |     auto deviceOptional = r.deviceOptional(1);
807 |     check_legacy_ctor_device(dispatch_key, deviceOptional);
808 |     return legacy_new_from_sequence(
809 |         options, scalar_type, deviceOptional, r.pyobject(0));
810 |   }
811 |   TORCH_CHECK(false, "new(): invalid arguments");
812 | }
813 | 
814 | // Handles ONLY torch.Tensor
815 | // Unlike the legacy dtype/device specialized constructors, this one is
816 | // relaxed to accept any device/dtype input tensor (even if it doesn't
817 | // match the default)
818 | Tensor base_tensor_ctor(PyObject* args, PyObject* kwargs) {
819 |   return legacy_tensor_generic_ctor_new(
820 |       torch::tensors::get_default_dispatch_key(),
821 |       torch::tensors::get_default_scalar_type(),
822 |       args,
823 |       kwargs,
824 |       CtorOrNew::BASE_CTOR);
825 | }
826 | 
827 | // Handles calls like torch.DoubleTensor, torch.cuda.FloatTensor,
828 | // torch.sparse.FloatTensor, etc.
829 | Tensor legacy_tensor_ctor(
830 |     c10::DispatchKey dispatch_key,
831 |     at::ScalarType scalar_type,
832 |     PyObject* args,
833 |     PyObject* kwargs) {
834 |   return legacy_tensor_generic_ctor_new(
835 |       dispatch_key, scalar_type, args, kwargs, CtorOrNew::CTOR);
836 | }
837 | 
838 | // Handles tensor.new(...)
839 | Tensor legacy_tensor_new(
840 |     c10::DispatchKey dispatch_key,
841 |     at::ScalarType scalar_type,
842 |     PyObject* args,
843 |     PyObject* kwargs) {
844 |   return legacy_tensor_generic_ctor_new(
845 |       dispatch_key, scalar_type, args, kwargs, CtorOrNew::NEW);
846 | }
847 | 
848 | Tensor indexing_tensor_from_data(
849 |     c10::TensorOptions options,
850 |     at::ScalarType scalar_type,
851 |     std::optional<Device> device,
852 |     PyObject* data) {
853 |   // Specific to tensor indexing, converts an indexing list to an
854 |   // indexing tensor (type Byte or Long)
855 |   ScalarType inferred_scalar_type = infer_scalar_type(data);
856 |   if (inferred_scalar_type == ScalarType::Byte ||
857 |       inferred_scalar_type == ScalarType::Bool) {
858 |     return internal_new_from_data(
859 |         options,
860 |         inferred_scalar_type,
```
- EN: Implements routines such as `legacy_new_from_sequence`, `new_with_sizes`, `base_tensor_ctor`, `legacy_tensor_generic_ctor_new`, `legacy_tensor_ctor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `legacy_new_from_sequence`、`new_with_sizes`、`base_tensor_ctor`、`legacy_tensor_generic_ctor_new`、`legacy_tensor_ctor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 861-959
```cpp
861 |         device,
862 |         data,
863 |         /*copy_variables=*/false,
864 |         /*copy_numpy=*/false,
865 |         /*type_inference=*/false);
866 |   } else {
867 |     return internal_new_from_data(
868 |         options,
869 |         scalar_type,
870 |         device,
871 |         data,
872 |         /*copy_variables=*/false,
873 |         /*copy_numpy=*/false,
874 |         /*type_inference=*/false);
875 |   }
876 | }
877 | 
878 | class CheckSparseTensorInvariantsContext {
879 |  public:
880 |   CheckSparseTensorInvariantsContext()
881 |       : state{at::globalContext().checkSparseTensorInvariants(
882 |             /*warn_when_uninitialized=*/true)} {}
883 |   ~CheckSparseTensorInvariantsContext() {
884 |     at::globalContext().setCheckSparseTensorInvariants(state);
885 |   }
886 |   CheckSparseTensorInvariantsContext(
887 |       const CheckSparseTensorInvariantsContext&) = delete;
888 |   CheckSparseTensorInvariantsContext(CheckSparseTensorInvariantsContext&&) =
889 |       delete;
890 |   CheckSparseTensorInvariantsContext& operator=(
891 |       const CheckSparseTensorInvariantsContext&) = delete;
892 |   CheckSparseTensorInvariantsContext& operator=(
893 |       CheckSparseTensorInvariantsContext&&) = delete;
894 | 
895 |  private:
896 |   std::optional<bool> state;
897 | };
898 | 
899 | static Tensor sparse_compressed_tensor_ctor_worker(
900 |     const std::string& name,
901 |     c10::DispatchKey dispatch_key,
902 |     at::ScalarType scalar_type,
903 |     PythonArgs& r,
904 |     std::optional<c10::Layout> required_layout) {
905 |   TORCH_INTERNAL_ASSERT(!isSparseCsr(dispatchKeyToBackend(dispatch_key)));
906 |   TORCH_INTERNAL_ASSERT(!isSparse(dispatchKeyToBackend(dispatch_key)));
907 |   enum {
908 |     ARG_COMPRESSED_INDICES = 0,
909 |     ARG_PLAIN_INDICES,
910 |     ARG_VALUES,
911 |     ARG_SIZE,
912 |     ARG_TYPE,
913 |     ARG_LAYOUT,
914 |     ARG_DEVICE,
915 |     ARG_PIN_MEMORY,
916 |     ARG_REQUIRES_GRAD,
917 |     ARG_CHECK_INVARIANTS,
918 |     ARGS_COUNT
919 |   };
920 |   enum {
921 |     ARG_VALUES1 = ARG_VALUES,
922 |     ARG_TYPE1,
923 |     ARG_LAYOUT1,
924 |     ARG_DEVICE1,
925 |     ARG_PIN_MEMORY1,
926 |     ARG_REQUIRES_GRAD1,
927 |     ARG_CHECK_INVARIANTS1,
928 |     ARGS_COUNT1
929 |   };
930 | 
931 |   auto safe_get_attr_string = [](PyObject* o,
932 |                                  const char* attr_name) -> PyObject* {
933 |     // Clear error indicator if attribute does not exists.
934 |     // Otherwise subsequent Python C API calls might return bogus values.
935 |     // See https://github.com/pytorch/pytorch/issues/58520 for more details
936 |     auto rc = PyObject_GetAttrString(o, attr_name);
937 |     if (!rc) {
938 |       if (!PyErr_ExceptionMatches(PyExc_AttributeError)) {
939 |         throw python_error();
940 |       }
941 |       // Warning: a wrong attribute error may be suppressed here
942 |       PyErr_Clear();
943 |     }
944 |     return rc;
945 |   };
946 |   THPObjectPtr compressed_indices_dtype_attr(
947 |       safe_get_attr_string(r.pyobject(ARG_COMPRESSED_INDICES), "dtype"));
948 |   THPObjectPtr plain_indices_dtype_attr(
949 |       safe_get_attr_string(r.pyobject(ARG_PLAIN_INDICES), "dtype"));
950 |   at::ScalarType compressed_indices_scalar_type = compressed_indices_dtype_attr
951 |       ? reinterpret_cast<THPDtype*>(compressed_indices_dtype_attr.get())
952 |             ->scalar_type
953 |       : kInt;
954 |   at::ScalarType plain_indices_scalar_type = plain_indices_dtype_attr
955 |       ? reinterpret_cast<THPDtype*>(plain_indices_dtype_attr.get())->scalar_type
956 |       : kInt;
957 |   CheckSparseTensorInvariantsContext
958 |       restores_check_sparse_tensor_invariants_global_state{};
959 | 
```
- EN: Defines or extends data abstractions such as `CheckSparseTensorInvariantsContext` that structure the state handled by this file. Implements routines such as `internal_new_from_data`, `sparse_compressed_tensor_ctor_worker`, `python_error`, `compressed_indices_dtype_attr`, `plain_indices_dtype_attr` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 定义或扩展了 `CheckSparseTensorInvariantsContext` 等数据抽象，用来组织本文件处理的状态。 实现了 `internal_new_from_data`、`sparse_compressed_tensor_ctor_worker`、`python_error`、`compressed_indices_dtype_attr`、`plain_indices_dtype_attr` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 960-1055
```cpp
 960 |   if (r.idx == 0) {
 961 |     const bool pin_memory = r.toBool(ARG_PIN_MEMORY);
 962 |     bool type_inference = r.isNone(ARG_TYPE);
 963 |     const auto inferred_options =
 964 |         typeIdWithDefault(r, ARG_DEVICE, dispatch_key);
 965 |     const auto inferred_scalar_type =
 966 |         r.scalartypeWithDefault(ARG_TYPE, scalar_type);
 967 |     auto deviceOptional = r.deviceOptional(ARG_DEVICE);
 968 |     at::OptionalDeviceGuard device_guard(deviceOptional);
 969 |     // the global state of invariants check flag will be restored via
 970 |     // CheckSparseTensorInvariantsContext destructor
 971 |     if (auto check_invariants = r.toBoolOptional(ARG_CHECK_INVARIANTS);
 972 |         check_invariants.has_value()) {
 973 |       at::globalContext().setCheckSparseTensorInvariants(check_invariants);
 974 |     }
 975 |     Tensor values = internal_new_from_data(
 976 |         inferred_options,
 977 |         inferred_scalar_type,
 978 |         deviceOptional,
 979 |         r.pyobject(ARG_VALUES),
 980 |         /*copy_variables=*/false,
 981 |         /*copy_numpy=*/true,
 982 |         /*type_inference=*/type_inference);
 983 |     Tensor compressed_indices = internal_new_from_data(
 984 |         values.options(),
 985 |         compressed_indices_scalar_type,
 986 |         deviceOptional,
 987 |         r.pyobject(ARG_COMPRESSED_INDICES),
 988 |         /*copy_variables=*/false,
 989 |         /*copy_numpy=*/true,
 990 |         /*type_inference=*/true);
 991 |     Tensor plain_indices = internal_new_from_data(
 992 |         values.options(),
 993 |         plain_indices_scalar_type,
 994 |         deviceOptional,
 995 |         r.pyobject(ARG_PLAIN_INDICES),
 996 |         /*copy_variables=*/false,
 997 |         /*copy_numpy=*/true,
 998 |         /*type_inference=*/true);
 999 |     std::optional<c10::Layout> layout =
1000 |         (required_layout
1001 |              ? r.layoutWithDefault(ARG_LAYOUT, required_layout.value())
1002 |              : r.layoutOptional(ARG_LAYOUT));
1003 |     if (required_layout.has_value()) {
1004 |       TORCH_CHECK(
1005 |           layout.has_value() && layout == required_layout,
1006 |           name,
1007 |           ": layout must be ",
1008 |           required_layout.value(),
1009 |           " but got ",
1010 |           layout);
1011 |     }
1012 |     return at::sparse_compressed_tensor(
1013 |                compressed_indices,
1014 |                plain_indices,
1015 |                values,
1016 |                r.intlist(ARG_SIZE),
1017 |                values.options().layout(layout).pinned_memory(pin_memory))
1018 |         .set_requires_grad(r.toBool(ARG_REQUIRES_GRAD));
1019 |   } else if (r.idx == 1) {
1020 |     bool type_inference = r.isNone(ARG_TYPE1);
1021 |     const auto inferred_options =
1022 |         typeIdWithDefault(r, ARG_DEVICE1, dispatch_key);
1023 |     const auto inferred_scalar_type =
1024 |         r.scalartypeWithDefault(ARG_TYPE1, scalar_type);
1025 |     auto deviceOptional = r.deviceOptional(ARG_DEVICE1);
1026 |     at::OptionalDeviceGuard device_guard(deviceOptional);
1027 |     const bool pin_memory = r.toBool(ARG_PIN_MEMORY1);
1028 |     // the global state of invariants check flag will be restored via
1029 |     // CheckSparseTensorInvariantsContext destructor
1030 |     if (auto check_invariants = r.toBoolOptional(ARG_CHECK_INVARIANTS1);
1031 |         check_invariants.has_value()) {
1032 |       at::globalContext().setCheckSparseTensorInvariants(check_invariants);
1033 |     }
1034 |     Tensor values = internal_new_from_data(
1035 |         inferred_options,
1036 |         inferred_scalar_type,
1037 |         deviceOptional,
1038 |         r.pyobject(ARG_VALUES),
1039 |         /*copy_variables=*/false,
1040 |         /*copy_numpy=*/true,
1041 |         /*type_inference=*/type_inference);
1042 |     Tensor compressed_indices = internal_new_from_data(
1043 |         values.options(),
1044 |         compressed_indices_scalar_type,
1045 |         deviceOptional,
1046 |         r.pyobject(ARG_COMPRESSED_INDICES),
1047 |         /*copy_variables=*/false,
1048 |         /*copy_numpy=*/true,
1049 |         /*type_inference=*/true);
1050 |     Tensor plain_indices = internal_new_from_data(
1051 |         values.options(),
1052 |         plain_indices_scalar_type,
1053 |         deviceOptional,
1054 |         r.pyobject(ARG_PLAIN_INDICES),
1055 |         /*copy_variables=*/false,
```
- EN: Implements routines such as `device_guard` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `device_guard` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1056-1150
```cpp
1056 |         /*copy_numpy=*/true,
1057 |         /*type_inference=*/true);
1058 |     std::optional<c10::Layout> layout =
1059 |         (required_layout
1060 |              ? r.layoutWithDefault(ARG_LAYOUT1, required_layout.value())
1061 |              : r.layoutOptional(ARG_LAYOUT1));
1062 |     if (required_layout.has_value()) {
1063 |       TORCH_CHECK(
1064 |           layout == required_layout,
1065 |           name,
1066 |           ": layout must be ",
1067 |           required_layout.value(),
1068 |           " but got ",
1069 |           layout);
1070 |     }
1071 |     return at::sparse_compressed_tensor(
1072 |                compressed_indices,
1073 |                plain_indices,
1074 |                values,
1075 |                values.options().layout(layout).pinned_memory(pin_memory))
1076 |         .set_requires_grad(r.toBool(ARG_REQUIRES_GRAD1));
1077 |   }
1078 |   TORCH_CHECK(false, name + ": invalid arguments");
1079 | }
1080 | 
1081 | Tensor sparse_compressed_tensor_ctor(
1082 |     c10::DispatchKey dispatch_key,
1083 |     at::ScalarType scalar_type,
1084 |     PythonArgs& r) {
1085 |   std::optional<c10::Layout> required_layout{};
1086 |   return sparse_compressed_tensor_ctor_worker(
1087 |       "sparse_compressed_tensor",
1088 |       dispatch_key,
1089 |       scalar_type,
1090 |       r,
1091 |       required_layout);
1092 | }
1093 | 
1094 | Tensor sparse_csr_tensor_ctor(
1095 |     c10::DispatchKey dispatch_key,
1096 |     at::ScalarType scalar_type,
1097 |     PythonArgs& r) {
1098 |   std::optional<c10::Layout> required_layout(c10::Layout::SparseCsr);
1099 |   return sparse_compressed_tensor_ctor_worker(
1100 |       "sparse_csr_tensor", dispatch_key, scalar_type, r, required_layout);
1101 | }
1102 | 
1103 | Tensor sparse_csc_tensor_ctor(
1104 |     c10::DispatchKey dispatch_key,
1105 |     at::ScalarType scalar_type,
1106 |     PythonArgs& r) {
1107 |   std::optional<c10::Layout> required_layout(c10::Layout::SparseCsc);
1108 |   return sparse_compressed_tensor_ctor_worker(
1109 |       "sparse_csc_tensor", dispatch_key, scalar_type, r, required_layout);
1110 | }
1111 | 
1112 | Tensor sparse_bsr_tensor_ctor(
1113 |     c10::DispatchKey dispatch_key,
1114 |     at::ScalarType scalar_type,
1115 |     PythonArgs& r) {
1116 |   std::optional<c10::Layout> required_layout(c10::Layout::SparseBsr);
1117 |   return sparse_compressed_tensor_ctor_worker(
1118 |       "sparse_bsr_tensor", dispatch_key, scalar_type, r, required_layout);
1119 | }
1120 | 
1121 | Tensor sparse_bsc_tensor_ctor(
1122 |     c10::DispatchKey dispatch_key,
1123 |     at::ScalarType scalar_type,
1124 |     PythonArgs& r) {
1125 |   std::optional<c10::Layout> required_layout(c10::Layout::SparseBsc);
1126 |   return sparse_compressed_tensor_ctor_worker(
1127 |       "sparse_bsc_tensor", dispatch_key, scalar_type, r, required_layout);
1128 | }
1129 | 
1130 | // Note [Ensuring sparse values and indices match devices]
1131 | // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
1132 | // In all places where we construct indices, we read out options from values
1133 | // (rather than use inferred_options).  Why?  This handles the case when
1134 | // values is a CUDA tensor, but indices is a non-Tensor value (and the device
1135 | // argument is not set).  Example:
1136 | //
1137 | //  torch.sparse_coo_tensor(([0, 1],), self.empty(2, 0).cuda(), (4, 0))
1138 | //
1139 | // Sparse tensors require both indices and values to live on the same device.
1140 | // If values lives on CUDA, we can infer where the indices should live, and
1141 | // should accept even ordinary index sequences (and just make sure we write them
1142 | // into the correct device).  values is the ONLY way we know that the index
1143 | // tensor should go to CUDA, so we have to get the information in somehow.
1144 | //
1145 | // This code is kind of jank.  For one, the dtype in options is silently ignored
1146 | // by internal_new_from_data.  Also, in classic janky code style, it used to
1147 | // not work quite right: if values lives on "cuda:1", before all we said was
1148 | // "this needs to be CUDA" and indices would be allocated on the wrong tensor.
1149 | // Options is more right and gets this correct.
1150 | 
```
- EN: Implements routines such as `sparse_compressed_tensor_ctor`, `sparse_compressed_tensor_ctor_worker`, `sparse_csr_tensor_ctor`, `required_layout`, `sparse_csc_tensor_ctor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `sparse_compressed_tensor_ctor`、`sparse_compressed_tensor_ctor_worker`、`sparse_csr_tensor_ctor`、`required_layout`、`sparse_csc_tensor_ctor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1151-1242
```cpp
1151 | Tensor sparse_coo_tensor_ctor(
1152 |     c10::DispatchKey dispatch_key,
1153 |     at::ScalarType scalar_type,
1154 |     PythonArgs& r) {
1155 |   TORCH_INTERNAL_ASSERT(!isSparse(dispatchKeyToBackend(dispatch_key)));
1156 |   TORCH_INTERNAL_ASSERT(!isSparseCsr(dispatchKeyToBackend(dispatch_key)));
1157 |   enum {
1158 |     ARG_INDICES = 0,
1159 |     ARG_VALUES,
1160 |     ARG_TYPE,
1161 |     ARG_DEVICE,
1162 |     ARG_PIN_MEMORY,
1163 |     ARG_REQUIRES_GRAD,
1164 |     ARG_CHECK_INVARIANTS,
1165 |     ARG_IS_COALESCED,
1166 |     ARGS_COUNT
1167 |   };
1168 |   enum {
1169 |     ARG_INDICES1 = 0,
1170 |     ARG_VALUES1,
1171 |     ARG_SIZE1,
1172 |     ARG_TYPE1,
1173 |     ARG_DEVICE1,
1174 |     ARG_PIN_MEMORY1,
1175 |     ARG_REQUIRES_GRAD1,
1176 |     ARG_CHECK_INVARIANTS1,
1177 |     ARG_IS_COALESCED1,
1178 |     ARGS_COUNT1
1179 |   };
1180 |   enum {
1181 |     ARG_SIZE2 = 0,
1182 |     ARG_TYPE2,
1183 |     ARG_DEVICE2,
1184 |     ARG_REQUIRES_GRAD2,
1185 |     ARG_CHECK_INVARIANTS2,
1186 |     ARGS_COUNT2
1187 |   };
1188 | 
1189 |   CheckSparseTensorInvariantsContext
1190 |       restores_check_sparse_tensor_invariants_global_state{};
1191 |   if (r.idx == 0) {
1192 |     bool pin_memory = r.toBool(ARG_PIN_MEMORY);
1193 |     bool type_inference = r.isNone(ARG_TYPE);
1194 |     const auto inferred_options =
1195 |         typeIdWithDefault(r, ARG_DEVICE, dispatch_key);
1196 |     const auto inferred_scalar_type =
1197 |         r.scalartypeWithDefault(ARG_TYPE, scalar_type);
1198 |     auto deviceOptional = r.deviceOptional(ARG_DEVICE);
1199 |     at::OptionalDeviceGuard device_guard(deviceOptional);
1200 |     if (auto check_invariants = r.toBoolOptional(ARG_CHECK_INVARIANTS);
1201 |         check_invariants.has_value()) {
1202 |       at::globalContext().setCheckSparseTensorInvariants(check_invariants);
1203 |     }
1204 | 
1205 |     // if no dtype provided, infer type based on value type.
1206 |     Tensor values = internal_new_from_data(
1207 |         inferred_options,
1208 |         inferred_scalar_type,
1209 |         deviceOptional,
1210 |         r.pyobject(ARG_VALUES),
1211 |         /*copy_variables=*/false,
1212 |         /*copy_numpy=*/true,
1213 |         /*type_inference=*/type_inference);
1214 |     // See Note [Ensuring sparse values and indices match devices]
1215 |     Tensor indices = internal_new_from_data(
1216 |         values.options(),
1217 |         kLong,
1218 |         deviceOptional,
1219 |         r.pyobject(ARG_INDICES),
1220 |         /*copy_variables=*/false,
1221 |         /*copy_numpy=*/true,
1222 |         /*type_inference=*/false);
1223 |     return at::sparse_coo_tensor(
1224 |                indices,
1225 |                values,
1226 |                values.options().layout(at::kSparse).pinned_memory(pin_memory),
1227 |                r.toBoolOptional(ARG_IS_COALESCED))
1228 |         .set_requires_grad(r.toBool(ARG_REQUIRES_GRAD));
1229 |   } else if (r.idx == 1) {
1230 |     bool pin_memory = r.toBool(ARG_PIN_MEMORY1);
1231 |     bool type_inference = r.isNone(ARG_TYPE1);
1232 |     const auto inferred_options =
1233 |         typeIdWithDefault(r, ARG_DEVICE1, dispatch_key);
1234 |     const auto inferred_scalar_type =
1235 |         r.scalartypeWithDefault(ARG_TYPE1, scalar_type);
1236 |     auto deviceOptional = r.deviceOptional(ARG_DEVICE1);
1237 |     at::OptionalDeviceGuard device_guard(deviceOptional);
1238 |     if (auto check_invariants = r.toBoolOptional(ARG_CHECK_INVARIANTS1);
1239 |         check_invariants.has_value()) {
1240 |       at::globalContext().setCheckSparseTensorInvariants(check_invariants);
1241 |     }
1242 | 
```
- EN: Implements routines such as `sparse_coo_tensor_ctor`, `device_guard` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `sparse_coo_tensor_ctor`、`device_guard` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 1243-1335
```cpp
1243 |     Tensor values = internal_new_from_data(
1244 |         inferred_options,
1245 |         inferred_scalar_type,
1246 |         deviceOptional,
1247 |         r.pyobject(ARG_VALUES1),
1248 |         /*copy_variables=*/false,
1249 |         /*copy_numpy=*/true,
1250 |         /*type_inference=*/type_inference);
1251 |     // See Note [Ensuring sparse values and indices match devices]
1252 |     Tensor indices = internal_new_from_data(
1253 |         values.options(),
1254 |         kLong,
1255 |         deviceOptional,
1256 |         r.pyobject(ARG_INDICES1),
1257 |         /*copy_variables=*/false,
1258 |         /*copy_numpy=*/true,
1259 |         /*type_inference=*/false);
1260 |     return at::sparse_coo_tensor(
1261 |                indices,
1262 |                values,
1263 |                r.intlist(ARG_SIZE1),
1264 |                values.options().layout(at::kSparse).pinned_memory(pin_memory),
1265 |                r.toBoolOptional(ARG_IS_COALESCED1))
1266 |         .set_requires_grad(r.toBool(ARG_REQUIRES_GRAD1));
1267 |   } else if (r.idx == 2) {
1268 |     const auto inferred_options =
1269 |         typeIdWithDefault(r, ARG_DEVICE2, dispatch_key);
1270 |     const auto inferred_scalar_type =
1271 |         r.scalartypeWithDefault(ARG_TYPE2, scalar_type);
1272 |     at::OptionalDeviceGuard device_guard(r.deviceOptional(ARG_DEVICE2));
1273 |     if (auto check_invariants = r.toBoolOptional(ARG_CHECK_INVARIANTS2);
1274 |         check_invariants.has_value()) {
1275 |       at::globalContext().setCheckSparseTensorInvariants(check_invariants);
1276 |     }
1277 |     return at::sparse_coo_tensor(
1278 |                r.intlist(ARG_SIZE2),
1279 |                inferred_options.dtype(inferred_scalar_type).layout(at::kSparse))
1280 |         .set_requires_grad(r.toBool(ARG_REQUIRES_GRAD2));
1281 |   }
1282 |   TORCH_CHECK(false, "sparse_coo_tensor(): invalid arguments");
1283 | }
1284 | 
1285 | void _validate_sparse_coo_tensor_args(
1286 |     c10::DispatchKey dispatch_key,
1287 |     at::ScalarType scalar_type,
1288 |     PyObject* args,
1289 |     PyObject* kwargs) {
1290 |   auto options = dispatchKeyToTensorOptions(dispatch_key);
1291 |   static PythonArgParser parser({
1292 |       "_validate_sparse_coo_tensor(PyObject* indices, PyObject* values, IntArrayRef size)",
1293 |   });
1294 | 
1295 |   ParsedArgs<3> parsed_args;
1296 |   auto r = parser.parse(args, kwargs, parsed_args);
1297 |   Tensor values = internal_new_from_data(
1298 |       options,
1299 |       scalar_type,
1300 |       std::nullopt,
1301 |       r.pyobject(1),
1302 |       /*copy_variables=*/false,
1303 |       /*copy_numpy=*/true,
1304 |       /*type_inference=*/true);
1305 |   // See Note [Ensuring sparse values and indices match devices]
1306 |   Tensor indices = internal_new_from_data(
1307 |       values.options(),
1308 |       kLong,
1309 |       std::nullopt,
1310 |       r.pyobject(0),
1311 |       /*copy_variables=*/false,
1312 |       /*copy_numpy=*/true,
1313 |       /*type_inference=*/false);
1314 |   at::native::_validate_sparse_coo_tensor_args(indices, values, r.intlist(2));
1315 | }
1316 | 
1317 | void _validate_sparse_compressed_tensor_args(
1318 |     c10::DispatchKey dispatch_key,
1319 |     at::ScalarType scalar_type,
1320 |     PyObject* args,
1321 |     PyObject* kwargs) {
1322 |   auto options = dispatchKeyToTensorOptions(dispatch_key);
1323 |   enum {
1324 |     ARG_COMPRESSED_INDICES = 0,
1325 |     ARG_PLAIN_INDICES,
1326 |     ARG_VALUES,
1327 |     ARG_SIZE,
1328 |     ARG_LAYOUT,
1329 |     ARGS_COUNT
1330 |   };
1331 | 
1332 |   const std::string signature =
1333 |       "_validate_sparse_compressed_tensor(PyObject* compressed_indices, PyObject* plain_indices, PyObject* values, IntArrayRef size, Layout layout)";
1334 |   static PythonArgParser parser({signature});
1335 | 
```
- EN: Implements routines such as `device_guard`, `_validate_sparse_coo_tensor_args`, `_validate_sparse_compressed_tensor_args` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `device_guard`、`_validate_sparse_coo_tensor_args`、`_validate_sparse_compressed_tensor_args` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1336-1437
```cpp
1336 |   ParsedArgs<ARGS_COUNT> parsed_args;
1337 |   auto r = parser.parse(args, kwargs, parsed_args);
1338 |   Tensor values = internal_new_from_data(
1339 |       options,
1340 |       scalar_type,
1341 |       std::nullopt,
1342 |       r.pyobject(ARG_VALUES),
1343 |       /*copy_variables=*/false,
1344 |       /*copy_numpy=*/true,
1345 |       /*type_inference=*/true);
1346 |   // See Note [Ensuring sparse values and indices match devices]
1347 |   Tensor compressed_indices = internal_new_from_data(
1348 |       values.options(),
1349 |       kInt,
1350 |       std::nullopt,
1351 |       r.pyobject(ARG_COMPRESSED_INDICES),
1352 |       /*copy_variables=*/false,
1353 |       /*copy_numpy=*/true,
1354 |       /*type_inference=*/true);
1355 |   Tensor plain_indices = internal_new_from_data(
1356 |       values.options(),
1357 |       kInt,
1358 |       std::nullopt,
1359 |       r.pyobject(ARG_PLAIN_INDICES),
1360 |       /*copy_variables=*/false,
1361 |       /*copy_numpy=*/true,
1362 |       /*type_inference=*/true);
1363 |   at::native::_validate_sparse_compressed_tensor_args(
1364 |       compressed_indices,
1365 |       plain_indices,
1366 |       values,
1367 |       r.intlist(ARG_SIZE),
1368 |       r.layout(ARG_LAYOUT));
1369 | }
1370 | 
1371 | template <c10::Layout required_layout>
1372 | static void _validate_sparse_compressed_tensor_args_template(
1373 |     c10::DispatchKey dispatch_key,
1374 |     at::ScalarType scalar_type,
1375 |     PyObject* args,
1376 |     PyObject* kwargs) {
1377 |   auto options = dispatchKeyToTensorOptions(dispatch_key);
1378 |   enum {
1379 |     ARG_COMPRESSED_INDICES = 0,
1380 |     ARG_PLAIN_INDICES,
1381 |     ARG_VALUES,
1382 |     ARG_SIZE,
1383 |     ARGS_COUNT
1384 |   };
1385 |   constexpr const char* sig = [] {
1386 |     if constexpr (required_layout == c10::Layout::SparseCsr)
1387 |       return "_validate_sparse_csr_tensor(PyObject* crow_indices, PyObject* col_indices, PyObject* values, IntArrayRef size)";
1388 |     else if constexpr (required_layout == c10::Layout::SparseCsc)
1389 |       return "_validate_sparse_csc_tensor(PyObject* ccol_indices, PyObject* row_indices, PyObject* values, IntArrayRef size)";
1390 |     else if constexpr (required_layout == c10::Layout::SparseBsr)
1391 |       return "_validate_sparse_bsr_tensor(PyObject* crow_indices, PyObject* col_indices, PyObject* values, IntArrayRef size)";
1392 |     else if constexpr (required_layout == c10::Layout::SparseBsc)
1393 |       return "_validate_sparse_bsc_tensor(PyObject* ccol_indices, PyObject* row_indices, PyObject* values, IntArrayRef size)";
1394 |   }();
1395 |   static PythonArgParser parser({sig});
1396 | 
1397 |   ParsedArgs<ARGS_COUNT> parsed_args;
1398 |   auto r = parser.parse(args, kwargs, parsed_args);
1399 |   Tensor values = internal_new_from_data(
1400 |       options,
1401 |       scalar_type,
1402 |       std::nullopt,
1403 |       r.pyobject(ARG_VALUES),
1404 |       /*copy_variables=*/false,
1405 |       /*copy_numpy=*/true,
1406 |       /*type_inference=*/true);
1407 |   // See Note [Ensuring sparse values and indices match devices]
1408 |   Tensor compressed_indices = internal_new_from_data(
1409 |       values.options(),
1410 |       kInt,
1411 |       std::nullopt,
1412 |       r.pyobject(ARG_COMPRESSED_INDICES),
1413 |       /*copy_variables=*/false,
1414 |       /*copy_numpy=*/true,
1415 |       /*type_inference=*/true);
1416 |   Tensor plain_indices = internal_new_from_data(
1417 |       values.options(),
1418 |       kInt,
1419 |       std::nullopt,
1420 |       r.pyobject(ARG_PLAIN_INDICES),
1421 |       /*copy_variables=*/false,
1422 |       /*copy_numpy=*/true,
1423 |       /*type_inference=*/true);
1424 | 
1425 |   at::native::_validate_sparse_compressed_tensor_args(
1426 |       compressed_indices, plain_indices, values, r.intlist(3), required_layout);
1427 | }
1428 | 
1429 | void _validate_sparse_csr_tensor_args(
1430 |     c10::DispatchKey dispatch_key,
1431 |     at::ScalarType scalar_type,
1432 |     PyObject* args,
1433 |     PyObject* kwargs) {
1434 |   _validate_sparse_compressed_tensor_args_template<c10::Layout::SparseCsr>(
1435 |       dispatch_key, scalar_type, args, kwargs);
1436 | }
1437 | 
```
- EN: Implements routines such as `_validate_sparse_compressed_tensor_args_template`, `_validate_sparse_csr_tensor_args` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `_validate_sparse_compressed_tensor_args_template`、`_validate_sparse_csr_tensor_args` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 1438-1532
```cpp
1438 | void _validate_sparse_csc_tensor_args(
1439 |     c10::DispatchKey dispatch_key,
1440 |     at::ScalarType scalar_type,
1441 |     PyObject* args,
1442 |     PyObject* kwargs) {
1443 |   _validate_sparse_compressed_tensor_args_template<c10::Layout::SparseCsc>(
1444 |       dispatch_key, scalar_type, args, kwargs);
1445 | }
1446 | 
1447 | void _validate_sparse_bsr_tensor_args(
1448 |     c10::DispatchKey dispatch_key,
1449 |     at::ScalarType scalar_type,
1450 |     PyObject* args,
1451 |     PyObject* kwargs) {
1452 |   _validate_sparse_compressed_tensor_args_template<c10::Layout::SparseBsr>(
1453 |       dispatch_key, scalar_type, args, kwargs);
1454 | }
1455 | 
1456 | void _validate_sparse_bsc_tensor_args(
1457 |     c10::DispatchKey dispatch_key,
1458 |     at::ScalarType scalar_type,
1459 |     PyObject* args,
1460 |     PyObject* kwargs) {
1461 |   _validate_sparse_compressed_tensor_args_template<c10::Layout::SparseBsc>(
1462 |       dispatch_key, scalar_type, args, kwargs);
1463 | }
1464 | 
1465 | Tensor tensor_ctor(
1466 |     c10::DispatchKey dispatch_key,
1467 |     at::ScalarType scalar_type,
1468 |     PythonArgs& r) {
1469 |   if (r.idx == 0) {
1470 |     PyObject* data = r.pyobject(0);
1471 |     if (THPVariable_Check(data)) {
1472 |       auto ret = PyErr_WarnEx(
1473 |           PyExc_UserWarning,
1474 |           "To copy construct from a tensor, it is recommended to use sourceTensor.detach().clone() "
1475 |           "or sourceTensor.detach().clone().requires_grad_(True), rather than torch.tensor(sourceTensor).",
1476 |           1);
1477 |       if (ret != 0)
1478 |         throw python_error();
1479 |     }
1480 | 
1481 |     bool type_inference = r.isNone(1);
1482 |     bool pin_memory = r.toBool(3);
1483 |     bool args_requires_grad = r.toBool(4);
1484 |     auto new_tensor = internal_new_from_data(
1485 |         typeIdWithDefault(r, 2, dispatch_key),
1486 |         r.scalartypeWithDefault(1, scalar_type),
1487 |         r.deviceOptional(2),
1488 |         data,
1489 |         /*copy_variables=*/true,
1490 |         /*copy_numpy=*/true,
1491 |         /*type_inference=*/type_inference,
1492 |         pin_memory);
1493 |     auto names = r.toDimnameListOptional(5);
1494 |     if (names) {
1495 |       at::namedinference::propagate_names_if_nonempty(
1496 |           new_tensor, *names, /*validate_names=*/true);
1497 |     }
1498 |     new_tensor.detach_(); // ensure new_tensor a leaf node
1499 |     new_tensor.set_requires_grad(args_requires_grad);
1500 |     return new_tensor;
1501 |   }
1502 |   TORCH_CHECK(false, "tensor(): invalid arguments");
1503 | }
1504 | 
1505 | Tensor as_tensor(
1506 |     c10::DispatchKey dispatch_key,
1507 |     at::ScalarType scalar_type,
1508 |     PythonArgs& r) {
1509 |   // TODO: add requires_grad once we decide on semantics for sharing data.
1510 |   if (r.idx == 0) {
1511 |     bool type_inference = r.isNone(1);
1512 |     return internal_new_from_data(
1513 |         typeIdWithDefault(r, 2, dispatch_key),
1514 |         r.scalartypeWithDefault(1, scalar_type),
1515 |         r.deviceOptional(2),
1516 |         r.pyobject(0),
1517 |         /*copy_variables=*/false,
1518 |         /*copy_numpy=*/false,
1519 |         /*type_inference=*/type_inference);
1520 |   }
1521 |   TORCH_CHECK(false, "tensor(): invalid arguments");
1522 | }
1523 | 
1524 | Tensor new_tensor(
1525 |     c10::DispatchKey dispatch_key,
1526 |     at::ScalarType scalar_type,
1527 |     PyObject* args,
1528 |     PyObject* kwargs) {
1529 |   static PythonArgParser parser({
1530 |       "new_tensor(PyObject* data, *, ScalarType dtype=None, Device? device=None, bool requires_grad=False)",
1531 |   });
1532 | 
```
- EN: Implements routines such as `_validate_sparse_csc_tensor_args`, `_validate_sparse_bsr_tensor_args`, `_validate_sparse_bsc_tensor_args`, `tensor_ctor`, `python_error` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `_validate_sparse_csc_tensor_args`、`_validate_sparse_bsr_tensor_args`、`_validate_sparse_bsc_tensor_args`、`tensor_ctor`、`python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1533-1624
```cpp
1533 |   ParsedArgs<4> parsed_args;
1534 |   auto r = parser.parse(args, kwargs, parsed_args);
1535 |   if (r.idx == 0) {
1536 |     PyObject* data = r.pyobject(0);
1537 |     if (THPVariable_Check(data)) {
1538 |       auto ret = PyErr_WarnEx(
1539 |           PyExc_UserWarning,
1540 |           "To copy construct from a tensor, it is recommended to use sourceTensor.detach().clone() "
1541 |           "or sourceTensor.detach().clone().requires_grad_(True), rather than tensor.new_tensor(sourceTensor).",
1542 |           1);
1543 |       if (ret != 0)
1544 |         throw python_error();
1545 |     }
1546 | 
1547 |     bool args_requires_grad = r.toBool(3);
1548 |     auto deviceOptional =
1549 |         device_or_from_dispatch_key(r.deviceOptional(2), dispatch_key);
1550 |     auto new_tensor = new_from_data_copy(
1551 |         typeIdWithDefault(r, 2, dispatch_key),
1552 |         r.scalartypeWithDefault(1, scalar_type),
1553 |         deviceOptional,
1554 |         data);
1555 |     new_tensor.detach_(); // ensure new_tensor a leaf node
1556 |     new_tensor.set_requires_grad(args_requires_grad);
1557 |     return new_tensor;
1558 |   }
1559 |   TORCH_CHECK(false, "new_tensor(): invalid arguments");
1560 | }
1561 | 
1562 | Tensor tensor_frombuffer(
1563 |     PyObject* buffer,
1564 |     ScalarType dtype,
1565 |     int64_t count,
1566 |     int64_t offset,
1567 |     bool requires_grad) {
1568 |   auto elsize = at::elementSize(dtype);
1569 |   size_t actual_count = 0;
1570 | 
1571 |   Py_buffer view;
1572 |   if (PyObject_GetBuffer(buffer, &view, PyBUF_WRITABLE) < 0) {
1573 |     TORCH_CHECK(
1574 |         PyObject_GetBuffer(buffer, &view, PyBUF_SIMPLE) >= 0,
1575 |         "could not retrieve buffer from object");
1576 |     TORCH_WARN_ONCE(
1577 |         "The given buffer is not writable, and PyTorch does "
1578 |         "not support non-writable tensors. This means you can write to the "
1579 |         "underlying (supposedly non-writable) buffer using the tensor. "
1580 |         "You may want to copy the buffer to protect its data or make it writable "
1581 |         "before converting it to a tensor. This type of warning will be "
1582 |         "suppressed for the rest of this program.");
1583 |     PyErr_Clear();
1584 |   }
1585 | 
1586 |   Py_INCREF(view.obj);
1587 |   THPObjectPtr obj(view.obj);
1588 | 
1589 |   auto len = view.len;
1590 |   auto buf = view.buf;
1591 |   PyBuffer_Release(&view);
1592 | 
1593 |   TORCH_CHECK_VALUE(
1594 |       len > 0 && count != 0,
1595 |       "both buffer length (",
1596 |       len,
1597 |       ") and count (",
1598 |       count,
1599 |       ") must not be 0");
1600 |   TORCH_CHECK_VALUE(
1601 |       offset >= 0 && offset < len,
1602 |       "offset (",
1603 |       offset,
1604 |       " bytes) must be non-negative and no greater than "
1605 |       "buffer length (",
1606 |       len,
1607 |       " bytes) minus 1");
1608 |   TORCH_CHECK_VALUE(
1609 |       count > 0 || (len - offset) % elsize == 0,
1610 |       "buffer length (",
1611 |       len - offset,
1612 |       " bytes) after offset (",
1613 |       offset,
1614 |       " bytes) "
1615 |       "must be a multiple of element size (",
1616 |       elsize,
1617 |       ")");
1618 | 
1619 |   if (count < 0) {
1620 |     actual_count = (len - offset) / elsize;
1621 |   } else {
1622 |     actual_count = static_cast<size_t>(count);
1623 |   }
1624 | 
```
- EN: Implements routines such as `python_error`, `tensor_frombuffer`, `obj` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `python_error`、`tensor_frombuffer`、`obj` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1625-1722
```cpp
1625 |   TORCH_CHECK_VALUE(
1626 |       static_cast<size_t>(offset) + actual_count * elsize <=
1627 |           static_cast<size_t>(len),
1628 |       "requested buffer length (",
1629 |       actual_count,
1630 |       " * ",
1631 |       elsize,
1632 |       " bytes) "
1633 |       "after offset (",
1634 |       offset,
1635 |       " bytes) must not be greater than actual "
1636 |       "buffer length (",
1637 |       len,
1638 |       " bytes)");
1639 | 
1640 |   auto offset_buf = static_cast<char*>(buf) + offset;
1641 |   auto options = TensorOptions().dtype(dtype).device(c10::kCPU);
1642 | 
1643 |   auto tensor = at::for_blob(offset_buf, static_cast<int64_t>(actual_count))
1644 |                     .options(options)
1645 |                     .deleter([obj = obj.release()](void*) {
1646 |                       pybind11::gil_scoped_acquire gil;
1647 |                       Py_DECREF(obj);
1648 |                     })
1649 |                     .make_tensor();
1650 |   tensor.set_requires_grad(requires_grad);
1651 |   return tensor;
1652 | }
1653 | 
1654 | namespace {
1655 | 
1656 | template <class T>
1657 | at::Tensor tensor_fromDLPackImpl(PyObject* data, T* tensor) {
1658 |   // HACK: Ensure that we hold the GIL here just in case the
1659 |   // managed tensor originating from a buggy NumPy build.
1660 |   bool is_numpy_dlpack_deleter_bugged =
1661 |       torch::utils::is_numpy_dlpack_deleter_bugged();
1662 | 
1663 |   auto deleter_maybe_gil = [=](void*) {
1664 |     if (tensor->deleter) {
1665 |       if (is_numpy_dlpack_deleter_bugged) {
1666 |         pybind11::gil_scoped_acquire gil;
1667 |         tensor->deleter(tensor);
1668 |       } else {
1669 |         tensor->deleter(tensor);
1670 |       }
1671 |     }
1672 |   };
1673 | 
1674 |   // atensor steals the ownership of the underlying storage. It also passes a
1675 |   // destructor function that will be called when the underlying storage goes
1676 |   // out of scope. When the destructor is called, the dlMTensor is destructed
1677 |   // too.
1678 |   auto atensor =
1679 |       at::DLPackTraits<T>::fromDLPack(tensor, std::move(deleter_maybe_gil));
1680 | 
1681 |   // Make sure this capsule will never be used again.
1682 |   PyCapsule_SetName(data, at::DLPackTraits<T>::used);
1683 | 
1684 |   // It is possible that the call to at::fromDLPack is the very first
1685 |   // call to create a Tensor in PyTorch. If so, then _lazy_init has
1686 |   // not been called, and the attempt to call createPyObject will fail
1687 |   // because cuda ATen types have not been registered in Python yet.
1688 |   // so if we have a cuda tensor, then we need to make sure
1689 |   // we have called _lazy_init here
1690 |   maybe_initialize_device(atensor.device());
1691 |   return atensor;
1692 | }
1693 | 
1694 | // Check whether `data` is a valid DLPack capsule.
1695 | // This function checks for the versioned and unversioned forms.
1696 | bool isValidDLPackCapsule(PyObject* data) {
1697 |   return PyCapsule_IsValid(
1698 |              data, at::DLPackTraits<DLManagedTensorVersioned>::capsule) ||
1699 |       PyCapsule_IsValid(data, at::DLPackTraits<DLManagedTensor>::capsule);
1700 | }
1701 | 
1702 | } // namespace
1703 | 
1704 | Tensor tensor_fromDLPack(PyObject* data) {
1705 |   const char* bad_capsule =
1706 |       "from_dlpack received an invalid capsule. "
1707 |       "Note that DLTensor capsules can be consumed only once, "
1708 |       "so you might have already constructed a tensor from it once.";
1709 | 
1710 |   if (PyCapsule_IsValid(
1711 |           data, at::DLPackTraits<DLManagedTensorVersioned>::capsule)) {
1712 |     auto versioned = (DLManagedTensorVersioned*)PyCapsule_GetPointer(
1713 |         data, at::DLPackTraits<DLManagedTensorVersioned>::capsule);
1714 | 
1715 |     TORCH_CHECK(versioned != nullptr, bad_capsule);
1716 |     TORCH_CHECK(
1717 |         versioned->version.major <= DLPACK_MAJOR_VERSION,
1718 |         "unsupported DLPack capsule major version: ",
1719 |         versioned->version.major,
1720 |         ". Maximum supported version: ",
1721 |         DLPACK_MAJOR_VERSION);
1722 | 
```
- EN: Defines or extends data abstractions such as `T` that structure the state handled by this file. Implements routines such as `tensor_fromDLPackImpl`, `isValidDLPackCapsule`, `PyCapsule_IsValid`, `tensor_fromDLPack` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 定义或扩展了 `T` 等数据抽象，用来组织本文件处理的状态。 实现了 `tensor_fromDLPackImpl`、`isValidDLPackCapsule`、`PyCapsule_IsValid`、`tensor_fromDLPack` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 1723-1820
```cpp
1723 |     return tensor_fromDLPackImpl(data, versioned);
1724 |   } else {
1725 |     auto managed = (DLManagedTensor*)PyCapsule_GetPointer(
1726 |         data, at::DLPackTraits<DLManagedTensor>::capsule);
1727 |     TORCH_CHECK(managed != nullptr, bad_capsule);
1728 |     return tensor_fromDLPackImpl(data, managed);
1729 |   }
1730 | }
1731 | 
1732 | Tensor asarray(
1733 |     PyObject* obj,
1734 |     std::optional<ScalarType> dtype,
1735 |     std::optional<Device> device,
1736 |     std::optional<bool> copy,
1737 |     std::optional<bool> requires_grad) {
1738 |   Tensor tensor;
1739 | 
1740 |   bool force_copy = copy.value_or(false);
1741 |   bool force_alias = !copy.value_or(true);
1742 |   bool should_warn_numpy_not_writable = false;
1743 | 
1744 |   // Used when:
1745 |   // 1. 'obj' implements the buffer protocol and no type is given.
1746 |   // 2. creating a new tensor from a Python sequence.
1747 |   auto dtype_unwrapped =
1748 |       dtype.value_or(torch::tensors::get_default_scalar_type());
1749 | 
1750 |   // Check whether 'obj' is a 'Tensor'
1751 |   if (THPVariable_Check(obj)) {
1752 |     tensor = THPVariable_Unpack(obj);
1753 |   }
1754 |   bool return_requires_grad =
1755 |       requires_grad.value_or(tensor.defined() ? tensor.requires_grad() : false);
1756 |   if (return_requires_grad && !requires_grad) {
1757 |     TORCH_WARN_ONCE(
1758 |         "torch.asarray: unspecified requires_grad now defaults to obj.requires_grad "
1759 |         "instead of False. Pass requires_grad=False explicitly to get the old behavior "
1760 |         "and silence this warning.")
1761 |   }
1762 | 
1763 | #ifdef USE_NUMPY
1764 |   if (!tensor.defined() && is_numpy_available()) {
1765 |     // Check whether 'obj' is a NumPy Array or Scalar.
1766 |     bool is_numpy_array = PyArray_Check(obj);
1767 |     bool is_numpy_scalar = PyArray_CheckScalar(obj);
1768 | 
1769 |     if (is_numpy_array || is_numpy_scalar) {
1770 |       THPObjectPtr ptr;
1771 |       auto arr = obj;
1772 | 
1773 |       // PyArray_CheckScalar is true for both scalars and 0-dim arrays, per
1774 |       // https://numpy.org/devdocs/reference/c-api/array.html#c.PyArray_CheckScalar
1775 |       // But for 0-dim arrays no `PyArray_FromScalar` call is needed
1776 |       if (is_numpy_scalar && !is_numpy_array) {
1777 |         TORCH_CHECK(
1778 |             !force_alias,
1779 |             "can't alias NumPy scalars. ",
1780 |             "Either remove copy=False or transform it in a ndarray. ")
1781 | 
1782 |         ptr = PyArray_FromScalar(obj, nullptr);
1783 |         arr = ptr.get();
1784 |       }
1785 | 
1786 |       tensor = tensor_from_numpy(arr, /*warn_if_not_writeable=*/false);
1787 |       should_warn_numpy_not_writable =
1788 |           !PyArray_ISWRITEABLE((PyArrayObject*)arr);
1789 | 
1790 |       if (is_numpy_scalar) {
1791 |         // Uses a newly cloned storage, instead of the shared one.
1792 |         // The THPObjectPtr will delete the previous storage in the
1793 |         // end of the previous scope.
1794 |         tensor = tensor.clone();
1795 | 
1796 |         // No need to clone again, later.
1797 |         force_copy = false;
1798 |       }
1799 |     }
1800 |   }
1801 | #endif
1802 | 
1803 |   // Check whether 'obj' is a 'DLPack' capsule
1804 |   if (!tensor.defined() && isValidDLPackCapsule(obj)) {
1805 |     tensor = tensor_fromDLPack(obj);
1806 |   }
1807 | 
1808 |   // Check whether 'obj' implements the buffer protocol
1809 |   if (!tensor.defined() && PyObject_CheckBuffer(obj) != 0) {
1810 |     tensor =
1811 |         tensor_frombuffer(obj, dtype_unwrapped, -1, 0, return_requires_grad);
1812 |   }
1813 | 
1814 |   if (tensor.defined()) {
1815 |     // Given an aliasable tensor, should we copy it?
1816 |     bool wrong_device = device.has_value() && device.value() != tensor.device();
1817 |     bool wrong_dtype =
1818 |         dtype.has_value() && dtype.value() != tensor.scalar_type();
1819 |     bool needs_copying = !copy.has_value() && (wrong_device || wrong_dtype);
1820 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `tensor_fromDLPackImpl`, `asarray` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `tensor_fromDLPackImpl`、`asarray` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1821-1897
```cpp
1821 |     // Given a defined tensor, we copy it if either we have to (copy=True) or
1822 |     // if we need to (copy=None) because of mismatched device or dtype.
1823 |     if (force_copy || needs_copying) {
1824 |       if (wrong_device || wrong_dtype) {
1825 |         tensor = tensor.to(
1826 |             device.value_or(tensor.device()),
1827 |             dtype.value_or(tensor.scalar_type()),
1828 |             /*non_blocking=*/false,
1829 |             /*copy=*/force_copy);
1830 |       } else {
1831 |         tensor = tensor.clone();
1832 |       }
1833 |     } else {
1834 |       // If we are not copying, we have to check whether we have the tensor
1835 |       // in the right device, with the right dtype.
1836 |       TORCH_CHECK_VALUE(
1837 |           !wrong_device,
1838 |           "can't alias tensor from device '",
1839 |           tensor.device(),
1840 |           "' to '",
1841 |           device.value(),
1842 |           "'.");
1843 |       TORCH_CHECK_VALUE(
1844 |           !wrong_dtype,
1845 |           "can't alias tensor with dtype '",
1846 |           tensor.scalar_type(),
1847 |           "' into dtype '",
1848 |           dtype.value(),
1849 |           "'.");
1850 |       // If tensor is a NumPy Array view, we warn the user about non-writeable
1851 |       // arrays if this is the case.
1852 |       if (should_warn_numpy_not_writable) {
1853 |         warn_numpy_not_writeable();
1854 |       }
1855 |     }
1856 | 
1857 |     // Setting 'requires_grad' when the tensor is not a leaf does not work.
1858 |     // Whenever that happens, we have to use 'detach'.
1859 |     if (!tensor.is_leaf() && !return_requires_grad) {
1860 |       tensor = tensor.detach();
1861 |     } else {
1862 |       tensor.set_requires_grad(return_requires_grad);
1863 |     }
1864 |   } else {
1865 |     // Undefined tensor means it does not implement neither DLPack nor
1866 |     // the buffer protocol. Last case is a sequence, in which case we must
1867 |     // copy (copy can't be false).
1868 |     TORCH_CHECK_VALUE(
1869 |         !force_alias, "can't alias arbitrary sequence into a tensor.");
1870 | 
1871 |     // Make tensor from sequence, inferring its type, and then convert
1872 |     // it to the desired type.
1873 |     // Type inference is activated only if the dtype has not been specified.
1874 |     // Otherwise, we force the unwrapped dtype.
1875 |     tensor = internal_new_from_data(
1876 |         TensorOptions(),
1877 |         dtype_unwrapped,
1878 |         device,
1879 |         obj,
1880 |         /* copy_variables = */ false,
1881 |         /* copy_numpy = */ false,
1882 |         /* type_inference = */ !dtype.has_value());
1883 |     tensor.set_requires_grad(return_requires_grad);
1884 |   }
1885 | 
1886 |   return tensor;
1887 | }
1888 | 
1889 | bool only_lift_cpu_tensors() {
1890 |   return kOnlyLiftCPUTensors;
1891 | }
1892 | 
1893 | void set_only_lift_cpu_tensors(bool value) {
1894 |   kOnlyLiftCPUTensors = value;
1895 | }
1896 | 
1897 | } // namespace torch::utils
```
- EN: Implements routines such as `only_lift_cpu_tensors`, `set_only_lift_cpu_tensors` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `only_lift_cpu_tensors`、`set_only_lift_cpu_tensors` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `CtorOrNew`, `CheckSparseTensorInvariantsContext`, `T`.
  - CN: `CtorOrNew`、`CheckSparseTensorInvariantsContext`、`T`。
- **Important routines / 重要例程**
  - EN: `build_options`, `new_with_sizes`, `new_with_storage`, `compute_sizes`, `python_error`, `infer_scalar_type`, `numpy_dtype_to_aten`, `arr`.
  - CN: `build_options`、`new_with_sizes`、`new_with_storage`、`compute_sizes`、`python_error`、`infer_scalar_type`、`numpy_dtype_to_aten`、`arr`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::utils`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::utils` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/python_headers.h>`, `<torch/csrc/utils/tensor_new.h>`, `<torch/csrc/DynamicTypes.h>`, `<torch/csrc/Exceptions.h>`, `<torch/csrc/Size.h>`, `<torch/csrc/autograd/generated/variable_factories.h>`, `<torch/csrc/autograd/variable.h>`, `<torch/csrc/utils/device_lazy_init.h>`, `<torch/csrc/utils/numpy_stub.h>`, `<torch/csrc/utils/pybind.h>`, `<torch/csrc/utils/python_arg_parser.h>`, `<torch/csrc/utils/python_numbers.h>`
- External includes / 外部头文件: `<pybind11/pybind11.h>`, `<optional>`, `<stdexcept>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
