# ProcessGroupNCCL.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/ProcessGroupNCCL.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides process-group backend logic in the c10d distributed process-group subsystem. Key types include `RocmWatchdogEventQueryContextGuard`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供进程组后端逻辑。 关键类型包括 `RocmWatchdogEventQueryContextGuard`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行

```cpp
1: #ifdef USE_C10D_NCCL
2: 
3: #include <nlohmann/json.hpp>
4: #include <exception>
5: #include <map>
6: #include <memory>
7: #include <mutex>
8: #include <sstream>
9: #include <stdexcept>
10: #include <tuple>
11: #include <utility>
12: 
13: #include <ATen/cuda/CUDAContext.h>
14: #include <ATen/cuda/CUDAGraph.h>
15: #include <c10/core/DeviceType.h>
16: #include <c10/cuda/CUDAAllocatorConfig.h>
17: #include <c10/cuda/CUDAException.h>
18: #include <c10/cuda/CUDAGraphsC10Utils.h>
19: #include <c10/cuda/CUDAGuard.h>
20: #include <c10/util/Exception.h>
21: #include <c10/util/Logging.h>
22: #include <c10/util/WaitCounter.h>
23: #include <c10/util/hash.h>
24: #include <c10/util/irange.h>
25: #include <c10/util/thread_name.h>
26: #include <torch/csrc/cuda/CUDAPluggableAllocator.h>
27: #include <torch/csrc/cuda/nccl.h>
28: #include <torch/csrc/distributed/c10d/FlightRecorder.hpp>
29: #include <torch/csrc/distributed/c10d/NCCLUtils.hpp>
30: #include <torch/csrc/distributed/c10d/NanCheck.hpp>
```

- EN: Lines 1-30 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 1-30 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 31-60 / 第 31-60 行

```cpp
31: #include <torch/csrc/distributed/c10d/ParamCommsUtils.hpp>
32: #include <torch/csrc/distributed/c10d/PrefixStore.hpp>
33: #include <torch/csrc/distributed/c10d/ProcessGroupNCCL.hpp>
34: #include <torch/csrc/distributed/c10d/TraceUtils.h>
35: #include <torch/csrc/distributed/c10d/Utils.hpp>
36: #include <torch/csrc/distributed/c10d/cuda/utils.hpp>
37: #include <torch/torch.h>
38: #include <optional>
39: 
40: namespace c10d {
41: 
42: constexpr const char* const kNCCLAbortedCommStoreKey = "NCCLABORTEDCOMM";
43: using FlightRecorderCUDA = FlightRecorder<at::cuda::CUDAEvent>;
44: 
45: namespace {
46: 
47: #if defined(NCCL_MAJOR) && \
48:     ((NCCL_MAJOR > 2) || (NCCL_MAJOR == 2) && (NCCL_MINOR >= 10))
49: #define NCCL_HAS_AVG 1
50: #endif // NCCL version >= 2.10
51: 
52: // NCCL op mapping
53: const std::map<ReduceOp::RedOpType, ncclRedOp_t> ncclOp = {
54:     {ReduceOp::MIN, ncclMin},
55:     {ReduceOp::MAX, ncclMax},
56:     {ReduceOp::SUM, ncclSum},
57:     {ReduceOp::PRODUCT, ncclProd},
58: #ifdef NCCL_HAS_AVG
59:     {ReduceOp::AVG, ncclAvg},
60: #endif // NCCL_HAS_AVG
```

- EN: Lines 31-60 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 31-60 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 61-90 / 第 61-90 行

```cpp
61: };
62: 
63: inline bool isUnsupportedFloat8(at::ScalarType t) {
64:   return (
65:       t == at::ScalarType::Float8_e5m2fnuz ||
66:       t == at::ScalarType::Float8_e4m3fnuz ||
67:       t == at::ScalarType::Float8_e8m0fnu
68: #ifndef NCCL_SUPPORTS_FP8
69:       || t == at::ScalarType::Float8_e5m2 || t == at::ScalarType::Float8_e4m3fn
70: #endif
71:   );
72: }
73: 
74: #ifdef ENABLE_NCCL_PREMUL_SUM_SUPPORT
75: template <typename T, ncclDataType_t dataType>
76: ncclRedOpRAII unpackPreMulSum(
77:     const ReduceOp& reduceOp,
78:     const ncclComm_t& comm) {
79:   const auto* preMulSupplement =
80:       reinterpret_cast<PreMulSumSupplement*>(reduceOp.supplement_.get());
81:   ncclRedOp_t preMulSum{};
82:   bool has_tensor = preMulSupplement->tensor_factor.defined();
83:   auto residence = has_tensor ? ncclScalarDevice : ncclScalarHostImmediate;
84:   const T* ptr_factor = has_tensor
85:       ? preMulSupplement->tensor_factor.const_data_ptr<T>()
86:       : nullptr;
87:   T scalar_factor = T(preMulSupplement->double_factor);
88:   ncclRedOpCreatePreMulSum(
89:       &preMulSum,
90:       // https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/api/ops.html#ncclredopcreatepremulsum
```

- EN: Lines 61-90 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `isUnsupportedFloat8`, `unpackPreMulSum`.
- CN: 第 61-90 行使用条件编译来适配特性开关、平台或可选后端；在 `isUnsupportedFloat8`、`unpackPreMulSum` 等例程中引入具体执行逻辑。

### Lines 91-120 / 第 91-120 行

```cpp
91:       // tells us that the scalar input is strictly a multiplier.
92:       // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
93:       /*scalar=*/has_tensor ? const_cast<T*>(ptr_factor) : &scalar_factor,
94:       dataType,
95:       residence,
96:       comm);
97:   return ncclRedOpRAII(preMulSum, comm);
98: }
99: #endif // ENABLE_NCCL_PREMUL_SUM_SUPPORT
100: 
101: ncclRedOpRAII getNcclReduceOp(
102:     const ReduceOp& reduceOp,
103:     at::Tensor& input,
104:     const ncclDataType_t& dataType,
105:     const ncclComm_t& comm) {
106:   try {
107:     if (input.scalar_type() == at::kBool) {
108:       if (reduceOp == ReduceOp::SUM) {
109:         // For bool tensors, map sum to max, which both represent a bitwise or.
110:         // This is to prevent overflow issues with sum, since we use uint8 to
111:         // represent a bool (see ncclDataType mapping).
112:         return ncclMax;
113:       }
114: #ifdef NCCL_HAS_AVG
115:       if (reduceOp == ReduceOp::AVG) {
116:         C10_THROW_ERROR(
117:             TypeError, "Cannot use ReduceOp.AVG with boolean inputs");
118:       }
119: #endif // NCCL_HAS_AVG
120:     }
```

- EN: Lines 91-120 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `getNcclReduceOp`, `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 91-120 行使用条件编译来适配特性开关、平台或可选后端；在 `getNcclReduceOp`、`C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 121-150 / 第 121-150 行

```cpp
121:     if (reduceOp == ReduceOp::PREMUL_SUM) {
122: #ifdef ENABLE_NCCL_PREMUL_SUM_SUPPORT
123:       switch (dataType) {
124:         case ncclHalf:
125:           return unpackPreMulSum<at::Half, ncclHalf>(reduceOp, comm);
126:         case ncclFloat:
127:           return unpackPreMulSum<float, ncclFloat>(reduceOp, comm);
128:         case ncclBfloat16:
129:           return unpackPreMulSum<float, ncclBfloat16>(reduceOp, comm);
130:         case ncclDouble:
131:           return unpackPreMulSum<double, ncclDouble>(reduceOp, comm);
132:         default:
133:           C10_THROW_ERROR(
134:               TypeError,
135:               "PreMulSum Data type must be half, float, bfloat16 or double");
136:       }
137: #else
138:       C10_THROW_ERROR(ValueError, "PreMulSum requires NCCL>=2.11.1");
139: #endif // ENABLE_NCCL_PREMUL_SUM_SUPPORT
140:     }
141:     return ncclOp.at(reduceOp);
142:   } catch (const std::out_of_range&) {
143:     switch (reduceOp) {
144:       case ReduceOp::AVG:
145:         C10_THROW_ERROR(
146:             ValueError,
147:             c10::str(
148:                 "AVG requires NCCL 2.10+. The current version is ",
149:                 NCCL_MAJOR,
150:                 ".",
```

- EN: Lines 121-150 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 121-150 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 151-180 / 第 151-180 行

```cpp
151:                 NCCL_MINOR));
152:         break;
153:       case ReduceOp::BAND:
154:         C10_THROW_ERROR(ValueError, "Cannot use ReduceOp.BAND with NCCL");
155:         break;
156:       case ReduceOp::BOR:
157:         C10_THROW_ERROR(ValueError, "Cannot use ReduceOp.BOR with NCCL");
158:         break;
159:       case ReduceOp::BXOR:
160:         C10_THROW_ERROR(ValueError, "Cannot use ReduceOp.BXOR with NCCL");
161:         break;
162:       default:
163:         C10_THROW_ERROR(ValueError, "Unhandled ReduceOp");
164:         break;
165:     }
166:   }
167: }
168: 
169: // Get a key string from device
170: inline std::string getKeyFromDevice(const at::Device& device) {
171:   return std::to_string(device.index());
172: }
173: 
174: std::string getKeySendRecv(int myRank, int peer) {
175:   int lowRank = myRank < peer ? myRank : peer;
176:   int highRank = myRank < peer ? peer : myRank;
177:   std::string sendRecvPair =
178:       std::to_string(lowRank) + ":" + std::to_string(highRank);
179:   return sendRecvPair;
180: }
```

- EN: Lines 151-180 introduces executable logic in routines such as `C10_THROW_ERROR`, `getKeyFromDevice`, `getKeySendRecv`; performs validation and error handling to keep distributed state consistent.
- CN: 第 151-180 行在 `C10_THROW_ERROR`、`getKeyFromDevice`、`getKeySendRecv` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 181-210 / 第 181-210 行

```cpp
181: 
182: // Get device from tensor
183: inline at::Device getDevice(at::Tensor& tensor) {
184:   return tensor.device();
185: }
186: 
187: // [Sync Streams] Helper that lets the input ncclStreams to wait for the current
188: // stream. NCCL communications run on ncclStreams, but input tensors are
189: // allocated on different streams (i.e., current streams). Communications on
190: // ncclStreams cannot start before pending input tensor ops on current streams
191: // finish. Otherwise, ops on two streams might read/write same tensors
192: // concurrently.
193: //
194: // The synchronization above alone is not enough. We also need to make sure
195: // input tensors are not freed before their usages on ncclStreams finish. This
196: // can be achieved by calling c10::cuda::CUDACachingAllocator::recordStream,
197: // which remembers the usage stream (ncclStream), creates an event on the usage
198: // stream when GC attempts to free the input tensor, and delays GC until that
199: // event is done.
200: void syncStream(
201:     at::Device& device,
202:     at::cuda::CUDAEvent& ncclEvent,
203:     at::cuda::CUDAStream& ncclStream) {
204:   ncclEvent.record(at::cuda::getCurrentCUDAStream(device.index()));
205:   ncclEvent.block(ncclStream);
206: }
207: 
208: std::string getNcclAbortedCommStoreKey(const std::string& ncclIdStr) {
209:   return std::string(kNCCLAbortedCommStoreKey) + ":" + ncclIdStr;
210: }
```

- EN: Lines 181-210 introduces executable logic in routines such as `getDevice`, `syncStream`, `getNcclAbortedCommStoreKey`; returns computed state or forwards results to the surrounding caller.
- CN: 第 181-210 行在 `getDevice`、`syncStream`、`getNcclAbortedCommStoreKey` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 211-240 / 第 211-240 行

```cpp
211: 
212: // Returns exception's what() given an exception_ptr instance.
213: std::string getExceptionMsgFromExceptionPtr(
214:     const std::exception_ptr& exceptionPtr) {
215:   TORCH_CHECK(exceptionPtr != nullptr);
216:   try {
217:     std::rethrow_exception(exceptionPtr);
218:   } catch (const std::exception& e) {
219:     return e.what();
220:   } catch (...) {
221:     return "Unknown exception type";
222:   }
223: }
224: 
225: #ifdef USE_ROCM
226: // Indicates that we're in the watchdog's event-query phase. This allows ROCm
227: // workaround behavior to be applied only to watchdog-side queries, while
228: // preserving existing behavior for user/main-thread `WorkNCCL::isCompleted()`
229: // and `wait()` calls.
230: thread_local bool g_in_rocm_watchdog_event_query_context = false;
231: 
232: struct RocmWatchdogEventQueryContextGuard {
233:   RocmWatchdogEventQueryContextGuard()
234:       : previous_(g_in_rocm_watchdog_event_query_context) {
235:     g_in_rocm_watchdog_event_query_context = true;
236:   }
237:   ~RocmWatchdogEventQueryContextGuard() {
238:     g_in_rocm_watchdog_event_query_context = previous_;
239:   }
240: 
```

- EN: Lines 211-240 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `RocmWatchdogEventQueryContextGuard`; introduces executable logic in routines such as `getExceptionMsgFromExceptionPtr`, `TORCH_CHECK`, `~RocmWatchdogEventQueryContextGuard`.
- CN: 第 211-240 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `RocmWatchdogEventQueryContextGuard` 等类型；在 `getExceptionMsgFromExceptionPtr`、`TORCH_CHECK`、`~RocmWatchdogEventQueryContextGuard` 等例程中引入具体执行逻辑。

### Lines 241-270 / 第 241-270 行

```cpp
241:  private:
242:   bool previous_;
243: };
244: #endif // USE_ROCM
245: 
246: #ifdef USE_ROCM
247: // Watchdog-side cudaEventQuery workaround for HIP runtimes without the
248: // capture-mode fix.
249: // TODO: Remove once all supported runtimes include
250: // https://github.com/ROCm/rocm-systems/pull/3176
251: bool queryEventWithRocmWatchdogCaptureWorkaround(
252:     const std::shared_ptr<at::cuda::CUDAEvent>& event) {
253:   if (!event->isCreated()) {
254:     return true;
255:   }
256: 
257:   // Must unconditionally return false here during watchdog + active capture:
258:   // on affected HIP runtimes, even calling cudaEventQuery from the watchdog
259:   // thread while another thread has GLOBAL capture active can invalidate that
260:   // capture and cause downstream failures. Skip the query entirely and report
261:   // "not complete yet"; the watchdog will re-poll once capture ends. Timeout
262:   // enforcement is also deferred during this window (see the
263:   // is_graph_capture_active() gate in the watchdog loop).
264:   if (g_in_rocm_watchdog_event_query_context &&
265:       at::cuda::is_graph_capture_active()) {
266:     return false;
267:   }
268: 
269:   const cudaError_t err =
270:       C10_CUDA_ERROR_HANDLED(cudaEventQuery(event->event()));
```

- EN: Lines 241-270 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `queryEventWithRocmWatchdogCaptureWorkaround`.
- CN: 第 241-270 行使用条件编译来适配特性开关、平台或可选后端；在 `queryEventWithRocmWatchdogCaptureWorkaround` 等例程中引入具体执行逻辑。

### Lines 271-300 / 第 271-300 行

```cpp
271:   if (err == cudaSuccess) {
272:     return true;
273:   } else if (err != cudaErrorNotReady) {
274:     C10_CUDA_CHECK(err);
275:   } else {
276:     // ignore and clear the error if not ready
277:     (void)cudaGetLastError();
278:   }
279: 
280:   return false;
281: }
282: #endif // USE_ROCM
283: 
284: inline void errorIfCapturingNonCapturableNCCL(c10::cuda::CaptureStatus status) {
285:   // parentheses avoid some compiler warnings
286:   static const uint64_t min_version =
287:       (((uint64_t)2) << 32) + (((uint64_t)9) << 16) + ((uint64_t)6);
288:   static const uint64_t cur_version = torch::cuda::nccl::version();
289:   if (cur_version < min_version) {
290:     TORCH_CHECK_WITH(
291:         NotImplementedError,
292:         status == c10::cuda::CaptureStatus::None,
293:         "Capturing NCCL collectives is only allowed with NCCL >= 2.9.6");
294:   }
295: }
296: 
297: // When TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK is set, all tensors (no
298: // matter how they have been allocated) are registered with all NCCL comms.
299: bool shouldAllCommunicatorsRegisterAllTensors() {
300: #ifdef NCCL_HAS_COMM_REGISTER
```

- EN: Lines 271-300 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `errorIfCapturingNonCapturableNCCL`, `TORCH_CHECK_WITH`, `shouldAllCommunicatorsRegisterAllTensors`; performs validation and error handling to keep distributed state consistent.
- CN: 第 271-300 行使用条件编译来适配特性开关、平台或可选后端；在 `errorIfCapturingNonCapturableNCCL`、`TORCH_CHECK_WITH`、`shouldAllCommunicatorsRegisterAllTensors` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 301-330 / 第 301-330 行

```cpp
301:   static const bool flag = [] {
302:     const bool flag =
303:         getCvarBool(TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK, false);
304:     if (flag &&
305:         c10::cuda::CUDACachingAllocator::CUDAAllocatorConfig::
306:             expandable_segments()) {
307:       LOG(INFO)
308:           << "disables TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK because it is not compatible with CUDA allocator expandable segments mode.";
309:       return false;
310:     }
311:     return flag;
312:   }();
313:   return flag;
314: #else
315:   return false;
316: #endif // NCCL_HAS_COMM_REGISTER
317: }
318: 
319: } // namespace
320: 
321: // Map each communicator to the memory pools registered with it.
322: // This map is used when the caching allocator allocates or frees segments, in
323: // order to register or deregister them with the relevant NCCL communicators.
324: // There are two modes to do so:
325: // - If TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK=1 then *ALL* segments
326: //   will be registered with *ALL* NCCL communicators (for the same device),
327: //   even if they were allocated with cudaMalloc (which NCCL doesn't like).
328: // - If a MemPool is explicitly registered with a ProcessGroup, then all its
329: //   segments (current and future) will be registered with the NCCL communicator
330: //   corresponding to the pool's device. This works best if the MemPool is set
```

- EN: Lines 301-330 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 301-330 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 331-360 / 第 331-360 行

```cpp
331: //   up to use ncclMemAlloc (which is exposed by the ProcessGroup).
332: // Implementation notes:
333: // - We cannot reuse devNCCLCommMap_ in each ProcessGroup because the key may be
334: //   ranks rather than device in point-to-point case.
335: // - This map has also to be maintained as global variable since the register
336: //   hooks are called outside the scope of any PG, thus we need traverse
337: //   communicators in all PGs.
338: 
339: // MemPoolSet has ids of mempools used with this communicator, and whether they
340: // were registered with window APIs or not
341: using MemPoolSet = std::unordered_set<
342:     std::tuple<c10::cuda::MempoolId_t, bool>,
343:     c10::hash<std::tuple<c10::cuda::MempoolId_t, bool>>>;
344: static std::unordered_map<std::shared_ptr<NCCLComm>, MemPoolSet>
345:     ncclCommMemPoolMap;
346: static std::mutex ncclCommMemPoolMapMutex;
347: 
348: std::atomic<bool> ProcessGroupNCCL::shouldDump_(false);
349: 
350: static void cacheAllocatorRegisterHook(
351:     const c10::CachingDeviceAllocator::TraceEntry& te) {
352:   // Register after SEGMENT_ALLOC
353:   if (te.action_ !=
354:       c10::CachingDeviceAllocator::TraceEntry::Action::SEGMENT_ALLOC) {
355:     return;
356:   }
357: 
358:   std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
359:   for (auto& [ncclComm, memPools] : ncclCommMemPoolMap) {
360:     if (te.device_ == ncclComm->getDeviceIndex()) {
```

- EN: Lines 331-360 introduces executable logic in routines such as `cacheAllocatorRegisterHook`; returns computed state or forwards results to the surrounding caller.
- CN: 第 331-360 行在 `cacheAllocatorRegisterHook` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 361-390 / 第 361-390 行

```cpp
361:       bool symm = false;
362:       bool should_register = shouldAllCommunicatorsRegisterAllTensors();
363:       auto it =
364:           std::find_if(memPools.begin(), memPools.end(), [&](const auto& tup) {
365:             return std::get<0>(tup) == te.mempool_;
366:           });
367:       if (it != memPools.end()) {
368:         should_register = true;
369:         symm = std::get<1>(*it);
370:       }
371:       if (should_register) {
372:         // NOLINTNEXTLINE(performance-no-int-to-ptr)
373:         ncclComm->registerSegment(
374:             reinterpret_cast<void*>(te.addr_),
375:             te.size_,
376:             /*errorOnRereg*/ false,
377:             /*window*/ symm);
378:       }
379:     }
380:   }
381: }
382: 
383: static void cacheAllocatorDeregisterHook(
384:     const c10::CachingDeviceAllocator::TraceEntry& te) {
385:   // deregister before SEGMENT_FREE
386:   if (te.action_ !=
387:       c10::CachingDeviceAllocator::TraceEntry::Action::SEGMENT_FREE) {
388:     return;
389:   }
390: 
```

- EN: Lines 361-390 introduces executable logic in routines such as `cacheAllocatorDeregisterHook`; returns computed state or forwards results to the surrounding caller.
- CN: 第 361-390 行在 `cacheAllocatorDeregisterHook` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 391-420 / 第 391-420 行

```cpp
391:   std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
392:   for (auto& [ncclComm, memPools] : ncclCommMemPoolMap) {
393:     if (te.device_ == ncclComm->getDeviceIndex()) {
394:       bool symm = false;
395:       bool should_register = shouldAllCommunicatorsRegisterAllTensors();
396:       auto it =
397:           std::find_if(memPools.begin(), memPools.end(), [&](const auto& tup) {
398:             return std::get<0>(tup) == te.mempool_;
399:           });
400:       if (it != memPools.end()) {
401:         should_register = true;
402:         symm = std::get<1>(*it);
403:       }
404:       if (should_register) {
405:         // NOLINTNEXTLINE(performance-no-int-to-ptr)
406:         ncclComm->deregisterSegment(reinterpret_cast<void*>(te.addr_), symm);
407:       }
408:     }
409:   }
410: }
411: 
412: static void attachAllocatorHooks() {
413:   static auto flag [[maybe_unused]] = [] {
414:     // Attaching hooks fails if CUDACachingAllocator is not initialized, so
415:     // Init for CUDA is called (and is a no-op if CUDA is already
416:     // initialized).
417:     at::globalContext().lazyInitDevice(c10::DeviceType::CUDA);
418:     c10::cuda::CUDACachingAllocator::attachAllocatorTraceTracker(
419:         &cacheAllocatorRegisterHook);
420:     c10::cuda::CUDACachingAllocator::attachAllocatorTraceTracker(
```

- EN: Lines 391-420 introduces executable logic in routines such as `attachAllocatorHooks`; returns computed state or forwards results to the surrounding caller.
- CN: 第 391-420 行在 `attachAllocatorHooks` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 421-450 / 第 421-450 行

```cpp
421:         &cacheAllocatorDeregisterHook);
422:     return true;
423:   }();
424: }
425: 
426: static std::
427:     unordered_map<std::string, std::unordered_map<std::string, std::string>>
428:     getNCCLCommDumpMap() {
429: #if (defined(IS_NCCLX) || defined(USE_ROCM)) && defined(NCCL_COMM_DUMP)
430:   std::unordered_map<
431:       std::string /* ncclUniqueID */,
432:       std::unordered_map<std::string, std::string> /* dump from this comm */>
433:       ncclDumpMap;
434:   // dump_nccl_trace is only called from the default PG (local_id_=0), but we
435:   // want to dump from all comms so we need to iterate over ncclCommMemPoolMap,
436:   // which is static
437:   std::vector<std::shared_ptr<NCCLComm>> allNCCLComms;
438:   // within the critical section, we don't want to dump while holding the lock
439:   // as dump might hang
440:   {
441:     std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
442:     for (auto& [ncclComm, _] : ncclCommMemPoolMap) {
443:       allNCCLComms.push_back(ncclComm);
444:     }
445:   }
446:   for (auto& ncclComm : allNCCLComms) {
447:     ncclDumpMap[ncclComm->getUniqueHash()] = ncclComm->ncclCommDump();
448:   }
449:   return ncclDumpMap;
450: #else
```

- EN: Lines 421-450 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `getNCCLCommDumpMap`.
- CN: 第 421-450 行使用条件编译来适配特性开关、平台或可选后端；在 `getNCCLCommDumpMap` 等例程中引入具体执行逻辑。

### Lines 451-480 / 第 451-480 行

```cpp
451:   return std::unordered_map<
452:       std::string,
453:       std::unordered_map<std::string, std::string>>();
454: #endif // (defined(IS_NCCLX) || defined(USE_ROCM)) && defined(NCCL_COMM_DUMP)
455: }
456: 
457: void reset_nccl_trace() {
458:   FlightRecorderCUDA::get()->reset_all();
459: }
460: 
461: std::string dump_nccl_trace(
462:     bool includeCollectives,
463:     bool includeStackTraces,
464:     bool onlyActive) {
465:   auto ncclDumpMap = getNCCLCommDumpMap();
466: #if defined(USE_ROCM) && defined(NCCL_COMM_DUMP)
467:   for (const auto& [ncclUniqueIDStr, dump] : ncclDumpMap) {
468:     printNcclCommProxyTrace("Received dump signal " + ncclUniqueIDStr, dump);
469:   }
470: #endif // defined(USE_ROCM) && defined(NCCL_COMM_DUMP)
471:   return FlightRecorderCUDA::get()->dump(
472:       ncclDumpMap, includeCollectives, includeStackTraces, onlyActive);
473: }
474: 
475: std::string dump_nccl_trace_json(bool includeCollectives, bool onlyActive) {
476:   auto ncclDumpMap = getNCCLCommDumpMap();
477:   return FlightRecorderCUDA::get()->dump_json(
478:       ncclDumpMap, includeCollectives, onlyActive);
479: }
480: 
```

- EN: Lines 451-480 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `reset_nccl_trace`, `dump_nccl_trace`, `printNcclCommProxyTrace`.
- CN: 第 451-480 行使用条件编译来适配特性开关、平台或可选后端；在 `reset_nccl_trace`、`dump_nccl_trace`、`printNcclCommProxyTrace` 等例程中引入具体执行逻辑。

### Lines 481-510 / 第 481-510 行

```cpp
481: std::optional<std::function<void(std::function<void(const std::string&)>)>>&
482: get_cpp_trace_dumper() {
483:   static std::optional<
484:       std::function<void(std::function<void(const std::string&)>)>>
485:       dumper(std::nullopt);
486:   return dumper;
487: }
488: 
489: gil_checker_t& get_gil_checker() {
490:   static gil_checker_t gil_checker = nullptr;
491:   return gil_checker;
492: }
493: 
494: static std::future<bool> launchAsyncGilCheck() {
495:   std::promise<bool> resultPromise;
496:   std::future<bool> resultFuture = resultPromise.get_future();
497:   TORCH_CHECK(get_gil_checker(), "Can't check GIL with null GIL checker");
498:   std::thread workerThread([promise = std::move(resultPromise)]() mutable {
499:     c10::setThreadName("pt_nccl_gil_chk");
500: 
501:     try {
502:       auto& gil_checker = get_gil_checker();
503:       promise.set_value((*gil_checker)());
504:     } catch (...) {
505:       promise.set_exception(std::current_exception());
506:     }
507:   });
508: 
509:   // Detach the thread to allow it to run independently
510:   workerThread.detach();
```

- EN: Lines 481-510 introduces executable logic in routines such as `get_cpp_trace_dumper`, `dumper`, `get_gil_checker`; performs validation and error handling to keep distributed state consistent.
- CN: 第 481-510 行在 `get_cpp_trace_dumper`、`dumper`、`get_gil_checker` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 511-540 / 第 511-540 行

```cpp
511: 
512:   return resultFuture;
513: }
514: 
515: const int64_t ProcessGroupNCCL::kWatchdogThreadSleepMillis = 100;
516: constexpr int64_t kSynchronizeBusyWaitMillis = 1;
517: thread_local uint64_t ProcessGroupNCCL::ncclActiveGroupCounter_ = 0;
518: 
519: std::ostream& operator<<(
520:     std::ostream& output,
521:     const ProcessGroupNCCL::WorkNCCL& workNCCL) {
522:   std::string workInfo;
523:   workInfo = c10::str(
524:       "WorkNCCL(",
525:       "SeqNum=",
526:       workNCCL.seq_,
527:       ", OpType=",
528:       opTypeToString(workNCCL.opType_),
529:       ", NumelIn=",
530:       workNCCL.numelIn_,
531:       ", NumelOut=",
532:       workNCCL.numelOut_,
533:       ", Timeout(ms)=",
534:       workNCCL.opTimeout_.count(),
535:       ")");
536:   return output << workInfo;
537: }
538: 
539: /* Implementation of TensorShelf class */
540: 
```

- EN: Lines 511-540 introduces executable logic in routines such as `operator<<`; returns computed state or forwards results to the surrounding caller.
- CN: 第 511-540 行在 `operator<<` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 541-570 / 第 541-570 行

```cpp
541: void TensorShelf::stash(std::vector<at::Tensor>& tensors) {
542:   std::lock_guard<std::mutex> lock(mutex_);
543:   tVector_.insert(tVector_.end(), tensors.begin(), tensors.end());
544: }
545: 
546: void TensorShelf::stash(TensorShelf& other) {
547:   std::vector<at::Tensor>& otherVec = other.get();
548:   this->stash(otherVec);
549: }
550: 
551: void TensorShelf::unstash() {
552:   this->clear();
553: }
554: 
555: bool TensorShelf::empty() {
556:   std::lock_guard<std::mutex> lock(mutex_);
557:   return tVector_.empty();
558: }
559: 
560: void TensorShelf::clear() {
561:   std::lock_guard<std::mutex> lock(mutex_);
562:   tVector_.clear();
563: }
564: 
565: std::vector<at::Tensor>& TensorShelf::get() {
566:   return tVector_;
567: }
568: 
569: ProcessGroupNCCL::WorkNCCL::WorkNCCL(
570:     std::string pgUID,
```

- EN: Lines 541-570 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 541-570 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 571-600 / 第 571-600 行

```cpp
571:     std::string pgDesc,
572:     at::Device& device,
573:     int rank,
574:     OpType opType,
575:     uint64_t seq,
576:     bool isP2P,
577:     const char* profilingTitle,
578:     const std::optional<std::vector<at::Tensor>>& inputs,
579:     bool enableTiming,
580:     bool cudaEventCacheEnabled,
581:     DebugLevel distDebugLevel)
582:     : Work(rank, opType, profilingTitle, inputs),
583:       pgUID_(std::move(pgUID)),
584:       pgDesc_(std::move(pgDesc)),
585:       device_(device),
586:       workStartTime_(std::chrono::steady_clock::now()),
587:       seq_(seq),
588:       isP2P_(isP2P),
589:       timingEnabled_(enableTiming),
590:       distDebugLevel_(distDebugLevel) {
591:   // Creates the CUDA event wrappers
592:   // Note: The actual events are lazily created when first recorded to with
593:   // DEFAULT_FLAGS = cudaEventDisableTiming.
594:   if (cudaEventCacheEnabled) {
595:     ncclStartEvent_ = enableTiming
596:         ? CUDAEventCache::get(device.index())->create(enableTiming)
597:         : nullptr;
598:     ncclEndEvent_ = CUDAEventCache::get(device.index())->create(enableTiming);
599:   } else {
600:     ncclStartEvent_ = enableTiming
```

- EN: Lines 571-600 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 571-600 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 601-630 / 第 601-630 行

```cpp
601:         ? std::make_shared<at::cuda::CUDAEvent>(cudaEventDefault)
602:         : nullptr;
603:     ncclEndEvent_ = std::make_shared<at::cuda::CUDAEvent>(
604:         enableTiming ? cudaEventDefault : cudaEventDisableTiming);
605:   }
606:   futureWorkResult_ =
607:       c10::make_intrusive<at::ivalue::Future>(c10::AnyEnumType::get());
608:   // other functions expect an initialized ptr
609:   stashed_for_allocator_safety_ = std::make_shared<TensorShelf>();
610: }
611: 
612: ProcessGroupNCCL::WorkNCCL::WorkNCCL(const WorkNCCL& w)
613:     : Work(w.rank_, w.opType_),
614:       std::enable_shared_from_this<WorkNCCL>(w),
615:       pgUID_(w.pgUID_),
616:       pgDesc_(w.pgDesc_),
617:       device_(w.device_),
618:       ncclStartEvent_(w.ncclStartEvent_),
619:       ncclEndEvent_(w.ncclEndEvent_),
620:       ncclComm_(w.ncclComm_),
621:       blockingWait_(w.blockingWait_),
622:       opTimeout_(w.opTimeout_),
623:       ownedEphermeralTimeout_(w.ownedEphermeralTimeout_),
624:       workStartTime_(w.workStartTime_),
625:       seq_(w.seq_),
626:       isP2P_(w.isP2P_),
627:       startTraceUpdated_(w.startTraceUpdated_),
628:       numelIn_(w.numelIn_),
629:       numelOut_(w.numelOut_),
630:       store_(w.store_),
```

- EN: Lines 601-630 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 601-630 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 631-660 / 第 631-660 行

```cpp
631:       // Note: the `work` returned to user and the `work` enqueued to watchdog
632:       // share the pointer to the tensor stash.  At least one of them should
633:       // clean the tensor stash, the earlier the better, i.e. user calling
634:       // `work.wait` than watchdog detecting work completion.
635:       stashed_for_allocator_safety_(w.stashed_for_allocator_safety_),
636:       futureWorkResult_(w.futureWorkResult_),
637:       timingEnabled_(w.timingEnabled_),
638:       trace_id_(w.trace_id_),
639:       trace_reset_epoch_(w.trace_reset_epoch_),
640:       distDebugLevel_(w.distDebugLevel_) {
641:   exception_ = w.exception_;
642: }
643: 
644: bool ProcessGroupNCCL::WorkNCCL::isCompleted() {
645:   if (!ncclComm_->isAborted()) {
646:     checkAndSetException();
647:   }
648:   return exception() || finishedGPUExecutionInternal();
649: }
650: 
651: bool ProcessGroupNCCL::WorkNCCL::isStarted() {
652:   if (!ncclComm_->isAborted()) {
653:     checkAndSetException();
654:   }
655:   return exception() || startedGPUExecutionInternal();
656: }
657: 
658: bool ProcessGroupNCCL::WorkNCCL::isSuccess() const {
659:   C10_THROW_ERROR(NotImplementedError, "WorkNCCL::isSuccess() is deprecated");
660: }
```

- EN: Lines 631-660 introduces executable logic in routines such as `checkAndSetException`; performs validation and error handling to keep distributed state consistent.
- CN: 第 631-660 行在 `checkAndSetException` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 661-690 / 第 661-690 行

```cpp
661: 
662: void ProcessGroupNCCL::WorkNCCL::checkAndSetException() {
663:   if (exception()) {
664:     // We already have an exception.
665:     return;
666:   }
667: 
668:   auto exception_ptr = checkForNCCLErrors();
669:   std::unique_lock<std::mutex> lock(mutex_);
670:   exception_ = exception_ptr;
671:   if (exception_) {
672:     LOG(ERROR) << logPrefix() << "Collective " << *this
673:                << " raised the following async exception: "
674:                << getExceptionMsgFromExceptionPtr(exception_);
675: 
676:     // Mark future result as ERROR
677:     if (futureWorkResult_ && !futureWorkResult_->completed()) {
678:       futureWorkResult_->markCompleted(
679:           at::IValue(static_cast<uint8_t>(WorkResult::COMM_ERROR)));
680:     }
681:   }
682: }
683: 
684: const std::string& ProcessGroupNCCL::WorkNCCL::logPrefix() const {
685:   static std::string prefix = c10::str("[Rank ", rank_, "] ");
686:   return prefix;
687: }
688: 
689: void ProcessGroupNCCL::WorkNCCL::setException(
690:     std::exception_ptr exception_ptr) {
```

- EN: Lines 661-690 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 661-690 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 691-720 / 第 691-720 行

```cpp
691:   std::unique_lock<std::mutex> lock(mutex_);
692:   exception_ = std::move(exception_ptr);
693: }
694: 
695: // Helper that checks if the NCCL kernels are completed on the GPUs
696: bool ProcessGroupNCCL::WorkNCCL::finishedGPUExecution() {
697:   checkAndSetException();
698:   return finishedGPUExecutionInternal();
699: }
700: 
701: bool ProcessGroupNCCL::WorkNCCL::startedGPUExecutionInternal() const {
702:   // if timing is disabled we won't have allocated start events
703:   if (!timingEnabled_) {
704:     return false;
705:   }
706:   // Checking the work's corresponding CUDA event's status
707: #ifdef USE_ROCM
708:   if (!queryEventWithRocmWatchdogCaptureWorkaround(ncclStartEvent_)) {
709: #else
710:   if (!ncclStartEvent_->query()) {
711: #endif
712:     return false;
713:   }
714:   return true;
715: }
716: 
717: bool ProcessGroupNCCL::WorkNCCL::finishedGPUExecutionInternal() const {
718:   // Checking the work's corresponding CUDA event's status
719:   // It calls `cudaEventQuery` eventually. Although this seems to be a
720:   // non-blocking call, but we did notice hangs in the past. It can
```

- EN: Lines 691-720 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `checkAndSetException`, `finishedGPUExecutionInternal`.
- CN: 第 691-720 行使用条件编译来适配特性开关、平台或可选后端；在 `checkAndSetException`、`finishedGPUExecutionInternal` 等例程中引入具体执行逻辑。

### Lines 721-750 / 第 721-750 行

```cpp
721:   // hang if another thread is holding the CUDA global context lock. For
722:   // example, when doing a `cudaDeviceSynchronize` or even
723:   // `cudaStreamSynchronize`.
724: #ifdef USE_ROCM
725:   if (!queryEventWithRocmWatchdogCaptureWorkaround(ncclEndEvent_)) {
726: #else
727:   if (!ncclEndEvent_->query()) {
728: #endif
729:     return false;
730:   }
731:   return true;
732: }
733: 
734: bool ProcessGroupNCCL::WorkNCCL::checkTimeout(
735:     std::optional<std::chrono::milliseconds> timeout) {
736:   STATIC_SCOPED_WAIT_COUNTER(
737:       pytorch.wait_counter.ProcessGroupNCCL__checkTimeout);
738:   auto currentTimepoint = std::chrono::steady_clock::now();
739:   auto timeElapsed = std::chrono::duration_cast<std::chrono::milliseconds>(
740:       currentTimepoint - workStartTime_);
741:   auto workTimeout = timeout ? *timeout : opTimeout_;
742: 
743:   if (timeElapsed < workTimeout) {
744:     return false;
745:   }
746: 
747:   // Timed out
748: 
749:   std::string exceptionMsg = c10::str(
750:       logPrefix(),
```

- EN: Lines 721-750 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 721-750 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 751-780 / 第 751-780 行

```cpp
751:       "Watchdog caught collective operation timeout: ",
752:       *this,
753:       " ran for ",
754:       timeElapsed.count(),
755:       " milliseconds before timing out.");
756: 
757:   LOG(ERROR) << exceptionMsg;
758: 
759:   std::exception_ptr exception_ptr =
760:       std::make_exception_ptr(C10_BUILD_ERROR(DistBackendError, exceptionMsg));
761:   if (!exception()) {
762:     // if there is already an error, we don't override it
763:     setException(exception_ptr);
764:   }
765: 
766:   // Mark future result as TIMEOUT
767:   if (futureWorkResult_ && !futureWorkResult_->completed()) {
768:     futureWorkResult_->markCompleted(
769:         at::IValue(static_cast<uint8_t>(WorkResult::TIMEOUT)));
770:   }
771:   return true;
772: }
773: 
774: // Print the traceback of the collective at call time
775: std::string ProcessGroupNCCL::WorkNCCL::getTraceback() const {
776:   // First step we get the corresponding record entry from FR, based on work's
777:   // trace_id_ and trace_reset_epoch_
778:   std::optional<FlightRecorderCUDA::Entry> entry =
779:       FlightRecorderCUDA::get()->getEntry(trace_id_, trace_reset_epoch_);
780:   if (entry.has_value()) {
```

- EN: Lines 751-780 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 751-780 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 781-810 / 第 781-810 行

```cpp
781:     auto entryVal = entry.value();
782:     // Get stack trace from FR entry, in string format
783:     // Note: `getTraceback` call below invokes `torch::symbolize`, which may
784:     // need to acquire the GIL. In order for watchdog to be block-free, we make
785:     // the call with std::async.
786:     auto future = std::async(
787:         std::launch::async, [&entryVal]() { return entryVal.getTraceback(); });
788:     // Wait for the future to complete or timeout
789:     auto status = future.wait_for(std::chrono::seconds(8));
790:     if (status == std::future_status::ready) {
791:       return future.get();
792:     }
793:   }
794:   return "";
795: }
796: 
797: // Print the traceback of the collective at call time
798: void ProcessGroupNCCL::WorkNCCL::printTraceback() const {
799:   std::string tracebackStr = getTraceback();
800:   if (!tracebackStr.empty()) {
801:     LOG(ERROR) << "Stack trace of the failed collective: \n" << tracebackStr;
802:   } // else, symbolizer probably timed out, we skip logging the stack trace.
803:   else {
804:     LOG(ERROR)
805:         << "Stack trace of the failed collective not found, "
806:         << "potentially because FlightRecorder is disabled. "
807:         << "You can enable it by setting TORCH_NCCL_TRACE_BUFFER_SIZE to a non-zero value.";
808:   }
809: }
810: 
```

- EN: Lines 781-810 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 781-810 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 811-840 / 第 811-840 行

```cpp
811: void ProcessGroupNCCL::WorkNCCL::handleException(
812:     ErrorHandlingMode errorHandling) {
813:   if (exception_) {
814:     auto exceptionMsg = c10::str(
815:         "Some NCCL operations have failed or timed out. Due to the ",
816:         "asynchronous nature of CUDA kernels, subsequent GPU operations ",
817:         "might run on corrupted/incomplete data.");
818:     LOG(ERROR) << logPrefix() << exceptionMsg;
819:     C10_LOG_API_USAGE_ONCE("ProcessGroupNCCL.WorkNCCL.handleException");
820: 
821:     auto logger = c10d::C10dLogger::getLogger();
822:     if (logger) {
823:       ::c10d::C10dLoggingData data;
824:       data.strings["work_nccl_exception"] =
825:           getExceptionMsgFromExceptionPtr(exception_);
826:       logger->log(data);
827:     }
828: 
829:     if (SHOULD_TEAR_DOWN(errorHandling)) {
830:       auto tearDownMsg = c10::str(
831:           "To avoid data inconsistency, we are taking the entire process down.");
832:       LOG(ERROR) << logPrefix() << tearDownMsg;
833:       std::rethrow_exception(exception_);
834:     }
835:   }
836: }
837: 
838: void ProcessGroupNCCL::WorkNCCL::synchronize() {
839:   synchronizeStream();
840:   if (c10d::allow_inflight_collective_as_graph_input()) {
```

- EN: Lines 811-840 introduces executable logic in routines such as `synchronizeStream`.
- CN: 第 811-840 行在 `synchronizeStream` 等例程中引入具体执行逻辑。

### Lines 841-870 / 第 841-870 行

```cpp
841:     c10d::unregister_work(
842:         c10::intrusive_ptr<
843:             ProcessGroupNCCL::WorkNCCL>::unsafe_reclaim_from_nonowning(this));
844:   }
845: }
846: 
847: void ProcessGroupNCCL::WorkNCCL::synchronizeStream() {
848:   auto currentStream = at::cuda::getCurrentCUDAStream(device_.index());
849:   // Block the current stream on the NCCL stream
850:   ncclEndEvent_->block(currentStream);
851:   // Unstage the stashed tensors so that CachingAllocator can recycle them
852:   // THIS MUST HAPPEN AFTER THE BLOCKING CALL ABOVE
853:   stashed_for_allocator_safety_->unstash();
854: }
855: 
856: // Same as calling synchronize() when blockingWait_ is false
857: bool ProcessGroupNCCL::WorkNCCL::wait(std::chrono::milliseconds timeout) {
858:   RECORD_PARAM_COMMS(
859:       std::make_tuple(static_cast<int64_t>(this->seq_), this->isP2P_), // seq
860:       std::make_tuple(pgUID_, pgDesc_), // PG name tuple
861:       rank_, // rank
862:       "wait", // collective name
863:       0, // inNelems
864:       0, // outNelems
865:       at::kByte, // dType
866:       std::vector<int64_t>(), // inSplitSizes
867:       std::vector<int64_t>(), // outSplitSizes
868:       -1,
869:       -1,
870:       static_cast<int>(1)); // number of device?
```

- EN: Lines 841-870 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 841-870 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 871-900 / 第 871-900 行

```cpp
871: 
872:   // synchronize() will block the current stream on the NCCL stream
873:   synchronize();
874: 
875:   // In case of blockingWait or a timeout value is specified by the user, we
876:   // block the CPU thread until the work is completed or timed out.
877:   if (blockingWait_ || timeout != kNoTimeout) {
878:     while (!isCompleted()) {
879:       bool timedOut = checkTimeout(
880:           timeout == kNoTimeout ? std::nullopt : std::make_optional(timeout));
881:       // Explicitly abort ncclComms here before throwing this timed out
882:       // exception to users.
883:       // If throwing timed out excepiton without aborting nccl communicators
884:       // here, it was observed that CUDA GPU will have 100% utilization and
885:       // can not run new events successfully.
886:       if (timedOut) {
887:         std::string exceptionMsg = c10::str(
888:             logPrefix(), "Work ", (*this), " timed out in blocking wait.");
889:         LOG(ERROR) << exceptionMsg;
890:         break;
891:       }
892:       // Yield
893:       std::this_thread::sleep_for(
894:           std::chrono::milliseconds(kSynchronizeBusyWaitMillis));
895:     }
896:   } else if (isBarrierOp_ && !isCompleted()) {
897:     // For barrier wait when timeout is unspecified, we block the CPU thread on
898:     // current stream. This is to minimize the CPU barrier wait time in healthy
899:     // path
900:     auto currentStream = at::cuda::getCurrentCUDAStream(device_.index());
```

- EN: Lines 871-900 introduces executable logic in routines such as `synchronize`.
- CN: 第 871-900 行在 `synchronize` 等例程中引入具体执行逻辑。

### Lines 901-930 / 第 901-930 行

```cpp
901:     // CUDAStream wrapper will correctly use a DeviceGuard here
902:     currentStream.synchronize();
903:   }
904: 
905:   // If exception is detected, throw it from the main CPU thread
906:   if (exception()) {
907:     // Abort NCCL communicators
908:     abort();
909:     // Throw exception (from main thread here)
910:     handleException(TearDown);
911:   }
912: 
913:   // TODO(kwen2501): this should be moved to c10d tests, to qualify a NCCL
914:   // upgrade. Once a NCCL version is qualified, this code should not be needed
915:   // at runtime.
916: #ifdef PGNCCL_ENABLE_HASH
917:   if (enableCollectiveHashDebug_.load()) {
918:     auto numel = getTensorsNumel(*outputs_);
919:     auto hashValue = hashTensors(*outputs_);
920:     PRINT_COLLECTIVE_HASH_SIGNATURE(
921:         "output", opTypeToString(opType_), numel, hashValue);
922:   }
923: #endif // PGNCCL_ENABLE_HASH
924:   // Always return true, because abort API is not implemented.
925:   return true;
926: }
927: 
928: void ProcessGroupNCCL::WorkNCCL::abort() {
929:   // dump before aborting for rcclexp
930: #if defined(USE_ROCM) && defined(NCCL_COMM_DUMP)
```

- EN: Lines 901-930 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `abort`; performs validation and error handling to keep distributed state consistent.
- CN: 第 901-930 行使用条件编译来适配特性开关、平台或可选后端；在 `abort` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 931-960 / 第 931-960 行

```cpp
931:   auto dumpMap = ncclComm_->ncclCommDump();
932:   printNcclCommProxyTrace("WorkNCCL::abort", dumpMap);
933: #endif // USE_ROCM && NCCL_COMM_DUMP
934: 
935:   // Abort all communicators of this work
936:   ncclComm_->abort();
937: 
938:   {
939:     std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
940:     ncclCommMemPoolMap.erase(ncclComm_);
941:   }
942: }
943: 
944: static std::atomic<size_t> process_group_id = 0;
945: 
946: constexpr const char* MULTI_DEVICE_ERROR_MSG =
947:     "Expecting one tensor only but got multiple. You are probably using multiple "
948:     "devices under one thread. The support for such usage has been deprecated. "
949:     "For details, please refer to "
950:     "https://pytorch.org/docs/stable/distributed.html#multi-gpu-collective-functions. "
951:     "ProcessGroupNCCL continues supporting multi-process and multi-thread modes.";
952: 
953: ProcessGroupNCCL::ProcessGroupNCCL(
954:     c10::intrusive_ptr<Store> store,
955:     int rank,
956:     int size,
957:     c10::intrusive_ptr<Options> options)
958:     : Backend(rank, size),
959:       store_(std::move(store)),
960:       options_(std::move(options)),
```

- EN: Lines 931-960 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `printNcclCommProxyTrace`.
- CN: 第 931-960 行使用条件编译来适配特性开关、平台或可选后端；在 `printNcclCommProxyTrace` 等例程中引入具体执行逻辑。

### Lines 961-990 / 第 961-990 行

```cpp
961:       terminateProcessGroup_(false),
962:       local_id_(process_group_id++),
963:       intraNodeComm_(nullptr) {
964:   TORCH_CHECK_WITH(
965:       ValueError,
966:       at::cuda::getNumGPUs() != 0,
967:       "ProcessGroupNCCL is only supported with GPUs, no GPUs found!");
968: 
969:   // getNcclVersion needs to get called before launching threads which can
970:   // potentially call getenv. getNcclVersion internally calls setenv to set some
971:   // environment variables from config file, which can race with getenv from
972:   // other threads and cause segfaults.
973:   const auto ncclVersion = getNcclVersion();
974:   this->setGroupUid(options_->group_name);
975:   this->localDeviceCount_ = static_cast<int>(at::cuda::getNumGPUs());
976:   logPrefix_ = createLogPrefix();
977:   blockingWait_ = getCvarBool(TORCH_NCCL_BLOCKING_WAIT, false);
978:   asyncErrorHandling_ = static_cast<ErrorHandlingMode>(
979:       getCvarInt(TORCH_NCCL_ASYNC_ERROR_HANDLING, 3 /*SkipCleanUp*/));
980:   enableNanCheck_ = getCvarBool(TORCH_NCCL_NAN_CHECK, false);
981:   cudaEventCacheEnabled_.store(getCvarBool(TORCH_NCCL_CUDA_EVENT_CACHE, true));
982:   // NOTE: This default value (2000) is duplicated in FlightRecorder.hpp.
983:   // Keep in sync. See FlightRecorder.hpp for details.
984:   traceBufferSize_ = getCvarInt(TORCH_NCCL_TRACE_BUFFER_SIZE, 2000);
985:   enableCollectiveHashDebug_ = (dist_debug_level_ >= DebugLevel::Detail);
986:   // store_ usually is wrapped with PrefixStore and the prefix is different
987:   // across different ProcessGroupNCCL(PG) instances. We need to get the
988:   // underlying non-PrefixStore for sharing global information shared across
989:   // different PGs.
990:   PrefixStore* prefixStore = dynamic_cast<PrefixStore*>(store_.get());
```

- EN: Lines 961-990 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 961-990 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 991-1020 / 第 991-1020 行

```cpp
991:   globalStore_ =
992:       prefixStore ? prefixStore->getUnderlyingNonPrefixStore() : store_;
993:   debugInfoPipeFile_ = getCvarString({"TORCH_NCCL_DEBUG_INFO_PIPE_FILE"}, "");
994:   auto desyncDebug = getCvarBool(TORCH_NCCL_DESYNC_DEBUG, false) ||
995:       (dist_debug_level_ >= DebugLevel::Detail);
996: #ifdef ENABLE_NCCL_ERROR_CHECKING
997:   enableTiming_.store(
998:       getCvarBool(TORCH_NCCL_ENABLE_TIMING, false) || desyncDebug);
999: #endif // ENABLE_NCCL_ERROR_CHECKING
1000:   if (getCvarBool(TORCH_NCCL_AVOID_RECORD_STREAMS, false)) {
1001:     TORCH_WARN_ONCE(
1002:         "TORCH_NCCL_AVOID_RECORD_STREAMS is the default now, this environment variable is thus deprecated.");
1003:   }
1004:   showSerializationWarning_ =
1005:       getCvarBool(TORCH_NCCL_SHOW_EAGER_INIT_P2P_SERIALIZATION_WARNING, true);
1006: 
1007:   if (blockingWait_) {
1008:     LOG(INFO)
1009:         << logPrefix()
1010:         << "TORCH_NCCL_BLOCKING_WAIT is enabled, NO watchdog thread is created.";
1011:   } else {
1012:     if (desyncDebug && asyncErrorHandling_ == NoHandling) {
1013:       LOG(INFO)
1014:           << logPrefix()
1015:           << "TORCH_NCCL_DESYNC_DEBUG and TORCH_NCCL_ASYNC_ERROR_HANDLING "
1016:           << "must both be enabled. "
1017:           << "Enabling TORCH_NCCL_ASYNC_ERROR_HANDLING.";
1018:       asyncErrorHandling_ = SkipCleanUp;
1019:     }
1020:   }
```

- EN: Lines 991-1020 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_WARN_ONCE`.
- CN: 第 991-1020 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_WARN_ONCE` 等例程中引入具体执行逻辑。

### Lines 1021-1050 / 第 1021-1050 行

```cpp
1021: 
1022:   // If deterministic mode is enabled, we need to disable the NVLS algorithm in
1023:   // NCCL.
1024:   // TODO: remove this once NVLS supports deterministic mode.
1025:   if (at::globalContext().deterministicAlgorithms()) {
1026:     // Check if user have already set NCCL_ALGO. If already set, leave it.
1027:     auto nccl_algo = c10::utils::get_env("NCCL_ALGO");
1028:     if (!nccl_algo.has_value()) {
1029:       LOG(INFO)
1030:           << "torch deterministic mode is enabled, "
1031:           << "disabling NVLS algorithm in NCCL which can lead to non-deterministic reduction.";
1032:       // Sorry we have to disable NVLS for all collectives, be it all-reduce
1033:       // or all-gather, because NCCL does not support per-collective
1034:       // algorithm selection today.
1035:       c10::utils::set_env("NCCL_ALGO", "^NVLS");
1036:     }
1037:   }
1038: 
1039:   // Initialize the heartbeat monitor/watchdog instance. This has to be done
1040:   // before the corresponding thread is launched to avoid the error.
1041:   heartbeatMonitor_ = std::make_unique<HeartbeatMonitor>(this);
1042:   watchdog_ = std::make_unique<Watchdog>(this);
1043: 
1044: #ifdef ENABLE_NCCL_ERROR_CHECKING
1045:   // in blockingWait mode, we don't need to enable the watchdog thread to check
1046:   // the timeout or nccl error because the main thread would throw an exception
1047:   // and it is the user's responsibility to handle the exception.
1048:   if (!blockingWait_) {
1049:     watchdog_->start();
1050:   }
```

- EN: Lines 1021-1050 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1021-1050 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1051-1080 / 第 1051-1080 行

```cpp
1051: #endif // ENABLE_NCCL_ERROR_CHECKING
1052: 
1053:   init();
1054:   const std::string OFF = "OFF";
1055:   std::string torch_distributed_debug =
1056:       getCvarString({"TORCH_DISTRIBUTED_DEBUG"}, OFF.c_str());
1057:   LOG(INFO) << logPrefix()
1058:             << "ProcessGroupNCCL initialization options: " << "size: " << size
1059:             << ", global rank: " << globalRank()
1060:             << ", TIMEOUT(ms): " << options_->timeout.count()
1061:             << ", USE_HIGH_PRIORITY_STREAM: "
1062:             << options_->is_high_priority_stream
1063:             << ", SPLIT_FROM: " << options_->split_from
1064:             << ", SPLIT_COLOR: " << options_->split_color
1065:             << ", PG Name: " << options_->group_name;
1066: 
1067:   if (local_id_ == 0) {
1068:     LOG(INFO) << logPrefix() << "ProcessGroupNCCL environments: "
1069:               << "NCCL version: " << ncclVersion
1070:               << ", TORCH_NCCL_ASYNC_ERROR_HANDLING: " << asyncErrorHandling_
1071:               << ", TORCH_NCCL_ENABLE_TIMING: " << enableTiming_.load()
1072:               << ", TORCH_NCCL_BLOCKING_WAIT: " << blockingWait_
1073:               << ", TORCH_DISTRIBUTED_DEBUG: " << torch_distributed_debug
1074: #ifdef NCCL_HAS_COMM_REGISTER
1075:               << ", TORCH_NCCL_USE_TENSOR_REGISTER_ALLOCATOR_HOOK: "
1076:               << shouldAllCommunicatorsRegisterAllTensors()
1077: #endif // NCCL_HAS_COMM_REGISTER
1078:               << ", TORCH_NCCL_TRACE_BUFFER_SIZE: " << traceBufferSize_
1079:               << ", TORCH_NCCL_NAN_CHECK: " << enableNanCheck_
1080:               << ", TORCH_NCCL_CUDA_EVENT_CACHE: " << cudaEventCacheEnabled_;
```

- EN: Lines 1051-1080 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `init`.
- CN: 第 1051-1080 行使用条件编译来适配特性开关、平台或可选后端；在 `init` 等例程中引入具体执行逻辑。

### Lines 1081-1110 / 第 1081-1110 行

```cpp
1081:   }
1082: 
1083:   getGlobalRankStartAndStride(
1084:       options_->global_ranks_in_group,
1085:       this->globalRankStart_,
1086:       this->globalRankStride_);
1087: 
1088:   // Attach hooks to cache allocator to trigger the hooks whenever a traced
1089:   // action is called. In the following hooks, we register a newly allocated
1090:   // segment when SEGMENT_ALLOC action occurs, and deregister a segment when
1091:   // SEGMENT_FREE action occurs.
1092:   if (shouldAllCommunicatorsRegisterAllTensors()) {
1093:     // This call is idempotent.
1094:     attachAllocatorHooks();
1095:   }
1096: }
1097: 
1098: void ProcessGroupNCCL::eagerConnectSingleDevice(at::Device device) {
1099:   const auto key = getKeyFromDevice(device);
1100:   LOG(INFO) << logPrefix() << "Eagerly connecting nccl backend with device "
1101:             << device;
1102:   initNCCLComm(key, device, OpType::ALLREDUCE);
1103:   eagerInit_ = true;
1104: }
1105: 
1106: bool ProcessGroupNCCL::useNonblocking() {
1107: #ifndef NCCL_HAS_COMM_NONBLOCKING
1108:   return false;
1109: #endif // NCCL_HAS_COMM_NONBLOCKING
1110:   // Already parsed, return the cached value
```

- EN: Lines 1081-1110 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `getGlobalRankStartAndStride`, `attachAllocatorHooks`, `initNCCLComm`.
- CN: 第 1081-1110 行使用条件编译来适配特性开关、平台或可选后端；在 `getGlobalRankStartAndStride`、`attachAllocatorHooks`、`initNCCLComm` 等例程中引入具体执行逻辑。

### Lines 1111-1140 / 第 1111-1140 行

```cpp
1111:   if (useNonblocking_.has_value()) {
1112:     return useNonblocking_.value();
1113:   }
1114:   // Get environment variable.
1115:   auto nbEnv = c10::utils::check_env("TORCH_NCCL_USE_COMM_NONBLOCKING");
1116: 
1117:   // 1st priority: Respect the user's setting
1118:   if (options_->config.blocking != NCCL_CONFIG_UNDEF_INT) {
1119:     useNonblocking_ = options_->config.blocking == 0;
1120:   }
1121:   // 2nd priority: Respect the environment variable
1122:   else if (nbEnv.has_value()) {
1123:     useNonblocking_ = nbEnv;
1124:   }
1125:   // 3rd priority: automatically use nonblocking if we are in eager init mode
1126:   // Note: this automatic selection is disabled in torch 2.7.1 to work around a
1127:   // hang in NCCL 2.26 in non-blocking mode. We can revisit if NCCL fixes the
1128:   // bug. See https://github.com/pytorch/pytorch/issues/153960
1129:   // else if (getBoundDeviceId()) {
1130:   //   useNonblocking_ = true;
1131:   // }
1132:   // 4th priority: otherwise, nonblocking = false to preserve old behavior
1133:   else {
1134:     useNonblocking_ = false;
1135:   }
1136: 
1137:   LOG(INFO) << logPrefix()
1138:             << "Using non-blocking mode: " << useNonblocking_.value();
1139:   return useNonblocking_.value();
1140: }
```

- EN: Lines 1111-1140 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1111-1140 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1141-1170 / 第 1141-1170 行

```cpp
1141: 
1142: void ProcessGroupNCCL::performNocolorSplit(at::Device device) {
1143:   // If our backend doesn't support splitting, this is a no-op for
1144:   // ranks not in the new subgroup (and ranks that would be in it will
1145:   // just use a new communicator rather than split).
1146: #ifdef NCCL_HAS_COMM_SPLIT
1147:   const auto key = getKeyFromDevice(device);
1148:   LOG(INFO) << logPrefix() << "Performing nocolor split on backend device "
1149:             << device << ", key " << key << ", i am " << this;
1150:   bool useNb = useNonblocking();
1151:   options_->config.blocking = useNb ? 0 : 1;
1152:   auto comm = getNCCLComm(key);
1153:   if (comm == nullptr) {
1154:     LOG(ERROR) << logPrefix()
1155:                << "No parent communicator exists for nocolor split";
1156:   }
1157:   NCCLComm::split(comm.get(), NCCL_SPLIT_NOCOLOR, rank_, options_->config);
1158: #endif // NCCL_HAS_COMM_SPLIT
1159: }
1160: 
1161: bool ProcessGroupNCCL::isInitialized() {
1162:   if (devNCCLCommMap_.empty()) {
1163:     return false;
1164:   }
1165:   std::lock_guard<std::mutex> lock(mutex_);
1166:   bool initialized = true;
1167:   for (const auto& [_, comm] : devNCCLCommMap_) {
1168:     if (!comm->isInitialized()) {
1169:       initialized = false;
1170:       break;
```

- EN: Lines 1141-1170 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1141-1170 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1171-1200 / 第 1171-1200 行

```cpp
1171:     }
1172:   }
1173:   return initialized;
1174: }
1175: 
1176: ErrorType ProcessGroupNCCL::getError() {
1177:   std::lock_guard<std::mutex> lock(errorMutex_);
1178:   return error_;
1179: }
1180: 
1181: void ProcessGroupNCCL::registerMemPool(at::cuda::MemPool* pool, bool symm) {
1182:   using c10::cuda::CUDACachingAllocator::SegmentInfo;
1183:   const auto key = std::to_string(pool->device());
1184:   LOG(INFO) << logPrefix()
1185:             << "Performing NCCL user buffer registration for all buffers in "
1186:             << "MemPool: " << pool->id() << ", device index: " << key
1187:             << ", i am " << this;
1188:   auto ncclComm = getNCCLComm(key);
1189:   if (ncclComm == nullptr) {
1190:     C10_THROW_ERROR(
1191:         DistBackendError,
1192:         "NCCL communicator has not been initialized before mem pool creation. You can pass `device_id` to init_process_group -- one way of eager initialization -- to work around this issue");
1193:   }
1194:   {
1195:     std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
1196:     auto iter = ncclCommMemPoolMap.find(ncclComm);
1197:     iter->second.insert(std::make_tuple(pool->id(), symm));
1198:   }
1199:   // We must ensure we're listening for allocator trace events in order to
1200:   // register future segments allocated in this pool (this call is idempotent).
```

- EN: Lines 1171-1200 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1171-1200 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1201-1230 / 第 1201-1230 行

```cpp
1201:   attachAllocatorHooks();
1202:   auto snapshot = c10::cuda::CUDACachingAllocator::snapshot(pool->id());
1203:   std::sort(
1204:       snapshot.segments.begin(),
1205:       snapshot.segments.end(),
1206:       [](const SegmentInfo& a, const SegmentInfo& b) {
1207:         return a.registration_counter < b.registration_counter;
1208:       });
1209:   for (const auto& segmentInfo : snapshot.segments) {
1210:     TORCH_INTERNAL_ASSERT(
1211:         segmentInfo.registration_counter >= 0,
1212:         "SegmentInfo has uninitialized registration counter");
1213:     TORCH_INTERNAL_ASSERT(
1214:         segmentInfo.device == pool->device(),
1215:         "Mismatch between CUDA memory segment device and pool's device");
1216:     ncclComm->registerSegment(
1217:         // NOLINTNEXTLINE(performance-no-int-to-ptr)
1218:         reinterpret_cast<void*>(segmentInfo.address),
1219:         segmentInfo.total_size,
1220:         /*errorOnRereg=*/false, // ignores reregistration error
1221:         /*window*/ symm); // whether to use NCCL symmetric memory
1222:   }
1223: }
1224: 
1225: void ProcessGroupNCCL::deregisterMemPool(at::cuda::MemPool* pool) {
1226:   const auto key = std::to_string(pool->device());
1227:   LOG(INFO) << logPrefix()
1228:             << "Performing NCCL user buffer deregistration for all buffers in "
1229:             << "MemPool: " << pool->id() << ", device index: " << key
1230:             << ", i am " << this;
```

- EN: Lines 1201-1230 introduces executable logic in routines such as `attachAllocatorHooks`, `TORCH_INTERNAL_ASSERT`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1201-1230 行在 `attachAllocatorHooks`、`TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1231-1260 / 第 1231-1260 行

```cpp
1231:   auto ncclComm = getNCCLComm(key);
1232:   if (ncclComm == nullptr) {
1233:     C10_THROW_ERROR(
1234:         DistBackendError,
1235:         "NCCL communicator has not been initialized before mem pool creation. You can pass `device_id` to init_process_group -- one way of eager initialization -- to work around this issue");
1236:   }
1237:   bool symm;
1238:   {
1239:     std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
1240:     auto iter = ncclCommMemPoolMap.find(ncclComm);
1241:     auto mempool_it = std::find_if(
1242:         iter->second.begin(), iter->second.end(), [&](const auto& tup) {
1243:           return std::get<0>(tup) == pool->id();
1244:         });
1245:     TORCH_CHECK(
1246:         mempool_it != iter->second.end(),
1247:         "Trying to unregister not previously registered pool");
1248:     symm = std::get<1>(*mempool_it);
1249:     iter->second.erase(mempool_it);
1250:   }
1251:   auto snapshot = c10::cuda::CUDACachingAllocator::snapshot(pool->id());
1252:   for (const auto& segmentInfo : snapshot.segments) {
1253:     TORCH_INTERNAL_ASSERT(
1254:         segmentInfo.device == pool->device(),
1255:         "Mismatch between CUDA memory segment device and pool's device");
1256:     // NOLINTNEXTLINE(performance-no-int-to-ptr)
1257:     ncclComm->deregisterSegment(
1258:         reinterpret_cast<void*>(segmentInfo.address), symm);
1259:   }
1260: }
```

- EN: Lines 1231-1260 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1231-1260 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1261-1290 / 第 1261-1290 行

```cpp
1261: 
1262: c10::intrusive_ptr<intra_node_comm::IntraNodeComm> ProcessGroupNCCL::
1263:     initIntraNodeComm() {
1264:   using IntraNodeComm = intra_node_comm::IntraNodeComm;
1265:   if (!IntraNodeComm::isEnabled()) {
1266:     return nullptr;
1267:   }
1268:   auto prefixStore = c10::make_intrusive<PrefixStore>("IntraNodeComm", store_);
1269:   const std::string groupName =
1270:       options_->group_name.empty() ? "0" : options_->group_name;
1271:   auto comm = c10::make_intrusive<IntraNodeComm>(
1272:       prefixStore, rank_, size_, std::nullopt, groupName);
1273:   if (comm->rendezvous()) {
1274:     return comm;
1275:   } else {
1276:     return nullptr;
1277:   }
1278: }
1279: 
1280: void ProcessGroupNCCL::setSequenceNumberForGroup() {
1281: } // NCCL just starts sequence numbers at 0.
1282: 
1283: uint64_t ProcessGroupNCCL::getSequenceNumberForGroup() {
1284:   return seqCollective_;
1285: }
1286: 
1287: void ProcessGroupNCCL::registerOnCompletionHook(
1288:     std::function<void(std::shared_ptr<WorkInfo>)>&& hook) {
1289:   TORCH_WARN_ONCE(
1290:       "ProcessGroupNCCL OnCompletion hook will be deprecated in favor of Flight Recorder. "
```

- EN: Lines 1261-1290 introduces executable logic in routines such as `initIntraNodeComm`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1261-1290 行在 `initIntraNodeComm` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1291-1320 / 第 1291-1320 行

```cpp
1291:       "Please check out FlightRecorder.hpp for information that is recorded at work completion. "
1292:       "You can file an issue if you want additional information to be recorded. "
1293:       "You can also file an RFC if you want Flight Recorder to accept plugins that customize the recording.")
1294: 
1295:   TORCH_CHECK_WITH(
1296:       DistBackendError,
1297:       onCompletionHook_ == nullptr,
1298:       "ProcessGroupNCCL OnCompletion hook already registered");
1299: 
1300:   TORCH_CHECK_WITH(
1301:       ValueError,
1302:       enableTiming_.load(),
1303:       "ProcessGroupNCCL OnCompletion hook requires recording start and end "
1304:       "events which require setting TORCH_NCCL_ENABLE_TIMING environment variable. "
1305:       "This is only available for NCCL version >= 2.4.");
1306:   onCompletionHook_ = std::move(hook);
1307:   onCompletionHookThread_ = std::thread(&ProcessGroupNCCL::runHookLoop, this);
1308: }
1309: 
1310: // must release GIL when calling this method
1311: void ProcessGroupNCCL::waitForPendingWorks() {
1312:   // Reasoning about hook completion:
1313:   // 1. waitForPendingWorks should be called after user code has finished
1314:   // calling
1315:   //    all collectives. This means, when we got here, all of the collectives
1316:   //    are either in workMetaList_ or has been erased from workMetaList_.
1317:   // 2. The watchdog thread grabs both locks to move Work object from the
1318:   //    workMetaList_ to the completedWorkList_, and the hook thread only erases
1319:   //    a Work object after the hook is returned. Therefore, after user code
1320:   //    calls a collective, its Work object is either in workMetaList_ or in
```

- EN: Lines 1291-1320 introduces executable logic in routines such as `TORCH_CHECK_WITH`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1291-1320 行在 `TORCH_CHECK_WITH` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1321-1350 / 第 1321-1350 行

```cpp
1321:   //    completedWorkList_ before it finishes.
1322:   // 3. We have three threads and two locks.
1323:   //      a. main thread (this function) grabs two locks atomically
1324:   //      b. watchdog thread (runLoop function) always grabs
1325:   //      workMetaListMutex_
1326:   //         first and then grabs completedWorkListMutex_.
1327:   //      c. hook thread (runHookLoop function) only grabs
1328:   //      completedWorkListMutex_. Therefore, locks are always acquired in the
1329:   //      same order and hence no deadlocks.
1330:   while (true) {
1331:     {
1332:       std::lock(workMetaListMutex_, completedWorkListMutex_);
1333:       std::lock_guard<std::mutex> lockWork(workMetaListMutex_, std::adopt_lock);
1334:       std::lock_guard<std::mutex> lockHook(
1335:           completedWorkListMutex_, std::adopt_lock);
1336: 
1337:       if (workMetaList_.empty() && completedWorkList_.empty()) {
1338:         return;
1339:       }
1340:     }
1341: 
1342:     std::this_thread::sleep_for(
1343:         std::chrono::milliseconds(kWatchdogThreadSleepMillis));
1344:   }
1345: }
1346: 
1347: void ProcessGroupNCCL::enableCollectivesTiming() {
1348:   enableTiming_.store(true);
1349: }
1350: 
```

- EN: Lines 1321-1350 introduces executable logic in routines such as `lockWork`, `lockHook`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1321-1350 行在 `lockWork`、`lockHook` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1351-1380 / 第 1351-1380 行

```cpp
1351: c10::intrusive_ptr<Backend> ProcessGroupNCCL::split(
1352:     const c10::intrusive_ptr<Store>& store,
1353:     const std::vector<int>& ranks,
1354:     const c10::intrusive_ptr<Backend::Options>& opts) {
1355:   auto deviceIdx = guessDeviceId();
1356:   TORCH_CHECK(
1357:       deviceIdx >= 0,
1358:       "ProcessGroupNCCL::split: rank ",
1359:       rank_,
1360:       " has no device is bound to this rank.");
1361:   auto device = at::Device(at::DeviceType::CUDA, deviceIdx);
1362:   auto it = std::find(ranks.begin(), ranks.end(), rank_);
1363:   int groupRank;
1364:   if (it == ranks.end()) {
1365:     // This rank is not in the new group, so no_color split should be called
1366:     performNocolorSplit(device);
1367:     return nullptr;
1368:   } else {
1369:     groupRank = std::distance(ranks.begin(), it);
1370:   }
1371: 
1372:   auto ncclOpts = c10::dynamic_intrusive_pointer_cast<Options>(opts);
1373:   TORCH_CHECK(ncclOpts != nullptr, "opts not a ProcessGroupNCCL::Options.");
1374: 
1375:   // TODO: we need to get rid of globalRanksInGroup eventually.
1376:   std::vector<uint64_t> globalRanksInGroup;
1377:   for (auto rank : ranks) {
1378:     globalRanksInGroup.emplace_back(groupRanks()[rank]);
1379:   }
1380:   ncclOpts->split_from =
```

- EN: Lines 1351-1380 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1351-1380 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1381-1410 / 第 1381-1410 行

```cpp
1381:       c10::intrusive_ptr<ProcessGroupNCCL>::unsafe_reclaim_from_nonowning(this);
1382:   ncclOpts->global_ranks_in_group = std::move(globalRanksInGroup);
1383:   // We use the lowest rank in the group as the split_color as each rank can
1384:   // only participate in one group.
1385:   // This value must be non-negative int32 and all ranks are.
1386:   ncclOpts->split_color = *std::min_element(ranks.cbegin(), ranks.cend());
1387:   auto pg = c10::make_intrusive<ProcessGroupNCCL>(
1388:       store->clone(), groupRank, ranks.size(), ncclOpts);
1389: #ifdef NCCL_COMM_DESCRIPTION
1390:   // We need to set the desc here so that when eager init the nccl, we can
1391:   // propagate desc to the nccl comm.
1392:   pg->setGroupDesc(ncclOpts->group_desc);
1393: #endif // NCCL_COMM_DESCRIPTION
1394:   pg->eagerConnectSingleDevice(device);
1395:   return c10::static_intrusive_pointer_cast<Backend>(pg);
1396: }
1397: 
1398: c10::intrusive_ptr<Backend> ProcessGroupNCCL::merge(
1399:     const c10::intrusive_ptr<Store>& store,
1400:     const c10::intrusive_ptr<Backend::Options>& opts,
1401:     const int& rank,
1402:     const int& size) {
1403:   auto ncclOpts = c10::dynamic_intrusive_pointer_cast<Options>(opts);
1404:   TORCH_CHECK(ncclOpts != nullptr, "opts not a ProcessGroupNCCL::Options.");
1405:   auto pg = c10::make_intrusive<ProcessGroupNCCL>(
1406:       store->clone(), rank, size, ncclOpts);
1407:   return c10::static_intrusive_pointer_cast<Backend>(pg);
1408: }
1409: 
1410: bool ProcessGroupNCCL::waitForFutureOrTimeout(
```

- EN: Lines 1381-1410 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1381-1410 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1411-1440 / 第 1411-1440 行

```cpp
1411:     std::future<bool>& fut,
1412:     const std::chrono::milliseconds& timeOutMilSec,
1413:     const std::string& futDescription,
1414:     ::c10d::C10dLoggingData& debugLog,
1415:     bool throwException) {
1416:   std::string errorMsg;
1417:   bool complete = false;
1418: 
1419:   TORCH_CHECK(fut.valid(), "Expected a valid future");
1420:   std::future_status status = fut.wait_for(timeOutMilSec);
1421:   if (status == std::future_status::ready) {
1422:     // Calling .get() will re-raise any exception from the future, and we don't
1423:     // care about the retval
1424:     try {
1425:       bool result = fut.get();
1426:       if (result) {
1427:         VLOG(2) << logPrefix()
1428:                 << "future successfully executed for: " << futDescription;
1429:         debugLog.strings["status"] = "SUCCESS";
1430:         complete = true;
1431:       }
1432:     } catch (const std::exception& e) {
1433:       errorMsg = c10::str(
1434:           logPrefix(),
1435:           "Exception thrown when waiting for future ",
1436:           futDescription,
1437:           ": ",
1438:           e.what());
1439: 
1440:       debugLog.strings["status"] = "EXCEPTION";
```

- EN: Lines 1411-1440 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1411-1440 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1441-1470 / 第 1441-1470 行

```cpp
1441:       debugLog.strings["exception_msg"] = e.what();
1442:       LOG(ERROR) << errorMsg;
1443:     } catch (...) {
1444:       errorMsg = c10::str(
1445:           logPrefix(),
1446:           "Unknown exception thrown when waiting for future ",
1447:           futDescription);
1448:       debugLog.strings["status"] = "EXCEPTION";
1449:       debugLog.strings["exception_msg"] = "Unknown exception";
1450:       LOG(ERROR) << errorMsg;
1451:     }
1452:   } else {
1453:     errorMsg = c10::str(
1454:         logPrefix(),
1455:         "Future for ",
1456:         futDescription,
1457:         " timed out after ",
1458:         timeOutMilSec.count(),
1459:         " ms");
1460:     debugLog.strings["status"] = "TIMEOUT";
1461:     LOG(ERROR) << errorMsg;
1462:   }
1463:   if (throwException && !errorMsg.empty()) {
1464:     C10_THROW_ERROR(DistBackendError, errorMsg);
1465:   }
1466:   return complete;
1467: }
1468: 
1469: void ProcessGroupNCCL::abortCommsFromMap(
1470:     std::unordered_map<std::string, std::shared_ptr<NCCLComm>>& ncclCommsMap,
```

- EN: Lines 1441-1470 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1441-1470 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1471-1500 / 第 1471-1500 行

```cpp
1471:     const std::optional<std::string>& abortReason) {
1472:   // The process may control multiple devices, loop through the communicators on
1473:   // each device
1474:   // NCCL expects Group abort when there are multiple communicators created in a
1475:   // device. Group abort requires 2.22.0 release and up.
1476:   if (getNcclVersionNumber() >= NCCL_VERSION(2, 22, 0)) {
1477:     groupStart();
1478:   }
1479:   for (auto& it : ncclCommsMap) {
1480:     auto& devName = it.first;
1481:     auto& ncclComm = it.second;
1482:     VLOG(2) << logPrefix() << "ProcessGroupNCCL destroying ncclComm_ "
1483:             << ncclComm->repr() << " on CUDA device: " << devName;
1484:     // abort() call now has GPU guard inside
1485:     ncclComm->abort(abortReason);
1486:     // Note that we don't remove the aborted communicators from the
1487:     // cache. The reason is that if we do remove the communicator
1488:     // from the cache, it is possible that a new collective operation
1489:     // calls `ncclCommInitRank` to create a new communicator whereas
1490:     // other ranks might have failed/timed out and didn't enter
1491:     // `ncclCommInitRank`. As a result, when there is a failure on
1492:     // a communicator the application receives an exception and its
1493:     // their responsibility to destroy the process group and recreate
1494:     // it to recover from errors.
1495: 
1496:     VLOG(2) << logPrefix() << "ProcessGroupNCCL destroyed "
1497:             << " communicator on CUDA device: " << devName;
1498:   }
1499:   if (getNcclVersionNumber() >= NCCL_VERSION(2, 22, 0)) {
1500:     groupEnd();
```

- EN: Lines 1471-1500 introduces executable logic in routines such as `groupStart`, `groupEnd`.
- CN: 第 1471-1500 行在 `groupStart`、`groupEnd` 等例程中引入具体执行逻辑。

### Lines 1501-1530 / 第 1501-1530 行

```cpp
1501:   }
1502: }
1503: 
1504: // Abort all communicators on this rank
1505: // Note: original name of this method is `abort`. It was renamed to
1506: // `abortComms` to distinguish from the `abort` method below. The `abort`
1507: // method calls `abortComms` but does more destruction than the latter.
1508: bool ProcessGroupNCCL::abortComms(
1509:     const std::optional<std::string>& abortReason) {
1510:   // Remove record from global ncclCommMemPoolMapMutex before aboarting,
1511:   // so that a new cache segment would not register to already aborted
1512:   // communicators. Note that ncclCommMemPoolMap is a global container which may
1513:   // contain other PG's communicators, thus we need to only erase communicators
1514:   // for the current PG.
1515:   {
1516:     std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
1517:     for (auto& [_, ncclComm] : devNCCLCommMap_) {
1518:       ncclCommMemPoolMap.erase(ncclComm);
1519:     }
1520:   }
1521: 
1522:   std::lock_guard<std::mutex> lock(mutex_);
1523:   abortCommsFromMap(devNCCLCommMap_, abortReason);
1524:   abortCommsFromMap(inInitializationCommMap_, abortReason);
1525:   return true;
1526: }
1527: 
1528: void ProcessGroupNCCL::dumpExtraDebuggingInfo() {
1529:   // This extra dump is intended to capture the current snapshot of collectives
1530:   // When this process group is terminated for some exception out of NCCL
```

- EN: Lines 1501-1530 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1501-1530 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1531-1560 / 第 1531-1560 行

```cpp
1531:   bool dumpExtraOnExec_ = getCvarBool(TORCH_NCCL_EXTRA_DUMP_ON_EXEC, false);
1532:   if (dumpExtraOnExec_) {
1533:     bool should_dump_local = false;
1534:     bool succeeded = shouldDump_.compare_exchange_strong(
1535:         should_dump_local,
1536:         true,
1537:         std::memory_order_release,
1538:         std::memory_order_acquire);
1539:     if (succeeded) {
1540:       LOG(INFO) << logPrefix() << "Sending extra dumping signal";
1541:       broadcastDumpSignal();
1542:       // When this routine is called, exception is captured so
1543:       // dumping by default_pg is not guaranteed due to early termination of
1544:       // process So we call dumping manually here
1545:       bool onlyActive = getCvarBool(TORCH_INCLUDE_ONLY_ACTIVE, false);
1546:       // Stacktrace is not included at the moment to prevent deadlock due to GIL
1547:       dumpDebuggingInfo(false, onlyActive);
1548:     }
1549:   }
1550: }
1551: 
1552: // Abort this backend.
1553: void ProcessGroupNCCL::abort() {
1554:   // This will log counter for how long the abort actually takes.
1555:   STATIC_SCOPED_WAIT_COUNTER(pytorch.ProcessGroupNCCL__abort);
1556: 
1557:   dumpExtraDebuggingInfo();
1558:   // Don't join threads here since the purpose of this method is to abort all
1559:   // communicators and signal the threads to exit. Joining on the threads could
1560:   // potentially block and hence avoid it in this method.
```

- EN: Lines 1531-1560 introduces executable logic in routines such as `broadcastDumpSignal`, `dumpExtraDebuggingInfo`.
- CN: 第 1531-1560 行在 `broadcastDumpSignal`、`dumpExtraDebuggingInfo` 等例程中引入具体执行逻辑。

### Lines 1561-1590 / 第 1561-1590 行

```cpp
1561:   terminateProcessGroup_.store(true);
1562:   watchdog_->notify();
1563:   // launch abort asynchronously and wait for it to complete or timeout
1564:   LOG(INFO) << logPrefix()
1565:             << "Launching ProcessGroupNCCL abort asynchronously.";
1566:   std::future<bool> fut =
1567:       std::async(std::launch::async, [this]() { return this->abortComms(); });
1568: 
1569:   ::c10d::C10dLoggingData debugLog;
1570:   waitForFutureOrTimeout(
1571:       fut, options_->timeout, "ProcessGroup abort", debugLog, true);
1572:   LOG(INFO) << logPrefix() << "ProcessGroupNCCL aborts successfully.";
1573: 
1574:   // We need to wait for abort to finish before we can safely shut down
1575:   // heartbeat monitoring thread.
1576:   heartbeatMonitor_->stop();
1577: }
1578: 
1579: // Difference between `abort()` and `shutdown()`:
1580: // 1. `abort()` will signal communicators to terminate all NCCL kernels
1581: // immediately.
1582: // 2. `shutdown()` will wait for all NCCL kernels to finish before destroying
1583: // communicators.
1584: 
1585: // Destroy (shutdown) this backend -- normal exit.
1586: void ProcessGroupNCCL::shutdown() {
1587:   LOG(INFO) << logPrefix()
1588:             << "Starting to destroy process group, flushing operations.";
1589:   // Flush all collectives
1590:   {
```

- EN: Lines 1561-1590 introduces executable logic in routines such as `waitForFutureOrTimeout`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1561-1590 行在 `waitForFutureOrTimeout` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1591-1620 / 第 1591-1620 行

```cpp
1591:     std::lock_guard<std::mutex> lock(mutex_);
1592:     for (auto& it : devNCCLCommMap_) {
1593:       auto& ncclComm = it.second;
1594:       ncclComm->finalize();
1595:     }
1596:   }
1597:   // Wait for all operations to complete.  If NCCL comm is non-blocking and
1598:   // timeout is reach, this will throw an exception.
1599:   for (auto& it : devNCCLCommMap_) {
1600:     auto& ncclComm = it.second;
1601:     // Use long interval to avoid acquiring CPU too frequently
1602:     ncclComm->waitReady(true);
1603:   }
1604:   // Deregister memory pool after finalizing all collectives
1605:   if (memPool_) {
1606:     try {
1607:       deregisterMemPool(memPool_.get());
1608:     } catch (...) {
1609:       LOG(ERROR) << logPrefix() << "Failed to deregister memory pool, ignoring";
1610:     }
1611:   }
1612:   // Tell watchdog to (1) flush its queue and (2) do not use comm objects
1613:   // anymore because I am going to destroy them now
1614:   LOG(INFO) << logPrefix() << "Operations flushed, joining watchdog thread.";
1615:   terminateProcessGroup_.store(true);
1616:   watchdog_->notify();
1617:   watchdog_->join();
1618:   if (onCompletionHookThread_.joinable()) {
1619:     onCompletionHookThread_.join();
1620:   }
```

- EN: Lines 1591-1620 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1591-1620 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1621-1650 / 第 1621-1650 行

```cpp
1621:   // Watchdog thread exiting, retire heartbeat monitoring thread now to avoid
1622:   // false alarm
1623:   heartbeatMonitor_->stop();
1624:   // Destroy the communicator, reclaim resources
1625:   LOG(INFO) << logPrefix() << "Watchdog joined, destroying NCCL communicators.";
1626:   {
1627:     std::lock_guard<std::mutex> lock(mutex_);
1628:     for (auto& it : devNCCLCommMap_) {
1629:       auto& ncclComm = it.second;
1630:       ncclComm->destroy();
1631:     }
1632:   }
1633:   LOG(INFO) << logPrefix() << "Destroy complete.";
1634: }
1635: 
1636: // NOLINTNEXTLINE(bugprone-exception-escape)
1637: ProcessGroupNCCL::~ProcessGroupNCCL() {
1638:   LOG(INFO) << logPrefix() << "ProcessGroupNCCL destructor entered.";
1639: 
1640:   // `shutdown()` or `abort` already called. Skip the favor of disposing
1641:   // communicators.
1642:   if (!terminateProcessGroup_.load()) {
1643:     // If user haven't explicitly destroy/shutdown process group, destructor
1644:     // needs to do so
1645:     // First print warning on first rank of each node
1646:     if (rank_ % localDeviceCount_ == 0) {
1647:       TORCH_WARN_ONCE(
1648:           "WARNING: destroy_process_group() was not called before program exit, "
1649:           "which can leak resources. For more info, please see "
1650:           "https://pytorch.org/docs/stable/distributed.html#shutdown");
```

- EN: Lines 1621-1650 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1621-1650 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1651-1680 / 第 1651-1680 行

```cpp
1651:     }
1652: 
1653:     // Note 1: in distributed_c10d.py, a reference to PG is held by the global
1654:     // context. Therefore, we are here only when the global context is tearing
1655:     // down, which means the entire program is exiting.  At this point, user
1656:     // will no longer care about the result of any collective, thus we can use
1657:     // abort instead of destroy to make the destruction non-blocking.
1658: 
1659:     // TODO: Note 1 is not true in case of a C++ program using libtorch, which
1660:     // does not have the global context mentioned. In that case, calling
1661:     // `abort()` here could lead to corrupted result. We should consider not
1662:     // doing anything and just let things leak. Adversarial example:
1663:     /*
1664:       Work routine(Tensor& t) {
1665:         pg = ProcessGroupNCCL(…);
1666:         w = pg.allReduce(t);
1667:         return w;
1668:       }
1669:     */
1670:     abort();
1671:   }
1672: 
1673:   // Make sure we've told threads to stop; doesn't hurt if we'd done so before.
1674:   // Tell watchdog and onCompletionHook:
1675:   terminateProcessGroup_.store(true);
1676:   watchdog_->notify();
1677:   // Tell heartbeat thread:
1678:   heartbeatMonitor_->stop();
1679: 
1680:   // Wait for all threads to finish before returning
```

- EN: Lines 1651-1680 introduces executable logic in routines such as `routine`, `abort`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1651-1680 行在 `routine`、`abort` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1681-1710 / 第 1681-1710 行

```cpp
1681:   watchdog_->join();
1682:   heartbeatMonitor_->join();
1683:   if (onCompletionHookThread_.joinable()) {
1684:     onCompletionHookThread_.join();
1685:     LOG(INFO) << logPrefix()
1686:               << "ProcessGroupNCCL onCompletionHookThread thread joined.";
1687:   }
1688: }
1689: 
1690: bool ProcessGroupNCCL::dumpDebuggingInfo(
1691:     bool includeStackTrace /*=true*/,
1692:     bool onlyActive /*=false*/) {
1693:   // This will log counter for how long dumpDebuggingInfo actually takes.
1694:   STATIC_SCOPED_WAIT_COUNTER(pytorch.ProcessGroupNCCL__dumpDebuggingInfo);
1695: 
1696:   // Serialize all calls to this function to avoid corrupting data, but allow
1697:   // multiple calls in one runtime. User is responsible for preserving the
1698:   // output file from an earlier call before a later call overwrites it.
1699:   static std::mutex writeDebugInfoMutex;
1700:   LOG(ERROR)
1701:       << logPrefix()
1702:       << "ProcessGroupNCCL preparing to dump debug info. Include stack trace: "
1703:       << includeStackTrace << ", only active collectives: " << onlyActive;
1704:   if (traceBufferSize_ > 0) {
1705:     // We dump nccl trace into local disk by default and users can register
1706:     // their customized writer by inheriting `DebugInfoWriter` via
1707:     // `registerDebugInfoWriter`.
1708:     auto ncclTrace = dump_nccl_trace(true, includeStackTrace, onlyActive);
1709:     // dump_nccl_trace will hang so we don't grab the global lock until we get
1710:     // the trace.
```

- EN: Lines 1681-1710 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1681-1710 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1711-1740 / 第 1711-1740 行

```cpp
1711:     std::lock_guard<std::mutex> lock(writeDebugInfoMutex);
1712:     DebugInfoWriter& writer = DebugInfoWriter::getWriter(globalRank());
1713:     LOG(INFO) << logPrefix() << "ProcessGroupNCCL dumping nccl trace to "
1714:               << writer.getWriterTarget();
1715:     writer.write(ncclTrace);
1716:     LOG(INFO) << logPrefix() << "Flight Recorder trace successfully dumped.";
1717:     return true;
1718:   }
1719:   return false;
1720: }
1721: 
1722: void ProcessGroupNCCL::terminateProcess(const std::string& errMsg) {
1723:   // Logging with `FATAL`, after errMsg printed, it calls `std::abort()`
1724:   // to terminate the program execution.
1725:   LOG(FATAL) << logPrefix() << errMsg;
1726: }
1727: 
1728: static long computeDeltaMS(
1729:     std::chrono::time_point<std::chrono::steady_clock> start,
1730:     std::chrono::time_point<std::chrono::steady_clock> end) {
1731:   return std::chrono::duration_cast<std::chrono::milliseconds>(end - start)
1732:       .count();
1733: }
1734: 
1735: void ProcessGroupNCCL::setEnableNanCheck(bool enableNanCheck) {
1736:   enableNanCheck_ = enableNanCheck;
1737: }
1738: 
1739: std::string ProcessGroupNCCL::HeartbeatMonitor::getNCCLWatchdogTimeoutErrorMsg(
1740:     const std::string& extraMsg) {
```

- EN: Lines 1711-1740 introduces executable logic in routines such as `computeDeltaMS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1711-1740 行在 `computeDeltaMS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1741-1770 / 第 1741-1770 行

```cpp
1741:   return c10::str(
1742:       pg_->logPrefix(),
1743:       "Received a dump signal due to a collective timeout from ",
1744:       extraMsg,
1745:       " and we will try our best to dump the debug info. ",
1746:       "Last enqueued NCCL work: ",
1747:       pg_->pgStatus_->lastEnqueuedSeq,
1748:       ", last completed NCCL work: ",
1749:       pg_->pgStatus_->lastCompletedSeq,
1750:       ".",
1751:       "This is most likely caused by incorrect usages of collectives, e.g., wrong ",
1752:       "sizes used across ranks, the order of collectives is not same for all ranks ",
1753:       "or the scheduled collective, for some reason, didn't run. Additionally, ",
1754:       "this can be caused by GIL deadlock or other reasons such as network errors or ",
1755:       "bugs in the communications library (e.g. NCCL), etc. ");
1756: }
1757: 
1758: std::string ProcessGroupNCCL::HeartbeatMonitor::getNCCLWatchdogTimeoutExitMsg(
1759:     const std::string& exitReason) {
1760:   return c10::str(
1761:       pg_->logPrefix(),
1762:       "Terminating the process after attempting to dump debug info, due to ",
1763:       exitReason,
1764:       ".");
1765: }
1766: 
1767: void ProcessGroupNCCL::HeartbeatMonitor::setLastWorkListUpdateTime(
1768:     std::chrono::time_point<std::chrono::steady_clock> time) {
1769:   // We intentionally let the race condition to happen but this is ok
1770:   // as long as we update the time, we know we are making progress.
```

- EN: Lines 1741-1770 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1741-1770 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1771-1800 / 第 1771-1800 行

```cpp
1771:   lastWorkListUpdateTime_ = time;
1772: }
1773: 
1774: int ProcessGroupNCCL::HeartbeatMonitor::getDumpTimeout() const {
1775:   return waitTimeoutDumpInMilSec_;
1776: }
1777: 
1778: ProcessGroupNCCL::HeartbeatMonitor::HeartbeatMonitor(ProcessGroupNCCL* pg) {
1779:   pg_ = pg;
1780:   heartbeatTimeoutInSec_ =
1781:       getCvarInt(TORCH_NCCL_HEARTBEAT_TIMEOUT_SEC, 60 * 8 /*8 Mins*/);
1782:   waitTimeoutDumpInMilSec_ =
1783:       getCvarInt(TORCH_NCCL_WAIT_TIMEOUT_DUMP_MILSEC, 15 * 1000 /*15 Sec*/);
1784:   coordCheckIntervalMilSec_ = getCvarInt(TORCH_NCCL_COORD_CHECK_MILSEC, 1000);
1785:   // TODO, we should either deprecate TORCH_NCCL_DUMP_ON_TIMEOUT
1786:   // or change its name to reflect that dump happens on exception including
1787:   // both timeout and other errors.
1788:   dumpOnTimeoutOrEx_ = getCvarBool(TORCH_NCCL_DUMP_ON_TIMEOUT, true);
1789:   // logging C++ stack isn't safe. Gate it with an ENV.
1790:   logCppStackOnUncleanShutdown_ =
1791:       getCvarBool(TORCH_NCCL_LOG_CPP_STACK_ON_UNCLEAN_SHUTDOWN, true);
1792:   watchdogHeartbeatMonitorEnabled_ =
1793:       getCvarBool(TORCH_NCCL_ENABLE_MONITORING, true);
1794: 
1795:   // print out ENV settings for the heartbeat monitor thread.
1796:   if (pg_->getUid() == 0) {
1797:     LOG(INFO)
1798:         << pg_->logPrefix() << "HeartbeatMonitor environments: "
1799:         << "TORCH_NCCL_ENABLE_MONITORING (Whether to kill program when no watchdog heartbeat detected): "
1800:         << watchdogHeartbeatMonitorEnabled_
```

- EN: Lines 1771-1800 introduces executable logic in routines such as `getCvarInt`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1771-1800 行在 `getCvarInt` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1801-1830 / 第 1801-1830 行

```cpp
1801:         << ", TORCH_NCCL_DUMP_ON_TIMEOUT: " << dumpOnTimeoutOrEx_
1802:         << ", TORCH_NCCL_WAIT_TIMEOUT_DUMP_MILSEC: " << waitTimeoutDumpInMilSec_
1803:         << ", TORCH_NCCL_HEARTBEAT_TIMEOUT_SEC: " << heartbeatTimeoutInSec_
1804:         << ", TORCH_NCCL_COORD_CHECK_MILSEC: " << coordCheckIntervalMilSec_
1805:         << ", TORCH_NCCL_LOG_CPP_STACK_ON_UNCLEAN_SHUTDOWN: "
1806:         << logCppStackOnUncleanShutdown_;
1807:   }
1808: }
1809: 
1810: void ProcessGroupNCCL::HeartbeatMonitor::stop() {
1811:   terminateHeartbeatMonitorThread_.store(true);
1812:   monitorWakeUpCV_.notify_one();
1813: }
1814: 
1815: void ProcessGroupNCCL::HeartbeatMonitor::start() {
1816:   TORCH_CHECK(
1817:       !ncclHeartbeatMonitorThread_.joinable(),
1818:       "HeartbeatMonitor thread already started");
1819:   ncclHeartbeatMonitorThread_ =
1820:       std::thread(&ProcessGroupNCCL::HeartbeatMonitor::runLoop, this);
1821: }
1822: 
1823: void ProcessGroupNCCL::HeartbeatMonitor::join() {
1824:   if (ncclHeartbeatMonitorThread_.joinable()) {
1825:     ncclHeartbeatMonitorThread_.join();
1826:     LOG(INFO) << pg_->logPrefix()
1827:               << "ProcessGroupNCCL heart beat monitor thread joined.";
1828:   }
1829: }
1830: 
```

- EN: Lines 1801-1830 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1801-1830 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1831-1860 / 第 1831-1860 行

```cpp
1831: void ProcessGroupNCCL::HeartbeatMonitor::runLoop() {
1832:   c10::setThreadName("pt_nccl_heartbt");
1833:   STATIC_SCOPED_WAIT_COUNTER(
1834:       pytorch.ProcessGroupNCCL__HeartbeatMonitor__runLoop);
1835: 
1836:   uint64_t heartBeatCounter = 0ULL;
1837:   std::string errorMsg;
1838:   std::string exitReason;
1839:   bool checkDumpSignal = (dumpOnTimeoutOrEx_ && pg_->getUid() == 0);
1840:   int monitorPollInterval = checkDumpSignal ? coordCheckIntervalMilSec_
1841:                                             : heartbeatTimeoutInSec_ * 1000;
1842:   auto lastTimePollStore = std::chrono::steady_clock::now();
1843:   auto lastTimeHeartBeatCheck = std::chrono::steady_clock::now();
1844:   std::optional<DumpPipe> dumpPipe = std::nullopt;
1845:   // Use a pool to temporarily store the futures to avoid blocking when the code
1846:   // exits the scope of when future is generated by std::async.
1847:   std::vector<std::future<bool>> futures;
1848: 
1849:   if (pg_->getUid() == 0) {
1850:     // DumpPipe is one per-trainer process, and its convenient to name them
1851:     // after 'global' ranks in the system, So we assume processgroup (uid)==0 is
1852:     // the global PG and has globally unique rank ids across trainers.
1853:     dumpPipe.emplace(
1854:         pg_->globalRank(), pg_->debugInfoPipeFile_, pg_->traceBufferSize_);
1855:   }
1856:   while (true) {
1857:     // This won't have any lock since this lock is only used here.
1858:     // Please be aware that mutex `monitorMutex_` should not be used
1859:     // somewhere else to avoid the deadlock.
1860:     std::unique_lock<std::mutex> lock(monitorMutex_);
```

- EN: Lines 1831-1860 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1831-1860 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1861-1890 / 第 1861-1890 行

```cpp
1861:     if (monitorWakeUpCV_.wait_for(
1862:             lock, std::chrono::milliseconds(monitorPollInterval), [&] {
1863:               return terminateHeartbeatMonitorThread_.load();
1864:             })) {
1865:       // For the normal complete or user interception, monitorWakeUpCV_
1866:       // will get notified, we early return and exit heartbeatMonitor.
1867:       return;
1868:     }
1869:     auto currentTime = std::chrono::steady_clock::now();
1870: 
1871:     // We put extra functionality in the thread for the default PG (aka,
1872:     // local_id_=0) because the signal is same across different PGs. We only
1873:     // need to run once per process to avoid duplicate things performed in too
1874:     // many separate threads. For example, we check a global flag on the
1875:     // TCPStore periodically to see if any PG on any rank observed a timeout and
1876:     // signaled peers to dump debugging info, and we avoid hammering the
1877:     // TCPStore from all PGs on the same rank.
1878:     if (checkDumpSignal) {
1879:       // There are two scenarios where monitor thread will dump on timeout:
1880:       // 1. The current rank is the first to observe a timeout in watchdog.
1881:       // (shouldDump_ was set to true by the watchdog thread).
1882:       // 2. Other ranks detected the timeout and signal the current rank to
1883:       // dump. In addition, monitor threads will dump if watchdog threads has no
1884:       // heartbeat or dumpPipe is not empty.
1885:       if (shouldDump_.load()) {
1886:         errorMsg = getNCCLWatchdogTimeoutErrorMsg("this local rank");
1887:         exitReason = "collective timeout or exception";
1888:         break;
1889:       }
1890:       // We poll store to see if some ranks have flagged a timeout when
```

- EN: Lines 1861-1890 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1861-1890 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1891-1920 / 第 1891-1920 行

```cpp
1891:       // we haven't polled for `heartbeat_timeout` seconds and there haven't
1892:       // any work added or removed for `watchdog_timeout` seconds.
1893:       if (computeDeltaMS(lastWorkListUpdateTime_, currentTime) >=
1894:               kWatchdogThreadSleepMillis &&
1895:           computeDeltaMS(lastTimePollStore, currentTime) >=
1896:               coordCheckIntervalMilSec_) {
1897:         lastTimePollStore = currentTime;
1898:         auto handleError = [&](const std::string& errorMessage) {
1899:           LOG(WARNING)
1900:               << pg_->logPrefix()
1901:               << "Failed to check the \"should dump\" flag on TCPStore, "
1902:               << "(maybe TCPStore server has shut down too early), with error: "
1903:               << errorMessage;
1904:           // We give up for now assuming TCPStore has been torn down.
1905:           return;
1906:         };
1907:         // Wrap globalStore_->check() in a try-catch block to avoid crashing if
1908:         // the store is not available.
1909:         bool checkExceptionDump = false;
1910:         try {
1911:           checkExceptionDump =
1912:               pg_->globalStore()->check({std::string(kStoreDumpKey)});
1913:         } catch (const c10::DistNetworkError& e) {
1914:           handleError(e.msg());
1915:         } catch (const std::exception& e) {
1916:           handleError(e.what());
1917:         }
1918: 
1919:         if (checkExceptionDump) {
1920:           int timeOutRank = -1;
```

- EN: Lines 1891-1920 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1891-1920 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1921-1950 / 第 1921-1950 行

```cpp
1921:           if (!shouldDump_.load()) {
1922:             LOG(ERROR)
1923:                 << pg_->logPrefix()
1924:                 << "Observed flight recorder dump signal from another rank via TCPStore.";
1925:           }
1926:           shouldDump_.store(true);
1927:           try {
1928:             auto vec = pg_->globalStore()->get(std::string(kStoreDumpKey));
1929:             TORCH_CHECK_WITH(
1930:                 DistBackendError,
1931:                 vec.size() == sizeof(int),
1932:                 "Invalid size for the timeout rank ID");
1933:             std::memcpy(&timeOutRank, vec.data(), vec.size());
1934:           } catch (const std::exception& e) {
1935:             LOG(ERROR) << pg_->logPrefix()
1936:                        << "Failed to get timeout rank ID from TCPStore."
1937:                        << e.what();
1938:           }
1939:           errorMsg =
1940:               getNCCLWatchdogTimeoutErrorMsg(c10::str(" rank ", timeOutRank));
1941:           exitReason = "collective timeout or exception";
1942:           break;
1943:         }
1944:       }
1945:     }
1946: 
1947:     if (computeDeltaMS(lastTimeHeartBeatCheck, currentTime) >=
1948:         heartbeatTimeoutInSec_ * 1000l) {
1949:       // Check the heart beat of watchdog thread.
1950:       lastTimeHeartBeatCheck = currentTime;
```

- EN: Lines 1921-1950 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1921-1950 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1951-1980 / 第 1951-1980 行

```cpp
1951:       auto heartbeat = pg_->getWatchdogHeartbt();
1952:       if (heartbeat != heartBeatCounter) {
1953:         heartBeatCounter = heartbeat;
1954:       } else {
1955:         shouldDump_.store(true);
1956:         // Watchdog heartbeat timeout.
1957:         errorMsg = c10::str(
1958:             pg_->logPrefix(),
1959:             "ProcessGroupNCCL's watchdog got stuck for ",
1960:             heartbeatTimeoutInSec_,
1961:             " seconds without making progress in monitoring enqueued collectives. ",
1962:             "This typically indicates a NCCL/CUDA API (e.g., CudaEventDestroy) hang blocking the watchdog, ",
1963:             "and could be triggered by another thread holding the GIL inside a ",
1964:             "CUDA api (for example, CudaEventDestroy), or other deadlock-prone behaviors.",
1965:             "If you suspect the watchdog is not actually stuck and a longer timeout would help, ",
1966:             "you can either increase the timeout (TORCH_NCCL_HEARTBEAT_TIMEOUT_SEC) to a larger value "
1967:             "or disable the heartbeat monitor (TORCH_NCCL_ENABLE_MONITORING=0)."
1968:             "If either of aforementioned helps, feel free to file an issue to PyTorch about the short timeout "
1969:             "or false positive abort; otherwise, please attempt to debug the hang. ");
1970:         exitReason = "ProcessGroupNCCL watchdog hang";
1971:         break;
1972:       }
1973:     }
1974:     // process a request to dump the trace. only PG uid 0 will respond to dump
1975:     // requests, but this is fine since all PG's feed into the same flight
1976:     // recorder and dump. After dump, the training should continue.
1977:     if (dumpPipe.has_value() && dumpPipe->shouldDump()) {
1978:       // best effort dump, not waiting for the dump here
1979:       bool onlyActive = getCvarBool(TORCH_INCLUDE_ONLY_ACTIVE, false);
1980:       LOG(INFO) << pg_->logPrefix()
```

- EN: Lines 1951-1980 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1951-1980 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1981-2010 / 第 1981-2010 行

```cpp
1981:                 << "Dump signal received through pipe, triggering FR dump.";
1982:       futures.emplace_back(std::async(std::launch::async, [this, onlyActive]() {
1983:         return this->pg_->dumpDebuggingInfo(true, onlyActive);
1984:       }));
1985:     }
1986:   }
1987:   LOG(ERROR) << errorMsg;
1988: 
1989:   // We perform some checks to help users debug the timeout/hang issue:
1990:   // 1. Dump the nccl trace (flight recorder) to help debug the issue
1991:   //    (timeout after waitTimeoutDumpInMilSec_, which is one minute).
1992:   // 2. Check if there is a GIL deadlock (timeout after 300ms).
1993:   // 3. Try to dump the c++ stacktraces (blocking and would hang,
1994:   //    users can turn this off by set
1995:   //    TORCH_NCCL_LOG_CPP_STACK_ON_UNCLEAN_SHUTDOWN=0).
1996: 
1997:   // Dump the nccl trace (flight recorder).
1998:   if (checkDumpSignal && shouldDump_.load()) {
1999:     // Store debug info to storage if no other thread does it. (By default to
2000:     // local disk)
2001:     bool dumpStackTrace = getCvarBool(TORCH_INCLUDE_STACK_TRACE, true);
2002:     bool onlyActive = getCvarBool(TORCH_INCLUDE_ONLY_ACTIVE, false);
2003:     ::c10d::C10dLoggingData debugLog;
2004:     debugLog.integers["pg_id"] = static_cast<int64_t>(pg_->getUid());
2005:     debugLog.integers["rank"] = pg_->getRank();
2006:     debugLog.integers["global_rank"] = pg_->globalRank();
2007:     debugLog.integers["world_size"] = pg_->getSize();
2008:     debugLog.strings["flight_recorder_version"] = c10d::version_val_str;
2009:     for (int i = 0; i < 2; i++) {
2010:       std::future<bool> asyncDebugDump =
```

- EN: Lines 1981-2010 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1981-2010 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2011-2040 / 第 2011-2040 行

```cpp
2011:           std::async(std::launch::async, [this, dumpStackTrace, onlyActive]() {
2012:             return this->pg_->dumpDebuggingInfo(dumpStackTrace, onlyActive);
2013:           });
2014: 
2015:       // wait for the dump until timeout - log data
2016:       auto complete = pg_->waitForFutureOrTimeout(
2017:           asyncDebugDump,
2018:           std::chrono::milliseconds(waitTimeoutDumpInMilSec_),
2019:           "Flight recorder dump in heartbeatMonitor",
2020:           debugLog,
2021:           false);
2022: 
2023:       if (complete) {
2024:         LOG(INFO)
2025:             << pg_->logPrefix()
2026:             << "Finished flight recorder successfully. Output can be analyzed using the fr_trace script.";
2027:         if (i > 0) {
2028:           debugLog.strings["exception_msg"] = "Dump with stack trace failed.";
2029:         }
2030:         break;
2031:       }
2032:       // If we failed to dump, try dumping without stack trace in the 2nd
2033:       // iteration.
2034:       dumpStackTrace = false;
2035:       futures.emplace_back(std::move(asyncDebugDump));
2036:     }
2037:     debugLog.integers["trace_enabled"] = int64_t(dumpStackTrace);
2038:     auto logger = c10d::C10dLogger::getLogger();
2039:     if (logger) {
2040:       logger->log(debugLog);
```

- EN: Lines 2011-2040 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2011-2040 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2041-2070 / 第 2041-2070 行

```cpp
2041:     }
2042:   }
2043: 
2044:   // GIL deadlock check.
2045:   if (get_gil_checker() != nullptr) {
2046:     auto fut = launchAsyncGilCheck();
2047:     auto kGilCheckTimeout = std::chrono::milliseconds(300);
2048:     auto futStatus = fut.wait_for(kGilCheckTimeout);
2049:     if (futStatus != std::future_status::ready) {
2050:       TORCH_CHECK(
2051:           futStatus != std::future_status::deferred,
2052:           "Expected the future to have been launched eagerly.");
2053:       LOG(ERROR)
2054:           << pg_->logPrefix()
2055:           << "Could not acquire GIL within 300 ms on exit, possible GIL induced hang";
2056:     }
2057:   } else {
2058:     VLOG(2)
2059:         << pg_->logPrefix()
2060:         << "GIL checker was not registered, perhaps this is a no-python build?";
2061:   }
2062: 
2063:   // Dump the c++ stacktraces.
2064:   auto& cpp_dumper = get_cpp_trace_dumper();
2065:   if (logCppStackOnUncleanShutdown_ && cpp_dumper.has_value()) {
2066:     LOG(INFO) << pg_->logPrefix() << "Dumping c++ stacktraces:";
2067:     cpp_dumper.value()([&](const std::string& line) {
2068:       LOG(INFO) << pg_->logPrefix() << line;
2069:     });
2070:     LOG(INFO) << pg_->logPrefix() << "Finished c++ stacktraces dump.";
```

- EN: Lines 2041-2070 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 2041-2070 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 2071-2100 / 第 2071-2100 行

```cpp
2071:   }
2072: 
2073:   // There are two possible cases for the watchdog thread exit:
2074:   // Case one: desync report runs quickly, and it follows the step:
2075:   // collective timeout -> desync -> exception handling -> throwing exception.
2076:   // The program will exit because of exception thrown and the code below will
2077:   // not be run.
2078:   //
2079:   // Case two: desync might be slow or get stuck and we need to wait
2080:   // extra time to avoid we kill the program too early.
2081:   //
2082:   // Or we get stuck in destructors, we will sleep for some time before calling
2083:   // std::abort() to kill the whole process.
2084:   if (pg_->terminateProcessGroup_.load() || shouldDump_.load()) {
2085:     for (int t = 0; t < heartbeatTimeoutInSec_; ++t) {
2086:       if (terminateHeartbeatMonitorThread_.load()) {
2087:         if (t > 0)
2088:           LOG(INFO)
2089:               << pg_->logPrefix() << "slept for " << t
2090:               << " seconds because we want to wait longer to verify there is indeed a watchdog hang.";
2091:         break;
2092:       }
2093:       std::this_thread::sleep_for(std::chrono::seconds(1));
2094:     }
2095:   }
2096: 
2097:   // At this point, we either already sleep for another `heartbeatTimeoutInSec_`
2098:   // or the thread has finished. Because we don't want to block the monitor
2099:   // thread, so We mark the thread detach and the dump of debug info becomes
2100:   // "best effort". If the process exit normally, marking it detach also makes
```

- EN: Lines 2071-2100 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2071-2100 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2101-2130 / 第 2101-2130 行

```cpp
2101:   // sense because we don't really care about dumping the debug info.
2102: 
2103:   // We already log completion inside the thread, so it may not be necessary to
2104:   // check the return value here.  We mainly use a future so we can exit early
2105:   // if done.
2106:   if (!terminateHeartbeatMonitorThread_.load()) {
2107:     // Create a error message reported from MonitorThread, so
2108:     // we throw exception and make the whole process to be killed.
2109:     // TODO(fduwjj): After having a hang debug wiki, we need to update the wiki
2110:     // url here.
2111:     if (watchdogHeartbeatMonitorEnabled_) {
2112:       pg_->terminateProcess(getNCCLWatchdogTimeoutExitMsg(exitReason));
2113:     } else {
2114:       // Ideally we want to merge this one with the above one, but we are going
2115:       // to remove the kill switch for monitor thread soon, so we keep this one
2116:       // for now.
2117:       LOG(ERROR)
2118:           << pg_->logPrefix()
2119:           << "ProcessGroupNCCL monitor thread is disabled, but would have terminated the process"
2120:           << "after attempting to dump debug info, due to " << exitReason
2121:           << '.';
2122:     }
2123:   }
2124: }
2125: 
2126: ProcessGroupNCCL::Watchdog::Watchdog(ProcessGroupNCCL* pg) {
2127:   pg_ = pg;
2128:   heartbeat_ = 1ULL;
2129:   rethrowCUDAErrors_ = getCvarBool(TORCH_NCCL_RETHROW_CUDA_ERRORS, true);
2130:   propagatePgError_ = getCvarBool(TORCH_NCCL_PROPAGATE_ERROR, false);
```

- EN: Lines 2101-2130 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2101-2130 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2131-2160 / 第 2131-2160 行

```cpp
2131:   desyncDebug_ = getCvarBool(TORCH_NCCL_DESYNC_DEBUG, false) ||
2132:       (pg_->dist_debug_level_ >= DebugLevel::Detail);
2133: 
2134:   // print out ENV settings for the watchdog thread.
2135:   if (pg_->getUid() == 0) {
2136:     LOG(INFO) << pg_->logPrefix() << "PGNCCL Watchdog environments: "
2137:               << "TORCH_NCCL_RETHROW_CUDA_ERRORS: " << rethrowCUDAErrors_
2138:               << ", TORCH_NCCL_PROPAGATE_ERROR: " << propagatePgError_
2139:               << ", TORCH_NCCL_DESYNC_DEBUG: " << desyncDebug_;
2140:   }
2141: 
2142:   // Enable Desync Debugger per user setting
2143:   if (desyncDebug_) {
2144:     desyncDebugger_.init(
2145:         pg_->getRank(),
2146:         pg_->getSize(),
2147:         pg_->globalRank(),
2148:         pg_->getUid(),
2149:         pg_->store_);
2150:   }
2151: }
2152: 
2153: void ProcessGroupNCCL::Watchdog::notify() {
2154:   workMetaListCV_.notify_one();
2155: }
2156: 
2157: void ProcessGroupNCCL::Watchdog::start() {
2158:   TORCH_CHECK(
2159:       !ncclCommWatchdogThread_.joinable(), "Watchdog thread already started");
2160:   ncclCommWatchdogThread_ = std::thread(&ProcessGroupNCCL::Watchdog::run, this);
```

- EN: Lines 2131-2160 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2131-2160 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2161-2190 / 第 2161-2190 行

```cpp
2161: }
2162: 
2163: void ProcessGroupNCCL::Watchdog::join() {
2164:   if (ncclCommWatchdogThread_.joinable()) {
2165:     ncclCommWatchdogThread_.join();
2166:     LOG(INFO) << pg_->logPrefix() << "ProcessGroupNCCL watchdog thread joined.";
2167:   }
2168: }
2169: 
2170: void ProcessGroupNCCL::Watchdog::run() {
2171:   c10::setThreadName("pt_nccl_watchdg");
2172:   STATIC_SCOPED_WAIT_COUNTER(pytorch.ProcessGroupNCCL__Watchdog__run);
2173: 
2174:   try {
2175:     VLOG(2) << pg_->logPrefix() << "Process group watchdog thread started!";
2176:     pg_->heartbeatMonitor_->start();
2177:     runLoop();
2178:     VLOG(2) << pg_->logPrefix()
2179:             << "Process group watchdog thread terminated normally";
2180:   } catch (std::exception& e) {
2181:     // This condition is triggered when any routine in watchdog gets an
2182:     // exception
2183:     pg_->dumpExtraDebuggingInfo();
2184:     if (std::string(e.what()).find("driver shutting down") !=
2185:         std::string::npos) {
2186:       VLOG(2)
2187:           << pg_->logPrefix()
2188:           << "main process destroyed cuda before watchdog loop exited, terminating watchdog."
2189:           << " (Watchdog caught exception: " << e.what();
2190: 
```

- EN: Lines 2161-2190 introduces executable logic in routines such as `runLoop`.
- CN: 第 2161-2190 行在 `runLoop` 等例程中引入具体执行逻辑。

### Lines 2191-2220 / 第 2191-2220 行

```cpp
2191:     } else {
2192:       // Append error message reported from runLoop
2193:       const auto exitMsg = c10::str(
2194:           pg_->logPrefix(),
2195:           "Process group watchdog thread terminated with exception: ",
2196:           e.what());
2197:       LOG(ERROR) << exitMsg;
2198:       if (C10_LIKELY(rethrowCUDAErrors_) ||
2199:           std::string(e.what()).find("CUDA Error") != std::string::npos) {
2200:         // TODO(whc) clean up the rethrow - why is it stored in a class var
2201:         // and rethrown?
2202:         watchDogException_ =
2203:             std::make_exception_ptr(C10_BUILD_ERROR(DistBackendError, exitMsg));
2204:         std::rethrow_exception(watchDogException_);
2205:       }
2206:     }
2207:   } catch (...) {
2208:     const auto exitMsg = c10::str(
2209:         pg_->logPrefix(),
2210:         "Process group watchdog thread terminated with exception: unknown");
2211:     LOG(ERROR) << exitMsg;
2212:     watchDogException_ =
2213:         std::make_exception_ptr(C10_BUILD_ERROR(DistBackendError, exitMsg));
2214:     std::rethrow_exception(watchDogException_);
2215:   }
2216: }
2217: 
2218: int ProcessGroupNCCL::Watchdog::getSignalSrcRank(
2219:     c10::intrusive_ptr<Store>& store,
2220:     const std::string& signal) {
```

- EN: Lines 2191-2220 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2191-2220 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2221-2250 / 第 2221-2250 行

```cpp
2221:   // This function is 'non blocking'. We first 'check' if the key exists in the
2222:   // store, then read/get the value only if the key exists.
2223:   int srcRank = -1;
2224:   bool signalExists = false;
2225:   try {
2226:     signalExists = store->check({signal});
2227:   } catch (const std::exception& e) {
2228:     LOG(WARNING) << pg_->logPrefix() << "Failed to check the signal " << signal
2229:                  << " on TCPStore, " << e.what();
2230:   }
2231:   if (!signalExists) {
2232:     return srcRank;
2233:   }
2234: 
2235:   // key exists, now read and parse the value (source rank)
2236:   std::vector<uint8_t> vec;
2237:   try {
2238:     vec = store->get(std::string(signal));
2239:   } catch (const std::exception& e) {
2240:     LOG(ERROR) << pg_->logPrefix() << "Failed to get source rank of the signal "
2241:                << signal << " from TCPStore." << e.what();
2242:   }
2243:   TORCH_CHECK_WITH(
2244:       DistBackendError,
2245:       vec.size() == sizeof(int),
2246:       "Invalid size for the timeout rank ID");
2247:   std::memcpy(&srcRank, vec.data(), vec.size());
2248:   return srcRank;
2249: }
2250: 
```

- EN: Lines 2221-2250 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2221-2250 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2251-2280 / 第 2251-2280 行

```cpp
2251: void ProcessGroupNCCL::Watchdog::checkAndSetRemoteError() {
2252:   // if the error is already set, no need to check again
2253:   if (pg_->getError() != ErrorType::SUCCESS) {
2254:     return;
2255:   }
2256:   // key/signal to read from the tcpstore is a string and pg specific:
2257:   // format is: remote_error:pg_uid
2258:   int remoteErrorRank = getSignalSrcRank(
2259:       pg_->store_, std::string(kStoreErrorSignalKey) + ':' + pg_->pg_uid_);
2260:   if (remoteErrorRank != -1) {
2261:     std::lock_guard<std::mutex> lock(pg_->errorMutex_);
2262:     pg_->error_ = ErrorType::REMOTE_ERROR;
2263:     LOG(ERROR) << c10::str(
2264:         pg_->logPrefix(),
2265:         " remote error detected from rank: ",
2266:         remoteErrorRank);
2267:   }
2268: }
2269: 
2270: void ProcessGroupNCCL::Watchdog::runLoop() {
2271:   bool done = false;
2272:   pg_->heartbeatMonitor_->setLastWorkListUpdateTime(
2273:       std::chrono::steady_clock::now());
2274:   auto lastStatusUpdateTime = std::chrono::steady_clock::now();
2275:   std::list<ProcessGroupNCCL::WorkNCCL> completedWorkList;
2276: 
2277:   while (!done || !pg_->terminateProcessGroup_.load()) {
2278:     std::unique_lock<std::mutex> lock(pg_->workMetaListMutex_);
2279:     // We busy-poll the work vector every kWatchdogThreadSleepMillis
2280:     // milliseconds as long as the atomic is True.
```

- EN: Lines 2251-2280 introduces executable logic in routines such as `lock`; returns computed state or forwards results to the surrounding caller.
- CN: 第 2251-2280 行在 `lock` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2281-2310 / 第 2281-2310 行

```cpp
2281:     workMetaListCV_.wait_for(
2282:         lock,
2283:         std::chrono::milliseconds(kWatchdogThreadSleepMillis),
2284:         [&]() -> bool { return pg_->terminateProcessGroup_.load(); });
2285:     // Bump up heart beat by one.
2286:     heartbeat_++;
2287: 
2288: // Some versions of GLOG support less-spammy version of LOG_EVERY_MS
2289: // in which case we don't want to spam the logs.
2290: #ifdef LOG_EVERY_MS
2291:     // Log the progress of this PG periodically
2292:     C10_LOG_EVERY_MS(INFO, kWorkStatusUpdatePeriodMs) << c10::str(
2293:         logPrefix(),
2294:         "NCCL Work update periodically: ",
2295:         "last enqueued NCCL work: ",
2296:         pg_->pgStatus_->lastEnqueuedSeq,
2297:         ", last completed NCCL work: ",
2298:         pg_->pgStatus_->lastCompletedSeq,
2299:         ".");
2300: #endif // LOG_EVERY_MS
2301:     auto logger = ::c10d::C10dLogger::getLogger();
2302:     if (logger &&
2303:         computeDeltaMS(
2304:             lastStatusUpdateTime, std::chrono::steady_clock::now()) >=
2305:             kWorkStatusUpdatePeriodMs) {
2306:       ::c10d::C10dLoggingData data;
2307:       // logging integers
2308:       data.integers["pg_id"] = static_cast<int64_t>(pg_->local_id_);
2309:       data.integers["rank"] = pg_->rank_;
2310:       data.integers["global_rank"] = pg_->globalRank();
```

- EN: Lines 2281-2310 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2281-2310 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2311-2340 / 第 2311-2340 行

```cpp
2311:       data.integers["last_enqueued_work"] = pg_->pgStatus_->lastEnqueuedSeq;
2312:       data.integers["last_started_work"] = pg_->pgStatus_->lastStartedSeq;
2313:       data.integers["last_completed_work"] = pg_->pgStatus_->lastCompletedSeq;
2314:       data.integers["last_enqueued_numel_in"] =
2315:           static_cast<int64_t>(pg_->pgStatus_->lastEnqueuedNumelIn);
2316:       data.integers["last_enqueued_numel_out"] =
2317:           static_cast<int64_t>(pg_->pgStatus_->lastEnqueuedNumelOut);
2318:       data.integers["last_completed_numel_in"] =
2319:           static_cast<int64_t>(pg_->pgStatus_->lastCompletedNumelIn);
2320:       data.integers["last_completed_numel_out"] =
2321:           static_cast<int64_t>(pg_->pgStatus_->lastCompletedNumelOut);
2322:       data.integers["last_started_numel_in"] =
2323:           static_cast<int64_t>(pg_->pgStatus_->lastStartedNumelIn);
2324:       data.integers["last_started_numel_out"] =
2325:           static_cast<int64_t>(pg_->pgStatus_->lastStartedNumelOut);
2326:       // logging strings
2327:       data.strings["last_enqueued_work_name"] =
2328:           pg_->pgStatus_->lastEnqueuedWorkName;
2329:       data.strings["last_started_work_name"] =
2330:           pg_->pgStatus_->lastStartedWorkName;
2331:       data.strings["last_completed_work_name"] =
2332:           pg_->pgStatus_->lastCompletedWorkName;
2333:       data.strings["pg_name"] = pg_->pg_uid_;
2334:       data.strings["pg_desc"] = pg_->pg_desc_;
2335:       logger->log(data);
2336:       lastStatusUpdateTime = std::chrono::steady_clock::now();
2337:     }
2338: 
2339:     if (propagatePgError_) {
2340:       // Check and set remote error if it has not been set before
```

- EN: Lines 2311-2340 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2311-2340 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2341-2370 / 第 2341-2370 行

```cpp
2341:       checkAndSetRemoteError();
2342:     }
2343: 
2344:     for (auto it = pg_->workMetaList_.begin(); it != pg_->workMetaList_.end();
2345:          /* no increment */) {
2346:       auto& work = *it;
2347:       // When terminateProcessGroup_ is true, communicators have already been
2348:       // aborted, So cannot check exception based on them. But watchdog needs to
2349:       // finish the check for the works that have already been enqueued to
2350:       // workMetaList_
2351: 
2352:       // check NCCL errors first
2353:       if (!pg_->terminateProcessGroup_.load()) {
2354:         work.checkAndSetException();
2355:       }
2356: 
2357:       if (work.exception()) {
2358:         // set the error to the first error found
2359:         std::lock_guard<std::mutex> lock(pg_->errorMutex_);
2360:         if (pg_->error_ == ErrorType::SUCCESS) {
2361:           pg_->error_ = ErrorType::COMM_ERROR;
2362:         }
2363:       }
2364: 
2365:       // Then check if work has timed out.
2366:       // Skip if work has encountered an error.
2367: 
2368:       bool timedout = false;
2369: #ifdef USE_ROCM
2370:       // On ROCm, watchdog event queries may be intentionally skipped during
```

- EN: Lines 2341-2370 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `checkAndSetRemoteError`, `lock`.
- CN: 第 2341-2370 行使用条件编译来适配特性开关、平台或可选后端；在 `checkAndSetRemoteError`、`lock` 等例程中引入具体执行逻辑。

### Lines 2371-2400 / 第 2371-2400 行

```cpp
2371:       // active graph capture to avoid HIP runtime capture invalidation.
2372:       // In that window, timeout checks can report false positives for
2373:       // otherwise-complete work, so we defer timeout enforcement.
2374:       // TODO: Remove once all supported HIP runtimes include:
2375:       // https://github.com/ROCm/clr/pull/3176
2376:       if (!at::cuda::is_graph_capture_active()) {
2377:         timedout = !work.exception() && work.checkTimeout();
2378:       }
2379: #else
2380:       timedout = !work.exception() && work.checkTimeout();
2381: #endif
2382: 
2383:       // Report desync state in case of timeout (if TORCH_NCCL_DESYNC_DEBUG is
2384:       // turned on; otherwise, run() is no-op)
2385:       if (timedout) {
2386:         std::lock_guard<std::mutex> lock(pg_->errorMutex_);
2387:         if (pg_->error_ == ErrorType::SUCCESS) {
2388:           pg_->error_ = ErrorType::TIMEOUT;
2389:         }
2390:         desyncDebugger_.run();
2391:       }
2392: 
2393:       // If work hits an exception (either an error or timeout)
2394:       if (work.exception()) {
2395:         LOG(ERROR) << c10::str(
2396:             pg_->logPrefix(),
2397:             " failure detected by watchdog at work sequence id: ",
2398:             work.seq_,
2399:             " PG status: last enqueued work: ",
2400:             pg_->pgStatus_->lastEnqueuedSeq,
```

- EN: Lines 2371-2400 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `lock`.
- CN: 第 2371-2400 行使用条件编译来适配特性开关、平台或可选后端；在 `lock` 等例程中引入具体执行逻辑。

### Lines 2401-2430 / 第 2401-2430 行

```cpp
2401:             ", last completed work: ",
2402:             pg_->pgStatus_->lastCompletedSeq);
2403: 
2404:         // Print the traceback of the collective at call time
2405:         work.printTraceback();
2406: 
2407:         // broadcast remote error signal to all other ranks in this specific PG.
2408:         // key/signal to write in the tcpstore is a string and pg specific:
2409:         // format is: remote_error:pg_uid
2410:         if (propagatePgError_) {
2411:           pg_->broadcastSignal(
2412:               pg_->store_,
2413:               std::string(kStoreErrorSignalKey) + ':' + pg_->pg_uid_,
2414:               pg_->rank_);
2415:         }
2416: 
2417:         // try to notify other ranks via global TCPStore to dump the flight
2418:         // recorder when a collective timeout or exception happens. Flight
2419:         // recorder behavior is independent of desync Debug.
2420:         pg_->broadcastDumpSignal();
2421:         // Give time for dumping before throwing exception for all ranks.
2422:         // It is hard to presume or control what the pattern of watchdog might
2423:         // look like, so it is better to let all ranks universally sleep for a
2424:         // short period of time, in this case, 60 seconds, which is also the
2425:         // maximum time we leave for FR dump.
2426:         std::this_thread::sleep_for(std::chrono::milliseconds(
2427:             pg_->heartbeatMonitor_->getDumpTimeout() * 4));
2428: 
2429:         if (SHOULD_CLEAN_UP(pg_->asyncErrorHandling_)) {
2430:           // Abort work and corresponding communicators
```

- EN: Lines 2401-2430 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2401-2430 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2431-2460 / 第 2431-2460 行

```cpp
2431:           work.abort();
2432:           // PG level abort, which would abort all other communicators on this
2433:           // rank
2434:           pg_->abortComms();
2435:         }
2436:         // Throw exception
2437:         work.handleException(pg_->asyncErrorHandling_);
2438:       }
2439: 
2440:       // Work status logging for desync debug
2441:       desyncDebugger_.logWorkStart(work);
2442: 
2443:       // allow watchdog to do an event query on a side thread
2444:       at::cuda::CUDAGuard device_guard(work.ncclEndEvent_->device_index());
2445:       at::cuda::CUDAStreamCaptureModeGuard g{cudaStreamCaptureModeThreadLocal};
2446: #ifdef USE_ROCM
2447:       // Mark this thread/scope as watchdog event-query context so the ROCm
2448:       // workaround applies only here (not to main-thread wait()/isCompleted()).
2449:       RocmWatchdogEventQueryContextGuard watchdog_event_query_context_guard;
2450: #endif
2451:       // a work could be started but not completed, so we should not update
2452:       // lastStartedSeq and lastStartedOpName if the work state is checked
2453:       // multiple times after the start
2454:       if (pg_->pgStatus_->lastStartedSeq < static_cast<int64_t>(work.seq_) &&
2455:           work.isStarted()) {
2456:         pg_->pgStatus_->lastStartedSeq = static_cast<int64_t>(work.seq_);
2457:         pg_->pgStatus_->lastStartedWorkName = opTypeToString(work.opType_);
2458:         pg_->pgStatus_->lastStartedNumelIn = work.numelIn_;
2459:         pg_->pgStatus_->lastStartedNumelOut = work.numelOut_;
2460:       }
```

- EN: Lines 2431-2460 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2431-2460 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2461-2490 / 第 2461-2490 行

```cpp
2461: 
2462:       // Clean up completed work
2463:       if (work.isCompleted()) {
2464:         // In case user didn't call `work.wait()` with async collectives,
2465:         // watchdog would unstage the stashed tensors when detecting completion
2466:         // of the collective, to prevent ProcessGroupNCCL from holding reference
2467:         // to those tensors forever.
2468:         // work.stashed_for_allocator_safety_->unstash();
2469:         // Update: it seems directly unstashing from watchdog thread would cause
2470:         // some rare problems. We thus move the unstashing to main thread,
2471:         // triggered by a next user call, see `workEnqueue`. But `work` is going
2472:         // to be destructed, so we transfer the work's shelf to a shelves
2473:         // structure owned by the PG.
2474:         if (!work.stashed_for_allocator_safety_->empty()) {
2475:           std::lock_guard<std::mutex> lock(pg_->shelvesMutex_);
2476:           // We are just pushing back a shared_ptr here, so the cost should be
2477:           // minimal
2478:           pg_->shelvesToUnstash_.push_back(work.stashed_for_allocator_safety_);
2479:         }
2480: 
2481:         if (pg_->enableTiming_ && logger) {
2482:           ::c10d::C10dLoggingData data;
2483:           // logging integers
2484:           data.strings["collective_duration"] =
2485:               std::to_string(work.getDuration());
2486:           data.integers["global_rank"] = pg_->globalRank();
2487:           data.integers["pg_id"] = static_cast<int64_t>(pg_->local_id_);
2488:           data.strings["pg_name"] = pg_->pg_uid_;
2489:           data.strings["pg_desc"] = pg_->pg_desc_;
2490:           data.integers["pg_rank"] = pg_->rank_;
```

- EN: Lines 2461-2490 introduces executable logic in routines such as `lock`.
- CN: 第 2461-2490 行在 `lock` 等例程中引入具体执行逻辑。

### Lines 2491-2520 / 第 2491-2520 行

```cpp
2491:           data.integers["world_size"] = pg_->size_;
2492:           data.strings["comm_backend"] = "nccl";
2493:           data.strings["comm_backend_version"] = getNcclVersion();
2494:           // TODO: We see errors for this line, revert it for now.
2495:           data.strings["collective_stack"] = "";
2496:           data.strings["collective_name"] = opTypeToString(work.opType_);
2497:           logger->log(data);
2498:         }
2499: 
2500:         // Work status logging for desync debug
2501:         desyncDebugger_.logWorkEnd(work);
2502: 
2503:         if (work.futureWorkResult_ && work.finishedGPUExecutionInternal() &&
2504:             !work.futureWorkResult_->completed()) {
2505:           work.futureWorkResult_->markCompleted(
2506:               at::IValue(static_cast<uint8_t>(WorkResult::SUCCESS)));
2507:         }
2508:         {
2509:           // Reset the timeout and first work if the work is completed.
2510:           std::lock_guard<std::mutex> timeoutLock(pg_->mtxTimeoutExtension_);
2511:           if (work.ownedEphermeralTimeout_.count() > 0) {
2512:             pg_->ephemeralTimeoutActive_ -= work.ownedEphermeralTimeout_;
2513:             pg_->ephemeralTimeoutInflight_ -= work.ownedEphermeralTimeout_;
2514:           }
2515:         }
2516:         pg_->pgStatus_->lastCompletedSeq = static_cast<int64_t>(work.seq_);
2517:         pg_->pgStatus_->lastCompletedWorkName = opTypeToString(work.opType_);
2518:         pg_->pgStatus_->lastCompletedNumelIn = work.numelIn_;
2519:         pg_->pgStatus_->lastCompletedNumelOut = work.numelOut_;
2520:         FlightRecorderCUDA::get()->retire_id(
```

- EN: Lines 2491-2520 introduces executable logic in routines such as `timeoutLock`.
- CN: 第 2491-2520 行在 `timeoutLock` 等例程中引入具体执行逻辑。

### Lines 2521-2550 / 第 2521-2550 行

```cpp
2521:             work.trace_id_, work.trace_reset_epoch_, true);
2522:         if (pg_->onCompletionHook_) {
2523:           // Move Work object to completedWorkList_ to be consumed by the hook
2524:           // thread
2525:           {
2526:             const std::lock_guard<std::mutex> lock(
2527:                 pg_->completedWorkListMutex_);
2528:             pg_->completedWorkList_.splice(
2529:                 pg_->completedWorkList_.end(), pg_->workMetaList_, it++);
2530:           }
2531:           pg_->completedWorkListCV_.notify_one();
2532:         } else {
2533:           it = pg_->workMetaList_.erase(it);
2534:           pg_->heartbeatMonitor_->setLastWorkListUpdateTime(
2535:               std::chrono::steady_clock::now());
2536:         }
2537:       } else {
2538:         // Increment the iterator if the current WorkNCCL object is not
2539:         // completed.
2540:         ++it;
2541:       }
2542:       // Increment heartbeat after each work processed,
2543:       // in case processing is slowed down (but not hung) by cuda api contention
2544:       heartbeat_++;
2545:     }
2546:     done = pg_->workMetaList_.empty();
2547:   }
2548: }
2549: 
2550: uint64_t ProcessGroupNCCL::Watchdog::getHeartbt() const {
```

- EN: Lines 2521-2550 introduces executable logic in routines such as `lock`.
- CN: 第 2521-2550 行在 `lock` 等例程中引入具体执行逻辑。

### Lines 2551-2580 / 第 2551-2580 行

```cpp
2551:   return heartbeat_.load();
2552: }
2553: 
2554: void ProcessGroupNCCL::Watchdog::setDesyncDebug(bool desyncDebug) {
2555:   desyncDebug_ = desyncDebug;
2556: }
2557: 
2558: // Initialize and enable DesyncDebugger
2559: void ProcessGroupNCCL::DesyncDebugger::init(
2560:     int rank,
2561:     int size,
2562:     int globalRank,
2563:     int pgId,
2564:     c10::intrusive_ptr<Store> store) {
2565:   rank_ = rank;
2566:   size_ = size;
2567:   globalRank_ = globalRank;
2568:   pgId_ = pgId;
2569:   store_ = std::move(store);
2570:   enabled_ = true;
2571:   traceKeyStart_ = getTraceStartKey("NCCL", rank);
2572:   traceKeyEnd_ = getTraceEndKey("NCCL", rank);
2573: }
2574: 
2575: // Run desync debug. This function is called by watchdog at time of timeout.
2576: void ProcessGroupNCCL::DesyncDebugger::run() {
2577:   if (!enabled_)
2578:     return;
2579:   auto logPrefix = c10::str("Rank ", rank_);
2580:   ::c10d::C10dLoggingData log;
```

- EN: Lines 2551-2580 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2551-2580 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2581-2610 / 第 2581-2610 行

```cpp
2581:   log.integers["pg_id"] = pgId_;
2582:   log.integers["rank"] = rank_;
2583:   log.integers["global_rank"] = globalRank_;
2584:   log.integers["world_size"] = size_;
2585:   // Use this to differentiate between flight recorder and desync debug report.
2586:   log.strings["flight_recorder_version"] = "-1";
2587: 
2588:   try {
2589:     std::string desyncMsg = retrieveDesyncReport(store_, "NCCL", rank_, size_);
2590:     log.strings["status"] = "SUCCESS";
2591:     LOG(ERROR) << logPrefix << desyncMsg;
2592:   } catch (const std::exception& e) {
2593:     log.strings["status"] = "EXCEPTION";
2594:     log.strings["exception_msg"] = e.what();
2595:     enabled_ = false;
2596:     LOG(ERROR) << logPrefix
2597:                << " Failed to retrieve TORCH_NCCL_DESYNC_DEBUG report. "
2598:                << " Please file an issue. Error: " << e.what();
2599:   } catch (...) {
2600:     enabled_ = false;
2601:     log.strings["status"] = "EXCEPTION";
2602:     log.strings["exception_msg"] = "Unknown exception";
2603:     LOG(ERROR)
2604:         << logPrefix
2605:         << " Failed to rerieve TORCH_NCCL_DESYNC_DEBUG report with unknown error."
2606:         << " Please file an issue.";
2607:   }
2608:   auto logger = c10d::C10dLogger::getLogger();
2609:   if (logger) {
2610:     logger->log(log);
```

- EN: Lines 2581-2610 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2581-2610 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2611-2640 / 第 2611-2640 行

```cpp
2611:   }
2612: }
2613: 
2614: // Log work start to store.
2615: void ProcessGroupNCCL::DesyncDebugger::logWorkStart(WorkNCCL& work) {
2616:   if (!enabled_)
2617:     return;
2618:   if (work.startTraceUpdated_)
2619:     return;
2620: 
2621:   work.startTraceUpdated_ = true;
2622:   // If not successful, disable the debugger
2623:   enabled_ = c10d::traceUpdate(
2624:       store_, traceKeyStart_, work.seq_, opTypeToString(work.opType_));
2625: }
2626: 
2627: // Log work end to store.
2628: void ProcessGroupNCCL::DesyncDebugger::logWorkEnd(WorkNCCL& work) {
2629:   if (!enabled_)
2630:     return;
2631: 
2632:   // In case the start of the work hasn't been logged
2633:   if (!work.startTraceUpdated_) {
2634:     logWorkStart(work);
2635:   }
2636: 
2637:   // If not successful, disable the debugger
2638:   enabled_ = c10d::traceUpdate(
2639:       store_, traceKeyEnd_, work.seq_, opTypeToString(work.opType_));
2640: }
```

- EN: Lines 2611-2640 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2611-2640 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2641-2670 / 第 2641-2670 行

```cpp
2641: 
2642: // We want to have both PG ID and global unique ID (guid) for the logging
2643: // prefix. PG ID records how many ProcessGroupNCCL objects were created on a
2644: // specific rank and is a stable index across ranks, which lets users reason
2645: // about, for example, the second PG we initialized on this rank is for FSDP,
2646: // and corresponds with PG ID = 1 on other ranks as well. Unlike PG ID, guid (or
2647: // group name) is a global unique ID across ranks. The guid is either a hash of
2648: // all the ranks in the group or a counter of how many times
2649: // `_process_group_name` is called, essentially it means how many times we
2650: // have PGs users have created. Before using split_group, even if
2651: // we are creating a new sub-PG, all ranks have to call the API at the same
2652: // time, and this makes `group_name` a unique identifier for a group (PG).
2653: std::string ProcessGroupNCCL::createLogPrefix() const {
2654:   if (!pg_desc_.empty() && pg_desc_ != "undefined") {
2655:     return c10::str(
2656:         "[PG ID ",
2657:         local_id_,
2658:         " PG GUID ",
2659:         pg_uid_,
2660:         "(",
2661:         pg_desc_,
2662:         ") Rank ",
2663:         rank_,
2664:         "] ");
2665:   }
2666:   return c10::str(
2667:       "[PG ID ", local_id_, " PG GUID ", pg_uid_, " Rank ", rank_, "] ");
2668: }
2669: 
2670: const std::string& ProcessGroupNCCL::logPrefix() const {
```

- EN: Lines 2641-2670 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2641-2670 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2671-2700 / 第 2671-2700 行

```cpp
2671:   return logPrefix_;
2672: }
2673: 
2674: const int& ProcessGroupNCCL::globalRank() const {
2675:   static int globalRank = rank_;
2676:   return globalRank;
2677: }
2678: 
2679: const c10::intrusive_ptr<Store>& ProcessGroupNCCL::globalStore() const {
2680:   return globalStore_;
2681: }
2682: 
2683: const std::vector<uint64_t>& ProcessGroupNCCL::groupRanks() const {
2684:   if (options_->global_ranks_in_group.empty() && local_id_ == 0) {
2685:     static std::vector<uint64_t> globalRanks(size_);
2686:     std::iota(globalRanks.begin(), globalRanks.end(), 0);
2687:     return globalRanks;
2688:   }
2689:   return options_->global_ranks_in_group;
2690: }
2691: 
2692: void ProcessGroupNCCL::addEphemeralTimeout(
2693:     const std::chrono::milliseconds& timeout) {
2694:   std::lock_guard<std::mutex> timeoutLock(mtxTimeoutExtension_);
2695:   ephemeralTimeoutActive_ += timeout;
2696: }
2697: 
2698: bool ProcessGroupNCCL::verifyWorkTimeoutForTest(
2699:     const c10::intrusive_ptr<Work>& work,
2700:     const std::chrono::milliseconds& timeout) {
```

- EN: Lines 2671-2700 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2671-2700 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2701-2730 / 第 2701-2730 行

```cpp
2701:   // Since collective returns a c10d::Work, we need to cast it to WorkNCCL.
2702:   if (auto workNCCL = c10::dynamic_intrusive_pointer_cast<WorkNCCL>(work)) {
2703:     // workNCCL is now a c10::intrusive_ptr<WorkNCCL>
2704:     return workNCCL->opTimeout_ == timeout;
2705:   }
2706:   C10_THROW_ERROR(
2707:       DistBackendError, "Non c10d::WorkNCCL object returned from collective");
2708: }
2709: 
2710: void ProcessGroupNCCL::broadcastSignal(
2711:     c10::intrusive_ptr<Store>& store,
2712:     const std::string& signal,
2713:     int srcRank) {
2714:   try {
2715:     auto vec = std::vector<uint8_t>(
2716:         reinterpret_cast<uint8_t*>(&srcRank),
2717:         reinterpret_cast<uint8_t*>(&srcRank) + sizeof(srcRank));
2718:     store->set(signal, vec);
2719:     LOG(INFO) << logPrefix() << "Broadcasting signal " << signal
2720:               << " to other ranks via TCPStore.";
2721:   } catch (const std::exception& e) {
2722:     LOG(ERROR) << logPrefix() << "Failed to broadcast signal " << signal
2723:                << " through TCPStore. Error: " << e.what();
2724:   }
2725: }
2726: 
2727: void ProcessGroupNCCL::broadcastDumpSignal() {
2728:   // broadcast dump signal to all other global ranks.
2729:   broadcastSignal(globalStore_, std::string(kStoreDumpKey), globalRank());
2730:   // signal the local rank to start dumping
```

- EN: Lines 2701-2730 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 2701-2730 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 2731-2760 / 第 2731-2760 行

```cpp
2731:   if (!shouldDump_.load()) {
2732:     LOG(ERROR) << logPrefix() << "First PG on this rank to signal dumping.";
2733:     // signal the monitor thread on PG0 to start dumping
2734:     shouldDump_.store(true);
2735:   }
2736: }
2737: 
2738: // NCCL recommends to evenly distribute ncclUniqueIds across the ranks
2739: // https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/communicators.html#init-rank-config
2740: // Let’s consider an example where:
2741: // nRanks = 10 (total ranks),
2742: // nIds = 3 (roots),
2743: // rmr = 10 % 3 = 1 (1 larger group),
2744: // rpr = 10 / 3 = 3 (base number of ranks per group).
2745: // rlim = 4
2746: // Output root:
2747: // For ranks [0, 1, 2, 3], root rank is 0 and index is 0.
2748: // For ranks [4, 5, 6], root rank is 4 and index is 1.
2749: // For ranks [7, 8, 9], root rank is 7 and index is 2.
2750: static int getRootIndex(const int rank, const int nRanks, const int nIds) {
2751:   const int rmr = nRanks % nIds;
2752:   const int rpr = nRanks / nIds;
2753:   // For the first rmr roots, we assign one more rank to the root.
2754:   const int rlim = rmr * (rpr + 1);
2755:   if (rank < rlim) {
2756:     // Root with `rpr + 1` ranks, (0, 1, 2, ..., rmr - 1).
2757:     return rank % (rpr + 1) ? -1 : rank / (rpr + 1);
2758:   } else {
2759:     // Root with `rpr` ranks, (rmr, rmr + 1, ..., nIds - 1).
2760:     return (rank - rlim) % rpr ? -1 : ((rank - rlim) / rpr) + rmr;
```

- EN: Lines 2731-2760 introduces executable logic in routines such as `getRootIndex`; returns computed state or forwards results to the surrounding caller.
- CN: 第 2731-2760 行在 `getRootIndex` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2761-2790 / 第 2761-2790 行

```cpp
2761:   }
2762: }
2763: 
2764: void ProcessGroupNCCL::runHookLoop() {
2765:   c10::setThreadName("pt_nccl_runhook");
2766: 
2767:   bool done = false;
2768:   while (!done || !terminateProcessGroup_.load()) {
2769:     std::unique_lock<std::mutex> lock(completedWorkListMutex_);
2770:     // We busy-poll the work vector every kWatchdogThreadSleepMillis
2771:     // milliseconds as long as the atomic is True.
2772:     completedWorkListCV_.wait_for(
2773:         lock,
2774:         std::chrono::milliseconds(kWatchdogThreadSleepMillis),
2775:         [&]() -> bool {
2776:           return !completedWorkList_.empty() || terminateProcessGroup_.load();
2777:         });
2778: 
2779:     try {
2780:       for (auto it = completedWorkList_.begin(); it != completedWorkList_.end();
2781:            /* no increment */) {
2782:         const WorkNCCL& work = *it;
2783:         // Hook might grab GIL, unlock first to prevent deadlock
2784:         lock.unlock();
2785: 
2786:         auto timeFinished = std::chrono::steady_clock::now();
2787:         auto timeStarted =
2788:             timeFinished +
2789:             std::chrono::duration_cast<std::chrono::steady_clock::duration>(
2790:                 work.workStartTime_ - std::chrono::steady_clock::now());
```

- EN: Lines 2761-2790 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2761-2790 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2791-2820 / 第 2791-2820 行

```cpp
2791:         onCompletionHook_(std::make_shared<WorkInfo>(
2792:             work.retrieveOpType(), // OpType
2793:             work.getSequencenumber(), // seq
2794:             timeStarted, // timeStarted
2795:             timeFinished, // timeFinished
2796:             std::chrono::duration<float, std::milli>(
2797:                 work.getDuration()) // activeDuration
2798:             ));
2799: 
2800:         lock.lock();
2801:         it = completedWorkList_.erase(it);
2802:       }
2803:     } catch (std::exception& e) {
2804:       if (std::string(e.what()).find("driver shutting down") !=
2805:           std::string::npos) {
2806:         LOG(INFO)
2807:             << logPrefix()
2808:             << "main process destroyed cuda before runHookLoop exited, terminating runHookLoop."
2809:             << " (runHookLoop caught exception: " << e.what();
2810: 
2811:       } else {
2812:         // PythonOnCompletionHook has already extracted Python exception message
2813:         // and wrapped it with a cpp one. So we no longer need to acquire GIL
2814:         // here.
2815:         const auto errorStr = c10::str(
2816:             "Caught exception on rank ",
2817:             rank_,
2818:             " while running onCompletion hook for ProcessGroupNCCL: ",
2819:             e.what(),
2820:             ". Aborting all communicators.");
```

- EN: Lines 2791-2820 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2791-2820 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2821-2850 / 第 2821-2850 行

```cpp
2821: 
2822:         // No need to call abort() on WorkNCCL here as that collective has
2823:         // already finished successfully at this point. We just need to abort
2824:         // the process Abort all NCCL Communicators on this ProcessGroupNCCL
2825:         // instance.
2826:         abortComms(errorStr);
2827:       }
2828:     }
2829: 
2830:     // Lock is still acquired at this point
2831:     done = completedWorkList_.empty();
2832:   }
2833: }
2834: 
2835: std::exception_ptr ProcessGroupNCCL::WorkNCCL::checkForNCCLErrors() {
2836:   return checkForNCCLErrorsInternal(ncclComm_);
2837: }
2838: 
2839: std::exception_ptr ProcessGroupNCCL::checkForNCCLErrors(
2840:     std::shared_ptr<NCCLComm>& ncclComm) {
2841:   return checkForNCCLErrorsInternal(ncclComm);
2842: }
2843: 
2844: std::exception_ptr ProcessGroupNCCL::checkForNCCLErrorsInternal(
2845:     std::shared_ptr<NCCLComm>& ncclComm) {
2846:   // Prioritize commFailureReason over checkForNcclError() result if
2847:   // commFailureReason is set.
2848:   auto commFailureReason = ncclComm->getNcclCommFailureReason();
2849:   if (commFailureReason != std::nullopt) {
2850:     return std::make_exception_ptr(C10_BUILD_ERROR(
```

- EN: Lines 2821-2850 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 2821-2850 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 2851-2880 / 第 2851-2880 行

```cpp
2851:         DistBackendError,
2852:         c10::str(
2853:             "NCCL communicator encountered error set by ProcessGroupNCCL: ",
2854:             *commFailureReason)));
2855:   }
2856:   ncclResult_t ncclAsyncErr = ncclComm->checkForNcclError();
2857:   // When nonblocking mode is enabled by TORCH_NCCL_USE_COMM_NONBLOCKING,
2858:   // ncclInProgress could be returned when there are pending NCCL calls.
2859:   // In this case, no exception should be thrown
2860: #ifdef NCCL_HAS_COMM_NONBLOCKING
2861:   // ncclInProgress is defined only if NCCL_HAS_COMM_NONBLOCKING is defined
2862:   if (ncclAsyncErr != ncclSuccess && ncclAsyncErr != ncclInProgress) {
2863: #else
2864:   if (ncclAsyncErr != ncclSuccess) {
2865: #endif // NCCL_HAS_COMM_NONBLOCKING
2866:     return std::make_exception_ptr(C10_BUILD_ERROR(
2867:         DistBackendError,
2868:         "NCCL error: " + ncclGetErrorWithVersion(ncclAsyncErr) + "\n" +
2869:             getNcclErrorDetailStr(ncclAsyncErr)));
2870:   }
2871: 
2872:   return nullptr;
2873: }
2874: 
2875: void ProcessGroupNCCL::broadcastUniqueNCCLID(
2876:     ncclUniqueId* ncclID,
2877:     bool isSingleP2POp,
2878:     const std::string& p2pKey,
2879:     int p2pRank) {
2880:   // For collective operations:
```

- EN: Lines 2851-2880 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2851-2880 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2881-2910 / 第 2881-2910 行

```cpp
2881:   // For every NCCL communicator that we create we need to broadcast
2882:   // a unique ID from rank 0 to all other ranks. This broadcast is
2883:   // done by rank 0 setting a key in the store and all other ranks
2884:   // retrieving the contents of that key. A single process group
2885:   // may create multiple NCCL communicators, so we use a sequence
2886:   // number to differentiate between them.
2887:   // For single point-to-point operations:
2888:   // The sequence number will only be increased on 2 out of all the
2889:   // processes in a Process Group. So all following collective
2890:   // operations will see different sequence numbers which will cause
2891:   // runtime errors. To avoid that, use the src:target pair instead
2892:   // of sequence number for p2p communications.
2893: 
2894:   std::string storeKey;
2895:   RECORD_PARAM_COMMS(
2896:       std::make_tuple(0, false), // seq
2897:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
2898:       rank_, // TODO: this might not work for P2P
2899:       "broadcastUniqueNCCLID", // collective name
2900:       0, // inNelems
2901:       0, // outNelems
2902:       at::kByte, // dType
2903:       std::vector<int64_t>(), // inSplitSizes
2904:       std::vector<int64_t>(), // outSplitSizes
2905:       globalRankStart_, // globalRankStart_
2906:       globalRankStride_, // globalRankStride_
2907:       size_); // worldSize
2908: 
2909:   if (!isSingleP2POp) {
2910:     storeKey = std::to_string(ncclCommCounter_++);
```

- EN: Lines 2881-2910 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 2881-2910 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 2911-2940 / 第 2911-2940 行

```cpp
2911:   } else {
2912:     storeKey = p2pKey;
2913:   }
2914:   if (rank_ == 0 || (isSingleP2POp && p2pRank == 0)) {
2915:     auto vec = std::vector<uint8_t>(
2916:         reinterpret_cast<uint8_t*>(ncclID),
2917:         reinterpret_cast<uint8_t*>(ncclID) + NCCL_UNIQUE_ID_BYTES);
2918:     store_->set(storeKey, vec);
2919:   } else {
2920:     try {
2921:       auto vec = store_->get(storeKey);
2922:       TORCH_CHECK_WITH(
2923:           DistBackendError,
2924:           vec.size() == NCCL_UNIQUE_ID_BYTES,
2925:           "Invalid size for ncclUniqueId");
2926:       std::memcpy(ncclID, vec.data(), vec.size());
2927:     } catch (const std::exception& e) {
2928:       std::string exceptionMsg = c10::str(
2929:           "[",
2930:           rank_,
2931:           "] is setting up NCCL communicator and "
2932:           "retrieving ncclUniqueId from [0] via c10d key-value store by key '",
2933:           storeKey,
2934:           "', but store->get('",
2935:           storeKey,
2936:           "') got error: ");
2937:       C10_THROW_ERROR(
2938:           DistBackendError,
2939:           exceptionMsg + e.what() +
2940:               ". This may indicate a possible application crash on rank 0 or a network set up issue.");
```

- EN: Lines 2911-2940 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2911-2940 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2941-2970 / 第 2941-2970 行

```cpp
2941:     } catch (...) {
2942:       C10_THROW_ERROR(
2943:           DistBackendError,
2944:           c10::str(
2945:               "Unknown exception while [",
2946:               rank_,
2947:               "] is setting up NCCL communicator and "
2948:               "retrieving ncclUniqueId from [0] via c10d key-value store by key '",
2949:               storeKey,
2950:               "'",
2951:               ". This may indicate a possible application crash on rank 0 or a network set up issue."));
2952:     }
2953:   }
2954: }
2955: 
2956: // We want to all-gather unique NCCL IDs from all roots using TCPStore.
2957: // This is first done by setting the ID by each root and then `multiGet` by all
2958: // ranks.
2959: void ProcessGroupNCCL::allgatherUniqueNCCLIDs(
2960:     int rootIdx,
2961:     ncclUniqueId* ncclID,
2962:     std::vector<ncclUniqueId>& ncclIDs) {
2963:   std::vector<std::string> storeKeys;
2964:   std::vector<std::vector<uint8_t>> results;
2965:   RECORD_PARAM_COMMS(
2966:       std::make_tuple(0, false), // seq
2967:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
2968:       rank_, // rank
2969:       "allgatherUniqueNCCLIDs", // collective name
2970:       0, // inNelems
```

- EN: Lines 2941-2970 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2941-2970 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 2971-3000 / 第 2971-3000 行

```cpp
2971:       0, // outNelems
2972:       at::kByte, // dType
2973:       std::vector<int64_t>(), // inSplitSizes
2974:       std::vector<int64_t>(), // outSplitSizes
2975:       globalRankStart_, // globalRankStart_
2976:       globalRankStride_, // globalRankStride_
2977:       size_); // worldSize
2978: 
2979:   for (size_t r = 0; r < ncclIDs.size(); r++) {
2980:     storeKeys.emplace_back("UniqueNCCLID:" + std::to_string(r));
2981:   }
2982:   // For non-root rank, rootIdx is set to -1.
2983:   if (rootIdx >= 0) {
2984:     auto vec = std::vector<uint8_t>(
2985:         reinterpret_cast<uint8_t*>(ncclID),
2986:         reinterpret_cast<uint8_t*>(ncclID) + NCCL_UNIQUE_ID_BYTES);
2987:     store_->set(storeKeys[rootIdx], vec);
2988:   }
2989:   try {
2990:     results = store_->multiGet(storeKeys);
2991:   } catch (const std::exception& e) {
2992:     nlohmann::json json_vec = storeKeys;
2993:     std::string exceptionMsg = c10::str(
2994:         "[",
2995:         rank_,
2996:         "] is setting up NCCL communicators and "
2997:         "retrieving ncclUniqueId from roots via TCPStore by key '",
2998:         json_vec.dump(),
2999:         "', but got error: ");
3000:     C10_THROW_ERROR(
```

- EN: Lines 2971-3000 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 2971-3000 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 3001-3030 / 第 3001-3030 行

```cpp
3001:         DistBackendError,
3002:         exceptionMsg + e.what() +
3003:             ". This may indicate a possible application crash on rank 0 or a network set up issue.");
3004:   } catch (...) {
3005:     nlohmann::json json_vec = storeKeys;
3006:     C10_THROW_ERROR(
3007:         DistBackendError,
3008:         c10::str(
3009:             "Unknown exception while [",
3010:             rank_,
3011:             "] is setting up NCCL communicators and "
3012:             "retrieving ncclUniqueIds from roots via TCPStore by key '",
3013:             json_vec.dump(),
3014:             "'",
3015:             ". This may indicate a possible application crash on rank 0 or a network set up issue."));
3016:   }
3017: 
3018:   for (size_t r = 0; r < ncclIDs.size(); r++) {
3019:     TORCH_CHECK_WITH(
3020:         DistBackendError,
3021:         results[r].size() == NCCL_UNIQUE_ID_BYTES,
3022:         "Invalid size for ncclUniqueId");
3023:     std::memcpy(&ncclIDs[r], results[r].data(), results[r].size());
3024:   }
3025: }
3026: 
3027: void ProcessGroupNCCL::destroyNCCLComms(const std::string& devNCCLCommMapKey) {
3028:   std::lock_guard<std::mutex> lock(mutex_);
3029:   if (devNCCLCommMap_.find(devNCCLCommMapKey) == devNCCLCommMap_.end()) {
3030:     TORCH_INTERNAL_ASSERT(
```

- EN: Lines 3001-3030 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 3001-3030 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 3031-3060 / 第 3031-3060 行

```cpp
3031:         false,
3032:         "Expected to find key ",
3033:         devNCCLCommMapKey,
3034:         " in NCCL communicator map.");
3035:   }
3036:   std::shared_ptr<NCCLComm>& ncclComm = devNCCLCommMap_[devNCCLCommMapKey];
3037:   // ncclCommDestroy(comm->getNcclComm()) results in segfault when PG is being
3038:   // destroyed, so using ncclCommAbort here.
3039:   ncclComm->abort();
3040:   // Remove communicators from the cache.
3041:   devNCCLCommMap_.erase(devNCCLCommMapKey);
3042:   // Clear used device indices.
3043:   usedDeviceIdxs_.clear();
3044: 
3045:   {
3046:     std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
3047:     ncclCommMemPoolMap.erase(ncclComm);
3048:   }
3049: }
3050: 
3051: std::shared_ptr<NCCLComm> ProcessGroupNCCL::initNCCLComm(
3052:     const std::string& deviceKey,
3053:     at::Device& device,
3054:     OpType opType,
3055:     int p2pRank,
3056:     bool isSendRecvSelf) {
3057:   // Sanity check
3058:   if (deviceKey.empty()) {
3059:     C10_THROW_ERROR(
3060:         DistBackendError,
```

- EN: Lines 3031-3060 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 3031-3060 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 3061-3090 / 第 3061-3090 行

```cpp
3061:         "Not able to create/get the NCCL Communicator since "
3062:         "the GPU devices are not known");
3063:   }
3064:   if (bound_device_id_) {
3065:     if (*bound_device_id_ != device) {
3066:       LOG(ERROR) << logPrefix() << "Tensor found on device " << device
3067:                  << " but backend constrained to " << *bound_device_id_;
3068:       C10_THROW_ERROR(
3069:           DistBackendError,
3070:           "Attempt to perform collective on tensor not on device passed to init_process_group");
3071:     }
3072:   }
3073: 
3074:   usedDeviceIdxs_.insert(device.index());
3075: 
3076:   // NCCL communicator not cached, create a new entry
3077:   std::shared_ptr<NCCLComm> ncclComm;
3078: 
3079:   // Create the unique NCCL ID and broadcast it
3080:   ncclUniqueId ncclID;
3081: 
3082:   // reset log prefix to include group_desc
3083:   logPrefix_ = createLogPrefix();
3084: 
3085: #ifdef NCCL_COMM_DESCRIPTION
3086:   // Pass process group name and description to NCCL communicator
3087:   std::string commDesc = pg_desc_ + ':' + pg_uid_;
3088:   options_->config.commDesc = strdup(commDesc.c_str());
3089: #endif // NCCL_COMM_DESCRIPTION
3090: 
```

- EN: Lines 3061-3090 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3061-3090 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3091-3120 / 第 3091-3120 行

```cpp
3091:   // For batch_isend_irecv, ncclGroupStart() would be called upfront
3092:   bool batchP2P = ncclActiveGroupCounter_ > 0;
3093:   bool singleP2POp = isP2POp(opType, batchP2P);
3094: 
3095:   // Get the device index
3096:   auto deviceIndex = device.index();
3097:   at::cuda::OptionalCUDAGuard gpuGuard(device);
3098: 
3099:   // [Group Start/End Note] This is used to ensure that nccl communicator will
3100:   // be created before communication primitives are called. Let's look at this
3101:   // example: Using the batch_isend_irecv to send a tensor to a target process.
3102:   // On the sender side, the corresponding underlying NCCL calls will look like
3103:   //   ncclGroupStart() // This is in batch_isend_irecv
3104:   //   ncclCommInitRank() // Inside NCCLComm::create
3105:   //   ncclSend()
3106:   //   ncclGroupEnd() // This is in batch_isend_irecv
3107:   // With this pattern, the nccl communicator will be created in the last
3108:   // ncclGroupEnd which means when ncclSend is processed, the passed
3109:   // communicator argument is NULL which will lead to runtime error. So we need
3110:   // to "close" all active nccl groups to ensure nccl communicator is actually
3111:   // created before encountering any communication calls. This is why we need
3112:   // the following for loop.
3113:   for (const auto i : c10::irange(ncclActiveGroupCounter_)) {
3114:     (void)i;
3115:     // comms have not been initiated yet, so can only check in blocking-way
3116:     C10D_NCCL_CHECK(ncclGroupEnd(), std::nullopt);
3117:   }
3118: 
3119:   // GPU world size and GPU rank
3120:   int numRanks = -1, rank = -1;
```

- EN: Lines 3091-3120 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3091-3120 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3121-3150 / 第 3121-3150 行

```cpp
3121: 
3122:   if (!singleP2POp) {
3123:     // Collective, all-to-all, or batch P2P
3124:     numRanks = getSize();
3125:     rank = getRank();
3126:   } else if (isSendRecvSelf) {
3127:     // Same process send and recv.
3128:     numRanks = 1;
3129:     rank = 0;
3130:   } else {
3131:     // For single point-to-point operation, there are only 2 processes
3132:     // involved so the GPU rank is either 0 or 1.
3133:     numRanks = 2;
3134:     rank = p2pRank;
3135:   }
3136: 
3137:   RECORD_PARAM_COMMS(
3138:       std::make_tuple(0, false), // seq
3139:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
3140:       rank, // rank
3141:       "init", // collective name
3142:       0, // inNelems
3143:       0, // outNelems
3144:       at::kByte, // dType
3145:       std::vector<int64_t>(), // inSplitSizes
3146:       std::vector<int64_t>(), // outSplitSizes
3147:       globalRankStart_, // globalRankStart_
3148:       globalRankStride_, // globalRankStride_
3149:       size_); // worldSize
3150: 
```

- EN: Lines 3121-3150 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3121-3150 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3151-3180 / 第 3151-3180 行

```cpp
3151: #ifdef NCCL_HAS_COMM_NONBLOCKING
3152:   bool useNb = useNonblocking();
3153:   options_->config.blocking = useNb ? 0 : 1;
3154: #endif // NCCL_HAS_COMM_NONBLOCKING
3155: 
3156: #ifdef NCCL_HAS_COMM_SPLIT
3157:   // Use split to create a new communicator only if:
3158:   // 1. The parent comm is known; AND
3159:   // 2. The new comm is not for a point-to-point operation.
3160:   // ncclCommSplit() is a collective call, so it does not work for P2P
3161:   // operations.
3162:   if (options_->split_from && !singleP2POp) {
3163:     // Find a valid, healthy communicator to split from if possible.
3164:     std::lock_guard<std::mutex> lock(options_->split_from->mutex_);
3165:     auto& other_comms = options_->split_from->devNCCLCommMap_;
3166:     auto dit = other_comms.find(getKeyFromDevice(device));
3167:     if (dit != other_comms.end()) {
3168:       auto& parentComm = dit->second;
3169:       if (parentComm != nullptr && !parentComm->isAborted()) {
3170:         LOG(INFO) << logPrefix() << "Splitting NCCL communicator from "
3171:                   << parentComm->repr();
3172:         ncclComm = NCCLComm::split(
3173:             parentComm.get(), options_->split_color, rank, options_->config);
3174:       }
3175:     }
3176:   }
3177: #endif // NCCL_HAS_COMM_SPLIT
3178: 
3179:   bool useScalableInit = false;
3180:   // (nranks / nroots) == 128 was the default NCCL recommended
```

- EN: Lines 3151-3180 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `lock`.
- CN: 第 3151-3180 行使用条件编译来适配特性开关、平台或可选后端；在 `lock` 等例程中引入具体执行逻辑。

### Lines 3181-3210 / 第 3181-3210 行

```cpp
3181:   // according to
3182:   // https://github.com/pytorch/pytorch/pull/136789#discussion_r1779171615.
3183:   auto ranksPerRoot = getCvarInt(TORCH_NCCL_RANKS_PER_ROOT, 128);
3184: #if defined(NCCL_HAS_INIT_RANK_SCALABLE) && defined(NCCL_HAS_CONFIG)
3185:   useScalableInit = !singleP2POp && (getSize() > ranksPerRoot);
3186: #endif // NCCL_HAS_INIT_RANK_SCALABLE && NCCL_HAS_CONFIG
3187: 
3188:   if (useScalableInit) {
3189:     auto numRoots = (getSize() + ranksPerRoot - 1) / ranksPerRoot;
3190:     std::vector<ncclUniqueId> ncclIDs(numRoots);
3191: 
3192:     if (!ncclComm) {
3193:       auto rootIdx = getRootIndex(rank_, getSize(), numRoots);
3194:       // We only need to get unique IDs for roots. For non-root rank, index is
3195:       // set to -1.
3196:       if (rootIdx >= 0) {
3197:         C10D_NCCL_CHECK(ncclGetUniqueId(&ncclID), std::nullopt);
3198:       }
3199:       // We only need to all-gather the ncclID if the rank is root.
3200:       auto timeStarted = std::chrono::steady_clock::now();
3201:       allgatherUniqueNCCLIDs(rootIdx, &ncclID, ncclIDs);
3202:       auto timerDeltaMs =
3203:           std::chrono::duration_cast<std::chrono::duration<double>>(
3204:               std::chrono::steady_clock::now() - timeStarted)
3205:               .count() *
3206:           1000;
3207:       LOG(INFO) << logPrefix()
3208:                 << "ProcessGroupNCCL all-gather unique IDs through store took "
3209:                 << timerDeltaMs << " ms";
3210: #if defined(NCCL_HAS_INIT_RANK_SCALABLE) && defined(NCCL_HAS_CONFIG)
```

- EN: Lines 3181-3210 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `allgatherUniqueNCCLIDs`.
- CN: 第 3181-3210 行使用条件编译来适配特性开关、平台或可选后端；在 `allgatherUniqueNCCLIDs` 等例程中引入具体执行逻辑。

### Lines 3211-3240 / 第 3211-3240 行

```cpp
3211:       ncclComm = NCCLComm::create_scalable(
3212:           numRanks, rank, ncclIDs, deviceIndex, options_->config);
3213: #else
3214:       C10_THROW_ERROR(
3215:           DistBackendError,
3216:           c10::str(
3217:               logPrefix(),
3218:               "create_scalable is called when useScalableInit is enabled but ",
3219:               "neither NCCL_HAS_INIT_RANK_SCALABLE nor NCCL_HAS_CONFIG is not defined, this should not happen "));
3220: #endif // NCCL_HAS_INIT_RANK_SCALABLE
3221:     }
3222:   } else {
3223:     // To simplify conditional nesting, just create the ncclComms[i]
3224:     // entry if it hasn't been yet rather than untangling the
3225:     // conditions that might have resulted in a split above.
3226:     if (!ncclComm) {
3227:       if (getCvarBool(TORCH_NCCL_BCAST_UNIQUEID, true) && !isSendRecvSelf) {
3228:         // For point-to-point communication, lower rank of the two will get
3229:         // unique id.
3230:         if (rank_ == 0 || (singleP2POp && p2pRank == 0)) {
3231:           C10D_NCCL_CHECK(ncclGetUniqueId(&ncclID), std::nullopt);
3232:         }
3233: 
3234:         // Broadcast so that each process can have a unique NCCL ID
3235:         auto timeStarted = std::chrono::steady_clock::now();
3236:         broadcastUniqueNCCLID(&ncclID, singleP2POp, deviceKey, p2pRank);
3237:         auto timerDeltaMs =
3238:             std::chrono::duration_cast<std::chrono::duration<double>>(
3239:                 std::chrono::steady_clock::now() - timeStarted)
3240:                 .count() *
```

- EN: Lines 3211-3240 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `broadcastUniqueNCCLID`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3211-3240 行使用条件编译来适配特性开关、平台或可选后端；在 `broadcastUniqueNCCLID` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3241-3270 / 第 3241-3270 行

```cpp
3241:             1000;
3242:         LOG(INFO) << logPrefix()
3243:                   << "ProcessGroupNCCL broadcast unique ID through store took "
3244:                   << timerDeltaMs << " ms";
3245:       }
3246: 
3247: #ifdef NCCL_HAS_CONFIG
3248:       ncclComm = NCCLComm::create(
3249:           numRanks, rank, ncclID, deviceIndex, options_->config);
3250: #else
3251:       ncclComm = NCCLComm::create(numRanks, rank, ncclID, deviceIndex);
3252: #endif // NCCL_HAS_CONFIG
3253:     }
3254:   }
3255: 
3256:   // Creates the NCCL streams
3257:   bool force_high = getCvarBool(TORCH_NCCL_HIGH_PRIORITY, false);
3258:   auto streamVal = at::cuda::getStreamFromPool(
3259:       options_->is_high_priority_stream || force_high);
3260: 
3261:   {
3262:     std::lock_guard<std::mutex> lock(mutex_);
3263:     inInitializationCommMap_.emplace(deviceKey, ncclComm);
3264:   }
3265: 
3266:   FlightRecorderCUDA::get()->record_pg_ranks(
3267:       std::make_tuple(pg_uid_, pg_desc_), groupRanks());
3268:   FlightRecorderCUDA::get()->record_accelerator_version(getNcclVersion());
3269: 
3270:   VLOG(2) << logPrefix() << "ProcessGroupNCCL created ncclComm_ "
```

- EN: Lines 3241-3270 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3241-3270 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3271-3300 / 第 3271-3300 行

```cpp
3271:           << ncclComm->repr()
3272:           << " on CUDA device: " << static_cast<int>(deviceIndex);
3273: 
3274:   // At this point NCCL should have been initialized, hence we can accurately
3275:   // get the env value even if NCCL sets it by reading from nccl.conf file
3276:   LOG(INFO) << logPrefix()
3277:             << "NCCL_DEBUG: " << getCvarString({"NCCL_DEBUG"}, "N/A");
3278: 
3279:   // See [Group Start/End Note]
3280:   for (const auto i : c10::irange(ncclActiveGroupCounter_)) {
3281:     (void)i;
3282:     C10D_NCCL_CHECK(ncclGroupStart(), std::nullopt);
3283:   }
3284: 
3285:   ncclStreams_.emplace(deviceKey, streamVal);
3286: 
3287:   // Note: these events are created with the (default) cudaEventDisableTiming
3288:   // flag This flag provides the best performance when used with
3289:   // cudaStreamWaitEvent() and cudaEventQuery(). Since we here don't measure the
3290:   // performance using cudaEvent, this should be set.
3291:   // TODO(kwen2501): is ncclEvents_ used anywhere else?
3292:   ncclEvents_.emplace(deviceKey, at::cuda::CUDAEvent(cudaEventDisableTiming));
3293: 
3294:   // Move the NCCL resource to cache
3295:   auto it = inInitializationCommMap_.find(deviceKey);
3296:   // A previous thread could've already removed devicesKey from
3297:   // inInitializationCommMap_ and added it to devNCCLCommMap_
3298:   if (it != inInitializationCommMap_.end()) {
3299:     devNCCLCommMap_.emplace(deviceKey, std::move(it->second));
3300:     inInitializationCommMap_.erase(deviceKey);
```

- EN: Lines 3271-3300 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3271-3300 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3301-3330 / 第 3301-3330 行

```cpp
3301: 
3302:     // Now ncclComms are fully initialized.
3303:     // Register all active CUDA memory segments in cache allocator to
3304:     // the new NCCL communicators
3305:     if (shouldAllCommunicatorsRegisterAllTensors()) {
3306:       auto snapshot = c10::cuda::CUDACachingAllocator::snapshot();
3307:       // Register the segment to a new NCCL communicator if on the same device
3308:       for (const auto& segmentInfo : snapshot.segments) {
3309:         TORCH_INTERNAL_ASSERT(
3310:             segmentInfo.device == device.index(),
3311:             "Mismatch between CUDA memory segment device and current device");
3312:         ncclComm->registerSegment(
3313:             // NOLINTNEXTLINE(performance-no-int-to-ptr)
3314:             reinterpret_cast<void*>(segmentInfo.address),
3315:             segmentInfo.total_size);
3316:       }
3317:     }
3318:     // Record the mapping between ncclComm and device index so that later
3319:     // register hook can register a newly allocated segment to communicators
3320:     // on the same device.
3321:     // NOTE: we need remove the communicator from this map when it is
3322:     // destroyed, otherwise may register onto an invalid communicator.
3323:     {
3324:       std::lock_guard<std::mutex> lock(ncclCommMemPoolMapMutex);
3325:       ncclCommMemPoolMap.emplace(ncclComm, MemPoolSet{});
3326:     }
3327:   }
3328: 
3329:   it = devNCCLCommMap_.find(deviceKey);
3330:   TORCH_INTERNAL_ASSERT(
```

- EN: Lines 3301-3330 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3301-3330 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3331-3360 / 第 3331-3360 行

```cpp
3331:       it != devNCCLCommMap_.end(), "Communicators not populated in cache!");
3332:   return it->second;
3333: }
3334: 
3335: int64_t ProcessGroupNCCL::getCommPtr() {
3336:   // Get the collective communicator on the current CUDA device.
3337:   auto device = at::Device(at::kCUDA, at::cuda::current_device());
3338:   std::string deviceKey = getKeyFromDevice(device);
3339:   auto ncclComm = getNCCLComm(deviceKey);
3340: 
3341:   // ncclComm is a nullptr if the communicator does not exist.
3342:   ncclComm_t comm = nullptr;
3343:   if (ncclComm != nullptr) {
3344:     comm = ncclComm->getNcclComm();
3345:   }
3346:   const int64_t commPtr = reinterpret_cast<int64_t>(comm);
3347:   return commPtr;
3348: }
3349: 
3350: std::shared_ptr<NCCLComm> ProcessGroupNCCL::getNCCLComm(
3351:     const std::string& deviceKey) {
3352:   std::lock_guard<std::mutex> lock(mutex_);
3353:   if (devNCCLCommMap_.find(deviceKey) != devNCCLCommMap_.end()) {
3354:     // Reuse the cached communicator if there is one.
3355:     return devNCCLCommMap_[deviceKey];
3356:   }
3357:   return nullptr;
3358: }
3359: 
3360: uint64_t ProcessGroupNCCL::getCommSplitCounter() const {
```

- EN: Lines 3331-3360 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 3331-3360 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 3361-3390 / 第 3361-3390 行

```cpp
3361:   uint64_t ret = 0;
3362:   for (const auto& i : devNCCLCommMap_) {
3363:     auto& ncclComm = i.second;
3364:     ret += ncclComm->getCommSplitCounter();
3365:   }
3366:   return ret;
3367: }
3368: 
3369: void ProcessGroupNCCL::suspend() {
3370:   auto device = at::Device(at::kCUDA, guessDeviceId());
3371:   std::string deviceKey = getKeyFromDevice(device);
3372:   auto ncclComm = getNCCLComm(deviceKey);
3373:   TORCH_CHECK(ncclComm != nullptr, "NCCL communicator not initialized.");
3374:   ncclComm->suspend();
3375: }
3376: 
3377: void ProcessGroupNCCL::resume() {
3378:   auto device = at::Device(at::kCUDA, guessDeviceId());
3379:   std::string deviceKey = getKeyFromDevice(device);
3380:   auto ncclComm = getNCCLComm(deviceKey);
3381:   TORCH_CHECK(ncclComm != nullptr, "NCCL communicator not initialized.");
3382:   ncclComm->resume();
3383: }
3384: 
3385: std::unordered_map<std::string, uint64_t> ProcessGroupNCCL::getMemoryStats() {
3386:   auto device = at::Device(at::kCUDA, guessDeviceId());
3387:   std::string deviceKey = getKeyFromDevice(device);
3388:   auto ncclComm = getNCCLComm(deviceKey);
3389:   TORCH_CHECK(ncclComm != nullptr, "NCCL communicator not initialized.");
3390:   return ncclComm->getMemoryStats();
```

- EN: Lines 3361-3390 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3361-3390 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3391-3420 / 第 3391-3420 行

```cpp
3391: }
3392: 
3393: namespace {
3394: 
3395: // Check validity of tensor
3396: void check_gpu_single_tensor(
3397:     const at::Tensor& tensor,
3398:     const bool p2p = false // whether operation is a P2P operation
3399: ) {
3400:   if (!tensor.is_cuda() || tensor.is_sparse()) {
3401:     C10_THROW_ERROR(ValueError, "Tensors must be CUDA and dense");
3402:   }
3403:   // Check memory format
3404:   if (!tensor.is_contiguous(tensor.suggest_memory_format())) {
3405:     // P2P is a bit relaxed, supporting transfer of a transposed tensor
3406:     if (p2p) {
3407:       // But must be dense still
3408:       if (!tensor.is_non_overlapping_and_dense()) {
3409:         C10_THROW_ERROR(
3410:             ValueError, "Tensors for P2P must be non-overlapping and dense");
3411:       }
3412:       TORCH_WARN_ONCE(
3413:           "Detected non-contiguous tensor in P2P operations. It is user "
3414:           "responsibility to guarantee that source and destination tensors have "
3415:           "the same contiguity format.");
3416:     } else {
3417:       C10_THROW_ERROR(ValueError, "Tensors must be contiguous");
3418:     }
3419:   }
3420: }
```

- EN: Lines 3391-3420 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `check_gpu_single_tensor`, `C10_THROW_ERROR`, `TORCH_WARN_ONCE`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3391-3420 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `check_gpu_single_tensor`、`C10_THROW_ERROR`、`TORCH_WARN_ONCE` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3421-3450 / 第 3421-3450 行

```cpp
3421: 
3422: // Checks that all `tensors' have the same type and shape and reside on the same
3423: // GPU.
3424: // TODO: test_c10d_nccl.py should consider adding tests for the error conditions
3425: // here, ie, that deliberately pass invalid tensors and check the right
3426: // exception is thrown. The "Expected list of tensors on the same device"
3427: // condition may be a challenge because the test would need to pass tensors on
3428: // different devices in the same process.
3429: int64_t check_gpu_tensors_same_device(const std::vector<at::Tensor>& tensors) {
3430:   if (tensors.empty()) {
3431:     C10_THROW_ERROR(ValueError, "Tensor list must be nonempty");
3432:   }
3433: 
3434:   const auto& first = tensors.front();
3435: 
3436:   int64_t total_numel = 0;
3437:   for (const auto& t : tensors) {
3438:     if (!t.is_cuda() || t.is_sparse()) {
3439:       C10_THROW_ERROR(ValueError, "Tensors must be CUDA and dense");
3440:     }
3441:     if (t.scalar_type() != first.scalar_type()) {
3442:       C10_THROW_ERROR(TypeError, "Tensors must have identical type");
3443:     }
3444:     if (!t.is_non_overlapping_and_dense()) {
3445:       C10_THROW_ERROR(ValueError, "Tensors must be non-overlapping and dense");
3446:     }
3447:     // If we're in this function, the user called a _coalesced collective
3448:     // on a set of tensors with potentially different sizes and strides.
3449:     // Therefore, we don't check for matching sizes and strides,
3450:     // but we do double-check tensors are on the same device.
```

- EN: Lines 3421-3450 introduces executable logic in routines such as `check_gpu_tensors_same_device`, `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3421-3450 行在 `check_gpu_tensors_same_device`、`C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3451-3480 / 第 3451-3480 行

```cpp
3451:     TORCH_CHECK_WITH(
3452:         ValueError,
3453:         t.get_device() == tensors[0].get_device(),
3454:         "Expected list of tensors on the same device");
3455:     total_numel += t.numel();
3456:   }
3457: 
3458:   return total_numel;
3459: }
3460: 
3461: bool check_same_size(const std::vector<at::Tensor>& input_tensors) {
3462:   for (const auto& input_tensor : input_tensors) {
3463:     if (!input_tensors[0].is_same_size(input_tensor)) {
3464:       return false;
3465:     }
3466:   }
3467:   return true;
3468: }
3469: 
3470: } // namespace
3471: 
3472: c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL> ProcessGroupNCCL::initWork(
3473:     at::Device& device,
3474:     int rank,
3475:     OpType opType,
3476:     bool isP2P,
3477:     const char* profilingTitle,
3478:     const std::vector<at::Tensor>& inputs,
3479:     const std::vector<at::Tensor>& outputs, // TODO(kwen2501): necessary?
3480:     bool record) {
```

- EN: Lines 3451-3480 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `check_same_size`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3451-3480 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `check_same_size` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3481-3510 / 第 3481-3510 行

```cpp
3481:   auto r = c10::make_intrusive<ProcessGroupNCCL::WorkNCCL>(
3482:       pg_uid_,
3483:       pg_desc_,
3484:       device,
3485:       rank,
3486:       opType,
3487:       isP2P ? seqP2P_ : seqCollective_,
3488:       isP2P,
3489:       profilingTitle,
3490:       profilingTitle != nullptr ? std::optional<std::vector<at::Tensor>>(inputs)
3491:                                 : std::nullopt,
3492:       enableTiming_.load(),
3493:       cudaEventCacheEnabled_.load(),
3494:       dist_debug_level_);
3495: 
3496:   if (record) {
3497:     bool isP2P = isP2POp(opType);
3498:     // Ideally record every work that we enqueue, rather than every work we
3499:     // create.
3500:     // - at the time of this PR we do not currently enqueue every created work
3501:     // - but it is unsafe to steal refs to start/end cuda events from Works that
3502:     //   may go out of scope before flight recorder has retired them,
3503:     //   so we must ensure that any work that is initialized via initWork will
3504:     //   be enqueued
3505:     // - initially, moved record() into workEnqueue(), but found that makes it
3506:     //   hard to get access to profilingTitle,
3507:     //   inputs, and outputs for metadata recording, and we don't want to attach
3508:     //   these objects to the Work because it has implications for keeping those
3509:     //   tensors alive longer and adds overhead when copying Work objects
3510:     //   between threads
```

- EN: Lines 3481-3510 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3481-3510 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3511-3540 / 第 3511-3540 行

```cpp
3511:     auto traceId = FlightRecorderCUDA::get()->recordWithResetEnabled(
3512:         local_id_,
3513:         std::make_tuple(pg_uid_, pg_desc_),
3514:         seqCollective_,
3515:         seqP2P_,
3516:         op_id_,
3517:         profilingTitle ? profilingTitle : "",
3518:         inputs,
3519:         outputs,
3520:         r->ncclStartEvent_.get(),
3521:         r->ncclEndEvent_.get(),
3522:         options_->timeout,
3523:         pgStatus_,
3524:         isP2P);
3525:     r->trace_id_ = traceId.id;
3526:     r->trace_reset_epoch_ = traceId.reset_epoch;
3527:   }
3528:   return r;
3529: }
3530: 
3531: // TODO(kwen2501): deprecate
3532: std::vector<at::Tensor> ProcessGroupNCCL::WorkNCCL::result() {
3533:   return *outputs_;
3534: }
3535: 
3536: c10::intrusive_ptr<c10::ivalue::Future> ProcessGroupNCCL::WorkNCCL::
3537:     getFuture() {
3538:   return future_;
3539: }
3540: 
```

- EN: Lines 3511-3540 introduces executable logic in routines such as `getFuture`; returns computed state or forwards results to the surrounding caller.
- CN: 第 3511-3540 行在 `getFuture` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 3541-3570 / 第 3541-3570 行

```cpp
3541: c10::intrusive_ptr<c10::ivalue::Future> ProcessGroupNCCL::WorkNCCL::
3542:     getFutureResult() {
3543:   return futureWorkResult_;
3544: }
3545: 
3546: float ProcessGroupNCCL::WorkNCCL::getDuration() const {
3547:   TORCH_CHECK(timingEnabled_, "getDuration only works if timing was enabled");
3548:   TORCH_CHECK(
3549:       ncclStartEvent_,
3550:       "getDuration only works if ncclStartEvents_ is populated, true if timing enabled");
3551:   TORCH_CHECK(
3552:       ncclEndEvent_,
3553:       "getDuration only works if ncclEndEvents_ is populated, which should always be true");
3554:   return ncclStartEvent_->elapsed_time(*ncclEndEvent_);
3555: }
3556: 
3557: uint64_t ProcessGroupNCCL::WorkNCCL::getSequencenumber() const {
3558:   return seq_;
3559: }
3560: 
3561: void ProcessGroupNCCL::assignTimeoutToWork(
3562:     const c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work,
3563:     const c10::intrusive_ptr<ProcessGroupNCCL::Options>& option) {
3564:   std::chrono::milliseconds timeout = option->timeout;
3565:   std::lock_guard<std::mutex> timeoutLock(mtxTimeoutExtension_);
3566:   if (ephemeralTimeoutActive_.count() > 0) {
3567:     timeout += ephemeralTimeoutActive_;
3568:   }
3569:   work->opTimeout_ = timeout;
3570:   work->ownedEphermeralTimeout_ =
```

- EN: Lines 3541-3570 introduces executable logic in routines such as `getFutureResult`, `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3541-3570 行在 `getFutureResult`、`TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3571-3600 / 第 3571-3600 行

```cpp
3571:       ephemeralTimeoutActive_ - ephemeralTimeoutInflight_;
3572:   ephemeralTimeoutInflight_ = ephemeralTimeoutActive_;
3573: }
3574: 
3575: void ProcessGroupNCCL::workEnqueue(
3576:     const c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {
3577:   // We clean up the TensorShelf's in case user hasn't called `work.wait()`.
3578:   // This has nothing to do with new work enqueue. We are just using a place
3579:   // that would be triggered by a next user call.
3580:   {
3581:     std::lock_guard<std::mutex> lock(shelvesMutex_);
3582:     for (auto& shelf : shelvesToUnstash_) {
3583:       shelf->unstash();
3584:     }
3585:     shelvesToUnstash_.clear();
3586:   }
3587: 
3588:   // in blockingWait_ mode, we don't need watchdog thread, so no need to enqueue
3589:   // the work
3590:   if (!terminateProcessGroup_.load() && !blockingWait_) {
3591:     std::lock_guard<std::mutex> lock(workMetaListMutex_);
3592:     // Avoid view tensors to be processed in cleanup thread.
3593:     // View tensors' destruction invokes autograd_meta, which
3594:     // needs to be destructed in user thread. Otherwise will
3595:     // get deadlock. Here we enqueue work without outputs_.
3596:     workMetaList_.emplace_back(*work);
3597:     // update the PG status related to the last enqueued work
3598:     pgStatus_->lastEnqueuedSeq = static_cast<int64_t>(work->seq_);
3599:     pgStatus_->lastEnqueuedWorkName = opTypeToString(work->opType_);
3600:     pgStatus_->lastEnqueuedNumelIn = work->numelIn_;
```

- EN: Lines 3571-3600 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3571-3600 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3601-3630 / 第 3601-3630 行

```cpp
3601:     pgStatus_->lastEnqueuedNumelOut = work->numelOut_;
3602:     heartbeatMonitor_->setLastWorkListUpdateTime(
3603:         std::chrono::steady_clock::now());
3604:   }
3605: }
3606: 
3607: ProcessGroupNCCL::Options::Options(bool is_high_priority_stream)
3608:     : Backend::Options(NCCL_BACKEND_NAME, kProcessGroupNCCLDefaultTimeout),
3609:       is_high_priority_stream(is_high_priority_stream) {}
3610: 
3611: static constexpr int CoalActive = 0x01, CoalColl = 0x02, CoalP2P = 0x04;
3612: 
3613: uint64_t ProcessGroupNCCL::getWatchdogHeartbt() const {
3614:   return watchdog_->getHeartbt();
3615: }
3616: 
3617: void ProcessGroupNCCL::startCoalescing() {
3618:   // Other collective ops bump seq_ before creating a work. Thus, if coalesced
3619:   // ops bump seq_ only after initing a work they will collide with (reuse) the
3620:   // seq_ of the last non-coalesced collective.  Previously, seq_ was bumped
3621:   // inside endCoalescing, but before initWork. Since we now record individual
3622:   // ops from a coalesce group into the flight recorder, we want to have the
3623:   // same seq_ for those ops and its 'endCoalescing' op. Hence we bump during
3624:   // start, which has one minor downside- we burn a seq_ if someone ever does a
3625:   // 'start' and 'end' coalescing region without doing an operation in between.
3626: 
3627:   coalescedDevice_.set_index(-1);
3628:   coalescedComm_ = nullptr;
3629:   coalescedTensors_.clear();
3630:   coalescing_state_ |= CoalActive;
```

- EN: Lines 3601-3630 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 3601-3630 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 3631-3660 / 第 3631-3660 行

```cpp
3631:   groupStart();
3632: }
3633: 
3634: // `optype` is for specifying a composite optype, such as ALLGATHER and
3635: // REDUCE_SCATTER
3636: c10::intrusive_ptr<Work> ProcessGroupNCCL::endCoalescing(OpType optype) {
3637:   if (coalescedComm_ == nullptr) {
3638:     // There is no actual work being coalesced, return here
3639:     groupEnd();
3640:     coalescing_state_ = 0;
3641:     return nullptr;
3642:   }
3643:   TORCH_CHECK(
3644:       coalescedDevice_.index() >= 0,
3645:       "Something went wrong. Did you call end_coalescing before start_coalescing?");
3646: 
3647:   // `coalescedComm_` should have same set of comms across collectives
3648:   auto comm = coalescedComm_;
3649:   // `coalescedDevice_` should have same set of devices across collectives
3650:   auto device = coalescedDevice_;
3651: 
3652:   // `getKeyFromDevice` is how we get keys for both collectives and batch P2P
3653:   const auto key = getKeyFromDevice(device);
3654:   auto ncclStream = ncclStreams_.at(key);
3655:   auto opProfilerTitle = optype != OpType::COALESCED
3656:       ? "nccl:" + opTypeToString(optype) + "_coalesced"
3657:       : "nccl:coalesced";
3658: 
3659:   // Create Work object
3660:   c10::cuda::CaptureStatus capture_status =
```

- EN: Lines 3631-3660 introduces executable logic in routines such as `groupStart`, `groupEnd`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3631-3660 行在 `groupStart`、`groupEnd` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3661-3690 / 第 3661-3690 行

```cpp
3661:       c10::cuda::currentStreamCaptureStatusMayInitCtx();
3662:   bool enqueue =
3663:       (coalescing_state_) && capture_status == c10::cuda::CaptureStatus::None;
3664:   auto work = initWork(
3665:       device,
3666:       rank_,
3667:       optype,
3668:       coalescing_state_ & CoalP2P,
3669:       opProfilerTitle.c_str(),
3670:       {},
3671:       {},
3672:       enqueue);
3673:   work->ncclComm_ = comm;
3674:   work->blockingWait_ = blockingWait_;
3675:   work->store_ = store_;
3676:   assignTimeoutToWork(work, options_);
3677: 
3678:   // Hand over references to tensors during coalescing to work's stash
3679:   work->stashed_for_allocator_safety_->stash(coalescedTensors_);
3680: 
3681:   // Record start before ncclGroupEnd
3682:   if (work->timingEnabled_) {
3683:     work->ncclStartEvent_->record(ncclStream);
3684:   }
3685: 
3686:   if (useNonblocking()) {
3687:     groupEndNonblocking(comm);
3688:   } else {
3689:     groupEnd();
3690:   }
```

- EN: Lines 3661-3690 introduces executable logic in routines such as `groupEnd`.
- CN: 第 3661-3690 行在 `groupEnd` 等例程中引入具体执行逻辑。

### Lines 3691-3720 / 第 3691-3720 行

```cpp
3691: 
3692:   // Record end after ncclGroupEnd
3693:   // TODO(eqy): is this still necessary if avoidRecordStreams_ is set?
3694:   work->ncclEndEvent_->record(ncclStream);
3695: 
3696:   if (enqueue) {
3697:     workEnqueue(work);
3698:   }
3699: 
3700:   {
3701:     c10::cuda::CUDAMultiStreamGuard streamGuard(ncclStream);
3702:     std::vector<at::Device> devices{device};
3703:     work->future_ = c10::make_intrusive<at::ivalue::Future>(
3704:         c10::ListType::create(c10::TensorType::get()), devices);
3705: 
3706:     // Add a callback that runs profiling end callbacks. wrapCallback() in CUDA
3707:     // future blocks the stream this callback runs on the corresponding
3708:     // ncclEndEvents_ ensuring appropriate synchronization.
3709:     if (work->recordFunctionEndCallback_) {
3710:       work->future_->addCallback(
3711:           [work](at::ivalue::Future& /* unused */) {
3712:             work->recordFunctionEndCallback_();
3713:           },
3714:           // uses_future = false allows us to skip synchronization in
3715:           // ivalue::Future, but is only valid as long as the lambda doesn't use
3716:           // the "Future" argument.
3717:           /*uses_future=*/false);
3718:     }
3719:     // Mark the future as completed since coalesced operations complete
3720:     // immediately
```

- EN: Lines 3691-3720 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3691-3720 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3721-3750 / 第 3721-3750 行

```cpp
3721:     work->future_->markCompleted(at::IValue(std::vector<at::Tensor>{}));
3722:   }
3723: 
3724:   // Reset coalescing state
3725:   coalescing_state_ = 0;
3726:   coalescedComm_ = nullptr;
3727:   coalescedTensors_.clear();
3728:   // If in async mode, return work; otherwise, kernel is enqueued on current
3729:   // stream, no need to return work
3730:   return coalescedAsync_ ? work : nullptr;
3731: }
3732: 
3733: c10::intrusive_ptr<Work> ProcessGroupNCCL::endCoalescing() {
3734:   // Default OpType to COALESCED if not specified
3735:   return endCoalescing(OpType::COALESCED);
3736: }
3737: 
3738: void ProcessGroupNCCL::startTimeEstimate() {
3739:   groupStart();
3740: }
3741: 
3742: float ProcessGroupNCCL::endTimeEstimate() {
3743: #ifdef NCCL_SIM_INFO_INITIALIZER
3744:   ncclSimInfo_t simInfo = NCCL_SIM_INFO_INITIALIZER;
3745:   C10D_NCCL_CHECK(ncclGroupSimulateEnd(&simInfo), std::nullopt);
3746:   --ncclActiveGroupCounter_;
3747:   return simInfo.estimatedTime;
3748: #else
3749:   TORCH_CHECK(
3750:       false,
```

- EN: Lines 3721-3750 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `endCoalescing`, `groupStart`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3721-3750 行使用条件编译来适配特性开关、平台或可选后端；在 `endCoalescing`、`groupStart` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3751-3780 / 第 3751-3780 行

```cpp
3751:       c10::str(
3752:           "The current nccl version does not support nccl comm time estimation. "));
3753: #endif
3754: }
3755: 
3756: template <typename Fn, typename PreProcess, typename PostProcess>
3757: c10::intrusive_ptr<Work> ProcessGroupNCCL::collective(
3758:     std::vector<at::Tensor>& inputs,
3759:     std::vector<at::Tensor>& outputs,
3760:     Fn fn,
3761:     PreProcess pre,
3762:     PostProcess post,
3763:     OpType opType,
3764:     bool asyncOp,
3765:     const char* profilingTitle,
3766:     bool nanCheck) {
3767:   // Environment setting by the user may add onto collective call's option
3768:   nanCheck &= enableNanCheck_;
3769: 
3770:   auto device = getDevice(inputs[0]);
3771:   // Guard must be created before `currentStreamCaptureStatusMayInitCtx`;
3772:   // otherwise, extra CUDA context could be created on device 0.
3773:   at::cuda::OptionalCUDAGuard gpuGuard(device);
3774: 
3775:   c10::cuda::CaptureStatus capture_status =
3776:       c10::cuda::currentStreamCaptureStatusMayInitCtx();
3777:   errorIfCapturingNonCapturableNCCL(capture_status);
3778: 
3779:   // Bump collective counter
3780:   if (!coalescing_state_) {
```

- EN: Lines 3751-3780 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3751-3780 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3781-3810 / 第 3781-3810 行

```cpp
3781:     seqCollective_++;
3782:   }
3783:   op_id_++;
3784: 
3785:   const auto key = getKeyFromDevice(device);
3786:   std::shared_ptr<NCCLComm> ncclComm = getNCCLComm(key);
3787:   if (ncclComm == nullptr) {
3788:     ncclComm = initNCCLComm(key, device, opType);
3789:   }
3790: 
3791:   if (coalescing_state_ & CoalActive) {
3792:     if ((coalescing_state_ & CoalColl) == 0) {
3793:       // First op in coalesced operations
3794:       seqCollective_++;
3795:     }
3796:     coalescing_state_ |= CoalColl;
3797:     if (coalescedDevice_.index() < 0) {
3798:       coalescedDevice_ = device;
3799:     } else {
3800:       TORCH_CHECK(
3801:           coalescedDevice_.index() == device.index(), MULTI_DEVICE_ERROR_MSG);
3802:     }
3803:     if (coalescedComm_ == nullptr) {
3804:       coalescedComm_ = ncclComm;
3805:     } else {
3806:       TORCH_CHECK(coalescedComm_ == ncclComm, MULTI_DEVICE_ERROR_MSG);
3807:     }
3808:     coalescedAsync_ = asyncOp;
3809:   }
3810: 
```

- EN: Lines 3781-3810 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3781-3810 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 3811-3840 / 第 3811-3840 行

```cpp
3811:   // in asyncOp=false [default] mode, we use currentStream as ncclStream
3812:   // otherwise, we use separate ncclStream and let it sync on currentStream
3813:   auto ncclStream = asyncOp ? ncclStreams_.at(key)
3814:                             : at::cuda::getCurrentCUDAStream(device.index());
3815:   if (asyncOp) {
3816:     // First let NCCL streams wait for input tensors allocation streams
3817:     syncStream(device, ncclEvents_[key], ncclStream);
3818:   }
3819: 
3820:   bool enqueue =
3821:       !coalescing_state_ && capture_status == c10::cuda::CaptureStatus::None;
3822:   auto work = initWork(
3823:       device, rank_, opType, false, profilingTitle, inputs, outputs, enqueue);
3824:   if (coalescing_state_) {
3825:     // When coalescing, we record events per op that lack timing/state
3826:     // information because there is no 'work' associated with them, and then
3827:     // later in endCoalescing we record a 'coalesced' Work which has
3828:     // timing/state updates via watchdog thread, but lacks op metadata such as
3829:     // input/output sizes and profilingTitle per-op in the group.
3830:     FlightRecorderCUDA::get()->recordWithResetEnabled(
3831:         local_id_,
3832:         std::make_tuple(pg_uid_, pg_desc_),
3833:         seqCollective_,
3834:         seqP2P_,
3835:         op_id_,
3836:         profilingTitle,
3837:         inputs,
3838:         outputs,
3839:         nullptr,
3840:         nullptr,
```

- EN: Lines 3811-3840 introduces executable logic in routines such as `syncStream`.
- CN: 第 3811-3840 行在 `syncStream` 等例程中引入具体执行逻辑。

### Lines 3841-3870 / 第 3841-3870 行

```cpp
3841:         options_->timeout,
3842:         pgStatus_,
3843:         /*isP2P=*/false);
3844:   }
3845: 
3846:   // Store references to outputs to be used by WorkNCCL::result and operator<<.
3847:   work->outputs_ = std::make_shared<std::vector<at::Tensor>>(outputs);
3848: 
3849:   // If we are performing sync operations, i.e. equeuing kernel onto "current"
3850:   // stream, we don't need to do anything for tensor lifetime management.
3851:   // Otherwise, we need to stage the tensors will `work.wait()`.
3852:   if (asyncOp) {
3853:     // First select which shelf to stash onto: to `work` if single collective;
3854:     // to an inflight shelf if coalescing.
3855:     if (coalescing_state_) {
3856:       coalescedTensors_.stash(inputs);
3857:       coalescedTensors_.stash(outputs);
3858:     } else {
3859:       work->stashed_for_allocator_safety_->stash(inputs);
3860:       work->stashed_for_allocator_safety_->stash(outputs);
3861:     }
3862:   }
3863: 
3864:   if (nanCheck) {
3865:     at::cuda::CUDAStreamGuard guard(ncclStream);
3866:     for (const auto& input : inputs) {
3867:       checkForNan(input);
3868:     }
3869:   }
3870: 
```

- EN: Lines 3841-3870 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3841-3870 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3871-3900 / 第 3871-3900 行

```cpp
3871:   // Start event should only be recorded before the ncclGroupStart()
3872:   if (work->timingEnabled_ && !coalescing_state_) {
3873:     work->ncclStartEvent_->record(ncclStream);
3874:   }
3875: 
3876:   pre(ncclStream, work);
3877: 
3878:   ncclComm_t comm = ncclComm->getNcclComm();
3879: 
3880:   // Both `inputs' and `outputs' are created on a worker stream and used in
3881:   // different ncclStreams.  Hence, both must record the ncclStream to
3882:   // prevent being freed before the collective finishes.
3883:   //
3884:   // We only record `inputs' here, and leave recording `outputs' to `fn' for
3885:   // operations where `inputs' and `outputs' are not the same.
3886:   //
3887:   // See [Sync Streams].
3888: 
3889: // Not all collectives have the same signature, e.g, all-reduce take in a Tensor
3890: // as the input and output while all-to-all take in a vector of Tensors as input
3891: // and output. Because we define the signature of the fn to take only single
3892: // tensor as input and output, we need to do a hack to get the first element in
3893: // the vector and pass it to fn.
3894: // TODO: we should clean up this in future (by either entirely removing lambda's
3895: // or removing input and output from lambda's signature).
3896: #ifndef NCCL_HAS_COMM_NONBLOCKING
3897:   C10D_NCCL_CHECK(
3898:       fn(inputs[0], outputs[0], comm, ncclStream),
3899:       ncclComm->getNcclCommFailureReason());
3900: #else
```

- EN: Lines 3871-3900 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3871-3900 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3901-3930 / 第 3901-3930 行

```cpp
3901:   C10D_NCCL_CHECK_TIMEOUT(
3902:       fn(inputs[0], outputs[0], comm, ncclStream),
3903:       ncclComm,
3904:       ncclComm->getNcclCommFailureReason());
3905: #endif // NCCL_HAS_COMM_NONBLOCKING
3906: 
3907:   post(ncclStream, work);
3908: 
3909:   // End event should only be recorded after the ncclGroupEnd()
3910:   if (!coalescing_state_) {
3911:     work->ncclEndEvent_->record(ncclStream);
3912:   }
3913:   work->ncclComm_ = ncclComm;
3914: 
3915:   {
3916:     c10::cuda::CUDAMultiStreamGuard streamGuard(ncclStream);
3917:     std::vector<at::Device> devices{device};
3918:     work->future_ = c10::make_intrusive<at::ivalue::Future>(
3919:         c10::ListType::create(c10::TensorType::get()), devices);
3920: 
3921:     // Add a callback that runs profiling end callbacks. wrapCallback() in CUDA
3922:     // future blocks the stream this callback runs on the corresponding
3923:     // ncclEndEvents_ ensuring appropriate synchronization.
3924:     if (work->recordFunctionEndCallback_) {
3925:       work->future_->addCallback(
3926:           [work](at::ivalue::Future& /* unused */) {
3927:             work->recordFunctionEndCallback_();
3928:           },
3929:           // uses_future = false allows us to skip synchronization in
3930:           // ivalue::Future, but is only valid as long as the lambda doesn't use
```

- EN: Lines 3901-3930 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3901-3930 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3931-3960 / 第 3931-3960 行

```cpp
3931:           // the "Future" argument.
3932:           /*uses_future=*/false);
3933:     }
3934:     work->future_->markCompleted(at::IValue(*work->outputs_));
3935:   }
3936: 
3937:   // Set appropriate work parameters.
3938:   work->blockingWait_ = blockingWait_;
3939:   work->store_ = store_;
3940:   assignTimeoutToWork(work, options_);
3941:   // Record size info for debug. We only record the size on the first device as
3942:   // multi-device per process is deprecated
3943:   work->numelIn_ = 0;
3944:   work->numelOut_ = 0;
3945:   for (const auto& input : inputs) {
3946:     work->numelIn_ += input.numel();
3947:   }
3948:   for (const auto& output : outputs) {
3949:     work->numelOut_ += output.numel();
3950:   }
3951: 
3952:   if (enqueue) {
3953:     workEnqueue(work);
3954:   }
3955: 
3956:   return asyncOp ? work : nullptr;
3957: }
3958: 
3959: template <typename Fn>
3960: c10::intrusive_ptr<Work> ProcessGroupNCCL::collectiveCoalesced(
```

- EN: Lines 3931-3960 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 3931-3960 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 3961-3990 / 第 3961-3990 行

```cpp
3961:     std::vector<at::Tensor>& inputs,
3962:     std::vector<at::Tensor>& outputs,
3963:     Fn fn,
3964:     OpType opType,
3965:     bool asyncOp,
3966:     const char* profilingTitle) {
3967:   // Currently, the API permits one scenario where inputs.size() and
3968:   // outputs.size() are > 0.
3969:   // 1. If the call was a _coalesced call, all inputs must be on the same
3970:   // device.
3971:   //    The group of nccl calls applies the collective separately to each input,
3972:   //    but the group as a whole should be efficient, and might even execute as
3973:   //    a single fused kernel.
3974:   auto device = getDevice(inputs[0]);
3975:   // Guard must be created before `currentStreamCaptureStatusMayInitCtx`;
3976:   // otherwise, extra CUDA context could be created on device 0.
3977:   at::cuda::OptionalCUDAGuard gpuGuard(device);
3978: 
3979:   c10::cuda::CaptureStatus capture_status =
3980:       c10::cuda::currentStreamCaptureStatusMayInitCtx();
3981:   errorIfCapturingNonCapturableNCCL(capture_status);
3982: 
3983:   // Bump collective counter
3984:   seqCollective_++;
3985: 
3986:   // For coalescingManager collectives, there is no individual c++ call per
3987:   // collective so there is no flight record and we increment seqCollective_ and
3988:   // op_id_ together. Compare this to startCoalescing/endCoalescing flow where
3989:   // we increment either seqP2P_ or seqCollective_ once per group and increment
3990:   // op_id_ once per individual operation within the group
```

- EN: Lines 3961-3990 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 3961-3990 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 3991-4020 / 第 3991-4020 行

```cpp
3991:   op_id_++;
3992: 
3993:   const auto key = getKeyFromDevice(device);
3994:   std::shared_ptr<NCCLComm> ncclComm = getNCCLComm(key);
3995:   if (ncclComm == nullptr) {
3996:     ncclComm = initNCCLComm(key, device, opType);
3997:   }
3998: 
3999:   if (coalescing_state_ & CoalActive) {
4000:     coalescing_state_ |= CoalColl;
4001:     if (coalescedDevice_.index() < 0) {
4002:       coalescedDevice_ = device;
4003:     } else {
4004:       TORCH_CHECK(
4005:           coalescedDevice_.index() == device.index(), MULTI_DEVICE_ERROR_MSG);
4006:     }
4007:     if (coalescedComm_ == nullptr) {
4008:       coalescedComm_ = ncclComm;
4009:     } else {
4010:       TORCH_CHECK(coalescedComm_ == ncclComm, MULTI_DEVICE_ERROR_MSG);
4011:     }
4012:     coalescedAsync_ = asyncOp;
4013:   }
4014: 
4015:   // in asyncOp=false [default] mode, we use currentStream as ncclStream
4016:   // otherwise, we use separate ncclStream and let it sync on currentStream
4017:   auto ncclStream = asyncOp ? ncclStreams_.at(key)
4018:                             : at::cuda::getCurrentCUDAStream(device.index());
4019:   if (asyncOp) {
4020:     // First let NCCL streams wait for input tensors allocation streams
```

- EN: Lines 3991-4020 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 3991-4020 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 4021-4050 / 第 4021-4050 行

```cpp
4021:     syncStream(device, ncclEvents_[key], ncclStream);
4022:   }
4023: 
4024:   auto work = initWork(
4025:       device,
4026:       rank_,
4027:       opType,
4028:       false,
4029:       profilingTitle,
4030:       inputs,
4031:       outputs,
4032:       /*record=*/true);
4033: 
4034:   // Store references to outputs to be used by WorkNCCL::result and operator<<.
4035:   work->outputs_ = std::make_shared<std::vector<at::Tensor>>(outputs);
4036: 
4037:   // If we are performing sync operations, i.e. equeuing kernel onto "current"
4038:   // stream, we don't need to do anything for tensor lifetime management.
4039:   // Otherwise, we need to stage the tensors will `work.wait()`.
4040:   if (asyncOp) {
4041:     work->stashed_for_allocator_safety_->stash(inputs);
4042:     work->stashed_for_allocator_safety_->stash(outputs);
4043:   }
4044: 
4045:   // Start event should only be recorded before the ncclGroupStart() (which
4046:   // happens inside AutoNcclGroup guard below)
4047:   if (work->timingEnabled_) {
4048:     work->ncclStartEvent_->record(ncclStream);
4049:   }
4050: 
```

- EN: Lines 4021-4050 introduces executable logic in routines such as `syncStream`.
- CN: 第 4021-4050 行在 `syncStream` 等例程中引入具体执行逻辑。

### Lines 4051-4080 / 第 4051-4080 行

```cpp
4051:   ncclComm_t comm = ncclComm->getNcclComm();
4052: 
4053: // TODO(kwen2501): this should be moved to c10d tests, to qualify a NCCL
4054: // upgrade. Once a NCCL version is qualified, this code should not be needed at
4055: // runtime.
4056: #ifdef PGNCCL_ENABLE_HASH
4057:   if (enableCollectiveHashDebug_.load()) {
4058:     auto numel = getTensorsNumel(inputs);
4059:     auto hashValue = hashTensors(inputs);
4060:     PRINT_COLLECTIVE_HASH_SIGNATURE(
4061:         "input", opTypeToString(opType), numel, hashValue);
4062:   }
4063: #endif // PGNCCL_ENABLE_HASH
4064: 
4065:   {
4066:     torch::cuda::nccl::AutoNcclGroup nccl_group_guard(comm, useNonblocking());
4067:     for (const auto i : c10::irange(inputs.size())) {
4068: #ifndef NCCL_HAS_COMM_NONBLOCKING
4069:       C10D_NCCL_CHECK(
4070:           fn(inputs[i], outputs[i], comm, ncclStream),
4071:           ncclComm->getNcclCommFailureReason());
4072: #else
4073:       C10D_NCCL_CHECK_TIMEOUT(
4074:           fn(inputs[i], outputs[i], comm, ncclStream),
4075:           ncclComm,
4076:           ncclComm->getNcclCommFailureReason());
4077: #endif // NCCL_HAS_COMM_NONBLOCKING
4078:     }
4079:   }
4080: 
```

- EN: Lines 4051-4080 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4051-4080 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4081-4110 / 第 4081-4110 行

```cpp
4081:   work->ncclEndEvent_->record(ncclStream);
4082:   work->ncclComm_ = ncclComm;
4083: 
4084:   {
4085:     c10::cuda::CUDAMultiStreamGuard streamGuard(ncclStream);
4086:     std::vector<at::Device> devices{device};
4087:     work->future_ = c10::make_intrusive<at::ivalue::Future>(
4088:         c10::ListType::create(c10::TensorType::get()), devices);
4089: 
4090:     // Add a callback that runs profiling end callbacks. wrapCallback() in CUDA
4091:     // future blocks the stream this callback runs on the corresponding
4092:     // ncclEndEvents_ ensuring appropriate synchronization.
4093:     if (work->recordFunctionEndCallback_) {
4094:       work->future_->addCallback(
4095:           [work](at::ivalue::Future& /* unused */) {
4096:             work->recordFunctionEndCallback_();
4097:           },
4098:           // uses_future = false allows us to skip synchronization in
4099:           // ivalue::Future, but is only valid as long as the lambda doesn't use
4100:           // the "Future" argument.
4101:           /*uses_future=*/false);
4102:     }
4103:     work->future_->markCompleted(at::IValue(*work->outputs_));
4104:   }
4105: 
4106:   // Set appropriate work parameters.
4107:   work->blockingWait_ = blockingWait_;
4108:   work->store_ = store_;
4109:   assignTimeoutToWork(work, options_);
4110:   // Record size info for debug. We only record the size on the first device as
```

- EN: Lines 4081-4110 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4081-4110 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4111-4140 / 第 4111-4140 行

```cpp
4111:   // multi-device per process is deprecated
4112:   work->numelIn_ = inputs[0].numel();
4113:   work->numelOut_ = outputs[0].numel();
4114: 
4115:   /* Note [cuda graph capture and workEnqueue]
4116: 
4117:   Normal behavior of the C10D watchdog is to query cuda events on work objects.
4118:   We disable this event query behavior during graph capture as it is disallowed
4119:   during capture under the strictest capture mode setting.
4120:   Note that previously recorded events (e.g., before the capture) can be queried
4121:   as the watchdog capture mode has been changed to thread-local, but user-side
4122:   event queries (from the main thread) via .is_completed() are still disallowed.
4123:   TODO(eqy): Is there a path to allowing workEnqueue during graph capture for
4124:   watchdog-thread usage only?
4125: 
4126:   TODO:
4127:    - Is our design for flight recorder safe in this context?  are we recording
4128:   any FR events during cudagraph capture? if so, they won't be safe to poll for
4129:   completion status.
4130:   */
4131:   if (capture_status == c10::cuda::CaptureStatus::None) {
4132:     workEnqueue(work);
4133:   }
4134:   // TODO(whc) if the work isn't enqueued, I don't feel great about returning
4135:   // it, since interactions with it by usercode won't behave normally - they
4136:   // won't observe work completion, for instance.  Will this lead to silent
4137:   // problems during capture?
4138:   return asyncOp ? work : nullptr;
4139: }
4140: 
```

- EN: Lines 4111-4140 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4111-4140 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4141-4170 / 第 4141-4170 行

```cpp
4141: template <typename Fn, typename PreProcess, typename PostProcess>
4142: c10::intrusive_ptr<Work> ProcessGroupNCCL::pointToPoint(
4143:     at::Tensor& tensor,
4144:     Fn fn,
4145:     int peer,
4146:     OpType opType,
4147:     PreProcess pre,
4148:     PostProcess post,
4149:     const char* profilingTitle) {
4150:   // avoidRecordStreams_ note:
4151:   // send, recv, and irecv should be ok with avoidRecordStreams,
4152:   // However, for isend, I don't think the API requires the user
4153:   // to wait() on the returned handle, so ProcessGroupNCCL can't know
4154:   // when it's safe to release the input back to the allocator,
4155:   // and the present call has no way to know it's not an isend.
4156:   // Therefore, we warn and fall back to the typical recordStream logic.
4157:   // TODO( kwen2501 ): revisit this when we have a better solution.
4158:   auto device = getDevice(tensor);
4159:   at::cuda::OptionalCUDAGuard gpuGuard(device);
4160: 
4161:   std::string key;
4162:   int p2pRank = -1, p2pTargetRank = -1;
4163:   bool isSendRecvSelf = rank_ == peer;
4164:   // For batch_isend_irecv, ncclGroupStart() would be called upfront
4165:   bool batchP2P = ncclActiveGroupCounter_ > 0;
4166: 
4167:   std::shared_ptr<NCCLComm> ncclComm = nullptr;
4168:   if (this->eagerInit_) {
4169:     /* In eagerInit mode, reuse the parent comm.  Do not lazily create
4170:      * p2p communicators. */
```

- EN: Lines 4141-4170 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4141-4170 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4171-4200 / 第 4171-4200 行

```cpp
4171:     if (!batchP2P && showSerializationWarning_) {
4172:       TORCH_WARN_ONCE(c10::str(
4173:           logPrefix(),
4174:           "An unbatched P2P op (send/recv) was called on this ProcessGroup with size ",
4175:           groupRanks().size(),
4176:           ".  In eager initialization mode, unbatched P2P ops are treated as ",
4177:           "independent collective ops, and are thus serialized with ",
4178:           "all other ops on this ProcessGroup, including other P2P ",
4179:           "ops. To avoid serialization, either create additional ",
4180:           "independent ProcessGroups for the P2P ops or use batched ",
4181:           "P2P ops. You can squash this warning by setting the environment variable ",
4182:           "TORCH_NCCL_SHOW_EAGER_INIT_P2P_SERIALIZATION_WARNING to false."));
4183:     }
4184: 
4185:     key = getKeyFromDevice(device);
4186:     p2pRank = rank_;
4187:     p2pTargetRank = peer;
4188:     ncclComm = getNCCLComm(key);
4189: 
4190:     TORCH_INTERNAL_ASSERT(
4191:         ncclComm != nullptr,
4192:         "Parent communicator missing in eager initialization mode.");
4193: 
4194:     if (!coalescing_state_) {
4195:       // Bump P2P sequence number. Don't do so if it's a batch P2P, it will be
4196:       // bumped in `startCoalescing`.
4197:       seqP2P_++;
4198:     }
4199:   } else if (batchP2P) {
4200:     // TODO(whc) - unclear why we special-case batchP2P to avoid this path, but
```

- EN: Lines 4171-4200 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`.
- CN: 第 4171-4200 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑。

### Lines 4201-4230 / 第 4201-4230 行

```cpp
4201:     // I preserved this existing special case.
4202:     key = getKeyFromDevice(device);
4203:     p2pRank = rank_;
4204:     p2pTargetRank = peer;
4205:     ncclComm = getNCCLComm(key);
4206:   } else {
4207:     // We create special 2-rank communicators for each pair of
4208:     // send/recv ranks.  This limitation exists for two reasons: (1)
4209:     // we use a single stream per communicator, so if multiple
4210:     // unbatched p2p operations are issued on the same communicator,
4211:     // they would map to the same stream and thus would be serialized;
4212:     // and (2) Nvidia NCCL does not allow multiple p2p operations to
4213:     // be issued on the same communicator over different streams.
4214: 
4215:     TORCH_WARN_ONCE(
4216:         "An unbatched P2P op (send/recv) was called on this ",
4217:         "ProcessGroup with size ",
4218:         groupRanks().size(),
4219:         ".  In lazy initialization mode, this will result in a new 2-rank",
4220:         " NCCL communicator to be created.");
4221: 
4222:     key = getKeySendRecv(rank_, peer);
4223:     /* if we are creating a new comm, reset the p2pRank and
4224:      * p2pTargetRank to correspond to this new 2-process communicator */
4225:     p2pRank = rank_ <= peer ? 0 : 1;
4226:     p2pTargetRank = isSendRecvSelf ? 0 : 1 - p2pRank;
4227:     ncclComm = getNCCLComm(key);
4228: 
4229:     if (!coalescing_state_) {
4230:       // Bump P2P sequence number.
```

- EN: Lines 4201-4230 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4201-4230 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4231-4260 / 第 4231-4260 行

```cpp
4231:       seqP2P_++;
4232:     }
4233:   }
4234: 
4235:   // Bump the logical operation counter regardless of whether this op is
4236:   // coalesced or individual
4237:   op_id_++;
4238: 
4239:   if (ncclComm == nullptr) {
4240:     // ncclComm should never be a nullptr in eager init mode.
4241:     // For lazy init mode, isSendRecvSelf is only valid for non-batch
4242:     // point-to-point operations.  For batch operations, force the
4243:     // argument to be false.
4244:     ncclComm =
4245:         initNCCLComm(key, device, opType, p2pRank, isSendRecvSelf && !batchP2P);
4246:   }
4247: 
4248:   if (coalescing_state_ & CoalActive) {
4249:     // Bump  seqP2P_ once per coalesced group, not once per individual op.
4250:     if ((coalescing_state_ & CoalP2P) == 0) {
4251:       seqP2P_++;
4252:     }
4253:     coalescing_state_ |= CoalP2P;
4254:     if (coalescedDevice_.index() < 0) {
4255:       coalescedDevice_ = device;
4256:     } else {
4257:       TORCH_CHECK(
4258:           coalescedDevice_.index() == device.index(), MULTI_DEVICE_ERROR_MSG);
4259:     }
4260:     if (coalescedComm_ == nullptr) {
```

- EN: Lines 4231-4260 introduces executable logic in routines such as `initNCCLComm`; performs validation and error handling to keep distributed state consistent.
- CN: 第 4231-4260 行在 `initNCCLComm` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 4261-4290 / 第 4261-4290 行

```cpp
4261:       coalescedComm_ = ncclComm;
4262:     } else {
4263:       TORCH_CHECK(coalescedComm_ == ncclComm, MULTI_DEVICE_ERROR_MSG);
4264:     }
4265:     // For now, P2P ops are always put on internal stream
4266:     coalescedAsync_ = true;
4267:   }
4268: 
4269:   // Used many times below, so we stash the unordered_map lookup
4270:   auto ncclStream = ncclStreams_.at(key);
4271:   // First let NCCL streams wait for input tensors allocation streams
4272:   syncStream(device, ncclEvents_[key], ncclStream);
4273: 
4274:   // Work itself will create the CUDA events on all GPUs of tensors
4275:   c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL> work;
4276:   if (coalescing_state_) {
4277:     // When coalescing, we record events per op that lack timing/state
4278:     // information because there is no 'work' associated with them, and then
4279:     // later in endCoalescing we record a 'coalesced' Work which has
4280:     // timing/state updates via watchdog thread, but lacks op metadata such as
4281:     // input/output sizes and profilingTitle per-op in the group.
4282:     FlightRecorderCUDA::get()->record(
4283:         local_id_,
4284:         std::make_tuple(pg_uid_, pg_desc_),
4285:         seqCollective_,
4286:         seqP2P_,
4287:         op_id_,
4288:         profilingTitle,
4289:         {tensor},
4290:         {tensor},
```

- EN: Lines 4261-4290 introduces executable logic in routines such as `TORCH_CHECK`, `syncStream`; performs validation and error handling to keep distributed state consistent.
- CN: 第 4261-4290 行在 `TORCH_CHECK`、`syncStream` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 4291-4320 / 第 4291-4320 行

```cpp
4291:         nullptr,
4292:         nullptr,
4293:         options_->timeout,
4294:         pgStatus_,
4295:         /*isP2P=*/true);
4296:     // TODO(whc) if we want to make the per-p2p-op flightrecorder entries get
4297:     // their timings/states updated by proxy when the Work obj representing the
4298:     // coalesce group gets its update, we could accumulate these trace_ids
4299:     // together and ask FlightRecorder to take the update from one Work and
4300:     // apply it to multiple entries
4301:   } else {
4302:     // Store references to outputs to be used by WorkNCCL::result and
4303:     // operator<<. Note that these outputs are only valid for recv(), as send()
4304:     // does not modify the inputs but we still create these outputs for use
4305:     // cases such as profiling.
4306: 
4307:     work = initWork(
4308:         device,
4309:         rank_,
4310:         opType,
4311:         true,
4312:         profilingTitle,
4313:         {tensor},
4314:         {},
4315:         /*record=*/false);
4316:     // This bypasses something in Work() that crashes if {tensor} is given as
4317:     // output, not sure what
4318:     work->outputs_ = std::make_shared<std::vector<at::Tensor>>();
4319:     work->outputs_->push_back(tensor);
4320:     // TODO(whc) because we don't pass output {tensor} to initWork, we tell
```

- EN: Lines 4291-4320 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4291-4320 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4321-4350 / 第 4321-4350 行

```cpp
4321:     // initWork to not record, and then we manually call record passing all the
4322:     // information it wants.
4323:     auto traceId = FlightRecorderCUDA::get()->recordWithResetEnabled(
4324:         local_id_,
4325:         std::make_tuple(pg_uid_, pg_desc_),
4326:         seqCollective_,
4327:         seqP2P_,
4328:         op_id_,
4329:         profilingTitle,
4330:         {tensor},
4331:         {tensor},
4332:         work->ncclStartEvent_.get(),
4333:         work->ncclEndEvent_.get(),
4334:         options_->timeout,
4335:         pgStatus_,
4336:         /*isP2P=*/true);
4337:     work->trace_id_ = traceId.id;
4338:     work->trace_reset_epoch_ = traceId.reset_epoch;
4339:   }
4340: 
4341:   // Only check for NaN for send ops, for recv ops `tensor` can be a random
4342:   // placeholder
4343:   if (enableNanCheck_ && opType == OpType::SEND) {
4344:     at::cuda::CUDAStreamGuard guard(ncclStream);
4345:     checkForNan(tensor);
4346:   }
4347: 
4348:   if (!coalescing_state_) {
4349:     // Start event should only be recorded before the ncclGroupStart()
4350:     if (work->timingEnabled_) {
```

- EN: Lines 4321-4350 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4321-4350 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4351-4380 / 第 4351-4380 行

```cpp
4351:       work->ncclStartEvent_->record(ncclStream);
4352:     }
4353: 
4354:     pre(ncclStream, work);
4355:   }
4356: 
4357:   // Both send tensor and recv tensor are created on a worker stream and used
4358:   // in different ncclStreams.  Hence, both must record the ncclStream to
4359:   // prevent being freed before the collective finishes.
4360:   //
4361:   // See [Sync Streams].
4362:   c10::cuda::CUDACachingAllocator::recordStream(
4363:       tensor.storage().data_ptr(), ncclStream);
4364: 
4365:   // This part seems common to both p2p and coalesced-p2p usage?
4366:   ncclComm_t comm_ = ncclComm->getNcclComm();
4367: 
4368: #ifndef NCCL_HAS_COMM_NONBLOCKING
4369:   C10D_NCCL_CHECK(
4370:       fn(tensor, comm_, ncclStream, p2pTargetRank),
4371:       ncclComm->getNcclCommFailureReason());
4372: #else
4373:   // In non-blocking mode, we need to use ncclGroup semantics to ensure that the
4374:   // kernel is enqueued for single-P2P ops.  Otherwise, the event record below
4375:   // may not capture the kernel, leading to data corruption.
4376:   ncclGroupStart();
4377:   C10D_NCCL_CHECK_NONBLOCKING(
4378:       fn(tensor, comm_, ncclStream, p2pTargetRank), std::nullopt);
4379:   C10D_NCCL_CHECK_TIMEOUT_GROUPEND(
4380:       ncclGroupEnd(), ncclComm, ncclComm->getNcclCommFailureReason());
```

- EN: Lines 4351-4380 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `ncclGroupStart`.
- CN: 第 4351-4380 行使用条件编译来适配特性开关、平台或可选后端；在 `ncclGroupStart` 等例程中引入具体执行逻辑。

### Lines 4381-4410 / 第 4381-4410 行

```cpp
4381: #endif // NCCL_HAS_COMM_NONBLOCKING
4382: 
4383:   if (!coalescing_state_) {
4384:     post(ncclStream);
4385: 
4386:     // End event should only be recorded after the ncclGroupEnd()
4387:     work->ncclEndEvent_->record(ncclStream);
4388:     work->ncclComm_ = ncclComm;
4389:     work->blockingWait_ = blockingWait_;
4390:     work->store_ = store_;
4391:     assignTimeoutToWork(work, options_);
4392:     // Record size info for debug. We only record the size on the first device
4393:     // as multi-device per process is deprecated
4394:     work->numelIn_ = work->numelOut_ = tensor.numel();
4395: 
4396:     // Future only needs to be created and marked completed with outputs for
4397:     // recv(), but still create future for use cases such as profiling even for
4398:     // send().
4399:     {
4400:       c10::cuda::CUDAMultiStreamGuard streamGuard(ncclStream);
4401:       std::vector<at::Device> devices{device};
4402:       work->future_ = c10::make_intrusive<at::ivalue::Future>(
4403:           c10::ListType::create(c10::TensorType::get()), devices);
4404:       work->future_->markCompleted(at::IValue(*work->outputs_));
4405:     }
4406: 
4407:     // Add a callback that runs profiling end callbacks. wrapCallback() in CUDA
4408:     // future blocks the stream this callback runs on the corresponding
4409:     // ncclEndEvents_ ensuring appropriate synchronization.
4410:     if (work->recordFunctionEndCallback_) {
```

- EN: Lines 4381-4410 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 4381-4410 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 4411-4440 / 第 4411-4440 行

```cpp
4411:       work->future_->addCallback(
4412:           [work](at::ivalue::Future& /* unused */) {
4413:             work->recordFunctionEndCallback_();
4414:           },
4415:           // uses_future = false allows us to skip synchronization in
4416:           // ivalue::Future, but is only valid as long as the lambda doesn't use
4417:           // the "Future" argument.
4418:           /*uses_future=*/false);
4419:     }
4420:   }
4421: 
4422:   // Enqueue P2P op so that it can be cancelled by NCCL watchdog
4423:   c10::cuda::CaptureStatus capture_status =
4424:       c10::cuda::currentStreamCaptureStatusMayInitCtx();
4425: 
4426:   if (!coalescing_state_ && capture_status == c10::cuda::CaptureStatus::None) {
4427:     workEnqueue(work);
4428:   }
4429:   return work;
4430: }
4431: 
4432: template <typename Fn, typename PreProcess, typename PostProcess>
4433: c10::intrusive_ptr<Work> ProcessGroupNCCL::collective(
4434:     at::Tensor& input,
4435:     at::Tensor& output,
4436:     Fn fn,
4437:     PreProcess pre,
4438:     PostProcess post,
4439:     OpType opType,
4440:     bool asyncOp,
```

- EN: Lines 4411-4440 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4411-4440 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4441-4470 / 第 4441-4470 行

```cpp
4441:     const char* profilingTitle,
4442:     bool nanCheck) {
4443:   auto inputs = std::vector<at::Tensor>{input};
4444:   auto outputs = std::vector<at::Tensor>{output};
4445:   return collective(
4446:       inputs,
4447:       outputs,
4448:       fn,
4449:       pre,
4450:       post,
4451:       opType,
4452:       asyncOp,
4453:       profilingTitle,
4454:       nanCheck);
4455: }
4456: 
4457: template <typename Fn>
4458: c10::intrusive_ptr<Work> ProcessGroupNCCL::collective(
4459:     at::Tensor& input,
4460:     at::Tensor& output,
4461:     Fn fn,
4462:     OpType opType,
4463:     bool asyncOp,
4464:     const char* profilingTitle,
4465:     bool nanCheck) {
4466:   auto inputs = std::vector<at::Tensor>{input};
4467:   auto outputs = std::vector<at::Tensor>{output};
4468:   return collective(
4469:       inputs,
4470:       outputs,
```

- EN: Lines 4441-4470 returns computed state or forwards results to the surrounding caller.
- CN: 第 4441-4470 行返回计算结果，或将结果继续传递给外围调用方。

### Lines 4471-4500 / 第 4471-4500 行

```cpp
4471:       fn,
4472:       [](at::cuda::CUDAStream&,
4473:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
4474:       [](at::cuda::CUDAStream&,
4475:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
4476:       opType,
4477:       asyncOp,
4478:       profilingTitle,
4479:       nanCheck);
4480: }
4481: 
4482: template <typename Fn>
4483: c10::intrusive_ptr<Work> ProcessGroupNCCL::pointToPoint(
4484:     at::Tensor& tensor,
4485:     Fn fn,
4486:     int peer,
4487:     OpType opType,
4488:     const char* profilingTitle) {
4489:   return pointToPoint(
4490:       tensor,
4491:       fn,
4492:       peer,
4493:       opType,
4494:       [](at::cuda::CUDAStream&,
4495:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
4496:       [](at::cuda::CUDAStream&) {},
4497:       profilingTitle);
4498: }
4499: 
4500: c10::intrusive_ptr<Work> ProcessGroupNCCL::allreduce_sparse(
```

- EN: Lines 4471-4500 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4471-4500 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4501-4530 / 第 4501-4530 行

```cpp
4501:     std::vector<at::Tensor>& tensors,
4502:     const AllreduceOptions& opts) {
4503:   TORCH_CHECK(tensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
4504:   auto tensor = tensors.back();
4505:   TORCH_CHECK(
4506:       !isUnsupportedFloat8(tensor.scalar_type()),
4507:       "Unsupported Float8 type for NCCL reduction");
4508: #ifdef IS_NCCLX
4509:   tensor = tensor.coalesce();
4510:   at::Tensor outputTensor =
4511:       torch::zeros(tensor.sizes(), tensor.options().layout(torch::kStrided));
4512:   auto work = collective(
4513:       tensor,
4514:       outputTensor,
4515:       [&](at::Tensor& input,
4516:           at::Tensor& output,
4517:           ncclComm_t comm,
4518:           at::cuda::CUDAStream& stream) {
4519:         auto ncclDataType = getNcclDataType(input.scalar_type());
4520:         auto ncclReduceOp =
4521:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
4522:         auto indices = input.indices();
4523:         auto sizes = input.sizes();
4524:         int colSize = sizes[1];
4525:         auto rows = indices[0];
4526:         size_t blockCount = rows.sizes()[0];
4527:         auto recvIndices = indices[0] * colSize;
4528: 
4529:         // prevent output and recvIndices from being freed
4530:         // TODO: not changing the lifetime management of outputs this time,
```

- EN: Lines 4501-4530 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4501-4530 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4531-4560 / 第 4531-4560 行

```cpp
4531:         // revisit later
4532:         c10::cuda::CUDACachingAllocator::recordStream(
4533:             output.storage().data_ptr(), stream);
4534:         c10::cuda::CUDACachingAllocator::recordStream(
4535:             recvIndices.storage().data_ptr(), stream);
4536:         auto result = ncclAllReduceSparseBlock(
4537:             input._values().data_ptr(), // sendbuff
4538:             recvIndices.data_ptr<int64_t>(), // recv_indices
4539:             blockCount, // block_count
4540:             colSize, // block_length
4541:             output.data_ptr(), // recvbuff
4542:             output.numel(), // recv_count
4543:             ncclDataType,
4544:             ncclReduceOp,
4545:             comm,
4546:             stream.stream());
4547:         return result;
4548:       },
4549:       [](at::cuda::CUDAStream& ncclStream,
4550:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
4551:       [&](at::cuda::CUDAStream& ncclStream,
4552:           c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {
4553:         // Convert output tensors to sparse and back into tensors.
4554:         at::cuda::CUDAStreamGuard guard(ncclStream);
4555:         if (opts.sparseIndices.has_value()) {
4556:           tensor = at::sparse_coo_tensor(
4557:               opts.sparseIndices.value(), outputTensor, tensor.sizes());
4558:         } else {
4559:           tensor = outputTensor.to_sparse();
4560:         }
```

- EN: Lines 4531-4560 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4531-4560 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4561-4590 / 第 4561-4590 行

```cpp
4561:       },
4562:       OpType::_ALLREDUCE_SPARSE,
4563:       opts.asyncOp,
4564:       "nccl:all_reduce_sparse");
4565:   return work;
4566: #else
4567:   // If the nccl branch is not "exp" then we just error
4568:   C10_THROW_ERROR(
4569:       Error,
4570:       "NCCL does not support all_reduce with sparse tensors. Please use dense tensors instead.");
4571: #endif // IS_NCCLX
4572: }
4573: 
4574: c10::intrusive_ptr<Work> ProcessGroupNCCL::allreduce_impl(
4575:     at::Tensor& tensor,
4576:     const char* profilingTitle,
4577:     const AllreduceOptions& opts) {
4578:   return collective(
4579:       tensor,
4580:       tensor,
4581:       [&](at::Tensor& input,
4582:           at::Tensor& output,
4583:           ncclComm_t comm,
4584:           at::cuda::CUDAStream& stream) {
4585:         auto ncclDataType = getNcclDataType(input.scalar_type());
4586:         auto ncclReduceOp =
4587:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
4588:         return ncclAllReduce(
4589:             input.data_ptr(),
4590:             output.data_ptr(),
```

- EN: Lines 4561-4590 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 4561-4590 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 4591-4620 / 第 4591-4620 行

```cpp
4591:             input.numel(),
4592:             ncclDataType,
4593:             ncclReduceOp,
4594:             comm,
4595:             stream.stream());
4596:       },
4597:       OpType::ALLREDUCE,
4598:       opts.asyncOp,
4599:       profilingTitle);
4600: }
4601: 
4602: c10::intrusive_ptr<Work> ProcessGroupNCCL::allreduce(
4603:     std::vector<at::Tensor>& tensors,
4604:     const AllreduceOptions& opts) {
4605:   TORCH_CHECK(tensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
4606:   auto tensor = tensors.back();
4607:   if (tensor.is_complex()) {
4608:     TORCH_CHECK(
4609:         c10d::isComplexViewAsRealAllowed(opts.reduceOp),
4610:         "all_reduce does not support",
4611:         opts.reduceOp,
4612:         "on complex tensors");
4613:     tensor = at::view_as_real(tensor);
4614:   }
4615:   check_gpu_single_tensor(tensor);
4616: 
4617:   if (opts.reduceOp == ReduceOp::SUM) {
4618:     using namespace intra_node_comm;
4619:     if (intraNodeComm_ == nullptr && IntraNodeComm::isEnabled()) {
4620:       intraNodeComm_ = initIntraNodeComm();
```

- EN: Lines 4591-4620 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4591-4620 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4621-4650 / 第 4621-4650 行

```cpp
4621:     }
4622:     if (intraNodeComm_ != nullptr) {
4623:       auto algo = intraNodeComm_->selectAllReduceAlgo(tensor);
4624:       if (algo != intra_node_comm::AllReduceAlgo::NONE) {
4625:         intraNodeComm_->allReduce(tensor, algo);
4626:         return c10::make_intrusive<IntraNodeCommWork>();
4627:       }
4628:     }
4629:   }
4630:   TORCH_CHECK(
4631:       !isUnsupportedFloat8(tensor.scalar_type()),
4632:       "Unsupported Float8 type for NCCL reduction");
4633:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
4634:       std::make_tuple(
4635:           static_cast<int64_t>(seqCollective_) + 1,
4636:           false), // seq + 1 to match collective
4637:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
4638:       tensors, // inputTensors
4639:       tensors, // outputTensors
4640:       rank_, // rank
4641:       "allreduce", // collective name
4642:       tensor.numel(), // inNelems
4643:       tensor.numel(), // outNelems
4644:       tensor.scalar_type(), // dType
4645:       std::vector<int64_t>(), // inSplitSizes
4646:       std::vector<int64_t>(), // outSplitSizes
4647:       globalRankStart_, // globalRankStart_
4648:       globalRankStride_, // globalRankStride_
4649:       this->getSize(), // worldSize
4650:       opts.asyncOp); // is asynchronized op
```

- EN: Lines 4621-4650 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4621-4650 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4651-4680 / 第 4651-4680 行

```cpp
4651: 
4652:   // avoidRecordStreams_ note: collective() will stash tensors.
4653:   return allreduce_impl(tensor, "nccl:all_reduce", opts);
4654: }
4655: 
4656: c10::intrusive_ptr<Work> ProcessGroupNCCL::allreduce_coalesced(
4657:     std::vector<at::Tensor>& tensors,
4658:     const AllreduceCoalescedOptions& opts) {
4659:   auto total_numel = check_gpu_tensors_same_device(tensors);
4660:   TORCH_CHECK(
4661:       !isUnsupportedFloat8(tensors.back().scalar_type()),
4662:       "Unsupported Float8 type for NCCL reduction");
4663: 
4664:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
4665:       std::make_tuple(
4666:           static_cast<int64_t>(seqCollective_) + 1,
4667:           false), // seq + 1 to match collective and assume only one collective
4668:                   // in coalesced range
4669:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
4670:       tensors, // inputTensors
4671:       tensors, // outputTensors
4672:       rank_, // rank
4673:       "allreduce_coalesced", // collective name
4674:       total_numel, // inNelems
4675:       total_numel, // outNelems
4676:       tensors[0].scalar_type(), // dType
4677:       // I'm not sure what in,outSplitSizes mean here.
4678:       std::vector<int64_t>(), // inSplitSizes
4679:       std::vector<int64_t>(), // outSplitSizes
4680:       globalRankStart_, // globalRankStart_
```

- EN: Lines 4651-4680 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4651-4680 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4681-4710 / 第 4681-4710 行

```cpp
4681:       globalRankStride_, // globalRankStride_
4682:       this->getSize(), // worldSize
4683:       opts.asyncOp); // is asynchronized op
4684: 
4685:   // avoidRecordStreams_ note: collective() will stash tensors.
4686:   return collectiveCoalesced(
4687:       tensors,
4688:       tensors,
4689:       [&](at::Tensor& input,
4690:           at::Tensor& output,
4691:           ncclComm_t comm,
4692:           at::cuda::CUDAStream& stream) {
4693:         auto ncclDataType = getNcclDataType(input.scalar_type());
4694:         auto ncclReduceOp =
4695:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
4696:         return ncclAllReduce(
4697:             input.data_ptr(),
4698:             output.data_ptr(),
4699:             input.numel(),
4700:             ncclDataType,
4701:             ncclReduceOp,
4702:             comm,
4703:             stream.stream());
4704:       },
4705:       OpType::COALESCED,
4706:       opts.asyncOp,
4707:       "nccl:allreduce_coalesced");
4708: }
4709: 
4710: c10::intrusive_ptr<Work> ProcessGroupNCCL::broadcast(
```

- EN: Lines 4681-4710 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4681-4710 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4711-4740 / 第 4711-4740 行

```cpp
4711:     std::vector<at::Tensor>& tensors,
4712:     const BroadcastOptions& opts) {
4713:   TORCH_CHECK(tensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
4714:   auto tensor = tensors.back();
4715:   if (tensor.is_complex()) {
4716:     tensor = at::view_as_real(tensor);
4717:   }
4718:   check_gpu_single_tensor(tensor);
4719: 
4720:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
4721:       std::make_tuple(
4722:           static_cast<int64_t>(seqCollective_) + 1,
4723:           false), // seq + 1 to match collective
4724:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
4725:       tensors, // inputTensors
4726:       tensors, // outputTensors
4727:       opts.rootRank, // root rank
4728:       "broadcast", // collective name
4729:       tensor.numel(), // inNelems
4730:       tensor.numel(), // outNelems
4731:       tensor.scalar_type(), // dType
4732:       std::vector<int64_t>(), // inSplitSizes
4733:       std::vector<int64_t>(), // outSplitSizes
4734:       globalRankStart_, // globalRankStart_
4735:       globalRankStride_, // globalRankStride_
4736:       this->getSize(), // worldSize
4737:       opts.asyncOp); // is asynchronized op
4738: 
4739:   const auto root = opts.rootRank + opts.rootTensor;
4740:   bool nanCheck = (root == rank_);
```

- EN: Lines 4711-4740 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4711-4740 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4741-4770 / 第 4741-4770 行

```cpp
4741: 
4742:   // avoidRecordStreams_ note: collective() will stash tensors.
4743:   return collective(
4744:       tensor,
4745:       tensor,
4746:       [&](at::Tensor& input,
4747:           at::Tensor& output,
4748:           ncclComm_t comm,
4749:           at::cuda::CUDAStream& stream) {
4750:         return ncclBcast(
4751:             input.data_ptr(),
4752:             input.numel(),
4753:             getNcclDataType(input.scalar_type()),
4754:             static_cast<int>(root),
4755:             comm,
4756:             stream.stream());
4757:       },
4758:       OpType::BROADCAST,
4759:       opts.asyncOp,
4760:       "nccl:broadcast",
4761:       nanCheck);
4762: }
4763: 
4764: // _broadcast_oop adds an out-of-place broadcast in PGNCCL
4765: // Custom collectives may be implemented by coalescing broadcast operations
4766: // One use-case is implementing a vector all_gather (all_gather_v)
4767: // where unevenly sized inputs are gathered among participating ranks
4768: // Since all_gather provides an out-of-place API, an all_gather_v
4769: // semantic implemented inside pg_nccl.all_gather also needs to support
4770: // out-of-place, for which an out-of-place broadcast is required to be added
```

- EN: Lines 4741-4770 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4741-4770 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4771-4800 / 第 4771-4800 行

```cpp
4771: c10::intrusive_ptr<Work> ProcessGroupNCCL::_broadcast_oop(
4772:     at::Tensor& outputTensor,
4773:     at::Tensor& inputTensor,
4774:     const BroadcastOptions& opts) {
4775:   if (outputTensor.numel() != inputTensor.numel()) {
4776:     C10_THROW_ERROR(
4777:         ValueError,
4778:         "Tensor input and output of _broadcast_oop must have the same number of elements ");
4779:   }
4780:   const auto root = opts.rootRank + opts.rootTensor;
4781:   bool nanCheck = (root == rank_);
4782:   return collective(
4783:       inputTensor,
4784:       outputTensor,
4785:       [&](at::Tensor& input,
4786:           at::Tensor& output,
4787:           ncclComm_t comm,
4788:           at::cuda::CUDAStream& stream) {
4789:         return ncclBroadcast(
4790:             input.data_ptr(),
4791:             output.data_ptr(),
4792:             input.numel(),
4793:             getNcclDataType(input.scalar_type()),
4794:             static_cast<int>(root),
4795:             comm,
4796:             stream.stream());
4797:       },
4798:       OpType::BROADCAST,
4799:       opts.asyncOp,
4800:       "nccl:_broadcast_oop",
```

- EN: Lines 4771-4800 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 4771-4800 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 4801-4830 / 第 4801-4830 行

```cpp
4801:       nanCheck);
4802: }
4803: 
4804: c10::intrusive_ptr<Work> ProcessGroupNCCL::reduce(
4805:     std::vector<at::Tensor>& tensors,
4806:     const ReduceOptions& opts) {
4807:   TORCH_CHECK(tensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
4808:   auto tensor = tensors.back();
4809:   if (tensor.is_complex()) {
4810:     TORCH_CHECK(
4811:         c10d::isComplexViewAsRealAllowed(opts.reduceOp),
4812:         "reduce does not support",
4813:         opts.reduceOp,
4814:         "on complex tensors");
4815:     tensor = at::view_as_real(tensor);
4816:   }
4817:   check_gpu_single_tensor(tensor);
4818:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
4819:       std::make_tuple(
4820:           static_cast<int64_t>(seqCollective_) + 1,
4821:           false), // seq + 1 to match collective
4822:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
4823:       tensors, // inputTensors
4824:       tensors, // outputTensors
4825:       opts.rootRank, // root rank
4826:       "reduce", // collective name
4827:       tensor.numel(), // inNelems
4828:       tensor.numel(), // outNelems
4829:       tensor.scalar_type(), // dType
4830:       std::vector<int64_t>(), // inSplitSizes
```

- EN: Lines 4801-4830 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4801-4830 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4831-4860 / 第 4831-4860 行

```cpp
4831:       std::vector<int64_t>(), // outSplitSizes
4832:       globalRankStart_, // globalRankStart_
4833:       globalRankStride_, // globalRankStride_
4834:       this->getSize(), // worldSize
4835:       opts.asyncOp); // is asynchronized op
4836: 
4837:   // avoidRecordStreams_ note: collective() will stash tensors.
4838:   return collective(
4839:       tensor,
4840:       tensor,
4841:       [&](at::Tensor& input,
4842:           at::Tensor& output,
4843:           ncclComm_t comm,
4844:           at::cuda::CUDAStream& stream) {
4845:         const auto root = opts.rootRank + opts.rootTensor;
4846:         auto ncclDataType = getNcclDataType(input.scalar_type());
4847:         auto ncclReduceOp =
4848:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
4849:         return ncclReduce(
4850:             input.data_ptr(),
4851:             output.data_ptr(),
4852:             input.numel(),
4853:             ncclDataType,
4854:             ncclReduceOp,
4855:             static_cast<int>(root),
4856:             comm,
4857:             stream.stream());
4858:       },
4859:       OpType::REDUCE,
4860:       opts.asyncOp,
```

- EN: Lines 4831-4860 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4831-4860 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4861-4890 / 第 4861-4890 行

```cpp
4861:       "nccl:reduce");
4862: }
4863: 
4864: // _reduce_oop exposes an out-of-place reduce from PGNCCL
4865: // Custom collectives may be implemented by coalescing reduce operations
4866: // One use-case is implementing a vector reduce_scatter (reduce_scatter_v)
4867: // where inputs are reduced and scattered unevenly among participating ranks
4868: // Since reduce_scatter provides an out-of-place API, a reduce_scatter_v
4869: // semantic implemented inside pg_nccl.reduce_scatter also needs to support
4870: // out-of-place, for which an out-of-place reduce is required to be added
4871: c10::intrusive_ptr<Work> ProcessGroupNCCL::_reduce_oop(
4872:     at::Tensor& outputTensor,
4873:     at::Tensor& inputTensor,
4874:     const ReduceOptions& opts) {
4875:   if (outputTensor.numel() != inputTensor.numel()) {
4876:     C10_THROW_ERROR(
4877:         ValueError,
4878:         "Tensor input and output of _reduce_oop must have the same number of elements ");
4879:   }
4880:   return collective(
4881:       inputTensor,
4882:       outputTensor,
4883:       [&](at::Tensor& input,
4884:           at::Tensor& output,
4885:           ncclComm_t comm,
4886:           at::cuda::CUDAStream& stream) {
4887:         const auto root = opts.rootRank + opts.rootTensor;
4888:         const auto ncclDataType = getNcclDataType(input.scalar_type());
4889:         const auto ncclReduceOp =
4890:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
```

- EN: Lines 4861-4890 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 4861-4890 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 4891-4920 / 第 4891-4920 行

```cpp
4891:         return ncclReduce(
4892:             input.data_ptr(),
4893:             output.data_ptr(),
4894:             input.numel(),
4895:             ncclDataType,
4896:             ncclReduceOp,
4897:             (int)root,
4898:             comm,
4899:             stream.stream());
4900:       },
4901:       OpType::REDUCE,
4902:       opts.asyncOp,
4903:       "nccl:_reduce_oop");
4904: }
4905: 
4906: c10::intrusive_ptr<Work> ProcessGroupNCCL::allgather(
4907:     std::vector<std::vector<at::Tensor>>& outputTensors,
4908:     std::vector<at::Tensor>& inputTensors,
4909:     const AllgatherOptions& opts) {
4910:   TORCH_CHECK(inputTensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
4911:   auto inputTensor = inputTensors.back();
4912:   check_gpu_single_tensor(inputTensor);
4913:   auto outputTensors_ = outputTensors.back();
4914: 
4915:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
4916:       std::make_tuple(
4917:           static_cast<int64_t>(seqCollective_) + 1,
4918:           false), // seq + 1 to match collective
4919:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
4920:       inputTensors, // inputTensors
```

- EN: Lines 4891-4920 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 4891-4920 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 4921-4950 / 第 4921-4950 行

```cpp
4921:       outputTensors, // outputTensors
4922:       rank_, // rank
4923:       "all_gather", // collective name
4924:       inputTensor.numel(), // inNelems
4925:       inputTensor.numel() * // outNelems
4926:           this->getSize(),
4927:       inputTensor.scalar_type(), // dType
4928:       std::vector<int64_t>(), // inSplitSizes
4929:       std::vector<int64_t>(), // outSplitSize
4930:       globalRankStart_, // globalRankStart_
4931:       globalRankStride_, // globalRankStride_
4932:       this->getSize(), // worldSize
4933:       opts.asyncOp); // is asynchronized op
4934: 
4935:   bool same_size = check_same_size(outputTensors_);
4936:   if (same_size) {
4937:     // Flatten a vector of tensors into a single, stacked tensor.
4938:     // we can handle only contiguous inputs, because we are
4939:     // just sending ptr and numel to nccl
4940:     inputTensor = inputTensor.contiguous();
4941:     at::Tensor outputFlattened = newLikeFlat(outputTensors_);
4942: 
4943:     return collective(
4944:         inputTensor,
4945:         outputFlattened,
4946:         [&](at::Tensor& input,
4947:             at::Tensor& output,
4948:             ncclComm_t comm,
4949:             at::cuda::CUDAStream& stream) {
4950:           // See [We actually don't need to stash anything here].
```

- EN: Lines 4921-4950 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4921-4950 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4951-4980 / 第 4951-4980 行

```cpp
4951:           return ncclAllGather(
4952:               input.data_ptr(),
4953:               output.data_ptr(),
4954:               input.numel(),
4955:               getNcclDataType(input.scalar_type()),
4956:               comm,
4957:               stream.stream());
4958:         },
4959:         [](at::cuda::CUDAStream& ncclStream,
4960:            c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {
4961:           // avoidRecordStreams_ note: We actually don't need to stash anything
4962:           // here.
4963:           //  - inputTensors is stashed onto work->stashed_for_allocator_safety_
4964:           //    in collective().
4965:           //  - outputFlattened is stashed onto work->outputs_ in collective().
4966:         },
4967:         [&](at::cuda::CUDAStream& ncclStream,
4968:             c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {
4969:           // User-facing outputTensors should be held by the user until after
4970:           // waiting on work_, or the call makes no sense. We do a stashing here
4971:           // in case user doesn't hold the outputTensors in downstream code,
4972:           // which can cause an early recycle by the CachingAllocator, which can
4973:           // lead to segfault or data corruption.
4974:           if (opts.asyncOp) {
4975:             work->stashed_for_allocator_safety_->stash(outputTensors_);
4976:           }
4977:           // Copy the flattened output tensors to the outputs.
4978:           at::cuda::CUDAStreamGuard guard(ncclStream);
4979:           for (const auto j : c10::irange(outputTensors_.size())) {
4980:             // See [We actually don't need to stash anything here].
```

- EN: Lines 4951-4980 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 4951-4980 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 4981-5010 / 第 4981-5010 行

```cpp
4981:             outputTensors_[j].copy_(
4982:                 outputFlattened[static_cast<int64_t>(j)], true);
4983:           }
4984:         },
4985:         OpType::ALLGATHER,
4986:         opts.asyncOp,
4987:         "nccl:all_gather");
4988:   } else {
4989:     const auto num_reduces = outputTensors_.size();
4990:     startCoalescing();
4991:     for (const int64_t i : c10::irange(static_cast<int64_t>(num_reduces))) {
4992:       auto& output = outputTensors_[i];
4993:       auto& input = (i == rank_) ? inputTensor : output;
4994:       auto broadcastOpts =
4995:           BroadcastOptions{i, int64_t(0), opts.timeout, opts.asyncOp};
4996:       _broadcast_oop(output, input, broadcastOpts);
4997:     }
4998:     auto work = endCoalescing(OpType::ALLGATHER);
4999:     return work;
5000:   }
5001: }
5002: 
5003: c10::intrusive_ptr<Work> ProcessGroupNCCL::allgather_coalesced(
5004:     std::vector<std::vector<at::Tensor>>& /* unused */,
5005:     std::vector<at::Tensor>& /* unused */,
5006:     const AllgatherOptions& /* unused */) {
5007:   C10_THROW_ERROR(
5008:       NotImplementedError,
5009:       "ProcessGroupNCCL does not support allgather_coalesced");
5010: }
```

- EN: Lines 4981-5010 introduces executable logic in routines such as `startCoalescing`, `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 4981-5010 行在 `startCoalescing`、`C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5011-5040 / 第 5011-5040 行

```cpp
5011: 
5012: c10::intrusive_ptr<Work> ProcessGroupNCCL::allgather_into_tensor_coalesced(
5013:     std::vector<at::Tensor>& outputs,
5014:     std::vector<at::Tensor>& inputs,
5015:     const AllgatherOptions& opts) {
5016:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5017:       std::make_tuple(
5018:           static_cast<int64_t>(seqCollective_) + 1,
5019:           false), // seq + 1 to match collective and assume only one collective
5020:                   // in coalesced range
5021:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5022:       inputs, // inputTensors
5023:       outputs, // outputTensors
5024:       rank_, // rank
5025:       "allgather_into_tensor_coalesced", // collective name
5026:       getTensorsNumel(inputs), // inNelems
5027:       getTensorsNumel(outputs), // outNelems
5028:       inputs[0].scalar_type(), // dType
5029:       std::vector<int64_t>(), // inSplitSizes
5030:       std::vector<int64_t>(), // outSplitSizes
5031:       globalRankStart_, // globalRankStart_
5032:       globalRankStride_, // globalRankStride_
5033:       this->getSize(), // worldSize
5034:       opts.asyncOp); // is asynchronized op
5035: 
5036:   return collectiveCoalesced(
5037:       inputs,
5038:       outputs,
5039:       [&](at::Tensor& input,
5040:           at::Tensor& output,
```

- EN: Lines 5011-5040 returns computed state or forwards results to the surrounding caller.
- CN: 第 5011-5040 行返回计算结果，或将结果继续传递给外围调用方。

### Lines 5041-5070 / 第 5041-5070 行

```cpp
5041:           ncclComm_t comm,
5042:           at::cuda::CUDAStream& stream) {
5043:         return ncclAllGather(
5044:             input.data_ptr(),
5045:             output.data_ptr(),
5046:             input.numel(),
5047:             getNcclDataType(input.scalar_type()),
5048:             comm,
5049:             stream.stream());
5050:       },
5051:       OpType::COALESCED,
5052:       opts.asyncOp,
5053:       "nccl:all_gather_into_tensor_coalesced");
5054: }
5055: 
5056: c10::intrusive_ptr<Work> ProcessGroupNCCL::reduce_scatter(
5057:     std::vector<at::Tensor>& outputTensors,
5058:     std::vector<std::vector<at::Tensor>>& inputTensors,
5059:     const ReduceScatterOptions& opts) {
5060:   TORCH_CHECK(outputTensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
5061:   auto outputTensor = outputTensors.back();
5062:   check_gpu_single_tensor(outputTensor);
5063:   auto inputTensors_ = inputTensors.back();
5064:   TORCH_CHECK(
5065:       !isUnsupportedFloat8(outputTensor.scalar_type()),
5066:       "Unsupported Float8 type for NCCL reduction");
5067: 
5068:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5069:       std::make_tuple(
5070:           static_cast<int64_t>(seqCollective_) + 1,
```

- EN: Lines 5041-5070 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 5041-5070 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 5071-5100 / 第 5071-5100 行

```cpp
5071:           false), // seq + 1 to match collective
5072:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5073:       inputTensors, // inputTensors
5074:       outputTensors, // outputTensors
5075:       rank_, // rank
5076:       "reduce_scatter", // collective name
5077:       outputTensor.numel() * this->getSize(), // inNelems
5078:       outputTensor.numel(), // outNelems
5079:       outputTensor.scalar_type(), // dType
5080:       std::vector<int64_t>(), // inSplitSizes
5081:       std::vector<int64_t>(), // outSplitSizes
5082:       globalRankStart_, // globalRankStart_
5083:       globalRankStride_, // globalRankStride_
5084:       this->getSize(), // worldSize
5085:       opts.asyncOp); // is asynchronized op
5086: 
5087:   bool same_size = check_same_size(inputTensors_);
5088:   if (same_size) {
5089:     // Flatten a vector of tensors into a single, stacked tensor.
5090:     outputTensor = outputTensor.contiguous();
5091:     at::Tensor inputFlattened = newLikeFlat(inputTensors_);
5092: 
5093:     return collective(
5094:         inputFlattened,
5095:         outputTensor,
5096:         [&](at::Tensor& input,
5097:             at::Tensor& output,
5098:             ncclComm_t comm,
5099:             at::cuda::CUDAStream& stream) {
5100:           // TODO: remove once upstream NCCL is fixed
```

- EN: Lines 5071-5100 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5071-5100 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5101-5130 / 第 5101-5130 行

```cpp
5101:           // https://github.com/pytorch/pytorch/issues/168092
5102:           if (this->getSize() == 1) {
5103:             at::cuda::CUDAStreamGuard guard(stream);
5104:             output.flatten().copy_(input.flatten(), true);
5105:             return ncclSuccess;
5106:           }
5107: 
5108:           const auto ncclDataType = getNcclDataType(input.scalar_type());
5109:           const auto ncclReduceOp =
5110:               getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
5111:           return ncclReduceScatter(
5112:               input.data_ptr(),
5113:               output.data_ptr(),
5114:               output.numel(),
5115:               ncclDataType,
5116:               ncclReduceOp,
5117:               comm,
5118:               stream.stream());
5119:         },
5120:         [&](at::cuda::CUDAStream& ncclStream,
5121:             c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {
5122:           // We only need to stash inputTensors.
5123:           //  - inputFlattened is stashed onto
5124:           //  work->stashed_for_allocator_safety_ in collective().
5125:           //  - User-facing outputTensors is stashed onto work->outputs_ in
5126:           //  collective(), and should also be held by the user until after
5127:           //  waiting on work_.
5128:           if (opts.asyncOp) {
5129:             work->stashed_for_allocator_safety_->stash(inputTensors_);
5130:           }
```

- EN: Lines 5101-5130 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5101-5130 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5131-5160 / 第 5131-5160 行

```cpp
5131:           // Copy the input tensors to the flattened inputs.
5132:           at::cuda::CUDAStreamGuard guard(ncclStream);
5133:           for (const auto j : c10::irange(inputTensors_.size())) {
5134:             inputFlattened[static_cast<int64_t>(j)].copy_(
5135:                 inputTensors_[j], true);
5136:           }
5137:         },
5138:         [&](at::cuda::CUDAStream&,
5139:             c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5140:         OpType::REDUCE_SCATTER,
5141:         opts.asyncOp,
5142:         "nccl:reduce_scatter");
5143:   } else {
5144:     const auto num_reduces = inputTensors_.size();
5145:     startCoalescing();
5146:     for (const int i : c10::irange(static_cast<int>(num_reduces))) {
5147:       auto& input = inputTensors_[i];
5148:       auto& output = (i == rank_) ? outputTensor : input;
5149:       auto reduceOpts = ReduceOptions{
5150:           opts.reduceOp,
5151:           static_cast<int64_t>(i),
5152:           static_cast<int64_t>(0),
5153:           opts.timeout,
5154:           opts.asyncOp};
5155:       _reduce_oop(output, input, reduceOpts);
5156:     }
5157:     auto work = endCoalescing(OpType::REDUCE_SCATTER);
5158:     return work;
5159:   }
5160: }
```

- EN: Lines 5131-5160 introduces executable logic in routines such as `startCoalescing`; returns computed state or forwards results to the surrounding caller.
- CN: 第 5131-5160 行在 `startCoalescing` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5161-5190 / 第 5161-5190 行

```cpp
5161: 
5162: c10::intrusive_ptr<Work> ProcessGroupNCCL::_reduce_scatter_base(
5163:     at::Tensor& outputTensor,
5164:     at::Tensor& inputTensor,
5165:     const ReduceScatterOptions& opts) {
5166:   if (inputTensor.dtype() != outputTensor.dtype()) {
5167:     C10_THROW_ERROR(
5168:         TypeError, "input tensor must be the same type as the output tensor.");
5169:   }
5170: 
5171:   if (inputTensor.numel() != outputTensor.numel() * size_) {
5172:     C10_THROW_ERROR(
5173:         ValueError,
5174:         "input tensor must be the same size as output size times world size");
5175:   }
5176: 
5177:   const auto& tensor = outputTensor;
5178:   TORCH_CHECK(
5179:       !isUnsupportedFloat8(tensor.scalar_type()),
5180:       "Unsupported Float8 type for NCCL reduction");
5181:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5182:       std::make_tuple(
5183:           static_cast<int64_t>(seqCollective_) + 1,
5184:           false), // seq + 1 to match collective
5185:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5186:       inputTensor, // inputTensor
5187:       outputTensor, // outputTensor
5188:       rank_, // rank
5189:       "_reduce_scatter_base", // collective name
5190:       inputTensor.numel(), // inNelems
```

- EN: Lines 5161-5190 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5161-5190 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5191-5220 / 第 5191-5220 行

```cpp
5191:       tensor.numel(), // outNelems
5192:       tensor.scalar_type(), // dtype
5193:       std::vector<int64_t>(), // inSplitSizes
5194:       std::vector<int64_t>(), // outSplitSizes
5195:       globalRankStart_, // globalRankStart_
5196:       globalRankStride_, // globalRankStride_
5197:       this->getSize(), // worldSize
5198:       opts.asyncOp); // is asynchronized op
5199: 
5200:   // avoidRecordStreams_ note: collective() will stash inputs and outputs.
5201:   // Note 2: for asyncOp = false, we don't want to record streams because we
5202:   // know that the NCCL stream will join back to the "current" stream right
5203:   // after this op. So we might just as well keep the stream ownership of the
5204:   // input/output tensors unchanged. The benefit would be that the
5205:   // allocation/free of the tensors would look deterministic to the "current"
5206:   // stream so that the caching allocator can reuse memory pool for this stream
5207:   // in a clever way. This setting is added for libraries like FSDP which uses
5208:   // `reduce_scatter_tensor`.
5209: 
5210:   return collective(
5211:       inputTensor,
5212:       outputTensor,
5213:       [&](at::Tensor& input,
5214:           at::Tensor& output,
5215:           ncclComm_t comm,
5216:           at::cuda::CUDAStream& stream) {
5217:         // TODO: remove once upstream NCCL is fixed
5218:         // https://github.com/pytorch/pytorch/issues/168092
5219:         if (this->getSize() == 1) {
5220:           at::cuda::CUDAStreamGuard guard(stream);
```

- EN: Lines 5191-5220 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5191-5220 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5221-5250 / 第 5221-5250 行

```cpp
5221:           output.flatten().copy_(input.flatten(), true);
5222:           return ncclSuccess;
5223:         }
5224: 
5225:         auto ncclDataType = getNcclDataType(input.scalar_type());
5226:         auto ncclReduceOp =
5227:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
5228:         return ncclReduceScatter(
5229:             input.data_ptr(),
5230:             output.data_ptr(),
5231:             output.numel(),
5232:             ncclDataType,
5233:             ncclReduceOp,
5234:             comm,
5235:             stream.stream());
5236:       },
5237:       OpType::_REDUCE_SCATTER_BASE,
5238:       opts.asyncOp,
5239:       "nccl:_reduce_scatter_base");
5240: }
5241: 
5242: c10::intrusive_ptr<Work> ProcessGroupNCCL::reduce_scatter_tensor_coalesced(
5243:     std::vector<at::Tensor>& outputs,
5244:     std::vector<at::Tensor>& inputs,
5245:     const ReduceScatterOptions& opts) {
5246:   TORCH_CHECK(
5247:       !isUnsupportedFloat8(inputs.back().scalar_type()),
5248:       "Unsupported Float8 type for NCCL reduction");
5249: 
5250:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
```

- EN: Lines 5221-5250 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 5221-5250 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 5251-5280 / 第 5251-5280 行

```cpp
5251:       std::make_tuple(
5252:           static_cast<int64_t>(seqCollective_) + 1,
5253:           false), // seq + 1 to match collective and assume only one collective
5254:                   // in coalesced range
5255:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5256:       inputs, // inputTensors
5257:       outputs, // outputTensors
5258:       rank_, // rank
5259:       "reduce_scatter_tensor_coalesced", // collective name
5260:       getTensorsNumel(inputs), // inNelems
5261:       getTensorsNumel(outputs), // outNelems
5262:       inputs[0].scalar_type(), // dType
5263:       std::vector<int64_t>(), // inSplitSizes
5264:       std::vector<int64_t>(), // outSplitSizes
5265:       globalRankStart_, // globalRankStart_
5266:       globalRankStride_, // globalRankStride_
5267:       this->getSize(), // worldSize
5268:       opts.asyncOp); // is asynchronized op
5269: 
5270:   return collectiveCoalesced(
5271:       inputs,
5272:       outputs,
5273:       [&](at::Tensor& input,
5274:           at::Tensor& output,
5275:           ncclComm_t comm,
5276:           at::cuda::CUDAStream& stream) {
5277:         // TODO: remove once upstream NCCL is fixed
5278:         // https://github.com/pytorch/pytorch/issues/168092
5279:         if (this->getSize() == 1) {
5280:           at::cuda::CUDAStreamGuard guard(stream);
```

- EN: Lines 5251-5280 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5251-5280 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5281-5310 / 第 5281-5310 行

```cpp
5281:           output.flatten().copy_(input.flatten(), true);
5282:           return ncclSuccess;
5283:         }
5284: 
5285:         auto ncclDataType = getNcclDataType(input.scalar_type());
5286:         auto ncclReduceOp =
5287:             getNcclReduceOp(opts.reduceOp, input, ncclDataType, comm);
5288:         return ncclReduceScatter(
5289:             input.data_ptr(),
5290:             output.data_ptr(),
5291:             output.numel(),
5292:             ncclDataType,
5293:             ncclReduceOp,
5294:             comm,
5295:             stream.stream());
5296:       },
5297:       OpType::COALESCED,
5298:       opts.asyncOp,
5299:       "nccl:reduce_scatter_tensor_coalesced");
5300: }
5301: 
5302: c10::DeviceIndex ProcessGroupNCCL::guessDeviceId() const {
5303:   // 1st choice: don't use this function if your API can take a device_id
5304:   // argument.
5305:   if (getBoundDeviceId().has_value()) {
5306:     // 2nd choice: Use the bound GPU device id if available.
5307:     // Bounded device id can be passed to `init_process_group`.
5308:     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
5309:     return getBoundDeviceId().value().index();
5310:   } else if (!usedDeviceIdxs_.empty()) {
```

- EN: Lines 5281-5310 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5281-5310 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5311-5340 / 第 5311-5340 行

```cpp
5311:     // 3rd choice: infer the device id from the used device ids.
5312:     return *usedDeviceIdxs_.begin();
5313:   }
5314:   // This means there is not yet a NCCL collective being called
5315:   // Here we have to use the best guesses and will use a single GPU to call
5316:   // allreduce to achieve barrier.
5317:   // In case the multiple processes fall into the same node, we use rank to
5318:   // ensure that each process is on a different GPU
5319:   // Note: it is better to use global rank because the group-local rank can be
5320:   // offset wrt the device id if intra-node GPUs are sharded into multiple
5321:   // dimensions.
5322:   int devIdx = globalRank() % localDeviceCount_;
5323:   if (devIdx == 0) { // only log on first rank of each node
5324:     LOG(WARNING) << c10::str(
5325:         "Guessing device ID based on global rank. ",
5326:         "This can cause a hang if rank to GPU mapping is heterogeneous. ",
5327:         "You can specify device_id in init_process_group()");
5328:   }
5329:   return static_cast<c10::DeviceIndex>(devIdx);
5330: }
5331: 
5332: c10::intrusive_ptr<Work> ProcessGroupNCCL::barrier(const BarrierOptions& opts) {
5333:   RECORD_PARAM_COMMS(
5334:       std::make_tuple(
5335:           static_cast<int64_t>(seqCollective_) + 1,
5336:           false), // seq + 1 to match collective
5337:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5338:       rank_, // rank
5339:       "barrier", // collective name
5340:       0, // inNelems
```

- EN: Lines 5311-5340 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5311-5340 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5341-5370 / 第 5341-5370 行

```cpp
5341:       0, // outNelems
5342:       at::kByte, // dType
5343:       std::vector<int64_t>(), // inSplitSizes
5344:       std::vector<int64_t>(), // outSplitSizes
5345:       globalRankStart_, // globalRankStart_
5346:       globalRankStride_, // globalRankStride_
5347:       this->getSize()); // worldSize
5348: 
5349:   // Device to use for barrier
5350:   c10::DeviceIndex barDevIdx = -1;
5351: 
5352:   // Select device to use for barrier
5353:   // 1st choice: Use user defined GPU device ids if provided
5354:   if (!opts.device_ids.empty()) {
5355:     // Use the first device id because PG NCCL is single-device now
5356:     barDevIdx = static_cast<c10::DeviceIndex>(opts.device_ids[0]);
5357:   } else {
5358:     // 2nd choice: Use the bound or used GPU device id if available.
5359:     barDevIdx = guessDeviceId();
5360:   }
5361: 
5362:   TORCH_CHECK_WITH(
5363:       ValueError,
5364:       barDevIdx >= 0,
5365:       "Failed to infer a GPU device id to perform barrier. ");
5366:   auto barDevice = at::Device(at::DeviceType::CUDA, barDevIdx);
5367: 
5368:   // Create a dummy tensor on the device
5369:   // Note: we use zeros() instead of empty() to prevent barrier from triggering
5370:   // alarm when NaN checker is enabled.
```

- EN: Lines 5341-5370 introduces executable logic in routines such as `TORCH_CHECK_WITH`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5341-5370 行在 `TORCH_CHECK_WITH` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5371-5400 / 第 5371-5400 行

```cpp
5371:   at::Tensor barrierTensor =
5372:       at::zeros({1}, at::TensorOptions().device(barDevice).dtype(at::kFloat));
5373: 
5374:   // All reduce to achieve the barrier
5375:   AllreduceOptions arOpts = AllreduceOptions();
5376:   arOpts.asyncOp = opts.asyncOp;
5377:   auto work = allreduce_impl(barrierTensor, "nccl:all_reduce_barrier", arOpts);
5378: 
5379:   if (opts.asyncOp) {
5380:     // Work will take over barrierTensors
5381:     auto ncclWork = dynamic_cast<ProcessGroupNCCL::WorkNCCL*>(work.get());
5382:     // If user specified async, the work should not be nullptr
5383:     TORCH_CHECK(ncclWork);
5384:     // Put a marker here so that `work.wait()` issue by users does
5385:     // barrier-specific thing: CPU sync
5386:     ncclWork->isBarrierOp_ = true;
5387:     return work;
5388:   }
5389: 
5390:   // Otherwise, we are in sync mode, we directly wait here.
5391:   // (It is a CPU wait for barrier)
5392:   auto currentStream = at::cuda::getCurrentCUDAStream(barDevIdx);
5393:   // CUDAStream wrapper will correctly use a DeviceGuard here
5394:   currentStream.synchronize();
5395:   // No work to return
5396:   return nullptr;
5397: }
5398: 
5399: c10::intrusive_ptr<Work> ProcessGroupNCCL::alltoall_base(
5400:     at::Tensor& outputTensor,
```

- EN: Lines 5371-5400 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 5371-5400 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 5401-5430 / 第 5401-5430 行

```cpp
5401:     at::Tensor& inputTensor,
5402:     std::vector<int64_t>& outputSplitSizes,
5403:     std::vector<int64_t>& inputSplitSizes,
5404:     const AllToAllOptions& opts) {
5405:   check_gpu_single_tensor(outputTensor);
5406:   check_gpu_single_tensor(inputTensor);
5407:   if (outputSplitSizes.empty() && inputSplitSizes.empty()) {
5408:     RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5409:         std::make_tuple(
5410:             static_cast<int64_t>(seqCollective_) + 1,
5411:             false), // seq + 1 to match collective
5412:         std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5413:         inputTensor, // inputTensor
5414:         outputTensor, // outputTensor
5415:         rank_, // rank
5416:         "all_to_allv", // collective name
5417:         inputTensor.numel(), // inNelems
5418:         outputTensor.numel(), // outNelems
5419:         inputTensor.scalar_type(), // dType
5420:         std::vector<int64_t>(), // inSplitSizes
5421:         std::vector<int64_t>(), // outSplitSizes
5422:         globalRankStart_, // globalRankStart_
5423:         globalRankStride_, // globalRankStride_
5424:         this->getSize(), // worldSize
5425:         opts.asyncOp); // is asynchronized op
5426: 
5427:     // avoidRecordStreams_ note: collective() will stash inputTensors and
5428:     // outputTensors.
5429:     return collective(
5430:         inputTensor,
```

- EN: Lines 5401-5430 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5401-5430 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5431-5460 / 第 5431-5460 行

```cpp
5431:         outputTensor,
5432:         [&](at::Tensor& input,
5433:             at::Tensor& output,
5434:             ncclComm_t comm,
5435:             at::cuda::CUDAStream& stream) {
5436:           torch::cuda::nccl::all2all_single_equal_split(
5437:               input, output, this->getSize(), comm, stream);
5438:           return ncclSuccess;
5439:         },
5440:         OpType::ALLTOALL_BASE,
5441:         opts.asyncOp,
5442:         "nccl:all_to_all");
5443:   } else {
5444:     c10d::checkSplitSizes(inputSplitSizes, inputTensor, size_);
5445:     c10d::checkSplitSizes(outputSplitSizes, outputTensor, size_);
5446: 
5447:     RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5448:         std::make_tuple(
5449:             static_cast<int64_t>(seqCollective_) + 1,
5450:             false), // seq + 1 to match collective
5451:         std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5452:         inputTensor, // inputTensor
5453:         outputTensor, // outputTensor
5454:         rank_, // rank
5455:         "all_to_allv", // collective name
5456:         inputTensor.numel(), // inNelems
5457:         outputTensor.numel(), // outNelems
5458:         inputTensor.scalar_type(), // dType
5459:         inputSplitSizes, // inSplitSizes
5460:         outputSplitSizes, // outSplitSizes
```

- EN: Lines 5431-5460 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5431-5460 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5461-5490 / 第 5461-5490 行

```cpp
5461:         globalRankStart_, // globalRankStart_
5462:         globalRankStride_, // globalRankStride_
5463:         this->getSize(), // worldSize
5464:         opts.asyncOp); // is asynchronized op
5465: 
5466:     // avoidRecordStreams_ note: collective() will stash inputTensors and
5467:     // outputTensors.
5468:     return collective(
5469:         inputTensor,
5470:         outputTensor,
5471:         [&](at::Tensor& input,
5472:             at::Tensor& output,
5473:             ncclComm_t comm,
5474:             at::cuda::CUDAStream& stream) {
5475:           std::vector<size_t> send_lengths(size_);
5476:           std::vector<size_t> recv_lengths(size_);
5477:           std::vector<size_t> send_offsets(size_);
5478:           std::vector<size_t> recv_offsets(size_);
5479:           c10d::computeLengthsAndOffsets(
5480:               inputSplitSizes, input, &send_lengths, &send_offsets);
5481:           c10d::computeLengthsAndOffsets(
5482:               outputSplitSizes, output, &recv_lengths, &recv_offsets);
5483:           // See [Sync Streams].
5484:           torch::cuda::nccl::all2all_single_unequal_split(
5485:               input.data_ptr(),
5486:               send_lengths.data(),
5487:               send_offsets.data(),
5488:               output.data_ptr(),
5489:               recv_lengths.data(),
5490:               recv_offsets.data(),
```

- EN: Lines 5461-5490 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5461-5490 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5491-5520 / 第 5491-5520 行

```cpp
5491:               input.element_size(),
5492:               input.scalar_type(),
5493:               comm,
5494:               stream);
5495:           return ncclSuccess;
5496:         },
5497:         OpType::ALLTOALL_BASE,
5498:         opts.asyncOp,
5499:         "nccl:all_to_all");
5500:   }
5501: }
5502: 
5503: c10::intrusive_ptr<Work> ProcessGroupNCCL::alltoall(
5504:     std::vector<at::Tensor>& outputTensors,
5505:     std::vector<at::Tensor>& inputTensors,
5506:     const AllToAllOptions& opts) {
5507:   int64_t input_total_numel = 0;
5508:   int64_t output_total_numel = 0;
5509:   // considering uneven all2all bw calculation
5510:   // use split sizes field to record tensor list sizes
5511:   std::vector<int64_t> inSplitSizes;
5512:   std::vector<int64_t> outSplitSizes;
5513: 
5514:   auto device = outputTensors[0].device();
5515:   for (const auto r : c10::irange(outputTensors.size())) {
5516:     check_gpu_single_tensor(outputTensors[r]);
5517:     check_gpu_single_tensor(inputTensors[r]);
5518:     TORCH_CHECK(
5519:         device == outputTensors[r].device() &&
5520:             device == inputTensors[r].device(),
```

- EN: Lines 5491-5520 introduces executable logic in routines such as `check_gpu_single_tensor`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5491-5520 行在 `check_gpu_single_tensor` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5521-5550 / 第 5521-5550 行

```cpp
5521:         "Tensors must be on the same device")
5522:     input_total_numel += inputTensors[r].numel();
5523:     output_total_numel += outputTensors[r].numel();
5524:     inSplitSizes.push_back(inputTensors[r].numel());
5525:     outSplitSizes.push_back(outputTensors[r].numel());
5526:   }
5527: 
5528:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5529:       std::make_tuple(
5530:           static_cast<int64_t>(seqCollective_) + 1,
5531:           false), // seq + 1 to match collective
5532:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5533:       inputTensors, // inputTensors
5534:       outputTensors, // outputTensors
5535:       rank_, // rank
5536:       "all_to_all", // collective name
5537:       input_total_numel, // inNelems
5538:       output_total_numel, // outNelems
5539:       inputTensors.front().scalar_type(), // dType
5540:       inSplitSizes, // inSplitSizes
5541:       outSplitSizes, // outSplitSizes
5542:       globalRankStart_, // globalRankStart_
5543:       globalRankStride_, // globalRankStride_
5544:       this->getSize(), // worldSize
5545:       opts.asyncOp); // is asynchronized op
5546: 
5547:   return collective(
5548:       inputTensors,
5549:       outputTensors,
5550:       [&](at::Tensor& /* unused */,
```

- EN: Lines 5521-5550 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5521-5550 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5551-5580 / 第 5551-5580 行

```cpp
5551:           at::Tensor& /* unused */,
5552:           ncclComm_t comm,
5553:           at::cuda::CUDAStream& stream) {
5554:         torch::cuda::nccl::all2all(outputTensors, inputTensors, comm, stream);
5555:         return ncclSuccess;
5556:       },
5557:       [&](at::cuda::CUDAStream&,
5558:           c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5559:       [](at::cuda::CUDAStream&,
5560:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5561:       OpType::ALLTOALL,
5562:       opts.asyncOp,
5563:       "nccl:all_to_all");
5564: }
5565: 
5566: c10::intrusive_ptr<Work> ProcessGroupNCCL::send(
5567:     std::vector<at::Tensor>& tensors,
5568:     int dstRank,
5569:     int /* unused */) {
5570:   TORCH_CHECK(tensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
5571:   auto tensor = tensors.back();
5572:   check_gpu_single_tensor(tensor, true);
5573: 
5574:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5575:       std::make_tuple(
5576:           static_cast<int64_t>(seqP2P_) + (coalescing_state_ & CoalP2P ? 0 : 1),
5577:           true), // the 1st p2p in coalesced range sets coalescing_state_ and
5578:                  // bumps seqP2P_
5579:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5580:       tensors, // inputTensors
```

- EN: Lines 5551-5580 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 5551-5580 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 5581-5610 / 第 5581-5610 行

```cpp
5581:       tensors, // outputTensors
5582:       dstRank, // dst rank
5583:       "send", // collective name
5584:       tensor.numel(), // inNelems
5585:       tensor.numel(), // outNelems
5586:       tensor.scalar_type(), // dType
5587:       std::vector<int64_t>(), // inSplitSizes
5588:       std::vector<int64_t>(), // outSplitSizes
5589:       globalRankStart_, // globalRankStart_
5590:       globalRankStride_, // globalRankStride_
5591:       this->getSize(), // worldSize
5592:       true); // is asynchronized op
5593: 
5594:   auto ret = pointToPoint(
5595:       tensor,
5596:       [&](at::Tensor& input,
5597:           ncclComm_t comm,
5598:           at::cuda::CUDAStream& stream,
5599:           int dst) {
5600:         auto ncclDataType = getNcclDataType(input.scalar_type());
5601:         return ncclSend(
5602:             input.data_ptr(),
5603:             input.numel(),
5604:             ncclDataType,
5605:             dst,
5606:             comm,
5607:             stream.stream());
5608:       },
5609:       dstRank,
5610:       OpType::SEND,
```

- EN: Lines 5581-5610 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5581-5610 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5611-5640 / 第 5611-5640 行

```cpp
5611:       c10::str("nccl:send ", rank_, "->", dstRank).c_str());
5612:   return ret;
5613: }
5614: 
5615: c10::intrusive_ptr<Work> ProcessGroupNCCL::recv(
5616:     std::vector<at::Tensor>& tensors,
5617:     int srcRank,
5618:     int /* unused */) {
5619:   TORCH_CHECK(tensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
5620:   auto tensor = tensors.back();
5621:   check_gpu_single_tensor(tensor, true);
5622: 
5623:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5624:       std::make_tuple(
5625:           static_cast<int64_t>(seqP2P_) + (coalescing_state_ & CoalP2P ? 0 : 1),
5626:           true), // the 1st p2p in coalesced range sets coalescing_state_ and
5627:                  // bumps seqP2P_
5628:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5629:       tensors, // inputTensors
5630:       tensors, // outputTensors
5631:       srcRank, // src rank
5632:       "recv", // collective name
5633:       tensor.numel(), // inNelems
5634:       tensor.numel(), // outNelems
5635:       tensor.scalar_type(), // dType
5636:       std::vector<int64_t>(), // inSplitSizes
5637:       std::vector<int64_t>(), // outSplitSizes
5638:       globalRankStart_, // globalRankStart_
5639:       globalRankStride_, // globalRankStride_
5640:       this->getSize(), // worldSize
```

- EN: Lines 5611-5640 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 5611-5640 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 5641-5670 / 第 5641-5670 行

```cpp
5641:       true); // is asynchronized op
5642: 
5643:   auto ret = pointToPoint(
5644:       tensor,
5645:       [&](at::Tensor& output,
5646:           ncclComm_t comm,
5647:           at::cuda::CUDAStream& stream,
5648:           int src) {
5649:         auto ncclDataType = getNcclDataType(output.scalar_type());
5650:         return ncclRecv(
5651:             output.data_ptr(),
5652:             output.numel(),
5653:             ncclDataType,
5654:             src,
5655:             comm,
5656:             stream.stream());
5657:       },
5658:       srcRank,
5659:       OpType::RECV,
5660:       c10::str("nccl:recv ", rank_, "<-", srcRank).c_str());
5661:   return ret;
5662: }
5663: 
5664: void ProcessGroupNCCL::groupStart() {
5665:   C10D_NCCL_CHECK(ncclGroupStart(), std::nullopt);
5666:   ++ncclActiveGroupCounter_;
5667: }
5668: 
5669: void ProcessGroupNCCL::groupEnd() {
5670:   C10D_NCCL_CHECK(ncclGroupEnd(), std::nullopt);
```

- EN: Lines 5641-5670 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5641-5670 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5671-5700 / 第 5671-5700 行

```cpp
5671:   --ncclActiveGroupCounter_;
5672: }
5673: 
5674: void ProcessGroupNCCL::groupEndNonblocking(
5675:     const std::shared_ptr<NCCLComm>& comm) {
5676: #ifndef NCCL_HAS_COMM_NONBLOCKING
5677:   C10D_NCCL_CHECK(ncclGroupEnd(), std::nullopt);
5678: #else
5679:   if (!useNonblocking()) {
5680:     C10D_NCCL_CHECK(ncclGroupEnd(), std::nullopt);
5681:   } else {
5682:     C10D_NCCL_CHECK_TIMEOUT_GROUPEND(ncclGroupEnd(), comm, std::nullopt);
5683:   }
5684: #endif // NCCL_HAS_COMM_NONBLOCKING
5685:   --ncclActiveGroupCounter_;
5686: }
5687: 
5688: c10::intrusive_ptr<Work> ProcessGroupNCCL::gather(
5689:     std::vector<std::vector<at::Tensor>>& outputTensors,
5690:     std::vector<at::Tensor>& inputTensors,
5691:     const GatherOptions& opts) {
5692:   static auto invalidArgument = [](const std::string& msg) {
5693:     C10_THROW_ERROR(ValueError, "ProcessGroupNCCL::gather: " + msg);
5694:   };
5695: 
5696:   assertRootRank(invalidArgument, opts.rootRank, size_);
5697: 
5698:   TORCH_CHECK(inputTensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
5699:   auto inputTensor = inputTensors.back();
5700:   check_gpu_single_tensor(inputTensor);
```

- EN: Lines 5671-5700 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5671-5700 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5701-5730 / 第 5701-5730 行

```cpp
5701: 
5702:   std::vector<at::Tensor> outputs;
5703: 
5704:   if (getRank() == opts.rootRank) {
5705:     if (outputTensors.size() != 1) {
5706:       std::stringstream ss;
5707:       ss << "requires a single-element output list containing a list with "
5708:          << getSize() << " tensors.";
5709:       invalidArgument(ss.str());
5710:     } else if (outputTensors[0].size() != static_cast<size_t>(getSize())) {
5711:       std::stringstream ss;
5712:       ss << "Incorrect output list size " << outputTensors[0].size()
5713:          << ". Output list size should be " << getSize()
5714:          << ", same as size of the process group.";
5715:       invalidArgument(ss.str());
5716:     }
5717: 
5718:     const auto& options = inputTensor.options();
5719:     const auto& sizes = inputTensor.sizes();
5720:     assertTypeAndSizesMatch(invalidArgument, outputTensors[0], options, sizes);
5721:     outputs = outputTensors[0];
5722:   } else {
5723:     // if not in the root rank, initialize outputs as empty list
5724:     if (!outputTensors.empty()) {
5725:       invalidArgument("requires empty output on non-root");
5726:     }
5727:     outputs = {};
5728:     // append a empty tensor to the list, we don't use it but the
5729:     // `collective` template function requires it to invoke its function
5730:     outputs.emplace_back();
```

- EN: Lines 5701-5730 introduces executable logic in routines such as `assertTypeAndSizesMatch`, `invalidArgument`.
- CN: 第 5701-5730 行在 `assertTypeAndSizesMatch`、`invalidArgument` 等例程中引入具体执行逻辑。

### Lines 5731-5760 / 第 5731-5760 行

```cpp
5731:   }
5732: 
5733:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5734:       std::make_tuple(
5735:           static_cast<int64_t>(seqCollective_) + 1,
5736:           false), // seq + 1 to match collective
5737:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5738:       inputTensors, // inputTensors
5739:       outputTensors, // outputTensors
5740:       opts.rootRank, // root rank
5741:       "gather", // collective name
5742:       inputTensor.numel(), // inNelems
5743:       inputTensor.numel() * this->getSize(), // outNelems
5744:       inputTensor.scalar_type(), // dType
5745:       std::vector<int64_t>(), // inSplitSizes
5746:       std::vector<int64_t>(), // outSplitSize
5747:       globalRankStart_, // globalRankStart_
5748:       globalRankStride_, // globalRankStride_
5749:       this->getSize(), // worldSize
5750:       opts.asyncOp); // is asynchronized op
5751: 
5752:   // avoidRecordStreams_ note: collective() will stash inputTensors and
5753:   // outputs, which == outputTensors[0] on the root rank where it matters.
5754: 
5755:   auto inputs = std::vector<at::Tensor>{inputTensor};
5756:   return collective(
5757:       inputs,
5758:       outputs, // just to fit the collective interface
5759:       [&](at::Tensor& /* unused */,
5760:           at::Tensor& /* unused */,
```

- EN: Lines 5731-5760 returns computed state or forwards results to the surrounding caller.
- CN: 第 5731-5760 行返回计算结果，或将结果继续传递给外围调用方。

### Lines 5761-5790 / 第 5761-5790 行

```cpp
5761:           ncclComm_t comm,
5762:           at::cuda::CUDAStream& stream) {
5763:         const auto root = opts.rootRank;
5764:         torch::cuda::nccl::gather(
5765:             inputTensor, outputs, comm, stream, static_cast<int32_t>(root));
5766:         return ncclSuccess;
5767:       },
5768:       [](at::cuda::CUDAStream&,
5769:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5770:       [](at::cuda::CUDAStream&,
5771:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5772:       OpType::GATHER,
5773:       opts.asyncOp,
5774:       "nccl:gather");
5775: }
5776: 
5777: c10::intrusive_ptr<Work> ProcessGroupNCCL::scatter(
5778:     std::vector<at::Tensor>& outputTensors,
5779:     std::vector<std::vector<at::Tensor>>& inputTensors,
5780:     const ScatterOptions& opts) {
5781:   static auto invalidArgument = [](const std::string& msg) {
5782:     C10_THROW_ERROR(ValueError, "ProcessGroupNCCL::scatter: " + msg);
5783:   };
5784: 
5785:   assertRootRank(invalidArgument, opts.rootRank, size_);
5786: 
5787:   TORCH_CHECK(outputTensors.size() == 1, MULTI_DEVICE_ERROR_MSG);
5788:   auto outputTensor = outputTensors.back();
5789: 
5790:   std::vector<at::Tensor> inputs;
```

- EN: Lines 5761-5790 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5761-5790 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5791-5820 / 第 5791-5820 行

```cpp
5791: 
5792:   if (getRank() == opts.rootRank) {
5793:     if (inputTensors.size() != 1) {
5794:       std::stringstream ss;
5795:       ss << "requires a single-element input list containing a list with "
5796:          << getSize() << " tensors.";
5797:       invalidArgument(ss.str());
5798:     } else if (inputTensors[0].size() != static_cast<size_t>(getSize())) {
5799:       std::stringstream ss;
5800:       ss << "Incorrect input list size " << inputTensors[0].size()
5801:          << ". Input list size should be " << getSize()
5802:          << ", same as size of the process group.";
5803:       invalidArgument(ss.str());
5804:     }
5805: 
5806:     const auto& options = outputTensor.options();
5807:     const auto& sizes = outputTensor.sizes();
5808:     assertTypeAndSizesMatch(invalidArgument, inputTensors[0], options, sizes);
5809:     inputs = inputTensors[0];
5810:   } else {
5811:     // if not in the root rank, initialize inputTensors as empty place holder
5812:     // with an empty list
5813:     if (!inputTensors.empty()) {
5814:       invalidArgument("requires empty input on non-root");
5815:     }
5816:     inputs = {};
5817:     // append a empty tensor to the list, we don't use it but the
5818:     // `collective` template function requires it to invoke its function
5819:     inputs.emplace_back();
5820:   }
```

- EN: Lines 5791-5820 introduces executable logic in routines such as `assertTypeAndSizesMatch`, `invalidArgument`.
- CN: 第 5791-5820 行在 `assertTypeAndSizesMatch`、`invalidArgument` 等例程中引入具体执行逻辑。

### Lines 5821-5850 / 第 5821-5850 行

```cpp
5821: 
5822:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5823:       std::make_tuple(
5824:           static_cast<int64_t>(seqCollective_) + 1,
5825:           false), // seq + 1 to match collective
5826:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5827:       inputTensors, // inputTensors
5828:       outputTensors, // outputTensors
5829:       opts.rootRank, // root rank
5830:       "scatter", // collective name
5831:       outputTensor.numel() * this->getSize(), // inNelems
5832:       outputTensor.numel(), // outNelems
5833:       outputTensor.scalar_type(), // dType
5834:       std::vector<int64_t>(), // inSplitSizes
5835:       std::vector<int64_t>(), // outSplitSize
5836:       globalRankStart_, // globalRankStart_
5837:       globalRankStride_, // globalRankStride_
5838:       this->getSize(), // worldSize
5839:       opts.asyncOp); // is asynchronized op
5840: 
5841:   // avoidRecordStreams_ note: collective() will stash outputTensors and
5842:   // inputs, which == inputTensors[0] on the root rank where it matters.
5843:   const auto root = opts.rootRank;
5844:   bool nanCheck = (rank_ == root);
5845: 
5846:   auto outputs = std::vector<at::Tensor>{outputTensor};
5847:   return collective(
5848:       outputs,
5849:       inputs, // just to fit the collective interface
5850:       [&](at::Tensor& /* unused */,
```

- EN: Lines 5821-5850 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5821-5850 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5851-5880 / 第 5851-5880 行

```cpp
5851:           at::Tensor& /* unused */,
5852:           ncclComm_t comm,
5853:           at::cuda::CUDAStream& stream) {
5854:         torch::cuda::nccl::scatter(
5855:             inputs, outputTensor, comm, stream, static_cast<int32_t>(root));
5856:         return ncclSuccess;
5857:       },
5858:       [](at::cuda::CUDAStream&,
5859:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5860:       [](at::cuda::CUDAStream&,
5861:          c10::intrusive_ptr<ProcessGroupNCCL::WorkNCCL>& work) {},
5862:       OpType::SCATTER,
5863:       opts.asyncOp,
5864:       "nccl:scatter",
5865:       nanCheck);
5866: }
5867: 
5868: c10::intrusive_ptr<Work> ProcessGroupNCCL::recvAnysource(
5869:     std::vector<at::Tensor>& /* unused */,
5870:     int /* unused */) {
5871:   C10_THROW_ERROR(
5872:       NotImplementedError, "ProcessGroupNCCL does not support recvAnysource");
5873: }
5874: 
5875: c10::intrusive_ptr<Work> ProcessGroupNCCL::_allgather_base(
5876:     at::Tensor& output_tensor,
5877:     at::Tensor& input_tensor,
5878:     const AllgatherOptions& opts) {
5879:   check_gpu_single_tensor(input_tensor);
5880:   check_gpu_single_tensor(output_tensor);
```

- EN: Lines 5851-5880 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5851-5880 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5881-5910 / 第 5881-5910 行

```cpp
5881: 
5882:   if (input_tensor.dtype() != output_tensor.dtype()) {
5883:     C10_THROW_ERROR(
5884:         TypeError, "output tensor must have the same type as input tensor");
5885:   }
5886: 
5887:   if (input_tensor.numel() * size_ != output_tensor.numel()) {
5888:     C10_THROW_ERROR(
5889:         ValueError,
5890:         "output tensor size must be equal to world_size times input tensor size");
5891:   }
5892: 
5893:   RECORD_PARAM_COMMS_DATA_WITH_ASYNC_OP(
5894:       std::make_tuple(
5895:           static_cast<int64_t>(seqCollective_) + 1,
5896:           false), // seq + 1 to match collective
5897:       std::make_tuple(pg_uid_, pg_desc_), // PG name tuple
5898:       input_tensor, // inputTensors
5899:       output_tensor, // outputTensors
5900:       rank_, // rank
5901:       "_allgather_base", // collective name
5902:       input_tensor.numel(), // inNelems
5903:       output_tensor.numel(), // outNelems
5904:       output_tensor.scalar_type(), // dType
5905:       std::vector<int64_t>(), // inSplitSizes
5906:       std::vector<int64_t>(), // outSplitSize
5907:       globalRankStart_, // globalRankStart_
5908:       globalRankStride_, // globalRankStride_
5909:       this->getSize(), // worldSize
5910:       opts.asyncOp); // is asynchronized op
```

- EN: Lines 5881-5910 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5881-5910 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5911-5940 / 第 5911-5940 行

```cpp
5911: 
5912:   // avoidRecordStreams_ note: collective() will stash inputs and outputs.
5913:   // Note 2: for asyncOp = false, we don't want to record streams because we
5914:   // know that the NCCL stream will join back to the "current" stream right
5915:   // after this op. So we might just as well keep the stream ownership of the
5916:   // input/output tensors unchanged. The benefit would be that the
5917:   // allocation/free of the tensors would look deterministic to the "current"
5918:   // stream so that the caching allocator can reuse memory pool for this stream
5919:   // in a clever way. This setting is added for libraries like FSDP which uses
5920:   // `all_gather_into_tensor`.
5921: 
5922:   return collective(
5923:       input_tensor,
5924:       output_tensor,
5925:       [&](at::Tensor& input,
5926:           at::Tensor& output,
5927:           ncclComm_t comm,
5928:           at::cuda::CUDAStream& stream) {
5929:         return ncclAllGather(
5930:             input.data_ptr(),
5931:             output.data_ptr(),
5932:             input.numel(),
5933:             getNcclDataType(input.scalar_type()),
5934:             comm,
5935:             stream.stream());
5936:       },
5937:       OpType::_ALLGATHER_BASE,
5938:       opts.asyncOp,
5939:       "nccl:_all_gather_base");
5940: }
```

- EN: Lines 5911-5940 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 5911-5940 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 5941-5970 / 第 5941-5970 行

```cpp
5941: 
5942: // Create a memory allocator for NCCL. This allocator is used to allocate memory
5943: // that supports NVLink Sharp functionality. This allocator is later pybinded to
5944: // python, so that users can use it to create MemPool. For example:
5945: // >>> pool = torch.cuda.MemPool(backend.mem_allocator)
5946: 
5947: // Allocate function
5948: static void* _ncclMemAlloc(size_t size, int device, void* stream) {
5949: #ifndef NCCL_HAS_MEM_ALLOC
5950:   TORCH_CHECK(
5951:       false, "NCCL mem allocator is not supported in this NCCL version");
5952: #else
5953:   LOG(INFO) << "NCCL mem allocator: allocating " << size << " bytes";
5954:   at::cuda::OptionalCUDAGuard gpuGuard(device);
5955:   void* ptr = nullptr;
5956:   TORCH_CHECK(ncclMemAlloc(&ptr, size) == ncclSuccess, "ncclMemAlloc failed");
5957:   return ptr;
5958: #endif // NCCL_HAS_MEM_ALLOC
5959: }
5960: 
5961: // Free function
5962: static void _ncclMemFree(void* ptr, size_t size, int device, void* stream) {
5963: #ifndef NCCL_HAS_MEM_ALLOC
5964:   TORCH_CHECK(
5965:       false, "NCCL mem allocator is not supported in this NCCL version");
5966: #else
5967:   LOG(INFO) << "NCCL mem allocator: freeing " << size << " bytes";
5968:   at::cuda::OptionalCUDAGuard gpuGuard(device);
5969:   TORCH_CHECK(ncclMemFree(ptr) == ncclSuccess, "ncclMemFree failed");
5970: #endif // NCCL_HAS_MEM_ALLOC
```

- EN: Lines 5941-5970 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `_ncclMemAlloc`, `TORCH_CHECK`, `_ncclMemFree`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5941-5970 行使用条件编译来适配特性开关、平台或可选后端；在 `_ncclMemAlloc`、`TORCH_CHECK`、`_ncclMemFree` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 5971-6000 / 第 5971-6000 行

```cpp
5971: }
5972: 
5973: // Create a `CUDAPluggableAllocator` that uses the above functions.
5974: std::shared_ptr<c10::Allocator> ProcessGroupNCCL::getMemAllocator() {
5975:   C10_LOG_API_USAGE_ONCE("ProcessGroupNCCL.getMemAllocator");
5976:   c10::DeviceIndex deviceIdx = guessDeviceId();
5977:   if (!supportsTensorAlloc(deviceIdx)) {
5978:     TORCH_CHECK(
5979:         false, "NCCL mem allocator is not supported in this NCCL version");
5980:   }
5981:   static std::shared_ptr<c10::cuda::CUDACachingAllocator::CUDAAllocator>
5982:       ncclMemAllocator =
5983:           torch::cuda::CUDAPluggableAllocator::createCustomAllocator(
5984:               _ncclMemAlloc, _ncclMemFree);
5985:   return ncclMemAllocator;
5986: }
5987: 
5988: bool ProcessGroupNCCL::supportsTensorAlloc(c10::DeviceIndex deviceIdx) {
5989:   // Check if NCCL has `ncclMemAlloc` and `ncclMemFree` functions
5990:   int version = 0;
5991:   // Rely on link-time versioning
5992:   ncclGetVersion(&version);
5993:   if (version < NCCL_VERSION(2, 19, 0)) {
5994:     return false;
5995:   }
5996: 
5997:   // We do an extra check to see if CUDA driver supports multicast.  If not, we
5998:   // will return false. Although `ncclMemAlloc` will fall back to regular
5999:   // `cudaMalloc` and hence not error out, we may still want to avoid creating a
6000:   // separate memory pool for NCCL.
```

- EN: Lines 5971-6000 introduces executable logic in routines such as `TORCH_CHECK`, `ncclGetVersion`; performs validation and error handling to keep distributed state consistent.
- CN: 第 5971-6000 行在 `TORCH_CHECK`、`ncclGetVersion` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 6001-6030 / 第 6001-6030 行

```cpp
6001:   return c10d::cuda::deviceSupportsMulticast(deviceIdx);
6002: }
6003: 
6004: at::Tensor ProcessGroupNCCL::allocateTensor(
6005:     long size,
6006:     at::TensorOptions options) {
6007:   // Some checks
6008:   TORCH_CHECK_VALUE(options.has_device(), "Tensor options must include device");
6009:   auto device = options.device();
6010:   TORCH_CHECK_VALUE(
6011:       device.is_cuda(),
6012:       "NCCL tensor allocator expects cuda type but got " + c10::str(device))
6013: 
6014:   at::cuda::OptionalCUDAGuard gpuGuard(device);
6015: 
6016:   // Create memory pool
6017:   if (!memPool_) {
6018:     // Needs a CUDAAllocator
6019:     auto allocator = std::static_pointer_cast<
6020:         c10::cuda::CUDACachingAllocator::CUDAAllocator>(getMemAllocator());
6021:     // Pool is created
6022:     memPool_ = std::make_unique<at::cuda::MemPool>(std::move(allocator));
6023:     // Register so that we call ncclCommRegister on all new allocations
6024:     registerMemPool(memPool_.get(), /*symmetric*/ false);
6025:     LOG(INFO) << logPrefix() << "Created memory pool";
6026:   }
6027: 
6028:   // Allocate tensor under this MemPool's context
6029:   auto tid = std::this_thread::get_id();
6030:   c10::cuda::CUDACachingAllocator::beginAllocateToPool(
```

- EN: Lines 6001-6030 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 6001-6030 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 6031-6060 / 第 6031-6060 行

```cpp
6031:       memPool_->device(), memPool_->id(), [=](cudaStream_t) {
6032:         auto current_tid = std::this_thread::get_id();
6033:         return current_tid == tid;
6034:       });
6035:   at::Tensor tensor = at::empty({size}, options);
6036:   c10::cuda::CUDACachingAllocator::endAllocateToPool(
6037:       memPool_->device(), memPool_->id());
6038:   c10::cuda::CUDACachingAllocator::releasePool(
6039:       memPool_->device(), memPool_->id());
6040:   LOG(INFO) << logPrefix() << "Allocated tensor of size " << size
6041:             << " from memory pool";
6042:   return tensor;
6043: }
6044: 
6045: #ifdef NCCL_HAS_COMM_SHRINK
6046: c10::intrusive_ptr<Backend> ProcessGroupNCCL::shrink(
6047:     const std::vector<int64_t>& ranks_to_exclude,
6048:     int shrink_flags,
6049:     const c10::intrusive_ptr<Backend::Options>& opts_override) {
6050:   // Runtime version check with better error message
6051:   auto runtime_version = torch::cuda::nccl::version();
6052:   TORCH_CHECK(
6053:       runtime_version >= NCCL_VERSION(2, 27, 0),
6054:       "ProcessGroupNCCL::shrink requires NCCL version 2.27.0 or later. "
6055:       "Found version: ",
6056:       runtime_version);
6057: 
6058:   // Early validation with detailed error messages
6059:   TORCH_CHECK_VALUE(
6060:       !ranks_to_exclude.empty(), "ranks_to_exclude cannot be empty");
```

- EN: Lines 6031-6060 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 6031-6060 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 6061-6090 / 第 6061-6090 行

```cpp
6061:   TORCH_CHECK_VALUE(
6062:       static_cast<int>(ranks_to_exclude.size()) < size_,
6063:       "Cannot exclude all ranks (",
6064:       ranks_to_exclude.size(),
6065:       " >= ",
6066:       size_,
6067:       ")");
6068: 
6069:   // Validate ranks and convert to int efficiently
6070:   std::vector<int> int_ranks_to_exclude;
6071:   int_ranks_to_exclude.reserve(ranks_to_exclude.size());
6072:   for (int64_t rank : ranks_to_exclude) {
6073:     TORCH_CHECK_VALUE(
6074:         rank >= 0 && rank < size_,
6075:         "Invalid rank ",
6076:         rank,
6077:         " for group size ",
6078:         size_);
6079:     int_ranks_to_exclude.push_back(static_cast<int>(rank));
6080:   }
6081: 
6082:   // Get primary communicator with better error context
6083:   auto primary_device_index = guessDeviceId();
6084:   auto primary_device = at::Device(at::kCUDA, primary_device_index);
6085:   const auto primary_key = getKeyFromDevice(primary_device);
6086: 
6087:   std::shared_ptr<NCCLComm> primary_comm = getNCCLComm(primary_key);
6088:   TORCH_CHECK(
6089:       primary_comm,
6090:       "Primary NCCL communicator for device ",
```

- EN: Lines 6061-6090 introduces executable logic in routines such as `TORCH_CHECK_VALUE`; performs validation and error handling to keep distributed state consistent.
- CN: 第 6061-6090 行在 `TORCH_CHECK_VALUE` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 6091-6120 / 第 6091-6120 行

```cpp
6091:       primary_device,
6092:       " (key: ",
6093:       primary_key,
6094:       ") is not initialized");
6095: 
6096:   // Cache device index before shrink operation
6097:   at::DeviceIndex parent_device_index = primary_comm->getDeviceIndex();
6098: 
6099:   ncclConfig_t* config = nullptr;
6100:   // Default to inheriting from parent options
6101:   bool high_priority_stream = options_->is_high_priority_stream;
6102:   if (opts_override) {
6103:     auto nccl_opts =
6104:         c10::static_intrusive_pointer_cast<ProcessGroupNCCL::Options>(
6105:             opts_override);
6106:     config = &nccl_opts->config;
6107:     // If user provided override options, honor is_high_priority_stream as well
6108:     high_priority_stream = nccl_opts->is_high_priority_stream;
6109:   }
6110: 
6111:   std::shared_ptr<NCCLComm> shrunk_comm = NCCLComm::shrink(
6112:       primary_comm.get(),
6113:       int_ranks_to_exclude,
6114:       (config != nullptr ? config : &options_->config),
6115:       shrink_flags);
6116: 
6117:   // Calculate new size and get NCCL-assigned rank
6118:   int new_size = size_ - static_cast<int>(ranks_to_exclude.size());
6119:   int new_rank = shrunk_comm->rank_;
6120: 
```

- EN: Lines 6091-6120 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 6091-6120 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 6121-6150 / 第 6121-6150 行

```cpp
6121:   // Create new ProcessGroupNCCL with optimized options cloning
6122:   auto new_store = store_->clone();
6123:   auto new_opts = ProcessGroupNCCL::Options::create(high_priority_stream);
6124:   new_opts->timeout = options_->timeout;
6125:   if (config != nullptr) {
6126:     new_opts->config = *config;
6127:   } else {
6128:     new_opts->config = options_->config;
6129:   }
6130: 
6131:   auto new_pg = c10::make_intrusive<ProcessGroupNCCL>(
6132:       new_store, new_rank, new_size, new_opts);
6133: 
6134:   // Set up the new process group with optimized device setup
6135:   new_pg->initializeDeviceStateForComm(
6136:       at::Device(at::kCUDA, parent_device_index), shrunk_comm);
6137: 
6138:   return c10::static_intrusive_pointer_cast<Backend>(new_pg);
6139: }
6140: 
6141: #else // !NCCL_HAS_COMM_SHRINK
6142: // Backend interface override: raise consistent error when shrink is
6143: // unsupported.
6144: c10::intrusive_ptr<Backend> ProcessGroupNCCL::shrink(
6145:     const std::vector<int64_t>& /*ranks_to_exclude*/,
6146:     int /*shrink_flags*/,
6147:     const c10::intrusive_ptr<Backend::Options>& /*opts_override*/) {
6148:   TORCH_CHECK(
6149:       false,
6150:       "ProcessGroupNCCL::shrink requires NCCL version 2.27.0 or later, "
```

- EN: Lines 6121-6150 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 6121-6150 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 6151-6180 / 第 6151-6180 行

```cpp
6151:       "but PyTorch was built with an older version or without NCCL shrink support.");
6152: }
6153: 
6154: #endif // NCCL_HAS_COMM_SHRINK
6155: 
6156: void ProcessGroupNCCL::initializeDeviceStateForComm(
6157:     const at::Device& device,
6158:     std::shared_ptr<NCCLComm> comm) {
6159:   const auto key = getKeyFromDevice(device);
6160:   std::unique_lock<std::mutex> lock(mutex_);
6161:   at::cuda::OptionalCUDAGuard gpuGuard(device);
6162: 
6163:   bool force_high = getCvarBool(TORCH_NCCL_HIGH_PRIORITY, false);
6164:   auto stream = at::cuda::getStreamFromPool(
6165:       options_->is_high_priority_stream || force_high);
6166: 
6167:   devNCCLCommMap_[key] = comm;
6168:   ncclStreams_.emplace(key, stream);
6169:   ncclEvents_.emplace(key, at::cuda::CUDAEvent(cudaEventDisableTiming));
6170:   usedDeviceIdxs_.insert(device.index());
6171: 
6172:   if (shouldAllCommunicatorsRegisterAllTensors()) {
6173:     std::lock_guard<std::mutex> map_lock(ncclCommMemPoolMapMutex);
6174:     ncclCommMemPoolMap.emplace(std::move(comm), MemPoolSet{});
6175:   }
6176: }
6177: 
6178: } // namespace c10d
6179: 
6180: #endif // USE_C10D_NCCL
```

- EN: Lines 6151-6180 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 6151-6180 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `RocmWatchdogEventQueryContextGuard`
- CN: 核心符号：`RocmWatchdogEventQueryContextGuard`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/FlightRecorder.hpp`, `torch/csrc/distributed/c10d/NCCLUtils.hpp`, `torch/csrc/distributed/c10d/NanCheck.hpp`, `torch/csrc/distributed/c10d/ParamCommsUtils.hpp`, `torch/csrc/distributed/c10d/PrefixStore.hpp`, `torch/csrc/distributed/c10d/ProcessGroupNCCL.hpp`, `torch/csrc/distributed/c10d/TraceUtils.h`, `torch/csrc/distributed/c10d/Utils.hpp`, `torch/csrc/distributed/c10d/cuda/utils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/cuda/CUDAContext.h`, `ATen/cuda/CUDAGraph.h`, `c10/core/DeviceType.h`, `c10/cuda/CUDAAllocatorConfig.h`, `c10/cuda/CUDAException.h`, `c10/cuda/CUDAGraphsC10Utils.h`, `c10/cuda/CUDAGuard.h`, `c10/util/Exception.h`, `c10/util/Logging.h`, `c10/util/WaitCounter.h`, `c10/util/hash.h`, `c10/util/irange.h`, ... (+4 more)
- External or system headers / 外部或系统头文件: `nlohmann/json.hpp`, `exception`, `map`, `memory`, `mutex`, `sstream`, `stdexcept`, `tuple`, `utility`, `optional`
- Local symbols / 本地符号: `RocmWatchdogEventQueryContextGuard`