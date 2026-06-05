# reducer.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/reducer.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for reducer in the c10d distributed process-group subsystem. Key types include `Logger`, `BucketAccumulator`, `TORCH_API`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供reducer 的接口与类型声明。 关键类型包括 `Logger`、`BucketAccumulator`、`TORCH_API`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: #pragma once
2: 
3: #include <c10/core/ScalarType.h>
4: #include <atomic>
5: #include <memory>
6: #include <mutex>
7: #include <tuple>
8: #include <unordered_map>
9: #include <vector>
10: 
11: #include <ATen/core/ivalue_inl.h>
12: #include <c10/macros/Macros.h>
13: #include <c10/util/intrusive_ptr.h>
14: #include <torch/csrc/autograd/function.h>
15: #include <torch/csrc/autograd/profiler.h>
16: #include <torch/csrc/autograd/variable.h>
17: #include <torch/csrc/distributed/c10d/ProcessGroup.hpp>
18: #include <torch/csrc/distributed/c10d/Utils.hpp>
19: #include <torch/csrc/distributed/c10d/comm.hpp>
20: #include <torch/csrc/distributed/c10d/debug.h>
```

- EN: Lines 1-20 pulls in the headers required by this translation unit or interface.
- CN: 第 1-20 行引入该实现单元或接口所需的头文件。

### Lines 21-40 / 第 21-40 行

```cpp
21: #include <torch/csrc/distributed/c10d/default_comm_hooks.hpp>
22: #include <torch/csrc/distributed/c10d/reducer_timer.hpp>
23: #ifndef _WIN32
24: #include <torch/csrc/distributed/autograd/context/context.h>
25: #endif
26: 
27: namespace c10d {
28: 
29: constexpr int kDefaultFirstBucketBytes = 1024 * 1024;
30: constexpr int kDefaultBucketBytesCap = 25 * 1024 * 1024;
31: // Collect runtime stats once for every kDDPRuntimeLoggingSampleRate iterations.
32: constexpr int kDDPRuntimeLoggingSampleRate = 100;
33: 
34: // Forward declaration
35: class Logger;
36: 
37: // Local accumulator type for a single bucket.
38: struct BucketAccumulator {
39:   std::vector<size_t> indices;
40:   size_t size = 0;
```

- EN: Lines 21-40 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 21-40 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 41-60 / 第 41-60 行

```cpp
41:   size_t size_limit = 0;
42: };
43: 
44: class TORCH_API Reducer {
45:  public:
46:   // The constructor takes a list of variables (i.e. parameters) for this
47:   // process's single model replica (as DDP assumes single-process
48:   // single-device). The bucket assignment for this reducer, `bucket_indices`,
49:   // is specified as a list of buckets, each of which is specified as a list of
50:   // indices into the bucket's `variables` list.
51:   explicit Reducer(
52:       std::vector<at::Tensor> params,
53:       std::vector<std::vector<size_t>> bucket_indices,
54:       c10::intrusive_ptr<c10d::ProcessGroup> process_group,
55:       std::vector<bool> expect_sparse_gradients,
56:       int64_t bucket_bytes_cap,
57:       bool find_unused_parameters,
58:       bool gradient_as_bucket_view,
59:       std::unordered_map<size_t, std::string> param_names,
60:       int64_t first_bucket_bytes_cap,
```

- EN: Lines 41-60 declares or defines types such as `TORCH_API`.
- CN: 第 41-60 行声明或定义了 `TORCH_API` 等类型。

### Lines 61-80 / 第 61-80 行

```cpp
61:       bool skip_all_reduce_unused_params,
62:       bool use_python_reducer,
63:       std::vector<int64_t> bucket_bytes_cap_list,
64:       bool batched_grad_copy = false);
65: 
66:   ~Reducer() noexcept(false);
67: 
68:   // To (re-)initialize bucket assignment, pass a list of buckets, each of
69:   // which is specified by a list of indices in the bucket's `variables` list.
70:   // This function performs validation that the variables within a bucket
71:   // all live on the same device and have the same dimensionality.
72:   void initialize_buckets(std::vector<std::vector<size_t>> bucket_indices);
73: 
74:   void autograd_hook(size_t index);
75: 
76:   // This function is called when the forward function has produced an output,
77:   // and the user wishes to reduce gradients in the backwards pass.
78:   // If they don't, and wish to accumulate gradients before reducing them,
79:   // a call to this function can simply be omitted.
80:   void prepare_for_backward(const std::vector<at::Tensor>& outputs);
```

- EN: Lines 61-80 introduces executable logic in routines such as `~Reducer`, `initialize_buckets`, `autograd_hook`.
- CN: 第 61-80 行在 `~Reducer`、`initialize_buckets`、`autograd_hook` 等例程中引入具体执行逻辑。

### Lines 81-100 / 第 81-100 行

```cpp
81: 
82:   // Called at the beginning of forward() inside DistributedDataParallel,
83:   // right now it captures the starting time of forward in each iteration.
84:   void prepare_for_forward();
85: 
86:   // Returns the relative time in nanoseconds when gradients were ready,
87:   // with respect to the time `prepare_for_backward` was called. The
88:   // vector is for parameters for a single model replica.
89:   std::vector<int64_t> get_backward_stats() const {
90:     return backward_stats_;
91:   }
92: 
93:   // Registers a hook to the reducer. The hook is `CommHookInterface`
94:   // type to allow both Python and CPP hooks. This function can only
95:   // be called once before calling backward.
96:   // Cannot combine with the call of `register_builtin_comm_hook`.
97:   void register_comm_hook(std::unique_ptr<CommHookInterface> iface);
98: 
99:   // Registers a built-in C++ comm hook to the reducer. This function can only
100:   // be called once before calling backward.
```

- EN: Lines 81-100 introduces executable logic in routines such as `prepare_for_forward`, `get_backward_stats`, `register_comm_hook`; returns computed state or forwards results to the surrounding caller.
- CN: 第 81-100 行在 `prepare_for_forward`、`get_backward_stats`、`register_comm_hook` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 101-120 / 第 101-120 行

```cpp
101:   // Cannot combine with the call of `register_comm_hook`.
102:   void register_builtin_comm_hook(c10d::BuiltinCommHookType comm_hook_type);
103: 
104:   // Informs reducer that optimizer is running in backward, so gradients
105:   // don't need to be copied from buckets as the optimizer would've already
106:   // been applied.
107:   void set_optimizer_in_backward() {
108:     optim_in_backward_ = true;
109:   }
110: 
111:   // Runs allreduce or installed communication hook given GradBucket instance.
112:   c10::intrusive_ptr<c10::ivalue::Future> run_comm_hook(
113:       GradBucket& grad_bucket);
114: 
115:   // Runs default allreduce hook.
116:   c10::intrusive_ptr<c10::ivalue::Future> run_allreduce_hook(
117:       GradBucket& grad_bucket);
118: 
119:   // Returns gradient buckets in sequential order of buckets_. This is the order
120:   // in which buckets are reduced across processes. If return_zero_tensors=true,
```

- EN: Lines 101-120 introduces executable logic in routines such as `register_builtin_comm_hook`, `set_optimizer_in_backward`, `run_comm_hook`.
- CN: 第 101-120 行在 `register_builtin_comm_hook`、`set_optimizer_in_backward`、`run_comm_hook` 等例程中引入具体执行逻辑。

### Lines 121-140 / 第 121-140 行

```cpp
121:   // will return zero tensors of the same shape instead of the true tensors.
122:   std::vector<c10d::GradBucket> get_grad_buckets(
123:       bool return_zero_tensors = true) const;
124: 
125:   // Rebuild buckets based on rebuilt_params_ and rebuilt_param_indices_
126:   // according to when tensors received grads in the backward pass.
127:   // TODO this function makes broadcast communication call and
128:   // could be overlapped with next forward() call, thus
129:   // it could be async. Will make it async when rebuilding buckets for
130:   // find_unused_parameters = true case, as we could rebuild buckets more than
131:   // once for find_unused_parameters = true case, where subgraphs are trained
132:   // and parameter indices order may change more frequently.
133:   // For find_unused_parameters = false case, buckets are only rebuilt once,
134:   // the performance cost is negligible. Returns true if the buckets were
135:   // rebuilt.
136:   bool rebuild_buckets();
137: 
138:   void setSparseMetadata(std::map<std::string, at::Tensor>& metadata);
139: 
140:   // Install futures that should be awaited at end of backwards. Currently these
```

- EN: Lines 121-140 introduces executable logic in routines such as `get_grad_buckets`, `rebuild_buckets`, `setSparseMetadata`; returns computed state or forwards results to the surrounding caller.
- CN: 第 121-140 行在 `get_grad_buckets`、`rebuild_buckets`、`setSparseMetadata` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 141-160 / 第 141-160 行

```cpp
141:   // are only used by user-defined custom buffer reduction hooks, but can be
142:   // generalized to any user-originating futures that need to be awaited.
143:   void install_futures(
144:       const c10::List<c10::intrusive_ptr<c10::ivalue::Future>>& futs);
145: 
146:   // Returns true if we should rebuild buckets, else false. We only rebuild
147:   // buckets once after the first iteration and never rebuild them if
148:   // find_unused_parameters_.
149:   inline bool should_rebuild_buckets() const {
150:     return (static_graph_ || !find_unused_parameters_) && !has_rebuilt_bucket_;
151:   }
152: 
153:   // Pushes all parameters to be rebuilt.
154:   void push_rebuilt_params_for_all_indices();
155: 
156:   // Creates and sets ForwardPassWorkHandle given a Work and the
157:   // corresponding tensor being reduced.
158:   void set_forward_pass_work_handle(
159:       c10::intrusive_ptr<c10d::Work> forwardPassWorkHandle,
160:       bool useStaticWorldSize);
```

- EN: Lines 141-160 introduces executable logic in routines such as `install_futures`, `should_rebuild_buckets`, `push_rebuilt_params_for_all_indices`; returns computed state or forwards results to the surrounding caller.
- CN: 第 141-160 行在 `install_futures`、`should_rebuild_buckets`、`push_rebuilt_params_for_all_indices` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 161-180 / 第 161-180 行

```cpp
161: 
162:   // Retrieve on-device tensors used to track locally unused parameters. It is
163:   // a tensor where index i = 1 if the Variable with that index has been used.
164:   at::Tensor get_local_used_map_on_device() const;
165: 
166:   // An function for users to set sample_rate of collecting
167:   // runtime stats. The time stats will be recorded for the
168:   // first 10 iterations, after 10 iterations time stats will be
169:   // recorded once every "sample_rate" training iterations.
170:   void set_ddp_runtime_logging_sample_rate(int sample_rate);
171: 
172:   // Specify the training graph is static.
173:   void set_static_graph();
174: 
175:   // Delay all reduce to be after all gradients' calculation is complete.
176:   void delay_all_reduce();
177: 
178:   void set_mixed_precision_param_dtype(c10::ScalarType dtype);
179: 
180:   // Weak reference to associated DDP logger. The reference is weak to avoid
```

- EN: Lines 161-180 introduces executable logic in routines such as `get_local_used_map_on_device`, `set_ddp_runtime_logging_sample_rate`, `set_static_graph`.
- CN: 第 161-180 行在 `get_local_used_map_on_device`、`set_ddp_runtime_logging_sample_rate`、`set_static_graph` 等例程中引入具体执行逻辑。

### Lines 181-200 / 第 181-200 行

```cpp
181:   // refcycle between reducer and logger.
182:   void set_logger(std::weak_ptr<c10d::Logger> logger);
183: 
184:   // When graph is not explicitly set by user as static and has unused
185:   // parameters, this will return whether the graph has been static until the
186:   // current iteration, which means unused params set has not changed.
187:   bool ddp_graph_static();
188: 
189:   // Removes autograd hooks registered by the Reducer on the model parameters.
190:   void remove_autograd_hooks();
191: 
192:   // Checks whether or not the reducer has finalized the current backward
193:   // iteration.
194:   void check_finalized();
195: 
196:   // Updates the underlying process group used by DDP with the new process
197:   // group.
198:   void update_process_group(
199:       c10::intrusive_ptr<c10d::ProcessGroup> new_process_group);
200: 
```

- EN: Lines 181-200 introduces executable logic in routines such as `set_logger`, `ddp_graph_static`, `remove_autograd_hooks`; returns computed state or forwards results to the surrounding caller.
- CN: 第 181-200 行在 `set_logger`、`ddp_graph_static`、`remove_autograd_hooks` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 201-220 / 第 201-220 行

```cpp
201:   // Resets reducer state.
202:   void reset_state();
203: 
204:  protected:
205:   // Forward declaration.
206:   struct Bucket;
207: 
208:   void push_rebuilt_params(const size_t& index);
209: 
210:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
211:   mutable std::mutex mutex_;
212:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
213:   const std::vector<at::Tensor> params_;
214:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
215:   c10::intrusive_ptr<::c10d::ProcessGroup> process_group_;
216:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
217:   std::vector<bool> expect_sparse_gradients_;
218: 
219:   std::vector<c10::intrusive_ptr<torch::autograd::Node>>
220:       grad_accumulators_; // NOLINT(cppcoreguidelines-non-private-member-variables-in-classes)
```

- EN: Lines 201-220 declares or defines types such as `Bucket`; introduces executable logic in routines such as `reset_state`, `push_rebuilt_params`.
- CN: 第 201-220 行声明或定义了 `Bucket` 等类型；在 `reset_state`、`push_rebuilt_params` 等例程中引入具体执行逻辑。

### Lines 221-240 / 第 221-240 行

```cpp
221:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
222:   std::unordered_map<torch::autograd::Node*, size_t> gradAccToVariableMap_;
223:   std::vector<std::pair<uintptr_t, c10::intrusive_ptr<torch::autograd::Node>>>
224:       hooks_; // NOLINT(cppcoreguidelines-non-private-member-variables-in-classes)
225: 
226:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
227:   bool expect_autograd_hooks_;
228:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
229:   bool require_finalize_;
230:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
231:   size_t next_bucket_;
232: 
233:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
234:   bool has_marked_unused_parameters_;
235:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
236:   const bool find_unused_parameters_;
237:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
238:   const bool gradient_as_bucket_view_;
239:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
240:   const bool batched_grad_copy_;
```

- EN: Lines 221-240 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 221-240 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 241-260 / 第 241-260 行

```cpp
241:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
242:   std::vector<size_t> unused_parameters_;
243:   // Previous iteration's unused params, used for checking if unused parameters
244:   // change between iterations. Only filled during the first backwards call.
245:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
246:   std::vector<size_t> prev_iteration_unused_parameters_;
247:   // Whether graph is static or not. When user does not explicitly set static
248:   // graph, the only possible dynamism is set of unused parameters changing
249:   // between iterations which is tracked by this flag.
250:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
251:   bool ddp_graph_static_{true};
252:   // Locally used parameter maps indicating if parameters are used locally
253:   // during the current iteration or no_sync session if no_sync is on.
254:   // Each map is a one-dim int32 tensor of number of parameters. These tensors
255:   // are marked in autograd_hook to indicate the corresponding param has been
256:   // used, and get allreduced in the end of backward step of current iteration
257:   // or no_sync session for figuring out the globally unused parameters.
258:   //
259:   // local_used_map_:     CPU tensor for bookkeeping locally used params
260:   // local_used_map_dev_: dev tensor for reducing globally unused params
```

- EN: Lines 241-260 continues the local implementation details and data flow for this file.
- CN: 第 241-260 行继续展开本文件的局部实现细节与数据流。

### Lines 261-280 / 第 261-280 行

```cpp
261:   at::Tensor local_used_map_;
262:   at::Tensor local_used_map_dev_;
263:   // Indicate that reduction is done and D2H copy is done as well.
264:   bool local_used_map_reduced_;
265: 
266:   // Weak pointer to associated DDP logger.
267:   std::weak_ptr<c10d::Logger> logger_;
268:   // List of futures installed by Reducer::install_futures that should be
269:   // awaited at the end of backwards pass.
270:   std::optional<c10::List<c10::intrusive_ptr<c10::ivalue::Future>>>
271:       installed_futures_{std::nullopt};
272:   // Mixed precision parameter dtype for bucket type checking.
273:   std::optional<c10::ScalarType> mixed_precision_param_dtype_{std::nullopt};
274: 
275:   // Work handle for allreduce on local_used_map_
276:   c10::intrusive_ptr<c10d::Work> local_used_work_;
277: 
278:   void mark_variable_ready_dense(size_t variable_index);
279: 
280:   void mark_variable_ready_sparse(size_t variable_index);
```

- EN: Lines 261-280 introduces executable logic in routines such as `mark_variable_ready_dense`, `mark_variable_ready_sparse`.
- CN: 第 261-280 行在 `mark_variable_ready_dense`、`mark_variable_ready_sparse` 等例程中引入具体执行逻辑。

### Lines 281-300 / 第 281-300 行

```cpp
281: 
282:   void mark_variable_ready(size_t variable_index);
283: 
284:   void mark_bucket_ready(size_t bucket_index);
285: 
286:   void finalize_bucket_dense(Bucket& bucket);
287: 
288:   void finalize_backward();
289: 
290:   // Returns list of model parameters corresponding to the given bucket.
291:   // bucket_index is a key to cache after buckets are rebuilt, after which this
292:   // mapping never changes.
293:   std::vector<at::Tensor> get_variables_for_bucket(
294:       size_t bucket_index,
295:       const Bucket& bucket) const;
296: 
297:   // Asserts that the reduction for the previous iteration has finished before
298:   // rebuilding buckets or kicking off the next one.
299:   void ensure_prior_reduction_finished();
300: 
```

- EN: Lines 281-300 introduces executable logic in routines such as `mark_variable_ready`, `mark_bucket_ready`, `finalize_bucket_dense`.
- CN: 第 281-300 行在 `mark_variable_ready`、`mark_bucket_ready`、`finalize_bucket_dense` 等例程中引入具体执行逻辑。

### Lines 301-320 / 第 301-320 行

```cpp
301:   // Broadcast rebuilt buckets from rank 0 to other ranks before initializing
302:   // the buckets
303:   void sync_bucket_indices(std::vector<std::vector<size_t>>& bucket_indices);
304: 
305:   // We'd like to use DistAutogradContext::GradCallback here but dist autograd
306:   // doesn't exist under Windows. So we just directly use the concrete type but
307:   // to preserve and enforce our original intent we do a static assert when dist
308:   // autograd is available.
309:   using GradCallback = std::function<bool(at::Tensor&)>;
310: #ifndef _WIN32
311:   static_assert(
312:       std::is_same_v<
313:           GradCallback,
314:           torch::distributed::autograd::DistAutogradContext::GradCallback>);
315: #endif
316:   void runGradCallbackForVariable(at::Tensor& variable, const GradCallback& cb);
317: 
318:   // Flushes deferred grad-to-bucket copies for a single bucket when
319:   // batched_grad_copy_ is enabled. Called from mark_variable_ready (when
320:   // bucket.pending == 0) and from delay_all_reduce (after all variables
```

- EN: Lines 301-320 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `sync_bucket_indices`, `static_assert`, `runGradCallbackForVariable`.
- CN: 第 301-320 行使用条件编译来适配特性开关、平台或可选后端；在 `sync_bucket_indices`、`static_assert`、`runGradCallbackForVariable` 等例程中引入具体执行逻辑。

### Lines 321-340 / 第 321-340 行

```cpp
321:   // are marked ready).
322:   void flush_deferred_copies(Bucket& bucket, size_t bucket_index);
323: 
324:   // This function is called inside `initialize_buckets()`. It initializes both
325:   // `bucket_views_in` and `bucket_views_out` with views for each variable's
326:   // gradient into the bucket's flattened `gradients` tensor. Views serve as
327:   // entry points to `copy_()` each grad's data in/out of the flattened
328:   // `gradients` tensor.
329:   void initialize_bucket_views(Bucket& bucket);
330: 
331:   // This function is called inside `finalize_backward`, it happens only if
332:   // DDP communication hook was registered to recreate just bucket_views_out
333:   // with the result of `future_work`.
334:   void populate_bucket_views_out(Bucket& bucket, at::Tensor& tensor);
335: 
336:   // If gradient_as_bucket_view_ is false, after allreduce buckets,
337:   // copy bucket results back to grads.
338:   void copy_bucket_to_grad(
339:       at::Tensor& variable,
340:       Reducer::Bucket& bucket,
```

- EN: Lines 321-340 introduces executable logic in routines such as `flush_deferred_copies`, `initialize_bucket_views`, `populate_bucket_views_out`.
- CN: 第 321-340 行在 `flush_deferred_copies`、`initialize_bucket_views`、`populate_bucket_views_out` 等例程中引入具体执行逻辑。

### Lines 341-360 / 第 341-360 行

```cpp
341:       size_t intra_bucket_index,
342:       bool global_unused);
343:   // Check layout of grad and bucket_view before copying the grad to bucket.
344:   void check_grad_layout(const at::Tensor& grad, const at::Tensor& bucket_view);
345: 
346:   // A bucket contains [1..N] gradients to be reduced, where the gradients
347:   // have the same dtype and device.
348:   // Coalescing gradients together before reducing can result in lower overhead
349:   // and/or faster time to completion. Coalescing requires the constituent
350:   // gradients to have the same dtype and device, and the resulting flattened
351:   // tensor uses that common dtype and device. The flattened tensor is filled
352:   // as the corresponding gradients are computed (triggered by autograd hooks),
353:   // and the buckets are reduced in a predetermined order consistent across
354:   // processes.
355:   struct Bucket {
356:     // Gradients of the bucket flattened into a 1-dimensional tensor
357:     at::Tensor gradients;
358: 
359:     // Views into the `gradients` tensor for each individual gradient
360:     // Each view is created with layout (size and stride) matching the
```

- EN: Lines 341-360 declares or defines types such as `Bucket`; introduces executable logic in routines such as `check_grad_layout`.
- CN: 第 341-360 行声明或定义了 `Bucket` 等类型；在 `check_grad_layout` 等例程中引入具体执行逻辑。

### Lines 361-380 / 第 361-380 行

```cpp
361:     // gradient's expected layout (see the "Gradient Layout Contract" in
362:     // torch/csrc/autograd/functions/accumulate_grad.h).
363:     // `bucket_views_in[i].copy_(grad)` and `grad.copy_(bucket_views_out[i])`
364:     // provide convenient ways to copy gradient data in/out of `gradients`,
365:     // respectively.
366:     // We keep both `bucket_views_in` and `bucket_views_out` because
367:     // registering a DDP communication hook may re-initialize
368:     // `bucket_views_out` with the value of the hook's `future_work` but we
369:     // still need separate views into the bucket's original flattened gradient
370:     // to copy in gradient data.
371:     std::vector<at::Tensor> bucket_views_in;
372:     std::vector<at::Tensor> bucket_views_out;
373: 
374:     // Variables whose gradients are held in this bucket
375:     // We use refcounted tensors here so that we can easily unflatten the
376:     // bucket's flattened `gradients` tensor into the participating variables
377:     // after reduction has completed.
378:     std::vector<at::Tensor> variables;
379: 
380:     // Per-variable offset/length into the flattened `gradients` tensor and
```

- EN: Lines 361-380 continues the local implementation details and data flow for this file.
- CN: 第 361-380 行继续展开本文件的局部实现细节与数据流。

### Lines 381-400 / 第 381-400 行

```cpp
381:     // the corresponding `GradBucket` instance for communication hooks
382:     std::vector<size_t> offsets;
383:     std::vector<size_t> lengths;
384: 
385:     // Per-variable sizes slicing into the bucket's `gradients` tensor
386:     std::vector<c10::IntArrayRef> sizes_vec;
387: 
388:     // Number of gradients left to be computed before the bucket is ready to
389:     // be reduced
390:     size_t pending;
391: 
392:     // Global indices of participating variables in the bucket
393:     std::vector<size_t> variable_indices;
394: 
395:     // Future work handle for DDP communication hook
396:     // If no hook is registered, a temporary vanilla allreduce hook is used.
397:     c10::intrusive_ptr<at::ivalue::Future> future_work;
398: 
399:     // if this bucket contains complex parameters
400:     bool is_complex_bucket = false;
```

- EN: Lines 381-400 continues the local implementation details and data flow for this file.
- CN: 第 381-400 行继续展开本文件的局部实现细节与数据流。

### Lines 401-420 / 第 401-420 行

```cpp
401: 
402:     // If this bucket should expect a single sparse gradient
403:     // If `true`, then this implies that `bucket.variables.size() == 1`.
404:     bool expect_sparse_gradient = false;
405: 
406:     // Sparse indices tensor
407:     std::optional<at::Tensor> sparse_tensor_indices = std::nullopt;
408: 
409:     // TODO(@pietern)
410:     // Memory copies from gradient tensors into the bucket are potentially
411:     // done on different CUDA streams. We record an event for every copy
412:     // so that we can synchronize with them prior to kicking off the reduction.
413:     // std::vector<at::cuda::CUDAEvent> events;
414: 
415:     // Intra-bucket indices of variables whose grad-to-bucket copies are
416:     // deferred for batching. Flushed as _foreach_copy_ + flat div_ when
417:     // pending == 0. Only used when batched_grad_copy is enabled.
418:     std::vector<size_t> deferred_copy_indices;
419:   };
420: 
```

- EN: Lines 401-420 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 401-420 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 421-440 / 第 421-440 行

```cpp
421:   std::vector<Bucket> buckets_;
422: 
423:   // A variable locator locates a particular variable in the reducer's buckets
424:   struct VariableLocator {
425:     // Index of the bucket containing the variable in the `buckets_` vector
426:     size_t bucket_index;
427:     // Index of the variable in the bucket, which may be used consistently
428:     // across `bucket_views_in`, `bucket_views_out`, `variables`, `offsets`,
429:     // `lengths`, `sizes_vec`, and `variable_indices` in `Bucket`
430:     size_t intra_bucket_index;
431: 
432:     VariableLocator() = default;
433: 
434:     VariableLocator(size_t bucket_index_, size_t intra_bucket_index_)
435:         : bucket_index(bucket_index_),
436:           intra_bucket_index(intra_bucket_index_) {}
437:   };
438: 
439:   // Map the index of a variable to its location in the bucket structure.
440:   std::vector<VariableLocator> variable_locators_;
```

- EN: Lines 421-440 declares or defines types such as `VariableLocator`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 421-440 行声明或定义了 `VariableLocator` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 441-460 / 第 441-460 行

```cpp
441: 
442:   // track the number of iterations to synchronize grads in training so far.
443:   long num_iterations_;
444:   // track distinct iteration of backward call. This is distinct from
445:   // num_iterations_, for example in the case of multiple forward before
446:   // backward.
447:   long num_bwd_calls_;
448:   // whether the first autograd hook for a distinct backward pass has been
449:   // called.
450:   bool first_autograd_hook_called_;
451:   // track the number of buckets that have been ready for
452:   // communication calls like allReduce or communication hooks.
453:   int num_buckets_ready_;
454:   // track the number of buckets that have been reduced.
455:   int num_buckets_reduced_;
456: 
457:   // Timing information.
458:   int64_t backward_compute_start_time_ = -1;
459:   std::unique_ptr<Timer> timer_;
460: 
```

- EN: Lines 441-460 continues the local implementation details and data flow for this file.
- CN: 第 441-460 行继续展开本文件的局部实现细节与数据流。

### Lines 461-480 / 第 461-480 行

```cpp
461:   // We collect the relative timestamp of every gradient being ready
462:   // when executing autograd. This can be used to derive a timeline of
463:   // the point in time buckets were ready, or ideal bucket assignment/ordering.
464:   std::vector<int64_t> backward_stats_;
465: 
466:   bool should_collect_runtime_stats();
467:   void record_forward_compute_start_time();
468:   void record_backward_compute_start_time();
469:   void record_backward_compute_end_time();
470:   void record_backward_comm_start_time();
471:   void record_backward_comm_end_time();
472: 
473:   int get_ddp_runtime_logging_sample_rate();
474:   int ddp_runtime_logging_sample_rate_ = kDDPRuntimeLoggingSampleRate;
475: 
476:   bool is_multi_device_module_ = false;
477: 
478:   // Following variables are to help build dynamic bucket order
479:   bool has_rebuilt_bucket_;
480:   std::vector<at::Tensor> rebuilt_params_;
```

- EN: Lines 461-480 introduces executable logic in routines such as `should_collect_runtime_stats`, `record_forward_compute_start_time`, `record_backward_compute_start_time`.
- CN: 第 461-480 行在 `should_collect_runtime_stats`、`record_forward_compute_start_time`、`record_backward_compute_start_time` 等例程中引入具体执行逻辑。

### Lines 481-500 / 第 481-500 行

```cpp
481:   std::vector<int64_t> rebuilt_param_indices_;
482:   const int64_t bucket_bytes_cap_;
483: 
484: #ifndef _WIN32
485:   struct RpcContext {
486:     using ContextPtr = torch::distributed::autograd::ContextPtr;
487:     // The shared_ptr is to hold the context instance.
488:     ContextPtr context_ptr_holder;
489:     std::atomic<ContextPtr::element_type*> context_ptr{nullptr};
490: 
491:     void set(ContextPtr&& new_context_ptr);
492:   };
493:   RpcContext rpc_context_;
494: #endif
495: 
496:   // A struct containing work handle and tensor for allreduce scheduled in
497:   // forward pass, if applicable.
498:   struct ForwardPassAllreduceWork {
499:     c10::intrusive_ptr<c10d::Work> workHandle;
500:     at::Tensor resultTensor;
```

- EN: Lines 481-500 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `RpcContext`, `ForwardPassAllreduceWork`; introduces executable logic in routines such as `set`.
- CN: 第 481-500 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `RpcContext`、`ForwardPassAllreduceWork` 等类型；在 `set` 等例程中引入具体执行逻辑。

### Lines 501-520 / 第 501-520 行

```cpp
501:     // whether we should divide by the initial world_size or the no. of
502:     // remaining DDP ranks.
503:     bool useStaticWorldSize;
504:   };
505: 
506:   // Handle for the currently scheduled allreduce in the forward pass, if
507:   // applicable.
508:   ForwardPassAllreduceWork forwardPassWorkHandle_;
509: 
510:   // Division factor for reduction of gradients.
511:   // Equal to the process group size, with an exception of handling uneven
512:   // input.
513:   int div_factor_;
514: 
515:   bool static_graph_;
516: 
517:   bool skip_all_reduce_unused_params_;
518: 
519:   // Key: size_t (index), Value: the number of times that a variable's
520:   // autograd_hook() should be triggered before marking this variable's grad as
```

- EN: Lines 501-520 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 501-520 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 521-540 / 第 521-540 行

```cpp
521:   // ready for communication. Map will not change after 1st iteration.
522:   std::unordered_map<size_t, int> numGradHooksTriggeredMap_;
523:   // Key: size_t (index), Value: the number of times that a variable's
524:   // autograd_hook() are left to be triggered before marking this variable's
525:   // grad as ready for communication. Map will change after 1st iteration to
526:   // track a grad is ready for communication or not.
527:   std::unordered_map<size_t, int> numGradHooksTriggeredMapPerIteration_;
528: 
529:  private:
530:   // reset counting for buckets before backward starts
531:   void reset_bucket_counting();
532:   // search unused parameters beore backward starts
533:   void search_unused_parameters(
534:       const std::vector<torch::autograd::Variable>& outputs);
535:   void set_divide_factor();
536:   // kick off all reduce for the ready bucket
537:   void all_reduce_bucket(Bucket& bucket);
538:   // kick off all reduce to local used map, it can help find global unused
539:   // parameters
540:   void all_reduce_local_used_map();
```

- EN: Lines 521-540 introduces executable logic in routines such as `reset_bucket_counting`, `search_unused_parameters`, `set_divide_factor`.
- CN: 第 521-540 行在 `reset_bucket_counting`、`search_unused_parameters`、`set_divide_factor` 等例程中引入具体执行逻辑。

### Lines 541-560 / 第 541-560 行

```cpp
541:   // initialize locally used parameter maps
542:   void initialize_local_used_map();
543:   // get current cuda stream
544:   const c10::Stream get_current_stream();
545:   bool dynamic_graph_find_unused();
546:   bool static_graph_first_iteration();
547:   bool static_graph_after_first_iteration();
548: 
549:   bool is_unused_bucket(Bucket& bucket);
550:   bool should_skip_all_reduce_bucket(Bucket& bucket);
551: 
552:   // comm_hook_ is used to access the DDP communication hook if registered.
553:   std::unique_ptr<CommHookInterface> comm_hook_;
554: 
555:   // Sparse metadata contains the indices that will be used
556:   // when calling into sparse allreduce.
557:   // This is only used in the sparse allreduce collective calls
558:   std::unique_ptr<std::map<std::string, at::Tensor>> sparse_metadata_;
559: 
560:   // Debug level setting. It is parsed once when Reducer is constructed, and
```

- EN: Lines 541-560 introduces executable logic in routines such as `initialize_local_used_map`, `get_current_stream`, `dynamic_graph_find_unused`.
- CN: 第 541-560 行在 `initialize_local_used_map`、`get_current_stream`、`dynamic_graph_find_unused` 等例程中引入具体执行逻辑。

### Lines 561-580 / 第 561-580 行

```cpp
561:   // remains the same across a single invocation of DDP training.
562:   DebugLevel ddp_debug_level_;
563:   // Mapping of variable index to fully qualified name of model to notify users
564:   // about errors when certain parameters do not get gradient.
565:   std::unordered_map<size_t, std::string> param_names_;
566:   // Variable indices stored sequentially in order of when the gradient is ready
567:   // for the current backwards pass.
568:   std::vector<int64_t> grad_ready_order_indices_;
569:   // Bytes capacity of first bucket, can be configured by user
570:   int64_t first_bucket_bytes_cap_;
571:   // Per iteration set of parameter indices that have been marked ready.
572:   std::unordered_set<size_t> perIterationReadyParams_;
573:   // Retrieves parameter names that have not been marked as ready as part of
574:   // previous iteration.
575:   std::vector<std::string> getUnmarkedParamsForIteration();
576:   // Retrieves parameter indices that have not been marked as ready as part of
577:   // previous iteration.
578:   std::vector<size_t> getUnmarkedParamIndicesForIteration();
579:   // Raises appropriate error if mark_variable_ready is called on the same
580:   // variable twice, which is unexpected.
```

- EN: Lines 561-580 introduces executable logic in routines such as `getUnmarkedParamsForIteration`, `getUnmarkedParamIndicesForIteration`.
- CN: 第 561-580 行在 `getUnmarkedParamsForIteration`、`getUnmarkedParamIndicesForIteration` 等例程中引入具体执行逻辑。

### Lines 581-600 / 第 581-600 行

```cpp
581:   void checkAndRaiseMarkedTwiceError(size_t curVariableIndex);
582:   // Retrieves parameter corresponding to the given VariableIndex.
583:   at::Tensor& get_param_from_index(size_t index);
584:   // Python reducer keeps C++ reducer initialized. To remove this flag,
585:   // we need to refactor the DDP wrapper's initialization.
586:   bool use_python_reducer_;
587: 
588:   const std::vector<int64_t> bucket_bytes_cap_list_;
589: 
590:   // Cached bucket index to model parameter mapping. Populated after buckets
591:   // are rebuilt after which this mapping is static.
592:   mutable std::unordered_map<size_t, std::vector<at::Tensor>>
593:       cached_variables_for_bucket_;
594: 
595:   bool optim_in_backward_{false};
596:   friend class Logger;
597: };
598: 
599: // This is equivalent to take_tensors but returns indices into the
600: // tensor list argument for bucket assignment. Also, it is aware
```

- EN: Lines 581-600 introduces executable logic in routines such as `checkAndRaiseMarkedTwiceError`, `get_param_from_index`.
- CN: 第 581-600 行在 `checkAndRaiseMarkedTwiceError`、`get_param_from_index` 等例程中引入具体执行逻辑。

### Lines 601-619 / 第 601-619 行

```cpp
601: // of device placement and will not allow buckets to span devices.
602: // The index of tensors[i] assigned to bucket is tensor_indices[i],
603: // when tensor_indices is empty, the index of tensors[i] assigned to
604: // bucket is i.
605: TORCH_API std::tuple<std::vector<std::vector<size_t>>, std::vector<size_t>>
606: compute_bucket_assignment_by_size(
607:     const std::vector<at::Tensor>& tensors,
608:     const std::vector<size_t>& bucket_size,
609:     const std::vector<bool>& expect_sparse_gradient = {},
610:     const std::vector<int64_t>& tensor_indices = {},
611:     const std::optional<std::weak_ptr<c10d::Logger>>& logger = {});
612: 
613: // Verify models across all processes are the same as model on rank 0 with
614: // respect to no. of params and matching dtype/size/layout.
615: TORCH_API void verify_params_across_processes(
616:     const c10::intrusive_ptr<c10d::ProcessGroup>& process_group,
617:     const std::vector<at::Tensor>& params,
618:     const std::optional<std::weak_ptr<c10d::Logger>>& logger);
619: } // namespace c10d
```

- EN: Lines 601-619 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `verify_params_across_processes`.
- CN: 第 601-619 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `verify_params_across_processes` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `Logger`, `BucketAccumulator`, `TORCH_API`, `Bucket`
- CN: 核心符号：`Logger`、`BucketAccumulator`、`TORCH_API`、`Bucket`
- EN: Notable themes: process-group orchestration, collective communication logic.
- CN: 值得关注的主题：进程组编排、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/ProcessGroup.hpp`, `torch/csrc/distributed/c10d/Utils.hpp`, `torch/csrc/distributed/c10d/comm.hpp`, `torch/csrc/distributed/c10d/debug.h`, `torch/csrc/distributed/c10d/default_comm_hooks.hpp`, `torch/csrc/distributed/c10d/reducer_timer.hpp`, `torch/csrc/distributed/autograd/context/context.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/core/ScalarType.h`, `ATen/core/ivalue_inl.h`, `c10/macros/Macros.h`, `c10/util/intrusive_ptr.h`, `torch/csrc/autograd/function.h`, `torch/csrc/autograd/profiler.h`, `torch/csrc/autograd/variable.h`
- External or system headers / 外部或系统头文件: `atomic`, `memory`, `mutex`, `tuple`, `unordered_map`, `vector`
- Local symbols / 本地符号: `Logger`, `BucketAccumulator`, `TORCH_API`, `Bucket`