# rpc_agent.h — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/rpc_agent.h`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for rpc agent in the distributed RPC layer. Key types include `TORCH_API`, `hash`.
- 用途 (CN): 该文件在分布式 RPC 层中提供rpc agent 的接口与类型声明。 关键类型包括 `TORCH_API`、`hash`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: 
3: #include <torch/csrc/distributed/rpc/message.h>
4: #include <torch/csrc/distributed/rpc/request_callback.h>
5: #include <torch/csrc/distributed/rpc/types.h>
6: 
7: #include <cctype>
8: #include <chrono>
9: #include <condition_variable>
10: #include <mutex>
11: #include <thread>
12: 
13: namespace torch::distributed::rpc {
14: 
15: using DeviceMap = std::unordered_map<c10::Device, c10::Device>;
16: 
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 17-32 / 第 17-32 行

```cpp
17: // Default RPC timeout
18: constexpr float kDefaultRpcTimeoutSeconds = 60;
19: // Unset RPC timeout. This is the value agent::send() will have if user does not
20: // pass in a specific timeout, and indicates that we must use the default
21: // timeout for RPCs.
22: constexpr float kUnsetRpcTimeout = -1;
23: constexpr auto kDefaultInitMethod = "env://";
24: constexpr float kSecToMsConversion = 1000;
25: constexpr auto kRpcTimeoutErrorStr =
26:     "RPC ran for more than set timeout ({} ms) and will now be marked with an error";
27: constexpr auto kDefaultNumWorkerThreads = 16;
28: 
29: using steady_clock_time_point =
30:     std::chrono::time_point<std::chrono::steady_clock>;
31: // Input is qualified name string, output is JIT StrongTypePtr
32: // Same as jit::TypeResolver, did not import jit::TypeResolver to here
```

- EN: Lines 17-32 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 17-32 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 33-48 / 第 33-48 行

```cpp
33: // because it could introduce cyclic dependencies.
34: using TypeResolver =
35:     std::function<c10::StrongTypePtr(const c10::QualifiedName&)>;
36: 
37: struct TORCH_API RpcBackendOptions {
38:   RpcBackendOptions()
39:       : RpcBackendOptions(kDefaultRpcTimeoutSeconds, kDefaultInitMethod) {}
40: 
41:   RpcBackendOptions(float rpcTimeoutSeconds, std::string initMethod)
42:       : rpcTimeoutSeconds(rpcTimeoutSeconds),
43:         initMethod(std::move(initMethod)) {
44:     TORCH_CHECK(rpcTimeoutSeconds >= 0, "RPC Timeout must be non-negative");
45:   }
46: 
47:   float rpcTimeoutSeconds;
48:   std::string initMethod;
```

- EN: Lines 33-48 declares or defines types such as `TORCH_API`; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 33-48 行声明或定义了 `TORCH_API` 等类型；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 49-64 / 第 49-64 行

```cpp
49: };
50: 
51: // A globally unique ID to identify an RpcAgent
52: struct TORCH_API WorkerInfo : torch::CustomClassHolder {
53:   WorkerInfo(std::string name, int64_t id);
54: 
55:   WorkerInfo(std::string name, worker_id_t id);
56: 
57:   bool operator==(const WorkerInfo& rhs) {
58:     return (id_ == rhs.id_) && (name_ == rhs.name_);
59:   }
60: 
61:   static constexpr size_t MAX_NAME_LEN = 128;
62: 
63:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
64:   const std::string name_;
```

- EN: Lines 49-64 declares or defines types such as `TORCH_API`; introduces executable logic in routines such as `WorkerInfo`, `operator==`.
- CN: 第 49-64 行声明或定义了 `TORCH_API` 等类型；在 `WorkerInfo`、`operator==` 等例程中引入具体执行逻辑。

### Lines 65-80 / 第 65-80 行

```cpp
65:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
66:   const worker_id_t id_;
67: };
68: 
69: struct TORCH_API RegisterWorkerInfoOnce {
70:   RegisterWorkerInfoOnce();
71: };
72: 
73: TORCH_API std::ostream& operator<<(
74:     std::ostream& os,
75:     const WorkerInfo& workerInfo);
76: 
77: // Struct for options to configure the RPC Retry protocol.
78: struct TORCH_API RpcRetryOptions {
79:   // Using a default constructor like all other Options structs in the RPC
80:   // codebase. TORCH_CHECKs for input validation are done in the
```

- EN: Lines 65-80 declares or defines types such as `TORCH_API`; introduces executable logic in routines such as `RegisterWorkerInfoOnce`, `operator<<`; performs validation and error handling to keep distributed state consistent.
- CN: 第 65-80 行声明或定义了 `TORCH_API` 等类型；在 `RegisterWorkerInfoOnce`、`operator<<` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 81-96 / 第 81-96 行

```cpp
81:   // sendWithRetries function.
82:   RpcRetryOptions() = default;
83:   // Maximum number of times we will retry the RPC
84:   int maxRetries{5};
85:   // Initial duration between consecutive RPC send attempts
86:   std::chrono::milliseconds rpcRetryDuration{std::chrono::milliseconds(1000)};
87:   // Constant for exponential backoff used while calculating future wait
88:   // durations
89:   float retryBackoff{1.5};
90: };
91: 
92: // Struct that stores all the metadata needed to retry a given RPC.
93: struct TORCH_API RpcRetryInfo {
94:   RpcRetryInfo(
95:       const WorkerInfo& to,
96:       c10::intrusive_ptr<Message> message,
```

- EN: Lines 81-96 declares or defines types such as `TORCH_API`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-96 行声明或定义了 `TORCH_API` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 97-112 / 第 97-112 行

```cpp
97:       c10::intrusive_ptr<JitFuture> originalFuture,
98:       int retryCount,
99:       RpcRetryOptions options)
100:       : to_(to),
101:         message_(std::move(message)),
102:         originalFuture_(std::move(originalFuture)),
103:         retryCount_(retryCount),
104:         options_(options) {}
105: 
106:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
107:   const WorkerInfo& to_;
108:   c10::intrusive_ptr<Message> message_;
109:   // Future that is returned to the caller of sendWithRetries().
110:   c10::intrusive_ptr<JitFuture> originalFuture_;
111:   // Number of send attempts completed so far.
112:   int retryCount_;
```

- EN: Lines 97-112 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 97-112 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 113-128 / 第 113-128 行

```cpp
113:   RpcRetryOptions options_;
114: };
115: 
116: // ``RpcAgent`` is the base class for sending and receiving RPC messages. It
117: // provides a unified ``send`` API for both request and response messages, and
118: // will invoke the given ``RequestCallback`` to process received requests. It
119: // should immediately become ready to serve request and accept response after
120: // construction.
121: class TORCH_API RpcAgent {
122:  public:
123:   // `WorkerInfo` is the globally unique identifier for this RpcAgent instance.
124:   // It contains a ``name_`` field and an ``id_`` field. ``name_`` is the
125:   // globally unique name for this ``RpcAgent``. It is up to the ``RpcAgent``
126:   // implementation to determine how to resolve names. ``id_`` is the globally
127:   // unique ID for this ``RpcAgent``. This should be determined by the
128:   // ``RpcAgent`` implementation.
```

- EN: Lines 113-128 declares or defines types such as `TORCH_API`.
- CN: 第 113-128 行声明或定义了 `TORCH_API` 等类型。

### Lines 129-144 / 第 129-144 行

```cpp
129:   // The ``RequestCallback`` will be invoked to handle received requests. This
130:   // ``RpcAgent`` base class makes no assumption on the thread-safeness of the
131:   // ``RequestCallback``. ``RpcAgent`` implementations need to make sure that
132:   // its threading model conform to ``RequestCallback``'s requirement.
133:   // NB: RpcAgent implementations should not start serving requests until
134:   // ``start()`` is called, as there could be other contexts that have not been
135:   // initialized yet at this time.
136:   RpcAgent(
137:       WorkerInfo id,
138:       std::unique_ptr<RequestCallback> cb,
139:       std::chrono::milliseconds rpcTimeout);
140: 
141:   virtual ~RpcAgent();
142: 
143:   // Send a message to the ``RpcAgent`` of id ``to`` and returns a
144:   // ``JitFuture`` ptr. The implementation must be asynchronous, i.e., it
```

- EN: Lines 129-144 introduces executable logic in routines such as `RpcAgent`, `~RpcAgent`.
- CN: 第 129-144 行在 `RpcAgent`、`~RpcAgent` 等例程中引入具体执行逻辑。

### Lines 145-160 / 第 145-160 行

```cpp
145:   // cannot block until it receives the response.
146:   //
147:   // If ``message.isRequest()`` is true, the ``JitFuture`` will be
148:   // completed when the response arrives. For other message types, the Future
149:   // should be ignored by the caller.
150:   virtual c10::intrusive_ptr<JitFuture> send(
151:       const WorkerInfo& to,
152:       c10::intrusive_ptr<Message> message,
153:       const float rpcTimeoutSeconds = kUnsetRpcTimeout,
154:       const DeviceMap& deviceMap = {}) = 0;
155: 
156:   // Retries sending the message up to maxRetries times until an ACK is
157:   // received. The duration between consecutive sends is increased over
158:   // time using an exponential backoff algorithm.
159:   //
160:   // Sends ``message`` to the ``RpcAgent`` of id ``to`` and returns a
```

- EN: Lines 145-160 continues the local implementation details and data flow for this file.
- CN: 第 145-160 行继续展开本文件的局部实现细节与数据流。

### Lines 161-176 / 第 161-176 行

```cpp
161:   // ``JitFuture`` ptr, just like send(). Caller can specify the maximum
162:   // number of retries for this RPC (default is 5), initial duration between
163:   // sends (default is 1000ms), and backoff constant (default is 1.5) by
164:   // passing in the RpcRetryOptions struct. This API might end up
165:   // executing a method twice on the remote end (it does not guarantee
166:   // exactly-once semantics). Therefore, the user must ensure their requests
167:   // are idempotent.
168:   c10::intrusive_ptr<JitFuture> sendWithRetries(
169:       const WorkerInfo& to,
170:       c10::intrusive_ptr<Message> message,
171:       RpcRetryOptions retryOptions = RpcRetryOptions());
172: 
173:   // Return a reference to the ``WorkerInfo`` of this RpcAgent.
174:   // NB: not using ``std::optional<const std::string&>`` here because we might
175:   // need to create a separate RPC API lib and avoid forcing all ``RpcAgent``
176:   // implementations to depend on libtorch.
```

- EN: Lines 161-176 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 161-176 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 177-192 / 第 177-192 行

```cpp
177:   const WorkerInfo& getWorkerInfo() const;
178: 
179:   // Return a reference to the ``WorkerInfo`` of the given ``workerName``.
180:   virtual const WorkerInfo& getWorkerInfo(
181:       const std::string& workerName) const = 0;
182: 
183:   virtual const WorkerInfo& getWorkerInfo(worker_id_t id) const = 0;
184: 
185:   virtual std::vector<WorkerInfo> getWorkerInfos() const = 0;
186: 
187:   // Retrieve the timeout for all RPCs.
188:   inline std::chrono::milliseconds getRpcTimeout() const {
189:     return rpcTimeout_.load();
190:   }
191: 
192:   // Set the timeout for all RPCs
```

- EN: Lines 177-192 introduces executable logic in routines such as `getWorkerInfo`, `getRpcTimeout`; returns computed state or forwards results to the surrounding caller.
- CN: 第 177-192 行在 `getWorkerInfo`、`getRpcTimeout` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 193-208 / 第 193-208 行

```cpp
193:   inline void setRpcTimeout(const std::chrono::milliseconds& rpcTimeout) {
194:     rpcTimeout_.store(rpcTimeout);
195:   }
196: 
197:   // Call sync and join all internal threads. This method should be called
198:   // before every RPC process exits.
199:   virtual void join(bool shutdown = false, float timeout = 0) = 0;
200: 
201:   // Synchronize the this process with other ``RpcAgent`` processes. Block until
202:   // all ``RpcAgent``s reach this method and send all pending messages.
203:   virtual void sync() = 0;
204: 
205:   // Sets up backend-agnostic state for accepting requests. Currently, this
206:   // entails setting rpcAgentRunning_ to true, creating the retry thread, and
207:   // calling the backend's startImpl.
208:   void start();
```

- EN: Lines 193-208 introduces executable logic in routines such as `setRpcTimeout`, `start`.
- CN: 第 193-208 行在 `setRpcTimeout`、`start` 等例程中引入具体执行逻辑。

### Lines 209-224 / 第 209-224 行

```cpp
209: 
210:   // Derived classes must override this function to start accepting requests.
211:   // This is used to initialize any backend-specific state. Users must call
212:   // start, not startImpl, to initialize the RPC Agent.
213:   virtual void startImpl() = 0;
214: 
215:   // Stop accepting requests and shutdown the RPC framework as soon as possible
216:   // by terminating all RPC threads.
217:   void shutdown();
218: 
219:   // Derived classes must override this function to start accepting requests.
220:   // THis is used to clean up any backend-specific state. Users must call
221:   // shutdown, not shutdownImpl, to shutdown the RPC Agent.
222:   virtual void shutdownImpl() = 0;
223: 
224:   // Check if current RPC agent is set.
```

- EN: Lines 209-224 introduces executable logic in routines such as `shutdown`.
- CN: 第 209-224 行在 `shutdown` 等例程中引入具体执行逻辑。

### Lines 225-240 / 第 225-240 行

```cpp
225:   static bool isCurrentRpcAgentSet();
226: 
227:   // Retrieve the valid current RPC agent.
228:   static std::shared_ptr<RpcAgent> getCurrentRpcAgent();
229: 
230:   // Set the current RPC agent.
231:   static void setCurrentRpcAgent(std::shared_ptr<RpcAgent> rpcAgent);
232: 
233:   // Retrieve metrics as KV map
234:   virtual std::unordered_map<std::string, std::string> getMetrics() = 0;
235: 
236:   // Retrieve debug info in addition to metrics as KV map
237:   virtual std::unordered_map<std::string, std::string> getDebugInfo();
238: 
239:   // Flag to control whether GIL wait times
240:   // should be profiled or not.
```

- EN: Lines 225-240 introduces executable logic in routines such as `isCurrentRpcAgentSet`, `getCurrentRpcAgent`, `setCurrentRpcAgent`.
- CN: 第 225-240 行在 `isCurrentRpcAgentSet`、`getCurrentRpcAgent`、`setCurrentRpcAgent` 等例程中引入具体执行逻辑。

### Lines 241-256 / 第 241-256 行

```cpp
241:   void enableGILProfiling(bool flag);
242: 
243:   // Retrieve whether we should profile GIL wait times or not.
244:   bool isGILProfilingEnabled();
245: 
246:   // Set type resolver that will be passed to JIT pickler to resolver type Ptr
247:   // based on type str.
248:   void setTypeResolver(std::shared_ptr<TypeResolver> typeResolver);
249: 
250:   // Get the type resolver
251:   std::shared_ptr<TypeResolver> getTypeResolver();
252: 
253:   // Retrieves the device map for the provided destination worker.
254:   virtual DeviceMap getDeviceMap(const WorkerInfo& dst) const;
255: 
256:   // Retrieve the (non-CPU) devices that are supported by the agent.
```

- EN: Lines 241-256 introduces executable logic in routines such as `enableGILProfiling`, `isGILProfilingEnabled`, `setTypeResolver`.
- CN: 第 241-256 行在 `enableGILProfiling`、`isGILProfilingEnabled`、`setTypeResolver` 等例程中引入具体执行逻辑。

### Lines 257-272 / 第 257-272 行

```cpp
257:   virtual const std::vector<c10::Device>& getDevices() const;
258: 
259:  protected:
260:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
261:   const WorkerInfo workerInfo_;
262:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
263:   const std::unique_ptr<RequestCallback> cb_;
264:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
265:   std::atomic<std::chrono::milliseconds> rpcTimeout_;
266:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
267:   std::atomic<bool> profilingEnabled_;
268:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
269:   std::shared_ptr<TypeResolver> typeResolver_;
270:   // Atomic boolean indicating whether this agent is running. It controls
271:   // whether several background threads should be running. It is set in
272:   // RpcAgent::start() and unset in the derived class shutdown().
```

- EN: Lines 257-272 introduces executable logic in routines such as `getDevices`.
- CN: 第 257-272 行在 `getDevices` 等例程中引入具体执行逻辑。

### Lines 273-288 / 第 273-288 行

```cpp
273:   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
274:   std::atomic<bool> rpcAgentRunning_;
275: 
276:  private:
277:   static std::shared_ptr<RpcAgent> currentRpcAgent_;
278:   // Add GIL wait time data point to metrics
279:   virtual void addGilWaitTime(const std::chrono::microseconds gilWaitTime) = 0;
280:   friend class PythonRpcHandler;
281: 
282:   // Map that stores metadata for RPC's that may need to be re-tried as well as
283:   // the timepoint at which we should re-try them.
284:   std::map<
285:       steady_clock_time_point,
286:       std::unordered_set<std::shared_ptr<RpcRetryInfo>>>
287:       rpcRetryMap_;
288: 
```

- EN: Lines 273-288 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 273-288 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 289-304 / 第 289-304 行

```cpp
289:   // Thread that checks for retryable RPC's in the rpcRetryMap_ and sleeps until
290:   // the next unACKed RPC's timeout has expired.
291:   std::thread rpcRetryThread_;
292: 
293:   // Function that rpcRetryThread_ calls in a loop as long as RpcAgent is
294:   // running.
295:   void retryExpiredRpcs();
296: 
297:   // This is the callback attached to futures corresponding to send retries.
298:   // This handles 3 cases: 1). send was completed, 2). send failed with an
299:   // error and we've done maxRetries failed send attempts, and 3). send
300:   // failed with an error and we have more retries to go. In case 1, we mark
301:   // the original future as complete. In case 2, we mark the future with an
302:   // error and do not retry again. In case 3, we move the RpcRetryInfo struct
303:   // to another time point in the map to schedule the RPC for a future send.
304:   void rpcRetryCallback(
```

- EN: Lines 289-304 introduces executable logic in routines such as `retryExpiredRpcs`.
- CN: 第 289-304 行在 `retryExpiredRpcs` 等例程中引入具体执行逻辑。

### Lines 305-320 / 第 305-320 行

```cpp
305:       JitFuture& message,
306:       steady_clock_time_point newTime,
307:       std::shared_ptr<RpcRetryInfo> earliestRpc);
308: 
309:   // Function that uses the exponential backoff algorithm to compute the next
310:   // time point to retry a given RPC.
311:   inline steady_clock_time_point computeNewRpcRetryTime(
312:       RpcRetryOptions& options,
313:       int retryCount) {
314:     // The exponential backoff algorithm being used here is:
315:     // newTime = timeNow + (retryDuration * (backoffConstant ^ retryCount)).
316:     std::chrono::milliseconds timedelta =
317:         std::chrono::duration_cast<std::chrono::milliseconds>(
318:             options.rpcRetryDuration * pow(options.retryBackoff, retryCount));
319:     return std::chrono::time_point_cast<std::chrono::milliseconds>(
320:         std::chrono::steady_clock::now() + timedelta);
```

- EN: Lines 305-320 introduces executable logic in routines such as `computeNewRpcRetryTime`; returns computed state or forwards results to the surrounding caller.
- CN: 第 305-320 行在 `computeNewRpcRetryTime` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 321-336 / 第 321-336 行

```cpp
321:   }
322: 
323:   // Condition Variable to signal when the rpcRetryMap_ has been populated.
324:   std::condition_variable rpcRetryMapCV_;
325: 
326:   // Mutex to protect RpcRetryMap_.
327:   std::mutex rpcRetryMutex_;
328: };
329: 
330: } // namespace torch::distributed::rpc
331: 
332: namespace std {
333: template <>
334: struct hash<torch::distributed::rpc::WorkerInfo> {
335:   std::size_t operator()(
336:       const torch::distributed::rpc::WorkerInfo& worker_info) const noexcept {
```

- EN: Lines 321-336 opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `hash`.
- CN: 第 321-336 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `hash` 等类型。

### Lines 337-340 / 第 337-340 行

```cpp
337:     return worker_info.id_;
338:   }
339: };
340: } // namespace std
```

- EN: Lines 337-340 opens or closes namespaces to place the code in the correct distributed component; returns computed state or forwards results to the surrounding caller.
- CN: 第 337-340 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；返回计算结果，或将结果继续传递给外围调用方。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `TORCH_API`, `hash`
- CN: 核心符号：`TORCH_API`、`hash`
- EN: Notable themes: RPC/RRef semantics.
- CN: 值得关注的主题：RPC/RRef 语义。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/rpc/message.h`, `torch/csrc/distributed/rpc/request_callback.h`, `torch/csrc/distributed/rpc/types.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: None / 无
- External or system headers / 外部或系统头文件: `cctype`, `chrono`, `condition_variable`, `mutex`, `thread`
- Local symbols / 本地符号: `TORCH_API`, `hash`