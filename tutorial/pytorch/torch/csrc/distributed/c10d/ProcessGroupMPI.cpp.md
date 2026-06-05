# ProcessGroupMPI.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/ProcessGroupMPI.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides process-group backend logic in the c10d distributed process-group subsystem. Representative routines include `cudaAwareMpiCheck`, `checkSingleTensorHelper`, `TORCH_CHECK`, `checkSingleTensor`, `checkSameSizeAndType`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供进程组后端逻辑。 代表性例程包括 `cudaAwareMpiCheck`、`checkSingleTensorHelper`、`TORCH_CHECK`、`checkSingleTensor`、`checkSameSizeAndType`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

```cpp
1: #include <torch/csrc/distributed/c10d/ProcessGroupMPI.hpp>
2: 
3: #ifdef USE_C10D_MPI
4: 
5: #include <iostream>
6: #include <map>
7: 
8: #include <c10/core/DeviceGuard.h>
9: #include <c10/util/irange.h>
10: #include <torch/csrc/distributed/c10d/ProcessGroup.hpp>
11: 
12: #if defined(OPEN_MPI) && OPEN_MPI
13: #include <mpi-ext.h> // Needed for CUDA-aware check
14: #endif
15: 
16: namespace c10d {
17: 
18: #define MPI_CHECK(cmd)                                                   \
19:   do {                                                                   \
20:     int mpiStatus = cmd;                                                 \
21:     if (mpiStatus != MPI_SUCCESS) {                                      \
22:       std::string err = "MPI error in: " + std::string(__FILE__) + ":" + \
23:           std::to_string(__LINE__) +                                     \
24:           ", with error code: " + std::to_string(mpiStatus);             \
```

- EN: Lines 1-24 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-24 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 25-48 / 第 25-48 行

```cpp
25:       TORCH_CHECK(false, err);                                           \
26:     }                                                                    \
27:   } while (0)
28: 
29: namespace {
30: 
31: // Op mapping
32: std::map<ReduceOp::RedOpType, MPI_Op> mpiOp = {
33:     {ReduceOp::MIN, MPI_MIN},
34:     {ReduceOp::MAX, MPI_MAX},
35:     {ReduceOp::SUM, MPI_SUM},
36:     {ReduceOp::PRODUCT, MPI_PROD},
37: };
38: // Type mapping
39: std::map<at::ScalarType, MPI_Datatype> mpiDatatype = {
40:     {at::kByte, MPI_UNSIGNED_CHAR},
41:     {at::kChar, MPI_CHAR},
42:     {at::kDouble, MPI_DOUBLE},
43:     {at::kFloat, MPI_FLOAT},
44: // FP16 is generally supported if MPIX_C_FLOAT16 exists
45: // for now the NVIDIA hpc sdk is built without (OpenMPI)
46: #if defined(MPIX_C_FLOAT16)
47:     {at::kHalf, MPIX_C_FLOAT16},
48: #endif
```

- EN: Lines 25-48 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 25-48 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 49-72 / 第 49-72 行

```cpp
49: #if defined(MPIX_C_BF16)
50:     {at::kBFloat16, MPIX_C_BF16},
51: #endif
52: #if defined(MPIX_BFLOAT16)
53:     {at::kBFloat16, MPIX_BFLOAT16},
54: #endif
55:     {at::kInt, MPI_INT},
56:     {at::kLong, MPI_LONG},
57:     {at::kShort, MPI_SHORT},
58: };
59: 
60: // Checking CUDA-aware MPI support, currently we only support CUDA aware
61: // MPI ops through Open MPI
62: bool cudaAwareMpiCheck() {
63: // Run time check
64: #if defined(MPIX_CUDA_AWARE_SUPPORT)
65:   if (MPIX_Query_cuda_support() == 1) {
66:     return true;
67:   } else {
68:     return false;
69:   }
70: // Recognize that Cray MPICH is CUDA-aware (used on Cray/HPE supercomputers)
71: #elif defined(MPIX_GPU_SUPPORT_CUDA)
72:   const char* cray_gpu_support = std::getenv("MPICH_GPU_SUPPORT_ENABLED");
```

- EN: Lines 49-72 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `cudaAwareMpiCheck`.
- CN: 第 49-72 行使用条件编译来适配特性开关、平台或可选后端；在 `cudaAwareMpiCheck` 等例程中引入具体执行逻辑。

### Lines 73-96 / 第 73-96 行

```cpp
73:   if (cray_gpu_support != nullptr && std::string(cray_gpu_support) == "1") {
74:     return true;
75:   } else {
76:     return false;
77:   }
78: #else // !defined(MPIX_CUDA_AWARE_SUPPORT) && !defined(MPIX_GPU_SUPPORT_CUDA)
79:   return false;
80: #endif // MPIX_CUDA_AWARE_SUPPORT
81: }
82: 
83: // Checking the input tensor's validity
84: void checkSingleTensorHelper(const at::Tensor& tensor) {
85:   if (!tensor.is_contiguous()) {
86:     TORCH_CHECK(false, "input tensor has to be contiguous");
87:   }
88:   if (tensor.is_sparse()) {
89:     TORCH_CHECK(false, "input tensor has to be dense");
90:   }
91:   if (tensor.is_cuda() && !cudaAwareMpiCheck()) {
92:     TORCH_CHECK(
93:         false,
94:         "CUDA tensor detected and the MPI used doesn't "
95:         "have CUDA-aware MPI support");
96:   }
```

- EN: Lines 73-96 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `checkSingleTensorHelper`, `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 73-96 行使用条件编译来适配特性开关、平台或可选后端；在 `checkSingleTensorHelper`、`TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 97-120 / 第 97-120 行

```cpp
97: }
98: 
99: void checkSingleTensor(const std::vector<at::Tensor>& tensors) {
100:   if (tensors.size() != 1) {
101:     TORCH_CHECK(
102:         false, "MPI process group does not support multi-GPU collectives");
103:   }
104:   checkSingleTensorHelper(tensors[0]);
105: }
106: 
107: void checkSameSizeAndType(
108:     const at::Tensor& t_in,
109:     const std::vector<at::Tensor>& tensors) {
110:   for (const auto& tensor : tensors) {
111:     if ((tensor.numel() != t_in.numel()) ||
112:         (tensor.scalar_type() != t_in.scalar_type())) {
113:       TORCH_CHECK(false, "Tensors are not equal in size or data type");
114:     }
115:     checkSingleTensorHelper(tensor);
116:   }
117: }
118: 
119: } // namespace
120: 
```

- EN: Lines 97-120 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `checkSingleTensor`, `TORCH_CHECK`, `checkSingleTensorHelper`; performs validation and error handling to keep distributed state consistent.
- CN: 第 97-120 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `checkSingleTensor`、`TORCH_CHECK`、`checkSingleTensorHelper` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 121-144 / 第 121-144 行

```cpp
121: std::vector<at::Tensor> ProcessGroupMPI::WorkMPI::result() {
122:   return outputTensors_;
123: }
124: 
125: c10::intrusive_ptr<c10::ivalue::Future> ProcessGroupMPI::WorkMPI::getFuture() {
126:   return future_;
127: }
128: 
129: void ProcessGroupMPI::WorkMPI::finishWorkMPIError(
130:     const std::exception_ptr& eptr) {
131:   future_->setError(eptr);
132:   finish(eptr);
133: }
134: 
135: void ProcessGroupMPI::WorkMPI::finishWorkMPI() {
136:   future_->markCompleted(at::IValue(outputTensors_));
137:   finish();
138: }
139: 
140: ProcessGroupMPI::AsyncWork::AsyncWork(
141:     MPI_Request request,
142:     std::vector<at::Tensor> outputTensors,
143:     const char* profilingTitle,
144:     const std::optional<std::vector<at::Tensor>>& inputTensors)
```

- EN: Lines 121-144 introduces executable logic in routines such as `finish`; returns computed state or forwards results to the surrounding caller.
- CN: 第 121-144 行在 `finish` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 145-168 / 第 145-168 行

```cpp
145:     : Work(-1, OpType::UNKNOWN, profilingTitle, inputTensors),
146:       outputTensors_(std::move(outputTensors)),
147:       request_(request) {
148:   memset(&status_, 0, sizeof(status_));
149: }
150: 
151: ProcessGroupMPI::AsyncWork::~AsyncWork() {
152:   if (request_ != MPI_REQUEST_NULL) {
153:     std::cerr
154:         << "Attempted destruction of AsyncWork before work has completed, "
155:         << "terminating the program." << '\n';
156:     std::terminate();
157:   }
158: }
159: 
160: bool ProcessGroupMPI::AsyncWork::isCompleted() {
161:   if (request_ == MPI_REQUEST_NULL) {
162:     return true;
163:   }
164: 
165:   std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
166:   int flag = 0;
167:   MPI_CHECK(MPI_Test(&request_, &flag, &status_));
168:   if (request_ != MPI_REQUEST_NULL) {
```

- EN: Lines 145-168 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 145-168 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 169-192 / 第 169-192 行

```cpp
169:     return false;
170:   }
171: 
172:   // request_ == MPI_REQUEST_NULL; the work has completed
173:   // Populate exception if request was not successful
174:   if (status_.MPI_ERROR != MPI_SUCCESS) {
175:     populateException();
176:   }
177: 
178:   return true;
179: }
180: 
181: bool ProcessGroupMPI::AsyncWork::isSuccess() const {
182:   if (request_ != MPI_REQUEST_NULL) {
183:     TORCH_CHECK(
184:         false,
185:         "Invalid call to AsyncWork::isSuccess before work has completed");
186:   }
187: 
188:   return status_.MPI_ERROR == MPI_SUCCESS;
189: }
190: 
191: int ProcessGroupMPI::AsyncWork::sourceRank() const {
192:   return status_.MPI_SOURCE;
```

- EN: Lines 169-192 introduces executable logic in routines such as `populateException`, `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 169-192 行在 `populateException`、`TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 193-216 / 第 193-216 行

```cpp
193: }
194: 
195: bool ProcessGroupMPI::AsyncWork::wait(std::chrono::milliseconds /* unused */) {
196:   if (request_ == MPI_REQUEST_NULL) {
197:     // AsyncWork needs to manually call profiling end callbacks if they are set,
198:     // since it does not call ProcessGroup::finish().
199:     if (Work::recordFunctionEndCallback_) {
200:       Work::recordFunctionEndCallback_();
201:       Work::recordFunctionEndCallback_ = nullptr;
202:     }
203:     return true;
204:   }
205: 
206:   std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
207:   MPI_CHECK(MPI_Wait(&request_, &status_));
208:   auto ok = (status_.MPI_ERROR == MPI_SUCCESS);
209: 
210:   // AsyncWork needs to manually call profiling end callbacks if they are set,
211:   // since it does not call ProcessGroup::finish().
212:   if (Work::recordFunctionEndCallback_) {
213:     Work::recordFunctionEndCallback_();
214:     Work::recordFunctionEndCallback_ = nullptr;
215:   }
216: 
```

- EN: Lines 193-216 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 193-216 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 217-240 / 第 217-240 行

```cpp
217:   if (!ok) {
218:     populateException();
219:     std::rethrow_exception(exception_);
220:   }
221:   if (c10d::allow_inflight_collective_as_graph_input()) {
222:     c10d::unregister_work(
223:         c10::intrusive_ptr<
224:             ProcessGroupMPI::AsyncWork>::unsafe_reclaim_from_nonowning(this));
225:   }
226:   // Always return true, because abort API is not implemented.
227:   return true;
228: }
229: 
230: void ProcessGroupMPI::AsyncWork::abort(){
231:     TORCH_CHECK(false, "ProcessGroupMPI::AsyncWork::abort not implemented.")}
232: 
233: std::vector<at::Tensor> ProcessGroupMPI::AsyncWork::result() {
234:   return outputTensors_;
235: }
236: 
237: void ProcessGroupMPI::AsyncWork::populateException() {
238:   std::array<char, MPI_MAX_ERROR_STRING> buf{};
239:   int len = buf.size();
240:   MPI_CHECK(MPI_Error_string(status_.MPI_ERROR, buf.data(), &len));
```

- EN: Lines 217-240 introduces executable logic in routines such as `populateException`; performs validation and error handling to keep distributed state consistent.
- CN: 第 217-240 行在 `populateException` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 241-264 / 第 241-264 行

```cpp
241:   exception_ =
242:       std::make_exception_ptr(std::runtime_error(std::string(buf.data(), len)));
243: }
244: 
245: // Static global states
246: int ProcessGroupMPI::mpiThreadSupport_ = 0;
247: std::mutex ProcessGroupMPI::pgGlobalMutex_;
248: 
249: void ProcessGroupMPI::mpiExit() {
250:   std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
251:   MPI_CHECK(MPI_Finalize());
252: }
253: 
254: void ProcessGroupMPI::initMPIOnce() {
255:   // Initialize MPI environment. We only want to initialize once.
256:   static bool init_mpi_flag [[maybe_unused]] = []() {
257:     int mpi_was_initialized = 0;
258:     MPI_CHECK(MPI_Initialized(&mpi_was_initialized));
259:     if (mpi_was_initialized == 0) {
260:       MPI_CHECK(MPI_Init_thread(
261:           nullptr, nullptr, MPI_THREAD_SERIALIZED, &mpiThreadSupport_));
262:       if (mpiThreadSupport_ < MPI_THREAD_SERIALIZED) {
263:         TORCH_CHECK(
264:             false,
```

- EN: Lines 241-264 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 241-264 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 265-288 / 第 265-288 行

```cpp
265:             "Used MPI implementation doesn't have the "
266:             "minimum level of threading support: "
267:             "MPI_THREAD_SERIALIZED. This is required by "
268:             "c10d package");
269:       }
270:       if (std::atexit(ProcessGroupMPI::mpiExit)) {
271:         TORCH_CHECK(false, "Fail to register the MPI exit handler");
272:       }
273:     } else {
274:       TORCH_WARN_ONCE("MPI was previously initialized.");
275:     }
276:     return true;
277:   }();
278: }
279: 
280: c10::intrusive_ptr<ProcessGroupMPI> ProcessGroupMPI::createProcessGroupMPI(
281:     std::vector<int> ranks) {
282:   // Once initialization
283:   initMPIOnce();
284: 
285:   MPI_Comm groupComm = MPI_COMM_WORLD;
286:   int rank = -1;
287:   int size = -1;
288: 
```

- EN: Lines 265-288 introduces executable logic in routines such as `TORCH_CHECK`, `TORCH_WARN_ONCE`, `initMPIOnce`; performs validation and error handling to keep distributed state consistent.
- CN: 第 265-288 行在 `TORCH_CHECK`、`TORCH_WARN_ONCE`、`initMPIOnce` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 289-312 / 第 289-312 行

```cpp
289:   {
290:     std::lock_guard<std::mutex> globalLock(pgGlobalMutex_);
291: 
292:     // If no ranks are specified, assume we're creating the root group
293:     if (!ranks.empty()) {
294:       MPI_Group worldGroup{};
295:       MPI_Group ranksGroup{};
296:       MPI_CHECK(MPI_Comm_group(MPI_COMM_WORLD, &worldGroup));
297:       MPI_CHECK(
298:           MPI_Group_incl(worldGroup, ranks.size(), ranks.data(), &ranksGroup));
299:       // `MPI_Comm_create` can be flaky in certain cases.
300:       // See: https://github.com/pytorch/pytorch/issues/53899
301:       constexpr int kMaxNumRetries = 3;
302:       bool groupComm_updated = false;
303:       MPI_Barrier(MPI_COMM_WORLD);
304:       for (const auto i : c10::irange(kMaxNumRetries)) {
305:         (void)i;
306:         if (MPI_Comm_create(MPI_COMM_WORLD, ranksGroup, &groupComm)) {
307:           groupComm_updated = true;
308:           break;
309:         }
310:       }
311:       MPI_CHECK(groupComm_updated);
312:       MPI_CHECK(MPI_Group_free(&worldGroup));
```

- EN: Lines 289-312 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 289-312 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 313-336 / 第 313-336 行

```cpp
313:       MPI_CHECK(MPI_Group_free(&ranksGroup));
314:     }
315: 
316:     // Fetch rank and world size for this group (MPI_COMM_WORLD or new)
317:     if (groupComm != MPI_COMM_NULL) {
318:       MPI_CHECK(MPI_Comm_rank(groupComm, &rank));
319:       MPI_CHECK(MPI_Comm_size(groupComm, &size));
320: 
321:       if (rank < 0 || size < 0) {
322:         TORCH_CHECK(false, "Failed to get the world_size / rank");
323:       }
324:     }
325:   }
326: 
327:   // If this process is not part of the group, we don't construct a
328:   // process group instance. This is in line with the semantics of the
329:   // other process group types.
330:   if (groupComm == MPI_COMM_NULL) {
331:     return c10::intrusive_ptr<ProcessGroupMPI>();
332:   }
333: 
334:   return c10::make_intrusive<ProcessGroupMPI>(rank, size, groupComm);
335: }
336: 
```

- EN: Lines 313-336 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 313-336 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 337-360 / 第 337-360 行

```cpp
337: ProcessGroupMPI::ProcessGroupMPI(int rank, int size, MPI_Comm pgComm)
338:     : Backend(rank, size), pgComm_(pgComm) {
339:   if (pgComm_ == MPI_COMM_NULL) {
340:     TORCH_CHECK(false, "pgComm_ must not be MPI_COMM_NULL");
341:   }
342: 
343:   // Start the worker thread accepting MPI calls
344:   workerThread_ = std::thread(&ProcessGroupMPI::runLoop, this);
345: 
346:   init();
347: }
348: 
349: ProcessGroupMPI::~ProcessGroupMPI() {
350:   destroy();
351: }
352: 
353: void ProcessGroupMPI::destroy() {
354:   std::unique_lock<std::mutex> lock(pgMutex_);
355:   queueConsumeCV_.wait(lock, [&] { return queue_.empty(); });
356: 
357:   // Queue is empty, signal stop
358:   stop_ = true;
359: 
360:   // Release lock to allow threads to terminate
```

- EN: Lines 337-360 introduces executable logic in routines such as `TORCH_CHECK`, `init`, `destroy`; performs validation and error handling to keep distributed state consistent.
- CN: 第 337-360 行在 `TORCH_CHECK`、`init`、`destroy` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 361-384 / 第 361-384 行

```cpp
361:   lock.unlock();
362:   queueProduceCV_.notify_all();
363: 
364:   // Join the single worker thread
365:   workerThread_.join();
366: }
367: 
368: void ProcessGroupMPI::abort() {
369:   destroy();
370:   MPI_Abort(pgComm_, EXIT_FAILURE);
371: }
372: 
373: void ProcessGroupMPI::runLoop() {
374:   std::unique_lock<std::mutex> lock(pgMutex_);
375: 
376:   while (!stop_) {
377:     if (queue_.empty()) {
378:       queueProduceCV_.wait(lock);
379:       continue;
380:     }
381: 
382:     auto workTuple = std::move(queue_.front());
383: 
384:     queue_.pop_front();
```

- EN: Lines 361-384 introduces executable logic in routines such as `destroy`.
- CN: 第 361-384 行在 `destroy` 等例程中引入具体执行逻辑。

### Lines 385-408 / 第 385-408 行

```cpp
385: 
386:     auto& workEntry = std::get<0>(workTuple);
387:     auto& work = std::get<1>(workTuple);
388: 
389:     lock.unlock();
390:     queueConsumeCV_.notify_one();
391: 
392:     try {
393:       workEntry->run(workEntry);
394:       work->finishWorkMPI();
395:     } catch (...) {
396:       work->finishWorkMPIError(std::current_exception());
397:     }
398: 
399:     lock.lock();
400:   }
401: }
402: 
403: c10::intrusive_ptr<Work> ProcessGroupMPI::enqueue(
404:     std::unique_ptr<WorkEntry> entry,
405:     const char* profilingTitle,
406:     const std::optional<std::vector<at::Tensor>>& inputTensors) {
407:   auto work =
408:       c10::make_intrusive<WorkMPI>(entry->dst, profilingTitle, inputTensors);
```

- EN: Lines 385-408 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 385-408 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 409-432 / 第 409-432 行

```cpp
409:   std::unique_lock<std::mutex> lock(pgMutex_);
410:   queue_.emplace_back(std::move(entry), work);
411:   lock.unlock();
412:   queueProduceCV_.notify_one();
413:   return work;
414: }
415: 
416: c10::intrusive_ptr<Work> ProcessGroupMPI::broadcast(
417:     std::vector<at::Tensor>& tensors,
418:     const BroadcastOptions& opts) {
419:   checkSingleTensor(tensors);
420:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
421:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
422:         auto data = (entry->src)[0];
423:         c10::DeviceGuard guard(data.device());
424:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
425:         MPI_CHECK(MPI_Bcast(
426:             data.data_ptr(),
427:             data.numel(),
428:             mpiDatatype.at(data.scalar_type()),
429:             opts.rootRank,
430:             pgComm_));
431:       };
432:   auto entry =
```

- EN: Lines 409-432 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 409-432 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 433-456 / 第 433-456 行

```cpp
433:       std::make_unique<WorkEntry>(&tensors, &tensors, std::move(runFunc));
434:   return enqueue(
435:       std::move(entry),
436:       "mpi:broadcast",
437:       std::optional<std::vector<at::Tensor>>(tensors));
438: }
439: 
440: c10::intrusive_ptr<Work> ProcessGroupMPI::allreduce(
441:     std::vector<at::Tensor>& tensors,
442:     const AllreduceOptions& opts) {
443:   checkSingleTensor(tensors);
444: 
445:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
446:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
447:         auto data = (entry->src)[0];
448:         c10::DeviceGuard guard(data.device());
449:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
450:         MPI_CHECK(MPI_Allreduce(
451:             MPI_IN_PLACE,
452:             data.data_ptr(),
453:             data.numel(),
454:             mpiDatatype.at(data.scalar_type()),
455:             mpiOp.at(opts.reduceOp),
456:             pgComm_));
```

- EN: Lines 433-456 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 433-456 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 457-480 / 第 457-480 行

```cpp
457:       };
458:   auto entry =
459:       std::make_unique<WorkEntry>(&tensors, &tensors, std::move(runFunc));
460:   return enqueue(
461:       std::move(entry),
462:       "mpi:all_reduce",
463:       std::optional<std::vector<at::Tensor>>(tensors));
464: }
465: 
466: c10::intrusive_ptr<Work> ProcessGroupMPI::allreduce_coalesced(
467:     std::vector<at::Tensor>& tensors,
468:     const AllreduceCoalescedOptions& opts) {
469:   TORCH_CHECK(false, "allreduce_coalesced is currently not supported with MPI");
470: }
471: 
472: c10::intrusive_ptr<Work> ProcessGroupMPI::reduce(
473:     std::vector<at::Tensor>& tensors,
474:     const ReduceOptions& opts) {
475:   checkSingleTensor(tensors);
476: 
477:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
478:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
479:         auto data = (entry->src)[0];
480:         auto dataPtr = (entry->src)[0].data_ptr();
```

- EN: Lines 457-480 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 457-480 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 481-504 / 第 481-504 行

```cpp
481:         void* sendbuf = (rank_ == opts.rootRank) ? MPI_IN_PLACE : dataPtr;
482:         void* recvbuf = (rank_ == opts.rootRank) ? dataPtr : nullptr;
483: 
484:         c10::DeviceGuard guard(data.device());
485:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
486:         MPI_CHECK(MPI_Reduce(
487:             sendbuf,
488:             recvbuf,
489:             data.numel(),
490:             mpiDatatype.at(data.scalar_type()),
491:             mpiOp.at(opts.reduceOp),
492:             opts.rootRank,
493:             pgComm_));
494:       };
495:   auto entry =
496:       std::make_unique<WorkEntry>(&tensors, &tensors, std::move(runFunc));
497:   return enqueue(
498:       std::move(entry),
499:       "mpi:reduce",
500:       std::optional<std::vector<at::Tensor>>(tensors));
501: }
502: 
503: c10::intrusive_ptr<Work> ProcessGroupMPI::allgather(
504:     std::vector<std::vector<at::Tensor>>& outputTensors,
```

- EN: Lines 481-504 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 481-504 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 505-528 / 第 505-528 行

```cpp
505:     std::vector<at::Tensor>& inputTensors,
506:     const AllgatherOptions& opts) {
507:   checkSingleTensor(inputTensors);
508:   if (outputTensors.size() != 1) {
509:     TORCH_CHECK(
510:         false,
511:         "MPI process group only supports a single "
512:         "tensor op");
513:   }
514:   if (static_cast<size_t>(size_) != outputTensors[0].size()) {
515:     TORCH_CHECK(
516:         false,
517:         "All gather: number of output tensors should equal "
518:         "to the world size");
519:   }
520: 
521:   checkSameSizeAndType(inputTensors[0], outputTensors[0]);
522: 
523:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
524:       [this](std::unique_ptr<WorkEntry>& entry) {
525:         auto data = (entry->src)[0];
526:         std::vector<at::Tensor> outputDataVec = entry->dst;
527:         auto flatOutputTensor = newLikeFlat(outputDataVec);
528: 
```

- EN: Lines 505-528 introduces executable logic in routines such as `TORCH_CHECK`, `checkSameSizeAndType`; performs validation and error handling to keep distributed state consistent.
- CN: 第 505-528 行在 `TORCH_CHECK`、`checkSameSizeAndType` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 529-552 / 第 529-552 行

```cpp
529:         c10::DeviceGuard guard(data.device());
530:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
531:         MPI_CHECK(MPI_Allgather(
532:             data.data_ptr(),
533:             data.numel(),
534:             mpiDatatype.at(data.scalar_type()),
535:             flatOutputTensor.data_ptr(),
536:             data.numel(),
537:             mpiDatatype.at(data.scalar_type()),
538:             pgComm_));
539: 
540:         for (const auto i : c10::irange(outputDataVec.size())) {
541:           outputDataVec[i].copy_(flatOutputTensor[static_cast<int64_t>(i)]);
542:         }
543:       };
544:   auto entry = std::make_unique<WorkEntry>(
545:       &inputTensors, &outputTensors[0], std::move(runFunc));
546:   return enqueue(
547:       std::move(entry),
548:       "mpi:all_gather",
549:       std::optional<std::vector<at::Tensor>>(inputTensors));
550: }
551: 
552: c10::intrusive_ptr<Work> ProcessGroupMPI::allgather_coalesced(
```

- EN: Lines 529-552 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 529-552 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 553-576 / 第 553-576 行

```cpp
553:     std::vector<std::vector<at::Tensor>>& /* unused */,
554:     std::vector<at::Tensor>& /* unused */,
555:     const AllgatherOptions& /* unused */) {
556:   TORCH_CHECK(false, "ProcessGroupMPI does not support allgather_coalesced");
557: }
558: 
559: c10::intrusive_ptr<Work> ProcessGroupMPI::gather(
560:     std::vector<std::vector<at::Tensor>>& outputTensors,
561:     std::vector<at::Tensor>& inputTensors,
562:     const GatherOptions& opts) {
563:   checkSingleTensor(inputTensors);
564: 
565:   if (rank_ != opts.rootRank) {
566:     if (!outputTensors.empty()) {
567:       TORCH_CHECK(
568:           false,
569:           "Gather: number of output tensors should be 0 "
570:           "for non-root");
571:     }
572:   } else {
573:     if (outputTensors.size() != 1) {
574:       TORCH_CHECK(false, "Gather: multi-GPU collective is not supported");
575:     }
576:     if (static_cast<size_t>(size_) != outputTensors[0].size()) {
```

- EN: Lines 553-576 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 553-576 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 577-600 / 第 577-600 行

```cpp
577:       TORCH_CHECK(
578:           false,
579:           "Gather: number of output tensors should equal "
580:           "to the world size");
581:     }
582:     checkSameSizeAndType(inputTensors[0], outputTensors[0]);
583:   }
584: 
585:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
586:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
587:         auto data = (entry->src)[0];
588:         void* recvbuf = nullptr;
589:         at::Tensor flatOutputTensor;
590: 
591:         std::vector<at::Tensor> dstdata = entry->dst;
592:         if (rank_ == opts.rootRank) {
593:           flatOutputTensor = newLikeFlat(dstdata);
594:           recvbuf = flatOutputTensor.data_ptr();
595:         }
596: 
597:         c10::DeviceGuard guard(data.device());
598:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
599:         MPI_CHECK(MPI_Gather(
600:             data.data_ptr(),
```

- EN: Lines 577-600 introduces executable logic in routines such as `TORCH_CHECK`, `checkSameSizeAndType`; performs validation and error handling to keep distributed state consistent.
- CN: 第 577-600 行在 `TORCH_CHECK`、`checkSameSizeAndType` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 601-624 / 第 601-624 行

```cpp
601:             data.numel(),
602:             mpiDatatype.at(data.scalar_type()),
603:             recvbuf,
604:             data.numel(),
605:             mpiDatatype.at(data.scalar_type()),
606:             opts.rootRank,
607:             pgComm_));
608: 
609:         if (rank_ == opts.rootRank) {
610:           const std::vector<at::Tensor>& outputDataVec = entry->dst;
611:           // copy the flattened output tensors to the outputs
612:           for (const auto i : c10::irange(outputDataVec.size())) {
613:             outputDataVec.at(i).copy_(
614:                 flatOutputTensor[static_cast<int64_t>(i)]);
615:           }
616:         }
617:       };
618: 
619:   if (rank_ == opts.rootRank) {
620:     auto entry = std::make_unique<WorkEntry>(
621:         &inputTensors, &outputTensors[0], std::move(runFunc));
622:     return enqueue(
623:         std::move(entry),
624:         "mpi:gather",
```

- EN: Lines 601-624 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 601-624 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 625-648 / 第 625-648 行

```cpp
625:         std::optional<std::vector<at::Tensor>>(inputTensors));
626:   } else {
627:     auto entry =
628:         std::make_unique<WorkEntry>(&inputTensors, nullptr, std::move(runFunc));
629:     return enqueue(
630:         std::move(entry),
631:         "mpi:gather",
632:         std::optional<std::vector<at::Tensor>>(inputTensors));
633:   }
634: }
635: 
636: c10::intrusive_ptr<Work> ProcessGroupMPI::scatter(
637:     std::vector<at::Tensor>& outputTensors,
638:     std::vector<std::vector<at::Tensor>>& inputTensors,
639:     const ScatterOptions& opts) {
640:   checkSingleTensor(outputTensors);
641: 
642:   if (rank_ != opts.rootRank) {
643:     if (!inputTensors.empty()) {
644:       TORCH_CHECK(
645:           false,
646:           "Scatter: number of input tensors should be 0 "
647:           "for non-root");
648:     }
```

- EN: Lines 625-648 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 625-648 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 649-672 / 第 649-672 行

```cpp
649:   } else {
650:     if (inputTensors.size() != 1) {
651:       TORCH_CHECK(false, "Scatter: multi-GPU collective is not supported");
652:     }
653:     if (static_cast<size_t>(size_) != inputTensors[0].size()) {
654:       TORCH_CHECK(
655:           false,
656:           "Scatter: number of input tensors should equal "
657:           "to the world size");
658:     }
659:     checkSameSizeAndType(outputTensors[0], inputTensors[0]);
660:   }
661: 
662:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
663:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
664:         auto data = (entry->dst)[0];
665:         void* sendbuf = nullptr;
666:         at::Tensor flatInputTensor;
667: 
668:         if (rank_ == opts.rootRank) {
669:           std::vector<at::Tensor>& inputDataVec = entry->src;
670:           flatInputTensor = newLikeFlat(inputDataVec);
671:           sendbuf = flatInputTensor.data_ptr();
672: 
```

- EN: Lines 649-672 introduces executable logic in routines such as `TORCH_CHECK`, `checkSameSizeAndType`; performs validation and error handling to keep distributed state consistent.
- CN: 第 649-672 行在 `TORCH_CHECK`、`checkSameSizeAndType` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 673-696 / 第 673-696 行

```cpp
673:           // copy the input tensors to the flatten large send buffer
674:           for (const auto i : c10::irange(inputDataVec.size())) {
675:             flatInputTensor[static_cast<int64_t>(i)].copy_(inputDataVec.at(i));
676:           }
677:         }
678: 
679:         c10::DeviceGuard guard(data.device());
680:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
681:         MPI_CHECK(MPI_Scatter(
682:             sendbuf,
683:             data.numel(),
684:             mpiDatatype.at(data.scalar_type()),
685:             data.data_ptr(),
686:             data.numel(),
687:             mpiDatatype.at(data.scalar_type()),
688:             opts.rootRank,
689:             pgComm_));
690:       };
691: 
692:   if (rank_ == opts.rootRank) {
693:     auto entry = std::make_unique<WorkEntry>(
694:         &inputTensors[0], &outputTensors, std::move(runFunc));
695:     return enqueue(
696:         std::move(entry),
```

- EN: Lines 673-696 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 673-696 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 697-720 / 第 697-720 行

```cpp
697:         "mpi:scatter",
698:         !inputTensors.empty()
699:             ? std::optional<std::vector<at::Tensor>>(inputTensors[0])
700:             : std::nullopt);
701:   } else {
702:     auto entry = std::make_unique<WorkEntry>(
703:         nullptr, &outputTensors, std::move(runFunc));
704:     return enqueue(
705:         std::move(entry),
706:         "mpi:scatter",
707:         !inputTensors.empty()
708:             ? std::optional<std::vector<at::Tensor>>(inputTensors[0])
709:             : std::nullopt);
710:   }
711: }
712: 
713: c10::intrusive_ptr<Work> ProcessGroupMPI::reduce_scatter(
714:     std::vector<at::Tensor>& outputTensors,
715:     std::vector<std::vector<at::Tensor>>& inputTensors,
716:     const ReduceScatterOptions& opts) {
717:   checkSingleTensor(outputTensors);
718:   if (inputTensors.size() != 1) {
719:     TORCH_CHECK(
720:         false,
```

- EN: Lines 697-720 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 697-720 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 721-744 / 第 721-744 行

```cpp
721:         "MPI process group only supports a single "
722:         "tensor op");
723:   }
724:   if (static_cast<size_t>(size_) != inputTensors[0].size()) {
725:     TORCH_CHECK(
726:         false,
727:         "Reduce scatter: number of input tensors should equal "
728:         "to the world size");
729:   }
730:   checkSameSizeAndType(outputTensors[0], inputTensors[0]);
731: 
732:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
733:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
734:         auto data = (entry->dst)[0];
735:         auto flatInputTensor = newLikeFlat(entry->src);
736:         for (const auto i : c10::irange(entry->src.size())) {
737:           flatInputTensor[static_cast<int64_t>(i)].copy_(entry->src[i]);
738:         }
739:         int recvcount = flatInputTensor.numel() / size_;
740: 
741:         c10::DeviceGuard guard(data.device());
742:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
743:         MPI_CHECK(MPI_Reduce_scatter_block(
744:             flatInputTensor.data_ptr(),
```

- EN: Lines 721-744 introduces executable logic in routines such as `TORCH_CHECK`, `checkSameSizeAndType`; performs validation and error handling to keep distributed state consistent.
- CN: 第 721-744 行在 `TORCH_CHECK`、`checkSameSizeAndType` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 745-768 / 第 745-768 行

```cpp
745:             data.data_ptr(),
746:             recvcount,
747:             mpiDatatype.at(data.scalar_type()),
748:             mpiOp.at(opts.reduceOp),
749:             pgComm_));
750:       };
751: 
752:   auto entry = std::make_unique<WorkEntry>(
753:       &inputTensors[0], &outputTensors, std::move(runFunc));
754:   return enqueue(
755:       std::move(entry),
756:       "mpi:reduce_scatter",
757:       std::optional<std::vector<at::Tensor>>(inputTensors[0]));
758: }
759: 
760: c10::intrusive_ptr<Work> ProcessGroupMPI::alltoall_base(
761:     at::Tensor& outputTensor,
762:     at::Tensor& inputTensor,
763:     std::vector<int64_t>& outputSplitSizes,
764:     std::vector<int64_t>& inputSplitSizes,
765:     const AllToAllOptions& opts) {
766:   checkSingleTensorHelper(inputTensor);
767:   checkSingleTensorHelper(outputTensor);
768: 
```

- EN: Lines 745-768 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 745-768 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 769-792 / 第 769-792 行

```cpp
769:   if (outputSplitSizes.empty() && inputSplitSizes.empty()) {
770:     // We can use alltoall
771:     TORCH_CHECK(
772:         outputTensor.numel() == inputTensor.numel() &&
773:             outputTensor.type() == inputTensor.type(),
774:         "Tensors are not equal in size or data type");
775:     TORCH_CHECK(
776:         outputTensor.size(0) % size_ == 0,
777:         "Tensor's dim 0 does not divide equally across group size");
778: 
779:     std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
780:         [this](std::unique_ptr<WorkEntry>& entry) {
781:           auto srcdata = (entry->src)[0];
782:           auto dstdata = (entry->dst)[0];
783:           c10::DeviceGuard guard(srcdata.device());
784:           std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
785:           MPI_CHECK(MPI_Alltoall(
786:               srcdata.data_ptr(),
787:               srcdata.numel() / size_,
788:               mpiDatatype.at(srcdata.scalar_type()),
789:               dstdata.data_ptr(),
790:               dstdata.numel() / size_,
791:               mpiDatatype.at(dstdata.scalar_type()),
792:               pgComm_));
```

- EN: Lines 769-792 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 769-792 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 793-816 / 第 793-816 行

```cpp
793:         };
794:     std::vector<at::Tensor> inputTensors = {inputTensor};
795:     std::vector<at::Tensor> outputTensors = {outputTensor};
796:     auto entry = std::make_unique<WorkEntry>(
797:         &inputTensors, &outputTensors, std::move(runFunc));
798:     return enqueue(
799:         std::move(entry),
800:         "mpi:all_to_all",
801:         std::optional<std::vector<at::Tensor>>(inputTensors));
802:   } else {
803:     // Need alltoallv
804:     c10d::checkSplitSizes(inputSplitSizes, inputTensor, size_);
805:     c10d::checkSplitSizes(outputSplitSizes, outputTensor, size_);
806:     std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
807:         [this, inputSplitSizes, outputSplitSizes](
808:             std::unique_ptr<WorkEntry>& entry) {
809:           auto srcdata = (entry->src)[0];
810:           auto dstdata = (entry->dst)[0];
811:           std::vector<int> send_lengths(size_);
812:           std::vector<int> recv_lengths(size_);
813:           std::vector<int> send_offsets(size_);
814:           std::vector<int> recv_offsets(size_);
815:           c10d::computeLengthsAndOffsets(
816:               inputSplitSizes, srcdata, &send_lengths, &send_offsets);
```

- EN: Lines 793-816 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 793-816 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 817-840 / 第 817-840 行

```cpp
817:           c10d::computeLengthsAndOffsets(
818:               outputSplitSizes, dstdata, &recv_lengths, &recv_offsets);
819:           c10::DeviceGuard guard(srcdata.device());
820:           std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
821:           MPI_CHECK(MPI_Alltoallv(
822:               srcdata.data_ptr(),
823:               send_lengths.data(),
824:               send_offsets.data(),
825:               mpiDatatype.at(srcdata.scalar_type()),
826:               dstdata.data_ptr(),
827:               recv_lengths.data(),
828:               recv_offsets.data(),
829:               mpiDatatype.at(dstdata.scalar_type()),
830:               pgComm_));
831:         };
832:     std::vector<at::Tensor> inputTensors = {inputTensor};
833:     std::vector<at::Tensor> outputTensors = {outputTensor};
834:     auto entry = std::make_unique<WorkEntry>(
835:         &inputTensors, &outputTensors, std::move(runFunc));
836:     return enqueue(
837:         std::move(entry),
838:         "mpi:all_to_all",
839:         std::optional<std::vector<at::Tensor>>(inputTensors));
840:   }
```

- EN: Lines 817-840 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 817-840 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 841-864 / 第 841-864 行

```cpp
841: }
842: 
843: c10::intrusive_ptr<Work> ProcessGroupMPI::alltoall(
844:     std::vector<at::Tensor>& outputTensors,
845:     std::vector<at::Tensor>& inputTensors,
846:     const AllToAllOptions& opts) {
847:   TORCH_CHECK(
848:       inputTensors.size() == static_cast<size_t>(size_),
849:       "Number of input tensors are not equal to group size");
850:   TORCH_CHECK(
851:       outputTensors.size() == static_cast<size_t>(size_),
852:       "Number of output tensors are not equal to group size");
853:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
854:       [this](std::unique_ptr<WorkEntry>& entry) {
855:         std::vector<int> send_lengths(size_);
856:         std::vector<int> recv_lengths(size_);
857:         std::vector<int> send_offsets(size_);
858:         std::vector<int> recv_offsets(size_);
859:         auto srcdata = entry->src;
860:         auto dstdata = entry->dst;
861:         auto src_len = c10d::computeLengthsAndOffsets(
862:             srcdata, &send_lengths, &send_offsets);
863:         auto dst_len = c10d::computeLengthsAndOffsets(
864:             dstdata, &recv_lengths, &recv_offsets);
```

- EN: Lines 841-864 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 841-864 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 865-888 / 第 865-888 行

```cpp
865:         std::vector<int64_t> send_lengthsL(
866:             send_lengths.begin(), send_lengths.end());
867:         std::vector<int64_t> recv_lengthsL(
868:             recv_lengths.begin(), recv_lengths.end());
869:         at::Tensor srcFlatData =
870:             at::empty({static_cast<int64_t>(src_len)}, srcdata[0].options());
871:         at::Tensor dstFlatData =
872:             at::empty({static_cast<int64_t>(dst_len)}, dstdata[0].options());
873:         auto srcFlatDataSplits =
874:             srcFlatData.split_with_sizes(c10::IntArrayRef(send_lengthsL), 0);
875:         for (const auto i : c10::irange(size_)) {
876:           srcFlatDataSplits[i].copy_(srcdata[i].view({-1}));
877:         }
878:         c10::DeviceGuard guard1(srcdata[0].device());
879:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
880:         MPI_CHECK(MPI_Alltoallv(
881:             srcFlatData.data_ptr(),
882:             send_lengths.data(),
883:             send_offsets.data(),
884:             mpiDatatype.at(srcdata[0].scalar_type()),
885:             dstFlatData.data_ptr(),
886:             recv_lengths.data(),
887:             recv_offsets.data(),
888:             mpiDatatype.at(dstdata[0].scalar_type()),
```

- EN: Lines 865-888 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 865-888 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 889-912 / 第 889-912 行

```cpp
889:             pgComm_));
890: 
891:         auto dstFlatDataSplits =
892:             dstFlatData.split_with_sizes(c10::IntArrayRef(recv_lengthsL), 0);
893:         for (const auto i : c10::irange(size_)) {
894:           dstdata[i].view({-1}).copy_(dstFlatDataSplits[i]);
895:         }
896:       };
897:   auto entry = std::make_unique<WorkEntry>(
898:       &inputTensors, &outputTensors, std::move(runFunc));
899:   return enqueue(
900:       std::move(entry),
901:       "mpi:all_to_all",
902:       std::optional<std::vector<at::Tensor>>(inputTensors));
903: }
904: 
905: c10::intrusive_ptr<Work> ProcessGroupMPI::send(
906:     std::vector<at::Tensor>& tensors,
907:     int dstRank,
908:     int tag) {
909:   checkSingleTensor(tensors);
910: 
911:   auto& tensor = tensors[0];
912:   MPI_Request request = MPI_REQUEST_NULL;
```

- EN: Lines 889-912 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 889-912 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 913-936 / 第 913-936 行

```cpp
913: 
914:   {
915:     c10::DeviceGuard guard(tensor.device());
916:     std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
917:     MPI_CHECK(MPI_Isend(
918:         tensor.data_ptr(),
919:         tensor.numel(),
920:         mpiDatatype.at(tensor.scalar_type()),
921:         dstRank,
922:         tag,
923:         pgComm_,
924:         &request));
925:   }
926: 
927:   return c10::make_intrusive<AsyncWork>(
928:       request,
929:       std::vector<at::Tensor>(),
930:       "mpi:send",
931:       std::optional<std::vector<at::Tensor>>(tensors));
932: }
933: 
934: c10::intrusive_ptr<Work> ProcessGroupMPI::recv(
935:     std::vector<at::Tensor>& tensors,
936:     int srcRank,
```

- EN: Lines 913-936 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 913-936 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 937-960 / 第 937-960 行

```cpp
937:     int tag) {
938:   checkSingleTensor(tensors);
939: 
940:   auto& tensor = tensors[0];
941:   MPI_Request request = MPI_REQUEST_NULL;
942: 
943:   {
944:     c10::DeviceGuard guard(tensor.device());
945:     std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
946:     MPI_CHECK(MPI_Irecv(
947:         tensor.data_ptr(),
948:         tensor.numel(),
949:         mpiDatatype.at(tensor.scalar_type()),
950:         srcRank,
951:         tag,
952:         pgComm_,
953:         &request));
954:   }
955: 
956:   return c10::make_intrusive<AsyncWork>(
957:       request,
958:       tensors,
959:       "mpi:recv",
960:       std::optional<std::vector<at::Tensor>>(tensors));
```

- EN: Lines 937-960 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 937-960 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 961-984 / 第 961-984 行

```cpp
961: }
962: 
963: c10::intrusive_ptr<Work> ProcessGroupMPI::recvAnysource(
964:     std::vector<at::Tensor>& tensors,
965:     int tag) {
966:   checkSingleTensor(tensors);
967: 
968:   auto& tensor = tensors[0];
969:   MPI_Request request = MPI_REQUEST_NULL;
970: 
971:   {
972:     c10::DeviceGuard guard(tensor.device());
973:     std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
974:     MPI_CHECK(MPI_Irecv(
975:         tensor.data_ptr(),
976:         tensor.numel(),
977:         mpiDatatype.at(tensor.scalar_type()),
978:         MPI_ANY_SOURCE,
979:         tag,
980:         pgComm_,
981:         &request));
982:   }
983: 
984:   return c10::make_intrusive<AsyncWork>(
```

- EN: Lines 961-984 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 961-984 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 985-1008 / 第 985-1008 行

```cpp
985:       request,
986:       tensors,
987:       "mpi:recvAnySource",
988:       std::optional<std::vector<at::Tensor>>(tensors));
989: }
990: 
991: c10::intrusive_ptr<Work> ProcessGroupMPI::barrier(const BarrierOptions& opts) {
992:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
993:       [this](std::unique_ptr<WorkEntry>& entry) {
994:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
995:         MPI_CHECK(MPI_Barrier(pgComm_));
996:       };
997:   auto entry =
998:       std::make_unique<WorkEntry>(nullptr, nullptr, std::move(runFunc));
999:   return enqueue(std::move(entry), "mpi:barrier", std::nullopt);
1000: }
1001: 
1002: c10::intrusive_ptr<Work> ProcessGroupMPI::_allgather_base(
1003:     at::Tensor& outputTensor,
1004:     at::Tensor& inputTensor,
1005:     const AllgatherOptions& opts) {
1006:   TORCH_CHECK(
1007:       outputTensor.numel() == inputTensor.numel() * size_,
1008:       "All gather: output tensor size must be equal to input tensor size times the world size");
```

- EN: Lines 985-1008 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 985-1008 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1009-1032 / 第 1009-1032 行

```cpp
1009: 
1010:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
1011:       [this](std::unique_ptr<WorkEntry>& entry) {
1012:         auto dstdata = (entry->dst)[0];
1013:         auto srcdata = (entry->src)[0];
1014:         c10::DeviceGuard guard(srcdata.device());
1015:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
1016:         MPI_CHECK(MPI_Allgather(
1017:             srcdata.data_ptr(),
1018:             srcdata.numel(),
1019:             mpiDatatype.at(srcdata.scalar_type()),
1020:             dstdata.data_ptr(),
1021:             srcdata.numel(),
1022:             mpiDatatype.at(dstdata.scalar_type()),
1023:             pgComm_));
1024:       };
1025: 
1026:   auto inputTensors = std::vector<at::Tensor>({inputTensor});
1027:   auto outputTensors = std::vector<at::Tensor>({outputTensor});
1028:   auto entry = std::make_unique<WorkEntry>(
1029:       &inputTensors, &outputTensors, std::move(runFunc));
1030:   return enqueue(
1031:       std::move(entry),
1032:       "mpi:_allgather_base",
```

- EN: Lines 1009-1032 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1009-1032 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1033-1056 / 第 1033-1056 行

```cpp
1033:       std::optional<std::vector<at::Tensor>>(inputTensors));
1034: }
1035: 
1036: c10::intrusive_ptr<Work> ProcessGroupMPI::_reduce_scatter_base(
1037:     at::Tensor& outputTensor,
1038:     at::Tensor& inputTensor,
1039:     const ReduceScatterOptions& opts) {
1040:   TORCH_CHECK(
1041:       outputTensor.numel() * size_ == inputTensor.numel(),
1042:       "Reduce scatter: input tensor size must be equal to output tensor size times the world size");
1043: 
1044:   std::function<void(std::unique_ptr<WorkEntry>&)> runFunc =
1045:       [opts, this](std::unique_ptr<WorkEntry>& entry) {
1046:         auto dstdata = (entry->dst)[0];
1047:         auto srcdata = (entry->src)[0];
1048:         c10::DeviceGuard guard(srcdata.device());
1049:         std::unique_lock<std::mutex> globalLock(pgGlobalMutex_);
1050:         MPI_CHECK(MPI_Reduce_scatter_block(
1051:             srcdata.data_ptr(),
1052:             dstdata.data_ptr(),
1053:             dstdata.numel(),
1054:             mpiDatatype.at(srcdata.scalar_type()),
1055:             mpiOp.at(opts.reduceOp),
1056:             pgComm_));
```

- EN: Lines 1033-1056 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1033-1056 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1057-1071 / 第 1057-1071 行

```cpp
1057:       };
1058: 
1059:   auto inputTensors = std::vector<at::Tensor>({inputTensor});
1060:   auto outputTensors = std::vector<at::Tensor>({outputTensor});
1061:   auto entry = std::make_unique<WorkEntry>(
1062:       &inputTensors, &outputTensors, std::move(runFunc));
1063:   return enqueue(
1064:       std::move(entry),
1065:       "mpi:_reduce_scatter_base",
1066:       std::optional<std::vector<at::Tensor>>(inputTensors));
1067: }
1068: 
1069: } // namespace c10d
1070: 
1071: #endif // USE_C10D_MPI
```

- EN: Lines 1057-1071 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1057-1071 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `cudaAwareMpiCheck`, `checkSingleTensorHelper`, `TORCH_CHECK`, `checkSingleTensor`, `checkSameSizeAndType`, `finish`
- CN: 核心符号：`cudaAwareMpiCheck`、`checkSingleTensorHelper`、`TORCH_CHECK`、`checkSingleTensor`、`checkSameSizeAndType`、`finish`
- EN: Notable themes: process-group orchestration, collective communication logic.
- CN: 值得关注的主题：进程组编排、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/ProcessGroupMPI.hpp`, `torch/csrc/distributed/c10d/ProcessGroup.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/core/DeviceGuard.h`, `c10/util/irange.h`
- External or system headers / 外部或系统头文件: `iostream`, `map`, `mpi-ext.h`
- Local symbols / 本地符号: `cudaAwareMpiCheck`, `checkSingleTensorHelper`, `TORCH_CHECK`, `checkSingleTensor`, `checkSameSizeAndType`, `finish`, `populateException`, `TORCH_WARN_ONCE`