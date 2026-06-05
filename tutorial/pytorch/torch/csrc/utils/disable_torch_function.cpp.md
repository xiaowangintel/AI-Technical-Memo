# disable_torch_function.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/disable_torch_function.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-52
```cpp
 1 | #include <torch/csrc/Exceptions.h>
 2 | #include <torch/csrc/autograd/python_variable.h>
 3 | #include <torch/csrc/utils/disable_torch_function.h>
 4 | #include <torch/csrc/utils/python_strings.h>
 5 | 
 6 | #include <ATen/PythonTorchFunctionTLS.h>
 7 | #include <fmt/format.h>
 8 | 
 9 | namespace torch {
10 | static PyObject* disabled_torch_function = nullptr;
11 | static PyObject* disabled_torch_dispatch = nullptr;
12 | 
13 | bool torch_function_enabled() {
14 |   return at::impl::PythonTorchFunctionTLS::get_disabled_state() ==
15 |       at::impl::TorchFunctionDisabledState::ENABLED;
16 | }
17 | 
18 | PyObject* disabled_torch_function_impl() {
19 |   return disabled_torch_function;
20 | }
21 | 
22 | void set_disabled_torch_function_impl(PyObject* value) {
23 |   disabled_torch_function = value;
24 | }
25 | 
26 | PyObject* disabled_torch_dispatch_impl() {
27 |   return disabled_torch_dispatch;
28 | }
29 | 
30 | void set_disabled_torch_dispatch_impl(PyObject* value) {
31 |   disabled_torch_dispatch = value;
32 | }
33 | } // namespace torch
34 | 
35 | typedef struct {
36 |   PyObject_HEAD
37 |   /* Type-specific fields go here. */
38 |   at::impl::TorchFunctionDisabledState old_state;
39 | } DisableTorchFunctionSubclass;
40 | 
41 | static PyObject* DisableTorchFunctionSubclass__enter(
42 |     PyObject* self,
43 |     PyObject* unused) {
44 |   const auto old_state = at::impl::PythonTorchFunctionTLS::get_disabled_state();
45 |   ((DisableTorchFunctionSubclass*)self)->old_state = old_state;
46 |   if (old_state == at::impl::TorchFunctionDisabledState::ENABLED) {
47 |     at::impl::PythonTorchFunctionTLS::set_disabled_state(
48 |         at::impl::TorchFunctionDisabledState::SUBCLASSES_DISABLED);
49 |   }
50 |   Py_RETURN_NONE;
51 | }
52 | 
```
- EN: Brings in project headers such as `<torch/csrc/Exceptions.h>`, `<torch/csrc/autograd/python_variable.h>`, `<torch/csrc/utils/disable_torch_function.h>`, `<torch/csrc/utils/python_strings.h>` and system or third-party headers such as `<fmt/format.h>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch`) so ownership matches the PyTorch subsystem layout. Implements routines such as `torch_function_enabled`, `disabled_torch_function_impl`, `set_disabled_torch_function_impl`, `disabled_torch_dispatch_impl`, `set_disabled_torch_dispatch_impl` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/Exceptions.h>`、`<torch/csrc/autograd/python_variable.h>`、`<torch/csrc/utils/disable_torch_function.h>`、`<torch/csrc/utils/python_strings.h>`以及系统或第三方头文件，例如 `<fmt/format.h>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `torch_function_enabled`、`disabled_torch_function_impl`、`set_disabled_torch_function_impl`、`disabled_torch_dispatch_impl`、`set_disabled_torch_dispatch_impl` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 53-100
```cpp
 53 | static PyObject* DisableTorchFunctionSubclass__exit(
 54 |     PyObject* self,
 55 |     PyObject* unused) {
 56 |   at::impl::PythonTorchFunctionTLS::set_disabled_state(
 57 |       ((DisableTorchFunctionSubclass*)self)->old_state);
 58 |   Py_RETURN_NONE;
 59 | }
 60 | 
 61 | PyObject* THPModule_isEnabledTorchFunction(PyObject* self, PyObject* unused) {
 62 |   if (torch::torch_function_enabled()) {
 63 |     Py_RETURN_TRUE;
 64 |   } else {
 65 |     Py_RETURN_FALSE;
 66 |   }
 67 | }
 68 | 
 69 | PyObject* THPModule_isAllDisabledTorchFunction(
 70 |     PyObject* self,
 71 |     PyObject* unused) {
 72 |   if (at::impl::torch_function_all_disabled()) {
 73 |     Py_RETURN_TRUE;
 74 |   } else {
 75 |     Py_RETURN_FALSE;
 76 |   }
 77 | }
 78 | 
 79 | static PyMethodDef DisableTorchFunctionSubclass_methods[] = { // NOLINT
 80 |     {"__enter__", DisableTorchFunctionSubclass__enter, METH_NOARGS, nullptr},
 81 |     {"__exit__", DisableTorchFunctionSubclass__exit, METH_VARARGS, nullptr},
 82 |     {nullptr, nullptr, 0, nullptr}};
 83 | 
 84 | static PyTypeObject DisableTorchFunctionSubclassType = {
 85 |     PyVarObject_HEAD_INIT(nullptr, 0)
 86 |     "torch._C.DisableTorchFunctionSubclass", /* tp_name */
 87 |     sizeof(DisableTorchFunctionSubclass), /* tp_basicsize */
 88 |     0, /* tp_itemsize */
 89 |     nullptr, /* tp_dealloc */
 90 |     0, /* tp_vectorcall_offset */
 91 |     nullptr, /* tp_getattr */
 92 |     nullptr, /* tp_setattr */
 93 |     nullptr, /* tp_reserved */
 94 |     nullptr, /* tp_repr */
 95 |     nullptr, /* tp_as_number */
 96 |     nullptr, /* tp_as_sequence */
 97 |     nullptr, /* tp_as_mapping */
 98 |     nullptr, /* tp_hash  */
 99 |     nullptr, /* tp_call */
100 |     nullptr, /* tp_str */
```
- EN: Implements routines such as `DisableTorchFunctionSubclass__exit`, `THPModule_isEnabledTorchFunction`, `THPModule_isAllDisabledTorchFunction` that expose the key API or control flow of this region.
- CN: 实现了 `DisableTorchFunctionSubclass__exit`、`THPModule_isEnabledTorchFunction`、`THPModule_isAllDisabledTorchFunction` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 101-152
```cpp
101 |     nullptr, /* tp_getattro */
102 |     nullptr, /* tp_setattro */
103 |     nullptr, /* tp_as_buffer */
104 |     Py_TPFLAGS_DEFAULT, /* tp_flags */
105 |     nullptr, /* tp_doc */
106 |     nullptr, /* tp_traverse */
107 |     nullptr, /* tp_clear */
108 |     nullptr, /* tp_richcompare */
109 |     0, /* tp_weaklistoffset */
110 |     nullptr, /* tp_iter */
111 |     nullptr, /* tp_iternext */
112 |     DisableTorchFunctionSubclass_methods, /* tp_methods */
113 |     nullptr, /* tp_members */
114 |     nullptr, /* tp_getset */
115 |     nullptr, /* tp_base */
116 |     nullptr, /* tp_dict */
117 |     nullptr, /* tp_descr_get */
118 |     nullptr, /* tp_descr_set */
119 |     0, /* tp_dictoffset */
120 |     nullptr, /* tp_init */
121 |     PyType_GenericAlloc, /* tp_alloc */
122 |     PyType_GenericNew, /* tp_new */
123 | };
124 | 
125 | PyObject* THPModule_DisableTorchFunctionSubclassType() {
126 |   if (PyType_Ready(&DisableTorchFunctionSubclassType) < 0) {
127 |     return nullptr;
128 |   }
129 | 
130 |   return (PyObject*)(&DisableTorchFunctionSubclassType);
131 | }
132 | 
133 | typedef struct {
134 |   PyObject_HEAD
135 |   /* Type-specific fields go here. */
136 |   at::impl::TorchFunctionDisabledState old_state;
137 | } DisableTorchFunction;
138 | 
139 | static PyObject* DisableTorchFunction__enter(PyObject* self, PyObject* unused) {
140 |   ((DisableTorchFunctionSubclass*)self)->old_state =
141 |       at::impl::PythonTorchFunctionTLS::get_disabled_state();
142 |   at::impl::PythonTorchFunctionTLS::set_disabled_state(
143 |       at::impl::TorchFunctionDisabledState::ALL_DISABLED);
144 |   Py_RETURN_NONE;
145 | }
146 | 
147 | static PyObject* DisableTorchFunction__exit(PyObject* self, PyObject* unused) {
148 |   at::impl::PythonTorchFunctionTLS::set_disabled_state(
149 |       ((DisableTorchFunctionSubclass*)self)->old_state);
150 |   Py_RETURN_NONE;
151 | }
152 | 
```
- EN: Implements routines such as `THPModule_DisableTorchFunctionSubclassType`, `DisableTorchFunction__enter`, `DisableTorchFunction__exit` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `THPModule_DisableTorchFunctionSubclassType`、`DisableTorchFunction__enter`、`DisableTorchFunction__exit` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 153-206
```cpp
153 | static PyMethodDef DisableTorchFunction_methods[] = { // NOLINT
154 |     {"__enter__", DisableTorchFunction__enter, METH_NOARGS, nullptr},
155 |     {"__exit__", DisableTorchFunction__exit, METH_VARARGS, nullptr},
156 |     {nullptr, nullptr, 0, nullptr}};
157 | 
158 | static PyTypeObject DisableTorchFunctionType = {
159 |     PyVarObject_HEAD_INIT(nullptr, 0)
160 |     "torch._C.DisableTorchFunction", /* tp_name */
161 |     sizeof(DisableTorchFunction), /* tp_basicsize */
162 |     0, /* tp_itemsize */
163 |     nullptr, /* tp_dealloc */
164 |     0, /* tp_vectorcall_offset */
165 |     nullptr, /* tp_getattr */
166 |     nullptr, /* tp_setattr */
167 |     nullptr, /* tp_reserved */
168 |     nullptr, /* tp_repr */
169 |     nullptr, /* tp_as_number */
170 |     nullptr, /* tp_as_sequence */
171 |     nullptr, /* tp_as_mapping */
172 |     nullptr, /* tp_hash  */
173 |     nullptr, /* tp_call */
174 |     nullptr, /* tp_str */
175 |     nullptr, /* tp_getattro */
176 |     nullptr, /* tp_setattro */
177 |     nullptr, /* tp_as_buffer */
178 |     Py_TPFLAGS_DEFAULT, /* tp_flags */
179 |     nullptr, /* tp_doc */
180 |     nullptr, /* tp_traverse */
181 |     nullptr, /* tp_clear */
182 |     nullptr, /* tp_richcompare */
183 |     0, /* tp_weaklistoffset */
184 |     nullptr, /* tp_iter */
185 |     nullptr, /* tp_iternext */
186 |     DisableTorchFunction_methods, /* tp_methods */
187 |     nullptr, /* tp_members */
188 |     nullptr, /* tp_getset */
189 |     nullptr, /* tp_base */
190 |     nullptr, /* tp_dict */
191 |     nullptr, /* tp_descr_get */
192 |     nullptr, /* tp_descr_set */
193 |     0, /* tp_dictoffset */
194 |     nullptr, /* tp_init */
195 |     PyType_GenericAlloc, /* tp_alloc */
196 |     PyType_GenericNew, /* tp_new */
197 | };
198 | 
199 | PyObject* THPModule_DisableTorchFunctionType() {
200 |   if (PyType_Ready(&DisableTorchFunctionType) < 0) {
201 |     return nullptr;
202 |   }
203 | 
204 |   return (PyObject*)(&DisableTorchFunctionType);
205 | }
206 | 
```
- EN: Implements routines such as `THPModule_DisableTorchFunctionType` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `THPModule_DisableTorchFunctionType` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 207-261
```cpp
207 | PyObject* THPModule_disable_torch_function(PyObject* self, PyObject* a) {
208 |   HANDLE_TH_ERRORS
209 |   PyObject *func = nullptr, *types = nullptr, *args = nullptr,
210 |            *kwargs = nullptr;
211 |   if (!PyArg_ParseTuple(a, "OO|OO", &func, &types, &args, &kwargs)) {
212 |     return nullptr;
213 |   }
214 |   py::tuple py_args;
215 |   if (args == nullptr) {
216 |     py_args = py::make_tuple();
217 |   } else if (PyList_Check(args)) {
218 |     py_args = py::reinterpret_steal<py::tuple>(PyList_AsTuple(args));
219 |   } else if (PyTuple_Check(args)) {
220 |     py_args = py::reinterpret_borrow<py::tuple>(args);
221 |   } else {
222 |     TORCH_CHECK_TYPE(
223 |         false,
224 |         fmt::format("expected List or Tuple (got {})", Py_TYPE(args)->tp_name));
225 |   }
226 | 
227 |   // These are all C-API calls so no exceptions will be raised
228 |   // and therefore no need for RAII approach to storing
229 |   // the old value.
230 |   auto old_value = at::impl::PythonTorchFunctionTLS::get_disabled_state();
231 |   if (old_value == at::impl::TorchFunctionDisabledState::ENABLED) {
232 |     at::impl::PythonTorchFunctionTLS::set_disabled_state(
233 |         at::impl::TorchFunctionDisabledState::SUBCLASSES_DISABLED);
234 |   }
235 |   // kwargs can safely be nullptr here.
236 |   PyObject* result = PyObject_Call(func, py_args.ptr(), kwargs);
237 |   at::impl::PythonTorchFunctionTLS::set_disabled_state(old_value);
238 |   return result;
239 |   END_HANDLE_TH_ERRORS
240 | }
241 | 
242 | PyObject* THPModule_disable_torch_dispatch(PyObject* self, PyObject* a) {
243 |   HANDLE_TH_ERRORS
244 |   PyObject *func = nullptr, *types = nullptr, *args = nullptr,
245 |            *kwargs = nullptr;
246 |   if (!PyArg_ParseTuple(a, "OO|OO", &func, &types, &args, &kwargs)) {
247 |     return nullptr;
248 |   }
249 |   py::tuple py_args;
250 |   if (args == nullptr) {
251 |     py_args = py::make_tuple();
252 |   } else if (PyList_Check(args)) {
253 |     py_args = py::reinterpret_steal<py::tuple>(PyList_AsTuple(args));
254 |   } else if (PyTuple_Check(args)) {
255 |     py_args = py::reinterpret_borrow<py::tuple>(args);
256 |   } else {
257 |     TORCH_CHECK_TYPE(
258 |         false,
259 |         fmt::format("expected List or Tuple (got {})", Py_TYPE(args)->tp_name));
260 |   }
261 | 
```
- EN: Implements routines such as `THPModule_disable_torch_function`, `THPModule_disable_torch_dispatch` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `THPModule_disable_torch_function`、`THPModule_disable_torch_dispatch` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 262-316
```cpp
262 |   // This implementation is not completely correct.  The moral
263 |   // meaning of this function is that we should do a redispatch
264 |   // "after" PythonKey, aka a redispatch() call.  But we don't have a
265 |   // dispatcher call here; we have an opaque Python object.
266 |   //
267 |   // What we have here is a close approximation: instead of redispatch(), we
268 |   // just exclude Python and all the keys before it, so that we will go
269 |   // to the next key after Python.  The difference, however, is we are
270 |   // now PERMANENTLY after Python.  We don't think there are any legitimate
271 |   // cases where we want to go for another round on the entire dispatcher key
272 |   // set, but if there are, then we will have to do something else here.
273 |   c10::impl::ExcludeDispatchKeyGuard guard_(
274 |       // TODO: add constructor for this specifically
275 |       c10::DispatchKeySet(c10::DispatchKeySet::FULL) -
276 |       c10::DispatchKeySet(
277 |           c10::DispatchKeySet::FULL_AFTER, c10::DispatchKey::Python)
278 |       // NB: off by one hazard here, but it works out: python key is not
279 |       // included in AFTER, so it is included in the negation (and that's
280 |       // correct: we want to exclude Python key and everything BEFORE it.)
281 |   );
282 |   auto r = PyObject_Call(func, py_args.ptr(), kwargs);
283 |   if (r == nullptr)
284 |     throw python_error();
285 |   return r;
286 |   END_HANDLE_TH_ERRORS
287 | }
288 | 
289 | // Makes sure that we don't check for __torch_function__ on basic Python types
290 | static bool is_basic_python_type(PyTypeObject* tp) {
291 |   return (
292 |       /* Basic number types */
293 |       tp == &PyBool_Type ||
294 | 
295 |       tp == &PyLong_Type || tp == &PyFloat_Type || tp == &PyComplex_Type ||
296 | 
297 |       /* Basic sequence types */
298 |       tp == &PyList_Type || tp == &PyTuple_Type || tp == &PyDict_Type ||
299 |       tp == &PySet_Type || tp == &PyFrozenSet_Type || tp == &PyUnicode_Type ||
300 |       tp == &PyBytes_Type ||
301 | 
302 |       /* other builtins */
303 |       tp == &PySlice_Type || tp == Py_TYPE(Py_None) ||
304 |       tp == Py_TYPE(Py_Ellipsis) || tp == Py_TYPE(Py_NotImplemented) ||
305 | 
306 |       PyModule_Check(tp) ||
307 |       /* sentinel to swallow trailing || */
308 |       false);
309 | }
310 | 
311 | inline static bool has_torch_function_attr(PyObject* obj) {
312 |   auto attr = PyObject_FastGetAttrString(obj, "__torch_function__");
313 |   return (
314 |       attr.ptr() != nullptr && attr.ptr() != torch::disabled_torch_function);
315 | }
316 | 
```
- EN: Implements routines such as `guard_`, `python_error`, `is_basic_python_type`, `has_torch_function_attr` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `guard_`、`python_error`、`is_basic_python_type`、`has_torch_function_attr` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 317-358
```cpp
317 | namespace torch {
318 | auto check_has_torch_function(PyObject* obj, bool ignore_mode) -> bool {
319 |   if (!ignore_mode && at::impl::torch_function_mode_enabled())
320 |     return true;
321 |   PyTypeObject* tp = Py_TYPE(obj);
322 |   return (
323 |       !THPVariable_CheckTypeExact(tp) && !is_basic_python_type(tp) &&
324 |       torch::torch_function_enabled() && has_torch_function_attr(obj));
325 | }
326 | 
327 | bool has_torch_function(c10::ArrayRef<PyObject*> args) {
328 |   for (const auto obj : args) {
329 |     if (has_torch_function(obj)) {
330 |       return true;
331 |     }
332 |   }
333 |   return false;
334 | }
335 | } // namespace torch
336 | 
337 | inline static bool sequence_has_torch_function(PyObject* args) {
338 |   Py_ssize_t nargs = PySequence_Fast_GET_SIZE(args);
339 |   for (Py_ssize_t i = 0; i < nargs; i++) {
340 |     PyObject* obj = PySequence_Fast_GET_ITEM(args, i);
341 |     if (torch::check_has_torch_function(obj)) {
342 |       return true;
343 |     }
344 |   }
345 |   return false;
346 | }
347 | 
348 | inline static bool array_has_torch_function(
349 |     PyObject* const* args,
350 |     Py_ssize_t nargs) {
351 |   for (Py_ssize_t i = 0; i < nargs; i++) {
352 |     if (torch::check_has_torch_function(args[i])) {
353 |       return true;
354 |     }
355 |   }
356 |   return false;
357 | }
358 | 
```
- EN: Places the implementation in namespace scopes (`torch`) so ownership matches the PyTorch subsystem layout. Implements routines such as `has_torch_function`, `sequence_has_torch_function`, `array_has_torch_function` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 把实现放入命名空间作用域（`torch`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `has_torch_function`、`sequence_has_torch_function`、`array_has_torch_function` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 359-401
```cpp
359 | PyObject* THPModule_has_torch_function(PyObject* /*unused*/, PyObject* arg) {
360 |   bool result = false;
361 |   if (PyTuple_CheckExact(arg) || PyList_CheckExact(arg)) {
362 |     // Fast path:
363 |     //   If we know that we have a tuple or list, we can skip an INCREF and
364 |     //   DECREF from PySequence_Fast. Core functions will always follow this
365 |     //   convention (almost always tuples), and it shaves ~3.5% off the cost of
366 |     //   the check.
367 |     result = sequence_has_torch_function(arg);
368 |   } else {
369 |     auto args = py::reinterpret_steal<py::object>(
370 |         PySequence_Fast(arg, "expected a sequence"));
371 |     if (!args) {
372 |       return nullptr;
373 |     }
374 |     result = sequence_has_torch_function(args.ptr());
375 |   }
376 | 
377 |   if (result) {
378 |     Py_RETURN_TRUE;
379 |   }
380 |   Py_RETURN_FALSE;
381 | }
382 | 
383 | PyObject* THPModule_has_torch_function_unary(
384 |     PyObject* /*unused*/,
385 |     PyObject* obj) {
386 |   // Special case `THPModule_has_torch_function` for the single arg case.
387 |   if (torch::check_has_torch_function(obj)) {
388 |     Py_RETURN_TRUE;
389 |   }
390 |   Py_RETURN_FALSE;
391 | }
392 | 
393 | PyObject* THPModule_has_torch_function_variadic(
394 |     PyObject* /*unused*/,
395 |     PyObject* const* args,
396 |     Py_ssize_t nargs) {
397 |   if (array_has_torch_function(args, nargs)) {
398 |     Py_RETURN_TRUE;
399 |   }
400 |   Py_RETURN_FALSE;
401 | }
```
- EN: Implements routines such as `THPModule_has_torch_function`, `THPModule_has_torch_function_unary`, `THPModule_has_torch_function_variadic` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `THPModule_has_torch_function`、`THPModule_has_torch_function_unary`、`THPModule_has_torch_function_variadic` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `torch_function_enabled`, `disabled_torch_function_impl`, `set_disabled_torch_function_impl`, `disabled_torch_dispatch_impl`, `set_disabled_torch_dispatch_impl`, `DisableTorchFunctionSubclass__enter`, `DisableTorchFunctionSubclass__exit`, `THPModule_isEnabledTorchFunction`.
  - CN: `torch_function_enabled`、`disabled_torch_function_impl`、`set_disabled_torch_function_impl`、`disabled_torch_dispatch_impl`、`set_disabled_torch_dispatch_impl`、`DisableTorchFunctionSubclass__enter`、`DisableTorchFunctionSubclass__exit`、`THPModule_isEnabledTorchFunction`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/Exceptions.h>`, `<torch/csrc/autograd/python_variable.h>`, `<torch/csrc/utils/disable_torch_function.h>`, `<torch/csrc/utils/python_strings.h>`, `<ATen/PythonTorchFunctionTLS.h>`
- External includes / 外部头文件: `<fmt/format.h>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
