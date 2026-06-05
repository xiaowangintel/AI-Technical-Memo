# collection.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/collection.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
  - CN: 实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Line-by-Line Analysis / 逐行分析

### Lines 1-53
```cpp
 1 | #pragma once
 2 | 
 3 | #include <cstdint>
 4 | #include <memory>
 5 | #include <mutex>
 6 | #include <type_traits>
 7 | #include <utility>
 8 | #include <variant>
 9 | 
10 | #include <ATen/Context.h>
11 | #include <c10/core/Device.h>
12 | #include <c10/core/TensorImpl.h>
13 | #include <c10/macros/Macros.h>
14 | #include <c10/util/ApproximateClock.h>
15 | #include <c10/util/flat_hash_map.h>
16 | #include <c10/util/strong_type.h>
17 | #include <torch/csrc/profiler/containers.h>
18 | #include <torch/csrc/profiler/data_flow.h>
19 | #include <torch/csrc/profiler/events.h>
20 | #include <torch/csrc/profiler/kineto_shim.h>
21 | #include <torch/csrc/profiler/orchestration/python_tracer.h>
22 | #include <torch/csrc/profiler/perf.h>
23 | #include <torch/csrc/profiler/stubs/base.h>
24 | #include <torch/csrc/profiler/util.h>
25 | #include <torch/csrc/utils/python_stub.h>
26 | 
27 | namespace torch::profiler::impl {
28 | 
29 | enum class EventType : uint8_t {
30 |   TorchOp = 0,
31 |   Backend,
32 |   Vulkan,
33 |   Allocation,
34 |   OutOfMemory,
35 |   PyCall,
36 |   PyCCall,
37 |   Kineto,
38 |   PythonGC
39 | };
40 | 
41 | // ============================================================================
42 | // == Value (Tensor, Scalar) summary ==========================================
43 | // ============================================================================
44 | struct TORCH_API RawTensorMetadataBase {
45 |   RawTensorMetadataBase() = default;
46 |   explicit RawTensorMetadataBase(const at::Tensor& t);
47 | 
48 |   StorageImplData data_;
49 |   c10::ScalarType dtype_{c10::ScalarType::Undefined};
50 |   c10::Layout layout_{c10::Layout::Strided};
51 |   uint32_t size_dim_{0};
52 | };
53 | 
```
- EN: Brings in project headers such as `<ATen/Context.h>`, `<c10/core/Device.h>`, `<c10/core/TensorImpl.h>`, `<c10/macros/Macros.h>` and system or third-party headers such as `<cstdint>`, `<memory>`, `<mutex>`, `<type_traits>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `EventType`, `RawTensorMetadataBase` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<ATen/Context.h>`、`<c10/core/Device.h>`、`<c10/core/TensorImpl.h>`、`<c10/macros/Macros.h>`以及系统或第三方头文件，例如 `<cstdint>`、`<memory>`、`<mutex>`、`<type_traits>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `EventType`、`RawTensorMetadataBase` 等数据抽象，用来组织本文件处理的状态。

### Lines 54-108
```cpp
 54 | // Collected during profiling.
 55 | struct TORCH_API RawTensorMetadata : RawTensorMetadataBase {
 56 |   RawTensorMetadata() = default;
 57 |   RawTensorMetadata(const RawTensorMetadata&) = default;
 58 |   RawTensorMetadata(RawTensorMetadata&&) noexcept = default;
 59 |   RawTensorMetadata& operator=(const RawTensorMetadata&) = default;
 60 |   RawTensorMetadata& operator=(RawTensorMetadata&&) noexcept = default;
 61 |   ~RawTensorMetadata() = default;
 62 |   explicit RawTensorMetadata(const at::Tensor& t);
 63 | 
 64 |   // Wrap `weak_self_` in `std::optional` and split device into components to
 65 |   // keep struct default constructable. (which the std::array initializer needs)
 66 |   std::optional<WeakTensor> weak_self_;
 67 |   c10::DeviceType device_type_{c10::DeviceType::CPU};
 68 |   c10::DeviceIndex device_index_{-1};
 69 | };
 70 | 
 71 | // Used during post processing.
 72 | struct TORCH_API TensorMetadata : public RawTensorMetadataBase {
 73 |   TensorMetadata(
 74 |       const RawTensorMetadata& r,
 75 |       std::vector<int64_t> sizes,
 76 |       std::vector<int64_t> strides);
 77 | 
 78 |   TensorImplAddress impl() const {
 79 |     return weak_self_.get();
 80 |   }
 81 | 
 82 |   WeakTensor weak_self_;
 83 |   c10::Device device_;
 84 |   std::vector<int64_t> sizes_;
 85 |   std::vector<int64_t> strides_;
 86 | 
 87 |   // Set during `calculateUniqueTensorIDs`.
 88 |   std::optional<TensorID> id_;
 89 |   std::optional<AllocationID> allocation_id_;
 90 | };
 91 | 
 92 | // Used during post processing.
 93 | struct TORCH_API ProfilerStepInfo {
 94 |   int64_t start_time_ns; // start time of the profiler step
 95 |   int64_t end_time_ns; // end time of the profiler step
 96 |   uint64_t out_idx; // index of the profiler step in the profiler "out" var in
 97 |                     // getRecords
 98 | 
 99 |   ProfilerStepInfo(int64_t start, int64_t end, uint64_t out_idx)
100 |       : start_time_ns(start), end_time_ns(end), out_idx(out_idx) {}
101 | };
102 | 
103 | using op_input_t = std::variant<
104 |     TensorMetadata,
105 |     std::vector<TensorMetadata>,
106 |     c10::IValue,
107 |     std::nullopt_t>;
108 | 
```
- EN: Defines or extends data abstractions such as `RawTensorMetadata`, `default`, `TensorMetadata`, `ProfilerStepInfo` that structure the state handled by this file. Implements routines such as `RawTensorMetadata`, `impl` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `RawTensorMetadata`、`default`、`TensorMetadata`、`ProfilerStepInfo` 等数据抽象，用来组织本文件处理的状态。 实现了 `RawTensorMetadata`、`impl` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 109-156
```cpp
109 | // ============================================================================
110 | // == ExtraFields =============================================================
111 | // ============================================================================
112 | template <EventType>
113 | struct ExtraFields;
114 | 
115 | struct TorchOpBasicFields {
116 |   int64_t sequence_number_{0};
117 |   uint64_t forward_tid_{0};
118 |   at::RecordScope scope_{};
119 |   bool is_async_{false};
120 |   uint64_t record_function_id_{0};
121 |   int64_t debug_handle_{0};
122 |   std::string name_;
123 |   std::string overload_name_;
124 | 
125 |   // Set in the exit callback.
126 |   uint64_t end_tid_{0};
127 | };
128 | 
129 | using jit_stack_t = std::vector<std::string>;
130 | using jit_modules_t = std::vector<std::string>;
131 | using extra_args_t = std::unordered_map<std::string, c10::IValue>;
132 | using extra_meta_t = std::unordered_map<std::string, std::string>;
133 | using kwinputs_t = std::unordered_map<std::string, c10::IValue>;
134 | 
135 | // Mirrors `libkineto::GenericTraceActivity::Flow`. Used during post processing
136 | // to embed Kineto events into the broader profiler tree structure.
137 | struct Flow {
138 |   uint32_t id{0};
139 |   uint32_t type{0};
140 |   uint32_t start{0};
141 | };
142 | 
143 | struct FallbackPair {
144 |   ProfilerVoidEventStub device_event_start_ = nullptr;
145 |   ProfilerVoidEventStub device_event_end_ = nullptr;
146 | };
147 | 
148 | template <>
149 | struct ExtraFields<EventType::TorchOp> : TorchOpBasicFields {
150 |   ExtraFields(
151 |       TorchOpBasicFields&& f,
152 |       uint64_t correlation_id,
153 |       c10::time_t end_time_ns,
154 |       std::vector<op_input_t>&& inputs,
155 |       std::vector<op_input_t>&& concrete_inputs,
156 |       jit_stack_t&& jit_stack,
```
- EN: Defines or extends data abstractions such as `ExtraFields`, `TorchOpBasicFields`, `Flow`, `FallbackPair` that structure the state handled by this file. At the statement level, this block stores long-lived member state for later calls.
- CN: 定义或扩展了 `ExtraFields`、`TorchOpBasicFields`、`Flow`、`FallbackPair` 等数据抽象，用来组织本文件处理的状态。 在语句层面，这一段保存供后续调用使用的长期成员状态。

### Lines 157-210
```cpp
157 |       jit_modules_t&& jit_modules,
158 |       extra_args_t&& extra_args,
159 |       extra_meta_t&& extra_meta,
160 |       kwinputs_t&& kwinputs,
161 |       FallbackPair&& device_fallback,
162 |       bool allow_tf32_cublas,
163 |       std::unique_ptr<perf_counters_t>&& perf_event_counters)
164 |       : TorchOpBasicFields(std::move(f)),
165 |         correlation_id_{correlation_id},
166 |         end_time_ns_{end_time_ns},
167 |         inputs_{std::move(inputs)},
168 |         concrete_inputs_{std::move(concrete_inputs)},
169 |         jit_stack_{std::move(jit_stack)},
170 |         jit_modules_{std::move(jit_modules)},
171 |         extra_args_{std::move(extra_args)},
172 |         extra_meta_{std::move(extra_meta)},
173 |         kwinputs_{std::move(kwinputs)},
174 |         device_fallback_{std::move(device_fallback)},
175 |         allow_tf32_cublas_{allow_tf32_cublas},
176 |         perf_event_counters_{std::move(perf_event_counters)} {}
177 |   uint64_t correlation_id_;
178 |   c10::time_t end_time_ns_;
179 |   std::vector<op_input_t> inputs_;
180 |   std::vector<op_input_t> concrete_inputs_;
181 |   jit_stack_t jit_stack_;
182 |   jit_modules_t jit_modules_;
183 |   extra_args_t extra_args_;
184 |   extra_meta_t extra_meta_;
185 |   kwinputs_t kwinputs_;
186 |   FallbackPair device_fallback_;
187 |   bool allow_tf32_cublas_;
188 |   std::unique_ptr<perf_counters_t> perf_event_counters_;
189 |   std::string metadata_json_;
190 |   Flow flow;
191 | };
192 | 
193 | template <>
194 | struct ExtraFields<EventType::Backend> {
195 |   int64_t start_time_us_;
196 |   int64_t end_time_us_;
197 |   int64_t debug_handle_;
198 |   at::RecordScope scope_;
199 |   std::string name_;
200 |   std::string backend_;
201 |   jit_stack_t jit_stack_;
202 |   jit_modules_t jit_modules_;
203 | };
204 | 
205 | template <>
206 | struct ExtraFields<EventType::PythonGC> {
207 |   std::string phase;
208 |   int64_t duration_ns_;
209 | };
210 | 
```
- EN: Defines or extends data abstractions such as `ExtraFields` that structure the state handled by this file. At the statement level, this block uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `ExtraFields` 等数据抽象，用来组织本文件处理的状态。 在语句层面，这一段使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 211-262
```cpp
211 | template <>
212 | struct ExtraFields<EventType::Vulkan> {
213 |   using raw_event_t = std::pair<c10::approx_time_t, vulkan_id_t>;
214 |   std::string name_;
215 |   int64_t duration_ns_{0};
216 |   // While building the event tree, we want to report a vulkan event's duration
217 |   // as 0 so that its end time doesn't exceed that of its parent cpu op
218 |   bool in_tree_building_{false};
219 | };
220 | 
221 | struct RawAllocation {
222 |   c10::approx_time_t start_time_;
223 |   void* ptr_;
224 |   int64_t alloc_size_;
225 |   size_t total_allocated_;
226 |   size_t total_reserved_;
227 |   c10::DeviceType device_type_;
228 |   c10::DeviceIndex device_index_;
229 | };
230 | 
231 | // For performance.
232 | static_assert(
233 |     std::is_trivial_v<RawAllocation>,
234 |     "Non-Trivial member of RawAllocation.");
235 | 
236 | template <>
237 | struct ExtraFields<EventType::Allocation> : RawAllocation {
238 |   ExtraFields(const RawAllocation& allocation) : RawAllocation(allocation) {}
239 | 
240 |   c10::Device device() const {
241 |     return {device_type_, device_index_};
242 |   }
243 | 
244 |   std::optional<TensorID> id_;
245 |   std::optional<AllocationID> allocation_id_;
246 | };
247 | 
248 | template <>
249 | struct ExtraFields<EventType::OutOfMemory> {
250 |   c10::approx_time_t start_time_;
251 |   int64_t alloc_size_;
252 |   size_t total_allocated_;
253 |   size_t total_reserved_;
254 |   c10::DeviceType device_type_;
255 |   c10::DeviceIndex device_index_;
256 | };
257 | 
258 | // For performance.
259 | static_assert(
260 |     std::is_trivial_v<ExtraFields<EventType::OutOfMemory>>,
261 |     "Non-Trivial member of ExtraFields<EventType::OutOfMemory>.");
262 | 
```
- EN: Defines or extends data abstractions such as `ExtraFields`, `RawAllocation` that structure the state handled by this file. Implements routines such as `device` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `ExtraFields`、`RawAllocation` 等数据抽象，用来组织本文件处理的状态。 实现了 `device` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 263-317
```cpp
263 | struct PyFrameState {
264 |   int line_no_;
265 |   at::StringView filename_;
266 |   at::StringView funcname_;
267 | };
268 | 
269 | template <typename T, typename Tag>
270 | using strong_t = strong::
271 |     type<T, Tag, strong::regular, strong::convertible_to<T>, strong::hashable>;
272 | 
273 | using PyModuleSelf = strong_t<PyObject*, struct PyModuleSelf_>;
274 | using PyModuleCls = strong_t<PyObject*, struct PyModuleCls_>;
275 | using PyMethod = strong_t</*PyMethodDef*/ void*, struct PyMethod_>;
276 | using PyOptimizerSelf = strong_t<PyObject*, struct PyOptSelf_>;
277 | using PyOptimizerCls = strong_t<PyObject*, struct PyOptimizer_>;
278 | 
279 | struct NNModuleInfo {
280 |   struct ParameterInfo {
281 |     std::string name_;
282 |     TensorMetadata metadata_;
283 |     std::optional<TensorMetadata> grad_metadata_;
284 |   };
285 | 
286 |   PyModuleSelf self_;
287 |   PyModuleCls cls_;
288 |   at::StringView cls_name_;
289 | 
290 |   std::vector<ParameterInfo> parameters_;
291 |   // Indicates that `self_` is the kth instance of `cls_` observed.
292 |   size_t id_{std::numeric_limits<size_t>::max()};
293 | };
294 | 
295 | struct OptimizerInfo {
296 |   struct ParameterInfo {
297 |     TensorMetadata metadata_;
298 |     std::optional<TensorMetadata> grad_metadata_;
299 |     std::vector<std::pair<std::string, TensorMetadata>> state_;
300 |   };
301 | 
302 |   PyOptimizerSelf self_;
303 |   PyOptimizerCls cls_;
304 |   at::StringView cls_name_;
305 | 
306 |   std::vector<ParameterInfo> parameters_;
307 | };
308 | 
309 | struct PyExtraFieldsBase {
310 |   PyExtraFieldsBase(
311 |       c10::time_t end_time_ns,
312 |       size_t python_tid,
313 |       PyFrameState caller)
314 |       : end_time_ns_{end_time_ns},
315 |         python_tid_{python_tid},
316 |         caller_{std::move(caller)} {}
317 | 
```
- EN: Defines or extends data abstractions such as `PyFrameState`, `PyModuleSelf_`, `PyModuleCls_`, `PyMethod_`, `PyOptSelf_` that structure the state handled by this file. At the statement level, this block uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `PyFrameState`、`PyModuleSelf_`、`PyModuleCls_`、`PyMethod_`、`PyOptSelf_` 等数据抽象，用来组织本文件处理的状态。 在语句层面，这一段使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 318-363
```cpp
318 |   c10::time_t end_time_ns_;
319 |   size_t python_tid_;
320 |   PyFrameState caller_;
321 | 
322 |   // kth python event observed. (Used by TensorBoard)
323 |   size_t id_{std::numeric_limits<size_t>::max()};
324 | };
325 | 
326 | template <>
327 | struct ExtraFields<EventType::PyCall> : public PyExtraFieldsBase {
328 |   struct args_t {
329 |     PyFrameState frame_state_;
330 |     std::optional<NNModuleInfo> module_info_;
331 |     std::optional<OptimizerInfo> optimizer_info_;
332 |   };
333 | 
334 |   ExtraFields(
335 |       c10::time_t end_time_ns,
336 |       size_t python_tid,
337 |       PyFrameState caller,
338 |       args_t args)
339 |       : PyExtraFieldsBase(end_time_ns, python_tid, std::move(caller)),
340 |         callsite_{std::move(args.frame_state_)},
341 |         module_{std::move(args.module_info_)},
342 |         optimizer_{std::move(args.optimizer_info_)} {}
343 | 
344 |   PyFrameState callsite_;
345 |   std::optional<NNModuleInfo> module_;
346 |   std::optional<OptimizerInfo> optimizer_;
347 | };
348 | 
349 | template <>
350 | struct ExtraFields<EventType::PyCCall> : public PyExtraFieldsBase {
351 |   using args_t = at::StringView;
352 | 
353 |   ExtraFields(
354 |       c10::time_t end_time_ns,
355 |       size_t python_tid,
356 |       PyFrameState caller,
357 |       args_t args)
358 |       : PyExtraFieldsBase(end_time_ns, python_tid, std::move(caller)),
359 |         function_name_{std::move(args)} {}
360 | 
361 |   at::StringView function_name_;
362 | };
363 | 
```
- EN: Defines or extends data abstractions such as `ExtraFields`, `args_t` that structure the state handled by this file. At the statement level, this block uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `ExtraFields`、`args_t` 等数据抽象，用来组织本文件处理的状态。 在语句层面，这一段使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 364-415
```cpp
364 | template <>
365 | struct ExtraFields<EventType::Kineto> {
366 |   std::string name_;
367 |   int64_t duration_ns_{0};
368 |   uint64_t correlation_id_{0};
369 |   libkineto::ActivityType activity_type_;
370 |   Flow flow;
371 |   std::weak_ptr<Result> linked_activity_;
372 |   std::string metadata_json_;
373 |   extra_meta_t extra_meta_;
374 | };
375 | 
376 | struct TORCH_API Result : public std::enable_shared_from_this<Result> {
377 |   template <typename... Args>
378 |   [[nodiscard]] static std::shared_ptr<Result> create(Args... args) {
379 |     return std::shared_ptr<Result>(new Result(std::forward<Args>(args)...));
380 |   }
381 | 
382 |   template <typename T>
383 |   auto visit(T&& visitor) {
384 |     return std::visit(std::forward<T>(visitor), extra_fields_);
385 |   }
386 | 
387 |   template <typename T>
388 |   auto visit(T&& visitor) const {
389 |     return std::visit(std::forward<T>(visitor), extra_fields_);
390 |   }
391 | 
392 |   template <typename T, typename Fn>
393 |   void visit_if_base(const Fn& fn) const {
394 |     visit([&](const auto& extra_fields) {
395 |       using extra_fields_t = typename std::remove_cv_t<
396 |           typename std::remove_reference_t<decltype(extra_fields)>>;
397 | 
398 |       if constexpr (std::is_base_of_v<T, extra_fields_t>) {
399 |         fn(extra_fields);
400 |       }
401 |     });
402 |   }
403 | 
404 |   EventType tag() const {
405 |     return visit([](const auto& i) { return deduceTag(i); });
406 |   }
407 | 
408 |   std::string name() const;
409 |   std::string overload_name() const;
410 |   libkineto::ActivityType kinetoType() const;
411 |   uint64_t correlationID() const;
412 |   int64_t endTimeNS() const;
413 |   uint64_t endTID() const;
414 |   c10::DeviceType deviceType() const;
415 | 
```
- EN: Defines or extends data abstractions such as `ExtraFields`, `Result` that structure the state handled by this file. Implements routines such as `visit`, `visit_if_base`, `tag`, `name`, `overload_name` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `ExtraFields`、`Result` 等数据抽象，用来组织本文件处理的状态。 实现了 `visit`、`visit_if_base`、`tag`、`name`、`overload_name` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 416-470
```cpp
416 |   int64_t start_time_ns_;
417 |   uint64_t start_tid_;
418 |   kineto::DeviceAndResource kineto_info_;
419 |   std::variant<
420 |       ExtraFields<EventType::TorchOp>,
421 |       ExtraFields<EventType::Backend>,
422 |       ExtraFields<EventType::Vulkan>,
423 |       ExtraFields<EventType::Allocation>,
424 |       ExtraFields<EventType::OutOfMemory>,
425 |       ExtraFields<EventType::PyCall>,
426 |       ExtraFields<EventType::PyCCall>,
427 |       ExtraFields<EventType::Kineto>,
428 |       ExtraFields<EventType::PythonGC>>
429 |       extra_fields_;
430 | 
431 |   std::weak_ptr<Result> parent_;
432 |   std::vector<std::shared_ptr<Result>> children_;
433 |   bool finished_{false};
434 |   bool hidden_{false};
435 |   const torch::profiler::impl::kineto::activity_t* kineto_activity_{nullptr};
436 | 
437 |  private:
438 |   template <EventType E>
439 |   Result(
440 |       int64_t start_time_ns,
441 |       uint64_t start_tid,
442 |       kineto::DeviceAndResource kineto_info,
443 |       ExtraFields<E>&& extra_fields)
444 |       : start_time_ns_{start_time_ns},
445 |         start_tid_{start_tid},
446 |         kineto_info_{kineto_info},
447 |         extra_fields_{std::move(extra_fields)} {}
448 | 
449 |   template <EventType E>
450 |   static EventType deduceTag(const ExtraFields<E>& /*unused*/) {
451 |     return E;
452 |   }
453 | };
454 | 
455 | struct KinetoObserverContext : public at::ObserverContext {
456 |   struct Event {
457 |     TorchOpBasicFields basic_fields_;
458 |     c10::approx_time_t start_time_;
459 | 
460 |     // Set in the exit callback.
461 |     c10::approx_time_t end_time_{
462 |         std::numeric_limits<c10::approx_time_t>::min()};
463 | 
464 |     bool allow_tf32_cublas_;
465 |     std::unique_ptr<perf_counters_t> counters_;
466 |     extra_meta_t* extra_nccl_meta_{};
467 |   };
468 | 
469 |   explicit KinetoObserverContext(Event* event) : event_{event} {}
470 | 
```
- EN: Defines or extends data abstractions such as `KinetoObserverContext`, `Event` that structure the state handled by this file. Implements routines such as `deduceTag` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `KinetoObserverContext`、`Event` 等数据抽象，用来组织本文件处理的状态。 实现了 `deduceTag` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 471-521
```cpp
471 |   Event* event_;
472 |   FallbackPair* fallback_{nullptr};
473 | };
474 | 
475 | constexpr int IO_ENCODER_DEFAULT_BLOCK_SIZE = 1024;
476 | 
477 | constexpr int SCALAR_LIST_LENGTH_LIMIT = 30;
478 | 
479 | // InputOutputEncoder
480 | // Stores each op_events' shapes and dtypes, and concrete values into a
481 | // contiguous AppendOnlyList so that we no longer create vectors for shapes
482 | // and dtypes on every op. Those vectors can be created during
483 | // post-processing.
484 | // It splits the data into two categories: input shapes and concrete inputs.
485 | class InputOutputEncoder final {
486 |  public:
487 |   void push(c10::ArrayRef<const c10::IValue> values);
488 | 
489 |   // Used during post-processing to unpack the encoded data.
490 |   // Each method returns a "supplier" lambda which takes no arguments;
491 |   // invoking the lambda once will return a list of args that represent
492 |   // the inputs for one op.
493 |   // The data is split into two streams: "input shapes" and "concrete inputs".
494 |   // Note: "auto" only works because these are only used in collection.cpp,
495 |   // where they are implemented.
496 |   auto getInputShapeGenerator();
497 |   auto getConcreteInputGenerator();
498 | 
499 |   bool isSupportedScalarList(const c10::IValue& list_candidate);
500 | 
501 |   void clear();
502 | 
503 |   enum class Tag {
504 |     Tensor = 0,
505 |     UndefinedTensor,
506 |     TensorListBegin, // TODO: generalize to other lists.
507 |     ScalarList,
508 |     Scalar,
509 |     Other,
510 |     TERMINATOR
511 |   };
512 | 
513 |   enum class IOType { Shapes, ConcreteInputs, None };
514 | 
515 |  private:
516 |   void push(const at::Tensor& t);
517 | 
518 |   // Implementation detail for getInputShapeGenerator and
519 |   // getConcreteInputGenerator
520 |   auto getIValueGenerator(const IOType& io_type);
521 | 
```
- EN: Defines or extends data abstractions such as `InputOutputEncoder`, `Tag`, `IOType` that structure the state handled by this file. Implements routines such as `push`, `getInputShapeGenerator`, `getConcreteInputGenerator`, `isSupportedScalarList`, `clear` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 定义或扩展了 `InputOutputEncoder`、`Tag`、`IOType` 等数据抽象，用来组织本文件处理的状态。 实现了 `push`、`getInputShapeGenerator`、`getConcreteInputGenerator`、`isSupportedScalarList`、`clear` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 522-574
```cpp
522 |   AppendOnlyList<Tag, IO_ENCODER_DEFAULT_BLOCK_SIZE> tags_;
523 |   AppendOnlyList<RawTensorMetadata, IO_ENCODER_DEFAULT_BLOCK_SIZE>
524 |       tensor_metadata_;
525 |   AppendOnlyList<int64_t, IO_ENCODER_DEFAULT_BLOCK_SIZE> tensor_sizes_strides_;
526 |   AppendOnlyList<c10::IValue, IO_ENCODER_DEFAULT_BLOCK_SIZE> ivalues_;
527 | };
528 | 
529 | using perf_profiler_t = torch::profiler::impl::linux_perf::PerfProfiler;
530 | 
531 | class TORCH_API ThreadLocalSubqueue {
532 |  public:
533 |   ThreadLocalSubqueue(const uint64_t tid, ProfilerConfig config);
534 | 
535 |   std::unique_ptr<KinetoObserverContext> begin_op(const at::RecordFunction& fn);
536 | 
537 |   template <class... Args>
538 |   void emplace_backend_event(Args&&... args) {
539 |     backend_events_.emplace_back(std::forward<Args>(args)...);
540 |   }
541 | 
542 |   template <class... Args>
543 |   void emplace_vulkan_event(Args&&... args) {
544 |     vulkan_events_.emplace_back(std::forward<Args>(args)...);
545 |   }
546 | 
547 |   template <class... Args>
548 |   void emplace_allocation_event(Args&&... args) {
549 |     allocations_.emplace_back(std::forward<Args>(args)...);
550 |   }
551 | 
552 |   template <class... Args>
553 |   void emplace_ooms_event(Args&&... args) {
554 |     ooms_.emplace_back(std::forward<Args>(args)...);
555 |   }
556 | 
557 |   template <class... Args>
558 |   void emplace_py_call(Args&&... args) {
559 |     py_calls_.emplace_back(std::forward<Args>(args)...);
560 |   }
561 | 
562 |   template <class... Args>
563 |   void emplace_gc_call(Args&&... args) {
564 |     pythongc_.emplace_back(std::forward<Args>(args)...);
565 |   }
566 | 
567 |   uint64_t tid() const {
568 |     return tid_;
569 |   }
570 | 
571 |   const kineto::DeviceAndResource& kineto_info() const {
572 |     return kineto_info_;
573 |   }
574 | 
```
- EN: Defines or extends data abstractions such as `ThreadLocalSubqueue` that structure the state handled by this file. Implements routines such as `ThreadLocalSubqueue`, `begin_op`, `emplace_backend_event`, `emplace_vulkan_event`, `emplace_allocation_event` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `ThreadLocalSubqueue` 等数据抽象，用来组织本文件处理的状态。 实现了 `ThreadLocalSubqueue`、`begin_op`、`emplace_backend_event`、`emplace_vulkan_event`、`emplace_allocation_event` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 575-630
```cpp
575 |   inline void disable_perf_profiler(perf_counters_t& counters) const {
576 |     perf_profiler_->Disable(counters);
577 |   }
578 | 
579 |  private:
580 |   uint64_t tid_;
581 |   ProfilerConfig config_;
582 |   kineto::DeviceAndResource kineto_info_;
583 |   std::unique_ptr<perf_profiler_t> perf_profiler_;
584 | 
585 |   friend class RecordQueue;
586 |   // See `containers.h` for block size benchmarks.
587 |   static constexpr size_t BlockSize = 512;
588 | 
589 |   struct TorchOpStorage {
590 |     // NB: This is a destructive operation.
591 |     void materialize(
592 |         std::vector<std::shared_ptr<Result>>& out,
593 |         std::vector<ProfilerStepInfo>& step_info,
594 |         const std::function<c10::time_t(c10::approx_time_t)>& time_converter,
595 |         const uint64_t tid,
596 |         const kineto::DeviceAndResource& kineto_info);
597 | 
598 |     template <typename T, size_t ChunkSize>
599 |     class EventBlock : public std::array<T, ChunkSize> {
600 |      public:
601 |       EventBlock();
602 |       uint64_t correlation_id(const T* ptr) const;
603 | 
604 |      private:
605 |       uint64_t id_start_;
606 |     };
607 | 
608 |     using event_t = KinetoObserverContext::Event;
609 |     class OpList : public AppendOnlyList<event_t, BlockSize, EventBlock> {
610 |      public:
611 |       template <class... Args>
612 |       std::pair<event_t*, uint64_t> emplace_back(Args&&... args);
613 |       static uint64_t correlationID(const OpList::Iterator& e);
614 |     } op_events_;
615 | 
616 |     // report_input_shapes
617 |     InputOutputEncoder inputs_outputs_;
618 | 
619 |     // with_stack (JIT)
620 |     AppendOnlyList<jit_stack_t, BlockSize> jit_stack_;
621 | 
622 |     // with_modules
623 |     AppendOnlyList<jit_modules_t, BlockSize> jit_modules_;
624 | 
625 |     // with_flops
626 |     AppendOnlyList<extra_args_t, BlockSize> extra_args_;
627 | 
628 |     // report extra metadata, i.e. collective communication meta
629 |     AppendOnlyList<extra_meta_t, BlockSize> extra_meta_;
630 | 
```
- EN: Defines or extends data abstractions such as `RecordQueue`, `EventBlock`, `OpList`, `TorchOpStorage` that structure the state handled by this file. Implements routines such as `disable_perf_profiler`, `materialize`, `EventBlock`, `correlation_id`, `emplace_back` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 定义或扩展了 `RecordQueue`、`EventBlock`、`OpList`、`TorchOpStorage` 等数据抽象，用来组织本文件处理的状态。 实现了 `disable_perf_profiler`、`materialize`、`EventBlock`、`correlation_id`、`emplace_back` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

### Lines 631-680
```cpp
631 |     // report kwinputs
632 |     AppendOnlyList<kwinputs_t, BlockSize> kwinputs_;
633 | 
634 |     // ProfilerState::KINETO_GPU_FALLBACK or
635 |     // ProfilerState::KINETO_PRIVATEUSE1_FALLBACK
636 |     AppendOnlyList<FallbackPair, BlockSize> device_fallback_;
637 |   } torch_ops_;
638 | 
639 |   // reportBackendEventToActiveKinetoProfiler
640 |   AppendOnlyList<ExtraFields<EventType::Backend>, BlockSize> backend_events_;
641 | 
642 |   // _reportVulkanEventToProfiler
643 |   AppendOnlyList<ExtraFields<EventType::Vulkan>::raw_event_t, BlockSize>
644 |       vulkan_events_;
645 | 
646 |   // reportMemoryUsage
647 |   AppendOnlyList<RawAllocation, BlockSize> allocations_;
648 | 
649 |   // reportOOMs
650 |   AppendOnlyList<ExtraFields<EventType::OutOfMemory>, BlockSize> ooms_;
651 | 
652 |   // with_stack (Python)
653 |   AppendOnlyList<
654 |       std::pair<python_tracer::TraceKey, c10::approx_time_t>,
655 |       BlockSize>
656 |       py_calls_;
657 |   // gc with_stack (Python)
658 |   AppendOnlyList<std::pair<std::string, c10::approx_time_t>, BlockSize>
659 |       pythongc_;
660 | };
661 | 
662 | class TORCH_API RecordQueue {
663 |  public:
664 |   RecordQueue(ProfilerConfig config, std::set<ActivityType> activities);
665 | 
666 |   bool tracePython() const;
667 |   bool getPythonGcEvents() const;
668 |   ThreadLocalSubqueue* getSubqueue();
669 |   void stop();
670 |   void restart();
671 | 
672 |   // NB: This is a destructive operation.
673 |   std::pair<
674 |       std::vector<std::shared_ptr<Result>>,
675 |       std::unique_ptr<torch::profiler::impl::kineto::ActivityTraceWrapper>>
676 |   getRecords(
677 |       std::function<c10::time_t(c10::approx_time_t)> time_converter,
678 |       uint64_t start_time_ns,
679 |       uint64_t end_time_ns);
680 | 
```
- EN: Defines or extends data abstractions such as `RecordQueue` that structure the state handled by this file. Implements routines such as `RecordQueue`, `tracePython`, `getPythonGcEvents`, `getSubqueue`, `stop` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 定义或扩展了 `RecordQueue` 等数据抽象，用来组织本文件处理的状态。 实现了 `RecordQueue`、`tracePython`、`getPythonGcEvents`、`getSubqueue`、`stop` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

### Lines 681-710
```cpp
681 |  private:
682 |   uint32_t id_;
683 |   ProfilerConfig config_;
684 |   std::set<ActivityType> activities_;
685 |   ska::flat_hash_map<uint64_t, std::unique_ptr<ThreadLocalSubqueue>>
686 |       sub_queues_;
687 |   std::mutex sub_queue_mutex_;
688 |   std::unique_ptr<python_tracer::PythonTracerBase> python_tracer_;
689 | };
690 | 
691 | TORCH_API bool get_record_concrete_inputs_enabled();
692 | TORCH_API void set_record_concrete_inputs_enabled_fn(
693 |     std::function<bool()> /*fn*/);
694 | TORCH_API void set_record_concrete_inputs_enabled_val(bool /*val*/);
695 | 
696 | TORCH_API bool get_fwd_bwd_enabled();
697 | TORCH_API void set_fwd_bwd_enabled_fn(std::function<bool()> /*fn*/);
698 | TORCH_API void set_fwd_bwd_enabled_val(bool /*val*/);
699 | 
700 | TORCH_API bool get_cuda_sync_enabled();
701 | TORCH_API void set_cuda_sync_enabled_fn(std::function<bool()> /*fn*/);
702 | TORCH_API void set_cuda_sync_enabled_val(bool /*val*/);
703 | 
704 | // Comms related RecordFunctions will record information about tensor storage
705 | // locations.
706 | TORCH_API bool get_record_tensor_addrs_enabled();
707 | TORCH_API void set_record_tensor_addrs_enabled_fn(std::function<bool()> /*fn*/);
708 | TORCH_API void set_record_tensor_addrs_enabled_val(bool /*val*/);
709 | 
710 | } // namespace torch::profiler::impl
```
- EN: Declares routines such as `get_record_concrete_inputs_enabled`, `set_record_concrete_inputs_enabled_fn`, `set_record_concrete_inputs_enabled_val`, `get_fwd_bwd_enabled`, `set_fwd_bwd_enabled_fn` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 声明了 `get_record_concrete_inputs_enabled`、`set_record_concrete_inputs_enabled_fn`、`set_record_concrete_inputs_enabled_val`、`get_fwd_bwd_enabled`、`set_fwd_bwd_enabled_fn` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `EventType`, `InputOutputEncoder`, `Tag`, `IOType`, `ThreadLocalSubqueue`, `RecordQueue`.
  - CN: `EventType`、`InputOutputEncoder`、`Tag`、`IOType`、`ThreadLocalSubqueue`、`RecordQueue`。
- **Important routines / 重要例程**
  - EN: `RawTensorMetadataBase`, `RawTensorMetadata`, `impl`, `device`, `visit`, `visit_if_base`, `tag`, `name`.
  - CN: `RawTensorMetadataBase`、`RawTensorMetadata`、`impl`、`device`、`visit`、`visit_if_base`、`tag`、`name`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<ATen/Context.h>`, `<c10/core/Device.h>`, `<c10/core/TensorImpl.h>`, `<c10/macros/Macros.h>`, `<c10/util/ApproximateClock.h>`, `<c10/util/flat_hash_map.h>`, `<c10/util/strong_type.h>`, `<torch/csrc/profiler/containers.h>`, `<torch/csrc/profiler/data_flow.h>`, `<torch/csrc/profiler/events.h>`, `<torch/csrc/profiler/kineto_shim.h>`, `<torch/csrc/profiler/orchestration/python_tracer.h>`
- External includes / 外部头文件: `<cstdint>`, `<memory>`, `<mutex>`, `<type_traits>`, `<utility>`, `<variant>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
