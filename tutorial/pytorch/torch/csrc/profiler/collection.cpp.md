# collection.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/collection.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
  - CN: 实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Line-by-Line Analysis / 逐行分析

### Lines 1-103
```cpp
  1 | #include <torch/csrc/profiler/collection.h>
  2 | #include <torch/csrc/profiler/orchestration/vulkan.h>
  3 | 
  4 | #include <algorithm>
  5 | #include <functional>
  6 | #include <limits>
  7 | #include <memory>
  8 | #include <queue>
  9 | #include <type_traits>
 10 | #include <utility>
 11 | 
 12 | #include <fmt/format.h>
 13 | #include <nlohmann/json.hpp>
 14 | 
 15 | #ifdef USE_KINETO
 16 | #include <libkineto.h>
 17 | #endif
 18 | 
 19 | #include <ATen/Context.h>
 20 | #include <ATen/record_function.h>
 21 | #include <c10/util/Exception.h>
 22 | #include <c10/util/flat_hash_map.h>
 23 | #include <c10/util/overloaded.h>
 24 | #include <torch/csrc/jit/runtime/interpreter.h>
 25 | #include <torch/csrc/profiler/data_flow.h>
 26 | #include <torch/csrc/profiler/kineto_shim.h>
 27 | 
 28 | namespace torch::profiler::impl {
 29 | using result_ptr_t = std::shared_ptr<Result>;
 30 | using trace_ptr_t =
 31 |     std::unique_ptr<torch::profiler::impl::kineto::ActivityTraceWrapper>;
 32 | 
 33 | RawTensorMetadataBase::RawTensorMetadataBase(const at::Tensor& t)
 34 |     : data_{t.has_storage() ? t.storage().data() : nullptr},
 35 |       dtype_{t.scalar_type()},
 36 |       layout_{t.layout()},
 37 |       size_dim_{static_cast<uint32_t>(t.sizes().size())} {
 38 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
 39 |       t.sizes().size() <= std::numeric_limits<uint32_t>::max(),
 40 |       "Cannot profile Tensors of size > uint32 max. Got dim: ",
 41 |       t.sizes().size());
 42 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
 43 |       t.sizes().size() == t.strides().size(),
 44 |       "Tensor has mismatching sizes and strides. Sizes: ",
 45 |       t.sizes().size(),
 46 |       " Strides: ",
 47 |       t.strides().size());
 48 | }
 49 | 
 50 | RawTensorMetadata::RawTensorMetadata(const at::Tensor& t)
 51 |     : RawTensorMetadataBase(t),
 52 |       weak_self_{WeakTensor(t)},
 53 |       device_type_{t.device().type()},
 54 |       device_index_{t.device().index()} {}
 55 | 
 56 | TensorMetadata::TensorMetadata(
 57 |     const RawTensorMetadata& r,
 58 |     std::vector<int64_t> sizes,
 59 |     std::vector<int64_t> strides)
 60 |     // NOLINTNEXTLINE(cppcoreguidelines-slicing)
 61 |     : RawTensorMetadataBase(r),
 62 |       weak_self_{r.weak_self_.value_or(WeakTensor(at::Tensor()))},
 63 |       device_{r.device_type_, r.device_index_},
 64 |       sizes_{std::move(sizes)},
 65 |       strides_{std::move(strides)} {
 66 |   SOFT_ASSERT(r.weak_self_.has_value());
 67 | }
 68 | 
 69 | // ============================================================================
 70 | // == PyTorch Ops =============================================================
 71 | // ============================================================================
 72 | 
 73 | namespace {
 74 | struct TagToIOType {
 75 |   InputOutputEncoder::Tag tag;
 76 |   InputOutputEncoder::IOType io_type;
 77 | };
 78 | 
 79 | constexpr int tagCount = ((int)InputOutputEncoder::Tag::TERMINATOR) + 1;
 80 | constexpr std::array<TagToIOType, tagCount> tag_map = {{
 81 |     {InputOutputEncoder::Tag::Tensor, InputOutputEncoder::IOType::Shapes},
 82 |     {InputOutputEncoder::Tag::UndefinedTensor,
 83 |      InputOutputEncoder::IOType::Shapes},
 84 |     {InputOutputEncoder::Tag::TensorListBegin,
 85 |      InputOutputEncoder::IOType::Shapes},
 86 |     {InputOutputEncoder::Tag::ScalarList,
 87 |      InputOutputEncoder::IOType::ConcreteInputs},
 88 |     {InputOutputEncoder::Tag::Scalar, InputOutputEncoder::IOType::Shapes},
 89 |     {InputOutputEncoder::Tag::Other, InputOutputEncoder::IOType::Shapes},
 90 |     {InputOutputEncoder::Tag::TERMINATOR, InputOutputEncoder::IOType::None},
 91 | }};
 92 | 
 93 | constexpr bool allTagsMapped(int idx = 0) {
 94 |   return tag_map[idx].tag == InputOutputEncoder::Tag::TERMINATOR ||
 95 |       ((idx == (int)tag_map[idx].tag) && allTagsMapped(idx + 1));
 96 | }
 97 | static_assert(allTagsMapped(), "tag_map is out of order");
 98 | 
 99 | constexpr InputOutputEncoder::IOType tagToIOType(InputOutputEncoder::Tag tag) {
100 |   return tag_map[(int)tag].io_type;
101 | }
102 | } // namespace
103 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/orchestration/vulkan.h>`, `<ATen/Context.h>`, `<ATen/record_function.h>` and system or third-party headers such as `<algorithm>`, `<functional>`, `<limits>`, `<memory>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `TagToIOType` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/collection.h>`、`<torch/csrc/profiler/orchestration/vulkan.h>`、`<ATen/Context.h>`、`<ATen/record_function.h>`以及系统或第三方头文件，例如 `<algorithm>`、`<functional>`、`<limits>`、`<memory>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `TagToIOType` 等数据抽象，用来组织本文件处理的状态。

### Lines 104-199
```cpp
104 | // ----------------------------
105 | // |  Input / Output encoder  |
106 | // ----------------------------
107 | void InputOutputEncoder::push(c10::ArrayRef<const c10::IValue> values) {
108 |   for (const auto& value : values) {
109 |     if (value.isTensor()) {
110 |       push(value.toTensor());
111 |     } else if (value.isScalar()) {
112 |       tags_.emplace_back(Tag::Scalar);
113 |       // Scalars are small enough that they are stored in ivalues without an
114 |       // extra memory alloc
115 |       // TODO: further optimize this by maybe giving Profiler access to the
116 |       // guts of IValue.
117 |       ivalues_.emplace_back(value);
118 |     } else if (value.isTensorList()) {
119 |       tags_.emplace_back(Tag::TensorListBegin);
120 |       for (const auto& t : value.toTensorList()) {
121 |         push(t);
122 |       }
123 |       tags_.emplace_back(Tag::TERMINATOR);
124 |     } else if (isSupportedScalarList(value)) {
125 |       tags_.emplace_back(Tag::ScalarList);
126 |       ivalues_.emplace_back(value);
127 |     } else {
128 |       tags_.emplace_back(Tag::Other);
129 |     }
130 |   }
131 |   tags_.emplace_back(Tag::TERMINATOR);
132 | }
133 | 
134 | void InputOutputEncoder::push(const at::Tensor& t) {
135 |   // TODO fix nested and symbolic sizes
136 |   if (t.defined() && !t.is_nested() &&
137 |       !t.unsafeGetTensorImpl()->has_symbolic_sizes_strides()) {
138 |     tags_.emplace_back(Tag::Tensor);
139 |     tensor_metadata_.emplace_back(t);
140 |     tensor_sizes_strides_.copy(t.sizes());
141 |     if (t.layout() == at::kStrided) {
142 |       // Only Strided layout tensors have strides
143 |       tensor_sizes_strides_.copy(t.strides());
144 |     }
145 |   } else {
146 |     tags_.emplace_back(Tag::UndefinedTensor);
147 |   }
148 | }
149 | 
150 | bool InputOutputEncoder::isSupportedScalarList(
151 |     const c10::IValue& list_candidate) {
152 |   // Scalar list can be very long. If a list is too long, we shouldn't
153 |   // collect it. This function checks whether the list is a scalar list
154 |   // and whether its length is sufficiently short.
155 | 
156 |   if (!get_record_concrete_inputs_enabled()) {
157 |     return false;
158 |   }
159 | 
160 |   if (!list_candidate.isList()) {
161 |     return false;
162 |   }
163 |   auto list_ref = list_candidate.toListRef();
164 |   if (C10_UNLIKELY(list_ref.empty())) {
165 |     return true;
166 |   }
167 |   if (C10_UNLIKELY(!list_ref[0].isScalar())) {
168 |     return false;
169 |   }
170 |   if (C10_UNLIKELY(list_ref.size() > SCALAR_LIST_LENGTH_LIMIT)) {
171 |     return false;
172 |   }
173 |   return true;
174 | }
175 | 
176 | // This function returns a lambda which is a custom-iterator-like getter.
177 | // Each invocation of the lambda returns input values for one op.
178 | //
179 | // io_type is used to filter the ivalues between 'Shapes' and 'Concrete Args'.
180 | // Shapes are used to represent the shapes of tensors. We save only the shapes
181 | //   of the tensors because tensors can be large.
182 | // Concrete args are separated to clarify that they are the actual values.
183 | auto InputOutputEncoder::getIValueGenerator(const IOType& io_type) {
184 |   return [this,
185 |           tag_it = tags_.begin(),
186 |           tensor_metadata_it = tensor_metadata_.begin(),
187 |           tensor_size_strides_it = tensor_sizes_strides_.begin(),
188 |           ivals_it = ivalues_.begin(),
189 |           io_type]() mutable {
190 |     auto decode_tensor = [&]() -> TensorMetadata {
191 |       std::vector<int64_t> sizes;
192 |       std::vector<int64_t> strides;
193 |       if (tensor_metadata_it.exhausted()) {
194 |         LOG(WARNING)
195 |             << "Tensor metadata exhausted prematurely. Reported shapes may be inaccurate!";
196 |         return {RawTensorMetadata(), sizes, strides};
197 |       }
198 |       const auto& raw_metadata = *tensor_metadata_it++;
199 |       for ([[maybe_unused]] const auto _ :
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 200-303
```cpp
200 |            c10::irange(raw_metadata.size_dim_)) {
201 |         if (tensor_size_strides_it.exhausted()) {
202 |           LOG(WARNING)
203 |               << "Expected Tensor Size mismatch with raw Tensor metadata. Reported shapes may be inaccurate!";
204 |           return {raw_metadata, sizes, strides};
205 |         }
206 |         sizes.push_back(*tensor_size_strides_it++);
207 |       }
208 |       if (raw_metadata.layout_ == at::kStrided) {
209 |         for ([[maybe_unused]] const auto _ :
210 |              c10::irange(raw_metadata.size_dim_)) {
211 |           if (tensor_size_strides_it.exhausted()) {
212 |             LOG(WARNING)
213 |                 << "Expected Tensor Strides mismatch with raw Tensor metadata. Reported shapes may be inaccurate!";
214 |             return {raw_metadata, sizes, strides};
215 |           }
216 |           strides.push_back(*tensor_size_strides_it++);
217 |         }
218 |       }
219 |       return {raw_metadata, sizes, strides};
220 |     };
221 | 
222 |     std::vector<op_input_t> out;
223 |     auto push_value = [&out, io_type](const Tag& tag, op_input_t input) {
224 |       if (io_type == tagToIOType(tag)) {
225 |         out.emplace_back(std::move(input));
226 |       } else {
227 |         out.emplace_back(std::nullopt);
228 |       }
229 |     };
230 | 
231 |     bool terminate = false;
232 |     while (!terminate && tag_it != tags_.end()) {
233 |       switch (*tag_it) {
234 |         case Tag::Tensor:
235 |           push_value(*tag_it, decode_tensor());
236 |           break;
237 | 
238 |         case Tag::TensorListBegin: {
239 |           std::vector<TensorMetadata> arg;
240 |           bool found_undefined = false;
241 |           while (*(++tag_it) != Tag::TERMINATOR) {
242 |             if (*tag_it == Tag::UndefinedTensor) {
243 |               found_undefined = true;
244 |               continue;
245 |             }
246 |             TORCH_INTERNAL_ASSERT(*tag_it == Tag::Tensor, (int)(*tag_it));
247 |             arg.emplace_back(decode_tensor());
248 |           }
249 |           if (found_undefined) {
250 |             push_value(*tag_it, std::nullopt);
251 |           } else {
252 |             push_value(Tag::TensorListBegin, std::move(arg));
253 |           }
254 |         } break;
255 | 
256 |         case Tag::ScalarList:
257 |         case Tag::Scalar:
258 |           push_value(*tag_it, *ivals_it++);
259 |           break;
260 | 
261 |         case Tag::UndefinedTensor:
262 |         case Tag::Other:
263 |           push_value(*tag_it, std::nullopt);
264 |           break;
265 | 
266 |         case Tag::TERMINATOR:
267 |           // This marks the end of this op.
268 |           terminate = true;
269 |           break;
270 | 
271 |         default:
272 |           break;
273 |       }
274 |       ++tag_it;
275 |     }
276 |     return out;
277 |   };
278 | }
279 | 
280 | auto InputOutputEncoder::getInputShapeGenerator() {
281 |   return getIValueGenerator(IOType::Shapes);
282 | }
283 | 
284 | auto InputOutputEncoder::getConcreteInputGenerator() {
285 |   return getIValueGenerator(IOType::ConcreteInputs);
286 | }
287 | 
288 | void InputOutputEncoder::clear() {
289 |   tags_.clear();
290 |   tensor_metadata_.clear();
291 |   tensor_sizes_strides_.clear();
292 |   ivalues_.clear();
293 | }
294 | 
295 | // ---------------------------------------------------
296 | // |  Correlation ID tracking (OpList & EventBlock)  |
297 | // ---------------------------------------------------
298 | template <typename T, size_t ChunkSize>
299 | ThreadLocalSubqueue::TorchOpStorage::EventBlock<T, ChunkSize>::EventBlock() {
300 |   static std::atomic<uint64_t> counter_{0};
301 |   id_start_ = 1 + ChunkSize * counter_++;
302 | }
303 | 
```
- EN: Implements routines such as `push_value`, `getIValueGenerator` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 实现了 `push_value`、`getIValueGenerator` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 304-398
```cpp
304 | template <class... Args>
305 | std::pair<KinetoObserverContext::Event*, uint64_t> ThreadLocalSubqueue::
306 |     TorchOpStorage::OpList::emplace_back(Args&&... args) {
307 |   auto event_ptr = AppendOnlyList::emplace_back(std::forward<Args>(args)...);
308 |   auto corr_id = buffer_last_->correlation_id(event_ptr);
309 |   return {event_ptr, corr_id};
310 | }
311 | 
312 | uint64_t ThreadLocalSubqueue::TorchOpStorage::OpList::correlationID(
313 |     const OpList::Iterator& e) {
314 |   return e.address().first->correlation_id(&*e);
315 | }
316 | 
317 | template <typename T, size_t ChunkSize>
318 | uint64_t ThreadLocalSubqueue::TorchOpStorage::EventBlock<T, ChunkSize>::
319 |     correlation_id(const T* ptr) const {
320 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
321 |       ptr >= this->data() && ptr < this->data() + ChunkSize);
322 |   return id_start_ + (ptr - this->data());
323 | }
324 | 
325 | // ---------------------------------
326 | // |  Collection (Observer logic)  |
327 | // ---------------------------------
328 | std::unique_ptr<KinetoObserverContext> ThreadLocalSubqueue::begin_op(
329 |     const at::RecordFunction& fn) {
330 |   auto overload_name = config_.experimental_config.capture_overload_names
331 |       ? fn.overload_name()
332 |       : "";
333 |   auto [event, corr_id] = torch_ops_.op_events_.emplace_back(
334 |       torch::profiler::impl::TorchOpBasicFields{
335 |           fn.seqNr(),
336 |           fn.forwardThreadId(),
337 |           fn.scope(),
338 |           fn.isAsync(),
339 |           fn.handle(),
340 |           fn.debugHandle(),
341 |           fn.name(),
342 |           overload_name});
343 |   if (config_.report_input_shapes) {
344 |     torch_ops_.inputs_outputs_.push(fn.inputs());
345 |     torch_ops_.kwinputs_.emplace_back(fn.kwinputs());
346 |   }
347 |   if (!config_.experimental_config.disable_external_correlation) {
348 |     if (fn.scope() == at::RecordScope::USER_SCOPE) {
349 |       torch::profiler::impl::kineto::pushUserCorrelationId(corr_id);
350 |     } else {
351 |       torch::profiler::impl::kineto::pushCorrelationId(corr_id);
352 |     }
353 |   }
354 | 
355 | #if !defined BUILD_LITE_INTERPRETER && !defined C10_MOBILE
356 |   // backward nodes source range corresponds to the forward node
357 |   // TODO: consider using C++ stack trace
358 |   if (config_.with_stack && fn.scope() != at::RecordScope::BACKWARD_FUNCTION) {
359 |     auto cs = torch::profiler::impl::prepareCallstack(jit::currentCallstack());
360 |     torch_ops_.jit_stack_.emplace_back(callstackStr(cs));
361 |   }
362 |   if (config_.with_modules &&
363 |       fn.scope() != at::RecordScope::BACKWARD_FUNCTION) {
364 |     torch_ops_.jit_modules_.emplace_back(jit::currentModuleHierarchy());
365 |   }
366 | #endif
367 |   if (config_.with_flops) {
368 |     torch_ops_.extra_args_.emplace_back(
369 |         torch::profiler::impl::saveExtraArgs(fn));
370 |   }
371 | 
372 |   auto out = std::make_unique<KinetoObserverContext>(event);
373 |   if (fn.isNcclMeta()) {
374 |     // Record NCCL metadata for specific CPU ops, switch off output
375 |     // introspection in this begin_op callback, we will do that in exit callback
376 |     // if needed.
377 |     torch::profiler::impl::SaveNcclMetaConfig ncclMetaConfig{
378 |         true, true, true, false};
379 |     out->event_->extra_nccl_meta_ = torch_ops_.extra_meta_.emplace_back(
380 |         torch::profiler::impl::saveNcclMeta(fn, ncclMetaConfig));
381 |   } else {
382 |     out->event_->extra_nccl_meta_ = torch_ops_.extra_meta_.emplace_back();
383 |   }
384 | 
385 |   if (config_.state == ProfilerState::KINETO_GPU_FALLBACK) {
386 |     try {
387 |       out->fallback_ = torch_ops_.device_fallback_.emplace_back();
388 |       torch::profiler::impl::cudaStubs()->record(
389 |           nullptr, &out->fallback_->device_event_start_, nullptr);
390 |     } catch (const std::exception& e) {
391 |       LOG(WARNING) << "Failed to record CUDA event. " << e.what();
392 |     }
393 |   } else if (config_.state == ProfilerState::KINETO_PRIVATEUSE1_FALLBACK) {
394 |     out->fallback_ = torch_ops_.device_fallback_.emplace_back();
395 |     torch::profiler::impl::privateuse1Stubs()->record(
396 |         nullptr, &out->fallback_->device_event_start_, nullptr);
397 |   }
398 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `correlation_id` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `correlation_id` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 399-490
```cpp
399 |   event->start_time_ = c10::getApproximateTime();
400 |   event->allow_tf32_cublas_ =
401 |       at::globalContext().float32Precision(
402 |           at::Float32Backend::CUDA, at::Float32Op::MATMUL) ==
403 |       at::Float32Precision::TF32;
404 |   if (!config_.experimental_config.performance_events.empty()) {
405 |     const size_t n = config_.experimental_config.performance_events.size();
406 |     event->counters_ = std::make_unique<perf_counters_t>(n, 0);
407 |     perf_profiler_->Enable();
408 |   }
409 |   return out;
410 | }
411 | 
412 | // ---------------
413 | // |  Collation  |
414 | // ---------------
415 | namespace {
416 | template <typename T>
417 | struct StealOrDefault {
418 |   explicit StealOrDefault(T& container)
419 |       : container_{container}, it_{container.begin()} {}
420 | 
421 |   StealOrDefault(const StealOrDefault&) = delete;
422 |   StealOrDefault(StealOrDefault&&) = delete;
423 |   StealOrDefault& operator=(const StealOrDefault&) = delete;
424 |   StealOrDefault& operator=(StealOrDefault&&) = delete;
425 |   ~StealOrDefault() {
426 |     container_.get().clear();
427 |   }
428 | 
429 |   typename T::Iterator::value_type operator()() {
430 |     if (it_.exhausted()) {
431 |       return typename T::Iterator::value_type();
432 |     } else {
433 |       auto result = std::move(*it_);
434 |       ++it_;
435 |       return result;
436 |     }
437 |   }
438 | 
439 |   std::reference_wrapper<T> container_;
440 |   typename T::Iterator it_;
441 | };
442 | } // namespace
443 | 
444 | static constexpr std::string_view profilerStepString = "ProfilerStep#";
445 | 
446 | void ThreadLocalSubqueue::TorchOpStorage::materialize(
447 |     std::vector<std::shared_ptr<Result>>& out,
448 |     std::vector<ProfilerStepInfo>& step_info,
449 |     const std::function<c10::time_t(c10::approx_time_t)>& time_converter,
450 |     const uint64_t tid,
451 |     const kineto::DeviceAndResource& kineto_info) {
452 |   // Plumb Autograd info to the top level annotation.
453 |   auto it = op_events_.begin();
454 |   for ([[maybe_unused]] const auto _ :
455 |        c10::irange(static_cast<int64_t>(op_events_.size()) - 1)) {
456 |     auto& first = it->basic_fields_;
457 |     auto& second = (++it)->basic_fields_;
458 |     if (first.scope_ == at::RecordScope::FUNCTION &&
459 |         second.scope_ == at::RecordScope::BACKWARD_FUNCTION &&
460 |         first.name_.rfind("autograd::engine::evaluate_function: ", 0) == 0) {
461 |       first.sequence_number_ = second.sequence_number_;
462 |       first.forward_tid_ = second.forward_tid_;
463 |     }
464 |   }
465 | 
466 |   // `AccumulateGrad` is an important marker for profile analysis; however the
467 |   // annotation relies on `c10::demangle` which is platform dependent. In
468 |   // particular, Windows will add a "struct " prefix.
469 |   const std::string accumulate_grad = "torch::autograd::AccumulateGrad";
470 |   const std::string windows_pattern = std::string("struct ") + accumulate_grad;
471 |   for (auto& event : op_events_) {
472 |     auto& name = event.basic_fields_.name_;
473 |     auto position = name.find(windows_pattern);
474 |     if (position != std::string::npos) {
475 |       name.replace(position, windows_pattern.size(), accumulate_grad);
476 |     }
477 |   }
478 | 
479 |   auto input_shape_getter = inputs_outputs_.getInputShapeGenerator();
480 |   auto concrete_input_getter = inputs_outputs_.getConcreteInputGenerator();
481 | 
482 |   // TODO: CTAD will take care of template args when we move to C++17
483 |   auto jit_stack = StealOrDefault<decltype(jit_stack_)>(jit_stack_);
484 |   auto jit_module = StealOrDefault<decltype(jit_modules_)>(jit_modules_);
485 |   auto extra_args = StealOrDefault<decltype(extra_args_)>(extra_args_);
486 |   auto extra_meta = StealOrDefault<decltype(extra_meta_)>(extra_meta_);
487 |   auto kwinputs = StealOrDefault<decltype(kwinputs_)>(kwinputs_);
488 |   auto gpu_fallback =
489 |       StealOrDefault<decltype(device_fallback_)>(device_fallback_);
490 | 
```
- EN: Defines or extends data abstractions such as `StealOrDefault` that structure the state handled by this file. Implements routines such as `operator` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `StealOrDefault` 等数据抽象，用来组织本文件处理的状态。 实现了 `operator` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 491-594
```cpp
491 |   for (auto event = op_events_.begin(); event != op_events_.end(); ++event) {
492 |     ExtraFields<EventType::TorchOp> e{
493 |         std::move(event->basic_fields_),
494 |         ThreadLocalSubqueue::TorchOpStorage::OpList::correlationID(event),
495 |         time_converter(event->end_time_),
496 |         input_shape_getter(),
497 |         concrete_input_getter(),
498 |         jit_stack(),
499 |         jit_module(),
500 |         extra_args(),
501 |         extra_meta(),
502 |         kwinputs(),
503 |         gpu_fallback(),
504 |         event->allow_tf32_cublas_,
505 |         std::move(event->counters_)};
506 | 
507 |     if (e.name_.find(profilerStepString) != std::string::npos) {
508 |       step_info.emplace_back(
509 |           time_converter(event->start_time_),
510 |           time_converter(event->end_time_),
511 |           out.size());
512 |     }
513 |     out.emplace_back(Result::create(
514 |         time_converter(event->start_time_), tid, kineto_info, std::move(e)));
515 |   }
516 | 
517 |   op_events_.clear();
518 |   inputs_outputs_.clear();
519 | }
520 | 
521 | template <size_t BlockSize>
522 | static void materialize_vulkan(
523 |     std::vector<std::shared_ptr<Result>>& out,
524 |     AppendOnlyList<ExtraFields<EventType::Vulkan>::raw_event_t, BlockSize>&
525 |         raw_events,
526 |     const std::function<c10::time_t(c10::approx_time_t)>& time_converter,
527 |     const uint64_t tid,
528 |     const kineto::DeviceAndResource& kineto_info) {
529 |   for (const auto& i : raw_events) {
530 |     const auto name_and_duration_ns =
531 |         torch::profiler::impl::vulkan::getShaderNameAndDurationNs(i.second);
532 | 
533 |     out.emplace_back(Result::create(
534 |         /*start_time_ns_=*/time_converter(i.first),
535 |         /*start_tid_=*/tid,
536 |         /*kineto_info_=*/kineto_info,
537 |         /*extra_fields_=*/
538 |         ExtraFields<EventType::Vulkan>{
539 |             /*name_=*/std::get<0>(name_and_duration_ns),
540 |             /*duration_ns_=*/
541 |             static_cast<int64_t>(std::get<1>(name_and_duration_ns)),
542 |             /*in_tree_building_=*/false}));
543 |   }
544 |   raw_events.clear();
545 | }
546 | 
547 | namespace {
548 | // See `RecordQueue::getSubqueue()` for an overview of this cache.
549 | struct SubQueueThreadCache {
550 |   uint32_t key_;
551 |   ThreadLocalSubqueue* ref_;
552 | };
553 | 
554 | // The astute observer will note that this leaves a dangling reference; nothing
555 | // in the teardown of `RecordQueue` or `ThreadLocalSubqueue` clears this value.
556 | // (And the raw pointer in `SubQueueThreadCache` will not extend the lifetime
557 | // of `*ref_`.) This is safe, however, because `getSubqueue` will check
558 | // `sub_queue_cache_.key_` before attempting to access `ref_`, and if `key_`
559 | // does not match the RecordQueue's *unique* `id_` it will evict
560 | // `sub_queue_cache_` and fall back to a different mechanism.
561 | std::atomic<uint32_t> queue_id_{0};
562 | thread_local SubQueueThreadCache sub_queue_cache_{0, nullptr};
563 | 
564 | std::string toString(const ExtraFields<EventType::PyCall>& e) {
565 |   if (e.module_.has_value()) {
566 |     return fmt::format(
567 |         "nn.Module: {}_{}", e.module_->cls_name_.str(), e.module_->id_);
568 |   }
569 |   return fmt::format(
570 |       "{}({}): {}",
571 |       e.callsite_.filename_.str(),
572 |       e.callsite_.line_no_,
573 |       e.callsite_.funcname_.str());
574 | }
575 | 
576 | auto scopeToType(at::RecordScope scope) {
577 |   return scope == at::RecordScope::USER_SCOPE
578 |       ? libkineto::ActivityType::USER_ANNOTATION
579 |       : libkineto::ActivityType::CPU_OP;
580 | }
581 | 
582 | int64_t torchOpEndNS(
583 |     const ExtraFields<EventType::TorchOp>& e,
584 |     const bool finished,
585 |     const std::weak_ptr<Result>& parent) {
586 |   if (finished && e.end_time_ns_ == std::numeric_limits<c10::time_t>::min()) {
587 |     auto p = parent.lock();
588 |     if (p) {
589 |       return p->endTimeNS();
590 |     }
591 |   }
592 |   return e.end_time_ns_;
593 | }
594 | 
```
- EN: Defines or extends data abstractions such as `SubQueueThreadCache` that structure the state handled by this file. Implements routines such as `materialize_vulkan`, `toString`, `scopeToType`, `torchOpEndNS` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `SubQueueThreadCache` 等数据抽象，用来组织本文件处理的状态。 实现了 `materialize_vulkan`、`toString`、`scopeToType`、`torchOpEndNS` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 595-686
```cpp
595 | auto kinetoEventCorrelationID(
596 |     const ExtraFields<EventType::Kineto>& e,
597 |     const std::weak_ptr<Result>& parent) {
598 |   if (e.correlation_id_) {
599 |     return e.correlation_id_;
600 |   }
601 |   auto p = parent.lock();
602 |   return p ? p->correlationID() : 0;
603 | }
604 | } // namespace
605 | 
606 | #define ATTRIBUTE(event_type, expr)                  \
607 |   [&](const ExtraFields<EventType::event_type>& e) { \
608 |     (void)e;                                         \
609 |     return expr;                                     \
610 |   }
611 | 
612 | std::string Result::name() const {
613 |   return visit(c10::overloaded(
614 |       ATTRIBUTE(Vulkan, std::string(e.name_)),
615 |       ATTRIBUTE(Allocation, std::string("[memory]")),
616 |       ATTRIBUTE(OutOfMemory, std::string("[OutOfMemory]")),
617 |       ATTRIBUTE(PyCall, toString(e)),
618 |       ATTRIBUTE(PyCCall, std::string(e.function_name_.str())),
619 |       ATTRIBUTE(PythonGC, std::string("Python GC")),
620 |       [](const auto& e) -> std::string { return e.name_; }));
621 | }
622 | 
623 | std::string Result::overload_name() const {
624 |   return visit(c10::overloaded(
625 |       ATTRIBUTE(TorchOp, std::string(e.overload_name_)),
626 |       [](const auto& e) -> std::string { return ""; }));
627 | }
628 | 
629 | libkineto::ActivityType Result::kinetoType() const {
630 |   return visit(c10::overloaded(
631 |       ATTRIBUTE(TorchOp, scopeToType(e.scope_)),
632 |       ATTRIBUTE(Backend, scopeToType(e.scope_)),
633 |       ATTRIBUTE(Vulkan, libkineto::ActivityType::CPU_OP),
634 |       ATTRIBUTE(Allocation, libkineto::ActivityType::CPU_INSTANT_EVENT),
635 |       ATTRIBUTE(OutOfMemory, libkineto::ActivityType::CPU_INSTANT_EVENT),
636 |       ATTRIBUTE(PyCall, libkineto::ActivityType::PYTHON_FUNCTION),
637 |       ATTRIBUTE(PyCCall, libkineto::ActivityType::PYTHON_FUNCTION),
638 |       ATTRIBUTE(PythonGC, libkineto::ActivityType::PYTHON_FUNCTION),
639 |       ATTRIBUTE(Kineto, e.activity_type_)));
640 | }
641 | 
642 | uint64_t Result::correlationID() const {
643 |   return visit(c10::overloaded(
644 |       ATTRIBUTE(TorchOp, e.correlation_id_),
645 |       ATTRIBUTE(Kineto, kinetoEventCorrelationID(e, parent_)),
646 |       [&](const auto&) -> uint64_t { return 0; }));
647 | }
648 | 
649 | int64_t Result::endTimeNS() const {
650 |   auto end_time_ns = visit(c10::overloaded(
651 |       ATTRIBUTE(TorchOp, torchOpEndNS(e, finished_, parent_)),
652 |       ATTRIBUTE(Backend, e.end_time_us_ * 1000),
653 |       ATTRIBUTE(
654 |           Vulkan, start_time_ns_ + (e.in_tree_building_ ? 0 : e.duration_ns_)),
655 |       ATTRIBUTE(Allocation, start_time_ns_),
656 |       ATTRIBUTE(OutOfMemory, start_time_ns_),
657 |       ATTRIBUTE(Kineto, start_time_ns_ + e.duration_ns_),
658 |       ATTRIBUTE(PythonGC, start_time_ns_ + e.duration_ns_),
659 |       [&](const auto& e) -> int64_t { return e.end_time_ns_; }));
660 | 
661 |   // In rare cases we're willing to tolerate ops which are missing an end time
662 |   // so long as they can borrow their parent's end time. A consequence of this,
663 |   // however, is that `endTimeNS` may not make sense until tree construction is
664 |   // complete.
665 |   auto end_time_is_valid =
666 |       !finished_ || SOFT_ASSERT(end_time_ns >= start_time_ns_, name());
667 |   return end_time_is_valid ? end_time_ns : start_time_ns_;
668 | }
669 | 
670 | uint64_t Result::endTID() const {
671 |   return visit(c10::overloaded(
672 |       ATTRIBUTE(TorchOp, e.end_tid_),
673 |       [&](const auto&) -> uint64_t { return start_tid_; }));
674 | }
675 | 
676 | c10::DeviceType Result::deviceType() const {
677 |   using torch::autograd::profiler::deviceTypeFromActivity;
678 |   return visit(c10::overloaded(
679 |       ATTRIBUTE(Vulkan, c10::DeviceType::Vulkan),
680 |       ATTRIBUTE(Allocation, e.device_type_),
681 |       ATTRIBUTE(OutOfMemory, e.device_type_),
682 |       ATTRIBUTE(Kineto, deviceTypeFromActivity(e.activity_type_)),
683 |       [&](const auto&) { return c10::DeviceType::CPU; }));
684 | }
685 | #undef ATTRIBUTE
686 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `kinetoEventCorrelationID`, `visit` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `kinetoEventCorrelationID`、`visit` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 687-774
```cpp
687 | ThreadLocalSubqueue::ThreadLocalSubqueue(
688 |     const uint64_t tid,
689 |     ProfilerConfig config)
690 |     : tid_{tid},
691 |       config_{std::move(config)},
692 |       kineto_info_{kineto::kineto_ids()} {
693 |   torch::profiler::impl::kineto::recordThreadInfo();
694 |   if (!config_.experimental_config.performance_events.empty()) {
695 |     perf_profiler_ =
696 |         std::make_unique<torch::profiler::impl::linux_perf::PerfProfiler>();
697 |     perf_profiler_->Configure(config_.experimental_config.performance_events);
698 |   }
699 | }
700 | 
701 | RecordQueue::RecordQueue(
702 |     ProfilerConfig config,
703 |     std::set<ActivityType> activities)
704 |     : id_(++queue_id_),
705 |       config_{std::move(config)},
706 |       activities_{std::move(activities)} {
707 |   if (tracePython()) {
708 |     python_tracer_ = python_tracer::PythonTracerBase::make(this);
709 |     if (getPythonGcEvents()) {
710 |       python_tracer_->register_gc_callback();
711 |     }
712 |   }
713 | }
714 | 
715 | bool RecordQueue::tracePython() const {
716 |   return config_.with_stack && activities_.count(ActivityType::CPU);
717 | }
718 | 
719 | bool RecordQueue::getPythonGcEvents() const {
720 |   return config_.experimental_config.record_python_gc_info;
721 | }
722 | 
723 | ThreadLocalSubqueue* RecordQueue::getSubqueue() {
724 |   // In the most common case, a thread will want to write to the same sub-queue
725 |   // that it wrote to last call. The only time that isn't true is if:
726 |   //  A) The profiler context has ended and we are in a new one.
727 |   //  B) Two profilers are active in different TLS contexts, and this thread
728 |   //     is a worker helping with intra-op parallelism.
729 |   // Since we expect this to be the OVERWHELMINGLY common case (>99%), we add a
730 |   // special thread_local cache so that we can skip the overall `flat_hash_map`
731 |   // (and corresponding lock).
732 |   if (id_ == sub_queue_cache_.key_) {
733 |     return sub_queue_cache_.ref_;
734 |   }
735 | 
736 |   const auto tid = at::RecordFunction::currentThreadId();
737 |   std::lock_guard<std::mutex> guard(sub_queue_mutex_);
738 |   auto it = sub_queues_.find(tid);
739 |   if (it == sub_queues_.end()) {
740 |     it = sub_queues_
741 |              .emplace(tid, std::make_unique<ThreadLocalSubqueue>(tid, config_))
742 |              .first;
743 |   }
744 | 
745 |   sub_queue_cache_ = SubQueueThreadCache{id_, it->second.get()};
746 |   return it->second.get();
747 | }
748 | 
749 | void RecordQueue::stop() {
750 |   if (python_tracer_) {
751 |     python_tracer_->stop();
752 |   }
753 | }
754 | 
755 | void RecordQueue::restart() {
756 |   if (python_tracer_) {
757 |     python_tracer_->restart();
758 |   }
759 | }
760 | 
761 | namespace {
762 | void mark_finished(std::shared_ptr<Result>& r) {
763 |   TORCH_INTERNAL_ASSERT(!r->finished_, r->name());
764 |   r->finished_ = true;
765 |   TORCH_INTERNAL_ASSERT(r->endTimeNS() >= r->start_time_ns_, r->name());
766 | }
767 | 
768 | #ifdef USE_KINETO
769 | // Assumption: Total threads number will not exceed 2^16-1, and total ops will
770 | // not exceed 2^48 -1.
771 | static uint64_t getForwardThreadKey(uint64_t tid, uint64_t seqNr) {
772 |   return ((tid << 48) | (seqNr & (((uint64_t)1 << 48) - 1)));
773 | }
774 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `guard`, `mark_finished`, `getForwardThreadKey` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `guard`、`mark_finished`、`getForwardThreadKey` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 775-872
```cpp
775 | void generateForwardBackwardLink(
776 |     const Result& profiler_result,
777 |     uint64_t& fwd_bwd_link_id,
778 |     libkineto::GenericTraceActivity& activity,
779 |     std::unordered_map<uint64_t, libkineto::GenericTraceActivity*>&
780 |         tidSeq2activity) {
781 |   const ExtraFields<EventType::TorchOp>& extra_fields =
782 |       std::get<ExtraFields<EventType::TorchOp>>(profiler_result.extra_fields_);
783 |   if (extra_fields.forward_tid_ > 0) {
784 |     // act is backward op.
785 |     uint64_t key = getForwardThreadKey(
786 |         extra_fields.forward_tid_, extra_fields.sequence_number_);
787 |     auto iter = tidSeq2activity.find(key);
788 |     if (iter != tidSeq2activity.end()) {
789 |       libkineto::GenericTraceActivity* fwd = iter->second;
790 |       fwd->flow.start = true;
791 |       activity.flow.id = fwd->flow.id = fwd_bwd_link_id;
792 |       activity.flow.type = fwd->flow.type = libkineto::kLinkFwdBwd;
793 |       ++fwd_bwd_link_id;
794 | 
795 |       // If there are multiple events that match this sequence/tid pair, we
796 |       // should delete this entry in the map to avoid inserting multiple "end"
797 |       // flow events.
798 |       tidSeq2activity.erase(iter);
799 |     }
800 |   } else if (profiler_result.start_tid_ != 0) {
801 |     // act is forward op.
802 |     uint64_t key = getForwardThreadKey(
803 |         profiler_result.start_tid_, extra_fields.sequence_number_);
804 |     // Assumption: Among all ops with same sequence number,
805 |     // the one with biggest start time is most likely launching backward op.
806 |     auto iter = tidSeq2activity.find(key);
807 |     if (iter == tidSeq2activity.end()) {
808 |       tidSeq2activity[key] = &activity;
809 |     } else {
810 |       // Now the sequence number is only incremented on creating a "Node"
811 |       // object for backward pass, by calling
812 |       // "at::sequence_number::get_and_increment()". Among all ops with same
813 |       // sequence number, the one with biggest startTime is the one launching
814 |       // backward op.
815 |       if (activity.startTime >= iter->second->startTime) {
816 |         tidSeq2activity[key] = &activity;
817 |       }
818 |     }
819 |   }
820 | }
821 | #endif // USE_KINETO
822 | 
823 | void generateForwardBackwardLinks(
824 |     std::unique_ptr<torch::profiler::impl::kineto::trace_t>& cpu_trace,
825 |     const std::vector<std::shared_ptr<Result>>& results) {
826 | #ifndef USE_KINETO
827 | }
828 | #else // USE_KINETO
829 |   TORCH_INTERNAL_ASSERT(cpu_trace->activities.size() == results.size());
830 | 
831 |   // startThreadId_seqNum to pointer of activity.
832 |   // Low-16bits of startThreadId and low-48bits seqNum are concatenated into
833 |   // one uint64_t variable as key.
834 | 
835 |   std::unordered_map<uint64_t, libkineto::GenericTraceActivity*>
836 |       tidSeq2activity;
837 |   uint64_t fwd_bwd_link_id = 1;
838 | 
839 |   using result_activity_t =
840 |       std::pair<Result*, libkineto::GenericTraceActivity*>;
841 |   std::vector<result_activity_t> torch_events;
842 | 
843 |   for (const auto idx : c10::irange(cpu_trace->activities.size())) {
844 |     auto& profiler_result = results[idx];
845 |     auto& activity = cpu_trace->activities[idx];
846 | 
847 |     // add information about an associated forward op, if a sequence number
848 |     // is available (e.g. during training)
849 | 
850 |     profiler_result->visit_if_base<ExtraFields<EventType::TorchOp>>(
851 |         [&](const auto& e) {
852 |           if (e.sequence_number_ >= 0) {
853 |             torch_events.emplace_back(profiler_result.get(), activity.get());
854 |           }
855 |         });
856 |   }
857 | 
858 |   // We need to visit the events in chronological order.
859 |   // So we sort them by end_time_ns_ before processing.
860 |   std::sort(
861 |       torch_events.begin(),
862 |       torch_events.end(),
863 |       [](const result_activity_t& left, const result_activity_t& right) {
864 |         auto left_end_time =
865 |             std::get<ExtraFields<EventType::TorchOp>>(left.first->extra_fields_)
866 |                 .end_time_ns_;
867 |         auto right_end_time = std::get<ExtraFields<EventType::TorchOp>>(
868 |                                   right.first->extra_fields_)
869 |                                   .end_time_ns_;
870 |         return left_end_time < right_end_time;
871 |       });
872 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `generateForwardBackwardLink`, `generateForwardBackwardLinks` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `generateForwardBackwardLink`、`generateForwardBackwardLinks` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 873-963
```cpp
873 |   for (auto& [profiler_result, activity] : torch_events) {
874 |     generateForwardBackwardLink(
875 |         *profiler_result, fwd_bwd_link_id, *activity, tidSeq2activity);
876 |   }
877 | }
878 | #endif // USE_KINETO
879 | 
880 | static constexpr const char* indexKey = "Ev Idx";
881 | 
882 | void passEventsToKineto(
883 |     const std::vector<std::shared_ptr<Result>>& results,
884 |     uint64_t start_time_ns,
885 |     uint64_t end_time_ns,
886 |     const ProfilerConfig& config) {
887 |   using namespace torch::profiler::impl::kineto;
888 |   TraceWrapper cpu_trace(
889 |       static_cast<int64_t>(start_time_ns), "PyTorch Profiler");
890 | 
891 |   // Generate Kineto events for each event recorded by the PyTorch profiler.
892 |   for (const auto i : c10::irange(results.size())) {
893 |     const auto& e = results[i];
894 |     // Here we are essentially setting the duration to -1 if the event never
895 |     // ends. This way Kineto will extend the event to the end of the trace. This
896 |     // is useful so that we can still have 0 duration events if necessary
897 |     // without extension
898 |     int64_t act_end_time = std::max(e->endTimeNS(), e->start_time_ns_ - 1);
899 |     std::string name = e->name();
900 |     if (!e->overload_name().empty()) {
901 |       name = fmt::format("{}.{}", e->name(), e->overload_name());
902 |     }
903 |     auto* activity = cpu_trace.addCPUActivity(
904 |         name,
905 |         e->kinetoType(),
906 |         e->kineto_info_,
907 |         e->correlationID(),
908 |         e->start_time_ns_,
909 |         act_end_time);
910 | 
911 |     TORCH_INTERNAL_ASSERT(activity || !kKinetoAvailable);
912 |     if (activity) {
913 |       addMetadata(activity, indexKey, std::to_string(i));
914 | 
915 |       // There is a longstanding regression for initializing
916 |       // on-demand Kineto activity handling. Enabling this path
917 |       // for Profiler API could cause side effects as much has changed since.
918 |       // Make a surgical fix here until we holistically assess the on-demand
919 |       // vs API path fragmentation, which has been snowballing in complexity
920 |       // and thus flakiness.
921 |       if (config.global()) {
922 |         e->kineto_activity_ = activity;
923 |       }
924 |     }
925 |   }
926 | 
927 |   if (get_fwd_bwd_enabled()) {
928 |     generateForwardBackwardLinks(cpu_trace.get(), results);
929 |   }
930 | 
931 |   // Kineto adds the events that it collected.
932 |   cpu_trace.transferCpuTrace(static_cast<int64_t>(end_time_ns));
933 | }
934 | 
935 | #ifdef USE_KINETO
936 | // There are two mechanisms that we use to connect Profiler and Kineto events.
937 | // The first is the correlation ID. The profiler pushes a unique integer at the
938 | // start of an op and pops it at the end. Kineto then associates the events
939 | // that it collects with that correlation ID and sets the linked activity of
940 | // the events that it collected to point to the profiler op.
941 | //
942 | // However, this is not a sufficient description because it does not retain
943 | // dependency information between kineto ops. Consider a call to `torch.add`.
944 | // Three events will be collected:
945 | //   `aten::add`          (TorchOp, collected by profiler)
946 | //   `cudaLaunchKernel`   (CUDA runtime event, collected by Kineto)
947 | //   `at::vectorized_...` (GPU kernel, collected by Kineto)
948 | // If we only relied on correlation IDs we would set both Kineto events as
949 | // children of the `at::add`, rather than the correct
950 | //   `at::add -> cudaLaunchKernel -> at::vectorized_...`
951 | //
952 | // Kineto surfaces this information through a second concept called a "flow".
953 | // In this example, the `cudaLaunchKernel` event is the start of a flow and the
954 | // GPU kernel has the same flow id but is not a start event. Thus, when merging
955 | // the Kineto events into the call tree we first add all events which are flow
956 | // start nodes. We then merge the rest, trying to pair them with flow starts
957 | // and falling back to correlation ID if necessary. For any nodes without
958 | // linked events the caller is determined using the normal tree construction
959 | // algorithm.
960 | class TransferEvents {
961 |   using itrace_t = libkineto::ITraceActivity;
962 |   using activity_t = torch::profiler::impl::kineto::activity_t;
963 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `TransferEvents` that structure the state handled by this file. Implements routines such as `passEventsToKineto`, `cpu_trace` that expose the key API or control flow of this region.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `TransferEvents` 等数据抽象，用来组织本文件处理的状态。 实现了 `passEventsToKineto`、`cpu_trace` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 964-1066
```cpp
 964 |  public:
 965 |   TransferEvents(
 966 |       std::vector<std::shared_ptr<Result>>& results,
 967 |       trace_ptr_t& trace,
 968 |       const ProfilerConfig& config)
 969 |       : results_{results}, config_{config} {
 970 |     auto* trace_activities_ptr = trace->get()->activities();
 971 |     TORCH_INTERNAL_ASSERT(trace_activities_ptr != nullptr);
 972 |     trace_activities_ = *trace_activities_ptr;
 973 |     reassociate();
 974 |     extractEventsFromTrace();
 975 |     setParents();
 976 |   }
 977 | 
 978 |  private:
 979 |   static long long extractIndex(const std::string& metadata_json) {
 980 |     static const auto prefix = fmt::format("\"{}\": ", indexKey);
 981 |     auto pos = metadata_json.find(prefix);
 982 |     return (pos == std::string::npos) ? unmatchedIndex : [&]() {
 983 |       auto end = metadata_json.find(',', pos);
 984 |       end = (end == std::string::npos) ? metadata_json.size() : end;
 985 |       return std::stoll(metadata_json.substr(pos + prefix.size(), end));
 986 |     }();
 987 |   }
 988 | 
 989 |   std::shared_ptr<Result> lookup(const itrace_t* key) {
 990 |     if (key == nullptr) {
 991 |       return nullptr;
 992 |     }
 993 | 
 994 |     // First check the map.
 995 |     auto it = kineto_events_.find(key);
 996 |     if (it != kineto_events_.end()) {
 997 |       return it->second;
 998 |     }
 999 | 
1000 |     // Then fallback to the encoded metadata.
1001 |     const auto index = extractIndex(key ? key->metadataJson() : "");
1002 |     if (index != unmatchedIndex) {
1003 |       auto out = results_.get().at(index);
1004 |       kineto_events_[key] = out;
1005 |       return out;
1006 |     }
1007 | 
1008 |     // And finally give up.
1009 |     return nullptr;
1010 |   }
1011 | 
1012 |   void reassociate() {
1013 |     // Match profiler events with the corresponding kineto events. Kineto may
1014 |     // have moved or copied the activities, so we have to recover the
1015 |     // relationship between `libkineto::ITraceActivity` and `Result`.
1016 |     for (const auto* activity : trace_activities_) {
1017 |       TORCH_INTERNAL_ASSERT(activity != nullptr);
1018 |       auto e = lookup(activity);
1019 |       if (e != nullptr) {
1020 |         TORCH_INTERNAL_ASSERT(e->kineto_activity_ == nullptr);
1021 |         e->kineto_activity_ = static_cast<const activity_t*>(activity);
1022 |       }
1023 |     }
1024 |     if (results_.get().size() != kineto_events_.size()) {
1025 |       TORCH_WARN(fmt::format(
1026 |           "Failed to recover relationship between all profiler and kineto events: "
1027 |           "{} vs. {}  reassociated.",
1028 |           results_.get().size(),
1029 |           kineto_events_.size()));
1030 |     }
1031 |   }
1032 | 
1033 |   bool isHiddenEvent(const itrace_t* activity) const {
1034 |     TORCH_INTERNAL_ASSERT(activity != nullptr);
1035 |     // Kineto uses "hidden" metadata to mark events that should be hidden.
1036 |     return activity->getMetadataValue("hidden") == "1";
1037 |   }
1038 | 
1039 |   std::shared_ptr<Result> resultFromActivity(const itrace_t* activity) {
1040 |     TORCH_INTERNAL_ASSERT(activity != nullptr);
1041 | 
1042 |     // Kineto is inconsistent with types, so we have to cast to int32.
1043 |     torch::profiler::impl::kineto::DeviceAndResource device_and_resource{
1044 |         static_cast<int32_t>(activity->deviceId()),
1045 |         static_cast<int32_t>(activity->resourceId())};
1046 | 
1047 |     auto event = Result::create(
1048 |         activity->timestamp(),
1049 |         noTID, // Placeholder
1050 |         device_and_resource,
1051 |         ExtraFields<EventType::Kineto>{
1052 |             activity->name(),
1053 |             activity->duration(),
1054 |             static_cast<uint64_t>(activity->correlationId()),
1055 |             activity->type(),
1056 |             {/*id=*/static_cast<uint32_t>(activity->flowId()),
1057 |              /*type=*/static_cast<uint32_t>(activity->flowType()),
1058 |              /*start=*/activity->flowStart()}});
1059 |     event->hidden_ = isHiddenEvent(activity);
1060 |     // NB: It's tempting to set `event->kineto_activity_`; however we can only
1061 |     // guarantee that the events we passed to Kineto are of type
1062 |     // `GenericTraceActivity`. Others may derive from ITraceActivity and thus
1063 |     // are not safe to cast.
1064 |     return event;
1065 |   }
1066 | 
```
- EN: Implements routines such as `extractIndex`, `lookup`, `reassociate`, `isHiddenEvent`, `resultFromActivity` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `extractIndex`、`lookup`、`reassociate`、`isHiddenEvent`、`resultFromActivity` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 1067-1170
```cpp
1067 |   std::shared_ptr<Result> toResult(const itrace_t* activity) {
1068 |     auto e = lookup(activity);
1069 | 
1070 |     // Until we are very sure that we can reassociate kineto and profiler
1071 |     // events we need to be very defensive.
1072 |     const auto type = activity->type();
1073 |     if (e == nullptr &&
1074 |         (type == libkineto::ActivityType::CPU_OP ||
1075 |          type == libkineto::ActivityType::CPU_INSTANT_EVENT ||
1076 |          type == libkineto::ActivityType::USER_ANNOTATION ||
1077 |          type == libkineto::ActivityType::PYTHON_FUNCTION)) {
1078 |       TORCH_WARN_ONCE(
1079 |           "Detected an event which was likely passed to kineto by the PyTorch "
1080 |           "profiler, but is not present in the set of known events: ",
1081 |           activity->name(),
1082 |           " This most likely means that Kineto has not "
1083 |           "maintained address stability for this event. Please report this to "
1084 |           "the PyTorch team.");
1085 |       return nullptr;
1086 |     }
1087 | 
1088 |     if (e == nullptr) {
1089 |       e = resultFromActivity(activity);
1090 |       results_.get().push_back(e);
1091 |       kineto_events_[activity] = e;
1092 |     }
1093 |     return e;
1094 |   }
1095 | 
1096 |   void extractEventsFromTrace() {
1097 |     for (const auto* activity : trace_activities_) {
1098 |       auto e = toResult(activity);
1099 |       if (e) {
1100 |         // Flow data for Kineto events is already set during
1101 |         // resultFromActivity(). TorchOp events need it copied here because
1102 |         // their Result is created during RecordFunction callbacks, before
1103 |         // flow data exists on the GenericTraceActivity.
1104 |         e->visit(c10::overloaded(
1105 |             [&](ExtraFields<EventType::TorchOp>& i) {
1106 |               i.flow = {
1107 |                   /*id=*/static_cast<uint32_t>(activity->flowId()),
1108 |                   /*type=*/static_cast<uint32_t>(activity->flowType()),
1109 |                   /*start=*/activity->flowStart()};
1110 |             },
1111 |             [](auto&) {}));
1112 |         if (config_.experimental_config.expose_kineto_event_metadata) {
1113 |           e->visit(c10::overloaded(
1114 |               [&](ExtraFields<EventType::TorchOp>& i) {
1115 |                 i.metadata_json_ = activity->metadataJson();
1116 |               },
1117 |               [&](ExtraFields<EventType::Kineto>& i) {
1118 |                 i.metadata_json_ = activity->metadataJson();
1119 |               },
1120 |               [](auto&) { return; }));
1121 |           // Parse metadataJson() into extra_meta_ so events() exposes
1122 |           // Kineto metadata as typed fields without export_chrome_trace().
1123 |           // Python schemas (profiler_util.py) are the single SOT for
1124 |           // which keys to expose and how to type-convert them.
1125 |           e->visit(c10::overloaded(
1126 |               [&](ExtraFields<EventType::Kineto>& i) {
1127 |                 auto json_str = activity->metadataJson();
1128 |                 if (!json_str.empty()) {
1129 |                   auto j = nlohmann::json::parse(
1130 |                       "{" + json_str + "}", nullptr, false);
1131 |                   if (!j.is_discarded()) {
1132 |                     for (auto& [key, val] : j.items()) {
1133 |                       i.extra_meta_.emplace(
1134 |                           key,
1135 |                           val.is_string() ? val.get<std::string>()
1136 |                                           : val.dump());
1137 |                     }
1138 |                   }
1139 |                 }
1140 |               },
1141 |               [](auto&) {}));
1142 |         }
1143 |         const auto* linked_activity = activity->linkedActivity();
1144 |         if (linked_activity) {
1145 |           e->visit(c10::overloaded(
1146 |               [&](ExtraFields<EventType::Kineto>& i) {
1147 |                 i.linked_activity_ = toResult(linked_activity);
1148 |               },
1149 |               [](auto&) { TORCH_INTERNAL_ASSERT(false); }));
1150 |         }
1151 |       }
1152 |     }
1153 |   }
1154 | 
1155 |   void setKinetoTID(
1156 |       std::shared_ptr<Result>& r,
1157 |       std::shared_ptr<Result> parent) {
1158 |     r->visit(c10::overloaded(
1159 |         [&]([[maybe_unused]] ExtraFields<EventType::Kineto>& i) {
1160 |           TORCH_INTERNAL_ASSERT(r->start_tid_ == noTID);
1161 |           r->start_tid_ = parent ? parent->start_tid_
1162 |                                  : at::RecordFunction::currentThreadId();
1163 |         },
1164 |         [](auto&) {}));
1165 | 
1166 |     for (auto& child : r->children_) {
1167 |       setKinetoTID(child, r);
1168 |     }
1169 |   }
1170 | 
```
- EN: Implements routines such as `toResult`, `extractEventsFromTrace`, `setKinetoTID` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on.
- CN: 实现了 `toResult`、`extractEventsFromTrace`、`setKinetoTID` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态。

### Lines 1171-1263
```cpp
1171 |   void setParents() {
1172 |     // First pass: Collect start events and set parent to linked event.
1173 |     ska::flat_hash_map<uint32_t, std::shared_ptr<Result>> flow_map;
1174 |     for (auto& e : results_.get()) {
1175 |       TORCH_INTERNAL_ASSERT(e != nullptr);
1176 |       e->visit(c10::overloaded(
1177 |           [&](const ExtraFields<EventType::Kineto>& i) {
1178 |             if (i.flow.type == libkineto::kLinkAsyncCpuGpu && i.flow.start) {
1179 |               auto inserted = flow_map.insert({i.flow.id, e});
1180 | #ifdef USE_ROCM
1181 |               if (inserted.second) {
1182 |                 TORCH_WARN_ONCE(
1183 |                     "ROCTracer produced duplicate flow start: ", i.flow.id);
1184 |               }
1185 | #else // USE_ROCM
1186 |               TORCH_INTERNAL_ASSERT(inserted.second);
1187 | #endif // USE_ROCM
1188 |             }
1189 |             TORCH_INTERNAL_ASSERT(e->parent_.expired());
1190 |             e->parent_ = i.linked_activity_;
1191 |           },
1192 |           [](const auto&) {}));
1193 |     }
1194 | 
1195 |     // Second pass
1196 |     for (auto& e : results_.get()) {
1197 |       e->visit(c10::overloaded(
1198 |           [&](const ExtraFields<EventType::Kineto>& i) {
1199 |             // Flow takes priority over linked event.
1200 |             const auto it = flow_map.find(i.flow.id);
1201 |             if (it != flow_map.end() &&
1202 |                 i.flow.type == libkineto::kLinkAsyncCpuGpu && !i.flow.start) {
1203 |               e->parent_ = it->second;
1204 |             }
1205 | 
1206 |             // If a parent was set we have to do some bookkeeping.
1207 |             auto parent = e->parent_.lock();
1208 |             if (parent) {
1209 |               parent->children_.push_back(e);
1210 |               mark_finished(e);
1211 |             }
1212 |           },
1213 |           [](const auto&) {}));
1214 |     }
1215 | 
1216 |     // Set TIDs now that we have established lineage.
1217 |     for (auto& e : results_.get()) {
1218 |       if (e->parent_.expired()) {
1219 |         setKinetoTID(e, nullptr);
1220 |       }
1221 |     }
1222 |   }
1223 | 
1224 |   static constexpr long long unmatchedIndex = -1;
1225 |   static constexpr auto noTID = std::numeric_limits<uint64_t>::max();
1226 |   std::reference_wrapper<std::vector<std::shared_ptr<Result>>> results_;
1227 |   const ProfilerConfig& config_;
1228 |   std::vector<const itrace_t*> trace_activities_;
1229 |   ska::flat_hash_map<const itrace_t*, std::shared_ptr<Result>> kineto_events_;
1230 | };
1231 | #else
1232 | class TransferEvents {
1233 |  public:
1234 |   template <class... Args>
1235 |   TransferEvents(Args&&... /*unused*/) {}
1236 | };
1237 | #endif
1238 | 
1239 | trace_ptr_t addKinetoEvents(
1240 |     std::vector<std::shared_ptr<Result>>& results,
1241 |     uint64_t start_time_ns,
1242 |     uint64_t end_time_ns,
1243 |     const ProfilerConfig& config) {
1244 |   using namespace torch::profiler::impl::kineto;
1245 |   passEventsToKineto(results, start_time_ns, end_time_ns, config);
1246 | 
1247 |   // In on demand mode kineto is directly controlled by other machinery.
1248 |   if (config.global()) {
1249 |     return nullptr;
1250 |   }
1251 | 
1252 |   auto trace = std::make_unique<ActivityTraceWrapper>(stopTrace());
1253 |   TORCH_INTERNAL_ASSERT(trace || !kKinetoAvailable);
1254 |   TransferEvents transfer{results, trace, config};
1255 |   return trace;
1256 | }
1257 | 
1258 | struct ResultGreater {
1259 |   bool operator()(const result_ptr_t& a, const result_ptr_t& b) const {
1260 |     return a->endTimeNS() > b->endTimeNS();
1261 |   }
1262 | };
1263 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `TransferEvents`, `ResultGreater` that structure the state handled by this file. Implements routines such as `setParents`, `addKinetoEvents`, `operator` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on; stores long-lived member state for later calls.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `TransferEvents`、`ResultGreater` 等数据抽象，用来组织本文件处理的状态。 实现了 `setParents`、`addKinetoEvents`、`operator` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态；保存供后续调用使用的长期成员状态。

### Lines 1264-1363
```cpp
1264 | void set_in_tree_building(
1265 |     std::vector<result_ptr_t>& results,
1266 |     const bool value) {
1267 |   for (result_ptr_t& r : results) {
1268 |     r->visit(c10::overloaded(
1269 |         [value](ExtraFields<EventType::Vulkan>& i) {
1270 |           i.in_tree_building_ = value;
1271 |         },
1272 |         [&](auto&) {
1273 |           // pass
1274 |         }));
1275 |   }
1276 | }
1277 | 
1278 | void build_tree(std::vector<std::shared_ptr<Result>>& sorted_events) {
1279 |   set_in_tree_building(sorted_events, true);
1280 | 
1281 |   using op_fields = ExtraFields<EventType::TorchOp>;
1282 |   ska::flat_hash_map<uint64_t, std::shared_ptr<Result>> stacks;
1283 |   std::priority_queue<result_ptr_t, std::vector<result_ptr_t>, ResultGreater>
1284 |       end_events_;
1285 | 
1286 |   auto push_event = [&stacks, &end_events_](std::shared_ptr<Result>& event) {
1287 |     // Kineto builds subtrees using correlation ids and flows, so some Kineto
1288 |     // events are already marked finished before the main tree building
1289 |     // algorithm. It's fine to ignore them; the root event of these subtrees
1290 |     // not a Kineto op and will be handled normally.
1291 |     if (std::holds_alternative<ExtraFields<EventType::Kineto>>(
1292 |             event->extra_fields_) &&
1293 |         event->finished_) {
1294 |       return;
1295 |     }
1296 | 
1297 |     TORCH_INTERNAL_ASSERT(event->parent_.expired());
1298 |     for (const auto& child : event->children_) {
1299 |       TORCH_INTERNAL_ASSERT(child->finished_);
1300 |     }
1301 |     TORCH_INTERNAL_ASSERT(!event->finished_);
1302 | 
1303 |     auto parent_it = stacks.find(event->start_tid_);
1304 |     if (parent_it == stacks.end()) {
1305 |       auto fwd_tid = event->visit(c10::overloaded(
1306 |           [](const op_fields& i) { return i.forward_tid_; },
1307 |           [](const auto&) -> uint64_t { return 0; }));
1308 |       if (fwd_tid) {
1309 |         parent_it = stacks.find(fwd_tid);
1310 |       }
1311 |     }
1312 | 
1313 |     if (parent_it != stacks.end()) {
1314 |       event->parent_ = parent_it->second;
1315 |       parent_it->second->children_.push_back(event);
1316 |     }
1317 | 
1318 |     if (event->endTimeNS() > event->start_time_ns_) {
1319 |       stacks[event->start_tid_] = event;
1320 |       end_events_.push(event);
1321 |     } else if (event->endTimeNS() == std::numeric_limits<c10::time_t>::min()) {
1322 |       // We use min time to indicate the lack of a termination event, so if we
1323 |       // encounter such a case we don't push to `end_events_`.
1324 |       stacks[event->start_tid_] = event;
1325 |     } else {
1326 |       mark_finished(event);
1327 |     }
1328 |   };
1329 | 
1330 |   auto pop_event = [&stacks](std::shared_ptr<Result> event) {
1331 |     if (event->finished_) {
1332 |       // This event was marked finished by a previous `pop_event` call.
1333 |       return;
1334 |     }
1335 | 
1336 |     auto start_tid = event->start_tid_;
1337 |     auto frame = stacks.at(start_tid);
1338 | 
1339 |     while (frame.get() != event.get()) {
1340 |       TORCH_INTERNAL_ASSERT(frame != nullptr);
1341 |       mark_finished(frame);
1342 |       TORCH_INTERNAL_ASSERT(!frame->parent_.expired());
1343 |       frame = frame->parent_.lock();
1344 |     }
1345 | 
1346 |     mark_finished(event);
1347 |     stacks.erase(start_tid);
1348 |     auto new_frame = event->parent_.lock();
1349 |     if (new_frame != nullptr) {
1350 |       stacks[start_tid] = new_frame;
1351 |     }
1352 |   };
1353 | 
1354 |   // Stack replay loop.
1355 |   for (auto& event : sorted_events) {
1356 |     while (!end_events_.empty() &&
1357 |            end_events_.top()->endTimeNS() < event->start_time_ns_) {
1358 |       pop_event(end_events_.top());
1359 |       end_events_.pop();
1360 |     }
1361 |     push_event(event);
1362 |   }
1363 | 
```
- EN: Implements routines such as `set_in_tree_building`, `build_tree` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `set_in_tree_building`、`build_tree` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 1364-1462
```cpp
1364 |   // Cleanup remaining exit events.
1365 |   while (!end_events_.empty()) {
1366 |     pop_event(end_events_.top());
1367 |     end_events_.pop();
1368 |   }
1369 | 
1370 |   set_in_tree_building(sorted_events, false);
1371 | }
1372 | 
1373 | /**
1374 |  * Adjust r's duration to be the max of its current duration and the sum of all
1375 |  * of its children's adjusted durations (keeping its start time the same)
1376 |  * (adjust all child durations recursively)
1377 |  */
1378 | int64_t adjust_durations_dfs(std::shared_ptr<Result>& r) {
1379 |   if (SOFT_ASSERT(r != nullptr)) {
1380 |     int64_t original_duration = r->endTimeNS() - r->start_time_ns_;
1381 |     int64_t children_total_duration = std::accumulate(
1382 |         r->children_.begin(),
1383 |         r->children_.end(),
1384 |         0,
1385 |         [](int64_t acc, std::shared_ptr<Result>& child) {
1386 |           return acc + adjust_durations_dfs(child);
1387 |         });
1388 | 
1389 |     if (children_total_duration > original_duration) {
1390 |       r->visit(c10::overloaded(
1391 |           [&r, &children_total_duration](ExtraFields<EventType::TorchOp>& i) {
1392 |             i.end_time_ns_ = r->start_time_ns_ + children_total_duration;
1393 |           },
1394 |           [&children_total_duration](ExtraFields<EventType::Vulkan>& i) {
1395 |             i.duration_ns_ = children_total_duration;
1396 |           },
1397 |           []([[maybe_unused]] ExtraFields<EventType::Allocation>& _) {
1398 |             // Pass- Allocation events can't have children
1399 |           },
1400 |           [&](auto&) {
1401 |             SOFT_ASSERT(
1402 |                 false,
1403 |                 "unexpected event type in mobile profiler adjust_durations_dfs: ",
1404 |                 r->name());
1405 |           }));
1406 |       return children_total_duration;
1407 |     } else {
1408 |       return original_duration;
1409 |     }
1410 |   } else {
1411 |     return 0;
1412 |   }
1413 | }
1414 | 
1415 | /**
1416 |  * 1) Adjust r's start time to be [new_start_time] (also adjusting end time and
1417 |       keeping duration the same)
1418 |  * 2) Recursively adjust r's children's start times, making them line up such
1419 |       that the last one ends at the same time as r
1420 |  * 3) Return r's final end time
1421 |  */
1422 | int64_t adjust_timestamps_dfs(
1423 |     std::shared_ptr<Result>& r,
1424 |     int64_t new_start_time) {
1425 |   if (SOFT_ASSERT(r != nullptr)) {
1426 |     if (r->start_time_ns_ != new_start_time) {
1427 |       // Adjust start time (keeping duration constant)
1428 |       r->visit(c10::overloaded(
1429 |           [&r, &new_start_time](ExtraFields<EventType::TorchOp>& i) {
1430 |             i.end_time_ns_ =
1431 |                 new_start_time + (i.end_time_ns_ - r->start_time_ns_);
1432 |           },
1433 |           []([[maybe_unused]] ExtraFields<EventType::Vulkan>& i) {
1434 |             // Pass- We don't need to manually adjust end time for Vulkan events
1435 |           },
1436 |           []([[maybe_unused]] ExtraFields<EventType::Allocation>& _) {
1437 |             // Pass- No duration or end time to adjust
1438 |           },
1439 |           [&](auto&) {
1440 |             SOFT_ASSERT(
1441 |                 false,
1442 |                 "unexpected event type in mobile profiler adjust_timestamps_dfs: ",
1443 |                 r->name());
1444 |           }));
1445 |       r->start_time_ns_ = new_start_time;
1446 |     }
1447 |     int64_t children_total_duration = std::accumulate(
1448 |         r->children_.begin(),
1449 |         r->children_.end(),
1450 |         0,
1451 |         [](int64_t acc, std::shared_ptr<Result>& child) {
1452 |           return acc + (child->endTimeNS() - child->start_time_ns_);
1453 |         });
1454 | 
1455 |     int64_t child_start_time = r->endTimeNS() - children_total_duration;
1456 |     for (std::shared_ptr<Result>& child : r->children_) {
1457 |       child_start_time = adjust_timestamps_dfs(child, child_start_time);
1458 |     }
1459 |   }
1460 |   return r->endTimeNS();
1461 | }
1462 | 
```
- EN: Implements routines such as `adjust_durations_dfs`, `adjust_timestamps_dfs` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `adjust_durations_dfs`、`adjust_timestamps_dfs` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 1463-1550
```cpp
1463 | /**
1464 |  * Adjust timestamps and durations of nodes in [out] such that
1465 |  *  - Vulkan event timelines are synchronized with CPU event times
1466 |  *  - Parent event timelines fully contain their child timelines
1467 |  *  - No overlaps in timelines for nodes at the same depth
1468 |  */
1469 | void adjust_timestamps(std::vector<std::shared_ptr<Result>>& out) {
1470 |   if (out.empty()) {
1471 |     return;
1472 |   }
1473 | 
1474 |   int64_t min_start_time = out[0]->start_time_ns_;
1475 |   for (std::shared_ptr<Result>& r : out) {
1476 |     // Only begin traversal for root nodes.
1477 |     if (r->parent_.expired()) {
1478 |       adjust_durations_dfs(r);
1479 |       min_start_time = adjust_timestamps_dfs(
1480 |           r,
1481 |           std::max(
1482 |               r->tag() != EventType::Vulkan
1483 |                   ? r->start_time_ns_
1484 |                   : std::numeric_limits<int64_t>::min(),
1485 |               min_start_time));
1486 |     }
1487 |   }
1488 | }
1489 | } // namespace
1490 | 
1491 | std::pair<
1492 |     std::vector<std::shared_ptr<Result>>,
1493 |     std::unique_ptr<torch::profiler::impl::kineto::ActivityTraceWrapper>>
1494 | RecordQueue::getRecords(
1495 |     std::function<c10::time_t(c10::approx_time_t)> time_converter,
1496 |     uint64_t start_time_ns,
1497 |     uint64_t end_time_ns) {
1498 |   auto converter = [&](c10::approx_time_t t) {
1499 |     return t == std::numeric_limits<c10::approx_time_t>::min()
1500 |         ? std::numeric_limits<c10::time_t>::min()
1501 |         : time_converter(t);
1502 |   };
1503 | 
1504 |   // Lambda that checks that only the right side of the base intersects with
1505 |   // ev_start and ev_end
1506 |   auto right_intersection_only =
1507 |       [&](ProfilerStepInfo base, int64_t ev_start, int64_t ev_end) {
1508 |         return (base.start_time_ns < ev_start) &&
1509 |             (base.end_time_ns <= ev_end && base.end_time_ns > ev_start);
1510 |       };
1511 |   std::vector<std::shared_ptr<Result>> out;
1512 |   std::vector<python_tracer::CompressedEvent> python_enters;
1513 |   std::vector<ProfilerStepInfo> step_info;
1514 |   long unsigned int step_idx = 0;
1515 |   for (auto& subqueue_it : sub_queues_) {
1516 |     auto& queue = *subqueue_it.second;
1517 |     auto materialize = [&](auto& events) {
1518 |       for (auto& i : events) {
1519 |         c10::time_t start_time_ns = 0;
1520 |         if constexpr (std::is_same_v<
1521 |                           std::remove_reference_t<decltype(i)>,
1522 |                           ExtraFields<EventType::Backend>>) {
1523 |           start_time_ns = i.start_time_us_ * 1000;
1524 |         } else {
1525 |           start_time_ns = converter(i.start_time_);
1526 |         }
1527 |         out.emplace_back(Result::create(
1528 |             /*start_time_ns_=*/start_time_ns,
1529 |             /*start_tid_=*/queue.tid(),
1530 |             /*kineto_info_=*/queue.kineto_info(),
1531 |             /*extra_fields_=*/std::move(i)));
1532 |       }
1533 |       events.clear();
1534 |     };
1535 | 
1536 |     queue.torch_ops_.materialize(
1537 |         out, step_info, converter, queue.tid(), queue.kineto_info());
1538 |     materialize(queue.backend_events_);
1539 |     materialize_vulkan(
1540 |         out, queue.vulkan_events_, converter, queue.tid(), queue.kineto_info());
1541 |     for (auto& i : queue.allocations_) {
1542 |       out.emplace_back(Result::create(
1543 |           /*start_time_ns_=*/converter(i.start_time_),
1544 |           /*start_tid_=*/queue.tid(),
1545 |           /*kineto_info_=*/queue.kineto_info(),
1546 |           /*extra_fields_=*/ExtraFields<EventType::Allocation>(i)));
1547 |     }
1548 |     queue.allocations_.clear();
1549 |     materialize(queue.ooms_);
1550 | 
```
- EN: Implements routines such as `adjust_timestamps` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `adjust_timestamps` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 1551-1653
```cpp
1551 |     std::optional<int64_t> pending_start;
1552 |     for (auto& e : queue.pythongc_) {
1553 |       if (e.first.find("start") != std::string::npos) {
1554 |         pending_start = e.second;
1555 |       } else if (e.first.find("stop") != std::string::npos) {
1556 |         if (pending_start.has_value()) {
1557 |           out.emplace_back(Result::create(
1558 |               /*start_time_ns_=*/converter(pending_start.value()),
1559 |               /*start_tid_=*/queue.tid(),
1560 |               /*kineto_info_=*/queue.kineto_info(),
1561 |               /*extra_fields_=*/
1562 |               // NOLINTNEXTLINE
1563 |               ExtraFields<EventType::PythonGC>{
1564 |                   e.first,
1565 |                   converter(e.second) - converter(pending_start.value())}));
1566 |           pending_start.reset();
1567 |         } else {
1568 |           // Handle the case where "stop" is found without a matching "start"
1569 |           // For example, you might want to log a warning or take other action:
1570 |           LOG(WARNING) << R"("stop" event found without a matching "start": )"
1571 |                        << e.first;
1572 |         }
1573 |       }
1574 |     }
1575 | 
1576 |     for (auto& i : queue.py_calls_) {
1577 |       python_enters.push_back(
1578 |           {i.first, queue.tid(), queue.kineto_info(), converter(i.second)});
1579 |     }
1580 |   }
1581 | 
1582 |   if (python_tracer_) {
1583 |     std::vector<std::shared_ptr<torch::profiler::impl::Result>> ev;
1584 |     try {
1585 |       ev = python_tracer_->getEvents(
1586 |           converter, python_enters, static_cast<c10::time_t>(end_time_ns));
1587 |     } catch (std::exception&) {
1588 |       // Normally addKinetoEvents() below will stop the trace - but if an
1589 |       // exception happens here then the events will never be stopped and future
1590 |       // runs will be broken - so make sure to stopTrace() if we see an
1591 |       // exception.
1592 |       torch::profiler::impl::kineto::stopTrace();
1593 |       throw;
1594 |     }
1595 |     // Placeholder for if we run out of ProfilerStep annotations
1596 |     ProfilerStepInfo defaultStep = {LLONG_MAX, LLONG_MAX, 0};
1597 |     ProfilerStepInfo step =
1598 |         step_idx < step_info.size() ? step_info[step_idx] : defaultStep;
1599 |     for (const auto& i : ev) {
1600 |       // Only adjust timestamps if experimental config is enabled
1601 |       if (config_.experimental_config.adjust_profiler_step) {
1602 |         // If event has start time after step end time we can continue to the
1603 |         // next step
1604 |         while (i->start_time_ns_ > step.end_time_ns) {
1605 |           step_idx++;
1606 |           step =
1607 |               step_idx < step_info.size() ? step_info[step_idx] : defaultStep;
1608 |         }
1609 |         // If Step annotation starts before event and ends before event ends
1610 |         // with intersection then we move the lefthand side of the step
1611 |         // annotation to the event start time
1612 |         if (right_intersection_only(step, i->start_time_ns_, i->endTimeNS())) {
1613 |           // NOLINTNEXTLINE(facebook-hte-LocalUncheckedArrayBounds)
1614 |           auto const& currStepRes = out[step.out_idx];
1615 |           currStepRes->start_time_ns_ = i->start_time_ns_ + 1;
1616 |           step_idx++;
1617 |           step =
1618 |               step_idx < step_info.size() ? step_info[step_idx] : defaultStep;
1619 |         }
1620 |       }
1621 |       out.push_back(i);
1622 |     }
1623 |     python_tracer_.reset();
1624 |   }
1625 | 
1626 |   if (config_.experimental_config.adjust_timestamps) {
1627 |     std::stable_sort(out.begin(), out.end(), [](const auto& a, const auto& b) {
1628 |       return a->start_time_ns_ < b->start_time_ns_;
1629 |     });
1630 |     build_tree(out);
1631 |     adjust_timestamps(out);
1632 |     for (auto& r : out) {
1633 |       r->parent_.reset();
1634 |       // Reset these so that second build_tree can happen
1635 |       r->finished_ = false;
1636 |       r->children_.clear();
1637 |     }
1638 |   }
1639 | 
1640 |   auto trace = addKinetoEvents(out, start_time_ns, end_time_ns, config_);
1641 | 
1642 |   std::stable_sort(out.begin(), out.end(), [](const auto& a, const auto& b) {
1643 |     return a->start_time_ns_ < b->start_time_ns_;
1644 |   });
1645 | 
1646 |   if (config_.report_input_shapes && config_.profile_memory) {
1647 |     calculateUniqueTensorIDs(out);
1648 |   }
1649 | 
1650 |   build_tree(out);
1651 |   return {out, std::move(trace)};
1652 | }
1653 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 1654-1733
```cpp
1654 | namespace {
1655 | std::function<bool()>& record_concrete_inputs_enabled_fn() {
1656 |   static std::function<bool()> fn = []() { return true; };
1657 |   return fn;
1658 | }
1659 | } // namespace
1660 | 
1661 | bool get_record_concrete_inputs_enabled() {
1662 |   return record_concrete_inputs_enabled_fn()();
1663 | }
1664 | 
1665 | void set_record_concrete_inputs_enabled_fn(std::function<bool()> fn) {
1666 |   record_concrete_inputs_enabled_fn() = std::move(fn);
1667 | }
1668 | 
1669 | void set_record_concrete_inputs_enabled_val(bool val) {
1670 |   record_concrete_inputs_enabled_fn() = [val]() { return val; };
1671 | }
1672 | 
1673 | namespace {
1674 | std::function<bool()>& fwd_bwd_enabled_fn() {
1675 |   static std::function<bool()> fn = []() { return true; };
1676 |   return fn;
1677 | }
1678 | } // namespace
1679 | 
1680 | bool get_fwd_bwd_enabled() {
1681 |   return fwd_bwd_enabled_fn()();
1682 | }
1683 | 
1684 | void set_fwd_bwd_enabled_fn(std::function<bool()> fn) {
1685 |   fwd_bwd_enabled_fn() = std::move(fn);
1686 | }
1687 | 
1688 | void set_fwd_bwd_enabled_val(bool val) {
1689 |   fwd_bwd_enabled_fn() = [val]() { return val; };
1690 | }
1691 | 
1692 | namespace {
1693 | std::function<bool()>& cuda_sync_enabled_fn() {
1694 |   static std::function<bool()> fn = []() { return false; };
1695 |   return fn;
1696 | }
1697 | } // namespace
1698 | 
1699 | bool get_cuda_sync_enabled() {
1700 |   return cuda_sync_enabled_fn()();
1701 | }
1702 | 
1703 | void set_cuda_sync_enabled_fn(std::function<bool()> fn) {
1704 |   cuda_sync_enabled_fn() = std::move(fn);
1705 | }
1706 | 
1707 | void set_cuda_sync_enabled_val(bool val) {
1708 |   cuda_sync_enabled_fn() = [val]() { return val; };
1709 | }
1710 | 
1711 | namespace {
1712 | std::function<bool()>& record_tensor_addrs_enabled() {
1713 |   static std::function<bool()> fn = []() { return false; };
1714 |   return fn;
1715 | }
1716 | } // namespace
1717 | 
1718 | bool get_record_tensor_addrs_enabled() {
1719 |   static std::optional<bool> cached_record_tensor_addrs_enabled;
1720 |   if (!cached_record_tensor_addrs_enabled.has_value()) {
1721 |     cached_record_tensor_addrs_enabled = record_tensor_addrs_enabled()();
1722 |   }
1723 |   return cached_record_tensor_addrs_enabled.value();
1724 | }
1725 | 
1726 | void set_record_tensor_addrs_enabled_fn(std::function<bool()> fn) {
1727 |   record_tensor_addrs_enabled() = std::move(fn);
1728 | }
1729 | 
1730 | void set_record_tensor_addrs_enabled_val(bool val) {
1731 |   record_tensor_addrs_enabled() = [val]() { return val; };
1732 | }
1733 | } // namespace torch::profiler::impl
```
- EN: Implements routines such as `get_record_concrete_inputs_enabled`, `record_concrete_inputs_enabled_fn`, `set_record_concrete_inputs_enabled_fn`, `set_record_concrete_inputs_enabled_val`, `get_fwd_bwd_enabled` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `get_record_concrete_inputs_enabled`、`record_concrete_inputs_enabled_fn`、`set_record_concrete_inputs_enabled_fn`、`set_record_concrete_inputs_enabled_val`、`get_fwd_bwd_enabled` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `TransferEvents`, `TagToIOType`, `StealOrDefault`, `SubQueueThreadCache`, `ResultGreater`.
  - CN: `TransferEvents`、`TagToIOType`、`StealOrDefault`、`SubQueueThreadCache`、`ResultGreater`。
- **Important routines / 重要例程**
  - EN: `allTagsMapped`, `tagToIOType`, `push_value`, `getIValueGenerator`, `correlation_id`, `operator`, `materialize_vulkan`, `toString`.
  - CN: `allTagsMapped`、`tagToIOType`、`push_value`、`getIValueGenerator`、`correlation_id`、`operator`、`materialize_vulkan`、`toString`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/orchestration/vulkan.h>`, `<ATen/Context.h>`, `<ATen/record_function.h>`, `<c10/util/Exception.h>`, `<c10/util/flat_hash_map.h>`, `<c10/util/overloaded.h>`, `<torch/csrc/jit/runtime/interpreter.h>`, `<torch/csrc/profiler/data_flow.h>`, `<torch/csrc/profiler/kineto_shim.h>`
- External includes / 外部头文件: `<algorithm>`, `<functional>`, `<limits>`, `<memory>`, `<queue>`, `<type_traits>`, `<utility>`, `<fmt/format.h>`, `<nlohmann/json.hpp>`, `<libkineto.h>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
