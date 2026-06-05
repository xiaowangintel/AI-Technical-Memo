# tensor_numpy.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/tensor_numpy.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-54
```cpp
 1 | #include <fmt/format.h>
 2 | #include <torch/csrc/THP.h>
 3 | #include <torch/csrc/utils/tensor_numpy.h>
 4 | #define WITH_NUMPY_IMPORT_ARRAY
 5 | #include <c10/util/irange.h>
 6 | #include <torch/csrc/utils/numpy_stub.h>
 7 | 
 8 | #ifndef USE_NUMPY
 9 | 
10 | namespace torch::utils {
11 | PyObject* tensor_to_numpy(const at::Tensor&, bool) {
12 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
13 | }
14 | at::Tensor tensor_from_numpy(
15 |     PyObject* obj,
16 |     bool warn_if_not_writeable /*=true*/) {
17 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
18 | }
19 | 
20 | bool is_numpy_available() {
21 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
22 | }
23 | 
24 | bool is_numpy_int(PyObject* obj) {
25 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
26 | }
27 | bool is_numpy_scalar(PyObject* obj) {
28 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
29 | }
30 | at::Tensor tensor_from_cuda_array_interface(
31 |     PyObject* obj,
32 |     std::optional<c10::Device> device_opt) {
33 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
34 | }
35 | 
36 | void warn_numpy_not_writeable() {
37 |   TORCH_CHECK(false, "PyTorch was compiled without NumPy support");
38 | }
39 | 
40 | // No-op stubs.
41 | void validate_numpy_for_dlpack_deleter_bug() {}
42 | 
43 | bool is_numpy_dlpack_deleter_bugged() {
44 |   return false;
45 | }
46 | } // namespace torch::utils
47 | 
48 | #else
49 | 
50 | #include <torch/csrc/DynamicTypes.h>
51 | #include <torch/csrc/Exceptions.h>
52 | #include <torch/csrc/autograd/python_variable.h>
53 | #include <torch/csrc/utils/object_ptr.h>
54 | 
```
- EN: Brings in project headers such as `<torch/csrc/THP.h>`, `<torch/csrc/utils/tensor_numpy.h>`, `<c10/util/irange.h>`, `<torch/csrc/utils/numpy_stub.h>` and system or third-party headers such as `<fmt/format.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::utils`) so ownership matches the PyTorch subsystem layout. Implements routines such as `tensor_to_numpy`, `tensor_from_numpy`, `is_numpy_available`, `is_numpy_int`, `is_numpy_scalar` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/THP.h>`、`<torch/csrc/utils/tensor_numpy.h>`、`<c10/util/irange.h>`、`<torch/csrc/utils/numpy_stub.h>`以及系统或第三方头文件，例如 `<fmt/format.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::utils`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `tensor_to_numpy`、`tensor_from_numpy`、`is_numpy_available`、`is_numpy_int`、`is_numpy_scalar` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 55-107
```cpp
 55 | #include <ATen/ATen.h>
 56 | #include <ATen/TensorUtils.h>
 57 | #include <memory>
 58 | #include <stdexcept>
 59 | 
 60 | using namespace at;
 61 | using namespace torch::autograd;
 62 | 
 63 | namespace torch::utils {
 64 | 
 65 | bool is_numpy_available() {
 66 |   static bool available = []() {
 67 |     if (_import_array() >= 0) {
 68 |       return true;
 69 |     }
 70 |     // Try to get exception message, print warning and return false
 71 |     std::string message = "Failed to initialize NumPy";
 72 |     PyObject *type = nullptr, *value = nullptr, *traceback = nullptr;
 73 |     PyErr_Fetch(&type, &value, &traceback);
 74 |     if (auto str = value ? PyObject_Str(value) : nullptr) {
 75 |       if (auto enc_str = PyUnicode_AsEncodedString(str, "utf-8", "strict")) {
 76 |         if (auto byte_str = PyBytes_AS_STRING(enc_str)) {
 77 |           message += ": " + std::string(byte_str);
 78 |         }
 79 |         Py_XDECREF(enc_str);
 80 |       }
 81 |       Py_XDECREF(str);
 82 |     }
 83 |     PyErr_Clear();
 84 |     TORCH_WARN(message);
 85 |     return false;
 86 |   }();
 87 |   return available;
 88 | }
 89 | static std::vector<npy_intp> to_numpy_shape(IntArrayRef x) {
 90 |   // shape and stride conversion from int64_t to npy_intp
 91 |   auto nelem = x.size();
 92 |   auto result = std::vector<npy_intp>(nelem);
 93 |   for (const auto i : c10::irange(nelem)) {
 94 |     result[i] = static_cast<npy_intp>(x[i]);
 95 |   }
 96 |   return result;
 97 | }
 98 | 
 99 | static std::vector<int64_t> to_aten_shape(int ndim, npy_intp* values) {
100 |   // shape and stride conversion from npy_intp to int64_t
101 |   auto result = std::vector<int64_t>(ndim);
102 |   for (const auto i : c10::irange(ndim)) {
103 |     result[i] = static_cast<int64_t>(values[i]);
104 |   }
105 |   return result;
106 | }
107 | 
```
- EN: Brings in project headers such as `<ATen/ATen.h>`, `<ATen/TensorUtils.h>` and system or third-party headers such as `<memory>`, `<stdexcept>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::utils`) so ownership matches the PyTorch subsystem layout. Implements routines such as `is_numpy_available`, `to_numpy_shape`, `to_aten_shape` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<ATen/ATen.h>`、`<ATen/TensorUtils.h>`以及系统或第三方头文件，例如 `<memory>`、`<stdexcept>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::utils`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `is_numpy_available`、`to_numpy_shape`、`to_aten_shape` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 108-161
```cpp
108 | static std::vector<int64_t> seq_to_aten_shape(PyObject* py_seq) {
109 |   int ndim = PySequence_Length(py_seq);
110 |   if (ndim == -1) {
111 |     TORCH_CHECK_TYPE(false, "shape and strides must be sequences");
112 |   }
113 |   auto result = std::vector<int64_t>(ndim);
114 |   for (const auto i : c10::irange(ndim)) {
115 |     auto item = THPObjectPtr(PySequence_GetItem(py_seq, i));
116 |     if (!item)
117 |       throw python_error();
118 | 
119 |     result[i] = PyLong_AsLongLong(item);
120 |     if (result[i] == -1 && PyErr_Occurred())
121 |       throw python_error();
122 |   }
123 |   return result;
124 | }
125 | 
126 | PyObject* tensor_to_numpy(const at::Tensor& tensor, bool force /*=false*/) {
127 |   TORCH_CHECK(is_numpy_available(), "Numpy is not available");
128 | 
129 |   TORCH_CHECK(
130 |       !tensor.unsafeGetTensorImpl()->is_python_dispatch(),
131 |       ".numpy() is not supported for tensor subclasses.");
132 | 
133 |   TORCH_CHECK_TYPE(
134 |       tensor.layout() == Layout::Strided,
135 |       "can't convert ",
136 |       c10::str(tensor.layout()).c_str(),
137 |       " layout tensor to numpy. ",
138 |       "Use Tensor.to_dense() first.");
139 | 
140 |   if (!force) {
141 |     TORCH_CHECK_TYPE(
142 |         tensor.device().type() == DeviceType::CPU,
143 |         "can't convert ",
144 |         tensor.device().str().c_str(),
145 |         " device type tensor to numpy. Use Tensor.cpu() to ",
146 |         "copy the tensor to host memory first.");
147 | 
148 |     TORCH_CHECK(
149 |         !at::_is_zerotensor(tensor),
150 |         " Cannot convert a ZeroTensor to numpy. Set force=True if you need the zero array.");
151 | 
152 |     TORCH_CHECK(
153 |         !(at::GradMode::is_enabled() && tensor.requires_grad()),
154 |         "Can't call numpy() on Tensor that requires grad. "
155 |         "Use tensor.detach().numpy() instead.");
156 | 
157 |     TORCH_CHECK(
158 |         !tensor.is_conj(),
159 |         "Can't call numpy() on Tensor that has conjugate bit set. ",
160 |         "Use tensor.resolve_conj().numpy() instead.");
161 | 
```
- EN: Implements routines such as `seq_to_aten_shape`, `python_error`, `tensor_to_numpy` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `seq_to_aten_shape`、`python_error`、`tensor_to_numpy` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 162-211
```cpp
162 |     TORCH_CHECK(
163 |         !tensor.is_neg(),
164 |         "Can't call numpy() on Tensor that has negative bit set. "
165 |         "Use tensor.resolve_neg().numpy() instead.");
166 |   }
167 | 
168 |   auto prepared_tensor = tensor.detach().cpu().resolve_conj().resolve_neg();
169 | 
170 |   auto dtype = aten_to_numpy_dtype(prepared_tensor.scalar_type());
171 |   auto sizes = to_numpy_shape(prepared_tensor.sizes());
172 |   auto strides = to_numpy_shape(prepared_tensor.strides());
173 | 
174 |   // NumPy strides use bytes. Torch strides use element counts.
175 |   auto element_size_in_bytes = prepared_tensor.element_size();
176 |   for (auto& stride : strides) {
177 |     stride *= element_size_in_bytes;
178 |   }
179 | 
180 |   auto array = THPObjectPtr(PyArray_New(
181 |       &PyArray_Type,
182 |       static_cast<int>(prepared_tensor.dim()),
183 |       sizes.data(),
184 |       dtype,
185 |       strides.data(),
186 |       prepared_tensor.data_ptr(),
187 |       0,
188 |       NPY_ARRAY_ALIGNED | NPY_ARRAY_WRITEABLE,
189 |       nullptr));
190 |   if (!array)
191 |     return nullptr;
192 | 
193 |   if (at::_is_zerotensor(tensor))
194 |     PyArray_FILLWBYTE(reinterpret_cast<PyArrayObject*>(array.get()), 0);
195 | 
196 |   // TODO: This attempts to keep the underlying memory alive by setting the base
197 |   // object of the ndarray to the tensor and disabling resizes on the storage.
198 |   // This is not sufficient. For example, the tensor's storage may be changed
199 |   // via Tensor.set_, which can free the underlying memory.
200 |   PyObject* py_tensor = THPVariable_Wrap(prepared_tensor);
201 |   if (!py_tensor)
202 |     throw python_error();
203 |   if (PyArray_SetBaseObject((PyArrayObject*)array.get(), py_tensor) == -1) {
204 |     return nullptr;
205 |   }
206 |   // Use the private storage API
207 |   prepared_tensor.storage().unsafeGetStorageImpl()->set_resizable(false);
208 | 
209 |   return array.release();
210 | }
211 | 
```
- EN: Implements routines such as `python_error` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 212-265
```cpp
212 | void warn_numpy_not_writeable() {
213 |   TORCH_WARN_ONCE(
214 |       "The given NumPy array is not writable, and PyTorch does "
215 |       "not support non-writable tensors. This means writing to this tensor "
216 |       "will result in undefined behavior. "
217 |       "You may want to copy the array to protect its data or make it writable "
218 |       "before converting it to a tensor. This type of warning will be "
219 |       "suppressed for the rest of this program.");
220 | }
221 | 
222 | at::Tensor tensor_from_numpy(
223 |     PyObject* obj,
224 |     bool warn_if_not_writeable /*=true*/) {
225 |   TORCH_CHECK(is_numpy_available(), "Numpy is not available");
226 |   TORCH_CHECK_TYPE(
227 |       PyArray_Check(obj),
228 |       "expected np.ndarray (got ",
229 |       Py_TYPE(obj)->tp_name,
230 |       ")");
231 |   auto array = (PyArrayObject*)obj;
232 | 
233 |   // warn_if_not_writable is true when a copy of numpy variable is created.
234 |   // the warning is suppressed when a copy is being created.
235 |   if (!PyArray_ISWRITEABLE(array) && warn_if_not_writeable) {
236 |     warn_numpy_not_writeable();
237 |   }
238 | 
239 |   int ndim = PyArray_NDIM(array);
240 |   auto sizes = to_aten_shape(ndim, PyArray_DIMS(array));
241 |   auto strides = to_aten_shape(ndim, PyArray_STRIDES(array));
242 |   // This must go before the INCREF and element_size checks
243 |   // in case the dtype mapping doesn't exist and an exception is thrown
244 |   auto torch_dtype = numpy_dtype_to_aten(PyArray_TYPE(array));
245 |   // NumPy strides use bytes. Torch strides use element counts.
246 |   const auto element_size_in_bytes = PyArray_ITEMSIZE(array);
247 |   TORCH_CHECK(element_size_in_bytes > 0, "element_size must be 0");
248 | 
249 |   for (auto& stride : strides) {
250 |     TORCH_CHECK_VALUE(
251 |         stride % element_size_in_bytes == 0,
252 |         "given numpy array strides not a multiple of the element byte size. "
253 |         "Copy the numpy array to reallocate the memory.");
254 |     stride /= element_size_in_bytes;
255 |   }
256 | 
257 |   for (const auto i : c10::irange(ndim)) {
258 |     TORCH_CHECK_VALUE(
259 |         strides[i] >= 0,
260 |         "At least one stride in the given numpy array is negative, "
261 |         "and tensors with negative strides are not currently supported. "
262 |         "(You can probably work around this by making a copy of your array "
263 |         " with array.copy().) ");
264 |   }
265 | 
```
- EN: Implements routines such as `warn_numpy_not_writeable`, `tensor_from_numpy` that expose the key API or control flow of this region. At the statement level, this block validates assumptions and reports descriptive failures.
- CN: 实现了 `warn_numpy_not_writeable`、`tensor_from_numpy` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段校验前提条件并报告明确错误。

### Lines 266-318
```cpp
266 |   void* data_ptr = PyArray_DATA(array);
267 |   TORCH_CHECK_VALUE(
268 |       PyArray_EquivByteorders(PyArray_DESCR(array)->byteorder, NPY_NATIVE),
269 |       "given numpy array has byte order different from the native byte order. "
270 |       "Conversion between byte orders is currently not supported.");
271 |   Py_INCREF(obj);
272 |   return at::lift_fresh(at::from_blob(
273 |       data_ptr,
274 |       sizes,
275 |       strides,
276 |       [obj](void* data) {
277 |         pybind11::gil_scoped_acquire gil;
278 |         Py_DECREF(obj);
279 |       },
280 |       at::device(kCPU).dtype(torch_dtype)));
281 | }
282 | 
283 | int aten_to_numpy_dtype(const ScalarType scalar_type) {
284 |   switch (scalar_type) {
285 |     case kDouble:
286 |       return NPY_DOUBLE;
287 |     case kFloat:
288 |       return NPY_FLOAT;
289 |     case kHalf:
290 |       return NPY_HALF;
291 |     case kComplexDouble:
292 |       return NPY_COMPLEX128;
293 |     case kComplexFloat:
294 |       return NPY_COMPLEX64;
295 |     case kLong:
296 |       return NPY_INT64;
297 |     case kInt:
298 |       return NPY_INT32;
299 |     case kShort:
300 |       return NPY_INT16;
301 |     case kChar:
302 |       return NPY_INT8;
303 |     case kByte:
304 |       return NPY_UINT8;
305 |     case kUInt16:
306 |       return NPY_UINT16;
307 |     case kUInt32:
308 |       return NPY_UINT32;
309 |     case kUInt64:
310 |       return NPY_UINT64;
311 |     case kBool:
312 |       return NPY_BOOL;
313 |     default:
314 |       TORCH_CHECK_TYPE(
315 |           false, "Got unsupported ScalarType ", toString(scalar_type));
316 |   }
317 | }
318 | 
```
- EN: Implements routines such as `aten_to_numpy_dtype`, `TORCH_CHECK_TYPE` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `aten_to_numpy_dtype`、`TORCH_CHECK_TYPE` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 319-374
```cpp
319 | ScalarType numpy_dtype_to_aten(int dtype) {
320 |   switch (dtype) {
321 |     case NPY_DOUBLE:
322 |       return kDouble;
323 |     case NPY_FLOAT:
324 |       return kFloat;
325 |     case NPY_HALF:
326 |       return kHalf;
327 |     case NPY_COMPLEX64:
328 |       return kComplexFloat;
329 |     case NPY_COMPLEX128:
330 |       return kComplexDouble;
331 |     case NPY_INT16:
332 |       return kShort;
333 |     case NPY_INT8:
334 |       return kChar;
335 |     case NPY_UINT8:
336 |       return kByte;
337 |     case NPY_UINT16:
338 |       return kUInt16;
339 |     case NPY_UINT32:
340 |       return kUInt32;
341 |     case NPY_UINT64:
342 |       return kUInt64;
343 |     case NPY_BOOL:
344 |       return kBool;
345 |     default:
346 |       // Workaround: MSVC does not support two switch cases that have the same
347 |       // value
348 |       if (dtype == NPY_INT || dtype == NPY_INT32) {
349 |         // To cover all cases we must use NPY_INT because
350 |         // NPY_INT32 is an alias which maybe equal to:
351 |         // - NPY_INT, when sizeof(int) = 4 and sizeof(long) = 8
352 |         // - NPY_LONG, when sizeof(int) = 4 and sizeof(long) = 4
353 |         return kInt;
354 |       } else if (dtype == NPY_LONGLONG || dtype == NPY_INT64) {
355 |         // NPY_INT64 is an alias which maybe equal to:
356 |         // - NPY_LONG, when sizeof(long) = 8 and sizeof(long long) = 8
357 |         // - NPY_LONGLONG, when sizeof(long) = 4 and sizeof(long long) = 8
358 |         return kLong;
359 |       } else {
360 |         break; // break as if this is one of the cases above because this is
361 |                // only a workaround
362 |       }
363 |   }
364 |   auto pytype = THPObjectPtr(PyArray_TypeObjectFromType(dtype));
365 |   if (!pytype)
366 |     throw python_error();
367 |   TORCH_CHECK_TYPE(
368 |       false,
369 |       fmt::format(
370 |           "can't convert np.ndarray of type {}. The only supported types are: "
371 |           "float64, float32, float16, complex64, complex128, int64, int32, int16, int8, uint64, uint32, uint16, uint8, and bool.",
372 |           ((PyTypeObject*)pytype.get())->tp_name));
373 | }
374 | 
```
- EN: Implements routines such as `numpy_dtype_to_aten`, `python_error` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `numpy_dtype_to_aten`、`python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 375-414
```cpp
375 | bool is_numpy_int(PyObject* obj) {
376 |   return is_numpy_available() && PyArray_IsScalar((obj), Integer);
377 | }
378 | 
379 | bool is_numpy_bool(PyObject* obj) {
380 |   return is_numpy_available() && PyArray_IsScalar((obj), Bool);
381 | }
382 | 
383 | bool is_numpy_scalar(PyObject* obj) {
384 |   return is_numpy_available() &&
385 |       (is_numpy_int(obj) || PyArray_IsScalar(obj, Bool) ||
386 |        PyArray_IsScalar(obj, Floating) ||
387 |        PyArray_IsScalar(obj, ComplexFloating));
388 | }
389 | 
390 | at::Tensor tensor_from_cuda_array_interface(
391 |     PyObject* obj,
392 |     std::optional<c10::Device> device_opt) {
393 |   TORCH_CHECK(is_numpy_available(), "Numpy is not available");
394 |   auto cuda_dict =
395 |       THPObjectPtr(PyObject_GetAttrString(obj, "__cuda_array_interface__"));
396 |   TORCH_INTERNAL_ASSERT(cuda_dict);
397 | 
398 |   if (!PyDict_Check(cuda_dict.get())) {
399 |     TORCH_CHECK_TYPE(false, "`__cuda_array_interface__` must be a dict");
400 |   }
401 | 
402 |   // Extract the `obj.__cuda_array_interface__['shape']` attribute
403 |   std::vector<int64_t> sizes;
404 |   {
405 |     PyObject* py_shape = nullptr;
406 |     if (PyDict_GetItemStringRef(cuda_dict, "shape", &py_shape) < 0) {
407 |       throw python_error();
408 |     }
409 |     if (py_shape == nullptr) {
410 |       TORCH_CHECK_TYPE(false, "attribute `shape` must exist");
411 |     }
412 |     sizes = seq_to_aten_shape(py_shape);
413 |   }
414 | 
```
- EN: Implements routines such as `is_numpy_int`, `is_numpy_available`, `is_numpy_bool`, `is_numpy_scalar`, `tensor_from_cuda_array_interface` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `is_numpy_int`、`is_numpy_available`、`is_numpy_bool`、`is_numpy_scalar`、`tensor_from_cuda_array_interface` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 415-464
```cpp
415 |   // Extract the `obj.__cuda_array_interface__['typestr']` attribute
416 |   ScalarType dtype{};
417 |   int64_t dtype_size_in_bytes = 0;
418 |   {
419 |     PyObject* py_typestr = nullptr;
420 |     if (PyDict_GetItemStringRef(cuda_dict, "typestr", &py_typestr) < 0) {
421 |       throw python_error();
422 |     }
423 |     if (py_typestr == nullptr) {
424 |       TORCH_CHECK_TYPE(false, "attribute `typestr` must exist");
425 |     }
426 |     PyArray_Descr* descr = nullptr;
427 |     TORCH_CHECK_VALUE(
428 |         PyArray_DescrConverter(py_typestr, &descr), "cannot parse `typestr`");
429 |     dtype = numpy_dtype_to_aten(descr->type_num);
430 | #if NPY_ABI_VERSION >= 0x02000000
431 |     dtype_size_in_bytes = PyDataType_ELSIZE(descr);
432 | #else
433 |     dtype_size_in_bytes = descr->elsize;
434 | #endif
435 |     TORCH_INTERNAL_ASSERT(dtype_size_in_bytes > 0);
436 |   }
437 | 
438 |   // Extract the `obj.__cuda_array_interface__['data']` attribute
439 |   void* data_ptr = nullptr;
440 |   {
441 |     PyObject* py_data = nullptr;
442 |     if (PyDict_GetItemStringRef(cuda_dict, "data", &py_data) < 0) {
443 |       throw python_error();
444 |     }
445 |     if (py_data == nullptr) {
446 |       TORCH_CHECK_TYPE(false, "attribute `shape` data exist");
447 |     }
448 |     if (!PyTuple_Check(py_data) || PyTuple_GET_SIZE(py_data) != 2) {
449 |       TORCH_CHECK_TYPE(false, "`data` must be a 2-tuple of (int, bool)");
450 |     }
451 |     data_ptr = PyLong_AsVoidPtr(PyTuple_GET_ITEM(py_data, 0));
452 |     if (data_ptr == nullptr && PyErr_Occurred()) {
453 |       throw python_error();
454 |     }
455 |     int read_only = PyObject_IsTrue(PyTuple_GET_ITEM(py_data, 1));
456 |     if (read_only == -1) {
457 |       throw python_error();
458 |     }
459 |     if (read_only) {
460 |       TORCH_CHECK_TYPE(
461 |           false, "the read only flag is not supported, should always be False");
462 |     }
463 |   }
464 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `python_error` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；校验前提条件并报告明确错误。

### Lines 465-513
```cpp
465 |   // Extract the `obj.__cuda_array_interface__['strides']` attribute
466 |   std::vector<int64_t> strides;
467 |   {
468 |     PyObject* py_strides = nullptr;
469 |     if (PyDict_GetItemStringRef(cuda_dict, "strides", &py_strides) < 0) {
470 |       throw python_error();
471 |     }
472 |     if (py_strides != nullptr && !Py_IsNone(py_strides)) {
473 |       if (PySequence_Length(py_strides) == -1 ||
474 |           static_cast<size_t>(PySequence_Length(py_strides)) != sizes.size()) {
475 |         TORCH_CHECK_TYPE(
476 |             false, "strides must be a sequence of the same length as shape");
477 |       }
478 |       strides = seq_to_aten_shape(py_strides);
479 | 
480 |       // __cuda_array_interface__ strides use bytes. Torch strides use element
481 |       // counts.
482 |       for (auto& stride : strides) {
483 |         TORCH_CHECK_VALUE(
484 |             stride % dtype_size_in_bytes == 0,
485 |             "given array strides not a multiple of the element byte size. "
486 |             "Make a copy of the array to reallocate the memory.");
487 |         stride /= dtype_size_in_bytes;
488 |       }
489 |     } else {
490 |       strides = at::detail::defaultStrides(sizes);
491 |     }
492 |   }
493 | 
494 |   const auto target_device = [&]() -> std::optional<Device> {
495 |     // note(crcrpar): zero-size arrays come with nullptr.
496 |     // ref:
497 |     // https://numba.readthedocs.io/en/stable/cuda/cuda_array_interface.html#cuda-array-interface-version-3
498 |     if (data_ptr != nullptr) {
499 |       if (device_opt.has_value() && device_opt->has_index()) {
500 |         // if device_opt is provided with explicit device index, use it
501 |         return device_opt;
502 |       } else {
503 |         // otherwise infer from cudaPointerGetAttributes later in from_blob
504 |         return std::nullopt;
505 |       }
506 |     } else {
507 |       const auto current_device = at::detail::getCUDAHooks().getCurrentDevice();
508 |       return Device(
509 |           kCUDA,
510 |           static_cast<DeviceIndex>(current_device > -1 ? current_device : 0));
511 |     }
512 |   }();
513 | 
```
- EN: Implements routines such as `python_error`, `Device` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `python_error`、`Device` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 514-569
```cpp
514 |   Py_INCREF(obj);
515 |   return at::from_blob(
516 |       data_ptr,
517 |       sizes,
518 |       strides,
519 |       [obj](void* data) {
520 |         pybind11::gil_scoped_acquire gil;
521 |         Py_DECREF(obj);
522 |       },
523 |       at::device(kCUDA).dtype(dtype),
524 |       target_device);
525 | }
526 | 
527 | // Mutated only once (during module init); behaves as an immutable variable
528 | // thereafter.
529 | static bool numpy_with_dlpack_deleter_bug_installed = false;
530 | 
531 | // NumPy implemented support for Dlpack capsules in version 1.22.0. However, the
532 | // initial implementation did not correctly handle the invocation of
533 | // `DLManagedTensor::deleter` in a no-GIL context. Until PyTorch 1.13.0, we
534 | // were implicitly holding the GIL when the deleter was invoked, but this
535 | // incurred a significant performance overhead when mem-unmapping large tensors.
536 | // Starting with PyTorch 1.13.0, we release the GIL in `THPVariable_clear` just
537 | // before deallocation, but this triggers the aforementioned bug in NumPy.
538 | //
539 | // The NumPy bug should be fixed in version 1.24.0, but all releases
540 | // between 1.22.0 and 1.23.5 result in internal assertion failures that
541 | // consequently lead to segfaults. To work around this, we need to selectively
542 | // disable the optimization whenever we detect a buggy NumPy installation.
543 | // We would ideally restrict the "fix" just to Dlpack-backed tensors that stem
544 | // from NumPy, but given that it is difficult to confidently detect the
545 | // provenance of such tensors, we have to resort to a more general approach.
546 | //
547 | // References:
548 | //  https://github.com/pytorch/pytorch/issues/88082
549 | //  https://github.com/pytorch/pytorch/issues/77139
550 | //  https://github.com/numpy/numpy/issues/22507
551 | void validate_numpy_for_dlpack_deleter_bug() {
552 |   // Ensure that we don't call this more than once per session.
553 |   static bool validated = false;
554 |   TORCH_INTERNAL_ASSERT(validated == false);
555 |   validated = true;
556 | 
557 |   THPObjectPtr numpy_module(PyImport_ImportModule("numpy"));
558 |   if (!numpy_module) {
559 |     PyErr_Clear();
560 |     return;
561 |   }
562 | 
563 |   THPObjectPtr version_attr(
564 |       PyObject_GetAttrString(numpy_module.get(), "__version__"));
565 |   if (!version_attr) {
566 |     PyErr_Clear();
567 |     return;
568 |   }
569 | 
```
- EN: Implements routines such as `validate_numpy_for_dlpack_deleter_bug`, `numpy_module`, `version_attr` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `validate_numpy_for_dlpack_deleter_bug`、`numpy_module`、`version_attr` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 570-590
```cpp
570 |   Py_ssize_t version_utf8_size = 0;
571 |   const char* version_utf8 =
572 |       PyUnicode_AsUTF8AndSize(version_attr.get(), &version_utf8_size);
573 |   if (!version_utf8_size) {
574 |     PyErr_Clear();
575 |     return;
576 |   }
577 |   std::string version(version_utf8, version_utf8_size);
578 |   if (version_utf8_size < 4)
579 |     return;
580 |   std::string truncated_version(version.substr(0, 4));
581 |   numpy_with_dlpack_deleter_bug_installed =
582 |       truncated_version == "1.22" || truncated_version == "1.23";
583 | }
584 | 
585 | bool is_numpy_dlpack_deleter_bugged() {
586 |   return numpy_with_dlpack_deleter_bug_installed;
587 | }
588 | } // namespace torch::utils
589 | 
590 | #endif // USE_NUMPY
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `version`, `truncated_version`, `is_numpy_dlpack_deleter_bugged` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `version`、`truncated_version`、`is_numpy_dlpack_deleter_bugged` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `tensor_to_numpy`, `tensor_from_numpy`, `is_numpy_available`, `is_numpy_int`, `is_numpy_scalar`, `tensor_from_cuda_array_interface`, `warn_numpy_not_writeable`, `validate_numpy_for_dlpack_deleter_bug`.
  - CN: `tensor_to_numpy`、`tensor_from_numpy`、`is_numpy_available`、`is_numpy_int`、`is_numpy_scalar`、`tensor_from_cuda_array_interface`、`warn_numpy_not_writeable`、`validate_numpy_for_dlpack_deleter_bug`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::utils`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::utils` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/THP.h>`, `<torch/csrc/utils/tensor_numpy.h>`, `<c10/util/irange.h>`, `<torch/csrc/utils/numpy_stub.h>`, `<torch/csrc/DynamicTypes.h>`, `<torch/csrc/Exceptions.h>`, `<torch/csrc/autograd/python_variable.h>`, `<torch/csrc/utils/object_ptr.h>`, `<ATen/ATen.h>`, `<ATen/TensorUtils.h>`
- External includes / 外部头文件: `<fmt/format.h>`, `<memory>`, `<stdexcept>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
