# reducer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/reducer.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides Python interoperability logic in the c10d distributed process-group subsystem. Key types include `CpuTimer`, `BucketKey`. It also exposes native functionality to Python.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供Python 互操作逻辑。 关键类型包括 `CpuTimer`、`BucketKey`。 同时它还会向 Python 暴露原生能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行

```cpp
1: #include <torch/csrc/distributed/c10d/reducer.hpp>
2: 
3: #include <torch/csrc/distributed/c10d/Utils.hpp>
4: #include <torch/csrc/distributed/c10d/default_comm_hooks.hpp>
5: 
6: #include <functional>
7: 
8: #include <c10/core/ScalarType.h>
9: #include <c10/util/Exception.h>
10: #include <c10/util/Logging.h>
11: #include <c10/util/hash.h>
12: #include <c10/util/irange.h>
13: #include <torch/csrc/autograd/engine.h>
14: #include <torch/csrc/autograd/function_hook.h>
15: #include <torch/csrc/autograd/utils/grad_layout_contract.h>
16: #include <torch/csrc/autograd/utils/lambda_post_hook.h>
17: #include <torch/csrc/distributed/c10d/comm.hpp>
18: #include <torch/csrc/distributed/c10d/logger.hpp>
19: #include <utility>
20: 
21: namespace c10d {
22: namespace {
23: 
24: constexpr int kUnsetDivFactor = -1;
25: 
26: // Macro that wraps TORCH_CHECK with DDP logging.
27: #define REDUCER_CHECK(cond, logger_, ...)             \
28:   if (C10_UNLIKELY_OR_CONST(!(cond))) {               \
29:     if (!logger_.expired()) {                         \
30:       logger_.lock()->set_error_and_log(__VA_ARGS__); \
```

- EN: Lines 1-30 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1-30 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 31-60 / 第 31-60 行

```cpp
31:     }                                                 \
32:     TORCH_CHECK(false, ##__VA_ARGS__);                \
33:   }
34: 
35: } // namespace
36: 
37: C10_DEFINE_TYPED_REGISTRY(
38:     TimerRegistry,
39:     c10::DeviceType,
40:     Timer,
41:     std::unique_ptr,
42:     c10::Device)
43: 
44: namespace {
45: 
46: class CpuTimer : public Timer {
47:  public:
48:   explicit CpuTimer(c10::Device /* unused */) {}
49: 
50:   std::optional<int64_t> measureDifference(Event start, Event end) override {
51:     int64_t start_time = getTimeRef(start);
52:     int64_t end_time = getTimeRef(end);
53:     // If cpu_end_time is not recorded in this iteration,
54:     // avg_time will return invalid value.
55:     // For some cases like DDP runs on non-sync mode, backward compute
56:     // end time can not be recorded in this iteration and thus can not
57:     // calculate the valid avg_time.
58:     // In this case, skip calculating the avg_time and return.
59:     if (end_time < start_time) {
60:       return std::nullopt;
```

- EN: Lines 31-60 opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `CpuTimer`; introduces executable logic in routines such as `CpuTimer`, `measureDifference`.
- CN: 第 31-60 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `CpuTimer` 等类型；在 `CpuTimer`、`measureDifference` 等例程中引入具体执行逻辑。

### Lines 61-90 / 第 61-90 行

```cpp
61:     }
62:     return end_time - start_time;
63:   }
64: };
65: 
66: C10_REGISTER_TYPED_CLASS(TimerRegistry, c10::kCPU, CpuTimer)
67: 
68: std::vector<at::Tensor> extractTensors(const c10::IValue& result) {
69:   if (result.isPyObject()) {
70:     return result.toPyObjectHolder()->extractTensors();
71:   }
72:   TORCH_INTERNAL_ASSERT(
73:       result.isTensor() || result.isTensorList(),
74:       "expected the hook result is either a Tensor or a TensorList found ",
75:       result.tagKind());
76: 
77:   if (result.isTensor()) {
78:     return {result.toTensor()};
79:   }
80: 
81:   return result.toTensorVector();
82: }
83: 
84: } // namespace
85: 
86: Reducer::Reducer(
87:     std::vector<at::Tensor> params,
88:     std::vector<std::vector<size_t>> bucket_indices,
89:     c10::intrusive_ptr<c10d::ProcessGroup> process_group,
90:     std::vector<bool> expect_sparse_gradients,
```

- EN: Lines 61-90 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `extractTensors`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 61-90 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `extractTensors` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 91-120 / 第 91-120 行

```cpp
91:     int64_t bucket_bytes_cap,
92:     bool find_unused_parameters,
93:     bool gradient_as_bucket_view,
94:     std::unordered_map<size_t, std::string> param_names,
95:     int64_t first_bucket_bytes_cap,
96:     bool skip_all_reduce_unused_params,
97:     bool use_python_reducer,
98:     std::vector<int64_t> bucket_bytes_cap_list,
99:     bool batched_grad_copy)
100:     : params_(std::move(params)),
101:       process_group_(std::move(process_group)),
102:       expect_sparse_gradients_(std::move(expect_sparse_gradients)),
103:       expect_autograd_hooks_(false),
104:       require_finalize_(false),
105:       next_bucket_(0),
106:       has_marked_unused_parameters_(false),
107:       find_unused_parameters_(find_unused_parameters),
108:       gradient_as_bucket_view_(gradient_as_bucket_view),
109:       batched_grad_copy_(batched_grad_copy),
110:       local_used_map_reduced_(false),
111:       num_iterations_(0),
112:       num_bwd_calls_(0),
113:       first_autograd_hook_called_(false),
114:       num_buckets_ready_(0),
115:       num_buckets_reduced_(0),
116:       has_rebuilt_bucket_(false),
117:       bucket_bytes_cap_(bucket_bytes_cap),
118:       div_factor_(kUnsetDivFactor),
119:       static_graph_(false),
120:       skip_all_reduce_unused_params_(skip_all_reduce_unused_params),
```

- EN: Lines 91-120 continues the local implementation details and data flow for this file.
- CN: 第 91-120 行继续展开本文件的局部实现细节与数据流。

### Lines 121-150 / 第 121-150 行

```cpp
121:       comm_hook_(nullptr),
122:       ddp_debug_level_(debug_level()),
123:       param_names_(std::move(param_names)),
124:       first_bucket_bytes_cap_(first_bucket_bytes_cap),
125:       use_python_reducer_(use_python_reducer),
126:       bucket_bytes_cap_list_(std::move(bucket_bytes_cap_list)) {
127:   C10_LOG_API_USAGE_ONCE("torch.distributed.ddp.reducer");
128:   TORCH_INTERNAL_ASSERT(!params_.empty(), "Expected at least one parameter.");
129: 
130:   if (ddp_debug_level_ != c10d::DebugLevel::Off) {
131:     LOG(INFO) << "Reducer initialized with bucket_bytes_cap: "
132:               << bucket_bytes_cap_
133:               << " first_bucket_bytes_cap: " << first_bucket_bytes_cap;
134:   }
135:   // Check whether the module is multi_device_module
136:   {
137:     std::set<int> unique_devices;
138:     for (const auto& v : params_) {
139:       auto device_idx = static_cast<int>(v.device().index());
140:       auto [_, inserted] = unique_devices.emplace(device_idx);
141:       if (inserted) {
142:         if (unique_devices.size() > 1) {
143:           is_multi_device_module_ = true;
144:           break;
145:         }
146:       }
147:     }
148:   }
149: 
150:   // For CUDA, record events only for single device module.
```

- EN: Lines 121-150 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 121-150 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 151-180 / 第 151-180 行

```cpp
151:   c10::Device device = params_[0].device();
152:   if (!(device.is_cuda() && is_multi_device_module_)) {
153:     timer_ = TimerRegistry()->Create(device.type(), device);
154:   }
155: 
156:   // If `expect_sparse_gradients` is not specified, initialize it such that
157:   // we do not expect sparse gradients for any parameter.
158:   if (expect_sparse_gradients_.empty()) {
159:     expect_sparse_gradients_ = std::vector<bool>(params_.size(), false);
160:   }
161:   TORCH_INTERNAL_ASSERT(expect_sparse_gradients_.size() == params_.size());
162: 
163:   // Initialize variable bucketing.
164:   // This can be reinitialized later after capturing runtime information.
165:   {
166:     std::lock_guard<std::mutex> lock(mutex_);
167:     initialize_buckets(std::move(bucket_indices));
168:   }
169: 
170:   // All variables are expected to have their `grad_fn` set to the gradient
171:   // accumulation function (since they are leaves in the autograd graph).
172:   // We store pointers to these functions such that we can check if they are
173:   // used in an autograd pass. If they are not, we know their grad tensors
174:   // can be marked as ready for reduction.
175:   {
176:     const auto variable_count = params_.size();
177:     grad_accumulators_.resize(variable_count);
178:     for (const auto variable_index : c10::irange(variable_count)) {
179:       auto& variable = params_[variable_index];
180: 
```

- EN: Lines 151-180 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 151-180 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 181-210 / 第 181-210 行

```cpp
181:       // The gradient accumulator function is lazily initialized once.
182:       // Therefore we can use its presence in the autograd graph as
183:       // evidence that the parameter has participated in an iteration.
184:       auto grad_accumulator = torch::autograd::impl::grad_accumulator(variable);
185: 
186: #ifndef _WIN32
187:       using torch::distributed::autograd::ThreadLocalDistAutogradContext;
188: #endif
189:       // Hook to execute after the gradient accumulator has executed.
190:       hooks_.emplace_back(
191:           grad_accumulator->add_post_hook(std::make_unique<
192:                                           torch::autograd::utils::
193:                                               LambdaPostHook>(
194:               [this, variable_index](
195:                   const torch::autograd::variable_list& outputs,
196:                   const torch::autograd::variable_list& /* unused */) {
197: #ifndef _WIN32
198:                 this->rpc_context_.set(
199:                     ThreadLocalDistAutogradContext::getContextPtr());
200: #endif
201:                 this->autograd_hook(variable_index);
202:                 return outputs;
203:               },
204:               [this](torch::autograd::CompiledNodeArgs& args) {
205:                 TORCH_CHECK(
206:                     this->use_python_reducer_,
207:                     "Compiled autograd is not compatible with C++ DDP Reducer, please use torch._dynamo.config.optimize_ddp=\"python_reducer\".");
208:               })),
209:           grad_accumulator);
210: 
```

- EN: Lines 181-210 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 181-210 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 211-240 / 第 211-240 行

```cpp
211:       // Map raw function pointer to parameter index.
212:       // This is used later on when the autograd graph is traversed
213:       // to check for parameters for which no gradient is computed, if
214:       // find_unused_parameters=True.
215:       // Note that the mapping of gradient accumulator to variable should be
216:       // one to one as we deduplicate shared parameters before constructing
217:       // Reducer.
218:       if (find_unused_parameters_) {
219:         gradAccToVariableMap_[grad_accumulator.get()] = variable_index;
220:       }
221: 
222:       numGradHooksTriggeredMap_[variable_index] = 0;
223: 
224:       // The gradient accumulator is stored as weak_ptr in the autograd
225:       // metadata of the variable, so we have to keep it alive here for
226:       // the raw pointer to be valid.
227:       REDUCER_CHECK(
228:           grad_accumulators_[variable_index] == nullptr,
229:           logger_,
230:           c10::str(
231:               "Reducer tried to register duplicate grad accumulator for variable ",
232:               variable_index));
233: 
234:       grad_accumulators_[variable_index] = std::move(grad_accumulator);
235:     }
236:   }
237: 
238:   // Initialize backward stats vector.
239:   {
240:     const auto variable_count = params_.size();
```

- EN: Lines 211-240 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 211-240 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 241-270 / 第 241-270 行

```cpp
241:     backward_stats_.resize(variable_count);
242:   }
243: 
244:   // See Note [Skip allreducing local_used_map_dev]
245:   if (find_unused_parameters_) {
246:     initialize_local_used_map();
247:   }
248: }
249: 
250: // Note [Skip allreducing local_used_map_dev]
251: // ~~~~~~~~~~~~~~~~~~~~~~~~~~
252: // If find_unused_parameters_ is set to false, there is no need to allreduce
253: // local_used_map_dev_, because all parameters will be reduced anyway.
254: // Therefore, we can avoid allocating memory for local_used_map and
255: // local_used_map_dev_ if find_unused_parameters_ is false.
256: 
257: // Note [DDP Communication Hook]
258: // ~~~~~~~~~~~~~~~~~~~~~~~~~~
259: // If DDP communication hook is not registered, the reducer reduces the buckets
260: // by just calling allreduce. If registered, it calls the hook and uses future
261: // work handle. If registered, reducer also skips dividing grads by world size.
262: // The reason for this is that the communication hook is expected to completely
263: // override how we perform communication and the user should have complete
264: // control over how the grads are handled.
265: //
266: // DDP communication hook is an enhancement that provides a hook which can be
267: // used to override how DDP communicates gradients across ranks, this can be
268: // used for algorithms like Gradient Compression/GossipGrad. This hook can be
269: // registered from Python API using `register_comm_hook`. `PythonCommHook`
270: // enables registering a Python hook and is a subclass of `CommHookInterface`.
```

- EN: Lines 241-270 introduces executable logic in routines such as `initialize_local_used_map`.
- CN: 第 241-270 行在 `initialize_local_used_map` 等例程中引入具体执行逻辑。

### Lines 271-300 / 第 271-300 行

```cpp
271: // Additionally, there are also some built-in C++ hook implementations that can
272: // be specified by calling `register_builtin_comm_hook` from Python API.
273: 
274: Reducer::~Reducer() noexcept(false) {
275:   remove_autograd_hooks();
276: }
277: 
278: bool Reducer::dynamic_graph_find_unused() {
279:   return !static_graph_ && find_unused_parameters_;
280: }
281: 
282: bool Reducer::static_graph_first_iteration() {
283:   return static_graph_ && num_bwd_calls_ == 1;
284: }
285: 
286: bool Reducer::static_graph_after_first_iteration() {
287:   return static_graph_ && num_bwd_calls_ > 1;
288: }
289: 
290: bool Reducer::ddp_graph_static() {
291:   std::lock_guard<std::mutex> lock(mutex_);
292:   return ddp_graph_static_;
293: }
294: 
295: void Reducer::initialize_local_used_map() {
296:   const auto variable_count = params_.size();
297:   at::TensorOptions options;
298:   options = options.dtype(at::kInt);
299: 
300:   // Deliberately don't pin the memory even if local_used_map_dev_ will
```

- EN: Lines 271-300 introduces executable logic in routines such as `remove_autograd_hooks`; returns computed state or forwards results to the surrounding caller.
- CN: 第 271-300 行在 `remove_autograd_hooks` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 301-330 / 第 301-330 行

```cpp
301:   // be cuda. See Note [local_used_map_ -> local_used_map_dev copying]
302:   local_used_map_ = at::zeros({static_cast<long>(variable_count)}, options);
303: 
304:   // This tensor needs to be on the same device as the replica params because
305:   // backend such as NCCL may not support CPU tensors, and hence it might not
306:   // work if we always put it on CPU. The dist backend for MTIA doesn't support
307:   // int32 allreduce for now, so it has to be placed on CPU.
308:   options = options.device(
309:       (params_[0].is_mtia()) ? c10::Device(c10::DeviceType::CPU)
310:                              : params_[0].device());
311:   local_used_map_dev_ = at::empty({static_cast<long>(variable_count)}, options);
312: }
313: 
314: void Reducer::check_grad_layout(
315:     const at::Tensor& grad,
316:     const at::Tensor& bucket_view) {
317:   // Ensure that the gradient type matches the bucket type, or mixed precision
318:   // type if we are training with mixed precision.
319:   auto type = mixed_precision_param_dtype_
320:       ? *mixed_precision_param_dtype_
321:       : bucket_view.options().dtype().toScalarType();
322:   REDUCER_CHECK(
323:       grad.options().dtype().toScalarType() == type,
324:       logger_,
325:       c10::str(
326:           "Expected ", type, ", got ", grad.options().dtype().toScalarType()));
327: 
328:   TORCH_INTERNAL_ASSERT(grad.device() == bucket_view.device());
329:   TORCH_INTERNAL_ASSERT(grad.numel() == bucket_view.numel());
330:   // AccumulateGrad doesn't HAVE to obey the grad layout contract.
```

- EN: Lines 301-330 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 301-330 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 331-360 / 第 331-360 行

```cpp
331:   // The penalty for disobedience is reduced performance, not numerical
332:   // death. Warnings here help diagnose poor DDP performance.
333:   if (grad.strides() != bucket_view.strides()) {
334:     TORCH_WARN_ONCE(
335:         "Grad strides do not match bucket view strides. "
336:         "This may indicate grad was not created according to the "
337:         "gradient layout contract, or that the param's strides "
338:         "changed since DDP was constructed.  This is not an error, "
339:         "but may impair performance.\n"
340:         "grad.sizes() = ",
341:         grad.sizes(),
342:         ", strides() = ",
343:         grad.strides(),
344:         '\n',
345:         "bucket_view.sizes() = ",
346:         bucket_view.sizes(),
347:         ", strides() = ",
348:         bucket_view.strides());
349:   }
350:   if (!gradient_as_bucket_view_) {
351:     TORCH_INTERNAL_ASSERT(!grad.is_alias_of(bucket_view));
352:   }
353: }
354: 
355: void Reducer::mark_variable_ready_dense(size_t variable_index) {
356:   const auto& bucket_index = variable_locators_[variable_index];
357:   auto& bucket = buckets_[bucket_index.bucket_index];
358:   auto& variable = bucket.variables[bucket_index.intra_bucket_index];
359:   auto& bucket_view = bucket.bucket_views_in[bucket_index.intra_bucket_index];
360: 
```

- EN: Lines 331-360 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 331-360 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 361-390 / 第 361-390 行

```cpp
361:   // Copy the contents of the gradient tensor to the corresponding part of the
362:   // bucket's flattened gradient tensor.
363:   // If the gradient is not set, we assume it wasn't computed as part of the
364:   // current backwards pass, and we zero the part of the bucket it would
365:   // otherwise hold.
366:   runGradCallbackForVariable(variable, [&](auto& grad) {
367:     if (grad.defined()) {
368:       this->check_grad_layout(grad, bucket_view);
369:       // When gradient_as_bucket_view_ is false, or even when
370:       // gradient_as_bucket_view_ is true, in rare cases users may set grad to
371:       // be None after every iteration. In these cases, grad and bucket_view are
372:       // pointing to different storages and thus need to copy grads to
373:       // bucket_view. If gradient_as_bucket_view_ is set as true, let grad point
374:       // to bucket_view. If grad has already been set as views of buckets in
375:       // previous iterations, no copy is needed.
376:       if (!grad.is_alias_of(bucket_view)) {
377:         if (batched_grad_copy_ && !grad.requires_grad()) {
378:           // Defer the copy — will be batched with _foreach_copy_ + flat div_
379:           // when bucket.pending == 0.
380:           bucket.deferred_copy_indices.push_back(
381:               bucket_index.intra_bucket_index);
382:         } else {
383:           if (comm_hook_ == nullptr) {
384:             auto wrapped = at::native::wrapped_scalar_tensor(1. / div_factor_);
385:             if (!grad.requires_grad()) {
386:               // Divides while copying into the bucket view to save one scan
387:               // over all the input parameters.
388:               RECORD_FUNCTION(
389:                   "torch::distributed::reducer::mul_out",
390:                   std::vector<c10::IValue>({bucket_view}))
```

- EN: Lines 361-390 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 361-390 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 391-420 / 第 391-420 行

```cpp
391:               at::mul_out(bucket_view, grad, wrapped);
392:             } else {
393:               // If DDP is running with create_graph=True, gradients
394:               // require_grad themselves in order to compute higher order
395:               // derivatives. However, DDP will not sync up these gradients
396:               // currently (see
397:               // https://github.com/pytorch/pytorch/issues/63812).
398:               C10_LOG_EVERY_N(WARNING, 1000)
399:                   << "Using DistributedDataParallel with create_graph=True "
400:                   << " is not well-supported. The higher-order gradient will "
401:                   << " not be synchronized across ranks, and backpropagation "
402:                   << " through all_reduce operations will not occur. If you require "
403:                   << " DDP to work with higher-order gradients for your use case, "
404:                   << " please ping https://github.com/pytorch/pytorch/issues/63929";
405:               if (batched_grad_copy_) {
406:                 C10_LOG_EVERY_N(WARNING, 1000)
407:                     << "batched_grad_copy is incompatible with "
408:                     << "create_graph=True and has been bypassed.";
409:               }
410:               auto div_result = at::mul(grad, wrapped);
411:               RECORD_FUNCTION(
412:                   "torch::distributed::reducer::copy_",
413:                   std::vector<c10::IValue>({bucket_view}))
414:               bucket_view.copy_(div_result);
415:             }
416:           } else {
417:             RECORD_FUNCTION(
418:                 "torch::distributed::reducer::copy_",
419:                 std::vector<c10::IValue>({bucket_view}))
420:             bucket_view.copy_(grad);
```

- EN: Lines 391-420 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 391-420 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 421-450 / 第 421-450 行

```cpp
421:           }
422: 
423:           if (gradient_as_bucket_view_) {
424:             grad = bucket_view;
425:             return true;
426:           }
427:         }
428:       } else {
429:         // If grad and bucket view point to the same storage, no need to copy.
430:         if (!batched_grad_copy_) {
431:           if (comm_hook_ == nullptr) {
432:             bucket_view.div_(div_factor_);
433:           }
434:         }
435:         // When batched_grad_copy_ is enabled, div_ is deferred to a single
436:         // flat bucket div_ in flush_deferred_copies.
437:       }
438:     } else {
439:       // Gradient is undefined. When find_unused_parameters=True, ensure it is
440:       // not marked as locally used, otherwise we will be allreducing zero's
441:       // instead of not touching .grad field of parameter.
442:       if (this->dynamic_graph_find_unused() ||
443:           this->static_graph_first_iteration()) {
444:         REDUCER_CHECK(
445:             local_used_map_[variable_index].item<int>() == 0,
446:             logger_,
447:             "Encountered gradient which is undefined, but still allreduced by "
448:             "DDP reducer. This indicates a bug in DDP implementation, please "
449:             "report a bug with a repro to PyTorch.");
450:       }
```

- EN: Lines 421-450 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 421-450 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 451-480 / 第 451-480 行

```cpp
451:       bucket_view.zero_();
452:     }
453:     return false;
454:   });
455: }
456: 
457: void Reducer::mark_variable_ready_sparse(size_t variable_index) {
458:   const auto& bucket_index = variable_locators_[variable_index];
459:   auto& bucket = buckets_[bucket_index.bucket_index];
460:   auto& variable = bucket.variables[bucket_index.intra_bucket_index];
461: 
462:   runGradCallbackForVariable(variable, [&](auto& grad) {
463:     REDUCER_CHECK(
464:         grad.defined(), logger_, "Expected sparse gradient to be defined.");
465:     REDUCER_CHECK(
466:         grad.options().layout() == c10::kSparse,
467:         logger_,
468:         "Expected variable to have sparse gradient.");
469: 
470:     // Copy the indices of sparse metadata
471:     if (sparse_metadata_) {
472:       grad = grad.coalesce();
473:       REDUCER_CHECK(
474:           !param_names_.empty(), logger_, "No parameter names were found");
475:       std::string& param_name = param_names_[variable_index];
476:       auto iter = sparse_metadata_->find(param_name);
477:       REDUCER_CHECK(
478:           iter != sparse_metadata_->end(),
479:           logger_,
480:           "param: " + param_name + " not found in sparse metadata");
```

- EN: Lines 451-480 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 451-480 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 481-510 / 第 481-510 行

```cpp
481:       bucket.sparse_tensor_indices =
482:           iter->second.to(at::kLong).unsqueeze(0).to(grad.device());
483:       auto indices = at::searchsorted(
484:           bucket.sparse_tensor_indices.value(), grad.indices(), false, false);
485:       // For indices we are using the ones set by sparse_metadata
486:       grad = at::sparse_coo_tensor(indices, grad.values(), grad.sizes());
487:     }
488: 
489:     // Sparse tensors cannot be grouped together with other sparse tensors in a
490:     // single reduction operation like we can for dense tensors. Therefore, the
491:     // `offsets` and `lengths` vectors in the bucket struct are empty, and
492:     // there is no pre-existing accumulation tensor.
493:     // Directly assign the sparse tensor to the `gradients` field.
494:     bucket.gradients = grad;
495:     // If no DDP comm hook is registered, the allreduce only sums up the
496:     // value, and a separate division is required.
497:     if (comm_hook_ == nullptr) {
498:       bucket.gradients.div_(div_factor_);
499:     }
500:     // The grad is modified in place and needs to be written back.
501:     return true;
502:   });
503: }
504: 
505: std::vector<c10d::GradBucket> Reducer::get_grad_buckets(
506:     bool return_zero_tensors) const {
507:   std::lock_guard<std::mutex> lock(mutex_);
508:   std::vector<c10d::GradBucket> gradBuckets;
509:   gradBuckets.reserve(buckets_.size());
510:   for (const auto i : c10::irange(buckets_.size())) {
```

- EN: Lines 481-510 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 481-510 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 511-540 / 第 511-540 行

```cpp
511:     auto& bucket = buckets_[i];
512:     auto variables_for_bucket = get_variables_for_bucket(i, bucket);
513:     gradBuckets.emplace_back(
514:         i,
515:         buckets_.size(),
516:         return_zero_tensors ? at::zeros_like(bucket.gradients)
517:                             : bucket.gradients,
518:         bucket.offsets,
519:         bucket.lengths,
520:         bucket.sizes_vec,
521:         variables_for_bucket,
522:         std::nullopt);
523:   }
524:   return gradBuckets;
525: }
526: 
527: void Reducer::set_forward_pass_work_handle(
528:     c10::intrusive_ptr<c10d::Work> forwardPassWorkHandle,
529:     bool useStaticWorldSize) {
530:   std::lock_guard<std::mutex> lock(mutex_);
531:   forwardPassWorkHandle_.workHandle = std::move(forwardPassWorkHandle);
532:   forwardPassWorkHandle_.useStaticWorldSize = useStaticWorldSize;
533: }
534: 
535: at::Tensor Reducer::get_local_used_map_on_device() const {
536:   std::lock_guard<std::mutex> lock(mutex_);
537:   return local_used_map_dev_;
538: }
539: 
540: void Reducer::push_rebuilt_params_for_all_indices() {
```

- EN: Lines 511-540 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 511-540 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 541-570 / 第 541-570 行

```cpp
541:   std::lock_guard<std::mutex> lock(mutex_);
542:   if (!should_rebuild_buckets() || !rebuilt_param_indices_.empty()) {
543:     return;
544:   }
545:   const auto variable_count = params_.size();
546:   for (const auto variable_index : c10::irange(variable_count)) {
547:     push_rebuilt_params(variable_index);
548:   }
549: }
550: 
551: void Reducer::push_rebuilt_params(const size_t& index) {
552:   rebuilt_params_.push_back(params_[index]);
553:   rebuilt_param_indices_.push_back(static_cast<int64_t>(index));
554: }
555: 
556: void Reducer::set_divide_factor() {
557:   // If it was scheduled, wait on allreduce in forward pass that tells us
558:   // division factor based on no. of currently participating processes.
559:   if (div_factor_ == kUnsetDivFactor) {
560:     div_factor_ = process_group_->getSize();
561:     auto& workHandle = forwardPassWorkHandle_.workHandle;
562:     if (workHandle && !forwardPassWorkHandle_.useStaticWorldSize) {
563:       workHandle->wait();
564:       // PyProcessGroup::PyWork doesn't expose value, so fetch it from the
565:       // future
566:       auto results = extractTensors(workHandle->getFuture()->value());
567: 
568:       // Guard against the results being empty
569:       TORCH_INTERNAL_ASSERT(!results.empty());
570:       at::Tensor& res = results.front();
```

- EN: Lines 541-570 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 541-570 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 571-600 / 第 571-600 行

```cpp
571:       div_factor_ = res.item().to<int>();
572:     }
573:   }
574: }
575: 
576: // This is called before training and converts the gradients to the dtype they
577: // should be reduced in.
578: void Reducer::set_mixed_precision_param_dtype(c10::ScalarType dtype) {
579:   mixed_precision_param_dtype_ = dtype;
580:   for (auto& bucket : buckets_) {
581:     bucket.gradients = bucket.gradients.to(dtype);
582:   }
583: }
584: 
585: // Right now delay_all_reduce is only called when static_graph_=true and
586: // num_iterations_==1.
587: void Reducer::delay_all_reduce() {
588:   std::lock_guard<std::mutex> lock(this->mutex_);
589: 
590:   if (should_collect_runtime_stats()) {
591:     record_backward_compute_end_time();
592:     record_backward_comm_start_time();
593:   }
594: 
595:   // launch all reduce local used map
596:   all_reduce_local_used_map();
597: 
598:   // prepare to set unused_parameters_, if it is static graph,
599:   // unused_parameters_ will not change after 1st iteration.
600:   unused_parameters_.clear();
```

- EN: Lines 571-600 introduces executable logic in routines such as `lock`, `record_backward_compute_end_time`, `record_backward_comm_start_time`.
- CN: 第 571-600 行在 `lock`、`record_backward_compute_end_time`、`record_backward_comm_start_time` 等例程中引入具体执行逻辑。

### Lines 601-630 / 第 601-630 行

```cpp
601: 
602:   require_finalize_ = true;
603:   // copy all gradients to buckets
604:   for (const auto variable_index : c10::irange(params_.size())) {
605:     // set unused_parameters_
606:     if (numGradHooksTriggeredMap_[variable_index] == 0) {
607:       unused_parameters_.push_back(variable_index);
608:     }
609:     set_divide_factor();
610:     if (expect_sparse_gradients_[variable_index]) {
611:       mark_variable_ready_sparse(variable_index);
612:     } else {
613:       mark_variable_ready_dense(variable_index);
614:     }
615:   }
616: 
617:   // To avoid confusion around why static graph is picking up
618:   // some parameters as unused on a rank vs not, we log
619:   // unused parameter names for each rank for better
620:   // debugability when TORCH_DISTRIBUTED_DEBUG is set to
621:   // INFO or DETAIL
622:   if (ddp_debug_level_ != c10d::DebugLevel::Off) {
623:     // construct one string to output
624:     std::ostringstream unused_params_stream;
625: 
626:     for (const auto& unused_index : unused_parameters_) {
627:       auto param_name = param_names_.find(unused_index);
628:       TORCH_INTERNAL_ASSERT(
629:           param_name != param_names_.end(),
630:           "Expected to find parameter name from unused parameters map in debug mode.");
```

- EN: Lines 601-630 introduces executable logic in routines such as `set_divide_factor`.
- CN: 第 601-630 行在 `set_divide_factor` 等例程中引入具体执行逻辑。

### Lines 631-660 / 第 631-660 行

```cpp
631:       // Add the param_name
632:       unused_params_stream << '{' << param_name->second << ',' << unused_index
633:                            << '}';
634:     }
635: 
636:     // Each rank prints out all the unused parameters detected
637:     if (!unused_parameters_.empty()) {
638:       LOG(INFO) << "[Rank " << process_group_->getRank() << "]: "
639:                 << "Parameter(s) (in the format of {param_name, index}): "
640:                 << unused_params_stream.str()
641:                 << " is(are) unused during first iteration. Since"
642:                 << " static_graph=True is enabled for DDP, we expect"
643:                 << " this set of unused parameters to remain consistent"
644:                 << " on this rank throughout the training.";
645:     }
646:   }
647: 
648:   // launch all reduces for all buckets
649:   for (const auto bucket_index : c10::irange(buckets_.size())) {
650:     auto& bucket = buckets_[bucket_index];
651:     if (batched_grad_copy_) {
652:       flush_deferred_copies(bucket, bucket_index);
653:     }
654:     all_reduce_bucket(bucket);
655:   }
656: 
657:   finalize_backward();
658: }
659: 
660: void Reducer::set_logger(std::weak_ptr<c10d::Logger> logger) {
```

- EN: Lines 631-660 introduces executable logic in routines such as `finalize_backward`.
- CN: 第 631-660 行在 `finalize_backward` 等例程中引入具体执行逻辑。

### Lines 661-690 / 第 661-690 行

```cpp
661:   logger_ = std::move(logger);
662: }
663: 
664: // The function `autograd_hook` is called after the gradient for a
665: // model parameter has been accumulated into its gradient tensor.
666: // This function is only to be called from the autograd thread.
667: void Reducer::autograd_hook(size_t index) {
668:   std::lock_guard<std::mutex> lock(this->mutex_);
669:   if (!first_autograd_hook_called_) {
670:     first_autograd_hook_called_ = true;
671:     num_bwd_calls_++;
672:   }
673: 
674:   // See Note [Skip allreducing local_used_map_dev]
675:   if (dynamic_graph_find_unused() || static_graph_first_iteration()) {
676:     // Since it gets here, this param has been used for this iteration. We want
677:     // to mark it in local_used_map_. During no_sync session, the same var can
678:     // be set multiple times, which is OK as does not affect correctness. As
679:     // long as it is used once during no_sync session, it is marked as used.
680:     // Only set it as locally used if the grad is defined. Otherwise, hooks can
681:     // be fired  with undefined grads, such as when not all outputs are used in
682:     // DDP when computing loss. In this case, we don't want to mark it as
683:     // locally used to ensure we don't touch the parameter's .grad field.
684:     auto& variable = get_param_from_index(index);
685:     runGradCallbackForVariable(variable, [&](auto& grad) {
686:       if (grad.defined()) {
687:         local_used_map_[static_cast<int64_t>(index)] = 1;
688:       }
689:       // The gradient is never modified.
690:       return false;
```

- EN: Lines 661-690 introduces executable logic in routines such as `lock`; returns computed state or forwards results to the surrounding caller.
- CN: 第 661-690 行在 `lock` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 691-720 / 第 691-720 行

```cpp
691:     });
692:   }
693: 
694:   if (static_graph_first_iteration()) {
695:     numGradHooksTriggeredMap_[index] += 1;
696:     return;
697:   }
698: 
699:   // Ignore if we don't expect to be called.
700:   // This may be the case if the user wants to accumulate gradients
701:   // for number of iterations before reducing them.
702:   if (!expect_autograd_hooks_) {
703:     return;
704:   }
705: 
706:   grad_ready_order_indices_.push_back(static_cast<int64_t>(index));
707: 
708:   // If `find_unused_parameters_` is true there may be model parameters that
709:   // went unused when computing the model output, they won't be part of the
710:   // autograd graph, and won't receive gradients. These parameters are
711:   // discovered in the `prepare_for_backward` function and their indexes stored
712:   // in the `unused_parameters_` vector.
713:   if (!has_marked_unused_parameters_) {
714:     has_marked_unused_parameters_ = true;
715:     for (const auto& unused_index : unused_parameters_) {
716:       mark_variable_ready(unused_index);
717:     }
718:   }
719: 
720:   // Rebuild bucket only if 1) it is the first time to rebuild bucket 2)
```

- EN: Lines 691-720 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 691-720 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 721-750 / 第 721-750 行

```cpp
721:   // static_graph_ is true or find_unused_parameters_ is false,
722:   // 3) this backward pass needs to run allreduce.
723:   // Here, we just dump tensors and their parameter indices into
724:   // rebuilt_params_ and rebuilt_param_indices_ based on gradient arriving
725:   // order, and then at the end of finalize_backward(), buckets will be
726:   // rebuilt based on rebuilt_params_ and rebuilt_param_indices_, and then
727:   // will be broadcasted and initialized.
728:   // If it is static graph, after 1st iteration, check if a variable
729:   // is ready for communication based on numGradHooksTriggeredMap_.
730:   if (static_graph_after_first_iteration()) {
731:     REDUCER_CHECK(
732:         numGradHooksTriggeredMapPerIteration_[index] > 0,
733:         logger_,
734:         "Your training graph has changed in this iteration, ",
735:         "e.g., one parameter is unused in first iteration, but ",
736:         "then got used in the second iteration. this is not ",
737:         "compatible with static_graph set to True.");
738:     if (--numGradHooksTriggeredMapPerIteration_[index] == 0) {
739:       if (should_rebuild_buckets()) {
740:         push_rebuilt_params(index);
741:       }
742:       // Finally mark variable for which this function was originally called.
743:       mark_variable_ready(index);
744:     }
745:   } else {
746:     if (should_rebuild_buckets()) {
747:       push_rebuilt_params(index);
748:     }
749:     // Finally mark variable for which this function was originally called.
750:     mark_variable_ready(index);
```

- EN: Lines 721-750 introduces executable logic in routines such as `REDUCER_CHECK`.
- CN: 第 721-750 行在 `REDUCER_CHECK` 等例程中引入具体执行逻辑。

### Lines 751-780 / 第 751-780 行

```cpp
751:   }
752: }
753: 
754: void Reducer::all_reduce_local_used_map() {
755:   // See Note [Skip allreducing local_used_map_dev]
756:   // H2D from local_used_map_ to local_used_map_dev_
757:   if (local_used_map_dev_.is_cuda() || local_used_map_dev_.is_privateuseone()) {
758:     // Note [local_used_map_ -> local_used_map_dev copying]
759:     // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
760:     // We do async H2D to avoid the blocking overhead. The async copy and
761:     // allreduce respect the current stream, so will be sequenced
762:     // correctly.
763:     //
764:     // Correct sequencing with respect to host operations is also
765:     // essential. The H2D copy_ is stream ordered, while the host's
766:     // changes to local_used_map_ are host ordered. If a large backlog of
767:     // cuda/privateuseone-stream work pushes the copy_ far into the future, and
768:     // if no blocking calls occur between now and finalize_backward()** such
769:     // that finalize_backward() re-zeroes local_used_map_ on the host
770:     // before the stream executes the copy_, copy_ will read those zeros
771:     // instead of the values we thought we told it to read here. Copying
772:     // local_used_map_ to a pinned temporary (which the pinned caching
773:     // allocator should supply asynchronously) avoids this nasty, rare
774:     // race condition.
775:     //
776:     // ** In the hoped-for case where all params are used, DDP itself
777:     // won't do any blocking work between now and the re-zeroing, so the
778:     // danger is real.
779:     //
780:     // Defensively ensures local_used_map_tmp is distinct from
```

- EN: Lines 751-780 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 751-780 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 781-810 / 第 781-810 行

```cpp
781:     // local_used_map_
782:     auto local_used_map_tmp = at::native::empty_like(
783:         local_used_map_,
784:         c10::optTypeMetaToScalarType(local_used_map_.options().dtype_opt()),
785:         local_used_map_.options().layout_opt(),
786:         local_used_map_.options().device_opt(),
787:         true /* pinned_memory */);
788:     // Paranoid asserts here because in some workloads, the pinned
789:     // allocator behaves in a way we don't understand, and may be bugged.
790:     // See https://github.com/pytorch/pytorch/pull/54474
791:     TORCH_INTERNAL_ASSERT(local_used_map_tmp.is_pinned());
792:     TORCH_INTERNAL_ASSERT(
793:         local_used_map_tmp.data_ptr() != local_used_map_.data_ptr());
794:     local_used_map_tmp.copy_(local_used_map_);
795:     local_used_map_dev_.copy_(local_used_map_tmp, true);
796:   } else if (local_used_map_dev_.is_mtia()) {
797:     // MTIA probably will have special logic in the future, following code might
798:     // be changed drastically. Therefore, a new if case is created for MTIA, for
799:     // now, the implementation is similar to the CUDA/privateuseone one, except
800:     // for the pin memory step.
801:     auto local_used_map_tmp = at::native::empty_like(
802:         local_used_map_,
803:         c10::optTypeMetaToScalarType(local_used_map_.options().dtype_opt()),
804:         local_used_map_.options().layout_opt(),
805:         local_used_map_.options().device_opt());
806:     local_used_map_tmp.copy_(local_used_map_);
807:     local_used_map_dev_.copy_(local_used_map_tmp, true);
808:   } else {
809:     local_used_map_dev_.copy_(local_used_map_, true);
810:   }
```

- EN: Lines 781-810 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 781-810 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 811-840 / 第 811-840 行

```cpp
811:   std::vector<at::Tensor> temp_local_used_map_dev_vec_ = {local_used_map_dev_};
812:   local_used_work_ = process_group_->allreduce(temp_local_used_map_dev_vec_);
813: }
814: 
815: at::Tensor& Reducer::get_param_from_index(size_t index) {
816:   const auto& bucket_index = variable_locators_[index];
817:   auto& bucket = buckets_[bucket_index.bucket_index];
818:   // Cannot simply access variable via `bucket.variables[variable_index]` since
819:   // return value is used in `runGradCallbackForVariable()` which does not
820:   // accept const tensors.
821:   auto& variable = bucket.variables[bucket_index.intra_bucket_index];
822:   return variable;
823: }
824: 
825: void Reducer::checkAndRaiseMarkedTwiceError(size_t index) {
826:   // Something is wrong if all variables contained in this bucket have
827:   // already been marked as ready.
828:   // We don't expect the same variable to be marked ready twice.
829:   bool marked_twice =
830:       perIterationReadyParams_.find(index) != perIterationReadyParams_.end();
831: 
832:   if (marked_twice) {
833:     // Report index of param that has been marked twice. In debug mode, also
834:     // report fully qualified parameter name.
835:     auto param_name = param_names_.find(index);
836:     const bool found_param_name = param_name != param_names_.end();
837:     TORCH_INTERNAL_ASSERT(
838:         ddp_debug_level_ == c10d::DebugLevel::Off || found_param_name,
839:         "Expected to find parameter name in debug mode.");
840:     std::string paramInfo = c10::str(
```

- EN: Lines 811-840 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`; returns computed state or forwards results to the surrounding caller.
- CN: 第 811-840 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 841-870 / 第 841-870 行

```cpp
841:         "Parameter at index ",
842:         index,
843:         found_param_name ? c10::str(" with name ", param_name->second) : "",
844:         " has been marked as ready twice. This means that multiple autograd engine ",
845:         " hooks have fired for this particular parameter during this iteration.");
846:     // param_names_ is empty in debug mode.
847:     if (!found_param_name) {
848:       paramInfo += c10::str(
849:           " You can set the environment variable TORCH_DISTRIBUTED_DEBUG to either",
850:           " INFO or DETAIL to print parameter names for further debugging.");
851:     }
852:     std::string common_error = c10::str(
853:         "Expected to mark a variable ready only once. ",
854:         "",
855:         "This error is caused by one of the following reasons: ",
856:         "1) Use of a module parameter outside the `forward` function. ",
857:         "Please make sure model parameters are not shared across multiple ",
858:         "concurrent forward-backward passes. or try to use _set_static_graph() ",
859:         "as a workaround if this module graph does not change ",
860:         "during training loop.",
861:         "2) Reused parameters in multiple reentrant backward passes. For ",
862:         "example, if you use multiple `checkpoint` functions to wrap the ",
863:         "same part of your model, it would result in the same set of ",
864:         "parameters been used by different reentrant backward passes ",
865:         "multiple times, and hence marking a variable ready multiple times. ",
866:         "DDP does not support such use cases in default. You can try to ",
867:         "use _set_static_graph() as a workaround if your module graph ",
868:         "does not change over iterations.");
869: 
870:     common_error += c10::str("\n", paramInfo);
```

- EN: Lines 841-870 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 841-870 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 871-900 / 第 871-900 行

```cpp
871: 
872:     REDUCER_CHECK(
873:         has_marked_unused_parameters_,
874:         logger_,
875:         common_error,
876:         "3) Incorrect unused parameter detection. The return value of the ",
877:         "`forward` function is inspected by the distributed data parallel ",
878:         "wrapper to figure out if any of the module's parameters went ",
879:         "unused. For unused parameters, DDP would not expect gradients from ",
880:         "then. However, if an unused parameter becomes part of the autograd ",
881:         "graph at a later point in time (e.g., in a reentrant backward when ",
882:         "using `checkpoint`), the gradient will show up unexpectedly. If all ",
883:         "parameters in the model participate in the backward pass, you can ",
884:         "disable unused parameter detection by passing the keyword argument ",
885:         "`find_unused_parameters=False` to ",
886:         "`torch.nn.parallel.DistributedDataParallel`. If unused parameters ",
887:         "in the model do not change over iterations, You can try to use ",
888:         "_set_static_graph() as a workaround if this module graph does not ",
889:         "change during training loop.");
890:     REDUCER_CHECK(!has_marked_unused_parameters_, logger_, common_error);
891:   }
892: }
893: 
894: void Reducer::mark_variable_ready(size_t variable_index) {
895:   REDUCER_CHECK(
896:       variable_index < variable_locators_.size(),
897:       logger_,
898:       "Out of range variable index.");
899: 
900:   checkAndRaiseMarkedTwiceError(variable_index);
```

- EN: Lines 871-900 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 871-900 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 901-930 / 第 901-930 行

```cpp
901:   perIterationReadyParams_.insert(variable_index);
902:   backward_stats_[variable_index] =
903:       current_time_in_nanos() - backward_compute_start_time_;
904: 
905:   // Any time we mark a variable ready (be it in line due to unused parameters,
906:   // or via an autograd hook), we require a call to the finalize function. If
907:   // this doesn't happen before the next iteration (or call to
908:   // `prepare_for_backwards`), we know something is wrong.
909:   require_finalize_ = true;
910: 
911:   const auto& bucket_index = variable_locators_[variable_index];
912:   auto& bucket = buckets_[bucket_index.bucket_index];
913: 
914:   set_divide_factor();
915: 
916:   if (bucket.expect_sparse_gradient) {
917:     mark_variable_ready_sparse(variable_index);
918:   } else {
919:     mark_variable_ready_dense(variable_index);
920:   }
921: 
922:   // TODO(@pietern): Make this work for both CPU/CUDA tensors.
923:   // When using CPU tensors we don't need to do this.
924:   // Record event so that we can wait for all of them.
925:   // auto& event = bucket.events[bucket_index.intra_bucket_index];
926:   // event.record();
927: 
928:   // Check if this was the final gradient for this bucket.
929:   if (--bucket.pending == 0) {
930:     // When batched_grad_copy_ is enabled, flush deferred copies and perform
```

- EN: Lines 901-930 introduces executable logic in routines such as `set_divide_factor`.
- CN: 第 901-930 行在 `set_divide_factor` 等例程中引入具体执行逻辑。

### Lines 931-960 / 第 931-960 行

```cpp
931:     // a single div on the flat bucket tensor instead of per-variable ops.
932:     if (batched_grad_copy_) {
933:       flush_deferred_copies(bucket, bucket_index.bucket_index);
934:     }
935:     mark_bucket_ready(bucket_index.bucket_index);
936:   }
937: 
938:   // Run finalizer function and kick off reduction for local_used_map once the
939:   // final bucket was marked ready.
940:   if (next_bucket_ == buckets_.size()) {
941:     if (dynamic_graph_find_unused()) {
942:       all_reduce_local_used_map();
943:     }
944: 
945:     torch::autograd::Engine::get_default_engine().queue_callback([this] {
946:       std::lock_guard<std::mutex> lock(this->mutex_);
947:       if (should_collect_runtime_stats()) {
948:         record_backward_compute_end_time();
949:       }
950:       // Check that all buckets were completed and had their work kicked off.
951:       TORCH_INTERNAL_ASSERT(next_bucket_ == buckets_.size());
952:       if (static_graph_after_first_iteration() && should_rebuild_buckets()) {
953:         for (const auto& unused_index : unused_parameters_) {
954:           push_rebuilt_params(unused_index);
955:         }
956:       }
957:       this->finalize_backward();
958:     });
959:   }
960: }
```

- EN: Lines 931-960 introduces executable logic in routines such as `all_reduce_local_used_map`, `lock`, `record_backward_compute_end_time`.
- CN: 第 931-960 行在 `all_reduce_local_used_map`、`lock`、`record_backward_compute_end_time` 等例程中引入具体执行逻辑。

### Lines 961-990 / 第 961-990 行

```cpp
961: 
962: c10::intrusive_ptr<c10::ivalue::Future> Reducer::run_comm_hook(
963:     GradBucket& grad_bucket) {
964:   if (comm_hook_ == nullptr) {
965:     return run_allreduce_hook(grad_bucket);
966:   } else {
967:     return comm_hook_->runHook(grad_bucket);
968:   }
969: }
970: 
971: c10::intrusive_ptr<c10::ivalue::Future> Reducer::run_allreduce_hook(
972:     GradBucket& grad_bucket) {
973:   _AllReduceBySumCommHook allreduce_hook(process_group_);
974:   return allreduce_hook.runHook(grad_bucket);
975: }
976: 
977: void Reducer::all_reduce_bucket(Bucket& bucket) {
978:   auto variables_for_bucket = get_variables_for_bucket(next_bucket_, bucket);
979:   // TODO(@pietern): Ensure proper synchronization with the CUDA events
980:   // that recorded copies into this `gradients` tensor. If these copies are
981:   // executed on non-default streams, the current stream for the device
982:   // that holds the `gradients` tensor must wait on these events.
983:   //
984:   // As long as autograd uses the default stream for every device,
985:   // these operations are implicitly sequenced, and we don't need to
986:   // do any extra synchronization here.
987:   const auto& tensor = bucket.gradients;
988: 
989:   GradBucket grad_bucket(
990:       next_bucket_,
```

- EN: Lines 961-990 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 961-990 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 991-1020 / 第 991-1020 行

```cpp
991:       buckets_.size(),
992:       tensor,
993:       bucket.offsets,
994:       bucket.lengths,
995:       bucket.sizes_vec,
996:       variables_for_bucket,
997:       bucket.sparse_tensor_indices);
998:   bucket.future_work = run_comm_hook(grad_bucket);
999: }
1000: 
1001: std::vector<at::Tensor> Reducer::get_variables_for_bucket(
1002:     size_t bucket_index,
1003:     const Bucket& bucket) const {
1004:   // Check if we have cached mapping previously.
1005:   if (has_rebuilt_bucket_ &&
1006:       cached_variables_for_bucket_.find(bucket_index) !=
1007:           cached_variables_for_bucket_.end()) {
1008:     return cached_variables_for_bucket_[bucket_index];
1009:   }
1010:   std::vector<at::Tensor> variables_for_bucket;
1011:   variables_for_bucket.reserve(bucket.variable_indices.size());
1012:   for (const auto& variable_index : bucket.variable_indices) {
1013:     // Grab bucket index where gradient is located using variable_locators_.
1014:     auto& bucket_index_for_variable = variable_locators_[variable_index];
1015:     // Grab the actual model parameter.
1016:     auto& variable =
1017:         bucket.variables[bucket_index_for_variable.intra_bucket_index];
1018:     variables_for_bucket.emplace_back(variable);
1019:   }
1020: 
```

- EN: Lines 991-1020 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 991-1020 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1021-1050 / 第 1021-1050 行

```cpp
1021:   if (has_rebuilt_bucket_) {
1022:     TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
1023:         cached_variables_for_bucket_.find(bucket_index) ==
1024:         cached_variables_for_bucket_.end());
1025:     cached_variables_for_bucket_.insert(
1026:         {bucket_index, std::move(variables_for_bucket)});
1027:     return cached_variables_for_bucket_[bucket_index];
1028:   } else {
1029:     return variables_for_bucket;
1030:   }
1031: }
1032: 
1033: bool Reducer::is_unused_bucket(Bucket& bucket) {
1034:   for (const auto& variable_index : bucket.variable_indices) {
1035:     if (std::find(
1036:             unused_parameters_.begin(),
1037:             unused_parameters_.end(),
1038:             variable_index) == unused_parameters_.end()) {
1039:       return false;
1040:     }
1041:   }
1042:   return true;
1043: }
1044: 
1045: bool Reducer::should_skip_all_reduce_bucket(Bucket& bucket) {
1046:   return is_unused_bucket(bucket) && skip_all_reduce_unused_params_;
1047: }
1048: 
1049: // Called when the bucket at the specified index is ready to be reduced.
1050: void Reducer::mark_bucket_ready(size_t bucket_index) {
```

- EN: Lines 1021-1050 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1021-1050 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1051-1080 / 第 1051-1080 行

```cpp
1051:   TORCH_INTERNAL_ASSERT(bucket_index >= next_bucket_);
1052: 
1053:   // Buckets are reduced in sequence. Ignore this bucket if
1054:   // it's not its turn to be reduced.
1055:   if (bucket_index > next_bucket_) {
1056:     return;
1057:   }
1058: 
1059:   // Keep going, until we either:
1060:   // - have kicked off reduction for all buckets, or
1061:   // - found a bucket that's not yet ready for reduction.
1062:   for (; next_bucket_ < buckets_.size() && buckets_[next_bucket_].pending == 0;
1063:        next_bucket_++) {
1064:     num_buckets_ready_++;
1065:     if (num_buckets_ready_ == 1 && should_collect_runtime_stats()) {
1066:       record_backward_comm_start_time();
1067:     }
1068:     auto& bucket = buckets_[next_bucket_];
1069:     if (!should_skip_all_reduce_bucket(bucket)) {
1070:       all_reduce_bucket(bucket);
1071:       num_buckets_reduced_++;
1072:     }
1073:   }
1074: }
1075: 
1076: void Reducer::install_futures(
1077:     const c10::List<c10::intrusive_ptr<c10::ivalue::Future>>& futs) {
1078:   // Append instead of overwrite so that this method can be called multiple
1079:   // times in one iteration.
1080:   if (!installed_futures_) {
```

- EN: Lines 1051-1080 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`, `record_backward_comm_start_time`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1051-1080 行在 `TORCH_INTERNAL_ASSERT`、`record_backward_comm_start_time` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1081-1110 / 第 1081-1110 行

```cpp
1081:     installed_futures_ = futs;
1082:   } else {
1083:     installed_futures_->append(futs);
1084:   }
1085: }
1086: 
1087: void Reducer::initialize_buckets(
1088:     std::vector<std::vector<size_t>> bucket_indices) {
1089:   // If initialize_buckets is called inside DDP constructor, then
1090:   // it does not matter rpc context ptr is nullptr or not, as grad
1091:   // will not be mutated.
1092:   // If initialize_buckets is called during training loop, e.g, inside
1093:   // rebuild_buckets(), since grad could be mutated and be pointed to
1094:   // bucket_view, then it needs to check rpc context ptr is nullptr or not,
1095:   // If rpc context ptr is nullptr, mutate variable.grad(); otherwise,
1096:   // mutate grad in rpc context.
1097: #ifndef _WIN32
1098:   using torch::distributed::autograd::ThreadLocalDistAutogradContext;
1099:   this->rpc_context_.set(ThreadLocalDistAutogradContext::getContextPtr());
1100: #endif
1101: 
1102:   // This shouldn't be called if we're expecting autograd hooks to fire.
1103:   REDUCER_CHECK(
1104:       !expect_autograd_hooks_,
1105:       logger_,
1106:       "`initialize_buckets` must NOT be called during autograd execution.");
1107: 
1108:   // Clear current bucket assignment.
1109:   buckets_.clear();
1110:   variable_locators_.clear();
```

- EN: Lines 1081-1110 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `REDUCER_CHECK`.
- CN: 第 1081-1110 行使用条件编译来适配特性开关、平台或可选后端；在 `REDUCER_CHECK` 等例程中引入具体执行逻辑。

### Lines 1111-1140 / 第 1111-1140 行

```cpp
1111: 
1112:   // Ensure we have a bucket index for every variable.
1113:   variable_locators_.resize(params_.size());
1114: 
1115:   // Iterate over buckets.
1116:   const auto bucket_count = bucket_indices.size();
1117:   buckets_.reserve(bucket_count);
1118:   for (const auto bucket_index : c10::irange(bucket_count)) {
1119:     Bucket bucket;
1120: 
1121:     // TODO(@pietern): Validate indices.
1122:     // Must be non-empty, unique, and unique across buckets.
1123:     REDUCER_CHECK(
1124:         !bucket_indices[bucket_index].empty(),
1125:         logger_,
1126:         "Empty bucket specified.");
1127: 
1128:     // Variables that expect sparse gradients must have their own bucket.
1129:     if (bucket_indices[bucket_index].size() == 1) {
1130:       const auto variable_index = bucket_indices[bucket_index].front();
1131:       bucket.expect_sparse_gradient = expect_sparse_gradients_[variable_index];
1132:     } else {
1133:       for (const auto variable_index : bucket_indices[bucket_index]) {
1134:         REDUCER_CHECK(
1135:             !expect_sparse_gradients_[variable_index],
1136:             logger_,
1137:             "Buckets with more than one variable cannot include variables ",
1138:             "that expect a sparse gradient.");
1139:       }
1140:     }
```

- EN: Lines 1111-1140 introduces executable logic in routines such as `REDUCER_CHECK`.
- CN: 第 1111-1140 行在 `REDUCER_CHECK` 等例程中引入具体执行逻辑。

### Lines 1141-1170 / 第 1141-1170 行

```cpp
1141: 
1142:     if (bucket.expect_sparse_gradient) {
1143:       const auto variable_index = bucket_indices[bucket_index].front();
1144:       const auto& variable = params_[variable_index];
1145:       TORCH_INTERNAL_ASSERT(bucket_indices[bucket_index].size() == 1);
1146:       bucket.variables = {variable};
1147:     } else {
1148:       at::TensorOptions options;
1149:       // The start index of the variable in the flattened tensor.
1150:       size_t offset = 0;
1151: 
1152:       // Reserve enough space for the per-variable fields stored in the bucket
1153:       // for efficiency.
1154:       const size_t num_variables = bucket_indices[bucket_index].size();
1155:       bucket.variables.reserve(num_variables);
1156:       bucket.offsets.reserve(num_variables);
1157:       bucket.lengths.reserve(num_variables);
1158:       bucket.sizes_vec.reserve(num_variables);
1159: 
1160:       // Iterate over bucket variables.
1161:       for (const auto variable_index : bucket_indices[bucket_index]) {
1162:         TORCH_INTERNAL_ASSERT(
1163:             variable_index < params_.size(),
1164:             "Out of range variable index specified.");
1165:         const auto& variable = params_[variable_index];
1166:         if (!options.has_device()) {
1167:           options = options.device(variable.device());
1168:         } else {
1169:           REDUCER_CHECK(
1170:               variable.device() == options.device(),
```

- EN: Lines 1141-1170 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1141-1170 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1171-1200 / 第 1171-1200 行

```cpp
1171:               logger_,
1172:               "All parameters in a bucket must be ",
1173:               "placed on the same device.");
1174:         }
1175:         if (!options.has_dtype()) {
1176:           options = options.dtype(variable.dtype());
1177:           if (variable.is_complex()) {
1178:             bucket.is_complex_bucket = true;
1179:           }
1180:         } else {
1181:           REDUCER_CHECK(
1182:               variable.dtype() == options.dtype(),
1183:               logger_,
1184:               "All parameters in a bucket must have the same dtype.");
1185:         }
1186:         const auto length = variable.numel();
1187:         bucket.variables.push_back(variable);
1188:         bucket.offsets.push_back(offset);
1189:         bucket.lengths.push_back(length);
1190:         bucket.sizes_vec.push_back(variable.sizes());
1191:         offset += length;
1192:       }
1193: 
1194:       // Make gradient type in the reduced precision if mixed precision is
1195:       // enabled. This ensures that the type is correct when e.g. rebuilding
1196:       // buckets.
1197:       if (mixed_precision_param_dtype_.has_value()) {
1198:         options = options.dtype(mixed_precision_param_dtype_);
1199:       }
1200: 
```

- EN: Lines 1171-1200 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1171-1200 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1201-1230 / 第 1201-1230 行

```cpp
1201:       // Allocate the bucket's flattened `gradients` tensor.
1202:       auto bucketSize = static_cast<long>(offset);
1203:       // Check if we can use comm-optimized memory pool to allocate tensor
1204:       c10::intrusive_ptr<Backend> backend = nullptr;
1205:       // An environment variable to disable comm-optimized memory pool.
1206:       // Default is 1 for now (disabled).
1207:       // TODO: turn it on by default once we have more confidence on it.
1208:       bool ddpDisableCommMem =
1209:           (getCvarString({"DDP_DISABLE_COMM_MEM"}, "1") == "1");
1210:       try {
1211:         backend = process_group_->getDefaultBackend();
1212:       } catch (...) {
1213:         // Sometimes the backend type can be `UNDEFINED` rather than `NCCL` or
1214:         // `GLOO`. In this case, we just fall back to the regular way of
1215:         // creating tensor
1216:         LOG(INFO)
1217:             << "Reducer: default comm backend not found, skipping bucket memory optimization";
1218:       }
1219:       if (ddpDisableCommMem == 0 && backend != nullptr &&
1220:           backend->supportsTensorAlloc(options.device().index())) {
1221:         // Comm-optimized memory pool is available, use it to allocate tensor
1222:         LOG(INFO)
1223:             << "Reducer: found comm-optimized memory allocator, using it to create bucket";
1224:         bucket.gradients = backend->allocateTensor(bucketSize, options);
1225:       } else {
1226:         // Plain creation of tensor
1227:         LOG(INFO)
1228:             << "Reducer: comm-optimized memory allocator not found, using regular one";
1229:         bucket.gradients = at::empty({bucketSize}, options);
1230: 
```

- EN: Lines 1201-1230 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1201-1230 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1231-1260 / 第 1231-1260 行

```cpp
1231:         if (bucket.is_complex_bucket) {
1232:           bucket.gradients = at::view_as_real(bucket.gradients).reshape({-1});
1233:         }
1234:       }
1235: 
1236:       // Note:  "Gradient Layout Contract"
1237:       //
1238:       // Here, create views into the `gradients` tensor for each variable's
1239:       // grad. Views serve as entry points to `copy_()` each grad's data in/out
1240:       // of the flattened `gradients` tensor.
1241:       //
1242:       // Gradients may have dense memory but non-row-major-contiguous strides
1243:       // (e.g. channels_last or channels_last_3d). For coalesced accesses
1244:       // during copy_s, it's beneficial for each view's layout to match its
1245:       // grad's layout.
1246:       //
1247:       // Specifically, we expect torch/csrc/autograd/functions/accumulate_grad.h
1248:       // produces grads that obey the "Gradient Layout Contract":
1249:       //   (1) if variable.is_non_overlapping_and_dense(), the stashed grad's
1250:       //       strides match variable.
1251:       //   (2) else, stashed grad is rowmajor contiguous.
1252:       // and create views to match.
1253:       //
1254:       // If AccumulateGrad breaks the contract, and produces a grad with an
1255:       // unexpected layout, performance will degrade due to poor memory access
1256:       // patterns when copy_ing grad data in and out of its bucket view.
1257:       // However, numerics remain correct, because the bucket view is the same
1258:       // on either end of the raw allreduce.  bucket_view_in.copy(grad)
1259:       // transposes
1260:       // (+ densifies) to the bucket view's layout, the data is allreduced,
```

- EN: Lines 1231-1260 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1231-1260 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1261-1290 / 第 1261-1290 行

```cpp
1261:       // then grad.copy_(bucket_view_out) transposes it back to grad's layout.
1262:       //
1263:       // The only way the numerics can go haywire is if the bucket views
1264:       // themselves have different layouts across processes.
1265:       // Bucket views' sizes and strides are set based on param layouts, using
1266:       // the same logic that (we expect) AccumulateGrad uses for their grads.
1267:       // Therefore, the only way a bucket view could have different layouts in
1268:       // different processes is if its param has a different layout in
1269:       // different processes. We can check that param layouts match across
1270:       // processes in Reducer's constructor by allreducing some metadata.
1271:       // Checking just once won't catch if someone messes with
1272:       // param layouts over time, but not messing with params after DDP
1273:       // construction is already a documented constraint.
1274:       initialize_bucket_views(bucket);
1275:     }
1276: 
1277:     // Map participating variables to this bucket.
1278:     size_t intra_bucket_index = 0;
1279:     for (const auto variable_index : bucket_indices[bucket_index]) {
1280:       TORCH_INTERNAL_ASSERT(
1281:           variable_index < variable_locators_.size(),
1282:           "Out of range variable index specified.");
1283:       variable_locators_[variable_index] =
1284:           VariableLocator(bucket_index, intra_bucket_index++);
1285:     }
1286:     bucket.variable_indices = std::move(bucket_indices[bucket_index]);
1287: 
1288:     buckets_.push_back(std::move(bucket));
1289:   }
1290: }
```

- EN: Lines 1261-1290 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1261-1290 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1291-1320 / 第 1291-1320 行

```cpp
1291: 
1292: // (see Note:  "Gradient Layout Contract" in initialize_buckets).
1293: void Reducer::initialize_bucket_views(Reducer::Bucket& bucket) {
1294:   const auto& gradients = bucket.gradients;
1295:   for (const auto i : c10::irange(bucket.variables.size())) {
1296:     auto& v = bucket.variables[i];
1297:     const auto offset = bucket.offsets[i];
1298:     const auto length = bucket.lengths[i];
1299: 
1300:     if (v.is_complex() && bucket.is_complex_bucket) {
1301:       const auto real_offset = offset * 2;
1302:       const auto real_length = length * 2;
1303: 
1304:       if (v.is_non_overlapping_and_dense()) {
1305:         auto complex_strides = v.strides();
1306:         std::vector<int64_t> real_strides;
1307:         real_strides.reserve(complex_strides.size() + 1);
1308:         for (auto s : complex_strides) {
1309:           real_strides.push_back(s * 2);
1310:         }
1311:         real_strides.push_back(1);
1312: 
1313:         auto complex_sizes = v.sizes();
1314:         std::vector<int64_t> real_sizes(
1315:             complex_sizes.begin(), complex_sizes.end());
1316:         real_sizes.push_back(2);
1317: 
1318:         auto real_view =
1319:             gradients.as_strided(real_sizes, real_strides, real_offset);
1320:         auto complex_view = at::view_as_complex(real_view);
```

- EN: Lines 1291-1320 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1291-1320 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1321-1350 / 第 1321-1350 行

```cpp
1321:         bucket.bucket_views_in.push_back(complex_view);
1322:       } else {
1323:         auto real_view = gradients.narrow(
1324:             0,
1325:             static_cast<int64_t>(real_offset),
1326:             static_cast<int64_t>(real_length));
1327:         auto complex_view = at::view_as_complex(
1328:             real_view.reshape({static_cast<int64_t>(length), 2}));
1329:         bucket.bucket_views_in.push_back(complex_view.view(v.sizes()));
1330:       }
1331:     } else {
1332:       if (v.is_non_overlapping_and_dense()) {
1333:         // If the param's memory is dense, match its layout, anticipating
1334:         // the autograd engine (AccumulateGrad) will also create gradients
1335:         // matching its layout.
1336:         bucket.bucket_views_in.push_back(
1337:             gradients.as_strided(v.sizes(), v.strides(), offset));
1338:       } else {
1339:         // Fall back to a C-style contiguous view, again anticipating
1340:         // AccumulateGrad will do the same when stashing grads for non-dense
1341:         // params.
1342:         bucket.bucket_views_in.push_back(gradients
1343:                                              .narrow(
1344:                                                  0,
1345:                                                  static_cast<int64_t>(offset),
1346:                                                  static_cast<int64_t>(length))
1347:                                              .view(v.sizes()));
1348:       }
1349:     }
1350:     // By default `bucket_views_out` and `bucket_views_in` are
```

- EN: Lines 1321-1350 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1321-1350 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1351-1380 / 第 1351-1380 行

```cpp
1351:     // essentially the same thing.
1352:     bucket.bucket_views_out = bucket.bucket_views_in;
1353: 
1354:     // If gradient_as_bucket_view_ is set as true, then there are two cases to
1355:     // handle: initialize_bucket_views could be called inside initialize_buckets
1356:     // when rebuild_buckets, if grad has already been defined/calculated in
1357:     // previous iteration, old grad needs to be copied into new bucket_view and
1358:     // let grad point to the new bucket_view, initialize_bucket_views could also
1359:     // be called inside initialize_buckets during construction. Grads are not
1360:     // defined during construction time, in this case, do not let grad point to
1361:     // bucket_view, because grads should be kept as being undefined for globally
1362:     // unused parameters.
1363:     if (gradient_as_bucket_view_) {
1364:       auto& bucket_view = bucket.bucket_views_in.back();
1365:       runGradCallbackForVariable(v, [&](auto& grad) {
1366:         if (grad.defined() && !grad.is_alias_of(bucket_view)) {
1367:           bucket_view.copy_(grad);
1368:           grad = bucket_view;
1369:           // The grad is modified and needs to be written back.
1370:           return true;
1371:         }
1372:         // The grad is not modified and does not need to be written back.
1373:         return false;
1374:       });
1375:     }
1376:   }
1377: }
1378: 
1379: // (see Note:  "Gradient Layout Contract" in initialize_buckets).
1380: void Reducer::populate_bucket_views_out(
```

- EN: Lines 1351-1380 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1351-1380 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1381-1410 / 第 1381-1410 行

```cpp
1381:     Reducer::Bucket& bucket,
1382:     at::Tensor& tensor) {
1383:   bucket.bucket_views_out.clear();
1384:   for (const auto i : c10::irange(bucket.variables.size())) {
1385:     const auto& v = bucket.variables[i];
1386:     const auto offset = bucket.offsets[i];
1387:     const auto length = bucket.lengths[i];
1388: 
1389:     if (v.is_complex() && bucket.is_complex_bucket) {
1390:       const auto real_offset = offset * 2;
1391: 
1392:       if (v.is_non_overlapping_and_dense()) {
1393:         auto complex_strides = v.strides();
1394:         std::vector<int64_t> real_strides;
1395:         real_strides.reserve(complex_strides.size() + 1);
1396:         for (auto s : complex_strides) {
1397:           real_strides.push_back(s * 2);
1398:         }
1399:         real_strides.push_back(1);
1400: 
1401:         auto complex_sizes = v.sizes();
1402:         std::vector<int64_t> real_sizes(
1403:             complex_sizes.begin(), complex_sizes.end());
1404:         real_sizes.push_back(2);
1405: 
1406:         auto real_view =
1407:             tensor.as_strided(real_sizes, real_strides, real_offset);
1408:         bucket.bucket_views_out.push_back(at::view_as_complex(real_view));
1409:       } else {
1410:         const auto real_length = length * 2;
```

- EN: Lines 1381-1410 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1381-1410 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1411-1440 / 第 1411-1440 行

```cpp
1411:         auto real_view = tensor.narrow(
1412:             0,
1413:             static_cast<int64_t>(real_offset),
1414:             static_cast<int64_t>(real_length));
1415:         auto complex_view = at::view_as_complex(
1416:             real_view.reshape({static_cast<int64_t>(length), 2}));
1417:         bucket.bucket_views_out.push_back(complex_view.view(v.sizes()));
1418:       }
1419:     } else {
1420:       if (v.is_non_overlapping_and_dense()) {
1421:         // If the param's memory is dense, match its layout, anticipating
1422:         // the autograd engine (AccumulateGrad) will also create gradients
1423:         // matching its layout.
1424:         bucket.bucket_views_out.push_back(
1425:             tensor.as_strided(v.sizes(), v.strides(), offset));
1426:       } else {
1427:         // Fall back to a C-style contiguous view, again anticipating
1428:         // AccumulateGrad will do the same when stashing grads for non-dense
1429:         // params.
1430:         bucket.bucket_views_out.push_back(tensor
1431:                                               .narrow(
1432:                                                   0,
1433:                                                   static_cast<int64_t>(offset),
1434:                                                   static_cast<int64_t>(length))
1435:                                               .view(v.sizes()));
1436:       }
1437:     }
1438:   }
1439: }
1440: 
```

- EN: Lines 1411-1440 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1411-1440 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1441-1470 / 第 1441-1470 行

```cpp
1441: void Reducer::prepare_for_forward() {
1442:   std::lock_guard<std::mutex> lock(mutex_);
1443:   num_iterations_++;
1444:   if (should_collect_runtime_stats()) {
1445:     record_forward_compute_start_time();
1446:   }
1447: }
1448: 
1449: void Reducer::reset_bucket_counting() {
1450:   next_bucket_ = 0;
1451:   // Reset num_buckets_ready_ at the beginning of backward computation
1452:   // in each iteration.
1453:   num_buckets_ready_ = 0;
1454: 
1455:   num_buckets_reduced_ = 0;
1456: 
1457:   for (auto& bucket : buckets_) {
1458:     bucket.pending = bucket.variables.size();
1459:     bucket.deferred_copy_indices.clear();
1460:   }
1461: 
1462:   if (static_graph_) {
1463:     numGradHooksTriggeredMapPerIteration_ = numGradHooksTriggeredMap_;
1464:   }
1465: }
1466: 
1467: // Traverse the autograd graph starting at the specified output.
1468: // All parameters for which we have a pointer to their gradient accumulation
1469: // functions, but don't show up in the autograd graph will be marked ready for
1470: // for reduction as soon as the first autograd hook is called. This is not
```

- EN: Lines 1441-1470 introduces executable logic in routines such as `record_forward_compute_start_time`.
- CN: 第 1441-1470 行在 `record_forward_compute_start_time` 等例程中引入具体执行逻辑。

### Lines 1471-1500 / 第 1471-1500 行

```cpp
1471: // done immediately because the model output may be ignored, and we only
1472: // want to start performing reductions on `torch.autograd.backward()`.
1473: void Reducer::search_unused_parameters(
1474:     const std::vector<torch::autograd::Variable>& outputs) {
1475:   std::unordered_set<torch::autograd::Node*> seen;
1476:   std::vector<torch::autograd::Node*> queue;
1477: 
1478:   RECORD_FUNCTION(
1479:       "torch.distributed.ddp.reducer::search_unused_parameters",
1480:       std::vector<c10::IValue>());
1481: 
1482:   // Seed queue with the grad functions of all outputs.
1483:   for (const auto& output : outputs) {
1484:     const auto& grad_fn = output.grad_fn();
1485:     if (grad_fn) {
1486:       queue.push_back(grad_fn.get());
1487:     }
1488:   }
1489: 
1490:   // Traverse the autograd graph starting at the specified output.
1491:   while (!queue.empty()) {
1492:     auto fn = queue.back();
1493:     queue.pop_back();
1494:     for (const auto& edge : fn->next_edges()) {
1495:       if (auto next_ptr = edge.function.get()) {
1496:         const bool was_inserted = seen.insert(next_ptr).second;
1497:         if (was_inserted) {
1498:           queue.push_back(next_ptr);
1499:         }
1500:       }
```

- EN: Lines 1471-1500 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1471-1500 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1501-1530 / 第 1501-1530 行

```cpp
1501:     }
1502:   }
1503: 
1504:   // Find accumulator functions that don't show up in this graph.
1505:   for (const auto& it : gradAccToVariableMap_) {
1506:     // If the accumulator function is present in the graph, we know
1507:     // a gradient will be computed for the corresponding parameter.
1508:     if (seen.count(it.first) == 0) {
1509:       if (ddp_debug_level_ == c10d::DebugLevel::Detail) {
1510:         const auto param_info = param_names_.find(it.second);
1511:         TORCH_INTERNAL_ASSERT(
1512:             param_info != param_names_.end(),
1513:             "Did not find variable index ",
1514:             it.second,
1515:             " in DDP parameter name mapping!");
1516:         const auto param_name = param_info->second;
1517:         LOG(INFO) << "[Rank " << process_group_->getRank() << "]: "
1518:                   << "Parameter " << param_name << " at index " << it.second
1519:                   << " is marked as unused.";
1520:       }
1521:       unused_parameters_.push_back(it.second);
1522:     }
1523:   }
1524: 
1525:   // Warn user about unnecessary perf hit if all parameters were used in
1526:   // forward.
1527:   if (unused_parameters_.empty()) {
1528:     TORCH_WARN_ONCE(
1529:         "find_unused_parameters=True was specified in DDP constructor, "
1530:         "but did not find any unused parameters in the forward pass. This flag "
```

- EN: Lines 1501-1530 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1501-1530 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1531-1560 / 第 1531-1560 行

```cpp
1531:         "results in an extra traversal of the autograd graph every iteration, "
1532:         " which can adversely affect performance. If your model indeed never "
1533:         "has any unused parameters in the forward pass, consider turning this "
1534:         "flag off. Note that this warning may be a false positive if your model "
1535:         "has flow control causing later iterations to have unused parameters.");
1536:   }
1537:   if (!static_graph_ && ddp_graph_static_) {
1538:     if (num_iterations_ > 1) {
1539:       // Graph is still static if the set of unused parameters did not change.
1540:       ddp_graph_static_ =
1541:           prev_iteration_unused_parameters_ == unused_parameters_;
1542: 
1543:       if (!ddp_graph_static_) {
1544:         // Log graph is not static. Logger takes care of ensuring this is done
1545:         // only once to avoid overhead.
1546:         logger_.lock()->log_if_graph_static(false);
1547:       }
1548:     }
1549:     prev_iteration_unused_parameters_ = unused_parameters_;
1550:   }
1551: }
1552: 
1553: void Reducer::prepare_for_backward(
1554:     const std::vector<torch::autograd::Variable>& outputs) {
1555:   std::lock_guard<std::mutex> lock(mutex_);
1556: 
1557:   backward_compute_start_time_ = current_time_in_nanos();
1558:   if (should_collect_runtime_stats()) {
1559:     record_backward_compute_start_time();
1560:   }
```

- EN: Lines 1531-1560 introduces executable logic in routines such as `record_backward_compute_start_time`.
- CN: 第 1531-1560 行在 `record_backward_compute_start_time` 等例程中引入具体执行逻辑。

### Lines 1561-1590 / 第 1561-1590 行

```cpp
1561: 
1562:   // Reset accounting.
1563:   expect_autograd_hooks_ = true;
1564:   // Clear gradient ready order as it can be different in the next iteration.
1565:   grad_ready_order_indices_.clear();
1566: 
1567:   reset_bucket_counting();
1568: 
1569:   // Reset unused parameter accounting.
1570:   has_marked_unused_parameters_ = false;
1571:   // Reset per iteration marked ready parameters.
1572:   perIterationReadyParams_.clear();
1573: 
1574:   // If static graph is not set, search graph to detect unused parameters.
1575:   // When static graph is set, unused_parameters_ will be detected and will
1576:   // not change after 1st iteration.
1577:   // If static_graph_ = false and find_unused_parameters_ is false,
1578:   // we assume that autograd hooks for ALL variables will be called,
1579:   // and we don't have to search the autograd graph for presence of these hooks.
1580:   if (dynamic_graph_find_unused()) {
1581:     unused_parameters_.clear();
1582:     search_unused_parameters(outputs);
1583:   }
1584: }
1585: 
1586: void Reducer::copy_bucket_to_grad(
1587:     at::Tensor& variable,
1588:     Reducer::Bucket& bucket,
1589:     size_t intra_bucket_index,
1590:     bool global_unused) {
```

- EN: Lines 1561-1590 introduces executable logic in routines such as `reset_bucket_counting`.
- CN: 第 1561-1590 行在 `reset_bucket_counting` 等例程中引入具体执行逻辑。

### Lines 1591-1620 / 第 1591-1620 行

```cpp
1591:   const auto& bucket_view = bucket.bucket_views_out[intra_bucket_index];
1592:   runGradCallbackForVariable(variable, [&](auto& grad) {
1593:     // If a parameter is globally unused, we keep its grad untouched.
1594:     if (!global_unused) {
1595:       if (!grad.defined()) {
1596:         // Creates grad according to the "Gradient Layout Contract"
1597:         // (see torch/csrc/autograd/functions/accumulate_grad.h)
1598:         grad =
1599:             torch::autograd::utils::clone_obey_contract(bucket_view, variable);
1600:       } else {
1601:         grad.copy_(bucket_view);
1602:       }
1603:       // The grad is modified and needs to be written back.
1604:       return true;
1605:     }
1606:     // The grad is not modified.
1607:     return false;
1608:   });
1609: }
1610: 
1611: std::vector<std::string> Reducer::getUnmarkedParamsForIteration() {
1612:   std::vector<std::string> unMarkedParamNames;
1613:   for (const auto& it : param_names_) {
1614:     if (perIterationReadyParams_.find(it.first) ==
1615:         perIterationReadyParams_.end()) {
1616:       unMarkedParamNames.push_back(it.second);
1617:     }
1618:   }
1619:   return unMarkedParamNames;
1620: }
```

- EN: Lines 1591-1620 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1591-1620 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1621-1650 / 第 1621-1650 行

```cpp
1621: 
1622: std::vector<size_t> Reducer::getUnmarkedParamIndicesForIteration() {
1623:   std::vector<size_t> unmarked_param_indices;
1624:   const auto variable_count = params_.size();
1625:   for (const auto variable_index : c10::irange(variable_count)) {
1626:     if (perIterationReadyParams_.find(variable_index) ==
1627:         perIterationReadyParams_.end()) {
1628:       unmarked_param_indices.push_back(variable_index);
1629:     }
1630:   }
1631:   return unmarked_param_indices;
1632: }
1633: 
1634: // A bucket with one or more dense tensors needs to be unflattened.
1635: void Reducer::finalize_bucket_dense(Bucket& bucket) {
1636:   for (const auto intra_bucket_index : c10::irange(bucket.variables.size())) {
1637:     auto& variable = bucket.variables[intra_bucket_index];
1638: 
1639:     bool global_unused = false;
1640:     // See Note [Skip allreducing local_used_map_dev]
1641:     if (static_graph_ || find_unused_parameters_) {
1642:       // Determine if this param has been used globally or not.
1643:       //
1644:       // If the variable was used locally, it is also used globally and then
1645:       // we don't need to wait for the reduction. Otherwise we lazily wait for
1646:       // the reduction to complete, only when we see a variable that was
1647:       // unused locally. Then we end up delaying the synchronization point
1648:       // that local_used_work_->wait() implies. If we don't have any unused
1649:       // parameters at all, we can skip waiting for the work to complete
1650:       // altogether, and cause negligible performance overhead for models
```

- EN: Lines 1621-1650 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1621-1650 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1651-1680 / 第 1651-1680 行

```cpp
1651:       // where all parameters are used. Such lazily waiting means minimizing
1652:       // performance impact for the big majority of models where all
1653:       // parameters are always used. Then we only pay the overhead cost if
1654:       // there is indeed a parameter that is locally unused, because we need
1655:       // to check if it's also globally unused.
1656:       int64_t variable_index =
1657:           static_cast<int64_t>(bucket.variable_indices[intra_bucket_index]);
1658:       // Note: global_unused might not be global yet. As we lazily wait for
1659:       // the reduction to complete, it becomes really global only if we get to
1660:       // the point as below where we wait for the reduction work, make D2H
1661:       // copy, and update global_unused with the real global consensus, i.e.
1662:       // local_used_map_reduced_ is true.
1663:       global_unused = local_used_map_[variable_index].item<int>() == 0;
1664:       if (global_unused && !local_used_map_reduced_) {
1665:         // Wait for local_used_map reduction to complete.
1666:         local_used_work_->wait();
1667:         // D2H from local_used_map_dev_ to local_used_map_
1668:         // Blocking copy, if local_used_map_dev_ is cuda
1669:         local_used_map_.copy_(local_used_map_dev_);
1670: 
1671:         global_unused = local_used_map_[variable_index].item<int>() == 0;
1672:         local_used_map_reduced_ = true;
1673:       }
1674:     }
1675: 
1676:     if (!gradient_as_bucket_view_) {
1677:       if (optim_in_backward_) {
1678:         // Return early if optimizer has already run.
1679:         runGradCallbackForVariable(variable, [&](auto& grad) { return true; });
1680:       } else {
```

- EN: Lines 1651-1680 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1651-1680 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1681-1710 / 第 1681-1710 行

```cpp
1681:         RECORD_FUNCTION(
1682:             "torch.distributed.ddp.reducer::copy_bucket_to_grad",
1683:             std::vector<c10::IValue>({variable}));
1684:         copy_bucket_to_grad(
1685:             variable, bucket, intra_bucket_index, global_unused);
1686:       }
1687:     } else {
1688:       const auto& bucket_view_out = bucket.bucket_views_out[intra_bucket_index];
1689:       auto& bucket_view_in = bucket.bucket_views_in[intra_bucket_index];
1690:       // If a communication hook is registered, then `bucket_view_out` stores
1691:       // the allreduced results in a newly allocated tensor, so we copy
1692:       // `bucket_view_out` back to `bucket_view_in` for this gradient.
1693:       if (!bucket_view_in.is_alias_of(bucket_view_out)) {
1694:         bucket_view_in.copy_(bucket_view_out);
1695:       }
1696:       runGradCallbackForVariable(variable, [&](auto& grad) {
1697:         if (optim_in_backward_) {
1698:           // Return early if optimizer has already run.
1699:           return true;
1700:         }
1701:         // If a parameter is globally unused, we keep its grad untouched.
1702:         if (!global_unused) {
1703:           // If grad is globally used but locally unused, let grad point to
1704:           // bucket_view_in
1705:           if (!grad.defined()) {
1706:             grad = bucket_view_in;
1707:           } else {
1708:             if (!grad.is_alias_of(bucket_view_in)) {
1709:               REDUCER_CHECK(
1710:                   false,
```

- EN: Lines 1681-1710 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1681-1710 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1711-1740 / 第 1711-1740 行

```cpp
1711:                   logger_,
1712:                   "Detected at least one parameter gradient is not the "
1713:                   "expected DDP bucket view with gradient_as_bucket_view=True. "
1714:                   "This may happen (for example) if multiple allreduce hooks "
1715:                   "were registered onto the same parameter. If you hit this error, "
1716:                   "please file an issue with a minimal repro.");
1717:             }
1718:           }
1719:           // The grad is modified and needs to be written back.
1720:           return true;
1721:         }
1722:         // The grad is not modified.
1723:         return false;
1724:       });
1725:     }
1726:   }
1727: }
1728: 
1729: void Reducer::finalize_backward() {
1730:   // No longer expect autograd hooks to fire after this function returns.
1731:   TORCH_INTERNAL_ASSERT(expect_autograd_hooks_);
1732:   expect_autograd_hooks_ = false;
1733:   // reset for the next iteration
1734:   first_autograd_hook_called_ = false;
1735: 
1736:   // No longer require call to finalize after this function returns.
1737:   TORCH_INTERNAL_ASSERT(require_finalize_);
1738:   require_finalize_ = false;
1739: 
1740:   // Wait for asynchronous reduction to complete, and unflatten the bucket's
```

- EN: Lines 1711-1740 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1711-1740 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1741-1770 / 第 1741-1770 行

```cpp
1741:   // flattened `gradients` tensor.
1742:   for (auto& bucket : buckets_) {
1743:     // See Note [DDP Communication Hook]
1744:     // It is possible that the bucket all_reduce is skipped if the bucket is
1745:     // unused bucket and skip_all_reduce_unused_params_ is true.
1746:     if (bucket.future_work == nullptr) {
1747:       TORCH_INTERNAL_ASSERT(
1748:           skip_all_reduce_unused_params_,
1749:           "currently only support to skip all reduce for unused params "
1750:           "when skip_all_reduce_unused_params_ is true.");
1751:       continue;
1752:     }
1753: 
1754:     bucket.future_work->wait();
1755:     auto future_result = comm_hook_ == nullptr
1756:         ? detail::parseCppCommHookResult(bucket.future_work->value())
1757:         : comm_hook_->parseHookResult(bucket.future_work->value());
1758:     if (bucket.expect_sparse_gradient) {
1759:       // sparse metadata is set so the bucket should have sparse_tensor_indices
1760:       if (sparse_metadata_) {
1761:         REDUCER_CHECK(
1762:             bucket.sparse_tensor_indices.has_value() &&
1763:                 bucket.sparse_tensor_indices.value().numel() ==
1764:                     bucket.gradients.sizes()[0],
1765:             logger_,
1766:             "Sparse metadata and gradient size mismatch");
1767:         auto sparse_result = at::sparse_coo_tensor(
1768:             bucket.sparse_tensor_indices.value(),
1769:             future_result,
1770:             bucket.gradients.sizes());
```

- EN: Lines 1741-1770 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`.
- CN: 第 1741-1770 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑。

### Lines 1771-1800 / 第 1771-1800 行

```cpp
1771:         bucket.gradients.copy_(sparse_result);
1772:       } else {
1773:         bucket.gradients.copy_(future_result);
1774:       }
1775:     } else {
1776:       // Reinitialize only `bucket_views_out` with the future_result by
1777:       // following the same logic in `initialize_buckets`.
1778:       populate_bucket_views_out(bucket, future_result);
1779:     }
1780: 
1781:     // Unset allreduce division factor, as it may change in next backwards pass
1782:     // when running with DDP join mode.
1783:     div_factor_ = kUnsetDivFactor;
1784: 
1785:     if (!bucket.expect_sparse_gradient) {
1786:       // We don't need to finalize the sparse bucket since the sparse grad and
1787:       // the bucket essentially point to the same storage. As a result, once
1788:       // the allreduce is done, the sparse grads are automatically updated.
1789:       finalize_bucket_dense(bucket);
1790:     }
1791:   }
1792: 
1793:   if (installed_futures_ != std::nullopt) {
1794:     c10::collectAll(*installed_futures_)->wait();
1795:     installed_futures_ = std::nullopt;
1796:   }
1797: 
1798:   // See Note [Skip allreducing local_used_maps_dev]
1799:   if (dynamic_graph_find_unused() || static_graph_first_iteration()) {
1800:     // Due to the lazy wait, it is possible that reduction of the current
```

- EN: Lines 1771-1800 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1771-1800 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1801-1830 / 第 1801-1830 行

```cpp
1801:     // iteration is still going when the one for next iteration gets kicked off.
1802:     // For such case, we want to wait explicitly to make sure the reduction does
1803:     // complete before kicking off next one. Otherwise the previous one may
1804:     // interfere, write to the device-side memory and clobber the content of
1805:     // local_unused_maps_dev_.
1806:     if (!local_used_map_reduced_) {
1807:       local_used_work_->wait();
1808:     }
1809:   }
1810: 
1811:   if (dynamic_graph_find_unused()) {
1812:     // Reset unused parameter accounting.
1813:     // See Note [local_used_map_ -> local_used_map_dev copying]
1814:     local_used_map_.fill_(0);
1815:     local_used_map_reduced_ = false;
1816:   }
1817: 
1818:   if (should_collect_runtime_stats()) {
1819:     record_backward_comm_end_time();
1820:   }
1821: 
1822:   sparse_metadata_.reset();
1823: }
1824: 
1825: void Reducer::runGradCallbackForVariable(
1826:     at::Tensor& variable,
1827:     const GradCallback& cb) {
1828: #ifdef _WIN32
1829:   cb(variable.mutable_grad());
1830: #else
```

- EN: Lines 1801-1830 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `record_backward_comm_end_time`.
- CN: 第 1801-1830 行使用条件编译来适配特性开关、平台或可选后端；在 `record_backward_comm_end_time` 等例程中引入具体执行逻辑。

### Lines 1831-1860 / 第 1831-1860 行

```cpp
1831:   auto context_ptr = rpc_context_.context_ptr.load();
1832:   if (context_ptr == nullptr) {
1833:     cb(variable.mutable_grad());
1834:   } else {
1835:     // Under distributed autograd
1836:     context_ptr->runGradCallbackForVariable(variable, cb);
1837:   }
1838: #endif
1839: }
1840: 
1841: void Reducer::flush_deferred_copies(Bucket& bucket, size_t bucket_index) {
1842:   // Sparse gradients are already divided in mark_variable_ready_sparse and
1843:   // communicated independently — skip to avoid double division.
1844:   if (bucket.expect_sparse_gradient) {
1845:     return;
1846:   }
1847:   if (!bucket.deferred_copy_indices.empty()) {
1848:     std::vector<at::Tensor> dsts;
1849:     std::vector<at::Tensor> srcs;
1850:     dsts.reserve(bucket.deferred_copy_indices.size());
1851:     srcs.reserve(bucket.deferred_copy_indices.size());
1852:     for (auto idx : bucket.deferred_copy_indices) {
1853:       auto grad = bucket.variables[idx].grad();
1854:       TORCH_INTERNAL_ASSERT(
1855:           grad.defined(),
1856:           "Gradient became undefined between defer and flush for variable ",
1857:           idx,
1858:           " in bucket ",
1859:           bucket_index,
1860:           ". This indicates a bug — gradients should not be modified during backward.");
```

- EN: Lines 1831-1860 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1831-1860 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1861-1890 / 第 1861-1890 行

```cpp
1861:       dsts.push_back(bucket.bucket_views_in[idx]);
1862:       srcs.push_back(grad);
1863:     }
1864:     at::_foreach_copy_(dsts, srcs);
1865: 
1866:     // Re-alias grads to bucket views if gradient_as_bucket_view
1867:     if (gradient_as_bucket_view_) {
1868:       for (auto idx : bucket.deferred_copy_indices) {
1869:         auto& variable = bucket.variables[idx];
1870:         auto& bucket_view = bucket.bucket_views_in[idx];
1871:         runGradCallbackForVariable(variable, [&](auto& grad) {
1872:           grad = bucket_view;
1873:           return true;
1874:         });
1875:       }
1876:     }
1877:     bucket.deferred_copy_indices.clear();
1878:   }
1879:   // Single div on the entire flat bucket tensor.
1880:   // This also divides regions zeroed for undefined gradients, which is a no-op
1881:   // (0 / div_factor_ == 0) but avoids the complexity of tracking whether any
1882:   // variable in the bucket had a defined grad.
1883:   if (comm_hook_ == nullptr) {
1884:     bucket.gradients.div_(div_factor_);
1885:   }
1886: }
1887: 
1888: #ifndef _WIN32
1889: void Reducer::RpcContext::set(ContextPtr&& new_context_ptr) {
1890:   // We should set 'new_context_ptr' even if it's nullptr. That means the
```

- EN: Lines 1861-1890 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1861-1890 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1891-1920 / 第 1891-1920 行

```cpp
1891:   // reducer is under a local backward run.
1892:   const auto new_context_raw_ptr = new_context_ptr.get();
1893:   if (context_ptr.exchange(new_context_raw_ptr) != new_context_raw_ptr) {
1894:     // Set the shared ptr to the context only if it's set first time.
1895:     // All call sites should use the same context ptr.
1896:     // Use an atomic to avoid data race from multiple threads.
1897:     context_ptr_holder = std::move(new_context_ptr);
1898:   }
1899: }
1900: #endif
1901: 
1902: void Reducer::sync_bucket_indices(
1903:     std::vector<std::vector<size_t>>& bucket_indices) {
1904:   auto num_buckets = bucket_indices.size();
1905:   std::vector<size_t> bucket_sizes;
1906:   bucket_sizes.reserve(num_buckets);
1907:   int64_t total_size = 0;
1908:   for (const auto i : c10::irange(num_buckets)) {
1909:     auto bucket_size = bucket_indices.at(i).size();
1910:     bucket_sizes.push_back(bucket_size);
1911:     total_size += static_cast<int64_t>(bucket_size);
1912:   }
1913: 
1914:   at::TensorOptions options;
1915:   options = options.dtype(at::kInt);
1916:   options = options.device(params_[0].device());
1917: 
1918:   // Group indices and num_bucket together into indices_tensor
1919:   // Broadcast this tensor first, as its size is equal among all processes
1920:   auto indices_tensor = at::empty({total_size + 1}, at::kInt);
```

- EN: Lines 1891-1920 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1891-1920 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1921-1950 / 第 1921-1950 行

```cpp
1921:   auto indices_accessor = indices_tensor.accessor<int, 1>();
1922:   auto indices_accessor_Index = 0;
1923:   for (const auto i : c10::irange(num_buckets)) {
1924:     const auto& bucket_size = bucket_indices.at(i).size();
1925:     for (const auto j : c10::irange(bucket_size)) {
1926:       indices_accessor[indices_accessor_Index++] =
1927:           static_cast<int>(bucket_indices[i][j]);
1928:     }
1929:   }
1930:   indices_accessor[indices_accessor_Index] = static_cast<int>(num_buckets);
1931: 
1932:   // Copy CPU tensor to device tensor, as the process_group_ could be NCCL and
1933:   // it can only broadcast device tensors.
1934:   auto indices_tensor_device = at::empty({total_size + 1}, options);
1935:   indices_tensor_device.copy_(indices_tensor, /*non_blocking=*/true);
1936:   std::vector<at::Tensor> indices_tensor_list = {indices_tensor_device};
1937:   process_group_->broadcast(indices_tensor_list)->wait();
1938:   indices_tensor.copy_(indices_tensor_list.front(), /*non_blocking=*/false);
1939: 
1940:   // Update num_buckets after receiving it from rank 0
1941:   num_buckets = indices_accessor[indices_accessor_Index];
1942: 
1943:   // Broadcast bucket_sizes
1944:   auto bucket_sizes_tensor =
1945:       at::empty({static_cast<int64_t>(num_buckets)}, at::kInt);
1946:   auto bucket_sizes_accessor = bucket_sizes_tensor.accessor<int, 1>();
1947:   for (const auto i : c10::irange(num_buckets)) {
1948:     // For rank != 0, it is possible that local num buckets bucket_sizes.size()
1949:     // is smaller than broadcasted num_buckets
1950:     bucket_sizes_accessor[static_cast<int64_t>(i)] = static_cast<int>(
```

- EN: Lines 1921-1950 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1921-1950 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1951-1980 / 第 1951-1980 行

```cpp
1951:         bucket_sizes.at(std::min(i, (bucket_sizes.size() - 1))));
1952:   }
1953:   auto bucket_sizes_tensor_device =
1954:       at::empty({static_cast<int64_t>(num_buckets)}, options);
1955:   bucket_sizes_tensor_device.copy_(bucket_sizes_tensor, /*non_blocking=*/true);
1956:   std::vector<at::Tensor> bucket_sizes_tensor_list = {
1957:       bucket_sizes_tensor_device};
1958:   process_group_->broadcast(bucket_sizes_tensor_list)->wait();
1959:   bucket_sizes_tensor.copy_(
1960:       bucket_sizes_tensor_list.front(), /*non_blocking=*/false);
1961: 
1962:   // Clear bucket_indices first, and then update bucket_indices using received
1963:   // num_buckets, bucket_sizes_tensor and indices_tensor from rank 0
1964:   bucket_indices.clear();
1965:   bucket_indices.reserve(num_buckets);
1966:   indices_accessor_Index = 0;
1967:   for (const auto i : c10::irange(num_buckets)) {
1968:     const auto& bucket_size = bucket_sizes_accessor[static_cast<int64_t>(i)];
1969:     TORCH_CHECK_WITH(
1970:         IndexError,
1971:         bucket_size >= 0 && bucket_size <= indices_accessor.size(0),
1972:         "received invalid bucket_size, was abort called?");
1973: 
1974:     std::vector<size_t> bucket;
1975:     bucket.reserve(bucket_size);
1976:     for (const auto j : c10::irange(bucket_size)) {
1977:       (void)j;
1978:       bucket.push_back(indices_accessor[indices_accessor_Index++]);
1979:     }
1980:     bucket_indices.emplace_back(std::move(bucket));
```

- EN: Lines 1951-1980 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1951-1980 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1981-2010 / 第 1981-2010 行

```cpp
1981:   }
1982: }
1983: 
1984: bool Reducer::rebuild_buckets() {
1985:   // Ensure reduction for previous backwards pass is finished. If user's model
1986:   // has unused parameters for example, this will raise an error recommending to
1987:   // run with find_unused_parameters=True, instead of the size mismatch
1988:   // exception below.
1989:   std::lock_guard<std::mutex> lock(mutex_);
1990:   ensure_prior_reduction_finished();
1991:   if (!should_rebuild_buckets() || rebuilt_params_.empty()) {
1992:     return false;
1993:   }
1994: 
1995:   TORCH_INTERNAL_ASSERT(
1996:       rebuilt_params_.size() == rebuilt_param_indices_.size(),
1997:       c10::str(
1998:           "rebuilt parameter tensors size is not same as rebuilt parameter indices size: ",
1999:           rebuilt_params_.size(),
2000:           " versus ",
2001:           rebuilt_param_indices_.size()));
2002:   TORCH_INTERNAL_ASSERT(
2003:       params_.size() == rebuilt_param_indices_.size(),
2004:       c10::str(
2005:           "rebuilt parameter indices size is not same as original model parameters size.",
2006:           "Original model param size is: ",
2007:           params_.size(),
2008:           " versus rebuilt params size of: ",
2009:           rebuilt_param_indices_.size()));
2010: 
```

- EN: Lines 1981-2010 introduces executable logic in routines such as `ensure_prior_reduction_finished`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1981-2010 行在 `ensure_prior_reduction_finished` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2011-2040 / 第 2011-2040 行

```cpp
2011:   // Use bucket_bytes_cap_list_ if provided (non-empty),
2012:   // otherwise fall back to the original logic using first_bucket_bytes_cap_
2013:   // and bucket_bytes_cap_ to preserve backward compatibility
2014:   std::vector<size_t> bucket_size_limits;
2015:   if (!bucket_bytes_cap_list_.empty()) {
2016:     bucket_size_limits.assign(
2017:         bucket_bytes_cap_list_.begin(), bucket_bytes_cap_list_.end());
2018:   } else {
2019:     bucket_size_limits.push_back(first_bucket_bytes_cap_);
2020:     bucket_size_limits.push_back(bucket_bytes_cap_);
2021:   }
2022: 
2023:   auto ddp_set_last_bucket_as_small =
2024:       (getCvarString({"DDP_SET_LAST_BUCKET_CAP"}, "N/A") == "1");
2025: 
2026:   if (ddp_set_last_bucket_as_small) {
2027:     // Reverse so that first_bucket_bytes_cap_ (smaller bucket) becomes the last
2028:     // bucket. We cannot simply pass in {bucket_bytes_cap_,
2029:     // first_bucket_bytes_cap} as the bucket order as we would immediately
2030:     // advance to the 2nd element after the first bucket, whereas we only want
2031:     // the last bucket to have a smaller size.
2032:     std::reverse(rebuilt_params_.begin(), rebuilt_params_.end());
2033:     std::reverse(rebuilt_param_indices_.begin(), rebuilt_param_indices_.end());
2034:   }
2035:   auto [rebuilt_bucket_indices, per_bucket_size_limits] =
2036:       compute_bucket_assignment_by_size(
2037:           rebuilt_params_,
2038:           bucket_size_limits,
2039:           expect_sparse_gradients_,
2040:           rebuilt_param_indices_,
```

- EN: Lines 2011-2040 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2011-2040 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2041-2070 / 第 2041-2070 行

```cpp
2041:           logger_);
2042: 
2043:   if (ddp_set_last_bucket_as_small) {
2044:     // Reverse again because buckets were rebuilt in the opposite of gradient
2045:     // ready order.
2046:     std::reverse(rebuilt_bucket_indices.begin(), rebuilt_bucket_indices.end());
2047:     std::reverse(per_bucket_size_limits.begin(), per_bucket_size_limits.end());
2048:   }
2049: 
2050:   if (ddp_debug_level_ != c10d::DebugLevel::Off) {
2051:     TORCH_INTERNAL_ASSERT(
2052:         rebuilt_bucket_indices.size() == per_bucket_size_limits.size())
2053:     LOG(INFO) << rebuilt_bucket_indices.size()
2054:               << " buckets rebuilt with size limits: "
2055:               << c10::Join(", ", per_bucket_size_limits) << " bytes.";
2056:   }
2057: 
2058:   // For rebuilt bucket indices, it needs to be synced across all ranks.
2059:   // Broadcast the newly rebuilt bucket indices from rank 0 in default.
2060:   // After syncing up rebuilt bucket indices, initialize buckets for reducer.
2061:   sync_bucket_indices(rebuilt_bucket_indices);
2062: 
2063:   has_rebuilt_bucket_ = true;
2064:   rebuilt_params_.clear();
2065:   rebuilt_param_indices_.clear();
2066: 
2067:   initialize_buckets(std::move(rebuilt_bucket_indices));
2068: 
2069:   return true;
2070: }
```

- EN: Lines 2041-2070 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2041-2070 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2071-2100 / 第 2071-2100 行

```cpp
2071: 
2072: void Reducer::setSparseMetadata(std::map<std::string, at::Tensor>& metadata) {
2073:   sparse_metadata_ =
2074:       std::make_unique<std::map<std::string, at::Tensor>>(metadata);
2075: }
2076: 
2077: // See Note [DDP Communication Hook]
2078: void Reducer::register_comm_hook(std::unique_ptr<CommHookInterface> iface) {
2079:   REDUCER_CHECK(
2080:       comm_hook_ == nullptr,
2081:       logger_,
2082:       "register_comm_hook or register_builtin_comm_hook can only be called once.");
2083: 
2084:   comm_hook_ = std::move(iface);
2085: }
2086: 
2087: // See Note [DDP Communication Hook]
2088: void Reducer::register_builtin_comm_hook(
2089:     c10d::BuiltinCommHookType comm_hook_type) {
2090:   REDUCER_CHECK(
2091:       comm_hook_ == nullptr,
2092:       logger_,
2093:       "register_builtin_comm_hook or register_comm_hook can only be called once.");
2094: 
2095:   switch (comm_hook_type) {
2096:     case c10d::BuiltinCommHookType::ALLREDUCE:
2097:       comm_hook_ = std::make_unique<c10d::AllReduceCommHook>(process_group_);
2098:       LOG(INFO) << "Built-in communication hook ALLREDUCE is registered.";
2099:       break;
2100:     case c10d::BuiltinCommHookType::FP16_COMPRESS:
```

- EN: Lines 2071-2100 introduces executable logic in routines such as `REDUCER_CHECK`.
- CN: 第 2071-2100 行在 `REDUCER_CHECK` 等例程中引入具体执行逻辑。

### Lines 2101-2130 / 第 2101-2130 行

```cpp
2101:       comm_hook_ = std::make_unique<c10d::FP16CompressCommHook>(process_group_);
2102:       LOG(INFO) << "Built-in communication hook FP16_COMPRESS is registered.";
2103:       break;
2104:     default:
2105:       TORCH_WARN_ONCE(
2106:           "Unknown built-in DDP comm hook type is provided. No comm hook will be used.");
2107:   }
2108: }
2109: 
2110: void Reducer::ensure_prior_reduction_finished() {
2111:   // Check that any prior reduction has finished.
2112:   // The variable `require_finalize_` is true until all gradients
2113:   // have been computed and reduction of all buckets has been kicked off.
2114:   if (require_finalize_) {
2115:     // Collect unmarked parameter indices, additionally, in debug mode retrieve
2116:     // parameter names.
2117:     auto unmarked_param_indices = getUnmarkedParamIndicesForIteration();
2118:     // We should have some unmarked parameter indices, otherwise we would not
2119:     // have run into this error branch.
2120:     TORCH_INTERNAL_ASSERT(!unmarked_param_indices.empty());
2121: 
2122:     std::string kBaseErrorMsg =
2123:         "Expected to have finished reduction in the prior iteration before "
2124:         "starting a new one. "
2125:         ""
2126:         "This error indicates that your module has parameters that were "
2127:         "not used in producing loss. ";
2128:     std::string kOutputsNotUsedInLossErrorMsg =
2129:         "making sure all "
2130:         "`forward` function outputs participate in calculating loss. ";
```

- EN: Lines 2101-2130 introduces executable logic in routines such as `TORCH_WARN_ONCE`.
- CN: 第 2101-2130 行在 `TORCH_WARN_ONCE` 等例程中引入具体执行逻辑。

### Lines 2131-2160 / 第 2131-2160 行

```cpp
2131:     std::string kDDPBugErrorMsg =
2132:         "\nIf you already have done the above, then the distributed "
2133:         "data parallel module wasn't able to locate the output tensors in the "
2134:         "return value of your module's `forward` function. "
2135:         "Please include the loss function and the structure of the return "
2136:         "value of `forward` of your module when reporting this issue (e.g. "
2137:         "list, dict, iterable).";
2138: 
2139:     if (static_graph_) {
2140:       kBaseErrorMsg =
2141:           "Expected to have finished reduction in the prior iteration before "
2142:           "starting a new one. "
2143:           "This error indicates that your training graph has changed "
2144:           "in this iteration, e.g., one parameter is used in first "
2145:           "iteration, but then got unused in the second iteration. "
2146:           "this is not compatible with static_graph set to True.";
2147:     } else if (!find_unused_parameters_) {
2148:       // Parameters may have been unused in forward pass, or not all outputs
2149:       // were used in producing loss.
2150:       kBaseErrorMsg +=
2151:           "You can enable unused parameter detection by passing the "
2152:           "keyword argument `find_unused_parameters=True` to "
2153:           "`torch.nn.parallel.DistributedDataParallel`, and by \n";
2154:       kBaseErrorMsg += kOutputsNotUsedInLossErrorMsg;
2155:       kBaseErrorMsg += kDDPBugErrorMsg;
2156:     } else {
2157:       // Note that it does not really matter whether unused_parameters_.empty(),
2158:       // since user may have enabled detection but this particular iteration
2159:       // could have used or not used all parameters.
2160:       kBaseErrorMsg +=
```

- EN: Lines 2131-2160 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2131-2160 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2161-2190 / 第 2161-2190 行

```cpp
2161:           "Since `find_unused_parameters=True` is enabled, this likely "
2162:           " means that not all `forward` outputs participate in computing loss. You can fix this by ";
2163:       kBaseErrorMsg += kOutputsNotUsedInLossErrorMsg;
2164:       kBaseErrorMsg += kDDPBugErrorMsg;
2165:     }
2166: 
2167:     const std::string unmarked_param_indices_info = c10::str(
2168:         "\n",
2169:         "Parameter indices which did not receive grad for rank ",
2170:         process_group_->getRank(),
2171:         ": ",
2172:         unmarked_param_indices);
2173: 
2174:     if (ddp_debug_level_ == DebugLevel::Off) {
2175:       // Without debug mode, log unmarked_param_indices, as well as
2176:       // recommendation to use debug mode to print parameter names.
2177:       kBaseErrorMsg += unmarked_param_indices_info;
2178:       kBaseErrorMsg +=
2179:           "\n In addition, you can set the environment variable "
2180:           "TORCH_DISTRIBUTED_DEBUG to either INFO or DETAIL to print out information "
2181:           "about which particular parameters did not receive gradient on this rank "
2182:           "as part of this error";
2183:     } else {
2184:       // Retrieve set of parameter names that did not receive gradient.
2185:       auto unmarkedParams = getUnmarkedParamsForIteration();
2186:       TORCH_INTERNAL_ASSERT(!unmarkedParams.empty());
2187:       for (const auto& s : unmarkedParams) {
2188:         LOG(INFO) << "[Rank " << process_group_->getRank() << "] "
2189:                   << "Parameter: " << s
2190:                   << " did not get gradient in backwards pass.";
```

- EN: Lines 2161-2190 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2161-2190 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2191-2220 / 第 2191-2220 行

```cpp
2191:       }
2192:       const std::string unmarkedParamInfo = c10::Join(", ", unmarkedParams);
2193:       // In debug mode, log param names and indices that went unused.
2194:       kBaseErrorMsg += c10::str(
2195:           "\n",
2196:           "Parameters which did not receive grad for rank ",
2197:           process_group_->getRank(),
2198:           ": ",
2199:           unmarkedParamInfo);
2200:       kBaseErrorMsg += unmarked_param_indices_info;
2201:     }
2202:     REDUCER_CHECK(false, logger_, kBaseErrorMsg);
2203:   }
2204: }
2205: 
2206: void Reducer::set_ddp_runtime_logging_sample_rate(int sample_rate) {
2207:   ddp_runtime_logging_sample_rate_ = sample_rate;
2208: }
2209: 
2210: int Reducer::get_ddp_runtime_logging_sample_rate() {
2211:   return ddp_runtime_logging_sample_rate_;
2212: }
2213: 
2214: bool Reducer::should_collect_runtime_stats() {
2215:   if (num_iterations_ > 0 &&
2216:       (num_iterations_ <= 10 ||
2217:        num_iterations_ % get_ddp_runtime_logging_sample_rate() == 0)) {
2218:     return true;
2219:   }
2220:   return false;
```

- EN: Lines 2191-2220 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2191-2220 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2221-2250 / 第 2221-2250 行

```cpp
2221: }
2222: 
2223: void Reducer::record_forward_compute_start_time() {
2224:   if (timer_) {
2225:     timer_->record(Timer::Event::kForwardStart);
2226:   }
2227: }
2228: 
2229: void Reducer::record_backward_compute_start_time() {
2230:   if (timer_) {
2231:     timer_->record(Timer::Event::kBackwardComputeStart);
2232:   }
2233: }
2234: 
2235: void Reducer::record_backward_compute_end_time() {
2236:   if (timer_) {
2237:     timer_->record(Timer::Event::kBackwardComputeEnd);
2238:   }
2239: }
2240: 
2241: void Reducer::record_backward_comm_start_time() {
2242:   if (timer_) {
2243:     timer_->record(Timer::Event::kBackwardCommStart);
2244:   }
2245: }
2246: 
2247: void Reducer::record_backward_comm_end_time() {
2248:   if (timer_) {
2249:     timer_->record(Timer::Event::kBackwardCommEnd);
2250:   }
```

- EN: Lines 2221-2250 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2221-2250 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2251-2280 / 第 2251-2280 行

```cpp
2251: }
2252: 
2253: void Reducer::set_static_graph() {
2254:   std::lock_guard<std::mutex> lock(mutex_);
2255:   REDUCER_CHECK(
2256:       num_iterations_ == 0,
2257:       logger_,
2258:       "set_static_graph() should be called before training loop starts "
2259:       "and after DistributedDataParallel is constructed.");
2260:   static_graph_ = true;
2261:   // when static_graph_ is set as true, always initialize_local_used_map
2262:   // and detect the global unused parameters in the first iteration.
2263:   initialize_local_used_map();
2264: }
2265: 
2266: namespace {
2267: 
2268: // Tensors may be coalesced into buckets. Buckets must contain tensors of
2269: // the same type, on the same device, so a bucket can identified by a
2270: // composite key of a tensor's type identifier and its device.
2271: struct BucketKey {
2272:   BucketKey(c10::ScalarType type, c10::Device device)
2273:       : type(type), device(device) {}
2274: 
2275:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const*)
2276:   const c10::ScalarType type;
2277:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const*)
2278:   const c10::Device device;
2279: 
2280:   // See torch/csrc/utils/hash.h for dispatch code.
```

- EN: Lines 2251-2280 opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `BucketKey`; introduces executable logic in routines such as `initialize_local_used_map`.
- CN: 第 2251-2280 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `BucketKey` 等类型；在 `initialize_local_used_map` 等例程中引入具体执行逻辑。

### Lines 2281-2310 / 第 2281-2310 行

```cpp
2281:   static size_t hash(const BucketKey& key) {
2282:     return c10::get_hash(key.type, key.device);
2283:   }
2284: };
2285: 
2286: inline bool operator==(const BucketKey& lhs, const BucketKey& rhs) {
2287:   return lhs.type == rhs.type && lhs.device == rhs.device;
2288: }
2289: 
2290: } // namespace
2291: 
2292: std::tuple<std::vector<std::vector<size_t>>, std::vector<size_t>>
2293: compute_bucket_assignment_by_size(
2294:     const std::vector<at::Tensor>& tensors,
2295:     const std::vector<size_t>& bucket_size_limits,
2296:     const std::vector<bool>& expect_sparse_gradient,
2297:     const std::vector<int64_t>& tensor_indices,
2298:     const std::optional<std::weak_ptr<c10d::Logger>>& logger) {
2299:   // Either expect_sparse_gradient is not specified or it has as many elements
2300:   // as the vector with tensors.
2301:   TORCH_INTERNAL_ASSERT(
2302:       expect_sparse_gradient.empty() ||
2303:       (tensors.size() == expect_sparse_gradient.size()));
2304:   TORCH_INTERNAL_ASSERT(!tensors.empty());
2305:   // Store bucket indices and their sizes together, because we later sort the
2306:   // resulting indices by minimum tensor index and want to keep sizes
2307:   // consistent.
2308:   std::vector<std::tuple<std::vector<size_t>, size_t>> result;
2309:   // Sparse tensors go in their own bucket, so they do not have an enforced size
2310:   // limit.
```

- EN: Lines 2281-2310 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `hash`, `operator==`, `compute_bucket_assignment_by_size`.
- CN: 第 2281-2310 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `hash`、`operator==`、`compute_bucket_assignment_by_size` 等例程中引入具体执行逻辑。

### Lines 2311-2340 / 第 2311-2340 行

```cpp
2311:   size_t kNoSizeLimit = 0;
2312:   result.reserve(tensors.size());
2313: 
2314:   // Keep iterator into the size_limit vector by tensor type and device.
2315:   // This is done so that we can use the consecutive bucket limits per type.
2316:   std::unordered_map<
2317:       BucketKey,
2318:       std::vector<size_t>::const_iterator,
2319:       c10::hash<BucketKey>>
2320:       bucket_size_limit_iterators;
2321: 
2322:   // Keep vector of indices and size accumulator by tensor type and device.
2323:   std::unordered_map<BucketKey, BucketAccumulator, c10::hash<BucketKey>>
2324:       buckets;
2325: 
2326:   for (const auto i : c10::irange(tensors.size())) {
2327:     const auto& tensor = tensors[i];
2328:     auto msg = std::string("No support for sparse tensors.");
2329:     if (logger.has_value()) {
2330:       REDUCER_CHECK(!tensor.is_sparse(), logger.value(), msg);
2331:     } else {
2332:       TORCH_CHECK(!tensor.is_sparse(), msg);
2333:     }
2334: 
2335:     // when tensor_indices is empty, the index of tensors[i] assigned to
2336:     // bucket is i, otherwise the tensor index is tensor_indices[i].
2337:     auto tensor_index = i;
2338:     if (!tensor_indices.empty()) {
2339:       tensor_index = tensor_indices[i];
2340:     }
```

- EN: Lines 2311-2340 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2311-2340 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2341-2370 / 第 2341-2370 行

```cpp
2341:     // If we expect a sparse gradient to be produced for this tensor, it cannot
2342:     // be grouped together with other gradients and gets its own bucket.
2343:     if (!expect_sparse_gradient.empty() &&
2344:         expect_sparse_gradient[tensor_index]) {
2345:       result.emplace_back(std::vector<size_t>({tensor_index}), kNoSizeLimit);
2346:       continue;
2347:     }
2348: 
2349:     auto key = BucketKey(tensor.scalar_type(), tensor.device());
2350:     auto& bucket = buckets[key];
2351:     bucket.indices.push_back(tensor_index);
2352:     bucket.size += tensor.numel() * tensor.element_size();
2353: 
2354:     // Initialize bucket size limit iterator if necessary.
2355:     if (bucket_size_limit_iterators.count(key) == 0) {
2356:       bucket_size_limit_iterators[key] = bucket_size_limits.begin();
2357:     }
2358: 
2359:     auto& bucket_size_limit_iterator = bucket_size_limit_iterators[key];
2360:     const auto bucket_size_limit = *bucket_size_limit_iterator;
2361:     bucket.size_limit = bucket_size_limit;
2362:     if (bucket.size >= bucket_size_limit) {
2363:       result.emplace_back(std::move(bucket.indices), bucket.size_limit);
2364:       bucket = BucketAccumulator();
2365: 
2366:       // Advance to the next bucket size limit for this type/device.
2367:       auto next = bucket_size_limit_iterator + 1;
2368:       if (next != bucket_size_limits.end()) {
2369:         bucket_size_limit_iterator = next;
2370:       }
```

- EN: Lines 2341-2370 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2341-2370 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2371-2400 / 第 2371-2400 行

```cpp
2371:     }
2372:   }
2373: 
2374:   // Add remaining buckets.
2375:   for (auto& it : buckets) {
2376:     auto& bucket = it.second;
2377:     if (!bucket.indices.empty()) {
2378:       result.emplace_back(std::move(bucket.indices), bucket.size_limit);
2379:     }
2380:   }
2381: 
2382:   // If tensor_indices is not empty, the order of the tensors is in the gradient
2383:   // ready order, so no need to sort.
2384:   // If tensor_indices is empty, sort resulting buckets by the minimum tensor
2385:   // index they include. We assume that the order of the tensors is the order in
2386:   // which they are used (or the reverse order in which their gradients are
2387:   // produced). This sorting step ensures that the buckets are ready in
2388:   // consecutive order.
2389:   if (tensor_indices.empty()) {
2390:     std::sort(
2391:         result.begin(),
2392:         result.end(),
2393:         [](const std::tuple<std::vector<size_t>, size_t>& a,
2394:            const std::tuple<std::vector<size_t>, size_t>& b) {
2395:           const auto& indices_a = std::get<0>(a);
2396:           const auto& indices_b = std::get<0>(b);
2397:           const auto amin =
2398:               std::min_element(indices_a.begin(), indices_a.end());
2399:           const auto bmin =
2400:               std::min_element(indices_b.begin(), indices_b.end());
```

- EN: Lines 2371-2400 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2371-2400 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2401-2430 / 第 2401-2430 行

```cpp
2401:           return *amin < *bmin;
2402:         });
2403:   }
2404: 
2405:   // Return bucket indices and size limits as separate entries in tuple, as some
2406:   // APIs only need to consume bucket indices.
2407:   std::vector<std::vector<size_t>> bucket_indices;
2408:   bucket_indices.reserve(result.size());
2409:   std::vector<size_t> per_bucket_size_limits;
2410:   per_bucket_size_limits.reserve(result.size());
2411:   for (const auto& bucket_indices_with_size : result) {
2412:     bucket_indices.emplace_back(std::get<0>(bucket_indices_with_size));
2413:     per_bucket_size_limits.emplace_back(std::get<1>(bucket_indices_with_size));
2414:   }
2415:   return std::make_tuple(bucket_indices, per_bucket_size_limits);
2416: }
2417: 
2418: // Verifies corresponding params in the model replica have the same
2419: // sizes/strides across processes.
2420: void verify_params_across_processes(
2421:     const c10::intrusive_ptr<c10d::ProcessGroup>& process_group,
2422:     const std::vector<at::Tensor>& params,
2423:     const std::optional<std::weak_ptr<c10d::Logger>>& logger) {
2424:   // First verify number of parameters to avoid inconsistent inputs into
2425:   // broadcast which can cause a crash.
2426:   // See https://github.com/pytorch/pytorch/issues/73547
2427: 
2428:   at::TensorOptions param_size_options;
2429:   param_size_options = param_size_options.dtype(at::kLong);
2430:   param_size_options = param_size_options.device(params[0].device());
```

- EN: Lines 2401-2430 introduces executable logic in routines such as `verify_params_across_processes`; returns computed state or forwards results to the surrounding caller.
- CN: 第 2401-2430 行在 `verify_params_across_processes` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2431-2460 / 第 2431-2460 行

```cpp
2431:   // Note: Not using tensor building API because of
2432:   // https://github.com/pytorch/pytorch/issues/74114
2433:   at::Tensor param_size_tensor =
2434:       at::tensor({static_cast<int64_t>(params.size())}, param_size_options);
2435: 
2436:   // Allgather and verify parameter size.
2437:   std::vector<std::vector<at::Tensor>> param_size_output_tensors;
2438:   param_size_output_tensors.emplace_back();
2439:   auto world_size = process_group->getSize();
2440:   for ([[maybe_unused]] const auto i : c10::irange(world_size)) {
2441:     param_size_output_tensors.front().emplace_back(
2442:         at::empty_like(param_size_tensor));
2443:   }
2444: 
2445:   std::vector<at::Tensor> param_size_vec{param_size_tensor};
2446:   process_group->allgather(param_size_output_tensors, param_size_vec)->wait();
2447:   auto result_size_tensors = param_size_output_tensors.front();
2448:   for (const auto i : c10::irange(world_size)) {
2449:     auto param_size_for_rank = result_size_tensors[i][0].item<int>();
2450:     TORCH_CHECK(
2451:         static_cast<size_t>(param_size_for_rank) == params.size(),
2452:         c10::str(
2453:             "DDP expects same model across all ranks, but Rank ",
2454:             process_group->getRank(),
2455:             " has ",
2456:             params.size(),
2457:             " params, while rank ",
2458:             i,
2459:             " has inconsistent ",
2460:             param_size_for_rank,
```

- EN: Lines 2431-2460 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2431-2460 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2461-2490 / 第 2461-2490 行

```cpp
2461:             " params."));
2462:   }
2463: 
2464:   // Continue with parameter shape verification.
2465:   size_t i = 0;
2466:   for (const auto& t : params) {
2467:     i += 2 * t.dim();
2468:   }
2469:   at::TensorOptions options;
2470:   options = options.dtype(at::kLong);
2471:   auto metadata = at::empty({static_cast<long>(i)}, options);
2472: 
2473:   // Technically, process 0 is the broadcast source, so only process 0 needs
2474:   // to populate metadata.  But no harm keeping work aligned across processes.
2475:   auto metadata_accessor = metadata.accessor<int64_t, 1>();
2476:   i = 0;
2477:   for (const auto& t : params) {
2478:     for (const auto& sz : t.sizes()) {
2479:       metadata_accessor[static_cast<int64_t>(i++)] = sz;
2480:     }
2481:     for (const auto& str : t.strides()) {
2482:       metadata_accessor[static_cast<int64_t>(i++)] = str;
2483:     }
2484:   }
2485: 
2486:   metadata = metadata.to(params[0].device());
2487:   std::vector<at::Tensor> vec{metadata};
2488:   process_group->broadcast(vec)->wait();
2489: 
2490:   // Technically, process 0 doesn't need to double-check metadata, because it
```

- EN: Lines 2461-2490 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2461-2490 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2491-2520 / 第 2491-2520 行

```cpp
2491:   // was the source.  But no harm keeping work aligned.
2492:   auto control = at::empty({static_cast<long>(i)}, options);
2493:   control.copy_(metadata, /*non_blocking=*/false);
2494:   auto control_accessor = control.accessor<int64_t, 1>();
2495:   i = 0;
2496:   for (const auto p : c10::irange(params.size())) {
2497:     const auto& t = params[p];
2498:     for (const auto& sz : t.sizes()) {
2499:       auto msg = c10::str(
2500:           "[",
2501:           process_group->getRank(),
2502:           "]: params[",
2503:           p,
2504:           "] in this process",
2505:           " with sizes ",
2506:           t.sizes(),
2507:           " appears not to match sizes of the same param in process 0.");
2508:       if (logger.has_value()) {
2509:         REDUCER_CHECK(sz == control_accessor[i++], logger.value(), msg)
2510:       } else {
2511:         TORCH_CHECK(sz == control_accessor[i++], msg)
2512:       }
2513:     }
2514:     for (const auto& str : t.strides()) {
2515:       auto msg = c10::str(
2516:           "params[",
2517:           p,
2518:           "] in this process",
2519:           " with sizes ",
2520:           t.sizes(),
```

- EN: Lines 2491-2520 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2491-2520 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2521-2550 / 第 2521-2550 行

```cpp
2521:           " appears not to match strides of the same param in process 0.");
2522:       if (logger.has_value()) {
2523:         REDUCER_CHECK(str == control_accessor[i++], logger.value(), msg)
2524:       } else {
2525:         TORCH_CHECK(str == control_accessor[i++], msg)
2526:       }
2527:     }
2528:   }
2529: }
2530: 
2531: void Reducer::remove_autograd_hooks() {
2532:   // Remove all hooks on variables registered by this Reducer. This is necessary
2533:   // to make DDP failure recoverable. Otherwise, multiple Reducer instances
2534:   // (from recoveries) will add their hooks to the original model, and those
2535:   // hooks will try to invoke methods on a deleted Reducer objects.
2536:   for (auto& hook : hooks_) {
2537:     auto& key = hook.first;
2538:     auto& grad_accumulator = hook.second;
2539: 
2540:     TORCH_INTERNAL_ASSERT(
2541:         grad_accumulator->del_post_hook(key),
2542:         "Reducer attempts to delete a non-existing hook.");
2543:   }
2544:   hooks_.clear();
2545: }
2546: 
2547: void Reducer::check_finalized() {
2548:   std::lock_guard<std::mutex> lock(mutex_);
2549:   ensure_prior_reduction_finished();
2550: }
```

- EN: Lines 2521-2550 introduces executable logic in routines such as `ensure_prior_reduction_finished`; performs validation and error handling to keep distributed state consistent.
- CN: 第 2521-2550 行在 `ensure_prior_reduction_finished` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 2551-2580 / 第 2551-2580 行

```cpp
2551: 
2552: void Reducer::update_process_group(
2553:     c10::intrusive_ptr<c10d::ProcessGroup> new_process_group) {
2554:   std::lock_guard<std::mutex> lock(mutex_);
2555:   process_group_ = std::move(new_process_group);
2556: }
2557: 
2558: void Reducer::reset_state() {
2559:   std::lock_guard<std::mutex> lock(mutex_);
2560:   // Force rebuild of buckets.
2561:   has_rebuilt_bucket_ = false;
2562:   rebuilt_params_.clear();
2563:   rebuilt_param_indices_.clear();
2564: 
2565:   // Ensure forward can run despite previous backward not succeeding.
2566:   expect_autograd_hooks_ = false;
2567:   require_finalize_ = false;
2568:   first_autograd_hook_called_ = false;
2569: 
2570:   // Unset allreduce division factor, as it may change in next backwards pass
2571:   // when running with DDP join mode.
2572:   div_factor_ = kUnsetDivFactor;
2573: 
2574:   // Reset unused parameter accounting.
2575:   // See Note [local_used_map_ -> local_used_map_dev copying]
2576:   if (find_unused_parameters_) {
2577:     local_used_map_.zero_();
2578:     local_used_map_reduced_ = false;
2579:   }
2580: }
```

- EN: Lines 2551-2580 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2551-2580 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2581-2582 / 第 2581-2582 行

```cpp
2581: 
2582: } // namespace c10d
```

- EN: Lines 2581-2582 opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 2581-2582 行打开或关闭命名空间，以便将代码放入正确的分布式组件中。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `CpuTimer`, `BucketKey`
- CN: 核心符号：`CpuTimer`、`BucketKey`
- EN: Notable themes: Python bindings, process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：Python 绑定、进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/reducer.hpp`, `torch/csrc/distributed/c10d/Utils.hpp`, `torch/csrc/distributed/c10d/default_comm_hooks.hpp`, `torch/csrc/distributed/c10d/comm.hpp`, `torch/csrc/distributed/c10d/logger.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/core/ScalarType.h`, `c10/util/Exception.h`, `c10/util/Logging.h`, `c10/util/hash.h`, `c10/util/irange.h`, `torch/csrc/autograd/engine.h`, `torch/csrc/autograd/function_hook.h`, `torch/csrc/autograd/utils/grad_layout_contract.h`, `torch/csrc/autograd/utils/lambda_post_hook.h`
- External or system headers / 外部或系统头文件: `functional`, `utility`
- Local symbols / 本地符号: `CpuTimer`, `BucketKey`