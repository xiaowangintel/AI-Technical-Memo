# rpc_agent.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/rpc_agent.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for rpc agent in the distributed RPC layer. Representative routines include `TORCH_CHECK`, `shutdown`, `startImpl`, `TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`, `shutdownImpl`.
- 用途 (CN): 该文件在分布式 RPC 层中提供rpc agent 的实现逻辑。 代表性例程包括 `TORCH_CHECK`、`shutdown`、`startImpl`、`TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`、`shutdownImpl`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <c10/util/DeadlockDetection.h>
2: #include <torch/csrc/distributed/rpc/rpc_agent.h>
3: 
4: namespace torch::distributed::rpc {
5: 
6: RegisterWorkerInfoOnce::RegisterWorkerInfoOnce() {
7:   // WorkerInfo needs to be registered exactly once. Since the op registration
8:   // happens in libtorch_python we wrap the class registration in a helper to
9:   // make sure that if there's multiple copies of Python such as used in
10:   // torch::deploy we only ever register it once.
11:   static auto workerInfo = torch::class_<WorkerInfo>("dist_rpc", "WorkerInfo")
12:                                .def(torch::init<std::string, int64_t>());
13: }
14: 
15: WorkerInfo::WorkerInfo(std::string name, int64_t id)
16:     : WorkerInfo(std::move(name), static_cast<worker_id_t>(id)) {
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 17-32 / 第 17-32 行

```cpp
17:   TORCH_CHECK(
18:       id <= std::numeric_limits<worker_id_t>::max(),
19:       "RPC worker id ",
20:       id,
21:       " out of bound of int16_t.");
22: }
23: 
24: WorkerInfo::WorkerInfo(std::string name, worker_id_t id)
25:     : name_(std::move(name)), id_(id) {
26:   bool validSize = name_.length() < MAX_NAME_LEN && !name_.empty();
27:   bool validChar =
28:       std::find_if(name_.begin(), name_.end(), [](char c) {
29:         return !(std::isalnum(c) || c == '-' || c == '_' || c == ':');
30:       }) == name_.end();
31:   TORCH_CHECK(
32:       validSize && validChar,
```

- EN: Lines 17-32 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 17-32 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 33-48 / 第 33-48 行

```cpp
33:       "Worker name must match ^[A-Za-z0-9-_:]*$, "
34:       "and must be non-empty and shorter than ",
35:       MAX_NAME_LEN,
36:       " chars, "
37:       "but got ",
38:       name_);
39: }
40: 
41: // Large Time Duration for waiting on the condition variable until the map is
42: // population. Cannot use
43: // std::chrono::time_point<std::chrono::steady_clock>::max() due to a known
44: // overflow-related bug.
45: constexpr auto kLargeTimeDuration = std::chrono::hours(10000);
46: 
47: RpcAgent::RpcAgent(
48:     WorkerInfo workerId,
```

- EN: Lines 33-48 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 33-48 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 49-64 / 第 49-64 行

```cpp
49:     std::unique_ptr<RequestCallback> cb,
50:     std::chrono::milliseconds rpcTimeout)
51:     : workerInfo_(std::move(workerId)),
52:       cb_(std::move(cb)),
53:       rpcTimeout_(rpcTimeout),
54:       profilingEnabled_(false),
55:       rpcAgentRunning_(false) {}
56: 
57: RpcAgent::~RpcAgent() {
58:   if (rpcAgentRunning_.load()) {
59:     shutdown();
60:   }
61: }
62: 
63: void RpcAgent::start() {
64:   rpcAgentRunning_.store(true);
```

- EN: Lines 49-64 introduces executable logic in routines such as `shutdown`.
- CN: 第 49-64 行在 `shutdown` 等例程中引入具体执行逻辑。

### Lines 65-80 / 第 65-80 行

```cpp
65:   rpcRetryThread_ = std::thread(&RpcAgent::retryExpiredRpcs, this);
66:   startImpl();
67: }
68: 
69: void RpcAgent::shutdown() {
70:   TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP();
71:   std::unique_lock<std::mutex> lock(rpcRetryMutex_);
72:   rpcAgentRunning_.store(false);
73:   lock.unlock();
74:   rpcRetryMapCV_.notify_one();
75:   if (rpcRetryThread_.joinable()) {
76:     rpcRetryThread_.join();
77:   }
78:   // NOLINTNEXTLINE(clang-analyzer-cplusplus.PureVirtualCall)
79:   shutdownImpl();
80: }
```

- EN: Lines 65-80 introduces executable logic in routines such as `startImpl`, `TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`, `shutdownImpl`.
- CN: 第 65-80 行在 `startImpl`、`TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`、`shutdownImpl` 等例程中引入具体执行逻辑。

### Lines 81-96 / 第 81-96 行

```cpp
81: 
82: c10::intrusive_ptr<JitFuture> RpcAgent::sendWithRetries(
83:     const WorkerInfo& to,
84:     c10::intrusive_ptr<Message> message,
85:     RpcRetryOptions retryOptions) {
86:   TORCH_CHECK(retryOptions.maxRetries >= 0, "maxRetries cannot be negative.");
87:   TORCH_CHECK(
88:       retryOptions.retryBackoff >= 1,
89:       "maxRetries cannot be exponentially decaying.");
90:   TORCH_CHECK(
91:       retryOptions.rpcRetryDuration.count() >= 0,
92:       "rpcRetryDuration cannot be negative.");
93: 
94:   auto originalFuture =
95:       c10::make_intrusive<JitFuture>(at::AnyClassType::get(), getDevices());
96:   steady_clock_time_point newTime =
```

- EN: Lines 81-96 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 81-96 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 97-112 / 第 97-112 行

```cpp
97:       computeNewRpcRetryTime(retryOptions, /* retryCount */ 0);
98:   auto firstRetryRpc = std::make_shared<RpcRetryInfo>(
99:       to,
100:       message,
101:       originalFuture,
102:       /* retryCount */ 0,
103:       retryOptions);
104:   auto jitFuture = send(to, std::move(message));
105:   jitFuture->addCallback([this, newTime, firstRetryRpc](JitFuture& future) {
106:     rpcRetryCallback(future, newTime, firstRetryRpc);
107:   });
108: 
109:   return originalFuture;
110: }
111: 
112: void RpcAgent::retryExpiredRpcs() {
```

- EN: Lines 97-112 introduces executable logic in routines such as `computeNewRpcRetryTime`; returns computed state or forwards results to the surrounding caller.
- CN: 第 97-112 行在 `computeNewRpcRetryTime` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 113-128 / 第 113-128 行

```cpp
113:   // Stores the retried futures so callbacks can be added outside the lock.
114:   std::vector<
115:       std::pair<c10::intrusive_ptr<JitFuture>, std::shared_ptr<RpcRetryInfo>>>
116:       futures;
117:   // Stores futures and exception messages for non-retriable error-ed futures.
118:   std::vector<std::pair<c10::intrusive_ptr<JitFuture>, std::string>>
119:       errorFutures;
120: 
121:   while (rpcAgentRunning_.load()) {
122:     std::unique_lock<std::mutex> lock(rpcRetryMutex_);
123: 
124:     // We must continue sleeping as long as the RPC Agent is running and when
125:     // either the Retry Map is empty, or when the Retry Map's earliest expiring
126:     // RPC is set to be retried in the future.
127:     steady_clock_time_point earliestTimeout =
128:         std::chrono::steady_clock::now() + kLargeTimeDuration;
```

- EN: Lines 113-128 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 113-128 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 129-144 / 第 129-144 行

```cpp
129: 
130:     for (;;) {
131:       if (!rpcAgentRunning_.load())
132:         return;
133:       if (std::chrono::steady_clock::now() >= earliestTimeout)
134:         break;
135:       if (!rpcRetryMap_.empty()) {
136:         earliestTimeout = rpcRetryMap_.begin()->first;
137:       }
138:       rpcRetryMapCV_.wait_until(lock, earliestTimeout);
139:     }
140: 
141:     // Updating these since something may have been added to the map while this
142:     // thread was sleeping.
143:     earliestTimeout = rpcRetryMap_.begin()->first;
144:     auto& earliestRpcList = rpcRetryMap_.begin()->second;
```

- EN: Lines 129-144 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 129-144 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 145-160 / 第 145-160 行

```cpp
145: 
146:     // We iterate through all the RPC's set to be retried at the current
147:     // timepoint, resend those RPC's, and add the RPC's and their futures to
148:     // a list to later attach callbacks. These callbacks either schedule
149:     // the RPC for a future retry or marks it with success/error depending on
150:     // the outcome of the current send. Then, we clean up the rpcRetryMap_.
151:     for (auto it = earliestRpcList.begin(); it != earliestRpcList.end();
152:          /* no increment */) {
153:       auto& earliestRpc = *it;
154:       c10::intrusive_ptr<JitFuture> jitFuture;
155: 
156:       // send() will throw an exception if an RPC is retried while the agent is
157:       // shutdown. We must catch this exception and mark the original future
158:       // with an error, since this RPC never succeeded and can no longer be
159:       // retried.
160:       try {
```

- EN: Lines 145-160 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-160 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-176 / 第 161-176 行

```cpp
161:         jitFuture = send(earliestRpc->to_, earliestRpc->message_);
162:         futures.emplace_back(jitFuture, earliestRpc);
163:       } catch (std::exception& e) {
164:         // We must store the futures and exception messages here and only mark
165:         // the futures with an error after releasing the lock.
166:         errorFutures.emplace_back(earliestRpc->originalFuture_, e.what());
167:       }
168: 
169:       // A callback will be attached to all futures for the retries in this
170:       // list. Thus they will either be rescheduled for future retries or they
171:       // will be marked as complete. We can safely delete them from the retry
172:       // Map for the current timepoint.
173:       it = earliestRpcList.erase(it);
174:     }
175: 
176:     // If there are no more RPC's set to be retried at the current timepoint,
```

- EN: Lines 161-176 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 161-176 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 177-192 / 第 177-192 行

```cpp
177:     // we can remove the corresponding unordered_set from the retry map.
178:     if (earliestRpcList.empty()) {
179:       rpcRetryMap_.erase(earliestTimeout);
180:     }
181: 
182:     lock.unlock();
183:     // We attach callbacks to the futures outside of the lock to prevent
184:     // potential deadlocks.
185:     for (const auto& it : futures) {
186:       auto jitFuture = it.first;
187:       auto earliestRpc = it.second;
188:       steady_clock_time_point newTime = computeNewRpcRetryTime(
189:           earliestRpc->options_, earliestRpc->retryCount_);
190:       earliestRpc->retryCount_++;
191: 
192:       jitFuture->addCallback([this, newTime, earliestRpc](JitFuture& future) {
```

- EN: Lines 177-192 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 177-192 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 193-208 / 第 193-208 行

```cpp
193:         rpcRetryCallback(future, newTime, earliestRpc);
194:       });
195:     }
196:     futures.clear();
197: 
198:     // For exceptions caught while retrying RPC's above, we set those futures
199:     // with errors now that we have released the lock.
200:     for (const auto& it : errorFutures) {
201:       auto errorFuture = it.first;
202:       auto errorMsg = it.second;
203:       errorFuture->setError(
204:           std::make_exception_ptr(std::runtime_error(errorMsg)));
205:     }
206:     errorFutures.clear();
207:   }
208: }
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 209-224 / 第 209-224 行

```cpp
209: 
210: void RpcAgent::rpcRetryCallback(
211:     JitFuture& jitFuture,
212:     steady_clock_time_point newTime,
213:     std::shared_ptr<RpcRetryInfo> earliestRpc) {
214:   if (jitFuture.hasError()) {
215:     // Adding one since we want to include the original send as well and not
216:     // just the retry count.
217:     LOG(INFO) << "Send try " << (earliestRpc->retryCount_ + 1) << " failed";
218:     if (!rpcAgentRunning_.load()) {
219:       // If the RPC Agent has shutdown, we cannot retry messages. Thus we mark
220:       // the future with an error since the RPC was never completed
221:       // successfully.
222:       earliestRpc->originalFuture_->setError(jitFuture.exception_ptr());
223:     } else if (earliestRpc->retryCount_ < earliestRpc->options_.maxRetries) {
224:       // If the previous future completed with an error and we haven't
```

- EN: Lines 209-224 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 209-224 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 225-240 / 第 225-240 行

```cpp
225:       // completed maxRetries send attempts, we move the earliestRpc
226:       // struct to a new time point in the retry map (effectively
227:       // scheduling it for a future retry.)
228:       {
229:         std::lock_guard<std::mutex> retryMapLock(rpcRetryMutex_);
230:         rpcRetryMap_[newTime].emplace(std::move(earliestRpc));
231:       }
232:       // The retry thread waits for the map to be populated. Thus we notify
233:       // once an item has been added.
234:       rpcRetryMapCV_.notify_one();
235:     } else {
236:       // We have completed maxRetries send attempts. We're now marking
237:       // the future with an error.
238:       std::string errorMessage = c10::str(
239:           "The RPC has not succeeded after the specified number of max retries (",
240:           earliestRpc->options_.maxRetries,
```

- EN: Lines 225-240 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 225-240 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 241-256 / 第 241-256 行

```cpp
241:           ").");
242:       earliestRpc->originalFuture_->setError(
243:           std::make_exception_ptr(std::runtime_error(errorMessage)));
244:     }
245:   } else {
246:     // This try succeeded, so we can make the original future as complete.
247:     earliestRpc->originalFuture_->markCompleted(
248:         jitFuture.value(), jitFuture.storages());
249:   }
250: }
251: 
252: const WorkerInfo& RpcAgent::getWorkerInfo() const {
253:   return workerInfo_;
254: }
255: 
256: std::shared_ptr<RpcAgent> RpcAgent::currentRpcAgent_ = nullptr;
```

- EN: Lines 241-256 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 241-256 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 257-272 / 第 257-272 行

```cpp
257: 
258: bool RpcAgent::isCurrentRpcAgentSet() {
259:   return std::atomic_load(&currentRpcAgent_) != nullptr;
260: }
261: 
262: std::shared_ptr<RpcAgent> RpcAgent::getCurrentRpcAgent() {
263:   std::shared_ptr<RpcAgent> agent = std::atomic_load(&currentRpcAgent_);
264:   TORCH_CHECK(
265:       agent,
266:       "Current RPC agent is not set! Did you initialize the RPC "
267:       "framework (e.g. by calling `rpc.init_rpc`)?");
268:   return agent;
269: }
270: 
271: void RpcAgent::setCurrentRpcAgent(std::shared_ptr<RpcAgent> rpcAgent) {
272:   if (rpcAgent) {
```

- EN: Lines 257-272 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 257-272 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 273-288 / 第 273-288 行

```cpp
273:     std::shared_ptr<RpcAgent> previousAgent;
274:     // Use compare_exchange so that we don't actually perform the exchange if
275:     // that would trigger the assert just below. See:
276:     // https://en.cppreference.com/w/cpp/atomic/atomic_compare_exchange
277:     std::atomic_compare_exchange_strong(
278:         &currentRpcAgent_, &previousAgent, std::move(rpcAgent));
279:     TORCH_INTERNAL_ASSERT(
280:         previousAgent == nullptr, "Current RPC agent is set!");
281:   } else {
282:     // We can't use compare_exchange (we don't know what value to expect) but we
283:     // don't need to, as the only case that would trigger the assert is if we
284:     // replaced nullptr with nullptr, which we can just do as it has no effect.
285:     std::shared_ptr<RpcAgent> previousAgent =
286:         std::atomic_exchange(&currentRpcAgent_, std::move(rpcAgent));
287:     TORCH_INTERNAL_ASSERT(
288:         previousAgent != nullptr, "Current RPC agent is not set!");
```

- EN: Lines 273-288 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`.
- CN: 第 273-288 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑。

### Lines 289-304 / 第 289-304 行

```cpp
289:   }
290: }
291: 
292: void RpcAgent::setTypeResolver(std::shared_ptr<TypeResolver> typeResolver) {
293:   typeResolver_ = std::move(typeResolver);
294: }
295: 
296: std::shared_ptr<TypeResolver> RpcAgent::getTypeResolver() {
297:   TORCH_INTERNAL_ASSERT(typeResolver_, "Type resolver is not set!");
298:   return typeResolver_;
299: }
300: 
301: void RpcAgent::enableGILProfiling(bool flag) {
302:   profilingEnabled_ = flag;
303: }
304: 
```

- EN: Lines 289-304 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`; returns computed state or forwards results to the surrounding caller.
- CN: 第 289-304 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 305-320 / 第 305-320 行

```cpp
305: bool RpcAgent::isGILProfilingEnabled() {
306:   return profilingEnabled_.load();
307: }
308: 
309: DeviceMap RpcAgent::getDeviceMap(const WorkerInfo& /* unused */) const {
310:   // Default implementation has no device map.
311:   return {};
312: }
313: 
314: const std::vector<c10::Device>& RpcAgent::getDevices() const {
315:   // By default the agent is CPU-only.
316:   static const std::vector<c10::Device> noDevices = {};
317:   return noDevices;
318: }
319: 
320: std::unordered_map<std::string, std::string> RpcAgent::getDebugInfo() {
```

- EN: Lines 305-320 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 305-320 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 321-332 / 第 321-332 行

```cpp
321:   /* This would later include more info other than metrics for eg: may include
322:      stack traces for the threads owned by the agent */
323:   // Default implementation: return getMetrics().
324:   return getMetrics();
325: }
326: 
327: std::ostream& operator<<(std::ostream& os, const WorkerInfo& workerInfo) {
328:   return os << "WorkerInfo(id=" << workerInfo.id_
329:             << ", name=" << workerInfo.name_ << ')';
330: }
331: 
332: } // namespace torch::distributed::rpc
```

- EN: Lines 321-332 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `getMetrics`, `operator<<`.
- CN: 第 321-332 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `getMetrics`、`operator<<` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `TORCH_CHECK`, `shutdown`, `startImpl`, `TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`, `shutdownImpl`, `computeNewRpcRetryTime`
- CN: 核心符号：`TORCH_CHECK`、`shutdown`、`startImpl`、`TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`、`shutdownImpl`、`computeNewRpcRetryTime`
- EN: Notable themes: store/state coordination, RPC/RRef semantics.
- CN: 值得关注的主题：存储/状态协调、RPC/RRef 语义。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/rpc/rpc_agent.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/util/DeadlockDetection.h`
- External or system headers / 外部或系统头文件: None / 无
- Local symbols / 本地符号: `TORCH_CHECK`, `shutdown`, `startImpl`, `TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`, `shutdownImpl`, `computeNewRpcRetryTime`, `TORCH_INTERNAL_ASSERT`, `getMetrics`