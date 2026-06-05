# NCCLUtils.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/NCCLUtils.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides utility helpers in the c10d distributed process-group subsystem. Key types include `NCCLComm`, `ncclRedOpRAII`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供工具辅助逻辑。 关键类型包括 `NCCLComm`、`ncclRedOpRAII`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: 
3: #ifdef USE_C10D_NCCL
4: 
5: #include <sched.h>
6: #include <cstdio>
7: #include <cstdlib>
8: 
9: #include <memory>
10: #include <mutex>
11: 
12: #include <ATen/ATen.h>
13: #include <ATen/cuda/CUDAEvent.h>
14: #include <c10/util/Exception.h>
15: #include <nccl.h>
16: #include <torch/csrc/cuda/nccl.h>
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 17-32 / 第 17-32 行

```cpp
17: #include <optional>
18: 
19: constexpr int64_t kCommInitBusyWaitMillis = 2;
20: 
21: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 14, 0)
22: #define NCCL_HAS_COMM_NONBLOCKING
23: #endif
24: 
25: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 18, 0)
26: #define NCCL_HAS_COMM_SPLIT
27: #endif
28: 
29: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 23, 0)
30: #define NCCL_HAS_INIT_RANK_SCALABLE
31: #endif
32: 
```

- EN: Lines 17-32 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 17-32 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 33-48 / 第 33-48 行

```cpp
33: // ncclGetLastError() is enabled only for NCCL versions 2.13+
34: // ncclRemoteError only exists in NCCL versions 2.13+
35: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 13, 0)
36: #define ENABLE_NCCL_GET_LAST_ERROR
37: #define NCCL_REMOTE_ERROR
38: #endif
39: 
40: static_assert(
41:     NCCL_VERSION_CODE >= NCCL_VERSION(2, 7, 0),
42:     "NCCL version must be 2.7 or later");
43: // The following macros represent features supported prior to NCCL 2.7,
44: // therefore we can define them unconditionally, given the static_assert above.
45: // TODO: remove these macros from code.
46: #define ENABLE_NCCL_ERROR_CHECKING
47: #define ENABLE_NCCL_P2P_SUPPORT
48: // End of macros for NCCL 2.7 and below.
```

- EN: Lines 33-48 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 33-48 行使用条件编译来适配特性开关、平台或可选后端。

### Lines 49-64 / 第 49-64 行

```cpp
49: 
50: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 11, 0)
51: #define ENABLE_NCCL_PREMUL_SUM_SUPPORT
52: #endif
53: 
54: // Note: the first version that supports ncclConfig_t is 2.14. Here we
55: // fast-forward the version requirement to 2.17 where ncclConfig_t has CTA and
56: // CGA fields because they have already been pybinded out.
57: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 17, 0)
58: #define NCCL_HAS_CONFIG
59: #endif
60: 
61: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 19, 0)
62: #define NCCL_HAS_COMM_REGISTER
63: #endif
64: 
```

- EN: Lines 49-64 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 49-64 行使用条件编译来适配特性开关、平台或可选后端。

### Lines 65-80 / 第 65-80 行

```cpp
65: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 27, 0)
66: #define NCCL_HAS_COMM_WINDOW_REGISTER
67: #endif
68: 
69: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 19, 0)
70: #define NCCL_HAS_MEM_ALLOC
71: #endif
72: 
73: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 26, 0)
74: #define NCCL_HAS_QOS
75: #endif
76: 
77: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 24, 0)
78: #define NCCL_SUPPORTS_FP8
79: #endif
80: 
```

- EN: Lines 65-80 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 65-80 行使用条件编译来适配特性开关、平台或可选后端。

### Lines 81-96 / 第 81-96 行

```cpp
81: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 27, 0)
82: #define NCCL_HAS_COLLNET
83: #endif
84: 
85: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 27, 0)
86: #define NCCL_HAS_CTA_POLICY
87: #endif
88: 
89: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 27, 0)
90: #define NCCL_HAS_NVLS_CTAS
91: #endif
92: 
93: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 27, 0)
94: #define NCCL_HAS_COMM_SHRINK
95: #endif
96: 
```

- EN: Lines 81-96 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 81-96 行使用条件编译来适配特性开关、平台或可选后端。

### Lines 97-112 / 第 97-112 行

```cpp
97: #if NCCL_VERSION_CODE >= NCCL_VERSION(2, 29, 7)
98: #define NCCL_HAS_COMM_OFFLOAD
99: #endif
100: 
101: // Macro to throw on a non-successful NCCL return value.
102: #define C10D_NCCL_CHECK(cmd, failureReason)                                   \
103:   do {                                                                        \
104:     ncclResult_t result = cmd;                                                \
105:     if (result != ncclSuccess) {                                              \
106:       std::string err = "NCCL error in: " + std::string(__FILE__) + ":" +     \
107:           std::to_string(__LINE__) + ", " + ncclGetErrorWithVersion(result) + \
108:           "\n" + getNcclErrorDetailStr(result, failureReason);                \
109:       TORCH_CHECK_WITH(DistBackendError, false, err);                         \
110:     }                                                                         \
111:   } while (0)
112: 
```

- EN: Lines 97-112 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 97-112 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 113-128 / 第 113-128 行

```cpp
113: // Macro to throw on a non-successful NCCL return value for NONBLOCKING calls.
114: #define C10D_NCCL_CHECK_NONBLOCKING(cmd, failureReason)                       \
115:   do {                                                                        \
116:     ncclResult_t result = cmd;                                                \
117:     if (result != ncclSuccess && result != ncclInProgress) {                  \
118:       std::string err = "NCCL error in: " + std::string(__FILE__) + ":" +     \
119:           std::to_string(__LINE__) + ", " + ncclGetErrorWithVersion(result) + \
120:           "\n" + getNcclErrorDetailStr(result, failureReason);                \
121:       TORCH_CHECK_WITH(DistBackendError, false, err);                         \
122:     }                                                                         \
123:   } while (0)
124: 
125: // Error out if (current time - startTime) is greater than timeout (sec).
126: #define C10D_CHECK_TIMEOUT(startTime, timeout)                              \
127:   do {                                                                      \
128:     auto currentTime = std::chrono::steady_clock::now();                    \
```

- EN: Lines 113-128 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 113-128 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 129-144 / 第 129-144 行

```cpp
129:     auto timeElapsed = std::chrono::duration_cast<std::chrono::seconds>(    \
130:                            currentTime - startTime)                         \
131:                            .count();                                        \
132:     if (timeElapsed > timeout) {                                            \
133:       std::string err = "NCCL timeout in: " + std::string(__FILE__) + ":" + \
134:           std::to_string(__LINE__);                                         \
135:       TORCH_CHECK_WITH(DistBackendError, false, err);                       \
136:     }                                                                       \
137:   } while (0)
138: 
139: // Macro to throw on a non-successful NCCL return value, non-blocking.
140: // Thread-safe: uses NCCLComm wrapper's getAsyncError() which acquires mutex
141: // before calling ncclCommGetAsyncError to prevent race conditions between
142: // watchdog and main threads.
143: #define C10D_NCCL_CHECK_TIMEOUT_BASE(                                         \
144:     cmd, commWrapper, failureReason, yield_fn)                                \
```

- EN: Lines 129-144 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 129-144 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 145-160 / 第 145-160 行

```cpp
145:   do {                                                                        \
146:     ncclResult_t result = cmd;                                                \
147:     auto startTimepoint = std::chrono::steady_clock::now();                   \
148:     auto timeout = nccl_nonblocking_timeout();                                \
149:     while (result == ncclInProgress) {                                        \
150:       C10D_CHECK_TIMEOUT(startTimepoint, timeout);                            \
151:       yield_fn;                                                               \
152:       commWrapper->getAsyncError(&result);                                    \
153:     }                                                                         \
154:     if (result != ncclSuccess) {                                              \
155:       std::string err = "NCCL error in: " + std::string(__FILE__) + ":" +     \
156:           std::to_string(__LINE__) + ", " + ncclGetErrorWithVersion(result) + \
157:           "\n" + getNcclErrorDetailStr(result, failureReason);                \
158:       TORCH_CHECK_WITH(DistBackendError, false, err);                         \
159:     }                                                                         \
160:   } while (0)
```

- EN: Lines 145-160 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-160 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-176 / 第 161-176 行

```cpp
161: 
162: // Sleep for kCommInitBusyWaitMillis milliseconds.
163: #define C10D_SCHED_SLEEP()     \
164:   std::this_thread::sleep_for( \
165:       std::chrono::milliseconds(kCommInitBusyWaitMillis))
166: 
167: // Macro to throw exception on a non-successful NCCL return value or timeout.
168: // This macro uses sched_yield() to yield the CPU.
169: // Thus suitable for NCCL calls that would quickly turn ncclSuccess, e.g.
170: // collectives.
171: #define C10D_NCCL_CHECK_TIMEOUT(cmd, commWrapper, failureReason) \
172:   C10D_NCCL_CHECK_TIMEOUT_BASE(cmd, commWrapper, failureReason, sched_yield())
173: 
174: // Macro to throw exception on a non-successful NCCL return value or timeout.
175: // This macro uses sleep to yield the CPU.
176: // Thus suitable for NCCL calls that would take longer to turn ncclSuccess, e.g.
```

- EN: Lines 161-176 performs validation and error handling to keep distributed state consistent; returns computed state or forwards results to the surrounding caller.
- CN: 第 161-176 行执行校验与错误处理，以保持分布式状态一致；返回计算结果，或将结果继续传递给外围调用方。

### Lines 177-192 / 第 177-192 行

```cpp
177: // ncclCommInitRankConfig, ncclCommFinalize, etc.
178: #define C10D_NCCL_CHECK_TIMEOUT_SLEEP(cmd, commWrapper, failureReason) \
179:   C10D_NCCL_CHECK_TIMEOUT_BASE(                                        \
180:       cmd, commWrapper, failureReason, C10D_SCHED_SLEEP())
181: 
182: #define C10D_NCCL_CHECK_TIMEOUT_GROUPEND(cmd, comm, failureReason)           \
183:   do {                                                                       \
184:     ncclResult_t state = cmd;                                                \
185:     auto startTimepoint = std::chrono::steady_clock::now();                  \
186:     auto timeout = nccl_nonblocking_timeout();                               \
187:     if (state == ncclInProgress) {                                           \
188:       do {                                                                   \
189:         C10D_CHECK_TIMEOUT(startTimepoint, timeout);                         \
190:         sched_yield();                                                       \
191:         comm->getAsyncError(&state);                                         \
192:       } while (state == ncclInProgress);                                     \
```

- EN: Lines 177-192 introduces executable logic in routines such as `sched_yield`.
- CN: 第 177-192 行在 `sched_yield` 等例程中引入具体执行逻辑。

### Lines 193-208 / 第 193-208 行

```cpp
193:     }                                                                        \
194:     if (state != ncclSuccess) {                                              \
195:       std::string err = "NCCL error in: " + std::string(__FILE__) + ":" +    \
196:           std::to_string(__LINE__) + ", " + ncclGetErrorWithVersion(state) + \
197:           "\n" + getNcclErrorDetailStr(state, failureReason);                \
198:       TORCH_CHECK_WITH(DistBackendError, false, err);                        \
199:     }                                                                        \
200:   } while (0)
201: 
202: // Macro to print and abort on a non-successful NCCL return value.
203: #define C10D_NCCL_ASSERT(cmd)                            \
204:   do {                                                   \
205:     ncclResult_t result = cmd;                           \
206:     if (result != ncclSuccess) {                         \
207:       std::string err = ncclGetErrorWithVersion(result); \
208:       fprintf(                                           \
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 209-224 / 第 209-224 行

```cpp
209:           stderr,                                        \
210:           "NCCL error in: %s:%d, %s\n",                  \
211:           __FILE__,                                      \
212:           __LINE__,                                      \
213:           err.c_str());                                  \
214:       abort();                                           \
215:     }                                                    \
216:   } while (0)
217: 
218: namespace c10d {
219: 
220: // NCCL type typing
221: static std::map<at::ScalarType, ncclDataType_t> ncclDataType = {
222:     {at::kChar, ncclInt8},
223:     {at::kByte, ncclUint8},
224:     {at::kFloat, ncclFloat},
```

- EN: Lines 209-224 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `abort`.
- CN: 第 209-224 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `abort` 等例程中引入具体执行逻辑。

### Lines 225-240 / 第 225-240 行

```cpp
225:     {at::kDouble, ncclDouble},
226:     {at::kInt, ncclInt32},
227:     {at::kLong, ncclInt64},
228:     {at::kHalf, ncclHalf},
229:     {at::kBool, ncclUint8},
230: #ifdef NCCL_SUPPORTS_FP8
231:     {at::kFloat8_e5m2, ncclFloat8e5m2},
232:     {at::kFloat8_e4m3fn, ncclFloat8e4m3},
233: #else
234:     {at::kFloat8_e5m2, ncclUint8},
235:     {at::kFloat8_e4m3fn, ncclUint8},
236: #endif
237:     // NVIDIA GPUs does not support the UZ version standing for "no negative
238:     // zero".  See https://onnx.ai/onnx/technical/float8.html
239:     {at::kFloat8_e4m3fnuz, ncclUint8},
240:     {at::kFloat8_e5m2fnuz, ncclUint8},
```

- EN: Lines 225-240 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 225-240 行使用条件编译来适配特性开关、平台或可选后端。

### Lines 241-256 / 第 241-256 行

```cpp
241: #if HAS_NCCL_BF16_DATATYPE
242:     {at::kBFloat16, ncclBfloat16},
243: #endif // HAS_NCCL_BF16_DATATYPE
244: };
245: 
246: TORCH_API size_t hashTensors(const std::vector<at::Tensor>& tensors);
247: TORCH_API std::string getNcclVersion();
248: TORCH_API std::tuple<int, int, int> getNcclVersionTuple();
249: TORCH_API int getNcclVersionNumber();
250: TORCH_API std::string ncclGetErrorWithVersion(ncclResult_t error);
251: int nccl_nonblocking_timeout();
252: 
253: // Provides additional detail into NCCL error codes based on when these are
254: // thrown in the NCCL codebase.
255: TORCH_API std::string getNcclErrorDetailStr(
256:     ncclResult_t error,
```

- EN: Lines 241-256 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `hashTensors`, `getNcclVersion`, `getNcclVersionTuple`.
- CN: 第 241-256 行使用条件编译来适配特性开关、平台或可选后端；在 `hashTensors`、`getNcclVersion`、`getNcclVersionTuple` 等例程中引入具体执行逻辑。

### Lines 257-272 / 第 257-272 行

```cpp
257:     std::optional<std::string> processGroupFailureReason = std::nullopt);
258: 
259: // Helper function that gets the data type and issues error if not supported
260: ncclDataType_t getNcclDataType(at::ScalarType type);
261: 
262: // RAII wrapper for NCCL communicator
263: class NCCLComm {
264:   using MutexType = std::recursive_mutex;
265:   using LockType = std::unique_lock<MutexType>;
266: 
267:  public:
268:   explicit NCCLComm(ncclComm_t ncclComm);
269: 
270:   NCCLComm() = default;
271: 
272:   ~NCCLComm() noexcept;
```

- EN: Lines 257-272 declares or defines types such as `NCCLComm`; introduces executable logic in routines such as `getNcclDataType`, `NCCLComm`, `~NCCLComm`.
- CN: 第 257-272 行声明或定义了 `NCCLComm` 等类型；在 `getNcclDataType`、`NCCLComm`、`~NCCLComm` 等例程中引入具体执行逻辑。

### Lines 273-288 / 第 273-288 行

```cpp
273: 
274:   void setUniqueHash(ncclUniqueId ncclId);
275:   void setUniqueHash(std::string hash);
276:   std::string getUniqueHash();
277: 
278:   static std::shared_ptr<NCCLComm> create(
279:       int numRanks,
280:       int rank,
281:       ncclUniqueId commId,
282:       at::DeviceIndex deviceIndex);
283: 
284: #ifdef NCCL_HAS_CONFIG
285:   static std::shared_ptr<NCCLComm> create(
286:       int numRanks,
287:       int rank,
288:       ncclUniqueId commId,
```

- EN: Lines 273-288 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `setUniqueHash`, `getUniqueHash`, `create`.
- CN: 第 273-288 行使用条件编译来适配特性开关、平台或可选后端；在 `setUniqueHash`、`getUniqueHash`、`create` 等例程中引入具体执行逻辑。

### Lines 289-304 / 第 289-304 行

```cpp
289:       at::DeviceIndex deviceIndex,
290:       ncclConfig_t& config);
291: #ifdef NCCL_HAS_INIT_RANK_SCALABLE
292:   static std::shared_ptr<NCCLComm> create_scalable(
293:       int numRanks,
294:       int rank,
295:       std::vector<ncclUniqueId>& commIds,
296:       at::DeviceIndex deviceIndex,
297:       ncclConfig_t& config);
298: #endif // NCCL_HAS_INIT_RANK_SCALABLE
299: #endif // NCCL_HAS_CONFIG
300: 
301: #ifdef NCCL_HAS_COMM_SPLIT
302:   static std::shared_ptr<NCCLComm> split(
303:       NCCLComm* source,
304:       int color_id,
```

- EN: Lines 289-304 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `create_scalable`.
- CN: 第 289-304 行使用条件编译来适配特性开关、平台或可选后端；在 `create_scalable` 等例程中引入具体执行逻辑。

### Lines 305-320 / 第 305-320 行

```cpp
305:       int rank,
306:       ncclConfig_t& config);
307: #endif // NCCL_HAS_COMM_SPLIT
308: 
309: #ifdef NCCL_HAS_COMM_SHRINK
310:   static std::shared_ptr<NCCLComm> shrink(
311:       NCCLComm* source,
312:       std::vector<int>& ranks_to_exclude,
313:       ncclConfig_t* config,
314:       int shrinkFlags = 0);
315: #endif // NCCL_HAS_COMM_SHRINK
316: 
317: #if (defined(IS_NCCLX) || defined(USE_ROCM)) && defined(NCCL_COMM_DUMP)
318:   std::unordered_map<std::string, std::string> ncclCommDump();
319: #endif
320: 
```

- EN: Lines 305-320 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `shrink`, `ncclCommDump`.
- CN: 第 305-320 行使用条件编译来适配特性开关、平台或可选后端；在 `shrink`、`ncclCommDump` 等例程中引入具体执行逻辑。

### Lines 321-336 / 第 321-336 行

```cpp
321:   at::DeviceIndex getDeviceIndex();
322: 
323:   // Must not be copyable
324:   NCCLComm(const NCCLComm&) = delete;
325:   NCCLComm& operator=(const NCCLComm&) = delete;
326: 
327:   // Do not support move assignment as there is no valid use case
328:   NCCLComm& operator=(NCCLComm&& other) = delete;
329: 
330:   // Move constructable
331:   // NOLINTNEXTLINE(*-noexcept-move-*)
332:   NCCLComm(NCCLComm&& other);
333: 
334:   ncclComm_t getNcclComm();
335: 
336:   // Wait for the communicator to be ready. This is a blocking function.
```

- EN: Lines 321-336 introduces executable logic in routines such as `getDeviceIndex`, `NCCLComm`, `getNcclComm`.
- CN: 第 321-336 行在 `getDeviceIndex`、`NCCLComm`、`getNcclComm` 等例程中引入具体执行逻辑。

### Lines 337-352 / 第 337-352 行

```cpp
337:   // Useful in nonblocking mode: NCCL requires the communicator to be ready
338:   // before issuing a second command.
339:   // Arguments:
340:   //   longInterval: if true, wait with sleep of an interval; otherwise, wait
341:   //   with `sched_yield` which is faster (but acquires CPU more frequently).
342:   //   Use `longInterval=true` when waiting for initialization or finalize to
343:   //   complete. Use `longInterval=false` when waiting collective call to return
344:   //   ncclSuccess.
345:   void waitReady(bool longInterval);
346: 
347:   std::optional<std::string> getNcclCommFailureReason() const;
348: 
349:   void abort(std::optional<std::string> commFailureReason = std::nullopt);
350: 
351:   // Finalize a communicator -- asking it to flush its operations. When the
352:   // communicator is marked as nonblocking, this is a nonblocking function;
```

- EN: Lines 337-352 introduces executable logic in routines such as `waitReady`, `getNcclCommFailureReason`, `abort`; returns computed state or forwards results to the surrounding caller.
- CN: 第 337-352 行在 `waitReady`、`getNcclCommFailureReason`、`abort` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 353-368 / 第 353-368 行

```cpp
353:   // otherwise, it will block till all operations complete.
354:   void finalize();
355: 
356:   // Destroy a communicator. This is a blocking function.
357:   void destroy();
358: 
359:   bool isInitialized() const;
360: 
361:   bool isAborted() const;
362: 
363:   uint64_t getCommSplitCounter() const;
364: 
365:   ncclResult_t checkForNcclError();
366: 
367:   // Thread-safe wrapper for ncclCommGetAsyncError that acquires the mutex
368:   // before calling the NCCL API. This is needed because NCCL does not provide
```

- EN: Lines 353-368 introduces executable logic in routines such as `finalize`, `destroy`, `isInitialized`.
- CN: 第 353-368 行在 `finalize`、`destroy`、`isInitialized` 等例程中引入具体执行逻辑。

### Lines 369-384 / 第 369-384 行

```cpp
369:   // thread-safety guarantees for ncclCommGetAsyncError, and both the main
370:   // thread and watchdog thread may call it concurrently.
371:   ncclResult_t getAsyncError(ncclResult_t* asyncError);
372: 
373:   ncclResult_t registerSegment(
374:       void* ptr,
375:       size_t size,
376:       bool errorOnRereg = true,
377:       bool window = false);
378: 
379:   ncclResult_t deregisterSegment(void* ptr, bool window = false);
380: 
381:   std::string repr() const;
382: 
383:   // APIs related to memory offload (require NCCL 2.29.7+ at runtime)
384:   void suspend();
```

- EN: Lines 369-384 introduces executable logic in routines such as `getAsyncError`, `registerSegment`, `deregisterSegment`.
- CN: 第 369-384 行在 `getAsyncError`、`registerSegment`、`deregisterSegment` 等例程中引入具体执行逻辑。

### Lines 385-400 / 第 385-400 行

```cpp
385: 
386:   void resume();
387: 
388:   std::unordered_map<std::string, uint64_t> getMemoryStats();
389: 
390:   friend class ProcessGroupNCCL;
391: 
392:  protected:
393:   // Unique hash for this communicator.
394:   std::string uniqueHash_;
395:   bool aborted_{false};
396:   uint64_t ncclCommSplitCounter_{0};
397:   ncclResult_t ncclAsyncErr_{ncclSuccess};
398:   mutable MutexType mutex_;
399:   // Rank that this communicator corresponds to.
400:   int rank_{};
```

- EN: Lines 385-400 introduces executable logic in routines such as `resume`, `getMemoryStats`.
- CN: 第 385-400 行在 `resume`、`getMemoryStats` 等例程中引入具体执行逻辑。

### Lines 401-416 / 第 401-416 行

```cpp
401:   // Optional reason for communicator failure, provided by ProcessGroupNCCL for
402:   // better error messaging.
403:   std::optional<std::string> commFailureReason_;
404:   bool initialized_{false};
405:   // Whether this communicator is using nonblocking mode. Recorded during comm
406:   // creation or split. For safety, we give a default value of true (more
407:   // protection).
408:   bool nonBlocking_{true};
409:   // Device index for which the NCCL comm is created
410:   at::DeviceIndex deviceIndex_{-1};
411: #ifdef NCCL_HAS_COMM_REGISTER
412:   // Stores handlers for tensors registered by NCCL
413:   std::unordered_map<void*, void*> registeredSegmentHandles_;
414: #endif // NCCL_HAS_COMM_REGISTER
415: 
416:  private:
```

- EN: Lines 401-416 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 401-416 行使用条件编译来适配特性开关、平台或可选后端。

### Lines 417-432 / 第 417-432 行

```cpp
417:   ncclComm_t ncclComm_{nullptr};
418: };
419: 
420: // Helper that automatically cleans up premul sums.
421: struct ncclRedOpRAII {
422:   ncclRedOpRAII() = default;
423:   ncclRedOpRAII(ncclRedOp_t op) : op_(op) {}
424:   ncclRedOpRAII(ncclRedOp_t op, ncclComm_t comm)
425:       : op_(op), comm_(comm), premul_sum_(true) {}
426:   ncclRedOpRAII(const ncclRedOpRAII&) = delete;
427:   ncclRedOpRAII& operator=(const ncclRedOpRAII&) = delete;
428:   ncclRedOpRAII(ncclRedOpRAII&& tmp) noexcept : ncclRedOpRAII() {
429:     std::swap(tmp.op_, this->op_);
430:     std::swap(tmp.comm_, this->comm_);
431:     std::swap(tmp.premul_sum_, this->premul_sum_);
432:   }
```

- EN: Lines 417-432 declares or defines types such as `ncclRedOpRAII`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 417-432 行声明或定义了 `ncclRedOpRAII` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 433-448 / 第 433-448 行

```cpp
433: #if defined(ENABLE_NCCL_PREMUL_SUM_SUPPORT)
434:   ~ncclRedOpRAII() {
435:     if (premul_sum_) {
436:       ncclRedOpDestroy(op_, comm_);
437:     }
438:   }
439: #endif // ENABLE_NCCL_PREMUL_SUM_SUPPORT
440:   operator ncclRedOp_t() const {
441:     return op_;
442:   }
443:   ncclRedOp_t op_{};
444:   ncclComm_t comm_{};
445:   bool premul_sum_ = false;
446: };
447: 
448: void printNcclCommProxyTrace(
```

- EN: Lines 433-448 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `~ncclRedOpRAII`, `ncclRedOp_t`.
- CN: 第 433-448 行使用条件编译来适配特性开关、平台或可选后端；在 `~ncclRedOpRAII`、`ncclRedOp_t` 等例程中引入具体执行逻辑。

### Lines 449-453 / 第 449-453 行

```cpp
449:     const std::string& dumpReason,
450:     const std::unordered_map<std::string, std::string>& dumpMap);
451: } // namespace c10d
452: 
453: #endif // USE_C10D_NCCL
```

- EN: Lines 449-453 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 449-453 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `NCCLComm`, `ncclRedOpRAII`
- CN: 核心符号：`NCCLComm`、`ncclRedOpRAII`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: None / 无
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ATen.h`, `ATen/cuda/CUDAEvent.h`, `c10/util/Exception.h`, `torch/csrc/cuda/nccl.h`
- External or system headers / 外部或系统头文件: `sched.h`, `cstdio`, `cstdlib`, `memory`, `mutex`, `nccl.h`, `optional`
- Local symbols / 本地符号: `NCCLComm`, `ncclRedOpRAII`