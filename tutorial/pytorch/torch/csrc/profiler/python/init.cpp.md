# init.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/python/init.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Bridges C++ components to Python-facing APIs or bindings.
  - CN: 把 C++ 组件桥接到面向 Python 的 API 或绑定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40
```cpp
 1 | #include <torch/csrc/profiler/python/init.h>
 2 | 
 3 | #include <ATen/record_function.h>
 4 | #include <c10/core/impl/PyInterpreter.h>
 5 | #include <c10/util/Exception.h>
 6 | #include <c10/util/overloaded.h>
 7 | #include <torch/csrc/DynamicTypes.h>
 8 | #include <torch/csrc/autograd/utils/wrap_outputs.h>
 9 | #include <torch/csrc/jit/python/pybind_utils.h>
10 | #include <torch/csrc/profiler/collection.h>
11 | #include <torch/csrc/profiler/python/combined_traceback.h>
12 | #include <torch/csrc/profiler/standalone/execution_trace_observer.h>
13 | #include <torch/csrc/utils/pybind.h>
14 | 
15 | struct THPCapturedTraceback {
16 |   PyObject_HEAD
17 |   std::shared_ptr<torch::CapturedTraceback> data;
18 | };
19 | 
20 | static int THPCapturedTraceback_traverse(
21 |     PyObject* self,
22 |     visitproc visit,
23 |     void* arg) {
24 |   return ((THPCapturedTraceback*)self)
25 |       ->data->traversePython((int (*)(void*, void*))visit, arg);
26 | }
27 | 
28 | static int THPCapturedTraceback_clear(PyObject* self) {
29 |   return ((THPCapturedTraceback*)self)->data->clearPython();
30 | }
31 | 
32 | static void THPCapturedTraceback_dealloc(PyObject* self_) {
33 |   auto* self = (THPCapturedTraceback*)self_;
34 |   PyObject_GC_UnTrack(self);
35 |   self->data.~shared_ptr<torch::CapturedTraceback>();
36 |   // promptly trigger delayed frees since we have GIL
37 |   torch::freeDeadCapturedTracebackFrames();
38 |   PyObject_GC_Del(self);
39 | }
40 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/python/init.h>`, `<ATen/record_function.h>`, `<c10/core/impl/PyInterpreter.h>`, `<c10/util/Exception.h>` so this section can use their types, APIs, or macros. Defines or extends data abstractions such as `THPCapturedTraceback` that structure the state handled by this file. Implements routines such as `THPCapturedTraceback_traverse`, `THPCapturedTraceback_clear`, `THPCapturedTraceback_dealloc` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/python/init.h>`、`<ATen/record_function.h>`、`<c10/core/impl/PyInterpreter.h>`、`<c10/util/Exception.h>`，使本段代码能够使用相关类型、API 或宏。 定义或扩展了 `THPCapturedTraceback` 等数据抽象，用来组织本文件处理的状态。 实现了 `THPCapturedTraceback_traverse`、`THPCapturedTraceback_clear`、`THPCapturedTraceback_dealloc` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 41-91
```cpp
41 | static PyTypeObject THPCapturedTracebackType = {
42 |     PyVarObject_HEAD_INIT(nullptr, 0)
43 |     "torch._C._profiler.CapturedTraceback", /* tp_name */
44 |     sizeof(THPCapturedTraceback), /* tp_basicsize */
45 |     0, /* tp_itemsize */
46 |     THPCapturedTraceback_dealloc, /* tp_dealloc */
47 |     0, /* tp_vectorcall_offset */
48 |     nullptr, /* tp_getattr */
49 |     nullptr, /* tp_setattr */
50 |     nullptr, /* tp_reserved */
51 |     nullptr, /* tp_repr */
52 |     nullptr, /* tp_as_number */
53 |     nullptr, /* tp_as_sequence */
54 |     nullptr, /* tp_as_mapping */
55 |     nullptr, /* tp_hash  */
56 |     nullptr, /* tp_call */
57 |     nullptr, /* tp_str */
58 |     nullptr, /* tp_getattro */
59 |     nullptr, /* tp_setattro */
60 |     nullptr, /* tp_as_buffer */
61 |     // NOLINTNEXTLINE(misc-redundant-expression)
62 |     Py_TPFLAGS_DEFAULT | Py_TPFLAGS_HAVE_GC, /* tp_flags */
63 |     nullptr, /* tp_doc */
64 |     (traverseproc)THPCapturedTraceback_traverse, /* tp_traverse */
65 |     (inquiry)THPCapturedTraceback_clear, /* tp_clear */
66 |     nullptr, /* tp_richcompare */
67 |     0, /* tp_weaklistoffset */
68 |     nullptr, /* tp_iter */
69 |     nullptr, /* tp_iternext */
70 |     nullptr, /* tp_methods */
71 |     nullptr, /* tp_members */
72 |     nullptr, /* tp_getset */
73 |     nullptr, /* tp_base */
74 |     nullptr, /* tp_dict */
75 |     nullptr, /* tp_descr_get */
76 |     nullptr, /* tp_descr_set */
77 |     0, /* tp_dictoffset */
78 |     nullptr, /* tp_init */
79 |     nullptr, /* tp_alloc */
80 |     nullptr, /* tp_new */
81 | };
82 | 
83 | namespace pybind11::detail {
84 | 
85 | template <>
86 | struct type_caster<std::shared_ptr<torch::CapturedTraceback>> {
87 |  public:
88 |   PYBIND11_TYPE_CASTER(
89 |       std::shared_ptr<torch::CapturedTraceback>,
90 |       _("torch._C._profiler.CapturedTraceback"));
91 | 
```
- EN: Places the implementation in namespace scopes (`pybind11::detail`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `type_caster` that structure the state handled by this file. Implements routines such as `PYBIND11_TYPE_CASTER` that expose the key API or control flow of this region.
- CN: 把实现放入命名空间作用域（`pybind11::detail`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `type_caster` 等数据抽象，用来组织本文件处理的状态。 实现了 `PYBIND11_TYPE_CASTER` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 92-146
```cpp
 92 |   bool load(handle src, bool /*unused*/) {
 93 |     if (Py_TYPE(src.ptr()) == &THPCapturedTracebackType) {
 94 |       value = reinterpret_cast<THPCapturedTraceback*>(src.ptr())->data;
 95 |       return true;
 96 |     }
 97 |     return false;
 98 |   }
 99 | 
100 |   static handle cast(
101 |       std::shared_ptr<torch::CapturedTraceback> src,
102 |       return_value_policy /* policy */,
103 |       handle /* parent */) {
104 |     auto* r = PyObject_GC_New(THPCapturedTraceback, &THPCapturedTracebackType);
105 |     new (&r->data) std::shared_ptr<torch::CapturedTraceback>(std::move(src));
106 |     return py::handle((PyObject*)r);
107 |   }
108 | };
109 | 
110 | } // namespace pybind11::detail
111 | 
112 | namespace torch::profiler {
113 | 
114 | /* [NOTE: RecordFunctionFast]
115 |  * This is an alternate way to call record_function from python.
116 |  * The torch.profiler.record_function context manager is slow (~14us on
117 |  * benchmarks in Aug 2023), which is usually fine for module-level annotations
118 |  * in python, but slow for per-op annotations. Part of the reason it is slow is
119 |  * because the calls go through the dispatcher, in order to make the
120 |  * record_function calls work with torchscript.
121 |  *
122 |  * This implementation doesn't go through the dispatcher and so it won't work
123 |  * with any feature relying on the dispatcher (e.g. torchscript or
124 |  * torch.compile)
125 |  *
126 |  * An alternate solution would be to implement a python context manager that
127 |  * calls into C++ for the enter/exit function:
128 |  *    @contextlib.contextmanager
129 |  *    def record_function_fast(name):
130 |  *      rf = torch._C._record_function_fast_enter(name)
131 |  *      try:
132 |  *        yield
133 |  *      finally:
134 |  *        torch._C._record_function_fast_exit(rf)
135 |  * The C++ implementation here is faster by ~0.2-0.4us per context manager.
136 |  */
137 | 
138 | namespace {
139 | struct RecordFunctionFast {
140 |   PyObject_HEAD
141 |   PyObject* name;
142 |   PyObject* input_values;
143 |   PyObject* keyword_values;
144 |   std::unique_ptr<at::RecordFunction> guard;
145 | };
146 | 
```
- EN: Places the implementation in namespace scopes (`torch::profiler`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `RecordFunctionFast` that structure the state handled by this file. Implements routines such as `load`, `cast` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 把实现放入命名空间作用域（`torch::profiler`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `RecordFunctionFast` 等数据抽象，用来组织本文件处理的状态。 实现了 `load`、`cast` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 147-194
```cpp
147 | PyObject* RecordFunctionFast_new(
148 |     PyTypeObject* subtype,
149 |     PyObject* args,
150 |     PyObject* kwargs) {
151 |   RecordFunctionFast* self = (RecordFunctionFast*)subtype->tp_alloc(subtype, 0);
152 |   if (self != nullptr) {
153 |     self->name = nullptr;
154 |     self->input_values = nullptr;
155 |     self->keyword_values = nullptr;
156 |     self->guard.reset();
157 |   }
158 |   return (PyObject*)self;
159 | }
160 | 
161 | int RecordFunctionFast_init(
162 |     PyObject* selfGeneric,
163 |     PyObject* args,
164 |     PyObject* kwargs) {
165 |   auto self = (RecordFunctionFast*)selfGeneric;
166 |   // NOLINTNEXTLINE(*-c-arrays*)
167 |   constexpr const char* kwlist[] = {
168 |       "name", "input_values", "keyword_values", nullptr};
169 |   PyObject* name = nullptr;
170 |   PyObject* input_values = nullptr;
171 |   PyObject* keyword_values = nullptr;
172 |   if (!PyArg_ParseTupleAndKeywords(
173 |           args,
174 |           kwargs,
175 |           "O|OO", // name is required PyObject, args and kwargs are optional
176 |                   // PyObjects
177 |           // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
178 |           const_cast<char**>(kwlist),
179 |           &name,
180 |           &input_values,
181 |           &keyword_values)) {
182 |     return -1;
183 |   }
184 |   if (name) {
185 |     TORCH_CHECK(
186 |         THPUtils_checkString(name),
187 |         "The name passed to RecordFunctionFast must be a string");
188 |     Py_INCREF(name);
189 |     self->name = name;
190 |   }
191 |   if (input_values) {
192 |     TORCH_CHECK(
193 |         PyList_Check(input_values) || PyTuple_Check(input_values),
194 |         "input_values must be a list or tuple");
```
- EN: Implements routines such as `RecordFunctionFast_new`, `RecordFunctionFast_init` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `RecordFunctionFast_new`、`RecordFunctionFast_init` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 195-242
```cpp
195 |     Py_INCREF(input_values);
196 |     self->input_values = input_values;
197 |   }
198 |   if (keyword_values) {
199 |     TORCH_CHECK(PyDict_Check(keyword_values), "keyword_values must be dict");
200 |     Py_INCREF(keyword_values);
201 |     self->keyword_values = keyword_values;
202 |   }
203 |   return 0;
204 | }
205 | 
206 | void RecordFunctionFast_dealloc(PyObject* selfGeneric) {
207 |   auto self = (RecordFunctionFast*)selfGeneric;
208 |   Py_CLEAR(self->name);
209 |   Py_CLEAR(self->input_values);
210 |   Py_CLEAR(self->keyword_values);
211 |   if (self->guard) {
212 |     self->guard.reset();
213 |   }
214 |   Py_TYPE(self)->tp_free(self);
215 | }
216 | 
217 | PyObject* RecordFunctionFast_enter(PyObject* selfGeneric, PyObject* unused) {
218 |   HANDLE_TH_ERRORS
219 |   if (torch::profiler::impl::ProfilerStateBase::get() != nullptr) {
220 |     auto self = (RecordFunctionFast*)selfGeneric;
221 |     TORCH_INTERNAL_ASSERT(
222 |         !self->guard,
223 |         "Trying to enter a new record_function_fast context but the guard is unexpectedly already set");
224 |     auto scope = at::RecordScope::FUNCTION;
225 |     std::vector<at::IValue> args;
226 |     std::unordered_map<std::string, at::IValue> kwargs;
227 |     bool profiler_need_input = torch::autograd::profiler::profilerEnabled() &&
228 |         torch::autograd::profiler::getProfilerConfig().report_input_shapes;
229 |     // parse through args if they exist
230 |     if (self->input_values != nullptr && profiler_need_input) {
231 |       THPObjectPtr input_fast(
232 |           PySequence_Fast(self->input_values, "input must be a sequence"));
233 |       PyObject** input_items = PySequence_Fast_ITEMS(input_fast.get());
234 |       for (int i = 0; i < PySequence_Fast_GET_SIZE(input_fast.get()); i++) {
235 |         PyObject* item = input_items[i];
236 |         auto match = torch::jit::tryToInferType(item);
237 |         if (match.success()) {
238 |           args.push_back(torch::jit::toIValue(item, match.type()));
239 |         }
240 |       }
241 |     }
242 | 
```
- EN: Implements routines such as `RecordFunctionFast_dealloc`, `RecordFunctionFast_enter`, `input_fast` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `RecordFunctionFast_dealloc`、`RecordFunctionFast_enter`、`input_fast` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 243-290
```cpp
243 |     // parse through kwargs if they exist
244 |     if (self->keyword_values != nullptr && profiler_need_input) {
245 |       const auto kw_size = PyDict_Size(self->keyword_values);
246 |       if (kw_size > 0) {
247 |         kwargs.reserve(static_cast<size_t>(kw_size));
248 |       }
249 |       Py_ssize_t pos = 0;
250 |       PyObject *key = nullptr, *value = nullptr;
251 |       while (PyDict_Next(self->keyword_values, &pos, &key, &value)) {
252 |         // Get the string representation of the key and value
253 |         std::string key_str = THPUtils_unpackString(key);
254 |         at::IValue ivalue;
255 |         if (THPUtils_checkString(value)) {
256 |           ivalue = at::IValue(THPUtils_unpackString(value));
257 |         } else {
258 |           // Handle other types (not strings, not lists)
259 |           auto match = torch::jit::tryToInferPrimitiveType(value);
260 |           if (match.success()) {
261 |             ivalue = torch::jit::toIValue(value, match.type());
262 |           } else if (PyList_Check(value)) {
263 |             // Handle list of strings
264 |             bool all_strings = true;
265 |             std::vector<std::string> string_list;
266 |             Py_ssize_t list_size = PyList_Size(value);
267 | 
268 |             for (Py_ssize_t i = 0; i < list_size; i++) {
269 |               PyObject* item = PyList_GetItem(value, i);
270 |               if (THPUtils_checkString(item)) {
271 |                 string_list.push_back(THPUtils_unpackString(item));
272 |               } else {
273 |                 all_strings = false;
274 |                 break;
275 |               }
276 |             }
277 | 
278 |             if (all_strings) {
279 |               c10::List<std::string> string_ivalue_list(string_list);
280 |               ivalue = at::IValue(string_ivalue_list);
281 |             } else {
282 |               TORCH_WARN(
283 |                   "Unable to infer type of value in the List for keyword: ",
284 |                   key_str);
285 |             }
286 |           } else {
287 |             TORCH_WARN("Unable to infer type of value for keyword: ", key_str);
288 |             ivalue = at::IValue("NULL");
289 |           }
290 |         }
```
- EN: Implements routines such as `string_ivalue_list` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide.
- CN: 实现了 `string_ivalue_list` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子。

### Lines 291-342
```cpp
291 |         kwargs[key_str] = ivalue;
292 |       }
293 |     }
294 |     auto it = kwargs.find("scope");
295 |     if (it != kwargs.end()) {
296 |       auto value = it->second;
297 |       if (value.isString()) {
298 |         auto value_str = value.toStringRef();
299 |         if (value_str == "user_scope") {
300 |           scope = at::RecordScope::USER_SCOPE;
301 |         }
302 |       }
303 |     }
304 |     self->guard = std::make_unique<at::RecordFunction>(scope);
305 |     self->guard->before(THPUtils_unpackString(self->name), &args, &kwargs);
306 |   }
307 |   Py_RETURN_NONE;
308 |   END_HANDLE_TH_ERRORS
309 | }
310 | 
311 | PyObject* RecordFunctionFast_exit(PyObject* selfGeneric, PyObject* unused) {
312 |   HANDLE_TH_ERRORS
313 |   if (torch::profiler::impl::ProfilerStateBase::get() != nullptr) {
314 |     auto self = (RecordFunctionFast*)selfGeneric;
315 |     TORCH_INTERNAL_ASSERT(
316 |         self->guard,
317 |         "Trying to exit an active record_function_fast context but no guard is set");
318 |     self->guard.reset();
319 |   }
320 |   Py_RETURN_NONE;
321 |   END_HANDLE_TH_ERRORS
322 | }
323 | } // namespace
324 | 
325 | void initPythonBindings(PyObject* module) {
326 |   auto rootModule = py::handle(module).cast<py::module>();
327 |   auto m = rootModule.def_submodule("_profiler");
328 | 
329 |   using namespace torch::profiler::impl;
330 | 
331 |   py::enum_<at::RecordScope>(m, "RecordScope")
332 |       .value("FUNCTION", at::RecordScope::FUNCTION)
333 |       .value("BACKWARD_FUNCTION", at::RecordScope::BACKWARD_FUNCTION)
334 |       .value("TORCHSCRIPT_FUNCTION", at::RecordScope::TORCHSCRIPT_FUNCTION)
335 |       .value("KERNEL_FUNCTION_DTYPE", at::RecordScope::KERNEL_FUNCTION_DTYPE)
336 |       .value("CUSTOM_CLASS", at::RecordScope::CUSTOM_CLASS)
337 |       .value("BUILD_FEATURE", at::RecordScope::BUILD_FEATURE)
338 |       .value("LITE_INTERPRETER", at::RecordScope::LITE_INTERPRETER)
339 |       .value("USER_SCOPE", at::RecordScope::USER_SCOPE)
340 |       .value("STATIC_RUNTIME_OP", at::RecordScope::STATIC_RUNTIME_OP)
341 |       .value("STATIC_RUNTIME_MODEL", at::RecordScope::STATIC_RUNTIME_MODEL);
342 | 
```
- EN: Implements routines such as `RecordFunctionFast_exit`, `initPythonBindings` that expose the key API or control flow of this region. Reads environment switches (`BACKWARD_FUNCTION`, `BUILD_FEATURE`, `CUSTOM_CLASS`, `FUNCTION`, `KERNEL_FUNCTION_DTYPE`, `LITE_INTERPRETER`) to tune runtime behavior.
- CN: 实现了 `RecordFunctionFast_exit`、`initPythonBindings` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`BACKWARD_FUNCTION`、`BUILD_FEATURE`、`CUSTOM_CLASS`、`FUNCTION`、`KERNEL_FUNCTION_DTYPE`、`LITE_INTERPRETER`）来调整运行时行为。

### Lines 343-390
```cpp
343 |   py::enum_<ProfilerState>(m, "ProfilerState")
344 |       .value("Disabled", ProfilerState::Disabled)
345 |       .value("CPU", ProfilerState::CPU)
346 |       .value("CUDA", ProfilerState::CUDA)
347 |       .value("NVTX", ProfilerState::NVTX)
348 |       .value("ITT", ProfilerState::ITT)
349 |       .value("PRIVATEUSE1", ProfilerState::PRIVATEUSE1)
350 |       .value("KINETO", ProfilerState::KINETO)
351 |       .value("KINETO_GPU_FALLBACK", ProfilerState::KINETO_GPU_FALLBACK)
352 |       .value(
353 |           "KINETO_PRIVATEUSE1_FALLBACK",
354 |           ProfilerState::KINETO_PRIVATEUSE1_FALLBACK)
355 |       .value("KINETO_PRIVATEUSE1", ProfilerState::KINETO_PRIVATEUSE1);
356 | 
357 |   py::enum_<ActiveProfilerType>(m, "ActiveProfilerType")
358 |       .value("NONE", ActiveProfilerType::NONE)
359 |       .value("LEGACY", ActiveProfilerType::LEGACY)
360 |       .value("KINETO", ActiveProfilerType::KINETO)
361 |       .value("NVTX", ActiveProfilerType::NVTX)
362 |       .value("ITT", ActiveProfilerType::ITT)
363 |       .value("PRIVATEUSE1", ActiveProfilerType::PRIVATEUSE1);
364 | 
365 |   py::enum_<ActivityType>(m, "ProfilerActivity")
366 |       .value("CPU", ActivityType::CPU)
367 |       .value("XPU", ActivityType::XPU)
368 |       .value("MTIA", ActivityType::MTIA)
369 |       .value("CUDA", ActivityType::CUDA)
370 |       .value("HPU", ActivityType::HPU)
371 |       .value("PrivateUse1", ActivityType::PrivateUse1);
372 | 
373 |   py::class_<ExperimentalConfig>(m, "_ExperimentalConfig")
374 |       .def(
375 |           py::init<
376 |               std::vector<std::string> /* profiler_metrics */,
377 |               bool /* profiler_measure_per_kernel */,
378 |               bool /* verbose */,
379 |               std::vector<std::string> /* performance_events  */,
380 |               bool /* enable_cuda_sync_events */,
381 |               bool /* adjust_profiler_step */,
382 |               bool /* disable_external_correlation*/,
383 |               bool /* profile_all_threads */,
384 |               bool /* capture_overload_names */,
385 |               bool /* record_python_gc_info */,
386 |               bool /* expose_kineto_event_metadata */,
387 |               std::string /* custom_profiler_config*/
388 |               >(),
389 |           "An experimental config for Kineto features. Please note that"
390 |           "backward compatibility is not guaranteed.\n"
```
- EN: Reads environment switches (`KINETO`, `KINETO_GPU_FALLBACK`, `KINETO_PRIVATEUSE1`, `KINETO_PRIVATEUSE1_FALLBACK`, `LEGACY`, `PRIVATEUSE1`) to tune runtime behavior.
- CN: 读取环境变量开关（`KINETO`、`KINETO_GPU_FALLBACK`、`KINETO_PRIVATEUSE1`、`KINETO_PRIVATEUSE1_FALLBACK`、`LEGACY`、`PRIVATEUSE1`）来调整运行时行为。

### Lines 391-438
```cpp
391 |           "    profiler_metrics : a list of CUPTI profiler metrics used\n"
392 |           "       to measure GPU performance events.\n"
393 |           "       If this list contains values Kineto runs in CUPTI profiler mode\n"
394 |           "    profiler_measure_per_kernel (bool) : whether to profile metrics per kernel\n"
395 |           "       or for the entire measurement duration.\n"
396 |           "    verbose (bool) : whether the trace file has `Call stack` field or not.\n"
397 |           "    performance_events : a list of profiler events to be used for measurement.\n"
398 |           "    enable_cuda_sync_events : for CUDA profiling mode, enable adding CUDA synchronization events\n"
399 |           "       that expose CUDA device, stream and event synchronization activities. This feature is new\n"
400 |           "       and currently disabled by default.\n"
401 |           "    adjust_profiler_step (bool) : whether to adjust the profiler step to\n"
402 |           "       match the parent python event duration. This feature is new and currently disabled by default.\n"
403 |           "    disable_external_correlation (bool) : whether to disable external correlation\n"
404 |           "    profile_all_threads (bool) : whether to profile all threads\n"
405 |           "    capture_overload_names (bool) : whether to include ATen overload names in the profile\n"
406 |           "    record_python_gc_info (bool) : adds python gc events to profile\n"
407 |           "    expose_kineto_event_metadata (bool) : whether to expose KinetoEvent metadata in the PyTorch Profiler\n"
408 |           "    custom_profiler_config (string) : Used to pass some configurations to the custom profiler backend.\n",
409 |           py::arg("profiler_metrics") = std::vector<std::string>(),
410 |           py::arg("profiler_measure_per_kernel") = false,
411 |           py::arg("verbose") = false,
412 |           py::arg("performance_events") = std::vector<std::string>(),
413 |           py::arg("enable_cuda_sync_events") = false,
414 |           py::arg("adjust_profiler_step") = false,
415 |           py::arg("disable_external_correlation") = false,
416 |           py::arg("profile_all_threads") = false,
417 |           py::arg("capture_overload_names") = false,
418 |           py::arg("record_python_gc_info") = false,
419 |           py::arg("expose_kineto_event_metadata") = false,
420 |           py::arg("custom_profiler_config") = "")
421 |       .def(py::pickle(
422 |           [](const ExperimentalConfig& p) { // __getstate__
423 |             py::list py_metrics;
424 |             for (const auto& metric : p.profiler_metrics) {
425 |               py::bytes mbytes(metric);
426 |               py_metrics.append(mbytes);
427 |             }
428 |             py::list py_perf_events;
429 |             for (const auto& event : p.performance_events) {
430 |               py::bytes mbytes(event);
431 |               py_perf_events.append(mbytes);
432 |             }
433 |             /* Return a tuple that fully encodes the state of the config */
434 |             return py::make_tuple(
435 |                 py_metrics,
436 |                 p.profiler_measure_per_kernel,
437 |                 p.verbose,
438 |                 py_perf_events,
```
- EN: Implements routines such as `mbytes` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `mbytes` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 439-479
```cpp
439 |                 p.enable_cuda_sync_events,
440 |                 p.adjust_profiler_step,
441 |                 p.disable_external_correlation,
442 |                 p.profile_all_threads,
443 |                 p.capture_overload_names,
444 |                 p.record_python_gc_info,
445 |                 p.expose_kineto_event_metadata,
446 |                 p.custom_profiler_config);
447 |           },
448 |           [](const py::tuple& t) { // __setstate__
449 |             TORCH_CHECK(t.size() >= 12, "Expected at least 12 values in state");
450 | 
451 |             py::list py_metrics = t[0].cast<py::list>();
452 |             std::vector<std::string> metrics;
453 |             metrics.reserve(py_metrics.size());
454 |             for (const auto& py_metric : py_metrics) {
455 |               metrics.push_back(py::str(py_metric));
456 |             }
457 | 
458 |             py::list py_perf_events = t[3].cast<py::list>();
459 |             std::vector<std::string> performance_events;
460 |             performance_events.reserve(py_perf_events.size());
461 |             for (const auto& py_perf_event : py_perf_events) {
462 |               performance_events.push_back(py::str(py_perf_event));
463 |             }
464 | 
465 |             return ExperimentalConfig(
466 |                 std::move(metrics),
467 |                 t[1].cast<bool>(),
468 |                 t[2].cast<bool>(),
469 |                 std::move(performance_events),
470 |                 t[4].cast<bool>(),
471 |                 t[5].cast<bool>(),
472 |                 t[6].cast<bool>(),
473 |                 t[7].cast<bool>(),
474 |                 t[8].cast<bool>(),
475 |                 t[9].cast<bool>(),
476 |                 t[10].cast<bool>(),
477 |                 t[11].cast<std::string>());
478 |           }));
479 | 
```
- EN: Implements routines such as `ExperimentalConfig` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `ExperimentalConfig` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 480-533
```cpp
480 |   py::class_<ProfilerConfig>(m, "ProfilerConfig")
481 |       .def(
482 |           py::init<
483 |               ProfilerState,
484 |               bool, /* report_input_shapes */
485 |               bool, /* profile_memory */
486 |               bool, /* with_stack */
487 |               bool, /* with_flops */
488 |               bool, /* with_modules */
489 |               ExperimentalConfig /* experimental_config */,
490 |               std::string /* trace_id */
491 |               >(),
492 |           py::arg("state"),
493 |           py::arg("report_input_shapes"),
494 |           py::arg("profile_memory"),
495 |           py::arg("with_stack"),
496 |           py::arg("with_flops"),
497 |           py::arg("with_modules"),
498 |           py::arg("experimental_config"),
499 |           py::arg("trace_id") = "" // Make trace_id the only optional param
500 |       );
501 | 
502 |   py::enum_<EventType>(m, "_EventType")
503 |       .value("TorchOp", EventType::TorchOp)
504 |       .value("Backend", EventType::Backend)
505 |       .value("Vulkan", EventType::Vulkan)
506 |       .value("Allocation", EventType::Allocation)
507 |       .value("PyCall", EventType::PyCall)
508 |       .value("PyCCall", EventType::PyCCall)
509 |       .value("Kineto", EventType::Kineto);
510 | 
511 |   py::class_<TensorMetadata>(m, "_TensorMetadata")
512 |       .def_property_readonly("impl_ptr", &TensorMetadata::impl)
513 |       .def_readonly("storage_data_ptr", &TensorMetadata::data_)
514 |       .def_readonly("id", &TensorMetadata::id_)
515 |       .def_readonly("allocation_id", &TensorMetadata::allocation_id_)
516 |       .def_property_readonly(
517 |           "layout",
518 |           [](const TensorMetadata& metadata) {
519 |             PyObject* layout_obj =
520 |                 torch::autograd::utils::wrap(metadata.layout_);
521 |             return py::reinterpret_borrow<py::object>(layout_obj);
522 |           })
523 |       .def_readonly("device", &TensorMetadata::device_)
524 |       .def_property_readonly(
525 |           "dtype",
526 |           [](const TensorMetadata& metadata) {
527 |             return py::reinterpret_borrow<py::object>(
528 |                 torch::autograd::utils::wrap(metadata.dtype_));
529 |           })
530 |       .def_readonly("dim", &TensorMetadata::size_dim_)
531 |       .def_readonly("sizes", &TensorMetadata::sizes_)
532 |       .def_readonly("strides", &TensorMetadata::strides_);
533 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 534-583
```cpp
534 |   using torch_op_t = ExtraFields<EventType::TorchOp>;
535 |   py::class_<torch_op_t>(m, "_ExtraFields_TorchOp")
536 |       .def_readonly("name", &torch_op_t::name_)
537 |       .def_property_readonly(
538 |           "inputs",
539 |           [](const torch_op_t& op) {
540 |             py::list out;
541 |             for (const auto& input : op.inputs_) {
542 |               std::visit(
543 |                   c10::overloaded(
544 |                       [&](const c10::IValue& v) {
545 |                         out.append(torch::jit::toPyObject(v));
546 |                       },
547 |                       [&](const std::nullopt_t&) { out.append(py::none()); },
548 |                       [&](const auto& v) { out.append(py::cast(v)); }),
549 |                   input);
550 |             }
551 |             return out;
552 |           })
553 |       .def_readonly("scope", &torch_op_t::scope_)
554 |       .def_readonly("sequence_number", &torch_op_t::sequence_number_)
555 |       .def_readonly("allow_tf32_cublas", &torch_op_t::allow_tf32_cublas_);
556 | 
557 |   // NOLINTNEXTLINE(bugprone-unused-raii)
558 |   py::class_<ExtraFields<EventType::Backend>>(m, "_ExtraFields_Backend");
559 |   // NOLINTNEXTLINE(bugprone-unused-raii)
560 |   py::class_<ExtraFields<EventType::Vulkan>>(m, "_ExtraFields_Vulkan");
561 | 
562 |   using allocation_t = ExtraFields<EventType::Allocation>;
563 |   py::class_<allocation_t>(m, "_ExtraFields_Allocation")
564 |       .def_property_readonly(
565 |           "ptr",
566 |           [](const allocation_t& a) {
567 |             return reinterpret_cast<intptr_t>(a.ptr_);
568 |           })
569 |       .def_readonly("id", &allocation_t::id_)
570 |       .def_readonly("allocation_id", &allocation_t::allocation_id_)
571 |       .def_readonly("alloc_size", &allocation_t::alloc_size_)
572 |       .def_readonly("total_allocated", &allocation_t::total_allocated_)
573 |       .def_readonly("total_reserved", &allocation_t::total_reserved_)
574 |       .def_property_readonly("device", &allocation_t::device);
575 | 
576 |   py::class_<PyFrameState>(m, "_PyFrameState")
577 |       .def_readonly("line_number", &PyFrameState::line_no_)
578 |       .def_property_readonly(
579 |           "file_name", [](const PyFrameState& s) { return s.filename_.str(); })
580 |       .def_property_readonly("function_name", [](const PyFrameState& s) {
581 |         return s.funcname_.str();
582 |       });
583 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 584-625
```cpp
584 |   py::class_<NNModuleInfo>(m, "_NNModuleInfo")
585 |       .def_property_readonly(
586 |           "parameters",
587 |           [](const NNModuleInfo& s) {
588 |             py::list out;
589 |             for (const auto& p : s.parameters_) {
590 |               out.append(
591 |                   py::make_tuple(p.name_, p.metadata_, p.grad_metadata_));
592 |             }
593 |             return out;
594 |           })
595 |       .def_property_readonly(
596 |           "cls_name", [](const NNModuleInfo& s) { return s.cls_name_.str(); })
597 |       .def_readonly("self_ptr", &NNModuleInfo::self_)
598 |       .def_readonly("cls_ptr", &NNModuleInfo::cls_);
599 | 
600 |   py::class_<OptimizerInfo>(m, "_OptimizerInfo")
601 |       .def_readonly("self_ptr", &OptimizerInfo::self_)
602 |       .def_property_readonly("parameters", [](const OptimizerInfo& s) {
603 |         py::list out;
604 |         for (const auto& p : s.parameters_) {
605 |           out.append(py::make_tuple(p.metadata_, p.grad_metadata_, p.state_));
606 |         }
607 |         return out;
608 |       });
609 | 
610 |   py::class_<ExtraFields<EventType::PyCall>>(m, "_ExtraFields_PyCall")
611 |       .def_readonly("callsite", &ExtraFields<EventType::PyCall>::callsite_)
612 |       .def_readonly("caller", &ExtraFields<EventType::PyCall>::caller_)
613 |       .def_readonly("module", &ExtraFields<EventType::PyCall>::module_)
614 |       .def_readonly("optimizer", &ExtraFields<EventType::PyCall>::optimizer_);
615 | 
616 |   py::class_<ExtraFields<EventType::PyCCall>>(m, "_ExtraFields_PyCCall")
617 |       .def_readonly("caller", &ExtraFields<EventType::PyCall>::caller_);
618 | 
619 |   // NOLINTNEXTLINE(bugprone-unused-raii)
620 |   py::class_<ExtraFields<EventType::OutOfMemory>>(
621 |       m, "_ExtraFields_OutOfMemory");
622 | 
623 |   // NOLINTNEXTLINE(bugprone-unused-raii)
624 |   py::class_<ExtraFields<EventType::Kineto>>(m, "_ExtraFields_Kineto");
625 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 626-677
```cpp
626 |   py::class_<Result, std::shared_ptr<Result>>(m, "_ProfilerEvent")
627 |       .def_property_readonly("name", &Result::name)
628 |       .def_property_readonly("overload_name", &Result::overload_name)
629 |       .def_property_readonly("tag", &Result::tag)
630 |       .def_readonly("extra_fields", &Result::extra_fields_)
631 |       .def_property_readonly(
632 |           "typed",
633 |           [](const Result& r) {
634 |             return py::make_tuple(
635 |                 r.tag(),
636 |                 py::cast(r.extra_fields_, py::return_value_policy::reference));
637 |           })
638 |       .def_property_readonly(
639 |           "id",
640 |           [](const Result& r) {
641 |             return reinterpret_cast<intptr_t>(r.shared_from_this().get());
642 |           })
643 |       .def_property_readonly(
644 |           "parent", [](const Result& r) { return r.parent_.lock(); })
645 |       .def_readonly("children", &Result::children_)
646 |       .def_readonly("start_time_ns", &Result::start_time_ns_)
647 |       .def_readonly("start_tid", &Result::start_tid_)
648 |       .def_property_readonly("correlation_id", &Result::correlationID)
649 |       .def_property_readonly("end_time_ns", &Result::endTimeNS)
650 |       .def_property_readonly("duration_time_ns", [](const Result& r) {
651 |         return r.endTimeNS() - r.start_time_ns_;
652 |       });
653 | 
654 |   // PyTorch profiler execution trace internal interface.
655 |   m.def(
656 |       "_add_execution_trace_observer",
657 |       &torch::profiler::impl::addExecutionTraceObserver,
658 |       py::arg("output_file_name"));
659 |   m.def(
660 |       "_remove_execution_trace_observer",
661 |       &torch::profiler::impl::removeExecutionTraceObserver);
662 |   m.def(
663 |       "_enable_execution_trace_observer",
664 |       &torch::profiler::impl::enableExecutionTraceObserver);
665 |   m.def(
666 |       "_disable_execution_trace_observer",
667 |       &torch::profiler::impl::disableExecutionTraceObserver);
668 |   m.def(
669 |       "_set_record_concrete_inputs_enabled_val",
670 |       &torch::profiler::impl::set_record_concrete_inputs_enabled_val);
671 |   m.def(
672 |       "_set_fwd_bwd_enabled_val",
673 |       &torch::profiler::impl::set_fwd_bwd_enabled_val);
674 |   m.def(
675 |       "_set_cuda_sync_enabled_val",
676 |       &torch::profiler::impl::set_cuda_sync_enabled_val);
677 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 678-730
```cpp
678 |   TORCH_CHECK(PyType_Ready(&THPCapturedTracebackType) >= 0);
679 |   PyModule_AddObject(
680 |       m.ptr(), "CapturedTraceback", (PyObject*)&THPCapturedTracebackType);
681 |   m.def(
682 |       "gather_traceback",
683 |       CapturedTraceback::gather,
684 |       py::arg("python") = true,
685 |       py::arg("script") = true,
686 |       py::arg("cpp") = true);
687 |   m.def("symbolize_tracebacks", [](const py::list& tbs) {
688 |     std::vector<CapturedTraceback*> tb_ptrs;
689 |     tb_ptrs.reserve(tbs.size());
690 |     for (py::handle tb : tbs) {
691 |       tb_ptrs.emplace_back(((THPCapturedTraceback*)tb.ptr())->data.get());
692 |     }
693 |     return py_symbolize(tb_ptrs);
694 |   });
695 |   // directly convert address pointers to frames, used for testing symbolize
696 |   m.def(
697 |       "symbolize_addresses",
698 |       [](const std::vector<uint64_t>& frames, const std::string& mode_s) {
699 |         std::vector<std::tuple<std::string, int64_t, std::string>> frames_out;
700 |         torch::unwind::Mode mode = torch::unwind::Mode::addr2line;
701 |         if (mode_s == "fast") {
702 |           mode = torch::unwind::Mode::fast;
703 |         } else if (mode_s == "addr2line") {
704 |           mode = torch::unwind::Mode::addr2line;
705 |         } else if (mode_s == "dladdr") {
706 |           mode = torch::unwind::Mode::dladdr;
707 |         } else {
708 |           TORCH_CHECK(false, "unexpected mode ", mode_s);
709 |         }
710 |         std::vector<void*> frames_p;
711 |         frames_p.reserve(frames.size());
712 |         for (auto f : frames) {
713 |           frames_p.push_back((void*)f); // NOLINT
714 |         }
715 |         auto frame_objects = unwind::symbolize(frames_p, mode);
716 |         frames_out.reserve(frame_objects.size());
717 |         for (auto& frame : frame_objects) {
718 |           frames_out.emplace_back(frame.filename, frame.lineno, frame.funcname);
719 |         }
720 |         return frames_out;
721 |       });
722 |   installCapturedTracebackPython();
723 | 
724 |   // NOLINTNEXTLINE(*-c-arrays*)
725 |   static PyMethodDef RecordFunctionFast_methods[] = {
726 |       {"__enter__", RecordFunctionFast_enter, METH_NOARGS, nullptr},
727 |       {"__exit__", RecordFunctionFast_exit, METH_VARARGS, nullptr},
728 |       {nullptr},
729 |   };
730 | 
```
- EN: Implements routines such as `py_symbolize` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `py_symbolize` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 731-756
```cpp
731 |   static PyTypeObject RecordFunctionFast_Type = {
732 |       PyVarObject_HEAD_INIT(nullptr, 0)
733 |   };
734 | 
735 |   RecordFunctionFast_Type.tp_name = "torch._C._profiler.RecordFunctionFast",
736 |   RecordFunctionFast_Type.tp_basicsize = sizeof(RecordFunctionFast);
737 |   RecordFunctionFast_Type.tp_dealloc = (destructor)RecordFunctionFast_dealloc;
738 |   RecordFunctionFast_Type.tp_flags = Py_TPFLAGS_DEFAULT;
739 |   RecordFunctionFast_Type.tp_methods = RecordFunctionFast_methods;
740 |   RecordFunctionFast_Type.tp_init = RecordFunctionFast_init;
741 |   RecordFunctionFast_Type.tp_new = RecordFunctionFast_new;
742 | 
743 |   if (PyType_Ready(&RecordFunctionFast_Type) < 0) {
744 |     throw python_error();
745 |   }
746 | 
747 |   Py_INCREF(&RecordFunctionFast_Type);
748 |   if (PyModule_AddObject(
749 |           m.ptr(),
750 |           "_RecordFunctionFast",
751 |           (PyObject*)&RecordFunctionFast_Type) != 0) {
752 |     Py_DECREF(&RecordFunctionFast_Type);
753 |     throw python_error();
754 |   }
755 | }
756 | } // namespace torch::profiler
```
- EN: Implements routines such as `python_error` that expose the key API or control flow of this region.
- CN: 实现了 `python_error` 等例程，它们构成了这一段的关键 API 或控制流程。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `THPCapturedTraceback`, `type_caster`, `RecordFunctionFast`.
  - CN: `THPCapturedTraceback`、`type_caster`、`RecordFunctionFast`。
- **Important routines / 重要例程**
  - EN: `THPCapturedTraceback_traverse`, `THPCapturedTraceback_clear`, `THPCapturedTraceback_dealloc`, `PYBIND11_TYPE_CASTER`, `load`, `cast`, `RecordFunctionFast_new`, `RecordFunctionFast_init`.
  - CN: `THPCapturedTraceback_traverse`、`THPCapturedTraceback_clear`、`THPCapturedTraceback_dealloc`、`PYBIND11_TYPE_CASTER`、`load`、`cast`、`RecordFunctionFast_new`、`RecordFunctionFast_init`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `pybind11::detail`, `torch::profiler`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `pybind11::detail`、`torch::profiler` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/python/init.h>`, `<ATen/record_function.h>`, `<c10/core/impl/PyInterpreter.h>`, `<c10/util/Exception.h>`, `<c10/util/overloaded.h>`, `<torch/csrc/DynamicTypes.h>`, `<torch/csrc/autograd/utils/wrap_outputs.h>`, `<torch/csrc/jit/python/pybind_utils.h>`, `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/python/combined_traceback.h>`, `<torch/csrc/profiler/standalone/execution_trace_observer.h>`, `<torch/csrc/utils/pybind.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
