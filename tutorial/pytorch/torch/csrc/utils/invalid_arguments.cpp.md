# invalid_arguments.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/invalid_arguments.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-53
```cpp
 1 | #include <torch/csrc/utils/invalid_arguments.h>
 2 | 
 3 | #include <torch/csrc/utils/python_strings.h>
 4 | 
 5 | #include <c10/util/irange.h>
 6 | 
 7 | #include <algorithm>
 8 | #include <memory>
 9 | #include <unordered_map>
10 | 
11 | namespace torch {
12 | 
13 | namespace {
14 | 
15 | std::string py_typename(PyObject* object) {
16 |   return Py_TYPE(object)->tp_name;
17 | }
18 | 
19 | struct Type {
20 |   Type() = default;
21 |   Type(const Type&) = default;
22 |   Type& operator=(const Type&) = default;
23 |   Type(Type&&) noexcept = default;
24 |   Type& operator=(Type&&) noexcept = default;
25 |   virtual bool is_matching(PyObject* object) = 0;
26 |   virtual ~Type() = default;
27 | };
28 | 
29 | struct SimpleType : public Type {
30 |   SimpleType(std::string& name) : name(name) {}
31 | 
32 |   bool is_matching(PyObject* object) override {
33 |     return py_typename(object) == name;
34 |   }
35 | 
36 |   std::string name;
37 | };
38 | 
39 | struct MultiType : public Type {
40 |   MultiType(std::initializer_list<std::string> accepted_types)
41 |       : types(accepted_types) {}
42 | 
43 |   bool is_matching(PyObject* object) override {
44 |     auto it = std::find(types.begin(), types.end(), py_typename(object));
45 |     return it != types.end();
46 |   }
47 | 
48 |   std::vector<std::string> types;
49 | };
50 | 
51 | struct NullableType : public Type {
52 |   NullableType(std::unique_ptr<Type> type) : type(std::move(type)) {}
53 | 
```
- EN: Brings in project headers such as `<torch/csrc/utils/invalid_arguments.h>`, `<torch/csrc/utils/python_strings.h>`, `<c10/util/irange.h>` and system or third-party headers such as `<algorithm>`, `<memory>`, `<unordered_map>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `Type`, `SimpleType`, `MultiType`, `NullableType` that structure the state handled by this file. Implements routines such as `py_typename`, `is_matching` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/utils/invalid_arguments.h>`、`<torch/csrc/utils/python_strings.h>`、`<c10/util/irange.h>`以及系统或第三方头文件，例如 `<algorithm>`、`<memory>`、`<unordered_map>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `Type`、`SimpleType`、`MultiType`、`NullableType` 等数据抽象，用来组织本文件处理的状态。 实现了 `py_typename`、`is_matching` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 54-107
```cpp
 54 |   bool is_matching(PyObject* object) override {
 55 |     return Py_IsNone(object) || type->is_matching(object);
 56 |   }
 57 | 
 58 |   std::unique_ptr<Type> type;
 59 | };
 60 | 
 61 | struct TupleType : public Type {
 62 |   TupleType(std::vector<std::unique_ptr<Type>> types)
 63 |       : types(std::move(types)) {}
 64 | 
 65 |   bool is_matching(PyObject* object) override {
 66 |     if (!PyTuple_Check(object))
 67 |       return false;
 68 |     auto num_elements = PyTuple_GET_SIZE(object);
 69 |     if (num_elements != (long)types.size())
 70 |       return false;
 71 |     for (const auto i : c10::irange(num_elements)) {
 72 |       if (!types[i]->is_matching(PyTuple_GET_ITEM(object, i)))
 73 |         return false;
 74 |     }
 75 |     return true;
 76 |   }
 77 | 
 78 |   std::vector<std::unique_ptr<Type>> types;
 79 | };
 80 | 
 81 | struct SequenceType : public Type {
 82 |   SequenceType(std::unique_ptr<Type> type) : type(std::move(type)) {}
 83 | 
 84 |   bool is_matching(PyObject* object) override {
 85 |     if (!PySequence_Check(object))
 86 |       return false;
 87 |     auto num_elements = PySequence_Length(object);
 88 |     for (const auto i : c10::irange(num_elements)) {
 89 |       if (!type->is_matching(
 90 |               py::reinterpret_steal<py::object>(PySequence_GetItem(object, i))
 91 |                   .ptr()))
 92 |         return false;
 93 |     }
 94 |     return true;
 95 |   }
 96 | 
 97 |   std::unique_ptr<Type> type;
 98 | };
 99 | 
100 | struct Argument {
101 |   Argument(std::string name, std::unique_ptr<Type> type)
102 |       : name(std::move(name)), type(std::move(type)) {}
103 | 
104 |   std::string name;
105 |   std::unique_ptr<Type> type;
106 | };
107 | 
```
- EN: Defines or extends data abstractions such as `TupleType`, `SequenceType`, `Argument` that structure the state handled by this file. Implements routines such as `is_matching`, `Py_IsNone` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 定义或扩展了 `TupleType`、`SequenceType`、`Argument` 等数据抽象，用来组织本文件处理的状态。 实现了 `is_matching`、`Py_IsNone` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 108-155
```cpp
108 | struct Option {
109 |   Option(std::vector<Argument> arguments, bool is_variadic, bool has_out)
110 |       : arguments(std::move(arguments)),
111 |         is_variadic(is_variadic),
112 |         has_out(has_out) {}
113 |   Option(bool is_variadic, bool has_out)
114 |       : is_variadic(is_variadic), has_out(has_out) {}
115 |   Option(const Option&) = delete;
116 |   Option(Option&& other) noexcept = default;
117 |   Option& operator=(const Option&) = delete;
118 |   Option& operator=(Option&&) = delete;
119 |   ~Option() = default;
120 | 
121 |   std::vector<Argument> arguments;
122 |   bool is_variadic;
123 |   bool has_out;
124 | };
125 | 
126 | std::vector<std::string> _splitString(
127 |     const std::string& s,
128 |     const std::string& delim) {
129 |   std::vector<std::string> tokens;
130 |   size_t start = 0;
131 |   size_t end = 0;
132 |   while ((end = s.find(delim, start)) != std::string::npos) {
133 |     tokens.push_back(s.substr(start, end - start));
134 |     start = end + delim.length();
135 |   }
136 |   tokens.push_back(s.substr(start));
137 |   return tokens;
138 | }
139 | 
140 | std::unique_ptr<Type> _buildType(std::string type_name, bool is_nullable) {
141 |   std::unique_ptr<Type> result;
142 |   if (type_name == "float") {
143 |     result = std::make_unique<MultiType>(MultiType{"float", "int", "long"});
144 |   } else if (type_name == "int") {
145 |     result = std::make_unique<MultiType>(MultiType{"int", "long"});
146 |   } else if (type_name.find("tuple[") == 0) {
147 |     auto type_list = type_name.substr(6);
148 |     type_list.pop_back();
149 |     std::vector<std::unique_ptr<Type>> types;
150 |     for (auto& type : _splitString(type_list, ","))
151 |       types.emplace_back(_buildType(type, false));
152 |     result = std::make_unique<TupleType>(std::move(types));
153 |   } else if (type_name.find("sequence[") == 0) {
154 |     auto subtype = type_name.substr(9);
155 |     subtype.pop_back();
```
- EN: Defines or extends data abstractions such as `Option` that structure the state handled by this file. Implements routines such as `_splitString`, `_buildType` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 定义或扩展了 `Option` 等数据抽象，用来组织本文件处理的状态。 实现了 `_splitString`、`_buildType` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 156-207
```cpp
156 |     result = std::make_unique<SequenceType>(_buildType(subtype, false));
157 |   } else {
158 |     result = std::make_unique<SimpleType>(type_name);
159 |   }
160 |   if (is_nullable)
161 |     result = std::make_unique<NullableType>(std::move(result));
162 |   return result;
163 | }
164 | 
165 | std::pair<Option, std::string> _parseOption(
166 |     const std::string& _option_str,
167 |     const std::unordered_map<std::string, PyObject*>& kwargs) {
168 |   if (_option_str == "no arguments")
169 |     return std::pair<Option, std::string>(Option(false, false), _option_str);
170 |   bool has_out = false;
171 |   std::vector<Argument> arguments;
172 |   std::string printable_option = _option_str;
173 |   std::string option_str = _option_str.substr(1, _option_str.length() - 2);
174 | 
175 |   /// XXX: this is a hack only for the out arg in TensorMethods
176 |   auto out_pos = printable_option.find('#');
177 |   if (out_pos != std::string::npos) {
178 |     if (kwargs.count("out") > 0) {
179 |       std::string kwonly_part = printable_option.substr(out_pos + 1);
180 |       printable_option.erase(out_pos);
181 |       printable_option += "*, ";
182 |       printable_option += kwonly_part;
183 |     } else if (out_pos >= 2) {
184 |       printable_option.erase(out_pos - 2);
185 |       printable_option += ")";
186 |     } else {
187 |       printable_option.erase(out_pos);
188 |       printable_option += ")";
189 |     }
190 |     has_out = true;
191 |   }
192 | 
193 |   for (auto& arg : _splitString(option_str, ", ")) {
194 |     bool is_nullable = false;
195 |     auto type_start_idx = 0;
196 |     if (arg[type_start_idx] == '#') {
197 |       type_start_idx++;
198 |     }
199 |     if (arg[type_start_idx] == '[') {
200 |       is_nullable = true;
201 |       type_start_idx++;
202 |       arg.erase(arg.length() - std::string(" or None]").length());
203 |     }
204 | 
205 |     auto type_end_idx = arg.find_last_of(' ');
206 |     auto name_start_idx = type_end_idx + 1;
207 | 
```
- EN: Implements routines such as `_parseOption` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `_parseOption` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 208-259
```cpp
208 |     // "type ... name" => "type ... name"
209 |     //          ^              ^
210 |     auto dots_idx = arg.find("...");
211 |     if (dots_idx != std::string::npos)
212 |       type_end_idx -= 4;
213 | 
214 |     std::string type_name =
215 |         arg.substr(type_start_idx, type_end_idx - type_start_idx);
216 |     std::string name = arg.substr(name_start_idx);
217 | 
218 |     arguments.emplace_back(name, _buildType(type_name, is_nullable));
219 |   }
220 | 
221 |   bool is_variadic = option_str.find("...") != std::string::npos;
222 |   return std::pair<Option, std::string>(
223 |       Option(std::move(arguments), is_variadic, has_out),
224 |       std::move(printable_option));
225 | }
226 | 
227 | bool _argcountMatch(
228 |     const Option& option,
229 |     const std::vector<PyObject*>& arguments,
230 |     const std::unordered_map<std::string, PyObject*>& kwargs) {
231 |   auto num_expected = option.arguments.size();
232 |   auto num_got = arguments.size() + kwargs.size();
233 |   // Note: variadic functions don't accept kwargs, so it's ok
234 |   if (option.has_out && kwargs.count("out") == 0)
235 |     num_expected--;
236 |   return num_got == num_expected ||
237 |       (option.is_variadic && num_got > num_expected);
238 | }
239 | 
240 | std::string _formattedArgDesc(
241 |     const Option& option,
242 |     const std::vector<PyObject*>& arguments,
243 |     const std::unordered_map<std::string, PyObject*>& kwargs) {
244 |   std::string red;
245 |   std::string reset_red;
246 |   std::string green;
247 |   std::string reset_green;
248 |   if (isatty(1) && isatty(2)) {
249 |     red = "\33[31;1m";
250 |     reset_red = "\33[0m";
251 |     green = "\33[32;1m";
252 |     reset_green = "\33[0m";
253 |   } else {
254 |     red = "!";
255 |     reset_red = "!";
256 |     green = "";
257 |     reset_green = "";
258 |   }
259 | 
```
- EN: Implements routines such as `_argcountMatch`, `_formattedArgDesc` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `_argcountMatch`、`_formattedArgDesc` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 260-307
```cpp
260 |   auto num_args = arguments.size() + kwargs.size();
261 |   std::string result = "(";
262 |   for (const auto i : c10::irange(num_args)) {
263 |     bool is_kwarg = i >= arguments.size();
264 |     PyObject* arg =
265 |         is_kwarg ? kwargs.at(option.arguments[i].name) : arguments[i];
266 | 
267 |     bool is_matching = false;
268 |     if (i < option.arguments.size()) {
269 |       is_matching = option.arguments[i].type->is_matching(arg);
270 |     } else if (option.is_variadic) {
271 |       is_matching = option.arguments.back().type->is_matching(arg);
272 |     }
273 | 
274 |     if (is_matching)
275 |       result += green;
276 |     else
277 |       result += red;
278 |     if (is_kwarg)
279 |       result += option.arguments[i].name + "=";
280 |     bool is_tuple = PyTuple_Check(arg);
281 |     if (is_tuple || PyList_Check(arg)) {
282 |       result += py_typename(arg) + " of ";
283 |       auto num_elements = PySequence_Length(arg);
284 |       if (is_tuple) {
285 |         result += "(";
286 |       } else {
287 |         result += "[";
288 |       }
289 |       for (const auto i : c10::irange(num_elements)) {
290 |         if (i != 0) {
291 |           result += ", ";
292 |         }
293 |         result += py_typename(
294 |             py::reinterpret_steal<py::object>(PySequence_GetItem(arg, i))
295 |                 .ptr());
296 |       }
297 |       if (is_tuple) {
298 |         if (num_elements == 1) {
299 |           result += ",";
300 |         }
301 |         result += ")";
302 |       } else {
303 |         result += "]";
304 |       }
305 |     } else {
306 |       result += py_typename(arg);
307 |     }
```
- EN: Continues the file's main role: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
- CN: 继续承担本文件的主要职责：提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

### Lines 308-359
```cpp
308 |     if (is_matching)
309 |       result += reset_green;
310 |     else
311 |       result += reset_red;
312 |     result += ", ";
313 |   }
314 |   if (!arguments.empty())
315 |     result.erase(result.length() - 2);
316 |   result += ")";
317 |   return result;
318 | }
319 | 
320 | std::string _argDesc(
321 |     const std::vector<PyObject*>& arguments,
322 |     const std::unordered_map<std::string, PyObject*>& kwargs) {
323 |   std::string result = "(";
324 |   for (auto& arg : arguments)
325 |     result += std::string(py_typename(arg)) + ", ";
326 |   for (auto& kwarg : kwargs)
327 |     result += kwarg.first + "=" + py_typename(kwarg.second) + ", ";
328 |   if (!arguments.empty())
329 |     result.erase(result.length() - 2);
330 |   result += ")";
331 |   return result;
332 | }
333 | 
334 | std::vector<std::string> _tryMatchKwargs(
335 |     const Option& option,
336 |     const std::unordered_map<std::string, PyObject*>& kwargs) {
337 |   std::vector<std::string> unmatched;
338 |   // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
339 |   int64_t start_idx = option.arguments.size() - kwargs.size();
340 |   if (option.has_out && kwargs.count("out") == 0)
341 |     start_idx--;
342 |   if (start_idx < 0)
343 |     start_idx = 0;
344 |   for (auto& entry : kwargs) {
345 |     bool found = false;
346 |     for (unsigned int i = start_idx; i < option.arguments.size(); i++) {
347 |       if (option.arguments[i].name == entry.first) {
348 |         found = true;
349 |         break;
350 |       }
351 |     }
352 |     if (!found)
353 |       unmatched.push_back(entry.first);
354 |   }
355 |   return unmatched;
356 | }
357 | 
358 | } // anonymous namespace
359 | 
```
- EN: Implements routines such as `_argDesc`, `_tryMatchKwargs` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `_argDesc`、`_tryMatchKwargs` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 360-407
```cpp
360 | std::string format_invalid_args(
361 |     PyObject* given_args,
362 |     PyObject* given_kwargs,
363 |     const std::string& function_name,
364 |     const std::vector<std::string>& options) {
365 |   std::vector<PyObject*> args;
366 |   std::unordered_map<std::string, PyObject*> kwargs;
367 |   std::string error_msg;
368 |   error_msg.reserve(2000);
369 |   error_msg += function_name;
370 |   error_msg += " received an invalid combination of arguments - ";
371 | 
372 |   Py_ssize_t num_args = PyTuple_Size(given_args);
373 |   for (const auto i : c10::irange(num_args)) {
374 |     PyObject* arg = PyTuple_GET_ITEM(given_args, i);
375 |     args.push_back(arg);
376 |   }
377 | 
378 |   bool has_kwargs = given_kwargs && PyDict_Size(given_kwargs) > 0;
379 |   if (has_kwargs) {
380 |     PyObject *key = nullptr, *value = nullptr;
381 |     Py_ssize_t pos = 0;
382 | 
383 |     Py_BEGIN_CRITICAL_SECTION(given_kwargs);
384 |     while (PyDict_Next(given_kwargs, &pos, &key, &value)) {
385 |       kwargs.emplace(THPUtils_unpackString(key), value);
386 |     }
387 |     Py_END_CRITICAL_SECTION();
388 |   }
389 | 
390 |   if (options.size() == 1) {
391 |     auto pair = _parseOption(options[0], kwargs);
392 |     auto& option = pair.first;
393 |     auto& option_str = pair.second;
394 |     std::vector<std::string> unmatched_kwargs;
395 |     if (has_kwargs)
396 |       unmatched_kwargs = _tryMatchKwargs(option, kwargs);
397 |     if (!unmatched_kwargs.empty()) {
398 |       error_msg += "got unrecognized keyword arguments: ";
399 |       for (auto& kwarg : unmatched_kwargs)
400 |         error_msg += kwarg + ", ";
401 |       error_msg.erase(error_msg.length() - 2);
402 |     } else {
403 |       error_msg += "got ";
404 |       if (_argcountMatch(option, args, kwargs)) {
405 |         error_msg += _formattedArgDesc(option, args, kwargs);
406 |       } else {
407 |         error_msg += _argDesc(args, kwargs);
```
- EN: Implements routines such as `format_invalid_args` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; builds container state that later execution depends on.
- CN: 实现了 `format_invalid_args` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；构建后续执行依赖的容器状态。

### Lines 408-446
```cpp
408 |       }
409 |       error_msg += ", but expected ";
410 |       error_msg += option_str;
411 |     }
412 |   } else {
413 |     error_msg += "got ";
414 |     error_msg += _argDesc(args, kwargs);
415 |     error_msg += ", but expected one of:\n";
416 |     for (auto& option_str : options) {
417 |       auto pair = _parseOption(option_str, kwargs);
418 |       auto& option = pair.first;
419 |       auto& printable_option_str = pair.second;
420 |       error_msg += " * ";
421 |       error_msg += printable_option_str;
422 |       error_msg += "\n";
423 |       if (_argcountMatch(option, args, kwargs)) {
424 |         std::vector<std::string> unmatched_kwargs;
425 |         if (has_kwargs)
426 |           unmatched_kwargs = _tryMatchKwargs(option, kwargs);
427 |         if (!unmatched_kwargs.empty()) {
428 |           error_msg +=
429 |               "      didn't match because some of the keywords were incorrect: ";
430 |           for (auto& kwarg : unmatched_kwargs)
431 |             error_msg += kwarg + ", ";
432 |           error_msg.erase(error_msg.length() - 2);
433 |           error_msg += "\n";
434 |         } else {
435 |           error_msg +=
436 |               "      didn't match because some of the arguments have invalid types: ";
437 |           error_msg += _formattedArgDesc(option, args, kwargs);
438 |           error_msg += "\n";
439 |         }
440 |       }
441 |     }
442 |   }
443 |   return error_msg;
444 | }
445 | 
446 | } // namespace torch
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `Type`, `SimpleType`, `MultiType`, `NullableType`, `TupleType`, `SequenceType`.
  - CN: `Type`、`SimpleType`、`MultiType`、`NullableType`、`TupleType`、`SequenceType`。
- **Important routines / 重要例程**
  - EN: `py_typename`, `is_matching`, `Py_IsNone`, `_splitString`, `_buildType`, `_parseOption`, `_argcountMatch`, `_formattedArgDesc`.
  - CN: `py_typename`、`is_matching`、`Py_IsNone`、`_splitString`、`_buildType`、`_parseOption`、`_argcountMatch`、`_formattedArgDesc`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/utils/invalid_arguments.h>`, `<torch/csrc/utils/python_strings.h>`, `<c10/util/irange.h>`
- External includes / 外部头文件: `<algorithm>`, `<memory>`, `<unordered_map>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
