# lazy_graph_executor.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/lazy_graph_executor.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-76
```cpp
 1 | #include <torch/csrc/lazy/core/lazy_graph_executor.h>
 2 | 
 3 | #include <ATen/ScalarOps.h>
 4 | #include <c10/util/Logging.h>
 5 | #include <c10/util/irange.h>
 6 | #include <torch/csrc/jit/jit_log.h>
 7 | #include <torch/csrc/lazy/core/config.h>
 8 | #include <torch/csrc/lazy/core/internal_ops/ltc_ops.h>
 9 | #include <torch/csrc/lazy/core/ir_dump_util.h>
10 | #include <torch/csrc/lazy/core/ir_util.h>
11 | #include <torch/csrc/lazy/core/tensor_util.h>
12 | #include <torch/csrc/lazy/core/unique.h>
13 | 
14 | #include <torch/csrc/lazy/core/debug_util.h>
15 | #include <torch/csrc/lazy/core/ir_builder.h>
16 | #include <torch/csrc/lazy/core/metrics.h>
17 | #include <torch/csrc/lazy/core/ops/arithmetic_ir_ops.h>
18 | #include <torch/csrc/lazy/core/thread_pool.h>
19 | 
20 | #include <utility>
21 | 
22 | namespace torch::lazy {
23 | namespace {
24 | 
25 | struct TlsData {
26 |   void Reset() {
27 |     trim_counter = 0;
28 |   }
29 | 
30 |   size_t trim_counter = 0;
31 | };
32 | 
33 | thread_local TlsData g_tls_data;
34 | 
35 | bool TensorCompare(const at::Tensor& t1, const at::Tensor& t2) {
36 |   if (t1.scalar_type() != t2.scalar_type() || t1.sizes() != t2.sizes()) {
37 |     return false;
38 |   }
39 |   // PyTorch currently has an issue comparing tensors which have NaN values in
40 |   // it. The compare is not deterministic. So we do memory compare here until
41 |   // the PyTorch equal() API is fixed.
42 |   at::Tensor contiguous_t1 = t1.contiguous();
43 |   at::Tensor contiguous_t2 = t2.contiguous();
44 |   return std::memcmp(
45 |              contiguous_t1.data_ptr(),
46 |              contiguous_t2.data_ptr(),
47 |              contiguous_t1.numel() * contiguous_t1.itemsize()) == 0;
48 | }
49 | 
50 | // Return true if any tensor in the list has an underlying IR (leaf or
51 | // operation).
52 | bool TensorsHaveIR(const std::vector<LazyTensorPtr>& tensors) {
53 |   for (const auto& tensor : tensors) {
54 |     if (tensor->CurrentDataHandle() || tensor->CurrentIrValue()) {
55 |       return true;
56 |     }
57 |   }
58 |   return false;
59 | }
60 | 
61 | std::atomic<LazyGraphExecutor*> lazy_graph_executor_registry;
62 | } // namespace
63 | 
64 | auto LazyGraphExecutor::DeviceContextArena::Get()
65 |     -> LazyGraphExecutor::DeviceContextArena* {
66 |   static DeviceContextArena* arena = new DeviceContextArena();
67 |   return arena;
68 | }
69 | 
70 | void LazyGraphExecutor::DeviceContextArena::RegisterTensor(
71 |     std::shared_ptr<LazyTensor::Data> data) {
72 |   DeviceContext* devctx = GetDeviceContext(data->device);
73 |   std::lock_guard<std::mutex> lock(devctx->lock);
74 |   devctx->tensors_data.emplace(data->unique_id, data);
75 | }
76 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/core/lazy_graph_executor.h>`, `<ATen/ScalarOps.h>`, `<c10/util/Logging.h>`, `<c10/util/irange.h>` and system or third-party headers such as `<utility>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `TlsData` that structure the state handled by this file. Implements routines such as `Reset`, `TensorCompare`, `TensorsHaveIR`, `lock` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/core/lazy_graph_executor.h>`、`<ATen/ScalarOps.h>`、`<c10/util/Logging.h>`、`<c10/util/irange.h>`以及系统或第三方头文件，例如 `<utility>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `TlsData` 等数据抽象，用来组织本文件处理的状态。 实现了 `Reset`、`TensorCompare`、`TensorsHaveIR`、`lock` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 77-147
```cpp
 77 | void LazyGraphExecutor::DeviceContextArena::UnregisterTensor(
 78 |     LazyTensor::Data* data) {
 79 |   DeviceContext* devctx = GetDeviceContext(data->device);
 80 |   std::lock_guard<std::mutex> lock(devctx->lock);
 81 |   devctx->tensors_data.erase(data->unique_id);
 82 | }
 83 | 
 84 | std::vector<LazyTensorPtr> LazyGraphExecutor::DeviceContextArena::
 85 |     GetLiveTensors(const BackendDevice* device) {
 86 |   std::vector<LazyTensorPtr> tensors;
 87 |   auto fn = [&](DeviceContext* devctx) {
 88 |     std::lock_guard<std::mutex> lock(devctx->lock);
 89 |     for (auto& uid_wptr : devctx->tensors_data) {
 90 |       std::shared_ptr<LazyTensor::Data> data = uid_wptr.second.lock();
 91 |       if (data != nullptr) {
 92 |         tensors.push_back(LazyTensor::Create(std::move(data)));
 93 |       }
 94 |     }
 95 |   };
 96 |   ForAllDeviceContexts(fn, device);
 97 |   return tensors;
 98 | }
 99 | 
100 | Value LazyGraphExecutor::DeviceContextArena::GetRngSeed(
101 |     const BackendDevice& device) {
102 |   static const at::ScalarType kSeedType = at::ScalarType::Long;
103 |   static const uint64_t kSeedMul = 214013;
104 |   static const uint64_t kSeedAdd = 2531011;
105 |   DeviceContext* devctx = GetDeviceContext(device);
106 |   std::lock_guard<std::mutex> lock(devctx->lock);
107 |   if (!devctx->seed_ir_value) {
108 |     devctx->seed_ir_value =
109 |         IrValueFromScalar(MakeIntScalar(devctx->seed), kSeedType, device);
110 |   }
111 |   // Keep the running seed as scalar as well, so we can return it directly
112 |   // without executing graphs.
113 |   devctx->running_seed = kSeedAdd + kSeedMul * devctx->running_seed;
114 |   // Compose new seeds from the root seed, to avoid creating too many
115 |   // computation parameters which might overflow the device capacity.
116 |   Value k = MakeScalar(MakeIntScalar(kSeedMul), kSeedType);
117 |   Value b = MakeScalar(MakeIntScalar(kSeedAdd), kSeedType);
118 |   devctx->seed_ir_value = b + k * devctx->seed_ir_value;
119 |   return devctx->seed_ir_value;
120 | }
121 | 
122 | uint64_t LazyGraphExecutor::DeviceContextArena::GetRunningSeed(
123 |     const BackendDevice& device) {
124 |   DeviceContext* devctx = GetDeviceContext(device);
125 |   std::lock_guard<std::mutex> lock(devctx->lock);
126 |   return devctx->running_seed;
127 | }
128 | 
129 | void LazyGraphExecutor::DeviceContextArena::SetRngSeed(
130 |     const BackendDevice& device,
131 |     uint64_t seed) {
132 |   DeviceContext* devctx = GetDeviceContext(device);
133 |   std::lock_guard<std::mutex> lock(devctx->lock);
134 |   devctx->seed = seed;
135 |   devctx->running_seed = devctx->seed;
136 |   devctx->seed_ir_value = Value();
137 | }
138 | 
139 | void LazyGraphExecutor::DeviceContextArena::MarkStep(
140 |     const BackendDevice& device) {
141 |   DeviceContext* devctx = GetDeviceContext(device);
142 |   std::lock_guard<std::mutex> lock(devctx->lock);
143 |   devctx->seed = 1012031 + devctx->seed * 7012063;
144 |   devctx->running_seed = devctx->seed;
145 |   devctx->seed_ir_value = Value();
146 | }
147 | 
```
- EN: Implements routines such as `lock`, `GetLiveTensors` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `lock`、`GetLiveTensors` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 148-221
```cpp
148 | std::vector<BackendDevice> LazyGraphExecutor::DeviceContextArena::
149 |     GetActiveDevices() {
150 |   std::vector<BackendDevice> active_devices;
151 |   std::lock_guard<std::mutex> lock(lock_);
152 |   active_devices.reserve(device_contexts_.size());
153 |   for (auto& device_contexts : device_contexts_) {
154 |     active_devices.push_back(device_contexts.first);
155 |   }
156 |   return active_devices;
157 | }
158 | 
159 | auto LazyGraphExecutor::DeviceContextArena::GetAllDeviceContexts()
160 |     -> std::vector<DeviceContext*> {
161 |   std::vector<DeviceContext*> all_device_contexts;
162 |   std::lock_guard<std::mutex> lock(lock_);
163 |   all_device_contexts.reserve(device_contexts_.size());
164 |   for (auto& device_contexts : device_contexts_) {
165 |     all_device_contexts.push_back(device_contexts.second);
166 |   }
167 |   return all_device_contexts;
168 | }
169 | 
170 | void LazyGraphExecutor::DeviceContextArena::ForAllDeviceContexts(
171 |     const std::function<void(DeviceContext*)>& fn,
172 |     const BackendDevice* device) {
173 |   if (device == nullptr) {
174 |     for (auto devctx : GetAllDeviceContexts()) {
175 |       fn(devctx);
176 |     }
177 |   } else {
178 |     fn(GetDeviceContext(*device));
179 |   }
180 | }
181 | 
182 | auto LazyGraphExecutor::DeviceContextArena::GetDeviceContext(
183 |     const BackendDevice& device) -> DeviceContext* {
184 |   std::lock_guard<std::mutex> lock(lock_);
185 |   auto it = device_contexts_.find(device);
186 |   if (it == device_contexts_.end()) {
187 |     it = device_contexts_.emplace(device, new DeviceContext()).first;
188 |   }
189 |   return it->second;
190 | }
191 | 
192 | Value LazyGraphExecutor::DeviceContextArena::IrValueFromScalar(
193 |     const at::Scalar& value,
194 |     at::ScalarType scalar_type,
195 |     const BackendDevice& device) {
196 |   at::Tensor tensor = at::scalar_tensor(value, at::TensorOptions(scalar_type));
197 |   BackendDataPtr device_data = TensorToDataHandle(tensor, device);
198 |   return MakeDeviceData(device_data);
199 | }
200 | 
201 | void LazyGraphExecutor::DeviceLocker::Lock() {
202 |   std::unique_lock<std::mutex> lock(mutex_);
203 |   cv_.wait(lock, [this] { return !locked_; });
204 |   CheckResetException();
205 |   locked_ = true;
206 | }
207 | 
208 | void LazyGraphExecutor::DeviceLocker::Unlock(std::exception_ptr exptr) {
209 |   std::lock_guard<std::mutex> lock(mutex_);
210 |   locked_ = false;
211 |   exptr_ = std::move(exptr);
212 |   cv_.notify_all();
213 | }
214 | 
215 | void LazyGraphExecutor::DeviceLocker::Barrier() {
216 |   std::unique_lock<std::mutex> lock(mutex_);
217 |   cv_.wait(lock, [this] { return !locked_; });
218 |   cv_.notify_all();
219 |   CheckResetException();
220 | }
221 | 
```
- EN: Implements routines such as `GetActiveDevices`, `lock`, `MakeDeviceData` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 实现了 `GetActiveDevices`、`lock`、`MakeDeviceData` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 222-301
```cpp
222 | void LazyGraphExecutor::DeviceLocker::CheckResetException() {
223 |   std::exception_ptr exptr = std::move(exptr_);
224 |   exptr_ = nullptr;
225 |   if (exptr != nullptr) {
226 |     std::rethrow_exception(exptr);
227 |   }
228 | }
229 | 
230 | auto LazyGraphExecutor::DeviceLockerArena::Get() -> DeviceLockerArena* {
231 |   static DeviceLockerArena* arena = new DeviceLockerArena();
232 |   return arena;
233 | }
234 | 
235 | auto LazyGraphExecutor::DeviceLockerArena::GetLocker(
236 |     const BackendDevice& device) -> std::shared_ptr<DeviceLocker> {
237 |   std::lock_guard<std::mutex> lock(mutex_);
238 |   auto it = lockers_.find(device);
239 |   if (it == lockers_.end()) {
240 |     it = lockers_.emplace(device, std::make_shared<DeviceLocker>(device)).first;
241 |   }
242 |   return it->second;
243 | }
244 | 
245 | void LazyGraphExecutor::DeviceLockerArena::DeviceBarrier(
246 |     const BackendDevice& device) {
247 |   auto locker = DeviceLockerArena::Get()->GetLocker(device);
248 |   locker->Barrier();
249 | }
250 | 
251 | std::vector<ExceptionCleanup> LazyGraphExecutor::DeviceLockerArena::LockDevices(
252 |     const std::set<BackendDevice>& devices) {
253 |   std::vector<ExceptionCleanup> unlocker;
254 |   unlocker.reserve(devices.size());
255 |   for (auto& device : devices) {
256 |     unlocker.emplace_back(LockDevice(device));
257 |   }
258 |   return unlocker;
259 | }
260 | 
261 | ExceptionCleanup LazyGraphExecutor::DeviceLockerArena::LockDevice(
262 |     const BackendDevice& device) {
263 |   VLOG(4) << "Waiting on device barrier for device " << device << " ...";
264 |   std::shared_ptr<DeviceLocker> locker;
265 |   {
266 |     TORCH_LAZY_TIMED("DeviceLockWait");
267 |     locker = DeviceLockerArena::Get()->GetLocker(device);
268 |     locker->Lock();
269 |   }
270 |   VLOG(4) << "Waiting on device barrier for device " << device << " done!";
271 |   return torch::lazy::ExceptionCleanup(
272 |       [locker = std::move(locker)](
273 |           torch::lazy::ExceptionCleanup::StatusType status) {
274 |         locker->Unlock(std::move(status));
275 |       });
276 | }
277 | 
278 | auto LazyGraphExecutor::DataCacheArena::Get() -> DataCacheArena* {
279 |   static DataCacheArena* arena =
280 |       new DataCacheArena(FLAGS_torch_lazy_device_data_cache_size);
281 |   return arena;
282 | }
283 | 
284 | LazyGraphExecutor::DataCacheArena::DataCacheArena(size_t max_cache_size)
285 |     : max_cache_size_(max_cache_size) {}
286 | 
287 | BackendDataPtr LazyGraphExecutor::DataCacheArena::GetDeviceData(
288 |     const at::Tensor& tensor,
289 |     const BackendDevice& device) {
290 |   DataCacheArena::DataCache* cache = Get()->GetDataCache(device);
291 |   ;
292 |   BackendDataPtr device_data = cache->Get(tensor);
293 |   if (device_data == nullptr) {
294 |     at::Tensor tensor_copy = CopyTensor(tensor);
295 |     device_data = TensorToDataHandle(tensor_copy, device);
296 |     cache->Add(std::move(tensor_copy), device_data);
297 |     TORCH_LAZY_COUNTER("DeviceDataCacheMiss", 1);
298 |   }
299 |   return device_data;
300 | }
301 | 
```
- EN: Implements routines such as `lock`, `DataCacheArena` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 实现了 `lock`、`DataCacheArena` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 302-378
```cpp
302 | BackendDataPtr LazyGraphExecutor::DataCacheArena::GetDeviceData(
303 |     const at::Scalar& value,
304 |     at::ScalarType scalar_type,
305 |     const BackendDevice& device) {
306 |   // Workaround since at::scalar_tensor doesn't support bfloat16 yet.
307 |   at::Tensor t = at::scalar_tensor(
308 |       value,
309 |       at::TensorOptions(
310 |           scalar_type == at::ScalarType::BFloat16 ? at::ScalarType::Float
311 |                                                   : scalar_type));
312 |   if (scalar_type == at::ScalarType::BFloat16) {
313 |     t = t.to(scalar_type);
314 |   }
315 |   return GetDeviceData(t, device);
316 | }
317 | 
318 | size_t LazyGraphExecutor::DataCacheArena::TensorHasher::operator()(
319 |     const at::Tensor& tensor) const {
320 |   return HashReduce(
321 |       HashCombine(GetEnumValue(tensor.scalar_type()), TensorHash(tensor)));
322 | }
323 | 
324 | bool LazyGraphExecutor::DataCacheArena::TensorComparer::operator()(
325 |     const at::Tensor& tensor1,
326 |     const at::Tensor& tensor2) const {
327 |   return TensorCompare(tensor1, tensor2);
328 | }
329 | 
330 | auto LazyGraphExecutor::DataCacheArena::GetDataCache(
331 |     const BackendDevice& device) -> DataCache* {
332 |   std::lock_guard<std::mutex> lock(mutex_);
333 |   if (FLAGS_torch_lazy_enable_device_data_cache) {
334 |     auto it = device_caches_.find(device);
335 |     if (it == device_caches_.end()) {
336 |       it = device_caches_
337 |                .emplace(device, std::make_unique<DataCache>(max_cache_size_))
338 |                .first;
339 |     }
340 |     return it->second.get();
341 |   } else {
342 |     // If cache is disabled then always return a zero size cache
343 |     static DataCache s_empty_cache(0);
344 |     return &s_empty_cache;
345 |   }
346 | }
347 | 
348 | void LazyGraphExecutor::Register(LazyGraphExecutor* executor) {
349 |   lazy_graph_executor_registry.store(executor);
350 | }
351 | LazyGraphExecutor* LazyGraphExecutor::Get() {
352 |   auto* executor = lazy_graph_executor_registry.load();
353 |   TORCH_CHECK(executor, "Lazy graph executor not registered.");
354 |   return executor;
355 | }
356 | 
357 | void LazyGraphExecutor::RegisterTensor(std::shared_ptr<LazyTensor::Data> data) {
358 |   DeviceContextArena::Get()->RegisterTensor(std::move(data));
359 |   TORCH_LAZY_COUNTER("CreateLtcTensor", 1);
360 | }
361 | 
362 | void LazyGraphExecutor::UnregisterTensor(LazyTensor::Data* data) {
363 |   DeviceContextArena::Get()->UnregisterTensor(data);
364 |   TORCH_LAZY_COUNTER("DestroyLtcTensor", 1);
365 | }
366 | 
367 | Value LazyGraphExecutor::GetRngSeed(const BackendDevice& device) {
368 |   return DeviceContextArena::Get()->GetRngSeed(device);
369 | }
370 | 
371 | uint64_t LazyGraphExecutor::GetRunningSeed(const BackendDevice& device) {
372 |   return DeviceContextArena::Get()->GetRunningSeed(device);
373 | }
374 | 
375 | void LazyGraphExecutor::SetRngSeed(const BackendDevice& device, uint64_t seed) {
376 |   DeviceContextArena::Get()->SetRngSeed(device, seed);
377 | }
378 | 
```
- EN: Implements routines such as `GetDeviceData`, `HashReduce`, `TensorCompare`, `lock`, `s_empty_cache` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 实现了 `GetDeviceData`、`HashReduce`、`TensorCompare`、`lock`、`s_empty_cache` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 379-458
```cpp
379 | void LazyGraphExecutor::DeviceBarrier(const BackendDevice& device) {
380 |   DeviceLockerArena::Get()->DeviceBarrier(device);
381 | }
382 | 
383 | BackendDataPtr LazyGraphExecutor::GetDeviceData(
384 |     const at::Tensor& tensor,
385 |     const BackendDevice& device) {
386 |   return DataCacheArena::Get()->GetDeviceData(tensor, device);
387 | }
388 | 
389 | BackendDataPtr LazyGraphExecutor::GetDeviceData(
390 |     const at::Scalar& value,
391 |     at::ScalarType scalar_type,
392 |     const BackendDevice& device) {
393 |   return DataCacheArena::Get()->GetDeviceData(value, scalar_type, device);
394 | }
395 | 
396 | std::vector<LazyTensorPtr> LazyGraphExecutor::GetLiveTensors(
397 |     const BackendDevice* device) {
398 |   return DeviceContextArena::Get()->GetLiveTensors(device);
399 | }
400 | 
401 | void LazyGraphExecutor::SyncLiveTensorsGraph(
402 |     const BackendDevice* device,
403 |     c10::ArrayRef<std::string> devices,
404 |     bool wait) {
405 |   auto tensors = GetLiveTensors(device);
406 |   VLOG(4) << tensors.size() << " live tensors: devices=("
407 |           << c10::Join(", ", devices) << ')';
408 |   SyncTensorsGraph(&tensors, devices, wait, /*sync_ltc_data=*/true);
409 | }
410 | 
411 | void LazyGraphExecutor::SyncTensorsGraph(
412 |     std::vector<LazyTensorPtr>* tensors,
413 |     c10::ArrayRef<std::string> devices,
414 |     bool wait,
415 |     bool sync_ltc_data) {
416 |   VLOG(4) << "Trying to sync the value of " << tensors->size() << " tensor(s)";
417 |   SyncTensorsConfig config;
418 |   config.sync_ltc_data = sync_ltc_data;
419 | 
420 |   auto async = SyncTensorsGraphInternal(tensors, devices, config);
421 |   if (FLAGS_torch_lazy_use_thread_pool && wait && async != nullptr) {
422 |     async->mwait.Wait();
423 |   }
424 | }
425 | 
426 | void LazyGraphExecutor::MarkStep(const BackendDevice& device) {
427 |   TORCH_LAZY_COUNTER("MarkStep", 1);
428 |   DeviceContextArena::Get()->MarkStep(device);
429 |   ScopePusher::ResetScopes();
430 |   ResetTrimCounter();
431 |   // Move TrieCache's current pointer back to its root
432 |   TrieCache::Get()->ResetCurrent();
433 | }
434 | 
435 | void LazyGraphExecutor::WaitDeviceOps(c10::ArrayRef<BackendDevice> devices) {
436 |   std::set<BackendDevice> wait_devices;
437 |   if (!devices.empty()) {
438 |     for (auto& device : devices) {
439 |       wait_devices.insert(device);
440 |     }
441 |   } else {
442 |     for (auto& device_str : DeviceContextArena::Get()->GetActiveDevices()) {
443 |       // TODO: Remove the last use of Device(const std::string& device_spec).
444 |       wait_devices.insert(BackendDevice(device_str));
445 |     }
446 |   }
447 |   // The LockDevices() API returns a vector of
448 |   // ExceptionCleanup object, which is going to be freed
449 |   // immediately, turning this operation into a lock barrier.
450 |   DeviceLockerArena::Get()->LockDevices(wait_devices);
451 | }
452 | 
453 | std::vector<at::Tensor> LazyGraphExecutor::GetTensors(
454 |     std::vector<LazyTensorPtr>* tensors) {
455 |   VLOG(4) << "Trying to get the value of " << tensors->size() << " tensor(s)";
456 |   return GetTensorsFused(tensors);
457 | }
458 | 
```
- EN: Implements routines such as `GetTensorsFused` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on.
- CN: 实现了 `GetTensorsFused` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态。

### Lines 459-533
```cpp
459 | void LazyGraphExecutor::ResetTrimCounter() const {
460 |   g_tls_data.Reset();
461 | }
462 | 
463 | size_t LazyGraphExecutor::IncTrimCounter() const {
464 |   return ++g_tls_data.trim_counter;
465 | }
466 | 
467 | std::string LazyGraphExecutor::DumpBackendComputation(
468 |     const std::vector<LazyTensorPtr>& tensors) {
469 |   std::vector<Value> ir_values;
470 |   for (auto& tensor : tensors) {
471 |     Value ir_value = tensor->CurrentIrValue();
472 |     if (ir_value) {
473 |       ir_values.push_back(std::move(ir_value));
474 |     }
475 |   }
476 |   return !ir_values.empty() ? DumpUtil::ToBackend(ir_values, BackendDevice())
477 |                             : std::string();
478 | }
479 | 
480 | Value LazyGraphExecutor::GetDeviceDataIrValue(
481 |     const at::Scalar& value,
482 |     c10::ScalarType type,
483 |     const BackendDevice& device) {
484 |   BackendDataPtr data = GetDeviceData(value, type, device);
485 |   data->SetInfo(std::make_shared<DeviceDataInfo>(
486 |       /*tensor_id=*/-1, /*read_only=*/true));
487 |   return MakeDeviceData(data);
488 | }
489 | 
490 | Value LazyGraphExecutor::GetIrValueForScalarFromCodegen(
491 |     const at::Scalar& value,
492 |     const BackendDevice& device) {
493 |   if (IsSpecialScalar(value)) {
494 |     return MakeScalar(value, value.type());
495 |   }
496 |   auto data = GetDeviceData(value, value.type(), device);
497 |   data->SetInfo(
498 |       std::make_shared<DeviceDataInfo>(/*tensor_id=*/-1, /*read_only=*/true));
499 |   return MakeDeviceData(data);
500 | }
501 | 
502 | Value LazyGraphExecutor::GetIrValueForScalar(
503 |     const at::Scalar& value,
504 |     c10::ScalarType type,
505 |     const BackendDevice& device) {
506 |   if (IsSpecialScalar(value)) {
507 |     return MakeScalar(value, type);
508 |   }
509 |   return GetDeviceDataIrValue(value, type, device);
510 | }
511 | 
512 | Value LazyGraphExecutor::GetIrValueForScalar(
513 |     const at::Scalar& value,
514 |     const BackendDevice& device) {
515 |   return GetIrValueForScalar(value, value.type(), device);
516 | }
517 | 
518 | Value LazyGraphExecutor::GetIrValueForExpandedScalar(
519 |     const at::Scalar& value,
520 |     const Shape& shape,
521 |     const BackendDevice& device) {
522 |   c10::ArrayRef<int64_t> dimensions = shape.sizes();
523 |   auto type = shape.scalar_type();
524 |   Value ir_value = GetIrValueForScalar(value, type, device);
525 |   if (!dimensions.empty()) {
526 |     ir_value = MakeExpand(
527 |         ir_value,
528 |         dimensions.vec(),
529 |         /*is_scalar_expand=*/true);
530 |   }
531 |   return ir_value;
532 | }
533 | 
```
- EN: Implements routines such as `MakeDeviceData`, `MakeScalar`, `GetDeviceDataIrValue`, `GetIrValueForScalar` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `MakeDeviceData`、`MakeScalar`、`GetDeviceDataIrValue`、`GetIrValueForScalar` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 534-597
```cpp
534 | LazyGraphExecutor::Async::Async(
535 |     SyncTensorCollection* coll,
536 |     std::vector<BackendDataPtr> parameters_data,
537 |     std::vector<BackendDataPtr> tensors_data,
538 |     ComputationCache::TypePtr cached_computation)
539 |     : mwait(1),
540 |       indices(std::move(coll->indices)),
541 |       unlocker(std::move(coll->unlocker)),
542 |       parameters_data(std::move(parameters_data)),
543 |       device(coll->device),
544 |       cached_computation(std::move(cached_computation)),
545 |       tensors_data(std::move(tensors_data)) {}
546 | 
547 | void LazyGraphExecutor::Async::Wait() {
548 |   mwait.Wait();
549 |   // Accessing other Async members is safe only after MultiWait::Wait()
550 |   // completes.
551 |   ExceptionCleanup::StatusType status;
552 |   for (auto& cleanup : unlocker) {
553 |     const ExceptionCleanup::StatusType& cleanup_status = cleanup.GetStatus();
554 |     if (cleanup_status != nullptr) {
555 |       if (status == nullptr) {
556 |         status = cleanup_status;
557 |       }
558 |       // If we observe the status here, no need to let it propagate to the next
559 |       // device lock operation.
560 |       cleanup.SetStatus(nullptr);
561 |     }
562 |   }
563 |   if (status != nullptr) {
564 |     std::rethrow_exception(status);
565 |   }
566 | }
567 | 
568 | bool LazyGraphExecutor::ShouldSyncTensor(const LazyTensorPtr& tensor) const {
569 |   return tensor->GetIrValue()->op() != ltc_not_supported;
570 | }
571 | 
572 | LazyGraphExecutor::SyncTensorCollection LazyGraphExecutor::CollectSyncTensors(
573 |     const std::vector<LazyTensorPtr>& tensors,
574 |     const SyncTensorsConfig& config) {
575 |   Unique<BackendDevice> unique_device;
576 |   for (const auto& tensor : tensors) {
577 |     unique_device.set(tensor->GetDevice());
578 |   }
579 |   SyncTensorCollection coll;
580 |   if (!unique_device) {
581 |     return coll;
582 |   }
583 |   if (!config.force_ltc_data && !TensorsHaveIR(tensors)) {
584 |     return coll;
585 |   }
586 | 
587 |   std::vector<at::Tensor> at_tensors;
588 |   std::vector<BackendDevice> devices;
589 |   std::vector<size_t> at_tensor_index;
590 |   std::unordered_set<int64_t> tensor_ids;
591 |   // The force_ltc_data controls aliasing compilation, so effectively the same
592 |   // graph with on/off force_ltc_data should not match, hash wise.
593 |   coll.hash = MHash(config.force_ltc_data);
594 |   coll.config = config;
595 |   coll.device = *unique_device;
596 |   coll.indices.reserve(tensors.size());
597 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 598-669
```cpp
598 |   for (const auto i : c10::irange(tensors.size())) {
599 |     if (tensor_ids.insert(tensors[i]->GetUniqueId()).second &&
600 |         tensors[i]->CurrentDataHandle() == nullptr) {
601 |       Value ir_value = tensors[i]->CurrentIrValue();
602 |       if (ir_value) {
603 |         if (ShouldSyncTensor(tensors[i])) {
604 |           TORCH_LAZY_COUNTER("SyncedTensorsWithIR", 1);
605 |           // Add only tensors which need to be synced.
606 |           coll.hash = HashCombine(coll.hash, ir_value.hash());
607 |           coll.indices.push_back(i);
608 |         }
609 |       } else if (config.force_ltc_data) {
610 |         // The tensor only has at::Tensor data. We need to queue it for a
611 |         // device upload.
612 |         std::optional<at::Tensor> tensor_data = tensors[i]->CurrentTensorData();
613 |         TORCH_CHECK(tensor_data);
614 |         at_tensors.push_back(*tensor_data);
615 |         devices.push_back(tensors[i]->GetDevice());
616 |         at_tensor_index.push_back(i);
617 |       }
618 |     }
619 |   }
620 |   if (!at_tensors.empty()) {
621 |     TORCH_LAZY_COUNTER("SyncTensorsToData", at_tensors.size());
622 |     std::vector<BackendDataPtr> handles =
623 |         CreateTensorsData(at_tensors, devices);
624 |     for (const auto i : c10::irange(handles.size())) {
625 |       // If we are here, it means that the IR Value for the tensor is not
626 |       // present. Also, we uploaded the at::Tensor data to the device, but such
627 |       // data is still valid so we leave it live on the lazy tensor (so that a
628 |       // following ToTensor() does not need to fetch it from device).
629 |       tensors[at_tensor_index[i]]->data()->handle = std::move(handles[i]);
630 |     }
631 |   }
632 |   VLOG(4) << "Tensors graph hash " << HashToString(coll.hash) << " on device "
633 |           << coll.device;
634 |   return coll;
635 | }
636 | 
637 | std::vector<Value> LazyGraphExecutor::CollectRoots(
638 |     const std::vector<LazyTensorPtr>& tensors,
639 |     c10::ArrayRef<size_t> indices) {
640 |   std::vector<Value> roots;
641 |   roots.reserve(indices.size());
642 |   for (auto index : indices) {
643 |     roots.push_back(tensors.at(index)->CurrentIrValue());
644 |   }
645 |   return roots;
646 | }
647 | 
648 | void LazyGraphExecutor::ExtractIRAndPrepareTensorData(
649 |     std::vector<LazyTensorPtr>* tensors,
650 |     const SyncTensorsConfig& config,
651 |     c10::ArrayRef<size_t> indices,
652 |     std::vector<Value>& ir_values,
653 |     std::vector<BackendDataPtr>& tensor_data_vec) {
654 |   ir_values.reserve(indices.size());
655 |   tensor_data_vec.reserve(indices.size());
656 |   for (auto index : indices) {
657 |     LazyTensorPtr& tensor = (*tensors)[index];
658 |     Value ir_value = tensor->CurrentIrValue();
659 |     ir_values.push_back(ir_value);
660 |     const BackendDevice& tensor_device = tensor->GetDevice();
661 |     BackendDataPtr handle = getBackend()->CreateDataPlaceholder(
662 |         tensor_device, std::move(tensor->shape()));
663 |     tensor_data_vec.push_back(handle);
664 |     if (tensor->CurrentDataHandle() == nullptr && config.sync_ltc_data) {
665 |       tensor->AssignIrValue(Value());
666 |     }
667 |   }
668 | }
669 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 670-735
```cpp
670 | std::vector<torch::lazy::BackendDataPtr> LazyGraphExecutor::SetTensorData(
671 |     std::vector<LazyTensorPtr>* tensors,
672 |     const SyncTensorsConfig& config,
673 |     c10::ArrayRef<size_t> indices,
674 |     const std::vector<BackendDataPtr>& tensor_data_vec) {
675 |   std::vector<BackendDataPtr> tensors_data;
676 |   tensors_data.reserve(indices.size());
677 |   for (const auto i : c10::irange(indices.size())) {
678 |     auto index = indices[i];
679 |     LazyTensorPtr& tensor = (*tensors)[index];
680 |     // If the config.force_ltc_data flag is true, the purpose of this tensor
681 |     // sync operation is to truncate the IR graph and materialize device data in
682 |     // place of IR graph, on selected tensors. But since operation will complete
683 |     // asynchronously, if a tensor does not already have device data, we need to
684 |     // install a placeholder. Since at this point we hold a lock on the device
685 |     // where the tensors reside (locks held within the coll structure, and moved
686 |     // into the async variable), any other operation trying to access the
687 |     // tensor's device data will have to wait until the asynchronous operation
688 |     // completes.
689 |     BackendDataPtr handle = tensor->CurrentDataHandle();
690 |     if (handle == nullptr && config.force_ltc_data) {
691 |       handle = tensor_data_vec[i];
692 |       // Note: We are not using SetHandleData method here since that method
693 |       // resets the ir_value. We have already done the resetting as part
694 |       // of ExtractIRAndPrepareTensorData to overlap with previous execution.
695 |       tensor->data()->handle = handle;
696 |       tensor->data()->tensor_data = std::nullopt;
697 |     }
698 |     tensors_data.emplace_back(std::move(handle));
699 |   }
700 |   return tensors_data;
701 | }
702 | 
703 | LazyGraphExecutor::PostOrderData LazyGraphExecutor::RunPostOrder(
704 |     const std::vector<Value>& ir_values,
705 |     SyncTensorCollection* coll) {
706 |   std::vector<const Node*> roots;
707 |   roots.reserve(ir_values.size());
708 |   for (const auto& ir_value : ir_values) {
709 |     roots.push_back(ir_value.node.get());
710 |   }
711 |   PostOrderData po_data;
712 |   po_data.post_order = Util::ComputePostOrder(roots, &po_data.emission_map);
713 |   std::unordered_map<BackendData::Handle, size_t> data_handles;
714 |   for (auto node : po_data.post_order) {
715 |     const auto backend_data = getBackend()->GetComputationDataFromNode(node);
716 |     if (backend_data) {
717 |       /* Acceptable race condition: HasValue may return false. This is OK
718 |        * since the conditional barrier is a performance optimization. */
719 |       if (!backend_data->HasValue()) {
720 |         TensorCollectionBarrier(coll);
721 |       }
722 |       BackendData::Handle handle = backend_data->GetHandle();
723 |       auto it = data_handles.find(handle);
724 |       if (it != data_handles.end()) {
725 |         po_data.parameter_sequence.push_back(it->second);
726 |       } else {
727 |         po_data.parameter_sequence.push_back(po_data.parameters_data.size());
728 |         data_handles[handle] = po_data.parameters_data.size();
729 |         po_data.parameters_data.push_back(backend_data);
730 |       }
731 |     }
732 |   }
733 |   return po_data;
734 | }
735 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 736-808
```cpp
736 | std::shared_ptr<LazyGraphExecutor::Async> LazyGraphExecutor::TryRunCachedSync(
737 |     std::vector<LazyTensorPtr>* tensors,
738 |     SyncTensorCollection* coll,
739 |     PostOrderData* po_data,
740 |     const std::vector<BackendDataPtr>& tensor_data_vec) {
741 |   ComputationCache::TypePtr cached_computation =
742 |       LookupCachedCompile(coll->hash);
743 |   if (cached_computation == nullptr) {
744 |     return nullptr;
745 |   }
746 |   if (GRAPH_DUMP_ENABLED) {
747 |     auto* comp = cached_computation->computation.get();
748 |     LOG(ERROR) << "Run a cached graph: " << comp->to_string() << '\n';
749 |   }
750 |   TORCH_LAZY_VALUE_METRIC("TensorsGraphSize", po_data->post_order.size());
751 |   VLOG(5) << "TensorsGraphSize=" << po_data->post_order.size();
752 | 
753 |   return ScheduleSyncTensorsGraph(
754 |       tensors,
755 |       coll,
756 |       std::move(po_data->parameters_data),
757 |       std::move(cached_computation),
758 |       tensor_data_vec);
759 | }
760 | 
761 | LazyGraphExecutor::CompilationResult LazyGraphExecutor::Compile(
762 |     const std::vector<LazyTensorPtr>& tensors,
763 |     c10::ArrayRef<std::string> devices,
764 |     const SyncTensorCollection& coll,
765 |     PostOrderData* po_data,
766 |     const std::vector<Value>& ir_values) {
767 |   auto lowering_ctx = LoweringContext::Create(
768 |       "SyncTensorsGraph",
769 |       coll.device,
770 |       po_data->post_order,
771 |       std::move(po_data->emission_map));
772 |   for (const auto& ir_value : ir_values) {
773 |     lowering_ctx->AddResult(ir_value);
774 |   }
775 | 
776 |   ComputationPtr computation = lowering_ctx->Build();
777 |   // If force_ltc_data is true it means that we did a proper sync and are
778 |   // inside a mark step. If GetTensors was called, force_ltc_data will
779 |   // be false meaning we are prematurely evaluating some value.
780 |   computation->in_mark_step = coll.config.force_ltc_data;
781 | 
782 |   VLOG(3) << "Compiling IR graph hash " << HashToString(coll.hash)
783 |           << " on device " << coll.device << " ...";
784 |   std::vector<ComputationPtr> computations =
785 |       getBackend()->Compile({computation});
786 |   VLOG(3) << "Compiling IR graph hash " << HashToString(coll.hash)
787 |           << " on device " << coll.device << " done!";
788 |   if (computation) {
789 |     // TODO(whc) should computation be allowed null here? (because it is in one
790 |     // case)
791 |     TORCH_CHECK(
792 |         computation->parameters_size() ==
793 |         static_cast<int>(po_data->parameters_data.size()));
794 |   }
795 | 
796 |   return {
797 |       /*device=*/coll.device,
798 |       /*emitted_nodes=*/lowering_ctx->GetEmittedNodeCount(),
799 |       /*computation=*/std::move(computations.front()),
800 |       /*parameters_data=*/std::move(po_data->parameters_data)};
801 | }
802 | 
803 | LazyGraphExecutor::ComputationCache* LazyGraphExecutor::GetComputationCache() {
804 |   static ComputationCache* cache =
805 |       new ComputationCache(FLAGS_torch_lazy_compilation_cache_size);
806 |   return cache;
807 | }
808 | 
```
- EN: Implements routines such as `ScheduleSyncTensorsGraph`, `ComputationCache` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `ScheduleSyncTensorsGraph`、`ComputationCache` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 809-888
```cpp
809 | LazyGraphExecutor::ComputationCache::TypePtr LazyGraphExecutor::
810 |     LookupCachedCompile(const hash_t& hash) {
811 |   ComputationCache::TypePtr cached_computation =
812 |       GetComputationCache()->Get(hash);
813 |   if (cached_computation == nullptr) {
814 |     TORCH_LAZY_COUNTER("UncachedCompile", 1);
815 |     return nullptr;
816 |   }
817 |   TORCH_LAZY_COUNTER("CachedCompile", 1);
818 |   return cached_computation;
819 | }
820 | 
821 | #if defined(_MSC_VER)
822 | #include <BaseTsd.h>
823 | typedef SSIZE_T ssize_t;
824 | #endif
825 | 
826 | std::shared_ptr<LazyGraphExecutor::Async> LazyGraphExecutor::
827 |     SyncTensorsGraphInternal(
828 |         std::vector<LazyTensorPtr>* tensors,
829 |         c10::ArrayRef<std::string> devices,
830 |         const SyncTensorsConfig& config) {
831 |   SyncTensorCollection coll = CollectSyncTensors(*tensors, config);
832 |   if (coll.indices.empty()) {
833 |     /* Ensure previous execution is complete before exiting this
834 |      * function */
835 |     TensorCollectionBarrier(&coll);
836 |     return nullptr;
837 |   }
838 |   DebugUtil::SaveTensorsGraphInfo(
839 |       "ScheduleSyncTensorsGraph", *tensors, &coll.indices);
840 |   std::vector<Value> ir_values;
841 |   std::vector<BackendDataPtr> tensor_data_vec;
842 |   ExtractIRAndPrepareTensorData(
843 |       tensors, coll.config, coll.indices, ir_values, tensor_data_vec);
844 |   PostOrderData po_data = RunPostOrder(ir_values, &coll);
845 |   coll.hash = HashCombine(coll.hash, Hash(po_data.parameter_sequence));
846 |   VLOG(4) << "Parameter sequence graph hash " << HashToString(coll.hash);
847 |   std::shared_ptr<Async> async =
848 |       TryRunCachedSync(tensors, &coll, &po_data, tensor_data_vec);
849 |   if (async != nullptr) {
850 |     return async;
851 |   }
852 | 
853 |   CompilationResult compile_result =
854 |       Compile(*tensors, devices, coll, &po_data, ir_values);
855 |   if (GRAPH_DUMP_ENABLED) {
856 |     auto* comp = compile_result.computation.get();
857 |     LOG(ERROR) << "Add a cached computation with hash " << coll.hash << '\n';
858 |     LOG(ERROR) << "Add a graph to cache: " << comp->to_string() << '\n';
859 |   }
860 | 
861 |   TORCH_LAZY_VALUE_METRIC("TensorsGraphSize", compile_result.emitted_nodes);
862 |   VLOG(5) << "TensorsGraphSize=" << compile_result.emitted_nodes;
863 | 
864 |   auto cached_computation = std::make_shared<CachedComputation>(
865 |       std::move(compile_result.computation));
866 |   GetComputationCache()->Add(coll.hash, cached_computation);
867 | 
868 |   return ScheduleSyncTensorsGraph(
869 |       tensors,
870 |       &coll,
871 |       std::move(compile_result.parameters_data),
872 |       std::move(cached_computation),
873 |       tensor_data_vec);
874 | }
875 | 
876 | std::shared_ptr<LazyGraphExecutor::Async> LazyGraphExecutor::
877 |     ScheduleSyncTensorsGraph(
878 |         SyncTensorCollection* coll,
879 |         std::vector<BackendDataPtr> parameters_data,
880 |         std::vector<BackendDataPtr> tensors_data,
881 |         ComputationCache::TypePtr cached_computation) {
882 |   TensorCollectionBarrier(coll);
883 |   std::shared_ptr<Async> async = std::make_shared<Async>(
884 |       coll,
885 |       std::move(parameters_data),
886 |       std::move(tensors_data),
887 |       std::move(cached_computation));
888 | 
```
- EN: Brings in system or third-party headers such as `<BaseTsd.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `LookupCachedCompile`, `SyncTensorsGraphInternal`, `ScheduleSyncTensorsGraph` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 这里引入了系统或第三方头文件，例如 `<BaseTsd.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `LookupCachedCompile`、`SyncTensorsGraphInternal`、`ScheduleSyncTensorsGraph` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 889-954
```cpp
889 |   auto syncfn = [async, hash = coll->hash]() {
890 |     try {
891 |       VLOG(3) << "Executing IR graph hash " << HashToString(hash)
892 |               << " on device " << async->device << " ...";
893 |       auto results = getBackend()->ExecuteComputation(
894 |           async->cached_computation->computation,
895 |           async->parameters_data,
896 |           async->device);
897 |       VLOG(3) << "Executing IR graph hash " << HashToString(hash)
898 |               << " on device " << async->device << " done!";
899 | 
900 |       TORCH_CHECK(
901 |           async->tensors_data.size() == results.size(),
902 |           "Expected number of outputs does not match TorchScript Stack size: ",
903 |           async->tensors_data.size(),
904 |           " != ",
905 |           results.size());
906 | 
907 |       for (const auto i : c10::irange(results.size())) {
908 |         if (async->tensors_data[i] != nullptr) {
909 |           async->tensors_data[i]->Assign(*results[i]);
910 |         } else {
911 |           async->tensors_data[i] = std::move(results[i]);
912 |         }
913 |       }
914 |     } catch (...) {
915 |       // There are two paths of discovery of an exception happening on an
916 |       // asynchronous task. One happens if the creator of the asynchronous task
917 |       // explicitly waits for completion, in which case the exception will be
918 |       // thrown from the Wait() API. Re-throwing the exception below makes sure
919 |       // this will be captured by the completer function created below, and
920 |       // surfaced by the Wait() API. But we also need to surface the exception
921 |       // even in case the caller does not wait, and that is accomplished by
922 |       // setting the unlockers status. In that case the exception will be
923 |       // surfaced when the user tries to acquire the device locks the next time.
924 |       for (auto& unlocker : async->unlocker) {
925 |         unlocker.SetStatus(std::current_exception());
926 |       }
927 |       throw;
928 |     }
929 |   };
930 | 
931 |   if (FLAGS_torch_lazy_use_thread_pool) {
932 |     ScheduleIoClosure(async->mwait.Completer(std::move(syncfn)));
933 |   } else {
934 |     syncfn();
935 |   }
936 |   return async;
937 | }
938 | 
939 | std::shared_ptr<LazyGraphExecutor::Async> LazyGraphExecutor::
940 |     ScheduleSyncTensorsGraph(
941 |         std::vector<LazyTensorPtr>* tensors,
942 |         SyncTensorCollection* coll,
943 |         std::vector<BackendDataPtr> parameters_data,
944 |         ComputationCache::TypePtr cached_computation,
945 |         const std::vector<BackendDataPtr>& tensor_data_vec) {
946 |   auto tensors_data =
947 |       SetTensorData(tensors, coll->config, coll->indices, tensor_data_vec);
948 |   return ScheduleSyncTensorsGraph(
949 |       coll,
950 |       std::move(parameters_data),
951 |       std::move(tensors_data),
952 |       std::move(cached_computation));
953 | }
954 | 
```
- EN: Implements routines such as `ScheduleSyncTensorsGraph` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `ScheduleSyncTensorsGraph` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 955-1026
```cpp
 955 | std::vector<at::Tensor> LazyGraphExecutor::GetTensorsFused(
 956 |     std::vector<LazyTensorPtr>* tensors) {
 957 |   SyncTensorsConfig config;
 958 |   config.force_ltc_data = false;
 959 |   auto async = SyncTensorsGraphInternal(tensors, {}, config);
 960 |   if (FLAGS_torch_lazy_use_thread_pool && async != nullptr) {
 961 |     async->mwait.Wait();
 962 |   }
 963 |   std::vector<BackendDataPtr> tensors_data = GatherTensorsData(
 964 |       *tensors,
 965 |       async != nullptr ? async->indices : c10::ArrayRef<size_t>(),
 966 |       async != nullptr ? async->tensors_data : c10::ArrayRef<BackendDataPtr>());
 967 |   return FetchTensors(
 968 |       tensors, tensors_data, async != nullptr ? &async->indices : nullptr);
 969 | }
 970 | 
 971 | // This gets tensors from the backend
 972 | // for TS backend, we'd ideally just cut through these layers and
 973 | // not need to copy the tensor, just move it
 974 | 
 975 | // for XLA backend, a copy is going to have to happen,
 976 | 
 977 | // could we replace the 'Data' object with an at::Tensor, which is 'undefined'
 978 | // unless a backend attaches a buffer to it?  That way we can have a
 979 | // 'PopulateTensor' method on backend, which can either attach an existing
 980 | // tensor buffer to the wrapper, or copy data?
 981 | std::vector<at::Tensor> LazyGraphExecutor::FetchTensors(
 982 |     std::vector<LazyTensorPtr>* tensors,
 983 |     c10::ArrayRef<BackendDataPtr> tensors_data,
 984 |     const std::vector<size_t>* indices) {
 985 |   std::vector<at::Tensor> results;
 986 |   size_t literals_index = 0;
 987 |   size_t sync_index = 0;
 988 |   results.reserve(tensors->size());
 989 |   for (const auto i : c10::irange(tensors->size())) {
 990 |     if (indices != nullptr && sync_index < indices->size() &&
 991 |         i == (*indices)[sync_index]) {
 992 |       results.push_back(getBackend()->MakeTensorFromComputationData(
 993 |           tensors_data[literals_index], (*tensors)[i]->dtype()));
 994 |       ++literals_index;
 995 |       ++sync_index;
 996 |     } else {
 997 |       std::optional<at::Tensor> tensor_data =
 998 |           (*tensors)[i]->CurrentTensorData();
 999 |       if (tensor_data) {
1000 |         results.push_back(*tensor_data);
1001 |       } else {
1002 |         TORCH_CHECK(literals_index < tensors_data.size());
1003 |         results.push_back(getBackend()->MakeTensorFromComputationData(
1004 |             tensors_data[literals_index], (*tensors)[i]->dtype()));
1005 |         ++literals_index;
1006 |       }
1007 |     }
1008 |   }
1009 |   return results;
1010 | }
1011 | 
1012 | std::vector<BackendDataPtr> LazyGraphExecutor::GatherTensorsData(
1013 |     const std::vector<LazyTensorPtr>& tensors,
1014 |     c10::ArrayRef<size_t> indices,
1015 |     c10::ArrayRef<BackendDataPtr> tensors_data) {
1016 |   std::vector<BackendDataPtr> result_tensors_data;
1017 |   std::unordered_map<int64_t, size_t> uid_index_map;
1018 |   size_t indices_index = 0;
1019 |   for (const auto i : c10::irange(tensors.size())) {
1020 |     int64_t tensor_id = tensors[i]->GetUniqueId();
1021 |     auto it = uid_index_map.find(tensor_id);
1022 |     if (it != uid_index_map.end()) {
1023 |       // Current tensor is a duplicate of a previously processed tensor that had
1024 |       // an IR Node to sync. Get the data from the tensor_data_map.
1025 |       result_tensors_data.push_back(result_tensors_data[it->second]);
1026 |     } else if (indices_index < indices.size() && i == indices[indices_index]) {
```
- EN: Implements routines such as `FetchTensors` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `FetchTensors` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 1027-1088
```cpp
1027 |       // If we are at the current index (it means that the tensor at index
1028 |       // 'i' had an IR node to sync), use the data held within the Async
1029 |       // object.
1030 |       uid_index_map.emplace(tensor_id, result_tensors_data.size());
1031 |       result_tensors_data.push_back(tensors_data[indices_index]);
1032 |       ++indices_index;
1033 |     } else if (!tensors[i]->CurrentTensorData()) {
1034 |       BackendDataPtr handle = tensors[i]->CurrentDataHandle();
1035 |       TORCH_CHECK(handle != nullptr);
1036 |       result_tensors_data.push_back(std::move(handle));
1037 |     }
1038 |   }
1039 |   return result_tensors_data;
1040 | }
1041 | 
1042 | void LazyGraphExecutor::TensorCollectionBarrier(SyncTensorCollection* coll) {
1043 |   if (coll) {
1044 |     static const std::string invalid_device(
1045 |         "Unknown0"); /* Temp solution to identify unassigned devices */
1046 |     if (coll->device.toString() == invalid_device || !coll->unlocker.empty()) {
1047 |       return;
1048 |     }
1049 |     VLOG(4) << "Waiting on device barrier for device " << coll->device
1050 |             << " ...";
1051 |     {
1052 |       TORCH_LAZY_TIMED("DeviceLockWait");
1053 |       coll->unlocker = DeviceLockerArena::Get()->LockDevices({coll->device});
1054 |     }
1055 |     VLOG(4) << "Waiting on device barrier for device " << coll->device
1056 |             << " done!";
1057 |   }
1058 | }
1059 | 
1060 | hash_t LazyGraphExecutor::GetGraphHash(
1061 |     const std::vector<LazyTensorPtr>& tensors) {
1062 |   SyncTensorsConfig config;
1063 |   config.sync_ltc_data = false;
1064 | 
1065 |   auto coll = CollectSyncTensors(tensors, config);
1066 |   std::vector<Value> ir_values;
1067 |   for (auto index : coll.indices) {
1068 |     Value ir_value = tensors[index]->CurrentIrValue();
1069 |     ir_values.push_back(ir_value);
1070 |   }
1071 |   auto po_data = RunPostOrder(ir_values, &coll);
1072 |   coll.hash = HashCombine(coll.hash, Hash(po_data.parameter_sequence));
1073 |   return coll.hash;
1074 | }
1075 | 
1076 | void LazyGraphExecutor::ClearComputationCache() {
1077 |   VLOG(4) << "Clearing the computation cache";
1078 |   GetComputationCache()->Clear();
1079 | }
1080 | 
1081 | void LazyGraphExecutor::RemoveFromComputationCache(const hash_t& hash) {
1082 |   VLOG(4) << "Removing computation cache for hash " << hash;
1083 |   if (!GetComputationCache()->Erase(hash)) {
1084 |     LOG(ERROR) << "There is no cached computation for hash " << hash << '\n';
1085 |   }
1086 | }
1087 | 
1088 | } // namespace torch::lazy
```
- EN: Implements routines such as `invalid_device` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 实现了 `invalid_device` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `TlsData`.
  - CN: `TlsData`。
- **Important routines / 重要例程**
  - EN: `Reset`, `TensorCompare`, `TensorsHaveIR`, `lock`, `GetLiveTensors`, `GetActiveDevices`, `MakeDeviceData`, `DataCacheArena`.
  - CN: `Reset`、`TensorCompare`、`TensorsHaveIR`、`lock`、`GetLiveTensors`、`GetActiveDevices`、`MakeDeviceData`、`DataCacheArena`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/core/lazy_graph_executor.h>`, `<ATen/ScalarOps.h>`, `<c10/util/Logging.h>`, `<c10/util/irange.h>`, `<torch/csrc/jit/jit_log.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/internal_ops/ltc_ops.h>`, `<torch/csrc/lazy/core/ir_dump_util.h>`, `<torch/csrc/lazy/core/ir_util.h>`, `<torch/csrc/lazy/core/tensor_util.h>`, `<torch/csrc/lazy/core/unique.h>`, `<torch/csrc/lazy/core/debug_util.h>`
- External includes / 外部头文件: `<utility>`, `<BaseTsd.h>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
