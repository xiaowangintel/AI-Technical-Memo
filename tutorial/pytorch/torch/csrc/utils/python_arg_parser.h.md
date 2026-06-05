# python_arg_parser.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/python_arg_parser.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Bridges C++ components to Python-facing APIs or bindings.
  - CN: 把 C++ 组件桥接到面向 Python 的 API 或绑定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-78
```cpp
 1 | #pragma once
 2 | 
 3 | // Parse arguments to Python functions implemented in C++
 4 | // This is similar to PyArg_ParseTupleAndKeywords(), but specifically handles
 5 | // the types relevant to PyTorch and distinguishes between overloaded function
 6 | // signatures.
 7 | //
 8 | // Example:
 9 | //
10 | //   static PythonArgParser parser({
11 | //     "norm(Scalar p, int64_t dim, bool keepdim=False)",
12 | //     "norm(Scalar p=2)",
13 | //   });
14 | //   ParsedArgs<3> parsed_args;
15 | //   auto r = parser.parse(args, kwargs, parsed_args);
16 | //   if (r.idx == 0) {
17 | //     norm(r.scalar(0), r.int64(1), r.bool(0));
18 | //   } else {
19 | //     norm(r.scalar(0));
20 | //   }
21 | //
22 | // We auto-generate most uses of PythonArgParser; the generated files
23 | // are torch/csrc/autograd/generated/python_*.cpp
24 | //
25 | // Some gotchas that you should watch out for:
26 | //
27 | //    - Note [Order of overloads matters]
28 | //      Order of overloads matters.  A set of input arguments may
29 | //      bind to multiple argument specs; we will always pick the
30 | //      first one in PythonArgParser.  However, when you are writing
31 | //      overloads in, e.g., native_functions.yaml, you don't have to
32 | //      worry about what order you write them, because the code
33 | //      generation logic always gives the overloads a canonical
34 | //      order, where Tensor overloads come first, before Scalar overloads.
35 | //      This logic is in sort_declarations in
36 | //      tools/autograd/gen_python_functions.py
37 | //
38 | //    - Zero-dim tensors (e.g., torch.tensor(2)) bind to both
39 | //      Scalar and Tensor, UNLESS they require grad (in which case
40 | //      they only bind to Tensor).
41 | 
42 | #include <fmt/format.h>
43 | #include <pybind11/pytypes.h>
44 | #include <torch/csrc/python_headers.h>
45 | 
46 | #include <torch/csrc/Device.h>
47 | #include <torch/csrc/Dtype.h>
48 | #include <torch/csrc/DynamicTypes.h>
49 | #include <torch/csrc/Exceptions.h>
50 | #include <torch/csrc/Export.h>
51 | #include <torch/csrc/Generator.h>
52 | #include <torch/csrc/Layout.h>
53 | #include <torch/csrc/MemoryFormat.h>
54 | #include <torch/csrc/QScheme.h>
55 | #include <torch/csrc/Stream.h>
56 | #include <torch/csrc/autograd/python_variable.h>
57 | #include <torch/csrc/autograd/variable.h>
58 | #include <torch/csrc/dynamo/eval_frame.h>
59 | #include <torch/csrc/jit/frontend/tracer.h>
60 | #include <torch/csrc/python_dimname.h>
61 | #include <torch/csrc/tensor/python_tensor.h>
62 | #include <torch/csrc/utils/disable_torch_function.h>
63 | #include <torch/csrc/utils/object_ptr.h>
64 | #include <torch/csrc/utils/pybind.h>
65 | #include <torch/csrc/utils/python_numbers.h>
66 | #include <torch/csrc/utils/python_strings.h>
67 | #include <torch/csrc/utils/python_symnode.h>
68 | #include <torch/csrc/utils/structseq.h>
69 | 
70 | #include <ATen/DeviceAccelerator.h>
71 | #include <ATen/PythonTorchFunctionTLS.h>
72 | #include <ATen/core/Tensor.h>
73 | #include <c10/util/Exception.h>
74 | #include <c10/util/irange.h>
75 | 
76 | #include <c10/core/SymFloat.h>
77 | #include <c10/core/SymNodeImpl.h>
78 | 
```
- EN: Brings in project headers such as `<torch/csrc/python_headers.h>`, `<torch/csrc/Device.h>`, `<torch/csrc/Dtype.h>`, `<torch/csrc/DynamicTypes.h>` and system or third-party headers such as `<fmt/format.h>`, `<pybind11/pytypes.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/python_headers.h>`、`<torch/csrc/Device.h>`、`<torch/csrc/Dtype.h>`、`<torch/csrc/DynamicTypes.h>`以及系统或第三方头文件，例如 `<fmt/format.h>`、`<pybind11/pytypes.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。

### Lines 79-157
```cpp
 79 | #include <c10/core/DispatchKeySet.h>
 80 | #include <array>
 81 | #include <cstddef>
 82 | #include <string>
 83 | #include <vector>
 84 | 
 85 | inline bool THPUtils_checkScalar(PyObject* obj) {
 86 | #ifdef USE_NUMPY
 87 |   if (torch::utils::is_numpy_scalar(obj)) {
 88 |     return true;
 89 |   }
 90 | #endif
 91 |   return PyFloat_Check(obj) || PyLong_Check(obj) || PyComplex_Check(obj) ||
 92 |       torch::is_symint(py::handle(obj)) || torch::is_dynint(py::handle(obj)) ||
 93 |       torch::is_symfloat(py::handle(obj)) || torch::is_symbool(py::handle(obj));
 94 | }
 95 | 
 96 | namespace torch {
 97 | 
 98 | TORCH_PYTHON_API bool should_allow_numbers_as_tensors(const std::string& name);
 99 | 
100 | enum class ParameterType {
101 |   TENSOR,
102 |   SCALAR,
103 |   INT64,
104 |   SYM_INT,
105 |   DOUBLE,
106 |   COMPLEX,
107 |   TENSOR_LIST,
108 |   INT_LIST,
109 |   GENERATOR,
110 |   BOOL,
111 |   STORAGE,
112 |   PYOBJECT,
113 |   SCALARTYPE,
114 |   LAYOUT,
115 |   MEMORY_FORMAT,
116 |   DEVICE,
117 |   STREAM,
118 |   STRING,
119 |   DIMNAME,
120 |   DIMNAME_LIST,
121 |   QSCHEME,
122 |   FLOAT_LIST,
123 |   SCALAR_LIST,
124 |   SYM_INT_LIST,
125 |   DISPATCH_KEY_SET
126 | };
127 | 
128 | struct PythonArgs;
129 | 
130 | // Contains bound Python arguments in declaration order
131 | template <int N>
132 | struct ParsedArgs {
133 |   ParsedArgs() : args() {}
134 |   // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
135 |   PyObject* args[N];
136 | };
137 | 
138 | // FunctionParameter is a single formal parameter of a Python function.
139 | // It is immutable once constructed.
140 | struct FunctionParameter {
141 |   FunctionParameter(const std::string& fmt, bool keyword_only);
142 | 
143 |   bool check(
144 |       PyObject* obj,
145 |       std::vector<PyObject*>& overloaded_args,
146 |       int argnum,
147 |       int64_t* failed_idx = nullptr);
148 | 
149 |   bool _check(
150 |       PyObject* obj,
151 |       std::vector<PyObject*>& overloaded_args,
152 |       int argnum,
153 |       int64_t* failed_idx = nullptr);
154 | 
155 |   void set_default_str(const std::string& str);
156 |   TORCH_PYTHON_API std::string type_name() const;
157 | 
```
- EN: Brings in project headers such as `<c10/core/DispatchKeySet.h>` and system or third-party headers such as `<array>`, `<cstddef>`, `<string>`, `<vector>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `ParameterType`, `PythonArgs`, `ParsedArgs`, `FunctionParameter` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/core/DispatchKeySet.h>`以及系统或第三方头文件，例如 `<array>`、`<cstddef>`、`<string>`、`<vector>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `ParameterType`、`PythonArgs`、`ParsedArgs`、`FunctionParameter` 等数据抽象，用来组织本文件处理的状态。

### Lines 158-235
```cpp
158 |   ParameterType type_;
159 |   bool optional{false};
160 |   bool allow_none{false};
161 |   bool keyword_only;
162 |   bool allow_numbers_as_tensors = false;
163 |   int size{0};
164 |   std::string name;
165 |   // having this as a raw PyObject * will presumably leak it, but these are only
166 |   // held by static objects anyway, and Py_Finalize can already be called when
167 |   // this is destructed.
168 |   PyObject* python_name;
169 |   // NOLINTNEXTLINE(cppcoreguidelines-avoid-magic-numbers)
170 |   at::SmallVector<PyObject*, 5> numpy_python_names;
171 |   at::Scalar default_scalar;
172 |   std::vector<int64_t> default_intlist;
173 |   std::string default_string;
174 |   union {
175 |     bool default_bool;
176 |     int64_t default_int;
177 |     double default_double;
178 |     // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
179 |     double default_complex[2]; // see Scalar
180 |     at::ScalarType default_scalartype;
181 |     at::Layout default_layout;
182 |   };
183 |   std::string default_value;
184 | };
185 | 
186 | // FunctionSignature represents a single valid signature for a Python function.
187 | // It is immutable once constructed. The contained data can be concurrently
188 | // accessed by multiple calls.
189 | struct FunctionSignature {
190 |   explicit FunctionSignature(const std::string& fmt, int index);
191 | 
192 |   bool parse(
193 |       PyObject* self,
194 |       PyObject* args,
195 |       PyObject* kwargs,
196 |       // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
197 |       PyObject* dst[],
198 |       std::vector<PyObject*>& overloaded_args,
199 |       bool raise_exception);
200 | 
201 |   std::string toString() const;
202 | 
203 |   std::string name;
204 |   std::vector<FunctionParameter> params;
205 |   size_t min_args{0};
206 |   size_t max_args{0};
207 |   size_t max_pos_args{0};
208 |   int index;
209 |   bool hidden{false};
210 |   bool deprecated{false};
211 | };
212 | 
213 | // A PythonArgParser contains a list of valid signatures. Instances are
214 | // typically global variables and should be immutable.
215 | struct PYBIND11_EXPORT PythonArgParser {
216 |   explicit PythonArgParser(
217 |       const std::vector<std::string>& fmts,
218 |       bool traceable = false);
219 | 
220 |   // meant only for `torch` functions.
221 |   template <int N>
222 |   inline PythonArgs parse(
223 |       PyObject* self,
224 |       PyObject* args,
225 |       PyObject* kwargs,
226 |       ParsedArgs<N>& dst);
227 | 
228 |   template <int N>
229 |   inline PythonArgs parse(PyObject* args, PyObject* kwargs, ParsedArgs<N>& dst);
230 | 
231 |   inline PythonArgs parse(PyObject* self, ParsedArgs<0>& dst);
232 | 
233 |   // Formatted strings of non-hidden signatures
234 |   std::vector<std::string> get_signatures() const;
235 | 
```
- EN: Defines or extends data abstractions such as `FunctionSignature`, `PythonArgParser` that structure the state handled by this file. Implements routines such as `FunctionSignature`, `parse`, `toString`, `PythonArgParser`, `get_signatures` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 定义或扩展了 `FunctionSignature`、`PythonArgParser` 等数据抽象，用来组织本文件处理的状态。 实现了 `FunctionSignature`、`parse`、`toString`、`PythonArgParser`、`get_signatures` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

### Lines 236-307
```cpp
236 |  private:
237 |   [[noreturn]] void print_error(
238 |       PyObject* self,
239 |       PyObject* args,
240 |       PyObject* kwargs,
241 |       // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
242 |       PyObject* parsed_args[]);
243 |   void check_deprecated(const FunctionSignature& signature);
244 |   PythonArgs raw_parse(
245 |       PyObject* self,
246 |       PyObject* args,
247 |       PyObject* kwargs,
248 |       // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
249 |       PyObject* parsed_args[]);
250 | 
251 |   std::vector<FunctionSignature> signatures_;
252 |   std::string function_name;
253 |   size_t max_args{0};
254 |   bool traceable;
255 | };
256 | 
257 | // PythonArgs contains bound Python arguments for an actual invocation
258 | // along with references to the matched signature.
259 | struct TORCH_PYTHON_API PythonArgs {
260 |   PythonArgs(
261 |       bool traceable,
262 |       const FunctionSignature& signature,
263 |       PyObject** args,
264 |       std::vector<PyObject*> overloaded_args)
265 |       : idx(signature.index),
266 |         traceable(traceable),
267 |         signature(signature),
268 |         args(args),
269 |         overloaded_args(std::move(overloaded_args)) {}
270 | 
271 |   int idx;
272 |   bool traceable;
273 |   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
274 |   const FunctionSignature& signature;
275 |   PyObject** args;
276 |   std::vector<PyObject*> overloaded_args; // NOTE: borrowed references
277 | 
278 |   inline bool has_torch_function();
279 |   inline std::string get_func_name();
280 |   inline at::Tensor tensor(int i);
281 |   inline std::optional<at::Tensor> optionalTensor(int i);
282 |   inline at::Scalar scalar(int i);
283 |   inline at::Scalar scalarWithDefault(int i, const at::Scalar& default_scalar);
284 |   inline std::vector<at::Scalar> scalarlist(int i);
285 |   inline std::vector<at::Tensor> tensorlist(int i);
286 |   inline torch::List<std::optional<at::Tensor>> list_of_optional_tensors(int i);
287 |   template <int N>
288 |   inline std::array<at::Tensor, N> tensorlist_n(int i);
289 |   inline std::vector<int64_t> intlist(int i);
290 |   inline std::vector<c10::SymInt> symintlist(int i);
291 |   inline c10::OptionalArray<int64_t> intlistOptional(int i);
292 |   inline c10::OptionalArray<c10::SymInt> symintlistOptional(int i);
293 |   inline std::vector<int64_t> intlistWithDefault(
294 |       int i,
295 |       std::vector<int64_t> default_intlist);
296 |   inline std::optional<at::Generator> generator(int i);
297 |   inline at::Storage storage(int i);
298 |   inline at::Storage storage(
299 |       int i,
300 |       at::ScalarType& storage_scalar_type,
301 |       bool& is_typed_storage);
302 |   inline c10::Stream stream(int i);
303 |   inline at::ScalarType scalartype(int i);
304 |   inline at::ScalarType scalartypeWithDefault(
305 |       int i,
306 |       at::ScalarType default_scalartype);
307 |   inline std::optional<at::ScalarType> scalartypeOptional(int i);
```
- EN: Defines or extends data abstractions such as `PythonArgs` that structure the state handled by this file. Implements routines such as `check_deprecated`, `raw_parse`, `has_torch_function`, `get_func_name`, `tensor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `PythonArgs` 等数据抽象，用来组织本文件处理的状态。 实现了 `check_deprecated`、`raw_parse`、`has_torch_function`、`get_func_name`、`tensor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 308-387
```cpp
308 |   inline std::optional<at::Scalar> scalarOptional(int i);
309 |   inline std::optional<int64_t> toInt64Optional(int i);
310 |   inline std::optional<c10::SymInt> toSymIntOptional(int i);
311 |   inline std::optional<bool> toBoolOptional(int i);
312 |   inline std::optional<double> toDoubleOptional(int i);
313 |   inline c10::OptionalArray<double> doublelistOptional(int i);
314 |   inline std::vector<double> doublelist(int i);
315 |   inline std::vector<double> getDoublelist(int i);
316 |   inline at::Layout layout(int i);
317 |   inline at::Layout layoutWithDefault(int i, at::Layout default_layout);
318 |   inline std::optional<at::Layout> layoutOptional(int i);
319 |   inline at::Device device(int i);
320 |   inline at::Device deviceWithDefault(int i, const at::Device& default_device);
321 |   inline std::optional<at::Device> deviceOptional(int i);
322 |   inline at::Dimname dimname(int i);
323 |   inline std::vector<at::Dimname> dimnamelist(int i);
324 |   inline std::optional<std::vector<at::Dimname>> toDimnameListOptional(int i);
325 |   inline at::MemoryFormat memoryformat(int i);
326 |   inline std::optional<at::MemoryFormat> memoryformatOptional(int i);
327 |   inline at::QScheme toQScheme(int i);
328 |   inline std::string string(int i);
329 |   inline std::string stringWithDefault(int i, const std::string& default_str);
330 |   inline std::optional<std::string> stringOptional(int i);
331 |   inline std::string_view stringView(int i);
332 |   inline std::string_view stringViewWithDefault(
333 |       int i,
334 |       const std::string_view default_str);
335 |   inline std::optional<std::string_view> stringViewOptional(int i);
336 |   inline PyObject* pyobject(int i);
337 |   inline int64_t toInt64(int i);
338 |   inline c10::SymInt toSymInt(int i);
339 |   inline c10::SymBool toSymBool(int i);
340 |   inline int64_t toInt64WithDefault(int i, int64_t default_int);
341 |   inline double toDouble(int i);
342 |   inline double toDoubleWithDefault(int i, double default_double);
343 |   inline c10::complex<double> toComplex(int i);
344 |   inline c10::complex<double> toComplexWithDefault(
345 |       int i,
346 |       c10::complex<double> default_complex);
347 |   inline bool toBool(int i);
348 |   inline bool toBoolWithDefault(int i, bool default_bool);
349 |   inline bool isNone(int i);
350 |   inline std::optional<c10::DispatchKeySet> toDispatchKeySetOptional(int i);
351 | 
352 |  private:
353 |   // Non-inline functions' symbols are exposed to torch_python DLL
354 |   // via TORCH_PYTHON_API tag at struct level.
355 |   at::Tensor tensor_slow(int i);
356 |   at::Scalar scalar_slow(int i);
357 |   at::Scalar scalar_slow(PyObject* arg);
358 | };
359 | 
360 | template <int N>
361 | inline PythonArgs PythonArgParser::parse(
362 |     PyObject* self,
363 |     PyObject* args,
364 |     PyObject* kwargs,
365 |     ParsedArgs<N>& dst) {
366 |   TORCH_CHECK_VALUE(
367 |       N >= max_args,
368 |       "PythonArgParser: dst ParsedArgs buffer does not have enough capacity, expected ",
369 |       max_args,
370 |       " (got ",
371 |       N,
372 |       ")");
373 |   return raw_parse(self, args, kwargs, dst.args);
374 | }
375 | 
376 | template <int N>
377 | inline PythonArgs PythonArgParser::parse(
378 |     PyObject* args,
379 |     PyObject* kwargs,
380 |     ParsedArgs<N>& dst) {
381 |   return parse(nullptr, args, kwargs, dst);
382 | }
383 | 
384 | inline PythonArgs PythonArgParser::parse(PyObject* self, ParsedArgs<0>& dst) {
385 |   return parse(self, nullptr, nullptr, dst);
386 | }
387 | 
```
- EN: Defines or extends data abstractions such as `level` that structure the state handled by this file. Implements routines such as `scalarOptional`, `toInt64Optional`, `toSymIntOptional`, `toBoolOptional`, `toDoubleOptional` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 定义或扩展了 `level` 等数据抽象，用来组织本文件处理的状态。 实现了 `scalarOptional`、`toInt64Optional`、`toSymIntOptional`、`toBoolOptional`、`toDoubleOptional` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 388-467
```cpp
388 | inline bool PythonArgs::has_torch_function() {
389 |   return !overloaded_args.empty() || at::impl::torch_function_mode_enabled();
390 | }
391 | 
392 | inline std::string PythonArgs::get_func_name() {
393 |   return signature.name;
394 | }
395 | 
396 | // TODO: this can return MaybeOwned
397 | inline at::Tensor PythonArgs::tensor(int i) {
398 |   if (args[i] && THPVariable_CheckExact(args[i])) {
399 |     return THPVariable_Unpack(args[i]);
400 |   }
401 |   return tensor_slow(i);
402 | }
403 | 
404 | inline std::optional<at::Tensor> PythonArgs::optionalTensor(int i) {
405 |   at::Tensor t = tensor(i);
406 |   // NOLINTNEXTLINE(bugprone-branch-clone)
407 |   if (t.defined()) {
408 |     return t;
409 |   } else {
410 |     return std::nullopt;
411 |   }
412 | }
413 | 
414 | inline at::Scalar PythonArgs::scalar(int i) {
415 |   if (!args[i])
416 |     return signature.params[i].default_scalar;
417 |   return scalar_slow(i);
418 | }
419 | 
420 | inline std::vector<at::Scalar> PythonArgs::scalarlist(int i) {
421 |   if (!args[i])
422 |     return std::vector<at::Scalar>();
423 |   auto tuple = PyTuple_Check(args[i]);
424 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(tuple || PyList_Check(args[i]));
425 |   // NOLINTNEXTLINE(bugprone-branch-clone)
426 |   auto size = tuple ? PyTuple_GET_SIZE(args[i]) : PyList_GET_SIZE(args[i]);
427 |   std::vector<at::Scalar> res(size);
428 |   for (const auto idx : c10::irange(size)) {
429 |     PyObject* obj =
430 |         tuple ? PyTuple_GET_ITEM(args[i], idx) : PyList_GET_ITEM(args[i], idx);
431 |     res[idx] = scalar_slow(obj);
432 |   }
433 |   return res;
434 | }
435 | 
436 | inline at::Scalar PythonArgs::scalarWithDefault(
437 |     int i,
438 |     const at::Scalar& default_scalar) {
439 |   if (!args[i])
440 |     return default_scalar;
441 |   return scalar_slow(i);
442 | }
443 | 
444 | inline std::optional<at::Scalar> PythonArgs::scalarOptional(int i) {
445 |   if (!args[i])
446 |     return std::nullopt;
447 |   return scalar_slow(i);
448 | }
449 | 
450 | inline std::vector<at::Tensor> PythonArgs::tensorlist(int i) {
451 |   if (!args[i])
452 |     return std::vector<at::Tensor>();
453 |   auto tuple = PyTuple_Check(args[i]);
454 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(tuple || PyList_Check(args[i]));
455 |   // NOLINTNEXTLINE(bugprone-branch-clone)
456 |   auto size = tuple ? PyTuple_GET_SIZE(args[i]) : PyList_GET_SIZE(args[i]);
457 |   std::vector<at::Tensor> res(size);
458 |   for (const auto idx : c10::irange(size)) {
459 |     PyObject* obj =
460 |         tuple ? PyTuple_GET_ITEM(args[i], idx) : PyList_GET_ITEM(args[i], idx);
461 |     // This is checked by the argument parser so it's safe to cast without
462 |     // checking if this is a tensor first
463 |     res[idx] = THPVariable_Unpack(obj);
464 |   }
465 |   return res;
466 | }
467 | 
```
- EN: Implements routines such as `THPVariable_Unpack`, `tensor_slow`, `scalar_slow`, `res` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `THPVariable_Unpack`、`tensor_slow`、`scalar_slow`、`res` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 468-547
```cpp
468 | inline torch::List<std::optional<at::Tensor>> PythonArgs::
469 |     list_of_optional_tensors(int i) {
470 |   if (!args[i])
471 |     return torch::List<std::optional<at::Tensor>>();
472 |   auto tuple = PyTuple_Check(args[i]);
473 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(tuple || PyList_Check(args[i]));
474 |   // NOLINTNEXTLINE(bugprone-branch-clone)
475 |   auto size = tuple ? PyTuple_GET_SIZE(args[i]) : PyList_GET_SIZE(args[i]);
476 |   torch::List<std::optional<at::Tensor>> res;
477 |   res.reserve(size);
478 |   for (const auto idx : c10::irange(size)) {
479 |     PyObject* obj =
480 |         tuple ? PyTuple_GET_ITEM(args[i], idx) : PyList_GET_ITEM(args[i], idx);
481 |     // This is checked by the argument parser so it's safe to cast without
482 |     // checking if this is a tensor first
483 |     res.push_back(THPVariable_Unpack(obj));
484 |   }
485 |   return res;
486 | }
487 | 
488 | template <int N>
489 | inline std::array<at::Tensor, N> PythonArgs::tensorlist_n(int i) {
490 |   auto res = std::array<at::Tensor, N>();
491 |   if (!args[i])
492 |     return res;
493 |   auto tuple = PyTuple_Check(args[i]);
494 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(tuple || PyList_Check(args[i]));
495 |   // NOLINTNEXTLINE(bugprone-branch-clone)
496 |   auto size = tuple ? PyTuple_GET_SIZE(args[i]) : PyList_GET_SIZE(args[i]);
497 |   if (size != N) {
498 |     TORCH_CHECK_TYPE(
499 |         false,
500 |         fmt::format("expected tuple of {} elements but got {}", N, size));
501 |   }
502 |   for (const auto idx : c10::irange(size)) {
503 |     PyObject* obj =
504 |         tuple ? PyTuple_GET_ITEM(args[i], idx) : PyList_GET_ITEM(args[i], idx);
505 |     // This is checked by the argument parser so it's safe to cast without
506 |     // checking if this is a tensor first
507 |     res[idx] = THPVariable_Unpack(obj);
508 |   }
509 |   return res;
510 | }
511 | 
512 | inline std::vector<int64_t> PythonArgs::intlist(int i) {
513 |   return intlistWithDefault(i, signature.params[i].default_intlist);
514 | }
515 | 
516 | inline PyObject* toPyObject(const c10::SymInt& symint) {
517 |   if (symint.is_symbolic()) {
518 |     auto r = py::cast(symint).release().ptr();
519 |     TORCH_INTERNAL_ASSERT(r);
520 |     return r;
521 |   } else {
522 |     auto m = symint.maybe_as_int();
523 |     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
524 |     return THPUtils_packInt64(m.value());
525 |   }
526 | }
527 | 
528 | inline void throw_intlist_exception(
529 |     const torch::PythonArgs* args,
530 |     size_t i,
531 |     PyObject* obj,
532 |     size_t idx,
533 |     const std::exception& e = python_error()) {
534 |   std::string error = strlen(e.what())
535 |       ? e.what()
536 |       : std::string("type must be ") + args->signature.params[i].type_name() +
537 |           ",but got " + Py_TYPE(obj)->tp_name;
538 |   TORCH_CHECK_TYPE(
539 |       false,
540 |       fmt::format(
541 |           "{}(): argument '{}' failed to unpack the object at pos {} with error \"{}\"",
542 |           args->signature.name,
543 |           args->signature.params[i].name,
544 |           idx + 1,
545 |           error));
546 | }
547 | 
```
- EN: Implements routines such as `list_of_optional_tensors`, `intlistWithDefault`, `toPyObject`, `THPUtils_packInt64`, `throw_intlist_exception` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `list_of_optional_tensors`、`intlistWithDefault`、`toPyObject`、`THPUtils_packInt64`、`throw_intlist_exception` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 548-619
```cpp
548 | inline std::vector<c10::SymInt> PythonArgs::symintlist(int i) {
549 |   if (!args[i]) {
550 |     return c10::fmap(signature.params[i].default_intlist, [](int64_t di) {
551 |       return c10::SymInt(di);
552 |     });
553 |   }
554 | 
555 |   const auto size1 = signature.params[i].size;
556 |   if (size1 > 0 && THPUtils_checkLong(args[i])) {
557 |     return std::vector<c10::SymInt>(
558 |         size1, c10::SymInt(THPUtils_unpackLong(args[i])));
559 |   }
560 | 
561 |   if (size1 > 0 && torch::is_symint(py::handle(args[i]))) {
562 |     auto si = py::handle(args[i]).cast<c10::SymInt>();
563 |     return std::vector<c10::SymInt>(size1, si);
564 |   }
565 | 
566 |   if (size1 > 0 && THPVariable_Check(args[i])) {
567 |     return std::vector<c10::SymInt>(
568 |         size1, THPVariable_Unpack(args[i]).item().toSymInt());
569 |   }
570 | 
571 |   PyObject* arg = args[i];
572 |   auto tuple = PyTuple_Check(arg);
573 |   if (!tuple) {
574 |     TORCH_INTERNAL_ASSERT(PyList_Check(arg), "expected tuple or list");
575 |   }
576 |   // NOLINTNEXTLINE(bugprone-branch-clone)
577 |   const auto size2 = tuple ? PyTuple_GET_SIZE(arg) : PyList_GET_SIZE(arg);
578 |   std::vector<c10::SymInt> res;
579 |   res.reserve(size2);
580 |   for (const auto idx : c10::irange(size2)) {
581 |     PyObject* obj =
582 |         tuple ? PyTuple_GET_ITEM(arg, idx) : PyList_GET_ITEM(arg, idx);
583 | 
584 |     // Elements of torch.Size are tensors during tracing, and we need to
585 |     // record extra information before they are turned into an IntArrayRef
586 |     if (traceable && jit::tracer::isTracing() && THPVariable_Check(obj)) {
587 |       auto& var = THPVariable_Unpack(obj);
588 |       jit::tracer::ArgumentStash::stashIntArrayRefElem(
589 |           signature.params[i].name, size2, idx, var);
590 |       try {
591 |         res.emplace_back(var.item<int64_t>());
592 |         continue;
593 |       } catch (std::exception& e) {
594 |         throw_intlist_exception(this, i, obj, idx, e);
595 |       }
596 |       continue;
597 |     } else {
598 |       // convert tensor to scalar outside of try / catch,
599 |       // so that Tensor subclass exceptions will not be caught.
600 |       if (THPUtils_checkLongExact(obj)) {
601 |         // Fast path for plain numbers
602 |         try {
603 |           res.emplace_back(THPUtils_unpackLong(obj));
604 |         } catch (std::exception& e) {
605 |           throw_intlist_exception(this, i, obj, idx, e);
606 |         }
607 |       } else if (THPVariable_Check(obj)) {
608 |         auto& var = THPVariable_Unpack(obj);
609 |         if (var.numel() != 1 ||
610 |             !at::isIntegralType(
611 |                 var.dtype().toScalarType(), /*include_bool*/ true)) {
612 |           throw_intlist_exception(this, i, obj, idx);
613 |         }
614 |         auto scalar = var.item();
615 |         TORCH_CHECK(scalar.isIntegral(/*include bool*/ false));
616 |         res.push_back(scalar.toSymInt());
617 |       } else {
618 |         try {
619 |           if (is_symint(py::handle(obj))) {
```
- EN: Implements routines such as `THPVariable_Unpack` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `THPVariable_Unpack` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 620-691
```cpp
620 |             res.push_back(py::handle(obj).cast<c10::SymInt>());
621 |           } else if (is_dynint(py::handle(obj))) {
622 |             res.emplace_back(py::handle(obj).cast<int>());
623 |           } else {
624 |             res.emplace_back(THPUtils_unpackIndex(obj));
625 |           }
626 |         } catch (std::exception& e) {
627 |           throw_intlist_exception(this, i, obj, idx, e);
628 |         }
629 |       }
630 |     }
631 |   }
632 | 
633 |   return res;
634 | }
635 | 
636 | inline std::vector<int64_t> PythonArgs::intlistWithDefault(
637 |     int i,
638 |     std::vector<int64_t> default_intlist) {
639 |   if (!args[i])
640 |     return default_intlist;
641 |   PyObject* arg = args[i];
642 |   const auto size1 = signature.params[i].size;
643 |   if (size1 > 0 && THPUtils_checkLong(arg)) {
644 |     return std::vector<int64_t>(size1, THPUtils_unpackLong(arg));
645 |   }
646 |   if (size1 > 0 && torch::is_symint(py::handle(arg))) {
647 |     return std::vector<int64_t>(
648 |         size1,
649 |         py::handle(arg).cast<c10::SymInt>().guard_int(__FILE__, __LINE__));
650 |   }
651 |   if (size1 > 0 && torch::is_dynint(py::handle(arg))) {
652 |     return std::vector<int64_t>(size1, py::handle(arg).cast<int>());
653 |   }
654 |   if (size1 > 0 && THPVariable_Check(arg)) {
655 |     return std::vector<int64_t>(size1, THPVariable_Unpack(arg).item<int64_t>());
656 |   }
657 |   auto tuple = PyTuple_Check(arg);
658 |   if (!tuple) {
659 |     TORCH_INTERNAL_ASSERT(PyList_Check(arg), "expected tuple or list");
660 |   }
661 |   // NOLINTNEXTLINE(bugprone-branch-clone)
662 |   const auto size2 = tuple ? PyTuple_GET_SIZE(arg) : PyList_GET_SIZE(arg);
663 |   std::vector<int64_t> res(size2);
664 |   for (const auto idx : c10::irange(size2)) {
665 |     PyObject* obj =
666 |         tuple ? PyTuple_GET_ITEM(arg, idx) : PyList_GET_ITEM(arg, idx);
667 |     // Elements of torch.Size are tensors during tracing, and we need to
668 |     // record extra information before they are turned into an IntArrayRef
669 |     if (traceable && jit::tracer::isTracing() && THPVariable_Check(obj)) {
670 |       auto& var = THPVariable_Unpack(obj);
671 |       jit::tracer::ArgumentStash::stashIntArrayRefElem(
672 |           signature.params[i].name, size2, idx, var);
673 |       try {
674 |         res[idx] = var.item<int64_t>();
675 |         continue;
676 |       } catch (std::exception& e) {
677 |         throw_intlist_exception(this, i, obj, idx, e);
678 |       }
679 |     } else {
680 |       // convert tensor to scalar outside of try / catch,
681 |       // so that Tensor subclass exceptions will not be caught.
682 |       if (THPUtils_checkLongExact(obj)) {
683 |         // Fast path for plain numbers
684 |         try {
685 |           res[idx] = THPUtils_unpackLong(obj);
686 |         } catch (std::exception& e) {
687 |           throw_intlist_exception(this, i, obj, idx, e);
688 |         }
689 |       } else if (torch::is_symint(py::handle(obj))) {
690 |         res[idx] = py::cast<c10::SymInt>(py::handle(obj))
691 |                        .guard_int(__FILE__, __LINE__);
```
- EN: Implements routines such as `res` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `res` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 692-768
```cpp
692 |       } else if (torch::is_dynint(py::handle(obj))) {
693 |         res[idx] = py::handle(obj).cast<int>();
694 |       } else if (THPVariable_Check(obj)) {
695 |         auto& var = THPVariable_Unpack(obj);
696 |         if (var.numel() != 1 ||
697 |             !at::isIntegralType(
698 |                 var.dtype().toScalarType(), /*include_bool*/ true)) {
699 |           throw_intlist_exception(this, i, obj, idx);
700 |         }
701 |         res[idx] = var.item<int64_t>();
702 |       } else {
703 |         try {
704 |           res[idx] = THPUtils_unpackIndex(obj);
705 |         } catch (std::exception& e) {
706 |           throw_intlist_exception(this, i, obj, idx, e);
707 |         }
708 |       }
709 |     }
710 |   }
711 |   return res;
712 | }
713 | 
714 | inline c10::OptionalArray<int64_t> PythonArgs::intlistOptional(int i) {
715 |   if (!args[i]) {
716 |     return {};
717 |   }
718 |   return intlist(i);
719 | }
720 | 
721 | inline c10::OptionalArray<c10::SymInt> PythonArgs::symintlistOptional(int i) {
722 |   if (!args[i]) {
723 |     return {};
724 |   }
725 |   return symintlist(i);
726 | }
727 | 
728 | inline std::vector<double> PythonArgs::getDoublelist(int i) {
729 |   PyObject* arg = args[i];
730 |   auto tuple = PyTuple_Check(arg);
731 |   if (!tuple) {
732 |     TORCH_INTERNAL_ASSERT(PyList_Check(arg), "expected tuple or list");
733 |   }
734 |   // NOLINTNEXTLINE(bugprone-branch-clone)
735 |   auto size = tuple ? PyTuple_GET_SIZE(arg) : PyList_GET_SIZE(arg);
736 |   std::vector<double> res(size);
737 |   for (const auto idx : c10::irange(size)) {
738 |     PyObject* obj =
739 |         tuple ? PyTuple_GET_ITEM(arg, idx) : PyList_GET_ITEM(arg, idx);
740 |     try {
741 |       if (torch::is_symfloat(py::handle(obj))) {
742 |         res[idx] = py::cast<c10::SymFloat>(py::handle(obj))
743 |                        .guard_float(__FILE__, __LINE__);
744 |       } else {
745 |         res[idx] = THPUtils_unpackDouble(obj);
746 |       }
747 |     } catch (const std::exception&) {
748 |       TORCH_CHECK_TYPE(
749 |           false,
750 |           fmt::format(
751 |               "{}(): argument '{}' must be {}, but found element of type {} at pos {}",
752 |               signature.name,
753 |               signature.params[i].name,
754 |               signature.params[i].type_name(),
755 |               Py_TYPE(obj)->tp_name,
756 |               idx + 1));
757 |     }
758 |   }
759 |   return res;
760 | }
761 | 
762 | inline c10::OptionalArray<double> PythonArgs::doublelistOptional(int i) {
763 |   if (!args[i]) {
764 |     return {};
765 |   }
766 |   return this->getDoublelist(i);
767 | }
768 | 
```
- EN: Implements routines such as `intlist`, `symintlist`, `res` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `intlist`、`symintlist`、`res` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 769-843
```cpp
769 | inline std::vector<double> PythonArgs::doublelist(int i) {
770 |   if (!args[i]) {
771 |     return {};
772 |   }
773 |   return this->getDoublelist(i);
774 | }
775 | 
776 | inline std::optional<c10::DispatchKeySet> PythonArgs::toDispatchKeySetOptional(
777 |     int i) {
778 |   if (!args[i]) {
779 |     return {};
780 |   }
781 |   return py::cast<c10::DispatchKeySet>(py::handle(args[i]));
782 | }
783 | 
784 | inline at::ScalarType PythonArgs::scalartypeWithDefault(
785 |     int i,
786 |     at::ScalarType default_scalartype) {
787 |   if (!args[i])
788 |     return default_scalartype;
789 |   return scalartype(i);
790 | }
791 | 
792 | inline at::ScalarType toScalarType(PyObject* obj) {
793 |   if (obj == (PyObject*)&PyFloat_Type) {
794 |     return at::ScalarType::Double;
795 |   }
796 |   if (obj == (PyObject*)&PyBool_Type) {
797 |     return at::ScalarType::Bool;
798 |   }
799 |   if (obj == (PyObject*)&PyLong_Type) {
800 |     return at::ScalarType::Long;
801 |   }
802 |   if (obj == (PyObject*)&PyComplex_Type) {
803 |     return at::ScalarType::ComplexDouble;
804 |   }
805 |   return reinterpret_cast<THPDtype*>(obj)->scalar_type;
806 | }
807 | 
808 | inline at::ScalarType PythonArgs::scalartype(int i) {
809 |   if (!args[i]) {
810 |     auto scalartype = signature.params[i].default_scalartype;
811 |     return (scalartype == at::ScalarType::Undefined)
812 |         ? torch::tensors::get_default_scalar_type()
813 |         : scalartype;
814 |   }
815 |   PyObject* obj = args[i];
816 |   return toScalarType(obj);
817 | }
818 | 
819 | inline std::optional<at::ScalarType> PythonArgs::scalartypeOptional(int i) {
820 |   if (!args[i])
821 |     return std::nullopt;
822 |   return scalartype(i);
823 | }
824 | 
825 | inline at::Layout toLayout(PyObject* obj) {
826 |   const auto layout = reinterpret_cast<THPLayout*>(obj);
827 |   return layout->layout;
828 | }
829 | 
830 | inline at::Layout PythonArgs::layout(int i) {
831 |   if (!args[i])
832 |     return signature.params[i].default_layout;
833 |   return toLayout(args[i]);
834 | }
835 | 
836 | inline at::Layout PythonArgs::layoutWithDefault(
837 |     int i,
838 |     at::Layout default_layout) {
839 |   if (!args[i])
840 |     return default_layout;
841 |   return layout(i);
842 | }
843 | 
```
- EN: Implements routines such as `scalartype`, `toScalarType`, `toLayout`, `layout` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `scalartype`、`toScalarType`、`toLayout`、`layout` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 844-921
```cpp
844 | inline std::optional<at::Layout> PythonArgs::layoutOptional(int i) {
845 |   if (!args[i])
846 |     return std::nullopt;
847 |   return layout(i);
848 | }
849 | 
850 | inline at::Device deviceFromLong(int64_t device_index) {
851 |   TORCH_CHECK(device_index >= 0, "Device index must not be negative");
852 |   return at::Device(
853 |       // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
854 |       at::getAccelerator(true).value(),
855 |       static_cast<c10::DeviceIndex>(device_index));
856 | }
857 | 
858 | inline at::Device toDevice(PyObject* obj) {
859 |   if (THPDevice_Check(obj)) {
860 |     const auto device = reinterpret_cast<THPDevice*>(obj);
861 |     return device->device;
862 |   }
863 |   if (THPUtils_checkLong(obj)) {
864 |     return deviceFromLong(THPUtils_unpackLong(obj));
865 |   }
866 |   if (torch::is_symint(py::handle(obj))) {
867 |     auto device_index =
868 |         py::cast<c10::SymInt>(py::handle(obj)).guard_int(__FILE__, __LINE__);
869 |     return deviceFromLong(device_index);
870 |   }
871 |   if (torch::is_dynint(py::handle(obj))) {
872 |     auto device_index = py::cast<int>(py::handle(obj));
873 |     return deviceFromLong(device_index);
874 |   }
875 |   const std::string& device_str = THPUtils_unpackString(obj);
876 |   return at::Device(device_str);
877 | }
878 | 
879 | inline at::Device PythonArgs::device(int i) {
880 |   if (!args[i]) {
881 |     return torch::tensors::get_default_device();
882 |   }
883 |   return toDevice(args[i]);
884 | }
885 | 
886 | inline at::Device PythonArgs::deviceWithDefault(
887 |     int i,
888 |     const at::Device& default_device) {
889 |   if (!args[i])
890 |     return default_device;
891 |   return device(i);
892 | }
893 | 
894 | inline std::optional<at::Device> PythonArgs::deviceOptional(int i) {
895 |   if (!args[i])
896 |     return std::nullopt;
897 |   return device(i);
898 | }
899 | 
900 | inline at::Dimname PythonArgs::dimname(int i) {
901 |   TORCH_INTERNAL_ASSERT(args[i] != nullptr);
902 |   return THPDimname_parse(args[i]);
903 | }
904 | 
905 | inline std::vector<at::Dimname> parseDimnameList(PyObject* arg) {
906 |   auto tuple = PyTuple_Check(arg);
907 |   if (!tuple) {
908 |     TORCH_INTERNAL_ASSERT(PyList_Check(arg), "expected tuple or list");
909 |   }
910 |   // NOLINTNEXTLINE(bugprone-branch-clone)
911 |   auto size = tuple ? PyTuple_GET_SIZE(arg) : PyList_GET_SIZE(arg);
912 |   std::vector<at::Dimname> res;
913 |   res.reserve(size);
914 |   for (const auto idx : c10::irange(size)) {
915 |     PyObject* obj =
916 |         tuple ? PyTuple_GET_ITEM(arg, idx) : PyList_GET_ITEM(arg, idx);
917 |     res.push_back(THPDimname_parse(obj));
918 |   }
919 |   return res;
920 | }
921 | 
```
- EN: Implements routines such as `layout`, `deviceFromLong`, `toDevice`, `device`, `THPDimname_parse` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `layout`、`deviceFromLong`、`toDevice`、`device`、`THPDimname_parse` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 922-1001
```cpp
 922 | inline std::optional<std::vector<at::Dimname>> PythonArgs::
 923 |     toDimnameListOptional(int i) {
 924 |   if (!args[i])
 925 |     return std::nullopt;
 926 |   return parseDimnameList(args[i]);
 927 | }
 928 | 
 929 | inline std::vector<at::Dimname> PythonArgs::dimnamelist(int i) {
 930 |   TORCH_INTERNAL_ASSERT(args[i]);
 931 |   PyObject* arg = args[i];
 932 |   auto size = signature.params[i].size;
 933 |   TORCH_INTERNAL_ASSERT(size == 0 || size == 1);
 934 |   if (size == 1 && THPUtils_checkDimname(arg)) {
 935 |     return {THPDimname_parse(arg)};
 936 |   }
 937 |   return parseDimnameList(arg);
 938 | }
 939 | 
 940 | inline at::MemoryFormat PythonArgs::memoryformat(int i) {
 941 |   if (!args[i])
 942 |     return at::MemoryFormat::Contiguous;
 943 |   TORCH_CHECK(
 944 |       THPMemoryFormat_Check(args[i]),
 945 |       "memory_format arg must be an instance of the torch.memory_format");
 946 |   const auto memory_format = reinterpret_cast<THPMemoryFormat*>(args[i]);
 947 |   return memory_format->memory_format;
 948 | }
 949 | 
 950 | inline std::optional<at::MemoryFormat> PythonArgs::memoryformatOptional(int i) {
 951 |   if (!args[i])
 952 |     return std::nullopt;
 953 |   return memoryformat(i);
 954 | }
 955 | 
 956 | inline at::QScheme PythonArgs::toQScheme(int i) {
 957 |   if (!args[i])
 958 |     return at::kPerTensorAffine;
 959 |   TORCH_CHECK(
 960 |       THPQScheme_Check(args[i]),
 961 |       "qscheme arg must be an instance of the torch.qscheme");
 962 |   const auto qscheme = reinterpret_cast<THPQScheme*>(args[i]);
 963 |   return qscheme->qscheme;
 964 | }
 965 | 
 966 | inline std::string PythonArgs::string(int i) {
 967 |   return stringWithDefault(i, signature.params[i].default_string);
 968 | }
 969 | 
 970 | inline std::string PythonArgs::stringWithDefault(
 971 |     int i,
 972 |     const std::string& default_str) {
 973 |   if (!args[i])
 974 |     return default_str;
 975 |   return THPUtils_unpackString(args[i]);
 976 | }
 977 | 
 978 | inline std::optional<std::string> PythonArgs::stringOptional(int i) {
 979 |   if (!args[i])
 980 |     return std::nullopt;
 981 |   return THPUtils_unpackString(args[i]);
 982 | }
 983 | 
 984 | inline std::string_view PythonArgs::stringView(int i) {
 985 |   return stringViewWithDefault(i, signature.params[i].default_string);
 986 | }
 987 | 
 988 | inline std::string_view PythonArgs::stringViewWithDefault(
 989 |     int i,
 990 |     const std::string_view default_str) {
 991 |   if (!args[i])
 992 |     return default_str;
 993 |   return THPUtils_unpackStringView(args[i]);
 994 | }
 995 | 
 996 | inline std::optional<std::string_view> PythonArgs::stringViewOptional(int i) {
 997 |   if (!args[i])
 998 |     return std::nullopt;
 999 |   return THPUtils_unpackStringView(args[i]);
1000 | }
1001 | 
```
- EN: Implements routines such as `toDimnameListOptional`, `parseDimnameList`, `memoryformat`, `stringWithDefault`, `THPUtils_unpackString` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `toDimnameListOptional`、`parseDimnameList`、`memoryformat`、`stringWithDefault`、`THPUtils_unpackString` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1002-1078
```cpp
1002 | inline int64_t PythonArgs::toInt64(int i) {
1003 |   if (!args[i])
1004 |     return signature.params[i].default_int;
1005 |   if (traceable && jit::tracer::isTracing() && THPVariable_Check(args[i])) {
1006 |     auto& var = THPVariable_Unpack(args[i]);
1007 |     jit::tracer::ArgumentStash::stashValue(
1008 |         signature.params[i].name, idx, var, c10::IntType::get());
1009 |   }
1010 |   if (torch::is_symint(py::handle(args[i]))) {
1011 |     return py::cast<c10::SymInt>(py::handle(args[i]))
1012 |         .guard_int(__FILE__, __LINE__);
1013 |   }
1014 |   if (torch::is_dynint(py::handle(args[i]))) {
1015 |     return py::cast<int>(py::handle(args[i]));
1016 |   }
1017 |   return THPUtils_unpackLong(args[i]);
1018 | }
1019 | 
1020 | inline c10::SymInt PythonArgs::toSymInt(int i) {
1021 |   if (!args[i]) {
1022 |     return c10::SymInt(signature.params[i].default_int);
1023 |   }
1024 | 
1025 |   if (traceable && jit::tracer::isTracing() && THPVariable_Check(args[i])) {
1026 |     auto& var = THPVariable_Unpack(args[i]);
1027 |     jit::tracer::ArgumentStash::stashValue(
1028 |         signature.params[i].name, idx, var, c10::IntType::get());
1029 |   }
1030 | 
1031 |   return py::cast<c10::SymInt>(py::handle(args[i]));
1032 | }
1033 | 
1034 | inline c10::SymBool PythonArgs::toSymBool(int i) {
1035 |   if (!args[i]) {
1036 |     return c10::SymBool(signature.params[i].default_bool);
1037 |   }
1038 |   if (traceable && jit::tracer::isTracing() && THPVariable_Check(args[i])) {
1039 |     auto& var = THPVariable_Unpack(args[i]);
1040 |     jit::tracer::ArgumentStash::stashValue(
1041 |         signature.params[i].name, idx, var, c10::BoolType::get());
1042 |   }
1043 | 
1044 |   return py::cast<c10::SymBool>(py::handle(args[i]));
1045 | }
1046 | 
1047 | inline int64_t PythonArgs::toInt64WithDefault(int i, int64_t default_int) {
1048 |   if (!args[i])
1049 |     return default_int;
1050 |   return toInt64(i);
1051 | }
1052 | 
1053 | inline std::optional<int64_t> PythonArgs::toInt64Optional(int i) {
1054 |   if (!args[i])
1055 |     return std::nullopt;
1056 |   return toInt64(i);
1057 | }
1058 | 
1059 | inline std::optional<c10::SymInt> PythonArgs::toSymIntOptional(int i) {
1060 |   if (!args[i])
1061 |     return std::nullopt;
1062 |   return toSymInt(i);
1063 | }
1064 | 
1065 | inline std::optional<bool> PythonArgs::toBoolOptional(int i) {
1066 |   if (!args[i]) {
1067 |     return std::nullopt;
1068 |   }
1069 |   return toBool(i);
1070 | }
1071 | 
1072 | inline std::optional<double> PythonArgs::toDoubleOptional(int i) {
1073 |   if (!args[i]) {
1074 |     return std::nullopt;
1075 |   }
1076 |   return toDouble(i);
1077 | }
1078 | 
```
- EN: Implements routines such as `THPUtils_unpackLong`, `toInt64`, `toSymInt`, `toBool`, `toDouble` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `THPUtils_unpackLong`、`toInt64`、`toSymInt`、`toBool`、`toDouble` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 1079-1155
```cpp
1079 | inline double PythonArgs::toDouble(int i) {
1080 |   if (!args[i])
1081 |     return signature.params[i].default_double;
1082 |   if (torch::is_symfloat(py::handle(args[i]))) {
1083 |     return py::cast<c10::SymFloat>(py::handle(args[i]))
1084 |         .guard_float(__FILE__, __LINE__);
1085 |   }
1086 |   if (torch::is_symint(py::handle(args[i]))) {
1087 |     return static_cast<double>(py::cast<c10::SymInt>(py::handle(args[i]))
1088 |                                    .guard_int(__FILE__, __LINE__));
1089 |   }
1090 |   if (torch::is_dynint(py::handle(args[i]))) {
1091 |     return static_cast<double>(py::cast<int>(py::handle(args[i])));
1092 |   }
1093 |   return THPUtils_unpackDouble(args[i]);
1094 | }
1095 | 
1096 | inline bool PythonArgs::toBool(int i) {
1097 |   if (!args[i]) {
1098 |     return signature.params[i].default_bool;
1099 |   }
1100 |   if (Py_IsTrue(args[i])) {
1101 |     return true;
1102 |   }
1103 |   if (Py_IsFalse(args[i])) {
1104 |     return false;
1105 |   }
1106 |   if (torch::is_symbool(py::handle(args[i]))) {
1107 |     return py::cast<c10::SymBool>(py::handle(args[i]))
1108 |         .guard_bool(__FILE__, __LINE__);
1109 |   }
1110 |   return false;
1111 | }
1112 | 
1113 | inline double PythonArgs::toDoubleWithDefault(int i, double default_double) {
1114 |   if (!args[i])
1115 |     return default_double;
1116 |   return toDouble(i);
1117 | }
1118 | 
1119 | inline c10::complex<double> PythonArgs::toComplex(int i) {
1120 |   if (!args[i])
1121 |     return *(reinterpret_cast<const c10::complex<double>*>(
1122 |         signature.params[i].default_complex));
1123 |   return THPUtils_unpackComplexDouble(args[i]);
1124 | }
1125 | 
1126 | inline c10::complex<double> PythonArgs::toComplexWithDefault(
1127 |     int i,
1128 |     c10::complex<double> default_complex) {
1129 |   if (!args[i])
1130 |     return default_complex;
1131 |   return toComplex(i);
1132 | }
1133 | 
1134 | inline bool PythonArgs::toBoolWithDefault(int i, bool default_bool) {
1135 |   if (!args[i])
1136 |     return default_bool;
1137 |   return toBool(i);
1138 | }
1139 | 
1140 | inline bool PythonArgs::isNone(int i) {
1141 |   return args[i] == nullptr;
1142 | }
1143 | 
1144 | inline std::optional<at::Generator> PythonArgs::generator(int i) {
1145 |   if (!args[i])
1146 |     return std::nullopt;
1147 |   return reinterpret_cast<THPGenerator*>(args[i])->cdata;
1148 | }
1149 | 
1150 | inline at::Storage PythonArgs::storage(int i) {
1151 |   if (!args[i])
1152 |     return at::Storage();
1153 |   return createStorage(args[i]);
1154 | }
1155 | 
```
- EN: Implements routines such as `THPUtils_unpackDouble`, `toDouble`, `THPUtils_unpackComplexDouble`, `toComplex`, `toBool` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `THPUtils_unpackDouble`、`toDouble`、`THPUtils_unpackComplexDouble`、`toComplex`、`toBool` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 1156-1227
```cpp
1156 | inline at::Storage PythonArgs::storage(
1157 |     int i,
1158 |     at::ScalarType& storage_scalar_type,
1159 |     bool& is_typed_storage) {
1160 |   at::Storage storage;
1161 |   if (!args[i]) {
1162 |     storage = at::Storage();
1163 |     is_typed_storage = false;
1164 |     storage_scalar_type = at::ScalarType::Undefined;
1165 |   } else {
1166 |     std::tie(storage, storage_scalar_type, is_typed_storage) =
1167 |         createStorageGetType(args[i]);
1168 |   }
1169 |   return storage;
1170 | }
1171 | 
1172 | inline c10::Stream PythonArgs::stream(int i) {
1173 |   if (!args[i])
1174 |     return c10::Stream(
1175 |         c10::Stream::Default::DEFAULT, c10::Device(c10::DeviceType::CPU, -1));
1176 |   if (!THPStream_Check(args[i])) {
1177 |     TORCH_CHECK_TYPE(
1178 |         false,
1179 |         fmt::format(
1180 |             "expected Stream object. Got '{}'", Py_TYPE(args[i])->tp_name));
1181 |   }
1182 |   return c10::Stream::unpack3(
1183 |       ((THPStream*)args[i])->stream_id,
1184 |       static_cast<c10::DeviceIndex>(((THPStream*)args[i])->device_index),
1185 |       static_cast<c10::DeviceType>(((THPStream*)args[i])->device_type));
1186 | }
1187 | 
1188 | inline PyObject* PythonArgs::pyobject(int i) {
1189 |   if (!args[i])
1190 |     return Py_None;
1191 |   return args[i];
1192 | }
1193 | 
1194 | /*
1195 |  *
1196 |  * Handle __torch_function__ overrides if we know that there are overloaded
1197 |  * arguments.  All objects stored in r.overloaded_args must have a
1198 |  * __torch_function__ implementation and the arguments must be ordered in order
1199 |  * of precedence. Precedence goes from left to right in the order of the
1200 |  * signature of the function the overloaded arguments were passed to, except
1201 |  * subclasses are always considered before superclasses.
1202 |  *
1203 |  * If the result of calling __torch_function__ is NotImplemented, the
1204 |  * next implementation in the precedence order is called. If all
1205 |  * arguments return NotImplemented from their __torch_function__
1206 |  * implementation, a TypeError is raised in Python.
1207 |  *
1208 |  * Assumes overloaded_args has at least one entry. All entries must have
1209 |  * a __torch_function__ attribute that resolves to a callable that
1210 |  * accepts a torch API function, a tuple of arguments, and a dict of
1211 |  * keyword arguments for the torch API function.
1212 |  *
1213 |  * It is sufficient to call PythonArgs::has_torch_function before
1214 |  * calling this function to verify that there are valid arguments
1215 |  * present. If that is not done then special care must be taken to
1216 |  * ensure there are arguments that are overloaded with
1217 |  * __torch_function__.
1218 |  *
1219 |  * See torch._overrides.handle_torch_function for the equivalent
1220 |  * code in the pure-python implementation.
1221 |  *
1222 |  * 'r' is a parsed PythonArgs instance, returned from
1223 |  * PythonArgParser::parse.
1224 |  *
1225 |  * 'args' is a reference to the python tuple of arguments to the torch
1226 |  * API function.
1227 |  *
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1228-1304
```cpp
1228 |  * 'kwargs' is a reference to the python dict of keyword arguments to
1229 |  * the torch API function.
1230 |  *
1231 |  * 'torch_api' is a reference to a python torch API namespace.
1232 |  *
1233 |  * 'torch_api_function' is the reference to the original torch method, usually,
1234 |  * we can use torch_api and func_name to get torch_api_function. In some cases,
1235 |  * e.g., torch custom op, we create the function in C++, if we still use
1236 |  * torch_api and func_name to fetch original api, a cyclic call will happen.
1237 |  *
1238 |  * 'overloaded_args' is the args which have overloaded __torch_function__.
1239 |  *
1240 |  * 'func_name' is the named of the original torch method.
1241 |  *
1242 |  * TODO: we could use different names for the following 'handle_torch_function'
1243 |  * instead of overloading.
1244 |  *
1245 |  */
1246 | // Used for Tensor methods with arguments.
1247 | auto handle_torch_function(
1248 |     PythonArgs& r,
1249 |     PyObject* self,
1250 |     PyObject* args,
1251 |     PyObject* kwargs,
1252 |     PyObject* torch_api,
1253 |     const char* module_name,
1254 |     const char* func_name_override = nullptr) -> PyObject*;
1255 | 
1256 | // Used for functions which needs to parse python args.
1257 | auto handle_torch_function(
1258 |     PythonArgs& r,
1259 |     PyObject* args,
1260 |     PyObject* kwargs,
1261 |     PyObject* torch_api,
1262 |     const char* module_name,
1263 |     const char* func_name_override = nullptr) -> PyObject*;
1264 | 
1265 | // Used for functions that have no argument parsing.
1266 | auto handle_torch_function(
1267 |     PyObject* self,
1268 |     const std::string& func_name,
1269 |     PyObject* args = nullptr,
1270 |     PyObject* kwargs = nullptr,
1271 |     PyObject* torch_api = THPVariableClass,
1272 |     const std::string& module_name = "torch.Tensor") -> PyObject*;
1273 | 
1274 | // Used for functions created in C++, e.g., C++ custom op, which doesn't use
1275 | // PythonArgParser to get overloaded_args.
1276 | enum class TorchFunctionName { TorchFunction, TorchDispatch };
1277 | 
1278 | auto TORCH_PYTHON_API handle_torch_function_no_python_arg_parser(
1279 |     at::ArrayRef<PyObject*> overloaded_args,
1280 |     PyObject* args,
1281 |     PyObject* kwargs,
1282 |     const char* func_name,
1283 |     PyObject* torch_api_function,
1284 |     const char* module_name,
1285 |     TorchFunctionName torch_function_name = TorchFunctionName::TorchFunction)
1286 |     -> PyObject*;
1287 | 
1288 | auto handle_torch_function_no_python_arg_parser(
1289 |     at::ArrayRef<PyObject*> overloaded_args,
1290 |     PyObject* args,
1291 |     PyObject* kwargs,
1292 |     const char* func_name,
1293 |     PyObject* torch_api_function,
1294 |     const char* module_name,
1295 |     const c10::OperatorHandle* opt_op,
1296 |     torch::jit::Stack* opt_stack,
1297 |     TorchFunctionName torch_function_name = TorchFunctionName::TorchFunction)
1298 |     -> PyObject*;
1299 | 
1300 | // Used for getters of Tensor properties
1301 | auto handle_torch_function_getter(
1302 |     THPVariable* self,
1303 |     const std::string& property_name) -> PyObject*;
1304 | 
```
- EN: Defines or extends data abstractions such as `TorchFunctionName` that structure the state handled by this file.
- CN: 定义或扩展了 `TorchFunctionName` 等数据抽象，用来组织本文件处理的状态。

### Lines 1305-1372
```cpp
1305 | // Used for setters of Tensor properties.
1306 | auto handle_torch_function_setter(
1307 |     THPVariable* self,
1308 |     const std::string& property_name,
1309 |     PyObject* value) -> int;
1310 | 
1311 | // Used for __getitem__ and __setitem__
1312 | auto handle_torch_function_indexing(
1313 |     PyObject* self,
1314 |     PyObject* index,
1315 |     PyObject* val = nullptr) -> PyObject*;
1316 | 
1317 | /*
1318 |  * Check if the input obj is Tensor type, including its subclass, or overloaded
1319 |  * type. If the type defines __torch_function__, it also returns true.
1320 |  * Otherwise returns false. If the class is not torch.Tensor, and it defines
1321 |  * __torch_function__, we append obj to overloaded_args.
1322 |  *
1323 |  * 'obj': the input argument to be checked
1324 |  * 'overloaded_args': the vector to append the overloaded args.
1325 |  */
1326 | bool is_tensor_and_append_overloaded(
1327 |     PyObject* obj,
1328 |     std::vector<PyObject*>* overloaded_args);
1329 | 
1330 | /*
1331 |  * Check if the input obj is Tensor List or Tensor Tuple type. First check
1332 |  * whether obj is Tuple or List type, if true, iterate over each element and
1333 |  * check whether it is Tensor type, including its subclass or overloaded type.
1334 |  * At the same time, the overloaded arg is appended to the overloaded_args.
1335 |  *
1336 |  * 'obj': the input argument to be checked
1337 |  * 'overloaded_args': the vector to append the overloaded args.
1338 |  * 'argnum': the number of total arguments of the function being checked.
1339 |  * 'throw_error': whether throw error if any element in the list or tuple is
1340 |  *                not tensor type or overloaded.
1341 |  */
1342 | bool is_tensor_list_and_append_overloaded(
1343 |     PyObject* obj,
1344 |     std::vector<PyObject*>* overloaded_args,
1345 |     size_t argnum,
1346 |     bool throw_error);
1347 | 
1348 | /* Given an argument that is definitely a tensor and is definitely overloaded,
1349 |  * append it to the overloaded arguments list.  Use this instead of
1350 |  * is_tensor_and_append_overloaded in situations where you have a PyObject
1351 |  * and you know it definitely is a Tensor and it is definitely overloaded.
1352 |  *
1353 |  * 'overloaded_args': the vector to append the overloaded args
1354 |  * 'obj': the input tensor that is overloaded
1355 |  */
1356 | void append_overloaded_tensor(
1357 |     std::vector<PyObject*>* overloaded_args,
1358 |     PyObject* obj);
1359 | 
1360 | /* Given an argument that is definitely a type and is definitely overloaded,
1361 |  * append it to the overloaded arguments list. Use this only with
1362 |  * __torch_dispatch__, where we operate on classes that have a
1363 |  * __torch_dispatch__ classmethod.
1364 |  *
1365 |  * 'overloaded_args': the vector to append the overloaded type
1366 |  * 'obj': the input class that has a __torch_dispatch__ classmethod.
1367 |  */
1368 | void append_overloaded_type(
1369 |     std::vector<PyObject*>* overloaded_args,
1370 |     PyObject* obj);
1371 | 
1372 | } // namespace torch
```
- EN: Defines or extends data abstractions such as `is`, `that` that structure the state handled by this file. Declares routines such as `is_tensor_and_append_overloaded`, `is_tensor_list_and_append_overloaded`, `append_overloaded_tensor`, `append_overloaded_type` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 定义或扩展了 `is`、`that` 等数据抽象，用来组织本文件处理的状态。 声明了 `is_tensor_and_append_overloaded`、`is_tensor_list_and_append_overloaded`、`append_overloaded_tensor`、`append_overloaded_type` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `ParameterType`, `TorchFunctionName`, `is`, `that`, `PythonArgs`, `ParsedArgs`.
  - CN: `ParameterType`、`TorchFunctionName`、`is`、`that`、`PythonArgs`、`ParsedArgs`。
- **Important routines / 重要例程**
  - EN: `THPUtils_checkScalar`, `PyFloat_Check`, `should_allow_numbers_as_tensors`, `check`, `_check`, `set_default_str`, `type_name`, `FunctionSignature`.
  - CN: `THPUtils_checkScalar`、`PyFloat_Check`、`should_allow_numbers_as_tensors`、`check`、`_check`、`set_default_str`、`type_name`、`FunctionSignature`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/python_headers.h>`, `<torch/csrc/Device.h>`, `<torch/csrc/Dtype.h>`, `<torch/csrc/DynamicTypes.h>`, `<torch/csrc/Exceptions.h>`, `<torch/csrc/Export.h>`, `<torch/csrc/Generator.h>`, `<torch/csrc/Layout.h>`, `<torch/csrc/MemoryFormat.h>`, `<torch/csrc/QScheme.h>`, `<torch/csrc/Stream.h>`, `<torch/csrc/autograd/python_variable.h>`
- External includes / 外部头文件: `<fmt/format.h>`, `<pybind11/pytypes.h>`, `<array>`, `<cstddef>`, `<string>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
