# profiler_python.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/profiler_python.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements autograd profiler integration used to observe operator execution and runtime behavior.
- 目的 (CN): 实现自动求导性能分析集成，用于观测算子执行与运行时行为。
- Lines: 1639
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

```cpp
 1: #include <torch/csrc/autograd/profiler_python.h>
 2: 
 3: #include <atomic>
 4: #include <cstdint>
 5: #include <deque>
 6: #include <limits>
 7: #include <memory>
 8: #include <queue>
 9: #include <string>
10: #include <unordered_map>
11: #include <utility>
12: #include <vector>
13: 
14: #include <Python.h>
15: #include <frameobject.h>
16: 
17: #include <ATen/core/TensorBase.h>
18: #include <c10/macros/Macros.h>
19: #include <c10/util/ApproximateClock.h>
20: #include <c10/util/Exception.h>
21: #include <c10/util/Logging.h>
22: #include <c10/util/ScopeExit.h>
23: #include <c10/util/Semaphore.h>
24: #include <c10/util/flat_hash_map.h>
25: #include <c10/util/irange.h>
26: #include <torch/csrc/autograd/python_variable.h>
27: #include <torch/csrc/profiler/collection.h>
28: #include <torch/csrc/profiler/containers.h>
29: #include <torch/csrc/profiler/orchestration/python_tracer.h>
30: #include <torch/csrc/profiler/util.h>
31: #include <torch/csrc/utils/pybind.h>
32: #include <torch/csrc/utils/python_compat.h>
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/profiler_python.h`, `atomic`, `cstdint`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/profiler_python.h`, `atomic`, `cstdint`，为后续实现建立所需的头文件基础。
### Lines 33-64

```cpp
33: #include <torch/csrc/utils/python_numbers.h>
34: #include <torch/csrc/utils/python_strings.h>
35: #include <optional>
36: 
37: namespace py = pybind11;
38: 
39: namespace torch::profiler::impl {
40: namespace {
41: enum CallType { PyCall = 0, PyModuleCall, PyCCall, PyOptimizerCall };
42: static constexpr size_t CallTypeSize = 4;
43: using no_ephemeral_t = std::tuple<>;
44: static constexpr uint64_t NoTID = std::numeric_limits<uint64_t>::max();
45: 
46: // ============================================================================
47: // == Miscellaneous structs and utils =========================================
48: // ============================================================================
49: struct CodeLocation {
50:   CodeLocation() = default;
51:   explicit CodeLocation(PyFrameObject* frame)
52:       : line_number_{PyFrame_GetLineNumber(frame)} {
53:     auto code = THPCodeObjectPtr(PyFrame_GetCode(frame));
54:     filename_ = THPUtils_unpackStringView(code->co_filename).data();
55:     name_ = THPUtils_unpackStringView(code->co_name).data();
56:   }
57: 
58:   bool operator==(const CodeLocation& other) const {
59:     return filename_ == other.filename_ && name_ == other.name_ &&
60:         line_number_ == other.line_number_;
61:   }
62: 
63:   const char* filename_{nullptr};
64:   const char* name_{nullptr};
```

- EN: These lines pull in dependencies such as `torch/csrc/utils/python_numbers.h`, `torch/csrc/utils/python_strings.h`, `optional`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. This range declares or shapes types such as `CallType`, `CodeLocation`. The main execution path in this span is carried by `max`, `CodeLocation`, `THPCodeObjectPtr`.
- CN: 这些行引入了依赖，例如 `torch/csrc/utils/python_numbers.h`, `torch/csrc/utils/python_strings.h`, `optional`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段声明或塑造了 ``CallType`, `CodeLocation`` 等类型。 这一段的主要执行路径由 `max`, `CodeLocation`, `THPCodeObjectPtr` 等函数/方法承载。
### Lines 65-96

```cpp
65:   int line_number_{0};
66: };
67: 
68: template <CallType C>
69: PyCodeObject* getCode();
70: 
71: template <>
72: PyCodeObject* getCode<CallType::PyModuleCall>() {
73:   static auto module_call_code = []() {
74:     pybind11::gil_scoped_acquire gil;
75:     auto res = py::module::import("torch.nn")
76:                    .attr("Module")
77:                    .attr("__call__")
78:                    .attr("__code__")
79:                    .ptr();
80:     TORCH_INTERNAL_ASSERT(PyCode_Check(res));
81:     return (PyCodeObject*)res;
82:   }();
83:   return module_call_code;
84: }
85: 
86: template <>
87: PyCodeObject* getCode<CallType::PyOptimizerCall>() {
88:   static auto optimizer_step_code = []() {
89:     pybind11::gil_scoped_acquire gil;
90:     auto res = py::module::import("torch.optim")
91:                    .attr("Optimizer")
92:                    .attr("_optimizer_step_code")
93:                    .attr("__code__")
94:                    .ptr();
95:     TORCH_INTERNAL_ASSERT(PyCode_Check(res));
96:     return (PyCodeObject*)res;
```

- EN: The main execution path in this span is carried by `getCode`, `import`, `TORCH_INTERNAL_ASSERT`. The logic emits runtime diagnostics or assertions to guard assumptions. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `getCode`, `import`, `TORCH_INTERNAL_ASSERT` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 97-128

```cpp
 97:   }();
 98:   return optimizer_step_code;
 99: }
100: 
101: } // namespace
102: } // namespace torch::profiler::impl
103: 
104: template <>
105: struct std::hash<torch::profiler::impl::CodeLocation> {
106:   size_t operator()(const torch::profiler::impl::CodeLocation& x) {
107:     return c10::get_hash(x.filename_, x.name_, x.line_number_);
108:   }
109: };
110: 
111: namespace torch::profiler::impl {
112: namespace {
113: // ============================================================================
114: // == CallTypeHelper: Tools for generic programming on specializations. =======
115: // ============================================================================
116: template <template <CallType> class ClassT>
117: class CallTypeHelper final {
118:  private:
119:   static_assert(
120:       CallType::PyCall == 0,
121:       "CallTypeHelper uses integer math which depends on a zero start.");
122:   static constexpr size_t End = CallTypeSize;
123: 
124:   template <size_t... I>
125:   static constexpr std::tuple<ClassT<(CallType)I>...> make_tuple_impl(
126:       std::index_sequence<I...>);
127: 
128:   template <size_t C, typename T, typename FunctorT, typename... Args>
```

- EN: They also open, refine, or close the relevant C++ namespace scope. This range declares or shapes types such as `std`, `CallTypeHelper`. The main execution path in this span is carried by `operator`, `get_hash`, `static_assert`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段声明或塑造了 ``std`, `CallTypeHelper`` 等类型。 这一段的主要执行路径由 `operator`, `get_hash`, `static_assert` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 129-160

```cpp
129:   static void map(T& t, FunctorT& f, Args&&... args) {
130:     f(std::get<C>(t), args...);
131:     if constexpr (C + 1 < End) {
132:       map<C + 1>(t, f, std::forward<Args>(args)...);
133:     }
134:   }
135: 
136:  public:
137:   using tuple_type = decltype(make_tuple_impl(std::make_index_sequence<End>{}));
138: 
139:   template <typename FunctorT, typename... Args>
140:   static void map(tuple_type& t, FunctorT& f, Args&&... args) {
141:     map<0>(t, f, std::forward<Args>(args)...);
142:   }
143: };
144: 
145: // ============================================================================
146: // == Event type definitions. =================================================
147: // ============================================================================
148: // When we are tracing a Python program, the general procedure is to record
149: // every time we enter or exit a function and later replay these events during
150: // post processing. Thus, during the profiling phase we want to do the MINIMAL
151: // amount of work to capture all of the information that we need; otherwise we
152: // will distort the profile. (While we don't wish to be terribly inefficient
153: // during post processing, we are willing to do extra fixup work in post if it
154: // reduces overhead in the profiling phase.)
155: //
156: // When the tracer first enters a frame, it constructs a CallKey for that
157: // location. The contents of the key vary by context. For a python function
158: // the key is the (PyCodeObject*, int) pair that defines the bytecode of the
159: // function. For an `nn.Module` the key is a (non-owning) pointer to `self`.
160: // For a bound C function it is a (non-owning) pointer to the bound function.
```

- EN: The main execution path in this span is carried by `map`, `f`, `constexpr`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `map`, `f`, `constexpr` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 161-192

```cpp
161: // A CallKey should be small, inexpensive, and POD.
162: //
163: // We then collect a CallKey<CallType::PyCall> for the calling frame for better
164: // source tracking. This pair is a `Callsite`, and serves as a first level key
165: // during tracing. We lookup the Callsite in a thread local cache which maps
166: // Callsite to a unique integer `TraceKey`. On a cache hit, we simply store the
167: // TraceKey and return. On a cache miss, we use a global value cache to store
168: // whatever fields we need from the two CallKeys, generate a new TraceKey, and
169: // update the local cache.
170: //
171: // During post processing we:
172: //   1) Determine the type represented by a TraceKey by checking which
173: //      sub-cache it appears in the thread local cache.
174: //   2) Look up the pair of CallKeys from the thread local cache.
175: //   3) Look up the expanded values of each CallKey from the global value cache.
176: //
177: // To add a new event type to the cache:
178: //   1) Add an entry to the `CallType` enum.
179: //   2) Add a specialization of Config which defined key_t, ephemeral_t and
180: //      cache_t.
181: //   3) Add a specialization of ValueCache::store and ValueCache::load.
182: //
183: // -------------------------
184: // -- Ephemeral arguments --
185: // -------------------------
186: // The value cache mechanism assumes that `key_t` is enough to specify the
187: // correct value. However it may not be possible to materialize a value using
188: // only an instance of `key_t`. As a result, the cache also accepts "ephemeral"
189: // inputs which can be used to populate the value cache. Ephemeral inputs come
190: // with two caveats:
191: //  1) They are NOT safe to save, and cannot be used after `ValueCache::store`.
192: //  2) They should be used to access data that is not expect to change from
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 193-224

```cpp
193: //     call to call, such as the name of a function.
194: 
195: template <CallType>
196: struct Config;
197: 
198: template <>
199: struct Config<CallType::PyCall> {
200:   using key_t = CodeLocation;
201:   using ephemeral_t = no_ephemeral_t;
202:   using cache_t = ska::flat_hash_map<key_t, PyFrameState>;
203:   static constexpr EventType event_type = EventType::PyCall;
204: };
205: 
206: template <typename Key, typename Cls, typename ParameterInfo>
207: struct ExtendedPyCallConfig {
208:   using key_t = Key;
209:   using cls_t = Cls;
210:   using ephemeral_t = PyFrameObject*;
211: 
212:   struct ClsAndParameters {
213:     cls_t cls_;
214:     std::vector<ParameterInfo> parameters_;
215:   };
216: 
217:   struct Cache {
218:     // `nn.Module.forward` or `optim.Optimizer._optimizer_step_code`
219:     std::optional<CodeLocation> location_;
220:     ska::flat_hash_map<key_t, ClsAndParameters> cls_and_parameters_;
221:     ska::flat_hash_map<cls_t, at::StringView> cls_names_;
222:   };
223:   using cache_t = Cache;
224: 
```

- EN: This range declares or shapes types such as `Config`, `ExtendedPyCallConfig`, `ClsAndParameters`.
- CN: 这一段声明或塑造了 ``Config`, `ExtendedPyCallConfig`, `ClsAndParameters`` 等类型。
### Lines 225-256

```cpp
225:   static constexpr EventType event_type = EventType::PyCall;
226: };
227: 
228: template <>
229: struct Config<CallType::PyModuleCall> : ExtendedPyCallConfig<
230:                                             PyModuleSelf,
231:                                             PyModuleCls,
232:                                             NNModuleInfo::ParameterInfo> {};
233: 
234: template <>
235: struct Config<CallType::PyOptimizerCall> : ExtendedPyCallConfig<
236:                                                PyOptimizerSelf,
237:                                                PyOptimizerCls,
238:                                                OptimizerInfo::ParameterInfo> {};
239: 
240: template <>
241: struct Config<CallType::PyCCall> {
242:   using key_t = PyMethod;
243:   using ephemeral_t = PyObject*;
244:   using cache_t = ska::flat_hash_map<key_t, at::StringView>;
245:   static constexpr EventType event_type = EventType::PyCCall;
246: };
247: 
248: // ============================================================================
249: // == Callsite & ValueCache: Storage during profiling =========================
250: // ============================================================================
251: template <CallType C>
252: class Callsite {
253:  public:
254:   static constexpr CallType call_type = C;
255:   using key_t = typename Config<C>::key_t;
256: 
```

- EN: This range declares or shapes types such as `Config`, `Callsite`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段声明或塑造了 ``Config`, `Callsite`` 等类型。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 257-288

```cpp
257:   static_assert(
258:       std::is_trivially_copyable_v<key_t>,
259:       "Key should be trivial, as it is passed by value.");
260: 
261:   template <typename U>
262:   Callsite(U value, PyFrameObject* f_back) : value_(value), caller_(f_back) {}
263: 
264:   bool operator==(const Callsite<C>& other) const {
265:     return value_ == other.value_ && caller_ == other.caller_;
266:   }
267: 
268:   key_t value_;
269:   Config<CallType::PyCall>::key_t caller_;
270: };
271: 
272: // ============================================================================
273: // == Type specific store and load implementations. ===========================
274: // ============================================================================
275: using PyCallKey = Config<CallType::PyCall>::key_t;
276: using PyModuleCallKey = Config<CallType::PyModuleCall>::key_t;
277: using PyCCallKey = Config<CallType::PyCCall>::key_t;
278: using PyOptimizerCallKey = Config<CallType::PyOptimizerCall>::key_t;
279: 
280: class ValueCache {
281:  public:
282:   ValueCache() = default;
283:   ValueCache(const ValueCache&) = delete;
284:   ValueCache& operator==(const ValueCache&) = delete;
285:   ValueCache(ValueCache&&) = default;
286:   ValueCache& operator==(ValueCache&&) = delete;
287:   ~ValueCache() = default;
288: 
```

- EN: This range declares or shapes types such as `ValueCache`. The main execution path in this span is carried by `static_assert`, `Callsite`, `ValueCache`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段声明或塑造了 ``ValueCache`` 等类型。 这一段的主要执行路径由 `static_assert`, `Callsite`, `ValueCache` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 289-320

```cpp
289:   template <CallType C>
290:   void store(const typename Config<C>::key_t&, typename Config<C>::ephemeral_t);
291: 
292:   template <CallType C>
293:   auto load(const Callsite<C>& callsite, size_t python_tid) const {
294:     auto caller = load<CallType::PyCall>(callsite.caller_);
295:     TORCH_INTERNAL_ASSERT(!caller.module_info_.has_value());
296:     return ExtraFields<Config<C>::event_type>{
297:         /*end_time_ns=*/std::numeric_limits<c10::time_t>::min(),
298:         python_tid,
299:         caller.frame_state_,
300:         load<C>(callsite.value_)};
301:   }
302: 
303:   std::optional<TensorMetadata> recordIfTensor(py::handle p);
304:   std::vector<std::pair<std::string, TensorMetadata>> unpackTensorMap(
305:       const py::dict& tensor_map);
306:   void trimPrefixes();
307: 
308:  private:
309:   template <CallType C>
310:   typename ExtraFields<Config<C>::event_type>::args_t load(
311:       const typename Config<C>::key_t&) const;
312: 
313:   template <CallType C>
314:   using State = typename Config<C>::cache_t;
315: 
316:   CallTypeHelper<State>::tuple_type state_;
317: };
318: 
319: template <CallType C>
320: typename Config<C>::cls_t set_class(
```

- EN: The main execution path in this span is carried by `store`, `load`, `TORCH_INTERNAL_ASSERT`. The logic emits runtime diagnostics or assertions to guard assumptions. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `store`, `load`, `TORCH_INTERNAL_ASSERT` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-352

```cpp
321:     ValueCache* value_cache,
322:     typename Config<C>::cache_t& cache,
323:     const typename Config<C>::key_t& key,
324:     const typename Config<C>::ephemeral_t& frame) {
325:   if (C10_UNLIKELY(!cache.location_.has_value())) {
326:     auto code = THPCodeObjectPtr(PyFrame_GetCode(frame));
327:     TORCH_INTERNAL_ASSERT(code.get() == getCode<C>());
328:     cache.location_ = PyCallKey(frame);
329:     value_cache->store<CallType::PyCall>(*cache.location_, no_ephemeral_t());
330:   }
331: 
332:   auto cls_handle = py::handle((PyObject*)key).attr("__class__");
333:   auto cls = typename Config<C>::cls_t(cls_handle.ptr());
334:   if (cache.cls_names_.find(cls) == cache.cls_names_.end()) {
335:     cache.cls_names_[cls] =
336:         at::StringView(py::str(cls_handle.attr("__name__")));
337:   }
338:   return cls;
339: }
340: 
341: TensorMetadata toTensorMetadata(PyObject* self) {
342:   TORCH_INTERNAL_ASSERT(THPVariable_CheckExact(self));
343:   const auto& t = THPVariable_Unpack(self);
344:   RawTensorMetadata m{t};
345:   return TensorMetadata{
346:       m,
347:       t.sizes().vec(),
348:       m.layout_ == at::kStrided ? t.strides().vec() : std::vector<int64_t>()};
349: }
350: 
351: std::optional<TensorMetadata> ValueCache::recordIfTensor(py::handle p) {
352:   return THPVariable_CheckExact(p.ptr())
```

- EN: The main execution path in this span is carried by `THPCodeObjectPtr`, `TORCH_INTERNAL_ASSERT`, `PyCallKey`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPCodeObjectPtr`, `TORCH_INTERNAL_ASSERT`, `PyCallKey` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 353-384

```cpp
353:       ? std::optional<TensorMetadata>{toTensorMetadata(p.ptr())}
354:       : std::nullopt;
355: }
356: 
357: std::vector<std::pair<std::string, TensorMetadata>> ValueCache::unpackTensorMap(
358:     const py::dict& tensor_map) {
359:   std::vector<std::pair<std::string, TensorMetadata>> out;
360:   for (auto& it : tensor_map) {
361:     auto* value = it.second.ptr();
362:     if (py::isinstance<py::str>(it.first) && THPVariable_CheckExact(value)) {
363:       out.emplace_back(
364:           py::cast<std::string>(it.first), toTensorMetadata(value));
365:     }
366:   }
367:   return out;
368: }
369: 
370: template <>
371: void ValueCache::store<CallType::PyCall>(
372:     const PyCallKey& key,
373:     no_ephemeral_t /*unused*/) {
374:   auto& locations = std::get<CallType::PyCall>(state_);
375:   if (C10_UNLIKELY(locations.find(key) == locations.end())) {
376:     locations[key] = {
377:         key.line_number_,
378:         at::StringView(key.filename_),
379:         at::StringView(key.name_)};
380:   }
381: }
382: 
383: template <>
384: ExtraFields<EventType::PyCall>::args_t ValueCache::load<CallType::PyCall>(
```

- EN: The main execution path in this span is carried by `unpackTensorMap`, `StringView`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `unpackTensorMap`, `StringView` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 385-416

```cpp
385:     const PyCallKey& key) const {
386:   return {std::get<CallType::PyCall>(state_).at(key), std::nullopt};
387: }
388: 
389: template <>
390: void ValueCache::store<CallType::PyModuleCall>(
391:     const PyModuleCallKey& key,
392:     Config<CallType::PyModuleCall>::ephemeral_t frame) {
393:   auto& cache = std::get<CallType::PyModuleCall>(state_);
394:   if (C10_UNLIKELY(
395:           cache.cls_and_parameters_.find(key) ==
396:           cache.cls_and_parameters_.end())) {
397:     auto cls = set_class<CallType::PyModuleCall>(this, cache, key, frame);
398: 
399:     py::dict params = py::handle((PyObject*)key).attr("_parameters");
400:     std::vector<NNModuleInfo::ParameterInfo> params_;
401:     for (auto& it : params) {
402:       auto* p = it.second.ptr();
403:       if (py::isinstance<py::str>(it.first) && THPVariable_CheckExact(p)) {
404:         params_.push_back(
405:             {it.first.cast<std::string>(),
406:              toTensorMetadata(p),
407:              recordIfTensor(py::getattr(it.second, "grad", py::none()))});
408:       }
409:     }
410:     cache.cls_and_parameters_[key] = {cls, std::move(params_)};
411:   }
412: }
413: 
414: template <>
415: ExtraFields<EventType::PyCall>::args_t ValueCache::load<CallType::PyModuleCall>(
416:     const PyModuleCallKey& key) const {
```

- EN: The main execution path in this span is carried by `handle`, `toTensorMetadata`, `recordIfTensor`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle`, `toTensorMetadata`, `recordIfTensor` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 417-448

```cpp
417:   auto& cache = std::get<CallType::PyModuleCall>(state_);
418:   TORCH_INTERNAL_ASSERT(cache.location_.has_value());
419:   const auto& cls_and_parameters = cache.cls_and_parameters_.at(key);
420:   const auto& cls = cls_and_parameters.cls_;
421:   NNModuleInfo info{
422:       key, cls, cache.cls_names_.at(cls), cls_and_parameters.parameters_};
423:   return {
424:       /*frame_state_=*/std::get<CallType::PyCall>(state_).at(*cache.location_),
425:       /*module_info_=*/std::move(info),
426:       /*optimizer_info_=*/std::nullopt};
427: }
428: 
429: template <>
430: void ValueCache::store<CallType::PyOptimizerCall>(
431:     const PyOptimizerCallKey& key,
432:     Config<CallType::PyOptimizerCall>::ephemeral_t frame) {
433:   auto& cache = std::get<CallType::PyOptimizerCall>(state_);
434:   if (C10_UNLIKELY(
435:           cache.cls_and_parameters_.find(key) ==
436:           cache.cls_and_parameters_.end())) {
437:     auto cls = set_class<CallType::PyOptimizerCall>(this, cache, key, frame);
438:     const py::handle self{(PyObject*)key};
439:     std::vector<OptimizerInfo::ParameterInfo> params;
440: 
441:     for (const auto& i : (py::list)self.attr("param_groups")) {
442:       for (auto& param : py::cast<py::dict>(i).attr("get")("params")) {
443:         if (THPVariable_CheckExact(param.ptr())) {
444:           // While `self.state` is permitted to store data in an arbitrary way,
445:           // all generic optimizers (SGD, Adam, etc) use param as the key since
446:           // the state in question is tied to particular parameters. We can
447:           // relax this assumption if the need arises.
448:           params.push_back(
```

- EN: The main execution path in this span is carried by `TORCH_INTERNAL_ASSERT`, `move`, `optimizers`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_INTERNAL_ASSERT`, `move`, `optimizers` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 449-480

```cpp
449:               {toTensorMetadata(param.ptr()),
450:                recordIfTensor(py::getattr(param, "grad", py::none())),
451:                unpackTensorMap(py::cast<py::dict>(self.attr("state"))
452:                                    .attr("get")(param, py::dict()))});
453:         }
454:       }
455:     }
456: 
457:     cache.cls_and_parameters_[key] = {cls, std::move(params)};
458:   }
459: }
460: 
461: template <>
462: ExtraFields<EventType::PyCall>::args_t ValueCache::load<
463:     CallType::PyOptimizerCall>(const PyOptimizerCallKey& key) const {
464:   auto& cache = std::get<CallType::PyOptimizerCall>(state_);
465:   const auto& cls_and_parameters = cache.cls_and_parameters_.at(key);
466:   auto cls = cls_and_parameters.cls_;
467:   OptimizerInfo info{
468:       key, cls, cache.cls_names_.at(cls), cls_and_parameters.parameters_};
469:   return {
470:       /*frame_state_=*/std::get<CallType::PyCall>(state_).at(
471:           // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
472:           cache.location_.value()),
473:       /*module_info_=*/std::nullopt,
474:       /*optimizer_info_=*/std::move(info)};
475: }
476: 
477: template <>
478: void ValueCache::store<CallType::PyCCall>(
479:     const PyCCallKey& key,
480:     Config<CallType::PyCCall>::ephemeral_t arg) {
```

- EN: The main execution path in this span is carried by `recordIfTensor`, `unpackTensorMap`, `move`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `recordIfTensor`, `unpackTensorMap`, `move` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-512

```cpp
481:   auto& names = std::get<CallType::PyCCall>(state_);
482:   if (C10_UNLIKELY(names.find(key) == names.end())) {
483:     names[key] = at::StringView(py::repr(arg));
484:   }
485: }
486: 
487: template <>
488: ExtraFields<EventType::PyCCall>::args_t ValueCache::load<CallType::PyCCall>(
489:     const PyCCallKey& key) const {
490:   return std::get<CallType::PyCCall>(state_).at(key);
491: }
492: 
493: // TODO: Use re2.
494: void ValueCache::trimPrefixes() {
495:   static const auto prefixes = []() {
496:     pybind11::gil_scoped_acquire gil;
497:     return py::module::import("torch.profiler.python_tracer")
498:         .attr("_prefix_regex")()
499:         .cast<std::vector<std::string>>();
500:   }();
501: 
502:   for (auto& it : std::get<CallType::PyCall>(state_)) {
503:     std::string filename = it.second.filename_.str();
504:     for (const auto& p : prefixes) {
505:       if (filename.compare(0, p.size(), p) == 0) {
506:         filename.erase(0, p.size());
507:         it.second.filename_ = at::StringView(filename);
508:         break;
509:       }
510:     }
511:   }
512: }
```

- EN: The main execution path in this span is carried by `StringView`, `trimPrefixes`, `import`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `StringView`, `trimPrefixes`, `import` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 513-544

```cpp
513: 
514: // ============================================================================
515: // == TraceKey cache ==========================================================
516: // ============================================================================
517: using python_tracer::TraceKey;
518: 
519: TraceKey nextKey() {
520:   static std::atomic<uint64_t> key{0};
521:   return TraceKey{++key};
522: }
523: 
524: template <CallType C>
525: struct TraceKeyCacheState {
526:   struct Hash {
527:     size_t operator()(const Callsite<C>& key) {
528:       return c10::get_hash(key.value_, key.caller_);
529:     }
530:   };
531: 
532:   TraceKey intern(
533:       Callsite<C> callsite,
534:       typename Config<C>::ephemeral_t ephemeral,
535:       ValueCache& value_cache) {
536:     auto it = state_.find(callsite);
537:     if (C10_UNLIKELY(it == state_.end())) {
538:       value_cache.store<C>(callsite.value_, ephemeral);
539:       value_cache.store<CallType::PyCall>(callsite.caller_, no_ephemeral_t());
540:       it = state_.insert({callsite, nextKey()}).first;
541:     }
542:     return it->second;
543:   }
544: 
```

- EN: This range declares or shapes types such as `TraceKeyCacheState`, `Hash`. The main execution path in this span is carried by `nextKey`, `operator`, `get_hash`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段声明或塑造了 ``TraceKeyCacheState`, `Hash`` 等类型。 这一段的主要执行路径由 `nextKey`, `operator`, `get_hash` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 545-576

```cpp
545:   auto lookup(Callsite<C>& callsite, ValueCache& value_cache) const {
546:     return std::make_pair(
547:         value_cache.load<C>(callsite.value_),
548:         value_cache.load<CallType::PyCall>(callsite.caller_));
549:   }
550: 
551:   ska::flat_hash_map<Callsite<C>, TraceKey, Hash> state_;
552: };
553: 
554: // ============================================================================
555: // == Core CPython data types =================================================
556: // ============================================================================
557: // PyObject passed as the second argument when enabling tracing via
558: // `PyEval_SetProfile`. A single shared instance is used for all threads;
559: // the callback resolves per-thread state via PyThreadState_Get().
560: class PythonTracer;
561: struct TraceContext {
562:   PyObject_HEAD
563:   PythonTracer* tracer_;
564: };
565: 
566: // CPython boilerplate to define `TraceContext` as a proper python object.
567: static PyTypeObject TraceContextType = {
568:     PyVarObject_HEAD_INIT(nullptr, 0)
569:     "TraceContext", /* tp_name */
570:     sizeof(TraceContext), /* tp_basicsize */
571:     0, /* tp_itemsize */
572:     nullptr, /* tp_dealloc */
573:     0,
574:     /* tp_vectorcall_offset */
575:     nullptr, /* tp_getattr */
576:     nullptr, /* tp_setattr */
```

- EN: This range declares or shapes types such as `PythonTracer`, `TraceContext`. The main execution path in this span is carried by `lookup`, `make_pair`, `PyThreadState_Get`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段声明或塑造了 ``PythonTracer`, `TraceContext`` 等类型。 这一段的主要执行路径由 `lookup`, `make_pair`, `PyThreadState_Get` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 577-608

```cpp
577:     nullptr, /* tp_reserved */
578:     nullptr, /* tp_repr */
579:     nullptr, /* tp_as_number */
580:     nullptr, /* tp_as_sequence */
581:     nullptr, /* tp_as_mapping */
582:     nullptr, /* tp_hash  */
583:     nullptr, /* tp_call */
584:     nullptr, /* tp_str */
585:     nullptr, /* tp_getattro */
586:     nullptr, /* tp_setattro */
587:     nullptr, /* tp_as_buffer */
588:     Py_TPFLAGS_DEFAULT, /* tp_flags */
589:     "Python tracer TLS", /* tp_doc */
590:     nullptr, /* tp_traverse */
591:     nullptr, /* tp_clear */
592:     nullptr, /* tp_richcompare */
593:     0, /* tp_weaklistoffset */
594:     nullptr, /* tp_iter */
595:     nullptr, /* tp_iternext */
596:     nullptr, /* tp_methods */
597:     nullptr, /* tp_members */
598:     nullptr, /* tp_getset */
599:     nullptr, /* tp_base */
600:     nullptr, /* tp_dict */
601:     nullptr, /* tp_descr_get */
602:     nullptr, /* tp_descr_set */
603:     0, /* tp_dictoffset */
604:     nullptr, /* tp_init */
605:     nullptr, /* tp_alloc */
606:     PyType_GenericNew, /* tp_new */
607:     nullptr /* tp_free */
608: };
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 609-640

```cpp
609: 
610: #if IS_PYTHON_3_14_PLUS
611: extern "C" void _PyEval_StopTheWorld(PyInterpreterState*);
612: extern "C" void _PyEval_StartTheWorld(PyInterpreterState*);
613: 
614: class StopTheWorldGuard {
615:  public:
616:   explicit StopTheWorldGuard(PyInterpreterState* interp) : interp_(interp) {
617:     _PyEval_StopTheWorld(interp_);
618:   }
619:   ~StopTheWorldGuard() {
620:     _PyEval_StartTheWorld(interp_);
621:   }
622:   StopTheWorldGuard(const StopTheWorldGuard&) = delete;
623:   StopTheWorldGuard& operator=(const StopTheWorldGuard&) = delete;
624: 
625:  private:
626:   PyInterpreterState* interp_;
627: };
628: #else
629: class StopTheWorldGuard {
630:  public:
631:   explicit StopTheWorldGuard(PyInterpreterState*) {}
632:   StopTheWorldGuard(const StopTheWorldGuard&) = delete;
633:   StopTheWorldGuard& operator=(const StopTheWorldGuard&) = delete;
634: };
635: #endif
636: 
637: // ============================================================================
638: // == Thread local cache ======================================================
639: // ============================================================================
640: struct ThreadLocalResults {
```

- EN: This range declares or shapes types such as `StopTheWorldGuard`, `ThreadLocalResults`. The main execution path in this span is carried by `_PyEval_StopTheWorld`, `_PyEval_StartTheWorld`, `StopTheWorldGuard`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段声明或塑造了 ``StopTheWorldGuard`, `ThreadLocalResults`` 等类型。 这一段的主要执行路径由 `_PyEval_StopTheWorld`, `_PyEval_StartTheWorld`, `StopTheWorldGuard` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 641-672

```cpp
641:   ThreadLocalResults(PythonTracer* active_tracer)
642:       : active_tracer_{active_tracer} {}
643: 
644:   ThreadLocalResults() = delete;
645:   ThreadLocalResults(const ThreadLocalResults&) = delete;
646:   ThreadLocalResults(ThreadLocalResults&&) = delete;
647:   ThreadLocalResults& operator=(const ThreadLocalResults&) = delete;
648:   ThreadLocalResults& operator=(const ThreadLocalResults&&) = delete;
649: 
650:   template <CallType C, EventType E, typename Ephemeral, typename... Args>
651:   TraceKey intern(Ephemeral ephemeral, Args... args) {
652:     static_assert(
653:         Config<C>::event_type == E,
654:         "ThreadLocalResults.intern called from the wrong typed context.");
655:     auto callsite = Callsite<C>(std::forward<Args>(args)...);
656:     return std::get<C>(trace_keys_).intern(callsite, ephemeral, value_cache_);
657:   }
658: 
659:   static constexpr size_t BLOCK_SIZE = 1024;
660: 
661:   ValueCache value_cache_;
662:   PythonTracer* active_tracer_;
663:   CallTypeHelper<TraceKeyCacheState>::tuple_type trace_keys_;
664:   AppendOnlyList<c10::approx_time_t, BLOCK_SIZE> exit_times_;
665:   AppendOnlyList<c10::approx_time_t, BLOCK_SIZE> c_exit_times_;
666: 
667:   int active_frames_{0};
668:   int remaining_start_frames_{0};
669: 
670:   // Guards against teardown racing with in-flight callbacks.
671:   // pyProfileFn acquires this on entry and releases on exit.
672:   // PythonTracer::stop() acquires each thread's semaphore after
```

- EN: The main execution path in this span is carried by `ThreadLocalResults`, `intern`, `static_assert`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `ThreadLocalResults`, `intern`, `static_assert` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 673-704

```cpp
673:   // clearing the profiling callback to ensure all callbacks have finished.
674:   c10::Semaphore profile_sem{1};
675: };
676: 
677: // ============================================================================
678: // == Tracing implementation ==================================================
679: // ============================================================================
680: #define IS_PYTHON_3_12 (PY_MAJOR_VERSION == 3 && PY_MINOR_VERSION == 12)
681: #if IS_PYTHON_3_12
682: // forward declarations
683: struct _PyEventHandler;
684: static PyObject* c_call_callback(
685:     _PyEventHandler* self,
686:     PyObject* const* args,
687:     size_t nargsf,
688:     PyObject* kwnames);
689: #endif
690: 
691: class PythonTracer final : public python_tracer::PythonTracerBase {
692:  public:
693:   PythonTracer(torch::profiler::impl::RecordQueue* queue);
694:   // NOLINTNEXTLINE(bugprone-exception-escape)
695:   ~PythonTracer() override;
696: 
697:   static int pyProfileFn(
698:       PyObject* obj,
699:       PyFrameObject* frame,
700:       int what,
701:       PyObject* arg);
702:   void register_gc_callback() override;
703:   void stop() override;
704:   void restart() override;
```

- EN: This range declares or shapes types such as `_PyEventHandler`, `PythonTracer`. The main execution path in this span is carried by `c_call_callback`, `PythonTracer`, `NOLINTNEXTLINE`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段声明或塑造了 ``_PyEventHandler`, `PythonTracer`` 等类型。 这一段的主要执行路径由 `c_call_callback`, `PythonTracer`, `NOLINTNEXTLINE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 705-736

```cpp
705:   std::vector<std::shared_ptr<Result>> getEvents(
706:       std::function<c10::time_t(c10::approx_time_t)> time_converter,
707:       std::vector<python_tracer::CompressedEvent>& enters,
708:       c10::time_t end_time_ns) override;
709: 
710:   struct StartFrame {
711:     TraceKey trace_key_;
712:     c10::approx_time_t start_time{};
713:   };
714: 
715:  private:
716:   void recordPyCall(
717:       ThreadLocalResults& tls,
718:       PyFrameObject* frame,
719:       bool is_startup_frame);
720: 
721:   static PyObject* gc_event_callback(PyObject* self, PyObject* args);
722: 
723:   void recordCCall(
724:       ThreadLocalResults& tls,
725:       PyFrameObject* frame,
726:       PyObject* arg,
727:       bool start_frame = false);
728: 
729:   ThreadLocalResults* findThreadLocalResults(PyThreadState* tstate) const;
730: 
731:   const std::vector<PyThreadState*> interpreterThreads() const;
732:   void setprofileAllThreads(Py_tracefunc func, PyObject* arg) const;
733: 
734:   std::atomic<bool> active_lock_{false};
735:   bool active_{false};
736:   bool gc_callback_registered_{false};
```

- EN: This range declares or shapes types such as `StartFrame`. The main execution path in this span is carried by `getEvents`, `time_t`, `recordPyCall`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段声明或塑造了 ``StartFrame`` 等类型。 这一段的主要执行路径由 `getEvents`, `time_t`, `recordPyCall` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 737-768

```cpp
737: 
738:   torch::profiler::impl::RecordQueue* queue_;
739:   PyInterpreterState* interpreter_{nullptr};
740:   PyCodeObject* module_call_code_;
741:   PyCodeObject* optimizer_hook_;
742:   TraceContext* shared_ctx_{nullptr};
743: 
744:   std::vector<StartFrame> start_frames_;
745:   std::deque<ThreadLocalResults> thread_local_results_;
746:   std::unordered_map<PyThreadState*, ThreadLocalResults*>
747:       thread_local_results_map_;
748: 
749: #if IS_PYTHON_3_12
750:   friend PyObject* c_call_callback(
751:       _PyEventHandler* self,
752:       PyObject* const* args,
753:       size_t nargsf,
754:       PyObject* kwnames);
755: #endif
756: };
757: 
758: #if IS_PYTHON_3_12
759: #define PROFILER_ID 2
760: #define PY_MONITORING_EVENT_CALL 4
761: 
762: static bool should_compensate_c_call_events() {
763:   static const bool result = []() {
764:     const char* version = Py_GetVersion();
765:     const char micro = version[5];
766:     return micro == '0' || (micro <= '4' && version[6] == ' ');
767:   }();
768:   return result;
```

- EN: The main execution path in this span is carried by `c_call_callback`, `should_compensate_c_call_events`, `Py_GetVersion`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `c_call_callback`, `should_compensate_c_call_events`, `Py_GetVersion` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 769-800

```cpp
769: }
770: 
771: struct _PyEventHandler {
772:   PyObject_HEAD
773:   vectorcallfunc vectorcall;
774: };
775: 
776: static PyTypeObject _PyEventHandler_Type = {
777:     PyVarObject_HEAD_INIT(&PyType_Type, 0) /* ob_base */
778:     "torch.profiler.python_tracer_event_handler", /* tp_name */
779:     sizeof(_PyEventHandler), /* tp_basicsize */
780:     0, /* tp_itemsize */
781:     (destructor)PyObject_Free, /* tp_dealloc */
782:     offsetof(_PyEventHandler, vectorcall), /* tp_vectorcall_offset */
783:     nullptr, /* tp_getattr */
784:     nullptr, /* tp_setattr */
785:     nullptr, /* tp_reserved */
786:     nullptr, /* tp_repr */
787:     nullptr, /* tp_as_number */
788:     nullptr, /* tp_as_sequence */
789:     nullptr, /* tp_as_mapping */
790:     nullptr, /* tp_hash */
791:     PyVectorcall_Call, /* tp_call */
792:     nullptr, /* tp_str */
793:     nullptr, /* tp_getattro */
794:     nullptr, /* tp_setattro */
795:     nullptr, /* tp_as_buffer */
796:     Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE | Py_TPFLAGS_HAVE_VECTORCALL |
797:         Py_TPFLAGS_DISALLOW_INSTANTIATION, /* tp_flags */
798: };
799: 
800: static PyObject* c_call_callback(
```

- EN: This range declares or shapes types such as `_PyEventHandler`. The main execution path in this span is carried by `PyVarObject_HEAD_INIT`, `offsetof`, `c_call_callback`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段声明或塑造了 ``_PyEventHandler`` 等类型。 这一段的主要执行路径由 `PyVarObject_HEAD_INIT`, `offsetof`, `c_call_callback` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 801-832

```cpp
801:     _PyEventHandler* self,
802:     PyObject* const* args,
803:     size_t nargsf,
804:     PyObject* kwnames) {
805:   // The logic of this function is based on sys_defile_call_or_return defined
806:   // in https://github.com/python/cpython/blob/v3.12.5/Python/legacy_tracing.c
807: 
808:   PyThreadState* tstate = PyThreadState_GET();
809:   if (tstate->c_profilefunc != PythonTracer::pyProfileFn) {
810:     // We don't care this case if tstate->c_profilefunc is not pyProfileFn,
811:     // just return normally.
812:     Py_RETURN_NONE;
813:   }
814: 
815:   PyObject* callable = args[2];
816:   if (Py_TYPE(callable) == &PyMethod_Type) {
817:     // The call event of a method with c function is missing on 3.12.0-3.12.4.
818:     // See
819:     // https://github.com/python/cpython/commit/257c413cd16ddabcedde413288d0bb93bf872da7
820:     // Other cases have already be handled by the legacy_tracing, so we only
821:     // need to handle this case.
822:     // The exception branches keep the same behavior as CPython.
823:     PyObject* func = PyMethod_GET_FUNCTION(callable);
824:     if (!func) {
825:       return NULL;
826:     }
827:     if (PyCFunction_Check(func)) {
828:       PyFrameObject* frame = PyEval_GetFrame();
829:       if (!frame) {
830:         PyErr_SetString(
831:             PyExc_SystemError, "Missing frame when calling profile function.");
832:         return NULL;
```

- EN: The main execution path in this span is carried by `PyThreadState_GET`, `PyMethod_GET_FUNCTION`, `PyEval_GetFrame`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyThreadState_GET`, `PyMethod_GET_FUNCTION`, `PyEval_GetFrame` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 833-864

```cpp
833:       }
834:       auto* tracer =
835:           reinterpret_cast<TraceContext*>(tstate->c_profileobj)->tracer_;
836:       auto* local_results = tracer->findThreadLocalResults(tstate);
837:       if (local_results) {
838:         Py_INCREF(frame);
839:         local_results->active_tracer_->recordCCall(*local_results, frame, func);
840:         Py_DECREF(frame);
841:       }
842:     }
843:   }
844:   Py_RETURN_NONE;
845: }
846: 
847: static void registerMonitoringCallback() {
848:   if (!should_compensate_c_call_events()) {
849:     return;
850:   }
851: 
852:   auto sys_module = THPObjectPtr(PyImport_ImportModule("sys"));
853:   if (!sys_module) {
854:     TORCH_WARN("Failed to import sys module.");
855:     PyErr_Clear();
856:     return;
857:   }
858:   auto monitoring =
859:       THPObjectPtr(PyObject_GetAttrString(sys_module, "monitoring"));
860:   if (!monitoring) {
861:     TORCH_WARN("Failed to get monitoring from sys module.");
862:     PyErr_Clear();
863:     return;
864:   }
```

- EN: The main execution path in this span is carried by `Py_INCREF`, `Py_DECREF`, `registerMonitoringCallback`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `Py_INCREF`, `Py_DECREF`, `registerMonitoringCallback` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 865-896

```cpp
865:   auto result = THPObjectPtr(PyObject_CallMethod(
866:       monitoring, "use_tool_id", "is", PROFILER_ID, "PyTorch Profiler"));
867:   if (!result) {
868:     TORCH_WARN("Failed to call sys.monitoring.use_tool_id");
869:     PyErr_Clear();
870:     return;
871:   }
872:   auto handler = THPObjectPtr(PyObject_NEW(PyObject, &_PyEventHandler_Type));
873:   if (!handler) {
874:     TORCH_WARN("Failed to create _PyEventHandler object.");
875:     PyErr_Clear();
876:     return;
877:   }
878:   reinterpret_cast<_PyEventHandler*>(handler.get())->vectorcall =
879:       (vectorcallfunc)c_call_callback;
880:   result = THPObjectPtr(PyObject_CallMethod(
881:       monitoring,
882:       "register_callback",
883:       "iiO",
884:       PROFILER_ID,
885:       1 << PY_MONITORING_EVENT_CALL,
886:       handler.get()));
887:   if (!result) {
888:     TORCH_WARN("Failed to call sys.monitoring.register_callback.");
889:     PyErr_Clear();
890:     return;
891:   }
892:   result = THPObjectPtr(PyObject_CallMethod(
893:       monitoring,
894:       "set_events",
895:       "ii",
896:       PROFILER_ID,
```

- EN: The main execution path in this span is carried by `THPObjectPtr`, `TORCH_WARN`, `PyErr_Clear`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPObjectPtr`, `TORCH_WARN`, `PyErr_Clear` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 897-928

```cpp
897:       1 << PY_MONITORING_EVENT_CALL));
898:   if (!result) {
899:     TORCH_WARN("Failed to call sys.monitoring.set_events.");
900:     PyErr_Clear();
901:     return;
902:   }
903: }
904: 
905: static void unregisterMonitoringCallback() {
906:   if (!should_compensate_c_call_events()) {
907:     return;
908:   }
909: 
910:   auto sys_module = THPObjectPtr(PyImport_ImportModule("sys"));
911:   if (!sys_module) {
912:     TORCH_WARN("Failed to import sys module.");
913:     PyErr_Clear();
914:     return;
915:   }
916:   auto monitoring =
917:       THPObjectPtr(PyObject_GetAttrString(sys_module, "monitoring"));
918:   if (!monitoring) {
919:     TORCH_WARN("Failed to get monitoring from sys module.");
920:     PyErr_Clear();
921:     return;
922:   }
923:   auto tool_name = THPObjectPtr(
924:       PyObject_CallMethod(monitoring, "get_tool", "i", PROFILER_ID));
925:   if (!tool_name) {
926:     TORCH_WARN("Failed to call sys.monitoring.use_tool_id");
927:     PyErr_Clear();
928:     return;
```

- EN: The main execution path in this span is carried by `TORCH_WARN`, `PyErr_Clear`, `unregisterMonitoringCallback`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_WARN`, `PyErr_Clear`, `unregisterMonitoringCallback` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 929-960

```cpp
929:   }
930:   if (!THPUtils_checkString(tool_name)) {
931:     return;
932:   }
933:   const char* str = THPUtils_unpackStringView(tool_name).data();
934:   if (strcmp(str, "PyTorch Profiler") != 0) {
935:     return;
936:   }
937:   auto none = THPObjectPtr(Py_None);
938:   Py_INCREF(Py_None);
939:   auto result = THPObjectPtr(PyObject_CallMethod(
940:       monitoring,
941:       "register_callback",
942:       "iiO",
943:       PROFILER_ID,
944:       1 << PY_MONITORING_EVENT_CALL,
945:       none.get()));
946:   if (!result) {
947:     TORCH_WARN("Failed to call sys.monitoring.register_callback.");
948:     PyErr_Clear();
949:     return;
950:   }
951:   result = THPObjectPtr(
952:       PyObject_CallMethod(monitoring, "set_events", "ii", PROFILER_ID, 0));
953:   if (!result) {
954:     TORCH_WARN("Failed to call sys.monitoring.set_events.");
955:     PyErr_Clear();
956:     return;
957:   }
958:   result = THPObjectPtr(
959:       PyObject_CallMethod(monitoring, "free_tool_id", "i", PROFILER_ID));
960:   if (!result) {
```

- EN: The main execution path in this span is carried by `THPUtils_unpackStringView`, `THPObjectPtr`, `Py_INCREF`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPUtils_unpackStringView`, `THPObjectPtr`, `Py_INCREF` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 961-992

```cpp
961:     TORCH_WARN("Failed to call sys.monitoring.free_tool_id.");
962:     PyErr_Clear();
963:     return;
964:   }
965: }
966: #endif
967: 
968: ThreadLocalResults* PythonTracer::findThreadLocalResults(
969:     PyThreadState* tstate) const {
970:   auto it = thread_local_results_map_.find(tstate);
971:   return it != thread_local_results_map_.end() ? it->second : nullptr;
972: }
973: 
974: const std::vector<PyThreadState*> PythonTracer::interpreterThreads() const {
975:   pybind11::gil_scoped_acquire gil;
976:   std::vector<PyThreadState*> out;
977:   if (SOFT_ASSERT(interpreter_)) {
978:     auto* thread_state = PyInterpreterState_ThreadHead(interpreter_);
979:     while (thread_state != nullptr) {
980:       out.push_back(thread_state);
981:       thread_state = PyThreadState_Next(thread_state);
982:     }
983:   }
984:   return out;
985: }
986: 
987: void PythonTracer::setprofileAllThreads(Py_tracefunc func, PyObject* arg)
988:     const {
989: #if IS_PYTHON_3_13_PLUS
990:   PyEval_SetProfileAllThreads(func, arg);
991: #else
992:   for (const auto thread_state : interpreterThreads()) {
```

- EN: The main execution path in this span is carried by `TORCH_WARN`, `PyErr_Clear`, `findThreadLocalResults`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_WARN`, `PyErr_Clear`, `findThreadLocalResults` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 993-1024

```cpp
 993:     if (_PyEval_SetProfile(thread_state, func, arg) < 0) {
 994:       PyErr_WriteUnraisable(nullptr);
 995:     }
 996:   }
 997: #endif
 998: }
 999: 
1000: // we are only registering on main thread while holding GIL so this should be
1001: // safe
1002: static PyObject* py_gc_callback = nullptr;
1003: // The C function to be called by Python's GC
1004: PyObject* PythonTracer::gc_event_callback(PyObject* self, PyObject* args) {
1005:   const char* phase;
1006:   PyObject* info;
1007:   if (!PyArg_ParseTuple(args, "sO", &phase, &info)) {
1008:     return nullptr;
1009:   }
1010:   PythonTracer* instance =
1011:       reinterpret_cast<PythonTracer*>(PyCapsule_GetPointer(self, nullptr));
1012:   if (!instance) {
1013:     PyErr_SetString(PyExc_RuntimeError, "Invalid tracer instance");
1014:     return nullptr;
1015:   }
1016:   instance->queue_->getSubqueue()->emplace_gc_call(
1017:       phase, c10::getApproximateTime());
1018:   Py_RETURN_NONE;
1019: }
1020: 
1021: PythonTracer::PythonTracer(torch::profiler::impl::RecordQueue* queue)
1022:     : queue_(queue),
1023: 
1024:       module_call_code_(getCode<CallType::PyModuleCall>()),
```

- EN: The main execution path in this span is carried by `PyErr_WriteUnraisable`, `gc_event_callback`, `PyErr_SetString`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_WriteUnraisable`, `gc_event_callback`, `PyErr_SetString` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1025-1056

```cpp
1025:       optimizer_hook_(getCode<CallType::PyOptimizerCall>()) {
1026:   TORCH_CHECK(queue_ != nullptr);
1027: 
1028:   bool expected{false};
1029:   active_ = active_lock_.compare_exchange_strong(expected, true);
1030:   if (!active_) {
1031:     TORCH_WARN(
1032:         "There is already an active Python tracer. "
1033:         "Refusing to register profile functions.");
1034:     return;
1035:   }
1036: 
1037: #if defined(Py_GIL_DISABLED) && !defined(IS_PYTHON_3_14_PLUS)
1038:   TORCH_WARN(
1039:       "The PyTorch profiler is not thread-safe on Python 3.13t. "
1040:       "Please use Python 3.14t or later.");
1041: #endif
1042: 
1043:   pybind11::gil_scoped_acquire gil;
1044:   interpreter_ = PyInterpreterState_Get();
1045: 
1046:   // Shared context passed as the profile arg to all threads.
1047:   shared_ctx_ = (TraceContext*)TraceContextType.tp_alloc(&TraceContextType, 0);
1048:   shared_ctx_->tracer_ = this;
1049: 
1050:   // Enable profiling on all threads. setprofileAllThreads handles its own
1051:   // synchronization (stop-the-world on free-threaded builds). The callback
1052:   // returns early because findThreadLocalResults returns nullptr for threads
1053:   // we haven't set up yet.
1054:   // Note: This profile will not compose with other CPython profilers, and
1055:   // cannot be round tripped via `sys.settrace(sys.gettrace())`
1056:   setprofileAllThreads(PythonTracer::pyProfileFn, (PyObject*)shared_ctx_);
```

- EN: The main execution path in this span is carried by `optimizer_hook_`, `TORCH_CHECK`, `TORCH_WARN`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `optimizer_hook_`, `TORCH_CHECK`, `TORCH_WARN` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1057-1088

```cpp
1057: 
1058:   // Capture existing frames on each thread's stack.
1059:   {
1060:     StopTheWorldGuard stw(interpreter_);
1061:     for (const auto thread_state : interpreterThreads()) {
1062:       thread_local_results_.emplace_back(this);
1063:       auto& tls = thread_local_results_.back();
1064:       thread_local_results_map_[thread_state] = &tls;
1065: 
1066:       // When we begin profiling there are already frames on the Python
1067:       // interpreter stack. To ensure a complete trace, we must push calls
1068:       // to all the prior frames onto our event stack. (We stop at depth=128)
1069: 
1070:       // NB: `PyThreadState_GetFrame` returns a strong reference.
1071:       std::vector<THPFrameObjectPtr> current_stack;
1072:       auto frame = PyThreadState_GetFrame(thread_state);
1073: 
1074:       size_t depth = 0; // Make sure we can't infinite loop.
1075:       while (frame != nullptr) {
1076:         current_stack.emplace_back(frame);
1077:         if (++depth == 128) {
1078:           break;
1079:         }
1080: 
1081:         // NB: `PyFrame_GetBack` returns a strong reference.
1082:         frame = PyFrame_GetBack(frame);
1083:       }
1084: 
1085:       for (auto it = current_stack.rbegin(); it != current_stack.rend(); it++) {
1086:         recordPyCall(tls, it->get(), true);
1087:         auto frame_refcount = Py_REFCNT(it->get());
1088: 
```

- EN: The main execution path in this span is carried by `stw`, `PyThreadState_GetFrame`, `PyFrame_GetBack`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `stw`, `PyThreadState_GetFrame`, `PyFrame_GetBack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 1089-1120

```cpp
1089:         // We hold one reference in `current_stack`, and the interpreter holds
1090:         // another.
1091:         TORCH_INTERNAL_ASSERT(frame_refcount >= 2, frame_refcount);
1092:       }
1093: 
1094:       tls.remaining_start_frames_ = tls.active_frames_;
1095:     }
1096:   }
1097: #if IS_PYTHON_3_12
1098:   registerMonitoringCallback();
1099: #endif
1100: }
1101: 
1102: void unregister_gc_callback() {
1103:   PyGILState_STATE gstate = PyGILState_Ensure();
1104:   PyObject* gc_module = PyImport_ImportModule("gc");
1105:   if (!gc_module) {
1106:     PyErr_Print();
1107:     PyGILState_Release(gstate);
1108:     return;
1109:   }
1110:   PyObject* callbacks = PyObject_GetAttrString(gc_module, "callbacks");
1111:   if (!callbacks || !PyList_Check(callbacks)) {
1112:     PyErr_Print();
1113:     Py_XDECREF(gc_module);
1114:     Py_XDECREF(callbacks);
1115:     PyGILState_Release(gstate);
1116:     return;
1117:   }
1118:   Py_ssize_t idx = PySequence_Index(callbacks, py_gc_callback);
1119:   if (idx >= 0) {
1120:     PySequence_DelItem(callbacks, idx);
```

- EN: The main execution path in this span is carried by `TORCH_INTERNAL_ASSERT`, `registerMonitoringCallback`, `unregister_gc_callback`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_INTERNAL_ASSERT`, `registerMonitoringCallback`, `unregister_gc_callback` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1121-1152

```cpp
1121:   } else {
1122:     // Not found, maybe already removed
1123:     PyErr_Clear();
1124:   }
1125:   Py_DECREF(callbacks);
1126:   Py_DECREF(gc_module);
1127:   Py_XDECREF(py_gc_callback);
1128:   py_gc_callback = nullptr;
1129:   PyGILState_Release(gstate);
1130: }
1131: 
1132: void PythonTracer::register_gc_callback() {
1133:   PyGILState_STATE gstate = PyGILState_Ensure();
1134:   PyObject* gc_module = PyImport_ImportModule("gc");
1135:   if (!gc_module) {
1136:     PyErr_Print();
1137:     PyGILState_Release(gstate);
1138:     return;
1139:   }
1140:   PyObject* callbacks = PyObject_GetAttrString(gc_module, "callbacks");
1141:   if (!callbacks || !PyList_Check(callbacks)) {
1142:     PyErr_Print();
1143:     Py_XDECREF(gc_module);
1144:     Py_XDECREF(callbacks);
1145:     PyGILState_Release(gstate);
1146:     return;
1147:   }
1148:   static PyMethodDef method_def = {
1149:       "gc_event_callback",
1150:       (PyCFunction)gc_event_callback,
1151:       METH_VARARGS,
1152:       nullptr};
```

- EN: The main execution path in this span is carried by `PyErr_Clear`, `Py_DECREF`, `Py_XDECREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyErr_Clear`, `Py_DECREF`, `Py_XDECREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1153-1184

```cpp
1153:   PyObject* capsule = PyCapsule_New(this, nullptr, nullptr);
1154:   py_gc_callback = PyCFunction_New(&method_def, capsule);
1155:   Py_DECREF(capsule); // PyCFunction_New increments refcount
1156:   if (PyList_Append(callbacks, py_gc_callback) < 0) {
1157:     PyErr_Print();
1158:   }
1159:   gc_callback_registered_ = true;
1160:   Py_DECREF(callbacks);
1161:   Py_DECREF(gc_module);
1162:   PyGILState_Release(gstate);
1163: }
1164: 
1165: void PythonTracer::stop() {
1166:   pybind11::gil_scoped_acquire gil;
1167:   if (gc_callback_registered_) {
1168:     unregister_gc_callback();
1169:     gc_callback_registered_ = false;
1170:   }
1171:   if (active_) {
1172:     setprofileAllThreads(nullptr, nullptr);
1173: 
1174:     // Wait for any in-flight pyProfileFn callbacks to finish. Threads inside
1175:     // pyProfileFn hold their thread's profile_sem. They may have temporarily
1176:     // released the GIL or parked mid-callback due to a stop-the-world event.
1177:     // Acquiring each semaphore here blocks until those callbacks complete.
1178:     {
1179:       pybind11::gil_scoped_release release;
1180:       for (auto& tls : thread_local_results_) {
1181:         tls.profile_sem.acquire();
1182:         tls.profile_sem.release();
1183:       }
1184:     }
```

- EN: The main execution path in this span is carried by `PyCapsule_New`, `PyCFunction_New`, `Py_DECREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyCapsule_New`, `PyCFunction_New`, `Py_DECREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1185-1216

```cpp
1185: 
1186: #if IS_PYTHON_3_12
1187:     unregisterMonitoringCallback();
1188: #endif
1189: 
1190:     auto lock_returned = active_lock_.compare_exchange_strong(active_, false);
1191:     active_ = false;
1192:     SOFT_ASSERT(lock_returned, "Failed to return python tracer lock.");
1193:   }
1194: }
1195: 
1196: void PythonTracer::restart() {
1197:   pybind11::gil_scoped_acquire gil;
1198:   active_ = active_lock_.compare_exchange_strong(active_, true);
1199:   if (!active_) {
1200:     TORCH_WARN(
1201:         "There is already an active Python tracer. "
1202:         "Refusing to register profile functions.");
1203:     return;
1204:   }
1205:   setprofileAllThreads(PythonTracer::pyProfileFn, (PyObject*)shared_ctx_);
1206: #if IS_PYTHON_3_12
1207:   registerMonitoringCallback();
1208: #endif
1209: }
1210: 
1211: // NOLINTNEXTLINE(bugprone-exception-escape)
1212: PythonTracer::~PythonTracer() {
1213:   if (active_) {
1214:     TORCH_WARN("`PythonTracer::stop()` was not called.");
1215:     stop();
1216:   }
```

- EN: The main execution path in this span is carried by `unregisterMonitoringCallback`, `SOFT_ASSERT`, `restart`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `unregisterMonitoringCallback`, `SOFT_ASSERT`, `restart` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1217-1248

```cpp
1217:   if (Py_IsInitialized() && !Py_IsFinalizing()) {
1218:     pybind11::gil_scoped_acquire gil;
1219:     Py_XDECREF((PyObject*)shared_ctx_);
1220:   }
1221: }
1222: 
1223: void PythonTracer::recordPyCall(
1224:     ThreadLocalResults& tls,
1225:     PyFrameObject* frame,
1226:     bool is_startup_frame) {
1227:   static constexpr auto E = EventType::PyCall;
1228:   const auto key = [&]() -> TraceKey {
1229:     auto code = THPCodeObjectPtr(PyFrame_GetCode(frame));
1230:     if (code.get() == module_call_code_) {
1231:       // By default, CPython stores locals in a "fast" format, with an array
1232:       // of names and an array of values. Consequently, frame->f_locals is
1233:       // NULL since the interpreter has no need to populate it.
1234:       //
1235:       // If these arrays were part of the public API then we could very
1236:       // quickly access `self`. Unfortunately they are not, and moreover are
1237:       // not stable across versions. As a result, we are forced to call
1238:       // `PyFrame_FastToLocals` which forces the interpreter to materialize
1239:       // the full dict of locals.
1240:       auto locals = THPObjectPtr(PyFrame_GetLocals(frame));
1241: 
1242: #if PY_MAJOR_VERSION < 3 || (PY_MAJOR_VERSION == 3 && PY_MINOR_VERSION < 13)
1243:       auto self =
1244:           THPObjectPtr(Py_XNewRef(PyDict_GetItemString(locals, "self")));
1245: #else
1246:       // In Python-3.13+ `PyFrame_GetLocals()` returns instance of
1247:       // PyFrameLocalsProxy_Type See PEP 667 for more info
1248:       auto self = THPObjectPtr(PyMapping_GetItemString(locals, "self"));
```

- EN: The main execution path in this span is carried by `Py_XDECREF`, `recordPyCall`, `THPCodeObjectPtr`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `Py_XDECREF`, `recordPyCall`, `THPCodeObjectPtr` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1249-1280

```cpp
1249: #endif
1250:       auto back = THPFrameObjectPtr(PyFrame_GetBack(frame));
1251:       TORCH_INTERNAL_ASSERT(back != nullptr);
1252:       return tls.intern<CallType::PyModuleCall, E>(
1253:           frame, self.get(), back.get());
1254:     } else if (code.get() == optimizer_hook_) {
1255:       auto locals = THPObjectPtr(PyFrame_GetLocals(frame));
1256: #if PY_MAJOR_VERSION < 3 || (PY_MAJOR_VERSION == 3 && PY_MINOR_VERSION < 13)
1257:       auto self =
1258:           THPObjectPtr(Py_XNewRef(PyDict_GetItemString(locals, "self")));
1259: #else
1260:       auto self = THPObjectPtr(PyMapping_GetItemString(locals, "self"));
1261: #endif
1262:       auto back = THPFrameObjectPtr(PyFrame_GetBack(frame));
1263:       TORCH_INTERNAL_ASSERT(back != nullptr);
1264:       return tls.intern<CallType::PyOptimizerCall, E>(
1265:           frame, self.get(), back.get());
1266:     } else {
1267:       auto back = THPFrameObjectPtr(PyFrame_GetBack(frame));
1268:       auto f_back = (back.get() != nullptr) ? back.get() : frame;
1269:       return tls.intern<CallType::PyCall, E>(no_ephemeral_t(), frame, f_back);
1270:     }
1271:   }();
1272:   const auto time = c10::getApproximateTime();
1273:   is_startup_frame ? start_frames_.push_back({key, time})
1274:                    : queue_->getSubqueue()->emplace_py_call(key, time);
1275:   ++tls.active_frames_;
1276: }
1277: 
1278: void PythonTracer::recordCCall(
1279:     ThreadLocalResults& tls,
1280:     PyFrameObject* frame,
```

- EN: The main execution path in this span is carried by `THPFrameObjectPtr`, `TORCH_INTERNAL_ASSERT`, `THPObjectPtr`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFrameObjectPtr`, `TORCH_INTERNAL_ASSERT`, `THPObjectPtr` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1312

```cpp
1281:     PyObject* arg,
1282:     bool start_frame) {
1283:   // for starting frames we duplicate callable python functions to avoid having
1284:   // empty C frames in trace when exiting
1285:   if (!start_frame) {
1286:     TORCH_INTERNAL_ASSERT_DEBUG_ONLY(PyCFunction_Check(arg));
1287:   }
1288:   auto fn = reinterpret_cast<PyCFunctionObject*>(arg);
1289: 
1290:   // NB: For C calls a new frame is not created, so we use `frame` rather than
1291:   //     `frame->f_back`.
1292:   auto key = tls.intern<CallType::PyCCall, EventType::PyCCall>(
1293:       arg, (void*)(fn->m_ml), frame);
1294:   queue_->getSubqueue()->emplace_py_call(key, c10::getApproximateTime());
1295:   ++tls.active_frames_;
1296: }
1297: 
1298: // ============================================================================
1299: // == Post processing =========================================================
1300: // ============================================================================
1301: struct Exit {
1302:   bool operator>(const Exit& other) const {
1303:     return t_ > other.t_;
1304:   }
1305: 
1306:   c10::time_t t_;
1307:   size_t python_tid_;
1308: };
1309: 
1310: class PostProcess {
1311:  public:
1312:   PostProcess(
```

- EN: This range declares or shapes types such as `Exit`, `PostProcess`. The main execution path in this span is carried by `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`, `PostProcess`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段声明或塑造了 ``Exit`, `PostProcess`` 等类型。 这一段的主要执行路径由 `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`, `PostProcess` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 1313-1344

```cpp
1313:       std::function<c10::time_t(c10::approx_time_t)> time_converter,
1314:       std::deque<ThreadLocalResults>& tls,
1315:       c10::time_t end_time_ns)
1316:       : end_time_{end_time_ns}, time_converter_{std::move(time_converter)} {
1317:     for (size_t python_tid : c10::irange(tls.size())) {
1318:       CallTypeHelper<TraceKeyCacheState>::map(
1319:           tls[python_tid].trace_keys_,
1320:           *this,
1321:           tls[python_tid].value_cache_,
1322:           python_tid);
1323: 
1324:       addExits<EventType::PyCall>(tls[python_tid].exit_times_, python_tid);
1325:       addExits<EventType::PyCCall>(tls[python_tid].c_exit_times_, python_tid);
1326:     }
1327:   }
1328: 
1329:   void set_start_frames(
1330:       const std::vector<PythonTracer::StartFrame>& start_frames,
1331:       std::vector<python_tracer::CompressedEvent>& enters) {
1332:     for (const auto& frame : start_frames) {
1333:       enters.push_back(
1334:           {frame.trace_key_,
1335:            NoTID, // Allows us to detect unhandled start frames
1336:            {},
1337:            time_converter_(frame.start_time)});
1338:     }
1339:   }
1340: 
1341:   template <CallType C>
1342:   void operator()(
1343:       const TraceKeyCacheState<C>& trace_cache,
1344:       const ValueCache& value_cache,
```

- EN: The main execution path in this span is carried by `time_t`, `move`, `map`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `time_t`, `move`, `map` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 1345-1376

```cpp
1345:       size_t python_tid) {
1346:     for (const auto& it : trace_cache.state_) {
1347:       const auto inserted = get_state<Config<C>::event_type>().fields_.insert(
1348:           {it.second, value_cache.load(it.first, python_tid)});
1349:       TORCH_INTERNAL_ASSERT(inserted.second, "Duplicate key: ", it.second);
1350:     }
1351:   }
1352: 
1353:   template <EventType E, size_t N>
1354:   void addExits(
1355:       AppendOnlyList<c10::approx_time_t, N>& exits,
1356:       size_t python_tid) {
1357:     for (const auto i : exits) {
1358:       get_state<E>().exits_.push({time_converter_(i), python_tid});
1359:     }
1360:   }
1361: 
1362:   std::vector<std::shared_ptr<Result>> run(
1363:       std::vector<python_tracer::CompressedEvent>& enters) {
1364:     std::stable_sort(
1365:         enters.begin(), enters.end(), [](const auto a, const auto b) {
1366:           return a.enter_t_ < b.enter_t_;
1367:         });
1368:     std::vector<std::shared_ptr<Result>> out;
1369:     populate<EventType::PyCall>(enters, out);
1370:     populate<EventType::PyCCall>(enters, out);
1371:     return out;
1372:   }
1373: 
1374:  private:
1375:   template <EventType E>
1376:   void populate(
```

- EN: The main execution path in this span is carried by `TORCH_INTERNAL_ASSERT`, `addExits`, `run`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `TORCH_INTERNAL_ASSERT`, `addExits`, `run` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1377-1408

```cpp
1377:       std::vector<python_tracer::CompressedEvent>& enters,
1378:       std::vector<std::shared_ptr<Result>>& out) {
1379:     using stack_t = std::vector<std::shared_ptr<Result>>;
1380:     const auto initial_size = out.size();
1381:     auto pop = [](stack_t& stack, c10::time_t t) {
1382:       if (!stack.empty()) {
1383:         std::get<ExtraFields<E>>(stack.back()->extra_fields_).end_time_ns_ = t;
1384:         stack.pop_back();
1385:       } else {
1386:         TORCH_WARN_ONCE(
1387:             "Python replay stack is empty during pop operation! May result in incorrect stack tracing.");
1388:       }
1389:     };
1390: 
1391:     ska::flat_hash_map<size_t, stack_t> stacks;
1392:     auto& state = get_state<E>();
1393:     // We already own the GIL at this point
1394:     for (const auto& enter : enters) {
1395:       auto fields_it = state.fields_.find(enter.key_);
1396:       if (fields_it != state.fields_.end()) {
1397:         while (!state.exits_.empty() &&
1398:                state.exits_.top().t_ < enter.enter_t_) {
1399:           auto& exit = state.exits_.top();
1400:           auto& tstack = stacks[exit.python_tid_];
1401:           pop(tstack, exit.t_);
1402:           state.exits_.pop();
1403:         }
1404:         out.push_back(Result::create(
1405:             enter.enter_t_,
1406:             enter.system_tid_,
1407:             enter.kineto_info_,
1408:             fields_it->second));
```

- EN: The main execution path in this span is carried by `TORCH_WARN_ONCE`, `pop`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `TORCH_WARN_ONCE`, `pop` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 1409-1440

```cpp
1409: 
1410:         stacks[fields_it->second.python_tid_].push_back(out.back());
1411:       }
1412:     }
1413: 
1414:     // Handle events which were still running when profiling ended.
1415:     for (auto& i : stacks) {
1416:       while (!i.second.empty()) {
1417:         pop(i.second, end_time_);
1418:       }
1419:     }
1420: 
1421:     // Assign system TIDs to start events based on the system TID of the next
1422:     // observed event with the same Python TID.
1423:     ska::flat_hash_map<size_t, std::pair<size_t, kineto::DeviceAndResource>>
1424:         tid_map;
1425:     auto it = out.rbegin();
1426:     for ([[maybe_unused]] auto _ : c10::irange(initial_size, out.size())) {
1427:       const auto python_tid =
1428:           std::get<ExtraFields<E>>((*it)->extra_fields_).python_tid_;
1429:       if ((*it)->start_tid_ == NoTID && SOFT_ASSERT(E == EventType::PyCall)) {
1430:         const auto& tid_info =
1431:             tid_map.insert({python_tid, {NoTID, kineto::DeviceAndResource()}})
1432:                 .first->second;
1433:         (*it)->start_tid_ = tid_info.first;
1434:         (*it)->kineto_info_ = tid_info.second;
1435:       }
1436:       tid_map[python_tid] = {(*it)->start_tid_, (*it)->kineto_info_};
1437:       ++it;
1438:     }
1439:   }
1440: 
```

- EN: The main execution path in this span is carried by `pop`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `pop` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 1441-1472

```cpp
1441:   template <EventType E>
1442:   struct State {
1443:     ska::flat_hash_map<TraceKey, ExtraFields<E>> fields_;
1444:     std::priority_queue<Exit, std::vector<Exit>, std::greater<>> exits_;
1445:   };
1446: 
1447:   template <EventType E>
1448:   auto& get_state() {
1449:     return std::get < E == EventType::PyCall ? 0 : 1 > (state_);
1450:   }
1451: 
1452:   c10::time_t end_time_;
1453:   std::function<c10::time_t(c10::approx_time_t)> time_converter_;
1454:   std::tuple<State<EventType::PyCall>, State<EventType::PyCCall>> state_;
1455: };
1456: 
1457: struct PythonIDVisitor {
1458:   void operator()(ExtraFields<EventType::PyCall>& py_call) {
1459:     py_call.id_ = ++current_python_id_;
1460:     if (py_call.module_.has_value()) {
1461:       auto& m = py_call.module_;
1462:       auto& module_ids = module_ids_[m->cls_];
1463:       m->id_ = module_ids.insert({m->self_, module_ids.size()}).first->second;
1464:     }
1465:   }
1466: 
1467:   void operator()(ExtraFields<EventType::PyCCall>& py_call) {
1468:     py_call.id_ = ++current_python_id_;
1469:   }
1470: 
1471:   template <typename T>
1472:   void operator()(T& /*unused*/) {}
```

- EN: This range declares or shapes types such as `State`, `PythonIDVisitor`. The main execution path in this span is carried by `get_state`, `time_t`, `operator`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段声明或塑造了 ``State`, `PythonIDVisitor`` 等类型。 这一段的主要执行路径由 `get_state`, `time_t`, `operator` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1473-1504

```cpp
1473: 
1474:   size_t current_python_id_{0};
1475:   ska::flat_hash_map<PyModuleCls, ska::flat_hash_map<PyModuleSelf, size_t>>
1476:       module_ids_;
1477: };
1478: 
1479: std::vector<std::shared_ptr<Result>> PythonTracer::getEvents(
1480:     std::function<c10::time_t(c10::approx_time_t)> time_converter,
1481:     std::vector<python_tracer::CompressedEvent>& enters,
1482:     c10::time_t end_time_ns) {
1483:   for (auto& tls : thread_local_results_) {
1484:     tls.value_cache_.trimPrefixes();
1485:   }
1486:   PostProcess post_process(
1487:       std::move(time_converter), thread_local_results_, end_time_ns);
1488:   post_process.set_start_frames(start_frames_, enters);
1489:   auto out = post_process.run(enters);
1490: 
1491:   std::stable_sort(out.begin(), out.end(), [](const auto& a, const auto& b) {
1492:     return a->start_time_ns_ < b->start_time_ns_;
1493:   });
1494: 
1495:   PythonIDVisitor id_visitor;
1496:   for (auto& i : out) {
1497:     std::visit(id_visitor, i->extra_fields_);
1498:   }
1499: 
1500:   return out;
1501: }
1502: // ============================================================================
1503: // == Memory Tracer ======================================================
1504: // ============================================================================
```

- EN: The main execution path in this span is carried by `getEvents`, `time_t`, `post_process`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `getEvents`, `time_t`, `post_process` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1505-1536

```cpp
1505: 
1506: // Assuming python_tracer::PythonMemoryTracerBase is defined elsewhere
1507: class PythonMemoryTracer final : public python_tracer::PythonMemoryTracerBase {
1508:  public:
1509:   explicit PythonMemoryTracer() = default;
1510:   ~PythonMemoryTracer() override = default;
1511:   void start() override;
1512:   void stop() override;
1513:   void export_memory_history(const std::string& path) override;
1514: };
1515: 
1516: static void toggle_memory_tracing(bool enable) {
1517:   pybind11::gil_scoped_acquire gil;
1518:   THPObjectPtr torch_cuda_memory_module(
1519:       PyImport_ImportModule("torch.cuda.memory"));
1520:   if (!torch_cuda_memory_module) {
1521:     return;
1522:   }
1523:   THPObjectPtr snapshot_func(PyObject_GetAttrString(
1524:       torch_cuda_memory_module.get(), "_record_memory_history_impl"));
1525:   if (!snapshot_func) {
1526:     return;
1527:   }
1528:   // Call the function with arguments
1529:   PyObject* args = PyTuple_New(6);
1530:   PyTuple_SetItem(
1531:       args, 0, enable ? PyUnicode_FromString("all") : Py_NewRef(Py_None));
1532:   PyTuple_SetItem(args, 1, PyUnicode_FromString("all")); // context
1533:   PyTuple_SetItem(args, 2, PyUnicode_FromString("all")); // stacks
1534:   PyTuple_SetItem(args, 3, THPUtils_packInt64(100000)); // max_entries
1535:   PyTuple_SetItem(args, 4, Py_NewRef(Py_None)); // device (None)
1536:   PyTuple_SetItem(args, 5, PyBool_FromLong(0)); // clear_history (False)
```

- EN: This range declares or shapes types such as `PythonMemoryTracer`. The main execution path in this span is carried by `PythonMemoryTracer`, `start`, `stop`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段声明或塑造了 ``PythonMemoryTracer`` 等类型。 这一段的主要执行路径由 `PythonMemoryTracer`, `start`, `stop` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1537-1568

```cpp
1537:   THPObjectPtr result(PyObject_Call(snapshot_func.get(), args, nullptr));
1538:   Py_DECREF(args);
1539: }
1540: 
1541: void PythonMemoryTracer::start() {
1542:   toggle_memory_tracing(true);
1543: }
1544: 
1545: void PythonMemoryTracer::export_memory_history(const std::string& path) {
1546:   pybind11::gil_scoped_acquire gil;
1547:   THPObjectPtr torch_cuda_memory_module(
1548:       PyImport_ImportModule("torch.cuda.memory"));
1549:   if (!torch_cuda_memory_module) {
1550:     return;
1551:   }
1552:   THPObjectPtr snapshot_func(
1553:       PyObject_GetAttrString(torch_cuda_memory_module.get(), "_dump_snapshot"));
1554:   if (!snapshot_func) {
1555:     return;
1556:   }
1557:   THPObjectPtr py_filename(PyUnicode_FromString(path.c_str()));
1558:   THPObjectPtr result(
1559:       PyObject_CallOneArg(snapshot_func.get(), py_filename.get()));
1560: }
1561: 
1562: void PythonMemoryTracer::stop() {
1563:   toggle_memory_tracing(false);
1564: }
1565: 
1566: // ============================================================================
1567: // == API =====================================================================
1568: // ============================================================================
```

- EN: The main execution path in this span is carried by `result`, `Py_DECREF`, `start`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `result`, `Py_DECREF`, `start` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1569-1600

```cpp
1569: int PythonTracer::pyProfileFn(
1570:     PyObject* obj,
1571:     PyFrameObject* frame,
1572:     int what,
1573:     PyObject* arg) {
1574:   HANDLE_TH_ERRORS
1575:   auto* tracer = reinterpret_cast<TraceContext*>(obj)->tracer_;
1576:   auto* local_results = tracer->findThreadLocalResults(PyThreadState_Get());
1577:   if (C10_UNLIKELY(!local_results)) {
1578:     return 0;
1579:   }
1580:   bool acquired = local_results->profile_sem.tryAcquire();
1581:   TORCH_INTERNAL_ASSERT(acquired, "pyProfileFn: profile_sem unexpectedly held");
1582:   // RAII release: ensures the semaphore is released on both normal
1583:   // return and C++ exception paths (e.g. from pybind11 in ValueCache::store).
1584:   auto release_sem =
1585:       c10::make_scope_exit([&]() { local_results->profile_sem.release(); });
1586:   switch (what) {
1587:     case PyTrace_CALL:
1588:       local_results->active_tracer_->recordPyCall(*local_results, frame, false);
1589:       break;
1590: 
1591:     case PyTrace_C_CALL:
1592:       local_results->active_tracer_->recordCCall(*local_results, frame, arg);
1593:       break;
1594: 
1595:     case PyTrace_RETURN:
1596:       local_results->exit_times_.emplace_back(c10::getApproximateTime());
1597:       local_results->active_frames_--;
1598:       if (local_results->active_frames_ <
1599:           local_results->remaining_start_frames_) {
1600:         local_results->remaining_start_frames_ = local_results->active_frames_;
```

- EN: The main execution path in this span is carried by `pyProfileFn`, `TORCH_INTERNAL_ASSERT`, `paths`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `pyProfileFn`, `TORCH_INTERNAL_ASSERT`, `paths` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1601-1632

```cpp
1601:       }
1602:       break;
1603: 
1604:     case PyTrace_C_EXCEPTION:
1605:     case PyTrace_C_RETURN:
1606:       if (local_results->active_frames_ >
1607:           local_results->remaining_start_frames_) {
1608:         local_results->c_exit_times_.emplace_back(c10::getApproximateTime());
1609:         local_results->active_frames_--;
1610:       }
1611:       break;
1612:   }
1613:   return 0;
1614:   END_HANDLE_TH_ERRORS_RET(-1)
1615: }
1616: 
1617: std::unique_ptr<python_tracer::PythonTracerBase> getTracer(
1618:     torch::profiler::impl::RecordQueue* queue) {
1619:   return std::make_unique<PythonTracer>(queue);
1620: }
1621: 
1622: std::unique_ptr<python_tracer::PythonMemoryTracerBase> getMemoryTracer() {
1623:   return std::make_unique<PythonMemoryTracer>();
1624: }
1625: 
1626: } // namespace
1627: } // namespace torch::profiler::impl
1628: 
1629: namespace torch::autograd::profiler::python_tracer {
1630: 
1631: void init() {
1632:   pybind11::gil_scoped_acquire gil;
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `END_HANDLE_TH_ERRORS_RET`, `getTracer`, `getMemoryTracer`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `END_HANDLE_TH_ERRORS_RET`, `getTracer`, `getMemoryTracer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1633-1639

```cpp
1633:   TORCH_CHECK(PyType_Ready(&torch::profiler::impl::TraceContextType) == 0);
1634:   torch::profiler::impl::python_tracer::registerTracer(
1635:       &torch::profiler::impl::getTracer);
1636:   torch::profiler::impl::python_tracer::registerMemoryTracer(
1637:       &torch::profiler::impl::getMemoryTracer);
1638: }
1639: } // namespace torch::autograd::profiler::python_tracer
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `TORCH_CHECK`, `registerTracer`, `registerMemoryTracer`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `TORCH_CHECK`, `registerTracer`, `registerMemoryTracer` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Runtime validation and diagnostics / 运行时校验与诊断
- State coordination and lifetime management / 状态协调与生命周期管理
- Stable ABI / C interface exposure / 稳定 ABI / C 接口暴露

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/autograd/profiler_python.h`, `atomic`, `cstdint`, `deque`, `limits`, `memory`, `queue`, `string`, `unordered_map`, `utility`
- Include roots / 头文件根模块: `ATen`, `c10`, `torch`
- Key symbols / 关键符号: `CallType`, `CodeLocation`, `std`, `CallTypeHelper`, `Config`, `ExtendedPyCallConfig`, `ClsAndParameters`, `Cache`, `Callsite`, `ValueCache`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
