# util.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/util.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
  - CN: 实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Line-by-Line Analysis / 逐行分析

### Lines 1-80
```cpp
 1 | #include <torch/csrc/profiler/collection.h>
 2 | #include <torch/csrc/profiler/util.h>
 3 | 
 4 | #include <c10/util/ArrayRef.h>
 5 | #include <c10/util/irange.h>
 6 | #include <fmt/format.h>
 7 | #include <fmt/ranges.h>
 8 | 
 9 | #ifdef USE_KINETO
10 | #include <libkineto.h>
11 | #endif
12 | #ifdef USE_DISTRIBUTED
13 | #include <c10/util/hash.h>
14 | #include <torch/csrc/distributed/c10d/ParamCommsUtils.hpp>
15 | #endif // USE_DISTRIBUTED
16 | 
17 | namespace torch::profiler::impl {
18 | 
19 | namespace {
20 | std::optional<bool> soft_assert_raises_;
21 | } // namespace
22 | 
23 | void setSoftAssertRaises(std::optional<bool> value) {
24 |   soft_assert_raises_ = value;
25 | }
26 | 
27 | bool softAssertRaises() {
28 |   return soft_assert_raises_.value_or(false);
29 | }
30 | 
31 | void logSoftAssert(
32 |     // @lint-ignore CLANGTIDY
33 |     const char* func,
34 |     // @lint-ignore CLANGTIDY
35 |     const char* file,
36 |     // @lint-ignore CLANGTIDY
37 |     uint32_t line,
38 |     // @lint-ignore CLANGTIDY
39 |     const char* cond,
40 |     // @lint-ignore CLANGTIDY
41 |     const char* args) {
42 | #ifdef USE_KINETO
43 |   std::string error;
44 |   error = fmt::format(
45 |       "{} SOFT ASSERT FAILED at {}:{}, func: {}, args: {}",
46 |       cond,
47 |       file,
48 |       line,
49 |       func,
50 |       args);
51 |   // TODO: Implement profile_id and group_profile_id as 3rd/4th arguments.
52 |   kineto::logInvariantViolation(cond, error, "", "");
53 | #endif
54 | }
55 | 
56 | void logSoftAssert(
57 |     // @lint-ignore CLANGTIDY
58 |     const char* func,
59 |     // @lint-ignore CLANGTIDY
60 |     const char* file,
61 |     // @lint-ignore CLANGTIDY
62 |     uint32_t line,
63 |     // @lint-ignore CLANGTIDY
64 |     const char* cond,
65 |     // @lint-ignore CLANGTIDY
66 |     const std::string& args) {
67 | #ifdef USE_KINETO
68 |   std::string error;
69 |   error = fmt::format(
70 |       "{} SOFT ASSERT FAILED at {}:{}, func: {}, args: {}",
71 |       cond,
72 |       file,
73 |       line,
74 |       func,
75 |       args);
76 |   // TODO: Implement profile_id and group_profile_id as 3rd/4th arguments.
77 |   kineto::logInvariantViolation(cond, error, "", "");
78 | #endif
79 | }
80 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/util.h>`, `<c10/util/ArrayRef.h>`, `<c10/util/irange.h>` and system or third-party headers such as `<fmt/format.h>`, `<fmt/ranges.h>`, `<libkineto.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. Implements routines such as `setSoftAssertRaises`, `softAssertRaises`, `logSoftAssert` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/collection.h>`、`<torch/csrc/profiler/util.h>`、`<c10/util/ArrayRef.h>`、`<c10/util/irange.h>`以及系统或第三方头文件，例如 `<fmt/format.h>`、`<fmt/ranges.h>`、`<libkineto.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `setSoftAssertRaises`、`softAssertRaises`、`logSoftAssert` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 81-153
```cpp
 81 | // ----------------------------------------------------------------------------
 82 | // -- NVTX --------------------------------------------------------------------
 83 | // ----------------------------------------------------------------------------
 84 | std::string getNvtxStr(
 85 |     const char* name,
 86 |     int64_t sequence_nr,
 87 |     const std::vector<std::vector<int64_t>>& shapes,
 88 |     at::RecordFunctionHandle op_id,
 89 |     const std::list<std::pair<at::RecordFunctionHandle, int>>& input_op_ids) {
 90 |   if (sequence_nr >= -1 || !shapes.empty()) {
 91 |     std::string str;
 92 |     if (sequence_nr >= 0) {
 93 |       str = fmt::format("{}, seq = {}", name, sequence_nr);
 94 |     } else if (sequence_nr == -1) {
 95 |       str = name;
 96 |     } else {
 97 | #if defined(USE_ROCM)
 98 |       // Only ROCM supports < -1 sequence_nr
 99 |       str = name;
100 | #endif
101 |     }
102 |     if (op_id > 0) {
103 |       str = fmt::format("{}, op_id = {}", str, op_id);
104 |     }
105 |     if (!shapes.empty()) {
106 |       str = fmt::format("{}, sizes = {}", str, shapesToStr(shapes));
107 |     }
108 |     // Include the op ids of the input edges so
109 |     // you can build the network graph
110 |     if (!input_op_ids.empty()) {
111 |       str = fmt::format(
112 |           "{}, input_op_ids = {}", str, inputOpIdsToStr(input_op_ids));
113 |     }
114 |     return str;
115 |   } else {
116 |     return name;
117 |   }
118 | }
119 | 
120 | // ----------------------------------------------------------------------------
121 | // -- Op context (shapes, call stack) -----------------------------------------
122 | // ----------------------------------------------------------------------------
123 | std::vector<FileLineFunc> prepareCallstack(
124 |     const std::vector<jit::StackEntry>& cs) {
125 |   std::vector<FileLineFunc> entries;
126 |   entries.reserve(cs.size());
127 |   for (const auto& entry : cs) {
128 |     auto& range = entry.range;
129 |     if (range.source()) {
130 |       auto& src = range.source();
131 |       if (src && src->filename()) {
132 |         auto line =
133 |             src->starting_line_no() + src->lineno_for_offset(range.start());
134 |         entries.emplace_back(
135 |             // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
136 |             FileLineFunc{*(src->filename()), line, entry.filename});
137 |       }
138 |     }
139 |   }
140 |   return entries;
141 | }
142 | 
143 | std::vector<std::string> callstackStr(const std::vector<FileLineFunc>& cs) {
144 |   std::vector<std::string> cs_str;
145 |   cs_str.reserve(cs.size());
146 |   for (const auto& entry : cs) {
147 |     std::stringstream loc;
148 |     loc << entry.filename << '(' << entry.line << "): " << entry.funcname;
149 |     cs_str.push_back(loc.str());
150 |   }
151 |   return cs_str;
152 | }
153 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `getNvtxStr`, `prepareCallstack`, `callstackStr` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `getNvtxStr`、`prepareCallstack`、`callstackStr` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 154-229
```cpp
154 | std::string stacksToStr(
155 |     const std::vector<std::string>& stacks,
156 |     const char* delim) {
157 |   std::ostringstream oss;
158 |   std::transform(
159 |       stacks.begin(),
160 |       stacks.end(),
161 |       std::ostream_iterator<std::string>(oss, delim),
162 |       [](std::string s) -> std::string {
163 | #ifdef _WIN32
164 |         // replace the windows backslash with forward slash
165 |         std::replace(s.begin(), s.end(), '\\', '/');
166 | #endif
167 |         return s;
168 |       });
169 |   auto rc = oss.str();
170 |   return "\"" + rc + "\"";
171 | }
172 | 
173 | static std::vector<std::vector<int64_t>> flattenList(
174 |     const c10::List<c10::IValue>& list) {
175 |   std::vector<std::vector<int64_t>> tensor_dims;
176 |   for (const c10::IValue& input : list) {
177 |     if (input.isTensor()) {
178 |       const at::Tensor& tensor = input.toTensor();
179 |       if (tensor.defined()) {
180 |         tensor_dims.push_back(input.toTensor().sizes().vec());
181 |       }
182 |     }
183 |   }
184 |   return tensor_dims;
185 | }
186 | 
187 | std::vector<std::vector<int64_t>> inputSizes(
188 |     const at::RecordFunction& fn,
189 |     bool flatten_list_enabled) {
190 |   std::vector<std::vector<int64_t>> sizes;
191 |   sizes.reserve(fn.inputs().size());
192 |   for (const c10::IValue& input : fn.inputs()) {
193 |     if (input.isTensor()) {
194 |       const at::Tensor& tensor = input.toTensor();
195 |       if (tensor.defined()) {
196 |         sizes.push_back(input.toTensor().sizes().vec());
197 |       } else {
198 |         sizes.emplace_back();
199 |       }
200 |     } else if (input.isList()) {
201 |       std::vector<std::vector<int64_t>> tmp_sizes;
202 |       if (flatten_list_enabled) {
203 |         tmp_sizes = flattenList(input.toList());
204 |       }
205 |       // Extend the current sizes array by the array returned from input sizes
206 |       if (!tmp_sizes.empty()) {
207 |         sizes.insert(sizes.end(), tmp_sizes.begin(), tmp_sizes.end());
208 |       } else {
209 |         sizes.emplace_back();
210 |       }
211 |     } else {
212 |       sizes.emplace_back();
213 |     }
214 |   }
215 |   return sizes;
216 | }
217 | 
218 | std::string shapesToStr(const std::vector<std::vector<int64_t>>& shapes) {
219 |   std::string str("[");
220 |   for (const auto t_idx : c10::irange(shapes.size())) {
221 |     if (t_idx > 0) {
222 |       str = fmt::format("{}, ", str);
223 |     }
224 |     str = fmt::format("{}{}", str, shapeToStr(shapes[t_idx]));
225 |   }
226 |   str = fmt::format("{}]", str);
227 |   return str;
228 | }
229 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `stacksToStr`, `flattenList`, `inputSizes`, `shapesToStr`, `str` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `stacksToStr`、`flattenList`、`inputSizes`、`shapesToStr`、`str` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态。

### Lines 230-301
```cpp
230 | std::string variantShapesToStr(const std::vector<shape>& shapes) {
231 |   std::string str("[");
232 |   for (const auto t_idx : c10::irange(shapes.size())) {
233 |     if (t_idx > 0) {
234 |       str = fmt::format("{}, ", str);
235 |     }
236 |     if (std::holds_alternative<std::vector<int64_t>>(shapes[t_idx])) {
237 |       const auto& shape = std::get<std::vector<int64_t>>(shapes[t_idx]);
238 |       str = fmt::format("{}{}", str, shapeToStr(shape));
239 |     } else if (std::holds_alternative<std::vector<std::vector<int64_t>>>(
240 |                    shapes[t_idx])) {
241 |       const auto& tensor_shape =
242 |           std::get<std::vector<std::vector<int64_t>>>(shapes[t_idx]);
243 |       if (tensor_shape.size() > TENSOR_LIST_DISPLAY_LENGTH_LIMIT) {
244 |         // skip if the tensor list is too long
245 |         str = fmt::format("{}[]", str);
246 |         continue;
247 |       }
248 |       str = fmt::format("{}[", str);
249 |       for (const auto s_idx : c10::irange(tensor_shape.size())) {
250 |         if (s_idx > 0) {
251 |           str = fmt::format("{}, ", str);
252 |         }
253 |         str = fmt::format("{}{}", str, shapeToStr(tensor_shape[s_idx]));
254 |       }
255 |       str = fmt::format("{}]", str);
256 |     }
257 |   }
258 |   str = fmt::format("{}]", str);
259 |   return str;
260 | }
261 | 
262 | std::string shapeToStr(const std::vector<int64_t>& shape) {
263 |   std::string str("[");
264 |   for (const auto s_idx : c10::irange(shape.size())) {
265 |     if (s_idx > 0) {
266 |       str = fmt::format("{}, ", str);
267 |     }
268 |     str = fmt::format("{}{}", str, shape[s_idx]);
269 |   }
270 |   str = fmt::format("{}]", str);
271 |   return str;
272 | }
273 | 
274 | std::string inputOpIdsToStr(
275 |     const std::list<std::pair<at::RecordFunctionHandle, int>>& input_op_ids) {
276 |   std::string str("[");
277 |   int idx = 0;
278 | 
279 |   for (const auto& op_id_info_pair : input_op_ids) {
280 |     if (idx++ > 0) {
281 |       str = fmt::format("{}, ", str);
282 |     }
283 |     // (OpId,OutputNr)
284 |     str = fmt::format(
285 |         "{}({},{})", str, op_id_info_pair.first, op_id_info_pair.second);
286 |   }
287 |   str = fmt::format("{}]", str);
288 |   return str;
289 | }
290 | 
291 | std::string strListToStr(const std::vector<std::string>& types) {
292 |   if (types.empty()) {
293 |     return "[]";
294 |   } else {
295 |     std::ostringstream oss;
296 |     std::transform(
297 |         types.begin(),
298 |         types.end(),
299 |         std::ostream_iterator<std::string>(oss, ", "),
300 |         [](const std::string& s) -> std::string { return "\"" + s + "\""; });
301 |     auto rc = oss.str();
```
- EN: Implements routines such as `variantShapesToStr`, `str`, `shapeToStr`, `inputOpIdsToStr`, `strListToStr` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `variantShapesToStr`、`str`、`shapeToStr`、`inputOpIdsToStr`、`strListToStr` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 302-377
```cpp
302 |     rc.erase(rc.length() - 2); // remove last ", "
303 |     return "[" + rc + "]";
304 |   }
305 | }
306 | std::string ivalueToStr(const c10::IValue& val, bool isString) {
307 |   std::stringstream ss;
308 |   if (val.isNone()) {
309 |     return "\"None\"";
310 |   } else {
311 |     ss.str("");
312 |     if (isString) {
313 |       ss << '"';
314 |     }
315 |     ss << val;
316 |     if (isString) {
317 |       ss << '"';
318 |     }
319 |     std::string mystr = ss.str();
320 | 
321 |     // For boolean the values that ivalue gives is "True" and "False" but
322 |     // json only takes "true" and "false" so we convert the string to lower case
323 |     if (val.isBool()) {
324 |       for (char& c : mystr) {
325 |         c = static_cast<char>(std::tolower(c));
326 |       }
327 |     }
328 | 
329 |     // A double quote can cause issues with the chrome tracing so force
330 |     // all inputs to not contain more than the 2 we add in this function
331 |     auto count = std::count(mystr.begin(), mystr.end(), '"');
332 |     return count > 2 ? "\"None\"" : mystr;
333 |   }
334 | }
335 | 
336 | std::string ivalueListToStr(const std::vector<c10::IValue>& list) {
337 |   std::vector<std::string> concrete_str_inputs;
338 |   std::stringstream ss;
339 |   for (const auto& val : list) {
340 |     if (val.isNone()) {
341 |       concrete_str_inputs.emplace_back("");
342 |     } else {
343 |       ss.str("");
344 |       ss << val;
345 |       concrete_str_inputs.emplace_back(ss.str());
346 |     }
347 |   }
348 |   return strListToStr(concrete_str_inputs);
349 | }
350 | 
351 | std::vector<std::string> inputTypes(const at::RecordFunction& fn) {
352 |   std::vector<std::string> types;
353 |   types.reserve(fn.inputs().size());
354 |   for (const c10::IValue& input : fn.inputs()) {
355 |     if (input.isTensor()) {
356 |       const at::Tensor& tensor = input.toTensor();
357 |       if (tensor.defined()) {
358 |         types.push_back(
359 |             static_cast<std::string>(input.toTensor().dtype().name()));
360 |       } else {
361 |         types.emplace_back();
362 |       }
363 |     } else if (input.isScalar() || input.isList()) {
364 |       types.push_back(input.tagKind());
365 |     } else {
366 |       types.emplace_back();
367 |     }
368 |   }
369 |   return types;
370 | }
371 | 
372 | // ----------------------------------------------------------------------------
373 | // -- NCCL Metadata -----------------------------------------------------------
374 | // ----------------------------------------------------------------------------
375 | 
376 | static constexpr int32_t kTruncateLength = 30;
377 | 
```
- EN: Implements routines such as `ivalueToStr`, `ivalueListToStr`, `strListToStr`, `inputTypes` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `ivalueToStr`、`ivalueListToStr`、`strListToStr`、`inputTypes` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 378-450
```cpp
378 | template <typename ListLikeType>
379 | static inline std::string format_list(
380 |     ListLikeType list,
381 |     bool truncate,
382 |     bool with_escaped_quotes = true) {
383 |   if (truncate && list.size() > kTruncateLength) {
384 |     if (with_escaped_quotes == true) {
385 |       auto x = fmt::format(
386 |           "\"[{}, ..., {}]\"",
387 |           fmt::join(list.begin(), list.begin() + kTruncateLength - 1, ", "),
388 |           *std::prev(list.end()));
389 |       return x;
390 |     } else {
391 |       auto x = fmt::format(
392 |           "[{}, ..., {}]",
393 |           fmt::join(list.begin(), list.begin() + kTruncateLength - 1, ", "),
394 |           *std::prev(list.end()));
395 |       return x;
396 |     }
397 |   }
398 |   if (with_escaped_quotes == true) {
399 |     auto x = fmt::format("\"[{}]\"", fmt::join(list.begin(), list.end(), ", "));
400 |     return x;
401 |   } else {
402 |     auto x = fmt::format("[{}]", fmt::join(list.begin(), list.end(), ", "));
403 |     return x;
404 |   }
405 | }
406 | 
407 | std::pair<bool, std::variant<int, std::vector<int>>> findStartAddrForTensors(
408 |     const c10::IValue& val) {
409 |   if (val.isTensor()) {
410 |     // Store hints about where the input starts in memory.
411 |     // Useful for debugging memory access patterns.
412 |     const auto& tensor = val.toTensor();
413 |     const int result = getTensorStartHint(tensor);
414 |     return {false, result};
415 |   } else if (val.isTuple()) {
416 |     const auto& val_tuple = val.toTupleRef().elements();
417 |     size_t tuple_size = val_tuple.size();
418 |     std::vector<int> responses;
419 |     responses.reserve(tuple_size);
420 |     for (const auto j : c10::irange(tuple_size)) {
421 |       auto [is_list, res] = findStartAddrForTensors(val_tuple[j]);
422 |       if (is_list) {
423 |         const auto& vec_res = std::get<std::vector<int>>(res);
424 |         responses.insert(responses.end(), vec_res.begin(), vec_res.end());
425 |       } else {
426 |         responses.push_back(std::get<int>(res));
427 |       }
428 |     }
429 |     return {true, responses};
430 |   } else if (val.isList()) {
431 |     const auto& val_list = val.toList();
432 |     size_t list_size = val_list.size();
433 |     std::vector<int> responses;
434 |     responses.reserve(list_size);
435 |     for (const auto j : c10::irange(list_size)) {
436 |       auto [is_list, res] = findStartAddrForTensors(val_list[j]);
437 |       if (is_list) {
438 |         auto const& vec_res = std::get<std::vector<int>>(res);
439 |         responses.insert(responses.end(), vec_res.begin(), vec_res.end());
440 |       } else {
441 |         responses.push_back(std::get<int>(res));
442 |       }
443 |     }
444 |     return {true, responses};
445 |   } else {
446 |     // push back an invalid value for indices representing non-tensor inputs
447 |     return {false, -1};
448 |   }
449 | }
450 | 
```
- EN: Implements routines such as `format_list`, `findStartAddrForTensors` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on.
- CN: 实现了 `format_list`、`findStartAddrForTensors` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态。

### Lines 451-528
```cpp
451 | std::unordered_map<std::string, std::string> saveNcclMeta(
452 |     // @lint-ignore CLANGTIDY
453 |     const at::RecordFunction& fn,
454 |     // @lint-ignore CLANGTIDY
455 |     const SaveNcclMetaConfig& config) {
456 |   std::unordered_map<std::string, std::string> map;
457 | #ifdef USE_DISTRIBUTED
458 |   auto debugInfo = dynamic_cast<ParamCommsDebugInfo*>(
459 |       c10::ThreadLocalDebugInfo::get(c10::DebugInfoKind::PARAM_COMMS_INFO));
460 | 
461 |   if (config.introspectMetadata) {
462 |     if (debugInfo == nullptr) {
463 |       LOG(WARNING) << "ParamCommsDebugInfo not available for function: "
464 |                    << fn.name();
465 |       return map;
466 |     }
467 |     auto& collective_name = debugInfo->getCollectiveName();
468 |     map.emplace(kCommsName, fmt::format("\"{}\"", collective_name));
469 |     map.emplace(
470 |         kDtype, fmt::format("\"{}\"", c10::toString(debugInfo->getDType())));
471 |     map.emplace(kInMsgNelems, std::to_string(debugInfo->getInMessageNelems()));
472 |     map.emplace(
473 |         kOutMsgNelems, std::to_string(debugInfo->getOutMessageNelems()));
474 | 
475 |     auto& inSplitSizes = debugInfo->getInputSplitSizes();
476 |     map.emplace(kInSplit, format_list(inSplitSizes, config.truncate));
477 | 
478 |     auto& outSplitSizes = debugInfo->getOutputSplitSizes();
479 |     map.emplace(kOutSplit, format_list(outSplitSizes, config.truncate));
480 | 
481 |     auto globalRankStart = debugInfo->getGlobalRankStart();
482 |     if (globalRankStart >= 0) {
483 |       map.emplace(kGlobalRankStart, std::to_string(globalRankStart));
484 |     }
485 |     auto globalRankStride = debugInfo->getGlobalRankStride();
486 |     if (globalRankStride > 0) {
487 |       map.emplace(kGlobalRankStride, std::to_string(globalRankStride));
488 |     }
489 |     map.emplace(kGroupSize, std::to_string(debugInfo->getWorldSize()));
490 |     auto& group_name = debugInfo->getProcessGroupName();
491 |     if (!group_name.empty()) {
492 |       map.emplace(kProcessGroupName, fmt::format("\"{}\"", group_name));
493 |     }
494 |     auto& group_desc = debugInfo->getProcessGroupDesc();
495 |     if (!group_desc.empty()) {
496 |       map.emplace(kProcessGroupDesc, fmt::format("\"{}\"", group_desc));
497 |     }
498 |     auto& groupRanks = debugInfo->getGroupRanks();
499 |     map.emplace(kGroupRanks, format_list(groupRanks, config.truncate));
500 | 
501 |     auto rank = debugInfo->getRank();
502 |     map.emplace(kRank, std::to_string(rank));
503 |     int nRanks = static_cast<int>(groupRanks.size());
504 |     if (collective_name == "send") {
505 |       if (rank >= 0 && rank < nRanks) {
506 |         map.emplace(kP2pDst, std::to_string(groupRanks[rank]));
507 |       }
508 |     } else if (collective_name == "recv") {
509 |       if (rank >= 0 && rank < nRanks) {
510 |         map.emplace(kP2pSrc, std::to_string(groupRanks[rank]));
511 |       }
512 |     }
513 | 
514 |     auto seqNum = debugInfo->getSequenceNumber();
515 |     if (seqNum >= 0) {
516 |       map.emplace(kSeqNum, std::to_string(seqNum));
517 | 
518 |       size_t comms_id = c10::get_hash(
519 |           debugInfo->getProcessGroupName(),
520 |           seqNum,
521 |           debugInfo->getIsP2P(),
522 |           globalRankStart,
523 |           globalRankStride,
524 |           debugInfo->getWorldSize());
525 |       map.emplace(kCommsId, std::to_string(comms_id));
526 |     }
527 |   }
528 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `saveNcclMeta` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `saveNcclMeta` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态。

### Lines 529-604
```cpp
529 |   map.emplace(
530 |       kIsAsynchronizedOp, std::to_string(debugInfo->isAsynchronizedOp()));
531 | 
532 |   if (get_record_tensor_addrs_enabled()) {
533 |     std::vector<std::string> addressList;
534 |     if (config.introspectInputs) {
535 |       auto num_inputs = fn.num_inputs();
536 |       const auto inputs = fn.inputs();
537 |       if (checkFunctionInputsForLogging(fn)) {
538 |         // need to account for Stack mode where the inputs are at the end.
539 |         size_t input_start = inputs.size() - num_inputs;
540 |         for (const auto i : c10::irange(input_start, inputs.size())) {
541 |           const c10::IValue& val = inputs[i];
542 |           auto [is_list, result] = findStartAddrForTensors(val);
543 |           if (is_list) {
544 |             auto const& list_result = std::get<std::vector<int>>(result);
545 |             addressList.push_back(
546 |                 format_list(list_result, config.truncate, false));
547 |           } else {
548 |             auto scalar_result = std::get<int>(result);
549 |             addressList.push_back(std::to_string(scalar_result));
550 |           }
551 |           // today we record a lot of metadata in record_param_comms that shows
552 |           // up as inputs. here we only need the addresses of the first inputs,
553 |           // which are the real tensor inputs to the collective call. So let's
554 |           // break out of the loop here.
555 |           break;
556 |         }
557 |         map.emplace(kInTensorsStart, format_list(addressList, false));
558 |         addressList.clear();
559 |       }
560 |     }
561 |     if (config.introspectOutputs) {
562 |       const auto& outputs = fn.outputs();
563 |       auto num_outputs = fn.num_outputs();
564 |       if (checkFunctionOutputsForLogging(fn)) {
565 |         // need to account for Stack mode where the outputs are at the end.
566 |         size_t output_start = outputs.size() - num_outputs;
567 |         for (const auto i : c10::irange(output_start, outputs.size())) {
568 |           const c10::IValue& val = outputs[i];
569 |           auto [is_list, result] = findStartAddrForTensors(val);
570 |           if (is_list) {
571 |             auto const& list_result = std::get<std::vector<int>>(result);
572 |             addressList.push_back(
573 |                 format_list(list_result, config.truncate, false));
574 |           } else {
575 |             auto scalar_result = std::get<int>(result);
576 |             addressList.push_back(std::to_string(scalar_result));
577 |           }
578 |         }
579 |         map.emplace(kOutTensorsStart, format_list(addressList, false));
580 |         addressList.clear();
581 |       }
582 |     }
583 |   }
584 | #endif // USE_DISTRIBUTED
585 |   return map;
586 | }
587 | 
588 | // ----------------------------------------------------------------------------
589 | // -- FLOPS -------------------------------------------------------------------
590 | // ----------------------------------------------------------------------------
591 | static constexpr auto kConv2dStride = 3;
592 | static constexpr auto kConv2dPadding = 4;
593 | static constexpr auto kConv2dDilation = 5;
594 | static constexpr auto kConv2dGroups = 6;
595 | 
596 | // List of supported operators
597 | static constexpr auto kConv2dOp = "aten::conv2d";
598 | static constexpr auto kMMOp = "aten::mm";
599 | static constexpr auto kAddMMOp = "aten::addmm";
600 | static constexpr auto kMulOp = "aten::mul";
601 | static constexpr auto kAddOp = "aten::add";
602 | static constexpr auto kBMMOp = "aten::bmm";
603 | static constexpr auto kBAddBMMOp = "aten::baddbmm";
604 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态。

### Lines 605-682
```cpp
605 | static constexpr auto kInputSize = "input_size";
606 | static constexpr auto kWeightSize = "weight_size";
607 | static constexpr auto kGroups = "groups";
608 | static constexpr auto kPadding = "padding";
609 | static constexpr auto kStride = "stride";
610 | static constexpr auto kDilation = "dilation";
611 | static constexpr auto kMatSize = "mat_size";
612 | static constexpr auto kMat1Size = "mat1_size";
613 | static constexpr auto kMat2Size = "mat2_size";
614 | 
615 | static std::vector<c10::IntArrayRef> getInputSizes(
616 |     const std::string& op_name,
617 |     size_t min_size,
618 |     c10::ArrayRef<const c10::IValue> inputs,
619 |     const c10::ArrayRef<int>& should_be_tensor) {
620 |   std::stringstream ss;
621 |   if (inputs.size() < min_size) {
622 |     ss << "Failed to save extra arguments for flops computation of op "
623 |        << op_name << ", min size: " << min_size
624 |        << ", actual size: " << inputs.size();
625 |     TORCH_WARN(ss.str());
626 |     return {};
627 |   }
628 |   std::vector<c10::IntArrayRef> inputSizes = {};
629 |   for (auto index : should_be_tensor) {
630 |     if (!inputs[index].isTensor()) {
631 |       ss << "Failed to save extra arguments for flops computation of op "
632 |          << op_name << ", input[" << index << "] must be a tensor.";
633 |       TORCH_WARN(ss.str());
634 |       return {};
635 |     }
636 |     at::Tensor t = inputs[index].toTensor();
637 |     if (t.is_nested()) {
638 |       ss << "Failed to save extra arguments for flops computation of op "
639 |          << op_name << " with input[" << index << "] as nested tensor.";
640 |       TORCH_WARN(ss.str());
641 |       return {};
642 |     }
643 |     inputSizes.emplace_back(t.sizes());
644 |   }
645 |   return inputSizes;
646 | }
647 | 
648 | std::unordered_map<std::string, c10::IValue> saveExtraArgs(
649 |     const at::RecordFunction& fn) {
650 |   // for specific types of fn, return the saved extra args for computing flops
651 |   std::unordered_map<std::string, c10::IValue> map;
652 |   auto inputs = fn.inputs();
653 |   std::string fname(fn.name());
654 | 
655 |   if (inputs.empty()) {
656 |     // Input shape is unavailable, return empty map
657 |     return map;
658 |   }
659 | 
660 |   if (fname == kConv2dOp) {
661 |     const auto inputSizes =
662 |         getInputSizes(fname, kConv2dGroups + 1, inputs, {0, 1});
663 |     if (inputSizes.empty()) {
664 |       return map;
665 |     }
666 |     if (inputSizes[1].size() != 4) {
667 |       TORCH_WARN(
668 |           "Failed to compute flops for op aten::conv2d because it requires a 4D kernel tensor.");
669 |       return map;
670 |     }
671 |     map[kInputSize] = at::IValue(inputSizes[0]);
672 |     map[kWeightSize] = at::IValue(inputSizes[1]);
673 |     map[kStride] = inputs[kConv2dStride];
674 |     map[kPadding] = inputs[kConv2dPadding];
675 |     map[kDilation] = inputs[kConv2dDilation];
676 |     map[kGroups] = inputs[kConv2dGroups];
677 |   } else if (fname == kMMOp) {
678 |     const auto inputSizes = getInputSizes(fname, 2, inputs, {0, 1});
679 |     if (inputSizes.empty()) {
680 |       return map;
681 |     }
682 | 
```
- EN: Implements routines such as `getInputSizes`, `saveExtraArgs`, `fname` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `getInputSizes`、`saveExtraArgs`、`fname` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 683-754
```cpp
683 |     map[kMat1Size] = at::IValue(inputSizes[0]);
684 |     map[kMat2Size] = at::IValue(inputSizes[1]);
685 |   } else if (fname == kAddMMOp) {
686 |     const auto inputSizes = getInputSizes(fname, 3, inputs, {0, 1, 2});
687 |     if (inputSizes.empty()) {
688 |       return map;
689 |     }
690 |     // Exact FLOP count depends on scaling factors alpha and beta but
691 |     // just assume these are +=1.
692 |     // (similar to http://www.netlib.org/lapack/lawnspdf/lawn41.pdf,
693 |     // "Operations Count for the BLAS and LAPACK", Table 3, SGEMM)
694 |     map[kMat1Size] = at::IValue(inputSizes[1]);
695 |     map[kMat2Size] = at::IValue(inputSizes[2]);
696 |   } else if (fname == kMulOp) {
697 |     const auto inputSizes = getInputSizes(fname, 1, inputs, {0});
698 |     if (inputSizes.empty()) {
699 |       return map;
700 |     }
701 |     map[kMatSize] = at::IValue(inputSizes[0]);
702 |   } else if (fname == kAddOp) {
703 |     const auto inputSizes = getInputSizes(fname, 1, inputs, {0});
704 |     if (inputSizes.empty()) {
705 |       return map;
706 |     }
707 |     map[kMatSize] = at::IValue(inputSizes[0]);
708 |   } else if (fname == kBMMOp) {
709 |     const auto inputSizes = getInputSizes(fname, 2, inputs, {0, 1});
710 |     if (inputSizes.empty()) {
711 |       return map;
712 |     }
713 | 
714 |     map[kMat1Size] = at::IValue(inputSizes[0]);
715 |     map[kMat2Size] = at::IValue(inputSizes[1]);
716 |   } else if (fname == kBAddBMMOp) {
717 |     const auto inputSizes = getInputSizes(fname, 3, inputs, {0, 1, 2});
718 |     if (inputSizes.empty()) {
719 |       return map;
720 |     }
721 | 
722 |     // Exact FLOP count depends on scaling factors alpha and beta but
723 |     // just assume these are +=1.
724 |     // (similar to http://www.netlib.org/lapack/lawnspdf/lawn41.pdf,
725 |     // "Operations Count for the BLAS and LAPACK", Table 3, SGEMM)
726 |     map[kMat1Size] = at::IValue(inputSizes[1]);
727 |     map[kMat2Size] = at::IValue(inputSizes[2]);
728 |   }
729 | 
730 |   return map;
731 | }
732 | 
733 | uint64_t computeFlops(
734 |     const std::string& op_name,
735 |     const std::unordered_map<std::string, c10::IValue>& extra_args) {
736 |   if (op_name == kConv2dOp) {
737 |     if (extra_args.find(kInputSize) == extra_args.end() ||
738 |         extra_args.find(kWeightSize) == extra_args.end() ||
739 |         extra_args.find(kGroups) == extra_args.end() ||
740 |         extra_args.find(kPadding) == extra_args.end() ||
741 |         extra_args.find(kStride) == extra_args.end() ||
742 |         extra_args.find(kDilation) == extra_args.end()) {
743 |       TORCH_WARN(
744 |           "Calculating flops for aten::conv2d requires groups, padding, stride, dilation, input_size, and weight_size in saved arguments.");
745 |       return 0;
746 |     }
747 |     auto input_sizes_ref = extra_args.at(kInputSize);
748 |     auto kernel_sizes_ref = extra_args.at(kWeightSize);
749 |     auto groups_ref = extra_args.at(kGroups);
750 |     auto padding_ref = extra_args.at(kPadding);
751 |     auto stride_ref = extra_args.at(kStride);
752 |     auto dilation_ref = extra_args.at(kDilation);
753 |     if (!input_sizes_ref.isIntList() || !kernel_sizes_ref.isIntList()) {
754 |       TORCH_WARN(
```
- EN: Implements routines such as `computeFlops` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `computeFlops` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 755-833
```cpp
755 |           "Failed to compute flops for op aten::conv2d because it requires input and weight tensor sizes.");
756 |       return 0;
757 |     }
758 |     if (!padding_ref.isIntList() || !stride_ref.isIntList() ||
759 |         !dilation_ref.isIntList()) {
760 |       TORCH_WARN(
761 |           "Failed to compute flops for op aten::conv2d because it requires padding, stride, and dilation values.");
762 |       return 0;
763 |     }
764 | 
765 |     const auto input_sizes = input_sizes_ref.toDimVector();
766 |     const auto kernel_sizes = kernel_sizes_ref.toDimVector();
767 |     const uint64_t groups = groups_ref.toInt();
768 |     const std::vector<int64_t> padding = padding_ref.toIntVector();
769 |     const std::vector<int64_t> stride = stride_ref.toIntVector();
770 |     const std::vector<int64_t> dilation = dilation_ref.toIntVector();
771 |     if (input_sizes.size() != 4 || kernel_sizes.size() != 4) {
772 |       TORCH_WARN(
773 |           "Failed to compute flops for op aten::conv2d because both input and weight must be size 4.");
774 |       return 0;
775 |     }
776 |     if (!groups) {
777 |       TORCH_WARN(
778 |           "Failed to compute flops for op aten::conv2d because group size must not be 0.");
779 |       return 0;
780 |     }
781 |     if (padding.size() != 2 || dilation.size() != 2) {
782 |       TORCH_WARN(
783 |           "Failed to compute flops for op aten::conv2d because both padding and dilation must be size 2.");
784 |       return 0;
785 |     }
786 |     if (stride.size() != 2 || (stride[0] * stride[1] == 0)) {
787 |       TORCH_WARN(
788 |           "Failed to compute flops for op aten::conv2d because stride must be size 2 and cannot be 0.");
789 |       return 0;
790 |     }
791 |     // format of the input is defined in
792 |     // torch.ao.nn.quantized.functional.conv2d()
793 |     const uint64_t conv2d_multiply_factor = 2;
794 |     auto [minibatch, in_channels, input_h, input_w] = std::make_tuple(
795 |         input_sizes[0], input_sizes[1], input_sizes[2], input_sizes[3]);
796 |     auto [out_channels, _, kernel_h, kernel_w] = std::make_tuple(
797 |         kernel_sizes[0], kernel_sizes[1], kernel_sizes[2], kernel_sizes[3]);
798 |     uint64_t output_h =
799 |         (input_h + 2 * padding[0] - dilation[0] * (kernel_h - 1) - 1) /
800 |             stride[0] +
801 |         1;
802 |     uint64_t output_w =
803 |         (input_w + 2 * padding[1] - dilation[1] * (kernel_w - 1) - 1) /
804 |             stride[1] +
805 |         1;
806 | 
807 |     return conv2d_multiply_factor * minibatch * output_h * output_w * kernel_h *
808 |         kernel_w * in_channels * out_channels / groups;
809 |   } else if (op_name == kMMOp || op_name == kAddMMOp) {
810 |     if (extra_args.find(kMat1Size) == extra_args.end() ||
811 |         extra_args.find(kMat2Size) == extra_args.end()) {
812 |       TORCH_WARN(
813 |           "Calculating flops for ",
814 |           op_name,
815 |           " requires mat1_size and mat2_size in saved arguments.");
816 |       return 0;
817 |     }
818 |     auto mat1_sizes_ref = extra_args.at(kMat1Size);
819 |     auto mat2_sizes_ref = extra_args.at(kMat2Size);
820 |     if (!mat1_sizes_ref.isIntList() || !mat2_sizes_ref.isIntList()) {
821 |       TORCH_WARN(
822 |           "Failed to compute flops for op ",
823 |           op_name,
824 |           " because it requires mat1_size and mat2_size to be IntList.");
825 |       return 0;
826 |     }
827 | 
828 |     const auto mat1_size = mat1_sizes_ref.toDimVector();
829 |     const auto mat2_size = mat2_sizes_ref.toDimVector();
830 |     if (mat1_size.empty()) {
831 |       return 0;
832 |     }
833 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 834-909
```cpp
834 |     int64_t overlap_dim = mat1_size.back();
835 |     if (overlap_dim == 0) {
836 |       return 0;
837 |     }
838 | 
839 |     const uint64_t gemm_multiply_factor = 2;
840 |     uint64_t flops = 1;
841 |     for (int64_t dim : mat1_size) {
842 |       flops *= dim;
843 |     }
844 |     flops /= overlap_dim;
845 |     for (int64_t dim : mat2_size) {
846 |       flops *= dim;
847 |     }
848 |     flops *= gemm_multiply_factor;
849 |     return flops;
850 |   } else if (op_name == kBMMOp || op_name == kBAddBMMOp) {
851 |     if (extra_args.find(kMat1Size) == extra_args.end() ||
852 |         extra_args.find(kMat2Size) == extra_args.end()) {
853 |       TORCH_WARN(
854 |           "Calculating flops for ",
855 |           op_name,
856 |           " requires mat1_size and mat2_size in saved arguments.");
857 |       return 0;
858 |     }
859 |     auto mat1_sizes_ref = extra_args.at(kMat1Size);
860 |     auto mat2_sizes_ref = extra_args.at(kMat2Size);
861 |     if (!mat1_sizes_ref.isIntList() || !mat2_sizes_ref.isIntList()) {
862 |       TORCH_WARN(
863 |           "Failed to compute flops for op ",
864 |           op_name,
865 |           " because it requires mat1_size and mat2_size to be IntList.");
866 |       return 0;
867 |     }
868 | 
869 |     const auto mat1_size = mat1_sizes_ref.toDimVector();
870 |     const auto mat2_size = mat2_sizes_ref.toDimVector();
871 |     if (mat1_size.empty()) {
872 |       return 0;
873 |     }
874 | 
875 |     int64_t batch_size = mat1_size.front();
876 |     if (batch_size == 0) {
877 |       return 0;
878 |     }
879 | 
880 |     int64_t overlap_dim = mat1_size.back();
881 |     if (overlap_dim == 0) {
882 |       return 0;
883 |     }
884 | 
885 |     const uint64_t gemm_multiply_factor = 2;
886 |     uint64_t flops = 1;
887 |     for (int64_t dim : mat1_size) {
888 |       flops *= dim;
889 |     }
890 |     flops /= overlap_dim;
891 |     flops /= batch_size;
892 |     for (int64_t dim : mat2_size) {
893 |       flops *= dim;
894 |     }
895 |     flops *= gemm_multiply_factor;
896 |     return flops;
897 |   } else if (op_name == kMulOp) {
898 |     if (extra_args.find(kMatSize) == extra_args.end()) {
899 |       TORCH_WARN(
900 |           "Calculating flops for aten::mul.Tensor requires mat_size in saved arguments.");
901 |       return 0;
902 |     }
903 |     auto mat_sizes = extra_args.at(kMatSize);
904 |     if (!mat_sizes.isIntList()) {
905 |       TORCH_WARN(
906 |           "Failed to compute flops for op aten::mul because it requires mat_size to be IntList.");
907 |       return 0;
908 |     }
909 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 910-978
```cpp
910 |     const auto mat_size = mat_sizes.toDimVector();
911 |     uint64_t flops = 1;
912 |     for (int64_t dim : mat_size) {
913 |       flops *= dim;
914 |     }
915 |     return flops;
916 |   } else if (op_name == kAddOp) {
917 |     if (extra_args.find(kMatSize) == extra_args.end()) {
918 |       TORCH_WARN(
919 |           "Calculating flops for aten::add.Tensor requires mat_size in saved arguments.");
920 |       return 0;
921 |     }
922 |     auto mat_sizes = extra_args.at(kMatSize);
923 |     if (!mat_sizes.isIntList()) {
924 |       TORCH_WARN(
925 |           "Failed to compute flops for op aten::add because it requires mat_size to be IntList.");
926 |       return 0;
927 |     }
928 | 
929 |     const auto mat_size = mat_sizes.toDimVector();
930 |     uint64_t flops = 1;
931 |     for (int64_t dim : mat_size) {
932 |       flops *= dim;
933 |     }
934 |     return flops;
935 |   }
936 |   return 0;
937 | }
938 | 
939 | // A function that takes an IValue
940 | // and returns a conventional string representation of the IValue
941 | // Currently it returns int representation of the last 20 bits of the address
942 | // value
943 | int getTensorStartHint(const at::Tensor& t) {
944 |   const auto tensor_impl = t.unsafeGetTensorImpl();
945 |   uintptr_t storage_addr = 0;
946 |   storage_addr = reinterpret_cast<uintptr_t>(tensor_impl->storage().data());
947 |   int last_bits = static_cast<int>(storage_addr & 0xFFFFF);
948 |   return last_bits;
949 | }
950 | 
951 | bool checkFunctionOutputsForLogging(const at::RecordFunction& fn) {
952 |   const auto& outputs = fn.outputs();
953 |   auto num_outputs = fn.num_outputs();
954 |   VLOG(2) << "outputs: " << num_outputs << ' ' << outputs.size() << '\n';
955 |   // We have two cases: for unboxed kernel, we have num_outputs ==
956 |   // outputs.size() for boxed kernel using stack, there could be more elements
957 |   // on the stack from previous ops.
958 |   // TORCH_INTERNAL_ASSERT(num_outputs <= outputs.size());
959 |   if (num_outputs > outputs.size()) {
960 |     return false;
961 |   }
962 |   return true;
963 | }
964 | 
965 | bool checkFunctionInputsForLogging(const at::RecordFunction& fn) {
966 |   auto num_inputs = fn.num_inputs();
967 |   const auto inputs = fn.inputs();
968 |   VLOG(2) << "inputs: " << num_inputs << ' ' << inputs.size() << '\n';
969 |   // We have two cases: for unboxed kernel, we have num_inputs ==
970 |   // inputs.size() for boxed kernel using stack, there could be more elements
971 |   // on the stack from previous ops.
972 |   // TORCH_INTERNAL_ASSERT(num_inputs <= inputs.size());
973 |   if (num_inputs > inputs.size()) {
974 |     return false;
975 |   }
976 |   return true;
977 | }
978 | } // namespace torch::profiler::impl
```
- EN: Implements routines such as `getTensorStartHint`, `checkFunctionOutputsForLogging`, `checkFunctionInputsForLogging` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `getTensorStartHint`、`checkFunctionOutputsForLogging`、`checkFunctionInputsForLogging` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `setSoftAssertRaises`, `softAssertRaises`, `logSoftAssert`, `getNvtxStr`, `prepareCallstack`, `callstackStr`, `stacksToStr`, `flattenList`.
  - CN: `setSoftAssertRaises`、`softAssertRaises`、`logSoftAssert`、`getNvtxStr`、`prepareCallstack`、`callstackStr`、`stacksToStr`、`flattenList`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/util.h>`, `<c10/util/ArrayRef.h>`, `<c10/util/irange.h>`, `<c10/util/hash.h>`, `<torch/csrc/distributed/c10d/ParamCommsUtils.hpp>`
- External includes / 外部头文件: `<fmt/format.h>`, `<fmt/ranges.h>`, `<libkineto.h>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
