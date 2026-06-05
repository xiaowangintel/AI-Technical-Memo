# ProcessGroupNCCL.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/ProcessGroupNCCL.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides process-group backend logic in the c10d distributed process-group subsystem. Key types include `DumpPipe`, `TensorShelf`, `TORCH_API`, `ErrorHandlingMode`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供进程组后端逻辑。 关键类型包括 `DumpPipe`、`TensorShelf`、`TORCH_API`、`ErrorHandlingMode`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

```cpp
1: #pragma once
2: 
3: #ifdef USE_C10D_NCCL
4: 
5: #if defined(__linux__)
6: #include <fcntl.h>
7: #include <sys/stat.h>
8: #include <sys/types.h>
9: #include <unistd.h>
10: #endif
11: 
12: #include <atomic>
13: #include <chrono>
14: #include <deque>
15: #include <future>
16: #include <iostream>
17: #include <list>
18: #include <mutex>
19: #include <thread>
20: #include <unordered_map>
21: 
22: #include <torch/csrc/distributed/c10d/Backend.hpp>
23: #include <torch/csrc/distributed/c10d/NCCLUtils.hpp>
24: #include <torch/csrc/distributed/c10d/PrefixStore.hpp>
```

- EN: Lines 1-24 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 1-24 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 25-48 / 第 25-48 行

```cpp
25: #include <torch/csrc/distributed/c10d/Store.hpp>
26: #include <torch/csrc/distributed/c10d/cuda/CUDAEventCache.hpp>
27: #include <torch/csrc/distributed/c10d/logger.hpp>
28: #include <torch/csrc/distributed/c10d/symm_mem/intra_node_comm.hpp>
29: 
30: #include <ATen/DynamicLibrary.h>
31: #include <ATen/cuda/CUDAContext.h>
32: #include <ATen/cuda/CUDAEvent.h>
33: #include <ATen/cuda/MemPool.h>
34: #include <c10/core/Stream.h>
35: #include <c10/core/StreamGuard.h>
36: #include <c10/cuda/CUDACachingAllocator.h>
37: #include <c10/cuda/CUDAGuard.h>
38: #include <c10/cuda/CUDAStream.h>
39: 
40: #include <torch/custom_class.h>
41: 
42: namespace c10d {
43: 
44: // Control broadcasting of NCCL uniqueId
45: static std::vector<std::string> TORCH_NCCL_BCAST_UNIQUEID = {
46:     "TORCH_NCCL_BCAST_UNIQUEID"};
47: 
48: // Control EagerInit P2P serialization warning
```

- EN: Lines 25-48 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 25-48 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 49-72 / 第 49-72 行

```cpp
49: static std::vector<std::string>
50:     TORCH_NCCL_SHOW_EAGER_INIT_P2P_SERIALIZATION_WARNING = {
51:         "TORCH_NCCL_SHOW_EAGER_INIT_P2P_SERIALIZATION_WARNING"};
52: 
53: // Control whether to always use high priority streams
54: static std::vector<std::string> TORCH_NCCL_HIGH_PRIORITY = {
55:     "TORCH_NCCL_HIGH_PRIORITY"};
56: 
57: // Control whether or not wait() is blocking or non-blocking.
58: static std::vector<std::string> TORCH_NCCL_BLOCKING_WAIT = {
59:     "TORCH_NCCL_BLOCKING_WAIT",
60:     "NCCL_BLOCKING_WAIT"};
61: 
62: // TODO: We want to eventually remove this variable and make users to use
63: // the default value (3 - SkipCleanUp).
64: // Control whether or not we perform Async Error Handling with NCCL.
65: static std::vector<std::string> TORCH_NCCL_ASYNC_ERROR_HANDLING = {
66:     "TORCH_NCCL_ASYNC_ERROR_HANDLING",
67:     "NCCL_ASYNC_ERROR_HANDLING"};
68: 
69: // Control whether dumping debug info on watchdog
70: // timeout is enabled. This variable must be set together with
71: // TORCH_NCCL_ENABLE_MONITORING=1 and TORCH_NCCL_TRACE_BUFFER_SIZE > 0.
72: static std::vector<std::string> TORCH_NCCL_DUMP_ON_TIMEOUT = {
```

- EN: Lines 49-72 continues the local implementation details and data flow for this file.
- CN: 第 49-72 行继续展开本文件的局部实现细节与数据流。

### Lines 73-96 / 第 73-96 行

```cpp
73:     "TORCH_NCCL_DUMP_ON_TIMEOUT"};
74: 
75: // Control whether to propagate NCCL errors to all ranks through TCPStore.
76: static std::vector<std::string> TORCH_NCCL_PROPAGATE_ERROR = {
77:     "TORCH_NCCL_PROPAGATE_ERROR"};
78: 
79: // Control whether Desync Debug is enabled. This variable must be set
80: // together with TORCH_NCCL_ASYNC_ERROR_HANDLING.
81: static std::vector<std::string> TORCH_NCCL_DESYNC_DEBUG = {
82:     "TORCH_NCCL_DESYNC_DEBUG",
83:     "NCCL_DESYNC_DEBUG"};
84: 
85: // Enable recording start-events for all ProcessGroupNCCL collectives, and
86: // compute accurate collective timing per-collective. (Note: end-events are
87: // recorded by default. Turn on this flag can increase chances of a watchdog
88: // hang due to performing a CUDA event query which eventually calls
89: // cudaEventElapsedTime() API.
90: static std::vector<std::string> TORCH_NCCL_ENABLE_TIMING = {
91:     "TORCH_NCCL_ENABLE_TIMING",
92:     "NCCL_ENABLE_TIMING"};
93: 
94: // Enable monitoring thread which aborts the process when the ProcessGroupNCCL
95: // Watchdog thread gets stuck and no heartbeat is detected after
96: // TORCH_NCCL_HEARTBEAT_TIMEOUT_SEC. This can happen due to calling CUDA/NCCL
```

- EN: Lines 73-96 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 73-96 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 97-120 / 第 97-120 行

```cpp
97: // APIs that may hang. It is Useful to prevent jobs being stuck for a prolonged
98: // time than necessary tying up cluster resources.
99: static std::vector<std::string> TORCH_NCCL_ENABLE_MONITORING = {
100:     "TORCH_NCCL_ENABLE_MONITORING"};
101: 
102: // Control the watchdog heartbeat timeout period after which the monitoring
103: // thread will abort the process.
104: static std::vector<std::string> TORCH_NCCL_HEARTBEAT_TIMEOUT_SEC = {
105:     "TORCH_NCCL_HEARTBEAT_TIMEOUT_SEC"};
106: 
107: // Whether to rethrow CUDA Errors in the watchdog (default true)
108: static std::vector<std::string> TORCH_NCCL_RETHROW_CUDA_ERRORS = {
109:     "TORCH_NCCL_RETHROW_CUDA_ERRORS"};
110: 
111: // The maximum number of events we store in the flight recorder's ring buffer.
112: // (One event could be the start or end of a collective, for example).
113: static std::vector<std::string> TORCH_NCCL_TRACE_BUFFER_SIZE = {
114:     "TORCH_NCCL_TRACE_BUFFER_SIZE"};
115: 
116: // Control how much extra time we will wait for dumping the debugging info
117: // before we exit and throws timeout exception.
118: static std::vector<std::string> TORCH_NCCL_WAIT_TIMEOUT_DUMP_MILSEC = {
119:     "TORCH_NCCL_WAIT_TIMEOUT_DUMP_MILSEC"};
120: 
```

- EN: Lines 97-120 performs validation and error handling to keep distributed state consistent.
- CN: 第 97-120 行执行校验与错误处理，以保持分布式状态一致。

### Lines 121-144 / 第 121-144 行

```cpp
121: // Control the interval inside the monitoring thread to check the coordinated
122: // signal from other ranks, e.g. to dump the debugging information.
123: static std::vector<std::string> TORCH_NCCL_COORD_CHECK_MILSEC = {
124:     "TORCH_NCCL_COORD_CHECK_MILSEC"};
125: 
126: // Whether to log C++ stack traces on unclean shutdown (default true)
127: static std::vector<std::string> TORCH_NCCL_LOG_CPP_STACK_ON_UNCLEAN_SHUTDOWN = {
128:     "TORCH_NCCL_LOG_CPP_STACK_ON_UNCLEAN_SHUTDOWN"};
129: 
130: // Whether to include only active collectives in the Flight Recorder trace
131: // (default false)
132: static std::vector<std::string> TORCH_NCCL_EXTRA_DUMP_ON_EXEC = {
133:     "TORCH_NCCL_EXTRA_DUMP_ON_EXEC"};
134: 
135: // Control whether to use CudaEventCache for the collective in watchdog thread.
136: // We noticed in the past when cuda global lock is held, destroying CudaEvent
137: // can cause a hang.
138: static std::vector<std::string> TORCH_NCCL_CUDA_EVENT_CACHE = {
139:     "TORCH_NCCL_CUDA_EVENT_CACHE"};
140: 
141: // Control the number of ranks each root can cover during NCCL comm init.
142: static std::vector<std::string> TORCH_NCCL_RANKS_PER_ROOT = {
143:     "TORCH_NCCL_RANKS_PER_ROOT"};
144: 
```

- EN: Lines 121-144 continues the local implementation details and data flow for this file.
- CN: 第 121-144 行继续展开本文件的局部实现细节与数据流。

### Lines 145-168 / 第 145-168 行

```cpp
145: static std::vector<std::string> TORCH_NCCL_NAN_CHECK = {"TORCH_NCCL_NAN_CHECK"};
146: 
147: constexpr const char* NCCL_BACKEND_NAME = "nccl";
148: 
149: constexpr const char* kStoreDumpKey = "exception_dump";
150: 
151: constexpr const char* kStoreErrorSignalKey = "remote_error";
152: 
153: constexpr const int kWorkStatusUpdatePeriodMs = 30 * 1000; // 30 seconds
154: 
155: constexpr auto kProcessGroupNCCLDefaultTimeout =
156:     std::chrono::milliseconds(10 * 60 * 1000);
157: 
158: // NoHandling: do not handle asynchronous NCCL errors
159: // TearDown: tear down process upon error, see `WorkNCCL::handleException`
160: // CleanUpOnly: just clean up collectives and abort communicators without
161: // tearing down process SkipCleanUp: (this is a temporary option and can be
162: // removed in future) tear down process without cleaning up NCCL communicators.
163: // This should be used as a last resort in case `ncclCommAbort` itself is
164: // hanging
165: enum ErrorHandlingMode {
166:   NoHandling = 0,
167:   TearDown = 1,
168:   CleanUpOnly = 2,
```

- EN: Lines 145-168 declares or defines types such as `ErrorHandlingMode`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 145-168 行声明或定义了 `ErrorHandlingMode` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 169-192 / 第 169-192 行

```cpp
169:   SkipCleanUp = 3
170: };
171: 
172: #define SHOULD_CLEAN_UP(a) (a != NoHandling && a != SkipCleanUp)
173: 
174: #define SHOULD_TEAR_DOWN(a) (a != NoHandling && a != CleanUpOnly)
175: 
176: #define PRINT_COLLECTIVE_HASH_SIGNATURE(phase, opType, numel, hashValue)      \
177:   LOG(WARNING) << logPrefix() << "Hash of " << phase << " to NCCL " << opType \
178:                << " with size " << numel << " is " << hashValue;
179: 
180: // If set, ProcessGroupNCCL doesn't use recordStream calls to ensure
181: // caching allocator safety for tensors used on both user-facing and
182: // internal comm streams.
183: // Instead, it stashes live references to those tensors until after
184: // user-facing streams are synced with comm streams.
185: // See stashed_for_allocator_safety_ below.
186: static std::vector<std::string> TORCH_NCCL_AVOID_RECORD_STREAMS = {
187:     "TORCH_NCCL_AVOID_RECORD_STREAMS"};
188: 
189: // If set, ProcessGroupNCCL registers postAlloc and preFree hooks to cuda cache
190: // allocator so that whenever a tensor is allocated or freed, ProcessGroupNCCL
191: // can register/deregister the tensor on all available NCCL communicators.
192: static std::vector<std::string> TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK =
```

- EN: Lines 169-192 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 169-192 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 193-216 / 第 193-216 行

```cpp
193:     {"TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK",
194:      "NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK"};
195: 
196: #if defined(__linux__)
197: struct DumpPipe {
198:   DumpPipe(int rank, const std::string& fileStem, int traceBufferSize) {
199:     if (fileStem.empty() || traceBufferSize <= 0) {
200:       return;
201:     }
202:     std::string filename = c10::str(fileStem, rank, ".pipe");
203:     TORCH_CHECK(
204:         unlink(filename.c_str()) != -1 || errno == ENOENT,
205:         "Error removing existing named pipe ",
206:         filename,
207:         ", Error: ",
208:         std::strerror(errno));
209:     TORCH_CHECK(
210:         mkfifo(filename.c_str(), 0666) != -1,
211:         "Error creating named pipe ",
212:         filename,
213:         ", Error: ",
214:         std::strerror(errno));
215:     fd_ = open(filename.c_str(), O_RDONLY | O_NONBLOCK);
216:     LOG(INFO) << "Pipe file " << filename
```

- EN: Lines 193-216 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `DumpPipe`; introduces executable logic in routines such as `DumpPipe`.
- CN: 第 193-216 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `DumpPipe` 等类型；在 `DumpPipe` 等例程中引入具体执行逻辑。

### Lines 217-240 / 第 217-240 行

```cpp
217:               << " has been opened, write to it to trigger NCCL Debug Dump.";
218:     TORCH_CHECK(fd_ != -1, "Error opening named pipe ", filename);
219:   }
220:   bool shouldDump() {
221:     if (fd_ == -1) {
222:       return false;
223:     }
224:     // NOLINTNEXTLINE(*array*)
225:     char buf[128]{};
226:     // non-blocking from O_NONBLOCK above.
227:     // Ignore EINTR because we already will poll this
228:     // again later.
229:     ssize_t bytesRead = read(fd_, &buf, 128);
230:     return bytesRead > 0;
231:   }
232:   ~DumpPipe() {
233:     if (fd_ != -1) {
234:       close(fd_);
235:     }
236:   }
237: 
238:  private:
239:   int fd_ = -1;
240: };
```

- EN: Lines 217-240 introduces executable logic in routines such as `TORCH_CHECK`, `shouldDump`, `~DumpPipe`; performs validation and error handling to keep distributed state consistent.
- CN: 第 217-240 行在 `TORCH_CHECK`、`shouldDump`、`~DumpPipe` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 241-264 / 第 241-264 行

```cpp
241: #else
242: struct DumpPipe {
243:   DumpPipe(int rank) {}
244:   bool shouldDump() {
245:     return false;
246:   }
247: };
248: #endif
249: 
250: // A shelf for stashing tensors between op call and `work.wait()`.
251: // Used in case of async ops.
252: class TensorShelf {
253:  public:
254:   // Stash tensors so that CachingAllocator cannot recycle them prematurely.
255:   void stash(std::vector<at::Tensor>& tensors);
256:   // Stash tensors from another shelf.
257:   void stash(TensorShelf& other);
258:   // Unstage the stashed tensors so that CachingAllocator can recycle them.
259:   // Same as `clear()`.
260:   void unstash();
261:   // Whether shelf is empty.
262:   bool empty();
263:   // Clear the shelf.
264:   void clear();
```

- EN: Lines 241-264 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `DumpPipe`, `TensorShelf`; introduces executable logic in routines such as `DumpPipe`, `shouldDump`, `stash`.
- CN: 第 241-264 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `DumpPipe`、`TensorShelf` 等类型；在 `DumpPipe`、`shouldDump`、`stash` 等例程中引入具体执行逻辑。

### Lines 265-288 / 第 265-288 行

```cpp
265: 
266:  protected:
267:   // Get the inner tensor vector. Use with caution as it is not protected by
268:   // mutex.
269:   std::vector<at::Tensor>& get();
270: 
271:  private:
272:   std::vector<at::Tensor> tVector_;
273:   // Need a mutex to protect `tVector_` because it can be potentially accessed
274:   // from both main thread and watchdog thread.
275:   std::mutex mutex_;
276: };
277: 
278: // ProcessGroupNCCL implements NCCL bindings for c10d.
279: //
280: // All functions of the class are expected to be called in the same order
281: // across all processes in the process group.  This is the only way that we
282: // can guarantee to match up the same calls among all processes.
283: //
284: // All NCCL functions provided by this class are asynchronous functions. More
285: // specifically, each NCCL call is scheduled on a separate CUDA stream that is
286: // different from the current CUDA stream. This is for the purpose of
287: // achieving potentially concurrency and better performance. As a result,
288: // it is the callers' responsibility to make sure that the CUDA stream their
```

- EN: Lines 265-288 introduces executable logic in routines such as `get`.
- CN: 第 265-288 行在 `get` 等例程中引入具体执行逻辑。

### Lines 289-312 / 第 289-312 行

```cpp
289: // code works on needs to wait for the NCCL operation from
290: // this class.
291: //
292: // This can be done by calling:
293: //
294: // either WorkNCCL::wait() or WorkNCCL::synchronize(), both achieves the same
295: // functionality and are synonyms.
296: //
297: // Also note that WorkNCCL::finishedGPUExecution() is a helper function only
298: // provided by ProcessGroupNCCL to check if the NCCL operation of WorkNCCL has
299: // finished execution on the GPU (not just scheduled).
300: //
301: // Example on using the NCCL process group
302: //
303: //   ProcessGroupNCCL pg(store, rank, size);
304: //   std::shared_ptr<WorkNCCL> work = pg.allreduce(tensors);
305: //
306: //   // At this point, NCCL kernel has already by queued successfully
307: //   // Now, let current stream wait for the NCCL to finish, this function is
308: //   // async operation as well
309: //
310: //   work->wait()
311: //
312: //   // Now continue on other work in the current stream.
```

- EN: Lines 289-312 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 289-312 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 313-336 / 第 313-336 行

```cpp
313: class TORCH_API ProcessGroupNCCL : public Backend {
314:  public:
315:   class WorkNCCL : public Work, public std::enable_shared_from_this<WorkNCCL> {
316:    public:
317:     friend struct WorkInfo;
318: 
319:     // Constructor takes a list of CUDA devices
320:     WorkNCCL(
321:         std::string pgUID,
322:         std::string pgDesc,
323:         at::Device& device,
324:         int rank,
325:         OpType opType,
326:         uint64_t seq,
327:         bool isP2P = false,
328:         const char* profilingTitle = nullptr,
329:         const std::optional<std::vector<at::Tensor>>& inputs = std::nullopt,
330:         bool enableTiming = false,
331:         bool cudaEventCacheEnabled = false,
332:         DebugLevel distDebugLevel = DebugLevel::Off);
333:     // Copy constructor doing partial copy without outputs_. Cleanup thread
334:     // monitors and removes finished works. However it will deadlock when
335:     // destructs outputs_ tensors who are view tensors in autograd graph.
336:     WorkNCCL(const WorkNCCL& w);
```

- EN: Lines 313-336 declares or defines types such as `TORCH_API`, `WorkNCCL`; introduces executable logic in routines such as `WorkNCCL`.
- CN: 第 313-336 行声明或定义了 `TORCH_API`、`WorkNCCL` 等类型；在 `WorkNCCL` 等例程中引入具体执行逻辑。

### Lines 337-360 / 第 337-360 行

```cpp
337: 
338:     ~WorkNCCL() override = default;
339: 
340:     // Checks if the NCCL kernel has started to execute.
341:     bool isStarted();
342: 
343:     // Checks if request has completed. In this specific case of NCCL, it checks
344:     // if the NCCL operation has completed on the GPU in its own NCCL stream.
345:     // Non-blocking operation.
346:     bool isCompleted() override;
347: 
348:     bool isSuccess() const override;
349: 
350:     // Same as calling synchronize() for NCCL work if timeout is not set.
351:     // Otherwise, it will block the CPU thread until the NCCL work is completed
352:     // or timed out. If timeout, exception will be thrown.
353:     bool wait(std::chrono::milliseconds timeout = kNoTimeout) override;
354: 
355:     void blockCurrentStream() override {
356:       synchronize();
357:     }
358: 
359:     void abort() override;
360: 
```

- EN: Lines 337-360 introduces executable logic in routines such as `isStarted`, `isCompleted`, `isSuccess`.
- CN: 第 337-360 行在 `isStarted`、`isCompleted`、`isSuccess` 等例程中引入具体执行逻辑。

### Lines 361-384 / 第 361-384 行

```cpp
361:     // Let current stream wait on the completion of the NCCL work
362:     // Throws on exceptions.
363:     void synchronize() override;
364: 
365:     // Synchronize streams by blocking each on the NCCL stream
366:     void synchronizeStream();
367: 
368:     // Helper function to handle exception (throw if needed).
369:     void handleException(ErrorHandlingMode asyncErrorHandling);
370: 
371:     // Helper function that checks if the NCCL kernels have finished
372:     // execution on the GPUs
373:     bool finishedGPUExecution();
374: 
375:     // Get a Future object that will be marked as completed internally.
376:     c10::intrusive_ptr<c10::ivalue::Future> getFuture() override;
377: 
378:     // Get a Future result of each work (e.g. success, different error types).
379:     // instead of the tensor output.
380:     c10::intrusive_ptr<c10::ivalue::Future> getFutureResult() override;
381: 
382:     float getDuration() const override;
383: 
384:     uint64_t getSequencenumber() const override;
```

- EN: Lines 361-384 introduces executable logic in routines such as `synchronize`, `synchronizeStream`, `handleException`; performs validation and error handling to keep distributed state consistent.
- CN: 第 361-384 行在 `synchronize`、`synchronizeStream`、`handleException` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 385-408 / 第 385-408 行

```cpp
385: 
386:     const std::string& logPrefix() const;
387: 
388:     // Helper function that sets an exception_ptr on the WorkNCCL object.
389:     void setException(std::exception_ptr exception_ptr);
390: 
391:     // Helper function that returns True if the WorkNCCL object has timed out
392:     // and False otherwise.
393:     // In case of timeout, set exception on the WorkNCCL object.
394:     bool checkTimeout(
395:         std::optional<std::chrono::milliseconds> timeout = std::nullopt);
396: 
397:     // Print the traceback of the collective at call time
398:     void printTraceback() const;
399: 
400:     std::string getTraceback() const;
401: 
402:     std::vector<at::Tensor> result() override;
403: 
404:    protected:
405:     // The process group unique id
406:     std::string pgUID_;
407: 
408:     // The process group description
```

- EN: Lines 385-408 introduces executable logic in routines such as `logPrefix`, `setException`, `checkTimeout`.
- CN: 第 385-408 行在 `logPrefix`、`setException`、`checkTimeout` 等例程中引入具体执行逻辑。

### Lines 409-432 / 第 409-432 行

```cpp
409:     std::string pgDesc_;
410: 
411:     // The cached list of CUDA devices to operate on
412:     at::Device device_;
413: 
414:     // The start CUDA event of NCCL operator tracking this work item. These
415:     // start CUDA events are needed by desync debugging if enabled.
416:     std::shared_ptr<at::cuda::CUDAEvent> ncclStartEvent_;
417: 
418:     // The end CUDA event of NCCL operator tracking this work item.
419:     std::shared_ptr<at::cuda::CUDAEvent> ncclEndEvent_;
420: 
421:     // The NCCL communicator used for this work item.
422:     std::shared_ptr<NCCLComm> ncclComm_;
423: 
424:     // whether this work is a barrier op
425:     bool isBarrierOp_{false};
426: 
427:     // Clone of blockingWait_ from ProcessGroupNCCL.
428:     bool blockingWait_{false};
429: 
430:     // Clone of opTimeout_ from ProcessGroupNCCL.
431:     std::chrono::milliseconds opTimeout_{};
432: 
```

- EN: Lines 409-432 continues the local implementation details and data flow for this file.
- CN: 第 409-432 行继续展开本文件的局部实现细节与数据流。

### Lines 433-456 / 第 433-456 行

```cpp
433:     // Ephemeral timeouts are owned by exactly one work,
434:     // and reset after that work completes.
435:     // There may be more than one ephemeral timeout active at the same time,
436:     // and this variable is used to track the ownership of ephemeral timeout.
437:     std::chrono::milliseconds ownedEphermeralTimeout_ =
438:         std::chrono::milliseconds(0);
439: 
440:     // Time point representing when the work started.
441:     std::chrono::time_point<std::chrono::steady_clock> workStartTime_;
442: 
443:     // Record the sequential number of collective or p2p.
444:     uint64_t seq_;
445:     bool isP2P_;
446: 
447:     // Indicates if the nccl start event has been updated to the store trace.
448:     // This will be used by desync debug.
449:     bool startTraceUpdated_{false};
450: 
451:     // Record collective sizes for debug. We only record the size on the first
452:     // device as multi-device per process is deprecated
453:     size_t numelIn_ = 0;
454:     size_t numelOut_ = 0;
455: 
456:     // Wrapper method for the static checkForNCCLErrors which can be overridden
```

- EN: Lines 433-456 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 433-456 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 457-480 / 第 457-480 行

```cpp
457:     // for tests.
458:     virtual std::exception_ptr checkForNCCLErrors();
459: 
460:     friend std::ostream& operator<<(
461:         std::ostream& output,
462:         const WorkNCCL& workNCCL);
463: 
464:     // Checks for NCCL errors and sets an appropriate exception_ptr.
465:     void checkAndSetException();
466: 
467:     // Just checks whether GPU execution has started, without modifying
468:     // exception_ptr.
469:     bool startedGPUExecutionInternal() const;
470: 
471:     // Just checks whether GPU execution has completed, without modifying
472:     // exception_ptr.
473:     bool finishedGPUExecutionInternal() const;
474: 
475:     // Reference to the store so that we can write aborted communicators
476:     // to the store.
477:     c10::intrusive_ptr<Store> store_;
478: 
479:     // Store a reference to NCCL collective's outputs, used by result and to
480:     // give a more descriptive message when representing the Work as a string.
```

- EN: Lines 457-480 introduces executable logic in routines such as `checkForNCCLErrors`, `operator<<`, `checkAndSetException`.
- CN: 第 457-480 行在 `checkForNCCLErrors`、`operator<<`、`checkAndSetException` 等例程中引入具体执行逻辑。

### Lines 481-504 / 第 481-504 行

```cpp
481:     std::shared_ptr<std::vector<at::Tensor>> outputs_;
482: 
483:     // TORCH_NCCL_AVOID_RECORD_STREAMS implementation helper.
484:     // Stores references to participating non-output tensors (ie inputs,
485:     // flattened intermediates).
486:     // We'll clear this list in synchronizeStream, just after user-facing
487:     // stream(s) are synced with the nccl work stream(s).
488:     // By keeping these refs (as well as outputs_) alive until after the
489:     // collective's work rejoins the user-facing streams, we achieve
490:     // caching allocator safety without any recordStream calls.
491:     // For in-place collectives, some refs stashed here may alias outputs_,
492:     // but that doesn't do any harm.
493:     std::shared_ptr<TensorShelf> stashed_for_allocator_safety_;
494: 
495:     // The future returned by getFuture.
496:     c10::intrusive_ptr<at::ivalue::Future> future_;
497: 
498:     // the future result (e.g., success or failure) of the work
499:     c10::intrusive_ptr<at::ivalue::Future> futureWorkResult_;
500: 
501:     bool timingEnabled_;
502:     // unique id used to tell the trace buffer that this
503:     // work has completed
504:     std::optional<uint64_t> trace_id_;
```

- EN: Lines 481-504 continues the local implementation details and data flow for this file.
- CN: 第 481-504 行继续展开本文件的局部实现细节与数据流。

### Lines 505-528 / 第 505-528 行

```cpp
505:     std::optional<uint64_t> trace_reset_epoch_;
506:     DebugLevel distDebugLevel_;
507:     friend class ProcessGroupNCCL;
508:   };
509: 
510:   struct Options : Backend::Options {
511:     // NOTE: timeout in ProcessGroupNCCL::Options denote the timeout for
512:     // operations. This is only used when blockingWait_ is enabled.
513:     explicit Options(bool is_high_priority_stream = false);
514:     Options(const Options&) = default;
515:     Options(Options&&) noexcept = default;
516:     Options& operator=(const Options&) = delete;
517:     Options& operator=(Options&&) noexcept = delete;
518:     ~Options() override = default;
519: 
520:     // return intrusive_ptr of the object
521:     static c10::intrusive_ptr<Options> create(
522:         bool is_high_priority_stream = false) {
523:       return c10::make_intrusive<Options>(is_high_priority_stream);
524:     }
525: 
526:     // Schedule NCCL operations on high priority CUDA streams
527:     bool is_high_priority_stream;
528: 
```

- EN: Lines 505-528 declares or defines types such as `Options`; introduces executable logic in routines such as `Options`, `create`.
- CN: 第 505-528 行声明或定义了 `Options` 等类型；在 `Options`、`create` 等例程中引入具体执行逻辑。

### Lines 529-552 / 第 529-552 行

```cpp
529: #ifdef NCCL_HAS_CONFIG
530:     // Configure ranks
531:     ncclConfig_t config = NCCL_CONFIG_INITIALIZER;
532: #endif
533: 
534:     // Optional "parent" backend and color to create communicators from
535:     // via `ncclCommSplit`
536:     c10::intrusive_ptr<ProcessGroupNCCL> split_from;
537:     // Color to use for `ncclCommSplit`, values:
538:     // * Non-negative value: in group;
539:     // * NCCL_SPLIT_NOCOLOR (-1): not in group;
540:     // * NCCL_SPLIT_NOCOLOR - 1: uninitialized.
541:     // [Note 1]: the type must be `int` instead of `int64_t` because NCCL API
542:     // accepts int. Otherwise, an implicit conversion may happen at the API call
543:     // and the value may become negative.
544:     // [Note 2]: this member is pybinded to Python, the value passed from Python
545:     // must be within the numerical range of C++ int. Otherwise, Python will
546:     // raise a RuntimeError saying type is incompatible. See also
547:     // `_process_group_color` in `distributed_c10d.py`.
548: #ifdef NCCL_HAS_COMM_SPLIT
549:     int split_color{NCCL_SPLIT_NOCOLOR - 1};
550: #else
551:     // [Note 3]: for older NCCL versions, NCCL_SPLIT_NOCOLOR is not defined. But
552:     // `split_color` is pybinded to Python, so we need to define it. So we use
```

- EN: Lines 529-552 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 529-552 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 553-576 / 第 553-576 行

```cpp
553:     // the int value of `NCCL_SPLIT_NOCOLOR` (-1) instead.
554:     int split_color{-2};
555: #endif
556:   };
557: 
558:   // Helper class related to TORCH_NCCL_DESYNC_DEBUG
559:   class DesyncDebugger {
560:    public:
561:     // Initialize and enable DesyncDebugger
562:     void init(
563:         int rank,
564:         int size,
565:         int globalRank,
566:         int pgId,
567:         c10::intrusive_ptr<Store> store);
568: 
569:     // Run desync debug. This function is called by watchdog at time of timeout.
570:     void run();
571: 
572:     // Log work start to store.
573:     void logWorkStart(WorkNCCL& work);
574: 
575:     // Log work end to store.
576:     void logWorkEnd(WorkNCCL& work);
```

- EN: Lines 553-576 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `DesyncDebugger`; introduces executable logic in routines such as `init`, `run`, `logWorkStart`.
- CN: 第 553-576 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `DesyncDebugger` 等类型；在 `init`、`run`、`logWorkStart` 等例程中引入具体执行逻辑。

### Lines 577-600 / 第 577-600 行

```cpp
577: 
578:    private:
579:     // Whether desync debug is enabled.
580:     // If false, all functions are no-op.
581:     bool enabled_{false};
582: 
583:     // From ProcessGroupNCCL
584:     int rank_;
585:     int size_;
586:     int globalRank_;
587:     int pgId_;
588: 
589:     // Reference to the store so that we can log start/end event.
590:     c10::intrusive_ptr<Store> store_;
591: 
592:     // The store keys to trace the last NCCL collective kernel CUDA events -
593:     // start event and end event respectively. These are used to do desync root
594:     // cause analysis.
595:     std::string traceKeyStart_;
596:     std::string traceKeyEnd_;
597:   };
598: 
599:   // Class that runs as a separate thread aside from watchdog
600:   // thread because we need to check the heartbeat from watchdog thread
```

- EN: Lines 577-600 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 577-600 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 601-624 / 第 601-624 行

```cpp
601:   // so that when we get stuck in some NCCL/CUDA calls,
602:   // we can dump the debugging information and abort the process.
603:   class HeartbeatMonitor {
604:    public:
605:     HeartbeatMonitor(ProcessGroupNCCL* pg);
606:     virtual ~HeartbeatMonitor() = default;
607: 
608:     // Start the heartbeat monitor thread.
609:     void start();
610: 
611:     // Join the heartbeat monitor thread.
612:     void join();
613: 
614:     // Run the actual loop to check watchdog heartbeat.
615:     virtual void runLoop();
616: 
617:     // Set the terminal flag and notify the heartbeat monitor thread to stop.
618:     void stop();
619: 
620:     // Set the last update time of watchdog thread.
621:     void setLastWorkListUpdateTime(
622:         std::chrono::time_point<std::chrono::steady_clock> time);
623: 
624:     int getDumpTimeout() const;
```

- EN: Lines 601-624 declares or defines types such as `HeartbeatMonitor`; introduces executable logic in routines such as `HeartbeatMonitor`, `start`, `join`.
- CN: 第 601-624 行声明或定义了 `HeartbeatMonitor` 等类型；在 `HeartbeatMonitor`、`start`、`join` 等例程中引入具体执行逻辑。

### Lines 625-648 / 第 625-648 行

```cpp
625: 
626:     // Util function to get the timeout error message
627:     std::string getNCCLWatchdogTimeoutErrorMsg(const std::string& extraMsg);
628: 
629:     // Util function to get the timeout exit message
630:     std::string getNCCLWatchdogTimeoutExitMsg(const std::string& exitReason);
631: 
632:    protected:
633:     // We need to keep a reference to the PG instance so that we can access
634:     // the member functions of the PG instance. We store a raw pointer on
635:     // purpose because the heartbeat monitor thread now still lives within the
636:     // lifetime of the PG instance.
637:     ProcessGroupNCCL* pg_;
638: 
639:    private:
640:     // Whether or not to print C++ stack traces to logs on unclean shutdown.
641:     bool logCppStackOnUncleanShutdown_;
642: 
643:     // The time interval used for deciding whether there is no watchdog
644:     // heartbeat.
645:     int heartbeatTimeoutInSec_;
646: 
647:     // timeout for the dump to finish.
648:     int waitTimeoutDumpInMilSec_;
```

- EN: Lines 625-648 introduces executable logic in routines such as `getNCCLWatchdogTimeoutErrorMsg`, `getNCCLWatchdogTimeoutExitMsg`.
- CN: 第 625-648 行在 `getNCCLWatchdogTimeoutErrorMsg`、`getNCCLWatchdogTimeoutExitMsg` 等例程中引入具体执行逻辑。

### Lines 649-672 / 第 649-672 行

```cpp
649: 
650:     // Interval of check coordinated signals in ProcessGroupNCCL from other
651:     // ranks e.g., trigger the dump of the debugging info for timeout when
652:     // notified.
653:     int coordCheckIntervalMilSec_;
654: 
655:     // We gate the heartbeat monitor thread so that we can roll it out
656:     // gradually.
657:     bool watchdogHeartbeatMonitorEnabled_;
658: 
659:     // Monitor thread which checks the heartbeat of Watchdog thread.
660:     // If the monitor thread finds there is no heartbeat, it will dump debug
661:     // info and then kill the watchdog thread to avoid hang.
662:     std::thread ncclHeartbeatMonitorThread_;
663: 
664:     // Whether or not we should terminate the heartbeat monitoring threads.
665:     std::atomic<bool> terminateHeartbeatMonitorThread_{false};
666: 
667:     // Condition Variable for monitor thread to wake up early
668:     std::condition_variable monitorWakeUpCV_;
669: 
670:     // Whether or not to dump debug info on exception including both watchdog
671:     // timeout and nccl errors.
672:     bool dumpOnTimeoutOrEx_;
```

- EN: Lines 649-672 continues the local implementation details and data flow for this file.
- CN: 第 649-672 行继续展开本文件的局部实现细节与数据流。

### Lines 673-696 / 第 673-696 行

```cpp
673: 
674:     // Mutex to Guard monitorWakeUpCV_
675:     std::mutex monitorMutex_;
676: 
677:     // The last update time of WorkList inside watchdog thread.
678:     std::chrono::time_point<std::chrono::steady_clock> lastWorkListUpdateTime_;
679:   };
680: 
681:   // Class that runs as a side thread to check whether the NCCL collective
682:   // is timed out or errors on the cached NCCL communicators.
683:   class Watchdog {
684:    public:
685:     Watchdog(ProcessGroupNCCL* pg);
686:     virtual ~Watchdog() = default;
687: 
688:     // Start the watchdog thread.
689:     void start();
690: 
691:     // Join the watchdog thread.
692:     void join();
693: 
694:     // Function that runs as part of a separate thread and checks for errors on
695:     // NCCL communicators. We need a separate thread to check for NCCL errors
696:     // since we can't rely on the user calling certain methods like wait(),
```

- EN: Lines 673-696 declares or defines types such as `Watchdog`; introduces executable logic in routines such as `Watchdog`, `start`, `join`.
- CN: 第 673-696 行声明或定义了 `Watchdog` 等类型；在 `Watchdog`、`start`、`join` 等例程中引入具体执行逻辑。

### Lines 697-720 / 第 697-720 行

```cpp
697:     // isCompleted() etc. to detect and remediate errors. In addition to this,
698:     // we need a mechanism to safely abort and remove NCCL communicators from
699:     // our cache. This can be done cleanly by having a thread for the
700:     // ProcessGroupNCCL class. Attempting to modify the communicator cache from
701:     // the WorkNCCL class might run into issues with object lifetime since the
702:     // ProcessGroupNCCL object might get destroyed before the WorkNCCL object.
703:     void run();
704: 
705:     // Watchdog's inside loop.
706:     // Takes care of cleaning up completed work, and aborting upon failure or
707:     // timeout.
708:     void runLoop();
709: 
710:     // Notify the loop inside watchdog.
711:     void notify();
712: 
713:     void checkAndSetRemoteError();
714: 
715:     // A helper function to get the src rank of a signal from the Store. This is
716:     // nonblocking function returning -1 if the signal is not available yet.
717:     int getSignalSrcRank(
718:         c10::intrusive_ptr<Store>& store,
719:         const std::string& signal);
720: 
```

- EN: Lines 697-720 introduces executable logic in routines such as `run`, `runLoop`, `notify`.
- CN: 第 697-720 行在 `run`、`runLoop`、`notify` 等例程中引入具体执行逻辑。

### Lines 721-744 / 第 721-744 行

```cpp
721:     uint64_t getHeartbt() const;
722: 
723:     void setDesyncDebug(bool desyncDebug);
724: 
725:    private:
726:     std::thread ncclCommWatchdogThread_;
727: 
728:     // We need to keep a reference to the PG instance so that we can access
729:     // the member functions of the PG instance. We store a raw pointer on
730:     // purpose because the watchdog thread now still lives within the
731:     // lifetime of the PG instance.
732:     ProcessGroupNCCL* pg_;
733: 
734:     // Whether the NCCL watchdog should rethrow CUDA errors.
735:     bool rethrowCUDAErrors_ = false;
736: 
737:     std::exception_ptr watchDogException_ = nullptr;
738: 
739:     // Condition Variable for watchdog thread sleep
740:     std::condition_variable workMetaListCV_;
741: 
742:     // Heartbeat of watchdog thread.
743:     std::atomic_uint64_t heartbeat_;
744: 
```

- EN: Lines 721-744 introduces executable logic in routines such as `getHeartbt`, `setDesyncDebug`; performs validation and error handling to keep distributed state consistent.
- CN: 第 721-744 行在 `getHeartbt`、`setDesyncDebug` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 745-768 / 第 745-768 行

```cpp
745:     // Whether or not to propagate detected errors to all ranks in the same PG
746:     // through TCPStore.
747:     bool propagatePgError_;
748: 
749:     // Whether or not to enable timeout root cause analysis.
750:     bool desyncDebug_;
751: 
752:     DesyncDebugger desyncDebugger_;
753:   };
754: 
755:   // If you wish to create multiple process groups, each with a potentially
756:   // different rank and size, you can do so by passing a new store instance
757:   // to each one. If you have only a single store object, you can
758:   // use the `c10d::PrefixStore` to derive scoped instances.
759:   // This is also what the Python API in torch.distributed does.
760:   //
761:   // The process group instance keeps a reference to the store because
762:   // it may be used long after the constructor runs. In fact, the constructor
763:   // doesn't create any NCCL communicators. A single NCCL communicator can
764:   // only be used on a specific set of devices, and are therefore created
765:   // on-demand when a collective runs. If another collective is executed later,
766:   // against a different set of devices, the process group creates another NCCL
767:   // communicator. These NCCL communicators are cached and reused if possible.
768:   //
```

- EN: Lines 745-768 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 745-768 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 769-792 / 第 769-792 行

```cpp
769:   ProcessGroupNCCL(
770:       c10::intrusive_ptr<Store> store,
771:       int rank,
772:       int size,
773:       c10::intrusive_ptr<Options> options = Options::create());
774: 
775:   // This constructor includes the deprecated `groupName` argument.
776:   // If you have existing code that uses the `groupName`, you can replace
777:   // it by specifying a `c10d::PrefixStore(groupName, store)` for store.
778:   C10_DEPRECATED ProcessGroupNCCL(
779:       const c10::intrusive_ptr<Store>& store,
780:       int rank,
781:       int size,
782:       const std::string& groupName,
783:       c10::intrusive_ptr<Options> options = Options::create())
784:       : ProcessGroupNCCL(store, rank, size, std::move(options)) {}
785: 
786:   ~ProcessGroupNCCL() override;
787: 
788:   // This function returns a local uid for ProcessGroupNCCL.
789:   uint64_t getUid() {
790:     return static_cast<uint64_t>(local_id_);
791:   }
792: 
```

- EN: Lines 769-792 introduces executable logic in routines such as `~ProcessGroupNCCL`, `getUid`; returns computed state or forwards results to the surrounding caller.
- CN: 第 769-792 行在 `~ProcessGroupNCCL`、`getUid` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 793-816 / 第 793-816 行

```cpp
793:   c10::intrusive_ptr<Options> getOptions() {
794:     return options_;
795:   }
796: 
797:   c10::intrusive_ptr<Backend::Options> getBackendOptions() override {
798:     return c10::static_intrusive_pointer_cast<Backend::Options>(options_);
799:   }
800: 
801:   const std::string getBackendName() const override {
802:     return std::string(NCCL_BACKEND_NAME);
803:   }
804: 
805:   bool supportsSplitting() const override {
806:     return true;
807:   }
808: 
809:   bool supportsCoalescing() const override {
810:     return true;
811:   }
812: 
813:   bool supportsTimeEstimation() const override {
814: #ifdef NCCL_SIM_INFO_INITIALIZER
815:     return true;
816: #else
```

- EN: Lines 793-816 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `getOptions`, `getBackendOptions`, `getBackendName`.
- CN: 第 793-816 行使用条件编译来适配特性开关、平台或可选后端；在 `getOptions`、`getBackendOptions`、`getBackendName` 等例程中引入具体执行逻辑。

### Lines 817-840 / 第 817-840 行

```cpp
817:     return false;
818: #endif
819:   }
820: 
821:   void setTimeout(std::chrono::milliseconds timeout) override {
822:     options_->timeout = timeout;
823:   }
824: 
825:   void startCoalescing() override;
826: 
827:   c10::intrusive_ptr<Work> endCoalescing() override;
828: 
829:   void startTimeEstimate();
830: 
831:   float endTimeEstimate();
832: 
833:   // For specifying a composite optype, such as ALLGATHER and REDUCE_SCATTER
834:   c10::intrusive_ptr<Work> endCoalescing(OpType optype);
835: 
836:   c10::intrusive_ptr<Work> broadcast(
837:       std::vector<at::Tensor>& tensors,
838:       const BroadcastOptions& opts = BroadcastOptions()) override;
839: 
840:   c10::intrusive_ptr<Work> _broadcast_oop(
```

- EN: Lines 817-840 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `setTimeout`, `startCoalescing`, `endCoalescing`.
- CN: 第 817-840 行使用条件编译来适配特性开关、平台或可选后端；在 `setTimeout`、`startCoalescing`、`endCoalescing` 等例程中引入具体执行逻辑。

### Lines 841-864 / 第 841-864 行

```cpp
841:       at::Tensor& outputTensors,
842:       at::Tensor& inputTensors,
843:       const BroadcastOptions& opts = BroadcastOptions());
844: 
845:   c10::intrusive_ptr<Work> allreduce_sparse(
846:       std::vector<at::Tensor>& tensors,
847:       const AllreduceOptions& opts = AllreduceOptions()) override;
848: 
849:   c10::intrusive_ptr<Work> allreduce(
850:       std::vector<at::Tensor>& tensors,
851:       const AllreduceOptions& opts = AllreduceOptions()) override;
852: 
853:   c10::intrusive_ptr<Work> allreduce_coalesced(
854:       std::vector<at::Tensor>& tensors,
855:       const AllreduceCoalescedOptions& opts =
856:           AllreduceCoalescedOptions()) override;
857: 
858:   c10::intrusive_ptr<Work> reduce(
859:       std::vector<at::Tensor>& tensors,
860:       const ReduceOptions& opts = ReduceOptions()) override;
861: 
862:   c10::intrusive_ptr<Work> _reduce_oop(
863:       at::Tensor& outputTensors,
864:       at::Tensor& inputTensors,
```

- EN: Lines 841-864 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 841-864 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 865-888 / 第 865-888 行

```cpp
865:       const ReduceOptions& opts = ReduceOptions());
866: 
867:   c10::intrusive_ptr<Work> allgather(
868:       std::vector<std::vector<at::Tensor>>& outputTensors,
869:       std::vector<at::Tensor>& inputTensors,
870:       const AllgatherOptions& opts = AllgatherOptions()) override;
871: 
872:   c10::intrusive_ptr<Work> _allgather_base(
873:       at::Tensor& outputbuffer,
874:       at::Tensor& inputbuffer,
875:       const AllgatherOptions& opts = AllgatherOptions()) override;
876: 
877:   c10::intrusive_ptr<Work> allgather_coalesced(
878:       std::vector<std::vector<at::Tensor>>& outputTensorLists,
879:       std::vector<at::Tensor>& inputTensors,
880:       const AllgatherOptions& opts = AllgatherOptions()) override;
881: 
882:   c10::intrusive_ptr<Work> allgather_into_tensor_coalesced(
883:       std::vector<at::Tensor>& outputs,
884:       std::vector<at::Tensor>& inputs,
885:       const AllgatherOptions& opts = AllgatherOptions()) override;
886: 
887:   c10::intrusive_ptr<Work> reduce_scatter(
888:       std::vector<at::Tensor>& outputTensors,
```

- EN: Lines 865-888 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 865-888 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 889-912 / 第 889-912 行

```cpp
889:       std::vector<std::vector<at::Tensor>>& inputTensors,
890:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
891: 
892:   c10::intrusive_ptr<Work> _reduce_scatter_base(
893:       at::Tensor& outputTensor,
894:       at::Tensor& inputTensor,
895:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
896: 
897:   c10::intrusive_ptr<Work> reduce_scatter_tensor_coalesced(
898:       std::vector<at::Tensor>& outputs,
899:       std::vector<at::Tensor>& inputs,
900:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
901: 
902:   c10::intrusive_ptr<Work> barrier(
903:       const BarrierOptions& opts = BarrierOptions()) override;
904: 
905:   c10::intrusive_ptr<Work> alltoall_base(
906:       at::Tensor& outputTensor,
907:       at::Tensor& inputTensor,
908:       std::vector<int64_t>& outputSplitSizes,
909:       std::vector<int64_t>& inputSplitSizes,
910:       const AllToAllOptions& opts = AllToAllOptions()) override;
911: 
912:   c10::intrusive_ptr<Work> alltoall(
```

- EN: Lines 889-912 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 889-912 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 913-936 / 第 913-936 行

```cpp
913:       std::vector<at::Tensor>& outputTensors,
914:       std::vector<at::Tensor>& inputTensors,
915:       const AllToAllOptions& opts = AllToAllOptions()) override;
916: 
917:   c10::intrusive_ptr<Work> send(
918:       std::vector<at::Tensor>& tensors,
919:       int dstRank,
920:       int tag) override;
921: 
922:   c10::intrusive_ptr<Work> recv(
923:       std::vector<at::Tensor>& tensors,
924:       int srcRank,
925:       int tag) override;
926: 
927:   int64_t getCommPtr();
928: 
929:   void groupStart();
930: 
931:   void groupEnd();
932: 
933:   void groupEndNonblocking(const std::shared_ptr<NCCLComm>& comm);
934: 
935:   c10::intrusive_ptr<Work> gather(
936:       std::vector<std::vector<at::Tensor>>& outputTensors,
```

- EN: Lines 913-936 introduces executable logic in routines such as `send`, `recv`, `getCommPtr`.
- CN: 第 913-936 行在 `send`、`recv`、`getCommPtr` 等例程中引入具体执行逻辑。

### Lines 937-960 / 第 937-960 行

```cpp
937:       std::vector<at::Tensor>& inputTensors,
938:       const GatherOptions& opts = GatherOptions()) override;
939: 
940:   c10::intrusive_ptr<Work> scatter(
941:       std::vector<at::Tensor>& outputTensors,
942:       std::vector<std::vector<at::Tensor>>& inputTensors,
943:       const ScatterOptions& opts = ScatterOptions()) override;
944: 
945:   // Unsupported Ops
946:   c10::intrusive_ptr<Work> recvAnysource(
947:       std::vector<at::Tensor>& tensors,
948:       int tag) override;
949: 
950:   // Agrees on an initial sequence number for the whole group by having rank 0
951:   // create it and broadcast it to other ranks using the store.
952:   void setSequenceNumberForGroup() override;
953: 
954:   // Retrieves the current sequence number for the whole group, which should be
955:   // in sync. If the returned number is not consistent across the group, it
956:   // may indicate that there is some sort of collective desynchronization.
957:   uint64_t getSequenceNumberForGroup() override;
958: 
959:   // Return the total number of splits the communicators held by this process
960:   // group have performed.  Counts ncclCommCreateFromRanks() for ncclx v2.21.5+
```

- EN: Lines 937-960 introduces executable logic in routines such as `recvAnysource`, `setSequenceNumberForGroup`, `getSequenceNumberForGroup`.
- CN: 第 937-960 行在 `recvAnysource`、`setSequenceNumberForGroup`、`getSequenceNumberForGroup` 等例程中引入具体执行逻辑。

### Lines 961-984 / 第 961-984 行

```cpp
961:   uint64_t getCommSplitCounter() const;
962: 
963:   void registerOnCompletionHook(
964:       std::function<void(std::shared_ptr<WorkInfo>)>&& hook) override;
965:   void waitForPendingWorks() override;
966: 
967:   void enableCollectivesTiming() override;
968: 
969:   c10::intrusive_ptr<Backend> split(
970:       const c10::intrusive_ptr<Store>& store,
971:       const std::vector<int>& ranks,
972:       const c10::intrusive_ptr<Backend::Options>& opts) override;
973: 
974:   c10::intrusive_ptr<Backend> merge(
975:       const c10::intrusive_ptr<Store>& store,
976:       const c10::intrusive_ptr<Backend::Options>& opts,
977:       const int& rank,
978:       const int& size) override;
979: 
980:   // Helper function for iteratively aborting communicators in the provided map
981:   void abortCommsFromMap(
982:       std::unordered_map<std::string, std::shared_ptr<NCCLComm>>& ncclCommsMap,
983:       const std::optional<std::string>& abortReason);
984: 
```

- EN: Lines 961-984 introduces executable logic in routines such as `getCommSplitCounter`, `waitForPendingWorks`, `enableCollectivesTiming`.
- CN: 第 961-984 行在 `getCommSplitCounter`、`waitForPendingWorks`、`enableCollectivesTiming` 等例程中引入具体执行逻辑。

### Lines 985-1008 / 第 985-1008 行

```cpp
985:   c10::intrusive_ptr<intra_node_comm::IntraNodeComm> initIntraNodeComm();
986: 
987:   // Destroy (shutdown) this backend -- normal exit.
988:   void shutdown() override;
989: 
990:   // Provides an API to abort the ProcessGroup (similar to ncclCommAbort)
991:   // instead of relying on ProcessGroupNCCL destructor.
992:   void abort() override;
993: 
994:   void eagerConnectSingleDevice(at::Device device) override;
995: 
996:   void performNocolorSplit(at::Device device);
997: 
998:   // If all comms on this PG are fully initialized, return true.
999:   bool isInitialized();
1000: 
1001:   ErrorType getError() override;
1002: 
1003:   bool supportsShrinking() const override {
1004: #ifdef NCCL_HAS_COMM_SHRINK
1005:     return true;
1006: #else
1007:     return false;
1008: #endif
```

- EN: Lines 985-1008 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `initIntraNodeComm`, `shutdown`, `abort`.
- CN: 第 985-1008 行使用条件编译来适配特性开关、平台或可选后端；在 `initIntraNodeComm`、`shutdown`、`abort` 等例程中引入具体执行逻辑。

### Lines 1009-1032 / 第 1009-1032 行

```cpp
1009:   }
1010: 
1011:   // Backend-style shrink override that returns a Backend instance.
1012:   c10::intrusive_ptr<Backend> shrink(
1013:       const std::vector<int64_t>& ranks_to_exclude,
1014:       int shrink_flags = 0,
1015:       const c10::intrusive_ptr<Backend::Options>& opts_override =
1016:           nullptr) override;
1017: 
1018:   std::shared_ptr<c10::Allocator> getMemAllocator() override;
1019: 
1020:   // Allocate tensor from communication-optimized memory pool
1021:   at::Tensor allocateTensor(long size, at::TensorOptions options = {}) override;
1022: 
1023:   // Whether tensor allocation from NCCL memory pool is supported
1024:   bool supportsTensorAlloc(c10::DeviceIndex deviceIdx) override;
1025: 
1026:   // Performs NCCL user buffer registration for all buffers in
1027:   // the given MemPool
1028:   void registerMemPool(at::cuda::MemPool* pool, bool symm = false);
1029: 
1030:   // Performs NCCL user buffer de-registration for all buffers in
1031:   // the given MemPool
1032:   void deregisterMemPool(at::cuda::MemPool* pool);
```

- EN: Lines 1009-1032 introduces executable logic in routines such as `shrink`, `getMemAllocator`, `supportsTensorAlloc`.
- CN: 第 1009-1032 行在 `shrink`、`getMemAllocator`、`supportsTensorAlloc` 等例程中引入具体执行逻辑。

### Lines 1033-1056 / 第 1033-1056 行

```cpp
1033: 
1034:   // This method adds a temporary extension for the timeout period,
1035:   // applying to all collectives between the calling of this API and
1036:   // the completion of the first collective on the GPU. While this feature
1037:   // provides flexibility in specific scenarios, it introduces statefulness
1038:   // to timeout setting. Therefore, it is advisable to use this API sparingly
1039:   // and consider alternative approaches, such as directly setting the timeout
1040:   // or utilizing a barrier collective (one can set any timeout to the barrier),
1041:   // whenever feasible.
1042:   void addEphemeralTimeout(const std::chrono::milliseconds& timeout);
1043: 
1044:   // This function is only intended for testing purposes because we don't
1045:   // want to expose the `WorkNCCL` via pybind. It verifies whether the
1046:   // `opTimeout_` of the provided WorkNCCL instance is the same as the specified
1047:   // timeout.
1048:   bool verifyWorkTimeoutForTest(
1049:       const c10::intrusive_ptr<Work>& work,
1050:       const std::chrono::milliseconds& timeout);
1051: 
1052:   void setEnableNanCheck(bool enableNanCheck);
1053: 
1054:   // APIs related to memory offload (require NCCL 2.29.7+ at runtime)
1055:   void suspend() override;
1056: 
```

- EN: Lines 1033-1056 introduces executable logic in routines such as `addEphemeralTimeout`, `verifyWorkTimeoutForTest`, `setEnableNanCheck`.
- CN: 第 1033-1056 行在 `addEphemeralTimeout`、`verifyWorkTimeoutForTest`、`setEnableNanCheck` 等例程中引入具体执行逻辑。

### Lines 1057-1080 / 第 1057-1080 行

```cpp
1057:   void resume() override;
1058: 
1059:   std::unordered_map<std::string, uint64_t> getMemoryStats() override;
1060: 
1061:  protected:
1062:   uint64_t getWatchdogHeartbt() const;
1063: 
1064:   // Instance of the heartbeat monitor thread.
1065:   std::unique_ptr<HeartbeatMonitor> heartbeatMonitor_;
1066: 
1067:   // Instance of the watchdog thread.
1068:   std::unique_ptr<Watchdog> watchdog_;
1069: 
1070:   // Helper that broadcasts nccl unique ID to all ranks through the store
1071:   void broadcastUniqueNCCLID(
1072:       ncclUniqueId* ncclID,
1073:       bool isSingleP2POp,
1074:       const std::string& devicesKey,
1075:       int p2pRank);
1076: 
1077:   // Helper that allgathers nccl unique IDs to all ranks through the store
1078:   void allgatherUniqueNCCLIDs(
1079:       int rootIdx,
1080:       ncclUniqueId* ncclID,
```

- EN: Lines 1057-1080 introduces executable logic in routines such as `resume`, `getMemoryStats`, `getWatchdogHeartbt`.
- CN: 第 1057-1080 行在 `resume`、`getMemoryStats`、`getWatchdogHeartbt` 等例程中引入具体执行逻辑。

### Lines 1081-1104 / 第 1081-1104 行

```cpp
1081:       std::vector<ncclUniqueId>& ncclIDs);
1082: 
1083:   // Helper that looks up the cached NCCL communicators only
1084:   std::shared_ptr<NCCLComm> getNCCLComm(const std::string& deviceKey);
1085: 
1086:   std::shared_ptr<NCCLComm> initNCCLComm(
1087:       const std::string& deviceKey,
1088:       at::Device& device,
1089:       OpType opType,
1090:       int p2pRank = 0,
1091:       bool isSendRecvSelf = false);
1092: 
1093:   // Initialize device-specific state (comm, stream, event, bookkeeping) for a
1094:   // given communicator on this process group instance.
1095:   void initializeDeviceStateForComm(
1096:       const at::Device& device,
1097:       std::shared_ptr<NCCLComm> comm);
1098: 
1099:   // Wrapper method which can be overridden for tests.
1100:   virtual std::exception_ptr checkForNCCLErrors(
1101:       std::shared_ptr<NCCLComm>& ncclComm);
1102: 
1103:   // Ensure thaht if record is True, the work obj will be enqueued via
1104:   // workEnqueue
```

- EN: Lines 1081-1104 introduces executable logic in routines such as `getNCCLComm`, `initNCCLComm`, `initializeDeviceStateForComm`.
- CN: 第 1081-1104 行在 `getNCCLComm`、`initNCCLComm`、`initializeDeviceStateForComm` 等例程中引入具体执行逻辑。

### Lines 1105-1128 / 第 1105-1128 行

```cpp
1105:   virtual c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL> initWork(
1106:       at::Device& device,
1107:       int rank,
1108:       OpType opType,
1109:       bool isP2P,
1110:       const char* profilingTitle = nullptr,
1111:       const std::vector<at::Tensor>& inputs = {},
1112:       const std::vector<at::Tensor>& outputs = {},
1113:       bool record = false);
1114: 
1115:   // In the timeout case and we will dump debug info such as the NCCL flight
1116:   // recorder to storage. Down the road, if we have more complicated or blocking
1117:   // operations, we might need to use a side thread to do it.
1118:   bool dumpDebuggingInfo(
1119:       bool includeStackTrace = true,
1120:       bool onlyActive = false);
1121: 
1122:   void dumpExtraDebuggingInfo();
1123: 
1124:   // Abort all communicators on this rank.
1125:   bool abortComms(const std::optional<std::string>& abortReason = std::nullopt);
1126: 
1127:   // A helper function to check if nonblocking API mode should be used.
1128:   // Use this helper instead of directly checking `useNonblocking_` variable.
```

- EN: Lines 1105-1128 introduces executable logic in routines such as `dumpDebuggingInfo`, `dumpExtraDebuggingInfo`, `abortComms`.
- CN: 第 1105-1128 行在 `dumpDebuggingInfo`、`dumpExtraDebuggingInfo`、`abortComms` 等例程中引入具体执行逻辑。

### Lines 1129-1152 / 第 1129-1152 行

```cpp
1129:   bool useNonblocking();
1130: 
1131:  protected:
1132:   int globalRankStart_{};
1133:   int globalRankStride_{};
1134: 
1135:  private:
1136:   bool eagerInit_{false};
1137:   bool showSerializationWarning_{true};
1138: 
1139:   // Helper that encapsulates work shared across all collective communication
1140:   // primitives.  The callbacks have the following signatures:
1141:   //
1142:   //    ncclResult_t fn(at::Tensor& input, at::Tensor& output,
1143:   //                    ncclComm_t, at::cuda::CUDAStream&);
1144:   //    void {pre,post}(std::vector<at::cuda::CUDAStream&>);
1145:   template <typename Fn>
1146:   c10::intrusive_ptr<Work> collective(
1147:       at::Tensor& input,
1148:       at::Tensor& output,
1149:       Fn fn,
1150:       OpType opType,
1151:       bool asyncOp,
1152:       const char* profilingTitle = nullptr,
```

- EN: Lines 1129-1152 introduces executable logic in routines such as `useNonblocking`.
- CN: 第 1129-1152 行在 `useNonblocking` 等例程中引入具体执行逻辑。

### Lines 1153-1176 / 第 1153-1176 行

```cpp
1153:       bool nanCheck = true);
1154: 
1155:   template <typename Fn, typename PreProcess, typename PostProcess>
1156:   c10::intrusive_ptr<Work> collective(
1157:       at::Tensor& input,
1158:       at::Tensor& output,
1159:       Fn fn,
1160:       PreProcess pre,
1161:       PostProcess post,
1162:       OpType opType,
1163:       bool asyncOp,
1164:       const char* profilingTitle = nullptr,
1165:       bool nanCheck = true);
1166: 
1167:   template <typename Fn, typename PreProcess, typename PostProcess>
1168:   c10::intrusive_ptr<Work> collective(
1169:       std::vector<at::Tensor>& inputs,
1170:       std::vector<at::Tensor>& outputs,
1171:       Fn fn,
1172:       PreProcess pre,
1173:       PostProcess post,
1174:       OpType opType,
1175:       bool asyncOp,
1176:       const char* profilingTitle = nullptr,
```

- EN: Lines 1153-1176 introduces executable logic in routines such as `collective`.
- CN: 第 1153-1176 行在 `collective` 等例程中引入具体执行逻辑。

### Lines 1177-1200 / 第 1177-1200 行

```cpp
1177:       bool nanCheck = true);
1178: 
1179:   template <typename Fn>
1180:   c10::intrusive_ptr<Work> collectiveCoalesced(
1181:       std::vector<at::Tensor>& input,
1182:       std::vector<at::Tensor>& output,
1183:       Fn fn,
1184:       OpType opType,
1185:       bool asyncOp,
1186:       const char* profilingTitle = nullptr);
1187: 
1188:   // Helper that encapsulates work shared across point-to-point communication
1189:   // primitives. It is the same structure as the helper used for collective
1190:   // communication primitives.
1191:   template <typename Fn>
1192:   c10::intrusive_ptr<Work> pointToPoint(
1193:       at::Tensor& tensor,
1194:       Fn fn,
1195:       int peer,
1196:       OpType opType,
1197:       const char* profilingTitle = nullptr);
1198: 
1199:   template <typename Fn, typename PreProcess, typename PostProcess>
1200:   c10::intrusive_ptr<Work> pointToPoint(
```

- EN: Lines 1177-1200 introduces executable logic in routines such as `collectiveCoalesced`, `pointToPoint`.
- CN: 第 1177-1200 行在 `collectiveCoalesced`、`pointToPoint` 等例程中引入具体执行逻辑。

### Lines 1201-1224 / 第 1201-1224 行

```cpp
1201:       at::Tensor& tensor,
1202:       Fn fn,
1203:       int peer,
1204:       OpType opType,
1205:       PreProcess pre,
1206:       PostProcess post,
1207:       const char* profilingTitle);
1208: 
1209:   c10::intrusive_ptr<Work> allreduce_impl(
1210:       at::Tensor& tensor,
1211:       const char* profilingTitle = "nccl:all_reduce",
1212:       const AllreduceOptions& opts = AllreduceOptions());
1213: 
1214:   // Checks for NCCL errors on each of the communicators and returns an
1215:   // appropriate exception_ptr (nullptr if no errors).
1216:   static std::exception_ptr checkForNCCLErrorsInternal(
1217:       std::shared_ptr<NCCLComm>& ncclComm);
1218: 
1219:   // Return the CUDA device most likely associated with this backend.
1220:   // If we aren't bound to a specific device, there is no strict
1221:   // guarantee that this heuristic is the correct assignment of ranks
1222:   // to GPUs that Python layers use, but in practice it tends to be.
1223:   // Fortunately we don't rely on this for correctness of any tensor
1224:   // operations, just for ancillary uses like barriers.
```

- EN: Lines 1201-1224 introduces executable logic in routines such as `checkForNCCLErrorsInternal`.
- CN: 第 1201-1224 行在 `checkForNCCLErrorsInternal` 等例程中引入具体执行逻辑。

### Lines 1225-1248 / 第 1225-1248 行

```cpp
1225:   at::Device guessDeviceForRank() const;
1226: 
1227:   // Destroys initialized NCCL communicators in devNCCLComMap_ given by input
1228:   // key. Throws if there are no communicators to destroy. Also removes
1229:   // communicators from the cache and clears used device indices.
1230:   void destroyNCCLComms(const std::string& devNCCLCommMapKey);
1231: 
1232:   void runHookLoop();
1233: 
1234:   // Generates a prefix that is unique to this process group and rank, for
1235:   // disambiguating logs
1236:   std::string createLogPrefix() const;
1237: 
1238:   // Returns the unique prefix created in createLogPrefix
1239:   const std::string& logPrefix() const;
1240: 
1241:   // Returns the global rank of the device. This function assumes that users
1242:   // always create a default global process group(PG) which includes all
1243:   // devices. It is called in the constructor of ProcessGroupNCCL, so it always
1244:   // return the rank_ of the very first PG created, aka, default global PG.
1245:   const int& globalRank() const;
1246: 
1247:   const c10::intrusive_ptr<Store>& globalStore() const;
1248: 
```

- EN: Lines 1225-1248 introduces executable logic in routines such as `guessDeviceForRank`, `destroyNCCLComms`, `runHookLoop`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1225-1248 行在 `guessDeviceForRank`、`destroyNCCLComms`、`runHookLoop` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1249-1272 / 第 1249-1272 行

```cpp
1249:   // Returns the global ranks of a PG.
1250:   const std::vector<uint64_t>& groupRanks() const;
1251: 
1252:   // Util function to assign timeout to each work.
1253:   void assignTimeoutToWork(
1254:       const c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work,
1255:       const c10::intrusive_ptr<Options>& option);
1256: 
1257:   // Broadcast flight-recorder dump signal
1258:   void broadcastDumpSignal();
1259: 
1260:   // A helper function to broadcast a signal (key) from a src rank to all other
1261:   // ranks using the specified store.
1262:   void broadcastSignal(
1263:       c10::intrusive_ptr<Store>& store,
1264:       const std::string& signal,
1265:       int srcRank);
1266: 
1267:  protected:
1268:   // Function that directly trigger std::abort so that the whole process
1269:   // gets terminated.
1270:   virtual void terminateProcess(const std::string& errMsg);
1271: 
1272:   // A helper function to wait for a future to complete or timeout.
```

- EN: Lines 1249-1272 introduces executable logic in routines such as `groupRanks`, `assignTimeoutToWork`, `broadcastDumpSignal`.
- CN: 第 1249-1272 行在 `groupRanks`、`assignTimeoutToWork`、`broadcastDumpSignal` 等例程中引入具体执行逻辑。

### Lines 1273-1296 / 第 1273-1296 行

```cpp
1273:   // Returns true if the future completes before timeout, false otherwise.
1274:   bool waitForFutureOrTimeout(
1275:       std::future<bool>& fut,
1276:       const std::chrono::milliseconds& timeOutMilSec,
1277:       const std::string& futDescription,
1278:       ::c10d::C10dLoggingData& debugLog,
1279:       bool throwException = false);
1280: 
1281:   // A helper function to guess the device id of the current rank, based on
1282:   // bounded device or used device. Do not use this function if you already know
1283:   // the device id to operate on.
1284:   c10::DeviceIndex guessDeviceId() const;
1285: 
1286:   static const int64_t kWatchdogThreadSleepMillis;
1287: 
1288:   // The store is used to broadcast the NCCL unique ID of rank 0. This store
1289:   // comes with prefix and it is different across ProcessGroup NCCL instances
1290:   // (aka, different ProcessGroups).
1291:   c10::intrusive_ptr<Store> store_;
1292: 
1293:   // Reference to the store without prefix so that keys are same across all
1294:   // ProcessGroup NCCL instances and (key, value) pairs written to the store are
1295:   // global.
1296:   c10::intrusive_ptr<Store> globalStore_;
```

- EN: Lines 1273-1296 introduces executable logic in routines such as `waitForFutureOrTimeout`, `guessDeviceId`.
- CN: 第 1273-1296 行在 `waitForFutureOrTimeout`、`guessDeviceId` 等例程中引入具体执行逻辑。

### Lines 1297-1320 / 第 1297-1320 行

```cpp
1297: 
1298:   // The lock which protects the write/read of
1299:   // ephemeralTimeoutActive_/ephemeralTimeoutInflight_.
1300:   // TODO(fduwjj): We need to have an audit on all mutexes we are adding here.
1301:   // And consolidate them if possible.
1302:   std::mutex mtxTimeoutExtension_;
1303: 
1304:   // The ephemeral timeout added on top of existing timeout for works issued
1305:   // before first work finishes.
1306:   std::chrono::milliseconds ephemeralTimeoutActive_ =
1307:       std::chrono::milliseconds(0);
1308: 
1309:   // The ephemeral timeout addition which has been already applied to work.
1310:   std::chrono::milliseconds ephemeralTimeoutInflight_ =
1311:       std::chrono::milliseconds(0);
1312: 
1313:   const c10::intrusive_ptr<Options> options_;
1314: 
1315:   // The number of NCCL communicators that have been created during
1316:   // the lifetime of this process group. This sequence number is
1317:   // used to scope keys used in the store.
1318:   uint64_t ncclCommCounter_{0};
1319: 
1320:   // The NCCL communicator that the process group has cached.
```

- EN: Lines 1297-1320 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1297-1320 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1321-1344 / 第 1321-1344 行

```cpp
1321:   //
1322:   // For collective operations:
1323:   // The key is a list of GPU devices that an operation is operating on
1324:   // The GPU devices are stored in a device sequence and the cache NCCL
1325:   // communicator is associated with this GPU device sequence
1326:   //
1327:   // e.g. If the process group op only uses device 0, then the value of
1328:   // the used device string stored (value of the hashmap) would be "0".
1329:   //
1330:   //      If the process group op uses device 0 - 7 and the each tensor of the
1331:   //      input tensor list is on device, 0, 1, 2, 3, 4, 5, 6, 7 separately,
1332:   //      then the value of the used device string (key) stored would be
1333:   //      "0,1,2,3,4,5,6,7"
1334:   //
1335:   //      If the process group op uses device 0 - 7 and the each tensor of the
1336:   //      input tensor list is on device, 0, 4, 5, 6, 7, 1, 2, 3 separately,
1337:   //      then the value of the used device string stored would be
1338:   //      "0,4,5,6,7,1,2,3"
1339:   //
1340:   //      Note that the order of the device for the tensor list matters.
1341:   //
1342:   // For point-to-point operations:
1343:   // The key is a string of my current rank and the peer process rank.
1344:   // e.g. If process 1 and process 2 are involved in a point-to-point
```

- EN: Lines 1321-1344 continues the local implementation details and data flow for this file.
- CN: 第 1321-1344 行继续展开本文件的局部实现细节与数据流。

### Lines 1345-1368 / 第 1345-1368 行

```cpp
1345:   // communication, the key will be "1:2" on both processes. Note: this is for
1346:   // the scenario where there is only 1 GPU per process. When it comes to
1347:   // multiple GPUs per process, this part may need to redesigned.
1348:   // TODO: we probably need a separate map for P2P comms
1349:   std::unordered_map<std::string, std::shared_ptr<NCCLComm>> devNCCLCommMap_;
1350: 
1351:   // The NCCL communicators currently in process of being initialized.
1352:   std::unordered_map<std::string, std::shared_ptr<NCCLComm>>
1353:       inInitializationCommMap_;
1354: 
1355:   // Mutex to guard maps like devNCCLCommMap_.
1356:   std::mutex mutex_;
1357: 
1358:   // Size of ring buffer where we store NCCL Traces for debugging.
1359:   int traceBufferSize_;
1360: 
1361:   // Stores TORCH_NCCL_DEBUG_INFO_PIPE_FILE
1362:   std::string debugInfoPipeFile_;
1363: 
1364:   // We gate the cudaEventCache so that we can roll it out gradually.
1365:   std::atomic<bool> cudaEventCacheEnabled_;
1366: 
1367:   std::thread onCompletionHookThread_;
1368: 
```

- EN: Lines 1345-1368 continues the local implementation details and data flow for this file.
- CN: 第 1345-1368 行继续展开本文件的局部实现细节与数据流。

### Lines 1369-1392 / 第 1369-1392 行

```cpp
1369:   // Whether or not we should terminate the watchdog and workCleanup threads.
1370:   std::atomic<bool> terminateProcessGroup_;
1371: 
1372:   // Whether there are hooks pending to be fired
1373:   std::atomic<bool> hasPendingHooks_;
1374: 
1375:   // This is the signal from watchdog threads to indicate whether the monitor
1376:   // thread should dump. Making it static so that it is accessible from all the
1377:   // PGs. With this flag, monitor thread would dump debug info under any one of
1378:   // the three conditions:
1379:   //
1380:   // 1: watchdog thread of any PG detects a collective timeout.
1381:   // 2: timeout signal is received from other ranks through tcpstore.
1382:   // 3: current PG's watchdog heartbeat timeout occurs.
1383:   //
1384:   // Note that only the monitor thread from PG0 will dump the debug info for
1385:   // case one and two so that the debug info is only dumped once.
1386:   static std::atomic<bool> shouldDump_;
1387: 
1388:   // Mutex to Guard workMetaList_
1389:   std::mutex workMetaListMutex_;
1390: 
1391:   bool writeDebugInfo_ = false;
1392: 
```

- EN: Lines 1369-1392 continues the local implementation details and data flow for this file.
- CN: 第 1369-1392 行继续展开本文件的局部实现细节与数据流。

### Lines 1393-1416 / 第 1393-1416 行

```cpp
1393:   // Vector to store WorkNCCL pointers
1394:   std::list<ProcessGroupNCCL::WorkNCCL> workMetaList_;
1395: 
1396:   // Mutex to Guard workMetaList_
1397:   std::mutex completedWorkListMutex_;
1398: 
1399:   // Condition Variable for watchdog thread sleep
1400:   std::condition_variable completedWorkListCV_;
1401: 
1402:   std::list<ProcessGroupNCCL::WorkNCCL> completedWorkList_;
1403: 
1404:   // Add Work Pointer to workVector
1405:   void workEnqueue(
1406:       const c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& /*work*/);
1407: 
1408:   // The CUDA streams used by NCCL kernels
1409:   std::unordered_map<std::string, at::cuda::CUDAStream> ncclStreams_;
1410: 
1411:   // The CUDA events used to sync NCCL streams
1412:   std::unordered_map<std::string, at::cuda::CUDAEvent> ncclEvents_;
1413: 
1414:   // Device Indexes used for all collectives in this group
1415:   std::set<c10::DeviceIndex> usedDeviceIdxs_;
1416: 
```

- EN: Lines 1393-1416 introduces executable logic in routines such as `workEnqueue`.
- CN: 第 1393-1416 行在 `workEnqueue` 等例程中引入具体执行逻辑。

### Lines 1417-1440 / 第 1417-1440 行

```cpp
1417:   // Flag to denote if a coalescing groupStart/groupEnd block is active
1418:   int coalescing_state_ = 0;
1419: 
1420:   // Stores device indexes for all collectives run inside a coalescing block
1421:   at::Device coalescedDevice_ = at::Device("cuda");
1422: 
1423:   // Stores communicators for all collectives run inside a coalescing block
1424:   std::shared_ptr<NCCLComm> coalescedComm_ = nullptr;
1425: 
1426:   // Whether the coalesced calls are sync or async.
1427:   bool coalescedAsync_{};
1428: 
1429:   // keeps track of input and output tensors when coalescing is in flight.  Will
1430:   // hand over these tensors to WorkNCCL's stash when coalescing is ended.
1431:   TensorShelf coalescedTensors_;
1432: 
1433:   // Some ops may have completed, but user still hasn't called `work.wait()`.
1434:   // When watchdog detects this, it transfers the TensorShelf from `work` to
1435:   // this `shelves` structure. Next time we execute ProcessGroupNCCL's methods
1436:   // on main thread, we clear the `shelves` in one shot. This is mainly because
1437:   // watchdog (a side thread) unstashing the shelf directly seems to cause some
1438:   // problem.
1439:   std::vector<std::shared_ptr<TensorShelf>> shelvesToUnstash_;
1440:   std::mutex shelvesMutex_;
```

- EN: Lines 1417-1440 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1417-1440 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1441-1464 / 第 1441-1464 行

```cpp
1441: 
1442:   // Whether or not wait() and synchronize() are blocking operations that wait
1443:   // for the operation to complete.
1444:   bool blockingWait_ = false;
1445: 
1446:   // Whether or not the workCleanupThread is used to perform async error
1447:   // handling.
1448:   ErrorHandlingMode asyncErrorHandling_ = NoHandling;
1449: 
1450:   ErrorType error_ = ErrorType::SUCCESS;
1451: 
1452:   std::mutex errorMutex_;
1453: 
1454:   // Whether or not to sleep after an exception is thrown in the watchdog.
1455:   bool sleepAfterException_{};
1456: 
1457:   // Whether or not to enable nan check for input tensors to collectives.
1458:   bool enableNanCheck_;
1459: 
1460:   // Whether or not to create start CUDAEvent and enable timing for start
1461:   // and end events. Note that enableTiming_ is always true if desyncDebug_
1462:   // is set to true.
1463:   std::atomic<bool> enableTiming_;
1464: 
```

- EN: Lines 1441-1464 continues the local implementation details and data flow for this file.
- CN: 第 1441-1464 行继续展开本文件的局部实现细节与数据流。

### Lines 1465-1488 / 第 1465-1488 行

```cpp
1465:   // Flag to enable the print of hash value of input/output of collectives for
1466:   // verification.
1467:   std::atomic<bool> enableCollectiveHashDebug_;
1468: 
1469:   // Whether or not TORCH_NCCL_AVOID_RECORD_STREAMS was set
1470:   bool avoidRecordStreams_ = false;
1471: 
1472:   // The number of active ncclGroupStart() calls. This counter will be increased
1473:   // by 1 when ncclGroupStart() is called and decreased by 1 when ncclGroupEnd()
1474:   // is called.
1475:   static thread_local uint64_t ncclActiveGroupCounter_;
1476: 
1477:   // Counting for the sequential number of NCCL collective call.
1478:   // (specifically, how many actual kernels we launched, which differs from
1479:   // op_id_ when coalescing is enabled)
1480:   uint64_t seqCollective_{0};
1481: 
1482:   // Counting for the sequential number of NCCL P2P calls.
1483:   uint64_t seqP2P_{0};
1484: 
1485:   // Incrementing counter for logical operations (collective or p2p) issued on
1486:   // the ProcessGroup
1487:   uint64_t op_id_{0};
1488: 
```

- EN: Lines 1465-1488 continues the local implementation details and data flow for this file.
- CN: 第 1465-1488 行继续展开本文件的局部实现细节与数据流。

### Lines 1489-1512 / 第 1489-1512 行

```cpp
1489:   // The number of ProcessGroupNCCL created on the current rank.
1490:   size_t local_id_;
1491: 
1492:   std::string logPrefix_;
1493: 
1494:   c10::intrusive_ptr<intra_node_comm::IntraNodeComm> intraNodeComm_;
1495: 
1496:   // Number of devices on this node.
1497:   int localDeviceCount_{0};
1498: 
1499:   std::shared_ptr<ProcessGroupStatus> pgStatus_ =
1500:       std::make_shared<ProcessGroupStatus>();
1501: 
1502:   // Internal cached value: use NCCL non-blocking API mode or not.
1503:   // Use `useNonblocking()` method instead of accessing this variable directly.
1504:   std::optional<bool> useNonblocking_{std::nullopt};
1505: 
1506:   // Communication-optimized memory pool associated with this PG
1507:   std::unique_ptr<at::cuda::MemPool> memPool_ = nullptr;
1508: };
1509: 
1510: // Reset the flighrecorder recordings for the current rank.
1511: TORCH_API void reset_nccl_trace();
1512: 
```

- EN: Lines 1489-1512 introduces executable logic in routines such as `reset_nccl_trace`.
- CN: 第 1489-1512 行在 `reset_nccl_trace` 等例程中引入具体执行逻辑。

### Lines 1513-1536 / 第 1513-1536 行

```cpp
1513: // Dumps the NCCL comm traces and additional information about the Process
1514: // Group.
1515: TORCH_API std::string dump_nccl_trace(
1516:     bool includeCollectives,
1517:     bool includeStackTraces,
1518:     bool onlyActive);
1519: 
1520: // Dumps the NCCL comm traces and additional information about the Process
1521: // Group in JSON formatted string.
1522: // We don't include stack traces in JSON format as it is far too much data.
1523: TORCH_API std::string dump_nccl_trace_json(
1524:     bool includeCollectives,
1525:     bool onlyActive);
1526: 
1527: // Gets a mutable reference to a global optional function.Heartbeat Monitor
1528: // will use this function to dump traces, if available. Inside fbcode, we
1529: // store a function here that uses an internal tool for process tracing
1530: TORCH_API std::optional<
1531:     std::function<void(std::function<void(const std::string&)>)>>&
1532: get_cpp_trace_dumper();
1533: 
1534: // Similar to get_cpp_trace_dumper, this stores a function defined in
1535: // torch-python layer that lets us check whether the GIL can be acquired,
1536: // helpful for instrumenting in cases where a hang was observed.
```

- EN: Lines 1513-1536 introduces executable logic in routines such as `dump_nccl_trace`, `dump_nccl_trace_json`, `get_cpp_trace_dumper`.
- CN: 第 1513-1536 行在 `dump_nccl_trace`、`dump_nccl_trace_json`、`get_cpp_trace_dumper` 等例程中引入具体执行逻辑。

### Lines 1537-1542 / 第 1537-1542 行

```cpp
1537: typedef bool (*gil_checker_t)();
1538: 
1539: TORCH_API gil_checker_t& get_gil_checker();
1540: } // namespace c10d
1541: 
1542: #endif // USE_C10D_NCCL
```

- EN: Lines 1537-1542 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `get_gil_checker`.
- CN: 第 1537-1542 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `get_gil_checker` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `DumpPipe`, `TensorShelf`, `TORCH_API`, `WorkNCCL`, `ErrorHandlingMode`
- CN: 核心符号：`DumpPipe`、`TensorShelf`、`TORCH_API`、`WorkNCCL`、`ErrorHandlingMode`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/Backend.hpp`, `torch/csrc/distributed/c10d/NCCLUtils.hpp`, `torch/csrc/distributed/c10d/PrefixStore.hpp`, `torch/csrc/distributed/c10d/Store.hpp`, `torch/csrc/distributed/c10d/cuda/CUDAEventCache.hpp`, `torch/csrc/distributed/c10d/logger.hpp`, `torch/csrc/distributed/c10d/symm_mem/intra_node_comm.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/DynamicLibrary.h`, `ATen/cuda/CUDAContext.h`, `ATen/cuda/CUDAEvent.h`, `ATen/cuda/MemPool.h`, `c10/core/Stream.h`, `c10/core/StreamGuard.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/cuda/CUDAGuard.h`, `c10/cuda/CUDAStream.h`, `torch/custom_class.h`
- External or system headers / 外部或系统头文件: `fcntl.h`, `sys/stat.h`, `sys/types.h`, `unistd.h`, `atomic`, `chrono`, `deque`, `future`, `iostream`, `list`, `mutex`, `thread`, ... (+1 more)
- Local symbols / 本地符号: `DumpPipe`, `TensorShelf`, `TORCH_API`, `WorkNCCL`, `ErrorHandlingMode`