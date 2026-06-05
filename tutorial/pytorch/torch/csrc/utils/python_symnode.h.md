# python_symnode.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/python_symnode.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Bridges C++ components to Python-facing APIs or bindings.
  - CN: 把 C++ 组件桥接到面向 Python 的 API 或绑定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-55
```cpp
 1 | #pragma once
 2 | 
 3 | #include <c10/core/SafePyObject.h>
 4 | #include <c10/core/SymNodeImpl.h>
 5 | 
 6 | #include <torch/csrc/PyInterpreter.h>
 7 | #include <torch/csrc/autograd/python_variable.h>
 8 | #include <torch/csrc/utils/pybind.h>
 9 | 
10 | namespace torch {
11 | 
12 | TORCH_PYTHON_API py::handle get_symint_class();
13 | TORCH_PYTHON_API py::handle get_symfloat_class();
14 | TORCH_PYTHON_API py::handle get_symbool_class();
15 | TORCH_PYTHON_API py::handle get_dynint_class();
16 | 
17 | // NB: These functions must not be called too early, otherwise torch not setup.
18 | // Alternate design is to have torch "register" the object to us
19 | inline bool is_symint(py::handle obj) {
20 |   return py::isinstance(obj, get_symint_class());
21 | }
22 | inline bool is_symfloat(py::handle obj) {
23 |   return py::isinstance(obj, get_symfloat_class());
24 | }
25 | inline bool is_symbool(py::handle obj) {
26 |   return py::isinstance(obj, get_symbool_class());
27 | }
28 | inline bool is_dynint(py::handle obj) {
29 |   return py::isinstance(obj, get_dynint_class());
30 | }
31 | 
32 | namespace impl {
33 | 
34 | // This c10::SymNodeImpl simply backends to a Python object that
35 | // implements the API.   The Python object is the source of truth,
36 | // this is just an adapter so C++ calls can get to the object.
37 | class PythonSymNodeImpl : public c10::SymNodeImpl {
38 |  public:
39 |   PythonSymNodeImpl(py::object pyobj) : c10::SymNodeImpl() {
40 |     pyobj_ = std::make_shared<c10::SafePyObject>(
41 |         pyobj.release().ptr(), getPyInterpreter());
42 |   }
43 | 
44 |   c10::SymNode wrap_int(int64_t num) override {
45 |     py::gil_scoped_acquire acquire;
46 |     auto r = getPyObj().attr("wrap_int")(num);
47 |     return c10::make_intrusive<PythonSymNodeImpl>(std::move(r));
48 |   }
49 | 
50 |   c10::SymNode wrap_float(double num) override {
51 |     py::gil_scoped_acquire acquire;
52 |     auto r = getPyObj().attr("wrap_float")(num);
53 |     return c10::make_intrusive<PythonSymNodeImpl>(std::move(r));
54 |   }
55 | 
```
- EN: Brings in project headers such as `<c10/core/SafePyObject.h>`, `<c10/core/SymNodeImpl.h>`, `<torch/csrc/PyInterpreter.h>`, `<torch/csrc/autograd/python_variable.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch`, `impl`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `PythonSymNodeImpl` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/core/SafePyObject.h>`、`<c10/core/SymNodeImpl.h>`、`<torch/csrc/PyInterpreter.h>`、`<torch/csrc/autograd/python_variable.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch`、`impl`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `PythonSymNodeImpl` 等数据抽象，用来组织本文件处理的状态。

### Lines 56-111
```cpp
 56 |   c10::SymNode wrap_bool(bool num) override {
 57 |     py::gil_scoped_acquire acquire;
 58 |     auto r = getPyObj().attr("wrap_bool")(num);
 59 |     return c10::make_intrusive<PythonSymNodeImpl>(std::move(r));
 60 |   }
 61 | 
 62 | #define TORCH_SYMNODE_SIZES_STRIDES(n)                                        \
 63 |   c10::SymNode n(                                                             \
 64 |       c10::ArrayRef<c10::SymNode> sizes, c10::ArrayRef<c10::SymNode> strides) \
 65 |       override {                                                              \
 66 |     py::gil_scoped_acquire acquire;                                           \
 67 |     auto r = getPyObj().attr(#n)(sizes, strides);                             \
 68 |     return c10::make_intrusive<PythonSymNodeImpl>(std::move(r));              \
 69 |   }
 70 | 
 71 |   // clang-format off
 72 |     TORCH_SYMNODE_SIZES_STRIDES(is_contiguous)
 73 |     TORCH_SYMNODE_SIZES_STRIDES(is_channels_last_contiguous_2d)
 74 |     TORCH_SYMNODE_SIZES_STRIDES(is_channels_last_contiguous_3d)
 75 |     TORCH_SYMNODE_SIZES_STRIDES(is_channels_last_strides_2d)
 76 |     TORCH_SYMNODE_SIZES_STRIDES(is_channels_last_strides_3d)
 77 |     TORCH_SYMNODE_SIZES_STRIDES(is_non_overlapping_and_dense)
 78 |   // clang-format on
 79 | 
 80 | #undef TORCH_SYMNODE_SIZES_STRIDES
 81 | 
 82 |   bool bool_() override {
 83 |     py::gil_scoped_acquire acquire;
 84 |     return getPyObj().attr("bool_")().is(py::handle(Py_True));
 85 |   }
 86 | 
 87 |   bool is_int() override {
 88 |     py::gil_scoped_acquire acquire;
 89 |     return getPyObj().attr("is_int")().is(py::handle(Py_True));
 90 |   }
 91 | 
 92 |   bool is_float() override {
 93 |     py::gil_scoped_acquire acquire;
 94 |     return getPyObj().attr("is_float")().is(py::handle(Py_True));
 95 |   }
 96 | 
 97 |   bool is_bool() override {
 98 |     py::gil_scoped_acquire acquire;
 99 |     return getPyObj().attr("is_bool")().is(py::handle(Py_True));
100 |   }
101 | 
102 |   bool is_nested_int() const override {
103 |     py::gil_scoped_acquire acquire;
104 |     return getPyObj().attr("is_nested_int")().is(py::handle(Py_True));
105 |   }
106 | 
107 |   bool has_hint() override {
108 |     py::gil_scoped_acquire acquire;
109 |     return getPyObj().attr("has_hint")().is(py::handle(Py_True));
110 |   }
111 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `wrap_bool`, `bool_`, `getPyObj`, `is_int`, `is_float` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `wrap_bool`、`bool_`、`getPyObj`、`is_int`、`is_float` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 112-166
```cpp
112 |   int64_t guard_int(const char* file, int64_t line) override {
113 |     py::gil_scoped_acquire acquire;
114 |     return getPyObj().attr("guard_int")(file, line).cast<int64_t>();
115 |   }
116 | 
117 |   double guard_float(const char* file, int64_t line) override {
118 |     py::gil_scoped_acquire acquire;
119 |     return getPyObj().attr("guard_float")(file, line).cast<double>();
120 |   }
121 | 
122 |   bool guard_bool(const char* file, int64_t line) override {
123 |     py::gil_scoped_acquire acquire;
124 |     return getPyObj().attr("guard_bool")(file, line).cast<bool>();
125 |   }
126 | 
127 |   bool expect_true(const char* file, int64_t line) override {
128 |     py::gil_scoped_acquire acquire;
129 |     return getPyObj().attr("expect_true")(file, line).cast<bool>();
130 |   }
131 | 
132 |   bool guard_size_oblivious(const char* file, int64_t line) override {
133 |     py::gil_scoped_acquire acquire;
134 |     return getPyObj().attr("guard_size_oblivious")(file, line).cast<bool>();
135 |   }
136 | 
137 |   bool guard_or_false(const char* file, int64_t line) override {
138 |     py::gil_scoped_acquire acquire;
139 |     return getPyObj().attr("guard_or_false")(file, line).cast<bool>();
140 |   }
141 | 
142 |   bool statically_known_true(const char* file, int64_t line) override {
143 |     py::gil_scoped_acquire acquire;
144 |     return getPyObj().attr("statically_known_true")(file, line).cast<bool>();
145 |   }
146 | 
147 |   bool guard_or_true(const char* file, int64_t line) override {
148 |     py::gil_scoped_acquire acquire;
149 |     return getPyObj().attr("guard_or_true")(file, line).cast<bool>();
150 |   }
151 | 
152 |   int64_t int_() override {
153 |     py::gil_scoped_acquire acquire;
154 |     return getPyObj().attr("int_")().cast<int64_t>();
155 |   }
156 | 
157 |   std::optional<int64_t> maybe_as_int() override {
158 |     py::gil_scoped_acquire acquire;
159 |     const auto& r = getPyObj().attr("maybe_as_int")();
160 |     if (r.is_none()) {
161 |       return std::nullopt;
162 |     } else {
163 |       return r.cast<int64_t>();
164 |     }
165 |   }
166 | 
```
- EN: Implements routines such as `guard_int`, `getPyObj`, `guard_float`, `guard_bool`, `expect_true` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `guard_int`、`getPyObj`、`guard_float`、`guard_bool`、`expect_true` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 167-219
```cpp
167 |   std::string str() override {
168 |     py::gil_scoped_acquire acquire;
169 |     return getPyObj().attr("str")().cast<std::string>();
170 |   }
171 | 
172 |   std::string _graph_repr() override {
173 |     py::gil_scoped_acquire acquire;
174 |     return getPyObj().attr("_graph_repr")().cast<std::string>();
175 |   }
176 | 
177 |   c10::SymNode dispatch_sym_ite_(
178 |       const char* fname,
179 |       const c10::SymNode& other,
180 |       const c10::SymNode& third) {
181 |     auto pother = dynamic_cast<PythonSymNodeImpl*>(other.get());
182 |     auto pthird = dynamic_cast<PythonSymNodeImpl*>(third.get());
183 |     TORCH_CHECK(pother);
184 |     TORCH_CHECK(pthird);
185 |     py::gil_scoped_acquire acquire;
186 |     auto r = getPyObj().attr(fname)(pother->getPyObj(), pthird->getPyObj());
187 |     return c10::make_intrusive<PythonSymNodeImpl>(r);
188 |   }
189 | 
190 |   c10::SymNode dispatch_common_(const char* fname, const c10::SymNode& other) {
191 |     auto pother = dynamic_cast<PythonSymNodeImpl*>(other.get());
192 |     TORCH_CHECK(pother);
193 |     py::gil_scoped_acquire acquire;
194 |     auto r = getPyObj().attr(fname)(pother->getPyObj());
195 |     return c10::make_intrusive<PythonSymNodeImpl>(r);
196 |   }
197 | 
198 |   c10::SymNode dispatch_common_(const char* fname) {
199 |     py::gil_scoped_acquire acquire;
200 |     auto r = getPyObj().attr(fname)();
201 |     return c10::make_intrusive<PythonSymNodeImpl>(r);
202 |   }
203 | 
204 |   c10::SymNode add(const c10::SymNode& other) override {
205 |     return dispatch_common_(__func__, other);
206 |   }
207 | 
208 |   c10::SymNode sub(const c10::SymNode& other) override {
209 |     return dispatch_common_(__func__, other);
210 |   }
211 | 
212 |   c10::SymNode mul(const c10::SymNode& other) override {
213 |     return dispatch_common_(__func__, other);
214 |   }
215 | 
216 |   c10::SymNode truediv(const c10::SymNode& other) override {
217 |     return dispatch_common_(__func__, other);
218 |   }
219 | 
```
- EN: Implements routines such as `str`, `getPyObj`, `_graph_repr`, `dispatch_sym_ite_`, `dispatch_common_` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `str`、`getPyObj`、`_graph_repr`、`dispatch_sym_ite_`、`dispatch_common_` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 220-275
```cpp
220 |   c10::SymNode float_truediv(const c10::SymNode& other) override {
221 |     return dispatch_common_(__func__, other);
222 |   }
223 | 
224 |   c10::SymNode int_truediv(const c10::SymNode& other) override {
225 |     return dispatch_common_(__func__, other);
226 |   }
227 | 
228 |   c10::SymNode pow(const c10::SymNode& other) override {
229 |     return dispatch_common_(__func__, other);
230 |   }
231 | 
232 |   c10::SymNode float_pow(const c10::SymNode& other) override {
233 |     return dispatch_common_(__func__, other);
234 |   }
235 | 
236 |   c10::SymNode pow_by_natural(const c10::SymNode& other) override {
237 |     return dispatch_common_(__func__, other);
238 |   }
239 | 
240 |   c10::SymNode floordiv(const c10::SymNode& other) override {
241 |     return dispatch_common_(__func__, other);
242 |   }
243 | 
244 |   c10::SymNode int_floordiv(const c10::SymNode& other) override {
245 |     return dispatch_common_(__func__, other);
246 |   }
247 | 
248 |   c10::SymNode mod(const c10::SymNode& other) override {
249 |     return dispatch_common_(__func__, other);
250 |   }
251 | 
252 |   c10::SymNode eq(const c10::SymNode& other) override {
253 |     return dispatch_common_(__func__, other);
254 |   }
255 | 
256 |   c10::SymNode ne(const c10::SymNode& other) override {
257 |     return dispatch_common_(__func__, other);
258 |   }
259 | 
260 |   c10::SymNode gt(const c10::SymNode& other) override {
261 |     return dispatch_common_(__func__, other);
262 |   }
263 | 
264 |   c10::SymNode lt(const c10::SymNode& other) override {
265 |     return dispatch_common_(__func__, other);
266 |   }
267 | 
268 |   c10::SymNode le(const c10::SymNode& other) override {
269 |     return dispatch_common_(__func__, other);
270 |   }
271 | 
272 |   c10::SymNode ge(const c10::SymNode& other) override {
273 |     return dispatch_common_(__func__, other);
274 |   }
275 | 
```
- EN: Implements routines such as `float_truediv`, `dispatch_common_`, `int_truediv`, `pow`, `float_pow` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `float_truediv`、`dispatch_common_`、`int_truediv`、`pow`、`float_pow` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 276-325
```cpp
276 |   c10::SymNode sym_min(const c10::SymNode& other) override {
277 |     return dispatch_common_(__func__, other);
278 |   }
279 |   c10::SymNode sym_max(const c10::SymNode& other) override {
280 |     return dispatch_common_(__func__, other);
281 |   }
282 | 
283 |   c10::SymNode sym_and(const c10::SymNode& other) override {
284 |     return dispatch_common_(__func__, other);
285 |   }
286 | 
287 |   c10::SymNode sym_or(const c10::SymNode& other) override {
288 |     return dispatch_common_(__func__, other);
289 |   }
290 | 
291 |   c10::SymNode sym_ite(const c10::SymNode& other, const c10::SymNode& third)
292 |       override {
293 |     return dispatch_sym_ite_(__func__, other, third);
294 |   }
295 | 
296 |   c10::SymNode sym_not() override {
297 |     return dispatch_common_(__func__);
298 |   }
299 | 
300 |   c10::SymNode ceil() override {
301 |     return dispatch_common_(__func__);
302 |   }
303 | 
304 |   c10::SymNode floor() override {
305 |     return dispatch_common_(__func__);
306 |   }
307 | 
308 |   c10::SymNode neg() override {
309 |     return dispatch_common_(__func__);
310 |   }
311 | 
312 |   c10::SymNode clone() override {
313 |     return dispatch_common_(__func__);
314 |   }
315 | 
316 |   c10::SymNode sym_float() override {
317 |     return dispatch_common_(__func__);
318 |   }
319 | 
320 |   py::handle getPyObj() const {
321 |     return py::handle(pyobj_->ptr(getPyInterpreter()));
322 |   }
323 |   std::shared_ptr<c10::SafePyObject> pyobj_ = nullptr;
324 | };
325 | 
```
- EN: Implements routines such as `sym_min`, `dispatch_common_`, `sym_max`, `sym_and`, `sym_or` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `sym_min`、`dispatch_common_`、`sym_max`、`sym_and`、`sym_or` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 326-327
```cpp
326 | } // namespace impl
327 | } // namespace torch
```
- EN: Continues the file's main role: Bridges C++ components to Python-facing APIs or bindings.
- CN: 继续承担本文件的主要职责：把 C++ 组件桥接到面向 Python 的 API 或绑定。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `PythonSymNodeImpl`.
  - CN: `PythonSymNodeImpl`。
- **Important routines / 重要例程**
  - EN: `get_symint_class`, `get_symfloat_class`, `get_symbool_class`, `get_dynint_class`, `is_symint`, `is_symfloat`, `is_symbool`, `is_dynint`.
  - CN: `get_symint_class`、`get_symfloat_class`、`get_symbool_class`、`get_dynint_class`、`is_symint`、`is_symfloat`、`is_symbool`、`is_dynint`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch`, `impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch`、`impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/core/SafePyObject.h>`, `<c10/core/SymNodeImpl.h>`, `<torch/csrc/PyInterpreter.h>`, `<torch/csrc/autograd/python_variable.h>`, `<torch/csrc/utils/pybind.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
